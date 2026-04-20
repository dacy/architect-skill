# Session Resumability Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add progressive document writing and session resume capability to the `/create-architect` workflow so users can pick up from the last completed phase in a new session.

**Architecture:** The Solution Intent document is created at Phase 1 end and written to after each phase. It acts as both the persistent state store and the final deliverable. The orchestrator's entry point auto-scans `docs/` for in-progress sessions before starting, and supports `--resume <path>` for explicit resumption.

**Tech Stack:** Markdown instruction files only — no code. All changes are to `.claude-plugin/` agent and command markdown files.

**Prerequisites:** Apply `docs/superpowers/plans/2026-04-19-create-architect-command.md` before this plan. That plan reorders Phase 4 (Domain Design) and Phase 5 (Chosen Approach) in the orchestrator, which the section-writing steps in Tasks 4–5 below depend on.

---

### Task 1: Update Command Entry Point

**Files:**
- Modify: `architecture-design/.claude-plugin/commands/create-architecture.md`

- [ ] **Step 1: Verify current file state**

Run:
```bash
grep -n "argument-hint\|resume\|template_path" "architecture-design/.claude-plugin/commands/create-architecture.md"
```
Expected: lines with `argument-hint` and `--template` but no mention of `--resume` or `resume_path`.

- [ ] **Step 2: Replace file content**

Write `architecture-design/.claude-plugin/commands/create-architecture.md` with:

```markdown
---
description: Design a system architecture through a guided 10-phase workflow, producing a Solution Intent document. Supports resuming in-progress sessions.
argument-hint: "[initiative description] [--template <path>] [--resume <path>]"
---

# create-architect Command

You are the entry point for the `/create-architect` command. Your only job is to parse the user's arguments and hand off to the `arch-orchestrator` agent.

## Parsing Arguments

Raw arguments: `$ARGUMENTS`

1. If `--resume <value>` appears anywhere in the arguments, extract `<value>` as `resume_path` and remove `--resume <value>` from the remaining text.
2. If `--template <value>` appears anywhere in the remaining arguments, extract `<value>` as `template_path` and remove `--template <value>` from the remaining text.
3. Everything remaining is `initiative_input`. It may be empty, a plain description, a structured brief, or a URL.

## Handoff

Invoke the `arch-orchestrator` agent immediately with:
- `initiative_input`: the parsed initiative text (empty string if nothing was provided)
- `template_path`: the template path (null if `--template` was not present)
- `resume_path`: the path to an in-progress Solution Intent document (null if `--resume` was not present)

Do not do any analysis or design work yourself. The orchestrator manages all phases.
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -c "resume_path" "architecture-design/.claude-plugin/commands/create-architecture.md"
```
Expected: `3` (in `--resume <value>` description, in `resume_path` variable name, and in the Handoff list).

- [ ] **Step 4: Commit**

```bash
git add architecture-design/.claude-plugin/commands/create-architecture.md
git commit -m "feat: add --resume flag to create-architect command"
```

---

### Task 2: Add Inputs, Entry Point, and Resume Flow to Orchestrator

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-orchestrator.md:14-19`

- [ ] **Step 1: Verify current inputs block**

Run:
```bash
grep -n "Inputs\|initiative_input\|template_path\|resume_path" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
```
Expected: lines with `initiative_input` and `template_path` but no `resume_path`.

- [ ] **Step 2: Replace the Inputs block and insert Entry Point + Resume Flow**

Find this exact block (around lines 14–19):
```
## Inputs
- `initiative_input`: raw text, structured brief, URL, or empty string
- `template_path`: path to output template or null

---
```

Replace with:
```
## Inputs
- `initiative_input`: raw text, structured brief, URL, or empty string
- `template_path`: path to output template or null
- `resume_path`: path to an in-progress `*-solution-intent.md` file, or null

---

## Entry Point

**Before starting Phase 1**, check for a resume scenario:

1. If `resume_path` is not null, proceed to **Resume Flow** below.
2. If `resume_path` is null, scan `docs/` for any file matching `*-solution-intent.md` (Read tool):
   - Read each file's frontmatter.
   - Collect files where `status: in-progress`.
   - If exactly one found: ask the user:
     > "I found an in-progress session: `<filename>`. Resume it, or start a new one?"
     > If they choose resume: set `resume_path` to that path and proceed to Resume Flow.
   - If multiple found: list them and ask which to resume (or start fresh). If the user picks one, set `resume_path` and proceed to Resume Flow.
   - If none found: proceed to Phase 1 normally.

---

## Resume Flow

When `resume_path` is set:

1. Read the document at `resume_path` in full (Read tool).
2. Extract `phase_completed` from frontmatter (integer).
3. Reconstruct all context variables from the document sections that exist:
   - `## Initiative Brief` → `initiative_name`, `goals`, `constraints`, `referenced_systems`
   - `## Context` → `exploration_context`; if a `### Codebase Context` subsection exists, also extract `codebase_context`
   - `## Clarifications` → `clarification_context`
   - `## Domain Design` → `ddd_output`
   - `## Chosen Approach` → `chosen_approach`, `approach_rationale`, `tech_flags`
   - `## Design Sections` → `api_event_output`, `data_output`, `deployment_output`, `security_output`, `ops_output`
   - `## Solution Intent Draft` → `solution_intent_draft`
   - `## Review Findings` → `review_findings`
4. Set `doc_path = resume_path`.
5. Create a TodoWrite task list for the remaining phases only (Phase `phase_completed + 1` through Phase 10).
6. Present the resume summary:
   > **Resuming:** <initiative_name>
   > **Completed:** Phases 1–<phase_completed>
   > **Last output:** <3–5 lines from the last-written section in the document>
   >
   > Ready to continue from Phase <phase_completed + 1>?
7. Wait for user confirmation, then jump to Phase `phase_completed + 1`.

---
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "Entry Point\|Resume Flow\|resume_path\|phase_completed + 1" "architecture-design/.claude-plugin/agents/arch-orchestrator.md" | head -20
```
Expected: lines showing `Entry Point`, `Resume Flow`, `resume_path`, and `phase_completed + 1`.

- [ ] **Step 4: Commit**

```bash
git add architecture-design/.claude-plugin/agents/arch-orchestrator.md
git commit -m "feat: add entry point auto-scan and resume flow to arch-orchestrator"
```

---

### Task 3: Add Document Creation to Phase 1 and Simplify Phase 10

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-orchestrator.md`

- [ ] **Step 1: Verify Phase 1 end**

Run:
```bash
grep -n "referenced_systems\|doc_path\|kebab" "architecture-design/.claude-plugin/agents/arch-orchestrator.md" | head -10
```
Expected: `referenced_systems` line present but no `doc_path` or `kebab` lines yet.

- [ ] **Step 2: Add document creation step to Phase 1**

Find this exact block (end of Phase 1):
```
Extract and record:
- `initiative_name`: 3–5 word title derived from input (used for kebab-case filename)
- `goals`: list of stated goals (empty if none)
- `constraints`: list of stated constraints (empty if none)
- `referenced_systems`: list of any URLs or named external systems mentioned

---
```

Replace with:
```
Extract and record:
- `initiative_name`: 3–5 word title derived from input (used for kebab-case filename)
- `goals`: list of stated goals (empty if none)
- `constraints`: list of stated constraints (empty if none)
- `referenced_systems`: list of any URLs or named external systems mentioned

**After extracting all Phase 1 fields:**

1. Derive `doc_path`:
   - Convert `initiative_name` to kebab-case: lowercase, replace spaces with hyphens, strip non-alphanumeric/hyphen characters.
   - Example: "ShopFlow B2C E-Commerce" → `docs/shopflow-b2c-e-commerce-solution-intent.md`
   - Set `doc_path = docs/<kebab-name>-solution-intent.md`

2. Write `doc_path` (Write tool) with this exact structure:
   ```
   ---
   status: in-progress
   phase_completed: 1
   ---

   ## Initiative Brief

   **Initiative:** <initiative_name>

   **Goals:**
   <goals as a bullet list; write "None stated" if empty>

   **Constraints:**
   <constraints as a bullet list; write "None stated" if empty>

   **Referenced Systems:**
   <referenced_systems as a bullet list; write "None" if empty>
   ```

---
```

- [ ] **Step 3: Verify Phase 1 addition**

Run:
```bash
grep -n "doc_path\|kebab-case\|phase_completed: 1\|Initiative Brief" "architecture-design/.claude-plugin/agents/arch-orchestrator.md" | head -10
```
Expected: all four terms appear in Phase 1.

- [ ] **Step 4: Replace Phase 10**

Find this exact block (current Phase 10):
```
## Phase 10 — Finalize

Derive filename: convert `initiative_name` to kebab-case and append `-solution-intent.md`.
Example: "Customer Notification Service" → `customer-notification-service-solution-intent.md`

Write the final document to: `docs/<filename>`

Confirm to user:
> "Architecture design complete. Solution Intent saved to `docs/<filename>`."
```

Replace with:
```
## Phase 10 — Finalize

The Solution Intent document has been written progressively throughout the workflow and already exists at `doc_path`.

1. Read `doc_path` (Read tool).
2. Replace `status: in-progress` with `status: complete` in the frontmatter.
3. Replace `phase_completed: 9` with `phase_completed: 10` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).
5. Confirm to the user:
   > "Architecture design complete. Solution Intent saved to `<doc_path>`."
```

- [ ] **Step 5: Verify Phase 10**

Run:
```bash
grep -n "status: complete\|phase_completed: 10\|doc_path" "architecture-design/.claude-plugin/agents/arch-orchestrator.md" | tail -10
```
Expected: Phase 10 contains `status: complete`, `phase_completed: 10`, and `doc_path`.

- [ ] **Step 6: Commit**

```bash
git add architecture-design/.claude-plugin/agents/arch-orchestrator.md
git commit -m "feat: add Phase 1 document creation and simplify Phase 10 finalize in arch-orchestrator"
```

---

### Task 4: Add Section-Writing Steps to Phases 2–5

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-orchestrator.md`

Note: Phase 4 = Domain Design (DDD) and Phase 5 = Chosen Approach (Strategist) — this matches the phase ordering applied by the `create-architect-command` prerequisite plan.

- [ ] **Step 1: Add section-writing to Phase 2**

Find this exact block (end of Phase 2):
```
Collect all returned context blocks into `exploration_context`.

If `referenced_systems` is empty, set `exploration_context` to `[]` and proceed.

---
```

Replace with:
```
Collect all returned context blocks into `exploration_context`.

If `referenced_systems` is empty, set `exploration_context` to `[]` and proceed.

**After completing Phase 2 (and Phase 2b if it ran):**

1. Read `doc_path` (Read tool).
2. Append to the document:
   ```
   ## Context

   <exploration_context blocks, one subsection per explored system>
   <if Phase 2b ran: append a ### Codebase Context subsection with codebase_context>
   ```
3. Replace `phase_completed: 1` with `phase_completed: 2` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).

---
```

- [ ] **Step 2: Add section-writing to Phase 3**

Find this exact block (end of Phase 3):
```
Stop when you can proceed without significant design assumptions.

Record all answers in `clarification_context`.

---
```

Replace with:
```
Stop when you can proceed without significant design assumptions.

Record all answers in `clarification_context`.

**After completing Phase 3:**

1. Read `doc_path` (Read tool).
2. Append to the document:
   ```
   ## Clarifications

   <all clarifying questions asked and answers received, as Q&A pairs>
   ```
3. Replace `phase_completed: 2` with `phase_completed: 3` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).

---
```

- [ ] **Step 3: Add section-writing to Phase 4 (Domain Design)**

Find this exact block (end of Phase 4 after the create-architect-command plan is applied — DDD phase):
```
Store approved output as `ddd_output`.

---
```

Replace with:
```
Store approved output as `ddd_output`.

**After completing Phase 4:**

1. Read `doc_path` (Read tool).
2. Append to the document:
   ```
   ## Domain Design

   <ddd_output — bounded contexts, service descriptions, ASCII diagram, and domain events>
   ```
3. Replace `phase_completed: 3` with `phase_completed: 4` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).

---
```

- [ ] **Step 4: Add section-writing to Phase 5 (Chosen Approach)**

Find this exact block (end of Phase 5 after the create-architect-command plan is applied — Strategist phase):
```
Record: `chosen_approach`, `approach_rationale`, `tech_flags`.

---
```

Replace with:
```
Record: `chosen_approach`, `approach_rationale`, `tech_flags`.

**After completing Phase 5:**

1. Read `doc_path` (Read tool).
2. Append to the document:
   ```
   ## Chosen Approach

   <chosen_approach, approach_rationale, and tech_flags from the approved strategist output>
   ```
3. Replace `phase_completed: 4` with `phase_completed: 5` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).

---
```

- [ ] **Step 5: Verify Phases 2–5**

Run:
```bash
grep -n "After completing Phase" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
```
Expected: at minimum 4 lines — one each for Phases 2, 3, 4, and 5.

- [ ] **Step 6: Commit**

```bash
git add architecture-design/.claude-plugin/agents/arch-orchestrator.md
git commit -m "feat: add progressive section writing to Phases 2-5 in arch-orchestrator"
```

---

### Task 5: Add Section-Writing Steps to Phases 6–9

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-orchestrator.md`

- [ ] **Step 1: Add section-writing to Phase 6 (Parallel Deep Design)**

Find this exact block (end of Phase 6):
```
Collect: `api_event_output`, `data_output`, `deployment_output`, `security_output`, `ops_output`.

---
```

Replace with:
```
Collect: `api_event_output`, `data_output`, `deployment_output`, `security_output`, `ops_output`.

**After completing Phase 6:**

1. Read `doc_path` (Read tool).
2. Append to the document:
   ```
   ## Design Sections

   ### API and Event Design
   <api_event_output>

   ### Data Design
   <data_output>

   ### Deployment Design
   <deployment_output>

   ### Security Design
   <security_output>

   ### Operations Design
   <ops_output>
   ```
3. Replace `phase_completed: 5` with `phase_completed: 6` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).

---
```

- [ ] **Step 2: Add section-writing to Phase 7 (Synthesis)**

Find this exact block (entire Phase 7):
```
## Phase 7 — Synthesis

Invoke `arch-synthesizer` with all collected outputs. Receive `solution_intent_draft`.

---
```

Replace with:
```
## Phase 7 — Synthesis

Invoke `arch-synthesizer` with all collected outputs. Receive `solution_intent_draft`.

**After completing Phase 7:**

1. Read `doc_path` (Read tool).
2. Append to the document:
   ```
   ## Solution Intent Draft

   <solution_intent_draft>
   ```
3. Replace `phase_completed: 6` with `phase_completed: 7` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).

---
```

- [ ] **Step 3: Add section-writing to Phase 8 (Review)**

Find this exact block (end of Phase 8):
```
Append all remaining findings (🟡 Important, 🟢 Minor) to Section 12 of the document.

---
```

Replace with:
```
Append all remaining findings (🟡 Important, 🟢 Minor) to Section 12 of the document.

**After completing Phase 8:**

1. Read `doc_path` (Read tool).
2. Append to the document:
   ```
   ## Review Findings

   <all review_findings — 🔴 Critical items with their resolutions, 🟡 Important, and 🟢 Minor>
   ```
3. Replace `phase_completed: 7` with `phase_completed: 8` in the frontmatter.
4. Write the updated content back to `doc_path` (Write tool).

---
```

- [ ] **Step 4: Add section-writing to Phase 9 (Template Formatting)**

Find this exact block (end of Phase 9):
```
If a template was found, invoke the `arch-template-formatter` skill with the document and template. Use the formatted output as the final document.

---
```

Replace with:
```
If a template was found, invoke the `arch-template-formatter` skill with the document and template. Use the formatted output as the final document.

**After completing Phase 9:**

- If a template was applied: write the formatted document to `doc_path`, prepending the following frontmatter block before the formatted content:
  ```
  ---
  status: in-progress
  phase_completed: 9
  ---
  ```
- If no template was found (Phase 9 skipped): read `doc_path`, replace `phase_completed: 8` with `phase_completed: 9` in the frontmatter, write back.

---
```

- [ ] **Step 5: Verify all section-writing blocks**

Run:
```bash
grep -n "After completing Phase" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
```
Expected: exactly 8 lines — one each for Phases 2 through 9.

- [ ] **Step 6: Commit**

```bash
git add architecture-design/.claude-plugin/agents/arch-orchestrator.md
git commit -m "feat: add progressive section writing to Phases 6-9 in arch-orchestrator"
```

---

### Task 6: Final Validation

**Files:** Read only.

- [ ] **Step 1: Check resume_path wired through both files**

Run:
```bash
grep -c "resume_path" "architecture-design/.claude-plugin/commands/create-architecture.md"
grep -c "resume_path" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
```
Expected: first command returns `3`; second returns at least `5`.

- [ ] **Step 2: Confirm Entry Point and Resume Flow are present**

Run:
```bash
grep -n "## Entry Point\|## Resume Flow" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
```
Expected: one line each for `## Entry Point` and `## Resume Flow`.

- [ ] **Step 3: Confirm all phases have a section-writing step**

Run:
```bash
grep -c "After completing Phase" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
```
Expected: `8`.

- [ ] **Step 4: Confirm Phase 1 creates doc_path and Phase 10 closes it**

Run:
```bash
grep -n "doc_path\|status: complete\|phase_completed: 1\b" "architecture-design/.claude-plugin/agents/arch-orchestrator.md" | grep -v "resume_path"
```
Expected: `doc_path =` assignment in Phase 1; `status: complete` and `phase_completed: 10` in Phase 10.

- [ ] **Step 5: Smoke-read the orchestrator top section**

Run:
```bash
head -80 "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
```
Expected: frontmatter, then `## Inputs` (with `resume_path`), then `---`, then `## Entry Point`, then `---`, then `## Resume Flow`, then `---`, then `## Phase 1`.
