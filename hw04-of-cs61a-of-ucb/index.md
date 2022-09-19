# Hw04 of CS61A of UCB(2021-Fall)


## Mobiles

---

### Q2: Weights

>   Implement the `planet` data abstraction by completing the `planet` constructor and the `size` selector so that a planet is represented using a two-element list where the first element is the string `'planet'` and the second element is its size.

From the description, we can know what is `planet`. It is a `['planet', size]`. Then we can take a look at the `mobile` function, the solution is quite similar.

```python
def planet(size):
    """Construct a planet of some size."""
    assert size > 0
    return ['planet', size]


def size(w):
    """Select the size of a planet."""
    assert is_planet(w), 'must call size on a planet'
    return w[1]
```

### Q3: Balanced

>   Implement the `balanced` function, which returns whether `m` is a balanced mobile. A mobile is balanced if both of the following conditions are met:
>
>   1.  The torque applied by its left arm is equal to that applied by its right arm. The torque of the left arm is the length of the left rod multiplied by the total weight hanging from that rod. Likewise for the right. For example, if the left arm has a length of `5`, and there is a `mobile` hanging at the end of the left arm of weight `10`, the torque on the left side of our mobile is `50`.
>   2.  Each of the mobiles hanging at the end of its arms is balanced.
>
>   Planets themselves are balanced, as there is nothing hanging off of them.

This problem is a little tricky. We need to solve this problem recursively. We can call a mobile is balanced if:

1.   It is a `planet`. THE BASE CASE!!
2.   It is a `arm` and the `total_weight(left_arm) == total_weight(right_arm)`. **And, the sub-mobile itself should be balanced**



The key: distinguish `arm`, `planet`, `mobile` these 3 concepts. :cry:

```python
def balanced(m):
    """Return whether m is balanced.

    >>> t, u, v = examples()
    >>> balanced(t)
    True
    >>> balanced(v)
    True
    >>> w = mobile(arm(3, t), arm(2, u))
    >>> balanced(w)
    False
    >>> balanced(mobile(arm(1, v), arm(1, w)))
    False
    >>> balanced(mobile(arm(1, w), arm(1, v)))
    False
    >>> from construct_check import check
    >>> # checking for abstraction barrier violations by banning indexing
    >>> check(HW_SOURCE_FILE, 'balanced', ['Index'])
    True
    """
    # planet is balanced 
    if is_planet(m):
        return True

    left_arm, right_arm = left(m), right(m)
    
    # end(...arm) will is a mobile or a planet
    left_val = length(left_arm) * total_weight(end(left_arm))
    right_val = length(right_arm) * total_weight(end(right_arm))
    return left_val == right_val and balanced(end(left_arm)) and balanced(end(right_arm))
```

### Q4: Totals

>   Implement `totals_tree`, which takes in a `mobile` or `planet` and returns a `tree` whose root is the total weight of the input. For a `planet`, `totals_tree` should return a leaf. For a `mobile`, `totals_tree` should return a tree whose label is that `mobile`'s total weight, and whose branches are `totals_tree`s for the ends of its arms. As a reminder, the description of the tree data abstraction can be found [here](http://composingprograms.com/pages/23-sequences.html#trees).

What we want to do is convert the `mobile` into a `tree`. The base case is to make a leaf when we encounter the `planet`, otherwise, we make a sub-tree instead.

```python
def totals_tree(m):
    """Return a tree representing the mobile with its total weight at the root.

    >>> t, u, v = examples()
    >>> print_tree(totals_tree(t))
    3
      2
      1
    >>> print_tree(totals_tree(u))
    6
      1
      5
        3
        2
    >>> print_tree(totals_tree(v))
    9
      3
        2
        1
      6
        1
        5
          3
          2
    >>> from construct_check import check
    >>> # checking for abstraction barrier violations by banning indexing
    >>> check(HW_SOURCE_FILE, 'totals_tree', ['Index'])
    True
    """
    if is_planet(m):
        return tree(size(m))

    return tree(total_weight(m), [totals_tree(i) for i in [end(left(m)), end(right(m))]])
```

## More trees

---

### Q5: Replace Loki at Leaf

>   Define `replace_loki_at_leaf`, which takes a tree `t` and a value `lokis_replacement`. `replace_loki_at_leaf` returns a new tree that's the same as `t` except that every leaf label equal to `"loki"` has been replaced with `lokis_replacement`.
>
>   If you want to learn about the Norse mythology referenced in this problem, you can read about it [here](https://en.wikipedia.org/wiki/Yggdrasil).

The base case is the `leaf` node. We need to check if its label is equal to 'loki'. If it is the truth, we will make a new leaf, otherwise, we will just return the original leaf.

```python
def replace_loki_at_leaf(t, lokis_replacement):
    """Returns a new tree where every leaf value equal to "loki" has
    been replaced with lokis_replacement.

    >>> yggdrasil = tree('odin',
    ...                  [tree('balder',
    ...                        [tree('loki'),
    ...                         tree('freya')]),
    ...                   tree('frigg',
    ...                        [tree('loki')]),
    ...                   tree('loki',
    ...                        [tree('sif'),
    ...                         tree('loki')]),
    ...                   tree('loki')])
    >>> laerad = copy_tree(yggdrasil) # copy yggdrasil for testing purposes
    >>> print_tree(replace_loki_at_leaf(yggdrasil, 'freya'))
    odin
      balder
        freya
        freya
      frigg
        freya
      loki
        sif
        freya
      freya
    >>> laerad == yggdrasil # Make sure original tree is unmodified
    True
    """
    if is_leaf(t):
        if label(t) == 'loki':
            return tree(lokis_replacement)
        return t
    return tree(label(t), [replace_loki_at_leaf(b, lokis_replacement) for b in branches(t)])
```

### Q6: Has Path

>   Write a function `has_path` that takes in a tree `t` and a string `word`. It returns `True` if there is a path that starts from the root where the entries along the path spell out the `word`, and `False` otherwise. (This data structure is called a trie, and it has a lot of cool applications, such as autocomplete). You may assume that every node's `label`is exactly one character.

1.   What is the base case ? We need to memorize the current word along the path. For example, we got `label(t)` when we start from the root of the tree(`h`). When we pay a visit to the `tree('i')`(the first branch), the current word will be `h` + `i` = `hi`. So we need to define a `helper` function. The base case is either the current word is equal to the target(`word`) or the current word is not in target(`word`), which means we don't need to go deeper into this tree
2.   How to break down the current problem into simpler ones ? We use `any` in python to check if any of the branches contain the word we want.

```python
def has_path(t, word):
    """Return whether there is a path in a tree where the entries along the path
    spell out a particular word.

    >>> greetings = tree('h', [tree('i'),
    ...                        tree('e', [tree('l', [tree('l', [tree('o')])]),
    ...                                   tree('y')])])
    >>> print_tree(greetings)
    h
      i
      e
        l
          l
            o
        y
    >>> has_path(greetings, 'h')
    True
    >>> has_path(greetings, 'i')
    False
    >>> has_path(greetings, 'hi')
    True
    >>> has_path(greetings, 'hello')
    True
    >>> has_path(greetings, 'hey')
    True
    >>> has_path(greetings, 'bye')
    False
    >>> has_path(greetings, 'hint')
    False
    """
    assert len(word) > 0, 'no path for empty word.'
    def helper(t, cur):
        if cur == word:
            return True
        if cur not in word:
            return False
        return any([helper(b, cur + label(b)) for b in branches(t)])
    return helper(t, label(t))
```

## Trees

---

### Q7: Preorder

>   Define the function `preorder`, which takes in a tree as an argument and returns a list of all the entries in the tree in the order that `print_tree` would print them.
>
>   The following diagram shows the order that the nodes would get printed, with the arrows representing function calls.

The preorder function is one of the basic methods when we want to visit every node in a tree. We need to memorize the label of the *root* before we visit its sub-tree.

```python
def preorder(t):
    """Return a list of the entries in this tree in the order that they
    would be visited by a preorder traversal (see problem description).

    >>> numbers = tree(1, [tree(2), tree(3, [tree(4), tree(5)]), tree(6, [tree(7)])])
    >>> preorder(numbers)
    [1, 2, 3, 4, 5, 6, 7]
    >>> preorder(tree(2, [tree(4, [tree(6)])]))
    [2, 4, 6]
    """
    result = []
    def helper(t):
        if t is not None:
            result.append(label(t))
            for b in branches(t):
                helper(b)
    helper(t)
    return result
```

## Data Abstraction

---

### Q8: Interval Abstraction

>   **Acknowledgements.** This interval arithmetic example is based on a classic problem from Structure and Interpretation of Computer Programs, [Section 2.1.4](https://sarabander.github.io/sicp/html/2_002e1.xhtml#g_t2_002e1_002e4).
>
>   **Introduction.** Alyssa P. Hacker is designing a system to help people solve engineering problems. One feature she wants to provide in her system is the ability to manipulate inexact quantities (such as measurements from physical devices) with known precision, so that when computations are done with such approximate quantities the results will be numbers of known precision. For example, if a measured quantity x lies between two numbers a and b, Alyssa would like her system to use this range in computations involving x.
>
>   Alyssa's idea is to implement interval arithmetic as a set of arithmetic operations for combining "intervals" (objects that represent the range of possible values of an inexact quantity). The result of adding, subracting, multiplying, or dividing two intervals is also an interval, one that represents the range of the result.
>
>   Alyssa suggests the existence of an abstraction called an "interval" that has two endpoints: a lower bound and an upper bound. She also presumes that, given the endpoints of an interval, she can create the interval using data abstraction. Using this constructor and the appropriate selectors, she defines the following operations:
>
>   

>   Alyssa's program is incomplete because she has not specified the implementation of the interval abstraction. She has implemented the constructor for you; fill in the implementation of the selectors.

The calculation system of this question is actually used to calculate the resistance. We know that there will be tolerance in the resistor (such as $\pm5\%$), so the real resistance value should be in an interval. This is the ` interval` indicates. The `interval` is just a `list` with size 2. To get the lower bound and the upper bound. We just need to return `x[0]` or `x[1]`

```python
def interval(a, b):
    """Construct an interval from a to b."""
    assert a <= b, 'Lower bound cannot be greater than upper bound'
    return [a, b]


def lower_bound(x):
    """Return the lower bound of interval x."""
    return x[0]


def upper_bound(x):
    """Return the upper bound of interval x."""
    return x[1]
```

### Q9: Interval Arithmetic

>   After implementing the abstraction, Alyssa decided to implement a few interval arithmetic functions.
>
>   This is her current implementation for **interval multiplication**. Unfortunately there are some data abstraction violations, so your task is to fix this code before [someone sets it on fire](https://youtu.be/7zu30DhJLKU?t=444).
>
>   ```python
>   def mul_interval(x, y):
>       """Return the interval that contains the product of any value in x and any
>       value in y."""
>       p1 = x[0] * y[0]
>       p2 = x[0] * y[1]
>       p3 = x[1] * y[0]
>       p4 = x[1] * y[1]
>       return [min(p1, p2, p3, p4), max(p1, p2, p3, p4)]
>   ```

There are many data abstraction violations in `mul_interval`. We can't use `x[0]` or `x[1]` to get the lower bound and the upper bound. We should use the function `lower_bound()` and the `upper_bound()` instead. In addition, we can't use `[...]` to make an interval. That's a violation too!

```python
def mul_interval(x, y):
    """Return the interval that contains the product of any value in x and any
    value in y."""
    p1 = lower_bound(x) * lower_bound(y)
    p2 = lower_bound(x) * upper_bound(y)
    p3 = upper_bound(x) * lower_bound(y)
    p4 = upper_bound(x) * upper_bound(y)
    return interval(min(p1, p2, p3, p4), max(p1, p2, p3, p4))
```

#### Interval Subtraction

>   Using a similar approach as `mul_interval` and `add_interval`, define a subtraction function for intervals. If you find yourself repeating code, see if you can reuse functions that have already been implemented.

This solution is almost identical to the `mul_interval`. If you are using vim as your editor, you just need to copy the content of `mul_interval` and use visual mode to select `*` then press `r-` to substitute them.

```python
def sub_interval(x, y):
    """Return the interval that contains the difference between any value in x
    and any value in y."""
    p1 = lower_bound(x) - lower_bound(y)
    p2 = lower_bound(x) - upper_bound(y)
    p3 = upper_bound(x) - lower_bound(y)
    p4 = upper_bound(x) - upper_bound(y)
    return interval(min(p1, p2, p3, p4), max(p1, p2, p3, p4))
```

#### Interval Division

>   Alyssa implements division below by multiplying by the reciprocal of `y`. A systems programmer looks over Alyssa's shoulder and comments that it is not clear what it means to divide by an interval that spans zero. Add an `assert`statement to Alyssa's code to ensure that no such interval is used as a divisor:

To avoid the divisor span zero, all we have to do is check if `lower_bound(y) > 0` :hugs: 

```python
def div_interval(x, y):
    """Return the interval that contains the quotient of any value in x divided by
    any value in y. Division is implemented as the multiplication of x by the
    reciprocal of y."""
    assert lower_bound(y) > 0, "AssertionError!"
    reciprocal_y = interval(1 / upper_bound(y), 1 / lower_bound(y))
    return mul_interval(x, reciprocal_y)
```

### Q10: Par Diff

>   After considerable work, Alyssa P. Hacker delivers her finished system. Several years later, after she has forgotten all about it, she gets a frenzied call from an irate user, Lem E. Tweakit. It seems that Lem has noticed that the[formula for parallel resistors](https://en.wikipedia.org/wiki/Series_and_parallel_circuits#Resistors_2) can be written in two algebraically equivalent ways:
>
>   ```
>   par1(r1, r2) = (r1 * r2) / (r1 + r2)
>   ```
>
>   or
>
>   ```
>   par2(r1, r2) = 1 / (1/r1 + 1/r2)
>   ```
>
>   He has written the following two programs, each of which computes the `parallel_resistors` formula differently:
>
>   ```python
>   def par2(r1, r2):
>       one = interval(1, 1)
>       rep_r1 = div_interval(one, r1)
>       rep_r2 = div_interval(one, r2)
>       return div_interval(one, add_interval(rep_r1, rep_r2))
>   ```
>
>   Lem points out that Alyssa's program gives different answers for the two ways of computing. Find two intervals `r1` and `r2` that demonstrate the difference in behavior between `par1` and `par2` when passed into each of the two functions.
>
>   Demonstrate that Lem is right. Investigate the behavior of the system on a variety of arithmetic expressions. Make some intervals `r1` and `r2`, and show that `par1` and `par2` can give different results.

If you want to understand the mechanism why this happens, you may check this [link](http://jots-jottings.blogspot.com/2011/09/sicp-exercise-214-broken-intervals.html)

```python
def check_par():
    """Return two intervals that give different results for parallel resistors.

    >>> r1, r2 = check_par()
    >>> x = par1(r1, r2)
    >>> y = par2(r1, r2)
    >>> lower_bound(x) != lower_bound(y) or upper_bound(x) != upper_bound(y)
    True
    """
    r1 = interval(5, 7)  
    r2 = interval(5, 7)  
    return r1, r2
```


