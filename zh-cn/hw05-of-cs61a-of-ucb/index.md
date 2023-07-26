# Hw05 - CS61A of UCB(2021-Fall)


### Q1: Generate Permutations

>   Given a sequence of unique elements, a *permutation* of the sequence is a list containing the elements of the sequence in some arbitrary order. For example, `[2, 1, 3]`, `[1, 3, 2]`, and `[3, 2, 1]` are some of the permutations of the sequence `[1, 2, 3]`.
>
>   Implement `gen_perms`, a generator function that takes in a sequence `seq` and returns a generator that yields all permutations of `seq`. For this question, assume that `seq` will not be empty.
>
>   Permutations may be yielded in any order. Note that the doctests test whether you are yielding all possible permutations, but not in any particular order. The built-in `sorted` function takes in an iterable object and returns a list containing the elements of the iterable in non-decreasing order.
>
>   >   *Hint:* If you had the permutations of all the elements in `seq` not including the first element, how could you use that to generate the permutations of the full `seq`?
>
>   >   *Hint:* Remember, it's possible to loop over generator objects because generators are iterators!

题意就是要我们返回一个 generator, 这个 generator 会一个个返回列表的所有排列组合. 这个 Hint 明摆着我们要用递归的方法解决这个问题. 很容易想到这一道题的 base case, 如下

```python
def gen_perms(seq):
    if len(seq) <= 1:
        yield seq
    ...
```

现在的问题就是要如何过渡到子问题, 根据提示我们可以猜: 我们应该递归处理除了第一个位置以外的元素, 再根据题目的另一个提示, 我们应该假定我们的 `gen_perms(seq)` 就是返回 `seq` 的全排列, 那我们只要将我们的第一个元素插入到全排列的任意一个位置即可. 举个例子来说, 比如我们处理 `[1, 2, 3]` 的全排列, 假设我们现在已经得到了 `[2, 3]` 的全排列——`[[2, 3], [3, 2]]`, 那么我们只要将 `1` 插入到 `[2, 3]` 的不同位置, 我们就可以得到 `[1, 2, 3], [2, 1, 3], [2, 3, 1]`, 然后我们对 `[3, 2]` 进行同样的步骤. 这样我们就得到了所有的全排列, 然后我们可以写出如下的代码

```python
def gen_perms(seq):
    """Generates all permutations of the given sequence. Each permutation is a
    list of the elements in SEQ in a different order. The permutations may be
    yielded in any order.

    >>> perms = gen_perms([100])
    >>> type(perms)
    <class 'generator'>
    >>> next(perms)
    [100]
    >>> try: #this piece of code prints "No more permutations!" if calling next would cause an error
    ...     next(perms)
    ... except StopIteration:
    ...     print('No more permutations!')
    No more permutations!
    >>> sorted(gen_perms([1, 2, 3])) # Returns a sorted list containing elements of the generator
    [[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]
    >>> sorted(gen_perms((10, 20, 30)))
    [[10, 20, 30], [10, 30, 20], [20, 10, 30], [20, 30, 10], [30, 10, 20], [30, 20, 10]]
    >>> sorted(gen_perms("ab"))
    [['a', 'b'], ['b', 'a']]
    """
    # This problem requires list type, see example above
    if type(seq) != list:
        seq = list(seq)
    # base case
    if len(seq) <= 1:
        yield seq
    else:
        # iterate every permutation in the generator
        for perm in gen_perms(seq[1:]):
            # enumerate every position for insertation
            for i in range(len(seq)):
                yield perm[:i] + seq[:1] + perm[i:]
```

### Q2: Yield Paths

>   Define a generator function `path_yielder` which takes in a tree `t`, a value `value`, and returns a generator object which yields each path from the root of `t` to a node that has label `value`.
>
>   Each path should be represented as a list of the labels along that path in the tree. You may yield the paths in any order.
>
>   We have provided a skeleton for you. You do not need to use this skeleton, but if your implementation diverges significantly from it, you might want to think about how you can get it to fit the skeleton.

题目让我们返回从根结点出发到达所有标签为 `value` 的路径. 注意给我们的提示, 我们要实现的函数应该如下所示:

```python
def path_yielder(t, value):
    "*** YOUR CODE HERE ***"
    for _______________ in _________________:
        for _______________ in _________________:
            "*** YOUR CODE HERE ***"
```

如果你仔细观察, 或许可以发现这一题的代码跟 Q1 竟是如此相似, 现在我们开始思考如何来解决这个问题. 首先要处理什么是 base case, 显然当我们遇到 label 是 `value` 的结点时候就到达了 base case 可以返回. 而将这个问题分解为更简单的子问题的方式也很简单, 我们要从当前结点出发, 遍历它的每一个子树, 只要路径上有就返回, 注意这里我们返回的是子路径(不包括当前结点), 然后我们加上当前结点的 label 即可.



用一个例子来说, 看下面的 docstring 里面的例子, 假设我们现在要找到所有到达 label 为 `3` 的所有路径, 我们从根结点 `1` 出发, 检查的它的每一个子树, 分别是 `2` 和 `5`. 然后我们接着对 `2` 这么处理, 在 `2` 的第一个子树中我们找到了 `3`, 此时我们 `yield [3]`. 当我们回溯到 `2` 的时候加上 `2`, 我们又得到了子路径 `[2, 3]`, 最后我们接着回溯到根结点, 得到 `[1] + [2, 3] = [1, 2, 3]`



代码如下:

```python
def path_yielder(t, value):
    """Yields all possible paths from the root of t to a node with the label
    value as a list.

    >>> t1 = tree(1, [tree(2, [tree(3), tree(4, [tree(6)]), tree(5)]), tree(5)])
    >>> print_tree(t1)
    1
      2
        3
        4
          6
        5
      5
    >>> next(path_yielder(t1, 6))
    [1, 2, 4, 6]
    >>> path_to_5 = path_yielder(t1, 5)
    >>> sorted(list(path_to_5))
    [[1, 2, 5], [1, 5]]

    >>> t2 = tree(0, [tree(2, [t1])])
    >>> print_tree(t2)
    0
      2
        1
          2
            3
            4
              6
            5
          5
    >>> path_to_2 = path_yielder(t2, 2)
    >>> sorted(list(path_to_2))
    [[0, 2], [0, 2, 1, 2]]
    """
    if label(t) == value:
        yield [label(t)]
    for b in branches(t):
        for path in path_yielder(b, value):
            yield [label(t)] + path
```

### Q3: Preorder

>   Define the function `preorder`, which takes in a tree as an argument and returns a list of all the entries in the tree in the order that `print_tree` would print them.
>
>   The following diagram shows the order that the nodes would get printed, with the arrows representing function calls.

又是树的先序遍历问题, 这真的是一个十分经典的问题. 我们要递归访问这一棵树的所有结点, 从根结点出发, 注意我们是先记住根结点的值再去看它的子树, 并递归式处理这样的问题.

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

### Q4: Generate Preorder

>   Similarly to `preorder` in Question 3, define the function `generate_preorder`, which takes in a tree as an argument and now instead `yield`s the entries in the tree in the order that `print_tree` would print them.
>
>   **Hint:** How can you modify your implementation of `preorder` to `yield from` your recursive calls instead of returning them?

仍旧是先序遍历的问题, 整体解法上很像 Q3, 只是现在我们不需要用一个 `result` 来存储遍历的序列, 因为我们要返回的是 generator. 但这个算法的逻辑还是一致的, 先看根结点, 然后去看它的子树的结点. 如果想对 `yield from` 有更深的了解, 我们可以看看这个人的[这篇文章](http://simeonvisser.com/posts/python-3-using-yield-from-in-generators-part-1.html)

```python
def generate_preorder(t):
    """Yield the entries in this tree in the order that they
    would be visited by a preorder traversal (see problem description).

    >>> numbers = tree(1, [tree(2), tree(3, [tree(4), tree(5)]), tree(6, [tree(7)])])
    >>> gen = generate_preorder(numbers)
    >>> next(gen)
    1
    >>> list(gen)
    [2, 3, 4, 5, 6, 7]
    """
    yield label(t)
    for b in branches(t):
        yield from generate_preorder(b)
```

### Q5: Remainder Generator

>   Like functions, generators can also be *higher-order*. For this problem, we will be writing `remainders_generator`, which yields a series of generator objects.
>
>   `remainders_generator` takes in an integer `m`, and yields `m` different generators. The first generator is a generator of multiples of `m`, i.e. numbers where the remainder is 0. The second is a generator of natural numbers with remainder 1 when divided by `m`. The last generator yields natural numbers with remainder `m - 1` when divided by `m`.
>
>   >   *Hint*: To create a generator of infinite natural numbers, you can call the `naturals` function that's provided in the starter code.
>
>   >   *Hint*: Consider defining an inner generator function. Each yielded generator varies only in that the elements of each generator have a particular remainder when divided by `m`. What does that tell you about the argument(s) that the inner function should take in?

这个问题如果你对 python 的 generator 一点都不了解的话, 一时要写出这样的代码是比较困难的, 我会推荐你先阅读这个[教程](https://realpython.com/introduction-to-python-generators/).



在阅读上面的材料之后我们开始以前来想这个问题要怎么解决. 其实它就是要返回 `m` 个 generator, 第 `i` 个 generator 里面分别是除以 `m` 得到余数为 `i` 的所有数.



注意, 我们要的是所有数, 所以我们可以现象我们要怎么得到第一个 generator, 其他 generator 其实都是类似的. 我们只要用一个 `while True` 死循环来一直找就行, 因为我们用的是 `yield`, 它每次找到一个就会暂停执行并返回那个数. 这样我们就得到了这个无限产生我们想要的数的 generator

```python
def helper(i, m):
    num = 1         # loop variable
    while True:
        if num % m == i:
            yield num
        num += 1
# you can give it a test
>>> it = helper(2, 3)
>>> next(it)
2				# 2 % 3 == 2
>>> next(it)
5				# 5 % 3 == 2
>>> next(it)
8				# 8 % 3 == 2
```

那么我们如何得到一个 generator 的 list 呢, 也很简单, 我们用 `for` 循环就好了

```python
def remainders_generator(m):
    """
    Yields m generators. The ith yielded generator yields natural numbers whose
    remainder is i when divided by m.

    >>> import types
    >>> [isinstance(gen, types.GeneratorType) for gen in remainders_generator(5)]
    [True, True, True, True, True]
    >>> remainders_four = remainders_generator(4)
    >>> for i in range(4):
    ...     print("First 3 natural numbers with remainder {0} when divided by 4:".format(i))
    ...     gen = next(remainders_four)
    ...     for _ in range(3):
    ...         print(next(gen))
    First 3 natural numbers with remainder 0 when divided by 4:
    4
    8
    12
    First 3 natural numbers with remainder 1 when divided by 4:
    1
    5
    9
    First 3 natural numbers with remainder 2 when divided by 4:
    2
    6
    10
    First 3 natural numbers with remainder 3 when divided by 4:
    3
    7
    11
    """
    def helper(i, m):
        num = 1         # loop variable
        while True:
            if num % m == i:
                yield num
            num += 1
    for i in range(m):
        yield helper(i, m)
```

