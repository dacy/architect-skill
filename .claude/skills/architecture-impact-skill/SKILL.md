---
name: architecture-impact-skill
description: Use this skill whenever a proposed change must be assessed against existing applications. Trigger on "impact analysis", "what apps are affected", "impact assessment", "how does this affect our systems", "application impacts", "current vs future state", or any time a Solution Intent touches existing systems. Use proactively — if a solution integrates with or changes existing applications, always run an impact assessment first.
---

# Architecture Impact Skill

## Overview

This skill produces a structured impact assessment documenting how a proposed change affects
the current and future states of each impacted application. It is typically the first
section produced within a Solution Intent when existing systems are involved.

Read `references/application-registry.md` for the company's application catalog format.

---

## Output

- **Markdown (.md)** — Confluence-ready
- Name file: `[initiative-name]-impact-assessment.md`

---

## Process

### Step 1: Identify impacted applications

Extract from the requirement or Solution Intent brief:
- Applications being directly changed (owners of the change)
- Applications being integrated with (consumers or upstream providers)
- Downstream applications receiving changed data or events
- Upstream applications this solution depends on

Cross-reference against the Application Registry (`references/application-registry.md`).

### Step 2: Document current state per application

For each impacted application:
- Current role and responsibilities in the ecosystem
- Current integration points (APIs, events, data feeds, UI)
- Current technology stack (high-level)
- Application owner / team

### Step 3: Document future state per application

For each impacted application:
- What changes: new integrations, modified interfaces, behavior changes
- What is added: new endpoints, event subscriptions, data flows, UI surfaces
- What is retired: deprecated endpoints, removed consumers, decommissioned feeds
- Estimated effort: Low / Medium / High

### Step 4: Classify impact

| Application | Impact Type | Effort | Risk | Owner |
|-------------|-------------|--------|------|-------|
| [App Name]  | New integration / Modified / Downstream / Retired | L/M/H | L/M/H | [Team] |

**Impact types:**
- **New integration** — application must implement a new interface
- **Modified** — existing interface or behavior changes
- **Downstream** — receives changed data or events with no direct code change required
- **Retired** — existing integration being decommissioned

### Step 5: Sequencing

Identify which applications must change before others can proceed. Document the
dependency chain and any hard sequencing constraints.

---

## Document Structure

```
# [Initiative Name] — Architecture Impact Assessment
**Date:** [date] | **Author:** [name] | **Classification:** Internal

---

## Summary
[2–3 sentences: how many apps affected, overall impact level, key risk areas]

## Impacted Applications

### [Application Name]
**App ID:** [ID from registry] | **Owner:** [Team] | **Criticality:** [tier]

**Current State:**
[Role and current integrations]

**Future State:**
[What changes, what's added, what's retired]

**Impact:** [New integration / Modified / Downstream / Retired]
**Effort:** [L/M/H] | **Risk:** [L/M/H]

---
[Repeat for each application]

## Impact Summary Table
| Application | App ID | Impact Type | Effort | Risk | Owner |
|-------------|--------|-------------|--------|------|-------|

## Delivery Sequencing
[Dependency chain or phasing notes]

## Open Questions
[Unresolved questions about specific application impacts]
```

---

## Reference Files

- `references/application-registry.md` — Application catalog format and known applications.
  **TODO:** Populate with your organization's application registry.
