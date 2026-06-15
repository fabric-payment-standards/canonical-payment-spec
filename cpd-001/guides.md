---
spec_id: FPSF-CPD-001
title: "Canonical Payment Definition — Guides"
family: "Canonical Payment Definition"
layer: "Guides"
version: "1.0.0"
status: Draft
date: 2026-03-25
author: "Adalton Reis <reis@fabricpaymentstandards.org>"
organization: Fabric Payment Standards Foundation
contact: specs@fabricpaymentstandards.org
license: Apache-2.0
---

# FPSF-CPD-001 — Guides

> **Layer: Guides** · Audience: architects, developers building on CPD-001
> For normative requirements, see the [Formal Specification](./SPEC.md).

---

## Guide 1: Mapping an Existing System to CPD-001

The Canonical Payment Definition is designed to be mapped onto existing systems, not to replace them. This guide walks through that mapping exercise.

### Step 1 — Identify your payment object

Find the data structure in your system that represents a single payment or transaction. This is your `Payment` equivalent. Map its unique identifier to `PaymentIdentifier`.

### Step 2 — Map participants

Identify who sends value (your `Payer`) and who receives it (your `Payee`). If your system uses internal account identifiers, these become the `Participant.id`. If your system has routing intermediaries, they map to participants with the `intermediary` role.

### Step 3 — Map value

Identify how your system represents amount and currency or asset type. These map directly to `Value.amount` and `Value.asset`.

### Step 4 — Map authorization

Find where your system records that the payer approved the payment. This may be a PIN verification record, a signature, a session token, or an institutional mandate. This maps to `AuthorizationProof`. The `method` field captures the mechanism; the `data` field captures the proof itself.

### Step 5 — Map lifecycle states

Walk through the states your payment can be in and map each to the closest CPD-001 state. Most systems have analogs of all six states, though they may use different names.

### Step 6 — Map interfaces

Identify the API calls or functions in your system that correspond to each CPD-001 interface. The mapping need not be one-to-one — a single CPD-001 interface may correspond to multiple internal steps in your system.

---

## Guide 2: Adopting CPD-001 in a New Specification

If you are writing a specification that builds on CPD-001 (as FPSF-CPP-001 and FPSF-SS-001 do), follow these conventions:

**Reference the model explicitly.** State in your specification's scope section that it adopts FPSF-CPD-001 as its payment model. Cite the version.

**Specialize, don't redefine.** If your protocol adds properties to the Payment object, add them as extensions to the CPD-001 model rather than replacing the model. Use the extensibility provisions in CPD-001 Section 8.

**Map lifecycle states.** Your protocol's lifecycle may be more detailed than CPD-001's. Map each of your states to the closest CPD-001 state. Document the mapping explicitly.

**Adopt the invariants.** The invariants in CPD-001 Section 7 (uniqueness, integrity, authorization binding, deterministic state, finality) MUST be preserved by any conformant extension.

---

## Guide 3: Using CPD-001 for Interoperability

When two systems need to exchange payment data and both have adopted CPD-001, the shared model provides a translation layer.

For each payment being exchanged:

1. Map the source system's payment object to CPD-001 entities.
2. Map the CPD-001 entities to the target system's payment object.
3. Verify that the authorization proof can be validated by the target system, or establish a trust arrangement that substitutes for direct validation.
4. Map the current lifecycle state and ensure the target system can honor the terminal state semantics.

This process does not eliminate integration work, but it replaces open-ended negotiation with a bounded mapping exercise.

---

*FPSF-CPD-001 v1.0.0 · Draft · Fabric Payment Standards Foundation · Apache-2.0*
