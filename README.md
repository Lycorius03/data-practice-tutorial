# Data Practice Tutorial

这是一套面向初学者的数据实践教程。课程从文件、路径、VS Code、Git 与 Python 环境讲起，逐步进入数据处理、探索、清洗、机器学习和完整项目实践。

教程正文使用中文，所有文件和目录使用英文名称，减少不同操作系统、终端和编程工具处理路径时产生的问题。主线练习统一使用 VS Code 和 `.py` 文件；Jupyter Notebook 只作为可选扩展作业。

## 从这里开始

第一次学习时，先阅读[完整课程大纲](tutorial-outline.md)，然后按照目录编号依次完成章节：

| 顺序 | 章节 | 内容 |
| --- | --- | --- |
| 01 | [Git and GitHub](01-git-and-github/README.md) | 文件基础、Git、GitHub、Branch、Fork、Pull Request 与 Pages |
| 02 | [Python Language Transition](02-python-language-transition/README.md) | 从 C、C++ 或 Java 快速迁移到 Python |
| 03 | [Python Environment Management](03-python-environment-management/README.md) | Anaconda、Conda 环境与 VS Code 解释器 |
| 04 | [NumPy and Pandas](04-numpy-and-pandas/README.md) | 数组、表格读取、筛选、分组和保存 |
| 05 | [Data Exploration and Visualization](05-data-exploration-and-visualization/README.md) | 基础统计、图表与探索报告 |
| 06 | [Data Cleaning and Preprocessing](06-data-cleaning-and-preprocessing/README.md) | 缺失、重复、类型、编码和特征构造 |
| 07 | [First Machine Learning Models](07-first-machine-learning-models/README.md) | 分类、回归与 Scikit-learn 基本流程 |
| 08 | [Model Evaluation](08-model-evaluation/README.md) | 指标、基线、交叉验证和实验比较 |
| 09 | [Complete Data Project](09-complete-data-project/README.md) | 独立完成一个可复现的数据项目 |

## 下载教程

在准备存放学习材料的父目录中打开 VS Code 终端，执行：

```powershell
git clone https://github.com/Lycorius03/data-practice-tutorial.git
cd data-practice-tutorial
```

`clone` 会创建 `data-practice-tutorial` 文件夹并下载教程。下载后，应在 VS Code 中打开这个新文件夹。

## 获取后续更新

进入已经下载的教程目录，再执行：

```powershell
git status
git pull
```

先用 `git status` 检查本地是否有修改。`git pull` 会获取 GitHub 上的新提交并合并到当前分支。如果你直接修改了教程原文件，且远程也修改了相同位置，可能产生冲突。因此个人实验应放在教程要求你另外创建的练习目录中。

## 学习约定

- 按编号学习，一个章节完成后再进入下一章。
- 命令在 VS Code 集成终端中执行，代码写入对应文件。
- 图片占位会写成“图片 + 三位数字 + 内容说明”，后续按编号补入截图。
- 不公开上传 API Key、Token、密码、私钥、签名文件或个人隐私数据。
- 遇到错误时保留完整报错，先检查文件名、扩展名、当前路径和所选环境。

序章一中的网页成品目录用于教师本地预览，已经通过 `.gitignore` 排除。学生仓库中只提供 `webpage-source.txt`，需要亲手创建 `index.html` 并粘贴源码。
