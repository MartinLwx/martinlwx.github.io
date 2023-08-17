# TF-IDF model


## What is the TF-IDF model

In previous [post]({{<ref "/content/posts/an-introduction-of-bag-of-word-model.md">}}), we talked about the bag-of-word model, which has many limitations. Today we take a step further to see if we can try to fix **one of the limitations - Each word has the same importance**.

> 💡 The crux of the problem - **How to define the word importance**?

One idea is: The more frequently a word appears **within a single document**, the more important it is **for that document**. *For instance, in an article discussing dogs, the word "dog" is likely to appear frequently, reflecting the document's main topic*.

But what if a word appears very frequently **in all documents**? *For example, the word "of" may appear quite often in every document, can we say "of" is important?* Clearly, that's not the case. So we have a clue here: If a word has a high frequency in **every document**, probably it's not significant and does not convey too much information.

Therefore, a reasonable solution should consider a word's frequency within **a single document** but also take into account its frequency crossing **multiple documents**. TF-IDF balances these two aspects.

In summary, the **intuition** behind TF-IDF is - **Similar documents *may* use similar words, while the importance of different words should vary**.

## TF-IDF in detail

TF-IDF = Term frequency(TF) + Inverse document frequency(IDF). Let's break it into two parts
### TF

The TF can be seen as a function of document $d$ and word $w$, and the equation is:

$$\text{TF}(w, d)=\frac{\text{frequency of}\ w\ \text{in}\ d}{\text{word counts of } d}$$

That is, we just need to calculate the frequency of word $w$ in the document $d$, and then divide it by the total number of words in $d$.

> 🐛 In Scikit-Learn, the computation of TF is a bit different. It doesn't involve dividing by the total number of words in the document. The purpose of dividing is to normalize the $\text{TF}(w, d)$ values within the document $d$, making them add up to one. In Scikit-Learn, this normalization process is performed after the TF-IDF calculation. We will demonstrate this with an example later.

### IDF

The goal of IDF is to reduce the importance of some common words that appear in each document. Therefore, the IDF is a function involving the word $w$ and the $corpus$.

$$
\text{IDF}(w, corpus)=log\ \frac{\text{document count of }corpus}{1+\text{count of document which contains }w}
$$

We add one in the denominator to avoid division by 0.

> 🤔️ The $corpus$ is gennerally fixed. So it can be treated as a constant. In that case, IDF can be considered as something that's only related to the word $w$

> 💡 Note the $log$ here. Are we using $log_2$, $log_{10}$ or $ln$? Different frameworks might have variations. *Scikit-Learn use $ln$*

> 🐛 In Scikit-Learn, the calculation of IDF differs from the equations mentioned above. By default, Scikit-Learn uses the following formula[^1]:

$$
\text{IDF}(w, corpus)=log\ \frac{1 + \text{document count of }corpus}{1+\text{count of document which contains }w} + 1
$$

🤔️ In my opinion, **the modification made by Scikit-Learn ensures that $\text{IDF}(w)$ cannot be less than 1**. In the origin equation, if a word $w$ appears in each document within the corpus, $\text{IDF}(w)$ could be a negative value. Therefore, Scikit-Learn's modification seems more practical. It provides a more intuitive comparison of the IDF values for different words.

### TF-IDF

$$
\text{TF-IDF}(w, d, corpus)=\text{TF}(w, d) * \text{IDF}(d, corpus)
$$

## The TF-IDF in Scikit-Learn

It's trivial to implement the TF-IDF algorithm. However, probably you will just use the well-established APIs provided by Scikit-Learn. Here, we will delve into how to calculate TF-IDF in Scikit-Learn. Let's proceed by continuing to use the official example from Scikit-Learn:

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

Let's retrieve the TF-IDF matrix using the APIs

```python
from sklearn.feature_extraction.text import TfidfVectorizer
# set norm=None for comparison
vectorizer = TfidfVectorizer(norm=None)
X = vectorizer.fit_transform(toy_corpus)
```

We can access the TF-IDF matrix using `X.toarray()` (Note that I set `norm=None` in the code snippet)

|           | and     | document | first   | is      | one     | second  | the | third   | this    |
| --------- | ------- | -------- | ------- | ------- | ------- | ------- | --- | ------- | ------- |
| document1 | 0.0     | 1.22314  | 1.51082 | 1.22314 | 0.0     | 0.0     | 1.0 | 0.0     | 1.22314 |
| document2 | 0.0     | 1.22314  | 0.0     | 1.22314 | 0.0     | 3.83258 | 1.0 | 0.0     | 1.22314 |
| document3 | 1.91629 | 0.0      | 0.0     | 0.0     | 1.91629 | 0.0     | 1.0 | 1.91629 | 0.0     |
| document4 | 0.0     | 1.22314  | 1.51082 | 1.22314 | 0.0     | 0.0     | 1.0 | 0.0     | 1.22314 |

Let me also put the bag-of-word matrix here:

|           | and | document | first | is  | one | second | the | third | this |
| --------- | --- | -------- | ----- | --- | --- | ------ | --- | ----- | ---- |
| document1 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |
| document2 | 0   | 1        | 0     | 1   | 0   | 2      | 1   | 0     | 1    |
| document3 | 1   | 0        | 0     | 0   | 1   | 0      | 1   | 1     | 0    |
| document4 | 0   | 1        | 1     | 1   | 0   | 0      | 1   | 0     | 1    |

🤔️ *Comparing these two matrices, we can find that the word importance of `document` and `first` inside the document1 has changed. The `TF-IDF` value for `document` is `1.22314`, while the TF-IDF value for `first` is `1.51082`, due to the unequal presence of these words in the corpus. However, the bag-of-word model fails to recognize this and considers both of them as having an importance of `1`*

We can retrieve the IDF value of each word by accessing the `idf_` attribute
```python
print(vectorizer.idf_)
# [1.91629073 1.22314355 1.51082562 1.22314355 1.91629073 
#  1.91629073 1.         1.91629073 1.22314355]
```

🤔️ If we multiply this IDF vector by the matrix output of the bag-of-word model(note that the IDF vector will be broadcasted), you would obtain the TF-IDF matrix calculated by Scikit-Learn. This confirms what we mentioned earlier:
- Scikit-Learn directly uses the output of the bag-of-word model as TF.
- Scikit-Learn's IDF calculation differs from the standard approach.


## Implement TF-IDF manually

We assume that each document within the corpus is tokenized, and we use the TF-IDF definition of Scikit-Learn

> 🐛 The code below is not optimized, just for demonstration :)

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

## Wrap up
In this post, we investigate how TF-IDF has brought improvements to the bag-of-word model: **by additionally considering a word's frequency across all documents to adjust its importance**. Through the analysis of official example in Scikit-Learn, we have also validated that, the importance of words are now different. This stands as a noteworthy enhancement to the bag-of-word model.

From an implementation perspective, the calculation of TF-IDF only requires traversing the corpus once. During the traversal, a few additional variables are maintained: the count of how many documents each word appears in, the total number of processed documents, etc. You may take a look at the source code of the `Dictionary` class in [Gensim](https://radimrehurek.com/gensim/index.html) for more details.

## Refs

[^1]: [TF-IDF term weighting](https://scikit-learn.org/stable/modules/feature_extraction.html#tfidf-term-weighting)

