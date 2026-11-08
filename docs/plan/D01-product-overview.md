# D1. Product overview

**Status:** Draft for PO review · **Owner:** Head Manager · **Phase:** 1

## Pitch
Endpaper is a journal you talk to. At night you tap Start and tell it about your day. Your words appear as you speak. When you tap End, Endpaper turns the ramble into a clean journal entry **that still sounds like you**: your words, your jokes, your rhythm, without the "um"s and the loops. Each day becomes a page in a book you can open and flip through, chapter by chapter, for years.

One line: **"Talk about your day. Read it back in your own voice."**

## Who it is for
| Segment | Why they come | Why they stay |
|---|---|---|
| **Lapsed journalers** (primary). People who have bought notebooks and journaling apps and stopped within weeks because writing takes effort at the end of a tiring day. | Talking for five minutes is easier than writing for twenty. | The book fills up; missing a night feels like a gap on the shelf. |
| **Voice-note people.** Already send long voice memos and dictate texts. | It is the habit they already have, pointed at themselves. | Entries read better than their raw dictation ever did. |
| **The PO's own pattern.** Dictate, paste into a chatbot, ask it to tidy up, paste into a notes app. | Endpaper does those four steps in one tap, keeps the original, and files it by date. | It stops sounding like a chatbot after a few entries, because it learns from their samples. |

Not for v1: therapy-style guided journaling, teams, shared journals, children under 13.

## Core loop (v1)
1. Open the app at night, tap **Start**.
2. Speak. A live transcript appears.
3. Tap **End**. The transcript is saved immediately (locally first, then to the server).
4. The AI rewrites it in the user's style, using their writing samples. A "writing your page" state shows while it works.
5. The entry becomes today's page in the book. The original transcript is one tap away. Both are editable.
6. The book hub opens to today; the user can turn pages or jump to a chapter (month/year) or date.

## Differentiator
"It sounds like you, not like AI." Competitors transcribe, summarise or prompt; most of them produce text with a recognisable assistant voice [VERIFY: competitor scan in D11]. Endpaper's rewrite is constrained to the user's vocabulary and rhythm, and it is **faithful**: it never adds events or feelings that weren't said and never drops a meaningful detail (see D7). The book presentation is the second differentiator: the archive is a pleasure to open, not a list of notes.

## Success metrics
Definitions are precise so they can be measured from day one without reading any journal content (counts and timestamps only; see D8).

| Metric | Definition | v1 target (beta, revisit after data) |
|---|---|---|
| **Activation** | % of signups who complete one recorded entry within 48 hours of signup | ≥ 60% |
| **Second-night return** | % of activated users who record again within 3 days | ≥ 50% |
| **Nightly retention (W4)** | % of activated users who record on ≥ 4 of 7 nights in their 4th week | ≥ 30% |
| **Trial-to-paid conversion** | % of trial starts that become a paid subscription at trial end | ≥ 25% (card not required at trial start, see open question in D3) |
| **Voice match** | Mean "sounds like me" rating, 1–5, prompted on a sample of entries (D7 evaluation) | ≥ 4.0 |
| **Faithfulness** | % of rated entries with no reported invented or missing details | ≥ 97% |
| **Paid churn** | Monthly cancellations / active paid subscribers | ≤ 8% |

Targets are starting assumptions, not industry facts. They are decision thresholds for the validation plan (D12).

## Business model (summary; detail in D9)
One plan, two billing intervals: monthly (recommendation **$6/month**) or yearly (recommendation **$48/year**, i.e. 4 months free). A 14-day free trial. No ads, no data sales, no free tier beyond the trial in v1.
