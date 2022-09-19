# How to use the semantic actions to generate the symbol tables in ANTLR4

## What are the semantic actions
When the parser processes the input code, it not only determines whether the syntax is correct but also performs some useful actions. These actions are called semantic actions. **In fact, it is a piece of code**, which is generally embedded in the rules of the grammar file. Then **when the parser applies this specific rule**, the code you set will be executed. From another perspective, semantic actions are actually ***"triggers"***, and the trigger condition is that the parser applies the corresponding rules.

Today's post is about an application of semantic actions-implementing a simple symbol table, the tool used here is [ANTLR4](https://www.antlr.org/).
## What's the symbol table
When the compiler processes our code, it will maintain a symbol table internally, which is used to store all the information about variables in the program: variable name, data type, scope to which the variable belongs, etc. The symbol table can be of this form:

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

## How to use actions in the ANTLR4
ANTLR4 is a powerful parse generator. As long as we provide the grammar file, it can automatically generate a parser for us. The generated parser can support multiple target languages. For example, I use [Python](https://pypi .org/project/antlr4-python3-runtime) as my target language, then the final generated parser is a `xxxParser.py` file. The `xxx` is your grammar filename.

ANTLR4 also provides methods that allow us to insert actions into the grammar file, and these actions will eventually be injected into the generated parser file. **Therefore, the programming language in which the action is written depends on the target language of your output parser**

> **⚠️It is assumed here that you have a basic understanding of how to write ANTLR4 grammar files (`*.g4`), and will not go into details about this. The article only focuses on how to insert actions into grammar files.**

### Basic overview
Below is a simplified code template for the generated parser file. 

In general, the locations we can inject are as follows(`<...>` in the code):

```python
<header>
class xxxParser(Parser):    
    ...
    <member>
    def rule(self):
        ...
        <action>
```
I will explain the different syntax for each location:

#### `<header>`
The ANTLR4 is originally written in Java, so if the target language you use is also Java, this will be more useful. Usually, it is used to place `import` statements. The format of the code to be injected into this location is as follows (in the `*.g4` file, the same below)

```antlr
@header{
    everything here will go to <header>
}
```
#### `<member>`
This position is used to put the member of the class, which can be a *field* or a *method*. The ANTLR4 supports injecting code into Lexer and Parser separately or simultaneously. The format for injecting code into this location is as follows:
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

> ⚠️In the `antlr4-python3-runtime` edition I use (4.10), it is not yet possible to annotate the *fields*. You can't put `# comments` like this.

#### `<action>`
In ANTLR4, an action is a code enclosed in **curly braces `{<specific-language-here>}`. As mentioned earlier, it is depending on what language your final output parser wants to be.

Actions are generally placed **after a symbol in a rule**. When the parser applies the rule, the corresponding action will be executed after finishing matching the symbol. The symbol here can be terminal or nonterminal.

**We can use `$symbol.attr` to access the corresponding attributes**, there are the following:

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
## An example
> **⚠️Only part of the code is given below, the complete code is in my [Github repo](https://github.com/MartinLwx/ECE573-Fall-2015-Purdue/blob/main/Project03.Symbol_table/Micro.g4)**

> ⚠️Because this project requires printing warnings before other outputs, so I use `warning_list` and `output_list` to store them temporarily, and then finally print them together.

The following is an example of the [project assignment] (https://engineering.purdue.edu/~milind/ece573/2015fall/project/step3/step3.html) of the compiler course offered by Purdue University in 2015. The entire project requires to Implementing a compiler for Micro language. The grammar of the Micro language can be found [here](https://engineering.purdue.edu/~milind/ece573/2015fall/project/grammar.txt). Below I will give a brief introduction to the homework requirements.

In this assignment we need to build a symbol table and print relevant information at the corresponding moment:

- Whenever we enter a new scope (which can be a function or a code block)
- Whenever we encounter variable declarations
- If the declared variable has been declared in the **outer scope**, print: `SHADOW WARNING <var_name>`
- If there is already a variable with the same name in the **current scope**, print: `DECLARATION ERROR <var_name>`. If this is the case, then the final program only outputs this information

```
Symbol table <scope_name>
name <var_name> type <type_name>
name <var_name> type <type_name> value <string_value>
```

### What data structure should I use
🤔The first question to be solved is, what data structure should be used to represent the symbol table? A symbol table should meet the following characteristics:

- Efficient query
- Efficient insertion
- Record the scope of the variable

It is not difficult to figure out that the data structure that supports efficient query and insertion is the **"hash table"**. To maintain the scope of variables, **We can put variables in the same hash table, and use a list to remember all scopes**. The structure should be like: `[{scope1} , {scope2},...]`. In order to remember the current scope, I use the `self.current_scope` variable. Whenever a new scope appears, save the current scope and insert a new scope into the list. Don't forget to update the `self.current_scope` also.

> 📒Conceptually, the list here is a stack

🤔What method are we going to implement?

- `lookup(identifier, value)`: Insert a variable into the current symbol table. According to the requirements of this assignment, we should also query whether the variable has been declared before
- `enter_new_scope()`: Save the current symbol table, enter the new scope, and initialize it.
- `exit_scope()`: Clear the current symbol table and find the previous symbol table(the top of the stack)


So the corresponding code in `@parser::members` are as follows

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
### Inject actions into the grammar rule
🤔The next step is to inject actions into the corresponding grammar rules. We want to output relevant information when variable declarations happen, so we must first observe what the rules of variable declaration in the Micro language are. The corresponding grammar rules are as follows

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
As you can see, the `var_decl` rule shows the basic structure to declare variables. You can declare one or more variables at a time, and each variable is separated by `,`, so we can inject the following action code at the end of this rule.

> ⚠️Note that if you are also using python, the indentation here is a bit weird, because each line starts from the leftmost, but the final generated code is fine. the ANTLR4 will take care of this.

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

We use `$id_list.text` to get the text corresponding to the variable declaration, and `$var_type.text` to get the corresponding variable type.
### How to solve the scope problem
Takes the `program` rule as an example (functions and code blocks are similar). The `program` rule is the starting rule of the Micro grammar, which specifies the big framework that a Micro program should have. The rule is as follows:

```antlr
program :   'PROGRAM' id 'BEGIN' pgm_body 'END' ;
```

After processing the `PROGRAM` token, we can create and initialize the first scope (global scope), and finally, exit the global scope after finishing processing. So we can quickly figure out where to inject the action

```antlr
program :   'PROGRAM' id 'BEGIN' pgm_body 'END' ;
                     ^                         ^
                     |                         2
```

The final code should be like this:

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
## Extended reading
The aforementioned example only involves simple actions injection, but ANTLR4 supports more powerful ways:

- For example, we can make a nonterminal return value
- If there are multiple nonterminals with the same name in the same rule, you can alias them

Here's an example from the book that nicely demonstrates the above two usages:

```antlr
e returns [int v]      
    : a=e op=('*'|'/') b=e  {$v = self.eval($a.v, $op, $b.v)}
    | a=e op=('+'|'-') b=e  {$v = self.eval($a.v, $op, $b.v)}
    | INT                   {$v = $INT.int}    
    | ID
```

## Wrap up
From this example, we can learn how to use actions in ANTLR4. Generating the symbol table is just one of the applications. Injecting actions into the code is a very intuitive way to quickly implement the functions you want. However, the disadvantage is also obvious. It is language-dependent, which means that once you change the target language output by ANTLR, all the actions you have written must be changed. Also, The grammar file will be a mess


## Refs
1. [Symbol table - wiki](https://en.wikipedia.org/wiki/Symbol_table)
2. *The Definitive ANTLR 4 Reference*

