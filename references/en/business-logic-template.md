# Project Knowledge Layer Templates (v1.3.0)

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
| Freshness determination available | Yes |
| Freshness determination method | Change file set (committed part `git diff --name-status -M <last_verified_commit>..HEAD` ∪ working-tree part `git status --porcelain`) matched against each BL entry's "Source Evidence", "Core Execution Flow", "Data Changes", "Trigger Entry Points", "External Impacts", "Preconditions", and "Related Tests" through the multi-layer impact analysis (see section 4.2) |

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
| Freshness determination available | No |
| Freshness determination method | No freshness determination; every run requires a full re-verification |
| Most recent incremental analysis | Not Found (no Git baseline, no incremental analysis) |

> Never fabricate a commit hash when there is no Git baseline.

## 3. Business Capability Index

> This section is the main body of the knowledge map. **The knowledge map must not be only a project introduction**; the business capability index must be its main body.
> The ID column must be a link to the BL entry; IDs increase **globally in order of first creation** and are unrelated to business domains — a domain determines grouping only, never numbering.
> Evidence status and freshness status are **two independent fields** shown in two separate columns; **conflating them is strictly prohibited**.

### 3.1 <Business Domain One>

| ID | Business Capability | Description | Evidence Status | Freshness Status |
|----|---------------------|-------------|-----------------|------------------|
| [BL-001](business/BL-001-<business-name>.md) | <business capability name> | <one sentence on the problem this capability solves> | Verified | Current |
| [BL-002](business/BL-002-<business-name>.md) | <business capability name> | <one sentence on the problem this capability solves> | Partially Verified | ⚠ Possibly Stale |

### 3.2 <Business Domain Two>

| ID | Business Capability | Description | Evidence Status | Freshness Status |
|----|---------------------|-------------|-----------------|------------------|
| [BL-003](business/BL-003-cancel-order.md) | Cancel Order | <one sentence on the problem this capability solves> | Verified | Current |

## 4. Common Business Question Index

| Business Question | Related Business Logic |
|-------------------|------------------------|
| How is this feature implemented? | [BL-001](business/BL-001-<business-name>.md) |
| Why can this not be deleted? | [BL-002](business/BL-002-<business-name>.md) |
| What happens after an order is cancelled? | [BL-003](business/BL-003-cancel-order.md) |
| Why does this status change? | [BL-003](business/BL-003-cancel-order.md) |
| What data does this API ultimately modify? | [BL-001](business/BL-001-<business-name>.md) |
| Where in the code is this business rule? | [BL-002](business/BL-002-<business-name>.md) |

## 5. Not Found / Inferred Entries Summary

| ID | Business Capability | Evidence Status | Reason and Limitations |
|----|---------------------|-----------------|------------------------|
| [BL-004](business/BL-004-<business-name>.md) | <business capability name> | Inferred | <inferred only from directory structure and naming; no call-chain evidence found> |
| [BL-005](business/BL-005-<business-name>.md) | <business capability name> | Not Found | <no entry point, test, or data access code found> |

## 6. Coverage and Limitations

- Covered: <business domains for which knowledge entries exist>
- Not covered: <business domains without entries yet, and why>
- Known limitations: <parts that cannot be confirmed from code, such as dynamic dispatch, reflection, config-driven branches, external system behavior>
````

### 1.3 Filling requirements

- The **Analysis Scope** must reflect what was actually scanned: whatever is listed as scanned must really have been read, and the reason for skipping sensitive files and dependency directories must be stated.
- The **Verification Baseline** is the global baseline; `last_verified_commit`, the verification date, and the "Most Recent Incremental Analysis" must come from real Git command output and real analysis results (the changed-file count includes uncommitted changes). Without Git, use the degraded form and state explicitly that "every run requires a full re-verification".
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

> This index is aggregated from the "Source Evidence", "Data Changes", "Trigger Entry Points", and "External Impacts" sections of every BL entry and must stay consistent with those entries; any inconsistency counts as a validation failure.

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

## 5. Uncovered Source Files

| Source Path | Note |
|-------------|------|
| `<important file that was scanned but is not referenced by any BL>` | <reason: no entry yet / infrastructure / confirmed to contain no business logic> |

## 6. Coverage and Limitations

- Covered: <business domains and source scope for which a reverse index exists>
- Not covered: <directories or modules left out of the index, and why>
- Known limitations: <dynamic dispatch, reflection, config-driven branches, generated code, and other parts that cannot be statically attributed to a BL>

## 7. Maintenance Rules

- **Generation mode**: scan source and BL entries in full and build this index in one pass.
- **Sync mode**: incremental maintenance, updated together with the BL entries; it **writes only the knowledge layer** (knowledge map, source index, BL entries) and does not touch the 7 traditional documents.
- **Query mode**: **read-only by default**; answer from the knowledge map, source index, and BL entries without writing any file.
- Whenever the "Source Evidence", "Data Changes", "Trigger Entry Points", or "External Impacts" of a BL entry is added, changed, or removed, the corresponding rows of this index must be updated in the same run.
- This index is the **reverse index (source → BL)** and must stay consistent with the forward references inside the BL entries (BL → source); any inconsistency counts as a validation failure and must be fixed before delivery.
````

### 2.3 Filling requirements

- This index **must be aggregated from the BL entries**; a separate hand-written list is not allowed, and any inconsistency with the BL entries counts as a validation failure.
- The four reverse index tables have the fixed columns `| Source Path | Symbol | Type | Covered BL |`, `| Table Name | Change Source | Covered BL |`, `| Entry Identifier | Type | Covered BL |`, and `| Configuration Item | Covered BL |`. Do not add, remove, or rename columns.
- The "Covered BL" column must hold links to BL entries; when the same source path is covered by several BLs, **list one row per BL** instead of packing several IDs into one row.
- The "Uncovered Source Files" table lists only **important files** (entry points, services, data access, configuration, migration scripts) with the reason they are uncovered; dependency directories, build artifacts, caches, and generated files are not listed.
- **Update it together during incremental sync**: after adding or deleting a BL entry, or after changing its evidence sections, the index must be re-aggregated; sync mode writes only the knowledge layer and does not touch the 7 traditional documents.
- Never present an inferred attribution as a confirmed one: reference relations that cannot be confirmed from code belong in "Coverage and Limitations" with the basis for the inference.

---

## 3. Business Logic Entry Template `business/BL-<NNN>-<business-name>.md`

### 3.1 Output path and numbering rules

- English: `Doc/<project-name>/en/business/BL-<NNN>-<business-name>.md`
- Chinese: `Doc/<项目名称>/cn/business/BL-<NNN>-<业务名称>.md`
- The same business logic uses the **same number** in every language.

**BL numbering rules**: `BL-` plus a three-digit number, increasing by **order of first creation**: `BL-001`, `BL-002`, `BL-003`, …

- Numbers are **never reused and never reordered**: once an entry's number is assigned it never changes, no matter how entries are added, deleted, or reclassified.
- When an entry is deleted its number is retired; a new entry takes the **smallest number currently unused** (for example, if `BL-003` was retired together with its entry, the next new entry uses `BL-003`).
- **No numbering by business domain**: the business domain is metadata, written in the `**Business Domain**: <domain name>` field of the BL entry header, and plays no part in numbering.
- When a business domain is reclassified, **change only the `**Business Domain**` field and never the ID**; when the knowledge map displays entries grouped by domain, group headings must not carry a number range.

### 3.2 Template body

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
`Current` / `⚠ Possibly Stale` / `Not Found (no Git baseline)` (choose exactly one; determined mechanically from Git)

## 15. Last Verified Version
- Git commit: `<commit hash>`
- Verification date: YYYY-MM-DD
````

### 3.3 Filling requirements and good/bad examples per section

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
| 14 | Freshness Status | Determined mechanically from Git, choose exactly one: `Current` / `⚠ Possibly Stale` / `Not Found (no Git baseline)`; **never write it merged with the evidence status**, and never substitute a model judgment for the Git determination. |
| 15 | Last Verified Version | Git commit + verification date (per-entry baseline); write "Not Found (no Git baseline)" when there is no Git. |

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

**Determined mechanically from Git, not judged by the model**; only three values exist:

| Freshness status | Meaning |
|------------------|---------|
| `Current` | None of the files and symbols related to the entry's evidence changed between the "Last Verified Version" commit and the current state (HEAD + working tree) |
| `⚠ Possibly Stale` | Something in that scope did change, and the entry was not re-verified in this run |
| `Not Found (no Git baseline)` | No Git or no valid HEAD, so freshness cannot be determined (see 4.4) |

**Step 1: Determine the change file set (uncommitted changes must be included)**

Change file set = **committed part** ∪ **working-tree part**:

| Part | Command | Scope |
|------|---------|-------|
| Committed part | `git diff --name-status -M <base>..HEAD` | Changes committed since the baseline; `-M` detects renames |
| Working-tree part | `git status --porcelain` | Staged, unstaged, and untracked (`??`) changes |

> **Never look only at `<base>..HEAD`**: code may already be changed but not yet committed, so comparing commits alone misses staleness; the working-tree part must be included.
> Untracked files count only when they fall inside the analysis scope; entries ignored by `.gitignore` and default skipped directories (dependencies, build artifacts, caches, etc.) are skipped.

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

Any layer hit and the entry was not re-verified in this run → set freshness to `⚠ Possibly Stale`; no layer hit → freshness is `Current`.

Supporting commands: `git rev-parse HEAD` (current baseline), `git log --oneline <base>..HEAD` (change overview), `git status --porcelain` (uncommitted changes), `git rev-list --count HEAD` (fourth part of the version number), `git log -1 --format=%an` (Git commit author).

### 4.3 The two fields are independent

- Evidence status and freshness status are **two mutually independent fields** that **must never be conflated or written merged**: `Verified` means "the implementation was read at the time", not that the content is still fresh; `⚠ Possibly Stale` means "the code has changed", not that the original evidence was wrong.
- The knowledge map shows them in **two separate columns**: `| ID | Business Capability | Description | Evidence Status | Freshness Status |`; the evidence status column holds only one of the four values, and the freshness status column holds only one of the three values.
- **Never** merge the two statuses into one column or one line: do not join the evidence status and the freshness status with a prefix, a slash, parentheses, or similar — each must occupy its own column or its own section.
- Inside a BL entry the two are likewise written in separate sections: "13. Evidence Status" and "14. Freshness Status".

### 4.4 Degradation without Git or without a valid HEAD

- Perform no freshness determination; record freshness as `Not Found (no Git baseline)`.
- State in the "Verification Baseline" section of the knowledge map that freshness determination is unavailable and that **every run requires a full re-verification**.
- **Never fabricate a commit hash**, and never substitute a date or a file timestamp for a commit.

### 4.5 Incremental maintenance baseline

- **Global baseline**: written in the "Verification Baseline" section of `00-Project Knowledge Map.md`, with the fields `last_verified_commit` + verification date.
- **Per-entry baseline**: written in the "Last Verified Version" section of each BL entry, with the fields Git commit + verification date.
- During an incremental update, re-verify only the affected entries (determined by the multi-layer analysis in 4.2); keep the "Last Verified Version" of unaffected entries at its original value instead of refreshing it to the current HEAD.
- After every incremental analysis, record in "Most Recent Incremental Analysis" of the knowledge map's "Verification Baseline" section: baseline commit, analysis date, changed-file count (including uncommitted), affected BL, re-verified BL, and BL still marked stale.
- The change file set must include uncommitted changes (`git status --porcelain`); **never compare only `<base>..HEAD`**.

### 4.6 Prohibitions

- **Never package model inference as fact**: inferred content must be explicitly labeled `Status: Inferred` with its basis, and assertive wording such as "the system will" or "it must be" is not allowed.
- Do not fabricate business facts, commit hashes, table names, APIs, test cases, or business rules; freshness determination must come from real Git command output, and **fabricating a commit hash in a freshness determination is prohibited**.
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

- All examples in the templates of this file (including the "cancel order" call chain, the source index example rows, `src/order/service.ts` → `cancelOrder()`, `orders.status: PENDING_PAYMENT → CANCELLED`, and the test file example) are **Example (placeholder, not project fact)**.
- In generated documents, wherever example content remains, the label line must remain as well: `> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.`
- Replace examples with real analysis results during generation; **never** write template examples into a delivered document as project facts.
