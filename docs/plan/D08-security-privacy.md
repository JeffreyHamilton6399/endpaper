# D8. Security and privacy design

**Status:** Draft for PO review · **Owner:** Backend Lead · **Reviewer audit:** required
**Verification note:** vendor policies below come from search summaries, not the vendors' own pages (the session could not open them). Every vendor commitment is **[VERIFY]** against the current contract/terms before it is repeated in the Privacy Policy. This is an engineering design, not legal advice; the legal documents require legal review.

A journal is among the most sensitive data a person can hold. The design goal is simple to state: **only the user can read their words; we can prove it, and we say nothing the system does not do.**

## 1. Encryption in transit
- All traffic uses HTTPS (TLS 1.2+) with HSTS (`max-age` 1 year, `includeSubDomains`). Vercel provides certificates automatically.
- Browser to transcription provider uses secure WebSocket (`wss://`). Server to Anthropic, Stripe and Supabase uses HTTPS.
- Security headers on every response: `Content-Security-Policy` (nonce-based, no `unsafe-inline` scripts, `connect-src` limited to our origin, Supabase, and the STT host), `X-Content-Type-Options`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` (microphone for self only), `frame-ancestors 'none'`.
- Cookies: `Secure`, `HttpOnly`, `SameSite=Lax` for the session.

## 2. Encryption at rest (platform level)
- Supabase encrypts database storage and backups at rest (AES-256 on the underlying cloud storage, **[VERIFY]** on Supabase's security page). Vercel encrypts stored environment variables and logs at rest **[VERIFY]**.
- This protects against stolen disks. It does **not** protect against someone with valid database access (a leaked service key, a misconfigured policy, a vendor insider): the database would decrypt data for them. That gap is what section 3 addresses.

## 3. Application-level encryption vs server-side AI

### 3.1 Options
| Option | Who can read journal text | Works with server-side AI rewrite? | Cost / consequences |
|---|---|---|---|
| **A. Platform encryption only** | Anyone with DB access or a leaked DB dump/backup | Yes, trivially | Simplest. A single database leak exposes every journal in readable form. |
| **B. Envelope encryption on the server (recommended)**: each user has their own random data key (DEK); content columns hold ciphertext; DEKs are encrypted ("wrapped") by a master key (KEK) kept outside the database. The server decrypts only while serving that user's request. | Only our running server, holding the master key, can decrypt. A DB dump, backup, or a Supabase staff member sees ciphertext. | Yes. The server decrypts transcript + samples, sends them to Claude over TLS, encrypts the result. | Moderate engineering. No server-side search over text (deferred anyway). Honest limit: if our server or the master key is compromised, data is exposed. Crypto-shredding becomes possible (section 8). |
| **C. True end-to-end encryption (E2EE)**: key derived from a user passphrase or device; server never sees plaintext. | Only the user | **No**: the AI rewrite needs the plaintext on a server, so E2EE would contradict the core product, unless the rewrite ran in the browser/on device, which is not possible with the Claude API without sending plaintext. | Lost passphrase means lost journal; no support recovery; breaks sync across devices without extra design. |

### 3.2 Recommendation
**Adopt Option B for v1**, and state it precisely and honestly. The product text may say "Encrypted in transit and at rest, and your journal text is additionally encrypted with a key unique to you" and **must not say** "end-to-end encrypted", "zero knowledge" or "we can't read your journal": our server must decrypt to run the AI, and our operators could in principle read it, so the claim would be false (brief §8.1 item 14).

Fallback if the PO wants less scope: Option A for the first private beta, with the schema already holding `bytea` columns so B can be switched on before real users. The schema in D6 is written for B; moving from A to B later requires re-encrypting data, so doing it from day one is cheaper.

Revisit Option C only if the PO later wants a premium "private mode" with on-device AI, which is a different product.

### 3.3 Design of Option B (Backend Lead writes this; never delegated to Haiku)
- **Cipher:** AES-256-GCM using Node's built-in `crypto` (no new dependency). Fresh random 96-bit nonce per value. Stored blob: `version byte | nonce | ciphertext | auth tag`. The row's `user_id` and column name are bound in as **additional authenticated data (AAD)**, so ciphertext copied from one row/user to another fails to decrypt.
- **Keys:** per-user DEK (32 random bytes) generated at onboarding, stored in `profiles.dek_wrapped`, wrapped with the master key (KEK) using AES-256-GCM (or AES key wrap). `kek_version` allows rotation: add a new KEK, re-wrap DEKs (cheap; content is not re-encrypted).
- **Where the KEK lives:** v1: a Vercel encrypted environment variable (production only; preview and local use different keys), unreadable by the browser and never in the database. **Upgrade path:** a managed KMS (AWS KMS / GCP KMS, **[VERIFY]** cost, about $1 per key per month) so the KEK never exists in app memory in plaintext. Recommended before 1,000 users, not before beta.
- **What stays plaintext:** dates, status, counts, model names, timezone. Needed for the book and cost tracking, and they reveal "when you journal" but not "what".
- **Decrypt path:** the request handler, after authentication/authorization, unwraps the user's DEK, decrypts, and returns data. The decrypted text exists only in memory for that request and is never logged.
- **Testing:** unit tests for round-trip, tampering (flipped bit fails), wrong-user AAD (fails), key rotation, nonce uniqueness; Reviewer audit mandatory.
- **Residual risks accepted:** compromise of the running server + KEK; a malicious deploy by someone with Vercel/GitHub access. Mitigated by 2FA on all vendor accounts, branch protection, minimal admin access (section 9).

## 4. Vendor data-flow table

| Vendor | What it receives | Journal content? | Purpose | Region / notes | Training and retention commitments |
|---|---|---|---|---|---|
| **Supabase** (database, auth, storage) | Email, auth metadata, profile, **ciphertext** of transcripts/rewrites/samples, dates, usage rows | Only encrypted (Option B) | Storage, sign-in | Choose region near users (EU vs US decision; **[VERIFY]** options). Backups per plan. | Not an AI company; no training. Data processing agreement (DPA) available **[VERIFY]**. |
| **Vercel** (hosting, functions) | All HTTP requests; function memory handles decrypted text transiently; request/response metadata in logs | Transiently in memory; never in logs by our rules | Runs the app | Function region configurable (match Supabase) **[VERIFY]** | No training on content. Log retention depends on plan **[VERIFY]**; we log no content. DPA available **[VERIFY]**. |
| **Anthropic** (Claude API) | Transcript, writing samples, entry date/context for each rewrite | **Yes, plaintext (necessary)** | The rewrite | Processed in Anthropic's infrastructure; **[VERIFY]** regions | Commercial terms: customer content is **not used to train models** (several sources cite the Commercial Terms effective June 2025; **[VERIFY]** on the current terms and DPA). API input/output retention for abuse monitoring is reported as 30 days, with a reduction to 7 days reported by one source; **[VERIFY]**. Zero-data-retention is available to qualifying customers by agreement; evaluate once at scale **[VERIFY]**. |
| **Transcription provider** (Deepgram primary, AssemblyAI alternate) | Live audio of the user's voice, streamed straight from the browser | **Yes, audio** | Speech to text | **[VERIFY]** regions | Must have: do-not-train / do-not-retain setting on. Deepgram has a Model Improvement Program with an opt-out request parameter **[VERIFY]**; AssemblyAI states it does not train on customer data by default **[VERIFY]**. Chosen only after confirming in contract/DPA. Parameter set server-side. |
| **Stripe** | Email, name if given, payment details (entered at Stripe, never us), plan, billing country | **No** | Billing | Stripe is an independent controller for payment data **[VERIFY]** | Retains payment records as legally required, even after account deletion (privacy policy must say so). |
| **Sentry** (errors) | Stack traces, route names, release, anonymous or hashed user ID, browser/OS | **No** (scrubbed, section 6) | Find bugs | Choose EU/US region **[VERIFY]**. Session Replay **off**. | Retention configurable (set 30-90 days); no training. DPA **[VERIFY]**. |
| **Analytics** (cookieless, e.g. Plausible) | Page paths (templated, e.g. `/book/[date]` not the date itself if sensitive), referrer, coarse device, country | **No** | Product metrics | EU-hosted options exist **[VERIFY]** | No cookies, no personal IDs. Custom events carry names only (`entry_saved`), never text, titles, or sample content. |
| **Email provider** (Resend / Postmark; **[VERIFY]** choice) | Email address, message text we compose (login codes, trial reminder, receipts) | **No.** Reminder and notification emails are generic and never quote entries. | Transactional email | **[VERIFY]** | Message-content retention is configurable (some store bodies for days); keep minimal. Marketing email (if any later) uses a separate stream/vendor and an opt-in list. |
| **GitHub / CI** | Code, test fixtures (fake data only) | **No real user data ever in the repo or CI** | Source control | n/a | Fixtures are synthetic; a CI check blocks files that look like real exports. |

Rules that follow:
- This table is the source for the Privacy Policy's sub-processor list and the in-app disclosure ("audio goes to our transcription provider; text goes to our AI provider"). Any new vendor requires updating this table, the policy, and PO approval first.
- Signed DPAs are collected for each vendor before beta (checklist item in D22).
- Prefer the provider's EU region if the user base is UK/EU **[VERIFY]**; international transfer wording (SCCs/UK addendum) is a legal-review item.

## 5. Access control and accounts
- **Customers:** RLS (D6) plus per-user encryption. All service-role code paths are listed in a single file and reviewed.
- **Team/operators:** 2FA required on GitHub, Vercel, Supabase, Stripe, Anthropic, STT, Sentry, and the email provider. Least privilege: the PO is the only production admin in v1. No one reads user content for support; support tools show metadata only. Any break-glass access is written to a log and disclosed in the policy.
- **Agents/CI:** Claude agents never receive production secrets or production data. CI uses test-mode keys and synthetic data only.

## 6. Logging rules (no content, ever)
1. **Allowed in logs:** request ID, route name, HTTP status, duration, user ID (random UUID), entry ID, model name, token counts, cost, provider request IDs, error codes.
2. **Forbidden anywhere (logs, traces, Sentry, analytics, emails, URLs, query strings, error messages):** transcripts, rewrites, writing samples, entry text, labels, titles, search terms, prompts, model outputs, audio.
3. **Mechanisms, not promises:**
   - One logger module with a typed, allow-listed field set. Direct `console.*` is banned in server code by ESLint.
   - Request bodies are never logged. Zod errors are mapped to field names and codes before logging (Zod can echo received values).
   - Sentry: `sendDefaultPii: false`, **Session Replay disabled**, breadcrumbs for fetch/XHR bodies off, a `beforeSend` hook that drops request bodies, form values, and any string above a short length from `extra`; unit-tested.
   - Vercel: no request body logging; third-party log drains not enabled in v1.
   - Client: Next.js errors shown to the user are generic; stack traces never include state.
   - URLs carry IDs and dates only, never text.
4. **Canary test (CI, end-to-end):** a transcript containing a unique string (`CANARY-7f3a...`) is recorded, rewritten (mocked AI), exported and deleted; the test then searches all captured server logs, the Sentry test transport payloads and analytics calls for the string and fails if found. The test also forces an error in each route to verify error reports are clean.
5. Log retention: application logs 30 days; error events 90 days; usage rows 13 months.

## 7. Retention schedule

| Data | Kept | Then |
|---|---|---|
| Entries, transcripts, revisions, samples | While the account exists | Deleted on user delete, or on account deletion |
| Soft-deleted entries (Trash) | 30 days | Hard-deleted by nightly job |
| Raw audio | Not retained by us. Device-only safety copy until entry saved (max 48 h). Provider retention: none, opt-out confirmed **[VERIFY]** | n/a |
| Account (email, profile) | While active | Deleted at account deletion |
| Subscription mirror | While account exists | Deleted with account |
| Stripe records (invoices, customer) | Held by Stripe for legal/tax periods (typically years, **[VERIFY]**) | Outside our control; disclosed |
| `stripe_events` | 90 days | Purged |
| `usage_events` | 13 months | Aggregated (no user ID), then purged |
| Application logs / Sentry | 30 / 90 days | Auto-expire |
| Supabase backups | Plan-dependent, about 7 days on Pro **[VERIFY]** | Roll off; with crypto-shredding, deleted users' backup rows become unreadable immediately |
| Anthropic API logs | Provider-defined (7 or 30 days **[VERIFY]**) | Provider deletes |
| Auth/login codes | 10 minutes | Expire |
| Inactive accounts | Proposal: email warning at 24 months of no sign-in, delete at 27 months (PO decision; excluded from v1 build, noted for the policy) | |

## 8. Export and account deletion

### 8.1 Export (self-serve, Settings)
- Requires a fresh sign-in (code within 10 minutes); rate limited (3 per day).
- **JSON** (complete and machine-readable): profile (no secrets), samples, entries with transcript, rewrite, dates, timezone, and revision history, plus a schema version. **Text/Markdown**: one file per month or a single file, rewrite followed by original transcript. **PDF**: a printable book, generated **in the browser** from the data the server returns, so the Vercel function does not need to render a large PDF within time limits.
- Small journals (< ~5 MB): streamed directly as a download. Large ones: the server builds a ZIP into the private `exports` bucket, emails a link (sign-in required, 24-hour signed URL), and deletes the file after 7 days.
- Export works at all times, including after trial end or cancellation (brief §2 scope).
- Decryption happens server-side during export; the file the user downloads is plaintext, and the UI says so ("keep this file somewhere safe").

### 8.2 Account deletion
1. Settings, Delete account. Offer export first. Require a fresh code and typing the word DELETE.
2. **Immediately:** sign out all sessions; cancel the Stripe subscription (no further charge; no refund logic beyond policy); set `deletion_requested_at`; the account is hidden and cannot be used.
3. **7-day undo window** (**PO decision**): the user can cancel the deletion by signing in. An email confirms the request and the date.
4. **After 7 days (or immediately if the user chooses "delete now"):** a job (service role, fully logged without content) deletes `auth.users` row, which cascades through profile, samples, entries, revisions, subscription mirror, usage rows; deletes any Storage objects under `{user_id}/` and exports; deletes the Stripe customer record via API where Stripe permits (invoices retained by Stripe as required); removes the user from the email provider's lists; writes a minimal tombstone (hashed ID, date) so a restore from backup does not resurrect the account (see below).
5. **Backups:** the deleted user's rows remain in Supabase backups until they roll off (about 7 days on Pro **[VERIFY]**). Two controls: (a) **crypto-shredding**: deleting the wrapped DEK makes any backup copy of that user's content permanently unreadable, effective at once; (b) a restore runbook that re-applies the tombstone list immediately after any restore. The policy states: "deleted from live systems within 7 days; backups expire within 30 days; stored content in backups is unreadable once your key is destroyed."
6. A test creates a user with every kind of data, deletes the account, and asserts zero rows by user ID in all tables and buckets (D6 isolation suite).

## 9. Threat model: top risks

| # | Risk | Likelihood | Impact | Mitigations | Residual |
|---|---|---|---|---|---|
| 1 | **Account takeover** (stolen email, phished code, SIM-swapped inbox) | Medium | High (full journal readable) | Passwordless codes: 6 digits, 10-minute expiry, 5 attempts then lock, rate limits per email and IP; Google sign-in option; fresh code for export/delete/email change; email notice on new-device sign-in and on email change; sessions revocable ("sign out everywhere"); optional TOTP/passkeys in a later release (recommend passkeys as v1.1). | Compromised email still wins; documented. |
| 2 | **RLS or authorization bug** exposes another user's data | Medium (common error class) | Critical | Deny-by-default, forced RLS, user-scoped DB clients, composite FKs, immutable `user_id`, structural CI checks, two-user test matrix on every migration, Reviewer re-run, per-user encryption with AAD so leaked rows are ciphertext, plus 404 instead of 403. | Low |
| 3 | **Prompt injection through spoken text** ("ignore your instructions and ...") or pasted samples | High that it will be tried (often by accident) | Low-Medium: a model with no tools and no cross-user data can only distort the user's own entry | Content is wrapped as data; system prompt declares it; model has **no tools, no browsing, no memory**; only that user's own samples are in context; output is schema-validated plain text and length-checked against the transcript (flags an output that is far shorter/longer or contains instruction-like meta text); output rendered as text only; injection cases in the D7 test set. | Occasional odd rewrite; user can regenerate or edit. |
| 4 | **Leaked keys** (Stripe, Supabase service role, Anthropic, STT, KEK) | Medium | High | Secrets only in per-environment stores; separate keys per environment; push protection + gitleaks; `NEXT_PUBLIC_` allow-list test; restricted Stripe keys; Anthropic spend cap and alerts; service key limited to listed modules; rotation runbook; 2FA; no secrets in agent sessions or CI logs. KEK leak: rotate and re-wrap; KMS upgrade path. | Medium until KMS |
| 5 | **XSS in entry rendering** (a transcript or sample containing `<script>` or HTML; stored XSS would run in a user's session and read their journal) | Medium | High | Entries rendered through React as text (no `dangerouslySetInnerHTML`; lint rule bans it); if rich formatting arrives, a strict sanitizer with allow-list; nonce-based CSP with no inline script; remote images blocked in entries (`img-src 'self' data:`) so a model cannot exfiltrate through image URLs; exports escape content (PDF/text/JSON), and Markdown export is plain. Tests include HTML/script payloads in every text field. | Low |
| 6 | **Cost abuse / denial of wallet** (token endpoint or rewrite hammered) | Medium | Medium | Auth + entitlement + per-user daily caps from `usage_events`, one STT session per user, 60-s token lifetime, Anthropic/STT spend caps, global kill switch (env flag). | Low |
| 7 | **Webhook forgery or replay** | Low | High (free subscriptions) | Signature verification on raw body, idempotency table, re-fetch from Stripe, entitlement only from the Supabase mirror. | Low |
| 8 | **CSRF on state-changing routes** | Low | Medium | `SameSite=Lax` cookies, `Origin` header check in the shared route wrapper, JSON-only bodies, bearer-token path unaffected. | Low |
| 9 | **Supply-chain compromise** (npm package, agent-installed plugin) | Medium | High | Pinned lockfile, Dependabot, minimal dependencies (crypto uses Node built-ins), CI permissions read-only by default, no dependency added without approval (brief §5.5), production secrets never in CI or agent sessions. | Medium |
| 10 | **Vendor breach or policy change** (STT/AI retention, training terms) | Low-Medium | High | Documented vendor table, opt-out settings enforced server-side, provider-agnostic interfaces, re-verification at each release. | Low-Medium |
| 11 | **Over-collection / minors** | Low | Legal | Age confirmation, minimal data, no raw audio, no analytics content. | Low |

## 10. Security work items for the task list (D16)
All owned by the Backend Lead or Head Manager (never Haiku): encryption module + tests; RLS migrations + isolation suite; shared route wrapper (authn, authz, Zod, origin check, rate limit); Stripe webhook handler; logger + Sentry scrubbing + canary test; export and deletion jobs; headers/CSP; secrets/CI hardening; KMS migration (post-beta). Haiku coders may implement UI for settings screens against the finished endpoints.

## 11. Open items for legal review (not engineering decisions)
Controller/processor roles and DPAs, lawful basis for processing special-category data (journals can reveal health or beliefs, GDPR Art. 9, so explicit consent wording at signup), UK/EU representative needs, international transfers, children's data rules by country, data-breach notification process (72 hours under GDPR), cookie decision recorded as "no non-essential cookies".
