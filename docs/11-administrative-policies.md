# HIPAA Administrative Policies

> **Reference**: 45 CFR § 164.308 — Administrative Safeguards
> Administrative safeguards are the policies and procedures that govern how the organization manages ePHI security at the people and process level.

---

## Policy 1: Security Management Process

**Reference**: 45 CFR § 164.308(a)(1)

### 1.1 Purpose

Establish a security management process to prevent, detect, contain, and correct security violations.

### 1.2 Risk Analysis

The organization shall conduct and document a thorough risk analysis of all ePHI at least annually and after any significant system change. See `docs/08-risk-analysis-template.md`.

### 1.3 Risk Management

Following each risk analysis, the organization shall implement security measures sufficient to reduce identified risks to a reasonable and appropriate level. Risk responses shall be documented in `templates/risk-register.md`.

### 1.4 Sanction Policy

Workforce members who violate HIPAA policies are subject to sanctions proportional to the severity of the violation:

| Severity | Examples | Sanction |
|----------|----------|----------|
| **Minor** | Forgetting to lock screen, sharing a password once | Verbal warning + retraining |
| **Moderate** | Accessing PHI outside job role, unencrypted device | Written warning + suspension pending review |
| **Severe** | Deliberate PHI disclosure, sharing PHI with unauthorized parties | Immediate termination + report to authorities if applicable |
| **Critical** | Sale or malicious use of PHI | Immediate termination + law enforcement referral |

- Sanctions apply regardless of whether harm resulted
- Sanctions shall be applied consistently regardless of the workforce member's seniority
- All sanctions shall be documented and retained for 6 years
- Retaliation against workforce members who report violations is prohibited

### 1.5 Information System Activity Review

The Security Officer or designee shall review system activity at the following intervals:

| Activity | Review Frequency |
|----------|-----------------|
| Cloud Audit Logs (GCP) | Monthly minimum; alerts reviewed within 24 hours |
| Firebase Auth unusual activity | Weekly |
| Failed login attempts | Weekly |
| PHI access logs (Cloud SQL) | Monthly |
| Budget/billing alerts | Real-time (automated) |
| Security Command Center findings | Monthly |

---

## Policy 2: Assigned Security Responsibility

**Reference**: 45 CFR § 164.308(a)(2)

### 2.1 Security Officer

The organization shall designate a HIPAA Security Officer responsible for:

- Developing and implementing security policies and procedures
- Conducting or overseeing risk analyses
- Responding to security incidents
- Maintaining HIPAA compliance documentation
- Serving as the primary contact for HHS in the event of an audit or investigation
- Reviewing system activity logs
- Coordinating staff training

**Current Security Officer**:

Name: _______________________________________________ Title: _______________________________________________ Email: _______________________________________________ Phone: _______________________________________________

**Designated**: _______________________________________________ (date) **Appointment documented by**: _______________________________________________

### 2.2 Privacy Officer

The organization shall designate a HIPAA Privacy Officer (may be the same person as the Security Officer for small organizations) responsible for:

- Developing and implementing privacy policies
- Handling patient/client rights requests (access, amendment, accounting of disclosures)
- Managing breach notifications
- Ensuring minimum necessary standard is applied to all PHI access

**Current Privacy Officer**:

Name: _______________________________________________ Title: _______________________________________________ Email: _______________________________________________ Phone: _______________________________________________

### 2.3 Succession Planning

In the event the Security or Privacy Officer role becomes vacant:
- The Executive Director assumes responsibilities temporarily
- A replacement must be designated within **30 days**
- Role transition must be documented

---

## Policy 3: Workforce Security

**Reference**: 45 CFR § 164.308(a)(3)

### 3.1 Authorization and Supervision

- Access to ePHI shall be granted only to workforce members who require it to perform their job functions (minimum necessary standard)
- Access levels shall be defined by role as specified in `docs/05-rbac-design.md`
- All access grants must be approved by the Security Officer or a designated manager
- Access shall be granted at the time of hire and reviewed at least annually

### 3.2 Workforce Clearance Procedure

Before granting ePHI access, verify:

- [ ] Background check completed (required for all roles accessing PHI)
- [ ] HIPAA training completed and documented (see `docs/12-staff-training-program.md`)
- [ ] Physical safeguards policy acknowledged (see `docs/10-physical-safeguards.md`)
- [ ] This administrative policy acknowledged (sign below or on file)
- [ ] MFA enrolled in Firebase Auth
- [ ] Role assigned in Firebase Auth custom claims (by admin only)
- [ ] Access limited to assigned clients only (for clinical/program staff)

### 3.3 Termination Procedures

Upon separation of any workforce member (voluntary or involuntary), complete the following **on the last day of employment** (or immediately for involuntary terminations):

- [ ] Revoke Firebase Auth account
- [ ] Remove all custom claims/roles
- [ ] Revoke GCP IAM access
- [ ] Revoke Salesforce access
- [ ] Revoke email and collaboration tool access
- [ ] Remote wipe organization-owned device (if applicable)
- [ ] Recover all organization-owned devices and access cards
- [ ] Change any shared passwords the individual knew
- [ ] Document all revocations with date and time

**Termination checklist completed by**: _______________________________________________ Date: _______________________________________________

---

## Policy 4: Information Access Management

**Reference**: 45 CFR § 164.308(a)(4)

### 4.1 Isolating Healthcare Clearinghouse Functions

Not applicable to most nonprofits. If your organization operates a healthcare clearinghouse function, document it separately.

### 4.2 Access Authorization

- Access to ePHI systems requires written or documented authorization
- Only the Security Officer or designated admin may grant access
- Access requests must specify: system, role, business justification, and approver

### 4.3 Access Establishment and Modification

- Role changes require a new access review and update within **5 business days**
- Access shall be modified immediately upon role change or reduction in responsibilities
- All access changes shall be logged

### 4.4 Access Reviews

- All ePHI access shall be reviewed quarterly
- Review shall confirm that current access matches current role
- Unused accounts (90+ days inactive) shall be suspended pending review

---

## Policy 5: Security Awareness and Training

**Reference**: 45 CFR § 164.308(a)(5)

See `docs/12-staff-training-program.md` for the full training program.

### 5.1 Training Requirements

- All workforce members with any access to ePHI must complete HIPAA training before being granted access
- Annual refresher training required for all staff
- Training must be documented; records retained for 6 years

### 5.2 Security Reminders

The Security Officer shall issue security reminders at least quarterly, covering topics such as:
- Phishing awareness
- Password hygiene
- Incident reporting procedures
- Recent changes to policies or procedures

### 5.3 Malicious Software Protection

- All workstations must run endpoint protection software
- Staff must not disable antivirus or security software
- Staff must report suspicious emails to the Security Officer before clicking links or attachments

### 5.4 Login Monitoring

- Staff must immediately report failed login attempts or unexpected account activity
- Staff must not share login credentials under any circumstances
- Staff must report any suspected account compromise immediately

---

## Policy 6: Security Incident Procedures

**Reference**: 45 CFR § 164.308(a)(6)

### 6.1 Identifying Security Incidents

A security incident is any attempted or successful unauthorized access, use, disclosure, modification, or destruction of ePHI or ePHI systems.

All workforce members must report suspected incidents to the Security Officer **immediately** — no exceptions.

**Security Officer contact for incidents**:

Name: _______________________________________________ Phone (24/7): _______________________________________________ Email: _______________________________________________

### 6.2 Response and Reporting

Follow the incident response runbook: `checklists/incident-response-runbook.md`

For breach determination and notification requirements: `docs/09-breach-notification-procedure.md`

### 6.3 Documentation

All security incidents must be documented regardless of whether they result in a reportable breach. Documentation must include:
- Date and time of discovery
- Description of the incident
- Systems and data affected
- Individuals potentially affected
- Response actions taken
- Breach determination outcome
- Corrective actions implemented

---

## Policy 7: Contingency Plan

**Reference**: 45 CFR § 164.308(a)(7)

### 7.1 Data Backup Plan

- Cloud SQL: Automated daily backups with Point-in-Time Recovery (PITR) enabled (see `checklists/gcp-configuration.md`)
- Cloud Storage: Object versioning enabled; soft delete enabled
- Firestore: Automated daily backups to Cloud Storage
- Backup retention: Minimum 30 days for operational data; 7 years for records subject to retention requirements

### 7.2 Disaster Recovery Plan

| Scenario | Recovery Procedure | RTO Target | RPO Target |
|----------|--------------------|-----------|-----------|
| Cloud SQL instance failure | GCP auto-failover (HA configuration) | < 5 minutes | < 5 minutes |
| Cloud SQL region outage | Restore from most recent backup | < 4 hours | < 24 hours |
| Data corruption | PITR restore to pre-corruption point | < 4 hours | < 1 hour |
| Firebase Auth outage | Read-only mode; no new logins | N/A (vendor) | N/A (vendor) |
| Full GCP region outage | Evaluate cross-region restore | < 24 hours | < 24 hours |

**Recovery test schedule**: Disaster recovery procedures shall be tested at least **annually**. Document test date, outcome, and any gaps identified.

### 7.3 Emergency Mode Operation

In the event of a system outage during which ePHI access is needed for patient care:
- Document manual procedures for critical care decisions
- Identify which workforce members have authority to access backup systems
- Ensure paper-based emergency records are available for critical client information

### 7.4 Applications and Data Criticality Analysis

| System | Criticality | Justification |
|--------|-------------|---------------|
| Cloud SQL (PHI) | Critical | Active client records required for service delivery |
| Cloud Functions | Critical | Required to access all PHI |
| Firebase Auth | Critical | Required for all staff authentication |
| Firestore | High | Operational coordination data |
| Cloud Storage | High | Document management |
| Firebase Hosting / WordPress | Medium | Client-facing; no PHI |

---

## Policy 8: Evaluation

**Reference**: 45 CFR § 164.308(a)(8)

### 8.1 Periodic Technical and Non-Technical Evaluation

The organization shall conduct evaluations of its security practices:

| Evaluation Type | Frequency | Owner |
|-----------------|-----------|-------|
| Risk analysis | Annually + after significant changes | Security Officer |
| Firestore Security Rules audit | Quarterly | Lead Developer |
| GCP IAM access review | Quarterly | Security Officer |
| Vendor BAA status review | Annually | Privacy Officer |
| Physical safeguards walkthrough | Annually | Security Officer |
| Penetration test / security scan | Annually (or when budget allows) | External vendor |
| Staff training completion review | Annually | Security Officer |

---

## Policy 9: Business Associate Contracts

**Reference**: 45 CFR § 164.308(b)(1)

### 9.1 BAA Requirement

The organization shall not permit a Business Associate to create, receive, maintain, or transmit ePHI on its behalf without a signed, HIPAA-compliant Business Associate Agreement in place.

### 9.2 BAA Inventory

Maintain a current inventory of all BAAs:

| Vendor | Service | BAA Signed | BAA Date | BAA Expiry / Review Date | Contact |
|--------|---------|-----------|----------|--------------------------|---------|
| Google LLC (Firebase/GCP) | Cloud infrastructure | Yes | | Annual review | Google Cloud Console |
| Salesforce, Inc. | CRM (NPSP) | Verify by tier | | | |
| Cloudflare, Inc. | CDN / WAF | Business+ plan only | | | |
| _(other vendors)_ | | | | | |

- [ ] Review BAA inventory annually
- [ ] Remove any vendor receiving ePHI that cannot or will not sign a BAA

---

## Acknowledgment of Administrative Policies

I, _______________________________________________, acknowledge that I have read, understood, and will comply with the organization's HIPAA Administrative Policies as of the date signed below.

Signature: _______________________________________________ Date: _______________________________________________

Name (printed): _______________________________________________ Title: _______________________________________________

---

## Related Documents

- `docs/05-rbac-design.md` — Access control implementation
- `docs/08-risk-analysis-template.md` — Risk analysis methodology
- `docs/09-breach-notification-procedure.md` — Incident response and notification
- `docs/10-physical-safeguards.md` — Physical security policies
- `docs/12-staff-training-program.md` — Training program details
- `checklists/incident-response-runbook.md` — Operational incident response
- `templates/risk-register.md` — Risk tracking
