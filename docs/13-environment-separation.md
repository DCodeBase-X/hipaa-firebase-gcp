# Environment Separation Strategy

> **Reference**: 45 CFR § 164.308(a)(1) — Security Management Process
> Using production PHI in development or testing environments is a HIPAA violation. This document defines the required environment structure and the rules governing each.

---

## The Rule

**Production PHI must never leave the production environment.** Development and testing use synthetic data only. This is non-negotiable regardless of how urgent a debugging situation feels.

---

## Environment Structure

Three separate GCP projects are required — one per environment. Sharing a project between environments undermines the access isolation that makes this boundary enforceable.

| Environment | GCP Project | Firebase Project | PHI Permitted | BAA Required |
|---|---|---|---|---|
| **Production** (`prod`) | `your-org-prod` | `your-org-prod` | Yes | Yes — signed |
| **Staging** (`staging`) | `your-org-staging` | `your-org-staging` | No — synthetic only | No |
| **Development** (`dev`) | `your-org-dev` | `your-org-dev` | No — synthetic only | No |

Each project has its own:
- Firebase Auth users (test accounts only in dev/staging)
- Firestore database
- Cloud SQL instance
- Cloud Storage buckets
- Service accounts with no cross-environment permissions
- IAM bindings — dev/staging credentials cannot authenticate to production

---

## Why Separate Projects, Not Separate Collections

A common shortcut is to use one Firebase project with environment-specific Firestore collections (`/dev_clients`, `/prod_clients`) or Cloud SQL schemas (`dev`, `prod`). **This does not provide adequate isolation.**

- A single IAM misconfiguration can grant dev access to prod data
- A Cloud Function deployed to the wrong environment can read the wrong database
- Audit logs mix production and non-production access, complicating investigations
- The Google BAA covers the project — a shared project means your dev environment is nominally in BAA scope, which creates compliance documentation complexity

Separate GCP projects eliminate all of these risks.

---

## Synthetic Data Requirements

Development and staging environments must be populated with **synthetic test data** — fabricated records that resemble real data in structure but contain no information about any real person.

**What synthetic data is:**
- Fake names generated from a name list (e.g., Faker.js, Python Faker)
- Random dates of birth in a realistic range
- Placeholder SSNs that follow format but are not real (e.g., `000-XX-XXXX` series reserved by SSA)
- Invented diagnoses and case notes using realistic clinical language but no real patient history

**What synthetic data is not:**
- Anonymized or de-identified real records — even de-identified data carries re-identification risk
- Truncated or masked real records — partial real data is still real data
- Records of staff members or volunteers — even non-PHI PII of real people is not appropriate test data

**Synthetic data generation:** Maintain a seeding script that populates dev/staging databases with a consistent, realistic dataset. Check this script into source control. Run it as part of environment provisioning.

---

## Code Promotion Path

Changes move through environments in one direction only:

```
dev → staging → prod

Developer writes code → deploys to dev
QA validates in staging → Security Officer approves
Production deploy requires sign-off
```

**No hotfixes directly to production without staging validation.** If a production issue is urgent enough to bypass staging, that urgency should be documented, the Security Officer must approve the exception, and the fix must be backported through staging before the next regular deploy.

---

## Environment-Specific Configuration

Cloud Functions, Firestore Security Rules, and application config must never have production credentials hardcoded. Configuration is injected at deploy time via Secret Manager, with separate secrets per environment.

```
Secret Manager — prod project:
  DB_CONNECTION_STRING → prod Cloud SQL private IP
  FIREBASE_PROJECT_ID  → your-org-prod

Secret Manager — staging project:
  DB_CONNECTION_STRING → staging Cloud SQL private IP
  FIREBASE_PROJECT_ID  → your-org-staging
```

**Firebase project ID must be validated at function startup.** If a Cloud Function receives a request and the project ID is not the expected value for that environment, it must refuse to execute. This is a last-resort guard against misconfigured deploys.

```javascript
// At the top of functions/index.js
const EXPECTED_PROJECT = process.env.GCLOUD_PROJECT;
const ALLOWED_PROJECTS = {
  prod:    'your-org-prod',
  staging: 'your-org-staging',
  dev:     'your-org-dev',
};

if (!Object.values(ALLOWED_PROJECTS).includes(EXPECTED_PROJECT)) {
  throw new Error(`Refusing to start: unknown project ${EXPECTED_PROJECT}`);
}
```

---

## Access Controls by Environment

| Access Type | Production | Staging | Development |
|---|---|---|---|
| Developer read access to database | ❌ Never | ✅ Yes | ✅ Yes |
| Developer write access to database | ❌ Never | ✅ Yes (synthetic data only) | ✅ Yes |
| Security Officer read access | ✅ Yes | ✅ Yes | ✅ Yes |
| Automated CI/CD deploy | ✅ With approval gate | ✅ Yes | ✅ Yes |
| Direct console access to Cloud SQL | ❌ Never — Cloud Function only | ✅ For debugging | ✅ Yes |

Production database access is through Cloud Functions only. No developer, regardless of seniority or urgency, connects directly to the production Cloud SQL instance. If a break-glass procedure is needed, it must be documented, approved by the Security Officer, and fully audit-logged.

---

## CI/CD Environment Gates

The deployment pipeline must enforce the environment promotion path:

```yaml
# Example GitHub Actions / Cloud Build gate structure

on_push_to_main:
  - run_tests (Firebase Emulator — dev)
  - deploy_to_staging
  - run_integration_tests (staging)

on_manual_approval (Security Officer):
  - deploy_to_production
  - notify_security_officer
  - write_deployment_audit_log
```

**Firestore Security Rules changes require additional gate:**
- Automated emulator tests must pass against all role/access permutations
- A second developer must review the rule diff
- Security Officer must approve before production deployment

---

## Incident: PHI Found in Non-Production Environment

If PHI from production is discovered in a dev or staging environment:

1. **Immediately** notify the Security Officer
2. **Isolate** the non-production environment — disable access until scope is determined
3. **Apply the four-factor breach assessment** (`docs/09-breach-notification-procedure.md §1.2`) — this may be a reportable breach
4. **Identify the path** — how did production PHI reach the non-production environment?
5. **Document and remediate** — update the risk register; implement controls to prevent recurrence

The presence of real PHI in a non-production environment that lacks production-level safeguards (BAA, audit logging, access controls) is a serious compliance event.

---

## Related Documents

- `docs/08-risk-analysis-template.md` — Environment risks should be included in the risk analysis
- `checklists/pre-launch-audit.md` — Environment separation must be verified before go-live
- `checklists/gcp-configuration.md` — Project-level settings apply per environment
- `docs/11-administrative-policies.md` §7 — Contingency plan covers production environment recovery
