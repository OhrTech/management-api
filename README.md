# OhrTech Management API

Manage your organisation's OhrTech devices from your own servers and tools: device status, managed
settings, fleets, messages and tasks, remote control (with a viewer you can embed in your web app)
and remote ADB. Everything a member of your organisation can do to a device in the
[portal](https://portal.ohrtech.net), over HTTPS.

* **Base URL:** `https://api.ohrtech.net`
* **Reference:** [`openapi.json`](openapi.json) (OpenAPI 3), browsable at <https://api.ohrtech.net/docs>
* **Embedding remote control:** [embedding.md](embedding.md)
* **Changes:** [CHANGELOG.md](CHANGELOG.md). Within `v1` the API only adds: new operations, new
  optional fields, new values. Anything else becomes `v2`, served alongside `v1`.

> This repo is the **Management API**, used from servers and desktop tools. The **Device SDK**
> (`device-sdk-*`, coming later) is a different thing: it is for apps running *on* a managed device
> to talk to the OhrTech Agent there.

## SDKs

| Language | Package | Source |
|---|---|---|
| .NET | [`OhrTech.Management`](https://www.nuget.org/packages/OhrTech.Management) | [management-sdk-dotnet](https://github.com/OhrTech/management-sdk-dotnet) |

Any OpenAPI generator works with `openapi.json` too.

## Signing in

Every call except `/v1/auth/*` takes `Authorization: Bearer <token>`, from one of two kinds of caller:

| Caller | Use it for | Token |
|---|---|---|
| **Service account** | Servers, integrations, automation | `ohrs_…`, long-lived, revocable. Create the account and its tokens in the portal under **Service accounts** (needs the ManageMembers permission). |
| **Person** | Tools people run | Device authorization flow ([RFC 8628](https://www.rfc-editor.org/rfc/rfc8628)): `POST /v1/auth/device` → show the person the code and link → they sign in to the portal as usual and approve → `POST /v1/auth/token` returns an access token (1 hour) and a refresh token (30 days from last use) → `POST /v1/auth/refresh`. |

A token only says who the caller is. What the caller may do is checked on every call, so a removed
permission, a disabled service account or a locked-out person stops working at once. Signing out
everywhere in the portal, or changing the password, ends a person's API sessions too.

## Permissions

Per organisation. Every member of an organisation already holds the first five; a service account
holds only what it was granted.

| Permission | Lets the caller |
|---|---|
| `ViewDevices` | See devices and fleets: status, installed apps, enrolments, tasks, sessions; ask a device to check in |
| `ViewSettings` | See devices' managed settings and what the devices reported |
| `EditSettings` | Change devices' managed settings; push and re-apply them |
| `MoveDevices` | Move devices between fleets |
| `SendMessages` | Show messages on devices' screens |
| `RemoteControl` | Remote control sessions |
| `RemoteAdb` | Remote ADB sessions |

**A service account's limits** belong to the account, so every token of the account has the same
access. Set them in the portal under Service accounts:

* **Devices:** fleets (each including every fleet below it, also ones created later) and single
  devices. None means the whole organisation. A scoped account may not move a device out of its scope.
* **Networks:** CIDR ranges its tokens may be used from (`203.0.113.0/24`, `2001:db8::/32`, single
  addresses). None means anywhere. A token used from elsewhere gets 401 saying so.
* **Embed sites:** the only web origins its remote control sessions may be embedded in (see
  [embedding.md](embedding.md)).

`GET /v1/me` shows the caller its organisations, effective permissions and limits, and
`GET /v1/devices/{id}` lists `capabilities.yourPermissions` for that device. A device outside the
caller's organisations or scope is always 404, never 403.

## What you can do

| Area | Calls |
|---|---|
| Devices and fleets | `GET /v1/devices?clientId&fleetId&online`, `GET /v1/devices/{id}`, `PUT /v1/devices/{id}/fleet`, `GET …/enrolments`, `GET …/apps`, `POST …/check-in`, `GET /v1/fleets` |
| Settings | `GET /v1/settings/catalog`, `GET\|PUT\|PATCH /v1/devices/{id}/settings`, `POST …/settings/push` |
| Messages and tasks | `POST /v1/devices/{id}/messages`, `GET\|POST …/tasks`, `GET …/tasks/{taskId}` |
| Remote control | `GET\|POST /v1/devices/{id}/remote-control-sessions`, `GET\|DELETE /v1/remote-control-sessions/{id}`, `POST …/join` |
| Remote ADB | `GET\|POST /v1/devices/{id}/adb-sessions`, `GET\|DELETE /v1/adb-sessions/{id}` (the `ohr` CLI runs the tunnel) |

A few behaviours worth knowing:

* **Settings** follow the portal's Settings tab: platform defaults, then the device's fleets from the
  top down, then the device's own overrides. `GET` shows `own`, `inherited`, `effective` and what the
  device `reported`. `PUT` replaces the device's overrides; `PATCH` changes some (`null` removes one).
  Both validate everything first (400 with per-setting `errors`, nothing stored), then send the policy
  to the device. `POST …/settings/push` with `{"reapply": [keys]}` re-applies settings the person at
  the device has changed.
* **Secrets never come back.** Wi-Fi passwords, WEP keys and enterprise credentials read as `"*"`, and
  a screen lock's code is left out. Send `"*"` back unchanged to keep the stored value.
* **Moving a device:** `PUT /v1/devices/{id}/fleet` with `{"fleetId": 12}`, or `{"fleetId": null}` for
  no fleet. `fleetId` is required, so a missing value never moves a device by accident.
* **Check-in:** `POST …/check-in` returns 202; a reachable device checks in within seconds (on mobile
  data it may wait for its next scheduled check-in). `online` means checked in within 20 minutes.
* **Tasks:** you can create `ShowToast`; OhrTech creates the rest (updates, settings). Page with
  `?limit&before=<last taskId>`, newest first.

## Errors

Errors are [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) problem details
(`application/problem+json`) with a `title` and a `detail` that says what to change:

| Status | Means |
|---|---|
| 400 | The request is invalid; settings errors are listed per setting under `errors` |
| 401 | No token, an expired or revoked token, or a service account token used outside its networks |
| 403 | You can see the device but lack the permission (the `detail` names it) |
| 404 | No such device, or not one you can reach |
| 409 | Conflicts, such as someone else's remote control session (`blockingSession`; retry with `takeOver`) |
| 429 | Too many sign-in attempts; wait and retry |

## License

[MIT](LICENSE) for this repo's documents and the OpenAPI document.
