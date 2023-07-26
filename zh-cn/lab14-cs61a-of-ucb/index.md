# Lab14 - CS61A of UCB(2021-Fall)


## Trees

---

### Q1: Prune Min

>   Write a function that prunes a `Tree` `t` mutatively. `t` and its branches always have zero or two branches. For the trees with two branches, reduce the number of branches from two to one by keeping the branch that has the smaller label value. Do nothing with trees with zero branches.
>
>   Prune the tree in a direction of your choosing (top down or bottom up). The result should be a linear tree.

递归的细节:

1.   叶子结点 : 直接返回
2.   只有一个分支 : 虽然当前节点满足条件, 但是它的子树里可能有不符合条件的节点, 所以我们要递归裁剪分支
3.   有两个分支 : 找到较小的分支, 把较大的分支裁剪掉(用 `del`)

```python
def prune_min(t):
    """Prune the tree mutatively.
    """
    # base case: the leaf node has 0 children
    if t.is_leaf():
        return 
    # go deeper if it has 1 child
    if len(t.branches) == 1:
        prune_min(t.branches[0])

    left, right = t.branches[0], t.branches[1]
    if left.label < right.label:
        del t.branches[1]           # prune right branch
        prune_min(left)
    else:
        del t.branches[0]           # prune left branch
        prune_min(right)
```



## Regex

---

### Q4: Address First Line

>   Write a regular expression that parses strings and returns any expressions which contain the first line of a US mailing address.
>
>   
>
>   US mailing addresses typically contain a block number, which is a sequence of 3-5 digits, following by a street name. The street name can consist of multiple words but will always end with a street type abbreviation, which itself is a sequence of 2-5 English letters. The street name can also optionally start with a cardinal direction ("N", "E", "W", "S"). Everything should be properly capitalized.
>
>   
>
>   Proper capitalization means that the first letter of each name is capitalized. It is fine to have things like "WeirdCApitalization" match.
>
>   
>
>   See the doctests for some examples.

我们在这里要匹配美国的邮件地址, 下面是关于这个的正则表达式的细节:

1.   开头有 3 - 5 个数字
2.   然后是一个街道的名字, 可能会有多个单词, 但是结尾一定是街道类型的缩写(2 - 5 个英文字母)

3.   但是街道名字也有可能是 "N", "E", "W", "S" 这四个字母开头

ps. 上面说到的是 “properly capitalized“ 的单词, 具体定义可以看问题描述. 我的理解就是单词的开头一定要是大写, 后面就随便了~~



下面我们分解问题, 把整个要匹配的地址分为上面提到的几个部分:

-   区号 : 这个简单, `\d{2,5}` 即可
-   可能的 "N", "E", "W", "S" 开头, 这个也简单, 用 `(?:[NSWE] )?` 注意里面有一个空格
-   然后就是街道名字要怎么写了, 街道名字可能包含多个单词, 每个单词都要求开头大写, 所以我们可以整合出这样的正则表达式: `[A-Z][A-Za-z]*(?: [A-Z][A-Za-z]*)*`. 首先要有一个单词, 然后后面可以有任意个空格 + 单词的组合
-   最后是要有街道类型的缩写, 注意这里仍然要是单词开头大写, 其他任意, 本来是 2 - 5 个字母, 现在去掉了开头的第一个大写字母之后, 剩下  1 - 4 个字母. ` [A-Z][A-Za-z]{1,4}\b`. 最后的 `\b` 是让正则表达式只匹配单词的结尾.

```python
def address_oneline(text):
    """
    Finds and returns expressions in text that represent the first line
    of a US mailing address.
    """
    block_number = r'\d{3,5}'
    cardinal_dir = r'(?:[NSWE] )?'       # whitespace is important!
    street = r'[A-Z][A-Za-z]*(?: [A-Z][A-Za-z]*)*'
    type_abbr = r' [A-Z][A-Za-z]{1,4}\b'
    street_name = f"{cardinal_dir}{street}{type_abbr}"
    return re.findall(f"{block_number} {street_name}", text)
```

## SQL

---

### Q5: Opening Times

>   You'd like to have lunch before 1pm. Create a `opening` table with the names of all Pizza places that open before 1pm, listed in reverse alphabetical order.

找到开店时间在下午一点前的, 注意数据库里的时间是 24 小时制

```sql
CREATE TABLE opening AS
SELECT name FROM pizzas
WHERE open < 13
ORDER BY name DESC;
```

### Q6: Double Pizza

>   If two meals are more than 6 hours apart, then there's nothing wrong with going to the same pizza place for both, right? Create a `double` table with three columns. The first columns is the earlier meal, the second is the later meal, and the third is the name of a pizza place. Only include rows that describe two meals that are **more than 6 hours apart** and a pizza place that is open for both of the meals. The rows may appear in any order.

考察表的联合, 注意细节:

1.   两顿饭之间间隔要超过六个小时
2.   第一顿饭的开始时间到第二顿饭的结束时间要在饭店的营业时间内

```sql
create TABLE double AS
SELECT m1.meal, m2.meal, p.name
FROM meals AS m1, meals AS m2, pizzas AS p 
WHERE m2.time - m1.time > 6 AND m1.time >= p.open AND m2.time <= p.close;
```

## Objects

---

### Q7: Player

>   First, let's implement the `Player` class. Fill in the `debate` and `speech` methods, that take in another `Player` `other`, and implement the correct behavior as detailed above. Here are two additional things to keep in mind:
>
>   -   In the `debate` method, you should call the provided `random` function, which returns a random float between 0 and 1. The player should gain 50 popularity if the random number is smaller than the probability described above, and lose 50 popularity otherwise.
>   -   Neither players' popularity should ever become negative. If this happens, set it equal to 0 instead.

计算方法在规则里面已经给出, 这里要注意 `speech` 方法里面要修改 `votes` 和 `popularity`

```python
class Player:
    def __init__(self, name):
        self.name = name
        self.votes = 0
        self.popularity = 100

    def debate(self, other):
        prob1 = max(0.1, self.popularity / (self.popularity + other.popularity))
        #prob2 = max(0.1, other.popularity / (self.popularity + other.popularity))
        if random() > prob1:
            self.popularity -= 50
        else:
            self.popularity += 50
        if self.popularity < 0:
            self.popularity = 0

    def speech(self, other):
        self.votes += (self.popularity // 10)
        self.popularity += (self.popularity // 10)
        other.popularity -= (other.popularity // 10)

    def choose(self, other):
        return self.speech
```

### Q8: Game

>   Now, implement the `Game` class. Fill in the `play` method, which should alternate between the two players, starting with `p1`, and have each player take one turn at a time. The `choose` method in the `Player` class returns the method, either `debate` or `speech`, that should be called to perform the action.
>
>   
>
>   In addition, fill in the `winner` property method, which should return the player with more votes, or `None` if the players are tied.

根据 `self.turn` 是奇数还是偶数来决定当前轮到谁, 注意 `choose` 方法返回的仍然是函数, 所以你需要给他参数才行

```python
class Game:
    def __init__(self, player1, player2):
        self.p1 = player1
        self.p2 = player2
        self.turn = 0

    def play(self):
        while not self.game_over:
            if self.turn % 2 == 0:
                self.p1.choose(self.p2)(self.p2)
            else:
                self.p2.choose(self.p1)(self.p1)
            self.turn += 1
        return self.winner

    @property
    def game_over(self):
        return max(self.p1.votes, self.p2.votes) >= 50 or self.turn >= 10

    @property
    def winner(self):
        if self.p1.votes > self.p2.votes:
            return self.p1
        else:
            return self.p2
```

### Q9: New Players

>   The `choose` method in the `Player` class is boring, because it always returns the `speech` method. Let's implement two new classes that inherit from `Player`, but have more interesting `choose` methods.
>
>   Implement the `choose` method in the `AggressivePlayer` class, which returns the `debate` method if the player's popularity is less than or equal to `other`'s popularity, and `speech` otherwise. Also implement the `choose` method in the `CautiousPlayer` class, which returns the `debate` method if the player's popularity is 0, and `speech` otherwise.

翻译一下题目的意思即可 :)

```python
class AggressivePlayer(Player):
    def choose(self, other):
        if self.popularity <= other.popularity:
            return self.debate
        else:
            return self.speech
        

class CautiousPlayer(Player):
    def choose(self, other):
        if self.popularity == 0:
            return self.debate 
        else:
            return self.speech
```

## Tree Recursion

---

>   Define the function `add_trees`, which takes in two trees and returns a new tree where each corresponding node from the first tree is added with the node from the second tree. If a node at any particular position is present in one tree but not the other, it should be present in the new tree as well.
>
>   *Hint*: You may want to use the built-in zip function to iterate over multiple sequences at once.

这一题要我们把一棵树合并到另外一棵树上去. 注意区分谁是主体(对后面思考如何递归解决很有用).



如何用递归解决?

-   base case:

    -   我们是把树 `t2` 加到树 `t1` 上, 那如果 `t2` 是 `None` 的话不就不用加了吗 ? 是的, 这个就是我们的一个 base case :hugs:

    -   在 `t2` 不为空的基础上, 如果 `t1` 因为空呢 ? 显然, 这个时候两棵树相加的结果就是 `t2`, 所以我们返回 `t2` 即可.

-   递归分解

    -   这个时候我们可以保证 `t1` 和 `t2` 都是非空的, **但是我们无法保证他们两个的孩子数目是一样多的**, 所以我们无法根据提示里使用 `zip`. 因为 `zip` 在处理两个两个不一样长的序列的时候, 长的序列多的部分是被忽略的. 这里应该用 `itertools` 里面的 `zip_longest`, 如果一个序列已经为空会返回 `None`. 这样我们才能保证这样递归调用最后可以回到我们前面讨论的 base case 中.
    -   那我们在当前节点做的事情就是: 相加两个节点的 label, 然后对他们的子树都递归调用即可. :rocket:

```python
def add_trees(t1, t2):
    # base case: no need to add_trees anymore
    if t2 is None:
        return t1
    if t1 is None:
        return t2
    else:
        # both of t1 and t2 are not None
        # however, the number of children may not equal
        return Tree(t1.label + t2.label, [add_trees(x, y) for x, y in zip_longest(t1.branches, t2.branches)])
```



## Linked Lists

---

### Q11: Fold Left

>   Write the left fold function by filling in the blanks.

这个是自顶向下计算的, 也就是我们在进入更深层的递归调用之前会先计算结果, 并将这个结果传递下去, 当我们到达了空结点(说明我们做完了所有的运算)就得到了结果. 所以 base case 里面的 `z` 已经不是一开始的 `z` 了, 而是我们最后要得到的运算结果的 `z`

```python
def foldl(link, fn, z):
    """ Left fold
    >>> lst = Link(3, Link(2, Link(1)))
    >>> foldl(lst, sub, 0) # (((0 - 3) - 2) - 1)
    -6
    >>> foldl(lst, add, 0) # (((0 + 3) + 2) + 1)
    6
    >>> foldl(lst, mul, 1) # (((1 * 3) * 2) * 1)
    6
    """
    if link is Link.empty:
        return z
    z = fn(z, link.first) 
    return foldl(link.rest, fn, z)
```

### Q12: Fold Right

>   Now write the right fold function.

其实这个问题会比 Q1 更为简单, 更符合我们对递归的认知. 我们从最底层逐层返回结果运算. **可以看下面的例子理解一下**

```python
def foldr(link, fn, z):
    """ Right fold
    >>> lst = Link(3, Link(2, Link(1)))
    >>> foldr(lst, sub, 0) # (3 - (2 - (1 - 0)))
    2
    >>> foldr(lst, add, 0) # (3 + (2 + (1 + 0)))
    6
    >>> foldr(lst, mul, 1) # (3 * (2 * (1 * 1)))
    6
    """
    if link.rest is Link.empty:
        return fn(link.first, z)
    return fn(link.first, foldr(link.rest, fn, z))
```

## Regex

---

### Q13: Basic URL Validation

>   In this problem, we will write a regular expression which matches a URL. URLs look like the following:
>
>   ![URL](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_URL/mdn-url-all.png)
>
>   For example, in the link `https://cs61a.org/resources/#regular-expressions`, we would have:
>
>   -   Scheme: `https`
>   -   Domain Name: `cs61a.org`
>   -   Path to the file: `/resources/`
>   -   Anchor: `#regular-expressions`
>
>   The port and parameters are not present in this example and you will not be required to match them for this problem.
>
>   You can reference [this documentation from MDN](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_URL) if you're curious about the various parts of a URL.
>
>   For this problem, a valid domain name consists of any sequence of letters, numbers, dashes, and periods. For a URL to be "valid," it must contain a valid domain name and will optionally have a scheme, path, and anchor.
>
>   A valid scheme will either be `http` or `https`.
>
>   Valid paths start with a slash and then must be a valid path to a file or directory. This means they should match something like `/composingprograms.html` or `path/to/file` but not `/composing.programs.html/`.
>
>   A valid anchor starts with `#`. While they are more complicated, for this problem assume that valid anchors will then be followed by letters, numbers, hyphens, or underscores.
>
>   >   **Hint 1**: You can use `\` to escape special characters in regex.
>
>   \>**Hint 2**: The provided code already handles making the scheme, path, and anchor optional by using non-capturing groups.

要学会将复杂的正则表达式进行拆解, 一个部分一个部分的完成. 题目已经为我们完成了这个工作. 接下来我讲解一下每个部分的设计要点

1.   `scheme` : 这个简单, 要么是 `http` 要么是 `https`, 我们可以用 `(?:...)` 来写
2.   `domain` : `www` 可以有也可以没有, 可以用 `(?:...)?` 来表达这点
3.   path to the file: 包含下面三种可能的格式: `/path/to/file.extension`, `/path`, `/file.extension`
4.   `anchor` : 这个可能包括字母、数字、短横线 -, 下划线 `_`

```python
def match_url(text):
    scheme = r'(?:https|http)://'
    domain = r'(?:\w+\.)?\w+\.\w+'
    path = r'(?:/\w+|/(\w+/)*)(\w+\.\w+)?'
    anchor = r'#[\w\-_]*'
    return bool(re.match(rf"^(?:{scheme})?{domain}(?:{path})?(?:{anchor})?$", text))
```

## BNF

---

### Q14: Simple CSV

>   CSV, which stands for "Comma Separated Values," is a file format to store columnar information. We will write a BNF grammar for a small subset of CSV, which we will call SimpleCSV.
>
>   
>
>   Create a grammar that reads SimpleCSV, where a file contains rows of words separated by commas. Words are characters `[a-zA-Z]` (and may be blank!) Spaces are not allowed in the file.

写一个简单的 csv 文件的 BNF, 这里说的简单是因为: 值只能是单词, 分隔符一定是 `,`



细节:

1.   有可能有 `,,,` 的情况出现, 所以我们需要在 `word` 里面用 `|` 来算上不放任何字符的情况

```
lines: line (newline line)* | line newline | line
line: word ("," word)*
word: WORD | 
newline: "\n"

%import common.WORD
```

