# Incident Response Runbook

> **For breach notification requirements and legal obligations**: See `docs/09-breach-notification-procedure.md`
> **For risk documentation**: See `templates/risk-register.md`

This runbook provides step-by-step operational procedures for responding to security incidents affecting ePHI. Follow these steps in order. Roles referenced are defined in `docs/11-administrative-policies.md`.

---

## Emergency Contacts

| Role | Name | Phone | Email |
|------|------|-------|-------|
| Security Officer | | | |
| Privacy Officer | | | |
| Executive Director | | | |
| Legal Counsel | | | |
| Lead Developer / IT | | | |
| GCP Support | N/A | [GCP Support Console](https://console.cloud.google.com/support) | |
| HHS OCR (breaches) | N/A | 1-800-368-1019 | OCRMail@hhs.gov |

---

## Incident Severity Classification

Classify the incident before beginning response. Severity determines response timeline and escalation path.

| Severity | Definition | Examples | Response SLA |
|----------|------------|----------|-------------|
| **P1 – Critical** | Active, ongoing breach of ePHI or complete system compromise | Ransomware, active unauthorized access with PHI exfiltration, database exposed publicly | Immediate — all hands |
| **P2 – High** | Confirmed breach of ePHI, contained but serious | Lost device with PHI, PHI emailed to wrong recipient, misconfigured security rule | < 2 hours |
| **P3 – Medium** | Suspected breach under investigation, or significant policy violation | Suspicious login, unexpected system change, report of unauthorized access | < 24 hours |
| **P4 – Low** | Security policy violation without apparent PHI exposure | Password shared, screen not locked, minor misconfiguration with no exposure | < 72 hours |

---

## Phase 1: Detection and Initial Triage

**Trigger**: Any staff member reports a suspected incident, or an automated alert fires.

### Step 1.1 — Receive and Log the Report

- [ ] Record the date and time the incident was reported
- [ ] Record the name and role of the person reporting
- [ ] Capture their full description of what they observed

**Initial incident log entry**:

| Field | Value |
|-------|-------|
| Incident ID | INC-YYYY-### |
| Date/Time Reported | |
| Reported By | |
| Description | |
| Systems Mentioned | |
| Assigned To (Security Officer) | |

### Step 1.2 — Classify Severity

Based on the report, assign initial severity (P1–P4). This may change as investigation proceeds.

Initial Severity: _______ Assigned by: _______________________________

### Step 1.3 — Notify Based on Severity

| Severity | Notify Immediately |
|----------|--------------------|
| P1 | Security Officer, Executive Director, Legal Counsel, Lead Developer |
| P2 | Security Officer, Legal Counsel |
| P3 | Security Officer |
| P4 | Security Officer |

---

## Phase 2: Containment

**Goal**: Stop ongoing harm. Do not allow further PHI exposure.

### Step 2.1 — Identify the Affected System(s)

- [ ] Which system is involved? (Firebase Auth / Cloud Functions / Firestore / Cloud SQL / Cloud Storage / Workstation / Other)
- [ ] Is the threat active or historical?
- [ ] Is ePHI actively being accessed or exfiltrated right now?

### Step 2.2 — Containment Actions by Incident Type

#### Compromised User Account

- [ ] Immediately disable the Firebase Auth account: Firebase Console → Authentication → Disable user
- [ ] Revoke all active sessions: Firebase Admin SDK `revokeRefreshTokens(uid)`
- [ ] Remove GCP IAM roles from the compromised account
- [ ] Change any shared credentials the account may have known
- [ ] Review Cloud Audit Logs for actions taken by the account in the last 30 days

```bash
# Revoke Firebase sessions via Admin SDK
firebase auth:revoke-tokens <uid>

# Or via GCP CLI
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="user:compromised@email.com" \
  --role="ROLE"
```

#### Misconfigured Firestore Security Rules

- [ ] Immediately deploy corrected security rules:
  ```bash
  firebase deploy --only firestore:rules
  ```
- [ ] Review Firestore audit logs to determine what was accessed during the window of misconfiguration
- [ ] Document the start and end time of the misconfiguration window
- [ ] Identify all documents that were in scope of the misconfigured rule

#### Exposed Cloud Storage Bucket

- [ ] Immediately remove public access:
  - GCP Console → Cloud Storage → Bucket → Permissions → Remove `allUsers` / `allAuthenticatedUsers`
  ```bash
  gsutil iam ch -d allUsers gs://BUCKET_NAME
  ```
- [ ] Enable uniform bucket-level access if not already enabled
- [ ] Review access logs to determine if any objects were accessed during the exposure window
- [ ] Identify all objects in the exposed bucket

#### Lost or Stolen Device

- [ ] Initiate remote wipe via MDM immediately
- [ ] Document wipe initiation time and confirmation
- [ ] Revoke all sessions for the device user (Firebase Auth, GCP)
- [ ] Assess: did the device have local copies of ePHI? Was the screen locked?
- [ ] Report to law enforcement if theft is suspected (obtain case number)

#### Ransomware / Malware

- [ ] Isolate affected workstations immediately (disconnect from network)
- [ ] Do NOT pay ransom without legal and executive approval
- [ ] Contact GCP Support if cloud systems are affected
- [ ] Initiate disaster recovery procedures (`docs/11-administrative-policies.md` § 7)
- [ ] Preserve all forensic evidence before any cleanup

#### PHI Sent to Wrong Recipient (Email/Fax)

- [ ] Immediately attempt to recall the email if the mail system supports it
- [ ] Contact the recipient directly and request they delete the message and confirm deletion
- [ ] Document recipient's response
- [ ] If recipient is another HIPAA-covered entity: they are likely obligated to protect it
- [ ] If recipient is an unauthorized party: high probability this is a reportable breach

#### Unauthorized Internal Access

- [ ] Preserve all logs before taking any action
- [ ] Suspend the workforce member's access pending investigation (do not terminate access until investigation is documented)
- [ ] Engage HR and Legal Counsel before any workforce action
- [ ] Review all actions taken by the account using Cloud Audit Logs

---

## Phase 3: Investigation and Evidence Collection

**Goal**: Determine the full scope of what happened.

### Step 3.1 — Preserve Evidence

- [ ] Export relevant Cloud Audit Logs before they age out (default retention: 400 days for Admin Activity, 30 days for Data Access)
  ```bash
  gcloud logging read "resource.type=gcs_bucket AND timestamp>=\"2024-01-01T00:00:00Z\"" \
    --format=json > incident-logs-$(date +%Y%m%d).json
  ```
- [ ] Screenshot any relevant console states
- [ ] Preserve any local device forensics before wiping
- [ ] Do not modify or delete logs

### Step 3.2 — Build an Incident Timeline

Document every relevant event with timestamps:

| Date/Time (UTC) | Event | Source (log/report/system) |
|-----------------|-------|---------------------------|
| | Incident originated (estimated) | |
| | Incident discovered | |
| | Incident reported | |
| | Containment initiated | |
| | Containment completed | |
| | Investigation completed | |

### Step 3.3 — Determine Scope of PHI Exposure

Answer the following:

- How many individuals' PHI was potentially exposed? _______________
- What types of PHI were involved? (health, MH, SUD, criminal justice, other) _______________
- What was the duration of exposure? _______________
- Who had unauthorized access? _______________
- Is there evidence the PHI was actually viewed, downloaded, or transmitted? _______________
- Has all unauthorized access been terminated? _______________

---

## Phase 4: Breach Determination

Apply the four-factor risk assessment per `docs/09-breach-notification-procedure.md § 1.2`.

| Factor | Assessment | Rationale |
|--------|------------|-----------|
| Nature and extent of PHI | Low / Medium / High | |
| Who accessed/received PHI | Low / Medium / High | |
| Whether PHI was actually acquired or viewed | Low / Medium / High | |
| Extent to which risk has been mitigated | Low / Medium / High | |

**Breach Determination**: Reportable Breach / Not a Reportable Breach

**Determined by**: _______________________________________________ Date: _______________________________________________

If reportable breach: proceed to `docs/09-breach-notification-procedure.md` for notification procedures.

---

## Phase 5: Notification (If Reportable Breach)

Follow `docs/09-breach-notification-procedure.md` in full.

**Notification deadlines from discovery date**:
- Affected individuals: 60 calendar days
- HHS (≥ 500 individuals): 60 calendar days
- HHS (< 500 individuals): Annual log, submitted by March 1 following year
- Media (≥ 500 in a state): 60 calendar days

Notification status:

| Recipient | Required | Date Sent | Method | Sent By |
|-----------|----------|-----------|--------|---------|
| Affected individuals | Y / N | | | |
| HHS | Y / N | | | |
| Media | Y / N | | | |

---

## Phase 6: Recovery and Remediation

### Step 6.1 — Restore Normal Operations

- [ ] Verify containment is complete before restoring access
- [ ] Test affected systems before returning to production use
- [ ] Restore from backup if data was corrupted or lost (document restore point used)
- [ ] Verify audit logs are flowing correctly post-recovery

### Step 6.2 — Corrective Action Plan

Develop and assign corrective actions to prevent recurrence:

| Finding | Corrective Action | Owner | Target Date | Completed |
|---------|------------------|-------|-------------|-----------|
| | | | | |

### Step 6.3 — Update Risk Analysis

- [ ] Add this incident as a threat event to the risk register (`templates/risk-register.md`)
- [ ] Update likelihood scores for related risks
- [ ] Update risk analysis document (`docs/08-risk-analysis-template.md`)

---

## Phase 7: Post-Incident Review

Schedule within **30 days** of incident closure.

### Review Agenda

1. Incident timeline walkthrough
2. Root cause identification
3. Were existing controls effective? What failed?
4. What did we do well in our response?
5. What should we do differently?
6. Are corrective actions on track?
7. Do policies or training need to be updated?

**Review date**: _______________________________________________ **Facilitator**: _______________________________________________

**Attendees**: _______________________________________________

**Key findings**: _______________________________________________

**Action items from review**:

| Action | Owner | Due Date |
|--------|-------|----------|
| | | |

---

## Incident Closure

**Incident ID**: _______________________________________________ **Final Severity**: _______________________________________________

**Incident closed by**: _______________________________________________ **Date**: _______________________________________________

**Breach determination**: Reportable / Not Reportable **Notifications completed**: Y / N / N/A

**Documentation package complete**:
- [ ] Initial incident report
- [ ] Evidence log
- [ ] Incident timeline
- [ ] Four-factor breach assessment
- [ ] Breach notification records (if applicable)
- [ ] Corrective action plan
- [ ] Post-incident review notes
- [ ] Updated risk register

All documentation retained for 6 years from date of closure.

---

## Related Documents

- `docs/09-breach-notification-procedure.md` — Notification requirements and templates
- `docs/11-administrative-policies.md` — Sanction policy, roles, contingency plan
- `docs/08-risk-analysis-template.md` — Update after incidents
- `templates/risk-register.md` — Add new risks identified
- `templates/breach-notification-letter.md` — Individual notification letter
