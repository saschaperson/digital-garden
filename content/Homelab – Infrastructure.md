# Homelab – Infrastructure

> CT 101 — Netzwerk-Infrastruktur und Monitoring. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
| --- | --- |
| CT ID | 101 |
| Hostname | `infrastructure` |
| IP | 192.168.178.10 |
| Cores | 2 |
| RAM | 1 GB |
| Swap | 512 MB |
| Disk | 4 GB (local-zfs) |
| DNS | 192.168.178.1 (Fritz!Box — kann nicht auf sich selbst zeigen) |
| Start Order | 1 |

Alle anderen Container nutzen .10 (diesen Container) als DNS. CT 101 selbst nutzt die Fritz!Box, weil Pi-hole nicht auf sich selbst als Upstream zeigen kann.

## Services

| Service | Port | Funktion |
| --- | --- | --- |
| Pi-hole | 80 (Admin) / 53 (DNS) | DNS-basiertes Ad-Blocking für alle Clients |
| Cloudflared | — (Outbound) | Cloudflare Tunnel für externe Services |
| Pulse | 7655 | Proxmox Monitoring (CPU, RAM, Disk, Temps aller LXCs) |
| Portainer | 9443 | Container-Management GUI (gelegentlich genutzt) |

### Pi-hole

Läuft mit `network_mode: host`. In unprivileged LXCs funktioniert Docker-NAT für Port 53 nicht zuverlässig, deshalb Host-Networking. Die Fritz!Box verweist auf .10 als DNS-Server für alle DHCP-Clients.

### Cloudflared

Konfiguriert über das Cloudflare Zero Trust Dashboard, nicht lokal. Der Tunnel-Token liegt in einer `.env`-Datei. Vier Public Hostnames (jellyfin, seerr, betterbahn, home). BetterBahn ist zusätzlich über Cloudflare Access mit E-Mail-OTP geschützt.

### Pulse

Proxmox-Monitoring über die Proxmox API. Verbunden mit `https://192.168.178.3:8006`, SSL-Verify deaktiviert (self-signed Zertifikat). API Token: `pulse@pam!monitoring` mit PVEAuditor-Rolle. Zeigt alle LXCs mit CPU, RAM, Disk, Netzwerk. Temperature Monitoring aktiviert.

### Portainer

Portainer CE als zentrale Verwaltung. Portainer Agents laufen auf CT 102, 103 und 104 (Port 9001). Wird gelegentlich genutzt, wenn ohne KI-Support am Stack gearbeitet wird und das GUI praktisch ist.

## Compose-Struktur

Zwei separate Compose-Files: `/opt/infrastructure/` (Pi-hole, Cloudflared, Portainer) und `/opt/pulse/` (Pulse).

### /opt/infrastructure/docker-compose.yml

```yaml
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    environment:
      - TUNNEL_TOKEN=${CLOUDFLARED_TUNNEL_TOKEN}
    command: tunnel run

  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Europe/Berlin
      - WEBPASSWORD=${PIHOLE_PASSWORD}
      - DNSMASQ_LISTENING=all
    volumes:
      - ./pihole/etc-pihole:/etc/pihole
      - ./pihole/etc-dnsmasq.d:/etc/dnsmasq.d
    dns:
      - 127.0.0.1
      - 1.1.1.1

  portainer:
    image: portainer/portainer-ce:lts
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9443:9443"
      - "8000:8000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

### /opt/pulse/docker-compose.yml

```yaml
services:
  pulse:
    image: rcourtman/pulse:latest
    container_name: pulse
    restart: unless-stopped
    ports:
      - "7655:7655"
    environment:
      PULSE_AUTH_USER: ${PULSE_USER}
      PULSE_AUTH_PASS: ${PULSE_PASS}
    volumes:
      - ./data:/data
```

Secrets in `.env`-Dateien im jeweiligen Verzeichnis.
