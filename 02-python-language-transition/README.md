# 序章二：已有编程基础者的快速语言迁移

> 本章目标：把你在 C、C++ 或 Java 中已经掌握的基本编程概念迁移到 Python，能够看懂并编写后续章节需要的简单程序。

## 1. 本章适合谁

本章不是完整的 Python 教程，也不能代替系统的语言学习。它适合已经理解变量、条件、循环和函数，至少学过一门 C、C++ 或 Java 的学习者。

如果你还不能独立写出“读入若干数字并求平均值”的程序，建议先系统学习一门编程语言。如果你已经理解程序的基本结构，只是不熟悉 Python 写法，本章足以帮助你开始后续实践。

本教程统一使用 VS Code 编写和运行 `.py` 文件。

## 2. 创建第一个 Python 文件

在 VS Code 中新建文件夹 `python-transfer`，再新建 `hello.py`：

```python
name = input("请输入你的名字：")
print(f"你好，{name}！")
```

在 VS Code 终端中运行：

```powershell
python hello.py
```

如果 `python` 命令不可用，先完成下一章的环境配置。不要为了消除红色波浪线而随意安装多个 Python。

（图片 019：VS Code 打开 hello.py，并标出编辑区、文件区和集成终端）

## 3. 最重要的语法差异

Python 通常不写分号，也不用大括号表示代码块，而是使用缩进。推荐每层缩进 4 个空格。

```python
score = 85

if score >= 60:
    print("通过")
else:
    print("需要继续练习")
```

冒号和缩进都是语法的一部分。下面的写法会出错：

```python
if score >= 60:
print("通过")  # 缺少缩进
```

## 4. 变量与常用数据类型

Python 不要求在变量名前写类型：

```python
age = 18            # int，整数
height = 1.72       # float，小数
name = "小明"       # str，字符串
is_student = True   # bool，布尔值
```

可以使用 `type()` 查看类型：

```python
print(type(age))
```

Python 是动态类型语言，但这不表示类型不重要。例如字符串不能直接和整数相加：

```python
year_text = "2026"
next_year = int(year_text) + 1
print(next_year)
```

常见转换函数包括 `int()`、`float()`、`str()` 和 `bool()`。

## 5. 输入与输出

`input()` 得到的结果永远是字符串，需要计算时要先转换：

```python
price = float(input("请输入单价："))
count = int(input("请输入数量："))
total = price * count
print(f"总价为：{total:.2f} 元")
```

`f"..."` 是格式化字符串。`{total:.2f}` 表示把数字显示为两位小数。

## 6. 条件判断

| 含义 | C/C++/Java | Python |
| --- | --- | --- |
| 相等 | `a == b` | `a == b` |
| 不等 | `a != b` | `a != b` |
| 与 | `&&` | `and` |
| 或 | `\|\|` | `or` |
| 非 | `!` | `not` |

Python 使用 `if`、`elif`、`else`：

```python
temperature = 27

if temperature >= 35:
    level = "高温"
elif temperature >= 25:
    level = "温暖"
else:
    level = "凉爽"

print(level)
```

## 7. 循环

### 7.1 `for` 循环

C/C++ 中常见的 `for (int i = 0; i < 5; i++)`，在 Python 中写为：

```python
for i in range(5):
    print(i)
```

输出是 `0` 到 `4`。`range(开始, 结束, 步长)` 不包含结束值：

```python
for number in range(2, 11, 2):
    print(number)
```

遍历容器时通常不需要下标：

```python
scores = [82, 91, 76]
for score in scores:
    print(score)
```

同时需要下标和值时使用 `enumerate()`：

```python
for index, score in enumerate(scores):
    print(index, score)
```

### 7.2 `while` 循环

```python
count = 3
while count > 0:
    print(count)
    count -= 1
```

Python 没有 `count++`。循环中忘记修改条件可能造成无限循环。

## 8. 列表、元组、集合和字典

### 8.1 列表 `list`

列表有顺序、可以修改，是后续最常见的容器之一：

```python
scores = [78, 85, 92]
scores.append(88)
print(scores[0])      # 第一个元素
print(scores[-1])     # 最后一个元素
print(scores[1:3])    # 下标 1 到 2
```

Python 下标从 0 开始。切片左边包含、右边不包含。

### 8.2 元组 `tuple`

元组有顺序但通常不修改：

```python
point = (120.5, 30.2)
x, y = point
```

### 8.3 集合 `set`

集合中的值不重复，适合去重和成员判断：

```python
categories = {"A", "B", "A"}
print(categories)  # 只保留一个 A
```

### 8.4 字典 `dict`

字典用键查找值，类似一条有字段名的记录：

```python
student = {
    "name": "小明",
    "score": 88,
    "passed": True,
}

print(student["name"])
student["score"] = 90
```

遍历字典：

```python
for key, value in student.items():
    print(key, value)
```

## 9. 字符串

```python
text = "  Python Data  "

print(text.strip())             # 去除两端空白
print(text.lower())             # 转为小写
print(text.replace("Data", "Practice"))
print("Python" in text)         # 判断是否包含
```

字符串也可以使用下标和切片，但不能直接修改其中某个字符。需要修改时，应生成新字符串。

## 10. 函数

C/C++：

```cpp
double average(double a, double b) {
    return (a + b) / 2;
}
```

Python：

```python
def average(a: float, b: float) -> float:
    return (a + b) / 2
```

类型标注便于阅读和检查，但 Python 运行时通常不会自动阻止错误类型。函数应该只负责一个清楚的任务：

```python
def is_passed(score: float, pass_line: float = 60) -> bool:
    return score >= pass_line
```

## 11. 文件读写

推荐使用 `with`，文件使用完后会自动关闭：

```python
with open("message.txt", "w", encoding="utf-8") as file:
    file.write("第一行\n")
    file.write("第二行\n")

with open("message.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

相对路径以“运行程序时终端所在的目录”为基础。如果提示找不到文件，先用终端的 `pwd` 检查当前位置，再检查文件名和目录。

## 12. 模块与库

Python 自带标准库，也可以安装第三方库：

```python
import math
from pathlib import Path

print(math.sqrt(16))
print(Path.cwd())
```

不要把自己的文件命名为 `pandas.py`、`numpy.py`、`random.py` 等库名，否则可能遮住真正的库。

## 13. 快速阅读陌生 Python 代码

按照下面的顺序阅读，不要一开始就逐字研究：

1. 找 `import`，判断代码依赖哪些库。
2. 找输入：数据来自键盘、文件，还是函数参数。
3. 找输出：打印、返回、保存了什么。
4. 找主流程中的条件、循环和函数调用。
5. 再进入关键函数，确认每一步如何改变数据。
6. 用一份很小的输入手工推演。
7. 实际运行，并比较结果是否与推演一致。

常见入口写法：

```python
def main() -> None:
    print("程序从这里开始")


if __name__ == "__main__":
    main()
```

初学阶段先知道：直接运行这个文件时会调用 `main()`；被其他文件导入时不会自动调用。

## 14. AI Coding 场景下的基本检查

AI 生成的代码只能当作候选答案，不能因为“看起来完整”就直接相信。至少完成以下检查：

- 逐行读懂变量、输入和输出，不提交自己无法解释的代码。
- 检查库名、函数名和参数是否真实存在。
- 检查文件路径是否适合自己的电脑，避免写死别人电脑的绝对路径。
- 使用一组正常输入、一组边界输入和一组错误输入测试。
- 检查是否偷偷读取或上传了不相关文件。
- 不把 API Key、密码、Token 和隐私数据发给未知服务。
- 错误发生时保留完整报错，从最下面的异常类型开始阅读。

## 15. 什么时候快速迁移足够

快速迁移通常足以完成：短小数据处理脚本、读写 CSV、调用成熟库、修改已有示例。遇到以下情况，应系统学习 Python：

- 程序规模开始增大，需要组织多个模块。
- 经常看不懂错误信息、作用域、对象或迭代器。
- 需要编写可长期维护的软件。
- 需要并发、网络、性能优化或较复杂的面向对象设计。

## 16. 实验一：同一问题的语言对照

### 问题

给定一组分数，计算平均分，统计及格人数，并找出最高分。先读懂 C++ 版本，再独立补全 Python 版本。

### C++ 参考实现

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<double> scores = {78, 92, 55, 88, 61};
    double sum = 0;
    double maximum = scores[0];
    int passed = 0;

    for (double score : scores) {
        sum += score;
        if (score >= 60) passed++;
        if (score > maximum) maximum = score;
    }

    cout << "average=" << sum / scores.size() << endl;
    cout << "passed=" << passed << endl;
    cout << "maximum=" << maximum << endl;
    return 0;
}
```

### Python 待完成版本

```python
scores = [78, 92, 55, 88, 61]

# TODO 1：计算总分
# TODO 2：统计大于等于 60 分的人数
# TODO 3：找出最高分
# TODO 4：打印平均分、及格人数和最高分
```

要求先使用循环实现，再尝试使用 `sum()`、`len()` 和 `max()` 简化。两种写法的结果必须一致。

## 17. 实验二：短小迁移练习

每题创建一个单独的 `.py` 文件，并至少测试两组输入。

1. 把 C/C++ 中判断奇偶数的程序改写成 Python。
2. 使用 `range()` 输出 1 到 100 中所有能被 3 整除的数。
3. 把字符串列表 `[' Alice ', 'BOB', '  Carol']` 清理为统一的小写姓名。
4. 使用字典保存三种商品的单价，根据商品名和数量计算总价。
5. 编写 `count_passed(scores)` 函数，返回及格人数；空列表应返回 0。

验收时需要说明：程序输入是什么、输出是什么、测试了哪些边界情况。

## 18. 实验三：阅读并解释代码

不要先运行，先逐行写下注释并预测输出：

```python
records = [
    {"name": "A", "value": 12},
    {"name": "B", "value": 7},
    {"name": "C", "value": 15},
]

selected = []
for record in records:
    if record["value"] >= 10:
        selected.append(record["name"])

print(selected)
```

回答以下问题：

- `records` 的类型是什么？其中每个元素是什么类型？
- 条件判断检查了什么？
- `append()` 改变了哪个变量？
- 如果把 `>= 10` 改为 `< 10`，输出会怎样变化？

最后运行代码，核对自己的预测。

## 19. 最终实验：命令行成绩小程序

### 背景与目标

为一个学习小组编写成绩汇总程序。程序读取多名学习者的姓名和分数，输出人数、平均分、最高分、最低分与及格名单，并把结果保存到文本文件。

### 必须完成

1. 使用列表和字典保存记录。
2. 把“计算统计量”和“生成报告”分别写成函数。
3. 至少正确处理 5 条记录。
4. 分数只能在 0 到 100 之间；错误数据应给出提示。
5. 结果保存为 `result.txt`，编码使用 UTF-8。
6. 在终端运行成功，并能解释主要代码。

### 建议开发顺序

先用固定数据完成统计，再增加键盘输入，最后增加数据校验与文件保存。每完成一步就运行一次，避免写完全部代码后才发现错误。

### 验收示例

```text
人数：5
平均分：76.80
最高分：95
最低分：52
及格名单：小明、小红、小刚、小林
```

### 扩展作业

- 按分数从高到低输出排名。
- 从文本文件读取数据，而不是写死在代码中。
- 为关键函数增加类型标注和简短说明。

## 20. 本章检查清单

- [ ] 我理解 Python 使用缩进表示代码块。
- [ ] 我会使用条件、循环、列表、字典和函数。
- [ ] 我会读写简单文本文件。
- [ ] 我能有步骤地检查 AI 生成的代码。
- [ ] 我独立完成了成绩小程序。
