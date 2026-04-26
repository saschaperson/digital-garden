---
title: Home Assistant
publish: true
date: 2026-04-26
tags:
  - homelab
  - smart-home
  - zigbee
  - home-assistant
---

# Home Assistant

> Zentrale Smart-Home-Plattform auf dem Raspberry Pi 4. Läuft als Home Assistant OS. Zurück zum [[Homelab]]-Überblick.

---

## Hardware

| Eigenschaft | Wert |
|-------------|------|
| Gerät | Raspberry Pi 4 (4 GB RAM) |
| OS | Home Assistant OS |
| Boot | NVMe SSD (via USB 2 — bewusst kein USB 3 wegen Zigbee-Interferenz) |
| Zigbee | RaspBee II HAT, angebunden via USB 2 Kabel |
| BLE | ESP32-S3 Proxy via WLAN |

### Warum USB 2 für Zigbee

USB 3.0 erzeugt Störstrahlung im 2,4-GHz-Band — genau wo Zigbee funkt. RaspBee II muss über ein USB-2-Kabel angebunden sein. Das gilt auch für die NVMe-SSD. Kostete 2,79€ und löste stundenlange Stabilitätsprobleme.

### Boot-Konfiguration

```
# /boot/firmware/config.txt
dtparam=i2c_vc=on,sd_poll_once=on
dtoverlay=miniuart-bt
dtoverlay=i2c-rtc,pcf85063
```

`dtoverlay=miniuart-bt` verschiebt Bluetooth auf den Mini-UART und gibt den primären UART für den RaspBee II frei. Falls Zigbee-Probleme auftreten: `dtoverlay=disable-bt` als Eskalation.

---

## Zigbee (ZHA)

| Einstellung | Wert |
|-------------|------|
| Integration | ZHA (nicht deCONZ) |
| Device | `/dev/ttyAMA0` |
| Baudrate | 38400 |
| Hardware Flow Control | Aktiviert |

---

## ESP32-S3 BLE Proxy

Der Pi kann Zigbee und Bluetooth nicht gleichzeitig zuverlässig betreiben. Ein ESP32-S3 übernimmt BLE-Sensordaten (Miflora-Pflanzensensoren) und leitet sie per WLAN an HA weiter.

| Eigenschaft | Wert |
|-------------|------|
| Framework | ESPHome (ESP-IDF) |
| Modus | `bluetooth_proxy: active: true` |

---

## iPad Dashboard

An der Wand montiertes iPad mini 5 als zentrale Steuerung. Drei Views: Steuerung, Pflanzen, Analyse.

### Scripts

| Script | Funktion |
|--------|----------|
| Bin da | Ankunft zuhause |
| Bin weg | Abwesenheit |
| Gute Nacht | Nacht-Modus |
| Gaming | Gaming-Beleuchtung |
| Fernsehen | TV-Beleuchtung |

### Automationen

| Automation | Trigger |
|------------|---------|
| Gießerinnerung | Zeitbasiert + Miflora-Sensor |
| Lüftungserinnerung | Luftfeuchtigkeit/CO2, temperaturabhängige Verzögerung via OpenWeatherMap |
| Badezimmer-Licht | ZHA IKEA Bilresa |

---

## CalDAV-Integration (Nextcloud)

Nextcloud liefert per CalDAV einen Geburtstagskalender für das iPad Dashboard. Details zur Nextcloud-Instanz in [[Nextcloud]].

```yaml
# In configuration.yaml
calendar:
  - platform: caldav
    url: https://cloud.<deine-domain>/remote.php/dav
    username: <user>
    password: !secret nextcloud_password
    calendars:
      - 'Contact birthdays'
```

---

## Backup

HA-Backups laufen automatisch auf einen Samba-Share am Proxmox-Host. Details in [[Homelab – Storage & Backup]].

| Einstellung | Wert |
|-------------|------|
| Schedule | Wöchentlich |
| Retention | 3 Backups |
| Encryption | Aktiviert |

**Wichtig:** Encryption Key separat im Passwort-Manager sichern — ohne ihn sind alle Backups nutzlos.

---

## Troubleshooting

- **ZHA startet nicht:** `ls -la /dev/ttyAMA0` prüfen. USB-2-Kabel angeschlossen?
- **Zigbee-Geräte fallen aus:** USB-3-Interferenz? Bluetooth deaktiviert? Abstand zu USB-3-Geräten prüfen
- **ESP32 nicht erreichbar:** WLAN-Signal, OTA-Passwort, ESPHome Dashboard prüfen
- **Backup auf Samba schlägt fehl:** Proxmox-Firewall Port 445 offen? Share-Name `homeassistant_backup` (Unterstrich, kein Bindestrich)
