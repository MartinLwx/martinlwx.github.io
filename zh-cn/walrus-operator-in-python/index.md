# 海象表达式简明教程（Python 3.8）


> 这一篇博客一开始是写在 jupyter notebook 里面而后转成 markdown 的。如果想要访问和运行本来的 notebook，请查阅这个[仓库](https://github.com/MartinLwx/oh_my_python)

## 引言

今天要说的是在 Python3.8 中引入的新特性：海象运算符（Walrus operator），这是一个备受争议的特性，但它最后还是通过并发布了🤔

在 Python 中，赋值语句（`=`）并不是 expression 而是 statement。海象表达式则是 expression。关于 statement 和 expression 的区别可以简单理解为：expression 总是会返回值，而 statement 不返回值。

两者的区别可以看下面的代码：


```python
# `=` is a statement, so it will print no value
x = 5   
```


```python
# `:=` is an expression, so it will evaluate to a value
# not recommended :)
(x := 5) 
```




    5



> 📒 这里海象表达式需要加上 `()` 是为了避免造成混淆。毕竟在 [Python设计哲学-Zen](https://peps.python.org/pep-0020/) 里面提到了一条："There should be one-- and preferably only one --obvious way to do it."。如果不加 `()` 就能使用的话，开发者就会陷入到该用哪一个的困惑之中🤕️

在 C/C++ 中，`=` 是 expression。如果你有相关的编程背景的话，你可能对下面的代码不陌生

```c
// = will store the value in the LHS(left-hand-side) variable.
// , and it has the value of the LHS
// so we store the result of `foo` function call to `a`
// , then we check if `a` > 0
while ( (a = foo(...)) > 0 ) {
    ...
}
```

但是在 Python 3.8 之前是无法做到类似的事情的，因为 `=` 在 Python 里面是一个 statement，并不会返回值。这就是海象表达式发挥作用的地方 🤩

## 语法规则

- 海象表达式的语法很简单：`NAME := EXPRESSION`。`:=` 左边放变量名，右边则是表达式。表达式的值会被绑定到 `NAME` 上。
    - 这里的 `NAME` 不能是属性或者索引


```python
# a dummy example. we bind `1 + 2 + 3` to `res` for future usage
if (res := 1 + 2 + 3) > 5:
    print(f"res is {res}")
```

    res is 6



```python
class foo:
    val: int = 0

some_foo = foo()

(some_foo.val := 1)
```


      Input In [4]
        (some_foo.val := 1)
         ^
    SyntaxError: cannot use assignment expressions with attribute




```python
x = [1, 2, 3]

(x[1] := 3)
```


      Input In [5]
        (x[1] := 3)
         ^
    SyntaxError: cannot use assignment expressions with subscript



- 关于 `NAME` 的作用域
    - 海象表达式不会引入新的作用域 🤩
    - `NAME` 可以在当前的作用域使用，同时有个例外：如果是在 list/dict/set comprehension 里面使用。则是是在 enclosing scope 里面. 可以看下面的例子：


```python
s = [1, 2, 3]
# the list comprehension forms a new scope. 
# its enclosing scope is the global scope
double_s = [item * 2 for item in s]

# `item` is not in the global scope :)
print(item)
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    Input In [6], in <cell line: 7>()
          4 double_s = [item * 2 for item in s]
          6 # `item` is not in the global scope :)
    ----> 7 print(item)


    NameError: name 'item' is not defined



```python
s = [1, 2, 3]
# the list comprehension forms a new scope. 
# its enclosing scope is the global scope
double_s = [last := item * 2 for item in s]

# so we can use `last` variable here
print(last)
```

- 优先级规则：除了比 `,` 高，比其他的都低


```python
x = 1, 2
x
```




    (1, 2)




```python
(x := 1, 2)
x
```




    1



## 适用场景

> 📒 结合参考了[^1]的内容和平常的编程经验。感觉海象表达式最方便的还是：处理返回值可能为 `None` 的函数

对于返回值可能为 `None` 的函数，以前经常是**先用 `=` 赋值语句保存函数的返回结果**。不为 `None` 的时候要使用返回值来进行下一步处理。此时代码看起来大概是这样：
```python
some_thing = foo(....)
if some_thing:
    ...
else:
    ...
```

*下面用 `re` 库举例子*


```python
import re

# define a regex pattern to extract digits in a string
DIGIT_PATTERN = r'\d+'  

text = 'There are 10 dogs'

# re.search will return None if no match was found.
match = re.search(DIGIT_PATTERN, text)
if match:
    # group(0) will return the entire match
    print(f"Find match: {match.group(0)}")   
else:
    print("Not match was found")
```

    Find match: 10


为了能在后面使用 `match.group()` 的时候 Python 解释器不会返回 `AttributeError: 'NoneType' object has no attribute 'group'` 错误，我们不得不用一个中间变量 `match` 暂时保存 `re.search` 的返回值对其进行检查，显得似乎有点冗余。我们无法直接将他们连起来：`re.search(DIGIT_PATTERN, text).group(0)`

> 📒 插点题外话：在 Rust 里面，可能返回 `None` 的返回类型是 `Option<T>`。我们可以使用 `?` 来处理这种情况，它会尝试提取里面的值，如果失败了就会尽早终止报错。所以在 Rust 里面，我们可以这样（假设 Rust 也有类似的 API）：`re.search(DIGIT_PATTERN, text)?.group(0)` 🍺

另外一点是：可读性稍弱，当然这是个人的主观感受。有一点不舒服是：只有可能在 `match is not None` 的时候我们才会使用 `match` 变量，我们不会在 `else` 分支里面使用。但是如果很快地从上往下看代码的话，单独另起一行的 `match = ...` 就像是后面都可以使用一样🤣

此时我们**可以选择用 `:=` 来绑定其返回值**


```python
if match := re.search(DIGIT_PATTERN, text):
    # group(0) will return the entire match
    print(f"Find match: {match.group(0)}")   
else:
    print("Not match was found")
```

    Find match: 10


[^1] 里面提到支持 `:=` 的一个理由是：调查显示，开发者往往喜欢写比较少行的代码而不是让代码更短。这里我们就少写了一行代码。同时看一眼就知道 `match` 的作用域👏


类似的，在 `while` 循环中我们也可以借用这个特性

```python
val = foo(...)
while val:
    # do something while val is not None
    val = foo(...)
```

## 🆚 `=`

几个值得一提的不同：
1. `=` 是 statement，`:=` 是 expression。这也决定了他们的适用场景是不同的
2. 只有 `=` 支持`foo = bar = 1` 这种连续使用的情况；而且 `=` 左边可以是 `foo.bar` 这种属性，或者是 `foo[1]` 这种索引的形式，但是 `:=` 左边只能是一个简单的变量名
3. `=` 支持 `+=` 这种 augmented 的形式，但是 `:=` 不行

## 总结

在我看来，海象表达式在我前面提到的引用场景中是挺有用的（推荐使用👍），可读性提高不少🚀。但是[^1]中的某些例子我觉得可能是负优化了👀

## 参考

[^1]: [PEP 572. Assignment Expressions]

