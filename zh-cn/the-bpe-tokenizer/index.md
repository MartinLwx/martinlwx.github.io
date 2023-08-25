# BPE 分词解密 - 实现方法与示例讲解


## BPE 简介

在 NLP 里面，一个核心的问题是，如何对文本进行分词？从分类的角度上面来说，可以分为：
- Char level
- Word level
- Subword level

先看 Char level 分词，顾名思义，就是把文本拆分成一个个字符单独表示，*比如 `highest -> h, i, g, h, e, s, t`*，一个显然的好处是，Vocab 不会太大，Vocab 的大小为字符集的大小，也不会遇到 Out-of-vocabulary(OOV) 的问题，但是**字符本身并没有传达太多的语义**，而且**分词之后会有太多的 token**，*光是一个 highest 就可以得到 7 个 token，难以想象很长的文本分出来会有多少个*😨

再看 Word level 分词，Word level 分词一般通过空格或者标点符号来把文本分成一个个单词，这样分词之后的 token 数量就不会太多，*比如 `Today is a good day` -> `Today, is, a, good, day`*。但 Word level 分词也有问题，*比如英文中的 `high, higher, highest` 这三个单词显然语义相似，因为另外两个只是比较级，但是 Word level 分词会把他们看成 3 个单独的单词*

🤔️ 那么是否存在一种折中的办法，使得我们*大概率*不会遇到 OOV 的问题；分词得到的 token 数量又不能太多；而且分词能够考虑到复合词、时态变化、单复数呢？这就是 Subword level 做的事情，*仍然是刚才的例子，根据英语的词根原理的话，我们可以把 `higher` 划分为 `high, er`，`highest` 划分为 `highest`*，所以 **Subword level 分词就是把一个单词用多个子词表示**，今天要介绍的 BPE 就属于 Subword level 的一种

BPE 的全称是 **B**yte **P**air **E**ncoding，这里有个细节值得思考，**什么是 Byte pair**？ASCII 编码的话任何字符都是 1 byte，但如果是 utf-8 编码呢？一个字符不一定是 1 byte，它可以是 3 bytes 也可以是 4 bytes，🤔️ 那如果 BPE 用在 utf-8 的文本里面，byte pair 又是什么东西？所以我感觉这里是有点歧义性的，因此**更好的理解方式也许是，byte pair 其实是 char pair，这里的 char 是 utf-8 的 char**

### BPE 训练流程

然后开始正式介绍 BPE 算法的训练流程，假设我们手头有一堆文档 $D$
1. 把每个文档 $d$ 变成一个个单词，*比如你可以简单用空格分词就好*
2. 统计每个单词 $w$ 在所有文档中的出现频率，并得到初始的字符集 `alphabet` 作为一开始的 Vocab（包括后面的 `</w>`）
3. 先将每个单词划分为一个个 utf-8 char，称为一个划分，*比如 `highest -> h, i, g, h, e, s, t`*
4. 然后，在每个单词的划分最后面加上 `</w>`，*那么现在 `highest -> h, i, g, h, e, s, t, </w>`*
5. 重复下面步骤直到满足两个条件中的任意一个：1）Vocab 达到上限。2）达到最大迭代次数
    1. 找到**最经常一起出现的 pair**，并记录这个合并规则放在 merge table 里面，同时把合并之后的结果放到 Vocab 里面
    2. 更新所有单词的划分，*假设我们发现 `(h, i)` 最经常一起出现，那么 `highest -> hi, g, h, e, s, t, </w>`*

你可能会有下面 3 个疑惑：
1. 为什么要统计词频？因为统计词频会让找最经常出现的 pair 更简单
2. 为什么要加 `</w>`？因为我们希望能够还原输入，因此需要做个标记表示这是单词之间的边界
3. 如果多个 pair 的词频一样怎么处理？这个不同实现可能不一样，但在我看来*应该*关系不大

> 💡 由此你可以发现，BPE 算法合并最经常一起出现的 pair 的时候，并不会跨越单词

### BPE 应用流程

在 BPE 完成训练之后，我们会得到一个 merge table，也会得到一个 Vocab，假设现在要处理文本 `s`

1. 和训练的时候采用一样的方法，先把 `s` 拆分成一个个单词，每个单词拆分为一个个 utf-8 char
2. 遍历 merge table，并检查每个合并规则是否可以用来更新每个单词的划分，可以的话就合并更新

> 💡 这里有个细节，我们提取的 merge rule 其实是按照出现的频率降序，那么我们**按顺序遍历 merge table 就已经*隐含*了「优先合并最经常出现的 pair」这件事了，注意体会**

### BPE 例子

停留在算法不提供例子的话，经常还是会云里雾里，所以现在来结合一个例子看 BPE 是如何工作的

*比如语料库是*
```python
corpus = ["highest", "higher", "lower", "lowest", "cooler", "coolest"]
```

*这里跳过统计词频，因为每一个都是 1。先把每个单词变成一个个 utf-8 字符然后加上 `</w>`*
```python
{
    "highest": ["h", "i", "g", "h", "e", "s", "t", "</w>"],
    "higher": ["h", "i", "g", "h", "e", "r", "</w>"],
    "lower": ["l", "o", "w", "e", "r", "</w>"],
    "lowest": ["l", "o", "w", "e", "s", "t", "</w>"],
    "cooler": ["c", "o", "o", "l", "e", "r", "</w>"],
    "collest": ["c", "o", "o", "l", "e", "s", "t", "</w>"],
}
```
*可以看到 `(e, s)` 总共出现了 3 次，是最多次的，然后根据这个重新划分。注意这里 `(e, r)` 其实也有一样的出现频率，所以选 `(e, r)` 合并也是可以的*
```python
{
    "highest": ["h", "i", "g", "h", "es", "t", "</w>"],
    "higher": ["h", "i", "g", "h", "e", "r", "</w>"],
    "lower": ["l", "o", "w", "e", "r", "</w>"],
    "lowest": ["l", "o", "w", "es", "t", "</w>"],
    "cooler": ["c", "o", "o", "l", "e", "r", "</w>"],
    "collest": ["c", "o", "o", "l", "es", "t", "</w>"],
}
```
*接下来发现最多的是 `(es, t)`，更新划分*
```python
{
    "highest": ["h", "i", "g", "h", "est", "</w>"],
    "higher": ["h", "i", "g", "h", "e", "r", "</w>"],
    "lower": ["l", "o", "w", "e", "r", "</w>"],
    "lowest": ["l", "o", "w", "est", "</w>"],
    "cooler": ["c", "o", "o", "l", "e", "r", "</w>"],
    "collest": ["c", "o", "o", "l", "est", "</w>"],
}
```
*接下来发现最多的是 `(est, </w>)`，更新划分*
```python
{
    "highest": ["h", "i", "g", "h", "est</w>"],
    "higher": ["h", "i", "g", "h", "e", "r", "</w>"],
    "lower": ["l", "o", "w", "e", "r", "</w>"],
    "lowest": ["l", "o", "w", "est</w>"],
    "cooler": ["c", "o", "o", "l", "e", "r", "</w>"],
    "collest": ["c", "o", "o", "l", "est</w>"],
}
```
*接下来发现最多的是 `(e, r)`，更新划分*
```python
{
    "highest": ["h", "i", "g", "h", "est</w>"],
    "higher": ["h", "i", "g", "h", "er", "</w>"],
    "lower": ["l", "o", "w", "er", "</w>"],
    "lowest": ["l", "o", "w", "est</w>"],
    "cooler": ["c", "o", "o", "l", "er", "</w>"],
    "collest": ["c", "o", "o", "l", "est</w>"],
}
```
*接下来发现最多的是 `(er, </w>)`，更新划分*
```python
{
    "highest": ["h", "i", "g", "h", "est</w>"],
    "higher": ["h", "i", "g", "h", "er</w>"],
    "lower": ["l", "o", "w", "er</w>"],
    "lowest": ["l", "o", "w", "est</w>"],
    "cooler": ["c", "o", "o", "l", "er</w>"],
    "collest": ["c", "o", "o", "l", "est</w>"],
}
```
*后面还可以继续迭代更新，这里就不展开了，相信上面的例子已经够清楚了。而且到这一步，我们已经得到了 `er, est` 这两个有意义的后缀*

### BPE 的 Huggingface 实现
Huggingface 提供的 API 还挺简单的，**可以注意到 `CharBPETokenizer` 的 `Char`，也证明了前面我说的*
```python
from tokenizers import CharBPETokenizer

# Instantiate tokenizer
tokenizer = CharBPETokenizer()

tokenizer.train_from_iterator(
    corpus,
    vocab_size=17,
    min_frequency=2,
)
```

### 手动实现 BPE
**最好理解一个算法的办法永远都是尝试自己实现一个**。我这里按照前面描述的算法流程实现了一个 `BPE` 类，如果初始化的时候设置 `debug=False` 就可以看到整个 BPE 是如何更新的

*首先来看构造函数和用来训练的 `train` 方法*
```python
from collections import defaultdict, Counter
from pprint import pprint


class BPE:
    def __init__(
        self,
        corpus: list[str],
        vocab_size: int,
        max_iter: int | None = None,
        debug: bool = False,
    ):
        self.corpus = corpus
        self.vocab_size = vocab_size
        self.vocab = []
        self.word_freq = Counter()
        self.splits = {}  # e.g. highest: [high, est</w>]
        self.merges = {}  # e.g. [high, est</w>]: highest
        self.max_iter = max_iter
        self.debug = debug

    def train(self):
        """Train a BPE Tokenizer"""
        # count the word frequency
        for document in self.corpus:
            # split each document in corpus by whitespace
            words = document.split()
            self.word_freq += Counter(words)

        # initialize the self.splits
        for word in self.word_freq:
            self.splits[word] = list(word) + ["</w>"]

        if self.debug:
            print(f"Init splits: {self.splits}")

        alphabet = set()
        for word in self.word_freq:
            alphabet |= set(list(word))
        alphabet.add("</w>")

        self.vocab = list(alphabet)
        self.vocab.sort()

        cnt = 0
        while len(self.vocab) < self.vocab_size:
            if self.max_iter and cnt >= self.max_iter:
                break

            # find the most frequent pair
            pair_freq = self.get_pairs_freq()

            if len(pair_freq) == 0:
                print("No pair available")
                break

            pair = max(pair_freq, key=pair_freq.get)

            self.update_splits(pair[0], pair[1])

            if self.debug:
                print(f"Updated splits: {self.splits}")

            self.merges[pair] = pair[0] + pair[1]

            self.vocab.append(pair[0] + pair[1])

            if self.debug:
                print(
                    f"Most frequent pair({max(pair_freq.values())} times) "
                    f"is : {pair[0]}, {pair[1]}. Vocab size: {len(self.vocab)}"
                )

            cnt += 1

```
*流程还是挺清晰的，核心的几个函数实现如下*
```python
    ...
    def update_splits(self, lhs: str, rhs: str):
        """If we see lhs and rhs appear consecutively, we merge them"""
        for word, word_split in self.splits.items():
            new_split = []
            cursor = 0
            while cursor < len(word_split):
                if (
                    word_split[cursor] == lhs
                    and cursor + 1 < len(word_split)
                    and word_split[cursor + 1] == rhs
                ):
                    new_split.append(lhs + rhs)
                    cursor += 2
                else:
                    new_split.append(word_split[cursor])
                    cursor += 1
            self.splits[word] = new_split

            # if word_split != new_split:
            #     print(f"old: {word_split}")
            #     print(f"new: {new_split}")

    def get_pairs_freq(self) -> dict:
        """Compute the pair frequency"""
        pairs_freq = defaultdict(int)
        for word, freq in self.word_freq.items():
            split = self.splits[word]
            for i in range(len(split)):
                if i + 1 < len(split):
                    pairs_freq[(split[i], split[i + 1])] += freq

        return pairs_freq
```
*最后我们就可以写一个 `tokenize` 函数*
```python
    ...
    def tokenize(self, s: str) -> list[str]:
        splits = [list(t) + ["</w>"] for t in s.split()]

        for lhs, rhs in self.merges:
            for idx, split in enumerate(splits):
                new_split = []
                cursor = 0
                while cursor < len(split):
                    if (
                        cursor + 1 < len(split)
                        and split[cursor] == lhs
                        and split[cursor + 1] == rhs
                    ):
                        new_split.append(lhs + rhs)
                        cursor += 2
                    else:
                        new_split.append(split[cursor])
                        cursor += 1
                assert "".join(new_split) == "".join(split)
                splits[idx] = new_split

        return sum(splits, [])
```

*尝试用自己写的 BPE 对刚才的 `corpus` 进行分词*
```python
bpe = BPE(corpus, vocab_size=17, debug=False)
bpe.train()
bpe.tokenize(" ". join(corpus))
```

输出是：
```python
['h', 'i', 'g', 'h', 'est</w>',
 'h', 'i', 'g', 'h', 'er</w>',
 'l', 'o', 'w', 'er</w>',
 'l', 'o', 'w', 'est</w>',
 'c', 'o', 'o', 'l', 'er</w>',
 'c', 'o', 'o', 'l', 'est</w>']
```

🤔️ 说明代码写对了，而且多亏了 `</w>`，我们可以很清楚看到单词之间的边界，也可以还原出本来的输入


### 总结

BPE 算法简单而且很好用，**但是当深入到实现的时候，你发现会有不少细节问题，但正是因为接触到这些细节才使得对 BPE 的理解更加深刻**

这里可以讨论一下 BPE 的局限性，那就是你会发现**把文档变成一个个单词我们这里用的是空格划分**，但是像中文的话，空格并不是单词之间的边界，这就会让事情变得比较棘手了起来

