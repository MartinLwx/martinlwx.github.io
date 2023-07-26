# Lab08 - CS61A of UCB(2021-Fall)


### Q2: Convert Link

>   Write a function `convert_link` that takes in a linked list and returns the sequence as a Python list. You may assume that the input list is shallow; that is none of the elements is another linked list.
>
>   Try to find both an iterative and recursive solution for this problem!

迭代的算法很简单, 我们只要创建一个 `result` list 来存储结果, 遍历链表的同时记住访问过的数即可

```python
def convert_link(link):
    """Takes a linked list and returns a Python list with the same elements.
    """
    result = []
    while link is not Link.empty:
        result.append(int(link.first))
        link = link.rest
    return result
```

递归的算法也简单, base case 就是我们遇到了空结点的时候, 这个时候返回的是空的值, 其他情况我们递归分解: 当前结点 + 链表的剩余结点

```python
def convert_link(link):
    """Takes a linked list and returns a Python list with the same elements.
    """
    # recursive solution
    if link is Link.empty:
        return []
    else:
        return [int(link.first)] + convert_link(link.rest)
```

## Trees

---

### Q4: Square

>   Write a function `label_squarer` that mutates a `Tree` with numerical labels so that each label is squared.

遍历树的所有节点, 将它的 label 都改为 label 的平方

```python
def label_squarer(t):
    """Mutates a Tree t by squaring all its elements.
    """
    # base case
    if t.is_leaf():
        t.label = t.label ** 2

    # check every branch
    for b in t.branches:
        t.label = t.label ** 2          # change the current node's label
        label_squarer(b)                # change branches
```

### Q5: Cumulative Mul

>   Write a function `cumulative_mul` that mutates the Tree `t` so that each node's label becomes the product of its label and all labels in the subtrees rooted at the node.

这一题的意思是: 我们要让每个节点的 label 等于它的所有孩子节点的 label 的乘积, 显然, 这是一个递归问题



base case 就是叶子结点, 此时返回叶子节点的 label. 递归分解问题就是遍历节点的每一个子树, 获得每个子节点返回值再乘以当前结点的 label.



注意我这里用到了 `math.prod` 这个函数返回可迭代对象的连乘结果, 如果你也要这样使用, 你应该在文件的开头写 `import math`

```python
def cumulative_mul(t):
    """Mutates t so that each node's label becomes the product of all labels in
    """
    # base case
    if t.is_leaf():
        return t.label
    # get all label value in subtree
    vals = [cumulative_mul(b) for b in t.branches]

    # calculate
    t.label *= math.prod(vals)
```

### Q6: Add Leaves

>   Implement `add_d_leaves`, a function that takes in a `Tree` instance `t` and a number `v`.
>
>   We define the depth of a node in `t` to be the number of edges from the root to that node. The depth of root is therefore 0.
>
>   For each node in the tree, you should add `d` leaves to it, where `d` is the depth of the node. Every added leaf should have a label of `v`. If the node at this depth has existing branches, you should add these leaves to the end of that list of branches.
>
>   For example, you should be adding 1 leaf with label `v` to each node at depth 1, 2 leaves to each node at depth 2, and so on.
>
>   Here is an example of a tree `t`(shown on the left) and the result after `add_d_leaves` is applied with `v` as 5.

翻译一下这一道题的意思: 我们要根据结点在树中的高度(根结点高度为 0)来给结点添加子节点, 这里说的符合条件是 label 为 `v`. 高度为 `d` 就增加 `d` 个子节点.



这题第一个困难是要如何获取当前结点的高度, 因为有了高度我们才能知道要增加多少个子节点, 我们可以维护一个参数表示当前结点的高度. 每当我们往树的更深一层前进的时候我们就将它 `+1`.



base case 就是在叶子结点, 我们根据它的高度增加子结点. 然后对于当前的节点, 我们需要对它的每个孩子节点重复这个步骤, 同时也要判断当前节点是否需要添加子节点

```python
def add_d_leaves(t, v):
    """Add d leaves containing v to each node at every depth d.
    """
    def helper(t, v, depth):
        # base case
        if t.is_leaf():
            for i in range(depth):
                t.branches.append(Tree(v))
            return

        # check every branch
        for b in t.branches:
            helper(b, v, depth + 1)
        

        # check current node
        for i in range(depth):
            t.branches.append(Tree(v))

    helper(t, v, 0)
```

## Optional Questions

---

### Q7: Every Other

>   Implement `every_other`, which takes a linked list `s`. It mutates `s` such that all of the odd-indexed elements (using 0-based indexing) are removed from the list. For example:
>
>   ```
>   >>> s = Link('a', Link('b', Link('c', Link('d'))))
>   >>> every_other(s)
>   >>> s.first
>   'a'
>   >>> s.rest.first
>   'c'
>   >>> s.rest.rest is Link.empty
>   True
>   ```
>
>   If `s` contains fewer than two elements, `s` remains unchanged.
>
>   >   Do not return anything! `every_other` should mutate the original list.

我们可以用迭代的方法来处理这个问题, 首先要处理的是长度的问题, 如果链表的长度小于 2, 那么直接返回. 



否则我们就维护两个指针指向当前位置和上一个访问的位置, 当我们要删除索引为奇数(索引从 0 开始)的点的时候就让上一个节点直接指向当前这个节点的下一个节点即可.

```python
def every_other(s):
    """Mutates a linked list so that all the odd-indiced elements are removed
    """
    # if it contains fewer than 2, do nothing
    if s is Link.empty or s.rest is Link.empty:
        return

    last_pos, pos = s, s.rest
    current_index = 1               # start from 2nd position
    while pos is not Link.empty:
        if current_index % 2 == 1: 
            last_pos.rest = pos.rest
        last_pos = pos
        pos = pos.rest
        current_index += 1
```

### Q8: Prune Small

>   Complete the function `prune_small` that takes in a `Tree` `t` and a number `n` and prunes `t` mutatively. If `t` or any of its branches has more than `n` branches, the `n` branches with the smallest labels should be kept and any other branches should be *pruned*, or removed, from the tree.

这一道题要我们裁剪这棵树, 具体要求是才见到每个节点最多 `n` 个子树, 如果超过了 `n` 就优先移除 label 比较大的, 其实这一道题已经为我们提供了代码, 只要挖空填写就好了. 



不难相处, 我们应该自顶向下(从根节点出发)裁剪, 所以第一个 `while` 循环要完成的动作是如果分支数 > `n`, 那么找到最大的删掉



而后面的 `for` 循环则是要让我们到子树中递归裁减

```python
def prune_small(t, n):
    """Prune the tree mutatively, keeping only the n branches
    of each node with the smallest label.
    """
    while len(t.branches) > n:
        largest = max(t.branches, key=lambda x: x.label)
        t.branches.remove(largest)

    for b in t.branches:
        prune_small(b, n)
```

