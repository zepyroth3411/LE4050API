# TEST-010 — SetExternalDeviceName

## Test Information

**Test ID:** TEST-010  
**Endpoint:** `SetExternalDeviceName`  
**Operation:** Rename Outputs and Zone Labels  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate modification of user-facing labels for:

- controllable outputs;
- alarm zones.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/SetExternalDeviceName
```

---

## 3. Authentication

```http
Content-Type: application/json
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

---

## 4. Test A — Rename Output

A previously discovered output was selected using its internal ID.

The API documentation specifies that ID refers to the output's discovery ID, not its DevicePIN.

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ID": 0,
  "Name": "PGM Test 1"
}
```

### Result

```json
{
  "Success": true,
  "ErrorCode": 0,
  "ErrorString": "OK",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

### Observed Result

The output label changed successfully in the official M2M application.

The previously displayed output name was replaced by:

```text
PGM Test 1
```

**Result:** PASS / VALIDATED

---

## 5. Test B — Rename Zone

The same endpoint was tested using the optional ZoneTypes collection.

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ID": 0,
  "Name": "PGM Test 1",
  "ZoneTypes": [
    {
      "Code": 1,
      "Name": "Zone Type 1"
    }
  ]
}
```

### Result

```json
{
  "Success": true,
  "ErrorCode": 0,
  "ErrorString": "OK",
  "ResponseBody": null,
  "SuccessCode": 0
}
```

### Observed Result

The label associated with zone 1 was successfully changed.

The new zone label was visible in the official M2M application.

```text
Zone 1 → Zone Type 1
```

**Result:** PASS / VALIDATED

---

## 6. ZoneTypes Behavior

The tested structure was:

```json
"ZoneTypes": [
  {
    "Code": 1,
    "Name": "Zone Type 1"
  }
]
```

Observed interpretation:

- `Code` → zone number
- `Name` → new user-facing zone label

The zone number itself was not changed.

Only its displayed label was modified.

---

## 7. Validation Result

**PASS / VALIDATED**

Successfully validated:

| Function | Result |
|---|---|
| Rename output using output ID | PASS |
| New output name visible in M2M app | PASS |
| Rename zone using ZoneTypes | PASS |
| New zone name visible in M2M app | PASS |
| API returns ErrorCode: 0 | PASS |

---

## 8. Security Notes

The following values were redacted:

- `AccessToken`
- `IMEI`
- Output internal ID