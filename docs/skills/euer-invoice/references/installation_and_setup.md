# Installation, Einrichtung und Skill-Updates

Diese Referenz beschreibt die Installation der CLI, die Bereitstellung der Java- und
KoSIT-Toolchain, das Aussteller-Onboarding (`seller set`), die Wahl des Freigabemodus,
die Lizenzaktivierung sowie den vollständigen Austausch des Skills bei Updates.
Export, Sicherung und Prüfgrenzen stehen in der [Betriebsreferenz](operations.md).

Die CLI selbst installiert keine Agentendateien. Dateien außerhalb des Skill-Ordners
(wie `AGENTS.md`, Mandantendossiers, `invoice.db` oder die `buchhaltung.db` von `euer`)
bleiben bei Skill- und CLI-Updates unberührt.

## 1. Skill in Agentensystemen installieren und aktualisieren

1. Ermittle mit `euer-invoice doctor --format json` den absoluten Bundle-Pfad
   (`skill.bundle_path`) und die erwartete Skill-Version (`skill.expected_version`) der
   tatsächlich aufgerufenen CLI.
2. Kopiere den **gesamten** Ordner aus `skill.bundle_path` in den Skill-Ordner deines
   Agentensystems (z. B. unter `skills/euer-invoice/` bei Hermes, Claude Code oder OpenCode).
   Ersetze die alte Skill-Kopie vollständig, einschließlich des Unterordners `references/`.
   Ergänze oder bearbeite Dateien im Skill nicht lokal.
3. Lade den Skill neu (neue Sitzung oder Agent neu starten), lies `SKILL.md` und bei
   Bedarf diese Referenzdatei.
4. Bestätige erst danach die Version aus deiner tatsächlich verwendeten `SKILL.md`:
   ```bash
   euer-invoice setup --set skill.version "0.4.1" --format json
   ```
5. Prüfe `euer-invoice doctor --format json`: Das Feld `skill.status` muss `"current"` sein.

- **Hermes:** Lege den vollständigen Ordner im Hermes-Skill-Verzeichnis ab; beim Update
  den Ordner vollständig austauschen und eine neue Hermes-Sitzung beginnen.
- **Claude Code:** Lege den vollständigen Ordner als Skill unter dem von Claude Code
  verwendeten Skill-Verzeichnis ab; beim Update den Ordner vollständig austauschen.
- **OpenCode:** Lege ihn im OpenCode-Skill-Verzeichnis ab; beim Update den Ordner
  vollständig austauschen und die Sitzung neu starten.

Prüfe den konkreten Skill-Pfad deiner Agenteninstallation. Kannst du den Skill wegen
fehlender Rechte nicht ersetzen, informiere den Nutzer. Für einen einzelnen Aufruf
kann `--ignore-skill-version` die Sperre übergehen; das behebt den abweichenden Stand nicht.

## 2. CLI-Installation

### Empfohlene Installation (im selben Environment wie `euer` oder via pipx)

Damit Hermes oder dein Terminal die CLI ausführen können, installiere `euer-invoice`:

```bash
# Im selben Python-Environment wie euer (z. B. venv deines Buchhaltungsordners):
pip install euer-invoice

# Oder als eigenständiges globales Tool via pipx:
pipx install euer-invoice

# Für Entwickler aus dem lokalen Checkout:
pip install -e /pfad/zu/euer-buchhaltung/euer-invoice
```

Wird `euer-invoice` im selben Python-Environment wie `euer` installiert, registriert der
Entry-Point die identischen Befehle automatisch auch unter `euer invoice`.
Die Standalone-CLI `euer-invoice` benötigt keine initialisierte `euer`-Datenbank und ist
der empfohlene Standardaufruf für Rechnungs-Agenten. Der `euer`-Host verlangt dagegen
seine initialisierte Core-Datenbank und bestätigte Core-Skill-Version auch für Plugins.

Paketinstallation und Online-Toolchain-Setup setzen veröffentlichte Release-Artefakte
voraus. Toolchain-Archive und öffentliche Nutzertexte liegen unter
[GitHub Releases](https://github.com/curiousmarkus/euer-invoice-releases/releases).
Der Download benötigt keinen Zugang zum privaten Entwicklungsrepository. Nach Veröffentlichung der zugehörigen Tap-Formula installiert
`brew install curiousmarkus/euer/euer-invoice` Python-Paket, Java und Bundle gemeinsam;
ein lokaler Maven-Build ist bei Paketinstallationen nicht nötig. Kunden- und
Entwurfsverwaltung sind auch ohne Java nutzbar, Erzeugung und Prüfung benötigen die Toolchain.

## 3. Java- und KoSIT-Toolchain einrichten

Für die Erzeugung von standardkonformen **XRechnung 3.0.2 CII**-Dateien und die
unabhängige Prüfung gegen das offizielle KoSIT-Regelbundle wird eine Java-Laufzeit
(Java >= 21) und die Toolchain benötigt.

1. **Java 21 bereitstellen:**
   * macOS: `brew install openjdk@21` (oder `JAVA_HOME`/`PATH` setzen).
   * Bei abweichendem Pfad Umgebungsvariable `EUER_INVOICE_JAVA` auf das `java`-Binary setzen.
   * Prüfen mit `java -version` oder `euer-invoice doctor --format json`.

2. **Toolchain einrichten:**
   ```bash
   euer-invoice setup --install-toolchain --format json
   ```
   *Offline-Variante:* Falls kein Internetzugang besteht oder ein lokales Archiv verwendet wird:
   ```bash
   euer-invoice setup --install-toolchain --from-archive /pfad/zu/euer-invoice-toolchain-0.4.1.tar.gz --format json
   ```

3. **Suchreihenfolge der Toolchain:**
   Explizite Variable `EUER_INVOICE_TOOLCHAIN`, `.tools/` im aktuellen Arbeitsverzeichnis,
   `.tools/` im Checkout, `sys.prefix/libexec/toolchain` und zuletzt das
   Benutzerdatenverzeichnis (`~/.local/share/euer-invoice/toolchain` bzw.
   `$XDG_DATA_HOME/euer-invoice/toolchain`).
   Unter Windows liegt die Benutzertoolchain unter `%LOCALAPPDATA%/euer-invoice/toolchain`.

Setup prüft Archiv- und Dateihashes und behält vorhandene Freigabeschlüssel bei.
Fehlt Java, bleibt das geprüfte Bundle installiert; folge dem Installationshinweis
und wiederhole Setup. Setup richtet ein Schlüsselpaar neben der gewählten Datenbank
ein, aktiviert dadurch allein aber nicht `strict`. Der Chat-Ablauf benötigt diese
Schlüssel nicht.

## 4. Ersteinrichtung (Aussteller-Profil / Onboarding)

Bevor Rechnungsentwürfe angelegt oder ausgestellt werden können, muss das Aussteller-Profil
hinterlegt werden.

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
   - **Kleinunternehmer (§ 19 UStG):** Setze `"small_business": true` und lass `"vat_id"` leer (`null` oder weglassen).
   - **Nummernkreis:** `"invoice_number_prefix"` steuert das Format der fortlaufenden Rechnungsnummer (z. B. `"RE-{year}-"` ergibt `RE-2026-000001`).

2. **Ausstellerprofil speichern:**
   ```bash
   euer-invoice seller set --file seller.json --format json
   ```
   Das Profil wird in der Rechnungsdatenbank (`invoice.db`) gespeichert.

## 5. Freigabemodus und Konfiguration

`chat` ist der Standard: Der Agent zeigt die HTML-Vorschau und registriert die
Zustimmung des Nutzers zur konkreten Revision. Webserver und Freigabeschlüssel sind
für diesen Ablauf nicht erforderlich. Die Engine authentifiziert dabei selbst kein
Chatkonto. `strict` verlangt einen Ed25519-Nachweis des authentifizierten Gateways.

Für die Einrichtung eines Strict-Gateways:

```bash
euer-invoice setup --set approval.mode strict --format json
export EUER_INVOICE_APPROVAL_PUBLIC_KEY_FILE=/konfiguration/approval-public.der
euer-invoice doctor --format json
```

Der Betreiber hinterlegt den öffentlichen X.509-DER-Schlüssel des Gateways. Dessen
privater Schlüssel bleibt außerhalb des Agentenzugriffs. Das Gateway darf erst nach
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

Die Konfiguration liegt unter `~/.config/euer-invoice/config.toml`, auf Windows unter
`%APPDATA%/euer-invoice/config.toml`; `EUER_INVOICE_CONFIG` setzt einen anderen Pfad.

```toml
[skill]
version = "0.4.1"

[approval]
mode = "chat"

[export]
directory = "./rechnungen"
```

`[export]` ist optional. Werte lassen sich unter Erhalt vorhandener Sektionen setzen:

```bash
euer-invoice setup --set export.directory "./rechnungen" --format json
```

| Umgebungsvariable | Bedeutung |
| --- | --- |
| `EUER_INVOICE_CONFIG` | Abweichender TOML-Konfigurationspfad |
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
  Alle Befehle unterstützen `--format json|table`; als Agent verwende JSON.
- Zugehörige Original-XMLs und Validierungsberichte werden schreibgeschützt unter
  `<db-name>.artifacts/`.
- **Architektur-Grundsatz:** Die Rechnungsdatenbank bleibt strikt von der `buchhaltung.db`
  des Core-Systems `euer` getrennt. Der Agent bildet die Brücke:
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
informativ. `doctor`, `setup` und `license` funktionieren ohne Skill-Bestätigung.
Bewahre Schlüssel vertraulich auf und gib in Antworten nur den Status wieder.

Die Offline-Prüfung verwendet das Ed25519-Format `EUER-LIC-1` und denselben
Master-Public-Key wie `euer-datev`. Jede Lizenz benötigt ein ausdrückliches Produkt:
`invoice` für Rechnungen, `datev` für DATEV oder `pro` für beide Module. Der `tier`
allein schaltet kein Produkt frei. `years: [2026]` erlaubt Rechnungen mit
Rechnungsdatum 2026 unabhängig vom aktuellen Kalenderjahr. Auch Pro-Lizenzen
beachten ihre Jahre; nur Kanzlei-Lizenzen mit `years: null` sind jahresunabhängig.
Die Produktgrenzen gelten weiterhin. Eine reine DATEV-Lizenz genügt nicht.

Schlüssel aus diesen Quellen werden zusammengeführt:

- `EUER_INVOICE_LICENSE`, mehrere Schlüssel mit Semikolon getrennt.
- `~/.config/euer-invoice/invoice.lic`, überschreibbar mit `EUER_INVOICE_LICENSE_FILE`.
- `~/.config/euer/pro.lic` und `~/.config/euer/invoice.lic`.

`XDG_CONFIG_HOME` bzw. Windows `%APPDATA%` ändern die Basis der Lizenzdateipfade.
Aktivierung erhält bereits gespeicherte Jahreslizenzen. Kombinierte Schlüssel
(`product: pro`) werden in `euer/pro.lic` gespeichert und von beiden Modulen erkannt;
eine zweite Aktivierung ist unnötig. Separate Produktschlüssel sind ebenfalls möglich.

Die Toolchain enthält unter `legal/` die Lizenzen, Drittanbieterhinweise und
erforderlichen Quellenarchive ihrer Komponenten. Den Toolchain-Pfad zeigt
`doctor.toolchain.toolchain`. Diese Beigaben sind auch ohne kostenpflichtigen
Produktschlüssel zugänglich; die jeweiligen Open-Source-Rechte bleiben erhalten.
