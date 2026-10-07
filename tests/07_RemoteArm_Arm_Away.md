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

Validate remote Away arming of a specific partition and verify behavior when the partition is not ready versus ready.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RemoteArm
```

## 3. Authentication

```http
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
Content-Type: application/json
```

## 4. Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 1,
  "PartitionNumber": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "UserNumber": "<REDACTED_ALARM_USER_NUMBER>",
  "ProtocolNumber": 4
}
```

`ArmingState: 1` is documented as Arm Away / arm everything.

---

## 5. Scenario A — Partition Not Ready

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

The command was rejected while the partition was not ready.

**Result:** EXPECTED REJECTION / VALIDATED

---

## 6. Scenario B — Partition Ready

After the panel was placed in a ready condition, the same operation succeeded.

```text
Success: true
ErrorCode: 0
ErrorString: OK
```

The physical panel entered its arming sequence.

Returned partition state:

```text
Name: Area 1
PartitionNumber: 1
DeviceState: 1
DeviceStateExt: 3
```

`DeviceStateExt: 3` is documented as **exit delay**.

**Result:** PASS / VALIDATED

---

## 7. Returned Zone State

Immediately after the successful command, the response reported:

| Zones | ZoneState | State |
|---|---:|---|
| 001–008 | 5 | BYPASSED |
| 009–016 | 1 | CLOSED |

The response is recorded as observed. This test did not establish the reason the first eight zones were reported as bypassed.

---

## 8. Response-State Synchronization

The successful `RemoteArm` response included updated `ExternalDevices`, allowing the initial exit-delay state to be observed without an additional status request.

---

## 9. Validation Result

**PASS / VALIDATED**

Validated:

- `RemoteArm` with `ArmingState: 1`;
- partition-specific Away arm;
- use of a panel PIN;
- rejection with `-1233 ERROR: NOT READY` when the partition was not ready;
- successful physical arming once ready;
- exit-delay state returned after successful execution;
- updated state included in the command response.

---

## 10. Security Notes

Access token, IMEI, panel PIN, alarm user number and installation-specific internal identifiers are redacted.
