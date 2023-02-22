# Hw05 of CS61A of UCB(2021-Fall)


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

This problem asks us to return a generator that will return all the permutations of the list. We can learn from the hint that we need to solve this problem recursively. It's easy to figure out the base case of this problem: 

```python
def gen_perms(seq):
    if len(seq) <= 1:
        yield seq
    ...
```

Now, the problem is how can we break down this problem into simpler ones? We can learn from the hint that we should recursively process elements other than the element in the first position. Try to think about a question like this: If we have already got the permutations of `l[1:]`, what should we do with `l[0]`. The answer is quite intuitive: we insert `l[0]` to every possible position in the permutations of `l[1:]`.



For example, we have `[2, 3]`, whose permutations are `[[2, 3], [3, 2]]`. First, we insert `1` to `[2, 3]`, which give us `[1, 2, 3], [2, 1, 3], [2, 3, 1]`. Then we repeat the procedure on `[3, 2]`.



So we can write the following code:

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

The problem asks us to `yield` all possible path leads us to `value`. Pay a attention to the hint. The template of the code looks like this:

```python
def path_yielder(t, value):
    "*** YOUR CODE HERE ***"
    for _______________ in _________________:
        for _______________ in _________________:
            "*** YOUR CODE HERE ***"
```

You can find it that the template is quite similar to the solution in Q1 if you observe it carefully. First, we should ask ourselves the most important question? - What is the base case? Well, apparently, we meet the base case when we encounter the node with the `value` label. It can be any branch of the current node, so we need to check each branch. When we find the sub-path, we return it and add the label of the current node. 



For example, check the example in the doctoring. We want all the paths to lead us to `3`. First, we start from the root node(`1`). It has 2 branches - `2` and `5`. In the first branch of `1`, we can find `3`, at this point we `yield [3]`. When we backtrack to `2`, we get the sub-path - `[2, 3]`. Finally, we will get `[1] + [2, 3] = [1, 2, 3]` when we come back to the root node(`1`)



The code shows as below:

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

Again, we need to solve the problem of the preorder traversal of the tree. We have to recursively visit all nodes of this tree, starting from the root node. Note that we first memorize the value of the root node and then look at its subtrees, and handle such problems recursively.

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

The overall solution is very similar to Q3, but now we don't need to use a `result` to store the traversed sequence, because what we want to return is a generator. However, the logic of this algorithm is still the same. If you want to have a deeper understanding of `yield from`, we can look at this person's [this article](http://simeonvisser.com/posts/python-3-using-yield- from-in-generators-part-1.html)

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

If you have no idea what is the generator in python, you will find it difficult to figure out the solution. I will recommend [this article](https://realpython.com/introduction-to-python-generators/)



After reading the article aforementioned, we may start to try to solve this problem. What the problem wants are `m` generators, the ith yielded generator yields natural numbers whose remainder is i when divided by m.



If we know how to get the first generator, then we know how to get all the generators. I believe that it is easy to write a function that returns natural numbers whose remainder is i when divided by m. With the help of `yield`, we don't need a list to store the numbers(Just like what I did in Q3).

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

Then how can we get a list of generators. `for` loop !:hugs:

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




