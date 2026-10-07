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

Validate that an `AccessToken` generated through `CreateAccessToken` can be used to access a protected API endpoint.

This test also validates retrieval of:

- the authenticated end-user profile;
- devices associated with the user account;
- the alarm-panel user number associated with each device;
- controller permissions available to the authenticated user.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/GetHAUserSettings
```

---

## 3. Authentication

This endpoint requires the access token obtained during TEST-002.

### Header

```http
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

During the validation test, the following header was also sent:

```http
Content-Type: application/json
```

---

## 4. Request

### Documented Request

According to the API specification:

- the endpoint takes no parameters;
- the request body is empty.

### Observed Test Request

During the validation test, an empty JSON object was sent:

```json
{}
```

The API accepted this request successfully.

Therefore:

- Empty request body → **DOCUMENTED**
- `{}` as request body → **VALIDATED**

No device identifier was required in the request.

---

## 5. Documented Behavior

According to the API specification, `GetHAUserSettings` returns the signed-in user's profile and the devices associated with the account.

The device/user association is exposed through:

- `AlarmIMEIUserNumbers`

Each entry may contain:

- `IMEI`
- `Number`
- `VirtualCardID`

The API specification explicitly states that:

- `AlarmIMEIUserNumbers[].Number`

is the alarm-panel user number to be sent with `RemoteArm` for the corresponding device.

The documented response model is:

```text
HAUserSettingsDTORes
├── Success
├── ErrorCode
├── ErrorString
└── HAUserSettings
```

Success or failure is reported through `ErrorCode` in the response body.

---

## 6. Sanitized Response

```json
{
  "SystemSettingsVersion": "1000",
  "HAUserSettings": {
    "Administrator": false,
    "AlarmIMEIUserNumbers": [
      {
        "IMEI": "<REDACTED_IMEI>",
        "Number": "<REDACTED_USER_NUMBER>",
        "VirtualCardID": null
      }
    ],
    "AvailableCultures": [
      {
        "Culture": "AUTO",
        "Description": "Default"
      },
      {
        "Culture": "en-US",
        "Description": "English"
      },
      {
        "Culture": "bg-BG",
        "Description": "Bulgarian"
      },
      {
        "Culture": "it-IT",
        "Description": "Italian"
      },
      {
        "Culture": "pt-PT",
        "Description": "Portuguese"
      },
      {
        "Culture": "lt-LT",
        "Description": "Lithuanian"
      },
      {
        "Culture": "fr-CA",
        "Description": "French (Canada)"
      },
      {
        "Culture": "cs-CZ",
        "Description": "Czech"
      },
      {
        "Culture": "el-GR",
        "Description": "Greek"
      },
      {
        "Culture": "nn-NO",
        "Description": "Norwegian"
      },
      {
        "Culture": "tr-TR",
        "Description": "Turkish"
      },
      {
        "Culture": "es-ES",
        "Description": "Spanish"
      },
      {
        "Culture": "da-DK",
        "Description": "Danish"
      },
      {
        "Culture": "nl-NL",
        "Description": "Dutch"
      },
      {
        "Culture": "sv-SE",
        "Description": "Swedish"
      },
      {
        "Culture": "de-DE",
        "Description": "German"
      },
      {
        "Culture": "fi-FI",
        "Description": "Finnish"
      },
      {
        "Culture": "hr-HR",
        "Description": "Croatian"
      },
      {
        "Culture": "is-IS",
        "Description": "Icelandic"
      }
    ],
    "ClientID": "<REDACTED_CLIENT_ID>",
    "ControllerPermitions": [
      {
        "IMEI": "<REDACTED_IMEI>",
        "AlarmUserNumber": "<REDACTED_USER_NUMBER>",
        "BypassEnable": true,
        "DiagnosticEnable": false,
        "EditEventsLabel": true,
        "IFTTTEnable": true,
        "MainUser": true,
        "RemoteArmDisarm": true,
        "EnabledRemoteArmDisarm": true,
        "EncryptionKey": null,
        "RemotePINUsage": 0,
        "PINLength": null,
        "RestrictedAccess": false,
        "OutputEnable": true,
        "ViewExternalOutputs": true,
        "UserPIN": null,
        "ViewEmergencyKeys": true,
        "RemoteOpenDoor": false,
        "AccessControlUserLevelId": null,
        "ManageRules": true,
        "CameraAccess": true
      }
    ],
    "Culture": "en-US",
    "DiagnosticEnable": false,
    "Email": null,
    "GDPRAgreements": false,
    "HasCreditAcount": false,
    "ID": "<REDACTED_USER_ID>",
    "Name": "",
    "OwnerUserID": "00000000-0000-0000-0000-000000000000",
    "PasswordNew": null,
    "PasswordOld": null,
    "PhoneCulture": "",
    "ReadOnly": false,
    "UserName": "<REDACTED_USERNAME>",
    "UserNameNew": null,
    "IsApproved": true,
    "IsLockedOut": false,
    "InitialPassChanged": 0,
    "AdminRequest": false
  },
  "PowerManage": null,
  "Success": true,
  "ErrorCode": 0,
  "ErrorString": "OK",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

---

## 7. Observed Result

The API returned HTTP 200.

The response body reported:

- `Success:     true`
- `ErrorCode:   0`
- `ErrorString: OK`

The authenticated user context reported:

- `Administrator: false`
- `AdminRequest:  false`
- `IsApproved:    true`
- `IsLockedOut:   false`
- `ReadOnly:      false`

At least one device was associated with the authenticated account.

The response included:

- `AlarmIMEIUserNumbers`
- `ControllerPermitions`

for the associated device.

---

## 8. Device and Alarm User Association

The response established the following relationship:

```text
Authenticated end user
        ↓
Associated device / IMEI
        ↓
Alarm-panel user number
```

The same alarm-panel user number was returned in:

- `AlarmIMEIUserNumbers[].Number`

and:

- `ControllerPermitions[].AlarmUserNumber`

This validates that the authenticated user is associated with a specific alarm-panel user number for the device.

The API documentation additionally states that this user number is the value to be supplied to `RemoteArm`.

The actual IMEI and alarm-panel user number have been redacted from this document.

---

## 9. Observed Controller Permissions

The following values were returned for the associated device.

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

Additional values observed:

- `RemotePINUsage:  0`
- `PINLength:       null`
- `EncryptionKey:   null`
- `UserPIN:         null`

These values are recorded exactly as returned by the API.

No additional meaning is assigned to them unless supported by the API specification or validated through subsequent functional tests.

**Note:** `ControllerPermitions` is preserved exactly as returned by the API, including its spelling.

---

## 10. Important Observations

### 10.1 End-user authentication context

The response returned:

- `Administrator: false`
- `AdminRequest: false`

This is consistent with the end-user authentication context established during TEST-001 and TEST-002.

**Status:** VALIDATED

### 10.2 Remote arm/disarm capability

The associated controller returned:

- `RemoteArmDisarm: true`
- `EnabledRemoteArmDisarm: true`

These flags indicate that remote arm/disarm capability is enabled for this user/device association.

However, these fields alone do not demonstrate successful physical arm/disarm operation.

- Permission observed: **YES**
- Functional operation: **PENDING VALIDATION**

### 10.3 Zone bypass capability

The response returned:

- `BypassEnable: true`

This indicates that bypass capability is enabled for this user/device association.

- Permission observed: **YES**
- Functional operation: **PENDING VALIDATION**

### 10.4 Output capability

The response returned:

- `OutputEnable: true`
- `ViewExternalOutputs: true`

This indicates that output functionality is enabled and exposed to this user/device association.

- Permission observed: **YES**
- Functional operation: **PENDING VALIDATION**

### 10.5 Emergency-key visibility

The response returned:

- `ViewEmergencyKeys: true`

This indicates that emergency-key functionality may be exposed to the authenticated user.

No emergency operation was executed during this test.

- Capability observed: **YES**
- Functional operation: **NOT TESTED**

---

## 11. Authentication Validation

This test confirms that the `AccessToken` generated during TEST-002 is accepted by a protected API endpoint when sent using:

```http
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

The authentication flow validated up to this point is:

```text
CreateAuthorizationCode
        ↓
AuthCode
        ↓
CreateAccessToken
        ↓
AccessToken
        ↓
M2MOAuth2Token
        ↓
GetHAUserSettings
        ↓
Authenticated user and associated device data
```

---

## 12. Validation Result

**PASS**

Validated during this test:

- protected endpoint authentication;
- correct use of the `M2MOAuth2Token` header;
- retrieval of the authenticated end-user profile;
- retrieval of associated device information;
- retrieval of the alarm-panel user number;
- relationship between the device and alarm-panel user number;
- retrieval of controller permission values;
- persistence of the end-user authentication context;
- successful acceptance of `{}` as the request body.

Documented but not independently required for the successful test:

- an empty request body is the API-defined request format.

---

## 13. Security Notes

The following values were removed from this document:

- `AccessToken`
- `IMEI`
- `ClientID`
- User ID / UUID
- `UserName`
- `AlarmUserNumber`

The following values must never be included in manufacturer-facing test evidence:

- Passwords
- `AuthCode`
- `AccessToken`
- `RefreshToken`
- `UserPIN`
- Other authentication secrets

---

## 14. Pending Functional Validation

The permission fields returned by this endpoint describe capabilities associated with the user/device relationship.

They do not independently prove that the corresponding operation succeeds on the physical alarm panel.

The following remain pending:

- Remote arm
- Remote disarm
- Zone bypass
- Communicator outputs
- Panel PGMs
- Emergency functions

---

## 15. Test Classification Summary

| Item | Status |
|---|---|
| End-user profile retrieval | VALIDATED |
| Device association retrieval | VALIDATED |
| Alarm user number retrieval | VALIDATED |
| Number intended for RemoteArm | DOCUMENTED |
| Controller permissions retrieval | VALIDATED |
| RemoteArmDisarm permission | OBSERVED: true |
| Bypass permission | OBSERVED: true |
| Output permission | OBSERVED: true |
| Emergency-key visibility | OBSERVED: true |
| Remote arm operation | PENDING VALIDATION |
| Remote disarm operation | PENDING VALIDATION |
| Bypass operation | PENDING VALIDATION |
| Output operation | PENDING VALIDATION |
| PGM operation | PENDING VALIDATION |