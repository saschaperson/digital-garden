---
title: Homelab – Infrastructure
publish: true
date: 2026-04-26
tags:
  - homelab
  - infrastructure
  - docker
---

# Homelab – Infrastructure

> CT 101 — Netzwerk-Infrastruktur und Monitoring. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 101 |
| Hostname | `infrastructure` |
| Cores | 2 |
| RAM | 1 GB |
| Swap | 512 MB |
| Disk | 4 GB (local-zfs) |
| Start Order | 1 |

CT 101 nutzt die Fritz!Box als DNS — Pi-hole kann nicht auf sich selbst als Upstream zeigen. Alle anderen Container nutzen CT 101 als DNS.

---

## Services

| Service | Port | Funktion |
|---------|------|----------|
| Pi-hole | 80 (Admin) / 53 (DNS) | DNS-basiertes Ad-Blocking |
| Cloudflared | — (Outbound) | Cloudflare Tunnel |
| Pulse | 7655 | Proxmox-Monitoring |
| Portainer | 9443 | Container-Management GUI |

---

## Pi-hole

Läuft mit `network_mode: host`. In unprivileged LXCs funktioniert Docker-NAT für Port 53 nicht zuverlässig.

### Einrichtung

Pi-hole-Updatelists für automatische Pflege der Adlists installieren:
- Repo: [github.com/jacklul/pihole-updatelists](https://github.com/jacklul/pihole-updatelists)
- Empfohlene Adlists: [v.firebog.net/hosts/lists.php?type=tick](https://v.firebog.net/hosts/lists.php?type=tick)
- Whitelist: [github.com/anudeepND/whitelist](https://raw.githubusercontent.com/anudeepND/whitelist/master/domains/whitelist.txt)
- Regex Blacklist: [github.com/mmotti/pihole-regex](https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list)

Wenn alles korrekt eingerichtet ist, zeigt das GUI unter Adlists „Managed by pihole-updatelists".

**Upstream DNS:** Unbound als rekursiver Resolver empfohlen — [docs.pi-hole.net/guides/dns/unbound](https://docs.pi-hole.net/guides/dns/unbound/).

### Fritz!Box-Einbindung

Pi-hole als DNS unter Heimnetz → Netzwerk → IPv4-Einstellungen → Lokaler DNS-Server eintragen. Die Fritz!Box verteilt diese Adresse per DHCP an alle Clients.

**Bekannte Einschränkung Fritz!Box 6591 Cable:** Nur ein DNS-Eintrag möglich, kein expliziter Fallback.

---

## Cloudflared

Konfiguriert ausschließlich über das Cloudflare Zero Trust Dashboard, nicht lokal. Der Tunnel-Token liegt in einer `.env`-Datei. Details zur Tunnel-Konfiguration in [[Homelab – Netzwerk & Zugang]].

---

## Pulse

Proxmox-Monitoring über die Proxmox API. SSL-Verify deaktiviert (self-signed Zertifikat). API-Token mit PVEAuditor-Rolle — Monitoring-only, kein Schreibzugriff. Zeigt alle LXCs mit CPU, RAM, Disk, Netzwerk, Temperatur.

---

## Portainer

Portainer CE als Container-Management GUI. Portainer Agents laufen zusätzlich auf CT 102, 103 und 104. Wird gelegentlich genutzt wenn das GUI praktischer ist als SSH.

---

## Compose

Zwei separate Compose-Files: `/opt/infrastructure/` (Pi-hole, Cloudflared, Portainer) und `/opt/pulse/` (Pulse).

```yaml
# /opt/infrastructure/docker-compose.yml
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

```yaml
# /opt/pulse/docker-compose.yml
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
