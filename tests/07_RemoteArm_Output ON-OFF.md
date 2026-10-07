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

Validate remote activation and deactivation of a controllable output reported by the API as `COMM. OUTPUT 1`.

---

## 2. Output Under Test

The device discovery response previously identified:

```text
Name:          COMM. OUTPUT 1
DevicePIN:     1
Enabled:       true
OnPINRequired: false
```

The same output is exposed as a controllable PGM/output in the official M2M application.

---

## 3. First Attempt — Without User PIN

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 3,
  "OutPIN": "1",
  "OutDelay": "1",
  "ProtocolNumber": 4
}
```

### Result

```text
Success:     false
ErrorCode:   -1234
ErrorString: ERROR: PIN REQUIRED
```

The output was not activated.

---

## 4. Output ON

The request was repeated with the panel user PIN.

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

### Physical Result

`COMM. OUTPUT 1` was successfully activated.

- `ArmingState: 3` = Output ON
- `OutPIN: "1"`    = COMM. OUTPUT 1

**Status:** PASS / VALIDATED

---

## 5. Output OFF

The output was subsequently deactivated using:

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 4,
  "OutPIN": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "ProtocolNumber": 4
}
```

### Physical Result

`COMM. OUTPUT 1` was successfully deactivated.

- `ArmingState: 4` = Output OFF
- `OutPIN: "1"`    = COMM. OUTPUT 1

**Status:** PASS / VALIDATED

---

## 6. Important Observation

During device discovery, the output reported:

- `OnPINRequired: false`

However, execution without `UserPIN` returned:

```text
ErrorCode: -1234
ERROR: PIN REQUIRED
```

After adding `UserPIN`, output control succeeded.

This creates a discrepancy between the discovery metadata and the actual execution requirement.

### Classification

| Item | Status |
|---|---|
| Output discovery | VALIDATED |
| Output ON | VALIDATED |
| Output OFF | VALIDATED |
| PIN required in execution | VALIDATED |
| `OnPINRequired: false` | OBSERVED |
| Metadata/runtime mismatch | OPEN QUESTION |

---

## 7. Validation Result

**PASS**

Validated:

- `ArmingState: 3` activates the output;
- `ArmingState: 4` deactivates the output;
- `OutPIN: "1"` selects `COMM. OUTPUT 1`;
- physical output control works;
- `UserPIN` was required by the real execution path.

---

## 8. Open Question for API Provider

Why does the device discovery response report:

- `OnPINRequired: false`

while the actual `RemoteArm` output command requires `UserPIN` and returns `-1234 PIN REQUIRED` when it is omitted?