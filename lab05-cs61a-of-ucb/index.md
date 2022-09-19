# Lab05 CS61A of UCB(2021-Fall)


## Lists

---

### Q1: Factors List

>   Write `factors_list`, which takes a number `n` and returns a list of its factors in ascending order.

We can know such a basic mathematical fact: the factor of a number can only be up to half of it (when the number is even). So we use `for i in range(1, n // 2 + 1)`

```python
def factors_list(n):
    """Return a list containing all the numbers that divide `n` evenly, except
    for the number itself. Make sure the list is in ascending order.

    >>> factors_list(6)
    [1, 2, 3]
    >>> factors_list(8)
    [1, 2, 4]
    >>> factors_list(28)
    [1, 2, 4, 7, 14]
    """
    all_factors = []
    # the biggest number which can divide `n` evenly will be `n // 2`
    for i in range(1, n // 2 + 1):
        if n % i == 0:
            all_factors.append(i)
    return all_factors
```

### Q2: Flatten

>   Write a function `flatten` that takes a list and "flattens" it. The list could be a deep list, meaning that there could be a multiple layers of nesting within the list.
>
>   For example, one use case of `flatten` could be the following:
>
>   ```
>   >>> lst = [1, [[2], 3], 4, [5, 6]]
>   >>> flatten(lst)
>   [1, 2, 3, 4, 5, 6]
>   ```
>
>   Make sure your solution does not mutate the input list.

Apparently, we can solve this probelm recursively. For nested list, we need to recursively decompose it.

1.   What is the base case ? If we get a empty `list` then we return `[]`
2.   How to break down the current problem into simpler ones ? That is how we can reduce the size of the problem, we can try to process the element in the first position of the nested `list` each time, depending on whether they are `list` type, which can break down this probelm into 2 cases.



Finally, the code will be like:

```python
def flatten(s):
    """Returns a flattened version of list s.

    >>> flatten([1, 2, 3])     # normal list
    [1, 2, 3]
    >>> x = [1, [2, 3], 4]     # deep list
    >>> flatten(x)
    [1, 2, 3, 4]
    >>> x # Ensure x is not mutated
    [1, [2, 3], 4]
    >>> x = [[1, [1, 1]], 1, [1, 1]] # deep list
    >>> flatten(x)
    [1, 1, 1, 1, 1, 1]
    >>> x
    [[1, [1, 1]], 1, [1, 1]]
    """
    if s == []:
        return []
    elif type(s[0]) == list:
        return flatten(s[0]) + flatten(s[1:])
    else:
        return s[:1] + flatten(s[1:])
```

## Data Abstraction

---

### Q3: Distance

>   We will now implement the function `distance`, which computes the distance between two city objects. Recall that the distance between two coordinate pairs `(x1, y1)` and `(x2, y2)` can be found by calculating the `sqrt` of `(x1 - x2)**2 + (y1 - y2)**2`. We have already imported `sqrt` for your convenience. Use the latitude and longitude of a city as its coordinates; you'll need to use the selectors to access this info!

We just need to call `get_lat` and `get_lon` on the arguments to get the value and then calculate it :hugs:

```python
def distance(city_a, city_b):
    """
    >>> city_a = make_city('city_a', 0, 1)
    >>> city_b = make_city('city_b', 0, 2)
    >>> distance(city_a, city_b)
    1.0
    >>> city_c = make_city('city_c', 6.5, 12)
    >>> city_d = make_city('city_d', 2.5, 15)
    >>> distance(city_c, city_d)
    5.0
    """
    x = (get_lat(city_a) - get_lat(city_b)) ** 2
    y = (get_lon(city_a) - get_lon(city_b)) ** 2
    return sqrt(x + y)
```

### Q4: Closer city

>   Next, implement `closer_city`, a function that takes a latitude, longitude, and two cities, and returns the name of the city that is relatively closer to the provided latitude and longitude.
>
>   You may only use the selectors and constructors introduced above and the `distance` function you just defined for this question.
>
>   >   **Hint**: How can you use your `distance` function to find the distance between the given location and each of the given cities?

According to the **Hint** and our `distance` function wrote in Q3, we know that we should build a *virtual city* based on `lat` and `lon`, and then use the `distance` function to calculate and compare the distance. This saves you from writing a lot of repetitive code.

```python
def closer_city(lat, lon, city_a, city_b):
    """
    Returns the name of either city_a or city_b, whichever is closest to
    coordinate (lat, lon). If the two cities are the same distance away
    from the coordinate, consider city_b to be the closer city.

    >>> berkeley = make_city('Berkeley', 37.87, 112.26)
    >>> stanford = make_city('Stanford', 34.05, 118.25)
    >>> closer_city(38.33, 121.44, berkeley, stanford)
    'Stanford'
    >>> bucharest = make_city('Bucharest', 44.43, 26.10)
    >>> vienna = make_city('Vienna', 48.20, 16.37)
    >>> closer_city(41.29, 174.78, bucharest, vienna)
    'Bucharest'
    """
    tmp = make_city('tmp', lat, lon)
    if distance(tmp, city_a) < distance(tmp, city_b):
        return get_name(city_a)
    else:
        return get_name(city_b)
```

## Trees

---

### Q6: Finding Berries!

>   The squirrels on campus need your help! There are a lot of trees on campus and the squirrels would like to know which ones contain berries. Define the function `berry_finder`, which takes in a tree and returns `True` if the tree contains a node with the value `'berry'` and `False` otherwise.
>
>   >   **Hint**: To iterate through each of the branches of a particular tree, you can consider using a `for` loop to get each branch.

This is a very classic tree recursion problem. We need to find the node whose label is `berry` in all nodes. The recursive idea is as follows:

1.   What is the base case ? Apparently, we come to the base case when we come to the leaf node. All we have to do here is to check the label of it.
2.   How to break down the current problem into simpler ones ? It is easy to figure out that we only need to have `berry` in a node of any branch, or the label of our current branch(*root* node in this branch) is `berry`.

```python
def berry_finder(t):
    """Returns True if t contains a node with the value 'berry' and 
    False otherwise.

    >>> scrat = tree('berry')
    >>> berry_finder(scrat)
    True
    >>> sproul = tree('roots', [tree('branch1', [tree('leaf'), tree('berry')]), tree('branch2')])
    >>> berry_finder(sproul)
    True
    >>> numbers = tree(1, [tree(2), tree(3, [tree(4), tree(5)]), tree(6, [tree(7)])])
    >>> berry_finder(numbers)
    False
    >>> t = tree(1, [tree('berry',[tree('not berry')])])
    >>> berry_finder(t)
    True
    """
    if is_leaf(t):
        return label(t) == 'berry'
    return any([berry_finder(b) for b in branches(t)]) or label(t) == 'berry'
```

### Q7: Sprout leaves

>   Define a function `sprout_leaves` that takes in a tree, `t`, and a list of leaves, `leaves`. It produces a new tree that is identical to `t`, but where each old leaf node has new branches, one for each leaf in `leaves`.
>
>   For example, say we have the tree `t = tree(1, [tree(2), tree(3, [tree(4)])])`:
>
>   ```
>     1
>    / \
>   2   3
>       |
>       4
>   ```
>
>   If we call `sprout_leaves(t, [5, 6])`, the result is the following tree:
>
>   ```
>          1
>        /   \
>       2     3
>      / \    |
>     5   6   4
>            / \
>           5   6
>   ```

This is also a problem can be solved recursively. Obviously, the base case is the leaf node again. 



When we encounter a leaf node, we need to add `leaves`. If it is a branch, then we have to check each of its subtrees.

```python
def sprout_leaves(t, leaves):
    """Sprout new leaves containing the data in leaves at each leaf in
    the original tree t and return the resulting tree.

    >>> t1 = tree(1, [tree(2), tree(3)])
    >>> print_tree(t1)
    1
      2
      3
    >>> new1 = sprout_leaves(t1, [4, 5])
    >>> print_tree(new1)
    1
      2
        4
        5
      3
        4
        5

    >>> t2 = tree(1, [tree(2, [tree(3)])])
    >>> print_tree(t2)
    1
      2
        3
    >>> new2 = sprout_leaves(t2, [6, 1, 2])
    >>> print_tree(new2)
    1
      2
        3
          6
          1
          2
    """
    if is_leaf(t):
        return tree(label(t), [tree(leaf) for leaf in leaves])
    return tree(label(t), [sprout_leaves(b, leaves) for b in branches(t)])
```


