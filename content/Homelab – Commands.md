---
title: Homelab – Commands
publish: true
date: 2026-04-26
tags:
  - homelab
  - runbook
  - commands
---

# Homelab – Commands

> Runbook für den täglichen Betrieb. Alle Befehle laufen auf dem Proxmox-Host als root, sofern nicht anders angegeben. Zurück zum [[Homelab]]-Überblick.

---

## Status & Health

```bash
# LXC-Status auf einen Blick
for ct in 101 102 103 104 105 106; do echo "CT $ct: $(pct status $ct)"; done

# MergerFS gemountet?
mount | grep mergerfs

# Docker-Container pro LXC
for ct in 101 102 103 104 105 106; do
  echo "=== CT $ct ==="
  pct exec $ct -- docker ps --format "{{.Names}}: {{.Status}}"
done

# RAM Host
free -h

# Disk-Belegung Pool
df -h /mnt/storage

# ZFS Pool-Status
zpool status rpool

# SMART (sda und sdb — sdc/sdd nur via CrystalDiskInfo auf Windows)
smartctl -H /dev/sda
smartctl -H /dev/sdb
docker exec scrutiny /opt/scrutiny/bin/scrutiny-collector-metrics run

# Tailscale
tailscale status

# Pi-hole DNS erreichbar?
dig google.com @192.168.178.10 +short

# Logs eines Services
pct exec 103 -- docker logs jellyfin --tail 50
pct exec 103 -- docker logs jellyfin --tail 50 -f   # live follow
```

---

## Updates

```bash
# 1. Proxmox-Host
apt update && apt upgrade -y

# 2. Alle LXCs
for ct in 101 102 103 104 105 106; do
  echo "=== CT $ct ==="
  pct exec $ct -- apt update -qq && pct exec $ct -- apt upgrade -y
done

# 3. Alle Docker Images in allen LXCs aktualisieren
# Findet alle docker-compose.yml unter /opt automatisch — unabhängig von der Ordnerstruktur
for ct in 101 102 103 104 105 106; do
  echo "=== CT $ct ==="
  pct exec $ct -- bash -c "
    find /opt -name 'docker-compose.yml' \
      -not -path '*/recyclarr/resources/*' \
      -exec dirname {} \; | while read dir; do
      echo \"  Updating \$dir\"
      cd \$dir && docker compose pull -q && docker compose up -d --quiet-pull
    done
  "
done

# 4. Danach aufräumen
for ct in 101 102 103 104 105 106; do
  pct exec $ct -- docker system prune -f
done
# Achtung: --volumes nie verwenden — löscht Daten
```

---

## Start / Stop / Restart

```bash
# Einzelner LXC
pct start 103
pct stop 103
pct restart 103

# Alle LXCs stoppen (vor geplantem Reboot, in umgekehrter Reihenfolge)
for ct in 106 105 104 103 102 101; do pct stop $ct; done

# Alle LXCs starten
for ct in 101 102 103 104 105 106; do pct start $ct; done

# Einzelnen Docker-Service neustarten
pct exec 103 -- docker restart jellyfin

# Ganzes Compose neu deployen (nach Config-Änderung)
pct exec 103 -- bash -c "cd /opt/media && docker compose up -d"

# Proxmox-Host geordnet neustarten
for ct in 106 105 104 103 102 101; do pct stop $ct; done
reboot
```

---

## Netzwerk

```bash
# Pi-hole Blocklisten aktualisieren
pct exec 101 -- docker exec pihole pihole -g

# Pi-hole DNS-Cache leeren und Listen neu laden
pct exec 101 -- docker exec pihole pihole reloaddns

# Tailscale neu verbinden
tailscale down && tailscale up \
  --advertise-routes=192.168.178.0/24 \
  --accept-routes \
  --accept-dns=false
```

---

## History löschen

```bash
# Proxmox-Host
history -c && history -w

# Alle LXCs
for ct in 101 102 103 104 105 106; do
  pct exec $ct -- bash -c "history -c && cat /dev/null > ~/.bash_history"
done
```

---

## Notfall-Reihenfolge

Nach ungeplanten Reboots oder wenn nicht alle Services laufen — immer in dieser Reihenfolge:

```bash
# 1. MergerFS gemountet?
mount | grep mergerfs
# Wenn nicht:
systemctl start mnt-storage.mount

# 2. LXC-Status
for ct in 101 102 103 104 105 106; do echo "CT $ct: $(pct status $ct)"; done
# Gestoppte LXCs starten:
pct start 102

# 3. Bind-Mount prüfen falls LXC-Start scheitert
pct config 102 | grep mp
# Falls mp0 fehlt (passiert nach Backup/Restore):
pct set 102 --mp0 /mnt/storage,mp=/mnt/media
pct set 105 --mp0 /mnt/storage/Photos,mp=/mnt/photos
pct set 106 --mp0 /mnt/storage/Documents,mp=/mnt/documents

# 4. Docker-Services prüfen
pct exec 103 -- docker ps -a
# Exited-Container neu starten:
pct exec 103 -- bash -c "cd /opt/media && docker compose up -d"
```

**Typische Ursache wenn CT 102/105/106 nicht starten:** MergerFS nicht gemountet — immer Schritt 1 zuerst.

**Disks manuell aushängen** (z.B. für SMART-Check): Vorher `pct stop 102 105 106`. Nach Remount wieder starten. Nie unmounten während die LXCs laufen.
