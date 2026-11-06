# euer-invoice

Lokale, deterministische Rechnungsverwaltung für KI-Agenten mit
**XRechnung 3.0.2 CII**. Der Agent führt den Dialog; die CLI verwaltet Daten,
berechnet Beträge und erzeugt unabhängig geprüfte Rechnungen.

- Aussteller, Kunden, Entwürfe, Ausstellung mit schreibgeschützter Originalablage,
  Storno-Vermerke und Zahlungen.
- HTML-Vorschau und ausdrückliche Nutzerfreigabe im Chat; optional signierte Gateway-
  Freigaben oder eine lokale Web-Vorschau.
- Finale Empfängerfassung und Begleit-Mail aus der ausgestellten XML; der Skill
  erzeugt mit einem vorhandenen lokalen Browser ein separates Lese-PDF oder liefert
  die Rechnungspositionen direkt als lesbaren Mailtext.
- EUR, 19 %/7 % auch gemischt, Kleinunternehmerbefreiung, steuerfreie Leistungen,
  Reverse Charge und bis zu 50 Positionen.
- Empfangene CII-/UBL-Rechnungen und hybride ZUGFeRD-/Factur-X-PDFs lesen und prüfen.

Python 3.11+ ohne zusätzliche Python-Laufzeitpakete, SQLite und Java 21+ für
Erzeugung und Prüfung. Die Engine verwendet Mustangproject und einen separaten,
gepinnten KoSIT-Validator. Nach eingerichteter Toolchain ist der Betrieb offline möglich.
ZUGFeRD-Ausgabe ist auf der Roadmap. PDF/A-Erzeugung, automatischer Bank-Sync und
Mahnungen sind noch nicht enthalten.
Die Empfangsprüfung nutzt derzeit das XRechnung-Regelbundle; sie validiert nicht
sämtliche ZUGFeRD-/Factur-X-Profile.

## Installation und Agenten-Skill

Mit Java 21+ im `PATH` oder unter `EUER_INVOICE_JAVA`:

```bash
pipx install euer-invoice
euer-invoice setup --install-toolchain --format json
euer-invoice doctor --format json
```

Die Paketinstallation und der Toolchain-Download setzen veröffentlichte PyPI-/Release-
Artefakte im [öffentlichen Download-Repository](https://github.com/curiousmarkus/euer-invoice-releases/releases) voraus.
Offline-Installation und Homebrew beschreibt die
[Setup-Referenz](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/references/installation_and_setup.md).

Installiere den **gesamten Skill-Ordner** aus `doctor.skill.bundle_path` im
Agentensystem, einschließlich `references/`. Lies die dortige `SKILL.md`, bestätige
anschließend ihre Version und richte das Ausstellerprofil ein. Die Schritte stehen
in der [Skill-Einrichtung](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/references/installation_and_setup.md#1-skill-in-agentensystemen-installieren-und-aktualisieren).
`doctor.skill.status` muss danach `current` sein.

## Nutzung

Die zentrale Bedienungsanleitung ist der [Agenten-Skill](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/SKILL.md).
Er führt durch Stammdaten, Entwurf, Klärung fehlender Angaben, technische Prüfung,
Nutzerfreigabe und Ausstellung. Änderungen erfordern eine neue Vorschau und Freigabe.

Empfangene Rechnungen lassen sich ohne Änderung der Rechnungsdatenbank analysieren:

```bash
euer-invoice inspect --file eingang.xml --html-out eingang.html --format json
```

`ok: true` bestätigt die Analyse; `valid` und `validation.status` zeigen den technischen
Prüfstatus. Details und Grenzen stehen im [Skill](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/SKILL.md#7-empfangene-e-rechnungen-prüfen-inspect).

Ohne Lizenz sind drei Ausstellungen je Kalendermonat des Rechnungsdatums und lokaler
Datenbank frei. Entwürfe, Vorschauen, Empfangsprüfungen, Stornierungen und Zahlungen
sind unbegrenzt frei; [Lizenzen](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/references/installation_and_setup.md#7-freikontingent-und-lizenzaktivierung)
schalten als Einmalkauf unbegrenzt viele Rechnungen für alle Rechnungsjahre frei.
DATEV bleibt separat über Jahreslizenzen lizenziert.

Sichere Datenbank **und** zugehöriges Artefaktverzeichnis gemeinsam. Vorgehen und
Wiederaufnahme nach einem Abbruch stehen in der
[Betriebsreferenz](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/references/operations.md).

## Dokumentation

| Thema | Einstieg |
| --- | --- |
| Bedienung durch den Agenten | [Skill](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/SKILL.md) |
| Installation, Konfiguration und Lizenzen | [Setup-Referenz](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/references/installation_and_setup.md) |
| Export, Sicherung und Prüfgrenzen | [Betriebsreferenz](https://github.com/curiousmarkus/euer-invoice-releases/blob/main/docs/skills/euer-invoice/references/operations.md) |

Entwicklungsumgebung, CI, Release-Verfahren und Architektur sind im privaten
Entwicklungsrepository dokumentiert.
