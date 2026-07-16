Hi, I'm Michele 👋

**AI Infrastructure Engineer.** I design and build infrastructure end-to-end — Rust Kubernetes operators, IaC→GitOps platforms on GKE, observability — with depth in LLM inference (profiling, serving, energy). Proven end-to-end on real GPUs. I trace behaviour to the source and measure what really happens under load; my work is reproducible, documented (ADRs, runbooks), and validated, not demoed. AI-assisted and async-first.

🌐 [inferscope](https://github.com/MicheleCampi/inferscope) · 📊 [OptimEngine live dashboard](https://optimengine.grafana.net/public-dashboards/21137ba340fc4b6e917a4b108db3e109) · ✍️ [Technical writing](https://michelecampi.github.io)

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

**Proven on real GPUs** · validated end-to-end serving Qwen2.5-7B on an NVIDIA L4: the control plane reconciles, the autoscaler brings up the GPU node, vLLM loads and warms, and the VllmService transitions Pending → Warming → Ready while the phase metric streams to Grafana. Getting there meant fixing the assumptions a kind/K3s-only operator carries into a managed cluster — RuntimeClass (GKE uses the device plugin with the default runtime, not an `nvidia` RuntimeClass), the vLLM serving invocation (`vllm serve` args, not env vars), and `LD_LIBRARY_PATH` for the GKE driver mount that the CUDA-12.8+ base image no longer finds.

**Stack** · Rust · kube-rs 2.x · k8s-openapi 0.26 (Kubernetes 1.34) · server-side apply · status subresource · CI with an end-to-end job on an ephemeral kind cluster (asserts the full lifecycle, owner reference, garbage collection) · public OpenMetrics endpoint · two-tag GHCR release pipeline · Apache-2.0

### inferscope — profiler and observability for LLM inference engines
A Rust profiler that drives an OpenAI-compatible inference engine through its HTTP API, captures per-token timing end-to-end, and correlates that timing with the engine process's CPU and GPU resource usage on a single shared wall clock. The point is the correlation: client-side latency and server-side hardware behaviour are two different truths, and the gap between them is where most inference performance problems hide. Outputs a plain-text report for terminal reading and a JSON document carrying both raw signals and derived metrics (TTFT, tokens-per-second excluding TTFT, inter-token latency percentiles, RSS aggregations, VRAM and per-device SM utilisation for multi-GPU runs).

**Stack** · Rust 1.83 · tokio multi-thread runtime · reqwest + SSE streaming · async /proc + NVML sampler with process-tree aggregation · six-crate Cargo workspace with strict separation of concerns (is-core pure types, is-probe network I/O, is-sysmon filesystem + GPU I/O, is-metrics Prometheus scrape, is-report presentation, inferscope CLI orchestrator)

**Validation** · 170 tests (default workspace suite; GPU/energy paths additionally gated behind `gpu-nvidia`) · CI gated on `-D warnings` · validated end-to-end across Ada (L4), Hopper (H100 SXM), and Ampere (4×A40) on Qwen 2.5 from 0.5B to 32B, against both llama.cpp and vLLM · per-device GPU metrics expose the asymmetry that cluster-aggregate readings hide on a TP=2 run (two busy GPUs at ~150 W, two idle at 33 W) — ADR-007 · `--sample-only` mode attaches to a running engine without driving load, the capability behind the Dynamo experiment below — ADR-009 · OTLP/HTTP export via OpenTelemetry 0.32 — ADR-008 · energy and efficiency from the NVML hardware energy counter — tokens-per-joule and tokens-per-watt, validated end-to-end on an A10 (Ampere) against a real llama.cpp workload — ADR-010 · KV-cache hit-rate scraped from the engine's Prometheus endpoint (vLLM prefix-cache, window-delta on `vllm:prefix_cache_hits` / `queries`) by the new is-metrics crate — the second efficiency metric, read on the same shared clock as tokens-per-joule — ADR-011

**Deployment** · multi-stage Dockerfile (rust:1.83-slim → nvidia/cuda runtime, non-root, ~1.65 GB) · public image at `ghcr.io/michelecampi/inferscope` semver-pinned, auto-published by GitHub Action on every `v*` tag · example `deploy/` manifests for docker-compose and a Kubernetes Job

**Hygiene** · MSRV pinned via rust-toolchain.toml · eleven Architecture Decision Records · SECURITY.md with explicit threat model · RUNBOOK.md with failure scenarios from real validation runs (Detection → Diagnosis → Fix) · pre-push hook enforcing fmt + clippy `-D warnings` · Apache-2.0

### Energy signature of KV-cache reuse in agentic workloads — +69% tokens/joule

An 18-run controlled experiment measuring how KV-cache hit rate changes the energy cost of agentic (ReAct-style) inference — Qwen2.5-32B on a single H100 SXM5, vLLM. Three regimes sweep prefix reuse from cold prompts (H0) to heavy shared-prefix reuse (H2): tokens/joule rises monotonically 5.69 → 6.92 → 9.63 — **+69.2% at H2 vs H0** (window-based), std < 3% everywhere, and the gradient survives intact under an injected-failure condition. The figure is a conservative bound: the fixed measurement window's idle tail is longest exactly where efficiency is highest, so exact active-window attribution would widen the gradient, not shrink it. The workload is prefill-dominated by design — constant short generation, the shape of agentic loops with long shared context and short tool-call outputs — so the gradient is a prefill/KV-reuse effect.

**What it demonstrates** · KV-cache hit-rate and hardware energy read on one shared clock (inferscope ADR-011 Prometheus scrape + NVML energy counter, ADR-012 per-phase attribution) · a measurement campaign run to a pre-committed acceptance matrix — 18/18 cells, zero aborts, zero exclusions, anomalies kept and documented · process discipline: mandatory node-off dress rehearsal against a fake engine before every GPU session, established after a hardening cycle that caught a silently-null energy path in the instrumentation itself

**Stack** · vLLM · Qwen2.5-32B bf16 · 1× H100 80GB SXM5 · inferscope (KV scrape + NVML energy) · every number regenerable from committed analysis scripts

*Repository public at article go-live (Aug 2026).*

### CUDA graphs trade-off — when graphs stop paying off

A 40-run controlled experiment isolating one vLLM flag — `enforce_eager` — across two model sizes (Qwen2.5-7B, 32B) and two load regimes, on an H100. The probe above flagged that CUDA graphs cost 3.2× at cold start; this measures what they pay back, and finds something the usual "just enable CUDA graphs" advice misses: the trade-off changes sign. Graphs speed up per-token decode in all eight cells measured (TPOT lower with graphs everywhere) — but on the 32B under sustained load, enabling them makes the *server* 5% slower end-to-end, completing identical work in more wall-clock. Faster kernel, slower server, same run. A separate NVML energy re-run on the same H100 confirms the inversion on a second, independent axis — the saturated 32B spends ~1.8% more joules per output token with graphs on, the energy counter and the throughput benchmark agreeing on one conclusion (per-phase prefill/decode attribution, ADR-012, validated here). The cold-start penalty (+7.4s on 7B, +15.9s on 32B) gives a concrete break-even: ~2,550 requests on the 7B, never on the saturated 32B.

**What it demonstrates** · isolating one variable cleanly and reading two measurement planes against one clock (inferscope for cold-start + per-device GPU, `vllm bench serve` for steady state) · killing the alternative explanations before believing the result — five-rep distributions separated (not noise), ITL ≈ TPOT (not preemption), identical tokens emitted (same work) · a two-factor causal model: the kernel-launch gain shrinks with model size, the captured-shape padding loss grows with queue overload, and their balance sets the sign · stating precisely what is measured versus what remains a hypothesis to validate by instrumenting the scheduler

**Stack** · vLLM 0.23.0 · 1× H100 80GB · Qwen2.5-7B/32B bf16 · inferscope (per-device NVML, full-bench energy window) + vllm bench serve · 12-run energy matrix (tokens/joule, 3 reps/cell) adds energy as a third axis to the throughput study · idempotent Rust-disciplined Python orchestrator · reproducible: every number regenerable from committed analysis scripts — `deep_analysis.py` over 40 throughput result files, `aggregate_energy.py` over the 12 energy JSON, with a PROVENANCE.md pinning hardware/driver/CUDA

Repository public at article go-live (Jul–Aug 2026).

### vllm-coldstart-probe — eBPF profiler for vLLM cold start
A Rust/eBPF tool that traces vLLM cold start at the kernel and driver boundary — the layer where process-level profilers stop. It attaches syscall tracepoints (openat, read, mmap, close) and uprobes on the libcuda C API (cuInit, cuModuleLoadData, cuMemAlloc, cuLaunchKernel), correlating both families on one timeline to answer where the seconds between "process start" and "first token" actually go. Complements inferscope: that profiler looks down from the process, this one looks up from the kernel — cold start is split across exactly the seam where most tools stop.

**Findings** · a four-phase study on Lambda A10/A100 under vLLM 0.22, every number from a capture. Kernel I/O is only ~7% of an ~18s cold start — the dominant cost is GPU warmup and synchronisation, not the disk. Parameters grow 4.6× but load time only 1.5× (sub-linear). Quantization multiplies warmup kernels (AWQ 4.1×, GPTQ 2.4× the cuLaunchKernel count of FP16). Enabling CUDA graphs makes cold start 3.2× slower and issues 79× the kernels — a real trade-off against steady-state speedup, which I went on to measure directly (see *CUDA graphs trade-off* below).

**Stack** · Rust · aya 0.13 eBPF · no_std kernel-side crate · static musl userspace binary · three-crate workspace · Apache-2.0

### Dynamo KV-router under saturation — a performance investigation
An A/B study of NVIDIA Dynamo's KV-aware router against round-robin, on 8×A100, across a scaling curve (N=2/4/8 workers) with a real production trace (Mooncake). The documentation presents the KV-router as faster; I wanted to measure how the benefit behaves as you add capacity. It inverts. Under saturation (N=2) the KV-router isn't "faster" — it sheds ~14% of requests with HTTP 503 to keep latency low for the rest, while round-robin admits everything and lets latency collapse to ~39s. It's a latency-vs-completeness trade-off, not a win, and it vanishes once you're no longer capacity-bound (N=4/8: zero failures either arm, no KV benefit). I traced the mechanism to the Dynamo source at the exact release tag (v1.2.0) — a worker-load monitor created only in KV mode, gated entirely on `--router-mode`.

**What it demonstrates** · reading and reasoning about a large unfamiliar Rust codebase · distributed-systems behaviour under load (load-shedding vs queueing) · triangulating a claim across client metrics (AIPerf), server-side per-device telemetry (inferscope), and source code · rejecting three wrong explanations before landing on the one the data supports

**Stack** · NVIDIA Dynamo 1.2.0 · vLLM runtime · Qwen3-8B · AIPerf fixed-schedule replay · inferscope `--sample-only` for server-side GPU telemetry · 8×A100-SXM4-40GB

[Repository with raw results, analysis scripts, and the full evidence chain →](https://github.com/MicheleCampi/dynamo-kv-router-ab) · *write-up upcoming (June 2026)*

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
- [Disaggregation isn't a deployment topology. In llm-d, it's a per-request decision.](https://michelecampi.github.io/systems-engineering/llm-inference/2026/07/16/llm-d-decode-first-disaggregation.html) — a source read-through of the llm-d EPP scheduler: decode-first orchestration, the prefix-based disaggregation decider, and role-asymmetric recovery — why prefill is an optimization and decode is the contract (Jul 2026)
- [NVIDIA's KV-router isn't faster. Under load it drops requests — and that's the design.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/06/27/dynamo-kv-router-saturation.html) — an A/B across a scaling curve on 8×A100: the KV-router sheds ~14% of requests under saturation to hold latency, traced to the Dynamo source at v1.2.0; the advantage inverts away from saturation (Jun 2026)
- [The profiler had to teach me about the hardware. The hardware taught me about the profiler.](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/06/05/h100-profiler-hardware-utilisation.html) — the L4 → H100 validation arc: a wrapper-PID bug found on L4, the fix, and what an H100 with a larger model revealed about both the profiler and the hardware budget (Jun 2026)
- [Profiling LLM inference: what your /proc sampler isn't telling you](https://michelecampi.github.io/observability/systems-engineering/llm-inference/2026/05/21/profiling-llm-inference-proc-sampler.html) — why a /proc-only view of an inference engine misses the resource that matters most, and how NVML sampling fills the gap (May 2026)
- [Why your OpenTelemetry trace shows nothing useful when the CPU is doing all the work](https://michelecampi.github.io/observability/systems-engineering/2026/05/17/otel-tracing-compute-bound-services.html) — why default auto-instrumentation fails for compute-bound services, with before/after traces on a real CP-SAT workload (May 2026)

[Full archive →](https://michelecampi.github.io)

**Upcoming**
- A cold-start series built on vllm-coldstart-probe: where vLLM cold start actually spends its time, the quantization cost — and the CUDA graphs trade-off, where measuring the full startup-vs-steady-state picture turned up a sign inversion between 7B and 32B that the kernel-level view alone couldn't predict
- *From terraform apply to a warm model* — the GKE inference platform capstone: IaC → GitOps → a served vLLM model, and the managed-GPU debugging it took (Aug 2026)
- *The energy signature of KV-cache reuse* — +69% tokens/joule across hit-rate regimes in agentic workloads, and why the number is a conservative bound (Aug 2026)

## Background
Nine years building quantitative systems for industrial operations — cost-by-workcenter modelling, margin frameworks, capacity analysis, forecasting infrastructure for mid-market manufacturers. Finance and Risk Management degree, 2013.

In the last two years I extended that into computational infrastructure: production constraint solvers, observability stacks, two Rust profilers for LLM inference (one sampling the process from above via /proc + NVML, one tracing the kernel and driver from below via eBPF), a cold-start-aware Kubernetes operator grown into a GPU fleet orchestrator validated under spot preemption, and a full IaC → GitOps → inference platform on GKE proven end-to-end on real GPUs — with a public EKS twin demonstrating the same GitOps contract on AWS. The domain depth is what makes the systems work grounded; the technical execution is what makes it useful in production.
