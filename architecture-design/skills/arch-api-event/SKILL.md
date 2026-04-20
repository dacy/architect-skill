---
name: arch-api-event
description: Use this skill for API design and external access patterns. Trigger on API design, REST endpoints, API versioning, authentication patterns, rate limiting, contract design, inter-service API boundaries, external REST access, or API gateway patterns. Also auto-triggered by the architect orchestrator for any solution with API surface area. Enforces enterprise API standards; flags deviations.
---

# API Patterns Skill — Enterprise Architecture Standards

## Purpose
Produce a complete, opinionated **API Design** section for a Solution Intent, or a standalone
API pattern analysis. Output is Confluence-ready Markdown.

---

## Standards Source

**Always check `references/` before generating output.**

```
references/
├── README.md              ← Start here — lists all available standards files
├── api-standards.md       ← Preferred styles, versioning, auth (when provided)
├── error-codes.md         ← enterprise error code taxonomy (when provided)
├── gateway-policy.md      ← API Gateway routing and approval process (when provided)
└── openapi-template.yaml  ← Standard OpenAPI spec template (when provided)
```

- **If standards files are present:** Load them and use as the authoritative enforcement source.
  Cite the specific file/section when flagging a deviation (e.g. "per api-standards.md §3.2").
- **If standards files are absent:** Fall back to the enterprise defaults defined below.

---

## Enterprise Defaults (used when references/ is not yet populated)

### API Style Priority
1. **REST** — default for all external-facing and most internal APIs
2. **gRPC** — preferred for internal high-throughput, low-latency service-to-service
3. **AsyncAPI / CloudEvents** — for event-driven contracts (pair with event-driven-design skill)

### Versioning
- URI versioning (`/v1/`, `/v2/`) — preferred
- Never break existing versions without deprecation notice
- Deprecation headers: `Deprecation: true`, `Sunset: <date>`

### Authentication
- OAuth 2.0 for external-facing APIs
- mTLS for internal service-to-service
- All external traffic via API Gateway

### Error Format
```json
{
  "error": {
    "code": "DOMAIN_ERROR_NAME",
    "message": "Human-readable description.",
    "traceId": "uuid",
    "timestamp": "ISO-8601"
  }
}
```
Always include `traceId` for observability.

### Rate Limiting
- Enforce at API Gateway layer
- Standard headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- 429 with `Retry-After` on exhaustion

---

## Deviation Flag Format

```markdown
> ⚠️ **Enterprise Pattern Deviation — [Category]**
> **Proposed:** [what the solution proposes]
> **Standard:** [what is required — cite references/ file if available]
> **Risk:** [why this matters]
> **Recommendation:** [remediation path or approval required]
```

---

## Output Format

Confluence-ready Markdown.
- **Standalone:** full section with `## API Design` heading
- **Orchestrated:** section body only (no top-level heading)

### Section Template

```markdown
## API Design

### API Style & Rationale
[Identify style(s) and rationale. Flag non-standard choices.]

> ⚠️ [Deviation block if applicable]

### API Inventory
| API / Endpoint Group | Style | Consumer | Auth | Rate Limited |
|----------------------|-------|----------|------|--------------|
| /v1/[resource] | REST | [who] | OAuth 2.0 | Yes |

### Endpoint Design
#### [Resource Name]
- `GET /v1/[resource]` — [purpose]
- `POST /v1/[resource]` — [purpose]
- `GET /v1/[resource]/{id}` — [purpose]

**Sample Request / Response (with traceId):**
[representative JSON]

### Versioning Strategy
[Approach, current version, deprecation plan]

### Authentication & Authorization
[OAuth flows, mTLS scope, JWT claims, API Gateway routing]

### Rate Limiting & Quotas
[Limits, enforcement point, headers]

### Error Handling
[Error format, domain-specific error codes]

### API Contracts & Schema Registry
[OpenAPI location, Protobuf registry, AsyncAPI docs]

### Open API Questions
- [Unresolved decisions]
```

---

## Diagram

Always include a Mermaid sequence diagram for the primary API flow:

```
sequenceDiagram
    participant C as Client
    participant GW as API Gateway
    participant S as Service
    participant DB as Data Store

    C->>GW: Request (auth token)
    GW->>GW: Validate token + rate check
    GW->>S: Forward (mTLS)
    S->>DB: Query/write
    DB-->>S: Result
    S-->>GW: Response + traceId
    GW-->>C: Response
```

---

## Workflow

### Standalone
1. Check `references/` — load standards files if present
2. Extract API surface from description; ask for missing details
3. Apply standards; flag deviations with ⚠️
4. Generate Markdown + diagram → present `.md`

### Orchestrated
1. Check `references/` — load standards files if present
2. Receive context brief; infer API surface
3. Return section body Markdown (no top-level heading)
