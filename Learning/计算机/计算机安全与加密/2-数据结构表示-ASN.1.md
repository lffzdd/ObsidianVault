---
aliases: 2-ASN.1
tags:
date created: 星期一, 三月 31日 2025, 9:06:39 晚上
date modified: 星期一, 三月 31日 2025, 9:09:53 晚上
---
# ASN. 1 语法
>好的！我们从零开始学习 **ASN.1（Abstract Syntax Notation One）** 的基本语法。你可以把 ASN.1 当成是一种**定义数据结构的语言**，类似于 JSON 或 XML 的 Schema，但它是为**二进制格式**设计的，并且更适合网络通信、加密协议等场景。
## **1. ASN.1 的基本结构**
ASN.1 的语法主要由 **数据类型** 和 **数据结构** 组成。它的基本结构如下：
```asn.1
名称 ::= 类型
```
`::=` 表示 " 定义为 "（类似 C 语言中的 `=`）。
### **示例**
定义一个整数：
```asn.1
MyNumber ::= INTEGER
```
这表示 **`MyNumber` 是一个整数类型**。
---
## **2. 基本数据类型**
ASN.1 提供了一些**内置的数据类型**，类似于 C 语言的 `int`, `char*` 或 JSON 的 `string`, `number` 等。
| 数据类型                | 说明                          |
| ------------------- | --------------------------- |
| `BOOLEAN`           | 布尔值（TRUE / FALSE）           |
| `INTEGER`           | 整数                          |
| `REAL`              | 浮点数                         |
| `OCTET STRING`      | 字节序列（类似于 `bytes[]`）         |
| `UTF8String`        | UTF-8 字符串                   |
| `IA5String`         | ASCII 字符串（类似 C 语言的 `char*`） |
| `NULL`              | 空值                          |
| `OBJECT IDENTIFIER` | 对象标识符（OID）                  |
### **示例**
```asn.1
isActive ::= BOOLEAN
age       ::= INTEGER
piValue   ::= REAL
data      ::= OCTET STRING
name      ::= UTF8String
email     ::= IA5String
```
等价于：
```json
{
    "isActive": true,
    "age": 25,
    "piValue": 3.1415,
    "data": "0xA1B2C3D4",
    "name": "Alice",
    "email": "alice@example.com"
}
```
---
## **3. 复合数据类型**
### **（1）SEQUENCE（类似结构体）**
`SEQUENCE` 相当于 **结构体（struct）**，用于定义多个字段的组合。
```asn.1
User ::= SEQUENCE {
    name    UTF8String,
    age     INTEGER,
    email   IA5String
}
```
等价于 JSON：
```json
{
    "name": "Alice",
    "age": 25,
    "email": "alice@example.com"
}
```
---
### **（2）SEQUENCE OF（类似数组）**
`SEQUENCE OF` 表示**列表（数组）**：
```asn.1
UserList ::= SEQUENCE OF User
```
表示 **`UserList` 是一个 `User` 结构的列表**：
```json
[
    {
        "name": "Alice",
        "age": 25,
        "email": "alice@example.com"
    },
    {
        "name": "Bob",
        "age": 30,
        "email": "bob@example.com"
    }
]
```
---
### **（3）SET（类似 SEQUENCE，但无序）**
`SET` 和 `SEQUENCE` 很像，但**它的字段是无序的**。
```asn.1
UserSet ::= SET {
    name  UTF8String,
    age   INTEGER
}
```
在二进制编码时，**SET 的字段可能会换顺序**，但解析时不会丢失信息。
---
### **（4）CHOICE（类似 Union，可选类型）**
`CHOICE` 表示多个可能的类型（**类似 C 语言的 `union`**）。
```asn.1
ContactInfo ::= CHOICE {
    phoneNumber  IA5String,
    email        UTF8String
}
```
等价于：
```json
{
    "email": "alice@example.com"
}
```
或：
```json
{
    "phoneNumber": "+123456789"
}
```
**一个 `CHOICE` 结构中只能存在一个字段**。
---
### **（5）ENUMERATED（类似枚举）**
```asn.1
Gender ::= ENUMERATED {
    male (0),
    female (1),
    other (2)
}
```
等价于：
```json
{
    "Gender": "male"
}
```
---
## **4. 复杂示例**
我们来定义一个 **银行账户信息**：
```asn.1
BankAccount ::= SEQUENCE {
    accountNumber  INTEGER,
    holderName     UTF8String,
    balance        REAL,
    transactions   SEQUENCE OF Transaction
}
Transaction ::= SEQUENCE {
    id            INTEGER,
    amount        REAL,
    transactionType ENUMERATED {
        deposit (0),
        withdrawal (1),
        transfer (2)
    }
}
```
等价的 JSON：
```json
{
    "accountNumber": 12345678,
    "holderName": "Alice",
    "balance": 2500.50,
    "transactions": [
        {
            "id": 1,
            "amount": 500.00,
            "transactionType": "deposit"
        },
        {
            "id": 2,
            "amount": 200.00,
            "transactionType": "withdrawal"
        }
    ]
}
```
---
## **5. ASN.1 编码（BER/DER）**
ASN.1 只是一种**描述语言**，数据的实际存储和传输需要用**BER、DER、PER**等编码格式。
比如：
```asn.1
age ::= INTEGER 25
```
编码成 **DER（二进制）**：
```
02 01 19
```
- `02`：INTEGER 类型
- `01`：长度 1 字节
- `19`：值 25（十六进制 0x19）
你可以用 **OpenSSL** 解析 ASN.1 数据：
```bash
openssl asn1parse -in certificate.der
```
---
## **6. 总结**
- **ASN.1 是一种数据结构定义语言**，用于描述数据格式。
- **基本类型**：`INTEGER`, `BOOLEAN`, `UTF8String`, `OCTET STRING` 等。
- **复合类型**：
    - `SEQUENCE`：类似结构体。
    - `SEQUENCE OF`：数组/列表。
    - `SET`：类似 `SEQUENCE`，但字段无序。
    - `CHOICE`：可选字段（类似 Union）。
    - `ENUMERATED`：枚举类型。
- **ASN.1 只是格式，编码要用 BER/DER/PER**。
---
**下一步学习方向：**
1. **分析真实 ASN.1 数据**（如 TLS/SSL 证书的 X.509）。
2. **学习 BER 和 DER 编码**，看看 ASN.1 是如何转成二进制数据的。
3. **使用 Python 或 OpenSSL 解析 ASN.1 结构**。
你想直接解析 ASN.1 证书，还是先了解 BER/DER 细节？🚀
# ASN. 1 是抽象格式, 不是实际存储
你的理解很对，ASN.1 和 JSON、XML 在某种程度上类似，都是用来**描述数据结构的格式**，但 ASN.1 并不直接用于存储数据，而是**定义数据的结构**。
### **ASN.1 vs. JSON vs. XML**
|特性|ASN.1|JSON|XML|
|---|---|---|---|
|**用途**|数据结构描述（协议、证书）|轻量级数据交换|结构化文档描述|
|**可读性**|类似编程语言，较抽象|可读性高|可读性较高|
|**存储格式**|需要 BER/DER/PER 等编码|纯文本|纯文本|
|**应用场景**|通信协议（TLS、X.509 证书）|Web API、配置文件|文档、配置文件|
|**存储大小**|二进制紧凑|结构清晰但较冗余|结构清晰但较冗余|
|**扩展性**|复杂但强大|易扩展|可扩展但解析慢|
### **ASN.1 并不常见的原因**
- JSON 和 XML 直接用于存储数据，因此你经常会看到 `.json` 和 `.xml` 文件。
- ASN.1 主要用于**定义数据格式**，实际存储时通常会**转换成二进制格式**（DER/BER 等），所以你不会直接看到 `.asn1` 文件，而是 `.der`, `.crt`, `.cer`, `.pem` 等文件。
### **ASN.1 只是描述，真正存储是 BER/DER/PER**
ASN.1 只是**数据的“抽象语法”**，但数据要存储和传输时，必须转换成**实际编码格式**：
1. **BER（Basic Encoding Rules）**：基础编码方式，允许多种编码方式，较灵活。
2. **DER（Distinguished Encoding Rules）**：BER 的子集，编码唯一确定（用于 TLS/SSL 证书）。
3. **PER（Packed Encoding Rules）**：比 BER 更紧凑，适用于低带宽环境（如 5G 协议）。
你平时看到的 **X.509 证书（`.der`、`.pem`）**，就是 **ASN.1 结构数据经过 DER 编码** 的二进制文件。
---
### **ASN.1 真实应用示例**
#### **TLS/SSL 证书**
X.509 证书就是基于 ASN.1 定义的，例如：
```asn.1
Certificate ::= SEQUENCE {
    tbsCertificate     TBSCertificate,
    signatureAlgorithm AlgorithmIdentifier,
    signatureValue     BIT STRING
}
```
**转换成 DER 后就是 `.cer` 文件**，你可以用 OpenSSL 解析：
```bash
openssl asn1parse -in cert.der -inform DER
```
#### **SIM 卡数据格式（3GPP）**
手机 SIM 卡中的信息（比如 ICCID）使用 ASN.1 定义：
```asn.1
ICCID ::= OCTET STRING (SIZE(10..20))
```
#### **比特币交易（Bitcoin Script）**
比特币的交易输入、输出数据结构也是 ASN.1 的变体，只是直接存储二进制数据。
### **总结**
- **ASN.1 本身不是存储格式，而是定义格式**，类似 JSON Schema，但更强大。
- **ASN.1 数据不会以文本形式存储，而是转换为 BER/DER/PER 进行存储**。
- **ASN.1 广泛用于安全通信协议、X.509 证书、SIM 卡、比特币等**。
你现在是想深入 ASN.1 的 **BER/DER 编码**，还是解析 **真实 ASN.1 数据**（比如 TLS 证书）？ 🚀