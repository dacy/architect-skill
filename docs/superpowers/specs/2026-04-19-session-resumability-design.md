# Session Resumability Design

## Goal

Allow users to resume an in-progress `/create-architect` workflow across sessions — picking up from the last completed phase without losing context.

## Background

The `/create-architect` workflow spans 10 phases and can take significant time. If a session ends before the workflow completes, the user has no way to continue — they must restart from scratch. The workflow already produces a Solution Intent document as its final output. This design leverages that document as both the state store and the recovery artifact.

---

## What We're Building

### State tracking: minimal YAML frontmatter

Every in-progress Solution Intent document carries a two-field frontmatter block at the top:

```yaml
---
status: in-progress
phase_completed: 3
---
```

- `status`: `in-progress` while the workflow is running; set to `complete` after Phase 10
- `phase_completed`: the index of the last fully completed phase (0 = document created, not yet completed Phase 1)

No other fields are stored in frontmatter. The document body sections already written contain the full context needed for reconstruction.

---

### Document lifecycle

The orchestrator creates the Solution Intent document at the end of Phase 1 and appends a section after each phase completes. The frontmatter `phase_completed` value is updated each time a section is written.

**Document path:** `docs/<initiative-name>-solution-intent.md`
- `initiative-name` is derived from the initiative name collected in Phase 1 (kebab-case, lowercased)
- Example: `docs/shopflow-b2c-ecommerce-solution-intent.md`

**Phase-by-phase writing:**

| Phase | Section written to document |
|-------|-----------------------------|
| 1 | `## Initiative Brief` (goals, constraints, referenced systems) |
| 2 | `## Context` (external system exploration findings) |
| 2b | Append codebase findings to `## Context` (if codebase explored) |
| 3 | `## Clarifications` (questions asked and answers received) |
| 4 | `## Domain Design` (bounded contexts, DDD output, ASCII diagram) |
| 5 | `## Chosen Approach` (strategist output: selected pattern + rationale) |
| 6 | `## Design Sections` (all five specialist outputs) |
| 7 | `## Solution Intent Draft` (synthesized narrative) |
| 8 | `## Review Findings` (reviewer feedback) |
| 9 | All sections reformatted in-place (template formatter output) |
| 10 | `status: complete` set in frontmatter |

---

### Resume trigger

The orchestrator supports two ways to resume:

**Explicit path:**
```
/create-architect --resume docs/shopflow-b2c-ecommerce-solution-intent.md
```

**Auto-scan (no path provided):**
When `/create-architect` is invoked with no arguments (or no `--resume` flag), the orchestrator checks for any `*-solution-intent.md` file in `docs/` with `status: in-progress`. If exactly one is found, it offers to resume. If multiple are found, it lists them and asks which to resume (or start fresh).

---

### Resume UX

When resuming, the orchestrator:

1. Reads the document front-to-back to reconstruct all context variables (initiative brief, exploration context, codebase context, clarifications, ddd_output, etc.)
2. Presents a brief summary of completed phases:

   > **Resuming:** ShopFlow B2C E-Commerce  
   > **Completed:** Phases 1–3 (Initiative Brief, Context, Clarifications)  
   > **Last output:** [last section written, quoted briefly — 3–5 lines]  
   >
   > Ready to continue from Phase 4 (Domain Design)?

3. Waits for the user to confirm or redirect before proceeding.

The summary is intentionally brief — the document already contains the full detail. The user can review it directly if needed.

---

## Files to Modify

| File | Change |
|------|--------|
| `architecture-design/.claude-plugin/agents/arch-orchestrator.md` | Add document creation logic to Phase 1; add section-writing step to each phase; add `--resume` handling at entry point; add auto-scan logic |
| `architecture-design/.claude-plugin/commands/create-architecture.md` | Document `--resume <path>` as a supported flag in the command description |

---

## Out of Scope

- Multiple concurrent in-progress sessions for the same initiative — not handled in this iteration; last-write wins on the document
- Resuming from a partially completed phase — resume always starts at the beginning of the next incomplete phase
- Automatic periodic saves mid-phase — section is written once, after the phase fully completes
- Cloud or remote storage — local file only
- Conflict resolution if the document is edited externally between sessions
