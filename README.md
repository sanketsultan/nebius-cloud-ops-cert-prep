# Nebius AI Cloud Ops Certification Prep

Study notes, mock exams, and reading guides for the Nebius AI Cloud Ops certification.

## Folder Structure

```
nebius-cloud-ops-cert-prep/
├── notes/
│   ├── 01-security-iam.md          # Domain 1: Security, Compliance & Billing
│   ├── 02-gpu-clusters.md          # Domain 2: GPU Clusters & Compute
│   ├── 03-workloads.md             # Domain 3: Training & Inference Workloads
│   └── 04-automation-observability.md  # Domain 4: Automation & Maintenance
├── mock-exams/
│   └── mock-exam-01.md             # 40-question full mock exam with answers
└── resources/
    └── reading-guide.md            # Exact docs.nebius.com pages to read, by priority
```

## Exam Overview

| Domain | Weight | Topic |
|--------|--------|-------|
| 1 | ~20% | Security, Compliance & Billing |
| 2 | ~35% | Setting Up & Operating GPU Clusters |
| 3 | ~20% | Running Training & Inference Workloads |
| 4 | ~25% | Platform Automation & Maintenance |

## Quick Tips

- Domain 2 is the heaviest — focus on InfiniBand, Kubernetes GPU setup, and storage types
- Know the difference between MysteryBox (secrets) and KMS (encryption keys)
- Know when to use Slurm vs Kubernetes for a workload
- Know `cordon` vs `drain` vs `delete` for node operations
- Terraform provider for Nebius is `nebius`, not `yandex` or `aws`

## Resources

- Exam guide: `Exam_Guide_Nebius_AI_Cloud_Ops.pdf`
- Official docs: https://docs.nebius.com
