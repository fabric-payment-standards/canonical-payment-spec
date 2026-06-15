---
spec_id: FPSF-CPD-001
title: "Canonical Payment Definition — Core Concepts"
family: "Canonical Payment Definition"
layer: "Core Concepts"
version: "1.0.0"
status: Draft
date: 2026-03-25
author: "Adalton Reis <reis@fabricpaymentstandards.org>"
organization: Fabric Payment Standards Foundation
contact: specs@fabricpaymentstandards.org
license: Apache-2.0
---

# FPSF-CPD-001 — Core Concepts

> **Layer: Core Concepts** · Audience: architects, developers, technical evaluators
> For normative requirements, see the [Formal Specification](./SPEC.md).

---

## The Payment as a Structured Act

The specification defines a payment not as a transaction record, but as a **structured act**: something that happens, with identifiable parties, a defined value, and a traceable outcome. This framing is intentional — it focuses on the observable properties of a payment rather than the mechanisms that implement it.

Three things must be true for an act to qualify as a payment under this model:

1. A payer causes value to move toward a payee.
2. A defined amount of a defined asset changes entitlement.
3. At least one system of record recognizes the change.

Everything else — how the authorization is expressed, what network carries the instruction, how settlement is finalized — is an implementation detail that this specification deliberately leaves open.

---

## Authorization as a First-Class Concept

One of the more consequential decisions in the specification is treating **authorization** as a distinct entity rather than a property of the payment instruction.

In traditional payment systems, authorization is often implicit — embedded in the act of presenting a card, entering a PIN, or clicking a confirm button. FPSF-CPD-001 makes it explicit: a payment has an `AuthorizationProof`, which must be cryptographically or institutionally bound to the payer, the value, and the payee.

This design makes three things possible:

- Authorization can be verified by any party with access to the proof, not just the originating institution.
- The binding constraint (payer + value + payee) prevents a proof issued for one payment from being reused for another.
- Different authorization mechanisms (signatures, multi-party approval, institutional mandate) can be accommodated within the same model.

---

## The Lifecycle as a Finite State Machine

The payment lifecycle is deliberately minimal: six states with a restricted set of permitted transitions. The key design decisions are:

**Monotonicity.** Transitions move forward only. There is no "undo" of a state. A payment that has reached `SETTLED` cannot be moved to `CANCELLED` under the CPD-001 model — reversals and refunds, if needed, are separate payments in the opposite direction.

**Three terminal states.** `SETTLED`, `FAILED`, and `CANCELLED` are all final. This reflects reality: a payment that has definitively succeeded, failed, or been revoked does not change state again.

**A single in-flight state.** The model does not attempt to enumerate the internal states of a processing network. Once a payment is submitted, it is `IN_FLIGHT` until it resolves. What happens inside that window — routing, clearing, queuing — is outside the scope of this specification.

---

## Digital Payment as a Specialization

The Digital Payment is not a separate concept but a constrained version of the general Payment model. The additional constraints are:

- Participants are identified by **digital identifiers** (not necessarily legal identity)
- Authorization is **machine-verifiable** (not reliant on physical presence or institutional trust alone)
- Transmission happens over **electronic channels**
- State transitions produce **machine-readable outputs**

These constraints are consistent with the design of all modern digital payment protocols, including those that use cryptographic signatures as authorization proofs.

---

## What the Interfaces Represent

The six interfaces defined in the specification (initiate, authorize, submit, settle, query, cancel) are not API definitions. They are **operational archetypes** — the minimal set of operations that any payment system must support, regardless of how it exposes them externally.

A card network, a blockchain settlement system, and an interbank wire system all implement these operations. They use different protocols, different data formats, and different trust models — but the underlying operations are the same. The interfaces make this visible.

---

*FPSF-CPD-001 v1.0.0 · Draft · Fabric Payment Standards Foundation · Apache-2.0*
