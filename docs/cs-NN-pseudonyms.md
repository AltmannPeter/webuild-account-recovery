# WE BUILD - Conformance Specification: Pseudonyms for Account Recovery and Known Offender Matching

Version 0.1

Table of Content:

- [Introduction](#1-introduction)
- [Scope](#2-scope)
- [Normative Language](#3-normative-language)
- [Security Properties and Requirements](#4-security-properties-and-requirements)
- [Roles and Components](#5-roles-and-components)
- [The Pseudonym Seed](#the-pseudonym-seed)

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

## 5. Roles and Components

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

**The Pseudonym Manager**:

1. Generates a unique, high-entropy seed, `pns` for the user
2. Embeds a cryptographic binding to `pns` in an attribute attestation
3. Issues the attestation

??? danger "Disclosure of the pseudonym seed"

    It is critical for user privacy that the pseudonym seed is never revealed. The binding in step 2 mitigates accidental disclosure (e.g., if the full attestation is exposed). By including a commitment to the seed rather than embedding the `pns` value directly, the Wallet Unit cannot disclose the seed as the binding is opened within the ZKP circuit during proof generation. This design shifts responsibility to the ZKP layer, which must prove properties over the committed seed without revealing it. It also requires the user's wallet to store the pseudonym seed (out of scope for this text).

**The Wallet Unit**:

- The user visits a site or service to access with a pseudonym
- The Wallet Unit derives a pseudonym specific to the site or service using: `nym = PRF(key=seed, data=scope||index)`
- The Wallet Unit generates a ZKP proving correct derivation

???+ info "ZKP disclosure of pseudonym"

    During presentation, the user would provide a ZKP for a statement such as: "I possess a valid issuer-signed attestation containing a hash binding `H` to a pseudonym seed. I know the preimage `pns` of `H`, and the presented pseudonym `nym` is derived by hashing `pns` concatenated with a scope `scp` and index `idx`.” 

## Pseudonym Generation [WIP]

EVERYTHING BELOW IS WIP

### Fields

In the attribute attestation:

- `pns`: A cryptographic commitment to the pseudonym seed, a high entropy and unique value specific to each user.

In the pseudonym presentation:

- `scp`: The scope that details the verifier identifier or the service identifier.
- `idx`: The index value used for rate-limiting pseudonyms.

### The Pseudonym Seed

??? info "The suggested SE approach"

    In Sweden, the pseudonym seed is proposed to be derived using a keyed PRF over user-specific data. Specifically, the current suggestion is for the pseudonym manager to compute the seed as `PRF(secret_key, personal_id_number)`. While personal identity numbers can change, such cases are rare, making this method sufficiently stable for seed derivation. Ultimately, the pseudonym manager is free to choose any input and method for generating the seed.

    Alternatives include having the issuer provide a pseudonym seed, and safe-keeping it.

### Pseudonym Derivation

When presenting a pseudonym, the pseudonym seed is used as a key for a PRF applied to a scope and index to create a directed pseudonym: `nym = PRF(key=pns, data=scp|index)`.

## Test Procedure

### Pseudonym Seed Derivation

- Personal id number: `"19080101-1234"`
- Secret key: `"ABC123"`
- pns: `""`
