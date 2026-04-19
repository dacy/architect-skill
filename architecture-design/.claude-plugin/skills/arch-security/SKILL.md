---
name: security-skill
description: Use this skill for security architecture design. Trigger on security design, data protection, encryption, PCI, Kafka security, API security, data at rest, data in transit, database protection, or any solution requiring security architecture. Covers transit encryption, data at rest, Kafka and database security, PCI patterns, and API Gateway connectivity. Always included in Solution Intent.
---

# Security Skill

## Overview

This skill designs security architecture across all layers of a solution: data in transit,
data at rest, messaging platform security, database protection, PCI compliance patterns,
and API Gateway connectivity. Security is designed in — not bolted on after.

Read `references/security-standards.md` for company security requirements, approved
encryption standards, PCI patterns, and API Gateway connectivity rules.

---

## Output

- **Markdown (.md)** — Security section for Solution Intent, or standalone analysis
- Name file: `[initiative-name]-security.md`

---

## Sections

### Data in Transit

All data moving between systems must be encrypted in transit.

| Connection | Protocol | TLS Version | Certificate Management |
|------------|----------|-------------|----------------------|
| External client → API Gateway | HTTPS | TLS 1.2+ | [Company-managed cert] |
| Internal service → service | mTLS / TLS | TLS 1.2+ | [Internal CA / service mesh] |
| Service → Database | TLS | TLS 1.2+ | [DB-specific cert] |
| Service → Kafka | TLS + SASL | TLS 1.2+ | [Kafka-specific config] |

See `references/security-standards.md` — Data in Transit for approved cipher suites
and certificate authority requirements.

### Data at Rest

| Data Store | Encryption | Key Management | Classification |
|------------|-----------|----------------|----------------|
| [Database] | AES-256 | [Company KMS / HSM] | [PII / PCI / Internal] |
| [File storage] | SSE | [Company KMS] | [Classification] |
| [Kafka topic] | [Encryption approach] | [Key management] | [Classification] |

Data classification drives encryption requirements — see `references/security-standards.md`
for classification tiers and required controls per tier.

### Kafka Data Protection

Kafka topics containing sensitive data require additional controls beyond transport encryption:

- **Topic-level ACLs:** producer and consumer access restricted by service identity
- **Message-level encryption:** required for PII/PCI data payloads (encrypt before publish)
- **Schema registry security:** schemas for sensitive topics access-controlled
- **Audit logging:** all producer/consumer activity logged for sensitive topics
- **Data retention:** retention period aligned to data classification and legal requirements

See `references/security-standards.md` — Kafka Security for approved ACL patterns
and message-level encryption approach.

### Database Protection

- **Authentication:** service accounts only; no shared credentials; no direct user access
- **Authorization:** principle of least privilege — services granted only needed permissions
- **Network isolation:** databases in isolated subnet; no public access
- **Audit logging:** all DDL and privileged DML logged
- **Credential rotation:** automated rotation per company policy
- **Backup encryption:** backups encrypted with same or stronger controls as primary

See `references/security-standards.md` — Database Protection.

### PCI Compliance Patterns

If the solution handles payment card data:

- **Scope assessment:** which components are in-scope for PCI DSS?
- **Scope reduction:** architecture changes to minimize PCI scope (tokenization, offload to payment processor)
- **Network segmentation:** PCI components isolated in dedicated network segment
- **Key management:** PCI-compliant encryption key lifecycle
- **Logging and monitoring:** all access to cardholder data logged and monitored
- **Approved payment patterns:** see `references/security-standards.md` — PCI Patterns

Flag any in-scope PCI components explicitly with ⚠️ PCI.

### API Gateway Connectivity Patterns

For any API exposed externally or consumed by external systems:

| Pattern | Use Case | Required Controls |
|---------|----------|-------------------|
| External consumer → API Gateway | Third-party or partner access | OAuth2 / API key, rate limiting, WAF |
| Internal service → internal API | Service-to-service | mTLS / service identity, internal gateway |
| Mobile / browser → API | End-user facing | OAuth2 PKCE, CORS policy, WAF |

See `references/security-standards.md` — API Gateway Patterns for company-approved
authentication flows and gateway configuration standards.

### Identity & Access Management

- **Human access:** SSO via company identity provider; no local accounts
- **Service identity:** workload identity or service account per service
- **Privileged access:** break-glass process for production access; all access logged
- **Secret management:** secrets stored in company-approved secrets manager; never in code or config files

---

## Document Structure

```
# [Initiative Name] — Security Design

## Data Classification
[What data this solution handles and its classification]

## Data in Transit
[Connection table: protocol, TLS, cert management]

## Data at Rest
[Store table: encryption, key management, classification]

## Kafka Security (if applicable)
[ACLs, message encryption, audit]

## Database Protection
[Auth, authz, network isolation, audit, backup]

## PCI Scope (if applicable)
[In-scope components, scope reduction, approved patterns]

## API Gateway Connectivity
[External and internal access patterns]

## Identity & Access Management
[Human and service identity, secrets management]

## Security Risks
[Residual risks and mitigations]
```

---

## Reference Files

- `references/security-standards.md` — Company security requirements: approved encryption
  standards, Kafka security patterns, database protection controls, PCI patterns,
  API Gateway connectivity standards, and data classification tiers.
  **TODO:** Populate with your organization's security standards and requirements.
