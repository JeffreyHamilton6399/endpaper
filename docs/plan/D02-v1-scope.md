# D2. v1 scope

**Status:** Draft for PO review · **Owner:** Head Manager

Rule of thumb: if it is not needed for "record tonight, read it back, keep paying", it is not v1.

## In scope (v1)
**Accounts and trust**
- Email sign-up and sign-in (magic link or one-time code; final method in D5), with an unticked Terms/Privacy consent box, a plain vendor disclosure, and age confirmation (13+, 16+ where required).
- Legal pages: Privacy Policy, Terms of Service, Refund and Cancellation Policy (drafts marked *requires legal review*). Cookie banner only if non-essential cookies are used (plan: none).
- Help page with crisis resources and a not-therapy statement.

**Onboarding**
- Paste one or more writing samples (with guidance on what works), or skip and add later.
- Time zone captured automatically and editable.

**Nightly recording**
- Start / End, live transcript, microphone permission explained before the browser prompt.
- In-progress transcript persisted locally and recovered after a crash, tab close or disconnect.
- Clear states for microphone blocked, offline and provider failure.
- Raw audio is not stored after transcription.

**Rewrite**
- Server-side Claude rewrite in the user's style, with a crafted "writing your page" state.
- Faithfulness safeguards and handling for very short, empty and very long input (D7).
- Retry if the rewrite fails; the transcript is never lost.

**The book**
- Book hub: cover, opens to today, chapters by month and year, jump to a date.
- Page-turn animation (single page on phones, two-page spread on wide screens), reduced-motion alternative, keyboard operation.
- Entry view: rewrite as the page, original transcript one tap away, both editable, revision history kept.
- Entry date can be corrected (for after-midnight recordings).

**Paying**
- 14-day trial, then monthly or yearly via Stripe Checkout; trial-ending reminder email; self-serve cancel via the Stripe Customer Portal; entitlement read from Supabase.
- Trial-ended state: user can still read and export everything, but cannot record new entries until they subscribe.

**Settings**
- Account (email, time zone), writing samples (view, edit, add, delete), reminder time preference stored (see note), export (JSON, plain text, PDF), permanent account deletion.

**Marketing**
- Landing page with real product visuals, price, start trial; footer with legal links and contact.

**Platform**
- Installable PWA, mobile-first, works on iOS Safari and Android Chrome; desktop supported.

**Note on reminders.** The brief lists "reminder time" in Settings but "reminders" as deferred. v1 stores the preferred time only and shows "Reminders are coming soon" honestly; sending reminders (push or email) ships in phase v1.1. Logged as an open question in D3/D22.

## Out of scope (v1), planned later (D10)
| Feature | Phase |
|---|---|
| Reminders (push/email) and streaks | v1.1 |
| "On this day" memories | v1.2 |
| Weekly / monthly recaps and insights | v1.2 |
| Photos on pages | v1.3 |
| Printed book | v2 |
| Native iOS / Android apps | v2 |
| Social sign-in (Apple/Google) if not chosen for v1 in D5 | v1.1 |
| Multiple journals, sharing, collaboration | Not planned |
| Languages other than English for the rewrite | Later; transcription may work, but the rewrite and evaluation are English-only in v1 |
| Audio playback of entries (requires retaining audio) | Not planned for v1; contradicts the data-minimisation default |
| Free tier after the trial | Not planned; revisit with conversion data |
| Guided prompts, mood tracking, therapy-like features | Not planned |
