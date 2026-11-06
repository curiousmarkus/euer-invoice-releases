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

Sichere Datenbank und Artefaktverzeichnis durch Dateikopien als zusammengehörige Einheit:

1. Alle schreibenden CLI-Prozesse und gegebenenfalls den Vorschau-Server stoppen.
2. Die Datenbank und das zugehörige Artefaktverzeichnis gemeinsam kopieren, z. B.
   `invoice.db` **und** `invoice.db.artifacts/`.
3. Die Kopie an einem getrennten Sicherungsort aufbewahren. Konfiguration,
   Lizenzdateien und benötigte Freigabeschlüssel zusätzlich sicher verwahren;
   private Schlüssel entsprechend ihrer Zugriffsgrenze getrennt schützen.
4. Beim Wiederherstellen Datenbank und Artefakte gemeinsam unter denselben relativen
   Namen zurückspielen. Nutze zunächst einen freien Zielort, um den bisherigen
   Bestand zu erhalten.
5. Mit `--db` die wiederhergestellte Datenbank wählen und mit `show --id ID`
   referenzierte Originale prüfen. Auch Vorschauen und benötigte Konfiguration prüfen,
   bevor der Betrieb wieder aufgenommen wird.

Relative Originalreferenzen bleiben bei einem gemeinsamen Umzug gültig.
Kundenkopien können aus den Originalen erneut exportiert werden. Hash-Prüfungen
ersetzen weder Backup noch die Aufbewahrung der zugehörigen Dateien.

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
