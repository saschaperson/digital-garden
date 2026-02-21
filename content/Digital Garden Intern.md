---
title: Digital Garden – Setup & Wartung
publish: false
date: 2026-02-20
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
   git add content/deine-datei.md
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

## Zettel-Übersicht

### Öffentlich (`publish: true`)

| Zettel | Thema |
|--------|-------|
| [[Digital Garden]] | Meta: Was ist das hier, Tech Stack |
| [[Homeserver]] | Hub: Hardware, Architektur, alle Service-Links, KI-Briefing |
| [[Home Assistant]] | Smart Home, ESP32, Dashboard, Automationen |
| [[AdBlock – Pi-hole & iSponsorBlockTV]] | DNS-Blocker + YouTube Sponsorblock |
| [[Vaultwarden]] | Passwortmanager |
| [[Nextcloud – CalDAV & CardDAV]] | Kontakte & Kalender |
| [[Cloudflare Tunnel & Portainer]] | Infrastruktur |
| [[Docker Compose – Homeserver]] | Compose + .env-Vorlage |
| [[Actual Budget]] | Budgeting |
| [[BamBuddy]] | 3D-Druck-Monitoring |

### Privat (`publish: false`)

| Zettel | Inhalt |
|--------|--------|
| [[Digital Garden – Setup & Wartung]] | Dieser Zettel – Quartz-Config, Deploy, Workflow |
| [[Docker .env – Homeserver]] | Echte Umgebungsvariablen |
| [[Servarr]] | Jellyfin + Arr-Stack |
| [[Nextcloud – Intern]] | Uberspace-Zugangsdaten, DNS, PHP-Config |
| [[Home Assistant – Intern]] | Secrets, MACs, Netzwerk-IPs |
| [[Cloudflare & Tailscale – Intern]] | Tunnel-ID, Tailnet, DNS-Records |

---

## Todo / Ideen

- [ ] Obsidian-Template für Standard-Frontmatter
- [ ] Tagsystem definieren (z.B. `#evergreen`, `#seedling`, `#stub`)
- [ ] Alte Repos (`saschaperson` & `saschafiedler`) durchgehen → relevante .md nach `content` ziehen → alte Pages deaktivieren → Repos archivieren
- [ ] Plausible oder Umami als Analytics testen
