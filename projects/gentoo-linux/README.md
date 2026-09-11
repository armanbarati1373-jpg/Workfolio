# 🐧 Gentoo Linux auf Proxmox

[← Workfolio](../../README.md)


> **Projektstatus am 27.08.2026 – Arbeit für heute beendet:** Netzwerk, SSH, Datenträger, Dateisysteme, Mount-Struktur, Stage3-Verifikation und -Installation sowie `make.conf` sind abgeschlossen. **Noch nicht ausgeführt:** DNS-Kopie in das neue System, Mounts von `/proc`, `/sys`, `/dev` und `/run` sowie Eintritt in den Chroot. Exakter Fortsetzungspunkt für den 28.08.2026: Abschnitt **Chroot vorbereiten**.

## Tagesbericht – 27.08.2026
### Zusammenfassung
Die neue Gentoo-VM unter Proxmox wurde von der Minimal-CD gestartet und bis zur vollständig vorbereiteten Stage3-Basisinstallation aufgebaut. Der Remote-Zugriff über MobaXterm/SSH funktioniert. Das Zielsystem liegt auf einer geprüften BIOS/MBR-Datenträgerstruktur mit XFS und aktivem Swap. Das offizielle Stage3 AMD64 OpenRC Multilib wurde per SHA256 und PGP verifiziert, entpackt und für die vorhandenen zwei vCPU konfiguriert.
### Durchgeführte Arbeiten
- Gentoo Minimal Installation CD in der neuen Proxmox-VM gestartet und automatische Root-Anmeldung bestätigt.
- Netzwerkschnittstelle `ens18` analysiert; fehlenden DHCP-Lease und automatisch vergebene `169.254.x.x`-Link-Local-Adresse erkannt.
- Statische IPv4-Adresse `192.168.50.156/24` und Gateway `192.168.50.250` eingerichtet.
- DNS mit `1.1.1.1` und `8.8.8.8` eingerichtet; IP- und Namensauflösungstests mit 0 % Paketverlust abgeschlossen.
- Temporäres Root-Passwort gesetzt, `sshd` gestartet und erfolgreiche SSH-Anmeldung über MobaXterm hergestellt.
- Bootmodus als BIOS/Legacy identifiziert; Zielplatte als leere QEMU-Festplatte `/dev/sda` mit 32 GiB bestätigt.
- MBR/DOS-Partitionstabelle erstellt und vor dem Schreiben kontrolliert:
	- `/dev/sda1`: 1 GiB, Boot-Flag, Typ 83, später XFS für `/boot`
	- `/dev/sda2`: 4 GiB, Typ 82, Swap
	- `/dev/sda3`: 27 GiB, Typ 83, XFS für Root
- Swap initialisiert und aktiviert; 4 GiB per `swapon --show` und `free -h` bestätigt.
- Root- und Boot-Dateisystem mit XFS erstellt und per `lsblk -f` kontrolliert.
- Mount-Struktur nach Korrektur der Reihenfolge erfolgreich hergestellt:
	- `/dev/sda3 → /mnt/gentoo`
	- `/dev/sda1 → /mnt/gentoo/boot`
- Systemzeit als korrekt bestätigt: `13:06 UTC` entspricht `15:06 Europe/Berlin`.
- Offizielles Stage3 `stage3-amd64-openrc-20260823T153057Z.tar.xz` geladen.
- Integrität per SHA256 mit Ergebnis `OK` bestätigt.
- Gentoo-Release-Schlüssel aus der LiveCD importiert und PGP-Signatur als `Good signature` bestätigt.
- Stage3 mit erhaltenen Permissions, Extended Attributes und numerischen Eigentümern nach `/mnt/gentoo` entpackt.
- Basisstruktur, Bash und merged-usr-Symlinks geprüft; belegter Platz rund 1,5 GiB.
- Hardware für Compile-Parallelität geprüft: 2 vCPU, 3,8 GiB RAM und 4 GiB Swap.
- Sichere Stage3-Compiler-Flags `-O2 -pipe` beibehalten und `MAKEOPTS="-j2"` einmalig ergänzt.
### Fehlerbilder und Lösungen
- **Network is unreachable:** Ursache war fehlende IPv4-Konfiguration; durch statische IP und Gateway behoben.
- **DNS-Auflösung fehlgeschlagen:** `/etc/resolv.conf` enthielt keinen Nameserver; `1.1.1.1` und `8.8.8.8` eingetragen.
- **`Arman: command not found`****:** Text wurde am Shell-Prompt als Befehl interpretiert; kein Systemfehler.
- **`sda3 xfs: command not found`****:** unvollständiger Befehl; mit `mkfs.xfs /dev/sda3` korrigiert.
- **Verdeckter und doppelter Boot-Mount:** Boot wurde vor Root gemountet; Mounts vollständig gelöst und in der Reihenfolge Root → Boot neu aufgebaut.
- **`df: no file systems processed`****:** `-ht` mit kleinem `t` statt `-hT` verwendet; Groß-/Kleinschreibung erklärt und korrigiert.
- **Fehlende ****`.asc`****-Datei bei GPG-Prüfung:** Signaturdatei nachgeladen und Prüfung erfolgreich wiederholt.
- **GPG Trust-Warnung:** als lokale Trust-Datenbankwarnung eingeordnet; kryptografische Ausgabe `Good signature` und offizieller Fingerprint wurden bestätigt.
### Technischer Endstand
- Netzwerk und SSH: funktionsfähig
- Datenträger und Partitionen: vollständig vorbereitet
- XFS und Swap: funktionsfähig
- Root/Boot-Mounts: korrekt
- Stage3 OpenRC Multilib: verifiziert und entpackt
- `make.conf`: geprüft, `MAKEOPTS="-j2"` gesetzt
- Chroot: noch nicht betreten
### Exakter nächster Schritt – 28.08.2026
1. `/etc/resolv.conf` mit `cp --dereference` nach `/mnt/gentoo/etc/` kopieren.
2. `/proc`, `/sys`, `/dev` und `/run` gemäß Gentoo-Handbuch einhängen und Mounts prüfen.
3. Mit `chroot /mnt/gentoo /bin/bash` in das neue System wechseln.
4. `/etc/profile` laden und Prompt sichtbar auf `(chroot)` setzen.
5. Danach Portage Repository und Systemprofil konfigurieren.
## 1. Projektübersicht
### Projektziel
Gentoo Linux wird schrittweise auf einer neuen virtuellen Maschine unter Proxmox installiert. Sämtliche Arbeitsschritte, Befehle, Prüfungen, Fehlerbilder, Ursachen, Entscheidungen, Lösungen und Nachweise werden fortlaufend und ausführlich dokumentiert.
Die Dokumentation wird parallel in zwei Formen geführt:
- **Projektseite:** vollständige technische Chronik mit Details und Diagnose.
- **Anleitung:** kurze, einfache und wiederholbare Schritte für eine spätere Neuinstallation.
Zugehöriger zweisprachiger Anleitung-Bereich: [
### Aktueller Stand
- **Virtualisierung:** Proxmox
- **Betriebssystem:** Gentoo Linux Minimal Installation CD
- **Konsole:** Proxmox-Konsole
- **Benutzer:** `root` (automatische Anmeldung in der Live-Umgebung)
- **IP-Adresse:** `192.168.50.156/24`
- **Remote-Zugriff:** MobaXterm über SSH vorgesehen
- **Status:** Für heute pausiert – Fortsetzung bei der Chroot-Vorbereitung
## 2. Beobachtungen und Diagnose
Beim Start erschien:
```plain text
sed: can't read /etc/conf.d/net: No such file or directory
Connecting... 0s [offline]
WARNING: NetworkManager has started, but is inactive
```
Die Live-Umgebung wurde dennoch vollständig gestartet. Die Meldung allein beweist keinen Netzwerkausfall. Vor dem SSH-Zugriff werden Schnittstelle, IP-Adresse, Route und Erreichbarkeit geprüft.
Außerdem wurde am Prompt `Arman` eingegeben. Die Shell interpretierte dies als Befehl und antwortete:
```plain text
-bash: Arman: command not found
```
Das ist kein Systemfehler.
## 3. Arbeitsschritt – SSH-Zugriff aus MobaXterm
### 3.1 Prüfung in der Proxmox-Konsole
```bash
ip -br a
ip route
ping -c 3 1.1.1.1
```
### 3.2 Temporäres Root-Passwort für die Live-Umgebung setzen
```bash
passwd root
```
Das Passwort wird nicht in Notion gespeichert oder in Screenshots sichtbar gemacht.
### 3.3 SSH-Dienst starten und prüfen
```bash
/etc/init.d/sshd start
rc-service sshd status
ss -lntp | grep ':22'
```
Falls `rc-service` in der Live-Umgebung nicht verfügbar ist, genügt die direkte Startmethode `/etc/init.d/sshd start`.
### 3.4 Verbindung aus MobaXterm
- Neue **SSH Session** erstellen.
- Unter **Remote host** die bekannte IP-Adresse eintragen.
- **Specify username** aktivieren und `root` verwenden.
- Port `22` verwenden.
- Beim ersten Verbindungsaufbau den Host-Key prüfen und akzeptieren.
- Das zuvor gesetzte temporäre Root-Passwort eingeben.
## 4. Sicherheits- und Betriebsnotizen
- Das Passwort gilt nur für die aktuelle LiveCD-Sitzung und muss nach einem Neustart gegebenenfalls neu gesetzt werden.
- Die IP-Adresse und das Passwort dürfen in Screenshots nicht gemeinsam sichtbar sein.
- Nach der eigentlichen Gentoo-Installation wird ein normaler Administrationsbenutzer mit SSH-Key eingerichtet; dauerhafter Root-Login per Passwort ist nicht das Ziel.
- Die Proxmox-Konsole bleibt als Rückfallzugang geöffnet, bis SSH erfolgreich getestet ist.
## 5. Nachweis und nächster Schritt
- [x] Ausgabe von `ip -br a` und `ip route` geprüft
- [x] Statische Adresse `192.168.50.156/24` eingerichtet und Ping erfolgreich getestet
- [x] Temporäres Root-Passwort gesetzt
- [x] `sshd` gestartet
- [x] SSH-Verbindung aus MobaXterm erfolgreich hergestellt
- [ ] Bootmodus und Datenträger read-only erfassen
- [ ] Vollständigen, lesbaren Screenshot des erfolgreichen MobaXterm-Terminals aufnehmen; Passwort ausblenden
- [ ] Danach Datenträgerlayout und Gentoo-Installationsplanung beginnen
## Fortschrittsnotiz – 27.08.2026
Die Gentoo-Minimal-LiveCD wurde in der neuen Proxmox-VM erfolgreich gestartet. Die erste Diagnose zeigt einen inaktiven NetworkManager beim Boot.
Die anschließende Prüfung bestätigt das Netzwerkproblem: `ens18` ist zwar aktiv, hat jedoch zunächst keine IPv4-Adresse. Die Routingtabelle ist leer und `ping 1.1.1.1` scheitert mit `Network is unreachable`. Der SSH-Dienst ist erwartungsgemäß noch gestoppt.
Der DHCP-Versuch lieferte anschließend nur `169.254.139.27/16`. Dies ist eine automatisch vergebene Link-Local-Adresse und kein gültiger Lease aus dem vorgesehenen Netz. Die Route `default dev ens18` besitzt kein nutzbares Gateway. Tests gegen `1.1.1.1` und `8.8.8.8` zeigen 100 % Paketverlust und `Destination Host Unreachable`. Nächster Schritt: In Proxmox Bridge/VLAN/Link prüfen und danach die bekannte statische IPv4-Adresse, das Präfix und das Gateway manuell setzen.
[Archivierte Screenshots und Nachweise vom 27.08.2026](screenshots-2026-08-27.md)
## Dokumentationsregel ab 02.09.2026
- Neue Gentoo-Schritte werden mit passenden Screenshots/Nachweisen dokumentiert.
- Screenshots werden direkt auf der Hauptprojektseite chronologisch und möglichst beim zugehörigen Arbeitsschritt abgelegt.
- Passwörter, SSH-Keys oder andere Zugangsdaten dürfen auf Screenshots nicht sichtbar sein.
- Die separate Nachweis-Seite wird nur noch als Altbestand/Archiv verwendet; neue Screenshots gehören in diese Hauptprojektseite.
## 📸 Screenshots / Nachweise – chronologisch
Die bisherigen Gentoo-Nachweise sind ab jetzt direkt in der Hauptprojektseite abgelegt. Die Reihenfolge entspricht dem tatsächlichen Arbeitsablauf vom 27.08.2026.
### 1. Gentoo Minimal LiveCD gestartet und Root-Anmeldung bestätigt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/4b38fb7ae0975a3e.png)
### 2. Kein IPv4-Lease, leere Routingtabelle und SSH noch gestoppt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/eb09bd68f2e0b3e0.png)
### 3. Link-Local-Adresse `169.254.x.x` und fehlgeschlagener Ping
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/bef63dfbda78bf9f.png)
### 4. Systemprüfung: IP, BIOS, Datenträger, RAM und DNS
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/c3ea47a43603d469.png)
### 5. Statische IP, Gateway und DNS korrigiert; Ping erfolgreich
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/7bbf73417974bc12.png)
### 6. Geplante Partitionstabelle in `fdisk`
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/01048d8bb29b5f86.png)
### 7. Finale MBR-Partitionen mit `fdisk -l` bestätigt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/5c81b9fa7f946a8c.png)
### 8. XFS für Boot erstellt und Swap aktiviert
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/d402ffe6cfd24fa1.png)
### 9. XFS für Root erstellt und mit `lsblk` geprüft
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/0d9e7405a271bdd4.png)
### 10. Erster Mount-Versuch mit falscher Reihenfolge
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/d46c410f0f7c0f51.png)
### 11. Doppelter/verdeckter Boot-Mount erkannt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/6f8893d3d879c15a.png)
### 12. Verbleibenden Boot-Mount kontrolliert
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/892e9185e5a183da.png)
### 13. Mount-Fehler analysiert und weiter eingegrenzt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/30b249c548765f74.png)
### 14. Mount-Struktur korrekt neu aufgebaut; Tippfehler bei `df -ht` erkannt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/f00137e175099990.png)
### 15. Root- und Boot-Mount mit `df -hT` bestätigt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/7931ee0bd4e3180f.png)
### 16. Aktuelles offizielles Stage3 ermittelt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/796d9e992ec59222.png)
### 17. Stage3 per SHA256 verifiziert
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/24d0f8f63a317c40.png)
### 18. Stage3 erfolgreich nach `/mnt/gentoo` entpackt und geprüft
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/297191539faae624.png)
### 19. Zwei vCPU geprüft und `MAKEOPTS="-j2"` gesetzt
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/a5269b20167167f8.png)

> Passwörter, SSH-Keys und andere Zugangsdaten werden in Screenshots weiterhin nicht sichtbar gespeichert.

## Chroot vorbereiten – DNS-Konfiguration
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/bf5cb2d88d6ba6c1.png)
## Chroot vorbereiten – Systemdateisysteme einhängen
In diesem Schritt werden `/proc`, `/sys`, `/dev` und `/run` in das neue Gentoo-System eingebunden, damit Prozesse, Geräte, Kernelinformationen und Laufzeitdaten im Chroot verfügbar sind.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/814dc4539ccacfc9.png)
## In das neue Gentoo-System wechseln (Chroot)
Nach dem Einhängen der Systemdateisysteme wird in die entpackte Gentoo-Installation unter `/mnt/gentoo` gewechselt. Anschließend wird das Gentoo-Profil geladen und der Shell-Prompt mit `(chroot)` gekennzeichnet, damit jederzeit erkennbar bleibt, dass Befehle jetzt im neuen System ausgeführt werden.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/22afc816b72bafc7.png)
## Portage-Repository vorbereiten
Nach dem Wechsel in den Chroot wird zuerst geprüft, ob die Repository-Konfiguration für Gentoo vorhanden ist. Anschließend wird ein aktueller Portage-Snapshot geladen, damit Paketinformationen und Ebuilds für die weiteren Installationsschritte verfügbar sind.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/6a7e80bcda4d7e3f.png)
## Gentoo-Systemprofil auswählen
Nach der Aktualisierung des Portage-Repositories wird das aktive Gentoo-Profil geprüft. Das Profil legt unter anderem Standard-USE-Flags, ABI-Vorgaben und grundlegende Systementscheidungen fest. Aktiv ist `default/linux/amd64/23.0`. Dieses stabile Standardprofil passt zur verwendeten AMD64-OpenRC-Multilib-Installation und bleibt daher unverändert. Ein Wechsel auf systemd-, Desktop-, no-multilib- oder hardened-Profile ist für diese Basisinstallation nicht notwendig.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/53725ce1b2f8a436.png)
## Zeitzone und Sprache konfigurieren
Für die weitere Installation wird die Zeitzone auf `Europe/Berlin` gesetzt. Anschließend werden die gewünschten Locales aktiviert und generiert, damit Datum, Uhrzeit, Zeichenkodierung und sprachabhängige Programmeinstellungen korrekt funktionieren.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/57dd538d634d8c7d.png)
## Locale konfigurieren
Für die Systemumgebung wurden die UTF-8-Locales `de_DE.UTF-8` und `en_US.UTF-8` aktiviert und generiert. Als Standard ist `en_US.UTF-8` gesetzt, damit Systemmeldungen und Werkzeuge während Installation und Administration auf Englisch bleiben; die deutsche Locale steht zusätzlich zur Verfügung.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/82f4e40c6a9155b8.png)
## Systembasis aktualisieren
Nach der Einrichtung von Repository, Profil, Zeitzone und Locale wird die installierte Systembasis gegen den aktuellen Portage-Stand geprüft und anschließend vollständig aktualisiert.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/887814dc408d0a45.png)

## Veraltete Abhängigkeiten prüfen und bereinigen
Nach dem erfolgreichen World-Update wird geprüft, ob nicht mehr benötigte Pakete und Abhängigkeiten vorhanden sind. `emerge --depclean` hat keine Pakete zur Entfernung ausgewählt. Da die Stage3-Installation zunächst keine eigene World-Datei enthielt, wurde `sys-apps/portage` anschließend mit `--noreplace` als bewusst installiertes Paket in `/var/lib/portage/world` aufgenommen. Damit ist der World-Set nicht mehr leer.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/27fde297bdd0b245.png)
## Kernel installieren
Für die virtuelle Maschine wird ein vorgebauter Gentoo-Distribution-Kernel verwendet, um eine zuverlässige Installation ohne langen lokalen Kernel-Build zu erhalten. Installiert ist `sys-kernel/gentoo-kernel-bin-6.18.48` mit Initramfs. Die Boot-Dateien `vmlinuz-6.18.48-gentoo-dist-bin`, `initramfs-6.18.48-gentoo-dist-bin.img`, `System.map-6.18.48-gentoo-dist-bin` und die zugehörige Kernel-Konfiguration liegen unter `/boot`. `eselect kernel list` zeigt `linux-6.18.48-gentoo-dist-bin` als aktives Kernel-Ziel.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/8412f6a2a4f0afe6.png)
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/93af16f6c3b54d0d.png)
## Dateisysteme dauerhaft einbinden (`/etc/fstab`)
Damit Root-Dateisystem, Boot-Partition und Swap nach dem Neustart automatisch eingebunden bzw. aktiviert werden, wird `/etc/fstab` anhand der vorhandenen UUIDs erstellt. Eingetragen sind das Root-Dateisystem auf XFS, die separate XFS-Boot-Partition sowie die Swap-Partition. Die vorhandenen Beispielzeilen aus der Stage3 bleiben auskommentiert und haben keine Wirkung.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/00b06615c67f986d.png)
## Permanente Netzwerkkonfiguration vorbereiten
Nach der Dateisystemkonfiguration wird die Netzwerkschnittstelle `ens18` dauerhaft eingerichtet, damit die statische IPv4-Adresse und das Gateway nach dem ersten Start des installierten Systems automatisch verfügbar sind.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/fe4269db83856613.png)
## Benutzerzugang und SSH vorbereiten
Für den späteren administrativen Zugriff wird ein dauerhaftes Root-Passwort gesetzt und anschließend ein normaler Benutzer mit Administrationsrechten vorbereitet. Danach wird der SSH-Dienst für den ersten Start des installierten Systems aktiviert. Zugangsdaten selbst werden nicht in der Dokumentation gespeichert.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/f2e27af5eafb681c.png)
## Administratorrechte für `arman` konfigurieren
Der Benutzer `arman` ist bereits Mitglied der Gruppe `wheel`. Im nächsten Schritt wird `sudo` eingerichtet, damit administrative Befehle gezielt mit erhöhten Rechten ausgeführt werden können, ohne dauerhaft als `root` zu arbeiten.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/a502b65af14f4d4b.png)
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/0e282b8bca67d023.png)
## GRUB-Bootloader installieren
Für die BIOS-/Legacy-VM wurde GRUB 2 als Bootloader installiert. GRUB wurde für die Plattform `i386-pc` direkt in den MBR von `/dev/sda` geschrieben. Anschließend wurde die Konfiguration unter `/boot/grub/grub.cfg` erzeugt. Dabei wurden der installierte Gentoo-Kernel `vmlinuz-6.18.48-gentoo-dist-bin` und die zugehörige Initramfs erkannt. Die Installation wurde ohne Fehler abgeschlossen.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/fdbd580652c13e1c.png)
## Abschlussprüfung vor dem ersten Neustart
Vor dem ersten Start des installierten Systems werden Bootloader, Kernel, `fstab`, Netzwerkdienste und SSH noch einmal kurz kontrolliert. Erst wenn diese Prüfungen ohne Fehler abgeschlossen sind, wird die Chroot-Umgebung verlassen und die virtuelle Maschine neu gestartet.
![Screenshot zum dokumentierten Arbeitsschritt](../../assets/0c22deabe09f0393.png)
