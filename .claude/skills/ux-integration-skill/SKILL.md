---
name: ux-integration-skill
description: Use this skill when designing any user-facing component that integrates with the company's operational desktop. Trigger on MFE, microfrontend, operational desktop, front-end architecture, UI integration, UX design, or any solution that includes a user interface. Do not design front-end architecture without this skill — the company has specific MFE container patterns that must be followed.
---

# UX Integration Skill

## Overview

This skill designs user interface components that integrate with the company's operational
desktop — a UI/UX container platform that hosts Microfrontends (MFEs). Every UI feature
must conform to company MFE patterns.

Read `references/mfe-standards.md` for company-specific MFE patterns, registration, and
shell communication contracts before producing any UX design.

---

## Output

- **Markdown (.md)** — UX Integration section for Solution Intent, or standalone analysis
- Name file: `[initiative-name]-ux-integration.md`

---

## Core Concepts

### Operational Desktop

The operational desktop is the company's UI shell — a host application that:
- Manages authentication and session
- Provides the navigation frame and global chrome
- Launches and hosts MFEs within defined layout regions
- Provides shared services (notifications, user context, theming)

No standalone web applications are deployed outside this shell without an architecture exception.

### Microfrontend (MFE)

An MFE is a self-contained UI module deployed independently and loaded by the operational
desktop at runtime. Each MFE:
- Owns a specific business capability's UI
- Is registered with the shell via the MFE registry
- Communicates with the shell and peer MFEs through defined contracts only
- Is independently deployable and versioned

See `references/mfe-standards.md` for the company's specific MFE framework, module federation
configuration, and lifecycle contracts.

---

## Process

### Step 1: Identify UI capabilities

From the requirement, identify:
- What screens, workflows, or interactions are needed?
- Which business capability does each UI surface serve?
- Is this a new MFE or an extension to an existing one?

### Step 2: MFE boundary definition

Define the MFE boundary — what does this MFE own vs. what does it delegate to the shell?

| Concern | Owner |
|---------|-------|
| Authentication | Shell |
| Global navigation | Shell |
| Notifications | Shell (via shared service) |
| [Feature workflow] | This MFE |
| [Feature data fetching] | This MFE |

### Step 3: Shell integration points

Document how the MFE integrates with the shell:
- **Registration:** MFE name, route(s), entry point URL, permissions required
- **Shared context consumed:** user profile, tenant, theme, locale
- **Events published to shell:** navigation requests, notification triggers
- **Events received from shell:** session expiry, context changes

### Step 4: MFE-to-MFE communication

If this MFE needs to communicate with peer MFEs:
- Use the shell's event bus (not direct imports between MFEs)
- Document each cross-MFE event: name, payload schema, publisher, consumers

### Step 5: Data integration

How does the MFE retrieve and mutate data?
- REST APIs (point to `api-patterns-skill` output for contracts)
- Real-time feeds (WebSocket, SSE — document connection management)
- Shared state via shell services (read-only context only)

### Step 6: Deployment and versioning

- MFE is deployed as a static bundle to a CDN
- Shell loads MFE via module federation at runtime (see `references/mfe-standards.md`)
- Version strategy: how are breaking changes managed?

---

## Document Structure

```
# [Initiative Name] — UX Integration Design

## MFE Overview
[Name, capability owned, new vs. extension]

## Shell Integration
[Registration, shared context, shell events]

## MFE Boundary
[What this MFE owns vs. shell vs. APIs]

## Cross-MFE Communication
[Event bus contracts, if any]

## Data Integration
[API calls, real-time feeds]

## Deployment
[CDN, module federation, versioning]

## Open Questions
[Unknowns requiring shell team input]
```

---

## Reference Files

- `references/mfe-standards.md` — Company MFE framework, module federation config,
  shell event bus API, registration contract, and lifecycle hooks.
  **TODO:** Populate with your organization's operational desktop and MFE standards.
