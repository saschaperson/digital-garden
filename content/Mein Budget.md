---
title: Actual Budget
publish: true
date: 2026-02-20
tags:
  - homelab
  - self-hosting
  - finance
---

# Actual Budget

> Zurück zum [[Homeserver]]-Überblick.

Self-hosted Budgeting-Tool. Offline-first, leichtgewichtig, läuft problemlos auf dem Pi.

---

## Setup

- **Image:** `actualbudget/actual-server:latest`
- **Port:** `5006`
- **Config-Verzeichnis:** `./actual:/data`

---

## Zugriff

Erreichbar über `http://<pi-ip>:5006` im lokalen Netzwerk und über Tailscale.

<!-- Ergänze: Wie nutzt du es? Kategorien, Budget-Strategie, Import-Quellen? -->

---

## Troubleshooting

- **Sync funktioniert nicht:** Container-Logs prüfen, Port erreichbar?
- **Daten weg nach Container-Neustart:** Volume korrekt gemountet? `./actual:/data`
