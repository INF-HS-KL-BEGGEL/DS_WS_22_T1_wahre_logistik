# Der Datensatz




## Der Datensatz selbst
... ist verfügbar unter [Google Drive](https://drive.google.com/file/d/1QXE9-UqFT1xJcDpKl78k5qg7dnN_ThyK/view?usp=sharing).
Link zur zugehörigen [Vorlesung](https://martins-wahre-logistik.blogspot.com/2022/10/logistics-case-studies-der-lieferschein.html)

Der original Datensatz umfastt 7 Tabellen {Q1, Q2, Q3, Q4, ABC-Zugriffe, Materialstämme, Kunden Stammdaten}.
Dabei beinhalten Q1 - Q4 die selben Spalten und umfassen zusammen über 500.000 Datensätze.

## PgAdmin und die Postgres Datenbank
ToDo: Hier stuff zum Aufbau der Datenbank und der Connection

## Import der Excel Daten in die Datenbank
Hierzu wurde die Definitonen der Tabellen in die Datenbank überführt und dann die Excel Tabellen in CSV Dateien formatiert, um sie anscließend über das Kontextmenü von PgAdmin zu importieren. Hierbei wurden die Tabellen Q1 - Q4 zusammengeführt zu "Verkaeufe" und eine neue Spalte ergänzt für die Quartalsnummer.
Beim Import sind mehrere Probleme aufgetreten aufgrund inkonsistenter Daten in der Excel.

Probleme:
1. Es existieren in Verkaeufe mehrere Dublikate ganzer Zeilen. Aufgrund der Funktionsweise einer relationalen Datenbank können diese ohnehin nicht überführt werden. Jedoch führt dies zu einer Fehlermeldung beim Import und das Dublikat kann nicht beim Import selbst entfernt werden. D.h. die Dublikate mussten im Vorfeld aus dem Datensatz gefiltert werden. Dazu ist eine Suche auf über 500.000 Zeilen erforderlich. Dies wurde selbstverständlich **nicht** händisch vorgenommen, statt dessen wurde ein kurzes Python Script eingesetzt.

```python
input =open("Verkauf.csv","r") # read the CSV file
lines=input.readlines()
output=open("VerkaufNew.csv","w")# write the result into a new CSV file

dublicates=0 # The actual amount of dublicate values
dublicateLines={} # The accurance of the dublicates as line numbers inside the csv file
lineNumber=0 # pointer towards the line within the csv file
otherLineNumber=0 # pointer towards the line within the csv file, which is getting compared to the first one
maxDepth=10 # maximal search depth - how much lines are getting compared
copys=[] # a container for all lines with dublicates

#select all lines to be checked for dublicates
for line in lines:
    lineNumber+=1
    copyExists=False
    otherLineNumber=lineNumber-1
    maxDepth=10 #commonly only following lines are dublicates

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
            #protocol the dublicate values
            dublicates+=1
            dublicateLines[dublicates]=lineNumber,otherLineNumber
            copyExists=True
            print("copy at: "+str(lineNumber)+" and "+str(otherLineNumber))
            break
        #stop if searchdepth exceeded
        if maxDepth==0:
            break

    if not copyExists and line not in copys:
        output.write(line)
```
Aus Performance Gründen wurde kein simples Brute-Force angewandt, das dies zu 500.000 * 500.000 Vergleichsoperationen führen würde. Stattdessen verlässt sich der Code auf die Tatsache das die Dublikate direkt auf die original Zeilen folgen. Dies legt Nahe, dass es sich im Datensatz um ein Copy-Paste Error handelt. Dieser Fehler ist zudem systematisch und tritt 5650 mal auf.

2. Die Tabelle "Materialstaemme" ist nicht vollständig. Zeile 20277 beinhlatte statt eigentlich nunmerischen Werten einen Platzhalter '#ZAHL!'. Dieser wurde der fehlenden Werte auf NULL in der Datenbank gesetzt.

###### weitere Auffäligkeiten, die nicht direkt zu Fehlern führen
3. Entgegen der Annahme, dass die Artikelnummer in der Tabelle "ABC-Zugriffe" als Fremdschlüssel auf die Artikelnummern in der Tabelle "Materialstaemme" verweist, ist dies nicht der Fall, da nicht alle Artikelnummern in der Referenz Tabelle vorhanden sind.

4. Die Spalte "Abgang" in der Tabelle "Verkaeufe" beinhaltet nicht auschließlich ganzzahlige Werte, sondern auch Fließkommazahlen.

5. Die Spalte "Materialnummer" in der Tabelle "Verkaeufe" beinhaltet nicht auschließlich nunmerische Werte, sondern auch vereinzelt Characters.
