下面我用你喜欢的“底层逻辑 + 历史演化 + 核心思想”方式讲解 Python 的 `logging` 系统——重点不是“怎么写代码”，而是理解它为什么要这样设计，这样你就能真正掌握它，而不是死背 API。

结构是：

1. Python 为什么要设计 logging？（底层动机）
2. logging 的总体架构（和 log4j 同源的思想）
3. 最最核心的三个组件（Logger / Handler / Formatter）
4. 最推荐的使用方式（代码示例 + 背后的原理）
5. logging 配置（代码、dict、INI 格式为什么存在）

讲完你会发现：Python logging 的使用其实是 **“应用日志阶段”设计思想的完美复刻**。

---

# 一、Python 为什么要做 logging？

## 1. 历史背景：print 已经不够用

随着 Python 越来越用于服务器、脚本、爬虫、后台任务，单靠 `print()` 完全不能满足日志的需求：

- print 强耦合到 stdout
- 无法按级别过滤
- 无法动态控制
- 无法把日志发到文件、syslog、网络
- 多模块项目会互相抢输出

所有这些痛点，90 年代 Java 的 log4j 都已经解决过。
Python 的设计者（Guido）看得很清楚：

> “我们不能让每个项目手搓一个 log 系统，需要统一的标准库。”

于是 Python 2.3（2003 年）加入了 `logging`。

它的设计思路几乎完全继承 log4j 的理念。

---

# 二、logging 的总体架构

可以把 logging 看成一个“小型消息路由系统”：

```
Logger ——> Handler ——> Output(文件/终端/网络)
        (决定要不要输出)   (决定怎么格式化)
```

三大角色各司其职：

- **Logger**：决定“这条日志是否应该被记录”（按级别过滤）
- **Handler**：决定“要把日志写到哪里”
- **Formatter**：决定“日志长什么样”

另外还有 Configurator：配置整个体系。

这套结构就是 log4j 的 Python 版。

---

# 三、三个核心组件（理解了这个，logging 就彻底通了）

## 1. Logger：应用内部的“说话者”

核心作用：

- 按命名空间（模块名）组织日志
- 决定日志是否需要输出（过滤级别）
- 提供 API：`logger.info()`, `logger.error()`

为什么用名字？

因为这是 log4j 的思想：

> “按模块划分日志，构成层次结构。”

Python 约定俗成：

```python
logger = logging.getLogger(__name__)
```

`__name__` 就是当前模块名，因此日志会按模块结构分组。

Logger 有两个重要属性：

- **level（过滤级别）**
- **handlers（输出器链）**

---

## 2. Handler：决定日志去哪儿

常见 Handler：

- `StreamHandler`：输出到 stdout
- `FileHandler`：输出到文件
- `RotatingFileHandler` / `TimedRotatingFileHandler`
	（按大小/时间自动切分）
	
- `SocketHandler`：发到网络
- `SysLogHandler`：发到 syslog

同一个 logger 可以挂多个 handler：

```
logger → handler1 → 控制台
       → handler2 → 文件
       → handler3 → syslog
```

这让你能做到：

- 开发阶段：写到控制台
- 生产环境：写文件 + 发远程日志系统
- 而业务代码完全不需要修改

---

## 3. Formatter：日志长什么样

Formatter 决定最终日志文本：

```python
"%(asctime)s - %(name)s - %(levelname)s - %(message)s"
```

Formatter 的存在意义：

- 把“日志内容”与“日志表现形式”解耦
- 以后可以随时改格式、改为 JSON

你可以选择：

- 人类可读格式
- JSON 结构化格式（便于日志收集系统处理）

---

# 四、Python logging 最推荐的用法（核心示例）

这里给你一个“工程实践级”的例子：

## 主程序（myapp/main.py）

```python
import logging
import logging.config
import yaml

# 加载配置文件
with open("logging.yaml", "r") as f:
    config = yaml.safe_load(f)
    logging.config.dictConfig(config)

logger = logging.getLogger(__name__)

def main():
    logger.info("程序启动")
    logger.debug("调试信息")
    logger.error("发生错误", exc_info=True)

if __name__ == "__main__":
    main()
```

## logging.yaml（推荐保持独立）

```yaml
version: 1
formatters:
  standard:
    format: "%(asctime)s [%(levelname)s] %(name)s: %(message)s"

handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: standard

  file:
    class: logging.handlers.TimedRotatingFileHandler
    filename: myapp.log
    when: midnight
    backupCount: 7
    formatter: standard

loggers:
  myapp:
    level: DEBUG
    handlers: [console, file]
    propagate: no

root:
  level: INFO
  handlers: [console]
```

这个结构和 log4j/application logging 一模一样：

- 所有 logger 按模块命名
- 配置放外面（动态可改）
- 不同 handler 输出到不同地方
- 格式化器可随时换（JSON、行式）

---

# 五、那最简单的用法是什么？（但不推荐用于工程）

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logging.info("hello")
```

`basicConfig()` 是给脚本、小工具用的。
真正的项目会用 `dictConfig()` 或 YAML 配置。

---

# 六、总结：如何在你的脑中建立“logging 心智模型”

Python 的 logging 不是一堆 API，而是一套非常成熟的**应用日志架构**：

- **Logger**
	决定是否记录、按模块分层组织。
	
- **Handler**
	决定写到哪里，可以多个并存。
	
- **Formatter**
	决定“日志长相”，可切换 JSON 等格式。
	
- **Configuration**
	把日志策略放配置文件中，和业务代码解耦。

理解这点之后，所有使用方式都自然了：

- 你用 logger 打日志
- handler 决定后端
- formatter 决定格式
- 配置决定策略

这样你的日志系统就是“专业级的”，而不是“printf 的高级版”。

---

如果你愿意，我可以继续讲：

- logging 的继承规则
- propagate 的机制（为什么有些模块日志重复输出）
- 如何优雅地做 JSON 结构化日志
- 如何和 FastAPI、Flask、Django 集成

或者，我们也可以从“可观测性”的视角讲讲为什么 logging 需要加 trace_id 等字段。