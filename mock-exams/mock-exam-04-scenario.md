# Nebius AI Cloud Ops — Mock Exam 4 (Scenario-Based, Hard)

**Format:** All scenarios. No recall — every question requires applied reasoning.
**Time:** ~55 minutes | **Pass mark:** 70% (28/40)

> Attempt all questions before checking the answer key at the bottom.

---

## Domain 1: Security, Compliance & Billing (Q1–Q8)

**Q1.** Your Nebius tenant has three projects: `research`, `staging`, `production`. A new ML engineer joins and needs to: read datasets from `research`, submit training jobs to `staging`, but have zero access to `production`. Nebius IAM groups are global to the tenant. How do you correctly implement this access model?

- A) Add the engineer to the `viewers` group — they can read research, but cannot submit jobs to staging
- B) Add the engineer to the `editors` group in `research` and `staging` only — but IAM groups apply tenant-wide, so this grants editor access to production too
- C) Nebius IAM groups are tenant-wide. Use resource-level permissions or separate the engineer into a service account scoped to specific projects. If the platform doesn't support project-level group scoping, create a dedicated service account for staging job submission.
- D) Add to `viewers` tenant-wide, then add to `editors` for research project only

---

**Q2.** During a security review, you discover a service account named `training-runner` has both an access key (for Object Storage) AND an authorized key (for Nebius API). The SA is in the `admins` group. It was created 18 months ago and no one knows who created it or why. What is the safest remediation approach?

- A) Delete the SA immediately to eliminate the risk
- B) Audit Logs: search for API calls made by `training-runner` in the last 90 days to understand what it actually does. If it's active, create a replacement SA with minimum required permissions, rotate credentials in consuming systems, then delete `training-runner`.
- C) Remove it from `admins`, downgrade to `editors`, and rotate the keys
- D) Keep the SA but revoke the authorized key — access keys are lower risk

---

**Q3.** A model weights file was accidentally deleted from Object Storage. Your team had versioning disabled on the bucket. The file was 1.2 TB and took 14 days to train. What does Nebius Object Storage offer to help recover this, and what is the actual answer?

- A) Nebius Object Storage keeps 30-day snapshots automatically — restore from snapshot
- B) With versioning disabled, deleted objects are permanently gone. Object Storage does not retain deleted data if versioning is off. This is an unrecoverable loss. Enable versioning on all critical buckets going forward.
- C) Contact Nebius support — they can restore from their internal backups
- D) The file is in the recycle bin for 7 days

---

**Q4.** Your security team requires that all data at rest in Nebius use customer-managed encryption keys (CMEK). You have a Network SSD attached to a GPU VM. Which statement is accurate?

- A) Network SSDs require CMEK — you must create a KMS key and attach it during disk creation
- B) Network SSDs in Nebius are always encrypted by default. You cannot use customer-managed KMS keys for Network SSD encryption — the platform handles it. CMEK is available for other services like Object Storage.
- C) Network SSDs are not encrypted by default — you must enable encryption manually
- D) Encryption on Network SSD is optional; enable it at creation time with your KMS key

---

**Q5.** An attacker gained access to a service account's authorized key file. The SA is in the `editors` group. In the 4 hours before you detected the breach, what is the maximum damage the attacker could have done?

- A) Only read Object Storage objects — authorized keys have read-only access
- B) An authorized key enables Nebius API access with the SA's IAM permissions. As an editor, the attacker could: create/delete VMs, modify node groups, read all data in accessible buckets, create new service accounts, and incur significant compute costs. Everything an editor can do.
- C) Authorized keys only work for Kubernetes API calls
- D) Create and delete resources but not read data — editors have no read access

---

**Q6.** You need to share a secret (a Hugging Face token) between two projects in the same Nebius tenant. The token needs to be accessible to pods in both projects. MysteryBox is a single-project service. How do you handle this?

- A) Store the token in a Kubernetes Secret in each project's cluster — manually copy it
- B) MysteryBox secrets are project-scoped. To share across projects: store the token in each project's MysteryBox separately (or use one project's secret and grant cross-project read access if supported). Alternatively, use a single service account that both projects' workloads can authenticate as.
- C) Use Object Storage to store the token file — it supports cross-project access
- D) MysteryBox supports cross-project sharing natively — use the `shared` flag

---

**Q7.** Your billing alerts are configured at 80% and 100% of your monthly budget. The 80% alert fires on day 8 of the month. Your month started on the 1st. What does this tell you about your burn rate, and what should you do?

- A) Nothing — alerts have a 48-hour delay, this is normal
- B) If you hit 80% of monthly budget on day 8, your burn rate is roughly 10% per day. You'll exhaust the budget by day 10. Immediately identify the cost driver (Billing reports by resource type), kill any runaway jobs, and either cut spend or request a budget increase before day 10.
- C) Increase the budget alert threshold to 150% to avoid false alarms
- D) Check if the billing month reset properly — this might be a billing system bug

---

**Q8.** An auditor asks for a list of all changes made to IAM groups over the past 6 months. Where do you get this?

- A) From the Nebius Monitoring dashboard — IAM changes are logged as metrics
- B) Audit Logs — filter by service `iam`, event types `AddMember`, `RemoveMember`, `CreateGroup`, `DeleteGroup` for the past 6 months. Export to a spreadsheet for the auditor.
- C) From MysteryBox audit trail
- D) IAM changes are not logged in Nebius

---

## Domain 2: Setting Up & Operating GPU Clusters (Q9–Q22)

**Q9.** You want to provision 512 H100 GPUs for a large training run. The GPUs must all be on the same InfiniBand fabric. You're in `eu-north1`. What is the maximum number of GPUs you can put on a single InfiniBand fabric in that region, and how many VMs does this require?

- A) 256 GPUs max per fabric (32 VMs × 8 GPUs) — can't do 512 on one fabric
- B) 512 H100 GPUs = 64 VMs (8 GPUs each). Multiple InfiniBand fabrics exist in `eu-north1` (fabric-2/3/4/6) but each fabric is independent. 512 GPUs on a single fabric is possible if the fabric has that capacity — contact Nebius to reserve capacity block.
- C) H100 GPUs don't support InfiniBand in eu-north1
- D) No limit — InfiniBand fabrics scale automatically

---

**Q10.** Your 16-node H200 cluster suddenly shows near-zero NCCL bandwidth (`< 1 GB/s`) after a network maintenance window. Before maintenance, all-reduce bandwidth was 380 GB/s. `ibstat` shows all ports Active. What is the most likely cause?

- A) NCCL needs to be reinstalled
- B) The InfiniBand switch was reconfigured during maintenance and P-Key (partition key) isolation broke. Your GPU cluster's P-Key may have been reset, causing cross-cluster traffic leakage or loss of fabric membership. Run `ibstat` + check P-Key assignments with `ibswitches` or contact Nebius network support.
- C) GPU memory is corrupted — run ECC check
- D) NCCL_IB_DISABLE was set to 1 during maintenance and not reset

---

**Q11.** This Terraform config provisions a Kubernetes cluster:

```hcl
resource "nebius_mk8s_v1_cluster" "training" {
  parent_id = var.project_id
  name      = "training-cluster"
  
  control_plane {
    version              = "1.31"
    subnet_id            = var.subnet_id
    etcd_cluster_size    = 1
    public_endpoint = true
  }
}
```

After applying, your team says the cluster is not suitable for production. What are the two issues?

- A) The name contains a hyphen (invalid) and version 1.31 is deprecated
- B) (1) `etcd_cluster_size = 1` means no HA — if the single etcd dies, the cluster is lost. Production requires 3. (2) Version is now immutable after creation — you'll need to recreate to upgrade. Consider using the latest available version (1.33) from the start.
- C) `public_endpoint = true` is insecure and etcd size is wrong
- D) The `parent_id` field is invalid in v1 clusters

---

**Q12.** A GPU node shows `STATUS: NotReady` for 10 minutes after a pod crash. `kubectl describe node gpu-node-05` shows:

```
Conditions:
  Type              Status
  MemoryPressure    False
  DiskPressure      False
  PIDPressure       False
  Ready             Unknown
```

`kubectl get pods -n kube-system | grep gpu-node-05` shows the kubelet pod is not running on that node. What do you do?

- A) Delete and recreate the node
- B) The kubelet is down on that node. SSH to the node and check: `systemctl status kubelet` and `journalctl -u kubelet -n 50`. The kubelet handles node registration — if it crashed, the node goes `Unknown`. Restart it: `systemctl restart kubelet`. If this repeats, investigate kubelet logs for the root cause.
- C) Cordon and drain the node to clear the condition
- D) Reinstall the GPU Operator on the node

---

**Q13.** You have a GPU cluster with `infiniband_fabric = "fabric-5"` in `eu-west1`. You want to add more GPU nodes for a larger training run, but there's no capacity available on fabric-5. A colleague suggests adding nodes on `fabric-7` in `eu-north1`. What is the impact?

- A) Nodes on different fabrics can communicate via Ethernet — NCCL will automatically fall back
- B) InfiniBand fabrics are isolated networks. Nodes on `fabric-5` and `fabric-7` cannot communicate via InfiniBand directly. Additionally, these are in different regions, adding significant network latency. For a single training job requiring high-bandwidth inter-node communication, this won't work — you'd need a single fabric.
- C) Add the nodes to `fabric-7` and configure NCCL to use both fabrics
- D) The InfiniBand fabric is a soft preference — nodes can join any cluster

---

**Q14.** You notice a VM with a NRD (Non-Redundant Disk) has been running for 3 months hosting a shared training dataset. You need to add more storage to this VM. Current NRD size: 186 GiB. You need 100 GiB more. What is the new minimum NRD size you can provision?

- A) 286 GiB (186 + 100)
- B) NRD disks must be provisioned in multiples of 93 GiB. Current: 186 GiB (2 × 93). 186 + 100 = 286 GiB — but 286 is not a multiple of 93. Next multiple: 279 GiB (3 × 93). But 279 < 286, so you need 4 × 93 = 372 GiB. The minimum NRD to provide at least 286 GiB is **372 GiB**.
- C) 279 GiB (3 × 93 GiB)
- D) NRD disks have no size restrictions — provision exactly 286 GiB

---

**Q15.** A team wants to share training data between 3 projects using the Nebius shared filesystem. Their data is 2 PiB. They set up the filesystem in project A. Projects B and C can't access it. What is the constraint?

- A) The filesystem needs to be resized to 2 PiB — default max is 1 PiB
- B) Nebius shared filesystem is scoped to a single project. Projects B and C cannot access a filesystem from project A regardless of permissions. To share data across projects, use Object Storage (which supports cross-project access with the right IAM setup).
- C) Enable cross-project sharing in the filesystem settings
- D) The filesystem supports up to 3 projects natively — check the project IDs in the mount configuration

---

**Q16.** Your `kubectl get nodes` output shows a GPU node labeled `node.kubernetes.io/disk-pressure=true`. Training pods that were running on this node are evicted. What caused this and what do you fix?

- A) GPU VRAM is full — restart the node
- B) The node's local disk is filling up. Common causes: NCCL debug logs, pod logs, core dumps, or container image layers. SSH to the node and run `df -h` to identify what's consuming space. Clean up log files, remove unused Docker images with `crictl rmi`, or expand the node's boot disk.
- C) The InfiniBand network traffic is causing disk pressure
- D) NRD disk attached to the node is failing

---

**Q17.** A senior engineer says: "We should use `IO M3` disks for all our GPU VMs for maximum performance." You know that IO M3 disks offer 75,000 IOPS and 3-way replication. What is the one scenario where IO M3 is NOT the right choice?

- A) When you need more than 75,000 IOPS
- B) For temporary scratch space during training (e.g., storing intermediate data that's regenerated each run). NRD (Non-Redundant Disk) offers similar IOPS at lower cost and without the overhead of 3-way replication. IO M3's 3-way replication adds cost and is unnecessary for ephemeral data.
- C) When attaching to a VM with more than 8 GPUs
- D) When using Kubernetes — IO M3 is only for Slurm

---

**Q18.** You want to preemptively prepare for NVIDIA driver maintenance across a 32-node GPU cluster. The process must: not interrupt currently running training jobs, update drivers during a maintenance window. What is the correct sequence?

- A) Drain all 32 nodes simultaneously, update drivers, uncordon all nodes
- B) For each node during the maintenance window: (1) Cordon the node (prevents new pod scheduling), (2) Wait for running pods on that node to complete their current step and checkpoint, (3) Drain the node (evict remaining pods), (4) Update the driver preset in the node group (triggers node recreation), (5) Verify the new node is `Ready` and GPU Operator shows "Done", (6) Uncordon and move to next node. This rolling approach keeps other nodes running throughout.
- C) Update the driver preset on the node group — it handles everything automatically
- D) Reboot all nodes simultaneously during the maintenance window

---

**Q19.** Your `nebius mk8s cluster get --id $CLUSTER_ID` returns:

```json
{
  "status": {
    "state": "DEGRADED"
  }
}
```

What does this state indicate and what should you do?

- A) The cluster is being upgraded — wait 10 minutes
- B) `DEGRADED` means one or more control plane components are unhealthy but the cluster is partially functional. Check etcd health, API server responsiveness, and node status. If etcd lost quorum (e.g., one of 3 etcd instances failed), you may have a split-brain scenario. Contact Nebius support — control plane repairs typically require their intervention.
- C) A node group is at 0 capacity — add nodes
- D) The cluster's Kubernetes version is outdated — upgrade it

---

**Q20.** You need GPUs for inference that has strict latency requirements (<5ms P99). The inference load is bursty — high for 2 hours per day, minimal otherwise. You're choosing between dedicated reserved VMs vs. GPU allocation on-demand. What is the tradeoff and what do you recommend?

- A) Always use reserved VMs — on-demand has too much latency overhead
- B) Reserved VMs guarantee capacity and avoid cold-start delays but cost 24/7 regardless of usage. For strict <5ms latency, the warm-up time to start an on-demand VM (minutes) would violate SLAs during bursts. Use reserved VMs but scale the instance count to handle peak load — idle costs are the price of latency guarantees.
- C) On-demand is always cheaper for bursty workloads
- D) Use Serverless AI Endpoint — it handles bursty inference automatically

---

**Q21.** A developer creates a GPU VM using the `1gpu-16vcpu-200gb` preset and wants to run an NCCL all-reduce test against another VM on the same InfiniBand fabric. The test returns `0 GB/s` bandwidth. `ibstat` on both VMs shows Active. What is the first thing to check?

- A) NCCL version mismatch between VMs
- B) Single-GPU presets are not part of a GPU cluster — the VMs are not connected to InfiniBand even though the physical hardware has it. InfiniBand is only available to VMs that are part of a Nebius GPU cluster with an InfiniBand fabric assigned. Individual VMs on 1-GPU presets use Ethernet interconnect.
- C) The test needs `NCCL_IB_DISABLE=0` set explicitly
- D) The InfiniBand firmware needs to be updated

---

**Q22.** A `nebius mk8s cluster update` command is run to change the CIDR allowlist on the public endpoint. Immediately after, developers say they can't reach the Kubernetes API. `kubectl get nodes` times out. What happened?

- A) CIDR updates require 30 minutes to propagate
- B) If your developer machines are outside the new CIDR allowlist, the update locked them out. Run the update again from a machine that has API access (or via Nebius console which doesn't go through the Kubernetes API endpoint) to add the correct CIDRs. This is a self-lockout from an incorrect allowlist.
- C) The cluster is degraded due to the update — wait for it to stabilize
- D) CIDR updates force a cluster restart — wait for it to come back

---

## Domain 3: Running Training & Inference Workloads (Q23–Q30)

**Q23.** A 128-node distributed training run using PyTorch DDP fails 10 minutes in with:

```
RuntimeError: Expected to have finished reduction in the prior iteration before 
starting a new one. This error indicates that your module has parameters that 
were not used in producing loss.
```

What is the cause and fix?

- A) InfiniBand bandwidth is saturated — reduce the number of nodes
- B) PyTorch DDP requires all parameters participate in every forward pass. If your model has conditional branches where some parameters are skipped (e.g., optional layers), DDP's all-reduce synchronization breaks. Fix: set `find_unused_parameters=True` in `DistributedDataParallel()`, or better, remove unused parameters from the model.
- C) NCCL can't synchronize across 128 nodes — split into two 64-node jobs
- D) The WORLD_SIZE environment variable doesn't match the actual number of processes

---

**Q24.** You're running inference for a Vision Transformer model. The model was trained in FP32, and inference is also running in FP32. A colleague suggests switching to FP8 to double throughput. You're on H200 GPUs. What should you tell them?

- A) FP8 inference is only supported on B200 and B300 — H200 doesn't support it
- B) H100 and later architectures (including H200) support FP8 via hardware Transformer Engine. Switching inference to FP8 can significantly increase throughput with acceptable accuracy loss for most vision models. However, you'll need to quantize the model weights from FP32 to FP8 first using TensorRT or `torch.float8` — it's not an automatic switch.
- C) FP8 inference would reduce accuracy to unacceptable levels for vision tasks
- D) FP8 inference is identical to FP16 — the colleague is correct but it's already supported automatically

---

**Q25.** Your Slurm batch job script:

```bash
#!/bin/bash
#SBATCH --nodes=8
#SBATCH --ntasks-per-node=8
#SBATCH --gres=gpu:8
#SBATCH --time=72:00:00
#SBATCH --partition=gpu-h100

srun python train.py
```

The job fails immediately with `slurmstepd: error: execve(): python: No such file or directory`. `python3` is available. What is the fix?

- A) Change `srun python` to `srun /usr/bin/python`
- B) The `python` binary doesn't exist on the Slurm nodes — only `python3`. Either add `module load python3` before `srun`, use `srun python3 train.py`, or add `alias python=python3` in the script. The simplest fix is `srun python3 train.py`.
- C) Install Python on all Slurm nodes via apt
- D) The `srun` command doesn't pass environment variables — set `PYTHONPATH` first

---

**Q26.** A Serverless AI inference endpoint is deployed with a 7B LLaMA model. Requests spike from 10 req/s to 500 req/s during peak hours. P99 latency spikes from 200ms to 8000ms during peak. What is happening and what should you do?

- A) The model is too large for the hardware — switch to a smaller model
- B) The Serverless AI Endpoint is a single always-on instance — it doesn't autoscale. 500 req/s to a single 7B model GPU endpoint will saturate it. Either: (1) Deploy multiple instances with a load balancer in front, (2) Enable batching in the inference server to process requests together, (3) Move to a Kubernetes-based deployment with HPA for autoscaling.
- C) The endpoint needs a GPU with more VRAM
- D) Serverless AI Endpoints autoscale automatically — wait for capacity to provision

---

**Q27.** A Slurm job array with 1000 tasks (`#SBATCH --array=1-1000`) runs a hyperparameter sweep. Each task runs for 10–120 minutes depending on config. By hour 6, you check `squeue` and see 900 tasks as `PD` (pending) and only 8 tasks `R` (running) with `Reason: Resources`. How do you get more tasks running simultaneously?

- A) Increase the array size to 2000
- B) Only 8 GPUs are available and each task uses 1 GPU. With `--gres=gpu:1` and 8 available GPUs, only 8 tasks can run at once. Options: (1) Request more GPU nodes from the cluster admin, (2) Reduce GPU count per task if possible, (3) Wait for running tasks to complete.
- C) Set `--partition=gpu-preempt` to preempt lower-priority jobs
- D) Submit the remaining jobs as a separate job script

---

**Q28.** Your training job checkpoints every 500 steps to a shared filesystem. The checkpoint size is 200 GB. Training runs for 10,000 steps. You realize you've stored every checkpoint and the shared filesystem is now 4 TB of checkpoints. What is the correct strategy going forward?

- A) Increase shared filesystem size to 10 PiB
- B) Keep only the last N checkpoints on the shared filesystem (set `save_total_limit=3` in Trainer args or implement cleanup in your script). Immediately after saving a new checkpoint, copy it to Object Storage for long-term retention. Delete old checkpoints from the shared filesystem as they're successfully archived.
- C) Reduce checkpoint frequency to every 5000 steps
- D) Compress all checkpoints — this will reduce size by 80%

---

**Q29.** You need to run a training job that requires RDMA (Remote Direct Memory Access) for maximum inter-node bandwidth. You're using B200 GPUs in `us-central1-b`. Which operators must be installed in your Kubernetes cluster, and in what order?

- A) GPU Operator only — it includes RDMA support
- B) (1) NVIDIA GPU Operator first (provides CUDA drivers, device plugin) — from Nebius chart repo. (2) NVIDIA Network Operator second (provides InfiniBand/RDMA support, required for B200) — also from Nebius chart repo. The Network Operator depends on the GPU Operator being installed first. Both must come from the Nebius-specific OCI registry, not the generic NVIDIA charts.
- C) Network Operator only — B200 doesn't need the GPU Operator
- D) Install both simultaneously — they have no dependencies on each other

---

**Q30.** An MLflow experiment tracking server is set up in `eu-north1`. A data scientist in the Paris office reports their training jobs in `eu-west1` are failing to connect to MLflow. You check and the MLflow server is running and reachable from `eu-north1`. What is the real issue?

- A) Firewall rules are blocking cross-region MLflow traffic
- B) MLflow Managed Service is not available in `eu-west1`. The data scientist's training jobs in `eu-west1` cannot use a managed MLflow service there — it's a regional availability limitation. They need to either: (a) run training in a region where MLflow is available, (b) use a self-hosted MLflow VM in `eu-west1`, or (c) connect to the `eu-north1` MLflow server via its public endpoint.
- C) MLflow doesn't support cross-region connections
- D) The `eu-north1` MLflow server needs to be deployed in `eu-west1` as a replica

---

## Domain 4: Platform Automation & Maintenance (Q31–Q40)

**Q31.** Your CI/CD pipeline runs `terraform plan` on every pull request. A junior engineer adds this to the Nebius cluster resource:

```hcl
lifecycle {
  ignore_changes = [control_plane[0].version]
}
```

Why is this incorrect and what is the consequence?

- A) `lifecycle.ignore_changes` is not valid in Terraform
- B) Kubernetes version is immutable on Nebius — if the version in the Terraform config differs from what's deployed, `ignore_changes` silently accepts the drift. Your Terraform state will show the old version while the actual cluster has a different version. If you ever need to recreate the cluster from Terraform state, it will create the wrong version. Remove the ignore_changes and update the config to match the actual deployed version.
- C) This is correct practice for immutable fields — it prevents accidental recreation
- D) The version field path is wrong — should be `control_plane.version`

---

**Q32.** You run `nebius iam service-account list` and get:

```
Error: rpc error: code = PermissionDenied desc = permission denied
```

You're logged in to the CLI with your user account. What is the most likely cause?

- A) The IAM service is down
- B) Your user account is in the `viewers` group, which does not have permission to list service accounts (or the `auditors` group, which has view-only access but cannot access certain IAM operations). You need to be in the `admins` or `editors` group to manage IAM resources. Check your group membership with `nebius iam group list-members`.
- C) The CLI needs to be reinstalled
- D) Service account listing requires a special flag: `nebius iam service-account list --project-id $PROJECT_ID`

---

**Q33.** Your Terraform state file is stored in an Object Storage bucket. A colleague runs `terraform apply` at the same time as you, causing a state conflict. What prevents this, and if it's not set up, how do you implement it?

- A) Terraform has built-in conflict detection — it will error if two people run it simultaneously
- B) State locking. When using S3-compatible backends (including Nebius Object Storage), Terraform can use a DynamoDB table or compatible lock table for state locking. On Nebius, if you're using Object Storage as backend, configure locking via the `dynamodb_table` parameter (or an equivalent) to prevent concurrent applies. Without this, two simultaneous applies can corrupt the state file.
- C) Use separate state files for each team member
- D) Object Storage automatically prevents concurrent writes

---

**Q34.** A GPU node repeatedly fails with CUDA OOM errors, but `nvidia-smi` shows only 30% VRAM utilization right before the crash. What diagnostic step helps most?

- A) Reduce the batch size in all training jobs
- B) Run `nvidia-smi --query-compute-apps=pid,used_memory --format=csv` to see per-process GPU memory use. Also check for fragmentation: multiple small allocations can prevent a large contiguous allocation even with 70% free. `torch.cuda.memory_summary()` from within PyTorch shows fragmentation details. Consider calling `torch.cuda.empty_cache()` periodically.
- C) Reinstall the NVIDIA driver
- D) Increase the node's swap space

---

**Q35.** You want to track GPU utilization per namespace in your Kubernetes cluster and alert when any namespace's average GPU utilization drops below 50% for more than 30 minutes (indicating idle expensive GPUs). Using Nebius Observability, how do you implement this?

- A) Configure an alert in the Nebius console on the `gpu_utilization` metric
- B) (1) Configure GPU metrics collection via Nebius Monitoring (it provides dcgm_exporter-equivalent metrics). (2) In Grafana, create a query: `avg by (namespace) (DCGM_FI_DEV_GPU_UTIL) < 50`. (3) Set alert condition: value < 50 for 30 minutes. (4) Link to a Nebius alerting channel. Without Nebius Monitoring integration, deploy DCGM exporter yourself and scrape with your Prometheus.
- C) Use Slurm's built-in GPU utilization tracking
- D) Parse `nvidia-smi` output with a cron job

---

**Q36.** After a `terraform destroy` accidentally removed a production Kubernetes cluster, your team needs to rebuild from scratch in minimal time. The Terraform state was also destroyed. What is the disaster recovery procedure?

- A) Re-run `terraform apply` from your git-committed `.tf` files — Terraform will recreate everything
- B) (1) Restore the Terraform state from the Object Storage backend (which survived because it's in a separate project) using versioning or backup. (2) Run `terraform plan` to understand what Terraform thinks is missing vs what still exists in Nebius. (3) Use `terraform import` for any resources that still exist. (4) Apply to recreate missing resources. If state is unrecoverable, recreate all resources from git-committed configs (but this requires reimporting resources manually).
- C) Contact Nebius support to restore the destroyed resources
- D) The cluster can be recreated from the node group snapshots

---

**Q37.** Your team receives a PagerDuty alert at 3 AM: `NCCL AllReduce timeout on node-12`. Your on-call runbook says to check InfiniBand first. You SSH to node-12 and run `ibstat`. Output:

```
CA 'mlx5_0'
    Port 1:
        State: Down
        Physical state: Disabled
```

What is the next step?

- A) Restart NCCL on node-12
- B) The InfiniBand port on node-12 is physically disabled — this is hardware-level, not software. You cannot fix this by restarting NCCL or the driver. Immediately cordon and drain node-12 to move the training job off it. Open a Nebius support ticket with the node ID and IB port state. The node needs hardware attention.
- C) Run `ibportstate 0 1 enable` to re-enable the port
- D) Reboot node-12 to reset the IB port state

---

**Q38.** Your team wants to implement GitOps for GPU cluster configuration using ArgoCD. What Nebius API mechanism allows ArgoCD to apply Terraform-managed infrastructure changes?

- A) ArgoCD doesn't support Terraform — use Helm charts instead
- B) Use ArgoCD's App-of-Apps pattern with Terraform + Atlantis or Terraform Cloud as the execution layer. ArgoCD monitors the Git repo for changes to `.tf` files and triggers Atlantis/Terraform Cloud to run `terraform apply`. ArgoCD itself doesn't run Terraform — it's the event trigger. Alternatively, use the Nebius Terraform provider in a Kubernetes-native way with the Crossplane provider for Nebius.
- C) Nebius has a native ArgoCD integration — enable it in the console
- D) ArgoCD can apply Kubernetes manifests directly to manage Nebius resources via the Kubernetes API

---

**Q39.** You're debugging why a training pod in Kubernetes exits with code `137`. GPU utilization was at 95% right before the exit. What does exit code 137 mean and what is the likely cause?

- A) The training job completed successfully — 137 is the normal exit code
- B) Exit code 137 = container received SIGKILL (128 + 9). This is typically OOMKill at the container level (CPU RAM, not GPU VRAM). If GPU utilization was 95%, the training job was also consuming CPU RAM for data loading. Check `kubectl describe pod <pod-name>` for `OOMKilled: true` in the state. Fix: increase the pod's `resources.limits.memory`.
- C) Exit code 137 = GPU driver crash
- D) Exit code 137 = pod was preempted by a higher-priority job

---

**Q40.** You have a Terraform module that creates a GPU VM with a specific boot image. The image ID changes when Nebius releases a new base image with updated drivers. Your Terraform module hardcodes `image_id = "ami-abc123"`. When a new image is released, what is the production-safe way to update all 100 GPU VMs?

- A) Update `image_id` in the Terraform config and run `terraform apply` — it recreates all 100 VMs at once
- B) Update `image_id` in the Terraform module. Use `terraform apply -target` to update one VM at a time, prefacing each with `cordon → drain → apply → verify → uncordon`. Automate this with a wrapper script that loops through all 100 VMs, completing one before starting the next. This ensures training jobs are never interrupted as long as the cluster has more than 1 node.
- C) Update `image_id` using the Nebius console on each VM
- D) Use `terraform taint` to mark all 100 VMs for recreation, then apply with parallelism=100

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **C** | Nebius IAM groups are tenant-wide. There is no native project-level group scoping. Use service accounts scoped to specific projects and minimum permissions for fine-grained access control. |
| 2 | **B** | Never blindly delete an unknown SA — it may be actively used by production workloads. Always audit first (Audit Logs), then create a minimum-permission replacement, rotate in consuming systems, then clean up. |
| 3 | **B** | No versioning = permanent deletion. Nebius Object Storage does not retain deleted objects without versioning enabled. Enable versioning on all critical buckets. This is a real and common data loss scenario. |
| 4 | **B** | Nebius Network SSD disks are always encrypted by default using platform-managed keys. Customer-managed KMS keys (CMEK) are not applicable to Network SSD — this is a platform-managed encryption scenario. |
| 5 | **B** | An authorized key = Nebius API access at the SA's permission level. An `editors` SA can create/delete VMs, manage most resources, access all data. This is why least-privilege matters — an authorized key with admin access is devastating. |
| 6 | **B** | MysteryBox is project-scoped. Cross-project secret sharing requires duplication or cross-project service account authentication. Plan your secret architecture around project boundaries. |
| 7 | **B** | Day 8 at 80% = 10% per day burn rate = budget exhaustion by day 10. Identify the root cause immediately (billing reports) and take action. This is a critical signal of a runaway workload. |
| 8 | **B** | Audit Logs capture all IAM control-plane changes. Filter by IAM service and group membership events. This is the authoritative source for compliance evidence of who changed what permissions when. |
| 9 | **B** | Multiple InfiniBand fabrics exist in eu-north1 (fabric-2/3/4/6). 64 × 8 = 512 GPUs. Whether a single fabric can support 512 GPUs depends on its capacity — reserve ahead of time for large runs. |
| 10 | **B** | P-Key (partition key) isolates GPU clusters on the same physical fabric. After a network maintenance window, P-Key assignments can be reset by the switch, causing communication failure even though ibstat shows Active ports. |
| 11 | **B** | Two issues: (1) `etcd_cluster_size=1` = no HA, production needs 3. (2) Kubernetes version is immutable on Nebius — plan your version choice carefully since you can't change it without recreating the cluster. |
| 12 | **B** | `Ready: Unknown` + missing kubelet pod = the kubelet process on the node is down. SSH and restart kubelet. If it keeps crashing, check kubelet logs for the real cause. |
| 13 | **B** | InfiniBand fabrics are isolated networks in specific regions. Cross-fabric, cross-region communication via InfiniBand is not possible. Nodes must be on the same fabric for high-bandwidth training. |
| 14 | **B** | NRD must be multiples of 93 GiB. 286 / 93 = 3.075, so round up to 4. 4 × 93 = 372 GiB is the minimum NRD size that accommodates the 286 GiB requirement. |
| 15 | **B** | Shared filesystem is single-project only. For cross-project data sharing, Object Storage is the correct tool — it supports cross-project access with proper IAM configuration. |
| 16 | **B** | `disk-pressure=true` is Kubernetes reporting the local disk is near full. SSH and use `df -h` to find what's consuming space. Common culprits: log files, container image layers, NCCL dump files. |
| 17 | **B** | IO M3's 3-way replication adds cost that's unnecessary for ephemeral/temporary scratch data. NRD (no replication) is cheaper for data you'll regenerate anyway. Match durability to data's value. |
| 18 | **B** | Rolling node updates are the production-safe approach. Cordon prevents new scheduling, checkpoint ensures no progress is lost, drain moves existing pods, then proceed with the update one node at a time. |
| 19 | **B** | `DEGRADED` indicates partial cluster health. The etcd quorum is most critical — with 3 etcd instances, losing 1 means quorum is maintained (2/3) but losing 2 means quorum loss and the cluster stops accepting writes. |
| 20 | **B** | Strict latency (<5ms) + burst pattern = reserved VMs are required. Cold-start time for on-demand VMs (minutes) violates the SLA. Accept the idle cost as the price of availability guarantee. |
| 21 | **B** | InfiniBand in Nebius is only available within GPU clusters (created via `nebius gpu-cluster` with fabric specified). Individual VMs, even on hardware with IB ports, use Ethernet for interconnect outside a cluster context. |
| 22 | **B** | A CIDR allowlist update that doesn't include your machines' IPs locks you out of the Kubernetes API. Recovery requires accessing the cluster via another method (Nebius console, bastion host within allowed CIDRs) to fix the allowlist. |
| 23 | **B** | PyTorch DDP error about unused parameters is a well-known issue with models that have conditional branches. Setting `find_unused_parameters=True` resolves it, though it adds overhead. Removing unused params is cleaner. |
| 24 | **B** | FP8 is supported on H100+ (Hopper architecture and later). H200 is an H100-class chip with more memory. FP8 requires model quantization — it's not automatic. The throughput gain is real and significant. |
| 25 | **B** | Unix exit code 127 = command not found. `python` ≠ `python3` on most modern Linux systems. Use `python3` or add a module load command. This is one of the most common Slurm job failures. |
| 26 | **B** | Serverless AI Endpoints do not autoscale. One endpoint = one inference backend. For high req/s, you need multiple instances or a proper autoscaling deployment on Kubernetes with HPA. |
| 27 | **B** | `Reason: Resources` = not enough GPUs in the cluster to run more tasks. Only 8 GPUs available, 8 tasks running = no capacity. Adding more GPU nodes is the only real fix for increasing parallelism. |
| 28 | **B** | Shared filesystem is for active training, not long-term storage. Keep 2-3 rolling checkpoints on shared filesystem, archive to Object Storage after each. Implement `save_total_limit` in your training code. |
| 29 | **B** | B200 requires both GPU Operator AND Network Operator. Install GPU Operator first, then Network Operator. Both must come from Nebius's OCI chart registry — generic NVIDIA charts don't work on Nebius infrastructure. |
| 30 | **B** | MLflow Managed Service is regionally limited — not available in `eu-west1`. This is a documented platform constraint, not a network/firewall issue. The solution is self-hosted MLflow or cross-region connection. |
| 31 | **B** | `ignore_changes` on an immutable field creates silent drift. Your Terraform state diverges from reality. If the cluster ever needs to be recreated from Terraform, it'll use the wrong version. Remove the ignore and document the immutability in comments instead. |
| 32 | **B** | `viewers` and `auditors` groups have limited IAM management permissions. IAM resource listing typically requires `editors` or `admins` access. Verify your group membership and escalate if needed. |
| 33 | **B** | S3 backends support state locking via an external lock table. Without it, concurrent `terraform apply` runs can interleave state writes and corrupt the state file. Always configure locking for shared infrastructure. |
| 34 | **B** | Low overall VRAM usage with OOM crashes indicates memory fragmentation or a single process spike. `nvidia-smi --query-compute-apps` reveals per-process usage. PyTorch's memory summary reveals fragmentation details. |
| 35 | **B** | Nebius Monitoring provides GPU metrics via its Prometheus integration. Create a Grafana alert on the per-namespace average — this identifies idle expensive GPUs so you can intervene before paying for wasted compute. |
| 36 | **B** | State recovery is the first priority. Object Storage versioning (if enabled) lets you restore previous state versions. Then use `terraform import` for surviving resources before recreating missing ones. |
| 37 | **B** | `State: Down` + `Physical state: Disabled` is hardware-level IB port failure. Software restarts won't fix it. Cordon/drain the node to maintain training continuity, then escalate to Nebius support for hardware repair. |
| 38 | **B** | ArgoCD + Terraform requires an execution layer (Atlantis, Terraform Cloud, or custom). ArgoCD is the trigger/monitor, not the executor for Terraform. Alternatively, Crossplane provides Kubernetes-native infrastructure management. |
| 39 | **B** | Exit code 137 = SIGKILL = OOMKill. CPU RAM exhaustion during data loading is common when GPU is fully utilized (GPU handles compute, CPU handles data pipeline). Increase `resources.limits.memory` in the pod spec. |
| 40 | **B** | Production-safe image updates require rolling replacement with cordon/drain per VM. Scripted `terraform apply -target` loops ensure training continuity. Never apply all 100 at once — that's simultaneous downtime across the entire cluster. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Exam ready |
| 30–35 | Close — focus on weak domains |
| 24–29 | More practice needed |
| <24 | Re-read notes with focus on applying knowledge |
