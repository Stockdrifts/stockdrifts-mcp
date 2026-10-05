# StockDrifts MCP Server

Ownership data for AI agents: rated US 13F funds and their average buy prices, normalized company KPIs, Japanese large-shareholder filings with investor ratings, and Korean insider buys.

- **Hosted endpoint:** `https://mcp.stockdrifts.io/mcp/` (Streamable HTTP)
- **Tools:** 32, all read-only
- **Auth:** API key sent as a bearer header
- **Docs:** https://www.stockdrifts.io/api-docs
- **Registry name:** `io.stockdrifts/mcp`

This repository holds the connection configs, the registry manifest and an agent skill. The server itself is hosted by StockDrifts; there is nothing to build or run.

## What is different about the data

| Dataset | What you get | Source |
|---|---|---|
| **Rated 13F funds** | 13F filers with star ratings, holdings by quarter, quarter-over-quarter changes, sector exposure, and each fund's average buy price per position | SEC 13F filings |
| **Company KPIs** | Company-specific operating metrics per ticker, such as segment revenue, customer counts, net revenue retention and backlog, as quarterly and yearly series | Company filings |
| **Japanese investors** | Large-shareholding (5%-rule) reports per stock and per investor, an investor leaderboard with star ratings, and stocks where several investors are filing at once | EDINET |
| **Korean insiders** | Executive and major-shareholder trades, top insider buys and sells, an insider leaderboard, and pre-disclosed trading plans matched to their executions | DART |

Japanese 5%-rule reports are filings by large shareholders. They are not the same thing as US director and officer trade reports.

## Connect

You need an API key. Create one at https://app.stockdrifts.io/settings/api-keys (API and MCP access is part of the Ultra plan, see [pricing](https://www.stockdrifts.io/pricing)).

Keep the key in an environment variable so it never lands in a config file or a repository:

```bash
export STOCKDRIFTS_API_KEY="sd_live_..."
```

### Claude Code

```bash
claude mcp add --transport http stockdrifts https://mcp.stockdrifts.io/mcp/ \
  --header "Authorization: Bearer $STOCKDRIFTS_API_KEY"
```

Or install this repository as a plugin (server plus skill). Claude Code asks for the key once and keeps it in your system's secure credential store:

```
/plugin marketplace add Stockdrifts/stockdrifts-mcp
/plugin install stockdrifts@stockdrifts
```

### Claude Desktop, Cursor and other JSON-config clients

```json
{
  "mcpServers": {
    "stockdrifts": {
      "type": "http",
      "url": "https://mcp.stockdrifts.io/mcp/",
      "headers": { "Authorization": "Bearer ${STOCKDRIFTS_API_KEY}" }
    }
  }
}
```

If your client does not expand environment variables, paste the key in place of `${STOCKDRIFTS_API_KEY}` and keep that file out of version control.

### Codex CLI

In `~/.codex/config.toml` (Codex spells the field `http_headers`):

```toml
[mcp_servers.stockdrifts]
url = "https://mcp.stockdrifts.io/mcp/"
http_headers = { Authorization = "Bearer sd_live_..." }
```

### One-command installer

The [`@stockdrifts/cli`](https://www.npmjs.com/package/@stockdrifts/cli) package registers a local stdio server with Claude Code, Claude Desktop, Cursor or Codex and stores the key in `~/.stockdrifts/config.json` (mode 600), so no secret is written into the agent's own config:

```bash
npx -y @stockdrifts/cli@latest install all --key sd_live_...
npx -y @stockdrifts/cli@latest doctor
```

## Example prompts

- "Which 5-star 13F funds hold NVDA in their top positions, and what is each fund's average buy price?"
- "What did this fund add and cut last quarter?"
- "Show Korean open-market insider buys above ₩1bn in the last 30 days."
- "Which pre-disclosed Korean insider buy plans have not been executed yet?"
- "Which Japanese stocks have several investors filing 5% reports at the same time?"
- "Give me the KPI history for this ticker."

## Tools

**US 13F funds**

| Tool | What it returns |
|---|---|
| `search_investors_us` | Fuzzy-search investors by name; resolves to CIKs ordered by AUM |
| `screen_investor_sector_exposure_us` | Screen investors by sector concentration (e.g. top biotech specialists) |
| `screen_investor_catalyst_record_us` | Screen funds by their track record on binary events (any sector) |
| `get_investor_holdings_us` | Latest 13F holdings for one investor (or for a specific period) |
| `list_investor_periods_us` | All 13F periods on file for one investor |
| `get_investor_changes_us` | QoQ position diff for one investor between two periods |
| `get_investor_sector_exposure_us` | Sector-exposure history for one investor (newest quarter first) |
| `get_investor_catalyst_record_us` | One fund's binary-event track record per sector, with best/worst calls |
| `list_ticker_holders_us` | Investors with `ticker` in their top-50 positions, sorted by position size |
| `screen_holdings_us` | Screen 13F holdings by period, ticker, value, and rating; returns each matched fund with only the position(s) that matched |

**Company KPIs**

| Tool | What it returns |
|---|---|
| `get_kpi` | Latest normalized KPI snapshot for one ticker |

**Japan (EDINET large-shareholding filings)**

| Tool | What it returns |
|---|---|
| `list_insider_filings_jp` | Shareholding-report filings for one JP ticker |
| `get_insider_summary_jp` | Pre-aggregated shareholding summary for one JP ticker |
| `list_recent_filings_jp` | Recent JP filings across all tickers |
| `list_top_buys_jp` | Top JP tickers by aggregated buying over a window |
| `list_top_sells_jp` | Top JP tickers by aggregated selling over a window |
| `list_investor_filings_jp` | All JP filings made by one investor (EDINET code) |
| `list_opportunities_jp` | Recent JP BUY/INITIAL filings whose price hasn't moved much yet |
| `list_leaderboard_jp` | Top JP investors by star rating + avg return |
| `list_convergence_jp` | JP tickers with multiple distinct investors filing on the same name |
| `search_companies_jp` | Search JP companies by stock_code, Japanese name, or English name |

**Korea (DART insider filings)**

| Tool | What it returns |
|---|---|
| `list_insider_trades_kr` | Insider trades for one KR ticker |
| `get_insider_summary_kr` | Pre-aggregated insider summary for one KR ticker |
| `list_recent_trades_kr` | Recent KR insider trades across all tickers |
| `list_top_buys_kr` | Top KR tickers by aggregated insider buying over a window |
| `list_top_sells_kr` | Top KR tickers by aggregated insider selling over a window |
| `list_opportunities_kr` | Recent KR open-market BUYs whose price hasn't moved much yet |
| `list_plans_kr` | KR pre-disclosure (D005) plans: announced but not yet executed trades |
| `get_plan_executions_kr` | Cross-reference one D005 plan against its actual executions |
| `search_companies_kr` | Search KR companies by stock_code, Korean name, or English name |
| `list_leaderboard_kr` | Top KR insiders by historical avg return on their buys |

**Cross-market screen**

| Tool | What it returns |
|---|---|
| `screen_insiders` | Screen insider activity by country, date, value, role, and ticker |

## Security

- Send the key only in the `Authorization` header. Do not put it in the URL, where it can end up in logs and shell history.
- The MCP server stores no keys. Your bearer token is forwarded to `api2.stockdrifts.io`, where authentication, rate limits and usage are enforced.
- Every tool is read-only. Nothing in this server writes, trades or changes account state.
- Without a key the server lists its tools and returns an `unauthorized` error for every call. No data is served.
- Rotate or revoke keys at https://app.stockdrifts.io/settings/api-keys.
- Report a security issue to hk@stockdrifts.io.

## Rate limits and errors

Limits are enforced per key in one-minute windows, and responses carry `X-RateLimit-*` headers. Errors use one shape: `{"error": {"code", "message", "hint"}}`. See the [API reference](https://www.stockdrifts.io/api-docs).

## Data notes

- Value fields carry their currency in the name (`value_usd`, `value_krw`, `value_jpy`). Nothing is converted silently.
- 13F periods are quarter-end dates such as `2026-06-30`. Leave the period out to get the latest quarter on file.
- 13F filings are quarterly and published up to 45 days after quarter end.
- The data is for research. It is not investment advice.

## License

The files in this repository are MIT licensed. Access to the StockDrifts API and the data it returns is a separate paid service under the StockDrifts [Terms](https://www.stockdrifts.io/legal/terms) and [Privacy Policy](https://www.stockdrifts.io/legal/privacy-policy).
