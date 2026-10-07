# M2M Mobile Integration API v3

## Test Context and Validation Methodology

**Document status:** Validation cycle completed  
**API:** M2M Mobile Integration API  
**API version:** v3  
**Environment:** Production  
**Base URL:** `https://app.m2mservices.com/CommonAdministrationService/api/v3`

---

## 1. Purpose

This repository records controlled validation of the M2M Mobile Integration API against a real alarm installation.

The test documents distinguish between:

- **DOCUMENTED** — behavior explicitly described by the API specification;
- **OBSERVED** — behavior returned or seen during testing;
- **VALIDATED** — behavior successfully reproduced during a controlled test;
- **NOT VALIDATED** — behavior that was tested but not successfully reproduced.

---

## 2. Test Environment

The validation used an end-user account associated with the test communicator.

Tested hardware reported by the API:

```text
Communicator: LE4050M-LA
Panel family: DSC PowerSeries NEO
Panel model: HS2064
Keybus mode: NEO
ProtocolNumber: 4
```

Authentication and functional requests were sent against the production API.

---

## 3. Authentication Context

The end-user authentication flow used:

```text
CreateAuthorizationCode
        ↓
CreateAccessToken
        ↓
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

`RegenerateAccessToken` was also validated using the refresh token in the same authentication header.

---

## 4. Test Methodology

Each endpoint was tested independently with the minimum request needed for the target operation. Where useful, additional controlled variants were executed to confirm error handling, optional parameters, or state transitions.

For state-changing operations, API results were correlated with the physical panel and, where available, the official M2M application or monitoring/event system.

HTTP status alone is not treated as the operation result. The effective result is taken from:

- `Success`
- `ErrorCode`
- `ErrorString` or `ErrorMsg`

---

## 5. Evidence and Redaction

Evidence is preserved as close as practical to the original API response while removing credentials and installation-specific identifiers.

Values redacted from public evidence include:

- usernames and passwords;
- authorization, access and refresh tokens;
- panel user PINs;
- IMEI and communicator serial numbers;
- SIM / ICCID identifiers;
- client, controller and user identifiers;
- account-specific UUIDs and internal installation identifiers.

API field names, error codes, status values and functional response structure are preserved.

---

## 6. Validation Matrix

| Test | Endpoint / Operation | Result |
|---|---|---|
| TEST-001 | CreateAuthorizationCode | PASS |
| TEST-002 | CreateAccessToken | PASS |
| TEST-003 | GetHAUserSettings | PASS |
| TEST-004 | RegenerateAccessToken | PASS |
| TEST-005 | GetAllDeviceData | PASS |
| TEST-006 | GetDeviceStatusData | PASS |
| TEST-007A | RemoteArm — Arm Away | PASS |
| TEST-007B | RemoteArm — Stay Arm | PASS |
| TEST-007C | RemoteArm — Arm With Bypass | PARTIAL / documented behavior not reproduced |
| TEST-007D | RemoteArm — Disarm | PASS |
| TEST-007E | RemoteArm — Output ON/OFF | PASS |
| TEST-008 | RemoteBypassExt — Bypass Zone | NOT VALIDATED |
| TEST-009 | TriggerPanelEmergencyButton | PASS |
| TEST-010 | SetExternalDeviceName | PASS |
| TEST-011 | SetExternalOutputsVisibility | PASS |

---

## 7. Repository Layout

```text
tests/      Controlled endpoint validation records
evidence/   Sanitized full responses used as supporting evidence
findings/   Cross-test findings, clarification requests and provider feedback
```

Individual test files are intended to remain factual and operation-focused. Cross-endpoint discrepancies and provider questions are kept separately in `findings/90_API_Findings_and_Feedback.md`.
