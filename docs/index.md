# Pseudonyms for the EUDI Wallet

## Background

Pseudonyms are a required feature of the EUDI Wallet. The Wallet must support user authentication via pseudonyms when the service provider does not require full identification. This enables service delivery without unnecessary disclosure of the user's identity.

As of ARF 2.8.0, identity-bound pseudonyms are explicitly supported, enabling use cases such as account recovery and fraud prevention. In contrast, the proposed CIR updates only support WebAuthn-based pseudonyms.

??? note "Pseudonyms in ARF 2.8.0"

    TLDR; Sections 2.5 and 4.7 of the ARF could be improved, but no changes are necessary to support the use case proposed for testing in WE BUILD.
    
    The [ARF Section 2.5](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md#25-pseudonyms) defines the purpose of pseudonyms as enabling service providers to recognize users across sessions within a defined scope, while avoiding unnecessary identification. The text herein proposes a broader, less restrictive, definition of pseudonyms.

    The ARF now recognizes that pseudonym requirements vary by use case. Annex 2 [Topic 11](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2.02-high-level-requirements-by-topic.md#a238-topic-11---pseudonyms) lists 4: A and B involve passkey-based authentication (with unclear PID-pseudonym binding in B), C introduces rate-limiting, and D requires pseudonym reuse across services. These map to three pseudonym types in [Section 4.7](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md#47-possible-implementations-of-pseudonyms). Overall, the distinction between types, use cases, and pseudonym features is not fully clear.

    Section 2.5 indicates further technical details will follow for use cases B–D. These are most relevant to our needs. Section 4.7.3 confirms support for identity-bound attestations, and calls for definitions to be provided by attestation providers via Rulebooks.

??? info "Noteworthy source in the legal text and additional reading"

    Articles 5 and 5b(9) of [Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj) state that the use of pseudonyms cannot be prohibited unless identification is required by Union or national law. Pseudonym management is a mandatory EUDIW function under Article 5a(4)(b). In addition, [CIR 2024/2979 Article 14](https://eur-lex.europa.eu/eli/reg_impl/2024/2979/oj/eng#art_14) mandates WebAuthn as the pseudonymization solution.

    A recent proposed update to the implementing acts clarifies the mandate for WebAuthn but omits any mention of additional pseudonym types. This creates a potential misalignment between ARF 2.8.0 and the draft legal text.

An earlier analysis ([Analysis and Alternatives](analysis-alternatives.md)) reviewed both the regulatory landscape and design space for enabling identity-bound pseudonyms that support account recovery and known offender matching.

??? note "Account recovery"

    Account recovery is the process of regaining access to a user account when the primary login mechanism is no longer available. Identity-bound pseudonyms offer significant benefits in this context: they give users a robust tool for initiating recovery and provide service providers with a secure, verifiable basis for managing the process.

??? note "Known offender matching"

    Known offender matching is a method used to secure account creation by preventing ban evasion. When a user is banned for misconduct, the service aims to block re-registration under a new identity or pseudonym. The process of checking whether a registering user has previously been banned is referred to as known offender matching.

The analysis identified key limitations in the mandated WebAuthn approach and evaluated three alternative designs:

1. **Alias-based pseudonyms**: Simple, user-controlled, and minimizes existing ecosystem disruption. However, they are limited by a finite set of pre-issued pseudonyms and less suitable for high-privacy use cases where site-specificity is required.
2. **Directed pseudonyms**: Offer site-specificity by default, but require a new pseudonym service and support for combined presentations.
3. **ZKP-based pseudonyms**: Leverage ZKP to enable stable and site-specific pseudonyms without requiring new actors. However, these require new wallet capabilities and
verifiers are required to be able to process a ZKP of pseudonym derivation.

Despite their differences, all three solutions share a common foundation: they rely on a pseudonym seed issued by a pseudonym manager (e.g., the PID Provider). This seed is embedded in an attestation and serves two purposes: it proves that a real person is behind the pseudonym (proof of humanhood), and it allows (without enforcing) the pseuodnym manager to unmask the pseudonym if required (e.g., in legal disputes or fraud investigations). Solutions that do not support such unmasking are considered out of scope.

## Proposal

Following the analysis, [Architecture Decision Record](adr-pseudonyms.md) commits to investigating and prototyping a ZKP-based pseudonym solution within WeBuild. This approach:

- Enables site-specific (and optionally service-specific) pseudonyms without introducing new ecosystem actors.
- Derives pseudonyms that are unlinkable across sites and services and do not leak any information about the identity subject.
- Enables rate-limited pseudonyms that enable restricting the number of pseudonyms a person can create per site and/or service.
- Enables stable pseudonyms that are recoverable even if the user changes their wallet device.
- Derives pseudonyms that can only be used by the identity subject controlling the proof-of-possession (PoP) key in the attestation.
- Supports critical use cases like ban enforcement.
- Provides a foundation for future-proof pseudonym functionality.

There are both technical and regulatory challenges involved. Empirical evaluation and iterative prototyping in WeBuild can provide insights into real-world tradeoffs, demonstrate feasibility, and provide evidence-based guidance for future regulatory frameworks.

The aim of this work is to develop a [Conformance Specification](cs-NN-pseudonyms.md) for WeBuild.
