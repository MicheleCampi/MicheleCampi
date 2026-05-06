# Hi, I'm Michele 👋

**Backend Engineer + 9 yrs operations domain.** Building [OptimEngine](https://optim-engine-production.up.railway.app) — production OR-Tools optimization service running on Python, FastAPI, and dual-stack MCP. Live, observable, and verifiable.

🌐 **Live API** · [optim-engine-production.up.railway.app](https://optim-engine-production.up.railway.app)
📊 **Public dashboard** · [optimengine.grafana.net](https://optimengine.grafana.net/public-dashboards/21137ba340fc4b6e917a4b108db3e109) — solver invocations, status mix, p95 latency (no login required)
✍️ **Technical writing** · [michelecampi.github.io](https://michelecampi.github.io)

---

## Featured work

### OptimEngine — production-grade mathematical optimization service

A 9-solver service exposing flexible job-shop scheduling (FJSP), vehicle routing with time windows (CVRPTW), bin packing, sensitivity analysis, robust optimization, Monte Carlo with CVaR risk metrics, Pareto multi-objective frontier, and prescriptive intelligence. Two interfaces: standard REST API + dual-stack MCP (open SSE at `/mcp`, OAuth 2.1-gated Streamable HTTP at `/mcp/v2`).

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT v9.0.0 · FastMCP · ScaleKit OAuth · Pydantic · Prometheus + Grafana · Railway

**Performance on realistic mid-market scenarios** · provably optimal schedules in 10–40 ms · stochastic CVaR (100 Monte Carlo scenarios) in ~2 s · sensitivity analysis (12 params × 5 perturbations) in <500 ms.

**Documentation repo** · [optim-engine-showcase](https://github.com/MicheleCampi/optim-engine-showcase) (architecture, 11-tool reference, design decisions)

### optim-arc-v3 — x402 payment gateway for autonomous AI agents

Next.js gateway implementing Circle Nanopayments / x402 v2 protocol, exposing 10 paid OptimEngine endpoints. Forked from `circlefin/arc-nanopayments`, deployed on Vercel, returns HTTP 402 with valid `GatewayWalletBatched` payment requirements.

[optim-arc-v3.vercel.app](https://optim-arc-v3.vercel.app)

---

## Currently building

- **Edge proxy for browser-based MCP access** · [optim-engine-proxy](https://github.com/MicheleCampi/optim-engine-proxy) — thin Vercel Edge proxy (TypeScript, ~200 LOC) sitting in front of OptimEngine's MCP server. Decouples browser-side concerns (CORS, demo embeds, sandbox iframes) from the production solver, which stays server-to-server only. Architectural decision documented in DESIGN.md *before* implementation; verified end-to-end with curl + integration tests. Live: [optim-engine-proxy.vercel.app](https://optim-engine-proxy.vercel.app)
- **Observability stack hardening** · Prometheus metrics, Grafana dashboard, alert rules on Railway production. Public dashboard now live (linked above). Currently working on automated load test runs to keep dashboard always populated.
- **OptimEngine v9.x extensions** · sequence-dependent setup times, per-machine duration scaling, availability windows, quality gates.
- **Public technical articles** · 5 published, 2 in draft for May 2026 (risk-premium framework, observability for OR-Tools services).

---

## Recent technical writing

- **[How fragile is your weekly plan? A risk-premium framework](https://michelecampi.github.io/2026/05/04/risk-premium-mid-market-manufacturing.html)** (May 2026) · Monte Carlo + CVaR applied to a real OR-Tools schedule. Doubling input volatility raises the risk premium from 4.2% to 7.2% — the plan is structurally robust, and the framework is reproducible via a single API call.
- **[What an OR-Tools solver finds in a week of contract packaging](https://michelecampi.github.io/2026/04/29/or-tools-week-contract-packaging.html)** (April 2026) · Synthetic but realistic mid-market case: 8 customer orders, 6 production lines, sequence-dependent setup times. Manual schedule lands at 190 quarter-hours. OptimEngine returns the proven optimum in 10 ms: 161 quarters.
- **[Why your AI assistant can't actually plan your factory](https://michelecampi.github.io/2026/04/25/why-ai-assistants-cant-plan-your-factory.html)** (April 2026) · Direct test comparing a frontier AI assistant running CP-SAT in its sandbox against a production constraint solver. Same problem, sharply different outcomes.
- **[Three production scheduling failures, and the math that would have caught them](https://michelecampi.github.io/2026/04/26/three-scheduling-failures-and-the-math-that-would-have-caught-them.html)** (April 2026) · Three operational patterns recurring across mid-market plants, each with hidden cause and quantitative method.
- **[How I exposed OR-Tools as a production MCP server](https://michelecampi.github.io/2026/04/15/how-i-exposed-or-tools-as-mcp.html)** (April 2026) · Building a Model Context Protocol server that wraps Google's constraint solver. What changes when AI agents can call your solver in natural language.

Full archive on [the blog](https://michelecampi.github.io/).

---

## Background

9+ years building quantitative systems for industrial operations — cost-by-workcenter modeling, margin frameworks, capacity analysis, and forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last 2 years extended that practice into computational infrastructure: production-grade constraint solvers, observability stacks, MCP server architecture, x402 payment APIs. The path is uncommon — domain depth from 9 years inside operations is what makes the optimization work credible, and the technical execution is what makes it useful.

---

## Reach out

📧 **michele.campi [at] outlook.com** *(replace [at] with @)*

Also on [X](https://x.com/MicheleC54474) and [Wellfound](https://wellfound.com/u/michele-campi) for engineering opportunities.
