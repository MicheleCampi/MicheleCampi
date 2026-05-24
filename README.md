# Hi, I'm Michele 👋

**Rust + AI infrastructure.** I build observability for LLM systems and production-grade optimization services. Nine years inside industrial operations before turning that domain depth into code.

🌐 [inferscope on GitHub](https://github.com/MicheleCampi/inferscope) · 📊 [OptimEngine live dashboard](https://...) · ✍️ [Technical writing](https://michelecampi.github.io)

---

## Featured work

### inferscope — profiler and observability for LLM inference engines

A Rust profiler that drives an OpenAI-compatible inference engine through its HTTP API, captures per-token timing end-to-end, and correlates that timing with the engine process's CPU and GPU resource usage on a single shared wall clock. Outputs a plain-text report for terminal reading and a JSON document carrying both raw signals and derived metrics (TTFT, tokens-per-second excluding TTFT, inter-token latency percentiles, RSS aggregations, optional VRAM and SM utilization).

**Stack** · Rust 1.83 · tokio multi-thread runtime · reqwest + SSE streaming · async /proc + NVML sampler with process-tree aggregation · five-crate Cargo workspace with strict separation of concerns (`is-core` pure types, `is-probe` network I/O, `is-sysmon` filesystem + GPU I/O, `is-report` presentation, `inferscope` CLI orchestrator)

**Validation** · 120 tests · CI gated on `-D warnings` (no unused imports, no clippy lints) · integration test on a synthetic bash + sleep parent-child pair exercises the v0.2.1 aggregation path end-to-end · v0.1.0 (16 May) tested end-to-end against llama.cpp b9165 with Qwen 2.5 0.5B Q4 on Ubuntu 24.04 / x86_64: TTFT 25 ms, 82.7 tokens/s, 588 MiB RSS · **v0.2.0 (20 May)** validated on NVIDIA RTX L4 via RunPod, same model: 381 tokens/s and 13 ms TTFT (warm), SM utilization peak 91% / mean 58%, VRAM 1.34 GB stable, power 37–39 W — ~4.6× throughput vs CPU baseline · **v0.2.1 (22 May)** validated on NVIDIA H100 SXM via RunPod, two-model contrast: Qwen 2.5 7B Q4 → 230 tokens/s, SM mean 48%, VRAM 5.6 GB / 80 GB, power mean 170 W (chip loafing); Qwen 2.5 32B Q4 → 69 tokens/s, SM mean 88%, VRAM 21.4 GB / 80 GB, power mean 439 W (chip earning its cost); wrapper-PID fix verified within 0.5% of ground-truth · **v0.2.1 (23 May)** multi-device sampling validated on 4×A40 via RunPod with llama.cpp tensor-split (TP=2 same-socket, TP=4 cross-socket); all four GPUs correctly enumerated, per-device VRAM, SM utilisation, and power timeline captured · **v0.2.1 (24 May)** engine-agnostic claim verified: same `inferscope` binary, same Qwen 2.5 7B model (AWQ quantization), run against vLLM 0.21 serving on H100 SXM. Cold-to-warm sequence captured (TTFT 22.84 ms → 20.71 ms, throughput 242 → 238 tok/s) plus a third data point exposing vLLM's two-tier startup cost (TTFT 651 ms on first request after process restart, recovering to 21 ms on the second). vLLM beats llama.cpp by ~50% on TTFT and ~19% on power per token at the cost of 13× VRAM (aggressive KV pool for batching).

**Hygiene** · MSRV pinned to Rust 1.83 via \`rust-toolchain.toml\` · six Architecture Decision Records covering profiling scope, token timing representation, sysmon correlation, report format, GPU sampling design, and process-tree aggregation · Apache-2.0

---

### OptimEngine — production OR-Tools optimization service

A 4-layer system exposing 11 MCP tools across 4 intelligence levels (9 optimization + 2 utility): flexible job-shop scheduling (FJSP), vehicle routing with time windows (CVRPTW), bin packing, sensitivity analysis, robust optimization, Monte Carlo with CVaR risk metrics, Pareto multi-objective frontier, prescriptive intelligence. Two interfaces: standard REST API and dual-stack MCP (open SSE at \`/mcp\`, OAuth 2.1-gated Streamable HTTP at \`/mcp/v2\`).

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT 9.15 · FastMCP · ScaleKit OAuth 2.1 · OpenTelemetry · Prometheus + Grafana Cloud + Grafana Alloy · Railway · Vercel Edge

**Performance** · Single-solve: provably optimal schedules in 10–40 ms · stochastic CVaR (100 Monte Carlo scenarios) ~2 s · sensitivity analysis (12 params × 5 perturbations) <500 ms · 757 requests / 0 failures across 4 Locust runs, full bottleneck analysis in [BENCHMARKS.md](...)

**Distribution surface** · Smithery.ai MCP registry (9 tools registered) · edge proxy on Vercel for browser MCP access · 36 x402 monetization endpoints live on Base Mainnet and Solana Mainnet (payment-gated solver access for autonomous agents)

**Hygiene** · 121 tests, 77% overall coverage (88% on business-logic engines) · CI on every push · threat model in [SECURITY.md](...) · operational runbook for 5 production incident classes in [RUNBOOK.md](...) · OpenTelemetry distributed tracing live on Grafana Cloud Tempo (manual sub-spans inside the CP-SAT solver entry points) · Grafana Alloy as scrape collector with \`remote_write\` to Grafana Cloud Mimir · Telegram alerting on production events · Dependabot weekly

Live, public, verifiable — the dashboard, the benchmarks, the test suite, and the runbook are all in the open. No "trust me" claims.

---

## Currently working on

**inferscope v0.3 — first-class multi-device GpuMetrics** · The v0.2.1 NVML backend already enumerates and samples all visible GPUs correctly (validated end-to-end on 4×A40 with TP=2 and TP=4 llama.cpp tensor-split runs). The v0.3 work is in the report layer: keep the cluster-aggregate view (still useful for "is the system busy?"), add a \`per_device\` block alongside it (necessary for "which device, doing what, drawing how much power?"), and switch the text report to per-device tables when \`device_count > 1\`. Design recorded in upcoming ADR-007. AMD GPU support via \`amd-smi\` deferred to v0.4.

**OptimEngine on-chain distribution** · Continued buildout of the x402 payment surface across Base and Solana, plus the402.ai service catalog. Real autonomous-agent buyers confirmed on stochastic optimization endpoints.

**Public technical articles** · Cadence ~1 article per month from June 2026 onward (post-consolidation), focused on observability for compute-bound services and Rust profiling of LLM inference.

---

## Recent technical writing

**Upcoming (June 2026)**

- *The profiler had to teach me about the hardware. The hardware taught me about the profiler.* — the L4 → H100 validation arc, including the wrapper-PID bug discovered on L4, the v0.2.1 fix, and what running the same tool on an H100 with a larger model revealed about both the profiler and the hardware budget. **Publishing 15 June 2026.**
- *Four GPUs, two sockets, one workload that didn't need any of it.* — multi-GPU profiling case study on 4×A40 with \`llama.cpp\` tensor-parallel, covering PCIe topology realities, asymmetric tensor splits, idle power tax, and why aggregate metrics hide what per-device metrics tell you. Includes addendum on engine-agnostic validation against vLLM 0.21 on H100. **Publishing 20 June 2026.**

**Already published**

- *Profiling LLM inference: what your /proc sampler isn't telling you* (May 2026) · Bug-discovery narrative behind inferscope v0.2: why a /proc-only view of an inference engine misses the resource that matters most, and how NVML sampling fills the gap.
- *Why your OpenTelemetry trace shows nothing useful when the CPU is doing all the work — a CP-SAT case study* (May 2026) · Why default OpenTelemetry auto-instrumentation fails for compute-bound services (solvers, ML inference, simulation engines). Before/after traces on a real CP-SAT workload showing how manual span instrumentation surfaces what auto-instrumentation hides.
- *How fragile is your weekly plan? A risk-premium framework* (May 2026) · Monte Carlo + CVaR applied to a real OR-Tools schedule. Doubling input volatility raises the risk premium from 4.2% to 7.2% — the plan is structurally robust, and the framework is reproducible via a single API call.
- *How I exposed OR-Tools as a production MCP server* (April 2026) · Building a Model Context Protocol server that wraps Google's constraint solver. What changes when AI agents can call your solver in natural language — and what stays the same about solver-side rigour.

Eight articles in total. Full archive on [the blog](https://michelecampi.github.io).

---

## Background

Nine years building quantitative systems for industrial operations — cost-by-workcenter modeling, margin frameworks, capacity analysis, forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last two years I extended that practice into computational infrastructure: production-grade constraint solvers, observability stacks, MCP server architecture, OAuth-protected APIs, and a Rust profiler for LLM inference engines with NVIDIA GPU sampling validated across three architectures (Ada, Hopper, Ampere), multi-device topologies, and two production inference engines (llama.cpp and vLLM). The path is uncommon — domain depth from nine years inside operations is what makes the optimization work credible, and the technical execution is what makes it useful in production.
