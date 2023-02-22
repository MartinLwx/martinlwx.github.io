# How to draw a simple relation graph in Python


## Intro

---

The process of drawing a simple relation graph in python can be broken down into 2 steps.

1. Define a graph. 
2. Draw a graph.

## Step 1. Define a graph

---

In this step, we will use the [networkx](https://networkx.org) package.

### Install tutorial

If you are using conda, you can just type `conda install networkx`

If you are using pip, you can just type `pip install networkx`

### Nodes

First of all, you need to create a graph.

```python
import networkx as nx
G = nx.Graph()
```

You can use different ways to add nodes.

1. Add one node at a time.
2. Add nodes from any iterable container
3. Add nodes along with node attributes. In this way, you can define many attributes of a node, such as color, size, etc.
4. Add nodes from another graph directly

```python
G.add_node(1)	                  # method 1

G.add_nodes_from([2, 3, 4, 5])    # method 2

G.add_nodes_from([                # method 3
    (6, {"color": "red"}),
    (7, {"color": "blue"})
])

G2 = nx.Graph()                   # method 4
G2.add_nodes_from([8, 9, 10])
G.add_nodes_from(G2)

# you can verify nodes by 
print(G.nodes)
# NodeView((1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
```

### Edges

Also, `networkx` has many ways to add edges, which is quite similar to add nodes. Let's just jump to the code:hugs:

```python
G.add_edge(1, 2)                   # method 1

G.add_edges_from([(1, 3), (1, 4)]) # method 2

G2.add_edge(8, 9)                  # method 3
G2.add_edge(8, 10)           
# WARNING: you can't just use G2 instead of G2.edges
G.add_edges_from(G2.edges)         # method 4  
```

### Get some information about a graph

Usually, we want to know some information about a graph. For example, we may want to know the number of nodes and edges, the degree of a node, etc. Of course, `networkx` implements this for us.:arrow_down:

```python
# nodes information
print(list(G.nodes))                        # output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
print(G.number_of_nodes())                  # output: 10

# edges information
print(list(G.edges))                        # output: [(1, 2), (1, 3), (1, 4), (8, 9), (8, 10)]
print(G.number_of_edges())                  # output: 5

# neighbors of a node
print(list(G.neighbors(1)))                 # output: [2, 3, 4]

# degree information
print(G.degree(1))                          # output: 3, because we have edges: (1, 2), (1, 3), (1, 4)
```

### manipulate nodes && edges

The operation of removing is the inverse process of the operation of adding.

```python
G.remove_node(5)                            
G.remove_nodes_from(G2)      

# similar to remove_node && remove_nodes_from
# you can add parameters you like
# G.remove_edge()
# G.remove_edges_from()
```

### 🤔

> 📒 The node type doesn't have to be an integer. It can also be a string. 
>

```python
new_G = nx.Graph()                          # Let's define a new graph

new_G.add_node("abcd")                      # Add a string node 
new_G.add_nodes_from("abcd")                # this line is different, we treat the string as an iterable cotainer

print(list(new_G.nodes))                    # output: ['abcd', 'a', 'n', 'c', 'd']
```

## Step 2. Draw a graph

---

Although we can draw a graph defined by `networkx` in `matplotlib.pyplot`, the generated image is static and not pretty in my opinion. So I will just skip the tutorial of drawing in `matplotlib.pyplot`. I will use [`pyvis`](https://pyvis.readthedocs.io/en/latest/) package instead. 

### Install tutorial

I didn't find how to use conda to install `pyvis`, so you may just use `pip install pyvis`. I remember that we can use `pip` in conda environment, but we need to run `conda install pip` at first(Not quite sure😨)

### How to use `pyvis`

```python
from pyvis.network import Network

net = Network('500px', '500px')
net.from_nx(G)												# The G is defined in Step 1.
net.show('net.html')
```

Then you can open the `net.html` in your local machine to interact with it. It looks like this:arrow_down:

![](/img/pyvis.gif)

I don't know how to integrate my html file into this blog. **You may check the result in [this](/urls/pyvis.html) (If you follow my tutorial)**

### 🤔

> 📒 In `pyvis`, the valid node attributes are:arrow_down:. So when we use `networkx` to add nodes, we can directly use these attributes. In this way, the `pyvis` will translate them for us.
>

```python
['size', 'value', 'title', 'x', 'y', 'label', 'color']
```

> Similarly, if you want to customize your edges' attributes, you may check the [docs](https://visjs.github.io/vis-network/docs/network/edges.html)
>
> Then you can use `G.add_edge(a, b, attribute1=, attribute2, ...)`

## References

---

1. [visjs docs](https://visjs.github.io/vis-network/docs/network/edges.html)
2. [pyvis docs](https://pyvis.readthedocs.io/en/latest/index.html)
3. [networkx tutorial](https://networkx.org/documentation/stable/tutorial.html)


