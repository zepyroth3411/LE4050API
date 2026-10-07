# TEST-007E — RemoteArm / Output ON-OFF

## Test Information

**Test ID:** TEST-007E  
**Endpoint:** `RemoteArm`  
**Operation:** Output / PGM ON-OFF  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate remote activation and deactivation of the controllable output returned as `COMM. OUTPUT 1`.

---

## 2. Output Under Test

Device discovery returned:

```text
Name: COMM. OUTPUT 1
DevicePIN: 1
Enabled: true
OnPINRequired: false
```

The same output was visible as a controllable output/PGM in the official M2M application.

---

## 3. Attempt Without User PIN

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 3,
  "OutPIN": "1",
  "OutDelay": "1",
  "ProtocolNumber": 4
}
```

Response:

```text
Success: false
ErrorCode: -1234
ErrorString: ERROR: PIN REQUIRED
```

The output was not activated.

---

## 4. Output ON

The request was repeated with the panel user PIN:

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 3,
  "OutPIN": "1",
  "OutDelay": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "ProtocolNumber": 4
}
```

`COMM. OUTPUT 1` activated successfully.

**Result:** PASS / VALIDATED

---

## 5. Output OFF

The output was then deactivated using:

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 4,
  "OutPIN": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "ProtocolNumber": 4
}
```

`COMM. OUTPUT 1` deactivated successfully.

**Result:** PASS / VALIDATED

---

## 6. Observed PIN Requirement

Discovery reported:

```text
OnPINRequired: false
```

Execution without `UserPIN` returned:

```text
ErrorCode: -1234
ERROR: PIN REQUIRED
```

Execution succeeded after `UserPIN` was included. Both values are retained as observed behavior.

---

## 7. Validation Result

**PASS / VALIDATED**

Validated:

- `ArmingState: 3` activates the selected output;
- `ArmingState: 4` deactivates it;
- `OutPIN: "1"` selects `COMM. OUTPUT 1`;
- physical output control works;
- the tested execution path required `UserPIN`.

---

## 8. Security Notes

Access token, IMEI and panel PIN are redacted.
