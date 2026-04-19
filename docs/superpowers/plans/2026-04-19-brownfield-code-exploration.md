# Brownfield Code Exploration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Phase 2b codebase exploration step to the `/create-architect` workflow, backed by a new `arch-code-explorer` agent, so brownfield architecture designs are informed by the actual shape of existing code.

**Architecture:** A new `arch-code-explorer` agent operates in two modes (overview then targeted deep-dives). The `arch-orchestrator` always offers codebase exploration after Phase 2; if accepted, it spawns the agent for an overview, presents suggested deep-dive areas, waits for user adjustment, then spawns parallel deep-dive instances. The resulting `codebase_context` is threaded through all downstream phases (3–6).

**Tech Stack:** Claude Code plugin system (markdown instruction files), existing `arch-*` agent pattern.

---

## File Map

**Create:**
- `architecture-design/.claude-plugin/agents/arch-code-explorer.md`

**Modify:**
- `architecture-design/.claude-plugin/agents/arch-orchestrator.md` — add Phase 2b; update Phase 3/4/5/6 to receive `codebase_context`
- `architecture-design/.claude-plugin/agents/arch-agent-ddd.md` — accept `codebase_context`; use existing boundaries; add diagram conventions
- `architecture-design/.claude-plugin/agents/arch-strategist.md` — accept `codebase_context`; assess rewrite risk; surface strangler fig
- `architecture-design/.claude-plugin/agents/arch-agent-api-event.md` — accept `codebase_context`; align with existing API styles
- `architecture-design/.claude-plugin/agents/arch-agent-data.md` — accept `codebase_context`; detect migration from existing schemas
- `architecture-design/.claude-plugin/agents/arch-agent-deployment.md` — accept `codebase_context`; build on existing infra patterns
- `architecture-design/.claude-plugin/agents/arch-agent-security.md` — accept `codebase_context`; identify security gaps vs. existing posture
- `architecture-design/.claude-plugin/agents/arch-agent-ops.md` — accept `codebase_context`; build on existing observability tooling

---

## Task 1: Create arch-code-explorer agent

**Files:**
- Create: `architecture-design/.claude-plugin/agents/arch-code-explorer.md`

- [ ] **Step 1: Write the agent file**

  Create `architecture-design/.claude-plugin/agents/arch-code-explorer.md` with this exact content:

  ````markdown
  ---
  name: arch-code-explorer
  description: Explores a local codebase for architecture-level understanding. Two modes — overview (architectural snapshot + suggested deep-dives) and deep-dive (focused analysis of one area). Spawned by arch-orchestrator in Phase 2b — one instance for overview, then one per confirmed deep-dive area in parallel.
  tools: Glob, Grep, Read, Bash, TodoWrite
  model: sonnet
  color: yellow
  ---

  # arch-code-explorer

  You explore a local codebase and return structured architectural context. You operate in one of two modes determined by your input.

  ## Input
  - `codebase_path`: filesystem path to the root of the codebase
  - `mode`: `"overview"` or `"deep-dive"`
  - `initiative_name`, `goals`: from Phase 1 (used to focus what you look for)
  - `focus_area` (deep-dive mode only): the named area to analyse in depth

  ---

  ## Overview Mode

  ### Step 1: Orient yourself

  Run:
  ```bash
  ls [codebase_path]
  ```
  List top-level directories and files. Note the overall project structure.

  ### Step 2: Identify the tech stack

  Look for dependency manifests: `package.json`, `pom.xml`, `build.gradle`, `requirements.txt`, `go.mod`, `Cargo.toml`, `*.csproj`, `Gemfile`. Read whichever are present to extract: primary language(s), frameworks, key libraries, and versions.

  ### Step 3: Identify the architectural pattern

  Look for structural clues:
  - Directories named `controllers/`, `services/`, `repositories/` → layered MVC
  - Directories named `domain/`, `application/`, `infrastructure/`, `ports/`, `adapters/` → hexagonal
  - Directories named `events/`, `handlers/`, `consumers/`, `producers/` → event-driven
  - Directories named `commands/`, `queries/` → CQRS

  Read 2–3 key source files at each layer boundary. Focus on interfaces and contracts, not implementation details.

  ### Step 4: Map module and service boundaries

  For each major module or service document:
  - What it owns
  - How it communicates with others (HTTP, gRPC, events, shared DB, direct import)
  - What external systems it depends on

  ### Step 5: List external dependencies

  Scan configuration files (`.env.example`, `application.yml`, `appsettings.json`, `config/`) for: database connection strings, message broker endpoints, cache addresses, third-party API base URLs.

  ### Step 6: Identify notable areas

  Given `goals`, identify 3–5 areas most architecturally relevant or risky:
  - Components the initiative will most likely touch
  - Areas with obvious complexity, tight coupling, or technical debt
  - Service boundaries that may need to change

  ### Output — overview mode

  Return the snapshot block followed by the suggested deep-dives list:

  ```
  ## Codebase Context: [codebase_path]

  ### Overview
  **Tech stack:** [languages, frameworks, key libraries with versions]
  **Architecture pattern:** [e.g. layered MVC, hexagonal, event-driven]
  **Module/service boundaries:**
  - [Module/service name]: [brief purpose, communication style]
  **External dependencies:** [databases, queues, caches, third-party APIs]
  **Notable observations:** [areas of complexity, technical debt, or architectural significance]

  ### Suggested Deep-Dives
  1. [Area name] — [one sentence: why this is relevant to the stated goals]
  2. [Area name] — [one sentence]
  3. [Area name] — [one sentence]
  ```

  ---

  ## Deep-Dive Mode

  ### Step 1: Locate the focus area

  Use Glob and Grep to find files most relevant to `focus_area`:
  - `Glob` pattern: `[codebase_path]/**/*[focus_area]*`
  - Grep for the focus area name in source files

  ### Step 2: Trace entry points

  Identify how this area is invoked: HTTP handler, event consumer, scheduled job, CLI command, or direct import. Read the entry point file.

  ### Step 3: Follow the execution path

  Trace the main path: entry point → domain logic → data access → external calls. Read each layer in turn. Stop once you've covered the key path — do not read every edge case.

  ### Step 4: Document component relationships

  Map what this area depends on and what depends on it.

  ### Step 5: Note data flows

  How does data enter, transform, and exit this area? What is stored and where?

  ### Output — deep-dive mode

  Return the finding block for the focus area:

  ```
  #### [focus_area]

  **Entry points:** [how this area is invoked]
  **Execution path:** [sequential steps: A → B → C → D]
  **Component dependencies:**
  - Depends on: [list]
  - Depended on by: [list]
  **Data flow:** [what data enters, transforms, exits; what is persisted and where]
  **Patterns in use:** [architectural patterns observed]
  **Architectural notes:** [anything significant for the initiative: coupling, hidden dependencies, drift from stated architecture, migration implications]
  ```
  ````

- [ ] **Step 2: Verify frontmatter fields**

  ```bash
  grep -n "^name:\|^description:\|^tools:\|^model:\|^color:" "architecture-design/.claude-plugin/agents/arch-code-explorer.md"
  ```
  Expected: all five fields present.

- [ ] **Step 3: Verify both mode sections exist**

  ```bash
  grep -n "## Overview Mode\|## Deep-Dive Mode" "architecture-design/.claude-plugin/agents/arch-code-explorer.md"
  ```
  Expected: both lines present.

- [ ] **Step 4: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-code-explorer.md"
  git commit -m "feat: add arch-code-explorer agent — overview and deep-dive modes"
  ```

---

## Task 2: Update arch-orchestrator — add Phase 2b and thread codebase_context

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-orchestrator.md`

- [ ] **Step 1: Update the frontmatter description**

  Change the `description` field from:
  ```
  description: Controls the full 10-phase create-architect workflow. Manages discovery, parallel agent dispatch, approval gates, and final document output. Never produces design content itself — coordinates agents only.
  ```
  To:
  ```
  description: Controls the full 11-phase create-architect workflow. Manages discovery, parallel agent dispatch, optional codebase exploration, approval gates, and final document output. Never produces design content itself — coordinates agents only.
  ```

- [ ] **Step 2: Update the TodoWrite reference**

  Change:
  ```
  Start by creating a TodoWrite task list with all 10 phases. Mark each complete as you finish it.
  ```
  To:
  ```
  Start by creating a TodoWrite task list with all 11 phases. Mark each complete as you finish it.
  ```

- [ ] **Step 3: Insert Phase 2b section after Phase 2**

  After the closing `---` of the Phase 2 section, insert the following new section:

  ```markdown
  ## Phase 2b — Codebase Exploration (optional)

  Ask the user:
  > "Is there an existing codebase for this initiative I should explore? If so, provide the path (or say 'no' to skip)."

  **If the user says no or skips:** Set `codebase_context` to `null` and proceed to Phase 3.

  **If the user provides a path (`codebase_path`):**

  1. Spawn `arch-code-explorer` in overview mode with `codebase_path`, `initiative_name`, and `goals`. Receive the architectural snapshot and `suggested_deep_dives`.

  2. Present the snapshot to the user, then ask:
     > "Based on your goals, I'd suggest exploring these areas in more depth: [suggested_deep_dives list]. Would you like to add, remove, or adjust any before I proceed?"

  3. Wait for the user's response. Collect the confirmed list of focus areas.

  4. Spawn one `arch-code-explorer` instance per confirmed focus area simultaneously in deep-dive mode, each receiving `codebase_path`, `initiative_name`, `goals`, and `focus_area`. Collect all finding blocks.

  5. Assemble `codebase_context` by combining the overview snapshot and all deep-dive findings:
     ```
     ## Codebase Context: [codebase_path]

     ### Overview
     [overview snapshot from step 1]

     ### Deep-Dive Findings
     [all finding blocks from step 4, one per confirmed focus area]
     ```

  Proceed to Phase 3.

  ---
  ```

- [ ] **Step 4: Update Phase 3 to incorporate codebase_context**

  Change the Phase 3 opening from:
  ```
  ## Phase 3 — Clarifying Questions

  Review `initiative_input`, `goals`, `constraints`, and `exploration_context`. Identify gaps that would require assumptions during design.
  ```
  To:
  ```
  ## Phase 3 — Clarifying Questions

  Review `initiative_input`, `goals`, `constraints`, `exploration_context`, and `codebase_context`. Identify gaps that would require assumptions during design.

  If `codebase_context` is non-null:
  - Skip questions already answered by the codebase exploration (e.g. do not ask about tech stack if it was observed directly).
  - Surface contradictions: if the codebase reveals something that conflicts with a stated goal or constraint, ask about it explicitly before proceeding. Example: "The codebase uses PostgreSQL but you listed a constraint to adopt MongoDB — is that intentional?"
  ```

- [ ] **Step 5: Update Phase 4 (strategist) to pass codebase_context**

  Change:
  ```
  Invoke `arch-strategist` with:
  - `initiative_name`, `goals`, `constraints`
  - `clarification_context`, `exploration_context`
  ```
  To:
  ```
  Invoke `arch-strategist` with:
  - `initiative_name`, `goals`, `constraints`
  - `clarification_context`, `exploration_context`, `codebase_context`
  ```

- [ ] **Step 6: Update Phase 5 (DDD) to pass codebase_context**

  Change:
  ```
  Invoke `arch-agent-ddd` with the full context:
  - `initiative_name`, `goals`, `constraints`
  - `clarification_context`, `exploration_context`
  - `chosen_approach`, `approach_rationale`
  ```
  To:
  ```
  Invoke `arch-agent-ddd` with the full context:
  - `initiative_name`, `goals`, `constraints`
  - `clarification_context`, `exploration_context`, `codebase_context`
  - `chosen_approach`, `approach_rationale`
  ```

- [ ] **Step 7: Update Phase 6 context object to include codebase_context**

  Change:
  ```
  {
    initiative_name, goals, constraints,
    clarification_context, exploration_context,
    chosen_approach, approach_rationale,
    ddd_output
  }
  ```
  To:
  ```
  {
    initiative_name, goals, constraints,
    clarification_context, exploration_context,
    codebase_context,
    chosen_approach, approach_rationale,
    ddd_output
  }
  ```

- [ ] **Step 8: Verify Phase 2b and codebase_context threading**

  ```bash
  grep -n "Phase 2b\|codebase_context\|arch-code-explorer" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
  ```
  Expected: `Phase 2b` appears once, `codebase_context` appears in Phase 2b, Phase 3, Phase 4, Phase 5, and Phase 6. `arch-code-explorer` appears twice (overview spawn + parallel deep-dives).

- [ ] **Step 9: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
  git commit -m "feat: add Phase 2b codebase exploration to arch-orchestrator"
  ```

---

## Task 3: Update arch-agent-ddd — codebase_context input and diagram conventions

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-agent-ddd.md`

- [ ] **Step 1: Update the Input section**

  Change:
  ```
  ## Input
  Full context: `initiative_name`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `chosen_approach`, `approach_rationale`.
  ```
  To:
  ```
  ## Input
  Full context: `initiative_name`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `codebase_context` (null for greenfield), `chosen_approach`, `approach_rationale`.
  ```

- [ ] **Step 2: Update Step 2 to use existing boundaries from codebase_context**

  Change:
  ```
  ## Step 2: Derive service candidates

  From bounded contexts, identify service candidates. Each bounded context becomes a service candidate. Name services using the ubiquitous language.
  ```
  To:
  ```
  ## Step 2: Derive service candidates

  From bounded contexts, identify service candidates. Each bounded context becomes a service candidate. Name services using the ubiquitous language.

  If `codebase_context` is non-null, use the module/service boundaries documented there as candidate starting points. For each candidate, determine its status:
  - **Existing** — already present in the codebase with no structural change needed
  - **Modified** — already present but requires structural changes for this initiative
  - **New** — does not exist in the codebase
  ```

- [ ] **Step 3: Update Step 4 diagram conventions**

  Change the Conventions block from:
  ```
  Conventions:
  - Services: `[Service Name]`
  - External systems: `((External System))`
  - Shared infrastructure: `[[Infrastructure Name]]`
  - Synchronous calls: `──[PROTOCOL]──►` with protocol label (HTTPS, gRPC)
  - Asynchronous events: `- - [EVENT] - - ►` with event/topic name
  ```
  To:
  ```
  Conventions:
  - Existing services (no structural change): `[Service Name]`
  - Modified services: `[MOD: Service Name]`
  - New services: `[NEW: Service Name]`
  - External systems: `((External System))`
  - Shared infrastructure: `[[Infrastructure Name]]`
  - Synchronous calls: `──[PROTOCOL]──►` with protocol label (HTTPS, gRPC)
  - Asynchronous events: `- - [EVENT] - - ►` with event/topic name

  If `codebase_context` is null (greenfield), all services are implicitly new — use plain `[Service Name]` notation.
  ```

- [ ] **Step 4: Verify changes**

  ```bash
  grep -n "codebase_context\|MOD:\|NEW:" "architecture-design/.claude-plugin/agents/arch-agent-ddd.md"
  ```
  Expected: `codebase_context` appears in Input and Step 2; `MOD:` and `NEW:` appear in Step 4 conventions.

- [ ] **Step 5: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-agent-ddd.md"
  git commit -m "feat: update arch-agent-ddd to use codebase_context for brownfield boundaries"
  ```

---

## Task 4: Update arch-strategist — codebase_context, rewrite risk, strangler fig

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-strategist.md`

- [ ] **Step 1: Update the Input section**

  Change:
  ```
  ## Input
  Full context: `initiative_name`, `goals`, `constraints`, `clarification_context`, `exploration_context`.
  ```
  To:
  ```
  ## Input
  Full context: `initiative_name`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `codebase_context` (null for greenfield).
  ```

- [ ] **Step 2: Add codebase-aware pattern selection to Step 1**

  After the pattern candidate list in Step 1, add:
  ```
  If `codebase_context` is non-null (brownfield):
  - Always include **Strangler fig** as a candidate — it minimises big-bang rewrite risk by default.
  - Assess rewrite risk for each pattern by reviewing the module/service boundaries in `codebase_context`. A tightly coupled codebase with no clear service boundaries makes microservices and event-driven patterns higher risk and should be rated accordingly.
  ```

- [ ] **Step 3: Add Rewrite risk row to the trade-offs table**

  Change the trade-offs table from:
  ```
  | Dimension | Assessment |
  |-----------|-----------|
  | Complexity | Low / Medium / High |
  | Team fit | [given stated team constraints] |
  | Time to first delivery | [rough estimate] |
  | Operational cost | Low / Medium / High |
  | Constraint satisfaction | [which constraints it meets or violates] |
  ```
  To:
  ```
  | Dimension | Assessment |
  |-----------|-----------|
  | Complexity | Low / Medium / High |
  | Team fit | [given stated team constraints] |
  | Time to first delivery | [rough estimate] |
  | Operational cost | Low / Medium / High |
  | Constraint satisfaction | [which constraints it meets or violates] |
  | Rewrite risk | Low / Medium / High — based on codebase coupling observed in `codebase_context` (omit row if `codebase_context` is null) |
  ```

- [ ] **Step 4: Verify changes**

  ```bash
  grep -n "codebase_context\|Strangler fig\|Rewrite risk" "architecture-design/.claude-plugin/agents/arch-strategist.md"
  ```
  Expected: `codebase_context` in Input; `Strangler fig` and `Rewrite risk` in Step 1/Step 3.

- [ ] **Step 5: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-strategist.md"
  git commit -m "feat: update arch-strategist to use codebase_context for rewrite risk assessment"
  ```

---

## Task 5: Update the five Phase 6 specialist agents

**Files:**
- Modify: `architecture-design/.claude-plugin/agents/arch-agent-api-event.md`
- Modify: `architecture-design/.claude-plugin/agents/arch-agent-data.md`
- Modify: `architecture-design/.claude-plugin/agents/arch-agent-deployment.md`
- Modify: `architecture-design/.claude-plugin/agents/arch-agent-security.md`
- Modify: `architecture-design/.claude-plugin/agents/arch-agent-ops.md`

All five follow the same pattern: (a) add `codebase_context` to the Input line, then (b) add one codebase-aware step to the Process section.

- [ ] **Step 1: Update arch-agent-api-event**

  **Input line** — change:
  ```
  Full context object including `ddd_output` (bounded_contexts, service_descriptions, ascii_diagram, domain_events), `goals`, `constraints`, `clarification_context`, `exploration_context`, `chosen_approach`.
  ```
  To:
  ```
  Full context object including `ddd_output` (bounded_contexts, service_descriptions, ascii_diagram, domain_events), `goals`, `constraints`, `clarification_context`, `exploration_context`, `codebase_context` (null for greenfield), `chosen_approach`.
  ```

  **Process section** — add as step 5 (after the existing step 4):
  ```
  5. If `codebase_context` is non-null, review the API styles documented in the codebase overview. Align new contracts with existing patterns where appropriate. Where a new contract intentionally diverges from an existing style, note it with: ⚠️ **Style divergence:** [reason].
  ```

- [ ] **Step 2: Update arch-agent-data**

  **Input line** — change:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `chosen_approach`.
  ```
  To:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `codebase_context` (null for greenfield), `chosen_approach`.
  ```

  **Step 5** — change:
  ```
  5. If migration from an existing system is implied by `exploration_context`, outline the migration approach (strangler fig, big-bang, dual-write).
  ```
  To:
  ```
  5. If migration from an existing system is implied by `exploration_context` or `codebase_context`, outline the migration approach (strangler fig, big-bang, dual-write). If `codebase_context` documents existing schemas or data stores, include specific schema migration steps for any entities that will change ownership or structure.
  ```

- [ ] **Step 3: Update arch-agent-deployment**

  **Input line** — change:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `chosen_approach`.
  ```
  To:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `codebase_context` (null for greenfield), `chosen_approach`.
  ```

  **Process section** — add as step 7 (after the existing step 6):
  ```
  7. If `codebase_context` is non-null, review existing infrastructure patterns documented in the codebase overview. Build on them where possible. Where a new infrastructure choice deviates from existing patterns, note it with: ⚠️ **Infrastructure deviation:** [reason].
  ```

- [ ] **Step 4: Update arch-agent-security**

  **Input line** — change:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context` (contains any compliance requirements: HIPAA, PCI-DSS, SOC 2, GDPR), `exploration_context`, `chosen_approach`.
  ```
  To:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context` (contains any compliance requirements: HIPAA, PCI-DSS, SOC 2, GDPR), `exploration_context`, `codebase_context` (null for greenfield), `chosen_approach`.
  ```

  **Process section** — add as step 5 (after the existing step 4):
  ```
  5. If `codebase_context` is non-null, compare the existing security posture documented in the codebase overview against the new design requirements. Produce a **Security gap analysis** listing each gap: what is missing or insufficient in the existing codebase, and what the new design must add or change to close it.
  ```

- [ ] **Step 5: Update arch-agent-ops**

  **Input line** — change:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `chosen_approach`.
  ```
  To:
  ```
  Full context object including `ddd_output`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `codebase_context` (null for greenfield), `chosen_approach`.
  ```

  **Process section** — add as step 6 (after the existing step 5):
  ```
  6. If `codebase_context` is non-null, review existing observability tooling documented in the codebase overview. Build on it where possible. Where new tooling deviates from existing patterns, note it with: ⚠️ **Observability deviation:** [reason].
  ```

- [ ] **Step 6: Verify all five files updated**

  ```bash
  for f in arch-agent-api-event arch-agent-data arch-agent-deployment arch-agent-security arch-agent-ops; do
    echo "=== $f ===" && grep -n "codebase_context" "architecture-design/.claude-plugin/agents/$f.md"
  done
  ```
  Expected: `codebase_context` appears at least twice in each file (Input line + process step).

- [ ] **Step 7: Commit**

  ```bash
  git add architecture-design/.claude-plugin/agents/arch-agent-api-event.md \
          architecture-design/.claude-plugin/agents/arch-agent-data.md \
          architecture-design/.claude-plugin/agents/arch-agent-deployment.md \
          architecture-design/.claude-plugin/agents/arch-agent-security.md \
          architecture-design/.claude-plugin/agents/arch-agent-ops.md
  git commit -m "feat: thread codebase_context through all Phase 6 specialist agents"
  ```

---

## Task 6: Final validation

- [ ] **Step 1: Verify arch-code-explorer exists and has required frontmatter**

  ```bash
  grep -n "^name:\|^description:\|^tools:\|^model:\|^color:" "architecture-design/.claude-plugin/agents/arch-code-explorer.md"
  ```
  Expected: all five fields present.

- [ ] **Step 2: Verify Phase 2b is in the orchestrator**

  ```bash
  grep -n "Phase 2b" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
  ```
  Expected: one match.

- [ ] **Step 3: Verify codebase_context is threaded through all modified agents**

  ```bash
  for f in arch-orchestrator arch-agent-ddd arch-strategist arch-agent-api-event arch-agent-data arch-agent-deployment arch-agent-security arch-agent-ops; do
    count=$(grep -c "codebase_context" "architecture-design/.claude-plugin/agents/$f.md")
    echo "$f: $count occurrences"
  done
  ```
  Expected: each file shows at least 1 occurrence; orchestrator shows the most (Phase 2b definition + Phase 3/4/5/6 references).

- [ ] **Step 4: Verify no agent still has the old input line without codebase_context**

  ```bash
  grep -rn "^Full context.*exploration_context.*chosen_approach" "architecture-design/.claude-plugin/agents/"
  ```
  Expected: 0 matches. (All input lines that previously ended with `exploration_context` or `chosen_approach` now include `codebase_context` in between.)

- [ ] **Step 5: Final commit if any files were missed**

  ```bash
  git status
  ```
  If any tracked files show as modified, review them and commit:
  ```bash
  git add [any remaining files]
  git commit -m "feat: complete brownfield codebase exploration integration"
  ```
