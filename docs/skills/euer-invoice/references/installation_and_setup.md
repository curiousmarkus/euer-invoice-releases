# Installation, Einrichtung und Skill-Updates

Installiere CLI, Toolchain und Skill nach den folgenden Schritten und richte das
Ausstellerprofil ein. Belasse bei Skill-Updates Dateien außerhalb des Skill-Ordners
unverändert, insbesondere `AGENTS.md`, Mandantendossiers und Rechnungsdatenbanken.
Export, Sicherung und Prüfgrenzen stehen in der [Betriebsreferenz](operations.md).

## Start ohne bestehende Installation

Arbeite im Rechnungsordner des Nutzers und verwende vorhandene Datenbankpfade.

1. Prüfe Betriebssystem, vorhandenen Paketmanager, Python (mindestens 3.11), Java
   (mindestens 21) und eine gegebenenfalls vorhandene `euer-invoice`-Installation.
2. Installiere bei fehlender CLI zunächst das Paket nach Abschnitt 2, dann Java und
   Toolchain nach Abschnitt 3. Linux/Headless nutzt den pipx-Weg; Homebrew ist eine
   Alternative. Windows nutzt den nativen Python-/pipx-Weg unten.
   Installiere nur fehlende Voraussetzungen. Bei Problemen liefern
   `euer-invoice --help`, `euer-invoice setup --help` und
   `euer-invoice doctor --format table` die nächsten Schritte.
3. Ermittle anschließend mit `doctor` das Skill-Bundle, installiere den vollständigen
   Skill nach Abschnitt 1 und lies dessen `SKILL.md` vor der Versionsbestätigung.
4. Richte das Ausstellerprofil nach Abschnitt 4 aus bekannten Angaben ein und frage
   fehlende Fakten nach. Bereite auf Wunsch einen Entwurf zur Prüfung vor; die
   Ausstellung erfolgt erst nach ausdrücklicher Nutzerfreigabe.

Schließe eine vollständige Einrichtung ab, wenn `doctor.ready` wahr ist.
`ok` bewertet die technische Diagnose; es bestätigt allein keine abgeschlossene
Einrichtung. `readiness.installation`, `agent_skill`, `inspection`, `drafting` und
`invoice_creation` unterscheiden die Abläufe. Für einen reinen Prüfauftrag genügt
`readiness.inspection`; Ausstellerdaten sind dafür nicht nötig.
`ready` bestätigt die Grundvoraussetzungen, keine einzelne Rechnung oder Freigabe.
Die Rechnung selbst braucht weiterhin vollständige Angaben, erfolgreiche Prüfung,
passende Lizenz/Freikontingent und ausdrückliche Zustimmung zur konkreten Vorschau.

Doctor legt keine Datenbank an und migriert keine vorhandene. `database_check.status`
unterscheidet `missing`, `uninitialized`, `migration_required`, `current` und `error`.
Bei fehlendem bzw. älterem Bestand kann der Agent ausdrücklich initialisieren:

```bash
euer-invoice setup --init-database --format json
```

Verwende den vorhandenen bzw. bewusst gewählten Rechnungsordner und DB-Pfad.
`next_steps` nennt je Schritt `id`, `actor`, bei ausführbaren Aktionen `command`
als Argumentliste und bei fehlenden Ausstellerdaten `required_fields_missing`.
Bearbeite technische Schritte mit `actor: agent` selbstständig, verwende vorhandene
Mandantendaten und frage nur noch fehlende fachliche Fakten. Systemfreigaben oder
eine ausdrücklich gewählte Strict-Gateway-Konfiguration bleiben Betreiberaufgaben.
Prüfe Doctor nach einem erledigten Einrichtungsschritt erneut. Vor einer nötigen
Migration werden die Ausstellerdaten nicht als fehlend ausgewiesen, sondern erst
nachdem der aktuelle Bestand lesbar ist (`seller.checked`).
Bei fremden oder beschädigten Datenbanken erhält Doctor den Bestand und meldet
den Fehler. `license` ist ohne lesbaren aktuellen Rechnungsbestand `null`.
Eine unlesbare Lizenzdatei meldet Doctor separat als `license.error` mit
`LICENSE_READ_ERROR` und einem nächsten Schritt `resolve_license`. Datenbank-,
Aussteller- und Entwurfsbereitschaft bleiben erhalten; `ok` und `ready` sind bis
zur Klärung falsch. Prüfe Lizenzpfad, Dateiformat und Leserechte statt eines DB-Restores.
Falls eine Systemfreigabe oder ein Sitzungsneustart erforderlich ist, benenne den
konkreten Schritt und setze die Einrichtung anschließend fort.

## 1. Skill in Agentensystemen installieren und aktualisieren

1. Ermittle mit `euer-invoice skill path --format json` den absoluten Bundle-Pfad
   (`bundle_path`), die `skill_file` und die `version` der tatsächlich aufgerufenen CLI.
   `--format table` gibt nur den Ordnerpfad aus. Das funktioniert ohne Java,
   Rechnungsdatenbank und lesbare Konfiguration. Auch `doctor.skill.bundle_path`
   zeigt diesen Ordner.
2. Ermittle den Skill-Elternordner der aktiven Agentenumgebung und installiere das
   **gesamte** Bundle einschließlich `references/`:
   ```bash
   euer-invoice skill install --target /pfad/zu/skills --format json
   # Beispiel Hermes:
   euer-invoice skill install --target ~/.hermes/profiles/buchhaltung/skills --format json
   ```
   Das Ziel ist immer `<target>/euer-invoice/`. Bei Updates ersetze den alten
   Skill mit demselben Befehl und zusätzlich `--replace` vollständig. Dabei werden
   auch veraltete Dateien innerhalb dieses Skill-Ordners entfernt; benachbarte
   Skills und Dateien bleiben erhalten. Vorhandene Ziele werden ohne `--replace`
   abgewiesen. Symlinks als Skill-Ziel und Überschneidungen mit dem Bundle werden
   abgewiesen. Alternativ kopiere den gesamten Bundle-Ordner über den
   Installationsweg des Agentensystems. Pflege lokale Notizen außerhalb des Skills.
3. Lade den Skill in der aktiven Umgebung neu. Ist dafür ein Sitzungsneustart nötig,
   bitte den Nutzer darum. Lies anschließend die geladene `SKILL.md`.
4. Bestätige erst danach die Version aus der geladenen `SKILL.md`:
   ```bash
   euer-invoice setup --set skill.version "0.5.0" --format json
   ```
5. Prüfe `euer-invoice doctor --format json`: Das Feld `skill.status` muss `"current"` sein.

Kannst du den Skill wegen fehlender Rechte nicht ersetzen, informiere den Nutzer.
Für einen einzelnen Aufruf kann `--ignore-skill-version` die Sperre übergehen;
das behebt den abweichenden Stand nicht.

## 2. CLI-Installation

### Linux / Headless ohne Homebrew

Voraussetzungen: Python >=3.11, pipx und Java >=21 im `PATH`. Installiere fehlende
Voraussetzungen über den Paketmanager der Distribution; ein Headless-JRE genügt.
Debian 13 bietet beispielsweise `pipx` und
[`openjdk-21-jre-headless`](https://packages.debian.org/trixie/openjdk-21-jre-headless).
Prüfe `python3 --version`, `pipx --version` und `java -version` vor dem Setup.
Bei älteren Distributionen unterscheidet sich die Java-Paketquelle.

```bash
pipx install euer-invoice
euer-invoice setup --install-toolchain --format json
euer-invoice doctor --format table
```

Setup lädt die passende Toolchain automatisch herunter. Der Standardweg braucht
keine `EUER_INVOICE_JAVA`-/`EUER_INVOICE_TOOLCHAIN`-Variablen und keinen Wrapper.
Falls pipx den Befehl nicht sichtbar macht, führe `pipx ensurepath` aus und starte
bei Bedarf die Agentensitzung neu; Cronjobs verwenden den absoluten CLI-Pfad.
Installiere danach den Skill nach Abschnitt 1 und das Ausstellerprofil nach
Abschnitt 4. Für Docker siehe [Container-Beispiel](container.md).

### macOS und Homebrew als Alternative

Auch macOS kann den pipx-Weg verwenden. Mit vorhandenem Homebrew auf macOS/Linux:

```bash
brew tap curiousmarkus/euer
brew install curiousmarkus/euer/euer-invoice
```

Falls Homebrew das Laden mit `untrusted tap` verweigert, gib gezielt diese Formula
frei und wiederhole die Installation:

```bash
brew trust --formula curiousmarkus/euer/euer-invoice
brew install curiousmarkus/euer/euer-invoice
```

Die Formula installiert Python, Java und die gepinnte Toolchain gemeinsam.

### Windows nativ (PowerShell)

WSL ist für `euer-invoice` nicht erforderlich. Verwende eine vorhandene Python-
Installation ab 3.11 und Java ab 21. Fehlt Python, nutze den
[offiziellen Windows-Installer](https://www.python.org/downloads/windows/).
Die folgenden Befehle setzen den Python-Launcher `py` voraus; alternativ verwende
den absoluten Pfad der gewählten Python-Installation.

```powershell
py --version
py -m pip install --user pipx
py -m pipx ensurepath
py -m pipx install euer-invoice
```

Nach `ensurepath` kann ein neues Terminal bzw. eine neue Agentensitzung nötig sein,
damit `euer-invoice` gefunden wird. Details: [Python Packaging Guide](https://packaging.python.org/en/latest/guides/installing-stand-alone-command-line-tools/).
Prüfe vor weiteren Schritten `euer-invoice --version` in der laufenden Shell.

Falls Java fehlt, installiere es beispielsweise mit vorhandenem WinGet:

```powershell
winget install --id EclipseAdoptium.Temurin.21.JDK --exact
```

Öffne nach der Installation bei Bedarf eine neue Sitzung und prüfe `java -version`.
Ohne WinGet ist der [Temurin-Installer](https://adoptium.net/installation) eine
Alternative. Der Java-Weg ist auch im [Microsoft-Leitfaden](https://learn.microsoft.com/windows/dev-environment/java/) beschrieben.
Danach folge Abschnitt 3 für die Toolchain und Abschnitt 1 für den Skill.
Ein Agent, der bereits in WSL läuft, verwendet stattdessen den Linux-Weg innerhalb
derselben Umgebung; mische Windows- und WSL-Installationen nicht.

### Bestehendes `euer`-Environment und Entwicklung

Installiere die CLI bei einem bestehenden Python-Environment in derselben Umgebung:

```bash
# Im selben Python-Environment wie euer:
pip install euer-invoice

# Aus einem lokalen Checkout:
pip install -e /pfad/zu/euer-buchhaltung/euer-invoice
```

Wird `euer-invoice` im selben Python-Environment wie `euer` installiert, registriert der
Entry-Point die identischen Befehle automatisch auch unter `euer invoice`.
Verwende standardmäßig die Standalone-CLI `euer-invoice`; sie benötigt keine
initialisierte `euer`-Datenbank. Der `euer`-Host verlangt dagegen
seine initialisierte Core-Datenbank und bestätigte Core-Skill-Version auch für Plugins.

Paketinstallation und Online-Toolchain-Setup setzen veröffentlichte Release-Artefakte
voraus. Toolchain-Archive und öffentliche Nutzertexte liegen unter
[GitHub Releases](https://github.com/curiousmarkus/euer-invoice-releases/releases).
Nutze die veröffentlichten Pakete ohne Zugriff auf das private Entwicklungsrepository
oder lokalen Maven-Build. Kunden- und Entwurfsverwaltung sind auch ohne Java nutzbar;
Erzeugung und Prüfung benötigen die Toolchain.

## 3. Java- und KoSIT-Toolchain einrichten

Für die Erzeugung von standardkonformen **XRechnung 3.0.2 CII**-Dateien und die
unabhängige Prüfung gegen das offizielle KoSIT-Regelbundle wird eine Java-Laufzeit
(Java >= 21) und die Toolchain benötigt.

1. **Java 21 bereitstellen:**
   * macOS: `brew install openjdk@21` und dessen `bin`-Ordner in den `PATH` aufnehmen.
   * Java-Suche: `EUER_INVOICE_JAVA`, dann `[toolchain] java_bin`, dann ein JDK im
     Toolchain-Ordner, dann `java` im `PATH`. `JAVA_HOME` wird nicht direkt gelesen.
   * Bei abweichendem Pfad dauerhaft konfigurieren:
     `euer-invoice setup --set toolchain.java_bin /absoluter/pfad/zu/java`.
   * Prüfen mit `java -version` oder `euer-invoice doctor --format json`.

2. **Toolchain einrichten:**
   ```bash
   euer-invoice setup --install-toolchain --format json
   ```
   Setup verwendet eine bereits durch diese CLI eingerichtete, zur aktuellen
   Release-Identität passende und weiterhin hashgeprüfte Toolchain erneut
   (`reused: true`). Sonst lädt es das passende Archiv von GitHub Releases.
   Fehlende Belege oder geänderte Dateien lösen eine Neuinstallation aus.
   Mit `--force` lässt sich diese ausdrücklich erzwingen. Ein explizites
   `--from-archive` wird immer geprüft und installiert.
   Es gibt einen aktiven Toolchain-Ordner; parallele CLI-Versionen werden nicht unterstützt.
   *Offline-Variante:* Falls kein Internetzugang besteht oder ein lokales Archiv verwendet wird:
   ```bash
   euer-invoice setup --install-toolchain --from-archive /pfad/zu/euer-invoice-toolchain-0.5.0.tar.gz --format json
   ```

3. **Suchreihenfolge der Toolchain:**
   Explizite Variable `EUER_INVOICE_TOOLCHAIN`, `[toolchain] path` aus der Config,
   `.tools/` im aktuellen Arbeitsverzeichnis,
   `.tools/` im Checkout, `sys.prefix/libexec/toolchain` und zuletzt das
   Benutzerdatenverzeichnis (`~/.local/share/euer-invoice/toolchain` bzw.
   `$XDG_DATA_HOME/euer-invoice/toolchain`).
   Unter Windows liegt die Benutzertoolchain unter `%LOCALAPPDATA%/euer-invoice/toolchain`.

Das Installationsziel von Setup ist `EUER_INVOICE_TOOLCHAIN`, dann `[toolchain] path`,
sonst das Benutzerdatenverzeichnis. Die `.tools/`-Suchpfade werden dafür nicht verwendet.
Konfigurierte und explizite ENV-Pfade unterstützen `~`. Relative TOML-Pfade beziehen
sich auf den Ordner der Konfigurationsdatei und bleiben beim Arbeitsverzeichniswechsel
gleich; relative ENV-Pfade beziehen sich auf das aktuelle Arbeitsverzeichnis.
Explizite ungültige Pfade werden gemeldet; sie lösen keinen stillen Fallback aus.

`doctor --format json` zeigt `config_path` und in `toolchain` den wirksamen Pfad,
`toolchain_source` sowie `java.binary`, `java.source` und die erkannte Hauptversion
`java.version`. Bei Laufzeitproblemen bleibt der Fehlercode `GENERATOR_UNAVAILABLE`;
`error.reason` unterscheidet `TOOLCHAIN_MISSING`, `TOOLCHAIN_INVALID`, `JAVA_NOT_FOUND`,
`JAVA_UNAVAILABLE`, `JAVA_VERSION_UNKNOWN` und `JAVA_TOO_OLD`. `error.remediation`
nennt den nächsten Schritt. Bei mehreren Problemen bleibt die Java-Diagnose zusätzlich
in `java.reason` sichtbar. Doctor lädt keine Dateien herunter.
Für Menschen zeigt `doctor --format table` die ASCII-Statusmarker `[OK]` und
`[FEHLER]` statt Unicode-Häkchen; JSON bleibt Standard.

Toolchain-Setup verändert ausschließlich die Toolchain und erzeugt weder
Rechnungsdatenbank noch Freigabeschlüssel. Fehlt Java, bleibt das geprüfte Bundle
installiert; stelle Java bereit und wiederhole Setup ohne erneuten Download.
Vorhandene Schlüssel bleiben erhalten.

Nur für eine ausdrücklich betriebene lokale Web-Vorschau lassen sich Schlüssel
separat einrichten:

```bash
euer-invoice setup --init-approval-keys --db /pfad/zu/invoice.db --format json
```

Das legt das Paar neben dem angegebenen DB-Pfad bzw. an den über
`EUER_INVOICE_APPROVAL_PRIVATE_KEY_FILE`/`EUER_INVOICE_APPROVAL_PUBLIC_KEY_FILE`
gewählten Pfaden an; die Datenbank wird dabei nicht erstellt. Ein vorhandenes
vollständiges Paar bleibt unverändert. Ein unvollständiges Paar wird als Fehler
behandelt. Der Befehl setzt keinen Freigabemodus. Der normale Chat-Ablauf benötigt
kein Schlüsselpaar; bei einem Strict-Gateway verbleibt dessen privater Schlüssel
außerhalb des Agentenzugriffs.

## 4. Ersteinrichtung (Aussteller-Profil / Onboarding)

Hinterlege vor dem ersten Rechnungsentwurf das Ausstellerprofil aus bekannten Angaben
und kläre fehlende Fakten mit dem Nutzer.

1. **Ausstellerdaten zusammenstellen (`seller.json`):**
   ```json
   {
     "legal_name": "Musterfirma GmbH",
     "street": "Musterstraße 12",
     "postal_code": "12345",
     "city": "Musterstadt",
     "country_code": "DE",
     "electronic_address": "buchhaltung@example.de",
     "electronic_address_scheme": "EM",
     "contact_name": "Max Mustermann",
     "contact_email": "rechnungen@example.de",
     "contact_phone": "+49 123 456789",
     "tax_number": "12/345/67890",
     "vat_id": "DE123456789",
     "small_business": false,
     "iban": "DE12500105170648489890",
     "payment_account_name": "Musterfirma GmbH",
     "default_payment_terms_days": 14,
     "invoice_number_prefix": "RE-{year}-"
   }
   ```
   - **Kleinunternehmer (§ 19 UStG):** Setze `"small_business": true` und hinterlege `"tax_exemption_reason"`, z. B. `"Steuerbefreiung für Kleinunternehmer gemäß § 19 UStG."`. Eine vorhandene USt-IdNr. darf in `"vat_id"` stehen und erfüllt die Verkäuferidentifikation nach BR-CO-26. Ohne USt-IdNr. sind im unterstützten Umfang eine Steuernummer und zusätzlich eine tatsächliche Verkäuferkennung (BT-29) oder Registerkennung (BT-30) nötig. Eine Steuernummer allein genügt für das erzeugte XRechnung-Profil nicht.
   - **Weitergabe der Steuerkennungen:** Sind `vat_id` und `tax_number` gesetzt, werden beide ins XML übernommen. Wer bei vorhandener USt-IdNr. die Steuernummer nicht weitergeben möchte, lässt `tax_number` weg oder entfernt sie ausdrücklich aus dem Profil. Bestehende Rechnungssnapshots bleiben unverändert; ein bearbeitbarer Entwurf übernimmt das Profil erst durch ein ausdrückliches Update mit `--refresh-seller`, das Prüfung und Freigabe invalidiert.
   - **Kundenspezifische Lieferantenkennung:** Pflege eine vom Empfänger vergebene Kreditorennummer als `seller_identifier` beim jeweiligen Kontakt. Das globale `seller.identifier` bleibt für empfängerübergreifende Verkäuferkennungen und Bestandsdaten erhalten. Kläre die tatsächliche Herkunft einer Bestandskennung, bevor sie einem Kontakt zugeordnet wird. Details und Priorität: [Identifikatoren](identifiers.md).
   - **Nummernkreis:** `"invoice_number_prefix"` steuert das Format der fortlaufenden Rechnungsnummer (z. B. `"RE-{year}-"` ergibt `RE-2026-000001`).

2. **Ausstellerprofil speichern:**
   ```bash
   euer-invoice seller set --file seller.json --format json
   ```
   Das Profil wird in der Rechnungsdatenbank (`invoice.db`) gespeichert.

## 5. Freigabemodus und Konfiguration

Im Standardmodus `chat` zeige die HTML-Vorschau und registriere die
Zustimmung des Nutzers zur konkreten Revision. Webserver und Freigabeschlüssel sind
für diesen Ablauf nicht erforderlich. Die Engine authentifiziert dabei selbst kein
Chatkonto. `strict` verlangt einen Ed25519-Nachweis des authentifizierten Gateways.

Für die Einrichtung eines Strict-Gateways:

```bash
euer-invoice setup --set approval.mode strict --format json
export EUER_INVOICE_APPROVAL_PUBLIC_KEY_FILE=/konfiguration/approval-public.der
euer-invoice doctor --format json
```

Der Betreiber hinterlegt den öffentlichen X.509-DER-Schlüssel des Gateways mit
absolutem Pfad. Doctor und Verifier entfernen umgebenden Whitespace und
lösen relative Pfade vom Arbeitsverzeichnis aus auf; ein wörtliches `~` im ENV-Wert
wird nicht zum Home-Verzeichnis expandiert. Der private Schlüssel bleibt außerhalb
des Agentenzugriffs. Das Gateway darf erst nach
authentifizierter Nutzerbestätigung der gezeigten Revision signieren. Betrieb und
Prüfung des Nachweises beschreibt die [Betriebsreferenz](operations.md#signierte-freigabe-und-optionale-web-vorschau).

Die Auflösung des Modus erfolgt in dieser Reihenfolge:

1. Explizites `EUER_INVOICE_APPROVAL_MODE` (`chat` oder `strict`).
2. Automatisch `strict` bei gesetztem `EUER_INVOICE_APPROVAL_PUBLIC_KEY_FILE`.
3. `[approval] mode` aus der Konfiguration.
4. Default `chat`.

Damit übersteuert `EUER_INVOICE_APPROVAL_MODE=chat` ausdrücklich auch einen gesetzten
Public-Key-Pfad. `doctor.approval_mode` zeigt den wirksamen Modus. Ungültige Werte
oder beschädigte Konfigurationen führen zu einem Fehler.

Die Konfiguration liegt unter `$XDG_CONFIG_HOME/euer-invoice/config.toml` bzw.
`~/.config/euer-invoice/config.toml`, auf Windows unter
`%APPDATA%/euer-invoice/config.toml`; `EUER_INVOICE_CONFIG` setzt einen anderen Pfad.
Bei einem Upgrade bleibt eine vorhandene `~/.config/euer-invoice/config.toml` aktiv,
solange im abweichenden XDG-Verzeichnis keine Config liegt. So bleiben bisherige
Freigabe-/Skill-Einstellungen erhalten. Verschiebe die vorhandene Datei ausdrücklich,
wenn du zu XDG wechseln möchtest; `doctor.config_path` zeigt den verwendeten Pfad.

```toml
[skill]
version = "0.5.0"

[approval]
mode = "chat"

[export]
directory = "./rechnungen"

# Optional für abweichende Laufzeitpfade; ENV hat Vorrang.
[toolchain]
path = "~/.local/share/euer-invoice/toolchain"
java_bin = "/usr/bin/java"

# Optional: sonst .euer-invoice-backups/ beim DB-Verzeichnis
[backups]
directory = "/pfad/zur/sicherung"
max_keep = 10
```

`[export]`, `[toolchain]` und `[backups]` sind optional. Werte lassen sich unter Erhalt vorhandener Sektionen setzen:

```bash
euer-invoice setup --set export.directory "./rechnungen" --format json
euer-invoice setup --set toolchain.java_bin "/usr/bin/java" --format json
```

| Umgebungsvariable | Bedeutung |
| --- | --- |
| `EUER_INVOICE_CONFIG` | Abweichender TOML-Konfigurationspfad |
| `EUER_INVOICE_BACKUP_DIR` | Sicherungsverzeichnis vor `[backups] directory`; siehe Betriebsreferenz |
| `XDG_CONFIG_HOME` | Config-Basis auf macOS/Linux, sofern kein `EUER_INVOICE_CONFIG` gesetzt ist |
| `EUER_INVOICE_DB` | Datenbank, sofern kein `--db` angegeben ist |
| `EUER_INVOICE_APPROVAL_MODE` | Expliziter Modus-Override mit höchster Priorität |
| `EUER_INVOICE_APPROVAL_PUBLIC_KEY_FILE` | Öffentlicher DER-Schlüssel; aktiviert ohne Modus-Override `strict` |
| `EUER_INVOICE_APPROVAL_PRIVATE_KEY_FILE` | Privater Schlüssel für Setup bzw. den optionalen Vorschau-Server |
| `EUER_INVOICE_PREVIEW_BASE_URL` | Basis-URL eines ausdrücklich betriebenen Vorschau-Servers |
| `EUER_INVOICE_EXPORT_DIR` | Exportverzeichnis; hat Vorrang vor `[export] directory` |
| `EUER_INVOICE_TOOLCHAIN` | Expliziter Toolchain-Pfad |
| `EUER_INVOICE_JAVA` | Pfad zur Java-Laufzeit ab Version 21 |
| `EUER_INVOICE_LICENSE` | Lizenzschlüssel, mehrere mit Semikolon getrennt |
| `EUER_INVOICE_LICENSE_FILE` | Abweichende Modul-Lizenzdatei |

## 6. Datenbank und Trennung von `euer`

- `euer-invoice` speichert Rechnungen, Kontakte und Zähler in einer eigenen SQLite-Datenbank:
  Ein explizites `--db PFAD` hat Vorrang vor `EUER_INVOICE_DB`, sonst gilt `./invoice.db`.
  Verwende `--format json` für die Auswertung der Antworten.
- Zugehörige Original-XMLs und Validierungsberichte liegen schreibgeschützt unter
  `<db-name>.artifacts/`.
- **Architektur-Grundsatz:** Die Rechnungsdatenbank bleibt strikt von der `buchhaltung.db`
  des Core-Systems `euer` getrennt. Verwende je nach Vorgang:
  - Bei Rechnungserstellung: `euer-invoice`
  - Bei Zahlungseingang: Betriebseinnahme in `euer` buchen + Zahlung in `euer-invoice` via
    `euer-invoice payment record --id <ID> --amount ... --date ...` dokumentieren.

## 7. Freikontingent und Lizenzaktivierung

Entwürfe, Vorschauen, Empfangsprüfungen, Stornierungen und Zahlungen sind unbegrenzt
frei. Ohne passende Lizenz sind drei Ausstellungen je Kalendermonat und lokaler
Rechnungsdatenbank möglich. Der Monat des Rechnungsdatums ist maßgeblich;
stornierte Originale zählen weiter. Die vierte Ausstellung scheitert vor der
Nummernvergabe mit `LICENSE_REQUIRED`. Ungenutzte Plätze werden nicht übertragen.
Idempotente Wiederholungen bleiben auch bei ausgeschöpftem Kontingent möglich.

```bash
euer-invoice license status --year 2026 --month 11 --format json
euer-invoice license activate SCHLUESSEL --format json
```

Ohne Zeitangabe gilt der aktuelle Monat. `doctor` und `license status` zeigen
`period`, `used_in_month`, `issued_this_month` und `remaining`; Jahreszähler sind nur
informativ. `--year` und `--month` wählen nur den Zeitraum des Freikontingents,
keine Lizenzjahre. `doctor`, `setup`, `skill`, `backup` und `license` funktionieren ohne Skill-Bestätigung.
Bewahre Schlüssel vertraulich auf und gib in Antworten nur den Status wieder.

Die Offline-Prüfung verwendet das Ed25519-Format `EUER-LIC-1` und denselben
Master-Public-Key wie `euer-datev`. Jede Lizenz benötigt ein ausdrückliches Produkt:
`invoice` für Rechnungen, `datev` für DATEV oder `pro` für beide Module. Der `tier`
allein schaltet kein Produkt frei. Der Invoice-Einmalkauf verwendet `product: invoice`
und `years: null`: alle Rechnungsjahre sind zeitlich unbefristet freigeschaltet,
auch 2027 und später. Für `product: datev` oder `pro` erfordert `years: null`
`tier: kanzlei`. Bei Pro-Schlüsseln begrenzt die Jahresliste ausschließlich DATEV;
Invoice bleibt für alle Rechnungsjahre freigeschaltet. Ein Invoice-Einmalkauf
schaltet DATEV nicht frei; die DATEV-Lizenz
gilt für das vereinbarte Buchungsjahr. Pakete mit Invoice für alle Jahre
und DATEV für ein einzelnes Jahr verwenden zwei getrennte Produktschlüssel.

Schlüssel aus diesen Quellen werden zusammengeführt:

- `EUER_INVOICE_LICENSE`, mehrere Schlüssel mit Semikolon getrennt.
- `~/.config/euer-invoice/invoice.lic`, überschreibbar mit `EUER_INVOICE_LICENSE_FILE`.
- `~/.config/euer/pro.lic` und `~/.config/euer/invoice.lic`.

`XDG_CONFIG_HOME` bzw. Windows `%APPDATA%` ändern die Basis der Lizenzdateipfade.
Aktivierung erhält andere gespeicherte Produktschlüssel. Kombinierte Schlüssel
(`product: pro`) werden in `euer/pro.lic` gespeichert und von beiden Modulen erkannt;
eine zweite Aktivierung ist unnötig. Separate Produktschlüssel sind ebenfalls möglich.

Die Toolchain enthält unter `legal/` die Lizenzen, Drittanbieterhinweise und
erforderlichen Quellenarchive ihrer Komponenten. Den Toolchain-Pfad zeigt
`doctor.toolchain.toolchain`. Diese Beigaben sind auch ohne kostenpflichtigen
Produktschlüssel zugänglich; die jeweiligen Open-Source-Rechte bleiben erhalten.
