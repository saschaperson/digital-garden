---
title: Cloudflare Tunnel & Portainer
publish: true
date: 2026-02-20
tags:
  - homelab
  - cloudflare
  - docker
  - infrastructure
---

# Cloudflare Tunnel & Portainer

> Zurück zum [[Homeserver]]-Überblick.

Zwei Infrastruktur-Services, die den Homeserver von außen erreichbar und von innen verwaltbar machen.

---

## Cloudflare Tunnel

Macht ausgewählte Services über das Internet erreichbar, ohne Port-Forwarding oder eine öffentliche IP zu benötigen. Der Container baut eine ausgehende Verbindung zu Cloudflare auf – kein eingehender Traffic direkt zum Pi.

### Setup

- **Image:** `cloudflare/cloudflared:latest`
- **Command:** `tunnel --no-autoupdate run`
- **Authentifizierung:** Tunnel-Token als Umgebungsvariable
- **Netzwerk:** Standard Docker Bridge

### Welche Services sind exponiert?

<!-- Ergänze hier, welche Services du über den Tunnel erreichbar machst -->

| Service | Subdomain | Notizen |
|---------|-----------|---------|
| | | |

### Warum nicht alles über Tunnel?

Sensible Services wie [[Vaultwarden]] laufen bewusst nur über Tailscale. Der Cloudflare Tunnel ist für Services, die entweder öffentlich sein dürfen oder von Geräten erreichbar sein müssen, auf denen kein Tailscale läuft.

---

## Portainer

Web-basierte Docker-Management-Oberfläche. Praktisch für schnellen Überblick über Container-Status, Logs und Ressourcenverbrauch, ohne SSH-Verbindung.

### Setup

- **Image:** `portainer/portainer-ce:latest`
- **Netzwerk:** `network_mode: host` – Web-UI direkt über Host-IP erreichbar
- **Port:** 9443 (HTTPS)
- **Volumes:**
  - Docker Socket durchgereicht für Container-Management
  - Persistente Daten in `./portainer_data`

### Zugriff

Erreichbar im lokalen Netzwerk und über Tailscale. Passwort beim ersten Aufruf gesetzt.

### Troubleshooting

- **Web-UI nicht erreichbar:** Port 9443 offen? `docker ps` → Container läuft?
- **Container werden nicht angezeigt:** Docker Socket korrekt gemountet? (`/var/run/docker.sock`)
- **Nach Pi-Neustart kein Login:** Portainer hat einen Timeout für die Ersteinrichtung – Container ggf. neu erstellen
