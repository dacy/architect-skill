# Brownfield Code Exploration Design

## Goal

Add a codebase exploration phase to the `/create-architect` workflow so that brownfield architecture design is informed by the actual shape of existing code — not just external documentation and clarifying questions.

## Background

The current workflow gathers context from external systems (Confluence, Jira, GitHub) via `arch-explorer` agents. When the initiative touches an existing local codebase, that context source is missing. Architects designing against unfamiliar code make unnecessary assumptions, propose approaches that don't fit existing constraints, and generate DDD decompositions that ignore established module boundaries.

---

## What We're Building

### New agent: `arch-code-explorer`

A new agent adapted from Anthropic's `code-explorer` for architecture-level analysis. Operates in two modes:

**Overview mode** (always runs first):
- Tech stack: languages, frameworks, key libraries and their versions
- Project structure: top-level directories and their architectural purpose
- Service/module boundaries and how they communicate (sync vs async, protocols)
- Key architectural patterns in use (layered, hexagonal, event-driven, CQRS, etc.)
- External dependencies: databases, queues, caches, third-party APIs
- Areas that appear architecturally significant, complex, or in flux

Returns a structured snapshot block plus `suggested_deep_dives`: 3–5 focus areas identified as worth investigating given the stated initiative goals.

**Deep-dive mode** (per focus area, run in parallel):
- Execution path tracing through the named area
- Component responsibilities and dependencies
- Data flows and transformation points
- Patterns and abstractions in use
- Architectural strengths and improvement opportunities specific to this area

Returns a focused finding block for the area.

**File:** `architecture-design/.claude-plugin/agents/arch-code-explorer.md`

---

### New Phase 2b: Codebase Exploration

Inserted between Phase 2 (external system exploration) and Phase 3 (clarifying questions).

**Flow:**

1. **Offer** — orchestrator always asks:
   > "Is there an existing codebase for this initiative I should explore? If so, provide the path (or say 'no' to skip)."

2. **Skip path** — if the user declines, set `codebase_context` to `null` and proceed to Phase 3. No other changes.

3. **Overview** — spawn `arch-code-explorer` in overview mode with:
   - `codebase_path`: provided path
   - `initiative_name`, `goals` (from Phase 1)
   
   Receive: architectural snapshot + `suggested_deep_dives`.

4. **Present and adjust** — show the user the snapshot, then:
   > "Based on your goals, I'd suggest exploring these areas in more depth: [list]. Would you like to add, remove, or adjust any before I proceed?"
   
   Wait for the user's response.

5. **Targeted dives** — spawn one `arch-code-explorer` instance per confirmed area simultaneously in deep-dive mode. Collect all finding blocks.

6. **Merge** — combine overview snapshot + deep-dive findings into `codebase_context`. Proceed to Phase 3.

`codebase_context` is stored separately from `exploration_context` (external systems) so downstream agents can distinguish constraints derived from existing code vs. external documentation.

**`codebase_context` structure:**
```
## Codebase Context: [codebase_path]

### Overview
**Tech stack:** [languages, frameworks, key libraries]
**Architecture pattern:** [e.g. layered MVC, hexagonal, event-driven]
**Module/service boundaries:** [list with brief purpose per boundary]
**External dependencies:** [databases, queues, caches, third-party APIs]
**Notable observations:** [areas of complexity, technical debt, or architectural significance]

### Deep-Dive Findings
#### [Focus Area 1]
[finding block]

#### [Focus Area 2]
[finding block]
```

---

### Downstream phase changes

All phases from Phase 3 onward receive `codebase_context` as part of the full context object. Changes per phase:

**Phase 3 — Clarifying Questions**
- Skip questions already answered by codebase exploration (e.g. tech stack, existing patterns).
- Surface contradictions: if the codebase reveals something that conflicts with a stated goal or constraint, ask about it explicitly. Example: "The codebase uses PostgreSQL but you listed a constraint to adopt MongoDB — is that intentional?"

**Phase 4 — Domain Design (DDD)**
- `arch-agent-ddd` uses existing module/service boundaries as candidate starting points for bounded contexts rather than deriving them purely from goals and constraints.
- The ASCII diagram should reflect what already exists and what is being added or changed. Convention: existing services use `[Service Name]`, new services use `[NEW: Service Name]`, services being modified use `[MOD: Service Name]`.

**Phase 5 — High-Level Approaches (Strategist)**
- `arch-strategist` uses `codebase_context` to assess rewrite risk for each pattern. An approach that requires restructuring a large, tightly coupled codebase should be rated higher complexity and flagged accordingly.
- Strangler fig should be surfaced as a candidate whenever an existing codebase is present.

**Phase 6 — Parallel Deep Design (all five specialists)**
- All agents receive `codebase_context`.
- Data agent: respect existing schemas; highlight migration steps if changes are required.
- Deployment agent: build on existing infrastructure patterns; note deviations and justify them.
- Security agent: identify gaps between what is already in place and what the new design requires.
- API/Event agent: align new contracts with existing API styles where appropriate.
- Ops agent: build on existing observability tooling if present.

---

## Files to Create

| File | Action |
|------|--------|
| `architecture-design/.claude-plugin/agents/arch-code-explorer.md` | Create |

## Files to Modify

| File | Change |
|------|--------|
| `architecture-design/.claude-plugin/agents/arch-orchestrator.md` | Add Phase 2b; add `codebase_context` to context object passed to all downstream phases |
| `architecture-design/.claude-plugin/agents/arch-agent-ddd.md` | Accept and use `codebase_context`; convention for new vs. existing services in ASCII diagram |
| `architecture-design/.claude-plugin/agents/arch-strategist.md` | Accept and use `codebase_context`; assess rewrite risk; surface strangler fig when codebase present |
| `architecture-design/.claude-plugin/agents/arch-agent-api-event.md` | Accept `codebase_context`; align new contracts with existing API styles |
| `architecture-design/.claude-plugin/agents/arch-agent-data.md` | Accept `codebase_context`; respect existing schemas; include migration steps |
| `architecture-design/.claude-plugin/agents/arch-agent-deployment.md` | Accept `codebase_context`; build on existing infra patterns |
| `architecture-design/.claude-plugin/agents/arch-agent-security.md` | Accept `codebase_context`; identify security gaps vs. existing posture |
| `architecture-design/.claude-plugin/agents/arch-agent-ops.md` | Accept `codebase_context`; build on existing observability tooling |

---

## Out of Scope

- Exploring remote codebases (GitHub, GitLab) — those are handled by `arch-explorer` via URL
- Automatic brownfield detection — the offer is always made; the user decides
- Code quality analysis or refactoring recommendations — the focus is architectural understanding only
- Exploring multiple codebases in a single initiative — single path only in this iteration
