---
name: tech-stack-skill
description: Use this skill for technology selection decisions. Trigger on tech stack, technology selection, what language, what framework, approved technologies, tech exception, or any time a technology choice is being made. The company has an opinionated tech stack — always consult this skill before recommending a technology to ensure alignment with approved choices.
---

# Tech Stack Skill

## Overview

This skill guides technology selection aligned to the company's opinionated tech stack.
The stack defines approved technologies by layer — anything outside the approved list
requires a formal exception before it can be used.

Read `references/tech-stack.md` for the full approved technology catalog, tier definitions,
and exception process before making any technology recommendation.

---

## Output

- **Markdown (.md)** — Tech Stack section for Solution Intent, or standalone analysis
- Name file: `[initiative-name]-tech-stack.md`

---

## Approval Tiers

Technologies fall into one of three tiers (see `references/tech-stack.md` for the full list):

| Tier | Meaning |
|------|---------|
| **Approved** | Standard choice — use without additional review |
| **Conditionally Approved** | Approved for specific use cases only — confirm fit |
| **Exception Required** | Not on the approved list — requires architecture board review |

---

## Process

### Step 1: Identify technology decisions needed

From the requirement, list every technology choice that must be made:
- Runtime language and version
- Application framework
- API style (REST, gRPC, GraphQL)
- Messaging / eventing platform
- Database(s)
- Caching
- UI framework (if applicable)
- Infrastructure / compute
- CI/CD tooling
- Observability stack

### Step 2: Match to approved stack

For each decision, look up the approved choice in `references/tech-stack.md` and document:

| Layer | Decision | Approved Choice | Tier | Notes |
|-------|----------|----------------|------|-------|
| Runtime | Language | [approved] | Approved | |
| Framework | API | [approved] | Approved | |
| Messaging | Events | [approved] | Approved | |
| ... | ... | ... | ... | |

### Step 3: Flag exceptions

For any technology not in the approved catalog:
1. Document the proposed technology and the reason the approved alternative doesn't fit
2. Identify the exception type (new technology, unsupported version, non-standard pattern)
3. Outline the exception approval path (see `references/tech-stack.md` — Exception Process)
4. Mark as ⚠️ in the tech stack table — no exceptions proceed without approval

### Step 4: Version alignment

Ensure proposed versions align with company-supported versions. Flag any end-of-life or
unsupported versions as risks.

---

## Document Structure

```
# [Initiative Name] — Tech Stack

## Technology Decisions

| Layer | Choice | Tier | Justification |
|-------|--------|------|---------------|
| [Layer] | [Technology v.x] | Approved / Conditional / Exception | [Why] |

## Exception Requests
[Technology | Reason approved option doesn't fit | Approval path]

## Version Risk Register
[Any EOL or unsupported version flags]

## Open Questions
```

---

## Reference Files

- `references/tech-stack.md` — Company approved technology catalog by layer, tier
  classifications, conditionally approved use cases, and exception process.
  **TODO:** Populate with your organization's opinionated tech stack.
