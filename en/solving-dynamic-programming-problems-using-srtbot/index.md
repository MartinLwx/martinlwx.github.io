# Solving DP problems by SRTBOT Framework


**Changelog**:
- [Update dependency graphs]({{< ref "#relate-subproblems"  >}}) @2023.04.13

## Intro

When solving algorithm problems, what often gives me a headache are dynamic programming problems(DP problems). They are the type of problems that I can't figure out on my own after thinking for a long time, but after seeing the answer, it suddenly becomes clear and reasonable. However, the next time I encounter a similar problem, I may forget how to solve it. I have also read many people's solutions and tried to digest and apply their ideas, but **I have been unable to find a particularly good framework that works for all dynamic programming problems**. It seems that everyone has their way of solving dynamic programming problems, and when I try to apply their methods to new problems, I always encounter difficulties. **Things start to change after I learned [MIT6.006](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/)**. The teacher presented 6 steps to solve dynamic programming problems, which is called the SRTBOT framework. I found it to be **so useful and practical** that I decided to write this blog post to share it with everyone 🙌


> 👉 Overall, this blog post is more suitable for those who already have a basic understanding of dynamic programming but have not yet found a systematic approach

## What's the DP problem?

The two main features of dynamic programming problems are "overlapping subproblems" and "optimal substructure" [^1]:
- **Overlapping subproblems**: Solving a dynamic programming problem often involves breaking it down into several overlapping subproblems.
- **Optimal substructure**: The solution to a large problem can be calculated by combining the optimal solutions of smaller subproblems.

It is important to note the terms "overlapping" and "subproblems" in the above definitions. Emphasis is placed on overlapping because the power of dynamic programming lies in its ability to remember the answers(by memorization) to solve subproblems. Therefore, the more overlapping subproblems there are, the more pronounced the advantage of dynamic programming.


> 🤔️ If the subproblems are not overlapping, then we may use another algorithm - Divide and conquer.


From the perspective of programming problems, the following two types of problems usually require dynamic programming algorithms:
- Optimization problems: **finding the optimal solution + overlapping subproblems**
- Combination problems: **finding the number of possible solutions**


> 📒 Alternatively, based on my personal experience, dynamic programming problems are generally recursive problems that can be solved by brute force, but they have a large number of overlapping subproblems, so they can be optimized using dynamic programming.

## SRTBOT Framework

SRTBOT is an acronym for 6 steps, which are [^2]:
- **S**ubproblems definition
- **R**elate subproblem solutions recursively
- **T**opological order to argue relation is acyclic and subproblems form a DAG
- **B**ase cases
- **O**riginal problem
- **T**ime analysis

Now let's talk about each step

### Subproblems definition

**Steps**
1. Define subproblems and describe their meaning **in words**. The subproblems defined here will **include parameters**. Specific techniques for defining subproblems will be mentioned later
2. The parameters of subproblems generally include **subsets of inputs**. Different DP problems may have different input formats: binary trees, single sequences, numbers, etc

Some tricks for defining subproblems:
- If the input is **a single sequence `A`**
	- Prefix form: Define `dp[i]` as the solution to the subproblem with input `A[:i+1]`, note that the interval representation here is based on Python and is left-closed and right-open
	- Suffix form: Define `dp[i]` as the solution to the subproblem with input `A[i:]`
	- Contiguous substrings of a sequence：Define `dp[i][j]` as the solution to the subproblem with input `A[i:j+1]`
- If the input is **double sequences `A` 和 `B`**
    - There are a total of $3 \times 3 = 9$ possible Cartesian products for the previous 3 types, depending on the specific situation
    - *For example, in the problem of finding the longest common subsequence(LCS), `dp[i][j]` can be used to represent the LCS of inputs `A[:i+1]` and `B[:j+1]`
- If the input is **number `k`**
    - Define `dp[k]` as the solution to the subproblem with input `K`.
- If the input is **a binary tree `r`**
    - Define `dp[r]` as the solution to the subproblem with input being the subtree with `r` as the root node. **In tree DP, pay attention to the relevance between "subproblem" and "subtree"**

> 📒 Advanced: Once you have mastered the general approach above, you can solve many dynamic programming problems. However, you may find it difficult to relate subproblems with your *loose* definition. When you have difficulties in associating subproblems, you can always try to **add constraints** to the subproblems you define or **expand the subproblems**. In MIT 6.006, this is called Subproblem Constraints and Expansion

Expanding subproblems is more common. Generally, this involves changing the form of `dp[i]` to `dp[i][j]`, which will also change the definition of your subproblems. *For example, in [198. House Robber](https://leetcode.com/problems/house-robber/), an additional state can be introduced to remember whether house `i` was robbed or not

I will also mention briefly what it means to add constraints to the definition of subproblems. *For example, if we define `dp[i]` as the solution with input `A[:i+1]`, assuming that each `A[i]` has two states of being selected or not, we can consider defining `dp[i]` as the solution with input `A[:i+1]` and **`A[i]` must be in the selected state** (this may be a bit abstract. I will provide a link to a problem that can be handled in this way later).

> 📒 You will find that **when defining subproblems, we don't think about how to calculate the value**. Don't start thinking about how to calculate the value when defining subproblems. When you use the SRTBOT framework to analyze, you will naturally know how to calculate it~


### Relate subproblem solutions recursively {#relate-subproblems}

"Identify a question about a subproblem solution that, if you knew the answer to, reduces the subproblem to smaller subproblem(s)"[^2]. If we try to write the formula, it will be likes this:

$$
dp[i] = f(dp[j_1],dp[j_2], ..., dp[j_k])\ where\ j_k <i
$$

Where $f$ is an abstract operation. It's the way how we relate different subproblem(s).

**The way to relate subproblem(s) is by making "decisions". Here comes the question, what decisions we get? Read the problem description and you will find the answer. **Then try to "locally brute-force" all possible answers to the question** :)

> 👻 It's fine that you find difficulties in finding the relationship at first. However, after getting enough practice, you will become comfortable.

> 🤔️ I found that when trying to recursively relate subproblem(s), it is always helpful to draw a dependency graph (nodes are subproblem and edges are "decisions"). The dependency graph not only clearly shows the relationship between subproblem(s) but also verify that there are overlapping subproblems
```python
                      dp[i] (apply f to aggregate results)
                     /  |   \
                 (?)/   |(?) \(?)
                   /    |     \
             dp[j_1] dp[j_2]   ...
                /   \  / \     / \
              ...   ...  ... ... ...
```


*For example, in the problem of [70. Climbing stairs](https://leetcode.com/problems/climbing-stairs/) on LeetCode, where each time one can climb one or two steps ($K=2$), reaching the `i`-th step must have come from either the `i-1`-th step or the `i-2`-th step. Since we are looking for all possible ways to climb the stairs, `dp[i] = dp[i-1] + dp[i-2]`*

```python
                    dp[i] (sum)
                     / \
    (climb one step)/   \(climb two steps)
                   /     \
             dp[i - 1]  dp[i - 2]
                /   \    /  \
              ...   ......  ...
```

> 🤔️ You can use this example to understand the meaning of "local" in "local brute-force": we only **use one decision one-time**. *For example, you may want to climb one step at a time and finally climb `n` steps and try to find a relationship betweena `dp[i]` and `dp[i - n]`*. That's not local!


### Topological order to argue relation is acyclic and subproblems form a DAG

You may have seen another name for dynamic programming problems elsewhere - "tabulation method". This is because when we use the bottom-up approach to solve dynamic programming problems, it can be seen as filling in a table. However, in my opinion, it is more helpful to understand dynamic programming by viewing it as a directed acyclic graph (DAG) - viewing subproblems as vertices on the graph and connecting "smaller subproblems" -> "larger subproblems" with directed edges. This graph will form a directed acyclic graph (DAG), and the dynamic programming algorithm is the process of traversing the topological order of the DAG

> 🤔️ Why is it a directed acyclic graph? Firstly, **directed edges represent the dependency between subproblems** and also **indicate the order in which we solve the subproblems**: to solve a subproblem, we need to **solve smaller subproblems first**. Secondly, it must be acyclic because dynamic programming remembers the subproblems that have been solved, and we cannot solve the same subproblem multiple times, so it must be acyclic

Understanding that order of solving subproblems in dynamic programming is the topological sorting of a DAG is very helpful in understanding tree DP problems. To solve a DP problem, we need to solve subproblems first. In the context of tree DP, the concepts of "subtree" and "subproblem" are equivalent. Therefore, solving subproblems first means calculating `dp[r.left]` and `dp[r.right]` before calculating `dp[r]`, which follows the post-order traversal order of a binary tree. Therefore, tree DP problems are often implemented through post-order traversal. If the tree is viewed as a graph (with directed edges pointing from child nodes to parent nodes), then the topological sorting is also in correspondence with the post-order traversal order

> 🤔️ When writing code, you may be unsure of how to update subproblems in the correct order. Just remember to **always solve smaller problems first** and then move to larger problems. Combined with the relationship equation among subproblems defined earlier, you will know what the update order is


### Base cases

Similar to the base case in recursive algorithms, it represents the solution to the independent smallest subproblems and is the starting point for deriving the relationship formula. This information can usually be determined by examining the problem description


### Original problem

Usually, the original problem corresponds to `dp[n]` or `dp[0]`, **but don't just memorize this**. You still need to combine the problem description with the form of the subproblems you defined to represent the original problem. However, don't worry about this because **this step is usually not the hard part of solving dynamic programming problems


### Time analysis

The dynamic programming algorithm is to solve all the subproblems that need to be solved and then calculates the original problem. Assuming there are a total of $n$ subproblems to be solved, the time complexity of dynamic programming can be solved using this formula:


$$
n * O(each\ subproblem) + O(original\ problem)
$$


## Get some practice

The only way to master this technique is *to get your hands dirty*. Only by applying the SRTBOT framework to dynamic programming problems can you appreciate its power. Below are some of the solutions to dynamic programming problems that I have written. In the solutions, I will talk about how to analyze dynamic programming problems using the SRTBOT framework. **I will continue to add the dynamic programming problems I have solved here, or you can also directly view the solutions on my Leetcode account👻**

> 📒 Note: My solution is not necessarily the optimal solution, *for example, sometimes we can use the technique of "state compression" to reduce space complexity, but I may not do this*, I **just apply the SRTBOT framework to these dynamic programming problems**

| Problem                                                                                              | Solution                                                                                                                    | Note                                     |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| [70. Climbing stairs](https://leetcode.com/problems/climbing-stairs/)                                | [Solution](https://leetcode.com/problems/climbing-stairs/solutions/3399398/solving-this-dp-problem-using-srtbot/)           | Number                                   |
| [746. Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/)             | [Solution](https://leetcode.com/problems/min-cost-climbing-stairs/solutions/3400428/solving-dp-problem-using-srtbot/)       | Single sequence                          |
| [198. House Robber](https://leetcode.com/problems/house-robber/)                                     | [Solution](https://leetcode.com/problems/house-robber/solutions/3401028/solving-dp-problems-using-srtbot/)                  | Single sequence + Subproblem expansion   |
| [322. Coin changes](https://leetcode.com/problems/palindromic-substrings/)                           | [Solution](https://leetcode.com/problems/coin-change/solutions/3404403/solving-dp-problem-using-srtbot/)                    | Number + non-$O(1)$ subproblem           |
| [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | [Solution](https://leetcode.com/problems/longest-increasing-subsequence/solutions/3409425/solving-dp-problem-using-srtbot/) | Single sequence + subproblem restriction |

## Refs

[^1]: [Dynamic programming - Wiki](https://en.wikipedia.org/wiki/Dynamic_programming)

[^2]: [Lecture 15 ~ 18 of MIT 6.006](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/video_galleries/lecture-videos/)

