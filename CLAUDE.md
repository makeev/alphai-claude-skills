# CLAUDE.md — working with the AlphAI MCP skills

This repo is a collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
**skills** that drive the [AlphAI](https://alphai.io) MCP server for
financial-news workflows. This file orients an agent working **in** this repo
(editing or adding skills) and **with** these skills (running them). Human setup
lives in [README.md](./README.md); this is the agent-facing companion.

## What's here

```
skills/<name>/SKILL.md   one skill each — frontmatter (name + description) + instructions
README.md                human onboarding: connect the MCP, install the skills
```

| Skill | Fires when the user wants… | Primary tools |
|---|---|---|
| `stock-brief` | a situational brief on one ticker | `alphai_ticker_news`, `alphai_earnings`, `alphai_trending`, `alphai_article` |
| `market-pulse` | "what's moving right now / today?" | `alphai_actionable_now`, `alphai_stories`, `alphai_trending` |
| `insider-radar` | an insider buying/selling scan (SEC Form 4) | `alphai_insider_news`, `alphai_insider_clusters` |
| `peer-readacross` | a two-ticker comparison / read-across | `alphai_pair_analysis` |
| `manage-alerts` | to list/add/remove alert subscriptions *(Basic/Pro)* | `alphai_alerts_*` |

## Prerequisite: the AlphAI MCP must be connected

These skills are thin orchestration over MCP tools — they do nothing without the
server connected:

```bash
claude mcp add --transport http alphai https://mcp.alphai.io/mcp
```

The first tool call opens a browser for OAuth 2.1 (no API key to paste). It's
**read-only news data** on the Free tier (100 calls/day, 20/min burst — no card); only the
`alphai_alerts_*` tools mutate state, and only the caller's own subscriptions.
If the `alphai_*` tools aren't in the tool list, stop and tell the user to run
the command above before continuing.

## The MCP tool contract (reference)

Use the exact names, params, and defaults below when a skill calls a tool or when
you edit one. Don't invent tools or params — if unsure, check
[alphai.io/mcp](https://alphai.io/mcp).

| Tool | Key params (defaults) |
|---|---|
| `alphai_news_search` | `query?` (alias `q`; full text over titles, entities, summaries — read `query_interpretation`), `tickers?`, `category?`, `source_type?`, `item?`, `event_type?` (corporate event types, OR; not with `query` or `sort="ingested"`), `from_date?`, `to_date?`, `min_relevance=4`, `page_size=10`, `cursor?`, `sort="published"\|"ingested"`, `collapse_stories=false` |
| `alphai_ticker_news` | `ticker`, `include_insider=true`, `event_type?`, `page_size=10`, `cursor?`, `collapse_stories=false` |
| `alphai_trending` | `limit=10`, `min_relevance=8`, `dedupe=true` |
| `alphai_stories` | `list="top"\|"latest"`, `limit=10`, `cursor?` (latest only) — Stories with two or more publishers, summarised; `summary_state` published / stale / preparing |
| `alphai_story` | `story_id`, `materials_limit=20` — lookup `status` found / merged (`redirect_to`) / not_a_story (`article`), plus `materials` |
| `alphai_actionable_now` | `limit=10`, `hours=6`, `min_novelty=7`, `min_actionability="high"\|"medium"`, `dedupe=true` |
| `alphai_insider_news` | `ticker?`, `is_10b5_1?` (10b5-1 plan status; false is not the same as chosen), `decision?` (`"discretionary"\|"plan_like"\|"unknown"`, the sale label; matches sales only), `from_date?`, `to_date?`, `min_relevance=4`, `page_size=10`, `cursor?`, `sort="published"\|"ingested"` — SEC Form 4 only |
| `alphai_insider_clusters` | `symbol?`, `days=30` (Free 30, Basic/Pro 90), `min_buyers=2`, `pattern="open_market"`, `sort="recent"\|"buyers"\|"value"`, `min_usd?`, `limit=10`, `cursor?` |
| `alphai_pair_analysis` | `ticker_a`, `ticker_b`, `min_relevance=4`, `limit=5` |
| `alphai_article` | `uid` |
| `alphai_earnings` | `ticker` |
| `alphai_calendar` | `days_ahead=7`, `days_back=0`, `importance?`, `event_keys?` |
| `alphai_macro` | `limit=10`, `hours=24`, `min_relevance=4`, `categories?`, `dedupe=true` |
| `alphai_radar` | `scope="market"\|"watchlist"`, `window="24h"\|"4h"`, `market="all"`, `sentiment="all"`, `sort="score"`, `order="desc"`, `tickers?`, `min_z?`, `limit=20`, `cursor?` |
| `alphai_watchlist_brief` | `hours=24`, `limit=20` — the caller's saved watchlist |
| `alphai_tickers` | `q?`, `sector?`, `limit=100`, `offset=0` — returns `next_report_date` (company-confirmed, never an estimate) |
| `alphai_alerts_list` | — *(Basic/Pro)* |
| `alphai_alerts_subscribe` | `ticker`, `category_filter?`, `min_relevance_score?` *(Basic/Pro)* |
| `alphai_alerts_unsubscribe` | `ticker` *(Basic/Pro)* |

- **`relevance_score`** is a 1–10 rating of how much a story matters for its
  tickers; most tools floor at 4 (`alphai_trending` floors at 8).
- **Categories (14):** `earnings`, `mergers_acquisitions`, `regulation`,
  `macro_economy`, `sector_analysis`, `market_movers`, `technology`,
  `commodities`, `crypto`, `ipo`, `geopolitics`, `insider`, `corporate_actions`,
  `other`.
- **Corporate events.** Items that come from an SEC 8-K, a 6-K earnings release
  or a PR Newswire release carry an `event` block in `alphai_news_search`,
  `alphai_ticker_news` and `alphai_article`: `type`, `role`, `stage`,
  `counterparty`, `facts` (quoted from the document, each
  `{name, value, number, number_high, unit, scale}`), `also` (the other events
  of the same document, with their own facts) and `verified` (false when a
  quoted fact was not found in the document and was left out, or while the
  facts are still being read). Press coverage and Form 4 rows have
  `event: null`.
- **Event types (17):** `earnings`, `earnings_date`, `exec_change`, `m_and_a`,
  `offering`, `debt_financing`, `dividend`, `buyback`, `contract_award`,
  `guidance_update`, `stock_split`, `listing_notice`, `shareholder_vote`,
  `bankruptcy`, `restructuring`, `clinical_regulatory`, `other`. `event_type`
  takes a list of them and also matches the types in `event.also`; with
  `collapse_stories=true` only the item that leads its story is matched.
- **Insider sale label.** Every Form 4 sale in the `insider` block of
  `alphai_insider_news`, and of `alphai_article` on a Form 4 row, carries
  `decision` and `decision_reason`. `discretionary`: no plan or paperwork in
  the filing explains the sale, the insider chose to sell. `plan_like`: the
  sale follows from a 10b5-1 plan or from paperwork, such as shares sold to pay
  the tax on vesting or to cover an option exercise. `unknown`: the filing does
  not say enough to tell. Only sales (`transaction_code` `S`) are labelled:
  purchases, a sale to the issuer (`D`) and other codes carry `decision: null`.
  An event with several sales is `discretionary` if any of them is, else
  `unknown` if any of them is, else `plan_like`. The label is read from the
  whole filing, not only from the 10b5-1 checkbox, so `is_10b5_1=false` still
  includes tax sales. As a filter, `decision` takes one value. The three values
  do not overlap: together with the events that hold no sale they add up to the
  unfiltered feed. The filter combines with `is_10b5_1`, `sort="ingested"` and
  the dates.
- **Decision reasons (12):** `plan_10b5_1`, `tax_withholding`,
  `exercise_cover`, `stated_non_discretionary` with `plan_like`; `open_market`,
  `exercise_and_sell`, `sold_all_vested` with `discretionary`;
  `partial_vest_sale`, `cover_ambiguous`, `mixed_acquisitions`,
  `price_disputed`, `no_lines` with `unknown`. The reason comes from the
  largest sale that set the label, and it can be `null` next to a label. If
  every sale carries `decision: null`, the label is unavailable: fall back to
  `is_10b5_1` and say so.
- Paginated tools return an opaque `next_cursor`; pass it back as `cursor` for the
  next (older) page. Don't synthesize cursors.
- `unknown_ticker=true` (or `unknown_tickers`) means the symbol isn't a recognized
  active ticker — surface it, don't fabricate coverage.

## House style (applies to every skill)

- **News, not advice.** Summarize what the reporting says. No buy/sell calls, no
  price targets.
- **Lead with relevance**, and say the number, so the user can calibrate.
- **Prefer `collapse_stories=true`** for clean briefs — it folds syndicated
  reprints into one row with a `sources_count` corroboration signal.
- **Actionable ≠ trending.** `alphai_actionable_now` = act-today; `alphai_trending`
  = biggest stories of the last 48h. Don't upgrade a trending item to "breaking".
- **An empty result is a valid answer.** Outside major market hours (nights,
  weekends — crypto trades 24/7, but high-actionability prints still cluster in
  US/European/Asian sessions), `alphai_actionable_now` returning `[]` is expected
  — widen `hours` / `min_actionability` once before concluding nothing happened.
  Don't pad a thin feed with speculation.
- **Insider nuance.** Report the *event*, not a count of tranches; an insider sale
  isn't automatically bearish (taxes, scheduled 10b5-1, option exercises). Call
  a sale the insider's choice only when `decision` is `discretionary`;
  `is_10b5_1=false` alone does not show it.
- **Event facts come from the document.** Take amounts, dates and names from
  `event.facts` and `event.also`, not from titles. When `verified` is false, say
  which detail is missing instead of filling it in.

## Adding or editing a skill

- A skill is a folder `skills/<kebab-name>/SKILL.md`. The frontmatter needs
  `name` and a trigger-style `description` ("Use when the user asks to …") — the
  description is what makes Claude auto-invoke it, so make it match real phrasing.
- Keep each skill focused and short. Reference exact tool names/params from the
  contract above; state the output shape and the guardrails.
- Mirror the house style. If a skill needs a paid tier (`alphai_alerts_*`), say so
  in the description and handle `tier_not_paid` gracefully.

## Links

- MCP setup & playground — <https://alphai.io/mcp>
- REST API & SDKs — <https://alphai.io/developers>
- Changelog — <https://alphai.io/changelog>
