# Mein Homelab

> Oder: Wie ein rosa Plastikregal mit einem gebrauchten PC drin mein digitales Leben verändert hat. Der technische Aufbau steht im [[Homelab]]-Zettel. Hier geht es um das Warum und das Wie-es-dazu-kam.

---

Ich bin mit dem Internet der frühen 2000er aufgewachsen. Deutsche Blogosphäre, netzpolitik.org im RSS-Reader, CCC-Vorträge als Podcast auf dem Schulweg. Die Idee, dass man Dinge selbst hosten kann und vielleicht sogar sollte, war bei mir schon eingepflanzt, bevor ich wusste, was ein Server ist. Nicht als großes ideologisches Statement, sondern als so ein Hintergrundgeräusch: Wenn du es selbst machen kannst, warum gibst du es aus der Hand?

Dann vergehen ein paar Jahre, in denen man das größtenteils ignoriert, weil Gmail halt einfach funktioniert und Spotify 10€ kostet und Netflix da ist. Kennt man ja.

Irgendwann habe ich mir einen Raspberry Pi gekauft. Erst zum Rumspielen, dann für Home Assistant, dann für Pi-hole. Das lief eine Weile zuverlässig, bis ich dachte: Der kann doch bestimmt auch als NAS und Medienserver herhalten. Konnte er nicht. Jedenfalls nicht gleichzeitig, nicht mit 4 GB RAM, und nicht ohne dass plötzlich alles andere auch wackelig wurde. Der Pi ist gut darin, eine Sache stabil zu machen. Drei Sachen gleichzeitig sind nicht seine Stärke.

Also musste was Stärkeres her. Aber ich wohne zur Miete. Ich habe keinen Keller mit Ethernet-Dose und Lüftung. Was auch immer ich baue, muss ins Wohnzimmer passen und darf nicht klingen wie ein startender Fön.

## Das Rabbit Hole

Irgendwann hat mir der YouTube-Algorithmus [dieses Video von Jeff Geerling](https://www.youtube.com/watch?v=y1GCIwLm3is) vorgeschlagen. Ein kleines 10-Zoll-Rack mit Mini-PCs drin, sauber verkabelt, leise, kompakt. Sah aus wie ein Möbelstück, nicht wie ein Serverraum. Ich war sofort angefixt.

Dann habe ich ein paar Monate auf [r/homelab](https://reddit.com/r/homelab) und [r/minilab](https://reddit.com/r/minilab) mitgelesen. Nicht aktiv geplant, eher so ein passives Sammeln. Welche Hardware nutzen Leute? Was kostet das? Welche Services sind sinnvoll und welche sind Nerd-Spielzeug, das man einmal einrichtet und dann nie wieder anfasst? Nach vielleicht drei Monaten hatte ich genug gesehen, habe Alerts auf Kleinanzeigen gestellt und auf das richtige Angebot für einen Mini-PC gewartet.

## Die Hardware

Ich habe mir erst mal angeschaut, was ich schon habe und wie ich das neu zusammenstecken kann, ohne den Kostenrahmen zu sprengen. Vorhanden: der Raspberry Pi, drei 2,5-Zoll-HDDs mit je 5 TB (über USB, ja, ich weiß), ein Bambu A1 3D-Drucker. Fehlte: ein Mini-PC.

Beim Kauf gab es dann den ersten Frustmoment. Ich hatte einen Mini-PC mit 32 GB RAM bestellt. Der Verkäufer hatte sich geirrt, nur 16 GB. Wir haben uns auf einen guten Preis geeinigt, aber in der Zwischenzeit waren andere Angebote mit mehr RAM aufgetaucht, die ich verpasst habe. Am Ende wurde es ein **Dell OptiPlex 3070 Micro** für 100€ auf Kleinanzeigen. Kein Mega-Schnäppchen wenn man vergangene Verkaufspreise anschaut, aber dem aktuellen Markt entsprechend. Intel i5-9500T, 16 GB RAM, 500 GB SSD.

Der Pi läuft weiterhin als dediziertes Smart-Home-Gerät mit Home Assistant. Bei der Gelegenheit habe ich ihn von der SD-Karte auf eine NVMe-SSD umgezogen, über ein [HAT](https://de.aliexpress.com/item/1005007500543515.html). Nicht weil die SD-Karte kaputt war, sondern weil alle immer sagen, dass es nicht die Frage ist ob sie ausfällt, sondern wann. Das HAT hat eine USB-3-Bridge, die dummerweise mit meinem Zigbee-Modul (RaspBee II) interferiert. Die Lösung: ein USB-2.0-Kabel für 2,79€. Willkommen im Homelab.

## Das Rack

Metall wäre natürlich schöner gewesen. Aber ich wusste ja noch gar nicht, ob das alles funktioniert. Ein Metallrack zu kaufen, bevor der erste Container läuft, fühlte sich falsch an. Also drucken.

Ich bin beim [Mod10](https://www.printables.com/model/1225275-modular-10-server-rack-mod10) gelandet, einem modularen 10-Zoll-Rack. Es hatte sich knapp gegen ein [anderes Modell](https://www.printables.com/model/1090551-modular-10-inch-server-rack-reworked) durchgesetzt, das mir zu sehr nach Serverraum aussah. Das Mod10 hat den cleanen Look, den ich wollte. Gedruckt in Bambu PLA Matte Sakura-Pink und Reality CR-PETG Weiß. Ja, pink. Es steht im Wohnzimmer.

Eigentlich fiel das Mod10 fast raus, weil ich mindestens 4,5U Höhe brauchte und das Rack knapp drunter lag. Aber es sah so aus, als könnte man die Crossmembers oben und unten gegen normale Blenden tauschen. Mein Router sollte oben rausschauen für besseres WLAN, also brauchte ich die Höhe. Also habe ich die 6€ für die Druckdaten bezahlt und losgelegt.

Was mir als Rack-Neuling niemand gesagt hat: Nicht alle Löcher in einem Rack haben denselben Abstand. Das ist historisch so gewachsen. In der Praxis hieß das, dass ich Blenden, die ich über die typische 1U-Grenze hinaus befestigen wollte, mehrfach modifizieren und nochmal drucken musste. Etwa 5€ Filament gingen für Revisionen drauf. An einer Stelle hat der erhöhte Druck auf die PLA-Struktur eine schmale Zwischenstrebe rausgebrochen. Sieht man nur von hinten.

Am Ende habe ich jetzt 5,5U und mit dem rausschauenden Router sogar 6U Höhe. Ob ich dabei an Stabilität einbüße, weiß ich ehrlich gesagt nicht. Bisher hält es.

## Das Zubehör

Neben dem Rack kamen noch Kleinteile dazu, die in der Summe erstaunlich viel ausmachen:

| Komponente | Preis |
| --- | --- |
| Dell OptiPlex 3070 Micro | 100,00€ |
| Filament (inkl. Fehldrucke und Revisionen) | ~20,00€ |
| [Steckdosenleiste](https://www.amazon.de/dp/B0DCNP5KZH) | 25,99€ |
| [Keystone RJ45 Adapter (10er Pack)](https://www.amazon.de/dp/B0FKMYD62X) | 8,02€ |
| [Rack Nuts + Schrauben Set](https://www.amazon.de/dp/B0CJNJBSYK) | 10,99€ |
| [M6 Rändelschrauben](https://www.amazon.de/dp/B0F8R5WKCD) | 14,97€ |
| [Kurze Ethernet-Kabel](https://de.aliexpress.com/item/1005009275420639.html) | 7,97€ |
| [NVMe HAT für Pi](https://de.aliexpress.com/item/1005007500543515.html) | 28,21€ |
| Pi-Passivkühler | 6,79€ |
| USB-2.0-Kabel | 2,79€ |
| **Gesamt** | **~225€** |

Die drei HDDs, den Pi und den 3D-Drucker hatte ich schon. Die Rändelschrauben waren technisch nicht nötig, aber ich fand sie schöner, und dass man sie ohne Werkzeug rein- und rausdrehen kann, hat sich als echte Erleichterung erwiesen, wenn man ständig am Rack bastelt.

Die Steckdosenleiste passt nach etwas Bohren ins Rack, hat aber ein Problem: Sie leitet mein Powerline-Signal nicht durch. Der Adapter steckt jetzt direkt in der Wand. Vorher war er in einer anderen Leiste und das Signal war schwach, aber da. Jetzt: nichts. Falls jemand eine schmale Leiste kennt, die Powerline durchlässt, gerne melden.

Die Keystones waren im 10er Pack auf Amazon günstiger als auf AliExpress. Das passiert selten genug, dass ich es erwähnen muss.

## Was darauf läuft

Der OptiPlex läuft mit Proxmox als Hypervisor, darauf sechs LXC-Container mit Docker. Irgendwann während der Planung habe ich mich an Claude gesetzt und angefangen, den Stack im Detail zu recherchieren und aufzubauen. Welche Services in welche Container, wie die Netzwerkstruktur aussehen soll, wie Backups funktionieren.

| Service | Wofür |
| --- | --- |
| Jellyfin | Filme, Serien, YouTube. Hardware-Transcoding über die Intel iGPU. |
| Immich | Fotos. Gesichtserkennung und Smart Search über OpenVINO. |
| Paperless-ngx | Rechnungen, Verträge, Briefe. Scan per iPhone, OCR auf Deutsch und Englisch. |
| Servarr-Stack | Automatische Medienverwaltung. Radarr, Sonarr, Prowlarr, SABnzbd, Seerr. |
| Home Assistant | Smart Home. Zigbee, Automationen, Thermostate. Läuft auf dem Pi. |
| Infrastruktur | Pi-hole, Cloudflare Tunnel, Monitoring. |

Plus Kleinzeug: Actual Budget für Finanzen, Speedtest Tracker, Homarr als Dashboard, Homebox für Haushaltsinventar, BetterBahn für Bahnverbindungen.

Andere Personen im Haushalt und in der Familie haben Zugriff auf den Server. Deshalb wurden manche Entscheidungen bewusst so getroffen, dass es einfach und stabil ist. Jellyfin läuft über Infuse auf dem Apple TV, Immich hat eine iOS-App, und für Seerr gibt es ein Web-Interface, das auch Leute bedienen können, die nicht wissen was Docker ist.

## Was ich anders machen würde

Beim **RAM** würde ich auf 32 GB bestehen. 16 GB reichen für den Start, aber man will immer noch einen Service mehr laufen lassen. Steht auf der Liste.

Beim **Rack** würde ich die Lochpositionen vorher genauer studieren, statt drei Blenden-Revisionen zu drucken.

Die **USB-HDDs** sind der offensichtliche Schwachpunkt. Aber ich hatte sie rumliegen, und in einer Phase, in der jede Komponente teurer wurde, wäre alles andere kein mal-eben-Ausprobieren mehr gewesen. Was die bessere Lösung ist, weiß ich noch nicht. Aber so ist es nicht der Endzustand.

Trotzdem: Ich würde es genauso wieder machen. Mit wenig Geld anfangen, ausprobieren, lernen, dann gezielt investieren. Das Perfekte ist der Feind des Angefangenen.

## Nachbauen?

Der technische Aufbau steht im [[Homelab]]-Zettel und den verlinkten Unter-Zetteln. Die sind so geschrieben, dass jemand (oder ein KI-Assistent) das Setup damit nachbauen könnte.

Wenn dich das Thema interessiert: Ein Raspberry Pi mit Pi-hole ist ein guter erster Schritt. Wenn du merkst, dass du mehr willst, such dir einen gebrauchten Mini-PC auf Kleinanzeigen. Der Rest ergibt sich.

---

## Links

- [Jeff Geerlings Mini Rack Projekt](https://mini-rack.jeffgeerling.com)
- [Mod10 Rack auf Printables](https://www.printables.com/model/1225275-modular-10-server-rack-mod10)
- [r/homelab](https://reddit.com/r/homelab) und [r/minilab](https://reddit.com/r/minilab)
- Mein technischer Stack: [[Homelab]]
