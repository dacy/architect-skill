# Database Standards

> **TODO:** Replace this placeholder with your organization's approved database catalog and standards.
> This file is read by the `database-design-skill` to enforce approved technology selection.

---

## Approved Database Catalog

> **TODO:** Fill in the approved databases for each use case.

| Use Case | Approved Technology | Version | Notes |
|----------|-------------------|---------|-------|
| Relational / transactional | [e.g., PostgreSQL, Aurora] | [version] | [constraints] |
| Key-value / caching | [e.g., ElastiCache Redis] | [version] | [constraints] |
| Document store | [e.g., DynamoDB, MongoDB Atlas] | [version] | [constraints] |
| Time-series | [e.g., Timestream, InfluxDB] | [version] | [constraints] |
| Full-text search | [e.g., OpenSearch] | [version] | [constraints] |
| Graph | [e.g., Neptune] | [version] | [constraints] |
| In-memory (session) | [e.g., ElastiCache Redis] | [version] | [constraints] |

---

## Selection Criteria

> **TODO:** Define how to choose between approved options when multiple apply.

For relational workloads, prefer [X] when:
- [criteria]

For document workloads, prefer [X] when:
- [criteria]

---

## Data at Rest Requirements

> **TODO:** Define encryption and key management requirements.

| Data Classification | Encryption Required | Key Management |
|--------------------|---------------------|----------------|
| PCI / Cardholder data | AES-256 mandatory | [Company HSM / KMS] |
| PII | AES-256 mandatory | [Company KMS] |
| Internal / Confidential | AES-256 required | [Company KMS] |
| Public | Encryption recommended | [Standard KMS] |

---

## Backup & Recovery Requirements

| Tier | Backup Frequency | Retention | RTO | RPO |
|------|-----------------|-----------|-----|-----|
| Tier 1 (Critical) | Continuous + daily | [period] | [X hrs] | [X min] |
| Tier 2 (Important) | Daily | [period] | [X hrs] | [X hrs] |
| Tier 3 (Standard) | Weekly | [period] | [X hrs] | [X hrs] |

---

## Exception Process

> **TODO:** Document the exception process for non-approved databases.

1. Submit exception request to: [Architecture Review Board / form link]
2. Required information: use case, why approved options don't fit, risk assessment
3. Review SLA: [X business days]
4. Approval required from: [Architecture lead / CTO / committee]

Exceptions are time-limited and require annual renewal.
