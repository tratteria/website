---
title: "Introduction to Tratteria"
toc: true
weight: 1
---

<style>
.doc-image {
    display: block;
    margin: 50px;
    padding: 0;
}
</style>

Welcome to the documentation for Tratteria, an open-source Transaction Tokens (TraTs) Service. This section provides an overview of Txn-Tokens and Tratteria.

## Background

### Microservices

Microservices architecture breakes down an application into smaller, independent services that interact through well-defined APIs. This approach offers several advantages including enhanced development speed and independent development.

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
- **Service-to-Service Trust:** Establishes mutual authentication and authorization between services.
- **User Trust:** Extends service-to-service trust to also assures the identity of the user making the request, confirming users are who they claim to be.
- **Assured Context**: Extends user trust to also assure the authorization context of the request.

## Moving Toward Assured Context with Transaction Tokens

To address these vulnerabilities, Transaction Tokens (TraTs) represent an evolution from traditional security measures to a more robust, context-aware framework. TraTs offer assurance of both user identity and authorization context at every step of a transaction.

### Transaction Tokens (TraTs)

Txn-Tokens are short-lived, cryptographically signed JSON Web Tokens that immutably preserve the user identity and authorization context of an external API invocation. They ensure that the user identity and authorization details of an external request, such as an API call, are maintained across all involved services within a microservices application. Additionally, Txn-Tokens enable these services to assert their involvement in the transaction chain to downstream workloads.

### Benefits of Transaction Tokens (TraTs)

Transaction Tokens (Txn-Tokens) offer several benefits:

- **Prevention of Spurious Invocations**: By verifying the presence of an external trigger, Txn-Tokens help prevent unauthorized internal calls within the network.

- **Immutable Context Preservation**: Txn-Tokens ensure that the user identity and authorization context of an external request are preserved across all workloads that process the request, helping prevent impersonation and unauthorized parameter modification.

- **Call Chain Assertion**: Allows workloads to immutably assert to downstream workloads that they were invoked in the call chain of the request.

- **Reduction of Blast Radius**: By validating each invocation in the chain, Txn-Tokens reduce the blast radius of security incidents. This limits the impact of compromised workloads, or malicious insiders, enhancing overall security and trust within the system.

- **Short-Lived Tokens**: Txn-Tokens are designed to be short-lived (e.g., 15 seconds), minimizing the risk of replay attacks if a token is discovered by an attacker.

- **Independence from Access Tokens**: Txn-Tokens do not incorporate external access tokens, thereby limiting exposure and minimizing the risk of token theft.

- **Request Context Preservation**: Preserves relevant request context information, such as IP address, authentication method, and requester workload, which can help downstream services make informed authorization decisions.

## Txn-Token Service

The Transaction Token Service (Txn-Token Service) issues Txn-Tokens to requesting workloads. Requesting workloads authenticate to the Txn-Token Service and request tokens by providing the necessary context, which the service uses to generate Txn-Tokens. A limited, pre-configured set of workloads, typically only the API gateway, can request Txn-Tokens. Secure methods, such as SPIFFE, are used to authenticate these workloads.


### How it Works

- **Initial Invocation**: When a user invokes an external endpoint in an API microservice, the service authenticates the request using conventional authorization mechanisms (e.g., OAuth 2.0).
- **Token Request**: The API microservice uses OAuth Token Exchange to request a Txn-Token from the Txn-Token Service, providing context about the request, such as request parameters, user’s identity, and the initial authorization token.
- **Token Issuance**: The Txn-Token Service verifies the requesting service using secure methods such as SPIFFE, validates the request, and issues a Txn-Token that includes immutable information about the user and the context of the request.
- **Propagation**: The API microservice uses the Txn-Token to authorize its subsequent calls to internal services. These downstream services reuse the same token to authorize their subsequent calls to other services.
- **Verification**: Each service in the call chain independently verify the Txn-Token to ensure that the request is valid and that the context has not been tampered with.

<img src="/img/docs/introduction/how_it_works.svg" alt="How It Works" class="doc-image">


## What is Tratteria?

Tratteria is an open-source implementation of Transaction Tokens (TraTs) Service. It's designed to facilitate secure, reliable, and efficient Txn-Token issuance and verification in microservices systems. Tratteria can be seamlessly integrated into existing systems.

Tratteria supports a variety of configurations and is highly customizable to meet the specific needs of different environments. It is compatible with Service Meshes, Open Policy Agent (OPA), and SPIFEE, making it a flexible option for modern infrastructures. Whether you're running native services or containerized applications, Tratteria offers seamless integration.

<img src="/img/docs/introduction/tratteria_workflow.svg" alt="Tratteria Workflow" class="doc-image">

## Acknowledgments

This documentation and the underlying technology are based on the concepts and drafts developed by Atul Tulshibagwale (SGNL), George Fletcher (Capital One), and Pieter Kasselman (Microsoft).

For detailed discussions and updates, visit the [OAuth Working Group mailing list](https://mailarchive.ietf.org/arch/browse/oauth/) or the [Github repository](https://github.com/oauth-wg/oauth-transaction-tokens).

