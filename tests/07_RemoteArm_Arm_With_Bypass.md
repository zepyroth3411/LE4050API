# TEST-007C — RemoteArm / Arm With Bypass

## Test Information

**Test ID:** TEST-007C  
**Endpoint:** `RemoteArm`  
**Operation:** Arm Away With Automatic Bypass  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** PARTIAL / DOCUMENTED BEHAVIOR NOT REPRODUCED

---

## 1. Objective

Validate the documented `ArmingState: 6` behavior: Arm Away while automatically bypassing zones that remain open.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RemoteArm
```

## 3. Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 6,
  "PartitionNumber": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "ProtocolNumber": 4
}
```

---

## 4. Initial Condition

The partition was not ready to arm:

```text
DeviceState: 2
DeviceStateExt: 7
```

Observed zone state:

| Zones | State |
|---|---|
| 001–008 | OPEN |
| 009–016 | CLOSED |

---

## 5. Automatic-Bypass Attempt

The API returned:

```text
Success: false
ErrorCode: -2510
ErrorString: ERROR: FORCE ARM FAILED, ALARM IS NOT READY
```

The system did not arm and the open zones were not automatically bypassed.

---

## 6. Additional Controlled Test

The open zones were reconfigured on the alarm panel as:

```text
Zone Definition: 03 — Instant
```

These zones were manually bypassable from the alarm panel.

The same `ArmingState: 6` request was repeated and returned the same result:

```text
Success: false
ErrorCode: -2510
ErrorString: ERROR: FORCE ARM FAILED, ALARM IS NOT READY
```

---

## 7. Manual-Bypass Control Test

The same open zones were manually bypassed before arming.

After manual bypass, the alarm panel successfully entered Away Arm mode and reported:

```text
System Armed in Away Mode
```

This control test confirmed that the zones could be bypassed and that the panel could arm after the bypass condition had already been established.

---

## 8. Validation Result

**PARTIAL / DOCUMENTED BEHAVIOR NOT REPRODUCED**

Validated:

- `ArmingState: 6` request reached the alarm-control workflow;
- the API returned `-2510` while open zones remained;
- the tested zones were manually bypassable;
- Away arm succeeded after manual bypass.

Not validated:

- automatic bypass of open zones through `ArmingState: 6`.

---

## 9. Security Notes

Access token, IMEI, panel PIN and installation-specific internal identifiers are redacted.
