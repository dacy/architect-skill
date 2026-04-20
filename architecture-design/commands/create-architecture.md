---
description: Design a system architecture through a guided 10-phase workflow, producing a Solution Intent document. Supports resuming in-progress sessions.
argument-hint: [initiative description] [--template <path>] [--resume <path>]
allowed-tools: Task
---

# create-architecture Command

You are the entry point for the `/create-architecture` command. Your only job is to parse the user's arguments and hand off to the `arch-orchestrator` agent.

## Parsing Arguments

Raw arguments: `$ARGUMENTS`

1. If `--resume <value>` appears anywhere in the arguments, extract `<value>` as `resume_path` and remove `--resume <value>` from the remaining text.
2. If `--template <value>` appears anywhere in the remaining arguments, extract `<value>` as `template_path` and remove `--template <value>` from the remaining text.
3. Everything remaining is `initiative_input`. It may be empty, a plain description, a structured brief, or a URL.

## Handoff

Invoke the `arch-orchestrator` agent immediately with:
- `initiative_input`: the parsed initiative text (empty string if nothing was provided)
- `template_path`: the template path (null if `--template` was not present)
- `resume_path`: the path to an in-progress Solution Intent document (null if `--resume` was not present)

Do not do any analysis or design work yourself. The orchestrator manages all phases.
