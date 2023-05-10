# SICP Exercise 1.46


## Question
> Several of the numerical methods described in this chapter are instances of an extremely general computational strategy known as *iterative improvement*. Iterative improvement says that, to compute something, we start with an initial guess for the answer, test if the guess is good enough, and otherwise improve the guess and continue the process using the improved guess as the new guess.  Write a procedure `iterative-improve` that takes two procedures as arguments: a method for telling whether a guess is good enough and a method for improving a guess. `Iterative-improve` should return as its value a procedure that takes a guess as argument and keeps improving the guess until it is good enough. Rewrite the sqrt procedure of section 1.1.7 and the fixed-point procedure of section 1.3.3 in terms of iterative-improve.

## Answer

Let's summarize what the problem asks us to do:
1. Write a procedure called `iterative-improve` with 2 arguments.
    1. The first argument is **a procedure** that can tell where **a guess** is good enough.
    2. The second argument is **a procedure** that can improve a guess (the input argument).
2. Rewrite `fixed-point` and `sqrt` using `iterative-improve`.

If you walk along the Exercise of Chapter 1, you can quickly write `iterative-improve` like this:
```racket
;; test: test if a guess is good enough
;; improve: how to improve a guess
(define (iterative-improve test improve)
  (lambda (guess)
    (if (test guess)
        guess
        ...)))
```

Using `lambda` to create an anonymous procedure and return is quite convenient. The main difficulty arises from the `...` part, that is, how should we **recursively** improve the guess? We can put `((iterative-improve good-enough? improve) (improve guess))` in the `...` and it will work correctly. However, this may not be very intuitive.

That's where helper functions come in handy. When designing a recursive procedure, we can **define an inner helper function to perform the heavy work** and simply return the helper function in the main body. 

```racket
(define (iterative-improve test improve)
  (define (helper guess)
    (if (test guess)
        guess
        (helper (improve guess))))
  helper)
```
It's easy to rewrite the `fixed-point` and `sqrt` procedures. All we have to do is define the corresponding `test` and `improve` procedures and then call `iterative-improve`.
```racket
(define (average x y)
  (/ (+ x y)
     2))

(define (square x)
  (* x x))

(define (fixed-point f first-guess)
  ;; it's fine to refer a variable in the enclosing scope
  (define (close-enough? v)
    (let ([next (f v)])
      (< (abs (- v next)) 0.00001)))
  ((iterative-improve close-enough? f) first-guess))

(define (sqrt x)
  ;; it's fine to refer a variable in the enclosing scope
  (define (good-enough? v)
    (< (abs (- (square v) x)) 0.001))
  (define (improve guess)
      (average guess (/ x guess)))
  ((iterative-improve good-enough? improve) 1.0)) 
```
 

---

🚧 The complete SICP exercise solution is still a work in progress. Please refers to [here](https://github.com/MartinLwx/SICP-Exercise-in-Racket)


