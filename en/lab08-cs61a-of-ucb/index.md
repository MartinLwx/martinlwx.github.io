# Lab08 - CS61A of UCB(2021-Fall)


### Q2: Convert Link

>   Write a function `convert_link` that takes in a linked list and returns the sequence as a Python list. You may assume that the input list is shallow; that is none of the elements is another linked list.
>
>   Try to find both an iterative and recursive solution for this problem!

It is easy to solve this problem iteratively. All we have to do is to make a `list` to store these nodes we have visited while we iterating this linklist.

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

The recursive algorithm is also easy. The base case is that when we encounter an empty node, an empty list will be returned. Otherwise, we will break down this problem: current node + the rest of tinkliest

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

The description indicates that we should make the label of the node equal to the product of its label and all labels in the subtrees in the tree.



The base is the leaf node, which has no subtrees. We will return its label. Otherwise, we check every subtree of the current node. The procedure of changing is bottom-up :hugs:



To make the code simple, I use `math.prod` here. If you also want to use this, you will need to `import math` at the beginning of the code.

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

We have to some details into account:

1.   How to get the height of the current node? We need to add leaves depending on the height, we can define a `helper` function with an additional parameter to represent height. Every time we go deeper in the tree, we will `+1`.
2.   How to solve it recursively? The base is the leaf node. We will add leaves to it and return here. Otherwise, we need to repeat the procedure to all its subtrees, and also determine whether the current node needs to add leaves.

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

First, If the length of the linklist is less than 2, we will keep it unchanged. Otherwise, we will solve this problem iteratively. 



We maintain 2 pointers - `last_pos` and `pos`, which indicate the current position and the last position we visited. If the current node is odd-indexed, we let `last_pos.rest = post.rest` and add the pointers.

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

Before we start to write the code, we need to think about what is the fast way to prune? The bottom-up way or the up-down way? The latter is the answer. Because we don't have to check the nodes in the subtrees we have pruned. You may figure out this by observing the template provided in the hint. :hugs:

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


