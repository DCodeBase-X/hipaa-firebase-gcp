# HIPAA Risk Analysis Template

> **Reference**: 45 CFR § 164.308(a)(1) — Required implementation specification under the Administrative Safeguards of the Security Rule.

HIPAA does not prescribe a specific risk analysis methodology, but requires that it be accurate, thorough, and documented. This template follows NIST SP 800-30 guidance, which HHS recognizes as an acceptable framework.

---

## How to Use This Document

1. Complete Section 1 (Scope) before beginning any analysis.
2. Work through Sections 2–5 with your Security Officer and relevant technical staff.
3. Transfer findings to the Risk Register (`templates/risk-register.md`).
4. Review and update this analysis at least **annually** and after any significant system change.
5. Retain completed copies for **6 years** per HIPAA documentation requirements.

---

## Section 1: Scope and Context

**Organization Name**: _______________________________________________

**Date of Analysis**: _______________________________________________

**Conducted By** (name and title): _______________________________________________

**Reviewed By** (Security Officer): _______________________________________________

**Next Scheduled Review**: _______________________________________________

### 1.1 Systems in Scope

List all systems that create, receive, maintain, or transmit ePHI:

| System | Description | ePHI Data Types | Location |
|--------|-------------|-----------------|----------|
| Cloud SQL (GCP) | Primary PHI datastore | Health, MH, SUD, criminal justice records | GCP us-central1 |
| Cloud Storage | Document storage | Intake forms, scanned records | GCP us-central1 |
| Firestore | Operational metadata | Client IDs, assignment data | GCP us-central1 |
| Cloud Functions | Data processing layer | Transient ePHI in transit | GCP us-central1 |
| Firebase Auth | Identity management | No PHI — auth tokens only | GCP us-central1 |
| Staff workstations | Data access terminals | Screen display of ePHI | On-premises |
| Staff mobile devices | Field access | Limited ePHI via app | Off-premises |
| _(add rows as needed)_ | | | |

### 1.2 ePHI Inventory

Enumerate all categories of ePHI the organization handles:

- [ ] Health/medical records
- [ ] Mental health records
- [ ] Substance use disorder (SUD) records
- [ ] Criminal justice records
- [ ] Children's services / juvenile records
- [ ] Social service assessments
- [ ] Case notes
- [ ] Demographic identifiers (name, DOB, address, SSN, phone)
- [ ] Financial information linked to PHI
- [ ] Other: _______________________________________________

---

## Section 2: Threat Identification

For each threat source, assess whether it is applicable to your environment.

### 2.1 Threat Source Categories

| Threat Source | Examples | Applicable? |
|---------------|----------|-------------|
| External malicious actors | Hackers, ransomware operators, phishing campaigns | Y / N |
| Internal malicious actors | Disgruntled employees, data theft | Y / N |
| Accidental insider actions | Misconfiguration, wrong recipient, lost device | Y / N |
| Natural disasters | Fire, flood, earthquake affecting infrastructure | Y / N |
| Third-party / vendor failures | Cloud provider outage, BAA violation by vendor | Y / N |
| Physical theft | Stolen laptop, stolen storage media | Y / N |
| Software vulnerabilities | Unpatched systems, insecure code | Y / N |
| Social engineering | Phishing, pretexting, vishing | Y / N |

### 2.2 Threat Event Catalog

Document specific threat events relevant to your systems:

| ID | Threat Event | Source | Target System |
|----|-------------|--------|---------------|
| T-01 | Unauthorized access via compromised credentials | External / Internal | Firebase Auth, Cloud SQL |
| T-02 | Ransomware encrypting Cloud SQL backups | External | Cloud SQL |
| T-03 | ePHI sent via unencrypted email | Accidental insider | Email |
| T-04 | Misconfigured Firestore Security Rules expose records | Accidental insider | Firestore |
| T-05 | Staff member accesses records outside their client assignment | Internal | Cloud SQL via Cloud Functions |
| T-06 | Lost/stolen staff device with cached session | Physical | Workstation / mobile |
| T-07 | Firebase Realtime Database accidentally re-enabled with PHI | Accidental insider | Firebase |
| T-08 | Third-party vendor without BAA receives PHI | Vendor failure | External service |
| T-09 | Cloud Storage bucket misconfigured as public | Accidental insider | Cloud Storage |
| T-10 | MFA bypass via SIM-swapping attack | External | Firebase Auth |
| _(add rows)_ | | | |

---

## Section 3: Vulnerability Identification

For each system in scope, identify relevant vulnerabilities:

| ID | Vulnerability | System Affected | Current Control |
|----|--------------|-----------------|-----------------|
| V-01 | No MFA enforced for all staff accounts | Firebase Auth | MFA enabled — verify enforcement |
| V-02 | Overly permissive IAM roles | GCP IAM | Least-privilege review needed |
| V-03 | Audit logs not reviewed regularly | Cloud Audit Logs | Automated alerts configured |
| V-04 | No endpoint encryption on staff laptops | Workstations | Policy required — see doc 10 |
| V-05 | Secrets hard-coded in old code branches | Source code | Secret Manager in use — verify |
| V-06 | No data loss prevention (DLP) scanning | All services | GCP DLP not yet configured |
| V-07 | Staff not trained on phishing recognition | Human | Training program required |
| V-08 | No formal media sanitization process | Physical media | Procedure required |
| V-09 | Firestore rules not tested on role change | Firestore | Manual testing only |
| V-10 | Vendor BAA status not reviewed annually | All vendors | Annual review not scheduled |
| _(add rows)_ | | | |

---

## Section 4: Risk Assessment

### 4.1 Likelihood and Impact Ratings

**Likelihood Scale**:
- **1 – Low**: Unlikely given current controls and environment
- **2 – Medium**: Possible, has happened in similar organizations
- **3 – High**: Likely without additional controls

**Impact Scale**:
- **1 – Low**: Minimal harm to individuals or organization, easily remediated
- **2 – Medium**: Moderate harm; limited PHI exposed; reportable breach possible
- **3 – High**: Significant harm to individuals; large-scale breach; regulatory action likely

**Risk Score** = Likelihood × Impact (1–9)

| Risk Level | Score Range |
|------------|-------------|
| Low | 1–2 |
| Medium | 3–4 |
| High | 6–9 |

### 4.2 Risk Register Summary

Transfer detailed entries to `templates/risk-register.md`. Summary here:

| Risk ID | Threat + Vulnerability | Likelihood | Impact | Score | Level | Owner |
|---------|----------------------|------------|--------|-------|-------|-------|
| R-01 | T-01 + V-01: Compromised credentials, no MFA | 2 | 3 | 6 | High | Security Officer |
| R-02 | T-04 + V-09: Firestore misconfiguration | 2 | 2 | 4 | Medium | Lead Developer |
| R-03 | T-06 + V-04: Lost device with session | 2 | 2 | 4 | Medium | IT Manager |
| R-04 | T-03 + V-07: ePHI sent via unencrypted email | 3 | 2 | 6 | High | Security Officer |
| R-05 | T-08 + V-10: Vendor without BAA receives PHI | 1 | 3 | 3 | Medium | Compliance Officer |
| _(add rows)_ | | | | | | |

---

## Section 5: Risk Response

For each identified risk, document the chosen response:

**Response Options**:
- **Mitigate**: Implement controls to reduce likelihood or impact
- **Accept**: Risk is within acceptable tolerance; document rationale
- **Transfer**: Shift risk via insurance, BAA, or contractual agreement
- **Avoid**: Eliminate the activity or system that creates the risk

| Risk ID | Response | Control / Action | Target Date | Status |
|---------|----------|-----------------|-------------|--------|
| R-01 | Mitigate | Enforce MFA for all staff in Firebase Auth | 30 days | Open |
| R-02 | Mitigate | Add automated Firestore rule testing to CI/CD | 60 days | Open |
| R-03 | Mitigate | Implement MDM with remote wipe; enforce screen lock | 45 days | Open |
| R-04 | Mitigate | Staff training + DLP rule to block PHI in email | 30 days | Open |
| R-05 | Mitigate | Annual vendor BAA audit; add to calendar | 14 days | Open |
| _(add rows)_ | | | | |

---

## Section 6: Residual Risk Acceptance

After planned controls are implemented, document residual risk:

| Risk ID | Residual Likelihood | Residual Impact | Residual Score | Accepted By | Date |
|---------|--------------------|-----------------|--------------:|-------------|------|
| R-01 | 1 | 3 | 3 | | |
| R-02 | 1 | 2 | 2 | | |
| R-03 | 1 | 2 | 2 | | |
| R-04 | 1 | 2 | 2 | | |
| R-05 | 1 | 3 | 3 | | |

**Overall Residual Risk Accepted By**:

Name: _______________________________________________ Title: _______________________________________________

Signature: _______________________________________________ Date: _______________________________________________

---

## Section 7: Review History

| Review Date | Trigger | Changes Made | Reviewed By |
|-------------|---------|--------------|-------------|
| | Initial analysis | N/A | |
| | Annual review | | |
| | System change: | | |
| | Incident response: | | |

---

## Related Documents

- `templates/risk-register.md` — Detailed risk tracking
- `docs/09-breach-notification-procedure.md` — Response when a risk event occurs
- `checklists/pre-launch-audit.md` — Technical verification of controls
- `docs/11-administrative-policies.md` — Workforce and access policies
