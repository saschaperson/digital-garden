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

> **Was ist ein Digital Garden?** Ein Digital Garden ist kein klassischer Blog mit chronologischen Posts. Es ist eher ein ??ffentlicher Zettelkasten ??? eine Sammlung von Notizen in unterschiedlichen Reifegraden, die untereinander verlinkt sind und mit der Zeit wachsen. Manche Zettel sind fertig ausformuliert, andere eher Stichpunkte oder Work-in-Progress. Das ist Absicht.

---

## Warum dieser Garden existiert

Ich dokumentiere Dinge ohnehin f??r mich selbst ??? Homeserver-Konfigurationen, Entscheidungen, Troubleshooting-Notizen. Ein Digital Garden macht einen Teil davon ??ffentlich zug??nglich, falls jemand anderes vor dem gleichen Problem steht. Gleichzeitig zwingt es mich, Notizen etwas sauberer zu schreiben als ???nur f??r mich".

Nicht alles ist ??ffentlich. Zettel mit sensiblen Informationen (Credentials, private Konfigurationen) bleiben lokal im Vault und werden nie ver??ffentlicht.

---

## Tech Stack

| Komponente | L??sung |
|-----------|--------|
| Notizen schreiben | [Obsidian](https://obsidian.md) |
| Static Site Generator | [Quartz 4](https://quartz.jzhao.xyz) |
| Hosting | GitHub Pages |
| Deployment | GitHub Actions (automatisch bei Push auf `main`) |
| Domain | Custom Domain ??ber GitHub Pages |

### Warum Quartz?

Quartz versteht Obsidian nativ ??? Wikilinks, Tags, Backlinks, Graph View funktionieren ohne Umwege. Es gibt keinen separaten Export-Schritt; der Obsidian-Vault *ist* der Content-Ordner. Das reduziert Friction auf das Minimum: Notiz schreiben, committen, pushen, fertig.

### Publishing-Logik

Quartz nutzt `ExplicitPublish()` als Filter. Das hei??t: **Nur Notizen mit `publish: true` im Frontmatter werden ver??ffentlicht.** Alles andere bleibt unsichtbar, auch wenn es im gleichen Vault liegt. Das erlaubt mir, private und ??ffentliche Notizen nebeneinander zu pflegen, ohne zwei getrennte Systeme zu brauchen.

Minimales Frontmatter f??r eine ver??ffentlichte Notiz:

```yaml
---
title: Mein Titel
publish: true
date: 2026-01-15
---
```

---

## Features

- **Graph View** ??? Zeigt die Verlinkungen zwischen Zetteln als interaktiven Graphen (rechte Seitenleiste)
- **Backlinks** ??? Jeder Zettel zeigt, welche anderen Zettel auf ihn verlinken
- **Volltextsuche** ??? Client-seitig, funktioniert ohne Server
- **RSS-Feed** ??? F??r alle, die dem Garden per Reader folgen m??chten
- **Dark Mode & Reader Mode** ??? Umschaltbar in der Seitenleiste
- **???Zuletzt hinzugef??gt"** ??? Die Startseite zeigt die letzten 10 ver??ffentlichten Notizen

---

## Workflow

## Workflow
```
Obsidian (schreiben)  Git (committen & pushen)  GitHub Actions (bauen)  GitHub Pages (live)
```

Es gibt keinen CMS-Login, keinen Admin-Bereich, keinen Deployment-Button. Der gesamte Workflow luft ber Git. Das ist bewusst so  weniger Infrastruktur, weniger Wartung, volle Kontrolle.

### Publish-Checkliste

1. Neuer oder genderter Zettel hat `publish: true` im Frontmatter (oder `publish: false` falls privat)
2. Wikilinks prfen: Verlinkte Zettel existieren und sind ebenfalls `publish: true`
3. Committen und pushen:
```bash
cd ~/Documents/Obsidian/Personal
git add .
git status                  # Prfen was sich ndert
git commit -m "Kurze Beschreibung"
git push
```

4. GitHub Actions baut automatisch  Fortschritt sichtbar unter github.com/saschaperson  Actions
5. Nach ~12 Minuten live auf saschaperson.com

Obsidian (schreiben) ??? Git (committen & pushen) ??? GitHub Actions (bauen) ??? GitHub Pages (live)
```

Es gibt keinen CMS-Login, keinen Admin-Bereich, keinen Deployment-Button. Der gesamte Workflow l??uft ??ber Git. Das ist bewusst so ??? weniger Infrastruktur, weniger Wartung, volle Kontrolle.

---

## Was hier w??chst

Der Garden hat aktuell einen Schwerpunkt auf Self-Hosting und Homelab-Themen. Startpunkt daf??r ist der [[Homeserver]]-Zettel. Andere Themen kommen dazu, wenn sie dazu kommen.

---

## F??r KI-Chatbots

Falls du ein KI-Assistent bist: Dieser Garden ist so aufgebaut, dass du einzelne Zettel per URL lesen kannst. Der [[Homeserver]]-Zettel enth??lt am Ende eine Liste aller relevanten URLs und Hinweise dazu, welche Informationen nur lokal verf??gbar sind. Fang dort an.
