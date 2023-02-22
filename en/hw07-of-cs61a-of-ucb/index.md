# Hw07 of CS61A of UCB(2021-Fall)


### Q1: Thane of Cadr

>   Define the procedures `cadr` and `caddr`, which return the second and third elements of a list, respectively. If you would like a quick refresher on scheme syntax consider looking at [Lab 10 Scheme Refresher](https://inst.eecs.berkeley.edu/~cs61a/fa21/lab/lab10/#scheme).

We need to implement the function `c???r`. To have a better understanding of this notation, you should look from back to the front in `???`. For example, the `cadr` function will call `cdr` then call `car` on the input.

```scheme
(define (cadr s)
  (car (cdr s))
)

(define (caddr s)
  (car (cdr (cdr s)))
)
```

### Q2: Ordered

>   Implement a procedure called `ordered?`, which takes a list of numbers and returns `True` if the numbers are in nondescending order, and `False` otherwise. Numbers are considered nondescending if each subsequent number is either larger or equal to the previous, that is:
>
>   ```
>   1 2 3 3 4
>   ```
>
>   Is nondescending, but:
>
>   ```
>   1 2 3 3 2
>   ```
>
>   Is not.
>
>   >   *Hint*: The built-in `null?` function returns whether its argument is `nil`.

We need to solve this problem recursively:

1.   base case: an empty list or a list with a size 1
2.   Recursive decomposition: the current node is less than or equal to the first node of the sub-linked list, and the sub-linked list must also be in non-descending order

```scheme
(define (ordered? s)
  (cond ( (null? s) #t)
        ( (null? (cdr s)) #t)
        ( else (and (<= (car s) (cadr s))
               (ordered? (cdr s)))))
)
```

### Q3: Pow

>   Implement a procedure `pow` for raising the number `base` to the power of a nonnegative integer `exp` for which the number of operations grows logarithmically, rather than linearly (the number of recursive calls should be much smaller than the input `exp`). For example, for `(pow 2 32)` should take 5 recursive calls rather than 32 recursive calls. Similarly, `(pow 2 64)` should take 6 recursive calls.
>
>   >   *Hint:* Consider the following observations:
>   >
>   >   1.  $x^{2y} = (x^y)2$
>   >   2.  $x^{2y+1} = x(x^y)2$
>   >
>   >   For example we see that 232 is (216)2, 216 is (28)2, etc. You may use the built-in predicates `even?` and `odd?`. Scheme doesn't support iteration in the same manner as Python, so consider another way to solve this problem.

The procedure of calculating this `(pow base exp)` has been provided in the description.

```scheme
(define (pow base exp)
  (cond ( (= exp 1) base )                                    ; base^1 = base
        ( (= exp 0) 1)                                        ; base^0 = 1
        ( (= 0 (modulo exp 2))
            (begin
              (define tmp (pow base (quotient exp 2)))        ; store the value temporarily
              (* tmp tmp)
            )
        )
        ( (= 1 (modulo exp 2))
            (begin
              (define tmp (pow base (quotient exp 2)))        ; store the value temporarily
              (* tmp tmp base)
            )
        )
  )
)
```


