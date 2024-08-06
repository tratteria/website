---
title: "TratteriaConfig"
weight: 3
toc: true
---

The TratteriaConfig resource defines general configuration settings that apply to all APIs and TraTs within a specified Kubernetes namespace.

Let's delve into the details of TraTs by examining an example.

```yaml
apiVersion: tratteria.io/v1alpha1
kind: TratteriaConfig
metadata:
  name: alpha-stocks-tratteriacfg
  namespace: alpha-stocks-dev
spec:
  token:
    issuer: "https://alphastocks.com/tratteria"
    audience: "https://alphastocks.com/"
    lifeTime: "15s"
  subjectTokens:
    OIDC:
      clientId: alpha-stocks-client
      providerURL: http://dex:5556/dex
      subjectField: email
    selfSigned:
      validation: false
      jwksEndpoint: "http://alphastocks.com/oidcprovider/.well-known/jwks.json"
  accessEvaluationAPI:
    enableAccessEvaluation: false
    endpoint: "https://alphastocks.authzen.com/access/v1/evaluation"
    authentication:
      method: "Bearer"
      token:
        value: "${AUTHORIZATION_API_BEARER_TOKEN}"
  tokenGenerationAuthorizedServiceIds:
      - "spiffe://dev.alphastocks.com/gateway"
```

TratteriaConfig includes the following sections:

##### token (Required)

Configures the TraTs and includes the following parameters:

  - **issuer (Required)**:
    Represents the Tratteria server that issues TraTs, used as the `iss` claim in the generated TraTs.
    
  - **audience (Required)**:
    Represents the intended recipient of the TraTs, used as the `aud` claim in the generated TraTs.

  - **lifeTime (Required)**  
    Defines the duration for which the generated TraTs are valid. The value chosen should align with your application's API response times, and should generally be in the order of seconds. Longer-lasting tokens are less secure. A good approach is to match the token’s lifetime to the API timeout value. For instance, if APIs requests time out after 15 seconds, setting the token lifetime to 15 seconds is advisable. The token lifetime should not exceed your API timeout duration.

##### subjectTokens (Required)

Subject tokens are required for uniquely identifying the individual, entity, or user involved in a transaction. The TraT token request includes a subject token. Tratteria validates the subject token and determines the value to specify as the `sub` of the issued TraT. As of now, Tratteria supports OIDC ID tokens and self-signed JWTs as subject tokens. Configuration for at least one method should be provided.

  - **OIDC (Optional):**  
    OIDC ID tokens serve as a means to uniquely identify users within an OpenID Connect (OIDC) framework. When OIDC ID tokens are used in TraT token requests as subject tokens, the requester must set the `subject_token_type` to `urn:ietf:params:oauth:token-type:id_token`.

    This configuration is optional. When configured, it includes the following:

      - **clientId (Required)**  
        The client application identifier within the OIDC framework.

      - **providerURL (Required)**  
        The URL of the OIDC provider.

      - **subjectField (Required)**  
        The claim or field of the OIDC ID token to be used as `sub` in the issued TraT.

    [Tratteria example application](https://github.com/tratteria/example-application) utilizes OIDC ID tokens for subject tokens. Refer to it for guidance on configuration and its usage as the subject token in the TraT request.

  - **selfSigned (Optional):**  
    Self-signed JWTs can also be used as subject tokens. In this case, the requester must set the `subject_token_type` in the TraT token request to `urn:ietf:params:oauth:token-type:self_signed`. The self-signed JWTs must contain the following claims:

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

    Check the [self-signed JWTs](https://www.ietf.org/archive/id/draft-ietf-oauth-transaction-tokens-03.html#:~:text=7.2.1.-,Self%2DSigned%20Subject%20Token%20Type,-A%20requester%20MAY) in the TraT draft specification for more details.

##### accessEvaluationAPI (Optional)

Tratteria supports access evaluation of transactions before issuing TraTs. Tratteria supports [AuthZen](https://openid.github.io/authzen/#name-access-evaluations-api) access-evaluation API.

This is an optional configuration. When configured, it requires the following:

  - **enableAccessEvaluation (Required)**
    Toggle to enable or disable access evaluation.

  - **endpoint (Required)**  
    The URL of the access-evaluation API endpoint.

  - **authentication (Required)**  
    The authentication method for the access-evaluation API. It includes the following:
    
      - **method (Required)**  
        The authentication method for the access-evaluation API. Currently, only `Bearer` token method is supported.

      - **token (Required)**  
        Specifies the token to be used with the authentication method. Currently, only `value` is supported. For security reasons, it is recommended to set this via an environment variable.

  - **requestMapping (Required)**  
    Specifies how to construct the request body for the access-evaluation API using JSON path expressions and YAML fields. The configuration allows for the construction of arbitrary JSONs using TraTs request components: `subject_token`, `purp`, `request_details`, and `request_context`. If a particular JSON path does not exist for a request, the field is omitted.


##### tokenGenerationAuthorizedServiceIds (Required)

SPIFFE IDs of services authorized to request TraTs. This list should include the SPIFFE ID of the Gateway, API microservice, Load Balancer, or any other service that receives external API calls and generates the initial TraT.

##### Environment Variables

Use environment variables to securely handle sensitive information, such as API tokens. These variables are automatically resolved during runtime at Tratteria service. Tratteria service reads the environment variables without writing them to the configuration file.
