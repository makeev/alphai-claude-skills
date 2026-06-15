# alphai-claude-skills

Ready-to-use [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
**skills** for the [AlphaAI](https://alphai.io) MCP server — relevance-scored,
ticker-linked financial news plus SEC Form 4 / 13F insider data, straight inside
your agent.

Each skill is a small `SKILL.md` that teaches Claude *when* and *how* to call the
alphai MCP tools for a concrete finance workflow — so you can just ask
*"brief me on NVDA"* or *"what's moving right now?"* and get a structured answer
instead of a raw tool dump.

| Skill | Use it when you want… | Tools it leans on |
|---|---|---|
| **stock-brief** | A situational brief on one ticker — top news, insider activity, what to watch | `alphai_ticker_news`, `alphai_trending`, `alphai_article` |
| **market-pulse** | "What's moving *right now* / today?" across the market | `alphai_actionable_now`, `alphai_trending` |
| **insider-radar** | A scan of insider buying/selling for a ticker or watchlist | `alphai_insider_news` |
| **peer-readacross** | A two-ticker comparison and cross-read (e.g. NVDA vs AMD) | `alphai_pair_analysis` |
| **manage-alerts** | To list / add / remove ticker news-alert subscriptions | `alphai_alerts_*` *(Basic/Pro)* |

---

## 1. Connect the alphai MCP to Claude Code

One line — the first tool call opens your browser to authorize (OAuth 2.1, no API
key to paste):

```bash
claude mcp add --transport http alphai https://mcp.alphai.io/mcp
```

The **Free** tier is 100 tool calls/hour with no card. The MCP is read-only news
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

All 11 tools the skills can reach (full schemas at
[alphai.io/mcp](https://alphai.io/mcp)):

| Tool | What it returns |
|---|---|
| `alphai_news_search` | Search the enriched feed (query, tickers, category, dates, `min_relevance`). |
| `alphai_ticker_news` | Latest news for one ticker (insider included by default). |
| `alphai_trending` | Top news from the last 48h ranked by relevance. |
| `alphai_actionable_now` | Breaking, decision-grade news from the last few hours. |
| `alphai_insider_news` | SEC Form 4 + 13F ownership-change news. |
| `alphai_pair_analysis` | News naming two tickers, plus each one's own recent news. |
| `alphai_article` | One article by `uid`, with full enrichment. |
| `alphai_tickers` | List/search supported stock & ETF tickers. |
| `alphai_alerts_list` | Your active alert subscriptions *(Basic/Pro)*. |
| `alphai_alerts_subscribe` | Add/update a ticker alert *(Basic/Pro)*. |
| `alphai_alerts_unsubscribe` | Disable a ticker alert *(Basic/Pro)*. |

**Relevance score** is the enricher's 1–10 rating of how much a story matters for
its tickers; most tools default to a floor of 6. **Categories** (14): `earnings`,
`mergers_acquisitions`, `regulation`, `macro_economy`, `sector_analysis`,
`market_movers`, `technology`, `commodities`, `crypto`, `ipo`, `geopolitics`,
`insider`, `corporate_actions`, `other`.

## Links

- MCP setup & playground — <https://alphai.io/mcp>
- AlphaAI MCP on Glama — <https://glama.ai/mcp/servers/makeev/alphai-mcp>
- REST API & SDKs — <https://alphai.io/developers>
- Changelog — <https://alphai.io/changelog>

## License

[MIT](./LICENSE)
