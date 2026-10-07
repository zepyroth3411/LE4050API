# TEST-006 — GetDeviceStatusData

## Test Information

**Test ID:** TEST-006  
**Endpoint:** `GetDeviceStatusData`  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate retrieval of current panel state after initial device discovery, including partition state, zone state, troubles, outputs, emergency keys and signal level.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/GetDeviceStatusData
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
  "ProtocolNumber": 4,
  "LoadSignalLevel": true
}
```

`SerialNumber` and `UserID` were omitted and were not required for this successful test.

---

## 5. Observed Result

The API returned HTTP 200 and:

```text
Success: true
ErrorCode: 0
ErrorString: OK
SignalLevel: 20
```

`ExternalDevices` was populated.

The complete sanitized response is stored in:

```text
evidence/06_GetDeviceStatusData_response_sanitized.json
```

---

## 6. Partition and Zone State

The returned partition state was:

```text
Name: Area 1
PartitionNumber: 1
DeviceState: 2
DeviceStateExt: 7
Enabled: true
```

`DeviceStateExt: 7` is documented as **not ready to arm**.

Sixteen zones were returned:

| Zones | ZoneState | State |
|---|---:|---|
| 001–008 | 2 | OPEN |
| 009–016 | 1 | CLOSED |

The values matched the state observed previously through `GetAllDeviceData`.

---

## 7. Trouble State

The endpoint returned the same active trouble list:

- `ServiceRequired`
- `MissingBattery`
- `BellCircuit`
- `LossofTimeorDate`

---

## 8. Output and Emergency-Key State

The response again returned `COMM. OUTPUT 1` through `COMM. OUTPUT 4`, together with their command metadata.

The `EmergencyKeys` entry again exposed:

- Fire
- Medical
- Panic

---

## 9. Signal-Level Retrieval

`LoadSignalLevel: true` produced:

```text
SignalLevel: "20"
```

Retrieval of the signal-level field is therefore validated. Its unit or scale was not established by this test.

---

## 10. Observed Response Scope

The API description characterizes this endpoint as a lighter state-refresh call. In the tested response it returned current partition, zones, troubles, outputs, emergency keys and signal level.

The partition object also continued to contain the arm/disarm command fields:

```text
CheckStateCommand: #RCARMED
OnStayCommand: #RCSTAYARM
OnCommand: #RCARM
OffCommand: #RCDISARM
```

This observation is retained as part of the actual response behavior.

---

## 11. Validation Result

**PASS / VALIDATED**

Validated:

- authenticated status retrieval;
- identification by IMEI;
- `ProtocolNumber: 4`;
- operation without explicit `UserID`;
- signal-level retrieval;
- partition-state retrieval;
- zone-state retrieval;
- trouble-state retrieval;
- output-state retrieval;
- emergency-key retrieval;
- consistency with the preceding discovery response.

---

## 12. Security Notes

Credentials, device identifiers and installation-specific internal identifiers are redacted from public evidence. Operational state fields are preserved.
