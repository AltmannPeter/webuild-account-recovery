# WE BUILD - Conformance Specification: Pseudonyms for Account Recovery and Known Offender Matching

Version 0.1

Table of Content:

- [Introduction](#1-introduction)
- [Scope](#2-scope)
- [Normative Language](#3-normative-language)
- [Security Properties and Requirements](#4-security-properties-and-requirements)
- [Roles and Components](#5-roles-and-components)
- [Protocol Overview](#6-protocol-overview)
- [High-level Flows](#7-high-level-flows)
- [Normative Requirements](#8-normative-requirements)
- [Interfaces](#9-interfaces)
- [Security Considerations](#10-security-considerations)
- [Conformance](#11-conformance)

## 1. Introduction

This document defines the **WeBuild Consortium Conformance Specification (CS)** for Zero-Knowledge Proof (ZKP) based pseudonyms for digital identity, based on the decision recorded in WE BUILD [ADR Pseudonyms](adr-pseudonyms.md).

The specification addresses limitations in the mandated WebAuthn-based pseudonym approach by enabling:

- **Stable pseudonyms**: Recoverable across wallet devices
- **Site-specificity**: Pseudonyms unique to each relying party or service
- **Identity binding**: Cryptographically tied to the identity subject
- **Known offender matching**: Enables ban enforcement without identity disclosure
- **Rate-limiting**: Verifiable limits on pseudonym creation per site/service
- **Unlinkability**: Pseudonyms appear unrelated across different services

This specification defines how Wallet Units, Pseudonym Managers, and Verifiers interoperate to enable privacy-preserving account recovery and known offender matching capabilities.

## 2. Scope

This specification defines:

- Requirements for embedding pseudonym seed commitments in attribute attestations
- ZKP-based pseudonym derivation
- ZKP-based pseudonym presentation protocols
- Verification procedures for pseudonym proofs
- Support for rate-limited, site-specific, and identity-bound pseudonyms

**Out of Scope**:

- Non-ZKP pseudonym approaches (alias-based, directed pseudonyms without ZKP)
- WebAuthn-based pseudonyms
- Pseudonyms that do not support unmasking by the pseudonym manager
- General attribute attestation issuance and presentation (covered in separate CS)

## 3. Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

## 4. Security Properties and Requirements

This specification ensures the following security properties and requirements:

- **Soundness**: Users SHALL be able to create and present pseudonyms only when controlling a valid attestation containing a pseudonym seed commitment. Verifiers SHALL be able to verify rate-limits on pseudonym generation.
- **Unforgeability**: It SHALL NOT be possible to create a pseudonym unlinked to a specific attestation holding the user's pseudonym seed.
- **Unlinkability**: Pseudonyms SHALL appear unrelated to other pseudonyms belonging to the same user across different services.
- **Rate-limitation**: Verifiers SHALL be able to declare and verify upper limits on site-specific and/or service-specific pseudonyms.
- **Secrecy**: The pseudonym seed value SHALL NOT be disclosable. Attestations SHALL store cryptographic commitments to the seed rather than cleartext values.
- **Transferability**: The pseudonym seed SHALL be recoverable when the previous wallet instance or attestation becomes unavailable.

## 5. Roles, and Components

This specification uses the following roles and components:

**Roles**:

- User (holder, prover): The identity subject who controls the Wallet Unit and the attestation from which pseudonyms are derived.
- Wallet Unit (WU): The EUDIW application that:
    - Requests, retrieves, and stores attestations containing pseudonym seed commitments
    – Generates ZKPs of pseudonym derivation
    – Presents derived pseudonyms to verifiers
    – Holds hardware-protected keys for proof-of-possession binding
- **Pseudonym Manager (Issuer)**: The entity that:
    - Generates distinct high-entropy pseudonym seeds for each identity subject
    - Issues attestations containing cryptographic commitments to those seeds
    - Retains capability to unmask pseudonyms when legally required
- **Verifier (Relying Party, Service Provider)**: An entity that:
    - Receives pseudonym presentations with associated ZKP proofs
    - Verifies pseudonym validity and rate-limiting properties
    - Enforces service-specific policies (e.g., known offender matching during account creation)
    - Provides services (e.g., account recovery) based on verified pseudonyms

**Components**:

- **Attestation ((Q)EAA, PID, Credential)**: A set of attributes, signed by the issuer, issued to the user, and presented to the verifier.
- **Pseudonym Seed**: A high-entropy, user-specific value from which site-specific pseudonyms are derived.
- **Pseudonym Seed Commitment**: A cryptographic commitment to the pseudonym seed, included as an attribute in the attestation.
- **ZKP of Pseudonym Derivation**: A zero-knowledge proof demonstrating correct pseudonym derivation without revealing the seed.

## 6. Protocol Overview

A ZKP-based pseudonym system operates in three main phases, involving the Pseudonym Manager (Issuer), the Wallet Unit (WU), and the Verifier:

**Seed issuance**:

1. The Pseudonym Manager generates a unique, high-entropy seed, `pns` for the user
2. The Pseudonym Manager creates a cryptographic binding to `pns`
3. The issuer embeds the binding in an attribute attestations, signs, and issues it

??? danger "Disclosure of the pseudonym seed"

    It is critical for user privacy that the pseudonym seed is never revealed. The binding in step 2 mitigates accidental disclosure (e.g., if the full attestation is exposed). By including a commitment to the seed rather than embedding the `pns` value directly, the Wallet Unit cannot disclose the seed as the binding is opened within the ZKP circuit during proof generation. This design shifts responsibility to the ZKP layer, which must prove properties over the committed seed without revealing it. It also requires the user's wallet to store the pseudonym seed (out of scope for this text).

??? info "The suggested SE approach"

    In Sweden, the pseudonym seed is proposed to be derived using a keyed PRF over user-specific data. Specifically, the current suggestion is for the pseudonym manager to compute the seed as `PRF(secret_key, personal_id_number)`. While personal identity numbers can change, such cases are rare, making this method sufficiently stable for seed derivation. Ultimately, the pseudonym manager is free to choose any input and method for generating the seed.

    Alternatives include having the issuer provide a pseudonym seed, and safe-keeping it.

**Pseudonym generation**:

- The user visits a site or service to access with a pseudonym
- The Wallet Unit derives a pseudonym specific to the site or service using the pseudonym seed (`pns`) a scope (`scp`) and index (`idx`): `nym = PRF(key=pns, data=scp||idx)`
- The Wallet Unit generates a ZKP proving correct derivation

??? info "ZKP disclosure of pseudonym"

    During presentation, the user would provide a ZKP for a statement such as: "I possess a valid issuer-signed attestation containing a hash binding `H` to a pseudonym seed. I know the preimage `pns` of `H`, and the presented pseudonym `nym` is derived by hashing `pns` concatenated with a scope `scp` and index `idx`.” 

**Pseudonym presentation**:

TEXT NEEDED ON HOW TO USE OID4VP TO PRESENT A ZKP PSEUDONYM. NO DETAILS HERE AS THESE WILL BE ADDED IN FLOW DESCRIPTION LATER.

**Pseudonym verification**:

The verifier:

- Checks that the presented pseudonym is derived from a valid issuer signed attestation
- Verifies that the correct scope (`scp`) was used
- Verifies rate-limiting with the index (`idx`) value
- Runs the verification circuit to validate the proof and public inputs
- Accepts or rejects service provisioning based on policy (e.g., known offender list)

## 7. High-level Flows

### 7.1. Pseudonym Seed Generation and Issuance Flow

Actors:

Preconditions:

Flow:

### 7.2. Pseudonym Generation and Presentation Flow

Actors:

Preconditions:

Flow:

### 7.3. Pseudonym Verification Flow

Actors:

Preconditions:

Flow:

### 7.4. Known Offender Matching Flow

Actors:

Preconditions:

Flow:

### 7.5. Account Recovery Flow

Actors:

Preconditions:

Flow:

## 8. Normative Requirements

### 8.1 Pseudonym Manager (Issuer) Requirements

### 8.2 Wallet Unit Requirements

### 8.3 Verifier (Relying Party) Requirements

### 8.4 Cryptographic Requirements

## 9. Interfaces

### 9.1 Pseudonym Seed Commitment Format

### 9.2 ZKP Proof Format

### 9.3 Pseudonym Presentation Format

## 10. Security Considerations

## 11. Conformance

### 11.1 Test Vectors
