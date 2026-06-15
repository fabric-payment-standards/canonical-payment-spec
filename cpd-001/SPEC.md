---
spec_id: FPSF-CPD-001
title: "Canonical Payment Definition"
family: "Canonical Payment Definition"
layer: "Formal Specification"
version: "1.0.0"
status: Draft
date: 2026-03-25
author: "Adalton Reis <reis@fabricpaymentstandards.org>"
organization: Fabric Payment Standards Foundation
contact: specs@fabricpaymentstandards.org
license: Apache-2.0
---

# FPSF-CPD-001 — Canonical Payment Definition

## Document Metadata

| Field | Value |
|---|---|
| **Spec ID** | FPSF-CPD-001 |
| **Title** | Canonical Payment Definition |
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Date** | 2026-03-25 |
| **Author** | Adalton Reis — [reis@fabricpaymentstandards.org](mailto:reis@fabricpaymentstandards.org) |
| **Organization** | Fabric Payment Standards Foundation |
| **Contact** | [specs@fabricpaymentstandards.org>](mailto:specs@fabricpaymentstandards.org>) |
| **License** | Apache-2.0 |

---

## Abstract

This document defines a minimal, neutral, and implementation-agnostic conceptual model of a **Payment** and a **Digital Payment**, including their essential properties, lifecycle states, and interaction interfaces.

The purpose of this specification is not to introduce new payment mechanisms, nor to regulate existing systems, but to **formalize the shared abstraction already present across global payment practices**. By making implicit structures explicit, it establishes a common language and reference model that institutions, networks, and systems can adopt without altering their internal operations.

This document serves as a foundational layer upon which further specifications, interoperability models, and reference implementations may be developed.

---

## 1. Scope

This specification:

- Defines the concept of a **Payment** independent of medium, infrastructure, or jurisdiction
- Defines a **Digital Payment** as a specialization of Payment
- Establishes a minimal set of **entities, properties, and lifecycle states**
- Describes **interfaces** that reflect operational patterns observed in existing systems

This specification does **not**:

- Define settlement mechanisms
- Define regulatory or compliance requirements
- Mandate specific technologies or cryptographic methods
- Replace existing financial standards or network rules

---

## 2. Normative Language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in RFC 2119.

---

## 3. Definitions

### 3.1 Payment

A **Payment** is:

> **A structured act by which a payer causes the transfer of value to a payee, resulting in a change of entitlement recognized by one or more systems of record.**

Key characteristics:

- Involves **at least two parties**: payer and payee
- Expresses **value** — an amount denominated in a defined asset
- Produces a **state transition**: a change in ownership or entitlement
- Is **recognized** by a system capable of recording or validating the change

### 3.2 Digital Payment

A **Digital Payment** is:

> **A Payment in which the expression of intent, authorization, transmission, and confirmation are performed through digital representations and processed by computational systems.**

Additional characteristics:

- Uses **digital identifiers** for participants
- Uses **machine-verifiable authorization** (e.g., cryptographic signatures, institutional credentials)
- Is processed through **electronic communication channels**
- Produces **machine-readable state transitions**

---

## 4. Core Entities

### 4.1 Participant

An entity capable of initiating or receiving a Payment.

```
Participant {
    id:    Identifier
    roles: Set<Role>   // e.g., payer, payee, intermediary
}
```

### 4.2 Value

The economic unit being transferred.

```
Value {
    amount: Numeric
    asset:  AssetType  // e.g., currency code, tokenized asset identifier
}
```

### 4.3 Payment Object

The canonical representation of a Payment.

```
Payment {
    id:            PaymentIdentifier

    payer:         Participant
    payee:         Participant

    value:         Value

    intent:        IntentDescriptor
    authorization: AuthorizationProof

    state:         PaymentState

    context:       ExecutionContext  // optional
}
```

### 4.4 Intent Descriptor

Describes the purpose and conditions of the Payment.

```
IntentDescriptor {
    reference:  String          // optional
    conditions: Set<Condition>  // optional
}
```

### 4.5 Authorization Proof

Evidence that the payer has approved the Payment. Implementations MUST bind the proof to the payer, the value, and the payee.

```
AuthorizationProof {
    method: AuthorizationMethod
    data:   Opaque
}
```

Non-exhaustive examples of authorization methods:

- Cryptographic signature (e.g., Ed25519, ECDSA)
- Multi-party approval
- Institutional authorization (e.g., bank mandate)

### 4.6 Execution Context (Optional)

Describes the environment in which the Payment is processed.

```
ExecutionContext {
    network:         Identifier      // optional
    rail:            Identifier      // optional
    settlementModel: SettlementType  // optional
}
```

---

## 5. Payment Lifecycle

A Payment progresses through a finite, ordered set of states:

```
enum PaymentState {
    CREATED     // Payment object instantiated
    AUTHORIZED  // Payer approval established
    IN_FLIGHT   // Submitted to processing system
    SETTLED     // Value transfer finalized
    FAILED      // Irrecoverable failure
    CANCELLED   // Explicitly revoked before settlement
}
```

### 5.1 State Transition Constraints

Permitted transitions:

```
CREATED    → AUTHORIZED
AUTHORIZED → IN_FLIGHT
IN_FLIGHT  → SETTLED
IN_FLIGHT  → FAILED
AUTHORIZED → CANCELLED
```

All transitions are:

- **Monotonic**: no reversal of state is permitted
- **Externally observable**: current state MUST be queryable by authorized parties

### 5.2 Terminal States

`SETTLED`, `FAILED`, and `CANCELLED` are terminal states. No further transitions are permitted once a Payment reaches one of these states.

---

## 6. Interfaces

The following interfaces represent **operational patterns** common across payment systems. They are expressed abstractly and do not prescribe transport, protocol, or encoding.

### 6.1 Payment Initiation

```
initiatePayment(input: PaymentDraft) -> Payment
```

- Constructs a Payment object from a draft
- Assigns a unique identifier
- Sets initial state: `CREATED`

### 6.2 Authorization

```
authorizePayment(paymentId, authorizationProof) -> Payment
```

- Attaches the authorization proof to the Payment
- Validates the proof against the payer, value, and payee
- Transitions state to `AUTHORIZED`

### 6.3 Submission

```
submitPayment(paymentId, context) -> Payment
```

- Submits the Payment to an execution environment
- Transitions state to `IN_FLIGHT`

### 6.4 Settlement

```
settlePayment(paymentId) -> Payment
```

- Finalizes the value transfer
- Transitions state to `SETTLED`

### 6.5 Status Query

```
getPaymentStatus(paymentId) -> PaymentState
```

- Returns the current state of the Payment
- MUST be consistent with the authoritative system of record

### 6.6 Cancellation

```
cancelPayment(paymentId) -> Payment
```

- Valid only before settlement (i.e., from `AUTHORIZED` state)
- Transitions state to `CANCELLED`

---

## 7. Invariants

All compliant implementations MUST preserve the following invariants:

**1. Uniqueness** — Each Payment has a globally or contextually unique identifier.

**2. Integrity** — Payment data MUST NOT be altered after authorization without invalidating the Payment.

**3. Authorization Binding** — The `AuthorizationProof` MUST be bound to the payer, the value, and the payee. A proof valid for one combination MUST NOT be reusable for a different combination.

**4. Deterministic State** — At any moment, a Payment has exactly one state.

**5. Finality** — `SETTLED`, `FAILED`, and `CANCELLED` are terminal states. No transition out of these states is permitted.

---

## 8. Extensibility

This specification is designed to be extended through:

- Additional lifecycle states (e.g., `REVERSED`, `REFUNDED`, `PENDING_REVIEW`)
- Richer intent descriptors
- Advanced authorization schemes (e.g., threshold cryptography, multi-signature)
- Interoperability layers between heterogeneous systems

Extensions MUST NOT contradict the invariants defined in Section 7.

---

## 9. Non-Goals

This specification intentionally excludes:

- Identity and KYC standards
- Compliance mechanisms (AML, sanctions screening)
- Liquidity or credit models
- Consensus or validation mechanisms
- Wire formats, messaging schemas (e.g., JSON schemas, XML)
- Specific network or settlement rail requirements

---

## 10. Conformance

An implementation is considered **FPSF-CPD-001 conformant** if:

- It represents Payments according to the model defined in Section 4
- It enforces the lifecycle constraints defined in Section 5
- It exposes operationally equivalent interfaces to those defined in Section 6
- It preserves all invariants defined in Section 7

---

## Normative References

| Reference | Description |
|---|---|
| RFC 2119 | Key words for use in RFCs to Indicate Requirement Levels |

---

*FPSF-CPD-001 v1.0.0 · Draft · Fabric Payment Standards Foundation · Apache-2.0*
