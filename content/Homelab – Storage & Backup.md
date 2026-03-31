# Homelab – Storage & Backup

> Storage-Architektur und Backup-Strategie. Zurück zum [[Homelab]]-Überblick.

---

## Storage

### MergerFS-Pool

Drei 2,5-Zoll HDDs (je ~4.5 TB) via USB am OptiPlex, zusammengefasst mit MergerFS zu einem logischen Pool unter `/mnt/storage`. USB ist der bekannte Schwachpunkt — kommt langfristig mit dem JMB585 SATA-Controller auf SATA.

| Disk | UUID | Mount |
| --- | --- | --- |
| sdc1 | `f1e84637-fdbd-423b-834e-730ef00c2330` | /mnt/disk1 |
| sdb1 | `a15e7212-8b53-433e-8a1d-bbd595640f68` | /mnt/disk2 |
| sdd1 | `54312f02-42cc-468b-9578-696c0613f051` | /mnt/disk3 |

MergerFS-Policy: `category.create=mfs` (neue Dateien auf die Platte mit dem meisten freien Platz). Reserved Blocks auf 1% reduziert (`tune2fs -m 1`). Nutzbare Kapazität: ~13.4 TB.

### fstab

```
UUID=f1e84637-...  /mnt/disk1  ext4  defaults,nofail,x-systemd.device-timeout=120  0 2
UUID=a15e7212-...  /mnt/disk2  ext4  defaults,nofail,x-systemd.device-timeout=120  0 2
UUID=54312f02-...  /mnt/disk3  ext4  defaults,nofail,x-systemd.device-timeout=120  0 2
/mnt/disk1:/mnt/disk2:/mnt/disk3  /mnt/storage  fuse.mergerfs  defaults,nonempty,x-systemd.requires=mnt-disk1.mount,x-systemd.requires=mnt-disk2.mount,x-systemd.requires=mnt-disk3.mount,x-systemd.after=mnt-disk1.mount,x-systemd.after=mnt-disk2.mount,x-systemd.after=mnt-disk3.mount,allow_other,use_ino,category.create=mfs,fsname=mergerfs,nofail  0 0
/dev/zvol/rpool/swap none swap sw 0 0
```

### MergerFS Boot-Fix

USB-HDDs brauchen ein paar Sekunden zum Spin-up nach dem Boot. Ohne den Fix starten die LXCs bevor MergerFS gemountet ist, und Services wie SABnzbd und Pinchflat sehen ein leeres Verzeichnis. Drei Maßnahmen:

1. **fstab:** `x-systemd.device-timeout=120` gibt den USB-Disks 120 Sekunden zum Auftauchen.
2. **fstab:** MergerFS-Eintrag hat `x-systemd.requires` und `x-systemd.after` für alle drei Disk-Mounts.
3. **systemd Drop-in:** `/etc/systemd/system/pve-guests.service.d/wait-for-storage.conf` sorgt dafür, dass `pve-guests.service` (der die LXCs startet) erst nach `mnt-storage.mount` läuft.

```ini
# /etc/systemd/system/pve-guests.service.d/wait-for-storage.conf
[Unit]
Requires=mnt-storage.mount
After=mnt-storage.mount
```

### Verzeichnisstruktur

```
/mnt/storage/
├── Movies/              Radarr (CT 102, rw)
├── TV Shows/            Sonarr (CT 102, rw)
├── YouTube/             Pinchflat (CT 102, rw)
├── Downloads/
│   ├── Complete/        SABnzbd (CT 102, rw)
│   └── Incomplete/      SABnzbd (CT 102, rw)
├── Photos/              Immich (CT 105, rw)
├── Documents/           Paperless-ngx (CT 106, rw)
└── Backups/
    ├── vzdump/          Proxmox vzdump (Host)
    ├── docker-configs/  Config-Backup-Script (Host)
    └── homeassistant/   HA Samba Backup (Pi)
```

### LXC Bind-Mounts

| CT | Mount auf Host | Mount in LXC | Modus | Host-UID |
| --- | --- | --- | --- | --- |
| 102 (servarr) | `/mnt/storage` | `/mnt/media` | rw | 101000 |
| 103 (media) | `/mnt/storage` | `/mnt/media` | ro | — |
| 104 (services) | `/mnt/storage` | `/mnt/media` | ro | — |
| 105 (photos) | `/mnt/storage/Photos` | `/mnt/photos` | rw | 100000 |
| 106 (documents) | `/mnt/storage/Documents` | `/mnt/documents` | rw | 101000 |

Die Host-UIDs sind unterschiedlich, weil Immich als root im Container läuft (UID 0 → Host 100000) und alle anderen als User 1000 (→ Host 101000). Bei neuen Bind-Mounts immer `chown` auf die richtige Host-UID.

### Geplant: SnapRAID-Parität

JMB585/ASM1166 M.2-SATA-Controller mit einer vierten 5 TB HDD als Parity-Disk. Migration von USB auf SATA ist transparent: UUIDs bleiben gleich, nur der Transport ändert sich.

---

## Backup

### Ebene 1: LXC-Snapshots (vzdump)

Wöchentlich, Sonntag 03:00. Konfiguriert in der Proxmox Web-UI unter Datacenter → Backup.

| Einstellung | Wert |
| --- | --- |
| Storage | `hdd-backup` |
| Schedule | Sonntag 03:00 |
| Selection | Alle CTs (101–106) |
| Compression | ZSTD |
| Mode | Snapshot |
| Retention | Keep Last = 4 |

### Ebene 2: Docker-Config-Backup

Wöchentliches tar der `/opt`-Verzeichnisse aller LXCs. Script: `/root/backup-configs.sh`, Cron: Sonntag 04:00 (`/etc/cron.d/backup-configs`).

```bash
#!/bin/bash
BACKUP_DIR="/mnt/storage/Backups/docker-configs"
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

HA OS internes Backup, wöchentlich, auf Samba-Share am OptiPlex.

| Einstellung | Wert |
| --- | --- |
| Samba-Share | `homeassistant_backup` auf 192.168.178.3 |
| Retention | 3 Backups |
| Encryption | Aktiviert (Emergency Kit sicher aufbewahren!) |
| Backup vor Update | Aktiviert |

### Ebene 4: Immich DB-Backups

Immich erstellt automatisch PostgreSQL-Dumps unter `/mnt/storage/Photos/backups/`. Kein manuelles Setup nötig.

### Ebene 5: Offsite (geplant)

Externe HDD einmal monatlich anstecken, rsync der Backups.

---

## Kritikalitäts-Matrix

| Kategorie | Daten | Backup-Ziel | Kritisch? |
| --- | --- | --- | --- |
| Unersetzbar | HA-Config, Zigbee-Pairing, Automationen | Samba-Share + Offsite | ✅ |
| Unersetzbar | Immich-Fotos, Paperless-Dokumente | HDD-Pool + Config-Backup | ✅ |
| Aufwändig | Servarr-Configs, Pi-hole, Compose-Files, .env-Dateien | vzdump + Config-tar | Mittel |
| Re-downloadbar | Filme, Serien, YouTube | Kein Backup (MergerFS, SnapRAID als Schutz) | Nein |

---

## Host-Tuning

| Einstellung | Wert | Datei |
| --- | --- | --- |
| ZFS ARC Limit | 2 GB | `/etc/modprobe.d/zfs.conf` |
| ZFS Auto-TRIM | on | `zpool set autotrim=on rpool` |
| ZFS Scrub | Monatlich, 1. des Monats, 02:00 | Cron |
| Swap | 4 GB ZFS zvol (`rpool/swap`) | `/etc/fstab` |
| Swappiness | 10 | `/etc/sysctl.d/99-swap.conf` |

Das ZFS ARC wurde von 4 auf 2 GB reduziert, um RAM für die Container freizugeben. Die Boot-SSD ist schnell genug, dass 2 GB ARC keinen spürbaren Unterschied machen. Swap als Sicherheitsnetz, Swappiness auf 10 damit nur bei echtem Druck geswappt wird.
