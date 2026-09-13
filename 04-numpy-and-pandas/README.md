# 第一章：NumPy 与 Pandas——开始处理数据

> 本章目标：使用 NumPy 完成基础数组运算，使用 Pandas 读取、查看、筛选、统计和保存表格数据。

## 1. 准备项目

在 VS Code 中创建并打开文件夹 `chapter01-data`，建议结构如下：

```text
chapter01-data/
├─ data/
│  └─ students.csv
├─ output/
└─ main.py
```

激活上一章创建的环境，确认库可导入：

```powershell
conda activate data-practice
python -c "import numpy, pandas; print('环境可用')"
```

在代码中通常使用约定俗成的简写：

```python
import numpy as np
import pandas as pd
```

## 2. NumPy 数组

### 2.1 创建和查看数组

```python
import numpy as np

scores = np.array([78, 85, 92, 66])
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

print(scores)
print(scores.shape)   # (4,)
print(matrix.shape)   # (2, 3)
print(scores.dtype)
print(scores.ndim)
```

- `shape`：每个维度有多少个元素。
- `dtype`：数组中元素的数据类型。
- `ndim`：数组维度数量。
- `size`：元素总数。

### 2.2 索引和切片

```python
print(scores[0])       # 第一个
print(scores[-1])      # 最后一个
print(scores[1:3])     # 下标 1 和 2
print(matrix[0, 1])    # 第 1 行第 2 列
print(matrix[:, 1])    # 所有行的第 2 列
```

NumPy 和 Python 列表一样从 0 开始计数，切片不包含结束位置。

### 2.3 数组运算

```python
prices = np.array([10.0, 20.0, 30.0])

print(prices + 2)
print(prices * 0.9)
print(prices ** 2)
```

运算会逐元素执行。Python 列表乘以 2 会重复内容，NumPy 数组乘以 2 会把每个数乘以 2：

```python
print([1, 2, 3] * 2)             # [1, 2, 3, 1, 2, 3]
print(np.array([1, 2, 3]) * 2)   # [2 4 6]
```

形状不兼容的数组不能随意运算。遇到报错时先打印两边的 `shape`。

### 2.4 常用统计

```python
scores = np.array([78, 85, 92, 66])

print(scores.sum())
print(scores.mean())
print(scores.min())
print(scores.max())
print(scores.std())
print(np.median(scores))
```

NumPy 很适合处理同类型的大量数值；普通列表更灵活，可以混合保存不同类型。表格数据则通常交给 Pandas。

## 3. Pandas 的 Series 和 DataFrame

`Series` 可以理解为“带索引的一列数据”：

```python
scores = pd.Series([82, 91, 76], name="score")
print(scores)
```

`DataFrame` 是有行、有列的二维表格：

```python
students = pd.DataFrame({
    "name": ["小明", "小红", "小刚"],
    "group": ["A", "B", "A"],
    "score": [82, 91, 76],
})

print(students)
```

## 4. 读取和保存 CSV

CSV 是用分隔符保存表格的纯文本文件。读取：

```python
df = pd.read_csv("data/students.csv")
```

保存：

```python
df.to_csv("output/students_result.csv", index=False, encoding="utf-8-sig")
```

`index=False` 可以避免额外保存 DataFrame 的行索引。`utf-8-sig` 便于部分 Windows 表格软件正确显示中文。

路径错误是初学者最常见的问题。如果提示文件不存在，检查 VS Code 是否打开了整个 `chapter01-data` 文件夹，并在该目录的终端运行程序。

## 5. 拿到表格后先看什么

```python
print(df.head())            # 前 5 行
print(df.tail())            # 后 5 行
print(df.shape)             # (行数, 列数)
print(df.columns)           # 列名
print(df.dtypes)            # 每列类型
df.info()                   # 非空数量和类型
print(df.describe())        # 数值列基础统计
```

不要一上来打印全部大表。先确认行列规模、列名、类型和少量样例。

## 6. 选择行和列

选择一列得到 `Series`：

```python
scores = df["score"]
```

选择多列得到 `DataFrame`：

```python
summary = df[["name", "score"]]
```

按标签选择使用 `loc`，按整数位置选择使用 `iloc`：

```python
print(df.loc[0, "name"])
print(df.loc[0:2, ["name", "score"]])
print(df.iloc[0:3, 0:2])
```

注意：`loc[0:2]` 通常包含标签 2，`iloc[0:3]` 不包含位置 3。

## 7. 条件筛选

```python
passed = df[df["score"] >= 60]
group_a = df[df["group"] == "A"]
selected = df[(df["score"] >= 80) & (df["attendance"] >= 0.9)]
```

多个条件必须分别加括号，并使用 `&` 表示“且”、`|` 表示“或”，不能直接使用普通 Python 的 `and`、`or`。

字符串条件可以使用：

```python
result = df[df["city"].isin(["北京", "上海"])]
```

## 8. 排序与修改

```python
sorted_df = df.sort_values("score", ascending=False)
df.loc[df["score"] > 100, "score"] = 100
```

多数 Pandas 操作会返回新对象。为了让过程清楚，初学阶段建议把结果赋给有意义的新变量，不要到处使用 `inplace=True`。

## 9. 增加与删除列

```python
df["total"] = df["score"] + df["bonus"]
df["passed"] = df["total"] >= 60

clean_df = df.drop(columns=["temporary_note"])
```

删除前确认列是否真的无用。原始数据最好保留，处理结果保存到 `output`，不要覆盖唯一的原始文件。

## 10. 计数、分组和统计

类别计数：

```python
print(df["group"].value_counts())
print(df["group"].value_counts(normalize=True))
```

分组统计：

```python
group_mean = df.groupby("group")["score"].mean()
print(group_mean)
```

同时计算多项结果：

```python
group_summary = (
    df.groupby("group", as_index=False)
      .agg(
          student_count=("name", "count"),
          average_score=("score", "mean"),
          highest_score=("score", "max"),
      )
)
```

## 11. 简单合并

假设一张表有成绩，另一张表有班级信息，可以根据共同的 `student_id` 合并：

```python
result = pd.merge(scores_df, students_df, on="student_id", how="left")
```

- `on`：使用哪一列匹配。
- `how="left"`：保留左表全部记录。

合并前检查两表的键是否重复，合并后检查行数。键重复可能让行数意外增加。

## 12. 识别缺失值

```python
print(df.isna().sum())
missing_rows = df[df["score"].isna()]
```

此处先识别，不急着删除或填补。缺失值如何处理取决于字段含义，第三章会专门讲解。

## 13. 综合实验：学习活动数据统计

### 实验背景

某学习小组记录了成员所在组别、练习时长、测验成绩和出勤率。你需要生成一份适合老师查看的分组结果表。

### 数据准备

在 `data/students.csv` 中保存以下内容：

```csv
student_id,name,group,study_hours,score,attendance
S001,小明,A,6.5,82,0.95
S002,小红,B,8.0,91,0.98
S003,小刚,A,3.5,58,0.80
S004,小林,B,5.0,76,0.92
S005,小雨,C,7.0,88,0.96
S006,小周,A,4.0,65,0.85
S007,小夏,C,2.5,54,0.78
S008,小陈,B,6.0,84,0.94
S009,小杨,C,5.5,79,0.90
S010,小安,A,9.0,96,1.00
```

### 任务要求

在 `main.py` 中按顺序完成：

1. 读取 CSV，输出前 5 行。
2. 输出行列数、列名、数据类型和缺失值数量。
3. 筛选成绩不低于 80 且出勤率不低于 0.9 的记录。
4. 按成绩从高到低排序。
5. 新增 `passed` 列，成绩不低于 60 为 `True`。
6. 新增 `efficiency` 列，计算 `score / study_hours`，保留两位小数。
7. 按 `group` 分组，计算人数、平均成绩、平均学习时长和及格人数。
8. 把筛选结果保存为 `output/excellent_students.csv`。
9. 把分组结果保存为 `output/group_summary.csv`。

（图片 028：实验完成后 VS Code 终端中的分组统计结果与 output 目录中的两个 CSV 文件）

### 起步代码

```python
from pathlib import Path

import pandas as pd


DATA_PATH = Path("data/students.csv")
OUTPUT_DIR = Path("output")


def main() -> None:
    OUTPUT_DIR.mkdir(exist_ok=True)
    df = pd.read_csv(DATA_PATH)

    # TODO：按实验要求完成查看、筛选、排序、计算和保存


if __name__ == "__main__":
    main()
```

### 验收标准

- 原始 CSV 没有被修改。
- 筛选结果只包含同时满足两个条件的记录。
- 排序方向正确。
- 分组结果每个组一行，并包含要求的四项统计。
- 两个输出文件均不包含多余索引列。
- 再次运行程序时仍能正常生成结果。

### 思考题

- 为什么分组结果中的“人数”和“及格人数”含义不同？
- 学习效率 `score / study_hours` 是否一定能公平评价学习者？
- 如果 `study_hours` 为 0，计算会出现什么问题？应该怎样处理？

### 扩展作业

另建 `clubs.csv`，包含 `student_id` 和 `club` 两列。使用 `merge` 把社团信息合并到原表，检查合并前后行数是否一致，并说明缺少社团信息时为什么会出现空值。

## 14. 本章检查清单

- [ ] 我能区分 Python 列表、NumPy 数组、Series 和 DataFrame。
- [ ] 我会读取、查看和保存 CSV。
- [ ] 我会选择、筛选、排序和创建列。
- [ ] 我会使用 `value_counts`、`groupby` 和简单 `merge`。
- [ ] 我会在处理前后检查数据形状和缺失值。
