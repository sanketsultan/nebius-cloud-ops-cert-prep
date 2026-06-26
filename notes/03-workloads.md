# Domain 3: Running Training & Inference Workloads (~20%)

---

## Slurm vs Kubernetes — When to Use Which

| Scenario | Use |
|----------|-----|
| MPI-based distributed training | Slurm (Soperator) |
| Tightly coupled HPC jobs | Slurm (Soperator) |
| Containerized training jobs | Kubernetes |
| Inference API server | Kubernetes |
| Batch inference (one-off) | Serverless AI jobs |
| Interactive Jupyter notebook | Applications (JupyterLab) |
| Experiment tracking | MLflow clusters |

---

## Slurm / Soperator

Soperator = Nebius's open-source solution that runs Slurm inside Kubernetes.

**Deployment options:**
- Managed Service for Soperator (Nebius handles it)
- Self-deployed on a Managed Kubernetes cluster
- Pro Solution (deployed by Nebius architects)

### Key Slurm Commands

```bash
sbatch train.sh          # Submit a batch job
squeue                   # List all jobs in queue
squeue -j <jobid>        # Check specific job status + reason
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

**PD for >10 minutes?** Check `squeue -j <jobid>` for reason, then `sinfo` to see if nodes are available. Common causes: quota exceeded, nodes down, wrong partition.

---

## PyTorch Distributed Training

Required environment variables for `torch.distributed`:

```bash
MASTER_ADDR=<node0-ip>   # Rendezvous host
MASTER_PORT=1234          # Rendezvous port
WORLD_SIZE=32             # Total number of processes (nodes x GPUs per node)
RANK=0                    # Rank of this process (0 to WORLD_SIZE-1)
```

### NCCL + InfiniBand

NCCL is the communication library used by PyTorch/TensorFlow for GPU collectives.

Force NCCL to use InfiniBand:
```bash
NCCL_IB_DISABLE=0
NCCL_IB_HCA=mlx5_0       # Your IB adapter name (check with ibstat)
NCCL_DEBUG=INFO           # Enable NCCL logging for debugging
```

If bandwidth is low between nodes: check `NCCL_IB_DISABLE` is not set to 1, and that IB links are active.

---

## Checkpointing

Always implement checkpoint-and-resume for long training jobs:
- Save model state to a **shared filesystem** at regular intervals
- On node failure, the job resumes from the last checkpoint
- Without checkpoints, a node failure restarts the entire job

---

## Inference

**Nebius Applications** provides turnkey inference servers:
- vLLM (for LLMs)
- Open WebUI
- JupyterLab

Deploy from the Applications catalog — no manual container orchestration needed.

**Diagnosing high inference latency:**
1. Check GPU utilization and GPU memory in Observability > Metrics
2. If GPU memory is near 100%: model is too large for available VRAM
3. If GPU compute is near 100%: compute-bound, need more GPUs or batching tuning

---

## Serverless AI

- Runs containerized workloads on-demand, no persistent infrastructure
- Best for: batch inference, one-off jobs, event-driven processing
- Scales to zero when not running (cost-efficient)
- Two modes: **Endpoints** (always-on inference) and **Jobs** (one-off runs)

---

## MLflow Clusters

- Managed MLflow for experiment tracking, metric logging, model registry
- Integrates with training jobs via the MLflow Python SDK
- Model artifacts can be stored in Object Storage

---

## Storage for Workloads

| Need | Storage |
|------|---------|
| Share dataset across training nodes | Shared filesystem |
| Save checkpoints from multi-node job | Shared filesystem |
| Store trained model artifacts | Object Storage |
| Fast local scratch space (single node) | Local NVMe |
| Download training data once | Object Storage → local copy |

---

## Exam Traps

- `WORLD_SIZE` = total processes, not total nodes
- NCCL uses InfiniBand by default only if `NCCL_IB_DISABLE=0` and IB adapters are present
- Serverless AI Jobs ≠ Serverless AI Endpoints (jobs are one-off, endpoints are persistent)
- MLflow = experiment tracking; it does not run the training itself
- Checkpoints must go to shared storage, not local disk, for multi-node jobs
