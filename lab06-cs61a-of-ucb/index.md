# Lab06 CS61A of UCB(2021-Fall)


## Mutability

---

>   Write a function which takes in a list `lst`, an argument `entry`, and another argument `elem`. This function will check through each item in `lst` to see if it is equal to `entry`. Upon finding an item equal to `entry`, the function should modify the list by placing `elem` into `lst` right after the item. At the end of the function, the modified list should be returned.
>
>   See the doctests for examples on how this function is utilized.
>
>   **Important:** Use list mutation to modify the original list. No new lists should be created or returned.
>
>   >   **Note:** If the values passed into `entry` and `elem` are equivalent, make sure you're not creating an infinitely long list while iterating through it. If you find that your code is taking more than a few seconds to run, the function may be in a loop of inserting new values.

Note that we can't insert elements when we iterate using `for` loop. I remember I once saw this in *Effective Python*. You can find out by debugging yourself. Because the range of `i` is determined when we use `for i in range(len (lst))`. But when we insert new values inside the `for` loop, the `list` actually becomes longer (but the `i` is still in the original range). Therefore, elements that exceed the original length will not be accessed.



Note that the code below is wrong 🙅‍♂️

```python
def insert_items(lst, entry, elem):
    is_the_same = (entry == elem)
    while True:
        no_entry = True
        for i in range(len(lst)):
            if lst[i] == entry:
                if i == len(lst) - 1:
                    lst.append(elem)
                else:
                    lst.insert(i + 1, elem)
                no_entry = False
            # avoid infinite loop
            if is_the_same:                 
                i += 1
        if no_entry:
            return lst
```

The correct solution should be to use the `while` loop with the `list.index(x[, start[, end]])` method, the code is as follows:

```python
def insert_items(lst, entry, elem):
    """Inserts elem into lst after each occurence of entry and then returns lst.

    >>> test_lst = [1, 5, 8, 5, 2, 3]
    >>> new_lst = insert_items(test_lst, 5, 7)
    >>> new_lst
    [1, 5, 7, 8, 5, 7, 2, 3]
    >>> double_lst = [1, 2, 1, 2, 3, 3]
    >>> double_lst = insert_items(double_lst, 3, 4)
    >>> double_lst
    [1, 2, 1, 2, 3, 4, 3, 4]
    >>> large_lst = [1, 4, 8]
    >>> large_lst2 = insert_items(large_lst, 4, 4)
    >>> large_lst2
    [1, 4, 4, 8]
    >>> large_lst3 = insert_items(large_lst2, 4, 6)
    >>> large_lst3
    [1, 4, 6, 4, 6, 8]
    >>> large_lst3 is large_lst
    True
    >>> # Ban creating new lists
    >>> from construct_check import check
    >>> check(HW_SOURCE_FILE, 'insert_items',
    ...       ['List', 'ListComp', 'Slice'])
    True
    """
    pos, cnt = 0, 0
    for i in lst:
        if i == entry:
            cnt += 1
    
    while cnt > 0:
        idx = lst.index(entry, pos)
        pos = idx + 1
        if idx == len(lst) - 1:
            lst.append(elem)
        else:
            lst.insert(idx + 1, elem)
        cnt -= 1
    return lst
```

## Iterators

---

### Q4: Count Occurrences

>   Implement `count_occurrences`, which takes in an iterator `t` and returns the number of times the value `x` appears in the first `n` elements of `t`. A value appears in a sequence of elements if it is equal to an entry in the sequence.
>
>   >   **Note:** You can assume that `t` will have at least `n` elements.

This question is mainly to let us learn to use the two functions: `iter` and `next`. We can use `while n > 0` to control access to only the first `n` elements. We will iterate the range while counting occurrences.

```python
def count_occurrences(t, n, x):
    """Return the number of times that x appears in the first n elements of iterator t.

    >>> s = iter([10, 9, 10, 9, 9, 10, 8, 8, 8, 7])
    >>> count_occurrences(s, 10, 9)
    3
    >>> s2 = iter([10, 9, 10, 9, 9, 10, 8, 8, 8, 7])
    >>> count_occurrences(s2, 3, 10)
    2
    >>> s = iter([3, 2, 2, 2, 1, 2, 1, 4, 4, 5, 5, 5])
    >>> count_occurrences(s, 1, 3)
    1
    >>> count_occurrences(s, 4, 2)
    3
    >>> next(s)
    2
    >>> s2 = iter([4, 1, 6, 6, 7, 7, 8, 8, 2, 2, 2, 5])
    >>> count_occurrences(s2, 6, 6)
    2
    """
    it = iter(t)
    cnt = 0
    while n > 0:
        val = next(it)
        if val == x:
            cnt += 1
        n -= 1
    return cnt
```

### Q5: Repeated

>   Implement `repeated`, which takes in an iterator `t` and returns the first value in `t` that appears `k` times in a row.
>
>   >   **Note:** You can assume that the iterator `t` will have a value that appears at least `k` times in a row. If you are receiving a `StopIteration`, your `repeated` function is likely not identifying the correct value.
>
>   Your implementation should iterate through the items in a way such that if the same iterator is passed into `repeated` twice, it should continue in the second call at the point it left off in the first. An example of this behavior is in the doctests.

We address two important issues in this problem:

1. How to find consecutive `k` values? We need to set `last_val` to remember what the previous value was, so that it can be compared with the current value(while we iterate the `list`). 
2. How to ensure that the next call will start from the point it left off in the first? If you have followed the course carefully, you may remember how to do it. We should use a higher-order function, define another function in `repeated` and bind the iterator to this nested function. This way we can guarantee that we can start from the previous position every time. If you forget, you can look at this [link](http://composingprograms.com/pages/24-mutable -data.html)(2.4.4 Local state)

```python
def repeated(t, k):
    """Return the first value in iterator T that appears K times in a row.
    Iterate through the items such that if the same iterator is passed into
    the function twice, it continues in the second call at the point it left
    off in the first.

    >>> s = iter([10, 9, 10, 9, 9, 10, 8, 8, 8, 7])
    >>> repeated(s, 2)
    9
    >>> s2 = iter([10, 9, 10, 9, 9, 10, 8, 8, 8, 7])
    >>> repeated(s2, 3)
    8
    >>> s = iter([3, 2, 2, 2, 1, 2, 1, 4, 4, 5, 5, 5])
    >>> repeated(s, 3)
    2
    >>> repeated(s, 3)
    5
    >>> s2 = iter([4, 1, 6, 6, 7, 7, 8, 8, 2, 2, 2, 5])
    >>> repeated(s2, 3)
    2
    """
    assert k > 1
    last_val, it = None, iter(t)
    def helper(k):
        nonlocal last_val
        nonlocal it
        cnt = 0

        while True:
            val = next(it)

            if last_val is None or val != last_val:
                last_val, cnt = val, 1
            else:
                cnt += 1
                if cnt == k:
                    return val
    return helper(k)
```


