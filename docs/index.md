# Pseudonyms for the EUDI Wallet

## Background

Pseudonyms are a mandatory feature of the EUDI Wallet. However, the current legal text restricts pseudonym generation to WebAuthn-based mechanisms. Such pseudonyms cannot support identity-bound pseudonyms, which may be important in use cases like account recovery or account creation.

??? info "Noteworthy source in the legal text and additional reading"

    Articles 5 and 5b(9) of [Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2024/1183/oj) state that the use of pseudonyms cannot be prohibited unless identification is required by Union or national law. Pseudonym management is a mandatory EUDIW function under Article 5a(4)(b). In addition, [CIR 2024/2979 Article 14](https://eur-lex.europa.eu/eli/reg_impl/2024/2979/oj/eng#art_14) mandates WebAuthn as the pseudonymization solution.

    However, growing recognition that WebAuthn does not cover all pseudonym use cases has prompted calls for alternative approaches.

The work presented in these pages analyzes the regulatory landscape and technical constraints around pseudonym management for account recovery with support for known offender matching.

??? info "Known offender matching"

    Known offender matching is a method used to secure account creation by preventing ban evasion. When a user is banned for misconduct, the service aims to block re-registration under a new identity or pseudonym. The process of checking whether a registering user has previously been banned is referred to as known offender matching.

The analysis in [Analysis and Alternatives](analysis-alternatives.md) highlights ambiguities and limitations in the mandated WebAuthn approach and evaluates three candidate designs:

1. **Alias-based pseudonyms**: Simple, user-controlled, and minimizes existing ecosystem disruption. However, they are limited by a finite set of pre-issued pseudonyms and less suitable for high-privacy use cases where site-specificity is required.
2. **Directed pseudonyms**: Offer site-specificity by default, but require a new pseudonym service and support for combined presentations.
3. **ZKP-based pseudonyms**: Leverage ZKP to enable stable and site-specific pseudonyms without requiring new actors. However, these require new wallet capabilities and
verifiers are required to be able to process a ZKP of pseudonym derivation.

All three solution candidates support identity-bound pseudonyms by relying on a pseudonym seed managed by the PID Provider.

## Proposal

Informed by this analysis, [Architecture Decision Record](adr-pseudonyms.md) commits to investigating and prototyping a ZKP-based pseudonym solution within WeBuild. This approach:

- Enables stable, site-specific pseudonyms without introducing new ecosystem actors
- Supports critical use cases like ban enforcement that WebAuthn cannot address
- Provides a foundation for future-proof pseudonym functionality

There are technical and regulatory challenges involved. Empirical evaluation and iterative prototyping in WeBuild can provide insights into real-world tradeoffs, demonstrate feasibility, and provide evidence-based guidance for future regulatory frameworks.

The aim of this work is to develop a [Conformance Specification](cs-NN-pseudonyms.md) for WeBuild.
