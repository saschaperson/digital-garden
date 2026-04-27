---
title: Homelab – Storage & Backup
publish: true
date: 2026-04-26
tags:
  - homelab
  - storage
  - backup
  - mergerfs
---

# Homelab – Storage & Backup

> Storage-Architektur und Backup-Strategie des Proxmox-Hosts. Zurück zum [[Homelab]]-Überblick.

---

## Storage

### MergerFS-Pool

Drei 2,5-Zoll HDDs via USB am OptiPlex, zusammengefasst mit MergerFS zu einem logischen Pool unter `/mnt/storage`. USB ist der bekannte Schwachpunkt — Langfristplan ist ein M.2-SATA-Controller (ASM1166, 6 Ports).

**MergerFS-Policy:** `category.create=mfs` — neue Dateien landen auf der Disk mit dem meisten freien Platz. Reserved Blocks auf 1% reduziert (`tune2fs -m 1`). Nutzbare Kapazität: ~13,5 TB.

### fstab

```
UUID=<disk1-uuid>  /mnt/disk1  ext4  defaults,nofail,x-systemd.device-timeout=120  0 2
UUID=<disk2-uuid>  /mnt/disk2  ext4  defaults,nofail,x-systemd.device-timeout=120  0 2
UUID=<disk3-uuid>  /mnt/disk3  ext4  defaults,nofail,x-systemd.device-timeout=120  0 2
/mnt/disk1:/mnt/disk2:/mnt/disk3  /mnt/storage  fuse.mergerfs  defaults,nonempty,x-systemd.requires=mnt-disk1.mount,x-systemd.requires=mnt-disk2.mount,x-systemd.requires=mnt-disk3.mount,x-systemd.after=mnt-disk1.mount,x-systemd.after=mnt-disk2.mount,x-systemd.after=mnt-disk3.mount,allow_other,use_ino,category.create=mfs,fsname=mergerfs,nofail  0 0
```

Konkrete UUIDs in `[[Homelab – Privat]]`.

### MergerFS Boot-Fix

USB-HDDs brauchen beim Boot ein paar Sekunden zum Spin-up. Ohne Fix starten LXCs bevor MergerFS gemountet ist — Services sehen ein leeres Verzeichnis. Drei Maßnahmen:

1. **fstab:** `x-systemd.device-timeout=120` gibt USB-Disks 120 Sekunden zum Erscheinen
2. **fstab:** MergerFS-Eintrag mit `x-systemd.requires` und `x-systemd.after` für alle drei Disk-Mounts
3. **systemd Drop-in:** `pve-guests.service` wartet auf `mnt-storage.mount` bevor LXCs gestartet werden

```ini
# /etc/systemd/system/pve-guests.service.d/wait-for-storage.conf
[Unit]
Requires=mnt-storage.mount
After=mnt-storage.mount
```

**Wichtig:** Wenn Disks manuell ausgehängt werden (z.B. für SMART-Checks), vorher abhängige LXCs stoppen (`pct stop 102 105 106`). Nach Remount wieder starten.

### Verzeichnisstruktur

```
/mnt/storage/
├── usenet/
│   ├── incomplete/      SABnzbd (CT 102, rw)
│   └── complete/
│       ├── movies/      SABnzbd → Radarr
│       └── tv/          SABnzbd → Sonarr
├── media/
│   ├── movies/          Radarr (CT 102, rw) | Jellyfin (CT 103, ro)
│   ├── tv/              Sonarr (CT 102, rw) | Jellyfin (CT 103, ro)
│   └── youtube/         Pinchflat (CT 102, rw) | Jellyfin (CT 103, ro)
├── photos/              Immich (CT 105, rw)
├── documents/           Paperless-ngx (CT 106, rw)
└── backup/
    ├── vzdump/          Proxmox vzdump (Host)
    ├── docker-configs/  Config-Backup-Script (Host)
    └── homeassistant/   HA Samba Backup (Pi)
```

**TRaSH-konforme Struktur:** Radarr, Sonarr und SABnzbd bekommen alle `/mnt/storage` als `/data` gemountet — ein einziges Dateisystem, Hardlinks und Atomic Moves funktionieren.

### LXC Bind-Mounts

| CT | Mount auf Host | Mount in LXC | Modus | Host-UID |
|----|----------------|--------------|-------|----------|
| 102 (servarr) | `/mnt/storage` | `/mnt/media` | rw | 101000 |
| 103 (media) | `/mnt/storage` | `/data` | ro | — |
| 105 (photos) | `/mnt/storage/photos` | `/mnt/photos` | rw | 100000 |
| 106 (documents) | `/mnt/storage/documents` | `/mnt/documents` | rw | 101000 |

CT 104 hat keinen Storage-Mount — kein Service dort braucht Medienzugriff.

---

## Backup

### Ebene 1: LXC-Snapshots (vzdump)

Wöchentlich, Sonntag 03:00. Konfiguriert in der Proxmox Web-UI unter Datacenter → Backup.

| Einstellung | Wert |
|-------------|------|
| Compression | ZSTD |
| Mode | Snapshot |
| Retention | Keep Last = 4 |

### Ebene 2: Docker-Config-Backup

Wöchentliches tar der `/opt`-Verzeichnisse aller LXCs. Script: `/root/backup-configs.sh`, Cron: Sonntag 04:00.

```bash
#!/bin/bash
BACKUP_DIR="/mnt/storage/backup/docker-configs"
DATE=$(date +%Y%m%d)
mkdir -p "$BACKUP_DIR"

for CT in 101 102 103 104 105 106; do
  pct exec $CT -- tar czf /tmp/config-backup.tar.gz -C /opt .
  pct pull $CT /tmp/config-backup.tar.gz "$BACKUP_DIR/ct${CT}-${DATE}.tar.gz"
  pct exec $CT -- rm /tmp/config-backup.tar.gz
done

find "$BACKUP_DIR" -name "*.tar.gz" -mtime +14 -delete
echo "$(date): Config backup completed" >> /var/log/backup-configs.log
```

### Ebene 3: Home Assistant Backup

HA OS internes Backup, wöchentlich, auf Samba-Share am OptiPlex (`homeassistant_backup`). Retention: 3 Backups. Encryption aktiviert — Emergency Kit (Encryption Key) separat im Passwort-Manager sichern.

### Ebene 4: Immich DB-Backups

Immich erstellt automatisch PostgreSQL-Dumps unter `/mnt/storage/Photos/backups/`.

### Ebene 5: Offsite

Geplant: Hetzner Storage Box + Borgbackup für Immich-Fotos und Paperless-Dokumente (beides unersetzbar). Muss automatisiert sein.

---

## Kritikalitäts-Matrix

| Kategorie | Daten | Kritisch |
|-----------|-------|----------|
| Unersetzbar | HA-Config, Zigbee-Pairing, Automationen | ✅ |
| Unersetzbar | Immich-Fotos, Paperless-Dokumente | ✅ |
| Aufwändig | Servarr-Configs, Pi-hole, Compose-Files | Mittel |
| Re-downloadbar | Filme, Serien, YouTube | Nein |

---

## Drive-Übersicht

| Dev | Modell | Gehäuse | Mount | Pool | SMART-Monitoring |
|-----|--------|---------|-------|------|-----------------|
| sda | Samsung SSD 840 EVO 500GB | intern | — | Boot | Scrutiny ✅ |
| sdb | Seagate ST5000LM000-2U8170 (Expansion SW) | USB 2 | /mnt/disk1 | MergerFS | Manuell ⚠️ |
| sdc | Seagate ST5000LM000-2AN170 (Portable) | USB 3 | /mnt/disk2 | MergerFS | Manuell ⚠️ |
| sdd | WD MyPassport ST5000LM000-2AN170 | USB 3 | /mnt/disk3 | MergerFS | Manuell ⚠️ |
| sde | WD WD50NPJZ (Intenso-Gehäuse) | USB 2 | /mnt/parity | SnapRAID Parity (geplant) | Scrutiny ✅ |

Nach SATA-Migration (ASM1166): alle fünf Drives von Scrutiny vollständig überwacht.

---

## Drive Health Monitoring

### Scrutiny

Läuft direkt auf dem Proxmox-Host (nicht in einem LXC) — notwendig für direkten Block-Device-Zugriff. Compose unter `/opt/scrutiny/`.

**Einschränkung:** Alle USB-Gehäuse (sdb, sdc, sdd, sde) blockieren SMART-Passthrough vollständig — USB-Bridge-Chips verweigern den Zugriff auf Linux. Nur sda (Boot-SSD, intern) wird von Scrutiny überwacht. Alle anderen Drives nur via CrystalDiskInfo auf Windows.

```yaml
# /opt/scrutiny/config/collector.yaml
version: 1
devices:
  - device: /dev/sda
    type: sat
```

```yaml
# /opt/scrutiny/docker-compose.yml
services:
  scrutiny:
    image: ghcr.io/analogj/scrutiny:master-omnibus
    container_name: scrutiny
    restart: unless-stopped
    cap_add:
      - SYS_RAWIO
    privileged: true
    ports:
      - "8080:8080"
    volumes:
      - /run/udev:/run/udev:ro
      - ./config:/opt/scrutiny/config
      - ./influxdb:/opt/scrutiny/influxdb
    devices:
      - /dev/sda
      - /dev/sdb
      - /dev/sdc
      - /dev/sdd
```

Manueller Scan: `docker exec scrutiny /opt/scrutiny/bin/scrutiny-collector-metrics run`

### SMART-Werte die beobachtet werden

Für alle HDDs (sdb, sdc, sdd) gelten diese kritischen Indikatoren:

| ID | Name | Bedeutung | Grenze |
|----|------|-----------|--------|
| 5 | Reallocated Sector Count | Sektoren physisch umgeschrieben weil defekt — direkte Vorwarnung für Ausfall | >0 = sofort handeln |
| 197 | Current Pending Sector Count | Sektoren die auf Verifikation warten — können noch gerettet werden | >0 = beobachten |
| 198 | Offline Uncorrectable Sector Count | Nicht wiederherstellbare Sektoren | >0 = sofort handeln |
| 187 | Reported Uncorrectable Errors | Fehler die ECC nicht korrigieren konnte | >0 = sofort handeln |
| 199 | UltraDMA CRC Error Count | Verbindungsfehler (Kabel, Gehäuse-Bridge) — steigend = Kabel/Gehäuse tauschen | Baseline merken, Anstieg beobachten |

Für die Boot-SSD (sda, Samsung 840 EVO) zusätzlich:

| Attribut | Bedeutung | Grenze |
|----------|-----------|--------|
| Reallocated Sectors | Wie bei HDDs | >0 = beobachten |
| Health % | Gesamtzustand laut Scrutiny | <90% = handeln |

### Manuelle SMART-Kontrolle sdc/sdd

Da Scrutiny diese Drives nicht erreicht: CrystalDiskInfo auf Windows, Disks direkt anschließen. Alle 4–6 Wochen oder nach Auffälligkeiten.

**Besonders beobachten:** sdd (Seagate Expansion SW, /mnt/disk3) — hatte beim ersten Check 16.332 CRC-Fehler. Baseline und Verlauf in `[[Homelab – Privat]]`.

---

## Host-Tuning

| Einstellung | Wert |
|-------------|------|
| ZFS ARC Limit | 2 GB (`/etc/modprobe.d/zfs.conf`) |
| ZFS Auto-TRIM | on |
| ZFS Scrub | Monatlich, 1. des Monats, 02:00 |
| Swap | 4 GB ZFS zvol, swappiness=10 |

ZFS ARC auf 2 GB reduziert um RAM für Container freizugeben. Die Boot-SSD ist schnell genug dass der kleinere Cache keinen spürbaren Unterschied macht.
