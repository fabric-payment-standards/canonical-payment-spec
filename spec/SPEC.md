---
document: canonical-payment/spec.md
title: Canonical Payment Definition — Conceptual Baseline (CPD-001)
description: Foundational specification defining the canonical model of a Payment and Digital Payment
spec_id: FPSF-CPD-001
version: 1.0.0
status: Draft
date: 2026-03-25
author: Adalton Reis <reis@fabricpaymentstandards.org>
organization: Fabric Payment Standards Foundation
contact: contact@fabricpaymentstandards.org
license: Apache-2.0
---

# FPSF-PD-001 — Payment Definition - PD-001

# Metadata

| Field            | Value                                           |
|------------------|-------------------------------------------------|
| **Document ID**  | FPSF-CPD-001                                    |
| **Title**        | Canonical Payment Definition — Conceptual Baseline (CPD-001)         |
| **Version**      | 0.1.0                                           |
| **Status**       | Draft                                           |
| **Date**         | 2026-03-25                                      |
| **Conformance**  | Self-contained                                  |
| **Author(s)**    | Adalton Reis (reis@fabricpaymentstandards.org)         |
| **Reviewers**    | —                                               |
| **Organization** | Fabric Payment Standards Foundation             |
| **Contact**      | contact@fabricpaymentstandards.org                       |
| **License**      | Apache License 2.0                              |

---


## **Abstract**

This document defines a minimal, neutral, and implementation-agnostic conceptual model of a **Payment** and a **Digital Payment**, including their essential properties, lifecycle, and interaction interfaces.

The purpose of this specification is not to introduce new payment mechanisms, nor to regulate existing systems, but to **formalize the shared abstraction already present across global payment practices**. By making implicit structures explicit, this specification establishes a common language and reference model that can be adopted by institutions, networks, and systems without altering their internal operations.

This document serves as a foundational layer upon which further specifications, interoperability models, and reference implementations may be developed.

---

## **1. Scope**

This specification:

* Defines the concept of a **Payment** independent of medium, infrastructure, or jurisdiction
* Defines a **Digital Payment** as a specialization of Payment
* Establishes a minimal set of **entities, properties, and lifecycle states**
* Describes **interfaces** that reflect de facto operational patterns observed in existing systems

This specification does **not**:

* Define settlement mechanisms
* Define regulatory or compliance requirements
* Mandate specific technologies or cryptographic methods
* Replace existing financial standards or network rules

---

## **2. Definitions**

### **2.1 Payment**

A **Payment** is:

> **A structured act by which a payer causes the transfer of value to a payee, resulting in a change of entitlement recognized by one or more systems of record.**

#### Key characteristics:

* Involves **at least two parties**: payer and payee
* Expresses **value** (amount and asset)
* Produces a **state transition** (before → after ownership/entitlement)
* Is **recognized** by a system capable of recording or validating the change

---

### **2.2 Digital Payment**

A **Digital Payment** is:

> **A Payment in which the expression of intent, authorization, transmission, and confirmation are performed through digital representations and processed by computational systems.**

#### Additional characteristics:

* Uses **digital identifiers** for participants
* Uses **machine-verifiable authorization** (e.g., signatures, credentials)
* Is processed through **electronic communication channels**
* Produces **machine-readable state transitions**

---

## **3. Core Entities**

### **3.1 Participant**

An entity capable of initiating or receiving a Payment.

```
Participant {
    id: Identifier
    roles: Set<Role>  // e.g., payer, payee, intermediary
}
```

---

### **3.2 Value**

The economic unit being transferred.

```
Value {
    amount: Numeric
    asset: AssetType   // e.g., currency, tokenized value
}
```

---

### **3.3 Payment Object**

The canonical representation of a Payment.

```
Payment {
    id: PaymentIdentifier

    payer: Participant
    payee: Participant

    value: Value

    intent: IntentDescriptor
    authorization: AuthorizationProof

    state: PaymentState

    context: ExecutionContext (optional)
}
```

---

### **3.4 Intent Descriptor**

Describes the purpose and conditions of the Payment.

```
IntentDescriptor {
    reference: String (optional)
    conditions: Set<Condition> (optional)
}
```

---

### **3.5 Authorization Proof**

Evidence that the payer has approved the Payment.

```
AuthorizationProof {
    method: AuthorizationMethod
    data: Opaque
}
```

Examples (non-exhaustive):

* cryptographic signature
* multi-party approval
* institutional authorization

---

### **3.6 Execution Context (Optional)**

Describes the environment in which the Payment is processed.

```
ExecutionContext {
    network: Identifier (optional)
    rail: Identifier (optional)
    settlementModel: SettlementType (optional)
}
```

---

## **4. Payment Lifecycle**

A Payment progresses through a finite set of states:

```
enum PaymentState {
    CREATED        // Payment object instantiated
    AUTHORIZED     // Payer approval established
    IN_FLIGHT      // Submitted to processing system
    SETTLED        // Value transfer finalized
    FAILED         // Irrecoverable failure
    CANCELLED      // Explicitly revoked before settlement
}
```

### **State Transition Constraints**

* `CREATED → AUTHORIZED`
* `AUTHORIZED → IN_FLIGHT`
* `IN_FLIGHT → SETTLED | FAILED`
* `AUTHORIZED → CANCELLED`

Transitions are:

* **monotonic** (no reversal of state)
* **externally observable** (must be queryable)

---

## **5. Interfaces (Conceptual)**

The following interfaces represent **de facto operations** common across payment systems.

They are expressed abstractly and do not prescribe transport or protocol.

---

### **5.1 Payment Initiation Interface**

```
initiatePayment(input: PaymentDraft) -> Payment
```

* Constructs a Payment object
* Assigns identifier
* Sets initial state: `CREATED`

---

### **5.2 Authorization Interface**

```
authorizePayment(paymentId, authorizationProof) -> Payment
```

* Attaches authorization proof
* Transitions state to `AUTHORIZED`

---

### **5.3 Submission Interface**

```
submitPayment(paymentId, context) -> Payment
```

* Submits Payment to execution environment
* Transitions state to `IN_FLIGHT`

---

### **5.4 Settlement Interface**

```
settlePayment(paymentId) -> Payment
```

* Finalizes value transfer
* Transitions state to `SETTLED`

---

### **5.5 Status Query Interface**

```
getPaymentStatus(paymentId) -> PaymentState
```

* Returns current state
* Must be consistent with system of record

---

### **5.6 Cancellation Interface**

```
cancelPayment(paymentId) -> Payment
```

* Valid only before settlement
* Transitions to `CANCELLED`

---

## **6. Invariants**

All compliant implementations must ensure:

1. **Uniqueness**

   * Each Payment has a globally or contextually unique identifier

2. **Integrity**

   * Payment data cannot be altered after authorization without invalidating the Payment

3. **Authorization Binding**

   * AuthorizationProof must be cryptographically or institutionally bound to:

     * payer
     * value
     * payee

4. **Deterministic State**

   * At any moment, a Payment has exactly one state

5. **Finality**

   * `SETTLED`, `FAILED`, and `CANCELLED` are terminal states

---

## **7. Non-Goals**

This specification intentionally avoids:

* Defining identity standards
* Defining compliance mechanisms (AML, KYC)
* Defining liquidity or credit models
* Defining consensus or validation mechanisms
* Defining messaging formats (e.g., XML, JSON schemas)

---

## **8. Extensibility**

This specification is designed to be extended through:

* Additional states (e.g., “reversed”, “refunded”)
* Richer intent descriptors
* Advanced authorization schemes (e.g., threshold cryptography)
* Interoperability layers between heterogeneous systems

---

## **9. Conformance**

An implementation is considered **PFS-compliant** if:

* It represents Payments according to the defined model
* It enforces lifecycle constraints
* It exposes equivalent operational interfaces
* It preserves the invariants defined in Section 6

---

## **Closing Note**

This specification does not attempt to unify payment systems.

It establishes a **shared descriptive foundation**—a stable anchor—upon which diverse systems may align without relinquishing their internal models, governance, or regulatory obligations.

Its value lies in **clarity, minimality, and universality**.
