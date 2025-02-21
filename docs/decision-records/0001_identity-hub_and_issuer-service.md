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

# Decision Record: Adding Tractus-X IdentityHub and Tractus-X IssuerService

## Problem statement

The current Tractus-X architecture is dependent on a proprietary credential service ("wallet") for both managing and issuing verifiable credentials.
In the past, the Managed Identity Wallet served as the Tractus-X credential service, but its design as a central managed wallet renders it unsuitable for use as a decentralised credential service.
This renders it impossible to run a Catena-X dataspace using only open-source software.
To remove this dependency, a new open-source credential service and an issuer service that does not rely on proprietary software are needed.
Both must implement the [Decentralized Claims Protocol](https://eclipse-dataspace-dcp.github.io/decentralized-claims-protocol/), which Catena-X has committed to.

## Evaluation criteria

Possible approaches are evaluated based on a set of requirements the issuer service should fulfil.

### Functional

The issuer service should support the following features without delegation to or reliance on external, proprietary services:

- credential creation
- VC signing
- VC revocation
- DID management
- key management
- implement the DCP protocol

### Non-functional

#### Reusability across Manufacturing-X

To enable interoperability with other dataspaces, the issuer service must not be specific to the Catena-X dataspace, but at the very least be reusable across Manufacturing-X.

#### Support for credential services independent of where they are hosted

Credential services in future releases may be hosted by various parties (e.g. operating company, ESP, dataspace member).
Therefor, the issuer service must not rely on the credential service being hosted by any specific entity.

#### Be open for extension

The Tractus-X architecture must be able to support potential new protocols and new credential types in the future without requiring extensive refactoring of the existing code base.

## Possible approaches

Regarding the credential service, the viable option is creating a Tractus-X distribution of the [Eclipse EDC IdentityHub](https://github.com/eclipse-edc/IdentityHub), which is an open-source Eclipse project of a credential service implementing the DCP protocol.
For the issuer service, there are two options:

- continue development on the current [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer)
- replace the current [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer) by a Tractus-X distribution of the [Eclipse EDC Issuer Service](https://github.com/eclipse-edc/IdentityHub) (currently still in the same repository as the Eclipse EDC Identity Hub)

### Option 1: Continue development on the current [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer)

#### Pros

- build on existing code base
- already integrated with portal backend

#### Cons

- delegation of features (signing and revoking VCs, DID and key management) to a proprietary credential service instance
- not open for extension to additional credential types or schemas without a code-level change and a redeployment (causing a service disruption)
- reliance on a central IdP (Keycloak) for issuing credentials
- not compliant with DCP issuance
- security concerns (client VCs are stored)

### Option 2: Introduce a Tractus-X distribution of the [Eclipse EDC Issuer Service](https://github.com/eclipse-edc/IdentityHub)

#### Pros

- supports all features listed in the [functional requirements](#functional) above without delegating their execution to external services
- introducing a Tractus-X distribution of an 
- it is open for extension to other protocols and credential schemas
- make use of the following features which are already implemented in the EDC libraries
  - JSON-LD algorithms
  - cryptographic utilities, keys, signatures
  - all model classes, support for VC DataModel 1.1 _and_ 2.0
  - credential status list services
  - a generic credential generation framework

#### Cons

- adds an additional dependency to the upstream eclipse-edc project
- requires significant integration work on the portal backend

## Decision

A Tractus-X distribution of the [Eclipse EDC IdentityHub](https://github.com/eclipse-edc/IdentityHub) is introduced as an open-source decentralised credential service for the Tractus-X architecture.
A Tractus-X distribution fo the [Eclipse EDC Issuer Service](https://github.com/eclipse-edc/IdentityHub) replaces the current [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer) as the implementation of the Tractus-X issuer.

### Rationale

- continuing development on the [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer) until all requirements listed above are met involves an amount of work essentially equivalent to a rewrite from scratch 
- introducing Tractus-X distributions of upstream dataspace-agnostic projects is essential for interoperability with other dataspaces
- using components from upstream projects as standard dependencies in Tractus-X distributions reduces development, testing, and maintenance effort within Tractus-X

### Actions

Add two new repositories (related PRs [tractusx-identityhub](https://github.com/eclipse-tractusx/.eclipsefdn/pull/117) and [tractusx-issuerservice](https://github.com/eclipse-tractusx/.eclipsefdn/pull/118)) for Tractus-X distributions of the [Eclipse EDC IdentityHub and IssuerService](https://github.com/eclipse-edc/IdentityHub):

- `tractusx-identityhub`
- `tractusx-issuerservice`

Use the relevant components from upstream developments as standard dependencies in the Tractus-X distributions.

Responsible committers for the new repositories as of now are Boris Rizov (@borisrizov-zf) and Rafael Magalhães (@rafaelmag110).

### Implications

From release 25.09 onwards, the current [ssi-credential-issuer](https://github.com/eclipse-tractusx/ssi-credential-issuer) will no longer be needed in the Catena-X architecture and will therefore no longer need to be maintained.
