> Version: v1.2.0 ｜ Author: [Soonkeira](https://github.com/Soonkeira)

# Project Doc Generator · Full Documentation (English)

## 1. Project Positioning

**Project Doc Generator** (`project-doc-generator`) reverse-engineers a **queryable, traceable and maintainable project knowledge system** from the existing code structure, business logic, APIs, database, call relationships and tests.

**The knowledge layer is the core; the 7 standard documents are merely different presentation views of it.** The knowledge layer answers "how does the system work?", while the standard documents answer "how should the project be described by software engineering standards?" and take the knowledge layer as their source of truth. All 7 documents are kept — nothing is deleted and nothing is renamed.

`Source Code → Code and business analysis → Project Knowledge Layer → 7 standard documents`

A business question can be traced back to source code along this chain:

```
Business question → Business capability → Business rules → Entry point/API → Call chain → Data changes → External dependencies → Source files/symbols → Tests
```

Questions it answers include: "How is this feature implemented?", "Why can this not be deleted?", "What happens after an order is cancelled?", "Why did this state change?", "Which data does this API ultimately modify?", "Where in the code is this business rule?".

> The questions above, and the business terms inside them, are **Example (placeholder, not project fact)** — they only illustrate query scenarios and do not represent the real business rules of any specific project.

## 2. Core Capabilities

| Capability | Description |
|------------|-------------|
| Project Knowledge Map | Indexes all business capabilities by business domain so a single file leads to the project's main business logic; includes the verification baseline and a common business question index |
| Reverse Business Logic Analysis | Organizes conclusions by business question (not by code file), reverse-engineering business capabilities, trigger entry points, business rules, key branches and failure paths |
| Source Evidence Tracing | Every important conclusion traces back to source files, symbols, APIs and tables, tagged with an evidence status |
| Business Call Chains | Records the full chain entry point → application service → domain object → repository → external dependency |
| Data Impact Analysis | Shows which tables, fields, caches, messages and files a piece of business logic ultimately changes |
| Relationship Mapping | Maps the dependencies among APIs, the database, MQ and external services |
| Git Verification Status | Records the verification baseline and each entry's last verified version, and derives freshness from them |
| 7 Standard Documents | A standardized document suite generated with the knowledge layer as the source of truth, kept backward compatible |

## 3. Output Structure

By default both the knowledge layer and all 7 standard documents are generated into `Doc/<project-name>/` of the analyzed project (`<project-name>` and `<business-name>` are naming patterns, replaced with real names at generation time):

```
Doc/<project-name>/
├── cn/
│   ├── 00-项目知识地图.md          # Business capability index + verification baseline + common business question index
│   ├── business/
│   │   └── BL-001-<业务名称>.md    # Business logic entry (14 sections)
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

The knowledge layer consists of `00-Project Knowledge Map.md` and `business/BL-*.md`; the 7 standard documents take it as their source of truth. IDs and file names correspond one-to-one between the Chinese and the English output.

## 4. Business Logic Entry Structure

**Numbering rule**: `BL-` plus a three-digit number, allocated in segments by business domain — the first domain uses `BL-001`–`BL-009`, and every subsequent domain takes 10 numbers (`BL-010`–`BL-019`, `BL-020`–`BL-029`, and so on). Allocated segments are never reused or reordered, and the same business logic keeps the same number in both languages.

**Fixed sections**: every BL entry must contain the following 14 sections, in exactly this order and with exactly these titles — no additions, no renames and no reordering.

| # | Section | Requirement |
|---|---------|-------------|
| 1 | Business Description | Explain from a business perspective what problem the capability solves, who benefits from it and in which scenario it is used |
| 2 | Trigger Entry Points | Real entry points: HTTP API / RPC / CLI / scheduled job / MQ consumer / event handler / UI action / internal method call; write "Not Found" when there is no evidence |
| 3 | Preconditions | Real conditions that must hold before execution (data state, permissions, external system availability, configuration switches) |
| 4 | Business Rules | When execution is allowed or rejected, state-transition conditions, permission requirements, value and lifecycle limits |
| 5 | Core Execution Flow | The real call chain, listed layer by layer down to the data access layer; vague descriptions are not allowed |
| 6 | Key Branches | if / switch / state machine / exception branches that really change business behavior, and how each branch differs |
| 7 | Data Changes | Which tables and fields are modified, what data is created or deleted, how states change, whether a transaction is involved |
| 8 | External Impacts | MQ / Redis / third-party API / file system / cache / object storage, etc.; write "Not Found" when there are none |
| 9 | Exceptions and Failure Paths | Only verified exception conditions and their handling results |
| 10 | Source Evidence | A two-column table (type, source); types include Controller / Service / Domain / Repository / Model / table / configuration item, and the source is written as `path → symbol` |
| 11 | Related Tests | Real test files and test methods; write "No corresponding automated tests found." when there are none — tests are never invented |
| 12 | Related Business Logic | Related BL IDs and names (upstream triggers, downstream dependencies, shared data) |
| 13 | Evidence Status | Choose exactly one, see the table below |
| 14 | Last Verified Version | Git commit + verification date (per-entry baseline); write "Not Found (no Git baseline)" when there is no Git |

**Two mutually independent fields** that must never be conflated:

| Field | Determined by | Values |
|-------|---------------|--------|
| Evidence status | Model judgement | `Verified` / `Partially Verified` / `Inferred` / `Not Found` |
| Freshness status | Mechanical Git diff check | `Current` / `⚠ Possibly Stale` |

`Verified` means "the implementation was read at the time" — not that the content is still fresh; `⚠ Possibly Stale` means "the code has changed" — not that the original evidence was wrong.

## 5. Operating Modes

| Mode | How to trigger | Flow |
|------|----------------|------|
| Generation mode | `$project-doc-generator 生成当前项目的完整项目文档`, or say "Generate project documents" / "Create project documents" | Analyze code → extract business knowledge → fetch version info → build the knowledge layer → generate the 7 documents with the knowledge layer as the source of truth → post-generation validation |
| Query mode | Just ask a business question | Read the knowledge map to locate the related BL entry → read that entry → answer along its call chain, business rules, data changes and source evidence; if the entry is marked `⚠ Possibly Stale`, re-check the source code before answering |

In query mode, if a BL entry is missing or cannot be verified, the tool states explicitly that the document is stale and that the source code prevails, then updates that entry.

## 6. Git Incremental Maintenance and Staleness Detection

1. **First run**: scan the whole codebase, build the knowledge map and all BL entries, then take the current commit with `git rev-parse HEAD` and write it into the "Verification Baseline" of the knowledge map as `last_verified_commit`, together with the verification date.
2. **Later runs**: read the existing `last_verified_commit`, run `git diff --name-status <last_verified_commit>..HEAD` to obtain the changed file set, and intersect it with the files listed under each BL entry's "Source Evidence".
3. **Re-verify only the affected entries**: the document set is not rewritten as a whole; unaffected entries stay as they are (no content rewrite, no renumbering). Each BL entry also has its own "Last Verified Version" commit.
4. **Staleness rule**: if a file listed under an entry's "Source Evidence" was modified between that entry's "Last Verified Version" commit and the current HEAD, and the entry was not re-verified in this run, its freshness becomes `⚠ Possibly Stale`; otherwise it stays `Current`.
5. **Baseline update**: only after every affected entry has been re-verified is the knowledge map's `last_verified_commit` advanced to the current HEAD.
6. **Degradation**: without Git or a valid `HEAD` no freshness determination is performed, freshness is recorded as `Not Found (no Git baseline)`, and the report states that every run needs a full re-check; when the baseline commit is unreachable (shallow clone, rebase, force-push) the run degrades to a full re-check with the reason stated. **Commit hashes are never fabricated.**

## 7. Usage

- Direct invocation: `$project-doc-generator 生成当前项目的完整项目文档`
- Natural language triggers: "Generate project documents", "Create project documents"; asking a business question enters query mode
- The current workspace is analyzed by default; when the user provides a project path, that path wins
- You can specify the language (Chinese / English / both), document numbers (e.g. `01,05`), output directory, author, and whether to overwrite
- Without further specification, the knowledge layer plus all 7 standard documents are generated; you can also generate only the knowledge layer, only the standard documents, or a subset of document numbers
- When target documents already exist, the conflicting files are listed and confirmation is requested instead of overwriting silently

**Author field** — resolved through a four-level priority: (1) author explicitly provided by the user → (2) default author `Soonkeira` → (3) Git commit author (`git log -1 --format=%an`) → (4) leave empty. Using the machine account name (`$env:USERNAME` / `whoami`) as the author is strictly prohibited.

**Version numbers**: with Git, `1.{yy}.{Mdd}.{commit-count}` (e.g. `1.26.509.1234`); without Git, `1.{yy}.{Mdd}.{hmm}` (e.g. `1.26.509.1430`). Every document header contains the project name, author, date, version and a change log.

## 8. Boundaries and Safety Rules

| Rule | Content |
|------|---------|
| Directory boundaries | `.git/`, `node_modules/`, `vendor/`, `dist/`, `build/`, `coverage/`, `.venv/`, cache and build-output directories are skipped by default |
| Sensitive data | Secrets, tokens and certificates are never read or written; `.env*`, `*.pem`, `*.key`, `credentials*`, `secrets*` are skipped; documents mention configuration item names only, never values |
| Modification scope | Only the user-specified documentation output directory is modified — never source code, configuration, dependency lock files or Git history |
| No fabrication | Databases, APIs, tests or deployment details that were not found are written as "Not Found / Not Applicable"; tables, endpoints, test cases and runtime parameters are never invented |
| No inference dressed as fact | When only structural inference is possible it must be labelled `Status: Inferred`; **model inference must never be packaged as fact** |
| Example labelling | Examples in templates must be labelled "Example (placeholder, not project fact)" |
| Large codebases | For a large codebase the analysis scope is stated first, and entry points, configuration, routing, domain models, database migrations, tests and deployment files are read first |

## 9. Repository Structure

```
project-doc-generator/
├── README.md                   # Project navigation home page
├── SKILL.md                    # Skill workflow and rules
├── Doc/{cn,en}/README.md       # Chinese / English full documentation
└── references/{cn,en}/
    ├── document-templates.md        # The 7 standard document templates
    ├── business-logic-template.md   # Knowledge map and business logic entry templates
    └── git-version-info.md          # Version number generation and Git info reference
```

## 10. Version History

| Version | Changes |
|---------|---------|
| v1.2.0 | Added the project knowledge layer: project knowledge map, reverse business logic analysis, source evidence tracing, business call chains, data impact analysis, Git incremental maintenance and document staleness detection; the 7 standard documents remain backward compatible |
| v1.1.0 | Unified output paths and file names; added selective generation, overwrite confirmation, safety exclusions, source evidence and post-generation validation |

## 11. Reference Files

| File | Content |
|------|---------|
| [SKILL.md](../../SKILL.md) | Skill workflow and rules (single authority) |
| [references/en/document-templates.md](../../references/en/document-templates.md) | The 7 standard document templates |
| [references/en/business-logic-template.md](../../references/en/business-logic-template.md) | Knowledge map and business logic entry templates, status definitions and rules |
| [references/en/git-version-info.md](../../references/en/git-version-info.md) | Version number algorithm and Git info command reference |
| [references/cn/*](../../references/cn/) | Chinese versions of the templates and references above |
| [README.md](../../README.md) | Project navigation home page |
