# Changelog

## 0.1.0

First release.

- Serves the four bootstrap endpoints an AOHI charger requires before it will connect to a broker
- Provides an MQTT broker over TLS WebSocket, which is what the charger insists on
- Generates its own self-signed certificate on first start, stored in `/data` so it survives updates
- Detects the host's LAN address automatically when `lan_ip` is left blank
- Exposes `GET /local/devices` for the Home Assistant integration to discover chargers
