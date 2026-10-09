## Projektüberblick

In diesem privaten Lernprojekt habe ich eine Ubuntu-Server-VM in VirtualBox eingerichtet, den Netzwerkzugriff vorbereitet, eine SSH-Verbindung von Windows aus hergestellt sowie eine Nextcloud-Instanz für die Dateiablage und -übertragung aufgesetzt und konfiguriert.

**Projektstatus:** Fernzugriff eingerichtet; Nextcloud-Dienst aktiv und über Webinterface erreichbar; Dateiübertragungen via SCP und Nextcloud erfolgreich getestet.

> **Hinweis zur Dokumentation:** Die Schritte wurden mit Unterstützung von KI und eigener praktischer Arbeit umgesetzt. Diese Dokumentation dient als nachvollziehbares Lernprotokoll. Nicht jeder Befehl muss auswendig beherrscht werden; wichtig ist, den Zweck erklären, die Schritte nachvollziehen und sie bei Bedarf sicher erneut ausführen zu können.

---

## Verwendete Umgebung

- **Virtualisierung:** Oracle VirtualBox
- **Serverbetriebssystem:** Ubuntu Server
- **Client:** Windows mit PowerShell & Webbrowser
- **Fernzugriff:** SSH (OpenSSH)
- **Dateiübertragung über SSH:** SCP
- **Cloud-Dateiablage:** Nextcloud (z. B. via Snap / Docker / Apache)
- **Netzwerk:** FRITZ!Box bzw. Heimrouter (Bridged Network)

---

## 1. Netzwerkkonfiguration der virtuellen Maschine

In VirtualBox wurde der Netzwerkadapter der Ubuntu-VM auf **Netzwerkbrücke (Bridged Adapter)** gestellt.
Damit erscheint die VM im Heimnetzwerk als eigenständiges Gerät und erhält eine eigene IP-Adresse vom Router.

Beispiel-IP-Adresse der VM:

```text
192.168.xx.xx

Die Adresse kann sich ändern, wenn der Router der VM eine neue DHCP-Adresse zuweist.
Für eine dauerhaft stabile Adresse kann später eine DHCP-Reservierung im Router eingerichtet werden.
Warum ist das wichtig?
Der Windows-PC muss die VM direkt über das lokale IP-Netzwerk erreichen können, bevor SSH,
Nextcloud oder andere Dienste funktionieren.

2. OpenSSH-Server installieren und aktivierenAuf der Ubuntu-VM wurden folgende Befehle ausgeführt:

sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh


Befehl                       Zweck
sudo apt update              Aktualisiert die lokalen Paketlisten.
sudo apt install -y
openssh-server               Installiert den OpenSSH-Server. -y bestätigt Rückfragen automatisch.
sudo systemctl enable
 --now ssh                   Aktiviert den SSH-Dienst für zukünftige Systemstarts und startet ihn sofort.

Optional lässt   sich der Dienststatus prüfen:

Bashsudo systemctl status ssh

3. Firewall-Konfiguration auf dem Ubuntu-Server (UFW)
Damit SSH- und Web-Zugriffe nicht von der internen Ubuntu-Firewall geblockt werden,
wurden die benötigten Ports explizit freigeschaltet:

Bash# SSH-Port freigeben
sudo ufw allow 22/tcp

# HTTP-Port für Nextcloud freigeben
sudo ufw allow 80/tcp

# Firewall aktivieren
sudo ufw enable

4. Verbindung vom Windows-PC per PowerShellIn der Windows-PowerShell wurde eine SSH-Verbindung zur VM aufgebaut:

PowerShellssh dein-benutzername@192.168.xx.xx

Beim ersten Verbindungsaufbau zeigt SSH den Host-Key-Fingerprint an, der einmalig bestätigt werden muss.
Anschließend erfolgt die Authentifizierung per Passwort.Was ermöglicht SSH?
SSH stellt eine verschlüsselte Verbindung zu einer entfernten Kommandozeile her.
So kann der Server vollumfänglich administriert werden, ohne direkt an der VirtualBox-Konsole arbeiten zu müssen.

5. Zeitsynchronisation prüfen und aktivierenWährend der Einrichtung trat bei HTTPS-/Zertifikatsprüfungen folgender Fehler auf:

Plaintextx509:

certificate has expired or is not yet valid

Eine falsch eingestellte Systemzeit lässt Zertifikatsprüfungen fehlschlagen.
Die Zeitsynchronisation wurde deshalb aktiviert und überprüft:

Bashsudo timedatectl set-ntp true
timedatectl status

Erkenntnis:

set-ntp true aktiviert die Netz-Zeitsynchronisation über NTP.
Bei weiterhin abweichender Uhrzeit müssen Synchronisationsstatus und Netzwerkverbindung gesondert geprüft werden.

6. Nextcloud installieren und im Browser einrichten
A. Installation auf dem Ubuntu-ServerNextcloud wurde auf dem Server installiert
(beispielsweise über das vereinfachte Snap-Paket oder als Container):

Bash# Beispiel für Snap-Installation:
sudo snap install nextcloud

B. Ersteinrichtung über das WebinterfaceAm Windows-PC wurde der Browser geöffnet und die Server-IP aufgerufen:

http://192.168.xx.xx

Beim ersten Aufruf wurde das Administrator-Konto (Benutzername und Passwort) angelegt.
Die Ersteinrichtung wurde abgeschlossen und das Nextcloud-Dashboard geladen.

7. Dateiübertragung testen:
SCP vs. NextcloudIn diesem Projekt wurden zwei unterschiedliche Wege zur Dateiübertragung getestet und gegenübergestellt:
Weg A:
Dateiübertragung per SCP (Kommandozeile)Für die direkte Übertragung zwischen Windows und Ubuntu wurde scp in der PowerShell verwendet:

PowerShellscp "C:\Pfad\zu\Datei.jpg" dein-benutzername@192.168.xx.xx:~/

Überträgt Dateien direkt in das Home-Verzeichnis (~/) des Linux-Benutzers.
Rechte beachten: Sollten Dateien systemweit oder für andere Dienste genutzt werden,
müssen Rechte und Besitzer mittels chmod bzw. chown angepasst werden.

Weg B: Cloud-Ablage per Nextcloud (Webinterface)Dateien wurden über die grafische Oberfläche im Browser
hochgeladen, strukturiert und verwaltet.
Unterschied: SCP kopiert Dateien direkt und verschlüsselt über das SSH-Protokoll;
Nextcloud bietet eine vollständige Cloud-Plattform inklusive Benutzerverwaltung, Teilen-Funktion und Weboberfläche.

8. Zugriff von außerhalb des Heimnetzes (Ausblick & Sicherheit)
Die Adresse 192.168.xx.xx ist eine private IPv4-Adresse und nur im lokalen Heimnetz erreichbar.
Sicherheitshinweis für Fernzugriffe:
Serverdienste wie SSH oder Nextcloud sollten niemals ungeschützt direkt per Port-Freigabe ins Internet gestellt werden.
Empfohlen wird die Absicherung über einen eigenen VPN-Zugang (z. B. WireGuard / OpenVPN) ins Heimnetz.

9. Sinnvolle Tests und FehlersucheBei Verbindungsproblemen kann schrittweise diagnostiziert werden:
Vom Windows-PC:

PowerShell# Erreichbarkeit im Netzwerk prüfen
ping 192.168.xx.xx

# SSH-Verbindung mit ausführlicher Diagnose starten

ssh -v dein-benutzername@192.168.xx.xx

Auf dem Ubuntu-Server:

Bash# IP-Adressen und Schnittstellen anzeigen
ip a

# Dienststatus von SSH prüfen
sudo systemctl status ssh

# Status der Firewall prüfen

sudo ufw status

10. Was ich in diesem Projekt praktisch gelernt habe
Virtualisierung & Netzwerke: Einrichtung einer VM mit Netzwerkeinbindung per Netzwerkbrücke (Bridged Adapter).
Linux-Administration: Paketinstallation (apt/snap), Systemdienste mit systemctl verwalten und Verzeichnisrechte verstehen.
Fernzugriff & Sicherheit: SSH-Server einrichten, Verbindungen herstellen und Ports gezielt mit ufw absichern.
Fehlerdiagnose: Zertifikatsfehler auf falsche Systemzeiten (timedatectl) zurückführen und beheben.
Cloud-Services: Unterschied zwischen CLI-Dateiübertragung (scp) und vollumfänglichen Web-Services (Nextcloud) praktisch nachvollziehen.

