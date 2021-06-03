**ACHTUNG:** Dies ist KEINE volle Übersetzung der englischen Dokumentation, sondern einfach nur ein Copy&Paste meiner ursprünglichen deutschsprachigen Notizen.

_**NOTE:** This is NOT a full translation of [README.md](README.md), but just a dump of what I had written already anyway while doing my reverse engineering._

Es gibt vier Parameter:
1. `format`: Kann auf `webp` gesetzt werden, `png` geht auch, alle anderen Eingaben die ich getestet habe (gif, jpg, jpeg) defaulten dazu, dass doch wieder ein PNG zurückgegeben wird.

2. `k`: sehr mysteriös. Von der Länge könnte es ein Unix-Timestamp sein, allerdings von vor mindestens fünf Jahren. Außerdem ist der Parameter negativ?! Ihn wegzulassen scheint optisch nichts zu verändern.

3. `time`: relativ offensichtlich. Datum im Format `YYYYMMDD-HHMM-2`, z.B.: `20210603-1415-2` Keine Ahnung wofür das `-2` steht.

4. `tiles`: Base64-encodierter String mit Angaben für die "Ebenen" die in der Karte enthalten sind. Decodierbar in JavaScript mit der Funktion `atob`, die Gegenrichtung mit `btoa`. Ist in mehrere Komponenten gegliedert, die durch Pipes (`|`) getrennt sind.

Splittet man einen Beispiel-Wert für `tiles` an den Stellen der Pipes auf, erhält man folgende Komponenten:
- topo
- 1;;0;0
- wetterradar/prozess/tiles/geolayer/rasterimages/rr_topography/v1/ZL8/512/132_84.jpg$r
- 2;;0;0;false
- wetterradar/prozess/tiles/rainlayerProg/2021/06/03/18/35/v24/ZL7/522/sprite/66_42.png;wetterradarglobal/prozess/tiles/rainlayerProg/2021/06/03/18/30/v14/ZL7/522/border/66_42.png$i
- 1;;0;0
- geo/prozess/karten/produktkarten/wetterradar/generate/rasterTiles/rr_geooverlay_cities_excluded/v2/ZL8/512/132_84.png

Lässt man beim letzten das `_cities_excluded` weg, erhält man ein Bild MIT Städtenamen. Aber Achtung: An den Stellen, an denen Städtenamen über Grenzen oder Flüsse hinweggehen, sind diese auch in der Variante ohne Städtenamen unterbrochen um Platz für die Labels zu schaffen.

Das `$r` sorgt dafür, dass das Regenradar tatsächlich eingeblendet wird
Das `$i` für die Grenzen und Städtenamen
`ZL` steht wahrscheinlich für "zoom level"

statt v24 funktioniert auch v21 - v26

zweite URL mit "border" und so kann auch einfach weggelassen werden

2;;0;1;false
a;b;c;d;e

a wohl von 1 bis 9, sonst sehe ich keinen Regen mehr
b scheint völlig egal zu sein
c scheint irrelevant zu sein, muss aber Int sein
d muss 0 oder 1 sein
e ob true oder false macht optisch keinen Unterschied, kann auch ganz weggelassen werden

a teilweise EXTREM relevant, man bekommt ganz andere Regenbilder wenn man's verändert!
