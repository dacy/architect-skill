---
name: arch-template-formatter
description: Reformats a completed Solution Intent document to match a provided template structure. Triggered by arch-orchestrator in Phase 9 when a template is available (runtime --template path or default references/solution-intent-template.md). Content is preserved — only structure, headings, and formatting change.
---

# arch-template-formatter Skill

## Overview

Takes a completed Solution Intent document and a target template, then maps the Solution Intent's content into the template's structure. No content is dropped — if a Solution Intent section has no matching template section, append it as an appendix at the end.

## Input
- `doc_path`: absolute path to the in-progress Solution Intent document (you will rewrite this file in place)
- `template_path`: absolute path to the target template markdown

## Process

### Step 1: Read both files

Read `doc_path` (the Solution Intent) and `template_path` (the target template).

### Step 2: Parse the template structure

Identify all sections and heading levels in the template. Note any placeholder patterns:
- `{{section_name}}` — direct injection point
- `[placeholder text]` in square brackets — replace with matching content
- `<!-- comment -->` — HTML comments describing what belongs there

### Step 3: Map Solution Intent sections to template sections

Use semantic matching — a template heading "Architecture Overview" maps to Solution Intent Section 3 (High-Level Architecture). Preserve the ASCII diagram, all tables, and all code blocks exactly. Do not paraphrase content — use the original text.

### Step 4: Preservation contract

The following sections MUST be preserved verbatim as appendices after the template's numbered sections, in this order:
1. `## Clarifications` (full Q&A history)
2. `## Review Findings` (all severity levels)

Any other Solution Intent content that has no matching template section goes into an `## Additional Information` appendix after those two.

### Step 5: Write the formatted document

Write the final document back to `doc_path`, using the template's structure with the Solution Intent's content. Maintain all markdown formatting, tables, and diagrams from the original. Preserve the frontmatter block (`status`, `phase_completed`) at the top; if the template does not account for it, re-prepend it after writing.

## Output

Confirm to the caller: `formatted: true, doc_path: <path>`. Do not return the document text.

## Reference Files

- `references/solution-intent-template.md` — default company Solution Intent template.
  **TODO:** Replace with your organisation's standard Solution Intent format. The current file contains a generic starting template.
