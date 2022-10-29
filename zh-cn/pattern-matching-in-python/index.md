# 模式匹配简明教程（Python 3.10）


> 这一篇博客一开始是写在 jupyter notebook 里面而后转成 markdown 的。如果想要访问和运行本来的 notebook，请查阅这个[仓库](https://github.com/MartinLwx/oh_my_python)

## 引言

今天我想要谈一下 Python 3.10 提出的新特性[^1]——模式匹配🎉

学习过 C 语言的人想必对下面的 `switch` 语句不陌生，

```c
switch (expression)
{
    case constant_1:
      // statements
      break;

    case constant_2:
      // statements
      break;
      
    // Fall through
    // the value of the expression can be either constant_3 or constant_4 :)
    case constant_3:
    case constant_4:
      
      // statements
    
    
    default:
      // default statements
}
```

概括一下，C 语言的 `switch` 语句的几点规则：
1. `expression` 必须是 `int` 或者 `char` 类型；`constant` 必须是 `int` 或 `char` 常量
2. `switch` 语句的执行过程： 计算出 `expression` 的值，拿着这个值**从上到下**检查是否和某一条 `case` 语句的 `constant` 相等，如果一样就会执行**里面的 statements 和后面的 `case` 语句的 statements，除非遇到了 `break`**。这个特性叫做 *Fall through*，我们可以利用这个特性，将多个 `case` 语句堆叠在一起，表示逻辑上的「或」的关系
3. 存在 `default` 表示默认情况，用于兜底，当前面的 `case` 语句都匹配失败的时候执行

Python 则并没有提供 `switch` 语句，我们可以用 `if...elif..elif..else` 达到同样的效果，*举例来说，假设我们要根据 `list` 的长度执行不同的操作，我们可以这么写*：


```python
some_list = [1, 2, 3, 4, 5]

if len(some_list) == 1:
    # do something when the length is 1
    ...
# or more pythonic way: elif len(some_list) in [3, 5]:
elif len(some_list) == 3 or len(some_list) == 5: 
    # do something when the length is 3 or 5
    ...
else:
    ...
```

上面这一连串的 `if...elif..elif..else` 其实可读性稍微差些，另外它还违反了 DRY(Don't repeat yourself) 原则，我们多次写下 `len(some_list)`

当然我们可以选择先用一个变量 `length` 记住 `some_list` 的长度，这样就可以让我们少打一些代码，但若情况更复杂写，这个技巧也不适用了

解决上面的一个更优雅的方式就是本文要讲到的：**Pattern matching** ⬇️


```python
match len(some_list):
    case 1:
        # do something when the length is 1
        ...
    case 3 | 5:
        # do something when the length is 3 or 5
        ...
    case _: # equal to the `default:`
        ...
```

## 基本的语法

下面我给出 Pattern matching 的基本语法

```python
match subject:
    case <pattern_1>:
        <action_1>
    case <pattern_2>:
        <action_2>
    case <pattern_3>:
        <action_3>
    # [Optional] wildcard to cover all situations
    case _:
        <action_wildcard>
```

从语法上看，和前面 C 语言的 `switch` 语句差不多。区别在于：
1. 虽然都是从上到下进行检查，但是 Python 的 pattern matching 不存在 Fall through 情况，只会执行匹配的 `case` 语句里面的代码。执行完就退出。所以也不用在每个 `case` 的代码块里最后加一个 `break`
2. 没有 `default` 关键字，但是我们可以用 `case _` 来捕获所有的情况，这其实用到了后面会讲到的 *Wildcard pattern*
3. 这里的 `subject` 和 `pattern` 比 C 语言的强大多了，**不仅仅是整型和字符类型**，`pattern` 彼此之间还可以**组合嵌套**。后面会对其进行详细说明

## Patterns

在 pattern matching 里面，Pattern 主要有下面两个作用：
1. 对 `subject` 的结构进行约束（structure constrait）
2. 可以使用变量绑定 `subject` 的某些部分（bind variables），用于后续的处理，见 *Capture pattern*

下面我们对不同的 patterns 进行探讨 :)

为了避免造成困惑，有必要提前进行一下说明：在 pattern matching 匹配序列时候，`()` 和 `[]` 都是可选的。*比如，`case foo, bar` 和 `case (foo, bar)` 和 `case [foo, bar]` 都是等价的*。**这点和我们给序列做 unpacking 的时候一致**

### Capture pattern

*Capture pattern* 的意思是说我们在检查 `pattern` 是否匹配的时候，可以用**变量名绑定到它的任意一个部分**，我们就可以**在匹配成功之后使用这些变量**


```python
some_list = ["foo", "bar"]

match some_list:
    # we want to match a seq which has length = 2
    # , we also use `first` and `second` to capture \
    # the 1st and 2nd elements here.
    case [first, second]:
        # we can access first, second now
        print(f'the 1st element: {first}, 2nd element: {second}')
```

    the 1st element: foo, 2nd element: bar


经常跟序列打交道的人想必对下面的代码不会陌生，我们**可以用 `*<name>` 来 unpacking**


```python
*before, last = [1, 2, 3, 4]              
assert last == 4, "Error"

first, *middle, last = [1, 2, 3, 4]
assert first == 1 and last == 4, "Error"

first, *rest = [1, 2, 3, 4]
assert first == 1, "Error"
```

类似的，在 pattern matching 里面也可以这样，**用一个变量捕获多个值**


```python
some_list = ["foo", "bar", "another_foo", "another_bar"]

match some_list:
    # we want to match a seq
    # , we also use `*rest` to capture the remaining elements
    case [first, *rest]:
        print(f'the 1st element: {first}, 2nd element: {rest}')
```

    the 1st element: foo, 2nd element: ['bar', 'another_foo', 'another_bar']


### Literal pattern

*Literal pattern* 指的是我们可以**指定字面值来对 Pattern 进行约束**。这里的字面值可以是 number literal, string literals, `True`, `False` 还有 `None`

> 📒 对于 number literals 和 string literals 这两者 Python 会使用 `==` 进行比较，而对于 `True/False/None` 这三个则是使用 `is` 来进行判断。注意这个细节


```python
some_list = ["foo", "bar"]

match some_list:
    # we want to match a seq which has length = 2
    # , the 1st element should be equal to "foo"
    # , and we use `second` to capture the 2nd element
    case ["foo", second]:
        print(f'the 2nd element: {second}')
```

    the 2nd element: bar



```python
some_list = [True]

match some_list:
    case [1]:
        print(f'Matched, 1 == True')
```

    Matched, 1 == True


### Wildcard pattern

这个其实在 Python 里面挺常见，我们常常使用 `_` 表示我们不关心某个变量是多少。在 pattern matching 里面，`_` 会和任何的东西匹配，但是**不会绑定任何变量**


```python
some_list = ["foo", "bar"]

match some_list:
    # we want to match a seq which has length = 2
    # , the 1st element should be equal to "foo"
    # , and we use `_` to ignore the 2nd value
    case ["foo", _]:
        print(f'the 2nd value: {_}')   
        # you should see empty output because we aren't binding value here
```

    the 2nd value: 


另外一个常见的用法就是之前出现过的 `case _`，因为 `_`会匹配任何情况，所以常常把 `case _` 放在最后表示默认情况


```python
some_list = ["foo", "bar"]

match some_list:
    # this case branch will not be matched
    case ["bar", _]:
        print('Match successfully')
    case _:
        print('Default case')
```

    Default case


### Or pattern

就像 `if` 条件语句我们可以使用 `or` 表示多种可能的匹配情况，在 pattern matching 里面我们也有类似的语法。跟其他大多数语言一样，Python 选择使用 `|` 来表达「或」的逻辑关系。我们可以很方便声明备选项


```python
some_list1 = ["foo"]
some_list2 = ["bar"]

match some_list1:
    # we want to match a seq which has length = 1
    # , the 1st element can be "foo" or "bar"
    case ["foo" | "bar"]:
        print('[First match]  Match foo or bar')

match some_list2:
    case ["foo" | "bar"]:
        print('[Second match] Match foo or bar')
```

    [First match]  Match foo or bar
    [Second match] Match foo or bar


上面的 Or pattern 的缺点是：我们无法知道我们具体匹配到了什么哪一个

### As pattern

在上一个例子中，我们可能匹配到多个选项，那么**如何知道我们具体匹配到哪一个选项**呢？因为我们**可能需要根据具体匹配到的东西来决定要如何处理**。在 pattern matching 里面，可以使用 `as` 来绑定变量


```python
some_list = ["foo"]

match some_list:
    # we want to match a seq which has length = 1
    # , the 1st element can be "foo" or "bar"
    # we bind matched string literal with `matched_element`
    case ("foo" | "bar") as matched_element:
        print(f'Match {matched_element}')
```

### Class pattern

Python 是一个动态类型语言，有时我们也会有**需要根据类型来决定是否要匹配**的时候。我们当然可以选择自己在后面使用 `isinstance()` 来判断，但还有更好的方法。下面我将从从基本的例子出发带大家看看**如何加上类型约束**


```python
some_list = ["foo", 1, 3.14]

match some_list:
    # match without type constraints
    case [s, v1, v2]:
        if isinstance(s, str) and isinstance(v1, int) and isinstance(v2, float):
            print(f'Match {s} - {v1} - {v2}')
```

    Match foo - 1 - 3.14


第一个写法：考虑用 *Capture pattern*，类型约束放代码块里


```python
some_list = ["foo", 1, 3.14]

match some_list:
    # match with type constraints
    case [str() as s, int() as v1, float() as v2]:
        print(f'Match {s} - {v1} - {v2}')
```

    Match foo - 1 - 3.14


此时我们在 `pattern` 里加上类型约束，这里写法上类似 *Literal pattern*，我们在对应位置声明我们想要匹配的类型，同时为了后面能输出，我们还需要 `as` 关键字将其绑定到变量上。但上面的写法**过于冗长**，好在 Python 为我们提供了语法糖🍬


```python
some_list = ["foo", 1, 3.14]

match some_list:
    # match with type constraints
    case [str(s), int(v1), float(v2)]:
        print(f'Match {s} - {v1} - {v2}')
```

    Match foo - 1 - 3.14


### Mapping pattern

前面都是对于一个序列的匹配，这里则是对 `dict` 的匹配。相信在看完前面的各种 pattern 的例子之后，理解 `dict` 的匹配也没有什么难度。但有下面几点注意事项：
1. `dict` 的匹配是通过限制 Key-Value 的结构。其中 Key 必须是字面值或者枚举类型的值（出于性能的考量），Value 则没有这个限制
2. 使用 `**<name>` 来捕获我们没有写在 `pattern` 里面的 Key-Value pair。否则**默认是忽略掉的**
	- 但是 `**_` 是不行的，因为本来就忽略掉，而 `**_` 中的 `_` 表示不绑定任何匹配的东西，纯粹是多此一举


```python
some_dict = {
    'first_name': 'foo', 
    'second_name': 'bar'
}

match some_dict:
    case {'first_name': first_name}:
        print(f'[First match]  The first_name: {first_name}')
      
    
match some_dict:
    case {'first_name': first_name, **rest}:
        print(f'[Second match] The rest: {rest}')
```

    [First match]  The first_name: foo
    [Second match] The rest: {'second_name': 'bar'}


### Value pattern

使用「有名字的变量」作为参数值或澄清特定值的含义是很好的编程风格，这是 *Literal pattern* 欠缺的。*比如 `case (HttpStatus.OK, body)` 是比 `case (200, body)` 好的*

在 Python 里面要实现 *Value pattern* 的挑战是要和前面的 *Capture pattern* 区分开，要让 Python 可以区分我们是要加一个「有名字的常量」这个约束还是我们在使用 *Capture pattern* 绑定变量。关于这点的讨论可以参见[^2]

最后 Python 提供的解决方案是一个受限的 *Value patter*n，它**仅支持 `foo.bar` 这种形式的 *Value pattern***。比较常见的就是用于枚举类型，看下面这个例子


```python
from enum import Enum

class HttpStatusCode(Enum):
    CONTINUE = 100
    OK = 200
    
some_list = [HttpStatusCode["OK"]]

match some_list:
    case [HttpStatusCode.OK as status_code]:
        print(f"Receive {status_code}")
```

    Receive HttpStatusCode.OK


## 将 Pattern matching 用在一个类上

如果只能将 pattern matching 用在内建的类型上，似乎用处没有那么大。但其实 Python 还允许我们对自己自定义的类的对象使用 pattern matching。

考虑到应用场景，我们对一个对象做 pattern matching 的时候**常常是想要检查这个对象是否为某个类**，我们**还可能关心它的某些字段**，想要**提取对应的字段的值**。但是在 Python 里，这个实现起来有困难[^2]，主要是类的字段非常多，大部分是 `__repr__` 这种 magic methods，而且这些**字段是无序的**。因为是无序的，我们无法直接在 `pattern` 里面**按位置绑定变量**，看下面这个例子：


```python
class Point:
    """ A simple class represents a Point in a 2D"""
    def __init__(x: int, y: int):
        self.x = x
        self.y = y
    
some_point = Point(1, 2)

match some_point:
    # the intuitive way, we want to match a Point type
    # , and we want to bind the `x` and `y` and their two fields respectively
    case Point(x, y):
        print(f"The x: {x}")
        print(f"The y: {y}")
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Input In [17], in <cell line: 7>()
          4         self.x = x
          5         self.y = y
    ----> 7 some_point = Point(1, 2)
          9 match some_point:
         10     # the intuitive way, we want to match a Point type
         11     # , and we want to bind the `x` and `y` and their two fields respectively
         12     case Point(x, y):
         13         print(f"The x: {x}")
         14         print(f"The y: {y}")


    TypeError: Point.__init__() takes 2 positional arguments but 3 were given


Python 提供了两种解决办法，从语法上看，跟我们在调用函数的时候非常像：我们可以选择按照位置传递参数，也可以选择用 `foo=bar` 这种形式

先说简单的这种：**用 `foo=bar` 的形式对字段进行约束**，并且**绑定变量到字段上**。 *`foo=bar` 的意思是我们要求这个类有 `foo` 字段，同时我们想要将 `bar` 绑定到实例上的 `foo` 字段上*


```python
class Point:
    """ A simple class represents a Point in a 2D"""
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y
    
some_point = Point(1, 2)

match some_point:
    # the intuitive way, we want to match a Point type
    # , and we want to bind the `x` and `y` and their two fields respectively
    case Point(x=x, y=y):
        print(f"The x: {x}")
        print(f"The y: {y}")
```

    The x: 1
    The y: 2


另外一种解决办法是：修改类的 `__match_args__` 属性，该属性**规定了字段的顺序**


```python
class Point:
    """ A simple class represents a Point in a 2D"""
    
    # we tell python that the order is first "x" and then "y"
    __match_args__ = ("x", "y")
    
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y
    
some_point = Point(1, 2)

match some_point:
    # the intuitive way, we want to match a Point type
    # , and we want to bind the `x` and `y` and their two fields respectively
    case Point(x, y):
        print(f"The x: {x}")
        print(f"The y: {y}")
```

    The x: 1
    The y: 2


如果你对 `@dataclass` 很熟悉的话[^3]，上面的代码可以大大简化，看下面


```python
from dataclasses import dataclass

@dataclass(match_args=True)
class Point:
    """ A simple class represents a Point in a 2D"""
    x: int
    y: int
        
print(f"The order is {Point.__match_args__}")
    
some_point = Point(1, 2)

match some_point:
    # the intuitive way, we want to match a Point type
    # , and we want to bind the `x` and `y` and their two fields respectively
    case Point(x, y):
        print(f"The x: {x}")
        print(f"The y: {y}")
```

    The order is ('x', 'y')
    The x: 1
    The y: 2


## Guard

有时候我们不仅关心模式是否匹配，我们还要加上某些限制。

试考虑这么一种情况，你要匹配有两个 `int` 值的序列，但是第一个元素要比第二个大，那要怎么写呢？结合前面的 Class pattern，我们不难写出下面的代码：


```python
some_list = [3, 4]

match some_list:
    case [int(first), int(second)]:
        if first > second:
            ...
        else:
            print("Expect first > second. Match failed")
```

    Expect first > second. Match failed


上面的写法固然可以，我们在代码块里面自己用 `if` 语句再检查一遍就行。但就像类型约束一样，Python 已经考虑到了这个需求，因此它提供了 **Guard 💂‍♀️ 机制**，使得我们可以把 `if` 语句这个判断挪到 `pattern` 的后面。这样*可读性会强很多*。遵循的语法规则如下所示：

```python
match subject:
    case <pattern> if <expression>:
        ...
```

1. 在 `<pattern>` 后面跟上一个 `if` 语句，用来**在 `<pattern>` 匹配之后**对其进行限制
2. 注意 Python 在这里 Evaluate 的顺序
	1. 先看 `<pattern>` 是否匹配
	2. 匹配的话，如果有绑定变量就绑定对应的变量
	3. 此时再看 `if <expression>` 语句是否返回 `True`。这里的 `<expression>` 可以用上一步绑定的变量。
3. 当且仅当 `<pattern>` 匹配 + `if` 语句返回 `True` 的时候才会执行相应的代码块。否则检查不通过，继续尝试匹配下一个 `<pattern>`


```python
some_list = [3, 4]

match some_list:
    case [int(first), int(second)] if first > second:
        print("Match successfully!")
```

## 总结

相比于使用 `if...elif...elif...else`，我会更喜欢 pattern matching 多一些，出于下面几点原因：
1. 我们可以很方便地在匹配的时候绑定值用于后续处理
2. 个人觉得可读性比较强，代码看起来没有那么乱
3. pattern matching 的各种 patterns 其实是可以嵌套组合的，这也是 pattern matching 真正强大的地方


上面就是 Python 3.10 引入的 pattern matching 的简短介绍🚀

## 参考

[^1]: [What's new in Python 3.10](https://docs.python.org/3/whatsnew/3.10.html)

[^2]: [PEP 635 – Structural Pattern Matching: Motivation and Rationale](https://peps.python.org/pep-0635/)

[^3]: [dataclasses - documentation](https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass)


