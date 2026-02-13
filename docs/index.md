# Pseudonyms for the EUDI Wallet

## Background

Pseudonyms are a required feature of the EUDI Wallet. The Wallet must support user authentication via pseudonyms when the service provider does not require full identification. This enables service delivery without unnecessary disclosure of the user's identity.

As of ARF 2.8.0, identity-bound pseudonyms are explicitly supported, enabling use cases such as account recovery and fraud prevention. In contrast, the proposed CIR updates only support WebAuthn-based pseudonyms.

??? note "Pseudonyms in ARF 2.8.0"

    TLDR; Sections 2.5 and 4.7 of the ARF could be improved, but no changes are necessary to support the use case proposed for testing in WE BUILD.
    
    The [ARF Section 2.5](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md#25-pseudonyms) defines the purpose of pseudonyms as enabling service providers to recognize users across sessions within a defined scope, while avoiding unnecessary identification. The text herein proposes a broader, less restrictive, definition of pseudonyms.

    The ARF now recognizes that pseudonym requirements vary by use case. Annex 2 [Topic 11](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2.02-high-level-requirements-by-topic.md#a238-topic-11---pseudonyms) lists 4: A and B involve passkey-based authentication (with unclear PID-pseudonym binding in B), C introduces rate-limiting, and D requires pseudonym reuse across services. These map to three pseudonym types in [Section 4.7](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md#47-possible-implementations-of-pseudonyms). Overall, the distinction between types, use cases, and pseudonym features is not fully clear.

    Section 2.5 indicates further technical details will follow for use cases B–D. These are most relevant to our needs. Section 4.7.3 confirms support for identity-bound attestations, and calls for definitions to be provided by attestation providers via Rulebooks.

??? info "The legal text"

    Articles 5 and 5b(9) of [Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj) state that the use of pseudonyms cannot be prohibited unless identification is required by Union or national law. Pseudonym management is a mandatory EUDIW function under Article 5a(4)(b). In addition, [CIR 2024/2979 Article 14](https://eur-lex.europa.eu/eli/reg_impl/2024/2979/oj/eng#art_14) mandates WebAuthn as the pseudonymization solution.

    A recent proposed update to the implementing acts clarifies the mandate for WebAuthn but omits any mention of additional pseudonym types. This creates a potential misalignment between ARF 2.8.0 and the draft legal text.

It remains unclear which pseudonym use cases will be supported in the ARF or the implementing acts. Neither source defines what constitutes a pseudonym, leading to conflicting interpretations based on varying use case requirements.

Here, a pseudonym is defined as an attribute representing a specific identity subject. Unlike other attributes that disclose information about the subject, a pseudonym is designed to prevent such disclosure.

Crucially, a pseudonym is not a single identifier type. Arguably, it is better to view pseudonyms as part of a *set of identifier types*, each configured to meet specific use case needs. As such, pseudonyms must be understood contextually and are meaningful only when configured to align with their intended use case. A practical approach is to begin with use case requirements and then define a corresponding pseudonym configuration.

## Toward Identity-bound Pseudonyms

Pseudonyms are often designed to meet specific operational requirements. In account management, two principal concerns are account recovery and known-offender matching.

**Account recovery** involves regaining access when the user no longer controls any primary login mechanism. This requires a stable, device-independent pseudonym. Identity-bound pseudonyms meet this need by allowing users to initiate recovery and giving service providers a secure and verifiable anchor for the process.

**Known-offender matching** secures account creation by preventing ban evasion. When a user is banned for misconduct, the system must detect and block re-registration under a new identity or pseudonym.

It remains unclear how a WebAuthn-based pseudonym could support either account recovery or known-offender matching. A solution that meets these requirements likely needs to be identity-bound. This proposal introduces a design based on a *pseudonym seed* issued by a pseudonym manager (e.g., a PID Provider). A commitment to the seed is embedded in an attestation and serves as input to a derivation function that generates the user's pseudonym. To preserve anonymity and avoid reliance on external services, the user generates a zero-knowledge proof (ZKP) of the pseudonym derivation locally on their device.

??? note "Pseudonym derivation"

    The pseudonym is computed via a PRF. The PRF takes as input the user's pseudonym seed, `pns`, a high entropy secret random value. The PRF additional takes a context, `ctx`, and index, `idx`, as input and computes: `pseudonym = PRF(pns, ctx, idx)`. This construction supports the derivation of structurally distinct pseudonyms under different operational scenarios:
    
    1. Directed pseudonyms: The service provider sets the context to the service identifier. To support known-offender matching, the service provider sets the index to a fixed value (e.g., `0`).
    2. Cross-service pseudonyms: Multiple service providers can share context and index to enable linkability across services.
    3. Alias-based pseudonyms: The user selects the index to define a consistent alias.

    Future extensions may include user- or device-supplied inputs to enable features such as device binding or unmaskability.

Based on the use case requirements above, the proposal is developed across two documents. The [Architecture Decision Record](adr-pseudonyms.md) commits to investigating and prototyping a ZKP-based pseudonym solution within WE BUILD, while the [Conformance Specification](cs-NN-pseudonyms.md) defines its implementation details.
