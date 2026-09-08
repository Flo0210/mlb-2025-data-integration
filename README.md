# MLB 2025 – Integration von CSV-Daten und Semantic Web (SPARQL) in SQLite

Ein Datenintegrations-Projekt, das drei heterogene Quellen zu einer gemeinsamen,
abfragbaren Datenbasis zusammenführt: eine lokale CSV-Datei mit Leistungsdaten der
MLB-Saison 2025, biographische Fakten aus DBpedia und eine persistente SQLite-Datenbank
als Zielsystem.

## Fragestellung

Wie hängen Alter und Schlaghand von Baseball-Spielern mit ihrer sportlichen Leistung
zusammen? Die Leistungsdaten liegen als CSV vor, die biographischen Angaben nicht –
sie müssen aus dem Semantic Web geholt und mit den vorhandenen Daten verknüpft werden.

## Architektur

1. **Lokale Datenbasis (CSV)** – Einlesen der Saisondaten 2025 mit pandas: Home Runs,
   Batting Average, Saison.
2. **Globales Wissen (RDF/SPARQL)** – Abfrage von Geburtsdatum (`dbo:birthDate`) und
   Schlaghand (`dbp:bats`) über den DBpedia-Endpoint für die Top-10-Home-Run-Hitter.
3. **Relationale Integration (SQLite)** – Beide Bestände wandern als getrennte Tabellen
   (`player_stats`, `player_biographies`) in eine persistente Datenbank. Die
   Zusammenführung erfolgt anschließend per SQL-`JOIN` über den Spielernamen.

## Datenaufbereitung

Die interessanteren Stellen des Projekts liegen im Cleaning:

- **Namensformat:** Die CSV liefert Namen als `Nachname, Vorname`. Ohne Umwandlung in
  `Vorname Nachname` findet DBpedia keinen einzigen Treffer. Die Spalte wird deshalb
  per `str.split()` zerlegt und neu zusammengesetzt.
- **Virtuoso-Anomalie:** DBpedia aggregiert bei leeren Feldern fehlerhaft. Statt `MAX()`
  und `GROUP BY` im SPARQL-Query zu verwenden, werden die rohen, ungruppierten Fakten
  geladen und die Aggregation in pandas erledigt.
- **Uneinheitliche Rückgaben:** Die Schlaghand kommt teils als Klartext, teils als
  URI zurück und wird auf den reinen Wert normalisiert.
- **Idempotenz:** `if_exists='replace'` sorgt dafür, dass das Notebook beliebig oft
  ausführbar bleibt, ohne dass Daten doppelt in den Tabellen landen.

## Auswertungen

- **A:** Top-Home-Run-Hitter mit Alter (im SQL aus dem Geburtsdatum berechnet) und
  Schlaghand.
- **B:** Batting Average im Verhältnis zum Alter, ohne erneute Abfrage von DBpedia –
  die Daten liegen nach dem ersten Durchlauf lokal vor.

## Technologien

Python · pandas · SPARQLWrapper · SQLite · Jupyter Notebook

## Ausführen

```bash
pip install pandas SPARQLWrapper notebook
jupyter notebook mlb_data_integration.ipynb
```

`stats.csv` muss im selben Verzeichnis liegen. Die Datei `baseball_2025.db` wird beim
Ausführen automatisch erzeugt. Für Schritt 2 wird eine Internetverbindung benötigt,
da DBpedia live abgefragt wird.

## Kontext

Einzelarbeit im Rahmen der Lehrveranstaltung *Data and Knowledge Engineering*,
Bachelorstudium Wirtschaftsinformatik, Wirtschaftsuniversität Wien.
