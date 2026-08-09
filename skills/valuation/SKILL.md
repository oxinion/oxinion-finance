---
name: valuation
description: >-
  Use this whenever someone asks whether a stock is cheap, expensive, or fairly
  valued, or wants a price target, intrinsic value, or fair-value estimate. Triggers
  on "is [ticker] cheap?", "what's it worth?", "build me a DCF", "what growth is the
  price pricing in?", "reverse DCF", "is the P/E justified?", "how does it compare to
  peers?", "sum-of-the-parts", "dividend discount", "what return can I expect?", or
  any request that ends in a value judgment on price. Use it even if they don't say
  "valuation" or "DCF" — "should I buy at $180" and "is this overvalued" are the same
  ask. This is the "so is it cheap?" skill. For what the company does and how it earns,
  route to company-understanding; for margins, growth, and balance-sheet drivers that
  FEED these models, route to financial-analysis; for downside and moat durability,
  route to risk-quality; for live price levels and technicals, market-monitoring; and
  to assemble a full thesis, investment-research.
version: 0.1.0
license: MIT
---

# Valuation

## Purpose

Answer one question: **at today's price, is this a good deal?** You do that by
estimating what the business is worth (or what the price already assumes) and
comparing it to the quote. Valuation is not a single formula — it is choosing the
method that fits the *question being asked*, stating every forward assumption as the
user's own, and stress-testing the one variable that actually swings the answer.

The output is never a single number pretending to be truth. It is a **range** —
base, bull, bear — plus a sensitivity grid, so the user sees how fragile the answer
is to inputs they can argue with.

## Decision guide — pick the method from the question

| The user is really asking... | Use | Why |
|---|---|---|
| "What is it worth on its cash flows?" | **Forward DCF** | Intrinsic value from projected free cash flow. |
| "Is the price crazy? What does it assume?" | **Reverse DCF** | Solve for the growth/margin the current price implies, then judge plausibility. |
| "Cheap vs peers / vs its own history?" | **Relative multiples** | Fast, market-anchored; needs a comparable set and a history. |
| "I'm buying it for the income." | **Dividend discount (DDM)** | Stable, dividend-paying names where payout is the return. |
| "What return will I actually earn?" | **Expected-return / IRR** | Combines yield + growth + multiple change into a forward return. |
| "It's really several businesses." | **Sum-of-the-parts (SOTP)** | Multi-segment names where one method hides the mix. |

Single-segment business? Skip SOTP and route to a whole-company method above.
When unsure, default to **reverse DCF** — it is the hardest to fool yourself with,
because the market gives you the answer and you only judge whether it is reasonable.

Always add two context layers regardless of method:
- **Historical multiple range** — current multiple vs its own 5–10y band.
- **Peer multiples** — the same multiple across a named comparable set.

## Inputs

- **Ticker or company.** Confirm the exact listing.
- **Forward assumptions from the user** — growth rate, margin path, discount rate
  (WACC or required return), terminal growth/exit multiple, dividend growth. If the
  user has none, propose defaults, label them as *your suggestions*, and make the
  user own them before you compute.
- **Uploaded filings/models** for historical drivers (revenue, margins, capex, FCF).
- **A peer list** — from the user, or proposed and confirmed.

## Data sources and provenance

Every number ties to a source. No exceptions.

- **Uploaded doc** → cite the page (e.g. `10-K FY24, p.62`).
- **Web** → cite the URL only. Never invent a page number for a web source.
- **Oxinion Finance MCP** → label `Source: Oxinion Finance` with its `updated_at`.
- **Can't verify** → write `needs verification`. Never fabricate a figure.

### Oxinion current-multiple anchor (Top 500 only)

For an Oxinion Global **Top 500** name, pull the current multiples as your
point-in-time anchor:

- `stock-fundamentals(symbol)` → `pe`, `pb`, `fcf_yield` (plus `roe`, `roic`,
  `revenue_cagr`, `eps_growth` if you need drivers). All ratios are **decimal
  fractions**: `fcf_yield` 0.045 → 4.5%; multiply rate factors by 100 and add `%`.
  Show `pe` and `pb` as `x` multiples (e.g. `18.4x`), never as `%`.
- `stock-analysis(symbol)` → current `price` for the quote you value against.

Treat these as **current, point-in-time only.** The MCP carries **no multiple
history and no time-series** — so the historical range and the peer set must come
from web or uploaded docs. Never backfill an Oxinion history from memory.

If the name is outside the Top 500, or there is no API key, or the MCP is
unreachable, the tool errors — **skip the Oxinion block entirely** and anchor the
current multiple from web/docs instead. The skill works fully without the MCP;
never substitute a remembered Oxinion number.

Banks and insurers return `null` for `roic`, `gross_profitability`, and `fcf_yield`
— for those, lean on P/B, ROE, and a DDM or residual-income lens rather than FCF.

## Workflow

**1. Assumptions first, explicit and owned.** Build an assumptions table before any
math. Every row: the value, whose it is (user vs. your proposed default), and a
one-line rationale or source. If the user won't commit to a number, present base/
bull/bear ranges rather than picking silently.

**2. Compute by running code, never in your head.** Write the DCF, reverse solve,
multiple math, DDM, or IRR as a small script and run it. Discounting, terminal
value, and IRR are error-prone by hand — running code makes the math auditable and
lets you re-run scenarios cheaply. Show the code's outputs, not mental arithmetic.

**3. Sensitivity on the swing variable.** Identify the input the answer is most
fragile to — usually discount rate × terminal growth for a DCF, or the exit
multiple for relative work — and build a 2-D grid varying it. This replaces false
precision with an honest picture of the range.

**4. Build the deliverable.** Assemble the HTML card / tables (below).

## Deliverable

A self-contained HTML card (inline styles, no external assets) containing:

1. **Header** — ticker, current price with source, the method chosen and one line
   on *why this method fits the question*.
2. **Assumptions table** — each input, its value, owner (user / proposed default),
   and source or rationale. This is the heart of the card; make it prominent.
3. **Valuation output** — **base / bull / bear** fair value (or implied return, or
   implied growth for a reverse DCF), each vs. the current price, with implied
   upside/downside %.
4. **Current-multiple anchor** — Oxinion `pe` / `pb` / `fcf_yield` (labeled
   `Source: Oxinion Finance` + `updated_at`) when available, beside the historical
   range and peer multiples from web/docs.
5. **Sensitivity grid** — the 2-D table on the swing variable, base case highlighted.
6. **Provenance footnotes** — every figure tied to its source; `needs verification`
   where unconfirmed.

Prefer a card the user can read in one screen; put supporting tables below the fold.

## Guardrails

- **The assumptions are the user's, not yours.** State them explicitly, attribute
  them, and never bury a forecast inside a "fair value." A DCF is an opinion with
  math attached.
- **No false precision.** Report ranges, not a single decimal-perfect target.
  "$150–$185, base ~$168" beats "$171.42". Round to the precision the inputs earn.
- **Oxinion multiples are current-only.** Use them as the present anchor; build
  history and peers elsewhere. Never fabricate an Oxinion time-series.
- **Composable with financial-analysis.** The growth, margin, and capex drivers that
  feed these models are that skill's job — pull them from there rather than
  guessing, and keep assumptions consistent across both.
- **Reverse-DCF sanity check.** For any bullish forward DCF, run the reverse DCF too
  and ask: is the implied growth actually achievable given the drivers?

## Common mistakes

| Mistake | Do instead |
|---|---|
| Single point "fair value" | Always base/bull/bear + sensitivity grid. |
| Discounting or IRR by hand | Run code; show the outputs. |
| Terminal value doing all the work | Report TV as a % of total value; flag if >75%. |
| Comparing a multiple with no history | Anchor to the name's own 5–10y band, not just today. |
| Peers picked to flatter the answer | Name the comp set and its selection logic up front. |
| Using FCF yield on a bank | Null on Oxinion for a reason — use P/B, ROE, DDM. |
| SOTP on a single-segment name | Route to a whole-company method. |
| Presenting your assumptions as facts | Attribute every forward input to the user. |
| Backfilling Oxinion history from memory | It has none; get history from web/docs. |

---

*Not investment advice. Valuation outputs are scenario estimates built on stated
assumptions, not recommendations to buy or sell.*
