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

Welcome to the documentation for Tratteria, an open-source [Transaction Tokens (TraTs)](/docs/transaction-token) Service. Tratteria is designed to facilitate secure and convenient TraTs issuance and verification in microservices systems.

<img src="/img/docs/introduction/tratteria_workflow.svg" alt="Tratteria Workflow" class="doc-image">

<br>

Tratteria supports TraTs generation and verification using Kubernetes resources and Tratteria sidecar agents. Tratteria lets you define how to generate the TraT for an external API and how to verify the TraT for the resulting internal requests of the external API. Additionally, Tratteria supports access evaluation for external APIs. Below is a sample Tratteria Kubernetes resource for an external API:

```yaml
apiVersion: tratteria.io/v1alpha1
kind: TraT
metadata:
  name: stock-trade-api-trat
  namespace: alpha-stocks-dev
spec:
  endpoint: "/api/order"
  method: "POST"
  purp: stock-trade
  azdMapping:
    stockId:
      required: true
      value: "${body.stockId}"
    action:
      required: true
      value: "${body.orderType}"
    quantity:
      required: true
      value: "${body.quantity}"
  services:
    - name: order
    - name: catalog
      endpoint: "catalog/catalog-update/{#stock-id}?update-type={#action}"
    - name: stocks
      endpoint: "stocks/?id={#id}"
      azd-mapping:
        stock:
          required: true
          value: "{$id}"
  accessEvaluation:
    subject:
      id: "${subject_token.email}"
    action:
      name: "${scope}"
    resource:
      stockId: "${body.stockId}"
```

The above specifies how to generate purpose and authorization details for the `POST /api/order` API, and it specifies who (the `order`, `catalog`, and `stocks` services) and how to verify the generated TraT. Additionally, the `accessEvaluation` section specifies how to perform access evaluations for the API.

To integrate Tratteria into your microservice application, start by [installing Tratteria](/docs/installation).

## Acknowledgments

This documentation and the underlying technology are based on the concepts and drafts developed by Atul Tulshibagwale (SGNL), George Fletcher (Capital One), and Pieter Kasselman (Microsoft).


