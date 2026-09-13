# 第五章：结果评价与简单实验改进

> 本章目标：选择基础评价指标，识别明显过拟合与数据泄漏，并用固定、可复现的实验比较方案，而不是凭感觉判断。

## 1. 能运行不等于有效

程序成功输出预测，只能说明流程没有立即报错。模型是否有用，需要用没有参与训练的数据和适合任务的指标评价，还要与简单基线比较。

## 2. 分类指标：Accuracy

Accuracy 是预测正确的样本数占总样本数的比例：

```python
from sklearn.metrics import accuracy_score

score = accuracy_score(y_valid, predictions)
```

如果 100 个样本中 95 个属于 A 类，那么永远预测 A 也能得到 95% Accuracy。因此类别严重不均衡时，不能只看 Accuracy。初学阶段至少同时查看类别数量和混淆矩阵：

```python
from sklearn.metrics import confusion_matrix

print(y_valid.value_counts())
print(confusion_matrix(y_valid, predictions))
```

## 3. 回归指标：MAE 与 RMSE

MAE 是绝对误差的平均值，与目标单位相同，容易解释：

```python
from sklearn.metrics import mean_absolute_error

mae = mean_absolute_error(y_valid, predictions)
```

RMSE 对较大的误差更敏感：

```python
from sklearn.metrics import root_mean_squared_error

rmse = root_mean_squared_error(y_valid, predictions)
```

对 MAE 和 RMSE，通常越小越好，但必须结合目标范围和简单基线解释。不同数据集的分数不能直接比较。

## 4. 建立基线

分类基线可以总是预测训练集中最常见类别；回归基线可以总是预测训练目标均值：

```python
from sklearn.dummy import DummyClassifier, DummyRegressor

baseline = DummyClassifier(strategy="most_frequent")
# 或 DummyRegressor(strategy="mean")
```

如果复杂模型没有明显优于简单基线，就需要检查数据、特征、模型或评价方式。

## 5. 训练效果与验证效果

```python
train_score = model.score(X_train, y_train)
valid_score = model.score(X_valid, y_valid)
```

训练很好、验证明显较差，可能是过拟合：模型记住了训练样本中的细节，却不能推广到新数据。训练和验证都差，可能是特征不足、模型不合适或数据质量存在问题。

这只是直观判断，不能仅凭一次划分下结论。

## 6. 交叉验证

交叉验证把训练数据轮流分成若干份，多次训练和评分：

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, X, y, cv=5, scoring="accuracy")
print(scores)
print("平均分：", scores.mean())
print("标准差：", scores.std())
```

回归可使用 `scoring="neg_mean_absolute_error"`，结果是负数，需要取相反数再解释。交叉验证更稳定，但它不会自动修复泄漏，预处理必须放入 Pipeline。

## 7. 使用 Pipeline 避免预处理泄漏

```python
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

model = make_pipeline(
    SimpleImputer(strategy="median"),
    StandardScaler(),
    LogisticRegression(max_iter=500),
)
```

交叉验证时，每一折只会用该折训练部分学习填补和缩放参数。不要先对完整数据 `fit_transform`，再进行交叉验证。

## 8. 公平比较实验

一次只改一个主要因素，并保持以下内容相同：

- 相同原始数据和标签定义。
- 相同训练/验证划分或相同交叉验证折数。
- 相同评价指标。
- 相同随机种子。
- 相同的数据清洗边界。

每次记录假设、改动、分数、运行时间和结论。只报告最好分数会隐藏失败尝试，也容易产生错误判断。

推荐把每次结果追加到同一个 CSV 文件，并在终端中查看：

```powershell
Get-Content .\output\experiment_results.csv
```

记录至少包含实验编号、唯一主要改动、验证结果和结论。若一行同时改了数据处理、特征和模型参数，即使分数提高，也无法判断究竟是哪项改动有效。

## 9. 常见数据泄漏

- 把标签或标签的直接变形放入特征。
- 使用预测发生之后才产生的字段。
- 用全体数据计算填补值、缩放参数或类别编码。
- 根据验证集不断选择特征，最后仍把同一验证集当作未知数据。
- 同一个人的多条高度相似记录同时出现在训练和验证中。

异常高的分数值得警惕。先检查泄漏，再庆祝模型优秀。

## 10. 综合实验：四次可复现实验

### 实验背景

继续使用第四章的鸢尾花分类任务。目标不是追求最高分，而是学习如何公平地判断修改是否有效。

### 固定设置

- 评价指标：5 折交叉验证 Accuracy。
- 交叉验证：`StratifiedKFold(n_splits=5, shuffle=True, random_state=42)`。
- 每个方案使用同一份数据和同一组折叠。
- 把所有结果保存到 `output/experiment_results.csv`。

### 四个实验

| 实验 | 唯一主要改动 | 要回答的问题 |
| --- | --- | --- |
| 1 | `DummyClassifier` 基线 | 正常模型是否超过最简单猜法？ |
| 2 | Logistic Regression，仅使用原始特征 | 基础模型表现如何？ |
| 3 | Pipeline 中增加 StandardScaler | 缩放是否带来稳定变化？ |
| 4 | 在实验 3 基础上修改 `C` 参数 | 参数变化是否真正改进？ |

每个实验记录：5 次分数、平均分、标准差、与基线的差值、改动和结论。

### 建议框架

```python
from sklearn.model_selection import StratifiedKFold, cross_val_score

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)


def evaluate(name, model, X, y):
    scores = cross_val_score(model, X, y, cv=cv, scoring="accuracy")
    return {
        "experiment": name,
        "fold_scores": ";".join(f"{score:.4f}" for score in scores),
        "mean_accuracy": scores.mean(),
        "std_accuracy": scores.std(),
    }
```

### 验收标准

- 四个实验均使用相同交叉验证对象和指标。
- 预处理包含在 Pipeline 内。
- 结果文件保留全部折分数，而不只保留最好分数。
- 结论考虑平均分和波动；极小差异不被夸大。
- 报告指出至少一种可能的数据泄漏，并说明本实验如何避免。

### 结果报告模板

| 实验 | 改动 | 平均 Accuracy | 标准差 | 相对基线 | 判断 |
| --- | --- | ---: | ---: | ---: | --- |
| 1 | 基线 |  |  | 0 |  |
| 2 | 基础模型 |  |  |  |  |
| 3 | 增加缩放 |  |  |  |  |
| 4 | 修改参数 |  |  |  |  |

最后回答：哪个修改有证据支持、哪个修改没有明显作用、你下一次只准备改变什么。

### 扩展作业

对第四章回归任务建立均值基线，用 5 折交叉验证比较 Linear Regression 的 MAE。注意 Scikit-learn 的负 MAE 评分方向，并在报告中转换为容易理解的正数误差。

## 11. 本章检查清单

- [ ] 我会为任务选择基础分类或回归指标。
- [ ] 我会建立简单基线。
- [ ] 我能直观识别过拟合迹象。
- [ ] 我会使用交叉验证和 Pipeline。
- [ ] 我能识别常见泄漏并公平记录实验。
