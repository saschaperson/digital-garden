---
title: Homelab – Storage & Backup
publish: true
date: 2026-03-15
tags:
  - homelab
  - mergerfs
  - backup
  - storage
---

# Homelab – Storage & Backup

> Storage-Architektur und Backup-Strategie für das Homelab. Zurück zum [[Homelab]]-Überblick.

---

## Storage

### MergerFS-Pool

Drei 2,5-Zoll HDDs (je ~4.5 TB) via USB am OptiPlex, zusammengefasst mit MergerFS zu einem logischen Pool unter `/mnt/storage`.

| Disk | Label | UUID | Mount | Transport |
|------|-------|------|-------|-----------|
| sdc1 | data1 | `f1e84637-fdbd-423b-834e-730ef00c2330` | /mnt/disk1 | USB 3 |
| sdd1 | data2 | `a15e7212-8b53-433e-8a1d-bbd595640f68` | /mnt/disk2 | USB |
| sde1 | data3 | `54312f02-42cc-468b-9578-696c0613f051` | /mnt/disk3 | USB |

MergerFS-Policy: `category.create=mfs` (neue Dateien landen auf der Platte mit dem meisten freien Platz).

Reserved Blocks auf allen drei Disks auf 1% reduziert (`tune2fs -m 1`). Nutzbare Kapazität: ~13.4 TB.

### fstab

```
UUID=f1e84637-fdbd-423b-834e-730ef00c2330  /mnt/disk1  ext4  defaults,nofail  0 2
UUID=a15e7212-8b53-433e-8a1d-bbd595640f68  /mnt/disk2  ext4  defaults,nofail  0 2
UUID=54312f02-42cc-468b-9578-696c0613f051  /mnt/disk3  ext4  defaults,nofail  0 2
/mnt/disk1:/mnt/disk2:/mnt/disk3  /mnt/storage  fuse.mergerfs  defaults,nonempty,allow_other,use_ino,category.create=mfs,fsname=mergerfs,nofail  0 0
```

### Verzeichnisstruktur

```
/mnt/storage/
├── Movies/              ← Radarr (CT 102, rw)
├── TV Shows/            ← Sonarr (CT 102, rw)
├── Downloads/
│   ├── Complete/        ← SABnzbd (CT 102, rw)
│   └── Incomplete/      ← SABnzbd (CT 102, rw)
└── Backups/
    ├── vzdump/          ← Proxmox vzdump (Host)
    ├── docker-configs/  ← Config-Backup-Script (Host)
    └── homeassistant/   ← HA Samba Backup (Pi)
```

Berechtigungen auf dem Host: Movies, TV Shows, Downloads sind `101000:101000` (= UID 1000 in unprivileged LXC). Backups sind `root:root` bzw. `sascha:sascha` (Samba-Share).

### LXC Bind-Mounts

| CT | Mount auf Host | Mount in LXC | Modus |
|----|---------------|--------------|-------|
| 102 (servarr) | `/mnt/storage` | `/mnt/media` | rw |
| 103 (media) | `/mnt/storage` | `/mnt/media` | ro |
| 104 (services) | `/mnt/storage` | `/mnt/media` | ro |

### Geplant: SnapRAID-Parität

Kommt mit dem JMB585/ASM1166 M.2-SATA-Controller und einer vierten 5 TB HDD als Parity-Disk. Migration von USB auf SATA ist transparent: UUIDs bleiben gleich, nur der Transport ändert sich. MergerFS und fstab müssen nicht angepasst werden.

---

## Backup

### Ebene 1: LXC-Snapshots (vzdump)

Wöchentlich, Sonntag 03:00. Konfiguriert in der Proxmox Web-UI unter Datacenter → Backup.

| Einstellung | Wert |
|-------------|------|
| Storage | `hdd-backup` (= `/mnt/storage/Backups/vzdump`) |
| Schedule | Sonntag 03:00 |
| Selection | Alle CTs (101–104) |
| Compression | ZSTD |
| Mode | Snapshot |
| Retention | Keep Last = 4 |

Geschätzte Größe pro Durchlauf: ~5–8 GB. Bei 4 Retention: ~20–32 GB.

### Ebene 2: Docker-Config-Backup

Wöchentliches tar der `/opt`-Verzeichnisse aller LXCs. Script: `/root/backup-configs.sh`, Cron: Sonntag 04:00.

```bash
#!/bin/bash
BACKUP_DIR="/mnt/storage/Backups/docker-configs"
DATE=$(date +%Y%m%d)
mkdir -p "$BACKUP_DIR"

for CT in 101 102 103 104; do
  pct exec $CT -- tar czf /tmp/config-backup.tar.gz -C /opt .
  pct pull $CT /tmp/config-backup.tar.gz "$BACKUP_DIR/ct${CT}-${DATE}.tar.gz"
  pct exec $CT -- rm /tmp/config-backup.tar.gz
done

find "$BACKUP_DIR" -name "*.tar.gz" -mtime +14 -delete
echo "$(date): Config backup completed" >> /var/log/backup-configs.log
```

### Ebene 3: Home Assistant Backup

HA OS internes Backup, wöchentlich, auf Samba-Share am OptiPlex.

| Einstellung | Wert |
|-------------|------|
| Samba-Share | `homeassistant_backup` auf 192.168.178.3 |
| Samba-User | `sascha` |
| Schedule | Custom Days, System Optimal Time |
| Retention | 3 Backups |
| Encryption | Aktiviert — Emergency Kit herunterladen und sicher aufbewahren! |
| Backup vor Update | Aktiviert |

Proxmox-Firewall hat Port 445 für das LAN freigegeben.

### Ebene 4: Offsite (geplant)

Script `/root/offsite-backup.sh` prüft täglich ob eine externe HDD angesteckt ist (per UUID). Falls ja: rsync der Backups, danach unmount. Externe HDD einmal monatlich anstecken.

---

## Kritikalitäts-Matrix

| Kategorie | Daten | Backup-Ziel | Kritisch? |
|-----------|-------|-------------|-----------|
| Unersetzbar | HA-Config, Zigbee-Pairing, Automationen | Samba-Share + Offsite | ✅ |
| Aufwändig | Servarr-Configs, Pi-hole, Cloudflared-Token, Compose-Files | vzdump + Config-tar | Mittel |
| Re-downloadbar | Filme, Serien, Medien | Kein Backup (MergerFS, SnapRAID als Schutz) | Nein |
