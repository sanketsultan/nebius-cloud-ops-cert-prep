# Nebius AI Cloud Ops — Mock Exam 3 (40 Questions)

**Exam format:** Multiple choice, single correct answer per question  
**Time:** ~50 minutes  
**Domains:**
- Domain 1: Security, Compliance & Billing (~20%) — Q1–Q8
- Domain 2: Setting Up & Operating GPU Clusters (~35%) — Q9–Q22
- Domain 3: Running Training & Inference Workloads (~20%) — Q23–Q30
- Domain 4: Platform Automation & Maintenance (~25%) — Q31–Q40

> Attempt all questions before checking the answer key at the bottom.

---

## Domain 1: Security, Compliance & Billing

**Q1.** Which Nebius IAM group gives users the ability to download objects from Object Storage?

- A) auditors
- B) viewers
- C) editors only
- D) admins only

---

**Q2.** A new service account needs to upload files to an Object Storage bucket. Which CLI command adds it to the editors group?

- A) `nebius iam service-account update --role editor`
- B) `nebius iam group-membership create --parent-id <editors_group_id> --member-id <sa_id>`
- C) `nebius iam add-role --sa <id> --role editor`
- D) `nebius iam service-account grant --group editors --id <id>`

---

**Q3.** What does the KMS symmetric key support that asymmetric keys do NOT?

- A) Key rotation
- B) Signing data
- C) Envelope encryption for arbitrary data sizes
- D) Multi-region replication

---

**Q4.** A developer hardcodes an API key in a Python script that gets pushed to GitHub. What Nebius service would have prevented this?

- A) Audit Logs
- B) KMS
- C) MysteryBox
- D) IAM groups

---

**Q5.** How many projects can one Nebius tenant contain?

- A) 1 (one project per tenant)
- B) Up to 5 (default quota)
- C) Multiple projects per tenant
- D) Projects are not associated with tenants

---

**Q6.** You want to track all `DeleteInstance` API calls made in your Nebius tenant over the last 30 days. Where do you look?

- A) Observability Logs
- B) Monitoring Metrics
- C) Audit Logs
- D) Billing history

---

**Q7.** Which statement about billing alerts in Nebius is true?

- A) VMs are stopped automatically when the budget threshold is reached
- B) You can configure alerts to terminate spot instances only
- C) Budget alerts are notifications only — resources continue running
- D) Budget alerts are sent via Audit Logs

---

**Q8.** A service account in Project A tries to list VMs in Project B. What happens?

- A) It succeeds if the service account is in the tenant's admins group
- B) It fails — service accounts can only work within their own project
- C) It succeeds if the service account has the `viewers` role in Project B
- D) It depends on cross-project IAM policies

---

## Domain 2: Setting Up & Operating GPU Clusters

**Q9.** Which GPU platform should you choose for the highest per-GPU HBM3e memory (288 GB) on Nebius?

- A) `gpu-h200-sxm`
- B) `gpu-b200-sxm`
- C) `gpu-b300-sxm`
- D) `gpu-h100-sxm`

---

**Q10.** You are creating a GPU cluster in `eu-north1` with H100 GPUs. Which InfiniBand fabrics are available?

- A) `fabric-5`, `fabric-7`
- B) `fabric-2`, `fabric-3`, `fabric-4`, `fabric-6`
- C) `eu-north1-a`, `eu-north1-b`
- D) `fabric-1`, `fabric-2`

---

**Q11.** An H200 GPU cluster in `eu-west1` uses which InfiniBand fabric?

- A) `fabric-7`
- B) `eu-north2-a`
- C) `fabric-5`
- D) `us-central1-a`

---

**Q12.** What is the maximum capacity of a Nebius shared filesystem?

- A) 100 TiB
- B) 1 PiB
- C) 5 PiB
- D) Unlimited

---

**Q13.** You need to create a Network SSD NRD disk that is approximately 200 GiB. Which sizes are valid?

- A) 186 GiB or 279 GiB
- B) 200 GiB exactly
- C) 190 GiB or 210 GiB
- D) 256 GiB

---

**Q14.** Which command installs the NVIDIA Network Operator from the Nebius chart repo?

- A) `helm install network-operator nvidia/network-operator`
- B) `helm install network-operator oci://cr.eu-north1.nebius.cloud/marketplace/nebius/nvidia-network-operator/chart/network-operator -n nvidia-network-operator --create-namespace --wait`
- C) `kubectl apply -f https://raw.githubusercontent.com/Mellanox/network-operator/master/config/deploy/operator-cfg.yaml`
- D) `nebius mk8s operator install --name nvidia-network-operator`

---

**Q15.** What CUDA driver version does preset `cuda13.0` provide?

- A) 570.x
- B) 550.x
- C) 580.x
- D) 12.0

---

**Q16.** You try to put an L40S VM into a GPU cluster. This fails because:

- A) L40S is not supported in your region
- B) L40S does not have InfiniBand connectivity — it is PCIe only
- C) L40S requires a different cluster type
- D) L40S VMs must be created before the cluster

---

**Q17.** How does InfiniBand Partition Key (P-Key) isolation affect two clusters on the same physical fabric?

- A) They share bandwidth evenly
- B) One cluster can see the other's traffic but not modify it
- C) They cannot communicate over InfiniBand — traffic is fully isolated by P-Key
- D) P-Keys only limit bandwidth, not visibility

---

**Q18.** What happens to running pods when you run `kubectl cordon <node>`?

- A) All pods are evicted immediately
- B) Running pods continue, but no new pods will be scheduled on that node
- C) Pods are paused
- D) DaemonSet pods are terminated

---

**Q19.** You need to run 8 × B200 VMs in a cluster in `me-west1`. Which fabric do you use?

- A) `us-central1-b`
- B) `me-west1-a`
- C) `fabric-5`
- D) `eu-north1-ib`

---

**Q20.** Network SSD IO M3 is specifically recommended for which workload?

- A) K8s worker node ephemeral storage
- B) Boot disks for control plane nodes
- C) High-performance distributed storage clusters like GlusterFS
- D) Object Storage caching

---

**Q21.** You want to list all GPU clusters in your project. Which CLI command is correct?

- A) `nebius compute gpu list`
- B) `nebius compute gpu-cluster list`
- C) `nebius mk8s cluster list --type gpu`
- D) `nebius compute instance list --gpu-cluster`

---

**Q22.** A shared filesystem can be attached to VMs in which scope?

- A) Any VM in any project in the same tenant
- B) Any VM in any tenant in the same region
- C) Only VMs in the same project
- D) Only VMs that share the same InfiniBand fabric

---

## Domain 3: Running Training & Inference Workloads

**Q23.** You want to deploy a model that handles real-time API requests from external users. Which Serverless AI deployment do you choose?

- A) Serverless AI Job
- B) Serverless AI Endpoint
- C) Managed Service for Soperator
- D) MLflow Endpoint

---

**Q24.** Which Soperator deployment option is fully managed by Nebius and can be deployed in any supported region?

- A) Self-deploy on Nebius Kubernetes
- B) Pro Solution for Soperator
- C) Managed Service for Soperator
- D) Self-deploy on other platforms

---

**Q25.** A Serverless AI Job finished running. How do you access its output?

- A) From the public URL assigned to the job
- B) From the job's Object Storage output bucket
- C) Jobs have no URL — output must be written to storage during execution
- D) From the MLflow artifact store

---

**Q26.** You're running a 64-GPU training job across 8 nodes (8 GPUs each). What should `RANK` be for the process running on GPU 0 of node 3?

- A) 3
- B) 0
- C) 24
- D) 8

---

**Q27.** You want to track multiple training runs — their hyperparameters, metrics, and artifacts — in a central UI. Which Nebius service handles this?

- A) Observability Logs
- B) Serverless AI Endpoints
- C) MLflow Managed Service
- D) Nebius Monitoring dashboards

---

**Q28.** Your NCCL-based distributed training job shows low inter-node bandwidth despite InfiniBand being connected. What environment variable do you check first?

- A) `MASTER_PORT`
- B) `NCCL_IB_DISABLE`
- C) `NCCL_SOCKET_IFNAME`
- D) `WORLD_SIZE`

---

**Q29.** A Slurm job has been in PD state for 30 minutes. What command gives you the reason?

- A) `sacct -j <jobid>`
- B) `sinfo -p gpu`
- C) `squeue -j <jobid>`
- D) `sbatch --debug <jobid>`

---

**Q30.** For storing training datasets that need to be accessed simultaneously by 16 worker nodes, what is the best storage option?

- A) Object Storage, reading directly from each node
- B) Network SSD NRD, one per node
- C) Shared filesystem
- D) Local NVMe on each node

---

## Domain 4: Platform Automation & Maintenance

**Q31.** In Nebius Terraform, you define a Kubernetes node group with H200 GPUs. What resource type do you use?

- A) `nebius_kubernetes_gpu_node`
- B) `nebius_kubernetes_node_group`
- C) `nebius_mk8s_node_group`
- D) `nebius_compute_gpu_group`

---

**Q32.** A colleague changed the `zone` field on a Terraform `nebius_compute_instance` resource. What will `terraform plan` show?

- A) No change — zone is updated in-place
- B) A warning but no action
- C) Force replacement — the instance will be destroyed and recreated
- D) An error — zone cannot be in Terraform config

---

**Q33.** Which Nebius CLI command retrieves kubeconfig credentials for a Managed Kubernetes cluster?

- A) `nebius mk8s cluster get-credentials --id <id> --external`
- B) `kubectl config set-cluster nebius --server <url>`
- C) `nebius compute k8s get-credentials <id>`
- D) `nebius mk8s cluster connect --id <id>`

---

**Q34.** What is the difference between Audit Logs and Observability Logs?

- A) Audit Logs are stored for 90 days; Observability Logs for 30 days
- B) Audit Logs = control plane API calls; Observability Logs = workload data plane events
- C) Audit Logs are only for billing; Observability Logs are for security
- D) They are different names for the same service

---

**Q35.** You need to alert your on-call team when GPU utilization drops below 10% during an expected training job. Which Nebius feature do you use?

- A) Audit Log webhook
- B) Billing budget alert
- C) Monitoring threshold alert
- D) Serverless AI health check

---

**Q36.** Your team has 3 engineers concurrently managing GPU cluster infrastructure with Terraform. What prevents state corruption?

- A) Using separate Terraform modules
- B) Remote backend with state locking
- C) Running Terraform in Docker containers
- D) Using `terraform workspace`

---

**Q37.** While debugging slow storage I/O on a GPU node, which command shows which process is consuming disk bandwidth in real time?

- A) `iostat -x 1`
- B) `iotop`
- C) `dmesg | grep io`
- D) `df -h`

---

**Q38.** After running `kubectl drain <node>`, which pods are NOT evicted?

- A) Pods with PodDisruptionBudget
- B) DaemonSet pods (with `--ignore-daemonsets` flag)
- C) Pods in kube-system
- D) Pods with emptyDir volumes

---

**Q39.** OpenTelemetry traces in Nebius Observability are useful for which scenario?

- A) Finding which node is running out of disk space
- B) Finding why a GPU is underutilized
- C) Finding which microservice adds the most latency in a multi-hop API call
- D) Auditing IAM changes

---

**Q40.** You want to verify that the NVIDIA GPU Operator driver was successfully installed on a node. What is the correct check?

- A) `kubectl get nodes --show-gpu`
- B) SSH to the node and run `nvidia-smi`
- C) Check the last log line of the `nvidia-driver-daemonset` pod in `nvidia-gpu-operator` namespace — it should say "Done, now waiting for signal"
- D) Run `helm status gpu-operator`

---

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | `viewers` can view resources AND access data inside them, including downloading objects. `auditors` cannot access data. |
| 2 | **B** | `group-membership create` with the group's ID and the SA's ID is the correct pattern from the Nebius CLI docs. |
| 3 | **C** | Symmetric keys are used for encrypt/decrypt operations including envelope encryption. Asymmetric keys are used for signing and verification. |
| 4 | **C** | MysteryBox stores secret values (API keys, tokens, passwords) that can be referenced from scripts without being hardcoded. |
| 5 | **C** | A tenant can contain multiple projects. Each project belongs to one tenant and one region. |
| 6 | **C** | Audit Logs record who did what to Nebius resources — including all API calls like `DeleteInstance`. |
| 7 | **C** | Budget alerts are notifications only. Resources keep running regardless of budget threshold. |
| 8 | **B** | Service accounts belong to a project and can only work with resources in their own project. |
| 9 | **C** | B300 SXM has 288 GB HBM3e per GPU — the highest in the Nebius lineup. |
| 10 | **B** | H100 SXM (`gpu-h100-sxm`) in `eu-north1` maps to fabrics `fabric-2`, `fabric-3`, `fabric-4`, and `fabric-6`. |
| 11 | **C** | `fabric-5` serves H200 (`gpu-h200-sxm`) GPUs in the `eu-west1` region. |
| 12 | **C** | Nebius shared filesystems support up to 5 PiB capacity. |
| 13 | **A** | NRD disk sizes must be multiples of 93 GiB. Valid sizes near 200 GiB: 93×2=186 GiB or 93×3=279 GiB. |
| 14 | **B** | The Nebius-specific chart registry is at `oci://cr.eu-north1.nebius.cloud/marketplace/nebius/...`. Do not use generic NVIDIA charts. |
| 15 | **C** | `cuda13.0` provides NVIDIA driver 580.x on Ubuntu 24.04. `cuda12.8` provides driver 570.x. |
| 16 | **B** | L40S is a PCIe-only GPU with no InfiniBand support. GPU clusters require InfiniBand. |
| 17 | **C** | P-Keys provide full isolation — clusters on the same physical fabric cannot communicate with each other over InfiniBand. |
| 18 | **B** | `cordon` marks the node as unschedulable but does NOT evict running pods. Only `drain` evicts existing pods. |
| 19 | **B** | B200 SXM in `me-west1` uses the `me-west1-a` InfiniBand fabric. |
| 20 | **C** | Network SSD IO M3 is explicitly recommended for performance-critical storage solutions like GlusterFS on Nebius. |
| 21 | **B** | `nebius compute gpu-cluster list` is the correct command. |
| 22 | **C** | Shared filesystems can only be attached to VMs within the same project. |
| 23 | **B** | Serverless AI Endpoints are always-on, have a public URL, and are designed for real-time inference. |
| 24 | **C** | Managed Service for Soperator is the fully-managed Nebius option, deployable in any region with a few clicks. |
| 25 | **C** | Jobs have no public URL and no persistent endpoint. Output must be written to Object Storage or a shared filesystem during execution. |
| 26 | **C** | RANK = (node_index × GPUs_per_node) + gpu_index = (3 × 8) + 0 = 24. |
| 27 | **C** | MLflow Managed Service is Nebius's experiment tracking and model lifecycle management platform. |
| 28 | **B** | If `NCCL_IB_DISABLE=1`, NCCL falls back to Ethernet instead of InfiniBand. Set to `0` to enable IB. |
| 29 | **C** | `squeue -j <jobid>` shows the current state and reason for pending jobs. |
| 30 | **C** | The shared filesystem is designed for simultaneous multi-VM read access. |
| 31 | **B** | The Terraform resource for Managed Kubernetes node groups is `nebius_kubernetes_node_group`. |
| 32 | **C** | `zone` is an immutable field. Terraform will mark the resource for replacement and show `# forces replacement` in the plan. |
| 33 | **A** | `nebius mk8s cluster get-credentials --id <id> --external` writes the kubeconfig to `~/.kube/config`. |
| 34 | **B** | Audit Logs track "who did what to Nebius resources." Observability Logs track "what happened inside your workloads." |
| 35 | **C** | Nebius Monitoring supports threshold-based alerts on metrics. `gpu_utilization` is a key metric you can alert on. |
| 36 | **B** | Remote backend with state locking prevents concurrent applies from corrupting the state file. |
| 37 | **B** | `iotop` shows real-time per-process I/O, identifying which process is the bandwidth consumer. |
| 38 | **B** | With `--ignore-daemonsets`, `drain` skips DaemonSet-managed pods because they run on every node and will restart immediately. |
| 39 | **C** | Distributed traces show the full request path across services with timing per span, ideal for finding latency hotspots. |
| 40 | **C** | The Nebius docs specify checking the last log line of the `nvidia-driver-daemonset` pod. "Done, now waiting for signal" confirms success. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Excellent — exam ready |
| 30–35 | Good — review weak domains |
| 24–29 | Fair — focused study needed |
| <24 | Re-read notes and docs |
