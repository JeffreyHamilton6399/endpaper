---
name: write-task-brief
description: Use when a lead or the Head Manager turns a work package from docs/tasks.md into a Task Brief for a coder agent. Produces a self-sufficient brief a no-context Haiku coder can execute, saved at docs/briefs/<task-id>.md.
---

# Write a Haiku Task Brief

The coder starts with **zero context**. If something is not in the brief or in a file the brief names, the coder does not know it.

## Before writing
1. Confirm the task is allowed for a coder: not schema/RLS, auth, payments/webhooks, encryption, the rewrite prompt, CI permissions or security-sensitive work (brief §5.3). If it is, stop: it belongs to the backend-lead or Head Manager.
2. Confirm the phase allows it (Phase 2: wireframes only; app code only after the PO authorizes Phase 3).
3. Check file overlap with other briefs in flight; note whether it can run in parallel.

## Write the brief
Copy the template from `docs/plan/D17-task-brief-template.md` and fill every section. Rules:
- **Files**: an exhaustive list of files to create or modify, plus files to read for context, plus files not to touch.
- **Requirements**: numbered, concrete, testable. Paste the exact copy, tokens, type signatures or interfaces needed rather than pointing vaguely at a doc.
- **Acceptance criteria**: observable outcomes, each checkable by a command or a precise manual step.
- **Verification**: the exact commands to run and what passing output looks like.
- **Dependencies**: "none" unless explicitly approved by the Head Manager.
- Include the coder rules block verbatim from the template.

## After writing
- Save to `docs/briefs/<task-id>.md` and set the task to "In progress" in `docs/tasks.md` when dispatched.
- Re-read it as if you knew nothing. Anything you had to infer is a gap; fix it.
