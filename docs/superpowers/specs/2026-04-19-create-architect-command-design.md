# Design Spec: `create-architect` Command
**Date:** 2026-04-19
**Status:** Approved

---

## Overview

The `create-architect` command is the primary entry point for the Architect AI plugin. It guides users through a structured, phased workflow to produce a comprehensive Solution Intent document, using parallel specialist agents for speed and depth. The design mirrors the feature-dev plugin's phased approach and adapts it for enterprise architecture work.

---

## Command Signature

```
/create-architect [initiative description] [--template <path>]
```

- **`initiative description`** — optional freeform text describing the initiative. If omitted, the orchestrator asks for it.
- **`--template <path>`** — optional path or URL to a custom output template. Overrides the plugin's built-in default at `references/solution-intent-template.md`. If neither is provided, the synthesized document structure is used as-is.

---

## Input Types

The command accepts any of three input forms. The orchestrator detects and handles each:

| Type | Example |
|------|---------|
| Free text | "We need a customer notification service" |
| Structured brief | Goals, constraints, non-goals already written out |
| URL / document reference | Confluence page, Jira epic, GitHub README |

---

## Phase Breakdown

### Phase 1 — Discovery
The orchestrator parses and classifies the input. Extracts:
- Initiative name
- Stated goals and constraints
- Any referenced systems or external URLs

### Phase 2 — Context Exploration *(parallel)*
One `arch-explorer` agent is spawned per referenced system or URL. Agents run concurrently. Each explores its target using the **Atlassian MCP** (Confluence pages, Jira epics) and **GitHub MCP** (READMEs, code, ADRs) as appropriate. Returns a structured context block:
- System name and tech stack
- Key APIs and interfaces exposed
- Data owned
- Upstream/downstream dependencies

### Phase 3 — Clarifying Questions
The orchestrator asks targeted questions one at a time. The number of questions is adaptive — more questions for complex or ambiguous initiatives, fewer for well-defined ones. Stops when it has enough context to proceed to design without significant assumptions.

### Phase 4 — High-Level Approaches *(user approval gate)*
`arch-strategist` presents 2–3 high-level architectural directions, each with:
- Core pattern (e.g., event-driven microservices, modular monolith, strangler fig)
- Trade-offs: complexity, team fit, time-to-market, operational cost
- Which stated constraints each option satisfies or violates
- Inline `arch-tech-stack` skill validation — flags any technology on the exception-required tier
- A clear recommendation with reasoning

**⏸ User selects an approach before Phase 5 starts.**

### Phase 5 — Domain Design *(blocking, user approval gate)*
`arch-agent-ddd` runs using the chosen direction and all gathered context. Applies the `arch-domain-driven-design` skill. Produces:

- **Bounded contexts** with clear ownership boundaries
- **Per-service descriptions**: name, responsibilities, data owned, key operations
- **ASCII connection diagram** showing services, sync/async relationships, and external integrations
- **Domain events** that cross context boundaries

Example diagram output:
```
[Mobile Client] ──HTTPS──► [API Gateway]
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
             [User Service] [Order Service] [Notification Service]
                    │            │
                    └────────────┘
                         │ Domain Events
                         ▼
                    [Event Bus]
                         │
                    [Audit Service]
```

**⏸ User approves service decomposition and high-level flow before Phase 6 starts.**

### Phase 6 — Parallel Deep Design *(parallel)*
The orchestrator determines which specialist agents to run based on confirmed scope. All selected agents run concurrently, each receiving the DDD output and full context as input.

| Agent | Skill | Key Outputs |
|-------|-------|-------------|
| `arch-agent-api-event` | `arch-api-event` | API styles, endpoint contracts, event schemas, versioning strategy |
| `arch-agent-data` | `arch-data` | Data ownership per service, storage choices, caching strategy, migration approach |
| `arch-agent-deployment` | `arch-deployment` | Compute pattern, IaC, CI/CD pipeline, environment strategy, network design |
| `arch-agent-security` | `arch-security` | AuthN/AuthZ model, secrets management, network controls, threat surface |
| `arch-agent-ops` | `arch-production-operations` | Observability stack, resiliency patterns, SLO targets, cost tagging strategy |

Each specialist agent invokes the `arch-tech-stack` skill inline whenever a technology selection is made. Technologies on the exception-required tier are flagged before being committed to the design.

### Phase 7 — Synthesis
`arch-synthesizer` merges all agent outputs into a single coherent Solution Intent draft. Responsibilities:
- Assembles outputs into the standard document template (see Output Format below)
- Detects and flags conflicts between specialist outputs (e.g., two agents picking incompatible messaging systems)
- Consolidates all technology decisions into Section 10

### Phase 8 — Review
`arch-reviewer` applies the `arch-review` skill as a cold peer review of the draft. Produces severity-flagged findings appended to Section 12 of the document:

| Severity | Meaning |
|----------|---------|
| 🔴 Critical | Blocks finalization — must be resolved |
| 🟡 Important | Should be addressed — user decides |
| 🟢 Minor | Suggestions only |

If 🔴 Critical findings exist, the orchestrator prompts the user to resolve them before proceeding.

### Phase 9 — Template Formatting *(optional)*
If a template is available (runtime `--template` path takes precedence over `references/solution-intent-template.md`), the `arch-template-formatter` skill reformats the approved Solution Intent to match the template structure. Skipped entirely if no template is configured.

### Phase 10 — Finalize
The orchestrator writes the final document to:
```
docs/<kebab-initiative-name>-solution-intent.md
```

---

## Output Format

The Solution Intent document follows this structure:

```
# Solution Intent: <Initiative Name>

## 1. Executive Summary
## 2. Goals & Constraints
## 3. High-Level Architecture
   - Chosen approach + rationale
   - ASCII service diagram
## 4. Domain Model & Service Decomposition
   - Bounded contexts
   - Per-service descriptions
## 5. API & Event Design
## 6. Data Architecture
## 7. Deployment & Infrastructure
## 8. Security Design
## 9. Production Readiness
## 10. Technology Decisions
    - Consolidated tech choices, tier flags noted
## 11. Open Questions & Risks
## 12. Review Findings
    - Severity-flagged findings from arch-reviewer
```

---

## Agent Inventory

| Agent | Phase | Parallelism | Skill(s) Used |
|-------|-------|-------------|---------------|
| `arch-orchestrator` | 1, 3, 6 dispatch, 9–10 | — | — |
| `arch-explorer` | 2 | Parallel (one per system) | — |
| `arch-strategist` | 4 | — | `arch-tech-stack` (inline) |
| `arch-agent-ddd` | 5 | — | `arch-domain-driven-design` |
| `arch-agent-api` | 6 | Parallel | `arch-api-event`, `arch-tech-stack` (inline) |
| `arch-agent-data` | 6 | Parallel | `arch-data`, `arch-tech-stack` (inline) |
| `arch-agent-deployment` | 6 | Parallel | `arch-deployment`, `arch-tech-stack` (inline) |
| `arch-agent-security` | 6 | Parallel | `arch-security`, `arch-tech-stack` (inline) |
| `arch-agent-ops` | 6 | Parallel | `arch-production-operations`, `arch-tech-stack` (inline) |
| `arch-synthesizer` | 7 | — | — |
| `arch-reviewer` | 8 | — | `arch-review` |

**Total: 11 agents + 1 command + 1 skill (`arch-template-formatter`)**

---

## Approval Gates

| Gate | Phase | What user approves |
|------|-------|--------------------|
| Approach selection | 4 | Architectural direction chosen from 2–3 options |
| Decomposition approval | 5 | Service boundaries, ASCII diagram, domain events |
| Review resolution | 8 | 🔴 Critical findings resolved before finalization |

---

## File Structure (post-implementation)

```
architecture-design/.claude-plugin/
├── agents/
│   ├── arch-orchestrator.md
│   ├── arch-explorer.md
│   ├── arch-strategist.md
│   ├── arch-agent-ddd.md
│   ├── arch-agent-api-event.md
│   ├── arch-agent-data.md
│   ├── arch-agent-deployment.md
│   ├── arch-agent-security.md
│   ├── arch-agent-ops.md
│   ├── arch-synthesizer.md
│   └── arch-reviewer.md
├── command/
│   └── create-architecture.md
├── skills/
│   ├── arch-api-event/
│   ├── arch-data/
│   ├── arch-deployment/
│   ├── arch-domain-driven-design/
│   ├── arch-production-operations/
│   ├── arch-review/
│   ├── arch-security/
│   ├── arch-tech-stack/
│   └── arch-template-formatter/   ← new skill
│       ├── SKILL.md
│       └── references/
│           └── solution-intent-template.md   ← default template
└── plugin.json   ← missing, must be created
```

---

## Decisions Log

| Question | Decision |
|----------|----------|
| `arch-explorer` data access | Uses Atlassian MCP (Confluence, Jira) and GitHub MCP for authenticated reads |
| Clarifying question count | Adaptive — scales with initiative complexity and ambiguity, no fixed cap |
| 🟡 Important review findings | Appended to Section 12 and left to the user; no second approval gate |
