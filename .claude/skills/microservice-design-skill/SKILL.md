---
name: microservice-design-skill
description: Use this skill to produce the Microservice Architecture section of a Solution Intent, or as a standalone service decomposition analysis. Trigger on microservices, service decomposition, bounded contexts, DDD, service mesh, inter-service communication, containerization, scheduled jobs, file processing, compute patterns, data access patterns, or breaking a monolith. Also auto-triggered by the architect orchestrator for multi-service solutions.
---

# Microservice Design Skill — Enterprise Architecture Standards

## Purpose
Produce a complete, opinionated **Microservice Architecture** section for a Solution Intent,
or a standalone service decomposition analysis. Output is Confluence-ready Markdown.

---

## Enterprise Microservice Standards (enforce these, flag deviations)

### Service Design Principles
- **Single Responsibility** — one bounded context per service; flag services that span contexts
- **Domain-Driven Design (DDD)** — decompose along business domain boundaries, not technical layers
- **12-Factor App** — required for all new microservices
  - Config via environment variables (never hardcoded)
  - Stateless processes; state in backing services
  - Logs as event streams (stdout → enterprise logging platform)
- **Loose coupling, high cohesion** — services own their data store; no shared databases

### Inter-Service Communication
| Pattern | Use Case | enterprise Preference |
|---------|----------|-----------------|
| REST (HTTP/JSON) | Synchronous, external-facing | ✅ Standard |
| gRPC | High-throughput internal sync calls | ✅ Preferred for internal |
| Kafka events | Async, decoupled workflows | ✅ Preferred for async |
| Direct DB sharing | Cross-service data access | ❌ Prohibited — flag immediately |
| Synchronous chains > 3 hops | Deep call chains | ⚠️ Flag — prefer async |

### Service Mesh
- **Istio** — preferred service mesh
- All inter-service traffic routes through Envoy sidecar proxy
- **mTLS mandatory** for all service-to-service communication
- Traffic policies, circuit breakers, and retries configured via Istio VirtualService/DestinationRule
- Observability: traces via Jaeger/Zipkin, metrics via Prometheus

### Deployment & Runtime
- **Containers (Docker)** — all services containerized
- **Kubernetes (enterprise Kafka cluster)** — orchestration standard
- **Helm charts** — packaging standard deployments
- Resource limits and requests must be defined (no unbounded containers)
- **Liveness + Readiness probes** — required on all services
- **Namespace isolation** — services scoped to domain namespace

### Data Ownership
- Each service owns its own data store — no cross-service DB access
- **Database per service** pattern enforced
- Allowed DB types: PostgreSQL, Oracle (legacy), Redis (cache), Cassandra (high-scale)
- Cross-service data needs: use API calls or events — never direct DB joins

### Resilience Patterns (required)
- **Circuit breaker** — via Istio or Resilience4j; required for all external/internal calls
- **Retry with exponential backoff** — max 3 retries, jitter applied
- **Timeout** — all outbound calls must have explicit timeout configured
- **Bulkhead** — thread pool isolation for critical downstream dependencies

### ⚠️ Flag These Anti-Patterns
- Shared databases across services
- Synchronous call chains > 3 hops deep
- Services that span multiple bounded contexts ("mega-services")
- Missing circuit breakers on downstream calls
- Hardcoded configuration (violates 12-factor)
- Containers without resource limits

---

## Output Format

Confluence-ready Markdown. Standalone = full section. Orchestrated = section body only.

### Section Template

```markdown
## Microservice Architecture

### Decomposition Rationale
[How were service boundaries determined? What DDD/domain analysis was applied?]

### Service Inventory
| Service Name | Bounded Context | Responsibility | Tech Stack | Owns Data Store |
|-------------|----------------|---------------|------------|-----------------|
| [svc-name] | [domain] | [what it does] | [lang/framework] | [DB type] |

### Inter-Service Communication Map
[Describe which services call which, and via what pattern (REST, gRPC, event)]

> ⚠️ **Enterprise Pattern Note:** [Flag any anti-patterns found — shared DB, deep chains, etc.]

### Service Mesh Configuration
[Istio setup, mTLS scope, traffic policies, circuit breaker configs]

### Data Ownership
| Service | Data Store | Type | Shared? |
|---------|-----------|------|---------|
| [svc] | [db-name] | [PostgreSQL/Redis/etc] | No (flag if Yes) |

### Resilience Design
| Service | Circuit Breaker | Timeout | Retry Policy | Bulkhead |
|---------|----------------|---------|--------------|----------|
| [svc] | Yes (Istio) | [Xms] | [3x, exp backoff] | Yes/No |

### Deployment Topology
[Kubernetes namespaces, node affinity, replica counts, HPA configuration]

### 12-Factor Compliance Checklist
| Factor | Compliant | Notes |
|--------|-----------|-------|
| Config via env vars | ✅/⚠️ | |
| Stateless processes | ✅/⚠️ | |
| Logs as streams | ✅/⚠️ | |
| [continue for relevant factors] | | |

### Open Microservice Design Questions
- [Unresolved decisions]
```

---

## Diagrams

### Service Topology Diagram
Always include. Shows services, their communication patterns, and data stores.

```
flowchart TD
    subgraph Domain A
        SvcA[Service A\nREST API]
        DBA[(DB A\nPostgreSQL)]
        SvcA --- DBA
    end

    subgraph Domain B
        SvcB[Service B\ngRPC]
        DBB[(DB B\nRedis)]
        SvcB --- DBB
    end

    subgraph Messaging
        Topic[Kafka Topic\ndomain.entity.event.v1]
    end

    Client([External Client]) -->|REST| SvcA
    SvcA -->|gRPC| SvcB
    SvcB -->|publish| Topic
    Topic -->|consume| SvcA
```

### Sequence Diagram (for key cross-service flows)
Include for the primary business flow to show inter-service orchestration.

---

## Workflow

### Standalone invocation
1. Extract services, domains, and interactions from user's description
2. Ask for: team ownership if known, existing stack constraints, cloud/k8s environment
3. Apply s; flag anti-patterns with ⚠️ callouts
4. Generate full Markdown section + topology diagram
5. Present `.md` file

### Orchestrated invocation
1. Receive context brief from architect skill
2. Infer service boundaries from solution description
3. Apply same standards enforcement
4. Return section body Markdown (no top-level `#` heading)

---

## enterprise Deviation Flag Format

```markdown
> ⚠️ **Enterprise Pattern Deviation — [Category]**
> **Proposed:** [what the solution proposes]
> **enterprise Standard:** [what is preferred]
> **Risk:** [coupling, data integrity, ops risk]
> **Recommendation:** [remediation path or approval required]
```
