# Disaster Recovery Runbook

> **Reference**: 45 CFR § 164.308(a)(7) — Contingency Plan
> **See also**: `docs/11-administrative-policies.md` §7 for RTO/RPO targets and policy context.

This runbook provides executable step-by-step procedures for each recovery scenario. It is not a policy document. It is the procedure you run when something breaks. Keep it accessible offline (printed or local copy) in case the system you are recovering is the one you would normally use to read it.

**Before every recovery procedure:** Export relevant Cloud Audit Logs before they age out. Default Data Access log retention is 30 days. If the incident is older than that, logs may already be gone.

---

## Emergency Contacts

| Role | Name | Phone | Email |
|------|------|-------|-------|
| Security Officer | | | |
| Lead Developer / IT | | | |
| Executive Director | | | |
| GCP Support | N/A | console.cloud.google.com/support | |

**GCP Project IDs:**

| Environment | Project ID |
|---|---|
| Production | |
| Staging | |

---

## Scenario 1: Cloud SQL Instance Failure (HA Failover)

**When:** Cloud SQL primary instance becomes unavailable. HA replica takes over automatically.

**RTO target:** < 5 minutes | **RPO target:** < 5 minutes

### Steps

1. **Verify the failover occurred**
   ```
   GCP Console → SQL → [instance name]
   Status should show "Runnable" with the standby now acting as primary
   ```

2. **Confirm Cloud Functions can connect**
   - Check Cloud Functions error rate in Cloud Monitoring
   - If connection errors persist > 2 minutes after failover, proceed to manual steps

3. **If automatic failover did not occur, initiate manual failover**
   ```bash
   gcloud sql instances failover INSTANCE_NAME --project=PROJECT_ID
   ```

4. **Verify application health**
   - Submit a test transaction through the application
   - Confirm the record appears in Cloud SQL
   - Check Cloud Audit Logs for the write event

5. **Document the incident**
   - Log in the incident register: `checklists/incident-response-runbook.md`
   - Note: HA failover is not a breach — document it as an infrastructure event

---

## Scenario 2: Cloud SQL Point-in-Time Recovery (PITR)

**When:** Data corruption, accidental deletion, or a bad migration requires restoring the database to a specific moment before the damage.

**RTO target:** < 4 hours | **RPO target:** < 1 hour

### Before You Start

- Confirm the target restore time (the moment *before* the corruption/deletion occurred)
- PITR restores to a **new instance** — the original instance continues running until you cut over
- You will need to update Cloud Function connection strings after the restore

### Steps

1. **Identify the target restore time**
   - Review audit logs to find the exact timestamp of the damaging operation
   - Add a 1–2 minute buffer before that timestamp as your restore target
   ```bash
   gcloud logging read \
     'resource.type="cloudsql_database" AND protoPayload.methodName="cloudsql.instances.query"' \
     --project=PROJECT_ID \
     --format=json \
     --freshness=72h | head -100
   ```

2. **Initiate PITR restore via Cloud Console**
   ```
   GCP Console → SQL → [instance name] → Backups
   Click "Restore" → Choose "Point-in-time recovery"
   Target time: [timestamp from step 1]
   Target instance: create a NEW instance (e.g., prod-db-restored-YYYYMMDD)
   Region: same as original
   ```
   Or via CLI:
   ```bash
   gcloud sql instances clone SOURCE_INSTANCE RESTORED_INSTANCE \
     --point-in-time="YYYY-MM-DDTHH:MM:SSZ" \
     --project=PROJECT_ID
   ```

3. **Wait for restore to complete** (typically 15–45 minutes for moderate database sizes)
   - Monitor progress: `GCP Console → SQL → [restored instance] → Overview`

4. **Verify data integrity on the restored instance**
   ```bash
   # Connect via Cloud SQL Auth Proxy to the restored instance
   cloud_sql_proxy -instances=PROJECT_ID:REGION:RESTORED_INSTANCE=tcp:5433

   # In a separate terminal
   psql -h 127.0.0.1 -p 5433 -U db_user -d db_name

   # Verify record counts match expectations
   SELECT COUNT(*) FROM clients;
   SELECT COUNT(*) FROM health_records;
   SELECT MAX(created_at) FROM health_records; -- confirm last record is before the damage
   ```

5. **Configure the restored instance**
   - Enable private IP on the restored instance (same VPC as original)
   - Disable public IP
   - Enable SSL (`require_ssl = on`)
   - Apply same database flags as the original instance

6. **Update Cloud Function connection strings**
   - Update the `DB_CONNECTION_STRING` secret in Secret Manager to point to the restored instance
   ```
   GCP Console → Secret Manager → DB_CONNECTION_STRING → New version
   Value: [private IP of restored instance]
   ```
   - Redeploy Cloud Functions to pick up the new secret version:
   ```bash
   firebase deploy --only functions
   ```

7. **Verify application connectivity**
   - Run a full access test for each PHI-touching Cloud Function
   - Confirm audit logs are flowing from the restored instance

8. **Decommission the damaged original instance**
   - Only after step 7 is confirmed healthy
   - Do not delete — disable the instance first and retain for 7 days before deletion
   ```bash
   gcloud sql instances patch ORIGINAL_INSTANCE --activation-policy=NEVER
   ```

9. **Document**
   - Record restore point used, start time, completion time, and data loss scope
   - Update the risk register
   - Initiate breach determination process if PHI was exposed during the damage window

---

## Scenario 3: Full Backup Restore (Region Outage or Instance Loss)

**When:** Cloud SQL instance is unrecoverable — PITR is unavailable or a full region is down.

**RTO target:** < 4 hours | **RPO target:** < 24 hours (last daily backup)

### Steps

1. **Identify the most recent valid backup**
   ```
   GCP Console → SQL → [instance name] → Backups
   Select the most recent backup with status "Successful" taken before the incident
   ```

2. **Restore to a new instance in an available region**
   ```bash
   gcloud sql instances create RESTORED_INSTANCE \
     --database-version=POSTGRES_14 \
     --region=us-east1 \
     --tier=db-custom-1-3840 \
     --no-assign-ip \
     --network=projects/PROJECT_ID/global/networks/VPC_NAME

   gcloud sql instances restore-backup RESTORED_INSTANCE \
     --backup-instance=ORIGINAL_INSTANCE \
     --backup-id=BACKUP_ID \
     --project=PROJECT_ID
   ```

3. **Reconfigure the instance** — repeat steps 5–8 from Scenario 2

4. **Update VPC Connector** if restoring to a different region
   - Create a new VPC Connector in the same region as the restored instance
   - Update Cloud Function VPC Connector configuration
   - Redeploy functions

5. **Communicate status to stakeholders**
   - Notify the Executive Director of the outage duration and data loss scope
   - If PHI was inaccessible for > 4 hours, review whether client services were impacted and document

---

## Scenario 4: Firestore Data Loss or Corruption

**When:** Firestore operational data is lost or corrupted.

**RTO target:** < 4 hours | **RPO target:** < 24 hours

### Steps

1. **Identify scope** — which collections are affected and what is the damage timestamp

2. **Restore from Firestore managed export**
   ```
   GCP Console → Firestore → Import/Export → Import
   Select the most recent export from Cloud Storage
   Bucket: gs://[project-id]-firestore-backups/
   ```
   Or via CLI:
   ```bash
   gcloud firestore import \
     gs://BUCKET_NAME/EXPORT_FOLDER \
     --collection-ids=COLLECTION_NAME \
     --project=PROJECT_ID
   ```
   > **Note:** Firestore import merges data — it does not wipe existing documents first. If you need a clean restore, contact GCP Support for managed restoration assistance.

3. **Verify collection document counts and spot-check records**

4. **Document the recovery**

---

## Scenario 5: Firebase Auth Outage

**When:** Firebase Auth is unavailable — staff cannot log in.

**RTO target:** N/A (vendor-managed) | **RPO target:** N/A

Firebase Auth availability is Google's responsibility under the BAA. Your organization cannot self-recover from a Firebase Auth outage — this is a vendor incident.

### Steps

1. **Confirm it is a Firebase outage**, not a local misconfiguration
   - Check Firebase Status Page: status.firebase.google.com
   - Check GCP Status: status.cloud.google.com

2. **Activate emergency mode operations** per `docs/11-administrative-policies.md` §7.3
   - Document manual procedures for critical client care decisions
   - Identify which workforce members have authority to access backup systems
   - Ensure paper-based emergency records are available for critical client information

3. **Communicate status** — notify staff that login is unavailable and provide an estimated resolution time from the Firebase Status Page

4. **When service is restored** — verify all staff can authenticate; confirm no sessions were invalidated unexpectedly

5. **Document the outage** — log the start time, end time, and any client service impact

---

## Scenario 6: Cloud Storage PHI Documents Inaccessible or Deleted

**When:** PHI documents in Cloud Storage are accidentally deleted, corrupted, or access is blocked.

### Recovering Deleted Objects (Soft Delete Enabled)

```bash
# List deleted objects in the recovery window
gsutil ls -l -a gs://BUCKET_NAME/PATH/ | grep "#"

# Restore a specific object version
gsutil cp gs://BUCKET_NAME/OBJECT#VERSION_ID gs://BUCKET_NAME/OBJECT
```

### Recovering from Object Versioning

```bash
# List all versions of a specific object
gsutil ls -la gs://BUCKET_NAME/path/to/object

# Restore a prior version
gsutil cp gs://BUCKET_NAME/path/to/object#GENERATION_NUMBER gs://BUCKET_NAME/path/to/object
```

### If Bucket Permissions Were Accidentally Changed

```bash
# Restore uniform bucket-level access
gsutil uniformbucketlevelaccess set on gs://BUCKET_NAME

# Remove any accidental public access
gsutil iam ch -d allUsers gs://BUCKET_NAME
gsutil iam ch -d allAuthenticatedUsers gs://BUCKET_NAME
```

---

## Annual DR Test Procedure

HIPAA requires testing disaster recovery procedures at least annually. Document the test here.

### Test Scope (minimum)
- [ ] PITR restore to staging environment from most recent backup
- [ ] Verify data integrity using record count and spot-check queries
- [ ] Time the full restore process — compare against RTO target
- [ ] Verify Cloud Functions connect to the restored instance
- [ ] Verify audit logs flow correctly from the restored instance
- [ ] Firestore import test on staging
- [ ] Confirm emergency contact list is current

### Test Log

| Test Date | Scenario Tested | RTO Achieved | Data Loss (RPO) | Gaps Found | Tester |
|---|---|---|---|---|---|
| | PITR restore | | | | |
| | Full backup restore | | | | |
| | Firestore import | | | | |

**Annual test signed off by:** _______________________________________________ Date: _______________________________________________

---

## Related Documents

- `docs/11-administrative-policies.md` §7 — Contingency plan policy and RTO/RPO targets
- `checklists/incident-response-runbook.md` — Incident classification and breach determination
- `checklists/gcp-configuration.md` — Backup configuration verification
- `docs/08-risk-analysis-template.md` — DR risks in the risk analysis
