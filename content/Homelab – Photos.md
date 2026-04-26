---
title: Homelab – Photos
publish: true
date: 2026-04-26
tags:
  - homelab
  - immich
  - photos
---

# Homelab – Photos

> CT 105 — Immich als Google-Photos-Alternative. Gesichtserkennung und Smart Search über OpenVINO auf der iGPU. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 105 |
| Hostname | `photos` |
| Cores | 4 |
| RAM | 4 GB |
| Swap | 1 GB |
| Disk | 16 GB (local-zfs) |
| iGPU | ✅ `dev0: /dev/dri/renderD128,mode=0666` |
| Bind-Mount | `/mnt/storage/Photos` → `/mnt/photos` (rw) |
| Start Order | 5 |

4 Cores und 4 GB RAM weil Immich bei ML-Jobs (Gesichtserkennung, Smart Search, OCR) deutlich mehr zieht als andere Services.

---

## Services

| Service | Funktion |
|---------|----------|
| immich-server | API, Web-UI, Video-Transcoding (VAAPI) |
| immich-machine-learning | Gesichtserkennung, CLIP, OCR (OpenVINO auf iGPU) |
| immich-postgres | PostgreSQL mit VectorChord |
| immich-redis | Valkey — Cache und Job Queue |

### Hardware-Beschleunigung

**Video-Transcoding:** VAAPI. In Immich Admin → Video Transcoding → Hardware Acceleration = VAAPI, Hardware Decoding aktiviert.

**Machine Learning:** OpenVINO-Image (`release-openvino`). Nutzt die iGPU für Gesichtserkennung (buffalo_l), Smart Search (ViT-B-32) und OCR (PP-OCRv5). Die iGPU wird mit CT 103 (Jellyfin) geteilt.

Verifizieren: `OpenVINOExecutionProvider` muss vor `CPUExecutionProvider` in den Logs erscheinen.

---

## Storage

```
CT 105 Root-Disk (16 GB, ZFS SSD)
├── /opt/immich/              Compose, .env, hwaccel-Files
└── /opt/immich/postgres/     PostgreSQL-Daten (MUSS auf SSD!)

/mnt/photos (Bind-Mount → /mnt/storage/Photos)
├── library/                  Hochgeladene Originale
├── thumbs/                   Generierte Thumbnails
├── encoded-video/            Transkodierte Videos
├── upload/                   Temporärer Upload
└── backups/                  Automatische DB-Dumps
```

**Wichtig:** PostgreSQL muss auf der SSD liegen — auf HDDs bricht die DB-Performance ein.

**Berechtigungen:** Immich läuft als root im Container (UID 0 = Host-UID 100000). Bind-Mount-Ordner auf dem Host entsprechend `chown`.

---

## Externer Zugang

Kein Cloudflare Tunnel — die Immich Mobile App funktioniert nicht hinter Cloudflare Access. Zugang von unterwegs über Tailscale.

---

## Compose

Basis ist die offizielle Immich-Compose. Anpassungen: VAAPI Transcoding + OpenVINO ML aktiviert über die mitgelieferten `hwaccel.transcoding.yml` und `hwaccel.ml.yml`.

```yaml
name: immich
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    extends:
      file: hwaccel.transcoding.yml
      service: vaapi
    volumes:
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    ports:
      - '2283:2283'
    depends_on:
      - redis
      - database
    restart: always

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-openvino
    extends:
      file: hwaccel.ml.yml
      service: openvino
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: always

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:9
    restart: always

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: always

volumes:
  model-cache:
```

---

## Tipps

- **Große Imports:** ML-Concurrency in Admin → Job Queues auf 1 lassen bei 4 GB RAM. Nachts laufen lassen.
- **Backup:** Immich erstellt automatisch DB-Dumps unter `/mnt/photos/backups/`.
- **Updates:** Vor Updates immer Release Notes lesen — Breaking Changes kommen vor.
