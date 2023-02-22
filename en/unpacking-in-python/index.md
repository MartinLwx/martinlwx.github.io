# Unpacking in Python 3.5


> This post is originally written in jupyter notebook and then convert to markdown. To get the original notebook files. Please check the [repo](https://github.com/MartinLwx/oh_my_python)
## Intro

Today I want to talk about the unpacking operators(`*` and `**`) in python.

## Basic usage

We use `*` for numeric data types to indicate we want to do multiplication. However, we can also apply `*` to **iterable objects**[^1], which means we want to unpack all the elements inside them.

> 📒The built-in iterable objects: `list`, `tuple`, `set`, and `dict`

### Starred assignment/expression

In the release of python 3.0, it is shipped with powerful iterable unpacking operations[^3], which is called the starred assignment/expression(or parallel assignment).

We are allowed to specify a *catch-all* name in the LHS(i.e. Left Hand Side) of `=` to catch values in the RHS(i.e. Right Hand Side). The example says more:

```python
>>> first, *rest, last = [1, 2, 3, 4, 5]
>>> first
1
>>> rest
[2, 3, 4]
>>> last 
5
```

> 📒 The syntax is quite simple: a variable name follows a star - `*foo`. we can put it anywhere in the LHS of `=` to catch items, but **only once**. Also, the type of `foo` will be `list`

In my opinion, this feature makes python code **more human-readable**👍

There are some restrictions though:
- We can't just use a single `*foo` in the LHS of `=` as a long assignment target. **The LHS must be in a list or tuple**
- It would be an error if the RHS of `=` doesn't have enough items to unpack

To demonstrate the restrictions:


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


> 📒 **Usually, we will combine `*` and `_`(i.e. `*_`) to indicate we don't care about the items it caught**


```python
first, *_ = [1, 2, 3]
first
```




    1



### More power

Start from Python 3.5, we can use `*` and `**` in more circumstances.[^2]

**Case 1**. we are allowed to use them as many times as we want inside function calls


```python
foo, bar = {'a': 1, 'b': 2}, {'c': 3, 'd': 4}
dict(**foo, **bar) # dict is a function
```




    {'a': 1, 'b': 2, 'c': 3, 'd': 4}



> 📒The keys in a dictionary remain in a right-to-left priority order[^2]. i.e. The later values will always override the earlier ones. *See the following example*:


```python
{**{'a': 1, 'b': 2}, **{'a': 3}}
```




    {'a': 3, 'b': 2}



> 📒When we use multiple `**` in function calls. We need to **make sure they have no duplicate keys.**


```python
dict(**{'a': 1, 'b': 2}, **{'a': 3})
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In [7], line 1
    ----> 1 dict(**{'a': 1, 'b': 2}, **{'a': 3})


    TypeError: dict() got multiple values for keyword argument 'a'


**Case 2.** We **can** use them in tuple/list/set/dict **literals**. But we **can't** use them inside the list/set/dict comprehensions😺


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


      Cell In [9], line 5
        [*sublist for sublist in matrix]
         ^
    SyntaxError: iterable unpacking cannot be used in comprehension



## Wrap up

The unpacking feature in python makes life easier. It's **an intuitive way to *destructure* the iterable object**. With the help of this operator, we can avoid some silly indexError🙅

## Refs

[^1]: [Python iterators](https://www.w3schools.com/python/python_iterators.asp)

[^2]: [PEP 448. Additional Unpacking Generalizations](https://peps.python.org/pep-0448/)

[^3]: [PEP 3132. Extended Iterable Unpacking](https://peps.python.org/pep-3132/)


