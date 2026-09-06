---
title: regex
tags:
  - linux
  - geek
  - tools
  - shell
categories: handbook
ShowContentinRSS: false
---

# regex

## vim

```vim
/^hello\>      " 匹配行首的单词 hello
:%s/foo/bar/g  " 全局替换 foo 为 bar
:%s/\(\d\+\)/[\1]/g  " 将所有数字用 [] 包裹（捕获组）
```

捕获组：`:%s/\(\d\+\)/[\1]/g` 将所有数字用 `[]` 包裹（捕获组）

删除空行

```vim
:g/^$/d # delete empty line
:g/^\s*$/d # delete line with only empty char
:%s/^\s*$//g # clean empty char
```

## shell

文件名匹配（通配符）、`=~` 操作符用于字符串匹配。Shell 本身使用 glob 模式（非标准正则），但 `[[ ]]` 中 `=~` 支持 **扩展正则表达式（ERE）**。

```shell
if [[ "hello" =~ ^he ]]; then
    echo "匹配"
fi
```

```
ls *.txt       # 匹配所有.txt文件
ls file?.log   # 匹配 file1.log, file2.log 等
```

正则表达式（Regex）在不同工具和编程语言中的语法和功能略有差异。以下是 **Shell、Vim、Lua、Python、grep、sed、ripgrep** 中正则表达式的用法对比和示例：

## Lua

- **用途** ：字符串匹配（ `string.match`, `string.gsub` ）。
- **特点** ：Lua 的 regex 是 **轻量级** 的，不支持常见的 `\d` 或 `\w` ，需用字符类如 `%d` 。

```lua
-- 提取数字
local str = "abc123"
local num = string.match(str, "%d+")  -- 返回 "123"
-- 替换
local new_str = string.gsub("hello world", "world", "Lua")
```

- `%a` 字母a-z,A-Z
- `%b` %bxy,以x和y进行成对匹配
- `%c` 控制字符ASCII码 十进制转义表示为\0 - \31
- `%d` 数字 0 - 9
- `%f` %f[char-set]，边界匹配，前一个不在范围内，后一个在
- `%g` 除了空格以外的可打印字符 十进制转义表示为\33 - \126
- `%l` 小写字母 a - z
- `%u` 大写字母 A - Z
- `%s` 空格 十进制转义表示为\32
- `%p` 标点符号，即!@#$%^&\*()`~-\_=+{}:"<>?[];',./| 32个字符
- `%w`字母数字 a - z, A - Z, 0 - 9`%x` 十六进制符号 0 - 9, a - f, A - F

## Python（re 模块）

- **特点** ：支持 **标准正则表达式（PCRE）** ，功能全面（如非贪婪匹配、命名组等）。
-

```python
import re
re.findall(r'\d+', 'abc123')      # 返回 ['123']
re.sub(r'foo', 'bar', 'foobar')   # 替换为 'barbar'
# 命名捕获组
match = re.search(r'(?P<year>\d{4})', '2023')
print(match.group('year'))  # 输出 '2023'
```

## grep

- **用途** ：文件内容搜索。
- **模式** ：
- `grep` 默认使用 **基本正则表达式（BRE）** 。
- `grep -E` 或 `egrep` 使用 **扩展正则表达式（ERE）** 。
- `grep -P` 支持 **PCRE** （部分系统）。
- **示例** ：

```
grep '^hello' file.txt       # 匹配行首的 hello（BRE）
grep -E '[0-9]{3}' file.txt # 匹配3位数字（ERE）
grep -P '\d+' file.txt      # 匹配数字（PCRE）
```

## sed

- **用途** ：流编辑器，支持查找替换。
- **模式** ：
- 默认使用 **BRE** 。
- `sed -E` 启用 **ERE** 。
- **示例** ：

```
sed 's/foo/bar/g' file.txt           # 替换 foo 为 bar（BRE）
sed -E 's/[0-9]+/NUM/g' file.txt     # 替换数字为 NUM（ERE）
sed 's/\(.*\)/\U\1/' file.txt        # 转换为大写（捕获组）
```

## ripgrep（rg）

- **用途** ：高性能文件搜索。
- **特点** ：默认使用 **PCRE** ，支持 Unicode 和智能大小写。
- **示例** ：

```
rg '\d{3}'       # 搜索3位数字
rg -i 'hello'    # 忽略大小写搜索
rg -U 'foo.*bar' # 多行匹配（跨行）
```

## 常见正则符号对比

| 功能     | PCRE/Python | BRE (grep/sed) | ERE (grep -E) | Vim       | Lua     |
| -------- | ----------- | -------------- | ------------- | --------- | ------- |
| 数字     | `\d`        | `[0-9]`        | `[0-9]`       | `\d`      | `%d`    |
| 单词字符 | `\w`        | `[[:alnum:]]`  | `\w`          | `\w`      | `%a`    |
| 量词     | `+``*``?`   | `\+` `\*` `\?` | `+` `*``?`    | `\+` `\*` | `+` `*` |
| 捕获组   | `(...)`     | `\(...\)`      | `(...)`       | `\(...\)` | `(...)` |
