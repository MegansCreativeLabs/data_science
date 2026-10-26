# 11 – Läsa filer i Python

> **Vecka 3 · Filhantering · Del 1**  
> Baserad på lärarens genomgång om att läsa textfiler (cirka 38 minuter). **CSV, JSON och att skriva till filer ingår inte i just den här videon** och behandlas separat.

## Quick Reference

| Kod | Vad den gör |
|---|---|
| `open("files/example.txt", "r", encoding="utf-8")` | Öppnar en textfil för läsning |
| `with open(...) as file:` | Öppnar och stänger filen automatiskt |
| `file.read()` | Läser återstående innehåll som en sträng |
| `file.readline()` | Läser nästa rad |
| `file.readlines()` | Läser återstående rader till en lista |
| `for line in file:` | Går igenom filen rad för rad |
| `file.seek(0)` | Flyttar läspositionen till början |
| `line.strip()` | Tar bort blanksteg och radbrytningar i början och slutet |
| `enumerate(file, start=1)` | Ger radnummer och rad samtidigt |
| `except FileNotFoundError:` | Hanterar att filen saknas |

## 1. Vad betyder filhantering?

Hittills har många Python-exempel arbetat med värden som skrivs direkt i koden. **Filhantering** betyder att programmet kan läsa information som ligger sparad på datorn, exempelvis i en `.txt`-fil.

Anta att projektet har följande mappar:

```text
mitt_projekt/
├── main.py
└── files/
    ├── example.txt
    └── numbers.txt
```

`files/example.txt` är en **relativ sökväg**: den utgår normalt från programmets aktuella arbetskatalog, inte nödvändigtvis från platsen där Python-filen ligger. I VS Code/Jupyter kan arbetskatalogen därför vara viktig.

## 2. Öppna, läs och stäng en fil

Läraren visar först det manuella sättet:

```python
file = open("files/example.txt", "r", encoding="utf-8")
content = file.read()
file.close()
print(content)
```

- `open(...)` öppnar filen och returnerar ett **filobjekt**.
- `"r"` betyder **read**, alltså läsläge. Det är standardläget om inget anges.
- `encoding="utf-8"` anger hur textens tecken ska tolkas, till exempel å, ä och ö.
- `file.read()` läser texten och ger tillbaka en **sträng**.
- `file.close()` stänger filen.

**Varför stänga?** Ett öppet filobjekt använder resurser. Om ett fel inträffar före `close()` kanske filen inte stängs som tänkt.

### Rekommenderat: `with open(...)`

```python
with open("files/example.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

**Vad betyder delarna?**

- `with` skapar ett sammanhang där filen hålls öppen.
- `as file` ger filobjektet ett variabelnamn som vi själva väljer.
- Den indragna koden körs medan filen är öppen.
- När `with`-blocket avslutas stängs filen automatiskt, även om ett fel uppstår inne i blocket.

Variabeln `content` går fortfarande att använda efter blocket. Ett `with`-block skapar inte ett eget variabel-scope i Python.

## 3. Tre sätt att läsa text

### `read()` – hela återstående filen

```python
with open("files/example.txt", encoding="utf-8") as file:
    text = file.read()
```

Resultatet är **en enda sträng**. Smidigt för små filer, men kan använda mycket minne för stora filer.

### `readline()` – en rad i taget

```python
with open("files/example.txt", encoding="utf-8") as file:
    first_line = file.readline()
    second_line = file.readline()

print(first_line)
print(second_line)
```

Varje anrop fortsätter från filens aktuella **läsposition**. Metoden börjar alltså inte om från första raden varje gång.

### `readlines()` – en lista med rader

```python
with open("files/example.txt", encoding="utf-8") as file:
    lines = file.readlines()

print(lines)
```

Resultatet är en lista av strängar, ofta med `\n` i slutet av varje rad. Observera skillnaden: **`readline()` (singular)** läser en rad; **`readlines()` (plural)** läser alla återstående rader.

## 4. Läspositionen och `seek(0)`

Tänk dig att filen har en osynlig markör som flyttas framåt när du läser.

```python
with open("files/example.txt", encoding="utf-8") as file:
    first = file.readline()
    second = file.readline()
    file.seek(0)  # Tillbaka till filens början
    first_again = file.readline()
```

`seek(0)` går tillbaka till början. För textfiler ska du **inte** generellt tänka att `seek(10)` betyder ”gå till tecken nummer 10”; positionerna är inte alltid samma sak som teckenpositioner, särskilt med UTF-8.

## 5. Loopa över filen – bra för större filer

```python
with open("files/example.txt", encoding="utf-8") as file:
    for line in file:
        print(line, end="")
```

Här läser Python filen **rad för rad**, i stället för att först skapa en lista med hela filens innehåll. `end=""` gör att `print()` inte lägger till en extra radbrytning; textfilens egna radbrytningar finns vanligtvis redan kvar.

### Vad är `\n`, `strip()` och `rstrip()`?

`\n` betyder **ny rad**. Om en rad innehåller `"Hej\n"` och `print()` också lägger till en ny rad kan utskriften få tomma rader emellan.

```python
line = "  Hej!  \n"
print(line.strip())    # "Hej!"
print(line.rstrip())   # "  Hej!"
```

- `.strip()` tar bort blanktecken **i båda ändar**.
- `.rstrip()` tar bort blanktecken **i slutet**.
- Ingen av dem tar bort vanliga mellanslag mitt inne i texten.

## 6. Numrera rader med `enumerate()`

```python
with open("files/example.txt", encoding="utf-8") as file:
    for number, line in enumerate(file, start=1):
        print(f"{number}: {line.rstrip()}")
```

`enumerate()` ger två saker vid varje varv: ett nummer och ett värde. Här är värdet en textrad. `start=1` betyder att första raden får nummer 1 i stället för 0.

## 7. Hantera en fil som inte finns

```python
try:
    with open("files/missing.txt", encoding="utf-8") as file:
        content = file.read()
    print(content)
except FileNotFoundError:
    print("Filen hittades inte. Kontrollera filnamn och sökväg.")
```

- `try` betyder ”försök köra denna kod”.
- `except FileNotFoundError` körs **om just det felet uppstår**.
- Programmet kan då visa ett begripligt meddelande i stället för att avbrytas av ett ohanterat fel.

Det fångar inte automatiskt andra fel, som att filen saknar rätt behörigheter eller att ett tal har fel format.

## 8. Praktiskt exempel: medelvärde från `numbers.txt`

Anta att `files/numbers.txt` innehåller:

```text
10
20

30
40
```

Vi vill läsa talen, hoppa över tomma rader och beräkna medelvärdet.

```python
values = []

with open("files/numbers.txt", "r", encoding="utf-8") as numbers_file:
    for line in numbers_file:
        line = line.strip()

        if line:  # Körs bara om raden inte är tom
            values.append(float(line))

if values:
    average = sum(values) / len(values)
    print(f"Medelvärde: {average}")
else:
    print("Inga tal i filen.")
```

**Steg för steg:**

1. `values = []` skapar en tom lista.
2. `with open(...)` öppnar filen för läsning.
3. `for line in numbers_file` går igenom en rad i taget.
4. `line.strip()` tar bort bland annat radbrytningen.
5. `if line:` kontrollerar att något återstår efter rensningen.
6. `float(line)` gör om texten `"10"` till talet `10.0`.
7. `.append(...)` lägger till talet i listan.
8. `if values:` ser till att listan inte är tom innan vi delar med dess längd.
9. `sum(values) / len(values)` räknar ut medelvärdet: summan delad med antalet tal.

**Resultat:** `Medelvärde: 25.0`

**Viktig precisering:** `float()` klarar normalt radbrytningar och omgivande blanksteg direkt. `strip()` är ändå användbart för att upptäcka tomma rader och göra texten tydligare. Om filen innehåller `hej` i stället för ett tal får du `ValueError`; det hanteras inte i just lärarens exempel.

## 9. Vanliga misstag

| Problem | Varför? | Lösning |
|---|---|---|
| `FileNotFoundError` | Fel sökväg eller arbetskatalog | Kontrollera var programmet körs och var filen finns |
| `read()` ger tom sträng andra gången | Läspositionen är redan i slutet | Använd `seek(0)` innan ny läsning |
| Tomma rader mellan utskrifterna | Både filen och `print()` ger radbrytning | Använd `end=""` eller `rstrip()` |
| `ValueError` vid `float(line)` | Raden är tom eller innehåller ogiltig text | Kontrollera/rensa innan konvertering |
| Svenska tecken visas fel | Teckenkodning stämmer inte | Kontrollera filens faktiska kodning; använd UTF-8 när filen är UTF-8 |
| `readline()` förväxlas med `readlines()` | Nästan samma namn | En rad respektive lista med rader |

## 10. Ordlista

| Begrepp | Förklaring |
|---|---|
| Fil | Sparad information på datorn |
| Sökväg | Adressen till en fil |
| Relativ sökväg | Adress utifrån aktuell arbetskatalog |
| Filobjekt | Objektet som `open()` returnerar och som vi läser genom |
| Läsläge (`r`) | Öppna en fil för läsning |
| Encoding | Hur texttecken lagras och tolkas |
| UTF-8 | Vanlig teckenkodning som hanterar svenska tecken |
| Sträng (`str`) | Textvärde i Python |
| Iteration | Ett varv i en loop |
| Exception | Fel som Python kan signalera och som kan hanteras |

## 11. Övningar

1. Öppna `files/example.txt` med `with open()` och skriv ut hela innehållet.
2. Skriv ut bara den första raden i filen.
3. Skriv ut filens rader med radnummer från 1 utan extra tomrader.
4. Läs talen i `files/numbers.txt` och skriv ut hur många tal som finns.
5. Vad är skillnaden mellan `readline()` och `readlines()`?
6. Varför är `with open()` säkrare än att själv skriva `close()`?

### Facit

**1.**
```python
with open("files/example.txt", encoding="utf-8") as file:
    print(file.read())
```

**2.**
```python
with open("files/example.txt", encoding="utf-8") as file:
    print(file.readline().rstrip())
```

**3.**
```python
with open("files/example.txt", encoding="utf-8") as file:
    for number, line in enumerate(file, start=1):
        print(f"{number}: {line.rstrip()}")
```

**4.**
```python
values = []
with open("files/numbers.txt", encoding="utf-8") as file:
    for line in file:
        line = line.strip()
        if line:
            values.append(float(line))
print(len(values))
```

**5.** `readline()` ger nästa rad som sträng; `readlines()` ger återstående rader som en lista av strängar.

**6.** `with` stänger filen automatiskt när blocket avslutas, även vid fel.

---

### Vad kommer härnäst?

Den här genomgången handlar **bara om att läsa textfiler**. Läraren avslutar med att nästa video handlar om att **skriva till filer**. Därefter kan vi bygga vidare med CSV och JSON, som också ingår i kursens vecka 3.
