# TEST-004 — RegenerateAccessToken

## Test Information

**Test ID:** TEST-004  
**Endpoint:** `RegenerateAccessToken`  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate generation of a new access-token pair using the refresh token without repeating username/password authentication.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RegenerateAccessToken
```

## 3. Authentication

The refresh token was sent in the `M2MOAuth2Token` header.

```http
M2MOAuth2Token: <REDACTED_REFRESH_TOKEN>
Content-Type: application/json
```

---

## 4. Request

```json
{}
```

The documented optional `UserId` field was not required for this test.

---

## 5. Observed Result

The API returned HTTP 200 and:

```text
Success: true
ErrorCode: 0
AdminLogin: false
AccessToken: returned
RefreshToken: returned
ExpirationDate: returned
BypassToken: null
```

Both returned tokens were non-empty.

---

## 6. Validation Result

**PASS / VALIDATED**

Validated:

- refresh-token authentication through `M2MOAuth2Token`;
- token regeneration without username/password;
- generation of a new `AccessToken`;
- generation of a new `RefreshToken`;
- return of a new `ExpirationDate`;
- operation with an empty JSON object and no explicit `UserId`.

Token lifetime and rotation semantics beyond this successful regeneration were not measured in this test.

---

## 7. Security Notes

Access tokens, refresh tokens and account identifiers are redacted from public evidence.
