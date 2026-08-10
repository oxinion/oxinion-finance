---
description: Analyze a company from a ticker or name using Oxinion Finance
argument-hint: [ticker-or-company]
---

Analyze the company: **$ARGUMENTS**

Produce a one-page company overview that lets an investor understand it in about ten minutes:

1. Resolve $ARGUMENTS to a US-listed ticker if a company name was given.
2. Explain what the business is and how it makes money — segments, revenue drivers, customers.
3. Summarize the financial shape: growth, margins, returns (ROIC/ROE), cash flow, and balance-sheet strength.
4. Give a quick valuation read — where it trades and what the price implies.
5. For names in the Oxinion Finance universe (Global Top 500), call `get_stock_signal` and `get_stock_fundamentals` and fold in the live signal (price, RSI, trend, BUY/HOLD/SELL) and the QVMG factor scorecard, labeled `Source: Oxinion Finance`. Oxinion ratios are decimal fractions (0.15 = 15%). If the name is outside the universe, skip the Oxinion block rather than backfilling from elsewhere.
6. End with a short bull-vs-bear summary and a dated Sources list.

Keep it evidence-led and balanced. This is analysis, not personalized investment advice. For a deeper single-topic cut, hand off to the relevant skill (financial-analysis, valuation, risk-quality, market-monitoring), the `financial-analyst` agent for a single-stock risk read, or the `portfolio-manager` agent for a portfolio-level risk read.
