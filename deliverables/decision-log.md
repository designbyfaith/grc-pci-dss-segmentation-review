# Decision Log: PCI DSS Scope and Segmentation Review

A running record of the judgment calls I made while building this project, and why.

| Date | Decision | Alternatives Considered | Why I Chose This |
|------|----------|-------------------------|------------------|
| 2026-09-27 | Extend the Sierra Meridian scenario instead of using the brief's MidTech case | Use MidTech as written; create a new company | I wanted to show that risks in an org don't live in silos. Sticking with Sierra Meridian let me follow the same weak spot, InvoiceTrack, from my risk acceptance project straight into the card environment. A standalone case would never have shown that. It also let me build in the messy, real-world problems a clean diagram hides: remote staff taking card numbers over the phone, calls being recorded, and customers emailing their card details. Using my own scenario instead of the brief's case also makes it clear the analysis is mine. |
| 2026-10-01 | Treat all corporate laptops as in scope today, with a defined path to exclude them | Exclude non-AR laptops now; keep them in scope permanently | Ordinary laptops share the network and login system with the services that protect the card environment, and segmentation has never been tested, so today they cannot be excluded. "Better safe than sorry" is not a reason on its own: scoping everything forever is expensive and spreads protection thin. So the scope document shows two states, today and after the fixes, and lists the evidence needed to exclude them. |
