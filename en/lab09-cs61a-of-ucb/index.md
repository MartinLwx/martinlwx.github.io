# Lab09 - CS61A of UCB(2021-Fall)


## Recursion and Tree Recursion

---

### Q1: Subsequences

>   A subsequence of a sequence `S` is a subset of elements from `S`, in the same order they appear in `S`. Consider the list `[1, 2, 3]`. Here are a few of it's subsequences `[]`, `[1, 3]`, `[2]`, and `[1, 2, 3]`.
>
>   Write a function that takes in a list and returns all possible subsequences of that list. The subsequences should be returned as a list of lists, where each nested list is a subsequence of the original input.
>
>   In order to accomplish this, you might first want to write a function `insert_into_all` that takes an item and a list of lists, adds the item to the beginning of each nested list, and returns the resulting list.

This problem requires us to write a function which can return all possible subsequences of its input.



The hint says that we need to implement the `insert_into_all` function first. We can do this by a simple list comprehension.

```python
def insert_into_all(item, nested_list):
    """Return a new list consisting of all the lists in nested_list,
    but with item added to the front of each. You can assume that
     nested_list is a list of lists.
    """
    return [[item] + l for l in nested_list]
```

The `insert_into_all` function indicates the way to solve this problem. We need to ask ourself what we can do with this function? We can slove it recursively. The input can be broken down into 2 parts: the first element and the subsequences of the rest part of input(`tmp`). We need to add the first element in `tmp`, which can be done with the `insert_into_all` function. Finally, what is the base case? It is an empty list or a list with a size 1.

```python
def subseqs(s):
    """Return a nested list (a list of lists) of all subsequences of S.
    The subsequences can appear in any order. You can assume S is a list.
    """
    if len(s) <= 1:
        return [[], s] if s !=[] else [[]]
    else:
        tmp = subseqs(s[1:])
        return insert_into_all(s[0], tmp) + tmp
```

### Q2: Non-Decreasing Subsequences

>   Just like the last question, we want to write a function that takes a list and returns a list of lists, where each individual list is a subsequence of the original input.
>
>   This time we have another condition: we only want the subsequences for which consecutive elements are *nondecreasing*. For example, `[1, 3, 2]` is a subsequence of `[1, 3, 2, 4]`, but since 2 < 3, this subsequence would *not* be included in our result.
>
>   **Fill in the blanks** to complete the implementation of the `non_decrease_subseqs` function. You may assume that the input list contains no negative elements.
>
>   You may use the provided helper function `insert_into_all`, which takes in an `item` and a list of lists and inserts the `item` to the front of each list.

In Q1, we have written the function which can return all the possible subsequences. However, we also require the subsequence should be non-decreasing.



The hint says that we need to implement a `subseq_helper` function, which has an additional parameter - `prev`. Because we now have requirements on the subsequences, then we need the `prev` to tell us if we could insert the first element into all subsequences.

```python
def non_decrease_subseqs(s):
    """Assuming that S is a list, return a nested list of all subsequences
    of S (a list of lists) for which the elements of the subsequence
    are strictly nondecreasing. The subsequences can appear in any order.
    """
    def subseq_helper(s, prev):
        if not s:
            return [[]]
        elif s[0] < prev:
            return subseq_helper(s[1:], prev)
        else:
            a = subseq_helper(s[1:], s[0])  # include s[0]
            b = subseq_helper(s[1:], prev)  # exclude s[0]
            return insert_into_all(s[0], a) + b
    return subseq_helper(s, 0)
```

### Q3: Number of Trees

>   A **full binary tree** is a tree where each node has either 2 branches or 0 branches, but never 1 branch.
>
>   Write a function which returns the number of unique full binary tree structures that have exactly n leaves.
>
>   For those interested in combinatorics, this problem does have a [closed form solution](http://en.wikipedia.org/wiki/Catalan_number)):

The answer is the Catalan number. So we need to write a function to calculate the ith Catalan number. As for why it is a catalan number, I can barely understand this. I found the simple explanation: the subtrees of a full binary tree must also be full binary trees. Assuming that we have `1` leaf node in the left subtree, then we have `n - 1` leaf nodes in the right subtree. Then there are `f(1) * f(n - 1)` possibilities. Similarly, we need to calculate `f(2) * f(n - 2)` ...



ps: The definition of a full binary tree here is not strict. Actually, it means all the degrees of nodes in the tree should be either 0 or 2.

```python
def num_trees(n):
    """Returns the number of unique full binary trees with exactly n leaves. E.g.,
    """
    if n == 1 or n == 2:
        return 1
    # catalan number
    ans = 0
    for i in range(1, n):
        ans += num_trees(i) * num_trees(n - i)
    return ans
```

## Generators

---

### Q4: Merge

>   Implement `merge(incr_a, incr_b)`, which takes two iterables `incr_a` and `incr_b` whose elements are ordered. `merge` yields elements from `incr_a` and `incr_b` in sorted order, eliminating repetition. You may assume `incr_a`and `incr_b` themselves do not contain repeats, and that none of the elements of either are `None`. You may **not**assume that the iterables are finite; either may produce an infinite stream of results.
>
>   You will probably find it helpful to use the two-argument version of the built-in `next` function: `next(incr, v)` is the same as `next(incr)`, except that instead of raising `StopIteration` when `incr` runs out of elements, it returns `v`.
>
>   See the doctest for examples of behavior.

What the `merge` do is merging two iterable objects. It can be assumed that the two iterable objects have no duplicate elements individually, and none of the elements is `None`. Because we can't assume that the two iterable objects are finite, we can't solve this in a brute-force way(Merge them and sort and use `set` to move duplicates).



The procedure is:

1.   If both of the two iterable objects are non-empty
     1.   Compare the first element each to compare
          1.   They are equivalent: `yield` one, and move the two iterators backward.
          2.   `yield` the smaller element, move its iterator
2.   Repeat until one of the two iterable objects is empty. Then we only need to iterate the non-empty one instead.

```python
def merge(incr_a, incr_b):
    """Yield the elements of strictly increasing iterables incr_a and incr_b, removing
    repeats. Assume that incr_a and incr_b have no repeats. incr_a or incr_b may or may not
    be infinite sequences.
    """
    iter_a, iter_b = iter(incr_a), iter(incr_b)
    next_a, next_b = next(iter_a, None), next(iter_b, None)

    # both are non-empty
    while next_a is not None and next_b is not None:
        val_a, val_b = next_a, next_b
        if val_a == val_b:
            yield next_a
            next_a, next_b = next(iter_a, None), next(iter_b, None)
        elif val_a < val_b:
            yield next_a
            next_a = next(iter_a, None)
        else:
            yield next_b
            next_b = next(iter_b, None)
    # incr_a is not empty
    while next_a:
        yield next_a
        next_a = next(iter_a, None)
    # incr_b is not empty
    while next_b:
        yield next_b
        next_b = next(iter_b, None)
```

## Objects

---

### Q5: Bank Account

>   Implement the class `Account`, which acts as a a Bank Account. `Account` should allow the account holder to deposit money into the account, withdraw money from the account, and view their transaction history. The Bank Account should also prevents a user from withdrawing more than the current balance.
>
>   Transaction history should be stored as a list of tuples, where each tuple contains the type of transaction and the transaction amount. For example a withdrawal of 500 should be stored as ('withdraw', 500)
>
>   >   **Hint:** You can call the `str` function on an integer to get a string representation of the integer. You might find this function useful when implementing the `__repr__` and `__str__` methods.
>   >
>   >   **Hint:** You can alternatively use [fstrings](https://realpython.com/python-f-strings/) to implement the `__repr__` and `__str__` methods cleanly.

The `Account` should be able to:

1.   deposit
2.   withdraw
3.   check the history of what we have done.



Looking into the `__repr__` method, we know we need to know the number of times the deposits and the withdrawals. So we can define two variables to memorize them.

```python
class Account:
    """A bank account that allows deposits and withdrawals.
    It tracks the current account balance and a transaction
    history of deposits and withdrawals.
    """

    interest = 0.02

    def __init__(self, account_holder):
        self.balance = 0
        self.holder = account_holder
        self.transactions = []
        self.withdraw_cnt = 0
        self.deposit_cnt = 0

    def deposit(self, amount):
        """Increase the account balance by amount, add the deposit
        to the transaction history, and return the new balance.
        """
        self.balance += amount
        self.transactions.append(('deposit', amount))
        self.deposit_cnt += 1
        return self.balance

    def withdraw(self, amount):
        """Decrease the account balance by amount, add the withdraw
        to the transaction history, and return the new balance.
        """
        if self.balance > amount:
            self.balance -= amount
            self.transactions.append(('withdraw', amount))
            self.withdraw_cnt += 1
            return self.balance
        # prevent illegal withdraw
        return self.balance

    def __str__(self):
        return f"{self.holder}'s Balance: ${self.balance}"

    def __repr__(self):
        return f"Accountholder: {self.holder}, Deposits: {self.deposit_cnt}, Withdraws: {self.withdraw_cnt}"
```

## Mutable Lists

---

### Q6: Trade

>   In the integer market, each participant has a list of positive integers to trade. When two participants meet, they trade the smallest non-empty prefix of their list of integers. A prefix is a slice that starts at index 0.
>
>   Write a function `trade` that exchanges the first `m` elements of list `first` with the first `n` elements of list `second`, such that the sums of those elements are equal, and the sum is as small as possible. If no such prefix exists, return the string `'No deal!'` and do not change either list. Otherwise change both lists and return `'Deal!'`. A partial implementation is provided.
>
>   >   **Hint:** You can mutate a slice of a list using *slice assignment*. To do so, specify a slice of the list `[i:j]` on the left-hand side of an assignment statement and another list on the right-hand side of the assignment statement. The operation will replace the entire given slice of the list from `i` inclusive to `j` exclusive with the elements from the given list. The slice and the given list need not be the same length.
>   >
>   >   ```
>   >   >>> a = [1, 2, 3, 4, 5, 6]
>   >   >>> b = a
>   >   >>> a[2:5] = [10, 11, 12, 13]
>   >   >>> a
>   >   [1, 2, 10, 11, 12, 13, 6]
>   >   >>> b
>   >   [1, 2, 10, 11, 12, 13, 6]
>   >   ```
>   >
>   >   Additionally, recall that the starting and ending indices for a slice can be left out and Python will use a default value. `lst[i:]` is the same as `lst[i:len(lst)]`, and `lst[:j]` is the same as `lst[0:j]`.

It has provided us with some code that can exchange elements. All we have to do is to make `m` and `n` stop in right place. We need to make sure that the `m` and `n` are leal indices.

```python
def trade(first, second):
    """Exchange the smallest prefixes of first and second that have equal sum.
    """
    m, n = 1, 1

    equal_prefix = lambda: sum(first[:m]) == sum(second[:n])
    while m <= len(first) and n <= len(second) and not equal_prefix():
        if sum(first[:m]) < sum(second[:n]):
            m += 1
        else:
            n += 1

    if equal_prefix():
        first[:m], second[:n] = second[:n], first[:m]
        return 'Deal!'
    else:
        return 'No deal!'
```

### Q7: Shuffle

>   Define a function `shuffle` that takes a sequence with an even number of elements (cards) and creates a new list that interleaves the elements of the first half with the elements of the second half.
>
>   To interleave two sequences `s0` and `s1` is to create a new sequence such that the new sequence contains (in this order) the first element of `s0`, the first element of `s1`, the second element of `s0`, the second element of `s1`, and so on. If the two lists are not the same length, then the leftover elements of the longer list should still appear at the end.
>
>   >   **Note:** If you're running into an issue where the special heart / diamond / spades / clubs symbols are erroring in the doctests, feel free to copy paste the below doctests into your file as these don't use the special characters and should not give an "illegal multibyte sequence" error.

This problem requires us to implement the `shuffle` function. For example, `[0, 1, 2, 3, 4, 5] = [0, 3, 1, 4, 2, 5]`.



The key: the relationship between the indices - `[0, 1, ..., len(cards) // 2, len(cards) // 2 + 1, ...]`. You can find that the indices of the elements in the corresponding positions of the first half and the second half differ by `len(cards) // 2`

```python
def shuffle(cards):
    """Return a shuffled list that interleaves the two halves of cards.
    """
    assert len(cards) % 2 == 0, 'len(cards) must be even'
    half = len(cards) // 2
    shuffled = []
    for i in range(half):
        shuffled.append(cards[i])
        shuffled.append(cards[i + half])
    return shuffled
```

## Linked Lists

---

### Q8: Insert

>   Implement a function `insert` that takes a `Link`, a `value`, and an `index`, and inserts the `value` into the `Link` at the given `index`. You can assume the linked list already has at least one element. Do not return anything -- `insert` should mutate the linked list.
>
>   >   **Note**: If the index is out of bounds, you should raise an `IndexError` with:
>   >
>   >   ```
>   >   raise IndexError('Out of bounds!')
>   >   ```

It is strange that if we insert a new node(the index = 0), we won't pass the `link is other_link` test(because we have changed the head of the link). So I change my way to insert: I make a copy of the current node and then change the value of the origin node.

```python
def insert(link, value, index):
    """Insert a value into a Link at the given index.
    """
    pos = link
    current_index = 0
    while pos is not Link.empty:
        if current_index == index:
            # make a copy of current node, and modify the current node's value \
            # which is equal to insert a new node :)
            current_copy = Link(pos.first, pos.rest)
            origin_next = pos.rest
            pos.first = value
            pos.rest = current_copy
            #print(f"link: {link.first}")
            return 
        pos = pos.rest
        current_index += 1
    raise IndexError('Out of bounds!')
```

### Q9: Deep Linked List Length

>   A linked list that contains one or more linked lists as elements is called a *deep* linked list. Write a function `deep_len` that takes in a (possibly deep) linked list and returns the *deep length* of that linked list. The deep length of a linked list is the total number of non-link elements in the list, as well as the total number of elements contained in all contained lists. See the function's doctests for examples of the deep length of linked lists.
>
>   >   **Hint:** Use `isinstance` to check if something is an instance of an object.

Deep Linked List Length is a nested list. We need to calculate how many nodes are in it. To solve the problem of the **nested** linklist, we can solve it recursively. 



The base case is an empty linklist or it is not a linklist. Otherwise, we will check the `lnk.first`(it may be a linklist) and the `lnk.rest`

```python
def deep_len(lnk):
    """ Returns the deep length of a possibly deep linked list.
    """
    # base case 1. an empty node
    if lnk is Link.empty:
        return 0
    # base case 2. an integer
    elif isinstance(lnk, int): 
        return 1
    else:
        return deep_len(lnk.first) + deep_len(lnk.rest)
```

### Q10: Linked Lists as Strings

>   Kevin and Jerry like different ways of displaying the linked list structure in Python. While Kevin likes box and pointer diagrams, Jerry prefers a more futuristic way. Write a function `make_to_string` that returns a function that converts the linked list to a string in their preferred style.
>
>   *Hint*: You can convert numbers to strings using the `str` function, and you can combine strings together using `+`.
>
>   ```
>   >>> str(4)
>   '4'
>   >>> 'cs ' + str(61) + 'a'
>   'cs 61a'
>   ```

The format:  `front + the value of the current node + mid + the string of lnk.rest + back` 

```python
def make_to_string(front, mid, back, empty_repr):
    """ Returns a function that turns linked lists to strings.
    """
    def printer(lnk):
        if lnk is Link.empty:
            return empty_repr
        else:
            return front + str(lnk.first) + mid + printer(lnk.rest) + back
    return printer
```

## Trees

---

### Q11: Long Paths

>   Implement `long_paths`, which returns a list of all *paths* in a tree with length at least `n`. A path in a tree is a list of node labels that starts with the root and ends at a leaf. Each subsequent element must be from a label of a branch of the previous value's node. The *length* of a path is the number of edges in the path (i.e. one less than the number of nodes in the path). Paths are ordered in the output list from left to right in the tree. See the doctests for some examples.

We need to return a nested list, each one of them represents a path with a length at least `n`. Notice that the path must end in a leaf node. 



Actually, it is a classical problem that can be solved by recursion and backtrace. The skeleton is like:

```python
def function_name(p):
    # base case 
    ...
    
    dothing thing
    ...              # recursively solve this problem
    undo what you have done
```

We need to *add* the label of the current node in our `current_path` list when we are about to go deepr in the tree and undo what our modifications when we are about move up in the tree.

```python
def long_paths(t, n):
    """Return a list of all paths in t with length at least n.
    """
    path_list = []
    def helper(t, current_path, length):
        nonlocal path_list
        if t.is_leaf():
            current_path.append(t.label)
            if length >= n:
                # warning: we need to pass a copy instead fo a ref
                path_list.append(current_path[:])
            current_path.pop()
            return
        current_path.append(t.label)
        for b in t.branches:
            helper(b, current_path, length + 1)
        current_path.pop()
    helper(t, [], 0)
    return path_list
```


