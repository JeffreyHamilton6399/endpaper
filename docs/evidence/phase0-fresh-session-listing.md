# Evidence: fresh-session skill and agent listing

Command (run 2026-11-07 from repo root, after commit 3c69b6f):

```bash
claude -p "Without using any tools, list (1) every skill name available to you via the Skill tool and (2) every subagent_type available to the Agent tool. Output two plain bulleted lists, nothing else." --max-turns 1
```

Claude Code version: 2.1.319 (Claude Code)

Output (verbatim):

```
- session-start-hook
- add-supabase-migration
- codebase-design
- dispatching-parallel-agents
- domain-modeling
- receiving-code-review
- requesting-code-review
- review-subagent-pr
- systematic-debugging
- test-driven-development
- verification-before-completion
- write-task-brief
- dataviz
- artifact-design
- artifact-diagramming
- artifact-capabilities
- update-config
- keybindings-help
- code-review
- simplify
- fewer-permission-prompts
- loop
- claude-api
- workflow-authoring
- run
- plugin-authoring
- init
- security-review
- anthropic-skills:artifact-emulator
- anthropic-skills:caveman
- anthropic-skills:docs
- anthropic-skills:docx
- anthropic-skills:google-workspace
- anthropic-skills:import-memory
- anthropic-skills:karpathy-coding-guidelines
- anthropic-skills:lean-mode
- anthropic-skills:morning
- anthropic-skills:pdf
- anthropic-skills:pptx
- anthropic-skills:skill-creator
- anthropic-skills:systematic-debugging
- anthropic-skills:test-driven-development
- anthropic-skills:ui-taste
- anthropic-skills:verification-before-completion
- anthropic-skills:xlsx

- fork
- backend-lead
- claude
- claude-code-guide
- coder
- design-lead
- Explore
- frontend-lead
- general-purpose
- Plan
- researcher
- reviewer
- statusline-setup
```

Project skills present: add-supabase-migration, codebase-design, dispatching-parallel-agents, domain-modeling, receiving-code-review, requesting-code-review, review-subagent-pr, systematic-debugging, test-driven-development, verification-before-completion, write-task-brief (11/11). Project agents present: backend-lead, coder, design-lead, frontend-lead, researcher, reviewer (6/6). Other entries come from the cloud environment or the user's account, not this repo.
