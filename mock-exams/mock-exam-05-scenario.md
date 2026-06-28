# Nebius AI Cloud Ops — Mock Exam 5 (Scenario-Based, Hard)

**Format:** All scenarios. Every question tests applied knowledge, not recall.
**Time:** ~55 minutes | **Pass mark:** 70% (28/40)

> Attempt all questions before checking the answer key at the bottom.

---

## Domain 1: Security, Compliance & Billing (Q1–Q8)

**Q1.** Your team deploys a Python service that fetches an API key from MysteryBox at startup. The key expires every 30 days. On day 31, the service crashes with `authentication failed`. The MysteryBox secret has a new primary version but the service is still using the old key. What is the design flaw and fix?

- A) MysteryBox should auto-rotate keys — contact Nebius to enable this
- B) The service caches the secret at startup and never re-fetches. Fix: implement a re-fetch mechanism on auth failure (retry with a freshly-fetched key), OR load the secret dynamically on each request rather than at startup. Never cache long-lived credentials in memory without a refresh path.
- C) Secrets should not be stored in MysteryBox — use environment variables instead
- D) The service's IAM token expired — refresh it separately

---

**Q2.** Your company runs separate Nebius tenants for EU customers (data residency requirement) and US customers. An engineer accidentally creates a VM in the EU tenant to process US customer data. From a compliance standpoint, what is the issue?

- A) No issue — data doesn't actually move between tenants based on where the VM is created
- B) Even if data hasn't been moved yet, creating processing infrastructure in the wrong tenant breaks data residency controls. The VM should be destroyed immediately, the incident should be documented, and the engineer should be reminded of the data residency requirements. Processing EU-tenant data for US customers (or vice versa) may violate GDPR or other data localization laws.
- C) Move the VM to the correct tenant using `nebius compute vm move`
- D) Add a tag to the VM to indicate it's processing US data — this satisfies compliance

---

**Q3.** You need to grant a third-party auditing firm read-only access to Audit Logs for the past 12 months. They should not be able to see any actual training data, model weights, or compute resources. Which group do you add their service account to?

- A) viewers — they can view resources and data
- B) auditors — they have read-only access to control plane logs and resource metadata, but explicitly cannot access data stored in Object Storage or other data services. This is the correct group for compliance auditors.
- C) editors — they need read access to generate audit reports
- D) Create a custom group with specific permissions — default groups are too broad

---

**Q4.** You have 50 GPU VMs all sharing one service account with admin credentials "for convenience." A security scan flags this as critical risk. Without breaking any running workloads, what is the minimum set of changes needed?

- A) This is acceptable for internal infrastructure — security scan has false positives
- B) Create purpose-specific service accounts with minimum permissions: (1) `inference-sa` with read access to model bucket, (2) `training-sa` with read/write to training data bucket, (3) `monitoring-sa` with metrics read access. Rotate the credentials in each VM's configuration over a maintenance window. Never share admin-level SAs across workloads.
- C) Rotate the credentials on the existing admin SA — that's sufficient
- D) Remove the SA from VMs and use personal user accounts instead

---

**Q5.** After a security incident, a junior SRE suggests: "Let's disable all Audit Logs to prevent attackers from reading our audit trails." What is wrong with this suggestion?

- A) Nothing — audit logs are a security risk if attackers access them
- B) Audit Logs are your primary forensic tool after an incident. Disabling them eliminates your ability to investigate what the attacker did, what resources were accessed, and when. Instead, restrict Audit Log access to `admins` only and consider exporting them to an external SIEM. Never disable audit logging.
- C) You can't disable Audit Logs in Nebius — they're always on
- D) Disable only the data-plane logs, not control-plane logs

---

**Q6.** Your GPU training costs are allocated across 5 teams by project. Team C's project spent $47,000 this month but their budget is $10,000. You check their project and find 200 idle GPU VMs running for 3 weeks. Who created them?

- A) Check Object Storage access logs — they would show large data downloads that triggered VM creation
- B) Check Audit Logs filtered by the project, resource type `compute.instance`, action `create`, and the date range 3 weeks ago. The log entry for each VM creation will show which IAM principal (user or SA) created it, and from what IP. This is the authoritative source for "who did this."
- C) Check Monitoring metrics — GPU utilization near 0% will identify who owns the idle VMs
- D) Check MysteryBox access logs — the VMs would have fetched credentials when created

---

**Q7.** An engineer stores a database password in a Kubernetes Secret inside a Nebius Managed Kubernetes cluster, base64-encoded. A security auditor flags this as insecure. The engineer argues "it's encrypted at rest because Nebius encrypts etcd." Who is right and why?

- A) The engineer is right — etcd encryption at rest is sufficient for secrets
- B) Both have valid points but the auditor is more right. While Nebius does encrypt the etcd backing store, Kubernetes Secrets encoded as base64 are easily decoded by anyone with `kubectl get secret` access. The better practice is to use MysteryBox and have the pod fetch the secret at runtime, or use sealed-secrets encryption so the value is encrypted even in the Kubernetes API and Git.
- C) The auditor is wrong — base64 is an encryption method
- D) Kubernetes Secrets are never encrypted at rest on any platform

---

**Q8.** Your monthly bill shows $180,000 in Object Storage costs. You investigate and find a misconfigured training job has been writing 1 TB of NCCL debug logs per hour to Object Storage for 2 weeks. What is the immediate mitigation and the root-cause fix?

- A) Delete all objects in the bucket — fastest way to stop charges
- B) Immediate: stop the training job writing the logs. Delete the accumulated debug log objects (use `aws s3 rm --recursive s3://bucket/nccl-logs/` via S3-compatible CLI). Root-cause fix: add a lifecycle policy to the Object Storage bucket that automatically deletes objects with the `/nccl-logs/` prefix after 7 days. Also set `NCCL_DEBUG=WARN` (not `INFO`) in training configs to reduce log volume.
- C) Move the logs to a cheaper storage tier
- D) Contact Nebius support to waive the charges

---

## Domain 2: Setting Up & Operating GPU Clusters (Q9–Q22)

**Q9.** You want to deploy a B300 GPU cluster. Your architect says "use `me-west1-a` — B300 is available there." You check the Nebius docs and see B300 is listed for `uk-south1-a`. What do you tell your architect?

- A) Both are correct — B300 is available in multiple regions
- B) B300 GPUs are currently available in `uk-south1-a`, not `me-west1-a`. `me-west1-a` hosts B200 GPUs. Using the wrong region for your GPU cluster type will fail at provisioning time. Verify GPU availability per region in the docs before creating clusters.
- C) B300 is only available via special request to Nebius sales
- D) The docs are wrong — trust your architect's knowledge

---

**Q10.** You have a 4-node cluster with `8gpu-200vcpu-1600gb` H200 presets in `eu-west1` (fabric-5). All nodes show Active InfiniBand. You run an NCCL bandwidth test and get 45 GB/s instead of the expected 400+ GB/s. What is the most likely misconfiguration?

- A) NCCL_IB_DISABLE=1 is set — set it to 0
- B) Check `NCCL_IB_HCA` — if it's set to the wrong adapter name, NCCL will use the wrong interface. Also check `NCCL_SOCKET_IFNAME` — if set to the Ethernet interface instead of IB, all traffic goes over Ethernet (typically 25–100 GB/s vs 400 GB/s for InfiniBand). Run `NCCL_DEBUG=INFO` to see which transport NCCL is actually using.
- C) The 4-node cluster is too small for full InfiniBand bandwidth
- D) H200 InfiniBand bandwidth is capped at 50 GB/s per port — this is expected

---

**Q11.** A Nebius Managed Kubernetes cluster's node group needs to expand from 4 to 16 GPU nodes to handle a new training workload. The node group `gpu-workers` was created with `fixed_node_count = 4`. How do you expand it?

- A) Create a new node group with 12 nodes and let Kubernetes balance between them
- B) Run `nebius mk8s node-group update --id $NODE_GROUP_ID --fixed-node-count 16` to resize the existing node group. Managed Kubernetes will provision the additional 12 nodes. Monitor with `kubectl get nodes` until all 16 show `Ready`.
- C) Delete the node group and recreate with 16 nodes
- D) Node group size is immutable — create a second node group with 12 nodes

---

**Q12.** You run `kubectl get nodes` and all nodes show `Ready`. But `kubectl get pods -n training` shows all training pods in `Pending` state with the reason `0/16 nodes are available: 16 node(s) had untolerated taint {gpu-maintenance: "true": NoSchedule}`. What happened and how do you fix it?

- A) The GPU driver is not installed on any node — reinstall GPU Operator
- B) Someone added a `gpu-maintenance: "true": NoSchedule` taint to all 16 nodes, preventing pod scheduling. If maintenance is complete, remove the taint: `kubectl taint nodes --all gpu-maintenance:NoSchedule-`. If maintenance is ongoing, this is expected behavior.
- C) The training pods need a toleration for GPU nodes
- D) The node group needs to be recreated without the taint

---

**Q13.** You're setting up a GPU cluster for distributed training using H100 GPUs in `eu-north1`. The training framework requires GPUDirect RDMA (data goes directly between GPU and NIC, bypassing CPU). What must be true about your setup for GPUDirect RDMA to work?

- A) GPUDirect RDMA works on any InfiniBand cluster automatically
- B) GPUDirect RDMA requires: (1) NVMe-capable InfiniBand HCAs with peer memory support, (2) NVIDIA Network Operator installed (configures the HCAs), (3) The `rdma` device plugin configured, (4) Pods must request the RDMA resource. On H100 clusters with Nebius's InfiniBand fabric, all hardware prerequisites are met — you need the Network Operator and proper pod spec.
- C) GPUDirect RDMA only works on B200 and B300 GPUs
- D) GPUDirect RDMA requires direct physical access to the IB switch — not available in cloud

---

**Q14.** A 32-node training job has been running for 20 hours. Node `gpu-node-22` begins showing GPU throttling (clocks reduced by 30%). Other nodes are healthy. What do you do to minimize job disruption?

- A) Ignore it — 30% throttling on 1 of 32 nodes has negligible impact
- B) A single throttled node in distributed training becomes the bottleneck — it slows all 32 nodes because all-reduce synchronization waits for the slowest. Cordon `gpu-node-22` to prevent new pods, then evaluate: if training can continue with 31 nodes (requires script changes), proceed. Otherwise trigger a checkpoint, drain the node, and restart training on a healthy replacement. Alert Nebius support about the throttling.
- C) Restart the NCCL process on node-22 to reset clock speeds
- D) Reduce the global batch size to compensate for the throttled node

---

**Q15.** Your team uses `virtiofs` to mount a Nebius shared filesystem into VMs. A VM writing large checkpoint files reports throughput of only 2 GiB/s when the filesystem spec says 8 GiB/s write throughput. What is the most likely cause?

- A) The shared filesystem is overloaded — too many concurrent clients
- B) The 8 GiB/s limit is per filesystem, not per client. Check: (1) How many other VMs are currently writing to the filesystem simultaneously, (2) Whether the VM itself is hitting its network bandwidth limit (VM network cap may be lower than 8 GiB/s), (3) Whether virtiofs is configured with enough queue depth. Also check that you're writing large sequential blocks, not many small random writes.
- C) virtiofs is not compatible with Nebius shared filesystem — use NFS instead
- D) The 8 GiB/s is theoretical — real-world performance is always 25% of spec

---

**Q16.** You receive this error when deploying GPU Operator from Nebius's chart repo:

```
Error: failed to pull image "cr.eu-north1.nebius.cloud/marketplace/nebius/gpu-operator/...:v24.9.2":
rpc error: code = Unknown desc = failed to pull and unpack image: 
failed to resolve reference: unauthorized
```

What is the cause?

- A) The GPU Operator version 24.9.2 doesn't exist — use a different version
- B) The Kubernetes node's service account doesn't have credentials to pull from Nebius's private container registry. You need to create an image pull secret with Object Storage access key credentials and either add it to the pod spec or add it as a default pull secret for the namespace. Alternatively, configure the node's containerd to authenticate to `cr.eu-north1.nebius.cloud`.
- C) The Nebius container registry is in a different region — use `cr.eu-west1.nebius.cloud`
- D) GPU Operator must be installed via `nebius mk8s operator install`, not Helm

---

**Q17.** You need to provision new GPU nodes faster during a training campaign. Node provisioning currently takes 15 minutes. A colleague suggests pre-warming nodes by creating them in advance and keeping them idle. What is the Nebius concept for this, and what is the cost implication?

- A) There is no pre-warming concept — 15 minutes is the standard boot time
- B) You can create capacity reservations (capacity blocks) in Nebius to guarantee GPU capacity ahead of time. Pre-provisioned idle nodes are billed at full rate even when idle. The tradeoff: faster time-to-training vs. cost of idle compute. For time-sensitive training campaigns, capacity blocks are justified.
- C) Pre-warm using spot instances — they cost nothing when idle
- D) Contact Nebius support to enable "fast provisioning mode" on your account

---

**Q18.** A training job requires 64 GPUs. You have a 8-node H200 cluster (8 GPUs per node = 64 GPUs total). `nvidia-smi` on each node shows all 8 GPUs. But `kubectl describe node` for each node shows `nvidia.com/gpu: 7` (only 7 GPUs allocatable). Where is the 8th GPU going?

- A) One GPU per node is reserved for the GPU Operator's device plugin
- B) One GPU per node is reserved for system processes or the MIG (Multi-Instance GPU) configuration. Check if MIG mode is enabled on GPU 0: `nvidia-smi -i 0 -q | grep "MIG Mode"`. Also check if the GPU Operator has a `DevicePlugin` config that reserves 1 GPU per node for monitoring (DCGM exporter runs on GPU 0 in some configs).
- C) The 8th GPU has a hardware fault — check nvidia-smi for errors
- D) Kubernetes can only allocate 7 GPUs per node regardless of how many are present

---

**Q19.** You need to delete a Managed Kubernetes cluster that has a node group with 8 running GPU nodes. The `nebius mk8s cluster delete` command fails with:

```
Error: cluster has active node groups, delete all node groups first
```

What is the correct deletion order?

- A) Use `--force` flag to bypass this check
- B) (1) Delete all node groups first: `nebius mk8s node-group delete --id $NODE_GROUP_ID`. (2) Wait for node group deletion to complete (`status: DELETED`). (3) Then delete the cluster: `nebius mk8s cluster delete --id $CLUSTER_ID`. Node groups must be fully deleted before the cluster can be removed.
- C) Drain all nodes first, then delete the cluster
- D) Delete the GPU VMs directly and the cluster deletion will succeed

---

**Q20.** Your company policy requires all GPU node groups to have `taints` set so only authorized ML workloads can run on GPU nodes. You forgot to add the taint when creating the node group. Can you add it now without recreating the node group?

- A) No — taints are set at node group creation and cannot be changed
- B) Yes — taints can be added to existing nodes with `kubectl taint nodes <node-name> <key>=<value>:<effect>`. For all nodes in the group: `kubectl taint nodes -l cloud.nebius.com/node-group-id=$NODE_GROUP_ID workload=ml:NoSchedule`. However, new nodes added to the group later won't have the taint unless you also update the node group's taint configuration in Nebius.
- C) No — you must recreate the node group with the taint
- D) Taints are applied at the pod level, not the node level

---

**Q21.** A data scientist says their training job is slow. You check and find the job is using 8 H100 GPUs on a single VM (no distributed training across nodes). GPU utilization is 40% on all 8 GPUs. CPU utilization is 100%. What is the bottleneck?

- A) The H100 GPUs are too fast for this workload — use L40S instead
- B) CPU is the bottleneck — data preprocessing or loading is not keeping up with GPU compute. Solutions: (1) Increase `num_workers` in the DataLoader to use more CPU cores for parallel data loading, (2) Pre-process and cache data before training, (3) Use NVIDIA DALI for GPU-accelerated data loading, (4) Move data closer to the VM (shared filesystem with higher IOPS).
- C) 40% GPU utilization is normal — the job is fine
- D) The 8 GPUs need to be split across 8 VMs for better performance

---

**Q22.** You're using Terraform to manage a Nebius GPU cluster. A teammate changes the `subnet_id` in the Terraform config and runs `terraform apply`. What happens?

- A) The cluster's subnet changes in-place — nodes migrate to the new subnet
- B) `subnet_id` is an immutable field. Changing it forces Terraform to destroy the existing cluster and create a new one in the new subnet. All running workloads are lost. This is a destructive change — always review `terraform plan` output before applying any changes to immutable fields.
- C) Terraform will warn but not actually change the subnet
- D) The subnet change applies to new nodes only — existing nodes stay in the old subnet

---

## Domain 3: Running Training & Inference Workloads (Q23–Q30)

**Q23.** Your PyTorch distributed training job uses `torchrun` with `--nproc_per_node=8 --nnodes=16`. The job starts but 4 random workers fail on startup with `Connection refused`. The other 12 nodes connect successfully. What is the most likely issue?

- A) 16 nodes is too many for torchrun — use Slurm instead
- B) The 4 failing workers can't reach the `MASTER_ADDR` (rendezvous node). Check: (1) Is `MASTER_ADDR` set to the correct hostname/IP? (2) Can those 4 nodes ping the master node? (3) Is `MASTER_PORT` blocked by a firewall or in use by another process? (4) Are those 4 nodes on a different subnet/security group? Start with `curl http://$MASTER_ADDR:$MASTER_PORT` from one of the failing nodes.
- C) Increase `NCCL_TIMEOUT` on the failing workers
- D) Set `NCCL_DEBUG=INFO` to see which nodes are failing

---

**Q24.** A Slurm job completes in 6 hours, but the last hour shows `NCCL AllReduce` latency spiking from 50ms to 800ms. The job completes successfully. GPU utilization stayed at 95%. What likely caused the latency spike in the final hour?

- A) InfiniBand congestion from other jobs — file a support ticket
- B) The final hour likely involves large checkpoint saves to the shared filesystem or Object Storage. Checkpoint writes compete for network bandwidth with InfiniBand all-reduce operations. The 800ms spike is the checkpoint write impacting NCCL bandwidth. Fix: use non-blocking async checkpoint writes or save checkpoints between training steps (not during).
- C) The Slurm scheduler is reclaiming resources after 5 hours
- D) NCCL is normally slower in the final hour as the model converges

---

**Q25.** You submit a Slurm job that runs a training script requiring `torch.distributed`. The job starts but immediately prints:

```
[rank0]: RuntimeError: address already in use
```

Then all ranks fail. What is happening?

- A) Multiple users are running jobs on the same nodes
- B) The default NCCL/Gloo init port (29500) is already in use on the master node. Either another job from a previous run didn't fully clean up, or another process claimed that port. Fix: set `MASTER_PORT` to a different port (e.g., `29501` or use a random available port: `MASTER_PORT=$(shuf -i 29500-29600 -n 1)`).
- C) `torch.distributed` is not installed on the Slurm nodes
- D) The job needs `--exclusive` flag to get dedicated nodes

---

**Q26.** Your inference endpoint is running a fine-tuned LLaMA-70B model on 4 × H200 GPUs using tensor parallelism. The endpoint's P99 latency is 3 seconds for a 1000-token response. Stakeholders want <1 second latency. What is the most impactful change?

- A) Increase to 8 × H200 GPUs — more GPUs = faster inference
- B) (1) Enable KV cache to avoid recomputing attention for cached prefixes. (2) Use speculative decoding with a smaller draft model to accelerate generation. (3) Quantize to FP8 on H200 (supported) — halves memory bandwidth requirements and increases throughput. (4) Enable continuous batching in vLLM to process multiple requests concurrently. Adding more GPUs helps throughput, not single-request latency.
- C) Reduce max sequence length from 1000 to 500 tokens
- D) Switch to H100 GPUs — they have better inference performance than H200

---

**Q27.** You're running 1000 hyperparameter search trials using Serverless AI Jobs (one trial = one job). After 500 trials, the billing shows $45,000. You expected $10,000 based on your estimate. What went wrong with your estimate and what should you check?

- A) Serverless AI Job billing has a 3× overhead — update your financial model
- B) Serverless AI Jobs bill per second from start until completion. If your trials are failing quickly (spending 5 minutes on startup overhead before 30 seconds of actual work), you're paying for overhead. Check: (1) actual job duration in billing vs. expected compute time, (2) whether jobs are failing and retrying (double billing), (3) whether you used a large GPU tier when a smaller one would suffice. Also check if jobs are hanging instead of failing fast.
- C) Serverless AI Job pricing changed — check the latest pricing page
- D) The 500 remaining trials will bring costs back in line

---

**Q28.** A researcher claims: "We should run our training on A100 GPUs in Nebius — they're proven for LLM training and cost-effective." How do you respond?

- A) Agree — A100 is the industry standard for LLM training
- B) Nebius does not offer A100 GPUs. The GPU portfolio includes H100, H200, B200, B300, L40S, and RTX PRO 6000. A100 is not and has never been available on Nebius. Redirect the researcher to H100 (closest equivalent) or H200 (higher memory for large models).
- C) A100 is available in some Nebius regions — check the latest availability list
- D) A100 was available previously but has been phased out — H100 is the replacement

---

**Q29.** A Slurm job using 16 nodes is in `CG` (Completing) state for 45 minutes. Other jobs are waiting. `squeue` shows it should have exited. What is the correct procedure?

- A) Wait — CG is normal and can take up to 1 hour
- B) `CG` means the job has finished but Slurm is waiting for all processes to clean up. After 45 minutes, this indicates a hung process. Run `scancel --signal=KILL <jobid>` to force-kill all processes. Then check the job's exit code with `sacct -j <jobid>` to understand what caused the hang. Common cause: a rank that crashed but left child processes (e.g., data loader workers) still running.
- C) Delete the entire job node group to free up the nodes
- D) Run `scontrol release <jobid>` to unstick it

---

**Q30.** You need to set `WORLD_SIZE` and `RANK` for a PyTorch distributed job running on 4 nodes with 8 GPUs each. What are the correct values for the process running on node 2 (0-indexed), GPU 5?

- A) WORLD_SIZE=4, RANK=5
- B) WORLD_SIZE = total processes = 4 nodes × 8 GPUs = 32. RANK = (node_index × GPUs_per_node) + local_GPU_index = (2 × 8) + 5 = 21. So WORLD_SIZE=32, RANK=21.
- C) WORLD_SIZE=8, RANK=21
- D) WORLD_SIZE=32, RANK=5

---

## Domain 4: Platform Automation & Maintenance (Q31–Q40)

**Q31.** You write this Terraform backend configuration for Nebius:

```hcl
terraform {
  backend "s3" {
    bucket     = "terraform-state"
    key        = "prod/gpu-cluster.tfstate"
    region     = "us-east-1"
    endpoint   = "https://storage.eu-north1.nebius.cloud/terraform-state"
    access_key = var.nebius_access_key
    secret_key = var.nebius_secret_key
  }
}
```

What is wrong with this configuration?

- A) The `region` should be `eu-north1` but this is cosmetic — Nebius ignores region for S3-compatible storage
- B) Two issues: (1) `endpoint` should be just `https://storage.eu-north1.nebius.cloud` — the bucket name should not be in the endpoint URL. (2) `access_key` and `secret_key` cannot use `var.*` in backend blocks — Terraform doesn't evaluate variables in backend configuration. Use environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) or a `.tfvars` file passed via `-backend-config` flag instead.
- C) The bucket name must match the `key` prefix
- D) Nebius Object Storage requires a different backend type — `s3` won't work

---

**Q32.** You run `nebius mk8s get-compatibility-matrix` and see that your cluster's Kubernetes version (1.31) is no longer in the supported matrix. The cluster continues to run normally. What is the risk and what action do you take?

- A) No risk — Kubernetes versions are supported indefinitely on Nebius
- B) An unsupported Kubernetes version means: (1) No security patches for K8s CVEs, (2) Nebius may not provide support for issues on that version, (3) New Nebius features may not be compatible. Since version is immutable, you must create a new cluster at a supported version and migrate workloads. Plan the migration before the version is deprecated.
- C) Run `nebius mk8s cluster update --version 1.33` to upgrade in-place
- D) Wait — Nebius will automatically migrate your cluster to a supported version

---

**Q33.** A Grafana dashboard shows this Prometheus query result for your GPU cluster:

```
DCGM_FI_DEV_GPU_UTIL{job="gpu-node-03"} = 0
DCGM_FI_DEV_SM_ACTIVE{job="gpu-node-03"} = 0
DCGM_FI_DEV_MEMORY_COPY_UTIL{job="gpu-node-03"} = 0
```

All three metrics are zero for 2 hours while a training job is supposedly running. What is the diagnosis?

- A) The GPU is idle between data loading phases
- B) All three metrics at zero for 2 hours with a running training job indicates the GPU is not actually doing any work. Possible causes: (1) The training job is stuck waiting (network deadlock, data loading bottleneck, waiting on a failed rank), (2) The pod is running but the training process crashed silently and the pod is stuck, (3) The DCGM exporter lost connection to the GPU. Check pod logs and `nvidia-smi` directly on the node.
- C) The DCGM exporter has a 2-hour reporting delay
- D) This is normal — GPU utilization drops to 0% during optimizer steps

---

**Q34.** You need to implement distributed tracing for a multi-service ML platform (data preprocessing → training → post-processing → model serving). A latency spike is occurring somewhere in the pipeline. Which Nebius Observability component helps you pinpoint the slow stage?

- A) Prometheus metrics — set up latency histograms per service
- B) Distributed traces (OpenTelemetry). Each request gets a trace ID that propagates through all services. The trace timeline shows: data preprocessing took 200ms, training request took 4500ms (the spike), post-processing took 100ms, serving took 50ms. This pinpoints the training stage without guessing. Use Nebius Observability's tracing backend to store and visualize traces.
- C) Audit Logs — they capture all service-to-service calls
- D) GPU metrics — the slow stage will show lower GPU utilization

---

**Q35.** Your team uses `nebius compute` CLI to create GPU VMs. A new engineer accidentally runs:

```bash
nebius compute instance delete --id instance-prod-001
```

This deletes a production training VM with a locally-stored 500 GB checkpoint. What Nebius feature could have prevented data loss (not the deletion itself, but the data loss)?

- A) Enable `prevent_destroy` in Terraform for the VM
- B) Attach the checkpoint data to a Network SSD disk (instead of the VM's local storage), and the disk should have been created as a separate resource with `prevent_destroy`. When the VM is deleted, a detached disk persists. Alternatively, write checkpoints to the shared filesystem or Object Storage — these survive VM deletion by design.
- C) Take VM snapshots every hour
- D) The `--dry-run` flag should have been used first

---

**Q36.** You write a monitoring alert: `avg(gpu_utilization) > 90 for 5 minutes`. This alert fires 50 times per day, all false positives. Training jobs legitimately use 95% GPU. How do you fix the alert without disabling GPU monitoring?

- A) Increase the threshold to 99%
- B) Change the condition: training jobs at 95% utilization are expected and healthy — the alert should fire only when GPU utilization is unexpectedly HIGH (e.g., a job that should be idle is consuming GPU, indicating a runaway process) or unexpectedly LOW (idle expensive GPUs). Fix: alert on `gpu_utilization < 50 for 30 minutes` during expected training hours, or alert on a different signal like `gpu_temperature > 88°C` for thermal issues.
- C) Add a silence rule during business hours
- D) Reduce the evaluation window to 1 minute

---

**Q37.** After a network outage, your Terraform state shows resources as existing, but they don't actually exist in Nebius (they were deleted during the outage). Running `terraform plan` shows no changes (Terraform thinks everything is fine). Why does this happen and how do you fix it?

- A) Terraform always re-checks resource existence on plan — this can't happen
- B) Terraform's state file records what Terraform last applied. If resources are deleted outside of Terraform (outage, manual deletion), the state file doesn't know. `terraform plan` compares config to state, not state to actual cloud. Run `terraform refresh` to sync state with actual Nebius resource state, then `terraform plan` will show the missing resources and `terraform apply` will recreate them.
- C) The outage corrupted the state file — restore from backup
- D) Use `terraform import` to re-import all resources

---

**Q38.** You have 200 GPU VMs managed by Terraform across 5 node groups. A `terraform plan` now shows a 3-minute runtime for just the planning phase. How do you speed up Terraform operations?

- A) Split the state file into smaller files manually
- B) Use Terraform workspaces to separate state files per node group (or per environment). Each workspace has its own state and plans only its resources. Alternatively, use Terraform modules with separate backends per logical group. Fewer resources per state file = faster plans. Also consider using `-target` for targeted changes instead of planning the entire 200-VM infrastructure each time.
- C) Increase the Terraform provider parallelism with `-parallelism=50`
- D) Use `terraform plan -refresh=false` to skip the state refresh

---

**Q39.** A GPU cluster node shows this in `dmesg`:

```
NVRM: Xid (PCI:0000:03:00): 79, pid='<unknown>', name=<unknown>, 
GPU-PCI id: 0000:03:00.0 
NVRM: XID 79: GPU-Reset Required.
```

What does Xid 79 mean and what do you do?

- A) Xid 79 is a driver error — restart the NVIDIA driver
- B) Xid 79 = GPU reset required due to an unrecoverable GPU error (typically ECC double-bit or unrecoverable fault). This GPU will not recover without a reset, and a reset requires the node to be rebooted. Actions: (1) Cordon the node immediately, (2) Drain all pods, (3) Reboot the node to reset the GPU, (4) If Xid 79 recurs after reboot, open a Nebius support ticket — the GPU may be failing.
- C) Xid 79 is a network error — check InfiniBand
- D) Xid 79 is informational — the GPU self-recovered

---

**Q40.** You need to run `nebius mk8s node-group update` to change a node group's labels. The command requires `--id`. You have the cluster name (`gpu-cluster-prod`) but not the node group ID. What is the fastest way to get the node group ID?

- A) Look it up in the Nebius console
- B) Run `nebius mk8s node-group list --parent-id $(nebius mk8s cluster get --name gpu-cluster-prod --format json | jq -r '.id')`. This chains two CLI commands: get the cluster ID by name, then list node groups in that cluster. Alternatively: `kubectl get nodes -o jsonpath='{.items[0].metadata.labels.cloud\.nebius\.com/node-group-id}'` if you have kubectl access.
- C) The node group ID is always the cluster ID with `-ng` suffix
- D) The node group name is used instead of ID: `--name gpu-workers`

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Secrets cached at startup without a refresh path fail on rotation. Services must either re-fetch on auth failure or use short-lived credentials that force periodic refresh. Never assume a cached credential remains valid. |
| 2 | **B** | Data residency violations can occur even before data moves — creating processing infrastructure in the wrong jurisdiction is itself a compliance violation in regulated environments. Immediate remediation and documentation are required. |
| 3 | **B** | `auditors` group: read-only control plane access, no data plane access. This is explicitly designed for compliance auditors. `viewers` can access actual data — too broad for a third-party auditing firm. |
| 4 | **B** | Shared admin SAs = blast radius of any compromise = all 50 VMs. Purpose-specific SAs with minimum permissions limit the damage from any single SA compromise to only the workloads that use it. |
| 5 | **B** | Audit Logs are forensic evidence, not a security risk. Disabling them makes incident investigation impossible. Restrict access (admins only), export to SIEM for additional protection, but never disable. |
| 6 | **B** | Audit Logs are the authoritative source for "who created this resource and when." Filter by project, resource type `compute.instance`, action `create`, and date range to find exactly who created those 200 VMs. |
| 7 | **B** | Base64 is encoding, not encryption. Anyone with `kubectl get secret -o yaml` can decode it in one command. MysteryBox + runtime secret injection is the correct approach. Etcd encryption protects data at rest but not against authorized K8s API access. |
| 8 | **B** | Immediate stop to halt billing + cleanup of accumulated objects + lifecycle policy to prevent recurrence + `NCCL_DEBUG=WARN` to reduce future log volume. This is the complete remediation. |
| 9 | **B** | B300 = `uk-south1-a`. B200 = `us-central1-b`, `me-west1-a`. Matching the correct GPU type to the correct region/fabric is critical — wrong region means provisioning failure. |
| 10 | **B** | 45 GB/s is consistent with 400Gbps Ethernet, not InfiniBand. If NCCL is using the wrong interface (`NCCL_SOCKET_IFNAME` set to eth0), it bypasses IB entirely. `NCCL_DEBUG=INFO` reveals which transport is actually selected. |
| 11 | **B** | `nebius mk8s node-group update --fixed-node-count 16` resizes the existing group. Managed Kubernetes provisions the new nodes automatically. No need to recreate the group — scaling is a supported update operation. |
| 12 | **B** | `NoSchedule` taints block all pods without matching tolerations. If all nodes are tainted `gpu-maintenance:NoSchedule` and no pods have that toleration, no pods can schedule. Remove the taint if maintenance is complete. |
| 13 | **B** | GPUDirect RDMA = GPU-to-NIC direct path. Requires: compatible HCAs (Mellanox/ConnectX), NVIDIA Network Operator (configures drivers + rdma device plugin), and pods requesting the RDMA resource. All H100 Nebius nodes have compatible hardware. |
| 14 | **B** | In synchronous distributed training, the slowest node sets the pace for all others. A 30% throttled node on 1/32 = cluster runs at 70% of its potential. Checkpoint and replace to restore full performance. |
| 15 | **B** | The 8 GiB/s is per client, not per filesystem. Multiple concurrent writers share this bandwidth. VM network limits can also cap throughput below the filesystem spec. Always benchmark with realistic concurrent access patterns. |
| 16 | **B** | Private container registry requires authentication. The node's containerd config or a Kubernetes image pull secret must include credentials for `cr.eu-north1.nebius.cloud`. Nebius uses Object Storage access key credentials for registry auth. |
| 17 | **B** | Capacity blocks (reservations) guarantee GPU availability ahead of time. Pre-provisioned nodes are billed at full rate. For training campaigns with strict deadlines, the cost of guaranteed availability is justified. |
| 18 | **B** | GPU Operator may configure DCGM to run on GPU 0, reserving it for monitoring. Check DCGM exporter configuration in the GPU Operator's DevicePlugin config. MIG mode can also make some GPUs appear differently. |
| 19 | **B** | Nebius requires node groups to be deleted before the cluster. The order is: delete node groups → wait for DELETED status → delete cluster. There is no `--force` bypass for this safety check. |
| 20 | **B** | Taints can be added to existing nodes with `kubectl taint`. However, Nebius node group taint config controls taints on newly provisioned nodes. Apply both to ensure consistency — existing nodes via kubectl, future nodes via node group config. |
| 21 | **B** | CPU at 100% with GPU at 40% = clear CPU bottleneck in data loading pipeline. The GPU is waiting for data. Increasing DataLoader `num_workers`, caching preprocessed data, or using GPU-accelerated data pipelines (DALI) resolves this. |
| 22 | **B** | `subnet_id` is immutable on Nebius clusters. Terraform will plan a destroy-and-recreate, which is destructive. Always run `terraform plan` and look for `[forces replacement]` on immutable field changes before applying. |
| 23 | **B** | `Connection refused` from specific workers points to network connectivity to the rendezvous node. The 4 failing nodes can't reach `MASTER_ADDR:MASTER_PORT`. Debug with connectivity tests from those specific nodes. |
| 24 | **B** | Checkpoint writes compete with training I/O on the same network path. NCCL all-reduce and checkpoint network writes share bandwidth. Solution: async checkpoint writes or explicitly scheduling checkpoints between training steps. |
| 25 | **B** | Port conflict from a previous uncleaned job. The distributed init process tries to bind to port 29500 and finds it already in use. Randomize the port per job to avoid this: `MASTER_PORT=$(shuf -i 29500-29600 -n 1)`. |
| 26 | **B** | For generation latency (not just throughput): KV cache avoids recomputing attended tokens, speculative decoding generates multiple tokens per step, FP8 on H200 doubles throughput, continuous batching improves GPU utilization. Adding GPUs improves throughput not single-request latency. |
| 27 | **B** | Serverless billing starts at job start, not compute start. If startup overhead (image pull, initialization) is 5 minutes and actual compute is 30 seconds, you're paying 10× what you expected. Profile actual vs. billed duration. |
| 28 | **B** | A100 has never been available on Nebius. This is critical exam knowledge — the portfolio is H100/H200/B200/B300/L40S/RTX PRO 6000. Any question suggesting A100 on Nebius is a distractor. |
| 29 | **B** | `CG` for 45 minutes = hung cleanup processes. `scancel --signal=KILL` force-kills everything. Then diagnose with `sacct` for the root cause. Data loader worker processes are a common culprit for zombie CG states. |
| 30 | **B** | WORLD_SIZE = total GPU processes = nodes × GPUs per node = 4 × 8 = 32. RANK = (node_index × GPUs_per_node) + local_rank = (2 × 8) + 5 = 21. This is fundamental PyTorch distributed training math. |
| 31 | **B** | Backend blocks in Terraform don't support variable references — only literal strings. Use environment variables for sensitive credentials. The endpoint URL format should also not include the bucket name (it's specified separately in `bucket`). |
| 32 | **B** | Unsupported K8s version = no patches, no support, potential incompatibility with new Nebius features. Since version is immutable, migration to a new cluster is the only path. Plan this before end-of-support dates. |
| 33 | **B** | All three GPU metrics at zero for 2 hours during training = GPU completely idle. This is not normal behavior during training. The process is stuck (deadlock, waiting on a failed peer, or silently crashed). Check pod logs immediately. |
| 34 | **B** | Distributed traces with OpenTelemetry are the purpose-built tool for latency attribution across microservices. Metrics give aggregate data; traces give per-request timing breakdown showing exactly which stage is slow. |
| 35 | **B** | Data on attached Network SSD disks persists after VM deletion (if created as a separate resource). Always store checkpoints on separate disks, shared filesystem, or Object Storage — never on VM local/ephemeral storage. |
| 36 | **B** | High GPU utilization during training is expected and healthy — don't alert on it. Alert on anomalies: too-low utilization (idle expensive GPUs) or temperature/ECC issues. Rethink what "problem" you're trying to detect. |
| 37 | **B** | `terraform refresh` syncs the state file with actual cloud resource state. After a refresh, `plan` will show the drift (resources missing in Nebius but present in state). Then `apply` recreates them. |
| 38 | **B** | Large state files = slow Terraform. Split by logical boundaries (node group, environment) with separate backends. `terraform workspace` or separate directories with separate `backend` configs are the production patterns. |
| 39 | **B** | Xid 79 is an NVIDIA GPU fatal error requiring a reset. The GPU cannot continue without a reboot. Cordon + drain to protect running jobs, then reboot. Recurring Xid 79 after reboot = hardware failure, escalate to Nebius. |
| 40 | **B** | Chain `nebius mk8s cluster get` (by name) to get cluster ID, then `nebius mk8s node-group list` to get node group IDs. This is the CLI-native approach. Alternatively use `kubectl` to read the node group ID from node labels. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Exam ready |
| 30–35 | Close — focus on weak domains |
| 24–29 | More practice needed |
| <24 | Re-read notes with focus on applying knowledge |
