# PCI DSS Scope Determination: Meridian Pay

## Document Information

| Field | Value |
|-------|-------|
| Organization | Sierra Meridian Field Services, Inc. (fictional) |
| Assessment Type | Internal pre-assessment readiness review (not a QSA assessment) |
| Assessment Date | October 1, 2026 |
| Assessor | Faith Olofintuyi, Risk & Compliance |
| Standard | PCI DSS v4.0.1 |
| QSA Assessment Scheduled | Q1 2027 |
| Version | 1.0 |

Supporting work: [scenario](../scenario.md), [card data flow map](card-data-flow.md), [trust relationship matrix](trust-relationship-matrix.md), [segmentation gap analysis](segmentation-gap-analysis.md).

---

## 1. Executive Summary

### Scope Statement

Meridian Pay's design document describes a small, walled-off card environment of five servers. That description is not accurate today. Card numbers reach AR staff, their laptops, the VPN, the email system, and an outside phone vendor, and the card servers share logins, backups, updates, and an unencrypted data link with the corporate network. Because segmentation has never been tested and the card servers trust the corporate login system, **the entire corporate network must be treated as in scope today**. With the changes in Section 8, scope can be reduced to a defined set of about a dozen systems before the QSA assessment.

### Two Scopes

| | Current State (today) | Target State (after Section 8 fixes) |
|---|---|---|
| Card environment | Documented CDE plus AR laptops, VPN, email, phone vendor, backups | Documented CDE plus backups |
| Corporate network | **In scope** (segmentation unproven; shared logins) | **Out of scope** (separate CDE logins; tested segmentation) |
| Ordinary employee laptops (about 1,500) | **In scope** | **Out of scope** |
| InvoiceTrack | In scope (connected, receives payment details) | Out of scope (no direct connection) |

### Key Findings
- [x] Employees handle full card numbers by phone and email (F1)
- [x] Card servers share core services with the corporate network (F2)
- [x] InvoiceTrack links the card environment to a system outsiders can reach (F3)
- [x] Call recordings and email keep security codes, which must never be stored (F6, F7)
- [x] Segmentation has never been tested (F8)

### Overall Risk Rating
[ ] Low | [ ] Medium | [x] High | [ ] Critical

High rather than Critical because there is no evidence of compromise and the assessment is still months away. Two findings (F6, F7) are individually Critical and are the first priority.

---

## 2. Cardholder Data Environment Definition

### 2.1 Systems That STORE Cardholder Data

| System | Data Stored | Retention | Justification |
|--------|-------------|-----------|---------------|
| Card database (CDE) | Card number, name, expiry (cards on file) | Life of service contract | Recurring billing |
| Backup server and backup storage (management zone) | Copies of the card database | Backup schedule | Backups hold the same data |
| Cloud phone system (vendor) | Call recordings and transcripts with card numbers and **security codes** | Unknown | Not intended; see F6 |
| AR shared inbox, archive, and email backups | Emails with card numbers and sometimes **security codes** | Unknown; "usually deleted" | Not intended; see F7 |
| SIEM (management zone) | Possibly card numbers in logs | Log retention period | Not yet checked; see F4 |

### 2.2 Systems That PROCESS Cardholder Data

| System | Function | Data Elements | Justification |
|--------|----------|---------------|---------------|
| Payment application (CDE) | Authorizes payments, manages cards on file | Card number, expiry, security code (in transit only) | Core payment function |
| Tokenization service and HSM (CDE) | Replaces card numbers with tokens; manages encryption keys | Card number | Protects stored data |
| AR laptops (about 25, assumption) | Staff key card numbers into the portal | Card number, expiry, security code | Phone and email payments |

### 2.3 Systems That TRANSMIT Cardholder Data

| System | Path | Encryption | Justification |
|--------|------|------------|---------------|
| Meridian Pay web servers and load balancer (DMZ) | Customer or AR laptop to payment application | Yes (HTTPS) | Payment intake |
| VPN gateway (corporate) | AR laptop to company network | Yes (VPN) | Remote staff payments |
| Connection to InvoiceTrack | Payment application to InvoiceTrack | **No** | Marks invoices paid; see F3 |
| Payment processor connection | Payment application to processor | Yes | Authorization |

---

## 3. Connected Systems Assessment

### 3.1 Direct Connections to CDE

| System | Connection | Purpose | In Scope? | Justification |
|--------|-----------|---------|-----------|---------------|
| InvoiceTrack | Payment app sends payment details | Mark invoices paid | Yes | Direct, unencrypted connection; reachable by outside partners |
| Active Directory | CDE servers accept AD logins | Authentication | Yes | Controls access to every CDE server |
| Jump host | Admin connections | Administration | Yes | Gateway for all admin access |
| Patch server | Pushes updates | Patching | Yes | Can install software on CDE servers |
| Backup server | Receives copies | Backup | Yes | Stores card data |
| SIEM | Receives logs | Monitoring | Yes | Security function for the CDE |
| DNS and time servers (assumption: corporate) | Lookups and clock sync | Infrastructure | Yes | Could redirect traffic or corrupt logs |

### 3.2 Indirect Connections (Via Intermediary)

| System | Path | Purpose | In Scope? | Justification |
|--------|------|---------|-----------|---------------|
| All corporate workstations | Same network and login system as CDE-connected services | General business | **Yes, today** | Segmentation untested; any compromised laptop could attack Active Directory and, through it, the CDE |
| Admin workstations | Connect to jump host | Administration | Yes | Can reach the CDE through the jump host |
| External partner accounts | Reach InvoiceTrack folders | Joint field work | Yes, today | One step from a system connected to the CDE |

---

## 4. Security-Impacting Systems

| System | Security Function | Impact if Compromised | In Scope? | Justification |
|--------|-------------------|----------------------|-----------|---------------|
| Active Directory | Logins for CDE servers | Full access to card data | Yes | Highest-impact shared service |
| Jump host | Admin gateway | Full admin access to CDE | Yes | Single point of entry |
| Patch server | Software updates | Malicious software on CDE | Yes | Can change CDE systems |
| SIEM | Detection | Attacks go unnoticed | Yes | Watches the CDE |
| VPN gateway | Remote access | Access to AR laptops' traffic and the network | Yes | Card data passes through it |
| DNS and time servers | Name lookups and clocks | Redirected traffic; unreliable logs | Yes | Lower risk, still in scope |

---

## 5. Trust Relationship Analysis

Summarized from the [trust relationship matrix](trust-relationship-matrix.md).

### 5.1 Authentication Dependencies

| CDE System | Authentication Source | Risk if Compromised | Mitigation |
|------------|----------------------|---------------------|------------|
| All CDE servers | Corporate Active Directory | Any stolen IT account could reach card data | Separate CDE accounts with multi-factor authentication (Req. 8.4.2) |
| Meridian Pay staff payment screen | Corporate Active Directory | Stolen AR login could take payments or view data | Remove staff card entry (secure payment links) |

### 5.2 Shared Services

| Service | Used By CDE | Used By Non-CDE | Segmentation Effective? |
|---------|-------------|-----------------|------------------------|
| DNS | Yes | Yes | No (not separated or tested) |
| Time (NTP) | Yes | Yes | No (low risk) |
| Active Directory | Yes | Yes | **No** |
| Backup | Yes | Yes | **No** |
| Logging (SIEM) | Yes | Yes | Not verified |
| Patching | Yes | Yes | **No** |

---

## 6. Segmentation Assessment

### 6.1 Segmentation Controls

| Control | Description | Effectiveness | Evidence |
|---------|-------------|---------------|----------|
| Firewall rules | Rules between zones exist | Weak | No rule review or business justification produced |
| Network separation | CDE has its own address range | Weak | Separate range, but shared services cross it |
| Access controls | Admin access via jump host | Unproven | No evidence provided (F5) |
| Monitoring | CDE logs to SIEM | Weak | Real card environment largely unmonitored (F4) |
| Segmentation testing | None | None | Never performed (F8) |

### 6.2 Segmentation Gaps

| Gap ID | Description | Risk | Remediation |
|--------|-------------|------|-------------|
| GAP-001 | Staff handle card numbers by phone and email | High | Secure payment links; stop email intake |
| GAP-002 | CDE trusts corporate Active Directory | High | Separate CDE accounts with MFA |
| GAP-003 | Unencrypted payment data link to InvoiceTrack | High | Remove direct link; send no card data |
| GAP-004 | Security codes in call recordings and email | High | Stop recording payments; Legal-approved purge |
| GAP-005 | Shared patch and backup servers | Med | Dedicated or hardened in-scope servers |
| GAP-006 | Jump host rule unproven | Med | Document, test, add MFA |
| GAP-007 | Segmentation never tested | High | Test after fixes (Req. 11.4.5) |

---

## 7. QSA Challenge Preparation

Full detail in the [segmentation gap analysis](segmentation-gap-analysis.md).

### 7.1 Anticipated Questions

| Area | Likely Question | Our Response | Evidence |
|------|-----------------|--------------|----------|
| Shared Services | "Which accounts can log in to CDE servers?" | Today, corporate AD accounts. Moving to separate CDE accounts with MFA. | Account list; MFA configuration |
| Admin Access | "Prove admin access only comes through the jump host." | Not yet proven. Rule to be documented and tested. | Firewall rules; test results |
| Logging | "Do your logs contain card numbers?" | Not yet checked. Log search scheduled. | Log review report |
| Backups | "Where are backups and who can reach them?" | Shared backup server, now treated as in scope and restricted. | Access list; encryption settings |
| Phone payments | "Where does a card number go on a phone payment?" | Today, through staff and recordings. Moving to secure payment links. | New process; vendor contract |
| InvoiceTrack | "What data goes to InvoiceTrack?" | Today, payment details, unencrypted. Moving to no card data and no direct link. | Sample message; new design |

---

## 8. Recommendations

### 8.1 Immediate Actions (0 to 30 days)
1. [ ] Pause call recording during payments and stop accepting card numbers by email (automatic reply with a secure payment link)
2. [ ] With Legal, define and run a purge of recordings and emails containing security codes, isolating anything under a hold
3. [ ] Search SIEM logs for card numbers and mask any found

### 8.2 Short-Term Actions (30 to 90 days)
1. [ ] Replace staff card entry with secure payment links (or a keypad-entry phone system) so AR staff never see card numbers
2. [ ] Remove the direct CDE-to-InvoiceTrack link; have InvoiceTrack receive only invoice number and "paid" status, without card data, through a path that does not connect to the CDE
3. [ ] Create separate CDE login accounts with multi-factor authentication; document, test, and enforce the jump host rule
4. [ ] Add PCI responsibilities to the phone vendor and payment processor contracts (Req. 12.8)

### 8.3 Long-Term Actions (90+ days, before the Q1 2027 assessment)
1. [ ] Run a segmentation penetration test on the corrected design (Req. 11.4.5)
2. [ ] Review all firewall rules between zones, with a business reason for each
3. [ ] Complete the InvoiceTrack documentation work in RA-2026-005 and close RA-2026-004

---

## 9. Defended Exclusions (Target State)

These systems are **in scope today** and can be **excluded** only once the conditions are met and evidenced.

| System | Exclusion Argument | Condition That Must Be Proven |
|--------|-------------------|-------------------------------|
| Ordinary employee laptops (about 1,500) | They never handle card data and cannot reach the CDE or the systems that secure it | CDE no longer trusts corporate logins; segmentation test shows no route from the corporate network into the CDE |
| AR laptops | AR staff no longer see or type card numbers | Secure payment links in use; staff payment screen removed or disabled for card entry |
| VPN gateway | No card data passes through it | Same as AR laptops |
| AR inbox and email system | Card numbers no longer accepted or kept | Automatic reply in place; purge completed and verified |
| Phone vendor | No card data in calls or recordings | Payment recording stopped or keypad entry in use; contract updated |
| InvoiceTrack | No connection to the CDE and no card data | Direct link removed; segmentation test confirms |
| Corporate Active Directory | CDE no longer depends on it | Separate CDE accounts in place and tested |

**Systems that stay in scope regardless:** payment application, card database, tokenization service, HSM, web servers and load balancer, jump host, backup system, patch server (or dedicated CDE patch server), SIEM, and the DNS and time sources the CDE uses.

---

## 10. Residual Risk Statement

After implementing recommendations, the following risks remain:

| Risk | Likelihood | Impact | Acceptance Required? |
|------|------------|--------|---------------------|
| Customers keep emailing card numbers despite the automatic reply | Medium | Medium | No (handled by procedure and training) |
| Emails with card data kept under a litigation hold | Low | High | Yes (Legal and risk owner) |
| Customers targeted by fake "payment link" scams copying our links | Medium | Medium | Yes (customer communications plan) |
| Legacy InvoiceTrack still fragile and poorly documented | Medium | Medium | Yes (tracked in RA-2026-004 and RA-2026-005) |
| Shared corporate DNS and time servers remain in scope | High | Low | No |
| Staff mistakes, such as writing down a card number heard on a call | Low | Medium | No (training and spot checks) |

---

## Appendices

### A. Network Diagram
See the zone table in [scenario.md](../scenario.md).

### B. Data Flow Diagram
See [card-data-flow.md](card-data-flow.md).

### C. Evidence Index

| Evidence ID | Description | Location |
|-------------|-------------|----------|
| E-01 | Discovery interview notes (IT, AR, Finance) | Risk & Compliance files |
| E-02 | Card data flow map | [card-data-flow.md](card-data-flow.md) |
| E-03 | Trust relationship matrix | [trust-relationship-matrix.md](trust-relationship-matrix.md) |
| E-04 | Gap analysis and QSA readiness | [segmentation-gap-analysis.md](segmentation-gap-analysis.md) |
| E-05 | Firewall rules export | To be obtained from IT |
| E-06 | Segmentation test report | To be commissioned |

---

## Approval

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Assessor | Faith Olofintuyi, Risk & Compliance | October 1, 2026 | |
| Reviewer | Director of IT Security | | |
| Approver | Chief Financial Officer | | |
