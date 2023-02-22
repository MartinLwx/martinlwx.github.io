# Lab12 CS61A of UCB(2021-Fall)


## Regular Expressions

---

### Q1: Calculator Ops

>   Write a regular expression that parses strings written in the 61A Calculator language and returns any expressions which have two numeric operands, leaving out the parentheses around them.

We need to write a regular expression to match a pattern - `(operand operator1 operator2)`. The operands consist of `+`, `-`, `*`, `/`. We can use `[]` here. Don't forget to put a `\` in front of `-` to escape it.

```python
def calculator_ops(calc_str):
    """
    Finds expressions from the Calculator language that have two
    numeric operands and returns the expression without the parentheses.
    """
    return re.findall(r'[+\-*/] \d+ \d+', calc_str)
```

## BNF

---

### Q3: Linked List BNF

>   In this problem, we're going to define a BNF that parses integer Linked Lists created in Python. We won't be handling `Link.empty`.
>
>   For reference, here are some examples of Linked Lists:
>
>   *Your implementation should be able to handle nested Linked Lists, such as the third example below.*
>
>   -   `Link(2)`
>   -   `Link(12, Link(2))`
>   -   `Link(5, Link(7, Link(Link(8, Link(9)))))`

The idea: The beginning of the linked list must be `Link(`, and then we can divide the linked list into `link_first` and `link_rest` parts, which are either numbers or another linked list(nested). The `link_rest` can be empty !

```
link: "Link(" link_first ")" | "Link(" link_first "," link_rest ")"
?link_first: NUMBER | link
?link_rest: NUMBER | link

%ignore /\s+/
%import common.NUMBER
```

### Q4: Tree BNF

>   Now, we will define a BNF to parse Trees with integer leaves created in Python.
>
>   Here are some examples of Trees:
>
>   *Your implementation should be able to handle Trees with no branches and one or more branches.*
>
>   -   `Tree(2)`
>   -   `Tree(6, [Tree(1), Tree(3, [Tree(1), Tree(2)])])`

The BNF of a tree:

1.   `tree_node`: It can a tree with only one node, or a tree with nodes and branches. The number of the branches can by `[0, ∞)`, so we can use the `*` in the regular expression. 
2.   `?label`: `NUMBER`
3.   `branches`: It can be only one node, or there are multiple nodes (in this case, we need to match the `,` signs)

```
tree_node: "Tree(" label ")" | "Tree(" label "," branches* ")"
?label: NUMBER
branches: "[" tree_node "]" | "[" tree_node "," tree_node+ "]"

%ignore /\s+/
%import common.NUMBER
```




