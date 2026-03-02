---
name: event-driven-design-skill
description: Use this skill to produce the Event-Driven Architecture section of a Solution Intent, or as a standalone event design analysis. Trigger on events, messaging, Kafka, topics, pub/sub, event streaming, event sourcing, CQRS, async workflows, or domain events. Also auto-triggered by the architect orchestrator for solutions with async communication. Kafka is the preferred platform. Enforces enterprise event standards; flags deviations.
---

# Event-Driven Design Skill — Enterprise Architecture Standards

## Purpose
Produce a complete, opinionated **Event-Driven Architecture** section for a Solution Intent,
or a standalone event design analysis. Output is Confluence-ready Markdown.

---

## Standards Source

**Always check `references/` before generating output.**

```
references/
├── README.md                ← Start here — lists all available standards files
├── event-standards.md       ← Kafka cluster policy, retention, topic conventions (when provided)
├── cloudevents-schema.md    ← enterprise CloudEvents envelope spec (when provided)
├── asyncapi-template.yaml   ← Standard AsyncAPI 2.x template (when provided)
├── schema-registry-guide.md ← Schema Registry location and conventions (when provided)
└── kafka-naming.md          ← Topic and consumer group naming rules (when provided)
```

- **If standards files are present:** Load them and use as the authoritative source.
  Cite the specific file/section when flagging a deviation.
- **If standards files are absent:** Use the enterprise defaults below.

---

## Messaging Platform

**Kafka is the preferred and default platform for all new event-driven designs.**

Flag any non-Kafka proposal with a deviation block and recommend migration or justification.

---

## Enterprise Defaults (used when references/ is not yet populated)

### Event Envelope — CloudEvents 1.0
All events must conform to CloudEvents 1.0:
```json
{
  "specversion": "1.0",
  "id": "uuid-v4",
  "source": "/jpmc/[domain]/[service]",
  "type": "com.jpmc.[domain].[entity].[past-tense-verb]",
  "time": "ISO-8601",
  "datacontenttype": "application/json",
  "data": { }
}
```

### Naming Conventions
**Event type:** `com.jpmc.[domain].[entity].[past-tense-verb]`
- `com.jpmc.payments.transaction.settled`
- `com.jpmc.accounts.account.created`

**Kafka topic:** `[domain].[entity].[event-type].[version]`
- `payments.transaction.settled.v1`
- `accounts.account.state.v2`

**Consumer group:** `[consuming-service].[topic]`

### Schema Registry
- All schemas registered in enterprise Schema Registry (Confluent-compatible)
- Avro preferred; JSON Schema acceptable
- BACKWARD compatibility required; FULL preferred
- Breaking changes require new topic version

### Consumer Requirements
- **Consumer groups** — each service has its own named group
- **At-least-once delivery** — idempotent consumers required; deduplicate on `event.id`
- **Dead Letter Topic (DLT)** — required for every consumer: `[topic].DLT`
- **Poison pill** — log, alert, route to DLT; never block a partition

### Ordering
- Partition key = entity ID for per-entity ordering
- No global ordering across partitions — document this explicitly
- If cross-entity ordering needed: flag and recommend saga/process manager pattern

### Retention
- Operational events: 7 days
- Audit/compliance events: 30 days (flag these explicitly)
- State/snapshot topics: compacted

---

## Deviation Flag Format

```markdown
> ⚠️ **Enterprise Pattern Deviation — [Category]**
> **Proposed:** [what the solution proposes]
> **Standard:** [what is required — cite references/ file if available]
> **Risk:** [data loss, compliance gap, ops burden]
> **Recommendation:** [remediation or approval path]
```

---

## Output Format

Confluence-ready Markdown.
- **Standalone:** full section with `## Event-Driven Architecture` heading
- **Orchestrated:** section body only (no top-level heading)

### Section Template

```markdown
## Event-Driven Architecture

### Event Design Rationale
[Why async/event-driven? What problems does it solve for this solution?]

### Event Taxonomy
| Event Name | Source Service | Topic | Consumers | Trigger |
|------------|---------------|-------|-----------|---------|
| com.jpmc.[domain].[entity].[verb] | [service] | [topic.v1] | [services] | [what causes it] |

### Event Schema (CloudEvents 1.0)
[Sample CloudEvents envelope for primary domain event(s)]

> ⚠️ [Deviation block if applicable]

### Topic Design
| Topic | Partitions | Retention | Partition Key | Compacted |
|-------|-----------|-----------|--------------|-----------|
| [topic.v1] | [n] | [7d/30d] | [entity id] | Yes/No |

### Consumer Design
| Consumer Group | Service | Idempotency Key | DLT |
|---------------|---------|-----------------|-----|
| [group-id] | [service] | event.id | [topic].DLT |

### Failure Handling
[DLT strategy, poison pill handling, retry policy, alerting on DLT growth]

### Ordering & Consistency
[Partition key rationale, ordering guarantees stated explicitly, saga if needed]

### Schema Registry
[Format, compatibility mode, versioning plan, registry location]

### Open Event Design Questions
- [Unresolved decisions]
```

---

## Diagrams

### Event Flow Diagram (always include)
```
flowchart LR
    SvcA[Producer A]
    SvcB[Producer B]
    T1[Topic: domain.entity.event.v1]
    DLT[DLT: domain.entity.event.v1.DLT]
    ConsX[Consumer X]
    ConsY[Consumer Y]

    SvcA -->|publish| T1
    SvcB -->|publish| T1
    T1 -->|consume| ConsX
    T1 -->|consume| ConsY
    ConsX -->|failed| DLT
```

### Consumer Sequence Diagram (include for complex flows)
```
sequenceDiagram
    participant P as Producer
    participant K as Kafka Topic
    participant C as Consumer
    participant DLT as Dead Letter Topic

    P->>K: Publish CloudEvent (id, type, data)
    K->>C: Deliver (at-least-once)
    C->>C: Check idempotency (event.id)
    alt Already processed
        C-->>K: Ack (skip)
    else New event
        C->>C: Process
        C-->>K: Ack
    end
    alt Failure
        C->>DLT: Route to DLT + alert
    end
```

---

## Workflow

### Standalone
1. Check `references/` — load standards files if present
2. Extract domain, entities, event flows from description
3. Ask if missing: consumer services, ordering requirements, compliance data classification
4. Apply standards; flag deviations with ⚠️
5. Generate Markdown + both diagrams → present `.md`

### Orchestrated
1. Check `references/` — load standards files if present
2. Receive context brief; infer event surface
3. Return section body Markdown (no top-level heading)
