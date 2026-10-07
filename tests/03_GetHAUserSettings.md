# TEST-003 — GetHAUserSettings

## Test Information

**Test ID:** TEST-003  
**Endpoint:** `GetHAUserSettings`  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate authenticated retrieval of the signed-in end-user profile, associated device information, alarm-panel user association and controller permissions.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/GetHAUserSettings
```

## 3. Authentication

```http
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
Content-Type: application/json
```

---

## 4. Request

The API specification defines an empty request body.

The validation request used:

```json
{}
```

The API accepted `{}` successfully.

---

## 5. Documented Behavior

The endpoint returns the signed-in user's profile and devices associated with the account.

`AlarmIMEIUserNumbers[].Number` is documented as the alarm-panel user number associated with the device and is the value used by `RemoteArm` as `UserNumber`.

---

## 6. Observed Result

The API returned HTTP 200 and:

```text
Success: true
ErrorCode: 0
ErrorString: OK
Administrator: false
AdminRequest: false
IsApproved: true
IsLockedOut: false
ReadOnly: false
```

At least one communicator was associated with the authenticated account.

The response returned both:

- `AlarmIMEIUserNumbers`
- `ControllerPermitions`

`ControllerPermitions` is preserved with the spelling returned by the API.

---

## 7. Device and Alarm-User Association

The same alarm-panel user number was returned in:

- `AlarmIMEIUserNumbers[].Number`
- `ControllerPermitions[].AlarmUserNumber`

This validated the association between the authenticated end user, the communicator and the corresponding alarm-panel user number.

Installation identifiers were redacted from the public test record.

---

## 8. Observed Controller Permissions

| Permission | Observed Value |
|---|---|
| BypassEnable | true |
| DiagnosticEnable | false |
| EditEventsLabel | true |
| IFTTTEnable | true |
| MainUser | true |
| RemoteArmDisarm | true |
| EnabledRemoteArmDisarm | true |
| RestrictedAccess | false |
| OutputEnable | true |
| ViewExternalOutputs | true |
| ViewEmergencyKeys | true |
| RemoteOpenDoor | false |
| ManageRules | true |
| CameraAccess | true |

Additional observed values:

```text
RemotePINUsage: 0
PINLength: null
EncryptionKey: null
UserPIN: null
```

These values are recorded as returned by the API. Functional behavior of the corresponding capabilities was tested separately in later test cases.

---

## 9. Validation Result

**PASS / VALIDATED**

Validated:

- protected endpoint authentication;
- use of the `M2MOAuth2Token` header;
- retrieval of the end-user profile;
- retrieval of associated communicator data;
- retrieval of alarm-panel user association;
- retrieval of controller permission values;
- acceptance of `{}` as the request body.

---

## 10. Security Notes

The following values are redacted from public evidence:

- `AccessToken`
- IMEI
- client and user identifiers
- username
- alarm user number
- authentication secrets and panel PINs
