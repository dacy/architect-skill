# Opinionated Tech Stack

> **TODO:** Replace this placeholder with your organization's approved technology catalog.
> This file is read by the `tech-stack-skill` to enforce approved technology selection.

---

## Approval Tiers

| Tier | Meaning |
|------|---------|
| **Approved** | Standard choice — use without additional review |
| **Conditional** | Approved for specific use cases only — confirm fit before using |
| **Exception Required** | Not on the list — requires Architecture Review Board approval |

---

## Backend / Runtime

| Technology | Version | Tier | Use Case / Constraints |
|-----------|---------|------|----------------------|
| [Language 1] | [x.x+] | Approved | [Primary language for services] |
| [Language 2] | [x.x+] | Conditional | [Only for specific use case] |

## Application Frameworks

| Technology | Version | Tier | Use Case / Constraints |
|-----------|---------|------|----------------------|
| [Framework 1] | [x.x+] | Approved | [e.g., primary API framework] |

## API Style

| Style | Tier | Use Case / Constraints |
|-------|------|----------------------|
| REST / JSON | Approved | Standard for all synchronous APIs |
| gRPC | Conditional | [When approved — e.g., high-throughput internal only] |
| GraphQL | Exception Required | [Constraints] |

## Messaging / Eventing

| Technology | Version | Tier | Use Case |
|-----------|---------|------|---------|
| [e.g., Kafka / MSK] | [x.x+] | Approved | Async events, streaming |

## Frontend / UI

| Technology | Version | Tier | Use Case |
|-----------|---------|------|---------|
| [e.g., React] | [18+] | Approved | MFE UI framework |
| [e.g., Angular] | [15+] | Conditional | [Constraints] |

## Infrastructure / Compute

| Technology | Tier | Use Case |
|-----------|------|---------|
| AWS EKS | Approved | Long-running containerized services |
| AWS Lambda | Approved | Event-triggered, short-duration functions |
| AWS ECS Fargate | Approved | Scheduled tasks, batch |

## CI/CD

| Technology | Tier | Use Case |
|-----------|------|---------|
| [e.g., GitHub Actions] | Approved | All pipelines |

## Observability

| Technology | Tier | Use Case |
|-----------|------|---------|
| [e.g., Datadog] | Approved | Metrics, tracing, logs |
| [e.g., PagerDuty] | Approved | Alerting |

---

## Exception Process

> **TODO:** Document the exception approval process.

1. Submit exception request to: [Architecture Review Board / form link]
2. Required information: proposed technology, why approved alternatives don't fit, risk
3. Review SLA: [X business days]
4. Approval required from: [Who]
