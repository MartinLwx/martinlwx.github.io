# f-strings in Python 3.6


> This post is originally written in jupyter notebook and then convert to markdown. To get the original notebook files. Please check the [repo](https://github.com/MartinLwx/oh_my_python)

## Intro

String formatting can be regarded as one of the most common activities in daily programming. We often need to output various strings and precisely control their format.

In some outdated Python tutorials, you may see the use of `%` to format strings. However, after Python 3.6, f-strings have become the optimal choice for formatting strings. 

The advantages include:
1. **Expressions can be embedded** in the string literals conveniently
2. **More human-readable**

*The following simple comparison shows the advantages of f-strings in readability*:


```python
name = "Martin"
f"My name is {name}"
```




    'My name is Martin'




```python
name = "Martin"
"My name is %s" % name
```




    'My name is Martin'



We implemented the same string output in different ways.

Without any effort, we can find that the f-strings here are to output the name (this requires you to use meaningful variables names, such as `name` here). On the contrary, it is slightly unnatural to look at the `%` format string. **We first saw it use `%s` as a placeholder, then we need to go to the right to find out which variable it refers to**. 

The example here is too short to show the real power of f-strings. Just trying to imagine that we are reading a long format string written in `%` syntax, we will need to look it back and forth to figure out what it means. Not just about reading the code, if you write a long format string by yourself, you have to check whether the order of the placeholders and variables is consistent, which is undoubtedly a burden. 😫

## Syntax rules

The syntax rules of f-strings are quite simple. I will just borrow the definition from[^1]:

`f'<text>{<expression><optional !s, !r, or !a> <optional:format_specifier> }<text>...'`

In general, it can be summarized as `f"..."` or `f'...'`.

Like regular strings in python, we can choose to use single quotes `''` or double quotes `""` to enclose the content of the string. Triple quotes - `"""..."""` is also available.

As for the other parts, we divide them into several subsections:

### `<text>`

It means the string literals. The simplest usage scenario is a string without any expressions, which makes f-strings behave like regular strings. *like `f"Hello world"`*. ~~It would be an overkill if we just use f-strings to represent regular strings~~

### `{<expression>}` 

Let's consider the simplest form: `{expression}` without any option parts (`<option...>` in aforementioned grammar)

The `{<expression>}` shows how to embed an expression in f-strings: just put it inside {}. Here is a simple example:


```python
left, right = 3, 5
f"{left} + {right} = {left + right}"
```




    '3 + 5 = 8'



Using `{}` as a placeholder naturally leads to a question-what if we want to output `{}` in the string.

The answer is also very simple, we only need to use this format: `{{expression}}`


```python
# 'hello' is an expression too. 
# Note that we need to use different quote syntax inside {}
f"{{'hello'}}"   
```




    "{'hello'}"



It is also worth mentioning that `:` and `!` and `\` **are not allowed in `expression`**

These characters rarely appear in `expression` though. One usage scenario of `\` is to escape quotes. However, we could just use a different type of quote inside the `expression`. As for `:`, a use case would be defining a lambda function in `expression`[^1]. In this situation, we only need to enclose the lambda expression within `()`:


```python
# note that we need to add () around the lambda expression
f"{(lambda x: x + 1)(3)}" 
```




    '4'



### `<option...>`

#### `<optional !s, !r, or !a>`

These are used to perform **type conversion** on `<expression>`
```python
{foo!s}     # it's equal to call str(foo) first
{foo!r}     # similarily, call repr(foo) first
{foo!a}     # similarily，call ascii(foo) first
```

In fact, these three are redundant[^1]. It is just for minimizing the differences with `str.format()`

#### `<optional:format_specifier>`

This part is to define how `expression` should be displayed, *for example, control the precision of decimals, or do string alignment, etc.*. 

Below I will briefly talk about some of them. For more detailed explanations, please refer to[^2]

> Note that this `format_specifier` begins with `:` (this explains why we are not allowed use `:` in `expression`)

The grammar of `format_specifier`:

`[[fill]align][sign][z][#][0][width][,][grouping_option][.precision][type]`

> In grammar, the `[...]` symbol means it is optional

##### `[[fill]align]`

This part is used to pad(specify a character with `fill`) the `expression` and do alignments. The `fill` here can be any character(defaults to a space).

> Note: We need to set minumul width by `[width]` when we declare `[[fill]align]`


The various ways of alignment options:

```python
<        # left-aligned 
>        # right-aligned
^        # centered
=        # only valid for numeric types, pad between sign and digits
```


```python
f"{-1:*^9}"  # set `fill` to *, and set width to 9
```




    '***-1****'




```python
f"{-1:*>9}"  # set `fill` to *, and set width to 9
```




    '*******-1'




```python
f"{-1:*<9}"  # set `fill` to *, and set width to 9
```




    '-1*******'




```python
f"{-1:*=9}"  # set `fill` to *, and set width to 9
```




    '-*******1'



##### `[sign]`

It decides how the sign will be represented and is only valid for numerical values.

```python
+      # both positive and negative
-      # only negative (default)
space  # a leading space for positive and minus sign for negative
```


```python
f"{1:+}"
```




    '+1'




```python
f"{-1:+}"
```




    '-1'




```python
assert f"{1}" == f"{1:-}"   # because it's the default behavior
```

##### `[z]`

After Python 3.11, we can use `z` to handle negative zero(i.e. `-0.`). According to the PEP[^3], programmars usually will suppress negative zero.


```python
x = -0.0001
f"{x:.1f}"   # set the precision to 1, so it will round to -0.0
```




    '-0.0'




```python
x = -0.0001
f"{x:z.1f}"  # with z, we will get 0.0 rather than -0.0
```




    '0.0'



##### `[#]` and `[type]`

`[#]` is only valid for integer, float, and complex types.
- integer: add the respective prefix for the different base(radix)
- float and complex: always add a decimal-point even if no digits follow it

How to interpret the numbers in the different bases(radixes)? It's the `[type]`'s job:
```python
b        # base 2
o        # base 8
d        # base 10
x        # base 16, low-case letters
X        # base 16, upper-case letters
```


```python
f"{15:#b}"     # represent 15 in base 2, use # to add prefix 0b
```




    '0b1111'




```python
f"{15:#X}"     # represent 15 in base 16, use # to add prefix 0X
```




    '0XF'




```python
f"{3:.0f}"
```




    '3'




```python
f"{3:#.0f}"
```




    '3.'



##### `[0][width]`

we set the minimal width by setting the `width`. If we also add the prefix `0` here, it means we will use `0` to pad the rest part.


```python
f"{123:5}"     # width 5
```




    '  123'




```python
f"{123:05}"    # width 5 with leading 0
```




    '00123'




```python
f"{123.1:5}"   # Note: the width includes the decimal point char
```




    '123.1'



##### `[grouping_option]`

Specify the thousands separators. Two options available[^4][^5]：
- `_`
- `,`

Both separators will make the long numbers more human-readable. 


```python
f"{123456789:,}"
```




    '123,456,789'




```python
f"{1234.56789:,}"
```




    '1,234.56789'




```python
f"{123456789:_}"
```




    '123_456_789'




```python
f"{1234.56789:_}"
```




    '1_234.56789'



##### `[.precision]`

How many digits should be displayed after the `.`?


```python
f"{123.456:.2f}"
```




    '123.46'



## Wrap up

When I start to learn python a long time ago, most of the tutorials says we should use `%` to format strings. After `str.format` appears, it became a better choice and outdated the `%` syntax. F-strings, released with python 3.6, has become the best practice now.🚀 It meets the zen of python - *There should be one– and preferably only one –obvious way to do it*

## Refs

[^1]: [PEP 498 – Literal String Interpolation](https://peps.python.org/pep-0498/)

[^2]: [Format Specifications](https://docs.python.org/3/library/string.html#format-specification-mini-language)

[^3]: [PEP 682 – Format Specifier for Signed Zero](https://peps.python.org/pep-0682/)

[^4]: [PEP 378 – Format Specifier for Thousands Separator](https://peps.python.org/pep-0378/)

[^5]: [PEP 515. Underscores in Numeric Literals](https://peps.python.org/pep-0515/)

[^6]: [Zen of python](https://en.wikipedia.org/wiki/Zen_of_Python)

