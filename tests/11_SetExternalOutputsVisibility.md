# TEST-011 — SetExternalOutputsVisibility

## Test Information

**Test ID:** TEST-011  
**Endpoint:** `setexternaloutputsvisibility`  
**Operation:** Show / Hide External Outputs  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS

---

## 1. Objective

Validate control of the visibility of the panel output / PGM section presented to the end user in the official M2M application.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/setexternaloutputsvisibility
```

---

## 3. Authentication

```http
Content-Type: application/json
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

---

## 4. Documented Behavior

The endpoint controls whether external / panel outputs are displayed to the end user.

- `ViewExternalOutputs: true` → show outputs
- `ViewExternalOutputs: false` → hide outputs

The operation controls visibility only.

---

## 5. Test A — Hide Outputs

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ViewExternalOutputs": false
}
```

### Response

```text
Success: true
ErrorCode: 0
ErrorString: OK
```

The response also returned updated configuration data containing:

```text
ViewExternalOutputs: false
```

### Observed Result

The PGM / output section disappeared from the official M2M application.

**Result:** PASS / VALIDATED

---

## 6. Test B — Restore Outputs

### Request

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ViewExternalOutputs": true
}
```

### Response

```text
Success: true
ErrorCode: 0
ErrorString: OK
```

The updated configuration returned:

```text
ViewExternalOutputs: true
```

### Observed Result

The PGM / output section became visible again in the official M2M application.

**Result:** PASS / VALIDATED

---

## 7. Validation Result

**PASS / VALIDATED**

Validated behavior:

| ViewExternalOutputs | Application Behavior | Result |
|---|---|---|
| false | PGM / output section hidden | PASS |
| true | PGM / output section visible | PASS |

---

## 8. Observed Response Behavior

The endpoint returned more data than a simple success envelope.

The response included an updated configuration object where:

- `ViewExternalOutputs`

reflected the value sent in the request.

---

## 9. Security Notes

The following values were redacted:

- `AccessToken`
- `IMEI`
- User identifiers
- Controller identifiers