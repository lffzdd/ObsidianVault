# Schema和服务器对象

![[attachments/eab7f87c-b9c6-4fbd-82a0-e3a013228acc.png]]

这个界面来自 DataGrip 连接 Oracle 数据库后的“数据库资源管理器”（Database Explorer）面板。

让我们拆解一下你看到的两个条目：

---

### 一、`SLYHBI` —— 实际是一个 **Schema（模式）**

在 Oracle 里，**每个用户默认对应一个 Schema**，可以理解为属于这个用户的“数据库命名空间”。

它的作用是用来组织表、视图、存储过程等对象。举个类比：

- 整个 Oracle 实例好比一个大型仓库；
- 每个 Schema 就是一间仓库房；
- 每个房间里可以放自己的表、视图、索引等对象。

所以 `SLYHBI` 通常表示一个 Oracle 用户名（比如登录账号），而 DataGrip 会自动展示该用户的 Schema 名作为节点。

你展开 `SLYHBI`，会看到类似：

- 表（Tables）
- 视图（Views）
- 索引（Indexes）
- 序列（Sequences）
- 过程（Procedures）等

这些都是 Schema 下的对象。

---

### 二、“服务器对象”——这是 DataGrip 对 **数据库级别对象** 的分类

这一部分不是 Schema 专属，而是整个数据库级别的全局对象，常包括：

- **用户（Users）**
- **角色（Roles）**
- **权限（Privileges）**
- **数据库链接（DB Links）**
- **调度任务、触发器** 等

这些对象是数据库管理员(DBA)层级的内容，用于管理安全、跨库访问和系统配置。

例如在“服务器对象”下你可能会看到：

- Public Synonyms（公共同义词）
- Database Links（远程连接配置）
- Scheduler Jobs（调度任务）

---

### 三、DataGrip 的层次逻辑总结

你可以这样理解这个结构：

```css
连接 (slyh)
│
├── SLYHBI （Schema 层，代表一个用户命名空间）
│   ├── Tables
│   ├── Views
│   └── Procedures
│
└── 服务器对象（Database 层，全局资源与配置）
    ├── Users
    ├── Roles
    └── Links
```

---

### 背后的设计动机

这个层次来自 Oracle 自身的“**多用户共享一个数据库实例**”的设计哲学。

不像 MySQL 那样用“一个数据库名”代表一组表，Oracle 里所有数据都在同一个物理数据库中，通过 **Schema** 来隔离不同用户的命名空间。

DataGrip 只是把这种关系用图形界面可视化展示出来。

---

如果你想深入理解：

- **Schema** 是逻辑隔离（不同用户的数据命名空间）；
- **服务器对象** 是全局资源；
- 而 **连接（slyh）** 是你在 DataGrip 里配置的访问入口，它绑定了一个具体的 Oracle 用户账号和连接字符串。

是否希望我进一步讲一下 Oracle 为什么选择“schema=用户”这种独特设计（和 MySQL、PostgreSQL 的差异）？

# 同义词，虚拟视图

> ![[attachments/a39b9ed7-3da9-47ea-b344-6de4155d8f81.png]]

非常好，这三个名称——**同义词、用户、虚拟视图**——都反映了 Oracle 数据库在“多用户、多层命名空间”设计下，为了组织和隔离资源而引入的机制。我们来逐个拆解，并讲清楚它们在 Oracle 架构中存在的**动机、历史、逻辑关系**。

---

## 一、同义词（Synonym）——为了解决“名字太长、跨用户访问困难”

### 1. 背景问题

在 Oracle 的早期设计中，**每个用户就是一个独立的 Schema**，这导致一个问题：

假设用户 `A` 想访问用户 `B` 的表，必须写出完整路径：

```sql
SELECT * FROM B.ORDERS;
```

如果这个表在应用里被频繁引用（比如 50 个 SQL 都用到），那么这些 SQL 都要写 `B.ORDERS`，很麻烦，也不利于迁移。

### 2. 设计动机

于是 Oracle 设计了 **Synonym（同义词）**——一种给对象起“别名”的机制。

同义词就像操作系统里的“快捷方式”，可以把对象映射成一个简短的名字。

举例：

```sql
CREATE SYNONYM ORDERS FOR B.ORDERS;
```

这样当前用户就能直接写：

```sql
SELECT * FROM ORDERS;
```

而不必每次带上 `B.`。

### 3. 进一步用途

同义词分为：

- **私有同义词（Private Synonym）**：属于某个用户；
- **公共同义词（Public Synonym）**：所有用户都能用。

这在大型系统里特别有用，比如应用开发团队统一用公共同义词屏蔽底层表的归属关系，方便迁移或切换数据库用户。

---

## 二、用户（User）——同时也是 Schema 的拥有者

### 1. 历史背景

Oracle 和 MySQL 最大的哲学差异在于：

- **MySQL 的 database 是逻辑数据库；用户只是权限控制。**
- **Oracle 的 user 同时是一个 Schema（命名空间）。**

所以在 Oracle 里你创建一个用户：

```sql
CREATE USER SLYHBI IDENTIFIED BY password;
```

系统就自动为他建了一个独立的 Schema：

他创建的所有对象（表、视图、索引等）都放在 `SLYHBI` 名下。

### 2. 设计目的

这是 Oracle 为了支持**多租户（multi-tenant）式共享数据库架构**的早期设计理念——

多个业务系统或团队可以共用一个物理数据库实例，但通过用户隔离对象命名空间和权限。

### 3. 在 DataGrip 中的体现

你展开“服务器对象 → 用户”，看到的那几十个用户，

其实就对应数据库中创建的所有 **Schema 拥有者**。

例如：

- `SYS`、`SYSTEM` 是数据库管理员；
- `HR` 是演示示例；
- `SLYHBI` 是你的业务账户。

---

## 三、虚拟视图（View / Virtual View）——为了解决“数据访问抽象层”的需求

### 1. 原始问题

在数据库早期，直接访问表意味着用户能看到所有底层字段。

但有时业务逻辑只需要部分字段、或需要做聚合、或不希望暴露表结构。

### 2. 设计动机

于是引入了 **视图（View）** 概念：

视图本身不存储数据，只存储一条 SQL 查询定义。

当你访问视图时，数据库动态执行这条查询，把结果返回——因此叫 **虚拟视图**（Virtual View）。

```sql
CREATE VIEW ACTIVE_USERS AS
SELECT ID, NAME FROM USERS WHERE STATUS = 'ACTIVE';
```

访问它：

```sql
SELECT * FROM ACTIVE_USERS;
```

相当于执行了那条 SQL。

### 3. 系统虚拟视图（Data Dictionary Views）

在 Oracle 中，“虚拟视图”有另一层含义——

**系统字典视图（Data Dictionary Views）**，即以 `USER_`、`ALL_`、`DBA_` 开头的那些。

它们并不是普通业务视图，而是数据库元数据的入口，比如：

- `USER_TABLES`：当前用户的表；
- `ALL_VIEWS`：当前用户可访问的所有视图；
- `DBA_USERS`：所有数据库用户。

它们本身是对底层系统表（如 `SYS.TAB$`、`OBJ$`）的封装，**是 Oracle 管理员级操作的窗口**。

DataGrip 在“虚拟视图”里展示的，就是这些系统层的视图。

---

## 四、总结结构化关系

|对象类别|本质|用途|背后设计思想|
|---|---|---|---|
|同义词 (Synonym)|名称别名|跨 Schema 访问或简化对象引用|解决命名复杂和迁移困难|
|用户 (User)|Schema 拥有者|逻辑隔离、权限控制|多用户共享同一实例|
|虚拟视图 (Virtual View)|查询定义|封装数据访问或系统元数据|数据抽象与安全隔离|

---

如果你愿意深入一点，我可以接着讲讲 Oracle 的三层命名体系（`USER_`、`ALL_`、`DBA_` 系列视图）是如何反映权限层次的，它直接体现了 Oracle 在数据可见性上的底层机制。是否继续讲下这一层？