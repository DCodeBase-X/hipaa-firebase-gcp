# HIPAA Fundamentals — What You're Actually Required to Do

This isn't a legal overview. It's a technical translation: what HIPAA's Security Rule requires, mapped to decisions you'll make when building on Firebase and GCP.

---

## What HIPAA Protects

**PHI (Protected Health Information):** Any health information that can be linked to a specific individual. This includes:

- Medical records, diagnoses, treatment history
- Mental health records and substance use treatment records
- Criminal justice records when collected in a healthcare or social services context
- Children's services records when health or behavioral data is involved
- Social Security numbers, dates of birth, addresses — when combined with health information
- Intake forms, assessments, case notes

For nonprofits doing reentry, housing, behavioral health, or social services: **your intake forms almost certainly collect PHI.** If you're asking about someone's health history, medications, mental health status, prior treatment, or disability status — that's PHI.

---

## The Three Safeguard Categories

HIPAA's Security Rule breaks into three types of safeguards. Here's what each means in practice:

### Administrative Safeguards
Policies, procedures, and training. Not code — decisions.

| Requirement | What It Means for Your Org |
|---|---|
| Security Officer | Designate someone responsible for HIPAA compliance |
| Risk Analysis | Document where PHI lives and what threats exist |
| Workforce Training | Train staff who access PHI on proper handling |
| Access Management | Define who gets access to what and why |
| Incident Response | Have a documented plan for breaches |

### Physical Safeguards
Controls over physical access to systems. For cloud-first nonprofits this is mostly handled by GCP, but you still own some of it.

| Requirement | Who Handles It |
|---|---|
| Facility access controls for servers | Google (covered under BAA) |
| Workstation security | You — devices that access PHI must be secured |
| Device and media controls | You — enforce device encryption, MDM policies |

### Technical Safeguards
This is where your architecture lives.

| Requirement | Technical Implementation |
|---|---|
| **Access Control** | Firebase Auth + Firestore Security Rules + IAM roles |
| **Audit Controls** | Cloud Audit Logs enabled for all PHI-touching services |
| **Integrity Controls** | Cloud SQL checksums, Firestore document versioning |
| **Person Authentication** | Firebase Auth with MFA enforced for staff roles |
| **Transmission Security** | HTTPS/TLS everywhere — Cloudflare + Firebase enforce this |

---

## The Minimum Necessary Standard

You must limit PHI access to the minimum necessary for a person to do their job.

In Firebase terms:
- A case manager should only read records for their assigned clients — not all clients
- A volunteer coordinator should have no access to clinical data
- An admin viewing reports should see aggregated data, not individual PHI
- Cloud Functions should only query the PHI fields they actually need

This is an architecture decision, not just a policy decision. Your Firestore Security Rules and Cloud SQL query patterns need to enforce it.

---

## Business Associate Agreements (BAAs)

A **Business Associate** is any vendor that handles PHI on your behalf. You need a signed BAA with every one of them.

**Required BAAs for a typical Firebase/GCP nonprofit stack:**
- Google Cloud (covers Firebase services that are in-scope — sign in Cloud Console)
- Salesforce (if using for case management — tier-dependent)
- Any email provider that sends messages containing PHI
- Any document storage provider that holds PHI files

**Not required (because PHI doesn't touch these services):**
- Hostinger (public site only — no PHI)
- Stripe / Donorbox (payment data only — PCI, not HIPAA)
- Cloudflare (if PHI is only in transit, encrypted — but get it in writing anyway)

**How to sign the Google BAA:**
1. Go to Google Cloud Console → IAM & Admin → Settings
2. Scroll to "HIPAA Business Associate Amendment"
3. Review and accept — this covers all GCP services that appear on Google's BAA-eligible list
4. **This must be done before you store any PHI in GCP/Firebase**

---

## Breach Notification

If PHI is improperly disclosed, HIPAA requires notification:
- To affected individuals: within 60 days of discovery
- To HHS: annually (or immediately if >500 individuals affected)
- To media: if >500 individuals in a state are affected

For cloud architecture, most breach scenarios come from:
- Misconfigured Firestore Security Rules (open access)
- Overly permissive IAM roles
- PHI logged in Cloud Functions logs (which are often less protected)
- PHI accidentally passed to uncovered services (Analytics, Crashlytics)

---

## What "HIPAA Compliant" Actually Means

There is no HIPAA certification. No government body audits your system and stamps it compliant. HIPAA compliance is self-attested — you assess your own risks, implement safeguards, document what you did, and are accountable if something goes wrong.

This means:
1. Architecture alone isn't enough — you need policies and training too
2. Documentation matters — your risk analysis and BAAs are your evidence
3. "Google is HIPAA compliant" doesn't mean your app is — you must configure it correctly

The architecture in this repo implements the technical safeguards. The administrative and physical safeguards — policies, training, incident response plans — are your organization's responsibility to develop.

---

**Next:** [Three-Layer Architecture →](02-three-layer-architecture.md)
