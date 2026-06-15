# CHANGELOG — CPD Specification Family

All notable changes to the Canonical Payment Definition (CPD) specification family are recorded here.

This log follows [Semantic Versioning 2.0.0](https://semver.org).

---

## FPSF-CPD-002 — Canonical Payment Request Definition

### [1.0.0] — 2026-06-15

**Status:** Draft  
**Author:** Adalton Reis \<reis@fabricpaymentstandards.org\>

**Summary:** Initial release of CPD-002. Defines the Canonical Payment Request (CPR) as a payer-agnostic, rail-neutral structure from which a CPD-001 Payment may be derived.

**Introduced:**

- Base `CanonicalPaymentRequest` interface: `cpr_version`, `request_id`, `created_at`, `expires_at`, `payee`, `value`, `memo`, `rail_hint`, `extensions`
- `PayeeDescriptor` and `RequestedValue` / `AssetDescriptor` structures
- `RailHint` enumeration: `ON_CHAIN`, `SEPA_CT`, `SEPA_INST`, `ACH`, `PIX`, `UPI`, `SWIFT`, `OPEN`
- Delivery model: QR code (`fpsf-cpr://` URI scheme), deep link, URL Token (ephemeral session), and direct JSON
- Ephemeral token requirements: max 15-minute lifetime, single-use, TLS-only
- Domain-specific sub-interfaces: `OnChainPaymentRequest` (SS-001 compatible), `IBANPaymentRequest`, `AccountPaymentRequest`, `InstantKeyPaymentRequest`
- Derivation interface (Section 6): formal mapping from CPR + `DerivationContext` to CPD-001 `Payment`
- Invariants section (Section 8): six invariants including no-payer-data rule
- Extensibility conventions: `fpsf:` prefix for Foundation extensions, reverse-domain prefix for third parties
- Conformance requirements for paying applications and payee systems

---

## FPSF-CPD-001 — Canonical Payment Definition

### [1.1.0] — 2026-06-15

**Status:** Draft  
**Author:** Adalton Reis \<reis@fabricpaymentstandards.org\>  
**Backward-compatible with:** v1.0.0

**Summary:** Minor update to bridge CPD-001 with the newly introduced CPD-002. All v1.0.0 structures, interfaces, and invariants are preserved unchanged.

**Added:**

- Section 2.3: `Payment Request` definition — brief normative description establishing the separation between Payment and Payment Request
- Section 3.7: `PaymentRequestReference` optional field on the `Payment` object — enables traceability when a payment originates from a CPR
- Section 5.7: `derivePayment` interface — references the derivation procedure defined in CPD-002 Section 6
- Section 9: Relationship to FPSF-CPD-002 — table and prose making the CPD-001 / CPD-002 boundary explicit
- Scope section updated: explicitly notes that Payment Request structure is out of scope for CPD-001
- Non-Goals section updated: explicitly excludes Payment Request definition
- Conformance section updated: notes optional CPD-002 conformance path
- Abstract updated: references CPD-002 as the companion specification

**Changed:**

- `Payment` object in Section 3.3: added optional `derived_from: PaymentRequestReference` field

**No breaking changes.** Implementations conforming to v1.0.0 remain conformant to v1.1.0 without modification.

---

### [1.0.0] — 2026-03-25

**Status:** Draft  
**Author:** Adalton Reis \<reis@fabricpaymentstandards.org\>

**Summary:** Initial release. Canonical Payment and Digital Payment definitions, lifecycle, interfaces, and invariants.

**Introduced:**

- `Payment` and `Digital Payment` definitions
- `Participant`, `Value`, `Payment` object, `IntentDescriptor`, `AuthorizationProof`, `ExecutionContext` structures
- `PaymentState` lifecycle: `CREATED`, `AUTHORIZED`, `IN_FLIGHT`, `SETTLED`, `FAILED`, `CANCELLED`
- State transition constraints and monotonicity invariant
- Conceptual interfaces: `initiatePayment`, `authorizePayment`, `submitPayment`, `settlePayment`, `getPaymentStatus`, `cancelPayment`
- Six invariants: uniqueness, integrity, authorization binding, deterministic state, finality
- Extensibility section
- Conformance section