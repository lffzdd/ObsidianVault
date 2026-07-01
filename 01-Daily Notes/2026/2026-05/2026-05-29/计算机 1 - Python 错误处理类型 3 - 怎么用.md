这是一个非常经典且有深度的问题！

简短的答案是：**在局部的具体业务逻辑中，不应该用；但在系统的最外层（如全局守护、服务器入口），我们“必须”用它。**

用不用 `except Exception as e`，取决于你的**代码所处的层级**。

---

### 一、 为什么在“局部业务逻辑”中，不应该用它？

像我们刚才写的 `run_dfa` 或者解析一个 JSON 字符串，这种属于**具体的、局部的业务步骤**。在这类地方用 `except Exception` 会带来灾难：

#### 1. 它会“指鹿为马”，掩盖真正的 Bug
假设你有下面这段代码：
```python
def get_user_age(user_data):
    try:
        # 假设我们想把年龄转成整数
        return int(user_data['age'])
    except Exception as e:
        return 18  # 发生任何错误，都默认返回 18 岁
```
如果传入的数据是 `{'age': '20'}`，一切正常。
但如果某天你手抖，把代码写成了：
```python
        return int(user_data['agee'])  # ⚠️ 键名写错了！本该报 KeyError
```
因为你用了 `except Exception`，这个 `KeyError` 就会被默默抓住，然后函数开开心心地返回了 `18`。你的程序不会报错，但用户的数据全错了。你可能要花几天时间去排查为什么所有人的年龄都变成了 18 岁。

#### 正确做法：
只捕获你能预料并处理的 `ValueError`（针对 `int()` 转换失败）：
```python
    try:
        return int(user_data['age'])
    except ValueError:
        return 18  # 只有在值不对时才默认 18 岁。如果键写错了，让它直接崩溃，方便我们发现 Bug！
```

---

### 二、 在什么情况下，“必须”用 `except Exception as e`？

虽然在局部代码里要禁用它，但在**整个系统的最外层入口**，我们必须用它来做**“兜底”**（也叫全局异常处理）。

#### 场景 1：Web 服务器 / API 接口
如果你在写一个网站后台，你绝对不希望某个用户输入了一个奇怪的字符，导致**整个服务器直接崩溃关机**。
```python
# 网站的最外层控制器
try:
    response = handle_user_request(request)
except Exception as e:
    # 1. 悄悄把详细的报错信息记录到日志文件里，方便程序员排查
    logger.error(f"服务器内部错误: {e}", exc_info=True)
    # 2. 给用户返回一个友好的 500 页面，而不是让服务器崩溃
    return "服务器开小差了，请稍后再试", 500
```

#### 场景 2：后台常驻任务 / 线程
比如一个每隔 5 分钟抓取一次网页的爬虫。你不能因为某一次网络瞬间断开报错，就让整个爬虫程序彻底死掉，以后再也不运行了。
```python
while True:
    try:
        crawl_data()
    except Exception as e:
        print(f"本次抓取失败: {e}，5分钟后重试...")
    time.sleep(300)
```

---

### 三、 总结：程序员的黄金法则

你可以把这个规则记成两句话：

1. **写具体功能时（如解析、计算、读写）**：
   * **只捕获你了解的错误**（如 `ValueError`, `KeyError`, `FileNotFoundError`）。
   * 宁可让程序崩溃，也不要用 `except Exception` 掩盖 Bug。
2. **写系统框架、最外层守护时**：
   * **大胆使用 `except Exception as e`**，但一定要配合 `logger.error` 把错误记下来，否则你永远不知道系统里发生了什么。