---
name: financial-analysis
description: >-
  Use this skill whenever someone wants to understand a company's actual numbers
  — "analyze the financials of [ticker]", "break down [company]'s income
  statement", "how are their margins trending", "is the FCF real", "walk me
  through this 10-K's cash flow", "what's their ROIC / ROE / leverage", "how did
  last quarter's earnings look", "did they beat", "what are they doing with the
  cash", "buybacks or dividends?", or when they upload a 10-K / 10-Q / annual
  report and want the statements read. Trigger even if they don't say
  "financial analysis" — any request that turns on revenue, margins, cash flow,
  balance-sheet strength, earnings surprises, or capital allocation belongs here.
  For a plain-English "what does this company do" start with company-understanding;
  for what the business is worth use valuation; for balance-sheet fragility and
  earnings-quality red flags use risk-quality; for price action and macro use
  market-monitoring; to assemble a full thesis use investment-research.
version: 0.1.0
license: MIT
---

# Financial Analysis

## Purpose

Explain a company through its numbers. Read the three statements (income
statement, balance sheet, cash flow), track multi-year trends (revenue, margins,
free cash flow), compute the ratios that matter (ROIC, ROE, margins, leverage,
growth), interpret the most recent earnings and any surprise, judge cash-flow
quality (FCF, cash conversion, capex intensity), and lay out how management
allocates capital (buybacks, dividends, M&A, debt, capex).

This is the "read the books" skill. It does not tell you what the company is
worth (that's valuation) or whether the accounting is trustworthy at the fraud
level (that's risk-quality) — but it will flag when the numbers stop making
sense, because a company whose operating income and free cash flow diverge for
years is telling you something.

## Input modes

Two dimensions: depth and source.

**Depth** — both paths ship a self-contained HTML card; depth sets how much goes in it.
- **Quick pull** — one or two questions ("what's their operating margin trend",
  "did they beat last quarter"). Build a **compact** HTML card answering just
  that — the relevant table plus a one-line read — and a one-line chat handoff.
- **Deep dive** — "analyze the financials" / a full statement read. Build the
  **full** HTML card (see Deliverable). This is the default when the ask is
  open-ended.

**Source**
- **Uploaded filing** — a 10-K, 10-Q, or annual report is attached. This is the
  most precise mode: cite the page for every number.
- **Web** — no upload. Pull primary filings from SEC EDGAR
  (https://www.efts.sec.gov/LATEST/search-index?q= and the company filing pages).
  Cite the URL. Never invent a page number for a web source.

Ask which only if genuinely ambiguous; usually the attachment (or lack of one)
decides it.

## Data sources

Ranked by trust for this skill:

1. **Uploaded filings** — the primary source when present. Numbers come straight
   from audited statements. Cite the page: `Source: 10-K FY2024, p.62`.
2. **SEC EDGAR via web** — primary source when nothing is uploaded. Cite the URL,
   no page number.
3. **Oxinion Finance MCP (QVMG factor layer)** — a cross-check, not the spine.
   For Oxinion Global Top 500 names it gives clean, current factor values (roe,
   roic, pe, pb, fcf_yield, revenue_cagr, eps_growth) you can hold up against
   what you computed from the statements. Label `Source: Oxinion Finance` plus
   the field's `updated_at`.

The Oxinion MCP has **no** filings, no full statements, no news, no historical
time-series. Statements and history always come from source 1 or 2.

### The Oxinion layer (optional cross-check)

Universe is the Oxinion Global Top 500. Outside it the tool errors; with no API
key or an unreachable MCP it also fails. In **either** case, skip the Oxinion
block entirely and rely on uploaded docs / web — the analysis must stand on its
own. Never backfill Oxinion numbers from memory.

- `get_stock_fundamentals(symbol)` — WEEKLY QVMG factors: pe, pb, roe, roic,
  gross_profitability, fcf_yield, price_momentum, revenue_cagr, eps_growth,
  updated_at. Use it to sanity-check your computed ROE/ROIC and growth rates.
- `get_stock_signal(symbol)` — DAILY signal: price, rsi, trend, score, signal
  (BUY/HOLD/SELL), index_names, updated_at. Mostly market-monitoring's turf;
  include only the price/date if you need a reference point.

**Reading the factors — they are decimal fractions.** `0.42` means 42%. For the
rate factors (roe, roic, gross_profitability, fcf_yield, price_momentum,
revenue_cagr, eps_growth) multiply by 100 and add `%`. Show `pe` and `pb` as `x`
multiples (e.g. `18.4x`), never as percentages.

**Banks and insurers** return `null` for roic, gross_profitability, and
fcf_yield — that is expected, not missing data. Omit those rows cleanly rather
than showing null.

When your statement-derived ROE and the Oxinion ROE disagree materially, say so
and show both — different periods and definitions explain most gaps, but the
reader deserves to see the divergence.

## Provenance (non-negotiable)

Every number ties to a source. No exceptions.

- Uploaded doc → cite the page: `Source: 10-K FY2024, p.62`.
- Web → URL only, never a fabricated page number.
- Oxinion MCP → `Source: Oxinion Finance` + the data's `updated_at`.
- Can't verify a figure → write `needs verification`. Never fabricate, never
  fill a gap from memory.

A cited wrong number can be checked; an uncited number is worthless.

## Workflow

**1. Gather.** Identify the company and mode. If a filing is uploaded, locate the
statements and note their page ranges. If web, pull the latest 10-K plus the most
recent 10-Q from EDGAR. For a Top 500 name, optionally call
`get_stock_fundamentals` for the factor cross-check — and degrade gracefully if it
fails.

**2. Compute via code — never in your head.** Ratios, YoY changes, CAGRs, margin
series, cash-conversion, and net-debt are all arithmetic on extracted line items.
Do that arithmetic by running code (write the raw figures into a small script and
print the results), so the math is reproducible and correct. Doing multi-year
percentage math mentally is how errors get shipped.

**3. Build.** Assemble the deliverable — a full HTML card for a deep dive, a
compact HTML card for a quick pull — plus a one-line chat handoff pointing to
it. Attach the source to every metric as you go, not at the end.

## Deliverable

**Every run ships a self-contained HTML card** — tables plus small inline charts
(SVG or lightweight inline JS; no external calls), openable straight from disk,
followed by a one- or two-line chat handoff (not a re-typing of the card). A
deep dive fills the full section set below; a quick pull produces a compact card
scoped to just what was asked. Either way: show **YoY changes**, cite every
metric, and warn when operating income and FCF diverge.

Sections, in order:

**1. Statements (condensed).** The three statements trimmed to what matters:
- Income statement: revenue, gross profit, operating income, net income, EPS.
- Balance sheet: cash, total debt, net debt, equity, key working-capital lines.
- Cash flow: operating cash flow, capex, free cash flow, financing flows
  (buybacks, dividends, debt issued/repaid).

**2. Multi-year trends.** 3–5 years where available. Revenue growth, gross /
operating / net margins, and FCF as a series, with small inline charts. Trend
beats any single year.

**3. Ratios.** ROIC, ROE, gross/operating/net margin, leverage (net debt /
EBITDA, interest coverage), and growth (revenue CAGR, EPS growth). Show the math
inputs so a reader can trace each ratio.

**4. Recent earnings.** The latest quarter: revenue and EPS, YoY, and — if a
consensus figure is available from web — the surprise vs. expectations. Note
guidance changes. If no reliable consensus exists, say so rather than guessing.

**5. Cash-flow quality.** Is the FCF real? Cash conversion (FCF / net income, or
OCF / net income), capex intensity (capex / revenue), and the gap between
reported earnings and cash generated. This is where you raise the
operating-income-vs-FCF warning if they've drifted apart.

**6. Capital allocation.** Where the cash goes: buybacks, dividends, M&A, debt
paydown, capex. Net share-count trend tells you whether buybacks actually shrank
the share base or just offset dilution.

### Banks and insurers — recast the frame

FCF, net debt, and capex intensity are meaningless for a bank. Swap the frame:
- Income: net interest income, net interest margin, fee income, provisions,
  efficiency ratio.
- Returns: ROE and ROA (not ROIC), plus ROTCE where available.
- Capital & risk: CET1 ratio, book value / tangible book value per share.
- Drop the FCF / cash-conversion / net-debt sections entirely.

Insurers: combined ratio, float, investment income, book value per share.

## Interpretation guidance (rough anchors)

Anchors, not verdicts — context (sector, cycle, capital intensity) rules. State
them as ranges and always relative to the company's own history and peers.

- **ROIC** — below the cost of capital (~8–10%) value is being destroyed;
  mid-teens is solid; 20%+ sustained signals a real moat. Compare ROIC to WACC,
  not to zero.
- **ROE** — 15%+ is strong, but check whether leverage is inflating it (ROE up
  while ROIC flat = balance-sheet leverage, not operating improvement).
- **Gross margin** — direction and stability matter more than the level;
  software 70%+, retail single-to-low-double digits — never compare across
  sectors.
- **Operating margin trend** — expanding on flat revenue = operating leverage;
  contracting on rising revenue = a cost or pricing problem.
- **Net debt / EBITDA** — under ~2x comfortable, 3–4x stretched, above that
  fragile — softer for utilities/telecom, harder for cyclicals.
- **Interest coverage** (EBIT / interest) — under ~3x is a warning.
- **Cash conversion** (FCF / net income) — near or above 1.0 over time means
  earnings are backed by cash; persistently well below 1.0 means earnings aren't
  converting — dig into working capital, capex, or accruals.
- **Revenue CAGR / EPS growth** — EPS growing faster than revenue is margin
  expansion or buybacks; slower is dilution or margin compression.

Cross-check every self-computed factor against the Oxinion QVMG value when
available, and flag material gaps rather than silently picking one.

## Common mistakes

| Mistake | Do instead |
| --- | --- |
| Doing multi-year percentage math in your head | Compute in code; print the numbers |
| Reporting a metric with no source | Cite page / URL / Oxinion + `updated_at`, or mark `needs verification` |
| Treating Oxinion decimals as percent-of-percent | `0.42` = 42%; multiply rate factors by 100; pe/pb stay `x` |
| Showing `null` roic/fcf for a bank | Recast the frame (NII, ROE, CET1); omit the null rows |
| Trusting net income while FCF lags for years | Warn on the operating-income vs. FCF divergence |
| Judging one year in isolation | Show the 3–5 year trend |
| Backfilling Oxinion numbers from memory when the MCP fails | Skip the Oxinion block; lean on filings/web |
| Reading a rising ROE as an operating win | Check whether leverage, not ROIC, drove it |
| Inventing a page number for a web figure | Web = URL only |
| Counting gross buybacks as share-count reduction | Track net shares outstanding for real dilution effect |

---

*Not investment advice. This skill analyzes reported financials for
understanding, not as a recommendation to buy or sell any security.*
