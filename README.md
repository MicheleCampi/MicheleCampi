Hi, I'm Michele 👋

**AI Platform Engineer.** I build the platforms that serve LLM inference — Kubernetes operators in Rust, Terraform→GitOps on GKE and AWS EKS, observability — so a GPU fleet places and recovers itself, not by hand. I measure what serving costs: tokens per joule, KV-cache reuse, cold start, failover, on real GPUs. Rust for the systems, Python for the measurement harnesses. I trace behaviour to the source; reproducible, documented (ADRs, runbooks), validated, not demoed. AI-assisted and async-first.

🌐 [inferscope](https://github.com/MicheleCampi/inferscope) · ⚙️ [vllm-coldstart-operator](https://github.com/MicheleCampi/vllm-coldstart-operator) · ✍️ [Technical writing](https://michelecampi.github.io)

---

## Featured work

### GKE platform — IaC → GitOps, end to end (serving a real vLLM workload)
The capstone that ties the inference work together: a reproducible Terraform-provisioned GKE cluster (regional, Workload Identity, shielded nodes, scale-to-zero GPU node pool) running an ArgoCD app-of-apps that deploys the cold-start operator, external-secrets (GCP Secret Manager via Workload Identity), and a Grafana Alloy → Mimir observability pipeline — then drives a real vLLM workload on the GPU through it. One `terraform apply` to a served, warm, observable model; one `terraform destroy` back to zero. The phase timeline of a real cold start lands on a Grafana dashboard as the signature artifact.

**What it demonstrates** · platform engineering across the whole path: infrastructure as code, GitOps reconciliation, secret management without secrets in git, in-cluster observability, and GPU workload lifecycle — plus the debugging that only surfaces on real managed GPUs (admission, invocation, dynamic linker), captured as a written post-mortem

**Stack** · Terraform (GCS backend, module structure) · GKE regional + L4 GPU node pool (scale-to-zero, ExtendedResourceToleration) · ArgoCD app-of-apps with sync waves · external-secrets + GCP Secret Manager + Workload Identity · Grafana Alloy + Mimir remote_write · vllm-coldstart-operator serving Qwen2.5-7B

*Repository public at article go-live (Aug 2026); engineering post-mortem written.*

### EKS twin — the same GitOps contract on AWS (public now)
The AWS counterpart of the GKE capstone, built and E2E-validated in a single session: Terraform-provisioned EKS 1.36 (S3 state backend with native lockfile, access entries in API mode), ArgoCD app-of-apps, and the cold-start operator deployed via GitOps — three CRDs served, everything Synced/Healthy, then destroyed back to zero. ~$1 total cost. CPU-only by design: the GPU behaviour of the same operator is measured on the A10 fleet — this repo proves the platform chain is cloud-portable, and states so honestly. Findings documented in-repo: Free Plan instance-type restriction caught at ASG launch, `--server-side` apply required for ArgoCD CRDs, ignoreDifferences generalized in Git and reconciled by ArgoCD.

**Stack** · Terraform (S3 backend, `use_lockfile`) · EKS 1.36, managed node group, API auth mode · ArgoCD app-of-apps · vllm-coldstart-operator via Helm

[Repository, E2E evidence and findings →](https://github.com/MicheleCampi/eks-llm-inference-platform)

### vllm-coldstart-operator — cold-start-aware vLLM serving and GPU fleet orchestration
A Rust operator (kube-rs) that treats cold start as a first-class lifecycle signal. Kubernetes marks a pod ready when its process is up; for an LLM server that's the wrong moment — the process is alive but still loading weights and warming the GPU. A VllmService reaches Ready only when it is warm and able to serve. It's the operational half of the cold-start line — the probe measures where cold start goes, this acts on it in-cluster.

**What it does** · VllmService CRD (model, replicas, warmupStrategy: Eager/Graph, runtimeClassName, extraArgs for engine tuning) · reconcile loop that server-side-applies an owned Deployment with garbage collection · maps warmupStrategy to the probe's Phase D finding about CUDA graphs · derives Pending → Warming → Ready from real Deployment readiness, written to the status subresource and exported as the `vcso_vllmservice_phase` metric

**Fleet orchestration under preemption — measured** · a FleetService CRD orchestrates vLLM placements across GPU nodes using warmth as the placement signal (a warm node with the model cached locally recovers in ~1 minute; a cold one in several). On a spot-preemption notice it surges a replacement onto the warmest survivor, waits for Ready, and only then drains — make-before-break, with a hysteresis cap against thundering herds. Validated on a real 3-node A10 fleet (k3s, vLLM digest-pinned) under closed-loop load, 3 repetitions: **zero errors on the unaffected service in every window of every rep**, replacement Ready in 57s, old pod killed only after (T+58/59s), max service gap 2.3s. Notice injection disclosed as the simulation boundary; full evidence (per-request JSONL, operator logs, k8s events, provenance) committed in-repo. The mechanic was validated first on a zero-cost kind rehearsal harness; the GPU session then reproduced it in ~1.5h for $5.58.

**Efficiency-aware placement — the signal chain, measured** · placement can rank nodes on more than warmth: a per-node reporter samples NVML energy and utilization, scrapes vLLM prefix-cache counters, and joins the two into tokens-per-joule on the same reporting round, publishing all of it to a NodeState status the planner reads. Validated in vivo on a 3-node A10 fleet across 8 repetitions (ABBA+BAAB, ~7 min each): signals populated on every node from real hardware, the two placement strategies diverging deterministically at both decision points, operator log clean throughout. The open question is stated as openly as the result — whether efficiency-aware placement actually improves fleet hit-rate and tokens/joule is not answered by this run, because the load generator drives a fixed endpoint and nothing routes traffic to the service just placed; both arms therefore measure the same node. The near-zero deltas are recorded in the experiment design as a topology limit, not published as a verdict on the strategy.

**Proven on real GPUs** · validated end-to-end serving Qwen2.5-7B on an NVIDIA L4: the control plane reconciles, the autoscaler brings up the GPU node, vLLM loads and warms, and the VllmService transitions Pending → Warming → Ready while the phase metric streams to Grafana. Getting there meant fixing the assumptions a kind/K3s-only operator carries into a managed cluster — RuntimeClass (GKE uses the device plugin with the default runtime, not an `nvidia` RuntimeClass), the vLLM serving invocation (`vllm serve` args, not env vars), and `LD_LIBRARY_PATH` for the GKE driver mount that the CUDA-12.8+ base image no longer finds.

**Stack** · Rust · kube-rs 2.x · k8s-openapi 0.26 (Kubernetes 1.34) · server-side apply · status subresource · CI with an end-to-end job on an ephemeral kind cluster (asserts the full lifecycle, owner reference, garbage collection) · public OpenMetrics endpoint · two-tag GHCR release pipeline · Apache-2.0

### inferscope — profiler and observability for LLM inference engines
A Rust profiler that drives an OpenAI-compatible inference engine through its HTTP API, captures per-token timing end-to-end, and correlates that timing with the engine process's CPU and GPU resource usage on a single shared wall clock. The point is the correlation: client-side latency and server-side hardware behaviour are two different truths, and the gap between them is where most inference performance problems hide. Outputs a plain-text report for terminal reading and a JSON document carrying both raw signals and derived metrics (TTFT, tokens-per-second excluding TTFT, inter-token latency percentiles, RSS aggregations, VRAM and per-device SM utilisation for multi-GPU runs).

**Stack** · Rust 1.85 · tokio multi-thread runtime · reqwest + SSE streaming · async /proc + NVML sampler with process-tree aggregation · six-crate Cargo workspace with strict separation of concerns (is-core pure types, is-probe network I/O, is-sysmon filesystem + GPU I/O, is-metrics Prometheus scrape, is-report presentation, inferscope CLI orchestrator)

**Validation** · 249 tests on the default gate, 254 with all features (`cargo test --workspace` / `--all-features`; GPU/energy paths additionally gated behind `gpu-nvidia`) · CI gated on `-D warnings` · validated end-to-end across Ada (L4), Hopper (H100 SXM), and Ampere (4×A40) on Qwen 2.5 from 0.5B to 32B, against both llama.cpp and vLLM · per-device GPU metrics expose the asymmetry that cluster-aggregate readings hide on a TP=2 run (two busy GPUs at ~150 W, two idle at 33 W) — ADR-007 · `--sample-only` mode attaches to a running engine without driving load, the capability behind the Dynamo experiment below — ADR-009 · OTLP/HTTP export via OpenTelemetry 0.32 — ADR-008 · energy and efficiency from the NVML hardware energy counter — tokens-per-joule and tokens-per-watt, validated end-to-end on an A10 (Ampere) against a real llama.cpp workload — ADR-010 · KV-cache hit-rate scraped from the engine's Prometheus endpoint (vLLM prefix-cache, window-delta on `vllm:prefix_cache_hits` / `queries`) by the new is-metrics crate — the second efficiency metric, read on the same shared clock as tokens-per-joule — ADR-011 · per-step trajectory attribution for agentic workloads (`--steps-file`): energy, token, and KV-cache deltas sliced per LLM call and tool execution, joined offline against driver-emitted step boundaries, with an unattributed remainder that reconciles exactly to the whole-run figure — per-step **energy** measured on an A10 against a live vLLM serving Qwen2.5-7B-Instruct (zero dropped steps, valid at one trajectory in flight); per-step **KV-cache** figures are exercised on fixtures only, not yet on real hardware — ADR-013 · multi-engine metric schema: SGLang read alongside vLLM, with the hit-rate accounting carried into the report because the two engines do not expose the same quantity — vLLM counts hits truncated to a block boundary, SGLang counts exact tokens at its default page size, so the rendered rate declares whether it is a lower bound; the engine is declared with `--engine` and never inferred from the scrape body, since a body of the wrong vocabulary yields no series and any tolerant parse would write that absence as a zero — parser and schema validated against fixtures transcribed from SGLang's own collector source, a live scrape against a running SGLang server still pending GPU — ADR-014

**Deployment** · multi-stage Dockerfile (rust:1.85-slim → nvidia/cuda runtime, non-root, ~1.65 GB) · public image at `ghcr.io/michelecampi/inferscope` semver-pinned, auto-published by GitHub Action on every `v*` tag · current release `v0.5.0` (signed tag, verified on GitHub), with the evidence behind every claim above versioned alongside the code in `validation-results/` · example `deploy/` manifests for docker-compose and a Kubernetes Job

**Hygiene** · MSRV pinned via rust-toolchain.toml · fourteen Architecture Decision Records · SECURITY.md with explicit threat model · RUNBOOK.md with failure scenarios from real validation runs (Detection → Diagnosis → Fix) · documented pre-flight gate in CONTRIBUTING.md, enforced publicly by five CI jobs including one that builds every optional feature · Apache-2.0

### [Energy signature of KV-cache reuse in agentic workloads](https://github.com/MicheleCampi/agentic-kv-energy-experiment) — +69% tokens/joule

An 18-run controlled experiment measuring how KV-cache hit rate changes the energy cost of agentic (ReAct-style) inference — Qwen2.5-32B on a single H100 SXM5, vLLM. Three regimes sweep prefix reuse from cold prompts (H0) to heavy shared-prefix reuse (H2): tokens/joule rises monotonically 5.69 → 6.92 → 9.63 — **+69.2% at H2 vs H0** (window-based), std < 3% everywhere, and the gradient survives intact under an injected-failure condition. The figure is a conservative bound: the fixed measurement window's idle tail is longest exactly where efficiency is highest, so exact active-window attribution would widen the gradient, not shrink it. The workload is prefill-dominated by design — constant short generation, the shape of agentic loops with long shared context and short tool-call outputs — so the gradient is a prefill/KV-reuse effect.

A second result came out of the same evidence: the two bases ADR-012 uses to apportion window energy to prefill and decode diverge monotonically with hit-rate into three non-overlapping bands, and the movement is entirely on the time-share side — `share_prefill_tok` stays flat at 0.996 across all 18 cells because a prompt-token counter has no term that responds to cache reuse, while measured prefill time collapses 91%. Per-token energy attribution is structurally blind to the optimisation it is meant to describe.

**What it demonstrates** · KV-cache hit-rate and hardware energy read on one shared clock (inferscope ADR-011 Prometheus scrape + NVML energy counter, ADR-012 per-phase attribution) · a measurement campaign run to a pre-committed acceptance matrix — 18/18 cells, zero aborts, zero exclusions, anomalies kept and documented · process discipline: mandatory node-off dress rehearsal against a fake engine before every GPU session, established after a hardening cycle that caught a silently-null energy path in the instrumentation itself

**Stack** · vLLM · Qwen2.5-32B bf16 · 1× H100 80GB SXM5 · inferscope (KV scrape + NVML energy) · every number regenerable from committed analysis scripts

*Repository public; [write-up](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/07/30/agentic-kv-energy.html) published 30 July 2026.*

### CUDA graphs trade-off — when graphs stop paying off

A 40-run controlled experiment isolating one vLLM flag — `enforce_eager` — across two model sizes (Qwen2.5-7B, 32B) and two load regimes, on an H100. The probe above flagged that CUDA graphs cost 3.2× at cold start; this measures what they pay back, and finds something the usual "just enable CUDA graphs" advice misses: the trade-off changes sign. Graphs speed up per-token decode in all four comparisons (TPOT lower with graphs in every model × regime cell) — but on the 32B under sustained load, enabling them makes the *server* 5.15% slower end-to-end, completing identical work in more wall-clock. Faster kernel, slower server, same run. A separate NVML energy re-run on the same H100 confirms the inversion on a second, independent axis — the saturated 32B spends ~1.8% more joules per output token with graphs on, the energy counter and the throughput benchmark agreeing on one conclusion (per-phase prefill/decode attribution, ADR-012, validated here). The cold-start penalty (+7.0s on 7B, +15.9s on 32B) gives a concrete break-even that depends on the regime: ~70 requests for a sporadic 7B replica, ~2,450 under sustained load, never on the saturated 32B. Write-up: [CUDA graphs always speed the kernel. They don't always speed the server.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/07/26/cuda-graphs-tradeoff.html) · evidence pinned at tag [`v1.0.0`](https://github.com/MicheleCampi/cuda-graphs-experiment/tree/v1.0.0).

**What it demonstrates** · isolating one variable cleanly and reading two measurement planes against one clock (inferscope for cold-start + per-device GPU, `vllm bench serve` for steady state) · killing the alternative explanations before believing the result — five-rep distributions separated (not noise), ITL ≈ TPOT (not preemption), identical tokens emitted (same work) · a two-factor causal model: the kernel-launch gain shrinks with model size, the captured-shape padding loss grows with queue overload, and their balance sets the sign · stating precisely what is measured versus what remains a hypothesis to validate by instrumenting the scheduler

**Stack** · vLLM 0.23.0 · 1× H100 80GB · Qwen2.5-7B/32B bf16 · inferscope (per-device NVML, full-bench energy window) + vllm bench serve · 12-run energy matrix (tokens/joule, 3 reps/cell) adds energy as a third axis to the throughput study · idempotent Rust-disciplined Python orchestrator · reproducible: every number regenerable from committed analysis scripts — `deep_analysis.py` over 40 throughput result files, `aggregate_energy.py` over the 12 energy JSON, with a PROVENANCE.md pinning hardware/driver/CUDA

*Repository public; [write-up](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/07/26/cuda-graphs-tradeoff.html) published 26 July 2026.*

### vllm-coldstart-probe — eBPF profiler for vLLM cold start
A Rust/eBPF tool that traces vLLM cold start at the kernel and driver boundary — the layer where process-level profilers stop. It attaches syscall tracepoints (openat, read, mmap, close) and uprobes on the libcuda C API (cuInit, cuModuleLoadData, cuMemAlloc, cuLaunchKernel), correlating both families on one timeline to answer where the seconds between "process start" and "first token" actually go. Complements inferscope: that profiler looks down from the process, this one looks up from the kernel — cold start is split across exactly the seam where most tools stop.

**Findings** · a four-phase study on Lambda A10/A100 under vLLM 0.22, every number from a capture. Kernel I/O is only ~7% of an ~18s cold start — the dominant cost is GPU warmup and synchronisation, not the disk. Parameters grow 4.6× but load time only 1.5× (sub-linear). Quantization multiplies warmup kernels (AWQ 4.1×, GPTQ 2.4× the cuLaunchKernel count of FP16). Enabling CUDA graphs makes cold start 3.2× slower and issues 79× the kernels — a real trade-off against steady-state speedup, which I went on to measure directly (see *CUDA graphs trade-off* below).

**Stack** · Rust · aya 0.13 eBPF · no_std kernel-side crate · static musl userspace binary · three-crate workspace · Apache-2.0

### Dynamo KV-router under saturation — a performance investigation
An A/B study of NVIDIA Dynamo's KV-aware router against round-robin, on 8×A100, across a scaling curve (N=2/4/8 workers) with a real production trace (Mooncake). The documentation presents the KV-router as faster; I wanted to measure how the benefit behaves as you add capacity. It inverts. Under saturation (N=2) the KV-router isn't "faster" — it sheds ~14% of requests with HTTP 503 to keep latency low for the rest, while round-robin admits everything and lets latency collapse to ~39s. It's a latency-vs-completeness trade-off, not a win, and it vanishes once you're no longer capacity-bound (N=4/8: zero failures either arm, no KV benefit). I traced the mechanism to the Dynamo source at the exact release tag (v1.2.0) — a worker-load monitor created only in KV mode, gated entirely on `--router-mode`.

**What it demonstrates** · reading and reasoning about a large unfamiliar Rust codebase · distributed-systems behaviour under load (load-shedding vs queueing) · triangulating a claim across client metrics (AIPerf), server-side per-device telemetry (inferscope), and source code · rejecting three wrong explanations before landing on the one the data supports

**Stack** · NVIDIA Dynamo 1.2.0 · vLLM runtime · Qwen3-8B · AIPerf fixed-schedule replay · inferscope `--sample-only` for server-side GPU telemetry · 8×A100-SXM4-40GB

[Repository with raw results, analysis scripts, and the full evidence chain →](https://github.com/MicheleCampi/dynamo-kv-router-ab) · *[write-up](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/06/27/dynamo-kv-router-saturation.html) published 27 June 2026.*

### OptimEngine — deployed OR-Tools optimisation service
A deployed constraint-solving service exposing OR-Tools CP-SAT through both a REST API and an MCP interface: flexible job-shop scheduling, vehicle routing with time windows, stochastic optimisation with CVaR risk metrics, sensitivity and Pareto analysis. The reason it's here: a self-taught project taken all the way to a deployed, observed, continuously running service — [live public dashboard](https://optimengine.grafana.net/public-dashboards/21137ba340fc4b6e917a4b108db3e109) — not a demo — the engineering discipline transfers regardless of domain.

**Stack** · Python 3.12 · FastAPI · OR-Tools CP-SAT 9.15 · OpenTelemetry distributed tracing · Prometheus + Grafana Cloud (live public dashboard) · Grafana Alloy · Railway · payment-gating layer built on x402 (Base/Solana) as part of the architecture

**Hygiene** · 121 tests, 77% coverage (88% on business-logic engines) · threat model in SECURITY.md · operational runbook for 5 incident classes · OpenTelemetry sub-spans inside the CP-SAT solver entry points · Alloy → Mimir remote_write pipeline · everything live, public, and verifiable — the dashboard, benchmarks, and test suite are in the open

---

## Open-source contributions
Beyond my own repositories, merged contributions to inference/AI-infrastructure projects — evidence of working inside large unfamiliar codebases to the standard their maintainers require:

- **NVIDIA AIPerf** ([#1020](https://github.com/ai-dynamo/aiperf/pull/1020)) — credential redaction
- **mistral.rs** ([#2189](https://github.com/EricLBuehler/mistral.rs/pull/2189)) — Prometheus metrics

In review: **llm-d** ([#2037](https://github.com/llm-d/llm-d-router/pull/2037)) — removing mid-stream TPOT predictions from the inference scheduler's Go codebase (−294/+16 across 6 Go files), under review by the assigned maintainer.

---

## Recent technical writing
16 articles since April 2026 (~4/month) on [michelecampi.github.io](https://michelecampi.github.io).

**Recent**

- [KV-cache reuse is an energy lever. Per-token attribution can't see it.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/07/30/agentic-kv-energy.html) — +69.2% tokens/joule from cold prompts to 93% prefix reuse on an H100, and why the token-share half of the energy apportionment never moves (Jul 2026)
- [CUDA graphs always speed the kernel. They don't always speed the server.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/07/26/cuda-graphs-tradeoff.html)
- [Four GPUs, two sockets, one workload that didn't need any of it.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/07/25/multigpu-tensor-parallel-a40.html)
- [The client measured the cost. Only the per-device view measured the trade-off.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/07/18/vllm-disagg-profiling-1p1d.html)
- [Disaggregation isn't a deployment topology. In llm-d, it's a per-request decision.](https://michelecampi.github.io/systems-engineering/llm-inference/2026/07/16/llm-d-decode-first-disaggregation.html) — a source read-through of the llm-d EPP scheduler: decode-first orchestration, the prefix-based disaggregation decider, and role-asymmetric recovery — why prefill is an optimization and decode is the contract (Jul 2026)
- [NVIDIA's KV-router isn't faster. Under load it drops requests — and that's the design.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/06/27/dynamo-kv-router-saturation.html) — an A/B across a scaling curve on 8×A100: the KV-router sheds ~14% of requests under saturation to hold latency, traced to the Dynamo source at v1.2.0; the advantage inverts away from saturation (Jun 2026)

[Full archive →](https://michelecampi.github.io)

**Upcoming**
- *From terraform apply to a warm model* — the GKE inference platform capstone: IaC → GitOps → a served vLLM model, and the managed-GPU debugging it took (Aug 2026)

## Background
Nine years building quantitative systems for industrial operations — cost-by-workcenter modelling, margin frameworks, capacity analysis, forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last two years I extended that into computational infrastructure: deployed constraint solvers, observability stacks, two Rust profilers for LLM inference (one sampling the process from above via /proc + NVML, one tracing the kernel and driver from below via eBPF), a cold-start-aware Kubernetes operator grown into a GPU fleet orchestrator validated under spot preemption, and a full IaC → GitOps → inference platform on GKE proven end-to-end on real GPUs — with a public EKS twin demonstrating the same GitOps contract on AWS. The domain depth is what makes the systems work grounded; the technical execution is what makes it hold up under real load.
