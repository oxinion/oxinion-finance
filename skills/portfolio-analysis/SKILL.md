---
name: portfolio-analysis
description: >
  This skill should be used when the user wants Oxinion Finance to review their
  own portfolio — "analyze my portfolio", "how does my portfolio look", "review
  my holdings", "what's my target allocation doing", "any red flags in my
  positions", "how are my holdings trending", or "give me a health check on my
  portfolio." Pulls the authenticated user's saved target allocation from
  Oxinion and enriches each holding with live price, trend, and signal. For a
  single ticker use stock-signal or factor-scorecard; to preview a rebalance use
  autopilot.
metadata:
  version: "0.1.0"
---

# Oxinion Portfolio Analysis

Review the authenticated user's target portfolio, enriched with live signals.

## When this runs

The user wants a look at *their own* portfolio or target allocation — not a
single stock. This reads the allocation saved in their Oxinion autopilot
preferences.

## Steps

1. **Call `portfolio-analysis`** (no arguments). It returns the user's target
   allocation with live price / trend / signal for each holding.
2. **If it returns an error or an empty allocation**, the user likely hasn't set
   up their target portfolio / autopilot preferences in Oxinion yet. Tell them
   that plainly and point them to configure it in Oxinion Finance, rather than
   guessing at holdings.

## Presenting the result

Give the user a clear, scannable review:

- **Holdings table** — each position with its target weight, latest price,
  trend, and BUY/HOLD/SELL signal.
- **Signal summary** — count how many holdings are BUY vs HOLD vs SELL, and call
  out any SELL-signalled positions by name so they stand out.
- **Trend read** — note which holdings are trending down versus up.
- **One-paragraph takeaway** — the overall shape: is the book mostly
  constructive, mixed, or leaning defensive?

Label the data `Source: Oxinion Finance`. Keep interpretation grounded in what
the tool returned — do not add holdings or prices from elsewhere.

Close with a one-line reminder that this is an automated review of a saved
allocation, not personalized investment advice.

## Handoffs

- Want to preview what a rebalance would look like? → `autopilot`.
- Want to drill into one holding? → `stock-signal` or `factor-scorecard`.
