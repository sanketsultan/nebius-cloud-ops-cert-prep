# Domain 2: Setting Up & Operating GPU Clusters (~35%)

This is the heaviest domain — 35% of the exam. Focus here first.

---

## GPU Instance Families

| GPU | Platform ID | Architecture | VRAM | IB Speed | Regions |
|-----|------------|-------------|------|---------|---------|
| NVIDIA B300 SXM | `gpu-b300-sxm` | Blackwell Ultra | 288 GB HBM3e | 800 Gbps | `uk-south1`* |
| NVIDIA B200 SXM | `gpu-b200-sxm` | Blackwell | 180 GB HBM3e | 400 Gbps | `us-central1` |
| NVIDIA B200 SXM | `gpu-b200-sxm-a` | Blackwell | 180 GB HBM3e | 400 Gbps | `me-west1` |
| NVIDIA H200 SXM | `gpu-h200-sxm` | Hopper | 141 GB HBM3e | 400 Gbps | `eu-north1`, `eu-north2`*, `eu-west1`, `us-central1` |
| NVIDIA H100 SXM | `gpu-h100-sxm` | Hopper | 80 GB HBM3 | 400 Gbps | `eu-north1` |
| NVIDIA RTX PRO 6000 | `gpu-rtx6000` | Blackwell PCIe | 96 GB GDDR7 | 400 Gbps | `us-central1` |
| NVIDIA L40S PCIe | `gpu-l40s-a` | Ada Lovelace | 48 GB GDDR6 | None | `eu-north1` |
| NVIDIA L40S PCIe | `gpu-l40s-d` | Ada Lovelace | 48 GB GDDR6 | None | `eu-north1` |

*\* = private region*

**Nebius does NOT offer:** A100, V100, or any older generation GPUs.

**L40S has NO InfiniBand** — PCIe only, not suitable for tightly coupled multi-node training.

---

## Preset Naming Convention

Format: `<gpu_count>gpu-<vcpu_count>vcpu-<ram>gb`

Example: `8gpu-128vcpu-1600gb` = 8 GPUs, 128 vCPUs, 1600 GiB RAM

**GPU cluster compatible presets (only 8-GPU presets work in clusters):**

| Platform | Cluster Preset |
|---------|---------------|
| B300 (`gpu-b300-sxm`) | `8gpu-192vcpu-2768gb` |
| B200 (`gpu-b200-sxm` / `gpu-b200-sxm-a`) | `8gpu-160vcpu-1792gb` |
| H200 (`gpu-h200-sxm`) | `8gpu-128vcpu-1600gb` |
| H100 (`gpu-h100-sxm`) | `8gpu-128vcpu-1600gb` |

---

## InfiniBand GPU Clusters

**What it is:** High-speed RDMA network fabric connecting GPU nodes.

**Key facts from docs:**
- Each GPU in a cluster VM connects via NIC providing **400 Gbps** (800 Gbps for B300)
- 8 GPUs per VM = **3.2 Tbps total bandwidth per node**
- Uses **GPUDirect RDMA** — data flows directly between GPU and NIC, **bypassing CPU**
- InfiniBand traffic is isolated between clusters using **Partition Keys (P-Keys)**
- Each GPU cluster gets a unique P-Key — nodes in different clusters cannot communicate over IB even on same physical fabric

**InfiniBand Fabrics** (physical — you select one when creating a cluster):

| Fabric | GPU Platform | Region |
|--------|------------|--------|
| `fabric-2`, `fabric-3`, `fabric-4`, `fabric-6` | H100 (`gpu-h100-sxm`) | `eu-north1` |
| `fabric-5` | H200 (`gpu-h200-sxm`) | `eu-west1` |
| `fabric-7` | H200 (`gpu-h200-sxm`) | `eu-north1` |
| `eu-north2-a` | H200 (`gpu-h200-sxm`) | `eu-north2`* |
| `me-west1-a` | B200 (`gpu-b200-sxm-a`) | `me-west1` |
| `uk-south1-a` | B300 (`gpu-b300-sxm`) | `uk-south1`* |
| `us-central1-a` | H200 (`gpu-h200-sxm`) | `us-central1` |
| `us-central1-b` | B200 (`gpu-b200-sxm`) | `us-central1` |

**Important:** VMs can ONLY be added to a GPU cluster **at creation time** — you cannot add an existing VM to a cluster.

**All VMs in a GPU cluster must be in the same project.**

### CLI — Creating a GPU cluster
```bash
export GPU_CLUSTER_ID=$(nebius compute gpu-cluster create \
  --name my-cluster \
  --infiniband-fabric fabric-7 \
  --format json | jq -r ".metadata.id")
```

### Validating InfiniBand
```bash
ibstat              # Check port state (should be Active)
ibstatus            # Summary of all IB ports
ibdiagnet           # Full fabric diagnostics
ibcheckerrors       # Check for link errors/flapping
dmesg | grep ib     # Kernel IB events
```

---

## Storage Types

Three disk types in Nebius Compute:

| Type | ID | IOPS (max) | Bandwidth | Reliability | Best For |
|------|----|-----------|-----------|-------------|---------|
| Network SSD | `network_ssd` | 40,000 write / 20,000 read | 450 MiB/s | Erasure coding (2 failures) | Boot disks, Slurm controllers |
| Network SSD NRD | `network_ssd_non_replicated` | 75,000 | 1 GiB/s | None — data loss on failure | K8s worker nodes (transient data) |
| Network SSD IO M3 | `network_ssd_io_m3` | 75,000 | 1 GiB/s | 3x replication | GlusterFS, databases |

**NRD size must be a multiple of 93 GiB.** IO M3 also in multiples of 93 GiB.

**Encryption:**
- Network SSD: encrypted by **default**, cannot disable
- NRD and IO M3: encryption is **optional**

**Disk performance scales with size** — more allocation units = more IOPS/bandwidth until max.

### Shared Filesystems

- One filesystem can be **attached to multiple VMs simultaneously**
- All VMs must be in the same **project** (not shareable across projects)
- Mounted as **virtiofs** device on VMs
- Encrypted by **default** (cannot disable)
- Capacity: up to **5 PiB**
- Max read bandwidth per client: **12 GiB/s**
- Max write bandwidth per client: **8 GiB/s**
- Performance scales: every **4 TiB** of size adds more bandwidth

### Storage Use Cases by Workload

| Stage | Recommended Storage |
|-------|-------------------|
| VM boot disk | Network SSD |
| K8s worker node disk | Network SSD NRD |
| Database on VM | Network SSD IO M3 |
| Dataset storage | Object Storage |
| Streaming dataset to training | Shared filesystem (SSD) |
| Sharing code between workers | Shared filesystem |
| Checkpoints during training | Shared filesystem → Object Storage after |
| Sharing model weights for inference | Shared filesystem |
| Sharing inference results | Object Storage |

---

## Kubernetes GPU Clusters

### Setting Up GPU Node Groups

When creating a node group, specify:
- `--template-resources-platform` — GPU platform (e.g. `gpu-h100-sxm`)
- `--template-resources-preset` — preset (e.g. `8gpu-128vcpu-1600gb`)
- `--template-gpu-settings-drivers-preset` — CUDA driver preset

**Driver presets:**

| Preset | NVIDIA Driver | OS |
|--------|-------------|-----|
| `cuda12.8` | 570.x | Ubuntu 24.04 |
| `cuda13.0` | 580.x | Ubuntu 24.04 |

**Cannot change VM platform, preset, or GPU cluster of an existing node group** — create a new one instead.

### NVIDIA Operators (installed via Helm)

**NVIDIA GPU Operator** — required for any node group with GPUs that doesn't use the Nebius boot disk image.

**NVIDIA Network Operator** — required when:
- Node group uses **B200 GPUs**, OR
- Node group is in a **GPU cluster with InfiniBand**

Install from Nebius chart repo (not generic NVIDIA charts):
```bash
# Network Operator (for IB)
helm install network-operator \
  oci://cr.eu-north1.nebius.cloud/marketplace/nebius/nvidia-network-operator/chart/network-operator \
  --version 25.7.0 -n nvidia-network-operator --create-namespace --wait

# GPU Operator
helm install gpu-operator \
  oci://cr.eu-north1.nebius.cloud/marketplace/nebius/nvidia-gpu-operator/chart/gpu-operator \
  --version v25.10.0 -n nvidia-gpu-operator --create-namespace --wait
```

**GPUDirect RDMA is enabled by default** when using InfiniBand.

### Key kubectl Commands for GPU Ops

```bash
# Check node GPU resources
kubectl describe node <node> | grep nvidia

# Check GPU operator logs
kubectl logs -n nvidia-gpu-operator $(kubectl get pods -n nvidia-gpu-operator \
  | grep nvidia-driver-daemonset | head -1 | awk '{print $1}') --tail 5

# Cordon — mark unschedulable, keep running pods
kubectl cordon <node>

# Drain — evict pods AND mark unschedulable
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# Uncordon — bring node back
kubectl uncordon <node>
```

**cordon vs drain:**
- `cordon` = no new pods, existing pods stay (use for monitoring a suspect node)
- `drain` = no new pods + evict existing pods (use for maintenance)

---

## Node Troubleshooting

| Symptom | First Check |
|---------|------------|
| Node NotReady | `nvidia-smi`, GPU operator logs |
| Pods Pending on Ready node | GPU operator not deployed or not healthy |
| IB link flapping | `dmesg`, `ibcheckerrors`, `ibstat` |
| GPU ECC memory errors | Hardware fault → drain node, open support ticket |
| Low bandwidth between nodes | Check `NCCL_IB_DISABLE`, `ibstat` for Active state |

---

## Exam Traps

- **A100 is NOT in Nebius** — never was. Only B300, B200, H200, H100, RTX PRO 6000, L40S
- **L40S has no InfiniBand** — cannot be used in GPU clusters
- VMs can only join a GPU cluster **at creation time**
- All cluster VMs must be in the **same project**
- P-Keys isolate IB traffic between clusters on the same physical fabric
- NRD disk size must be **multiple of 93 GiB**
- Network SSD is **always encrypted**; NRD and IO M3 encryption is optional
- `cordon` does NOT evict pods; `drain` does
- Network Operator is needed for **B200 GPUs** even without IB
- Install operators from **Nebius chart repo**, not generic NVIDIA charts
