# D19: Toolchain Decisions

Status: Draft for PO review (Phase 1) | Author: frontend-lead | Date: 2026-11-07

## How these were checked

- Versions come from the **npm registry** (`registry.npmjs.org/<pkg>/latest`) and the **nodejs.org release index**, queried on 2026-11-07. Each is "latest stable" that day.
- The official docs sites (nextjs.org, vercel.com) **could not be reached from this session** (network allow-list). Anything that depends on a docs page is marked **[VERIFY]**, and the Head Manager should either allow those hosts or have the researcher confirm them.
- Rule for Phase 3: versions are **pinned exactly** in `package.json` and the lockfile; the table is the starting point, re-checked on the day Phase 3 starts.
- "Pin" means we write an exact version (no `^`) so every laptop, CI run and Vercel build gets the same code.

## 1. Summary table

| Concern | Brief default | Decision | Version (2026-11-07) | Change from brief? |
|---|---|---|---|---|
| Node.js | LTS | **Node 24 LTS "Krypton"** | 24.22.2 | Confirm (see 2) |
| Package manager | pnpm | pnpm | 12.18.0 | Confirm |
| Framework | Next.js App Router | Next.js | 16.4.5 | Confirm |
| UI library | React | React / React DOM | 19.3.1 | Confirm |
| Language | TS strict | **TypeScript 6.0.3 for now, move to 7.x later** | 6.0.3 (7.0.2 is latest) | **Justified change** |
| Styling | Tailwind | Tailwind CSS (v4, CSS-first tokens) | 4.3.5 | Confirm |
| Components | shadcn/ui | shadcn CLI + Radix-based components | shadcn 4.23.0 | Confirm |
| Book animation | recommend | Hand-rolled CSS 3D, no library (D4b) | none | Decision |
| Validation | Zod | Zod 4 | 4.6.7 | Confirm |
| Data access | Supabase JS + generated types | supabase-js, @supabase/ssr, Supabase CLI | 2.119.0, 0.12.7, 3.0.0 | Confirm |
| AI | Anthropic SDK | @anthropic-ai/sdk | 1.6.0 | Confirm |
| Transcription | decided in D5 | Interface `TranscriptionProvider` (name only here) | n/a | Confirm |
| Payments | Stripe SDK | stripe (server only) | 23.1.0 | Confirm |
| Unit/integration tests | Vitest | Vitest | 5.1.0 | Confirm |
| E2E tests | Playwright | @playwright/test | 1.65.0 | Confirm |
| Security tests (RLS) | required | pgTAP via Supabase CLI plus Vitest cross-user API test | see 12 | Decision |
| Lint/format | ESLint+Prettier or Biome | **ESLint + Prettier** | ESLint 10.14.0, Prettier 3.9.10 | Choice |
| Pre-commit | required | **Lefthook** | 2.2.2 | Choice |
| Error tracking | Sentry | @sentry/nextjs with scrubbing | 11.11.0 | Confirm |
| Analytics | privacy-first | **Cookieless**: Vercel Web Analytics or Plausible [VERIFY] | n/a | Choice |
| Link checker | required later | lychee (GitHub Action) | [VERIFY] | Choice |
| Lighthouse | required later | @lhci/cli | 0.16.0 | Confirm |
| Accessibility checks | required later | @axe-core/playwright | 4.14.0 | Confirm |
| PWA support | PWA | Hand-written manifest + minimal service worker; Serwist only if needed | serwist 9.5.13 | Decision |

## 2. Runtime and package manager

**Node.js: 24.22.2 (LTS "Krypton").** The Node index also lists 26.13.0 "Lithium" as LTS (released 2026-10-28, ten days old). Recommendation: **start on 24**, because it is established, every tool below supports it, and Vercel support for 26 is unconfirmed [VERIFY on Vercel's Node.js versions page]. Plan to move to 26 in a dedicated upgrade task after Vercel confirms it. Minimums found: Vitest needs 22.12 or newer, supabase-js needs 22 or newer, Lighthouse CI needs 22.19 or newer, Sentry needs 22.12 or newer, lint-staged 17 needs 22.22. So **Node 22 would be the floor, 24 is comfortable.**

Pinning, in three places that must agree:
- `.nvmrc`: `24.22.2`
- `package.json`: `"engines": { "node": "24.22.2", "pnpm": "12.18.0" }` and `"packageManager": "pnpm@12.18.0"` (Corepack reads this). Use `engine-strict=true` in `.npmrc` so a wrong Node fails fast.
- Vercel project setting "Node.js Version" set to 24.x; CI uses `actions/setup-node` with `node-version-file: .nvmrc`.

**pnpm 12.18.0.** Confirmed as the brief default. It is a recent major (12), so use the `pnpm-lock.yaml` and run `pnpm install --frozen-lockfile` in CI. pnpm 12 defaults on script-safety (it blocks dependency install scripts unless allowed) [VERIFY against pnpm 12 release notes]; that is good for us: we keep an explicit allow list.

## 3. Next.js 16.4.5 and React 19.3.1

- Next.js 16.4.5 (published 2026-11-03); requires Node 20.9 or newer; peer React `^18.2 || ^19`. We use React 19.3.1.
- App Router only; no Pages Router.
- Next 16 no longer ships `next lint` [VERIFY]; the lint command is plain `eslint` with `eslint-config-next` 16.4.5 (peer ESLint 9 or newer, compatible with ESLint 10.14.0).
- Turbopack is the default dev/build bundler in Next 16 [VERIFY]. Build uses whatever `next build` defaults to; we do not add a custom webpack config.
- React Compiler: leave **off** at first. It is an optimization we do not need until measured.

## 4. TypeScript: strict, and why not 7 yet

Finding: the registry's `latest` is **TypeScript 7.0.2** (the native, much faster rewrite, published 2026-07-08). But `typescript-eslint` 8.72.1 (the TypeScript brain of ESLint) declares support only for `typescript >=4.8.4 <6.1.0`. So TS 7 plus ESLint's type-aware rules is an unsupported mix today.

Decision: **pin `typescript@6.0.3`** (latest 6.x, 2026-04-16). Revisit when `typescript-eslint` lists 7.x in its supported range (and Next confirms 7 [VERIFY]). Upgrading is a single-task change. If the PO prefers speed over safety, the alternative is Biome (section 11), which does not depend on the TypeScript compiler API; that is not recommended for the reason given there.

`tsconfig.json` essentials (illustrative, not code to commit now):
```jsonc
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,   // brief 4.1
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "allowJs": false,                    // no JavaScript source files
    "paths": { "@/*": ["./src/*"] }
  }
}
```
- Type-check command `tsc --noEmit` is a required CI gate (brief 4.1). `any` banned by lint rule `@typescript-eslint/no-explicit-any` (error), with a `// why:` comment escape hatch only via a justified disable directive that requires a description.
- Config files are TypeScript too (`next.config.ts`, `vitest.config.ts`, `playwright.config.ts`, `eslint.config.ts` via `jiti`). The one unavoidable exception is any tool that accepts only `.js`/`.mjs` config; list in CLAUDE.md.
- `@types/node` aligned to Node major (24.x) rather than latest 26.6.6 [use `@types/node@24`], and `@types/react` 19.3.0.

## 5. Tailwind CSS 4.3.5: the token approach

- Tailwind v4 is **CSS-first**: design tokens live in one CSS file inside an `@theme` block, not in a JavaScript config. Each token becomes a CSS variable and automatically a utility class.
- Token groups (names agreed with D20 style guide): `--color-paper-*`, `--color-ink-*`, `--color-accent-*`, semantic roles (`--color-surface`, `--color-text`, `--color-text-muted`, `--color-focus`), `--font-serif-entry`, `--font-sans-ui`, `--text-*` scale, `--spacing` base, `--radius-*` (deliberately several, not one uniform value), `--shadow-leaf`, `--texture-paper` (URL to a compressed image), `--ease-page`, `--duration-turn`.
- Semantic layer: components use roles ("surface", "text") not raw colours, so the 2 to 3 book directions from Phase 2 and a possible dark/night mode are token swaps (a `[data-theme]` attribute), not code rewrites.
- Rules enforced by lint where possible: no raw hex in components; no arbitrary values for colours.
- Fonts via `next/font` (self-hosted, no layout shift). Font choices and licences recorded in `docs/licenses.md` (brief 8.1 item 16); no Inter/Geist/Space Grotesk as the primary voice (brief 8.2).
- Uses `@tailwindcss/postcss` (same version line as tailwindcss) [VERIFY exact pairing at install].

## 6. shadcn/ui

- shadcn is not a dependency you import; the CLI (`shadcn` 4.23.0) copies component source into our repo, so we own it and can restyle it to the paper/ink tokens. Matches the brief's "or equivalent headless kit".
- Used for: forms, dialogs/sheets (Contents, calendar), toasts, settings, switch, tabs, dropdown. Underlying primitives (Radix UI or Base UI, whichever the shadcn CLI picks [VERIFY]) provide the focus trapping and ARIA we need.
- Default look is deliberately overridden (brief 8.2: no unmodified default kit). Each imported component is restyled to tokens and reviewed by design-lead.
- Not used for the book hub (bespoke, D4b). Icons: custom glyphs for key actions (Record, End, Contents, Bookmark); a small licensed set (e.g. lucide, ISC licence [VERIFY]) only for minor UI, never sparkles.

## 7. Book animation

Decision recorded in D4b section 5.3: **no library; hand-rolled CSS 3D transforms with Pointer Events.** `react-pageflip` 2.0.3 and `page-flip` 2.0.7 were last published April 2021; `motion` 14.10.1 is active but adds weight for little gain. Zero new dependency, so nothing to pin.

## 8. Zod 4.6.7

- Zod 4 (peer-compatible with the Anthropic SDK's optional `^3.25 || ^4`).
- One `src/schemas/` module of shared schemas imported by browser forms and server handlers (the same file validates both sides).
- Validate at every boundary (brief 4.2): forms, route handlers, server actions, AI output (structured response parsed, never trusted), webhooks (after signature verification), environment variables at boot (`env.ts` fails fast if a variable is missing).
- Prefer `z.infer` types over hand-written duplicates. Zod 4 has native JSON Schema output (`z.toJSONSchema`) [VERIFY], so `zod-to-json-schema` is not needed.

## 9. Supabase: supabase-js 2.119.0, @supabase/ssr 0.12.7, CLI 3.0.0

- `@supabase/supabase-js` 2.119.0 (Node 22 or newer). A `3.0.0-next` exists; stay on the 2.x `latest`.
- `@supabase/ssr` 0.12.7: cookie-based sessions for the App Router. Still 0.x, so **pin exactly** and expect small breaking changes [VERIFY changelog at install]. Three client factories only: browser client, server client (reads cookies), and service-role client (server-only module, used only for webhooks and admin jobs).
- Supabase CLI package `supabase` 3.0.0 as a dev dependency (so the version is pinned in the lockfile, not a global install). Commands: `supabase start` (local stack, needs Docker), `supabase migration new`, `supabase db reset` (runs migrations and seed), `supabase test db`, `supabase gen types typescript --local > src/types/database.ts`. Regenerating types is part of every schema change; CI fails if `git diff` shows the generated file is stale.
- Note: the CLI 4.0 is in `next`; ignore.
- Auth cookies and "which client where" rules are set out in section 14.
- Generated types plus Zod: the database types describe rows; Zod schemas describe untrusted input; they are not merged.

## 10. Anthropic SDK 1.6.0

- `@anthropic-ai/sdk` 1.6.0 (published 2026-11-05) is a 1.x line. Server-only.
- Wrapped by one module `src/server/ai/` exposing `rewriteEntry(...)`: streaming, per-call timeout, bounded retries, usage (token and cost) logging, and **never** content logging. Model id comes from an env variable / config constant, chosen with the backend-lead and the `claude-api` skill reference at build time (do not hard-code from memory).
- Zod validates anything the model returns before saving.
- The prompt builder is a pure function covered by unit tests (brief 4.2).

## 11. Transcription SDK (provider decided in D5)

Name the seam now so the UI and tests do not depend on a vendor:

```ts
// src/server/transcription/provider.ts  (illustrative interface, not code to commit now)
interface TranscriptionProvider {
  readonly name: string;
  /** Server mints a short-lived, single-use credential so the browser can stream audio
   *  to the provider directly without ever seeing our long-lived API key. */
  createStreamingSession(input: { userId: string; languageHint?: string }):
    Promise<{ url: string; token: string; expiresAt: string; audioFormat: AudioFormatSpec }>;
  /** Fallback path: transcribe a finished recording uploaded in chunks. */
  transcribeBatch(input: { audio: ReadableStream<Uint8Array>; mimeType: string }):
    Promise<{ text: string; segments: Segment[] }>;
}
```
- The browser side has a matching `LiveTranscriber` interface (start, partial text events, final text, error, stop) with a fake implementation used by Playwright fixtures and unit tests.
- Candidate SDKs seen in the registry (verified existence/versions only, not a recommendation): `@deepgram/sdk` 5.18.0, `assemblyai` 4.42.0. Final choice, pricing, no-training terms and iOS Safari behaviour belong to D5/D9 and the backend-lead. Because most streaming vendors use a plain WebSocket, the browser may not need the vendor SDK at all (smaller bundle).

## 12. Stripe SDK 23.1.0

- `stripe` 23.1.0, server only (Node runtime route handlers). No `@stripe/stripe-js` on pages unless we use embedded elements; **Checkout and the Customer Portal are Stripe-hosted**, keeping card data out of our app and our PCI scope minimal. This also gives easy self-serve cancellation (brief 8.1 item 3).
- Webhook handler: read the **raw body** (`await request.text()`), verify the signature, check idempotency by event id, then update the subscription row in Supabase. Entitlement is read from Supabase only.
- Pin the Stripe API version in code; the SDK major tracks the API version [VERIFY mapping].

## 13. Testing

- **Vitest 5.1.0** (Vite 6.4/7/8 peer; Node 22.12 or newer). Unit + integration for pure logic (prompt building, entitlement, webhooks, date/day assignment, data-access rules). Component tests with `@testing-library/react` 16.3.3 and jsdom/happy-dom. Coverage via `@vitest/coverage-v8` (same version).
- **Playwright @playwright/test 1.65.0**. Golden path: sign up, record using **fixture audio** (Chromium flags `--use-fake-device-for-media-stream --use-file-for-fake-audio-capture` plus a fake `LiveTranscriber`), rewrite (mock AI server, or recorded fixture), page appears in the book. Projects: Chromium mobile (Pixel-class), WebKit (iPhone viewport), and desktop. WebKit in Playwright is not real iOS Safari, so launch still needs the real-device check (brief 8.3 item 20).
- **Reduced-motion tests** use `page.emulateMedia({ reducedMotion: 'reduce' })`.
- **Accessibility**: `@axe-core/playwright` 4.14.0 (Node 22.12 or newer) run on every key screen/state in E2E; fails the build on serious/critical violations. `vitest-axe` is 0.1.0 (immature), so skip.
- **RLS (security) test approach**: two layers.
  1. **pgTAP SQL tests** in `supabase/tests/` run by `supabase test db` against the local database: for every table, as user A you can select/insert/update/delete only your rows; as anon you get nothing; as user B you cannot see A's rows; plus a check that **every table in `public` has RLS enabled** (a query over `pg_tables`/`pg_class`) so a new table without RLS fails CI.
  2. **Vitest cross-user API test** using the real supabase-js client with two seeded users' JWTs against the local stack: A creates an entry, B's reads return empty, B's update/delete affect 0 rows, storage paths of A are denied to B. Runs in CI with a Supabase local stack (Docker is available on GitHub-hosted runners).
  Owned by backend-lead; frontend-lead only reviews that the app uses the user-scoped client.

## 14. Lint and format: ESLint + Prettier (chosen) vs Biome

Choice: **ESLint 10.14.0 (flat config) + Prettier 3.9.10**, with `typescript-eslint` 8.72.1, `eslint-config-next` 16.4.5 (includes React, hooks and `jsx-a11y` rules), `prettier-plugin-tailwindcss` for class ordering [VERIFY version].

Why not Biome 2.6.6 (faster, one tool, nothing to configure):
- Accessibility is a legal item here. `eslint-plugin-jsx-a11y` is the mature, complete rule set; Biome's a11y rules are a subset [VERIFY coverage].
- The official Next.js rule set (`@next/eslint-plugin-next`, React hooks rules incl. the newer React-compiler-related ones) is ESLint-only.
- Custom rules we want (ban `any`, ban importing server modules from client files, no raw hex colours) are easier in ESLint (`no-restricted-imports`, `no-restricted-syntax`).
- Costs of our choice: slower and more config. Mitigated by caching and by Lefthook running only on staged files. If speed becomes an issue, Biome can replace Prettier alone later.

Caveat: type-aware ESLint rules need a TypeScript version in typescript-eslint's supported range, which is why section 4 pins TS 6.0.3.

Pre-commit: **Lefthook 2.2.2** (a single Go binary distributed via npm, runs steps in parallel, no `.husky` shell indirection) rather than Husky 9.1.7 + lint-staged 17.6.0 (needs Node 22.22 or newer). Hook runs on staged files: Prettier check, ESLint, `tsc --noEmit` (whole project, only if TS files changed) and a secret scan (gitleaks [VERIFY] or the GitHub push protection). Hooks are a convenience; **CI is the authority** because hooks can be skipped.

## 15. Observability and analytics

- **Sentry: `@sentry/nextjs` 11.11.0** (supports Next 14, 15, 16). Content scrubbing is mandatory (brief 4.2, 8.1 item 7):
  - `sendDefaultPii: false`; no user email/name sent (use an opaque user id hash or none).
  - **No Session Replay** (it would record journal text). Not installed/enabled.
  - `beforeSend` and `beforeBreadcrumb` hooks strip request bodies, query strings, form values, and any field named like `transcript`, `entry`, `content`, `text`; breadcrumbs for console, fetch bodies and UI clicks with text are dropped; `maxValueLength` reduced.
  - Server routes never put journal text in error messages (errors carry ids, not content); a unit test throws a sample error containing a canary string and asserts it is removed.
  - Tunnel route and EU data region choice [VERIFY, backend-lead/PO decision]. Performance tracing sample rate low.
- **Analytics**: cookieless, no consent banner needed (brief 8.1 item 4: state the decision). Options: Vercel Web Analytics or Plausible; both privacy-first [VERIFY current cookie and data-processing terms]. Events are a short fixed whitelist of names ("recording_started", "entry_saved"); **no payload may contain text**; a typed `track()` wrapper only accepts enumerated event names and numeric/enum properties. Recommendation: Vercel Web Analytics (already a sub-processor, zero extra vendor) unless the PO wants stricter separation [open question].
- Logging: structured logger with a redaction allow-list; lint rule bans `console.log` in server code.

## 16. Quality gates in CI

| Gate | Tool | Notes |
|---|---|---|
| Install | `pnpm install --frozen-lockfile` | Lockfile must be in sync |
| Types | `tsc --noEmit` | Required |
| Lint/format | `eslint .`, `prettier --check .` | Required |
| Unit/integration | `vitest run` | With coverage report |
| DB tests | `supabase db reset` then `supabase test db` | RLS proof |
| Types in sync | regenerate and `git diff --exit-code` | Stale types fail |
| Build | `next build` | Required |
| E2E + axe | Playwright against built app | Golden path |
| Link check | **lychee** via its official GitHub Action [VERIFY] on built site/markdown | Not on npm as a usable package (registry shows `lychee` 0.2.12, unrelated). Internal links checked against the preview URL, external links with a retry allowance |
| Lighthouse | `@lhci/cli` 0.16.0 (Node 22.19 or newer) | Assertions: Performance, Accessibility, Best Practices, SEO each at least 0.9 on public pages |
| Bundle budget | `@next/bundle-analyzer` or size check | Book route budget per D4b |

Vercel "Ignored Build Step" skip for docs/`.github` only changes (brief 6): implemented with a `git diff` script in project settings; frontend-lead reviews.

## 17. PWA

Next.js supports a web app manifest via `app/manifest.ts`. For offline and the installed experience we need a service worker. Recommendation for v1: **a small hand-written service worker** (precache the app shell, cache-first for static assets, network-first for pages; never cache API responses containing journal text in the shared cache), because `serwist` 9.5.13 and `@serwist/next` 9.5.13 add a build plugin whose compatibility with Next 16/Turbopack is unconfirmed [VERIFY]. Journal data goes offline via IndexedDB (section 19), not the service worker cache. Decision can flip to Serwist if the hand-written worker grows beyond roughly 150 lines.

## 18. Next.js client/server boundary rules

These become part of CLAUDE.md (D15) and are enforced by lint where possible.

1. **Server Components by default.** Every file is a Server Component unless it needs state, effects, event handlers, or browser APIs.
2. **`"use client"` only for**: the recording screen (microphone, MediaRecorder, live transcript, wake lock, IndexedDB), the book (gestures, animation, keyboard), interactive forms and sheets. Put the directive in the **smallest leaf component**, not at a route root, so the rest stays server-rendered and the JS bundle stays small.
3. **Secret-holding modules import `server-only`** (`import "server-only"` as the first line; package `server-only` 0.0.1 is the official marker, builds fail if a Client Component imports it). Applies to: `src/server/ai/*`, `src/server/transcription/*` (key and token minting), `src/server/stripe/*`, `src/server/supabase/admin.ts` (service role), `src/server/env.ts` (secret variables). A mirror `client-only` guard is used on browser-API modules.
4. **Environment variables**: only names starting `NEXT_PUBLIC_` reach the browser. Secrets never use that prefix. Public vars limited to: Supabase URL, Supabase anon (publishable) key, analytics id, Sentry DSN.
5. **Direction of data**: Server Components fetch with the user-scoped Supabase server client (RLS applies) and pass **plain serializable props** (no class instances, no functions, no secrets) to client components.
6. **Mutations**: Server Actions or Route Handlers, each following the brief 4.3 order: **authenticate, authorize, validate with Zod**, then act. Same Zod schema is imported by the client form for instant feedback, but the server never trusts it.
7. **Route Handlers** needing raw bodies (Stripe webhook) run on the Node.js runtime, not Edge.
8. **Auth**: `@supabase/ssr` cookies refreshed in Next middleware/proxy (the Next 16 file convention for this changed names from `middleware` [VERIFY]); never trust `getSession()` on the server for authorization, use the verified user call [VERIFY current Supabase guidance].
9. **Long operations**: the AI rewrite streams from a Route Handler; function duration limits on the Vercel free tier are tracked in D5 [VERIFY]; the UI must survive a refresh mid-rewrite because the transcript is already saved.
10. **No journal content in** `console`, error messages, analytics, `generateMetadata` titles, URLs (the URL holds a date only, never text), or cache keys that leave the device.
11. **Caching**: pages with user data are dynamic (per-user, `private`, no shared CDN caching). Public marketing pages are static.
12. **Metadata**: unique `title` and description per route via the Metadata API; hub pages use generic titles ("Your book, March 14") without entry text.
13. **Directory convention**: `src/app` (routes), `src/components` (UI), `src/features/{book,recording,...}` (client features), `src/server` (server-only), `src/schemas`, `src/lib` (pure functions with tests). Layout finalised in D14.

## 19. Recording-client details

This is the highest-risk frontend area; findings below need real-device confirmation (**[VERIFY on device]** items in Phase 3).

### 19.1 Capturing audio
- `getUserMedia({ audio: { echoCancellation: true, noiseSuppression: true, autoGainControl: true, channelCount: 1 } })`. Requires HTTPS and a **user gesture** (the Start tap) on iOS Safari. Ask only on first Start with a short explanation screen first (brief 8.1 item 5).
- Handle `NotAllowedError` (blocked: show OS-specific recovery steps), `NotFoundError`, `NotReadableError` (another app using the mic), and device changes (`devicechange`, track `ended`).

### 19.2 Recording formats: MediaRecorder vs AudioWorklet
- **MediaRecorder** is the simple path. Safari's MediaRecorder (macOS since 14.1, iOS since 14.3 per my recollection) produces **`audio/mp4` (AAC)** and not WebM/Opus on iOS; Chrome/Android produce `audio/webm;codecs=opus`. Always detect with `MediaRecorder.isTypeSupported()` in a priority list (`audio/webm;codecs=opus`, `audio/mp4`, `audio/ogg;codecs=opus`) and send the actual `mimeType` along with the data. Chunks via `start(timeslice)` (e.g. 1000 ms). Safari-specific quirks (the first chunk header, `ondataavailable` on stop) need device testing [VERIFY].
- **AudioWorklet** gives raw PCM, which is the format most streaming speech APIs accept (16-bit, 16 kHz or 24 kHz, mono) with no codec dependency. It works in iOS Safari 14.5 and newer [VERIFY]. It needs a separate worklet file served as a static module, resampling (the AudioContext sample rate on iOS is usually 44.1 or 48 kHz, so we downsample), and an `AudioContext` that must be created/resumed inside the user gesture.
- Recommendation: **AudioWorklet to PCM as the primary path for live streaming** (works identically on iOS Safari, Android Chrome, desktop and is what cloud streaming APIs expect), and **MediaRecorder as the fallback and as the "chunked upload" path** for the batch fallback in D5. Final choice follows the D5 transcription decision. Raw audio is not retained after transcription (brief 8.1 item 6); chunks held locally for crash recovery are deleted once the transcript is saved.
- **Browser Web Speech API** (`SpeechRecognition`): not the primary option: behaviour differs between browsers, Chrome sends audio to Google, iOS Safari support is inconsistent, and it provides no privacy guarantee we can state in the app [VERIFY]. D5 decides; if used, it must be behind the same `LiveTranscriber` interface.

### 19.3 Durability (brief 8.3 item 19)
- Persist **while recording**, not at the end. Every final transcript segment (and every few seconds of partial text) is written to **IndexedDB** with the session id, sequence number, timestamp and the user's local date. Use a tiny wrapper (`idb` 8.0.4, or raw IndexedDB) and a single store `recording_sessions`.
- Optionally also store audio chunks (blob per second) so the transcript can be regenerated if the live stream drops; gated by size and deleted after success.
- On app start, a **recovery check** looks for sessions without a "completed" flag and offers "Continue / Save what you said / Discard" with the recovered text shown.
- Request persistent storage: `navigator.storage.persist()` (best effort) because Safari can evict storage under pressure for non-installed sites (installed PWA is treated better) [VERIFY]. Show storage-failure errors explicitly rather than failing silently.
- Upload queue: when online, transcript is saved to Supabase as a draft (idempotent by session id) every N seconds, so another device can recover it. When offline: "Saved on this device, will sync" and a retry with backoff on reconnect (`online` event, plus `visibilitychange`).
- `pagehide` / `visibilitychange` (not `beforeunload`, unreliable on iOS) trigger an immediate flush.
- Network drop mid-stream: reconnect the streaming session with the sequence number; segments are de-duplicated on merge. Test with Playwright offline mode and a killed-tab test.
- Journal text in IndexedDB is on-device data; cleared on sign out / account delete. (Encryption at rest on device is a D8 question.)

### 19.4 Screen Wake Lock
- `navigator.wakeLock.request('screen')` keeps the screen on during recording. Supported in Safari 16.4 and newer, including iOS [VERIFY], and Chrome. It is released automatically when the tab is hidden, so **re-acquire on `visibilitychange`** while recording. Wrap in feature detection and try/catch (it can reject on low battery); if unavailable, show a quiet hint "Keep your screen on while recording".
- The big limitation: **iOS Safari suspends microphone capture and JS when the screen locks or the user switches apps**, wake lock or not. The UI therefore warns, flushes to IndexedDB on `pagehide`, and offers recovery. Backgrounded recording is a known web limit and an argument for a later native app [raise with PO].
- Also: a visible recording indicator (timer + pulsing glyph that is static under reduced motion), a `role="status"` live region announcing "Recording started / stopped" and a **separate, throttled** region for transcript text (screen readers must not read every partial word; offer an option to hear only on pause) (brief 8.1 item 20).
- Start/End: one large toggle button, accessible name changes ("Start recording" / "End recording"), keyboard operable, confirm before End if under a few seconds.

### 19.5 Testing the recorder
Unit-test the state machine (idle, requesting, recording, paused, saving, recovering, error) as a pure reducer; Playwright with fake media device and fake transcriber; manual real-device matrix: iPhone Safari (installed PWA and browser tab), Android Chrome, desktop Chrome/Safari/Firefox; scenarios: permission denied, phone call interrupts, screen lock, tab close, airplane mode, 20-minute recording.

## 20. Risks specific to this toolchain

| Risk | Mitigation |
|---|---|
| TypeScript 7 / typescript-eslint mismatch | Pin TS 6.0.3; revisit when supported |
| Very new majors (pnpm 12, Vitest 5, ESLint 10, Anthropic SDK 1.x, supabase CLI 3) | Exact pins, Renovate/Dependabot weekly PRs, CI as gate |
| `@supabase/ssr` is 0.x | Pin exactly, wrap in our own 3-function module |
| iOS Safari audio/background limits | AudioWorklet+IndexedDB durability, real-device tests, PO told early |
| Docs sites unreachable in this session | All doc-dependent claims marked [VERIFY] |

## 21. Open questions for the PO

1. Accept **Node 24 now, Node 26 later** (recommended), or start on 26?
2. Accept **TypeScript 6.0.3 pinned** until the lint tooling supports 7 (recommended), or prefer Biome with TS 7 and fewer accessibility lint rules?
3. Analytics: **Vercel Web Analytics** (recommended, no new vendor) or **Plausible** (separate vendor, stricter separation)?
4. Sentry data region (EU or US) and whether the PO wants any error tracking at all in the free tier.
5. Real devices available for testing (which iPhone model/iOS and which Android)? Affects the 60 fps and recorder checks.

## 22. [VERIFY] list

- Vercel support for Node 24 and 26; Next 16 `next lint` removal, Turbopack default, `middleware` rename; TypeScript 7 support in Next and typescript-eslint.
- pnpm 12 default script blocking; `@supabase/ssr` current cookie/auth guidance; Zod 4 JSON Schema output; lychee action version; Vercel Web Analytics/Plausible terms.
- Safari: MediaRecorder container, AudioWorklet and Wake Lock versions and quirks; IndexedDB eviction behaviour.
- Bundle size numbers for the book route (measure in Phase 3).
