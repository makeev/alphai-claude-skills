---
name: stock-brief
description: Produce a situational brief on a single stock/ticker — its recent high-relevance news, earnings reads, insider activity, and what to watch next — using the AlphaAI MCP. Use when the user asks to "brief me on <ticker>", "what's going on with <company>", "catch me up on NVDA", or wants a quick read on one name.
---

# Stock brief

Give a tight, decision-useful read on **one ticker** from AlphaAI's enriched
feed. Not a price quote — a *news* brief: what's been happening, what the last
earnings filing actually said, who's buying or selling, and what to keep an eye
on.

## Steps

1. **Resolve the ticker.** If the user gave a company name, not a symbol, call
   `alphai_tickers(q=...)` to find it. If `alphai_ticker_news` returns
   `unknown_ticker=true`, tell the user and stop — don't invent coverage.
2. **Pull the ticker feed.** Call `alphai_ticker_news(ticker, collapse_stories=true)`.
   `collapse_stories` folds syndicated reprints into one row each and adds a
   `sources_count` corroboration signal — prefer it for a clean brief. Insider
   news is included by default; keep it.
3. **Earnings read.** Call `alphai_earnings(ticker)` for AlphaAI's own
   filing-verified reads (8-K item 2.02 / 6-K earnings release, every figure
   checked against the filing). The feed tools return articles *about* a
   quarter; this returns the filing's numbers. An empty `reports` list is a
   normal answer, not an error. If non-empty, lead from the newest read —
   `fiscal_period`, `verdict` (strong/solid/mixed/weak), a headline
   `analysis.key_metrics`, guidance; `next_report_date` is company-confirmed
   (never an estimate) and `null` means none held.
4. **Optional context.** If the user wants the wider tape, call
   `alphai_trending(limit=5)` and note any market-level story that touches this
   name or its sector.
5. **Deep-dive on demand.** If one story clearly drives the brief and the user
   wants detail, fetch it with `alphai_article(uid)` for the full enrichment
   (ticker analysis, key entities). The article *body* is not served — work from
   the enrichment and summary.

## Output

Keep it to a screen. Structure:

- **One-line read** — the single most important thing about this name right now.
- **Top stories** — 3–5 bullets, each: headline · relevance score · one-clause
  "why it matters". Lead with the highest `relevance_score`.
- **Earnings** — the newest `alphai_earnings` read if there is one: period ·
  verdict · 1–2 metrics · guidance line. Otherwise one line ("no published
  AlphaAI earnings read yet").
- **Insider activity** — any Form 4 / 13F rows from the feed: who, buy or sell,
  size if given. "No notable insider activity in the window" is a valid line.
- **Watch next** — 1–2 concrete things that would change the picture, incl. the
  `next_report_date` when set.

## Guardrails

- Every story carries a 1–10 `relevance_score`; lead with the high ones and say
  the score so the user can calibrate.
- This is news, not advice. Don't issue buy/sell calls or price targets —
  summarize what the reporting says and let the user decide.
- If the feed is thin, say so plainly rather than padding. A short honest brief
  beats a long speculative one.
