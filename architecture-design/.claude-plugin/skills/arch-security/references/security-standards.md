# Security Standards

> **TODO:** Replace this placeholder with your organization's security standards and requirements.
> This file is read by the `security-skill` to enforce approved security patterns.

---

## Data Classification Tiers

> **TODO:** Define data classification tiers and required controls per tier.

| Tier | Description | Examples | Controls Required |
|------|-------------|---------|-------------------|
| PCI / Cardholder | Payment card data | Card numbers, CVV | PCI DSS — see PCI Patterns section |
| PII | Personal identifiable info | Name, email, SSN | Encryption at rest + in transit, access logging |
| Confidential | Business sensitive | Trade secrets, contracts | Encryption at rest + in transit |
| Internal | Internal use only | Architecture docs, configs | TLS in transit; encryption at rest recommended |
| Public | No restriction | Marketing content | TLS in transit |

---

## Data in Transit Standards

> **TODO:** Define approved protocols and TLS requirements.

- **Minimum TLS version:** TLS 1.2 (TLS 1.3 preferred)
- **Approved cipher suites:** [List or reference to company policy]
- **Certificate authority:** [Internal CA name + external CA for public endpoints]
- **Certificate rotation:** [Rotation period + automated tooling]
- **mTLS required for:** [inter-service calls, Kafka, all internal APIs]

---

## Data at Rest Standards

> **TODO:** Define encryption and key management requirements.

- **Encryption algorithm:** AES-256
- **Key management:** [Company KMS solution — e.g., AWS KMS, HashiCorp Vault]
- **Key rotation:** [Rotation period per classification]
- **HSM required for:** [PCI scope, master key material]

---

## Kafka Security

> **TODO:** Define Kafka-specific security controls.

### Transport
- All Kafka connections use TLS — no plaintext
- SASL mechanism: [e.g., SASL/SCRAM, SASL/OAUTHBEARER]

### Authorization (ACLs)
- ACLs required on all topics in production
- Each service has its own service account (no shared Kafka credentials)
- Producers: WRITE permission to own topics only
- Consumers: READ permission to subscribed topics only
- Admin: only the platform team holds admin ACLs

### Message-Level Encryption (sensitive topics)
- Required for topics carrying PII or PCI data
- Encrypt payload before publish; decrypt after consume
- Approved envelope encryption approach: [describe or reference]

### Audit Logging
- All producer/consumer activity logged for PII/PCI topics
- Log destination: [Company SIEM]
- Retention: [period per classification]

### Topic Naming Convention
`[domain].[entity].[action]` — e.g., `payments.transaction.created`

---

## Database Protection

> **TODO:** Define database security controls.

- **Authentication:** Service accounts only; no human direct access to production DB
- **Credential management:** Stored in [approved secrets manager]; rotated every [X days]
- **Network:** Databases in isolated subnet; no public endpoints
- **Audit logging:** All DDL + privileged DML logged to [Company SIEM]
- **Backup encryption:** Same classification-appropriate encryption as primary store
- **Approved connection:** TLS required for all database connections

---

## PCI Patterns

> **TODO:** Define PCI compliance patterns and scope reduction strategies.

### Scope Assessment
Components that store, process, or transmit cardholder data are in PCI scope.

### Scope Reduction Strategies
- **Tokenization:** Replace card data with non-sensitive token; process tokens only
- **Payment processor offload:** Direct card entry to PCI-compliant processor iframe; never handle raw card data
- **Network segmentation:** Isolate PCI components in dedicated network segment with strict ingress/egress

### PCI-Specific Controls
- Network segmentation: dedicated VPC or subnet; strict security groups
- Key management: PCI-compliant HSM for encryption keys
- Logging: all access to cardholder data logged and retained for minimum 1 year
- Vulnerability scanning: quarterly + after significant changes
- Penetration testing: annual minimum

### Approved PCI Patterns
[TODO: List company-approved payment integration patterns here]

---

## API Gateway Connectivity Patterns

> **TODO:** Define approved API Gateway patterns and authentication flows.

| Access Pattern | Gateway | Auth Method | Additional Controls |
|----------------|---------|-------------|---------------------|
| External consumer (partner) | [External Gateway] | OAuth2 client credentials / API key | Rate limiting, WAF, IP allowlist |
| External consumer (end user) | [External Gateway] | OAuth2 PKCE | Rate limiting, WAF, CORS |
| Internal service → service | [Internal Gateway / service mesh] | mTLS / workload identity | No internet exposure |
| Mobile / browser | [External Gateway] | OAuth2 PKCE | WAF, bot protection |

### Rate Limiting
[TODO: Define default rate limits by access pattern]

### WAF Rules
[TODO: Reference company WAF rule set]

---

## Identity & Access Management

> **TODO:** Define IAM standards.

- **Human SSO:** [Company identity provider — e.g., Okta, Azure AD]
- **MFA:** Required for all human access; phishing-resistant (FIDO2/WebAuthn) for privileged
- **Service identity:** [Workload identity approach — e.g., IRSA, Workload Identity]
- **Secrets management:** [Approved tool — e.g., AWS Secrets Manager, Vault]
- **Privileged access:** Break-glass process only; all access logged; session recording
- **Access review:** Quarterly access review required for all production systems
