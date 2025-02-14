<!--
#######################################################################

Tractus-X - Special Interest Group (SIG) Architecture

Copyright (c) 2025 Contributors to the Eclipse Foundation

See the NOTICE file(s) distributed with this work for additional
information regarding copyright ownership.

This work is made available under the terms of the
Creative Commons Attribution 4.0 International (CC-BY-4.0) license,
which is available at
https://creativecommons.org/licenses/by/4.0/legalcode.

SPDX-License-Identifier: CC-BY-4.0

#######################################################################
-->

# Decision Proposal: Adding Tractus-X IdentityHub and Tractux-X IssuerService

## Proposed decision

Add two new repositories for Tractus-X distributions of the [Eclipse EDC IdentityHub and IssuerService](https://github.com/eclipse-edc/IdentityHub):

- `tractusx-identityhub`
- `tractusx-issuerservice`

From release 25.09 onwards, the current [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer) will no longer be needed in the Catena-X architecture and will therefore no longer need to be maintained.

Responsible committers for the new repositories as of now are Boris Rizov (@borisrizov-zf) and Rafael Magalhães (@rafaelmag110).

## Rationale

Going forward, the Tractus-X IdentityHub distribution (based on [EDC IdentityHub](https://github.com/eclipse-edc/IdentityHub)) will serve as the open-source basis for credential services ("wallets") in Tractus-X.
In the same manner, the Tractus-X Issuer Service will replace the current [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer) component.
Therefore, Tractus-X distributions of these upstream components are needed, which will be developed and maintained in the aforementioned Tractus-X repositories.

The current [SSI Credential Issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer) has several drawbacks:

- a hard dependency on a proprietary wallet instance for basic features such as creating, signing and revoking verifiable credentials, DID document management and key management.
- adding additional credential types or schemas requires a code-level change and a redeployment (service disruption)
- reliance on central IdP (Keycloak) for issuing credentials
- security concerns (client credentials are stored)
- not compliant with DCP (Issuance)

## Approach

Provision two new repositories (related PRs [tractusx-identityhub](https://github.com/eclipse-tractusx/.eclipsefdn/pull/117) and [tractusx-issuerservice](https://github.com/eclipse-tractusx/.eclipsefdn/pull/118)), and create Tractus-X distributions of the aforementioned upstream components.

Upstream developments will automatically filter down into the Tracts-X distributions, thus limiting the development, testing and maintenance surface significantly.

## Possible alternatives

Theoretically, it would be possible to continue development of the `ssi-credential-issuer` component until all of the aforementioned drawbacks are addressed.
However, doing so would amount to an almost complete rewrite of the application due to significant architectural differences, while additionally suffering from the following drawbacks:

- duplicate all development and testing efforts around DCP that are _already implemented upstream,_ specifically:
  - JSON-LD algorithms (Note that this has nothing to do with the proof type of credentials, JSON-LD is simply required by the DCP endpoints.)
  - cryptographic utilities, keys, signatures
  - all model classes, support for VC DataModel 1.1 _and_ 2.0
  - credential status list services
  - a generic credential generation framework
- devise, implement and test a complete extensibility framework
- implement, test, and maintain an admin API to be consumed by portal

Given the development and maintenance efforts that would be necessary, refactoring the `ssi-credential-issuer` is not a viable alternative.
