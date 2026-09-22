# Project Knowledge Layer Templates (v1.4.2)

> **Positioning**: The knowledge layer is the core; the 7 traditional documents (01–07) are only different presentation views of the knowledge layer. The 7 document templates are in `references/en/document-templates.md`, and their file names, numbering, and paths remain unchanged.
>
> **Traceability chain**:
>
> ```
> Business question → Business capability → Business rule → Entry/API → Call chain → Data changes → External dependencies → Source files/symbols → Tests
> ```
>
> **Bidirectional index**: BL entry → source (the "Source Evidence", "Data Changes", "Trigger Entry Points", and "External Impacts" sections of each BL entry) and source → BL (`00-Source Index.md`) are reverse indexes of each other and must stay consistent.
>
> **Query mode (read-only)**: read-only means **not modifying files**, not "not reading source". Query mode may read source code, `git diff`, and related tests for immediate verification, but **must not modify the knowledge layer or the source**. The normal path is "question → knowledge map → BL → answer directly"; when a BL is missing, its freshness status is not `Current`, or the evidence is insufficient, take "BL → source index (locate symbols and files) → read the actual source → read-only verification → answer". **Never tell the user to go run a sync just because the documentation is stale.**
>
> **Author acquisition priority**: `SKILL.md`, section "Author Information Acquisition", is the single authority (four levels: (1) author explicitly provided by the user → (2) local default author `Soonkeira` (https://github.com/Soonkeira) → (3) Git commit author `git log -1 --format=%an` → (4) leave empty). Using a machine account name (`$env:USERNAME` / `whoami`) as the author is **strictly prohibited**.
>
> **Example labeling**: All example content in the templates of this file is placeholder content and must be explicitly labeled `Example (placeholder, not project fact)`; replace it with real analysis results when generating documents.
>
> **This file contains**:
> 1. Knowledge map template `00-Project Knowledge Map.md`
> 2. Source index template `00-Source Index.md`
> 3. Business logic entry template `business/BL-<NNN>-<business-name>.md`
> 4. Status definitions and determination rules
> 5. Traceability good/bad examples
> 6. Example content labeling rules

---

## 1. Knowledge Map Template `00-Project Knowledge Map.md`

### 1.1 Output path

`Doc/<project-name>/en/00-Project Knowledge Map.md` (Chinese counterpart: `Doc/<项目名称>/cn/00-项目知识地图.md`)

### 1.2 Template body

> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.

````markdown
# Project Knowledge Map

**Project Name**: XXX Project
**Author**: <Author>
**Date**: YYYY-MM-DD
**Version**: 1.26.509.1234

## Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.26.509.1234 | 2026-05-09 | <Author> | Initial version: business capability index and verification baseline established |

---

## 1. Analysis Scope

| Item | Content |
|------|---------|
| Analysis root | `<project root>` |
| Directories scanned | `src/`, `app/`, `config/`, `tests/` |
| Directories skipped | `.git/`, `node_modules/`, `vendor/`, `dist/`, `build/`, `coverage/`, `.venv/`, cache and generated-artifact directories |
| Files skipped | `.env*`, `*.pem`, `*.key`, `credentials*`, `secrets*` |
| Not covered | `<modules or directories not read, and why>` |

## 2. Verification Baseline

| Field | Value |
|-------|-------|
| last_verified_commit | `<commit hash>` |
| Verification date | YYYY-MM-DD |
| Working tree status | `Clean` / `Dirty` (determination command `git status --porcelain`: empty output means `Clean`) |
| Verification status | `Formal baseline` / `Provisional working-tree analysis` |
| Formal baseline | `<commit hash>` (advanced after re-verification only while the working tree is `Clean`; kept at the previous formal baseline, not advanced, while `Dirty`) |
| Freshness determination available | Yes |
| Freshness determination method | Change file set (committed part `git diff --name-status -M <last_verified_commit>..HEAD` ∪ working-tree part `git status --porcelain`) matched against each BL entry's "Source Evidence", "Core Execution Flow", "Data Changes", "Trigger Entry Points", "External Impacts", "Preconditions", and "Related Tests" through the multi-layer impact analysis (see section 4.2) |
| traditional_docs_status | `current` / `outdated` / `provisional` |
| traditional_docs_working_tree | `Clean` / `Dirty` / `Not Found (no Git)` |
| traditional_docs_generated_from_commit | `<commit hash at which the 7 traditional documents were generated>` |

> **The verification baseline advances only while the working tree is clean**: while the working tree is `Dirty` you may analyze, answer questions, and update BL content, but you **must not advance `last_verified_commit`** and **must not mark an entry `Current`**; that entry's freshness status is `Provisional Working-Tree Analysis`. Once the working tree becomes `Clean` again, those `Provisional Working-Tree Analysis` entries must unconditionally re-enter this run's affected set and be re-verified against the source; until all of them are re-verified they must not be marked `Current` and the formal baseline must not be advanced (see 4.2 step 4). Rationale: a BL may have been generated from **uncommitted working-tree code**; if the user later discards those changes with `git restore`, HEAD is unchanged and the working tree is clean again, yet the BL describes an implementation that **no longer exists** while the system still considers it "current".
>
> **Traditional document freshness**: sync mode does **not** update the 7 traditional documents, so after a sync (and once the code has changed) `traditional_docs_status` must be `outdated` until generation mode is run again, which returns it to `current`. The three values of `traditional_docs_status`: `current` = the traditional documents were generated from a **Clean** working tree and the knowledge layer has found no code change newer than them; `outdated` = the code / knowledge layer has already changed but the 7 traditional documents have not been regenerated; `provisional` = the traditional documents were generated on a **Dirty** working tree, so their content may include uncommitted code and **the HEAD commit must not be treated as their only source**. Generation mode: Clean → `current` + `traditional_docs_working_tree` = `Clean`; Dirty → `provisional` + `traditional_docs_working_tree` = `Dirty`; sync mode (code changed but 01–07 not regenerated) → `outdated`; re-running generation mode follows the same Clean / Dirty rule.
> `traditional_docs_status` / `traditional_docs_generated_from_commit` / `traditional_docs_working_tree` are **all recorded in `00-Project Knowledge Map.md`**; **`00-Project Knowledge Map.md` = the document status center**. Documents 01–07 remain generated artifacts and sync mode **does not modify** them, keeping sync mode lightweight; to tell whether the traditional documents are up to date, read these three fields in the knowledge map.

**Three baseline states must be distinguished (three mutually exclusive branches)**:

- **Branch A — no Git (or invalid HEAD)**: use the "Degraded form when there is no Git or no valid HEAD" below; record freshness as `Not Found (no Git baseline)`; every run requires a **full re-verification**; **never fabricate a commit hash**.
- **Branch B — Git exists, but no formal baseline has been established yet** (`last_verified_commit` is empty / `Formal baseline` = `not established`): see "Branch B values" below.
- **Branch C — a formal baseline already exists**: normal incremental sync (see 4.2 and 4.5).

**Branch B values (Example (placeholder, not project fact))**:

| Field | Value |
|-------|-------|
| last_verified_commit | not established |
| Formal baseline | not established |
| Verification status | `Provisional working-tree analysis` |
| Working tree status | `Dirty` |

> **Branch B performs no incremental diff of the form `<base>..HEAD`** (there is no valid `<base>` at this point) and must perform a **full re-verification**:
> - Working tree **`Clean`** → after the full re-verification, establish the formal baseline: `last_verified_commit` = current HEAD, `Formal baseline` = current HEAD, `Verification status` = `Formal baseline`, `Working tree status` = `Clean`, and the entries verified in this run = `Current`.
> - Working tree **`Dirty`** → keep `Verification status` = `Provisional working-tree analysis` and `Working tree status` = `Dirty`; keep `last_verified_commit` and `Formal baseline` at `not established`; **do not establish a formal baseline and do not mark anything `Current`**.
>
> **It is strictly prohibited** to write "Git exists but no formal baseline has been established" as `Not Found (no Git baseline)` — Git really does exist, and the two situations must be distinguished.

### Most Recent Incremental Analysis

| Field | Value |
|-------|-------|
| Baseline commit | `<commit hash>` |
| Analysis date | YYYY-MM-DD |
| Changed files (including uncommitted) | <N> |
| Affected BL | [BL-001](business/BL-001-<business-name>.md), [BL-002](business/BL-002-<business-name>.md) |
| BL re-verified | [BL-001](business/BL-001-<business-name>.md) |
| BL still marked stale | [BL-002](business/BL-002-<business-name>.md) |

**Degraded form when there is no Git or no valid HEAD**:

| Field | Value |
|-------|-------|
| last_verified_commit | Not Found (no Git baseline) |
| Verification date | YYYY-MM-DD |
| Working tree status | Not Found (no Git baseline) |
| Verification status | `Provisional working-tree analysis` |
| Formal baseline | Not Found (no Git baseline) |
| Freshness determination available | No |
| Freshness determination method | No freshness determination; every run requires a full re-verification |
| traditional_docs_status | Not Found (no Git baseline) (none of `current` / `outdated` / `provisional` can be determined) |
| traditional_docs_working_tree | Not Found (no Git) |
| traditional_docs_generated_from_commit | Not Found (no Git baseline) |
| Most recent incremental analysis | Not Found (no Git baseline, no incremental analysis) |

> Never fabricate a commit hash when there is no Git baseline (including `traditional_docs_generated_from_commit`).

## 3. ID Registry

| Item | Value |
|------|-------|
| Current highest ID | `BL-006` |
| Next available ID | `BL-007` |
| Retired IDs (tombstones, never reused) | `BL-006` (former "Order Batch Export", retired YYYY-MM-DD) |

> ID assignment rule: a new entry's ID = **historical highest BL ID + 1** (the current highest is `BL-006`, so the next new entry is `BL-007`).
> A deleted ID is a **permanent tombstone that is never reused**; **gaps caused by deletion are allowed** (a gap is normal, not an error). **A stable ID matters far more than consecutive numbering**.
> **Why this registry is required**: without it, "never reused" cannot be guaranteed — when the entry holding the highest ID is deleted, looking only at the current highest ID falls back to reusing a retired ID; read this table before assigning an ID.

## 4. Business Capability Index

> This section is the main body of the knowledge map. **The knowledge map must not be only a project introduction**; the business capability index must be its main body.
> The ID column must be a link to the BL entry; IDs increase by **historical highest BL ID + 1** (see the "ID Registry" section and the numbering rules in section 3.1) and are unrelated to business domains — a domain determines grouping only, never numbering.
> Evidence status and freshness status are **two independent fields** shown in two separate columns; **conflating them is strictly prohibited**.

### 4.1 <Business Domain One>

| ID | Business Capability | Description | Evidence Status | Freshness Status |
|----|---------------------|-------------|-----------------|------------------|
| [BL-001](business/BL-001-<business-name>.md) | <business capability name> | <one sentence on the problem this capability solves> | Verified | Current |
| [BL-002](business/BL-002-<business-name>.md) | <business capability name> | <one sentence on the problem this capability solves> | Partially Verified | ⚠ Possibly Stale |

### 4.2 <Business Domain Two>

| ID | Business Capability | Description | Evidence Status | Freshness Status |
|----|---------------------|-------------|-----------------|------------------|
| [BL-003](business/BL-003-cancel-order.md) | Cancel Order | <one sentence on the problem this capability solves> | Verified | Current |

## 5. Common Business Question Index

| Business Question | Related Business Logic |
|-------------------|------------------------|
| How is this feature implemented? | [BL-001](business/BL-001-<business-name>.md) |
| Why can this not be deleted? | [BL-002](business/BL-002-<business-name>.md) |
| What happens after an order is cancelled? | [BL-003](business/BL-003-cancel-order.md) |
| Why does this status change? | [BL-003](business/BL-003-cancel-order.md) |
| What data does this API ultimately modify? | [BL-001](business/BL-001-<business-name>.md) |
| Where in the code is this business rule? | [BL-002](business/BL-002-<business-name>.md) |

## 6. Not Found / Inferred Entries Summary

| ID | Business Capability | Evidence Status | Reason and Limitations |
|----|---------------------|-----------------|------------------------|
| [BL-004](business/BL-004-<business-name>.md) | <business capability name> | Inferred | <inferred only from directory structure and naming; no call-chain evidence found> |
| [BL-005](business/BL-005-<business-name>.md) | <business capability name> | Not Found | <no entry point, test, or data access code found> |

## 7. Coverage and Limitations

- Covered: <business domains for which knowledge entries exist>
- Not covered: <business domains without entries yet, and why>
- Known limitations: <parts that cannot be confirmed from code, such as dynamic dispatch, reflection, config-driven branches, external system behavior>
````

### 1.3 Filling requirements

- The **Analysis Scope** must reflect what was actually scanned: whatever is listed as scanned must really have been read, and the reason for skipping sensitive files and dependency directories must be stated.
- The **Verification Baseline** is the global baseline; `last_verified_commit`, the verification date, and the "Most Recent Incremental Analysis" must come from real Git command output and real analysis results (the changed-file count includes uncommitted changes). Without Git, use the degraded form and state explicitly that "every run requires a full re-verification"; **when Git exists but no formal baseline has been established (branch B), a full re-verification is mandatory and no incremental diff of the form `<base>..HEAD` may be performed** (see the branch notes in the "Verification Baseline" section).
- **The verification baseline advances only while the working tree is clean**: run `git status --porcelain` first to determine the `Working tree status` (empty output = `Clean`, any output = `Dirty`). While `Clean`, you may advance `last_verified_commit` and the `Formal baseline` after re-verification and mark the re-verified entries `Current`; while `Dirty`, you **must not advance `last_verified_commit`** and **must not mark anything `Current`**, and the `Verification status` is `Provisional working-tree analysis`.
- **Traditional document freshness**: `traditional_docs_status` holds one of three values — `current` (the traditional documents were generated from a **Clean** working tree and the knowledge layer has found no code change newer than them) / `outdated` (the code / knowledge layer has already changed but the 7 traditional documents have not been regenerated) / `provisional` (the traditional documents were generated on a **Dirty** working tree, so their content may include uncommitted code and **the HEAD commit must not be treated as their only source**); `traditional_docs_working_tree` holds `Clean` / `Dirty` / `Not Found (no Git)`; `traditional_docs_generated_from_commit` holds the commit at which the 7 traditional documents were generated. Generation mode: Clean → `current` + `Clean`, Dirty → `provisional` + `Dirty`; sync mode does not update the 7 traditional documents, so after a sync (and once the code has changed) it must be `outdated` until generation mode is run again, which returns it to `current`.
- **Traditional document status is maintained only in the knowledge map**: `traditional_docs_status` / `traditional_docs_generated_from_commit` / `traditional_docs_working_tree` are **all recorded in `00-Project Knowledge Map.md`**; **`00-Project Knowledge Map.md` = the document status center**. Documents 01–07 remain generated artifacts and sync mode **does not modify** them, keeping sync mode lightweight; to tell whether the traditional documents are up to date, read these three fields in the knowledge map, and **do not add a separate stale field to 01–07**.
- The **ID Registry** must stay consistent with the business capability index and the BL entries: `Current highest ID` is the historical highest BL ID (the historical maximum including retired IDs), `Next available ID` = current highest ID + 1, and `Retired IDs` lists every tombstone individually with its retirement date. A retired ID must not appear in the business capability index at the same time (this table is Example (placeholder, not project fact); replace it with real IDs when generating).
- The **Business Capability Index** is grouped by business domain, one table per group, with the fixed columns `| ID | Business Capability | Description | Evidence Status | Freshness Status |`. Do not add, remove, or rename columns, and do not merge the two statuses into one column.
- **The Business Capability Index is displayed grouped by business domain**: a group heading holds only the business domain name and **must not carry any number range**; the business domain must match the `**Business Domain**` field in the header of each BL entry.
- The **Not Found / Inferred Entries Summary** lists only entries whose evidence status is `Inferred` or `Not Found`, with their evidence gaps; that table holds the evidence status only, never the freshness status.
- The **Common Business Question Index** must map real user questions to BL IDs, with links that jump directly to the corresponding entry.
- **Being only a project introduction is prohibited**: a knowledge map without a business capability index counts as incomplete and must not be delivered.

---

## 2. Source Index Template `00-Source Index.md`

### 2.1 Output path

`Doc/<project-name>/en/00-Source Index.md` (Chinese counterpart: `Doc/<项目名称>/cn/00-源码索引.md`)

### 2.2 Template body

> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.

````markdown
# Source Index

**Project Name**: XXX Project
**Author**: <Author>
**Date**: YYYY-MM-DD
**Version**: 1.26.509.1234

## Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.26.509.1234 | 2026-05-09 | <Author> | Initial version: source → BL reverse index established |

---

> This index is aggregated from the "Source Evidence", "Data Changes", "Trigger Entry Points", "External Impacts", and "Related Tests" sections of every BL entry and must stay consistent with those entries; any inconsistency counts as a validation failure.

## 1. Source Files and Symbols → BL

| Source Path | Symbol | Type | Covered BL |
|-------------|--------|------|------------|
| `src/order/controller.ts` | `OrderController.cancel()` | Controller | [BL-003](business/BL-003-cancel-order.md) |
| `src/order/service.ts` | `OrderService.cancelOrder()` | Service | [BL-003](business/BL-003-cancel-order.md) |
| `src/order/repository.ts` | `OrderRepository.save()` | Repository | [BL-003](business/BL-003-cancel-order.md) |
| `src/order/repository.ts` | `OrderRepository.findById()` | Repository | [BL-001](business/BL-001-<business-name>.md) |

## 2. Data Tables / Migrations → BL

| Table Name | Change Source | Covered BL |
|------------|---------------|------------|
| `orders` | `src/order/repository.ts` → `OrderRepository.save()` | [BL-003](business/BL-003-cancel-order.md) |
| `orders` | `migrations/20260501_add_cancel_reason.sql` | [BL-003](business/BL-003-cancel-order.md) |

## 3. APIs / Entry Points → BL

| Entry Identifier | Type | Covered BL |
|------------------|------|------------|
| `POST /api/orders/{id}/cancel` | HTTP API | [BL-003](business/BL-003-cancel-order.md) |

## 4. Configuration Items → BL

| Configuration Item | Covered BL |
|--------------------|------------|
| `ORDER_CANCEL_WINDOW_MINUTES` | [BL-003](business/BL-003-cancel-order.md) |

## 5. External Dependencies / Resources → BL

| Type | Identifier | Covered BL |
|------|------------|------------|
| Kafka Topic | `order.cancelled` | [BL-003](business/BL-003-cancel-order.md) |
| Redis Key | `order:cancel:lock:<id>` | [BL-003](business/BL-003-cancel-order.md) |

> Type values: Kafka Topic / MQ / Redis Key / Cache / External API / Object Storage / File System / SMS / Email, etc.
> Data source: the "External Impacts" section of each BL entry; entries without external impacts do not appear in this table.

## 6. Tests → BL

| Test File | Test Method | Covered BL |
|-----------|-------------|------------|
| `tests/order/cancel.spec.ts` | `should cancel pending order` | [BL-003](business/BL-003-cancel-order.md) |

> Data source: the "Related Tests" section of each BL entry; entries without tests do not appear in this table (fabricating test files or test methods is prohibited).

## 7. Uncovered Source Files

| Source Path | Note |
|-------------|------|
| `<important file that was scanned but is not referenced by any BL>` | <reason: no entry yet / infrastructure / confirmed to contain no business logic> |

## 8. Coverage and Limitations

- Covered: <business domains and source scope for which a reverse index exists>
- Not covered: <directories or modules left out of the index, and why>
- Known limitations: <dynamic dispatch, reflection, config-driven branches, generated code, and other parts that cannot be statically attributed to a BL>

## 9. Maintenance Rules

- **Generation mode**: scan source and BL entries in full and build this index in one pass.
- **Sync mode**: incremental maintenance, updated together with the BL entries; it **writes only the knowledge layer** (knowledge map, source index, BL entries) and does not touch the 7 traditional documents.
- **Query mode**: **read-only by default** — read-only means **not modifying files**, not "not reading source". Query mode may read source code, `git diff`, and related tests for immediate verification, but **must not modify the knowledge layer or the source**.
- Normal path in query mode: question → knowledge map → BL → answer directly.
- When a BL is missing, its freshness status is not `Current`, or the evidence is insufficient: BL → **source index** (locate symbols and files) → read the actual source → read-only verification → answer.
- **Never tell the user to go run a sync just because the documentation is stale**: verify read-only and answer whenever possible.
- Whenever the "Source Evidence", "Data Changes", "Trigger Entry Points", "External Impacts", or "Related Tests" of a BL entry is added, changed, or removed, the corresponding rows of this index must be updated in the same run.
- This index is the **reverse index (source → BL)** and must stay consistent with the forward references inside the BL entries (BL → source); any inconsistency counts as a validation failure and must be fixed before delivery.
````

### 2.3 Filling requirements

- This index **must be aggregated from the BL entries**; a separate hand-written list is not allowed, and any inconsistency with the BL entries counts as a validation failure.
- The six reverse index tables have the fixed columns `| Source Path | Symbol | Type | Covered BL |`, `| Table Name | Change Source | Covered BL |`, `| Entry Identifier | Type | Covered BL |`, `| Configuration Item | Covered BL |`, `| Type | Identifier | Covered BL |`, and `| Test File | Test Method | Covered BL |`. Do not add, remove, or rename columns.
- The "Covered BL" column must hold links to BL entries; when the same source path is covered by several BLs, **list one row per BL** instead of packing several IDs into one row.
- The fifth table, "External Dependencies / Resources → BL", is sourced from the "External Impacts" section of each BL entry; `Type` takes values such as Kafka Topic / MQ / Redis Key / Cache / External API / Object Storage / File System / SMS / Email, and `Identifier` holds the real topic name, key pattern, API address, or storage path.
- The sixth table, "Tests → BL", is sourced from the "Related Tests" section of each BL entry; `Test File` and `Test Method` must be tests that really exist (confirmable by searching the test directory); **fabricating test files or test methods is prohibited**.
- The "Uncovered Source Files" table lists only **important files** (entry points, services, data access, configuration, migration scripts) with the reason they are uncovered; dependency directories, build artifacts, caches, and generated files are not listed.
- **Update it together during incremental sync**: after adding or deleting a BL entry, or after changing its evidence sections, the index must be re-aggregated; sync mode writes only the knowledge layer and does not touch the 7 traditional documents.
- Never present an inferred attribution as a confirmed one: reference relations that cannot be confirmed from code belong in "Coverage and Limitations" with the basis for the inference.

---

## 3. Business Logic Entry Template `business/BL-<NNN>-<business-name>.md`

### 3.1 Output path and numbering rules

- English: `Doc/<project-name>/en/business/BL-<NNN>-<business-name>.md`
- Chinese: `Doc/<项目名称>/cn/business/BL-<NNN>-<业务名称>.md`
- The same business logic uses the **same number** in every language.

**BL numbering rules**: `BL-` plus a three-digit number, assigned as **historical highest BL ID + 1**: `BL-001`, `BL-002`, `BL-003`, …

- **ID assignment rule**: a new entry's ID = **historical highest BL ID + 1** (the current highest is `BL-006`, so the next new entry is `BL-007`). The historical highest ID and the retired IDs are recorded in the "ID Registry" section of the knowledge map; read that section before assigning an ID.
- **Deletion means tombstone**: a deleted ID is a **permanent placeholder that is never reused**; for example, after `BL-006` is deleted it stays as a permanent placeholder, and the next new entry is `BL-007` (= historical highest ID + 1). Note that in this example the historical highest ID is itself a retired one: looking only at the highest ID among live entries would give `BL-005` and would wrongly reuse `BL-006` — which is exactly why the ID Registry must be maintained.
- **Gaps are allowed**: a gap caused by deletion is normal, not an error; **never reuse a retired ID just to keep the numbering consecutive**.
- **A stable ID matters far more than consecutive numbering**: once an entry's ID is assigned it never changes, no matter how entries are added, deleted, or reclassified; IDs are **never reused and never reordered**.
- **No numbering by business domain**: the business domain is metadata, written in the `**Business Domain**: <domain name>` field of the BL entry header, and plays no part in numbering.
- When a business domain is reclassified, **change only the `**Business Domain**` field and never the ID**; when the knowledge map displays entries grouped by domain, group headings must not carry a number range.
- After every ID assignment or retirement, the `Current highest ID` / `Next available ID` / `Retired IDs` entries of the knowledge map's "ID Registry" must be updated in the same run.

### 3.2 BL Granularity Rules

- **A BL = a business capability or use case (Use Case) perceivable by users/business**, not a Controller method and not a technical function.
- Several APIs that are only different entry points of the same business capability → **may belong to the same BL**.
- One API that has completely different business outcomes, transaction boundaries, or business lifecycles → **may be split into several BLs**.
- **CRUD does not default to "one BL per endpoint"**.
- Technical utility functions, Repository CRUD, and DTO conversions **do not become BLs of their own**.
- Name BLs by **stable business semantics** (for example "Create Order", "Cancel Order", "Review Refund", "Allocate Public IP"), not by technical actions.
- **Refactoring the Controller / Service layers must not change BL IDs** (IDs follow business semantics, not code structure).
- **Purpose**: keep the size of the generated knowledge system stable across different models and different runs (avoiding 28 entries in one run and 64 in the next).

**Example determination**: if `POST /users`, `GET /users/:id`, `PUT /users/:id`, and `DELETE /users/:id` belong to the same business capability "User Management", they may be merged into one BL; but if "Deactivate User" has its own transaction boundary and lifecycle, it becomes a separate entry.

### 3.3 Template body

> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.

````markdown
# BL-003 Cancel Order

**Project Name**: XXX Project
**Business Domain**: <domain name>
**Author**: <Author>
**Date**: YYYY-MM-DD
**Version**: 1.26.509.1234

## Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.26.509.1234 | 2026-05-09 | <Author> | Initial version |

> The header information (project name / business domain / author / date / version + change log) does not count as one of the 15 fixed sections below.

---

## 1. Business Description
<Explain, from the user/business perspective, what problem this business solves and who benefits. Do not start by describing code.>

## 2. Trigger Entry Points

| Type | Entry | Description |
|------|-------|-------------|
| <HTTP API / RPC / CLI / scheduled job / MQ consumer / event handler / UI action / internal method call> | `<entry identifier>` | <description> |

## 3. Preconditions
- <real conditions that must hold before execution>

## 4. Business Rules

| Rule | Condition | Result |
|------|-----------|--------|
| <rule name> | <condition> | allowed / rejected (<reason>) |

## 5. Core Execution Flow

> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.

```text
OrderController.cancel()
↓
OrderService.cancelOrder()
↓
Order.cancel()
↓
InventoryService.release()
↓
OrderRepository.save()
```

## 6. Key Branches
- <if / switch / state machine / exception branches and how each one changes business behavior>

## 7. Data Changes

| Table / Store | Field | Change | Transaction |
|---------------|-------|--------|-------------|
| `<table name>` | `<field name>` | `<old value> → <new value>` | Yes / No |

## 8. External Impacts
<MQ / Redis / third-party API / file system / cache / object storage / payment system / SMS and email, etc.; write "Not Found" when there are none.>

## 9. Exceptions and Failure Paths

| Exception condition | Handling result |
|---------------------|-----------------|
| <verified exception condition> | <handling result> |

## 10. Source Evidence

| Type | Source |
|------|--------|
| Controller / Service / Domain / Repository / Model / table / configuration item | `<path>` → `<symbol>` |

## 11. Related Tests

| Test file | Test method | Coverage |
|-----------|-------------|----------|
| `<test file path>` | `<test method name>` | <coverage point> |

## 12. Related Business Logic
- `BL-<NNN>` <related business logic name>

## 13. Evidence Status
`Verified` / `Partially Verified` / `Inferred` / `Not Found` (choose exactly one; judged by the model)

## 14. Freshness Status
`Current` / `⚠ Possibly Stale` / `Provisional Working-Tree Analysis` / `Not Found (no Git baseline)` (choose exactly one; freshness status is determined from Git change detection together with the source map and multi-layer impact analysis)

## 15. Last Verified Version
- Git commit: `<commit hash>` / `not established (Git exists, but no formal baseline has been established yet)` / `Not Found (no Git baseline)`
- Verification date: YYYY-MM-DD
- Working tree status: `Clean` / `Dirty`

> Git commit takes exactly one of three values: `<commit hash>` (branch C, a formal baseline exists) / `not established (Git exists, but no formal baseline has been established yet)` (branch B, Git exists but no formal baseline has been established yet) / `Not Found (no Git baseline)` (branch A, no Git or no valid HEAD).
> While the working tree is `Dirty`, **keep the formal baseline commit** (do not advance it) and append the line: "This analysis is based on an uncommitted working tree; the formal baseline was not advanced."
> **When no formal baseline has been established you must not fill in or fabricate a commit, and you must not miswrite it as `Not Found (no Git baseline)` (Git really does exist; the two must be distinguished)** (see the "Verification Baseline" section and 4.4).

The complete form for branch B (Git exists, but no formal baseline has been established yet):

```markdown
## 15. Last Verified Version
- Git commit: not established (Git exists, but no formal baseline has been established yet)
- Verification date: YYYY-MM-DD
- Working tree status: Dirty
- Note: this analysis is based on an uncommitted working tree; the formal baseline was not advanced.
```
````

### 3.4 Filling requirements and good/bad examples per section

**The order and titles of the 15 fixed sections are verbatim; do not add, remove, rename, or reorder them.**

The header metadata block (project name / business domain / author / date / version + change log) does not count as one of these 15 sections.

| # | Section | Filling requirement |
|---|---------|---------------------|
| 1 | Business Description | Explain, from the user/business perspective, what problem this business solves, who benefits, and in which scenarios it is used; **do not start by describing code**. |
| 2 | Trigger Entry Points | List the real entry points: HTTP API / RPC / CLI / scheduled job / MQ consumer / event handler / UI action / internal method call. Write "Not Found" when there is no entry-point evidence. |
| 3 | Preconditions | The real conditions that must hold before execution (data state, permissions, external system availability, configuration switches); do not write empty statements such as "the system is running normally". |
| 4 | Business Rules | When execution is allowed, when it is rejected, state transition conditions, permission requirements, numeric limits, and lifecycle limits. |
| 5 | Core Execution Flow | Must be the **real call chain**, listed layer by layer with arrows down to the data access layer; **vague descriptions are prohibited** (for example "validate and then update the order"). |
| 6 | Key Branches | Conditions such as if / switch / state machine / exception branches that **genuinely affect business behavior**, with the difference each branch makes. |
| 7 | Data Changes | Which tables and fields are modified, what data is created or deleted, how statuses change, and whether a transaction is involved. |
| 8 | External Impacts | MQ / Redis / third-party API / file system / cache / object storage / payment system / SMS and email, etc.; **write "Not Found" explicitly when there are none**. |
| 9 | Exceptions and Failure Paths | Only **verified** exception conditions and handling results; do not list unverified exceptions. |
| 10 | Source Evidence | Two-column table `\| Type \| Source \|`, with types covering Controller / Service / Domain / Repository / Model / table / configuration item, and the source written as `path → symbol`. |
| 11 | Related Tests | Real test files and test methods; when there are none, write "No corresponding automated tests found." **Fabricating tests is prohibited.** |
| 12 | Related Business Logic | Related BL IDs and names (upstream triggers, downstream dependencies, shared data). |
| 13 | Evidence Status | Choose exactly one: `Verified` / `Partially Verified` / `Inferred` / `Not Found`, judged by the model. |
| 14 | Freshness Status | **Freshness status is determined from Git change detection together with the source map and multi-layer impact analysis**; choose exactly one: `Current` / `⚠ Possibly Stale` / `Provisional Working-Tree Analysis` / `Not Found (no Git baseline)`; **never write it merged with the evidence status**, and never substitute a model judgment for Git change detection plus multi-layer impact analysis. |
| 15 | Last Verified Version | Git commit (exactly one of three: `<commit hash>` / `not established (Git exists, but no formal baseline has been established yet)` / `Not Found (no Git baseline)`) + verification date + working tree status (per-entry baseline); while the working tree is `Dirty` keep the formal baseline commit and append the line "This analysis is based on an uncommitted working tree; the formal baseline was not advanced"; when no formal baseline has been established (branch B) write `not established (Git exists, but no formal baseline has been established yet)`, and **never fabricate a commit or miswrite it as `Not Found (no Git baseline)`**; when there is no Git (branch A) write "Not Found (no Git baseline)". |

**How to write section 5, "Core Execution Flow"**:

> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.

```
OrderController.cancel()
↓
OrderService.cancelOrder()
↓
Order.cancel()
↓
InventoryService.release()
↓
OrderRepository.save()
```

- Good: every layer names a **real symbol** that can be traced back to an entry in the source evidence table.
- Bad: "First validate the parameters, then update the order status, and finally release the inventory." — that is not a call chain and cannot be located in code.

**How to write section 7, "Data Changes"**:

- Good: `orders.status: PENDING_PAYMENT → CANCELLED`, together with whether it happens in the same transaction and whether optimistic locking or version updates are involved.
- Bad: "Update the order table." — no field, no direction, no transaction information.

**How to write section 10, "Source Evidence"**:

- Good: `src/order/service.ts` → `cancelOrder()`
- Bad: `order service` — a directory/module-level description cannot locate a symbol.

**How to write section 11, "Related Tests"**:

- Good: list only test files and test methods that really exist (confirmable by searching the test directory).
- Bad: inventing test files that do not exist; when there are no tests, write "No corresponding automated tests found."

---

## 4. Status Definitions and Determination Rules

### 4.1 Evidence status

**Judged by the model**; each BL entry and each index row in the knowledge map **must choose exactly one of four**:

| Evidence status | Meaning | Basis for determination |
|-----------------|---------|-------------------------|
| `Verified` | Directly confirmed from source code | Clear source evidence (file → symbol); call chain, data changes, and branches were all read in the corresponding implementation |
| `Partially Verified` | Main conclusions confirmed, some links unconfirmed | Core chain confirmed, but some branches, exceptions, external impacts, or data changes were not read in the implementation |
| `Inferred` | Inferred from naming, directory structure, configuration, or documentation | No direct implementation evidence found; inference is only possible from structure or naming, and the basis for inference must be stated |
| `Not Found` | No evidence found at all | No entry point, implementation, test, or data access code found |

### 4.2 Freshness status

**Freshness status is determined from Git change detection together with the source map and multi-layer impact analysis** — it is neither a purely mechanical Git determination nor a purely model-based judgment; only four values exist:

- **Git is responsible for Change Detection**: HEAD, baseline, the change file set, renames/deletions, and the `Clean`/`Dirty` working tree — purely mechanical results.
- **Impact analysis is responsible for Impact Analysis**: symbol impact, API impact, data impact, configuration impact, and call-chain impact — not a purely Git-based judgment.
- **Together they produce the Freshness Status**.

| Freshness status | Meaning |
|------------------|---------|
| `Current` | The working tree is `Clean`, and none of the files and symbols related to the entry's evidence changed between the "Last Verified Version" commit and the current working tree |
| `⚠ Possibly Stale` | Something in that scope did change, and the entry was not re-verified in this run |
| `Provisional Working-Tree Analysis` | This analysis is based on a `Dirty` working tree (the formal baseline was not advanced); the entry's conclusions hold only for the current working tree |
| `Not Found (no Git baseline)` | No Git or no valid HEAD, so freshness cannot be determined (see 4.4) |

> Marking an entry `Current` requires a `Clean` working tree: when `git status --porcelain` produces output, the entry must be recorded as `Provisional Working-Tree Analysis` even if it was re-verified and hit no impact layer (see 4.5).

**Step 1: Determine the change file set (uncommitted changes must be included)**

Change file set = **committed part** ∪ **working-tree part**:

| Part | Command | Scope |
|------|---------|-------|
| Committed part | `git diff --name-status -M <base>..HEAD` | Changes committed since the baseline; `-M` detects renames |
| Working-tree part | `git status --porcelain` | Staged, unstaged, and untracked (`??`) changes |

> **Never look only at `<base>..HEAD`**: code may already be changed but not yet committed, so comparing commits alone misses staleness; the working-tree part must be included.
> Untracked files count only when they fall inside the analysis scope; entries ignored by `.gitignore` and default skipped directories (dependencies, build artifacts, caches, etc.) are skipped.
> **"The change file set includes uncommitted changes" and "the baseline advances only on a clean tree" do not conflict**: the former is used to **detect which entries are affected** (while `Dirty`, uncommitted changes must likewise enter the impact analysis, or staleness is missed), while the latter is used to **decide whether the formal baseline may be advanced** (while `Dirty`, `last_verified_commit` must not be advanced and no entry may be marked `Current`).

**Step 2: Multi-layer impact analysis** (any layer hit means the entry is affected, and the hit basis must be recorded):

| Layer | Determination method |
|-------|----------------------|
| a. Path layer | changed files ∩ the entry's "Source Evidence" paths |
| b. Symbol layer | symbols modified/added/deleted in the diff ∩ the call-chain symbols in the entry's "Core Execution Flow" and the symbol column of "Source Evidence" |
| c. Data layer | table names / field names / migrations touched by the change ∩ the entry's "Data Changes" |
| d. Interface layer | routes / APIs / entry signatures touched by the change ∩ the entry's "Trigger Entry Points" |
| e. Configuration layer | configuration items / environment variables touched by the change ∩ the entry's "External Impacts" and "Preconditions" |
| f. Test layer | test files touched by the change ∩ the entry's "Related Tests" → mark "test evidence pending re-check" (the business logic itself is not necessarily stale) |
| g. Rename / deletion | evidence file renamed → "path broken, needs fixing"; evidence file deleted → "evidence file deleted, re-verify or retire this entry" |

**Additional report**: if a changed file defines a symbol that appears in some BL's call chain but is not listed under that BL's "Source Evidence" → mark "evidence list incomplete, needs completion".

**Step 3: Reach the conclusion**

- Any layer hit and the entry was not re-verified in this run → set freshness to `⚠ Possibly Stale`; no layer hit → freshness is `Current`.
- **But both advancing the formal baseline and marking `Current` require a clean working tree**: when `git status --porcelain` produces output (`Dirty`), `last_verified_commit` must not be advanced and nothing may be marked `Current`; the freshness status is `Provisional Working-Tree Analysis`.
- A `Dirty` working tree affects only **baseline advancement** and the **`Current` marker**, never analysis or answers: the entry can still be analyzed, answered from, and updated.

**Step 4: Mandatory recovery of provisional entries (while the working tree is `Clean`; always required)**

The affected BL set must be the **union**, not merely the hits of the impact analysis:

```text
affected BL =
  BLs hit by impact analysis (Git change detection + source map + multi-layer determination)
  ∪ all BLs whose freshness_status = Provisional Working-Tree Analysis (while the current working tree is Clean)
```

- **Whenever a BL's current freshness status is `Provisional Working-Tree Analysis`, then once the working tree later becomes `Clean` that BL must unconditionally join this run's affected set and be re-verified by reading the current source, regardless of whether a `<baseline>..HEAD` diff exists.**
- Before **all** of these provisional BLs have been re-verified: **must not mark them `Current`**, **must not treat the knowledge layer as consistent again**, and **must not advance the formal baseline**.
- After re-verification, assign the status that matches reality: content consistent with the current `Clean` source → `Current`; cannot be confirmed → record `Partially Verified` / `Inferred` / `⚠ Possibly Stale` as appropriate; refresh `00-Source Index.md` in the same run.
- **Why this is necessary (the `git restore` scenario)**: formal baseline = `abc123` → code is modified in the working tree (`Dirty`) → after a sync the BL body is updated from the `Dirty` code and its freshness status is `Provisional Working-Tree Analysis` → the user discards the changes with `git restore` → HEAD is still `abc123`, the working tree is `Clean` again, and `git diff abc123..HEAD` is empty. The BL body may still describe temporary code that no longer exists, and the regular Git impact analysis **cannot detect it** (there is no diff); only this rule, which forces `Provisional Working-Tree Analysis` entries back into the affected set, can find and fix that kind of knowledge pollution.

Supporting commands: `git rev-parse HEAD` (current baseline), `git log --oneline <base>..HEAD` (change overview), `git status --porcelain` (uncommitted changes), `git rev-list --count HEAD` (fourth part of the version number), `git log -1 --format=%an` (Git commit author).

### 4.3 The two fields are independent

- Evidence status and freshness status are **two mutually independent fields** that **must never be conflated or written merged**: `Verified` means "the implementation was read at the time", not that the content is still fresh; `⚠ Possibly Stale` means "the code has changed", not that the original evidence was wrong.
- The knowledge map shows them in **two separate columns**: `| ID | Business Capability | Description | Evidence Status | Freshness Status |`; the evidence status column holds only one of the four values, and the freshness status column holds only one of the four values.
- **Never** merge the two statuses into one column or one line: do not join the evidence status and the freshness status with a prefix, a slash, parentheses, or similar — each must occupy its own column or its own section.
- Inside a BL entry the two are likewise written in separate sections: "13. Evidence Status" and "14. Freshness Status".

### 4.4 Degradation without Git or without a valid HEAD

- **This section applies only to branch A (no Git or no valid HEAD)**: Git existing while no formal baseline has been established yet (`last_verified_commit` / `Formal baseline` = `not established`) is **branch B**; it requires a full re-verification and keeps `Provisional working-tree analysis`, and writing it as `Not Found (no Git baseline)` is **strictly prohibited**.
- Perform no freshness determination; record freshness as `Not Found (no Git baseline)`.
- State in the "Verification Baseline" section of the knowledge map that freshness determination is unavailable and that **every run requires a full re-verification**.
- **Never fabricate a commit hash**, and never substitute a date or a file timestamp for a commit.

### 4.5 Incremental maintenance baseline

- **Global baseline**: written in the "Verification Baseline" section of `00-Project Knowledge Map.md`, with the fields `last_verified_commit` + verification date + `Working tree status` + `Verification status` + `Formal baseline`.
- **Per-entry baseline**: written in the "Last Verified Version" section of each BL entry, with the fields Git commit + verification date + working tree status.
- **Advance the baseline only while the working tree is clean**, determined with `git status --porcelain`:
  - Working tree `Clean` (no output): you may advance `last_verified_commit` and the `Formal baseline` after re-verification, and mark the re-verified entries `Current`.
  - Working tree `Dirty` (output present): you may analyze, answer, and update BL content, but you **must not advance `last_verified_commit`** and **must not mark anything `Current`**; the entry's freshness status is `Provisional Working-Tree Analysis`, and the entry's "Last Verified Version" keeps the formal baseline commit (in branch B, where no formal baseline exists yet, write `not established (Git exists, but no formal baseline has been established yet)`; **never fabricate a commit**) and appends the line "This analysis is based on an uncommitted working tree; the formal baseline was not advanced."
  - **Rationale**: a BL may have been generated from **uncommitted working-tree code**; if the user later discards those changes with `git restore`, HEAD is unchanged and the working tree is clean again, yet the BL describes an implementation that **no longer exists** while the system still considers it "current".
- During an incremental update, re-verify only the affected entries (determined by the multi-layer analysis in 4.2); keep the "Last Verified Version" of unaffected entries at its original value instead of refreshing it to the current HEAD.
- **The affected set must include every `Provisional Working-Tree Analysis` entry**: while the working tree is `Clean`, this run's affected set = BLs hit by the impact analysis ∪ all BLs whose freshness status is still `Provisional Working-Tree Analysis` (see 4.2 step 4); **`last_verified_commit` must not be advanced until all of them have been re-verified**, and they must not be marked `Current` or treated as a consistent knowledge layer.
- **While no formal baseline has been established (branch B), perform no incremental diff of the form `<base>..HEAD`** (there is no valid `<base>`); a **full re-verification** is mandatory: working tree `Clean` → after the re-verification, establish the formal baseline (`last_verified_commit` = `Formal baseline` = current HEAD, `Verification status` = `Formal baseline`, `Working tree status` = `Clean`, and the entries verified in this run = `Current`); working tree `Dirty` → keep `last_verified_commit` and `Formal baseline` at `not established` and `Verification status` at `Provisional working-tree analysis`, and **do not establish a formal baseline or mark anything `Current`**.
- After every incremental analysis, record in "Most Recent Incremental Analysis" of the knowledge map's "Verification Baseline" section: baseline commit, analysis date, changed-file count (including uncommitted), affected BL, re-verified BL, and BL still marked stale.
- The change file set must include uncommitted changes (`git status --porcelain`); **never compare only `<base>..HEAD`**. This is an **impact detection** rule and does not conflict with the **baseline advancement** rule above ("advance the baseline only while the working tree is clean") — see 4.2 steps 1 and 3.

### 4.6 Prohibitions

- **Never package model inference as fact**: inferred content must be explicitly labeled `Status: Inferred` with its basis, and assertive wording such as "the system will" or "it must be" is not allowed.
- Do not fabricate business facts, commit hashes, table names, APIs, test cases, or business rules; freshness determination must come from real Git command output, and **fabricating a commit hash in a freshness determination is prohibited**.
- **Never advance the formal baseline or mark anything `Current` on a `Dirty` working tree**: uncommitted changes may be discarded by `git restore`, at which point the BL describes an implementation that no longer exists (see 4.5).
- **Never mark anything `Current` while un-re-verified `Provisional Working-Tree Analysis` entries exist**: once the working tree becomes `Clean`, those entries must unconditionally re-enter the affected set and be re-verified against the source (see 4.2 step 4); before all of them are re-verified you **must not mark them `Current`**, **must not treat the knowledge layer as consistent again**, and **must not advance the formal baseline**.
- **Never fabricate a commit when no formal baseline has been established, and never miswrite it as `Not Found (no Git baseline)`**: for branch B (Git exists, but no formal baseline has been established yet) the "Last Verified Version" reads `not established (Git exists, but no formal baseline has been established yet)` (see 3.4 section 15, the "Verification Baseline" section, and 4.4).
- **Never reuse a retired BL ID**: deletion means tombstone, IDs are always assigned as "historical highest ID + 1", and the knowledge map's "ID Registry" must be updated in the same run (see 3.1).
- **It is strictly prohibited** to write "Git exists but no formal baseline has been established" (branch B) as `Not Found (no Git baseline)`: Git really does exist and only the formal baseline is not established yet, so the two must be distinguished (see the "Verification Baseline" section and 4.4).
- Do not delete the existing 7 document templates, perform large-scale renames, or introduce runtime dependencies; do not introduce a database, web UI, RAG, or vector database.

---

## 5. Traceability Good/Bad Examples

### 5.1 Business rules

- Bad: "The system checks user permissions."
- Good: "The system checks in `PermissionService.checkPermission()` whether the current user has the operation permission for that resource."

### 5.2 Call chain

- Bad: "Call the order service to complete the cancellation."
- Good: `OrderController.cancel()` → `OrderService.cancelOrder()` → `Order.cancel()` → `InventoryService.release()` → `OrderRepository.save()`

### 5.3 Data changes

- Bad: "Update the order status."
- Good: `orders.status: PENDING_PAYMENT → CANCELLED`, with `orders.updated_at` updated in the same transaction.

### 5.4 Tests

- Bad: "Already covered by unit tests."
- Good: "`tests/order/cancel.spec.ts` → `should cancel pending order` covers the normal cancellation path." (When no test is found, write "No corresponding automated tests found.")

### 5.5 Labeling inference

- When only structural inference is possible (for example, seeing an `XxxRepository` name without reading its implementation), explicitly label it `Status: Inferred` and state the basis (directory structure / naming / configuration item / migration script).
- Inferred content must not be placed in the `path → symbol` column of the "Source Evidence" table as if it were direct evidence; if it must be kept, put it on a separate line labeled "(inferred)".

---

## 6. Example Content Labeling Rules

- All examples in the templates of this file (including the "cancel order" call chain, the source index example rows, the ID registry example rows, the external dependencies / resources example rows, the tests → BL example row, `src/order/service.ts` → `cancelOrder()`, `orders.status: PENDING_PAYMENT → CANCELLED`, and the test file example) are **Example (placeholder, not project fact)**.
- In generated documents, wherever example content remains, the label line must remain as well: `> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.`
- Replace examples with real analysis results during generation; **never** write template examples into a delivered document as project facts.
