---
name: company-understanding
description: >-
  Build a one-page "company overview card" that lets an investor understand any
  public company in about 10 minutes. Give it a ticker and it analyzes instantly
  from web sources; upload a 10-K or annual report and it does a precise,
  page-cited analysis. For Oxinion Global Top 500 names it also folds in the
  Oxinion Finance MCP — the live signal (price, RSI, trend, BUY/HOLD/SELL) and
  the QVMG factor scorecard — as an extra trusted source. Covers US-listed
  stocks. Use this whenever someone asks to understand, analyze, or summarize a
  company: "analyze [ticker]", "what does [company] do", "break down [company]",
  "company overview", "summary card", "analyze this 10-K", "walk me through this
  annual report", or names a company and wants to know how it makes money — even
  if they don't say the word "overview". This is the entry point for
  understanding a company; for deeper single-topic cuts hand off to
  financial-analysis, valuation, risk-quality, market-monitoring, or the full
  investment-research note.
version: 0.1.0
license: MIT
---

# Company Understanding

## Purpose

The most common mistake a retail investor makes is **putting money into a
business they don't actually understand**. This skill produces a **one-page card
that makes any company understandable in about 10 minutes**.

**Core principle: the business model comes first. Financials come second.** A
reader who can't say in one sentence how the company makes money isn't ready to
look at its margins.

## Absolute rules

1. **Output in English.** Keep standard finance terms as-is (Operating Income,
   Free Cash Flow); no need to over-explain common ones, but any genuinely
   specialized term gets a short plain-English gloss in parentheses the first
   time it appears.
2. **No number without a source.** This is what separates the card from a
   confident-sounding guess, and one unsourced figure makes the reader distrust
   all of them:
   - From an uploaded document → **cite the page** (e.g. `Source: 10-K FY2024,
     p.62`).
   - From web search → **cite the URL only.** Never invent a page number for a
     web source — a fake citation is worse than none.
   - From the Oxinion MCP → label it `Source: Oxinion Finance` with the data's
     `updated_at`.
   - Can't verify → write `needs verification`. Do not fabricate the number.
3. **If a 12-year-old couldn't follow it, rewrite it.** Jargon is a failure of
   explanation, not a sign of rigor.
4. **Deliver the result as a single self-contained HTML artifact** (the card),
   not a long wall of chat text. The card is the product; chat is just a one-line
   handoff.

## Input modes (two-tier)

### Mode A — ticker only (default · fast)

The user gave only a ticker or company name. Proceed from web search (plus the
Oxinion MCP if the name is in the Top 500).

- In this mode, **do not cite page numbers** — only URLs (and Oxinion, if used).
- Show at the top of the card: `📡 Web-sourced · upload a 10-K (or annual
  report) for page-cited precision`.

### Mode B — document uploaded (precise)

A 10-K, 10-Q, or annual report is attached.

- **Treat the uploaded document as the primary source.** Web search is a
  supplement only (latest price, recent news); the Oxinion MCP is a supplement
  for the market signal and factor read.
- Cite page numbers throughout.
- Show at the top of the card: `📄 Based on {document name} · page-cited`.

### Filings source

- **US-listed** → 10-K (annual), 10-Q, source: SEC EDGAR.

### Oxinion MCP layer (Global Top 500 only)

For any name in the Oxinion universe, call these in parallel with your other
research — they're a fast, model-computed cross-check the reader can trust:

- **`stock-analysis(symbol)`** — daily signal: `price`, `rsi`, `trend`, `score`,
  `signal` (BUY/HOLD/SELL), `index_names`, `updated_at`.
- **`stock-fundamentals(symbol)`** — weekly QVMG factors: `pe`, `pb`, `roe`,
  `roic`, `gross_profitability`, `fcf_yield`, `price_momentum`, `revenue_cagr`,
  `eps_growth`, `updated_at`.

Two things to remember, because they're the usual failure points:
- **QVMG ratios are decimal fractions**: `0.42` means **42%**, not 0.42%.
  Multiply the rate factors (ROE, ROIC, gross profitability, FCF yield,
  momentum, revenue CAGR, EPS growth) by 100 and add `%`. P/E and P/B are
  multiples — show as `24.1×`, never a percent.
- If the ticker isn't in the universe, the tool errors — that's fine, just
  **skip the Oxinion block entirely and lean on web/docs**. Never backfill the
  signal or factors from memory; a mixed-provenance number defeats the purpose.

---

## Card structure (keep this order)

### ① The 2-minute drill — one sentence (top, largest)

**The heart of the card.** Peter Lynch's two-minute drill: what this company
does to make money, in **one sentence a 12-year-old would understand**.

Rules:
- One sentence. Two sentences means you failed.
- No industry-speak. If you reach for "platform", "solution", or "ecosystem",
  you failed.
- It must contain: **what they make → for whom → how they get paid**.

Good:
> **Disney invents characters people fall in love with, then squeezes each one
> for a lifetime — through movies, subscriptions, toys, and theme-park tickets.**

Bad (rewrite it):
> Disney is a global diversified entertainment platform engaged in the creation
> and distribution of media and entertainment content and the operation of theme
> parks.

### ② How it makes money (a picture)

**Draw this as an SVG or HTML diagram inside the artifact.** No text lists —
the diagram is this skill's signature.

Must contain:
- **The value chain**: who they buy from (suppliers) → what they make → who they
  sell to (customers) → where the money flows.
- Each arrow labeled with the **direction of money** and **amount/share** where
  known.
- If there's a flywheel, draw it as a loop (e.g. Disney characters → films →
  merch → parks → reinforce the characters).

Visual rules:
- Three colors max. Black + a blue family + one accent.
- Text inside boxes is English, five words or fewer.
- If it doesn't read at a glance, it failed. Never exceed seven elements.

### ③ Revenue breakdown — two axes

Show **both axes**. Table plus a simple bar/pie chart.

| Axis | What it shows |
|---|---|
| **By segment** | Revenue, share (%), and operating margin per segment. "Where they earn and where they bleed" must be visible. |
| **By geography** | Revenue share by country/region. If foreign share is high → a one-line note on FX and geopolitical risk. |

If geographic data isn't disclosed, write `not disclosed`. Do not estimate it.

### ④ Customers & competitors

- **Customers**: who pays. B2C/B2B, and customer concentration — if any single
  customer is >10% of revenue, always flag it (that's a risk).
- **Competitors**: three of them, each one line on "how this company is
  different".

### ⑤ The industry's real KPIs (1–2)

Find **the metric that actually matters for this business** and show a 5-year
(minimum 3-year) trend. Not revenue or profit — the operating driver.

Examples by industry:
- Retail / restaurants → same-store sales growth.
- Streaming → subscribers, ARPU, churn.
- Theme parks → attendance, per-capita spending.
- Semiconductors → utilization, inventory turnover.
- Banks → net interest margin (NIM), delinquency rate.
- SaaS → net revenue retention (NRR), customer acquisition cost (CAC).

**Always add a one-line reason the KPI matters.** (e.g. "Same-store sales strips
out growth that's just from opening new stores — it shows whether the existing
business is actually healthier.")

### ⑥ Management & ownership (short · ≤3 lines)

- CEO name, tenure, one line of prior background.
- Is the founder still involved?
- Insider ownership / major holders.

### ⑦ Shareholder returns

- **Dividends**: yield, payout ratio, consecutive years of increases.
- **Buybacks**: dollar amount over the last three years, and whether the share
  count actually fell (buybacks that are re-issued as employee comp are
  meaningless — check this).

### ⑧ How this company dies (one line) → three risks

**Order matters.** The death scenario comes first, then the risks.

- **The death scenario, one line**: a *story*, not a risk list.
  > e.g. "The moment people stop feeling anything toward Disney's characters,
  > every one of its businesses collapses at once."
- **Three risks**: exactly three, each 1–2 lines. Don't copy the filing's Risk
  Factors verbatim — pick only the ones that **actually hit the money**.

### ⑨ Core financials (5-year · min 3-year) — table AND charts

Show four metrics as **both a table and charts**.

| Metric | Why you watch it (put this one-liner on the card) |
|---|---|
| Revenue | Is the business growing? |
| Operating Income | Do they make money from the core business? (+ operating margin trend) |
| Free Cash Flow (FCF) | **The cash that's actually left.** Earnings can be managed; cash is harder. |
| Total Debt | Can they pay it back? (+ net debt / EBITDA) |

Rules:
- Include **year-over-year change (YoY %)** in the table.
- Draw the charts inside the artifact (four small charts, or a 2×2 grid).
- **Cite a source on every metric** (page number in Mode B).
- If Operating Income and FCF diverge sharply → **always add a one-line
  warning** ("Operating income rose but FCF fell — worth understanding why.").

### ⑨½ Oxinion signal & factor read (Top 500 only)

When the Oxinion MCP returned data, add a compact block so the reader sees the
market's current read next to the fundamentals:

- **Signal** (daily): price, the BUY/HOLD/SELL call, trend, and RSI with its zone
  (>70 overbought, <30 oversold, 30–70 neutral). One line on what it implies.
- **QVMG factor scorecard** (weekly): Quality (ROIC, ROE, gross profitability),
  Value (P/E, P/B, FCF yield), Momentum (12-1m), Growth (revenue CAGR, EPS
  growth) — decimals shown as percentages, multiples as `×`, each with a
  one-word read (Strong / Fair / Weak).

Stamp both timestamps (daily signal vs weekly factors) and label the source
`Oxinion Finance`. If the name is outside the Top 500, omit this block — don't
apologize for it in the card, just leave it out.

### ⑩ What I still don't know (closing)

Two or three questions this card couldn't answer. This tells the user what to dig
into next.
> e.g. "When Disney+ turns profitable isn't answerable from this material — you'd
> need the latest earnings call."

---

## Workflow

### Step 1 — confirm the input

- Identify the ticker/company.
- Is a document uploaded? → decide Mode A vs B.
- Is the name in the Oxinion Top 500? → plan to call the MCP layer.
- If Mode A: add the one-line "upload a 10-K for page-cited precision" note.

### Step 2 — gather the data

**Mode A (web search)** — run searches like:
```
"{company} business model revenue segments"
"{TICKER} revenue by segment {latest year}"
"{TICKER} 10-K annual report"
"{TICKER} free cash flow total debt 5 year"
```
- Prefer primary sources (SEC EDGAR, IR pages). Don't use blogs or forums.
- If sources disagree, follow the primary source; if the gap is large, note it on
  the card.

**Mode B (uploaded document)**
- Read the uploaded document end to end first.
- Revenue breakdown → segment notes.
- Financials → the statements plus the cash flow statement.
- FCF = operating cash flow − capital expenditure (CapEx). **Show this
  calculation in a card footnote.**
- Record page numbers precisely.

**Oxinion MCP (Top 500)** — call `stock-analysis` and `stock-fundamentals` in
parallel with the above; convert the QVMG decimals to percentages.

### Step 3 — calculate

Do every calculation (YoY, shares, FCF, multiples) **by running code**, never in
your head. A silent arithmetic slip in the financials undermines the whole card.

### Step 4 — build the card

- Follow order ①–⑩ exactly, as **one HTML artifact**.
- Format: HTML with the diagram and charts inline, everything self-contained
  (inline CSS/JS; no external assets, no browser storage).
- Style: white background, black text, blue accent. Minimal. Readable even
  printed.
- **Skimmable in one scroll.** If it's too long, compress.

### Step 5 — one-line handoff

Below the card, a short chat note:
- If Mode A → the upload suggestion.
- One notable finding, if any.
- Keep it short. The card is the body.

---

## Common mistakes (don't)

| Mistake | Why it fails |
|---|---|
| Industry jargon in the 2-minute drill | If a 12-year-old can't follow it, it failed |
| Listing the money-making structure as text | It must be a picture — that's the differentiator |
| Skipping geographic revenue | FX and geopolitical risk go invisible |
| Listing five or more risks | Three. Nobody reads all five |
| Page numbers on a web source | **The most dangerous mistake** — it breaks trust |
| Estimating a number | If you don't know it, write `needs verification` |
| Reporting QVMG as raw decimals | 0.42 ROE is 42%, not 0.42% — convert first |
| Backfilling Oxinion data for a non-Top-500 name | Mixed provenance defeats the whole card |

## Example output

`examples/RY-company-overview.html` is a finished card (Royal Bank of Canada,
Mode A / web-sourced). Use it as the visual and structural target: the blue-accent
minimal style, the SVG money-flow diagram, the source tag on every table, the
Oxinion signal & factor block, and — because RBC is a bank — the ⑨ financials
recast to a bank frame (revenue, net income, ROE, CET1) instead of FCF/net-debt.
It also shows the honest habits in action: `needs verification` where a figure
couldn't be sourced, and a closing "what I still don't know".

## References

- US filings: https://www.sec.gov/edgar/searchedgar/companysearch
- Oxinion Finance: https://finance.oxinion.com
