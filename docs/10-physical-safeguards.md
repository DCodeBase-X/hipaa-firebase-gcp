# HIPAA Physical Safeguards

> **Reference**: 45 CFR § 164.310 — Physical Safeguards
> These are required safeguards for all Covered Entities and Business Associates under the HIPAA Security Rule.

Physical safeguards govern access to the physical locations and devices that contain or access ePHI. Even when your ePHI lives in the cloud, staff workstations, mobile devices, and office facilities remain within scope.

---

## Section 1: Facility Access Controls

**Standard**: Implement policies and procedures to limit physical access to electronic information systems and the facilities in which they are housed.

### 1.1 Facility Security Plan

- [ ] Identify all physical locations where ePHI is accessed (offices, home workstations, remote sites)
- [ ] Implement access controls for each location (key cards, locks, visitor log)
- [ ] Restrict server rooms and networking equipment to authorized personnel only
- [ ] Ensure areas where ePHI is displayed on screens are not visible to unauthorized visitors
- [ ] Post "No Unauthorized Access" signage on restricted areas

### 1.2 Visitor Access

- [ ] Maintain a visitor log for all facility locations
- [ ] Escort all visitors in areas where ePHI may be accessed or visible
- [ ] Issue temporary visitor badges that are returned upon departure
- [ ] Brief visitors on PHI confidentiality expectations

### 1.3 Workstation Placement

- [ ] Position workstation screens so they are not visible through windows or from common areas
- [ ] Use privacy screens on monitors in open office environments
- [ ] Ensure no PHI is printed and left unattended
- [ ] Shred all physical documents containing PHI immediately after use

---

## Section 2: Workstation Use Policy

**Standard**: Implement policies specifying the proper functions of workstations and how workstations should be used and positioned.

### 2.1 Authorized Use

All staff who access ePHI via workstation must adhere to the following:

- Use only organization-approved devices to access ePHI
- Never use personal (non-managed) devices to access ePHI unless explicitly authorized and enrolled in MDM
- Never access ePHI from public computers (libraries, internet cafes, shared kiosks)
- Never allow family members or others to use a work device
- Lock the workstation immediately when leaving it unattended (Windows: Win+L, Mac: Ctrl+Cmd+Q)

### 2.2 Automatic Screen Lock

All workstations and mobile devices must be configured with:

| Setting | Required Value |
|---------|---------------|
| Screen lock timeout | 5 minutes or less of inactivity |
| Password/PIN required to unlock | Yes |
| Password complexity | Minimum 12 characters, mixed case + numbers |
| Biometric unlock (optional) | Permitted as supplement, not sole factor |

### 2.3 Remote Work

Staff accessing ePHI from home or remote locations must:

- [ ] Use a VPN if connecting to any on-premises resources
- [ ] Use only an organization-managed device
- [ ] Ensure no unauthorized individuals can view the screen
- [ ] Use a password-protected home Wi-Fi network (WPA2 or WPA3)
- [ ] Never use public Wi-Fi without a VPN

---

## Section 3: Workstation Security (Technical)

**Standard**: Implement physical safeguards for all workstations that access ePHI.

### 3.1 Device Encryption

All devices that access ePHI must have full-disk encryption enabled:

| Device Type | Required Encryption | Verification Method |
|-------------|--------------------|--------------------|
| Mac laptops/desktops | FileVault 2 enabled | System Settings → Privacy & Security → FileVault |
| Windows laptops/desktops | BitLocker enabled | Control Panel → BitLocker Drive Encryption |
| iOS devices | Enabled by default when passcode is set | Settings → Touch/Face ID & Passcode |
| Android devices | Enabled by default (Android 6+) | Settings → Security → Encryption |
| External drives used for ePHI | Encrypted (VeraCrypt or native OS encryption) | Verify before use |

- [ ] Audit encryption status on all devices quarterly
- [ ] Document which devices are authorized to access ePHI

### 3.2 Mobile Device Management (MDM)

For organizations with 5+ staff devices accessing ePHI, implement MDM:

**Recommended MDM options for nonprofits**:
- **Jamf** (Apple devices) — discounted nonprofit pricing
- **Microsoft Intune** — included with Microsoft 365 Business Premium
- **Google Workspace MDM** — basic, included with Workspace

Minimum MDM policy requirements:
- [ ] Remote wipe capability
- [ ] Enforce screen lock and encryption
- [ ] Block installation of unauthorized apps
- [ ] Report device compliance status
- [ ] Prevent ePHI from being copied to personal cloud storage (iCloud personal, Dropbox, etc.)

### 3.3 Antivirus and Patching

- [ ] Install reputable endpoint protection on all workstations (macOS, Windows)
- [ ] Enable automatic OS updates; require installation within 14 days of release
- [ ] Enable automatic app updates for all ePHI-adjacent applications
- [ ] Run monthly vulnerability scans on all managed devices

---

## Section 4: Device and Media Controls

**Standard**: Implement policies governing receipt, removal, and disposal of hardware and electronic media that contain ePHI.

### 4.1 Device Inventory

Maintain a current inventory of all devices that access ePHI:

| Asset ID | Device Type | Make/Model | Serial Number | Assigned User | Encryption Status | MDM Enrolled | Notes |
|----------|-------------|------------|--------------|---------------|------------------|--------------|-------|
| | | | | | | | |

- [ ] Update inventory when devices are added, reassigned, or retired
- [ ] Review inventory quarterly

### 4.2 Device Acquisition

When receiving a new device:
- [ ] Enroll in MDM before granting access to ePHI
- [ ] Enable full-disk encryption before first use
- [ ] Apply all available security updates
- [ ] Document in asset inventory
- [ ] Brief assigned user on device security policies

### 4.3 Device Transfer and Reassignment

Before reassigning a device to another user:
- [ ] Perform a full data wipe (factory reset or secure erase)
- [ ] Verify wipe completed successfully before reassignment
- [ ] Update asset inventory with new assignee
- [ ] Remove previous user's credentials from all accounts

### 4.4 Device Disposal and Media Sanitization

Before disposing of any device that has ever had access to ePHI:

| Media Type | Required Sanitization Method |
|------------|----------------------------|
| SSD / Flash storage | Cryptographic erase (preferred) or manufacturer secure erase tool |
| HDD (spinning disk) | DoD 5220.22-M 3-pass wipe OR physical destruction |
| Optical media (CD/DVD) | Physical destruction (shred or degauss) |
| Mobile phones/tablets | Factory reset + remove SIM/SD card |
| USB drives | Secure erase + physical destruction if disposing |
| Paper records | Cross-cut shredder (DIN 66399 Level P-4 or higher) |

- [ ] Document all device disposals with method used, date, and responsible party
- [ ] Retain disposal records for 6 years

### 4.5 Lost or Stolen Devices

If a device with access to ePHI is lost or stolen:

1. **Immediately** notify the Security Officer
2. **Within 1 hour**: Remote wipe via MDM if enrolled
3. **Within 24 hours**: Revoke all credentials and sessions for that device
4. **Assess**: Does this constitute a HIPAA breach? Apply the four-factor test (`docs/09-breach-notification-procedure.md`)
5. **Document**: Log the incident in the breach incident log
6. Report to law enforcement if theft is suspected

**MDM Remote Wipe Procedure**:
- [MDM console URL]: _______________________________________________
- Authorized to initiate wipe: _______________________________________________ (role)
- Confirm wipe completed: verify in MDM console and document

---

## Section 5: Physical Safeguard Policy Acknowledgment

All workforce members with access to ePHI or ePHI-capable devices must sign this acknowledgment annually.

---

I, _______________________________________________, acknowledge that I have read, understood, and will comply with the organization's Physical Safeguards policies as of the date signed below.

I understand that:
- My assigned device must be encrypted and MDM-enrolled
- I must lock my screen when leaving my workstation unattended
- I must not allow unauthorized individuals to use my work device or view ePHI on my screen
- Lost or stolen devices must be reported to the Security Officer immediately
- Violations of these policies may result in disciplinary action up to and including termination

Signature: _______________________________________________ Date: _______________________________________________

Name (printed): _______________________________________________ Title: _______________________________________________

---

## Related Documents

- `docs/11-administrative-policies.md` — Workforce sanctions and access management
- `docs/12-staff-training-program.md` — Training requirements
- `checklists/incident-response-runbook.md` — Lost/stolen device response
- `docs/08-risk-analysis-template.md` — Physical risks in the risk analysis
