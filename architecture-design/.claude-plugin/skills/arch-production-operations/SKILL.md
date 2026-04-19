---
name: production-operations-skill
description: Use this skill for production readiness design. Trigger on observability, monitoring, alerting, resiliency, performance design, cost optimization, SLO, SLA, or any solution being prepared for production. Covers the full production readiness picture — observability, resiliency, performance, and cost optimization. Always included in Solution Intent.
---

# Production Operations Skill

## Overview

This skill designs for production readiness across four dimensions: observability,
resiliency, performance, and cost optimization. Production standards must be addressed
before a solution is considered complete.

Read `references/operations-standards.md` for company tooling, SLO targets, approved
alerting platforms, and cost optimization patterns.

---

## Output

- **Markdown (.md)** — Production Operations section for Solution Intent, or standalone
- Name file: `[initiative-name]-production-operations.md`

---

## Sections

### Observability

**Logging**

| Component | Log Level | Log Format | Destination | Retention |
|-----------|-----------|-----------|-------------|-----------|
| [Service] | INFO / WARN / ERROR | [JSON structured] | [Company log platform] | [Period] |

Standards:
- Structured JSON logging required (no unstructured text logs in production)
- Correlation IDs propagated across all service calls
- No PII or sensitive data in logs
- See `references/operations-standards.md` — Logging Standards

**Metrics**

Key metrics to instrument per service:
- Request rate (requests/sec)
- Error rate (% errors by type)
- Latency (p50, p95, p99)
- Saturation (CPU, memory, queue depth, connection pool)

Platform: see `references/operations-standards.md` — Metrics Platform.

**Distributed Tracing**

- Traces required for all inter-service calls
- Sampling rate and approved tracing platform per `references/operations-standards.md`

**Alerting**

| Alert | Condition | Severity | Notification Channel | Runbook |
|-------|-----------|----------|---------------------|---------|
| High error rate | Error rate > X% for Y min | P1 | [PagerDuty / Slack] | [Link] |
| Latency breach | p99 > Xms for Y min | P2 | [Channel] | [Link] |
| [Custom alert] | [Condition] | [Sev] | [Channel] | [Link] |

Runbooks must exist before a service goes to production.

**SLO Definition**

| SLO | Target | Measurement Window | Error Budget |
|-----|--------|-------------------|--------------|
| Availability | 99.9% | 30 days | 43.8 min/month |
| Latency (p99) | < Xms | 30 days | — |
| Error rate | < X% | 30 days | — |

---

### Resiliency

**Failure mode analysis**

For each external dependency, document the failure mode and mitigation:

| Dependency | Failure Mode | Mitigation Pattern |
|------------|-------------|-------------------|
| [Upstream API] | Unavailable / slow | Circuit breaker + timeout + retry with backoff |
| [Database] | Connection exhaustion | Connection pool limits + timeout |
| [Kafka] | Topic lag / broker failure | Consumer retry + DLQ |

**Resiliency patterns to apply:**

- **Circuit breaker:** on all synchronous downstream calls; trip at [X]% error rate
- **Retry with exponential backoff:** for transient failures; max [N] retries; jitter applied
- **Timeout:** every external call has an explicit timeout; no indefinite waits
- **Bulkhead:** isolate thread pools for critical vs. non-critical calls
- **Dead-letter queue (DLQ):** for all async consumers; DLQ monitored and alerted
- **Graceful degradation:** what does the service return if a dependency is unavailable?

**Recovery targets:**

| Metric | Target |
|--------|--------|
| RTO (Recovery Time Objective) | [X minutes] |
| RPO (Recovery Point Objective) | [X minutes] |

---

### Performance

**Baseline targets:**

| Operation | p50 Target | p99 Target | Throughput |
|-----------|-----------|-----------|------------|
| [API endpoint] | Xms | Xms | X req/s |
| [Background job] | — | — | X records/s |

**Performance design decisions:**

- Caching strategy: see `data-architecture-skill` output
- Database indexing: see `database-design-skill` output
- Async offload: which operations are async to protect response time?
- Horizontal scaling: what is the scaling unit and trigger?
- Load testing: what load test approach and tools are planned?

---

### Cost Optimization

**Cost drivers identification:**

| Component | Cost Driver | Estimated Cost Profile |
|-----------|-------------|----------------------|
| [Compute] | Instance type × hours | [High / Medium / Low] |
| [Storage] | GB stored × months | [High / Medium / Low] |
| [Data transfer] | GB egress | [High / Medium / Low] |
| [Kafka] | Partition-hours | [High / Medium / Low] |

**Optimization patterns:**

- Right-sizing: use the smallest instance type that meets performance targets
- Auto-scaling: scale down during off-peak; set minimum to availability requirement
- Reserved capacity: commit to reserved instances for stable baseline load
- Data lifecycle: archive or delete data that exceeds retention requirements
- Egress costs: minimize cross-AZ and cross-region data transfer where possible

See `references/operations-standards.md` — Cost Optimization for company tagging
requirements (cost allocation tags required on all resources).

---

## Document Structure

```
# [Initiative Name] — Production Operations

## SLO Targets
[Availability, latency, error rate]

## Observability
[Logging, metrics, tracing, alerting — with runbook links]

## Resiliency
[Failure mode table, patterns applied, RTO/RPO]

## Performance
[Baseline targets, design decisions]

## Cost Optimization
[Cost driver table, optimization patterns]

## Open Questions
```

---

## Reference Files

- `references/operations-standards.md` — Company observability tooling, SLO targets,
  alerting platform, resiliency pattern standards, and cost optimization requirements.
  **TODO:** Populate with your organization's production operations standards.
