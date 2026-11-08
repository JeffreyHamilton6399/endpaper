# D14. Repository layout

**Status:** Draft · **Owner:** Head Manager. Folders marked *(Phase 3)* do not exist until the PO authorizes building.

```
endpaper/
├── CLAUDE.md                     project memory for every session and agent (D15)
├── README.md                     (Phase 3) human quick start
├── .claude/
│   ├── settings.json             project permission rules
│   ├── agents/                   team definitions (design-lead, frontend-lead, backend-lead, coder, researcher, reviewer)
│   └── skills/                   vendored + project skills (VENDORED.md lists sources and pins)
├── .github/                      (Phase 3) workflows/ci.yml, pull_request_template.md, CODEOWNERS
├── docs/
│   ├── plan/                     approved plan: manager-brief.md, D01–D22
│   ├── briefs/                   Task Briefs, one file per task: T-xx.md
│   ├── research/                 researcher findings, recorded by the Head Manager
│   ├── evidence/                 command outputs and agent-run proof
│   ├── reviews/                  Reviewer outputs, verbatim, one file per PR round
│   ├── tasks.md                  task list with status
│   ├── decisions.md              decision log
│   ├── oversight-log.md          every Reviewer finding + response + resolution
│   ├── launch-checklist.md       quality bar, mapped to tasks
│   ├── licenses.md               every font/image/icon/sound and its licence
│   └── legal/                    (Phase 3) drafts of Privacy, Terms, Refunds (marked "requires legal review")
├── wireframes/                   THROWAWAY Phase 2 prototype (static HTML/CSS). Deleted or archived after approval.
├── src/                          (Phase 3)
│   ├── app/                      Next.js App Router routes
│   │   ├── (marketing)/          landing, legal pages (server components, public)
│   │   ├── (auth)/               sign-in, sign-up, callback
│   │   ├── (app)/                authenticated: book, record, entry/[id], settings, subscription
│   │   └── api/                  route handlers: rewrite, transcription-token, stripe/webhook, export, account/delete
│   ├── components/               ui/ (shadcn), book/, recording/, layout/
│   ├── features/                 feature modules, each with pure logic + tests:
│   │   ├── entries/              entry-date rules, revision logic
│   │   ├── rewrite/              prompt builder, output schema, edge-case rules (server-only)
│   │   ├── transcription/        provider-agnostic interface + adapters
│   │   ├── billing/              entitlement logic, webhook handlers (server-only)
│   │   └── export/               JSON / text / PDF builders
│   ├── lib/
│   │   ├── db/                   supabase clients (server/browser), types.gen.ts (generated, committed)
│   │   ├── env/                  Zod-validated env (server.ts imports "server-only")
│   │   └── observability/        Sentry init with content scrubbing, cost logger
│   └── styles/                   tokens.css (from D20), globals.css
├── supabase/                     (Phase 3) config.toml, migrations/, seed.sql, tests/ (RLS isolation)
├── tests/                        (Phase 3) e2e/ (Playwright), fixtures/ (fixture audio, sample transcripts)
├── evals/rewrite/                (Phase 3) D7 evaluation set + runner (synthetic / consented data only)
└── public/                       (Phase 3) icons, manifest, fonts (if self-hosted), OG images
```

## Conventions
- **File names:** kebab-case (`entry-date.ts`); React components PascalCase exports in kebab-case files.
- **Tests** sit next to the code: `entry-date.test.ts`. RLS tests in `supabase/tests/`; e2e in `tests/e2e/`.
- **Server-only modules** (anything holding a secret or calling Anthropic/Stripe/transcription) `import "server-only"` at the top, so an accidental client import fails the build.
- **Generated files** (`types.gen.ts`) are committed and never edited by hand.
- **Feature modules** export a small public interface from `index.ts`; other modules import only from there (deep modules, see `.claude/skills/codebase-design`).
- **Task IDs** `T-xx` link tasks.md ↔ briefs ↔ branches (`task/T-xx-slug`) ↔ PRs.
- **Finding IDs** `R-<PR#>.<round>.<n>` link reviews ↔ oversight log.
