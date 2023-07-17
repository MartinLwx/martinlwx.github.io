# 用 SRTBOT 框架分析动态规划问题


**Changelog**:
- [更新依赖图]({{< ref "#relate-subproblems"  >}}) @2023.04.13

## 引言

在做算法题的时候，让我头疼的经常是动态规划问题，它属于那种自己琢磨半天想不出来，但是看了答案之后会恍然大悟，下次再做的话很有可能又忘记了的一类问题。我也曾经看了很多人的题解，试图消化、吸收、应用他们的思路，但我**一直**找不到一个归纳得特别好的框架，似乎每个人都有自己解决动态规划的思路，将他们的思路应用在没有见过的动态规划问题的时候我总是遇到困难，而他们的方法轮*似乎*也无法让我处理所有的动态规划问题。这种寻寻觅觅对动态规划似懂非懂的状态，终于在我看完 [MIT6.006](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/) 的课程之后发生了改变，课上老师提出了解决动态规划问题的 6 个步骤——被称为 **SRTBOT 框架**，我发现它是**如此地好用**，因此我决定写下这篇博客来与大家分享🙌

> 👉 总的来说，这篇博客更适合已经对动态规划有了初步了解但是还没有找到系统方法论人的阅读

## 什么是动态规划问题
动态规划问题的两大特征：**重叠子问题**和**最优子结构**[^1]
- **重叠子问题**：解决一个动态规划问题往往是将它分解为若干个**重叠的子问题**
- **最优子结构**：即**大问题**的解可以通过组合**小问题**的**最优解**计算出来

前面的定义要特别注意的是「重叠」以及「子问题」，强调重叠是因为动态规划的厉害之处就在于它会记住解决过的子问题的答案，那么**重叠的子问题越多，动态规划的优势更加明显**

> 🤔️ 如果子问题并不重叠，那么就应该用分治法解决

上面是从定义的角度来谈动态规划问题的，从编程题目的角度来看，**下面两大类问题一般都是要用动态规划算法**：
- 最优化问题：**求最值 + 重叠子问题**
- 组合问题：**求解所有可能的解法的数量**

> 📒 或者，根据我的个人刷题经验来说：动态规划问题一般是可以暴力解决的递归问题，但是存在大量重叠子问题因此可以对其进行优化

## SRTBOT 框架

SRTBOT 是 6 个步骤的首字母缩写，分别是[^2]：
- **S**ubproblems definition
- **R**elate subproblem solutions recursively
- **T**opological order to argue relation is acyclic and subproblems form a DAG
- **B**ase cases
- **O**riginal problem
- **T**ime analysis

下面我来分别讲讲上面每一个步骤需要做些什么✈️

### 定义子问题（Subproblems definition）

**包含下面几个步骤**
1. 定义子问题并用文字描述其含义，这里定义的子问题会包含参数。**具体定义子问题的技巧会在后面提到**
2. 子问题的参数一般**包含输入的子集**，不同的动态规划问题可能有不同的输入格式：二叉树、单序列、数字等

在设计子问题的时候可以参考下面几种：
- 如果输入是**单个序列 `A`**
	- 前缀形式（Prefix form）：定义 `dp[i]` 表示输入为 `A[:i+1]` 的子问题的解，注意这里的区间表示是基于 Python 的，左闭右开
	- 后缀形式（Suffix form）：定义 `dp[i]` 表示输入为 `A[i:]` 的子问题的解
	- 子串形式（Contiguous substrings of a sequence）：定义 `dp[i][j]` 表示输入为 `A[i:j+1]` 的子问题的解
- 如果输入是**两个序列 `A` 和 `B`**
	- 为之前 3 种可能的笛卡尔乘积，一共有 $3\times 3=9$ 种，视具体情况而定
    - *比如最长公共子序列的问题，可以用 `dp[i][j]` 表示输入为 `A[:i+1]` 和 `B[:i+1]` 的最长公共子序列*
- 如果输入是**一个数字 `k`**
	- 定义 `dp[k]` 为输入为 `k` 的子问题的解
- 如果输入是**一棵二叉树**
    - 定义 `dp[r]` 为输入为以 `r` 为根节点的子树的这个子问题的解。**在树形 DP 中要注意「子问题」和「子树」这个对应的关系**

> 📒 进阶：掌握了上面的一般思路之后，其实已经能够解决很多动态规划问题，但你有时候会发现你定义的子问题**不够精确**，导致难以无法关联子问题，**当你在关联子问题有困难的时候总是可以尝试给你定义的子问题加上约束，或者是对子问题进行扩展**。在 MIT 6.006 中称为 Subproblem Constraints and Expansion

对子问题进行扩展会更为常见一些，一般此时就是将定义的 `dp[i]` 变成 `dp[i][j]` 这样的形式，**同时修改你的子问题定义**。*举例来说，在 [198. 打家劫舍](https://leetcode.cn/problems/house-robber/) 中可以考虑额外引入一个状态记住是否偷了房屋 `i`*

也稍微提一下对子问题的定义加上限制是什么意思，*比如我们定义 `dp[i]` 是输入为 `A[:i+1]` 的解，假设每个 `A[i]` 存在选中与不选中两个状态，那么可以考虑将 `dp[i]` 定义为 输入为 `A[:i+1]` **而且 `A[i]` 一定是选中状态**的时候的解*（这么说可能有点抽象，待我之后找到可以这样处理的问题后贴个链接在这）

> 📒 你会发现，**定义子问题的时候我们根本不会去思考要如何计算出来**，千万不要在定义子问题的时候就开始想要怎么计算出这个值，当你使用 SRTBOT 框架分析到最后，你就自然而然知道怎么计算了~


### 递归式关联子问题（Relate subproblem solutions recursively） {#relate-subproblems}

试着提出一个问题——如果你知道这个问题的答案，你就能够将这个大问题分解为更小的子问题。可能用数学公式表述会更明显一点：

$$
dp[i] = f(dp[j_1],dp[j_2], ..., dp[j_k])\ where\ j_k <i
$$
其中 $f$ 是一个抽象的操作，即「大问题」和「小问题」的关联方式

**上面提出的问题的答案一般就是“局部暴力枚举”**，这也是我们关联不同子问题的主要手段。方法是：看问题描述，从中归纳出允许的“决策”，**“决策”会导致我们从一个子问题转移到另外一个子问题**。思考的时候注意力放在 `nums[i]`、更小的子问题 `dp[j_k]` 和可行的决策这三者上

> 👻 如果一开始做动态规划想不出来也没关系，这东西熟练了之后就比较容易想到不同子问题如何通过不同的决策关联起来

> 🤔️ 我发现，在想办法递归关联子问题的时候，画出依赖图总是能给我很大帮助（结点为子问题，边为“决策”），依赖图不仅清晰展示了子问题之间的关联方式，还可以验证有重叠子问题出现
```python
                      dp[i] (apply f to aggregate results)
                     /  |   \
                 (?)/   |(?) \(?)
                   /    |     \
             dp[j_1] dp[j_2]   ...
                /   \  / \     / \
              ...   ...  ... ... ...
```

*举例来说，在 [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/) 问题中，每次只能爬一个台阶或者两个台阶（$K=2$），那么爬到第 `i` 个台阶一定是从第 `i - 1` 个台阶和第 `i - 2` 个台阶过来的，又因为求解的是所有可能的走法，因此 `dp[i] = dp[i - 1] + dp[i - 2]`，画出依赖图如下*
```python
                    dp[i] (sum)
                     / \
    (climb one step)/   \(climb two steps)
                   /     \
             dp[i - 1]  dp[i - 2]
                /   \    /  \
              ...   ......  ...
```

> 🤔️ 可以通过这个例子理解“局部暴力枚举”中的“局部”的含义：我们只考虑做一次决策，*比如上面爬楼梯，每次爬一个台阶最后连续爬 `n` 个台阶 关联 `dp[i]` 和 `dp[i - n]` 这种方式就不是局部*


### 根据拓扑排序确定解决子问题的顺序（Topological order to argue relation is acyclic and subproblems form a DAG）

你可能在别的地方看到过动态规划问题的另外一个名字——“表格法”。因为当我们采用“自底向上”的解法解决动态规划问题的时候可以看成是在填表格。但在我看来，**将动态规划看成一张有向无环图会更有助于对动态规划的理解**——将子问题看成是图上的顶点，用**有向边连接「小的子问题」-> 「大的子问题」**，这个图会构成一个有向无环图（DAG），**动态规划算法其实就是 DAG 的拓扑排序过程**。即「DAG 拓扑排序」=「自底向上方法」=「表格法」。上面的依赖图“自底向上”就是在做拓扑排序

> 🤔️ 为什么是有向无环图？首先，有向边**表示了子问题之间的依赖关系**也表示了**我们求解子问题的顺序**：要解决一个子问题，需要先求解出更小的子问题；其次，它必须是无环的，因为动态规划会记住求解过的子问题，我们**不可能多次求解同一个子问题**，因此它必须是无环的。

意识到动态规划的子问题求解顺序是 DAG 的拓扑排序有什么用呢？这在**很大程度上帮我理解了树形 DP 问题**：动态规划要先求解子问题 -- 在树形 DP 中，显然“子树”和“子问题”的概念是对应的，因此要先解决子问题其实意味着计算 `dp[r]` 的时候应该先计算 `dp[r.left]` 和 `dp[r.right]`，而这恰恰遵循了二叉树的后序遍历顺序，**因此树形 DP 问题往往是通过后序遍历实现的**。如果把树看成图的话（注意边由孩子结点指向父节点），那么拓扑排序也是和后序遍历的顺序对应的

> 🤔️ 实际写代码的时候，你可能对如何用正确的顺序更新子问题有疑问，只要记住**永远都是先解决小问题，然后才到大问题**，再看一眼前面定义的关系式，你就知道更新顺序是什么了

### 边界情况（Base cases）

类似递归算法中的 base case，表示**独立的**最小子问题的解，是关系式推导的起点，**这种信息一般可以通过看题目看出来**

### 原始问题（Original problem）

通常来说，原问题一般都对应 `dp[n]` 或者 `dp[0]`，**但是不要死记硬背**，还是要结合题意、你定义的子问题形式。但不必担心，**因为这一步通常不是求解动态规划问题的难点**

### 时间复杂度分析（Time analysis）

动态规划算法就是求解所有需要求解的子问题，然后计算原始问题，假设一共有 $n$ 个子问题要求解，那么动态规划的时间复杂度可以用下面这个公式解决：

$$
n * O(each\ subproblem) + O(original\ problem)
$$


## 应用

光是谈理论总是一知半解，只有真正开始把 SRTBOT 框架应用于动态规划问题才能够体会它的强大之处，下面是我整理的部分动态规划问题的题解，在题解里面我会谈到如何用 SRTBOT 分析动态规划问题。**后序会不断把做的动态规划的题目搬运到这里来，或者也可以直接看 Leetcode 账号查看题解👻**

> 📒 注：我的解法并不一定是最优的解法，*比如有时候可以用「状态压缩」的技巧减少空间复杂度等，我可能并不会这么做*，我**只是将 SRTBOT 框架应用于这些动态规划问题**

| Problem                                                                                                           | Solution                                                                                                                                   | Note                 |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------- |
| [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)                                                       | [题解](https://leetcode.cn/problems/climbing-stairs/solution/srtbot-kuang-jia-jie-jue-dong-tai-gui-hu-1vo5/)                               | 数                   |
| [746. 使用最小花费爬楼梯](https://leetcode.cn/problems/min-cost-climbing-stairs/)                                 | [题解](https://leetcode.cn/problems/min-cost-climbing-stairs/solution/yong-srtbot-kuang-jia-jie-jue-dong-tai-g-imb0/)                      | 单序列               |
| [198. 打家劫舍](https://leetcode.cn/problems/house-robber/)                                                       | [题解](https://leetcode.cn/problems/house-robber/solution/yong-srtbot-kuang-jia-jie-jue-dong-tai-g-cdib/)                                  | 单序列 + 扩展子问题  |
| [322. 零钱兑换](https://leetcode.cn/problems/coin-change/solution/yong-srtbot-kuang-jia-jie-jue-dong-tai-g-2c9s/) | [题解](https://leetcode.cn/problems/coin-change/solution/yong-srtbot-kuang-jia-jie-jue-dong-tai-g-2c9s/)                                   | 数 + 非$O(1)$ 子问题 |
| [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)                               | [题解](https://leetcode.cn/problems/longest-increasing-subsequence/solution/srtbot-dp-by-martinlwx-d73e/)                                  | 单序列 + 限制子问题  |
| [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)                                    | [题解](https://leetcode.cn/problems/longest-palindromic-substring/solution/yong-srtbot-kuang-jia-jie-jue-dong-tai-g-i29p/)                 | 子串                 |
| [91. 解码方法](https://leetcode.cn/problems/decode-ways/)                                                         | [题解](https://leetcode.cn/problems/decode-ways/solution/yong-srtbot-kuang-jia-jie-jue-dong-tai-g-gd54/)                                   | 单序列               |
| [139. 单词拆分](https://leetcode.cn/problems/word-break/)                                                         | [题解](https://leetcode.cn/problems/word-break/solution/yong-srtbot-jie-jue-dong-tai-gui-hua-wen-p3um/)                                    | 单序列               |
| [279. 完全平方数](https://leetcode.cn/problems/perfect-squares/)                                                  | [题解](https://leetcode.cn/problems/perfect-squares/solution/yong-srtbot-kuang-jia-jie-jue-dong-tai-g-uuyg/)                               | 单数字               |
| [673. 最长递增子序列的个数](https://leetcode.cn/problems/number-of-longest-increasing-subsequence/)               | [题解](https://leetcode.cn/problems/number-of-longest-increasing-subsequence/solution/tong-shi-ji-zhu-lischang-du-he-ge-shu-de-sgrt/)      | 单序列               |
| [62. 不同路径](https://leetcode.cn/problems/unique-paths/)                                                        | [题解](https://leetcode.cn/problems/unique-paths/solution/yong-srtbot-kuang-jia-chu-li-dong-tai-gu-za9c/)                                  | 网格                 |
| [1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)                                  | [题解](https://leetcode.cn/problems/longest-common-subsequence/solution/srtbot-kuang-jia-jie-jue-jing-dian-lcs-w-ebjf/)                    | 双序列 + 扩展子问题  |
| [309. 最佳买卖股票时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)      | [题解](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/solution/you-dian-bu-yi-yang-de-dong-tai-gui-hua-scaqf/) | 单序列 + 扩展子问题  |
| [1911. 最大子序列交替和](https://leetcode.cn/problems/maximum-alternating-subsequence-sum/)                       | [题解](https://leetcode.cn/problems/maximum-alternating-subsequence-sum/solution/cong-ling-kai-shi-de-dp-tui-dao-si-lu-by-crrd/)           | 单序列 + 扩展子问题  |
| [1220. 统计元音字母序列的数目](https://leetcode.cn/problems/count-vowels-permutation/)                            | [题解](https://leetcode.cn/problems/count-vowels-permutation/solution/jian-dan-dong-tai-gui-hua-by-martinlwx-5ien/)                        | 数 + 扩展子问题      |



## 参考

[^1]: [Dynamic programming - Wiki](https://en.wikipedia.org/wiki/Dynamic_programming)

[^2]: [Lecture 15 ~ 18 of MIT 6.006](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/video_galleries/lecture-videos/)
