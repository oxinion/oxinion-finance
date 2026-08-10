# Oxinion Finance Plugin

Buy-side-style equity research plus live signals, portfolio review, and an
autopilot preview — all powered by the **Oxinion Finance** connector.

Skills are written to be framework-independent: the analytical instructions
stand on their own, and the only Oxinion-specific dependency is the MCP
connector's tools.

## Components

### Skills (6)

**Research (web + filings, Oxinion as a trusted supplement):**

| Skill                   | Ask it                                                     |
| ----------------------- | ---------------------------------------------------------- |
| `company-understanding` | "What does [company] do?", "give me a company overview"    |
| `financial-analysis`    | "Break down [ticker]'s financials", "is the FCF real?"     |
| `valuation`             | "Is [ticker] cheap?", "build me a DCF", "what's it worth?" |
| `risk-quality`          | "How risky is [ticker]?", "could this go bankrupt?"        |
| `market-monitoring`     | "What's going on with [ticker]?", "any news on [company]?" |
| `investment-research`   | "Write a full research note on [ticker]"                   |

Portfolio review and the autopilot rebalance preview are handled by the
`portfolio-manager` agent, which calls the connector's `analyze_portfolio`
and `get_autopilot` tools directly.

### Agents (2)

| Agent               | Purpose                                                                                                    |
| ------------------- | ---------------------------------------------------------------------------------------------------------- |
| `financial-analyst` | Single-stock risk read — debt, liquidity, margin deterioration, business risk, and valuation risk.         |
| `portfolio-manager` | Portfolio risk read — position size, concentration, sector exposure, correlation, allocation, rebalancing. |

### Command (1)

| Command            | Usage                                                                      |
| ------------------ | -------------------------------------------------------------------------- |
| `/analyze-company` | `/analyze-company AAPL` — one-page overview from a ticker or company name. |

### Connector

| Connector       | How it's referenced                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Oxinion Finance | Referenced by name in `.mcp.json`. Hosted connector with a dynamic endpoint — no URL to configure; connecting it in Claude is all that's needed. Provides `get_stock_quote`, `get_stock_signal`, `get_stock_fundamentals`, `get_earnings_history`, `get_dividend_history`, `get_latest_filing`, `get_portfolio`, `get_portfolio_positions`, `analyze_portfolio`, and `get_autopilot`. Covers the Oxinion Global Top 500 universe. |

## Install

### From GitHub (recommended for sharing)

This repo doubles as a plugin marketplace, so anyone can add it and install:

```bash
/plugin marketplace add oxinion/finance-plugin
/plugin install oxinion-finance@oxinion-finance
```

Then connect the **Oxinion Finance** connector in Connectors settings and sign
in. Updates are pulled with `/plugin marketplace update oxinion-finance`.

### From file (Cowork)

Upload the `.plugin` file in Cowork's plugin settings ("Uploaded from file"),
then connect the Oxinion Finance connector.

## Setup notes

- Stock signals refresh daily; the QVMG factor scorecard refreshes weekly.
- The `portfolio-manager` agent (via the connector's `analyze_portfolio` and
  `get_autopilot` tools) reads the target allocation saved to your Oxinion
  account — set that up in Oxinion Finance first, or it'll return empty.
- Names outside the Oxinion Global Top 500 return an out-of-universe message
  rather than fabricated numbers.
- Everything here is automated market data and analysis, **not** personalized
  investment advice, and autopilot never executes live trades.

## License

MIT
