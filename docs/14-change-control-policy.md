# Change Control Policy

> **Reference**: 45 CFR § 164.308(a)(1) (Security Management Process)
> Changes to PHI systems, access control logic, or BAA-covered services must be reviewed and approved before reaching production. This policy ensures the risk analysis stays current after every significant system change.

**Version**: 1.0
**Effective Date**: [YYYY-MM-DD]
**Review Cycle**: Annual, or after any significant system change

---

## Purpose

HIPAA requires that the organization's risk analysis be updated after any significant system change (45 CFR § 164.308(a)(1)). Without a defined change control process, the risk analysis becomes stale the moment a developer merges a pull request.

This policy establishes a lightweight process appropriate for a small nonprofit (~12 staff). The goal is not bureaucratic overhead; it is a short, reliable loop that ensures:

1. Changes touching PHI systems are reviewed before production deployment
2. The Security Officer approves high-risk changes
3. The risk register is evaluated after significant changes
4. Every production change is documented in the change log

---

## Scope: Which Changes Require This Process

Not every code change is in scope. The change control process applies whenever a proposed change touches any of the following:

### In-Scope Change Categories

| Category | Examples |
|---|---|
| **PHI data stores** | Cloud SQL schema changes (tables, columns, indexes); Cloud Storage bucket configuration (access controls, retention policies, lifecycle rules) |
| **Access control logic** | Firestore Security Rules; RBAC custom claims in Firebase Auth; IAM role bindings for any GCP service |
| **Cloud Functions** | New functions; modified functions handling or gating PHI access; changes to function service accounts or permissions |
| **BAA-covered service configuration** | Firebase project settings; GCP project IAM policies; Cloud SQL instance settings; Cloud Storage uniform bucket-level access settings |
| **Authentication configuration** | Firebase Auth settings; MFA enforcement; session timeout values |
| **CI/CD pipeline** | Changes to the deployment pipeline itself that could affect environment gates or approval steps |

### Out of Scope

Routine changes that do not affect PHI systems, access control, or BAA-covered services (such as updates to the public-facing website, non-PHI frontend code, or documentation) do not require this process. When in doubt, ask the Security Officer.

---

## Change Categories and Required Approvals

Changes are classified into two tiers based on risk level.

| Tier | Description | Required Steps |
|---|---|---|
| **Tier 1: Standard** | In-scope change (see above) that follows the normal code promotion path | Developer review → staging validation → Security Officer sign-off → production deploy → change log entry → risk register check |
| **Tier 2: Emergency** | Critical security patch that cannot wait for staging validation | Security Officer verbal approval → expedited deploy → written follow-up within 24 hours → change log entry → risk register check |

All Tier 1 and Tier 2 changes must be logged in the change log (see Section 4) and must trigger a risk register evaluation (see Section 5).

---

## Tier 1: Standard Change Procedure

### Step 1: Developer Preparation

Before submitting a change for review, the developer must:

- [ ] Identify whether the change is in-scope (use the table above)
- [ ] Write a brief change description: what is changing, why, and what the impact is if it fails
- [ ] Confirm the change has been tested in the dev environment
- [ ] Open a pull request (PR) with the change description in the PR body

### Step 2: Peer Review

- [ ] A second developer reviews the PR diff
- [ ] For Firestore Security Rules changes: the reviewer must verify the new rules against all defined RBAC role permutations (see `docs/05-rbac-design.md`)
- [ ] Reviewer approves or requests changes before the PR can merge

### Step 3: Staging Validation

- [ ] PR is merged to the staging branch and deployed to the staging environment
- [ ] Automated CI/CD tests run against staging (see `docs/13-environment-separation.md` §CI/CD Environment Gates)
- [ ] Functional validation is performed against synthetic data: no production PHI is used in staging
- [ ] If tests fail, the change does not advance to production

### Step 4: Security Officer Sign-Off

The Security Officer must review and approve the change before it deploys to production. Sign-off is required for every in-scope change, regardless of perceived risk level.

**Approval method**: Written approval via email is acceptable. Slack message approval is acceptable only if the workspace's message-history retention policy covers the full 6-year HIPAA documentation retention period required by 45 CFR § 164.316(b)(2)(i). A comment in the PR thread is also acceptable. Verbal approval is not sufficient for standard changes.

**What the Security Officer reviews**:
- The change description and PR diff (or a plain-language summary from the developer)
- Staging test results
- Whether the change introduces or modifies any access to PHI
- Whether the change affects any BAA-covered service configuration

**The Security Officer may approve, approve with conditions, or reject the change.** Rejected changes return to the developer for revision.

### Step 5: Production Deployment

- [ ] Security Officer approval is confirmed before the CI/CD pipeline is unblocked for production
- [ ] The deployment pipeline enforces the approval gate (see `docs/13-environment-separation.md`)
- [ ] After deployment, the developer confirms the change is functioning as expected in production
- [ ] If the production deployment fails, the developer initiates rollback and notifies the Security Officer

### Step 6: Change Log and Risk Register

- [ ] Developer creates a change log entry (see Section 4 below) within 24 hours of deployment
- [ ] Security Officer evaluates the risk register (see Section 5 below) to determine if the change creates or modifies any risks

---

## Tier 2: Emergency Change Procedure

An emergency change applies when a critical security vulnerability requires an immediate fix to production, for example a zero-day exploit in a dependency that grants unauthorized PHI access.

The emergency procedure exists to prevent harm while maintaining accountability. It is not a general-purpose shortcut for schedule pressure.

### Emergency Criteria

A change qualifies as an emergency only if **all three** of the following are true:

1. The vulnerability or failure poses an active or imminent risk to PHI confidentiality, integrity, or availability
2. Waiting for full staging validation would meaningfully increase the harm to clients or the organization
3. The Security Officer agrees that emergency handling is warranted

### Emergency Steps

1. **Verbal Security Officer approval**: The developer contacts the Security Officer directly (phone or in-person) and explains the situation. The Security Officer gives verbal approval to proceed.

2. **Expedited staging check**: Even under emergency conditions, deploy to staging and run automated tests if the timeline permits: even 15 minutes of staging validation reduces risk. If staging cannot be used, the developer documents why.

3. **Production deployment**: Proceed with the fix.

4. **Written follow-up within 24 hours**: The Security Officer documents the emergency approval in writing: an email to the organization's compliance file is sufficient. This written record must include:
   - What the emergency was
   - Why normal process was bypassed
   - Who approved it and when
   - What was deployed
   - Outcome of the deployment

5. **Change log entry**: Complete within 24 hours of deployment (see Section 4).

6. **Risk register evaluation**: Complete within 48 hours (see Section 5).

**Important**: Using the emergency procedure more than twice in a 12-month period without a documented pattern review is a signal that the standard process has a structural problem. The Security Officer should initiate a review of the development and deployment process.

---

## Change Log

All in-scope production changes, whether Tier 1 or Tier 2, must be recorded in the change log. The log is a lightweight written record, not a formal ITSM ticket system. The change log is the organization's primary evidence of compliance with the risk analysis update obligation under 45 CFR § 164.308(a)(1) and will be requested during HHS Office for Civil Rights audits or investigations.

**Where the log lives**: `[ORGANIZATION TO SPECIFY, e.g., a shared document, a dedicated GitHub repository page, or a designated folder in Google Drive]`

### Change Log Entry Format

Each entry must capture the following fields:

| Field | Description |
|---|---|
| **Change ID** | Sequential identifier: `CHG-YYYY-###` (e.g., `CHG-2025-001`) |
| **Date Deployed** | Date the change reached production |
| **Description** | What changed, in plain language (2–4 sentences) |
| **Change Category** | Which in-scope category applies (PHI data store / Access control / Cloud Function / BAA service / Auth config / CI/CD) |
| **Tier** | Tier 1 (standard) or Tier 2 (emergency) |
| **Developer** | Who implemented the change |
| **Approver** | Security Officer name and approval date |
| **PR / Commit Reference** | Link to the pull request or commit hash |
| **Risk Register Updated** | Yes / No / Not Required: with date if updated |
| **Staging Bypass** | If Tier 2: reason staging was partially or fully bypassed |

### Example Entry

| Field | Value |
|---|---|
| Change ID | CHG-2025-001 |
| Date Deployed | 2025-03-14 |
| Description | Added `case_closed_date` column to the `clients` table in Cloud SQL. Updated the Cloud Function `getClientRecord` to return this field. No existing data was modified. |
| Change Category | PHI data store; Cloud Function |
| Tier | Tier 1 |
| Developer | [Developer Name] |
| Approver | [Security Officer Name], approved 2025-03-13 |
| PR / Commit Reference | github.com/your-org/repo/pull/47 |
| Risk Register Updated | No: change determined not to introduce new risks |
| Staging Bypass | N/A |

---

## Risk Register Evaluation After Changes

Every in-scope change must trigger a review of `templates/risk-register.md` to determine whether the change introduces, modifies, or closes any risks.

This satisfies the requirement in 45 CFR § 164.308(a)(1) to update the risk analysis after significant system changes, and is consistent with Policy 1.2 (Risk Analysis) and Policy 8.1 in `docs/11-administrative-policies.md`. Policy 1.2 explicitly states that the risk analysis must be updated "after any significant system change"; any change meeting the in-scope criteria defined in this document constitutes a "significant system change" under 45 CFR § 164.308(a)(1).

### Who Performs the Evaluation

The Security Officer performs or delegates the risk register evaluation. For Tier 1 changes, this must be completed before the change log entry is marked closed. For Tier 2 (emergency) changes, completion within 48 hours of deployment is required.

### Evaluation Questions

For each in-scope change, the evaluator considers:

1. **Does this change expand the attack surface?** Examples: new Cloud Function endpoint, new IAM role, new storage bucket, new data field containing PHI.

2. **Does this change modify who can access PHI?** Examples: updated Firestore Security Rules, changed custom claims logic, modified service account permissions.

3. **Does this change affect data integrity controls?** Examples: schema changes, validation changes in Cloud Functions, modified data transformation logic.

4. **Does this change affect audit logging?** Examples: modified Cloud Function that generates access logs, changed retention configuration.

5. **Does this change affect a BAA-covered service configuration?** Any configuration change to a Google service covered by the GCP BAA that could affect PHI handling.

### Outcomes

| Evaluation Outcome | Action Required |
|---|---|
| No new risks identified | Document in change log: "Risk register reviewed: no new risks identified." No further action. |
| New risk identified | Add a new entry to `templates/risk-register.md` using the standard risk ID format (R-YYYY-###). Assign a risk owner and response. |
| Existing risk closed by this change | Update the existing risk register entry to reflect the changed status. |
| Existing risk modified (higher or lower) by this change | Update the risk score and response in the existing entry. Document the change that triggered the update. |

---

## CI/CD Technical Enforcement

This policy is supported by the technical environment gate structure described in `docs/13-environment-separation.md`. The CI/CD pipeline enforces:

- Automated test passage before staging deployment
- Manual approval gate (Security Officer) before production deployment
- Deployment audit logging for all production deploys
- Firestore Security Rules changes trigger an additional automated emulator test suite before the approval gate opens

The policy and the technical gate structure are complementary: the policy defines what must happen; the CI/CD pipeline makes it structurally difficult to skip those steps.

**Policy takes precedence over technical capability.** If a developer has the technical ability to bypass the approval gate (e.g., through direct GCP console access), doing so without Security Officer approval is a policy violation and subject to the sanctions in `docs/11-administrative-policies.md` §1.4.

---

## Retention

All change log entries and associated approval documentation must be retained for **6 years** from the date of the change, consistent with the general HIPAA documentation retention requirement (45 CFR § 164.316(b)(2)(i)).

---

## Related Documents

- `docs/13-environment-separation.md`: Three-environment strategy and CI/CD gate structure that technically enforces this policy
- `templates/risk-register.md`: Living risk register; updated after every in-scope change
- `docs/08-risk-analysis-template.md`: NIST-aligned risk analysis methodology
- `docs/11-administrative-policies.md`: Policy 8.1 requires risk analysis after significant changes; §1.4 defines sanctions for policy violations
- `docs/05-rbac-design.md`: RBAC role definitions referenced during Security Rules peer review
