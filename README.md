# Michele Campi

**Cloud Infrastructure Engineer · GPU & LLM inference · AWS, GCP, Kubernetes, Terraform**

I design and run cloud infrastructure on AWS and Google Cloud, from VPC and IAM to
delivery pipelines and GPU inference on Kubernetes, and I put a measured price on it.
Every number on this page links to the evidence behind it.

**Open to remote roles** · Italy (CET), async-first · Italian (native), English (professional) · [LinkedIn](https://www.linkedin.com/in/michele-campi)

## How I work remotely

- **Decisions in writing.** 25 architecture decision records in the operator, each stating its evidence; a STRIDE threat model whose counts CI checks.
- **Budget first, bill after.** Every paid GPU session has a cap approved before it starts and is reconciled against the real bill afterwards.
- **Incidents reviewed in writing.** Three blameless [postmortems](https://github.com/MicheleCampi/llm-inference-platform/blob/main/docs/postmortems/README.md) of real incidents (GPU stockouts in two regions, a billing line nobody estimated, a CI apply that failed halfway), each fix a merged PR; a [platform runbook](https://github.com/MicheleCampi/llm-inference-platform/blob/main/docs/runbook.md) and a [production readiness review](https://github.com/MicheleCampi/llm-inference-platform/blob/main/docs/prr.md) with what is still open.
- **Reproducible by default.** Protocols are fixed before data is collected, results ship with the command that regenerates them, negative results are published.
- **Gaps stated, not hidden.** Every repository lists what is not done yet.
- **End to end.** From Terraform and CI to a Rust operator to fixes merged upstream.

## Selected work

| Repository | What it shows |
|---|---|
| [aws-ecs-delivery-platform](https://github.com/MicheleCampi/aws-ecs-delivery-platform) | ECS Fargate behind an ALB with Cognito, DEV/QA/PROD image promotion, Terraform apply gated through GitHub OIDC. Measured: cut-over in 158 s with 83/83 health checks at 200, rollback in 174 s with 91/91. |
| [gke-llm-inference-platform](https://github.com/MicheleCampi/gke-llm-inference-platform) | Terraform and Argo CD on GKE: scale-to-zero L4 pool, keyless `terraform plan` on every PR, signed release tags. |
| [eks-llm-inference-platform](https://github.com/MicheleCampi/eks-llm-inference-platform) | The AWS twin on EKS, `terraform test` in CI with pinned tools. |
| [vllm-coldstart-operator](https://github.com/MicheleCampi/vllm-coldstart-operator) | Kubernetes operator in Rust: placement, preemption, per-team objectives and cost; 104 unit tests and end-to-end jobs on kind. |
| [inferscope](https://github.com/MicheleCampi/inferscope) | Rust profiler: per-token timing correlated with /proc and NVML; 319 tests. |
| [llm-inference-platform](https://github.com/MicheleCampi/llm-inference-platform) | Entry point: how the pieces fit, threat model, status checked in CI. |

## Stack

- **AWS:** ECS Fargate, ALB, Cognito, ECR, IAM, CodeBuild/CodePipeline, CloudWatch, EKS
- **Google Cloud:** GKE, Workload Identity, billing export
- **Delivery:** Terraform, Argo CD, GitHub Actions with OIDC, signed releases
- **Observability:** Prometheus, Grafana, multiwindow burn-rate alerts tested with promtool
- **Languages:** Rust, Go ([llm-d router, merged](https://github.com/llm-d/llm-d-router/pull/2037)), Python, Bash
- **GPU inference:** vLLM, NVIDIA Dynamo, multi-tenant LoRA, eBPF, NVML

## Measurements behind the design decisions

Every result has a protocol fixed before the data was collected, and negative results are published as results.

- **Cost per token:** one L4 inside its SLOs costs 4.38–8.07 EUR per million generated tokens; every paid GPU session is reconciled against the real bill.
- **Warmth-aware failover** on 3×A10: zero errors on the unaffected service, replacement Ready in 57 s before the old pod drained.
- **Autoscaling that counts warming replicas:** 5.85× fewer replica-seconds than a naive autoscaler (kind + simulator).
- **Agentic workloads:** [+69% tokens/joule at high KV-cache reuse (18 cells); across 15 tool-latency cells generation cost stays flat, and the +56% per million tokens is time spent waiting](https://github.com/MicheleCampi/agentic-kv-energy-experiment).
- **Multi-tenant LoRA:** [8 adapters use +6–7% energy per token, and traffic skew costs nothing measurable](https://github.com/MicheleCampi/lora-multitenancy-experiment).
- **Instruments:** [inferscope](https://github.com/MicheleCampi/inferscope) (Rust, NVML energy, KV hit rate) and an eBPF [cold-start probe](https://github.com/MicheleCampi/vllm-coldstart-probe).

## Upstream

Merged in **llm-d** ([#2037](https://github.com/llm-d/llm-d-router/pull/2037), [#2393](https://github.com/llm-d/llm-d-router/pull/2393), plus [#2822](https://github.com/llm-d/llm-d-router/issues/2822) reported, fixed upstream in [#2848](https://github.com/llm-d/llm-d-router/pull/2848)), **vLLM** ([#55016](https://github.com/vllm-project/vllm/pull/55016), [#54990](https://github.com/vllm-project/vllm/pull/54990)), **NVIDIA AIPerf** ([#1020](https://github.com/ai-dynamo/aiperf/pull/1020)) and **mistral.rs** ([#2189](https://github.com/EricLBuehler/mistral.rs/pull/2189)).

## Background

Eight years building cost and capacity models for manufacturers; Finance and Risk Management degree. That is why my infrastructure starts from the budget.
