# PCI DSS Scope and Segmentation Review: Meridian Pay

**Author:** Faith Olofintuyi, GRC professional and attorney

A pre-assessment review of whether a company's "segmented" card payment environment really is, written to survive questioning by a Qualified Security Assessor (QSA). It treats segmentation as a claim to be proven, not a box on a diagram.

> The company, people, and figures are fictional.

## The Situation in Brief

Sierra Meridian Field Services, a Sacramento oil and gas contractor, started accepting card payments in 2026 through a new portal called Meridian Pay. IT documented a neatly walled-off card environment. Discovery found card numbers traveling through remote employees' laptops, call recordings, an email inbox, and a legacy system on the corporate network.

## What's in This Repo

| File | What It Is |
|------|------------|
| [Scenario](scenario.md) | The company, the network, the documentation claims, and what discovery found |
| [Decision Log](deliverables/decision-log.md) | The judgment calls I made, and why |
| [Project Brief](PROJECT-BRIEF.md) | The original exercise brief (credited to its author) |

## Status

- [x] Scenario
- [ ] Card data flow diagram
- [ ] Trust relationship matrix
- [ ] Segmentation gap analysis (5+ findings)
- [ ] QSA challenge preparation
- [ ] Scope determination document
- [ ] Reflection
