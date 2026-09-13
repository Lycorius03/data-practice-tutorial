# 第二章：数据探索、基础统计与可视化

> 本章目标：面对一份陌生数据，能够有顺序地提出问题、计算基础统计量、选择合适图表，并用证据写出简短结论。

## 1. 数据探索不是“随便画图”

数据探索的任务是发现数据大致长什么样、是否可信，以及哪些现象值得继续研究。一个稳妥的顺序是：

```text
理解字段含义 → 检查规模与类型 → 检查缺失和重复
→ 查看单个变量分布 → 比较不同群体 → 查看变量关系 → 总结发现与限制
```

每张图都应回答一个问题。不要先画很多图，再试图为它们编故事。

## 2. 拿到陌生数据先检查什么

```python
import pandas as pd

df = pd.read_csv("data/activity.csv")
print(df.head())
print(df.shape)
print(df.columns)
print(df.dtypes)
df.info()
print(df.isna().sum())
print("重复行：", df.duplicated().sum())
```

还要阅读数据说明，确认每一行代表什么、单位是什么、数据在什么时间和条件下采集。没有字段说明时，不要仅凭列名猜测。

## 3. 数值数据和类别数据

- **数值数据**可以进行有意义的算术运算，例如年龄、时长、价格。
- **类别数据**表示分组或标签，例如城市、设备类型、是否通过。

编号即使由数字组成，也可能是类别，例如 `student_id=1001`。对编号计算平均值通常没有意义。日期也需要转换为日期类型，不能简单当普通字符串处理。

## 4. 基础统计的直观理解

```python
column = df["score"]

print(column.min())
print(column.max())
print(column.mean())
print(column.median())
print(column.var())
print(column.std())
```

- **均值**：所有值之和除以数量，容易受到极端值影响。
- **中位数**：排序后位于中间的值，对极端值更稳健。
- **方差与标准差**：反映数据相对均值的分散程度。标准差越大，通常说明差异越大。

统计量必须结合单位和分布解释。平均学习时长 5 小时，并不表示每个人都接近 5 小时。

## 5. 类别数量与分组比较

```python
print(df["group"].value_counts())
print(df["group"].value_counts(normalize=True))

comparison = df.groupby("group")["score"].agg(["count", "mean", "median", "std"])
print(comparison)
```

比较均值之前先看每组数量。样本很少的组，其结果可能不稳定。还要结合中位数、分布和异常点，而不是只看一个平均数。

## 6. 相关性不等于因果

```python
correlation = df[["study_hours", "score", "sleep_hours"]].corr()
print(correlation)
```

相关系数接近 1 表示较强的同向线性关系，接近 -1 表示较强的反向线性关系，接近 0 表示线性关系不明显。但相关不能证明因果。

即使学习时长与成绩相关，也不能直接断言“增加学习时长一定导致成绩提升”。原有基础、课程难度、记录误差等因素都可能同时影响二者。图表和相关系数提供线索，不自动提供因果结论。

## 7. Matplotlib 基础

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(8, 5))
# 在这里绘图
plt.title("Chart title")
plt.xlabel("X label")
plt.ylabel("Y label")
plt.tight_layout()
plt.savefig("output/chart.png", dpi=150)
plt.close()
```

本教程建议保存图片后再查看。`plt.close()` 可以避免连续绘图时内容叠加。若中文字体显示为方块，可先使用英文图题和轴标题，不要从未知网站下载字体文件。

## 8. 四种常用图表

### 8.1 柱状图：比较类别

```python
counts = df["group"].value_counts().sort_index()
counts.plot(kind="bar")
```

柱状图比较不同类别的数量或统计值。类别轴不是连续刻度，通常不应改成折线图。

### 8.2 直方图：观察数值分布

```python
df["score"].plot(kind="hist", bins=10, edgecolor="black")
```

`bins` 是区间数量。区间太少会隐藏细节，太多会产生噪声，应尝试几个合理值。

### 8.3 散点图：观察两个数值变量

```python
plt.scatter(df["study_hours"], df["score"], alpha=0.7)
```

观察整体方向、点的密集区域和远离其他点的记录。散点关系不一定是直线。

### 8.4 箱线图：比较分布与可疑异常点

```python
df.boxplot(column="score", by="group")
plt.suptitle("")
```

箱线图中的独立点表示统计意义上的潜在异常，并不等于错误数据。它可能是真实但少见的情况，需要回到原始记录核实。

## 9. 从问题到结论

好的探索问题应具体，例如：

- 各组样本数量是否均衡？
- 不同组的成绩中位数是否明显不同？
- 学习时长与成绩是否存在可见的线性趋势？
- 哪些记录可能是异常点？

结论应包含“观察到什么”和“不能说明什么”：

> A 组的成绩中位数高于 B 组，但 A 组样本更少，且两组分布有明显重叠。当前结果只能说明这份数据中的差异，不能证明组别导致成绩变化。

## 10. 综合实验：学习行为探索报告

### 实验背景

老师提供 `data/activity.csv`，每行是一名学习者的匿名记录，包含：`student_id`、`group`、`study_hours`、`sleep_hours`、`score`、`completed_tasks`。本实验只探索，不擅自删除或修改原始数据。

### 实验产物

```text
chapter02-exploration/
├─ data/activity.csv
├─ output/
│  ├─ group_counts.png
│  ├─ score_distribution.png
│  ├─ hours_score_scatter.png
│  ├─ group_score_boxplot.png
│  └─ summary.csv
├─ analysis.py
└─ report.md
```

### 任务步骤

1. 读取数据，记录行列数、字段、类型、缺失数量和重复数量。
2. 对数值列计算最小值、最大值、均值、中位数和标准差。
3. 统计各组数量和比例。
4. 生成四幅图：组别柱状图、成绩直方图、学习时长与成绩散点图、分组成绩箱线图。
5. 计算数值变量的相关系数表。
6. 生成 `summary.csv`，每组包含样本量、成绩均值、中位数和标准差。
7. 在 `report.md` 中回答：哪组人数最多、哪些组差异明显、是否存在可疑异常点、哪些变量可能有关联。
8. 每个答案至少引用一个统计量或一张图，并写出一项限制。

在项目根目录一次生成全部产物，再用命令检查：

```powershell
python .\analysis.py
Get-ChildItem .\output
Get-Content .\output\summary.csv
```

输出目录中应同时出现四张 PNG 图和 `summary.csv`。图表本身可在 VS Code 文件区单击打开；教程不提供固定答案图，因为你需要根据自己的运行结果判断坐标、标题、图例和数据是否一致。

### 报告模板

```markdown
# 学习行为探索报告

## 数据概况
- 行数：
- 列数：
- 缺失与重复情况：

## 主要问题与发现
### 问题一：各组数量是否均衡？
- 证据：
- 结论：
- 限制：

## 数据质量提醒

## 下一步建议
```

### 验收标准

- 四幅图都有标题、轴标签，保存后可独立打开。
- 图表类型与问题匹配，没有使用三维图或无意义装饰。
- 报告中的数值能在程序输出或结果表中找到。
- 没有把相关写成因果，也没有把箱线图异常点直接当作错误。
- 全部分析可通过一次运行 `python analysis.py` 重新生成。

### 扩展作业

分别使用 5、10、20 个区间绘制成绩直方图，比较视觉结论是否变化，并解释为什么绘图参数也会影响观察。

## 11. 本章检查清单

- [ ] 我能区分数值数据和类别数据。
- [ ] 我能解释均值、中位数和标准差的直观含义。
- [ ] 我会根据问题选择四种常用图表。
- [ ] 我知道相关不代表因果。
- [ ] 我能用统计证据写出带有限制的结论。
