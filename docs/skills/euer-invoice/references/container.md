# Docker / Headless

Dieses Beispiel installiert eine veröffentlichte Paketversion und deren gepinnte
Toolchain beim Image-Build. Der Laufzeitbetrieb kann anschließend offline erfolgen.
Für eine lokale Installation ohne Container verwende den
[Linux-Quickstart](installation_and_setup.md#linux--headless-ohne-homebrew).

Speichere den folgenden Block als `Dockerfile` in einem leeren Build-Verzeichnis.
Die öffentliche Installationsabnahme verwendet genau diesen Block. Lokal ist das
Beispiel erst nach einem erfolgreichen Containerlauf als abgenommen zu behandeln.

```dockerfile
FROM python:3.13-slim-trixie
ARG VERSION
RUN apt-get update \
    && apt-get install -y --no-install-recommends openjdk-21-jre-headless \
    && rm -rf /var/lib/apt/lists/* \
    && test -n "$VERSION" \
    && pip install --no-cache-dir "euer-invoice==$VERSION" \
    && useradd --create-home --uid 10001 invoice \
    && mkdir -p /data /home/invoice/.config/euer-invoice \
    && chown -R invoice:invoice /data /home/invoice/.config
USER invoice
WORKDIR /home/invoice
RUN euer-invoice setup --install-toolchain --format json \
    && test ! -e invoice.db.approval.private.der \
    && test ! -e invoice.db
WORKDIR /data
ENV EUER_INVOICE_DB=/data/invoice.db
ENTRYPOINT ["euer-invoice"]
CMD ["doctor", "--format", "table"]
```

Setze `INVOICE_VERSION` auf eine freigegebene Paketversion mit dem getrennten
Toolchain-/Schlüssel-Setup. Die historische Version 0.4.1 enthält diese Trennung
noch nicht; der Build prüft, dass Setup keine Schlüssel oder Datenbank anlegt.

```bash
docker build --build-arg VERSION="${INVOICE_VERSION:?Freigegebene Paketversion setzen}" -t euer-invoice-local .
docker volume create euer-invoice-data
docker volume create euer-invoice-config
docker run --rm --network none \
  -v euer-invoice-data:/data \
  -v euer-invoice-config:/home/invoice/.config/euer-invoice \
  euer-invoice-local doctor --format table
```

Das Image verwendet den Benutzer `invoice` (UID 10001). Benannte Volumes erhalten
Datenbank, Artefakte und Konfiguration über Container-Neustarts hinweg. Bei
Bind-Mounts müssen die Verzeichnisse für diesen Benutzer beschreibbar sein.
Sichere `/data` und die Konfiguration; Details zur konsistenten Sicherung stehen
in der [Betriebsreferenz](operations.md).

Die Toolchain bleibt im Image im Benutzerdatenverzeichnis; Java wird über `PATH`
gefunden. Das Setup erzeugt weder Rechnungsdatenbank noch Freigabeschlüssel.
Für `strict` hinterlege einen
eigenen Gateway-Public-Key nach der [Setup-Referenz](installation_and_setup.md).
Ein Paket-/Toolchain-Update erfolgt durch erneuten Image-Build mit gewählter Version.

Nutze dieselben Volumes bei weiteren CLI-Aufrufen. Installiere den Skill in der
Agentenumgebung und lies ihn vor der Versionsbestätigung; innerhalb des Containers
zeigt `skill path` dessen Quelle. Die Konfiguration wird anschließend mit
`setup --set skill.version VERSION` im Config-Volume bestätigt. Ältere veröffentlichte
CLI-Versionen können die neueren `skill`-Befehle noch nicht enthalten; deren Bundle
ist über `doctor.skill.bundle_path` verfügbar.

Basis: [offizielles Python-Image](https://hub.docker.com/_/python) mit Debian Trixie
und dessen [Headless-Java-21-Paket](https://packages.debian.org/trixie/openjdk-21-jre-headless).
Ein erfolgreicher Lauf gilt für die geprüfte Architektur. Termux/PRoot und andere
Basisdistributionen benötigen eigene Abnahmen.
