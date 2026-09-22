> Version: v1.4.3 ｜ Author: [Soonkeira](https://github.com/Soonkeira)

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
| Source-to-BL Reverse Index | Looks business logic up from source code (previously only BL → Source was possible): aggregates each BL entry's source evidence, data changes, trigger entry points, external impacts and related tests into **six reverse index tables** — source files and symbols → BL, tables/migrations → BL, APIs/entry points → BL, configuration items → BL, external dependencies/resources → BL, and tests → BL |
| Bidirectional Index Consistency | The source index must stay consistent with the BL entries; any mismatch counts as a validation failure and must be fixed before the verification baseline can advance |
| Relationship Mapping | Maps the dependencies among APIs, the database, MQ and external services |
| Git Verification Status | Records the knowledge map's verification baseline and each BL entry's last verified version; **freshness status is determined from Git change detection together with the source map and multi-layer impact analysis** (Git performs change detection: HEAD, baseline, changed file set, `Clean`/`Dirty` working tree; impact analysis covers symbols, APIs, data, configuration and call chains) and is a field independent from evidence status; **the verification baseline advances only while the working tree is clean** (`git status --porcelain` produces no output = `Clean`) — while the working tree is `Dirty` you may analyze and update BL content, but **must not advance the formal baseline and must not mark anything `Current`**; that entry's freshness status is `Provisional Working-Tree Analysis` |
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

The knowledge layer consists of `00-Project Knowledge Map.md`, `00-Source Index.md` and `business/BL-*.md`; **the source index must stay consistent with the BL entries**, and any mismatch counts as a validation failure. The source index contains **six reverse index tables** (source files and symbols, tables/migrations, APIs/entry points, configuration items, external dependencies/resources, and tests → BL). IDs and file names correspond one-to-one between the Chinese and the English output.

## 4. Business Logic Entry Structure

**Numbering rule**: `BL-` plus a three-digit number, assigned as **historical highest BL ID + 1**: `BL-001`, `BL-002`, `BL-003`, … (when the current highest is `BL-006`, the next new entry is `BL-007`), with no numbering by business domain. A deleted ID is a **permanent tombstone that is never reused**; **gaps caused by deletion are allowed** (a gap is normal, not an error). The knowledge map's "ID Registry" must be read before assigning an ID; an existing ID never changes and is never reordered — **a stable ID matters far more than consecutive numbering**. The same business logic keeps the same number in both languages.

**Business domain is metadata**: the domain is written in the entry header field `**Business Domain**: <business-domain-name>`. Reclassifying domains changes only that field and **never changes an ID**. The knowledge map still groups entries by business domain, but group headings **must not carry number ranges**.

**BL Granularity Rules**: **a BL = a business capability or use case (Use Case) perceivable by users/business**, not a Controller method and not a technical function. Apply the rules below; the purpose is to **keep the size of the generated knowledge system stable across different models and different runs** (avoiding 28 entries in one run and 64 in the next):

- Several APIs that are only **different entry points** of the same business capability → **may belong to the same BL**
- One API that has **completely different business outcomes, transaction boundaries, or business lifecycles** → **may be split into several BLs**
- **CRUD does not default to "one BL per endpoint"**
- Technical utility functions, Repository CRUD, and DTO conversions **do not become BLs of their own**
- Name BLs by **stable business semantics** (for example "Create Order", "Cancel Order", "Review Refund", "Allocate Public IP"), not by technical actions
- **Refactoring the Controller / Service layers must not change BL IDs** (IDs follow business semantics, not code structure)

**Example determination**: if `POST /users`, `GET /users/:id`, `PUT /users/:id`, and `DELETE /users/:id` belong to the same business capability "User Management", they may be merged into one BL; but if "Deactivate User" has its own transaction boundary and lifecycle, it becomes a separate entry.

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
| 14 | Freshness Status | Choose exactly one of four, see the table below; **freshness status is determined from Git change detection together with the source map and multi-layer impact analysis**, and is independent from evidence status |
| 15 | Last Verified Version | Git commit (**exactly one of three**: `<commit hash>` / `not established (Git exists, but no formal baseline has been established yet)` / `Not Found (no Git baseline)`) + verification date (per-entry baseline); while the working tree is `Dirty` keep the formal baseline commit and append the line "This analysis is based on an uncommitted working tree; the formal baseline was not advanced"; **when no formal baseline has been established write `not established (Git exists, but no formal baseline has been established yet)` — a commit hash must never be filled in or fabricated**; `not established (Git exists, but no formal baseline has been established yet)` and `Not Found (no Git baseline)` **must continue to be distinguished and never conflated** (the former = Git and HEAD exist but no valid formal baseline commit yet; the latter = no Git or no valid HEAD) |

**Two mutually independent fields** that must never be conflated or merged into a single display:

| Field | Determined by | Values |
|-------|---------------|--------|
| Evidence status | Model judgement (one of four) | `Verified` / `Partially Verified` / `Inferred` / `Not Found` |
| Freshness status | Git change detection + multi-layer impact analysis (one of four) | `Current` / `⚠ Possibly Stale` / `Provisional Working-Tree Analysis` / `Not Found (no Git baseline)` |

The knowledge map index table always shows them in **two separate columns**, in the order `| ID | Business Capability | Description | Evidence Status | Freshness Status |`; the freshness marker must never be prefixed to the evidence status, and the two must never be merged into a single display in any form.

`Verified` means "the implementation was read at the time" — not that the content is still fresh; `⚠ Possibly Stale` means "the code has changed" — not that the original evidence was wrong.

## 5. Operating Modes

| Mode | How to trigger | Flow |
|------|----------------|------|
| Generation mode | `$project-doc-generator 生成当前项目的完整项目文档`, or say "Generate project documents" / "Create project documents" | Full scan of the code → extract business knowledge → fetch version info → build the knowledge layer (knowledge map + source index + BL entries) → generate the 7 standard documents with the knowledge layer as the source of truth (writes files) → post-generation validation |
| Query mode (read-only by default) | Just ask a business question | Reads the knowledge map, the source index and the BL entries, locates the related entry, and answers along its call chain, business rules, data changes and source evidence — **writes no files** ("read-only" = modifying no files, not "not reading source"); **source code, `git diff` and related tests may be read for immediate verification**, and a missing entry, a freshness status other than `Current`, or insufficient evidence is handled by verifying against the actual source and answering — **the user is never asked to run a sync first**; it only switches to sync mode when the user explicitly asks for an update |
| Sync mode (incremental maintenance) | Ask for an incremental sync, e.g. "Sync project documents" / "Sync the knowledge layer" | Pre-run branch determination (no Git / Git present but no formal baseline yet / formal baseline exists) → change impact analysis → re-verify only the affected entries → refresh freshness status → update the source index → advance the verification baseline; writes the knowledge layer only and never touches the 7 standard documents |

Query mode is read-only by default and never rewrites a file just because a question was asked (read-only verification may read the source code). Sync mode maintains the knowledge layer only (knowledge map, source index and BL entries); the 7 standard documents are refreshed only by running generation mode again.

## 6. Git Incremental Maintenance and Staleness Detection

1. **Pre-run branch determination (three mutually exclusive states)**: before any incremental maintenance, determine which branch this run belongs to — the three branches are mutually exclusive, and the result decides whether the run is incremental or full:
   - **Branch A — no Git (or an invalid HEAD)**: use the "no Git" degraded mode, record the freshness status as `Not Found (no Git baseline)`, and perform a **full re-verification** on every run
   - **Branch B — Git exists, but no formal baseline has been established yet** (`last_verified_commit` is empty / `Formal baseline` = `not established`): **perform no incremental diff of the form `<base>..HEAD`** (there is no valid `<base>` at this point) and perform a **full re-verification**; working tree **Clean** → after the re-verification, establish the formal baseline (`last_verified_commit` = `Formal baseline` = current HEAD, verification state = `Formal baseline`, entries verified in this run = `Current`); working tree **Dirty** → keep the verification state at `Provisional Working-Tree Analysis` and `last_verified_commit` / `Formal baseline` at `not established`, and **do not establish a formal baseline or mark anything `Current`**
   - **Branch C — a formal baseline already exists**: run the normal incremental sync (from step 3)

   Writing "Git exists but no formal baseline has been established" (branch B) as `Not Found (no Git baseline)` is **strictly prohibited**: Git really does exist, and the two must be distinguished.
2. **First run**: scan the whole codebase and build the knowledge map (including the "Verification Baseline" and the "ID Registry" that immediately follows it), the source index and all BL entries; then run `git status --porcelain` to determine the **working-tree state**: when **Clean**, take the current commit with `git rev-parse HEAD` and write it into the "Verification Baseline" as `last_verified_commit` (same value as the `formal baseline`) together with the verification date, record the working-tree state as `Clean` and the verification state as `formal baseline`, and mark the entries verified in this run `Current` (that is, the formal baseline is established once the full re-verification completes); when **Dirty**, **do not advance `last_verified_commit`** (keep `last_verified_commit` and the `formal baseline` at `not established`) and **do not mark anything `Current`**, recording the working-tree state as `Dirty` and the verification state as `Provisional Working-Tree Analysis`. Also write the three traditional-document freshness fields: `traditional_docs_generated_from_commit` (the HEAD commit of this generation) plus `traditional_docs_working_tree` and `traditional_docs_status`, which take their values from the working-tree state — **Clean → `Clean` + `current`**; **Dirty → `Dirty` + `provisional`** (this run does generate the 7 traditional documents, but their content may include uncommitted code); and update the ID Registry's current highest ID / next available ID / retired IDs.
3. **Later runs (changed file set)**: read the existing `last_verified_commit`, working-tree state, verification state, `traditional_docs_*` and the ID Registry; the changed file set is the **committed part ∪ the working-tree part**:
   - Committed part: `git diff --name-status -M <last_verified_commit>..HEAD`
   - Working-tree part: `git status --porcelain` (covers staged, unstaged and untracked `??` entries)

   **Looking only at `<last_verified_commit>..HEAD` is not enough**: code may already have been changed without being committed, and comparing commit ranges alone would miss stale entries.
4. **Change impact analysis (multi-layer)**: intersecting file paths alone is not enough. Each layer below is evaluated, and a hit on any layer marks the entry affected and records the basis:
   - Path layer: changed files ∩ the paths in the entry's "Source Evidence"
   - Symbol layer: changed symbols ∩ the symbols on the call chain
   - Data layer: table names / fields / migrations ∩ the entry's "Data Changes"
   - Interface layer: routes / entry-point signatures ∩ the entry's "Trigger Entry Points"
   - Configuration layer: configuration items ∩ the entry's "External Impacts" and "Preconditions"
   - Test layer: test files ∩ the entry's "Related Tests" → mark "test evidence pending re-check"
   - Rename / delete: a broken path must be repaired; a deleted evidence file requires re-verification
5. **Mandatory recovery of provisional entries (required while the working tree is `Clean`)**: the affected set (affected BL) must be the **union**, not merely the impact-analysis hits:

   ```
   affected BL =
     BL entries hit by impact analysis (Git change detection + source map + multi-layer determination)
     ∪ all BL entries whose freshness_status = Provisional Working-Tree Analysis (when the current working tree is Clean)
   ```

   - **Whenever a BL's current freshness status is `Provisional Working-Tree Analysis`, that BL must unconditionally join this run's affected set once the working tree later becomes `Clean`, and be re-verified by reading the current source — regardless of whether a `<baseline>..HEAD` diff exists.**
   - Until **all** of these provisional BLs have been re-verified: they **must not be marked `Current`**, the knowledge layer **must not be treated as consistent again**, and the formal baseline **must not be advanced**.
   **After re-verification, the two fields must be updated separately** (evidence status and freshness status are judged independently and must never be merged):
   - **Evidence status**: record `Verified` / `Partially Verified` / `Inferred` / `Not Found` according to how much of the implementation was actually read in this run.
   - **Freshness status**: record `Current` only when **every affected conclusion of that entry has been fully checked against the current Clean source**; when the re-verification is incomplete or unresolved impact items remain, **keep `⚠ Possibly Stale`**.
   - `00-Source Index.md` is refreshed in the same run.
   - **Why this is required (the `git restore` scenario)**: formal baseline = `abc123` → code modified in the working tree (`Dirty`) → after a sync the BL body is updated from the `Dirty` code and the freshness status becomes `Provisional Working-Tree Analysis` → the user discards the changes with `git restore` → HEAD is still `abc123`, the working tree is `Clean` again, and `git diff abc123..HEAD` is empty. The BL body may still describe temporary code that no longer exists, yet the normal Git impact analysis **cannot detect it** (there is no diff); only by force-including these entries in the affected set can such knowledge contamination be found and corrected.
6. **Re-verify only the affected entries**: the document set is not rewritten as a whole; unaffected entries stay as they are (no content rewrite, no renumbering, no change to an existing ID). Each BL entry also has its own "Last Verified Version" commit.
7. **Refresh the freshness status and the source index**: after the affected entries are re-verified their "Freshness Status" (one of four values) is updated, and changes to source evidence, data changes, trigger entry points, external impacts and related tests are propagated into the **six reverse index tables** of `00-Source Index.md` so that the source index and the BL entries stay consistent; when BL entries are added or retired, the knowledge map's "ID Registry" is updated in the same run (a deleted ID is a permanent tombstone that is never reused, and gaps are allowed).
8. **Staleness rule**: **freshness status is determined from Git change detection together with the source map and multi-layer impact analysis** — if a file listed under an entry's "Source Evidence" was modified after that entry's "Last Verified Version" commit (**including uncommitted changes**) and the entry was not re-verified in this run, its freshness becomes `⚠ Possibly Stale`; otherwise it stays `Current`.
9. **Baseline update (advanced only while the working tree is clean)**: **the formal baseline may be advanced only when all three conditions hold**: ① the working tree is `Clean`; ② **every affected BL has been processed**; ③ **no affected BL is still `Provisional Working-Tree Analysis` or `⚠ Possibly Stale`**. Only after every affected entry has been re-verified, and only when `git status --porcelain` produces no output (`Clean`), is the knowledge map's `last_verified_commit` advanced to the current HEAD (same value as the `formal baseline`), and the entries re-verified in this run are marked `Current`; when the working tree is `Dirty`, `last_verified_commit` is **not advanced** and nothing is marked `Current`, the working-tree state is recorded as `Dirty`, the verification state as `Provisional Working-Tree Analysis`, the affected entries' freshness status as `Provisional Working-Tree Analysis`, and their "Last Verified Version" gains the line "This analysis is based on an uncommitted working tree; the formal baseline was not advanced." **Rationale**: a BL may have been generated from uncommitted working-tree code; if the user later discards those changes with `git restore`, HEAD is unchanged and the working tree is clean again, yet the BL describes an implementation that no longer exists while the system still considers it "current".
10. **Record the "last incremental analysis"**: the knowledge map records the analysis time, the baseline commit, the number of changed files (including uncommitted ones), the working-tree state, the affected entries, the basis for each judgement and any items left unhandled.
11. **Traditional document freshness (three values)**: sync mode does not update the 7 traditional documents, so after a sync (and once the code has changed, i.e. the changed file set is not empty) `traditional_docs_status` must be set to `outdated` (`traditional_docs_generated_from_commit` and `traditional_docs_working_tree` keep the values from generation time); it returns to `current` (Clean) or `provisional` (Dirty), according to the working-tree state at that moment, only when the user explicitly asks for the 7 traditional documents to be regenerated (usually by running generation mode again). The three values: `current` = the traditional documents were generated from a **Clean** working tree and the knowledge layer has found no code change newer than them; `outdated` = the code / knowledge layer has already changed but the 7 traditional documents have not been regenerated; `provisional` = the traditional documents were generated on a **Dirty** working tree, so their content may include uncommitted code and **the HEAD commit must not be treated as their only source**. The three fields (`traditional_docs_status` / `traditional_docs_generated_from_commit` / `traditional_docs_working_tree`) are **all recorded in `00-Project Knowledge Map.md`** (**`00-Project Knowledge Map.md` = the document status center**); documents 01–07 remain generated artifacts and **gain no separate stale field**, and sync mode **does not modify** them either (keeping sync mode lightweight) — to tell whether the traditional documents are up to date, read these three fields in the knowledge map.
12. **Degradation**: **branch A** (no Git or no valid `HEAD`) performs no freshness determination, records freshness as `Not Found (no Git baseline)`, and states that every run needs a full re-check; **branch B** (Git exists but no formal baseline has been established yet) **may not perform any incremental diff of the form `<base>..HEAD`** and must perform a **full re-verification**, and must **not** record it as `Not Found (no Git baseline)`; when the baseline commit is unreachable (shallow clone, rebase, force-push) the run degrades to a full re-check with the reason stated. **Commit hashes are never fabricated.**

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
| v1.4.3 | **Rule correction** — no new features and no architecture upgrade: **① the status-dimension conflation in the Provisional BL mandatory-recovery rule is fixed** — `Partially Verified` / `Inferred` are an **evidence status**, while `Current` / `⚠ Possibly Stale` are a **freshness status**; the two dimensions are **judged independently and must never be merged**. **After re-verification, the two fields must be updated separately**: the evidence status records `Verified` / `Partially Verified` / `Inferred` / `Not Found` according to how much of the implementation was actually read in this run, while the freshness status records `Current` only when **every affected conclusion of that entry has been fully checked against the current Clean source** and **keeps `⚠ Possibly Stale`** when the re-verification is incomplete or unresolved impact items remain (the earlier merged result list has been replaced wholesale); **② the conditions for advancing the formal baseline are tightened** — **the formal baseline may be advanced only when all three conditions hold**: ① the working tree is `Clean`; ② **every affected BL has been processed**; ③ **no affected BL is still `Provisional Working-Tree Analysis` or `⚠ Possibly Stale`** |
| v1.4.2 | **Rule wrap-up (final minor fix)** — no new features and no new document types: **① a new Provisional BL mandatory-recovery rule** fixing **knowledge contamination by provisional BLs after Dirty → Clean** — the `affected BL` set is the **union** (BL entries hit by impact analysis ∪ all BL entries whose freshness status is `Provisional Working-Tree Analysis`, when the current working tree is `Clean`); these entries **must unconditionally join this run's affected set and be re-verified by reading the current source**, **regardless of whether a `<baseline>..HEAD` diff exists** (after `git restore` discards uncommitted changes the diff is empty, so the normal impact analysis cannot detect it); until **all** of them have been re-verified: they **must not be marked `Current`**, the knowledge layer **must not be treated as consistent again**, and the formal baseline **must not be advanced**; after re-verification the result is recorded according to the actual situation, and `00-Source Index.md` is refreshed in the same run (that wording was corrected in v1.4.3 so that evidence status and freshness status are judged separately); **② the commit field of BL section 15 is now explicitly one of three values when no formal baseline has been established** (`<commit hash>` / `not established (Git exists, but no formal baseline has been established yet)` / `Not Found (no Git baseline)`) — `not established (Git exists, but no formal baseline has been established yet)` (Git and HEAD exist, but no valid formal baseline commit yet) and `Not Found (no Git baseline)` (no Git or no valid HEAD) **must continue to be distinguished and never conflated**, and a commit hash must never be filled in or fabricated |
| v1.4.1 | **Rule-consistency wrap-up** — no new features and no new document types: **① a new "pre-run branch determination" with three mutually exclusive branches** — **branch A** (no Git or an invalid HEAD) uses the "no Git" degraded mode and records freshness as `Not Found (no Git baseline)`; **branch B** (Git exists but no formal baseline has been established yet: `last_verified_commit` / `Formal baseline` = `not established`) **performs no incremental diff of the form `<base>..HEAD`** (there is no valid base) and instead performs a **full re-verification** — the formal baseline is established only when the working tree is Clean, while a Dirty working tree keeps `not established`, records `Provisional Working-Tree Analysis` and marks nothing `Current` (**writing branch B as `Not Found (no Git baseline)` is strictly prohibited**); **branch C** (a formal baseline already exists) runs the normal incremental sync; **② `traditional_docs_status` grows from two values to three** (`current` / `outdated` / `provisional`) and a new field **`traditional_docs_working_tree`** (`Clean` / `Dirty` / `Not Found (no Git)`) is added — Clean generation → `current` + `Clean`, Dirty generation → `provisional` + `Dirty`; the three fields are all recorded in `00-Project Knowledge Map.md` (**the document status center**), documents 01–07 gain no separate stale field and sync mode does not modify them; **③ the 01–07 template version numbers become dynamic placeholders**, replaced at generation time by the version-number rule instead of a hard-coded fixed version; **④ the freshness-status wording is unified as "determined from Git change detection together with the source map and multi-layer impact analysis"**, no longer claiming a purely mechanical Git determination (the `Clean` / `Dirty` working tree, the commit baseline and the changed file set remain **mechanical Git results**); **⑤ real-project acceptance guidance added** (six tests A–F, standards in SKILL.md) |
| v1.4.0 | Fixes the rule contradiction and the consistency hole of v1.3.0, in six parts: **① the numbering rule becomes "historical highest BL ID + 1"** — a deleted ID is a **permanent tombstone that is never reused**, **gaps caused by deletion are allowed**, and the knowledge map gains an **"ID Registry"** recording the current highest ID / next available ID / retired IDs (v1.3.0's old ID-assignment wording amounts to reusing retired IDs and has been replaced wholesale); **② the verification baseline advances only while the working tree is clean** — `last_verified_commit` is advanced and entries are marked `Current` only when `git status --porcelain` produces no output (`Clean`); while the working tree is `Dirty` you may analyze, answer and update BL content, but must not advance the formal baseline or mark anything `Current`, and that entry's freshness status is `Provisional Working-Tree Analysis`; freshness status is now a four-value field (adding `Provisional Working-Tree Analysis`); **③ "read-only" in query mode means modifying no files, not "not reading source"** — source code, `git diff` and related tests may be read for immediate verification, and the user is never asked to run a sync first just because the documentation is stale; **④ the source index grows to six reverse index tables**, adding "external dependencies/resources → BL" and "tests → BL"; **⑤ new "BL granularity rules"** (a BL = a business capability or use case perceivable by users/business, CRUD does not default to one BL per endpoint, technical utility functions do not become BLs of their own); **⑥ the knowledge map's "Verification Baseline" gains `traditional_docs_status` and `traditional_docs_generated_from_commit`** to show whether the 7 traditional documents have fallen behind the knowledge layer |
| v1.3.0 | Three core upgrades to the knowledge layer: **stable IDs** — BL numbering now increments simply in creation order, the business domain is demoted to entry metadata, and reclassification never changes an ID; **bidirectional indexing** — a new source index `00-Source Index.md` aggregates each BL entry's source evidence, data changes, trigger entry points and external impacts into reverse index tables, forming a bidirectional index with the BL entries; **precise staleness detection** — the changed file set now covers uncommitted changes (`git status --porcelain`), and change impact is analysed across multiple layers (paths / symbols / data / interfaces / configuration / tests / renames and deletions). Also: BL entries gained a "Freshness Status" section, bringing the fixed section count to 15, and evidence status and freshness status are displayed as two independent fields that must never be merged; the operating modes were split into generation / query (read-only by default, writes no files) / sync (incremental maintenance) |
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
