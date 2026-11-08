# Vendored third-party skills

These skills are **copied** (vendored) into `.claude/skills/` so they load automatically in every Claude Code session and sub-agent on this repo, including ephemeral cloud sessions, with no network fetch and no plugin hooks. They are byte-for-byte copies of the pinned commits below (except the deleted `agents/openai.yaml` files from mattpocock/skills, which are Codex metadata Claude Code does not use). Do not edit them in place; to update, re-vet the new commit and replace the folder.

| Skill folder | Source repo | Pinned commit | Release | License |
|---|---|---|---|---|
| test-driven-development | github.com/obra/superpowers | 22f0b463e7a5e579904dd6211a520321035526f7 | v7.0.2 | MIT (`LICENSE-superpowers`) |
| systematic-debugging | github.com/obra/superpowers | 22f0b463e7a5e579904dd6211a520321035526f7 | v7.0.2 | MIT |
| verification-before-completion | github.com/obra/superpowers | 22f0b463e7a5e579904dd6211a520321035526f7 | v7.0.2 | MIT |
| receiving-code-review | github.com/obra/superpowers | 22f0b463e7a5e579904dd6211a520321035526f7 | v7.0.2 | MIT |
| dispatching-parallel-agents | github.com/obra/superpowers | 22f0b463e7a5e579904dd6211a520321035526f7 | v7.0.2 | MIT |
| codebase-design | github.com/mattpocock/skills | 4ae2317ddb22b046e26455e4f9c9b3aaaf62212d | plugin v1.3.2 | MIT (`LICENSE-mattpocock-skills`) |
| domain-modeling | github.com/mattpocock/skills | 4ae2317ddb22b046e26455e4f9c9b3aaaf62212d | plugin v1.3.2 | MIT |

Project-authored skills (not vendored): `add-supabase-migration`, `write-task-brief`, `review-subagent-pr`.

Full vetting notes, the skipped list and precedence rules: `docs/plan/D18-tooling-report.md`.

## Cross-references that do not resolve
Some vendored text mentions `superpowers:<name>` skills. Project skills load without the `superpowers:` prefix, so read `superpowers:test-driven-development` as `test-driven-development` and so on. References to skills we did **not** vendor or later removed (`requesting-code-review`, `subagent-driven-development`, `executing-plans`, `using-git-worktrees`, `writing-skills`) are superseded by the team process in `CLAUDE.md` and `docs/plan/manager-brief.md` §5.4.
