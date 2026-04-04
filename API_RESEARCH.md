# Te Piha — Smart Home API Research

*Researched 2026-04-04. Focus: feasibility for a guest-facing web app with remote control.*

---

## Summary Matrix

| # | Product | Public Cloud API? | Local API? | Web App Feasible? |
|---|---------|:-:|:-:|:-:|
| 1 | anywAiR (Fujitsu heat pump) | ❌ | ✅ Local REST | ⚠️ Local only |
| 2 | BluOS (Bluesound speakers) | ❌ | ✅ Local REST | ⚠️ Local only |
| 3 | TP-Link Deco | ❌ | ⚠️ Unofficial | ❌ Not practical |
| 4 | EZVIZ cameras | ✅ Cloud API | ❌ | ✅ Yes |
| 5 | Master blinds (Automate/Pulse) | ❌ | ✅ Local API | ⚠️ Local only |
| 6 | Holman iGardener | ❌ | ❌ BLE only | ❌ No |
| 7 | Yale Home (smart locks) | ✅ Via yalexs/Seam | ❌ | ✅ Yes (with effort) |
| 8 | Davis WeatherLink | ✅ Cloud REST v2 | ✅ Local HTTP | ✅ Yes |
| 9 | Holman WiFi Water Timer | ⚠️ Tuya-based cloud | ⚠️ Tuya local | ⚠️ Via Tuya |
| 10 | Mammotion mower | ✅ Cloud MQTT | ❌ | ⚠️ Unofficial |
| 11 | BlueEye pool monitor | ❌ | ❌ | ❌ No evidence |

---

## Detailed Findings

### 1. anywAiR (Fujitsu General Heat Pump / AC)

- **Type:** Two variants exist:
  - **Ducted controller** (rebranded Advantage Air / MyAir) — has a **local REST API on port 10211**. Well-documented via Home Assistant's Advantage Air integration.
  - **Wi-Fi Adaptor II** (split systems) — uses IntesisHome cloud platform. Can be controlled via IntesisHome API (cloud, requires IntesisHome account credentials).
- **Local API (ducted):** HTTP GET/POST on `http://<ip>:10211/`. Returns JSON. Can set temperature, mode, fan speed, zones on/off.
- **Cloud API (splits via IntesisHome):** WebSocket-based cloud API. Requires IntesisHome credentials.
- **Auth:** No auth on local API (ducted). IntesisHome requires account login.
- **Actions:** Set temp, mode (heat/cool/auto/fan), fan speed, zone control (ducted), on/off.
- **Web app verdict:** ⚠️ Ducted variant works great on local network — simple REST calls, no auth. Split system via IntesisHome is more complex. **Need to confirm which variant is installed.**

### 2. BluOS (Bluesound Multiroom Speakers)

- **Type:** Local HTTP REST API on port 11000.
- **Official docs:** [BluOS Custom Integration API v1.7 PDF](https://bluos.io/wp-content/uploads/2025/06/BluOS-Custom-Integration-API_v1.7.pdf) — officially published by Lenbrook/BluOS.
- **Auth:** None. Open local API.
- **Endpoints:** `http://<player-ip>:11000/<command>`
  - `/Status` — current playback status
  - `/Play` — play, `/Pause`, `/Stop`, `/Skip`, `/Back`
  - `/Volume?level=50` — set volume
  - `/Presets` — list presets, `/Preset?id=X` — load preset
  - `/Sources` — list available sources
  - `/AddSlave`, `/RemoveSlave` — grouping
- **Actions:** Play/pause/skip, volume, source selection, presets, grouping/ungrouping, browse music.
- **Web app verdict:** ⚠️ **Excellent local API** — simple HTTP calls, no auth, well-documented. But local network only. A web app served from a local server (or with a local proxy) would work perfectly. No cloud API exists.

### 3. TP-Link Deco (Mesh WiFi)

- **Type:** No official API. TP-Link has confirmed no local API is available.
- **Unofficial:** Community reverse-engineering exists (GitHub gists for X90, Go wrapper for M4). These hit the local web interface's encrypted API. Fragile and model-specific.
- **Auth:** Complex — uses RSA-encrypted password exchange with the local web UI.
- **Actions (unofficial):** List connected clients, reboot, basic status.
- **Web app verdict:** ❌ **Not practical.** No public API, unofficial methods are fragile. WiFi management isn't a useful guest control anyway — recommend excluding from web app.

### 4. EZVIZ / EZView (Security Cameras)

- **Type:** ✅ **Official Cloud API** — EZVIZ Open Platform at `open.ezviz.com` / `isgpopen.ezviz.com`.
- **Auth:** Developer account required. AppKey + AppSecret → access token. OAuth-style.
- **Endpoints:** REST API for device management, live streaming, PTZ control, event/alarm queries.
- **Actions:** 
  - Live video streaming (RTMP/HLS via cloud)
  - PTZ control (pan/tilt/zoom)
  - Capture snapshots
  - Arm/disarm motion detection
  - View event history
  - Device status queries
- **SDKs:** Web JS SDK, iOS, Android SDKs available.
- **Web app verdict:** ✅ **Very feasible.** Official cloud REST API + Web JS SDK means you can embed live camera views and controls directly in a web page. Need to register as developer and get API credentials.

### 5. Master (Motorized Blinds/Sheers/Curtains)

- **Note:** "Master" likely refers to blinds using **Automate** (by Rollease Acmeda) motors with a **Pulse hub**. This is the dominant motorized blind platform in AU/NZ.
- **Type:** Local API via Pulse Hub.
  - **Pulse v1:** Custom TCP protocol on local network. Python library: `aiopulse`.
  - **Pulse v2:** Updated protocol. Python library: `aiopulse2`. Home Assistant integration available.
  - **Pulse PRO:** Newer hub, Matter-compatible.
- **Auth:** None on local network.
- **Actions:** Open/close/stop blinds, set position (0-100%), list blinds, get status.
- **Cloud:** The Automate Pulse app uses a cloud service but no public cloud API exists.
- **Web app verdict:** ⚠️ **Local network only.** Works well with a local proxy/server. Simple commands: set position, open, close. Need to confirm they have a Pulse hub and which version.

### 6. Holman iGardener (Garden Irrigation)

- **Type:** **Bluetooth only.** The iGardener app communicates with timers via BLE (Bluetooth Low Energy).
- **No WiFi, no cloud, no API.** This is the older/basic Holman line.
- **Actions via BLE:** Set watering schedules, manual run, configure zones. But only from the phone app within BLE range (~10m).
- **Web app verdict:** ❌ **Not feasible.** Bluetooth-only means no network API whatsoever. Cannot be controlled from a web page. See #9 (Holman WiFi) for the WiFi-capable version.

### 7. Yale Home (Smart Locks)

- **Type:** Cloud API via unofficial but well-maintained libraries.
  - **yalexs** Python library (GitHub: `bdraco/yalexs`) — reverse-engineered Yale/August cloud API. Used by Home Assistant's official Yale integration.
  - **Seam API** (seam.co) — commercial third-party universal lock API that supports Yale. REST API with proper docs. Paid service.
- **Auth:** 
  - yalexs: Email + password + 2FA verification code. Access token cached.
  - Seam: API key + OAuth connect flow.
- **Actions:** Lock/unlock, get lock status, view lock history, manage access codes (pin codes), check battery.
- **Security note:** ⚠️ Exposing lock control to a guest web app is a **significant security consideration**. Recommend read-only status display at most, or carefully scoped access codes.
- **Web app verdict:** ✅ **Technically feasible** via yalexs cloud API or Seam. But security implications are serious — lock/unlock via a web app needs careful access control. Better approach: use Yale's built-in guest access codes rather than API control.

### 8. Davis WeatherLink (Weather Station)

- **Type:** ✅ **Official REST API (v2)** — best-documented API in this list.
  - **Cloud API v2:** `https://api.weatherlink.com/v2/` — full weather data access.
  - **Local API:** WeatherLink Live device serves JSON over HTTP on local network + UDP broadcasts.
- **Auth (Cloud):** API Key + API Secret. Register at WeatherLink Developer Portal. HMAC-signed requests.
- **Auth (Local):** None — open HTTP on local network.
- **Endpoints (Cloud):**
  - `/current/{station-id}` — current conditions
  - `/historic/{station-id}` — historical data
  - `/stations` — list stations
  - `/sensors` — sensor catalog
- **Actions:** Read-only (weather data). Temperature, humidity, wind, rain, barometric pressure, UV, solar radiation, etc.
- **Web app verdict:** ✅ **Excellent.** Official, well-documented cloud REST API. Perfect for displaying weather data on a guest web app. Read-only by nature. Free tier available.

### 9. Holman WiFi Water Timer (WX1)

- **Type:** Uses **Tuya platform** under the hood. WiFi-connected via Holman Home app.
- **Cloud:** Tuya Cloud API is available (developer.tuya.com). Requires Tuya IoT developer account. REST API.
- **Local:** Tuya local protocol (encrypted). Can be accessed via `tinytuya` Python library with device keys.
- **Auth:** Tuya Cloud requires OAuth 2.0 (client_id + secret). Local requires extracting device keys from Tuya cloud.
- **Actions:** Turn zones on/off, set watering duration, get status, set schedules.
- **Challenges:** Getting Tuya device keys requires linking Holman Home account → Tuya Smart app → Tuya IoT Platform. Process is documented but fiddly.
- **Web app verdict:** ⚠️ **Possible via Tuya Cloud API** but setup is complex. Once configured, REST calls can turn watering on/off. The Tuya Cloud API has rate limits and requires maintaining auth tokens.

### 10. Mammotion (Robot Lawn Mower)

- **Type:** Cloud control via MQTT (Mammotion's cloud). No official public API.
  - **PyMammotion** (GitHub: `mikey0000/PyMammotion`) — reverse-engineered Python library supporting MQTT cloud, BLE, and WiFi communication.
  - Home Assistant integration available via HACS.
- **Auth:** Mammotion account email + password → cloud MQTT session.
- **Actions:** Start/stop mowing, return to dock, get status (battery, position, mowing progress), set mowing schedule.
- **Models supported:** Luba, Luba 2, Yuka series.
- **Web app verdict:** ⚠️ **Technically possible** via PyMammotion cloud MQTT. But it's unofficial, could break with firmware updates. For a guest app, basic controls (start/stop/dock/status) would be most useful.

### 11. BlueEye (Pool/Water Monitoring)

- **Type:** ❌ **No public API found.** 
- **Research notes:** "BlueEye" as a pool monitor doesn't appear to have a well-known developer platform. Search results returned unrelated products (Blue Eye security monitoring, Blue Connect by Riiot Labs, etc.).
- **Possible confusion:** Could this be **Blue Connect** (Riiot Labs), **Sutro**, **pHin**, or another pool monitor brand? Need to confirm exact product.
- **If Blue Connect:** No public API. Some HA users have hacked email-based integrations.
- **Web app verdict:** ❌ **Not feasible without identifying the exact product.** Need clarification on which pool monitor is in use.

---

## Recommendations for Guest Web App

### Tier 1 — Ready to integrate (cloud APIs, well-documented)
1. **Davis WeatherLink** — Official REST API, read-only weather display. Easiest win.
2. **EZVIZ cameras** — Official cloud API + JS SDK for live camera views.

### Tier 2 — Feasible with local proxy server
3. **BluOS speakers** — Excellent local REST API. Perfect for guest music control.
4. **anywAiR heat pump** — Local REST API (ducted) or IntesisHome cloud (splits).
5. **Master/Automate blinds** — Local API via Pulse hub. Open/close/position.

### Tier 3 — Possible but complex or unofficial
6. **Yale locks** — Cloud API exists but security concerns for guest access. Consider display-only.
7. **Holman WiFi timer** — Via Tuya Cloud API. Complex setup, ongoing maintenance.
8. **Mammotion mower** — Unofficial cloud MQTT. Basic start/stop/status.

### Tier 4 — Not feasible
9. **TP-Link Deco** — No API, not useful for guests anyway.
10. **Holman iGardener** — Bluetooth only, no network API.
11. **BlueEye pool** — No API found. Need to identify exact product.

---

## Architecture Note

For Tier 2 (local API) products, you'd need a **local proxy/bridge server** on the same network as the devices. This server would:
- Expose a REST API accessible from the internet (with auth)
- Forward commands to local devices (BluOS, anywAiR, Automate hub)
- This could be a Raspberry Pi, NAS, or any always-on device running Node.js/Python

A **Home Assistant** instance could serve as this bridge — it already has integrations for anywAiR, BluOS, Automate Pulse, Yale, and Mammotion. Its REST API could then be exposed (with auth) to the guest web app.
