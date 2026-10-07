# TEST-009 — TriggerPanelEmergencyButton

## Test Information

**Test ID:** TEST-009  
**Endpoint:** `triggerpanelemergencybutton`  
**Operation:** Panel Emergency Keys  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate remote activation of the emergency functions exposed by the alarm panel:

- Fire
- Medical
- Panic

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/triggerpanelemergencybutton
```

---

## 3. Authentication

```http
Content-Type: application/json
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

---

## 4. Documented Emergency Keys

The API defines:

- `EmergencyKey: 1` → Fire
- `EmergencyKey: 2` → Medical
- `EmergencyKey: 3` → Panic

---

## 5. Test A — Fire Emergency

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "EmergencyKey": 1
}
```

### API Response

```json
{
  "Success": true,
  "ErrorCode": 0,
  "ErrorString": "OK",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

### Observed Result

The Fire emergency was successfully activated.

The alarm panel entered an alarm condition and produced an audible indication.

The event was also observed in the monitoring/event system as a fire condition.

Observed event:

```text
FA0000
Fire alarm condition
```

A corresponding restoration event was subsequently observed:

```text
FH0000
Fire alarm condition restored
```

**Result:** PASS / VALIDATED

---

## 6. Test B — Medical Emergency

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "EmergencyKey": 2
}
```

### API Response

```json
{
  "Success": true,
  "ErrorCode": 0,
  "ErrorString": "OK",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

### Observed Result

The Medical emergency was successfully activated.

No audible indication was observed on the tested panel.

The emergency event was received by the monitoring/event system.

Observed event:

```text
MA0000
Medical / emergency assistance request
```

A corresponding restoration event was subsequently observed:

```text
MH0000
Medical emergency condition restored
```

**Result:** PASS / VALIDATED

---

## 7. Test C — Panic Emergency

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "EmergencyKey": 3
}
```

### API Response

```json
{
  "Success": true,
  "ErrorCode": 0,
  "ErrorString": "OK",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

### Observed Result

The Panic emergency function was successfully activated.

No audible indication was observed on the tested panel.

**Result:** PASS / VALIDATED

---

## 8. Observed Behavior Summary

| EmergencyKey | Function | API Result | Panel Behavior | Validation |
|---|---|---|---|---|
| 1 | Fire | Success | Audible alarm / alarm condition | PASS |
| 2 | Medical | Success | No audible indication observed | PASS |
| 3 | Panic | Success | No audible indication observed | PASS |

---

## 9. Validation Result

**PASS / VALIDATED**

The endpoint successfully activated all three emergency functions supported by the tested panel:

- 1 → Fire
- 2 → Medical
- 3 → Panic

The API returned:

```text
Success: true
ErrorCode: 0
ErrorString: OK
```

for the tested operations.

Fire produced an audible alarm condition on the tested installation.

Medical and Panic were successfully activated without an audible indication being observed on the panel.

The monitoring/event system independently confirmed receipt of emergency events during testing.

---

## 10. Security Notes

The following values were redacted:

- `AccessToken`
- `IMEI`
- Account identifiers