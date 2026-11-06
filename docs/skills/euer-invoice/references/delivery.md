# Versandpaket: XML, Lese-PDF und Begleit-Mail

Erstelle nach erfolgreicher Ausstellung oder bei einer Kopien-Anfrage das Versandpaket
aus der finalen XML. Halte XML und Lese-PDF als getrennte Dateien; ZUGFeRD-Ausgabe ist geplant.

## 1. Finale Dateien exportieren

Prüfe mit `show --id ID`, dass die Rechnung `FINALIZED` ist. Exportiere in einen
Kundenordner außerhalb der internen Artefaktablage:

```bash
euer-invoice export --id ID --out versand/rechnung.xml \
  --html-out versand/rechnung.html --email-out versand/begleitmail.txt --format json
```

Nutze bei vorhandener Nummer das gleiche Namenspräfix für XML, HTML und PDF.
Die Antwort enthält `exported_path`, `recipient_html_path`, `email_draft_path` und
`email_draft` mit `subject` und `body`. Alle Rechnungsdaten der Empfängerfassung und
des Mailentwurfs stammen aus der integritätsgeprüften finalen XML. Die HTML-Datei
enthält die endgültige Rechnungsnummer, Positionen, Steuern und Zahlungsangaben,
keine Freigabefunktion oder internen Dateipfade. Sie bietet „Drucken / als PDF speichern“.
Für eine reine E-Mail-Alternative genügt `--email-out` neben dem XML-Export.

Verwende für Kundenkopien die finale Empfängerfassung, nicht `preview_html_path`
des Entwurfs. Rechnungsinhalte bleiben unverändert; ergänze im Mailentwurf bei Bedarf
Anrede und Gruß. Eine zusätzliche lesbare Darstellung ersetzt die XML nicht.
Für stornierte Rechnungen ist nur der bisherige XML-Export zur Ablage verfügbar;
erstelle daraus kein neues Versandpaket als aktuelle Zahlungsaufforderung.

## 2. Lokal ein Lese-PDF erzeugen

Prüfe zuerst vorhandene lokale Browser: `chromium`, `chromium-browser`, `google-chrome`,
`brave-browser` im PATH; unter macOS auch die ausführbaren Dateien unter
`/Applications/Google Chrome.app/Contents/MacOS/Google Chrome` bzw.
`/Applications/Brave Browser.app/Contents/MacOS/Brave Browser`. Unter Windows nutze
einen vorhandenen Chrome-/Edge-Pfad. Prüfe auch einen bereits eingerichteten
Chromium-Headless-Browser in der laufenden Umgebung. Verwende einen vorhandenen
Renderer; scheitert dessen Start, prüfe einen anderen bereits verfügbaren Browser.
Die CLI benötigt ihn nur für diesen optionalen Druckschritt. Ein Browser-Download
ist keine Voraussetzung für Ausstellung oder E-Mail-Alternative.

Schreibe den folgenden Code in eine temporäre `render_pdf.py` und führe sie mit dem
absoluten Browserpfad sowie den HTML- und PDF-Pfaden als Argumenten aus:
`python3 render_pdf.py BROWSER rechnung.html rechnung.pdf`.
Übergib Dateipfade als Argumentliste, auch bei Leerzeichen.

```python
import os
import subprocess
import sys
import tempfile
from pathlib import Path

browser, source, destination = sys.argv[1:]
source = Path(source).resolve(strict=True)
destination = Path(destination).resolve()
if source.suffix.lower() != ".html" or destination.suffix.lower() != ".pdf":
    raise SystemExit("HTML-Quelle und PDF-Zieldatei erforderlich")
if destination.exists():
    raise SystemExit("PDF-Ziel existiert bereits; neuen Dateinamen wählen")
destination.parent.mkdir(parents=True, exist_ok=True)
with tempfile.TemporaryDirectory(prefix="euer-print-") as profile:
    output = Path(profile) / "lesefassung.pdf"
    subprocess.run([
        browser, "--headless", "--no-first-run", "--disable-extensions",
        "--disable-background-networking", "--no-pdf-header-footer",
        f"--user-data-dir={profile}", f"--print-to-pdf={output}", source.as_uri(),
    ], check=True, capture_output=True, timeout=60)
    content = output.read_bytes()
    if not content.startswith(b"%PDF-") or len(content) < 1000:
        raise SystemExit("Kein lesbares PDF erzeugt")
    # Write only a completed PDF; preserve an existing destination on races.
    with destination.open("xb") as stream:
        stream.write(content)
    os.chmod(destination, 0o600)
print(destination)
```

Das Template verwendet A4, feste Druckränder und lokale Systemschriften. Positionen
werden möglichst zusammengehalten, lange Texte dürfen auf Folgeseiten umbrechen.
Der Browser nutzt ein temporäres Profil und blendet URL-/Datum-Druckkopfzeilen aus.
Die XML wird weder eingebettet noch verändert. Das Ergebnis ist ein gewöhnliches
Lese-PDF, keine ZUGFeRD- oder PDF/A-Zusage.

Prüfe vor Übergabe Rechnungsnummer, Positionen, Beträge, Fälligkeit und IBAN gegen die
XML bzw. den erzeugten Mailtext. Kontrolliere bei mehreren Seiten auch die letzte
Position und den Zahlbetrag; abgeschnittene Inhalte gelten als Druckfehler.

## 3. Ergebnis und E-Mail-Alternative

Bei erfolgreichem Druck liefere XML und PDF zusammen mit dem Begleit-Mailentwurf.
Ergänze dort den Hinweis auf die beigefügte PDF-Lesefassung. Das HTML dient lokal
zur Ansicht und zum erneuten Drucken; es muss nicht mitversendet werden.

Fehlt der Browser, läuft der Druck in ein Timeout oder scheitert die Sichtkontrolle,
liefere XML plus `email_draft.body` mit Positionen, Summen, Steuerhinweisen und
Zahlungsdaten. Sage konkret, weshalb kein PDF vorliegt. Melde einen fehlgeschlagenen
Druck nicht als Erfolg und lasse Rechnungspositionen weiterhin vollständig in der XML.

Übernimm für einen beauftragten Versand die bekannte Rechnungsadresse des Kunden.
Eine elektronische Endpunktkennung in der XML ist nicht automatisch eine E-Mail-Adresse.
Nutze einen verfügbaren Mailkanal nur im Rahmen des Nutzerauftrags und prüfe dessen
Versandbestätigung. Ohne Mailkanal stelle Dateien und Text zum Versand bereit und
benenne diesen Status. Ein Export allein bestätigt keine Zustellung.
