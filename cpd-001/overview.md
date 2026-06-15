---
spec_id: FPSF-CPD-001
title: "Canonical Payment Definition — Overview"
family: "Canonical Payment Definition"
layer: "Overview"
version: "1.0.0"
status: Draft
date: 2026-03-25
author: "Adalton Reis <reis@fabricpaymentstandards.org>"
organization: Fabric Payment Standards Foundation
contact: specs@fabricpaymentstandards.org
license: Apache-2.0
---

# FPSF-CPD-001 — Overview

> **Layer: Overview** · Audience: institutional evaluators, architects, regulators
> For normative requirements, see the [Formal Specification](./SPEC.md).

---

## What This Is

Every payment system that has ever existed shares the same underlying structure: someone causes value to move from one party to another, and that transfer is recorded somewhere. A card payment, a wire transfer, a digital wallet transaction, a stablecoin transfer — all of them, at their core, are instances of the same abstraction.

FPSF-CPD-001 makes that abstraction explicit.

The Canonical Payment Definition is not a new payment system. It is a precise description of what a payment already is — a shared conceptual foundation that institutions, networks, and systems can reference without changing how they work internally.

---

## Why This Matters

Digital payments today are built on many different infrastructures, each with its own data models, terminology, and integration patterns. When two systems need to interoperate, they must negotiate their differences manually, specification by specification, API by API.

A common abstract model does not eliminate that work, but it dramatically reduces it. When both sides of an integration share a reference model, the negotiation becomes a mapping exercise rather than a conceptual reconstruction from scratch.

FPSF-CPD-001 provides that reference model.

---

## What It Defines

The specification defines:

- A **Payment** and a **Digital Payment** as formal abstractions
- The minimal set of **entities** (payer, payee, value, authorization, context) involved in any payment
- A **lifecycle** with a finite set of states and permitted transitions
- A set of **interfaces** reflecting the operations common to all payment systems

It deliberately excludes everything that varies between implementations: settlement mechanics, compliance requirements, cryptographic algorithms, wire formats, and network-specific rules.

---

## How Other Specifications Use It

FPSF-CPD-001 is the root of the Foundation's specification tree. Other specifications adopt it as their payment model:

- **FPSF-CPP-001** (CashPack Protocol) uses the Payment lifecycle and Authorization model as the basis for its bearer instrument flows
- **FPSF-SS-001** (Stablecoin Stack) maps its on-chain settlement operations to the CPD-001 lifecycle states

Adopting a common base means that a developer who understands CPD-001 can navigate any specification in the family with less friction.

---

## Document Map

| Layer | Document | Purpose |
|---|---|---|
| **Formal Specification** | [SPEC.md](./SPEC.md) | Normative definitions |
| Overview | *this document* | What it is and why it exists |
| [Core Concepts](./core-concepts.md) | Key abstractions explained |
| [Reference](./reference.md) | Glossary |
| [Guides](./guides.md) | Adoption guidance |
| [Governance](./governance.md) | Versioning and changelog |

---

*FPSF-CPD-001 v1.0.0 · Draft · Fabric Payment Standards Foundation · Apache-2.0*
