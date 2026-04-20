# Data Architecture Standards

> **TODO:** Replace this placeholder with your organization's data architecture standards.
> This file is read by the `data-architecture-skill` to enforce approved data patterns.

---

## Data Consumption Patterns

> **TODO:** Define approved patterns for how systems consume data from each other.

| Pattern | When Approved | Notes |
|---------|--------------|-------|
| REST API (synchronous pull) | Standard for on-demand reads | [constraints] |
| Kafka event subscription | Standard for async / streaming | [constraints] |
| Direct DB read | Not approved without exception | [exception conditions] |
| File extract (batch) | Legacy / partner integration only | [file format standards] |
| CDC (Change Data Capture) | Analytical platform integration | [approved CDC tooling] |

---

## Caching Standards

> **TODO:** Define approved caching technology and patterns.

- **Approved distributed cache:** [e.g., ElastiCache Redis — version X.X]
- **Local / in-process cache:** Not approved for shared state
- **TTL policy:** All cache entries must have an explicit TTL (no indefinite caching)
- **Default pattern:** Cache-aside (lazy loading); document justification for others
- **Cache key conventions:** [naming convention]
- **Cache invalidation:** Event-driven invalidation preferred over TTL-only for correctness-critical data

---

## Analytical Platform Integration

> **TODO:** Define the company's analytical platform and how services publish to it.

- **Approved analytical platform:** [e.g., Snowflake, Redshift, Databricks]
- **Publishing mechanism:** [e.g., Kafka Connect → platform, Glue ETL, CDC]
- **Schema registry:** [where analytical schemas are registered]
- **Data quality gate:** [required validation before data enters the platform]
- **Latency tiers:**
  - Near-real-time: [X minutes — approved mechanism]
  - Hourly batch: [approved mechanism]
  - Daily batch: [approved mechanism]

---

## Data Publishing Standards

> **TODO:** Define how services publish data for other systems to consume.

- **Preferred mechanism:** Kafka events for real-time; REST API for on-demand
- **Schema format:** [e.g., Avro with Schema Registry, JSON Schema]
- **Schema registry:** [URL + governance process]
- **Breaking change policy:** [versioning and deprecation process]
- **Event naming convention:** `[domain].[entity].[action]` — e.g., `payments.transaction.created`

---

## Data Migration Patterns

> **TODO:** Define approved data migration approaches.

| Pattern | When to Use | Considerations |
|---------|------------|----------------|
| Big-bang | Small datasets, low risk | Requires maintenance window |
| Phased / incremental | Large datasets, high risk | Longer timeline, parallel run needed |
| Parallel run | Critical systems | Highest safety; double-write during transition |
| Strangler fig | Replacing legacy system | Gradual cutover by feature/domain |

**Required for all migrations:**
- Data validation script run before and after
- Rollback procedure documented and tested
- Go/no-go criteria defined before cutover begins
- Cutover rehearsal in staging environment

---

## Data Movement Tools

> **TODO:** Define approved tools for data movement.

| Use Case | Approved Tool | Notes |
|---------|--------------|-------|
| Batch ETL | [e.g., AWS Glue] | [constraints] |
| Stream processing | [e.g., Kafka Streams, Flink] | [constraints] |
| Database replication | [e.g., DMS] | [constraints] |
| File transfer | [e.g., SFTP + S3] | [standards] |
