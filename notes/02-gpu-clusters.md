# Domain 2: Setting Up & Operating GPU Clusters (~35%)

This is the heaviest domain — 35% of the exam. Focus here first.

---

## GPU Instance Families

| GPU | Use Case |
|-----|---------|
| H100 SXM | Large-scale distributed training, flagship |
| A100 80GB | Training and inference, high memory |
| H200 | Latest gen, very large models |

Nebius does **not** offer V100 or older generation GPUs.

Each instance has a fixed GPU count — you select the platform and count at creation time. GPU count and platform type are **immutable** (changing them forces VM recreation in Terraform).

---

## InfiniBand

**What it is:** High-speed, low-latency network fabric for GPU-to-GPU communication.

**What it provides over Ethernet:**
- RDMA (Remote Direct Memory Access) — bypasses CPU, GPU writes directly to remote GPU memory
- Much lower latency (microseconds vs milliseconds)
- Higher bandwidth for collective operations (AllReduce, AllGather)

**When you need it:** Any distributed training across multiple nodes. Single-node multi-GPU jobs use NVLink instead.

**Validating IB links:**
```bash
ibstat          # Check port state and link speed
ibstatus        # Summary of all IB ports
ibdiagnet       # Full fabric diagnostics
ibcheckerrors   # Check for link errors/flapping
```

**IB link flapping:** Check `dmesg` and `ibcheckerrors` — usually a cable or switch port issue.

---

## Kubernetes GPU Clusters

### Setting Up GPU Node Groups

1. Create a Managed Kubernetes cluster
2. Add a GPU node group (select GPU platform, count per node)
3. Deploy the **NVIDIA device plugin** daemonset — this is what exposes `nvidia.com/gpu` resources to the scheduler
4. (Optional) Deploy GPU Operator for full lifecycle management

**If pods stay Pending after node is Ready:** NVIDIA device plugin is missing or not running.

### Key kubectl Commands for GPU Ops

```bash
# Check node GPU resources
kubectl describe node <node> | grep nvidia

# Check device plugin logs
kubectl logs -n kube-system -l name=nvidia-device-plugin-ds

# Verify nvidia-smi on the node
kubectl debug node/<node> -it --image=nvidia/cuda:12.0-base -- nvidia-smi

# Cordon — mark unschedulable, keep running pods
kubectl cordon <node>

# Drain — evict pods AND mark unschedulable
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# Uncordon — bring node back
kubectl uncordon <node>
```

**cordon vs drain:**
- `cordon` = no new pods, existing pods stay running (use for monitoring a suspect node)
- `drain` = no new pods + evict existing pods (use for maintenance)

### Topology-Aware Scheduling

Schedule pods on nodes sharing the same InfiniBand switch to minimize hop count and latency. Configured via node labels and pod affinity/anti-affinity rules.

---

## Storage Types

| Type | Use Case | Multi-node? |
|------|---------|-------------|
| Network SSD (block disk) | High-IOPS single-node storage | No |
| Network HDD (block disk) | Large, low-cost datasets on one node | No |
| Local NVMe | Fastest possible I/O, ephemeral | No |
| Shared filesystem | Dataset/checkpoint access across nodes | Yes |
| Object Storage | Model artifacts, large datasets, S3-compatible | Yes (via API) |

**Rule:** If multiple nodes need concurrent access → shared filesystem.
**Rule:** If one node needs fast random I/O → network SSD.
**Rule:** Long-term artifact storage → Object Storage (S3-compatible).

---

## VPC & Networking

- GPU cluster nodes should live in a **private subnet**
- Use a **NAT gateway** for outbound internet (pulling images) without public IPs
- Security groups control inbound/outbound traffic between nodes
- InfiniBand operates at the fabric level, separate from VPC networking

---

## Node Troubleshooting

| Symptom | First Check |
|---------|------------|
| Node NotReady | `nvidia-smi`, device plugin logs |
| Pods Pending on Ready node | NVIDIA device plugin not deployed |
| High GPU memory, slow training | Memory-bound model, check `nvidia-smi` |
| IB link flapping | `dmesg`, `ibcheckerrors`, `ibstat` |
| GPU ECC memory errors | Hardware fault → drain node, open support ticket |
| Node repeatedly restarting | `dmesg`, kernel logs, GPU error counters |

---

## Exam Traps

- NVIDIA device plugin ≠ GPU driver (driver is pre-installed on GPU nodes, plugin is a K8s component)
- `cordon` does NOT evict pods; `drain` does
- Shared filesystem for multi-node, block disk for single-node
- InfiniBand is for multi-node; NVLink is for intra-node GPU communication
- ECC memory errors = hardware fault, not a software fix
