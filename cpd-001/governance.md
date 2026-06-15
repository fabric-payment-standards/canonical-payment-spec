---
spec_id: FPSF-CPD-001
title: "Canonical Payment Definition — Governance"
family: "Canonical Payment Definition"
layer: "Governance"
version: "1.0.0"
status: Draft
date: 2026-03-25
author: "Adalton Reis <reis@fabricpaymentstandards.org>"
organization: Fabric Payment Standards Foundation
contact: specs@fabricpaymentstandards.org
license: Apache-2.0
---

# FPSF-CPD-001 — Governance

> **Layer: Governance** · Audience: contributors, adopters, compliance reviewers

---

## Versioning Policy

This specification follows Semantic Versioning 2.0.0.

| Segment | Increment when... |
|---|---|
| **MAJOR** | A breaking change is made to the data model, lifecycle, invariants, or interface definitions. Adopters of the previous major version must re-evaluate conformance. |
| **MINOR** | New entities, states, or interfaces are added in a backward-compatible manner. Existing conformant implementations are unaffected. |
| **PATCH** | Editorial corrections, clarifications, or non-normative changes. Conformance is unaffected. |

Changes of MAJOR version require a review period of no less than 30 days with documented rationale and a migration guide.

---

## Stability Commitment

FPSF-CPD-001 is intentionally minimal. The Foundation's commitment is to keep it stable. Breaking changes will be rare and will only be considered when required to correct a fundamental design issue, not to add features. Features are added through minor versions or through dependent specifications.

---

## Relationship to Other Specifications

FPSF-CPD-001 is the root of the Foundation's specification tree. Any specification that adopts it MUST:

- Declare the CPD-001 version it conforms to
- Preserve all invariants defined in CPD-001 Section 7
- Not redefine core terms in ways that contradict CPD-001 definitions

---

## Contributing

The Foundation welcomes contributions from financial institutions, payment networks, fintech developers, researchers, and regulators.

- **Specification repository:** https://github.com/fabric-payment-standards/specs
- **Community:** https://discord.com/invite/fabric-payment-standards
- **Contact:** specs@fabricpaymentstandards.org

---

## Changelog

| Version | Date | Summary |
|---|---|---|
| 1.0.0 | 2026-03-25 | Initial release. Defines the canonical payment model, lifecycle, interfaces, and invariants. |

---

*FPSF-CPD-001 v1.0.0 · Draft · Fabric Payment Standards Foundation · Apache-2.0*
