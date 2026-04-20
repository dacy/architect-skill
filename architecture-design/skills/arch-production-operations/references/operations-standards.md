# Production Operations Standards

> **TODO:** Replace this placeholder with your organization's production operations standards.
> This file is read by the `production-operations-skill` to enforce approved patterns.

---

## Observability Platform

> **TODO:** Define the company's approved observability tooling.

| Concern | Approved Tool | Notes |
|---------|--------------|-------|
| Metrics | [e.g., Datadog, CloudWatch] | [instrumentation library] |
| Distributed tracing | [e.g., Datadog APM, Jaeger, X-Ray] | [sampling rate] |
| Logging | [e.g., Datadog Logs, Splunk, CloudWatch Logs] | [log shipping method] |
| Alerting | [e.g., PagerDuty, OpsGenie] | [escalation policy] |
| Dashboards | [e.g., Datadog, Grafana] | [required dashboard template] |

---

## Logging Standards

- **Format:** Structured JSON — no unstructured plaintext logs in production
- **Required fields:** `timestamp`, `level`, `service`, `correlation_id`, `message`
- **Correlation ID:** Must be propagated across all service hops (via HTTP header / Kafka header)
- **Log levels:** ERROR for actionable failures, WARN for degraded state, INFO for key events, DEBUG for dev only (disabled in production)
- **PII / sensitive data:** Never log PII, credentials, or card data — mask or omit
- **Retention:** [Period per environment — e.g., 30 days dev, 90 days prod]

---

## Metrics Standards

### Required Metrics per Service (RED + USE)

| Metric | Description |
|--------|-------------|
| Request rate | Requests per second |
| Error rate | Errors per second (and as % of requests) |
| Latency | p50, p95, p99 response time |
| CPU utilization | Per instance / pod |
| Memory utilization | Per instance / pod |
| Queue / backlog depth | For async consumers |
| Connection pool saturation | For DB and external connections |

---

## SLO Standards

> **TODO:** Define default SLO targets by service tier.

| Service Tier | Availability Target | Latency p99 Target | Error Budget |
|-------------|--------------------|--------------------|--------------|
| Tier 1 (Critical) | 99.9% / 30d | < [X]ms | 43.8 min/month |
| Tier 2 (Important) | 99.5% / 30d | < [X]ms | 3.6 hrs/month |
| Tier 3 (Standard) | 99.0% / 30d | < [X]ms | 7.2 hrs/month |

---

## Alerting Standards

- **P1 (Critical):** Page on-call immediately; auto-escalate after [X] min if not acknowledged
- **P2 (High):** Page on-call; 15 min response SLA
- **P3 (Medium):** Ticket created; next business day response
- **Alert fatigue:** No noisy alerts — every alert must be actionable with a runbook
- **Runbook required:** All P1/P2 alerts must link to a runbook before production go-live

---

## Resiliency Patterns

> **TODO:** Define required resiliency patterns and configuration.

| Pattern | Requirement |
|---------|------------|
| Circuit breaker | Required on all synchronous downstream calls |
| Timeout | Every external call must have an explicit timeout — no indefinite waits |
| Retry | Exponential backoff with jitter; max [3] retries; retry only on transient errors |
| Bulkhead | Separate thread pools for critical vs non-critical dependencies |
| DLQ | Required for all Kafka consumers; DLQ monitored and alerted |
| Graceful degradation | Define fallback behavior for each dependency failure |

### Approved Libraries / Configuration
[TODO: Document approved circuit breaker / retry libraries per language]

---

## Performance Standards

> **TODO:** Define performance testing requirements.

- **Load testing required before production:** Yes for all Tier 1 and Tier 2 services
- **Approved load testing tool:** [e.g., k6, Locust, JMeter]
- **Load test gate:** p99 latency within SLO target at 2x expected peak load
- **Performance baseline:** Captured in staging and stored in [location]

---

## Cost Optimization Standards

> **TODO:** Define cost tagging and optimization requirements.

### Required Resource Tags (all AWS resources)
| Tag | Values | Purpose |
|-----|--------|---------|
| `app-id` | [Application ID] | Cost allocation |
| `environment` | dev / test / staging / prod | Cost allocation |
| `team` | [Team name] | Chargeback |
| `project` | [Project / initiative name] | Budget tracking |

Untagged resources will be flagged and may be subject to automated shutdown in non-prod.

### Rightsizing
- Start with the smallest instance type that meets performance targets under load test
- Review instance utilization monthly; downsize if CPU/memory < [X]% average
- Reserved Instances / Savings Plans required for stable baseline compute (>6 months)

### Auto-scaling
- Horizontal scaling required for all stateless services (no manual scaling in production)
- Scale-in policy: gradual; protect against flapping
- Minimum instances for availability: [Tier 1: 3, Tier 2: 2, Tier 3: 1]
