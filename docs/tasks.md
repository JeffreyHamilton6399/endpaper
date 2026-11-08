# Tasks (D16: v1 task breakdown)

Status values: **To do / In progress / In review / Done**. Owners: **HM** = Head Manager, **BE** = backend-lead, **FE** = frontend-lead, **DL** = design-lead, **C** = Haiku coder (from a brief written by the named lead). Security-sensitive work is never **C** (brief §5.3). "Par." = can run in parallel with other tasks in the same group once dependencies are met.

## Phases 0–2 (authorized)
| ID | Title | Owner | Status | Evidence |
|---|---|---|---|---|
| P0-a | Vet and vendor skills; project skills; settings | HM | In review | `.claude/skills/`, `docs/plan/D18-tooling-report.md`, review R-P0.1 |
| P0-b | Define agent team; run each agent once | HM | In review | `.claude/agents/`, `docs/evidence/phase0-agent-runs.md` |
| P1 | Plan D1–D22 | HM + leads | In review | `docs/plan/` |
| P2 | Clickable wireframes, 3 book directions | DL specs, C builds | In review | `wireframes/` |

## Phase 3+ (NOT authorized; plan only)
Each task gets one branch `task/T-xx-slug` and one PR. Acceptance criteria are summarised; the full version goes in its Task Brief.

### Group A: Foundation, CI, observability
| ID | Title | Objective / key files | Depends | Owner (why) | Acceptance (summary) | Par. |
|---|---|---|---|---|---|---|
| T-01 | Scaffold app | Next.js, TS strict, pnpm, Node pin, Tailwind tokens from D20; `package.json`, `tsconfig.json`, `.nvmrc`, `src/app/` | – | FE (sets conventions every later brief relies on) | `pnpm typecheck/lint/test/build` pass; `noUncheckedIndexedAccess` on | N |
| T-02 | CI workflow | `.github/workflows/ci.yml`: install, typecheck, lint, test, build; least-privilege `permissions:` | T-01 | BE (CI permissions are security-sensitive) | Required checks run on a PR; `permissions: contents: read` | N |
| T-03 | Local Supabase + env schema | `supabase/config.toml`, `src/lib/env/` Zod env, `.env.example` | T-01 | BE (secrets handling) | `supabase start` works; app fails fast on a missing env var; no secret in repo | Y |
| T-04 | Core schema + RLS | Migrations for D6 tables, RLS deny-by-default, generated types | T-03 | BE (schema/RLS never delegated) | `db reset` clean; every table RLS on; types committed | N |
| T-05 | RLS isolation test suite | pgTAP + two-user Vitest test; CI job | T-04 | BE | A cannot read/write B on every table; CI gate | N |
| T-06 | Sentry with content scrubbing | `src/lib/observability/` | T-01 | BE (privacy-critical) | Canary-string test proves journal text never leaves in an event | Y |
| T-07 | Cookieless analytics wrapper | typed `track()` with allow-listed events | T-01 | C from FE brief (well-specified) | Only allow-listed names compile; no cookies set | Y |
| T-08 | Vercel project + Ignored Build Step | `vercel.json` / script skipping docs-only changes | T-01 | HM | Docs-only PR does not build; app PR gets a preview | Y |
| T-09 | Link check, Lighthouse CI, axe in CI | workflow jobs | T-02, T-70 | C from FE brief | Broken link fails CI; Lighthouse ≥ 90 on public pages | Y |

### Group B: Auth, onboarding, legal
| T-10 | Auth (email OTP/magic link) | Supabase Auth, `@supabase/ssr`, middleware/proxy, sign-in/up routes | T-04 | BE (auth) | Session cookies httpOnly; protected routes redirect; e2e sign-in | N |
| T-11 | Sign-up consent + age gate | Unticked Terms/Privacy box, vendor disclosure, 13+/16+ confirmation stored with timestamp and version | T-10 | BE logic + C UI from DL brief | Cannot submit unticked; consent row recorded | N |
| T-12 | Legal page drafts | `docs/legal/`, `(marketing)/privacy`, `/terms`, `/refunds`, `/help` (crisis resources) | T-01 | DL writes copy; C builds pages | Pages marked "Draft: requires legal review"; footer links work | Y |
| T-13 | Onboarding: writing samples | Paste/add/skip; Zod validation; stored via RLS | T-10 | C from FE brief (server action reviewed by BE) | Samples saved and visible only to owner; skip path works | Y |
| T-14 | Time-zone capture | Auto-detect + edit in settings | T-10 | C from FE brief | Stored IANA zone; edit persists | Y |

### Group C: Recording and transcription
| T-20 | Transcription interface + adapter | `src/features/transcription/`: provider interface, primary adapter (D5) | T-03 | BE (vendor credentials) | Contract tests with recorded fixtures | N |
| T-21 | Short-lived token route | `api/transcription-token`: auth → entitlement → Zod → mint token | T-20, T-50 | BE (secrets) | Token TTL ≤ 60 s; unauthenticated → 401; non-entitled → 402 | N |
| T-22 | Recorder client | AudioWorklet PCM + MediaRecorder fallback, wake lock | T-01 | FE (browser quirks need judgement) | Works on iOS Safari + Android Chrome (manual device test) | Y |
| T-23 | Live transcript UI | Start/End, live region, accessible names, mic explainer, states | T-22 | C from FE brief | axe clean; screen reader announces state; 360 px OK | N |
| T-24 | Recording durability | IndexedDB autosave, recovery prompt, flush on pagehide | T-22 | C from FE brief | Kill tab mid-recording → transcript recovered | N |
| T-25 | Save transcript | Server action stores transcript + entry_date (T-40) | T-04, T-40 | BE | Saved under correct date; RLS enforced | N |
| T-26 | Error states | Mic blocked, offline ("saved on this device, will sync"), provider down | T-23 | C from DL/FE brief | Each state reachable in a test, plain-language copy | Y |

### Group D: Rewrite
| T-30 | Rewrite prompt + schema | `src/features/rewrite/`: prompt builder, Zod output, edge rules | T-04 | HM (prompt never delegated) | Unit tests for short/empty/long; injection fixtures | N |
| T-31 | Rewrite route | `api/rewrite`: auth → owner → entitlement → Zod; streaming; timeout/retry; cost log (no content) | T-30, T-50 | BE | Streams; retries once; logs tokens/cost only (test asserts) | N |
| T-32 | Rewrite eval harness | `evals/rewrite/` runner + set (D7 §7) | T-30 | HM | Thresholds reported; run on prompt/model change | Y |
| T-33 | "Writing your page" state | Crafted waiting state, layout-matched | T-31 | C from DL brief | No blank screen or bare spinner; reduced motion | Y |

### Group E: Book hub and entry view
| T-40 | Entry-date rule | Pure function (sample brief in D17) | T-01 | C from BE brief | All D17 acceptance criteria | Y |
| T-41 | Book data loader | Month-chunked queries + book index | T-04 | BE (data access/RLS) | Paginates; RLS tests | Y |
| T-42 | Page-turn component | CSS 3D + Pointer Events per D4b; reduced-motion cross-fade | T-01 | FE (signature interaction) | 60 fps on reference Android; keyboard; reduced motion | Y |
| T-43 | Book hub screen | Cover, chapters, open to latest, jump to date, URL per date | T-41, T-42 | C from FE brief | D4b acceptance list | N |
| T-44 | Entry view + transcript toggle | Rewrite page, original one tap away | T-43 | C from FE brief | Toggle keyboard-operable; state announced | N |
| T-45 | Editing + revisions | Edit rewrite and transcript; revision rows; change date | T-44, T-04 | BE storage + C UI | Revision saved per edit; "Saved" confirmation | N |
| T-46 | Skeletons + empty states | First day, empty chapter | T-43 | C from DL brief | Layout-matched skeletons | Y |

### Group F: Payments
| T-50 | Entitlement model | Pure entitlement function + subscriptions table usage | T-04 | BE (payments) | Trialing/active/past_due/canceled cases tested | N |
| T-51 | Stripe Checkout | Monthly/yearly, trial, tax disclosed | T-50 | BE | Test-mode checkout creates subscription | N |
| T-52 | Stripe webhook | Signature verify, idempotent via stripe_events, updates subscriptions | T-50 | BE | Replay of same event is a no-op; bad signature 400 | N |
| T-53 | Customer Portal + cancel | Self-serve cancel, as easy as signup | T-51 | BE + C UI | Cancel in ≤ 3 taps from Settings; no confirmshaming | N |
| T-54 | Trial reminder email | Sent ≥ 3 days before trial ends; transactional only | T-52 | BE | Test sends in staging; no marketing content | Y |
| T-55 | Trial-ended state | Read/export allowed; record blocked with honest copy | T-50 | C from DL brief | e2e for expired trial | Y |

### Group G: Settings, export, delete
| T-60 | Settings screen | Account, samples, reminder time (stored only), links | T-13 | C from DL brief | All controls work; success confirmations | Y |
| T-61 | Export | JSON + plain text + PDF of all entries | T-04 | BE (bulk data access) + C PDF layout | Export contains everything, nothing of other users | Y |
| T-62 | Account deletion | Delete DB rows, storage, auth user, Stripe customer; backups expire within stated window | T-52 | BE (irreversible, security-critical) | Re-login impossible; rows gone; documented backup window | N |

### Group H: Landing, marketing, polish
| T-70 | Landing page | Value prop, price, start trial, real screenshots | T-43 | C from DL brief | No prohibited patterns; Lighthouse ≥ 90 | Y |
| T-71 | PWA | Manifest, icons, theme colour, service worker (offline shell) | T-01 | C from FE brief | Installable on iOS and Android | Y |
| T-72 | Polish sweep | Titles, meta, OG, 404/error pages, dynamic year, mailto, logo → home | T-70 | C from DL brief | Launch checklist P-items pass | Y |
| T-73 | `docs/licenses.md` | Every font/icon/image/sound licence | T-70 | DL | Every asset listed with licence | Y |
| T-74 | Waitlist page (validation) | Priced waitlist per D12 (can ship before the app) | T-01 | C from DL brief | Captures email + chosen price; consent; no cookies | Y |

### Group I: QA and launch
| T-80 | Golden-path e2e | Sign up → record (fixture audio) → rewrite → page appears | T-31, T-43 | C from FE brief | Green in CI | N |
| T-81 | Real-device test pass | iOS Safari + Android Chrome checklist | T-80 | HM + PO | Signed-off checklist | N |
| T-82 | Accessibility audit | Manual keyboard + screen reader + contrast on textures | T-80 | FE | WCAG 2.2 AA issues fixed or waived by PO | N |
| T-83 | Security review | Threat model D8 re-check, secrets scan, RLS audit | T-80 | Reviewer + BE | No open Critical/Major | N |
| T-84 | Legal review handoff | Drafts to a lawyer; update copy | T-12 | PO (only the PO can engage counsel) | Lawyer sign-off recorded | Y |
| T-85 | Launch checklist sign-off | Every item done or waived | all | HM + PO | `docs/launch-checklist.md` complete | N |
