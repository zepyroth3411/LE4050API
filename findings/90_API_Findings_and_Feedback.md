# M2M Mobile Integration API v3 — Findings and Feedback

## Document Purpose

This document consolidates cross-test observations that may require clarification from the API provider. Individual files under `tests/` are intentionally limited to factual endpoint behavior and test results.

Test environment:

```text
Communicator: LE4050M-LA
Panel: DSC PowerSeries NEO HS2064
API: M2M Mobile Integration API v3
Environment: Production
Primary protocol value used: ProtocolNumber 4
```

---

## 1. RemoteArm — Automatic Bypass (`ArmingState: 6`)

### Documented

`ArmingState: 6` is described as Arm Away while bypassing zones left open.

### Observed

With multiple open zones, the API consistently returned:

```text
Success: false
ErrorCode: -2510
ErrorString: ERROR: FORCE ARM FAILED, ALARM IS NOT READY
```

The same zones were configured as DSC zone definition `03 — Instant` and were manually bypassable. The result remained unchanged.

When the same zones were manually bypassed first, Away arming succeeded and the panel reported:

```text
System Armed in Away Mode
```

### Clarification Requested

- Are additional communicator, account or panel settings required for automatic bypass?
- Are there zone-type restrictions for this operation?
- Does the operation require a specific alarm user number, PIN privilege or user permission?
- What conditions are intended to produce error `-2510`?

---

## 2. RemoteBypassExt — Documented Request Returns `-1002`

### Documented

The endpoint accepts `Partition`, `BypassZonesId` / `UnbypassZonesId` and `UserPIN`, with optional device/user fields.

### Observed

The documented request structure was reproduced against an existing device, partition and zone.

Tested variations included:

- documented request structure;
- with and without `ProtocolNumber: 4`;
- explicit `AlarmUserId`;
- regenerated AccessToken.

Every variation returned:

```text
Success: false
ErrorCode: -1002
ErrorString: ERROR: INCOMINMG DATA
```

The spelling `INCOMINMG` is preserved exactly as returned by the API.

### Clarification Requested

- What request field or account condition is required for `RemoteBypassExt`?
- Is `AlarmUserId` expected to be a different value from the alarm-panel user number returned by `GetHAUserSettings`?
- Is the endpoint supported for LE4050M-LA + NEO HS2064 installations?
- What is the intended definition of error `-1002` for this endpoint?

---

## 3. Output PIN Metadata vs Runtime Requirement

### Observed Discovery Metadata

`COMM. OUTPUT 1` returned:

```text
OnPINRequired: false
```

### Observed Runtime Behavior

The output activation request without `UserPIN` returned:

```text
ErrorCode: -1234
ErrorString: ERROR: PIN REQUIRED
```

The same request succeeded after `UserPIN` was supplied. Output ON and OFF were physically validated.

### Clarification Requested

- Should `OnPINRequired` have been `true` for this installation?
- Should clients always retry with `UserPIN` after `-1234`, regardless of discovery metadata?

---

## 4. `COMM. OUTPUT` Naming and `Regime: 5`

Discovery returned four entries:

```text
COMM. OUTPUT 1
COMM. OUTPUT 2
COMM. OUTPUT 3
COMM. OUTPUT 4
```

All were returned with `Regime: 5` and `#setextgpio` command templates.

The official M2M application exposed the same entries as controllable output/PGM functions, and TEST-007E confirmed physical ON/OFF operation.

The API documentation separately distinguishes communicator pins from panel PGMs in the `RemoteArm` `ArmingState` behavior.

### Clarification Requested

- What is the intended semantic distinction between a communicator output and a panel PGM in discovery results?
- Is `Regime: 5` expected for entries named `COMM. OUTPUT` on this communicator/panel combination?

---

## 5. GetDeviceStatusData Response Scope

The endpoint is described as a lighter state-refresh operation and as returning the device output list without the arming settings.

The tested response included:

- partition state;
- zone states;
- troubles;
- emergency keys;
- outputs;
- signal level;
- arm/disarm command fields such as `#RCARM`, `#RCSTAYARM` and `#RCDISARM`.

### Clarification Requested

- What specifically is omitted by the phrase "without the arming settings"?
- Is `GetDeviceStatusData` the recommended endpoint for continuous state polling?

---

## 6. Polling Frequency and Rate Limits

The API reference does not provide a tested production polling interval or explicit rate-limit guidance.

### Clarification Requested

- Recommended polling interval for `GetDeviceStatusData`.
- Per-user, per-device or per-IP rate limits.
- Whether a push/webhook mechanism exists for device-state changes.

---

## 7. SignalLevel Scale

`GetDeviceStatusData` with `LoadSignalLevel: true` returned:

```text
SignalLevel: "20"
```

### Clarification Requested

- Unit and scale used by `SignalLevel`.
- Expected minimum/maximum values and recommended interpretation for UI display.

---

## 8. Remote Command Latency

One validated disarm operation took approximately:

```text
8.44 seconds
```

from request submission to API response.

This is a single observed round-trip value and is not presented as a benchmark.

### Clarification Requested

- Typical and maximum expected latency for remote arm/disarm/output commands.
- Whether the HTTP response waits for confirmation from the communicator/panel before returning success.

---

## 9. Refresh-Token Lifecycle

`RegenerateAccessToken` successfully returned a new AccessToken, RefreshToken and expiration value.

The validation did not establish:

- refresh-token lifetime;
- whether refresh tokens are single-use;
- whether the previous refresh token is invalidated immediately;
- whether multiple access tokens may coexist.

### Clarification Requested

Please provide the intended token rotation and coexistence rules.

---

## 10. End-User vs Dealer/Admin Session Scope

During preliminary validation an administrative/dealer authentication succeeded with `AdminRequest: true`, but the tested client/device workflow did not expose the communicator through that session. Formal endpoint validation therefore used the communicator-associated end-user account.

### Clarification Requested

- Is Mobile Integration API v3 intended primarily for end-user sessions?
- Is there a separate dealer/admin endpoint or API for enumerating client communicators?

---

## 11. ProtocolNumber Guidance

Most validation used:

```text
ProtocolNumber: 4
```

The specification also describes different state-reporting behavior for later protocol versions.

### Clarification Requested

- Is ProtocolNumber 4 still the recommended production value for LE4050M-LA integrations?
- If version 5 or later is recommended, what compatibility considerations apply?

---

## 12. SetExternalOutputsVisibility Response Shape

The endpoint is documented with a base success response.

In the validated call, the response also returned updated configuration data in `ResponseBody`, including the new `ViewExternalOutputs` value.

The official M2M application immediately reflected the change: `false` hid the PGM/output section and `true` restored it.

### Clarification Requested

- Is the configuration object in `ResponseBody` part of the stable response contract?

---

## 13. Compatibility Information Requested

For implementation planning, clarification would also be useful on:

- supported LE4050 firmware ranges;
- supported DSC NEO panel/firmware combinations;
- whether capability/permission flags returned by discovery should be treated as authoritative for UI gating.

---

## Summary

The majority of tested API operations were successfully reproduced against the physical installation. The two unresolved functional areas are:

1. automatic bypass during `RemoteArm` with `ArmingState: 6`;
2. direct zone bypass through `RemoteBypassExt`.

The remaining items above are documentation, response-contract or operational clarifications that would help make a production integration deterministic.
