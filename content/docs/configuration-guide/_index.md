---
title: "Configuration Guide"
weight: 4
toc: true
---

Tratteria supports the configuration of TraTs generation and verification for APIs through Kubernetes resources. This document outlines the resources available and provides examples of how to use them in an application.

# Resource Types

Tratteria utilizes three types of Kubernetes resources to manage TraTs generation and verification:

1. [TraT](/docs/configuration-guide/trat) - Defines the generation and verification of the TraT for an external API.

2. [TraTExclusion](/docs/configuration-guide/trat-exclusion) - Specifies service endpoints that do not require TraTs verification.

3. [TratteriaConfig](/docs/configuration-guide/tratteria-config) - Sets general configurations that apply to all APIs and TraTs.

<br>

For practical application and additional context, please refer to [example application configs](/docs/configuration-guide/example-application-configs) which details how the [example application](https://github.com/tratteria/example-application) defines these resources for TraTs generation and verification.