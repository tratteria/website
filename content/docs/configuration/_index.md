---
title: "Configuration Guide"
weight: 3
toc: true
---

This guide provides detailed explanations for Tratteria configuration parameters.

&nbsp;

- [Overview](#overview)

- [Example Configuration File](#example-configuration-file)

- [Configuration Parameters](#configuration-parameters)

  - [Issuer and Audience](#issuer-and-audience-required)

  - [Token](#token-required)

  - [Keys](#keys-optional)

  - [SPIFFE](#spiffe-optional)

  - [Subject Tokens](#subject-tokens-required)

  - [Access Evaluation](#access-evaluation-optional)

- [Environment Variables](#environment-variables)

&nbsp;

### Overview

Tratteria is configured using a YAML file, which is passed as a command line argument when starting Tratteria. This file supports configuration through environmental variables and JSON path expressions. 

&nbsp;

### Example Configuration File

```yaml
# Tratteria Example Configuration
#
# This file includes detailed configuration options with comments explaining each setting.
# Customize this configuration according to your environment needs.



# The issuer URL indicates the TraT server that issues the token.
issuer: https://example.org/tts

# The audience URL defines the intended recipient of the token.
audience: https://example.org/

# Token settings.
token:
  lifeTime: "15s"  # The lifetime of the token.

# Keys for token signing (optional). Automatically generated if not provided. JWKS is available at ./well-known/jwks.json.
# If decided to generated and distribute keys manually, see: https://github.com/SGNL-ai/Tratteria/blob/main/example-application/deployments/kubernetes/tratteria/deploy.sh as reference.
keys:
  privateKey: ${PRIVATE_KEY}
  jwks: ${JWKS}
  keyID: ${KEY_ID}

# SPIFFE settings (optional). If omitted, ensure an alternative method is used to securely authenticate requesting workloads.
spiffe:
  endpoint_socket: unix:///run/spire/sockets/agent.sock  # SPIFFE endpoint socket.
  serviceID: spiffe://example.org/tts                     # Tratteria SPIFFE ID.
  authorizedServiceIDs:                                   # List of SPIFFE IDs that are authorized to request TraTs.
    - spiffe://example.org/gateway

# Configuration for subject tokens (at least one type must be provided).
subjectTokens:
  OIDC:  # OpenID Connect Token (optional).
    clientId: example-client                     # Client ID for OIDC.
    providerURL: http://example.org/oidcprovider  # OIDC provider URL.
    subjectField: email                          # Claim in ID token used as the subject identifier.
  selfSigned:  # Self-signed token (optional).
    validation: true                             # Toggle for self-signed tokens signature validation. Ensure this is enabled in production settings.
    jwksEndpoint: http://example.org/oidcprovider/.well-known/jwks.json  # JWKS endpoint for self-signed token keys.

# Toggle to enable access evaluation.
enableAccessEvaluation: true

# Access evaluation API configuration (optional).
accessEvaluationAPI:
  endpoint: https://example.authzen.com/access/v1/evaluation  # Endpoint for access evaluation.
  authentication: 
    method: Bearer  # Authentication method.
    token:
      value: ${AUTHORIZATION_API_BEARER_TOKEN}  # Token value for Bearer authentication.

#requestMapping: Define how to construct the request body for the Authorization API using JSON path expressions. This mapping pulls specific values from
#the transaction-token request (subject_token, scope, request_details, and request_context) using JSON path expression and assigns them to respective
#fields below to construct the JSON request body for the Authorization API.
  requestMapping:
    subject:
      id: "$.subject_token.email"
    action:
      name: "$.scope"
    resource:
      stockID: "$.request_details.stockID"
      transactionID: "$.request_details.transactionID"
    context: "$.request_context"
```

&nbsp;

### Configuration Parameters

##### Issuer and Audience (Required)

The `issuer` field represents the Tratteria server that issues the TraTs. This is used as the issuer i.e `iss` claim of the generated TraTs. The `audience` field represents the intended recipient i.e `aud` claim of the generated TraTs.

##### Token (Required)

Configuration for TraTs. It includes the following:

  - **Life Time (Required)**  
    Defines the duration for which the generated TraTs are valid. The value chosen should align with your application's API response times, and should generally be in the order of seconds. Longer-lasting tokens are less secure. A good approach is to match the token’s lifetime to the API timeout value. For instance, if APIs requests time out after 15 seconds, setting the token lifetime to 15 seconds is advisable. The token lifetime should not exceed your API timeout duration.

##### Keys (Optional)

Keys used for signing TraTs. This is an optional field. When configured, it includes the following:

  - **privateKey (Required)**  
    Specifies the private key used for signing tokens. It should be provided in the Base64-encoded format, representing the PEM-encoded key data. It is recommended to set this via an environment variable for security.

  - **jwks (Required)**  
    Public Key JSON Web Key Sets (JWKS).
    
  - **keyID (Required)**  
    A unique identifier for the key.


If this configuration is not provided, the key is automatically generated by Tratteria. This configuration is useful if you want to distribute the public key through your infrastructure. In such cases, you can generate the key separately, supply it through the configuration, and distribute the public key using your infrastructure. For this, you can check out the example application's [Tratteria deployment](https://github.com/SGNL-ai/tratteria/tree/main/example-application/deployment/kubernates/tratteria), which generates the key seperately and distributes the public key using its Kubernetes infrastructure.

For both supplied and automatically generated keys, the public key JSON Web Key Sets (JWKS) is available at the `GET /.well-known/jwks.json` endpoint. If you are not providing this configuration, you can access the public key through this endpoint.

##### SPIFFE (Optional)

Configuration for SPIFEE service-to-service trust. Tratteria supports authenticating the TraT requesting workloads using SPIRE. This is an optional field. When configured, it includes the following:

  - **endpoint_socket (Required)**  
    Specifies the Unix socket for the SPIFFE endpoint.

  - **serviceID (Required)**  
    SPIFFE ID for Tratteria, identifying it within the SPIFFE framework.

  - **authorizedServiceIDs (Required)**  
    SPIFFE IDs authorized to request TraTs. This list should include the SPIFFE ID of the Gateway, API microservice, Load Balancer, or any other service that receives external API calls and generates the initial TraT. Additionally, the list should include the SPIFFE IDs of any other service that needs to request replacement TraTs.

If you're not using SPIFFE and not providing this configuration, make sure to use an alternative method to securely authenticate requesting workloads. Avoid insecure mechanisms like long-lived shared secrets.

##### Subject Tokens (Required)

Subject tokens are required for uniquely identifying the individual, entity, or user involved in a transaction. The TraT request includes subject token. Tratteria validates the subject token and determine the value to specify as the `sub` of the issued TraT. As of now, Tratteria supports OIDC ID tokens and self-signed JWTs as subject tokens. Configuration for at least one method should be provided.

  - **OIDC (Optional):**  
    OIDC ID tokens serve as a means to uniquely identify users within an OpenID Connect (OIDC) framework. When OIDC ID tokens are used in TraT requests as subject tokens, the requester must set the `subject_token_type` to `urn:ietf:params:oauth:token-type:id_token`.

    This configuration is optional. When configured, it includes the following:

      - **clientId (Required)**  
        The client application identifier within the OIDC framework.

      - **providerURL (Required)**  
        The URL of the OIDC provider.

      - **subjectField (Required)**  
        The claim or field of the OIDC ID token to be used as `sub` in the issued TraT.

    [Example application](https://github.com/SGNL-ai/tratteria/tree/main/example-application) utilizes OIDC ID tokens for subject tokens. Refer to it for guidance on configuration and its usage as the subject token in the TraT request.

  - **selfSigned (Optional):**  
    Self-signed JWTs can also be used as subject tokens. In this case, the requester must set the `subject_token_type` in the TraT request to `urn:ietf:params:oauth:token-type:self_signed`. The self-signed JWTs must contain the following claims:

      - **iss:** The unique identifier of the requesting workload. Tratteria utilizes this value in determining the `req_wl` value in the issued TraT.

      - **sub:** The subject for whom the TraT is being requested. Tratteria use this value for the `sub` value in the issued TraT.

      - **aud:** The unique identifier of the Tratteria. Tratteria verifies that this value matches its configured `issuer` value.

      - **iat:** The time at which the self-signed JWT was created.

      - **exp:** The expiration time for the self-signed JWT.

      Additionally, the self-signed JWTs may contain other claims.

      Tratteria extracts the `sub` claim of the self-signed JWTs into the `sub` claim of the issued TraT.
      
    This is an optional configuration. When configured, it includes the following:

      - **validation (Required)**  
        Determines whether validation of the self-signed JWTs is enabled (`true`) or disabled (`false`). Ensure this is enabled in production settings.

      - **jwksEndpoint (Required)**  
        The URL where the JSON Web Key Set (JWKS) can be found for validating self-signed JWTs signatures.

    Check the [self-signed JWTs](https://www.ietf.org/archive/id/draft-ietf-oauth-transaction-tokens-01.html) in the TraT draft specification for more details.

##### Access Evaluation (Optional)
Tratteria supports access evaluation of transactions before issuing TraTs. Tratteria supports [AuthZen](https://openid.github.io/authzen/#name-access-evaluations-api) access-evaluation API.

This is an optional configuration. When configured, it requires the following:

  - **endpoint (Required)**  
    The URL of the access-evaluation API endpoint.

  - **authentication (Required)**  
    The authentication method for the access-evaluation API. It includes the following:
    
      - **method (Required)**  
        The authentication method for the access-evaluation API. Currently, only `Bearer` token method is supported.

      - **token (Required)**  
        Specifies the token to be used with the authentication method. Currently, only `value` is supported. For security reasons, it is recommended to set this via an environment variable.

  - **requestMapping (Required)**  
    Specifies how to construct the request body for the access-evaluation API using JSON path expressions and YAML fields. The configuration allows for the construction of arbitrary JSONs using TraTs request components: `subject_token`, `scope`, `request_details`, and `request_context`. If a particular JSON path does not exist for a request, the field is omitted.

&nbsp;

### Environment Variables

Use environment variables to securely handle sensitive information, such as private keys and API tokens. These variables are automatically resolved at runtime when Tratteria starts. Tratteria reads the environment variables without writing them to the configuration file.


&nbsp;

For a practical example of a configuration setup, refer to the example application [configuration file](https://github.com/SGNL-ai/tratteria/tree/main/example-application/deployment/kubernates/tratteria/config.yaml).