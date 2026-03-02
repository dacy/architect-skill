---
name: database-design-skill
description: Use this skill for database selection and design. Trigger on database selection, which database, data storage, SQL vs NoSQL, database design, schema design, or any solution requiring persistent storage. Applies the company's approved database list and exception process. Use this skill for every data persistence decision — technology must be aligned before design proceeds.
---

# Database Design Skill

## Overview

This skill guides database selection and schema design aligned to the company's approved
database standards. Selection is not open-ended — there is an opinionated set of approved
databases and a formal exception process for anything outside it.

Read `references/database-standards.md` for the approved database catalog, selection
criteria, and exception process before making any recommendation.

---

## Output

- **Markdown (.md)** — Database Design section for Solution Intent, or standalone analysis
- Name file: `[initiative-name]-database-design.md`

---

## Process

### Step 1: Understand data requirements

For each data entity or aggregate:
- Volume: expected record count, growth rate
- Access pattern: read-heavy, write-heavy, mixed
- Query pattern: key-value lookup, relational joins, full-text search, time-series, graph
- Consistency requirement: strong, eventual, or relaxed
- Retention: how long must data be kept?
- Sensitivity: PII, PCI, confidential?

### Step 2: Database selection

Map requirements to the approved database catalog (`references/database-standards.md`):

| Requirement | Approved Option(s) |
|-------------|-------------------|
| Relational / transactional | [Company-approved RDBMS] |
| Key-value / caching | [Company-approved cache] |
| Document store | [Company-approved option] |
| Time-series | [Company-approved option] |
| Search | [Company-approved option] |
| Graph | [Company-approved option] |

If the required capability is not covered by an approved database, initiate the exception
process (see `references/database-standards.md` — Exception Process).

### Step 3: Schema design (for relational stores)

- Define tables, columns, data types, and constraints
- Identify primary keys, foreign keys, and indexes
- Apply normalization appropriate to the access pattern
- Flag any denormalization decisions with justification

### Step 4: Data at rest protection

For each database:
- Encryption at rest: required for all stores (document algorithm and key management)
- Access control: who/what can connect? (service accounts, no shared credentials)
- Backup and recovery: RPO and RTO targets
- See `security-skill` for full data protection patterns

### Step 5: Multi-tenancy (if applicable)

- Is data shared across tenants, partitioned by schema, or separated by database?
- Document the tenancy model and its impact on access control and backup

---

## Document Structure

```
# [Initiative Name] — Database Design

## Data Requirements Summary
[Table: Entity | Volume | Access Pattern | Sensitivity]

## Database Selection
[Approved database chosen per entity, with justification]

## Schema Design
[Tables/collections/schemas as appropriate]

## Data at Rest Protection
[Encryption, access control, backup/recovery]

## Exception Requests
[Any non-approved technology with justification and approval path]

## Open Questions
```

---

## Reference Files

- `references/database-standards.md` — Company approved database catalog, selection
  criteria per use case, exception process, and data at rest requirements.
  **TODO:** Populate with your organization's approved database list and standards.
