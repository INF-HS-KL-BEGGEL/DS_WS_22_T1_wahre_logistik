# Der Datensatz




## Der Datensatz selbst
... ist verfügbar unter [Google Drive](https://drive.google.com/file/d/1QXE9-UqFT1xJcDpKl78k5qg7dnN_ThyK/view?usp=sharing).
Link zur zugehörigen [Vorlesung](https://martins-wahre-logistik.blogspot.com/2022/10/logistics-case-studies-der-lieferschein.html)

Der original Datensatz umfasst 7 Tabellen {Q1, Q2, Q3, Q4, ABC-Zugriffe, Materialstämme, Kunden Stammdaten}.
Dabei beinhalten Q1 - Q4 die selben Spalten und umfassen zusammen über 500.000 Datensätze.

## PgAdmin und die Postgres Datenbank
ToDo: Hier stuff zum Aufbau der Datenbank und der Connection

## Import der Excel Daten in die Datenbank
Hierzu wurde die Definitionen der Tabellen in die Datenbank überführt und dann die Excel Tabellen in CSV Dateien formatiert, um sie anschließend über das Kontextmenü von PgAdmin zu importieren. Hierbei wurden die Tabellen Q1 - Q4 zusammengeführt zu "Verkaeufe" und eine neue Spalte ergänzt für die Quartalsnummer.
Beim Import sind mehrere Probleme aufgetreten aufgrund inkonsistenter Daten in der Excel.

Probleme:
1. Es existieren in Verkaeufe mehrere Duplikate ganzer Zeilen. Aufgrund der Funktionsweise einer relationalen Datenbank können diese ohnehin nicht überführt werden. Jedoch führt dies zu einer Fehlermeldung beim Import und das Duplikat kann nicht beim Import selbst entfernt werden. D.h. die Duplikate mussten im Vorfeld aus dem Datensatz gefiltert werden. Dazu ist eine Suche auf über 500.000 Zeilen erforderlich. Dies wurde selbstverständlich **nicht** händisch vorgenommen, statt dessen wurde ein kurzes Python Skript eingesetzt.

```python
input =open("Verkauf.csv","r") # read the CSV file
lines=input.readlines()
output=open("VerkaufNew.csv","w")# write the result into a new CSV file

duplicates=0 # The actual amount of duplicate values
duplicateLines={} # The accurance of the duplicates as line numbers inside the csv file
lineNumber=0 # pointer towards the line within the csv file
otherLineNumber=0 # pointer towards the line within the csv file, which is getting compared to the first one
maxDepth=10 # maximal search depth - how much lines are getting compared
copys=[] # a container for all lines with duplicates

#select all lines to be checked for duplicates
for line in lines:
    lineNumber+=1
    copyExists=False
    otherLineNumber=lineNumber-1
    maxDepth=10 #commonly only following lines are duplicates

    #only compare the upfollowing <maxDepth> lines
    for otherLineNumber in range (lineNumber,lineNumber+10):
        # stop if file end is reached
        if otherLineNumber>len(lines):
            break
        #select the line to compare with
        otherLine=lines[otherLineNumber-1]
        maxDepth-=1
        #check if the lines are identical and are obviously not one and the same
        if line==otherLine and lineNumber!=otherLineNumber:
            #protocol the duplicate values
            duplicates+=1
            duplicateLines[duplicates]=lineNumber,otherLineNumber
            copyExists=True
            print("copy at: "+str(lineNumber)+" and "+str(otherLineNumber))
            break
        #stop if searchdepth exceeded
        if maxDepth==0:
            break

    if not copyExists and line not in copys:
        output.write(line)
```
Aus Performance Gründen wurde kein simples Brute-Force angewandt, das dies zu 500.000 * 500.000 Vergleichsoperationen führen würde. Stattdessen verlässt sich der Code auf die Tatsache das die Duplikate direkt auf die original Zeilen folgen. Dies legt Nahe, dass es sich im Datensatz um ein Copy-Paste Error handelt. Dieser Fehler ist zudem systematisch und tritt 5650 mal auf.

2. Die Tabelle "Materialstaemme" ist nicht vollständig. Zeile 20277 beinhaltete statt eigentlich numerischen Werten einen Platzhalter '#ZAHL!'. Dieser wurde der fehlenden Werte auf NULL in der Datenbank gesetzt.

###### weitere Auffälligkeiten, die nicht direkt zu Fehlern führen
3. Entgegen der Annahme, dass die Artikelnummer in der Tabelle "ABC-Zugriffe" als Fremdschlüssel auf die Artikelnummern in der Tabelle "Materialstaemme" verweist, ist dies nicht der Fall, da nicht alle Artikelnummern in der Referenz Tabelle vorhanden sind.

4. Die Spalte "Abgang" in der Tabelle "Verkaeufe" beinhaltet nicht ausschließlich ganzzahlige Werte, sondern auch Fließkommazahlen.

5. Die Spalte "Materialnummer" in der Tabelle "Verkaeufe" beinhaltet nicht ausschließlich numerische Werte, sondern auch vereinzelt Characters.

## Anpassen der Daten für weitere Bearbeitung in R

### Erstellen einer neuen Spalte für numerische Preise in Verkauefe

```sql
ALTER TABLE IF EXISTS public.verkaeufe ADD COLUMN vk_preis_num numeric;
```

### Kopieren der *String* Preis Spalte (vk_preis) in die *Numeric* Preis Spalte (vk_preis_num)
Alle Punktzeichen außer das Letzte, Leerzeichen und Eurozeichen werden vor dem Cast in einen numerischen Wert entfernt und dann in die neue Spalte eingetragen.
```sql
UPDATE public.verkaeufe SET vk_preis_num = REGEXP_REPLACE(vk_preis, '(\.(?=[^.]*\.)|\s|€)', '', 'g')::DECIMAL;
```
