# SICP Exercise 1.34


## Question
> Suppose we define the procedure `f`. What happens if we (perversely) ask the interpreter to evaluate the combination `(f f)`?
```racket
(define (square x)
  (* x x))

(define (f g)
  (g 2))
```
Then we have

```racket
(f square)                          ;; 4

(f (lambda (z) (* z (+ z 1))))      ;; 6 = 2 * 3
```


## Answer

Recall what the applicative-order evaluation says: **We need to evaluate all arguments and then we apply procedure on these arguments**.

The first step: we evalute the **argument** `f` in `(f f)`.
```racket
;; replace g with f
;; (define (f g)
;;   (g 2))

(f 2)
```

The second step: we evaluate the *argument* `2` in `(f 2)`. **Note that the `2` here is regarded as a procedure rather than a primitive number.**
```racket
;; replace g with 2
;; (define (f g)
;;   (g 2))

(2 2)
```

So we finally get the S-expression `(2 2)`. It's regarded as a function/procedure application. That's how we interprete an S-expression :)

However, **the `2` is not a procedure at all**. This explains why Racket says:
```
application: not a procedure;
expected a procedure that can be applied to arguments
given: 2
```

