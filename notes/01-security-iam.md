# Domain 1: Security, Compliance & Billing (~20%)

---

## IAM Structure: Tenant > Project > Resource

- **Tenant** — your top-level workspace. Contains projects, users, billing, quotas, audit logs, and groups. Created automatically on sign-up. Cannot be deleted.
- **Project** — isolated workspace inside a tenant. Each project belongs to one region. Contains resources (VMs, K8s clusters, buckets), service accounts, and quotas.
- **Resource** — anything managed by a service (VM, disk, bucket, K8s cluster, etc.)

---

## IAM Default Groups (Tenant-Level)

Nebius uses **groups** to control access. The four default groups from least to most access:

| Group | What it can do |
|-------|---------------|
| `auditors` | View **certain** resource types, **no access to data** inside them |
| `viewers` | View **most** resource types AND **access data** (e.g. download objects) |
| `editors` | View **and manage** most resources, access data |
| `admins` | View and manage **all** resources and data |

**Key point:** Being on the tenant user list does NOT grant access to resources — you must be added to a group.

---

## Service Accounts

- Belong to a **project** (not the tenant)
- Can only work with resources in their own project
- Used for CLI, automation, CI/CD — never use personal user accounts for this
- Two key types for authentication:
  - **Access keys** — for AWS-compatible APIs (e.g. Object Storage S3 API)
  - **Authorized keys** — to obtain IAM tokens for service account authentication

### CLI commands for service accounts
```bash
# Create a service account
nebius iam service-account create --name <name>

# Add to editors group
nebius iam group-membership create \
  --parent-id <editors_group_id> \
  --member-id <service_account_id>

# Delete
nebius iam service-account delete --id <id>
```

### Terraform resource
```hcl
resource "nebius_iam_v1_service_account" "my_sa" {
  name      = "my-sa"
  parent_id = "<project_id>"
}

resource "nebius_iam_v1_group_membership" "editors" {
  parent_id = "<editors_group_id>"
  member_id = nebius_iam_v1_service_account.my_sa.id
}
```

---

## Key Management Service (KMS)

- Stores and manages **cryptographic keys**
- Supports **symmetric** and **asymmetric** keys
- Use cases: encrypt/decrypt data, sign hashes, envelope encryption
- Currently in **preview**, available in all regions
- Separate from MysteryBox — KMS is for keys, not secret values

---

## MysteryBox

- Stores **sensitive data in encrypted form**: API keys, tokens, certificates
- Secrets are **versioned** — each secret has versions; one version is set as **primary**
- Use in scripts and config files to avoid hardcoding sensitive data
- Currently in **preview**, available in all regions

**MysteryBox structure:** Secret → Secret Versions → Payload (the actual value)

**KMS vs MysteryBox:**
- KMS = manages cryptographic keys for encrypting data
- MysteryBox = stores secret values (API keys, tokens, passwords)

---

## Audit Logs

- Records **who did what and when** with your resources
- For security, compliance, and accountability
- Currently in **preview**, available in all regions
- Features:
  - **View** events in the console
  - **Export** events to Object Storage for long-term retention
  - **Filter** events by service, resource, time, action type
- Event structure includes: who, what action, resource, timestamp

**Audit Logs vs Observability Logs:**
- Audit Logs = control plane actions (API calls to Nebius services)
- Observability Logs = data plane (what happens inside your workloads)

---

## Billing

- Billing operates at tenant level
- Budgets and alerts notify when spend crosses a threshold — **resources keep running**
- GPU VMs billed per second of runtime
- Billing changes from Oct 1, 2025: unified billing for GPUs, vCPUs and RAM

---

## Disk Encryption

- **Network SSD**: encryption enabled by **default**, cannot be disabled
- **Network SSD NRD and IO M3**: encryption is **optional**
- Encryption available for both boot and secondary disks
- Encryption managed via KMS

---

## Shared Responsibility Model

| Nebius Responsibility | Customer Responsibility |
|----------------------|------------------------|
| Physical hardware | IAM group assignments |
| Hypervisor | Service account key management |
| Network fabric | Data encryption (where optional) |
| Platform availability | MysteryBox secret rotation |
| Storage hardware | Compliance configuration |

---

## Exam Traps

- There are **4 default groups**: auditors, viewers, editors, admins — not just 3
- `auditors` can view resources but **cannot access data** inside them
- `viewers` CAN access data (download objects etc.)
- Service accounts belong to **project**, not tenant
- Service accounts use **access keys** for S3-compatible APIs and **authorized keys** for IAM tokens
- KMS = keys for encryption; MysteryBox = secret values
- Audit Logs can be **exported to Object Storage**
- Budget alerts do NOT stop VMs automatically
