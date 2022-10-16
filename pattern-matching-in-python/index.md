# Pattern Matching in Python


> This post is originally written in jupyter notebook and then convert to markdown. To get the original notebook files. Please check the [repo](https://github.com/MartinLwx/oh_my_python)


## Intro

Today I want to talk about the new feature bring in Python 3.10 -- Pattern matching 🎉

Those who have learned C language must be familiar with the following `switch` statement:

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

To recap, the syntax rules of the `switch` statement:
1. `expression` should be the `int` or `char` type. `constant` should be the `int` or `char` constant
2. The execution process of the `switch` statement: Calculate the value of `expression`, and compare this value with each `constant` **from top to bottom**. If they are equal, the statements **inside the specific `case` and all the `case` after the matched `case`** will be executed, unless it finds a `break` statement. This feature is called *Fall through*, and we can use this feature to stack multiple `case` statements together to represent a logical "or" relationship
3. The `default` branch will be executed when all the previous `case` branches fail to match

Python does not provide a `switch` statement, but we can use `if...elif..elif..else` to achieve the same effect, *for example, suppose we want to perform different operations depending on the length of the `list`, We can write this*:


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

The above series of `if...elif..elif..else` is actually slightly less readable, and it also violates the DRY(Don't repeat yourself) principle, we write `len(some_list)` many times.

Of course, we can choose to use a variable `length` to remember the length of `some_list` first, so that we can type less code. However, if the situation is more complicated, this trick is not applicable.

A more elegant way to handle this situation is what this article is about: **Pattern matching** ⬇️


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

## Syntax rules

The syntax of the pattern matching is:

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

Syntactically, it is similar to the aforementioned `switch` statement in C. The differences are:
1. There is no *fall through* in pattern matching. **Only the code inside the matched `case` branch** will be executed. So we don't need to add a `break` statement at the end of each `case` block
2. No `default` available. But we can use `case _` to capture all cases, which is called the *Wildcard pattern* that will be discussed later
3. The `subject` and `pattern` here are much more powerful than the C language, **not only the integer and char types**, but `pattern` can also be **combined and nested** with each other. It will be explained in detail later

## Patterns

In pattern matching, a pattern basically does two things:
1. Structure constraints: we can add constraints to the `subject` in various ways. *For example the length, the specific value in a specific position, etc*
2. Bind variables: we can bind some names in the pattern to component elements of the subject. See the *Capture pattern* below.

Below I discuss different patterns :)

To avoid confusion, it's worth explaining in advance that both `()` and `[]` are optional when pattern matching matches a sequence. *For example, `case foo, bar` and `case (foo, bar)` and `case [foo, bar]` are equivalent*. They have the same meaning.

### Capture pattern

*Capture pattern* means that when we check whether a `pattern` matches, we can use **variable names to bind to any part of it**, and we can **use these variables later**


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


People who often deal with sequences must be familiar with the following code:


```python
*before, last = [1, 2, 3, 4]              
assert last == 4, "Error"

first, *middle, last = [1, 2, 3, 4]
assert first == 1 and last == 4, "Error"

first, *rest = [1, 2, 3, 4]
assert first == 1, "Error"
```

Similarly, we can do this in pattern matching:


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

*Literal pattern* means that we can **specify literal values in specific positions**. The literals here can be number literals, string literals, `True`, `False`, and `None`

> 📒 Note: For number literals and string literals, python will use `==` for comparison, and for `True/False/None` these three will use `is`.


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

It is a common practice in python that: use `_` to indicate that we don't care about the value. Similarly, `_` can be used here, and it **will not bind any variables**


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


Another common usage is the `case _` that appeared before. The `_` will match everything, so `case _` is often put at the end to indicate the default case


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

Just like in the `if` statement we can use `or`, in pattern matching we also have a similar syntax. Like most other languages, Python chooses to use `|` to express an "or" logical relationship. We can declare the alternatives conveniently


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


The disadvantage of the Or pattern above is that we have no way of knowing which one we matched exactly

### As pattern

In the previous example, we may match any of the alternatives, so how do we know which one we match? Because we **may need to decide what to do based on which one is matched**. In pattern matching, we can use `as` to bind a value


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

Python is a dynamically typed language, and sometimes we need to **decide whether to match based on the type**. The intuitive way is using `isinstance()` to check if it is qualified, but there is a better way. Next, I will show you how to add type constraints step by step:


```python
some_list = ["foo", 1, 3.14]

match some_list:
    # match without type constraints
    case [s, v1, v2]:
        if isinstance(s, str) and isinstance(v1, int) and isinstance(v2, float):
            print(f'Match {s} - {v1} - {v2}')
```

    Match foo - 1 - 3.14


The intuitive way: we use the *Capture pattern* to bind values, and use `isinstance` to check the type in the code block


```python
some_list = ["foo", 1, 3.14]

match some_list:
    # match with type constraints
    case [str() as s, int() as v1, float() as v2]:
        print(f'Match {s} - {v1} - {v2}')
```

    Match foo - 1 - 3.14


The syntax here is quite similar to the previous *Literal pattern* + *As pattern*. i.e. We declare the type we want to match in the corresponding position. At the same time, we use the `as` keyword to bind the value. 

Less typing but still less elegant :(. Fortunately, python provides us with syntactic sugar 🍬


```python
some_list = ["foo", 1, 3.14]

match some_list:
    # match with type constraints
    case [str(s), int(v1), float(v2)]:
        print(f'Match {s} - {v1} - {v2}')
```

    Match foo - 1 - 3.14


### Mapping pattern

The previous ones are all matching against a sequence, here we are matching against the `dict`. I believe that after reading the previous examples, it is not difficult to understand the following example. But there are a few things to keep in mind:
1. we match `dict` based on its present keys. The keys here must be a literal value or a value of enum type (for performance considerations), values do not have this restriction
2. we can use `**<name>` to capture the key-value pair we didn't write in `pattern`. Otherwise **the undeclared key-value pairs will be ignored**


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

It's good practice to use "named variables" as parameter values or to clarify the meaning of specific values. *e.g. `case (HttpStatus.OK, body)` is better than `case (200, body)`*

The challenge of implementing the *Value pattern* in Python is to distinguish it from the previous *Capture pattern*. A discussion of this can be found at [^2]

The final solution provided by python is a restricted *Value pattern* that only supports *Value pattern* of the form `foo.bar`. Common usage will be combined with the enum type, see the following example


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


## Use pattern matching on a class

If we can only use pattern matching on built-in types, it doesn't seem to be that useful. But in fact, Python also allows us to use pattern matching on objects of our own custom classes.

Considering the application scenario, when we use pattern matching on an object, we often want to check whether the object is a certain class type. We may also care about some of its fields and want to extract the values. 

But in Python, this is difficult to implement[^2], mainly because the class has a lot of fields, most of which are special methods like `__repr__`, and these **fields are unordered**. Because it is unordered, we **can't directly bind variables by position** in `pattern`, see the following example:


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


Python provides two solutions, which are syntactically very similar to calling a function: we can choose to pass arguments by position, or we can choose to use the form `foo=bar`

Let's talk about the simple one first: add constraints by using `foo=bar`, which means that the object should have a field called `foo` and we want to bind `bar` to it.


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


Another solution is to modify the `__match_args__` attribute of the class, which **specifies the order of the fields**


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


if you are familiar with `@dataclass`[^3], you can do this:


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

Sometimes we not only care about whether the pattern matches, but we also want to impose certain restrictions.

Consider such a situation where we want to match a sequence containing two `int` values, but we require the first element shoudld be larger than the second. How would you write it? Combined with the previous *Class pattern*, it is not difficult for us to write the following code:


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


The above solution is fine, we definitely can use the `if` statement in the code block to add constraints. But just like type constraints, Python has taken this requirement into account, so it provides a **Guard 💂‍♀️ mechanism**, which allows us to move the `if` statement to the end of the `pattern` for readability. 

The syntax rules to follow:

````python
match subject:
    case <pattern> if <expression>:
        ...
````

1. `<pattern>` followed by an `if` statement
2. Note how python evaluates this here:
    1. Check if `<pattern>` matches
    2. If it matches, bind values if we declared
    3. Check if the `if <expression>` statement returns `True`. The `<expression>` here can use the previously bound variable in step 2
3. The code inside the code block will be executed if and only if `<pattern>` matches + `if` statement returns `True`. Otherwise, it will fail and try to match the next `<pattern>`


```python
some_list = [3, 4]

match some_list:
    case [int(first), int(second)] if first > second:
        print("Match successfully!")
```

## Wrap up

I would prefer pattern matching to the `if...elif...elif...else` for several reasons:
1. We can easily bind values for subsequent processing when matching
2. For better readabilityg
3. The various patterns of pattern matching can actually be nested and combined, that's what makes pattern matching a shining point.

The above is a brief introduction to pattern matching introduced in Python 3.10 🚀

## 参考

[^1]: [What's new in Python 3.10](https://docs.python.org/3/whatsnew/3.10.html)

[^2]: [PEP 635 – Structural Pattern Matching: Motivation and Rationale](https://peps.python.org/pep-0635/)

[^3]: [dataclasses - documentation](https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass)


