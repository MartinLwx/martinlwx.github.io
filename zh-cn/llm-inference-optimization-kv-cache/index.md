# LLM 推理加速 - KV Cache


## 背景
LLM 用于推理的时候就是不断基于前面的所有 token 生成下一个 token

*假设现在已经生成了 $t$ 个 token，用 $x_{1:t}$ 表示。在下一轮，LLM 会生成 $x_{1:t+1}$，注意他们的前 $t$ 个 token 是一样的*

$$x_{1:t+1}=\text{LLM}(x_{1:t})$$

下一步也是相似的

$$x_{1:t+2}=\text{LLM}(x_{1:t+1})$$

概括来说，**每一轮用上一轮的输出当成新的输入让 LLM 预测**，一般这个过程会持续到输出达到提前设定的最大长度或者是 LLM 自己生成了特殊的结束 token

## KV Cache 原理

> LLM 的推理过程很好理解，但是这个简单的实现存在一个问题——**存在不少的重复计算**导致计算效率不是很高🧐

只需要看 LLM 的连续两次前向传播推理计算就很好理解为什么说存在重复计算了

*比如考虑下面这一步*

$$
x_{1:t+1}=\text{LLM}(x_{1:t})
$$

LLM 的输入是 $x_{1:t}$，先只看最后一个 token $x_t$，它的 query 向量会和前面的每个 token 以及自己产生的 key 向量计算内积

$$\mathbf q_{t}^T\mathbf k_{1},\mathbf q_{t}^T\mathbf k_{2},...,\mathbf q_{t}^T\mathbf k_{t}$$

*然后看下一步*

$$
x_{1:t+2}=\text{LLM}(x_{1:t+1})
$$

LLM 的输入是 $x_{1:t+1}$，看最后一个 token $x_{t+1}$，它的 query 向量也会和前面的每个 token 以及自己产生的 key 向量计算内积

$$\mathbf q_{t+1}^T\mathbf k_{1},\mathbf q_{t+1}^T\mathbf k_{2},...,\mathbf q_{t+1}^T\mathbf k_{t+1}$$

此时考虑 $x_{1:t+1}$ 的 token $x_t$，它也是类似的步骤

$$\mathbf q_{t}^T\mathbf k_{1},\mathbf q_{t}^T\mathbf k_{2},...,\mathbf q_{t}^T\mathbf k_{t}$$

**可以看到，这个计算完全和上一轮的计算重复了**，对于在 $x_t$ 之前的 token 也是这个道理。我们需要**重新计算**得到 $x_{1:t}$ 的所有 key 向量和 value 向量，而这些向量的值**其实是不会变的**🧐


> 🤔 那么我们只需要把之前 token 的 key 向量和 value 向量都缓存起来，那么就没有必要重复计算了，这就是 KV Cache 的核心思想

在运用了 KV Cache 之后，除了第一轮以外的每一轮，我们**都只需要关注输入的最后一个 token**，用这个 token 计算得到它的 query, key, value 向量，然后拿着这三个新向量和之前**所有缓存的 key/value 向量**计算自注意力

*我画了一个示意图来帮助理解开启 KV Cache 之后发生了什么，考虑 LLM 从无到有开始生成 4 个 token，其中蓝色的部分表示 KV Cache 里面缓存的值；红色则表示没有被缓存*

{{< figure src="/img/kv_cache_4_steps.png" >}}

## KV Cache 的效率分析

KV Cache 加速推理的原理是：在自注意力层，本来每次要做矩阵乘法 $\mathbf Q\mathbf K^T$，现在因为 KV Cache 的存在，我们不需要整个 $\mathbf Q$ 和 $\mathbf K^T$ 做矩阵乘法，只需要**每次输入的最后一个 token 的 query 向量** $\mathbf q$ 和 $\mathbf K$ 做向量-矩阵乘法，之后更新 KV Cache 缓存即可

采用了 KV Cache 的话 LLM  的推理过程可以看成 **2 个阶段**
1. 第一次迭代的时候，此时 KV Cache 为空，所有的输入的 token 都需要为其计算 key, value, query 向量，其中 key 和 value 会被缓存起来
2. 后续的每一次迭代，**只需要为新的 token 计算 key、value、query**，并更新 KV Cache

KV Cache 加速推理的代价是显存占用会变高，所以它是**空间换时间**的办法，关于开不开 KV Cache 的显存占用的对比可以看 [这里](https://discuss.huggingface.co/t/generate-using-k-v-cache-is-faster-but-no-difference-to-memory-usage/31272)。我在这里放一个总结：
- 用 KV Cache - `2 * hidden_size * num_layers * decoder_length`
- 不用 KV Cache - `2 * hidden_size * 1 * decoder_length`


## KV Cache API

Huggingface 的 `model.generate` API 有一个参数为 `use_cache`，可以用来控制是否开启 KV Cache，**这个选项是默认打开的**[^1]

## 参考

[^1]: [GenerationConfig](https://huggingface.co/docs/transformers/v4.34.0/en/main_classes/text_generation#transformers.GenerationConfig)


