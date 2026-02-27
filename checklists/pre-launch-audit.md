# Pre-Launch HIPAA Audit Checklist

Run this checklist before any PHI enters your production system. Every item should be verified — not assumed — with evidence documented.

This is the checklist I use with nonprofit clients before go-live. Work through it section by section and sign off each area.

---

## Section 1: Legal and Administrative Readiness

- [ ] **Google BAA is signed and on file**
  - Signed in Cloud Console: IAM & Admin → Settings → HIPAA BAA
  - A copy of the acceptance confirmation is saved to your records

- [ ] **BAAs are signed with every vendor that handles PHI**
  - [ ] Google Cloud (covers Firebase + GCP in-scope services)
  - [ ] Salesforce (if using for case management — verify edition)
  - [ ] Email provider (if PHI will appear in message bodies)
  - [ ] Any other vendor with PHI access

- [ ] **Security Officer is designated** and their responsibilities are documented
- [ ] **Risk Analysis document exists** — identifies where PHI lives, who accesses it, and what threats exist
- [ ] **Incident Response Plan exists** — defines what to do if a breach occurs, who to notify, and within what timeframe
- [ ] **Staff training on PHI handling** is complete for anyone who will access the system
- [ ] **Privacy Policy and Terms of Service** are published and accurately describe data handling practices

---

## Section 2: Architecture Verification

- [ ] **PHI never exists in Layer 1** (WordPress / Hostinger / public web)
  - Test: submit a contact form, verify the data goes nowhere near a database with PHI
  - Test: review all WordPress plugins — none should be connecting to Cloud SQL

- [ ] **Firebase Realtime Database contains zero PHI** — or is disabled entirely
  - Verify in Firebase Console → Realtime Database → Data

- [ ] **Firestore schema review** — open every collection and verify no PHI fields exist
  - Document the schema: what collections, what fields, what's in each

- [ ] **Cloud SQL is the only PHI data store** — no PHI in CSV files, spreadsheets, or email attachments

- [ ] **VPC configuration** — Cloud SQL has no public IP
  - Verify in Cloud Console → SQL → [instance] → Connections → confirm "Public IP" is disabled

- [ ] **Cloud Function → Cloud SQL connection uses VPC connector**
  - Deploy a test function and verify it can reach Cloud SQL only via private IP

---

## Section 3: Access Control Verification

- [ ] **Test each role** — create test accounts for each role and verify access boundaries:

| Test | Expected Result | Actual Result |
|---|---|---|
| `volunteer` account tries to read client PHI | Denied | |
| `program_staff` account tries to read clinical notes | Denied | |
| `clinical_staff` account tries to read unassigned client | Denied | |
| `clinical_staff` account reads assigned client | Allowed | |
| `admin` account reads any client | Allowed | |
| `client` account reads own record | Allowed | |
| `client` account reads another client's record | Denied | |
| Unauthenticated request to any Cloud Function | Denied | |

- [ ] **MFA is enforced** — attempt to access clinical features without MFA enrolled; verify redirect to enrollment
- [ ] **Role assignment is admin-only** — attempt to set custom claims from a non-admin account; verify failure
- [ ] **Firestore Security Rules** are tested in the Rules Playground for all deny cases above

---

## Section 4: Encryption Verification

- [ ] **All traffic uses HTTPS** — verify with browser dev tools that no requests go over HTTP
- [ ] **Cloud SQL requires SSL** — connect to the database with SSL disabled; verify connection is rejected
- [ ] **Cloud Storage bucket is not public** — attempt to access a storage URL without authentication; verify 403
- [ ] **Cloudflare HTTPS is enforced** — "Always Use HTTPS" and HSTS are enabled

---

## Section 5: Audit Logging Verification

- [ ] **Cloud Audit Logs are flowing** — perform a test PHI read and verify an audit log entry appears
  - Cloud Console → Logging → Logs Explorer
  - Filter: `logName="projects/{id}/logs/cloudaudit.googleapis.com%2Fdata_access"`
  - Verify the entry shows: who accessed, what resource, when

- [ ] **Cloud Function PHI access is logged** — verify your audit log entries are being written to Firestore/Cloud Logging with `clientId`, `staffId`, `action`, and `timestamp`

- [ ] **No PHI appears in Cloud Function logs** — search Cloud Logging for sample PHI field names (diagnosis terms, medication names); verify zero results

- [ ] **Log export sink is active** — verify logs are flowing to long-term Cloud Storage bucket

---

## Section 6: Breach Readiness

- [ ] **You know where all PHI lives** — you can answer "where is PHI stored in this system" in under 60 seconds
- [ ] **You can identify who accessed a specific client record and when** — test this with a real audit log query
- [ ] **You can revoke a compromised account** — disable a test user in Firebase Auth and verify immediate loss of access
- [ ] **Incident response contacts are documented** — who gets called if a breach is discovered at 2am
- [ ] **Breach notification template exists** — a draft notification to affected individuals and HHS

---

## Section 7: Operational Security

- [ ] **No PHI in source code** — search codebase for sample client names, test data with real PHI
  ```bash
  # Search for common PHI patterns in code
  grep -r "ssn\|social_security\|diagnosis\|prescription" src/ --include="*.js"
  ```
- [ ] **`.gitignore` excludes** `.env` files, credential files, any local test data with PHI
- [ ] **Production environment variables** are in Secret Manager — not hardcoded, not in `firebase.json`
- [ ] **Test accounts with real PHI** do not exist in production — use synthetic test data only
- [ ] **Backup restoration has been tested** — a backup was actually restored to verify it works

---

## Final Sign-Off

**Go-live is approved when all sections above are complete.**

| Section | Reviewer | Date | Notes |
|---|---|---|---|
| 1. Legal & Administrative | | | |
| 2. Architecture | | | |
| 3. Access Control | | | |
| 4. Encryption | | | |
| 5. Audit Logging | | | |
| 6. Breach Readiness | | | |
| 7. Operational Security | | | |
| **Overall Go-Live Approval** | | | |

---

> **Post-launch:** Schedule a re-review of this checklist every 6 months, and immediately after any significant architectural change.
