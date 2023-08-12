# 词袋模型简介


## 什么是词袋模型

在 NLP 中，我们需要将文档（document）表示为向量，这是因为机器学习只能够处理数字。也就是说，我们要找到下面这么一个*神奇*的函数：

$$
f(\text{document}) = vector
$$

今天要讨论的是词袋模型（bag-of-word, BoW），词袋模型可以让我们把输入的文档转变成一个向量表示

> 💡 尽管词袋模型在 2023 年已经过时了，我仍然鼓励你学习词袋模型，并且思考下面几个**重要问题**：
- Motivation 是什么？
- 优缺点是什么？
- 如何把它变得更好？

### 词袋模型的 motivation 和直观理解
在了解词袋模型细节之前，我想先给你一个词袋模型可能是有用的**直观理解** —— **相似的文档用的词*也许*是差不多的**

你可能持反对意见，并且可以给出许多反例。我承认，这也是为什么我们需要更加强大的模型 :)

### 词袋模型细节

构造一个词袋模型只需要做两件事情
1. 创建词汇表（Vocab），每个单词都在词汇表里面有一个独特的 ID（一般从 `0` 开始）。**注意词袋模型输出的向量的长度等于词汇表的大小**
2. 遍历语料库中的每个文档，把这个文档新出现的单词添加到词汇表里面

在构造好词袋模型之后，就可以把任何文档都转变成向量表示了。方法也很简单，只要统计文档中每个单词出现的次数。值得一提的是，对于文档中不在词汇表里面的单词，**一般直接就忽略不计**

让我们来结合一个简单例子理解[^1]


```python
toy_corpus = [
    'This is the first document.',
    'This is the second second document.',
    'And the third one.',
    'Is this the first document?',
]
```

删除标点符号，然后用空格分词，同时把所有的单词变成小写，预处理之后，我们就可以得到


```python
tokenized_toy_corpus = [
    ['this', 'is', 'the', 'first', 'document'],
    ['this', 'is', 'the', 'second', 'second', 'document'],
    ['and', 'the', 'third', 'one'],
    ['is', 'this', 'the', 'first', 'document']
]
```

为了简单，我们这里考虑把所有文档的所有单词都包括在词汇表里面


```python
flatten_list_as_set = set(sum(tokenized_toy_corpus, start=[]))
print(f"the toy vocab size: {len(flatten_list_as_set)}")
```

    the toy vocab size: 9


> 💡 一个用 `sum` 函数摊平 list 的小技巧 :D

然后，让我们给词汇表里面的每个单词一个独特的 ID


```python
toy_token2id = {}
for token in sorted(flatten_list_as_set):
    toy_token2id[token] = len(toy_token2id)

print(toy_token2id)
```

    {
        'and': 0, 'document': 1, 'first': 2, 
        'is': 3, 'one': 4, 'second': 5,
        'the': 6, 'third': 7, 'this': 8
    }

可以看到，词汇表的大小是 `9`，那么我们也就知道，每个文档可以用一个长度为 `9` 的向量表示

让我们手动计算一下每个文档的词袋模型向量来检查我们是否理解正确


```python
BoW_matrix = []
for document in tokenized_toy_corpus:
    temp = [0] * 9
    for token in document:
        temp[toy_token2id[token]] += 1
    BoW_matrix.append(temp)
print(BoW_matrix)
```

    [
        [0, 1, 1, 1, 0, 0, 1, 0, 1],
        [0, 1, 0, 1, 0, 2, 1, 0, 1],
        [1, 0, 0, 0, 1, 0, 1, 1, 0],
        [0, 1, 1, 1, 0, 0, 1, 0, 1]
    ]

直接看数字的话不是很直观，可以多增加一行一列，就能把单词和词频对应上了。如果你检查答案[^1]的话，你会发现和我们算出来的一模一样

|           | and | document | first | is  | one | second | the | third | this |
| --------- | --- | -------- | ----- | --- | --- | ------ | --- | ----- | ---- |
| document1 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |
| document2 | 0   | 1        | 0     | 1   | 0   | 2      | 1   | 0     | 1    |
| document3 | 1   | 0        | 0     | 0   | 1   | 0      | 1   | 1     | 0    |
| document4 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |

如何解读这个表格？

**每一行就是对应文档的词袋模型输出的向量表示**，以 document2 为例，它包含了下面这些单词
- `document` * 1
- `is` * 1
- `second` * 2
- `the` * 1
- `this` * 1

之前提到，分词之后的 document2 是 `['this', 'is', 'the', 'second', 'second', 'document']`，显然和这个向量表示是对应的

现在你就知道如何解读这个了矩阵了 :D

> 🧐 你也许注意到了，这个矩阵中有很多 `0`。BoW 的矩阵确实是这样的，是一个稀疏矩阵，这也是它的缺点之一


```python
from sklearn.metrics.pairwise import cosine_similarity
```

我们可以用向量内积计算 2 个向量之间的相似度

之前的 `tokenized_toy_corpus` 如下
```python
tokenized_toy_corpus = [
    ['this', 'is', 'the', 'first', 'document'],
    ['this', 'is', 'the', 'second', 'second', 'document'],
    ['and', 'the', 'third', 'one'],
    ['is', 'this', 'the', 'first', 'document']
]
```

现在，假设查询（query）是最后一个文档 —— `['is', 'this', 'the', 'first', 'document']`，前 3 个文档中哪一个跟它最像？

我们可以一眼看出答案是第一个，那么机器能否也发现这点？


```python
print(
    cosine_similarity([BoW_matrix[3]], [BoW_matrix[0]]),
    cosine_similarity([BoW_matrix[3]], [BoW_matrix[1]]),
    cosine_similarity([BoW_matrix[3]], [BoW_matrix[2]]),
)
```

    [[1.]] [[0.63245553]] [[0.2236068]]


🤔️ 机器也看出来了

## 更复杂的真实数据集

前面的例子太简单了，下面我会用一个更复杂的真实数据集 —— [CodeSearchNet](https://huggingface.co/datasets/code_search_net)。它包含了许多编程语言的函数，这里我挑 Python 来展示

当然你也可以选择你感兴趣的编程语言 :)

```python
from datasets import load_dataset
from gensim import corpora

def process_data(partition: str) -> list[str]:
    """
    Get data from the datasets library from huggingface.
    Only keep the `whole_func_string` column

    Arg
    ---
    `partition`: train/validation/test

    Return
    -----
        return a list of python functions
    """
    raw_datasets = load_dataset("code_search_net", "python")
    return raw_datasets[partition]["whole_func_string"]
```

取决于你的网络这可能得花上一点时间，压缩文件大小是 941MB


```python
# use the test dataset to speed up the process
corpus = process_data("test")
```

来看一下数据中的一个例子


```python
print(corpus[0])
```

    def get_vid_from_url(url):
            """Extracts video ID from URL.
            """
            return match1(url, r'youtu\.be/([^?/]+)') or \
              match1(url, r'youtube\.com/embed/([^/?]+)') or \
              match1(url, r'youtube\.com/v/([^/?]+)') or \
              match1(url, r'youtube\.com/watch/([^/?]+)') or \
              parse_query_param(url, 'v') or \
              parse_query_param(parse_query_param(url, 'u'), 'v')


在前面的英文分词中，我们删除了标点符号，用空格分词。代码的分词则比较不一样，编程语言有自己对应的语法（上下文无关文法），那么就可以用词法分析器拿到一个个 token。我这里用 Python 自带的 `ast` 和 `tokenize` 简单实现了一下

> 下面的函数如果看不懂也没关系，可以直接跳过。用词法分析器分词只是为了得到更精确的分词结果 :)


```python
import ast
from io import BytesIO
import tokenize

def get_token_stream(code: str) -> list[str]:
    """
    Tokenize the source code and return a token stream

    Note that the following token type will be removed:
    - COMMENT
    - NEWLINE
    - NL
    - INDENT
    - DEDENT
    - ENCODING
    - STRING
    """
    # see https://docs.python.org/3/library/token.html
    useless_token_type = {
        tokenize.COMMENT,
        tokenize.NEWLINE,
        tokenize.NL,  # non-terminating newline
        tokenize.INDENT,
        tokenize.DEDENT,
        tokenize.ENCODING,
        tokenize.STRING,
    }
    parse_tree = ast.parse(code)
    origin_tokens = tokenize.tokenize(BytesIO(code.encode("utf-8")).readline)
    token_as_strlist = [
        token.string
        for token in origin_tokens
        if token.type not in useless_token_type
    ]

    return token_as_strlist
```

注意两件事情：
- 我删除了**所有**的字符串，包括文档、f-string、普通字符串和注释等
- 我**没有**把变量名或者函数名根据 `camelCase` 或者 `snake_case` 这两种命名惯例进一步分词

首先，先用 `get_token_stream` 把数据中的 Python 代码都分词一下。注意数据中包含了 Python2 和 Python3，这里直接丢弃了 Python2 的代码


```python
from tqdm.auto import tqdm

py2_cnt, py3_cnt = 0, 0
codes = []
for code in tqdm(corpus):
    try:
        codes.append(get_token_stream(code))
        py3_cnt += 1
    except SyntaxError:
        py2_cnt += 1
print(f"Python2: {py2_cnt}, Python3: {py3_cnt}")
```

让我们验证一下 `get_token_stream` 是否分词分对了


```python
print(codes[0])
```

    [
        'def', 'get_vid_from_url', '(', 'url', ')',
        ':', 'return', 'match1', '(', 'url', ',', ')',
        'or', 'match1', '(', 'url', ',', ')', 'or',
        'match1', '(', 'url', ',', ')', 'or', 'match1',
        '(', 'url', ',', ')', 'or', 'parse_query_param',
        '(', 'url', ',', ')', 'or', 'parse_query_param',
        '(', 'parse_query_param', '(', 'url', ',', ')',
        ',', ')', ''
    ]


现在，可以利用 [Gensim](https://radimrehurek.com/gensim/) 提供的 API 来创建词汇表


```python
from gensim import corpora

dictionary = corpora.Dictionary(codes)

print(dictionary)
```

    Dictionary<77242 unique tokens: ['', '(', ')', ',', ':']...>


可以看到，词汇表太大了，让我们看看能否优化一下

一般来说，我们不会关心**只出现一次**的 token，因此，我们可以把他们删除掉


```python
once_ids = [
    token_id
    for token_id, doc_freq in dictionary.dfs.items()
    if doc_freq == 1
]

dictionary.filter_tokens(once_ids)
dictionary.compactify()

print(dictionary)
```

    Dictionary<31933 unique tokens: ['', '(', ')', ',', ':']...>


现在只剩下 `31933` tokens 了，这已经好多了，我猜测 token 这么多的原因是有很多变量/函数名都不一样 🧐

`Dictionary` 提供了 `most_common` 方法可以找到词频最多的 token，让我们看下能否发现一些有意思的事情

```python
dictionary.most_common(25)
```




    [
        ('.', 202834), ('(', 199868), (')', 199868), (',', 162853),
        ('=', 142808), (':', 110829), ('self', 71699), ('[', 55736),
        (']', 55736), ('if', 40272), ('return', 24021), ('def', 23557),
        ('', 21948), ('None', 19797), ('in', 19437), ('for', 13509),
        ('1', 13345), ('0', 13213), ('not', 11826), ('else', 10634),
        ('+', 10617), ('==', 9323), ('name', 8290), ('is', 7601), ('-', 7544)
    ]


🧐 发现了一个有意思的现象，`(` 和 `)` 的词频一样，`[` 和 `]` 也是的，因为代码的语法就是这么规定的

然后用 `doc2bow` API 就可以得到每个文档（代码）的词袋模型向量


```python
BoW_matrix_for_code = [dictionary.doc2bow(d) for d in codes]
print(BoW_matrix_for_code[0])
```

    [
        (0, 1), (1, 8), (2, 8), (3, 7),
        (4, 1), (5, 1), (6, 1), (7, 4),
        (8, 5), (9, 3), (10, 1), (11, 7)
    ]

`doc2bow` 的返回值是一个 tuple list，每个 tuple 的含义是 `(token 的 ID，这个 token 的词频)`，注意统计词频是在这个文档里面进行的

Gensim 用的这个格式是合理的，因为 BoW 算出来的矩阵是稀疏的，而且这里的词汇表大小有 `31933`，只展示非 0 的位置就清楚很多

让我们把每个 token id 替换为对应的字符串表示


```python
[
    (dictionary[token_id], cnt)
    for token_id, cnt in BoW_matrix_for_code[0]
]
```




    [
        ('', 1), ('(', 8), (')', 8), (',', 7),
        (':', 1), ('def', 1), ('get_vid_from_url', 1), ('match1', 4),
        ('or', 5), ('parse_query_param', 3), ('return', 1), ('url', 7)
    ]



下一件想要做的事情是：**能否让词袋模型根据我们提供的 Python 代码找到相似的 Python 代码**？


```python
from gensim.similarities import Similarity

indexer = Similarity(
    output_prefix=None,
    corpus=BoW_vectors,
    num_features=len(dictionary),
    num_best=3,                  # let's see Top-3 result
)
```

随便写一个 Python 函数来检查词袋模型是否返回了相似的代码


```python
query = """def foo(x):
    if x > 5:
        if x > 10:
            return x + 1
        else:
            return x - 1
    else:
        if x < 0:
            return x + 1
        else:
            return x - 1
"""
```


```python
indexer[dictionary.doc2bow(get_token_stream(query))]
```




    [
        (19669, 0.7191814184188843),
        (19805, 0.705620288848877),
        (5958, 0.6945071220397949)
    ]




```python
print(corpus[19669])
```

    def add_sms_spec_to_request(self, req, federation='', loes=None,
                                context=''):
        """
        Update a request with signed metadata statements.
        
        :param req: The request 
        :param federation: Federation Operator ID
        :param loes: List of :py:class:`fedoidc.operator.LessOrEqual` instances
        :param context:
        :return: The updated request
        """
        if federation:  # A specific federation or list of federations
            if isinstance(federation, list):
                req.update(self.gather_metadata_statements(federation,
                                                           context=context))
            else:
                req.update(self.gather_metadata_statements([federation],
                                                           context=context))
        else:  # All federations I belong to
            if loes:
                _fos = list([r.fo for r in loes])
                req.update(self.gather_metadata_statements(_fos,
                                                           context=context))
            else:
                req.update(self.gather_metadata_statements(context=context))

        return req


你可能会发现查询和结果似乎*在某种意义上*匹配。它们有一些相似的*语法*信息（嵌套的 `if-else` 结构）

然而，**在大多数情况下，词袋模型给出的结果很差**。这是合理的，因为词袋模型太简单了，无法找到代码之间的关系

## 总结
现在，我们来总结一下 BoW 的一些缺点。你可能已经自己弄清楚了其中一些：
1. 单词之间的顺序信息没有得到保留。*`The cat chased the dog` 和 `The dog chased the cat` 意思完全不一样*
2. 没有语义信息。 *词袋模型将每个单词视为一个独立的实体*
3. 词袋模型输出的向量是高维稀疏向量。 *计算量很大，向量长度取决于你的词汇表大小*
4. 每个词都有相同的重要性。 *但有些词可能提供更多信息*
5. 无法处理不在词汇表里面的单词。 *如果文档包含很多不在词汇表中的单词要怎么办*？
6. ...

词袋模型有很多缺点，你可能只会在教程里面看到它。鉴于这些限制，后面人们又开发了 Word2Vec、GloVe 和基于 Transformer 的架构（例如 BERT、GPT）等更先进的模型来克服这些缺点

## 参考

[^1]: [CountVectorizer](https://scikit-learn.org/stable/modules/feature_extraction.html#common-vectorizer-usage)


