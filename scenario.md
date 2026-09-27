# Scenario: Meridian Pay

> Author: Faith Olofintuyi
> Fictional scenario. The company, people, systems, and figures are invented for this portfolio. Sierra Meridian also appears in my [risk acceptance project](https://github.com/designbyfaith/grc-risk-acceptance).

## The Organization

Sierra Meridian Field Services, Inc. (fictional) is an oil and gas operations contractor headquartered in Sacramento, California, with 1,610 employees (about 70% remote) and about $285M in annual revenue. It holds a SOC 2 Type II report and is subject to the California Consumer Privacy Act (CCPA).

## What Changed

Until 2026, customers paid Sierra Meridian only by bank transfer. Smaller customers, mostly independent well operators, asked to pay by card. In March 2026 the company launched **Meridian Pay**:

- **Online payments:** customers log in to a portal and pay service invoices by card.
- **Cards on file:** customers on monthly service contracts can save a card for automatic billing.
- **Phone payments:** customers can call Accounts Receivable (AR), and an AR specialist types the card number into the portal's staff payment screen.

Volume is about 48,000 card transactions a year (assumption). Because Sierra Meridian **stores card numbers** for recurring billing, its acquiring bank requires an assessment by a Qualified Security Assessor (QSA) under PCI DSS v4.0.1. The first assessment is scheduled for Q1 2027.

## The Network (as documented by IT)

| Zone | Address Range | What Is There |
|------|---------------|---------------|
| DMZ | 10.1.0.0/24 | Meridian Pay web servers (3), load balancer |
| Corporate | 10.2.0.0/16 | Active Directory domain controllers, file servers, jump host, employee workstations, **InvoiceTrack** server (legacy AP/AR system), VPN gateway for remote staff |
| Cardholder Data Environment (CDE) | 10.3.0.0/24 | Payment application server, card database (cards on file), hardware security module (HSM), tokenization service, log forwarder |
| Management | 10.4.0.0/24 | SIEM (central logging), backup server, patch management server |
| Cloud services | N/A | Company email (AR shared inbox), cloud phone system with call recording |

## What the Documentation Claims

The Meridian Pay security design document states:

1. "The CDE is fully segmented from the corporate network."
2. "Only authorized systems can communicate with the CDE."
3. "Administrative access requires going through the jump host."
4. "All CDE traffic is logged and monitored."
5. "Employees never see or handle full card numbers; the portal and processor handle all card data."

## What I Learned in Discovery

These facts came out in interviews with IT, AR, and Finance. None of them appear in the design document.

- **Remote phone payments.** AR specialists take card numbers over the phone from home and type them into the portal's staff payment screen on company laptops, connected through the VPN.
- **Call recordings.** The cloud phone system records all AR calls "for quality purposes." Card numbers and security codes are spoken aloud and captured in the recordings.
- **Card details by email.** Some customers email card numbers, and sometimes security codes, to the AR shared inbox. AR staff copy them into the portal and "usually" delete the emails.
- **InvoiceTrack connection.** The payment application calls InvoiceTrack, on the corporate network, to look up open invoices and mark them paid.
- **Shared services.** CDE servers use corporate Active Directory accounts for logins, get patches from the management zone, and are backed up to the shared backup server.
- **Logging.** CDE logs flow to the SIEM in the management zone. Nobody has checked whether the logs contain card numbers.

## Constraints

- The first QSA assessment is in Q1 2027. There is no time for a redesign before then.
- IT is still short-staffed after the last reduction in force (see RA-2026-005 in the risk acceptance project).
- AR needs to keep taking phone payments; about 30% of card payments come in by phone (assumption).
- Budget for PCI work is limited and must be justified against scope reduction.

## My Role

I am on the Risk & Compliance team, preparing the scope determination and QSA readiness review before the assessment.
