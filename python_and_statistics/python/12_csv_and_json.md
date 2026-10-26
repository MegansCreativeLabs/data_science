# Lektion 12 – CSV och JSON i Python

> **Vecka 3 · Filhantering · Baserad på lärarens genomgång (cirka 18 minuter)**

## Quick Reference

| Vad vill du göra? | Kod | Förklaring |
|---|---|---|
| Importera CSV-stöd | `import csv` | Pythons inbyggda modul för CSV |
| Skriva en CSV-rad | `writer.writerow(["registration", "color"])` | Varje element blir ett fält |
| Läsa CSV | `csv.reader(file)` | Ger en rad i taget som lista av strängar |
| Importera JSON-stöd | `import json` | Pythons inbyggda modul för JSON |
| Spara JSON till fil | `json.dump(data, file, indent=4, ensure_ascii=False)` | Skriver Python-data i JSON-format |
| Läsa JSON från fil | `json.load(file)` | Omvandlar JSON till Python-data |
| Skriva över fil | `open(path, "w", ...)` | **Varning:** tidigare innehåll ersätts |
| Läsa fil | `open(path, "r", ...)` | Läser befintligt innehåll |

## 1. Vad är CSV?

**CSV** betyder *Comma-Separated Values* – kommaseparerade värden. Det är ett textformat för **tabeller**. Varje rad är en post, och värdena på raden separeras av ett tecken, ofta komma.

Exempel på filen `cars.csv`:

```csv
registration,color
ABC123,red
XYZ789,blue
```

- `registration,color` är **rubrikraden** (kolumnnamnen).
- `ABC123,red` är **en datarad**.
- CSV passar bra för enkel tabellformad data, till exempel försäljning, beställningar eller bilar.
- CSV är vanlig text, inte en Excel-arbetsbok, även om Excel kan öppna CSV-filer.

**Viktigt:** Avgränsaren kan också vara semikolon (`;`), särskilt i svenska filer. Då måste du ange `delimiter=";"` i läsaren/skrivaren.

## 2. Skriva en CSV-fil

```python
import csv

with open("files/cars.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerow(["registration", "color"])
    writer.writerow(["ABC123", "red"])
    writer.writerow(["XYZ789", "blue"])
```

### Förstå varje del

1. `import csv` ger oss tillgång till CSV-modulen. Den ingår i Python – du behöver inte installera den med pip.
2. `with open(...) as file` öppnar filen och stänger den automatiskt när blocket är klart.
3. `"files/cars.csv"` är sökvägen. Mappen `files` måste redan finnas.
4. `"w"` betyder *write*: skriv till filen. **Finns filen redan skrivs dess gamla innehåll över.**
5. `newline=""` gör att CSV-modulen kan hantera radbrytningarna korrekt; det är särskilt viktigt i Windows.
6. `encoding="utf-8"` anger hur texttecken lagras, så att exempelvis å, ä och ö kan hanteras.
7. `writer = csv.writer(file)` skapar ett **skrivarobjekt** som vet hur CSV ska formateras.
8. `writer.writerow(...)` skriver **en rad**. Listans element blir separata kolumner.

**Vad är ett skrivarobjekt?** `writer` är bara ett variabelnamn. `csv.writer(file)` skapar ett objekt som vi kan be att skriva CSV-rader med metoden `.writerow()`.

## 3. Läsa en CSV-fil

```python
import csv

with open("files/cars.csv", "r", encoding="utf-8", newline="") as file:
    reader = csv.reader(file)

    for row in reader:
        print(row)
```

Utskrift:

```text
['registration', 'color']
['ABC123', 'red']
['XYZ789', 'blue']
```

**Varför listor?** `csv.reader(file)` tolkar avgränsarna och ger tillbaka en lista för varje rad. Varje fält blir normalt en **sträng**, även om det ser ut som ett tal.

```python
with open("files/cars.csv", "r", encoding="utf-8", newline="") as file:
    reader = csv.reader(file)
    next(reader)  # Hoppa över rubrikraden

    for row in reader:
        print(f"Registrering: {row[0]}, färg: {row[1]}")
```

- `row[0]` är första kolumnen, registreringsnumret.
- `row[1]` är andra kolumnen, färgen.
- `next(reader)` används här för att hoppa över rubrikerna. Detta är ett **extra tips**, inte en central del av lärarens exempel.

## 4. Vad är JSON?

**JSON** betyder *JavaScript Object Notation*. Det är ett textformat för att spara och utbyta data. Det används ofta i **API-svar**, inställningsfiler och data som innehåller flera nivåer.

JSON påminner om en Python-dictionary:

```python
person = {
    "name": "Joakim",
    "age": 38,
    "city": "Västerås",
    "is_married": True
}
```

Samma data som JSON:

```json
{
    "name": "Joakim",
    "age": 38,
    "city": "Västerås",
    "is_married": true
}
```

**Skillnaden att lägga märke till:** Python skriver `True`, medan JSON skriver `true`. Python har även `False`/`None`, där JSON använder `false`/`null`.

JSON kan innehålla objekt, listor (arrays), strängar, tal, booleska värden och `null`. Det behöver alltså inte vara just en dictionary.

## 5. Skriva JSON till en fil

```python
import json

person = {
    "name": "Joakim",
    "age": 38,
    "city": "Västerås",
    "is_married": True
}

with open("files/person.json", "w", encoding="utf-8") as file:
    json.dump(person, file, indent=4, ensure_ascii=False)
```

Förklaring:

| Del | Vad betyder den? |
|---|---|
| `json.dump(...)` | Spara Python-data som JSON **i en fil** |
| `person` | Objektet/datan som ska sparas |
| `file` | Filen som ska få innehållet |
| `indent=4` | Indrag på fyra mellanslag så filen blir lättläst |
| `ensure_ascii=False` | Skriv exempelvis `ä` som ett vanligt tecken i stället för en Unicode-escape |
| `encoding="utf-8"` | Spara textfilen med UTF-8-kodning |

**Varför både UTF-8 och `ensure_ascii=False`?** UTF-8 bestämmer hur tecken kodas i filen. `ensure_ascii=False` styr hur JSON-modulen **skriver** tecken som å, ä och ö. Det är två olika saker.

## 6. Läsa JSON från en fil

```python
import json

with open("files/person.json", "r", encoding="utf-8") as file:
    person = json.load(file)

print(person)
print(person["name"])
```

Här sker två steg:

1. `json.load(file)` läser JSON-texten och omvandlar den till ett Python-objekt.
2. Eftersom filen innehåller ett JSON-objekt blir resultatet i det här exemplet en **dictionary**. Då kan vi använda nyckeln `"name"`.

Om JSON-filen i stället innehåller en lista, får vi en Python-lista tillbaka.

### `dump` eller `load`?

```text
Python-dictionary -- json.dump() --> person.json
Python-dictionary <-- json.load() --- person.json
```

Kom ihåg: **dump = spara**, **load = läsa in**.

## 7. CSV eller JSON – vilket ska jag välja?

| CSV | JSON |
|---|---|
| Enkel tabell med rader och kolumner | Data med nycklar, listor och flera nivåer |
| Exempel: orderlista, försäljningsdata | Exempel: API-svar, inställningar, personuppgifter |
| Läses med `csv.reader()` | Läses med `json.load()` |
| Skrives med `csv.writer()` | Skrives med `json.dump()` |

Lärarens huvudpoäng: **CSV är praktiskt för tabeller; JSON är praktiskt för mer komplexa datastrukturer.** JSON är inte nödvändigtvis *ostrukturerad* data – ofta kallas det **semistrukturerad data**. Det sista är ett förtydligande utöver lärarens ordval.

## 8. Vanliga misstag

| Problem | Varför händer det? | Lösning |
|---|---|---|
| Extra tomma rader i CSV på Windows | Radbrytningar hanteras dubbelt | Använd `newline=""` när du öppnar CSV |
| `FileNotFoundError` | Filen/sökvägen finns inte | Kontrollera arbetsmapp och filnamn |
| `row[1]` ger fel | Raden har färre än två fält | Kontrollera filens struktur |
| `"w"` tar bort tidigare data | Skrivläget ersätter innehållet | Använd rätt läge och säkerhetskopiera viktig data |
| Å/ä/ö visas konstigt i JSON | JSON-tecken har escapats | Använd `ensure_ascii=False` och UTF-8 |
| `json.load()` ger fel | Filen innehåller ogiltig JSON | Kontrollera syntax, citattecken och kommatecken |
| CSV hamnar i en enda kolumn | Fel avgränsare | Ange exempelvis `delimiter=";"` |

## 9. Miniövningar

**Övning 1.** Vad betyder CSV och vad används det till?

**Övning 2.** Vad gör `newline=""` och varför är det extra viktigt i Windows?

**Övning 3.** Vad är skillnaden mellan `csv.writer(file)` och `writer.writerow([...])`?

**Övning 4.** Varför används `json.dump()` när man sparar och `json.load()` när man läser?

**Övning 5.** Skriv kod som sparar en dictionary med `name` och `city` till `files/student.json` och sedan läser in den.

## Facit

**1.** *Comma-Separated Values*. Ett textformat för tabellformad data där värden på en rad skiljs åt med en avgränsare.

**2.** Det låter CSV-modulen hantera radbrytningarna och förebygger extra tomrader, särskilt på Windows.

**3.** `csv.writer(file)` **skapar** ett skrivarobjekt. `.writerow([...])` **använder** det objektet för att skriva en rad.

**4.** `dump()` omvandlar Python-data till JSON och skriver i filen; `load()` läser JSON från filen och omvandlar till Python-data.

**5.**

```python
import json

student = {"name": "Sara", "city": "Helsingborg"}

with open("files/student.json", "w", encoding="utf-8") as file:
    json.dump(student, file, indent=4, ensure_ascii=False)

with open("files/student.json", "r", encoding="utf-8") as file:
    loaded_student = json.load(file)

print(loaded_student["name"])
```

*Mappen `files` måste finnas innan koden körs.*

---

### Begreppslista

- **Filformat:** Regler för hur information sparas i en fil.
- **CSV:** Textformat för tabellformad data.
- **JSON:** Textformat för strukturerad/semistrukturerad data.
- **Modul:** Python-kod som kan importeras, exempelvis `csv` och `json`.
- **Skrivarobjekt:** Ett objekt som kan skriva CSV-rader till en fil.
- **Läsarobjekt:** Ett objekt som kan tolka CSV-rader från en fil.
- **Dictionary:** Python-samling av nyckel–värde-par.
- **Encoding:** Teckenkodning, exempelvis UTF-8.
- **Indrag (`indent`):** Mellanslag som gör JSON mer lättläst.

**Källa:** Lärarens videogenomgång *CSV och JSON* (transkribering delad av användaren). Extra förtydliganden har märkts ut i texten där de går utöver genomgången.
