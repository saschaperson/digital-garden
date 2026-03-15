---
title: Homelab – Shopping List
publish: false
date: 2026-03-15
tags:
  - homelab
  - hardware
---

# Homelab – Shopping List

> ⛔ **Nicht veröffentlichen.** Geplante Hardware-Upgrades. Zurück zum [[Homelab]]-Überblick.

---

## Aktueller Stand

Bereits vorhanden und im Einsatz:
- Dell OptiPlex 3070 Micro (i5-9500T, 16 GB RAM, 500 GB SATA SSD)
- 3× 5 TB 2,5" HDDs (aus externen Gehäusen ausgebaut, via USB als MergerFS-Pool)
- Raspberry Pi 4 (HA OS, NVMe Boot, RaspBee II)
- Bambu A1 3D-Drucker

---

## Nächste Anschaffungen

### 1. M.2 SATA-Controller

Ersetzt die USB-Anbindung der HDDs durch natives SATA. Steckt in den M.2-Slot des OptiPlex (Boot-SSD muss auf den SATA-Slot umziehen).

| Chip | Ports | Interface | Preis |
|------|-------|-----------|-------|
| **ASM1166** | 6× SATA | PCIe 3.0 x2 | 10–18 € |
| JMB585 | 5× SATA | PCIe 3.0 x2 | 10–15 € |

Suchbegriffe: AliExpress `ASM1166 M.2 SATA 6 port` / `JMB585 M.2 NVMe to SATA`

**Achtung:** Boot-SSD muss vom M.2-Slot auf den 2,5" SATA-Slot umziehen. Eine kleine SATA-SSD (128 GB) wird benötigt, oder die bestehende 500 GB SSD umklemmen falls sie 2,5" SATA ist.

### 2. Vierte 5 TB HDD (SnapRAID Parity)

Für SnapRAID-Parität über den MergerFS-Pool. Kann auch eine 2,5" USB-HDD sein die ausgebaut wird.

| Komponente | Preis |
|------------|-------|
| 5 TB 2,5" HDD (z.B. Seagate Expansion, ausbauen) | 80–120 € |

### 3. HDD-Stromversorgung

2,5" HDDs brauchen nur 5V über den SATA-Power-Stecker. Mit SATA-Controller kein USB-Power mehr.

| Komponente | Preis |
|------------|-------|
| 5V/12V SATA-Netzteil | 8–12 € |
| SATA Power Splitter 1→4 | 2–3 € |
| SATA III Datenkabel 30cm (4×) | 3–5 € |

### 4. SATA Boot-SSD (falls nötig)

| Komponente | Preis |
|------------|-------|
| 128 GB SATA 2,5" SSD | 15–18 € |

---

## Parkplatz (später)

| Komponente | Wofür | Preis |
|------------|-------|-------|
| JetKVM + DC Extension | Hardware-KVM-over-IP | 85–95 € |
| Nous A5T Smart Strip | Remote Power per HA | 25–30 € |
| DeskPi RackMate T0 + Zubehör | 10" Rack | 55–85 € |
| Managed Switch (TP-Link TL-SG108E) | VLANs | 25–30 € |
| VLAN-fähiger AP (TP-Link EAP245) | IoT-Isolation | 50–70 € |

---

## Migration USB → SATA (wenn Controller da ist)

1. Boot-SSD auf 2,5" SATA umziehen, M.2-Slot freimachen
2. SATA-Controller in M.2-Slot einsetzen
3. HDDs per SATA-Kabel anschließen (aus USB-Gehäusen bereits ausgebaut)
4. HDDs an SATA-Netzteil + Splitter anschließen
5. fstab: UUIDs bleiben gleich — nur Transport ändert sich, kein Datenverlust
6. MergerFS-Pool funktioniert sofort weiter
7. Vierte HDD anschließen, formatieren, SnapRAID konfigurieren
