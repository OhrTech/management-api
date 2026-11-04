# Changelog

Changes to the OhrTech Management API (`openapi.json`). Within `v1` the API only adds.

## 2026-11-05: update timing, app kinds, fleet policies

- `GetUpdateTiming`, `SetUpdateTiming`, `ClearUpdateTiming` (`/v1/fleets/{fleetId}/update-timing`):
  when a fleet's devices install app updates: `screen_off` (default), `window` (`days`, `start`,
  `end`, device local time), `window_or_screen_off` or `immediate`; inherited down the fleet tree.
- `ManagedAppDto.kind`: `org`, `catalog` or `platform`; `orgId` and `orgName` are null for catalog
  and platform apps, whose versions OhrTech releases. `ListApps` takes `kind` and lists them too.
  `AppVersionDto.notes` and `security`; `SetAppVersionStateRequest.state` is optional, with `notes`
  and `security` (OhrTech's apps). `CreateAppRequest.kind`. 409 "Reserved package" for a platform
  app's package; 409 "Platform app" when assigning one to a fleet.
- `GetFleetAppPolicy`, `SetFleetAppPolicy`, `ClearFleetAppPolicy`
  (`/v1/fleets/{fleetId}/apps/{appId}/policy`): `automatic`, or `manual` with an approved version,
  per fleet and catalog/platform app; `reminders`.
- Rollouts: statuses `held` and `waiting`, per-device `targetVersionCode` and `policy`, and `held`
  and `waiting` counts.

## 2026-11-04: ids with a type prefix

The one change to `v1` that is not an addition, made the day after it was published:

- Every id is a string whose prefix says what it names, such as `dev_01jftb4z0y9nn9yesabj71tyt0`:
  `org_` an organisation, `flt_` a fleet, `dev_` a device, `app_` an app, `tsk_` a task, `usr_` a
  person, `sa_` a service account, `adb_` and `rc_` remote ADB and remote control sessions. Ids were
  integers, and sessions UUIDs. Path parameters declare each kind's pattern.
- Organisations are `org` throughout: `orgId` and `orgName` (were `clientId` and `clientName`),
  `MeDto.orgs` (was `clients`), and the `orgId` query parameter.
- An id of another kind, such as a fleet's id where a device's belongs, is 400 ("Not a device id").

## 2026-11-04: your apps

- `ListApps`, `CreateApp`, `GetApp`, `UpdateApp` (`/v1/apps`), `ListAppVersions`,
  `UploadApk` (`POST /v1/apps/{appId}/versions`, the body is the APK; refused with 400
  "Wrong app" if it is another package, 409 if the version is not newer or the signing key differs),
  `SetAppVersionState` (`held`, `available` or `enforced`), `GetAppRollout`, and a fleet's apps
  (`ListFleetApps`, `AddFleetApp`, `RemoveFleetApp`).
- Several apps may share a package (flavours for different fleets); a fleet gets at most one app per
  package.
- The permission `ManageApps`.

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
