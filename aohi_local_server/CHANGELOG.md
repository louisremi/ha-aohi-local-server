# Changelog

## 0.1.2

The **BLE provisioning tool now lives here**, at
<https://louisremi.github.io/ha-aohi-local-server/>, and has been reworked around the mistake it
used to invite.

- **Simple mode by default**: enter your Home Assistant machine's address and the endpoints are
  filled in for you. Advanced mode is a checkbox away for custom ports or host3.
- **`ws://` is now refused.** The old tool suggested it in a placeholder, and a charger given a
  plain `ws://` address opens no socket at all — silently, with no error, looking exactly like a
  dead server. host2 must be `wss://`.
- **Writes are confirmed.** The tool waits for the charger's acknowledgement instead of reporting
  success for anything it managed to send.
- Browser guidance up front: Android + Chrome is the reliable path, Brave has Web Bluetooth off by
  default, Firefox and Safari cannot do it at all.
- Removed the endpoint probe (no command ever replies — settled) and the "restore official
  endpoints" button (the factory values cannot be read back, so it was writing a guess; a factory
  reset is the real way back).

Pairing instructions also corrected: a **short press** reaches pairing mode and keeps the charger's
current configuration, and new endpoints only apply after a **power-cycle**.

The add-on also has an icon and logo now, instead of a blank placeholder in the store.

## 0.1.1

Fixes the add-on failing to start with `s6-overlay-suexec: fatal: can only run as pid 1`.

The Home Assistant base image ships s6-overlay v3, whose init insists on being pid 1. Docker's
own init took that slot, so s6 aborted before the server ever ran. Turning it off with
`init: false` is what the base image expects.

Also brings the add-on configuration up to date with current Home Assistant:

- Drops `armv7`, which Home Assistant no longer supports as of 2025.12
- Drops the `map` entry for `/data`, which is always mounted and writable anyway
- Drops `boot`, which was set to its own default

## 0.1.0

First release.

- Serves the four bootstrap endpoints an AOHI charger requires before it will connect to a broker
- Provides an MQTT broker over TLS WebSocket, which is what the charger insists on
- Generates its own self-signed certificate on first start, stored in `/data` so it survives updates
- Detects the host's LAN address automatically when `lan_ip` is left blank
- Exposes `GET /local/devices` for the Home Assistant integration to discover chargers
