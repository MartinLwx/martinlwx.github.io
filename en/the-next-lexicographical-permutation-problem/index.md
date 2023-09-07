# The next lexicographical permutation problem


## Intro

Occasionally, you may want to **get the next/prev lexicographical permutation of a sequence**. How would you do that? If you are a C++ programmer, you are probably familiar with the `next_permutation`[^1] and `prev_permutation`[^2] APIs. However, Python does not provide the counterparts. So the topic today is how to do this in Python. Since the solutions of prev lexicographical permutation and the next lexicographical permutation are very similar, let us focus on the next lexicographical permutation problem.

## Algorithm

What may surprise you is that someone came up with a solution to this problem in the 14th century[^3]. Assume the sequence is `a`. The following steps show how to get the next lexicographical permutation
1. Find the **largest index `k`** such that `a[k] < a[k + 1]`. If no such index exists, the permutation is the last permutation
2. Find the **largest index `l`** greater than `k` such that `a[k] < a[l]`
3. Swap the value of `a[k]` with that of `a[l]`
4. Reverse the sequence from `a[k + 1]` up to and including the final element `a[n]`


That is how you get the next lexicographical permutation. But **why this is correct**? Now let's dive into the details to figure out this.

## Algorithm explained

Let's forget this algorithm and try to imagine how would you solve the next lexicographical permutation problem. If you are familiar with the backtracing algorithm, you might try to enumerate all possible permutations and try to find the specific one. However, the time complexity is awful which is $O(N!)$. That brings us to the following question

> **Q1** - What are the characteristics of generating the next lexicographical permutation problem during backtracking?

*Let's use a simple example `[1, 2, 3, 4]` and enumerate its permutations in lexicographical order.*

```Python
[[1, 2, 3, 4],
 [1, 2, 4, 3],
 [1, 3, 2, 4],
 [1, 3, 4, 2],
 [1, 4, 2, 3],
 [1, 4, 3, 2],
 [2, 1, 3, 4],
 ...  # omit
 [4, 3, 2, 1]]
```
*What makes `[2, 1, 3, 4]` the next lexicographical permutation of `[1, 4, 3, 2]`? Could you spot any patterns? More specifically, why did the value of the first position change from `1` to `2`? What's the characteristics of `[1, 4, 3, 2]`?* **The answer is: `4, 3, 2` is the longest non-increasing suffix(LNIS)**. Does this rule apply to any situation? *Let's see another example, `[1, 2, 4, 3] -> [1, 3, 2, 4]`. We can see that the LNIS of `[1, 2, 4, 3]` is `[4, 3]`, and the second position changes from `2` to `3`*. Let's line these up to make it more clear

```Python
[1, 4, 3, 2]
    ^
    # LNIS
[2, 1, 3, 4]
 ^
 # change this

[1, 2, 4, 3]
       ^
       # LNIS
[1, 3, 2, 4]
    ^
    # change this
```

So we can conclude - **Find the start position `j` of LNIS in the sequence and try to increase the value in position `j - 1`**. This explains the step 1 of the aforementioned algorithm. If you are not convinced by the above observation, then you can try to understand this from the backtracing perspective. When will we increase the value of position `j - 1`? When we have **already** enumerated all the permutations of `a[j:]` and returns from the recursive function call. The `a[j:]` subsequence would be in non-increasing order since we try to set the value of each position in lexicographical order.

```Python
indices: 0, 1, 2, ..., j - 1, j, ..., n - 1
                   # a[j - 1] < a[j]
```

> **A1** - To generate the next lexicographical permutation, we will always try to increase the previous position of LNIS

For the convenience of subsequent discussion, we call the position on the left of the LNIS found earlier as `pivot` (that is, `j - 1` in front). So now comes the second question

> **Q2** - How to change the value of `a[pivot]`?

We are concerned about the next lexicographical permutation, so we want to make `a[pivot]` **slightly bigger**. But **how can we find the next greater value of `a[pivot]`?** The answer is: this element is located in `a[pivot + 1:]`, that is, we need to find a position whose value is bigger than `a[pivot]`, and this value should be as small as possible.

Do not forget that `a[pivot+1:]` is the LNIS, which means that we can **search `a[pivot + 1:] from right to left**. This explains the step 2 of the aforementioned algorithm.


> **A2** - **Search `a[pivot + 1:]` from right to left** whose value is bigger than `a[pivot]` and this value should be as small as possible. Let's denote the index of this value as `r`

Now there are only two remaining steps to explain, and they are related

> **Q3** - Why should we swap `a[pivot]` and `a[r]`, then reverse the `a[pivot + 1:]` suffix?

Theoretically speaking, after we modify the value of `a[pivot]`, the part of `a[pivot + 1:]` also needs to be modified, so the question is how to modify it. From the backtracing perspective, `a[pivot + 1]` should start from the smallest lexicographical order, that is, **`a[pivot + 1:]` should be in non-decreasing order. Interestingly**, the swap operation **does not** change the order of `a[pivot + 1:]`, which means the `a[pivot + 1:]` is still in non-increasing order. To make it in non-decreasing order, we just need to reverse this part. This explains the step 3 & 4 of this algorithm

> **A3** - The swap operation does not change the order of `a[pivot + 1:]`. We reverse `a[pivot + 1:]` to make it in non-decreasing order


## Code

The Leetcode offers a [practice](https://leetcode.com/problems/next-permutation/description/), you can examine your understanding there. I would suggest that try to implement this on your own at first and then check my code if needed :)


```Python
class Solution:
    def nextPermutation(self, nums: List[int]) -> None:
        # Step 1. Find the rightmost i s.t. a[i] > a[i - 1]
        #         and set pivot to i - 1
        pivot = -1
        for i in reversed(range(len(nums))):
            if i - 1 >= 0 and nums[i] > nums[i - 1]:
                pivot = i - 1
                break
        if pivot == -1:
            nums.reverse() 
            return

        # Step 2. Find the rightmost value s.t. a[i] > a[pivot]
        for i in reversed(range(len(nums))):
            if nums[i] > nums[pivot]:
                nums[i], nums[pivot] = nums[pivot], nums[i]
                break

        # Step 3. Reverse the (pivot, len(nums)) part
        left, right = pivot + 1, len(nums) - 1
        while left < right:
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
            right -= 1

        return None
```

## Wrap up

The key to understanding the algorithm of the next lexicographical permutation problem is the backtracing algorithm(in my humble opinion). I hope you find this post helpful :)


## Refs

[^1]: [std::next_permutation](https://en.cppreference.com/w/cpp/algorithm/next_permutation)
[^2]: [std::prev_permutation](https://en.cppreference.com/w/cpp/algorithm/prev_permutation)
[^3]: [Permutation-Wiki](https://en.wikipedia.org/wiki/Permutation)

