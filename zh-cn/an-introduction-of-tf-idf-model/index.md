# TF-IDF 模型


## 什么是 TF-IDF 模型

在之前的[文章]({{<ref "/content/posts/an-introduction-of-bag-of-word-model.zh-cn.md">}})中谈到了词袋模型，也讲到了它的许多不足，在今天的这篇文章中，我们要尝试解决词袋模型的**缺点之一：每个词的重要性是一样的**

> 💡 那么，核心问题就是————**如何定义「单词的重要性」这个概念**？

一个想法是：一个单词在**一个文档**里面出现得越频繁，则这个单词**对于这个文档来说**越重要。*比如一篇讨论狗的文章，大概率文章里面会出现很多「狗」，即词频高的单词反映了文档的主题*

但如果这个词在语料库的所有文档中都出现得很频繁呢？*比如「的」*，在每个文档中，它的词频应该都不低，那能说这个单词更重要吗？显然是不行的，「的」更多是服务于语法结构而没包含太多的语义信息。于是我们有了另外一个线索：如果一个单词在**每个文档**中的词频都很高，那么这个单词的重要性应该就没那么重要

因此**一个合理的解决方案应该考虑单词在「单个文档」的词频，但也应该考虑它在「多个文档」的词频**，TF-IDF 就兼顾了这两点

概括来说，TF-IDF 的“直觉”理解就是————**相似的文档用的词*也许*是差不多的，而不同的单词的重要性应该是不一样的**


## TF-IDF 模型细节

TF-IDF 由 2 个部分组成：词频（Term frequency，TF） + 逆文档频率（Inverse document frequency，IDF），先说 TF 再说 IDF

### TF

所谓 TF，可以看成是关于文档 $d$ 和单词 $w$ 的函数 $\text{TF}(w, d)$，计算公式如下：

$$\text{TF}(w, d)=\frac{\text{frequency of}\ w\ \text{in}\ d}{\text{word counts of } d}$$

也就是统计文档 $d$ 的单词 $w$ 的词频，然后除以对应文档 $d$ 的总的单词数量即可

> 🐛 在 Scikit-Learn 框架中，TF 的计算不大一样，**它并没有除文档的总的单词数量**。其实除以文档的总的单词数量是为了让一个文档 $d$ 的所有单词的 $\text{TF}(w, d)$ 加起来为一，起到一个归一化的作用。Scikit-Learn 是在 TF-IDF 计算完成之后才做归一化的。后面我们会用一个例子证明这点

### IDF

在写下 IDF 的具体公式之前，别忘了我们的目的————降低每个文档都出现的常见词的重要性。所以 IDF 是一个关于单词 $w$ 和语料库 $corpus$ 的函数

$$
\text{IDF}(w, corpus)=log\ \frac{\text{document count of }corpus}{1+\text{count of document which contains }w}
$$

注意分母位置的 1 避免了除 0 的情况

> 🤔️ corpus 一般是固定下来不变的，可以看成一个常量而不是自变量，那么就可以把 IDF 看成是只跟单词 $w$ 有关系

> 💡 注意一个问题，这里的 $log$ 是 $log_2$ 还是 $log_{10}$ 还是 $ln$ ？不同的框架的实现可能会有差异，*Scikit-Learn 用的是自然对数 $ln$*

> 🐛 在 Scikit-Learn 框架中，IDF 的计算跟上面的公式并不一样，默认情况下 Scikit-Learn 的 IDF 是这样的[^1]

$$
\text{IDF}(w, corpus)=log\ \frac{1 + \text{document count of }corpus}{1+\text{count of document which contains }w} + 1
$$

🤔️ 我个人理解的话就是，**Scikit-Learn 做的改动使得 $\text{IDF}(t)$ 不可能小于 1**，而本来的公式如果一个单词 $w$ 在每个文档都出现了的话，$\text{IDF}(w)$ 算出来会是负的，因此 Scikit-Learn 的改动感觉更实用了，不同单词的 IDF 比较起来就很直观

### TF-IDF

把 TF 和 IDF 相乘，就得到了最后的 TF-IDF 表达式：
$$
\text{TF-IDF}(w, d, corpus)=\text{TF}(w, d) * \text{IDF}(d, corpus)
$$


## Scikit-Learn 的 TF-IDF

虽然 TF-IDF 实现起来很简单，但是实际用的时候，你多半还是不会自己实现 TF-IDF，而是直接用成熟的 Scikit-Learn 提供的 API。这里我们来研究一下 Scikit-Learn 怎么计算 TF-IDF。我们继续沿用 Scikit-Learn 的官方例子


```python
toy_corpus = [
    'This is the first document.',
    'This is the second second document.',
    'And the third one.',
    'Is this the first document?',
]

tokenized_toy_corpus = [
    ['this', 'is', 'the', 'first', 'document'],
    ['this', 'is', 'the', 'second', 'second', 'document'],
    ['and', 'the', 'third', 'one'],
    ['is', 'this', 'the', 'first', 'document']
]
```
让我们先来看看 Scikit-Learn 里面如何计算 TF-IDF

```python
from sklearn.feature_extraction.text import TfidfVectorizer
# set norm=None for comparison
vectorizer = TfidfVectorizer(norm=None)
X = vectorizer.fit_transform(toy_corpus)
```

通过 `X.toarray()` 就可以拿到 TF-IDF 矩阵，*如下所示*（注意我关闭了归一化，所以一个文档内的不同单词的 TF-IDF 值加起来不为 1）

|           | and     | document | first   | is      | one     | second  | the | third   | this    |
| --------- | ------- | -------- | ------- | ------- | ------- | ------- | --- | ------- | ------- |
| document1 | 0.0     | 1.22314  | 1.51082 | 1.22314 | 0.0     | 0.0     | 1.0 | 0.0     | 1.22314 |
| document2 | 0.0     | 1.22314  | 0.0     | 1.22314 | 0.0     | 3.83258 | 1.0 | 0.0     | 1.22314 |
| document3 | 1.91629 | 0.0      | 0.0     | 0.0     | 1.91629 | 0.0     | 1.0 | 1.91629 | 0.0     |
| document4 | 0.0     | 1.22314  | 1.51082 | 1.22314 | 0.0     | 0.0     | 1.0 | 0.0     | 1.22314 |

下面则是之前词袋模型的输出

|           | and | document | first | is  | one | second | the | third | this |
| --------- | --- | -------- | ----- | --- | --- | ------ | --- | ----- | ---- |
| document1 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |
| document2 | 0   | 1        | 0     | 1   | 0   | 2      | 1   | 0     | 1    |
| document3 | 1   | 0        | 0     | 0   | 1   | 0      | 1   | 1     | 0    |
| document4 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |


🤔️ *对比一下两者你会发现，现在 document1 文档里面 `document`，`first` 这两个单词的重要性不一样了：`document` 的 TF-IDF 是 `1.22314`，而 `first` 的 TF-IDF 是 `1.51082`，因为语料库中包含他们的文档数量并不相同。而词袋模型认识不到这点，认为两者都是 `1`*


通过 vectorizer 的 `idf_` 属性我们就能够拿到 Scikit-Learn 计算出来的每个单词的 IDF

```python
print(vectorizer.idf_)
# [1.91629073 1.22314355 1.51082562 1.22314355 1.91629073 
#  1.91629073 1.         1.91629073 1.22314355]
```

🤔️ 此时你要是把这个 IDF 向量乘以词袋模型输出的矩阵（注意向量会被广播），你就得到了 Scikit-Learn 算出来的 TF-IDF 矩阵，这验证了前面我们说的：
- Scikit-Learn 直接用词袋模型的输出当成 TF
- Scikit-Learn 的 IDF 计算跟标准的不一样

## 手动实现 TF-IDF

手动实现也并不复杂，下面假设语料库或者是文档都是分词好的，采用和 Scikit-Learn 一样的 TF-IDF 计算方式

> 🐛 注意下面的代码未经优化，只是为了演示

```python
import math


def TF(word: str, tokenized_document: list[str]) -> float:
    return tokenized_document.count(word)


def IDF(word: str, tokenized_corpus: list[list[str]]) -> float:
    doc_count_contains_word = 0
    for doc in tokenized_corpus:
        if word in doc:
            doc_count_contains_word += 1

    return math.log((1 + len(tokenized_corpus)) / (1 + doc_count_contains_word)) + 1


def TF_IDF(
    word: str, tokenized_document: list[str], tokenized_corpus: list[list[str]]
) -> float:
    return TF(word, tokenized_document) * IDF(word, tokenized_corpus)
```

## 总结
在这篇文章中，我们研究了 TF-IDF 如何对词袋模型做出了改进：**额外考虑一个单词在所有文档中的词频来调整它的重要性**。通过分析官方的例子，我们也验证了确实单词的重要性现在是不一样的了，这是对词袋模型一个不错的改进

从实现的角度来说，TF-IDF 的计算只需要遍历语料库一次即可，在遍历的过程中额外维护几个变量：每个单词出现在了多少个文档里面；处理的文档数量等，[Gensim](https://radimrehurek.com/gensim/index.html) 的 `Dictionary` 类就是这么设计的，感兴趣的话可以看下源代码

## 参考

[^1]: [TF-IDF term weighting](https://scikit-learn.org/stable/modules/feature_extraction.html#tfidf-term-weighting)

