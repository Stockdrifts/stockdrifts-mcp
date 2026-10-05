---
name: stockdrifts
description: Research stock ownership with the StockDrifts MCP tools. Use when the user asks which funds hold or bought a stock, what a 13F fund's holdings, changes or average buy price are, for company KPI history, for Korean insider buying (DART), or for Japanese large-shareholder filings and investor ratings (EDINET).
---

# StockDrifts ownership research

The `stockdrifts` MCP server exposes 32 read-only tools. Tool names end in the market they cover: `_us` (13F funds), `_kr` (Korea, DART), `_jp` (Japan, EDINET). `get_kpi` and `screen_insiders` are cross-market.

## Pick the tool by question

| Question | Tool |
|---|---|
| Which rated funds hold this stock, and at what average buy price? | `screen_holdings_us` with `tickers` and `ratings` |
| Who are the largest holders of this stock? | `list_ticker_holders_us` |
| What does this fund own? What changed last quarter? | `search_investors_us` to get the CIK, then `get_investor_holdings_us` or `get_investor_changes_us` |
| Which funds specialise in a sector? | `screen_investor_sector_exposure_us` |
| KPI history for a company | `get_kpi` |
| Korean insider buying across the market | `list_top_buys_kr`, `list_recent_trades_kr`, or `screen_insiders` with `country: "KR"` |
| Korean trades announced but not yet executed | `list_plans_kr`, then `get_plan_executions_kr` |
| Korean insiders with the best record | `list_leaderboard_kr` |
| Japanese stocks several investors are filing on | `list_convergence_jp` |
| Japanese investors by rating | `list_leaderboard_jp`, then `list_investor_filings_jp` with the EDINET code |
| Find a Korean or Japanese company code | `search_companies_kr`, `search_companies_jp` |

## Rules that prevent wrong answers

- Resolve names to identifiers first. Funds need a CIK (`search_investors_us`). Fund search matches the firm name, not the manager's name. Korean and Japanese tools need a stock code.
- 13F periods are quarter-end dates such as `2026-06-30`. Omit the period to get the latest quarter on file.
- Ratings are passed as strings, one bucket each, including half stars: `"4"`, `"4.5"`, `"5"`.
- Value fields carry their currency in the name (`value_usd`, `value_krw`, `value_jpy`). Do not convert unless the user asks, and say so when you do.
- 13F data is quarterly and published up to 45 days after quarter end. State the period when you quote a position.
- Japanese 5%-rule reports are filings by large shareholders. Do not describe them as director or officer trades.
- `price_current` and return fields on filings are computed when the filing was processed, not today. Do not compare them across filings as if they shared one date.
- List endpoints page with `page` (0-indexed) and `limit`. A page shorter than `limit` is the last page.

## Reporting

Quote the filing date or period beside every figure, name the filer, and keep the source currency. This is research data, not investment advice.
