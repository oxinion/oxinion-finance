---
name: investment-research
description: >-
  Use this to produce a full buy-side-style research note on a public company — the umbrella
  deliverable that folds history, business model, competitive landscape, moat, financials and
  KPIs, valuation, price action, the latest earnings, management and ownership, and recent news
  into one self-contained HTML report with a bull-vs-bear box and a dated Sources list. Trigger
  whenever someone says "write a research note on [ticker]", "give me a full write-up on
  [company]", "deep dive on [company]", "should I look at [ticker]", "build me an investment
  memo", "research [company] end to end", or hands you a 10-K and asks for the complete picture —
  even if they don't say the words "research note" or "memo". This skill orchestrates the logic of
  the single-topic siblings; when a reader wants only one slice, point them to the deeper cuts:
  company-understanding (what the business is), financial-analysis (the statements), valuation
  (what it's worth), risk-quality (what can break it), and market-monitoring (the live tape).
version: 0.1.0
license: MIT
---

# Investment Research

## Purpose

Produce one coherent, buy-side-style research note (~1,500-2,000 words) that a reader can act on
without opening ten tabs. This is the umbrella skill: it combines the work of the five
single-topic siblings into a single narrative and a single deliverable. Every number is tied to a
source, both sides of the debate get equal airtime, and the output is a self-contained HTML file.

The note answers one question — "what is the honest, evidence-based case for and against owning
this?" — by walking a fixed section order from business model through to recent news.

## When to use

- The reader wants the whole picture, not one slice: "full write-up", "deep dive", "research note".
- They are sizing up a position and need the bull and bear cases laid side by side.
- They handed you a filing or transcript and want it woven into a complete company view.

If the ask is narrower, hand off to the deeper cut instead of writing the full note:

| Reader wants only… | Use sibling |
|---|---|
| What the business does, its history and moat | company-understanding |
| The financial statements dissected | financial-analysis |
| What it's worth vs history and peers | valuation |
| What can break the thesis | risk-quality |
| The live signal and price tape | market-monitoring |

## The note's section-by-section structure

Write these sections in this order. Each is a few tight paragraphs, not a data dump.

1. **History and business model** — how the company came to be, how it makes money, the segments
   and revenue mix. Draw on company-understanding logic.
2. **Competitive landscape** — who it competes with, market share, where the fight is.
3. **Moat** — the durable advantages (network effects, scale, switching costs, brand, IP) and how
   strong the evidence is that they persist.
4. **Full financial and KPI analysis** — growth, margins, returns on capital, cash generation,
   balance sheet, and the operating KPIs that matter for this business. Draw on
   financial-analysis logic.
5. **Valuation vs its own history and peers** — where multiples sit relative to the company's own
   range and its peer set; what's priced in. Draw on valuation logic.
6. **Price action and the bull/bear debate** — the recent tape, the live signal, and the framed
   two-sided argument. Draw on market-monitoring and risk-quality logic.
7. **Latest earnings** — the most recent quarter: prints vs expectations, guidance, transcript
   color.
8. **Management and ownership** — leadership track record, insider and institutional ownership,
   incentive alignment from the proxy.
9. **Recent news** — the last few months of material developments.

## Data-sourcing rules

Two source classes. Know which carries which, and never blur them.

**Oxinion Finance MCP first, for the numbers it carries.** Universe is the Oxinion Global Top ~500
("Top 500"). For an in-universe name, pull the live signal and the factor read before reaching for
the web:

- `get_stock_signal(symbol)` — DAILY: price, rsi, trend, score, signal (BUY/HOLD/SELL),
  index_names, updated_at. This is the live tape for the price-action section.
- `get_stock_fundamentals(symbol)` — WEEKLY QVMG: pe, pb, roe, roic, gross_profitability, fcf_yield,
  price_momentum, revenue_cagr, eps_growth, updated_at. Rate factors are decimal fractions
  (0.42 = 42% — multiply by 100 and add "%"); pe and pb read as `x`. Banks and insurers return
  null for roic, gross_profitability, and fcf_yield — say so rather than inventing figures.

Label every Oxinion figure `Source: Oxinion Finance` with its `updated_at` date.

If the symbol is outside the Top 500, or there is no API key, or the tool is unreachable, the call
errors. In every one of those cases, skip the Oxinion block entirely and lean on web and uploaded
docs. Never backfill an Oxinion-shaped number from memory. The note must read as complete without
the MCP.

**Web, always attributed, for everything the MCP lacks.** The MCP has no transcripts, no
consensus, no proxy/compensation, no filings, no news, no insider/13F, no full statements, and no
history. All of that is web-sourced — SEC EDGAR is the primary for filings, proxies, and
statements:

- Uploaded document → cite the page number.
- Web source → URL plus the access date. Never invent a page number for a web source.
- Oxinion MCP → `Source: Oxinion Finance` plus `updated_at`.
- Can't verify a figure → mark it `needs verification`. Never fabricate.

## Workflow

1. **Gather across sources.** For a Top 500 name, call `get_stock_signal` and `get_stock_fundamentals`
   first and record each value with its `updated_at`. Then collect what the MCP can't give:
   filings and statements from SEC EDGAR, the latest transcript, consensus, the proxy, and recent
   news — each captured with its URL and access date, or page number for uploads. If the MCP is
   unavailable or the name is out of universe, source the price and multiples from the web instead
   and attribute them.
2. **Compute via code, never in your head.** Growth rates, margins, multiple ranges, peer medians,
   CAGRs — write and run a small script so the arithmetic is reproducible and auditable. Show the
   inputs. Do not do multi-step math mentally.
3. **Draft the note.** Follow the nine-section order. Keep it to ~1,500-2,000 words. Lead each
   section with the conclusion, then the evidence, then the caveat.
4. **Balance bull and bear.** Before finalizing, write the bull case and the bear case at equal
   length and equal seriousness. If one side is thin, you haven't researched it yet — go back.

## Deliverable structure

A single self-contained HTML research note with a clean typographic layout:

- **Header** — company, ticker, date, one-line thesis, and the live signal (BUY/HOLD/SELL) if the
  name is in the Top 500.
- **Body** — the nine sections above, in order, with clear headings.
- **A few inline charts** — e.g., revenue and margin trend, the valuation multiple vs its own
  history, the price line. Keep them simple and labeled; generate the values via code.
- **An explicit bull vs bear box** — two columns, side by side, equal weight.
- **Sources list** — every source with its date: web URLs with access dates, uploaded-doc page
  citations, and Oxinion figures marked `Source: Oxinion Finance` with `updated_at`. Anything
  unconfirmed carries `needs verification`.

Inline all CSS; keep the file openable on its own with no external assets.

## Guardrails

- **Evenhanded.** The bull and bear cases get equal length and equal rigor. A note that only
  argues one direction is not finished.
- **Not investment advice.** State it plainly in the note.
- **No fabricated numbers.** Every figure traces to Oxinion (with `updated_at`), a web URL (with
  date), or an uploaded page. If you can't verify it, mark it `needs verification` and move on.
- **Don't blur sources.** Never present a web or remembered figure as an Oxinion figure, or an
  Oxinion figure as a filing.
- **Math in code.** Reproducible arithmetic only.

## Common mistakes

| Mistake | Do instead |
|---|---|
| Backfilling Oxinion fields from memory when the name is outside the Top 500 or the MCP is down | Skip the Oxinion block; source and attribute from the web |
| Reading QVMG rate factors as whole numbers (0.42 shown as "0.42%") | Multiply decimal fractions by 100 (0.42 → 42%); pe/pb are `x` |
| Inventing a page number for a web source | URL plus access date only; pages are for uploaded docs |
| Quoting roic or fcf_yield for a bank or insurer | Those fields are null for financials — say so |
| Doing CAGRs and peer medians in your head | Compute via a script and show the inputs |
| Writing a one-sided note | Bull and bear at equal length before you finalize |
| Presenting a remembered figure as verified | Mark it `needs verification` |
| Treating consensus, transcripts, or the proxy as MCP data | Those are web-only, attributed with URL and date |

---

*This note is for research and education only and is not investment advice.* Present both sides:
the bull case for durable growth and the bear case for the risks that unwind it, weighted by the
evidence — and let the reader decide.
