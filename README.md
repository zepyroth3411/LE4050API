# LE4050API

Independent validation records for the M2M Mobile Integration API v3 using an M2M LE4050M-LA communicator connected to a DSC PowerSeries NEO HS2064 panel.

> This repository documents observed API behavior from controlled tests. It is not an official manufacturer specification.

## Scope

The validation covers authentication, device discovery, live status, remote arm/disarm, output control, zone bypass, panel emergency keys and user-facing output/zone configuration.

Base API:

```text
https://app.m2mservices.com/CommonAdministrationService/api/v3
```

## Validation Summary

| Area | Result |
|---|---|
| Authentication and token creation | PASS |
| Token regeneration | PASS |
| User/device settings | PASS |
| Device discovery | PASS |
| Live status retrieval | PASS |
| Arm Away | PASS |
| Stay Arm | PASS |
| Arm With Automatic Bypass | PARTIAL |
| Disarm | PASS |
| Output ON/OFF | PASS |
| RemoteBypassExt | NOT VALIDATED |
| Fire / Medical / Panic emergency keys | PASS |
| Rename output / zone labels | PASS |
| Show / hide output section | PASS |

## Repository Structure

```text
tests/
  Endpoint-by-endpoint controlled validation records

evidence/
  Sanitized full API responses used as supporting evidence

findings/
  Cross-test discrepancies, clarification requests and integration feedback

Insomnia_APICollection.yaml
  Sanitized Insomnia request collection

mobile-integration.html
  API reference used during validation
```

Start with:

- `tests/00_Test_Context.md` for methodology and the complete test matrix;
- `findings/90_API_Findings_and_Feedback.md` for consolidated clarification requests.

## Security

The public repository intentionally redacts:

- usernames and passwords;
- authorization/access/refresh tokens;
- panel PINs;
- IMEI and serial numbers;
- SIM/ICCID values;
- client, user and controller identifiers;
- other installation-specific identifiers.

The Insomnia collection contains placeholders and must be populated locally before use.

## Validation Terminology

- **DOCUMENTED** — explicitly stated by the supplied API specification.
- **OBSERVED** — returned or seen during testing.
- **VALIDATED** — successfully reproduced in a controlled test.
- **NOT VALIDATED** — tested but not successfully reproduced.

## Current Result

Most tested operations behaved as documented. Two areas require additional clarification: automatic bypass through `RemoteArm` with `ArmingState: 6`, and direct zone bypass through `RemoteBypassExt`. Additional cross-test observations are recorded in the findings document.
