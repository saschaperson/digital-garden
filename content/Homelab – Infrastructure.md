---
title: Homelab – Infrastructure
publish: true
date: 2026-03-15
tags:
  - homelab
  - pi-hole
  - dns
  - cloudflare
---

# Homelab – Infrastructure

> CT 101 auf dem Proxmox-Host. Enthält Pi-hole (DNS-basierter Werbeblocker) und Cloudflared (Cloudflare Tunnel). Zurück zum [[Homelab]]-Überblick.

---

## Container-Konfiguration

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 101 |
| Hostname | `infrastructure` |
| IP | 192.168.178.10 |
| RAM | 512 MB |
| Disk | 4 GB (ZFS Subvolume) |
| Cores | (Proxmox Default) |
| Startup Order | 1 (startet zuerst — DNS muss vor allem anderen laufen) |

### Compose & Secrets

- Compose: `/opt/infrastructure/docker-compose.yml`
- Env: `/opt/infrastructure/.env` (Berechtigungen: `600`)
- Enthält: `CLOUDFLARED_TUNNEL_TOKEN`, `PIHOLE_PASSWORD`

---

## Pi-hole

Netzwerkweiter DNS-Adblocker. Alle Clients im Heimnetz nutzen Pi-hole als DNS-Server.

### Netzwerk-Konfiguration

Pi-hole läuft mit `network_mode: host`. Das ist notwendig, weil Docker-NAT in einer unprivileged LXC Port 53 nicht korrekt weiterleitet.

Die Fritz!Box ist so konfiguriert:
- IPv4 lokaler DNS-Server: `192.168.178.10`
- IPv6: Router Advertisement aktiv, DNSv6 via RA deaktiviert, DHCPv6deaktiviert
- Ergebnis: Alle Clients nutzen IPv4-DNS über Pi-hole, IPv6-Internet funktioniert normal

### Web-Interface

`http://192.168.178.10/admin` (Port 80, nicht 8080). Passwort gesetzt via `docker exec -it pihole pihole -a -p`.

### Blocklisten

Automatisch aktualisiert über `pihole-updatelists` (im Image `jacklul/pihole` integriert):

| Quelle | URL |
|--------|-----|
| Firebog Ticked Lists | `https://v.firebog.net/hosts/lists.php?type=tick` |
| mmotti Regex | `https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list` |

### DNS-Leak-Prüfung

macOS-Geräte können gelegentlich auf die Fritz!Box als IPv6-DNS zurückfallen. Prüfen mit:

```bash
# Auf dem Mac:
scutil --dns | grep "nameserver"
# Sollte NUR 192.168.178.10 zeigen
```

Falls IPv6-DNS der Fritz!Box auftaucht: DNS auf dem Mac manuell auf `192.168.178.10` setzen (Systemeinstellungen → Netzwerk → Wi-Fi → Details → DNS).

---

## Cloudflared

Cloudflare Tunnel für sichere externe Erreichbarkeit ohne Port-Forwarding. Exponiert ausschließlich Jellyfin (CT 103, `http://192.168.178.12:8096`). Konfiguration im Cloudflare Zero Trust Dashboard unter Networks → Tunnels.

Actual Budget und alle Admin-Interfaces sind bewusst nicht im Tunnel — Zugriff von unterwegs läuft ausschließlich über Tailscale.

---

## Troubleshooting

- **DNS-Auflösung funktioniert nicht:** `nslookup google.com 192.168.178.10` von einem Client testen
- **Pi-hole Web-UI nicht erreichbar:** `docker ps` in der LXC prüfen, Port 80 muss auf host-Netzwerk lauschen
- **Cloudflared-Tunnel "unhealthy":** `docker logs cloudflared --tail 20` — häufigste Ursache: Token abgelaufen oder Tunnel im Dashboard deaktiviert
- **Seite fälschlich geblockt:** Pi-hole → Query Log → Whitelist
