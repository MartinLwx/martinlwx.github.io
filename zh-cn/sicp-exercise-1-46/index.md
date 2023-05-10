# SICP 练习 1.46


## Question
> Several of the numerical methods described in this chapter are instances of an extremely general computational strategy known as *iterative improvement*. Iterative improvement says that, to compute something, we start with an initial guess for the answer, test if the guess is good enough, and otherwise improve the guess and continue the process using the improved guess as the new guess.  Write a procedure `iterative-improve` that takes two procedures as arguments: a method for telling whether a guess is good enough and a method for improving a guess. `Iterative-improve` should return as its value a procedure that takes a guess as argument and keeps improving the guess until it is good enough. Rewrite the sqrt procedure of section 1.1.7 and the fixed-point procedure of section 1.3.3 in terms of iterative-improve.

## Answer

让我们总结一下题目要让我们做什么：
1. 写一个函数，函数名为 `iterative-improve`，它包含了 2 个参数
    1. 第一个参数是函数，用于判断**猜测**是否足够好
    2. 第二个参数仍然是一个函数，用于改进猜测
2. 用 `iterative-improve` 重写 `fixed-point` 和 `sqrt`

如果你一路做完了 SICP 第一章节的练习，那么你可以很轻松写出如下的代码:
```racket
;; test: test if a guess is good enough
;; improve: how to improve a guess
(define (iterative-improve test improve)
  (lambda (guess)
    (if (test guess)
        guess
        ...)))
```

在函数里面用 `lambda` 返回另外一个函数很方便，但是这一道题要这么用有点“绕”，主要的困难来自于：我们要在 `...` 的部分放什么？放 `((iterative-improve good-enough? improve) (improve guess))` 是可以的，但是我感觉这样写并不是很直观

这样的场景在写递归函数的时候其实经常出现，这种时候我们可以在主函数里面定义一个辅助函数，最后在主函数里面返回这个辅助函数即可。代码如下：

```racket
(define (iterative-improve test improve)
  (define (helper guess)
    (if (test guess)
        guess
        (helper (improve guess))))
  helper)
```

写完了 `iterative-improve` 之后，要重写 `fixed-point` 和 `sqrt` 函数是件很容易的事情。我们只需要定义好对应的 `test` 和 `improve` 是什么然后调用 `iterative-improve`
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

🚧 完整 SICP 练习题解仍在[施工中](https://github.com/MartinLwx/SICP-Exercise-in-Racket)

