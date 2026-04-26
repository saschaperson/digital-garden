---
title: Homelab – Netzwerk & Zugang
publish: true
date: 2026-04-26
tags:
  - homelab
  - netzwerk
  - tailscale
  - cloudflare
---

# Homelab – Netzwerk & Zugang

> Die Zugangsschicht des Homelab-Ökosystems: lokales Netzwerk via Fritz!Box, externer Zugang über Cloudflare Tunnel und Tailscale. Zurück zum [[Homelab]]-Überblick.

---

## Fritz!Box

Zentrale Netzwerkkomponente. DHCP-Server für das LAN, verweist auf Pi-hole als primären DNS-Server für alle Clients. IPv6 Router Advertisement aktiv, DNSv6 via RA und DHCPv6 deaktiviert.

**Bekannte Einschränkung:** Die Fritz!Box 6591 Cable erlaubt im UI nur einen DNS-Eintrag — kein expliziter Fallback-DNS konfigurierbar. Fällt Pi-hole aus, greift die Fritz!Box auf ihren eigenen Resolver zurück.

### Fritz!Box Recovery (EVA-Tools)

Falls die Fritz!Box in einer Bootloop steckt oder ein fehlgeschlagenes Update recovered werden muss:

```powershell
# eva_tools herunterladen, dann im PowerShell:
./EVA-Discover.ps1                                          # Bootloader abfangen
./EVA-FTP-Client.ps1 -ScriptBlock { SetEnvironmentValue DMC "RTL=y,SL1" }  # RTL auf y
./EVA-FTP-Client.ps1 -ScriptBlock { RebootTheDevice }       # Neustart
```

---

## Tailscale

Läuft auf dem **Proxmox-Host** direkt (nicht in einem LXC) als Subnet Router für das gesamte LAN. Alle LXCs und der Pi 4 sind damit von Tailscale-Geräten aus erreichbar, ohne dass auf jedem Gerät ein Agent laufen muss.

**Warum Subnet Router statt Agent pro Gerät:** Zentrale Konfiguration, ein einziger Punkt für ACLs, kein Overhead pro LXC.

**`--accept-dns=false`:** Der Host verwaltet DNS selbst über Pi-hole. Tailscale MagicDNS wird nicht verwendet.

### MagicDNS — bewusst nicht genutzt

Der Subnet Router macht alle lokalen IPs (192.168.178.x) von überall erreichbar sobald Tailscale aktiv ist — dieselbe URL funktioniert zuhause im LAN und von unterwegs via Tailscale. MagicDNS-Hostnamen (z.B. `proxmox` statt `192.168.178.3`) wären eine komfortablere Schreibweise, fügen aber keine Funktionalität hinzu.

**Einzige Ausnahme:** Wenn das Heimnetz-Subnetz (192.168.178.x) mit dem Netz am aktuellen Standort kollidiert (passiert gelegentlich in Hotels oder bei anderen Heimnetzwerken), versagt der Subnet Router. In diesem Fall sind die Tailscale-IPs aus `[[Homelab – Privat]]` der Fallback.

**Key Expiry deaktiviert:** Für Server-Geräte nötig — sonst alle 180 Tage manuelle Reauthentifizierung.

### Tailscale Serve

Actual Budget benötigt HTTPS wegen SharedArrayBuffer. Da kein öffentlicher Tunnel sinnvoll ist, löst Tailscale Serve das Problem: der Proxmox-Host stellt den Service unter einer Tailscale-HTTPS-URL mit gültigem Zertifikat bereit. Die Serve-Konfiguration überlebt Reboots automatisch.

```bash
tailscale serve --bg --https=5006 http://<ct-ip>:5006
```

---

## Cloudflare Zero Trust

Zwei Ebenen, ein System: **Cloudflare Tunnel** transportiert den Traffic, **Cloudflare Access** sichert ihn ab. Beides wird im Cloudflare Zero Trust Dashboard konfiguriert — kein lokales Config-File.

### Cloudflare Tunnel

Der Cloudflared-Container in CT 101 baut eine ausgehende Verbindung zu Cloudflare auf. Kein Port-Forwarding, keine öffentliche IP nötig. Vier Services sind über Public Hostnames exponiert.

**Warum nicht alles über Tunnel:** Clients wie Infuse (Jellyfin) und die Immich Mobile App können keinen Browser-basierten OAuth-Flow. Diese Services laufen ausschließlich über Tailscale.

### Cloudflare Access

Optionale Schutzschicht vor einzelnen Tunnel-Services. Authentifizierung per E-Mail-OTP — kein eigenes Passwort nötig. Cookie-Domain gilt für alle Access-geschützten Subdomains: einmal einloggen, überall gültig.

**Welche Services Access bekommen:** Nur Services ohne eigene Authentifizierung (z.B. BetterBahn). Services mit eigenem Login (Jellyfin, Seerr, Homarr) laufen ohne Access davor.

### Exponierte Services

| Service | Schutz |
|---------|--------|
| Jellyfin | Eigener Login (kein Access — Infuse kann keinen Browser-Flow) |
| Seerr | Eigener Login |
| Homarr | Eigener Login |
| BetterBahn | Cloudflare Access (E-Mail-OTP) |

Konkrete Subdomains und Ports in `[[Homelab – Privat]]`.

---

## IP-Schema

Feste Reservierungen per DHCP in der Fritz!Box. Konkrete IPs in `[[Homelab – Privat]]`.

| Bereich | Geräte |
|---------|--------|
| Infrastruktur (.1–.9) | Fritz!Box, Powerline, OptiPlex, Pi 4 |
| LXC-Container (.10–.15) | CT 101–106 |
| Feste Geräte (.20–.39) | Apple TV, TV, PlayStation, Roborock, Bambu A1, ESP32, Aqara Hub |
| DHCP-Pool (.50–.254) | Alle anderen |
