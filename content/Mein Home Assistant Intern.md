---
title: Home Assistant – Intern
publish: false
date: 2026-02-20
tags:
  - homelab
  - home-assistant
  - credentials
---

# Home Assistant – Intern

> ⛔ **Nicht veröffentlichen.** Enthält Secrets, MAC-Adressen und interne Netzwerkdetails. Öffentliche Version: [[Home Assistant]].

---

## Zugriff

| Was | Wert |
|-----|------|
| Lokale URL | `http://<pi-ip>:8123` |
| Tailscale URL | <!-- Tailscale-Hostname hier --> |
| Admin-User | <!-- hier eintragen --> |
| Passwort | <!-- hier eintragen --> |

---

## secrets.yaml

```yaml
# /home/sascha/docker/homeserver/homeassistant/secrets.yaml

nextcloud_password: # Nextcloud-Passwort für CalDAV

wifi_ssid: # WLAN-Name für ESPHome
wifi_password: # WLAN-Passwort für ESPHome

esp32_api_key: # ESPHome API Encryption Key
esp32_ota_password: # ESPHome OTA-Passwort

# Weitere Secrets hier ergänzen
```

---

## ESP32 BLE Proxy – Gerätedetails

| Eigenschaft | Wert |
|-------------|------|
| Board | <!-- genaues Board-Modell --> |
| IP (statisch/DHCP) | <!-- IP-Adresse --> |
| MAC | <!-- MAC-Adresse --> |
| Standort | <!-- wo steht er? --> |

### BLE-Sensoren – MAC-Adressen

| Sensor | Typ | MAC | Raum |
|--------|-----|-----|------|
| <!-- z.B. Xiaomi LYWSD03MMC --> | Temp/Humidity | `XX:XX:XX:XX:XX:XX` | <!-- --> |
| <!-- z.B. Mi Flower Care --> | Plant Sensor | `XX:XX:XX:XX:XX:XX` | <!-- --> |

---

## Zigbee-Geräte – Details

| Gerät | Modell | IEEE | Raum | Notizen |
|-------|--------|------|------|---------|
| | | | | |

---

## Netzwerk

| Gerät / Service | IP | MAC | Notizen |
|-----------------|-----|-----|---------|
| Raspberry Pi 4 | | | Statisch oder DHCP-Reservation |
| ESP32 BLE Proxy | | | |
| iPad Dashboard | | | |
| <!-- Router --> | | | |

---

## Integrations-Tokens & API-Keys

<!-- Falls du externe Integrationen nutzt (z.B. Spotify, Wetter-API, etc.) -->

| Integration | Token / Key |
|-------------|-------------|
| | |

---

## Backup-Pfade

```bash
# HA Config-Backup (manuell)
cp -r /home/sascha/docker/homeserver/homeassistant /pfad/zum/backup/

# Oder per HA: Settings → System → Backups
```
