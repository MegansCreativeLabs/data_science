# Lesson 06 – Lists

> **Python & Statistics | MegansCreativeLabs**  
> Pedagogiska anteckningar utifrån lärarens genomgång av listor i Python.

## Quick Reference – snabb uppslagsguide

| Vad vill du göra? | Python | Förklaring |
|---|---|---|
| Skapa en lista | `fruits = ["apple", "banana"]` | Flera värden i en variabel |
| Första elementet | `fruits[0]` | Index börjar på 0 |
| Sista elementet | `fruits[-1]` | Negativt index räknar bakifrån |
| Del av lista | `fruits[0:2]` | Start ingår, slut ingår inte |
| Ändra ett värde | `fruits[0] = "pear"` | Listor är mutabla |
| Lägg till sist | `fruits.append("kiwi")` | Ändrar den befintliga listan |
| Ta bort sista | `fruits.pop()` | Tar bort och returnerar sista värdet |
| Ta bort ett värde | `fruits.remove("banana")` | Tar bort första matchningen |
| Ta bort på index | `del fruits[0]` | Tar bort element på viss plats |
| Antal element | `len(fruits)` | Listans längd |
| Räkna förekomster | `fruits.count("kiwi")` | Hur många gånger värdet finns |
| Minsta/största | `min(numbers)` / `max(numbers)` | Extremvärden |
| Sorterad kopia | `sorted(numbers)` | Ursprunglig lista behålls |
| Sortera listan | `numbers.sort()` | Ursprunglig lista ändras |
| Slå ihop listor | `first + second` | Skapar en ny lista |
| Medelvärde/median | `statistics.mean(numbers)` / `statistics.median(numbers)` | Kräver `import statistics` |
| Värde i nästlad lista | `matrix[1][2]` | Först inre lista, sedan element |

---

## 1. Vad är en lista?

En **lista (`list`)** är en datatyp som samlar flera värden i **en enda variabel**. Tänk på den som en rad numrerade fack: varje fack innehåller ett värde.

```python
fruits = ["apple", "banana", "orange"]
print(fruits)
# ['apple', 'banana', 'orange']
```

- `fruits` är variabelns namn.
- `[` och `]` markerar början och slutet på listan.
- Kommatecken skiljer elementen åt.
- Varje frukt är en **string**, därför står den inom citattecken.

En lista kan innehålla olika datatyper, men läraren rekommenderar att hålla liknande värden tillsammans när det är praktiskt. Det blir enklare att bearbeta dem senare.

## 2. Indexering – hitta ett element

Python börjar räkna positioner från **0**, inte 1.

```python
fruits = ["apple", "banana", "orange"]

print(fruits[0])   # apple
print(fruits[1])   # banana
print(fruits[2])   # orange
print(fruits[-1])  # orange
```

| Index | 0 | 1 | 2 |
|---|---|---|---|
| Värde | apple | banana | orange |

**Kom ihåg:** `fruits[1]` betyder *andra elementet*, inte första. `-1` är sista elementet, `-2` näst sista.

## 3. Slicing – hämta flera element

En **slice** tar ut en del av listan med syntaxen `lista[start:stop]`.

```python
fruits = ["apple", "banana", "orange", "kiwi"]
print(fruits[0:2])  # ['apple', 'banana']
print(fruits[1:3])  # ['banana', 'orange']
```

**Varför ingår inte sista positionen?** I `[0:2]` börjar Python på index `0` och stannar **före** index `2`. Det blir alltså index `0` och `1` – två element. Det gör att `stop - start` ger antalet element i en vanlig slice.

Slicing **skapar en ny lista** med de utvalda elementen. Den ursprungliga listan ändras inte av att du bara läser en slice.

## 4. Mutabla listor – du kan ändra innehållet

**Mutabel (`mutable`)** betyder att ett objekt kan förändras efter att det skapats. Listor är mutabla, till skillnad från strings som inte kan ändras tecken för tecken.

```python
fruits = ["apple", "banana", "orange"]
fruits[0] = "pear"
print(fruits)
# ['pear', 'banana', 'orange']
```

Här ersätter vi elementet på index `0`. Vi skapar inte en ny lista – vi ändrar innehållet i den befintliga.

## 5. Lägga till element med `.append()`

```python
fruits = ["apple", "banana"]
fruits.append("kiwi")
print(fruits)
# ['apple', 'banana', 'kiwi']
```

Punkten i `fruits.append(...)` betyder att vi använder en **metod som tillhör listan**. `append()` lägger till **ett element längst bak** och ändrar listan direkt.

**Jupyter-tips:** Om du kör samma cell med `fruits.append("kiwi")` två gånger läggs `"kiwi"` till två gånger. Det är inte ett fel i Python – koden har körts två gånger.

## 6. Tre sätt att ta bort element

```python
fruits = ["apple", "banana", "kiwi", "banana"]

fruits.pop()           # Tar bort sista elementet: banana
fruits.remove("banana") # Tar bort första förekomsten av banana
del fruits[0]          # Tar bort elementet på index 0: apple

print(fruits)
# ['kiwi']
```

| Metod | Du anger | Vad händer? |
|---|---|---|
| `.pop()` | Inget, eller ett index | Tar bort ett element och **returnerar** det |
| `.remove(value)` | Ett värde | Tar bort **första** matchande värdet |
| `del lista[index]` | En position | Tar bort elementet på den positionen |

```python
fruits = ["apple", "banana", "kiwi"]
removed = fruits.pop()
print(removed)  # kiwi
print(fruits)   # ['apple', 'banana']
```

`pop()` är användbar när du både vill ta bort och **använda** det borttagna värdet. `remove()` ger fel om värdet inte finns i listan.

## 7. Inbyggda funktioner och listmetoder

```python
numbers = [3, 1, 4, 1, 5, 9]

print(len(numbers))     # 6
print(numbers.count(1)) # 2
print(max(numbers))     # 9
print(min(numbers))     # 1
print(sum(numbers))     # 23
```

**Skillnaden mellan funktion och metod:**

- `len(numbers)` är en **inbyggd funktion**: listan skickas in mellan parenteserna.
- `numbers.count(1)` är en **listmetod**: punkten kopplar anropet till just listan `numbers`.

`sum()` är ett extra praktiskt exempel för numeriska listor.

## 8. `sorted()` jämfört med `.sort()`

Det här är en viktig skillnad när du analyserar data.

```python
numbers = [4, 1, 9, 3]

new_numbers = sorted(numbers)
print(new_numbers)  # [1, 3, 4, 9]
print(numbers)      # [4, 1, 9, 3]
```

`sorted(numbers)` **returnerar en ny sorterad lista**. Den gamla påverkas inte.

```python
numbers = [4, 1, 9, 3]
numbers.sort()
print(numbers)  # [1, 3, 4, 9]
```

`numbers.sort()` **ändrar listan direkt** och returnerar `None`. Skriv därför inte `new_numbers = numbers.sort()` om du vill få en sorterad lista i en ny variabel.

## 9. Medelvärde och median

Läraren visar både Pythons standardmodul `statistics` och det externa paketet NumPy.

**Medelvärde (`mean`)**: summan av alla värden delad med antalet värden.  
**Median (`median`)**: värdet i mitten när talen sorteras; vid jämnt antal används medelvärdet av de två mittersta.

```python
import statistics

numbers = [10, 20, 30, 40, 50, 60]
print(statistics.mean(numbers))    # 35
print(statistics.median(numbers))  # 35.0
```

Samma beräkningar med NumPy:

```python
import numpy as np

numbers = [10, 20, 30, 40, 50, 60]
print(np.mean(numbers))    # 35.0
print(np.median(numbers))  # 35.0
```

`as np` ger paketet ett kortare namn. **`statistics` följer med Python**, men **NumPy behöver vara installerat i den Python-miljö du använder**. I en aktiverad `.venv` kan du vid behov köra `python -m pip install numpy`.

## 10. Slå ihop listor med `+`

```python
numbers = [10, 20, 30]
extra_numbers = [40, 50]
all_numbers = numbers + extra_numbers

print(all_numbers)
# [10, 20, 30, 40, 50]
```

Här betyder `+` **sätt ihop listorna**, inte addera deras tal. De två ursprungliga listorna är oförändrade.

## 11. Nästlade listor – listor inuti listor

En **nästlad lista (`nested list`)** är en lista som innehåller andra listor.

```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
matrix = [names, ages]

print(matrix[0])     # ['Alice', 'Bob', 'Charlie']
print(matrix[0][1])  # Bob
print(matrix[1][2])  # 35
```

Så läser du `matrix[1][2]` steg för steg:

1. `matrix[1]` väljer **den andra inre listan**: `ages`.
2. `[2]` väljer **det tredje elementet** i den listan: `35`.

Tänk **först vilken lista, sedan vilken plats i den listan**. Läraren kallar sin nästlade struktur för en matris; här är det fortfarande vanliga Python-listor, inte en NumPy-array.

## 12. Vanliga nybörjarmisstag

| Misstag | Varför det blir fel | Tänk så här |
|---|---|---|
| `fruits[1]` för första elementet | Index börjar på 0 | Använd `fruits[0]` |
| Tro att `[0:2]` ger tre element | Slutindex ingår inte | Ger index 0 och 1 |
| Köra `append()`-cellen flera gånger | Listan ändras vid varje körning | Kör om cellen som skapar listan |
| `fruits.remove(0)` för att ta bort första elementet | `remove()` söker ett **värde** | Använd `del fruits[0]` eller `fruits.pop(0)` |
| `new = numbers.sort()` | `.sort()` returnerar `None` | Använd `sorted(numbers)` |
| `matrix[1][2]` känns obegripligt | Två index används i följd | Välj först lista, sedan element |

## 13. Miniövningar

### Övning 1 – Index och slicing

```python
colors = ["red", "blue", "green", "yellow"]
```

Vad skrivs ut av `colors[0]`, `colors[-1]` och `colors[1:3]`?

### Övning 2 – Ändra listan

Börja med `fruits = ["apple", "banana"]`. Lägg till `"kiwi"`, ändra `"apple"` till `"pear"` och skriv ut listan.

### Övning 3 – Nästlade listor

```python
names = ["Anna", "Bo", "Cleo"]
ages = [22, 31, 28]
data = [names, ages]
```

Hur hämtar du åldern `31`?

<details>
<summary>Facit – visa när du vill kontrollera</summary>

**1:** `red`, `yellow`, `['blue', 'green']`.

**2:**

```python
fruits = ["apple", "banana"]
fruits.append("kiwi")
fruits[0] = "pear"
print(fruits)  # ['pear', 'banana', 'kiwi']
```

**3:** `data[1][1]` – den andra inre listan och det andra elementet.

</details>

---

### Sammanfattning

En lista samlar flera värden i en variabel. Du hittar element med **index**, väljer delar med **slicing** och kan ändra innehållet eftersom listor är **mutabla**. Metoder som `.append()`, `.pop()` och `.sort()` ändrar listan, medan exempelvis `sorted()` ger en ny sorterad lista. Nästlade listor låter dig organisera information i flera nivåer.

*Anteckningarna följer lärarens genomgång; några extra exempel och förtydliganden har lagts till för att underlätta repetition.*
