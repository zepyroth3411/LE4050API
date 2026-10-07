# TEST-008 — RemoteBypassExt / Bypass Zone

## Test Information

**Test ID:** TEST-008  
**Endpoint:** `RemoteBypassExt`  
**Operation:** Bypass Zone  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** NOT VALIDATED  

---

## 1. Objective

Validate remote bypass of an alarm zone using the `RemoteBypassExt` endpoint.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RemoteBypassExt
```

---

## 3. Authentication

```http
Content-Type: application/json
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

A valid `AccessToken` was used.

The `AccessToken` was also regenerated during testing and the operation was repeated with the newly generated token.

---

## 4. Documented Request

The endpoint accepts:

- `IMEI`
- `SerialNumber`
- `ProtocolNumber`
- `ICCID`
- `AlarmUserId`
- `UserPIN`
- `Partition`
- `BypassZonesId`
- `UnbypassZonesId`

For bypassing zones:

- `BypassZonesId` = array of zone IDs to bypass

For restoring zones:

- `UnbypassZonesId` = array of zone IDs to remove from bypass

---

## 5. Test 1 — Documented Request Structure

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "Partition": "1",
  "BypassZonesId": [1],
  "UserPIN": "<REDACTED_USER_PIN>"
}
```

### Response

```json
{
  "ExternalDevices": null,
  "Success": false,
  "ErrorCode": -1002,
  "ErrorString": "ERROR: INCOMINMG DATA",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

---

## 6. Test 2 — With ProtocolNumber

The request was repeated including:

```json
{
  "ProtocolNumber": 4
}
```

### Result

The operation returned the same error:

- `Success: false`
- `ErrorCode: -1002`
- `ErrorString: ERROR: INCOMINMG DATA`

---

## 7. Test 3 — With AlarmUserId

The request was repeated including the alarm user identifier.

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "AlarmUserId": "<REDACTED_ALARM_USER_ID>",
  "Partition": "1",
  "BypassZonesId": [1],
  "UserPIN": "<REDACTED_USER_PIN>"
}
```

### Response

```json
{
  "ExternalDevices": null,
  "Success": false,
  "ErrorCode": -1002,
  "ErrorString": "ERROR: INCOMINMG DATA",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

---

## 8. Token Validation

The `AccessToken` was regenerated and the request was repeated using the new `AccessToken`.

The result remained unchanged:

- `ErrorCode: -1002`
- `ErrorString: ERROR: INCOMINMG DATA`

---

## 9. Validation Result

**NOT VALIDATED**

The endpoint was reachable and returned HTTP 200, but zone bypass was not completed successfully.

Observed application response:

```text
Success: false
ErrorCode: -1002
ErrorString: ERROR: INCOMINMG DATA
```

No zone state change was produced during the tested requests.

---

## 10. Tested Variations

| Variation | Result |
|---|---|
| Documented request structure | -1002 |
| With ProtocolNumber: 4 | -1002 |
| Without ProtocolNumber | -1002 |
| With AlarmUserId | -1002 |
| New AccessToken | -1002 |

---

## 11. Security Notes

The following values were redacted from this test document:

- `AccessToken`
- `IMEI`
- `UserPIN`
- `AlarmUserId`