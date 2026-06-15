---
spec_id: FPSF-CPD-001
title: "Canonical Payment Definition — Reference"
family: "Canonical Payment Definition"
layer: "Reference"
version: "1.0.0"
status: Draft
date: 2026-03-25
author: "Adalton Reis <reis@fabricpaymentstandards.org>"
organization: Fabric Payment Standards Foundation
contact: specs@fabricpaymentstandards.org
license: Apache-2.0
---

# FPSF-CPD-001 — Reference

> **Layer: Reference** · Audience: developers, implementers
> For normative requirements, see the [Formal Specification](./SPEC.md).

---

## Glossary

| Term | Definition |
|---|---|
| **Payment** | A structured act by which a payer causes the transfer of value to a payee, resulting in a change of entitlement recognized by one or more systems of record. |
| **Digital Payment** | A Payment in which the expression of intent, authorization, transmission, and confirmation are performed through digital representations and processed by computational systems. |
| **Payer** | The participant who initiates the transfer of value. |
| **Payee** | The participant who receives the transferred value. |
| **Intermediary** | A participant who facilitates a Payment without being the ultimate sender or receiver of value. |
| **Value** | The economic unit being transferred, defined by an amount and an asset type. |
| **Asset** | The denomination of the value being transferred (e.g., a currency code, a tokenized asset identifier). |
| **Payment Identifier** | A unique identifier assigned to a Payment at creation. MUST be unique within the relevant system of record. |
| **Intent Descriptor** | A structure describing the purpose and conditions of a Payment. Optional. |
| **Authorization Proof** | Evidence that the payer has approved the Payment. MUST be bound to the payer, value, and payee. |
| **Authorization Method** | The mechanism by which the Authorization Proof was produced (e.g., cryptographic signature, institutional mandate). |
| **Execution Context** | Optional metadata describing the environment in which a Payment is processed (network, rail, settlement model). |
| **Payment Draft** | An incomplete Payment object used as input to the initiation interface. Contains all fields except identifier and state. |
| **System of Record** | Any system that authoritatively records the state of a Payment or the change in entitlement resulting from it. |

---

## Payment State Reference

| State | Description | Terminal |
|---|---|---|
| `CREATED` | Payment object instantiated; not yet authorized. | No |
| `AUTHORIZED` | Payer approval established; ready for submission. | No |
| `IN_FLIGHT` | Submitted to a processing system; awaiting resolution. | No |
| `SETTLED` | Value transfer finalized. | **Yes** |
| `FAILED` | Irrecoverable failure during processing. | **Yes** |
| `CANCELLED` | Explicitly revoked by the payer before settlement. | **Yes** |

---

## Permitted State Transitions

| From | To |
|---|---|
| `CREATED` | `AUTHORIZED` |
| `AUTHORIZED` | `IN_FLIGHT` |
| `AUTHORIZED` | `CANCELLED` |
| `IN_FLIGHT` | `SETTLED` |
| `IN_FLIGHT` | `FAILED` |

All other transitions are prohibited.

---

## Interface Summary

| Interface | Input | Output | State Transition |
|---|---|---|---|
| `initiatePayment` | `PaymentDraft` | `Payment` | → `CREATED` |
| `authorizePayment` | `paymentId`, `AuthorizationProof` | `Payment` | `CREATED` → `AUTHORIZED` |
| `submitPayment` | `paymentId`, `ExecutionContext` | `Payment` | `AUTHORIZED` → `IN_FLIGHT` |
| `settlePayment` | `paymentId` | `Payment` | `IN_FLIGHT` → `SETTLED` |
| `getPaymentStatus` | `paymentId` | `PaymentState` | — |
| `cancelPayment` | `paymentId` | `Payment` | `AUTHORIZED` → `CANCELLED` |

---

*FPSF-CPD-001 v1.0.0 · Draft · Fabric Payment Standards Foundation · Apache-2.0*
