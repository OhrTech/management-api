# Changelog

Changes to the OhrTech Management API (`openapi.json`). Within `v1` the API only adds.

## 2026-11-08: the Device SDK's server side

For apps that use the [Device SDK](https://docs.ohrtech.net/device-sdk/) on your devices.

- Device SDK access: `GetSdkAccess`, `SetSdkAccess`, `ClearSdkAccess`
  (`/v1/fleets/{fleetId}/apps/{appId}/sdk-access`): which areas an app may change on a fleet's
  devices (`wifi`, `network`, `kiosk`, `power`, `settings`), inherited down the fleet tree.
  `ListDeviceEvents` (`GET /v1/devices/{deviceId}/events`): what apps changed through the SDK, what
  the Agent refused, and apps' own events.
- Push: `SendPush` (`POST /v1/push`, up to 500 sealed and signed messages, `normal` or `high`
  priority, `ttl`, `collapseKey`; needs the new `SendPush` permission), `GetPushStatus`
  (`GET /v1/push/{id}`), `ListPushTokens` (`GET /v1/devices/{deviceId}/push-tokens`). A token ends
  when the app registers again or unregisters, is uninstalled, or the device no longer has the app
  in its fleet (`the token has ended (left_fleet)`).
- Keys and attestation: `VerifyKeyAttestation` (`POST /v1/devices/{deviceId}/key-attestations/verify`)
  for an app's TEE KEM keys; `VerifyAttestation` (`POST /v1/attestations/verify`), `GetAttestationRoot`
  and `GetAttestationStatus` (`/v1/attestations/root`, `/v1/attestations/status`, no sign-in), and
  `RevokeDeviceAttestation` (`POST /v1/devices/{deviceId}/attestation/revoke`).
- Managed app configuration: `GetAppConfigSchema` (`GET /v1/apps/{appId}/config-schema`), and
  `Get`/`Set`/`Clear` `FleetAppConfig` and `DeviceAppConfig`
  (`/v1/fleets/{fleetId}/apps/{appId}/config`, `/v1/devices/{deviceId}/apps/{appId}/config`). Secret
  values are sealed and shown as `"*"`.
- Kiosk: `ExitKiosk` (`POST /v1/devices/{deviceId}/kiosk/exit`): lift kiosk mode for some minutes
  without the exit PIN, or end that early.
- Crash reports and diagnostics: `ListAppCrashes` (`GET /v1/apps/{appId}/crashes`), `ListCrashReports`
  (`GET /v1/apps/{appId}/crashes/{signature}`), `ListDeviceCrashes` (`GET /v1/devices/{deviceId}/crashes`);
  `RequestLogBundle` (`POST /v1/devices/{deviceId}/apps/{appId}/log-bundles`) and `ListLogBundles`
  (`GET /v1/devices/{deviceId}/log-bundles`, with an hour's download link).
- USB devices: `GetUsbDevices`, `SetUsbDevices`, `ClearUsbDevices`
  (`/v1/fleets/{fleetId}/apps/{appId}/usb-devices`): devices and accessories an app may use without
  asking; `ListDevicePeripherals` (`GET /v1/devices/{deviceId}/usb-devices`): what is attached.
- Certificates for apps: `CreateIssuingCa`, `ListIssuingCas` (`/v1/orgs/{orgId}/issuing-cas`),
  `UploadIssuingCaCertificate` (`PUT .../{issuingCaId}/org-certificate`), `RevokeIssuingCa`
  (`POST .../{issuingCaId}/revoke`); `GetAppCertificates`, `SetAppCertificates`, `ClearAppCertificates`
  (`/v1/fleets/{fleetId}/apps/{appId}/certificates`); `ListDeviceCertificates`
  (`GET /v1/devices/{deviceId}/certificates`); `GetDeviceIdentityRoot` (`GET /v1/device-identity/root`,
  no sign-in).
- New settings: the kiosk exit PIN and its triggers, `apps.crash_reports`, `usb.devices` and
  `apps.certificates` (the last two set per app with the operations above).

## 2026-11-06: privileged permissions by OhrTech approval

- `ListAppPermissions` (`GET /v1/apps/{appId}/permissions`): the privileged Android permissions an
  app's versions request, each `granted`, `waiting` (not approved; the app runs without it) or
  `never`, with `approvedFor` (`org` or `app`) and the requesting `versionCodes`.
- `ListOrgPermissionApprovals` (`GET /v1/orgs/{orgId}/permission-approvals`): what OhrTech approved
  for an organisation; every app of the organisation that requests one gets it.
- OhrTech staff only: `ApproveOrgPermission`, `WithdrawOrgPermission`
  (`/v1/orgs/{orgId}/permission-approvals/{permission}`) and `ApproveAppPermission`,
  `WithdrawAppPermission` (`/v1/apps/{appId}/permission-approvals/{permission}`, OhrTech's apps).
  400 "Never granted", "No approval needed", "Platform only" or "Not a privileged permission".
- Apps the hub installs get the runtime and special-access permissions they request automatically.

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
