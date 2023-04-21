# CS61A 的项目四之 Scheme 解释器实现 (2021-Fall)


## 引言

最近正在跟着[《Crafting interpreter》](https://craftinginterpreters.com/)这本书写解释器，原本书里面用 Java 实现了一个 Tree-walker 解释器 jlox，我正在用 Python 重写一遍，称为 [pylox](https://github.com/MartinLwx/pylox)。看了这本书感觉对解释器的理解越来越深刻了，很推荐👍。此时的我突然想起来之前看完的 CS61A 的 Scheme 解释器还有几个小问题没有解决，导致它一直是未完成的状态，于是今天我打开了这个项目，打算从头到尾捋一遍，讲讲思路。

> 注：Scheme 解释器这个项目比较大，所以**我只复制了题目描述中的重要部分**，完整的描述还是要回去看项目主页。同时**代码只显示核心的部分**。

## Part 1. The Evaluator
### Problem 1
> Implement the `define` and `lookup` methods of the Frame class...`bindings` is a dictionary representing the bindings in the frame...`parent` is the parent Frame instance...The *environment* for a `Frame` instance consists of that frame, its parent frame, and all its ancestor frames, including the Global Frame.

`define` 函数很简单，就是一个字符串（`symbol`）到 Scheme 值（`value`）的映射，参数都给你写好了

`lookup` 函数的具体执行过程在本来的题目描述中已经列出来了，照着做就行，**迭代和递归的解法都可以**，我感觉迭代的解法会比较简单
```python
...
def define(self, symbol, value):
    """Define Scheme SYMBOL to have VALUE."""
    self.bindings[symbol] = value

def lookup(self, symbol):
    """Return the value bound to SYMBOL. Errors if SYMBOL is not found."""
    # Case 1. we check if the symbol is in the current frame
    if symbol in self.bindings.keys():
        return self.bindings[symbol]
    else:
        # Case 2. we check the parent of the current frame repreatly
        pos = self.parent
        while pos is not None:
            if symbol in pos.bindings.keys():
                return pos.bindings[symbol]
            pos = pos.parent
    # Case 3. we can't find the symbol
    raise SchemeError("unknown identifier: {0}".format(symbol))
...
```
### Problem 2
> To be able to call built-in procedures, such as `+`, you need to complete the `BuiltinProcedure` case within the `scheme_apply` function in `scheme_eval_apply.py`. Built-in procedures are applied by calling a corresponding Python function that implements the procedure.

跟着题目的要求做即可，没有什么难度。**值得一提的是要和 `nil` 判断而不是和 `None` 判断**，不然你可能在第三题一直得到 "incorrect number of arguments..."，我发现我之前没有做出来就是这里没写好

```python
def scheme_apply(procedure, args, env):
    ...
    if isinstance(procedure, BuiltinProcedure):
        # Convert the Scheme list to a Python list of arguments
        args_list = []
        pos = args
        while pos is not nil:
            if pos.first is not nil:
                args_list.append(pos.first)
            else:
                args_list.append(nil)
            pos = pos.rest
        # Add the current environment if procedure.expect_env == True
        if procedure.expect_env:
            args_list.append(env)
        # Call procedure.py_func on all arguments
        try:
            return procedure.py_func(*args_list)
        except TypeError as e:
            raise SchemeError(f"incorrect number of arguments, {e}")
    ...
```
### Problem 3
> Implement the missing part of `scheme_eval`, which evaluates a call expression...You'll have to recursively call `scheme_eval` in the first two steps...The `map` method of `Pair` returns a new Scheme list constructed by applying a *one-argument function* to every item in a Scheme list...Important: do not mutate the passed-in `expr`. That would change a program as it's being evaluated, creating strange and incorrect effects.

这一道题也很直白，可能的一个难点是，`rest.map` 的参数是一个 “one-argument function”，也就是只接受一个参数，但是题目提供的 `scheme_eval` 有 2 个参数。所以**需要对函数进行转化**，当然这里可以写一个 lambda 表达式包装一下 `scheme_eval`。**我选择用 `functools` 包提供的 `partial` 函数，它的用途就是绑定函数的部分参数并返回一个新的函数**。第一次见到 `partial` 这种用法还是在函数式编程语言里面，不少函数式编程语言都是原生就支持这个功能。

```python
def scheme_eval(expr, env, _=None):  # Optional third argument is ignored
    ...
    else:
        # Evaluate the operator(first argument)
        operator = scheme_eval(first, env)
        validate_procedure(operator)
        # Evaluate all of the operands(other arguments)
        from functools import partial
        operands = rest.map(partial(scheme_eval, env=env))

        return scheme_apply(operator, operands, env)
```
### Problem 4
> The type of the first operand tells us what is being defined...implement just the first part, which evaluates the second operand to obtain a value and binds the first operand, a symbol, to that value. Then, `do_define_form` returns the symbol that was bound.

这里只要求实现 `define` 的第一个功能——绑定变量，具体绑定的方式其实我们已经在 Problem 1 里面实现好了，就是 `Frame` 类的 `define` 方法，因此绑定变量只要调用 `env.define` 即可。

根据 `define` 绑定变量的写法: `(define a some_val)`，可以通过 `.rest.first` 拿到对应的 `some_val` 用 `scheme_eval` 进行估值

```python
def do_define_form(expressions, env):
    ...
    if scheme_symbolp(signature):
        # assigning a name to a value e.g. (define x (+ 1 2))
        validate_form(
            expressions, 2, 2
        )  # Checks that expressions is a list of length exactly 2
        env.define(signature, scheme_eval(expressions.rest.first, env))
        return signature
    ...
```
### Problem 5
> Implement the `do_quote_form` function in `scheme_forms.py` so that it simply returns the unevaluated operand of the `(quote ...)` expression.

`validate_form(expressions, 1, 1)` 确保输入长度为 1，即检查是否为 `'...` 形式，我们只需要直接返回即可

```python
def do_quote_form(expressions, env):
    validate_form(expressions, 1, 1)
    return expressions.first
```

## Part 2. Procedures
### Problem 6
> Change the `eval_all` function in `scheme_eval_apply.py` (which is called from `do_begin_form` in `scheme_forms.py`) to complete the implementation of the `begin` special form (spec). A `begin` expression is evaluated by evaluating all sub-expressions in order. The value of the `begin` expression is the value of the final sub-expression.

其实这是一个递归的过程：
1. 先检查 `expressions` 是否为 `nil`，是的话返回 `None` 表示没有定义
2. 继续检查 `expressions.rest` 是否为 `nil`，是的话返回 `expressions.first` 的评估结果，否则继续递归调用

```python
def eval_all(expressions, env):
    if expressions is nil:
        return None
    res = scheme_eval(expressions.first, env)
    if expressions.rest is nil:
        return res
    else:
        return eval_all(expressions.rest, env)
```

### Problem 7
> Implement the `do_lambda_form` function (spec), which creates and returns a `LambdaProcedure` instance

在 Problem 6 里面已经说了 `LambdaProcedure` 的结构，调用一下它的构造函数就行

```python
def do_lambda_form(expressions, env):
    validate_form(expressions, 2)
    formals = expressions.first
    validate_formals(formals)
    return LambdaProcedure(formals, expressions.rest, env)
```

### Problem 8
> This method takes in two arguments: `formals`, which is a Scheme list of symbols, and `vals`, which is a Scheme list of values. It should return a new child frame, binding the formal parameters to the values.

题目的步骤已经够详细了，这里就不展开了

```python
def make_child_frame(self, formals, vals):
    if len(formals) != len(vals):
        raise SchemeError("Incorrect number of arguments to function call")
    sub_frame = Frame(self)
    # iterate
    pos1, pos2 = formals, vals
    while pos1 is not nil:
        key, value = pos1.first, pos2.first
        sub_frame.define(key, value)
        pos1, pos2 = pos1.rest, pos2.rest
    return sub_frame
```

### Problem 9
> You should first create a new `Frame` instance using the `make_child_frame` method of the appropriate parent frame, binding formal parameters to argument values. Then, evaluate each of the expressions of the body of the procedure using `eval_all` within this new frame.

这里刚好用了 Problem 8 写的 `make_child_frame` 函数

```python
def scheme_apply(procedure, args, env):
    ...
    elif isinstance(procedure, LambdaProcedure):
        child_frame = procedure.env.make_child_frame(procedure.formals, args)
        return eval_all(procedure.body, child_frame)
    ...
```

### Problem 10
> Modify the `do_define_form` function in `scheme_forms.py` so that it correctly handles `define (...) ...)` expressions

和之前的相比，差别主要在 `env.define` 的第 2 个参数，用前面写好的 `do_lambda_form` 或者直接调用 `LambdaProcedure` 也可以

```python
def do_define_form(expressions, env):
    ...
    elif isinstance(signature, Pair) and scheme_symbolp(signature.first):
        # defining a named procedure e.g. (define (f x y) (+ x y))

        # the signature is (f x y)
        formals = signature.rest  # (x y)
        validate_formals(formals)

        # now we need to parse (+ x y)
        env.define(signature.first, LambdaProcedure(formals, expressions.rest, env))
        return signature.first  # f
    ...
```

### Problem 11
> Implement `do_mu_form` in `scheme_forms.py` to evaluate the `mu` special form. A `mu` expression evaluates to a `MuProcedure`. Most of the `MuProcedure` class (defined in `scheme_classes.py`) has been provided for you.

`MuProcedure` 的特别之处在于 dynamic scoping，参数的值取决于调用的时候环境里面有什么。`scheme_apply` 函数的参数 `env` 就表示了当前环境，我们只需要构造一个 child frame 并在里面评估 `MuProcedure` 即可

```python
def scheme_apply(procedure, args, env):
    ...
    elif isinstance(procedure, MuProcedure):
        child_frame = env.make_child_frame(procedure.formals, args)
        return eval_all(procedure.body, child_frame)
    ...
def do_mu_form(expressions, env):
    validate_form(expressions, 2)
    formals = expressions.first
    validate_formals(formals)
    return MuProcedure(formals, expressions.rest)
```

## Part 3. Special Forms
### Problem 12
> Implement `do_and_form` and `do_or_form` so that `and` and `or` expressions are evaluated correctly. The logical forms `and` and `or` are *short-circuiting*

`do_and_form` 和 `do_or_form` 都可以用递归写：
- `do_and_form`：base case 为 `nil` 此时返回为 `True`，从头到尾检查，一旦发现不为 `True` 的就立刻返回
- `do_or_form`：base case 为 `nil` 此时返回 `False`，从头到尾检查，一旦发现为 `True` 的就立刻返回

```python
def do_and_form(expressions, env):
    # base case: (and)
    if expressions is nil:
        return True
    front = scheme_eval(expressions.first, env)
    if is_scheme_true(front):
        if expressions.rest is nil:
            return front
        else:
            return do_and_form(expressions.rest, env)
    else:
        return front

def do_or_form(expressions, env):
    # base case: (or)
    if expressions is nil:
        return False
    front = scheme_eval(expressions.first, env)
    if is_scheme_false(front):
        if expressions.rest is nil:
            return front
        else:
            return do_or_form(expressions.rest, env)
    else:
        return front
```
### Problem 13
> Fill in the missing parts of `do_cond_form` so that it correctly implements `cond`, returning the value of the first result sub-expression corresponding to a true predicate, or the result sub-expression corresponding to `else`.

按照题目的意思来就行


```python
def do_cond_form(expressions, env):
        ...
        if is_scheme_true(test):
            # no sub-expression
            if clause.rest is nil:
                return test
            return eval_all(clause.rest, env)
        ...
```

### Problem 14
> Implement `make_let_frame` in `scheme_forms.py`, which returns a child frame of `env` that binds the symbol in each element of `bindings` to the value of its corresponding expression. The `bindings` Scheme list contains pairs that each contain a symbol and a corresponding expression.

遍历每一个 `binding`，收集参数名和值到 `names` 和 `values` 就行

```python
def make_let_frame(bindings, env):
    if not scheme_listp(bindings):
        raise SchemeError("bad bindings list in let form")
    names = values = nil

    # bingding: (<name> <expression>)
    # bingdings: ( (<name1> <expression1>) (<name2> <expression2>) ...)
    pos = bindings
    while pos is not nil:
        front = pos.first  # i.e the first binding
        validate_form(front, 2, 2)  # verify the structure is (<name> <expression>)
        names = Pair(front.first, names)
        values = Pair(eval_all(front.rest, env), values)
        pos = pos.rest
    validate_formals(names)

    return = env.make_child_frame(names, values)
```

### Problem 15
> Implement the `enumerate` procedure, which takes in a list of values and returns a list of two-element lists, where the first element is the index of the value, and the second element is the value itself.

通过递归就可以实现，在下面我实现了一个 `helper` 递归函数，参数是输入 `input` 和索引 `index`：
- base case：输入 `input` 为空，则返回 `'()`
- 其他情况：递归调用，注意参数变化：`input -> (cdr input)` 和 `index -> (+ index 1)`

```scheme
(define (enumerate s)
  (begin
      ;; a helper funtion
      (define (helper input index) 
        (cond ((null? input) '())             ;; base case: return () if it is nil
              (else (cons (cons index (cons (car input) nil))
                          (helper (cdr input) (+ index 1))))))   ;; recursive call
      (helper s 0))
  )
```

### Problem 16
> Implement the `merge` procedure, which takes in a comparator function `inorder?` and two lists that are sorted, and combines the two lists into a single sorted list. A comparator defines an ordering by comparing two values and returning a true value if and only if the two values are ordered. Here, sorted means sorted according to the comparator

经典算法：合并 2 个有序列表，每次取出 2 个列表的头个元素，对应下面的 `(car list1) (car list2)`，然后进行比较，根据不同情况进行递归调用


```scheme
(define (merge inorder? list1 list2)
  (cond ((null? list1) list2)           ;; base case: list1 is empty
        ((null? list2) list1)           ;; base case: list2 is empty
        ((inorder? (car list1) (car list2))           
            (cons (car list1) (merge inorder? (cdr list1) list2)))      ;; consume list1
        (else
            (cons (car list2) (merge inorder? list1 (cdr list2)))))     ;; consume list2
  )
```

