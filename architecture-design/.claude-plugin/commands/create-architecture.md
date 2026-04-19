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
