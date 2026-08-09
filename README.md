# Oxinion Finance Plugin

Buy-side-style equity research plus live signals, portfolio review, and an
autopilot preview — all powered by the **Oxinion Finance** connector.

Skills are written to be framework-independent: the analytical instructions
stand on their own, and the only Oxinion-specific dependency is the MCP
connector's four tools.

## Components

### Skills (8)

**Research (web + filings, Oxinion as a trusted supplement):**

| Skill                   | Ask it                                                     |
| ----------------------- | ---------------------------------------------------------- |
| `company-understanding` | "What does [company] do?", "give me a company overview"    |
| `financial-analysis`    | "Break down [ticker]'s financials", "is the FCF real?"     |
| `valuation`             | "Is [ticker] cheap?", "build me a DCF", "what's it worth?" |
| `risk-quality`          | "How risky is [ticker]?", "could this go bankrupt?"        |
| `market-monitoring`     | "What's going on with [ticker]?", "any news on [company]?" |
| `investment-research`   | "Write a full research note on [ticker]"                   |

**Oxinion account tools (thin wrappers over the connector):**

| Skill                | Ask it                                        | Notes                                                           |
| -------------------- | --------------------------------------------- | --------------------------------------------------------------- |
| `portfolio-analysis` | "Analyze my portfolio", "review my holdings"  | Reads your saved target allocation, enriched with live signals. |
| `autopilot`          | "Run autopilot", "what would a rebalance do?" | **Preview only** — never places live trades.                    |

### Agents (2)

| Agent                   | Purpose                                                                                     |
| ----------------------- | ------------------------------------------------------------------------------------------- |
| `financial-analyst`     | Numbers-first read of one company's statements, margins, cash flow, and capital allocation. |
| `investment-researcher` | Full buy-side research note end to end, with a bull-vs-bear box and dated sources.          |

### Command (1)

| Command            | Usage                                                                      |
| ------------------ | -------------------------------------------------------------------------- |
| `/analyze-company` | `/analyze-company AAPL` — one-page overview from a ticker or company name. |

### Connector

| Connector       | How it's referenced                                                                                                                                                                                                                                                                      |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Oxinion Finance | Referenced by name in `.mcp.json`. Hosted connector with a dynamic endpoint — no URL to configure; connecting it in Claude is all that's needed. Provides `stock-analysis`, `stock-fundamentals`, `portfolio-analysis`, and `run-autopilot`. Covers the Oxinion Global Top 100 universe. |

## Install

### From GitHub (recommended for sharing)

This repo doubles as a plugin marketplace, so anyone can add it and install:

```bash
/plugin marketplace add oxinion/oxinion-finance-plugin
/plugin install oxinion-finance@oxinion-finance
```

Then connect the **Oxinion Finance** connector in Connectors settings and sign
in. Updates are pulled with `/plugin marketplace update oxinion-finance`.

### From file (Cowork)

Upload the `.plugin` file in Cowork's plugin settings ("Uploaded from file"),
then connect the Oxinion Finance connector.

## Setup notes

- Stock signals refresh daily; the QVMG factor scorecard refreshes weekly.
- `portfolio-analysis` and `autopilot` read the target allocation saved to your
  Oxinion account — set that up in Oxinion Finance first, or they'll return
  empty.
- Names outside the Oxinion Global Top 100 return an out-of-universe message
  rather than fabricated numbers.
- Everything here is automated market data and analysis, **not** personalized
  investment advice, and autopilot never executes live trades.

## License

MIT
