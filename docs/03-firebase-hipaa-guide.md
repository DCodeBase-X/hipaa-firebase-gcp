# Firebase HIPAA Guide

Firebase is Google's developer-facing platform. It's distinct from GCP in how it's marketed and how developers use it — but from a HIPAA perspective, what matters is which Firebase services are covered under Google's BAA and which are not.

This page covers what's covered, the critical gotchas, and the specific configuration required for each service.

---

## What Is and Isn't Covered

### Covered Under Google's BAA ✅

These Firebase services can handle PHI — **after you sign the BAA and configure them correctly:**

| Service | PHI Allowed | Key Requirements |
|---|---|---|
| **Cloud Firestore** | ✅ | Enable audit logging; use server-side security rules |
| **Firebase Authentication** | ⚠️ Limited | Identity only — no clinical data in user profiles or custom claims |
| **Cloud Functions for Firebase** | ✅ | Don't log PHI; use VPC connector for Cloud SQL access |
| **Cloud Storage for Firebase** | ✅ | Enable uniform bucket-level access; no public buckets |
| **Cloud SQL** (via Firebase Admin) | ✅ | Private IP only; enable CMEK; audit logging on |

### NOT Covered Under Google's BAA ❌

These Firebase services **cannot touch PHI**. If PHI is passing through them, you are out of compliance:

| Service | Why It's Not Covered |
|---|---|
| **Firebase Realtime Database** | Explicitly excluded from Google's BAA |
| **Firebase Hosting** | Not in BAA — for public content only |
| **Firebase Analytics / Google Analytics** | Data goes to Google's analytics infrastructure, not BAA-covered |
| **Crashlytics** | Crash reports may contain device state — not covered |
| **Firebase Performance Monitoring** | Same as Crashlytics |
| **Firebase Remote Config** | Config values could leak PHI in logs |
| **Firebase A/B Testing** | Analytics-based, not covered |

### The Critical Gotcha: Realtime Database vs. Firestore

This is the mistake teams make most often. Firebase launched with Realtime Database. Many tutorials, older codebases, and Firebase quickstarts use it. **Realtime Database is not in Google's BAA.** Firestore — which launched later and is the current recommended database — is.

If you inherited a Firebase codebase or are following older tutorials, check which database you're using before storing any PHI.

```javascript
// REALTIME DATABASE — Not HIPAA covered ❌
import { getDatabase, ref, set } from "firebase/database";
const db = getDatabase();
set(ref(db, 'clients/' + clientId), clientData);

// FIRESTORE — HIPAA covered ✅ (with correct config)
import { getFirestore, doc, setDoc } from "firebase/firestore";
const db = getFirestore();
await setDoc(doc(db, "clients", clientId), clientData);
```

---

## Signing the Google BAA

Before storing any PHI:

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Navigate to **IAM & Admin → Settings**
3. Scroll to **"HIPAA Business Associate Amendment"**
4. Review and accept

This covers all Firebase and GCP services that appear on [Google's BAA-eligible services list](https://cloud.google.com/security/compliance/hipaa-compliance). The list is updated periodically — bookmark it and review it when you add new services.

---

## Firestore Configuration

### Security Rules

Firestore Security Rules are your access control layer. The default rules for new projects are either fully open or fully closed depending on when the project was created. Neither is right for PHI.

**Start with deny-all:**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Deny everything by default
    match /{document=**} {
      allow read, write: if false;
    }

    // Staff can read/write client records they are assigned to
    match /clients/{clientId} {
      allow read, write: if request.auth != null
        && request.auth.token.role in ['admin', 'clinical_staff']
        && (request.auth.token.role == 'admin'
            || resource.data.assignedStaffId == request.auth.uid);
    }

    // Admins only for audit logs
    match /audit_logs/{logId} {
      allow read: if request.auth != null
        && request.auth.token.role == 'admin';
      allow write: if false; // Written by Cloud Functions only
    }

    // Clients can read their own records
    match /client_portal/{clientId} {
      allow read: if request.auth != null
        && request.auth.uid == clientId;
      allow write: if false;
    }
  }
}
```

### Audit Logging

Enable Cloud Audit Logs for Firestore:

1. Cloud Console → **IAM & Admin → Audit Logs**
2. Find **Cloud Datastore API** (Firestore's underlying API)
3. Enable: **Admin Read, Data Read, Data Write**

Every PHI access is now logged to Cloud Logging with timestamp, user identity, and operation.

### Data Location

Set Firestore's data location to a US multi-region at project creation — this cannot be changed later:
- `us-central` or `us-east1` for single region
- `nam5` (US multi-region) for higher availability

---

## Firebase Authentication Configuration

Firebase Auth is covered under the BAA for identity purposes. What you can store:
- UID (generated by Firebase)
- Email address
- Display name
- Custom claims (roles, permissions)

What you **cannot** store in Firebase Auth profiles or custom claims:
- Diagnosis codes, health status, medications
- Criminal justice record details
- Disability status, mental health history
- Any PHI field

Custom claims are the right place for RBAC roles. They're included in the ID token and available in Firestore Security Rules:

```javascript
// Cloud Function to set role on user creation (server-side only)
exports.setUserRole = functions.https.onCall(async (data, context) => {
  // Verify caller is admin
  if (!context.auth?.token?.role === 'admin') {
    throw new functions.https.HttpsError('permission-denied', 'Admin only');
  }

  const { uid, role } = data;
  const validRoles = ['admin', 'clinical_staff', 'program_staff', 'volunteer'];

  if (!validRoles.includes(role)) {
    throw new functions.https.HttpsError('invalid-argument', 'Invalid role');
  }

  await admin.auth().setCustomUserClaims(uid, { role });
  return { success: true };
});
```

**Enforce MFA for staff roles.** Volunteers and lower-access roles can optionally require MFA. Admin and clinical staff should require it:

```javascript
// In your auth flow, check MFA enrollment for clinical roles
const user = auth.currentUser;
if (userRole === 'clinical_staff' && !user.multiFactor.enrolledFactors.length) {
  // Redirect to MFA enrollment before allowing access
  router.push('/enroll-mfa');
}
```

---

## Cloud Functions for Firebase

Cloud Functions are the correct place for all PHI business logic. The client app should never query PHI databases directly.

**The pattern:**
```
Client App
    │
    ▼ (HTTPS callable, authenticated)
Cloud Function
    │
    ▼ (Private VPC — never public)
Cloud SQL / Firestore PHI collections
```

**Critical rules for Cloud Functions handling PHI:**

**1. Never log PHI**
```javascript
// ❌ Wrong — PHI appears in Cloud Logging
console.log('Processing client:', clientData);

// ✅ Correct — log the operation, not the data
console.log('Processing record for clientId:', clientId, 'operation: intake_update');
```

**2. Validate auth and role before any PHI access**
```javascript
exports.getClientRecord = functions.https.onCall(async (data, context) => {
  // Always verify auth first
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Login required');
  }

  const role = context.auth.token.role;
  if (!['admin', 'clinical_staff'].includes(role)) {
    throw new functions.https.HttpsError('permission-denied', 'Insufficient access');
  }

  // Additional check: clinical staff can only access assigned clients
  if (role === 'clinical_staff') {
    const assignment = await admin.firestore()
      .collection('assignments')
      .where('staffId', '==', context.auth.uid)
      .where('clientId', '==', data.clientId)
      .get();

    if (assignment.empty) {
      throw new functions.https.HttpsError('permission-denied', 'Not assigned to this client');
    }
  }

  // Now safe to fetch PHI
  const record = await getClientFromSQL(data.clientId);
  return record;
});
```

**3. Use VPC connector for Cloud SQL access**

Never expose Cloud SQL to the public internet. Use a VPC connector so Cloud Functions access the database over a private network:

```
Cloud Function → VPC Connector → Private IP → Cloud SQL
```

Configure in `firebase.json` or Cloud Console:
- Create a VPC connector in the same region as your Cloud SQL instance
- Attach it to your Cloud Function deployment
- Set Cloud SQL to private IP only (disable public IP)

---

**Next:** [Multi-Vendor Boundaries →](04-multi-vendor-boundaries.md)
