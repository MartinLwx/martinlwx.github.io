# An Introduction of Bag of Word Model


## What is the bag-of-word model?

In NLP, we need to represent each document as a vector because machine learning can only accept input as numbers. That is, we want to find a *magic* function that:
$$
f(\text{document}) = vector
$$

Today's topic is **bag-of-word(BoW) model**, which can transform a document into a vector representation.

> 💡 Although the BoW model is outdated in 2023, I still encourage you to learn from the history and think about some **essential problems**:
- What is the motivation?
- What are the pros and cons?
- How can we make it better?

> 💡 Note that I may use word and token interchangeably

### Motivation & intuition
Before we dive into the details, I want to give you an **intuition** why BoW may work - **Similar documents *may* use similar words**

You may object to this intuition and show some good counterexamples, and I agree with your point. That's why we need more powerful models rather than BoW :)

### BoW model in detail

In BoW, you need to do **two things**:
1. Create a vocabulary. Each token in the vocab is assigned a unique id (usually, it will start from `0`). **The length of the BoW vector will be equal to the size of the vocab**
2. For each document in the corpus, identify words that are not currently present in the existing vocabulary, and subsequently incorporate these words into the vocabulary list.


After constructing a BoW model, we can use it to transform any document into a vector representation. The procedure is simple, we just count the occurrences of each word in the document. Note that we **only** consider vocab words and ignore the out-of-vocabulary(OOV) words.

Let's use a toy example to illustrate this idea[^1]

```python
toy_corpus = [
    'This is the first document.',
    'This is the second second document.',
    'And the third one.',
    'Is this the first document?',
]
```

Remove punctuation, then tokenize with spaces, and also convert all the words to lowercase. After preprocessing, we can obtain:


```python
tokenized_toy_corpus = [
    ['this', 'is', 'the', 'first', 'document'],
    ['this', 'is', 'the', 'second', 'second', 'document'],
    ['and', 'the', 'third', 'one'],
    ['is', 'this', 'the', 'first', 'document']
]
```

To simplify matters, let's encompass all words within the corpus and incorporate them into our vocabulary.



```python
flatten_list_as_set = set(sum(tokenized_toy_corpus, start=[]))
print(f"the toy vocab size: {len(flatten_list_as_set)}")
```

    the toy vocab size: 9


> 💡 A nice trick to flatten this list :D

Now, let's assign a unique token id to each word in the vocab


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


The vocab size is `9`, then we know we can represent each document as a vector with a length `9` by counting the words

Let's manually implement this to see if we understand the ideas


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


The numbers may not be so intuitive, let's add more information to make this better. If you check the answer[^1], that's exactly what we calculated

|           | and | document | first | is  | one | second | the | third | this |
| --------- | --- | -------- | ----- | --- | --- | ------ | --- | ----- | ---- |
| document1 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |
| document2 | 0   | 1        | 0     | 1   | 0   | 2      | 1   | 0     | 1    |
| document3 | 1   | 0        | 0     | 0   | 1   | 0      | 1   | 1     | 0    |
| document4 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |

Here comes the question: How to read this?

**Each row is a BoW vector of the corresponding document**. Take the 2nd row as an example, it means the document2 has:
- `document` * 1
- `is` * 1
- `second` * 2
- `the` * 1
- `this` * 1

Recall that the tokenized document2 is `['this', 'is', 'the', 'second', 'second', 'document']`, which is aligned with the vector representation

Now you know how to interpret the BoW matrix. :D

> 🧐 You might have observed that there are so many `0` in this matrix. Indeed, the BoW matrix tends to be sparse. That's one of the limitations of BoW


```python
from sklearn.metrics.pairwise import cosine_similarity
```

We can use the inner product to measure the similarity between two vectors.

Recall our `tokenized_toy_corpus`
```python
tokenized_toy_corpus = [
    ['this', 'is', 'the', 'first', 'document'],
    ['this', 'is', 'the', 'second', 'second', 'document'],
    ['and', 'the', 'third', 'one'],
    ['is', 'this', 'the', 'first', 'document']
]
```

Now, let's say the query is the last document - `['is', 'this', 'the', 'first', 'document']`, which document has the highest similarity except the query?

We as humans can find this at a glance. The first document should be the answer. Let's check if the machine can figure out this:


```python
print(
    cosine_similarity([BoW_matrix[3]], [BoW_matrix[0]]),
    cosine_similarity([BoW_matrix[3]], [BoW_matrix[1]]),
    cosine_similarity([BoW_matrix[3]], [BoW_matrix[2]]),
)
```

    [[1.]] [[0.63245553]] [[0.2236068]]


The machine agrees with us. 🤔️

## Beyond the toy example

The toy example is not quite exciting in my opinion. So I use a real-world dataset - [CodeSearchNet](https://huggingface.co/datasets/code_search_net) to play the BoW model.


The CodeSearchNet contains various functions from many programming languages, I just pick the Python code to analyze.

You are free to investigate another programming language :)

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


This may take a while depending on your network condition(941MB)


```python
# use the test dataset to speed up the process
corpus = process_data("test")
```

Let's see a simple example


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


Different from the English text, the programming language has well-defined grammar(context-free grammar). So we can tokenize the source code by a lexer. I use the built-in `ast` module and `tokenize` module to achieve this

> Feel free to skip this function if you can't understand how a lexer works. The reason behind using a lexer is to make the tokenization process more accurate :)


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

Two things to notice are:
- We remove **all** strings, including docstring, f-string, comment, etc.
- We **do not** tokenize the variable name or function name using `camelCase` or `snake_case` convention

First, let's use `get_token_stream` to tokenize each function within this dataset. Note that the dataset contains Python2 code, which can't be processed by the auxiliary function I have crafted. Consequently, I choose to remove the Python2 code.


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

Let's make sure the `get_token_stream` function works correctly


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


Now, we can leverage the API provided by the [Gensim](https://radimrehurek.com/gensim/) to create a vocabulary


```python
from gensim import corpora

dictionary = corpora.Dictionary(codes)

print(dictionary)
```

    Dictionary<77242 unique tokens: ['', '(', ')', ',', ':']...>


That's a **huge** dictionary. Let's see if we can optimize this. 🤔️

Usually, we are not interested in tokens that **only appear once** in our corpus. So we can remove them


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


Reduced to a mere `31933` tokens, a notably improved outcome. The abundance of distinct function/variable names might be contributing to this phenomenon 🧐

We can use the `most_common` method provided by the `Dictionary` class to see if we could find some interesting things


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



🧐 Found an interesting phenomenon, `(` and `)` have the same word frequency, and `[` and `]` are also the same, which **makes sense** since that's what the grammar requires

Now let's use the `doc2bow` API to generate BoW vector for each document(code)


```python
BoW_matrix_for_code = [dictionary.doc2bow(d) for d in codes]
print(BoW_matrix_for_code[0])
```

    [
        (0, 1), (1, 8), (2, 8), (3, 7),
        (4, 1), (5, 1), (6, 1), (7, 4),
        (8, 5), (9, 3), (10, 1), (11, 7)
    ]


The return value of `doc2bow` is a list of tuples, each tuple is `(token id, count of occurrences)` within its document.

The format is reasonable because now we have a vocab with size `31933`. We don't want to see a vector with size `31933` and there are so many zeros!

Let's replace the token id with the corresponding string


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



The next thing I want to do is: **Can we find similar Python code using BoW**?


```python
from gensim.similarities import Similarity

indexer = Similarity(
    output_prefix=None,
    corpus=BoW_vectors,
    num_features=len(dictionary),
    num_best=3,                  # let's see Top-3 result
)
```

Write a Python code as you wish and check if the BoW model returns the similar code


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


You may find that the query and the result seem to match *in some sense*. They have some similar *syntactic* information (the nested `if-else` structures).

However, in most circumstances, the BoW model gives a poor result. That's reasonable because the BoW is too naive to find the relationship between codes

## Wrap up

Now, let's summarize some limitations of BoW. You may have figured out some of them by yourself:
1. Loss of word order information. *`The cat chased the dog` is different from `The dog chased the cat`*
2. No semantic information. *BoW treats each word as an independent entity*
3. The BoW vector is a high-dimensional sparse vector. *It is computationally expensive and the size depends on your vocab size*
4. Each word has the same importance. *Some words may be more informative*
5. Does not handle out-of-vocabulary problems. *What if a document contains many OOV tokens?*
6. ...

The BoW model has so many drawbacks that you probably only would see it in tutorials for educational purposes. In light of these limitations, more advanced models like Word2Vec, GloVe, and transformer-based architectures (e.g., BERT, GPT) have been developed to overcome these drawbacks and provide better representations of text.

## Refs

[^1]: [CountVectorizer](https://scikit-learn.org/stable/modules/feature_extraction.html#common-vectorizer-usage)


