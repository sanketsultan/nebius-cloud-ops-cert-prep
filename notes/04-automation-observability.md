# Domain 4: Platform Automation & Maintenance (~25%)

---

## Terraform

Nebius has its own Terraform provider.

```hcl
terraform {
  required_providers {
    nebius = {
      source  = "nebius/nebius"
    }
  }
}

provider "nebius" {
  # credentials configured via env or config file
}
```

**Not** `yandex`, not `aws`, not `google` — it's `nebius`.

### Remote State (Required for Teams)

```hcl
terraform {
  backend "s3" {
    # Nebius Object Storage is S3-compatible
    bucket   = "my-tf-state"
    key      = "nebius/gpu-cluster/terraform.tfstate"
    endpoint = "https://storage.eu-north1.nebius.cloud"
    region   = "eu-north1"
  }
}
```

Benefits: shared state across team, state locking prevents concurrent applies.

### Common Resource Types

```hcl
resource "nebius_compute_instance" "gpu_vm" { ... }
resource "nebius_kubernetes_cluster" "k8s" { ... }
resource "nebius_kubernetes_node_group" "gpu_nodes" { ... }
resource "nebius_storage_disk" "data_disk" { ... }
resource "nebius_vpc_network" "net" { ... }
resource "nebius_vpc_subnet" "subnet" { ... }
```

### Why Plan Shows Destroy + Recreate

When Terraform wants to destroy and recreate a resource unexpectedly:
- An **immutable field** was changed (e.g., GPU platform, boot disk type, zone)
- Immutable fields cannot be updated in-place — the resource must be recreated
- Check `terraform plan` output for `# forces replacement` annotation

---

## Nebius CLI

```bash
# List compute instances
nebius compute instance list

# Start / stop a VM
nebius compute instance start --id <id>
nebius compute instance stop --id <id>

# Get VM details
nebius compute instance get --id <id>

# List Kubernetes clusters
nebius kubernetes cluster list

# Get IAM service account
nebius iam service-account get --id <id>
```

Pattern: `nebius <service> <resource> <verb> [flags]`

---

## Observability Stack

### Metrics and Alerts

- Collects infrastructure metrics: CPU, GPU utilization, memory, disk, network
- Create threshold alerts (e.g., GPU memory > 90% → notify)
- Notification channels: email, webhooks
- GPU metrics available per-node, per-GPU

**Key GPU metrics:**
- `gpu_utilization` — compute usage %
- `gpu_memory_used` — VRAM usage
- `gpu_temperature` — thermal throttling risk

### Logs

- Collects application and system logs from VMs and Kubernetes pods
- Supports log search, filtering, and export
- Custom log ingestion via API

**Audit Logs vs Observability Logs:**
- Audit Logs = who did what to Nebius resources (control plane actions)
- Observability Logs = what's happening inside your workloads (data plane)

### Traces

- Distributed tracing for microservices and APIs
- OpenTelemetry compatible
- Shows request flow end-to-end across services
- Use when debugging slow API requests spanning multiple services

---

## Node Maintenance Workflow

### Planned Maintenance (Your Own)

```bash
# 1. Cordon to stop new scheduling
kubectl cordon <node>

# 2. Drain to evict existing pods gracefully
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# 3. Perform maintenance (driver update, OS patch, etc.)

# 4. Bring node back
kubectl uncordon <node>
```

### Rolling Node Group Upgrade (Zero Downtime)

Do one node at a time:
1. Cordon node
2. Drain node
3. Perform upgrade (e.g., driver update)
4. Uncordon node
5. Verify pods rescheduled and GPUs healthy
6. Move to next node

Never upgrade all nodes simultaneously — this causes full cluster outage.

---

## Diagnosing Common Issues

### GPU Node NotReady

```bash
# On the node
nvidia-smi                    # Are GPUs visible?
dmesg | grep -i nvidia        # Driver errors?
systemctl status kubelet      # Is kubelet running?

# From kubectl
kubectl describe node <node>  # Events and conditions
kubectl get pods -n kube-system | grep nvidia  # Device plugin running?
```

### Storage IOPS Bottleneck

Signs: training throughput lower than expected, high disk wait time
```bash
iostat -x 1          # Check %util and await
iotop                # Which process is doing I/O
```

Resolution: upgrade to network SSD, or pre-load data to local NVMe before training.

### InfiniBand Issues

```bash
ibstat               # Check port state (should be Active)
ibstatus             # Summary
ibcheckerrors        # Count link errors
ibdiagnet            # Full fabric scan
dmesg | grep ib      # Kernel IB events
```

---

## Exam Traps

- Terraform provider is `nebius`, not `yandex`
- `cordon` ≠ `drain` — cordon doesn't evict pods
- Traces ≠ Logs — traces show request flow, logs show text output
- Audit Logs are control-plane events, not application logs
- State locking requires remote backend — local state has no locking
- Rolling upgrades: one node at a time, not all at once
