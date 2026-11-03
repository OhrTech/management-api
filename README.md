# OhrTech Management API

The OpenAPI document of the OhrTech Management API, the cloud API at `https://api.ohrtech.net` that
your servers and tools use to manage your OhrTech devices: device status, managed settings, fleets,
messages and tasks, remote control (with a viewer you can embed in your web app) and remote ADB.

**Documentation: <https://docs.ohrtech.net/management-api/>**, with a quickstart, guides, complete
recipes and the [interactive API reference](https://docs.ohrtech.net/management-api/reference/).

This repository holds:

- [`openapi.json`](openapi.json): the OpenAPI 3 document, exactly as the API serves it at
  `https://api.ohrtech.net/v1/openapi.json`. Generate a client from it in any language.
- [`CHANGELOG.md`](CHANGELOG.md): what changed, and when. Within `v1` the API only adds: new
  operations, new optional fields, new values. Anything else becomes `v2`, served alongside `v1`.

> The Management API runs on your servers and manages many devices. The **Device SDK**
> (`device-sdk-*`, coming later) is for apps running *on* a device, to work with the OhrTech Agent
> there. Both are parts of the OhrTech SDK.

## SDKs

| Language | Package | Source |
|---|---|---|
| .NET | [`OhrTech.Management`](https://www.nuget.org/packages/OhrTech.Management) | [management-sdk-dotnet](https://github.com/OhrTech/management-sdk-dotnet) |

## License

[MIT](LICENSE).
