# Understanding GAT throught MPNN


## What's MPNN

Justin Gilmer proposed the MPNN (Message Passing Neural Network) framework [^1] for describing graph neural network models used in supervised learning on graphs. I found this to be a useful framework that provides a clear understanding of how different GNN models work and facilitates a quick grasp of the differences between them. Considering a node $v$ on the graph $G$, the update procedure for its vector representation $h_v$ is as follows:

$$m_v^{t+1}=\sum_{u\in \mathcal{N}(v)}M_t(h_v^t,h_u^t,e_{vu})$$
$$h_v^{t+1}=U_t(h_v^t,m_v^{t+1})$$

where
- $u$ is the neighbor of $v$, and we use $\mathcal{N}(v)$ to represent all its neighbors
- $e_{vu}$ is optional, which represents the edge feature
- $M_t$ is the message function, $m_v^{t+1}$ is the aggregation result of all message from neighbors
- $U_t$ is the vertex update function

After updating the vector representations of all nodes on the graph, we may need to perform graph-level classification tasks, which correspond to the following formula in the MPNN framework:
$$\hat y=R(\{h_v^T|v\in G\})$$

where
- $R$ is the readout function, which computes a feature vector for the whole graph (if you're doing a graph-level classification problem)


## What's GAT
> 🧐 I have found that linking the formulas with code can help with understanding. Therefore, I will provide relevant code(with `...` representing omitted parts). The code is sourced from the [official `GATConv` module](https://docs.dgl.ai/en/latest/_modules/dgl/nn/pytorch/conv/gatconv.html#GATConv) in DGL.

> 🧐 We can stack multiple GAT modules easily. The following discussion is from the perspective of a specific node $v$ in a particular layer $l$.

#### Step 1. Apply linear transformation to all nodes
$$h_v^{l}=W^lh_v^{l}$$

Let's assume that the length of the vector representation for each node is denoted as $F$. In the first step, a linear transformation is applied to the vector of **each node** on the graph, where $W\in\mathcal{R}^{F'\times F}$. Therefore, the length of each node is updated with a length of $F'$. To distinguish vectors from different layers, superscript $l$ is used to indicate that it belongs to the $l$-th layer. Note that **within the layer $l$, all nodes share the same weight matrix $W^l$**


> 📒 **Note that the $h_v^l$ or $h_u^l$ mentioned later have undergone linear transformations**.

```python
class GATConv(nn.Module):
    def __init__(self, ...):
        ...
        self.fc = nn.Linear(
            self._in_src_feats, out_feats * num_heads, bias=False
        )
        ...

    def forward(self, graph, feat, ...):
        """
        Args
        ----
            feat: (N, *, D_in) where D_in is the size of input feature

        Returns
        -------
            torch.Tensor
                (N, *, num_heads, D_out)

        """
        ...
        src_prefix_shape = dst_prefix_shape = feat.shape[:-1]
        h_src = h_dst = self.feat_drop(feat)
        # h_src: (N, *, D_in)
        feat_src = feat_dst = self.fc(h_src).view(
            *src_prefix_shape, self._num_heads, self._out_feats
        )
        # feat_src/feat_dst: (N, *, num_heads, out_feat)
        ...
```

Note that in the above code, the presence of two identical `feat_src` and `feat_dst` variables in DGL is due to the adoption of a mathematically equivalent but computationally more efficient implementation. This will be explained later


#### Step 2. Compute the attention
$$e_{vu}^l=LeakyReLU\Big((a^l)^T[h_v^{l}||h_u^{l}]\Big)$$

$$\alpha_{vu}^l=Softmax_u(e_{vu}^l)$$

The second step is to compute the attention between the central node $v$ and all its neighboring nodes. In the above formula:
- $e_{vu}^l$ represents the attention coefficient. The paper mentions that different attention computation methods can be used. In the GAT paper, the authors chose to use a single-layer feedforward neural network(FNN) to compute the attention[^2]. **Note that the $e_{vu}^l$ here is unrelated to $e_{vu}$ in MPNN; it just happens to have similar notation**
- $||$ denotes the concatenation operation. It means that we concatenate the vector representations of the central node and its corresponding neighbor nodes, resulting in a vector of length $2F'$, as indicated by $[h_v^{l}||h_u^{l}]$ in the formula. This concatenated vector is then fed into the aforementioned single-layer FNN, represented as $(a^l)^T[h_v^{l}||h_u^{l}]$, where $(a^l)^T$ refers to the learnable parameters of the single-layer FNN in the $l$-th layer
- $LeakyReLU$ is the activation function
- Finally, we apply Softmax on all neighbors of node $v$ to normalize the attention coefficients

> 🤔️ Step 1 and 2 correspond to the computation of $m_v^{t+1}$ in the MPNN framework.
##### Multi-head attention
Just as Transformers have multi-head attention, the authors of GAT also employ the mechanism of multi-head attention during node updates:
$$h_v^{l+1}= ||^{K^l} \sigma(\sum_{u\in\mathcal{N}(i)}\alpha_{vu}^{(k,l)}W^{(k,l)}h_u^{l})$$

The notations are getting more complex, but with careful consideration, they can still be understood. The superscript $(k,l)$ indicates the $k$-th head in the $l$-th layer. Here, $K^l$ represents the number of heads in the $l$-th layer. The meaning of the above formula is that each head will compute a vector representation, and these vectors from different heads will be concatenated together.


```python
class GATConv(nn.Module):
    def __init__(self, ...):
        ...
        self.attn_l = nn.Parameter(
            th.FloatTensor(size=(1, num_heads, out_feats))
        )
        self.attn_r = nn.Parameter(
            th.FloatTensor(size=(1, num_heads, out_feats))
        )
        self.leaky_relu = nn.LeakyReLU(negative_slope)
        ...

    def forward(self, graph, feat, ...):
        """
        Args
        ----
            feat: (N, *, D_in) where D_in is the size of input feature

        Returns
        -------
            torch.Tensor
                (N, *, num_heads, D_out)

        """
        # feat_src/feat_dst: (N, *, num_heads, out_feat)
        el = (feat_src * self.attn_l).sum(dim=-1).unsqueeze(-1)
        er = (feat_dst * self.attn_r).sum(dim=-1).unsqueeze(-1)
        # el/er: (N, *, num_heads, 1)
        graph.srcdata.update({"ft": feat_src, "el": el})
        graph.dstdata.update({"er": er})
        graph.apply_edges(fn.u_add_v("el", "er", "e"))
        e = self.leaky_relu(graph.edata.pop("e"))
        # e: (N, *, num_heads, 1)

        # normalization
        graph.edata["a"] = self.attn_drop(edge_softmax(graph, e))
        # a: (N, *, num_heads, 1)

        # weighted sum
        graph.update_all(
            # ft: (N, *, num_heads, out_feat)
            # a: (N, *, num_heads, 1)
            # m: (N, *, num_heads, out_feat)
            fn.u_mul_e("ft", "a", "m"), 
            fn.sum("m", "ft")
        )
        rst = graph.dstdata["ft"]
        # rst: (N, *, num_heads, out_feat)
        ...
```
The implementation of `GATConv` in DGL is based on this equation:
$$a^T[h_v||h_u]=a_l^Th_v+a_r^Th_u$$

Why it is more efficient?
1. We don't need to store $[h_v||h_u]$ on edges (DGL will store messages in edge)
2. The addition could be optimized with DGL's built-in function `u_add_v`
#### Step 3. Aggregation

The final way of updating the nodes is by taking the weighted sum of the neighbor's vector representations using the corresponding attention scores. It can be expressed as follows:
$$h_v^{l+1}=\sigma(\sum_{u\in \mathcal{N}(i)}\alpha_{vu}W^{l}h_u^{l})$$

> 🤔️ The above corresponds to the computation of $h_v^{t+1}$ in the MPNN framework. It is worth noting that when calculating $h_t^{l+1}$ in GAT, the previous layer representation $h_t^l$ is not used. Additionally, GAT was proposed to address node classification problems on graphs and does not involve any graph readout operations.

In the GAT paper, the authors intended to use GAT for node-level classification tasks[^2]. Suppose we stack $L$ layers of GAT. It would be unreasonable to use concatenation in the final(prediction) layer. Therefore, in the last GAT layer, the authors take the average of multiple heads before applying the activation function. If the activation function used here is Softmax, it can be directly used for node classification[^2]. The formula is as follows:
$$final\ embedding\ of\ h_v= \sigma\Big(\frac{1}{K^L}\sum_{k=1}^{K^L}\sum_{u\in\mathcal{N}(i)}\alpha_{vu}^{(k,L)}W^{(k,L)}h_u^{L}\Big)$$

## Implementation and usage

> 🤔️ [DGL](https://www.dgl.ai/)'s design is based on the MPNN framework, but their formulas are slightly different. They also introduce an aggregation function, denoted as $\rho$, which determines how a node aggregates all the information received from its neighbors. **I thought their formulas are more generalized**. They have thoughtfully provided a tutorial on how to use DGL's MPNN-related functions, which can be found [here](https://docs.dgl.ai/en/latest/guide/message.html) 👍👍👍

As for the implementation of GAT, the DGL offers [GATConv](https://docs.dgl.ai/en/latest/generated/dgl.nn.pytorch.conv.GATConv.html#dgl.nn.pytorch.conv.GATConv). The DGL team also write a good tutorial about using the built-in `message_func` and `reduce_func` to [implemente GAT manually](https://docs.dgl.ai/en/latest/tutorials/models/1_gnn/9_gat.html)

Please refers to [here](https://github.com/dmlc/dgl/blob/master/examples/core/gat/train.py) to see a full training example

## Wrap up
Above is how the GAT can be explained using the MPNN framework, with the inclusion of DGL source code. Using attention to compute importances between nodes appears to be a natural approach and can be seen as a generalization of GCN. GAT is capable of learning local structural representations of graphs effectively, and the attention computation can be parallelized, making it highly efficient. Cheers! 🍻🍻🍻


## Refs

[^1]: Gilmer J, Schoenholz S S, Riley P F, et al. Neural message passing for quantum chemistry[C]//International conference on machine learning. PMLR, 2017: 1263-1272. [arXiv](https://arxiv.org/abs/1704.01212)
[^2]: Veličković P, Cucurull G, Casanova A, et al. Graph attention networks[J]. arXiv preprint arXiv:1710.10903, 2017. [arXiv](https://arxiv.org/abs/1710.10903)

