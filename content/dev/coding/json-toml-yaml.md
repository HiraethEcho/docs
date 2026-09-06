---
title: json toml yaml
date: 2026-08-10
summary: 三种数据格式的对比与使用
tags:
  - code
  - geek
categories: handbook
topics:
ai: true
---

# json toml yaml

## intro

json、toml、yaml 是三种记录数据的格式，都是纯文本、跨语言。用途类似，但设计哲学不同：

- **json**：机器友好。数据交换事实标准（API、前后端通信），无注释，语法严格
- **yaml**：人类友好。缩进即结构，配置/文档常用（docker-compose、k8s、CI），但灵活性带来坑
- **toml**：配置友好。`key = value` 的键值对，类型显式，无歧义（Cargo.toml、pyproject.toml、go.mod 前的时代是 Cargo 与 Python 生态标配）

## basic

三种格式都表达同样的基本类型：

- string：`"hello"`
- number：整数、浮点
- boolean：true / false
- null / none / nil：空值
- list / array：有序列表
- object / table：键值对映射

写法差异：

| 类型 | json | yaml | toml |
|---|---|---|---|
| 字符串 | `"双引号必须"` | 引号可省 | 引号可省 |
| 布尔 | `true` / `false` | `true` / `false` | `true` / `false` |
| 空值 | `null` | `null` / `~` / 留空 | 无内置，用注释或约定 |
| 列表 | `[1, 2]` | `- 1` | `[1, 2]` 或多行 |
| 对象 | `{"k": "v"}` | 缩进 | `[table]` 节、`[[表数组]]`、内联 `{ }` |

## example

同一个例子：一个班级的记录——班级名、人数、班主任、学生列表（姓名/性别/身高/成绩）。三种写法表达同一份数据：对象、嵌套对象、对象数组。再看几个其他场景，各有侧重。

### json

```json
{
  "name": "高一三班",
  "number": 50,
  "teacher": {
    "name": "张伟",
    "subject": "数学"
  },
  "students": [
    {
      "name": "小明",
      "gender": "男",
      "height": 160,
      "grades": {
        "math": 90,
        "english": 85
      }
    },
    {
      "name": "小红",
      "gender": "女",
      "height": 172,
      "grades": {
        "math": 95,
        "english": 92
      }
    }
  ]
}
```

json 里没有别的写法：键必须加引号，结构全靠 `{}` 与 `[]` 嵌套。

### yaml

```yaml
name: 高一三班
number: 50
teacher:
  name: 张伟
  subject: 数学
students:
  - name: 小明
    gender: 男
    height: 160
    grades:
      math: 90
      english: 85
  - name: 小红
    gender: 女
    height: 172
    grades:
      math: 95
      english: 92
```

yaml 用缩进表示层级，`-` 表示列表项，最省字符。

### toml

```toml
name = "高一三班"
number = 50

[teacher]
name = "张伟"
subject = "数学"

[[students]]
name = "小明"
gender = "男"
height = 160

[students.grades]
math = 90
english = 85

[[students]]
name = "小红"
gender = "女"
height = 172

[students.grades]
math = 95
english = 92
```

`[teacher]` 定义表（对象），`[[students]]` 定义表数组（对象列表），每个 `[[students]]` 追加一个元素。`[students.grades]` 挂在最近的一个 `[[students]]` 元素下，作为它的子表。

### toml 写法变体

同样一份数据，toml 有多种等价的写法。

**点号键**——不写表头，直接展开：

```toml
teacher.name = "张伟"
teacher.subject = "数学"
```

**内联表**——小对象压成一行，用 `{ }`：

```toml
teacher = { name = "张伟", subject = "数学" }
```

```toml
[[students]]
name = "小明"
gender = "男"
height = 160
grades = { math = 90, english = 85 }
```

**整个列表内联**——

```toml
students = [
  { name = "小明", gender = "男", height = 160, grades = { math = 90, english = 85 } },
  { name = "小红", gender = "女", height = 172, grades = { math = 95, english = 92 } },
]
```

数组允许尾逗号；内联表内不允许换行（多行对象必须用 `[[ ]]` / `[ ]` 表头展开）。

**同一个键只能有一种写法**：`students` 已经用 `[[students]]` 定义过，就不能再用 `students = [...]` 赋值，反之亦然——toml 会直接报错，避免歧义。

### 例2：日期时间

记录一次发布的时间：

```json
{
  "release": {
    "date": "2026-08-10",
    "time": "10:30:00",
    "datetime": "2026-08-10T10:30:00+08:00"
  }
}
```

```yaml
release:
  date: 2026-08-10
  time: 10:30:00
  datetime: 2026-08-10 10:30:00 +08:00
```

```toml
[release]
date = 2026-08-10
time = 10:30:00
datetime = 2026-08-10T10:30:00+08:00
```

toml 里日期时间是原生类型，解析后直接得到日期对象；json 里只能是字符串，自己解析。yaml 1.1 会把裸日期当 timestamp 解析，不同解析器行为不一——又一个坑。

### 例3：深层嵌套

服务配置，三层嵌套：

```json
{
  "server": {
    "host": "0.0.0.0",
    "port": 8080,
    "database": {
      "pool": { "max": 20, "min": 2 },
      "backup": { "enabled": true }
    }
  }
}
```

```yaml
server:
  host: 0.0.0.0
  port: 8080
  database:
    pool:
      max: 20
      min: 2
    backup:
      enabled: true
```

```toml
[server]
host = "0.0.0.0"
port = 8080

[server.database]
[server.database.pool]
max = 20
min = 2

[server.database.backup]
enabled = true
```

yaml 缩进一眼看出层级；toml 要心里拼 `[server.database.pool]` 路径，层级越深越费劲；json 括号多但结构明确，机器读最稳。

### 例4：yaml 锚点复用

多个服务共享同一组默认值：

```yaml
defaults: &defaults
  retries: 3
  timeout: 30

service_a:
  <<: *defaults
  url: /api/a

service_b:
  <<: *defaults
  url: /api/b
```

`&defaults` 定义锚点，`*defaults` 引用，`<<` 合并键展开，等价于：

```yaml
service_a:
  retries: 3
  timeout: 30
  url: /api/a
```

json/toml 没有复用机制，只能复制粘贴。（`<<` 合并键是 yaml 1.1 特性，部分解析器支持不一。）

### 例5：表数组套表数组

商品列表，每个商品有零件列表：

```json
{
  "products": [
    { "name": "火腿", "parts": [{ "name": "骨头" }] },
    { "name": "鸡蛋", "parts": [{ "name": "蛋壳" }] }
  ]
}
```

```yaml
products:
  - name: 火腿
    parts:
      - name: 骨头
  - name: 鸡蛋
    parts:
      - name: 蛋壳
```

```toml
[[products]]
name = "火腿"

[[products.parts]]
name = "骨头"

[[products]]
name = "鸡蛋"

[[products.parts]]
name = "蛋壳"
```

`[[products.parts]]` 同样挂到最近一个 `[[products]]` 元素下——表数组里再嵌表数组，toml 也能表达，只是顺序读起来要对照。

## difference

### json

特点：

- 语法最严格：键必须加引号，字符串必须双引号，无注释
- 类型是语言无关的：JS 的 `JSON.parse`、Python 的 `json` 模块直接映射
- 不支持日期等特殊类型，全靠字符串约定
- 不允许尾逗号

优点：跨语言无歧义，生态最广，任何语言都有标准库支持。缺点：手写难、无注释、冗余（括号多），不适合人类维护的配置文件。

用途：API 请求/响应、前端状态、数据交换。

### yaml

特点：

- 缩进即结构，2 空格层级
- 字符串可省略引号（但有歧义：`yes` 在某些解析器里是布尔）
- 支持注释 `#`
- 锚点 `&` / 别名 `*` 复用节点，多文档 `---` 分隔
- 超级集是 json：合法 json 也是合法 yaml

优点：可读性最强，写注释方便，表达嵌套最省字符。缺点：缩进错误静默改变结构、隐式类型转换坑多、解析器之间行为不一致。

用途：配置文件（docker-compose、k8s、GitHub Actions、Ansible）、文档 frontmatter。

### toml

特点：

- 显式类型：`key = value`，值自带类型标注
- `[table]` 节定义嵌套表，`[[array-of-tables]]` 定义表数组，内联表 `{ key = value }` 压单行
- 一个键只能定义一次，重复定义直接报错
- 原生支持日期时间（RFC 3339）
- 支持多行字符串 `"""`、注释 `#`
- 不支持 null——用注释或删除键表达"没有"

优点：无歧义（每种写法只有一种解析结果）、类型明确、适合手写配置。缺点：嵌套深时 `[a.b.c]` 节路径难读，表达复杂层级不如 yaml 直观。

用途：项目配置（Cargo.toml、pyproject.toml、deno.json 之外几乎成了 Rust/Python 生态默认）、应用设置。

### 选型

- 程序间传数据 → **json**
- 人写的配置，层级多、要注释 → **yaml**
- 人写的配置，类型要严格、无歧义 → **toml**
