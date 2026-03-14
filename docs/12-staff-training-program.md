# HIPAA Staff Training Program

> **Reference**: 45 CFR § 164.308(a)(5) — Security Awareness and Training (Administrative Safeguard)
> 45 CFR § 164.530(b) — Training (Privacy Rule)

HIPAA requires training for all workforce members whose work involves PHI. Training must occur at hire and periodically thereafter. This document provides a complete training curriculum, delivery guidance, and tracking system.

---

## Training Requirements Summary

| Workforce Segment | Initial Training | Annual Refresher | Documentation Required |
|-------------------|-----------------|-----------------|----------------------|
| All staff (any PHI access) | Before access granted | Yes | Signed attestation |
| Clinical staff (direct PHI access) | Before access granted | Yes | Signed attestation + role training |
| Program staff (limited PHI access) | Before access granted | Yes | Signed attestation |
| Volunteers (no PHI access) | At onboarding | Recommended | Attestation |
| IT / Developers | Before system access | Yes | Technical training attestation |
| Executive leadership | At hire / appointment | Yes | Signed attestation |
| New contractors / Business Associates | Before work begins | Per BAA terms | BAA + attestation |

**Training records must be retained for 6 years.**

---

## Module 1: HIPAA Fundamentals (All Staff — 60 minutes)

### Learning Objectives

After completing this module, the workforce member will be able to:
- Define PHI and identify examples relevant to our organization
- Explain the three HIPAA rules and their relevance to their role
- Describe the minimum necessary standard
- Identify who to contact when they have questions or suspect a violation

### Topics

**1.1 What is HIPAA and Why It Matters**
- Brief history and purpose of HIPAA
- Who HIPAA applies to (Covered Entities and Business Associates)
- Consequences of violations: civil penalties ($100–$50,000+ per violation), criminal charges
- Our organization's specific obligations

**1.2 What is PHI?**
- The 18 HIPAA identifiers (name, DOB, SSN, address, phone, email, MRN, etc.)
- PHI specific to our client population: health records, mental health, substance use, criminal justice, children's data
- ePHI vs. paper PHI
- What is NOT PHI (de-identified data, aggregate statistics)

**1.3 The Three HIPAA Rules**
- Privacy Rule: client rights, permitted uses and disclosures, minimum necessary
- Security Rule: administrative, physical, technical safeguards for ePHI
- Breach Notification Rule: what constitutes a breach, our reporting obligations

**1.4 Minimum Necessary Standard**
- Only access PHI needed for your specific job function
- Don't look up client records out of curiosity, even for clients you know
- Don't share PHI beyond what the recipient needs

**1.5 Client Rights**
- Right to access their own records
- Right to request amendment
- Right to an accounting of disclosures
- Right to request restrictions
- How to direct client rights requests to the Privacy Officer

**Assessment**: 10-question quiz, minimum 80% to pass

---

## Module 2: Security Practices (All Staff — 45 minutes)

### Learning Objectives

After completing this module, the workforce member will be able to:
- Apply password and authentication best practices
- Identify phishing and social engineering attempts
- Follow proper procedures for workstation and device security
- Report security incidents correctly

### Topics

**2.1 Password and Authentication Security**
- Minimum 12-character passwords with complexity
- Never reuse passwords across systems
- Use of a password manager (recommended tools)
- Multi-factor authentication: what it is, how to set it up, why it's required
- Never share passwords — not even with the Security Officer

**2.2 Phishing and Social Engineering**
- How to identify phishing emails (urgency, suspicious links, unexpected attachments)
- Vishing (voice phishing) and pretexting tactics
- What to do if you receive a suspicious email: do not click — forward to Security Officer
- Real examples relevant to healthcare nonprofits

**2.3 Device and Workstation Security**
- Lock your screen when leaving it unattended (5-minute auto-lock is mandatory)
- Never leave PHI visible on screen in public or shared spaces
- Full-disk encryption is required on all work devices
- No personal use of work devices for ePHI-adjacent tasks

**2.4 Safe Communication of PHI**
- Approved methods: in-app messaging, encrypted email (verify before sending)
- Never email PHI via personal email or non-encrypted channels
- Never send PHI via SMS/text
- Never discuss PHI in public areas, elevators, or on speakerphone
- Fax machines: confirm recipient before sending; retrieve immediately upon delivery

**2.5 Incident Reporting**
- What to report: suspicious emails, lost devices, unexpected system behavior, suspected unauthorized access
- When to report: immediately — same day
- Who to report to: Security Officer (contact info on file)
- What happens after you report (brief overview of incident response)

**Assessment**: Scenario-based quiz, 5 scenarios, minimum 4 correct

---

## Module 3: Role-Specific Training

### Module 3A: Clinical Staff (Additional 45 minutes)

For staff with direct access to health records, mental health data, SUD records, and case notes.

**Topics**:
- Your specific access role and what data you can access
- The principle of clinical staff assignment: you only access records for clients assigned to you
- Case notes and documentation best practices (what to record, what not to record)
- Handling especially sensitive PHI: mental health, SUD, HIV status — additional protections apply
- Disclosure requirements and restrictions (e.g., 42 CFR Part 2 for SUD records)
- Telehealth and remote session security

### Module 3B: IT and Developer Staff (Additional 60 minutes)

For staff with access to system configuration, code, databases, or infrastructure.

**Topics**:
- Our three-layer architecture and data flow (see `docs/02-three-layer-architecture.md`)
- What data lives where: Cloud SQL (PHI only), Firestore (operational), Layer 1 (no PHI)
- Firebase services that are NOT covered by Google's BAA — never store PHI there
- Firestore Security Rules: deny-all default, role-based access patterns
- Cloud Functions: authentication, role validation, no PHI in logs
- Secret Manager: all credentials must live here; never hard-code secrets
- VPC and private IP configuration for Cloud SQL
- Cloud Audit Logs: what they capture, why they cannot be disabled
- Secure coding practices: input validation, parameterized queries, output encoding
- Incident response: your role when a technical incident occurs

### Module 3C: Administrative and Program Staff (Additional 30 minutes)

For staff with access to client operational data but not direct PHI.

**Topics**:
- Your access role: what you can and cannot access
- Minimum necessary in daily operations (scheduling, intake forms, referrals)
- Handling paper documents with PHI
- Proper disposal of PHI (shredding)
- Redirecting client rights requests to the Privacy Officer

---

## Module 4: Breach Recognition and Reporting (All Staff — 30 minutes)

### Learning Objectives

After completing this module, the workforce member will be able to:
- Recognize situations that may constitute a HIPAA breach
- Know exactly what to do in the first hour after discovering a potential breach
- Understand their role in breach response

### Topics

**4.1 What Is a Breach?**
- Definition: unauthorized access, use, or disclosure of PHI
- Examples: emailing PHI to the wrong person, lost device, misconfigured system, hacked account
- Your job: report it, don't investigate it yourself

**4.2 First Response Steps**
1. Stop the potential breach if possible (close the browser, lock the device, stop the email)
2. Do not delete anything
3. Do not attempt to investigate or fix it yourself
4. Contact the Security Officer immediately — phone or in-person
5. Document what you observed: time, what you saw, what you did

**4.3 What Happens Next**
- Security Officer leads investigation and breach determination
- You will be asked for a written statement
- Depending on outcome: no action, internal action, or formal breach notification process
- Your cooperation is required; your good-faith report is protected

**Assessment**: 5-question true/false quiz, 100% required

---

## Training Delivery Options

| Format | Appropriate For | Notes |
|--------|----------------|-------|
| In-person workshop | Initial hire training, annual refresher | Most effective; allows Q&A |
| Video + quiz (LMS) | Ongoing, remote staff | HIPAA-specific LMS options: KnowBe4, Compliancy Group, Accountable HQ |
| Self-paced reading + quiz | Supplemental modules | Less effective alone; better as reinforcement |
| Table-top exercises | Breach scenarios, incident response | Annual; executive and clinical staff |

**Recommended LMS for nonprofits**: Accountable HQ (nonprofit pricing available) or HHS's free ONC training resources.

---

## Training Completion Tracking

### Individual Training Record

| Workforce Member | Role | Module 1 | Module 2 | Module 3 | Module 4 | Annual Refresher | Signed Attestation on File |
|-----------------|------|---------|---------|---------|---------|-----------------|--------------------------|
| | | Date / Score | Date / Score | Date / Score | Date / Score | Date | Y / N |

### Training Attestation Form

---

I, _______________________________________________, confirm that I have completed the following HIPAA training modules and understand my responsibilities under HIPAA:

- [ ] Module 1: HIPAA Fundamentals — Completed: _______________ Score: ___/10
- [ ] Module 2: Security Practices — Completed: _______________ Score: ___/5
- [ ] Module 3 (role-specific): _______________ — Completed: _______________
- [ ] Module 4: Breach Recognition — Completed: _______________ Score: ___/5

I understand that:
- I am responsible for protecting PHI in all forms (electronic and paper)
- I must report suspected security incidents or breaches immediately
- Violations of HIPAA policies may result in disciplinary action, termination, or legal consequences
- I may ask the Privacy or Security Officer any questions about my obligations

Signature: _______________________________________________ Date: _______________________________________________

Name (printed): _______________________________________________ Title: _______________________________________________

Department: _______________________________________________ Supervisor: _______________________________________________

---

**Training administered by**: _______________________________________________ Date: _______________________________________________

---

## Annual Refresher Requirements

Annual refresher training must be completed by December 31 of each year. Topics to include:

- Review of any policy changes from the prior year
- New or emerging threats (phishing trends, new attack vectors)
- Lessons learned from any internal incidents
- Any changes to the technical environment (new systems, deprecated services)
- Regulatory updates (new HHS guidance, state law changes)

**Annual refresher schedule**:

| Year | Training Date | Format | Completion Rate | Notes |
|------|--------------|--------|----------------|-------|
| | | | | |

---

## Related Documents

- `docs/11-administrative-policies.md` — Sanction policy and workforce security
- `docs/10-physical-safeguards.md` — Device and workstation policies reviewed in training
- `docs/09-breach-notification-procedure.md` — Breach response covered in Module 4
- `docs/05-rbac-design.md` — Role definitions referenced in Module 3
