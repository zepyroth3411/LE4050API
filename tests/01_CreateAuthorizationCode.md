# TEST-001 — CreateAuthorizationCode

## Test Information

**Test ID:** TEST-001  
**Endpoint:** `CreateAuthorizationCode`  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS  

---

## 1. Objective

Validate the authentication of an end-user account and verify that the API returns a temporary authorization code that can subsequently be exchanged for an access token.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/CreateAuthorizationCode
```

---

## 3. Documented Behavior

According to the API specification:

- the request requires `UserName`, `UserPass` and `AdminRequest`;
- `AdminRequest: false` is used for an end-user/client login;
- a successful authentication returns an `AuthCode`;
- the `AuthCode` is intended to be used by `CreateAccessToken`;
- `TwoFactorCodeRequired` indicates whether a second authentication factor is required;
- operation success or failure is reported in the response body.

---

## 4. Request

### Headers

```http
Content-Type: application/json
```

### Body

```json
{
  "UserName": "<END_USER>",
  "UserPass": "<REDACTED>",
  "AdminRequest": false
}
```

---

## 5. Raw Response

```json
{
  "Success": true,
  "ErrorMsg": null,
  "ErrorCode": 0,
  "AuthCode": "<REDACTED_AUTH_CODE>",
  "AdminLogin": false,
  "TwoFactorCodeRequired": false,
  "MaskedEmail": ""
}
```

---

## 6. Observed Result

The API returned HTTP 200.

The response body reported:

- `Success: true`
- `ErrorCode: 0`
- `AdminLogin: false`
- `TwoFactorCodeRequired: false`

A non-empty `AuthCode` was returned.

No masked email address was returned.

---

## 7. Interpretation

The end-user credentials were accepted successfully.

The API created an authorization code that can be used in the next authentication step.

`AdminLogin: false` confirms that the resulting authentication context corresponds to a client/end-user session rather than an administrative session.

No two-factor authentication challenge was required for this account during this test.

---

## 8. Validation Result

**PASS**

Validated:

- end-user authentication;
- `AdminRequest: false`;
- successful authorization-code generation;
- `ErrorCode: 0`;
- non-admin authentication context;
- authentication without two-factor challenge.

---

## 9. Security Notes

The following values have been removed from the test evidence:

- `UserName`
- `UserPass`
- `AuthCode`

Real credentials and authorization codes must not be stored in public or manufacturer-facing documentation.