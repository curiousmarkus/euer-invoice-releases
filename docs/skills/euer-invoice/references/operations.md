# Betrieb: Vorschau, Export, Sicherung und Empfangsprüfung

Konfigurationswerte stehen in der [Setup-Referenz](installation_and_setup.md).

## Vorschau und Kundenexport

`draft preview` speichert die HTML-Vorschau immer intern. `preview_html_path` enthält
ihren absoluten Pfad; `show --id ID` liefert ihn erneut. Mit `--html-out PFAD` entsteht
zusätzlich eine atomar geschriebene Kopie. Die Exportantwort nennt diese Kopie,
`show` weiterhin das interne Original. Nach Änderungen ist eine neue Vorschau
und eine neue Nutzerfreigabe der Revision nötig. Die statische Ansicht enthält
keine Freigabe- oder Server-Buttons.

Ausgestellte XML-Originale und Berichte liegen inhaltsadressiert und schreibgeschützt
unter `<db-name>.artifacts/`; `artifact.file_path` verweist auf das Original.
Kundenkopien sind zusätzliche Dateien, die sich ohne Änderung des Originals versenden
lassen. Ist `[export] directory` oder `EUER_INVOICE_EXPORT_DIR` gesetzt, erzeugt `issue`
die Kopie automatisch und liefert `artifact.exported_path`.
Ein relativer Standard-Exportordner wird am Verzeichnis der Datenbank aufgelöst.
Das Namensmuster ist `{nummer}_{aussteller}_{kunde}.xml`; Namen werden für Dateipfade
bereinigt und gekürzt.

Exportiere eine Kundenkopie mit:

```bash
euer-invoice export --id ID --out rechnung.xml --format json
# Finale Empfängerfassung und Mailentwurf aus derselben XML:
euer-invoice export --id ID --out rechnung.xml --html-out rechnung.html --email-out begleitmail.txt --format json
# Mit konfiguriertem Standard-Exportordner genügt:
euer-invoice export --id ID --format json
```

Die Antwort enthält `exported_path`. Versende bei beauftragtem Rechnungsversand die
Kundenkopie. Die Originalablage ist kein Ersatz für eine Sicherung oder eine
organisatorische Archivierung.

`recipient_html_path` enthält die druckbare finale Lesefassung;
`email_draft_path` den Mailtext. `email_draft` liefert Betreff und Nachricht auch als JSON.
Für ein separates Lese-PDF oder die E-Mail-Alternative folge dem Ablauf in
[Versandpaket und PDF](delivery.md).
Die einzelnen Exportdateien werden atomar geschrieben; bei einem Speicherfehler kann
bereits ein Teil des Pakets vorliegen. Wiederhole den Export nach Behebung des Fehlers.

## Wiederaufnahme nach Abbruch

Nach Timeout oder Prozessabbruch denselben `issue`-Auftrag mit demselben
Idempotenzschlüssel wiederholen. Nummer, verbrauchte Freigabe und Abschlusszustand
werden gemeinsam erst nach dauerhaft gespeicherten Originalen committed.
`show` und Wiederholungen prüfen die Integrität referenzierter Originale und Berichte.
Bei `ARTIFACT_INTEGRITY` die Ursache klären und aus einer vollständigen Sicherung
wiederherstellen; Originale nicht von Hand verändern.

Bei `CONFLICT_REVISION` den aktuellen Entwurf lesen und erneut abstimmen.
`IDEMPOTENCY_CONFLICT` verlangt eine Klärung der Auftragszuordnung.

## Sicherung und Umzug

Die CLI sichert Datenbank und referenzierte Rechnungsdateien gemeinsam in einer
ZIP-Datei. Vor der ersten schreibenden Transaktion eines Tages wird automatisch
lokal gesichert; vor einer Migration zusätzlich der aktuelle Altbestand. Das gilt
auch für Änderungen über den Vorschau-Server. Scheitert die nötige Sicherung,
bleibt die Datenänderung aus. Doctor, Empfangsprüfung und Lesezugriffe lösen keine
Sicherung aus. Ein neuer leerer Bestand braucht noch kein Vorab-Backup.

### Explizites Backup und Prüfung

```bash
euer-invoice --db invoice.db backup create --out sicherung.zip --format json
euer-invoice backup verify --file sicherung.zip --format json
euer-invoice --db invoice.db backup check-live --format json
```

`create` erstellt per SQLite-Online-Backup einen konsistenten Snapshot, prüft
Datenbankintegrität, Fremdschlüssel und Rechnungs-Snapshot-Hashes und nimmt die
referenzierten Original-XMLs, Prüfberichte und gespeicherten Vorschauen hinzu.
Die Dateihashes müssen stimmen. Fehlende oder veränderte referenzierte Dateien
führen zu `ARTIFACT_INTEGRITY`; ein falscher Rechnungs-Snapshot zu
`SNAPSHOT_INTEGRITY`. Erst das vollständig geprüfte Archiv wird veröffentlicht.
Ein bestehendes Archiv wird auch mit explizitem `--out` nicht überschrieben.
Mit explizitem Ziel funktioniert die Sicherung auch bei defekter App-Konfiguration.

`verify` prüft das Archiv isoliert und verändert keinen Rechnungsbestand.
`check-live` prüft den gewählten aktuellen Bestand ohne ZIP-Erstellung. Doctor
weist die Integrität zusätzlich unter `artifacts_ok` aus; `null` bedeutet, dass
kein aktueller lesbarer Bestand geprüft werden konnte. Bei einem Fehler bleibt
`ready` falsch und `next_steps` enthält einen Klärungs-/Wiederherstellungsschritt.
Alle `backup`-Befehle funktionieren ohne Java und ohne Skill-Versionsbestätigung.

Antworten enthalten je nach Befehl `backup_path`, `backup_sha256`, `created_at`,
`counts` sowie `verified` oder `restored`. `managed: true` kennzeichnet Archive,
die der automatischen Rotation unterliegen; explizite Ausgabedateien sind `false`. Das ZIP enthält `invoice.db`,
`manifest.json` und ausschließlich die referenzierten Dateien unter `artifacts/`.
Unreferenzierte Altdateien, Kundenexporte, Konfiguration, Lizenzdateien und private
Freigabeschlüssel gehören nicht ins Bundle. Verwahre die nötige Konfiguration und
Schlüssel zusätzlich gemäß ihrer Zugriffsgrenze; ein Backup enthält vertrauliche
Rechnungsdaten und Freigabehistorie.

### Freier Zielbestand für Restore

```bash
euer-invoice --db wiederhergestellt/invoice.db backup restore --file sicherung.zip --format json
euer-invoice --db wiederhergestellt/invoice.db backup check-live --format json
euer-invoice --db wiederhergestellt/invoice.db show --id RECHNUNGS_ID --format json
```

Wähle einen neuen Zielbestand. Bereits vorhandene DB-, Artefakt- oder Journaldateien
werden mit `RESTORE_TARGET_EXISTS` geschützt; ein Überschreiben mit `--force` ist
nicht enthalten. Die vollständige Vorabprüfung erfolgt, bevor das Zielverzeichnis
angelegt wird. Unterstützte ältere Schemata werden im isolierten Staging migriert.
Originaldateien erhalten Schreibschutz; die Datenbank wird zuletzt veröffentlicht.
Ein Prozessabbruch kann einen Artefaktordner ohne DB zurücklassen: Erhalte den
Teilstand und nutze für einen erneuten Restore einen weiteren freien Zielort.

Prüfe danach bekannte Rechnungen, Zahlungs-/Stornodaten und die benötigte
Konfiguration. Relative Originalreferenzen und historische absolute Vorschau- bzw.
Fehlerberichtpfade werden am neuen Bestand aufgelöst. Ein `BACKUP_RESTORED`-Eintrag
hält die Herkunft fest. Nimm anschließend bewusst nur einen aktiven Rechnungsbestand
in Betrieb. Kläre bereits nach dem Backup ausgestellte Rechnungen, bevor du die
Nummernfolge fortsetzt; ein Restore kennt spätere Vorgänge nicht.

### Standardablage und Rotation

Standard ist `.euer-invoice-backups/` im Verzeichnis der Datenbank. Diese Ablage
bleibt bei wechselnden Agentensitzungen gleich und liegt beim Docker-Beispiel im
Daten-Volume. Dateinamen trennen Bestände über ihren absoluten Datenbankpfad.
Automatische und ohne `--out` erzeugte Backups rotieren je Datenbank auf die
neuesten zehn Archive; explizite Ausgabedateien werden nicht automatisch gelöscht.
Vorhandene tägliche Sicherungen werden geprüft und bei Beschädigung neu erstellt.
Die Tagesgrenze entspricht dem lokalen Datum des ausführenden Prozesses.

Bei vorhandenem anderen Sicherungsziel kann der Agent es dauerhaft hinterlegen:

```bash
euer-invoice setup --set backups.directory /pfad/zum/sicherungsordner --format json
euer-invoice setup --set backups.max_keep 10 --format json
```

Alternativ setzt `EUER_INVOICE_BACKUP_DIR` das Verzeichnis mit höchster Priorität.
Relative TOML-Pfade beziehen sich auf den Config-Ordner, relative ENV-Pfade auf das
Arbeitsverzeichnis. Erfrage hierfür keine Routineentscheidungen, wenn Standardablage
oder ein bekanntes Sicherungsziel passen. Ein volles oder unzugängliches Ziel ist
als technisches Problem zu beheben; schalte die Sicherung nicht stillschweigend ab.
Bei `BACKUP_BUSY` nach Ende des konkurrierenden Vorgangs erneut versuchen.

Eine lokale Sicherung schützt nicht vor Geräteverlust. Die tägliche Sicherung
enthält den Stand vor der ersten Änderung, nicht automatisch alle später am Tag
ausgestellten Rechnungen. Erstelle für einen aktuellen Stand explizit ein Backup
und nutze vorhandene getrennte Sicherungsorte des Nutzers. Grenzen: höchstens
500 MB Archiv-/entpackte Gesamtdaten, 10.000 Einträge und 1 MB Manifest. Ungültige
Archive oder überschrittene Grenzen werden mit `BACKUP_INVALID` abgewiesen.

## Signierte Freigabe und optionale Web-Vorschau

Im Strict-Modus liefert das authentifizierte Gateway nach ausdrücklicher
Nutzerzustimmung den Nachweis für `draft approve --proof approval.json`.
ID, Revision, Snapshot-Hash und Akteur müssen identisch sein. Die Registrierung
verlangt eine gültige Ed25519-Signatur, eine unbenutzte Nonce und einen
zeitzonenbewussten UTC-Zeitstempel mit einem Alter von höchstens 15 Minuten.
Eine bereits registrierte Freigabe bleibt für die spätere Ausstellung gebunden;
die 15 Minuten sind kein Retry-Timeout für `issue`.

Ein in `chat` optional gelieferter Nachweis wird ebenfalls vollständig geprüft.
Beim Wechsel zu `strict` muss eine bestehende unsignierte Chat-Freigabe signiert
für dieselbe unveränderte Revision erneuert werden. Erfolgreich finalisierte
Aufträge bleiben idempotent wiederholbar. Der Betreiber schützt Konfiguration,
Datenbank und Toolchain; der private Gateway-Schlüssel bleibt außerhalb des
Agentenzugriffs.

Für einen ausdrücklich betriebenen lokalen Vorschau-Server:

```bash
export EUER_INVOICE_PREVIEW_BASE_URL=http://127.0.0.1:8787
euer-invoice --db ./invoice.db serve-preview --host 127.0.0.1 --port 8787
```

Der Server erzeugt bei Bedarf Schlüssel neben der Datenbank und setzt den
öffentlichen Prüfschlüssel für seinen eigenen Prozess. `preview_url` ist nur bei
betriebenem Server nutzbar. Im Browser kann der Nutzer entscheiden und das XML
herunterladen; prüfe danach mit `show`, ob `FINALIZED` oder nach Ablehnung `DRAFT`
vorliegt. Der Standardablauf verwendet die HTML-Datei und die Freigabe im Chat.

## Grenzen der Empfangsprüfung

`inspect` liest CII, UBL Invoice/CreditNote und XML-Anhänge aus hybriden PDFs.
Geprüft wird gegen das gepinnte XRechnung-Regelbundle. Andere EN-16931-Profile
können gelesen werden, erhalten ohne passenden positiven KoSIT-Prüfbeleg aber kein
`VALID`. Eine allgemeine profilabhängige Validierung und zusätzliche technische
PDF/A-/Hybrid-Prüfungen sind noch nicht implementiert.

XML ist auf 8 MB, PDF auf 15 MB und daraus extrahiertes XML auf 8 MB begrenzt.
DTD/Entitäten, verschlüsselte oder beschädigte PDFs und widersprüchliche XML-Anhänge
werden abgewiesen. Bekannte Anhangnamen sind `factur-x.xml`, `zugferd-invoice.xml`
und `xrechnung.xml`; unterstützt werden EmbeddedFiles-NameTree und Associated Files.

`ok: true` bedeutet erfolgreiche Analyse, nicht Konformität. Prüfe `valid`,
`validation.status`, Regelfehler und Warnungen. Der technische Prüfstatus ersetzt
keine Prüfung von Belegart, Zahlungsauftrag, Kategorie oder Vorsteuerberechtigung.
Die HTML-Leseansicht hat keine Freigabefunktion und die Analyse verändert die
Rechnungsdatenbank nicht.
