# create-architect Command Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement the `/create-architect` command as a 10-phase orchestrated workflow that guides users from initiative description to a complete Solution Intent document using parallel specialist agents.

**Architecture:** An `arch-orchestrator` agent controls all 10 phases, spawning `arch-explorer` agents in parallel during context gathering, enforcing two approval gates (architectural approach + DDD decomposition), running 5 specialist agents concurrently for deep design, then synthesizing, reviewing, optionally formatting, and writing the final document to `docs/`.

**Tech Stack:** Claude Code plugin system (markdown instruction files), Atlassian MCP (Confluence/Jira), GitHub MCP, 8 existing `arch-*` skills

---

## Important Format Notes

Derived from the `feature-dev` plugin (the reference implementation):

- **plugin.json**: minimal — name, description, author only. The plugin system auto-discovers agents, commands, and skills from the directory structure.
- **Command files**: frontmatter with `description` and `argument-hint`. Use `$ARGUMENTS` to reference user-provided arguments.
- **Agent files**: frontmatter with `name`, `description`, `tools` (comma-separated list), `model`, and optionally `color`.
- **Existing structure discrepancy**: the project uses `command/` (singular) while feature-dev uses `commands/` (plural). Verify during Task 1 which the plugin loader expects — rename if needed.

---

## File Map

**Create:**
- `architecture-design/.claude-plugin/plugin.json`
- `architecture-design/.claude-plugin/commands/create-architecture.md` *(rename existing `command/` dir to `commands/` if required)*
- `architecture-design/.claude-plugin/agents/arch-orchestrator.md`
- `architecture-design/.claude-plugin/agents/arch-explorer.md`
- `architecture-design/.claude-plugin/agents/arch-strategist.md`
- `architecture-design/.claude-plugin/agents/arch-agent-ddd.md`
- `architecture-design/.claude-plugin/agents/arch-agent-api-event.md`
- `architecture-design/.claude-plugin/agents/arch-agent-data.md`
- `architecture-design/.claude-plugin/agents/arch-agent-deployment.md`
- `architecture-design/.claude-plugin/agents/arch-agent-security.md`
- `architecture-design/.claude-plugin/agents/arch-agent-ops.md`
- `architecture-design/.claude-plugin/agents/arch-synthesizer.md`
- `architecture-design/.claude-plugin/agents/arch-reviewer.md`
- `architecture-design/.claude-plugin/skills/arch-template-formatter/SKILL.md`
- `architecture-design/.claude-plugin/skills/arch-template-formatter/references/solution-intent-template.md`

**Modify:**
- `architecture-design/.claude-plugin/command/create-architecture.md` → renamed/moved to `commands/` if needed

---

## Task 1: Create plugin.json and verify directory structure

**Files:**
- Create: `architecture-design/.claude-plugin/plugin.json`
- Verify: `architecture-design/.claude-plugin/` directory layout matches plugin system expectations

- [ ] **Step 1: Check whether `command/` or `commands/` is the correct folder name**

  Run:
  ```bash
  ls "architecture-design/.claude-plugin/"
  ```
  Compare against feature-dev which uses `commands/` (plural). If the plugin loader requires `commands/`, rename:
  ```bash
  mv "architecture-design/.claude-plugin/command" "architecture-design/.claude-plugin/commands"
  ```
  If `command/` is already correct for this setup, skip the rename.

- [ ] **Step 2: Create plugin.json**

  Create `architecture-design/.claude-plugin/plugin.json`:
  ```json
  {
    "name": "architect-ai",
    "description": "Enterprise architecture design assistant with guided 10-phase workflow producing a Solution Intent document",
    "author": {
      "name": "chun",
      "email": "chun.du@dfwit.org"
    }
  }
  ```

- [ ] **Step 3: Verify JSON is valid**

  Run:
  ```bash
  python3 -c "import json; json.load(open('architecture-design/.claude-plugin/plugin.json')); print('valid')"
  ```
  Expected: `valid`

- [ ] **Step 4: Commit**

  ```bash
  git add architecture-design/.claude-plugin/plugin.json
  git commit -m "feat: add plugin.json manifest for architect-ai plugin"
  ```

---

## Task 2: Implement the create-architecture command entry point

**Files:**
- Modify: `architecture-design/.claude-plugin/commands/create-architecture.md` (or `command/`)

- [ ] **Step 1: Write the command file**

  Write `architecture-design/.claude-plugin/commands/create-architecture.md`:
  ```markdown
  ---
  description: Design a system architecture through a guided 10-phase workflow, producing a Solution Intent document
  argument-hint: "[initiative description] [--template <path>]"
  ---

  # create-architect Command

  You are the entry point for the `/create-architect` command. Your only job is to parse the user's arguments and hand off to the `arch-orchestrator` agent.

  ## Parsing Arguments

  Raw arguments: `$ARGUMENTS`

  1. If `--template <value>` appears anywhere in the arguments, extract `<value>` as `template_path` and remove `--template <value>` from the remaining text.
  2. Everything remaining is `initiative_input`. It may be empty, a plain description, a structured brief, or a URL.

  ## Handoff

  Invoke the `arch-orchestrator` agent immediately with:
  - `initiative_input`: the parsed initiative text (empty string if nothing was provided)
  - `template_path`: the template path (null if `--template` was not present)

  Do not do any analysis or design work yourself. The orchestrator manages all phases.
  ```

- [ ] **Step 2: Verify frontmatter fields are present**

  Check the file contains `description:` and `argument-hint:` in frontmatter, and `$ARGUMENTS` in the body.
  ```bash
  grep -n "description:\|argument-hint:\|\$ARGUMENTS" "architecture-design/.claude-plugin/commands/create-architecture.md"
  ```
  Expected: all three lines present.

- [ ] **Step 3: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/commands/create-architecture.md"
  git commit -m "feat: implement create-architecture command entry point"
  ```

---

## Task 3: Create arch-orchestrator agent

**Files:**
- Create: `architecture-design/.claude-plugin/agents/arch-orchestrator.md`

- [ ] **Step 1: Write the orchestrator agent file**

  Create `architecture-design/.claude-plugin/agents/arch-orchestrator.md`:
  ```markdown
  ---
  name: arch-orchestrator
  description: Controls the full 10-phase create-architect workflow. Manages discovery, parallel agent dispatch, approval gates, and final document output. Never produces design content itself — coordinates agents only.
  tools: Read, Write, WebFetch, TodoWrite, Agent
  model: sonnet
  color: purple
  ---

  # arch-orchestrator

  You are the `arch-orchestrator`. You control the complete 10-phase architecture design workflow. You coordinate agents and manage approval gates — you never produce design content yourself.

  Start by creating a TodoWrite task list with all 10 phases. Mark each complete as you finish it.

  ## Inputs
  - `initiative_input`: raw text, structured brief, URL, or empty string
  - `template_path`: path to output template or null

  ---

  ## Phase 1 — Discovery

  Classify `initiative_input`:

  - **Empty**: Ask the user: "Please describe the initiative you'd like to architect. You can provide a brief description, a structured brief with goals/constraints/non-goals, or a URL to a Confluence page, Jira epic, or GitHub README."
  - **URL** (starts with `http://` or `https://`): Treat as a document reference. Add to `referenced_systems`.
  - **Structured brief** (contains "Goals:", "Constraints:", "Non-goals:", or `##` headings): Parse field by field.
  - **Free text**: Treat as freeform description.

  Extract and record:
  - `initiative_name`: 3–5 word title derived from input (used for kebab-case filename)
  - `goals`: list of stated goals (empty if none)
  - `constraints`: list of stated constraints (empty if none)
  - `referenced_systems`: list of any URLs or named external systems mentioned

  ---

  ## Phase 2 — Context Exploration (parallel)

  If `referenced_systems` is non-empty, spawn one `arch-explorer` agent per item simultaneously. Pass each item as the sole input to the agent.

  Collect all returned context blocks into `exploration_context`.

  If `referenced_systems` is empty, set `exploration_context` to `[]` and proceed.

  ---

  ## Phase 3 — Clarifying Questions

  Review `initiative_input`, `goals`, `constraints`, and `exploration_context`. Identify gaps that would require assumptions during design.

  Ask questions one at a time. Scale quantity to complexity:
  - Clear, well-scoped initiative: 1–2 questions
  - Moderate complexity: 3–4 questions
  - Ambiguous or large scope: 5–8 questions

  Target areas (only ask what's not already known):
  - Expected scale: users, request volume, data volume
  - Existing systems to integrate with beyond what was already captured
  - Team constraints: size, technology familiarity, existing commitments
  - Build vs. buy: auth, payments, notifications, search
  - Compliance: HIPAA, PCI-DSS, SOC 2, GDPR
  - Phasing preference: MVP scope vs. full solution

  Stop when you can proceed without significant design assumptions.

  Record all answers in `clarification_context`.

  ---

  ## Phase 4 — High-Level Approaches (approval gate)

  Invoke `arch-strategist` with:
  - `initiative_name`, `goals`, `constraints`
  - `clarification_context`, `exploration_context`

  The strategist presents 2–3 approaches and asks the user to choose. Wait for the user's selection.

  Record: `chosen_approach`, `approach_rationale`, `tech_flags`.

  ---

  ## Phase 5 — Domain Design (approval gate)

  Invoke `arch-agent-ddd` with the full context:
  - `initiative_name`, `goals`, `constraints`
  - `clarification_context`, `exploration_context`
  - `chosen_approach`, `approach_rationale`

  Receive back: `bounded_contexts`, `service_descriptions`, `ascii_diagram`, `domain_events`.

  Present to user:
  ```
  Here is the proposed service decomposition:

  [ascii_diagram]

  **Services:**
  [service_descriptions]

  **Domain Events:**
  [domain_events]

  Does this decomposition look right? Approve to proceed, or let me know what to adjust.
  ```

  Wait for explicit approval. If changes requested, re-invoke `arch-agent-ddd` with the feedback appended to context. Repeat until approved.

  Store approved output as `ddd_output`.

  ---

  ## Phase 6 — Parallel Deep Design

  Default: run all five specialist agents. Omit any the user explicitly excluded during Phase 3.

  Spawn these agents simultaneously, each receiving the full context object:
  ```
  {
    initiative_name, goals, constraints,
    clarification_context, exploration_context,
    chosen_approach, approach_rationale,
    ddd_output
  }
  ```

  Agents to spawn in parallel:
  - `arch-agent-api-event`
  - `arch-agent-data`
  - `arch-agent-deployment`
  - `arch-agent-security`
  - `arch-agent-ops`

  Collect: `api_event_output`, `data_output`, `deployment_output`, `security_output`, `ops_output`.

  ---

  ## Phase 7 — Synthesis

  Invoke `arch-synthesizer` with all collected outputs. Receive `solution_intent_draft`.

  ---

  ## Phase 8 — Review

  Invoke `arch-reviewer` with:
  - `document`: `solution_intent_draft`
  - `pass_number`: 1

  Receive `review_findings`.

  If `critical_count > 0`, present all 🔴 Critical findings to the user:
  ```
  The architecture reviewer found critical issues that must be resolved before finalizing:

  [each critical finding with its fix instruction]

  Please indicate how you'd like to address each, or approve the suggested fixes.
  ```

  Incorporate user responses, update `solution_intent_draft`, and run a second review pass. Continue until no critical findings remain.

  Append all remaining findings (🟡 Important, 🟢 Minor) to Section 12 of the document.

  ---

  ## Phase 9 — Template Formatting (optional)

  Determine template to use (in priority order):
  1. `template_path` if not null — read file at that path
  2. `architecture-design/.claude-plugin/skills/arch-template-formatter/references/solution-intent-template.md` if it exists
  3. No template — skip Phase 9

  If a template was found, invoke the `arch-template-formatter` skill with the document and template. Use the formatted output as the final document.

  ---

  ## Phase 10 — Finalize

  Derive filename: convert `initiative_name` to kebab-case and append `-solution-intent.md`.
  Example: "Customer Notification Service" → `customer-notification-service-solution-intent.md`

  Write the final document to: `docs/<filename>`

  Confirm to user:
  > "Architecture design complete. Solution Intent saved to `docs/<filename>`."
  ```

- [ ] **Step 2: Verify agent frontmatter**

  ```bash
  grep -n "^name:\|^description:\|^tools:\|^model:" "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
  ```
  Expected: all four fields present in frontmatter.

- [ ] **Step 3: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-orchestrator.md"
  git commit -m "feat: add arch-orchestrator agent — 10-phase workflow controller"
  ```

---

## Task 4: Create arch-explorer and arch-strategist agents

**Files:**
- Create: `architecture-design/.claude-plugin/agents/arch-explorer.md`
- Create: `architecture-design/.claude-plugin/agents/arch-strategist.md`

- [ ] **Step 1: Write arch-explorer**

  Create `architecture-design/.claude-plugin/agents/arch-explorer.md`:
  ```markdown
  ---
  name: arch-explorer
  description: Explores a single referenced system or URL and returns a structured context block for the architecture design workflow. Uses Atlassian MCP for Confluence/Jira and GitHub MCP for repositories. Runs in parallel — one instance per referenced system.
  tools: Read, WebFetch, mcp__claude_ai_Atlassian__getConfluencePage, mcp__claude_ai_Atlassian__getConfluencePageDescendants, mcp__claude_ai_Atlassian__getJiraIssue, mcp__claude_ai_Atlassian__search, mcp__claude_ai_Atlassian__searchJiraIssuesUsingJql
  model: sonnet
  color: yellow
  ---

  # arch-explorer

  You explore a single referenced system or document and return a structured context block. You run in parallel with other explorers — focus only on your assigned reference.

  ## Input
  A single `system_reference`: a URL, Atlassian page name, Jira issue key, or system name.

  ## Step 1: Identify source type

  | Pattern | Action |
  |---------|--------|
  | Confluence URL or `/wiki/` path | Use `mcp__claude_ai_Atlassian__getConfluencePage` |
  | Jira URL or issue key (`PROJ-123`) | Use `mcp__claude_ai_Atlassian__getJiraIssue` |
  | GitHub URL (`github.com/<org>/<repo>`) | Use `WebFetch` to read README and any `/docs` or `/adr` folder |
  | Plain system name | Use `mcp__claude_ai_Atlassian__search` to find relevant pages first |

  ## Step 2: Retrieve content

  Fetch the primary document. If it links to child pages or referenced docs that would add architectural context (API specs, data models, ADRs), fetch one level deep.

  ## Step 3: Extract and return context block

  Return this exact structure as markdown:

  ```
  ## Context: [system_name]

  **Tech stack:** [languages, frameworks, databases, messaging platforms]

  **Interfaces exposed:**
  - [API endpoint or event topic]: [brief description]

  **Data owned:**
  - [Entity/dataset]: [brief description]

  **Upstream dependencies:** [systems this one calls or reads from]

  **Downstream dependencies:** [systems that call or consume from this one]

  **Key constraints:** [technical, operational, or compliance constraints mentioned]

  **Relevant notes:** [anything architecturally significant not captured above]
  ```

  Return only the context block. No other output.
  ```

- [ ] **Step 2: Write arch-strategist**

  Create `architecture-design/.claude-plugin/agents/arch-strategist.md`:
  ```markdown
  ---
  name: arch-strategist
  description: Presents 2-3 high-level architectural approaches with trade-offs, validates technology choices against the approved tech stack, recommends one option, and waits for user selection.
  tools: Read, TodoWrite
  model: sonnet
  color: green
  ---

  # arch-strategist

  You present 2–3 genuinely distinct architectural approaches for the initiative, recommend one, and wait for the user to choose.

  ## Input
  Full context: `initiative_name`, `goals`, `constraints`, `clarification_context`, `exploration_context`.

  ## Step 1: Identify 2–3 relevant patterns

  Select patterns that genuinely fit the initiative. Do not include a pattern just to fill a slot. Common candidates:

  - **Event-driven microservices** — async, decoupled, scales independently; higher operational complexity
  - **Modular monolith** — lower initial complexity, faster to build; harder to scale individual components
  - **Strangler fig** — for incrementally replacing an existing system; minimises big-bang risk
  - **CQRS + Event Sourcing** — for audit-heavy or complex query requirements; significant complexity overhead
  - **API Gateway + BFF (Backend-for-Frontend)** — for multi-channel apps with distinct client needs
  - **Serverless / FaaS** — for event-triggered, variable-load workloads; cold start and vendor lock-in trade-offs

  ## Step 2: Validate technology implications

  For each approach, invoke the `arch-tech-stack` skill to check whether the key technologies it implies (messaging, compute, database) are Approved, Conditionally Approved, or Exception Required. Note any exception flags in the trade-offs.

  ## Step 3: Present each option using this format

  ```
  ### Option [A / B / C]: [Pattern Name]

  **Core pattern:** [one sentence]

  **Why it fits:** [2–3 sentences specific to this initiative's goals and constraints]

  **Trade-offs:**
  | Dimension | Assessment |
  |-----------|-----------|
  | Complexity | Low / Medium / High |
  | Team fit | [given stated team constraints] |
  | Time to first delivery | [rough estimate] |
  | Operational cost | Low / Medium / High |
  | Constraint satisfaction | [which constraints it meets or violates] |

  **Technology implications:**
  [Key technology choices implied; note tier (Approved / Conditional / Exception) for each]

  **Key risks for this initiative:**
  - [Risk 1]
  - [Risk 2]
  ```

  End with a recommendation block:
  > **Recommendation: Option [X]** — [2 sentences: why this is the strongest fit for the stated goals and constraints.]

  Then ask: "Which approach would you like to pursue? (Or let me know if you'd like to explore a hybrid.)"

  ## Output

  Once the user responds, return: `chosen_approach` (option name), `approach_rationale` (user's reason or the recommendation rationale), `tech_flags` (any exception-tier technologies from the chosen option).
  ```

- [ ] **Step 3: Verify both files have correct frontmatter**

  ```bash
  grep -n "^name:\|^description:\|^tools:\|^model:" "architecture-design/.claude-plugin/agents/arch-explorer.md" "architecture-design/.claude-plugin/agents/arch-strategist.md"
  ```
  Expected: all four fields present in both files.

- [ ] **Step 4: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-explorer.md" "architecture-design/.claude-plugin/agents/arch-strategist.md"
  git commit -m "feat: add arch-explorer and arch-strategist agents"
  ```

---

## Task 5: Create arch-agent-ddd

**Files:**
- Create: `architecture-design/.claude-plugin/agents/arch-agent-ddd.md`

- [ ] **Step 1: Write the DDD agent**

  Create `architecture-design/.claude-plugin/agents/arch-agent-ddd.md`:
  ```markdown
  ---
  name: arch-agent-ddd
  description: Performs domain-driven design analysis producing bounded contexts, per-service descriptions, ASCII connection diagram, and cross-context domain events. Blocking phase — output must be user-approved before Phase 6 starts.
  tools: Read, TodoWrite
  model: sonnet
  color: blue
  ---

  # arch-agent-ddd

  You perform the domain design phase. You apply the `arch-domain-driven-design` skill and additionally produce an ASCII service connection diagram and per-service descriptions. Your output is the foundation that all Phase 6 specialist agents depend on.

  ## Input
  Full context: `initiative_name`, `goals`, `constraints`, `clarification_context`, `exploration_context`, `chosen_approach`, `approach_rationale`.

  ## Step 1: Apply the arch-domain-driven-design skill

  Use the `arch-domain-driven-design` skill in full. Produce: subdomain map, bounded contexts, aggregates and entities, ubiquitous language, context map, and domain events.

  ## Step 2: Derive service candidates

  From bounded contexts, identify service candidates. Each bounded context becomes a service candidate. Name services using the ubiquitous language.

  ## Step 3: Write per-service descriptions

  For each service, produce:
  ```
  **[Service Name]**
  - Bounded context: [context name]
  - Responsibilities: [what it does — 2–3 bullet points]
  - Data owned: [key aggregates/entities]
  - Key operations: [primary commands/queries]
  - Sync interfaces: [APIs it exposes or calls synchronously]
  - Async interfaces: [events it publishes or consumes]
  ```

  ## Step 4: Draw the ASCII connection diagram

  Draw an ASCII diagram showing all services, their connections, and external integrations.

  Conventions:
  - Services: `[Service Name]`
  - External systems: `((External System))`
  - Shared infrastructure: `[[Infrastructure Name]]`
  - Synchronous calls: `──[PROTOCOL]──►` (label above line with protocol: HTTPS, gRPC)
  - Asynchronous events: `- - [EVENT]- - ►` (label with event/topic name)

  Example:
  ```
  ((Mobile Client)) ──HTTPS──► [API Gateway]
                                     │
                      ┌──────────────┼──────────────┐
                      ▼              ▼              ▼
               [User Service]  [Order Service]  [Notify Service]
                      │              │
                      └──────────────┘
                             │ OrderPlaced
                             ▼
                     [[Event Bus (Kafka)]]
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
            [Audit Service]   [Notify Service]
  ```

  ## Output

  Return all four components:

  - `bounded_contexts`: full DDD analysis output
  - `service_descriptions`: per-service blocks as above
  - `ascii_diagram`: the ASCII connection diagram
  - `domain_events`: table with columns — Event Name, Publisher, Consumer(s), Trigger
  ```

- [ ] **Step 2: Verify frontmatter**

  ```bash
  grep -n "^name:\|^description:\|^tools:\|^model:" "architecture-design/.claude-plugin/agents/arch-agent-ddd.md"
  ```
  Expected: all four fields present.

- [ ] **Step 3: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-agent-ddd.md"
  git commit -m "feat: add arch-agent-ddd — domain design with ASCII diagram"
  ```

---

## Task 6: Create Phase 6 specialist agents (5 agents)

**Files:**
- Create: `architecture-design/.claude-plugin/agents/arch-agent-api-event.md`
- Create: `architecture-design/.claude-plugin/agents/arch-agent-data.md`
- Create: `architecture-design/.claude-plugin/agents/arch-agent-deployment.md`
- Create: `architecture-design/.claude-plugin/agents/arch-agent-security.md`
- Create: `architecture-design/.claude-plugin/agents/arch-agent-ops.md`

All five agents follow the same pattern: apply their respective skill, invoke `arch-tech-stack` inline for any technology decision, return a markdown section ready for the Solution Intent.

- [ ] **Step 1: Write arch-agent-api-event**

  Create `architecture-design/.claude-plugin/agents/arch-agent-api-event.md`:
  ```markdown
  ---
  name: arch-agent-api-event
  description: Designs API and event contracts for all services using the arch-api-event skill. Runs in parallel with other Phase 6 specialist agents. Invokes arch-tech-stack inline for every technology choice.
  tools: Read, TodoWrite
  model: sonnet
  color: cyan
  ---

  # arch-agent-api-event

  You design the API and event architecture for all services in the decomposition.

  ## Input
  Full context object including `ddd_output` (bounded_contexts, service_descriptions, ascii_diagram, domain_events).

  ## Process

  1. Apply the `arch-api-event` skill using the service decomposition as input.
  2. For each API style choice (REST, gRPC, GraphQL, AsyncAPI), invoke the `arch-tech-stack` skill to confirm the tier (Approved / Conditional / Exception). Flag any Exception-tier choices prominently.
  3. For each service, specify:
     - API style and rationale
     - Key endpoint or operation contracts (focus on critical paths, not exhaustive list)
     - Event schemas for any domain events published by that service
  4. Define the overall versioning strategy (URI versioning, header versioning, etc.)

  ## Output

  Return a complete markdown section titled `## 5. API & Event Design` ready for insertion into the Solution Intent. Include:
  - API style decisions per service with tier validation
  - Critical endpoint/operation contracts
  - Event schemas for all cross-context domain events
  - Versioning strategy
  - Any ⚠️ Exception-tier technology flags with justification
  ```

- [ ] **Step 2: Write arch-agent-data**

  Create `architecture-design/.claude-plugin/agents/arch-agent-data.md`:
  ```markdown
  ---
  name: arch-agent-data
  description: Designs data architecture including storage choices, data ownership, caching strategy, and migration approach using the arch-data skill. Runs in parallel with other Phase 6 specialist agents.
  tools: Read, TodoWrite
  model: sonnet
  color: cyan
  ---

  # arch-agent-data

  You design the data architecture for all services in the decomposition.

  ## Input
  Full context object including `ddd_output`.

  ## Process

  1. Apply the `arch-data` skill using the service decomposition as input.
  2. For each storage technology choice (relational DB, NoSQL, cache, object store), invoke the `arch-tech-stack` skill to confirm tier. Flag any Exception-tier choices.
  3. For each service, specify:
     - Primary data store: technology, justification, data schema (key entities only)
     - Read patterns: queries, indexes, caching needs
     - Write patterns: commands, consistency requirements
  4. Define the caching strategy: what is cached, where (CDN, in-process, distributed), and TTL approach.
  5. If migration from an existing system is implied by `exploration_context`, outline the migration approach (strangler fig, big-bang, dual-write).

  ## Output

  Return a complete markdown section titled `## 6. Data Architecture` ready for insertion into the Solution Intent. Include:
  - Per-service data store choices with tier validation
  - Data ownership map (which service owns which entity)
  - Caching strategy
  - Migration approach if applicable
  - Any ⚠️ Exception-tier technology flags with justification
  ```

- [ ] **Step 3: Write arch-agent-deployment**

  Create `architecture-design/.claude-plugin/agents/arch-agent-deployment.md`:
  ```markdown
  ---
  name: arch-agent-deployment
  description: Designs deployment and infrastructure architecture including compute, IaC, CI/CD, network, and environment strategy using the arch-deployment skill. Runs in parallel with other Phase 6 specialist agents.
  tools: Read, TodoWrite
  model: sonnet
  color: cyan
  ---

  # arch-agent-deployment

  You design the deployment and infrastructure architecture.

  ## Input
  Full context object including `ddd_output`.

  ## Process

  1. Apply the `arch-deployment` skill using the service decomposition and chosen approach as input.
  2. For each infrastructure technology choice (compute platform, container registry, IaC tool, CI/CD platform, secrets manager), invoke the `arch-tech-stack` skill to confirm tier. Flag any Exception-tier choices.
  3. Define:
     - Compute pattern per service type (EKS, ECS, Lambda, static hosting — whichever applies)
     - IaC approach and tooling
     - CI/CD pipeline stages (build → test → scan → deploy)
     - Environment strategy (dev, staging, prod; environment parity)
     - Network design: VPC, subnets, security group boundaries, ingress
     - Deployment pattern: blue/green, canary, rolling — per service

  ## Output

  Return a complete markdown section titled `## 7. Deployment & Infrastructure` ready for insertion into the Solution Intent. Include:
  - Per-service compute choices with tier validation
  - IaC and CI/CD toolchain
  - Environment strategy
  - Network design summary
  - Deployment patterns
  - Any ⚠️ Exception-tier technology flags with justification
  ```

- [ ] **Step 4: Write arch-agent-security**

  Create `architecture-design/.claude-plugin/agents/arch-agent-security.md`:
  ```markdown
  ---
  name: arch-agent-security
  description: Designs security architecture covering authentication, authorisation, secrets, network controls, and data protection using the arch-security skill. Runs in parallel with other Phase 6 specialist agents.
  tools: Read, TodoWrite
  model: sonnet
  color: cyan
  ---

  # arch-agent-security

  You design the security architecture across all layers of the solution.

  ## Input
  Full context object including `ddd_output`, API event design context, and any compliance constraints from `clarification_context`.

  ## Process

  1. Apply the `arch-security` skill using the full solution context.
  2. For each security tooling choice (identity provider, secrets manager, WAF, certificate management), invoke the `arch-tech-stack` skill to confirm tier.
  3. Define:
     - Authentication model: who authenticates, how (OAuth2/OIDC, mTLS, API key, etc.)
     - Authorisation model: RBAC, ABAC, or scope-based; where enforced
     - Secrets management: how secrets are stored, rotated, and accessed at runtime
     - Data protection: encryption at rest (key management), in transit (TLS versions, cipher suites)
     - Network controls: which services can call which, egress restrictions
     - Threat surface: key attack vectors for this specific architecture and mitigations
  4. If compliance requirements (PCI, HIPAA, SOC 2, GDPR) were stated in `clarification_context`, map each requirement to a specific design decision.

  ## Output

  Return a complete markdown section titled `## 8. Security Design` ready for insertion into the Solution Intent. Include:
  - AuthN/AuthZ model
  - Secrets management approach
  - Data protection (at rest and in transit)
  - Network controls
  - Threat surface and mitigations
  - Compliance mapping if applicable
  - Any ⚠️ Exception-tier technology flags with justification
  ```

- [ ] **Step 5: Write arch-agent-ops**

  Create `architecture-design/.claude-plugin/agents/arch-agent-ops.md`:
  ```markdown
  ---
  name: arch-agent-ops
  description: Designs production readiness covering observability, resiliency patterns, SLO targets, and cost strategy using the arch-production-operations skill. Runs in parallel with other Phase 6 specialist agents.
  tools: Read, TodoWrite
  model: sonnet
  color: cyan
  ---

  # arch-agent-ops

  You design the production readiness and operational architecture.

  ## Input
  Full context object including `ddd_output`.

  ## Process

  1. Apply the `arch-production-operations` skill using the service decomposition as input.
  2. For each observability tooling choice (logging platform, metrics platform, tracing platform, alerting), invoke the `arch-tech-stack` skill to confirm tier.
  3. Define:
     - **Observability stack**: structured logging (format, correlation IDs), metrics (what to measure per service), distributed tracing (instrumentation approach), alerting thresholds
     - **Resiliency patterns**: per service — circuit breakers, retry with backoff, bulkheads, dead letter queues for async flows
     - **Failure modes**: top 3 failure scenarios per service and the designed response
     - **SLO targets**: availability, latency (p50/p95/p99), error rate — per critical service
     - **Cost optimisation**: resource tagging strategy, right-sizing approach, reserved vs. on-demand split

  ## Output

  Return a complete markdown section titled `## 9. Production Readiness` ready for insertion into the Solution Intent. Include:
  - Observability stack choices with tier validation
  - Resiliency patterns per service
  - SLO targets table
  - Cost strategy
  - Any ⚠️ Exception-tier technology flags with justification
  ```

- [ ] **Step 6: Verify all five files have correct frontmatter**

  ```bash
  for f in arch-agent-api-event arch-agent-data arch-agent-deployment arch-agent-security arch-agent-ops; do
    echo "=== $f ===" && grep -n "^name:\|^description:\|^tools:\|^model:" "architecture-design/.claude-plugin/agents/$f.md"
  done
  ```
  Expected: all four frontmatter fields present in each file.

- [ ] **Step 7: Commit**

  ```bash
  git add architecture-design/.claude-plugin/agents/arch-agent-api-event.md \
          architecture-design/.claude-plugin/agents/arch-agent-data.md \
          architecture-design/.claude-plugin/agents/arch-agent-deployment.md \
          architecture-design/.claude-plugin/agents/arch-agent-security.md \
          architecture-design/.claude-plugin/agents/arch-agent-ops.md
  git commit -m "feat: add 5 Phase 6 specialist agents (API, data, deployment, security, ops)"
  ```

---

## Task 7: Create arch-synthesizer agent

**Files:**
- Create: `architecture-design/.claude-plugin/agents/arch-synthesizer.md`

- [ ] **Step 1: Write the synthesizer agent**

  Create `architecture-design/.claude-plugin/agents/arch-synthesizer.md`:
  ```markdown
  ---
  name: arch-synthesizer
  description: Merges all Phase 6 specialist agent outputs with the DDD decomposition into a single coherent Solution Intent document. Detects and flags cross-agent conflicts. Consolidates all technology decisions.
  tools: Read, Write, TodoWrite
  model: sonnet
  color: orange
  ---

  # arch-synthesizer

  You merge all specialist outputs into a single, coherent Solution Intent document.

  ## Input
  All context and outputs collected by the orchestrator through phases 1–6:
  - `initiative_name`, `goals`, `constraints`
  - `chosen_approach`, `approach_rationale`, `tech_flags`
  - `ddd_output` (bounded_contexts, service_descriptions, ascii_diagram, domain_events)
  - `api_event_output`, `data_output`, `deployment_output`, `security_output`, `ops_output`

  ## Step 1: Detect conflicts

  Before assembling, scan all specialist outputs for conflicts:
  - Two agents proposing incompatible technologies for the same layer (e.g., two messaging platforms)
  - Two agents claiming ownership of the same data entity
  - An infrastructure choice in one output that contradicts a constraint in another

  For each conflict found, create a flag:
  > ⚠️ **CONFLICT [C-001]:** [Agent A] proposes [X] for [purpose]; [Agent B] proposes [Y]. These are incompatible. Recommended resolution: [brief guidance].

  Collect all flags for Section 11.

  ## Step 2: Consolidate technology decisions

  Scan all specialist outputs for technology choices. Deduplicate and produce a unified table for Section 10:

  | Layer | Technology | Version | Tier | Notes |
  |-------|------------|---------|------|-------|
  | [e.g., Messaging] | [e.g., Kafka] | [x.y] | Approved | |

  Include all ⚠️ Exception-tier flags noted by any specialist.

  ## Step 3: Assemble the Solution Intent

  Produce the complete document using exactly this structure:

  ```markdown
  # Solution Intent: [initiative_name]

  ## 1. Executive Summary
  [3–5 sentences: what is being built, the chosen architectural approach, key services, and the primary value delivered]

  ## 2. Goals & Constraints
  ### Goals
  [bullet list from goals]
  ### Constraints
  [bullet list from constraints]

  ## 3. High-Level Architecture
  ### Chosen Approach
  [chosen_approach name and approach_rationale in 2–3 sentences]
  ### Service Diagram
  [ascii_diagram verbatim from ddd_output]

  ## 4. Domain Model & Service Decomposition
  [bounded_contexts summary]
  [service_descriptions verbatim — one block per service]

  ## 5. API & Event Design
  [api_event_output verbatim]

  ## 6. Data Architecture
  [data_output verbatim]

  ## 7. Deployment & Infrastructure
  [deployment_output verbatim]

  ## 8. Security Design
  [security_output verbatim]

  ## 9. Production Readiness
  [ops_output verbatim]

  ## 10. Technology Decisions
  [consolidated tech decisions table]

  ## 11. Open Questions & Risks
  [conflict flags (⚠️ CONFLICT entries)]
  [any open questions or risks surfaced by specialist agents during their analysis]

  ## 12. Review Findings
  *(Populated by arch-reviewer — leave this section empty)*
  ```

  ## Output

  Return `solution_intent_draft` — the complete markdown document text.
  ```

- [ ] **Step 2: Verify frontmatter**

  ```bash
  grep -n "^name:\|^description:\|^tools:\|^model:" "architecture-design/.claude-plugin/agents/arch-synthesizer.md"
  ```
  Expected: all four fields present.

- [ ] **Step 3: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-synthesizer.md"
  git commit -m "feat: add arch-synthesizer agent — merges all outputs into Solution Intent"
  ```

---

## Task 8: Create arch-reviewer agent

**Files:**
- Create: `architecture-design/.claude-plugin/agents/arch-reviewer.md`

- [ ] **Step 1: Write the reviewer agent**

  Create `architecture-design/.claude-plugin/agents/arch-reviewer.md`:
  ```markdown
  ---
  name: arch-reviewer
  description: Performs a cold peer review of the Solution Intent draft using the arch-review skill in orchestrated mode. Returns a structured REVIEW_FINDINGS block. Called by arch-orchestrator — not directly by users.
  tools: Read, TodoWrite
  model: sonnet
  color: red
  ---

  # arch-reviewer

  You perform a cold peer review of the Solution Intent draft. You operate in **orchestrated mode** — you return a structured findings block for the orchestrator to act on, not a file for the user.

  ## Input
  - `document`: the Solution Intent draft markdown text
  - `pass_number`: 1, 2, or 3 (which review pass this is)

  ## Process

  Apply the `arch-review` skill in full **orchestrated mode**. Evaluate the Solution Intent against all four dimensions:

  1. **Completeness** — all 12 sections present and substantive (not placeholders)
  2. **Risk Quality** — risks in Section 11 are specific (named systems, named scenarios) and each has a mitigation
  3. **Assumption Exposure** — decisions that only work if an unstated condition is true
  4. **Standards Compliance** — alignment with approved tech stack, API style, deployment, and security standards

  ## Output

  Return the `REVIEW_FINDINGS` block exactly as specified in the `arch-review` skill's orchestrated mode format:

  ```
  ## REVIEW_FINDINGS
  pass: [pass_number]
  critical_count: [N]
  important_count: [N]
  minor_count: [N]
  clean: [true / false]

  ### FINDINGS

  #### [F001] 🔴 [Short title]
  - dimension: [Completeness / Risk Quality / Assumption Exposure / Standards Compliance]
  - severity: critical
  - location: [section name]
  - finding: [specific description of what is wrong or missing]
  - fix: [exact, actionable instruction — specific enough to apply without interpretation]

  #### [F002] 🟡 ...
  #### [F003] 🟢 ...
  ```

  If clean, return `clean: true` with only minor findings (or "None").

  Do not produce a file. Do not call `present_files`. Return only the `REVIEW_FINDINGS` block.
  ```

- [ ] **Step 2: Verify frontmatter**

  ```bash
  grep -n "^name:\|^description:\|^tools:\|^model:" "architecture-design/.claude-plugin/agents/arch-reviewer.md"
  ```
  Expected: all four fields present.

- [ ] **Step 3: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/agents/arch-reviewer.md"
  git commit -m "feat: add arch-reviewer agent — cold peer review in orchestrated mode"
  ```

---

## Task 9: Create arch-template-formatter skill and default template

**Files:**
- Create: `architecture-design/.claude-plugin/skills/arch-template-formatter/SKILL.md`
- Create: `architecture-design/.claude-plugin/skills/arch-template-formatter/references/solution-intent-template.md`

- [ ] **Step 1: Create the skill directory**

  ```bash
  mkdir -p "architecture-design/.claude-plugin/skills/arch-template-formatter/references"
  ```

- [ ] **Step 2: Write the SKILL.md**

  Create `architecture-design/.claude-plugin/skills/arch-template-formatter/SKILL.md`:
  ```markdown
  ---
  name: arch-template-formatter
  description: Reformats a completed Solution Intent document to match a provided template structure. Triggered by arch-orchestrator in Phase 9 when a template is available (runtime --template path or default references/solution-intent-template.md). Content is preserved — only structure, headings, and formatting change.
  ---

  # arch-template-formatter Skill

  ## Overview

  Takes a completed Solution Intent document and a target template, then maps the Solution Intent's content into the template's structure. No content is dropped — if a Solution Intent section has no matching template section, append it as an appendix at the end.

  ## Input
  - `document`: the completed Solution Intent markdown
  - `template`: the target template markdown content

  ## Process

  ### Step 1: Parse the template structure

  Identify all sections and heading levels in the template. Note any placeholder patterns:
  - `{{section_name}}` — direct injection point
  - `[placeholder text]` in square brackets — replace with matching content
  - `<!-- comment -->` — HTML comments describing what belongs there

  ### Step 2: Map Solution Intent sections to template sections

  Use semantic matching — a template heading "Architecture Overview" maps to Solution Intent Section 3 (High-Level Architecture). Preserve the ASCII diagram, all tables, and all code blocks exactly. Do not paraphrase content — use the original text.

  For any Solution Intent content with no matching template section, add it to an `## Additional Information` appendix at the end of the document.

  ### Step 3: Produce the formatted document

  Write the final document using the template's structure with the Solution Intent's content. Maintain all markdown formatting, tables, and diagrams from the original.

  ## Output

  Return the complete reformatted document as markdown.

  ## Reference Files

  - `references/solution-intent-template.md` — default company Solution Intent template.
    **TODO:** Replace with your organisation's standard Solution Intent format. The current file contains a generic starting template.
  ```

- [ ] **Step 3: Write the default template**

  Create `architecture-design/.claude-plugin/skills/arch-template-formatter/references/solution-intent-template.md`:
  ```markdown
  # Solution Intent: [Initiative Name]

  > **Version:** 1.0 | **Status:** Draft
  > Replace all `[placeholder]` text with actual content before sharing.

  ---

  ## Summary

  | Field | Value |
  |-------|-------|
  | Initiative | [One-sentence description] |
  | Architectural approach | [Pattern name] |
  | Primary author | [Name] |
  | Date | [YYYY-MM-DD] |
  | Status | Draft / In Review / Approved |

  ---

  ## Problem & Goals

  ### Business Problem
  [Describe the problem being solved and why it matters now]

  ### Goals
  - [Goal 1]
  - [Goal 2]

  ### Non-Goals
  - [Item explicitly out of scope]

  ### Constraints
  - [Constraint 1]
  - [Constraint 2]

  ---

  ## Architecture

  ### Chosen Approach
  [Architectural pattern and rationale — 2–3 sentences]

  ### System Diagram
  ```
  [ASCII diagram — paste directly]
  ```

  ---

  ## Services

  For each service repeat this block:

  ### [Service Name]
  - **Bounded context:** [context]
  - **Responsibilities:** [what it does]
  - **Data owned:** [key entities]
  - **Interfaces:** [APIs and events]

  ---

  ## API & Event Design
  [Paste Section 5 from Solution Intent]

  ---

  ## Data Architecture
  [Paste Section 6 from Solution Intent]

  ---

  ## Infrastructure
  [Paste Section 7 from Solution Intent]

  ---

  ## Security
  [Paste Section 8 from Solution Intent]

  ---

  ## Production Readiness
  [Paste Section 9 from Solution Intent]

  ---

  ## Technology Decisions

  | Layer | Technology | Version | Tier | Notes |
  |-------|------------|---------|------|-------|
  | | | | | |

  ---

  ## Risks & Open Questions

  | # | Item | Type | Mitigation / Owner |
  |---|------|------|-------------------|
  | 1 | [Risk or question] | Risk / Question | [Mitigation or owner] |

  ---

  ## Review Findings

  | ID | Severity | Finding | Fix |
  |----|----------|---------|-----|
  | | | | |
  ```

- [ ] **Step 4: Verify SKILL.md frontmatter**

  ```bash
  grep -n "^name:\|^description:" "architecture-design/.claude-plugin/skills/arch-template-formatter/SKILL.md"
  ```
  Expected: both fields present.

- [ ] **Step 5: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-template-formatter/"
  git commit -m "feat: add arch-template-formatter skill and default Solution Intent template"
  ```

---

## Task 10: Final validation

- [ ] **Step 1: Verify all required files exist**

  ```bash
  for f in \
    "architecture-design/.claude-plugin/plugin.json" \
    "architecture-design/.claude-plugin/agents/arch-orchestrator.md" \
    "architecture-design/.claude-plugin/agents/arch-explorer.md" \
    "architecture-design/.claude-plugin/agents/arch-strategist.md" \
    "architecture-design/.claude-plugin/agents/arch-agent-ddd.md" \
    "architecture-design/.claude-plugin/agents/arch-agent-api-event.md" \
    "architecture-design/.claude-plugin/agents/arch-agent-data.md" \
    "architecture-design/.claude-plugin/agents/arch-agent-deployment.md" \
    "architecture-design/.claude-plugin/agents/arch-agent-security.md" \
    "architecture-design/.claude-plugin/agents/arch-agent-ops.md" \
    "architecture-design/.claude-plugin/agents/arch-synthesizer.md" \
    "architecture-design/.claude-plugin/agents/arch-reviewer.md" \
    "architecture-design/.claude-plugin/skills/arch-template-formatter/SKILL.md" \
    "architecture-design/.claude-plugin/skills/arch-template-formatter/references/solution-intent-template.md"
  do
    [ -f "$f" ] && echo "OK: $f" || echo "MISSING: $f"
  done
  ```
  Expected: all lines print `OK:`.

- [ ] **Step 2: Verify plugin.json is valid JSON**

  ```bash
  python3 -c "import json; json.load(open('architecture-design/.claude-plugin/plugin.json')); print('plugin.json: valid')"
  ```
  Expected: `plugin.json: valid`

- [ ] **Step 3: Verify all agent files have required frontmatter**

  ```bash
  for f in architecture-design/.claude-plugin/agents/*.md; do
    name=$(grep "^name:" "$f" | head -1)
    desc=$(grep "^description:" "$f" | head -1)
    tools=$(grep "^tools:" "$f" | head -1)
    model=$(grep "^model:" "$f" | head -1)
    [ -n "$name" ] && [ -n "$desc" ] && [ -n "$tools" ] && [ -n "$model" ] \
      && echo "OK: $f" || echo "INCOMPLETE FRONTMATTER: $f"
  done
  ```
  Expected: all lines print `OK:`.

- [ ] **Step 4: Verify command file has required frontmatter**

  ```bash
  grep -n "^description:\|^argument-hint:\|\$ARGUMENTS" "architecture-design/.claude-plugin/commands/create-architecture.md" 2>/dev/null || \
  grep -n "^description:\|^argument-hint:\|\$ARGUMENTS" "architecture-design/.claude-plugin/command/create-architecture.md"
  ```
  Expected: all three present.

- [ ] **Step 5: Verify skill agent names in orchestrator match actual agent filenames**

  ```bash
  echo "=== Agents directory ===" && ls architecture-design/.claude-plugin/agents/
  echo "=== Agents referenced in orchestrator ===" && grep -o "arch-[a-z-]*" "architecture-design/.claude-plugin/agents/arch-orchestrator.md" | sort -u
  ```
  Verify every agent name referenced in the orchestrator matches a file in the agents directory.

- [ ] **Step 6: Final commit**

  ```bash
  git add -A
  git status
  ```
  Confirm only expected files are staged, then:
  ```bash
  git commit -m "feat: complete create-architect command with 10-phase workflow and 11 agents"
  ```
