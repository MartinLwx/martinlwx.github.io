# Hw03 of CS61A of UCB(2021-Fall)


## hw03. Recursion, Tree Recursion

---

### Q1: Num eights

>   Write a recursive function `num_eights` that takes a positive integer `pos` and returns the number of times the digit 8 appears in `pos`.
>
>   
>
>   **Important:** Use recursion; the tests will fail if you use any assignment statements. (You can however use function definitions if you so wish.)

It is easy to think of the answer to this question, for we have seen a similar one in lecture. We can split the `pos` into `all_bust_last` and `last`. When we arrive at the base case, we just need to check if it is equals to 8. Because we can not use `=` in this problem, we need to use function instead.

```python
def is_single_digit(digits):
    return digits // 10 == 0

def is_eight(digit):
    return int(digit == 8)

def num_eights(pos):
    """Returns the number of times 8 appears as a digit of pos.

    >>> num_eights(3)
    0
    >>> num_eights(8)
    1
    >>> num_eights(88888888)
    8
    >>> num_eights(2638)
    1
    >>> num_eights(86380)
    2
    >>> num_eights(12345)
    0
    >>> from construct_check import check
    >>> # ban all assignment statements
    >>> check(HW_SOURCE_FILE, 'num_eights',
    ...       ['Assign', 'AnnAssign', 'AugAssign', 'NamedExpr'])
    True
    """
    # base case: if pos is a single digit, check if it == 8
    if is_single_digit(pos):
        return is_eight(pos)
    else:
        return is_eight(pos % 10) + num_eights(pos // 10)
```

### Q2: Ping-pong

>   The ping-pong sequence counts up starting from 1 and is always either counting up or counting down. At element `k`, the direction switches if `k` is a multiple of 8 or contains the digit 8. The first 30 elements of the ping-pong sequence are listed below, with direction swaps marked using brackets at the 8th, 16th, 18th, 24th, and 28th elements:
>
>   | Index          | 1    | 2    | 3    | 4    | 5    | 6    | 7    | [8]  | 9    | 10   | 11   | 12   | 13   | 14   | 15   | [16] | 17   | [18] | 19   | 20   | 21   | 22   | 23   |
>   | :------------- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
>   | PingPong Value | 1    | 2    | 3    | 4    | 5    | 6    | 7    | [8]  | 7    | 6    | 5    | 4    | 3    | 2    | 1    | [0]  | 1    | [2]  | 1    | 0    | -1   | -2   | -3   |
>
>   | Index (cont.)  | [24] | 25   | 26   | 27   | [28] | 29   | 30   |
>   | :------------- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
>   | PingPong Value | [-4] | -3   | -2   | -1   | [0]  | -1   | -2   |
>
>   Implement a function `pingpong` that returns the nth element of the ping-pong sequence *without using any assignment statements*. (You are allowed to use function definitions.)
>
>   You may use the function `num_eights`, which you defined in the previous question.
>
>   **Important:** Use recursion; the tests will fail if you use any assignment statements. (You can however use function definitions if you so wish.)

At first, we need to define what is the base case in this problem. 



Let's solve this problem from scratch. The iterative solutioin is quite easy and intuitive. We need to track these things:arrow_down:

1.   index(`i`), we need to count it to `n`.
2.   value(`val`), we need to keep track of this state. When we count `i` to `n`, this is the expected output.
3.   `direction`, this is a variable that is either equal to `-1` or `1`. We maintain this variable to indicate counting up or counting down. If it is a multiple of 8 or contains the digit 8, we need to change its value.

```python
def pingpong(n):
    i, direction = 0, 1
    val = 0
    while i < n:
        val += direction
        i += 1
        if num_eights(i) > 0 or i % 8 == 0:
            direction = -direction
    return val
```

The problem is how to convert this iterative solution to recursive solution. The parameters of this function should be these variables which changes in `while` loop. The next problem is: *what is the base case*? Apparently, the base case can be defined as `index < 8`, then return `index`. Because in this range, `index` = `value`. Then, how can we know what the `direction` is? If we define `direction(n)` as the direction when the `index = n`, we can reason its value from `direction(n-1)`. The base case are quite similar. Finally, the recursive solution will be like:

```python
def pingpong(n):
    """Return the nth element of the ping-pong sequence.

    >>> pingpong(8)
    8
    >>> pingpong(10)
    6
    >>> pingpong(15)
    1
    >>> pingpong(21)
    -1
    >>> pingpong(22)
    -2
    >>> pingpong(30)
    -2
    >>> pingpong(68)
    0
    >>> pingpong(69)
    -1
    >>> pingpong(80)
    0
    >>> pingpong(81)
    1
    >>> pingpong(82)
    0
    >>> pingpong(100)
    -6
    >>> from construct_check import check
    >>> # ban assignment statements
    >>> check(HW_SOURCE_FILE, 'pingpong',
            ...       ['Assign', 'AnnAssign', 'AugAssign', 'NamedExpr'])
    True
    """
    def direction(n):
        # the base case
        if n < 8:
            return 1
        elif num_eights(n - 1) > 0 or (n - 1) % 8 == 0:
            return -1 * direction(n - 1)
        return direction(n - 1)
    # the base case
    if n < 8:
        return n
    else:
        return pingpong(n - 1) + direction(n)
```

### Q3: Missing Digits

>   Write the recursive function `missing_digits` that takes a number `n` that is sorted in non-decreasing order (for example, `12289` is valid but `15362` and `98764` are not). It returns the number of missing digits in `n`. A missing digit is a number between the first and last digit of `n` of a that is not in `n`.

The solution of this problem is quite similiar to Q1: split this problem by `n % 100` and `n // 10`. The procudure is(take `1248` for an example):arrow_down:

1.   split `1248` into `12` and `48`
2.   check missing digits of `48` and `12` recursively.

The reason why we split this problem by `n % 100` and `n // 10` rather than `n % 10` and `n // 10` is that we need to process 2 adjacent digits each time. `n % 100` will give us last 2 digits.

```python
def missing_digits(n):
    """Given a number a that is in sorted, non-decreasing order,
    return the number of missing digits in n. A missing digit is
    a number between the first and last digit of a that is not in n.
    >>> missing_digits(1248) # 3, 5, 6, 7
    4
    >>> missing_digits(19) # 2, 3, 4, 5, 6, 7, 8
    7
    >>> missing_digits(1122) # No missing numbers
    0
    >>> missing_digits(123456) # No missing numbers
    0
    >>> missing_digits(3558) # 4, 6, 7
    3
    >>> missing_digits(35578) # 4, 6
    2
    >>> missing_digits(12456) # 3
    1
    >>> missing_digits(16789) # 2, 3, 4, 5
    4
    >>> missing_digits(4) # No missing numbers between 4 and 4
    0
    >>> from construct_check import check
    >>> # ban while or for loops
    >>> check(HW_SOURCE_FILE, 'missing_digits', ['While', 'For'])
    True
    """
    if n < 10:
        return 0
    elif n < 100:
        if n // 10 == n % 10:
            return 0
        else:
            return n % 10 - n // 10 - 1
    return missing_digits(n // 10) + missing_digits(n % 100)
```

### Q4: Count coins

>   Given a positive integer `change`, a set of coins makes change for `change` if the sum of the values of the coins is `change`. Here we will use standard US Coin values: 1, 5, 10, 25. For example, the following sets make change for `15`:
>
>   -   15 1-cent coins
>   -   10 1-cent, 1 5-cent coins
>   -   5 1-cent, 2 5-cent coins
>   -   5 1-cent, 1 10-cent coins
>   -   3 5-cent coins
>   -   1 5-cent, 1 10-cent coin
>
>   Thus, there are 6 ways to make change for `15`. Write a **recursive** function `count_coins` that takes a positive integer `change` and returns the number of ways to make change for `change` using coins.
>
>   You can use either of the functions given to you:
>
>   -   `ascending_coin` will return the next larger coin denomination from the input, i.e. `ascending_coin(5)` is `10`.
>   -   `descending_coin` will return the next smaller coin denomination from the input, i.e. `descending_coin(5)` is `1`.
>
>   There are two main ways in which you can approach this problem. One way uses `ascending_coin`, and another uses `descending_coin`.
>
>   **Important:** Use recursion; the tests will fail if you use loops.
>
>   >   **Hint:** Refer the [implementation](http://composingprograms.com/pages/17-recursive-functions.html#example-partitions) of `count_partitions` for an example of how to count the ways to sum up to a final value with smaller parts. If you need to keep track of more than one value across recursive calls, consider writing a helper function.



Similiar to this [implementation](http://composingprograms.com/pages/17-recursive-functions.html#example-partitions), we can slove this problem use tree-recursive function. For every coin value, we can pick or drop. So we can write code like this:arrow_down:

```python
def count_coins(change):
    """Return the number of ways to make change using coins of value of 1, 5, 10, 25.
    >>> count_coins(15)
    6
    >>> count_coins(10)
    4
    >>> count_coins(20)
    9
    >>> count_coins(100) # How many ways to make change for a dollar?
    242
    >>> count_coins(200)
    1463
    >>> from construct_check import check
    >>> # ban iteration
    >>> check(HW_SOURCE_FILE, 'count_coins', ['While', 'For'])
    True
    """
    def helper(coin, n):
        if not coin:
            return 0
        if coin == n:       # 1 coin for n
            return 1
        if coin > n:        # the coin value is too large
            return 0
        pickcoin = helper(coin, n - coin)
        dropcoin = helper(ascending_coin(coin), n)
        return pickcoin + dropcoin
    return helper(1, change)
```












