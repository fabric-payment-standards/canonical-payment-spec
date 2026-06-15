---
document: canonical-payment/cpd-002.md
title: Canonical Payment Request Definition (CPD-002)
description: Specification defining the canonical model of a Payment Request — a portable, payer-agnostic, rail-neutral structure from which a Payment may be derived
spec_id: FPSF-CPD-002
version: 1.0.0
status: Draft
date: 2026-06-15
author: Adalton Reis <reis@fabricpaymentstandards.org>
organization: Fabric Payment Standards Foundation
contact: contact@fabricpaymentstandards.org
license: Apache-2.0
conformsTo: FPSF-CPD-001 v1.1.0
---

# FPSF-CPD-002 — Canonical Payment Request Definition (CPD-002)

## Metadata

| Field            | Value                                                              |
|------------------|--------------------------------------------------------------------|
| **Document ID**  | FPSF-CPD-002                                                       |
| **Title**        | Canonical Payment Request Definition (CPD-002)                     |
| **Version**      | 1.0.0                                                              |
| **Status**       | Draft                                                              |
| **Date**         | 2026-06-15                                                         |
| **Conformance**  | Self-contained; references FPSF-CPD-001 v1.1.0                     |
| **Author(s)**    | Adalton Reis (reis@fabricpaymentstandards.org)                     |
| **Reviewers**    | —                                                                  |
| **Organization** | Fabric Payment Standards Foundation                                |
| **Contact**      | contact@fabricpaymentstandards.org                                 |
| **License**      | Apache License 2.0                                                 |

---

## Abstract

This document defines the **Canonical Payment Request (CPR)** — a portable, payer-agnostic, rail-neutral structure that expresses a payee's intent to receive value, from which a conformant Payment (as defined in FPSF-CPD-001) may be derived.

A Payment Request is not a Payment. It carries no information about the payer, no authorization proof, and no state. It is the structured expression of a payment need — the raw material a paying application reads, interprets, and uses to initiate a payment through its own mechanisms.

This specification defines the base CPR structure, a delivery model for portable transport (QR code, deep-link, URL token), a derivation interface connecting a request to a CPD-001 Payment, and a set of domain-specific sub-interfaces for common payment rails.

---

## 1. Scope

This specification:

- Defines the **Canonical Payment Request** as a payer-agnostic, rail-neutral structure
- Establishes a minimal **base interface** common to all payment request types
- Defines **domain-specific sub-interfaces** for on-chain/stablecoin, traditional account-based, and SEPA/IBAN-based payments
- Specifies a **delivery model** for portable transport of payment requests
- Defines the **derivation relationship** between a Payment Request and a canonical Payment (CPD-001)
- Describes **extension and versioning** conventions

This specification does **not**:

- Define how a payment is authorized, settled, or confirmed
- Define identity or compliance requirements for the payer
- Prescribe specific wallet or application UX behavior
- Replace existing payment initiation standards (ISO 20022, SEPA Credit Transfer, ERC-681, etc.)

---

## 2. Definitions

### 2.1 Payment Request

A **Payment Request** is:

> **A structured, payer-agnostic expression of a payee's intent to receive value, providing sufficient information for a conformant application to derive and initiate a Payment on the payer's behalf.**

#### Key characteristics:

- Contains **no payer information** — identity, account, or wallet of the payer is never included
- Carries **no payment state** — no status, no authorization, no confirmation
- Is **rail-neutral at the base level** — the base interface does not mandate a specific settlement mechanism
- Is **time-bounded** — every request carries a validity window
- Is **single-use by convention** — a request should not be accepted after it has been fulfilled or expired

### 2.2 Derivation

**Derivation** is the act by which a paying application reads a Payment Request and produces a Payment (as defined in FPSF-CPD-001) by binding payer information, selecting the appropriate rail, and constructing the authorization. The Payment Request is an input to this process; it does not participate in or constrain it beyond the stated parameters.

### 2.3 Payee

The entity or system generating and presenting the Payment Request — a person, merchant, application, or automated system.

### 2.4 Paying Application

Any application (wallet, banking app, payment service) that reads a Payment Request and is capable of initiating a conformant Payment from it.

---

## 3. Base Payment Request Interface

All conformant Payment Requests, regardless of payment rail, MUST include the following fields. Sub-interfaces add rail-specific fields on top of this base.

```
CanonicalPaymentRequest {
    cpr_version:    String              // MUST be "CPR-1.0"
    request_id:     Identifier          // Globally unique. UUIDv4 RECOMMENDED.
    created_at:     Timestamp           // ISO 8601 UTC
    expires_at:     Timestamp           // ISO 8601 UTC — after this, the request MUST NOT be fulfilled
    payee:          PayeeDescriptor
    value:          RequestedValue
    memo:           String (optional)   // Human-readable description; max 140 characters
    rail_hint:      RailHint (optional) // Non-binding indication of preferred rail
    extensions:     Map<String, Any> (optional)
}
```

### 3.1 PayeeDescriptor

Identifies the receiving party. The fields provided depend on the payment rail. At least one of the optional fields MUST be present in a conformant request.

```
PayeeDescriptor {
    display_name:   String (optional)   // Human-readable name of the payee
    rail_address:   RailAddress         // Rail-specific address (see Section 5)
}
```

### 3.2 RequestedValue

Expresses the amount and asset the payee expects to receive. If `amount` is omitted, the payer is expected to determine the amount (e.g., open-ended donation requests).

```
RequestedValue {
    amount:         Numeric (optional)  // Amount in the smallest unit of the asset
    asset:          AssetDescriptor
}

AssetDescriptor {
    type:           AssetType           // "fiat", "stablecoin", "crypto", or "tokenized"
    code:           String              // ISO 4217 for fiat; token symbol or contract ref for others
    chain_id:       String (optional)   // Required when type is "stablecoin" or "tokenized"
}
```

### 3.3 RailHint

A non-binding signal indicating the payee's preferred settlement rail. Paying applications MAY ignore this and use any rail they support, provided the value and asset constraints are honored.

```
enum RailHint {
    ON_CHAIN        // Stablecoin or crypto settlement
    SEPA_CT         // SEPA Credit Transfer
    SEPA_INST       // SEPA Instant
    ACH             // US ACH
    PIX             // Brazilian PIX
    UPI             // Indian UPI
    SWIFT           // International wire
    OPEN            // No preference
}
```

---

## 4. Delivery Model

A Payment Request MAY be delivered to a paying application through any of the following mechanisms. Implementations SHOULD support at least QR code and URL Token delivery.

### 4.1 QR Code

The Payment Request is encoded as a URI and rendered as a QR code. The paying application scans the QR code, resolves the URI, retrieves the request payload, and presents it to the user.

**URI format:**

```
fpsf-cpr://<token>?endpoint=<url>
```

- `<token>` — a short-lived, single-use ephemeral token (see Section 4.3)
- `endpoint` — the URL from which the full CPR JSON payload can be retrieved

Alternatively, where inline encoding is feasible (e.g., small requests), the full JSON payload MAY be Base64url-encoded and embedded directly:

```
fpsf-cpr://inline/<base64url-encoded-json>
```

### 4.2 Deep Link

The same URI format used for QR codes is used as a mobile deep link, allowing OS-level routing to the appropriate paying application.

### 4.3 URL Token (Ephemeral Session)

When the payment request is served from a URL, the endpoint MUST be protected by an **ephemeral session token** — a short-lived, single-use credential that authorizes retrieval of the CPR payload.

**Token requirements:**

- MUST expire within a payee-configured window (RECOMMENDED: no longer than 15 minutes)
- MUST be invalidated after first successful retrieval
- MUST NOT be reusable — repeated use of the same token MUST return an error
- MUST be transmitted only over TLS

**Retrieval response:**

```http
GET /cpr/<token>
Content-Type: application/json

{
    "cpr_version": "CPR-1.0",
    "request_id": "...",
    ...
}
```

### 4.4 File / Direct JSON

A CPR payload MAY be transmitted as a JSON file through any channel (email attachment, messaging app, NFC, etc.). Paying applications MUST validate the `cpr_version`, `request_id`, and `expires_at` fields before processing.

### 4.5 Expiry and Single-Use Convention

Once a Payment Request has been successfully fulfilled, the system generating it SHOULD mark it as consumed and reject subsequent fulfillment attempts against the same `request_id`. This is a convention, not a cryptographic guarantee at the CPR level — enforcement is the payee system's responsibility.

---

## 5. Domain-Specific Sub-Interfaces

The following sub-interfaces extend the base CPR with fields specific to a payment rail. The `rail_hint` in the base request indicates which sub-interface applies.

### 5.1 On-Chain Sub-Interface (Stablecoin / Tokenized)

Used for payment requests to be fulfilled via on-chain stablecoin or tokenized asset transfer, including deployments conforming to FPSF-SS-001.

```
OnChainPaymentRequest extends CanonicalPaymentRequest {
    rail_hint:          "ON_CHAIN"          // MUST be set
    chain_id:           String              // EVM chain ID or equivalent
    receiver_address:   String              // EVM address of the intended recipient
    domain_separator:   String (optional)   // EIP-712 domain separator of the target token
    order_reference:    String (optional)   // 16-byte reference for reconciliation
    acquirer_id:        String (optional)   // 16-byte Acquirer ID (Zero-UUID if absent)
    contract_address:   String (optional)   // Settlement contract address, if applicable
}
```

**Example (FPSF-SS-001 compatible):**

```json
{
    "cpr_version": "CPR-1.0",
    "request_id": "e4a1b2c3-d4e5-6f70-8190-a1b2c3d4e5f6",
    "created_at": "2026-06-15T14:00:00Z",
    "expires_at": "2026-06-15T14:15:00Z",
    "payee": {
        "display_name": "Alice",
        "rail_address": {
            "receiver_address": "0xAbCd1234...",
            "chain_id": "137"
        }
    },
    "value": {
        "amount": "5000000",
        "asset": {
            "type": "stablecoin",
            "code": "USDC",
            "chain_id": "137"
        }
    },
    "memo": "Dinner at Trattoria Roma",
    "rail_hint": "ON_CHAIN",
    "chain_id": "137",
    "receiver_address": "0xAbCd1234...",
    "domain_separator": "0x...",
    "order_reference": "0x..."
}
```

### 5.2 IBAN-Based Sub-Interface

Used for credit transfers to an IBAN-identified account. Covers SEPA Credit Transfer, SEPA Instant, SWIFT, and similar rail types.

```
IBANPaymentRequest extends CanonicalPaymentRequest {
    iban:               String              // Full IBAN of the receiving account
    bic:                String (optional)   // BIC/SWIFT code of the receiving institution
    beneficiary_name:   String              // Legal name of the account holder
    remittance_info:    String (optional)   // Unstructured remittance information (max 140 chars)
    end_to_end_id:      String (optional)   // End-to-end reference for reconciliation
    purpose_code:       String (optional)   // ISO 20022 purpose code (e.g., "GDDS", "SALA")
}
```

**Example (SEPA Instant):**

```json
{
    "cpr_version": "CPR-1.0",
    "request_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "created_at": "2026-06-15T19:00:00Z",
    "expires_at": "2026-06-15T19:10:00Z",
    "payee": {
        "display_name": "Trattoria Roma S.r.l.",
        "rail_address": {
            "iban": "IT60X0542811101000000123456",
            "bic": "BPPIITRR"
        }
    },
    "value": {
        "amount": "4200",
        "asset": {
            "type": "fiat",
            "code": "EUR"
        }
    },
    "memo": "Table 7 — 15 Jun 2026",
    "rail_hint": "SEPA_INST",
    "iban": "IT60X0542811101000000123456",
    "bic": "BPPIITRR",
    "beneficiary_name": "Trattoria Roma S.r.l.",
    "end_to_end_id": "E2E20260615001",
    "purpose_code": "GDDS"
}
```

### 5.3 Account-Based Sub-Interface (Non-IBAN)

Used for domestic and international account-based transfers where IBAN is not used.

```
AccountPaymentRequest extends CanonicalPaymentRequest {
    account_number:     String              // Account number in the local format
    routing_code:       String (optional)   // Routing number, sort code, BSB, IFSC, etc.
    routing_code_type:  String (optional)   // "ABA", "SORT_CODE", "BSB", "IFSC", etc.
    bank_name:          String (optional)
    beneficiary_name:   String
    remittance_info:    String (optional)   // Max 140 characters
    end_to_end_id:      String (optional)
}
```

### 5.4 Instant Payment Sub-Interface (PIX, UPI, FPS, etc.)

Used for domestic instant payment systems that use a key-based addressing model.

```
InstantKeyPaymentRequest extends CanonicalPaymentRequest {
    key_type:           String              // "CPF", "PHONE", "EMAIL", "EVP", "UPI_ID", etc.
    key_value:          String              // The actual key value
    beneficiary_name:   String (optional)
    txid:               String (optional)   // Merchant transaction ID (e.g., PIX TXID)
}
```

**Example (PIX):**

```json
{
    "cpr_version": "CPR-1.0",
    "request_id": "f9e8d7c6-b5a4-3210-fedc-ba9876543210",
    "created_at": "2026-06-15T20:30:00Z",
    "expires_at": "2026-06-15T20:45:00Z",
    "payee": {
        "display_name": "Maria Souza",
        "rail_address": {
            "key_type": "CPF",
            "key_value": "012.345.678-90"
        }
    },
    "value": {
        "amount": "2500",
        "asset": {
            "type": "fiat",
            "code": "BRL"
        }
    },
    "memo": "Aluguel julho",
    "rail_hint": "PIX",
    "key_type": "CPF",
    "key_value": "012.345.678-90",
    "beneficiary_name": "Maria Souza",
    "txid": "PIX20260615001"
}
```

---

## 6. Derivation Interface

This section defines how a conformant paying application derives a **Payment** (FPSF-CPD-001) from a **Payment Request** (FPSF-CPD-002). The derivation process makes explicit what the CPR does and does not provide.

### 6.1 Derivation Inputs

```
DerivationContext {
    request:        CanonicalPaymentRequest     // The CPR being fulfilled
    payer:          Participant                 // Resolved by the paying application
    rail:           ExecutionContext            // Chosen by the paying application
    authorization:  AuthorizationProof          // Produced by the paying application
}
```

The fields `payer`, `rail`, and `authorization` are **never present in the CPR**. They are supplied by the paying application at derivation time.

### 6.2 Derivation Steps

1. **Parse and validate** the CPR: verify `cpr_version`, `request_id`, `expires_at` (MUST be in the future), and structural integrity.
2. **Select rail**: the paying application chooses an appropriate settlement rail, using `rail_hint` as a non-binding signal. The application MAY reject the request if it cannot serve the indicated rail.
3. **Resolve payer**: the paying application binds its own user (payer) identity and account to the transaction. This information does not come from the CPR.
4. **Construct value**: map `RequestedValue` to the concrete `Value` structure defined in CPD-001. If the CPR asset and the payer's available asset differ (e.g., currency conversion), this is the paying application's responsibility.
5. **Construct IntentDescriptor**: derive from `memo` (as `reference`) and any conditions implied by the sub-interface fields.
6. **Produce AuthorizationProof**: the paying application generates authorization according to its own mechanism (cryptographic signature, institutional approval, etc.).
7. **Construct Payment**: assemble the CPD-001 Payment object with the resolved payer, payee (from `PayeeDescriptor`), value, intent, and authorization. Bind `request_id` to the payment's context for traceability.

### 6.3 Formal Derivation Mapping

```
derive(DerivationContext) -> Payment {

    payment.id              = generate()
    payment.payer           = context.payer
    payment.payee           = resolve(request.payee)
    payment.value           = map(request.value, context.rail)
    payment.intent          = IntentDescriptor {
                                reference:  request.memo ?? request.request_id
                                conditions: rail_specific(request)
                              }
    payment.authorization   = context.authorization
    payment.state           = CREATED
    payment.context         = context.rail
}
```

### 6.4 What Derivation Does Not Do

- Derivation does NOT transfer state from the CPR to the Payment. The CPR has no state.
- Derivation does NOT require the payee system to be involved at authorization time.
- Derivation does NOT constrain the payer's choice of rail beyond the `rail_hint` signal.
- A single CPR MAY, in principle, be used to derive multiple payment attempts (e.g., retry after failure), subject to the single-use convention in Section 4.5.

---

## 7. Interoperability Considerations

### 7.1 Rail Mismatch

A paying application that receives a CPR with a `rail_hint` it does not support SHOULD display a human-readable message indicating that the payment method is not supported, and MAY offer the user an alternative if one is available. It MUST NOT silently ignore the request.

### 7.2 Currency and Asset Conversion

When the CPR specifies an asset the paying application cannot directly use, conversion is at the paying application's discretion and must be disclosed to the payer before confirmation. The CPR does not govern the exchange or conversion process.

### 7.3 Partial Payments

The CPR base interface does not support partial payments. If `amount` is specified, a conformant paying application MUST fulfill the full amount or reject the request. Sub-interfaces MAY define partial payment semantics through extensions.

### 7.4 Open Amount Requests

When `amount` is omitted in `RequestedValue`, the paying application MUST prompt the payer to specify the amount. The payer-specified amount becomes the payment amount at derivation.

---

## 8. Invariants

All conformant implementations MUST ensure:

1. **No payer data in the request** — a CPR MUST NOT contain any field that identifies, references, or assumes a specific payer

2. **Expiry enforcement** — a paying application MUST NOT fulfill a request whose `expires_at` is in the past

3. **Request_id uniqueness** — payee systems MUST assign a globally unique `request_id` to every request they generate

4. **Rail hint is advisory** — `rail_hint` MUST NOT be treated as a mandatory constraint by the paying application

5. **Value integrity** — the `amount` and `asset` in the CPR MUST be preserved unmodified in the derived Payment; any conversion is additional context, not a substitution

6. **Separation of concerns** — the CPR carries no authorization, no settlement instruction, and no payer context; these are always added at derivation time

---

## 9. Extensibility

The `extensions` field in the base interface accommodates future or proprietary fields without breaking conformance. Implementations MUST ignore unrecognized extension fields.

New domain-specific sub-interfaces MAY be defined by:

- The Foundation, through the specification process
- Third parties, through documented proprietary extensions

Sub-interfaces defined by the Foundation will use the prefix `fpsf:` in extension keys. Third-party sub-interfaces SHOULD use a reverse-domain prefix to avoid collision.

---

## 10. Non-Goals

This specification intentionally avoids:

- Defining payment authorization or signing schemes
- Defining how the payee system tracks fulfillment
- Defining identity verification or compliance requirements
- Prescribing QR code visual design or wallet UX patterns
- Replacing ISO 20022, SEPA, ERC-681, or any existing standard

---

## 11. Conformance

A paying application is considered **CPD-002-conformant** if:

- It can parse and validate a base `CanonicalPaymentRequest`
- It enforces expiry: requests with `expires_at` in the past are rejected
- It performs derivation as described in Section 6
- It ignores unrecognized `rail_hint` values gracefully rather than erroring
- It never includes payer data in any field of the CPR it transmits or stores

A payee system is considered **CPD-002-conformant** if:

- It generates requests with a unique `request_id` per request
- It sets a bounded `expires_at`
- It serves CPR payloads only over TLS when using the URL Token delivery model
- It does not include payer data in the CPR

---

## Closing Note

The Canonical Payment Request is the missing link between the expression of payment need and the act of payment. By separating the two — by making the request structurally independent of the payer, the rail, and the mechanism — CPD-002 enables payment applications of every kind to participate in the same ecosystem without requiring them to understand each other's internals.

Its value lies in **portability, neutrality, and simplicity**.
