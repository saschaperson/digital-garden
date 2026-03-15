---
title: Home Assistant
publish: true
date: 2026-03-15
tags:
  - homelab
  - smart-home
  - zigbee
  - home-assistant
---

# Home Assistant

> Zentrale Smart-Home-Plattform auf dem Raspberry Pi 4. Läuft als Home Assistant OS (nicht mehr als Docker-Container). Zurück zum [[Homelab]]-Überblick.

---

## Hardware

| Eigenschaft | Wert |
|-------------|------|
| Gerät | Raspberry Pi 4 (4 GB RAM) |
| IP | 192.168.178.4 |
| OS | Home Assistant OS |
| Boot | NVMe SSD (via USB 2 — bewusst kein USB 3 wegen Zigbee-Interferenz) |
| Zigbee | RaspBee II HAT, angebunden via USB 2 Kabel |
| BLE | ESP32-S3 Proxy via WLAN |

### Boot-Konfiguration

```
# /boot/firmware/config.txt
dtparam=i2c_vc=on,sd_poll_once=on
dtoverlay=miniuart-bt
dtoverlay=i2c-rtc,pcf85063

# /boot/firmware/cmdline.txt
apparmor=1 security=apparmor
```

`dtoverlay=miniuart-bt` verschiebt Bluetooth auf den Mini-UART und gibt den primären UART für den RaspBee II frei. Falls Zigbee-Probleme auftreten: `dtoverlay=disable-bt` als Eskalation.

### Wichtig: USB 2 für Zigbee

USB 3.0 erzeugt Störstrahlung im 2,4-GHz-Band (genau wo Zigbee funkt). Der RaspBee II HAT muss über ein USB-2-Kabel angebunden sein, nicht USB 3. Das gilt auch für die NVMe-SSD — ebenfalls an USB 2.

---

## Zigbee (ZHA)

Integration: ZHA (Zigbee Home Automation), nicht deCONZ.

| Einstellung | Wert |
|-------------|------|
| Device | `/dev/ttyAMA0` |
| Baudrate | 38400 |
| Hardware Flow Control | Aktiviert |

### Zigbee-Geräte

| Gerät | Typ | Raum |
|-------|-----|------|
| Aqara Hub | Bridge | Wohnzimmer |
| *(weitere Geräte hier ergänzen)* | | |

---

## ESP32-S3 BLE Proxy

Der Pi kann Zigbee und Bluetooth nicht gleichzeitig zuverlässig betreiben. Ein ESP32-S3 übernimmt BLE-Sensordaten und leitet sie per WLAN an HA weiter.

| Eigenschaft | Wert |
|-------------|------|
| IP | 192.168.178.32 |
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
| Gießerinnerung | Zeitbasiert + Pflanzensensor |
| Lüftungserinnerung | Luftfeuchtigkeit/CO2 |

---

## Backup

HA-Backups laufen automatisch auf einen Samba-Share am Proxmox-Host (`//192.168.178.3/homeassistant_backup`). Details siehe [[Homelab – Storage & Backup]].

| Einstellung | Wert |
|-------------|------|
| Ziel | Network Storage (Samba) |
| Schedule | Wöchentlich |
| Retention | 3 Backups |
| Encryption | Aktiviert |
| Backup vor Update | Aktiviert |

Emergency Kit (Encryption Key) separat sichern — ohne den Key sind die Backups nutzlos.

---

## CalDAV-Integration

Nextcloud (extern auf Uberspace) liefert per CalDAV einen Geburtstagskalender für das iPad Dashboard.

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

---

## Troubleshooting

- **ZHA startet nicht:** `ls -la /dev/ttyAMA0` prüfen. Ist USB 2 Kabel angeschlossen (nicht USB 3)?
- **Zigbee-Geräte fallen aus:** USB 3 Interferenz? Bluetooth deaktiviert? Abstand zu USB 3 Geräten prüfen
- **ESP32 nicht erreichbar:** WLAN-Signal, OTA-Passwort, ESPHome Dashboard prüfen
- **Backup auf Samba schlägt fehl:** Proxmox-Firewall Port 445 offen? Samba-User/Passwort korrekt? Share-Name `homeassistant_backup` (Unterstrich, kein Bindestrich)
