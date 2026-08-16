# Changelog

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
