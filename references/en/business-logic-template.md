# Project Knowledge Layer Templates (v1.2.0)

> **Positioning**: The knowledge layer is the core; the 7 traditional documents (01–07) are only different presentation views of the knowledge layer. The 7 document templates are in `references/en/document-templates.md`, and their file names, numbering, and paths remain unchanged.
>
> **Traceability chain**:
>
> ```
> Business question → Business capability → Business rule → Entry/API → Call chain → Data changes → External dependencies → Source files/symbols → Tests
> ```
>
> **Author acquisition priority**: `SKILL.md`, section "Author Information Acquisition", is the single authority (four levels: (1) author explicitly provided by the user → (2) local default author `Soonkeira` (https://github.com/Soonkeira) → (3) Git commit author `git log -1 --format=%an` → (4) leave empty). Using a machine account name (`$env:USERNAME` / `whoami`) as the author is **strictly prohibited**.
>
> **Example labeling**: All example content in the templates of this file is placeholder content and must be explicitly labeled `Example (placeholder, not project fact)`; replace it with real analysis results when generating documents.
>
> **This file contains**:
> 1. Knowledge map template `00-Project Knowledge Map.md`
> 2. Business logic entry template `business/BL-<NNN>-<business-name>.md`
> 3. Status definitions and determination rules
> 4. Traceability good/bad examples
> 5. Example content labeling rules

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
| Freshness determination method | Intersect `git diff --name-status <last_verified_commit>..HEAD` with the file set listed under "Source Evidence" of each BL entry |

**Degraded form when there is no Git or no valid HEAD**:

| Field | Value |
|-------|-------|
| last_verified_commit | Not Found (no Git baseline) |
| Verification date | YYYY-MM-DD |
| Freshness determination available | No |
| Freshness determination method | No freshness determination; every run requires a full re-verification |

> Never fabricate a commit hash when there is no Git baseline.

## 3. Business Capability Index

> This section is the main body of the knowledge map. **The knowledge map must not be only a project introduction**; the business capability index must be its main body.
> The ID column must be a link to the BL entry; the Status column holds the evidence status, and when the entry's freshness status is possibly stale, prefix the evidence status with `⚠ Possibly Stale`.

### 3.1 <Business Domain One> (BL-001–BL-009)

| ID | Business Capability | Description | Status |
|----|---------------------|-------------|--------|
| [BL-001](business/BL-001-<business-name>.md) | <business capability name> | <one sentence on the problem this capability solves> | Verified |
| [BL-002](business/BL-002-<business-name>.md) | <business capability name> | <one sentence on the problem this capability solves> | ⚠ Possibly Stale Partially Verified |

### 3.2 <Business Domain Two> (BL-010–BL-019)

| ID | Business Capability | Description | Status |
|----|---------------------|-------------|--------|
| [BL-010](business/BL-010-cancel-order.md) | Cancel Order | <one sentence on the problem this capability solves> | Verified |

## 4. Common Business Question Index

| Business Question | Related Business Logic |
|-------------------|------------------------|
| How is this feature implemented? | [BL-001](business/BL-001-<business-name>.md) |
| Why can this not be deleted? | [BL-002](business/BL-002-<business-name>.md) |
| What happens after an order is cancelled? | [BL-010](business/BL-010-cancel-order.md) |
| Why does this status change? | [BL-010](business/BL-010-cancel-order.md) |
| What data does this API ultimately modify? | [BL-001](business/BL-001-<business-name>.md) |
| Where in the code is this business rule? | [BL-002](business/BL-002-<business-name>.md) |

## 5. Not Found / Inferred Entries Summary

| ID | Business Capability | Status | Reason and Limitations |
|----|---------------------|--------|------------------------|
| [BL-003](business/BL-003-<business-name>.md) | <business capability name> | Inferred | <inferred only from directory structure and naming; no call-chain evidence found> |
| [BL-004](business/BL-004-<business-name>.md) | <business capability name> | Not Found | <no entry point, test, or data access code found> |

## 6. Coverage and Limitations

- Covered: <business domains for which knowledge entries exist>
- Not covered: <business domains without entries yet, and why>
- Known limitations: <parts that cannot be confirmed from code, such as dynamic dispatch, reflection, config-driven branches, external system behavior>
````

### 1.3 Filling requirements

- The **Analysis Scope** must reflect what was actually scanned: whatever is listed as scanned must really have been read, and the reason for skipping sensitive files and dependency directories must be stated.
- The **Verification Baseline** is the global baseline; `last_verified_commit` and the verification date must come from real Git command output. Without Git, use the degraded form and state explicitly that "every run requires a full re-verification".
- The **Business Capability Index** is grouped by business domain, one table per group, with the fixed columns `| ID | Business Capability | Description | Status |`. Do not add, remove, or rename columns.
- Annotate each business domain heading with its number range (for example `(BL-010–BL-019)`) so numbering continuity can be checked.
- The **Common Business Question Index** must map real user questions to BL IDs, with links that jump directly to the corresponding entry.
- **Being only a project introduction is prohibited**: a knowledge map without a business capability index counts as incomplete and must not be delivered.

---

## 2. Business Logic Entry Template `business/BL-<NNN>-<business-name>.md`

### 2.1 Output path and numbering rules

- English: `Doc/<project-name>/en/business/BL-<NNN>-<business-name>.md`
- Chinese: `Doc/<项目名称>/cn/business/BL-<NNN>-<业务名称>.md`
- The same business logic uses the **same number** in every language.

**BL numbering rules**: `BL-` plus a three-digit number, allocated in segments by business domain:

| Business domain | Number range |
|-----------------|--------------|
| First business domain | `BL-001`–`BL-009` |
| Second business domain | `BL-010`–`BL-019` |
| Third business domain | `BL-020`–`BL-029` |
| Nth business domain (N ≥ 2) | `BL-{(N-1)*10}`–`BL-{(N-1)*10+9}` |

Once allocated, a range is **never reused and never reordered**; new business domains append a new range at the end, and numbers of deleted entries are not recycled.

### 2.2 Template body

> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.

````markdown
# BL-010 Cancel Order

**Project Name**: XXX Project
**Author**: <Author>
**Date**: YYYY-MM-DD
**Version**: 1.26.509.1234

## Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.26.509.1234 | 2026-05-09 | <Author> | Initial version |

> The header information does not count as one of the 14 fixed sections below.

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
`Verified` / `Partially Verified` / `Inferred` / `Not Found` (choose exactly one)

## 14. Last Verified Version
- Git commit: `<commit hash>`
- Verification date: YYYY-MM-DD
````

### 2.3 Filling requirements and good/bad examples per section

**The order and titles of the 14 fixed sections are verbatim; do not add, remove, rename, or reorder them.**

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
| 13 | Evidence Status | Choose exactly one: `Verified` / `Partially Verified` / `Inferred` / `Not Found`. |
| 14 | Last Verified Version | Git commit + verification date (per-entry baseline); write "Not Found (no Git baseline)" when there is no Git. |

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

## 3. Status Definitions and Determination Rules

### 3.1 Evidence status

**Judged by the model**; each BL entry and each index row in the knowledge map **must choose exactly one of four**:

| Evidence status | Meaning | Basis for determination |
|-----------------|---------|-------------------------|
| `Verified` | Directly confirmed from source code | Clear source evidence (file → symbol); call chain, data changes, and branches were all read in the corresponding implementation |
| `Partially Verified` | Main conclusions confirmed, some links unconfirmed | Core chain confirmed, but some branches, exceptions, external impacts, or data changes were not read in the implementation |
| `Inferred` | Inferred from naming, directory structure, configuration, or documentation | No direct implementation evidence found; inference is only possible from structure or naming, and the basis for inference must be stated |
| `Not Found` | No evidence found at all | No entry point, implementation, test, or data access code found |

### 3.2 Freshness

**Determined mechanically from Git diff, not judged by the model**; only two values exist:

| Freshness | Meaning |
|-----------|---------|
| `Current` | None of the files listed under the entry's "Source Evidence" were modified between the "Last Verified Version" commit and the current HEAD |
| `⚠ Possibly Stale` | At least one file in that set was modified, and the entry was not re-verified in this run |

**Determination procedure (per entry)**:

1. Take the commit in the entry's "Last Verified Version" as the baseline `<base>`.
2. Run `git diff --name-status <base>..HEAD` to obtain the list of files modified since the baseline.
3. **Intersect** that file list with the file set listed under the entry's "Source Evidence".
4. Non-empty intersection and the entry was not re-verified in this run → set freshness to `⚠ Possibly Stale`; empty intersection → freshness is `Current`.

Supporting commands: `git rev-parse HEAD` (current baseline), `git log --oneline <base>..HEAD` (change overview), `git rev-list --count HEAD` (fourth part of the version number), `git log -1 --format=%an` (Git commit author).

### 3.3 The two fields are independent

- Evidence status and freshness are **two mutually independent fields** and **must never be conflated**: `Verified` means "the implementation was read at the time", not that the content is still fresh; `⚠ Possibly Stale` means "the code has changed", not that the original evidence was wrong.
- The Status column of the knowledge map shows the **evidence status**; when the entry's freshness is possibly stale, prefix the evidence status with `⚠ Possibly Stale` (for example `⚠ Possibly Stale Partially Verified`).

### 3.4 Degradation without Git or without a valid HEAD

- Perform no freshness determination; record freshness as `Not Found (no Git baseline)`.
- State in the "Verification Baseline" section of the knowledge map that freshness determination is unavailable and that **every run requires a full re-verification**.
- **Never fabricate a commit hash**, and never substitute a date or a file timestamp for a commit.

### 3.5 Incremental maintenance baseline

- **Global baseline**: written in the "Verification Baseline" section of `00-Project Knowledge Map.md`, with the fields `last_verified_commit` + verification date.
- **Per-entry baseline**: written in the "Last Verified Version" section of each BL entry, with the fields Git commit + verification date.
- During an incremental update, re-verify only the entries changed since the baseline (determined by the intersection rule in 3.2); keep the "Last Verified Version" of unchanged entries at its original value instead of refreshing it to the current HEAD.

### 3.6 Prohibitions

- **Never package model inference as fact**: inferred content must be explicitly labeled `Status: Inferred` with its basis, and assertive wording such as "the system will" or "it must be" is not allowed.
- Do not fabricate commit hashes, table names, APIs, test cases, or business rules.
- Do not delete the existing 7 document templates, perform large-scale renames, or introduce runtime dependencies; do not introduce a database, web UI, RAG, or vector database.

---

## 4. Traceability Good/Bad Examples

### 4.1 Business rules

- Bad: "The system checks user permissions."
- Good: "The system checks in `PermissionService.checkPermission()` whether the current user has the operation permission for that resource."

### 4.2 Call chain

- Bad: "Call the order service to complete the cancellation."
- Good: `OrderController.cancel()` → `OrderService.cancelOrder()` → `Order.cancel()` → `InventoryService.release()` → `OrderRepository.save()`

### 4.3 Data changes

- Bad: "Update the order status."
- Good: `orders.status: PENDING_PAYMENT → CANCELLED`, with `orders.updated_at` updated in the same transaction.

### 4.4 Tests

- Bad: "Already covered by unit tests."
- Good: "`tests/order/cancel.spec.ts` → `should cancel pending order` covers the normal cancellation path." (When no test is found, write "No corresponding automated tests found.")

### 4.5 Labeling inference

- When only structural inference is possible (for example, seeing an `XxxRepository` name without reading its implementation), explicitly label it `Status: Inferred` and state the basis (directory structure / naming / configuration item / migration script).
- Inferred content must not be placed in the `path → symbol` column of the "Source Evidence" table as if it were direct evidence; if it must be kept, put it on a separate line labeled "(inferred)".

---

## 5. Example Content Labeling Rules

- All examples in the templates of this file (including the "cancel order" call chain, `src/order/service.ts` → `cancelOrder()`, `orders.status: PENDING_PAYMENT → CANCELLED`, and the test file example) are **Example (placeholder, not project fact)**.
- In generated documents, wherever example content remains, the label line must remain as well: `> Example (placeholder, not project fact). Replace it with real analysis results when generating documents.`
- Replace examples with real analysis results during generation; **never** write template examples into a delivered document as project facts.
