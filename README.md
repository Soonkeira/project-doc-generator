# 项目文档生成器 / Project Doc Generator

<p align="center">
  <b>中文</b> &nbsp;|&nbsp;
  <a href="Doc/en/README.md">English</a>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/version-v1.2.0-blue.svg" alt="Version: v1.2.0">
  <img src="https://img.shields.io/badge/author-Soonkeira-orange.svg" alt="Author">
</p>

---

> 版本：**v1.2.0**

> 作者：[**Soonkeira**](https://github.com/Soonkeira)

## 版本记录

- **v1.2.0**：新增项目知识层——项目知识地图、业务逻辑反向分析、源码证据追踪、业务调用链、数据影响分析、Git 增量维护、文档过期检测；7 套标准文档保留兼容。
- **v1.1.0**：统一输出路径和文件名，增加选择性生成、覆盖确认、安全排除、来源证据和生成后校验。

> **中文**：自动解析项目代码结构与业务逻辑，一键批量生成需求规格、概要设计、详细设计、数据库设计、API 文档、测试计划、部署手册等全套标准化项目文档；并从代码反推出可查询、可追溯、可持续维护的项目知识体系（项目知识地图 + 业务逻辑条目），7 套标准文档是知识层的展示视图。

> **English**: Automatically analyze project code structure and business logic, then generate a complete suite of standardized project documents — requirements specification, architecture design, detailed design, database design, API docs, test plan, and deployment guide — all in one click. It also reverse-engineers a queryable, traceable, and maintainable project knowledge system (Project Knowledge Map + Business Logic entries), with the 7 standard documents serving as presentation views of that knowledge layer.

---

## 📖 文档导航 / Documentation

| 语言 / Language | 链接 / Link |
|:---:|:---|
| 🇨🇳 中文文档 | [./Doc/cn/README.md](https://github.com/Soonkeira/project-doc-generator/blob/main/Doc/cn/README.md) |
| 🇺🇸 English Docs | [./Doc/en/README.md](https://github.com/Soonkeira/project-doc-generator/blob/main/Doc/en/README.md) |

---

## 🚀 快速开始 / Quick Start

<details open>
<summary><b>中文</b></summary>

1. 在对话中上传项目代码或告知项目目录路径
2. 输入 **`$project-doc-generator`**，再说明要生成的语言、文档编号和输出目录；也可以直接输入 **"形成项目文档"** 或 **"生成项目文档"**
3. Skill 自动分析代码 → 获取版本信息 → 生成项目知识层（`00-项目知识地图.md` + `business/BL-*.md`）→ 以知识层为事实来源生成 7 套标准文档
4. 默认输出至 `Doc/<项目名>/cn/` 和 `Doc/<项目名>/en/`；已有文件时需先确认是否覆盖
5. **查询模式**：直接提问业务问题，例如"订单取消以后做了什么？"，Skill 会读取 `00-项目知识地图.md` 定位相关 BL 条目，再顺着其中的业务规则、调用链、数据变化和源码证据反查到具体源码位置；若该条目已标记 `⚠ 可能过期`，会先回源码核对再作答

</details>

<details>
<summary><b>English</b></summary>

1. Upload your project code or provide the project directory path in the conversation
2. Enter **`$project-doc-generator`** and specify the language, document numbers, and output directory; you can also say **"Generate project documents"** or **"Create project documents"**
3. The Skill auto-analyzes code → fetches version info → builds the project knowledge layer (`00-Project Knowledge Map.md` + `business/BL-*.md`) → generates the 7 standard documents using the knowledge layer as the source of truth
4. By default, documents are output to `Doc/<project-name>/cn/` and `Doc/<project-name>/en/`; existing files require overwrite confirmation
5. **Query mode**: just ask a business question, e.g. "What happens after an order is cancelled?". The Skill reads `00-Project Knowledge Map.md`, locates the related BL entry, and traces the business rules, call chain, data changes, and source evidence back to the actual source code; if that entry is marked `⚠ possibly stale`, it re-checks the source code before answering

</details>

---

## 🧩 核心能力 / Core Capabilities

### 🧠 项目知识层 / Project Knowledge Layer

| 能力 Capability | 说明 Description |
|:---|:---|
| 🗺️ 项目知识地图 / Project Knowledge Map | 按业务域索引全部业务能力，附验证基线与常见业务问题索引 / Indexes all business capabilities by domain, with a verification baseline and a common business question index |
| 🔍 业务逻辑反向分析 / Reverse Business Logic Analysis | 从源码反推业务能力、触发入口、业务规则、关键分支与异常路径 / Reverse-engineers business capabilities, entry points, business rules, key branches, and failure paths from source code |
| 🔗 源码证据追踪 / Source Evidence Tracing | 每条业务结论都能反查到源码文件与符号，并标注证据状态 / Every business conclusion traces back to source files and symbols, tagged with an evidence status |
| ⛓️ 业务调用链 / Business Call Chains | 记录 入口 → 应用服务 → 领域对象 → 仓储 → 外部依赖 的完整链路 / Records the full chain from entry point to application service, domain object, repository, and external dependency |
| 💾 数据影响分析 / Data Impact Analysis | 说明一段业务逻辑最终改动了哪些表、字段、缓存、消息与文件 / Shows which tables, fields, caches, messages, and files a piece of business logic ultimately changes |
| 🔄 Git 增量维护 / Git-based Incremental Maintenance | 基于 Git diff 只重新验证受影响的业务条目，不全量重写 / Re-verifies only the affected business entries based on Git diff instead of rewriting everything |
| ⏰ 文档过期检测 / Document Staleness Detection | 源码证据文件在基线之后被改动且未重新验证时，标记 `⚠ 可能过期` / Marks entries as `⚠ possibly stale` when their evidence files changed after the baseline commit without re-verification |

### 📄 传统 7 套标准文档（兼容保留）/ 7 Standard Documents (Backward Compatible)

| 文档 / Document | 说明 Description |
|:---|:---|
| 01 需求规格说明书 / Requirements Specification | 功能概述、功能需求、业务规则、非功能需求 |
| 02 概要设计 / System Overview Design | 系统架构、模块划分、类图、核心流程 |
| 03 详细设计 / Detailed Design | 方法算法流程、分支逻辑、关键实现细节 |
| 04 数据库设计 / Database Design | 数据表结构、字段定义、表间关系、索引 |
| 05 API 文档 / API Documentation | 接口清单、入参出参、调用示例、异常场景 |
| 06 测试计划 / Test Plan | 单元测试用例、边界条件、并发测试 |
| 07 部署手册与用户手册 / Deployment & User Manual | 环境要求、配置项、部署步骤、操作指南、FAQ |

> 文件名、编号与输出路径与 v1.1.0 完全一致，已有项目可无缝升级。

---

## ✨ 核心特性 / Feature Highlights

| 特性 Feature | 说明 Description |
|:---|:---|
| 🧠 智能代码分析 / Intelligent Code Analysis | 自动识别项目目录结构、领域模型、业务逻辑与设计模式 / Auto-identifies project structure, domain models, business logic, and design patterns |
| 📄 7 套标准文档 / 7 Standard Documents | 覆盖需求 → 设计 → 数据库 → API → 测试 → 部署全生命周期 / Covers the full lifecycle from requirements through deployment |
| 🔢 自动版本管理 / Auto Versioning | 基于 Git 提交次数或时间戳生成规范版本号 / Generates standardized version numbers based on Git commit count or timestamp |
| 📝 内置变更日志 / Built-in Changelog | 每份文档自动附带版本历史与变更记录 / Every document includes version history and change logs |
| 📁 自动归档目录 / Auto Archive Directory | 一键输出至 `Doc/` 专属目录，结构清晰 / One-click output to a dedicated `Doc/` directory with clear structure |

---

## 📂 文档产出清单 / Generated Document List

| # | 文档 / Document | 内容概要 / Content Summary |
|:---:|:---|:---|
| 00 | 项目知识地图 / Project Knowledge Map | 业务能力索引（按业务域分组）、验证基线、常见业务问题索引 |
| BL | 业务逻辑条目 / Business Logic Entries | 每条业务能力的 14 个章节：业务说明、触发入口、前置条件、业务规则、核心执行流程、关键分支、数据变化、外部影响、异常与失败路径、源码证据、相关测试、关联业务、证据状态、最后验证版本 |
| 01 | 需求规格说明书 / Requirements Specification | 功能概述、功能需求、业务规则、非功能需求 |
| 02 | 概要设计 / System Overview Design | 系统架构、模块划分、类图、核心流程 |
| 03 | 详细设计 / Detailed Design | 方法算法流程、分支逻辑、关键实现细节 |
| 04 | 数据库设计 / Database Design | 数据表结构、字段定义、表间关系、索引 |
| 05 | API 文档 / API Documentation | 接口清单、入参出参、调用示例、异常场景 |
| 06 | 测试计划 / Test Plan | 单元测试用例、边界条件、并发测试 |
| 07 | 部署手册与用户手册 / Deployment & User Manual | 环境要求、配置项、部署步骤、操作指南、FAQ |

> **分工关系 / Division of Labor**：知识层（`00-项目知识地图` + `business/BL-*.md`）回答"系统怎么工作"，传统 7 套文档回答"按软件工程标准应如何描述"。传统文档以知识层为事实来源，两者冲突时以知识层中已验证的源码证据为准。
>
> The knowledge layer (`00-Project Knowledge Map` + `business/BL-*.md`) answers "how the system works", while the 7 standard documents answer "how the project should be described by software engineering standards". The standard documents take the knowledge layer as their source of truth.

---

## 📁 项目结构 / Project Structure

**Skill 仓库 / This repository**

```
project-doc-generator/
├── README.md                   # 项目导航首页 (你在这里)
├── SKILL.md                    # Skill 工作流与规则定义
├── LICENSE                     # MIT License
├── Doc/
│   ├── cn/README.md            # 中文完整文档
│   └── en/README.md            # English full documentation
└── references/
    ├── cn/
    │   ├── document-templates.md
    │   ├── business-logic-template.md
    │   └── git-version-info.md
    └── en/
        ├── document-templates.md
        ├── business-logic-template.md
        └── git-version-info.md
```

**生成到被分析项目中的产出 / Output generated into the analyzed project**

```
Doc/<项目名>/
├── cn/
│   ├── 00-项目知识地图.md          # 知识层：业务能力索引 + 验证基线
│   ├── business/
│   │   └── BL-001-<业务名称>.md    # 知识层：业务逻辑条目（14 章节）
│   ├── 01-需求规格.md
│   ├── 02-概要设计.md
│   ├── 03-详细设计.md
│   ├── 04-数据库设计.md
│   ├── 05-API文档.md
│   ├── 06-测试计划.md
│   └── 07-部署手册与用户手册.md
└── en/
    ├── 00-Project Knowledge Map.md
    ├── business/
    │   └── BL-001-<business-name>.md
    ├── 01-Requirement Specification.md
    ├── 02-Overview Design.md
    ├── 03-Detailed Design.md
    ├── 04-Database Design.md
    ├── 05-API Documentation.md
    ├── 06-Test Plan.md
    └── 07-Deployment & User Manual.md
```

---

## 👤 作者 / Author

**[Soonkeira](https://github.com/Soonkeira)**
