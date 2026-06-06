Hi, I'm Michele 👋

**Rust systems engineer.** Performance and observability for production systems — demonstrated deep on LLM inference. I trace behaviour to the source code and measure what really happens under load. Async-first, portfolio-driven.

🌐 [inferscope](https://github.com/MicheleCampi/inferscope) · 📊 [OptimEngine live dashboard](https://optimengine.grafana.net/public-dashboards/21137ba340fc4b6e917a4b108db3e109) · ✍️ [Technical writing](https://michelecampi.github.io)

---

## Featured work

### inferscope — profiler and observability for LLM inference engines
A Rust profiler that drives an OpenAI-compatible inference engine through its HTTP API, captures per-token timing end-to-end, and correlates that timing with the engine process's CPU and GPU resource usage on a single shared wall clock. The point is the correlation: client-side latency and server-side hardware behaviour are two different truths, and the gap between them is where most inference performance problems hide. Outputs a plain-text report for terminal reading and a JSON document carrying both raw signals and derived metrics (TTFT, tokens-per-second excluding TTFT, inter-token latency percentiles, RSS aggregations, VRAM and per-device SM utilisation for multi-GPU runs).

**Stack** · Rust 1.83 · tokio multi-thread runtime · reqwest + SSE streaming · async /proc + NVML sampler with process-tree aggregation · five-crate Cargo workspace with strict separation of concerns (is-core pure types, is-probe network I/O, is-sysmon filesystem + GPU I/O, is-report presentation, inferscope CLI orchestrator)

**Validation** · 122 tests · CI gated on `-D warnings` · validated end-to-end across Ada (L4), Hopper (H100 SXM), and Ampere (4×A40) on Qwen 2.5 from 0.5B to 32B, against both llama.cpp and vLLM 0.21 · per-device GPU metrics expose the asymmetry that cluster-aggregate readings hide on a TP=2 run (two busy GPUs at ~150 W, two idle at 33 W) — ADR-007 · `--sample-only` mode attaches to a running engine without driving load, the capability behind the Dynamo experiment below — ADR-009 · OTLP/HTTP export via OpenTelemetry 0.32 — ADR-008

**Deployment** · multi-stage Dockerfile (rust:1.83-slim → nvidia/cuda runtime, non-root, ~1.65 GB) · public image at `ghcr.io/michelecampi/inferscope` semver-pinned, auto-published by GitHub Action on every `v*` tag · example `deploy/` manifests for docker-compose and a Kubernetes Job

**Hygiene** · MSRV pinned via rust-toolchain.toml · nine Architecture Decision Records · SECURITY.md with explicit threat model · RUNBOOK.md with failure scenarios from real validation runs (Detection → Diagnosis → Fix) · pre-push hook enforcing fmt + clippy `-D warnings` · Apache-2.0

### Dynamo KV-router under saturation — a performance investigation
An A/B study of NVIDIA Dynamo's KV-aware router against round-robin, on 8×A100, across a scaling curve (N=2/4/8 workers) with a real production trace (Mooncake). The documentation presents the KV-router as faster; I wanted to measure how the benefit behaves as you add capacity. It inverts. Under saturation (N=2) the KV-router isn't "faster" — it sheds ~14% of requests with HTTP 503 to keep latency low for the rest, while round-robin admits everything and lets latency collapse to ~39s. It's a latency-vs-completeness trade-off, not a win, and it vanishes once you're no longer capacity-bound (N=4/8: zero failures either arm, no KV benefit). I traced the mechanism to the Dynamo source at the exact release tag (v1.2.0) — a worker-load monitor created only in KV mode, gated entirely on `--router-mode`.

**What it demonstrates** · reading and reasoning about a large unfamiliar Rust codebase · distributed-systems behaviour under load (load-shedding vs queueing) · triangulating a claim across client metrics (AIPerf), server-side per-device telemetry (inferscope), and source code · rejecting three wrong explanations before landing on the one the data supports

**Stack** · NVIDIA Dynamo 1.2.0 · vLLM runtime · Qwen3-8B · AIPerf fixed-schedule replay · inferscope `--sample-only` for server-side GPU telemetry · 8×A100-SXM4-40GB

[Repository with raw results, analysis scripts, and the full evidence chain →](https://github.com/MicheleCampi/dynamo-kv-router-ab) · *write-up upcoming (June 2026)*

### vllm-coldstart-probe — eBPF profiler for vLLM cold start
A Rust/eBPF tool that traces vLLM cold start at the kernel and driver boundary — the layer where process-level profilers stop. It attaches syscall tracepoints (openat, read, mmap, close) and uprobes on the libcuda C API (cuInit, cuModuleLoadData, cuMemAlloc, cuLaunchKernel), correlating both families on one timeline to answer where the seconds between "process start" and "first token" actually go. Complements inferscope: that profiler looks down from the process, this one looks up from the kernel — cold start is split across exactly the seam where most tools stop.

**Findings** · a four-phase study on Lambda A10/A100 under vLLM 0.22, every number from a capture. Kernel I/O is only ~7% of an ~18s cold start — the dominant cost is GPU warmup and synchronisation, not the disk. Parameters grow 4.6× but load time only 1.5× (sub-linear). Quantization multiplies warmup kernels (AWQ 4.1×, GPTQ 2.4× the cuLaunchKernel count of FP16). Enabling CUDA graphs makes cold start 3.2× slower and issues 79× the kernels — a real trade-off against steady-state speedup for scale-to-zero.

**Stack** · Rust · aya 0.13 eBPF · no_std kernel-side crate · static musl userspace binary · three-crate workspace · Apache-2.0

### vllm-coldstart-operator — Kubernetes operator for cold-start-aware vLLM
A Rust operator (kube-rs) that treats cold start as a first-class lifecycle signal. Kubernetes marks a pod ready when its process is up; for an LLM server that's the wrong moment — the process is alive but still loading weights and warming the GPU. A VllmService reaches Ready only when it is warm and able to serve. It's the operational half of the cold-start line — the probe measures where cold start goes, this acts on it in-cluster.

**What it does** · VllmService CRD (model, replicas, warmupStrategy: Eager/Graph) · reconcile loop that server-side-applies an owned Deployment with garbage collection · maps warmupStrategy to the probe's Phase D finding about CUDA graphs · derives Pending → Warming → Ready from real Deployment readiness, written to the status subresource

**Stack** · Rust · kube-rs 2.x · k8s-openapi 0.26 (Kubernetes 1.34) · server-side apply · status subresource · CI with an end-to-end job on an ephemeral kind cluster (asserts the full lifecycle, owner reference, garbage collection) · honest about scope: control plane real and tested, data plane a documented placeholder until GPU integration · Apache-2.0

### OptimEngine — production OR-Tools optimisation service
A production constraint-solving service exposing OR-Tools CP-SAT through both a REST API and an MCP interface: flexible job-shop scheduling, vehicle routing with time windows, stochastic optimisation with CVaR risk metrics, sensitivity and Pareto analysis. The reason it's here: it's a real service that has run in production with full observability, not a demo — the engineering discipline transfers regardless of domain.

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT 9.15 · OpenTelemetry distributed tracing · Prometheus + Grafana Cloud (live public dashboard) · Grafana Alloy · Railway · payment-gating layer built on x402 (Base/Solana) as part of the architecture

**Hygiene** · 121 tests, 77% coverage (88% on business-logic engines) · threat model in SECURITY.md · operational runbook for 5 incident classes · OpenTelemetry sub-spans inside the CP-SAT solver entry points · Alloy → Mimir remote_write pipeline · everything live, public, and verifiable — the dashboard, benchmarks, and test suite are in the open

---

## Open-source contributions
Beyond my own repositories, merged contributions to inference/AI-infrastructure projects — evidence of working inside large unfamiliar codebases to the standard their maintainers require:

- **NVIDIA AIPerf** ([#1020](https://github.com/ai-dynamo/aiperf/pull/1020)) — credential redaction
- **mistral.rs** ([#2189](https://github.com/EricLBuehler/mistral.rs/pull/2189)) — Prometheus metrics

---

## Recent technical writing
Cadence ~1 article/month on [michelecampi.github.io](https://michelecampi.github.io).

**Recent**
- [The profiler had to teach me about the hardware. The hardware taught me about the profiler.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/06/05/h100-profiler-hardware-utilisation.html) — the L4 → H100 validation arc: a wrapper-PID bug found on L4, the fix, and what an H100 with a larger model revealed about both the profiler and the hardware budget (Jun 2026)
- [Profiling LLM inference: what your /proc sampler isn't telling you](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/05/21/profiling-llm-inference-proc-sampler.html) — why a /proc-only view of an inference engine misses the resource that matters most, and how NVML sampling fills the gap (May 2026)
- [Why your OpenTelemetry trace shows nothing useful when the CPU is doing all the work](https://michelecampi.github.io/observability/systems-engineering/2026/05/17/otel-tracing-compute-bound-services.html) — why default auto-instrumentation fails for compute-bound services, with before/after traces on a real CP-SAT workload (May 2026)
- [How fragile is your weekly plan? A risk-premium framework](https://michelecampi.github.io/2026/05/04/risk-premium-mid-market-manufacturing.html) — Monte Carlo + CVaR on a real OR-Tools schedule (May 2026)
- [How I exposed OR-Tools as a production MCP server](https://michelecampi.github.io/2026/04/21/how-i-exposed-or-tools-as-mcp.html) — wrapping a constraint solver so AI agents can call it in natural language (Apr 2026)

[Full archive →](https://michelecampi.github.io)

**Upcoming**
- *NVIDIA's KV-router isn't faster — under load it drops requests, and that's the design* — the Dynamo scaling-curve experiment above, write-up in progress (June 2026)
- A two-part cold-start series built on vllm-coldstart-probe: where vLLM cold start actually spends its time, and what quantization and CUDA graphs cost at startup

## Background
Nine years building quantitative systems for industrial operations — cost-by-workcenter modelling, margin frameworks, capacity analysis, forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last two years I extended that into computational infrastructure: production constraint solvers, observability stacks, and two Rust profilers for LLM inference — one sampling the process from above (/proc + NVML, validated across Ada, Hopper, and Ampere, both llama.cpp and vLLM), one tracing the kernel and driver from below (eBPF, syscalls + libcuda). The domain depth is what makes the systems work grounded; the technical execution is what makes it useful in production.
