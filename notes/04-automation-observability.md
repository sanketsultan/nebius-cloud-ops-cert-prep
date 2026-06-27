# Domain 4: Platform Automation & Maintenance (~25%)

---

## Terraform

Nebius has its own Terraform provider — source is `nebius/nebius`.

```hcl
terraform {
  required_providers {
    nebius = {
      source  = "nebius/nebius"
    }
  }
}

provider "nebius" {}
```

**Not** `yandex`, not `aws`, not `google` — it is `nebius`.

### Authentication

Authenticate as a service account or user account. Service account is recommended for CI/CD. Uses authorized keys (IAM token) or access keys depending on the resource.

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
resource "nebius_iam_v1_service_account" "sa" { ... }
resource "nebius_iam_v1_group_membership" "editors" { ... }
resource "nebius_kubernetes_cluster" "k8s" { ... }
resource "nebius_kubernetes_node_group" "gpu_nodes" { ... }
resource "nebius_storage_disk" "data_disk" { ... }
resource "nebius_vpc_network" "net" { ... }
resource "nebius_vpc_subnet" "subnet" { ... }
```

### Why Plan Shows Destroy + Recreate

When Terraform wants to destroy and recreate unexpectedly:
- An **immutable field** was changed (e.g. GPU platform, boot disk type, zone, GPU cluster ID)
- Immutable fields cannot be updated in-place — resource must be recreated
- Check `terraform plan` output for `# forces replacement` annotation

---

## Nebius CLI

Pattern: `nebius <service> <resource> <verb> [flags]`

```bash
# Compute
nebius compute instance list
nebius compute instance start --id <id>
nebius compute instance stop --id <id>
nebius compute instance get --id <id>
nebius compute gpu-cluster list
nebius compute gpu-cluster create --name <name> --infiniband-fabric <fabric>
nebius compute gpu-cluster delete <id>
nebius compute platform list   # List available GPU platforms

# Kubernetes
nebius mk8s cluster list
nebius mk8s cluster get-credentials --id <id> --external
nebius mk8s node-group create ...
nebius mk8s node-group update --id <id> --template-gpu-settings-drivers-preset cuda12.8
nebius mk8s node-group get-compatibility-matrix --cluster-kubernetes-version 1.33 --platform gpu-h200-sxm

# IAM
nebius iam service-account create --name <name>
nebius iam service-account delete --id <id>
nebius iam group get-by-name --name editors --parent-id <tenant_id>
nebius iam group-membership create --parent-id <group_id> --member-id <sa_id>

# Profile
nebius profile update --parent-id <project_id>
cat ~/.nebius/config.yaml
```

---

## Observability Stack

### Metrics and Alerts (Monitoring)

Three ways to access metrics:
1. **Preconfigured dashboards** in the web console — key metrics per resource out of the box
2. **Prometheus integration** — full metrics range via Prometheus queries
3. **Grafana integration** — visualize and filter metrics

**Alerts:** Set threshold-based alerts that trigger when metrics cross configured values.

**Key GPU metrics:**
- `gpu_utilization` — compute usage %
- `gpu_memory_used` — VRAM usage
- `gpu_temperature` — thermal throttling risk

### Logs

- Collects logs from VMs and Kubernetes pods
- Supports search, filtering, and export
- Custom log ingestion via API

**Audit Logs vs Observability Logs:**
- Audit Logs = who did what to Nebius resources (control plane, API calls)
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
# 1. Stop new scheduling
kubectl cordon <node>

# 2. Evict existing pods gracefully
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# 3. Perform maintenance (driver update, OS patch, etc.)

# 4. Bring node back
kubectl uncordon <node>
```

### Changing Driver Preset (Triggers Node Recreation)

```bash
nebius mk8s node-group update \
  --id <group_id> \
  --template-gpu-settings-drivers-preset cuda13.0
```

**Warning:** Managed Kubernetes recreates all nodes when you change the driver preset — it cordons, drains and deletes existing nodes per the group's deployment strategy.

### Rolling Node Group Upgrade (Zero Downtime)

Do one node at a time:
1. Cordon node
2. Drain node
3. Perform upgrade
4. Uncordon node
5. Verify GPUs healthy (`nvidia-smi`, check GPU operator logs)
6. Move to next node

Never upgrade all nodes simultaneously.

---

## Diagnosing Common Issues

### GPU Node NotReady

```bash
# On the node
nvidia-smi                          # Are GPUs visible?
dmesg | grep -i nvidia              # Driver errors?
systemctl status kubelet            # Is kubelet running?

# From kubectl
kubectl describe node <node>        # Events and conditions
kubectl get pods -n nvidia-gpu-operator  # GPU operator running?
```

### InfiniBand Issues

```bash
ibstat               # Check port state (should be Active, not Down)
ibstatus             # Summary
ibcheckerrors        # Count link errors
ibdiagnet            # Full fabric scan
dmesg | grep ib      # Kernel IB events
```

### Storage IOPS Bottleneck

Signs: training throughput lower than expected, high disk wait
```bash
iostat -x 1    # Check %util and await
iotop          # Which process is doing I/O
```

Resolution: upgrade to Network SSD IO M3, or pre-load data to shared filesystem before training.

### NCCL Network Operator Verification (after install)

```bash
kubectl get nicclusterpolicy.mellanox.com nic-cluster-policy \
  -n nvidia-network-operator -o json | jq -r '.status'
# state should be "ready"
```

---

## Exam Traps

- Terraform provider is `nebius/nebius`, not `yandex`
- `cordon` does NOT evict pods; `drain` does
- Traces ≠ Logs — traces show distributed request flow, logs show text output
- Audit Logs = control-plane events; Observability Logs = workload events
- State locking requires remote backend — local state has no locking
- Changing driver preset **recreates all nodes** in the group
- Rolling upgrades: one node at a time, not all at once
- Monitoring has Prometheus AND Grafana integrations — not just dashboards
- `nebius compute platform list` shows available GPU platforms for your project
