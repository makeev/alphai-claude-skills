# alphai-claude-skills

Ready-to-use [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
**skills** for the [AlphAI](https://alphai.io) MCP server — relevance-scored,
ticker-linked financial news, SEC Form 4 insider trades, and filing-verified
earnings reads, straight inside your agent.

Each skill is a small `SKILL.md` that teaches Claude *when* and *how* to call the
alphai MCP tools for a concrete finance workflow — so you can just ask
*"brief me on NVDA"* or *"what's moving right now?"* and get a structured answer
instead of a raw tool dump.

| Skill | Use it when you want… | Tools it leans on |
|---|---|---|
| **stock-brief** | A situational brief on one ticker — top news, earnings read, corporate events, insider activity, what to watch | `alphai_ticker_news`, `alphai_earnings`, `alphai_trending`, `alphai_article` |
| **market-pulse** | "What's moving *right now* / today?" across the market | `alphai_actionable_now`, `alphai_stories`, `alphai_trending`, `alphai_news_search` |
| **insider-radar** | A scan of insider buying/selling (SEC Form 4) for a ticker, a watchlist or the market | `alphai_insider_news`, `alphai_insider_clusters` |
| **peer-readacross** | A two-ticker comparison and cross-read (e.g. NVDA vs AMD) | `alphai_pair_analysis` |
| **manage-alerts** | To list / add / remove ticker news-alert subscriptions | `alphai_alerts_*` *(Basic/Pro)* |

---

## 1. Connect the alphai MCP to Claude Code

One line — Claude Code prompts for OAuth on connect and opens your browser to
authorize (OAuth 2.1, no API key to paste; re-authenticate any time with `/mcp`):

```bash
claude mcp add --transport http alphai https://mcp.alphai.io/mcp
```

Headless box (no browser)? Use an alphai API key from
[alphai.io/account](https://alphai.io/account) as a static Bearer header
instead:

```bash
claude mcp add --transport http alphai https://mcp.alphai.io/mcp \
  --header "Authorization: Bearer ak_live_YOUR_KEY"
```

The **Free** tier is 100 tool calls/day (20/min burst) with no card. The MCP is read-only news
data; nothing here places trades or touches your account beyond your own alert
subscriptions. See [alphai.io/mcp](https://alphai.io/mcp) for other clients
(Claude Desktop, Cursor, VS Code, Gemini, …).

## 2. Install the skills

Skills are just folders. Copy the ones you want into a skills directory Claude
Code reads:

```bash
# Personal — available in every project on this machine
cp -r skills/* ~/.claude/skills/

# …or per-project — checked in for your team
mkdir -p .claude/skills && cp -r skills/* .claude/skills/
```

Claude Code discovers them automatically. Run `/help` or just start a chat — when
your request matches a skill's description, Claude invokes it.

## 3. Use them

You don't call skills explicitly; you describe what you want and Claude picks the
right one:

- *"Give me a brief on TSLA."* → **stock-brief**
- *"What's the big story in the market this morning?"* → **market-pulse**
- *"Any notable insider buying in PLTR lately?"* → **insider-radar**
- *"Compare NVDA and AMD — what's the read-across?"* → **peer-readacross**
- *"Subscribe me to high-relevance AAPL earnings alerts."* → **manage-alerts**

---

## Tool reference

All 19 tools the skills can reach (full schemas at
[alphai.io/mcp](https://alphai.io/mcp)):

| Tool | What it returns |
|---|---|
| `alphai_news_search` | Full-text `query` over titles, entities and summaries, and/or filters (tickers, category, dates, `min_relevance`, `source_type`, `event_type`). |
| `alphai_ticker_news` | Latest news for one ticker (insider included by default); `event_type` keeps corporate events of the given types. |
| `alphai_trending` | Top articles from the last 48h ranked by relevance, reprints collapsed. |
| `alphai_stories` | Stories — events two or more publishers reported, each summarised from its articles; `list="top"` ranks the last 48h by importance, coverage and freshness, `list="latest"` pages newest-first. |
| `alphai_story` | One story by `story_id` (the key every news item carries): summary, key facts with their sources, per-ticker read, open questions, and the articles behind it. `not_a_story` means one publisher carried the event. |
| `alphai_actionable_now` | Breaking, decision-grade news from the last few hours. |
| `alphai_insider_news` | SEC Form 4 insider trades as news rows with a structured `insider` block; every sale is labelled `discretionary`, `plan_like` or `unknown`, and `decision="discretionary"` keeps the sales an insider chose to make. |
| `alphai_insider_clusters` | Issuers where several insiders bought on the open market in one run (IPO/PIPE/plan lots filtered out). |
| `alphai_pair_analysis` | News naming two tickers, plus each one's own recent news. |
| `alphai_article` | One article by `uid`, with full enrichment (adds a structured `earnings` read on SEC filings; an 8-K, a 6-K earnings release or a PR Newswire release also returns its `event` block). |
| `alphai_earnings` | AlphAI's filing-verified earnings reads per ticker, plus the next report date. |
| `alphai_calendar` | Scheduled US macro releases (CPI, FOMC, jobless claims) with the coverage that followed. |
| `alphai_macro` | Macro-economy feed — prints, central banks, rates, FX, commodities. |
| `alphai_radar` | Rank news activity and tone across the market or your saved watchlist, with SEC context per ticker. |
| `alphai_watchlist_brief` | Your saved watchlist in one read: news grouped by story, SEC filings, next earnings dates. |
| `alphai_tickers` | List/search supported tickers — US stocks, ETFs, crypto & foreign listings, incl. `next_report_date`. |
| `alphai_alerts_list` | Your active alert subscriptions *(Basic/Pro)*. |
| `alphai_alerts_subscribe` | Add/update a ticker alert *(Basic/Pro)*. |
| `alphai_alerts_unsubscribe` | Disable a ticker alert *(Basic/Pro)*. |

**Relevance score** is the enricher's 1–10 rating of how much a story matters for
its tickers; most tools default to a floor of 4. **Categories** (14): `earnings`,
`mergers_acquisitions`, `regulation`, `macro_economy`, `sector_analysis`,
`market_movers`, `technology`, `commodities`, `crypto`, `ipo`, `geopolitics`,
`insider`, `corporate_actions`, `other`.

**Corporate events:** news items that come from an SEC 8-K, a 6-K earnings
release or a PR Newswire release carry an `event` block with a `type` (17
values, such as `m_and_a`, `offering`, `buyback`, `dividend`, `exec_change` and
`guidance_update`), the company's `role` and `facts` quoted from the document.
`event_type` on `alphai_news_search` and `alphai_ticker_news` filters by type;
in `alphai_news_search` it does not combine with `query` or `sort="ingested"`.
`verified` is false when a quoted fact could not be found in the document and
was left out. Press coverage and Form 4 rows have no event.

**Insider sale label:** every Form 4 sale in the `insider` block carries
`decision`. It is `discretionary` when no plan or paperwork in the filing
explains the sale, `plan_like` when the sale follows from a 10b5-1 plan or from
paperwork, such as shares sold to pay the tax on vesting or to cover an option
exercise, and `unknown` when the filing does not say enough to tell.
`decision_reason` names what decided it, for example `tax_withholding` or
`exercise_cover`. Purchases and other transaction codes carry `null`. The label
is read from the whole filing, not only from the 10b5-1 checkbox, so
`is_10b5_1=false` still includes tax sales. To keep only the sales an insider
chose to make, pass `decision="discretionary"` to `alphai_insider_news`.

## Links

- MCP setup & playground — <https://alphai.io/mcp>
- AlphAI MCP on Glama — <https://glama.ai/mcp/connectors/io.github.makeev/alphai-mcp>
- AlphAI MCP on mcp.so — <https://mcp.so/server/alphai-news/makeev>
- REST API & SDKs — <https://alphai.io/developers>
- Changelog — <https://alphai.io/changelog>

## License

[MIT](./LICENSE)
