# Hi, I'm Michele 👋

Rust + AI infrastructure. I build observability for LLM systems and production-grade optimization services. Nine years inside industrial operations before turning that domain depth into code.

🌐 [inferscope on GitHub](https://github.com/MicheleCampi/inferscope) · 📊 [OptimEngine live dashboard](https://...) · ✍️ [Technical writing](https://michelecampi.github.io)

---

## Featured work

### inferscope — profiler and observability for LLM inference engines

A Rust profiler that drives an OpenAI-compatible inference engine through its HTTP API, captures per-token timing end-to-end, and correlates that timing with the engine process's CPU and GPU resource usage on a single shared wall clock. Outputs a plain-text report for terminal reading and a JSON document carrying both raw signals and derived metrics (TTFT, tokens-per-second excluding TTFT, inter-token latency percentiles, RSS aggregations, optional VRAM and SM utilization).

**Stack** · Rust 1.83 · tokio multi-thread runtime · reqwest + SSE streaming · async /proc + NVML sampler · five-crate Cargo workspace with strict separation of concerns (`is-core` pure types, `is-probe` network I/O, `is-sysmon` filesystem + GPU I/O, `is-report` presentation, `inferscope` CLI orchestrator)

**Validation** · 109 tests · CI gated on `-D warnings` (no unused imports, no clippy lints) · v0.1.0 (16 May) tested end-to-end against llama.cpp b9165 with Qwen 2.5 0.5B Q4 on Ubuntu 24.04 / x86_64: TTFT 25 ms, 82.7 tokens/s, 588 MiB RSS · v0.2.0 (20 May) validated on NVIDIA RTX L4 via RunPod, same model: 375.7 tokens/s and 15 ms TTFT (warm), SM utilization peak 83% / mean 41%, VRAM 1.25 GiB stable, power draw 35–39 W — ~4.5× throughput vs CPU baseline, with the cold-vs-warm TTFT delta (330 ms → 15 ms) captured by the sampler.

**Hygiene** · MSRV pinned to Rust 1.83 via `rust-toolchain.toml` · five Architecture Decision Records covering profiling scope, token timing representation, sysmon correlation, report format, and GPU sampling design · Apache-2.0

### OptimEngine — production OR-Tools optimization service

A 4-layer system exposing 11 MCP tools across 4 intelligence levels (9 optimization + 2 utility): flexible job-shop scheduling (FJSP), vehicle routing with time windows (CVRPTW), bin packing, sensitivity analysis, robust optimization, Monte Carlo with CVaR risk metrics, Pareto multi-objective frontier, prescriptive intelligence. Two interfaces: standard REST API and dual-stack MCP (open SSE at `/mcp`, OAuth 2.1-gated Streamable HTTP at `/mcp/v2`).

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT 9.15 · FastMCP · ScaleKit OAuth 2.1 · OpenTelemetry · Prometheus + Grafana Cloud + Grafana Alloy · Railway · Vercel Edge

**Performance** · Single-solve: provably optimal schedules in 10–40 ms · stochastic CVaR (100 Monte Carlo scenarios) ~2 s · sensitivity analysis (12 params × 5 perturbations) <500 ms · 757 requests / 0 failures across 4 Locust runs, full bottleneck analysis in `BENCHMARKS.md`

**Distribution surface** · Smithery.ai MCP registry (9 tools registered) · edge proxy on Vercel for browser MCP access · 36 x402 monetization endpoints live on Base Mainnet and Solana Mainnet (payment-gated solver access for autonomous agents)

**Hygiene** · 121 tests, 77% overall coverage (88% on business-logic engines) · CI on every push · threat model in `SECURITY.md` · operational runbook for 5 production incident classes in `RUNBOOK.md` · OpenTelemetry distributed tracing live on Grafana Cloud Tempo (manual sub-spans inside the CP-SAT solver entry points) · Grafana Alloy as scrape collector with remote_write to Grafana Cloud Mimir · Telegram alerting on production events · Dependabot weekly

*Live, public, verifiable — the dashboard, the benchmarks, the test suite, and the runbook are all in the open. No "trust me" claims.*

---

## Currently working on

**inferscope H100 case study** · Next validation run on H100 with a larger model (Qwen 7B or Llama-class) to exercise the NVML sampler under real VRAM-bound workloads. v0.3 roadmap adds AMD GPU support via amd-smi.

**OptimEngine on-chain distribution** · Continued buildout of the x402 payment surface across Base and Solana, plus the402.ai service catalog. Real autonomous-agent buyers confirmed on stochastic optimization endpoints.

**Public technical articles** · Cadence ~1 article per month from June 2026 onward (post-consolidation), focused on observability for compute-bound services and Rust profiling of LLM inference.

---

## Recent technical writing

**[Profiling LLM inference: what your /proc sampler isn't telling you](https://michelecampi.github.io/...)** (May 2026) · Bug-discovery narrative behind inferscope v0.2: why a /proc-only view of an inference engine misses the resource that matters most, and how NVML sampling fills the gap.

**[Why your OpenTelemetry trace shows nothing useful when the CPU is doing all the work — a CP-SAT case study](https://michelecampi.github.io/...)** (May 2026) · Why default OpenTelemetry auto-instrumentation fails for compute-bound services (solvers, ML inference, simulation engines). Before/after traces on a real CP-SAT workload showing how manual span instrumentation surfaces what auto-instrumentation hides.

**[How fragile is your weekly plan? A risk-premium framework](https://michelecampi.github.io/...)** (May 2026) · Monte Carlo + CVaR applied to a real OR-Tools schedule. Doubling input volatility raises the risk premium from 4.2% to 7.2% — the plan is structurally robust, and the framework is reproducible via a single API call.

**[How I exposed OR-Tools as a production MCP server](https://michelecampi.github.io/...)** (April 2026) · Building a Model Context Protocol server that wraps Google's constraint solver. What changes when AI agents can call your solver in natural language — and what stays the same about solver-side rigour.

*Eight articles in total. Full archive on [the blog](https://michelecampi.github.io).*

---

## Background

Nine years building quantitative systems for industrial operations — cost-by-workcenter modeling, margin frameworks, capacity analysis, forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last two years I extended that practice into computational infrastructure: production-grade constraint solvers, observability stacks, MCP server architecture, OAuth-protected APIs, and now a Rust profiler for LLM inference engines with NVIDIA GPU sampling. The path is uncommon — domain depth from nine years inside operations is what makes the optimization work credible, and the technical execution is what makes it useful in production.

---

## Reach out

📧 michele.campi [at] outlook.com (replace [at] with @)

Also on Wellfound and X for engineering opportunities.
