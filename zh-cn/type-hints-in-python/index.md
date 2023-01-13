# Python 的类型提示：是什么以及为什么


## 引言

一开始吸引我学习 Python 的是它的动态语言特性，以及它的**鸭子类型（Duck typing）系统——我们不关心具体的类型是什么，我们只关心它的行为**。得益于 Python 的动态语言特性，我们不需要声明具体类型，这很大程度上加快了开发速度，而且去掉了不少心智负担，再加上强大的第三方库支持，Python 成为了我最爱用的编程语言😺

而随着 PEP 484[^2] 的提出，Python 决定引入类型提示（Type hint），这似乎又向静态类型语言看齐了？**但其实非也，Python 仍然是一门动态编程语言，它的类型提示是可选项，加不加都可以，不会对程序运行产生影响**。

这样听起来似乎没有必要专门写这篇博客来介绍 Python 的类型提示特性，但我发现**写类型提示还是有不少好处的**：
- 可以使用类型检查工具对代码进行检查，*比如 [Mypy](https://github.com/python/mypy)*
- IDE 的代码补全会更加智能，推荐的 API 更准确，我们用错了 API 也能及时发现。**这可能是我选择写类型提示的最大动力**
- 对抗代码复杂性。**类型提示暴露了 API 的不少信息**。作为开发者，我们只要看一下这样的函数签名就能知道个大概，而不用经常去看文档


```python
! python --version
```

    Python 3.11.0


## 函数的类型提示语法

早在 Python 3.0，类型提示的语法就已经确定了下来[^1]：
- 函数参数：`name[: type] [ = default_val ]`，其中 `[]` 表示这是可选项
- 函数返回类型：用 `-> return_type` 表示
- 我们可以通过函数的 `__annotations__` 属性访问到参数的类型信息，该属性**返回一个字典，key 为参数名，value 为类型**。==不推荐直接访问该属性，而应该通过 `inspect` 模块（> Python 3.10）或者 `typing` 模块（Python 3.5 ~ 3.9）里面的对应方法来获取这个信息==。如下所示：


```python
def maximum(a: float, b: float) -> float:
    """ A simple function to return the maximum elements of two floats"""
    return max(a, b)


# >= Python 3.10, do this
import inspect
assert inspect.get_annotations(maximum) == maximum.__annotations__

# Python 3.5 ~ 3.9
import typing
assert typing.get_type_hints(maximum) == maximum.__annotations__

inspect.get_annotations(maximum)
```




    {'a': float, 'b': float, 'return': float}



> 📒 再次强调，类型提示信息不会对程序的运行产生任何影响，这意味着**即使我们违背了类型提示信息，程序也可以正常运行**。只是**静态代码检查工具会对此抛出警告**


```python
# returns the maximum of two strings
# , but we declared the arguments should be float!
maximum('hello', 'world')     
```




    'world'



## 变量的类型提示语法

在 Python 3.6 之前，如果要给一个变量加上类型提示只能使用 Type comments，也就是在注释里面用 `# type ...` 声明[^2]，但在 PEP 526 中提出了变量的类型提示语法，和函数参数的语法是类似的[^3]


```python
a: int        # undefined typed value
a: int = 0    # typed value with default value
a
```




    0



类似的，我们可以通过 module 的 `__annotations__` 属性访问类型提示信息


```python
__annotations__
```




    {'a': int}



## 常见使用场景

### 简单的内建类型

这里说的简单内建类型就是：`int`、`str` 等，也**可以是第三方库里面定义的类型**。*前面提到的 `maximum` 函数的 2 个参数就是用 `float`*

### `Any` 类型

**`Any` 表示什么类型都有可能**。但它跟 `object` 并不相同[^2]

基本上我们可以认为一个不包含类型提示的函数的参数类型和返回类型都是 `Any`
```python
def foo(x):
    ...

# it assumes:
def foo(x: Any) -> Any:
    ...
```

### 集合类型和映射类型

我们称集合里面的每个东西为 item，那么我们如何给集合和 item 都加上类型提示信息？==Python 采用 `[]` 记号支持这个特性，`[]` 里面指定 item 的类型==。比如，一个包含字符串的列表这个类型可以写作 `list[str]`，非常清楚。

注意：Python 3.9 发布了 PEP 585 [^6]，允许我们直接使用自带的 `list`、`dict` 等而无需使用 `typing` 模块对应的类型。下面我列出了不同，更为详细的请查阅原文 [^6]：

| < Python 3.9                        | >= Python 3.9                            |
| ----------------------------------- | ---------------------------------------- |
| `typing.Tuple`                      | `tuple`                                  |
| `typing.Dict`                       | `dict`                                   |
| `typing.List`                       | `list`                                   |
| `typing.Set`                        | `set`                                    |
| `typing.Frozenset`                  | `frozenset`                              |
| `typing.Type`                       | `type`                                   |
| `typing.AbstractSet`                | `collections.abc.Set`                    |
| `typing.ContextManager`             | `contextlib.AbstractContextManager`      |
| `typing.AsyncContextManager`        | `contextlib.AbstractAsyncContextManager` |
| `typing.Pattern, typing.re.Pattern` | `re.Pattern`                             |
| `typing.Match, typing.re.Match`     | `re.Match`                               |

> 📒 **`typing` 模块的部分特性将来会被移除。所以我后面会使用最新的语法✏️**


```python
string_list: list[str] = ['hello', 'world']
    
# tuple[type1, type2, ..., typen] with fixed size
date: tuple[int, int, int] = (2023, 1, 11)
    
string_count: dict[str, int] = {
    'hello': 1,
    'world': 2,
}
```


```python
__annotations__
```




    {'a': int,
     'string_list': list[str],
     'date': tuple[int, int, int],
     'string_count': dict[str, int]}



下面的 `join_str_list` 函数接受一个包含字符串的列表，用空格将他们连接起来，然后返回这个字符串


```python
def join_str_list(string_list: list[str]) -> str:
    """ join all string in a list"""
    return ' '.join(string_list)

print(join_str_list(string_list))
print(inspect.get_annotations(join_str_list))
```

    hello world
    {'string_list': list[str], 'return': <class 'str'>}


> 📒 在 Python 3.9+，我们可以使用 `tuple[type1, ...]` 表示类型全都是 `type1` 的任意长度的 `tuple`


```python
def sum_variable_integers(data: tuple[int, ...]):
    """ Sum all integers of a tuple"""
    sum_val = 0
    for integer in data:
        sum_val += integer
    
    return sum_val

print(sum_variable_integers((1, 2, 3)))
print(sum_variable_integers((3,)))
```

    6
    3


众所周知，Python 的 `list` 可以放置任何类型，那么如何用类型提示说明这一点呢？可以用前面的 `Any` 类型：
```python
list[Any]
```

其实直接用 `list` 就可以，而且更为简洁

> 📒 如果这 3 个基本的集合类型不能满足你的要求，可以查阅 [`collections.abc`](https://docs.python.org/3/library/collections.abc.html) 的文档了解更多

### 类型别名（Type alias）

有时候，类型会变得很复杂而且很长，我们并不想要在每一个地方都写上一长串的类型提示。那么我们应该怎么做呢？一个通常的做法是为其取一个有意义的别名。定义别名的方法也很简单：
```python
AliasName = Type
```

*拿前面的 `date` 日期类型作为例子。`date` 类型被我定义为一个 `tuple`，每个位置上的类型都是 `int`。那么，`list[tuple[int, int, int]` 表示一个日期的列表这个类型*。为了让代码更有可读性，我们可以给他起个别名叫做 `Date`，看下面的例子：


```python
Date = tuple[int, int, int]
DateList = list[Date]

def print_date_list(l: DateList):
    """ Print all dates in the format `year-month-day` in the date list"""
    for year, month, day in l:
        print(f'{year}-{month}-{day}')
    
print_date_list([(2022, 1, 1), (2023, 1, 3)])
print(inspect.get_annotations(print_date_list))
```

    2022-1-1
    2023-1-3
    {'l': list[tuple[int, int, int]]}


类型别名的语法很容易和定义全局变量混淆，所以 PEP 163（>= Python 3.10）提出了更加 explicit 的方式[^7]：
```python
AliasName: TypeAlias = Type
```


```python
from typing import TypeAlias

Date: TypeAlias = tuple[int, int, int]
```

### 参数化的泛型

其他编程语言中经常使用大写字母比如 `T` 等表示参数化的泛型，在 Python 里面，我们使用 `TypeVar` 关键字，如同文档所说，一共有 3 种用法：
```python
T = TypeVar('T')            # Can be anything
S = TypeVar('S', bound=str) # Can be any subtype of str
A = TypeVar('A', str, bytes)# Must be exactly str or bytes
```

总的来说，`TypeVar` 提供了 2 种方式让我们对泛型作出限制：
1. 用 `bound=some_type`，那么我们就只能传入 `some_type` 的 *subtype*
2. 直接指定允许的类型


> 📒 关于 subtype 的定义可以参考 PEP 483[^4]，概括来说为 2 点：a）每个类型都是自己的 subtype。b）OOP 中，子类是父类的 subtype


```python
from typing import TypeVar

GenericString = TypeVar('GenericString', str, bytes)

def process(s: GenericString):
    """ The GenericString can be either str or bytes"""
    ...
```

> 📒 实际上 Python 已经帮你定义好了可能是 `str` 也可能是 `bytes` 的类型——`typing.AnyStr`

### Optional 和 Union 类型

`Optional[type1]` 用于表示类型可能是 `type1` 也可能是 `None`。

`Union[type1, type2, ...]` 表示允许的类型**可能是我们指定的多个可能类型的的任意一种**，也就是逻辑上的「或」关系**。因此，`Optional[type1]` 其实和 `Union[type1, None]` 是一个意思


```python
from typing import Optional, Union

def parse(s: Union[str, int]) -> Optional[int]:
    """ Parse `s` and get an integer value. The `s` may be a string.
    Return None if fail
    """
    if isinstance(s, str):
        if not s.isdigit():
            return None
        else:
            return int(s)
    elif isinstance(s, int):
        return s

assert parse('foo') is None
assert parse('123') == 123
assert parse(123) == 123
inspect.get_annotations(parse)
```




    {'s': typing.Union[str, int], 'return': typing.Optional[int]}



在 Python 3.10 里面，新引入了 `|`，可以用来代替 `Union` 的写法[^5]。其他语言中也可以见到这种用法：*比如 Rust 的 pattern matching 也采用了 `|` 分隔多个可能的 pattern*。那么我们前面写的 `parse` 函数就可以写成下面这种形式：


```python
def parse(s: str | int) -> int | None:
    """ Parse `s` and get an integer value. The `s` may be a string.
    Return None if fail
    """
    if isinstance(s, str):
        if not s.isdigit():
            return None
        else:
            return int(s)
    elif isinstance(s, int):
        return s

inspect.get_annotations(parse)
```




    {'s': str | int, 'return': int | None}



### 可调用对象（Callable）

下面用函数作为例子，**Python 的函数是 first-class object，所以函数可以是另一个函数的参数或者返回值**。类型提示对此的支持是：
```python
Callable[[ParamType1, ParamType2], ReturnType]
```

*让我们定义一个 `apply` 函数，入参为可调用对象和要处理的数据，将这个可调用对象应用在数据上*


```python
# from typing import Callable   # Python < 3.9
from collections.abc import Callable

def apply(f: Callable[[str | int], int | None], data: list):
    """ Apply callable object on data. The `Callable[[str | int], int | None]` 
    is the type hints of `parse` we aforementioned
    """
    for d in data:
        print(f(d))

apply(parse, ['hello', 123])
```

    None
    123


## 总结

以上就是本篇博客的全部内容，我其实只谈到了我认为比较有用的一些类型提示使用场景，不少的高级特性比如 Static protocol[^8] 等没有提到，这些内容留给读者自己探索。

我对类型提示的态度仍然是：**当它变得复杂的时候就不用，除非这个代价是值得的**。

根据我的开发经验，下面附上关于类型提示的几点建议🎯：
1. **决定要加上什么类型提示的时候，考虑这个类型能够做什么**。Python 3.8 提出的 static protocols 就很好解决了这点[^8]。感觉有点像 Rust 中的 traits
2. **对于函数返回类型，力求返回的类型越精确越好**。*设想一个返回类型为 `Union[str, int]`，我们还需要自己手动再进行一次类型检查，就显得这个类型提示不是很有必要*
3. **通过了代码静态类型检查也不意味着程序就没有 bug 了**，软件测试才是软件工程领域保证软件质量的标准做法～
4. 将这份 [cheatsheet](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html) 加入你的书签👍

## 参考

[^1]: [PEP 3107. Function Annotations](https://peps.python.org/pep-3107/)

[^2]: [PEP 484. Type Hints](https://peps.python.org/pep-0484/)

[^3]: [PEP 526. Syntax for Variable Annotations](https://peps.python.org/pep-0526/)

[^4]: [PEP 483. The Theory of Type Hints](https://peps.python.org/pep-0483/)

[^5]: [PEP 604. Allow writing union types as X | Y](https://peps.python.org/pep-0604/)

[^6]: [PEP 585. Type Hinting Generics In Standard Collections](https://peps.python.org/pep-0585/)

[^7]: [PEP 613. Explicit Type Aliases](https://peps.python.org/pep-0613/)

[^8]: [PEP 544. Protocols: Structural subtyping (static duck typing)](https://peps.python.org/pep-0544/)


