---
title: "Generating and Verifying TraTs"
weight: 4
toc: true
---

To generate and verify TraTs, you must first write configurations specifying how to generate the TraT for a particular external API and how to verify the TraT for its resultant internal requests. This can be specified by defining the following resources:

1. [TraT](/docs/configuration-guide/trat)

    Outlines the method for generating the TraT for an external API and details the verification rules for all resultant internal requests. For more information, see [TraT](/docs/configuration-guide/trat).

2. [TraTExclusion](/docs/configuration-guide/trat-exclusion)

    Defines which endpoints within a microservice should bypass TraT verification. This is useful for non-functional APIs such as health and monitoring endpoints, or functional endpoints that are not yet TraT-ready. For further details, refer to [TraTExclusion](/docs/configuration-guide/trat-exclusion).

3. [TratteriaConfig](/docs/configuration-guide/tratteria-config)

    Establishes general TraT configurations applicable across all external APIs. Detailed information can be found at [TratteriaConfig](/docs/configuration-guide/tratteria-config).


<br>

For a comprehensive guidance on these resources, check out the [configuration guide](/docs/configuration-guide).

## Generating TraTs

After configuring and deploying the above resources, authorized services can request TraTs from the Tratteria service at the `POST /txn-token` endpoint.

#### TraT Token Request Parameters

To request TraTs, include the following parameters in the request body, formatted as `application/x-www-form-urlencoded`:

**grant_type (REQUIRED)**

Should be set to: `urn:ietf:params:oauth:grant-type:token-exchange`

**audience (REQUIRED)**

The audience for which TraTs are intented for. This value should be the same as the one configured in the [TratteriaConfig](/docs/configuration-guide/tratteria-config) resource.

**requested_token_type (REQUIRED)**

Should be set to `urn:ietf:params:oauth:token-type:txn-token`

**subject_token (REQUIRED)**

The individual, entity, or user involved in the transaction. As configured in the [TratteriaConfig](/docs/configuration-guide/tratteria-config) resource, it should be either OIDC ID token or self-signed JWT tokens.

**subject_token_type**

Should be set to `urn:ietf:params:oauth:token-type:self_signed` for self-signed JWT tokens and to `urn:ietf:params:oauth:token-type:id_token` for OIDC ID tokens.

**request_details**

Includes data about the request. It should contain the following:

*endpoint*: The request URL path.

*method*: The HTTP method of the request (e.g., `GET`, `POST`, `PUT`, `DELETE`).

*body*: The payload of the HTTP request in the JSON format.

*headers*: The HTTP headers of the request as a JSON object.

*queryParameters*: URL query parameters of the request as a JSON object.

Below is an example `request_details`:

```json
{
    "endpoint": "/order",
    "method": "POST",
    "body": {
        "stock": "NASDAQ:MSFT",
        "action": "Buy",
        "quantity": 5
    },
    "headers": {
        "Content-Type": "application/json"
    },
    "queryParameters": {}
}
```

The TraT token request expects `request_details` in the base64url encoded JSON format; thus, it should be encoded accordingly.

**request_context**

Transaction requester context. It can contain information such as the IP address of the originating user, information about the computational entity that requested the TraT, and the contextual attributes of the originating request.

For details on `request_context`, check out [TraT draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/).

The TraT token request expects `request_context` in the base64url encoded JSON format; thus, it should be encoded accordingly.

#### TraT Request Example

Including the above parameters, below is an example of the TraT token request using curl:

```bash
curl --location 'https://tratteria/token_endpoint' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'requested_token_type=urn:ietf:params:oauth:token-type:txn_token' \
--data-urlencode 'audience=https://example.org/' \
--data-urlencode 'subject_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiIsImtpZCI6ImVjZGZjNjg1MDNhMThhYjM3ZTdjNWMyYmJkMGFjNjc3In0.eyJzdWIiOnsiZm9ybWF0IjoiZW1haWwiLCJlbWFpbCI6InNoZXJsb2NrQGRldGVjdC5jb20ifSwibmFtZSI6IlNoZXJsb2NrIEhvbWVzIiwiaXNzIjoiaHR0cHM6Ly9zdG9jay10cmFkaW5nLmNvbS9zdWJqZWN0LXRva2VuLWdlbmVyYXRvciIsImV4cCI6MTcxNDk5NDQyMCwiaWF0IjoxNzE0NzgzNDA1fQ.eXIMNTilR4awBlheDcsAxW2yw2UkO8oB8-RSQKQfprKJXUwhb0MeEcqENVVtezB4jWNR7xZ1Yx_Q0yrAHOVgUA' \
--data-urlencode 'subject_token_type=urn:ietf:params:oauth:token-type:self_signed' \
--data-urlencode 'request_details=ewogICAgImVuZHBvaW50IjogIi9vcmRlciIsCiAgICAibWV0aG9kIjogIlBPU1QiLAogICAgImJvZHkiOiB7CiAgICAgICAgInN0b2NrIjogIk5BU0RBUTpNU0ZUIiwKICAgICAgICAiYWN0aW9uIjogIkJ1eSIsCiAgICAgICAgInF1YW50aXR5IjogNQogICAgfSwKICAgICJoZWFkZXJzIjogewogICAgICAgICJDb250ZW50LVR5cGUiOiAiYXBwbGljYXRpb24vanNvbiIKICAgIH0sCiAgICAicXVlcnlQYXJhbWV0ZXJzIjoge30KfQ==' \
--data-urlencode 'grant_type=urn:ietf:params:oauth:grant-type:token-exchange' \
--data-urlencode 'request_context=ewogICJyZXFfaXAiOiAiMTkyLjEyOC4wLjg5Igp9'
```

For a practical example of requesting TraTs, please refer to the Tratteria Example Application's [Gateway service](https://github.com/tratteria/example-application/tree/main/gateway), which requests TraTs for the application.

For more details and context on TraT token request, consult the [TraTs draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/).

#### TraT Token Request Response

A successful TraT token request response contains the following:

**token_type**

Set to `N_A`

**issued_token_type**

Set to `urn:ietf:params:oauth:token-type:txn_token`

**access_token**

Set to the **TraT** generated by the request.

Below is an example TraT token request response:

```json
{
    "token_type": "N_A",
    "issued_token_type": "urn:ietf:params:oauth:token-type:txn_token",
    "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IlFOVkVUMTh5ZStRT0UvdVVsa1hFa3c9PSIsInR5cCI6InR4bl90b2tlbiJ9.eyJhdWQiOiJodHRwczovL2V4YW1wbGUub3JnLyIsImF6ZCI6eyJhY3Rpb24iOiJCdXkiLCJxdWFudGl0eSI6NSwic3RvY2siOiJOQVNEQVE6TVNGVCJ9LCJleHAiOjE3MTUyMzAxMzMsImlhdCI6MTcxNTIzMDExOCwiaXNzIjoiaHR0cHM6Ly9leGFtcGxlLm9yZy90dHMiLCJwdXJwIjoic3RvY2tzLnRyYWRlIiwicmN0eCI6eyJyZXFfaXAiOiIxOTIuMTI4LjAuODkifSwic3ViIjp7ImVtYWlsIjoic2hlcmxvY2tAZGV0ZWN0LmNvbSIsImZvcm1hdCI6ImVtYWlsIn0sInR4biI6ImUzNDAyNWI5LTI2MjQtNGY1ZC1iZDFhLTgzZGVmODM3OWViZSJ9.kse_HrxLxgI583Z5uez3jBX5ylgR7oFTSwCIv5zsuYGGiBpDn-OBPmvEVyxIcT2dZC5WD82AMRjUrTy4wFquZReeuIxminMTqocWBl2v4Pu8uLvcEnH-Tv9qa8MtgXcU0a-xELWGulqe2UAUv_3Mi0Wb20QMgURcFaFcT7ccZXX_xHsrsboKavS7H_bWhEcwR3FvXKt1YwY3zDXiUHZaxjqGTqrv0V8wDFSZmnVFapT5RRH2tarYOmuKwv3MbdaXT0ZHBvQ2S2fBzOZ47WaTiTv9sKB-N8am94J3I_bo15dCauOm_bXZGT5ybbuBAp23D3297ARIYl74Xf_MJcapKA"
}
```

where `access_token` is set to the generated TraT, which has the the following body:

```json
{
    "aud": "https://example.org/",
    "azd": {
        "action": "Buy",
        "quantity": 5,
        "stock": "NASDAQ:MSFT"
    },
    "exp": 1715230133,
    "iat": 1715230118,
    "iss": "https://example.org/tts",
    "purp": "stocks.trade",
    "rctx": {
        "req_ip": "192.128.0.89"
    },
    "sub": {
        "email": "sherlock@detect.com",
        "format": "email"
    },
    "txn": "e34025b9-2624-4f5d-bd1a-83def8379ebe"
}
```

For details on TraT claims check out [TraTs draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/).

## Verifying TraTs

TraTs are verified using [Tratteria agents](https://github.com/tratteria/tratteria-agent), which are sidecar containers that run alongside microservice containers. For details on how to verify TraTs using Tratteria agents, visit [Tratteria agents readme](https://github.com/tratteria/tratteria-agent).

&nbsp;