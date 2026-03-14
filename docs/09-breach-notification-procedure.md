# HIPAA Breach Notification Procedure

> **Reference**: 45 CFR §§ 164.400–414 — Breach Notification Rule
> **Also see**: `checklists/incident-response-runbook.md` for the step-by-step operational response.

---

## Overview

A **breach** under HIPAA is the acquisition, access, use, or disclosure of Protected Health Information (PHI) in a manner not permitted by the Privacy Rule that compromises the security or privacy of the PHI.

This procedure defines:
1. How to determine if an event is a reportable breach
2. Who must be notified, and when
3. What the notifications must contain
4. How to document the incident

**Default assumption**: Treat every unauthorized PHI access as a breach until the four-factor risk assessment proves otherwise.

---

## Section 1: Breach vs. Non-Breach Determination

Not every security incident is a reportable breach. Apply the **four-factor risk assessment** below to determine if notification is required.

### 1.1 Threshold: Was PHI Involved?

First, confirm that the incident involved PHI (as defined in `docs/06-data-classification.md`). If no PHI was involved, this is a security incident, not a HIPAA breach — document it but notification is not required.

### 1.2 Four-Factor Risk Assessment

If PHI was involved, assess all four factors. If **any factor indicates high probability of compromise**, the incident is presumed a reportable breach.

| Factor | Questions to Answer | Assessment |
|--------|--------------------|-----------:|
| **1. Nature and extent of PHI** | What types of PHI were involved? How many individuals? Did it include especially sensitive data (SUD, mental health, criminal justice)? | Low / Medium / High |
| **2. Who accessed/received the PHI** | Was it another covered entity? A person obligated to protect it? An unknown external party? | Low / Medium / High |
| **3. Whether PHI was actually acquired or viewed** | Did the unauthorized party actually see or download the data, or was it merely accessible? | Low / Medium / High |
| **4. Extent to which risk has been mitigated** | Was the data recovered? Was the unauthorized party identified and confirmed to have not retained or shared it? | Low / Medium / High |

**Outcome**:
- If risk to individuals is **low probability**: Document the assessment and rationale. No notification required.
- If risk is **not low probability**: Proceed to Section 2. This is a reportable breach.

**Decision documented by**: _______________________________________________ Date: _______________________________________________

---

## Section 2: Notification Requirements

### 2.1 Notification Timeline

| Recipient | Deadline | Trigger |
|-----------|----------|---------|
| Affected individuals | **60 calendar days** from discovery | All reportable breaches |
| HHS (Secretary) | **60 calendar days** from discovery (breaches ≥ 500 individuals) | Large breaches |
| HHS (Secretary) — annual log | **60 days after end of calendar year** | Small breaches (< 500 individuals) |
| Prominent media outlets | **60 calendar days** from discovery | Breaches ≥ 500 individuals **in a single state or jurisdiction** |
| Business Associates | **Without unreasonable delay**, not to exceed 60 days | If BA caused or discovered the breach |

> **Discovery date**: The first day the organization knew or should have known a breach occurred — not when the investigation is complete.

### 2.2 Required Content: Individual Notification

Each notification to affected individuals **must** include:

1. A brief description of what happened, including the date of the breach and the date of discovery (if known)
2. The types of unsecured PHI involved (e.g., name, DOB, health record number, diagnosis)
3. Steps individuals should take to protect themselves from potential harm
4. A brief description of what the organization is doing to investigate, mitigate harm, and prevent future incidents
5. Contact information for individuals to ask questions (toll-free number, email, mailing address, or website)

See `templates/breach-notification-letter.md` for a pre-drafted template.

### 2.3 Notification Methods

**Individual notification** (in order of preference):
- **Written notice** by first-class mail to the individual's last known address
- **Email** if the individual has agreed to receive electronic notices
- **Substitute notice** if contact information is insufficient or out of date:
  - < 10 individuals: Alternative written, telephone, or other means
  - ≥ 10 individuals: Conspicuous posting on the organization's website for 90 days, OR notice in major print/broadcast media in the affected area

**HHS notification**:
- Report via the HHS Breach Reporting Portal: https://ocrportal.hhs.gov/ocr/breach/wizard_breach.jsf
- For small breaches (< 500): Log internally; submit annual report to HHS by March 1 of the following year

**Media notification** (breaches ≥ 500 in a state/jurisdiction):
- Contact prominent print and broadcast media in the affected area
- Provide the same content as individual notifications
- Document which outlets were contacted and when

---

## Section 3: Breach Response Roles

| Role | Responsibility |
|------|---------------|
| **Security Officer** | Leads investigation; determines breach vs. non-breach; authorizes notifications |
| **Privacy Officer** | Drafts and sends individual notifications; maintains breach log |
| **Legal Counsel** | Reviews notifications before sending; advises on state law requirements |
| **Executive Director / CEO** | Approves media notifications; signs HHS report |
| **IT / Developer** | Provides technical details of the incident; implements remediation |
| **Communications** | Manages public messaging if media notification required |

---

## Section 4: State Law Considerations

HIPAA sets the federal floor. Many states have breach notification laws that are **stricter** — shorter timelines, broader definitions of personal information, or additional notification requirements. Always verify your state's law:

| Consideration | Check Required |
|---------------|---------------|
| State notification timeline (may be < 60 days) | Yes |
| State definition of personal information (may be broader than PHI) | Yes |
| State-specific required notification content | Yes |
| State Attorney General notification requirement | Yes |
| Credit monitoring requirement | Some states |

**Your state**: _______________________________________________ **State law reference**: _______________________________________________

---

## Section 5: Documentation Requirements

HIPAA requires documentation of **all** breach determinations — including incidents determined NOT to be reportable breaches. Retain for **6 years** from the date of creation or last effective date.

### 5.1 Breach Incident Log

Maintain a running log of all security incidents and breach determinations:

| Incident ID | Discovery Date | Description | PHI Involved | Individuals Affected | Breach Determination | Notifications Sent | Documented By |
|-------------|---------------|-------------|--------------|---------------------|---------------------|-------------------|---------------|
| | | | Y / N | | Breach / Not Breach | Y / N | |

### 5.2 Per-Incident Documentation Package

For each incident, retain:
- [ ] Initial incident report
- [ ] Four-factor risk assessment worksheet (completed)
- [ ] Breach/non-breach determination with rationale
- [ ] Copies of all notifications sent (individual, HHS, media)
- [ ] Dates notifications were sent
- [ ] Evidence of delivery where applicable
- [ ] Investigation findings and timeline
- [ ] Corrective action plan and implementation evidence
- [ ] Final incident closure report

---

## Section 6: Business Associate Breach Notifications

If a Business Associate discovers a breach:

- The BA must notify your organization **without unreasonable delay** and no later than **60 days** after discovery
- Your organization's 60-day notification clock to individuals/HHS starts on the **date the BA discovered the breach**, not the date they notified you
- Ensure your BAA with each vendor explicitly requires timely breach notification

**BA Notification Contact**:

Name: _______________________________________________ Email: _______________________________________________ Phone: _______________________________________________

---

## Section 7: Post-Breach Review

Within 30 days of breach resolution, conduct a lessons-learned review:

- [ ] Root cause identified and documented
- [ ] Corrective action plan developed and assigned
- [ ] Risk analysis updated to reflect new threat/vulnerability
- [ ] Policies and procedures updated if gaps were identified
- [ ] Staff retrained if human error was a contributing factor
- [ ] Controls tested to confirm remediation is effective

**Review date**: _______________________________________________ **Facilitated by**: _______________________________________________

---

## Related Documents

- `templates/breach-notification-letter.md` — Pre-drafted individual notification letter
- `checklists/incident-response-runbook.md` — Step-by-step operational response
- `docs/08-risk-analysis-template.md` — Update after any breach
- `docs/11-administrative-policies.md` — Sanctions and workforce policies
