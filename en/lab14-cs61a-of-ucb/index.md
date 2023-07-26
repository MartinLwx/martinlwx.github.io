# Lab14 - CS61A of UCB(2021-Fall)


## Trees

---

### Q1: Prune Min

>   Write a function that prunes a `Tree` `t` mutatively. `t` and its branches always have zero or two branches. For the trees with two branches, reduce the number of branches from two to one by keeping the branch that has the smaller label value. Do nothing with trees with zero branches.
>
>   Prune the tree in a direction of your choosing (top down or bottom up). The result should be a linear tree.

The details:

1.   the leaf node: It is the base case, we `return`
2.   a node with only one branch: the current node meets the requirements of the description. However, its subtree may violate. So we still need to recursively process the branches.
3.   a node with two branches: Find the smaller one, and `del` the bigger one.

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

The details of the regex:

1.   a block number has a sequence of 3-5 digits
2.   The street name may contain multiple words, but the last one should be the abbreviation(2-5 English letters)
3.   The street name **may** start with one of "N", "E", "W", "S"



ps. The word should be properly capitalized(check the description)



Let's break down this problem:  

-   a block number: `\d{2,5}` is enough
-   The possible prefix -  "N", "E", "W", "S". Use `(?:[NSWE] )?`. Keep in mind that it has whitespace inside
-   The street name: we should first have a word, then we can repeat the whitespace + word combination as many times as we want. ✅ `[A-Z][A-Za-z]*(?: [A-Z][A-Za-z]*)*`.
-   The abbreviation: Why do I use `{1,4}` instead of `{2,5}`? The length of the abbreviation is between 2 and 5. However, after excluding the first capitalized letter, it should be in the range of 1 - 4.

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

Find the pizza place whose opening time is before 1:00 pm. Note that the time in the database is the 24-hour clock

```sql
CREATE TABLE opening AS
SELECT name FROM pizzas
WHERE open < 13
ORDER BY name DESC;
```

### Q6: Double Pizza

>   If two meals are more than 6 hours apart, then there's nothing wrong with going to the same pizza place for both, right? Create a `double` table with three columns. The first columns is the earlier meal, the second is the later meal, and the third is the name of a pizza place. Only include rows that describe two meals that are **more than 6 hours apart** and a pizza place that is open for both of the meals. The rows may appear in any order.

The details:

1.   Two meals should be more than 6 hours apart.
2.   The start time of the first meal and the end time of the second meal must be within the business hours of the restaurant

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

The calculation method has been given in the description. Note that both of `votes` and `popularity` need to be modified in the `speech` method

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

We determine who is the current player based on whether `self.turn` is odd or even. Note that the `choose` method returns a function, so we need to pass the parameters

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

:)

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

### Q10: Add trees

>   Define the function `add_trees`, which takes in two trees and returns a new tree where each corresponding node from the first tree is added with the node from the second tree. If a node at any particular position is present in one tree but not the other, it should be present in the new tree as well.
>
>   *Hint*: You may want to use the built-in zip function to iterate over multiple sequences at once.

This problem asks us to add one tree to another. Note who is the subject (useful for later thinking about how to solve it recursively).



How to solve it with recursion?

-   base case:

    -   We are adding the tree `t2` to the tree `t1`. What if `t2` is `None`, do `t2` still need to be added? No, this is one of our base cases :hugs:. 

    -   On the basis that `t2` is not `None`, what if `t1` is empty? Obviously, the result of adding two trees at this time is `t2`, so we can just return `t2`.

-   recursive decomposition

    -   At this time, we can guarantee that `t1` and `t2` are both non-empty, **but we can't guarantee that they have the same number of children**, so we can't use `zip` according to the hint. Because when `zip` is processing the two sequences of different lengths, some elements of the longer one are ignored. The `zip_longest` in `itertools` should be used here, and `None` will be returned if a sequence is already empty. This way we can guarantee that this recursive call will eventually come to the base case we discussed earlier. (**Don't forget to write `from itertools import zip_longest`**)
    -   Then what we do at the current node is: add the labels of the two nodes, and then recursively call `add_trees` on their subtrees. :rocket:

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

This is calculated from top to bottom, that is, we will calculate the result before entering deeper recursive calls. When we reach the empty node (indicating that we have done all the operations), we get the `z` in the base case, but it is not the original `z` at the first call, but the `z` of the operation result we want to get at the end.

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

In fact, this problem will be simpler than Q1, and it is more in line with our understanding of recursion. We return the result operation layer by layer from the bottom layer. **You can see the following examples to understand this**

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

## 

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

The details:

1.   `scheme` : Either `http` or `https`, which can be expressed by `(?:...)`.
2.   `domain` : The `www` may show at the beginning of the domain, which can be expressed by `(?:...)?`
3.   path to the file: `/path/to/file.extension`, `/path`, `/file.extension`. They are all valid
4.   `anchor` : letters, numbers, hyphens, or underscores.

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

Write the BNF of a simple csv file. It is simple because: the value can only be a word, and them should be seperated by `,`



The details:

1. There may be cases of `,,,`, so we need to use `|` in `word` to count the cases where no characters are placed

```
lines: line (newline line)* | line newline | line
line: word ("," word)*
word: WORD | 
newline: "\n"

%import common.WORD
```


