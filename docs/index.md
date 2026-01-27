# Pseudonyms for the EUDI Wallet

## Background 

Pseudonyms are a mandatory feature of the EUDI Wallet. However, the current legal text restricts pseudonym generation to WebAuthn-based mechanisms. Such pseudonyms cannot support identity-bound pseudonyms, which are important in use cases like account recovery with known offender matching.

The work presented in these pages analyzes the regulatory landscape and technical constraints around pseudonym management for account recovery with known offender matching. The analysis in "Analysis and Alternatives" highlights ambiguities and limitations in the mandated WebAuthn approach and evaluates three candidate designs:

1. **Alias-based pseudonyms**: Simple, user-controlled, and minimizes existing ecosystem disruption. However, they are limited by a finite set of pre-issued pseudonyms and less suitable for high-privacy use cases where site-specificity is required.
2. **Directed pseudonyms**: Offer site-specificity by default, but require a new pseudonym service and support for combined presentations.
3. **ZKP-based pseudonyms**: Leverage ZKP to enable stable and site-specific pseudonyms without requiring new actors. However, these require new wallet capabilities and
verifiers are required to be able to process a ZKP of pseudonym derivation.

All three solution candidates support identity-bound pseudonyms by relying on a pseudonym seed managed by the PID Provider.

## Proposal

Informed by this analysis, "Architecture Decision Record" commits to investigating and prototyping a ZKP-based pseudonym solution within WeBuild. This approach:

- Enables stable, site-specific pseudonyms without introducing new ecosystem actors
- Supports critical use cases like ban enforcement that WebAuthn cannot address
- Provides a foundation for future-proof pseudonym functionality

There are technical and regulatory challenges involved. Empirical evaluation and iterative prototyping in WeBuild can provide insights into real-world tradeoffs, demonstrate feasibility, and provide evidence-based guidance for future regulatory frameworks.

The aim of this work is to develop a conformance specification for WeBuild.