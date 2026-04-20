# Standards Governance Design

## Goal

Enable organizational standards (API style, event patterns, service-to-service communication, data transfer, deployment, security, operations, approved products) to be maintained autonomously by the teams that own them, while being automatically applied by every architect workflow design session.

## Background

Standards are currently documented on Confluence, organized by concern rather than by team. Different domain teams own different concerns. The standards don't change frequently, but when they do, the owning team should be able to update them without coordinating with the plugin maintainer or touching agent files.

The architect plugin already has specialist skills that produce design sections (`arch-api-event`, `arch-data`, `arch-deployment`, `arch-security`, `arch-production-operations`, `arch-tech-stack`). These skills already have `references/` folders, and their SKILL.md files already contain logic to load those references before generating output. The loading infrastructure is complete.

**What is missing is the content** — the reference files are stubs or placeholders. The work is to populate them from Confluence and establish the ownership model so teams can maintain them going forward.

---

## What We're Building

### Standard ownership model

Each skill's `references/` folder is the authoritative source of truth for that concern. The owning team edits their file directly — no cross-team PR approval required, no plugin maintainer involvement for content changes.

**Full ownership mapping:**

| Skill | Reference file | Concern | Owned by |
|-------|---------------|---------|----------|
| `arch-api-event` | `references/api-guidelines.md` | Internal and external API style, versioning, contract patterns, service-to-service communication | API team |
| `arch-api-event` | `references/event-guidelines.md` | Event and messaging patterns, topic naming, schema standards | Event/messaging team (or API team if co-owned) |
| `arch-data` | `references/data-standards.md` *(exists)* | Data transfer between services, ownership rules, storage patterns | Data team |
| `arch-deployment` | `references/deployment-standards.md` *(exists)* | Infrastructure patterns, IaC tooling, CI/CD pipeline standards | Platform team |
| `arch-security` | `references/security-standards.md` *(exists)* | AuthN/AuthZ patterns, secrets management, network controls | Security team |
| `arch-production-operations` | `references/operations-standards.md` *(exists)* | Observability tooling, SLO baselines, on-call standards | Ops team |
| `arch-tech-stack` | `references/tech-stack.md` *(exists, structured template)* | Approved vendor and product catalog, approval tiers, exception process | Governance/platform team |

Note: `arch-api-event/references/` currently contains only a `README.md` describing what files are needed. The two guidelines files above must be created. All other skills already have their reference file in place as a placeholder.

---

### Reference file format

Reference files are plain markdown. No frontmatter, no required structure. Teams write them in whatever format suits them — prose, tables, bullet lists, decision records. The only convention is a header comment:

```markdown
<!-- Owner: [team name] | Last updated: YYYY-MM-DD -->
<!-- Authoritative standard for [concern]. Edit directly; no cross-team PR approval needed. -->

[standard content — any markdown format]
```

The skill reads the entire file and reasons over it. No machine-parsing is required.

---

### How existing SKILL.md files already apply references

The loading behaviour is already present in all skills. For example:

- `arch-api-event/SKILL.md` — "Always check `references/` before generating output. If standards files are present: load them and use as the authoritative enforcement source."
- `arch-security/SKILL.md` — "Read `references/security-standards.md` for company security requirements, approved encryption standards, PCI patterns, and API Gateway connectivity rules."

No SKILL.md changes are needed. The skills already handle both cases: reference file present (use it as authoritative) and absent (fall back to built-in defaults).

---

### Deviation handling convention

When a design choice conflicts with a loaded standard, skills should flag it inline:

```
⚠️ **Standard deviation:** [recommended choice] conflicts with [standard rule].
Justification: [technical or business reason].
Exception process: [as specified in the reference file, or "consult the owning team"].
```

Where this pattern is not already present in a skill, it should be added. A quick audit of the existing SKILL.md files during implementation will determine which (if any) need this addition.

---

### Confluence migration (one-time per domain)

1. Plugin maintainer creates the two missing `arch-api-event` reference file stubs.
2. For each concern, the owning team fetches the current Confluence page (using `arch-explorer` or manually) and pastes the content into their reference file, adapting it to markdown.
3. Team commits their reference file directly to the repo.
4. The Confluence page is updated with a note: "Source of truth moved to `[repo path]`."

This is a one-time task per standard domain. The plugin maintainer's only ongoing role is wiring new skills if new standard domains are ever added — standard content changes require no plugin maintainer involvement.

---

## Files to Create

| File | Note |
|------|------|
| `architecture-design/.claude-plugin/skills/arch-api-event/references/api-guidelines.md` | New — API and service-to-service communication standards stub |
| `architecture-design/.claude-plugin/skills/arch-api-event/references/event-guidelines.md` | New — event and messaging standards stub |

## Files to Modify (content only, not structure)

| File | Change |
|------|--------|
| `architecture-design/.claude-plugin/skills/arch-data/references/data-standards.md` | Populate placeholder with real standards from Confluence |
| `architecture-design/.claude-plugin/skills/arch-deployment/references/deployment-standards.md` | Populate placeholder with real standards from Confluence |
| `architecture-design/.claude-plugin/skills/arch-security/references/security-standards.md` | Populate placeholder with real standards from Confluence |
| `architecture-design/.claude-plugin/skills/arch-production-operations/references/operations-standards.md` | Populate placeholder with real standards from Confluence |
| `architecture-design/.claude-plugin/skills/arch-tech-stack/references/tech-stack.md` | Populate structured template with real approved products from Confluence |

## Files to Audit (may need deviation handling added)

| File | Check |
|------|-------|
| `architecture-design/.claude-plugin/skills/arch-data/SKILL.md` | Verify deviation flagging pattern is present |
| `architecture-design/.claude-plugin/skills/arch-deployment/SKILL.md` | Verify deviation flagging pattern is present |
| `architecture-design/.claude-plugin/skills/arch-production-operations/SKILL.md` | Verify deviation flagging pattern is present |

(`arch-api-event` and `arch-security` already have deviation handling documented. `arch-tech-stack` uses an approval-tier model — verify that it emits explicit ⚠️ flags when a chosen technology is Conditional or Exception Required.)

---

## Out of Scope

- SKILL.md structural changes — the loading mechanism already exists in all skills
- Automated Confluence sync — migration is a one-time manual step per domain
- Standards enforcement outside the architect workflow — applies only during `/create-architect` sessions
- Cross-skill standards — each skill owns its references independently
- Standards versioning UI — git history on reference files provides full audit trail
