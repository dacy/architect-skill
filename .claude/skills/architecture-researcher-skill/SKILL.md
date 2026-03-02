---
name: architecture-researcher-skill
description: Use this skill to research existing enterprise capabilities before or during architecture design. Trigger on "what APIs exist for X", "is there already a service for Y", "who owns Z", "research what's available", "find the application owner", or any research into existing systems or capabilities. Also auto-triggered by the architect skill at the start of any Solution Intent. Searches Confluence, API Marketplace, Application Registry, and web. Outputs a structured Research Brief in Markdown.
---

# Architecture Research Agent

## Purpose
Research existing enterprise capabilities, systems, owners, and documentation before
committing to an architecture design. The core principle: **don't build what already
exists, and don't ignore what's coming**.

This agent surfaces:
- **Existing APIs** that could satisfy requirements (avoid redundant builds)
- **Registered applications**, their owners, and their roadmaps
- **Confluence documentation** on existing capabilities and future plans
- **Key contacts** to reach out to (Design Authority, App Owners, Architects)
- **Gaps** where nothing adequate exists yet

---

## Source Configuration

**Always check `references/sources.md` before running any research.**

This file defines how to connect to your organization's internal sources.
If it doesn't exist yet, copy `references/sources-template.md` to `sources.md` and fill it in.

```
references/
├── sources.md           ← Your configured source endpoints/URLs (create from template)
├── sources-template.md  ← How to configure each source type
└── contacts.md          ← Known Design Authority / Architecture contacts (optional)
```

### Sources Searched

| Source | What it provides | Access pattern |
|--------|-----------------|----------------|
| **Confluence** | Docs, existing designs, roadmaps, team pages | REST API or web browse |
| **API Marketplace** | Internal API catalog — name, owner, endpoint, docs | REST API or web browse |
| **Application Registry / CMDB** | Registered apps, owners, tech stack, status | REST API or web browse |
| **Web search** | Public vendor docs, open-source options, industry patterns | web_search tool |

If a source is not yet configured in `references/sources.md`, note it as "not configured"
in the Research Brief and proceed with what is available.

---

## Research Depth

This agent performs **deep-dive research** by default:

| Layer | What it finds |
|-------|--------------|
| **Discovery** | Name, description, owner, link |
| **Capability** | What the API/app actually does, version, SLA |
| **Documentation** | Confluence pages, API specs, design docs |
| **Roadmap** | Future plans, upcoming versions, known deprecations |
| **Gaps** | What's missing, what's partial, what's planned but not ready |
| **Contacts** | App Owner, Design Authority, Architect responsible |

---

## Research Process

### Step 1 — Understand the requirement

Extract from the user's description (or from the Solution Intent context brief):
- What **capability** is needed? (e.g. "customer identity lookup", "trade data enrichment")
- What **domain** does it touch? (e.g. payments, accounts, risk, identity)
- What **integration type** is expected? (API call, batch, read-only query, etc.)
- Any **known system names** already mentioned?

If invoked standalone, ask the user for this context before searching.
If invoked by the architect orchestrator, extract it from the context brief.

### Step 2 — Search all configured sources

#### Confluence
- Search for pages matching capability/domain keywords
- Look for: existing architecture docs, design decisions, team pages, roadmap pages
- Capture: page title, author, last-updated date, URL, relevance summary
- Flag pages that mention future plans or announced deprecations

#### API Marketplace
- Search by capability keywords and domain
- For each match capture: name, version, owner team, description, endpoint base URL, spec link
- Note: Is it production-ready? Deprecated? Is a newer version coming?

#### Application Registry / CMDB
- Search for registered applications in the relevant domain
- For each match capture: app name, owner, tech stack, status (active / decommissioning / planned), dependencies
- Flag any apps being sunset that the solution should avoid coupling to

#### Web Search
- Search for public documentation on vendor products or open-source options
- Use `web_search` tool with targeted queries
- Focus on: official docs, known limitations, integration patterns, recent changes

### Step 3 — Synthesize findings

Organize everything into four buckets:

1. **✅ Available now** — existing capabilities that meet the requirement today
2. **⚡ Available soon** — capabilities on a published roadmap (note expected timeframe)
3. **⚠️ Partial fit** — capabilities that partially meet the need; gaps identified
4. **❌ Gap** — nothing adequate exists; new build or procurement likely needed

### Step 4 — Identify contacts

For each relevant system found, identify:
- **Application Owner** — who to contact for integration questions
- **Design Authority / Architect** — who governs the domain
- **Product Manager** — who owns the roadmap (if roadmap discussion needed)

Check `references/contacts.md` first. If not listed there, note what was found in
the registry or documentation.

### Step 5 — Draft outreach messages

For each key contact identified, draft a ready-to-send email.
Use the **Outreach Templates** section below, tailored to the specific system and questions.

### Step 6 — Produce and present the Research Brief

Output the Research Brief as a `.md` file and present it using `present_files`.
File name: `[initiative-name]-research-brief.md`

---

## Research Brief Template

```markdown
# Architecture Research Brief
**Requirement:** [what was researched]
**Date:** [date]
**Status:** Draft — pending stakeholder confirmation

---

## Summary
[3–5 sentences: what was found, what's ready to use, what the key gap is,
and who the critical contact is]

---

## Available Capabilities

### ✅ Available Now
| Name | Type | Owner | Description | Link | Fit |
|------|------|-------|-------------|------|-----|
| [name] | API / App | [team] | [what it does] | [link] | Full / Partial |

### ⚡ Available Soon (on Roadmap)
| Name | Type | Expected | Owner | Notes |
|------|------|----------|-------|-------|
| [name] | [type] | [Q / date] | [team] | [what's changing] |

### ⚠️ Partial Fit — Gaps Identified
| Name | What it covers | What's missing | Workaround? |
|------|---------------|----------------|-------------|
| [name] | [capability] | [gap] | [yes/no + notes] |

### ❌ Gaps — Nothing Adequate Found
| Capability Needed | Closest Option | Gap | Recommendation |
|------------------|---------------|-----|----------------|
| [capability] | [closest thing] | [gap] | Build / Procure / Wait for roadmap |

---

## Relevant Documentation

| Title | Source | URL | Relevance | Last Updated |
|-------|--------|-----|-----------|--------------|
| [title] | Confluence / Web | [link] | [why relevant] | [date] |

---

## Key Contacts

| Name | Role | Domain / System | Contact | What to Ask |
|------|------|----------------|---------|-------------|
| [name] | App Owner / Design Authority / Architect | [domain] | [email / Slack] | [specific question] |

---

## Drafted Outreach

[One ready-to-send email per key contact — see below]

---

## Research Gaps

**Sources not searched (not yet configured):**
- [list skipped sources with reason]

**Open questions requiring human follow-up:**
- [things the research couldn't determine]

---

## Recommendations

**Reuse:** [Existing capabilities to leverage — with justification]
**Design toward:** [Upcoming roadmap items worth waiting for — with timing risk]
**Build new:** [What genuinely needs to be new — confirmed gaps only]
**Avoid:** [Systems being decommissioned or deprecated]
```

---

## Outreach Templates

Draft one email per key contact, tailored to the specific system and questions found.

### Application Owner — Integration Question
```
Subject: Integration Question — [Initiative Name] re: [Their System]

Hi [Name],

I'm leading the architecture for [initiative] and while researching existing
capabilities I came across [their system/API]. It looks like it may be relevant
to what we're building.

I'd like to understand:
1. [Specific capability question]
2. [Specific roadmap / future plans question]
3. [Any known integration constraints or onboarding process]

Would you have 30 minutes for a quick call, or happy to connect async if easier?

Thanks,
[Your name]
```

### Design Authority — Pattern Alignment
```
Subject: Architecture Review Request — [Initiative Name]

Hi [Name],

I'm designing [brief description] and want to make sure we're aligned with current
enterprise patterns before finalizing the approach.

Key decisions I'd like your input on:
1. [Decision 1]
2. [Decision 2]

I can share a draft Quick Read or Solution Intent for review if helpful.
Would you be available for a brief review, or should I submit through [review process]?

Thanks,
[Your name]
```

### Product Manager — Roadmap Alignment
```
Subject: Roadmap Question — [Capability Area]

Hi [Name],

I'm working on [initiative] and found that [their product] has [capability] on the
roadmap. We're deciding whether to build this ourselves or design toward your
upcoming release.

Could you share:
1. Confirmed timeline for [capability]?
2. Whether early access or beta is available?
3. Whether [specific requirement] is in scope?

This would help us avoid duplication and align our design with your direction.

Thanks,
[Your name]
```

---

## Integration with Architect Orchestrator

When auto-triggered at the start of a Solution Intent, the agent receives:

```
Initiative: [name]
Problem: [summary]
Proposed approach: [direction]
Capability keywords: [comma-separated search terms]
Domains: [comma-separated domains]
Output mode: orchestrated
```

The agent runs full research and returns the Research Brief Markdown to the architect skill,
which uses findings to inform Current State, Dependencies, Risks, and Open Questions sections.
The Research Brief is also saved as a standalone file for the user.

---

## Quality Checklist

Before presenting the brief:
- [ ] All configured sources searched (skipped sources noted with reason)
- [ ] Each finding has an owner/contact identified
- [ ] Roadmap items include expected timeframe
- [ ] Gaps section is honest — no forced fits
- [ ] One ready-to-send outreach email drafted per key contact
- [ ] Recommendations split cleanly: Reuse / Design toward / Build / Avoid
- [ ] Open questions explicitly listed
- [ ] File saved as `[initiative-name]-research-brief.md` and presented
