`matplotlib.pyplot` 是 Python 中一个用于创建图形和可视化数据的强大库 `matplotlib` 的子库。它提供了一个类似于 MATLAB 的绘图框架，可以方便地生成各种类型的图表。
# 结构
`matplotlib.pyplot` 包含许多函数，每个函数在当前的图形上对图表进行一些修改，如创建一个新图形、在图形上绘制线条、添加标签等。以下是一些常用的 `pyplot` 函数：
- **创建图形和子图**
  - `figure()`: 创建一个新的图形。
  - `subplot()`: 创建一个新的子图，或选择一个已有的子图。
- **绘图**
  - `plot()`: 绘制线图。
  - `scatter()`: 绘制散点图。
  - `bar()`: 绘制条形图。
  - `hist()`: 绘制直方图。
  - `pie()`: 绘制饼图。
- **设置图形属性**
  - `title()`: 设置图形标题。
  - `xlabel()`, `ylabel()`: 设置轴标签。
  - `legend()`: 添加图例。
  - `xlim()`, `ylim()`: 设置轴的范围。
- **显示和保存图形**
  - `show()`: 显示图形。
  - `savefig()`: 保存图形为文件。
## 用法
以下是一个使用 `matplotlib.pyplot` 创建简单线图的示例：
```python
import matplotlib.pyplot as plt
# 创建数据
x = [1, 2, 3, 4, 5]
y = [2, 3, 5, 7, 11]
# 创建一个新的图形
plt.figure()
# 绘制线图
plt.plot(x, y, label='Prime Numbers')
# 设置标题和标签
plt.title('Line Chart Example')
plt.xlabel('X Axis')
plt.ylabel('Y Axis')
# 添加图例
plt.legend()
# 显示图形
plt.show()
```
这个示例代码的具体步骤如下：
1. **导入库**：导入 `matplotlib.pyplot` 并使用别名 `plt`。
2. **创建数据**：定义要绘制的 x 和 y 数据。
3. **创建图形**：使用 `plt.figure()` 创建一个新的图形。
4. **绘制图表**：使用 `plt.plot()` 绘制线图，并为线条添加标签。
5. **设置属性**：使用 `plt.title()`、`plt.xlabel()` 和 `plt.ylabel()` 设置标题和轴标签。
6. **添加图例**：使用 `plt.legend()` 添加图例。
7. **显示图形**：使用 `plt.show()` 显示图形。
# 子图
## 使用 `subplot` 创建子图
#### 基本用法
`subplot(nrows, ncols, index)` 函数用于创建子图：
- `nrows` 是图形的行数。
- `ncols` 是图形的列数。
- `index` 指定当前子图的位置。
#### 示例
以下是一个包含两个子图的示例：
```python
import matplotlib.pyplot as plt
# 创建数据
x1 = [1, 2, 3, 4, 5]
y1 = [1, 4, 9, 16, 25]
x2 = [1, 2, 3, 4, 5]
y2 = [2, 3, 5, 7, 11]
# 创建一个包含2个子图的图形
plt.figure()
# 第一个子图
plt.subplot(2, 1, 1)  # 2行1列的子图中的第1个
plt.plot(x1, y1, label='Squared Numbers')
plt.title('Subplot 1')
plt.xlabel('X Axis')
plt.ylabel('Y Axis')
plt.legend()
# 第二个子图
plt.subplot(2, 1, 2)  # 2行1列的子图中的第2个
plt.plot(x2, y2, label='Prime Numbers')
plt.title('Subplot 2')
plt.xlabel('X Axis')
plt.ylabel('Y Axis')
plt.legend()
# 显示图形
plt.tight_layout()  # 调整子图之间的间距
plt.show()
```
## 使用 `subplots` 创建子图
`subplots` 函数是一个更方便的方法，可以一次创建一个包含多个子图的图形对象，并返回一个图形对象和一个子图对象的数组。
`axs` 是 `matplotlib.pyplot.subplots()` 函数返回的一个数组或二维数组，包含所有子图的轴对象（`Axes` 对象）。这些轴对象是我们用来在每个子图中进行绘制和设置属性的基础。
`Axes` 对象是 `Figure` 对象的子集。在 `matplotlib` 中，`Figure` 对象代表整个图形窗口或整个绘图区域，而 `Axes` 对象代表图形中的一个单独的子图或绘图区域。
当我们调用 `subplots()` 函数时，它会返回两个对象：
1. `fig`：图形对象（`Figure`），表示整个绘图区域。
2. `axs`：轴对象数组（`Axes`），每个元素代表一个子图。
### 使用 `subplots` 的示例

当我们创建单个子图时，`axs` 是一个单一的轴对象，而不是数组。
```python
import matplotlib.pyplot as plt

# 创建一个包含2行2列的子图
fig, axs = plt.subplots(2, 2)

# 第一个子图
axs[0, 0].plot([1, 2, 3], [1, 4, 9])
axs[0, 0].set_title('Subplot 1')
axs[0, 0].set_xlabel('X Axis')
axs[0, 0].set_ylabel('Y Axis')

# 第二个子图
axs[0, 1].plot([1, 2, 3], [2, 3, 5])
axs[0, 1].set_title('Subplot 2')
axs[0, 1].set_xlabel('X Axis')
axs[0, 1].set_ylabel('Y Axis')

# 第三个子图
axs[1, 0].plot([1, 2, 3], [2, 5, 10])
axs[1, 0].set_title('Subplot 3')
axs[1, 0].set_xlabel('X Axis')
axs[1, 0].set_ylabel('Y Axis')

# 第四个子图
axs[1, 1].plot([1, 2, 3], [3, 6, 9])
axs[1, 1].set_title('Subplot 4')
axs[1, 1].set_xlabel('X Axis')
axs[1, 1].set_ylabel('Y Axis')

# 调整子图之间的间距
fig.tight_layout()

# 显示图形
plt.show()

```