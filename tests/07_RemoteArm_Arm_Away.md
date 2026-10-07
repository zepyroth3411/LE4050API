# TEST-007A — RemoteArm / Arm Away

## Test Information

**Test ID:** TEST-007A  
**Endpoint:** `RemoteArm`  
**Operation:** Arm Away  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS  

---

## 1. Objective

Validate remote arming of a specific alarm partition using the `RemoteArm` endpoint.

The test also validates the behavior of the API when:

- the partition is not ready to arm;
- the partition becomes ready;
- the same arm command is then accepted successfully.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RemoteArm
```

---

## 3. Authentication

```http
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
Content-Type: application/json
```

---

## 4. Documented Behavior

According to the API specification:

- `ArmingState 0` = Disarm
- `ArmingState 1` = Arm Away / Arm everything
- `ArmingState 2` = Stay arm
- `ArmingState 6` = Arm Away + bypass open zones
- `ArmingState 7` = Stay arm + bypass open zones

`PartitionNumber` identifies the partition to arm or disarm.

If `PartitionNumber` is omitted, the operation applies to the whole system.

A `UserPIN` may be required depending on the panel configuration.

---

## 5. Request

The validated Arm Away request used:

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 1,
  "PartitionNumber": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "UserNumber": "<REDACTED_USER_NUMBER>",
  "ProtocolNumber": 4
}
```

---

## 6. Scenario A — Panel Not Ready

### Initial Condition

The alarm partition was not ready to arm due to its current zone/state condition.

### Response

```json
{
  "ZonesInfo": null,
  "Success": false,
  "ErrorCode": -1233,
  "ErrorString": "ERROR: NOT READY",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

### Observed Result

The API rejected the arm operation.

- `Success:     false`
- `ErrorCode:   -1233`
- `ErrorString: ERROR: NOT READY`

### Interpretation

The command reached the alarm-control workflow, but the partition condition prevented arming.

The rejection was consistent with the previously observed **NOT READY** partition state.

**Result:** EXPECTED REJECTION / VALIDATED

---

## 7. Scenario B — Panel Ready

After the alarm panel was placed in a ready condition, the same Arm Away operation was executed again.

### Response Result

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

The API accepted the remote arm command successfully.

- `Success:     true`
- `ErrorCode:   0`
- `ErrorString: OK`

The physical alarm panel began its arming sequence.

**Result:** PASS / VALIDATED

---

## 8. Returned Partition State

The successful response returned the current partition information inside `ExternalDevices`.

Observed state:

```text
Name:            Area 1
PartitionNumber: 1
DeviceState:     1
DeviceStateExt:  3
```

According to the API specification:

- `DeviceStateExt 3 = EXIT DELAY`

This is consistent with the panel entering its exit-delay period immediately after the Arm Away command was accepted.

---

## 9. Returned Arm/Disarm Commands

The partition continued to expose:

```text
CheckStateCommand: #RCARMED
OnStayCommand:     #RCSTAYARM
OnCommand:         #RCARM
OffCommand:        #RCDISARM
OnPINRequired:     true
OffPINRequired:    true
```

This confirms that the returned object represents the remotely controllable alarm partition.

---

## 10. Zone State After Successful Arm Command

Immediately after successful command execution, the response reported:

- Zones 001–008 → `ZoneState 5`
- Zones 009–016 → `ZoneState 1`

According to the API specification:

- `ZoneState 1 = CLOSED`
- `ZoneState 5 = BYPASSED`

Therefore, the response observed:

- Zones 001–008 → **BYPASSED**
- Zones 009–016 → **CLOSED**

This behavior is recorded exactly as returned by the API.

The reason the first eight zones were reported as bypassed is not inferred in this test.

**Validation Classification:**

- Returned zone state → **OBSERVED**
- Reason for bypass   → **NOT DETERMINED**

---

## 11. Response State Synchronization

The successful `RemoteArm` response included `ExternalDevices`.

This allowed the API response itself to expose the updated partition state immediately after the command.

A separate `GetDeviceStatusData` call was not required to observe the initial exit-delay state.

This behavior is consistent with the API documentation.

---

## 12. Error Handling Validated

This test validated two distinct outcomes from the same command.

| Condition | Success | ErrorCode | Meaning |
|---|---|---|---|
| Panel not ready | false | -1233 | ERROR: NOT READY |
| Panel ready | true | 0 | Command accepted |

This demonstrates that HTTP 200 alone does not indicate successful execution.

The result must be evaluated from:

- `Success`
- `ErrorCode`
- `ErrorString`

---

## 13. Validation Result

**PASS**

Validated:

- authenticated `RemoteArm` operation;
- Arm Away using `ArmingState: 1`;
- partition-specific operation;
- use of a panel PIN;
- rejection when the panel is not ready;
- `-1233 ERROR: NOT READY`;
- successful arm when the panel is ready;
- physical arming sequence initiated;
- exit-delay state returned after success;
- updated `ExternalDevices` returned with the command response.

---

## 14. Security Notes

The following values must be redacted from external documentation:

- `AccessToken`
- `IMEI`
- `UserPIN`
- `UserNumber`
- Internal IDs
- `ControllerID`

The PIN used to arm the physical alarm panel must never be included in manufacturer-facing evidence.

---

## 15. Open Observation

After successful Arm Away execution, several zones were returned as:

- `ZoneState: 5`

which the API specification defines as bypassed.

If no manual or prior bypass action was performed before this test, clarification may be required regarding why these zones transitioned to or were reported as bypassed during a normal `ArmingState: 1` operation.

---

## 16. Next Subtest

**TEST-007B — RemoteArm / Disarm**

The next test should use:

- `ArmingState: 0`