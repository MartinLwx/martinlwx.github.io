# Hw09 of CS61A of UCB(2021-Fall)


### Q2: Roman Numerals

>   Write a regular expression that finds any string of letters that resemble a Roman numeral and aren't part of another word. A Roman numeral is made up of the letters I, V, X, L, C, D, M and is at least one letter long.
>
>   >   For the purposes of this problem, don't worry about whether or not a Roman numeral is valid. For example, "VIIIII" is not a Roman numeral, but it is fine if your regex matches it.

The details: 

1.   The letters contain I, V, X, L, C, D, M. We can use `[]` to represent the logical OR
2.   The roman NUmerals is at least one letter long. We can use `+` to match 1 or more repetitions. 
3.   The roman NUmerals can not be a part of another word. After checking the documentation, I found `\b` is quite useful, which means matching the empty string, but only at the beginning or end of a word. That's exactly what we want.

```python
def roman_numerals(text):
    """
    Finds any string of letters that could be a Roman numeral
    (made up of the letters I, V, X, L, C, D, M).
    """
    return re.findall(r'\b[IVXLCDM]+\b', text)
```

### Q3: CS Classes

>   On reddit.com, there is an /r/berkeley subreddit for discussions about everything UC Berkeley. However, there is such a large amount of CS-related posts that those posts are auto-tagged so that readers can choose to ignore them or read only them.
>
>   
>
>   Write a regular expression that finds strings that resemble a CS class- starting with "CS", followed by a number, and then optionally followed by "A", "B", or "C". Your search should be case insensitive, so both "CS61A" and "cs61a" would match.

The details:

1.   Either the `CS` or the `ABC` is case insensitive. The former can be solved by `|`, and the latter can be solved by `[]`
2.   The whitespace may exist between `CS` and the digits

```python
def cs_classes(post):
    """
    Returns strings that look like a Berkeley CS class,
    starting with "CS", followed by a number, optionally ending with A, B, or C
    and potentially with a space between "CS" and the number.
    Case insensitive.
    """
    return bool(re.search(r'cs|CS ?\d+[ABCabc]?', post))
```

### Q4: Time for Times

>   You're given a body of text and told that within it are some times. Write a regular expression which, for a few examples, would match the following:
>
>   ```
>   ['05:24', '7:23', '23:59', '12:22', '00:00']
>   ```
>
>   but would not match these invalid "times"
>
>   ```
>   ['05:64', '70:23']
>   ```
>
>   You may find [non-capturing groups](https://www.regular-expressions.info/brackets.html#noncap) helpful to use for this question.

The legal range of the time: `00:00 ~ 23:59`



The details:

1.   The first digit and the second digit: The leading `0` may exist. When the first digit is `2`, the second digit should be in the range of `0 ~ 3`. (`2?:??` ~ `23:59`)
2.   The third digit and the 4th digit: `00 ~ 59`. `[0-5][0-9]`

It is a little complicated for a new beginner like me :(



The hint shows the `(?:)` may be helpful, so I have a look at this and suddenly know how to make the regex work :+1:. It means matching but not capturing. 

```python
def match_time(text):
    return re.findall(r'(?:[01]?\d|2[0-3]):[0-5][0-9](?:AM)?', text)
```

### Q5: Most Common Area Code

>   Write a function which takes in a body of text and finds the most common area code. Area codes must be part of a valid phone number.
>
>   To solve this problem, we will first write a regular expression which finds valid phone numbers and captures the area code. See the docstring of `area_codes` for specifics on what qualifies as a valid phone number.

We know the area code may appear in the front of a phone number. The task is to find the most common area code. First, we need to implement a function called `area_codes` to parse a string to get all potential area codes and return. After that, we use `list.count` to find the most common one.



The details.

1.   A valid phone number should be 10-digit.
2.   The length of the area code is 3. Sometimes, it may have parentheses around. 
3.   The hyphens and spaces may appear after the third and sixth digits



How to match a potential `()` ? By `(?:)?` :hugs:

```python
def area_codes(text):
    """
    Finds all phone numbers in text and captures the area code. Phone numbers
    have 10 digits total and may have parentheses around the area code, and
    hyphens or spaces after the third and sixth digits.
    """
    return re.findall(r'(?:\()?(\d{3})(?:\)?)(?: |-)?\d{3}(?: |-)?\d{4}\b', text)


def most_common_code(text):
    """
    Takes in an input string which contains at least one phone number (and
    may contain more) and returns the most common area code among all phone
    numbers in the input. If there are multiple area codes with the same
    frequency, return the first one that appears in the input text.
    """
    area_codes_list = area_codes(text)
    cnts = [area_codes_list.count(e) for e in area_codes_list]          # count every area_code
    max_cnt_idx = cnts.index(max(cnts))                                 # get the index of the max value
    return area_codes_list[max_cnt_idx]
```


