> Version: v1.3.0 ｜ Author: [Soonkeira](https://github.com/Soonkeira)

# Project Doc Generator · Full Documentation (English)

## 1. Project Positioning

**Project Doc Generator** (`project-doc-generator`) reverse-engineers a **queryable, traceable and maintainable project knowledge system** from the existing code structure, business logic, APIs, database, call relationships and tests.

**The knowledge layer is the core; the 7 standard documents are merely different presentation views of it.** The knowledge layer answers "how does the system work?", while the standard documents answer "how should the project be described by software engineering standards?" and take the knowledge layer as their source of truth. All 7 documents are kept — nothing is deleted and nothing is renamed.

The knowledge layer consists of three parts: the **Project Knowledge Map**, the **Source Index** and the **BL business logic entries**. The knowledge map indexes from business domain down to BL entries, while the source index looks BL entries up from the source code; together they form a **bidirectional index** between BL entries and source code. The knowledge layer can therefore answer both "how is this business capability implemented?" and "which business does this piece of code belong to?".

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
| Source-to-BL Reverse Index | Looks business logic up from source code (previously only BL → Source was possible): aggregates each BL entry's source evidence, data changes, trigger entry points and external impacts into four reverse index tables — source files and symbols → BL, tables/migrations → BL, APIs/entry points → BL, configuration items → BL |
| Bidirectional Index Consistency | The source index must stay consistent with the BL entries; any mismatch counts as a validation failure and must be fixed before the verification baseline can advance |
| Relationship Mapping | Maps the dependencies among APIs, the database, MQ and external services |
| Git Verification Status | Records the knowledge map's verification baseline and each BL entry's last verified version; freshness status is determined mechanically by Git and is a field independent from evidence status |
| 7 Standard Documents | A standardized document suite generated with the knowledge layer as the source of truth, kept backward compatible |

## 3. Output Structure

By default both the knowledge layer and all 7 standard documents are generated into `Doc/<project-name>/` of the analyzed project (`<project-name>` and `<business-name>` are naming patterns, replaced with real names at generation time):

```
Doc/<project-name>/
├── cn/
│   ├── 00-项目知识地图.md          # Business capability index + verification baseline + common business question index
│   ├── 00-源码索引.md              # Source → BL reverse index
│   ├── business/
│   │   └── BL-001-<业务名称>.md    # Business logic entry (15 sections)
│   ├── 01-需求规格.md
│   ├── 02-概要设计.md
│   ├── 03-详细设计.md
│   ├── 04-数据库设计.md
│   ├── 05-API文档.md
│   ├── 06-测试计划.md
│   └── 07-部署手册与用户手册.md
└── en/
    ├── 00-Project Knowledge Map.md
    ├── 00-Source Index.md          # Source → BL reverse index
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

The knowledge layer consists of `00-Project Knowledge Map.md`, `00-Source Index.md` and `business/BL-*.md`; **the source index must stay consistent with the BL entries**, and any mismatch counts as a validation failure. IDs and file names correspond one-to-one between the Chinese and the English output.

## 4. Business Logic Entry Structure

**Numbering rule**: `BL-` plus a three-digit number, incremented simply in the order entries are first created: `BL-001`, `BL-002`, `BL-003`, … — numbers are no longer allocated in segments by business domain. Numbers are never reused and never reordered; when an entry is deleted its number is retired, and a new entry takes the smallest number currently unused. The same business logic keeps the same number in both languages.

**Business domain is metadata**: the domain is written in the entry header field `**Business Domain**: <business-domain-name>`. Reclassifying domains changes only that field and **never changes an ID**. The knowledge map still groups entries by business domain, but group headings **must not carry number ranges**.

**Fixed sections**: every BL entry must contain the following 15 sections, in exactly this order and with exactly these titles — no additions, no renames and no reordering.

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
| 14 | Freshness Status | Choose exactly one of three, see the table below; determined mechanically by Git and independent from evidence status |
| 15 | Last Verified Version | Git commit + verification date (per-entry baseline); write "Not Found (no Git baseline)" when there is no Git |

**Two mutually independent fields** that must never be conflated or merged into a single display:

| Field | Determined by | Values |
|-------|---------------|--------|
| Evidence status | Model judgement (one of four) | `Verified` / `Partially Verified` / `Inferred` / `Not Found` |
| Freshness status | Mechanical Git check (one of three) | `Current` / `⚠ Possibly Stale` / `Not Found (no Git baseline)` |

The knowledge map index table always shows them in **two separate columns**, in the order `| ID | Business Capability | Description | Evidence Status | Freshness Status |`; the freshness marker must never be prefixed to the evidence status, and the two must never be merged into a single display in any form.

`Verified` means "the implementation was read at the time" — not that the content is still fresh; `⚠ Possibly Stale` means "the code has changed" — not that the original evidence was wrong.

## 5. Operating Modes

| Mode | How to trigger | Flow |
|------|----------------|------|
| Generation mode | `$project-doc-generator 生成当前项目的完整项目文档`, or say "Generate project documents" / "Create project documents" | Full scan of the code → extract business knowledge → fetch version info → build the knowledge layer (knowledge map + source index + BL entries) → generate the 7 standard documents with the knowledge layer as the source of truth (writes files) → post-generation validation |
| Query mode (read-only by default) | Just ask a business question | Reads only the knowledge map, the source index and the BL entries, locates the related entry, and answers along its call chain, business rules, data changes and source evidence — **writes no files**; when an entry is missing or possibly stale it states explicitly that "the source code prevails" and suggests running sync mode |
| Sync mode (incremental maintenance) | Ask for an incremental sync, e.g. "Sync project documents" / "Sync the knowledge layer" | Change impact analysis → re-verify only the affected entries → refresh freshness status → update the source index → advance the verification baseline; writes the knowledge layer only and never touches the 7 standard documents |

Query mode is read-only by default and never rewrites a file just because a question was asked. Sync mode maintains the knowledge layer only (knowledge map, source index and BL entries); the 7 standard documents are refreshed only by running generation mode again.

## 6. Git Incremental Maintenance and Staleness Detection

1. **First run**: scan the whole codebase, build the knowledge map, the source index and all BL entries, then take the current commit with `git rev-parse HEAD` and write it into the "Verification Baseline" of the knowledge map as `last_verified_commit`, together with the verification date.
2. **Later runs (changed file set)**: read the existing `last_verified_commit`; the changed file set is the **committed part ∪ the working-tree part**:
   - Committed part: `git diff --name-status -M <last_verified_commit>..HEAD`
   - Working-tree part: `git status --porcelain` (covers staged, unstaged and untracked `??` entries)

   **Looking only at `<last_verified_commit>..HEAD` is not enough**: code may already have been changed without being committed, and comparing commit ranges alone would miss stale entries.
3. **Change impact analysis (multi-layer)**: intersecting file paths alone is not enough. Each layer below is evaluated, and a hit on any layer marks the entry affected and records the basis:
   - Path layer: changed files ∩ the paths in the entry's "Source Evidence"
   - Symbol layer: changed symbols ∩ the symbols on the call chain
   - Data layer: table names / fields / migrations ∩ the entry's "Data Changes"
   - Interface layer: routes / entry-point signatures ∩ the entry's "Trigger Entry Points"
   - Configuration layer: configuration items ∩ the entry's "External Impacts" and "Preconditions"
   - Test layer: test files ∩ the entry's "Related Tests" → mark "test evidence pending re-check"
   - Rename / delete: a broken path must be repaired; a deleted evidence file requires re-verification
4. **Re-verify only the affected entries**: the document set is not rewritten as a whole; unaffected entries stay as they are (no content rewrite, no renumbering). Each BL entry also has its own "Last Verified Version" commit.
5. **Refresh the freshness status and the source index**: after the affected entries are re-verified their "Freshness Status" is updated, and changes to source evidence, data changes, trigger entry points and external impacts are propagated into `00-Source Index.md` so that the source index and the BL entries stay consistent.
6. **Staleness rule**: if a file listed under an entry's "Source Evidence" was modified after that entry's "Last Verified Version" commit (**including uncommitted changes**) and the entry was not re-verified in this run, its freshness becomes `⚠ Possibly Stale`; otherwise it stays `Current`.
7. **Baseline update**: only after every affected entry has been re-verified is the knowledge map's `last_verified_commit` advanced to the current HEAD.
8. **Record the "last incremental analysis"**: the knowledge map records the analysis time, the baseline commit, the number of changed files, the affected entries, the basis for each judgement and any items left unhandled.
9. **Degradation**: without Git or a valid `HEAD` no freshness determination is performed, freshness is recorded as `Not Found (no Git baseline)`, and the report states that every run needs a full re-check; when the baseline commit is unreachable (shallow clone, rebase, force-push) the run degrades to a full re-check with the reason stated. **Commit hashes are never fabricated.**

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
├── LICENSE                     # MIT License
├── Doc/{cn,en}/README.md       # Chinese / English full documentation
└── references/{cn,en}/
    ├── document-templates.md        # The 7 standard document templates
    ├── business-logic-template.md   # Knowledge map and business logic entry templates
    └── git-version-info.md          # Version number generation and Git info reference
```

## 10. Version History

| Version | Changes |
|---------|---------|
| v1.3.0 | Three core upgrades to the knowledge layer: **stable IDs** — BL numbering now increments simply in creation order, the business domain is demoted to entry metadata, and reclassification never changes an ID; **bidirectional indexing** — a new source index `00-Source Index.md` aggregates each BL entry's source evidence, data changes, trigger entry points and external impacts into four reverse index tables, forming a bidirectional index with the BL entries; **precise staleness detection** — the changed file set now covers uncommitted changes (`git status --porcelain`), and change impact is analysed across multiple layers (paths / symbols / data / interfaces / configuration / tests / renames and deletions). Also: BL entries gained a "Freshness Status" section, bringing the fixed section count to 15, and evidence status and freshness status are displayed as two independent fields that must never be merged; the operating modes were split into generation / query (read-only by default, writes no files) / sync (incremental maintenance) |
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
