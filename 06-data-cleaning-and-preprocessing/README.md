# 第三章：数据清洗、预处理与简单特征构造

> 本章目标：发现常见数据问题，制定可解释、可重复的处理规则，并保证训练数据和未来数据使用相同的预处理方式。

## 1. 清洗不是“让表格看起来整齐”

真实数据可能包含缺失、重复、拼写不一致、错误类型和超出合理范围的值。清洗的目标是让数据能够被正确解释和处理，同时保留原始信息与操作记录。

基本原则：

- 不覆盖唯一的原始文件。
- 先检查，再决定如何处理。
- 每条规则都说明理由。
- 处理前后都检查行数、列数和关键统计量。
- 不根据验证结果反复“修饰”数据。

## 2. 建立数据质量报告

```python
print(df.shape)
print(df.head())
print(df.dtypes)
print(df.isna().sum())
print("重复行：", df.duplicated().sum())
print(df.describe(include="all"))
```

还要检查类别取值：

```python
for column in ["city", "membership"]:
    print(column)
    print(df[column].value_counts(dropna=False))
```

## 3. 缺失值

缺失可能表示“未填写”“不适用”“采集失败”，三者含义不同。

常见处理方式：

```python
# 删除关键字段缺失的记录
df = df.dropna(subset=["target"])

# 数值列用中位数填补
median_age = df["age"].median()
df["age"] = df["age"].fillna(median_age)

# 类别列填为明确的未知类别
df["city"] = df["city"].fillna("Unknown")
```

不能简单规定“所有缺失都填 0”。0 可能是真实值，填入后会改变分布。填补所用统计量应从训练数据计算，不能偷看验证数据或未来数据。

## 4. 重复数据

```python
duplicate_rows = df[df.duplicated(keep=False)]
print(duplicate_rows)
df = df.drop_duplicates()
```

完全相同的两行也不一定必然重复，例如两次合法交易可能恰好相同。应优先根据业务唯一编号判断：

```python
print(df[df.duplicated(subset=["record_id"], keep=False)])
```

如果同一编号内容冲突，需要制定保留规则或回到来源核实，不能随意保留第一条。

## 5. 修正数据类型

```python
df["age"] = pd.to_numeric(df["age"], errors="coerce")
df["date"] = pd.to_datetime(df["date"], errors="coerce")
df["is_member"] = df["is_member"].map({"yes": True, "no": False})
```

`errors="coerce"` 会把无法转换的内容变成缺失值。转换后必须再次统计缺失，确认哪些记录转换失败。

## 6. 字符串与类别统一

```python
df["city"] = df["city"].str.strip().str.lower()
df["city"] = df["city"].replace({
    "beijing": "北京",
    "bj": "北京",
})
```

只有确认不同写法表示同一类别时才能合并。不要因为拼写相似就擅自替换。

## 7. 异常数据

先使用规则检查不可能值：

```python
invalid_age = df[(df["age"] < 0) | (df["age"] > 120)]
invalid_rate = df[~df["completion_rate"].between(0, 1)]
```

再结合统计和图形观察极端值。处理选择包括核实并修正、设为缺失、限制到合理边界、删除记录或保留并增加标记。选择取决于字段含义，必须记录原因。

## 8. 类别编码与数值缩放

### 8.1 One-hot 编码

没有自然顺序的类别通常使用 one-hot 编码：

```python
encoded = pd.get_dummies(df, columns=["city"], dtype=int)
```

不要随意把城市映射为北京=1、上海=2、广州=3，这会制造并不存在的大小关系。

### 8.2 标准化

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
df[["age_scaled", "income_scaled"]] = scaler.fit_transform(df[["age", "income"]])
```

`fit` 学习均值和标准差，`transform` 使用已学参数转换数据。正式建模时只能在训练集上 `fit`，再对验证集和未来数据 `transform`。

## 9. 删除字段与构造特征

可删除明显无用、泄漏答案或只起标识作用的字段，但先保留原始副本：

```python
model_df = df.drop(columns=["record_id", "free_text_note"])
```

特征构造是根据已有字段生成可能更有用的新字段：

```python
df["total_spend"] = df["unit_price"] * df["quantity"]
df["days_since_join"] = (reference_date - df["join_date"]).dt.days
df["has_missing_contact"] = df["contact"].isna().astype(int)
```

新特征必须在预测时也能计算，且不能使用预测时才知道的答案。

## 10. 避免数据污染

数据污染或数据泄漏是指训练过程使用了本不应该知道的信息。常见错误：

- 先用全体数据计算均值填补，再划分训练和验证集。
- 根据最终标签决定怎样修正特征。
- 把答案列的变形、事后结果或人工总结放进特征。
- 训练数据和未来数据分别用不同规则编码。

安全顺序通常是：先划分训练与验证，再只用训练集学习填补、编码和缩放参数。Scikit-learn 的 `Pipeline` 会在后续章节帮助我们固定这个顺序。

## 11. 综合实验：清理活动报名脏数据

### 实验背景

活动报名表由多人手工录入，出现了空值、重复、空格、类别不一致和异常范围。你要输出干净数据与质量报告，不能修改原始文件。

### 原始数据

将下面内容保存为 `data/registrations_dirty.csv`：

```csv
record_id,name,city,age,hours,member,signup_date
R001, 小明 ,北京,19,4.5,yes,2026-08-01
R002,小红,Shanghai,20,6,no,2026-08-02
R003,小刚,BJ,,3.5,YES,2026-08-03
R004,小林, 上海 ,twenty-one,5.0,no,2026-08-04
R004,小林, 上海 ,twenty-one,5.0,no,2026-08-04
R005,小雨,广州,18,-2,yes,wrong-date
R006,小周,,150,7.5,,2026-08-06
R007,小夏,Guangzhou,22,8,No,2026-08-07
R008,小陈,北京,20,,yes,2026-08-08
```

### 清洗规则

1. 姓名去除两端空格，不改动姓名内容。
2. 城市去除空格，并把 `BJ` 统一为“北京”、`Shanghai` 统一为“上海”、`Guangzhou` 统一为“广州”；缺失填为 `Unknown`。
3. 年龄转换为数值。无法转换或不在 0 到 120 之间的值设为缺失，再使用有效年龄的中位数填补。
4. `hours` 转换为数值；小于 0 的值视为无效并设为缺失，使用有效值中位数填补。
5. `member` 忽略大小写，统一为布尔值；缺失填为 `False`。
6. 日期转换为日期类型，失败记录保留缺失，不凭空猜日期。
7. 根据 `record_id` 和其他字段完全一致的情况删除重复行。
8. 新增 `hours_level`：小于 4 为 `low`，4 到小于 7 为 `medium`，7 及以上为 `high`。

（图片 030：清洗前后数据质量统计对比，突出缺失数、重复数和类别取值变化）

### 必须产物

- `clean_data.py`：可重复运行的清洗程序。
- `output/registrations_clean.csv`：清洗结果。
- `output/quality_report.txt`：清洗前后行数、重复数、各列缺失数、年龄范围、工时范围和城市类别。
- `cleaning_notes.md`：逐条说明规则、理由和未解决问题。

### 验收标准

- 原始 CSV 保持不变。
- 每一条清洗规则都能在代码中找到。
- 转换失败没有被悄悄隐藏，报告中记录日期仍有缺失。
- 清洗后编号唯一，年龄与工时均在合理范围内。
- 再次运行得到相同结果。

### 思考与扩展

- 年龄用中位数填补会带来什么偏差？
- 错误日期为什么不应该直接填成今天？
- 把清洗逻辑拆成 `clean_city`、`clean_member` 等函数，并为每个函数准备 3 个测试输入。

## 12. 本章检查清单

- [ ] 我会识别缺失、重复、类型、类别和范围问题。
- [ ] 我能为处理规则说明理由并保留原始数据。
- [ ] 我理解类别编码和数值缩放的用途。
- [ ] 我知道预处理参数只能从训练数据学习。
- [ ] 我能输出清洗结果和质量报告。
