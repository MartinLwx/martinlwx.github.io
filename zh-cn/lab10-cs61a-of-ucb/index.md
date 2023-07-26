# Lab10 - CS61A of UCB(2021-Fall)


## Q2: Over or Under

>   Define a procedure `over-or-under` which takes in a number `num1` and a number `num2` and returns the following:
>
>   -   -1 if `num1` is less than `num2`
>   -   0 if `num1` is equal to `num2`
>   -   1 if `num1` is greater than `num2`
>
>   >   Challenge: Implement this in 2 different ways using `if` and `cond`!
>
>   ```
>   (define (over-or-under num1 num2)
>     'YOUR-CODE-HERE
>   )
>   ```

代码其实本身不难, 主要是适应 scheme 语言的写法, 条件分支有两种写法:

1.   `(if <predicate> <consequent> <alternative>)`
2.   `(cond (<condition> <consequent>) ...)`

```scheme
(define (over-or-under num1 num2) 
    (if (< num1 num2) 
            (print -1))
    (if (= num1 num2)
            (print 0))
    (if (> num1 num2)
            (print 1))
)

(define (over-or-under num1 num2)
  (cond ( (< num1 num2) (print -1) )
        ( (= num1 num2) (print 0)  )
        ( (> num1 num2) (print 1)  ))
)
```

### Q3: Make Adder

>   Write the procedure `make-adder` which takes in an initial number, `num`, and then returns a procedure. This returned procedure takes in a number `inc` and returns the result of `num + inc`.
>
>   >   *Hint*: To return a procedure, you can either return a `lambda` expression or `define` another nested procedure. Remember that Scheme will automatically return the last clause in your procedure.
>   >
>   >   You can find documentation on the syntax of `lambda` expressions in [the 61A scheme specification!](https://cs61a.org/articles/scheme-spec/#lambda)

实现高阶函数的功能, 依旧是锻炼 scheme 语言的掌握程度的. 题目都是之前课上讲过的. 这里我用匿名函数来实现

```scheme
(define (make-adder num)
  (lambda (inc) (+ num inc))
)
```

### Q4: Compose

>   Write the procedure `composed`, which takes in procedures `f` and `g` and outputs a new procedure. This new procedure takes in a number `x` and outputs the result of calling `f` on `g` of `x`.

用 scheme 语言实现符合数学中的复合函数, 也就是高阶函数. 这里同样可以用 lambda 函数

```scheme
(define (composed f g)
  (lambda (x) (f (g x) ) )
)
```

### Q5: Make a List

>   In this problem you will create the list with the following box-and-pointer diagram:
>
>   ![linked list](https://inst.eecs.berkeley.edu/~cs61a/fa21/lab/lab10/assets/list2.png)
>
>   >   Challenge: try to create this list in multiple ways, and using multiple list constructors!要求 

题目要求我们按照给定的链表结构来生成对应的链表. 主要考察的是对 scheme 语言中 list 的掌握. 可以有多种实现方式

1.   `cons` 的方式, 这个方式很容易眼花, 最好是写完之后在[这里](https://code.cs61a.org) 验证一下. 这里我真的写得头有点晕 :cry:
2.   `list` 的方式, 这个代码会比较短, 注意我们每次在调用 `(list ...)` 相当于在链表中多创建了一个方向

```scheme
(define lst 
  (cons (cons 1 nil)
        (cons 2 (cons (cons 3 (cons 4 nil))
                      (cons 5 nil))))
)

(define lst 
  (list (list 1)
        2 (list 3 4)
        5)
)
```

### Q6: Remove

>   Implement a procedure `remove` that takes in a list and returns a new list with *all* instances of `item` removed from `lst`. You may assume the list will only consist of numbers and will not have nested lists.
>
>   >   *Hint*: You might find the built-in `filter` procedure useful (though it is definitely possible to complete this question without it).
>   >
>   >   You can find information about how to use `filter` in [the 61A Scheme builtin specification](https://cs61a.org/articles/scheme-builtins/#pair-and-list-manipulation)!

这一题就是要我们在 scheme 的列表中移除掉值等于 `item` 的元素然后返回新的这个列表. 其实 scheme 的列表也就是链表. 所以这一题等效于我们要在链表中移除指定值的元素. 显然, 这可以用递归来解决! 而且这一道题说没有嵌套列表的情况存在, 这一道题就更简单了 !



显然 base case 就是链表为空的情况, 我们直接返回空. 否则:

1.   当前节点的值 = `item`, 我们删除它, 递归处理子链表
2.   当前节点的值 != `item`, 我们保留它, 递归处理子链表

```scheme
(define (remove item lst)
  (cond ( (null? lst) '() )             ; base case
        ( (= item (car lst)) (remove item (cdr lst)))           ; exclude item
        ( else (cons (car lst) (remove item (cdr lst)))))       ; include item
)
```

