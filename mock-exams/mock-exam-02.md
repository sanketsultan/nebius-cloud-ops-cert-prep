# Nebius AI Cloud Ops — Mock Exam 2 (40 Questions)

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

**Q1.** In Nebius IAM, which default group can view resources but cannot access data inside them?

- A) viewers
- B) editors
- C) auditors
- D) admins

---

**Q2.** A service account needs to access Nebius Object Storage using the AWS CLI. Which type of key does it need?

- A) Authorized key
- B) Access key
- C) IAM token
- D) SSH key

---

**Q3.** Where do projects sit in the Nebius resource hierarchy?

- A) Projects contain tenants
- B) Projects are inside tenants
- C) Projects and tenants are at the same level
- D) Projects are inside resources

---

**Q4.** A new user is added to a Nebius tenant. They cannot see any resources. What is the most likely reason?

- A) Their account has not been verified
- B) They have not been added to any IAM group
- C) The tenant has no projects
- D) Their service account key has expired

---

**Q5.** MysteryBox secrets are structured as Secret → Secret Version → Payload. What is the purpose of having multiple versions?

- A) Each version stores a different secret name
- B) Versions allow secret rotation without breaking existing references
- C) Versions are required for billing purposes
- D) Each version maps to a different IAM role

---

**Q6.** You want to encrypt data stored on a Network SSD Non-replicated disk. What is correct?

- A) NRD disks are always encrypted and it cannot be disabled
- B) NRD disks cannot be encrypted
- C) NRD disk encryption is optional and must be enabled at creation
- D) NRD disks use KMS automatically

---

**Q7.** Which statement about Audit Logs is correct?

- A) Audit Logs only record failed API calls
- B) Audit Logs can be exported to Object Storage for long-term retention
- C) Audit Logs are stored inside Observability Logs
- D) Audit Logs only capture billing events

---

**Q8.** A budget alert fires in the Nebius console. What happens to running GPU VMs?

- A) They are stopped immediately
- B) They are paused and can be resumed
- C) Nothing — resources keep running, it is a notification only
- D) New VM creation is blocked but existing VMs continue

---

## Domain 2: Setting Up & Operating GPU Clusters

**Q9.** You need to run distributed training across 8 H200 nodes in `eu-north1`. Which InfiniBand fabric should you select?

- A) `fabric-2`
- B) `fabric-5`
- C) `fabric-7`
- D) `us-central1-a`

---

**Q10.** What does GPUDirect RDMA provide in a Nebius GPU cluster?

- A) Automatic GPU driver updates
- B) Data flows directly between GPU and NIC, bypassing the CPU
- C) Remote monitoring of GPU temperatures
- D) Automatic load balancing between GPU nodes

---

**Q11.** You have created a GPU VM and want to add it to a GPU cluster. Can you do this?

- A) Yes, using `nebius compute gpu-cluster add-vm`
- B) Yes, from the web console under GPU clusters
- C) No — VMs can only be added to a GPU cluster at creation time
- D) Yes, but only within 24 hours of VM creation

---

**Q12.** Two GPU clusters share the same physical InfiniBand fabric. Can their nodes communicate with each other over InfiniBand?

- A) Yes, all nodes on the same fabric can communicate
- B) No — each cluster gets a unique Partition Key (P-Key) that isolates its traffic
- C) Yes, but only with explicit security group rules
- D) No — different clusters must use different fabrics

---

**Q13.** You need 4 H100 GPUs on a single VM for inference. Which preset do you use?

- A) `4gpu-64vcpu-800gb`
- B) `1gpu-16vcpu-200gb` × 4
- C) H100 only has 1-GPU and 8-GPU presets — 4-GPU is not available
- D) `4gpu-128vcpu-1600gb`

---

**Q14.** What is the maximum read bandwidth per client for a Nebius shared filesystem?

- A) 450 MiB/s
- B) 1 GiB/s
- C) 8 GiB/s
- D) 12 GiB/s

---

**Q15.** You want to build a GlusterFS storage cluster on Nebius VMs that needs both high performance AND reliability. Which disk type do you choose?

- A) Network SSD
- B) Network SSD Non-replicated
- C) Network SSD IO M3
- D) Object Storage

---

**Q16.** An L40S GPU VM cannot be added to a GPU cluster. Why?

- A) L40S is too old for InfiniBand
- B) L40S uses PCIe and has no InfiniBand support
- C) L40S requires a special fabric not yet available
- D) L40S is only available in private regions

---

**Q17.** You are creating a Kubernetes node group with B200 GPUs without using the Nebius boot disk image. Which operators must you install?

- A) NVIDIA GPU Operator only
- B) NVIDIA Network Operator only
- C) Both NVIDIA GPU Operator and NVIDIA Network Operator
- D) Neither — B200 GPUs self-configure

---

**Q18.** You update the CUDA driver preset for an existing Kubernetes node group from `cuda12.8` to `cuda13.0`. What happens?

- A) Drivers update in place with no disruption
- B) Only new nodes get the new driver
- C) Managed Kubernetes recreates all nodes per the group's deployment strategy
- D) The update fails — you must create a new node group

---

**Q19.** Which command checks which GPU platforms and CUDA driver presets are compatible for your Kubernetes cluster version?

- A) `nebius compute platform list`
- B) `nebius mk8s node-group get-compatibility-matrix --cluster-kubernetes-version 1.33 --platform gpu-h200-sxm`
- C) `kubectl get nodes -o wide`
- D) `nebius mk8s cluster describe --gpu-info`

---

**Q20.** A Network SSD Non-replicated disk fails. What happens to its data?

- A) Data is recovered from erasure coding
- B) Data is recovered from the 3-way mirror
- C) Data is lost — NRD has no redundancy
- D) Data is automatically backed up to Object Storage

---

**Q21.** All VMs in a GPU cluster must be in the same ___?

- A) Region
- B) Tenant
- C) Project
- D) InfiniBand fabric

---

**Q22.** What is the total InfiniBand bandwidth for a single 8-GPU H100 node in a Nebius GPU cluster?

- A) 400 Gbps
- B) 800 Gbps
- C) 1.6 Tbps
- D) 3.2 Tbps

---

## Domain 3: Running Training & Inference Workloads

**Q23.** Which Soperator deployment option is best for enterprise-scale workloads with reserved capacity and expert assistance?

- A) Managed Service for Soperator
- B) Self-deploy on Nebius Kubernetes
- C) Pro Solution for Soperator
- D) Self-deploy on-premises

---

**Q24.** You want GPU worker nodes in Managed Service for Soperator. What must you have beforehand?

- A) A Kubernetes cluster
- B) Capacity block groups that reserve GPUs
- C) A pre-configured InfiniBand fabric
- D) An MLflow cluster

---

**Q25.** A Serverless AI endpoint differs from a Serverless AI job in which key way?

- A) Endpoints use GPUs; jobs use CPUs only
- B) Endpoints have a public URL and stay running; jobs terminate after completion
- C) Endpoints are cheaper than jobs
- D) Jobs support InfiniBand; endpoints do not

---

**Q26.** MLflow Managed Service is NOT available in which regions?

- A) `eu-north1` and `eu-west1`
- B) `eu-north2` and `eu-west1`
- C) `us-central1` and `me-west1`
- D) All regions support MLflow

---

**Q27.** During a distributed training job, you want to save checkpoints frequently for fault tolerance. After training completes, you want cost-efficient long-term storage. What is the recommended two-stage approach?

- A) Local NVMe during training → delete after training
- B) Object Storage during training → shared filesystem after
- C) Shared filesystem during training → Object Storage after
- D) Network SSD during training → NRD after

---

**Q28.** Your PyTorch job uses 4 nodes with 8 GPUs each. What should `WORLD_SIZE` be set to?

- A) 4
- B) 8
- C) 32
- D) 16

---

**Q29.** You need to authenticate your Python script to access Nebius Object Storage using boto3. Which credential type do you use?

- A) Authorized key (IAM token)
- B) SSH key pair
- C) Access key (S3-compatible)
- D) MysteryBox payload directly

---

**Q30.** A Slurm job keeps failing with exit code 1 immediately after starting. What do you check first?

- A) `sinfo` for node availability
- B) `sacct -j <jobid>` for job accounting details and exit reason
- C) Object Storage bucket permissions
- D) InfiniBand link status

---

## Domain 4: Platform Automation & Maintenance

**Q31.** You want to store your Nebius Terraform state in Object Storage so your team can share it. Which backend type do you use in Terraform?

- A) `gcs`
- B) `azurerm`
- C) `s3`
- D) `nebius`

---

**Q32.** Which CLI command lists the available GPU platforms for your project?

- A) `nebius compute instance list --gpu`
- B) `nebius compute platform list`
- C) `nebius gpu list --project`
- D) `nebius compute gpu-cluster list-platforms`

---

**Q33.** A Terraform `apply` is failing because another engineer is running `apply` at the same time. What resolves this?

- A) Use a larger VM for Terraform
- B) Configure a remote backend with state locking
- C) Split the Terraform code into smaller modules
- D) Run Terraform with `--parallelism=1`

---

**Q34.** You want to visualize Nebius GPU metrics in your existing Grafana installation. Which Nebius Observability integration do you use?

- A) Export metrics to Object Storage and import to Grafana
- B) Use the built-in Nebius Grafana dashboards only
- C) Nebius Monitoring's Grafana integration
- D) Install the Nebius Grafana plugin manually

---

**Q35.** What is the correct order of operations for zero-downtime maintenance on a Kubernetes GPU node?

- A) Delete node → recreate → uncordon
- B) Drain → perform maintenance → uncordon
- C) Cordon → perform maintenance → uncordon
- D) Cordon → drain → perform maintenance → uncordon

---

**Q36.** After installing the NVIDIA Network Operator, how do you verify it installed correctly?

- A) Run `ibstat` on each node
- B) Check `kubectl get nicclusterpolicy.mellanox.com nic-cluster-policy -n nvidia-network-operator -o json | jq -r '.status'` — state should be "ready"
- C) Run `nvidia-smi` on each node
- D) Check `kubectl get nodes` for Ready status

---

**Q37.** You change the `--infiniband-fabric` field in a Terraform resource for a GPU cluster. What does Terraform do?

- A) Updates the cluster in-place
- B) Ignores the change since fabrics cannot be changed
- C) Destroys and recreates the GPU cluster
- D) Outputs a warning but applies no change

---

**Q38.** You need to check the NVIDIA driver installation logs in a Kubernetes GPU node group. Which command do you run?

- A) `kubectl logs -n kube-system nvidia-device-plugin`
- B) Loop over `nvidia-driver-daemonset` pods in `nvidia-gpu-operator` namespace and check last log line
- C) `kubectl exec -n nvidia-gpu-operator -- nvidia-smi`
- D) `journalctl -u nvidia-driver`

---

**Q39.** Which Nebius Observability tool would you use to find which microservice in a chain is causing a 3-second API response delay?

- A) Metrics and Alerts
- B) Logs
- C) Traces
- D) Audit Logs

---

**Q40.** A colleague says "I'll just commit the Terraform state file to Git so everyone has it." What is wrong with this approach?

- A) Git does not support large files
- B) The state file may contain sensitive values and has no locking — concurrent applies can corrupt it
- C) Terraform cannot read state from Git
- D) Nothing is wrong — it is a valid approach

---

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **C** | `auditors` can view certain resource types but have no access to data within them. `viewers` can view resources AND access data. |
| 2 | **B** | Access keys are used for AWS-compatible APIs like Object Storage. Authorized keys are for obtaining IAM tokens. |
| 3 | **B** | Hierarchy is: Tenant > Project > Resource. Projects belong to one tenant and one region. |
| 4 | **B** | Being on the tenant user list does NOT grant access. Users must be added to a group. |
| 5 | **B** | Versioning allows secret rotation by creating a new version and setting it as primary, without breaking existing references. |
| 6 | **C** | NRD and IO M3 encryption is optional. Standard Network SSD is always encrypted by default. |
| 7 | **B** | Audit Logs support viewing, filtering, and exporting events to Object Storage. They capture all API calls, not just failures. |
| 8 | **C** | Budget alerts are notifications only. Resources continue running unless you explicitly take action. |
| 9 | **C** | `fabric-7` hosts H200 (`gpu-h200-sxm`) GPUs in `eu-north1`. `fabric-5` is H200 but in `eu-west1`. |
| 10 | **B** | GPUDirect RDMA lets data flow directly between each GPU and its NIC without CPU involvement. |
| 11 | **C** | VMs can only join a GPU cluster at creation time. You cannot add an existing VM to a cluster. |
| 12 | **B** | Nebius uses InfiniBand Partition Keys (P-Keys) to isolate traffic between clusters even on the same physical fabric. |
| 13 | **C** | H100 SXM only offers 1-GPU and 8-GPU presets. There is no 4-GPU H100 preset on Nebius. |
| 14 | **D** | Shared filesystem specs: max read bandwidth per client = 12 GiB/s, max write = 8 GiB/s. |
| 15 | **C** | Network SSD IO M3 is designed for performance-critical storage with reliability through 3-way replication. Nebius docs recommend it for GlusterFS. |
| 16 | **B** | L40S GPUs are PCIe-based with no InfiniBand connectivity. GPU clusters require InfiniBand-capable GPUs. |
| 17 | **C** | B200 GPUs require the Network Operator regardless of InfiniBand usage. Without the Nebius boot image, the GPU Operator is also required. |
| 18 | **C** | Changing the driver preset triggers node recreation: Managed Kubernetes creates replacement nodes, then cordons, drains, and deletes existing nodes. |
| 19 | **B** | `get-compatibility-matrix` returns supported driver presets and OS combinations for a given K8s version and GPU platform. |
| 20 | **C** | NRD disks have no redundancy. The docs explicitly state: "If a disk fails, its data will be lost." |
| 21 | **C** | All VMs in a GPU cluster — including Managed Kubernetes nodes — must be in the same project. |
| 22 | **D** | Each of 8 GPUs connects via a NIC at 400 Gbps. 8 × 400 Gbps = 3.2 Tbps total per node. |
| 23 | **C** | Pro Solution for Soperator is the expert-run, enterprise option with reserved capacity and discounted pricing. |
| 24 | **B** | GPU worker nodes in Managed Service for Soperator are only available if you have capacity block groups reserving GPUs. |
| 25 | **B** | Endpoints are interactive, have a public URL, and run until terminated. Jobs are non-interactive and terminate on task completion. |
| 26 | **B** | MLflow is available in all Nebius regions except `eu-north2` and `eu-west1`. |
| 27 | **C** | Docs recommend: shared filesystem for fast checkpoint saves during training, then async transfer to Object Storage for long-term storage. |
| 28 | **C** | WORLD_SIZE = total number of processes = nodes × GPUs per node = 4 × 8 = 32. |
| 29 | **C** | Object Storage uses AWS S3-compatible API. Authentication requires access keys. |
| 30 | **B** | `sacct` shows completed/failed job details including exit codes and reasons. |
| 31 | **C** | Nebius Object Storage is S3-compatible, so you use the `s3` Terraform backend with the Nebius Object Storage endpoint. |
| 32 | **B** | `nebius compute platform list` returns available platforms and presets for the current project. |
| 33 | **B** | Remote backends with state locking prevent concurrent applies and protect the state file from corruption. |
| 34 | **C** | Nebius Monitoring provides a Grafana integration that gives access to the full range of metrics for visualization. |
| 35 | **D** | Cordon first (stops new scheduling), then drain (evicts existing pods), then maintain, then uncordon. |
| 36 | **B** | The NICClusterPolicy status is the official way to verify Network Operator installation. `state-OFED` must show "ready". |
| 37 | **C** | InfiniBand fabric is an immutable field. Changing it forces resource replacement (destroy + recreate). |
| 38 | **B** | Loop over pods matching `nvidia-driver-daemonset` in `nvidia-gpu-operator` namespace — if last log line says "Done, now waiting for signal", driver installed correctly. |
| 39 | **C** | Distributed Traces show end-to-end request flow across services and are the right tool for identifying latency in specific spans. |
| 40 | **B** | Terraform state can contain sensitive data. Git has no locking mechanism — concurrent applies can cause state corruption. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Excellent — exam ready |
| 30–35 | Good — review weak domains |
| 24–29 | Fair — focused study needed |
| <24 | Re-read notes and docs |
