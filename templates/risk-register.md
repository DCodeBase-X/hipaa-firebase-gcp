# HIPAA Risk Register

> **Reference**: 45 CFR § 164.308(a)(1) — Risk Analysis and Risk Management
> **See also**: `docs/08-risk-analysis-template.md` for the full risk analysis methodology.

This register is the living record of all identified risks to ePHI. It should be updated:
- After each annual risk analysis
- After any significant system change
- After any security incident
- When new threats or vulnerabilities are identified

Retain all versions of this document for **6 years**.

---

## How to Use This Register

1. Assign each risk a unique ID (R-YYYY-###)
2. Score likelihood (1–3) and impact (1–3); Risk Score = L × I
3. Assign a risk owner responsible for implementing the response
4. Track the status of each risk response
5. Document residual risk after controls are implemented
6. Mark risks as Closed only when the risk is fully mitigated or formally accepted

**Risk Levels**:
- Score 1–2: Low
- Score 3–4: Medium
- Score 6–9: High

---

## Current Risk Register

### R-2024-001

| Field | Value |
|-------|-------|
| **Risk ID** | R-2024-001 |
| **Date Identified** | |
| **Risk Title** | Compromised staff credentials — no MFA |
| **Threat Event** | Unauthorized access via stolen or phished password |
| **Vulnerability** | MFA not enforced for all Firebase Auth accounts |
| **ePHI Systems at Risk** | Firebase Auth, Cloud SQL (via Cloud Functions) |
| **Likelihood (1–3)** | 2 |
| **Impact (1–3)** | 3 |
| **Risk Score** | 6 |
| **Risk Level** | High |
| **Risk Owner** | Security Officer |
| **Response Type** | Mitigate |
| **Planned Control** | Enforce MFA for all staff accounts in Firebase Auth; block sign-in without MFA |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed |
| **Residual Likelihood** | 1 |
| **Residual Impact** | 3 |
| **Residual Score** | 3 |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A |
| **Notes** | |

---

### R-2024-002

| Field | Value |
|-------|-------|
| **Risk ID** | R-2024-002 |
| **Date Identified** | |
| **Risk Title** | PHI sent via unencrypted email |
| **Threat Event** | Staff member emails ePHI to client, colleague, or incorrect recipient via unsecured email |
| **Vulnerability** | Staff not trained on secure communication; no DLP controls on email |
| **ePHI Systems at Risk** | Email (external channel) |
| **Likelihood (1–3)** | 3 |
| **Impact (1–3)** | 2 |
| **Risk Score** | 6 |
| **Risk Level** | High |
| **Risk Owner** | Security Officer |
| **Response Type** | Mitigate |
| **Planned Control** | Staff training (Module 2); DLP rule on mail system to flag PHI patterns; approved secure messaging guidance |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed |
| **Residual Likelihood** | 1 |
| **Residual Impact** | 2 |
| **Residual Score** | 2 |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A |
| **Notes** | |

---

### R-2024-003

| Field | Value |
|-------|-------|
| **Risk ID** | R-2024-003 |
| **Date Identified** | |
| **Risk Title** | Firestore Security Rules misconfiguration |
| **Threat Event** | Accidental rule change exposes records to unauthorized users |
| **Vulnerability** | Security rules not tested in CI/CD; manual deployments possible |
| **ePHI Systems at Risk** | Firestore (operational data; client IDs linked to PHI) |
| **Likelihood (1–3)** | 2 |
| **Impact (1–3)** | 2 |
| **Risk Score** | 4 |
| **Risk Level** | Medium |
| **Risk Owner** | Lead Developer |
| **Response Type** | Mitigate |
| **Planned Control** | Add automated Firestore rule testing to CI/CD pipeline; require PR review for rule changes; enable alerting for rule changes |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed |
| **Residual Likelihood** | 1 |
| **Residual Impact** | 2 |
| **Residual Score** | 2 |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A |
| **Notes** | |

---

### R-2024-004

| Field | Value |
|-------|-------|
| **Risk ID** | R-2024-004 |
| **Date Identified** | |
| **Risk Title** | Lost or stolen unencrypted staff device |
| **Threat Event** | Device containing cached ePHI access or active sessions is lost or stolen |
| **Vulnerability** | Devices not enrolled in MDM; full-disk encryption not verified; screen lock not enforced |
| **ePHI Systems at Risk** | All ePHI systems (via active browser sessions or cached data) |
| **Likelihood (1–3)** | 2 |
| **Impact (1–3)** | 2 |
| **Risk Score** | 4 |
| **Risk Level** | Medium |
| **Risk Owner** | IT Manager / Security Officer |
| **Response Type** | Mitigate |
| **Planned Control** | MDM enrollment for all ePHI-capable devices; verify encryption; enforce screen lock; implement remote wipe capability |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed |
| **Residual Likelihood** | 1 |
| **Residual Impact** | 2 |
| **Residual Score** | 2 |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A |
| **Notes** | See `docs/10-physical-safeguards.md` § 3.2 |

---

### R-2024-005

| Field | Value |
|-------|-------|
| **Risk ID** | R-2024-005 |
| **Date Identified** | |
| **Risk Title** | Vendor without BAA receives ePHI |
| **Threat Event** | ePHI transmitted to or stored by a vendor that has not signed a BAA |
| **Vulnerability** | Vendor BAA inventory not maintained; new integrations added without BAA review |
| **ePHI Systems at Risk** | Any system with external integrations |
| **Likelihood (1–3)** | 1 |
| **Impact (1–3)** | 3 |
| **Risk Score** | 3 |
| **Risk Level** | Medium |
| **Risk Owner** | Privacy Officer |
| **Response Type** | Mitigate |
| **Planned Control** | Maintain BAA inventory (see `docs/11-administrative-policies.md`); require Privacy Officer approval before any new vendor integration; annual BAA review |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed |
| **Residual Likelihood** | 1 |
| **Residual Impact** | 3 |
| **Residual Score** | 3 |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A |
| **Notes** | Firebase Realtime Database is explicitly excluded — never store PHI there |

---

### R-2024-006

| Field | Value |
|-------|-------|
| **Risk ID** | R-2024-006 |
| **Date Identified** | |
| **Risk Title** | Cloud Storage bucket misconfigured as public |
| **Threat Event** | PHI documents publicly accessible due to misconfigured bucket permissions |
| **Vulnerability** | IAM misconfiguration or incorrect bucket-level access settings |
| **ePHI Systems at Risk** | Cloud Storage |
| **Likelihood (1–3)** | 1 |
| **Impact (1–3)** | 3 |
| **Risk Score** | 3 |
| **Risk Level** | Medium |
| **Risk Owner** | Lead Developer |
| **Response Type** | Mitigate |
| **Planned Control** | Uniform bucket-level access enforced; Security Command Center misconfiguration alerts enabled; quarterly IAM audit |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed |
| **Residual Likelihood** | 1 |
| **Residual Impact** | 3 |
| **Residual Score** | 3 |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A |
| **Notes** | |

---

### R-2024-007

| Field | Value |
|-------|-------|
| **Risk ID** | R-2024-007 |
| **Date Identified** | |
| **Risk Title** | Insufficient audit logging — PHI access not traceable |
| **Threat Event** | Security incident occurs but cannot be fully investigated due to missing logs |
| **Vulnerability** | Cloud Audit Logs Data Access logs not enabled; log retention too short |
| **ePHI Systems at Risk** | Cloud SQL, Firestore, Cloud Storage, Cloud Functions |
| **Likelihood (1–3)** | 1 |
| **Impact (1–3)** | 3 |
| **Risk Score** | 3 |
| **Risk Level** | Medium |
| **Risk Owner** | Security Officer |
| **Response Type** | Mitigate |
| **Planned Control** | Enable Data Read + Data Write audit logs for all ePHI services; configure log export to Cloud Storage for long-term retention; verify monthly |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed |
| **Residual Likelihood** | 1 |
| **Residual Impact** | 2 |
| **Residual Score** | 2 |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A |
| **Notes** | See `checklists/gcp-configuration.md` — Audit Logs section |

---

## Risk Register Summary Dashboard

| Risk ID | Title | Score | Level | Status | Owner |
|---------|-------|------:|-------|--------|-------|
| R-2024-001 | Compromised credentials — no MFA | 6 | High | | |
| R-2024-002 | PHI sent via unencrypted email | 6 | High | | |
| R-2024-003 | Firestore Security Rules misconfiguration | 4 | Medium | | |
| R-2024-004 | Lost/stolen unencrypted device | 4 | Medium | | |
| R-2024-005 | Vendor without BAA receives ePHI | 3 | Medium | | |
| R-2024-006 | Cloud Storage bucket misconfigured as public | 3 | Medium | | |
| R-2024-007 | Insufficient audit logging | 3 | Medium | | |
| _(add new risks below)_ | | | | | |

---

## Adding New Risk Entries

Copy the template below for each new risk:

```
### R-YYYY-###

| Field | Value |
|-------|-------|
| **Risk ID** | R-YYYY-### |
| **Date Identified** | |
| **Risk Title** | |
| **Threat Event** | |
| **Vulnerability** | |
| **ePHI Systems at Risk** | |
| **Likelihood (1–3)** | |
| **Impact (1–3)** | |
| **Risk Score** | |
| **Risk Level** | |
| **Risk Owner** | |
| **Response Type** | Mitigate / Accept / Transfer / Avoid |
| **Planned Control** | |
| **Target Completion** | |
| **Status** | Open / In Progress / Completed / Closed |
| **Residual Likelihood** | |
| **Residual Impact** | |
| **Residual Score** | |
| **Residual Risk Accepted By** | |
| **Date Accepted** | |
| **Related Incident** | N/A or INC-YYYY-### |
| **Notes** | |
```

---

## Review History

| Review Date | Trigger | Risks Added | Risks Closed | Reviewed By |
|-------------|---------|-------------|-------------|-------------|
| | Initial creation | 7 | 0 | |
| | Annual review | | | |
| | System change: | | | |
| | Post-incident: | | | |

---

## Related Documents

- `docs/08-risk-analysis-template.md` — Methodology for identifying and scoring risks
- `docs/11-administrative-policies.md` — Risk management policy
- `checklists/incident-response-runbook.md` — Update register after incidents
- `checklists/pre-launch-audit.md` — Verify controls before go-live
