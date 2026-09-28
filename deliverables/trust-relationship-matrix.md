# Trust Relationship Matrix: Meridian Pay

> Fictional scenario. See [scenario.md](../scenario.md) and the [card data flow map](card-data-flow.md).

**Question this answers:** What crosses the wall around the cardholder data environment (CDE), and how dangerous is each crossing?

**Why it matters:** Segmentation only removes a system from PCI scope if it cannot affect the security of the CDE. Every shared service is a door in the wall. A system on the other side of a poorly guarded door stays in scope, and it becomes a path for an attacker.

## The Matrix

| # | Door | What Crosses | Risk | Why | Scope Impact |
|---|------|--------------|------|-----|--------------|
| 1 | **InvoiceTrack** | Payment app sends payment details to InvoiceTrack (corporate zone) to mark invoices paid, over an unencrypted connection | **Critical** | InvoiceTrack's folders are reachable by external partners through inherited permissions ([RA-2026-004](https://github.com/designbyfaith/grc-risk-acceptance)), and nobody understands its configuration ([RA-2026-005](https://github.com/designbyfaith/grc-risk-acceptance)). An unencrypted link means anyone on the corporate network could read what passes. Outsiders are one step from the CDE. | InvoiceTrack is at least a **connected system** and is in scope. If card data reaches it, it becomes **part of the CDE**. |
| 2 | **Logins (Active Directory)** | CDE servers accept logins from corporate AD accounts | **Critical** | Whoever controls AD controls every CDE login. A phished IT account anywhere in the company could lead straight to card data. | AD domain controllers are **security-impacting** and in scope, pulling much of the corporate identity system in with them. |
| 3 | **Remote access (VPN)** | AR staff type card numbers on home laptops and send them through the VPN | **High** | Card data travels through the VPN and sits, however briefly, on laptops outside company premises. | The VPN gateway and **every AR laptop** used for phone or email payments are in scope. |
| 4 | **Admin access (jump host)** | IT staff administer CDE servers through the jump host | **High** | The jump host is one door to every CDE server. If it is compromised, or if admins reach it from ordinary laptops without strong protection, the whole CDE is exposed. | Jump host is in scope. So are the admin workstations used to reach it, unless strongly isolated. |
| 5 | **Software updates** | Patch server in the management zone pushes updates to CDE servers | **High** | A server that can install software on CDE systems can also install malicious software. | Patch server is **security-impacting** and in scope. |
| 6 | **Backups** | Card database is copied to the shared backup server | **High** | Backups contain the same card numbers as the database. | Backup server, backup storage, and any offsite copies **store card data** and are part of the CDE. |
| 7 | **Logs** | CDE activity logs go to the SIEM in the management zone | **Medium** | The SIEM protects the CDE by watching it, so it is in scope regardless. If logs contain full card numbers, it also stores card data. Not yet checked. | SIEM is **security-impacting** and in scope; possibly part of the CDE. |
| 8 | **Name lookups (DNS)** *(assumption: corporate DNS)* | CDE servers look up addresses using corporate DNS | **Medium** | A tampered DNS server could redirect CDE traffic to an attacker. | Corporate DNS servers are **security-impacting** and in scope. |
| 9 | **Time sync** *(assumption: corporate time server)* | CDE servers set their clocks from a corporate time server | **Low** | Wrong clocks make logs unreliable for investigations, but do not directly expose data. | Time source is in scope as security-impacting, with low risk. |

## Why InvoiceTrack Ranks First

Active Directory is the textbook highest-risk door, because it holds the keys to every login. I rank InvoiceTrack alongside it, and first for this company, because of what my earlier projects found:

- **Outsiders already have a foothold.** External partners can reach InvoiceTrack's folders today.
- **Nobody can safely change it.** The engineer who built it is gone, and there is no documentation.
- **The connection is readable.** Payment details cross the wall unencrypted.

Put together, InvoiceTrack is where the risk from two separate projects meets the card environment.

**A note on encryption:** PCI DSS strictly requires encryption when card data crosses open, public networks (Requirement 4.2.1). Internal connections are not held to the same rule. The unencrypted link is still a serious weakness here, because the corporate network is not trustworthy: outsiders reach parts of it, and remote staff connect from home.

## What a QSA Will Ask

| Door | Likely Question |
|------|-----------------|
| InvoiceTrack | "Exactly what data does the payment app send to InvoiceTrack? Show me." |
| Active Directory | "Which accounts can log in to CDE servers, and is multi-factor authentication required?" (Requirement 8.4.2) |
| VPN | "How do you know which laptops have handled card numbers?" and "Is multi-factor authentication required for remote access?" (Requirement 8.4.3) |
| Jump host | "From which devices can admins reach the jump host, and how are those devices protected?" |
| Patch server | "Who can push software to CDE systems, and how is that controlled?" |
| Backups | "Where are backups stored, who can access them, and are they encrypted?" |
| Logs | "Have you confirmed that no full card numbers appear in logs?" |

## Summary

Of nine doors, two are **critical** and four are **high**. The documented claim that the CDE is "fully segmented" does not hold. At a minimum, InvoiceTrack, Active Directory, the VPN, AR laptops, the jump host, the patch server, the backup system, and the SIEM all fall into scope.
