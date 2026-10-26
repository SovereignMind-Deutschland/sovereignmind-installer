# SovereignMind – Installer

<p align="center"><img src="logo.svg" alt="SovereignMind" width="300"></p>

**On-Premise KI-Assistent für den DACH-Mittelstand.** SovereignMind ist ein DSGVO-konformer
Chat-Assistent über die eigenen Firmendokumente (RAG), der komplett lokal beim Kunden läuft.
Dokumente und Prompts verlassen das Netzwerk nicht, es gibt keine Anbindung an OpenAI, Google
oder andere Cloud-KI-APIs.

Dieses Repo enthält nur das, was für die Installation gebraucht wird. Der Quellcode ist nicht Teil davon.

**Stand:** SovereignMind wird derzeit ausschließlich als Demo zur Erprobung bereitgestellt. Die Nutzung im Produktivbetrieb ist nicht gestattet.
Ob und wann offizielle Lizenzen angeboten werden, ist offen; ein Anspruch auf den Abschluss einer Lizenz besteht nicht.

Präsentation: die aktuelle Fassung mit 18 Folien liegt auf der Github Page unter `praesentation/`. Themen: Problem, Lösung, Architektur, Funktionen, Assistenten, Schnittstellen, Stimmen, Tech-Stack und Geschäftsmodell).

## Wofür ist das gedacht?

Kanzleien, Personalabteilungen, Gesundheitswesen, Industrie: Überall liegen sensible Dokumente
(Verträge, Handbücher, Akten, Fertigungspläne), die nicht in eine US-Cloud dürfen. Ohne
freigegebene Alternative landen sie oft trotzdem in privaten ChatGPT-Accounts. SovereignMind
ist die selbst gehostete Alternative.

## Funktionen

- **Lokale KI:** Sprachmodell und Embeddings laufen auf der Hardware des Kunden (Ollama nativ
  auf dem Rechner). Das Chat-Modell wird automatisch nach
  verfügbarem VRAM gewählt (NVIDIA, AMD, sonst CPU) und kann der Admin im Bereich
  „KI-Einstellungen“ wechseln.
- **Chat mit Quellenbelegen:** Fragen in natürlicher Sprache, Antworten mit klickbaren
  Verweisen ins Originaldokument. Antworten immer auf Deutsch.
- **Dokumentenverwaltung:** PDF, DOCX, Tabellen u. a. werden automatisch geparst und
  durchsuchbar gemacht. Kategorien, Duplikaterkennung, Verarbeitungsstatus.
- **Mehrere Nutzer und Rollen:** Admin und Member, eigenes Branding pro Unternehmen.
- **Assistenten und Prompt-Bibliothek:** Assistenten mit Rolle, Persönlichkeit, Avatar und
  eigener Stimme sowie wiederverwendbare Vorlagen für das ganze Team.
- **Anbindung interner Systeme:** HTTP-Schnittstellen (DMS, ERP, Fachdatenbanken) lassen sich
  als Werkzeuge einbinden, die das Modell bei Bedarf selbst aufruft. Fertige Vorlagen für
  HubSpot, Zendesk, Stripe, Notion, Jira Cloud, Hero und Microsoft 365.
- **Antwort-Cache:** Wiederholte Fragen werden schneller beantwortet (Retrieval-Cache, optional Antwort-Cache).
  Berechtigungen und Wissensstand fließen in den Cache ein; der Admin kann ihn leeren oder zurücksetzen.
- **Dateien im Chat:** Dokumente direkt in einer Unterhaltung hochladen, ohne sie dauerhaft
  abzulegen.
- **Sprache:** Vorlesen mit mehreren Stimmen (Ramona, Karlsson, Thorsten), lokal
  verarbeitet (Qwen3-TTS, standardmäßig auf der CPU; die GPU wählt der Admin im Voice-Tab, die Unterstützung wird dann nachgeladen);
  der Admin schaltet es pro Unternehmen frei. Das Diktat ist noch nicht verfügbar.
- **Wissenspakete:** Vorverarbeitete Gesetze und Regelwerke als Pakete, die der Admin
  aus dem Katalog installiert oder als Datei importiert.
- **Themes:** Jeder Nutzer wählt Stil, Hell/Dunkel und Akzentfarbe; das Unternehmen kann eine
  Akzentfarbe vorgeben.
- **Support-Nachrichten:** Nutzer schreiben dem Admin über ein Kontaktformular.
- **Admin-Bereich:** Nutzer, Dokumente, Branding, Lizenz, KI-Einstellungen (Modelle, Assistenten,
  Vorlagen, Sprache), Wissenspakete, Support, Nutzungsstatistik, Logs, Hardware-Dashboard
  (unter Windows auf Wunsch mit Companion-Agent für genaue Host-Werte).

## Architektur in Kürze

```
Web-Oberfläche ──> Backend (Auth, Rollen, Lizenz, Mandanten)
                      ├─ Inferenz (Ollama)
                      ├─ Vektordatenbank (Qdrant)
                      ├─ Dokumentenverarbeitung (Docling)
                      ├─ Sprache (Vorlesen)
                      └─ PostgreSQL
```

Alles läuft nach der Installation als Dienst im Hintergrund. Es ist weder Docker noch WSL noch
sonst eine Vorarbeit nötig; Ollama und die Modelle liefert der Installer mit.

## Voraussetzungen

- Windows 10 (1809) / 11 / Server 2019+ (x64) oder Ubuntu 24.04 / Debian 12 (x64)
- Administratorrechte (Windows) bzw. `sudo` (Linux)
- Linux: ca. 13 GB Download und ca. 26 GB Platz nach der Installation
- Für gute Antwortzeiten eine GPU mit ausreichend VRAM; ohne GPU läuft alles auf der CPU, deutlich langsamer
- Ein Lizenzschlüssel (Token) und die E-Mail-Adresse, auf die er ausgestellt wurde (vom Anbieter)

## Installation

Windows: `SovereignMind-Setup-Full-<Version>.exe` (samt `-N.bin`-Teilen; zweite Variante: `-Minimal`) aus den Releases herunterladen und ausführen.

Linux: eine Datei, ein Befehl. Das Bundle entpackt sich selbst, installiert die Pakete nacheinander und löscht alles
Entpackte am Ende wieder. Eine Release-Datei darf bei GitHub höchstens 2 GiB groß sein, deshalb liegt das Bundle in Teilen
(`sovereignmind-installer-full_<Version>_amd64.run.part01` bis `partNN`; zweite Variante: `-minimal`). Alle Teile und `SHA256SUMS.txt` in einen Ordner laden:

```bash
cat sovereignmind-installer-full_<Version>_amd64.run.part* > sovereignmind-installer-full_<Version>_amd64.run
sha256sum -c --ignore-missing SHA256SUMS.txt        # jede Zeile muss "OK" sein
sudo bash sovereignmind-installer-full_<Version>_amd64.run
```

Platz: ca. 13 GB Download, 7,7 GB (beim Update 13 GB) vorübergehend unter `/var/tmp`, ca. 26 GB installiert. Bei USB-Stick oder
eigenem Download-Server entfällt das Zusammenfügen. Optionen (`--accept-eula`, `--yes`, `--port`, `--no-lan`, `--full-backup` ...):
`bash <datei>.run --help`. Ein vorhandenes, älteres Ollama wird auf die mitgelieferte Version angehoben.

Danach ist die Oberfläche unter <http://localhost:5043> (mit eingerichtetem HTTPS-Namen auch unter `https://chat.example.de`) erreichbar. Beim ersten Aufruf führt die App
durch die Einrichtung (Lizenzschlüssel, Unternehmen, Admin-Passwort). Die Aktivierung läuft offline.

Vor der Installation ist der Endnutzer-Lizenzvertrag (EULA) zu bestätigen: Das Windows-Setup zeigt ihn als Lizenzseite,
das Linux-Bundle fragt per debconf (`sovereignmind/accept-eula`). Ohne Zustimmung wird nicht installiert. Automatisierte
Installation: Windows `/VERYSILENT` (gilt als Zustimmung), Linux `sudo bash <datei>.run --accept-eula --yes`.

Prüfsummen stehen in `SHA256SUMS.txt` neben den Downloads.

### Demo (ohne Lizenz, 24 Stunden)

Zum Ausprobieren gibt es keinen eigenen Installer: Installiert wird dieselbe Software wie bei der Vollversion; beim ersten Aufruf der Weboberfläche zeigt sie eine
Willkommensseite mit „Einrichten mit Lizenz“ und „Demo starten“. Die Demo läuft ohne Lizenzschlüssel höchstens 24 Stunden
(es zählt die Betriebszeit der Dienste, spätestens 24 Stunden nach dem ersten Start), löscht danach alle Daten (die App steht dann wieder auf der Willkommensseite und
lässt sich neu starten) und ist nur auf dem eigenen Rechner (`http://localhost:5043`) oder unter dem eingerichteten HTTPS-Namen (siehe „HTTPS“) erreichbar; unter jedem anderen Namen lehnt die Software sie ab.
Zugangsdaten: Admin mit Passwort `sovereignmind`, Nutzer `test@sovereignmind.de` mit Passwort `sovereignmind`. Eingeschränkt sind u. a. Nutzerverwaltung (ein Sitzplatz),
Lizenz, Schnittstellen, Dokumentenzahl und -größe (50 / 200 MB); Sicherungen und Recovery-Kit gibt es in der Demo nicht. Mit einem gültigen Lizenzschlüssel schaltet ein Admin
die laufende Demo zur Vollversion frei (Daten übernehmen oder neu beginnen).

Die Release-Dateien sind die der Vollversion (siehe oben). Der Endnutzer-Lizenzvertrag (Entwurf) wird in der App vor dem Start der Demo bestätigt.

### HTTPS

Der Installer bringt Caddy als HTTPS-Proxy mit (Windows: Setup-Seite „HTTPS“, Dienst `SovereignMindProxy`; Linux: Dienst `sovereignmind-proxy`, Einrichtung mit
`sudo sovereignmind https chat.example.de <modus>`). Chat (`chat.`) und Admin-Bereich (`admin.`) sind zwei getrennte Apps mit eigenem Namen hinter demselben Backend; die Demo startet auch dort.
Für andere Geräte muss der LAN-Zugriff an sein (`sovereignmind lan on`).

| Modus | Zertifikat | Hinweis |
| --- | --- | --- |
| `internal` (Standard) | interne CA | Root-Zertifikat einmalig je Gerät installieren, sonst Browserwarnung |
| `acme-http` | Let's Encrypt (HTTP-01) | öffentlicher Name, Port 80 aus dem Internet |
| `acme-dns` | Let's Encrypt (DNS-01) | eigene Domain, API-Schlüssel des DNS-Anbieters |
| `wildcard` | mitgeliefertes Wildcard-Zertifikat | keine Browserwarnung; wird erst mit gültiger Lizenz aktiv, bis dahin `internal`; Namen per Hosts-Zeile oder Router-DNS |

## Lizenzen

Die Setup-Seite „Lizenzhinweise“ und das Paket nennen alle Drittkomponenten. Qwen3-TTS (Apache License 2.0) liegt mit
Volltext und NOTICE im Installationsordner unter `licenses`. Die mitgelieferten Avatare sind KI-generierte Gesichter
(als „KI-generiert“ gekennzeichnet) und CC0-Illustrationen; es sind keine Bilder echter Personen.

## Bedienung nach der Installation

Windows: Die Tray-App „SovereignMind Control“ zeigt den Status, startet und stoppt die Dienste und öffnet die Weboberfläche.

Linux: Das Kommando `sovereignmind` (kein root nötig für `open`, `url`, `status`; die übrigen Befehle rufen selbst `sudo` auf):

| Befehl | Wirkung |
| --- | --- |
| `sovereignmind open` | Weboberfläche im Browser öffnen |
| `sovereignmind status` | Version, Adressen, Gesundheit, Dienste, Autostart, Ollama, Ordner |
| `sovereignmind url` | Adresse lokal und im Netzwerk |
| `sovereignmind start` / `stop` | alle Dienste starten bzw. stoppen |
| `sovereignmind restart [backend]` | alle Dienste oder nur das Backend neu starten |
| `sovereignmind autostart on\|off` | Start beim Hochfahren ein/aus |
| `sovereignmind logs [dienst] [--once]` | Protokoll (backend, postgres, qdrant, ingestion, voice, ollama) |
| `sovereignmind lan on\|off [IP]` | Zugriff aus dem Netzwerk ein/aus |
| `sovereignmind https NAME [MODUS]` | HTTPS-Name und Zertifikatsmodus einrichten, `https off` schaltet ab |
| `sovereignmind backup` | Sicherung jetzt starten |

Beispielausgabe von `sovereignmind status`:

```text
SovereignMind 2026.10.19.1606   ● läuft

Weboberfläche:           http://localhost:5043
Im Netzwerk:             http://192.168.0.42:5043
Gesundheit:              antwortet

Dienste:
  Datenbank (PostgreSQL)        ● aktiv
  Vektordatenbank (Qdrant)      ● aktiv
  Dokumentenimport              ● aktiv
  Sprache                       ● aktiv
  Ollama (KI-Modelle)           ● aktiv
  Backend und Weboberfläche     ● aktiv

Autostart:               an
Ollama:                  vorhandene Installation (0.40.4)
Geladene Modelle:        keine (werden bei der ersten Anfrage geladen)
Nächste Sicherung:       Tue 2026-10-20

Programme:               /opt/sovereignmind
Daten:                   /var/lib/sovereignmind
Konfiguration:           /etc/sovereignmind
Logs:                    sovereignmind logs   (oder journalctl -u 'sovereignmind-*')
```

## Betrieb

Update (neues Bundle bzw. neues Setup ausführen, Daten und Lizenz bleiben erhalten), Sicherung, Wiederherstellung
(Recovery-Kit) und Deinstallation beschreibt das Betreiber-Handbuch, das mit dem Anbieter-Support ausgeliefert wird.
Linux entfernen: `sudo apt remove sovereignmind` (mit Daten: `sudo apt purge sovereignmind`).
