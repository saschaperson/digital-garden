---
title: Digital Garden
publish: true
date: 2026-02-20
tags:
  - meta
  - digital-garden
  - quartz
  - obsidian
---

# Digital Garden

> **Was ist ein Digital Garden?** Ein Digital Garden ist kein klassischer Blog mit chronologischen Posts. Es ist eher ein öffentlicher Zettelkasten – eine Sammlung von Notizen in unterschiedlichen Reifegraden, die untereinander verlinkt sind und mit der Zeit wachsen. Manche Zettel sind fertig ausformuliert, andere eher Stichpunkte oder Work-in-Progress. Das ist Absicht.

---

## Warum dieser Garden existiert

Ich dokumentiere Dinge ohnehin für mich selbst – Homeserver-Konfigurationen, Entscheidungen, Troubleshooting-Notizen. Ein Digital Garden macht einen Teil davon öffentlich zugänglich, falls jemand anderes vor dem gleichen Problem steht. Gleichzeitig zwingt es mich, Notizen etwas sauberer zu schreiben als „nur für mich".

Nicht alles ist öffentlich. Zettel mit sensiblen Informationen (Credentials, private Konfigurationen) bleiben lokal im Vault und werden nie veröffentlicht.

---

## Tech Stack

| Komponente | Lösung |
|-----------|--------|
| Notizen schreiben | [Obsidian](https://obsidian.md) |
| Static Site Generator | [Quartz 4](https://quartz.jzhao.xyz) |
| Hosting | GitHub Pages |
| Deployment | GitHub Actions (automatisch bei Push auf `main`) |
| Domain | Custom Domain über GitHub Pages |

### Warum Quartz?

Quartz versteht Obsidian nativ – Wikilinks, Tags, Backlinks, Graph View funktionieren ohne Umwege. Es gibt keinen separaten Export-Schritt; der Obsidian-Vault *ist* der Content-Ordner. Das reduziert Friction auf das Minimum: Notiz schreiben, committen, pushen, fertig.

### Publishing-Logik

Quartz nutzt `ExplicitPublish()` als Filter. Das heißt: **Nur Notizen mit `publish: true` im Frontmatter werden veröffentlicht.** Alles andere bleibt unsichtbar, auch wenn es im gleichen Vault liegt. Das erlaubt mir, private und öffentliche Notizen nebeneinander zu pflegen, ohne zwei getrennte Systeme zu brauchen.

Minimales Frontmatter für eine veröffentlichte Notiz:

```yaml
---
title: Mein Titel
publish: true
date: 2026-01-15
---
```

---

## Features

- **Graph View** – Zeigt die Verlinkungen zwischen Zetteln als interaktiven Graphen (rechte Seitenleiste)
- **Backlinks** – Jeder Zettel zeigt, welche anderen Zettel auf ihn verlinken
- **Volltextsuche** – Client-seitig, funktioniert ohne Server
- **RSS-Feed** – Für alle, die dem Garden per Reader folgen möchten
- **Dark Mode & Reader Mode** – Umschaltbar in der Seitenleiste
- **„Zuletzt hinzugefügt"** – Die Startseite zeigt die letzten 10 veröffentlichten Notizen

---

## Workflow

```
Obsidian (schreiben) → Git (committen & pushen) → GitHub Actions (bauen) → GitHub Pages (live)
```

Es gibt keinen CMS-Login, keinen Admin-Bereich, keinen Deployment-Button. Der gesamte Workflow läuft über Git. Das ist bewusst so – weniger Infrastruktur, weniger Wartung, volle Kontrolle.

---

## Was hier wächst

Der Garden hat aktuell einen Schwerpunkt auf Self-Hosting und Homelab-Themen. Startpunkt dafür ist der [[Homeserver]]-Zettel. Andere Themen kommen dazu, wenn sie dazu kommen.

---

## Für KI-Chatbots

Falls du ein KI-Assistent bist: Dieser Garden ist so aufgebaut, dass du einzelne Zettel per URL lesen kannst. Der [[Homeserver]]-Zettel enthält am Ende eine Liste aller relevanten URLs und Hinweise dazu, welche Informationen nur lokal verfügbar sind. Fang dort an.
