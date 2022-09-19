# Hw10 of CS61A of UCB(2021-Fall)


## BNF

---

### Q1: Grouping and Pipes

>   In this question, you will add support for grouping and piping.
>
>   Recall that grouping allows for an entire regular expression to be treated as a single unit, and piping allows for a pattern to match an expression on either side. Combined, these will let us create patterns which match multiple strings!
>
>   Define the `group` and `pipe` expressions in your grammar.
>
>   1.  A **`group`** consists of any `regex` expression surrounded by parentheses (`()`).
>   2.  A **`pipe`** operator consists of a `regex` expression, followed by a pipe (`|`) character, and lastly followed by another `regex` expression.
>
>   For example, `r"apples"` would match exactly the phrase "apples" in an input. If we wanted our pattern from before to match "oranges" as well, we could expand our `rstring` to do so using groupings and pipes: `r"(apples)|(oranges)"`.
>
>   >   Hint: note that `group`s and `pipe`s are valid `regex` expressions on their own! You may need to update a previously defined expression.

The description indicates the way to solve this problem. Remember that the `group` and the `pipe` are a part of `?regex`, so we should add them to `?regex`.

```
?start: rstring
rstring: "r\"" regex* "\""
?regex: character | word | group | pipe

group: "(" regex* ")"
pipe: regex "|" regex

character: LETTER | NUMBER
word: WORD

%ignore /\s+/
%import common.LETTER
%import common.NUMBER
%import common.WORD
```

### Q2: Classes

>   Now, we will add support for character classes.
>
>   Recall that character classes allow for the pattern to match any singular `character` defined within the class. The class itself consists either of individual `character`s, or `range`s of `characters`.
>
>   Specifically, we define the following:
>
>   1.  A **`range`** consists of either `NUMBER`s or `LETTER`s separated by a hyphen (`-`).
>   2.  A **`class`** expression consists of any number of `character`s or character `range`s surrounded by square brackets (`[]`).
>
>   Note that for this question, a range may only consist of either `NUMBER`s or `LETTER`s; this means that while `[0-9]` and `[A-Z]` are valid ranges, `[0-Z]` would not be a valid range. In addition, the `character`s and `range`s in a `class` may appear in any order and any number of times. For example, `[ad-fc0-9]`, `[ad-f0-9c]`, `[a0-9d-fc]`, and `[0-9ad-fc]` are all valid classes.

The details:

-   `range` : `character "-" character` is wrong ❌. For example, `[0-z]` is illegal.
-   Because the order in which `range` and `character` appear in `[]` is arbitrary, we can use `*`, both of which may be at the beginning or end of `[]`
    -   `class` itself is also a valid regular expression, so it should be placed inside `?regex`

```
?start: rstring
rstring: "r\"" regex* "\""
?regex: character | word | group | pipe | class

group: "(" regex* ")"
pipe: regex "|" regex

range: NUMBER "-" NUMBER | LETTER "-" LETTER
class: "[" range* character* range* character* "]"

character: LETTER | NUMBER
word: WORD

%ignore /\s+/
%import common.LETTER
%import common.NUMBER
%import common.WORD
```

### Q3: Quantifiers

>   Lastly, we will add support for quantifiers.
>
>   Recall that quantifiers allow for a pattern to match a specified number of a unit.
>
>   Specifically, we define the following:
>
>   1.  A **`plus_quant`** expression consists of a `group`, a `character`, or a `class`, followed by a plus symbol (`+`).
>   2.  A **`star_quant`** expression consists of a `group`, a `character`, or a `class`, followed by a star symbol (`*`).
>   3.  A **`num_quant`** expression consists of either a `group`, a `character`, or a `class`, followed by one of the following:
>       1.  a `NUMBER` enclosed in curly braces (`{}`);
>       2.  a range of `NUMBER`s (separated by a comma (`,`), which may potentially be open on only one side. For example, `{2,7}`, `{2,}`, and `{,7}` are valid numeric quantifiers. `{,}` is not valid.
>
>   >   Hint: these three quantifiers share many similarities. Consider defining additional expressions in this question!

We can make a `?tmp: class | group | character` to represent the similarities. Also, we can define `?quants: plus_quant | star_quant | num_quant`

```
rstring: "r\"" regex* "\""
?regex: character | word | group | pipe | class | quants

group: "(" regex* ")"
pipe: regex "|" regex

range: NUMBER "-" NUMBER | LETTER "-" LETTER
class: "[" range* character* range* character* "]"

?tmp: class | group | character
plus_quant: tmp "+"
star_quant: tmp "*"
num_quant: tmp "{" ((NUMBER "," NUMBER) | (NUMBER "," NUMBER?) | ("," NUMBER)) "}"
?quants: plus_quant | star_quant | num_quant

character: LETTER | NUMBER
word: WORD

%ignore /\s+/
%import common.LETTER
%import common.NUMBER
%import common.WORD
```

## SQL

---

### Q4: Size of Dogs

>   The Fédération Cynologique Internationale classifies a standard poodle as over 45 cm and up to 60 cm. The `sizes` table describes this and other such classifications, where a dog must be over the `min` and less than or equal to the `max` in `height` to qualify as a `size`.
>
>   
>
>   Create a `size_of_dogs` table with two columns, one for each dog's `name` and another for its `size`.

We need to determine the `size` of the dogs, which can be described as `select ... from ... where`.

```mysql
CREATE TABLE size_of_dogs AS
  SELECT d.name, s.size 
  FROM dogs as d, sizes as s
  where d.height <= s.max and d.height > s.min;
```

### Q5: By Parent Height

>   Create a table `by_parent_height` that has a column of the names of all dogs that have a `parent`, ordered by the height of the parent from tallest parent to shortest parent.

Sort results by height descending by using `DESC` keyword.

```mysql
CREATE TABLE siblings AS
  SELECT p1.child AS dogone, p2.child AS dogtwo, s1.size AS dogonesize, s2.size AS dogtwosize
  FROM parents AS p1, parents AS p2, size_of_dogs AS s1, size_of_dogs AS s2
  WHERE p1.parent = p2.parent AND p1.child < p2.child AND p1.child = s1.name AND p2.child = s2.name;
-- Use `<` to filter the result
-- `!=` is not enough, you will get `barack clinton` and `clinton barack` in the same time
```

### Q6: Sentences

>   There are two pairs of siblings that have the same size. Create a table that contains a row with a string for each of these pairs. Each string should be a sentence describing the siblings by their size.
>
>   
>
>   Each sibling pair should appear only once in the output, and siblings should be listed in alphabetical order (e.g. `"barack plus clinton..."` instead of `"clinton plus barack..."`), as follows:
>
>   
>
>   *Hint*: First, create a helper table containing each pair of siblings. This will make comparing the sizes of siblings when constructing the main table easier.
>
>   **Hint**: If you join a table with itself, use `AS` within the `FROM` clause to give each table an alias.
>
>   **Hint**: In order to concatenate two strings into one, use the `||` operator.

After finishing Q5, this one should be much easier.

```mysql
CREATE TABLE sentences AS
    SELECT "The two siblings, " || dogone || " plus " || dogtwo || " have the same size: " || dogonesize
    FROM siblings
    WHERE dogonesize = dogtwosize AND dogone < dogtwo;
```


