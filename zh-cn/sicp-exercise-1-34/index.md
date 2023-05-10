# SICP 练习 1.34


## 题目
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


## 答案

回忆之前书上讲的 applicative-order evaluation：**我们需要先计算所有参数的值，然后将这些参数代入计算**

第一步，先计算**参数** `f`，过程如下
```racket
;; replace g with f
;; (define (f g)
;;   (g 2))

(f 2)
```
第二步：计算*参数* 2。**注意这里的 `2` 被看成是参数而不是一个数字**
```racket
;; replace g with 2
;; (define (f g)
;;   (g 2))

(2 2)
```

最后就得到了 `(2 2)` 这个 S 表达式，它被看成是 function/procedure application。这也是求 S 表达式的值的方式 :)

但是，**这里的 `2` 不是一个函数/过程**，这也解释了为什么 Racket 输入了如下信息：
```
application: not a procedure;
expected a procedure that can be applied to arguments
given: 2

```
---

🚧 完整 SICP 练习题解仍在[施工中](https://github.com/MartinLwx/SICP-Exercise-in-Racket)

