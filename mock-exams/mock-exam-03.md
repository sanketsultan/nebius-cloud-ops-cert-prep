# Nebius AI Cloud Ops — Mock Exam 3 (Scenario-Based)

**Format:** Every question is a scenario. Pick the best action or answer.
**Time:** ~55 minutes | **Pass mark:** 70% (28/40)

> Attempt all questions before checking the answer key at the bottom.

---

## Domain 1: Security, Compliance & Billing (Q1–Q8)

**Q1.** You are onboarding a new ML team to your Nebius tenant. They need to: create and manage GPU VMs, read/write Object Storage, but NOT be able to invite other users or change IAM settings. Which group do you assign them to?

- A) admins — they need broad access
- B) editors — they can manage most resources and access data but cannot manage IAM at tenant level
- C) viewers — sufficient for GPU VM management
- D) auditors — they only need read access initially

---

**Q2.** A security incident response requires you to immediately revoke all access for a service account that was compromised. The SA has an access key and is in the editors group. What is the complete set of actions?

- A) Delete the access key only — the SA cannot do anything without its key
- B) Delete the access key AND remove the SA from the editors group AND delete any authorized keys — all credential and permission vectors must be closed simultaneously
- C) Remove the SA from the editors group only — keys become unusable without group membership
- D) Delete the service account entirely — this automatically revokes all associated keys and group memberships

---

**Q3.** A compliance audit requires you to demonstrate that no unauthorized person accessed model weights stored in an Object Storage bucket over the last 3 months. Which combination of Nebius services provides this evidence?

- A) Audit Logs (who accessed which resources) + Object Storage access logs (which objects were read)
- B) Monitoring Metrics — track Object Storage request count per user
- C) Billing history — show which service accounts incurred Object Storage egress costs
- D) MysteryBox audit trail — all Object Storage accesses are logged there

---

**Q4.** Your production Kubernetes cluster uses a service account to pull images from a private Nebius Container Registry. The pull started failing today with `401 Unauthorized`. The service account and its access key exist. The pod spec references the correct image. What is the most likely cause and fix?

- A) The Container Registry is down — wait for Nebius to resolve it
- B) The access key was rotated or deleted. Regenerate an access key for the service account, update the Kubernetes image pull secret with the new key, and ensure the SA is in the editors group.
- C) The pod spec needs to reference the updated `imagePullPolicy: Always`
- D) The service account needs to be moved from project-level to tenant-level

---

**Q5.** You discover that a training job has been reading training data from Object Storage using a service account that also has write and delete access to the production model bucket. You need to fix this without disrupting the running training job. What do you do?

- A) Immediately remove the SA from the editors group — this stops the training job
- B) Create a new service account with read-only access (viewers) to the training data bucket only. Update the next training job to use this new SA. Remove excessive permissions from the old SA once no active jobs use it.
- C) Change the Object Storage bucket policy to read-only
- D) Add a KMS encryption key to the production model bucket to prevent writes

---

**Q6.** You need to rotate a database password used by 50 Kubernetes pods across 3 different deployments. The password is stored in MysteryBox. What is the rotation procedure with minimum downtime?

- A) Update the password in all 3 deployment manifests and roll them out one by one
- B) Create a new version of the MysteryBox secret with the new password, set it as primary. Pods that fetch the secret on their next request will get the new password. For pods that cache it, trigger a rolling restart.
- C) Delete the old MysteryBox secret and create a new one with a different name
- D) Update the Kubernetes Secret and trigger rolling restarts of all 50 pods simultaneously

---

**Q7.** A team member says: "Our KMS symmetric key is used to encrypt model artifacts in Object Storage. If we delete the KMS key, we can delete the encrypted objects too." What is wrong with this statement?

- A) Nothing — deleting the KMS key makes the encrypted data unreadable and then the objects can be deleted
- B) Deleting the KMS key makes all data encrypted with it permanently inaccessible and unrecoverable — including the model artifacts. The objects still exist in Object Storage but cannot be decrypted. You would lose the models permanently.
- C) Deleting a KMS key automatically decrypts all data first
- D) KMS symmetric keys cannot be deleted once used

---

**Q8.** Your team's GPU spend jumped 300% this month. You need to identify which project and service account caused the spike. Where do you look?

- A) Audit Logs — filter by VM creation events to see what was provisioned
- B) Billing reports filtered by project and resource type, combined with Audit Logs to see which SA created the expensive resources and when
- C) Monitoring Metrics — check `gpu_utilization` per service account
- D) Object Storage access logs — excessive model downloads would indicate training activity

---

## Domain 2: Setting Up & Operating GPU Clusters (Q9–Q22)

**Q9.** You need to train a model that requires 2,304 GB of total GPU memory (to hold model weights during training). Reviewing the Nebius GPU options, which platform and how many VMs do you need at minimum?

- A) H100 SXM (80 GB × 8 = 640 GB per VM) — need 4 VMs (2,560 GB total, covers 2,304 GB)
- B) H200 SXM (141 GB × 8 = 1,128 GB per VM) — need 3 VMs (3,384 GB total, covers 2,304 GB)
- C) B200 SXM (180 GB × 8 = 1,440 GB per VM) — need 2 VMs (2,880 GB total, covers 2,304 GB)
- D) B300 SXM (288 GB × 8 = 2,304 GB per VM) — need 1 VM (exactly 2,304 GB)

---

**Q10.** Your 8-node H100 training cluster is experiencing intermittent slowdowns every few hours. `ibstat` shows all ports as `Active`. `ibcheckerrors` shows zero errors. NCCL bandwidth tests pass. But `nvidia-smi -q | grep -i ecc` shows:

```
ECC Errors
  Volatile
    Single Bit: 142 (increasing)
    Double Bit: 0
```

What should you do?

- A) Ignore it — single-bit ECC errors are corrected automatically and do not affect training correctness
- B) Single-bit ECC errors are corrected by hardware but their increasing count indicates GPU DRAM degradation. Drain the affected node now — a double-bit uncorrectable error could corrupt training results without warning. Open a support ticket.
- C) Restart the GPU driver on the affected node
- D) Reduce the batch size to reduce memory pressure

---

**Q11.** You create a Managed Kubernetes cluster and notice the default settings were applied: public endpoint enabled, 3 etcd stores. Your security team asks you to restrict the API endpoint to your company's IP only (`198.51.100.0/24`) without rebuilding the cluster. How do you do this?

- A) Rebuild the cluster with `--control-plane-endpoints-public-endpoint=false`
- B) Run `nebius mk8s cluster update --id $ID --control-plane-endpoints-public-endpoint-allowed-cidrs 198.51.100.0/24` to add the CIDR restriction without rebuilding
- C) Update the VPC subnet routing table to restrict access
- D) Configure a Kubernetes NetworkPolicy to block external IPs

---

**Q12.** A GPU node was rebooted for maintenance. After coming back online, `kubectl get nodes` shows it as `Ready`. But when you submit a GPU job to this node specifically using `nodeSelector`, the pod stays pending with `0/1 nodes available: 1 Insufficient nvidia.com/gpu`. What do you check?

- A) Whether the node is cordoned
- B) Check GPU Operator driver daemonset log for this node — after a reboot, the driver reinstalls. It may still be in progress. Wait for "Done, now waiting for signal" in the logs.
- C) Whether the node has enough CPU resources
- D) Whether the GPU job has the correct `nvidia.com/gpu` resource request in its spec

---

**Q13.** You have a 4-node B200 cluster. You want to add InfiniBand connectivity. The Nebius Managed Kubernetes node group was created without the Network Operator. What is the correct installation command?

- A) `helm install network-operator nvidia/network-operator -n kube-system`
- B) `helm install network-operator oci://cr.eu-north1.nebius.cloud/marketplace/nebius/nvidia-network-operator/chart/network-operator --version 25.7.0 -n nvidia-network-operator --create-namespace --wait`
- C) `kubectl apply -f https://raw.githubusercontent.com/Mellanox/network-operator/master/deploy/operator.yaml`
- D) `nebius mk8s operator install nvidia-network-operator --cluster-id $ID`

---

**Q14.** Your training job uses 4 nodes in `us-central1`. You want maximum bandwidth between nodes. You chose `us-central1-a` as the InfiniBand fabric. The VMs are B200 GPUs. Will this work?

- A) Yes — `us-central1-a` supports all GPU types in `us-central1`
- B) No — `us-central1-a` is for H200 GPUs. B200 GPUs in `us-central1` use fabric `us-central1-b`. Using the wrong fabric means B200 VMs cannot join the cluster.
- C) Yes — any GPU can use any fabric in the same region
- D) No — B200 is not available in `us-central1`

---

**Q15.** A shared filesystem is approaching capacity (4.8 PiB used of 5 PiB maximum). Training jobs will fail if they can't write checkpoints. What is the immediate action?

- A) Request a quota increase from Nebius support for more than 5 PiB
- B) Move older checkpoints and datasets from the shared filesystem to Object Storage (cheaper, virtually unlimited capacity), freeing up space. The 5 PiB limit is per filesystem — you cannot expand beyond this.
- C) Create a second shared filesystem and split the data
- D) Enable compression on the shared filesystem

---

**Q16.** You need to set up storage for a Slurm controller VM. It needs: (a) survive disk failures without data loss, (b) 20,000 IOPS minimum, (c) encrypted. Which disk type do you choose?

- A) Network SSD NRD — 75,000 IOPS, encryption optional
- B) Network SSD IO M3 — 75,000 IOPS, 3-way replication (survives failures), enable encryption at creation
- C) Network SSD standard — encrypted by default, erasure coding, but only 40,000 write IOPS
- D) Shared filesystem — always encrypted, survives failures

---

**Q17.** A training job is failing on node-06 with CUDA errors. You drain node-06 and the job restarts on the remaining 7 nodes. However, 30 minutes later, node-03 also starts showing CUDA errors. You check and find no ECC errors on node-03. What do you investigate next?

- A) Reinstall the GPU Operator on both nodes
- B) Check `nvidia-smi -q -d TEMPERATURE` on node-03 — cascading CUDA errors across multiple nodes after a previous node failure can indicate thermal issues or a power supply problem affecting the rack. Check for hardware issues at the infrastructure level.
- C) The training script has a memory leak — restart it with a smaller batch size
- D) NCCL is overloading node-03 after node-06 was removed

---

**Q18.** You run `kubectl drain gpu-node-02` and immediately see:

```
error: unable to drain node "gpu-node-02", aborting command...
There are pending nodes to be drained:
 gpu-node-02
Cannot evict pod as it would violate the pod's disruption budget.
```

The training job is already failing on this node. What is the fastest way to drain it?

- A) Wait for the PDB to allow eviction — this could take hours
- B) Delete the PDB: `kubectl delete pdb <pdb-name>`, then re-run drain. The node is already failing so the PDB's protection purpose is moot. Recreate the PDB after the node is drained.
- C) Use `kubectl delete pod --force --grace-period=0` for each pod manually
- D) Reboot the node to force pod eviction

---

**Q19.** Your team wants to upgrade from Kubernetes 1.32 to 1.33 in your production GPU cluster. How do you do this on Nebius?

- A) Run `nebius mk8s cluster update --id $ID --control-plane-version 1.33` for an in-place upgrade
- B) Kubernetes version is immutable — create a new 1.33 cluster, migrate all node groups and workloads to it, then decommission the 1.32 cluster. Plan carefully — the old cluster must be drained first.
- C) Update the version in the Terraform resource and apply — it upgrades in-place
- D) Kubernetes upgrades happen automatically on Nebius

---

**Q20.** You want to run a total of 32 H100 GPUs across 4 VMs for training. But you need them all connected via InfiniBand. Which preset and cluster setup is required?

- A) `4gpu-64vcpu-800gb` × 4 VMs in a GPU cluster
- B) H100 only supports 8-GPU presets for cluster use. Use `8gpu-128vcpu-1600gb` — but that's 8 GPUs per VM × 4 VMs = 32 GPUs. Create the 4 VMs in an H100 GPU cluster using `fabric-2/3/4/6` in `eu-north1`.
- C) `1gpu-16vcpu-200gb` × 32 VMs in a GPU cluster
- D) Any H100 preset works in a GPU cluster — GPU count per VM doesn't matter

---

**Q21.** You have a Kubernetes GPU node group and want to update the driver preset from `cuda12.8` to `cuda13.0`. Before running the update, what must you warn users about?

- A) Driver preset updates fail on node groups with more than 4 nodes
- B) Changing the driver preset triggers rolling node recreation — Managed Kubernetes will cordon, drain, delete existing nodes, and create replacements with the new driver. All running GPU jobs will be evicted during this process. Users must save checkpoints.
- C) The driver preset can only be changed once per month
- D) `cuda13.0` is only available on H200 and B200 nodes, not H100

---

**Q22.** You create a new Kubernetes cluster and forget to specify `etcd_cluster_size`. What is the default behavior?

- A) Cluster is created with 1 etcd store (HA disabled)
- B) Cluster creation fails — etcd cluster size is required
- C) Cluster is created with 3 etcd stores (HA enabled) — this is the default
- D) Cluster is created with 5 etcd stores for maximum HA

---

## Domain 3: Running Training & Inference Workloads (Q23–Q30)

**Q23.** A 5-day training job is running on 64 H200 GPUs. At day 4, the team realizes they forgot to save any checkpoints. A node failure occurs on day 4 at 23:00. What happens?

- A) The job automatically restarts from the last checkpoint
- B) Without checkpoints, the entire 4 days of training is lost. The job must restart from scratch. This is why checkpointing is essential — save model state to shared filesystem every N steps.
- C) Nebius automatically saves snapshots of GPU VM memory every hour
- D) The job can resume from the last epoch boundary automatically

---

**Q24.** You submit a Slurm job and it runs but exits after 30 seconds with no output. `sacct -j 3001` shows `ExitCode 127:0`. What does exit code 127 mean and what is the fix?

- A) Exit code 127 = memory error — increase `--mem` in the job script
- B) Exit code 127 = command not found — the executable specified in the job script doesn't exist on the nodes. Check the script's `#SBATCH` directives and the actual binary path. Common cause: wrong module loaded or incorrect path.
- C) Exit code 127 = InfiniBand error — check `ibstat`
- D) Exit code 127 = job was cancelled by the scheduler

---

**Q25.** Your team runs hyperparameter sweeps with 200 different configurations. Each run needs 1 H100 GPU for 20 minutes. They ask whether to use (A) a dedicated 8-GPU Kubernetes cluster running all the time, or (B) Serverless AI Jobs one per configuration. Currently the 8-GPU cluster is idle 80% of the time between sweep runs. What do you recommend?

- A) Option A — dedicated cluster avoids cold start overhead
- B) Option B — 200 jobs × 20 minutes × 1 GPU = 66.7 GPU-hours billed. Option A: 8 GPUs × 24hrs × days = vastly more billed time even when idle. Serverless AI Jobs are dramatically cheaper for intermittent workloads.
- C) Option A with cluster autoscaler to scale to zero between runs
- D) Neither — use Soperator for hyperparameter sweeps

---

**Q26.** A data scientist's training pod keeps restarting with `OOMKilled`. GPU memory usage is 95%. The batch size is already at minimum (1). What are the possible solutions?

- A) Increase the GPU count in the pod spec
- B) Options: (1) Enable gradient checkpointing in the model to trade compute for memory, (2) Use mixed precision (FP16/BF16) if training in FP32, (3) Move to a GPU with more VRAM (e.g., H200 141GB vs H100 80GB), (4) Use model parallelism to split across multiple GPUs
- C) Increase the pod's CPU memory limit — OOMKilled means CPU RAM, not GPU VRAM
- D) Restart the GPU Operator to reset VRAM allocation

---

**Q27.** A Slurm job requesting GPUs submitted via `sbatch` has been in `PD` state for 6 hours. `squeue -j 5001` shows `Reason: AssocGrpGRES`. What does this mean?

- A) Not enough GPU nodes — wait for nodes to become available
- B) The job is exceeding the GPU resource limit set in the Slurm association for the user or account. This is a quota enforcement issue — check with the Slurm admin to increase the GPU allocation limit or reduce the job's GPU request.
- C) The GPU hardware is faulty — `GRES` stands for GPU Resource Error State
- D) The job's time limit exceeds the partition's maximum wall time

---

**Q28.** You need to run inference on a 70B parameter model. An H100 SXM VM (8 × 80 GB = 640 GB) is not enough. What is the minimum number of H200 SXM VMs (8 × 141 GB = 1,128 GB each) needed to hold the model weights in FP16 (70B × 2 bytes ≈ 140 GB) plus intermediate activations (estimate 2× the weight size)?

- A) 1 VM — 1,128 GB > 140 GB × 3 = 420 GB
- B) 1 VM is sufficient for FP16 weights (140 GB) plus estimated activation memory (~280 GB) = ~420 GB total, well within 1,128 GB per H200 VM
- C) 2 VMs — to be safe with KV cache for inference
- D) 3 VMs — large models always need multiple VMs

---

**Q29.** Your team wants to use MLflow to track experiments for a project based in `eu-north2`. They created the MLflow cluster and it's not showing up. Your manager suggests opening a support ticket. What should you tell them first?

- A) Open the support ticket — MLflow should be available everywhere
- B) MLflow Managed Service is not available in `eu-north2` or `eu-west1`. This is a documented regional limitation, not a bug. The team needs to either move their project to a supported region or use a self-hosted MLflow on a VM in `eu-north2`.
- C) Try refreshing the console — it's a UI caching issue
- D) The MLflow cluster needs to be in the same project as the training jobs

---

**Q30.** A Serverless AI Job has been running for 3 days but shows no progress in logs. GPU utilization is 0%. The job is still in `Running` state and billing is accumulating. What is the issue and what do you do?

- A) This is normal — GPU utilization drops to 0% between data loading and compute steps
- B) The job is stuck (deadlocked, waiting on a network call, or failed silently). Check job logs for the last output. If the job is genuinely stuck, cancel it with the Nebius CLI to stop billing, fix the underlying issue, and resubmit.
- C) GPU utilization 0% means the job is using CPU only — this is fine
- D) Wait 24 more hours — long-running jobs sometimes have extended preprocessing phases

---

## Domain 4: Platform Automation & Maintenance (Q31–Q40)

**Q31.** A colleague writes this Terraform for a Nebius Kubernetes cluster:

```hcl
resource "nebius_kubernetes_cluster" "prod" {
  name      = "prod-gpu"
  parent_id = var.project_id
}
```

They run `terraform apply` and get `Error: Invalid resource type`. What is wrong?

- A) The `parent_id` field is not valid for Kubernetes clusters
- B) The resource type is wrong — it should be `nebius_mk8s_v1_cluster`, not `nebius_kubernetes_cluster`
- C) The cluster name cannot contain a hyphen
- D) The Nebius provider is not initialized

---

**Q32.** Your team has separate Terraform configs for dev, staging, and production environments, all stored in one Git repository. A new engineer accidentally runs `terraform apply` in the production directory thinking it was dev and destroys 3 production GPU clusters. What should be in place to prevent this?

- A) Separate Git repositories for each environment
- B) Enable `prevent_destroy = true` in lifecycle blocks for critical production resources, and require manual approval for production applies in the CI/CD pipeline. Also use separate state files with separate access credentials per environment.
- C) Use Terraform workspaces to separate environments
- D) Add a `count = 0` flag to disable production resources by default

---

**Q33.** You need to check if GPU node `gpu-node-07` still has the correct driver after a maintenance window. You SSH to the node. Which command confirms the NVIDIA driver version installed?

- A) `nvidia-smi --query-gpu=driver_version --format=csv`
- B) `kubectl describe node gpu-node-07 | grep driver`
- C) Check the GPU Operator daemonset pod log: last line should say "Done, now waiting for signal" and the log should contain the expected driver version (e.g., 580.x for `cuda13.0`)
- D) Both A and C together — A confirms the OS-level driver version, C confirms the Kubernetes-level operator installed correctly

---

**Q34.** You receive an alert: `gpu_utilization` on all 8 nodes dropped from 95% to 0% at 2:47 AM. You check `kubectl get pods -n training` and all pods show `Completed`. The training job was expected to run until 6 AM. What is most likely the explanation?

- A) InfiniBand failed and terminated all pods
- B) The training job finished early — the `Completed` pod status is the strongest evidence. Check training logs for loss convergence or a stopping criterion being met. GPU utilization dropping to 0% with pods in `Completed` state = normal job completion.
- C) A GPU hardware failure terminated all pods
- D) The Kubernetes scheduler evicted all pods for resource reclamation

---

**Q35.** You manage GPU infrastructure with Terraform. A vendor gave you a new Terraform module to deploy their ML platform on Nebius. The module's `main.tf` contains:

```hcl
provider "yandex" {
  token = var.nebius_token
}
```

What is wrong and what should you tell the vendor?

- A) Nothing — Nebius is built on Yandex Cloud, so the Yandex provider works
- B) Nebius has its own Terraform provider (`nebius/nebius`). The `yandex` provider is for Yandex Cloud, which is a separate platform. The vendor needs to update the module to use `provider "nebius" {}` and the correct Nebius resource types.
- C) The `yandex` provider works but needs the Nebius endpoint configured
- D) Ask the vendor to use the `aws` provider instead

---

**Q36.** You run `kubectl get nodes` and see:

```
NAME         STATUS                     
node-01     Ready,SchedulingDisabled   
node-02     Ready,SchedulingDisabled   
node-03     Ready                      
node-04     Ready                      
```

All training jobs are routing to node-03 and node-04 only. Node-01 and node-02 are idle. What happened and what do you do?

- A) Nodes 01 and 02 have hardware failures — replace them
- B) Nodes 01 and 02 are cordoned (`SchedulingDisabled`). No new pods can be scheduled on them. If this was unintentional, run `kubectl uncordon node-01 node-02` to restore normal scheduling.
- C) Node-03 and node-04 have higher priority in the scheduler
- D) The training job has a nodeSelector targeting only node-03 and node-04

---

**Q37.** Your Monitoring dashboard shows this for the inference cluster:

```
gpu_temperature: 91°C (node-02, GPU 0)
gpu_temperature: 72°C (all other GPUs)
```

The inference endpoint is serving requests normally. What should you do?

- A) Nothing — GPU temperatures up to 95°C are within spec
- B) Check `nvidia-smi -q -d TEMPERATURE -i 0` on node-02 to confirm and check for `Clocks Throttle Reason: SW Thermal Slowdown`. At 91°C, thermal throttling may be reducing GPU clock speed. Alert Nebius support about potential cooling issues on that specific GPU/node.
- C) Immediately terminate all inference pods on node-02
- D) Reduce the inference batch size on node-02 to lower GPU load

---

**Q38.** You are setting up Prometheus monitoring for your Nebius GPU cluster. A colleague suggests installing the `nvidia-dcgm-exporter` manually on each node. What is the Nebius-recommended approach?

- A) Manual DCGM exporter is the only option
- B) Use Nebius Monitoring's native Prometheus integration — it provides GPU metrics without installing exporters manually. Connect your Prometheus instance to Nebius Monitoring's endpoint.
- C) Install node_exporter on each node and configure custom GPU metrics
- D) Use the Grafana agent to scrape metrics from each node

---

**Q39.** A training job fails with `NCCL error: unhandled system error, NCCL version 2.18.1`. You check and `NCCL_IB_DISABLE=0`, `ibstat` shows Active ports, and the Network Operator is installed. What do you check next?

- A) Increase `NCCL_TIMEOUT`
- B) Verify `NICClusterPolicy` status: `kubectl get nicclusterpolicy.mellanox.com nic-cluster-policy -n nvidia-network-operator -o json | jq '.status.state'` — it must show "ready". If not, the Network Operator hasn't fully initialized despite being installed.
- C) Set `NCCL_DEBUG=WARN` to suppress the error
- D) Reinstall the GPU Operator

---

**Q40.** You manage 50 GPU VMs with Terraform. A critical security patch requires updating the boot disk image for all VMs. You update the `image_id` in the Terraform resource and run `terraform plan`. The plan shows 50 resources will be destroyed and recreated. This would terminate all running training jobs. What is the better approach?

- A) Apply the plan during off-hours — 50 VMs × ~5 minutes each = accept the downtime
- B) Use `terraform apply -target=nebius_compute_instance.vm["node-01"]` to update one VM at a time. For each VM: (1) cordon and drain it to move jobs to other nodes, (2) apply the image update for just that VM (it recreates), (3) verify it comes back healthy, (4) move to the next. This treats the patch as a rolling update.
- C) Use `terraform taint` on all 50 VMs at once and apply
- D) Update the image directly via the Nebius console on each VM without Terraform

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | `editors` can manage most resources and access data but cannot modify IAM at tenant level (adding users, changing groups). This matches the requirements: VM creation/management + Object Storage + no IAM changes. |
| 2 | **D** | For a compromised SA, delete everything: access keys (API credentials), authorized keys (IAM token credentials), and remove from all groups (permissions). Deleting the SA itself is the most complete action — it cascades. |
| 3 | **A** | Audit Logs show who accessed which Nebius resources (control-plane API calls). Object Storage access logs track which specific objects were read/written. Together they prove access or absence thereof. |
| 4 | **B** | `401 Unauthorized` from Container Registry = authentication failure. The most likely cause is a rotated/deleted access key. Regenerate the key, update the K8s image pull secret, and verify the SA has editors access to the registry. |
| 5 | **B** | Never remove permissions from a running job's SA — it breaks the job. Create a new minimal-permission SA for future jobs, then clean up the old SA once no active jobs depend on it. |
| 6 | **B** | MysteryBox versioning enables zero-downtime rotation. Create a new version → set as primary → apps fetch on next call get new password. Pods caching the old value need a rolling restart (not a simultaneous restart which would cause downtime). |
| 7 | **B** | Deleting a KMS key that encrypted data makes that data permanently inaccessible. The encrypted objects remain in Object Storage but are unreadable. This is an irreversible data loss scenario. |
| 8 | **B** | Billing reports by project/resource identify the cost spike source. Audit Logs then reveal which SA created those resources and when. Together they provide full accountability for the spend increase. |
| 9 | **C** | B200: 180 GB × 8 = 1,440 GB per VM. 2 VMs = 2,880 GB > 2,304 GB required. This is the minimum VM count. B300 (288 GB × 8 = 2,304 GB) also works with 1 VM if available. |
| 10 | **B** | Single-bit ECC errors are auto-corrected but an increasing count signals DRAM degradation. The next step is uncorrectable double-bit errors which corrupt data silently. Drain proactively before a catastrophic failure during training. |
| 11 | **B** | `nebius mk8s cluster update` with `--control-plane-endpoints-public-endpoint-allowed-cidrs` adds IP restrictions to the existing public endpoint without rebuilding the cluster. This is a mutable field. |
| 12 | **B** | After a reboot, the GPU Operator driver daemonset reinstalls the driver. This takes several minutes. The node shows `Ready` in K8s before the GPU driver finishes installing. Wait for "Done, now waiting for signal" before scheduling GPU jobs. |
| 13 | **B** | Always install Nebius operators from the Nebius-specific OCI chart registry. The generic NVIDIA or Mellanox charts are not configured for Nebius's infrastructure and will not work correctly. |
| 14 | **B** | `us-central1-a` = H200 fabric. `us-central1-b` = B200 fabric. Using the wrong fabric causes VMs to fail joining the GPU cluster. Always match GPU type to its correct fabric. |
| 15 | **B** | 5 PiB is the maximum per filesystem. Move cold data (old checkpoints, stale datasets) to Object Storage immediately. Shared filesystem is for active training data; Object Storage is for archival. You cannot expand past 5 PiB per filesystem. |
| 16 | **B** | Requires: survive failures (NRD fails this — no redundancy), 20,000 IOPS (IO M3 = 75,000 ✓), encrypted (enable at creation). IO M3 with 3-way replication and optional encryption is the correct choice. Network SSD standard only does 40k write IOPS. |
| 17 | **B** | Cascading node failures not explained by ECC or NCCL issues may indicate rack-level hardware problems (cooling, power). Check temperatures across all nodes in the same rack. Escalate to Nebius support if the pattern continues. |
| 18 | **B** | When the node is already failing and training is broken, the PDB's protection purpose is already violated. Delete the PDB temporarily to unblock drain, then recreate it after. Force-deleting pods with `--grace-period=0` can corrupt data. |
| 19 | **B** | Kubernetes version is immutable on Nebius. In-place upgrades are not supported. You must create a new cluster with the target version. Plan workload migration carefully — new cluster, new node groups, redeploy workloads. |
| 20 | **B** | Only 8-GPU presets work in GPU clusters. `8gpu-128vcpu-1600gb` × 4 VMs = 32 H100 GPUs connected via InfiniBand in `eu-north1` using `fabric-2/3/4/6`. There is no 4-GPU preset for H100 cluster use. |
| 21 | **B** | Driver preset change = rolling node recreation. Users must checkpoint their jobs before the upgrade. Managed Kubernetes handles the rolling update automatically but all running GPU pods will be evicted during their node's recreation. |
| 22 | **C** | The default for `etcd_cluster_size` is 3 (HA enabled). If you don't specify it, Nebius creates 3 etcd stores. This is also free — HA doesn't cost extra. |
| 23 | **B** | No checkpoints = no resume. 4 days of computation is lost. The job must restart from scratch. This is the textbook example of why checkpointing every N steps is non-negotiable for long training runs. |
| 24 | **B** | Exit code 127 universally means "command not found" in Unix/Linux. The script's executable doesn't exist on the Slurm nodes. Common causes: wrong module loaded, incorrect path, binary not installed on all nodes. |
| 25 | **B** | 200 jobs × 20 min × 1 GPU = 66.7 GPU-hours. Dedicated 8-GPU cluster idle 80% of the time costs 8 GPU-hours per hour of wall time. For irregular sweep workloads, Serverless AI Jobs can be 10x cheaper. |
| 26 | **B** | OOMKilled with batch_size=1 means the model itself doesn't fit in VRAM, not just the batch. Gradient checkpointing trades recompute for memory. Mixed precision halves weight memory. More VRAM GPU is the cleanest solution. |
| 27 | **B** | `AssocGrpGRES` = Association Group GRES (Generic Resource) limit exceeded. This is Slurm's account-level GPU quota enforcement. Not a hardware issue — an administrative resource limit. Contact the Slurm admin. |
| 28 | **B** | 70B FP16 weights = 140 GB. Estimated activation memory for inference is much less than training (no optimizer states, no gradients). A single H200 VM with 1,128 GB of total VRAM handles a 70B model inference comfortably. |
| 29 | **B** | `eu-north2` is one of two regions where MLflow Managed Service is explicitly unavailable. This is a documented platform limitation. Opening a support ticket won't help — it's intentional, not a bug. |
| 30 | **B** | Zero GPU utilization for an extended period in a "Running" Serverless AI Job indicates the job is stuck. Check logs for the last activity. Cancel to stop billing, debug the issue (deadlock, network failure, silent exception), then resubmit. |
| 31 | **B** | `nebius_kubernetes_cluster` is not a valid Nebius Terraform resource type. The correct resource is `nebius_mk8s_v1_cluster`. This is a common mistake — the resource name does not follow the pattern you might expect. |
| 32 | **B** | `prevent_destroy = true` in lifecycle blocks prevents accidental `terraform destroy`. Manual approval gates in CI/CD prevent unauthorized applies. Separate credentials per environment prevent using prod credentials in dev accidentally. |
| 33 | **D** | Both together: `nvidia-smi --query-gpu=driver_version` confirms what the OS sees, while the GPU Operator daemonset logs confirm that the Kubernetes operator successfully managed the installation at the right version. |
| 34 | **B** | Pods in `Completed` state + zero GPU utilization is the normal signature of a finished job. The job likely converged early or a stopping criterion was met. Check training logs for loss curves and final metrics before assuming anything is wrong. |
| 35 | **B** | Nebius is a separate cloud platform from Yandex Cloud despite historical connections. The Terraform provider is `nebius/nebius`, not `yandex`. Using the wrong provider causes authentication failures and resource type mismatches. |
| 36 | **B** | `SchedulingDisabled` = cordoned. If unintentional, uncordon them immediately. Running `kubectl uncordon node-01 node-02` restores them to normal scheduling. The training jobs will start spreading across all 4 nodes again. |
| 37 | **B** | 91°C is near the throttling threshold. `nvidia-smi` can confirm if thermal throttling is active. If clocks are being reduced, inference latency will increase. Report to Nebius support as this is a datacenter cooling issue, not something you can fix. |
| 38 | **B** | Nebius Monitoring's native Prometheus integration is the recommended path — no manual exporter installation needed. It provides GPU metrics out of the box. Manual DCGM setup is complex and maintenance-heavy. |
| 39 | **B** | `NICClusterPolicy` status being not "ready" means the Network Operator components haven't fully initialized. Even though the operator is installed, it's not operational. This is the most common hidden cause of NCCL IB failures after Network Operator install. |
| 40 | **B** | `terraform apply -target` allows updating one resource at a time. Combined with cordon/drain (moving jobs off the target VM before its recreation), this achieves a rolling image update with no simultaneous downtime across all 50 VMs. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Exam ready |
| 30–35 | Close — focus on weak domains |
| 24–29 | More practice needed |
| <24 | Re-read notes with focus on applying knowledge |
