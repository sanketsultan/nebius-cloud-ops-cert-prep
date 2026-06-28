# Nebius AI Cloud Ops — Mock Exam 1 (Scenario-Based)

**Format:** Every question is a scenario. Pick the best action or answer.
**Time:** ~55 minutes | **Pass mark:** 70% (28/40)

> Attempt all questions before checking the answer key at the bottom.

---

## Domain 1: Security, Compliance & Billing (Q1–Q8)

**Q1.** Your ML platform team has three sub-groups: data engineers who need to upload datasets to Object Storage, model trainers who need to start/stop GPU VMs, and auditors from the compliance team who need to review what resources exist. You need to assign the minimum required access to each group. Which mapping is correct?

- A) Data engineers → editors, trainers → editors, compliance → viewers
- B) Data engineers → viewers, trainers → editors, compliance → auditors
- C) Data engineers → editors, trainers → editors, compliance → auditors
- D) All three → editors, since compliance auditors also need data access

---

**Q2.** A Terraform pipeline has been running fine for months using a service account with an access key. Suddenly it starts failing with `403 Forbidden` when trying to create new VMs. The service account still exists. What is the most likely cause?

- A) The Nebius region went down
- B) The access key was deleted or rotated, and the pipeline is using the old credentials
- C) The service account was removed from its IAM group
- D) Both B and C are equally likely — check both

---

**Q3.** A junior engineer asks: "I created a service account and added it to the editors group at tenant level. But it still can't create VMs in our project. Why?" What is the correct explanation?

- A) Editors cannot create VMs — they need admins access
- B) Service accounts need to be in the editors group at project level, not tenant level, to manage project resources
- C) The service account also needs an authorized key to use IAM
- D) VM creation requires the service account to have an SSH key registered

---

**Q4.** Your training script fetches a third-party API key from an environment variable `API_KEY` set in a Kubernetes ConfigMap. After a key rotation, you update the ConfigMap but the running pods still use the old key and fail. You need a solution where key rotation takes effect without pod restarts. What do you implement?

- A) Use a Kubernetes Secret instead of ConfigMap
- B) Store the key in MysteryBox, update the primary version, and have the script fetch it from MysteryBox API at each run — the new version takes effect immediately without restarts
- C) Set the environment variable directly on the node
- D) Use a Kubernetes init container to fetch the key on pod start

---

**Q5.** Your security team asks you to prove that a specific VM (`vm-abc123`) was deleted last Tuesday at 14:00 UTC and identify who deleted it. Which Nebius service provides this, and what do you look for?

- A) Observability Logs — filter by VM ID and time
- B) Monitoring Metrics — check for a drop in VM count around that time
- C) Audit Logs — filter by `eventType: DeleteInstance` and `resource.id: vm-abc123`, look at `subject` field for who did it
- D) Billing history — a VM deletion shows as a billing event

---

**Q6.** A budget alert fires at $8,000 (80% of your $10,000 monthly budget). Your team has 16 H100 VMs running a time-critical training job. A manager says "shut down 8 VMs to stay under budget." A junior engineer says "the alert already paused 8 VMs automatically." Who is correct?

- A) The junior engineer — Nebius pauses the oldest VMs when the budget threshold is hit
- B) The manager — budget alerts are notifications only; no VMs were paused. You must manually stop them.
- C) Both are wrong — Nebius stops all VMs when the budget is exceeded
- D) The junior engineer — spot VMs are paused but on-demand VMs continue

---

**Q7.** You are setting up encryption for a new project. The security policy says: "All data must be encrypted. Customer-controlled keys must be used for sensitive disks." You have two disk types: Network SSD for boot disks and Network SSD NRD for worker caches. What do you need to do?

- A) Enable encryption on both disk types — neither is encrypted by default
- B) Network SSD is already encrypted by default. For NRD, explicitly enable encryption at disk creation and configure KMS keys for sensitive disks.
- C) Attach a KMS key to both disk types — Nebius doesn't encrypt anything by default
- D) Only boot disks can be encrypted on Nebius

---

**Q8.** An engineer ran `nebius iam service-account create --name etl-pipeline` and created an access key. They configured the key in their AWS CLI profile and ran `aws s3 ls s3://data-bucket --endpoint-url https://storage.eu-north1.nebius.cloud`. They get `Access Denied`. `aws configure list` shows the key is set correctly. What is missing?

- A) The endpoint URL format is wrong
- B) The service account was never added to an IAM group — it has no permissions at all
- C) `aws s3 ls` does not work with Nebius Object Storage
- D) The bucket name must include the project ID as a prefix

---

## Domain 2: Setting Up & Operating GPU Clusters (Q9–Q22)

**Q9.** You are provisioning an 8-node H200 training cluster in `eu-north1`. You create the GPU cluster with `--infiniband-fabric fabric-5`. When you try to launch H200 VMs into this cluster, they fail to join. Your colleague suggests using `fabric-7` instead. Who is right and why?

- A) You are right — `fabric-5` supports H200 GPUs
- B) Your colleague is right — `fabric-5` is for H200 in `eu-west1`. H200 in `eu-north1` uses `fabric-7`.
- C) Both fabrics support H200 anywhere — the issue is elsewhere
- D) Your colleague is wrong — `fabric-7` is for B200 GPUs

---

**Q10.** After deploying a Kubernetes GPU node group with H100s, you run `kubectl get nodes` — all 8 nodes are `Ready`. You submit a test job requesting `nvidia.com/gpu: 1` and it stays `Pending`. You run `kubectl describe pod <pod>` and see `0/8 nodes are available: 8 Insufficient nvidia.com/gpu`. What is the most likely cause?

- A) The nodes don't have enough CPU for the pod
- B) The NVIDIA GPU Operator is not installed — without it, the device plugin doesn't run and GPUs aren't advertised to the Kubernetes scheduler
- C) The pod needs to be in the `nvidia-gpu-operator` namespace
- D) The node group needs to be in a GPU cluster with InfiniBand

---

**Q11.** A running 16-node training job suddenly loses communication between nodes. You SSH to node-07 and run `ibstat`. You see all ports in `Active` state. You run `ibcheckerrors` and see high error counts on `mlx5_2`. What is your next step?

- A) Reboot node-07 — high error counts clear on reboot
- B) The IB port `mlx5_2` has physical link errors. Drain node-07, move the job to the remaining 15 nodes, and open a Nebius support ticket for hardware inspection.
- C) Reinstall the NVIDIA Network Operator on node-07
- D) Increase `NCCL_IB_TIMEOUT` in the training script

---

**Q12.** Your team has a 4-node H100 GPU cluster in `eu-north1`. Management wants to move it to `us-central1` to reduce latency for US-based users. What is the correct approach?

- A) Use `nebius compute gpu-cluster update --region us-central1`
- B) GPU clusters are region-locked — you must create a new cluster in `us-central1` and recreate all node VMs there
- C) Use Terraform's `moved` block to migrate the cluster
- D) Clone the existing cluster to `us-central1` via the console

---

**Q13.** You need to create 4 GPU nodes for a Kubernetes cluster. Each node needs 8 B200 GPUs and InfiniBand connectivity. You create the node group but don't install any operators (the nodes use Nebius boot disk images). After the nodes come up, InfiniBand is not working. What did you miss?

- A) Nothing — Nebius boot disk images include everything needed
- B) You need to install the NVIDIA Network Operator from the Nebius chart repo — it is required for B200 GPUs even with Nebius boot disk images
- C) You need to install the NVIDIA GPU Operator
- D) You need to manually configure the InfiniBand P-Key

---

**Q14.** A colleague says: "Two of our training clusters share the same `fabric-7` fabric. I'm worried their InfiniBand traffic can interfere with each other and affect performance." How do you respond?

- A) "You're right — we should move one cluster to `fabric-2` to isolate them"
- B) "Nebius assigns a unique Partition Key (P-Key) to each cluster. Even on the same physical fabric, clusters are completely isolated — they cannot see or interfere with each other's IB traffic"
- C) "IB bandwidth is shared equally between clusters on the same fabric — it could halve your throughput"
- D) "You're right — use separate fabrics per cluster to guarantee isolation"

---

**Q15.** You want to upgrade the CUDA driver from `cuda12.8` to `cuda13.0` on a running node group that has active training jobs. What actually happens when you run the upgrade command, and what should you do first?

- A) The driver upgrades in-place with no disruption to running jobs. No preparation needed.
- B) Managed Kubernetes will cordon, drain, and recreate all nodes. First drain the nodes yourself and ensure all jobs have saved checkpoints, then run the upgrade.
- C) Only new nodes get the new driver. Existing nodes are unaffected.
- D) The upgrade queues for the next maintenance window automatically

---

**Q16.** You run `kubectl drain gpu-node-05 --ignore-daemonsets --delete-emptydir-data` and it hangs for 10 minutes on evicting one pod. `kubectl describe pod <pod>` shows it has a PodDisruptionBudget. Training is already failing on this node. What is the fastest safe resolution?

- A) Kill the pod with `kubectl delete pod --grace-period=0 --force`
- B) Wait — the PDB will eventually allow eviction when other replicas stabilize
- C) Remove the PDB temporarily with `kubectl delete pdb <pdb-name>`, drain the node, then recreate the PDB
- D) Cordon the node instead and restart the whole training job

---

**Q17.** Your storage team proposes using Network SSD NRD disks for a new Slurm controller VM that stores job logs and accounting data. The controller must be highly available and data must survive disk failures. Should you use NRD for this?

- A) Yes — NRD has the highest IOPS and is ideal for write-heavy workloads
- B) No — NRD has no redundancy. If the disk fails, all job logs and accounting data are lost permanently. Use Network SSD (standard, with erasure coding) or IO M3 instead.
- C) Yes — NRD disks are automatically replicated within the zone
- D) Yes, but only if you enable encryption at creation time

---

**Q18.** A researcher wants to access a shared training dataset from 20 GPU nodes simultaneously. The dataset is 8 TB. She suggests copying it to a Network SSD NRD disk on the primary node and sharing it over NFS. What is the better Nebius-native approach?

- A) Her approach is correct — NRD on the primary node is the fastest option
- B) Use a Nebius Shared Filesystem — it supports concurrent multi-VM access natively, mounts as virtiofs, and scales to 5 PiB without manual NFS setup
- C) Use Object Storage with boto3 on each node
- D) Use a Network SSD IO M3 disk and share via Samba

---

**Q19.** You need to create a Managed Kubernetes cluster for GPU workloads. The cluster's control plane should not be accessible from the public internet — only from VMs within the same subnet. Additionally, you want 3 etcd stores for HA. Which create command is correct?

- A) `nebius mk8s cluster create --name gpu-k8s --control-plane-endpoints-public-endpoint=false --control-plane-etcd-cluster-size 3`
- B) `nebius mk8s cluster create --name gpu-k8s --private=true --etcd=3`
- C) `nebius mk8s cluster create --name gpu-k8s --internal-only --ha-enabled`
- D) `nebius mk8s cluster create --name gpu-k8s` (defaults are sufficient)

---

**Q20.** You have an existing H100 Kubernetes node group and want to check if `cuda13.0` driver preset is supported with Kubernetes 1.33 before upgrading. Which command gives you this information?

- A) `nebius compute platform list`
- B) `nebius mk8s node-group get-compatibility-matrix --cluster-kubernetes-version 1.33 --platform gpu-h100-sxm`
- C) `kubectl get nodes -o wide`
- D) `nebius mk8s cluster describe --id <id> --show-gpu-options`

---

**Q21.** A training job finishes but results are inconsistent. You suspect one of the 8 nodes had GPU errors during training. How do you check for GPU hardware errors after the fact?

- A) Check `kubectl logs` for the training pod
- B) SSH to each node and run `nvidia-smi -q | grep -i ecc` to check ECC error counts, and `dmesg | grep -i nvidia` for hardware-level GPU errors
- C) Check Monitoring Metrics for `gpu_utilization` drops
- D) Check Audit Logs for GPU-related events

---

**Q22.** Your Kubernetes cluster has `etcd_cluster_size = 1` (HA disabled). During a routine maintenance on the single etcd node, the cluster control plane becomes unavailable for 45 minutes. What should have been configured to prevent this?

- A) A second control plane node in a different zone
- B) `etcd_cluster_size = 3` (HA enabled — the default) — 3 etcd stores means the cluster survives one etcd node failure without downtime
- C) An external etcd cluster on separate VMs
- D) Automated etcd backups to Object Storage

---

## Domain 3: Running Training & Inference Workloads (Q23–Q30)

**Q23.** A 72-hour LLM pre-training job is running on 32 H200 nodes. At hour 58, node-14 crashes due to a hardware fault. The job fails and must restart from the beginning. What should have been in place to resume from hour 58 instead?

- A) The job should have used spot instances with automatic restart
- B) Checkpoint-and-resume — save model weights, optimizer state, and step number to a shared filesystem every N steps. On node failure, restart from the last checkpoint instead of from scratch.
- C) Use Serverless AI Jobs instead of Kubernetes — they handle node failures automatically
- D) Run the job on a single large node to avoid multi-node failures

---

**Q24.** You deploy a fine-tuned LLM as a Serverless AI Endpoint. Traffic is minimal overnight (5 req/hr) but peaks at 2,000 req/hr during business hours. Your team expects to pay almost nothing overnight. After seeing the bill, they're surprised they were charged for overnight hours. Why?

- A) Serverless AI Endpoints scale to zero automatically after 30 minutes of inactivity
- B) Serverless AI Endpoints run continuously until explicitly stopped — they are billed while running regardless of traffic volume
- C) The 5 req/hr overnight kept the endpoint "active" and prevented scale-to-zero
- D) There is a minimum billing period of 24 hours for Serverless AI Endpoints

---

**Q25.** A Slurm job requesting 8 nodes has been `PD` for 2 hours. `squeue -j 2301` shows `Reason: ReqNodeNotAvail`. `sinfo` shows:

```
PARTITION  NODES  STATE  NODELIST
gpu        4      idle   node[01-04]
gpu        4      drain  node[05-08]
```

What is happening and what do you do?

- A) Submit the job with `--nodes=4` to use the available idle nodes
- B) 4 nodes are idle but 4 are drained. The job needs 8 but only 4 are available. Undrain node[05-08] if they're healthy (`scontrol update nodename=node[05-08] state=resume`), then the job can start.
- C) Cancel and resubmit — PD jobs older than 2 hours are automatically failed
- D) The partition does not support 8-node jobs — check partition limits with `sinfo -p gpu`

---

**Q26.** Your 4-node PyTorch distributed training job starts but hangs indefinitely at initialization. No error is shown. You check `NCCL_DEBUG=INFO` logs and see:

```
NCCL INFO Bootstrap: Using interface eth0:10.0.0.1
NCCL INFO NET/IB: No device found.
```

The nodes have InfiniBand hardware. What is the root cause and fix?

- A) `MASTER_ADDR` is set to the wrong IP — update it to the IB interface IP
- B) NCCL cannot find the InfiniBand device. Check that `NCCL_IB_DISABLE=0`, the Network Operator is installed and its `NICClusterPolicy` state is "ready", and verify `ibstat` shows Active ports.
- C) `WORLD_SIZE` is wrong — set it to match the number of nodes, not GPUs
- D) The training script is missing `torch.distributed.init_process_group(backend="nccl")`

---

**Q27.** Your team needs MLflow for experiment tracking. A team member in `eu-north2` says MLflow doesn't appear in their console. Another team member in `eu-north1` can see it fine. What do you tell the first team member?

- A) MLflow requires a Soperator cluster to be running first — create one in `eu-north2`
- B) MLflow Managed Service is not available in `eu-north2` — use `eu-north1` or another supported region. This is a documented regional limitation.
- C) They need to be added to the `editors` group to see MLflow in the console
- D) MLflow is in preview and requires a support request to enable per region

---

**Q28.** You are running inference using vLLM on a Serverless AI Endpoint. Response latency is consistently 8 seconds for a simple prompt. You check the endpoint metrics and see `gpu_memory_used = 79.8 GB / 80 GB` (99.75%). GPU compute utilization is 15%. What is happening?

- A) The model is CPU-bound — move to a larger VM
- B) VRAM is nearly full, causing constant memory swapping between GPU and CPU. The low compute utilization is caused by the GPU stalling on memory operations. Reduce max model sequence length, use a smaller model, or move to H200 with 141 GB VRAM.
- C) InfiniBand is saturated on the inference node
- D) The endpoint needs more replicas — add horizontal scaling

---

**Q29.** A researcher wants to run 100 hyperparameter sweep jobs, each needing 1 GPU for 30 minutes. They ask whether to use a dedicated 8-GPU Kubernetes cluster (always-on) or Serverless AI Jobs. What do you recommend and why?

- A) Dedicated K8s cluster — faster startup time makes it more efficient for 100 jobs
- B) Serverless AI Jobs — no persistent infra, billed per second of actual compute. For 100 × 30-minute jobs, you pay only for 50 GPU-hours. A dedicated cluster charges for all 8 GPUs × 24 hours × days running, even when idle between jobs.
- C) Managed Service for Soperator — Slurm's queue management is best for many short jobs
- D) A dedicated K8s cluster with cluster autoscaler to scale to zero between jobs

---

**Q30.** Your training script reads a 5 TB dataset from Object Storage on every epoch. 10 epochs takes 6 hours. Your colleague says it should take 2 hours based on GPU compute alone. The extra 4 hours are being lost to data loading. What is the most effective fix?

- A) Increase the VM's network bandwidth by using a larger instance type
- B) Before training starts, copy the full 5 TB dataset to a Nebius Shared Filesystem. Read all epochs from there — the shared filesystem has up to 12 GiB/s read bandwidth vs. Object Storage's per-request HTTP overhead.
- C) Use multiple Object Storage buckets in parallel to increase throughput
- D) Enable S3 Transfer Acceleration on the Object Storage bucket

---

## Domain 4: Platform Automation & Maintenance (Q31–Q40)

**Q31.** Your Terraform config manages a GPU cluster. A colleague changes `infiniband_fabric` from `fabric-7` to `fabric-2` in the Terraform resource to fix a mistake. Before they run `terraform apply`, what do you warn them about?

- A) The fabric field is case-sensitive — make sure it's lowercase
- B) `infiniband_fabric` is immutable — changing it will destroy the existing cluster and create a new one, taking down all running training jobs. Plan a maintenance window and ensure jobs are checkpointed first.
- C) The fabric change will only apply to new nodes, not existing ones
- D) Terraform cannot change InfiniBand fabrics — do it in the console

---

**Q32.** You have this Terraform backend config:

```hcl
backend "s3" {
  bucket   = "nebius-tf-state"
  key      = "prod/terraform.tfstate"
  endpoint = "https://storage.eu-north1.nebius.cloud"
  region   = "eu-north1"
}
```

Two engineers both run `terraform apply` at the same time. Engineer A finishes first. Engineer B's apply also succeeds without error. Later, you discover that some resources are missing. What went wrong?

- A) The S3 backend does not support Nebius Object Storage
- B) State locking was not configured — both applies ran on the same state file simultaneously, causing corruption. One engineer's changes were overwritten. Add DynamoDB or equivalent locking.
- C) Both engineers used the same `key` path — use separate state files per engineer
- D) The `endpoint` field is wrong

---

**Q33.** You run `terraform plan` and see:

```
# nebius_mk8s_v1_cluster.prod must be replaced
-/+ resource "nebius_mk8s_v1_cluster" "prod" {
    ~ name              = "prod-gpu" -> "prod-gpu-v2"
    # (forces new resource)
}
```

But you only changed the `name`. Why is the cluster being replaced?

- A) Cluster names are immutable on Nebius — any name change forces replacement
- B) The `name` field itself forces replacement — use labels for mutable identifiers instead
- C) Something else in the config also changed — inspect the full diff with `terraform plan -out=plan.tfplan && terraform show plan.tfplan`
- D) This is a Terraform provider bug — ignore the replacement warning

---

**Q34.** At 3am, your on-call dashboard shows `gpu_utilization` dropped from 97% to 0% across all nodes of a training cluster. No job failure alerts fired. What is the first thing you check and why?

- A) Run `ibstat` on all nodes — zero utilization means InfiniBand failed
- B) Check `squeue` (Slurm) or `kubectl get pods` — zero GPU utilization most likely means the job completed normally or was cancelled, not a hardware failure. Check job status before assuming infrastructure failure.
- C) Check Audit Logs for any DeleteInstance events
- D) Restart the GPU Operator on all nodes

---

**Q35.** A node group update command recreated all nodes and now `kubectl describe node gpu-node-01 | grep nvidia` shows `nvidia.com/gpu: 0` on the new nodes, even though `nvidia-smi` works fine. What is happening?

- A) The GPU Operator was not reinstalled after node recreation — install it again
- B) The driver daemonset pod is still installing. Check: `kubectl logs -n nvidia-gpu-operator $(kubectl get pods -n nvidia-gpu-operator | grep nvidia-driver-daemonset | head -1 | awk '{print $1}') --tail 1`. Wait for "Done, now waiting for signal".
- C) The new nodes are in a different InfiniBand fabric
- D) The node group preset is wrong — recreate with the correct GPU platform

---

**Q36.** Your team manages 12 different Nebius projects with Terraform. Each project has its own GPU cluster, node groups, and IAM setup. A new engineer asks if they should keep all projects in one Terraform root module. What do you advise?

- A) Yes — one root module is easiest to manage and ensures consistent state
- B) No — use separate Terraform modules per project with separate state files in Object Storage. This isolates blast radius (one broken apply doesn't affect other projects) and keeps state files manageable.
- C) Yes — but use Terraform workspaces to separate environments within one module
- D) No — use a monorepo with one state file but separate directories per project

---

**Q37.** You receive this alert: `gpu_temperature > 90°C` on `node-03`. You check `nvidia-smi` and confirm the temperature. Training is still running. What is the immediate risk and action?

- A) High temperature is normal for H100s under load — no action needed
- B) At 90°C+, the GPU may thermal throttle (reduce clock speed), slowing training. Check airflow/cooling in the data center (Nebius responsibility) by opening a support ticket. In the meantime, monitor if compute throughput drops — if it does, drain and replace the node.
- C) Immediately terminate all jobs on node-03 and reboot it
- D) Increase the VRAM allocation to reduce GPU compute load

---

**Q38.** You run a rolling upgrade: cordon node-01, drain node-01, upgrade, uncordon. You then do the same for all 8 nodes in parallel to save time. The next morning the training job has failed with all nodes in `NotReady`. What was the mistake?

- A) You forgot to reinstall the GPU Operator after upgrading
- B) Draining all nodes simultaneously removed all available capacity. Jobs had nowhere to run. Rolling upgrades must be done one node at a time — cordon, drain, upgrade, uncordon, verify, then move to the next.
- C) The upgrade command requires a maintenance window flag
- D) You should have used `kubectl delete node` instead of drain

---

**Q39.** You open Nebius Observability to debug why your inference API has a p99 latency of 12 seconds. Logs show no errors. Metrics show GPU at 40% and memory at 60%. What Observability tool gives you the most direct answer?

- A) Set up a new Monitoring alert for latency
- B) Use Distributed Traces — a trace of a slow request will show exactly which service/span in the request chain is consuming the 12 seconds, even when logs show no errors
- C) Filter logs by response time > 5s
- D) Compare GPU utilization dashboards across nodes

---

**Q40.** Your team's Nebius infrastructure is defined entirely in Terraform. A developer manually creates a new subnet in the Nebius console (not via Terraform) to test something. The next day, another engineer runs `terraform apply` from the existing codebase. What happens to the manually created subnet?

- A) Terraform detects and imports the subnet automatically
- B) Terraform ignores resources it doesn't manage — the subnet stays untouched
- C) Nothing — `terraform apply` only modifies resources defined in the config and tracked in state. The manually created subnet is unknown to Terraform and is unaffected.
- D) Terraform deletes the subnet because it's not in state

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **C** | Data engineers upload (write) = editors. Trainers start/stop VMs (manage) = editors. Compliance auditors review what exists but must not access data = auditors. |
| 2 | **D** | Both are plausible. A rotated key causes 403. Being removed from IAM group also causes 403. Check the access key validity first, then IAM group membership. |
| 3 | **B** | IAM groups at tenant level grant access across tenant-level resources. To manage resources inside a specific project, the SA must be in the group at project scope. |
| 4 | **B** | MysteryBox versioning lets you rotate the secret (create a new version, set as primary) and the script fetches the current primary version on each run — no pod restarts needed. |
| 5 | **C** | Audit Logs record all API calls with who, what, when. Filter by `eventType: DeleteInstance` and the VM's resource ID. The `subject` field identifies the actor. |
| 6 | **B** | Budget alerts are notifications only. No automatic resource action occurs. The manager is right that manual action is needed; the junior engineer's assumption is wrong. |
| 7 | **B** | Network SSD is always encrypted by default. NRD requires explicit encryption enablement at creation time. KMS can be used for customer-controlled key management on top. |
| 8 | **B** | Creating a service account and generating an access key are necessary but not sufficient. The SA must also be added to an IAM group (at minimum `viewers`) to have any permissions to operate on resources. |
| 9 | **B** | `fabric-5` = H200 in `eu-west1`. `fabric-7` = H200 in `eu-north1`. Fabric and GPU type must match the region where the cluster is created. |
| 10 | **B** | Nodes `Ready` but `Insufficient nvidia.com/gpu` means the GPU device plugin is not running. The NVIDIA GPU Operator installs and manages the device plugin. Without it, `nvidia.com/gpu` resources are never registered. |
| 11 | **B** | High error counts on a specific IB port indicate a physical link problem. Drain the node to preserve the job on remaining nodes and raise a hardware support ticket. |
| 12 | **B** | GPU clusters are tied to a specific region and InfiniBand fabric. Region is immutable. You must create a new cluster in `us-central1` with the appropriate fabric and recreate VMs. |
| 13 | **B** | B200 GPUs always require the Network Operator, even with Nebius boot disk images. The boot disk image handles the GPU Operator components, but the Network Operator must still be installed for InfiniBand. |
| 14 | **B** | InfiniBand P-Keys provide full hardware-level isolation between clusters on the same fabric. Neither can see the other's traffic regardless of sharing the physical fabric. |
| 15 | **B** | Changing the driver preset triggers automated node recreation — Managed Kubernetes drains existing nodes and creates new ones. Save checkpoints and warn users before running the upgrade. |
| 16 | **C** | Temporarily removing the PDB allows drain to complete. If the node has already failed and training is broken, the risk of brief PDB removal is acceptable. Recreate the PDB immediately after draining. |
| 17 | **B** | NRD = no redundancy. Disk failure = total data loss. For a Slurm controller holding accounting data, use Network SSD (erasure coding, 2 failure tolerance) or IO M3 (3-way replication). |
| 18 | **B** | Nebius Shared Filesystem is the native solution: concurrent multi-VM access, virtiofs mount, up to 5 PiB, 12 GiB/s read per client. Manual NFS is complex and adds a single point of failure. |
| 19 | **A** | `--control-plane-endpoints-public-endpoint=false` disables public access. `--control-plane-etcd-cluster-size 3` sets HA. Both must be explicit — the public endpoint is on by default. |
| 20 | **B** | `get-compatibility-matrix` returns supported driver presets and OS images for a given K8s version and GPU platform. This is the right command before changing driver presets. |
| 21 | **B** | `nvidia-smi -q | grep -i ecc` shows accumulated ECC error counts per GPU. `dmesg | grep -i nvidia` shows kernel-level GPU hardware events. Together they reveal hardware errors from the training run. |
| 22 | **B** | 3 etcd stores is the default and ensures cluster control plane availability even if one etcd node fails. With 1 etcd store, any single failure takes down the cluster. HA doesn't cost extra. |
| 23 | **B** | Checkpointing is the standard fault-tolerance pattern for long training runs. Save to shared filesystem (fast write) during training. On failure, resume from last checkpoint. |
| 24 | **B** | Serverless AI Endpoints are always-on services billed by runtime, not by requests. To avoid overnight charges, explicitly stop the endpoint and restart it in the morning. |
| 25 | **B** | 4 nodes are drained (not accepting jobs). Undrain them if healthy. The job needs 8 nodes — it will remain pending until all 8 are available. |
| 26 | **B** | `NET/IB: No device found` in NCCL logs means NCCL cannot see the InfiniBand hardware. Check: (1) `NCCL_IB_DISABLE=0`, (2) Network Operator NICClusterPolicy = "ready", (3) `ibstat` shows Active ports. |
| 27 | **B** | MLflow Managed Service has documented regional unavailability in `eu-north2` and `eu-west1`. This is a platform limitation, not a permissions issue. |
| 28 | **B** | 99.75% VRAM usage with low compute utilization = memory bandwidth bottleneck. The GPU spends most time on memory operations, not compute. Reduce context length or move to higher-VRAM GPU. |
| 29 | **B** | Serverless AI Jobs are ideal for many short independent jobs. You pay only for actual runtime. A dedicated cluster charges for all GPUs for the entire duration, even when idle between jobs. |
| 30 | **B** | Copying to a Shared Filesystem eliminates per-request HTTP overhead of Object Storage and provides up to 12 GiB/s read throughput per client — much faster for repeated epoch reads of the same data. |
| 31 | **B** | `infiniband_fabric` is immutable. Terraform will destroy and recreate the entire cluster. All running jobs will be terminated. Coordinate a maintenance window and ensure checkpoints are saved. |
| 32 | **B** | The S3 backend alone does not provide state locking. Without locking, concurrent applies can corrupt the state file. Add DynamoDB-compatible locking or use a backend that supports it. |
| 33 | **C** | The `name` field alone shouldn't force replacement on most resources — inspect the full plan output. Something else in the config likely changed simultaneously (e.g., subnet, version). |
| 34 | **B** | Zero GPU utilization is most often a completed or cancelled job, not infrastructure failure. Check job status first. Only escalate to infrastructure investigation if the job is still supposed to be running. |
| 35 | **B** | After node recreation, the driver daemonset reinstalls the driver. `nvidia.com/gpu: 0` is temporary — it resolves once the driver daemonset log shows "Done, now waiting for signal". |
| 36 | **B** | Separate state files per project isolate blast radius and keep state manageable. Modules provide code reuse without coupling state. |
| 37 | **B** | 90°C is in the thermal throttling range for H100s. Monitor for clock throttling (`nvidia-smi -q | grep "Clocks Throttle"`). Alert Nebius support about potential cooling issues in the datacenter. |
| 38 | **B** | Draining all nodes simultaneously removes all capacity. Rolling upgrades must be sequential: one node at a time. Always verify the node is healthy and back in service before moving to the next. |
| 39 | **B** | Distributed Traces show per-span latency across the full request chain. When metrics look normal and logs show no errors, traces are the only tool that reveals where the time is actually going. |
| 40 | **C** | Terraform only manages resources in its state file. Manually created resources are invisible to Terraform — it will not touch, import, or delete them unless you explicitly import them with `terraform import`. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Exam ready |
| 30–35 | Close — focus on weak domains |
| 24–29 | More practice needed |
| <24 | Re-read notes with focus on applying knowledge |
