# Allgemein
Legen Sie alle Dateien, die dem Speichern von Umgebungsvariablen dienen, in diesem Directory ab.

**Wichtig:** Achten Sie immer darauf, dass die Dateien, die Sie hier ablegen, nicht in der Git-Versionshistorie auftauchen. Sonst könnten sensible Informationen (z. B. Credentials) geleaked werden. Gegebenenfalls muss hierfür die [gitignore-File](../.gitignore) ergänzt werden.

# Beispiel
In R-Projekten werden Umgebungsvariablen gerne in .Renviron-Files abgelegt. Diese haben beispielsweise folgende Struktur:
```
vname=John
lname=Doe
age=42
```