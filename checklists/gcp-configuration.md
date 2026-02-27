# GCP Configuration Checklist

Google Cloud Platform settings beyond Firebase — IAM, networking, audit logging, and project-level controls.

---

## Project & Organization Settings

- [ ] **Google BAA is signed** — confirmed in IAM & Admin → Settings
- [ ] A dedicated GCP project is used for the HIPAA workload — not shared with non-HIPAA applications
- [ ] Project has a meaningful name that identifies the environment (prod, staging)
- [ ] **Billing alerts** are configured at 50%, 90%, and 100% of monthly budget
- [ ] Project **labels** include `environment`, `hipaa-scope: true`, `owner`

---

## Identity and Access Management (IAM)

- [ ] **Principle of least privilege** is applied to all service accounts and users
- [ ] No IAM bindings use `allUsers` or `allAuthenticatedUsers` on PHI resources
- [ ] **Service accounts are purpose-specific** — one service account per Cloud Function group, not a shared account
- [ ] Service accounts do not have `roles/owner` or `roles/editor` — use granular roles
- [ ] **IAM conditions** are used where possible to restrict access by time, IP, or resource
- [ ] Human users with IAM access use **Google Workspace accounts**, not personal Gmail
- [ ] **IAM audit logging** is enabled: IAM Admin Read + Data Read at the org/project level
- [ ] Unused service accounts are disabled or deleted
- [ ] Service account keys are **not stored in code repositories** — use Workload Identity Federation or Secret Manager

---

## VPC and Networking

- [ ] A dedicated **VPC network** is configured for the HIPAA project (not the default VPC)
- [ ] **Private Google Access** is enabled on subnets — allows Cloud Functions and Cloud SQL to communicate without traversing the public internet
- [ ] **VPC connector** is configured for Cloud Functions → Cloud SQL private access
- [ ] Cloud SQL instance has **private IP only** — public IP is disabled
- [ ] **Firewall rules** are reviewed — no rules allow ingress from `0.0.0.0/0` on database ports
- [ ] **VPC Flow Logs** are enabled for audit visibility into network traffic

---

## Cloud Audit Logs

This is non-negotiable. Every PHI-relevant service must have audit logging enabled.

- [ ] Go to IAM & Admin → Audit Logs and verify the following have **Admin Read + Data Read + Data Write** enabled:
  - [ ] Cloud Datastore API (Firestore)
  - [ ] Cloud SQL Admin API
  - [ ] Cloud Storage
  - [ ] Cloud Functions
  - [ ] Identity and Access Management (IAM) API
  - [ ] Cloud Key Management Service (if using CMEK)

- [ ] **Log retention** is configured — default is 30 days for Data Access logs; extend to 1 year+ for HIPAA audit trails
- [ ] Audit logs are **exported to Cloud Storage** for long-term retention (use a log sink)
- [ ] Log export bucket has **Object Lock** or retention policy to prevent deletion

**Setting up a log sink for long-term retention:**
```
Cloud Console → Logging → Log Router → Create Sink
Sink name: hipaa-audit-long-term
Sink destination: Cloud Storage bucket (hipaa-audit-logs-{project-id})
Log filter:
  logName="projects/{project-id}/logs/cloudaudit.googleapis.com%2Fdata_access"
```

---

## Secret Manager

- [ ] **Secret Manager** is used for all credentials — database passwords, API keys, service account keys
- [ ] Cloud Functions access secrets via Secret Manager API — not environment variables in plaintext
- [ ] Secrets have **rotation policies** configured
- [ ] Access to Secret Manager is restricted to necessary service accounts only
- [ ] Secret versions are **disabled (not deleted)** when rotated, to preserve audit trail

---

## Cloud SQL Checklist (GCP Level)

- [ ] Instance is in the **same region** as Cloud Functions to minimize latency and egress cost
- [ ] **Deletion protection** is enabled — prevents accidental instance deletion
- [ ] **Maintenance window** is set to a low-traffic window (weekend nights)
- [ ] **Backup configuration:**
  - Automated daily backups enabled
  - Backup retention: 7 days minimum (30 days recommended)
  - Point-in-time recovery enabled
  - Backup location: same region as instance, or multi-region for DR
- [ ] **Database flags** are set appropriately:
  - `log_connections = on`
  - `log_disconnections = on`
  - `log_duration = on` (for compliance audit purposes)
  - `require_ssl = on`
- [ ] **High Availability** is evaluated — enable if uptime SLA matters (doubles cost)
- [ ] Database schema has a dedicated `phi` schema or database — makes access auditing cleaner

---

## Cloud Storage (GCP Level)

- [ ] PHI buckets are in a **US location** (not EU or Asia unless required by contract)
- [ ] **Retention policies** are locked on PHI buckets — prevents premature deletion
- [ ] **Soft delete** is enabled — 30+ day recovery window for accidentally deleted objects
- [ ] Bucket-level audit logging is verified in the Audit Logs section
- [ ] **CORS configuration** is set to restrict which domains can access storage objects

---

## Security Command Center

- [ ] **Security Command Center** is activated (Standard tier is free)
- [ ] Review findings at project launch and monthly thereafter
- [ ] **Web Security Scanner** is run against any public-facing endpoints
- [ ] Critical and High findings are remediated before go-live

---

**Sign-off:**

| Area | Reviewer | Date | Status |
|---|---|---|---|
| Project & org settings | | | |
| IAM configuration | | | |
| VPC and networking | | | |
| Audit logging | | | |
| Secret Manager | | | |
| Cloud SQL (GCP level) | | | |
| Security Command Center | | | |

---

**Next:** [Pre-Launch Audit →](pre-launch-audit.md)
