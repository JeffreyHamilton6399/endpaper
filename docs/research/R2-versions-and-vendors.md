# R2. Research: versions, vendor pricing and limits

**Agent:** researcher (Haiku), run 2026-11-07; npm versions re-verified by the Head Manager from raw registry JSON (`curl https://registry.npmjs.org/<pkg>/latest`) the same day. Vendor sites marked [VERIFY] were unreachable from this session (network allowlist); figures come from search snippets or third parties.

## Versions (verified 2026-11-07 from the npm registry)
| Package | Latest |
|---|---|
| next | 16.4.5 |
| react | 19.3.1 |
| typescript | 7.0.2 |
| tailwindcss | 4.3.5 |
| zod | 4.6.7 |
| @supabase/supabase-js | 2.119.0 |
| @supabase/ssr | 0.12.7 |
| supabase (CLI) | 3.0.0 |
| stripe | 23.1.0 |
| @anthropic-ai/sdk | 1.6.0 |
| vitest | 5.1.0 |
| @playwright/test | 1.65.0 |
| @biomejs/biome | 2.6.6 |
| eslint / prettier | 10.14.0 / 3.9.10 |
| @sentry/nextjs | 11.11.0 |
| pnpm | 12.18.0 |
| motion | 14.10.1 |
| page-flip / react-pageflip | 2.0.7 / 2.0.3 |
| Node.js LTS | **v26.13.0 "Lithium"** (2026-10-28), from https://nodejs.org/dist/index.json |

## Speech recognition
- **Web Speech API**: listed as supported on iOS Safari 14.5+ and Chrome for Android; not Firefox Android (caniuse via search snippet) [VERIFY]. Where audio is processed, continuous-mode behaviour, and silence timeouts were **not confirmed** (MDN unreachable). Treat as unreliable as the primary engine for a core feature until tested on real devices.
- **Streaming STT vendors** (all [VERIFY] on official pricing pages):

| Vendor | Streaming price (third-party figure) | Browser short-lived token |
|---|---|---|
| AssemblyAI Universal-Streaming | ~$0.15/hr; billed for full socket time incl. idle | Reported to exist [VERIFY] |
| Deepgram Nova-3 | ~$0.0048–0.0092/min | Not confirmed |
| Speechmatics | ~$0.20–0.43/hr | Not confirmed |
| OpenAI realtime transcription | ~$0.017/min (realtime-whisper); gpt-4o-transcribe ~$0.006/min | Ephemeral client secrets (documented pattern) [VERIFY] |
| Google Cloud STT v2 | ~$0.016/min | OAuth / service accounts; no browser token confirmed |

Data-retention and no-training policies for all speech vendors: **not confirmed** [VERIFY before vendor selection].

## Anthropic (official page fetched: https://platform.claude.com/docs/en/about-claude/pricing)
| Model | Input / output per million tokens |
|---|---|
| Haiku 5.5 (≤100K prompt) | $0.10 / $0.50 |
| Sonnet 5.5 | $2 / $10 |
| Opus 5.5 | $4 / $20 |
| Fable 5.1 | $10 / $50 |
Cache reads ≈ 0.1× input; 5-minute cache writes 1.25× input.
Training: Commercial Terms §B: "Anthropic may not train models on Customer Content from Services." https://www.anthropic.com/legal/commercial-terms

## Hosting and payments (all [VERIFY])
- **Vercel Hobby is for non-commercial personal use only**; a paid product needs Pro (~$20/user/month). Function max duration: 300 s default; higher configurable (sources conflict).
- **Supabase Free**: 500 MB database, pauses after ~7 days of low activity. **Pro** ~$25/month, no pausing, daily backups; PITR is an add-on (price not confirmed).
- **Stripe**: ~2.9% + $0.30 per US card charge; Billing ~0.7% of billing volume. Customer Portal supports self-serve cancellation (incl. cancel at period end).
- **Cookieless analytics**: Vercel Web Analytics (no cookies; included quota on paid plans), Plausible (~$9/mo), Fathom (~$14/mo).

## Security note
The researcher reported that one search result contained an embedded instruction to use Bash for file edits. It was ignored (the researcher is read-only). Logged in `docs/oversight-log.md` as an observed injection attempt in web content.
