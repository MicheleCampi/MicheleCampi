Hi, I'm Michele 👋

**Cloud Infrastructure Engineer · AI/GPU workloads · cost & performance, measured.**
I build cloud infrastructure on AWS and GKE, from VPC and IAM to delivery pipelines and GPU inference, and I put a measured price on it. Every claim here is checkable with a command.

**Open to remote roles** · Italy (CET), async-first · Italian (native), English (professional) · [LinkedIn](https://www.linkedin.com/in/michele-campi)

🧭 [llm-inference-platform](https://github.com/MicheleCampi/llm-inference-platform), the entry point · ☁️ [aws-ecs-delivery-platform](https://github.com/MicheleCampi/aws-ecs-delivery-platform) · ✍️ [Technical writing](https://michelecampi.github.io)

## The platform

Designed, built and verified under simulated production load; evidence in CI. Built on five months of experiments on real GPUs.

- **Declare:** a `VllmService` with `objective: {ttftSeconds, percentile}` and `budget: {eurPerMonth}`. Out-of-range values are rejected by the CRD and by a CI negative test.
- **Provision and deliver:** Terraform, then an ArgoCD app-of-apps pinned to signed release tags. `terraform plan` runs on every PR through keyless OIDC and needs an approval.
- **Operate:** a Rust operator ([vllm-coldstart-operator](https://github.com/MicheleCampi/vllm-coldstart-operator)) marks a model Ready only when it is warm, and exports the team's objective and budget as metrics.
- **Observe:** multiwindow burn-rate alerts page each team on its own error budget (`1 − percentile/100`), unit-tested with promtool, including mutation tests.
- **Two clouds:** [GKE](https://github.com/MicheleCampi/gke-llm-inference-platform) (end to end on an L4) and an [EKS twin](https://github.com/MicheleCampi/eks-llm-inference-platform). Both are single-region. Edge, DDoS and multi-region exist as design only.
- **AWS delivery:** [ECS Fargate behind an ALB with Cognito](https://github.com/MicheleCampi/aws-ecs-delivery-platform), DEV/QA/PROD image promotion, gated apply via OIDC, zero-downtime cut-over and rollback tested in prod.

What is done, partial and missing is listed in the [roadmap and known gaps](https://github.com/MicheleCampi/llm-inference-platform).

## Measurements behind the design decisions

Every result has a protocol fixed before the data was collected, and negative results are published as results.

- **Cost per token:** one L4 inside its SLOs costs 4.38–8.07 EUR per million generated tokens; every paid GPU session is reconciled against the real bill.
- **Warmth-aware failover** on 3×A10: zero errors on the unaffected service, replacement Ready in 57 s before the old pod drained.
- **Autoscaling that counts warming replicas:** 5.85× fewer replica-seconds than a naive autoscaler (kind + simulator).
- **Agentic workloads:** [+69% tokens/joule from KV reuse; "release the GPU during tool calls" falsified on 15/15 cells](https://github.com/MicheleCampi/agentic-kv-energy-experiment).
- **Multi-tenant LoRA:** [8 adapters use +6–7% energy per token, and traffic skew costs nothing measurable](https://github.com/MicheleCampi/lora-multitenancy-experiment).
- **Instruments:** [inferscope](https://github.com/MicheleCampi/inferscope) (Rust, NVML energy, KV hit rate) and an eBPF [cold-start probe](https://github.com/MicheleCampi/vllm-coldstart-probe).

## Upstream

Merged in **llm-d** ([#2037](https://github.com/llm-d/llm-d-router/pull/2037), [#2393](https://github.com/llm-d/llm-d-router/pull/2393), plus [#2822](https://github.com/llm-d/llm-d-router/issues/2822) reported, fixed upstream in [#2848](https://github.com/llm-d/llm-d-router/pull/2848)), **vLLM** ([#55016](https://github.com/vllm-project/vllm/pull/55016), [#54990](https://github.com/vllm-project/vllm/pull/54990)), **NVIDIA AIPerf** ([#1020](https://github.com/ai-dynamo/aiperf/pull/1020)) and **mistral.rs** ([#2189](https://github.com/EricLBuehler/mistral.rs/pull/2189)).

## Background

Eight years building cost and capacity models for manufacturers; Finance and Risk Management degree. That is why the platform starts from the budget.
