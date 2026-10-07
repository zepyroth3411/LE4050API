# TEST-005 — GetAllDeviceData

## Test Information

**Test ID:** TEST-005  
**Endpoint:** `GetAllDeviceData`  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate retrieval of the current communicator, panel, partition, zone, output, trouble and emergency-key information associated with the authenticated end-user account.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/GetAllDeviceData
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
  "ProtocolNumber": 4
}
```

The API specification also permits device identification by `SerialNumber`.

---

## 5. Observed Response

The response returned HTTP 200 with:

```text
Success: true
ErrorCode: 0
ErrorString: OK
```

The complete sanitized response is stored in:

```text
evidence/05_GetAllDeviceData_response_sanitized.json
```

---

## 6. Communicator and Panel Identification

The API identified the tested installation as:

```text
Communicator model: LE4050M-LA
Communicator status: Connected
Keybus mode: NEO
Keybus submode: HS2064
ControlExternalDevices: true
AllowRemotePermissions: true
RemoteAccessDynamicControls: true
```

---

## 7. Panel Capabilities

Observed values included:

```text
EnableZonesState: true
EnableBypass: true
Connected: true
ArmDisarmDisabled: false
HasPanelEmergencyKeys: true
```

The emergency-key list contained:

- Fire
- Medical
- Panic

---

## 8. Partition Discovery

One alarm partition was returned:

```text
Name: Area 1
Regime: 3
PartitionNumber: 1
Enabled: true
OnPINRequired: true
OffPINRequired: true
```

Command definitions returned for the partition:

```text
CheckStateCommand: #RCARMED
OnStayCommand: #RCSTAYARM
OnCommand: #RCARM
OffCommand: #RCDISARM
```

At the time of this request:

```text
DeviceState: 2
DeviceStateExt: 7
```

`DeviceStateExt: 7` is documented as **not ready to arm**.

---

## 9. Zone Discovery

Sixteen zones were returned.

| Zones | ZoneState | Interpreted State |
|---|---:|---|
| 001–008 | 2 | OPEN |
| 009–016 | 1 | CLOSED |

This was consistent with the partition reporting a not-ready condition.

---

## 10. Trouble Information

The partition returned:

- `ServiceRequired`
- `MissingBattery`
- `BellCircuit`
- `LossofTimeorDate`

The values are preserved exactly as returned by the API.

---

## 11. Output Discovery

Four controllable entries were returned:

| Name | DevicePIN | Regime | Enabled | OnPINRequired |
|---|---:|---:|---|---|
| COMM. OUTPUT 1 | 1 | 5 | true | false |
| COMM. OUTPUT 2 | 2 | 5 | true | false |
| COMM. OUTPUT 3 | 3 | 5 | true | false |
| COMM. OUTPUT 4 | 4 | 5 | true | false |

The returned command template followed:

```text
#setextgpio,<OUTPUT>,0,{DELAY},{USERID},{PIN},{PARTITION}
```

Functional output control was validated separately in TEST-007E.

---

## 12. Validation Result

**PASS / VALIDATED**

Validated in this test:

- communicator discovery and model identification;
- connected panel family/model identification;
- connection status;
- partition discovery and current partition state;
- zone discovery and current zone states;
- trouble retrieval;
- output discovery;
- emergency-key discovery.

Functional actions discovered here were validated separately by later endpoint tests.

---

## 13. Security Notes

Public evidence redacts credentials, IMEI, communicator serial number, SIM identifiers, client/user/controller identifiers and other installation-specific IDs while preserving API field names and operational state values.
