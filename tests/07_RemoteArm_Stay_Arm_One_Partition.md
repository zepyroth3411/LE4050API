# TEST-007B — RemoteArm / Stay Arm

## Test Information

**Test ID:** TEST-007B  
**Endpoint:** `RemoteArm`  
**Operation:** Stay Arm  
**Method:** `POST`  
**API Version:** v3  
**Environment:** Production  
**Status:** VALIDATED / PASS  

---

## 1. Objective

Validate remote Stay Arm on a specific alarm partition.

---

## 2. Endpoint

```http
POST https://app.m2mservices.com/CommonAdministrationService/api/v3/RemoteArm
```

---

## 3. Request

### Headers

```http
Content-Type: application/json
M2MOAuth2Token: <REDACTED_ACCESS_TOKEN>
```

### Body

```json
{
  "IMEI": "<REDACTED_IMEI>",
  "ArmingState": 2,
  "PartitionNumber": "1",
  "UserPIN": "<REDACTED_USER_PIN>",
  "ProtocolNumber": 4
}
```

---

## 4. Documented Behavior

According to the API specification:

- `ArmingState: 2` = Stay Arm

- `PartitionNumber: "1"` limits the operation to Partition 1.

The partition previously reported the following command definition:

- `OnStayCommand: #RCSTAYARM`

---

## 5. Observed Result

The API returned HTTP 200.

- `Success:     true`
- `ErrorCode:   0`
- `ErrorString: OK`

The returned partition state was:

- `DeviceState:    3`
- `DeviceStateExt: 11`

According to the API specification:

- `DeviceStateExt 11 = Stay Exit Delay`

The physical panel entered the Stay Arm sequence successfully.

---

## 6. Validation Result

**PASS**

Validated:

- `ArmingState: 2`;
- partition-specific Stay Arm;
- use of panel PIN;
- successful remote command;
- Stay Exit Delay state returned after the command;
- physical Stay Arm sequence initiated.

---

## 7. Security Notes

The following values were redacted:

- `AccessToken`
- `IMEI`
- `UserPIN`