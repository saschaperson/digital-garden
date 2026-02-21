---
title: Docker .env – Homeserver
publish: false
date: 2026-02-20
tags:
  - homelab
  - docker
---

# Docker .env – Homeserver

> ⛔ **Dieser Zettel wird nicht veröffentlicht.** Enthält echte Credentials.

```bash
# Hier deine echte .env einfügen und pflegen.
# Direkt aus Obsidian rauskopierbar.

TZ=Europe/Berlin
PUID=1000
PGID=1000

# Pi-hole
PIHOLE_FTL_API_PASSWORD=

# Plex
PLEX_CLAIM=

# Cloudflare Tunnel
CLOUDFLARED_TUNNEL_TOKEN=

# Vaultwarden
VW_DOMAIN=
VW_SIGNUPS_ALLOWED=false
VW_ADMIN_TOKEN=

# Servarr (falls in separater Compose)
# ...
```
