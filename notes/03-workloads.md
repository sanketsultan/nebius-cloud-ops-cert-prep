# Domain 3: Running Training & Inference Workloads (~20%)

---

## Slurm vs Kubernetes vs Serverless — When to Use Which

| Scenario | Use |
|----------|-----|
| MPI-based distributed HPC training | Slurm (Soperator) |
| Tightly coupled jobs with queue management | Slurm (Soperator) |
| Containerized training jobs | Kubernetes |
| Inference API server (always-on) | Kubernetes or Serverless AI Endpoint |
| Batch inference (one-off, no persistent infra) | Serverless AI Job |
| Interactive Jupyter notebook | Applications (JupyterLab) |
| Experiment tracking and model registry | MLflow clusters |

---

## Soperator — Deployment Options

Soperator = Nebius open-source solution that combines Slurm + Kubernetes into one infrastructure.

**Four deployment options:**

| Option | Who manages infra | Notes |
|--------|-----------------|-------|
| **Managed Service for Soperator** | Nebius | Deploy in any region with a few clicks. GPU worker nodes require **capacity block groups** |
| **Pro Solution for Soperator** | Nebius experts | Enterprise-scale, customized, reserved capacity, contact sales |
| **Self-deploy on Nebius K8s** | You | Use Terraform recipe from Nebius solution library |
| **Self-deploy on other platforms** | You | Tested only on Nebius; issues via GitHub |

**Important:** GPU worker nodes in Managed Service for Soperator require **capacity block groups** that reserve GPUs in advance.

---

## Key Slurm Commands

```bash
sbatch train.sh          # Submit a batch job
squeue                   # List all jobs in queue
squeue -j <jobid>        # Check specific job + reason for state
scancel <jobid>          # Cancel a job
sinfo                    # Show node status and partitions
sacct -j <jobid>         # Check completed job accounting
```

### Job States

| State | Meaning |
|-------|---------|
| PD (Pending) | Waiting for resources |
| R (Running) | Job is executing |
| CG (Completing) | Job finishing up |
| F (Failed) | Job failed |
| CA (Cancelled) | Job was cancelled |

**PD for >10 minutes?** Check `squeue -j <jobid>` for reason, then `sinfo` to see if nodes are available. Common causes: capacity block not reserved, quota exceeded, nodes down, wrong partition.

---

## Serverless AI

Two deployment types:

| | **Endpoint** | **Job** |
|--|-------------|---------|
| Workflow | Interactive, listens for requests | Non-interactive, terminates on completion |
| Stop/start | Yes | No |
| Public URL | Yes | No |
| Typical lifetime | Hours to days | Minutes to days |
| Use cases | Real-time inference, A/B testing, serving models | Batch preprocessing, training, fine-tuning, batch inference, scientific simulations |

**Billing:** Per-second, usage-based. Only active endpoints and jobs are billed. Scales to zero when not running.

**Under the hood:** Serverless AI runs on Compute containers over VMs — you don't manage the VMs.

---

## MLflow (Managed Service)

- Highly available platform for **ML experiment lifecycle management**
- Track experiments, organize model versions, compare metrics, deploy models
- Access model artifacts, training results, hyperparameters in one interface
- Available in all regions **except** `eu-north2` and `eu-west1`
- Integrates via the MLflow Python SDK from your training jobs

---

## Object Storage

- **AWS S3-compatible API** — works with AWS CLI, boto3, and any S3-compatible tool
- Use `access keys` (service account) for authentication
- Terraform state can be stored in Object Storage (S3 backend)
- Use cases: datasets, model artifacts, inference results, audit log exports
- Available in all regions

```bash
# Using AWS CLI with Nebius Object Storage
aws s3 ls s3://my-bucket \
  --endpoint-url https://storage.eu-north1.nebius.cloud
```

---

## PyTorch Distributed Training

Required environment variables for `torch.distributed`:

```bash
MASTER_ADDR=<node0-ip>    # Rendezvous host
MASTER_PORT=1234           # Rendezvous port
WORLD_SIZE=32              # Total processes (nodes × GPUs per node)
RANK=0                     # Rank of this process (0 to WORLD_SIZE-1)
```

### NCCL + InfiniBand

Force NCCL to use InfiniBand:
```bash
NCCL_IB_DISABLE=0
NCCL_IB_HCA=mlx5_0        # Your IB adapter name (check with ibstat)
NCCL_DEBUG=INFO            # Enable NCCL logging for debugging
```

If bandwidth is low between nodes: check `NCCL_IB_DISABLE` is not 1, IB links are Active (`ibstat`), Network Operator is installed.

---

## Checkpointing

Always implement checkpoint-and-resume for long training jobs:
- Save model state to a **shared filesystem** at regular intervals during training
- After training, transfer checkpoints to **Object Storage** for cost-efficient long-term storage
- On node failure, job resumes from last checkpoint
- Without checkpoints, a node failure restarts the entire job

---

## Storage for Workloads Summary

| Need | Storage |
|------|---------|
| Share dataset across training nodes | Shared filesystem |
| Save checkpoints from multi-node job | Shared filesystem → Object Storage |
| Store trained model artifacts | Object Storage |
| Fast local scratch space (single node) | Network SSD NRD |
| Download training data once, reuse | Object Storage → local copy |
| Share inference results | Object Storage |
| Terraform state file | Object Storage (S3 backend) |

---

## Exam Traps

- Soperator GPU worker nodes require **capacity block groups** — not just any GPU quota
- Four Soperator deployment options — Managed, Pro Solution, Self-deploy on Nebius K8s, Self-deploy other
- Serverless AI has TWO types: **Endpoints** (always-on, public URL) and **Jobs** (one-off, no URL)
- MLflow not available in `eu-north2` and `eu-west1`
- Object Storage uses **AWS S3-compatible API** — authenticate with access keys
- `WORLD_SIZE` = total processes, not total nodes
- NCCL uses IB only if `NCCL_IB_DISABLE=0` and Network Operator is installed
- Checkpoints: shared filesystem during training, Object Storage for long-term
- Object Storage can store Terraform state (S3 backend)
