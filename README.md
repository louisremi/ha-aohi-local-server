# AOHI Local Server — Home Assistant add-on

Run AOHI smart chargers **entirely on your own network**, with no cloud account and no internet
dependency.

## Install

[![Add this add-on repository to your Home Assistant instance.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Flouisremi%2Fha-aohi-local-server)

The button adds this repository to your add-on store. Then install **AOHI Local Server** and start
it.

If the button does nothing — it relies on [My Home Assistant](https://my.home-assistant.io/),
which has to be enabled on your instance — add it by hand instead:

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

Pointing a charger at this add-on means **reprovisioning it over Bluetooth**, and afterwards it
will no longer work with the official AOHI mobile app. Getting there needs no factory reset — the
provisioning tool reads the charger's current WiFi credentials back out of it — but **going back to
the cloud does**: a factory reset, then re-adding it in the app.

A charger provisioned here is also **committed to this address**. If the add-on is stopped, or the
machine running Home Assistant is off, that charger has nothing to talk to and is simply
unreachable until it comes back. Consider keeping a second charger on the cloud while you decide
whether this suits you.

## Pairing a charger

1. Start the add-on and read its log; it prints the address to use.
2. Put the charger into pairing mode so it advertises over Bluetooth — a short press is enough.
   New endpoints are only committed when the charger completes its WiFi setup, so the tool sends
   your WiFi credentials straight after them; run its **WiFi config** read first and it fills them
   in from the charger itself.
3. Open the **[BLE provisioning tool](https://louisremi.github.io/ha-aohi-local-server/)** and
   enter your Home Assistant machine's address. It fills in the endpoints for you.

   Use **Android + Chrome** if you can — desktop Web Bluetooth is patchier, and Brave has it
   switched off by default. The tool's own page explains this.
4. Give it your WiFi credentials in the same tool.
5. **Power-cycle the charger.** New endpoints do not take effect until it reboots.

It should appear in the add-on log within a minute or two, as
`issued broker credentials to <serial>`.

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
