# Hw01 of CS61A of UCB(2021-Fall)


## hw01. Control

---

### Q1. Welcome Forms

Skip :)

### Q2. A Plus Abs B

> Fill in the blanks in the following function for adding `a` to the absolute value of `b`, without calling `abs`. You may **not** modify any of the provided code other than the two blanks.

This problem is easy if we know that we can bind names to functions. In this problem:

- When `b < 0`, `a + abs(b) = a - b`, so we should use `sub`
- When `b > 0`, `a + abs(b) = a + b`, so we should use `add`

```python
def a_plus_abs_b(a, b):
    """Return a+abs(b), but without calling abs.

    >>> a_plus_abs_b(2, 3)
    5
    >>> a_plus_abs_b(2, -3)
    5
    """
    if b < 0:
        f = sub         # b < 0 -> a+abs(b) = a - b, using sub
    else:
        f = add         # b > 0 -> a+abs(b) = a + b, using add
    return f(a, b)
```

### Q3. Two of Three

> Write a function that takes three *positive* numbers as arguments and returns the sum of the squares of the two smallest numbers. **Use only a single line for the body of the function.**

The hint shows that we may need `max` or `min` in this problem. 



At first, I tried to enumerate all possibilities. The smallest numbers may be `x` and `y`, or `x` and `z`, or `y` and `z`. We can just code like this: `return min(x**2 + y**2, x**2 + z**2, y**2 + z**2)`. 



We can also think about the problem from another angle ⬇️

```python
def two_of_three(x, y, z):
    """Return a*a + b*b, where a and b are the two smallest members of the
    positive numbers x, y, and z.

    >>> two_of_three(1, 2, 3)
    5
    >>> two_of_three(5, 3, 1)
    10
    >>> two_of_three(10, 2, 8)
    68
    >>> two_of_three(5, 5, 5)
    50
    """
    # by substracting largest-number * largest-number
    return x**2 + y**2 + z**2 - max(x, y, z)**2 
```

### Q4. Largest Factor

> Write a function that takes an integer `n` that is **greater than 1** and returns the largest integer that is smaller than `n` and evenly divides `n`.

With some basic knowledge of math, we can decompose this problem into two cases

- if `n` is an even number, the answer is `n // 2`

- if `n` is an odd number, the answer can be found by check `n % factor == 0`. the range of factor is `1 ~ n // 2`

```python
def largest_factor(n):
    """Return the largest factor of n that is smaller than n.

    >>> largest_factor(15) # factors are 1, 3, 5
    5
    >>> largest_factor(80) # factors are 1, 2, 4, 5, 8, 10, 16, 20, 40
    40
    >>> largest_factor(13) # factor is 1 since 13 is prime
    1
    """
    if n % 2 == 0:
        return n // 2
    else:
        factors = [i for i in range(n//2, 0, -1) if n % i == 0]      # get all factors
        return factors[0]                                            # the biggest one is the largest one
```

### Q5. If Function Refactor

> In this problem, we should refactor out code to avoid `ZeroDivisionError`

The `ZeroDivisionError` happens when we call function expression. `Python` will try to evaluate the whole expression recursively. 



Although the hw01 says *Your second job is to edit `invert_short` and `change_short` so that they have the same behavior as `invert` and `change` but still have just one line each. You will also need to edit `limited`. You don't need to use `and` or `or` or`if` in `invert`; just pay attention to when the division takes place.*, I tried to solve this problem without editing `limited` functio and use `if` :crying_cat_face:

```python
def limited(x, z, limit):
    """Logic that is common to invert and change."""
    if x != 0:
        return min(z, limit)
    else:
        return limit


def invert_short(x, limit):
    """Return 1/x, but with a limit.

    >>> x = 0.2
    >>> 1/x
    5.0
    >>> invert_short(x, 100)
    5.0
    >>> invert_short(x, 2)    # 2 is smaller than 5
    2

    >>> x = 0
    >>> invert_short(x, 100)  # No error, even though 1/x divides by 0!
    100
    """
    # the ZeroDivisionError happens here when we call limited function
    return limited(x, 1 / x, limit) if x != 0 else limited(x, 0, limit)


def change_short(x, y, limit):
    """Return abs(y - x) as a fraction of x, but with a limit.

    >>> x, y = 2, 5
    >>> abs(y - x) / x
    1.5
    >>> change_short(x, y, 100)
    1.5
    >>> change_short(x, y, 1)    # 1 is smaller than 1.5
    1

    >>> x = 0
    >>> change_short(x, y, 100)  # No error, even though abs(y - x) / x divides by 0!
    100
    """
    # the ZeroDivisionError happens here when we call limited function
    return limited(x, abs(y - x) / x, limit) if x != 0 else limited(x, 0, limit)
```

### Q6.Hailstone

> Douglas Hofstadter's Pulitzer-prize-winning book, *Gödel, Escher, Bach*, poses the following mathematical puzzle.
>
> 1. Pick a positive integer `n` as the start.
> 2. If `n` is even, divide it by 2.
> 3. If `n` is odd, multiply it by 3 and add 1.
> 4. Continue this process until `n` is 1.
>
> The number `n` will travel up and down but eventually end at 1 (at least for all numbers that have ever been tried -- nobody has ever proved that the sequence will terminate). Analogously, a hailstone travels up and down in the atmosphere before eventually landing on earth.
>
> This sequence of values of `n` is often called a Hailstone sequence. Write a function that takes a single argument with formal parameter name `n`, prints out the hailstone sequence starting at `n`, and returns the number of steps in the sequence:

This problem is easy. We just follow the 4 steps and count.

```python
def hailstone(n):
    """Print the hailstone sequence starting at n and return its
    length.

    >>> a = hailstone(10)
    10
    5
    16
    8
    4
    2
    1
    >>> a
    7
    """
    cnt = 0
    while n != 1:
        print(n)
        if n % 2 == 0:
            n //= 2
        else:
            n = n * 3 + 1
        cnt += 1
    print(n)          # print 1
    return cnt + 1    # +1 for `n=1`
```


