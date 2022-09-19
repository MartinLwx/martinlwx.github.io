# How to use the semantic actions to generate the symbol tables in ANTLR4


## 什么是 semantic actions
当 Parser 处理输入的代码的时候不仅要判断是否语法和句法都正确，还可以执行一些有用的操作，这些操作就叫做 Semantic actions。**其实也就是一段代码**，一般嵌入在在语法文件的规则里面。那么**当 parser 应用这个规则**的时候就会执行你设置的这段代码。换个角度理解，semantic actions 其实**就是“触发器”**，触发条件就是 parser 应用了对应的规则。

今天这篇文章要探讨的就是一个关于 semantic actions 的应用——实现一个简单的 symbol table，用到的工具是 [ANTLR4](https://www.antlr.org/)。
## 什么是 symbol table
编译器在处理我们的代码的时候会在内部维护一个 symbol table，用来存储程序里面**所有关于变量的信息**：变量名、数据类型、变量所属的作用域等。symbol table 可以是下面这种形式：

<center>

| Symbol name |       Type       |        Scope       |
|:-----------:|:----------------:|:------------------:|
|     bar     | function, double |       extern       |
|      x      |      double      | function parameter |
|     foo     | function, double |       global       |
|    count    |        int       | function parameter |
|     sum     |      double      |     block local    |
|      i      |        int       | for-loop statement |

</center>

## 如何在 ANTLR 的语法文件里面注入动作
ANTLR4 是一个强大的 parse generator，我们只要编写好语法文件，就能让它帮我们自动生成 Parser，生成的 Parser 可以支持多种目标语言，比如我自己用的是 [Python](https://pypi.org/project/antlr4-python3-runtime/)，那么最后生成的 Parser 就是 Python 文件。

ANTLR4 也提供了方法让我们可以在语法文件里面插入动作，这些动作最后都会被 ANTLR4 注入到生成的 Parser 文件里面。**因此，动作用什么语言写取决于你输出 Parser 的目标语言是什么**

> **⚠️这里假定你对如何编写 ANTLR4 的语法文件（`*.g4`）有基本了解，不会进行赘述，本文只集中介绍如何在语法文件里面插入动作**

### 动作注入的位置和方法
下面是简化的生成的 Parser 文件的代码模板。总体上来看，我们能注入的位置如下所示：

```python
<header>
class xxxParser(Parser):    
    ...
    <member>
    def rule(self):
        ...
        <action>
```
下面我对每个位置的不同语法进行说明：

#### `<header>`
ANTLR4 本来就是用 Java 写的，所以如果你用的目标语言也是 Java 的话，这个会比较用得到。一般就是用来放 `import` 语句的。如果是 python 的话哪里都能 `import` 就比较没差。要往这个位置注入代码的格式如下（放在 `*.g4` 文件里面，下同）

```antlr
@header{
    everything here will go to <header>
}
```
#### `<member>`
这个放的就是类的成员，可以是字段也可以是方法。ANTLR4 支持往 Lexer 和 Parser 分别或者同时注入代码。要往这个位置注入代码的格式如下：

```antlr
@members {
    everything here will go to <member> in xxxLexer && xxxParser
}
@Lexer::members {
    everything here will go to <member> in xxxLexer
}
@Parser::members {
    everything here will go to <member> in xxxParser
}
```

> ⚠️在我使用的 `antlr4-python3-runtime` 中（4.10），还无法在插入类的字段进行注释

#### `<action>`
在 ANTLR4 中，动作是用**花括号 `{<specific-language-here>}` 括起来的代码**。正如前面提到的，看你最后输出的 Parser 想要是什么语言，你就用什么语言写动作。

动作一般会放在一条规则的某个 symbol 之后，意思是说在 parser 应用这条规则的时候执行到这里就执行相应的动作。这里的 symbol 可以是 terminal 也可以是 nonterminal。

**我们可以用 `$symbol.attr` 的方式访问到对应的属性**，有下面这几个：

```python
$terminal.text           # origin text
$terminal.type           # an integer stands for type
$terminal.attributes
$terminal.line           # the line number
$terminal.pos            # the position of the first char in the line，0-based
$terminal.index          
$terminal.channel        # the channel of this terminal, won't discuss in this post
$terminal.int            # return an interger if this terminal is an integer

$nonterminal.text            # origin text
$nonterminal.start           # 1st Token
$nonterminal.stop            # last Token
$nonterminal.ctx             # return context object
```
## 举个例子
> **⚠️下面都只会给出部分代码，完整的代码在我的 [Github 项目](https://github.com/MartinLwx/ECE573-Fall-2015-Purdue/blob/main/Project03.Symbol_table/Micro.g4)**

> ⚠️因为这个项目要求把 warning 输出在其他输出前面，所以我用 `warning_list` 和 `output_list` 来存储，最后再一起输出

下面以普渡大学 2015 年的开设的编译器课程的[项目作业](https://engineering.purdue.edu/~milind/ece573/2015fall/project/step3/step3.html)为例，整个项目要实现一个 Micro 语言的编译器，关于 Micro 语言的语法，可以在[这里](https://engineering.purdue.edu/~milind/ece573/2015fall/project/grammar.txt)看到。下面我会对步骤三的作业要求进行一个简单的介绍。

在这个作业中我们要构建一个 symbol table，并在相应的时刻输出相关的信息：

- 每当我们进入一个新的作用域之前（可以是函数，也可以是代码块），要进行相关的输出
- 如果遇到变量声明，就输出变量名和变量类型，有值的话也要输出。
- 如果声明的变量在**外部作用域**已经声明过的话要输出：`SHADOW WARNING <var_name>`，适用于有嵌套的作用域出现的情况
- 如果在**当前作用域**已经有同名的变量，就要输出：`DECLARATION ERROR <var_name>`。如果出现了这种情况，那么最后程序只输出这个信息

```
Symbol table <scope_name>
name <var_name> type <type_name>
name <var_name> type <type_name> value <string_value>
```

### 数据结构的选取及对应的方法
🤔先要解决的问题是，要用什么数据结构表示 symbol table？一个 symbol table 应该要满足下面的特点：

- 要能够支持高效率地查询
- 要能够支持高效率地插入，因为程序中可能包含大量的变量声明
- 要能够记录变量的作用域

不难想出，**支持高效率查询和插入的数据结构就是「哈希表」**。为了维护变量的作用域，**可以把同个作用域下的变量放在同个哈希表里面，用一个列表来记住所有的变量作用域**，也就是说是 `[{scope1}, {scope2},...]` 这样。同时维护一个 `self.current_scope` 来记住当前处于哪一个变量作用域里面。每当有一个新的作用域出现的时候，保存当前的作用域，往列表里插入一个新的作用域，再更新 `self.current_scope`。

> 📒 这里的 list 其实就是用来模拟栈

🤔我们要实现什么样的方法？起码要有下面这几个：

- `lookup(identifier, value)`：插入变量到当前的 symbol table 里面。根据这个项目作业的要求，还应该查询是否变量已经被声明过
- `enter_new_scope()`：保存当前的 symbol table，进入下一个变量作用域并初始化为空
- `exit_scope()`：清空当前的 symbol table，找到上一个 symbol table


所以对应 `@parser:members` 里面的代码如下

```python
@parser::members {
def init(self):
    self.current_scope = None
    self.block_count = 0
    self.warning_list = []                    # just for printing
    self.output_list = []                      # just for printing
    self.declaration_error = ''
    
def enter_new_scope(self):
    if not hasattr(self, '_scopes'):
        setattr(self, '_scopes', [])
    # save the current_scope
    import copy
    if len(self._scopes) > 0:
        self._scopes.append(copy.deepcopy(self.current_scope))
    self._scopes.append({})
    self.current_scope =  self._scopes[-1]

def exit_scope(self):
    del self._scopes[-1]
    if len(self._scopes) > 0:
        self.current_scope =  self._scopes[-1]

def lookup(self, identifier, value):
    # check all scopes
    found = False
    for scope in self._scopes[:-1][::-1]:
        #print(f"the scope: {scope}")
        if identifier in scope:
            found = True
    if found:
        self.warning_list.append(f"SHADOW WARNING {identifier}")
    # only record the 1st declaration error
    if identifier in self.current_scope and self.declaration_error == '':
        self.declaration_error = f"DECLARATION ERROR {identifier}"
    self.current_scope[identifier] = value
}
```
### 注入动作到变量声明中
🧐接下来就是要在对应的语法规则里面注入动作。我们想要在定义变量的时候输出相关的信息，所以我们先要观察在 Micro 语言中变量声明的规则是怎么样的，如下所示：

```antlr
...
var_decl    :   var_type id_list ';'  ; 
var_type    :   'FLOAT'              
            |   'INT' 
            ;          
any_type    :   var_type 
            |   'VOID' 
            ;
id_list     :   id id_tail ;
id_tail     :   ',' id id_tail 
            | 
            ;
...
```
可以看到，声明变量对应的是 `var_decl` 规则，一次可以声明一至多个变量，每一个变量之间用 `,` 隔开的，所以我们可以在这条规则的末尾最后注入如下的动作代码。

> ⚠️要注意如果你也是使用 python 的话，这里所以缩进会有点怪，因为每一行都要从最左边开始，但最后生成的代码是没有问题的

```antlr
...
var_decl    :   var_type id_list ';' 
            {
# NOTE: the indentation is correct, ANLTR4 will handle this for us :)
# for all variable declarations, we should output the name && type
# in the same variable declaration, it means all of the variables have the same type
for variable in $id_list.text.split(','):
    self.lookup(variable, None)
    self.output_list.append(f"name {variable} type {$var_type.text}")
            }
            ; 
...
```
我们用 `$id_list.text` 来获取本来的变量声明对应的文本，用 `$var_type.text` 来获取对应的变量类型
### 关于变量作用域的处理
下面以 `program` 规则为例（函数和代码块的类似），`progarm` 规则是 Micro 语法的起始规则，规定了 Micro 程序应该要有的大框架，规则如下所示：

```antlr
program :   'PROGRAM' id 'BEGIN' pgm_body 'END' ;
```

我们处理完 `PROGRAM` 这个 token 之后就可以初始化第一个作用域（全局作用域），最后要处理完程序再退出这个全局作用域，所以我们可以很快想到应该把动作注入在哪里

```antlr
program :   'PROGRAM' id 'BEGIN' pgm_body 'END' ;
                     ^                         ^
                     |                         2
```

最后插入相关的动作

```antlr
program :   'PROGRAM' 
        {
self.init()
self.output_list.append("Symbol table GLOBAL")
self.enter_new_scope()
        }   id 'BEGIN' pgm_body 'END' 
        {
self.exit_scope()
# output everything after we parsing this program
if self.declaration_error != '':
    print(self.declaration_error)
else:
    if len(self.warning_list) > 0:
        print('\n'.join(self.warning_list))
    print('\n'.join(self.output_list))
        }
        ;
```
## 扩展阅读
前面只涉及到简单的动作注入，ANTLR4 其实支持的功能还更多：

- 比如我们可以让 nonterminal 返回值
- 在同一条规则如果出现多个同名的 nonterminal 的话可以起名字

下面是来自书中的一个例子，就很好地展示了上面两个用法

```antlr
e returns [int v]      
    : a=e op=('*'|'/') b=e  {$v = self.eval($a.v, $op, $b.v)}
    | a=e op=('+'|'-') b=e  {$v = self.eval($a.v, $op, $b.v)}
    | INT                   {$v = $INT.int}    
    | ID
```
## 总结
从这个例子中我们可以学习到要如何在 ANTLR4 里面使用动作，生成 symbol table 只是其中一个运用。在代码里面注入动作是很符合直觉的做法，可以快速实现自己想要的功能。不过缺点也很明显，它是 language-dependent 的，也就是说你一旦换了 ANTLR 输出的目标语言，你注入的动作全部要改成新的目标语言。另外就是会让本来干净的语法文件一团糟。


## 参考
1. [Symbol table - wiki](https://en.wikipedia.org/wiki/Symbol_table)
2. 《ANTLR4 权威指南》

