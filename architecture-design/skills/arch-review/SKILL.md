---
name: arch-review
description: Use this skill to review an architecture document and return structured findings. Auto-triggered by the architect skill after every Quick Read and Solution Intent as an internal quality pass. Also trigger standalone on "review this doc", "critique this architecture", "check this before I share it", or "assess this solution intent". Reviews completeness, risk quality, assumption exposure, and standards compliance.
---

# Architecture Reviewer Sub-Agent

## Purpose
Perform a cold peer review of a completed Quick Read or Solution Intent and return
structured findings to the architect orchestrator. The architect uses these findings
to fix the document, then calls this reviewer again — up to 2 passes total.

"Cold" means: evaluate only what is explicitly present. Do not assume good intent
filled in what's missing.

**Two modes:**
- **Orchestrated** — returns a structured findings object (no file output); architect fixes and loops
- **Standalone** — produces an annotated Markdown document for the user directly

---

## Review Dimensions

### 1. Completeness
- Are all required sections present and substantive?
- **Quick Read required:** title block, problem statement, proposed approach,
  key capabilities, dependencies & risks, next steps
- **Solution Intent required:** problem statement & goals, proposed solution &
  diagrams, component breakdown, risks/assumptions/dependencies
- Flag: missing sections, placeholder text, sections with fewer than 2 meaningful points

### 2. Risk Quality
- Are risks specific (named system, named scenario) or generic?
- Does each risk have a mitigation, not just an acknowledgment?
- Are obvious risks absent? (rollback, data integrity, downstream dependency)
- Flag: generic risks, unmitigated high-impact risks, obvious missing risks

### 3. Assumption Exposure
- What is the document assuming without stating it?
- Common hidden assumptions: team availability, budget, downstream system readiness,
  data quality, regulatory clearance, non-prod environment parity
- Flag: any decision that only works if an unstated condition is true

### 4. Standards Compliance
- Does the solution align with enterprise patterns from sub-skill references?
- Check: API style, messaging platform, service decomposition, caching strategy
- Flag: deviations not called out, and called-out deviations with no mitigation

---

## Severity Levels

| Severity | Meaning |
|----------|---------|
| 🔴 Critical | Material error or missing required element — must fix |
| 🟡 Important | Weakens the document significantly — should fix |
| 🟢 Minor | Polish opportunity — fix if possible |

### Critical issue criteria
- A required section is missing entirely
- A high-impact risk has no mitigation and no acknowledgment
- The document contradicts itself
- A standards deviation with no justification

---

## Orchestrated Mode — Findings Object

When called by the architect orchestrator, return findings as a structured Markdown
block (not a file) in this exact format so the architect can parse and act on it:

```markdown
## REVIEW_FINDINGS
pass: [1 / 2 / 3]
critical_count: [N]
important_count: [N]
minor_count: [N]
clean: [true / false]

### FINDINGS

#### [F001] [severity emoji] [Short title]
- dimension: [Completeness / Risk Quality / Assumption Exposure / Standards Compliance]
- severity: [critical / important / minor]
- location: [section name where the issue lives]
- finding: [what is wrong or missing — be specific]
- fix: [exact instruction for what the architect should add, change, or remove]

#### [F002] ...
```

If `clean: true` (no critical or important findings), include:
```markdown
## REVIEW_FINDINGS
pass: [N]
critical_count: 0
important_count: 0
minor_count: [N]
clean: true
### FINDINGS
[list any minor findings, or "None" if fully clean]
```

---

## Standalone Mode — Annotated Document

When invoked directly by the user (not orchestrated), produce a single annotated
Markdown file with:

1. **Review Summary block** at the top:
```markdown
---
## 📋 Review Summary
**Date:** [date] | **Pass:** Standalone
**Status:** ✅ Ready to share / ⛔ Critical issues found

| Dimension | Status | 🔴 | 🟡 | 🟢 |
|-----------|--------|----|----|-----|
| Completeness | 🟢/🟡/🔴 | N | N | N |
| Risk Quality | ... | ... | ... | ... |
| Assumption Exposure | ... | ... | ... | ... |
| Standards Compliance | ... | ... | ... | ... |

**What works well:** [specific genuine strengths]
---
```

2. **Original document** with inline callouts inserted at the exact location of each finding:
```markdown
> 🔴 **[CRITICAL — Risk Quality]** Only one risk listed with no mitigation.
> *Fix: Add rollback, data integrity, and downstream dependency risks with named mitigations.*
```

Save as: `[initiative-name]-[doc-type]-reviewed.md`
Present with `present_files`.

---

## Workflow

### Orchestrated (called by architect in review-fix loop)
1. Receive the document and the current pass number (1, 2, or 3)
2. Run all four dimensions — evaluate only what is explicitly present
3. Classify every finding by severity
4. Return the structured `REVIEW_FINDINGS` block — no file, no present_files
5. Architect reads findings, applies fixes, and calls reviewer again if needed

### Standalone
1. If document not provided, ask user to paste or attach it
2. Run all four dimensions
3. Produce annotated Markdown file
4. Present with `present_files`

---

## Tone of Fix Instructions

Fix instructions must be **actionable and specific** — not vague.

❌ "Add more detail to the risk section."
✅ "Add a risk row for rollback failure: likelihood Medium, impact High, mitigation: blue/green deployment with automated smoke tests."

Every finding's `fix:` line should be precise enough that the architect can apply it
without further interpretation.
