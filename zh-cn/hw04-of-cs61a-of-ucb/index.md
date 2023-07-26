# Hw04 - CS61A of UCB(2021-Fall)


## Mobiles

---

### Q2: Weights

>   Implement the `planet` data abstraction by completing the `planet` constructor and the `size` selector so that a planet is represented using a two-element list where the first element is the string `'planet'` and the second element is its size.

从问题的描述中我们可以知道什么是 `planet`. 就是一个长度为 2 的 `list`, 内容是 `['planet', size]`. 可以参考 `mobile` 函数, 这两个函数的代码是很类似的

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

这个问题需要用递归的方法来解决. 我们判断一个 mobile 平衡的条件如下

1.   它是 `planet`, 根据描述可以知道本身是平衡的
2.   它是 `arm`, 并且 `total_weight(left_arm) == total_weight(right_arm)`. **并且它的所有子树都要符合这个平衡的条件**. 注意不要漏掉对子树的判断



写出代码的关键: 要区分开 `arm`, `planet`, `mobile` 这三个概念并且要能够知道用什么对应的函数来处理. :cry:

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

我们在这个问题中想要把 `mobile` 转换为 `tree`. 递归解法的 base case 是当我们遇到叶子结点而且是 `planet` 的时候. 否则我们就递归分析它的子树

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

递归解法的 base case 仍然是叶子结点, 我们要检查它的 label 是否为 'loki'. 如果是的话, 我们就创建一个新的叶子结点并返回. 否则我们返回本来的叶子结点即可.

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

1.   什么是这道题的 base case ? 我们要记住从根结点出发到当前所在结点一路经过的结点拼起来的字符串是什么. 举例来说, 在一开始的时候我们在根结点, 我们的字符串是 `label(t)`. 也就是 `h`. 当我们到了根结点的第一个子结点的时候, 我们就得到了 `h` + `i` = `hi`. 为了实现这样的功能我们可以另外写个递归函数. 然后在 `has_path` 里面调用即可. 一个提高效率的方法是如果我们当前得到的字符串不是目标字符串的一部分(开头位置), 那么我们显然没有必要继续到更深的子树里面寻找. 这也是递归的剪枝操作.
2.   怎么把问题分解为更简单的子问题 ? 我们可以使用 python 提供的 `any` 函数, 只要任意一个子树上存在这样的路径即可. 

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

先序遍历是遍历树的一个基本方法. 我们要先记住根结点的值再去访问它的子树上的结点.

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

这道题的计算系统其实是用来计算电阻的. 我们知道电阻会存在误差(比如 $\pm5\%$), 所以真正的电阻值应该是在一个区间里面的. 这就是这个所谓的的 `interval` 表示的. 所以 `interval` 其实也就是一个长度为 2 的 `list` 而已. 要得到它两侧的范围我们只要根据索引返回 `x[0]` 或 `x[1]` 即可.

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
>      """Return the interval that contains the product of any value in x and any
>      value in y."""
>      p1 = x[0] * y[0]
>      p2 = x[0] * y[1]
>      p3 = x[1] * y[0]
>      p4 = x[1] * y[1]
>      return [min(p1, p2, p3, p4), max(p1, p2, p3, p4)]
>   ```

这里面有很多违反了数据抽象原则的操作, 比如我们访问电阻的上下界的时候不应该直接用 `x[0]` 或 `x[1]`. ⚠️ 这其实是破坏了封装性, 万一以后我们换个一个实现电阻的方式, 这个代码就用不了了. 所以正确的方法应该是用前面实现的 `lower_bound()` 和 `upper_bound()` 这两个函数. 除此之外, 我们在创建一个 `interval` 的时候也应该用 `interval` 这个构造函数.

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

这个代码如果和 `mul_interval` 的几乎一模一样, 只是换了个操作符而已. 如果你跟我一样用的是 vim 作为编辑器, 那么用 `<ctrl-v>` 选中四个 `*` 之后我们用 `r-` 就可以快速替换掉了.

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

检查是否除数 > 0

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

如果你想知道这个的详细解释的话, 可以看一下这个 [链接](http://jots-jottings.blogspot.com/2011/09/sicp-exercise-214-broken-intervals.html)

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

