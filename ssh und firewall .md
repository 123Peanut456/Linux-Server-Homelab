Nextcloud Lab Setup & Server Hardening Documentation
Diese Dokumentation beschreibt die schrittweise Absicherung und Vorbereitung eines Ubuntu Server 24.04 LTS als Host-System für eine Docker-basierte Nextcloud-Instanz.

📋 Inhaltsverzeichnis
Phase 1: Vorbereitung & Firewall-Konfiguration (UFW)

Phase 2: SSH Key-Based Authentication (SSH-Schlüsselpaar)

Phase 3: Fehlersuche & Lessons Learned (Troubleshooting)

Phase 1: Vorbereitung & Firewall-Konfiguration (UFW)
1.1 Aktualisierung des Grundsystems
Befehle:

Bash
sudo apt update && sudo apt upgrade -y
Warum & Wofür?
Vor der Installation von Docker oder Exponierung von Diensten muss das Betriebssystem auf den neuesten Stand gebracht werden. 
Dadurch werden bekannte Sicherheitslücken (CVEs) geschlossen und die Systemstabilität für den späteren Container-Betrieb gewährleistet.

1.2 Konfiguration der Uncomplicated Firewall (UFW)
Befehle:

Bash
# Standard-Regeln festlegen: Alles Ausgehende erlauben, alles Eingehende blockieren
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Einzige benötigte Ports für das Server-Lab freigeben
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP (Nextcloud / Proxy)
sudo ufw allow 443/tcp   # HTTPS (TLS/SSL-Verschlüsselung)

# Firewall aktivieren und Status prüfen
sudo ufw enable
sudo ufw status verbose
Warum & Wofür?

Default Deny:
Verhindert, dass unbefugte Dienste oder versehentlich geöffnete Ports aus dem Netzwerk erreichbar sind (Prinzip der minimalen Rechte / Least Privilege).

Port 22 (SSH): Ermöglicht die Fernwartung des Servers per Terminal.

Port 80 & 443 (HTTP/HTTPS):
Notwendig für den späteren Zugriff auf die Nextcloud-Oberfläche sowie für das automatische Beziehen von SSL-Zertifikaten (z. B. via Let's Encrypt / Traefik / Nginx Proxy Manager).

Phase 2: SSH Key-Based Authentication (SSH-Schlüsselpaar)
2.1 Erstellung des Ed25519-Schlüsselpaares (Windows Client)
Befehl (Windows cmd / PowerShell):

DOS
ssh-keygen -t ed25519 -C "admin-nextcloud-lab"
Warum & Wofür?
Passwort-Anmeldungen sind anfällig für Brute-Force-Angriffe und Wörterbuch-Attacken. 
Ein Ed25519-Schlüsselpaar ist ein moderner, extrem sicherer und performanter Krypto-Standard auf Basis elliptischer Kurven. Er erzeugt zwei Schlüssel:

id_ed25519 (Private Key): Verbleibt streng geheim auf dem Windows-Client.

id_ed25519.pub (Public Key): Wird auf den Server übertragen.

2.2 Schlüssel auf dem Ubuntu-Server hinterlegen
Befehle (Ubuntu Server):

Bash
# Verstecktes SSH-Verzeichnis des Benutzers erstellen
mkdir -p ~/.ssh

# Öffentlichen Schlüssel in die Datei authorized_keys eintragen
nano ~/.ssh/authorized_keys
Inhalt der authorized_keys (eine einzelne, zusammenhängende Zeile):

Plaintext
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILIJ87EX9jD3nc9jFSjRniDS1+y6XiC1KhWgHj0XRH8x dein-name@example.com
Warum & Wofür?
Der SSH-Daemon auf dem Server liest bei einem Verbindungsaufbau die Datei ~/.ssh/authorized_keys aus.
Wenn der vom Client angebotene Schlüssel mit einem Eintrag in dieser Datei übereinstimmt, gewährt der Server Zugriff – ganz ohne Übertragung eines Passworts.

2.3 Absicherung der Dateirechte (POSIX Permissions)
Befehle (Ubuntu Server):

Bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
Warum & Wofür?
Der SSH-Dienst (sshd) besitzt einen eingebauten Sicherheitsmechanismus (StrictModes). 
Wenn Ordner oder Dateien für andere Systembenutzer schreibbar oder lesbar sind, 
ignoriert OpenSSH den Schlüssel aus Sicherheitsgründen komplett und fordert weiterhin ein Passwort an:

chmod 700 ~/.ssh: Nur der Eigentümer darf im Ordner lesen, schreiben und ihn betreten (rwx------).

chmod 600 ~/.ssh/authorized_keys: Nur der Eigentümer darf die Schlüsseldatei lesen und beschreiben (rw-------).

Phase 3: Fehlersuche & Lessons Learned (Troubleshooting)
Bei der Einrichtung traten typische Praxis-Hürden auf, die wie folgt analysiert und gelöst wurden:

Ordnerstruktur (/ssh/ vs. ~/.ssh/):

Problem: Der Ordner wurde anfangs fälschlicherweise als ~/ssh/ ohne führenden Punkt angelegt.

Ursache: Linux behandelt Ordner ohne Punkt als normale, sichtbare Verzeichnisse.
OpenSSH sucht Schlüssel jedoch zwingend im versteckten Systemordner ~/.ssh/.

Lösung: 
Korrektes Anlegen via mkdir -p ~/.ssh.

Syntaxfehler im Public Key (Fehlendes Leerzeichen):

Problem: 
Der Login schlug trotz korrekter Rechte fehl und fragte weiter nach dem Passwort.

Ursache:
Bei der manuellen Eingabe im nano-Editor verschmolz der Schlüssel-String mit dem Kommentar-Teil (...8xdein-name@example.com). 
Dadurch konnte OpenSSH das Schlüsselformat nicht mehr parsen.

Lösung: Einbauen des fehlenden Leerzeichens zwischen dem eigentlichen Schlüssel und der Kennung (...8x dein-name@...).

### 3.1 Praktische Herausforderungen beim Setup
* **Versteckte Systemordner in Linux:** 
  Erkenntnis: OpenSSH akzeptiert Schlüssel ausschließlich im versteckten Ordner `~/.ssh/` (mit führendem Punkt).
  Ein Pfad wie `~/ssh/` wird vom SSH-Daemon komplett ignoriert.
* **Exakte Syntax von SSH-Keys:**
  Erkenntnis: Öffentliche Schlüssel bestehen aus drei durch Leerzeichen getrennten Teilen (`<Typ> <Key> <Kommentar>`).
  Fehlt das Leerzeichen vor dem Kommentar, schlägt das Parsing fehl und SSH fällt auf die Passwort-Authentifizierung zurück.
* **Klippen bei der Terminal-Übertragung:**
  Erkenntnis: Beim Kopieren zwischen Windows und Linux-Terminal (Rechtsklick vs. Strg+Shift+V) können Zeichen verloren gehen.
  Der manuelle Weg über den `nano`-Editor erwies sich als die zuverlässigste Methode zur präzisen Erstellung.
