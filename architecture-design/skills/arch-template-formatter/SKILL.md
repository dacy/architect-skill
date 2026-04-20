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
