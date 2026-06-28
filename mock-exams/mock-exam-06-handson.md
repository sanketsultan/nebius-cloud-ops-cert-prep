# Nebius AI Cloud Ops — Mock Exam 6 (Scenario-Based, Hard)

**Format:** Command outputs, error messages, and real ops scenarios. Every question requires diagnosis and action.
**Time:** ~55 minutes | **Pass mark:** 70% (28/40)

> Attempt all questions before checking the answer key at the bottom.

---

## Domain 1: Security, Compliance & Billing (Q1–Q8)

**Q1.** You run the following and get:

```bash
$ nebius iam whoami
{
  "subject": {
    "type": "SERVICE_ACCOUNT",
    "id": "sa-xyz-9921"
  },
  "membership": [
    {"group": "auditors"}
  ]
}
```

You then run:

```bash
$ nebius compute instance list
Error: rpc error: code = PermissionDenied desc = access denied to resource
```

What is happening and what is the fix?

- A) The CLI is misconfigured — reinstall it
- B) You're authenticated as a service account in the `auditors` group. `auditors` have view-only access to control plane logs but cannot list compute resources. To list instances, you need to be in the `viewers` or `editors` group. Either switch to a higher-privilege profile or request that this SA be added to the `viewers` group.
- C) `nebius compute instance list` requires a `--project-id` flag
- D) The service account needs a new access key — the current one expired

---

**Q2.** A security team member sends you this Audit Log entry:

```json
{
  "eventType": "DeleteObject",
  "requestTime": "2026-06-15T03:47:22Z",
  "source": {
    "type": "SERVICE_ACCOUNT",
    "id": "sa-abc-0088",
    "name": "training-pipeline-sa"
  },
  "resourcePath": "s3://prod-models/gpt4-finetune-v3/weights.bin",
  "status": "OK"
}
```

The team says this model weights file was deleted without authorization. What is your immediate response?

- A) The SA's access key must be rotated — it was compromised
- B) Immediately revoke the SA's access by: (1) Deleting all access keys for `sa-abc-0088`, (2) Removing it from all IAM groups, (3) Checking all other actions this SA performed in the same time window (filter Audit Logs for `sa-abc-0088` in the past 24 hours to see what else was done). Verify whether the model weights exist elsewhere (other bucket, other version). Escalate as security incident.
- C) Restore the file from Object Storage backup
- D) The SA created the file, so it has permission to delete it — this is not an incident

---

**Q3.** Your billing dashboard shows:

```
This month's spend:
  compute.instance (GPU): $12,400
  storage.object (egress): $89,000
  storage.disk: $1,200
  
Budget: $15,000
Alert threshold: 80% ($12,000)
```

The alert fired. You check Object Storage egress. What is the first diagnostic question?

- A) Which bucket is generating the egress?
- B) Is the egress to the public internet (expensive) or to same-region Nebius services?
- C) Both: which bucket AND where the egress is going. Check billing breakdown by resource (which bucket) and by destination (egress type). $89K of Object Storage egress typically indicates a misconfiguration — e.g., training data being downloaded from the internet on every job instead of read from a local shared filesystem.
- D) Whether Object Storage pricing changed this month

---

**Q4.** You need to create a MysteryBox secret for a database password. The current password is `Tr@inDB#2024`. In 30 days, it will rotate to `Tr@inDB#2025`. How do you set this up so rotation is seamless?

- A) Create one secret, update the value when the password changes
- B) Create the secret with the current value as version 1. When the password changes: (1) Add version 2 with `Tr@inDB#2025`, (2) Set version 2 as primary. Applications that fetch the latest primary version automatically use the new password. Applications that cached the old value continue working until their next refresh. No downtime required.
- C) Create two separate secrets — one for current, one for next password
- D) Store the password rotation schedule in MysteryBox metadata

---

**Q5.** You're reviewing a Terraform plan that will apply changes to your Nebius IAM configuration. You see:

```
  # nebius_iam_group_membership.ml_team_editor will be created
  + resource "nebius_iam_group_membership" "ml_team_editor" {
      + group_id  = "grp-editors"
      + member_id = "sa-pipeline-prod"
    }
```

The plan was submitted by a junior engineer as part of a "CI/CD automation improvement." What is the concern?

- A) No concern — adding a SA to editors is routine
- B) Adding `sa-pipeline-prod` to the `editors` group gives that SA tenant-wide resource management capability. If `sa-pipeline-prod` is used by a CI/CD pipeline, any pipeline vulnerability could be exploited to modify production infrastructure. Verify: what does this SA actually need? Can a more specific permission satisfy the requirement? Reject and request minimum-privilege scoping.
- C) The group ID should be the full group name, not `grp-editors`
- D) Service accounts cannot be added to the editors group — only humans

---

**Q6.** The following KMS key exists in your Nebius project:

```
ID: kms-key-0029
Name: model-weights-enc
Algorithm: AES-256
Status: Active
Created: 2025-01-10
Rotation: Every 90 days
Last rotated: 2026-03-01
```

Today is June 28, 2026. What action is needed?

- A) No action — automatic rotation is configured
- B) Last rotation: March 1, 2026. +90 days = May 30, 2026. Today is June 28 — the key should have rotated 29 days ago. Either automatic rotation is silently failing, or there's a bug in the rotation policy. Check rotation status and manually trigger rotation if needed. Set up an alert for missed KMS rotations.
- C) Rotate the key immediately — 90 days is too long for a production key
- D) The key is current — next rotation is September 1

---

**Q7.** You export Audit Logs to an Object Storage bucket for a 6-month retention policy. After 6 months, the bucket now contains 2.3 TB of logs. Storage costs are significant. What is the right approach?

- A) Delete all logs older than 6 months manually each month
- B) Configure an Object Storage lifecycle policy that automatically deletes objects older than 6 months. Set it once — it runs automatically. Also evaluate whether 6 months of retention is a compliance requirement or just a habit.
- C) Compress the logs — they should compress to 10% of their current size
- D) Stop exporting Audit Logs — the export wasn't providing value

---

**Q8.** A new Nebius project needs to be created for a model training team. Their requirements: (a) training engineers can deploy VMs and manage resources, (b) data scientists can read/write training data but not provision infrastructure, (c) ML ops can see everything but change nothing. What is the correct mapping to Nebius default IAM groups?

- A) (a) admins, (b) editors, (c) viewers
- B) (a) editors, (b) viewers (with bucket-level write policy for data), (c) auditors — read-only control plane access, no data access
- C) (a) admins, (b) viewers, (c) auditors
- D) (a) editors, (b) editors, (c) viewers

---

## Domain 2: Setting Up & Operating GPU Clusters (Q9–Q22)

**Q9.** You run `kubectl get nodes` on a fresh Nebius Managed Kubernetes cluster with 4 H100 GPU nodes:

```
NAME           STATUS   ROLES    AGE   VERSION
gpu-node-01    Ready    <none>   5m    v1.33.0
gpu-node-02    Ready    <none>   5m    v1.33.0
gpu-node-03    Ready    <none>   5m    v1.33.0
gpu-node-04    Ready    <none>   5m    v1.33.0
```

You then run `kubectl describe node gpu-node-01 | grep nvidia` and get no output. What does this mean and what is the next step?

- A) H100 nodes don't expose GPU information via kubectl
- B) No `nvidia.com/gpu` in node allocatable resources means the GPU Operator is not installed. The nodes are up but GPUs are not registered with Kubernetes. Install NVIDIA GPU Operator from Nebius's chart repo: `helm install gpu-operator oci://cr.eu-north1.nebius.cloud/marketplace/nebius/nvidia-gpu-operator/chart/gpu-operator -n gpu-operator --create-namespace --wait`.
- C) Run `kubectl label node gpu-node-01 nvidia.com/gpu=8` to register the GPUs
- D) The GPU Operator is installing — wait 30 minutes for it to complete

---

**Q10.** You run an NCCL bandwidth test on your 8-node H100 cluster and get:

```
# nccl-tests allreduce
[  WARN  ] Rank 0: NCCL_IB_DISABLE not set, defaulting to IB disabled
busbw: 28.5 GB/s
```

Expected bandwidth for H100 with InfiniBand is ~340 GB/s. What is the misconfiguration?

- A) The NCCL version is outdated — upgrade NCCL
- B) The warning says InfiniBand is disabled because `NCCL_IB_DISABLE` is not set — NCCL defaults to disabling IB when the variable is absent. Set `NCCL_IB_DISABLE=0` in the training job's environment. The 28.5 GB/s is consistent with Ethernet-only throughput.
- C) The InfiniBand fabric is not configured — add `infiniband_fabric` to the GPU cluster
- D) `nccl-tests` doesn't support InfiniBand — use a different benchmark

---

**Q11.** Your Terraform config for a Nebius node group looks like:

```hcl
resource "nebius_mk8s_v1_node_group" "gpu_workers" {
  parent_id = nebius_mk8s_v1_cluster.main.id
  
  template {
    resources {
      platform = "gpu-h100-sxm"
      preset   = "8gpu-128vcpu-1600gb"
    }
    boot_disk {
      type       = "NETWORK_SSD"
      size_bytes = 107374182400  # 100 GiB
    }
  }
  
  fixed_node_count = 8
  
  gpu_cluster {
    id = nebius_gpu_cluster.ib_cluster.id
  }
}
```

A training engineer says the nodes provisioned but GPU jobs fail with `permission denied: /dev/nvidia*`. What is missing?

- A) The node group needs `nvidia_driver_version` specified
- B) The GPU Operator (and nvidia-container-toolkit) is not installed in the cluster. Without it, containers cannot access `/dev/nvidia*` devices even though the hardware exists. The node group provision just creates VMs — the GPU Operator must be separately installed via Helm to configure device plugins, drivers, and container runtime hooks.
- C) Add `security_context: privileged: true` to all GPU pods
- D) The `platform = "gpu-h100-sxm"` field is invalid — use `platform = "h100"`

---

**Q12.** You check a training node and see:

```bash
$ nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 570.133.07     Driver Version: 570.133.07    CUDA Version: 12.8 |
+-------------------------------+----------------------+----------------------+
|   0  H100 SXM5 80GB  On       | 00000000:03:00.0 Off |                    0 |
| N/A  34C    P0   108W /  700W |      0MiB /  81920MiB |      0%      Default |
```

What CUDA and driver version is this node running, and which Nebius driver preset was used?

- A) CUDA 12.8, Driver 570.x — this is the `cuda12.8` preset
- B) CUDA 13.0, Driver 570.x — this is the `cuda13.0` preset
- C) CUDA 12.8, Driver 580.x — this is the `cuda12.8` preset
- D) Driver 570.x corresponds to the `cuda13.0` preset

---

**Q13.** You want to create an H200 GPU cluster in `eu-west1`. You run:

```bash
nebius gpu-cluster create \
  --name h200-cluster \
  --region eu-west1 \
  --infiniband-fabric fabric-7
```

The command fails with `Error: infiniband-fabric 'fabric-7' is not available in region 'eu-west1'`. What is the correct fabric?

- A) `fabric-6` — for all EU regions
- B) `fabric-5` is the InfiniBand fabric for H200 GPUs in `eu-west1`. `fabric-7` is for H200 in `eu-north1`. Each region has its own fabric assignments.
- C) `fabric-4` — for H200 everywhere
- D) H200 in eu-west1 doesn't support InfiniBand

---

**Q14.** During a training run, you see this on node `gpu-node-07`:

```bash
$ ibstat
CA 'mlx5_0'
  Port 1:
    State: Active
    Physical state: LinkUp
    Rate: 200
    
CA 'mlx5_1' 
  Port 1:
    State: Active
    Physical state: LinkUp
    Rate: 200
```

But NCCL all-reduce is only using one HCA. What environment variable do you set to use both?

- A) `NCCL_HCA_COUNT=2`
- B) Set `NCCL_IB_HCA=mlx5_0,mlx5_1` to tell NCCL to use both HCAs. With two 200 Gbps ports, this doubles available InfiniBand bandwidth for all-reduce operations.
- C) Set `NCCL_NET=IB2` to enable dual-HCA mode
- D) NCCL automatically uses all available HCAs — no configuration needed

---

**Q15.** You run `kubectl describe pod training-job-0` and see:

```
Events:
  Warning  Failed   2m   kubelet  Error: failed to create containerd task: 
           failed to create shim task: OCI runtime create failed: 
           container_linux.go:349: starting container process caused: 
           rootfs_linux.go:76: mounting /dev/nvidia-uvm on /dev/nvidia-uvm: 
           no such file or directory
```

What is the root cause?

- A) The pod's security context doesn't allow device mounting
- B) `/dev/nvidia-uvm` doesn't exist on the node — the NVIDIA UVM kernel module is not loaded. This happens when the GPU Operator's driver daemonset failed to install the driver on this node. Check: `kubectl get pods -n gpu-operator` to see if the driver pod on this node is healthy.
- C) The container image doesn't include CUDA libraries
- D) The node doesn't have a GPU — wrong node group was selected

---

**Q16.** A Slurm cluster administrator shows you this:

```bash
$ sinfo
PARTITION  AVAIL  TIMELIMIT  NODES  STATE    NODELIST
gpu-h100      up   infinite      2    alloc   gpu-[01-02]
gpu-h100      up   infinite      6     idle   gpu-[03-08]
gpu-h100      up   infinite      2     down   gpu-[09-10]
```

A researcher wants to run a job requiring 8 nodes with 8 GPUs each (64 GPUs total). Is this possible now?

- A) Yes — 2 alloc + 6 idle = 8 nodes available
- B) No. `alloc` nodes are occupied by other jobs. `down` nodes are unavailable. Only 6 `idle` nodes are available right now. An 8-node job requires waiting for at least 2 more nodes to become free (either `gpu-01` or `gpu-02` finish, or `gpu-09/10` come back up).
- C) Yes — submit with `--begin=now+1h` and the scheduler will preempt alloc nodes
- D) Yes — use `--nodes=6` and request 10 GPUs per node to compensate

---

**Q17.** You check node provisioning status after creating a node group and see:

```bash
$ nebius mk8s node-group get --id $NG_ID
{
  "status": {
    "state": "PROVISIONING",
    "nodes": [
      {"id": "node-001", "state": "RUNNING"},
      {"id": "node-002", "state": "RUNNING"},
      {"id": "node-003", "state": "PROVISIONING"},
      {"id": "node-004", "state": "PROVISIONING"}
    ]
  }
}
```

After 30 minutes, nodes 3 and 4 are still in `PROVISIONING`. What do you do?

- A) Delete and recreate the node group — provisioning is stuck
- B) Check: (1) quota limits — you may not have enough GPU VM quota for additional instances, (2) zone capacity — the requested GPU type may not have enough capacity right now, (3) `nebius compute instance list` to see if the underlying VMs were created. If quota or capacity is the issue, contact Nebius support. Don't delete the entire node group — the two running nodes are healthy.
- C) Wait — GPU provisioning can take up to 2 hours
- D) Reboot nodes 3 and 4 via the Nebius console

---

**Q18.** You see this in the GPU Operator daemonset logs on a newly joined node:

```
[INFO]  Loading kernel module: nvidia
[INFO]  Loading kernel module: nvidia_uvm
[INFO]  Loading kernel module: nvidia_modeset
[ERROR] Failed to load kernel module nvidia_peermem: module not found
[INFO]  Done, now waiting for signal
```

Should you be concerned about the `nvidia_peermem` error?

- A) Yes — nvidia_peermem is required for all GPU operations
- B) `nvidia_peermem` enables GPUDirect RDMA (direct GPU-to-NIC memory transfer). If your workload doesn't require GPUDirect RDMA (e.g., using Ethernet-based NCCL), this warning is safe to ignore. For InfiniBand-based distributed training that needs max bandwidth, install the Network Operator — it provides peermem support.
- C) Ignore it always — peermem is automatically retried on demand
- D) Reinstall the GPU Operator with `--set driver.rdmaHostMofed=true`

---

**Q19.** You run `kubectl top nodes` and see:

```
NAME          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
gpu-node-01   15800m       99%    620Gi           97%
gpu-node-02   210m         1%     12Gi            2%
```

All 8 training pods are on gpu-node-01. gpu-node-02 is empty. What is most likely wrong?

- A) Node-02 has less memory — pods require high memory so they all went to node-01
- B) Check if gpu-node-02 has a `NoSchedule` taint, or if the training pods have a `nodeSelector`/`nodeAffinity` rule pinning them to node-01. Run `kubectl describe pod <training-pod>` to see scheduling constraints and events. All 8 pods on 1 node when 2 nodes are available is a scheduling misconfiguration.
- C) Kubernetes scheduler is broken — restart kube-scheduler
- D) This is normal — Kubernetes bin-packs pods onto the fewest nodes

---

**Q20.** A network engineer asks you: "Which InfiniBand fabric connects to a B200 GPU cluster in `us-central1`?" What is the answer?

- A) `fabric-5` — it's the universal B-series fabric
- B) `us-central1-b` is the InfiniBand fabric designation for B200 GPUs in `us-central1`. `us-central1-a` is for H200 GPUs. Nebius uses zone names as fabric identifiers in the `us-central1` region, while `eu-north1` uses `fabric-N` naming.
- C) `fabric-3` — all B200 GPUs use fabric-3
- D) B200 in us-central1 doesn't support InfiniBand

---

**Q21.** After running `nebius mk8s cluster create`, you need to configure `kubectl` to connect to it. What is the correct command?

- A) `kubectl config set-context $(nebius mk8s cluster get-credentials --id $CLUSTER_ID)`
- B) Run `nebius mk8s cluster get-credentials --id $CLUSTER_ID` — this downloads the kubeconfig and merges it into your `~/.kube/config`. Then verify with `kubectl get nodes`.
- C) Download the kubeconfig from the Nebius console and manually place it at `~/.kube/config`
- D) `kubectl config set-cluster nebius --server=$(nebius mk8s cluster get --id $CLUSTER_ID | jq -r '.endpoint')`

---

**Q22.** You receive this from `kubectl describe node gpu-node-04`:

```
Conditions:
  Type             Status  Reason
  MemoryPressure   False   KubeletHasSufficientMemory
  DiskPressure     True    KubeletHasDiskPressure
  PIDPressure      False   KubeletHasSufficientPID
  Ready            True    KubeletReady
```

The node shows `Ready` but also `DiskPressure`. What is the situation and what do you do?

- A) The node is fine — DiskPressure and Ready can coexist without issues
- B) `DiskPressure=True` means disk is nearly full. The node is still `Ready` but Kubernetes will soon start evicting pods and blocking new scheduling. Immediate action: SSH to node, run `df -h`, find what's consuming space (log files, container layers, core dumps, NCCL debug logs), and clean up before Kubernetes takes automated actions.
- C) Restart kubelet to clear the DiskPressure condition
- D) The boot disk needs to be expanded — contact Nebius support

---

## Domain 3: Running Training & Inference Workloads (Q23–Q30)

**Q23.** Your training script uses this checkpoint save logic:

```python
if step % 500 == 0:
    torch.save({
        'step': step,
        'model': model.state_dict(),
        'optimizer': optimizer.state_dict(),
    }, f'/shared-fs/checkpoint-{step}.pt')
```

On step 3000, the shared filesystem is full and `torch.save` throws `OSError: No space left on device`. The training job crashes. What is the immediate fix and long-term solution?

- A) Increase shared filesystem size to 100 PiB
- B) Immediate: SSH to a node, delete old checkpoints (keep checkpoint-2500.pt and checkpoint-3000.pt). Long-term: implement rolling checkpoint cleanup in the training code — after each save succeeds, delete checkpoints older than N steps. Add error handling around `torch.save` to log failures without crashing the job.
- C) Change the checkpoint path to Object Storage using `s3://` prefix
- D) Reduce checkpoint frequency to every 2000 steps

---

**Q24.** You're looking at this Slurm sacct output:

```bash
$ sacct -j 7042 --format=JobID,JobName,State,ExitCode,ElapsedRaw
JobID     JobName    State       ExitCode ElapsedRaw
-------   -------    -----       -------- ----------
7042      train_v2   FAILED      1:0      14782
7042.0    batch      COMPLETED   0:0      14782
7042.1    extern     COMPLETED   0:0      14782
```

The parent job failed (exit code 1:0) but the batch step completed successfully. What does exit code `1:0` mean?

- A) There's a Slurm bug — if batch completed, the job should show COMPLETED
- B) Exit code `1:0` = Unix exit code 1 (non-zero = error, `:0` = no signal). The training script inside the job returned exit code 1. The Slurm `batch` step (the shell wrapper) ran and completed, but the Python training script inside it returned an error. Check the job's stderr output for the actual Python exception.
- C) The job ran for 14,782 seconds and was cancelled
- D) Exit code 1:0 means it was preempted by a higher-priority job

---

**Q25.** Your Serverless AI Job for batch inference runs for 4 hours and shows status `SUCCEEDED`. But the output bucket is empty. What do you check first?

- A) The job failed silently — the SUCCEEDED status is wrong
- B) `SUCCEEDED` means the process exited 0. Empty output bucket despite success indicates: (1) The script may be writing to the wrong path, (2) The SA running the job may lack write access to the output bucket, (3) The script uses AWS S3 endpoints instead of Nebius S3-compatible endpoints. Check job logs for any write errors and verify SA permissions on the output bucket.
- C) The inference output is still uploading — wait 30 minutes
- D) The Nebius Object Storage bucket needs to be in the same region as the job

---

**Q26.** You're debugging a training job. The log shows:

```
[rank 0] Step 100: loss=2.341
[rank 0] Step 200: loss=2.339
...
[rank 0] Step 800: loss=2.342
NCCL WARN Timeout, rank 0 waiting for ranks: 12 13 14 15
NCCL WARN Connect timeout reached
RuntimeError: NCCL timeout after 1800s
```

Ranks 12-15 are on nodes `gpu-node-13` through `gpu-node-16`. What do you check on those specific nodes?

- A) NCCL timeout means the network is overloaded — reduce the batch size
- B) Ranks 12-15 stopped responding. On nodes 13-16: (1) `ibstat` — check IB port state (look for `Disabled` or `Down`), (2) `nvidia-smi` — check for GPU errors or thermal throttling, (3) `dmesg | grep -i nvidia` — check for Xid errors, (4) Check if pods are still running: `kubectl get pods -n training -o wide | grep gpu-node-1[3-6]`. One node losing IB connectivity causes its ranks to hang until timeout.
- C) Increase `NCCL_TIMEOUT` to 3600s to avoid this error
- D) Restart NCCL on ranks 12-15

---

**Q27.** You run this Slurm job on a 4-node H200 cluster (8 GPUs per node):

```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=8
#SBATCH --gres=gpu:8
export WORLD_SIZE=$((SLURM_NNODES * 8))
export RANK=$SLURM_PROCID
srun python3 train.py
```

What is the value of `WORLD_SIZE` at runtime?

- A) 4 (number of nodes)
- B) `WORLD_SIZE = SLURM_NNODES × 8 = 4 × 8 = 32`. Each node has 8 GPUs, each runs 1 process per GPU, total 32 distributed processes.
- C) 8 (GPUs per node)
- D) 1 (single job ID)

---

**Q28.** You run the following:

```bash
$ nebius ai serverless-job get --id job-abc-1234 --format json | jq '.status'
{
  "state": "RUNNING",
  "startedAt": "2026-06-27T18:00:00Z",
  "progress": {
    "gpu_utilization": 0,
    "elapsed_seconds": 86400
  }
}
```

Today is June 28, 2026. GPU utilization is 0% after 24 hours. What do you do?

- A) Wait — large model training can take multiple days with low GPU utilization
- B) Zero GPU utilization for 24 hours = the job is stuck. Either a rank failed silently and others are waiting, or there's a deadlock. Check logs: `nebius ai serverless-job logs --id job-abc-1234`. If logs show no progress, cancel the job to stop billing, debug, and resubmit.
- C) The job is preprocessing data — GPU utilization is always 0% during this phase
- D) Restart the job without cancelling first

---

**Q29.** A researcher asks you to compare B200 vs H100 for a 2-week FP16 transformer training run. The model is 70B parameters. Which do you recommend and why?

- A) H100 — it's battle-tested and has more software support
- B) B200 is the better choice: (1) 180 GB HBM3e vs H100 80 GB — a 70B FP16 model (~140 GB) fits in a single B200 VM (8 × 180 GB = 1,440 GB) without tensor parallelism, while on H100 (8 × 80 GB = 640 GB) you still fit it but with less headroom for activations, (2) Better memory bandwidth per GPU for memory-bound transformer workloads, (3) FP4 support for future quantization. If the model fits in H100 VRAM comfortably, H100 is a valid choice for its mature ecosystem.
- C) H100 — B200 doesn't support FP16 training
- D) They're equivalent for FP16 training — choose based on price per hour

---

**Q30.** Your inference service shows these metrics:

```
GPU utilization: 92%
GPU memory: 78% (112 GB / 141 GB)
CPU utilization: 8%
Request queue depth: 0
P99 latency: 45ms
```

Your manager wants you to "optimize the inference endpoint." What do you tell them?

- A) Reduce GPU count — the endpoint is overprovisioned
- B) The metrics look healthy: high GPU utilization (efficient), reasonable memory headroom, empty queue (not overloaded), 45ms P99. Optimization without a defined problem risks instability. Ask: what problem are we solving? If lower cost at same latency, explore FP8 quantization. If lower latency, try speculative decoding. If higher throughput, try continuous batching. Don't optimize a working system without a specific target.
- C) Enable autoscaling to add more GPU capacity
- D) Switch to a B300 GPU for better inference performance

---

## Domain 4: Platform Automation & Maintenance (Q31–Q40)

**Q31.** You run `terraform plan` and see:

```
  ~ resource "nebius_mk8s_v1_cluster" "prod" {
      ~ control_plane {
          ~ etcd_cluster_size = 3 -> 1   # (forces replacement)
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.
```

A colleague says "it's fine — we're just reducing etcd replicas to save cost." What do you do?

- A) Apply it — 1 etcd is sufficient for a GPU training cluster
- B) Block this plan. (1) Reducing etcd from 3 to 1 eliminates HA — one etcd failure = cluster loss. (2) This forces replacement = destroy the production cluster and create a new one. All running training jobs will be lost. (3) The cost savings are negligible compared to the risk. Reject the change, explain the consequences, require proper review.
- C) Apply it during off-hours when no training jobs are running
- D) The change will apply in-place — etcd can be scaled down safely

---

**Q32.** You run `terraform apply` and see:

```
Error: Error creating node group: rpc error: code = ResourceExhausted 
desc = quota exceeded: GPU instance quota for project 
'proj-ml-train': requested 8, available 6
```

What do you do?

- A) Split the node group into two groups of 4 nodes each
- B) Your project's GPU instance quota is 6 but you're requesting 8. Options: (1) Request a quota increase via Nebius console → Project settings → Quotas (takes a few business days), (2) Temporarily use `fixed_node_count = 6` to stay within current quota. Don't delete existing VMs to free quota — that disrupts running workloads.
- C) Use a different GPU type — H100 quota may be higher
- D) Use `-parallelism=1` flag to create nodes one at a time

---

**Q33.** Your monitoring dashboard shows:

```
[ALERT] gpu_temperature{node="gpu-node-08", gpu="0"} = 94°C  
        Threshold: 90°C, Duration: 15 minutes
```

Simultaneously, `nvidia-smi` on that node shows:

```
Clk Throttle Reason:   SW Thermal Slowdown: Active
```

What is the correct response?

- A) Increase the alert threshold to 97°C — H100 max is 100°C
- B) 94°C sustained for 15 minutes with active thermal throttling is serious. GPU clocks are reduced, directly impacting training performance. Actions: (1) Cordon gpu-node-08 to prevent new pod scheduling, (2) Check if throttling is impacting your training job (NCCL slowdowns), (3) Open a Nebius support ticket — this is a datacenter cooling issue that requires physical intervention.
- C) Restart the GPU to reset the temperature
- D) Reduce batch size on gpu-node-08 to lower compute load and temperature

---

**Q34.** You receive a `terraform plan` output showing:

```hcl
# nebius_mk8s_v1_cluster.production will be updated in-place
  ~ resource "nebius_mk8s_v1_cluster" "production" {
      ~ control_plane {
          ~ public_endpoint_allowed_cidrs = [
              - "10.0.0.0/8",
              + "10.0.0.0/8",
              + "198.51.100.0/24",
            ]
        }
    }
```

This is adding your new office IP to the Kubernetes API server's allowed CIDRs. Is this safe to apply?

- A) No — CIDR changes force cluster recreation
- B) Yes — `public_endpoint_allowed_cidrs` is a mutable field (in-place update, no recreation). The plan shows the old `10.0.0.0/8` is kept (`-` then `+` is how Terraform shows list modifications when keeping existing items). No existing access is removed. Safe to apply.
- C) Yes but only apply during a maintenance window
- D) No — you need to recreate the cluster to change CIDR allowlists

---

**Q35.** You need to investigate why a training pod OOMKilled at 3 AM. The pod is gone. What are your options for finding the details?

- A) Check `kubectl get events -n training` — pod events are kept 24 hours
- B) Multiple approaches: (1) `kubectl get events -n training --field-selector reason=OOMKilling` if events haven't expired (~1 hour default), (2) Check `container_memory_working_set_bytes` metric in Nebius Monitoring for that pod at 3 AM, (3) `kubectl logs <pod-name> --previous -n training` for pre-crash logs if the pod object still exists, (4) `dmesg | grep -i oom` on the node. Kubernetes events expire quickly — use your observability stack for historical OOM investigation.
- C) SSH to the node and check `dmesg` for OOM events only
- D) Check `kubectl logs <pod-name> --previous -n training` only

---

**Q36.** You're asked to verify the Nebius Terraform provider version being used. What is the source of truth?

- A) `cat .terraform/providers.json`
- B) `.terraform.lock.hcl` pins the exact provider version after `terraform init`. Also run `terraform providers` in the module directory to see resolved constraints. The `nebius/nebius` provider version is recorded in the lock file and cannot drift without an explicit `terraform init -upgrade`.
- C) `nebius --version` shows the provider version
- D) Check `terraform.tfvars` for the provider version variable

---

**Q37.** You run:

```bash
$ kubectl get pods -n gpu-operator
NAME                                    READY   STATUS    RESTARTS
gpu-operator-7d6b5c4f8-xk9p2          1/1     Running   0
nvidia-device-plugin-daemonset-4xm9l  0/1     Pending   0
nvidia-device-plugin-daemonset-jk8p3  0/1     Pending   0  
nvidia-driver-daemonset-6vhqw         1/1     Running   0
nvidia-driver-daemonset-p8sw2         1/1     Running   0
```

Two device plugin pods are Pending. What is likely causing this?

- A) The device plugin pods need GPU access to schedule — catch-22
- B) Device plugin daemonset pods pending on specific nodes usually means those nodes have a `NoSchedule` taint that the device plugin daemonset's tolerations don't cover. Check: `kubectl describe pod nvidia-device-plugin-daemonset-4xm9l` for the scheduling failure reason. GPU Operator daemonsets must tolerate all node taints to run on every node.
- C) The device plugin needs more GPU memory to run
- D) Two device plugins can't run simultaneously — delete one of the pending pods

---

**Q38.** A colleague runs:

```bash
nebius compute instance delete --id inst-gpu-prod-07 --async
```

Then loses network connectivity and can't check if it worked. How do you verify the deletion status when network is restored?

- A) Log into the Nebius console — it shows real-time resource state
- B) Run `nebius compute instance get --id inst-gpu-prod-07`. If deleted: returns `Error: resource not found` or state `DELETED`. If still deleting: shows state `DELETING`. The `--async` flag means deletion started but didn't wait to confirm. You can also check the operation status with the operation ID that was returned by the original command.
- C) Check `terraform state list` to see if the instance is still tracked
- D) There's no way to check — async operations don't provide status

---

**Q39.** You check the distributed traces for a model serving pipeline and see:

```
Trace ID: tr-99182
  [0ms]     API Gateway    → Request received
  [2ms]     Load Balancer  → Request forwarded  
  [4ms]     Inference Svc  → Request received
  [4ms]     Inference Svc  → Model forward pass start
  [3847ms]  Inference Svc  → Model forward pass complete
  [3850ms]  API Gateway    → Response sent
```

The model forward pass takes 3843ms but should be <500ms. What do you investigate?

- A) The inference service is having network issues — 3843ms is latency
- B) 3843ms forward pass vs expected 500ms = ~7.7× slower. Investigate on the inference node: (1) `nvidia-smi` — is the GPU actually being used during inference? (2) Is the model in FP32 instead of FP16? (FP32 is ~6× slower on transformer workloads), (3) Is the model accidentally on CPU (`model.device == 'cpu'`)? (4) Is batch size 1 when the model was optimized for larger batches? (5) Is GPU throttling active (`nvidia-smi -q | grep Throttle`)?
- C) The API gateway is throttling requests — check rate limits
- D) The load balancer is adding 46ms overhead — optimize networking

---

**Q40.** You want to check if two Nebius Kubernetes clusters have compatible node group configurations before migrating workloads from cluster A to cluster B. Which commands help?

- A) `kubectl diff cluster-a cluster-b`
- B) Run `nebius mk8s get-compatibility-matrix` to see supported K8s versions. Then: `nebius mk8s node-group list --parent-id $CLUSTER_A_ID` and `nebius mk8s node-group list --parent-id $CLUSTER_B_ID`. Check that cluster B has: same or newer K8s version, same GPU platform, sufficient node capacity, and compatible subnet/networking. Workload compatibility also requires checking resource requests against what cluster B can provide.
- C) `nebius mk8s cluster compare --source $A_ID --dest $B_ID`
- D) Compare Terraform state files for both clusters

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | `auditors` group has view-only access to Audit Logs and metadata but cannot list or manage compute resources. `viewers` or `editors` is needed to list instances. The PermissionDenied error confirms the group membership is the issue. |
| 2 | **B** | An unauthorized deletion of production model weights is a security incident. Immediate revocation of the SA's access + full audit of what the SA did in the same time window is the correct IR response. |
| 3 | **C** | You need both pieces of information: which bucket (to find root cause) and where the egress is going (to understand billing). Both together explain the $89K charge. |
| 4 | **B** | MysteryBox versioning enables zero-downtime rotation. Create new version → set as primary → apps fetch fresh copy on next call. No service restart required for apps that fetch on each use. |
| 5 | **B** | Adding a CI/CD SA to `editors` is a privilege escalation risk. If the pipeline is compromised, attackers get `editors` access to the entire tenant. Least-privilege review is required. |
| 6 | **B** | Last rotation: March 1. +90 days = May 30. Today = June 28 = 29 days overdue. Automatic rotation is silently failing. Verify and manually trigger. Set up failure alerts for KMS rotation. |
| 7 | **B** | Object Storage lifecycle policies automate retention management — set once, runs forever. Validate whether 6-month retention is an actual compliance requirement or just historical practice. |
| 8 | **B** | Editors = manage resources + data. Viewers = read resources + data (with appropriate bucket policies). Auditors = read-only control plane, no data. The correct mapping is: (a) editors, (b) viewers, (c) auditors. |
| 9 | **B** | No `nvidia.com/gpu` in node resources = GPU Operator not installed. Nodes exist but GPUs are invisible to Kubernetes. Install from Nebius OCI chart registry — not the generic NVIDIA charts. |
| 10 | **B** | The warning explicitly says IB is disabled because `NCCL_IB_DISABLE` is not set. Absent = disabled by default. Set `NCCL_IB_DISABLE=0` to enable. The 28.5 GB/s bandwidth confirms Ethernet was used. |
| 11 | **B** | Terraform creates VMs (the nodes), not GPU-ready containers. GPU Operator must be separately installed to: load kernel modules, install drivers, configure containerd GPU runtime, and register GPUs as K8s resources. |
| 12 | **A** | Driver 570.x = `cuda12.8` preset (Ubuntu 24.04). Driver 580.x = `cuda13.0` preset. The `nvidia-smi` output shows 570.133.07 which maps directly to `cuda12.8`. This mapping is exam-critical knowledge. |
| 13 | **B** | `fabric-5` = H200 in `eu-west1`. `fabric-7` = H200 in `eu-north1`. The error message confirms the region-fabric mismatch. Always verify the correct fabric for each region before creating GPU clusters. |
| 14 | **B** | `NCCL_IB_HCA=mlx5_0,mlx5_1` enables multi-rail InfiniBand, using both HCAs for double the bandwidth. Without this, NCCL uses only one HCA even if two are available and Active. |
| 15 | **B** | `/dev/nvidia-uvm` = NVIDIA UVM kernel module device node. If it's missing, the module didn't load. The GPU Operator's driver daemonset is responsible for loading it. Check the driver pod health on that specific node. |
| 16 | **B** | `alloc` = occupied by other jobs. `down` = unavailable. Only 6 `idle` nodes are free. An 8-node job cannot run until at least 2 more nodes become available. The researcher must wait. |
| 17 | **B** | 30 minutes stuck in PROVISIONING usually means quota exhaustion or capacity shortage, not a software hang. Check quota first, then capacity availability. Don't delete healthy running nodes. |
| 18 | **B** | `nvidia_peermem` = GPUDirect RDMA support. Only needed for direct GPU-to-NIC memory transfer (InfiniBand-based training requiring peak bandwidth). For Ethernet-based NCCL, safe to ignore. Network Operator provides peermem for IB clusters. |
| 19 | **B** | All pods on one node = scheduling constraint. `kubectl describe pod` reveals the reason. Check for nodeSelector, nodeAffinity, or NoSchedule taints on node-02. This is almost always a configuration issue, not a scheduler bug. |
| 20 | **B** | `us-central1-b` = B200 InfiniBand fabric. `us-central1-a` = H200. Nebius uses zone names as fabric IDs in `us-central1`. Different naming convention vs. `eu-north1`'s `fabric-N` scheme. |
| 21 | **B** | `nebius mk8s cluster get-credentials` downloads and merges the kubeconfig for Nebius Managed Kubernetes. It's the CLI-native way to configure kubectl access. |
| 22 | **B** | `DiskPressure=True` is a pre-failure warning. Kubernetes will escalate to pod eviction and scheduling blocks if not addressed. Clean up disk space now — don't wait for the node to take automated action. |
| 23 | **B** | Immediate: free space by deleting old checkpoints. Long-term: rolling cleanup in training code (`save_total_limit`). Error handling around `torch.save` prevents job crashes from disk full conditions. |
| 24 | **B** | Exit code `1:0`: the `1` is the Unix exit code (error), the `0` is the signal (no signal = clean process exit with error code). The Slurm batch step (shell) completed; the Python script inside it returned 1. Check Python stderr. |
| 25 | **B** | `SUCCEEDED` = process exit 0. Empty output = writes silently failed. Common causes: wrong S3 endpoint (using AWS instead of Nebius), wrong path, or missing write permissions. Job logs will show write errors if any occurred. |
| 26 | **B** | Ranks 12-15 on specific nodes stopping simultaneously = localized node problem, not a global network issue. Check those specific nodes for IB port failures, GPU Xid errors, or thermal throttling. One bad node causes all its ranks to hang. |
| 27 | **B** | `WORLD_SIZE = 4 × 8 = 32`. 4 nodes × 8 processes per node = 32 total distributed processes. This is fundamental distributed training math — memorize the formula: WORLD_SIZE = nodes × GPUs_per_node. |
| 28 | **B** | Zero GPU utilization for 24 hours during a "running" Serverless AI Job = stuck. This is also burning GPU-hours with zero useful work. Cancel immediately, check logs for root cause, debug, then resubmit. |
| 29 | **B** | B200 (180 GB) vs H100 (80 GB): for a 70B FP16 model (~140 GB weights), B200 has much more headroom for activations and KV cache. B200's memory bandwidth advantage benefits transformer workloads. If H100 fits the model, it's still viable, but B200 is technically superior. |
| 30 | **B** | Optimizing a healthy system without a defined target is risky. 92% GPU utilization is good. Ask what "optimize" means: lower cost, lower latency, higher throughput? Each has a different solution. Identify the problem before changing a working system. |
| 31 | **B** | `etcd_cluster_size: 3 → 1` forces cluster recreation (destroys production) AND removes HA. This is catastrophic. Block immediately, explain the two issues (HA loss + forced recreation), require proper review process. |
| 32 | **B** | `ResourceExhausted` = project GPU quota limit hit. Request quota increase via console. Temporary workaround: lower node count to stay within quota. Don't disrupt running workloads to free quota. |
| 33 | **B** | 94°C + active thermal throttling = cooling issue requiring datacenter intervention. Cordon the node to prevent new jobs, assess impact on current training, open Nebius support ticket. You cannot fix datacenter cooling yourself. |
| 34 | **B** | The diff shows the old CIDR being kept and a new one added. `public_endpoint_allowed_cidrs` is a mutable field (in-place update). No cluster recreation. No existing access removed. Safe to apply. |
| 35 | **B** | OOMKill investigation requires all available sources: kubectl events (short-lived), Nebius Monitoring metrics (historical memory), previous pod logs, and node dmesg. Use all tools together for a complete picture. |
| 36 | **B** | `.terraform.lock.hcl` is the authoritative pinned provider version. `terraform providers` shows resolved constraints. These two together give you the exact provider version in use. `nebius --version` is the CLI version, not the Terraform provider version. |
| 37 | **B** | DaemonSet pods pending = scheduling issue on those specific nodes. Most common cause: node taints that the daemonset's `tolerations` don't include. `kubectl describe pod` on the pending pod shows the scheduling failure reason explicitly. |
| 38 | **B** | `nebius compute instance get --id inst-gpu-prod-07` checks current resource state directly. `DELETED` or `not found` = deletion succeeded. `DELETING` = still in progress. The `--async` flag returns an operation ID for async status tracking. |
| 39 | **B** | Distributed traces pinpoint which stage is slow (inference forward pass). Now diagnose why the forward pass is slow: GPU utilization, model precision (FP32 vs FP16), model device (GPU vs CPU), batch size efficiency. Traces identify where; metrics explain why. |
| 40 | **B** | `nebius mk8s get-compatibility-matrix` + `node-group list` for both clusters gives you the information needed for migration planning: K8s version compatibility, GPU platform match, and available capacity in the destination cluster. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Exam ready |
| 30–35 | Close — focus on weak domains |
| 24–29 | More practice needed |
| <24 | Re-read notes with focus on applying knowledge |
