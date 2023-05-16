# SICP Exercise 2.27


## Question
> Modify your `reverse` procedure of exercise 2.18 to produce a `deep-reverse` procedure that takes a list as an argument and returns as its value the list with its elements reversed and with all sublists deep-reversed as well.
```racket
(define x (list (list 1 2) (list 3 4)))
;; x - ((1 2) (3 4))
(deep-reverse x)
;; the output should be ((4 3) (2 1))
```

## Answer

In the previous Exercise 2.18, we ignore the fact that the elements of a list may still be lists too. And this exercise requires us to reverse the entire list **recursively**.

To design a recursive algorithm for a specific problem, we only need to figure out two things:
- What is the base case? As the problem description says, the argument of `deep-reverse` is a list. Usually, the base case of a list structure is an empty list - `'()`
- How to recursively related the subproblems?
    - Fact 1: **`(cdr l)` always returns a list, and `(car l)` always return the 1st element of a list(the 1st element may be a list too)**
    - Fact 2: The invariant of `deep-reverse` - **its argument is a list**
    - Based on the two points, we know the criteria for dividing the problems - **check whether the 1st element of the list is an atomic item**

The code should look like
```Racket
;; invariants: the argument of deep-reverse are always a list
(define (deep-reverse l)
  (cond ((null? l) '())     ;; base case
        (else (let ([remains (deep-reverse (cdr l))])
                (if (pair? (car l))
                    ;; the arguments of append procedure should be lists too
                    (append remains (list (deep-reverse (car l))))
                    (append remains (list (car l))))))))
```

There are some test cases I wrote
```Racket
(deep-reverse '(2 3))
(deep-reverse '((2 3)))
(deep-reverse '((2 3) 1))
(deep-reverse '(5 (2 3) 1))
(deep-reverse '((4 2) (2 3) 1))
(deep-reverse '(5 (2 3) (5 2)))
```

---

🚧 The complete SICP exercise solution is still a work in progress. Please refers to [here](https://github.com/MartinLwx/SICP-Exercise-in-Racket)


