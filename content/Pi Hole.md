# Pi Hole

Adlist Updater installieren: https://github.com/jacklul/pihole-updatelists
Im Github Repo empfohlene Adlists etc. in die Konfigutrationsdatei des Updaters hinzufügen. Updater einmal durchlaufen lassen.

Dort empfohlene Ad-lists: [https://v.firebog.net/hosts/lists.php?type=tick](https://v.firebog.net/hosts/lists.php?type=tick)
Dort empfohlene Whitelist: [https://raw.githubusercontent.com/anudeepND/whitelist/master/domains/whitelist.txt](https://raw.githubusercontent.com/anudeepND/whitelist/master/domains/whitelist.txt)
Dort empfohlene Regex Blacklist: [https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list](https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list)

Wenn alles funktioniert hat, sieht man im GUI unter Adlists "Managed by pihole-updatelists"

Zusätzlich unbound als Upstream DNS Server installieren: https://docs.pi-hole.net/guides/dns/unbound/
Pi-Hole als DNS in der Fritzbox hinterlegen, der allen Clients promoted werden soll. Das geht unter "Heimnetz","Netzwerk", "iPv4 Einstellungen"