# 第四章：第一次使用机器学习工具

> 本章目标：理解特征、标签、训练集和验证集，使用 Scikit-learn 完成一次分类任务和一次回归任务的完整流程。

## 1. 机器学习任务在做什么

监督学习从已有样本中学习“输入到答案”的关系，再对没有答案的新样本进行预测。

- **特征 `X`**：用于预测的信息，通常是多列数据。
- **标签 `y`**：希望模型预测的答案，通常是一列。
- **样本**：一行记录。
- **模型**：从训练数据中学到规律、接收特征并输出预测的程序对象。

分类预测离散类别，例如是否通过；回归预测连续数值，例如价格或用时。

## 2. 为什么要划分数据

如果只在训练过的数据上评分，就像背完答案后再做同一张试卷，无法判断模型面对新数据的能力。通常把数据分成：

- **训练集**：用于 `fit`，让模型学习。
- **验证集**：训练时不参与学习，用于检查新数据上的表现。

可以先把完整过程记成下面这条文字流程：

```text
原始数据
  -> 分出特征 X 和标签 y
  -> train_test_split 划分训练集与验证集
  -> model.fit(X_train, y_train)
  -> model.predict(X_valid)
  -> 用 y_valid 评价预测结果
```

最重要的边界是：验证集用于检查结果，不能提前交给模型学习。

```python
from sklearn.model_selection import train_test_split

X_train, X_valid, y_train, y_valid = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)
```

`test_size=0.2` 表示保留 20% 做验证。`random_state` 固定随机划分，使别人能复现实验。分类中常加入 `stratify=y`，尽量保持两部分的类别比例。

## 3. Scikit-learn 的统一用法

大多数模型遵循相似接口：

```python
model = SomeModel(...)
model.fit(X_train, y_train)
predictions = model.predict(X_valid)
```

- `fit`：用训练特征和训练标签学习。
- `predict`：对新特征生成预测。

输入的行数必须对应：`X_train` 的每一行都有一个 `y_train`。预测时特征列、顺序和处理方式必须与训练时一致。

## 4. 第一次分类

下面使用 Scikit-learn 自带的鸢尾花数据，不需要联网下载：

```python
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split

data = load_iris(as_frame=True)
X = data.data
y = data.target

X_train, X_valid, y_train, y_valid = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
    stratify=y,
)

model = LogisticRegression(max_iter=500)
model.fit(X_train, y_train)
predictions = model.predict(X_valid)

print("验证集样本数：", len(y_valid))
print("前 5 个预测：", predictions[:5])
print("Accuracy：", accuracy_score(y_valid, predictions))
```

此处重点是跑通流程，不推导算法公式。Accuracy 的详细解释放在下一章。

## 5. 第一次回归

使用自带糖尿病进展数据完成数值预测：

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error
from sklearn.model_selection import train_test_split

data = load_diabetes(as_frame=True)
X = data.data
y = data.target

X_train, X_valid, y_train, y_valid = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42,
)

model = LinearRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_valid)

print("前 5 个真实值：", y_valid.iloc[:5].to_list())
print("前 5 个预测值：", predictions[:5])
print("MAE：", mean_absolute_error(y_valid, predictions))
```

预测值与真实值通常不会完全相同。模型不是查字典，而是在根据训练数据做近似预测。

## 6. 常见错误

- 把标签列同时留在 `X` 中，模型相当于偷看答案。
- 对训练集和验证集分别随意创建不同列。
- 在 `predict` 前忘记 `fit`。
- 分类标签与回归数值混淆，选错模型。
- 只打印分数，不保留划分方式和随机种子。
- 用验证集反复调到满意后，还把它称为完全未知数据。

## 7. 实验一：小型分类任务

### 背景

使用鸢尾花的花萼和花瓣测量值预测植物类别。数据内置，字段可通过 `data.DESCR` 查看。

### 任务

1. 创建 `classification.py`，加载数据并打印 `X.shape`、`y.value_counts()` 和字段名。
2. 以 20% 作为验证集，设置 `random_state=42` 和 `stratify=y`。
3. 训练 `LogisticRegression(max_iter=500)`。
4. 输出前 10 个真实类别与预测类别。
5. 计算 Accuracy。
6. 创建 `output/classification_predictions.csv`，包含 `actual`、`predicted` 和 `correct`。
7. 在 `experiment.md` 中用自己的话解释 `fit` 和 `predict`。

### 验收

- 标签没有出现在特征中。
- 训练集和验证集没有混用。
- 输出 CSV 行数等于验证集样本数。
- 固定随机种子后重复运行结果一致。

## 8. 实验二：小型回归任务

### 背景

使用内置糖尿病数据的 10 个数值特征预测连续目标。该数据只用于练习工具流程，不能把结果当作医疗建议。

### 任务

1. 创建 `regression.py`，查看数据规模、字段和目标统计量。
2. 按 80%/20% 划分数据，设置 `random_state=42`。
3. 训练 `LinearRegression`。
4. 计算 MAE。
5. 保存 `output/regression_predictions.csv`，包括 `actual`、`predicted`、`absolute_error`。
6. 找出绝对误差最大的 5 条验证记录，但不要据此删除它们。

### 验收

- 完整经历读取、查看、划分、训练、预测、保存。
- MAE 由验证集真实值和验证集预测值计算。
- 能解释为什么回归预测可能带小数。
- 不对数据含义作未经证实的因果或医学解释。

## 9. 对照记录

在 `experiment.md` 中完成：

| 项目 | 分类任务 | 回归任务 |
| --- | --- | --- |
| 标签表示什么 |  |  |
| 使用的模型 |  |  |
| 预测输出形式 |  |  |
| 基础评价指标 |  |  |

## 10. 本章检查清单

- [ ] 我能区分特征 `X` 和标签 `y`。
- [ ] 我能区分分类与回归。
- [ ] 我理解训练集和验证集的作用。
- [ ] 我会使用 `train_test_split`、`fit` 和 `predict`。
- [ ] 我完成了分类与回归两个完整流程。
