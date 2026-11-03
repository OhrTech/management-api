# Embedding remote control

Show a device's screen inside your own web app, such as a help desk, and let your staff control it
there without signing in to the OhrTech portal.

## 1. Open a session (your server)

```http
POST /v1/devices/42/remote-control-sessions
Authorization: Bearer ohrs_…
Content-Type: application/json

{"embedOrigins": ["https://support.example.com"]}
```

Optional fields: `consent` (how the person at the device is asked: within what your organisation's
policy allows) and `takeOver: true` (end someone else's open session, after a 409 that names it in
`blockingSession`). The 201 response:

```json
{
  "session": {"sessionId": "…", "deviceId": 42, "status": "pending", "consent": "notify", "expiresAt": "…"},
  "connection": {"url": "wss://live.ohrtech.net", "token": "…"},
  "embed": {"url": "https://api.ohrtech.net/embed/v1/remote-control/…#token=…", "expiresAt": "…"},
  "policyConsent": "notify"
}
```

Needs the `RemoteControl` permission on that device. `connection` is for a fully custom viewer; most
apps only need `embed`.

## 2. Show it (your page)

```html
<iframe src="EMBED_URL" allow="fullscreen; clipboard-read"
        style="width:100%;height:720px;border:0"></iframe>
```

* **Who may frame it:** only the `embedOrigins` named when the session was opened (at most 5;
  `https://host[:port]`, or `http://localhost` for development). Browsers enforce this with CSP
  `frame-ancestors`. A session opened without any can still be opened directly in a new tab, but
  framed nowhere.
* If a service account has **Embed sites** set in the portal, those are the only origins it may name,
  and a session that names none gets all of them.
* **The link is a credential.** Treat it like a password until `embed.expiresAt` (10 minutes): hand
  it only to the page that shows it. Its token sits after `#`, which browsers never send to a server
  or in a `Referer`. Reloading the page within that time reconnects; after that, call
  `POST /v1/remote-control-sessions/{id}/join` for a fresh link. A viewer that is connected stays
  connected.
* **When the viewer leaves**, for 60 seconds, the session ends. You can also end it with
  `DELETE /v1/remote-control-sessions/{id}` or the `end` command below.

**Options** go in the fragment after the token: `#token=…&toolbar=0` hides the toolbar, `&end=0` hides
only its End button.

**Input:** pointer = one finger, right click = Back, mouse wheel = scroll; keyboard and paste while
the screen has focus. The toolbar has Back, Home, Recents, volume, Power, Refresh, Full screen and End.

## 3. Talk to it (optional)

The viewer reports to your page with `postMessage`, only to the session's embed origins:

```js
window.addEventListener("message", e => {
  if (e.origin !== "https://api.ohrtech.net" || e.data?.source !== "ohrtech-remote-control") return;
  switch (e.data.type) {
    case "state":  // e.data.state: waiting | awaiting_consent | active | ended | error; e.data.detail
    case "video":  // e.data.width, e.data.height: the device screen's size
    case "device": // the device's own report: state, consent, width, height, reason
  }
});
```

Each message also carries `session`, the session id. Your page can send commands back (accepted only
from the embed origins):

```js
const viewer = document.querySelector("iframe").contentWindow;
viewer.postMessage({type: "key", key: "home"}, "https://api.ohrtech.net");
viewer.postMessage({type: "text", text: "hello"}, "https://api.ohrtech.net");
viewer.postMessage({type: "refresh"}, "https://api.ohrtech.net"); // ask for a full frame
viewer.postMessage({type: "focus"}, "https://api.ohrtech.net");   // give the screen keyboard focus
viewer.postMessage({type: "end"}, "https://api.ohrtech.net");     // end the session
```

Keys: `back` `home` `recents` `power` `volume_up` `volume_down` `notifications` `enter` `backspace`
`delete` `tab` `escape` `left` `right` `up` `down` `page_up` `page_down` `move_home` `move_end`.
