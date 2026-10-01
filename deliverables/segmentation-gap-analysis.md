# Segmentation Gap Analysis and QSA Readiness: Meridian Pay

> Fictional scenario. See [scenario.md](../scenario.md), the [card data flow map](card-data-flow.md), and the [trust relationship matrix](trust-relationship-matrix.md).

**Question this answers:** Do the five promises in the Meridian Pay security design document hold up, and what will a Qualified Security Assessor (QSA) ask about each gap?

## Scorecard

| # | Documented Claim | Verdict | Finding |
|---|------------------|---------|---------|
| 1 | "The CDE is fully segmented from the corporate network." | **False** | F2 |
| 2 | "Only authorized systems can communicate with the CDE." | **False** | F3 |
| 3 | "Administrative access requires going through the jump host." | **Unproven** | F5 |
| 4 | "All CDE traffic is logged and monitored." | **False** | F4 |
| 5 | "Employees never see or handle full card numbers." | **False** | F1 |

Four of five claims are false, and the fifth has no evidence behind it. Three further findings (F6 to F8) are gaps the design document does not mention at all.

---

## F1: Employees Handle Full Card Numbers Every Day

**Severity:** Critical

**Documented claim:** "Employees never see or handle full card numbers; the portal and processor handle all card data."

**Reality:** AR specialists hear card numbers on calls, read them in emails, copy them to their clipboard, and type them into the portal from home laptops. About 30% of card payments come in by phone.

**Why it matters:** Every AR laptop, the VPN, the email system, and the phone vendor's systems handle card data. The real card environment is many times larger than the documented one.

**QSA challenge:** "Walk me through a phone payment from start to finish. Where does the card number go?"

**Evidence required:** A complete data flow for phone and email payments, and an inventory of every device used to take them.

**Recommendation:** Stop card numbers reaching staff. Replace phone payments with a secure payment link sent by text or email, or a phone system that lets customers key in their number so staff never hear it. Stop accepting card numbers by email (see F7).

---

## F2: The CDE Shares Core Services With the Corporate Network

**Severity:** Critical

**Documented claim:** "The CDE is fully segmented from the corporate network."

**Reality:** CDE servers log in with corporate Active Directory accounts, get updates from the shared patch server, are backed up to the shared backup server, and exchange data with InvoiceTrack on the corporate network.

**Why it matters:** A wall with four open doors is not a wall. Each shared service pulls corporate systems into scope and gives an attacker a route in. Most importantly, anyone who takes over an IT account in Active Directory can reach card data.

**QSA challenge:** "Show me the firewall rules between the corporate zone and the CDE, and explain the business reason for each one."

**Evidence required:** Current firewall rule set with a business justification for every rule; list of Active Directory accounts that can log in to CDE servers.

**Recommendation:** Give the CDE its own login accounts (separate from everyday corporate accounts) with multi-factor authentication, as Requirement 8.4.2 expects for all access into the CDE. Treat the patch and backup servers as in scope and protect them to the same standard.

---

## F3: InvoiceTrack Links the CDE to a System Outsiders Can Reach

**Severity:** Critical

**Documented claim:** "Only authorized systems can communicate with the CDE."

**Reality:** The payment application sends payment details to InvoiceTrack over an unencrypted connection. External partners can reach InvoiceTrack's folders through inherited permissions ([RA-2026-004](https://github.com/designbyfaith/grc-risk-acceptance)), and nobody now employed understands its configuration ([RA-2026-005](https://github.com/designbyfaith/grc-risk-acceptance)).

**Why it matters:** "Authorized" means approved and controlled. A system that outsiders can reach, that nobody can safely change, and that receives readable payment details is not controlled. It is one step from the CDE.

**QSA challenge:** "Exactly what data goes to InvoiceTrack? Show me a sample message."

**Evidence required:** A captured sample of the data sent to InvoiceTrack; firewall rules limiting the connection; InvoiceTrack's access list.

**Recommendation:** Send InvoiceTrack only what it needs (invoice number and "paid" status, never card data). Encrypt the connection. Limit the firewall rule to that single connection. Complete the permission mapping in RA-2026-005, which this finding now makes more urgent.

---

## F4: Large Parts of the Real Card Environment Are Not Logged

**Severity:** High

**Documented claim:** "All CDE traffic is logged and monitored."

**Reality:** The documented CDE sends logs to the SIEM, but nobody has checked whether those logs contain card numbers. The parts of the real card environment outside the documented CDE (AR laptops, the AR inbox, call recordings, InvoiceTrack) have no card-focused monitoring, and InvoiceTrack's folders have no file audit logging at all.

**Why it matters:** If card data is stolen from the inbox or a laptop, nothing would raise an alarm, and an investigation would have little to work with.

**QSA challenge:** "Show me that your logs contain no full card numbers, and show me the alerts you would get if someone copied the card database."

**Evidence required:** A log review confirming no full card numbers; SIEM alert rules for the CDE; logging coverage for every in-scope system.

**Recommendation:** Search existing logs for card numbers and mask any found. Extend logging and alerting to every system now in scope.

---

## F5: The Jump Host Claim Has No Evidence Behind It

**Severity:** High (until proven otherwise)

**Documented claim:** "Administrative access requires going through the jump host."

**Reality:** Unproven. No firewall rules or access records have been produced to show that CDE servers refuse admin connections from anywhere else. Because CDE servers accept corporate Active Directory logins (F2), an IT admin may be able to connect directly.

**Why it matters:** A claim without evidence is treated as false in an assessment.

**QSA challenge:** "Prove that a CDE server rejects an admin connection that does not come from the jump host."

**Evidence required:** Firewall rules allowing admin traffic into the CDE only from the jump host; a test showing a direct connection is refused; multi-factor authentication on the jump host.

**Recommendation:** Confirm and document the rule, test it, and protect the jump host itself with multi-factor authentication and restricted admin devices.

---

## F6: Call Recordings Keep Security Codes

**Severity:** Critical

**Documented claim:** None. The design document does not mention the phone system.

**Reality:** The cloud phone system records every AR call, and customers read out card numbers and security codes. The recordings and automatic transcripts keep both.

**Why it matters:** PCI DSS forbids storing security codes after a payment is approved (Requirement 3.3.1). This is one of the most serious violations an assessment can find. The phone vendor is also handling card data, which makes it a third-party service provider the company must manage under Requirement 12.8.

**QSA challenge:** "Do your call recordings contain card numbers or security codes? How long are they kept, and who can play them?"

**Evidence required:** Phone system settings; recording retention policy; contract and PCI compliance evidence from the phone vendor.

**Recommendation:** Pause recording during payments, or adopt a system that keeps card details out of the call (see F1). Securely delete existing recordings that contain security codes, after Legal confirms no hold applies. Add PCI responsibilities to the phone vendor's contract.

**Legal note:** California requires the consent of all parties to record confidential calls (Penal Code §632). Confirm customers hear a recording notice at the start of every AR call.

---

## F7: Card Numbers and Security Codes Sit in the Email System

**Severity:** High

**Documented claim:** None.

**Reality:** Customers email card numbers, and sometimes security codes, to the AR shared inbox. Staff "usually" delete them, but copies remain in deleted items, archives, backups, synced phones, and quoted replies.

**Why it matters:** The company email system, every user with inbox access, and email backups may now be in scope. Security codes in email are another Requirement 3.3.1 violation.

**QSA challenge:** "Search your email system for card numbers. What do you find?"

**Evidence required:** Results of a search of the AR inbox and archive for card numbers; an email handling procedure.

**Recommendation:** Send an automatic reply telling customers not to email card details, with a link to pay securely. Run a supervised search and purge. Train AR staff on what to do when card data arrives by email.

**Legal note:** Purging email can conflict with retention rules or a litigation hold. Legal must approve the cleanup scope first, and any held emails containing card data should be isolated and protected rather than kept in an open inbox.

---

## F8: Segmentation Has Never Been Tested

**Severity:** High

**Documented claim:** The design document describes segmentation but cites no test.

**Reality:** Meridian Pay launched in March 2026. No penetration test has checked whether the wall actually blocks traffic.

**Why it matters:** PCI DSS requires segmentation to be tested at least once every 12 months and after any change (Requirement 11.4.5). A QSA cannot accept reduced scope without that test.

**QSA challenge:** "Show me your most recent segmentation test results."

**Evidence required:** A segmentation penetration test report covering every route into the CDE.

**Recommendation:** Commission a segmentation test before the Q1 2027 assessment, after the fixes above, so the results reflect the corrected design.

---

## Priorities Before the QSA Assessment

| Priority | Action | Findings Addressed |
|----------|--------|--------------------|
| 1 | Stop taking card numbers by phone and email; switch to secure payment links | F1, F6, F7 |
| 2 | Stop recording security codes; clean up existing recordings and email (with Legal) | F6, F7 |
| 3 | Strip card data from the InvoiceTrack connection and encrypt it | F3 |
| 4 | Separate CDE login accounts and require multi-factor authentication | F2, F5 |
| 5 | Prove the jump host rule; review all firewall rules | F2, F5 |
| 6 | Check logs for card numbers; extend monitoring | F4 |
| 7 | Run a segmentation test after fixes | F8 |

Priority 1 matters most: it removes card data from people, laptops, the VPN, email, and the phone vendor in one step, and shrinks the card environment back toward what the design document intended.
