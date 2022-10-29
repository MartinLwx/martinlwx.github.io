# Walrus Operator in Python3.8


> This post is originally written in jupyter notebook and then convert to markdown. To get the original notebook files. Please check the [repo](https://github.com/MartinLwx/oh_my_python)


## Intro

Today I’m going to talk about a new feature introduced in Python 3.8: the Walrus operator（`:=`）, which is a much-debated feature, but it’s finally passed and released 🤔

In Python, an assignment statement (`=`) is not an expression but a statement. Walrus operator is expression though. The difference between statement and expression can be simply understood as: expression always returns a value, while statement does not return a value.

The difference between the two can be seen in the following code:


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



> 📒 The `()` needs to be added to avoid confusion. According to the [Zen of Python](https://peps.python.org/pep-0020/): "There should be one-- and preferably only one --obvious way to do it.". If we can use it without `()`, we definitely will be confused about which one to use 🤕️

In C/C++, `=` is an expression. We can write this:

```c
// = will store the value in the LHS(left-hand-side) variable.
// , and it has the value of the LHS
// so we store the result of `foo` function call to `a`
// , then we check if `a` > 0
while ( (a = foo(...)) > 0 ) {
    ...
}
```

Before Python 3.8, it was not possible to do something similar. Because the `=` is a statement. This is where the walrus expression comes into play 🤩

## Syntax rules

- The syntax of walrus expressions is quite simple: `NAME := EXPRESSION`. `:=` assigns the value of `EXPRESSION` to `NAME`.
    - we are not allowed to use attributes or subscripts as `NAME`. See the code below


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



- The scope of `NAME`
    - Walrus expressions do not introduce new scopes 🤩
    - `NAME` can be used in the current scope, with one exception: if it is used inside a list/dict/set comprehension. We can use `NAME` in the enclosing scope. See the following example:


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

- Order of evaluation: lower than others except the comma(`,`)


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



## Usage

> 📒 Combined with the content of [^1] and my personal programming experience. The most convenient thing about walrus expressions is: dealing with functions whose return value may be `None`

To deal with functions that may return `None`, we will usually **first use the `=` statement to save the return result of the function**. Then we check if it is `None` so we can refer to it later safely
```python
some_thing = foo(....)
if some_thing:
    ...
else:
    ...
```

*I will use the `re` as an example*


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


To avoid getting an `AttributeError: 'NoneType' object has no attribute 'group'` error when using `match.group()` later, we have to use an intermediate variable `match` to temporarily hold the return value rather then chain them together: `re.search(DIGIT_PATTERN, text).group(0)`. It may seem kind of redundant though.

> 📒 In Rust, the return type that may return `None` is called `Option<T>`. We can use `?` to handle this situation, it will try to extract the value inside, and if it fails it will terminate early with an error. So in Rust, we can do this (assuming Rust has a similar API): `re.search(DIGIT_PATTERN, text)?.group(0)` 🍺

The readability is slightly weaker in my opinion, of course, this is quite personal. i.e. We can only use the `match` variable when `match is not None`. There is no chance to use it inside the `else` branch. However, if we look at the code from the top down quickly, `match = ...` on a separate line is like the `match` can be used later everywhere 🤣

Instead, **we can use `:=` to bind its return value** here:


```python
if match := re.search(DIGIT_PATTERN, text):
    # group(0) will return the entire match
    print(f"Find match: {match.group(0)}")   
else:
    print("Not match was found")
```

    Find match: 10


[^1] One of the reasons for supporting `:=` is that research shows that developers tend to write fewer lines of code rather than shorter code. That's what we did here. At the same time, we can see the scope of `match` at a glance 👏


Similarly, we can use this feature in `while` loops too:

```python
val = foo(...)
while val:
    # do something while val is not None
    val = foo(...)
```

## 🆚 `=`

A few differences worth mentioning:
1. `=` is a statement, `:=` is an expression. This also determines their application scenarios.
2. Only `=` supports the continuous use of `foo = bar = 1`; and the left side of `=` can be an attribute like `foo.bar`, or an subscript like `foo[1]`, but `:=` can only be a simple variable name on the left
3. `=` supports the augmented form of `+=`, but `:=` does not

## Wrap up

In my opinion, the walrus operator is quite useful in the aforementioned scenarios (Recommended 👍), and the readability is much improved 🚀. But some examples in [^1] just make me more confused

## 参考

[^1]: [PEP 572. Assignment Expressions]


