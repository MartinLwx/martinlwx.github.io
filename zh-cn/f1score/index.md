# 从混淆矩阵到 F1 score


## 什么是混淆矩阵

---

**每一列表示实际情况，每一行表示我们的预测**，这样组合起来就得到了一个混淆矩阵，比如一个二分类的任务，可以画出如下的混淆矩阵⬇️

<center>

|       | Positive            | Negative            |
| ----- | ------------------- | ------------------- |
| True  | TP = True Positive  | FP = False Positive |
| False | FN = False Negative | TN = True Negative  |

</center>

### accuracy

> 有多少样本我们预测正确了

预测正确的其实有下面两种情形⬇️

- TP：本来是 positive，你也认为是 positive 的
- TN：本来是 negative，你也认为是 negative 的

那么用 **TP + TN**（其实就是主对角线）除以总的样本数就可以得到 accuracy，

📒或者**直接看表格，其实就是 $\frac{主对角线}{四个单元格} =\frac{TP+TN}{TP+TN+FN+FP}$**

一般来说 accuracy 是好用的评估指标，但是在某些情况下不是这样子的，比如样本数据不均衡的时候

<center>

|              | 类别 A | 类别 B |
| ------------ | ------ | ------ |
| 判断是类别 A | 0      | 0      |
| 判断是类别 B | 1      | 99     |

</center>

假设类别 A 有 1 个，类别 B 有 99 个，如果不管输入的是什么都返回类别 B 的正确率是多少呢？

答案是 $\frac{0+99}{0+0+1+99}=99\%$，我们可以说这个分类器效果很好吗，这显然是很荒谬的😂

### Precision

> 所有预测为 True 的样本里面，有多少是真的 positive 的

从定义出发可以知道，其实就是表格中**判断为 True 的这一行中 positive 的比例**，那么就是 $\frac{TP}{TP+FP}$

### Recall

> 所有本来是 positive 的，有多少被我们成功预测了出来

从定义出发，所有本来是 positive 的就是 **Positive 这一列之和**，我们成功预测的是 TP，那么就是 $\frac{TP}{TP+FN}$

📒**我们常常要在 Precision 和 Recall 中进行 tradeoff，因为其中一个升高，另一个就会降低**

## F1 score

---

正是因为 Precision 和 Recall 的互斥特性，在衡量分类器好坏的时候会给我们带来困扰，特别是两者相差不多的情况下，于是就需要将这两个指标合并起来用另一个指标表示，也就是 F1 score

F1 score 的计算方式如下：

$$F1\ score = \frac{2PR}{P+R}$$

其中 Precision = $P$； Recall = $R$

### F1 score 的不同计算方法

1. Macro：分别计算每个类别的 $P$ 和 $R$，再计算总的平均的 $P$ 和 $R$，最后用这个计算 F1 score
2. Micro：合并多个类别的统计结果到一个表格里面，在分别算 $P$ 和 $R$ 和 F1 score

配合后面的例子🌰食用

### F1 score 的几个规律

1. F1 score 永远在 precision 和 recall 之间，
2. F1 score 会给低的部分（$P\ or\ R$）更多的权重，所以如果算术平均值是一样的情况下，哪个分类器的短板更短，它的 F1 就会越差
   1. 情况一：比如 $P$ 和 $R$ 分别是 $60\%$ 和 $60\%$，可以算他们的 F1 score = $60\%$
   2. 情况二：比如 $P$ 和 $R$ 分别是 $50\%$ 和 $70\%$，可以看到他们和上一种情况的平均值是一样的，但是他们的 F1 score = $58.3\%$
3. 由 1. 可知：📒 高 F1 score 不意味着分类器更好或更适合你的任务，有时候你可能更关注 Precision 或者 Recall 这两个中的一个

## 一个完整的例子🌰

---

<center>


|                | class 0 | class 1 | class 2 |
| -------------- | ------- | ------- | ------- |
| predict_class0 | 2       | 0       | 0       |
| predict_class1 | 1       | 0       | 1       |
| predict_class2 | 0       | 2       | 0       |

</center>

### Macro 的计算方式

要先得到每一个类别的 Precision 和 Recall，先画出表格⬇️

<center>

|                    | class 0 | Not class 0 |
| ------------------ | ------- | ----------- |
| predict_class0     | 2       | 1           |
| predict_not_class0 | 0       | 3           |

</center>

<center>

|                    | class 1 | Not class 1 |
| ------------------ | ------- | ----------- |
| predict_class1     | 0       | 2           |
| predict_not_class1 | 2       | 2           |

</center>

<center>

|                    | class 2 | Not class 2 |
| ------------------ | ------- | ----------- |
| predict_class2     | 0       | 1           |
| predict_not_class2 | 2       | 3           |

</center>

可以得到如下的结果

- class 0：Precision = $\frac{2}{3}$；Recall = $1$
- class 1：Precision = $0$；Recall = $0$
- class 2：Precision = $0$；Recall = $0$
- 那么平均的 Precision 就是 $\frac{2}{3} * \frac{1}{3} = \frac{2}{9}$；平均的 Recall 是 $\frac{1}{3}$，代入 F1 score 的计算公式可以得到 $\frac{4}{15}=0.26666$

### Micro 的计算方式

将三个表格**叠起来**（对应位置相加），可以得到如下的表格

<center>

|                    | class ? | Not class ? |
| ------------------ | ------- | ----------- |
| predict_class?     | 2       | 4           |
| predict_not_class? | 4       | 8           |

</center>

代入公式算得 F1 score = $1/3=0.333333$

## 代码验证

---

```Python
from sklearn.metrics import f1_score, confusion_matrix

y_true = [0, 1, 2, 0, 1, 2]
y_pred = [0, 2, 1, 0, 0, 1]

print(confusion_matrix(y_true, y_pred))             # confusion matrix
# [[2 0 0]
#  [1 0 1]
#  [0 2 0]]

print(f1_score(y_true, y_pred, average='macro'))    # 0.26666

print(f1_score(y_true, y_pred, average='micro'))    # 0.33333
```

## 参考

---

1. [sklearn 的 `f1_score` 介绍](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)


