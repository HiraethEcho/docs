---
title: Caldav and iCalendar
date: 2026-08-30
ai: true
---

# About Caldav and iCalendar

在 vibe coding 任务日程管理器 calman 时和 deepseek 对话学习的一些知识总结。

## 1. 日历协议与数据格式概述

> 用户希望了解 RFC 5545、CalDAV 协议以及 iCalendar 数据格式的基础概念、核心组件和用法。

### RFC 5545（iCalendar）与 CalDAV 的关系

- **RFC 5545** 定义了日历数据的“内容格式”（iCalendar/.ics），包括事件（VEVENT）、待办（VTODO）、日志（VJOURNAL）和忙闲（VFREEBUY）等核心组件。
- **CalDAV（RFC 4791）** 定义了如何通过网络“传输和管理”这些数据，基于 HTTP/WebDAV 协议，支持 CRUD 操作、高级查询和调度扩展。
- 两者的关系可以理解为：RFC 5545 规定“日历数据长什么样”，CalDAV 规定“如何存取这些数据”。

### iCalendar 格式的核心结构

- **层级结构**：一个 `.ics` 文件以 `VCALENDAR` 为顶层容器，内部可包含 `VEVENT`、`VTODO`、`VTIMEZONE` 等组件，每个组件由多个属性（如 `DTSTART`、`SUMMARY`）定义。
- **核心属性**：包括 `DTSTART`/`DTEND`（开始/结束）、`SUMMARY`（标题）、`UID`（全局唯一标识）、`RRULE`（重复规则）、`EXDATE`（排除日期）等。
- **时区处理**：支持三种时间表示方式——UTC 时间（带 `Z` 后缀）、本地时间（无时区信息）、带 `TZID` 引用的本地时间。

## 2. 重复规则（RRULE）详解

> 用户希望深入理解 RRULE 的语法、求值逻辑以及如何用它表达复杂的重复模式。

### RRULE 的语法与规则部分

- **基本结构**：由 `FREQ`（必须）、`UNTIL`/`COUNT`（互斥的结束条件）以及多个可选的 `BY*` 规则部分组成，用分号分隔，例如 `FREQ=WEEKLY;BYDAY=MO,WE;COUNT=10`。
- **常用规则部分**：`INTERVAL`（间隔）、`WKST`（周起始日）、`BYMONTH`、`BYWEEKNO`、`BYYEARDAY`、`BYMONTHDAY`、`BYDAY`（支持数字表示第几个，如 `2TU`）、`BYSETPOS`（位置筛选）等。

### 求值顺序与注意事项

- **筛选顺序**：按照 `BYMONTH` → `BYWEEKNO` → `BYYEARDAY` → `BYMONTHDAY` → `BYDAY` → `BYHOUR` → ... → `BYSETPOS` 的顺序对候选日期进行过滤。
- **重要规则**：`RRULE` 的起点由 `DTSTART` 决定；若无 `UNTIL` 或 `COUNT` 则视为无限重复；`EXDATE` 用于排除特定实例，优先级最高。

### 具体场景的 RRULE 写法

- **每周周二和周四**：`RRULE:FREQ=WEEKLY;BYDAY=TU,TH`
- **每月的最后一天**：`RRULE:FREQ=MONTHLY;BYMONTHDAY=-1`（`-1` 表示倒数第一天，自适应各月天数）
- **每月的最后一个周五**：`RRULE:FREQ=MONTHLY;BYDAY=-1FR`

## 3. Rust 生态中的日历与时间处理库

> 用户询问 Rust 中处理 iCalendar 格式、自然语言解析以及日期时间处理的库，并希望了解其依赖项的作用。

### iCalendar 处理库

- **`icalendar`**：提供强类型的 Builder 和 Parser，适合完整读写 `.ics` 文件。
- **`ics`**：侧重于生成 `.ics` 文件，使用方便。
- **`ical-rs`**：侧重解析，支持 iCalendar 和 vCard，但不验证字段有效性。
- **`vparser`**：底层、非验证性解析器，支持 `no_std` 环境。

### 自然语言转 RRULE 与日期解析

- **`text2rrule`**：唯一专注于将自然语言（如 `"every two weeks on friday"`）转换为标准 `RRULE` 字符串的库。
- **`chrono-english` / `nattydate`**：将自然语言日期时间（如 `"tomorrow at 3pm"`）解析为具体时刻，可与 `text2rrule` 组合使用。

### 依赖项（Cargo.toml）功能解析

- **`anyhow` / `thiserror`**：简易错误处理与自定义错误类型。
- **`chrono` + `chrono-tz`**：核心日期时间处理与时区支持。
- **`clap`**：命令行参数解析（支持 derive 宏）。
- **`crossterm` + `ratatui`**：终端控制与 TUI 界面构建（可选）。
- **`serde` + `serde_json`/`toml`**：序列化与配置管理。
- **`uuid`**：生成全局唯一标识符（用于 `UID` 属性）。

### chrono 库核心功能

- **核心类型**：`DateTime<Tz>`（带时区）、`NaiveDateTime`（无时区）、`NaiveDate`、`NaiveTime`、`Duration`。
- **主要功能**：获取当前时间、安全构建日期时间、与 `Duration` 进行运算、支持 RFC 3339 和 ISO 8601 格式解析与自定义格式化（`strftime`）。
- **时区与 Features**：提供 `Utc` 和 `Local`，支持通过 `chrono-tz` 扩展 IANA 时区；可选 features 包括 `serde`、`clock` 等。

## 4. Radicale 与 iOS 的周期任务兼容性分析

> 用户关注 Radicale（CalDAV 服务器）与 iOS 客户端在处理周期任务（VEVENT 和 VTODO）时的兼容性、存储策略和已知问题。

### 两者的处理哲学

- **Radicale**：遵循“存储规则，按需计算”的原则，在服务器端存储带有 `RRULE` 的主事件，通常不主动展开周期事件（对 `expand` 支持有限）。
- **iOS**：采用“展示实例，本地智能”的策略，在本地展开 `RRULE` 并显示所有实例，减轻服务器负担。

### 兼容性现状

- **基础使用可行**：对于标准的 `VEVENT` 和 `VTODO`，两者协同工作基本顺畅。
- **已知缺陷**：
  - Radicale 在处理 `expand` 查询时**不支持覆盖实例（`RECURRENCE-ID`）**，可能导致 iOS 无法正确显示修改后的单次实例。
  - 复杂查询（如时间范围 + 覆盖实例）可能返回冲突数据（同时返回旧实例和新覆盖实例）。
  - 修改“所有未来项”可能因 `RRULESET` 处理 bug 导致数据损坏或丢失。

## 5. 周期任务的修改、删除与完成机制

> 用户希望了解在 iCalendar 标准下，对周期任务的单次实例进行修改、删除或标记完成时，底层 ICS 数据如何变化，以及 iOS 和 Radicale 的处理差异。

### 修改“仅此一项”（`.ThisEvent`）

- **标准做法**：保留主事件（`RRULE` 不变），新增一个独立的“覆盖实例”，通过 `RECURRENCE-ID` 指向被修改的原始实例。
- **ICS 变化**：主事件不变，新增 `VEVENT`/`VTODO` 组件，包含 `RECURRENCE-ID` 和修改后的属性。
- **iOS 实现**：对应 `EKSpan.thisEvent`，创建覆盖实例。

### 修改“所有未来项”（`.FutureEvents`）

- **标准做法**：将原事件“一分为二”——旧序列增加 `UNTIL` 截断在修改点之前，新序列从修改点开始作为独立事件（新 `UID`）。
- **ICS 变化**：原事件添加 `UNTIL`，新建一个完整的事件（含新的 `RRULE`），通过 `RELATED-TO` 可选关联。
- **iOS 实现**：对应 `EKSpan.futureEvents`，创建新序列。

### 删除特定实例

- **标准做法**：在主事件的 `EXDATE` 属性中列出要删除的日期时间。
- **ICS 变化**：主事件的 `RRULE` 不变，仅增加 `EXDATE` 行。
- **兼容性问题**：Radicale 在多客户端同步时可能存在缺陷，导致删除操作无法被其他客户端识别，甚至数据被恢复。

### 标记 VTODO 为“完成”

- **方式一（创建例外）**：为当前实例创建覆盖实例，设置 `STATUS:COMPLETED` 和 `COMPLETED` 时间戳，主任务序列保持不变。
- **方式二（滚动到下一次）**：更新主任务的 `DTSTART`/`DUE` 为下一次时间，相当于“跳”到下一个周期。
- **兼容性**：不同客户端（如 iOS 提醒事项、Evolution）和服务器（Radicale、Nextcloud）处理方式可能存在差异，可能导致同步问题。

## 6. ICS 格式细节与时间表示法

> 用户询问 ICS 中日期时间格式的具体规范、`Z` 后缀的含义，以及 ISO 8601 持续时间（`P17D`）与 `RRULE` 的区别。

### iCalendar 日期时间格式

- **三种形式**：
  1.  **本地时间（浮动）**：`YYYYMMDDTHHMMSS`，不绑定时区，如 `DTSTART:20260828T155959`。
  2.  **UTC 时间（绝对）**：`YYYYMMDDTHHMMSSZ`，末尾带 `Z`，如 `DTSTART:20260828T155959Z`。
  3.  **带时区引用的本地时间**：`;TZID=Asia/Shanghai:YYYYMMDDTHHMMSS`，明确指定时区。
- **严格规则**：**禁止使用 UTC 偏移量**（如 `+08:00`）；秒为必需；推荐使用基本格式（无分隔符）。

### `Z` 后缀的含义

- **带 `Z`**：表示 UTC 时间，是一个绝对、固定的全球时间点，不受时区影响。适用于 `DTSTAMP`、`COMPLETED` 等元数据。
- **不带 `Z`（且无 `TZID`）**：表示本地时间（浮动时间），随用户时区变化，适合闹钟等场景，但在跨时区事件中容易造成误解。
- **最佳实践**：对于记录精确时刻的属性，必须使用带 `Z` 的 UTC 时间；对于事件时间，建议使用 `TZID` + 不带 `Z` 或直接使用 UTC。

### ISO 8601 持续时间（`P17D`）与 RRULE 的区别

- **`P17D`（ISO 8601）**：表示“时间长度/间隔”，如 17 天，用于定义持续时间或周期性间隔（配合 `R` 前缀）。
- **`RRULE`（RFC 5545）**：表示“重复规律”，用于定义日历事件的重复模式，如每周二、周四。
- **结论**：“每周的周二和周四”应使用 `RRULE:FREQ=WEEKLY;BYDAY=TU,TH`，而不是 `P17D`。
