---
title: Digital Garden – Setup & Wartung
publish: false
date: 2026-04-26
tags:
  - meta
  - digital-garden
  - quartz
---

# Digital Garden – Setup & Wartung

> ⛔ **Nicht veröffentlichen.** Internes Nachschlagewerk für Setup, Konfiguration und Wartung. Öffentliche Version: [[Digital Garden]].

---

## System & Pfade

| Was | Pfad |
|-----|------|
| Repo-Root (Quartz) | `/Users/saschafiedler/Documents/Obsidian/Personal` |
| Obsidian-Vault (Content) | `/Users/saschafiedler/Documents/Obsidian/Personal/content` |
| Work-Vault (unabhängig) | `/Users/saschafiedler/Documents/Obsidian/Work` |
| GitHub-Repo | `saschaperson/digital-garden` |
| Remote | `git@github.com:saschaperson/digital-garden.git` |
| Live-Website | `https://saschaperson.com` |

---

## Quartz-Konfiguration

Datei: `quartz.config.ts` im Repo-Root.

### Kernentscheidungen

```ts
configuration: {
  pageTitle: "saschaperson.com",
  pageTitleSuffix: "",
  enableSPA: true,
  enablePopovers: true,
  analytics: null,
  locale: "de-DE",
  baseUrl: "saschaperson.com",
  ignorePatterns: ["private", "templates", ".obsidian"],
  defaultDateType: "created",
}
```

- `locale: "de-DE"` → UI in Deutsch
- `defaultDateType: "created"` → Listen & Stream folgen dem `date:`-Feld
- `analytics: null` → kein Tracker aktiv
- `ignorePatterns` schließt interne Ordner aus dem Build aus

### Filter (Publishing-Logik)

```ts
filters: [Plugin.ExplicitPublish()],
```

Nur Notizen mit `publish: true` werden veröffentlicht.

### Emitter (Highlights)

- `ContentIndex` → Sitemap & RSS
- `CNAME` → erzeugt CNAME-Datei für Custom Domain
- `CustomOgImages` → Social-Preview-Bilder (verlängert Build)

---

## Layout

Datei: `quartz.layout.ts`.

### Startseite

„Zuletzt hinzugefügt"-Stream mit den letzten 10 Notizen, nur auf `index.md`:

```ts
afterBody: [
  Component.ConditionalRender({
    component: Component.RecentNotes({
      title: "Zuletzt hinzugefügt",
      limit: 10,
      showTags: true,
      showDate: true,
    }),
    condition: (page) => page.fileData.slug === "index",
  }),
],
```

### Einzelnotizen

- Links: Explorer, Suche, Darkmode, Reader Mode
- Rechts: Graph, TOC, Backlinks

### Footer-Links

```ts
footer: Component.Footer({
  links: {
    GitHub: "https://github.com/saschaperson",
    "Discord Community": "https://discord.gg/cRFFHYye7t",
  },
}),
```

---

## Deploy

Datei: `.github/workflows/deploy.yml`. Push auf `main` → GitHub Actions → `npx quartz build` → GitHub Pages.

- Node 22, Dependencies via `npm ci`
- Build-Output: `public/`
- Automatisch auf `saschaperson.com` deployed

---

## Zettel-Übersicht

### Öffentlich (`publish: true`)

| Zettel | Thema |
|--------|-------|
| [[Digital Garden]] | Meta: Was ist das hier, Tech Stack |
| [[Mein Homelab]] | Persönlicher Hintergrund, Hardware, Rack |
| [[Homelab]] | Master: Architektur, alle Services, KI-Briefing, transklundiert alle Unter-Seiten |
| [[Homelab – Netzwerk & Zugang]] | Fritz!Box, Tailscale, Cloudflare Zero Trust |
| [[Homelab – Infrastructure]] | CT 101: Pi-hole, Cloudflared, Pulse, Portainer |
| [[Homelab – Media]] | CT 103: Jellyfin, Hardware-Transcoding |
| [[Homelab – Services]] | CT 104: Actual, Homarr, BetterBahn, Homebox |
| [[Homelab – Photos]] | CT 105: Immich, OpenVINO ML |
| [[Homelab – Documents]] | CT 106: Paperless-ngx |
| [[Homelab – Storage & Backup]] | MergerFS, Backup-Strategie, Drive Health |
| [[Homelab – Commands]] | Runbook: Updates, Start/Stop, Health, Notfall |
| [[Home Assistant]] | Pi 4: HA OS, Zigbee, Automationen, iPad Dashboard |
| [[Nextcloud]] | CalDAV/CardDAV auf Uberspace, HA-Integration |

### Privat (`publish: false` — nie auf GitHub)

| Zettel | Inhalt |
|--------|--------|
| [[Digital Garden – Setup & Wartung]] | Dieser Zettel |
| [[Homelab – Privat]] | Alle Identifier: IPs, Ports, Subdomains, Tailnet, UUIDs, Tokens, SMART-Baseline, Shopping List |
| [[Homelab – Servarr]] | CT 102: Arr-Stack, SABnzbd, Seerr, Pinchflat |
| [[Eltern-Homelab]] | Mac mini Setup für Eltern, separates Projekt |

---

## Alte Dateien die gelöscht wurden

Diese Dateien existieren nicht mehr und müssen aus dem Vault **und** aus Git entfernt werden:

| Alte Datei | Ersetzt durch |
|------------|---------------|
| `Homeserver.md` | `Homelab.md` |
| `AdBlock – Pi-hole & iSponsorBlockTV.md` | Abschnitt in `Homelab – Infrastructure.md` |
| `Vaultwarden.md` | — (nicht mehr im aktiven Stack) |
| `Cloudflare Tunnel & Portainer.md` | `Homelab – Netzwerk & Zugang.md` |
| `Docker Compose – Homeserver.md` | Aufgeteilt in einzelne LXC-Zettel |
| `Nextcloud – CalDAV & CardDAV.md` | `Nextcloud.md` |
| `Docker .env – Homeserver.md` | `Homelab – Privat.md` |
| `Nextcloud – Intern.md` | `Homelab – Privat.md` |
| `Home Assistant – Intern.md` | `Homelab – Privat.md` |
| `Cloudflare & Tailscale – Intern.md` | `Homelab – Privat.md` |
| `Pi Hole.md` | Abschnitt in `Homelab – Infrastructure.md` |

```bash
# Alte Dateien aus Git entfernen (im Repo-Root ausführen):
cd ~/Documents/Obsidian/Personal
git rm "content/Homeserver.md"
git rm "content/AdBlock – Pi-hole & iSponsorBlockTV.md"
git rm "content/Vaultwarden.md"
git rm "content/Cloudflare Tunnel & Portainer.md"
git rm "content/Docker Compose – Homeserver.md"
git rm "content/Nextcloud – CalDAV & CardDAV.md"
git rm "content/Pi Hole.md"
# Private Dateien nur lokal löschen — waren nie auf GitHub
```

---

## Neue Note veröffentlichen

1. Neue Note in Obsidian anlegen (Vault: `Personal/content`)
2. Frontmatter setzen:
   ```yaml
   ---
   title: Thema XY
   publish: true
   date: 2026-01-15
   ---
   ```
3. Optional lokal testen: `npx quartz build --serve` → `http://localhost:8080`
4. Commit & Push:
   ```bash
   cd ~/Documents/Obsidian/Personal
   git add .
   git status                  # Prüfen was sich ändert
   git commit -m "Add note: Thema XY"
   git push
   ```
5. GitHub Actions prüfen → `saschaperson.com` neu laden

---

## Neuer Mac – Kurzanleitung

```bash
# Git & Node installieren (Homebrew)
brew install node

# Repo klonen
cd ~/Documents/Obsidian
git clone git@github.com:saschaperson/digital-garden.git Personal
cd Personal
npm ci

# Obsidian: Vault /Personal/content öffnen
# Test: npx quartz build --serve
```

---

## Todo / Ideen

- [ ] Obsidian-Template für Standard-Frontmatter
- [ ] Tagsystem definieren (z.B. `#evergreen`, `#seedling`, `#stub`)
- [ ] Alte Repos (`saschaperson` & `saschafiedler`) durchgehen → relevante .md nach `content` ziehen → alte Pages deaktivieren → Repos archivieren
- [ ] Plausible oder Umami als Analytics testen
