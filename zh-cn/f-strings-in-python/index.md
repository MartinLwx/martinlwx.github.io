# f-strings in Python 3.6


> 这一篇博客一开始是写在 jupyter notebook 里面而后转成 markdown 的。如果想要访问和运行本来的 notebook，请查阅这个[仓库](https://github.com/MartinLwx/oh_my_python)

## 引言

字符串格式化绝对可以算得上日常生活中最为常用的功能之一，我们经常需要输出各种字符串还需要精确控制其格式

在一些过时的 Python 教程上，还可以看到使用 `%` 来格式化字符串。但其实在 Python 3.6 之后，f-strings 已经成了格式化字符串的最优解，优点包括：
1. 可以在字符串字面值（String literal）里面**内嵌表达式**
2. **可读性非常强**

*下面的简单对比就可以看出 f-strings 在可读性上的优势*：


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



在上面的例子中我们分别用两种方式实现了同样功能的字符串输出。

看一眼 f-strings 就知道这个字符串是要输出名字（当然这要求你用的是有意义的变量，比如这里用的是 `name`）。而如果用旧的 `%` 字符串格式化字符串，则稍微显得不那么自然，**我们首先会看到 `%s` 占位符，此时我们需要向右看才知道这里会放置什么变量**。

这里的例子不长所以没啥差别，但是如果用 `%` 格式化的字符串很长的话，看代码就会有割裂感，你的眼睛需要来回左右移动。而且这不仅仅是阅读代码上不大自然，如果是自己写很长的字符串，还得核对一遍占位符和变量的顺序是否一致，这无疑是一个负担。😫

## 语法规则

f-strings 的语法规则很简单，下面我借用[^1]里面的定义：

`f'<text>{<expression><optional !s, !r, or !a> <optional:format_specifier> }<text>...'`

整体上来看可以将其归纳为 `f"..."` 或者 `f'...'` 这种形式。

像普通的字符串那样，我们可以选择用单引号 `''` 或者双引号 `""` 将字符串的内容括起来，当然 `"""..."""` 这种三个引号的也是 OK 的


至于其他部分，我们分成几个小节进行讲解：

### `<text>`
放置字符串字面值，最简单的使用场景是不带任何表达式的字符串，此时 f-strings 就退化成了普通字符串。*比如 `f"Hello world"`*。~~当然这完全没有必要用 f-strings~~

### `{<expression>}` 
我们先考虑最简单的情况： 不包含任何可选部分（即上面语法中的 `<option...>` ）的 `{expression}`

上面其实已经指明了如何在 f-strings 里面嵌入表达式：只要将其放在 `{}` 里面即可。*下面是一个简单的例子*


```python
left, right = 3, 5
f"{left} + {right} = {left + right}"
```




    '3 + 5 = 8'



用 `{}` 来充当占位符就很自然引申出一个问题——如果要在字符串里面输出 `{}` 这两个字符怎么办？

答案也很简单，我们只要用 `{{expression}}` 这种格式即可


```python
# 'hello' is an expression too. 
# Note that we need to use different quote syntax inside {}
f"{{'hello'}}"   
```




    "{'hello'}"



值得一提的还有，**`expression` 里面不允许出现 `:` 和 `!` 和 `\`**

实际上 `expression` 里面也很少会包含这几个字符。`\` 的一个使用场景是用来转义 `'` 或者 `"`，但其实只要我们内外用的是不同的引号格式即可。至于 `:`，在 PEP[^1] 里面举例可能应用场景是 lambda 表达式，此时我们只需用一对括号将 lambda 表达式括起来即可：


```python
# note that we need to add () around the lambda expression
f"{(lambda x: x + 1)(3)}" 
```




    '4'



### `<option...>`

#### `<optional !s, !r, or !a>`

这几个是用来对 `<expression>` 做**类型转换**的
```python
{foo:!s}     # it's equal to call str(foo) first
{foo:!r}     # similarily, call repr(foo) first
{foo:!a}     # similarily，call ascii(foo) first
```

这三个**了解即可，一般也不会怎么用到**，真要用的话还是**自己直接调用对应的函数会比较直观**，PEP 498[^1] 里面自己也说了这三个只是为了最小化和之前的 `str.format` 的差别

#### `<optional:format_specifier>`

这一部分就是对前面 `expression` 的值进行“修饰”，*比如我们想要控制小数的精度，或者做字符串对齐等等*。下面我会简单讲讲其中几个部分。更为详细的完整支持选项可以参考[^2]

> 注意这个部分的开头用的是 `:`（这也一定程度上解释了为什么 Python 不允许 `expression` 里面出现 `:`）

可以先看看 `format_specifier` 的语法描述：

`[[fill]align][sign][z][#][0][width][,][grouping_option][.precision][type]`

> 在语法里面，`[]` 表示是可选项

##### `[[fill]align]`

主要用来对字符串进行输出对齐，这里的 `fill` 可以是任意的字符（默认情况下是空格）用于填充剩下的部分

> 注意：`[[fill]align]` 需要和 `width` 一起搭配使用，**没有指定输出宽度的情况下是没有用的**

`align` 支持下面这几种对齐方式：
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

用于控制数值类型的符号位显示，支持下面这几种

```python
+      # both positive and negative
-      # only negative (default)
space  # a leading spaces for positive and minus sign for negative
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

在 Python 3.11 里面，增加了可选的 `z` 用来处理 `-0.`。因为调查显示[^3]，大多数情况下我们都不想得到 `-0.0` 这样的输出


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



##### `[#]` 和 `[type]`

`[#]` 只对 integer、float 和 complex 类型有效
- integer：会**根据你选择输出的进制增加相应的前缀**，*比如二进制就增加 `0b`*
- float 和 complex：**总是输出小数点**，就算小数点之后没有数字

如何用不同的进制显示？这就是 `[type]` 的工作：
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

width 是用来控制显示的最小长度，如果加上 `0`，那么就会在数值前面补 0


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

指定**千分位分隔符**，有两种选项[^4][^5]：
- `_`
- `,`

在数字很大的时候，这两个千分位分隔符都会让可读性大大增强！


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

指定小数点之后要保留几位，会自动舍入


```python
f"{123.456:.2f}"
```




    '123.46'



## 总结

在我刚开始学习 Python 的时候，当时的教程都是在用 `%` 来格式化输出，后来的推荐是 `str.format` 方法。而到了 f-string 随着 Python 3.6 发布之后，似乎统一了字符串输出的 Best practice，大家都在用这个。这也是符合 Python 设计哲学的——应该只有一种显而易见的方式做到一件事🚀

## 参考

[^1]: [PEP 498. Literal String Interpolation](https://peps.python.org/pep-0498/)

[^2]: [Format Specifications](https://docs.python.org/3/library/string.html#format-specification-mini-language)

[^3]: [PEP 682. Format Specifier for Signed Zero](https://peps.python.org/pep-0682/)

[^4]: [PEP 378. Format Specifier for Thousands Separator](https://peps.python.org/pep-0378/)

[^5]: [PEP 515. Underscores in Numeric Literals](https://peps.python.org/pep-0515/)


