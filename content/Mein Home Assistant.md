---
title: Home Assistant
publish: true
date: 2026-02-20
tags:
  - homelab
  - smart-home
  - zigbee
  - home-assistant
---

# Home Assistant

> Zurück zum [[Homeserver]]-Überblick.

Home Assistant ist die zentrale Smart-Home-Plattform auf dem Raspberry Pi 4. Läuft als Docker-Container mit `network_mode: host` für mDNS/UPnP Discovery und direktem Zugriff auf den RaspBee II Zigbee-Stick.

---

## Setup

- **Image:** `ghcr.io/home-assistant/home-assistant:stable`
- **Netzwerk:** `network_mode: host`
- **Config-Verzeichnis:** `./homeassistant:/config`
- **Zigbee-Stick:** RaspBee II über `/dev/ttyAMA0` durchgereicht
- **Zigbee-Integration:** ZHA (Zigbee Home Automation)

### Zigbee-Geräte

<!-- Liste deiner Zigbee-Geräte hier ergänzen -->

| Gerät | Typ | Raum | Notizen |
|-------|-----|------|---------|
| | | | |

---

## ESP32-S3 BLE Proxy

Der Raspberry Pi kann Zigbee (RaspBee II) und Bluetooth nicht gleichzeitig zuverlässig betreiben. Lösung: Ein ESP32-S3 übernimmt BLE und leitet die Daten per WLAN an Home Assistant weiter.

### Hardware

- **Board:** ESP32-S3 DevKit (z. B. ESP32-S3-WROOM-1)
- **Stromversorgung:** USB-C, beliebiges Netzteil
- **Standort:** <!-- Wo steht der ESP32? -->

### ESPHome-Konfiguration

```yaml
# ESP32 BLE Proxy – ESPHome Config
# Anpassen und in ESPHome Dashboard flashen

esphome:
  name: esp32-ble-proxy
  friendly_name: "BLE Proxy"

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
  encryption:
    key: !secret api_encryption_key

ota:
  - platform: esphome
    password: !secret ota_password

logger:

bluetooth_proxy:
  active: true

# Optional: ESP32 als Sensor für Temperatur/Luftfeuchtigkeit
# sensor:
#   - platform: xiaomi_hhccjcy01
#     mac_address: "XX:XX:XX:XX:XX:XX"
#     ...
```

### BLE-Sensoren

<!-- Liste der BLE-Sensoren hier ergänzen -->

| Sensor | Typ | MAC | Raum |
|--------|-----|-----|------|
| | | | |

### Troubleshooting ESP32

- **ESP32 nicht erreichbar:** WLAN-Signalstärke prüfen, OTA-Passwort korrekt?
- **BLE-Geräte nicht erkannt:** `bluetooth_proxy: active: true` gesetzt? ESP32 in Reichweite?
- **Flash schlägt fehl:** USB-Kabel mit Datenleitung verwenden (nicht nur Ladekabel)

---

## iPad Dashboard

Ein an der Wand montiertes iPad dient als zentrale Steuerung für die Wohnung. Home Assistant wird im Kiosk-Modus über den Browser angezeigt.

### Dashboard-Konfiguration

<!-- Füge hier dein Dashboard-YAML ein. Beispielstruktur: -->

```yaml
# iPad Dashboard – Übersicht
# Pfad: /config/dashboards/ipad/overview.yaml

views:
  - title: Home
    path: home
    type: sections
    cards:
      # Hier deine Cards ergänzen
      []
```

### Dashboard-Tipps

- **Kiosk-Modus:** Fully Kiosk Browser (Android) oder die native HA Companion App mit Dashboard-Pinning
- **Bildschirm-Steuerung:** Bewegungsmelder oder Anwesenheitserkennung, um das Display nur bei Bedarf einzuschalten
- **Design:** Themes über HACS verfügbar, z. B. Mushroom Cards für ein aufgeräumtes UI

---

## Automationen & Scripts

Home Assistant Actions (ehemals „Services") und Automationen, die im Alltag laufen.

### Action 1: <!-- Name ergänzen -->

```yaml
# Beschreibung: 
# Trigger: 
# Aktion: 

automation:
  - alias: ""
    trigger: []
    condition: []
    action: []
```

### Action 2: <!-- Name ergänzen -->

```yaml
automation:
  - alias: ""
    trigger: []
    condition: []
    action: []
```

### Action 3: <!-- Name ergänzen -->

```yaml
automation:
  - alias: ""
    trigger: []
    condition: []
    action: []
```

### Action 4: <!-- Name ergänzen -->

```yaml
automation:
  - alias: ""
    trigger: []
    condition: []
    action: []
```

### Action 5: <!-- Name ergänzen -->

```yaml
automation:
  - alias: ""
    trigger: []
    condition: []
    action: []
```

### Action 6: <!-- Name ergänzen -->

```yaml
automation:
  - alias: ""
    trigger: []
    condition: []
    action: []
```

---

## CalDAV-Integration (Geburtstagskalender)

Nextcloud liefert per CalDAV einen automatischen Geburtstagskalender, der auf dem iPad Dashboard als Calendar Card angezeigt wird.

```yaml
# In configuration.yaml
calendar:
  - platform: caldav
    url: https://cloud.saschafiedler.com/remote.php/dav
    username: sascha
    password: !secret nextcloud_password
    calendars:
      - 'Contact birthdays'
```

Mehr dazu im [[Nextcloud – CalDAV & CardDAV]]-Zettel.

---

## Troubleshooting

- **ZHA startet nicht:** `ls -la /dev/ttyAMA0` prüfen, ggf. Bluetooth in `/boot/config.txt` deaktivieren (`dtoverlay=disable-bt`)
- **Container startet nicht:** `docker logs -n 50 homeassistant`
- **Automation feuert nicht:** Developer Tools → Services → manuell testen
- **Dashboard lädt langsam:** Browser-Cache leeren, Companion App Cache löschen
