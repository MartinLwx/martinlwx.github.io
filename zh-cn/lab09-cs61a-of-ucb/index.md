# Lab09 - CS61A of UCB(2021-Fall)

## Recursion and Tree Recursion

---

### Q1: Subsequences

>   A subsequence of a sequence `S` is a subset of elements from `S`, in the same order they appear in `S`. Consider the list `[1, 2, 3]`. Here are a few of it's subsequences `[]`, `[1, 3]`, `[2]`, and `[1, 2, 3]`.
>
>   Write a function that takes in a list and returns all possible subsequences of that list. The subsequences should be returned as a list of lists, where each nested list is a subsequence of the original input.
>
>   In order to accomplish this, you might first want to write a function `insert_into_all` that takes an item and a list of lists, adds the item to the beginning of each nested list, and returns the resulting list.

这一道题要求我们返回一个列表的所有可能子序列, 返回的格式是列表的列表, 每一个都是可能的子序列



题目要求我们首先完成一个函数: 功能是把 `item` 添加到嵌套列表的每个子列表的开头, 这个其实用 list comprehension 就可以了.

```python
def insert_into_all(item, nested_list):
    """Return a new list consisting of all the lists in nested_list,
    but with item added to the front of each. You can assume that
     nested_list is a list of lists.
    """
    return [[item] + l for l in nested_list]
```



这其实是题目给我们的提示, 我们现在思考有了这个函数我们要怎么找到所有可能的子序列呢? 我们可以递归分解问题: 当前元素 + 剩下元素的所有可能子序列(假设是 `tmp`). 那么我们只要把当前元素加到 `tmp` 中的每个列表再加上 `tmp` 即可. 这刚好就用上了题目让我们实现的 `insert_into_all()` 函数. 那么最后我们要思考什么对应这个 base case. 显然如果一个空的列表的子序列为空列表, 如果列表长度为 1, 则存在两个可能子序列 - 它自己 + 空列表. 最后我们就可以写出这样的代码

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

这一道题是在 Q1 的基础上改编而来的, 相当于提出了一个更高的要求, 我们要求子序列同时是**非降序的**.



这一题的提示告诉我们要实现一个 `subseq_helper` 函数, 这其实很好理解, 因为我们现在对子序列的大小顺序有要求, 那么我们就要多一个参数用来比较大小, 这样我们在插入 item 到列表的时候才知道可不可以插入(维持非降序). 比如 `[1, 3, 2]`, 当我们在检查 `3` 的时候我们发现 `3` < `1`, 显然我们不能把 `3` 插入到之前生成的子序列列表里. 而其他情况我们则可以选择把 `s[0]` 加到子序列列表中或者不加.

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

题意: 有 `n` 个叶子结点的完全二叉树可能有几种 ? 答案是卡特兰数, 所以我们要实现的其实是卡特兰数的递归写法. 至于为什么是卡特兰数我也想不大明白, 比较能接受的解释是, 完全二叉树的左右子树肯定也是完全二叉树, 假设左子树有 `1` 个叶子结点, 右子树就有 `n - 1` 个叶子结点, 那么此时就有 `f(1) * f(n - 1)` 种可能, 类似的, 如果左子树有 `2` 个叶子结点, 那就是 `f(2) * f(n - 2)`, 这样累加起来就是卡特兰数.



ps: 这里的完全二叉树不是严格意义上的, 确切来说这里指的是所有节点的度只能为 0 或者 2 的树

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

`merge` 函数的功能是合并两个有序的可迭代对象, 同时要做去重的工作, 可以假设两个有序的可迭代对象本身是没有元素重复的, 而且没有任何一个元素是 None. 同时不可以假定这两个可迭代对象是有限序列, 它们可能无序的(这样你就不能暴力合并为一个有序可迭代对象再去重)



因为两个可迭代对象本身不包含重复元素, 所以这一道题处理起来比较简单, 我们只要重复下面的过程: 

1.   如果两个可迭代对象都是非空
     1.   各取一个元素进行比较
          1.   如果一样大: 返回一个, 同时两个 iterator 都要往后移动
          2.   其中一个比较小: 返回小的这个, 移动小的这个可迭代对象的 iterator, 大的元素的 iterator 不动
2.   如果重复上面的操作导致其中一个已经空了, 那么接下来的问题就比较简单了, 此时我们只要用 `while` 循环不断从某一个可迭代对象中返回元素即可.



代码如下:

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

实现一个 `Account` 类, 要求有以下功能:

1.   存款
2.   取款, 钱不够的时候不让取
3.   查看操作历史. 转账历史是 tuple 的列表, 每个 tuple 包括了操作的类型和转账的金额



整体上而言这题不难, 看 `__repr__` 我们可以知道要求返回存款和取款的次数, 这里可以用两个变量来记住.

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

题意: 交换两个列表的开头几个元素(`m` 和 `n` 可以不等长), 使得两边被用来交换的子列表的和(前缀和)是一样的, 而且这个和要越小越好. 



在代码里已经为我们提供了交换元素的函数, 我们要做的就是让 `m` 和 `n` 停在正确的位置(他们的和一样), 这里用 `while` 循环来实现, 只要两个的索引是有效的(不然他们会一直增加, `while` 循环就会变为死循环)而且前缀和不想等, 我们移动 `m` 或者 `n` 指针.

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

这一道题就是要我们完成洗牌的功能, 洗牌的意思是前一半和后一半的元素交替出现, *举例来说*:`[0, 1, 2, 3, 4, 5] = [0, 3, 1, 4, 2, 5]`. 你可以看到奇数索引的是后一半的元素, 偶数索引的是前一半元素. 



这一道题的关键在于弄清楚洗牌之后的索引和原来的索引对应的关系, 总结来来说:`[0, 1, ..., len(cards) // 2, len(cards) // 2 + 1, ...]`. 你可以发现前一半和后一半对应位置的元素的索引相差 `len(cards) // 2`

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

根据指定的索引 `index` 在链表中插入元素, 如果索引非法, 抛出错误



这一题有点奇怪的地方在于, 它要求我们在原来的链表上进行修改, 但是如果我们要在链表的开头进行插入一个新节点, 会无法通过它的 `link is other_link` 的判断(因为插入后链表头是一个新的节点), 所以我这里想的办法是每次在插入前我们拷贝当前结点, 然后修改当前结点的值为想要插入的值, 这样等效于我们做了插入

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

Deep Linked List Length 其实就是一个可能包含链表为结点的嵌套链表结构. 这一道题要求我们算这种嵌套列表一共有多少个元素. 其实就是之前做的摊平链表的那种题目. 显然, 这是符合递归的嵌套结构, 所以我们可以用递归的办法解决.



base case 就是空链表或者它是一个元素而不是链表. 其他情况我们就递归处理链表的第一个节点和除了第一个结点以外的子链表 :hugs:

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

简单来说就是想要根据不同人的需求来打印链表, 具体格式就是 `front + 当前结点的值 + mid + 子链表的 + back` 这样

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

返回一个嵌套列表, 每个子列表表示长度至少为 `n` 的路径. 这里说的路径一定是叶子结点的路径 ! 路径的长度可以理解为从根结点出发到达叶子结点经过的边数.



这一道题其实是经典的递归与回溯问题, 我们要为其写一个 `helper` 函数, 要记住我们当前经过的点的路径, 以及路径的长度. 递归与回溯的模板大概如下:

```python
def function_name(p):
    # base case 
    ...
    
    dothing thing
    ...              # recursively solve this problem
    recall what you have done
```

放到我们这里就是我们要在往更深层递归的时候加上当前节点的 label, 当我们回溯的时候撤销我们对之前的添加. 代码如下:

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

