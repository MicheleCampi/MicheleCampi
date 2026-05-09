# Hi, I'm Michele 👋

**Backend Engineer · 9 yrs operations domain · Controller turned builder.**

I build and operate [OptimEngine](https://github.com/MicheleCampi/optim-engine) — a production OR-Tools optimization service running on Python, FastAPI, and dual-stack MCP. Live, observable, tested, and verifiable.

🌐 **Live API** · [optim-engine-production.up.railway.app](https://optim-engine-production.up.railway.app)
📊 **Public Grafana dashboard** · [solver invocations · status mix · p95 latency](https://optimengine.grafana.net/public-dashboards/21137ba340fc4b6e917a4b108db3e109) (no login)
✍️ **Technical writing** · [michelecampi.github.io](https://michelecampi.github.io)

---

## Featured work

### OptimEngine — production-grade mathematical optimization service

A 4-layer system exposing **11 MCP tools** across 4 intelligence levels (9 optimization + 2 utility): flexible job-shop scheduling (FJSP), vehicle routing with time windows (CVRPTW), bin packing, schedule validation, sensitivity analysis, robust optimization, Monte Carlo with CVaR risk metrics, Pareto multi-objective frontier, and prescriptive intelligence. Two interfaces: standard REST API + dual-stack MCP (open SSE at `/mcp`, OAuth 2.1-gated Streamable HTTP at `/mcp/v2`).

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT 9.15 · FastMCP · ScaleKit OAuth 2.1 · Pydantic v2 · OpenTelemetry · Prometheus + Grafana Cloud · Railway · Vercel (edge proxy)

**Performance**
- Single-solve: provably optimal schedules in 10–40 ms · stochastic CVaR (100 Monte Carlo scenarios) ~2 s · sensitivity analysis (12 params × 5 perturbations) <500 ms
- Under sustained load: **757 requests, 0 failures across 4 Locust runs**, ~200 ms infrastructure floor, full bottleneck analysis in [BENCHMARKS.md](https://github.com/MicheleCampi/optim-engine/blob/main/BENCHMARKS.md)

**Engineering hygiene**
- **121 tests, 77 % overall coverage (88 % on business-logic engines)** — every push gated by [GitHub Actions CI](https://github.com/MicheleCampi/optim-engine/actions/workflows/tests.yml)
- Threat model and security policy: [SECURITY.md](https://github.com/MicheleCampi/optim-engine/blob/main/SECURITY.md)
- Operational runbook covering 5 production incident classes: [RUNBOOK.md](https://github.com/MicheleCampi/optim-engine/blob/main/RUNBOOK.md)
- Dependabot weekly · OpenTelemetry tracing · Telegram alerting on production events

**Live, public, verifiable** — the dashboard, the benchmarks, the test suite, and the runbook are all in the open. No "trust me" claims.

---

## Currently working on

- **OptimEngine v9.x extensions** · sequence-dependent setup times, per-machine duration scaling, availability windows, quality gates, all backward-compatible with v8 requests.
- **Distributed tracing deep-dive** · OpenTelemetry traces for CP-SAT solves end-to-end, Grafana visualizations of per-stage latency, technical write-up in progress.
- **Public technical articles** · scheduled cadence through May 2026 covering profiling decisions, CORS-to-edge-proxy migration, observability for OR-Tools services, and a multi-tenant solver retrospective.

---

## Recent technical writing

- **[How fragile is your weekly plan? A risk-premium framework](https://michelecampi.github.io/2026/05/04/risk-premium-mid-market-manufacturing.html)** (May 2026) · Monte Carlo + CVaR applied to a real OR-Tools schedule. Doubling input volatility raises the risk premium from 4.2 % to 7.2 % — the plan is structurally robust, and the framework is reproducible via a single API call.
- **[What an OR-Tools solver finds in a week of contract packaging](https://michelecampi.github.io/2026/04/29/or-tools-week-contract-packaging.html)** (April 2026) · Synthetic but realistic mid-market case: 8 customer orders, 6 production lines, sequence-dependent setup times. Manual schedule lands at 190 quarter-hours. OptimEngine returns the proven optimum in 10 ms: 161 quarters.
- **[Why your AI assistant can't actually plan your factory](https://michelecampi.github.io/2026/04/25/why-ai-assistants-cant-plan-your-factory.html)** (April 2026) · Direct test comparing a frontier AI assistant running CP-SAT in its sandbox against a production constraint solver. Same problem, sharply different outcomes.
- **[Three production scheduling failures, and the math that would have caught them](https://michelecampi.github.io/2026/04/26/three-scheduling-failures-and-the-math-that-would-have-caught-them.html)** (April 2026) · Three operational patterns recurring across mid-market plants, each with hidden cause and quantitative method.
- **[How I exposed OR-Tools as a production MCP server](https://michelecampi.github.io/2026/04/15/how-i-exposed-or-tools-as-mcp.html)** (April 2026) · Building a Model Context Protocol server that wraps Google's constraint solver. What changes when AI agents can call your solver in natural language.

Full archive on [the blog](https://michelecampi.github.io/).

---

## Background

9+ years building quantitative systems for industrial operations — cost-by-workcenter modeling, margin frameworks, capacity analysis, and forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last 2 years I extended that practice into computational infrastructure: production-grade constraint solvers, observability stacks, MCP server architecture, OAuth-protected APIs. The path is uncommon — domain depth from 9 years inside operations is what makes the optimization work credible, and the technical execution is what makes it useful.

---

## Reach out

📧 **michele.campi [at] outlook.com** *(replace [at] with @)*

Also on [X](https://x.com/MicheleC54474) and [Wellfound](https://wellfound.com/u/michele-campi) for engineering opportunities.
