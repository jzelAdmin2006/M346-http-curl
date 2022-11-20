# HTTP mit `curl`

Das Hypertext Transfer Protocol [HTTP](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol) ist eines der wichtigsten Protokolle auf [Anwendungsebene](https://de.wikipedia.org/wiki/OSI-Modell#Schicht_7_%E2%80%93_Anwendungsschicht_(Application_Layer)). Das Präfix `http://` (bzw. die verschlüsselte Variante `https://` mit [TLS](https://de.wikipedia.org/wiki/Transport_Layer_Security)) von Webseiten-URLs dürfte von der Browser-Adresszeilen bekannt sein. HTTP(S) wird jedoch nicht nur in Browsern verwendet, sondern beispielsweise auch für Anwendungsschnittstellen zwischen Frontend und Backend (via [REST](https://de.wikipedia.org/wiki/Representational_State_Transfer)-API oder per [GraphQL](https://de.wikipedia.org/wiki/GraphQL)) einer Web-Anwendung, oder aber zum klonen von Git-Repositories (ausserhalb des sluz-WiFis, wohlgemerkt).

Eine übersichtliche, englischsprachige [Einführung in HTTP](https://www.freecodecamp.org/news/http-and-everything-you-need-to-know-about-it/) finden Sie auf FreeCodeCamp.org.

## Einführung das `curl`-Kommandozeilenwerkzeug

Das Kommandozeilenwerkzeug [`curl`](https://curl.se/) erlaubt den Zugriff auf HTTP-Ressourcen über die Kommandozeile. Der `curl`-Befehl steht auf der Ubuntu-VM über das Terminal bzw. auf Windows über die Git Bash zur Verfügung. Das [offizielle Tutorial](https://curl.se/docs/manual.html) und die [manpage](https://curl.se/docs/manpage.html) erklären den Gebrauch von `curl`.

Im Folgenden sollen die wichtigsten Handgriffe eingeübt werden, damit anschliessend die Konfiguration von [meow](https://code.frickelbude.ch/m346/meow) durchgeführt werden kann.

### `GET`-Anfragen

Eine HTTP-Ressource kann mit dem `curl`-Befehl per `GET`-Methode anhand ihrer URL geladen werden:

    $ curl -X GET https://code.frickelbude.ch/api/v1/version
    {"version":"1.17.3"}

In diesem Fall kommt ein JSON-Dokument zurück, welches die aktuelle Version des Gitea-Servers angibt.

Mit dem Parameter `-X` kann die HTTP-Methode bestimmt werden. Da standardmässig `GET` verwendet wird, kann die Angabe im obigen Beispiel auch weggelassen werden:

    $ curl https://code.frickelbude.ch/api/v1/version
    {"version":"1.17.3"}

Das Ergebnis kann via Umleitung (mit dem `>`-Operator) als Datei abgespeichert werden:

    $ curl https://code.frickelbude.ch/api/v1/version > version.json
    $ cat version.json
    {"version":"1.17.3"}

Mit `GET`-Anfragen werden Informationen heruntergeladen, aber nicht auf dem Server modifiziert.

### `POST`-Anfragen

Will man eine serverseitige Ressource erstellen oder bearbeiten, verwendet man die Methode `POST`. Dies kann explizit mit dem Parameter `-X` angegeben werden:

    $ curl -X POST https://code.frickelbude.ch/api/v1/markdown
    {"message":"[]: Empty Content-Type","url":"https://code.frickelbude.ch/api/swagger"}

Eine solche Anfrage ist jedoch nicht sinnvoll, da die zu erstellende Ressource nicht mitgegeben worden ist.

Der obige Endpoint (`/api/v1/markdown/raw`) rendert ein mitgegebenes Markdown-Dokument als HTML und kann folgendermassen verwendet werden:

    $ curl -X POST -H 'Content-Type: text/plain' -H 'accept: text/html' -d '# Hello' https://code.frickelbude.ch/api/v1/markdown/raw
    <h1 id="user-content-hello">Hello</h1>

- Der Parameter `-X` (kurz für `--request`) gibt an, dass eine `POST`-Anfrage ausgeführt wird.
- Der Parameter `-H` (kurz für `--header`) definiert einen Request-Header. Hier wurden zwei davon definiert:
    1. `Content-Type: text/plain` gibt an, dass hier Klartext mitgegeben wird.
    2. `accept: text/html` gibt an, dass man als Antwort ein HTML-Dokument erwartet.
- Der Parameter `-d` gibt (kurz für `--data`) gibt die Nutzdaten (den sogenannten _Request Body_) mit.
    - Hier handelt es sich um ein minimales Markdown-Dokument, bestehend aus der Überschrift `# Hello`.

Das Beispiel soll erweitert werden, indem ein längeres Markdown-Dokument mitgegeben wird (`hello.md`):

    This _is_ a **test** for ~~HTML~~ Markdown rendering using `curl`.

Möchte man dem `POST`-Request die Datei mitgeben, kann man wiederum den `-d`-Parameter verwenden. Dem Dateinamen (`hello.md`) muss aber ein `@`-Zeichen vorangestellt werden (`@hello.md`):

    $ curl -X POST -H 'Content-Type: text/plain' -H 'accept: text/html' -d @hello.md https://code.frickelbude.ch/api/v1/markdown/raw
    <p>This <em>is</em> a <strong>test</strong> for <del>HTML</del> Markdown rendering using <code>curl</code>.</p>

Die Ausgabe kann wiederum mit `>` in eine Datei umgeleitet werden:

    $ curl -X POST -H 'Content-Type: text/plain' -H 'accept: text/html' -d @hello.md https://code.frickelbude.ch/api/v1/markdown/raw >hello.html

Dies sollte genügen, um die [Übungen](exercises.md) lösen zu können! Wenn Ihnen `curl` Mühe bereitet, können Sie auch [Postman](https://www.postman.com/) verwenden. (Der Vorteil von `curl` ist, dass man den Befehl in Skripten verwenden kann.)