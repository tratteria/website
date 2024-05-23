---
Title: "Generating TraTs"
weight: 2
toc: true
---

This guide explains how to generate TraTs using Tratteria after its successful setup, as described in the [Running Tratteria](/docs/quickstart/running-tratteria) guide.

&nbsp;

- [Overview](#overview)

- [TraTs Request Parameters](#trat-request-parameters)

- [TraTs Request](#trat-request)

- [TraTs Response](#trat-response)

&nbsp;

### Overview

Once Tratteria is running, either as a native Go server or inside a Docker container, you can generate TraTs. The service is accessible at `localhost:9090` and provides the http endpoint `POST /token_endpoint` for token generation.


&nbsp;

#### TraT Request Parameters

To request TraTs, the following parameters must be provided in the request:

**grant_type (REQUIRED)**

Set to

```plaintext
urn:ietf:params:oauth:grant-type:token-exchange
```


**audience (REQUIRED)**

The audience for which TraTs are intented for. This value should be the same as the one configured in the configuration file. As configured in the quick-start config file, we will set it to

```plaintext
https://example.org/
```

**scope (REQUIRED)**

For this guide, we will generate TraTs for trading stocks. We will use the following scope for this transaction:

```plaintext
stocks.trade
```

**requested_token_type (REQUIRED)**

Set to

```plaintext
urn:ietf:params:oauth:token-type:txn-token
```

**subject_token (REQUIRED)**

The individual, entity, or user involved in the transaction. As we have configured Tratteria with self-signed subject tokens, we can pass a custom JWT as the subject token. You can generate your subject token using [this online tool](https://www.scottbrady91.com/tools/jwt) or using other available methods.

Alternatively, for now, you can use the following JWT as well:

```plaintext
eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiIsImtpZCI6ImVjZGZjNjg1MDNhMThhYjM3ZTdjNWMyYmJkMGFjNjc3In0.eyJzdWIiOnsiZm9ybWF0IjoiZW1haWwiLCJlbWFpbCI6InNoZXJsb2NrQGRldGVjdC5jb20ifSwibmFtZSI6IlNoZXJsb2NrIEhvbWVzIiwiaXNzIjoiaHR0cHM6Ly9zdG9jay10cmFkaW5nLmNvbS9zdWJqZWN0LXRva2VuLWdlbmVyYXRvciIsImV4cCI6MTcxNDk5NDQyMCwiaWF0IjoxNzE0NzgzNDA1fQ.eXIMNTilR4awBlheDcsAxW2yw2UkO8oB8-RSQKQfprKJXUwhb0MeEcqENVVtezB4jWNR7xZ1Yx_Q0yrAHOVgUA
```

The above JWT has the below body:

```json
{
    "sub": {
        "format": "email",
        "email": "sherlock@detect.com"
    },
    "name": "Sherlock Homes",
    "iss": "https://stock-trading.com/subject-token-generator",
    "exp": 1714994420,
    "iat": 1714783405
}
```

Tratteria verifies the signature of the self-signed subject tokens it receives in TraT requests. However, for simplicity, we have disabled the verification in the quick-start configuration. Therefore, there's need to concern ourselves with the keys used for signing these self-signed tokens in this guide. However, in a production-level setting, you would enable this verification to ensure the security and integrity of self-signed tokens. For guidance on how to enable and configure this, please cleck [this guide](#).

**subject_token_type**

As we are using self-signed subject token, so we will set it to

```plaintext
urn:ietf:params:oauth:token-type:self_signed
```

**request_details**

Authorization context of the transaction. Since this transaction is for stocks trading, let's use the below as the request details:

```json
{
    "action": "Buy",
    "quantity": 5,
    "stock": "NASDAQ:MSFT"
}
```

The TraT request requires request details in base64url encoded JSON format. You can encode the JSON using [this online tool](https://www.base64url.com/) or other available methods. The encoded version will appear like this:

```plaintext
ewogICAgImFjdGlvbiI6ICJCdXkiLAogICAgInF1YW50aXR5IjogNSwKICAgICJzdG9jayI6ICJOQVNEQVE6TVNGVCIKfQ
```

**request_context**

Transaction requester context. It can contain information such as the IP address of the originating user, information about the computational entity that requested the TraT, and the contextual attributes of the originating request. For this guide, we will include the IP address of the client making the trade transaction in the request context.

```json
{
  "req_ip": "192.128.0.89"
}
```

The TraT request requires request context in base64url encoded JSON format. You can encode the JSON using [this online tool](https://www.base64url.com/) or other available methods. The encoded version will appear like this:

```plaintext
ewogICJyZXFfaXAiOiAiMTkyLjEyOC4wLjg5Igp9
```

&nbsp;

### TraT Request

With the above parameters, we will get the following TraT request:


```bash
curl --location 'http://localhost:9090/token_endpoint' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'requested_token_type=urn:ietf:params:oauth:token-type:txn_token' \
--data-urlencode 'audience=https://example.org/' \
--data-urlencode 'subject_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiIsImtpZCI6ImVjZGZjNjg1MDNhMThhYjM3ZTdjNWMyYmJkMGFjNjc3In0.eyJzdWIiOnsiZm9ybWF0IjoiZW1haWwiLCJlbWFpbCI6InNoZXJsb2NrQGRldGVjdC5jb20ifSwibmFtZSI6IlNoZXJsb2NrIEhvbWVzIiwiaXNzIjoiaHR0cHM6Ly9zdG9jay10cmFkaW5nLmNvbS9zdWJqZWN0LXRva2VuLWdlbmVyYXRvciIsImV4cCI6MTcxNDk5NDQyMCwiaWF0IjoxNzE0NzgzNDA1fQ.eXIMNTilR4awBlheDcsAxW2yw2UkO8oB8-RSQKQfprKJXUwhb0MeEcqENVVtezB4jWNR7xZ1Yx_Q0yrAHOVgUA' \
--data-urlencode 'subject_token_type=urn:ietf:params:oauth:token-type:self_signed' \
--data-urlencode 'request_details=ewogICAgImFjdGlvbiI6ICJCdXkiLAogICAgInF1YW50aXR5IjogNSwKICAgICJzdG9jayI6ICJOQVNEQVE6TVNGVCIKfQ' \
--data-urlencode 'grant_type=urn:ietf:params:oauth:grant-type:token-exchange' \
--data-urlencode 'scope=stocks.trade' \
--data-urlencode 'request_context=ewogICJyZXFfaXAiOiAiMTkyLjEyOC4wLjg5Igp9'
```

You can execute this command directly from a terminal or from tools such as Postman to perform the TraT request.

&nbsp;

### TraT Response

A successful TraT response contains the following:

**token_type**

Set to

```plaintext
N_A
```

**issued_token_type**

Set to

```plaintext
urn:ietf:params:oauth:token-type:TraT
```

**access_token**

Set to the **TraT** generated by the request.

&nbsp;

In this guide, we will get the below TraT response:

```json
{
    "token_type": "N_A",
    "issued_token_type": "urn:ietf:params:oauth:token-type:txn_token",
    "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IlFOVkVUMTh5ZStRT0UvdVVsa1hFa3c9PSIsInR5cCI6InR4bl90b2tlbiJ9.eyJhdWQiOiJodHRwczovL2V4YW1wbGUub3JnLyIsImF6ZCI6eyJhY3Rpb24iOiJCdXkiLCJxdWFudGl0eSI6NSwic3RvY2siOiJOQVNEQVE6TVNGVCJ9LCJleHAiOjE3MTUyMzAxMzMsImlhdCI6MTcxNTIzMDExOCwiaXNzIjoiaHR0cHM6Ly9leGFtcGxlLm9yZy90dHMiLCJwdXJwIjoic3RvY2tzLnRyYWRlIiwicmN0eCI6eyJyZXFfaXAiOiIxOTIuMTI4LjAuODkifSwic3ViIjp7ImVtYWlsIjoic2hlcmxvY2tAZGV0ZWN0LmNvbSIsImZvcm1hdCI6ImVtYWlsIn0sInR4biI6ImUzNDAyNWI5LTI2MjQtNGY1ZC1iZDFhLTgzZGVmODM3OWViZSJ9.kse_HrxLxgI583Z5uez3jBX5ylgR7oFTSwCIv5zsuYGGiBpDn-OBPmvEVyxIcT2dZC5WD82AMRjUrTy4wFquZReeuIxminMTqocWBl2v4Pu8uLvcEnH-Tv9qa8MtgXcU0a-xELWGulqe2UAUv_3Mi0Wb20QMgURcFaFcT7ccZXX_xHsrsboKavS7H_bWhEcwR3FvXKt1YwY3zDXiUHZaxjqGTqrv0V8wDFSZmnVFapT5RRH2tarYOmuKwv3MbdaXT0ZHBvQ2S2fBzOZ47WaTiTv9sKB-N8am94J3I_bo15dCauOm_bXZGT5ybbuBAp23D3297ARIYl74Xf_MJcapKA"
}
```

where `access_token` is set to the TraT. It will have the below body:

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

For details on TraT claims check [this guide](#).

&nbsp;

After successfully generating TraTs (TraTs), you might want to verify them. For detailed instructions on this next step, please visit [Verifying TraTs](/docs/quickstart/verifying-trats).


This quick start guide provides the basic steps to get Tratteria up and running for initial learning and evaluation purposes. Remember to adjust the configurations as per the requirements for any setup beyond initial evaluations. Check [this guide](#) for detailed instructions on configuring and integrating production-level Tratteria.
