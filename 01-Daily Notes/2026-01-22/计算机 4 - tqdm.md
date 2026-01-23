`tqdm` 是 Python 里最常用的**进度条库**，特别适合你现在这种经常写训练循环 / 跑数据处理代码的场景（和你最近在学 PyTorch、写训练流程很搭）。

我按「最常用 → 进阶 → 在 PyTorch 里用」给你一套实用速查。

---

## 一、安装

```bash
pip install tqdm
```

---

## 二、最基础用法（包一层循环就行）

### 1️⃣ 普通 for 循环

```python
from tqdm import tqdm
import time

for i in tqdm(range(100)):
    time.sleep(0.05)
```

效果：终端会显示一个实时进度条 + 百分比 + 速度 + 剩余时间。

---

### 2️⃣ 给进度条加说明文字

```python
for i in tqdm(range(100), desc="Processing"):
    ...
```

---

## 三、手动更新进度（不在 for 里）

适合你自己控制循环次数的时候：

```python
from tqdm import tqdm
import time

pbar = tqdm(total=100)

for i in range(100):
    time.sleep(0.05)
    pbar.update(1)

pbar.close()
```

---

## 四、在 PyTorch 训练中最常见的用法 ⭐（强烈推荐）

这是标准训练循环写法：

```python
from tqdm import tqdm

for epoch in range(num_epochs):
    pbar = tqdm(train_loader, desc=f"Epoch {epoch+1}")

    for batch in pbar:
        x, y = batch
        loss = model(x)

        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        # 动态显示 loss
        pbar.set_postfix(loss=loss.item())
```

这样你会看到：

- 当前 epoch
- 进度百分比 / 剩余时间
- 右侧实时显示 `loss=...`

这在训练时非常有用。

---

## 五、显示多个指标（loss / acc 等）

```python
pbar.set_postfix({
    "loss": f"{loss.item():.4f}",
    "lr": optimizer.param_groups[0]["lr"]
})
```

---

## 六、嵌套进度条（epoch + batch 两层）

```python
from tqdm import trange, tqdm

for epoch in trange(10, desc="Epoch"):
    for batch in tqdm(train_loader, desc="Batch", leave=False):
        ...
```

`leave=False` 可以防止刷屏。

---

## 七、在 Jupyter Notebook 里用（很重要）

在 notebook 里要用这个版本，不然显示会怪：

```python
from tqdm.notebook import tqdm
```

或者通用写法：

```python
from tqdm.auto import tqdm
```

👉 `tqdm.auto` 会自动判断是终端还是 notebook，最推荐。

---

## 八、常用参数速查表

```python
tqdm(iterable,
     desc="说明文字",
     total=总数,
     leave=True,        # 是否保留进度条
     ncols=100,        # 宽度
     mininterval=0.1)  # 最小刷新间隔
```

---

## 九、一个你现在阶段最实用的完整训练模板（结合你最近在学的 PyTorch）

```python
from tqdm.auto import tqdm

for epoch in range(num_epochs):
    model.train()
    pbar = tqdm(train_loader, desc=f"Epoch {epoch+1}/{num_epochs}")

    for x, y in pbar:
        x, y = x.to(device), y.to(device)

        out = model(x)
        loss = criterion(out, y)

        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        pbar.set_postfix(loss=f"{loss.item():.4f}")
```

---

如果你愿意，我可以：

- 帮你**直接改你现在的训练代码**加上 tqdm
- 或给你一份「PyTorch 标准训练 + 验证 + tqdm 显示 loss/acc」的完整模板

你现在是主要在终端跑，还是在 Jupyter / notebook 里跑训练？