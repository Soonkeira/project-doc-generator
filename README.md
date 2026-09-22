# 项目文档生成器 / Project Doc Generator

<p align="center">
  <b>中文</b> &nbsp;|&nbsp;
  <a href="Doc/en/README.md">English</a>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/version-v1.4.0-blue.svg" alt="Version: v1.4.0">
  <img src="https://img.shields.io/badge/author-Soonkeira-orange.svg" alt="Author">
</p>

---

> 版本：**v1.4.0**

> 作者：[**Soonkeira**](https://github.com/Soonkeira)

## 版本记录

- **v1.4.0**：修正 v1.3.0 的规则矛盾与一致性漏洞，共六项。① **编号规则改为「历史最大编号 + 1」**，删除的编号**永久 tombstone、永不复用**（**允许因删除产生跳号**，跳号不是错误），并在知识地图新增**「编号登记（ID Registry）」**章节记录当前最大编号 / 下一个可用编号 / 已废弃编号——v1.3.0 的旧取号口径等于复用已废弃编号，已整段替换；② **验证基线只在工作区干净时推进**（`git status --porcelain` 无输出 = `Clean`），工作区 `Dirty` 时可以分析、可以回答、可以更新 BL 内容，但**不得推进 `last_verified_commit`、不得标记 `最新`**，该条时效状态记 **`临时工作区分析`**——否则未提交代码被 `git restore` 丢弃后，BL 描述的是一份已不存在的实现却被系统当作「最新」；时效状态明确为**四值**（新增 `临时工作区分析`）；③ **查询模式的「只读」= 不修改文件，不是不读源码**：允许读取源码、`git diff` 与相关测试做即时核实，**不得因为文档过期就要求用户先去跑同步**；④ **源码索引扩为六张反向索引表**，新增「外部依赖/资源 → BL」与「测试 → BL」；⑤ 新增**「BL 粒度判定」**规则（BL = 用户/业务可感知的业务能力或用例，CRUD 不默认一接口一个 BL，工具函数不独立成条），保证不同模型、不同时间生成的规模稳定；⑥ 知识地图「验证基线」新增 **`traditional_docs_status`** 与 **`traditional_docs_generated_from_commit`**，用于判断 7 套传统文档是否已落后于知识层。
- **v1.3.0**：解决知识层三个核心问题——**稳定 ID、双向索引、精确失效判定**。① BL 编号改为按首次建立顺序简单递增（`BL-001`、`BL-002`…），取消按业务域分段分配，业务域降级为条目 metadata，重新分类不再改动 ID；② 新增源码索引 `00-源码索引.md`，形成 Source ↔ BL 双向索引（原来只有 BL → Source 单向）；③ 变更检测覆盖未提交修改（`git status --porcelain`），并新增路径/符号/数据/接口/配置/测试/重命名删除多层变更影响分析，失效判定更准确。同时：BL 条目新增「时效状态」章节，固定章节增至 15 个；证据状态与时效状态拆为两个独立字段分别显示；运行模式由「生成 / 查询」拆为「生成 / 查询 / 同步」，查询模式默认只读、不写任何文件，新增同步模式做增量维护。
- **v1.2.0**：新增项目知识层——项目知识地图、业务逻辑反向分析、源码证据追踪、业务调用链、数据影响分析、Git 增量维护、文档过期检测；7 套标准文档保留兼容。
- **v1.1.0**：统一输出路径和文件名，增加选择性生成、覆盖确认、安全排除、来源证据和生成后校验。

> **中文**：自动解析项目代码结构与业务逻辑，一键批量生成需求规格、概要设计、详细设计、数据库设计、API 文档、测试计划、部署手册等全套标准化项目文档；并从代码反推出可查询、可追溯、可持续维护的项目知识体系（项目知识地图 + 源码索引 + 业务逻辑条目），7 套标准文档是知识层的展示视图。

> **English**: Automatically analyze project code structure and business logic, then generate a complete suite of standardized project documents — requirements specification, architecture design, detailed design, database design, API docs, test plan, and deployment guide — all in one click. It also reverse-engineers a queryable, traceable, and maintainable project knowledge system (Project Knowledge Map + Source Index + Business Logic entries), with the 7 standard documents serving as presentation views of that knowledge layer.

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
3. Skill 自动分析代码 → 获取版本信息 → 生成项目知识层（`00-项目知识地图.md` + `00-源码索引.md` + `business/BL-*.md`）→ 以知识层为事实来源生成 7 套标准文档
4. 默认输出至 `Doc/<项目名>/cn/` 和 `Doc/<项目名>/en/`；已有文件时需先确认是否覆盖
5. **三种运行模式**：
   - **生成模式**：全量扫描 → 建立知识层 + 7 套传统文档（写入）
   - **查询模式（默认只读）**：直接提问业务问题，例如"订单取消以后做了什么？"，Skill 读取 `00-项目知识地图.md`、`00-源码索引.md` 与 BL 条目并作答，**不写任何文件**（「只读」指不修改文件，不是不读源码）；遇到条目缺失或可能过期时会直接读源码、`git diff` 与相关测试即时核实再作答，**不需要先跑同步**；只有用户明确要求更新文档时才转入同步模式
   - **同步模式**：增量维护——变更影响分析 → 只重新验证受影响的条目 → 刷新时效状态 → 更新源码索引 → 推进验证基线；只写知识层，不改 7 套传统文档

</details>

<details>
<summary><b>English</b></summary>

1. Upload your project code or provide the project directory path in the conversation
2. Enter **`$project-doc-generator`** and specify the language, document numbers, and output directory; you can also say **"Generate project documents"** or **"Create project documents"**
3. The Skill auto-analyzes code → fetches version info → builds the project knowledge layer (`00-Project Knowledge Map.md` + `00-Source Index.md` + `business/BL-*.md`) → generates the 7 standard documents using the knowledge layer as the source of truth
4. By default, documents are output to `Doc/<project-name>/cn/` and `Doc/<project-name>/en/`; existing files require overwrite confirmation
5. **Three operating modes**:
   - **Generation mode**: full scan → build the knowledge layer + the 7 standard documents (writes files)
   - **Query mode (read-only by default)**: just ask a business question, e.g. "What happens after an order is cancelled?". The Skill reads `00-Project Knowledge Map.md`, `00-Source Index.md` and the BL entries to answer and **writes no files** ("read-only" means modifying no files, not "not reading source"); when an entry is missing or possibly stale it reads the source code, `git diff` and related tests for immediate verification before answering, so **running a sync first is never required**; it only switches to sync mode when the user explicitly asks for an update
   - **Sync mode**: incremental maintenance — change impact analysis → re-verify only the affected entries → refresh freshness status → update the source index → advance the verification baseline; it writes the knowledge layer only and never touches the 7 standard documents

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
| 🔄 Git 增量维护 / Git-based Incremental Maintenance | 知识地图记录验证基线（`last_verified_commit`），每条 BL 条目另记自己的最后验证版本；按多层变更影响分析（路径/符号/数据/接口/配置/测试/重命名与删除）只重新验证受影响的业务条目，不全量重写 / The knowledge map records the verification baseline (`last_verified_commit`) and each BL entry keeps its own last verified version; a multi-layer change impact analysis (paths / symbols / data / interfaces / configuration / tests / renames and deletions) re-verifies only the affected entries instead of rewriting everything |
| 🔁 源码反向索引 / Source-to-BL Reverse Index | 从源码反查业务逻辑：源码文件与符号 → BL、数据表/迁移 → BL、API/入口 → BL、配置项 → BL、外部依赖/资源 → BL、测试 → BL **六张反向索引表**（`00-源码索引.md`）/ Looks business logic up from source code: **six reverse index tables** (`00-Source Index.md`) for source files and symbols → BL, tables/migrations → BL, APIs/entry points → BL, configuration items → BL, external dependencies/resources → BL, and tests → BL |
| 🔗 双向索引一致性 / Bidirectional Index Consistency | 源码索引由各 BL 条目的源码证据、数据变化、触发入口、外部影响与相关测试汇总而来，必须与 BL 条目保持一致；不一致视为校验失败 / The source index is aggregated from each BL entry's source evidence, data changes, trigger entry points, external impacts and related tests and must stay consistent with the BL entries; any mismatch counts as a validation failure |
| 🔎 只读查询核实 / Read-only Query Verification | 查询模式的「只读」指**不修改任何文件**，不是不读源码：允许读取源码、`git diff` 与相关测试即时核实，条目缺失或可能过期时直接核实后作答，**不要求用户先跑同步** / Query mode's "read-only" means **modifying no files**, not "not reading source": source code, `git diff` and related tests may be read for immediate verification, and a missing or possibly stale entry is verified on the spot instead of asking the user to run a sync first |
| ⏰ 文档过期检测 / Document Staleness Detection | 源码证据文件在基线之后被改动（**含未提交修改**）且未重新验证时，标记 `⚠ 可能过期`；工作区 `Dirty` 时本次分析记 **`临时工作区分析`** 且**不推进正式基线** / Marks entries as `⚠ possibly stale` when their evidence files changed after the baseline — **including uncommitted changes** — without re-verification; while the working tree is `Dirty` the run is recorded as **`Provisional Working-Tree Analysis`** and the formal baseline is not advanced |

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
| 00b | 源码索引 / Source Index | 源码 → BL 反向索引：源码文件与符号、数据表/迁移、API/入口、配置项、外部依赖/资源、测试**六张反向索引表**，另含未覆盖的源码文件、覆盖范围与限制、维护规则 |
| BL | 业务逻辑条目 / Business Logic Entries | 每条业务能力的 15 个章节：业务说明、触发入口、前置条件、业务规则、核心执行流程、关键分支、数据变化、外部影响、异常与失败路径、源码证据、相关测试、关联业务、证据状态、时效状态、最后验证版本 |
| 01 | 需求规格说明书 / Requirements Specification | 功能概述、功能需求、业务规则、非功能需求 |
| 02 | 概要设计 / System Overview Design | 系统架构、模块划分、类图、核心流程 |
| 03 | 详细设计 / Detailed Design | 方法算法流程、分支逻辑、关键实现细节 |
| 04 | 数据库设计 / Database Design | 数据表结构、字段定义、表间关系、索引 |
| 05 | API 文档 / API Documentation | 接口清单、入参出参、调用示例、异常场景 |
| 06 | 测试计划 / Test Plan | 单元测试用例、边界条件、并发测试 |
| 07 | 部署手册与用户手册 / Deployment & User Manual | 环境要求、配置项、部署步骤、操作指南、FAQ |

> **分工关系 / Division of Labor**：知识层（`00-项目知识地图` + `00-源码索引` + `business/BL-*.md`）回答"系统怎么工作"，传统 7 套文档回答"按软件工程标准应如何描述"。知识地图与源码索引构成 BL ↔ 源码的双向索引：知识地图从业务域索引到 BL，源码索引从源码反查到 BL。传统文档以知识层为事实来源，两者冲突时以知识层中已验证的源码证据为准。
>
> The knowledge layer (`00-Project Knowledge Map` + `00-Source Index` + `business/BL-*.md`) answers "how the system works", while the 7 standard documents answer "how the project should be described by software engineering standards". The knowledge map and the source index together form a bidirectional index between BL entries and source code: the knowledge map indexes from business domain down to BL entries, while the source index looks business logic up from the source code. The standard documents take the knowledge layer as their source of truth.

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
│   ├── 00-源码索引.md              # 知识层：源码 → BL 反向索引
│   ├── business/
│   │   └── BL-001-<业务名称>.md    # 知识层：业务逻辑条目（15 章节）
│   ├── 01-需求规格.md
│   ├── 02-概要设计.md
│   ├── 03-详细设计.md
│   ├── 04-数据库设计.md
│   ├── 05-API文档.md
│   ├── 06-测试计划.md
│   └── 07-部署手册与用户手册.md
└── en/
    ├── 00-Project Knowledge Map.md
    ├── 00-Source Index.md          # Knowledge layer: source → BL reverse index
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
