# M2M Mobile Integration API v3

## Test Context and Validation Methodology

**Document status:** Draft  
**API:** M2M Mobile Integration API  
**API version:** v3  
**Environment:** Production  
**Base URL:** `https://app.m2mservices.com/CommonAdministrationService/api/v3`

---

## 1. Purpose

This validation is intended to document the behavior of the M2M Mobile Integration API through controlled and reproducible tests.

The validation will distinguish between:

- behavior explicitly described by the API specification;
- behavior observed during testing;
- functionality successfully validated against a real device;
- functionality that remains pending validation.

---

## 2. Validation terminology

### DOCUMENTED

Behavior explicitly described by the M2M Mobile Integration API specification.

### OBSERVED

Behavior seen during testing but not necessarily described as a formal API requirement.

### VALIDATED

Behavior successfully reproduced during a controlled test.

### PENDING VALIDATION

Functionality described by the API but not yet tested.

---

## 3. Test account strategy

The formal validation will use an end-user account associated with the test communicator.

The real username and password will not be included in this documentation.

Credentials will be represented as:

```text
UserName: <END_USER>
UserPass: <REDACTED>
```

Access tokens, refresh tokens, authorization codes and panel PINs will also be redacted.

---

## 4. Preliminary observation regarding account types

During preliminary testing, an administrative/dealer account was successfully authenticated using:

```text
AdminRequest: true
```

However, that session did not provide access to the communicator through the client/device workflow used during the tests.

An end-user account associated with the communicator was therefore selected for the formal validation.

This behavior is recorded as an observation from the current test environment and is not being treated as a general restriction of the API unless confirmed by the manufacturer.

---

## 5. Test methodology

Each endpoint will be validated independently.

For every test, the following information will be recorded:

1. Test identifier
2. Endpoint
3. Purpose
4. Relevant behavior documented by the API
5. Request method
6. Request headers
7. Request body
8. Raw response
9. Observed result
10. Interpretation
11. Validation status
12. Open questions

Sensitive information will always be redacted.

---

## 6. Evidence handling

The API response will be stored as close as possible to the original response.

The following values may be replaced for security reasons:

- `UserName`
- `UserPass`
- `AuthCode`
- `TwoFactorAuthCode`
- `AccessToken`
- `RefreshToken`
- `UserPIN`

No API field names, response structures, error codes or status values will be modified.

---

## 7. Test sequence

The initial validation sequence will be:

- **TEST-001** — CreateAuthorizationCode
- **TEST-002** — CreateAccessToken
- **TEST-003** — GetHAUserSettings
- **TEST-004** — GetAllDeviceData
- **TEST-005** — GetDeviceStatusData

Additional endpoints will be added as testing progresses.