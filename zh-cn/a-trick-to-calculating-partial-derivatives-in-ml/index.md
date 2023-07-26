# 机器学习求解梯度的小技巧


## 引言
也许你和我一样在求解机器学习的梯度时有各种困难，即使看着相关的 [Cookbook](https://www.math.uwaterloo.ca/~hwolkowi/matrixcookbook.pdf) 一边推导依然是有困惑，今天我要分享的是最近[学习到](https://youtu.be/JLg1HkzDsKI)的一个实用技巧：**在机器学习中，求解偏导数的时候可以先全部看成标量处理，最后让维度匹配即可**

> **免责声明**：运用如上的技巧并**不保证**梯度是正确的，可能维度是对的，但是梯度是错的，强调一下**为了确保梯度计算正确请做好梯度检查**

## 应用
> 💡 大写的黑色粗体字母表示矩阵，不加粗的字母都是标量

### 反向传播矩阵形式推导
在之前的 [反向传播推导]({{< ref "/content/posts/backpropagation-tutorial.zh-cn.md" >}}) 过程中，我从标量角度来推导了反向传播，因为这个比较好理解，但如果你尝试**实现**反向传播或者是前向传播，你会发现都是用**矩阵形式**，我们总是把它变成矩阵乘法，因为矩阵乘法会快很多。因此需要弄清楚「反向传播的矩阵形式」是如何进行的，下面我将利用前面提到的技巧来进行推导

> 为了公式简洁，下面省略了 bias 只考虑 weight


考虑一个简单的 $L$ 层的 MLP 模型，用 $\mathbf Z^l$ 表示 $l$ 层的输出，特别的，将输入层也用 $\mathbf Z$ 表示，有
$$\mathbf Z^0=\mathbf X$$

其中 $\mathbf X\in\mathcal{R}^{m\times d}$，$m$ 为样本数量，$d$ 为每个样本的特征长度

模型 $f_\theta$ 的输出是 $$f_\theta(\mathbf X)=\mathbf Z^{L}$$
其中 $\theta$ 表示模型的可学习参数

相邻两层之间的关系是 $$\mathbf Z^{l+1}=\sigma_{l+1}(\mathbf Z^l\mathbf W^{l+1}),l=0,...,L-1$$
其中 $\sigma_{l+1}$ 为 $l+1$ 层的激活函数

维度信息如下:
$$\mathbf Z^l\in\mathcal{R}^{m\times n_l}$$
$$\mathbf W^{l+1} \in \mathcal R^{n_l\times n_{l+1}}$$

其中 $n_l$ 用于表示 $l$ 层的神经元个数

---

我们想要确定损失 $J$ 对模型任意可学习参数的梯度（标量对矩阵求导），这样才能用梯度下降算法更新可学习参数，考虑我们要求解 $\mathbf W^l$ 的梯度
$$
\frac{\partial J}{\partial \mathbf W^l}=\frac{\partial J}{\partial\mathbf Z^{L}}\cdot \frac{\partial \mathbf Z^{L}}{\partial\mathbf Z^{L-1}}\cdot ...\cdot \frac{\partial \mathbf Z^{l+1}}{\partial\mathbf Z^{l}}\cdot\frac{\partial \mathbf Z^{l}}{\partial\mathbf W^l}
$$

🤔️ 那如果求解的是关于 $\mathbf W^{l+1}$ 的梯度呢？
$$
\frac{\partial J}{\partial \mathbf W^{l-1}}=\frac{\partial J}{\partial\mathbf Z^{L}}\cdot \frac{\partial \mathbf Z^{L}}{\partial\mathbf Z^{L-1}}\cdot ...\cdot \frac{\partial \mathbf Z^{l+1}}{\partial\mathbf Z^{l}} \cdot \frac{\partial \mathbf Z^{l}}{\partial\mathbf Z^{l-1}}\cdot\frac{\partial \mathbf Z^{l-1}}{\partial\mathbf W^{l-1}}
$$

你会发现，**不同参数的梯度公式存在大量共同的部分**，因此我们可以引入额外一个记号 $\mathbf G^l$，表示损失对 $\mathbf Z^l$ 的梯度
$$\mathbf G^{l}=\frac{\partial J}{\partial \mathbf Z^{l}}$$

下面我们就可以推导 $\mathbf G^l$ 和 $\mathbf G^{l+1}$ 的关系

$$
\begin{equation}
\begin{aligned}
\mathbf G^{l} &=\frac{\partial J}{\partial \mathbf Z^{l+1}}\cdot\frac{\partial \mathbf Z^{l+1}}{\partial \mathbf Z^l} \\\\\\
&=\mathbf G^{l+1}\cdot \frac{\partial \mathbf Z^{l+1}}{\partial \mathbf Z^l} \\\\\\
&=\mathbf G^{l+1}\cdot \frac{\partial \sigma_{l+1}(\mathbf Z^{l}\mathbf W^{l+1})}{\partial \mathbf Z^{l}\mathbf W^{l+1}}\cdot\frac{\partial \mathbf Z^{l}\mathbf W^{l+1}}{\partial \mathbf Z^l} \\\\\\
&=\mathbf G^{l+1}\cdot \sigma'(\mathbf Z^{l}\mathbf W^{l+1})\cdot \mathbf W^{l+1}\ (cheat)
\end{aligned}
\end{equation}
$$

在上面的最后一行，我们用**把矩阵当成标量处理**直接求导，然后根据前面说的，接下来**让维度匹配**就可以，先来看上面的每个部分的维度
$$\mathbf G^{l+1}\in\mathcal{R}^{m\times n_{l+1}}$$
$$\sigma_{l+1}'(\mathbf Z^{l}\mathbf W^{l+1})\in\mathcal{R}^{m\times n_{l+1}}$$
$$
\mathbf W^{l+1}\in\mathcal{R}^{n_l\times n_{l+1}}
$$
我们想要得到大小为 $m\times n_1$ 的矩阵，因为
$$\mathbf G^l\in\mathcal{R}^{m\times n_l}$$

所以可以这么凑
$$\mathbf G^{l}=\Big (\mathbf G^{l+1}\odot\sigma_{l+1}'(\mathbf Z^{l}\mathbf W^{l+1})\Big )(\mathbf{W}^{l+1})^T=\Big (\mathbf G^{l+1}\odot\sigma_{l+1}'(\mathbf Z^{l+1})\Big )(\mathbf{W}^{l+1})^T$$

现在回到我们本来要做的事情——求解关于 $\mathbf W^l$ 的梯度：
$$\frac{\partial J}{\partial \mathbf W^l}=\mathbf G^{l}\cdot\frac{\partial \mathbf Z^l}{\mathbf W^l}$$

让我们来展开上面的公式
$$
\begin{equation}
\begin{aligned}
\frac{\partial J}{\partial \mathbf W^l}&=\mathbf G^{l}\cdot\frac{\partial \mathbf Z^l}{\mathbf W^l} \\\\\\
&= \mathbf G^{l}\cdot\frac{\partial \mathbf \sigma_{l+1}(\mathbf Z^{l-1}\mathbf W^l)}{\partial \mathbf Z^{l-1}\mathbf W^l}\cdot \frac{\partial \mathbf Z^{l-1}\mathbf W^l}{\partial\mathbf W^l} \\\\\\
&= \mathbf G^{l}\cdot\mathbf \sigma_{l+1}'(\mathbf Z^{l-1}\mathbf W^l)\cdot \mathbf Z^{l-1}(cheat)
\end{aligned} 
\end{equation}
$$

我们想要得到和 $\mathbf W^l$ 一样大小的矩阵：$(n_l, n_{l+1})$，整理一下上面的不同部分

$$
\frac{\partial J}{\partial \mathbf W^l}=(\mathbf Z^{l-1})^T\Big(\mathbf G^{l}\odot\mathbf \sigma_{l+1}'(\mathbf Z^{l-1}\mathbf W^l) \Big )\\
=(\mathbf Z^{l-1})^T\Big(\mathbf G^{l}\odot\mathbf \sigma_{l+1}'(\mathbf Z^l) \Big )\\
$$

利用 $\mathbf G^l$ 和 $\mathbf G^{l+1}$ 的关系，我们可以从 $\mathbf G^{l+1}$ 推导出 $\mathbf G^l$，然后也可以进一步计算 $\mathbf W^l$ 的梯度，也就是从后往前计算，这也就是反向传播的含义
> 🤔️ 注意 $\mathbf G^l$ 的计算和 $\frac{\partial J}{\partial \mathbf W^l}$ 用到了 $\mathbf Z^{l-1}, \mathbf Z^{l}, \mathbf Z^{l+1}$，也就是不同层的激活函数的输出，这意味着**在反向传播的时候我们需要缓存前向传播的值**，缓存意味着需要消耗内存，所以这就是为啥模型越大，GPU 就需要更多的显存

### 线性回归矩阵形式梯度推导
之前在 [线性回归]({{<ref "/content/posts/linear-regression-model-guide-theory.zh-cn.md">}}) 里面我们需要求解下面这个公式
$$
\begin{aligned} 
\frac{\partial}{\partial \theta}\ J(w,b) 
&= \frac{\partial}{\partial \theta}\ \frac{1}{2m}(\mathbf X\theta - \vec{y})^T(\mathbf X\theta - \vec{y}) \\\\\\
\end{aligned} 
$$
利用维度分析的技巧，可以这么推导
$$
\begin{equation}
\begin{aligned} 
\frac{\partial}{\partial \theta}\ J(w,b) 
&= \frac{\partial}{\partial \theta}\ \frac{1}{2m}(\mathbf X\theta - \vec{y})^T(\mathbf X\theta - \vec{y}) \\\\\\
&= \frac{1}{2m}\frac{\partial(\ \mathbf X\theta - \vec{y})^T(\mathbf X\theta - \vec{y}) }{\partial \mathbf X\theta-\vec y}\cdot \frac{\partial \mathbf X\theta-\vec y}{\partial \theta}\\\\\\
&= \frac{1}{2m}\cdot2(\mathbf X\theta-\vec y)\cdot\mathbf X\ (cheat)
\end{aligned} 
\end{equation}
$$

考虑维度信息
$$\mathbf X\theta-\vec y\in\mathcal{R}^{m\times 1}$$
$$\mathbf X\in\mathcal{R}^{m\times(n+1)}$$
我们想要得到的是跟 $\theta$ 一样维度大小的：
$$\theta\in\mathcal{R}^{(n+1)\times 1}$$
因此直接凑出来就好：
$$\frac{1}{m}\mathbf X^T(\mathbf X\theta-\vec y)$$

## 总结
尽管这是一个**不严格**的技巧，像是作弊一样😨，但我发现还蛮**实用**的✌️，学会这个技巧之后，机器学习里面的公式推导会轻松很多，当然别忘了利用梯度检查确保式子是正确的 :)


