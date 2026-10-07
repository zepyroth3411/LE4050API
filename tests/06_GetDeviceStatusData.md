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

Validate the device-status endpoint intended for state refresh after the initial device information has been loaded.

This test evaluates whether the endpoint can be used to periodically retrieve:

- current partition state;
- current zone states;
- current output information;
- panel trouble information;
- emergency-key availability;
- device signal level.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/GetDeviceStatusData
```

---

## 3. Authentication

```http
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
Content-Type: application/json
```

---

## 4. Documented Behavior

According to the API specification, `GetDeviceStatusData` is intended as a lighter state-refresh operation once the application has already loaded the device.

The request may identify the device using:

- `IMEI`
- or `SerialNumber`

If both are supplied, `IMEI` takes precedence.

The request model also exposes:

- `UserID`
- `ProtocolNumber`
- `LoadSignalLevel`

The response model exposes:

- `Success`
- `ErrorCode`
- `ErrorString`
- `ExternalDevices`
- `SignalLevel`

---

## 5. Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ProtocolNumber": 4,
  "LoadSignalLevel": true
}
```

The following fields were not supplied:

- `SerialNumber`
- `UserID`

The request was accepted successfully without them.

---

## 6. Sanitized Response Structure

The successful response followed this general structure:

```json
{
  "ExternalDevices": [
    {
      "Name": "EmergencyKeys",
      "...": "..."
    },
    {
      "Name": "Area 1",
      "DeviceState": 2,
      "DeviceStateExt": 7,
      "ZonesInfo": [
        "..."
      ]
    },
    {
      "Name": "COMM. OUTPUT 1",
      "...": "..."
    },
    {
      "Name": "COMM. OUTPUT 2",
      "...": "..."
    },
    {
      "Name": "COMM. OUTPUT 3",
      "...": "..."
    },
    {
      "Name": "COMM. OUTPUT 4",
      "...": "..."
    }
  ],
  "SignalLevel": "20",
  "Success": true,
  "ErrorCode": 0,
  "ErrorString": "OK"
}
```

The complete sanitized response should be stored separately as test evidence.

---

## 7. Observed Result

The API returned HTTP 200.

The response body reported:

- `Success:     true`
- `ErrorCode:   0`
- `ErrorString: OK`
- `SignalLevel: 20`

`ExternalDevices` was populated successfully.

---

## 8. Signal-Level Validation

The request included:

- `LoadSignalLevel: true`

The response returned:

- `SignalLevel: "20"`

Therefore, retrieval of signal level through this request option is:

**VALIDATED**

The exact unit or interpretation of the value `20` is not inferred from this test and should be confirmed by the API provider if required.

---

## 9. Partition State

The response returned:

```text
Name:            Area 1
Regime:          3
PartitionNumber: 1
DeviceState:     2
DeviceStateExt:  7
Enabled:         true
```

According to the API specification:

- `DeviceStateExt 7 = NOT READY TO ARM`

The partition therefore continued to report a not-ready condition at the moment of this request.

---

## 10. Zone State Refresh

Sixteen zones were returned.

Observed state summary:

- Zones 001–008 → `ZoneState 2`
- Zones 009–016 → `ZoneState 1`

According to the API specification:

- `ZoneState 1 = CLOSED`
- `ZoneState 2 = OPEN`

Therefore, the current state observed during this request was:

- Zones 001–008 → **OPEN**
- Zones 009–016 → **CLOSED**

This matches the state previously observed using `GetAllDeviceData`.

---

## 11. Trouble State

The partition returned:

- `ServiceRequired`
- `MissingBattery`
- `BellCircuit`
- `LossofTimeorDate`

These trouble values were again available through the status-refresh endpoint.

This confirms that the endpoint can expose current panel trouble information as part of the polling response.

---

## 12. Output State Retrieval

The following outputs were returned again:

| Name | DevicePIN | Regime | Enabled |
|---|---|---|---|
| COMM. OUTPUT 1 | 1 | 5 | true |
| COMM. OUTPUT 2 | 2 | 5 | true |
| COMM. OUTPUT 3 | 3 | 5 | true |
| COMM. OUTPUT 4 | 4 | 5 | true |

The output entries retained their associated command definitions.

Example:

```text
COMM. OUTPUT 1
DevicePIN: 1
OnCommand:
#setextgpio,1,0,{DELAY},{USERID},{PIN},{PARTITION}
```

Actual output activation remains pending separate functional validation.

---

## 13. Emergency Keys

The endpoint returned:

- Fire
- Medical
- Panic

under the `EmergencyKeys` entry.

Emergency-key discovery therefore remains available through this status-refresh call.

No emergency command was executed.

---

## 14. Comparison with GetAllDeviceData

### GetAllDeviceData

Observed use:

- Initial device discovery
- Communicator information
- Panel identification
- Permissions
- Partitions
- Zones
- Outputs
- Device configuration

### GetDeviceStatusData

Observed use:

- Current partition state
- Current zone states
- Troubles
- Outputs
- Emergency keys
- Signal level

This makes `GetDeviceStatusData` a suitable candidate for periodic state refresh after initial device discovery.

---

## 15. Application Pattern Observed

A possible client workflow based on the tested behavior is:

```text
Application starts
        ↓
GetAllDeviceData
        ↓
Discover device topology and capabilities
        ↓
Build UI
        ↓
GetDeviceStatusData
        ↓
Refresh current state periodically
        ↓
Update partition / zones / troubles / outputs
```

This pattern is consistent with the API description of `GetDeviceStatusData` as a lighter polling operation.

---

## 16. Documentation Observation

The API description refers to this endpoint as:

> the device's output list without the arming settings

However, the actual tested response included considerably more than output state.

Observed data included:

- partition state
- zone states
- arm/disarm command definitions
- troubles
- emergency keys
- outputs
- signal level

For example, the partition still contained:

```text
CheckStateCommand: #RCARMED
OnStayCommand:     #RCSTAYARM
OnCommand:         #RCARM
OffCommand:        #RCDISARM
```

### Open Question

The API provider may wish to clarify what is specifically meant by *"without the arming settings"*, since arm/disarm-related fields remain present inside the returned `ExternalDevices`.

---

## 17. Validation Result

**PASS**

Validated:

- authenticated status retrieval;
- device identification using `IMEI`;
- `ProtocolNumber: 4`;
- operation without explicit `UserID`;
- `LoadSignalLevel: true`;
- signal-level retrieval;
- partition-state retrieval;
- zone-state retrieval;
- trouble-state retrieval;
- output discovery;
- emergency-key discovery;
- consistency with the previous `GetAllDeviceData` state.

---

## 18. UI / Polling Relevance

Based on the observed response, this endpoint can provide the data needed for a compact live alarm-system view.

For example:

```text
┌──────────────────────────────┐
│ AREA 1                       │
│ NOT READY                    │
│                              │
│ Zones                        │
│ 01  OPEN                     │
│ 02  OPEN                     │
│ 03  OPEN                     │
│ ...                          │
│ 09  CLOSED                   │
│                              │
│ Troubles: 4                  │
│ Signal: 20                   │
│                              │
│ Outputs: 4                   │
└──────────────────────────────┘
```

The actual polling interval has not yet been determined.

Rate limits and recommended refresh frequency should be confirmed before implementing continuous production polling.

---

## 19. Security Notes

The following values should be removed from external test evidence:

- `AccessToken`
- `IMEI`
- Internal IDs
- `ControllerID`
- User identifiers

Operational values such as:

- `DeviceState`
- `DeviceStateExt`
- `ZoneState`
- `SignalLevel`
- Troubles
- Output names

may remain visible because they are directly relevant to validating the endpoint behavior.

---

## 20. Evidence

Complete sanitized response:
```text
evidence/06_GetDeviceStatusData_response_sanitized.json
```