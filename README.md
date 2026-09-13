# Rule-Based Market Signal Automation

An event-driven automation that evaluates live market data through a chain of
sequential rule "gates" and logs a signal only when every condition passes.
Built to remove discretionary judgment from a trading strategy by encoding it
as deterministic, testable logic.

**Status:** Validation phase — runs on a schedule and logs signals only.
No live order execution.

---

## Architecture

The system is split into two parts that communicate over HTTP:

**1. Signal Engine (n8n + JavaScript)**
Six sequential rule-gates run in a self-hosted n8n workflow (Docker). Each gate
only executes if the previous one passed, so the first failed condition
short-circuits the whole chain — no trade.

**2. Data Bridge (Python + Flask)**
A lightweight Flask microservice wraps a broker SDK and exposes JSON HTTP
endpoints. It is schema-matched to the market-data REST API the workflow
originally used, so the data source can be swapped with zero changes to the
gate logic.


---

## The Gate Logic

| Gate | Rule |
|------|------|
| 1 — Premarket Break | Price breaks above the premarket high |
| 2 — Market Strength | Index (QQQ) is above its open **and** rising |
| 3 — Zone Detection  | Identify live imbalance zones above the break level |
| 4 — Return          | Price returns into one of those zones |
| 5 — Reaction        | Strong rejection candle + confirmation |
| 5b — Invalidation   | Cancel if an opposing imbalance formed on the pullback |
| 6 — Verdict         | Reaction valid **and** not invalidated → log signal |

---

## Tech Stack

- **Languages:** Python, JavaScript
- **Automation:** n8n (self-hosted, Docker)
- **Backend:** Flask (REST/JSON microservice)
- **Integrations:** Market-data REST API, broker SDK
- **Concepts:** API integration, event-driven automation, fail-safe design,
  scheduled execution, containerized services

---

## Engineering Notes

- **Fail-safe data handling:** missing or empty data returns a guard value that
  makes a false signal impossible, rather than crashing or misfiring.
- **Container networking:** the Dockerized workflow reaches the host bridge via
  `host.docker.internal`, with the Flask service bound to all interfaces.
- **Debugging:** production issues were traced through execution logs across
  Docker networking, cross-node references, and JSON serialization.

---

## Built With AI Assistance

This project was developed in collaboration with AI coding assistants (Claude),
used for architecture design, debugging production errors, translating logic
between JavaScript and Python, and learning Python from a beginner baseline.
AI accelerated the build while I owned every design decision and verified all
logic through manual testing with controlled inputs.

