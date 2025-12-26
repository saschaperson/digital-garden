Homebridge installieren: https://github.com/homebridge/homebridge/wiki/Install-Homebridge-on-Raspbian

Am besten Node.js auf dem RaspPi direkt updaten, damit man lange Ruhe hat: sudo hb-service update-node

# Raspbee II
Man kann das Plugin Hue nutzen um Zigbee Geräte mit Homebridge zu verknüpfen
Oder z2m für mehr Zigbee Vielfalt: Installationsanleitung hier: https://www.youtube.com/watch?v=BWJmE0ofO8E, wobei das hier die offizielle Anleitung ist.
https://pastebin.com/Mf2HDgn6

# Apple TV On/Off
pyatv installieren. Allerdings sudo pip3 install --upgrade pyatv nutzen
npm install request --save im Terminal von Homebridge 

# Roborock Plugin
Wenn nach dem Token gefragt wird, kann man den mit diesem Tool herausfinden: https://github.com/PiotrMachowski/Xiaomi-cloud-tokens-extractor


