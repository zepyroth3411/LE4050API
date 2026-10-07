# TEST-007D — RemoteArm / Disarm

## Test Information

**Test ID:** TEST-007D  
**Endpoint:** `RemoteArm`  
**Operation:** Disarm  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate remote disarming of a specific alarm partition.

---

## 2. Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 0,
  "PartitionNumber": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "ProtocolNumber": 4
}
```

---

## 3. Documented Behavior

According to the API specification:

- `ArmingState: 0` = Disarm

The operation was directed to:

- `PartitionNumber: "1"`

---

## 4. Observed Result

The API returned HTTP 200.

- `Success:     true`
- `ErrorCode:   0`
- `ErrorString: OK`

The physical alarm panel was successfully disarmed.

Returned partition state:

- `DeviceState:    2`
- `DeviceStateExt: 7`

The partition was therefore disarmed, while the panel remained **NOT READY** because zones were open.

---

## 5. Response Time Observation

The client observed an approximate API response time of:

```text
8.44 seconds
```

This value represents the complete request/response round trip.

The test does not determine which portion of the delay belongs to:

- M2M cloud processing;
- Internet transport;
- cellular communication with the communicator;
- communicator processing;
- alarm-panel communication;
- post-command state confirmation.

Additional repeated measurements would be required to characterize command latency accurately.

---

## 6. Validation Result

**PASS**

Validated:

- `ArmingState: 0`;
- partition-specific remote disarm;
- panel PIN usage;
- successful physical disarm;
- updated partition state returned after execution.

---

## 7. Security Notes

The following values were redacted:

- `AccessToken`
- `IMEI`
- `UserPIN`