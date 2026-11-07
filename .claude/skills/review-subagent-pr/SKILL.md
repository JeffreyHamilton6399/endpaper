---
name: review-subagent-pr
description: Use when the Head Manager has coder or lead output ready and needs to verify it, get the lead's review, run the independent Reviewer, record findings and report to the PO. Covers the brief §5.4 steps 4-7 and §10.
---

# Review a sub-agent PR

## 1. Verify yourself first (evidence, not reports)
- Read the actual diff (`git diff <base>...HEAD`), not just the coder's report.
- Confirm only the brief's listed files changed.
- Run the checks: `pnpm typecheck`, `pnpm lint`, `pnpm test`, `pnpm build` (Phase 3+), or the brief's own verification commands (Phase 2). Paste the results into the Manager Report.

## 2. Lead review
Ask the owning lead to review the diff against the brief and `CLAUDE.md`. Send FIX notes back to the coder with numbered, specific items. A task that fails review twice is escalated to the lead or Head Manager.

## 3. Independent Reviewer
Invoke the `reviewer` agent with **only**: the PR number/branch, the base ref, the task ID(s), and pointers to the brief and relevant docs. **Do not** include your reasoning, conclusions or the coder's self-assessment (brief §10.1.3). Ask it to return its standard output format.

## 4. Respond to findings
- Answer every finding by ID: Accepted (with the fix), Partially accepted, or Declined (with reasons). A declined Critical or Major goes to the PO.
- Record each finding, your response and its resolution in `docs/oversight-log.md`.
- Re-run the Reviewer for the next round if anything Critical/Major was fixed. Maximum 3 rounds, then escalate to the PO.

## 5. Report
- Post the Reviewer's output unedited as a PR comment (when a PR exists).
- Update the Manager Report in the PR description and end your reply to the PO with it, including the verdict and all Critical/Major findings unedited.
- The PO merges. Never merge yourself.
