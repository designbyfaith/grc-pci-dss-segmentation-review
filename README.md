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
| [Card Data Flow Map](deliverables/card-data-flow.md) | Every way a card number enters the company, and everywhere it goes |
| [Trust Relationship Matrix](deliverables/trust-relationship-matrix.md) | The nine "doors" through the wall around the card environment, rated by risk |
| [Gap Analysis and QSA Readiness](deliverables/segmentation-gap-analysis.md) | Eight findings, each with the auditor's likely question, the evidence needed, and the fix |
| [Scope Determination](deliverables/scope-determination.md) | What is in scope today, what can be excluded after the fixes, and the proof required |
| [Decision Log](deliverables/decision-log.md) | The judgment calls I made, and why |
| [Reflection](reflection.md) | What I learned, and the risks that remain |
| [Project Brief](PROJECT-BRIEF.md) | The original exercise brief (credited to its author) |

## Status

All written deliverables are complete. A video walkthrough will follow.

- [x] Scenario
- [x] Card data flow diagram
- [x] Trust relationship matrix
- [x] Segmentation gap analysis (5+ findings)
- [x] QSA challenge preparation
- [x] Scope determination document
- [x] Reflection
