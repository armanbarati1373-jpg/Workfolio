# 📘 Gentoo LiveCD – Netzwerk und SSH mit MobaXterm

[← Workfolio](../../../README.md)


> **Ziel:** Eine neue Gentoo-LiveCD-VM in Proxmox ins Netzwerk bringen und anschließend als `root` über MobaXterm/SSH erreichen.

## Zugehöriges Projekt
[🐧 Gentoo Linux auf Proxmox](../README.md)
## Benötigte Werte
- Statische IP: `192.168.50.156`
- Präfix: `/24`
- Subnetzmaske: `255.255.255.0`
- Gateway: `192.168.50.250`
- Schnittstelle: `ens18`
- SSH-Port: `22`

> Passwörter niemals in dieser Dokumentation oder auf Screenshots speichern. Vor einer Wiederholung prüfen, ob `192.168.50.156` weiterhin für diese VM reserviert ist.

## 1. VM und LiveCD starten
1. In Proxmox die neue VM öffnen.
2. Prüfen, dass das Network Device mit der richtigen externen Bridge verbunden und **Link down** nicht aktiviert ist.
3. Die Gentoo Minimal Installation CD starten.
4. Warten, bis `livecd ~ #` erscheint. Die LiveCD meldet `root` automatisch an.
## 2. Netzwerk prüfen
```bash
ip -br a
ip route
```
`ens18` muss `UP` sein. Eine korrekte IPv4-Adresse und Default-Route müssen sichtbar sein. Falls nur `169.254.x.x/16` erscheint, ist DHCP fehlgeschlagen. Diese Link-Local-Adresse nicht für MobaXterm verwenden.
## 3. Statische IP temporär setzen
```bash
ip addr flush dev ens18
ip addr add 192.168.50.156/24 dev ens18
ip link set ens18 up
ip route replace default via 192.168.50.250 dev ens18
```
Diese Einstellung gilt nur in der aktuellen LiveCD-Sitzung und muss nach einem Neustart erneut gesetzt werden.
## 4. Netzwerk testen
```bash
ip -br a
ip route
ping -c 3 192.168.50.250
ping -c 3 1.1.1.1
```
Erwartung:
- `ens18 UP 192.168.50.156/24`
- `default via 192.168.50.250 dev ens18`
- Gateway und Internet antworten.
Bei `Destination Host Unreachable` IP, Präfix, Gateway und Proxmox-Bridge/VLAN prüfen.
## 5. Root-Passwort setzen
```bash
passwd root
```
1. Neues Passwort eingeben und Enter drücken.
2. Dasselbe Passwort erneut eingeben.
3. Es werden keine Zeichen oder Sterne angezeigt; das ist normal.
## 6. SSH starten
```bash
rc-service sshd start
rc-service sshd status
```
Erwartung: `status: started`
Optional Port 22 prüfen:
```bash
ss -lntp | grep ':22'
```
## 7. MobaXterm verbinden
1. **Session** → **SSH** öffnen.
2. **Remote host:** `192.168.50.156`
3. **Specify username:** `root`
4. **Port:** `22`
5. Verbindung öffnen, den ersten Host-Key bestätigen und das gesetzte Passwort eingeben.
## 8. Erfolg prüfen
```bash
hostname
whoami
ip -br a
```
`whoami` muss `root` und `ens18` die Adresse `192.168.50.156/24` anzeigen. Die Proxmox-Konsole offenlassen, bis SSH sicher funktioniert.
## Häufige Fehler
### `Arman: command not found`
`Arman` wurde als Befehl eingegeben. Die LiveCD ist bereits als `root` angemeldet.
### `Network is unreachable`
Eine nutzbare IP oder Default-Route fehlt. Schritte 2 bis 4 wiederholen.
### `169.254.x.x`
DHCP hat keinen Lease geliefert. Schritt 3 verwenden.
### `sshd` zeigt `stopped`
```bash
rc-service sshd start
```
Danach den Status erneut prüfen.
### MobaXterm verbindet nicht
- Von Windows die IP `192.168.50.156` anpingen.
- Benutzer `root` und Port `22` prüfen.
- `sshd`-Status prüfen.
- Proxmox-Bridge, VLAN, Link und Firewall prüfen.
## Kurzfassung
```bash
ip addr flush dev ens18
ip addr add 192.168.50.156/24 dev ens18
ip link set ens18 up
ip route replace default via 192.168.50.250 dev ens18
ping -c 3 1.1.1.1
passwd root
rc-service sshd start
rc-service sshd status
```
Danach MobaXterm mit `root@192.168.50.156` auf Port `22` öffnen.
## Stand
- Erstellt: 27.08.2026
- Meilenstein: Gentoo LiveCD, Netzwerk und SSH
## 9. Vor der Partitionierung: System nur lesen
Noch nichts partitionieren oder formatieren. Zuerst ausführen:
```bash
whoami
hostname
ip -br a
ip route
test -d /sys/firmware/efi && echo UEFI || echo BIOS
lsblk -e7 -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,MODEL
fdisk -l
free -h
cat /etc/resolv.conf
```
Die Ausgabe bestimmt Bootmodus, korrekten Zieldatenträger und Partitionsplan. Erst nach Kontrolle folgt die Partitionierung.
**Ergebnis der Prüfung:** BIOS/Legacy-Boot; Zielplatte `/dev/sda`, 32 GiB, QEMU HARDDISK, ohne sichtbare Partitionen; 3,8 GiB RAM; kein Swap. Die LiveCD verwendet `/dev/sr0` und `/dev/loop0`. DNS war zunächst nicht funktionsfähig, weil `/etc/resolv.conf` keinen Nameserver enthielt. Danach wurden `1.1.1.1` und `8.8.8.8` eingetragen. Ping zu IP und `gentoo.org` war mit 0 % Verlust erfolgreich. `dhcpcd` erzeugte erneut eine Link-Local-Adresse; da die statische Route korrekt bevorzugt wird, bleibt sie während der SSH-Installation vorerst bestehen.
**Geplanter BIOS/MBR-Aufbau für ****`/dev/sda`**** (32 GiB):** `/dev/sda1` 1 GiB Boot, `/dev/sda2` 4 GiB Swap, `/dev/sda3` restlicher Platz Root. Vor dem Speichern mit `w` wird die Ausgabe von `p` kontrolliert.
- Nächste Erweiterung: bestätigten Datenträger mit GPT/UEFI oder passendem BIOS-Schema vorbereiten
