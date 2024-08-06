---
title: "TraT"
weight: 1
toc: true
---

A TraT resource corresponds to a TraT type, defining a TraT for an external API. It defines how to generate the TraT for an external API and how to verify the TraT in the resulting internal requests. Additionally, it supports access evaluation for external APIs. A TraT resource comprises four sections: External API Specification, TraT Generation, TraT Verification, and Access Evaluation.

The external API specification section specifies the external API for which the TraT is defined. TraT generation section supports constructing the TraT using request URL parameters, headers, body, and literal constants. TraT verification section support listing the internal APIs that are invoked by the external API and rules to verify the TraT in them. And the access evaluation section supports specifying the rule to construct access evaluation request body for the external API.

Let's delve into the details of TraTs by examining an example.

```yaml
apiVersion: tratteria.io/v1alpha1
kind: TraT
metadata:
  name: order-trade-api-trat
  namespace: alpha-stocks-dev
spec:
  path: "order/trade/{#stockId}?action={#action}"
  method: "POST"
  purp: trade
  azdMapping:
    quantity:
      required: true
      value: "${body.quantity}"
    action:
      required: true
      value: "${action}"
    stockId:
      required: true
      value: "${stockId}"
  services:
    - name: order
    - name: catalog
      path: "catalog/catalog-update/{#stockId}?update-type={#action}"
    - name: stocks
      path: "stocks/?id={#id}"
      method: "GET"
      azdMapping:
        stockId:
          required: true
          value: "{$id}"
    - name: payment
      path: "payment/trade?numbers={#numbers}"
      azdMapping:
        quantity:
          required: true
          value: "{$numbers}"
  accessEvaluation:
    subject:
      id: "${subject_token.email}"
    action:
      name: "${scope}"
    resource:
      stockId: "${stockId}"
```

Below are the four sections of the above TraT:

#### External API Specification Section

```yaml
path: "order/trade/{#stockId}?action={#action}"
method: "POST"
```

The above configuration specifies the external API for which the `order-trade-api-trat` is defined. The path supports the definition of variables that can be referenced later. A variable is defined by placing a name prefixed with a `#` inside the curly braces. In the above path, the `stockId` path parameter and `action` query parameter are defined.

#### TraT Generation Section

```yaml
purp: trade
azdMapping:
quantity:
    required: true
    value: "${body.quantity}"
action:
    required: true
    value: "${action}"
stock:
    required: true
    value: "${stockId}"
```

The above configuration specifies how to construct the TraT purpose and authorization details for the external API. It supports constructing them using the following parameters and their JSON path expressions:

1. Request Body (referenced using `${body}`)
2. Request Header (referenced using `${header}`)
3. Variables defined in the API specification section (referenced using `${variable_name}`)
4. Literal string constants

The above generation rule produces a TraT as follows:

```plaintext
{
    ...
    purp: "trade",
    adz: {
        quantity: ...,
        action: ...,
        stock: ...,
    }
    ...
}
```

The `required` field in the `azdMapping` is used for TraT verification. If required is set to `true`, the field must be present in both the TraT and the request. If set to `false`, then the field is either not present in both the TraT and the request, or present in both. A TraT field being present only in the TraT or only in the request is never valid.

#### TraT Verification Section

```yaml
- name: order
- name: catalog
    path: "catalog/catalog-update/{#stockId}?update-type={#action}"
- name: stocks
    path: "stocks/?id={#id}"
    method: "GET"
    azdMapping:
    stockId:
        required: true
        value: "{$id}"
- name: payment
    path: "payment/trade?numbers={#numbers}"
    azdMapping:
    quantity:
        required: true
        value: "{$numbers}"
```

The above section lists the services and their APIs that are invoked while processing the `POST order/trade/{#stockId}?action={#action}` external API. For each service, the default `path`, `method`, and `azdMapping` may be used or each field can be overridden with different values.

In the above, the `order`, `catalog`, `stocks`, and `payment` services' endpoints are invoked when processing the external trade request.

**Order service**: All `path`, `method`, and `azdMapping` are inherited from the original definition.

**Catalog service**: `azdMapping` is inherited from the original definition.

**Stocks service**: None is inherited.

**Payment service**: None is inherited.

#### Access Evaluation Section

This section is optional. If access evaluation is enabled, then this section details how to construct the access evaluation request body for the external API.

```yaml
accessEvaluation:
subject:
    id: "${subject_token.email}"
action:
    name: "${scope}"
resource:
    stockId: "${stockId}"
```

This section supports referencing all default azdMapping references (`${body}`, `${header}`, and `${variable_name}`) as well as transaction-token request components (`grant_type`, `requested_token_type`, `audience`, `purpose`, `subject-token`, `subject-token-type`, `request-details`, and `request-context`).

The configuration above generates an access evaluation request body as follows:

```plaintext
{
    "subject": {
        "id": ...
    },
    "action": {
        "name": ...
    },
    "resource": {
        "stockId": ...
    }
}
```