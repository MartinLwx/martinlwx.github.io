# SICP 练习 2.27


## 题目
> Modify your `reverse` procedure of exercise 2.18 to produce a `deep-reverse` procedure that taks a list as argument and returns as its value the list with its elements reversed and with all sublists deep-reversed as well.
```racket
(define x (list (list 1 2) (list 3 4)))
;; x - ((1 2) (3 4))
(deep-reverse x)
;; the output should be ((4 3) (2 1))
```

## 答案

在前面的练习 2.18 中实现的 `reverse` 忽略了列表中的元素可能仍然是列表的情况。这里要求的实现的 `deep-reverse` 则要求我们**递归式**翻转整个列表。

我认为这题很好体现了如何设计一个递归程序的思想，基本上弄明白下面几点函数就不会出错
- 递归的 base case 是什么？对于列表来说，空列表很自然就是 base case，此时返回 `'()` 即可
- 接下来如何细分情况来递归调用？
    - 在细分情况之前，应该记住 `car` 和 `cdr` 的特点，**`(cdr l)` 总是返回列表，`(car l)` 总是返回第一个元素（但不保证是否为原子项，即第一个元素也可能为列表）**
    - `deep-reverse` 有什么不变量（invariants）？根据题目所说，**它的参数总是为列表**
    - 根据前面两点，我们已经不难得出细分情况的标准——即**检查列表的第一个元素是否为原子项**

代码如下
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

顺便附上一些测试用例
```Racket
(deep-reverse '(2 3))
(deep-reverse '((2 3)))
(deep-reverse '((2 3) 1))
(deep-reverse '(5 (2 3) 1))
(deep-reverse '((4 2) (2 3) 1))
(deep-reverse '(5 (2 3) (5 2)))
```

---

🚧 完整 SICP 练习题解仍在[施工中](https://github.com/MartinLwx/SICP-Exercise-in-Racket)

