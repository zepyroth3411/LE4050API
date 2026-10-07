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

Validate the documented `RemoteArm` behavior that should arm the alarm system in Away mode while automatically bypassing zones that remain open.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RemoteArm
```

---

## 3. Request

### Headers

```http
Content-Type: application/json
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

### Body

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

## 4. Documented Behavior

According to the API specification:

- `ArmingState: 6` = Arm Away and bypass open zones

The documented behavior indicates that open zones should be bypassed automatically so that the system can still complete the arming process.

---

## 5. Initial Test Condition

The partition was not ready to arm.

Observed partition state:

- `DeviceState:    2`
- `DeviceStateExt: 7`

According to the API specification:

- `DeviceStateExt 7 = NOT READY TO ARM`

The following zones were open:

- Zones 001–008 → OPEN
- Zones 009–016 → CLOSED

The open zones reported:

- `ZoneState: 2`

---

## 6. First Arm-With-Bypass Attempt

The `RemoteArm` request was sent using:

- `ArmingState: 6`

### Result

```text
Success:     false
ErrorCode:   -2510
ErrorString: ERROR: FORCE ARM FAILED, ALARM IS NOT READY
```

The system did not arm.

The open zones remained in their original condition.

The API did not automatically bypass the zones during this attempt.

---

## 7. Additional Zone Configuration Test

To rule out zone-type restrictions as a possible cause, the open zones were reconfigured on the alarm panel as:

- `Zone Definition: 03 — Instant`

These zones were selected because they can normally be bypassed manually from the alarm panel.

After changing the zone definitions, the same request was repeated:

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 6,
  "PartitionNumber": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "ProtocolNumber": 4
}
```

### Result

The API returned the same failure:

```text
Success:     false
ErrorCode:   -2510
ErrorString: ERROR: FORCE ARM FAILED, ALARM IS NOT READY
```

Changing the zones to a bypassable zone definition did not cause `ArmingState: 6` to automatically bypass them.

---

## 8. Manual Bypass Control Test

As a control test, the same open zones were manually bypassed before sending the arm command.

After the zones had already been bypassed, the system was armed.

### Result

The alarm panel successfully entered Away Arm mode.

The system reported:

```text
System Armed in Away Mode
```

This confirms that:

- the zones themselves can be bypassed;
- the alarm panel can arm successfully once those zones are bypassed;
- Away arming works through the remote-control path;
- the failure is specifically associated with the automatic bypass behavior expected from `ArmingState: 6`.

---

## 9. Observed Behavior

The tested behavior was:

```text
Open bypassable zones
        ↓
RemoteArm / ArmingState 6
        ↓
Automatic bypass NOT performed
        ↓
ERROR -2510
FORCE ARM FAILED, ALARM IS NOT READY
```

However:

```text
Open zones
        ↓
Manual bypass
        ↓
Remote arm
        ↓
System Armed in Away Mode
```

---

## 10. Interpretation

The documented automatic-bypass behavior of:

- `ArmingState: 6`

was not reproduced on the tested installation.

The test confirms that the alarm panel itself supports bypassing these zones and can successfully arm in Away mode after they are manually bypassed.

Therefore, the observed failure cannot be attributed only to the zones being inherently non-bypassable.

The available response does not establish why the API does not perform the automatic bypass.

Possible implementation requirements or restrictions should not be assumed without confirmation from the API provider.

---

## 11. Observed Error

The following error was consistently returned when attempting automatic force arming while zones remained open:

```text
ErrorCode: -2510
ErrorString: ERROR: FORCE ARM FAILED, ALARM IS NOT READY
```

This error was observed in testing.

Its exact documented meaning or required remediation was not found in the available API specification.

---

## 12. Validation Result

**PARTIAL / DOCUMENTED BEHAVIOR NOT REPRODUCED**

Validated:

- `ArmingState: 6` request is accepted by the API;
- the request reaches the alarm-control workflow;
- open zones produce a force-arm attempt;
- failure is returned as `ErrorCode: -2510`;
- tested zones are manually bypassable;
- the panel can successfully arm in Away mode after manual bypass;
- the panel reports `System Armed in Away Mode` after successful arming.

Not validated:

- automatic bypass of open zones through `ArmingState: 6`.

---

## 13. Documentation Discrepancy

The API specification describes:

- `ArmingState: 6` = Arm everything and bypass open zones

Observed behavior on the tested configuration:

```text
ArmingState: 6
        ↓
Open zones remain unbypassed
        ↓
Force arm fails
        ↓
ErrorCode: -2510
```

Manual bypassing of those same zones allows the system to arm successfully.

This difference should be clarified with the API provider.

---

## 14. Open Questions for API Provider

1. Does `ArmingState: 6` require any additional panel, account or communicator configuration before automatic bypass is allowed?
2. Are only specific zone types supported by the automatic-bypass operation?
3. Does automatic bypass require a particular user permission, panel user number or master-level PIN?
4. What conditions produce:
   ```text
   ErrorCode: -2510
   ERROR: FORCE ARM FAILED, ALARM IS NOT READY
   ```
5. Is automatic bypass through `ArmingState: 6` supported on:
   - LE4050M-LA
   - DSC PowerSeries NEO
   - HS2064
6. Is there a separate bypass command that must be executed before calling `RemoteArm`?

---

## 15. Security Notes

The following values were redacted:

- `AccessToken`
- `IMEI`
- `UserPIN`
- Internal IDs
- `ControllerID`

---

## 16. Test Classification

| Item | Status |
|---|---|
| ArmingState: 6 accepted | VALIDATED |
| Force-arm attempt executed | VALIDATED |
| -2510 returned when not ready | OBSERVED |
| Zones manually bypassable | VALIDATED |
| Away arm after manual bypass | VALIDATED |
| Automatic bypass through ArmingState: 6 | NOT VALIDATED |
| Reason for automatic-bypass failure | UNKNOWN |
| Manufacturer clarification required | YES |