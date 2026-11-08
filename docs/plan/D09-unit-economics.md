# D9. Unit economics

**Status:** Draft · **Owner:** Head Manager · Prices from `docs/research/R2-versions-and-vendors.md`. Anthropic prices were read from the official pricing page; **all other vendor prices are [VERIFY]** (official pages were unreachable from this session).

## Assumptions (change these and the model changes)
| # | Assumption | Value | Why |
|---|---|---|---|
| A1 | Entries per active paying user per month | 20 | Nightly habit with misses; D1 target is 4+ nights a week |
| A2 | Speaking time per entry | 6 min (~900 words at ~150 words/min) | PO's own habit; to validate in dogfooding |
| A3 | Transcription session overhead (idle, reconnects) | +20% | AssemblyAI bills full socket time |
| A4 | Rewrite input tokens | ~5,200 (system 700 + samples 3,300 + transcript 1,200) | D7 sample budget |
| A5 | Rewrite output tokens | ~800 | 50–80% of transcript length |
| A6 | Rewrite retries / re-runs | +30% | Failures, user re-runs |
| A7 | Price | $6/month or $48/year | D1 recommendation |
| A8 | Billing mix | 60% monthly, 40% yearly | Placeholder until real data |
| A9 | Trial-to-paid | 25% | D1 target |
| A10 | Entries during a 14-day trial | 7 | Half the nights |

## Cost per entry
| Item | Base case | Pessimistic case |
|---|---|---|
| Transcription | AssemblyAI Universal-Streaming ~$0.15/hr → 7.2 min billed = **$0.018** | OpenAI realtime ~$0.017/min → **$0.122** |
| Rewrite | Sonnet 5.5 ($2 / $10 per M tokens): $0.0104 + $0.008 = $0.018 × 1.3 = **$0.024** | Opus 5.5 ($4 / $20): $0.037 × 1.3 = **$0.048** |
| **Total per entry** | **$0.042** | **$0.170** |

Prompt caching (samples and system prompt repeat each night) can lower the input part further [VERIFY at build time]; not counted here.

## Per paying user per month
| Line | Base | Pessimistic |
|---|---|---|
| Average revenue (60% × $6 + 40% × $4) | $5.20 | $5.20 |
| Payment processing: monthly $6 → 2.9% + $0.30 + 0.7% Billing = $0.52; yearly → $2.03/yr = $0.17/mo; blended | −$0.38 | −$0.38 |
| Variable AI + transcription (20 entries) | −$0.84 | −$3.40 |
| **Contribution per paying user** | **$3.98** | **$1.42** |

Sales tax/VAT is collected on top of the price shown and remitted; Stripe Tax adds a fee [VERIFY]. Not counted as cost.

**Trial cost:** each paying user costs about 4 trials (at 25% conversion) × 7 entries × $0.042 = **$1.18 one-time** (base), $4.76 pessimistic. Paid back in the first month.

## Fixed monthly costs (estimates, [VERIFY])
| Item | 100 users | 1,000 users | 10,000 users |
|---|---|---|---|
| Vercel Pro (Hobby forbids commercial use) | $20 | $20 | $20 + usage ≈ $150 |
| Supabase Pro (+ larger compute at scale, PITR add-on) | $25 | $60 | $300 |
| Error tracking (Sentry) | $0 | $26 | $80 |
| Analytics (cookieless) | $9 | $14 | $50 |
| Transactional email | $0 | $20 | $50 |
| Domain, misc. | $5 | $10 | $20 |
| **Total fixed** | **≈ $60** | **≈ $150** | **≈ $650** |

## Monthly totals (paying users)
| Paying users | Revenue | Variable + payments (base) | Fixed | **Profit (base)** | Profit (pessimistic) |
|---|---|---|---|---|---|
| 100 | $520 | $122 | $60 | **$338** | $82 |
| 1,000 | $5,200 | $1,220 | $150 | **$3,830** | $1,270 |
| 10,000 | $52,000 | $12,200 | $650 | **$39,150** | $13,550 |

## Break-even
- **Base:** $60 fixed ÷ $3.98 ≈ **16 paying users**.
- **Pessimistic:** $60 ÷ $1.42 ≈ **43 paying users**.
- The real constraint is not infrastructure. It is one-time costs and the founder's time: a lawyer's review of the legal drafts (budget estimate $1,500–$3,000 [VERIFY]), the trademark search/filing if pursued, and marketing.

## What this tells us
1. At $6/month the product has healthy margins even if transcription costs 7× the base case. **Price is not constrained by cost**; set it by what users will pay (test $6 vs $8 on the waitlist, D12).
2. The biggest cost swing is the **transcription vendor**. Choosing it (D5) matters more than the AI model.
3. Heavy users (60-minute nightly sessions) would cost ~10× the base. A soft limit (60 minutes per recording, D7) keeps the worst case bounded; no monthly cap is proposed for v1.
4. Do not offer a permanent free tier in v1: each free active user costs ~$0.84/month with no revenue.
