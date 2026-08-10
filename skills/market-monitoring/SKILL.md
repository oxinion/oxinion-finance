---
name: market-monitoring
description: >-
  Use this whenever someone wants to know "what's happening lately?" with a
  stock or a list of stocks — "what's going on with NVDA", "any news on Apple",
  "how has TSLA moved this month", "did the market like their earnings", "are
  insiders buying", "did 13F holders trim", "what did they just file", or "give
  me a morning watchlist for these tickers." Reach for it for price-action
  reviews (returns, drawdown, volatility), recent-news triage, earnings
  reactions, insider (Form 4) and institutional (13F) activity, filing changes
  (10-K/10-Q/8-K), and multi-ticker watchlist dashboards — even if they don't
  say "monitor" or "news" and just paste a ticker asking "anything I should
  know?" For a business deep-dive use company-understanding; for statements and
  ratios use financial-analysis; for fair-value work use valuation; for
  balance-sheet and earnings-quality use risk-quality; for a full written
  thesis use investment-research.
version: 0.1.0
license: MIT
---

# Market Monitoring

## Purpose

Answer "what's happening lately?" — the news-and-price layer that sits on top of
the fundamentals. This skill is modal: the user's intent picks the mode. It
covers recent price action, headline triage, earnings reactions, insider and
institutional flows, filing changes, and multi-ticker watchlists. Every run
ships a self-contained HTML file — a monitor card for a single name, a
re-openable dashboard for a watchlist — with a short chat handoff pointing to it.

Where this skill is strongest: **Oxinion Finance** provides the live per-symbol
signal (price, RSI, trend, score, BUY/HOLD/SELL) for Top 500 names, and
`analyze_portfolio()` powers the user's own watchlist with those same signals.
Everything else — news, insider, 13F, filings — is web and SEC EDGAR. The MCP
carries none of that, so the skill must work fully without it.

## Monitoring modes

Pick the mode from intent; you can chain several in one run.

- **Price analysis** — Returns over multiple windows (1D, 1W, 1M, 3M, YTD, 1Y),
  max drawdown, and annualized volatility. Compute these from a price series in
  code, never by eye. Pull the live snapshot from Oxinion when the name is in
  the Top 500; otherwise take prices from the web.
- **News analysis** — Recent headlines ranked by materiality (1 = major …
  5 = routine) and tagged by event type: earnings, guidance, M&A, buyback,
  regulatory, legal, executive, restructuring. Cite each headline's URL. Web
  only.
- **Earnings reaction** — The stock's raw price move around the last one or two
  prints and the post-earnings drift over the following days. Reactions are
  raw, not market- or sector-adjusted; say so. Report date, surprise direction
  if known, the 1-day move, and the drift.
- **Insider activity** — Open-market buys and sells separated from grants,
  option exercises, and tax-withholding dispositions, because only discretionary
  open-market trades carry a real signal. Source: SEC Form 4 via EDGAR. US-only.
- **Institutional / 13F changes** — Notable new positions, adds, trims, and
  exits from quarterly 13F filings. Lagged up to 45 days after quarter-end; note
  the as-of date. US-only, EDGAR.
- **Filing analysis** — Index the recent 10-K, 10-Q, and 8-K filings and
  summarize what changed (new risk factors, guidance, item 8.01 events).
  Source: EDGAR primary.
- **Watchlist monitor** — Many tickers grouped by attention level (who moved,
  what filed, what hit the news). One data pass feeds both a chat summary and an
  HTML dashboard built for re-opening each morning.

## Data sources

- **Oxinion Finance MCP** — the trusted live layer for Top 500 names.
  - `get_stock_signal(symbol)`: daily price, rsi, trend, score, signal
    (BUY/HOLD/SELL), index_names, updated_at. This is the core price/signal
    source for this skill.
  - `get_stock_fundamentals(symbol)`: weekly QVMG (pe, pb, roe, roic,
    gross_profitability, fcf_yield, price_momentum, revenue_cagr, eps_growth,
    updated_at). Use when a mode needs a valuation or quality touchpoint. Values
    are decimal fractions — 0.42 means 42%; multiply rate factors by 100 and add
    "%", show pe/pb as `x` (e.g. `pe 24.1x`).
  - `analyze_portfolio()`: the authenticated user's holdings and target
    allocation enriched with live signals — the backbone of a personal
    watchlist.
  - `get_autopilot()`: dry-run rebalance preview only; it never trades. Mention
    it only if the user asks "what would rebalancing do?"
  - If a symbol is outside the Top 500 the tool errors; a missing API key or an
    unreachable server fails the same way. In every failure case, skip the
    Oxinion block silently and lean on the web — never backfill a signal from
    memory.
- **Web + SEC EDGAR** — everything the MCP does not have: news, insider (Form 4),
  13F, and filings (10-K/10-Q/8-K). Treat EDGAR as the primary source for
  anything filing-based.

## Workflow

1. **Read the ask.** Identify the ticker(s) and which mode(s) apply. One name or
   many? Which windows or filings matter?
2. **Pull the Oxinion signal** for each Top 500 name via `get_stock_signal`. For a
   personal watchlist call `analyze_portfolio()` once. On any error, drop the
   Oxinion block for that name and continue on the web.
3. **Gather the web/EDGAR layer** the chosen modes need: headlines, Form 4, 13F,
   filing index. Keep source URLs as you go.
4. **Compute in code.** For returns, drawdown, and volatility, load the price
   series and run a script — do not do the arithmetic mentally.
5. **Score and classify.** Rank headlines 1–5 by materiality; tag each by event
   type; split insider trades into open-market vs grants/exercises/tax.
6. **Assemble the deliverable.** Chat summary by default; add the HTML dashboard
   for watchlist runs.
7. **Attribute every number.** No orphan figures (see Provenance).

## Deliverables

**Every run ships a self-contained HTML file** (inline CSS/JS, no external
assets, openable straight from disk) — a single name produces a monitor card, a
list produces the dashboard. Follow the HTML with a short chat handoff (one or
two lines: the headline finding and a pointer to the file), not a full re-typing
of the card. The file is the product; the chat is the pointer.

**Single name — monitor card (HTML).** A one-page card built from the run:

- **Header**: company, ticker, run date, and the mode(s) covered.
- **Signal / price band**: the Oxinion signal badge (BUY/HOLD/SELL) + RSI + trend
  when the name is in the Top 500; if it's outside the universe or the MCP
  failed, show the web price and say "no Oxinion signal" rather than guessing.
- **Price stats**: the return windows and drawdown/volatility you actually
  computed from a price series; mark any you couldn't compute `needs
  verification` rather than eyeballing.
- **Earnings reaction** (when relevant): date, beat/miss, the raw 1-day move and
  the drift, labelled raw.
- **Ranked headlines**: the 1–5 materiality list with event-type tags, each
  linked to its URL.
- **Coverage-limits note**: US-only insider/13F, foreign-private-issuer Form 4
  exemptions, Oxinion universe — stated as limits, never as "no activity".
- **Provenance footer**: run time and per-source attribution.

**Watchlist — dashboard (HTML).** A self-contained, single-file page built from
one data pass, designed to be re-opened each morning. Include:

- One row per ticker grouped by attention level (High / Medium / Low).
- **Sortable return columns** (1D, 1W, 1M, 3M, YTD, 1Y).
- **Price sparklines** per ticker.
- The Oxinion signal badge (BUY/HOLD/SELL) and RSI where available; leave blank
  for non-Top-500 names rather than guessing.
- A compact "what's new" cell: top-ranked headline, latest filing, insider flag.
- A footer stamp with the run time and per-source provenance.

Cap a watchlist run at a sensible **~25 tickers** so one data pass stays fast and
rate-limits behave; if the user lists more, batch and say so.

## Materiality scoring rubric (1–5)

Rank each headline by likely impact on the investment case:

- **1 — Major.** Moves the thesis: M&A, CEO/CFO change, guidance cut or raise,
  large buyback/dividend action, major regulatory or legal ruling, restatement.
- **2 — Significant.** Earnings beat/miss, meaningful new product or contract,
  analyst-day guidance, sizable capital raise.
- **3 — Notable.** Mid-size partnerships, management commentary, notable
  price-target or rating changes.
- **4 — Minor.** Routine product news, small contracts, conference appearances.
- **5 — Routine.** Reiterations, boilerplate PR, scheduling notices.

Tag every headline with an event type (earnings / guidance / M&A / buyback /
regulatory / legal / executive / restructuring) alongside its score.

## Coverage limits

- **Insider (Form 4) and 13F are US-only.** For non-US names, state this as a
  coverage limit — never report it as "no insider activity" or "no institutional
  changes."
- **Oxinion signals cover the Top 500 only.** Outside that universe, or on any
  MCP failure, there is no live signal; use web prices and say so.
- **13F is lagged** up to 45 days after quarter-end; always show the as-of date.
- **Earnings reactions are raw**, not market- or sector-adjusted.
- **~25 tickers per run** keeps a single pass sane; batch beyond that.

## Provenance

Every number ties to a source:

- Uploaded document → cite the page.
- Web → URL only; never invent a page number.
- Oxinion MCP → label `Source: Oxinion Finance` plus the field's `updated_at`.
- Can't verify → mark `needs verification`; never fabricate a figure.

## Common mistakes

| Mistake | Do instead |
| --- | --- |
| Computing returns/drawdown/volatility in your head | Run code over the price series |
| Backfilling an Oxinion signal from memory after an error | Skip the block; use web prices and note it |
| Calling a non-US name's empty Form 4 "no insider activity" | State US-only coverage limit |
| Treating stale 13F as current | Show the as-of date; flag the ~45-day lag |
| Market-adjusting an earnings reaction | Report the raw move and label it raw |
| Mixing option grants/exercises into "insider buying" | Separate open-market trades from grants/exercises/tax |
| Inventing a page number for a web headline | URL only for web sources |
| Dumping 60 tickers into one watchlist pass | Cap ~25; batch and say so |
| Treating `get_autopilot()` as a trade | It is a dry-run preview only |

---

*Not investment advice. Informational analysis only; verify before acting.*
