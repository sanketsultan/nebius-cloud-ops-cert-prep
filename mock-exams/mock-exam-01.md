# Nebius AI Cloud Ops — Full Mock Exam (40 Questions)

**Exam format:** Multiple choice, single correct answer per question
**Domains covered:**
- Domain 1: Security, Compliance & Billing (~20%) — Q1–Q8
- Domain 2: Setting Up & Operating GPU Clusters (~35%) — Q9–Q22
- Domain 3: Running Training & Inference Workloads (~20%) — Q23–Q30
- Domain 4: Platform Automation & Maintenance (~25%) — Q31–Q40

---

## Domain 1: Security, Compliance & Billing

**Q1.** You need to grant a CI/CD pipeline the ability to push images to Nebius Container Registry but nothing else. What should you create and assign?

- A) A personal user account with container.editor role
- B) A service account with container.editor role scoped to the project
- C) A service account with iam.admin role
- D) A user group with storage.admin role

**Answer: B**
*Service accounts are the correct identity for automated workloads. Scope to the minimum necessary role.*

---

**Q2.** Which Nebius service encrypts data at rest using customer-controlled cryptographic keys?

- A) MysteryBox
- B) Audit Logs
- C) Key Management Service (KMS)
- D) IAM

**Answer: C**
*KMS manages encryption keys. MysteryBox stores secrets/tokens, not keys for data encryption.*

---

**Q3.** A compliance team asks for a full record of all API calls made to your Nebius project over the last 30 days. Which service provides this?

- A) Observability > Logs
- B) Observability > Traces
- C) Audit Logs
- D) Metrics and Alerts

**Answer: C**
*Audit Logs capture user and service API activity for security and compliance purposes.*

---

**Q4.** Your team has three sub-teams each needing different levels of access to the same project. What is the correct Nebius IAM approach?

- A) Create one shared service account for all teams
- B) Assign roles at the individual user or service account level per team
- C) Give all teams iam.admin so they can manage themselves
- D) Use Object Storage bucket policies instead of IAM

**Answer: B**
*IAM roles should be assigned at the appropriate granularity — per user or service account — following least-privilege.*

---

**Q5.** You store a third-party API key that your training job fetches at startup. Which Nebius service is most appropriate?

- A) KMS
- B) Object Storage
- C) MysteryBox
- D) Container Registry

**Answer: C**
*MysteryBox is designed for storing and retrieving sensitive secrets like API keys and tokens at runtime.*

---

**Q6.** A budget alert in Nebius billing is triggered. What does this mean?

- A) Your account has been suspended
- B) Spending has reached a configured threshold; resources continue running
- C) All running VMs are automatically stopped
- D) Your quota has been reduced

**Answer: B**
*Budget alerts are notifications only. They do not stop resources automatically unless explicitly configured to do so.*

---

**Q7.** Which IAM role scope applies to all projects within a tenant?

- A) Project-level role
- B) Resource-level role
- C) Tenant-level role
- D) Subnet-level role

**Answer: C**
*Nebius IAM has tenant-level and project-level scopes. Tenant-level roles apply across all projects.*

---

**Q8.** Under the shared responsibility model on Nebius AI Cloud, which of the following is the customer's responsibility?

- A) Physical hardware security
- B) Hypervisor patching
- C) Data encryption and IAM role assignments
- D) Network fabric maintenance

**Answer: C**
*The customer is responsible for data protection, IAM configuration, and workload security. Nebius manages physical and hypervisor layers.*

---

## Domain 2: Setting Up & Operating GPU Clusters

**Q9.** You need to run a distributed training job across 8 H100 GPU nodes with maximum inter-node bandwidth. Which network fabric should your cluster use?

- A) Standard 1 Gbps Ethernet
- B) 10 Gbps Ethernet
- C) InfiniBand
- D) VPC peering

**Answer: C**
*InfiniBand provides RDMA capability and the high bandwidth required for tightly coupled multi-node GPU training.*

---

**Q10.** After creating a Kubernetes GPU node group, pods requesting `nvidia.com/gpu` remain in `Pending` state. The nodes are `Ready`. What is the most likely cause?

- A) VPC security group blocking pod traffic
- B) NVIDIA device plugin daemonset is not deployed or not running
- C) Object Storage bucket is full
- D) IAM role missing on the node group

**Answer: B**
*The NVIDIA device plugin advertises GPU resources to the Kubernetes scheduler. Without it, GPU resource requests cannot be satisfied.*

---

**Q11.** Which command verifies that GPUs are visible and healthy on a Linux node?

- A) `kubectl get nodes -o wide`
- B) `lspci | grep NVIDIA`
- C) `nvidia-smi`
- D) `dmesg | grep GPU`

**Answer: C**
*`nvidia-smi` is the standard tool to check GPU presence, driver version, memory, and health status.*

---

**Q12.** You are attaching persistent storage to a single GPU VM for storing datasets. Which storage type gives the best IOPS for random read workloads?

- A) Object Storage bucket
- B) Network HDD disk
- C) Network SSD disk
- D) Shared filesystem

**Answer: C**
*Network SSD provides the highest IOPS for block storage attached to a single VM.*

---

**Q13.** You need to share a large dataset directory across 16 GPU nodes simultaneously during training. What storage option do you choose?

- A) One network SSD disk mounted on each node independently
- B) Shared filesystem mounted on all nodes
- C) Object Storage with per-node download scripts
- D) Container Registry layer caching

**Answer: B**
*A shared filesystem (e.g., NFS-based) allows concurrent read/write access from multiple nodes simultaneously.*

---

**Q14.** What is the primary purpose of configuring topology-aware scheduling in a GPU cluster with InfiniBand?

- A) To reduce billing costs
- B) To ensure pods are scheduled on nodes that share the same InfiniBand switch for minimum latency
- C) To automatically restart failed pods
- D) To balance CPU usage across node groups

**Answer: B**
*Topology awareness places communicating pods on nodes within the same InfiniBand domain, reducing hop count and latency.*

---

**Q15.** A node in your Kubernetes GPU cluster is experiencing GPU errors. You want to prevent new pods from being scheduled on it while keeping existing pods running. Which kubectl command do you use?

- A) `kubectl delete node <node>`
- B) `kubectl taint node <node>`
- C) `kubectl cordon <node>`
- D) `kubectl drain <node>`

**Answer: C**
*`cordon` marks a node as unschedulable without evicting running pods. `drain` also evicts existing pods.*

---

**Q16.** Which Nebius compute offering is designed specifically for Slurm-based HPC workloads?

- A) Managed Kubernetes clusters
- B) Serverless AI endpoints
- C) Soperator clusters
- D) Applications (JupyterLab)

**Answer: C**
*Soperator is Nebius's managed Slurm operator for running HPC-style workloads.*

---

**Q17.** You create a VPC for a GPU cluster and need nodes to reach the internet for pulling container images, but you do not want nodes to have public IPs. What do you configure?

- A) Assign public IPs to each node
- B) Configure a NAT gateway in the VPC
- C) Use VPC peering to another project
- D) Open all outbound security group rules

**Answer: B**
*A NAT gateway allows outbound internet access from private nodes without exposing them with public IPs.*

---

**Q18.** After provisioning a GPU cluster, you want to validate that InfiniBand links are active. Which tool do you run on a node?

- A) `ping` between nodes
- B) `ibstat` or `ibstatus`
- C) `nvidia-smi topo -m`
- D) `netstat -an`

**Answer: B**
*`ibstat` / `ibstatus` are standard InfiniBand tools to check port state and link activity.*

---

**Q19.** Which of the following is NOT a valid GPU instance family available on Nebius AI Cloud?

- A) H100 SXM
- B) A100 80GB
- C) V100 SXM2
- D) H200

**Answer: C** *(based on current Nebius catalog; always verify against latest docs)*
*Nebius focuses on H100 and A100 GPU families. V100 is a previous-generation GPU not in the Nebius lineup.*

---

**Q20.** You want to run a GPU VM with a custom OS image that includes pre-installed ML libraries. Which Nebius feature supports this?

- A) Container Registry only
- B) Custom disk images for Compute VMs
- C) MLflow clusters
- D) Serverless AI

**Answer: B**
*Nebius Compute supports launching VMs from custom disk images, allowing pre-baked environments.*

---

**Q21.** A GPU node is repeatedly restarting. After SSHing in, you see GPU ECC memory errors in `nvidia-smi`. What is the appropriate action?

- A) Reboot the node and ignore the errors
- B) Drain the node, report the hardware fault to Nebius support, and replace it
- C) Increase the VM disk size
- D) Restart the NVIDIA device plugin

**Answer: B**
*ECC errors indicate hardware-level memory faults. The node should be drained and the issue escalated to the cloud provider for hardware replacement.*

---

**Q22.** What does `kubectl drain <node> --ignore-daemonsets` do?

- A) Deletes the node permanently
- B) Evicts all pods (except daemonset pods) from the node and marks it unschedulable
- C) Restarts the kubelet on the node
- D) Removes GPU resources from the node

**Answer: B**
*`drain` is used for graceful node maintenance — it evicts workload pods while leaving daemonsets (like device plugins) intact.*

---

## Domain 3: Running Training & Inference Workloads

**Q23.** You submit a Slurm job with `sbatch train.sh`. After 5 minutes it is still in `PD` state. Which command gives you the reason?

- A) `slog 1042`
- B) `squeue -j <jobid> --reason`
- C) `sbatch --status <jobid>`
- D) `sinfo --job <jobid>`

**Answer: B**
*`squeue -j <jobid>` with the `--reason` or `-o` format flags shows why a job is pending (e.g., resources unavailable, priority).*

---

**Q24.** You are running a PyTorch distributed training job on 4 Kubernetes nodes. Which environment variables must be set for `torch.distributed` to initialize correctly?

- A) `CUDA_VISIBLE_DEVICES` and `GPU_COUNT`
- B) `MASTER_ADDR`, `MASTER_PORT`, `WORLD_SIZE`, and `RANK`
- C) `NCCL_DEBUG` and `IB_PORT`
- D) `KUBECONFIG` and `NODE_NAME`

**Answer: B**
*PyTorch distributed requires MASTER_ADDR/PORT for rendezvous, WORLD_SIZE for total process count, and RANK for each process identity.*

---

**Q25.** Your inference endpoint deployed via Nebius Applications (vLLM) is returning high latency. What metric do you check first in Observability?

- A) VPC packet loss
- B) GPU memory utilization and GPU compute utilization on the inference node
- C) Object Storage request latency
- D) IAM token refresh rate

**Answer: B**
*High inference latency typically correlates with GPU memory saturation (model too large for VRAM) or GPU compute bottleneck.*

---

**Q26.** You need to save model checkpoints every 10 minutes during a long training run. Multiple nodes need to write to the same location. Which storage approach is correct?

- A) Write to local NVMe on node 0 only
- B) Write to a shared filesystem accessible by all nodes
- C) Write to Container Registry
- D) Write to a separate VM's disk over SSH

**Answer: B**
*A shared filesystem allows all training nodes to write checkpoints to the same path without coordination overhead.*

---

**Q27.** Which Nebius service provides managed experiment tracking and a model registry for ML workflows?

- A) Serverless AI
- B) MLflow clusters
- C) Soperator
- D) Container Registry

**Answer: B**
*Nebius MLflow clusters provide managed MLflow for experiment tracking, metric logging, and model registry.*

---

**Q28.** You want to run a one-off batch inference job on 100,000 images without managing persistent infrastructure. Which Nebius service fits best?

- A) Managed Kubernetes cluster (always-on)
- B) Serverless AI jobs
- C) Soperator (Slurm)
- D) PostgreSQL cluster

**Answer: B**
*Serverless AI jobs run containerized workloads on-demand without persistent infrastructure, ideal for batch processing.*

---

**Q29.** Your training job uses NCCL for GPU communication. You observe low bandwidth between nodes. What environment variable do you set to force NCCL to use InfiniBand?

- A) `NCCL_SOCKET_IFNAME=ib0`
- B) `NCCL_IB_DISABLE=0` and ensure `NCCL_IB_HCA` points to the IB interface
- C) `CUDA_VISIBLE_DEVICES=all`
- D) `IB_FORCE=true`

**Answer: B**
*Setting `NCCL_IB_DISABLE=0` enables InfiniBand in NCCL. `NCCL_IB_HCA` specifies which IB adapter to use.*

---

**Q30.** A data scientist wants an interactive GPU environment with Jupyter notebooks without managing a full VM. Which Nebius offering is most appropriate?

- A) Soperator cluster
- B) Applications — JupyterLab
- C) Container Registry
- D) PostgreSQL cluster

**Answer: B**
*Nebius Applications includes a turnkey JupyterLab deployment with GPU support.*

---

## Domain 4: Platform Automation & Maintenance

**Q31.** You are writing Terraform to provision a Nebius GPU VM. Which provider block is required?

- A) `provider "aws" {}`
- B) `provider "google" {}`
- C) `provider "nebius" {}`
- D) `provider "yandex" {}`

**Answer: C**
*Nebius has its own Terraform provider (`nebius`). It is distinct from other providers.*

---

**Q32.** You want Terraform state to be shared across your team and locked during applies. What do you configure?

- A) Local `terraform.tfstate` on each developer's laptop
- B) Remote backend (e.g., S3-compatible Object Storage) with state locking
- C) Git repository for the state file
- D) Email the state file to team members before each apply

**Answer: B**
*Remote backends with locking prevent concurrent applies and ensure a single source of truth for state.*

---

**Q33.** Which `nebius` CLI command lists all running compute instances in a project?

- A) `nebius compute instance list`
- B) `nebius vm show --all`
- C) `nebius list resources`
- D) `nebius compute describe`

**Answer: A**
*The Nebius CLI follows a `nebius <service> <resource> <verb>` pattern.*

---

**Q34.** You want to alert when a GPU node's disk usage exceeds 85%. Which sequence is correct in Nebius Observability?

- A) Logs > Create filter > Set threshold
- B) Metrics and Alerts > Select disk metric > Create threshold alert > Configure notification channel
- C) Audit Logs > Set disk threshold
- D) Traces > Add disk span

**Answer: B**
*Metrics and Alerts is the correct service for threshold-based infrastructure alerting.*

---

**Q35.** A planned maintenance window is announced for your GPU node. What should you do before the window starts to avoid job failures?

- A) Do nothing; Nebius migrates workloads automatically
- B) Drain the node to reschedule pods, then let maintenance proceed
- C) Delete the node group
- D) Snapshot all running pods

**Answer: B**
*Draining the node gracefully evicts workloads before maintenance, preventing abrupt job failures.*

---

**Q36.** You need to roll out a new NVIDIA driver version to all nodes in a Kubernetes GPU cluster with zero downtime. What is the correct approach?

- A) SSH into all nodes simultaneously and run the upgrade
- B) Cordon and drain one node at a time, upgrade the driver, uncordon, then repeat
- C) Delete the entire node group and recreate it
- D) Update the driver only on the control plane

**Answer: B**
*Rolling one-node-at-a-time drain-upgrade-uncordon is the standard zero-downtime node maintenance pattern.*

---

**Q37.** Your Terraform `plan` shows it wants to destroy and recreate a GPU VM you did not intend to change. What is the most likely cause?

- A) The Nebius provider is outdated
- B) An immutable field (e.g., instance platform or boot disk type) was changed in the resource definition
- C) The state file is missing
- D) Terraform cannot manage GPU VMs

**Answer: B**
*Immutable fields force resource replacement. Check which field changed and whether it can be set in-place.*

---

**Q38.** You observe intermittent InfiniBand link flapping between two nodes during training. What logs do you check?

- A) Kubernetes event logs for pod restarts
- B) `dmesg` and `ibdiagnet` / `ibcheckerrors` on the affected nodes
- C) Nebius billing logs
- D) MLflow experiment logs

**Answer: B**
*`dmesg` shows kernel-level IB events; `ibdiagnet` scans the fabric for errors and link state changes.*

---

**Q39.** Which of the following best describes the Nebius Terraform provider's relationship to the Nebius gRPC API?

- A) The Terraform provider calls the gRPC API under the hood to manage resources
- B) They are completely separate systems with no relationship
- C) The gRPC API is only for billing operations
- D) Terraform directly modifies the Nebius database

**Answer: A**
*The Terraform provider translates HCL resource definitions into gRPC API calls against Nebius control plane endpoints.*

---

**Q40.** After a training node crashes and recovers, your distributed job fails to resume. What is the most reliable mitigation strategy to implement before the next run?

- A) Use a larger VM type
- B) Implement checkpoint-and-resume logic saving state to a shared filesystem at regular intervals
- C) Disable InfiniBand
- D) Run all training on a single node to avoid distributed failure modes

**Answer: B**
*Checkpointing to durable shared storage allows a job to resume from the last checkpoint after a node failure, avoiding full restarts.*

---

## Answer Key

| Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|
| 1 | B | 11 | C | 21 | B | 31 | C |
| 2 | C | 12 | C | 22 | B | 32 | B |
| 3 | C | 13 | B | 23 | B | 33 | A |
| 4 | B | 14 | B | 24 | B | 34 | B |
| 5 | C | 15 | C | 25 | B | 35 | B |
| 6 | B | 16 | C | 26 | B | 36 | B |
| 7 | C | 17 | B | 27 | B | 37 | B |
| 8 | C | 18 | B | 28 | B | 38 | B |
| 9 | C | 19 | C | 29 | B | 39 | A |
| 10 | B | 20 | B | 30 | B | 40 | B |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Excellent — exam ready |
| 30–35 | Good — review weak domains |
| 24–29 | Fair — focused study needed |
| <24 | Re-read the exam guide and docs |

---

*Source: Nebius AI Cloud Ops Exam Guide + https://docs.nebius.com/*
