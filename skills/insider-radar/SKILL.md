---
name: insider-radar
description: Scan SEC Form 4 insider trades for a ticker, a watchlist or the whole market using the AlphAI MCP — open-market buys, cluster buying by several insiders, and discretionary sales told apart from pre-planned 10b5-1 ones. Use when the user asks about "insider buying", "insider selling", "Form 4 activity", "who's buying <ticker>", "are insiders buying", or wants an insider-activity sweep.
---

# Insider radar

Surface what company insiders do with their own stock: **SEC Form 4** trades by
officers, directors and 10% owners. AlphAI turns each filing into a
relevance-scored news row with a structured `insider` block (side, shares,
`total_value_usd`, `is_10b5_1`, role, `transaction_date`, `filed_at`), so the
numbers come from the filing, not from a press rewrite.

## Steps

1. **Single ticker.** Call `alphai_insider_news(ticker="<SYM>")`. If the user
   wants only the strongest signal, raise the floor with `min_relevance=8`.
2. **Discretionary trades only.** A 10b5-1 sale was scheduled months earlier and
   carries no timing decision. When the user asks "is anyone *choosing* to sell
   or buy", pass `is_10b5_1=false` to drop planned trades.
3. **Cluster buying.** For "are insiders buying" across a name or the market,
   call `alphai_insider_clusters()` (or `symbol="<SYM>"`): issuers where several
   different insiders bought on the open market in one run. IPO/PIPE allocations,
   ESPP and plan lots are already filtered out of the default `open_market` view.
   Free accounts reach 30 days back, Basic and Pro 90.
4. **Watchlist.** Call `alphai_insider_news(ticker=...)` once per name. If the
   user gave none, call `alphai_insider_news()` unfiltered for the broad tape.
5. **Time-box on request.** For "this week" / "since earnings", pass
   `from_date` (ISO, UTC). Paginate with `cursor` only if they ask for more.

## Output

- **Notable buys** — open-market purchases first: ticker · who/role · size ·
  relevance. These are what most users actually want.
- **Cluster buying** — if `alphai_insider_clusters` found any: ticker · number of
  independent buyers · total · the window.
- **Notable sells** — say whether each is discretionary or under a 10b5-1 plan
  (`is_10b5_1`), and the seller's role.
- **Read** — one line: leaning accumulation, distribution, or just routine?

## Guardrails

- An insider sale is not automatically bearish (taxes, diversification, scheduled
  plans). Say what the filing shows; don't over-read it.
- One filing can span several tranches sharing a row — report the *event*, not a
  count of line items.
- `filed_at` can be days after `transaction_date`; say which date you mean.
- AlphAI does not carry fund-holdings filings (13F, 13G, 13D). If the user asks
  what a fund owns, say so plainly; press coverage of a stake may still turn up
  via `alphai_news_search(query="<fund> stake <company>")`.
- Lead with `relevance_score`; "no notable insider activity in the window" is a
  valid, useful answer.
- News, not advice.
