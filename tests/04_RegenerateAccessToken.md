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

Validate that a previously issued `RefreshToken` can be used to obtain a new access token without repeating the username/password authentication flow.

This test also validates the minimum request structure accepted by the API.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RegenerateAccessToken
```

---

## 3. Documented Behavior

According to the API specification:

- this endpoint is used when the access token must be refreshed;
- the `RefreshToken`, not the current `AccessToken`, must be sent in the `M2MOAuth2Token` HTTP header;
- the request body may be empty;
- an optional `UserId` may be supplied;
- the response returns a new authentication token set;
- an invalid refresh token returns error code `-2002`.

---

## 4. Authentication

The refresh token obtained during TEST-002 was sent as:

```http
M2MOAuth2Token: <REDACTED_REFRESH_TOKEN>
```

The access token was not used for this request.

---

## 5. Request

### Headers

```http
Content-Type: application/json
M2MOAuth2Token: <REDACTED_REFRESH_TOKEN>
```

### Body

```json
{}
```

The optional `UserId` field was not sent.

---

## 6. Sanitized Raw Response

```json
{
  "Success": true,
  "ErrorMsg": null,
  "ErrorCode": 0,
  "AdminLogin": false,
  "AccessToken": "<REDACTED_NEW_ACCESS_TOKEN>",
  "RefreshToken": "<REDACTED_NEW_REFRESH_TOKEN>",
  "ExpirationDate": "<EXPIRATION_TIMESTAMP>",
  "BypassToken": null
}
```

---

## 7. Observed Result

The API returned HTTP 200.

The response body reported:

- `Success: true`
- `ErrorCode: 0`
- `AdminLogin: false`
- `AccessToken: returned`
- `RefreshToken: returned`
- `ExpirationDate: returned`
- `BypassToken: null`

Both the new `AccessToken` and new `RefreshToken` were non-empty.

---

## 8. Interpretation

The refresh-token flow was successfully validated.

The API accepted the previously issued `RefreshToken` through the `M2MOAuth2Token` header and generated a new authentication token pair.

The minimum request:

```json
{}
```

was accepted successfully.

Therefore, `UserId` is not required for the tested refresh flow.

---

## 9. Token Rotation Observation

The API returned:

- new `AccessToken`
- new `RefreshToken`
- new `ExpirationDate`

This demonstrates that the endpoint issues a new token pair.

At this stage, it should not be assumed that the previous `RefreshToken` becomes immediately invalid after use.

The validity or invalidation behavior of previously issued refresh tokens remains:

**PENDING VALIDATION**

unless explicitly confirmed by the API provider or tested separately.

---

## 10. Additional Observed Fields

The actual response again included:

- `AdminLogin`
- `BypassToken`

Observed values:

- `AdminLogin: false`
- `BypassToken: null`

These fields have now appeared consistently in both:

- `CreateAccessToken`
- `RegenerateAccessToken`

Their exact purpose should not be inferred unless formally documented or confirmed by the API provider.

---

## 11. Validation Result

**PASS**

Validated:

- refresh-token authentication;
- use of `RefreshToken` in the `M2MOAuth2Token` header;
- empty JSON request body;
- generation of a new `AccessToken`;
- generation of a new `RefreshToken`;
- generation of a new `ExpirationDate`;
- preservation of the end-user authentication context;
- `UserId` not required in the tested scenario.

---

## 12. Security Notes

The following values were removed from the evidence:

- `RefreshToken` used in request
- New `AccessToken`
- New `RefreshToken`

Authentication tokens must not be included in manufacturer-facing documentation.

---

## 13. Authentication Flow Validated So Far

```text
CreateAuthorizationCode
        ↓
AuthCode
        ↓
CreateAccessToken
        ↓
AccessToken + RefreshToken
        ↓
Protected API calls
        ↓
RegenerateAccessToken
        ↓
New AccessToken + New RefreshToken
```