**不需要多张显卡！** 

```
num_workers
```

 是 **CPU 进程**，不是 GPU。让我详细解释：

---

## num_workers：多进程数据加载

### 工作原理

```
num_workers=0（默认）:
  主进程: [加载数据] → [等待] → [GPU训练] → [加载数据] → [等待] → [GPU训练]
                                    GPU空闲 ↑
                                    
num_workers=4:
  子进程1: [加载batch_1] [加载batch_5] ...
  子进程2: [加载batch_2] [加载batch_6] ...
  子进程3: [加载batch_3] [加载batch_7] ...
  子进程4: [加载batch_4] [加载batch_8] ...
  主进程:  [GPU训练batch_1] [GPU训练batch_2] [GPU训练batch_3] ...
           ↑ 数据已经准备好了，GPU 不用等待！
```

### 关键点

- 使用的是 **CPU 进程**，不是 GPU
- 提前加载下一个 batch，GPU 不用等待数据
- 单卡也能用！

---

## pin_memory：锁页内存

### 工作原理

```
pin_memory=False:
  CPU内存 → [复制到可分页内存] → [复制到GPU]

                    ↑ 多一步，慢

pin_memory=True:
  CPU内存（锁页）→ [直接复制到GPU]

                    ↑ DMA传输，快！
```

### 关键点

- 数据在 **锁定的 CPU 内存** 中，不会被操作系统移动
- GPU 可以直接通过 DMA 读取，速度更快
- 会占用一点额外的 CPU 内存

---

## 使用建议

```python

data_loader = DataLoader(
    dataset,
    batch_size=256,
    collate_fn=collate_fn,
    num_workers=4,      # 一般设为 CPU 核心数的一半
    pin_memory=True     # 有 GPU 就开启
)
```

### num_workers 设多少？

|CPU 核心数|建议 num_workers|
|---|---|
|4 核|2|
|8 核|4|
|16 核|4-8|
|32 核|8-16|

**太大反而会慢**（进程切换开销）

---

## Windows 注意事项 ⚠️

Windows 上 

```
num_workers > 0
```

 需要把训练代码放在 

```
if __name__ == "__main__":
```

 里：

python

if __name__ == "__main__":

    trainTransformer(...)

否则会报错。

---

## 总结

|参数|需要多卡？|作用|
|---|---|---|
|```<br>num_workers<br>```|❌ 不需要|多个 CPU 进程提前加载数据|
|```<br>pin_memory<br>```|❌ 不需要|加速 CPU→GPU 数据传输|