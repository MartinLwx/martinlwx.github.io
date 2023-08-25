# BPE Tokenization Demystified: Implementation and Examples


## BPE

In NLP, one crux of problems is - how to tokenize the text. There are three methods available:
- Char-level
- Word-level
- Subword-level

Let's talk about the Char-level tokenizer. That is, we tokenize the text into a char stream. *For instance, `highest -> h, i, g, h, e, s, t`*. One advantage of the Char-level tokenizer is that the size of Vocab won't be that large. The size of Vocab is equal to the size of the alphabet. So you probably won't meet the infamous Out-of-vocabulary(OOV) problem. However, the downside is that **the char itself does not convey too much information**, and **we will get too many tokens after tokenizing**. *Try to imagine that a simple word highest will give us 7 tokens*😨

The Word-level tokenizer is slightly better. In the Word-level tokenizer, we usually divide text into individual words using whitespace or punctuation marks. so the number of tokens after tokenizing won't be too high. *For instance, `Today is a good day` -> `Today, is, a, good, day`*. However, the Word-level tokenizer is not perfect. In English, the words `high, higher, highest` are semantically similar, with the latter two being comparative forms, but Word-level tokenization would treat them as three separate words.

🤔️ Is there a better way such that we most likely avoid the OOV problem, have a reasonable number of tokens after tokenizing, and also consider compound words, tense variations, etc.? This is where subword level tokenization comes into play. Using the same example as before, a subword tokenizer will most likely tokenize it as `high, er`, `high, est`. Therefore, **the subword level tokenization involves representing a word using multiple subwords**. The method I'm introducing today, BPE (**B**yte-**P**air **E**ncoding), belongs to the subword level category.

There is one *subtle* thing to think about - **What does the byte in the byte pair mean**? We know that every char is 1 byte if we are using the ASCII encoding. However, what if we are using utf-8 encoding? A char may be 1 ~ 4 bytes. 🤔️ So, **if BPE is applied to utf-8 text, what does a byte pair refer to**? Thus, I feel there's a bit of ambiguity here. Therefore, **a better way to understand it might be that a byte pair is essentially a char pair, where char here refers to a utf-8 character**.


### BPE training phase

Now let's figure out how the BPE tokenizer got trained. Let's assume that we have some documents $D$
1. For each $d$, we will transform the documents into word list *in some way*. *For instance, you may choose to split the document by whitespace to get words*.
2. Count the word freq for each word $w$ in $D$, and we can also get the alphabet of $D$ as the initial vocab(plus the `</w>`)
3. For each word, transform the word into a utf-8 char list. We call it a split. *For example, `highest -> h, i, g, h, e, s, t`*
4. Append `</w>` to each utf-8 list. *e.g. `highest -> h, i, g, h, e, s, t, </w>`*
5. Repeat the following steps until any one of the two conditions is met: 1) Vocab reaches the upper limit. 2) Reach the maximum number of iterations
    1. Find **the most frequent pair**, add it to a merge table, and add the merged result to the vocab
    2. Update all splits of all words. *For example, the most frequent pair may be `(h, i)` in our previous example, then we will do `highest -> hi, g, h, e, s, t, </w>`*

You may have 3 puzzles:
1. Why word frequency? Because we want to find the most frequent pair easily
2. Why append `</w>`? Because we want to reconstruct the input later, we use `</w>` to mark that it is the end of a word
3. What if we have multiple pairs with the same frequency? How to handle this may vary in different implementations, but *shouldn't* have much impact in my opinion.

> 💡 You can observe that when the BPE algorithm merges the most frequently occurring pair, it doesn't cross over words.

### How to use a trained BPE?

After we trained a BPE tokenizer, we will obtain a merge table and a vocab. Assuming that we now need to tokenize the text `s`

1. Use the same method as during training, start by splitting `s` into individual words, with each word further divided into utf-8 char.
2. Iterate through the merge table and check if each merge rule can be applied to update the split of each word.

> 💡 An important detail here is that the merge rules we extracted are sorted in descending order of frequency. Thus, by sequentially traversing the merge table, we are *implicitly* incorporating the notion of prioritizing the merging of the most frequently occurring pairs.



### BPE example

Let's use a motivating example to see how BPE got trained.


*Let's say the corpus is*
```python
corpus = ["highest", "higher", "lower", "lowest", "cooler", "coolest"]
```

*We do not need to compute the word frequency here in that each word appears exactly once. First, let's transformer each word into utf-8 chars and append `</w>`*
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
*We can find that the most frequently occurring pair is `(e, s)`. Thus, we use this merge rule to update the splits. Note that we can also choose `(e, r)` because they have the same frequency*
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
*This time, the most frequently occurring pair is `(es, t)`. We follow the same procedure and update the splits*
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
*The most frequently occurring is `(est, </w>)`*
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
*The most frequently occurring is `(e, r)`*
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
*The most frequently occurring is `(er, </w>)`*
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

*I believe the example above is quite clear enough. We can find that the BPE tokenizer finds two meaningful suffixes `er` and `est`*

### BPE tokenizer in Huggingface
The API provided by the Huggingface is quite simple. *You may notice that it uses `Char` in the class name, which confirms what I mentioned earlier*
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

### Implemene a BPE tokenizer
The best way to understand how an algorithm works is to always try to implement one. Here I implement a `BPE` class which follows the algorithm described earlier. If you initialize with `debug=False`, you can observe how the entire BPE update process works

*First, let's check the constructor and the `train` method*
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
*The implementations of the key functions are as follows*
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

*Now we can implement a `tokenize` method*

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
*Try to tokenize the `corpus` mentioned earlier using our hand crafted `BPE` tokenizer*
```python
bpe = BPE(corpus, vocab_size=17, debug=False)
bpe.train()
bpe.tokenize(" ". join(corpus))
```

The output is:
```python
['h', 'i', 'g', 'h', 'est</w>',
 'h', 'i', 'g', 'h', 'er</w>',
 'l', 'o', 'w', 'er</w>',
 'l', 'o', 'w', 'est</w>',
 'c', 'o', 'o', 'l', 'er</w>',
 'c', 'o', 'o', 'l', 'est</w>']
```

🤔️ This indicates that our implementation is correct, and thanks to `</w>`, we can see the word boundaries between words and even reconstruct the original input.


### Wrap up

The BPE tokenize is simple and practical, but **when you delve into its implementation, you will encounter several details. However, it's precisely by engaging with these intricacies that your understanding of BPE becomes more profound**.

Let's also talk about some limitations of BPE. For instance, you will notice that we are using whitespace to split text, which **works for whitespaced language**. However, for languages like Chinese, spaces don't define word boundaries, which makes things more complex and calls for a better tokenizing method.

