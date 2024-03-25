# Erstellung von Tabellen
Wir verwenden den CREATE-Befehl, um eine Tabelle zu erstellen.
```SQL
CREATE TABLE Kunde(
    ID INT,
    Name VARCHAR(50),
    Geburtsdatum DATE
);
```
## Primärschlüssel
Es gibt zwei Varianten, den Primärschlüssel festzusetzen.

Direkt nach dem Attribut:
```SQL
CREATE TABLE Kunde(
    ID INT PRIMARY KEY,
    Name VARCHAR(50),
    Geburtsdatum DATE
);
```

Oder nach der Definition
```SQL
CREATE TABLE Kunde(
    ID INT,
    Name VARCHAR(50),
    Geburtsdatum DATE,
    PRIMARY KEY (ID)
);
```

Der Primary Key hat die Eigenschaften:
- Eindeutig innerhalb der Tabelle (UNIQUE)
- Darf nicht NULL sein (NOT NULL)
- Wird indexiert, wodurch eine schnelle und effiziente Suche ermöglicht wird (z.B. für JOINS, da PK = FK)

## Fremdschlüssel
Der Fremdschlüssel wird meist am ende der Tabellendefinition gesetzt, da der Befehl etwas umfangreicher werden kann
```SQL
CREATE TABLE Bestellung(
    Bestellnummer INT,
    KundeID INT,
    Preis DECIMAL,
    PRIMARY KEY (Bestellnummer),
    FOREIGN KEY (KundeID) REFERENCES Kunde(ID)
);
```

Der Foreign Key:
- DARF Null sein
- Muss NICHT UNIQUE sein
- Wird indexiert

Fremdschlüssel können mit Integritätsregeln versehen werden.

Beispiel für **referenzielle Integrität**, bei der das Löschen eines Datensatzes mit einem Primary Key nicht möglich ist, sofern dieser noch von einem Foreign Key referenziert wird:
```SQL
CREATE TABLE Bestellung(
    Bestellnummer INT,
    KundeID INT,
    Preis DECIMAL,
    PRIMARY KEY (Bestellnummer),
    FOREIGN KEY (KundeID) REFERENCES Kunde(ID)
    ON DELETE DENY ON UPDATE DENY
);
```

Beispiel für **Lösch- und Änderungsübernahme**, bei der das Update von einem Primary Key direkt die Foreign Keys mitändert, die den Primary Key referenziert haben:

```SQL
CREATE TABLE Bestellung(
    Bestellnummer INT,
    KundeID INT,
    Preis DECIMAL,
    PRIMARY KEY (Bestellnummer),
    FOREIGN KEY (KundeID) REFERENCES Kunde(ID)
    ON DELETE CASCADE ON UPDATE CASCADE
);
```

# Einfügen von Datensätzen

Datensätze werden mit dem INSERT-Befehl erstellt.

```SQL
INSERT INTO Tabellenname(Attribut1, Attribut2) VALUES (Wert1, Wert2);
```

Wenn jedes Attribut in genau der Reihenfolge, wie sie in der Tabelle aufzufinden sind, beschrieben werden soll, dann kann die Attributliste weggelassen werden.

```SQL
INSERT INTO Tabellenname VALUES (Wert1, Wert2, Wert3...);
```

## Wie gelangen NULL-Werte in die Datenbank?
Variante 1: Bei dem Werten in dem INSERT wird Explizit ein NULL eingefügt:
```SQL
INSERT INTO Tabellenname VALUES (Wert1, NULL, Wert3...);
```

Variante 2: Ein Attribut auslassen, dieses wird automatisch NULL, sofern kein DEFAULT-Wert gesetzt wird
```SQL
INSERT INTO Tabellenname(Attribut1, Attribut3) VALUES (Wert1, Wert3);
```
-> Attribut2 ist NULL

Beispiel für Tabelle
```SQL
CEATE TABLE Kunde(
    ID INT,
    Name TEXT,
    Age INT
);
```
Der Name wird NULL bei den Befehlen:
```SQL
INSERT INTO Kunde VALUES (1, NULL, 23);
INSERT INTO Kunde(ID, Age) VALUES (1, 23);
```

Beispiel mit DEFAULT-Wert:
```SQL
CEATE TABLE Kunde(
    ID INT,
    Name TEXT DEFAULT 'Anonym',
    Age INT
);
```
Nun wird der Name NULL bei:
```SQL
INSERT INTO Kunde VALUES (1, NULL, 23);
```
Jedoch 'Anonym' bei
```SQL
INSERT INTO Kunde(ID, Age) VALUES (1, 23);
```
# Bearbeiten von Datensätzen

Der UPDATE-Befehl wird für die Änderung von Werten in bestehenden Datensätzen verwendet.

```SQL
UPDATE Tabellenname SET Attribut=Wert WHERE Bedingung
```

Beispiel für folgende Tabelle Benutzer:
| Attribut | Datentyp |
| --- | --- |
| ID | INT |
| Name | VARCHAR(50) |
| Geburtsdatum | DATE |

Setze den Namen für den Benutzer *123* auf *Thomas*
```
UPDATE Benutzer SET Name='Thomas' WHERE ID=123
```

# Auswahl von Datensätzen

Datensätze werden mit dem SELECT-Befehl ausgewählt.

Auswahl von allen Datensätzen in der Tabelle Kunde:
```SQL
SELECT * FROM Kunde
```

Einzelne Attribute/Spalten können anstelle des Asterisk-Symbols mit Komma getrennt werden

```SQL
SELECT Name, ID FROM Kunde
```

Einzelene Datensätze/Zeilen können mit einer WHERE-Clause ausgewählt werden

```SQL
SELECT * FROM Kunde WHERE Name='Thomas'
```

Bei einem Broad-Match (Ungefähr-Vergleich) kann nach Mustern gesucht werden. Die Groß- und Kleinschreibung wird dabei immer ignoriert.

```SQL
SELECT * FROM Kunde WHERE Name LIKE 'tho%'
```

Gängige Platzhalter (Wildcards) sind
| Platzhalter | Bedeutung |
| --- | --- |
| % | 0-unendlich Zeichen |
| _ | Genau 1 Zeichen |

Beispiel um Thomas auszuwählen mit dem _ Platzhalter:

```SQL
SELECT * FROM Kunde WHERE Name LIKE 'tho___'
```

Die Ergebnisse können mittels ORDER BY nach einem oder mehreren Attributen sortiert werden:
```SQL
SELECT * FROM Kunde ORDER BY Name, Alter
```
-> Erst 30-jährige Anna, dann 10-jähriger Thomas, dann 20-jähriger Thomas

LIMIT limitiert die Anzahl an Datensätzen, die Ausgegeben werden. Dabei werden die ersten $n$ Datensätze ausgegeben.
```SQL
SELECT * FROM Kunde LIMIT 3;
```

Wird z.B. nur das Attribut Name ausgegeben, und Thomas soll nicht doppelt vorkommen, wird DISTINCT verwendet
```SQL
SELECT DISTINCT Name FROM Kunde;
```
Distinct bezieht sich dabei **nicht** auf das erste Attribut, sondern auf die gesamte Zeile.

Um Spalten zu verrechnen, werden die einfachen Rechenoperatoren +, -, *, / verwendet. Beispiel für das Berechnen des Bruttos:

```SQL
SELECT Nettogehalt + Abgaben;
```

Da die Überschrift des berechneten Attributes nun die gesamte Berechnung enthält, kann ein Spaltenalias verwendet werden

```SQL
SELECT (Nettogehalt + Abgaben) AS Bruttogehalt;
```

## Aggregatfunktionen
Während mittels den einfachen Rechenoperatoren die Attribute (Spalten) verrechnet werden, können mittels Aggregatfunktionen die Datensätze (Zeilen) miteinander verrechnet werden. Gängige Aggregatfunktionen sind:

| Aggregatfunktion | Beschreibung |
| --- | --- |
| MIN | Minimalwert |
| MAX | Maximalwert |
| AVG | Durchschnitt der Werte |
| SUM | Summe der Werte |
| COUNT | Zählt die Anzahl an Werte |

Standardmäßig werden **alle** Zeilen miteinander aggregiert. Das Ergebnis ist daher nur eine Zeile.

Beispiel Anzahl der Kunden/Datensätze in Tabelle:

```SQL
SELECT COUNT(*) FROM Kunde;
```

Wird ein Attribut anstelle von * gecountet, ist das Ergebnis gleich, sofern kein NULL-Wert vorhanden ist. NULL-Werte werden nicht mitgezählt. (Ähnlich verhält es sich für AVG)

```SQL
SELECT COUNT(Name) AS Anz_Kunden FROM Kunde;
```

Soll die Ausgabe mehr als nur eine Zeile enthalten, aber eine Aggregatfunktion verwendet werden, dann **muss** ein Group By verwendet werden. Das Group By sorgt dafür, dass nicht alle Zeilen auf eine runteraggregiert werden, sondern alle Zeilen innerhalb einer Gruppe auf eine runteraggregiert werden.

```SQL
SELECT COUNT(*) AS Anz_Kunden FROM Kunde GROUP BY Kundentyp;
```

Aggregierte als auch aggregierbare Attribute können mitausgegeben werden. Also insbesondere die Attribute, nach denen gruppiert wird.

```SQL
SELECT Kundentyp, COUNT(*) AS Anz_Kunden FROM Kunde GROUP BY Kundentyp;
```

Achtung: Das Ausgeben von nicht-aggregierten Werten führt zu einem Fehler. Der Grund dafür ist, dass SQL das Ergebnis nicht mehr eindeutig bestimmen kann.

~~SELECT Kundentyp, Name, COUNT(*) AS Anz_Kunden FROM Kunde GROUP BY Kundentyp;~~

Warum geht das nicht? Beispiel für die erwartete Ausgabe:

| Kundentyp | Name | Anz_Kunden |
| --- | --- | --- |
|Privatkunde | ???? WAT SE FACK | 734 |
|Geschäftskunde | ???? WAT SE FACK | 274

Soll z.B. eine Auswahl auf die aggregierten Werte durchgeführt werden, also z.B. nur das COUNT-Ergebnis der Gruppen angezeigt werden, bei denen es über 500 ist, so wird HAVING verwendet.

```SQL
SELECT Kundentyp, COUNT(*) AS Anz_Kunden FROM Kunde 
GROUP BY Kundentyp HAVING Anz_Kunden > 500;
```

Also die logische Reihenfolge der Abarbeitung der Teile im SQL SELECT-Befehl (genannt order of execution) ist

0. JOIN
1. Vorauswahl mit WHERE auf (gejointe oder) ursprüngliche Tabelle
2. Reduzieren der Tabelle auf die im SELECT angegebenen Attribute und bei Aggregatfunktionen dementsprechend auch die Zeilen
3. Nachauswahl mit HAVING auf die gekürzte Tabelle inklusive berechneter Werte

Das sorgt dafür, dass eben die Berechnungen schon feststehen und bedingungen für diese geschrieben werden können, aber auch dass **keine** Bedingungen im Having auf Attribute gesetzt werden können, die nicht im SELECT mit ausgegeben werden.

## Tabellen verbinden

Relevant für den FISI ist nur das "INNER JOIN". Dabei gibt es zwei Varianten, die Tabellen mittels eines INNER JOINS zu verbinden.

### Kartesisches Produkt

Bei dem kartesischen Produkt wird wird jede Zeile aus der einen Tabelle mit jeder Zeile aus der anderen Tabelle kombiniert. Die Angabe erfolgt über eine Auflistung der Tabellen im *FROM*.

```SQL
SELECT * FROM Tabelle1, Tabelle2
```

Hat nun Tabelle1 20 Datensätze und Tabelle2 5 Datensätze, so führt dies zu 100 Datensätzen/Kombinationen im karthesischen Produkt.

Um damit ein *INNER JOIN* umzusetzen, muss zusätzlich eine *WHERE* Bedingungen hinzugefügt werden. Diese setzt Primary Key = Foreign Key. Dadurch werden die Referenzen durch den Foreign Key auf den Primary Key korrekt umgesetzt und ein korrektes INNER JOIN durchgeführt.

```SQL
SELECT * FROM Tabelle1, Tabelle2
WHERE Tabelle1.PK = Tabelle2.FK
```

Da Tabelle 1 20 Datensätze und Tabelle 2 100 Datensätze besitzt, hat das Ergebnis bis zu 100 Ausgaben.

Beispiel:
- Tabelle Kunde mit 20 Kundeninformationen und PK ID
- Tabelle Bestellung mit 100 Bestellinformationen und FK KundeID auf Kunde.ID

-> Ergebnis sind die 100 Bestellinformationen ergänzt um die dazugehörigen Kundeninformationen.

### INNER JOIN

Zu demselben Ergebnis führt das INNER JOIN über den *INNER JOIN* Befehl.

```SQL
SELECT * FROM Tabelle1
INNER JOIN Tabelle2 ON Tabelle1.PK = Tabelle2.FK
```

Das *INNER* (und *OUTER*) kann weggelassen werden.

```SQL
SELECT * FROM Tabelle1
JOIN Tabelle2 ON Tabelle1.PK = Tabelle2.FK
```

Der einzige Vorteil bei dieser Variante ist eine "modularere" Schreibweise. Der JOIN-Befehl ragt nicht in die normalen WHERE-Auswahlbedingungen rein.

### Andere JOIN-Typen

Ein weiterer, für den FISI nicht klausrrelevanter, JOIN Typ ist der *LEFT JOIN*. Bei dem Left Join werden die Datensätze aus der ersten Tabelle beibehalten, selbst wenn kein Treffer zu dem Key in der zweiten Tabelle vorlag. Die Werte, die aus der zweiten Tabelle hätten kommen müssen, werden durch *NULL* ersetzt.

```SQL
SELECT * FROM Kunde
LEFT JOIN Bestellung ON KundenNr = KundenNrFK;
```

| KundenNr | Name | BestellNr | KundenNrFK | Bestelltwert |
| --- | --- | --- | --- | --- |
| 1 | Thomas | 1 | 1 | 70 |
| 1 | Thomas | 2 | 1 | 86 |
| 2 | Anna | 5 | 2 | 99 |
| 3 | Max | NULL | NULL | NULL |

Die letzte Zeile würde bei einem Inner Join entfallen.

Der Große Vorteil: Soll nun geschaut werden, welcher Kunde wie viel bestellt hat, dann wird es ermöglicht auch die Kunden aufzuführen, die nichts bestellt haben. Der Grund dafür ist, dass die Aggregatfunktionen (insb. COUNT) *NULL* Werte nicht beachten. Daher ist dies ein Fall, wo COUNT(*) falsch und z.B. COUNT(BestellNr), COUNT(KundenNrFK) oder COUNT(Bestellwert) richtig wären.

RIGHT JOIN funktioniert genau wie das LEFT JOIN. Blos wird bei dem LEFT JOIN die rechte (zweite) Tabelle an die linke (erste) Tabelle angejoint. Bei dem right join vice versa.

```SQL
SELECT * FROM Kunde
RIGHT JOIN Bestellung ON KundenNr = KundenNrFK;
```

| KundenNr | Name | BestellNr | KundenNrFK | Bestelltwert |
| --- | --- | --- | --- | --- |
| 1 | Thomas | 1 | 1 | 70 |
| 1 | Thomas | 2 | 1 | 86 |
| 2 | Anna | 5 | 2 | 99 |
| NULL | NULL | 12 | 4 | 21 |

-> In diesem Beispiel wäre Kunde 4 gelöscht worden.

Das OUTER JOIN Kombiniert beides. Es joint, was zu joinen ist und fügt hinzu, was in beiden Tabellen ausgeblieben ist.

| KundenNr | Name | BestellNr | KundenNrFK | Bestelltwert |
| --- | --- | --- | --- | --- |
| 1 | Thomas | 1 | 1 | 70 |
| 1 | Thomas | 2 | 1 | 86 |
| 2 | Anna | 5 | 2 | 99 |
| 3 | Max | NULL | NULL | NULL |
| NULL | NULL | 12 | 4 | 21 |