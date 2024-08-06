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

Welcome to the documentation for Tratteria, an open-source [Transaction Tokens](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/) (TraTs) Service. This guide will help you understand what Tratteria is, how it works, and how to implement it in your microservices architecture.

## TraT

TraTs (Transaction Tokens) are short-lived JWTs that assure identity and context in a microservices call chain. Learn more about TraTs [here](/docs/transaction-token). The example below describes the salient features of a TraT:

<img src="/img/docs/introduction/what_is_a_trat.jpg" alt="What Is a TraT" class="doc-image">

## Tratteria Architecture

Tratteria is designed to facilitate secure and convenient TraT issuance and verification in microservices systems. It involves the Tratteria Service for issuing TraTs, the Tratteria Agent sidecar for verifying TraTs, and Tratteria Kubernetes resources for specifying generation and verification rules for TraTs.

<img src="/img/docs/introduction/tratteria_workflow.svg" alt="Tratteria Workflow" class="doc-image">

<br>

## Tratteria Modes

Tratteria can operate in two modes:

* **The Interception Mode**: Enables existing applications to adopt TraTs without (almost) any code changes. It injects Tratteria Agent sidecar containers into Kubernetes pods, and the application continues to operate the way it used to, except the path, query and body of each call are verified against an associated TraT.

  If a service needs to forward a TraT to a downstream service, then it needs to add the `Txn-token` HTTP header and include the TraT as the value of that header in outbound calls. If a microservice does not make any downstream calls, then it does not need to change.

* **The Delegation Mode**: In this approach, the application explicitly calls the Tratteria Agent within its Kubernetes pod to verify TraTs. As a result, the application needs to make this change to its code to use Tratteria. This mode is more secure than the interception mode, as it avoids scenarios where sidecar could potentially be bypassed. In addition, a delegation based approach allows the application to pack the call parameter information in the Txn-Token header, and can potentially eliminate having to send it separately through query parameters or the body.

  This mode is suitable for environments where intercepting incoming requests is not possible or desired, for example, in environments with a service mesh that is already intercepting incoming requests.

## Tratteria Resource

Tratteria lets you define how to generate the TraT for an external API and how to verify the TraT for the resulting internal requests of the external API using Kubernetes resources. Additionally, it supports specifying access evaluation for external APIs.

Below is a sample Tratteria Kubernetes resource for the `POST api/order/trade/{#stockId}` external API. Hover your mouse over the text below to find out more about what each line means:

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
.tooltip {
    position: relative;
    display: inline;
    cursor: help;
}
.tooltip .tooltiptext {
    visibility: hidden;
    min-width: 400px;
    max-width: 600px;
    background-color: #555;
    color: #fff;
    text-align: center;
    border-radius: 6px;
    padding: 5px 10px;
    position: absolute;
    z-index: 100;
    left: 0;
    top: 0px;
    transform: translateY(-100%);
    opacity: 0;
    transition: opacity 0.5s, visibility 0.5s;
    white-space: normal;
    word-wrap: break-word;
}
pre {
    background-color: #1e1e1e;
    border: none;
    color: #d4d4d4;
    font-family: 'Courier New', monospace;
    font-size: 15px;
    line-height: 1.5;
    overflow: auto;
    padding: 10px;
    border-radius: 5px;
    width: auto;
    display: block; 
}
.yaml-key {
    color: #f92672;
}
.yaml-value {
    color: #ae81ff;
}
.yaml-string {
    color: #e6db74;
}
</style>
</head>
<body>
<pre>
<code class="yaml"><span class="yaml-key">apiVersion</span>: <span class="yaml-value">tratteria.io/v1alpha1</span>
<span class="yaml-key">kind</span>: <span class="yaml-value">TraT</span>
<span class="yaml-key">metadata</span>:
  <span class="tooltip"><span class="yaml-key">name</span>: <span class="yaml-value">trade-api-trat</span><span class="tooltiptext">The name of the TraT used for this external API. This uniquely identifies the TraT type within the Tratteria system.</span></span>
  <span class="yaml-key">namespace</span>: <span class="yaml-value">app-dev</span>
<span class="yaml-key">spec</span>:
  <span class="tooltip"><span class="yaml-key">path</span>: <span class="yaml-value">"api/order/trade/{#stockId}?action={#action}"</span><span class="tooltiptext">The default URL path, which, together with the method field below, results in this type of TraT being generated or validated. The URL may used by external API services or internal microservices.</span></span>
  <span class="tooltip"><span class="yaml-key">method</span>: <span class="yaml-value">"POST"</span><span class="tooltiptext">The HTTP method for which this TraT resource is defined (see `path` above).</span></span>
  <span class="tooltip"><span class="yaml-key">purp</span>: <span class="yaml-value">"stock-trade"</span><span class="tooltiptext">The purpose of the TraT, which is included in the `purp` field of TraTs of this type.</span></span>
  <span class="tooltip"><span class="yaml-key">azdMapping</span>:
    <span class="yaml-key">stockId</span>:
      <span class="yaml-key">required</span>: <span class="yaml-value">true</span>
      <span class="yaml-key">value</span>: <span class="yaml-string">"${stockId}"</span>
    <span class="yaml-key">action</span>:
      <span class="yaml-key">required</span>: <span class="yaml-value">true</span>
      <span class="yaml-key">value</span>: <span class="yaml-string">"${action}"</span>
    <span class="yaml-key">quantity</span>:
      <span class="yaml-key">required</span>: <span class="yaml-value">true</span>
      <span class="yaml-key">value</span>: <span class="yaml-string">"${body.quantity}"</span><span class="tooltiptext">The immutable context preserved in this TraT and how to construct and verify it.</span></span>
  <span class="tooltip"><span class="yaml-key">services</span>:
    - <span class="yaml-key">name</span>: <span class="yaml-value">order</span>
    - <span class="yaml-key">name</span>: <span class="yaml-value">catalog</span>
      <span class="tooltip"><span class="yaml-key">path</span>: <span class="yaml-string">"catalog/catalog-update/{#stockId}?update-type={#action}"</span><span class="tooltiptext">The catalog service overrides just the path.</span></span>
    - <span class="yaml-key">name</span>: <span class="yaml-value">stocks</span>
      <span class="tooltip"><span class="yaml-key">path</span>: <span class="yaml-string">"stocks/{#id}"</span><span class="tooltiptext">The stocks service overrides the path, method and how it receives information that needs to be verified against the TraT content.</span></span>
      <span class="tooltip"><span class="yaml-key">method</span>: <span class="yaml-string">"GET"</span><span class="tooltiptext">As noted above, the stocks service overrides defaults for the HTTP method.</span></span>
      <span class="tooltip"><span class="yaml-key">azdMapping</span>:
        <span class="yaml-key">stockId</span>:
          <span class="yaml-key">required</span>: <span class="yaml-value">true</span><span class="tooltiptext">As noted above, the stocks service overrides defaults for TraT verification</span></span>
          <span class="yaml-key">value</span>: <span class="yaml-string">"{$id}"</span><span class="tooltiptext">The list of microservice APIs that are invoked while processing this external API. They may use defaults specified above or override them.</span></span>
  <span class="tooltip"><span class="yaml-key">accessEvaluation</span>:
    <span class="yaml-key">subject</span>:
      <span class="yaml-key">id</span>: <span class="yaml-string">"${subject_token.email}"</span>
    <span class="yaml-key">action</span>:
      <span class="yaml-key">name</span>: <span class="yaml-string">"${purp}"</span>
    <span class="yaml-key">resource</span>:
      <span class="yaml-key">stockId</span>: <span class="yaml-string">"${request_details.body.stockId}"</span><span class="tooltiptext">Tratteria can call out to an AuthZEN API to evaluate whether execution should proceed. This specifies how to construct the request for access evaluation.</span></span></code>
</pre>

<script>
document.addEventListener('DOMContentLoaded', function() {
    const tooltips = document.querySelectorAll('.tooltip');
    
    function hideAllTooltips() {
        document.querySelectorAll('.tooltip .tooltiptext').forEach(tt => {
            tt.style.visibility = 'hidden';
            tt.style.opacity = '0';
        });
    }

    tooltips.forEach(tooltip => {
        tooltip.addEventListener('mouseover', function(e) {
            e.stopPropagation();

            hideAllTooltips();

            const tooltipText = Array.from(this.children).find(child => child.classList.contains('tooltiptext'));
            if (tooltipText) {
                tooltipText.style.visibility = 'visible';
                tooltipText.style.opacity = '1';
            }
        });

        tooltip.addEventListener('mouseout', function() {
            const tooltipText = Array.from(this.children).find(child => child.classList.contains('tooltiptext'));
            if (tooltipText) {
                tooltipText.style.visibility = 'hidden';
                tooltipText.style.opacity = '0';
            }
        });
    });
});

</script>
</body>
</html>

The above specifies how to generate purpose and authorization details for the `POST api/order/trade/{#stockId}` API, and it specifies who (the `order`, `catalog`, and `stocks` services) and how to verify the generated TraT. Additionally, the `accessEvaluation` section specifies how to perform access evaluations for the API.

To quickly see Tratteria in action, checkout the [Quickstart Guide](/docs/quickstart).

To integrate Tratteria into your microservice application, start by [installing Tratteria](/docs/installation), which can be deployed in environments with or without a service mesh.

## Acknowledgments

This documentation and the underlying technology are based on the concepts and [draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/) developed by Atul Tulshibagwale (SGNL), George Fletcher (Capital One), and Pieter Kasselman (Microsoft).

## Contact Us

For inquiries about Tratteria, please email us at: [tratteria@sgnl.ai](mailto:tratteria@sgnl.ai)