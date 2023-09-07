# Type hints: what and why


## Intro

I was immediately drawn to Python when I first encountered it due to its dynamic language features. Python use the "duck typing" design, which means that the type of an object is not as important as its behavior. This feature allows for faster development and a reduction in burdensome type declarations. Additionally, the support of powerful third-party libraries solidifies Python as my preferred programming language.😺

However, with the proposal of PEP 484[^2], Python decided to introduce type hints, which seem to be in line with statically typed languages. **It’s not true though, Python’s type hints are optional, and it has no runtime effect**. 

It seems that writing this blog post specifically to introduce Python's type hints is unnecessary, but I have found that using them in my code still provides **several benefits**:
- Static type checker can check your code. *For example, [Mypy](https://github.com/python/mypy)*
- The code completion in IDE will become more intelligent. It will also report a bug if we use the wrong APIs. **This is probably the biggest motivation for me to choose to write type hints**
- Manage code complexity. **Type hints expose useful information about APIs**. As a developer, we can get a general idea by just looking at the signature of such an annotated function, without having to check the docstrings frequently.


```python
! python --version
```

    Python 3.11.0


## Function annotations

Shipped with Python 3.0, the syntax of type hints has been established[^1]:
- Arguments: `name[: type] [ = default_val ]`, the `[]` means optional
- Return type: the syntax is `-> return_type`
- We can access the `__annotations__` attribute to get type hints informatios. It returns a dict with `{ParamName: ParamType}`. ==It's a bad practice to access this attribute directly. We can use the functions inside the `inspect` module(Python 3.10+) or the `typing` module(Python 3.5 ~ 3.9)==. See the following example:


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



> 📒 It's important to reiterate that type hints in Python have no impact on the runtime of the program. This means that even if we violate the type hints, the program will still execute as normal. However, a static type checker like Mypy will throw warnings.


```python
# returns the maximum of two strings
# , but we declared the arguments should be float!
maximum('hello', 'world')     
```




    'world'



## Variables annotations

Prior to Python 3.6, the only way to annotate the type of a variable was to include this information in comments, such as `# type ...`[^2]. This is referred to as "type comments." With the introduction of PEP 526, a new syntax for variable annotations was established, which is similar to the syntax used for annotating function arguments[^3]


```python
a: int        # undefined typed value
a: int = 0    # typed value with default value
a
```




    0



Similarly, we can access the module level `__annotations__` attributes to get type hints information.


```python
__annotations__
```




    {'a': int}



## Common usage

### Simple built-in types

The simple built-in types refer to `int`, `str` etc. They can also be types defined in third-party packages. *As an example, the arguments of the previously mentioned maximum function use float.*

### `Any` type

**The `Any` type denotes that it can be any type**. But it's different from the `object` type[^2]

We can think of an unannotated function that is *annotated* with the `Any` type.

```python
def foo(x):
    ...

# it assumes:
def foo(x: Any) -> Any:
    ...
```

### Collections and Mappings

We refer to each element inside a collection as an *item*. To add type hints for both the collection type and the item type, Python uses the `[]` notation. For example, to express a list of strings, we would write `list[str]`. This notation makes it clear that the list contains elements of the `str` type.

Notes: After PEP 585(Python 3.9+)[^6], we can use the built-in `list`, `dict` etc instead of the counterparts in the `typing` module[^6].


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

> 📒 **Some features of the `typing` module will be removed in the future, so I will use the latest syntax below✏️**


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



The `join_str_list` function accepts a list of strings and uses the whitespace to join them.


```python
def join_str_list(string_list: list[str]) -> str:
    """ join all string in a list"""
    return ' '.join(string_list)

print(join_str_list(string_list))
print(inspect.get_annotations(join_str_list))
```

    hello world
    {'string_list': list[str], 'return': <class 'str'>}


> 📒 In Python 3.9+, we can use `tuple[type1, ...]` to represent a tuple of any length whose types are all `type1`


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


As we know, we can put any type of variable in the `list`. How do we add type hints for such a list? The solution is `Any` type:
```python
list[Any]
```

We can just use `list`, which is less verbose.

> 📒 If you want to know more collections types, please refer to [`collections.abc`](https://docs.python.org/3/library/collections.abc.html)

### Type alias

Sometimes, the type will be so complicated that we don't want to write it everywhere. So what do we do? Well, we can give it an alias with a meaningful name. The syntax is simple:
```python
AliasName = Type
```

Take the previously defined `date` type as an example. A `date` is a `tuple` containing three `int` types. The type `list[tuple[int, int, int]` would denote a list of dates. To make the code more readable, it can be beneficial to give this type an alias, such as `Date`. See the following example:


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


The syntax of type alias is quite similar to defining a global variable. To make it more explicit, PEP 163(Python 3.10+) proposes a better way[^7]:

```python
AliasName: TypeAlias = Type
```


```python
from typing import TypeAlias

Date: TypeAlias = tuple[int, int, int]
```

### Parameterized generic types

In other programming languages, usually, they use an uppercase letter like `T` to denote a parameterized generic type. In python, we use `TypeVar` to do same thing. As the docs say:
```python
T = TypeVar('T')            # Can be anything
S = TypeVar('S', bound=str) # Can be any subtype of str
A = TypeVar('A', str, bytes)# Must be exactly str or bytes
```

To summarize, `TypeVar` provides two ways for us to restrict the generic types:

1. use `bound=some_type`, then we can only pass the *subtype* of `some_type`.
2. specify the allowed types directly


> 📒 The definition of *subtype* is in PEP 483[^4]. In general: each type is its own subtype; In Object-oriented programming, the subclass is the subtype of its superclass


```python
from typing import TypeVar

GenericString = TypeVar('GenericString', str, bytes)

def process(s: GenericString):
    """ The GenericString can be either str or bytes"""
    ...
```

> 📒 The `typing` module already provides us a `AnyStr` type to represent either `str` or `bytes`

### Optional and Union

`Optional[type1]` represents a type that can be either `type1` or `None`.

The `Union[type1, type2, ...]` means that the allowed type is one of the types we specified, which is the logical *or* relationship. So we can know that the `Optional[type1]` is equal to `Union[type1, None]`.


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



In Python 3.10+, it introduces `|` to replace the `Union`. Some other programming languages also use `|`. *For example, Rust use `|` to seperate multiple possible patterns*. The `parse` function we defined earlier can be written in the following form: 


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



### Callables

We use the function as an example of *callable*. In Python, a function is a first-class object, which means that it can be a parameter or a return value of another function. The type hints to support this:
```python
Callable[[ParamType1, ParamType2], ReturnType]
```

*Let's define a `apply` function which can apply a function on the data*


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

### Class

In Python 3.11[^9], the `Self` type is proposed, which represents an instance of the current class. We can use `Self` within the class definition everywhere :) ~~No need to use `TypeVar` anymore~~


```python
from typing import Self

class Shape:
    def set_scale(self, scale: float) -> Self:
        self.scale = scale
        return self
```

> 💡 Rust also uses `Self` to denote the current object, which usually appears in an impl block.

## Summary

Well, that's the entire content of this blog. I have covered some of the most common and practical uses of type hints that I have found useful. However, it is not an exhaustive guide, and there are more advanced features such as static protocols[^8] that readers can explore.

Personally, I follow the strategy of using type hints only when the added benefits outweigh the potential added complexity. **In situations where the type hints become too complex, I choose not to use them unless it is deemed necessary.**


I will also give you some advice based on my experiences🎯:
1. **When you decide what to add, try to think what this type it can do**[^8]. The static protocols[^8] are well suited to this requirement. This makes me think of the traits in Rust :) 
2. **Always make the return type as precise as possible**. Just try to imagine that a function returns `Union[str, int]`. We have to check it manually, which makes me fell like the type hints are a little unnecessary.
3. **Even if you pass the type checker, you are not free of bugs**. Software testing is the standard practice to make your programs work as expected.
4. Add the [cheatsheet](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html) to your bookmark 👍

## Refs

[^1]: [PEP 3107. Function Annotations](https://peps.python.org/pep-3107/)

[^2]: [PEP 484. Type Hints](https://peps.python.org/pep-0484/)

[^3]: [PEP 526. Syntax for Variable Annotations](https://peps.python.org/pep-0526/)

[^4]: [PEP 483. The Theory of Type Hints](https://peps.python.org/pep-0483/)

[^5]: [PEP 604. Allow writing union types as X | Y](https://peps.python.org/pep-0604/)

[^6]: [PEP 585. Type Hinting Generics In Standard Collections](https://peps.python.org/pep-0585/)

[^7]: [PEP 613. Explicit Type Aliases](https://peps.python.org/pep-0613/)

[^8]: [PEP 544. Protocols: Structural subtyping (static duck typing)](https://peps.python.org/pep-0544/)

[^9]: [PEP 673. Self Type](https://peps.python.org/pep-0673/)

