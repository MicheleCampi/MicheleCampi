## Hi, I'm Michele 👋

**Rust + AI infrastructure.** I build observability for LLM systems and production-grade optimization services. Nine years inside industrial operations before turning that domain depth into code.

🌐 [inferscope on GitHub](https://github.com/MicheleCampi/inferscope) · 📊 [OptimEngine live dashboard](https://optimengine.grafana.net/public-dashboards/21137ba340fc4b6e917a4b108db3e109) · ✍️ [Technical writing](https://michelecampi.github.io)

---

## Featured work

### [inferscope](https://github.com/MicheleCampi/inferscope) — profiler and observability for LLM inference engines

A Rust profiler that drives an OpenAI-compatible inference engine through its HTTP API, captures per-token timing end-to-end, and correlates that timing with the engine process's `/proc`-sampled RSS, CPU, and thread count over a single shared wall clock. Outputs a plain-text report for terminal reading and a JSON document carrying both raw signals and derived metrics (TTFT, tokens-per-second excluding TTFT, inter-token latency percentiles, RSS aggregations).

**Stack** · Rust 1.83 · tokio multi-thread runtime · reqwest + SSE streaming · async `/proc` sampler · five-crate Cargo workspace with strict separation of concerns (`is-core` pure types, `is-probe` network I/O, `is-sysmon` filesystem I/O, `is-report` presentation, `inferscope` CLI orchestrator)

**Validation** · 96 tests · CI gated on `-D warnings` (no unused imports, no clippy lints) · v0.1.0 tested end-to-end against llama.cpp b9165 with Qwen 2.5 0.5B Q4 · TTFT 25 ms, 82.7 tokens/s, 588 MiB RSS on a single Ubuntu 24.04 / x86_64 host

**Hygiene** · MSRV pinned to Rust 1.83 via `rust-toolchain.toml` · four Architecture Decision Records covering profiling scope, token timing representation, sysmon correlation, and report metrics · Apache-2.0

---

### [OptimEngine](https://github.com/MicheleCampi/optim-engine) — production OR-Tools optimization service

A 4-layer system exposing 11 MCP tools across 4 intelligence levels (9 optimization + 2 utility): flexible job-shop scheduling (FJSP), vehicle routing with time windows (CVRPTW), bin packing, sensitivity analysis, robust optimization, Monte Carlo with CVaR risk metrics, Pareto multi-objective frontier, prescriptive intelligence. Two interfaces: standard REST API and dual-stack MCP (open SSE at `/mcp`, OAuth 2.1-gated Streamable HTTP at `/mcp/v2`).

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT 9.15 · FastMCP · ScaleKit OAuth 2.1 · OpenTelemetry · Prometheus + Grafana Cloud · Railway

**Performance** · Single-solve: provably optimal schedules in 10–40 ms · stochastic CVaR (100 Monte Carlo scenarios) ~2 s · sensitivity analysis (12 params × 5 perturbations) <500 ms · 757 requests / 0 failures across 4 Locust runs, full bottleneck analysis in `BENCHMARKS.md`

**Hygiene** · 121 tests, 77 % overall coverage (88 % on business-logic engines) · CI on every push · threat model in `SECURITY.md` · operational runbook for 5 production incident classes in `RUNBOOK.md` · OpenTelemetry tracing · Telegram alerting on production events · Dependabot weekly

Live, public, verifiable — the dashboard, the benchmarks, the test suite, and the runbook are all in the open. No "trust me" claims.

---

## Currently working on

**inferscope GPU validation** · Running v0.1.0 against GPU-hosted engines on Lambda Cloud (Qwen-larger and Llama-class models on A100 / H100) to characterise the resource correlation under VRAM-bound workloads, ahead of v0.2 which adds NVML / ROCm SMI sampling to the existing `/proc` sampler.

**OptimEngine v9.x extensions** · sequence-dependent setup times, per-machine duration scaling, availability windows, quality gates — all backward-compatible with v8 requests.

**Public technical articles** · cadenced through summer 2026, focused on observability for systems software (distributed tracing for CP-SAT solves, profiling decisions, OTel + Tempo integration).

---

## Recent technical writing

__[Why your OpenTelemetry trace shows nothing useful when the CPU is doing all the work — a CP-SAT case study](https://michelecampi.github.io/observability/systems-engineering/2026/05/17/otel-tracing-compute-bound-services.html)__ (May 2026) · Why default OpenTelemetry auto-instrumentation fails for compute-bound services (solvers, ML inference, simulation engines). Before/after traces on a real CP-SAT workload showing how manual span instrumentation surfaces what auto-instrumentation hides.

[**Why your AI assistant can't actually plan your factory**](https://michelecampi.github.io/) (April 2026) · A direct comparison between a frontier AI assistant running CP-SAT in its sandbox and a production constraint solver. Same problem, sharply different outcomes. Where general-purpose models meet the wall of dedicated solver engineering.

[**How I exposed OR-Tools as a production MCP server**](https://michelecampi.github.io/) (April 2026) · Building a Model Context Protocol server that wraps Google's constraint solver. What changes when AI agents can call your solver in natural language — and what stays the same about solver-side rigour.

[**How fragile is your weekly plan? A risk-premium framework**](https://michelecampi.github.io/) (May 2026) · Monte Carlo + CVaR applied to a real OR-Tools schedule. Doubling input volatility raises the risk premium from 4.2 % to 7.2 % — the plan is structurally robust, and the framework is reproducible via a single API call.

Full archive on [the blog](https://michelecampi.github.io).

---

## Background

Nine years building quantitative systems for industrial operations — cost-by-workcenter modeling, margin frameworks, capacity analysis, forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last two years I extended that practice into computational infrastructure: production-grade constraint solvers, observability stacks, MCP server architecture, OAuth-protected APIs, and now a Rust profiler for LLM inference engines. The path is uncommon — domain depth from nine years inside operations is what makes the optimization work credible, and the technical execution is what makes it useful in production.

---

## Reach out

📧 michele.campi [at] outlook.com (replace [at] with @)

Also on [Wellfound](https://wellfound.com/u/michele-campi) and [X](https://x.com/MicheleCampi_) for engineering opportunities.
