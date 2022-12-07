# Setup für das Routing

## Datenbank
Es wird weiterhin ein Postgresql-Datenbankserver verwendet. Hier haben wir eine Datenbank `routing_db` für alle Routing-bezogenen Aufgaben angelegt.

## Kartendaten
Als Grundlage für das Routing verwenden wir Kartendaten von [OpenStreetMap (OSM)](https://www.openstreetmap.de/). Diese haben wir von [Geofabrik](http://download.geofabrik.de/europe/germany.html) im pbf-Format heruntergeladen.

## osm2po
Nun müssen die OSM-Daten noch in unsere PostgresDB importiert werden. Hierfür verwenden wir das Tool [osm2po.de](https://osm2po.de/). Mit osm2po generieren wir aus der heruntergeladenen pbf-File zwei sql-insert-Skripte (eines für die Kanten und eines für die Knoten).

```
java -Xmx24g -jar osm2po-core-5.5.5-signed.jar prefix=de tileSize=x PATH_TO_PBF postp.0.class=de.cm.osm2po.plugins.postp.PgRoutingWriter postp.1.class=de.cm.osm2po.plugins.postp.PgVertexWriter
```

Anschließend führen wir noch die beiden erstellten SQL-Skripte aus:
```
psql --host HOST --port PORT --username USERNAME --password --dbname DB_NAME --file de_2po_4pgr.sql
```

```
psql --host HOST --port PORT --username USERNAME --password --dbname DB_NAME --file de_2po_vertex.sql
```


## pgRouting
Um Routen zu berechnen wird die Postgres-Extensions [pgRouting](https://pgrouting.org/) verwendet.

### Installation
TODO: Beschreiben

### Umbenennen der Tabellen
- Für den Einsatz von pgRouting müssen wir die mit OSM-Daten befüllten Tabellen noch leicht anpassen, da pgRouting gewisse Namenskonventionen erwartet.
	- Die Tabellen haben wir wie folgt benannt:
		- de_edges
		- de_edges_vertices_pgr
    - In der vertices-Tabelle müssen wir die geom-Spalte zu "the_geom" umbenennen. Aus Konsistenzgründen haben wir dies bei der edges-Tabelle analog gemacht.

### Überprüfen der Netzwerk-Topology
Vor dem eigentlich Routing sollte man zunächst noch überprüfen, ob es Fehler in der Netzwerk-Topology gibt:

```
SELECT pgr_analyzeGraph('de_edges', 0.000002, 'the_geom', 'id');
```
