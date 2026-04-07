---
title: typst记录
date: 2026-03-17
summary: 在这里记下写typst过程中遇到的各种问题和解决方法（希望能解决）
tags:
categories:
topics:
series:
status: wip
---

# Typst记录

## basic

基本类型为内容块 _content_。分为标记模式和代码模式。默认为标记模式。

### 标记模式

| 效果          | 标记语法     | 示例                            |
| ------------- | ------------ | ------------------------------- |
| **加粗**      | `*文本*`     | `*重要*` → **重要**             |
| _斜体_        | `_文本_`     | `_强调_` → _强调_               |
| `代码`        | `` `代码` `` | `` `let x = 1` `` → `let x = 1` |
| ~~删除线~~    | `-文本-`     | `-删除-` → ~~删除~~             |
| <u>下划线</u> | `+文本+`     | `+下划线+` → <u>下划线</u>      |

> **注意**：这些标记符号可以嵌套使用，比如 `*_加粗斜体_*` 会同时应用加粗和斜体。

| 效果     | 标记语法 | 示例                          |
| -------- | -------- | ----------------------------- |
| 上标     | `^内容^` | `x^2^` → x²                   |
| 下标     | `~内容~` | `H~2~O` → H₂O                 |
| 行内数学 | `$...$`  | `$E = mc^2$` → 呈现为数学公式 |

| 功能         | 标记语法      | 示例                             |
| ------------ | ------------- | -------------------------------- |
| 超链接       | `[文本](url)` | `[Typst官网](https://typst.app)` |
| 内部交叉引用 | `@label`      | `如 @fig:example 所示`           |
| 文献引用     | `@citekey`    | `Knuth @knuth1984`               |

- **标题**：用 `=` 表示标题级别，如 `= 一级标题`、`== 二级标题`。
- **无序列表**：用 `-` 或 `+` 开头。
- **有序列表**：用 `+` 后跟数字（或直接 `+` 自动编号）。
- **引用块**：用 `> ` 开头。

示例：

```typst
= 第一章
== 第一节
- 项目1
- 项目2
> 这是一个引用块。
```

列表的函数写法

```typst
#list(
  [
    第一项
    akljsdfkjk
  ],
  [
    + 第二项
      ajsfdkalsj $a^2 + b^2 =c^2$
    + $ a^2 + b^2 =c^2 $
      kljasdfl
  ],
  marker: [-],
  spacing: 1em,
)

#enum(
  numbering: "(a)",
  [
    选项a
  ],
  [
    选项b
  ],
  list(
    [
      wired,
    ],
    [syntax],
  ),
  [
    something else
    #list(
      marker: [-],
      [
        子选项i
      ],
      [
        子选项ii
      ],
      [
        #enum(
          [
            第一项
            akljsdfkjk
          ],
          [
            第二项
            ajsfdkalsj $a^2 + b^2 =c^2$

            $ a^2 + b^2 =c^2 $
            kljasdfl
          ],
        )

      ],
    )
  ],
)
```

### block

```typst
// 一个简单的内容块
[这是一个普通段落]

// 内容块可以嵌套和包含代码
#let my-box = rect[#lorem(10)]   // 矩形内生成 10 段 Lorem ipsum

// 内容块可以作为函数参数
#text(red)[我是红色文字]          // [我是红色文字] 就是一个内容块
```

### 代码模式

`#` 调用函数进入代码模式

```typst
// 标记模式：普通文字
Hello, world!

// 井号引入代码模式：计算 1+2 的结果
#(1 + 2)   // 输出 3

// 调用函数
#text(red)[这段文字变红]
```

用`()`传递参数

`let`用来定义

```typst
// 定义变量
#let name = "张三"
#name           // 输出：张三

// 定义函数（可以有参数和返回值）
#let greet(name) = [你好，#name!]
#greet("李四")   // 输出：你好，李四!

// 定义带标签参数的函数
#let boxed(content) = rect[#content]
#boxed[重要内容]
```

### label and cite

#### label

`label` 用于给一个元素（如标题、图表、公式等）起一个唯一的名字，方便后续用 `ref` 引用。

```typst
#figure(
  rect(width: 100%, height: 2cm),
  caption: [这是一个图],
) <fig:example>   // 标签写在尖括号里，紧跟在元素后面

#let my-label = "fig:dynamic"

#figure(
  rect(width: 100%),
  caption: [动态标签图]
) #label(my-label)

= 章节标题
#label("sec:title")   // 单独给上一个标题添加标签

= 重要章节 <sec:important>

在 @sec:title 中...

```

#### ref

`ref` 用于引用之前打上标签的元素，它会自动生成带有编号的链接（如“图1”）。

```typst
如图 #ref(<fig:example>) 所示。
```

也可以使用带语法的简写 `@`：

```typst
如图 @fig:example 所示。
```

输出效果类似：“如图 1 所示”。如果希望自定义引用文本，可以使用 `#ref(<label>, supplement: [自定义])`。

## Templates and Packages

### 自定义标记样式

你可以通过 `#set` 和 `#show` 来改变这些标记符号的最终样式。例如，让所有加粗文字变成红色：

```typst
#show strong: set text(red)   // strong 对应 *...*
*危险*   // 显示为红色加粗
```

也可以给所有代码块添加灰色背景：

```typst
#show raw: set block(fill: luma(240))
```

更多

```typst
// 选中所有 heading，并将其 text 颜色设置为蓝色
#show heading.where(level: 2): set text(blue)

== 我是二级标题  // 这个标题会自动变成蓝色
```

and

```typst
// 定义一个函数，接收标题内容，然后返回一个包含内容的矩形框
#show heading: it => rect[
  #it.body
]

= 我是一个有外框的标题
```

### import

`#import "@preview/theorion:0.5.0": *`

- **`@preview/xxx`**：`@preview` 是官方包的命名空间。
- **`:版本号`**：跟在包名后面的是**版本号**（如 `:0.1.0`）

| 写法                        | 效果                                                                                                   | 如何使用                                       |
| :-------------------------- | :----------------------------------------------------------------------------------------------------- | :--------------------------------------------- |
| `import "utils.typ"`        | 把整个文件作为一个模块导入，但名字被“锁”在了模块里。                                                   | `#utils.alert("Hi")` (需要加前缀)              |
| `import "utils.typ": alert` | 只“开门”取出了 `alert` 这一个函数。                                                                    | `#alert("Hi")` (可以直接使用)                  |
| `import "utils.typ": *`     | **把你问题里的 `:*`**。这是通配符，意思是把 `utils.typ` 里**所有**公共的函数和变量都“搬”到当前作用域。 | `#alert("Hi")` 和 `#warn("Oops")` 都能直接用。 |

## From LaTeX

### bib

假设有一个 `ref.bib` 文件，内容如下（BibTeX 格式）：

```bib
@article{knuth1984,
  author = {Knuth, Donald E.},
  title = {Literate Programming},
  journal = {The Computer Journal},
  volume = {27},
  number = {2},
  pages = {97--111},
  year = {1984}
}
```

```typst
// 引入参考文献数据库
#bibliography("ref.bib")

// 引用文献
Knuth 提出了文式编程 @knuth1984。

// 或者使用 #cite 函数
#cite(<knuth1984>, form: "proposition")   // 可以指定引用格式
```

`@knuth1984` 会被自动渲染为作者年份格式（如 “Knuth 1984”），并生成一个链接。  
**注意**：`bibliography` 函数通常放在文档末尾，它会根据所有被引用的条目自动生成参考文献列表。如果希望显示全部条目（即使未引用），可以添加参数 `full: true`。

### 自定义引用样式

通过 `style` 参数指定 CSL 文件（Citation Style Language）来改变引用格式：

```typst
#bibliography("ref.bib", style: "apa.csl")
```

Typst 内置了多种常见样式（如 "chicago-author-date", "ieee" 等），也可以下载 `.csl` 文件使用。

## Tricks

标签命名习惯：通常用 `前缀:名称` 的形式，如 `fig:`、`sec:`、`eq:`，但不是必须的，只是为了可读性。

---
