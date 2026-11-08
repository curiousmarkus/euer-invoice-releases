# Steuer- und Lieferantenkennungen

## Fehlende Identifikation klären

Prüfe die beiden Anforderungen getrennt anhand von `required_fields_missing`:

| Pfad | Benötigte Angabe |
| --- | --- |
| `seller.identifier_or_legal_registration_id_or_vat_id` | Verkäuferkennung BT-29, Registerkennung BT-30 oder USt-IdNr. BT-31 (BR-CO-26) |
| `seller.vat_id_or_tax_number` | USt-IdNr. BT-31 oder Steuernummer BT-32 |

Eine USt-IdNr. erfüllt beide Anforderungen. Kleinunternehmer dürfen sie verwenden;
Steuerkategorie und Steuersatz ergeben sich aus der Steuerbehandlung der Rechnung.
Steuernummer plus tatsächliche Verkäufer- oder Registerkennung ist ebenfalls möglich.
Verwende nur vorhandene, zum Aussteller und Empfänger passende Kennungen. Frage nach
fehlenden Fakten; erfinde keine Nummer und leite sie nicht aus anderen Feldern ab.

Für Kleinunternehmerrechnungen nennt § 34a UStDV auch die Kleinunternehmer-IdNr.
Die Engine unterstützt hierfür derzeit nur USt-IdNr. und Steuernummer. Wirtschafts-
IdNr., persönliche steuerliche IdNr. und Kleinunternehmer-IdNr. sind keine Ersatzwerte
für `vat_id`. Eine USt-IdNr. kann beim BZSt beantragt werden; sie ist keine allgemeine
Voraussetzung für den Kleinunternehmerstatus. Die Formatprüfung bestätigt weder
Vergabe noch Inhaberschaft einer Nummer.

## Lieferanten-/Kreditorennummer beim Empfänger

Eine vom Kunden vergebene Lieferantennummer identifiziert **den Aussteller im System
dieses Kunden**. Pflege sie im betreffenden Kontakt als `seller_identifier`, z. B.
mit dieser `contact-patch.json`:

```json
{"seller_identifier": "KRED-4711"}
```

```bash
euer-invoice contact update --id CONTACT_ID --patch contact-patch.json --idempotency-key AUFTRAG-CONTACT --format json
```

Bei `draft create` über `contact_id` wird die Nummer übernommen. Ohne gespeicherten
Kontakt oder für eine einzelne abweichende Rechnung setze `buyer.seller_identifier`.
Im Rechnungssnapshot steht der wirksame Wert als `seller.identifier`; das XML enthält
ihn als Verkäuferkennung BT-29. Käuferkennung BT-46 und Käuferreferenz/Leitweg-ID BT-10
bleiben davon getrennt. USt-IdNr. und Registerkennung bleiben zusätzlich erhalten.

Die Priorität ist: ausdrücklicher Inline-Wert vor Kontaktwert vor globalem
`seller.identifier`. `buyer.seller_identifier: null` lässt BT-29 für diese Rechnung
ausdrücklich weg, einschließlich eines globalen Fallbacks. Ein Leerstring ist ungültig.
Ein Kontakt-Update mit `seller_identifier: null` entfernt dagegen den Kontaktwert;
zukünftige Entwürfe können dann wieder den globalen Fallback verwenden.

## Bestehende Entwürfe, Empfängerwechsel und Klonen

Kontaktänderungen wirken auf neue Übernahmen. Bearbeite einen bestehenden Entwurf
ausdrücklich per `draft update` mit `buyer.seller_identifier`; Prüfung und Freigabe
werden dadurch ungültig. `--refresh-seller` behält die im Entwurf getroffene
kundenspezifische Entscheidung einschließlich `null` bei.

Änderst du bereits befüllte Empfängerangaben zu Name, Anschrift, Land oder USt-IdNr.,
bestätige oder ersetze eine vorhandene kundenspezifische Kennung im selben
`buyer`-Patch oder entferne sie mit `null`. Die Kontaktbindung wird bei solchen
Änderungen gelöst. Ergänzungen bisher fehlender Angaben und Änderungen der
E-Mail-Adresse erfordern keine erneute Bestätigung der Kennung.

`draft clone` übernimmt den gespeicherten Empfänger samt Beziehungsangabe und liest
keine aktuellen Kontaktwerte nach. Globale Bestandskennungen werden nicht automatisch
auf Kontakte verteilt. Kläre, ob eine solche Nummer kundenspezifisch ist, und bereinige
die Stammdaten dann ausdrücklich. Ausgestellte Originale bleiben unverändert.

## Datenschutz und Lesefassungen

Sind Steuernummer und USt-IdNr. im Aussteller-Snapshot enthalten, werden beide ins XML
und die Lesefassung übernommen. Für den Verzicht auf die Steuernummer bei vorhandener
USt-IdNr. folge der [Setup-Referenz](installation_and_setup.md#4-ersteinrichtung-aussteller-profil--onboarding).
Ein ausgeblendeter HTML-Text würde die Steuernummer im XML nicht entfernen.

Entwurfsansichten verwenden ihren Snapshot, Empfänger- und Prüfansichten das XML.
Kategorie `E` allein bedeutet keinen Kleinunternehmerstatus; maßgeblich für die
Darstellung ist der tatsächlich enthaltene Befreiungsgrund. Der Browserdruck nutzt
dieselben Angaben.
