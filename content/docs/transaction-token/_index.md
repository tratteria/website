---
title: "Transaction Tokens (TraTs)"
weight: 6
toc: true
---

## Background
Transaction Tokens (TraTs) are described in a draft specification from the IETF OAuth working group. See here: [Transaction Tokens](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/). This document describes the background and need for Transaction Tokens.

### Microservices

A microservices architecture breaks down applications into smaller, independent services that interact through well-defined APIs. This approach offers several advantages including enhanced development velocity and independent development and release cycles.

Microservices are deployed and run within a secured "virtual private cloud" (VPC).

<img src="/img/docs/introduction/microservices.svg" alt="Microservices Diagram" class="doc-image">


### Security Limitations of VPCs

Despite providing a layer of security, VPCs are not impervious to threats:

- **Software Supply Chain Vulnerabilities:** Malicious code can enter VPCs through the supply chain, potentially leading to Remote Code Execution (RCE) attacks within cloud environment.
- **Privileged User Compromise:** Insiders or attackers with elevated privileges can exploit their access, often remaining undetected and causing significant damage.

### VPCs Need More Security

VPCs often rely on implicit trust models that can leave systems vulnerable. To enhance security, it's essential to transition towards a context-aware security model:

<img src="/img/docs/introduction/vpc_security_concerns.svg" alt="VPC Security Concerns Diagram" class="doc-image">

- **Implicit Trust**: Services within the VPC trust each other.
- **Service-to-Service Trust:** Establishes mutual authentication and authorization between services. This is sometimes achieved using [SPIFFE](https://spiffe.io/)
- **User Trust:** Extends service-to-service trust to also assures the identity of the user making the request, confirming users are who they claim to be. Some developers pass external ID Tokens to assert the identity of the calling user, or use OAuth [Token Introspection](https://datatracker.ietf.org/doc/html/rfc7662) in order to obtain the identity of the user initiating the transaction. Both these approaches are susceptible to token replay attacks. 
- **Assured Context**: Extends user trust to also assure the authorization context of the request. This can be achieved using TraTs.

## Moving Toward Assured Context with Transaction Tokens

To address these vulnerabilities, TraTs represent an evolution from traditional security measures to a more robust, context-aware framework. TraTs offer assurance of both user identity and authorization context at every step of a transaction.

### Transaction Tokens (TraTs)

TraTs are short-lived, cryptographically signed JSON Web Tokens (JWTs) that immutably preserve the user identity and authorization context of an external API invocation. They ensure that the user identity and authorization details of an external request, such as an API call, are maintained across all involved services within a microservices application. Additionally, TraTs enable these services to assert their involvement in the transaction chain to downstream workloads.

### Benefits of TraTs

TraTs offer the following benefits:

- **Prevention of Spurious Invocations**: By verifying the presence of an external trigger, TraTs help prevent unauthorized internal calls within the network.

- **Immutable Context Preservation**: TraTs ensure that the user identity and authorization context of an external request are preserved across all workloads that process the request, helping prevent impersonation and unauthorized parameter modification.

- **Call Chain Assertion**: Allows workloads to immutably assert to downstream workloads that they were invoked in the call chain of the request.

- **Reduction of Blast Radius**: By validating each invocation in the chain, TraTs reduce the blast radius of security incidents. This limits the impact of compromised workloads, or malicious insiders, enhancing overall security and trust within the system.

- **Short-Lived Tokens**: TraTs are designed to be short-lived (e.g., 15 seconds), minimizing the risk of replay attacks if a token is discovered by an attacker.

- **Independence from Access Tokens**: TraTs do not incorporate external access tokens, thereby limiting exposure and minimizing the risk of token theft.

- **Request Context Preservation**: Preserves relevant request context information, such as IP address, authentication method, and requester workload, which can help downstream services make informed authorization decisions.

## TraTs Service

The Transaction Token Service (TraTs Service) issues TraTs to requesting workloads. Requesting workloads authenticate to the TraTs Service and request tokens by providing the necessary context, which the service uses to generate TraTs. A limited, pre-configured set of workloads, typically only the API gateway, can request TraTs. Secure methods, such as SPIFFE, are used to authenticate these workloads.


### How it Works

- **Initial Invocation**: When a user invokes an external endpoint in an API microservice, the service authenticates the request using conventional authorization mechanisms (e.g., OAuth 2.0).
- **Token Request**: The API microservice uses a specific profile of OAuth Token Exchange (defined in the Transaction Tokens [draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/)) to request a TraT from the TraTs Service, providing context about the request, such as request parameters, user’s identity, and the initial authorization token.
- **Token Issuance**: The TraTs Service verifies the requesting service using secure methods such as SPIFFE, validates the request, and issues a TraT that includes immutable information about the user and the context of the request.
- **Propagation**: The API microservice uses the TraT to authorize its subsequent calls to internal services. These downstream services reuse the same token to authorize their subsequent calls to other services.
- **Verification**: Each service in the call chain independently verify the TraT to ensure that the request is valid and that the context has not been tampered with.

<img src="/img/docs/introduction/how_it_works.svg" alt="How It Works" class="doc-image">

&nbsp;