## 三种属性定义方式的区别

### 1️⃣ **类属性（在 [__init__](cci:1://file:///f:/Projects/llm/src/Transformer/vocab.py:22:4-28:34) 前定义）**

```python
class Vocabulary:
    # 第 18-21 行：类属性
    PAD_TOKEN = "<PAD>"
    UNK_TOKEN = "<UNK>"
    BOS_TOKEN = "<BOS>"
    EOS_TOKEN = "<EOS>"
```

**特点：**
- 属于**类本身**，不属于实例
- 所有实例**共享**同一个值
- 可以通过类名或实例访问
- 通常用于**常量**或**默认配置**

**示例：**
```python
# 通过类访问
print(Vocabulary.PAD_TOKEN)  # "<PAD>"

# 通过实例访问
vocab1 = Vocabulary()
vocab2 = Vocabulary()
print(vocab1.PAD_TOKEN)  # "<PAD>"
print(vocab2.PAD_TOKEN)  # "<PAD>"

# 所有实例共享同一个值
print(vocab1.PAD_TOKEN is vocab2.PAD_TOKEN)  # True
```

---

### 2️⃣ **实例属性（在 [__init__](cci:1://file:///f:/Projects/llm/src/Transformer/vocab.py:22:4-28:34) 中定义）**

```python
def __init__(self):
    # 第 25-26 行：实例属性
    self.token_to_id: Dict[str, int] = {}
    self.id_to_token: Dict[int, str] = {}
```

**特点：**
- 属于**每个实例**，每个对象有自己的副本
- 不同实例的值**相互独立**
- 只能通过实例访问
- 可以**修改**，每个实例可以有不同的值

**示例：**
```python
vocab1 = Vocabulary()
vocab2 = Vocabulary()

vocab1.add_token("hello")
vocab2.add_token("world")

print(vocab1.token_to_id)  # {'<PAD>': 0, ..., 'hello': 4}
print(vocab2.token_to_id)  # {'<PAD>': 0, ..., 'world': 4}

# 两个实例的字典是独立的
print(vocab1.token_to_id is vocab2.token_to_id)  # False
```

---

### 3️⃣ **属性装饰器 `@property`（动态计算的属性）**

```python
# 第 91-93 行：@property
@property
def pad_id(self) -> int:
    return self.token_to_id[self.PAD_TOKEN]
```

**特点：**
- 看起来像属性，实际上是**方法**
- 每次访问时**动态计算**
- **只读**（除非定义 setter）
- 可以添加逻辑、验证或计算

**示例：**
```python
vocab = Vocabulary()

# 像访问属性一样使用（不需要括号）
print(vocab.pad_id)  # 0

# 但实际上是调用方法，每次都会查询 token_to_id
# 等价于：vocab.token_to_id[vocab.PAD_TOKEN]

# 不能直接赋值（只读）
vocab.pad_id = 10  # ❌ AttributeError: can't set attribute
```

---

## 对比总结

| 特性 | 类属性 | 实例属性 | @property |
|------|--------|----------|-----------|
| **定义位置** | 类内，[__init__](cci:1://file:///f:/Projects/llm/src/Transformer/vocab.py:22:4-28:34) 外 | [__init__](cci:1://file:///f:/Projects/llm/src/Transformer/vocab.py:22:4-28:34) 内 | 方法上加装饰器 |
| **所属** | 类本身 | 每个实例 | 每个实例 |
| **共享性** | 所有实例共享 | 每个实例独立 | 每个实例独立 |
| **访问方式** | `类名.属性` 或 `实例.属性` | `实例.属性` | `实例.属性` |
| **可修改性** | 可修改（但会影响所有实例） | 可修改 | 只读（除非定义 setter） |
| **计算时机** | 类加载时 | 实例创建时 | 每次访问时 |
| **典型用途** | 常量、默认值 | 实例状态 | 动态计算、封装逻辑 |

---

## 在这个代码中的应用场景

### 为什么 `PAD_TOKEN` 是类属性？

```python
PAD_TOKEN = "<PAD>"  # 类属性
```
- 因为 `"<PAD>"` 是一个**常量**，所有 [Vocabulary](cci:2://file:///f:/Projects/llm/src/Transformer/vocab.py:13:0-124:20) 实例都应该使用相同的值
- 不需要每个实例都存储一份副本

### 为什么 `token_to_id` 是实例属性？

```python
self.token_to_id: Dict[str, int] = {}  # 实例属性
```
- 因为每个词表实例的内容**不同**
- 需要独立存储和修改

### 为什么 [pad_id](cci:1://file:///f:/Projects/llm/src/Transformer/vocab.py:90:4-92:47) 使用 `@property`？

```python
@property
def pad_id(self) -> int:
    return self.token_to_id[self.PAD_TOKEN]
```
- 提供**便捷的访问方式**：`vocab.pad_id` 而不是 `vocab.token_to_id[vocab.PAD_TOKEN]`
- **动态获取**：如果 `token_to_id` 被重新加载，[pad_id](cci:1://file:///f:/Projects/llm/src/Transformer/vocab.py:90:4-92:47) 会自动更新
- **只读保护**：防止意外修改这个重要的 ID

---

## 实际使用对比

```python
vocab = Vocabulary()

# 类属性：常量
print(vocab.PAD_TOKEN)  # "<PAD>"

# 实例属性：可变状态
vocab.add_token("hello")
print(vocab.token_to_id)  # {..., 'hello': 4}

# @property：便捷访问
print(vocab.pad_id)  # 0（等同于 vocab.token_to_id["<PAD>"]）
```

希望这个解释清楚了！如果还有疑问，欢迎继续提问 😊