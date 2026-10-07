# M2M Mobile Integration API v3 — Validation Feedback

## Purpose

This document summarizes the main findings from a controlled validation of the M2M Mobile Integration API v3 against a real installation.

The goal is not to report general API failures. Most tested operations worked successfully. The purpose of this feedback is to clarify a small number of functional inconsistencies and implementation details that would be important for a production integration.

## Test Environment

```text
Communicator: LE4050M-LA
Panel: DSC PowerSeries NEO
Panel model: HS2064
API: M2M Mobile Integration API v3
Environment: Production
Primary protocol value used: ProtocolNumber 4
```

## Successfully Validated

The following operations were successfully reproduced against the physical installation:

- CreateAuthorizationCode
- CreateAccessToken
- RegenerateAccessToken
- GetHAUserSettings
- GetAllDeviceData
- GetDeviceStatusData
- RemoteArm — Arm Away
- RemoteArm — Stay Arm
- RemoteArm — Disarm
- RemoteArm — Output ON
- RemoteArm — Output OFF
- Panel emergency key — Fire
- Panel emergency key — Medical
- Panel emergency key — Panic
- SetExternalDeviceName — output label
- SetExternalDeviceName — zone label
- SetExternalOutputsVisibility — hide outputs
- SetExternalOutputsVisibility — restore outputs

The two functional areas that could not be reproduced as documented were:

1. Automatic bypass using `RemoteArm` with `ArmingState: 6`.
2. Direct zone bypass using `RemoteBypassExt`.

---

## 1. RemoteBypassExt Returns Error -1002

### Documented Request

The endpoint documentation indicates that a zone can be bypassed using a request containing the target device, partition, zone list and user PIN.

A test matching the documented request structure was performed.

### Observed Result

The endpoint was reachable and returned HTTP 200, but the operation consistently failed with:

```text
Success: false
ErrorCode: -1002
ErrorString: ERROR: INCOMINMG DATA
```

The following variations were also tested:

- with `ProtocolNumber: 4`;
- without `ProtocolNumber`;
- with an explicit `AlarmUserId`;
- after regenerating the AccessToken.

The result remained unchanged.

### Clarification Requested

Could you please provide:

- a known-working `RemoteBypassExt` request for an LE4050M-LA connected to a DSC PowerSeries NEO HS2064;
- the expected meaning and source of `AlarmUserId`;
- the intended meaning of error `-1002` for this endpoint;
- confirmation that `RemoteBypassExt` is supported for this communicator/panel combination.

The spelling `INCOMINMG DATA` above is preserved exactly as returned by the production API.

Related validation record:

```text
tests/08_RemoteBypassExt_Bypass Zone.md
```

---

## 2. RemoteArm ArmingState 6 — Automatic Bypass Not Reproduced

### Documented Behavior

`ArmingState: 6` is described as an Away arm operation that should automatically bypass open zones.

### Observed Result

With multiple open zones, the API returned:

```text
Success: false
ErrorCode: -2510
ErrorString: ERROR: FORCE ARM FAILED, ALARM IS NOT READY
```

The open zones were then configured as DSC zone definition:

```text
03 — Instant
```

Those zones were confirmed to be manually bypassable.

The same `ArmingState: 6` request was repeated and returned the same `-2510` result.

As a control test, the same zones were manually bypassed first. The panel then armed successfully in Away mode and reported:

```text
System Armed in Away Mode
```

### Clarification Requested

Does `ArmingState: 6` require any additional:

- communicator configuration;
- panel configuration;
- account permission;
- alarm-user privilege;
- PIN privilege;
- zone configuration?

Please also clarify the conditions that are expected to produce error `-2510`.

Related validation record:

```text
tests/07_RemoteArm_Arm_With_Bypass.md
```

---

## 3. Output Metadata Reports OnPINRequired false, but Runtime Requires PIN

### Discovery Result

The discovered output returned:

```text
Name: COMM. OUTPUT 1
DevicePIN: 1
Enabled: true
OnPINRequired: false
```

### Runtime Result

An output activation request without `UserPIN` returned:

```text
Success: false
ErrorCode: -1234
ErrorString: ERROR: PIN REQUIRED
```

The same request succeeded after `UserPIN` was included.

Both output ON and output OFF were physically validated.

### Clarification Requested

Should clients treat `OnPINRequired` as authoritative?

If so, should this value have been `true` for the tested output?

If not, what field or rule should clients use to determine whether `UserPIN` must be supplied?

Related validation record:

```text
tests/07_RemoteArm_Output ON-OFF.md
```

---

## 4. Distinguishing COMM. OUTPUT from Panel PGM

Device discovery returned four controllable entries:

```text
COMM. OUTPUT 1
COMM. OUTPUT 2
COMM. OUTPUT 3
COMM. OUTPUT 4
```

They were returned with:

```text
Regime: 5
DevicePIN: 1..4
OnCommand: #setextgpio,...
```

The same outputs were exposed as controllable output/PGM functions in the official M2M application, and physical ON/OFF control was validated using `RemoteArm` with `ArmingState: 3` and `ArmingState: 4`.

The API documentation also distinguishes communicator outputs from panel PGMs in the `RemoteArm` arming-state definitions.

### Clarification Requested

What is the canonical way for a client to distinguish:

- communicator outputs;
- panel PGMs?

Should the client rely on:

- `Regime`;
- `DevicePIN`;
- command type such as `#setextgpio`;
- another field?

Please also confirm whether `Regime: 5` is expected for entries named `COMM. OUTPUT` on this device combination.

---

## 5. Recommended Use of GetDeviceStatusData

`GetDeviceStatusData` worked successfully as a state-refresh endpoint.

The tested response included:

- partition state;
- zone states;
- trouble state;
- outputs;
- emergency keys;
- signal level;
- arm/disarm command fields such as `#RCARM`, `#RCSTAYARM` and `#RCDISARM`.

### Clarification Requested

Is the recommended application flow:

```text
GetAllDeviceData
        ↓
initial discovery / topology
        ↓
GetDeviceStatusData
        ↓
periodic state refresh
```

If so, please provide:

- recommended polling interval;
- applicable rate limits;
- whether limits are applied per user, device, token or IP;
- whether an event-driven, push or webhook mechanism exists for device-state changes.

Related validation record:

```text
tests/06_GetDeviceStatusData.md
```

---

## 6. Dealer/Admin vs End-User Integration Scope

During preliminary testing, an administrative/dealer account authenticated successfully using:

```text
AdminRequest: true
```

However, the client/device workflow used during this validation did not expose the communicator through that session in the same way as the associated end-user account.

The formal endpoint validation was therefore completed using the end-user account associated with the communicator.

### Clarification Requested

Is Mobile Integration API v3 primarily designed around end-user sessions?

For dealer/admin integrations, is there a separate endpoint, workflow or API for:

- enumerating customers;
- enumerating communicators;
- mapping communicators to end-user accounts;
- operating devices across multiple client accounts?

---

## 7. RefreshToken Lifecycle

`RegenerateAccessToken` successfully returned:

- a new AccessToken;
- a new RefreshToken;
- a new expiration value.

The validation did not establish the lifecycle rules for refresh tokens.

### Clarification Requested

Please confirm:

- RefreshToken lifetime;
- whether RefreshToken values are single-use;
- whether the previous RefreshToken is invalidated immediately after regeneration;
- whether multiple AccessTokens may coexist;
- whether multiple authenticated application sessions are supported for the same end user.

---

## 8. ProtocolNumber and Compatibility Guidance

Most validation was completed successfully using:

```text
ProtocolNumber: 4
```

### Clarification Requested

For new LE4050M-LA integrations:

- is `ProtocolNumber: 4` the recommended production value;
- should a later protocol version be used instead;
- what compatibility differences should be expected between protocol versions?

For implementation planning, it would also be useful to confirm the supported compatibility matrix for:

- LE4050M-LA firmware versions;
- DSC PowerSeries NEO panels;
- HS2064 firmware versions.

---

## Closing Summary

The majority of the tested API behaved successfully against the physical installation.

The main functional items requiring clarification are:

1. `RemoteBypassExt` consistently returning `-1002`.
2. `RemoteArm` with `ArmingState: 6` not automatically bypassing open zones.
3. Output discovery reporting `OnPINRequired: false` while the runtime command requires `UserPIN`.

The remaining questions are primarily implementation-contract and production-integration details intended to ensure the API is consumed according to the expected architecture.
