# Firebase Configuration Checklist

Every Firebase setting that must be verified before handling PHI. Work through this top to bottom — each item maps to a specific HIPAA technical safeguard requirement.

---

## Prerequisites

- [ ] Firebase project is on the **Blaze (pay-as-you-go)** plan — Spark plan cannot be used for HIPAA
- [ ] **Google BAA has been signed** in Cloud Console (IAM & Admin → Settings → HIPAA BAA)
- [ ] A named Security Officer is designated for your organization
- [ ] A written Risk Analysis document exists (even a simple one)

---

## Firebase Authentication

- [ ] **Email/password auth is enabled** with strong password policy enforced
- [ ] **MFA is enabled** for the project (Authentication → Sign-in method → Multi-factor)
- [ ] MFA is **enforced** (not just available) for `admin` and `clinical_staff` role users
- [ ] **Session expiration** is set appropriately — tokens should not be indefinite for PHI access
- [ ] **Authorized domains** list is reviewed — remove any domains that shouldn't be authenticating
- [ ] Custom claims are used for roles — roles are NOT stored as user-editable profile fields
- [ ] Role assignment functions are server-side only (Cloud Functions + Admin SDK)
- [ ] No PHI fields exist in Firebase user profiles (`displayName`, `photoURL`, custom claims)
- [ ] **Anonymous auth** is disabled unless there is a specific, documented use case

---

## Cloud Firestore

- [ ] **Security rules default to deny-all** — `allow read, write: if false` as the base rule
- [ ] All rules are **role-validated** — check `request.auth.token.role` explicitly
- [ ] **Clinical staff rules** enforce client assignment — staff can only access assigned client records
- [ ] **No PHI fields** exist in any Firestore document — confirmed by schema review
- [ ] Firestore **audit logging is enabled**: Cloud Console → IAM & Admin → Audit Logs → Cloud Datastore API → Data Read + Data Write + Admin Read
- [ ] Firestore data location is set to a **US region** and cannot be changed (verify at project creation)
- [ ] **Security rules are tested** — use the Firebase Rules Playground to verify deny/allow cases
- [ ] Production rules do **not** have any `allow read, write: if true` statements

---

## Cloud Functions

- [ ] All PHI-accessing functions **validate authentication** before any logic executes
- [ ] All PHI-accessing functions **validate role** (not just authentication)
- [ ] Clinical staff functions **validate client assignment** before returning PHI
- [ ] **PHI is not logged** — `console.log()` calls do not include PHI field values
- [ ] Cloud Functions are deployed in the **same region** as Cloud SQL
- [ ] Functions use a **VPC connector** to access Cloud SQL — not the public IP
- [ ] Service account used by Cloud Functions has **minimum necessary IAM permissions**
- [ ] **CORS configuration** restricts which origins can call HTTPS callable functions
- [ ] Functions have **appropriate timeout limits** set (don't leave long-running processes with PHI in memory)

---

## Cloud SQL

- [ ] **Public IP is disabled** — Cloud SQL instance has private IP only
- [ ] **Authorized networks list is empty** — no IP ranges should have direct access
- [ ] Database connections use **Cloud SQL Auth Proxy** or **VPC Private IP** — not public connection strings
- [ ] **Encryption at rest** is enabled (Google-managed keys, minimum — CMEK preferred)
- [ ] **Encryption in transit** — SSL/TLS is required for all database connections (`require_ssl = on`)
- [ ] **Audit logging** is enabled for the database (log all queries that touch PHI tables — use `pgaudit` for PostgreSQL)
- [ ] **Automated backups** are enabled with a retention period that meets your policy
- [ ] **Point-in-time recovery** is enabled
- [ ] Database users follow **least privilege** — application service accounts cannot drop tables or modify schema
- [ ] A separate read-only database user exists for reporting/analytics queries

---

## Cloud Storage

- [ ] **Uniform bucket-level access** is enabled on all PHI buckets — no per-object ACLs
- [ ] **No public buckets** — all PHI buckets are private
- [ ] **Object versioning** is enabled — allows recovery if PHI is accidentally deleted or overwritten
- [ ] **Bucket lifecycle policies** are configured for retention compliance
- [ ] Firebase Storage security rules mirror Firestore rules — deny by default
- [ ] PHI documents are **not served via Firebase Hosting** (Hosting is not BAA-covered)
- [ ] Signed URLs for PHI document access have **short expiration times** (minutes, not hours/days)

---

## Services to Disable or Verify Are Unused

- [ ] **Firebase Realtime Database** — disabled or confirmed to contain zero PHI
- [ ] **Firebase Analytics** — confirmed PHI is never passed in event parameters or user properties
- [ ] **Crashlytics** — confirmed crash reports are sanitized before sending (no PHI in device state)
- [ ] **Firebase Remote Config** — reviewed to confirm no PHI values in config parameters
- [ ] **Firebase Hosting** — confirmed it serves only public content with no PHI

---

## Monitoring and Alerting

- [ ] **Cloud Monitoring alerts** are configured for unusual authentication activity (failed logins, off-hours access)
- [ ] **Budget alerts** are configured in Cloud Console — unexpected spend can indicate unauthorized usage
- [ ] **Error Reporting** is reviewed periodically — errors may surface misconfigured security rules
- [ ] A process exists to review Cloud Audit Logs for PHI access anomalies at least monthly

---

**Sign-off:** Before go-live, this checklist should be reviewed and signed by the designated Security Officer.

| Item | Reviewer | Date | Notes |
|---|---|---|---|
| Firebase Auth configuration | | | |
| Firestore rules and audit logging | | | |
| Cloud Functions PHI access controls | | | |
| Cloud SQL private access and encryption | | | |
| Storage bucket configuration | | | |
| Disabled/verified uncovered services | | | |

---

**Next:** [GCP Configuration →](gcp-configuration.md) | [Pre-Launch Audit →](pre-launch-audit.md)
