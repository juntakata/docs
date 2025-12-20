# How to acquire tokens in Entra

## Authorization code grant flow (non PKCE)

### Authorize request

```http
GET https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/authorize?client_id=fbe7c3f4-06c1-40cb-bbd7-5399d79da22c&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid%20profile%20offline_access&response_type=code
```

### Token request with an authorization code (non PKCE)

```http
POST https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=fbe7c3f4-06c1-40cb-bbd7-5399d79da22c
&scope=openid profile offline_access
&code={removed}
&redirect_uri=https://jwt.ms
&grant_type=authorization_code
&client_secret={removed}
```

### Token request with a refresh token (non PKCE)

```http
POST https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=fbe7c3f4-06c1-40cb-bbd7-5399d79da22c
&scope=openid profile offline_access
&refresh_token={removed}
&redirect_uri=https://jwt.ms
&grant_type=refresh_token
&client_secret={removed}
```

## Authorization code grant flow (PKCE for SPA)

### Authorize request

```http
GET https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/authorize?client_id=070fea46-48c5-41d1-89dc-15bffd345251&response_type=code&redirect_uri=https%3A%2F%2Fjwt.ms&response_mode=query&scope=openid%20User.Read&state=12345&code_challenge=jhv8uMaK9tHBjt-TTvTmXDtVQp-_AyUitWAe3CM-Pdk&code_challenge_method=S256
```

### Token request with an authorization code (SPA)

```http
POST https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded
Origin: login.microsoftonline.com

client_id=070fea46-48c5-41d1-89dc-15bffd345251
&scope=openid User.Read
&code={removed}
&redirect_uri=https://jwt.ms
&grant_type=authorization_code
&code_verifier=ChjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

## ROPC

```http
POST https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=436345e0-ffa1-4c87-b0d3-6b1e0f5b4145
&scope=User.Read openid
&username=testuser@contoso.com
&password={removed}
&grant_type=password
```

## Client credentials grant flow

### Request

```http
POST https://login.microsoftonline.com/contoso.com/oauth2/v2.0/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

client_id=d141b9a5-2ba3-42f3-911c-8a20e6f50102
&grant_type=client_credentials
&scope=https://graph.microsoft.com/.default
&client_secret={removed}
```

## OBO (On-behalf-of) flow

### Sample application information

```text
App Display Name: Native app 01
Client ID: b253d6dc-a157-4f17-9c14-a5aa40c5db06
Tenant ID: 829b5a7b-10a1-4eb3-91c8-4d6ebf3f59e5
```

```text
App Display Name: Web API 01
Client ID: 85263a10-c7b5-442d-b181-c59d7bef727a
Tenant ID: 829b5a7b-10a1-4eb3-91c8-4d6ebf3f59e5
Application ID URI: api://85263a10-c7b5-442d-b181-c59d7bef727a
Client Secret: {removed}
```
```text
App Display Name: Web API 02
Client ID: 638325e1-116a-462d-9517-ffea47a34ad3
Tenant ID: 829b5a7b-10a1-4eb3-91c8-4d6ebf3f59e5
Application ID URI: api://638325e1-116a-462d-9517-ffea47a34ad3
```

### Authorize request

```http
GET https://login.microsoftonline.com/contoso.com/oauth2/v2.0/authorize?client_id=b253d6dc-a157-4f17-9c14-a5aa40c5db06&response_type=code&redirect_uri=https%3A%2F%2Fjwt.ms&response_mode=query&scope=api%3A%2F%2F85263a10-c7b5-442d-b181-c59d7bef727a%2FFiles.Read&state=12345
```

### Token Request that acquires an access token for Web API 01

```http
POST https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

client_id=b253d6dc-a157-4f17-9c14-a5aa40c5db06
&grant_type=authorization_code
&scope=api://85263a10-c7b5-442d-b181-c59d7bef727a/Files.Read
&code={removed}
&redirect_uri=https://jwt.ms
```

### Token Request that acquires another access token on behalf of the user

```http
POST https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded
    
grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&client_id=85263a10-c7b5-442d-b181-c59d7bef727a
&client_secret={removed}
&assertion={removed}
&scope=api://638325e1-116a-462d-9517-ffea47a34ad3/Files.Read
&requested_token_use=on_behalf_of
```

If the token request for the client ID 85263a10-c7b5-442d-b181-c59d7bef727a is blocked or interrupted by Conditional Access policies, Entra ID returns the following response.

```json
{
  "error": "invalid_grant",
  "error_description": "AADSTS53003: Access has been blocked by Conditional Access policies. The access policy does not allow token issuance. Trace ID: ed6b9266-dbc4-464b-b104-aef436513100 Correlation ID: a4c1f60b-9262-4643-8664-5914de5cdac4 Timestamp: 2025-12-20 16:18:04Z",
  "error_codes": [
    53003
  ],
  "timestamp": "2025-12-20 16:18:04Z",
  "trace_id": "ed6b9266-dbc4-464b-b104-aef436513100",
  "correlation_id": "a4c1f60b-9262-4643-8664-5914de5cdac4",
  "error_uri": "https://login.microsoftonline.com/error?code=53003",
  "suberror": "message_only",
  "claims": "{\"access_token\":{\"capolids\":{\"essential\":true,\"values\":[\"478d99cb-b03b-42d3-aa34-cf1ff08ab3ab\",\"14c76d6e-7249-4394-b876-56562726946d\"]}}}"
}
```

In this case, the client needs to send the following authorize request with the given claims parameter. The claims string needs to be unescaped and URL encoded.

```http
GET https://login.microsoftonline.com/contoso.onmicrosoft.com/oauth2/v2.0/authorize?client_id=b253d6dc-a157-4f17-9c14-a5aa40c5db06&response_type=code&redirect_uri=https%3A%2F%2Fjwt.ms&response_mode=query&scope=api%3A%2F%2F85263a10-c7b5-442d-b181-c59d7bef727a%2FFiles.Read&state=12345&claims=%7B%22access_token%22%3A%7B%22capolids%22%3A%7B%22essential%22%3Atrue%2C%22values%22%3A%5B%22478d99cb-b03b-42d3-aa34-cf1ff08ab3ab%22%2C%2214c76d6e-7249-4394-b876-56562726946d%22%5D%7D%7D%7D
```
