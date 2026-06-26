# Nebius Docs Reading Guide

Prioritized by exam weight. Read in this order.

---

## Priority 1 — Domain 2: GPU Clusters (35% of exam)

Read these first, they're the most tested.

| Page | URL | Why It Matters |
|------|-----|---------------|
| VM types and GPUs | docs.nebius.com/compute/virtual-machines/types | GPU families, instance specs |
| GPU clusters with InfiniBand | docs.nebius.com/compute/clusters/gpu | How to create and configure IB clusters |
| Testing InfiniBand | docs.nebius.com/compute/clusters/gpu/test | ibstat, NCCL test commands |
| Storage types | docs.nebius.com/compute/storage/types | Disk vs shared filesystem vs object storage |
| Attaching/mounting volumes | docs.nebius.com/compute/storage/use | How to actually use storage on VMs |
| Setting up GPUs in K8s | docs.nebius.com/kubernetes/gpu/set-up | Device plugin, GPU operator |
| InfiniBand in K8s | docs.nebius.com/kubernetes/gpu/clusters | IB for Kubernetes workloads |
| Node group management | docs.nebius.com/kubernetes/node-groups/manage | Creating, scaling, maintaining node groups |
| Monitoring VMs | docs.nebius.com/compute/monitoring/virtual-machines | GPU metrics, alerts |

---

## Priority 2 — Domain 4: Automation & Observability (25% of exam)

| Page | URL | Why It Matters |
|------|-----|---------------|
| Terraform provider | docs.nebius.com/terraform-provider | Resource types, provider config |
| CLI reference | docs.nebius.com/cli | Common commands |
| Metrics and Alerts | docs.nebius.com/observability/monitoring | Setting up GPU alerts |
| Logs | docs.nebius.com/observability/logging | Log search and filtering |
| Traces | docs.nebius.com/observability/traces | Distributed tracing |
| K8s monitoring | docs.nebius.com/kubernetes/monitoring | Cluster and node health |
| VM monitoring | docs.nebius.com/compute/monitoring/virtual-machines | Per-node GPU metrics |

---

## Priority 3 — Domain 1: Security & Billing (20% of exam)

| Page | URL | Why It Matters |
|------|-----|---------------|
| IAM overview | docs.nebius.com/iam/overview | How IAM works, scopes |
| Service accounts | docs.nebius.com/iam/service-accounts/manage | Creating and using service accounts |
| Groups and roles | docs.nebius.com/iam/authorization/groups | Role assignment |
| KMS | docs.nebius.com/kms | Encryption key management |
| MysteryBox | docs.nebius.com/mysterybox | Secrets management |
| Audit Logs | docs.nebius.com/audit-logs | Who did what and when |
| Billing | docs.nebius.com/signup-billing | Budgets, alerts, invoices |

---

## Priority 4 — Domain 3: Workloads (20% of exam)

| Page | URL | Why It Matters |
|------|-----|---------------|
| Soperator overview | docs.nebius.com/slurm-soperator | What Soperator is |
| Deployment methods | docs.nebius.com/slurm-soperator/deploy/overview | Managed vs self-deployed |
| Connecting to Slurm nodes | docs.nebius.com/slurm-soperator/clusters/connect | SSH into login/worker nodes |
| Running batch jobs | docs.nebius.com/slurm-soperator (jobs section) | sbatch, squeue, scancel |
| Managing jobs | docs.nebius.com/slurm-soperator (management section) | Job states, queue management |
| MLflow clusters | docs.nebius.com/mlflow | Experiment tracking |
| Serverless AI | docs.nebius.com/serverless | Jobs and endpoints |

---

## Skip (Low Exam Weight)

- Container Registry (skim only)
- PostgreSQL clusters (skim only)
- VPC deep-dives (covered enough in cluster setup)
- Applications catalog details (know JupyterLab and vLLM exist)
- Digital rights / GDPR pages
- gRPC API reference

---

## Quick Reference: Key URLs

```
https://docs.nebius.com/compute/clusters/gpu         ← most important page
https://docs.nebius.com/kubernetes/gpu/set-up
https://docs.nebius.com/slurm-soperator
https://docs.nebius.com/iam/overview
https://docs.nebius.com/terraform-provider
https://docs.nebius.com/observability/monitoring
```
