---
title: Homelab – Media
publish: true
date: 2026-03-15
tags:
  - homelab
  - jellyfin
  - media-server
---

# Homelab – Media

> CT 103 auf dem Proxmox-Host. Jellyfin als Open-Source-Mediaserver mit Intel iGPU Hardware-Transcoding. Zurück zum [[Homelab]]-Überblick.

---

## Container-Konfiguration

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 103 |
| Hostname | `media` |
| IP | 192.168.178.12 |
| RAM | 2048 MB |
| Disk | 20 GB (ZFS Subvolume) |
| Cores | 2 |
| Mount | `/mnt/storage` → `/mnt/media` (read-only) |

### iGPU-Passthrough

Die Intel UHD 630 (i5-9500T) wird für VA-API Hardware-Transcoding durchgereicht. In der LXC-Config (`/etc/pve/lxc/103.conf`):

```
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.mount.entry: /dev/dri/card0 dev/dri/card0 none bind,optional,create=file
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
```

GID-Mapping für die Render-Gruppe (Host-GID 993):

```
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 993
lxc.idmap: g 993 993 1
lxc.idmap: g 994 100994 64542
```

In der Docker Compose: `group_add: ["993"]` und `devices: ["/dev/dri/renderD128:/dev/dri/renderD128", "/dev/dri/card0:/dev/dri/card0"]`.

---

## Jellyfin

- **Image:** `lscr.io/linuxserver/jellyfin:latest`
- **Port:** 8096
- **Medien:** `/mnt/media` (read-only)
- **Extern:** `https://jellyfin.saschafiedler.com` via Cloudflare Tunnel (CT 101)

### Hardware-Transcoding

| Einstellung | Wert |
|-------------|------|
| Hardware Acceleration | Video Acceleration API (VA-API) |
| VA-API Device | `/dev/dri/renderD128` |
| Driver | Intel iHD (Gen Graphics) |
| Unterstützte Codecs (Decode) | H.264, HEVC, HEVC 10bit, VP8, VP9, VP9 10bit, VC1 |
| Unterstützte Codecs (Encode) | H.264, HEVC |
| AV1 | Nicht unterstützt (erst ab Intel Arc) — Häkchen deaktiviert |
| Low-Power Encoder | Deaktiviert (kein HuC-Firmware konfiguriert) |
| Tone Mapping | VPP + Standard, BT.2390 |
| Throttle Transcodes | Aktiviert, 180s Buffer |
| Encoding Preset | Fast |

### Prüfung

```bash
# VA-API verifizieren:
docker exec jellyfin /usr/lib/jellyfin-ffmpeg/vainfo
# Sollte Intel iHD Driver und Profile auflisten

# Während eines Transcodes:
# Jellyfin Dashboard zeigt "(HW)" neben aktiven Streams
```

---

## Troubleshooting

- **Kein Hardware-Transcoding:** Prüfe ob `/dev/dri/renderD128` in der LXC sichtbar ist (`ls -la /dev/dri/`), ob `group_add: ["993"]` in der Compose steht, und ob VA-API in Jellyfin aktiviert ist
- **Medien fehlen nach Storage-Umbau:** Jellyfin Dashboard → Libraries → Scan All Libraries
- **Buffering bei Remote-Wiedergabe:** Cloudflare Tunnel hat kein Bandbreiten-Limit, aber Transcoding-Qualität in Jellyfin-Client reduzieren falls Upload nicht reicht (250 Mbit/s)
- **Transcode-Cache voll:** Standard-Pfad ist `/config/transcodes` auf dem LXC-Rootfs. Bei Bedarf auf tmpfs oder separates Volume umziehen
