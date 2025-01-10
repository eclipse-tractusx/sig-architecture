<!--
#######################################################################

Tractus-X - Special Interest Group (SIG) Architecture

Copyright (c) 2024, 2025 Contributors to the Eclipse Foundation

See the NOTICE file(s) distributed with this work for additional
information regarding copyright ownership.

This work is made available under the terms of the
Creative Commons Attribution 4.0 International (CC-BY-4.0) license,
which is available at
https://creativecommons.org/licenses/by/4.0/legalcode.

SPDX-License-Identifier: CC-BY-4.0

#######################################################################
-->

<div align="center">
  <img alt="Version:  v0.1" src="https://img.shields.io/badge/Version-v0.1-blue?style=for-the-badge">
  <img alt="STATUS: RECOMMENDED" src="https://img.shields.io/badge/Status-RECOMMENDED-FF00C9?style=for-the-badge">
  <br>
  <h1> Connect & Exchange Dataspace Usage Pattern </h1>
</div>

## Metadata

|**Version**| **Created at** | **Last Reviewed at** |
|-|-|-|
| v0.1 | Dec, 17th, 2024 | Jan, 9th, 2025 |

|**Type**|**Category**|
|-|-|
| Usage Pattern | Dataspace Adoption |

|**Objective**|
|-|
| Reduce complexity, Increase Performance |

|**Dependencies/Target Group**|
|-|
| [Tractus-X Eclipse Dataspace Connector](), Any Tractus-X Consumer Application (DPP, DCM, IRS, PURIS, etc...) |

### Authors & Reviewers

<!--Add yourself to the list if you review/contribute to this document-->

| Name                  | Company | GitHub                                     | Role                                    |
| --------------------- | ------- | ------------------------------------------ | --------------------------------------- |
| Lars Geyer-Blaumeiser | Cofinity-X | [@lgblaumeiser](https://github.com/lgblaumeiser) | Architect Enablement Services & Eclipse Tractus-X Committer |
| Mathias Brunkow Moser | Catena-X e.V. | [@matbmoser](https://github.com/matbmoser) | Chief Software Architect & Eclipse Tractus-X Project Lead |
| Andrea Bertagnolli  | Think It | [@ndr-brt](https://github.com/ndr-brt)  | Software Engineer & Eclipse Tractus-X Committer                |
|                       |         |                                            |                                         |

## Index

- [Metadata](#metadata)
  - [Authors \& Reviewers](#authors--reviewers)
- [Index](#index)
- [1.  Context](#1--context)
- [2. The pattern](#2-the-pattern)
  - [2.1 Name](#21-name)
  - [2.2 Description](#22-description)
  - [2.3 Story line](#23-story-line)
    - [2.3.1 Prerequisites](#231-prerequisites)
    - [2.3.2 Connect Phase](#232-connect-phase)
    - [2.3.3 Exchange Phase](#233-exchange-phase)
- [3. Potential Users](#3-potential-users)
- [4. Advantages](#4-advantages)
- [5. Downsides](#5-downsides)
- [6. Disclaimers](#6-disclaimers)
- [NOTICE](#notice)

## 1.  Context

This data space usage pattern comes from the idea of:

> You don't need to understand the HTTP protocol to use the internet as an application

The same applies to:

> You don't need to understand the DSP protocol to use the Tractus-X network as an application

Nowadays, there is a big overhead at the "Application" layer from Tractus-X. Every application is using the Eclipse Dataspace Connector (EDC) in their own way. In case of Tractus-X we are using the [Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc).
The problem is that there is no harmonization on the "contract" management in between the consumer and provider applications. At the moment contract agreements are being opened for every data retrieval and they are not being reused, creating a overhead at the [Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc) gateway.

There is missing a common understanding on how to manage the open connections, i.e., where finalized contract agreements exist. This is causing that data retrievals from the Tractus-X Applications take a significant ammount of time. This is due because all use case application developer need to understand the EDC interactions without much guidance on favorable patterns that would improve efficiency of the exchange. Therefore, there is a clear need for showing data consumer applications how they should implement the retrieval of data and the management of existing connections.

The "EDR" (Endpoint Data Reference) Interface has been built for simplifying the negotiation and transfer process for the application. It has been introduced with the Jupiter release from Catena-X, however there is no usage pattern recommendation yet provided for application developer.



Another issue to consider is, that there is still some need to optimize the connector which will lead to changing patterns in the interaction with the connector. The proposed pattern helps in this regard as well, as it simplifies the interactions between application and connector. As a consequence, adapting to future changes will get simpler and maintainability of the application is improved.

In the end from the application side there is only two things that matter when consuming information:

- Which is the "Dataplane URL"
- Which authorization token I need.

The rest can be abstracted, and this is going to be defined here in this usage pattern which could be implemented in a reference implementation or at any application.

![Connect and Exchange Pattern Simple](./media/connect&exchange-context.svg)


## 2. The pattern

**Architecture Design Patterns Used**: Adapter + Proxy

### 2.1 Name

<h1>Connect & Exchange</h1>

*Name Description*: It is so simple to use Tractus-X, you don't need to understand how the policy agreement works, just send your conditions and the application you want to talk with, connect, and exchange data then until any condition changes (detailed below in 2.3.2 [Step 5]).

>**Note**: Alternative names can be still discussed

### 2.2 Description

The Eclipse Dataspace Connector is an "Infrastructure Enabler", i.e., it enriches the ability to provide or consume data from another party with data sovereignty. This is achieved by entering a legally binding contract with proven participants into the exchange. It also allows providers to offer data to dataspace participants not known to him with defined policies and to also restrict the access to data to a defined subset of the dataspace participants. Once a contract is established, the consumer can access the data until the contract terminates, i.e., a continuous data connection is established between services that reside behind the connector which they can make use of until the contract expires.

The common practise to renegotiate a contract for the same asset multiple times is not necessary and just makes the whole effort to consume data more time consuming and creates lots of redundant legally binding contracts.

![Connect and Exchange EDC](./media/edc-connector.svg)


The only thing that it creates is a "PIPE" in between the companies and the connection remains open.

![Connect and Exchange Pipe](./media/edc-connector-infoflow.svg)

The new implementation of the EDR interface gets rid of the prior need for a callback to retrieve the access token for retrieving the data from the provider. This was often a problem because the applications always required a "DNS resolvable domain" for the EDR token to be call backed into the application. But with the EDR interface now the application can be deployed, e.g., in a private cloud setting at the consumer and still can interact with the EDC without any backchannel that would require, e.g., a firewall hole to make the application reachable from the EDC.

There was effectuated several proofs of concepts using this pattern, and it works perfect! The point is that the negotiation does not need to be redone every time, if the data "pipe" is open the data exchange can flow, at least until one from the following conditions change:

- New/change of policies to be accepted were defined by the consumer
- A different connector assets wants to be accessed
- The contract has an expiration time and that has been reached.

As soon as the contract negotiation is finalized, the consumer connector based on identifiers related to the contract receive the access token via the EDR interface. Token renewal is done automatically for him, if the available token expires. ```

So the connection keeps open and is really **FAST** (just depends on the content weight and the HTTP overhead).

In the end the level of abstraction is to say, as a consumer, I want to be able to call a "edc library service", telling my conditions Policies, Asset to Connect, and then I can make "do_post" or "do_get" and all the connection details will be abstracted.

![Architecture edc library](./media/architecture-edc-library.svg)

With this approach the application can shift from one application to another. For example "Digital Twin Registry" and "Submodel Service" or any other app that I have the connection open.

> [!WARNING]
> This pattern is currently only valid for "Consumer PULL" interactions. The EDR interface currently does not supports the "Provider PUSH" interaction.

### 2.3 Story line

#### 2.3.1 Prerequisites

The following prerequisites are needed, how they are found is out of the scope, and is defined is up to the use case to define that.

- The partner BPN (future DID) needs to be known.
- The EDC needs to be known.
- The Policies semantic & constraints need to be defined beforehand
- API path from the asset/application behind the EDC provider needs to be known beforehand (API Standard Interface)
  - The asset/application I want to exchange needs to be known beforehand, and referenced in the EDC "uniquely"/as standardized by Catena-X or any other dataspace.

#### 2.3.2 Connect Phase

1. Consumer calls "do_get" for "Application A/Asset1" with a specific "API path" which was found in the catalog of the EDC, and send a list of "policies that can be accepted".
2. "Proxy" application calls the EDC and checks if the "Asset1" found in the Catalog contains at least one of the "policies that can be accepted", and selects the combination from asset and policy.
3. If found EDR negotiation will be done for the asset and policy selected.
4. The consumer shall wait around 10 seconds for the negotiation to appear, if it does not appear in 20 seconds then its broken. (Note that the negotiation status can still be checked)
5. If an EDR is available, the `negotiationId` can be stored in the application side, so it can be retrieved. And to reduce the need for calls to the EDC to get the EDR, the application can just store the:
   - `contractAgreementId` → Tracks negotiation status in EDC, enables EDR refresh
   - `transferProcessId` → Enables the Access Token retrieval at the EDC.

> [!IMPORTANT]
> The application **MUST** store this information under:
>
> - For which partner (BPN)
> - For which EDC of the partner
> - Which combination of policies were introduced in the input (hash)
> - Which asset/application was selected
>
> If all this conditions stay the same the connection will remain open, until the EDC cuts the connection (because the contract agreement is not more valid)

#### 2.3.3 Exchange Phase

6. The application requests for the authorization token, for this `transferProcessId` and requests that the EDC refreshes it automatically does the "auto_refresh".

- If the token can not be retrieved, go back to step 1 and redo the connection
- If the token is available, get the token and the dataplane URL

7. Query the data plane URL concatenated with the "API path" provided at the beginning of the connection.
8. "Proxy" will "proxy" the response back to the application.

Now when the application wants to retrieve data again, the "proxy" application will reuse the `transferProcessId` to query the token (step 6) and will concatenate any other specified "API path" to the data plane URL (step 8) until the open connection expires, so the (step 2) will be repeated.

![Connect and Exchange Detailed](./media/connect&exchange-detailed.svg)

In this way connections can be open 1 a year (10 seconds) between to companies for a specific application, and then it can be accessed over and over again, reusing this "tunnel" or "pipe" that is created between these two companies, retrieving data in less than `0,6253s` over and over again. And you can even open a second, third, fourth connection and shift in between applications and conditions.

## 3. Potential Users

Applications like the:

- Digital Product Passport
- Item Relationship Service
- Certificate Management (possible use case)
- PURIS
- Simple Data Exchanger (needs to be refactored, or better recreated with a new name and purpose)
- Demand and Capacity Management
  
+ All applications that use the same asset, e.g. a DTR, in multiple requests over a longer period of time.

## 4. Advantages

- Enables the usage of the EDC in a local machine or private infrastructure.
  - Allows applications to be deployed in local and that can use the EDC as consumers.
    - Example would be to have a "Tractus-X Browser" for the public data approach (https://github.com/eclipse-tractusx/sig-release/issues/924).
- Use the EDC interfaces that are already implemented and used by some applications.
- The data exchange is really, really fast. (Connection takes apron 10 seconds [EDC Dependence], and data exchange just depends on the HTTP time).
- The data exchange is secure and the pipe open between two companies is a "secure channel". And this secure channel is maintained up to date between the EDC consumer and EDC provider automatically, always refreshed.
- Easy to use for an application, only query "do_get" or "do_post" the rest will be handled.
- The Eclipse Dataspace Connector can evolve and be optimized very fast and often without braking changes for the end user application.
- Reduces the effort for maintenance for apps using the EDC. Just one component needs to be maintained.
- Easier the adoption of new applications which do not need to include the logic from the EDC.
- Applications can retrieve data from different applications and shift connections in milliseconds. Example: First I call the DTR, then I call the Submodel Service, then I call the DTR again, then I call the submodel service. All this in milliseconds (which is now not the case)

## 5. Downsides

- Depends on the EDR interface of the EDC.
       - Currently only support "Consumer PULL", the EDR interface should be updated to support also "Provider PUSH".
       - If it breaks, is discontinued or is not maintained it will not be more valid this pattern and the logic of the EDR will need to be reimplemented.
       - The first EDR negotiation takes 10 seconds and requires an interactive intent to check when the "EDR" is available at the cache, maybe a callback can still be implemented to notify that the data is ready. But in this way a "domain" will be needed from the application side.
- Requires the usage of an intermediate to talk with the EDC as an application.
  - Maintenance of this component is required
  - The maintenance can be complex (requires the EDC developers to maintain it)
- Requires timeout configurations, for waiting for the EDR of the EDC.
- Would require the development of a new "Simple Data Exchanger" application, maybe a "Kickstart KIT" ??
  - Or maybe it can be an EDC extension
- Providing a Library in multiple programming languages would be benefitial to implement this pattern in a harmonized way. This library shall provides a service that is able to talk with the EDC management API as a consumer, and is able to retrieve data for the applications by simply doing "do_post" and "do_get"

## 6. Disclaimers

- It was supposed that all the EDC systems are working and deployed in Internet at a working Catena-X/Tractus-X environment, with all their identities being already validated.
- Some aspects from the DSP protocol were abstracted to reduce the complexity of the diagrams.
- It was supposed that the policies are standardized and are known beforehand, because it was known before the first catalog call.
- The catalog request could be done using the vanilla "catalog" request or the federated catalog, it is not and will not be specified.
- This concept just specifies how to handle the "authorization" as a data consumer. The same "application" can be used in both consumer and provider sides, for enabling a harmonized "data consuming" interface.
- This applies to all the Tractus-X components that use the EDC connector, and enables them to use the EDC in a simple way, allowing the data retrieval to be optimized and harmonized between components. Fostering more compatibility with the Tractus-X EDC, allowing applications to be easily interoperable with the Dataspace, because it will be easy to use and integrate.
- The frequency of the cache "dump" is up to the consumer to decide.
- The way on how to find the "EDC" connector is not specified, it is at least required to start the EDR negotiation, for retrieving the data.
- The way of finding the partner you want to connect with was not specified, leaving open the possibility of abstracting this functionality in the future. BPN → DID → EDC → Federated Catalog


## NOTICE

This work is licensed under the [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).

- SPDX-License-Identifier: CC-BY-4.0
- SPDX-FileCopyrightText: 2024, 2025 Contributors to the Eclipse Foundation
- Source URL: https://github.com/eclipse-tractusx/digital-product-pass
