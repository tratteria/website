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

Tratteria supports TraTs generation and verification using Kubernetes resources and Tratteria sidecar agents. Tratteria lets you define how to generate the TraT for an external API and how to verify the TraT for the resulting internal requests of the external API. Additionally, Tratteria supports access evaluation for external APIs.

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
  <span class="tooltip"><span class="yaml-key">endpoint</span>: <span class="yaml-value">"api/order/trade/{#stockId}?action={#action}"</span><span class="tooltiptext">The default URL path, which, together with the method field below, results in this type of TraT being generated or validated. The URL may used by external API services or internal microservices.</span></span>
  <span class="tooltip"><span class="yaml-key">method</span>: <span class="yaml-value">"POST"</span><span class="tooltiptext">The HTTP method for which this TraT resource is defined (see `endpoint` above).</span></span>
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
      <span class="tooltip"><span class="yaml-key">endpoint</span>: <span class="yaml-string">"catalog/catalog-update/{#stockId}?update-type={#action}"</span><span class="tooltiptext">The catalog service overrides just the endpoint.</span></span>
    - <span class="yaml-key">name</span>: <span class="yaml-value">stocks</span>
      <span class="tooltip"><span class="yaml-key">endpoint</span>: <span class="yaml-string">"stocks/{#id}"</span><span class="tooltiptext">The stocks service overrides the endpoint, method and how it receives information that needs to be verified against the TraT content.</span></span>
      <span class="tooltip"><span class="yaml-key">method</span>: <span class="yaml-string">"GET"</span><span class="tooltiptext">As noted above, the stocks service overrides defaults for the HTTP method.</span></span>
      <span class="tooltip"><span class="yaml-key">azdMapping</span>:
        <span class="yaml-key">stockId</span>:
          <span class="yaml-key">required</span>: <span class="yaml-value">true</span><span class="tooltiptext">As noted above, the stocks service overrides defaults for TraT verification</span></span>
          <span class="yaml-key">value</span>: <span class="yaml-string">"{$id}"</span><span class="tooltiptext">List of services that use this TraT type. They may use defaults specified above or override them.</span></span>
  <span class="tooltip"><span class="yaml-key">accessEvaluation</span>:
    <span class="yaml-key">subject</span>:
      <span class="yaml-key">id</span>: <span class="yaml-string">"${subject_token.email}"</span>
    <span class="yaml-key">action</span>:
      <span class="yaml-key">name</span>: <span class="yaml-string">"${scope}"</span>
    <span class="yaml-key">resource</span>:
      <span class="yaml-key">stockId</span>: <span class="yaml-string">"${body.stockId}"</span><span class="tooltiptext">Tratteria can call out to an AuthZEN API to evaluate whether execution should proceed. This specifies how to construct the request for access evaluation.</span></span></code>
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

To integrate Tratteria into your microservice application, start by [installing Tratteria](/docs/installation).

## Acknowledgments

This documentation and the underlying technology are based on the concepts and [draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/) developed by Atul Tulshibagwale (SGNL), George Fletcher (Capital One), and Pieter Kasselman (Microsoft).