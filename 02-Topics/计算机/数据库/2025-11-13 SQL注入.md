# 什么是SQL注入

SQL 注入这个词听起来有点黑客味道，但它的本质其实非常朴素：**当你的程序把“用户输入”当成“SQL 代码的一部分”去执行时，就可能被人利用来执行你本不想执行的 SQL。**

为了讲清楚底层逻辑，我们从“为什么会出现”开始。

---

# 一、为什么会出现 SQL 注入？（历史与底层动机）

在互联网早期，很多程序是这样拼接 SQL 的：

```python
sql = "SELECT * FROM users WHERE name = '" + user_input + "'"
```

如果用户输入的是：

```
alice
```

那没有问题。
但如果用户输入：

```
' OR '1'='1
```

SQL 会变成：

```sql
SELECT * FROM users WHERE name = '' OR '1'='1'
```

`'1'='1'` 永远为真
→ 攻击者就能绕过登录验证、查到所有用户信息。

这就叫 **SQL 注入**。
根源很简单：
**程序把用户输入和 SQL 代码混在一起拼起来执行了。**

这就像你在命令行输入：

```
rm -rf /home/user/  # 删除用户文件
```

结果有人让你执行：

```
rm -rf /home/user/ ; rm -rf /
```

于是整台机器被删了。

---

# 二、SQL 注入实际能干什么？

攻击者可以通过“构造巧妙的输入字符串”改变 SQL 的逻辑，从而达到：

- 绕过登录
- 读取不该看的表
- 修改数据
- 删除表
- 建立自己的管理员账户
- 执行危险操作（如 `DROP TABLE users`）

强一点的攻击甚至能接管整个数据库。

---

# 三、为什么使用绑定变量（`:table_name`）能避免注入？

你的原代码：

```python
cursor.execute(query, table_name=table_name.upper())
```

相当于告诉数据库两件事：

1. SQL **模板** 是确定的：

```sql
SELECT COLUMN_NAME FROM ALL_TAB_COLUMNS WHERE TABLE_NAME = :table_name
```

1. `:table_name` 是 **数据**，不是SQL片段。
	数据库会将它当成“参数值”处理，而不是字符串拼接到 SQL 里。

于是，即便攻击者输入：

```
"xxx' OR '1'='1"
```

数据库看到的仍然是：

```
TABLE_NAME = 'xxx'' OR ''1''=''1'
```

被当作普通字符串处理，不会改变 SQL 逻辑。

这就是**绑定变量（bind variable）**的意义：
**把“代码”和“数据”严格区分，用户输入永远不能变成 SQL 指令。**

---

# 四、SQL 注入的本质

一句话：

> SQL 注入 = 用户输入被当成 SQL 代码执行。
> 绑定变量 = 用户输入永远只当“值”，不会变成代码。

这和浏览器的 XSS（跨站脚本）非常类似：
都是“数据变成代码”导致的攻击。

---

# 五、SQL 注入为什么一直存在？（演化原因）

因为开发者喜欢偷懒，用字符串拼接 SQL：

```python
sql = f"SELECT * FROM users WHERE name = '{user_input}'"
```

但现代所有数据库驱动（cx_Oracle、psycopg2、pymysql 等）都支持绑定变量，就是为了彻底解决这个问题。

---

如果你愿意，我可以展示：

- **一个简单可运行的“被注入”示例（Python + SQLite）**
- **和使用绑定变量的安全版本**

也能更直观地看到它是怎么“侵入 SQL 的语法结构”的。

# SQL注入示例

好，我们做一个**最小可复现的“被注入”登录接口**，再对比一个安全版本。
下面用的是 SQLite / Python 示例，但原理跟你用 Oracle 的 cx_Oracle 一样。

---

## 1. 准备一张用户表（概念上的）

假设数据库里有张表：

```sql
CREATE TABLE users (
    id       INTEGER PRIMARY KEY,
    username TEXT,
    password TEXT
);

INSERT INTO users (username, password)
VALUES ('alice', '123456'),
       ('bob',   'password');
```

现在我们要写一个“检查用户名密码是否正确”的函数。

---

## 2. 易被 SQL 注入的写法：字符串拼接

坏味道写法：

```python
def login_vulnerable(conn, username, password):
    # ❌ 把用户输入直接拼进 SQL 字符串
    sql = f"""
    SELECT * FROM users
    WHERE username = '{username}' AND password = '{password}'
    """
    print("执行的 SQL：", sql)
    cursor = conn.cursor()
    cursor.execute(sql)
    row = cursor.fetchone()
    cursor.close()
    return row is not None
```

### 正常用户输入

```python
login_vulnerable(conn, "alice", "123456")
```

打印的 SQL：

```sql
SELECT * FROM users
WHERE username = 'alice' AND password = '123456'
```

逻辑完全正常。

---

## 3. 攻击者如何“注入”？

攻击者根本不打算老老实实填密码，他直接填：

```python
username = "alice' --"
password = "随便"
login_vulnerable(conn, username, password)
```

拼接后的 SQL 会变成：

```sql
SELECT * FROM users
WHERE username = 'alice' --' AND password = '随便'
```

在 SQL 里 `--` 表示注释，后面整段都被当作注释忽略。
所以真正执行的其实是：

```sql
SELECT * FROM users
WHERE username = 'alice'
```

密码条件被“注释掉”了，
→ 只要 `alice` 这个用户存在，查询就会返回一行
→ `row is not None` 为真
→ **在逻辑上就是“无需密码成功登录”。**

这就是一个经典的 SQL 注入例子：
**用户输入改变了 SQL 的语法结构，而不是单纯提供了数据。**

你可以想象更狠的写法，比如：

```python
username = "alice'; DELETE FROM users; --"
```

拼接后可能生成：

```sql
SELECT * FROM users
WHERE username = 'alice'; DELETE FROM users; --' AND password = '随便'
```

如果数据库/驱动允许多语句执行，那直接把表删了。

---

## 4. 安全写法：绑定变量 / 参数化查询

现在我们改写成安全版本：

### SQLite 风格示例（占位符 `?`）：

```python
def login_safe(conn, username, password):
    sql = """
    SELECT * FROM users
    WHERE username = ? AND password = ?
    """
    print("SQL 模板：", sql)
    print("参数：", (username, password))
    cursor = conn.cursor()
    cursor.execute(sql, (username, password))
    row = cursor.fetchone()
    cursor.close()
    return row is not None
```

### Oracle / cx_Oracle 风格（命名绑定变量 `:username`）：

```python
def login_safe_oracle(conn, username, password):
    sql = """
    SELECT * FROM users
    WHERE username = :username AND password = :password
    """
    cursor = conn.cursor()
    cursor.execute(sql, username=username, password=password)
    row = cursor.fetchone()
    cursor.close()
    return row is not None
```

现在再用同样的“恶意输入”：

```python
username = "alice' --"
password = "随便"
login_safe(conn, username, password)
```

数据库内部看到的是：

- SQL 代码（固定）：

	```sql
    SELECT * FROM users
    WHERE username = ? AND password = ?
    ```
	
- 参数值：
	- `?1 = "alice' --"`
	- `?2 = "随便"`

它会把 `"alice' --"` 当成**纯字符串**去匹配 `username` 字段，
而不是把里面的引号与 `--` 当成 SQL 语法。
结果就是：
这个用户名根本不存在 → 查询返回空 → 登录失败。

**攻击者的“那一坨字符串”再花哨也只是值，不会变成代码。**

---

## 5. 把这件事抽象成一句话

- 不安全的写法：
	
	> 先把用户输入拼进 SQL 字符串，再整体当代码执行。  
	> → 用户输入有机会“长成一段 SQL 程序”，这就是注入。
	
- 安全的写法：
	
	> SQL 模板固定，用户输入通过绑定变量传进去，只当“值”。  
	> → 代码和数据在数据库内部是分开的，语法树早就解析完了，  
	> → 用户输入根本没有介入 SQL 结构的机会。

---

## 6. 回到你那段代码

```python
query = """
    SELECT COLUMN_NAME
    FROM ALL_TAB_COLUMNS
    WHERE TABLE_NAME = :table_name
    ORDER BY COLUMN_ID
"""
cursor.execute(query, table_name=table_name.upper())
```

- 使用 `:table_name` 作为绑定变量
- `table_name.upper()` 的值只会被当作 `TABLE_NAME` 字段的**比较值**，
	无论用户传入什么，都不可能插进 SQL 语法结构里。

所以它是**正确且安全的写法**。

---

再往下一层看，SQL 注入本质上就是「**字符串 + 解释执行**」这一类语言/系统的通病（和 shell 注入、模板注入、XSS 都是同一个思想族谱），只要你把“用户输入”和“代码字符串”混在一起，就给了别人“修改语法树”的机会。理解这一点后，很多安全最佳实践都会变得特别自然。