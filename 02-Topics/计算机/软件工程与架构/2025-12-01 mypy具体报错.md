# 导入问题

>`base\config.py:1: error: Library stubs not installed for "yaml" [import-untyped]`

这句话不是在说你的 `yaml` 模块有问题，而是在说：

**mypy 找不到 `yaml` 这个库的类型信息（type stubs），因此它无法对这个库进行静态类型检查。**

它抱怨的是 _类型信息缺失_，不是 _代码错误_。

---

## 一、它字面上包含什么信息？

报错内容：

```
Library stubs not installed for "yaml"  [import-untyped]
```

翻译成“类型检查器的语言”就是：

> “我找不到你导入的 `yaml` 这个库的（.pyi）类型声明文件，所以我只能把它当成 untyped module。”

换句话说：

- 你导入了：`import yaml`
- mypy 去找 yaml 的类型信息（.pyi “stub 文件”）
- 找不到 → 退化成 `Any`
- → 产生 `import-untyped` 的警告

这是 mypy 非常典型的“缺少类型声明”提示。

## 二、为什么 mypy 需要“stubs”？

因为 mypy 并不会执行库代码，它需要靠“类型提示文件（type stubs）”来理解库的 API：

- `.pyi` 就像“头文件”
- 里面只写函数签名、类型、返回值
- 不包含实现

比如：

```python
# yaml.pyi
def load(stream: Any) -> dict[str, Any]: ...
```

mypy 通过这种文件才能知道 `yaml.load()` 返回什么。

如果没有这种文件，mypy 就只能说：

```
from yaml import load
# load: Any
```

全部变成 Any → 你失去类型检查的大部分意义。

## 三、PyYAML 默认没有自带类型声明

你用的 `yaml` 其实是 PyYAML：

```
pip install PyYAML
```

但 PyYAML 很老 → 历史原因 → 没有官方类型注解  
（直到近几年才有人开始补，但目前依旧不完整）

所以 mypy 才会说：

> 没类型，我没法检查。

## 四、怎么解决？

有三个办法：

### 方案 1：安装第三方提供的类型 stubs（推荐）

在 Typing 社区有专门的补充包：

```
pip install types-PyYAML
```

这是为 PyYAML 提供 `.pyi` 文件的补丁包。

安装后 mypy 就能找到类型信息，不会再报错。

### 方案 2：告诉 mypy：“这个模块我不检查，别抱怨”

在 `mypy.ini` / `pyproject.toml` 配置里加：

```toml
[mypy-yaml]
ignore_missing_imports = True
```

这样：

- 你继续用 YAML
- mypy 不再抱怨
- 但与 YAML 相关的类型检查会变弱（退化成 Any）

适用于“不在乎 YAML 的类型安全”的情况。

### 方案 3：自己写 stub 文件（高级玩法）

你可以创建：

```
your_project/
    yaml.pyi
```

里面写你要用的 API 的签名。  
工业项目中常这么做，但日常没必要。

## 五、底层设计动机：为什么 mypy 不直接分析库的 Python 代码？

因为：

1. 有些库是 **C 扩展（PyYAML 有）**，mypy 看不到实现
2. 动态语言的代码太灵活，静态分析很难完整推断
3. 类型检查要求确定性，而 Python 实现可能太动态
4. stub 文件是类型系统世界的“接口”，更稳定、更可控

所以 mypy 的理念是：

> “库实现是什么不重要，只要你告诉我它的‘类型形状’（API 类型）就行。”

这也是静态类型系统设计的一个核心原则：  
**用接口（signature）来约束，而不是分析运行时。**

## 六、总结一句话

报错的本质：

> **PyYAML 没有类型声明文件，所以 mypy 无法检查它的 API，你需要安装 types-PyYAML 或关闭此错误。**

---

如果你想，我还能继续从更底层的角度讲：

- 为什么 type stub 的设计来源于 C 语言的头文件机制
- stub 文件是如何与 mypy 的 typed AST 结合的
- 为什么 typed Python 要用“外部 stubs”，而不是像 TypeScript 那样把类型放 package 内
- mypy 如何区分 “typed module” vs “untyped module”
- TypedDict、Protocol、PEP 561 这些机制如何统一“第三方库的类型”

每一条都是类型系统设计史里非常有趣的演化支线。

# 