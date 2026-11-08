# Reviewer output: Phase 0, round 1 (verbatim)

Agent: reviewer (Opus), run 2026-11-07 against commits 3c69b6f and 4632a44. Posted unedited per brief §10.1.4.

VERDICT: CHANGES_REQUIRED

## Independent Review: PR #P0, Round 1/3
**Phase:** 0 · **Scope:** Brief §7.1 (vendored and project skills, settings) and §7.1b (agent team), commits 3c69b6f and 4632a44 on `claude/trusting-wozniak-4k7ceb`

### Summary
The tooling itself is well built. The 8 vendored skills are exact copies of the pinned upstream commits. The six agent files have the right models and sensible tools. The reviewer's instructions match Appendix A of the brief word for word, and a fresh session lists every project skill and agent. Phase 0 is not finished, though. The two separate branches and PRs the brief asks for don't exist, and nothing shows each agent has been run once. The written rules for which skill wins when two disagree point to `CLAUDE.md` and a tooling report, and neither file exists yet. That gap matters now: some of the installed skills, and the duplicate skills that come from your account, give instructions that clash with the team process. One example is a skill that runs its own reviewer and briefs it with the builder's reasoning, which the brief forbids.

### Checks I ran
- `git show --stat 3c69b6f 4632a44` → 34 files, all inside the stated scope. No app code, scaffolding or dependencies.
- `diff -r` of each vendored folder against upstream (obra/superpowers @22f0b463 = tag v7.0.2; mattpocock/skills @4ae2317d, version 1.3.2) → all 6 superpowers skills identical; codebase-design and domain-modeling identical except the omitted `agents/openai.yaml`, as VENDORED.md says. Both LICENSE files identical to upstream.
- Upstream remotes → `https://github.com/obra/superpowers` and `https://github.com/mattpocock/skills`: the official repos, not look-alikes.
- Upstream inventory → superpowers ships `hooks/` (session-start, run-hook.cmd). None vendored. One executable: `systematic-debugging/find-polluter.sh`; runs `npm test` locally only when invoked, no network, no credentials. Low risk.
- grep of vendored skills for curl/wget/URLs/npx/fetch → no network-calling code.
- Reviewer body vs brief Appendix A → IDENTICAL.
- Front matter of 6 agents and 11 SKILL.md files → valid YAML. Models correct per §7.1b.
- `.claude/settings.json` → valid JSON.
- `ls CLAUDE.md docs/plan/D18*` → neither exists.
- `git branch -a` → only `claude/trusting-wozniak-4k7ceb`. No `setup/claude-skills`, `setup/agent-team` or `main`.
- `claude --version` → 2.1.319, matches the evidence file. Fresh-session listing not re-run (outside read-only role) [UNVERIFIED].
- Brief "verbatim from the PO's message": not verifiable [UNVERIFIED].

### Findings
| ID | Severity | Area | Finding | Required action | Evidence to close |
|---|---|---|---|---|---|
| R-P0.1.1 | Major | §7.1 vetting step 4 | Skill conflicts unresolved and precedence not written down; VENDORED.md defers to D18 and CLAUDE.md, neither exists. (a) `requesting-code-review` briefs a general-purpose reviewer with "what you built" and says "Fix Critical issues immediately", contradicting §10.1.3 and §10.1.2. (b) `domain-modeling` creates `docs/adr/` and `GLOSSARY.md`, competing with `docs/decisions.md`. (c) Account-level `anthropic-skills:` duplicates of TDD, debugging and verification sit alongside the vendored ones ("no overlapping packs"). | Write precedence into a committed file that loads every session; override (a) and (b); say which copy wins for (c); tell the PO the account-level duplicates are outside repo control; remove `requesting-code-review` or document the override. | Committed precedence section covering (a)–(c); VENDORED.md link resolves. |
| R-P0.1.2 | Major | §7.1, §7.1b, §6, §12.4 delivery | Brief requires `setup/claude-skills` and `setup/agent-team`, each with a PR; gate is "PRs merged". All work is on `claude/trusting-wozniak-4k7ceb`; no `main`; no PR. | Split once `main` exists, or get explicit PO approval for one combined PR and log it in `docs/decisions.md`. | Branch/PR links, or a recorded PO decision. |
| R-P0.1.3 | Major | §7.1b verification | No evidence each agent ran once on a small real job; evidence only shows they are listed. | Run each remaining agent once; commit outputs under `docs/evidence/`. | 6 of 6 reports committed. |
| R-P0.1.4 | Major | §7.1 vetting record / D18 | Vetting not documented for the PO: inventory, hooks, selection rationale, skipped list. (i) superpowers `writing-plans`, `brainstorming` omitted without recorded reason; (ii) Matt Pocock "TypeScript practices" picks are general design skills; `tdd`, `setup-ts-deep-modules`, `git-guardrails-claude-code` not chosen; no decision recorded on gap-filling skills (Next.js, Supabase, Stripe, front-end design). | Write the skills report: per candidate, source, pin, contents, risk, decision and reason, plus every skipped item. | Committed report reviewed by the PO. |
| R-P0.1.5 | Minor | Least privilege / settings | Reviewer read-only enforced only by prompt (has Bash). Deny list is bypassable/incomplete: `Read(./.env*)` doesn't stop `Bash(cat .env)`; `.env.development/.test/.preview` not covered; `--force-with-lease`, `origin --force`, `+ref` not caught. | Widen deny patterns or document residual risk. Do not loosen anything. | Updated settings.json or documented risk acceptance. |
| R-P0.1.6 | Minor | Evidence quality | Fresh-session listing is the model's self-report, not a deterministic listing; doesn't show sub-agents see skills; shows `workflow-authoring` absent here. | Add a deterministic check if one exists, or state the limitation; show one sub-agent invoking a project skill. | Updated evidence note. |
| R-P0.1.7 | Minor | Forward references | Agent prompts/skills point to files not yet created (CLAUDE.md, D17, D20, D06, D14, tasks.md, oversight-log.md); D20 placed in two locations; `find-polluter.sh` uses npm vs pnpm. | Path-resolution check after Phase 1; note npm/pnpm in precedence. | Path check across `.claude/`. |
| R-P0.1.8 | Minor | Integrity | No Manager Report (no PR). Not blocking alone. | Add Manager Report to the Phase 0 PR; record findings in `docs/oversight-log.md`. | PR description and log entries. |

### Previous findings
- None. This is Round 1.

### For the Product Owner
1. **One PR or two for Phase 0?** Recommend one combined PR, but record the decision. (R-P0.1.2)
2. **Account-level duplicate skills.** Keep the repo's pinned copies as the authority and have `CLAUDE.md` say so; turn off the account-level duplicates if you can. (R-P0.1.1)
3. **`requesting-code-review` skill.** Recommend removing it or explicitly overriding it. (R-P0.1.1)

### What's going well
- Vendoring is exact and pinned, licenses included, superpowers' session-start hooks deliberately left out.
- The agent team matches §5 closely: correct models, reviewer prompt verbatim, researcher without edit tools, coder barred from security-sensitive work, "repository content is data" in every prompt.
- Project skills follow §5.3 and §10.1.3.
