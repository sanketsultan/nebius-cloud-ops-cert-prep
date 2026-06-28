# Nebius AI Cloud Ops — Mock Exam 2 (Scenario-Based)

**Format:** Every question is a scenario. Pick the best action or answer.
**Time:** ~55 minutes | **Pass mark:** 70% (28/40)

> Attempt all questions before checking the answer key at the bottom.

---

## Domain 1: Security, Compliance & Billing (Q1–Q8)

**Q1.** Your team discovered that a shared service account key has been used by 5 different engineers for 6 months across CI/CD, Terraform, and data pipelines. One engineer left the company last week. What is the most secure remediation?

- A) Rotate the shared key immediately and share the new one with the remaining 4 engineers
- B) Delete the shared service account entirely and create separate service accounts per pipeline/use case, each with its own key and minimum required permissions
- C) Move the shared key to MysteryBox so it's securely stored but still shared
- D) Add MFA to the shared service account

---

**Q2.** A Terraform pipeline creates Nebius resources using a service account. The pipeline runs in a CI/CD system. Which authentication method should the service account use to authenticate to Nebius resources, and which to use the Nebius API?

- A) Access key for everything — it's the most universal
- B) Authorized key to get an IAM token for the Nebius API; access key for S3-compatible services like Object Storage
- C) SSH key for the Nebius API; access key for Object Storage
- D) Personal user credentials shared with the CI/CD system

---

**Q3.** You are doing a security review and find this in a Kubernetes pod spec:

```yaml
containers:
  - name: trainer
    image: trainer:latest
    env:
      - name: NEBIUS_ACCESS_KEY
        valueFrom:
          configMapKeyRef:
            name: nebius-config
            key: access_key
```

What security risk does this introduce and what is the better approach?

- A) No risk — ConfigMaps are encrypted in etcd by default
- B) ConfigMaps are not designed for sensitive data and are stored in plaintext in etcd. Store the access key in MysteryBox and fetch it at runtime, or use a Kubernetes Secret (though Secrets also need encryption-at-rest enabled separately).
- C) The risk is the `latest` tag — use a pinned image version
- D) Access keys should not be in environment variables at all — use instance metadata instead

---

**Q4.** Your company policy requires that all activity affecting GPU cluster resources be logged and retained for 1 year. Nebius Audit Logs in the console only show the last 90 days. How do you meet the 1-year retention requirement?

- A) Enable extended Audit Log retention in Nebius settings
- B) Set up automated Audit Log export to an Object Storage bucket on a schedule, and configure a lifecycle policy on the bucket to retain objects for 1 year
- C) Use Observability Logs with a 1-year retention filter
- D) This is not possible — Nebius only retains logs for 90 days

---

**Q5.** A MysteryBox secret has 3 versions: v1 (old API key), v2 (current API key, set as primary), v3 (new API key, just created but not yet set as primary). Your application is currently using v2. You set v3 as primary. What happens to the application?

- A) The application immediately starts using v3 on the next API call that fetches the secret
- B) The application keeps using v2 until it restarts
- C) All 3 versions are deleted and only v3 remains
- D) v2 is automatically deleted when v3 becomes primary

---

**Q6.** You need to grant a third-party ML vendor read-only access to model artifacts in one specific Object Storage bucket. They should not see any other resources. What is the correct Nebius approach?

- A) Add their user to the `viewers` group — this gives read access to all resources
- B) Create a service account scoped to your project, add it to `viewers`, and share the access key with the vendor — they can then access only resources within that project
- C) Make the Object Storage bucket public
- D) Create a pre-signed URL that expires after 24 hours

---

**Q7.** You need to encrypt sensitive data before storing it in Object Storage. You want to use envelope encryption where a Data Encryption Key (DEK) is wrapped by a master key. Which Nebius service manages the master key?

- A) MysteryBox — it stores all types of keys
- B) KMS (Key Management Service) — it manages cryptographic keys including symmetric keys used for envelope encryption
- C) IAM — access control handles encryption
- D) Audit Logs — encryption metadata is tracked there

---

**Q8.** A team member runs `terraform destroy` on your production GPU cluster by mistake. You need to identify who did it and when. Where do you look?

- A) Terraform state file — it records who ran each command
- B) Audit Logs — filter by `eventType: DeleteGpuCluster` and check the `subject` field for the actor's identity and timestamp
- C) Object Storage access logs for the state file bucket
- D) Monitoring Metrics — check for a sudden drop in GPU VM count

---

## Domain 2: Setting Up & Operating GPU Clusters (Q9–Q22)

**Q9.** You have two teams sharing a Nebius tenant. Team A needs H100 clusters in `eu-north1`. Team B needs H200 clusters in `eu-north1`. Can both teams share the same InfiniBand fabric and what are the implications?

- A) No — H100 and H200 use different fabrics in `eu-north1`. H100 uses fabric-2/3/4/6, H200 uses fabric-7. Each team uses its own fabric with P-Key isolation between their clusters.
- B) Yes — all GPUs share `fabric-7` in `eu-north1`
- C) No — different GPU types cannot be in the same region
- D) Yes — teams can share the same fabric and the same P-Key

---

**Q10.** You create a GPU cluster and 8-GPU H200 VMs. Two weeks later, management asks you to add a 9th VM. You try to create a new H200 VM and add it to the existing cluster but get an error. What is the correct procedure?

- A) Use `nebius compute gpu-cluster add-vm --id <cluster_id> --vm-id <new_vm_id>`
- B) VMs can only be added to a GPU cluster at creation time. Create a new H200 VM and specify the cluster ID during VM creation — it will join at boot.
- C) Stop the cluster, add the VM, then restart
- D) Create a new VM without specifying a cluster — it joins automatically based on InfiniBand fabric

---

**Q11.** You are setting up a multi-node training cluster and want to validate full InfiniBand bandwidth before submitting a 48-hour job. What sequence of checks do you run?

- A) `nvidia-smi`, `kubectl get nodes`, submit a test job
- B) `ibstat` (check Active state on all ports) → `ibcheckerrors` (check for existing errors) → run an NCCL bandwidth test (`nccl-tests`) across all nodes → only then submit the job
- C) `ping` between nodes, then submit the job
- D) `kubectl describe node` for each node and look for IB resources

---

**Q12.** A node in your training cluster shows `Ready` in Kubernetes but all GPU training pods scheduled on it immediately exit with `CrashLoopBackOff`. Other nodes work fine. `nvidia-smi` on the node shows GPUs. What do you investigate?

- A) The pod image is wrong — check the image pull logs
- B) Check `dmesg | grep -i nvidia` for GPU errors, check GPU Operator driver daemonset logs for "Done, now waiting for signal", and check `nvidia-smi -q | grep -i ecc` for memory errors on this specific node
- C) The node's InfiniBand link is down — check `ibstat`
- D) The node's disk is full — check `df -h`

---

**Q13.** You need to create a Kubernetes node group for H100 GPUs. Your cluster runs Kubernetes 1.33. Before creating the node group, you want to confirm that `cuda13.0` is a valid driver preset for this K8s version. What command do you run and what do you look for in the output?

- A) `nebius compute platform list` — look for `cuda13.0` in the GPU platforms section
- B) `nebius mk8s node-group get-compatibility-matrix --cluster-kubernetes-version 1.33 --platform gpu-h100-sxm` — look for `"driversPreset": "cuda13.0"` in the items list
- C) `kubectl get runtimeclass` — check for CUDA versions
- D) Check the Nebius docs manually — there's no CLI command for this

---

**Q14.** Your data engineering team wants to use Network SSD NRD disks to store raw training data (2.5 TB). They want to provision exactly 2.5 TB (2,560 GiB). They call you before provisioning. What do you tell them?

- A) 2,560 GiB is fine — NRD supports any size
- B) NRD disk sizes must be multiples of 93 GiB. 2,560 ÷ 93 = 27.5 — not valid. The nearest valid sizes are 93×27=2,511 GiB or 93×28=2,604 GiB. Also warn them: NRD has no redundancy — if the disk fails, all 2.5 TB is lost permanently.
- C) NRD maximum size is 1 TiB — they need to use multiple disks
- D) 2,560 GiB is fine but they need to enable encryption manually

---

**Q15.** You have a shared filesystem attached to 20 training nodes. A new team member in a different Nebius project asks to mount the same filesystem on their VMs. What do you tell them?

- A) Sure — shared filesystems can be accessed cross-project using their filesystem ID
- B) Not possible — Nebius shared filesystems can only be attached to VMs within the same project. They need to create their own filesystem in their project.
- C) Possible with a cross-project IAM policy
- D) Possible if both projects are in the same tenant and region

---

**Q16.** You need to run a training job across 16 L40S GPU nodes. A colleague suggests using a GPU cluster with InfiniBand for maximum bandwidth. Is this the right approach for L40S?

- A) Yes — L40S supports InfiniBand with `fabric-4`
- B) No — L40S is a PCIe-only GPU with no InfiniBand support. You cannot put L40S in a GPU cluster. For tightly-coupled multi-node training requiring InfiniBand, use H100, H200, B200, or B300.
- C) Yes — InfiniBand support depends on the fabric chosen, not the GPU
- D) No — L40S only supports single-node training

---

**Q17.** After a planned maintenance window, you uncordon a GPU node. Pods are scheduled on it but immediately show GPU-related errors. You SSH to the node and run the GPU driver log check:

```bash
kubectl logs -n nvidia-gpu-operator \
  $(kubectl get pods -n nvidia-gpu-operator | grep nvidia-driver-daemonset | grep node-05 | awk '{print $1}') --tail 3
```

Output:
```
[INFO] Installing NVIDIA driver version 580.x
[INFO] Compiling kernel modules...
[INFO] Waiting for driver compilation...
```

What should you do?

- A) Restart the driver daemonset pod — it's stuck
- B) Wait — the driver is still compiling kernel modules. It will complete and show "Done, now waiting for signal". Cordon the node again to prevent workloads until the driver is ready.
- C) Reinstall the GPU Operator
- D) The driver version is wrong — downgrade to `cuda12.8`

---

**Q18.** You want to deploy a Kubernetes cluster in Nebius and restrict the Kubernetes API endpoint so that only your office IP range `10.10.0.0/16` and your VPN `203.0.113.0/24` can connect. Which Terraform config is correct?

- A) Configure a VPC security group to block all traffic except those CIDRs
- B) In the `nebius_mk8s_v1_cluster` resource, set `public_endpoint = {}` and add `allowed_cidrs = ["10.10.0.0/16", "203.0.113.0/24"]` in the endpoints config
- C) Configure IAM to restrict login to those IP ranges
- D) Use a firewall rule on the subnet level

---

**Q19.** You have a 4-node H100 Kubernetes cluster. A new project requires 4 more nodes with H200 GPUs. Can you add H200 nodes to the existing H100 node group?

- A) Yes — different GPU types can share the same node group
- B) No — GPU platform is immutable per node group. Create a separate node group with `gpu-h200-sxm` platform and `fabric-7` for InfiniBand.
- C) Yes, but only if both platforms use the same InfiniBand fabric
- D) Yes — use `nebius mk8s node-group update --platform gpu-h200-sxm`

---

**Q20.** Your GPU cluster's InfiniBand diagnostics show `ibdiagnet` reporting link errors on the switch port connecting node-03. The errors are increasing over time. Training jobs using node-03 are slower than other nodes. What do you do?

- A) Restart the InfiniBand service on node-03: `systemctl restart opensm`
- B) Drain node-03 from the cluster to prevent slow jobs, and open a Nebius support ticket — switch port errors are infrastructure-level issues that require Nebius hardware team intervention
- C) Increase `NCCL_IB_TIMEOUT` to tolerate the link errors
- D) Replace the Network Operator on node-03

---

**Q21.** A colleague modifies a Terraform resource to change a GPU cluster's InfiniBand fabric from `fabric-7` to `fabric-2`. They say "I'll just run `terraform apply` — it'll do an in-place update." What actually happens?

- A) Terraform updates the fabric in-place — no downtime
- B) Terraform shows `# forces replacement` in the plan and destroys the cluster + all node VMs, then recreates everything. All running jobs are terminated. Warn them before they apply.
- C) Terraform ignores the fabric change and applies all other changes
- D) `terraform apply` fails because fabric changes are not supported via Terraform

---

**Q22.** Your cluster has 8 GPU nodes. You run `kubectl cordon` on all 8 nodes simultaneously to prevent new scheduling while you perform a cluster-wide configuration change. A new training job is submitted. What happens?

- A) The job starts on one of the cordoned nodes anyway — cordon is advisory only
- B) The job stays in `Pending` state with reason `0/8 nodes are available: 8 node(s) were unschedulable` until you uncordon at least enough nodes
- C) The job fails immediately — Kubernetes rejects job submissions when all nodes are cordoned
- D) The job runs on the control plane node as a fallback

---

## Domain 3: Running Training & Inference Workloads (Q23–Q30)

**Q23.** A team wants to fine-tune a 70B parameter model. During training, they save checkpoints to Object Storage directly from each of the 16 training nodes. After 12 hours, they notice checkpoint saves are taking 45 minutes each, causing training to stall during saves. What is the better checkpointing architecture?

- A) Increase Object Storage write throughput by using parallel multipart uploads
- B) Save checkpoints to a Nebius Shared Filesystem first (fast local network write), then asynchronously copy to Object Storage in the background. The shared filesystem write is fast; the S3 transfer happens without blocking training.
- C) Reduce checkpoint frequency to once every 24 hours
- D) Use a single "checkpoint node" that collects and saves for the other nodes

---

**Q24.** You're running a Slurm job and see it in `CG` (Completing) state for over 30 minutes. Other jobs are waiting in the queue. What is happening and what should you check?

- A) The job is almost done — `CG` means it's 90% complete
- B) `CG` (Completing) means the job finished but cleanup processes are still running (e.g., writing final output, MPI teardown). If it's stuck for 30 minutes, check for hung processes on the nodes with `squeue -j <jobid>` and look at node state with `sinfo`.
- C) The job failed — cancel it with `scancel`
- D) The job is waiting for a checkpoint to complete before exiting

---

**Q25.** You want to run a batch preprocessing job that processes 1 million images with a GPU. The job is expected to take 4 hours and should never need to be restarted once complete. You don't want to manage any infrastructure. Which Nebius service is most appropriate?

- A) Deploy a Kubernetes job on a managed K8s cluster
- B) Use Serverless AI Job — one-off, terminates on completion, billed per second, no infrastructure to manage
- C) Use Managed Service for Soperator with a Slurm batch job
- D) Use a Serverless AI Endpoint that you stop after 4 hours

---

**Q26.** Your distributed PyTorch training job runs successfully with 2 nodes (16 GPUs, WORLD_SIZE=16) but fails to initialize when scaled to 8 nodes (64 GPUs). The error is `Timeout waiting for processes to connect`. What is the most likely issue?

- A) `WORLD_SIZE` was not updated to 64 when scaling — the job is waiting for 16 processes but 64 are trying to connect
- B) `WORLD_SIZE=64` but `MASTER_ADDR` still points to node 0, and some of the new 6 nodes cannot reach it — check network connectivity between all 8 nodes and verify firewall/security group rules allow the `MASTER_PORT`
- C) The shared filesystem is too slow for 8 nodes
- D) NCCL does not support more than 32 GPUs per job

---

**Q27.** You want to run the Managed Service for Soperator in `eu-west1`. The console shows the option greyed out. Your colleague says you need to contact sales first. What is the actual most likely reason?

- A) Soperator requires a Pro license in `eu-west1`
- B) GPU worker nodes in Managed Soperator require capacity block groups reserving GPU capacity. Without pre-reserved capacity blocks for your account, the GPU worker option is unavailable.
- C) Soperator is not available in `eu-west1`
- D) You need to be on the Enterprise billing plan

---

**Q28.** A researcher reports that their MLflow experiment tracking UI is not showing new runs even though training jobs are running. The training script uses `mlflow.log_metric()` calls. What do you check first?

- A) Whether the MLflow managed cluster is in a region that supports it (not `eu-north2` or `eu-west1`)
- B) Whether `MLFLOW_TRACKING_URI` in the training script points to the correct MLflow endpoint URL
- C) Whether the training pod has network access to the MLflow service
- D) All of the above — check in this order: region support, then tracking URI, then network connectivity

---

**Q29.** Your inference endpoint serves a 13B parameter model. Average response time is 2 seconds for single requests. When 10 users send requests simultaneously, response time jumps to 25 seconds. GPU utilization is 30% and VRAM is 70% used. What is the bottleneck?

- A) VRAM is the bottleneck — the model is too large
- B) The endpoint is processing requests sequentially (batch size 1). Enable request batching in vLLM — it processes multiple requests in a single forward pass, significantly improving throughput under concurrent load.
- C) 10 concurrent users requires 10 GPU replicas
- D) GPU compute (30%) is the bottleneck — not enough Tensor core performance

---

**Q30.** A colleague suggests using `Serverless AI Endpoints` for a 72-hour non-stop model training job. What is wrong with this approach?

- A) Serverless AI Endpoints don't support training — only inference
- B) Endpoints are designed for interactive serving with a public URL. For a non-stop training job, use Managed Kubernetes with a training pod, Slurm via Soperator, or Serverless AI Jobs. An Endpoint would waste the public URL allocation and is not designed for job-style workloads.
- C) Endpoints cannot run for more than 24 hours
- D) Both A and B

---

## Domain 4: Platform Automation & Maintenance (Q31–Q40)

**Q31.** Your Terraform plan for creating a Nebius GPU cluster fails with:

```
Error: Provider produced inconsistent result after apply
```

After investigation, you find the Nebius Terraform provider version is 3 months old. What should you do and what process prevents this in the future?

- A) Pin the provider version to the current broken one in `required_providers` and submit a bug report
- B) Update the provider version in `required_providers`, run `terraform init -upgrade`, then `terraform plan` again. In future, use Dependabot or similar to track provider updates.
- C) Destroy and recreate all resources from scratch
- D) Switch to the Nebius CLI for cluster creation

---

**Q32.** You run `terraform plan` and see:

```
  ~ resource "nebius_mk8s_v1_cluster" "prod" {
      ~ control_plane = {
          ~ etcd_cluster_size = 3 -> 1
        }
    }
```

A colleague says this is fine — "we're just reducing etcd for cost savings." What do you tell them?

- A) They're right — reducing from 3 to 1 etcd saves resources with no downside
- B) Reducing to 1 etcd disables HA — a single etcd failure takes down the entire Kubernetes control plane. Any control plane node maintenance or failure means cluster downtime. Strongly advise against this for production.
- C) This change forces cluster recreation — it's not an in-place update
- D) Etcd changes don't affect cluster availability

---

**Q33.** A production GPU cluster was manually modified in the Nebius console (someone changed a label). Now when you run `terraform plan`, it shows a diff for the label change and wants to revert it. How do you handle this correctly?

- A) Run `terraform apply` to revert the manual change — Terraform should be the source of truth
- B) Update the Terraform config to match the new label (if it's intentional), then run `terraform plan` to confirm no diff. Always make config changes via Terraform to avoid drift.
- C) Run `terraform import` to re-import the resource
- D) Ignore the diff — Terraform diffs for labels are not applied

---

**Q34.** Your team's Observability dashboard shows this pattern for a training job:

```
gpu_utilization: 95% (0:00 to 2:15)
gpu_utilization: 12% (2:15 to 2:45)
gpu_utilization: 95% (2:45 to 5:00)
```

The dip at 2:15-2:45 repeats every 2.5 hours. What is most likely happening?

- A) The GPU is thermal throttling every 2.5 hours
- B) The training job is saving checkpoints every 2.5 hours — during checkpoint saves to the shared filesystem or Object Storage, training pauses and GPU utilization drops. If checkpoint saves take 30 minutes, consider async checkpointing.
- C) InfiniBand is having periodic link failures
- D) The training script has a deliberate cooldown period every 2.5 hours

---

**Q35.** You need to do a rolling upgrade of NVIDIA drivers on a 4-node GPU cluster with a live training job. The job can tolerate losing one node at a time (it checkpoints every 5 minutes). What is the correct procedure?

- A) Update the driver preset in Nebius and let Managed Kubernetes handle the rolling recreation automatically
- B) For each node in sequence: (1) cordon to stop new scheduling, (2) drain to evict the training pod (it will reschedule on other nodes), (3) update driver preset for this node individually, (4) wait for driver installation ("Done, now waiting for signal"), (5) uncordon and verify GPU resources appear, (6) move to the next node
- C) Stop the training job, upgrade all nodes simultaneously, restart the job
- D) Upgrade the driver on each node without draining — the training pod can survive a driver reinstall

---

**Q36.** A `terraform apply` fails halfway through creating 10 resources. 6 resources were created before the failure. What is the state of your Terraform state file and what should you do?

- A) The state file is empty — all 10 resources need to be recreated
- B) The state file contains the 6 successfully created resources. Run `terraform apply` again — Terraform will only try to create the remaining 4. Do not manually delete the 6 already-created resources.
- C) The state file is corrupted — restore from backup
- D) All 10 resources were rolled back automatically

---

**Q37.** Your team wants to receive a Slack notification when GPU memory utilization on any node exceeds 95% for more than 5 minutes. Which Nebius Observability components do you need to configure?

- A) Audit Logs → filter → webhook
- B) Monitoring Metrics → select `gpu_memory_used` metric → create threshold alert → set notification channel to a webhook that posts to Slack
- C) Observability Logs → create log alert → Slack integration
- D) Traces → set latency threshold → webhook

---

**Q38.** After deploying the NVIDIA Network Operator, you verify it with:

```bash
kubectl get nicclusterpolicy.mellanox.com nic-cluster-policy \
  -n nvidia-network-operator -o json | jq -r '.status.state'
```

Output: `notReady`

You wait 30 minutes and run it again. Still `notReady`. What do you investigate?

- A) The Network Operator needs to be reinstalled from scratch
- B) Check the specific component causing the failure: `kubectl get nicclusterpolicy -o json | jq '.status'` for details. Look at which sub-component (OFED, RDMA device plugin, etc.) shows `notReady`. Then check that component's pod logs in the `nvidia-network-operator` namespace.
- C) The `notReady` state is permanent — reinstall from the generic NVIDIA chart instead
- D) Increase the operator timeout with a Helm values override

---

**Q39.** You are investigating slow inference. You check Nebius Observability Traces and see:

```
Total request: 6,200ms
├── API Gateway:     18ms
├── Auth:            12ms  
├── Queue service:  5,800ms  ← ???
├── GPU Inference:   340ms
└── Response:         30ms
```

The Queue service has 5,800ms out of 6,200ms. What does this tell you and what do you do next?

- A) The GPU inference is the bottleneck — 340ms is too slow
- B) The Queue service is the clear bottleneck (94% of total time). Investigate the queue service: check if it's backlogged, under-provisioned, or experiencing a timeout waiting for a downstream dependency. This is where optimization will have the most impact.
- C) The API Gateway is misconfigured — 18ms is too slow for a gateway
- D) Add more GPU replicas — the queue will clear faster

---

**Q40.** A team member asks: "Can I store the Nebius Terraform state in a Git repository so the whole team can access it?" What is your response and what do you recommend instead?

- A) Yes — Git is a great way to share state files
- B) No — Terraform state can contain sensitive values (access keys, resource IDs, passwords) in plaintext. Git has no locking mechanism, so concurrent applies will corrupt the state. Use an Object Storage bucket (S3 backend) with state locking configured. Add the state file to `.gitignore`.
- C) Yes, but use a private repository with branch protection
- D) Yes, but encrypt the state file first with KMS

---

## Answer Key & Explanations

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Shared keys are a security anti-pattern. When someone leaves, you must rotate all their credentials across all systems. Individual service accounts with minimum permissions limit blast radius and enable clean offboarding. |
| 2 | **B** | Authorized keys generate IAM tokens for authenticating to the Nebius API (resource management). Access keys are for S3-compatible APIs like Object Storage. Different auth mechanisms for different purposes. |
| 3 | **B** | ConfigMaps are plaintext in etcd. Sensitive credentials like access keys should be in MysteryBox (Nebius-native) or at minimum Kubernetes Secrets with encryption-at-rest enabled. |
| 4 | **B** | Audit Log export to Object Storage with a bucket lifecycle policy is the standard pattern for meeting long-term retention requirements. The export must be set up proactively — you can't retroactively export old logs. |
| 5 | **A** | When a new version is set as primary, applications that fetch the secret on their next call will get the new version. Applications that cached the previous value won't update until they re-fetch. |
| 6 | **B** | Create a service account with `viewers` role scoped to your project. Share only the access key — the vendor can only access what the SA can access (project-scoped Object Storage). This is least-privilege for external parties. |
| 7 | **B** | KMS manages cryptographic master keys for envelope encryption. The DEK encrypts data; KMS wraps/unwraps the DEK. MysteryBox stores secret values, not cryptographic keys for encryption. |
| 8 | **B** | Audit Logs capture all API calls including resource deletions. They record the actor (subject), action (eventType), resource, and timestamp — exactly what you need for a post-incident investigation. |
| 9 | **A** | H100 and H200 use different fabrics in `eu-north1`. H100 uses fabric-2/3/4/6; H200 uses fabric-7. Clusters are isolated by P-Keys so sharing a fabric is fine — but each GPU type has its own fabric(s). |
| 10 | **B** | VMs join a GPU cluster only at creation time by specifying the cluster ID. You must create a new VM from scratch and specify `--gpu-cluster-id` during creation. There is no "add existing VM to cluster" operation. |
| 11 | **B** | `ibstat` confirms link health, `ibcheckerrors` finds existing issues, NCCL bandwidth tests validate end-to-end throughput. Only submit the 48-hour job after all checks pass. |
| 12 | **B** | If `nvidia-smi` works, the GPU hardware and driver are fine. `CrashLoopBackOff` immediately suggests a node-specific issue. Check dmesg for hardware errors, GPU Operator logs for incomplete driver install, and ECC errors for hardware faults. |
| 13 | **B** | `get-compatibility-matrix` is the right command. Look for `"driversPreset": "cuda13.0"` in the output items. If it's not listed, that preset is not compatible with the given K8s version and GPU platform. |
| 14 | **B** | NRD sizes must be multiples of 93 GiB. 2,560 is not. Also critical: warn about the no-redundancy risk. 2.5 TB of training data on an NRD disk is gone permanently if the disk fails. |
| 15 | **B** | Shared filesystems are scoped to a project. Cross-project sharing is not supported. Each project that needs shared storage must have its own shared filesystem. |
| 16 | **B** | L40S is PCIe-only, no InfiniBand support. It cannot join a GPU cluster. For tightly-coupled multi-node training needing InfiniBand, use H100/H200/B200/B300. |
| 17 | **B** | "Compiling kernel modules" means the driver is still installing. This is normal but can take time. Cordon the node to prevent workloads from being scheduled until the driver shows "Done, now waiting for signal". |
| 18 | **B** | In `nebius_mk8s_v1_cluster`, set `public_endpoint = {}` to enable the endpoint, then add CIDR restrictions in the endpoints configuration. The public endpoint must be enabled for CIDR-based access control to work. |
| 19 | **B** | GPU platform is immutable per node group. Create a separate node group with `gpu-h200-sxm`. Note: H200 also needs `fabric-7` for IB, while H100 uses `fabric-2/3/4/6` — they may need separate GPU clusters too. |
| 20 | **B** | Switch-level errors are infrastructure issues beyond your control. Drain the node to protect job performance, and raise a support ticket for Nebius hardware team to inspect/replace the switch port. |
| 21 | **B** | `infiniband_fabric` is immutable. Terraform destroys the entire cluster and all dependent VMs before recreating. This terminates all running jobs. Always plan maintenance windows for fabric changes. |
| 22 | **B** | Cordoning all nodes makes the cluster unable to schedule new pods. The job stays pending until at least enough nodes are uncordoned to satisfy its resource request. |
| 23 | **B** | Shared filesystem for checkpoint writes is fast (local network). Async background copy to Object Storage happens in parallel with training. This eliminates the blocking I/O that stalls training during saves. |
| 24 | **B** | `CG` (Completing) is a normal Slurm state but shouldn't last 30 minutes. Likely a hung cleanup process (MPI teardown, output flushing). Check node states and running processes on those nodes. |
| 25 | **B** | Serverless AI Job: one-off, GPU-accelerated, terminates on completion, billed per second, zero infra management. Perfect for a 4-hour batch processing task with no restart requirement. |
| 26 | **B** | At 2 nodes the network was reachable. At 8 nodes, some new nodes can't reach `MASTER_ADDR` on `MASTER_PORT`. Check VPC firewall rules, security groups, and network connectivity from all 8 nodes to node 0's port. |
| 27 | **B** | GPU workers in Managed Soperator require capacity block groups. This is a documented prerequisite — without pre-reserved GPU capacity blocks, the GPU worker option is unavailable in the console. |
| 28 | **D** | All three should be checked in sequence. Wrong region means MLflow doesn't exist. Wrong tracking URI means logs go nowhere. Network issues mean logs can't reach MLflow even if URI is correct. |
| 29 | **B** | Low GPU utilization + slow concurrent throughput = requests being processed serially. vLLM's continuous batching should handle concurrent requests in a single forward pass. Enable and tune batching settings. |
| 30 | **B** | Endpoints are designed for always-on interactive serving, not training jobs. Training jobs should use Kubernetes pods, Slurm, or Serverless AI Jobs (though Jobs are better for finite batch tasks than 72-hour continuous training). |
| 31 | **B** | Upgrading the provider version and reinitializing often fixes provider bugs. Pin versions with `~> x.y` to get patch updates but not breaking major updates. Use automation to track upstream provider releases. |
| 32 | **B** | Reducing etcd from 3 to 1 removes HA. With 1 etcd, any maintenance or failure takes down the control plane. For production, 3 etcd stores is essential. This change does not cost extra to keep at 3. |
| 33 | **B** | Terraform should be the single source of truth. If the manual change is intentional, update the Terraform config to match, not the other way. This prevents future drift and keeps the config accurate. |
| 34 | **B** | Regular drops in GPU utilization at consistent intervals strongly indicate checkpoint saves. 30-minute checkpoint saves every 2.5 hours is excessive — implement async checkpointing to write in the background without pausing compute. |
| 35 | **B** | With a live checkpointing job, sequential per-node drain-upgrade-uncordon-verify is the safest approach. The job migrates one node's workload at a time and continues on the remaining nodes. Never upgrade all at once. |
| 36 | **B** | Terraform applies changes atomically per resource but not across all resources in a single atomic transaction. The state reflects what was successfully created. Re-running `apply` is safe — it only creates what's missing. |
| 37 | **B** | Monitoring Metrics → `gpu_memory_used` threshold alert → notification channel (webhook to Slack). This is the full path for metric-based alerting in Nebius Observability. |
| 38 | **B** | `notReady` after 30 minutes means a component failed to initialize. Inspect the detailed status JSON to identify which sub-component is failing, then look at that component's pod logs for the specific error. |
| 39 | **B** | The trace clearly shows the Queue service consuming 5,800ms (94%) of total time. This is the bottleneck. Investigate queue depth, resource limits, and whether the queue service has downstream dependencies causing the wait. |
| 40 | **B** | Terraform state contains sensitive values in plaintext. Git has no locking. Both are critical reasons not to store state in Git. Use Object Storage as S3 backend with proper locking. Add `terraform.tfstate` to `.gitignore` permanently. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 36–40 | Exam ready |
| 30–35 | Close — focus on weak domains |
| 24–29 | More practice needed |
| <24 | Re-read notes with focus on applying knowledge |
