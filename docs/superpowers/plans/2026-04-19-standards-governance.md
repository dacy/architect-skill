# Standards Governance Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Wire organizational standards into the architect workflow by creating missing reference file stubs for `arch-api-event`, standardizing deviation handling across all specialist skills, and migrating existing Confluence standards into the plugin's reference files.

**Architecture:** All specialist skills already have `references/` folders and SKILL.md logic to load them. The infrastructure is complete. This plan adds the two missing `arch-api-event` reference files, ensures every skill emits consistent ⚠️ deviation flags when a design choice conflicts with a loaded standard, and migrates Confluence content into the five existing placeholder stubs. Standard owners then own their reference file directly with no plugin involvement needed for future updates.

**Tech Stack:** Claude Code plugin system (markdown instruction files), existing specialist skills (`arch-api-event`, `arch-data`, `arch-deployment`, `arch-security`, `arch-production-operations`, `arch-tech-stack`).

---

## File Map

**Create:**
- `architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md`
- `architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md`

**Modify (deviation handling):**
- `architecture-design/.claude-plugin/skills/arch-data/SKILL.md`
- `architecture-design/.claude-plugin/skills/arch-deployment/SKILL.md`
- `architecture-design/.claude-plugin/skills/arch-production-operations/SKILL.md`

**Modify (content migration — replace placeholders with real standards):**
- `architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md`
- `architecture-design/.claude-plugin/skills/arch-deployment/references/deployment-standards.md`
- `architecture-design/.claude-plugin/skills/arch-security/references/security-standards.md`
- `architecture-design/.claude-plugin/skills/arch-production-operations/references/operations-standards.md`
- `architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md`

---

## Task 1: Create arch-api-event reference file stubs

**Files:**
- Create: `architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md`
- Create: `architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md`

- [ ] **Step 1: Create api-guidelines.md stub**

  Create `architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md` with this exact content:

  ```markdown
  <!-- Owner: API team | Last updated: YYYY-MM-DD -->
  <!-- Authoritative standard for API design and service-to-service communication. Edit directly; no cross-team PR approval needed. -->

  # API & Service Communication Standards

  > Replace this file with your organization's actual API standards (migrated from Confluence).
  > Sections below show the expected structure — populate each from your Confluence page.

  ---

  ## API Style

  | Style | Approved For | Notes |
  |-------|-------------|-------|
  | REST / JSON | [e.g., all external and internal synchronous APIs] | [versioning policy] |
  | gRPC | [e.g., high-throughput internal only] | [when approved] |
  | GraphQL | [e.g., exception required] | [approval conditions] |

  ---

  ## Versioning Policy

  [Document your versioning strategy: URI versioning, header versioning, deprecation timeline, breaking change policy]

  ---

  ## Service-to-Service Communication

  | Pattern | Approved For | Notes |
  |---------|-------------|-------|
  | Synchronous REST | [use cases] | [timeout standards, retry policy] |
  | gRPC | [use cases] | [constraints] |
  | Async event (Kafka) | [use cases] | [event naming conventions] |
  | Direct DB access | [e.g., not approved] | [exception conditions] |

  ---

  ## Internal vs External API Rules

  [Document what distinguishes an internal API from an external one, and whether different standards apply to each]

  ---

  ## Exception Process

  [How to request approval for a pattern not listed above]
  ```

- [ ] **Step 2: Create event-guidelines.md stub**

  Create `architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md` with this exact content:

  ```markdown
  <!-- Owner: Event/messaging team | Last updated: YYYY-MM-DD -->
  <!-- Authoritative standard for event and messaging design. Edit directly; no cross-team PR approval needed. -->

  # Event & Messaging Standards

  > Replace this file with your organization's actual event/messaging standards (migrated from Confluence).

  ---

  ## Approved Messaging Platforms

  | Platform | Approved For | Notes |
  |----------|-------------|-------|
  | [e.g., Kafka / MSK] | [use cases] | [topic naming convention, partition strategy] |
  | [e.g., SQS] | [use cases] | [queue naming convention] |
  | [e.g., SNS] | [use cases] | [fan-out patterns] |

  ---

  ## Topic / Queue Naming Convention

  [Document the naming pattern: e.g., `[domain].[entity].[event-type]` → `orders.order.placed`]

  ---

  ## Event Schema Standards

  [Document schema format (Avro, JSON Schema, Protobuf), schema registry requirements, versioning approach]

  ---

  ## Event Ordering & Delivery Guarantees

  [Document at-least-once vs exactly-once expectations, ordering guarantees, idempotency requirements]

  ---

  ## Exception Process

  [How to request use of a messaging platform not listed above]
  ```

- [ ] **Step 3: Verify both files exist**

  ```bash
  ls "architecture-design/.claude-plugin/skills/arch-api-event/references/"
  ```
  Expected: `README.md`, `api-guidelines.md`, `event-guidelines.md`

- [ ] **Step 4: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md" \
          "architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md"
  git commit -m "feat: add arch-api-event reference file stubs for API and event standards"
  ```

---

## Task 2: Standardize deviation handling in arch-data, arch-deployment, and arch-production-operations SKILL.md files

`arch-api-event` and `arch-tech-stack` already have explicit deviation/exception flagging. The three skills below need a consistent **Standards Deviation** section added so that all skills emit ⚠️ markers in the same format.

**Files:**
- Modify: `architecture-design/.claude-plugin/skills/arch-data/SKILL.md`
- Modify: `architecture-design/.claude-plugin/skills/arch-deployment/SKILL.md`
- Modify: `architecture-design/.claude-plugin/skills/arch-production-operations/SKILL.md`

- [ ] **Step 1: Add Standards Deviation section to arch-data/SKILL.md**

  Append the following section at the end of `architecture-design/.claude-plugin/skills/arch-data/SKILL.md` (before any closing content):

  ```markdown
  ---

  ## Standards Deviation Handling

  When any data architecture decision conflicts with a rule in `references/data-standards.md`, do not silently override the standard. Flag the deviation inline using this exact format:

  ```
  ⚠️ **Standard deviation:** [recommended choice] conflicts with [specific rule from data-standards.md].
  Justification: [technical or business reason this deviation is necessary].
  Exception process: [as documented in references/data-standards.md, or "consult the Data team"].
  ```

  Every deviation must appear in the output so the architect and reviewer can act on it.
  ```

- [ ] **Step 2: Verify arch-data deviation section**

  ```bash
  grep -n "Standard deviation\|Standards Deviation" "architecture-design/.claude-plugin/skills/arch-data/SKILL.md"
  ```
  Expected: at least one match.

- [ ] **Step 3: Add Standards Deviation section to arch-deployment/SKILL.md**

  Append the following section at the end of `architecture-design/.claude-plugin/skills/arch-deployment/SKILL.md`:

  ```markdown
  ---

  ## Standards Deviation Handling

  When any deployment or infrastructure decision conflicts with a rule in `references/deployment-standards.md`, flag the deviation inline:

  ```
  ⚠️ **Standard deviation:** [recommended choice] conflicts with [specific rule from deployment-standards.md].
  Justification: [technical or business reason].
  Exception process: [as documented in references/deployment-standards.md, or "consult the Platform team"].
  ```

  Every deviation must appear in the output so the architect and reviewer can act on it.
  ```

- [ ] **Step 4: Verify arch-deployment deviation section**

  ```bash
  grep -n "Standard deviation\|Standards Deviation" "architecture-design/.claude-plugin/skills/arch-deployment/SKILL.md"
  ```
  Expected: at least one match.

- [ ] **Step 5: Add Standards Deviation section to arch-production-operations/SKILL.md**

  Append the following section at the end of `architecture-design/.claude-plugin/skills/arch-production-operations/SKILL.md`:

  ```markdown
  ---

  ## Standards Deviation Handling

  When any operations or observability decision conflicts with a rule in `references/operations-standards.md`, flag the deviation inline:

  ```
  ⚠️ **Standard deviation:** [recommended choice] conflicts with [specific rule from operations-standards.md].
  Justification: [technical or business reason].
  Exception process: [as documented in references/operations-standards.md, or "consult the Ops team"].
  ```

  Every deviation must appear in the output so the architect and reviewer can act on it.
  ```

- [ ] **Step 6: Verify all three files have deviation handling**

  ```bash
  for f in arch-data arch-deployment arch-production-operations; do
    echo "=== $f ===" && grep -c "Standard deviation" "architecture-design/.claude-plugin/skills/$f/SKILL.md"
  done
  ```
  Expected: each prints `1` or higher.

- [ ] **Step 7: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-data/SKILL.md" \
          "architecture-design/.claude-plugin/skills/arch-deployment/SKILL.md" \
          "architecture-design/.claude-plugin/skills/arch-production-operations/SKILL.md"
  git commit -m "feat: standardize deviation handling pattern across specialist skills"
  ```

---

## Tasks 3–7: Migrate Confluence standards into reference files

**Important:** Tasks 3–7 require the owning team to provide the Confluence page URL or content for their domain. The steps below describe the migration process. The `arch-explorer` agent (available in this plugin) can fetch Confluence pages automatically if you have the URL — use the Atlassian MCP tools with the page URL or CQL search.

Each task follows the same pattern:
1. Fetch content from Confluence (via arch-explorer or manual copy)
2. Replace all `> **TODO:**` blocks and `[placeholder]` values with real standards
3. Add the owner comment header
4. Verify no TODO markers remain
5. Commit

---

## Task 3: Migrate data standards into arch-data/references/data-standards.md

**Files:**
- Modify: `architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md`

- [ ] **Step 1: Fetch data standards from Confluence**

  Obtain the Confluence page URL for the data team's standards page. Then either:
  - Use `arch-explorer` agent with the Confluence URL to fetch and summarise the content, or
  - Copy the content directly from Confluence into your editor

- [ ] **Step 2: Replace placeholder content**

  Open `architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md`. Replace:
  - The `> **TODO:**` header block at the top with the owner comment header:
    ```markdown
    <!-- Owner: Data team | Last updated: YYYY-MM-DD -->
    <!-- Authoritative standard for data architecture. Edit directly; no cross-team PR approval needed. -->
    ```
  - Each `> **TODO:** ...` block with the actual standard for that section from Confluence
  - Each `[placeholder]` value in tables with real approved values, constraints, and notes

- [ ] **Step 3: Verify no TODO markers remain**

  ```bash
  grep -n "TODO" "architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md"
  ```
  Expected: 0 matches.

- [ ] **Step 4: Verify owner header is present**

  ```bash
  grep -n "Owner:" "architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md"
  ```
  Expected: 1 match on line 1.

- [ ] **Step 5: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md"
  git commit -m "docs: migrate data standards from Confluence into arch-data references"
  ```

---

## Task 4: Migrate deployment standards into arch-deployment/references/deployment-standards.md

**Files:**
- Modify: `architecture-design/.claude-plugin/skills/arch-deployment/references/deployment-standards.md`

- [ ] **Step 1: Fetch deployment standards from Confluence**

  Obtain the Confluence URL for the platform team's deployment standards page. Fetch via arch-explorer or copy manually.

- [ ] **Step 2: Replace placeholder content**

  Open `architecture-design/.claude-plugin/skills/arch-deployment/references/deployment-standards.md`. Replace all `> **TODO:**` blocks and `[placeholder]` table values with real content. Add owner header:
  ```markdown
  <!-- Owner: Platform team | Last updated: YYYY-MM-DD -->
  <!-- Authoritative standard for deployment and infrastructure. Edit directly; no cross-team PR approval needed. -->
  ```

- [ ] **Step 3: Verify no TODO markers remain**

  ```bash
  grep -n "TODO" "architecture-design/.claude-plugin/skills/arch-deployment/references/deployment-standards.md"
  ```
  Expected: 0 matches.

- [ ] **Step 4: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-deployment/references/deployment-standards.md"
  git commit -m "docs: migrate deployment standards from Confluence into arch-deployment references"
  ```

---

## Task 5: Migrate security standards into arch-security/references/security-standards.md

**Files:**
- Modify: `architecture-design/.claude-plugin/skills/arch-security/references/security-standards.md`

- [ ] **Step 1: Fetch security standards from Confluence**

  Obtain the Confluence URL for the security team's standards page. Fetch via arch-explorer or copy manually.

- [ ] **Step 2: Replace placeholder content**

  Open `architecture-design/.claude-plugin/skills/arch-security/references/security-standards.md`. Replace all `> **TODO:**` blocks and `[placeholder]` values with real content. Add owner header:
  ```markdown
  <!-- Owner: Security team | Last updated: YYYY-MM-DD -->
  <!-- Authoritative standard for security architecture. Edit directly; no cross-team PR approval needed. -->
  ```

- [ ] **Step 3: Verify no TODO markers remain**

  ```bash
  grep -n "TODO" "architecture-design/.claude-plugin/skills/arch-security/references/security-standards.md"
  ```
  Expected: 0 matches.

- [ ] **Step 4: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-security/references/security-standards.md"
  git commit -m "docs: migrate security standards from Confluence into arch-security references"
  ```

---

## Task 6: Migrate operations standards into arch-production-operations/references/operations-standards.md

**Files:**
- Modify: `architecture-design/.claude-plugin/skills/arch-production-operations/references/operations-standards.md`

- [ ] **Step 1: Fetch operations standards from Confluence**

  Obtain the Confluence URL for the ops team's standards page. Fetch via arch-explorer or copy manually.

- [ ] **Step 2: Replace placeholder content**

  Open `architecture-design/.claude-plugin/skills/arch-production-operations/references/operations-standards.md`. Replace all `> **TODO:**` blocks and `[placeholder]` values with real content. Add owner header:
  ```markdown
  <!-- Owner: Ops team | Last updated: YYYY-MM-DD -->
  <!-- Authoritative standard for production operations. Edit directly; no cross-team PR approval needed. -->
  ```

- [ ] **Step 3: Verify no TODO markers remain**

  ```bash
  grep -n "TODO" "architecture-design/.claude-plugin/skills/arch-production-operations/references/operations-standards.md"
  ```
  Expected: 0 matches.

- [ ] **Step 4: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-production-operations/references/operations-standards.md"
  git commit -m "docs: migrate operations standards from Confluence into arch-production-operations references"
  ```

---

## Task 7: Migrate approved product catalog into arch-tech-stack/references/tech-stack.md

**Files:**
- Modify: `architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md`

- [ ] **Step 1: Fetch approved product catalog from Confluence**

  Obtain the Confluence URL for the governance/platform team's approved technology catalog. Fetch via arch-explorer or copy manually.

- [ ] **Step 2: Replace placeholder content**

  Open `architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md`. Replace:
  - The `> **TODO:**` header with the owner comment:
    ```markdown
    <!-- Owner: Governance/platform team | Last updated: YYYY-MM-DD -->
    <!-- Authoritative approved product catalog. Edit directly; no cross-team PR approval needed. -->
    ```
  - All `[Language 1]`, `[Framework 1]`, `[e.g., ...]` placeholders in every table section with real approved technologies, versions, tiers, and constraints
  - The Exception Process `> **TODO:**` block with the real exception process steps

- [ ] **Step 3: Verify no TODO markers remain**

  ```bash
  grep -n "TODO" "architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md"
  ```
  Expected: 0 matches.

- [ ] **Step 4: Verify approval tiers are populated**

  ```bash
  grep -c "Approved\|Conditional\|Exception" "architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md"
  ```
  Expected: more than 5 matches (one per technology row).

- [ ] **Step 5: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md"
  git commit -m "docs: migrate approved product catalog from Confluence into arch-tech-stack references"
  ```

---

## Task 8: Migrate API and event standards into arch-api-event reference files

**Files:**
- Modify: `architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md`
- Modify: `architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md`

- [ ] **Step 1: Fetch API and event standards from Confluence**

  Obtain the Confluence URLs for the API team's and event/messaging team's standards pages. Fetch each via arch-explorer or copy manually.

- [ ] **Step 2: Replace api-guidelines.md placeholder content**

  Open `architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md`. Replace all `[e.g., ...]` and `[Document ...]` placeholders with real standards content from Confluence. Update the `YYYY-MM-DD` in the owner header to today's date.

- [ ] **Step 3: Replace event-guidelines.md placeholder content**

  Open `architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md`. Replace all `[e.g., ...]` and `[Document ...]` placeholders with real standards content from Confluence. Update the date.

- [ ] **Step 4: Verify no placeholder markers remain in either file**

  ```bash
  grep -n "\[e\.g\.\|Document \.\.\." \
    "architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md" \
    "architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md"
  ```
  Expected: 0 matches.

- [ ] **Step 5: Commit**

  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md" \
          "architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md"
  git commit -m "docs: migrate API and event standards from Confluence into arch-api-event references"
  ```

---

## Task 9: Final validation

- [ ] **Step 1: Verify all reference files have owner headers**

  ```bash
  for f in \
    "architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md" \
    "architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md" \
    "architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md" \
    "architecture-design/.claude-plugin/skills/arch-deployment/references/deployment-standards.md" \
    "architecture-design/.claude-plugin/skills/arch-security/references/security-standards.md" \
    "architecture-design/.claude-plugin/skills/arch-production-operations/references/operations-standards.md" \
    "architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md"; do
    header=$(head -1 "$f")
    echo "$f: $header"
  done
  ```
  Expected: every line starts with `<!-- Owner:`.

- [ ] **Step 2: Verify all three SKILL.md files have deviation handling**

  ```bash
  for f in arch-data arch-deployment arch-production-operations; do
    count=$(grep -c "Standard deviation" "architecture-design/.claude-plugin/skills/$f/SKILL.md")
    echo "$f: $count"
  done
  ```
  Expected: each prints `1` or higher.

- [ ] **Step 3: Verify arch-tech-stack SKILL.md emits ⚠️ flags for exceptions**

  ```bash
  grep -n "⚠️\|Exception\|exception" "architecture-design/.claude-plugin/skills/arch-tech-stack/SKILL.md"
  ```
  Expected: at least one line containing `⚠️` and at least one line referencing the exception process. If `⚠️` is absent, append this section to the file:

  ```markdown
  ---

  ## Standards Deviation Handling

  When a required technology is not in the approved catalog or falls under Conditional or Exception Required tier:

  ```
  ⚠️ **Standard deviation:** [proposed technology] is [Conditional / Exception Required] per references/tech-stack.md.
  Justification: [why the approved alternative doesn't fit].
  Exception process: [as documented in references/tech-stack.md — Exception Process section].
  ```

  No exception-tier technology proceeds to implementation without architecture board approval.
  ```

  Then commit:
  ```bash
  git add "architecture-design/.claude-plugin/skills/arch-tech-stack/SKILL.md"
  git commit -m "feat: add explicit deviation handling to arch-tech-stack skill"
  ```

- [ ] **Step 4: Verify no reference files still contain TODO markers**

  ```bash
  grep -rn "TODO" architecture-design/.claude-plugin/skills/*/references/
  ```
  Expected: 0 matches (after all migration tasks are complete).
