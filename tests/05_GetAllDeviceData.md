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

Validate retrieval of the current device and alarm-panel configuration associated with an authenticated end-user account.

The test is intended to identify:

- communicator information;
- connected alarm-panel type;
- connection status;
- partitions;
- zones and zone states;
- available outputs;
- remote-control capabilities;
- panel trouble information;
- emergency-key availability.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/GetAllDeviceData
```

---

## 3. Authentication

```http
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
Content-Type: application/json
```

---

## 4. Documented Request

According to the API specification, the device can be identified using either `IMEI` or `SerialNumber`.

If both are supplied, `IMEI` takes precedence.

For this test:

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ProtocolNumber": 4
}
```

---

## 5. Documented Behavior

According to the API specification, `GetAllDeviceData` returns the current state of the device, including:

- partitions;
- zones;
- outputs;
- arming state.

Entries returned in `ExternalDevices` contain a `Regime` field that identifies the type of object.

Partition entries may also include:

- `DeviceState`;
- `DeviceStateExt`;
- `ZonesInfo`.

---

## 6. Observed Response Structure

The successful response contained the following major sections:

- `ClientDeviceDataV2Response`
- `AlarmControlSettingsV2Response`
- `AlarmZoneUserGroupPartitionV2Response`
- `CamerasDataV2Response`
- `NFC_Tags`
- `Success`
- `ErrorCode`
- `ErrorString`

The complete sanitized response is stored separately as test evidence.

---

## 7. Communicator and Panel Detection

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

This validates that the API can identify both the communicator and the connected DSC panel family/model.

---

## 8. Panel Capabilities

The response reported:

- `EnableZonesState:      true`
- `EnableBypass:          true`
- `Connected:             true`
- `ArmDisarmDisabled:     false`
- `HasPanelEmergencyKeys: true`

---

## 9. Emergency Keys

The following emergency keys were returned:

- Fire
- Medical
- Panic

No emergency action was executed during this test.

- Discovery: **VALIDATED**
- Functional execution: **NOT TESTED**

---

## 10. Partition Discovery

One alarm partition was discovered:

```text
Name:            Area 1
Regime:          3
PartitionNumber: 1
Enabled:         true
```

The following command definitions were returned:

- `CheckStateCommand: #RCARMED`
- `OnStayCommand:     #RCSTAYARM`
- `OnCommand:         #RCARM`
- `OffCommand:        #RCDISARM`

The partition also reported:

- `OnPINRequired:  true`
- `OffPINRequired: true`

This demonstrates that remote arm/disarm command definitions exist for the discovered partition.

Actual arm/disarm execution remains pending separate validation.

---

## 11. Current Partition State

The partition returned:

- `DeviceState:    2`
- `DeviceStateExt: 7`

According to the API specification:

- `DeviceStateExt 7 = not ready to arm`

Therefore, at the moment of this request the partition reported a **NOT_READY** condition.

---

## 12. Zone Discovery

Sixteen zones were returned for Partition 1.

Observed state summary:

- Zones 001–008 → `ZoneState 2`
- Zones 009–016 → `ZoneState 1`

According to the API specification:

- `ZoneState 1 = closed`
- `ZoneState 2 = open`

Therefore, during this test:

- Zones 001–008 → **OPEN**
- Zones 009–016 → **CLOSED**

This state is consistent with the partition reporting `DeviceStateExt: 7` (NOT_READY).

---

## 13. Trouble Information

The partition returned the following troubles:

- `ServiceRequired`
- `MissingBattery`
- `BellCircuit`
- `LossofTimeorDate`

These values are recorded exactly as returned by the API.

No attempt is made in this test to diagnose or resolve these trouble conditions.

---

## 14. Output Discovery

Four external outputs were returned:

| Name | DevicePIN | Regime | Enabled | OnPINRequired |
|---|---|---|---|---|
| COMM. OUTPUT 1 | 1 | 5 | true | false |
| COMM. OUTPUT 2 | 2 | 5 | true | false |
| COMM. OUTPUT 3 | 3 | 5 | true | false |
| COMM. OUTPUT 4 | 4 | 5 | true | false |

The returned command templates followed the pattern:

```text
#setextgpio,<OUTPUT>,0,{DELAY},{USERID},{PIN},{PARTITION}
```

Actual output activation remains pending functional validation.

---

## 15. Documentation Clarification Required

The API specification describes:

- `Regime 3 = alarm partition`
- `Regime 5 = momentary panel PGM`

However, the tested device returned four entries named:

- COMM. OUTPUT 1
- COMM. OUTPUT 2
- COMM. OUTPUT 3
- COMM. OUTPUT 4

with:

- `Regime: 5`

and `#setextgpio` command templates.

This behavior is recorded as observed and should not be reclassified without confirmation from the API provider.

### Open Question

What is the intended distinction between:

- communicator outputs;
- panel PGMs;

when an entry named `COMM. OUTPUT` is returned with `Regime: 5`?

---

## 16. Validation Result

**PASS**

Validated:

- authenticated device-data retrieval;
- communicator discovery;
- communicator model identification;
- connected DSC panel identification;
- connection status;
- partition discovery;
- zone discovery;
- current zone-state retrieval;
- current partition-state retrieval;
- trouble retrieval;
- output discovery;
- emergency-key discovery.

Not yet functionally validated:

- Remote arm
- Remote disarm
- Zone bypass
- Output activation
- Panel PGM activation
- Emergency commands

---

## 17. Security Notes

The following values should be redacted from manufacturer-facing test evidence unless specifically required for technical support:

- `AccessToken`
- `IMEI`
- `SerialNumber`
- `ControllerClientID`
- `ClientName`
- `CardNo`
- `BackupCardNo`
- `SiteNo`
- `ConnectedProxyUrls`
- `ConnectedEndpointUrls`
- Internal IDs
- User IDs / UUIDs
- `AlarmUserNumber`

Hardware/software identification values such as:

- `HSCode`
- `MIDLetVersion`
- `KeybusMode`
- `KeybusSubMode`

may remain visible because they are relevant to reproducibility.

---

## 18. Evidence

Complete sanitized response:

```text
evidence/05_GetAllDeviceData_response_sanitized.json
```