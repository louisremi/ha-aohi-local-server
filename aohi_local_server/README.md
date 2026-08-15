# AOHI Local Server — Home Assistant add-on

Run AOHI smart chargers **entirely on your own network**, with no cloud account and no internet
dependency.

## Install

1. In Home Assistant: **Settings → Add-ons → Add-on Store**
2. Top-right menu → **Repositories** → add
   `https://github.com/louisremi/ha-aohi-local-server`
3. Install **AOHI Local Server**, then start it

Then pair a charger to it (see below) and add it to Home Assistant with the
[AOHI Smart Charger integration](https://github.com/louisremi/ha-aohi-charger), choosing
**Local server** when asked.

## What it is for

The [AOHI Smart Charger integration](https://github.com/louisremi/ha-aohi-charger) normally talks
to AOHI's cloud. This add-on replaces that cloud: the charger connects here instead, so control
keeps working when AOHI's servers or your internet connection are unavailable.

It is **entirely optional**. The integration works fine against the cloud without it.

## Before you start — this is not reversible in place

Pointing a charger at this add-on requires a **factory reset and Bluetooth reprovisioning**, and
afterwards it will no longer work with the official AOHI mobile app. Going back means another
factory reset and re-adding it in the app.

A charger provisioned here is also **committed to this address**. If the add-on is stopped, or the
machine running Home Assistant is off, that charger has nothing to talk to and is simply
unreachable until it comes back. Consider keeping a second charger on the cloud while you decide
whether this suits you.

## Pairing a charger

1. Start the add-on and read its log; it prints the two values you need.
2. Factory reset the charger so it advertises over Bluetooth.
3. Use the
   [BLE provisioning tool](https://louisremi.github.io/ha-aohi-charger/tools/aohi-ble-provisioning.html)
   (Chrome or Edge) to set:

   | Field | Value |
   |---|---|
   | host1 | `http://<home-assistant-ip>:8099` |
   | host2 | `wss://<home-assistant-ip>:8098/ws/` |

   Both must be **38 characters or fewer**, which a LAN address satisfies comfortably.
4. Give it your WiFi credentials in the same tool. It should appear in the add-on log within a
   minute or two.

## Options

| Option | Default | Notes |
|---|---|---|
| `lan_ip` | *(blank)* | The address chargers use to reach Home Assistant. Blank means detect it automatically, which is right unless the machine has several networks. It is baked into the certificate. |
| `http_port` | `8099` | Bootstrap endpoints (plain HTTP) |
| `mqtt_port` | `8098` | MQTT broker (TLS WebSocket) |
| `weather_city` | `Local` | Shown on the charger's own display |
| `weather_tz` | `UTC` | Time zone reported to the charger |
| `weather_offset` | `0` | UTC offset in seconds, e.g. `7200` for CEST |
| `verbose` | `false` | Log every routed message. Noisy; for diagnosis only. |

The add-on uses host networking, so the ports are those of the Home Assistant machine itself.

## How it works

The charger talks to two endpoints, both settable over Bluetooth:

- **host1** — a REST bootstrap it completes before doing anything else: a device login, broker
  credentials, a time sync and a weather lookup. Plain `http://` is fine.
- **host2** — MQTT over WebSocket, where the actual control happens. This one **must** be `wss://`;
  given a plain `ws://` address the charger silently never connects. The firmware does not verify
  the certificate, so the self-signed one this add-on generates is enough — no domain and no
  certificate authority.

Topics and payloads are unchanged from the cloud, so anything that speaks the AOHI protocol works.
Full protocol notes are
[here](https://github.com/louisremi/ha-aohi-charger/blob/main/tools/PROTOCOL.md).

## Checking it works

From any machine with Node installed:

```bash
node check.js <home-assistant-ip> 8099 8098
```

It reports whether the server is reachable, which chargers have registered, and whether a real
command round-trip succeeds — three failures that look identical from Home Assistant but need
different fixes.

## Credit

The Bluetooth protocol this depends on was reverse engineered by
[atc1441](https://github.com/atc1441/AOHi_280W_Charger_Hacking).

Unofficial and not affiliated with AOHI or I4SEASON.
