# Trade

Multi-leg options trading assistant — concrete strikes, IV-aware structures, probability-weighted scenarios. Single skill with three subcommands, modeled on the [`pbakaus/impeccable`](https://github.com/pbakaus/impeccable) pattern.

## Commands

```
/trade setup                           # scaffold a personal knowledge directory
/trade import <file_path>              # parse one PDF / screenshot / text artifact into YAML
/trade report [tickers | basket]       # daily 资金流向 read (degraded — rebuilding on Alpaca)
/trade analysis [ticker | situation]   # default — trade analysis flow
/trade <natural language>              # any unrecognized first word routes to analysis
```

Each subcommand has its own reference file under `references/commands/`. The main `SKILL.md` carries always-on context (Hard Rule, Response Rules, Core Principles, Structure-to-Regime matrix) plus the routing logic.

## Triggers

- Trade analysis requests, options strategy recommendations, post-mortems
- Mentions of multi-leg structures: Jade Lizard, bull put / bear call spread, iron condor, diagonal, calendar
- Earnings positioning, IV / IV crush, channel checks, AH price action
- Any single-stock options play in a US-equity context
- Personal-knowledge management: "save this substack post", "parse this tweet screenshot", "set up my trade knowledge"

Full trigger list in the `description` field of `SKILL.md`.

## Platform

**CLI only** — primary market/options data via the **Alpaca MCP server** (read-only; Greeks/IV/chains/news/indexes); fundamentals/transcripts/estimates via the connected **FMP** MCP. (`report` 散户/大单/机构 flow + dealer GEX are being rebuilt on Alpaca raw trades — currently deferred.)

## Setup

> ⚠️ **Read-only proof required before use.** The Alpaca MCP server can place real trades and **fails open** — a missing or misspelled `ALPACA_TOOLSETS` enables every tool. Do not use this skill for analysis until the read-only proof (step 3) passes; a connected-but-unverified server is **DO NOT USE**.

1. **Add the Alpaca MCP server (Control A — allow-list).** In your MCP client config (project `.mcp.json` — a `.mcp.json.example` ships in the repo root), insert your own keys:
   ```json
   {
     "mcpServers": {
       "alpaca": {
         "command": "uvx",
         "args": ["alpaca-mcp-server"],
         "env": {
           "ALPACA_API_KEY": "<your-key>",
           "ALPACA_SECRET_KEY": "<your-secret>",
           "ALPACA_PAPER_TRADE": "true",
           "ALPACA_TOOLSETS": "stock-data,options-data,assets,news,index-data,corporate-actions"
         }
       }
     }
   }
   ```
   The allow-list **omits `trading` / `account` / `watchlists`**, so no order/position/exercise/account/watchlist tool is registered. Names must match exactly (lowercase, hyphenated) — one typo re-enables everything. **Never set `ALPACA_PAPER_TRADE=false`**: it is not a safety control, and data entitlement follows your keys regardless of it.

2. **Add the harness deny-list (Control B — independent backstop)** to `.claude/settings.json` (or `settings.local.json`), so mutating tools stay barred even if Control A regresses:
   ```json
   "permissions": {
     "deny": [
       "mcp__alpaca__place_stock_order", "mcp__alpaca__place_crypto_order",
       "mcp__alpaca__place_option_order", "mcp__alpaca__replace_order_by_id",
       "mcp__alpaca__cancel_order_by_id", "mcp__alpaca__cancel_all_orders",
       "mcp__alpaca__close_position", "mcp__alpaca__close_all_positions",
       "mcp__alpaca__exercise_options_position", "mcp__alpaca__do_not_exercise_options_position",
       "mcp__alpaca__update_account_config",
       "mcp__alpaca__create_watchlist", "mcp__alpaca__update_watchlist_by_id",
       "mcp__alpaca__delete_watchlist_by_id", "mcp__alpaca__add_asset_to_watchlist_by_id",
       "mcp__alpaca__remove_asset_from_watchlist_by_id"
     ]
   }
   ```

3. **Read-only proof (mandatory).** After connecting, enumerate the `alpaca` server's tools and confirm **zero** `place_*` / `replace_order_*` / `cancel_*` / `close_position` / `close_all_positions` / `exercise_*` / `do_not_exercise_*` / `update_account_config` / watchlist-mutator tools are present. Then smoke-test reads: `get_option_snapshot` on QQQ (Greeks+IV), `get_index_latest_values` (VIX), `get_corporate_actions`.

4. **Fundamentals (FMP).** Keep the connected FMP (Financial Modeling Prep) MCP for fundamentals/statements, transcripts, and analyst estimates — the data class Alpaca does not provide. Some endpoints may be plan-gated; on denial, state "unavailable" rather than fabricating. Refer to FMP by name (its MCP tool prefix is install-specific — do not hardcode a per-environment hash).

5. (Optional) Run `/trade setup` once to scaffold a personal knowledge directory for substack posts, X / twitter threads, and writedowns.

## Reference Files

### Always-relevant frameworks

| File | Description |
|---|---|
| `references/strategies.md` | Structure-to-regime matching, LEAPS stock replacement, setup checklist, position management |
| `references/gamma-framework.md` | Dealer GEX + options chain + IV term + flow → multi-factor probability map |
| `references/price-action-framework.md` | Orderbook microstructure mental model — buy/sell imbalance, vacuum zones, consensus shifts |

### Subcommand references (lazy-loaded by the router)

| File | Subcommand |
|---|---|
| `references/commands/setup.md` | `/trade setup` workflow |
| `references/commands/import.md` | `/trade import` workflow (raw artifact → YAML) |
| `references/commands/analysis.md` | Default analysis preflight + situation → reference map |
| `references/commands/report.md` | `/trade report` daily capital-flow read (degraded — Alpaca rebuild pending) |

### Lazy-loaded library

| File | Description |
|---|---|
| `references/pitfalls/index.md` | Index of 27 trading pitfalls (severity-tagged, lookup by trade type) |
| `references/pitfalls/NN-*.md` | One file per pitfall — loaded only when relevant |
| `references/ticker/index.md` | Index of closed trade case studies |
| `references/ticker/<name>.md` | One file per case study (INTC, Mag-7, APP, NOK, TSEM, CBRS, SNOW, MDB, SATS, VIX, 6981) |

### Templates (used by `/trade setup`)

| File | Copied to |
|---|---|
| `references/commands/templates/knowledge-README.md` | `<knowledge>/README.md` |
| `references/commands/templates/substack-template.yaml` | `<knowledge>/substack/_template.yaml` |
| `references/commands/templates/twitter-template.yaml` | `<knowledge>/twitter/_template.yaml` |
| `references/commands/templates/writedown-template.md` | `<knowledge>/writedowns/_template.md` |

## Coverage

- 27 analytical pitfalls covering consensus anchoring, flow misreading, IV crush traps, T+1 reverse drift, LEAPS vega tax, manipulator-tape recognition, channel-check sample bias, AH order-book fades, demand-IV vs event-IV, vega-axis sanity checks, VIX futures mechanics, and more.
- 11 detailed case studies (INTC, Mag-7, APP, NOK, TSEM, CBRS, SNOW, MDB, SATS, VIX, 6981) showing thesis evolution, structure selection, and post-mortem lessons.
- Structure-to-regime quick reference covering high/low IV regimes paired with directional / neutral / manipulator-tape views.
- Personal-knowledge layer for the user's own substack / X / writedown collection, auto-loaded on every analysis.
