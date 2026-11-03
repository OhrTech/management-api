# Changelog

Changes to the OhrTech Management API (`openapi.json`). Within `v1` the API only adds.

## 2026-11-03: corrections

- `OpenRemoteControlRequest.takeOver` is optional (default `false`); it was published as required.
- `OsDto` fields have descriptions. `os.securityPatch` is the patch level from the device's enrolment
  attestation and does not follow later OS updates.

## 2026-11-03: v1

First public version: devices and fleets, managed settings, messages and tasks, remote control with
an embeddable viewer, remote ADB, person sign-in (device code flow) and service accounts with
per-account permissions, device scope, network allowlist and embed sites.
