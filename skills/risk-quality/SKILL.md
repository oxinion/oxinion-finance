---
name: risk-quality
description: >-
  Judge how sound and how risky a company is before trusting its numbers. Use this
  whenever someone asks "is this company safe", "how risky is [ticker]", "could this
  go bankrupt", "is the balance sheet healthy", "are the earnings real / high quality",
  "run an Altman Z / Piotroski / Beneish / accruals check", "how much debt can they
  handle", "is management cooking the books", or wants a red-flag / distress / quality
  screen — even if they never say the words "risk" or "quality" and just ask whether a
  stock is a landmine. Pairs with the sibling skills: company-understanding (what the
  business is), financial-analysis (statement mechanics), valuation (what it's worth),
  market-monitoring (live signal), and investment-research (the full write-up). Reach
  for those when the question is about worth or story rather than soundness and survival.
version: 0.1.0
license: MIT
---

# Risk & Quality

## Purpose

Separate a good business from a fragile one, and a real earnings stream from an
engineered one. Four lenses: **financial health** (can it pay its bills), **business
quality** (are the returns real and durable), **earnings quality** (does reported
profit turn into cash), and **distress risk** (bankruptcy and manipulation flags).
Output a plain-English verdict — Strong / Watch / Distress — not a wall of ratios.

This is a soundness screen, not a price call. Valuation risk is noted, but "what is it
worth" belongs to the valuation skill.

## Inputs & modes

- **Ticker only** → pull the latest figures from the web (most recent 10-K/10-Q,
  investor page). Cite the URL. Never invent a page number for web data.
- **Uploaded filing** (10-K, 10-Q, annual report) → the precise path. Cite the page
  for every number.
- **Oxinion Top 500 name** → layer the QVMG quality factors on top as a cross-check.

Always state which mode you ran. If a required input is missing (e.g. no cash flow
statement), mark the dependent score `needs verification` rather than guessing.

## Data sources

1. **Uploaded docs** — highest trust for fundamentals. Cite page numbers.
2. **Web** — filings, investor relations, reputable aggregators. Cite the URL only;
   never fabricate a page.
3. **Oxinion Finance MCP** — a cross-check layer, Top 500 only:
   - `get_stock_fundamentals(symbol)` (weekly QVMG): `roe`, `roic`, `gross_profitability`,
     `fcf_yield`, `pe`, `pb`, `revenue_cagr`, `eps_growth`, `price_momentum`,
     `updated_at`. Values are **decimal fractions** — 0.42 = 42%; multiply rate
     factors by 100 and add `%`. `pe`/`pb` are `x` multiples. Banks and insurers
     return `null` for `roic`, `gross_profitability`, `fcf_yield` — that is expected,
     not a red flag.
   - `get_stock_signal(symbol)` (daily): `price`, `rsi`, `trend`, `score`, `signal`,
     `updated_at` — context, not a quality measure.
   - Label anything from the MCP `Source: Oxinion Finance` with its `updated_at`.

The MCP has **no filings, no history, no statements, and no distress or quality
scores**. Altman Z, Piotroski F, Beneish M and Sloan accruals are **always computed
from web/doc inputs**. If the symbol is outside the Top 500, or the MCP is unreachable
or has no key, **skip the Oxinion block entirely** and rely on web/docs — never backfill
Oxinion figures from memory. The skill works fully without the MCP.

## Compute the scores in code, never in your head

Multi-input scores are error-prone by hand. Pull the raw line items, then run a short
script. Show each score **with its inputs** so the reader can audit it.

- **Altman Z-Score** (manufacturers): `1.2*(WC/TA) + 1.4*(RE/TA) + 3.3*(EBIT/TA) +
  0.6*(MktCap/TotLiab) + 1.0*(Sales/TA)`. Zones: **> 2.99 safe**, **1.81–2.99 grey**,
  **< 1.81 distress**. Use Z''-Score for non-manufacturers/services.
- **Piotroski F-Score** (0–9): nine profitability, leverage, and efficiency checks.
  **8–9 strong, 0–2 weak.**
- **Beneish M-Score**: eight-ratio manipulation model. **> -1.78 flags possible
  earnings manipulation.** Treat as a smoke alarm, not a conviction.
- **Sloan accruals ratio**: `(NetIncome − CFO − CFI) / avg TotAssets`, or the simpler
  `(NI − CFO) / avg TA`. High positive accruals = profit not backed by cash; lower is
  cleaner.

State the template you assumed (manufacturer vs. service) next to Z, since the
coefficients differ.

## The four lenses (rough anchors, not thresholds)

**1. Financial health**
- Leverage: net debt / EBITDA (< 1x conservative, > 4x stretched), debt / equity,
  debt-to-cap.
- Liquidity: current ratio (~1.5–3 comfortable), quick ratio, cash ratio.
- Coverage / solvency: interest coverage (EBIT / interest; < 2x fragile, > 6x
  comfortable), EBITDA − capex vs. interest, cash-flow-to-debt (CFO / total debt).

**2. Business quality**
- ROIC vs. WACC — value is created only when ROIC clears the cost of capital.
- Margin level and stability (gross, operating, FCF).
- FCF consistency across the cycle; durability of returns (moat, pricing power).
- Oxinion `roic`, `roe`, `gross_profitability` are the Top-500 cross-check here.

**3. Earnings quality**
- Accruals ratio (above) — the cleanest single tell.
- GAAP vs. adjusted / non-GAAP gap — a wide, persistent, growing gap is a flag.
- Cash conversion: CFO / net income near or above 1 is healthy.

**4. Distress risk**
- Altman Z, Piotroski F, Beneish M as computed above.
- Refinancing wall (near-term maturities vs. cash + FCF), covenant headroom, customer
  or supplier concentration.

**Risk taxonomy** — tag each finding: **business** (execution, concentration,
disruption), **industry** (cyclicality, regulation, secular decline), **financial**
(leverage, liquidity, refi), **valuation** (priced for perfection). This keeps
"the business is risky" distinct from "the stock is risky".

## Template-awareness and heuristics-not-verdicts

Not every ratio fits every company. **Gate out** the ones that produce nonsense rather
than forcing them:
- **Banks**: net debt/EBITDA and interest coverage are meaningless (deposits are raw
  material). Use capital ratios (CET1, tier 1), NPL / coverage ratio, ROE, ROA,
  efficiency ratio. Oxinion returns `null` for `roic`/`gross_profitability`/`fcf_yield`
  here — expected.
- **Insurers**: combined ratio, reserve adequacy, investment leverage — not EBITDA
  leverage.
- **REITs**: use FFO / AFFO and net debt / EBITDA (not net income); debt-to-assets
  and interest coverage still apply.
- Deep operating-lease or financing arms distort classic leverage — note the
  adjustment.

**Heuristics are not verdicts.** Z, F, M, and accruals are screening signals with real
false-positive rates. But treat them asymmetrically: **a credible distress or
manipulation flag OVERRIDES a strong-fundamentals read.** A company with great margins
and a Beneish M above -1.78 or an Altman Z in the distress zone lands in **Watch** or
**Distress**, not Strong — the downside of ignoring a real flag dwarfs the cost of a
false alarm. Say what would clear the flag.

## Deliverable

A **self-contained HTML card** (inline CSS, no external assets) that leads with
**zones, not numbers**:

- **Header**: company, ticker, mode (ticker / uploaded doc), date, one-line verdict.
- **Zone banner**: **Strong / Watch / Distress**, color-coded, one sentence why.
- **Four lens tiles**: financial health, business quality, earnings quality, distress —
  each a mini Strong/Watch/Distress with its two or three driving figures.
- **Scores block**: Altman Z, Piotroski F, Beneish M, accruals — **each shown with its
  inputs and zone**, plus the template assumed.
- **Oxinion cross-check** (Top 500 only): QVMG quality factors, `Source: Oxinion
  Finance` + `updated_at`. Omit the block entirely if unavailable.
- **Risk taxonomy list**: top business / industry / financial / valuation risks.
- **Provenance footer**: every number → page (docs), URL (web), or Oxinion label;
  anything unverified marked `needs verification`.

## Common mistakes

| Mistake | Do instead |
| --- | --- |
| Forcing net debt/EBITDA on a bank | Gate by template — use capital and NPL ratios |
| Computing Altman Z by hand | Run code; show every input |
| Reading Oxinion 0.18 as "0.18%" | It's a fraction — 0.18 = 18% |
| Backfilling Oxinion from memory when off-universe | Skip the block; use web/docs |
| Invoking a page number for a web figure | URL only for web; page only for uploads |
| Letting strong margins outweigh a distress/M-score flag | Flag overrides — drop to Watch/Distress |
| Treating adjusted EPS as the truth | Compare to GAAP; check the accruals ratio |
| Reporting a lone score as a verdict | Combine four lenses; state confidence |
| Guessing a missing line item | Mark `needs verification` |

---

*Not investment advice. Screening heuristics for research only; verify before acting.*
