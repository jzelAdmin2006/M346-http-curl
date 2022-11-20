# Übungen zu HTTP und `curl`: `meow`-Konfiguration

Mit den folgenden Übungen soll das Verständnis von HTTP und der Gebrauch von `curl` eingeübt werden.

Dazu werden zuerst zwei Komponenten des Monitoring-Systems `meow` in Betrieb genommen. Anschliessend werden verschiedene Endpoints mithilfe der `config`-Komponente via HTTP konfiguriert.

Die `config`-Komponente wird optional (Zusatzaufgabe) erweitert, sodass man auch Endpoints, die man nicht mehr monitoren möchte, löschen kann.

## Repository forken und klonen

1. Gehen Sie auf das [meow-Repository](https://code.frickelbude.ch/m346/meow).
2. Erstellen Sie einen Fork von diesem Repository.
3. Klonen Sie anschliessend diesen Fork.
4. Öffnen Sie das geklonte `meow`-Verzeichnis in Visual Studio Code.

Das Projekt enthält u.a. folgende Ordner und Dateien:

- `alertingCmd`: Der Alerting-Server, der später zum Einsatz kommen soll.
- `canaryCmd`: Ein einfacher HTTP-Server fürs lokale Testen.
- `configCmd`: Der Konfigurationsserver, der die zu monitorenden Endpoints verwaltet.
- `probeCmd`: Der Daemon, der die _Liveness Probe_ der Endpoints durchführt und das Ergegbnis dieser Überprüfungen loggt.
- `endpoint.go`: Enthält die `Endpoint`-Datenstruktur sowie verschiedene Hilfsfunktionen dazu.
- `endpoint.json`: Enthält einen beispielhaften JSON-Payload, um einen Endpoint zu konfigurieren.
- `logfile.go`: Eine einfache Implementierung für die Logdatei.
- `meow.go`: Enthält Definitionen für verschiedene Emojis, die zum Loggen verwendet werden können.
- `README.md`: Informationen zum Projekt.
- `sample.cfg.csv`: Eine Beispieldatenbank für Endpoints im [CSV-Format](https://de.wikipedia.org/wiki/CSV_(Dateiformat)).

Sie können die Dateien gelegentlich öffnen und überfliegen. Einiges der Inhalte sollte Ihnen schon bekannt vorkommen. Andere Sachen dürften eher neu für Sie sein.

## Canary-Komponente verwenden

Ein sogenannter Canary-Endpunkt erlaubt es dem Client zu prüfen, ob ein Server verfügbar ist, ohne dabei Nutzdaten abzufragen oder zu modifizieren. Canary-Endpunkte werden speziell fürs Monitoring bereitgestellt.

In der Datei `canaryCmd/canary.go` ist ein minimalistischer HTTP-Server definiert, der _nur_ einen Canary-Endpoint anbietet.

**Aufgabe 1**: Starten Sie den Canary-Server folgendermassen:

    go run canaryCmd/canary.go

Der Server läuft standardmässig auf Port 9000.

Versuchen Sie nun per `curl` auf den Endpunkt `/canary` dieses Servers zuzugreifen.

Speichern Sie den Befehl mit dem Ergebnis in der Datei `get-canary.sh` ab und fügen Sie diese Datei diesem Git-Repository hinzu.

## Config-Komponente in Betrieb nehmen

Betrachten Sie die Datei `sample.cfg.csv`. Diese definiert die folgenden drei Endpunkte:

1. `go-dev`: https://go.dev/doc/
2. `libvirt`: https://libvirt.org/
3. `frickelbude`: https://code.frickelbude.ch/api/v1/version

**Aufgabe 2**: Kopieren Sie diese Datei ins gleiche Verzeichnis und geben Sie der Kopie den Namen `config.csv`.

Betrachten Sie nun die Datei `endpoint.go`, genauer den Datentyp `Endpoint`. Diese Struktur definiert sechs Felder, die auch in der CSV-Datei (`config.csv`) mit konkreten Werten angegeben sind.

**Aufgabe 3**: Betrachten Sie die CSV-Datei `config.csv` und überlegen Sie sich, welcher Wert zu welchem Feld in der Struktur `Endpoint` passt. Öffnen Sie anschliessend die Datei `endpoint-fields.md` **in diesem Repository** und ergänzen Sie die Tabelle mit den Angaben aus `config.csv`. Als vierte Zeile können Sie einen eigenen Endpoint definieren. Fügen Sie die Datei dem Git-Repository hinzu und synchronisieren Sie dieses mit dem Server.

Sie können den `config`-Server nun folgendermassen starten:

    go run configCmd/config.go -file config.csv

## Interaktion mit dem Config-Server per HTTP-Schnittstelle und `curl`

Der `config`-Server läuft nun (auf `localhost:8000`). Er unterstützt die folgenden Endpunkte:

- `GET /endpoints`: gibt eine Liste sämtlicher konfigurierter Endpoints zurück
- `GET /endpoints/[identifier]`: gibt einen Endpoint anhand seines Identifiers zurück
- `POST /endpoints/[identifier]`: erstellt oder überschreibt einen Endpoint in der Konfiguration

**Aufgabe 4**: Machen Sie mit `curl` einen `GET`-Request auf `/endpoints`. Leiten Sie die Ausgabe nach `all-endpoints.json` um. Dokumentieren Sie Ihren Befehl in der Datei `get-endpoints.sh`. Fügen Sie beide Dateien diesem Git-Repository hinzu.

**Aufgabe 5**: Machen Sie mit `curl` einen `GET`-Request auf einen spezifischen Endpoint anhand dessen Identifiers. Leiten Sie die Ausgabe nach `endpoint-[identifier].json` um, wobei Sie den Platzhalter `[identifier]` durch Ihren spezifischen Identifier ersetzen. Dokumentieren Sie Ihren Befehl in der Datei `get-endpoint-[identifier].sh`. Fügen Sie beide Dateien diesem Git-Repository hinzu.

**Aufgabe 6**: Kopieren Sie die Datei `endpoint-[identifier].json` nach `new-endpoint.json`. Öffnen Sie nun `new-endpoint.json` in einem Texteditor. Ersetzen Sie alle Werte in dieser Datei durch diejenigen, die Sie vorher bei Aufgabe 3 als vierte Zeile ergänzt haben. Machen Sie nun mit `curl` eine `POST`-Anfrage ([siehe Einführung](README.md#post-anfragen)) an den `config`-Server. Dieser sollte den neuen Endpunkt nun in seine Datenbank (siehe `config.csv` im `meow`-Verzeichnis) aufgenommen haben. Dokumentieren Sie Ihren Befehl in der Datei `post-endpoint-[identifier].sh`. Fügen Sie diese Datei und `new-endpoint.json` diesem Git-Repository hinzu.

## Zusatzaufgabe: Ergänzung des Config-Servers um `DELETE`-Endpunkt

Der `config`-Server (Datei `configCmd/config.go`) enthält in der `main()`-Funktion einen `TODO`-Kommentar. Der `/endpoints/[identifier]`-Endpunkt soll neu auch die `DELETE`-Methode unterstützen, damit man per `curl -X DELETE` auch Endpoints aus der Konfiguration heraus löschen kann, die man nicht länger monitoren möchte.

**Aufgabe 7**: Lesen Sie den Code der Funktion `postEndpoint` weiter unten in der Datei und versuchen Sie ihn zu verstehen. Einen Teil dieses Codes können Sie für die Implementierung der `DELETE`-Methode übernehmen.

**Aufgabe 8**: Ergänzen Sie nun das `switch`/`case`-Statement beim `TODO`-Kommentar um einen weiteren Fall (`http.MethodDelete`). Hier soll die Funktion `deleteEndpoint()` aufgerufen werden. Implementieren Sie diese Funktion und testen Sie diese mit `curl`. (Sie müssen den Server dabei nach jeder Änderung neu starten.)
