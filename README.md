Hi, I'm Michele 👋
Rust + AI infrastructure. I build observability for LLM systems and production-grade optimization services. Nine years inside industrial operations before turning that domain depth into code.

🌐 [inferscope on GitHub](https://github.com/MicheleCampi/inferscope) · 📊 [OptimEngine live dashboard](https://optimengine.grafana.net/public-dashboards/21137ba340fc4b6e917a4b108db3e109) · ✍️ [Technical writing](https://michelecampi.github.io)

## Featured work

### inferscope — profiler and observability for LLM inference engines

A Rust profiler that drives an OpenAI-compatible inference engine through its HTTP API, captures per-token timing end-to-end, and correlates that timing with the engine process's CPU and GPU resource usage on a single shared wall clock. Outputs a plain-text report for terminal reading and a JSON document carrying both raw signals and derived metrics (TTFT, tokens-per-second excluding TTFT, inter-token latency percentiles, RSS aggregations, VRAM and SM utilization with per-device breakdown for multi-GPU runs).

**Stack** · Rust 1.83 · tokio multi-thread runtime · reqwest + SSE streaming · async /proc + NVML sampler with process-tree aggregation · five-crate Cargo workspace with strict separation of concerns (is-core pure types, is-probe network I/O, is-sysmon filesystem + GPU I/O, is-report presentation, inferscope CLI orchestrator)

**Validation** · 121 tests · CI gated on -D warnings (no unused imports, no clippy lints) · integration test on a synthetic bash + sleep parent-child pair exercises the v0.2.1 aggregation path end-to-end · v0.1.0 (16 May) tested end-to-end against llama.cpp b9165 with Qwen 2.5 0.5B Q4 on Ubuntu 24.04 / x86_64: TTFT 25 ms, 82.7 tokens/s, 588 MiB RSS · v0.2.0 (20 May) validated on NVIDIA RTX L4 via RunPod, same model: 381 tokens/s and 13 ms TTFT (warm), SM utilization peak 91% / mean 58%, VRAM 1.34 GB stable, power 37–39 W — ~4.6× throughput vs CPU baseline · v0.2.1 (22 May) validated on NVIDIA H100 SXM via RunPod, two-model contrast: Qwen 2.5 7B Q4 → 230 tokens/s, SM mean 48%, VRAM 5.6 GB / 80 GB, power mean 170 W (chip loafing); Qwen 2.5 32B Q4 → 69 tokens/s, SM mean 88%, VRAM 21.4 GB / 80 GB, power mean 439 W (chip earning its cost); wrapper-PID fix verified within 0.5% of ground-truth · v0.2.1 (23 May) multi-device sampling validated on 4×A40 via RunPod with llama.cpp tensor-split (TP=2 same-socket, TP=4 cross-socket); all four GPUs correctly enumerated, per-device VRAM, SM utilisation, and power timeline captured · v0.2.1 (24 May) engine-agnostic claim verified: same inferscope binary, same Qwen 2.5 7B model (AWQ quantization), run against vLLM 0.21 serving on H100 SXM. Cold-to-warm sequence captured (TTFT 22.84 ms → 20.71 ms, throughput 242 → 238 tok/s) plus a third data point exposing vLLM's two-tier startup cost (TTFT 651 ms on first request after process restart, recovering to 21 ms on the second). vLLM beats llama.cpp by ~50% on TTFT and ~19% on power per token at the cost of 13× VRAM (aggressive KV pool for batching) · v0.3.0 (25 May) released with first-class per-device GPU metrics in the JSON output and per-device block in the text report — the asymmetry that cluster-aggregate readings hide on a TP=2 run (two busy GPUs at 148/152 W each, two idle ones at 33 W) is now visible without consulting the human summary; documented in ADR-007.

**Deployment** · Multi-stage Dockerfile (rust:1.83-slim builder → nvidia/cuda:13.0.2-runtime runtime, non-root UID 1000, ~1.65 GB image) · public image at ghcr.io/michelecampi/inferscope with semver-pinned tags (0.3.0, 0.3, 0, latest) auto-published by GitHub Action on every v*.*.* git tag · example deploy/ manifests with docker-compose for local runs and a Kubernetes Job manifest for cluster runs (NVIDIA Device Plugin resource request, backoffLimit: 0, design trade-offs documented in the directory README)

**Reproducible benchmarks** · The benchmarks/ directory contains three verified cross-hardware case studies. Every number is pulled directly from inferscope's JSON output or per-run summary report, never from memory: cross-hardware comparison (L4, H100 SXM, 4×A40) on three Qwen 2.5 sizes; multi-device deep-dive on 4×A40 with TP=2 vs TP=4 (data shows TP=4 cross-socket is statistically indistinguishable from TP=2 single-socket for a 7B model — against the going-in hypothesis); vLLM vs llama.cpp head-to-head on H100 with the cold/warm-outlier/warm-steady three-run methodology that exposes the cudagraph capture stall.

**Hygiene** · MSRV pinned to Rust 1.83 via rust-toolchain.toml · seven Architecture Decision Records covering profiling scope, token timing representation, sysmon correlation, report format, GPU sampling design, process-tree aggregation, and per-device GPU metrics · SECURITY.md with explicit threat model and known limitations (single-maintainer SPOF, unsigned image) · RUNBOOK.md with seven failure scenarios from real validation runs structured Detection → Diagnosis → Fix · pre-push git hook enforces cargo fmt --all --check and RUSTFLAGS="-D warnings" cargo clippy --workspace --all-targets before every push · Apache-2.0

### vllm-coldstart-probe — eBPF profiler for vLLM cold start

A Rust/eBPF tool that traces vLLM cold start at the kernel and driver boundary, the layer where process-level profilers stop. It attaches syscall tracepoints (`openat`, `read`, `mmap`, `close`, each enter+exit) and uprobes/uretprobes on the libcuda C API (`cuInit`, `cuModuleLoadData`, `cuMemAlloc_v2`, `cuLaunchKernel`, each entry+return), correlating both families on one timeline to answer where the seconds between "process start" and "first token" actually go. Complements inferscope: that profiler looks down from the process, this one looks up from the kernel — cold start is split across exactly the seam where most tools stop.

**Stack** · Rust · aya 0.13 eBPF · `no_std` kernel-side crate · static musl userspace binary (runs on any x86_64 kernel ≥5.8 with BTF) · stdlib-only Python analysis · three-crate workspace (probe-common shared `#[repr(C)]` types, probe-ebpf kernel programs, probe userspace loader)

**Findings** · A four-phase study on Lambda Labs A10/A100 under vLLM 0.22, every number from a capture, none from memory. **Phase A** (where time goes, Mistral-7B FP16): kernel I/O is ~7% of an ~18 s cold start; the dominant cost is GPU warmup and synchronization between the traced calls, not the disk. **Phase B** (scaling, Qwen2.5-AWQ 7B/14B/32B): parameters grow 4.6×, load time only 1.5× — strongly sub-linear; kernel I/O is flat across sizes. **Phase C** (quantization, same Qwen2.5-7B in FP16/AWQ/GPTQ): identical architecture, but quantization multiplies warmup kernels — AWQ 4.1× and GPTQ 2.4× the `cuLaunchKernel` count of FP16 (dequantization kernels), and AWQ ≠ GPTQ at cold start. **Phase D** (workarounds): enabling CUDA graphs (`enforce_eager=False`) makes cold start 3.2× slower and issues 79× the kernels (66,353 vs 843) as vLLM captures every batch shape — a real trade-off against steady-state speedup for scale-to-zero; a warm page cache saves ~3 s.

**Hygiene** · pre-push CI gate (`cargo fmt --all --check` and `RUSTFLAGS="-D warnings" cargo clippy --workspace --all-targets`) · README documents all four study phases with per-phase findings tables · datasets reproducible via the included analysis script · Apache-2.0

### vllm-coldstart-operator — Kubernetes operator for cold-start-aware vLLM

A Rust operator (kube-rs) that manages vLLM inference replicas and treats cold start as a first-class lifecycle signal. Kubernetes marks a pod ready when its process is up; for an LLM server that is the wrong moment — the process is alive but still loading weights and warming the GPU. The operator models that gap: a `VllmService` reaches `Ready` only when it is warm and able to serve, not merely running. It is the operational half of the cold-start line — the probe measures where cold start goes, this acts on it in cluster.

**What it does** · `VllmService` CRD (`model`, `replicas`, `warmupStrategy`: Eager/Graph) · reconcile loop that server-side-applies an owned Deployment with an owner reference (automatic garbage collection on delete) · maps `warmupStrategy` to pod config, the operational lever behind the probe's Phase D finding that CUDA graphs make cold start ~3× slower · derives a `Pending → Warming → Ready` phase from real Deployment readiness, written to the status subresource (no reconcile loop)

**Stack** · Rust · kube-rs 2.x · k8s-openapi 0.26 (Kubernetes 1.34 API) · tokio · server-side apply · status subresource

**Hygiene** · unit tests on the lifecycle logic · CI with two jobs: fmt + clippy (`-D warnings`) + test + build, and an end-to-end job that spins up an ephemeral kind cluster and asserts the full lifecycle (Deployment created, status reaches Ready, owner reference set, garbage-collected on delete) with bounded polling, not fixed sleeps · two ADRs (why Rust/kube-rs over Go; cold start as a first-class state) · RUNBOOK · honest about scope: the control plane is real and tested, the data plane is a documented placeholder until GPU integration · Apache-2.0

### OptimEngine — production OR-Tools optimization service

A 4-layer system exposing 11 MCP tools across 4 intelligence levels (9 optimization + 2 utility): flexible job-shop scheduling (FJSP), vehicle routing with time windows (CVRPTW), bin packing, sensitivity analysis, robust optimization, Monte Carlo with CVaR risk metrics, Pareto multi-objective frontier, prescriptive intelligence. Two interfaces: standard REST API and dual-stack MCP (open SSE at /mcp, OAuth 2.1-gated Streamable HTTP at /mcp/v2).

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT 9.15 · FastMCP · ScaleKit OAuth 2.1 · OpenTelemetry · Prometheus + Grafana Cloud + Grafana Alloy · Railway · Vercel Edge

**Performance** · Single-solve: provably optimal schedules in 10–40 ms · stochastic CVaR (100 Monte Carlo scenarios) ~2 s · sensitivity analysis (12 params × 5 perturbations) <500 ms · 757 requests / 0 failures across 4 Locust runs, full bottleneck analysis in BENCHMARKS.md

**Distribution surface** · Smithery.ai MCP registry (9 tools registered) · edge proxy on Vercel for browser MCP access · 36 x402 monetization endpoints live on Base Mainnet and Solana Mainnet (payment-gated solver access for autonomous agents)

**Hygiene** · 121 tests, 77% overall coverage (88% on business-logic engines) · CI on every push · threat model in SECURITY.md · operational runbook for 5 production incident classes in RUNBOOK.md · OpenTelemetry distributed tracing live on Grafana Cloud Tempo (manual sub-spans inside the CP-SAT solver entry points) · Grafana Alloy as scrape collector with remote_write to Grafana Cloud Mimir · Telegram alerting on production events · Dependabot weekly

Live, public, verifiable — the dashboard, the benchmarks, the test suite, and the runbook are all in the open. No "trust me" claims.

## Currently working on

Public technical writing, observability for compute-bound services and Rust profiling of LLM inference. Two inferscope validation pieces land in June 2026 (the L4 → H100 arc and the multi-device 4×A40 case study); a two-part cold-start series follows, built on vllm-coldstart-probe. Cadence target ~1 article per month, post-consolidation after a denser April–May.

inferscope v0.3.1+ patches as feedback surfaces from real-world use of the v0.3.0 image. The release shipped clean (121 tests green, CI verde, image publicly pullable), but production exposure tends to find what local validation doesn't.

OptimEngine on-chain distribution. Continued buildout of the x402 payment surface across Base and Solana, plus the402.ai service catalog. Real autonomous-agent buyers confirmed on stochastic optimization endpoints.

## Recent technical writing

**Upcoming (June 2026)**

The profiler had to teach me about the hardware. The hardware taught me about the profiler. — the L4 → H100 validation arc, including the wrapper-PID bug discovered on L4, the v0.2.1 fix, and what running the same tool on an H100 with a larger model revealed about both the profiler and the hardware budget.

Four GPUs, two sockets, one workload that didn't need any of it. — multi-GPU profiling case study on 4×A40 with llama.cpp tensor-parallel, covering PCIe topology realities, asymmetric tensor splits, idle power tax, and why aggregate metrics hide what per-device metrics tell you. Includes addendum on engine-agnostic validation against vLLM 0.21 on H100.

**Next up (cold-start series, dates TBA)**

Where vLLM cold start actually spends its time — and why it isn't the disk. — an eBPF probe traces syscalls and libcuda across four models from 7B to 32B; kernel I/O turns out to be ~7% of an 18-second start, the rest is GPU warmup and synchronization. Built on vllm-coldstart-probe.

Every vLLM optimization I tried made cold start worse — except one. — measuring what quantization (FP16 vs AWQ vs GPTQ) and CUDA graphs cost at startup. CUDA graphs make cold start 3.2× slower while speeding up steady-state inference; the quantization scheme you pick has a cold-start cost on top of its VRAM profile.

**Already published**

Profiling LLM inference: what your /proc sampler isn't telling you (May 2026) · Bug-discovery narrative behind inferscope v0.2: why a /proc-only view of an inference engine misses the resource that matters most, and how NVML sampling fills the gap.

Why your OpenTelemetry trace shows nothing useful when the CPU is doing all the work — a CP-SAT case study (May 2026) · Why default OpenTelemetry auto-instrumentation fails for compute-bound services (solvers, ML inference, simulation engines). Before/after traces on a real CP-SAT workload showing how manual span instrumentation surfaces what auto-instrumentation hides.

How fragile is your weekly plan? A risk-premium framework (May 2026) · Monte Carlo + CVaR applied to a real OR-Tools schedule. Doubling input volatility raises the risk premium from 4.2% to 7.2% — the plan is structurally robust, and the framework is reproducible via a single API call.

How I exposed OR-Tools as a production MCP server (April 2026) · Building a Model Context Protocol server that wraps Google's constraint solver. What changes when AI agents can call your solver in natural language — and what stays the same about solver-side rigour.

Full archive on the blog.

## Background

Nine years building quantitative systems for industrial operations — cost-by-workcenter modeling, margin frameworks, capacity analysis, forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last two years I extended that practice into computational infrastructure: production-grade constraint solvers, observability stacks, MCP server architecture, OAuth-protected APIs, and two Rust profilers for LLM inference — one sampling the process from above (/proc + NVML, validated across Ada, Hopper, and Ampere, multi-device topologies, and both llama.cpp and vLLM), one tracing the kernel and driver from below (eBPF, syscalls + libcuda). The path is uncommon — domain depth from nine years inside operations is what makes the optimization work credible, and the technical execution is what makes it useful in production.

