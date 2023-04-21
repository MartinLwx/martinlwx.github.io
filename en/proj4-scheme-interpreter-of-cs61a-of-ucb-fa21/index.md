# Solution of Proj4. Scheme Interpreter of CS61A (2021-Fall)


## Intro

Recently, I am reading a book called [*Crafting interpreters*](https://craftinginterpreters.com/) written by Robert Nystrom. In the original book, a Tree-walker interpreter jlox was implemented in Java. And I am trying to rewrite in Python - [pylox](https://github.com/MartinLwx/pylox). I highly recommend it👍. At this moment, I suddenly remembered that there were a few small issues with the Scheme interpreter for CS61A that I had not resolved after finishing it a year ago, which kept it in an unfinished state. So today I opened the project and intended to run through it from beginning to end and talk about the ideas.


> Note: I only quote **part** of the original problem description. To make the post more compact, I also omit the irrelevant code :)

## Part 1. The Evaluator
### Problem 1
> Implement the `define` and `lookup` methods of the Frame class...`bindings` is a dictionary representing the bindings in the frame...`parent` is the parent Frame instance...The *environment* for a `Frame` instance consists of that frame, its parent frame, and all its ancestor frames, including the Global Frame.

Problem 1 is trivial in my opinion. To implement the `define` function, you just need to save the `{ symbol: value }` in `self.bindings`. As for the `lookup` function, we can write a iterative solution or recursive one. The iterative solution is more intuitive though.

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

The full problem description elaborates on the procedure of implementing `scheme_apply` function. Notice that you should distinguish `nil` and `None`. This subtle bug may cause you other correct solutions fail to pass the test suite.


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

Is' quite straightforward to implement `scheme_eval`. The main obstacle in the way may comes from the requirement that we need to pass one-argument function to `rest.map`. However, the `scheme_eval` has two arguments(let's ignore the optional part). To make the `scheme_eval` a one-argument function, we may write a simple `lambda` to wrapper it, or we can use the `partial` function in `functools` packages to *fix* arguments for a specefic function. I choose to use the latter solution.

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

In problem 1, we have implemented the `define` method, which can bind value to a symbol. All we need to do is pass the correct arguments to this method. To access the `some_val` part in `(define a some_val)`, we can use the `.rest.first`

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

`validate_form(expressions, 1, 1)` ensures that the `expressions` is `'...`. And we just need to return it.

```python
def do_quote_form(expressions, env):
    validate_form(expressions, 1, 1)
    return expressions.first
```

## Part 2. Procedures
### Problem 6
> Change the `eval_all` function in `scheme_eval_apply.py` (which is called from `do_begin_form` in `scheme_forms.py`) to complete the implementation of the `begin` special form (spec). A `begin` expression is evaluated by evaluating all sub-expressions in order. The value of the `begin` expression is the value of the final sub-expression.

Let's write the recursive solution:
1. Check if `expressions` is `nil`, and return `None` if it holds
2. Check f `expressions.rest` is `nil`, return the evaluation result of `expressions.first` if it holds, or recursively call `eval_all`

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

By reading the problem description of Problem 6, we know how to construct a `LambdaProcedure`

```python
def do_lambda_form(expressions, env):
    validate_form(expressions, 2)
    formals = expressions.first
    validate_formals(formals)
    return LambdaProcedure(formals, expressions.rest, env)
```

### Problem 8
> This method takes in two arguments: `formals`, which is a Scheme list of symbols, and `vals`, which is a Scheme list of values. It should return a new child frame, binding the formal parameters to the values.

The full problem description is well written and this problem us trivial.

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

In this problem, we can use the `make_child_frame` implemented in Problem 8.

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

Use `do_lambda_form` or just call the constructer of `LambdaProcedure`.


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

The key to this problem is understanding dynamic scoping. We can make a child frame and evaluate the `MuProcedure` and it should work as expected.

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

I write `do_and_form` and `do_or_form` recursively:
- `do_and_form`: the base case is `nil` and we should return `True`. Otherwise, we check each one and return immediately if we found *False*
- `do_or_form`: the base case is `nil` and we should return `False`. Otherwise, we check each one and return immediately if we found *True*

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

Just do as the problem description says.

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

Check each `binding` and use the `Pair` to collection the `names` and `values`.

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

We can implement the `enumerate` procedure recursively by writing a helper function called `helper`:
- base case: the `input` is `nil` and we just return `'()`
- other cases: recursively call the `helper` function. Notice that how the arguments change: `input -> (cdr input)` and `index -> (+ index 1)`

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

A classic interview problem: merge two sorted lists. We just need to compare the *head* of each list and recursively call `merge` in the right arguments form


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




