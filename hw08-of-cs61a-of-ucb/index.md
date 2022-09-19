# Hw08 of CS61A of UCB(2021-Fall)


### Q1: My Filter

>   Write a procedure `my-filter`, which takes a predicate `func` and a list `lst`, and returns a new list containing only elements of the list that satisfy the predicate. The output should contain the elements in the same order that they appeared in the original list.
>
>   **Note:** Make sure that you are not just calling the built-in `filter` function in Scheme - we are asking you to re-implement this!

We need to implement a function that contains elements which are qualified `(func e)` = `#t` in the list.



The base case: an empty list. Otherwise, we check if `(car lst)` can be kept.

```scheme
(define (my-filter func lst)
  (cond ((null? lst) '())
        ((func (car lst))
           (cons (car lst) (my-filter func (cdr lst))))
        (else (my-filter func (cdr lst))))
)
```

### Q2: Interleave

>   Implement the function `interleave`, which takes a two lists `s1` and `s2` as arguments. `interleave` should return a new list that interleaves the elements of the two lists. (In other words, the resulting list should contain elements alternating between `s1` and `s2`.)
>
>   If one of the input lists to `interleave` is shorter than the other, then `interleave` should alternate elements from both lists until one list has no more elements, and then the remaining elements from the longer list should be added to the end of the new list.

It can be solved recursively.



The base case: both lists are empty. We need to return `nil`



Notion: the first parameter of interleave - `first_param`, the second parameter of interleave - `second_param`

We **always** add `(car fist_param)` to our result. And then we will recursively call `(interleave second_param (cdr first_param))`. Apparently, we will add the first element of `second_param` to our result. Repeat this procedure we will get the correct answer. What if the first list is empty? We call `(interleave second_param first_param)` instead :hugs:

```scheme
(define (interleave s1 s2)
  (cond ((and (null? s1) (null? s2)) nil)                ; base case: return nil if both are empty
        ((null? s1) (interleave s2 s1))                  ; change the positions of s1 and s2
        (else (cons (car s1) (interleave s2 (cdr s1))))) ; we always insert (car s1) to the result :)
)
```

### Q3: Accumulate

>   Fill in the definition for the procedure `accumulate`, which merges the first `n` natural numbers (ie. 1 to n, inclusive) according to the following parameters:
>
>   1.  `merger`: a function of two arguments
>   2.  `start`: a number with which we start merging with
>   3.  `n`: the number of natural numbers to merge
>   4.  `term`: a function of one argument that computes the *n*th term of a sequence
>
>   For example, we can find the product of all the numbers from 1 to 5 by using the multiplication operator as the `merger`, and starting our product at 1:

We need to calculate the "sum" of the first `n` natural numbers(`[1, n]`). The definitin of "sum" is defined by `merger`. We also have `term` to compute the ith term of a sequence.



We can't use `while` loop in scheme(If you don't have implement this by yourself). So we need a recursive solution. The base case: `n = 1`, we `merge` 1 and `start`. Otherwise, we will call `(accumulate merger start (- n 1) term)` :)

```scheme
(define (accumulate merger start n term)
  (cond ((= n 1) (merger (term n) start))           ; base case: n = 1
        (else (merger (term n) (accumulate merger start (- n 1) term))))
)
```

### Q4: No Repeats

>   Implement `no-repeats`, which takes a list of numbers `lst` as input and returns a list that has all of the unique elements of `lst` in the order that they first appear, but no repeats. For example, `(no-repeats (list 5 4 5 4 2 2))`evaluates to `(5 4 2)`.
>
>   >   **Hint:** How can you make the first time you see an element in the input list be the first and only time you see the element in the resulting list you return?
>
>   >   **Hint:** You may find it helpful to use the `my-filter` procedure with a helper `lambda` function to use as a filter. To test if two numbers are equal, use the `=` procedure. To test if two numbers are not equal, use the `not`procedure in combination with `=.`

The base case: an empty list. We will return `nil`. Otherwise, we need to delete all elements that are equal to `(car lst)` in `(cdr lst)`, which can implemented by the `my-filter` procedure in Q1.



(ps. I have been looking for bugs for a long time because of an extra set of parentheses. :cry:)

```scheme
(define (no-repeats lst)
  (cond ((null? lst) nil)
        (else (cons (car lst) 
                    (no-repeats 
                       ; choose the elements that are not equal to `car lst`
                       (my-filter (lambda (x) (not (= x (car lst)))) (cdr lst))))))
)
```


