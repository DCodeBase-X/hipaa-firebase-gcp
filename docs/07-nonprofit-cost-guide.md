# Nonprofit Cost Guide

The typical HIPAA architecture guides assume enterprise budgets. This page gives real cost estimates for the Firebase/GCP stack at nonprofit scale — the kind of organization that has 10–150 clients, 5–30 staff, and a technology budget measured in thousands, not hundreds of thousands.

---

## The Bottom Line First

A production-ready, HIPAA-compliant Firebase/GCP system for a small-to-mid nonprofit costs:

| Stage | Monthly Estimate | Who It's For |
|---|---|---|
| **Early (startup)** | $50–$150/mo | < 25 active clients, < 10 staff |
| **Operating** | $150–$400/mo | 25–100 clients, 10–30 staff |
| **Established** | $400–$900/mo | 100–200 clients, 30–75 staff |
| **Scaling** | $900–$2,500/mo | 200+ clients, 75+ staff, multi-location |

These do not include Salesforce, Hostinger, Cloudflare, or email — those are separate line items.

---

## Service-by-Service Breakdown

### Firebase / GCP Core

**Firebase Authentication**
- Free for up to 10,000 monthly active users on the Blaze plan
- Cost: **$0** for virtually all nonprofits

> **Important:** You must be on the **Blaze (pay-as-you-go)** plan to sign the BAA and use HIPAA-eligible services. You cannot use the free Spark plan for HIPAA workloads — even if your usage would be within the free tier limits.

**Cloud Firestore**
- First 1GB storage: free
- Reads: $0.06/100K, Writes: $0.18/100K, Deletes: $0.02/100K
- Estimate for operational nonprofit: **$5–$30/mo**

**Cloud Functions**
- 2M free invocations/month
- $0.40/million after that
- Estimate for most nonprofits: **$0–$10/mo**

**Cloud SQL (PostgreSQL or MySQL)**
This is your primary PHI cost center.

| Instance | vCPU | RAM | Storage | Monthly Cost |
|---|---|---|---|---|
| `db-f1-micro` | shared | 0.6GB | 10GB SSD | ~$8–$15 |
| `db-g1-small` | shared | 1.7GB | 20GB SSD | ~$25–$40 |
| `db-custom-1-3840` | 1 | 3.75GB | 50GB SSD | ~$60–$90 |
| `db-custom-2-7680` | 2 | 7.5GB | 100GB SSD | ~$120–$175 |

**Recommendation by stage:**
- Startup: `db-f1-micro` or `db-g1-small`
- Operating: `db-g1-small` or `db-custom-1-3840`
- Established: `db-custom-1-3840` + HA configuration (~$180–$250/mo)

**Cloud Storage**
- First 5GB: free
- $0.026/GB/month after that
- For document storage (scanned forms, assessments): typically **$5–$25/mo**

**Cloud Logging / Cloud Audit Logs**
- First 50GB/month: free
- $0.01/GB after
- Estimate for audit logging: **$0–$10/mo**

**VPC Connector (for Cloud SQL private access)**
- ~$0.01/GB data processed
- Estimate: **$5–$20/mo**

---

### Full Monthly Estimate by Stage

**Early Stage (~25 clients, ~10 staff)**
| Service | Monthly Cost |
|---|---|
| Firebase Auth (Blaze) | $0 |
| Firestore | $5 |
| Cloud Functions | $0 |
| Cloud SQL (db-g1-small) | $30 |
| Cloud Storage (10GB) | $5 |
| VPC Connector | $5 |
| Cloud Logging | $0 |
| **Total GCP/Firebase** | **~$45–$65/mo** |

**Operating Stage (~75 clients, ~20 staff)**
| Service | Monthly Cost |
|---|---|
| Firebase Auth | $0 |
| Firestore | $20 |
| Cloud Functions | $5 |
| Cloud SQL (db-custom-1-3840) | $75 |
| Cloud Storage (50GB) | $15 |
| VPC Connector | $12 |
| Cloud Logging | $5 |
| **Total GCP/Firebase** | **~$130–$175/mo** |

**Established Stage (~150 clients, ~40 staff, HA)**
| Service | Monthly Cost |
|---|---|
| Firebase Auth | $0 |
| Firestore | $45 |
| Cloud Functions | $15 |
| Cloud SQL (2 vCPU + HA) | $250 |
| Cloud Storage (150GB) | $40 |
| VPC Connector | $20 |
| Cloud Logging | $15 |
| **Total GCP/Firebase** | **~$385–$450/mo** |

---

### Full Stack Monthly Budget (Operating Stage)

Including all vendors for a complete picture:

| Vendor | Service | Monthly |
|---|---|---|
| Google (Firebase/GCP) | Core infrastructure | $130–$175 |
| Hostinger | WordPress hosting | $15–$30 |
| Cloudflare | DNS, WAF, SSL | $0–$20 (Free or Pro) |
| Salesforce NPSP | Nonprofit CRM | $0 (NPSP is free) or $75+ (Sales Cloud) |
| Email (Google Workspace) | Org email | $12–$18/user |
| Donorbox | Donation processing | $0 + 1.5–2% transaction fee |
| **Total** | | **~$200–$350/mo + email** |

---

## Cost Reduction Strategies for Nonprofits

**1. Google for Nonprofits**
Eligible nonprofits receive $20,000/year in Google Cloud credits. Apply at [google.com/nonprofits](https://www.google.com/nonprofits/). This alone covers most early-stage infrastructure costs.

**2. Salesforce NPSP is free**
Salesforce donates 10 licenses of NPSP to eligible 501(c)(3)s through the Power of Us program. This is a $25,000+/year value. Apply at [salesforce.org/power-of-us](https://www.salesforce.org/power-of-us/).

**3. Firebase free tier counts on Blaze**
The Blaze plan still includes the same free tiers as Spark — you're only billed for usage beyond those limits. Being on Blaze (required for HIPAA BAA) doesn't mean you lose the free tier.

**4. Start small on Cloud SQL**
`db-f1-micro` is genuinely sufficient for a startup nonprofit. Upgrade as you actually hit performance limits rather than pre-provisioning for theoretical scale.

**5. Cloudflare Free covers the basics**
Cloudflare's free plan includes DDoS protection, CDN, and SSL. The paid plans add WAF rules and the option to sign a BAA. Start free and upgrade when the WAF becomes important.

---

## Where Costs Escalate

Be aware of these cost drivers before you hit them:

| Driver | Impact | Mitigation |
|---|---|---|
| Cloud SQL HA (High Availability) | 2× instance cost | Enable HA only when uptime SLA matters |
| Large document storage | $0.026/GB ongoing | Lifecycle policies to archive old files to Coldline ($0.004/GB) |
| Cloud Function egress | $0.08–$0.12/GB | Minimize large data transfers; paginate queries |
| Cloud Logging overages | $0.01/GB | Exclude high-volume non-audit logs from long-term storage |
| Firestore read-heavy workloads | Can spike | Cache frequent reads; use subcollections wisely |

---

**Next:** [Firebase Configuration Checklist →](../checklists/firebase-configuration.md)
