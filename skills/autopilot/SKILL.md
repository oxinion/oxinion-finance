---
name: autopilot
description: >
  This skill should be used when the user wants to preview an Oxinion Finance
  autopilot rebalance — "run autopilot", "what would autopilot do", "preview my
  rebalance", "how would Oxinion rebalance my portfolio", "show me the autopilot
  suggestion", or "what trades would rebalancing make?" Returns a preview of the
  target the autopilot workflow would rebalance the user's portfolio toward,
  based on their saved preferences. This is preview-only — it never places live
  trades. To review the current portfolio without rebalancing use
  portfolio-analysis.
metadata:
  version: "0.1.0"
---

# Oxinion Autopilot (Preview)

Preview the rebalance that Oxinion's autopilot workflow would target.

## When this runs

The user wants to see what autopilot *would do* — the target allocation and the
implied moves — before acting. This is always a dry run.

## Critical framing

**This is preview-only. Real order execution is not wired up, so this never
places live trades.** Say this clearly whenever presenting results, so the user
is never left thinking trades were executed.

## Steps

1. **Call `run-autopilot`** (no arguments). It returns the target the autopilot
   workflow would rebalance toward, based on the user's saved preferences.
2. **If it returns an error or empty result**, the user probably hasn't set up
   their autopilot preferences / target portfolio in Oxinion yet. Tell them so
   and point them to configure it in Oxinion Finance.

## Presenting the result

- **Target allocation** — the holdings and weights autopilot would move toward.
- **Implied changes** — where the tool surfaces current-vs-target, summarize the
  direction of each move (trim / add / new / exit). If only the target is
  returned, present the target and note that current positions come from
  `portfolio-analysis`.
- **Preview banner** — restate up front and again at the end that no trades were
  placed; this is a simulation of what autopilot would target.

Label the data `Source: Oxinion Finance`. Close with a one-line reminder that
this is an automated preview, not personalized investment advice or a
transaction.

## Handoffs

- Want to review the current book first? → `portfolio-analysis`.
- Want to sanity-check a single name in the target? → `stock-signal` or
  `factor-scorecard`.
