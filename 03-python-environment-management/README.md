# 序章三：Anaconda 与 Python 环境管理

> 本章目标：在 Windows 上建立一个独立、可复现的 Python 环境，使用 VS Code 正确运行 `.py` 文件，并能判断“包装到了错误环境”这类常见问题。

## 1. 为什么需要独立环境

不同项目可能需要不同版本的 Python 和第三方库。例如，项目甲需要某个库的旧版本，项目乙需要新版本。如果所有项目共用同一套环境，一次升级就可能让另一个项目无法运行。

Conda 环境可以理解为一个独立工具箱：每个工具箱有自己的 Python 和库。后续每个项目都应使用单独环境，不要把所有包装进 `base` 环境。

三个概念要分清：

- **Python 解释器**：真正执行 `.py` 文件的程序。
- **环境**：某个解释器及其已经安装的库的集合。
- **VS Code**：编辑器；它必须选择正确解释器，才能使用正确环境。

## 2. 安装 Anaconda

1. 打开 Anaconda 官方下载页面，选择 Windows 版本。
2. 下载 64 位图形安装程序。
3. 运行安装程序，普通个人电脑通常选择仅为当前用户安装。
4. 安装位置尽量使用英文路径，不要随意改动高级选项。
5. 完成安装后，在开始菜单打开 **Anaconda Prompt**。

在 Anaconda Prompt 中检查：

```powershell
conda --version
python --version
where.exe conda
```

前两条命令应显示版本号，`where.exe conda` 应显示 Conda 的安装位置。若命令找不到，先关闭并重新打开 Anaconda Prompt；仍然无效时，再检查安装是否完成，不要立即重复安装多套 Python。

## 3. 创建环境

为本教程创建名为 `data-practice` 的环境，并指定 Python 3.11：

```powershell
conda create -n data-practice python=3.11
```

出现确认提示时输入 `y`。环境名建议使用小写英文、数字和连字符，不要包含空格。

激活环境：

```powershell
conda activate data-practice
```

成功后，终端提示符前会出现 `(data-practice)`。

退出当前环境：

```powershell
conda deactivate
```

查看所有环境：

```powershell
conda env list
```

当前环境所在行通常带有 `*`。

## 4. 安装第三方库

先激活环境，再安装本教程使用的基础库：

```powershell
conda activate data-practice
conda install numpy pandas matplotlib scikit-learn
```

Conda 和 Pip 都能安装包，但初学阶段遵循这套规则：

1. 先激活目标环境。
2. 能用 Conda 安装时优先使用 Conda。
3. Conda 中没有所需包时，再在同一环境中使用 `python -m pip install 包名`。
4. 不要在一次安装失败后反复混用多个环境和多个命令。

使用下面的形式比直接写 `pip` 更容易确认安装目标：

```powershell
python -m pip install package-name
```

查看 Conda 已安装内容：

```powershell
conda list
```

查看当前 Python 对应的 Pip 包：

```powershell
python -m pip list
```

## 5. 在 VS Code 中选择正确解释器

1. 安装 VS Code。
2. 在 VS Code 终端安装 Microsoft 发布的 **Python** 扩展：

   ```powershell
   code --install-extension ms-python.python
   ```

   如果系统提示找不到 `code` 命令，再打开左侧“扩展”，搜索 `Python`，确认发布者为 Microsoft 后安装。

3. 使用“文件 → 打开文件夹”打开项目目录。
4. 按 `Ctrl+Shift+P` 打开命令面板。
5. 搜索并运行 `Python: Select Interpreter`。
6. 选择名称中包含 `data-practice` 的解释器。
7. 新建终端。终端前方应出现环境名。

选择后不要只凭界面判断。先在 VS Code 终端执行：

```powershell
conda env list
python -c "import sys; print(sys.executable)"
python -m pip --version
```

`conda env list` 中带 `*` 的应是 `data-practice`。后两条命令显示的 Python 和 Pip 路径也都应位于这个环境中。

在项目中创建 `check_env.py`：

```python
import sys

import numpy
import pandas
import sklearn

print("Python 路径：", sys.executable)
print("Python 版本：", sys.version)
print("NumPy 版本：", numpy.__version__)
print("Pandas 版本：", pandas.__version__)
print("Scikit-learn 版本：", sklearn.__version__)
```

在 VS Code 终端运行：

```powershell
python check_env.py
```

`sys.executable` 应指向 `data-practice` 环境，而不是其他 Python。

## 6. `.py` 文件的推荐运行方式

本教程的主线全部使用 `.py` 文件：

1. 在 VS Code 中打开整个项目文件夹，而不是只打开单个文件。
2. 确认右下角解释器属于当前环境。
3. 打开“终端 → 新建终端”。
4. 使用 `python 文件名.py` 运行。
5. 报错时保留完整信息，从最后一行开始阅读。

使用集成终端的好处是：能看见执行的命令、当前目录和完整输出，也更接近实际项目工作方式。

## 7. 常见的“包装错环境”问题

典型现象是：刚刚安装了 Pandas，运行代码却仍然提示：

```text
ModuleNotFoundError: No module named 'pandas'
```

按以下顺序检查：

```powershell
conda env list
python -c "import sys; print(sys.executable)"
python -m pip show pandas
```

然后检查 VS Code 右下角的解释器。常见原因包括：

- 安装包时没有激活目标环境。
- VS Code 选择了另一套 Python。
- 旧终端在切换解释器前就已打开。
- 使用了一个 Python 的 `pip`，却使用另一个 Python 运行。

修复时不要先卸载所有软件。先确认当前解释器路径，再把包装到这个解释器对应的环境中。

修复后重新打开 VS Code 终端，再次执行以下命令验证：

```powershell
python -c "import sys; print(sys.executable)"
python -m pip show pandas
```

第一条确认正在运行哪一个 Python，第二条的 `Location` 确认 Pandas 安装到了哪里。两处都指向 `data-practice` 才算真正解决。

## 8. 导出和恢复环境

### 8.1 导出完整 Conda 环境

```powershell
conda activate data-practice
conda env export --from-history > environment.yml
```

`--from-history` 主要记录主动安装的包，通常比完整导出更适合作为学习项目配置。

根据文件创建环境：

```powershell
conda env create -f environment.yml
```

环境文件示例：

```yaml
name: data-practice
dependencies:
  - python=3.11
  - numpy
  - pandas
  - matplotlib
  - scikit-learn
```

### 8.2 导出 Pip 依赖

如果项目主要使用 Pip，可以执行：

```powershell
python -m pip freeze > requirements.txt
python -m pip install -r requirements.txt
```

初学阶段一个项目保留一种主要环境说明即可，不必同时维护多份互相矛盾的配置。

## 9. 删除环境

先退出要删除的环境：

```powershell
conda deactivate
conda remove -n environment-name --all
```

删除是不可逆操作。执行前用 `conda env list` 再次核对环境名，绝对不要删除仍在使用的项目环境。

## 10. Jupyter 的定位：可选扩展作业

Jupyter Notebook 会把说明、代码和结果放在分块页面中，常用于探索和演示。但它也容易让初学者忽略代码执行顺序、工作目录和可复现性。因此本教程不把 Notebook 作为主开发方式，所有正式练习都使用 VS Code 中的 `.py` 文件。

如果已经完成主实验，可以把 Jupyter 作为新增作业：

```powershell
conda activate data-practice
conda install jupyter ipykernel
python -m ipykernel install --user --name data-practice --display-name "Python (data-practice)"
jupyter kernelspec list
jupyter notebook
```

`jupyter kernelspec list` 的结果中应出现 `data-practice`。打开页面后要选择 `Python (data-practice)` 内核。Notebook 能运行不代表 VS Code 已选择同一环境，两者需要分别检查。

## 11. 主实验：建立并验证独立环境

### 实验背景

你将为后续数据练习创建一套独立环境，并用一个小程序证明 VS Code、Python 和第三方库确实连接正确。

### 实验目标

- 创建、激活和退出 Conda 环境。
- 正确安装指定库。
- 在 VS Code 中选择相同环境的解释器。
- 导出可供别人重建的环境文件。

### 实验步骤

1. 创建 `data-practice` 环境，Python 版本指定为 3.11。
2. 激活环境，安装 NumPy、Pandas、Matplotlib 和 Scikit-learn。
3. 创建文件夹 `environment-lab`，在 VS Code 中打开整个文件夹。
4. 选择 `data-practice` 解释器，并新建终端。
5. 创建 `environment_test.py`：

```python
import sys

import numpy as np
import pandas as pd

numbers = np.array([2, 4, 6, 8])
table = pd.DataFrame({"number": numbers, "square": numbers ** 2})

print("解释器：", sys.executable)
print(table)
print("平均值：", numbers.mean())
```

6. 在终端运行 `python environment_test.py`。
7. 把输出和解释器路径记录到 `experiment_record.md`，但不要记录个人账号或敏感路径。
8. 导出 `environment.yml`。
9. 打开 `environment.yml`，确认包含环境名、Python 和所需库。

### 验收标准

- `conda env list` 中存在该环境。
- 程序正常输出表格与平均值 5.0。
- VS Code 选择的解释器与终端使用的解释器一致。
- `environment.yml` 能说明项目需要的环境。

### 恢复练习

不要删除正在使用的环境。将 `environment.yml` 中的 `name` 改为 `data-practice-copy`，再使用它创建副本。在副本中运行同一程序；验证成功后，可以删除副本。

### 可选新增作业：体验 Jupyter

完成主实验后，把 `environment_test.py` 的内容分成 3 个 Notebook 单元格运行，并回答：如果乱序运行单元格，会出现什么问题？为什么正式作业仍要求提交 `.py` 文件？

## 12. 本章检查清单

- [ ] 我能解释为什么每个项目应使用独立环境。
- [ ] 我会创建、查看、激活、退出和谨慎删除环境。
- [ ] 我能判断 VS Code 当前选择了哪个解释器。
- [ ] 我能定位“包装错环境”的原因。
- [ ] 我能导出并恢复环境。
