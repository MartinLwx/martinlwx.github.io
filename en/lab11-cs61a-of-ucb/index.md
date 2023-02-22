# Lab11 CS61A of UCB(2021-Fall)


## Context

---

### Problem 1

>   >   **Important:** Your code for this part should go in `buffer.py`.
>
>   Your job in this part is to implement the `current` and `pop_first` methods of the `Buffer` class.
>
>   `current` should return the current token of the current line we're on in the `Buffer` instance *without* removing it. If there are no more tokens in the current line, then `current` should move onto the next valid line, and return the **first** token of *this* line. If there are no more tokens left to return from the entire source (we've reached the end of all input lines), then `current` should return `None` (this logic is already provided for you in the `except StopIteration` block).
>
>   If we call `current` multiple times in a row, we should get the same result since calls to `current` won't change what token we're returning.
>
>   >   You may find `self.index` helpful while implementing these functions, but you are not required to reference it in your solution.
>
>   >   **Hint:** What instance attribute can we use to keep track of where we are in the current line?
>
>   >   **Hint:** If we've reached the end of the current line, then `self.more_on_line()` will return `False`. In that case, how do we "reset" our position to the beginning of the next line?
>
>   `pop_first` should return the current token of the `Buffer` instance, and move onto the next potential token (to be returned on the next call to `pop_first`). If there are no more tokens left to return from the entire source (we've reached the end of all input lines), then `pop_first` should return `None`.
>
>   >   **Hint:** Do we need to update anything to move onto the next potential token?

We need to implement two functions in this problem: `current` and `pop_first`



The requirements are listed in the description

```python
class Buffer:
    """A Buffer provides a way of accessing a sequence of tokens across lines.

    Its constructor takes an iterator, called "the source", that returns the
    next line of tokens as a list each time it is queried, or None to indicate
    the end of data.

    The Buffer in effect concatenates the sequences returned from its source
    and then supplies the items from them one at a time through its pop_first()
    method, calling the source for more sequences of items only when needed.

    In addition, Buffer provides a current method to look at the
    next item to be supplied, without sequencing past it.

    The __str__ method prints all tokens read so far, up to the end of the
    current line, and marks the current token with >>.
    """

    def __init__(self, source):
        self.index = 0
        self.source = source
        self.current_line = ()
        self.current()

    def pop_first(self):
        """Remove the next item from self and return it. If self has
        exhausted its source, returns None."""
        current = self.current()
        self.index += 1
        return current

    def current(self):
        """Return the current element, or None if none exists."""
        # if there are any token in current line we don't return 
        while not self.more_on_line():
            self.index = 0
            try:
                self.current_line = next(self.source)
            except StopIteration:
                self.current_line = ()
                return None
        return self.current_line[self.index]

    def more_on_line(self):
        return self.index < len(self.current_line)
```

## Internal Representations

---

### Problem 2

>   >   **Important:** Your code for this part should go in `scheme_reader.py`.
>
>   Your job in this part is to write the parsing functionality, which consists of two mutually recursive functions:`scheme_read` and `read_tail`. Each function takes in a single `src` parameter, which is a `Buffer` instance.
>
>   -   `scheme_read` removes enough tokens from `src` to form a single expression and returns that expression in the correct [internal representation](https://inst.eecs.berkeley.edu/~cs61a/fa21/lab/lab11/#internal-representations).
>   -   `read_tail` expects to read the rest of a list or `Pair`, assuming the open parenthesis of that list or `Pair` has already been removed by `scheme_read`. It will read expressions (and thus remove tokens) until the matching closing parenthesis `)` is seen. This list of expressions is returned as a linked list of `Pair` instances.
>
>   In short, `scheme_read` returns the next single complete expression in the buffer and `read_tail` returns the rest of a list or `Pair` in the buffer. Both functions mutate the buffer, removing the tokens that have already been processed.
>
>   The behavior of both functions depends on the first token currently in `src`. They should be implemented as follows:
>
>   `scheme_read`:
>
>   -   If the current token is the string `"nil"`, return the `nil` object.
>   -   If the current token is `(`, the expression is a pair or list. Call `read_tail` on the rest of `src` and return its result.
>   -   If the current token is `'`, the rest of the buffer should be processed as a `quote` expression. You will implement this portion in the next problem.
>   -   If the next token is not a delimiter, then it must be a primitive expression (i.e. a number, boolean). Return it. **Provided**
>   -   If none of the above cases apply, raise an error. **Provided**
>
>   `read_tail`:
>
>   -   If there are no more tokens, then the list is missing a close parenthesis and we should raise an error. **Provided**
>   -   If the token is `)`, then we've reached the end of the list or pair. **Remove this token from the buffer** and return the `nil` object.
>   -   If none of the above cases apply, the next token is the operator in a combination. For example, `src` could contain `+ 2 3)`. To parse this:
>       1.  `scheme_read` the next complete expression in the buffer.
>       2.  Call `read_tail` to read the rest of the combination until the matching closing parenthesis.
>       3.  Return the results as a `Pair` instance, where the first element is the next complete expression from (1) and the second element is the rest of the combination from (2).

The code for this question is put together with the next question :)

### Problem 3

>   >   **Important:** Your code for this part should go in `scheme_reader.py`.
>
>   Your task in this problem is to complete the implementation of `scheme_read` by allowing the function to now be able to handle quoted expressions.
>
>   In Scheme, quoted expressions such as `'<expr>` are equivalent to `(quote <expr>)`. That means that we need to wrap the expression following `'` (which you can get by recursively calling `scheme_read`) into the `quote` special form, which is a Scheme list (as with all special forms).
>
>   In our representation, a `Pair` represents a Scheme list. You should therefore wrap the expression following `'` in a `Pair`.
>
>   For example, `'bagel`, or `["'", "bagel"]` after being tokenized, should be represented as `Pair('quote', Pair('bagel', nil))`. `'(1 2)` (or `["'", "(", 1, 2, ")"]`) should be represented as `Pair('quote', Pair(Pair(1, Pair(2, nil)), nil))`.

We need to implement the `'` in the scheme language. Actually, the description indicates the way to solve this problem: *which you can get by recursively calling `scheme_read`*. We need to make a new `Pair`, whose first element is `quote`, and recursively call `scheme_reader` to handle the expression.

```python
def scheme_read(src):
    """Read the next expression from SRC, a Buffer of tokens.
    """
    if src.current() is None:
        raise EOFError
    val = src.pop_first()  # Get and remove the first token
    if val == 'nil':
        return nil
    elif val == '(':
        return read_tail(src)
    elif val == "'":
        return Pair('quote', Pair(scheme_read(src), nil))
    elif val not in DELIMITERS:
        return val
    else:
        raise SyntaxError('unexpected token: {0}'.format(val))


def read_tail(src):
    """Return the remainder of a list in SRC, starting before an element or ).
    """
    try:
        if src.current() is None:
            raise SyntaxError('unexpected end of file')
        elif src.current() == ')':
            src.pop_first()
            return nil
        else:
            return Pair(scheme_read(src), read_tail(src))
    except EOFError:
        raise SyntaxError('unexpected end of file')
```


