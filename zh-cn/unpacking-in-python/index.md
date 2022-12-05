# Python 3.5 的解包操作符


> 这一篇博客一开始是写在 jupyter notebook 里面而后转成 markdown 的。如果想要访问和运行本来的 notebook，请查阅这个[仓库](https://github.com/MartinLwx/oh_my_python)

## 引言
今天我想要聊聊 Python 中用于解包（Unpacking）的两个操作符号——`*` 和 `**`

## 基本用法

`*` 最为常见的用法是用来表示乘法。但我们也可以将 `*` 用于任意一个可迭代对象（iterable object）[^1]上，表示我们想要提取里面所有的值

> 📒 Python 自带的可迭代对象包括：`list`, `tuple`, `set`, 和 `dict`

### Starred assignment/expression

随着 Python 3.0 的发布，Python 支持使用 `*` 来解包任意可迭代对象[^3]。这个被称之为 Starred assignment，也叫做 Starred expressions。我也有看到称之为 parallel assignment 的。

我们可以在 `=` 左边声明一个特殊的变量表示捕获*所有变量**。看下面这个直观的例子：

```python
>>> first, *rest, last = [1, 2, 3, 4, 5]
>>> first
1
>>> rest
[2, 3, 4]
>>> last 
5
```

> 📒 语法很简单：一个星号后面紧跟着变量名 - `*foo`。我们可以在 `=` 左边任意位置放置它，但是**只能最多使用一次这样的变量**。以及，`foo` 的类型为 `list`

在我看来，Python 的这个特性很大程度上提高了代码的可读性

不过它也有一些局限：
- 我们不能仅仅在 `=` 左边使用一个 `*foo` 当成唯一的被赋值目标。`=` 左侧必须是一个 list 或者是 tuple
- 如果 `=` 右边的值不够用于解包的话，程序就会报错

下面的例子证明了这两点：


```python
*first = [1, 2, 3]
```


      Cell In [1], line 1
        *first = [1, 2, 3]
        ^
    SyntaxError: starred assignment target must be in a list or tuple




```python
# just add `,` would be fine
# now the LHS is a tuple
*first, = [1, 2, 3]       
first
```




    [1, 2, 3]




```python
first, second, *rest = [1]
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Cell In [3], line 1
    ----> 1 first, second, *rest = [1]


    ValueError: not enough values to unpack (expected at least 2, got 1)


> 📒 常见的一个做法是将 `*` 和 `_` 结合在一起使用（也就是 `*_`），表示我们不关心它捕获的变量


```python
first, *_ = [1, 2, 3]
first
```

### 更进一步

从 Python 3.5 开始，我们可以在更多的情况下使用 `*` 和 `**`[^2]

**情况1⃣️**：在函数调用里面，我们想要使用几次就可以使用几次


```python
foo, bar = {'a': 1, 'b': 2}, {'c': 3, 'd': 4}
dict(**foo, **bar) # dict is a function
```

> 📒`dict` 中的 keys 的优先级是右边大于左边。换句话说，后面出现的 key 的值总是会覆盖前面出现的。*看下面这个例子*


```python
{**{'a': 1, 'b': 2}, **{'a': 3}}
```




    {'a': 3, 'b': 2}



> 📒当我们在函数调用里面使用 `**` 的时候，需要注意个问题：我们**需要确保没有重复的 key**


```python
dict(**{'a': 1, 'b': 2}, **{'a': 3})
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In [5], line 1
    ----> 1 dict(**{'a': 1, 'b': 2}, **{'a': 3})


    TypeError: dict() got multiple values for keyword argument 'a'


**情况2⃣️**：我们可以在 tuple/list/set/dict **字面值**中使用。但是不能在 list/set/dict comprehensions 里面使用😺


```python
# an example drawn from PEP 448
>>> *range(4), 4
(0, 1, 2, 3, 4)
>>> [*range(4), 4]
[0, 1, 2, 3, 4]
>>> {*range(4), 4}
{0, 1, 2, 3, 4}
>>> {'x': 1, **{'y': 2}}
{'x': 1, 'y': 2}
```




    {'x': 1, 'y': 2}




```python
matrix = [
    [1, 2, 3]
    [4, 5, 6]
]
[*sublist for sublist in matrix]
```


      Cell In [7], line 5
        [*sublist for sublist in matrix]
         ^
    SyntaxError: iterable unpacking cannot be used in comprehension



## 总结

Python 的解包操作符让编程轻松很多。它提供了一种**直观的解构可迭代对象的手段**。在这个操作符的帮助下，我们可以避免一些愚蠢的索引错误🙅

## 参考

[^1]: [Python iterators](https://www.w3schools.com/python/python_iterators.asp)

[^2]: [PEP 448. Additional Unpacking Generalizations](https://peps.python.org/pep-0448/)

[^3]: [PEP 3132. Extended Iterable Unpacking](https://peps.python.org/pep-3132/)


