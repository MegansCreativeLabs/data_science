# Komplettering till lektion 11 – Skriva till textfiler i Python

> **KOMPLETTERANDE LEKTION – EJ VERIFIERAD MOT KURSMATERIALET**  
> Detta är en självständig genomgång av filskrivning. Den bygger **inte** på en transkribering av lärarens video om att skriva textfiler och är inte jämförd med kursplanen. Filen heter **11b** för att inte ändra numreringen på de befintliga lektionerna.

## Målet
Kunna skapa en textfil, skriva text, lägga till nya rader och förstå skillnaden mellan lägena `w`, `a` och `x`.

## Snabbguide

| Kod | Betydelse |
|---|---|
| `open("fil.txt", "w")` | Skrivläge: skapar filen eller skriver över befintligt innehåll |
| `open("fil.txt", "a")` | Lägg-till-läge: skriver sist i filen |
| `open("fil.txt", "x")` | Skapar ny fil; ger fel om den redan finns |
| `file.write("Hej")` | Skriver text och returnerar antal skrivna tecken |
| `\n` | Radbrytning |
| `encoding="utf-8"` | Hanterar bland annat svenska tecken |
| `with open(...) as file:` | Stänger filen automatiskt efter blocket |

## 1. Skapa en textfil

```python
with open("anteckningar.txt", "w", encoding="utf-8") as file:
    file.write("Min första anteckning\n")
```

**Rad för rad:**

1. `with` sköter stängningen av filen automatiskt.
2. `"anteckningar.txt"` är filnamnet. Utan annan sökväg hamnar filen i programmets aktuella arbetskatalog.
3. `"w"` betyder **write** – skriv. Om filen redan finns skrivs dess gamla innehåll över.
4. `encoding="utf-8"` hjälper oss att spara svenska bokstäver korrekt.
5. `file.write(...)` skriver texten. `\n` gör att nästa text börjar på en ny rad.

**Viktigt:** `w` kan radera gammalt innehåll. Testa på en övningsfil, inte på något du vill behålla.

## 2. Lägg till text utan att radera det gamla

```python
with open("anteckningar.txt", "a", encoding="utf-8") as file:
    file.write("En andra anteckning\n")
```

`"a"` står för **append**. Den nya texten läggs efter det befintliga innehållet. Filen skapas om den inte redan finns.

## 3. Skriva flera rader med en loop

```python
uppgifter = ["Läsa Python", "Öva Pandas", "Repetera Git"]

with open("att_gora.txt", "w", encoding="utf-8") as file:
    for uppgift in uppgifter:
        file.write(uppgift + "\n")
```

`for` går igenom varje text i listan. `+ "\n"` lägger till en radbrytning efter varje uppgift.

## 4. Läsa tillbaka det vi skrivit

```python
with open("att_gora.txt", "r", encoding="utf-8") as file:
    innehall = file.read()

print(innehall)
```

`"r"` betyder **read**. Vi läser filen efter att skrivningen har avslutats.

## 5. Skriv ett värde som inte är text

```python
antal = 5

with open("resultat.txt", "w", encoding="utf-8") as file:
    file.write(f"Antal: {antal}\n")
```

`write()` förväntar sig en **sträng**. Därför använder vi en f-sträng för att kombinera text och ett tal. `file.write(antal)` skulle ge `TypeError`.

## 6. Skapa bara om filen inte redan finns

```python
try:
    with open("ny_fil.txt", "x", encoding="utf-8") as file:
        file.write("Ny fil!\n")
except FileExistsError:
    print("Filen finns redan, så inget skrevs över.")
```

`"x"` är bra när vi vill undvika att råka skriva över en befintlig fil.

## 7. Vanliga misstag

- **Gammal text försvinner:** du använde `w` i stället för `a`.
- **All text hamnar på samma rad:** du glömde `\n`.
- **`TypeError`:** du försökte skriva ett tal direkt med `write()`.
- **Filen syns inte där du väntade dig:** kontrollera aktuell arbetskatalog med `import os; print(os.getcwd())`.
- **Mappen finns inte:** `open()` skapar filer, men inte automatiskt saknade mappar.

## 8. Övningar

1. Skapa `dagbok.txt` och skriv en rad.
2. Kör programmet igen med `a` och lägg till en rad utan att ta bort den första.
3. Spara tre namn från en lista, ett per rad.
4. Läs filen och skriv ut innehållet.

### Facit

```python
with open("dagbok.txt", "w", encoding="utf-8") as file:
    file.write("Idag övade jag Python.\n")

with open("dagbok.txt", "a", encoding="utf-8") as file:
    file.write("Jag lärde mig att använda append.\n")

namn = ["Anna", "Bo", "Cecilia"]
with open("namn.txt", "w", encoding="utf-8") as file:
    for person in namn:
        file.write(person + "\n")

with open("namn.txt", "r", encoding="utf-8") as file:
    print(file.read())
```

---
**Kursplansstatus:** Ej verifierad mot lärarens video eller kursplan. Denna fil är en fristående komplettering till lektion 11.
