# SPARQL-Demo mit dem Wikidata-SPARQL-Endpoint 

## Wikidata-SPARQL-GUI

- Achtung: Der Wikidata-SPARQL-Endpoint sowie die GUI bieten einige Erleichterungen und Erweiterungen, die über die eigentliche SPARQL-Definition hinausgehen!

- - -

- gehe auf https://query.wikidata.org
- Es erscheint eine Graphische Benutzeroberfläche (GUI) zum Absenden von SPARQL-Anfragen und Anzeigen der Ergebnisse
- Im Editorfeld wird die Anfrage formuliert, im unteren Abschnitt werden die Ergebnisse angezeigt.
- Über den Button "Beispiele" stehen Beispielanfragen zur Verfügung, die das Potential der Schnittstelle und Datenbasis 
  demonstrieren und gute Einstiegspunkte für Learning-by-Doing bieten.

## Krankenhäuser in Deutschland

- Gehe auf das Beispiel "Karte mit Krankenhäusern". Führe die Anfrage aus.
- Durch die Zeile "#defaultView:Map" wird automatisch das Ergebnis als Karte angezeigt.
  - Wikidata bietet eine Aufbereitung der Ergebnisse in verschiedenen Formaten an, wie Karten oder Diagramme.
    Die [Dokumentation](https://www.wikidata.org/wiki/Special:MyLanguage/Wikidata:SPARQL_query_service/Wikidata_Query_Help/Result_Views)
    der Funktionalität ist über das "?" in der Ergebnisleiste erreichbar.   
- Wir wollen die Karte auf Krankenhäuser in Deutschland reduzieren
  - Wir fügen ein Tripel hinzu, das besagt, dass sich das Krankenhaus (`?item`) in Deutschland befinden soll.
  - Oberhalb der schließenden geschweiften Klammer fügen wir die Zeile `?item wdt:P17 wd:Q183.` ein.
  - Beim Tippen von `wdt:` und `wd:` erscheint ein Hinweis, dass man nach Eigenschaften bzw. Entities suchen kann.
    Mit STRG-Leertaste erscheint eine Treffermenge. Wikidata setzt dann den richtigen Identifier ein.
- Wir führen die Anfrage aus. Die Karte fokussiert sich auf Deutschland. Wir sehen einen Ausreißer in Polen.
  Nach Klick auf den Punkt und Betrachtung der Wikidata-Seite der Entity wird klar, dass es sich um einen
  Fehleintrag handelt.
- Wir können einzelne Entities über einen Filter aussschließen
  - Wir fügen unter die vorige Zeile folgende Zeile hinzu: `FILTER(?item != wd:Q57014971)`
  - Wir führen die Anfrage erneut aus und sehen, dass das Krankenhaus nicht mehr in der Ergebnisliste enthalten ist.

## Hinzufügen des Name-Service

- Unsere Ergebnisliste enthält noch keine Namen der Krankenhäuser
- Wikidata stellt für SPARQL einen speziellen Name-Service bereit, der die Namen/Titel ("Label") der in der Anfrage auftauchenden
  Entities in verschiedenen Sprachen abrufen und einbinden kann. Dies ist eine Wikidata-eigene Erweiterung!
- In anderen Beispielen wie "Metrostationen in Paris" können wir erkennen, wie der Service eingebunden wird.
- Wir fügen wiederum am Ende hinzu: `SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],de,en" }`
- Wir müssen weiterhin die Variablenliste abändern, damit die Labels in der Ergebnisliste auftauchen:
  `DISTINCT *` zu `?item ?itemLabel ?geo`.
 
## Anzahl der Betten   

- Schauen wir uns einen Krankenhaus-Datensatz an, sehen wir, dass es als Eigenschaft die Anzahl der Betten gibt. (`wdt:P6801`)
- Wir können sie uns mit ausgeben lassen, indem wir die Zeile `?item wdt:P6801 ?betten` hinzufügen und `?betten` in die Variablenliste aufnehmen.
- Wenn wir die Anfrage ausführen, ist die Ergebnismenge erheblich geschrumpft.
  Das liegt daran, dass nurnoch Krankenhäuser berücksichtigt werden, bei denen die Bettenzahl hinterlegt ist.
- Wir können auch die Krankenhäuser ohne Bettenzahl wieder mit aufnehmen, indem wir die Bettenzahl optional machen.
  Wir ändern obige Zeile ab in: `OPTIONAL { ?item wdt:P6801 ?betten }`
- Nun erscheinen wieder alle Krankenhäuser
- Fügen wir ganz am Ende hinter der geschweiften Klammer ein `ORDER BY DESC(?betten)` hinzu, wird
  die Ergebnismenge nach Anzahl der Betten sortiert. Auf der Kartenansicht wirkt sich dies nicht aus,
  sehr wohl aber in der Tabellenansicht.

## Werte zuweisen

- Die Karte unterstützt die Variable `?rgb`, um die Farbe des Punktes zu bestimmen.
- Wir wollen erreichen, dass die Farbe die Bettenzahl widerspiegelt.
- Mit `BIND( ... AS ?rgb)` definieren wir eine Variable, die es in den Daten nicht gibt und weisen ihr einen Wert zu
- Mittels `IF(test, then, else)` können wir Fallunterscheidungen treffen
- Wir fügen am Ende vor der geschweiften Klammer hinzu `BIND(IF(!BOUND(?betten), "000000", IF (?betten > 1000, "0000ff", "00ff00")) AS ?rgb)`
- Mit `BOUND` überprüfen wir, ob die Bettenzahl gegeben ist

## Betten pro Mitarbeiten

- Wir fügen hinzu:
  `  ?item wdt:P1128 ?beschaeftigte. BIND(?beschaeftigte / ?betten AS ?ratio)`
- Man könnte die Farbe nun nach der Anzahl der Beschäftigten pro Bett staffeln.

