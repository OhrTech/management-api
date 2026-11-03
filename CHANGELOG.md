# Changelog

Changes to the OhrTech Management API (`openapi.json`). Within `v1` the API only adds.

## 2026-11-03: shell commands, current security patch

- `POST /v1/devices/{deviceId}/shell` (`RunShellCommand`) runs a command on a device as `adb shell`
  would. It needs `RemoteAdb`. The finished task's `shell` holds `exitCode`, `stdout`, `stderr` (the
  first 64 KiB of each), `truncated`, `timedOut`, `durationMs` and `error`.
- `TaskDto.createdBy`: the person or service account that created a task.
- `os.securityPatch` is the Android security patch level the device runs now (`YYYY-MM-DD`), reported
  at every check-in; it was the enrolment attestation's `YYYYMM`. The device list gains
  `securityPatch` and `shellCommands`, and `capabilities` gains `shellCommands`.

## 2026-11-03: corrections

- `OpenRemoteControlRequest.takeOver` is optional (default `false`); it was published as required.
- `OsDto` fields have descriptions. `os.securityPatch` is the patch level from the device's enrolment
  attestation and does not follow later OS updates.

## 2026-11-03: v1

First public version: devices and fleets, managed settings, messages and tasks, remote control with
an embeddable viewer, remote ADB, person sign-in (device code flow) and service accounts with
per-account permissions, device scope, network allowlist and embed sites.
