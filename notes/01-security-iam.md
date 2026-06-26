# Domain 1: Security, Compliance & Billing (~20%)

## IAM Overview

Nebius IAM controls who can do what on which resources.

**Two identity types:**
- **Users** — humans logging in via the console
- **Service accounts** — non-human identities for automation, CI/CD, and workloads

**Two scope levels:**
- **Tenant-level** — applies across all projects in the organization
- **Project-level** — applies only within a specific project

Always assign the narrowest scope and least-privileged role needed.

---

## Key IAM Roles to Know

| Role | What it allows |
|------|---------------|
| `viewer` | Read-only access to a resource |
| `editor` | Read + write, cannot manage IAM |
| `admin` | Full control including IAM management |
| `storage.viewer` | Read objects from Object Storage |
| `storage.editor` | Read + write objects |
| `container.editor` | Push/pull images to Container Registry |
| `iam.serviceAccountUser` | Use a service account (impersonate it) |

**Rule:** CI/CD pipelines and training jobs must use service accounts, not user credentials.

---

## Key Management Service (KMS)

- Manages **cryptographic keys** for encrypting data at rest
- Customer-managed keys: you control rotation and deletion
- Platform-managed keys: Nebius handles key lifecycle
- Used with Object Storage, disks, and other services

**Not for storing secrets** — that's MysteryBox.

---

## MysteryBox

- Stores **secrets**: API keys, tokens, passwords, connection strings
- Applications fetch secrets at runtime via API
- Secrets are versioned and access-controlled via IAM
- Think: Nebius equivalent of AWS Secrets Manager / HashiCorp Vault

**KMS vs MysteryBox:**
- KMS = encrypts data (keys)
- MysteryBox = stores secrets (values)

---

## Audit Logs

- Records all **API calls** made to Nebius resources
- Captures: who, what action, on which resource, when, from where
- Retained for compliance and security investigation
- Use case: "Who deleted the GPU cluster?" → check Audit Logs

**Not the same as application logs** — those live in Observability > Logs.

---

## Billing

- Budget alerts notify when spend crosses a threshold — resources keep running
- Resources are NOT auto-stopped by budget alerts (unless configured separately)
- Billing is per project; costs are tracked at tenant level
- GPU VMs are billed per second of runtime

---

## Shared Responsibility Model

| Nebius Responsibility | Customer Responsibility |
|----------------------|------------------------|
| Physical hardware | IAM role assignments |
| Hypervisor | Data encryption |
| Network fabric | Workload security |
| Platform availability | Secret rotation |
| Storage hardware | Compliance configuration |

---

## Exam Traps

- MysteryBox stores secrets, KMS stores keys — don't confuse them
- Audit Logs != application logs
- Budget alerts don't stop VMs
- Service accounts for automation, not personal user accounts
- Tenant-level roles apply across ALL projects — be careful assigning them
