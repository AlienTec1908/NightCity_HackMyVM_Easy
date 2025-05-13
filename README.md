# NightCity - HackMyVM (Easy)

![NightCity.png](NightCity.png)

## Übersicht

*   **VM:** NightCity
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=NightCity)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 2023-06-19
*   **Original-Writeup:** https://alientec1908.github.io/NightCity_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, die User- und Root-Flags der Maschine "NightCity" zu erlangen. Der Weg dorthin begann mit der Entdeckung eines anonymen, beschreibbaren FTP-Zugangs und Hinweisen aus `robots.txt` und einer `reminder.txt`-Datei. Ein in einem Bild (`most-wanted.jpg` aus dem Webverzeichnis `/secret/`) mittels Steganographie verstecktes Passwort (`ThisIsTheRealPassw0rd!`) ermöglichte den SSH-Login als Benutzer `batman` (User-Flag). Ein weiteres Passwort (`ThatMadeMeL4ugh!`), das durch Aufhellen eines anderen Bildes sichtbar wurde, erlaubte den Wechsel zum Benutzer `joker`. Die Root-Flag (in Form von ASCII-Art) wurde im versteckten Home-Verzeichnis `.joker` gefunden, lesbar für den Benutzer `joker`.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `vi` / Editor
*   `nikto`
*   `nmap`
*   `gobuster`
*   `ftp`
*   `curl`
*   `stegseek`
*   `base64`
*   `hydra`
*   `msfconsole` (Metasploit, für vsftpd Exploit-Versuch)
*   `ssh`
*   `sudo` (versucht)
*   `find`
*   `id`
*   `getcap`
*   `ls`
*   `ss`
*   Bildbearbeitung (impliziert für Passwortfund)
*   Standard Linux-Befehle (`cat`, `su`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "NightCity" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration (Web, FTP):**
    *   IP-Adresse des Ziels (192.168.2.110) mit `arp-scan` identifiziert. Hostname `nightcity.vuln` in `/etc/hosts` eingetragen.
    *   `nmap`-Scan offenbarte Port 21 (FTP, vsftpd 3.0.3, anonymer Login erlaubt, Verzeichnis `/reminder` beschreibbar), Port 22 (SSH, OpenSSH 7.6p1) und Port 80 (HTTP, Apache 2.4.29).
    *   `nikto` und `gobuster` auf Port 80 fanden u.a. die Verzeichnisse `/secret/` und `/images/` (beide mit Directory Indexing) sowie `/robots.txt`.
    *   Anonymer FTP-Login war erfolgreich. Die Datei `/reminder/reminder.txt` wurde heruntergeladen ("Local user is in the coordinates"). Ein Upload-Versuch scheiterte.
    *   `robots.txt` enthielt den Hinweis: "Robin has the key!!".
    *   Im Webverzeichnis `/secret/` wurden die Bilder `most-wanted.jpg`, `some-light.jpg`, `veryImportant.jpg` gefunden.

2.  **Vulnerability Analysis (Steganographie & Credentials) & Initial Access (SSH als `batman`):**
    *   Mittels `stegseek most-wanted.jpg /usr/share/wordlists/rockyou.txt` wurde die Passphrase `japon` gefunden. Eine versteckte Datei (`pass.txt`) wurde extrahiert.
    *   Der Inhalt von `pass.txt` war Base64-kodiert und enthielt nach Dekodierung das Passwort `ThisIsTheRealPassw0rd!`.
    *   `hydra` bestätigte die Credentials `batman`:`ThisIsTheRealPassw0rd!` für FTP und SSH.
    *   Ein Exploit-Versuch gegen vsftpd 2.3.4 (Backdoor) mit Metasploit schlug fehl, da die Version 3.0.3 lief.
    *   Erfolgreicher SSH-Login als `batman` mit dem Passwort `ThisIsTheRealPassw0rd!`.
    *   Die User-Flag war identisch mit dem Passwort: `ThisIsTheRealPassw0rd!`.

3.  **Privilege Escalation (von `batman` zu `joker`):**
    *   `sudo -l` als `batman` zeigte keine `sudo`-Rechte.
    *   Standard-Enumeration (SUID, Capabilities, beschreibbare Dateien) lieferte keine direkten Vektoren.
    *   Durch Aufhellen eines der Bilder aus `/secret/` (vermutlich `some-light.jpg` oder `veryImportant.jpg`) wurde ein weiteres Passwort sichtbar: `ThatMadeMeL4ugh!`.
    *   Mittels `su joker` und Eingabe des Passworts `ThatMadeMeL4ugh!` wurde erfolgreich zum Benutzer `joker` gewechselt.

4.  **Flag Access (als `joker`):**
    *   `ls -la /home/` zeigte ein verstecktes Home-Verzeichnis `/home/.joker`.
    *   In `/home/.joker/` wurde die Datei `flag.txt` gefunden. Diese gehörte `root`, war aber für andere lesbar (`-rw-r--r--`).
    *   `cat flag.txt` zeigte die Root-Flag in Form von ASCII-Art und den Text "Good job!! You just discovered the criminal!". Das eigentliche Flag war identisch mit dem Passwort für `joker`: `ThatMadeMeL4ugh!`.

## Wichtige Schwachstellen und Konzepte

*   **Anonymer FTP-Zugang:** Ermöglichte das Herunterladen von Hinweisen.
*   **Steganographie:** Ein Passwort war mittels Steghide in einem Bild versteckt und konnte mit `stegseek` und einer Wortliste extrahiert werden.
*   **Information Disclosure in Bildern:** Ein weiteres Passwort wurde durch visuelle Analyse (Aufhellen) eines Bildes gefunden.
*   **Hinweise in Textdateien:** `robots.txt` und `reminder.txt` gaben Hinweise auf Benutzer oder Passwörter.
*   **Schwache Passwörter:** Die gefundenen Passwörter waren durch Enumeration oder Steganographie auffindbar.
*   **Unsichere Dateiberechtigungen:** Die Root-Flag-Datei (`/home/.joker/flag.txt`) war für den Benutzer `joker` lesbar, obwohl sie `root` gehörte.
*   **Versteckte Verzeichnisse:** Das Home-Verzeichnis von `joker` war als `.joker` versteckt.

## Flags

*   **User Flag (Passwort für `batman`):** `ThisIsTheRealPassw0rd!`
*   **Root Flag (Passwort für `joker` / Inhalt von `/home/.joker/flag.txt`):** `ThatMadeMeL4ugh!`

## Tags

`HackMyVM`, `NightCity`, `Easy`, `Anonymous FTP`, `Steganography`, `stegseek`, `Information Disclosure`, `Password Cracking`, `SSH`, `Linux`, `Privilege Escalation`, `Apache`
