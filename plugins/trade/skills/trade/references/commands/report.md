---
type: Command Reference
title: "/trade report [tickers | basket]"
description: Today's capital-flow / 资金流向 read for one or more names. DEGRADED — the 散户 / 大单 / 机构 split + GEX are being rebuilt on Alpaca raw options trades; currently returns price/% + per-contract IV·Greeks·OI + notable large prints + qualitative news tone. Read-only, not investment advice.
tags: [command, report, capital-flow, money-flow, options-flow, funds-flow]
timestamp: 2026-06-22T20:00:00Z
---

# /trade report &lt;tickers | basket&gt;

A daily **capital-flow / 资金流向** read across one or more names: who is buying vs selling today, split as a **散户 / 大单 / 机构** proxy, plus the price/volume context — rendered as a comparison table with a cross-section synthesis.

Runs whenever the user invokes `/trade report ...`, or asks for 资金流向 / 流入流出 / 净流入·净流出 / 散户·大单·机构 / capital flow / money flow / "who's buying" across a name or a basket.

> **⚠️ DEGRADED — read this 口径 (data-source reality) FIRST, and state it in every reply.** The previous 散户 / 大单 / 机构 split was built on Funda's *aggregated* options premium-flow, which is **retired**. The Alpaca rebuild (classifying raw option trades into signed net call/put premium, sweeps, GEX) is **deferred** — see [Deferred](#deferred-alpaca-rebuild). Until it lands, this command returns only what Alpaca gives cleanly:
>
> - **Price / % change** ← Alpaca `get_stock_snapshot` (vs prior close; mind market holidays).
> - **Per-contract IV · Greeks · OI** ← `get_option_snapshot` across near-the-money strikes.
> - **Notable large prints** ← biggest single trades from `get_option_trades` — raw and **unclassified** (not yet net bullish/bearish premium).
> - **散户 tone** ← Alpaca `get_news` headlines — **qualitative, not a scored feed**; thin on small / niche names.
> - **大单 / 机构 net-premium split, sweeps, GEX** ← **deferred** (see below). Say so; do not fabricate a split.
> - **机构 stock-side daily net flow** ← **not available** here. The *true* moomoo three-layer stock flow needs a logged-in **FutuOpenD gateway + `futu-api` SDK** (`get_financial_unusual`, env-gated, usually not running): `pip install futu-api` + start FutuOpenD on `127.0.0.1:11111` (you can install the SDK but cannot log in their gateway). See `futu-capital-anomaly` skill.

## Arguments

- **Explicit tickers** (space- or comma-separated): `report COHR LITE MU` → run those.
- **A sector / theme word** (e.g. "光" / optical, "存储" / memory, "光模块+存储"): **confirm the ticker universe first** (propose constituents + a market, ask via `AskUserQuestion`) — don't silently guess a basket. Once confirmed, group the output by basket.
- **Optional date**: default **today**. The endpoints return the latest session; if the user names a date, pass it through where the endpoint supports `date=`.
- Mixed baskets → render one table per basket so the cross-section reads cleanly.

## Workflow

> **Degraded:** until the [Deferred](#deferred-alpaca-rebuild) rebuild lands, run sections 1–2 and present the limited read honestly; the section-4 classification is the **rebuild target**, not currently computable.

### 1. Pull, per ticker (Alpaca, read-only)

| # | Tool | Gives | Use for |
|---|---|---|---|
| 1 | `get_stock_snapshot` / `get_stock_latest_quote` | latest + prior close | **涨跌%** = last vs prior close (mind market-holiday gaps — prior *trading* day, e.g. Juneteenth) |
| 2 | `get_option_snapshot` (near-the-money strikes) | per-contract **IV, Greeks (δ/γ/θ/vega), OI** | IV / Greeks / OI context |
| 3 | `get_option_trades` | raw trade prints (price × size) | **notable large prints only** — biggest tickets as an *activity* flag; raw, **unclassified** |
| 4 | `get_news` | recent headlines | **散户 tone** — qualitative, not scored |

For more than ~3 tickers, batch the calls in one small loop rather than dozens of separate calls.

### 2. Derive what's available now

- **涨跌%** — from #1 (mind holiday gaps).
- **IV / Greeks / OI** — from #2, near-the-money.
- **Activity flag** — from #3: the largest prints by premium (`price × size × 100`), labelled **unclassified** (not yet net bullish/bearish — that is the deferred part).
- **散户 tone** — from #4: qualitative direction from headlines, with the thin-coverage caveat.
- **财报日** — from `get_corporate_actions`, where listed.

### 3. Deferred — the smart-money split is not yet computable

The signed **净期权流向 (牛−熊)**, **净 Call / 净 Put 权利金** (with the "negative call premium = calls net SOLD" sign), **放量倍数**, and **盘口 ask-vs-bid** all require ask/bid-side trade classification **not yet** rebuilt from raw Alpaca trades — see [Deferred](#deferred-alpaca-rebuild). Until then, do **not** output these metrics or a 聪明钱 verdict; present sections 1–2 and say the split is rebuilding.

### 4. Classify each name (聪明钱判定) — REBUILD TARGET (not yet active)

> ⚠️ The table below is the **target** once the [Deferred](#deferred-alpaca-rebuild) classification lands; its triggers need signed net premium + ask/bid side, not yet available from raw Alpaca trades. Do **not** emit a 多头确认/背离 verdict until then.

| Label | Trigger |
|---|---|
| 🟢 **多头确认** | price up **and** net flow bullish (牛>熊) **and** calls net bought (net_call_prem>0, call ask>bid) **and** puts net sold — ideally with call volume ≥ ~1× avg (放量). Clean, confirmed long. |
| 🔴 **背离 / 派发** | **price up but options bearish** — calls net SOLD (net_call_prem<0, call ask<bid) and/or 熊>牛. The "价涨期权背离" tell; the relative weak name. |
| 🟡 **价拉·期权没跟** | price up but options **light** (volume << avg) and net flow ~flat. Momentum not yet confirmed by smart money — needs follow-through. |
| ⚖️ **双押 / 事件** | **both** call and put premium strongly net-bought **and** earnings within ~1–2 weeks → earnings straddle positioning. **Don't read the big "inflow" as single-direction conviction.** |

Always flag **earnings proximity** (from #2): a name reporting in days explains two-sided premium; a name reporting weeks out gives a *cleaner* directional read.

### 5. Output (degraded)

- **One table per basket** with the columns available now: `票 | 涨跌% | ATM IV | 大单 (largest prints, unclassified) | 散户 tone`. State that 净期权流向 / 净Call·Put 权利金 / 盘口 / 聪明钱判定 are **deferred**.
- A short **cross-section** line on price + IV + notable activity — **no** clean-long/distribution verdict until the split is rebuilt.
- A **散户 (news tone)** line: qualitative direction, thin-coverage caveat (not a scored feed).
- Respond in the user's language (**Chinese** by default — see User Profile).

## Constraints

- **Read-only.** This is data presentation, never a trade recommendation, price target, or buy/sell call. Close with a one-line **非投资建议** note.
- **State the 口径 every time**: the 大单/机构 net-premium split + GEX are **deferred** (Alpaca rebuild pending); currently only price/% + ATM IV·Greeks·OI + **unclassified** large prints + qualitative news tone; no stock-side three-layer net flow.
- **A single big order ≠ smart money** — read the *aggregate* premium, not one print. See [`../pitfalls/02-single-flow-not-smart-money.md`](../pitfalls/02-single-flow-not-smart-money.md).
- **Options flow is dealer-/positioning-driven, not "retail money"** — see [`../pitfalls/17-dealer-flow-not-retail.md`](../pitfalls/17-dealer-flow-not-retail.md).
- **Don't fabricate** numbers or a retail/institutional split the feed doesn't provide. If an endpoint errors or a name has no listed options, say so for that name and continue.
- This is a **read**, not the full structure flow — if the user then wants to *act* (size, pick a structure, model P/L), route to [`analysis.md`](analysis.md) and run the three-axes / bull-conviction checks there.

## Related

- [`../pitfalls/02-single-flow-not-smart-money.md`](../pitfalls/02-single-flow-not-smart-money.md) — one institutional order isn't edge.
- [`../pitfalls/17-dealer-flow-not-retail.md`](../pitfalls/17-dealer-flow-not-retail.md) — options flow is dealer hedging, not retail direction.
- [`../pitfalls/20-post-earnings-momentum-vs-fade.md`](../pitfalls/20-post-earnings-momentum-vs-fade.md) · [`../pitfalls/21-event-iv-vs-demand-iv.md`](../pitfalls/21-event-iv-vs-demand-iv.md) — pull flow + check the catalyst clock before any "fade / IV crush" call.
- [`../gamma-framework.md`](../gamma-framework.md) — dealer-positioning / GEX context. **GEX recompute is deferred** (Alpaca Σ gamma×OI; dealer-sign convention to be defined) — see Deferred below.
- [`analysis.md`](analysis.md) — when the read turns into an actual trade decision.

## Deferred (Alpaca rebuild)

The full 散户 / 大单 / 机构 split and dealer GEX are **not yet** rebuilt from Alpaca raw data. This is a follow-up with OPEN QUESTIONS — resolve against live tool output before implementing:

1. **Side classification (Lee-Ready):** classify each `get_option_trades` print as ask-side (buy) or bid-side (sell) using the prevailing NBBO *at the trade's timestamp*. **Open:** `get_option_latest_quote` is *latest* only — confirm a historical-option-quote-at-timestamp capability exists in the enabled toolsets; if not, this needs a different method (re-design, not a fill-in).
2. **Normative rule:** pick ONE — strict quote rule (≥ ask = buy) *or* mid-based (> mid = buy). Document the choice.
3. **Signed premium:** subtract bid-side (sell) premium so `net_call_premium` / `net_put_premium` carry the sign section 4 needs (**negative call premium = calls net SOLD**). Unsigned sums silently break the 🔴 背离 trigger.
4. **Sweep vs big-ticket (keep separate):** a *sweep* = same-direction prints across multiple venues; a *big ticket* = ≥ ~$50k premium. Do not merge them into one column.
5. **放量倍数 / OI** from `get_option_snapshot` + `get_option_bars` (avg N-day volume).
6. **GEX:** Σ over the chain of `gamma × OI × 100 × spot` from `get_option_snapshot`; **define the dealer-sign convention** in [`../gamma-framework.md`](../gamma-framework.md) first (not currently documented there — it sets the zero-gamma flip level).

Until these land, `report` runs in the degraded mode described above and says so.
