# Lektion 10 – Dictionaries och Sets i Python

> **Källa:** Lärarens genomgång om dictionaries och sets (cirka 25 minuter). Förklaringarna följer genomgångens exempel med personer, frukter och bilar. Där ett exempel i videon gav ett fel eller ett missvisande resultat förklaras varför.

## Quick Reference

| Begrepp / kod | Vad det betyder |
|---|---|
| `dict` | En samling **nyckel–värde-par** |
| `person = {"name": "Joakim", "age": 38}` | Skapar en dictionary |
| `person["name"]` | Hämtar värdet för nyckeln `name` |
| `person.get("email")` | Hämtar värdet, eller `None` om nyckeln saknas |
| `person["age"] = 27` | Ändrar ett befintligt värde |
| `person["city"] = "Västerås"` | Lägger till en ny nyckel och ett värde |
| `del person["city"]` | Tar bort ett nyckel–värde-par |
| `person.keys()` | Ger en vy över nycklarna |
| `person.values()` | Ger en vy över värdena |
| `person.items()` | Ger en vy över `(nyckel, värde)`-par |
| `for key, value in person.items():` | Loopar över både nyckel och värde |
| `set` | En samling **unika** värden utan garanterad ordning |
| `set(["apple", "apple", "banana"])` | Tar bort dubbletter: `apple` finns bara en gång |
| `while True:` / `break` | Upprepa tills vi väljer att avsluta |

---

## 1. Vad är en dictionary?

En **dictionary** (ordbok) lagrar information som **nyckel–värde-par**. Nyckeln är det namn vi använder för att hitta ett värde.

```python
person = {
    "name": "Joakim",
    "age": 38,
    "city": "Västerås"
}
```

Här är `"name"` en **nyckel** och `"Joakim"` dess **värde**. `"age"` är en annan nyckel, vars värde är heltalet `38`.

**Jämför med en lista:**

```python
person_lista = ["Joakim", 38, "Västerås"]
print(person_lista[0])  # Joakim – vi använder index 0

print(person["name"])  # Joakim – vi använder nyckeln "name"
```

En dictionary är särskilt användbar när uppgifterna har tydliga namn, till exempel `price`, `units` och `revenue` i försäljningsdata.

**Kom ihåg:** En dictionary använder `{}` och kolon `:` mellan nyckel och värde. Strängnycklar skrivs med citattecken. Nycklar behöver inte alltid vara strängar, men de måste vara av en hashbar typ. En nyckel kan bara finnas en gång i samma dictionary.

## 2. Läsa, ändra, lägga till och ta bort

```python
person = {"name": "Joakim", "age": 38, "city": "Västerås"}

# Läsa ett värde
print(person["age"])      # 38

# Ändra ett värde (nyckeln finns redan)
person["age"] = 27

# Lägga till ett värde (nyckeln finns inte ännu)
person["email"] = "joakim@example.com"

# Ta bort en nyckel och dess värde
del person["city"]

print(person)
# {'name': 'Joakim', 'age': 27, 'email': 'joakim@example.com'}
```

**Varför används samma syntax för att ändra och lägga till?** Python kontrollerar om nyckeln redan finns. Finns den uppdateras värdet, annars skapas ett nytt par.

### Om nyckeln saknas

```python
print(person["phone"])       # KeyError: 'phone'
print(person.get("phone"))   # None – inget fel
print(person.get("phone", "Saknas"))  # Saknas
```

Använd `get()` när en uppgift kanske inte finns. `None` betyder att inget värde returnerades här.

## 3. `keys()`, `values()` och `items()`

```python
person = {"name": "Joakim", "age": 27, "email": "joakim@example.com"}

print(person.keys())    # dict_keys(['name', 'age', 'email'])
print(person.values())  # dict_values(['Joakim', 27, 'joakim@example.com'])
print(person.items())   # dict_items([('name', 'Joakim'), ...])
```

| Metod | Ger oss | När är den användbar? |
|---|---|---|
| `.keys()` | Alla nycklar | När vi vill veta vilka fält som finns |
| `.values()` | Alla värden | När vi vill arbeta med innehållet |
| `.items()` | Nyckel och värde tillsammans | När vi vill loopa över båda |

**Viktigt:** Metoderna returnerar *vyer* (`dict_keys`, `dict_values`, `dict_items`), inte vanliga listor. De går att loopa över direkt. Behöver vi verkligen en lista kan vi skriva `list(person.values())`.

## 4. Loopa genom en dictionary

```python
person = {"name": "Joakim", "age": 27}

for key, value in person.items():
    print(f"{key}: {value}")
```

**Utskrift:**

```text
name: Joakim
age: 27
```

Så fungerar raden:

1. `person.items()` ger ett par åt gången, till exempel `("name", "Joakim")`.
2. `key, value` **packar upp** paret i två variabler.
3. `print()` använder båda variablerna.

Om vi skriver `for key, value in person:` utan `.items()` försöker Python packa upp **nycklarna**, inte nyckel–värde-paren. Det kan ge felmeddelandet `ValueError: too many values to unpack` – precis det som hände i lärarens bilexempel. Vill du bara ha nycklar räcker `for key in person:`.

## 5. Vad är ett set?

Ett **set** är en samling där varje värde får förekomma **en enda gång**. Till skillnad från en lista har ett set ingen garanterad ordning och vi hämtar inte element med index.

```python
fruits = {"apple", "banana", "orange", "apple"}
print(fruits)  # Varje frukt förekommer bara en gång; ordningen kan variera
```

Om vi redan har en lista med dubbletter kan vi omvandla den:

```python
fruits_list = ["apple", "banana", "orange", "apple"]
unique_fruits = set(fruits_list)

print(unique_fruits)  # {'apple', 'banana', 'orange'} i valfri ordning
```

| Lista (`list`) | Set (`set`) |
|---|---|
| Tillåter dubbletter | Tar bort dubbletter |
| Behåller ordningen | Ingen garanterad ordning |
| Kan nås med index, t.ex. `[0]` | Kan inte nås med index |
| Skapas t.ex. med `[]` | Skapas t.ex. med `{1, 2}` |

**Vanlig fälla:** `{}` skapar en *tom dictionary*, inte ett tomt set. Ett tomt set skapas med `set()`.

## 6. Praktiskt exempel: bilar i ett garage

I videon skapar läraren ett litet program som sparar **registreringsnummer som nycklar** och **bilfärger som värden**.

```python
def cars_in_garage(dictionary):
    """Skriv ut registreringsnummer och färg för varje bil."""
    for registration, color in dictionary.items():
        print(f"{registration} is a {color} car")


cars = {}  # Tom dictionary, skapad före loopen

while True:
    registration = input("Enter car registration number: ")
    color = input("Enter car color: ")

    cars[registration] = color

    another = input("Add another car? (yes/no): ").strip().lower()
    if another != "yes":
        break

cars_in_garage(cars)
```

### Varför ser koden ut så här?

- `cars = {}` skapas **före** loopen så att tidigare bilar inte försvinner vid nästa varv.
- `while True:` upprepar inmatningen tills vi säger stopp.
- `input()` ger alltid en **sträng**.
- `cars[registration] = color` sparar färgen under bilens registreringsnummer.
- `break` avslutar loopen när svaret inte är `yes`.
- Funktionen tar emot en dictionary som **argument**; inne i funktionen heter parametern `dictionary`.
- `.items()` behövs för att få både registreringsnummer och färg i `for`-loopen.

**Observera:** Om samma registreringsnummer skrivs in igen ersätts den tidigare färgen. En dictionary kan inte ha två identiska nycklar.

Läraren visar också `continue`. Det betyder *hoppa direkt till nästa varv av loopen*. I just detta exempel behövs inte `continue`, eftersom loopen ändå går vidare när `if`-satsen är färdig.

## 7. Räkna bilar efter färg

Vi kan först ta fram alla färger från vår dictionary:

```python
cars = {
    "ABC123": "red",
    "DEF456": "black",
    "GHI789": "red",
    "JKL012": "blue"
}

colors = list(cars.values())
print(colors)  # ['red', 'black', 'red', 'blue']
```

Om vi loopar direkt över `colors` och räknar varje färg får vi raden för `red` **två gånger**. Därför använder läraren ett set för att bara loopa över de **unika färgerna**:

```python
def car_color_count(dictionary):
    colors = list(dictionary.values())
    unique_colors = set(colors)

    for color in unique_colors:
        amount = colors.count(color)
        print(f"There are {amount} {color} cars")


car_color_count(cars)
```

**Exempel på utskrift** (ordningen kan variera):

```text
There are 2 red cars
There are 1 black cars
There are 1 blue cars
```

**Viktig skillnad:** `set(colors)` tar bort dubbletter från *listan med färgnamn som vi loopar över*, inte från bilarna i `cars`. Vi räknar fortfarande alla röda bilar med `colors.count("red")`.

### Ett alternativ när du kommer längre

För större datamängder finns `Counter` i Pythons standardbibliotek:

```python
from collections import Counter

counts = Counter(cars.values())
print(counts)  # Counter({'red': 2, 'black': 1, 'blue': 1})
```

Det här är ett extra alternativ – lärarens exempel med `set()` och `.count()` är bra för att förstå grunderna.

## 8. Vanliga misstag och förklaringar

| Misstag | Varför det blir fel | Lösning |
|---|---|---|
| `person[0]` | En dictionary använder nycklar, inte listindex | `person["name"]` |
| `person[name]` | Python tolkar `name` som en variabel | `person["name"]` om nyckeln är text |
| `person["missing"]` | Nyckeln finns inte | `person.get("missing")` |
| `for key, value in person:` | Iteration över dictionary ger nycklar | `for key, value in person.items():` |
| `fruits[0]` där `fruits` är ett set | Set har inget index | Loopa med `for fruit in fruits:` |
| `empty_set = {}` | Det skapar en dictionary | `empty_set = set()` |
| Skapa `cars = {}` inne i `while` | Samlingen nollställs varje varv | Skapa den före loopen |
| Tro att `set(colors)` tar bort bilar | Det tar bara bort upprepade färgnamn ur setet | Räkna i ursprungslistan med `.count()` |

## 9. Övningar

**Övning 1 – Person:** Skapa en dictionary med `name`, `age` och `city`. Skriv ut namnet, ändra åldern och lägg till en e-postadress.

**Övning 2 – Loopa:** Skriv ut alla nycklar och värden i din person-dictionary med `.items()`.

**Övning 3 – Dubbletter:** Utgå från `fruits = ["apple", "pear", "apple", "banana", "pear"]`. Skapa ett set med de unika frukterna.

**Övning 4 – Garage:** Skapa en dictionary med tre registreringsnummer och färger. Skriv ut hur många bilar som har respektive färg utan att skriva ut samma färg flera gånger.

## 10. Facit

**Övning 1**

```python
person = {"name": "Sara", "age": 25, "city": "Malmö"}
print(person["name"])
person["age"] = 26
person["email"] = "sara@example.com"
print(person)
```

**Övning 2**

```python
for key, value in person.items():
    print(f"{key}: {value}")
```

**Övning 3**

```python
fruits = ["apple", "pear", "apple", "banana", "pear"]
unique_fruits = set(fruits)
print(unique_fruits)
```

**Övning 4**

```python
cars = {"ABC123": "red", "DEF456": "blue", "GHI789": "red"}
colors = list(cars.values())

for color in set(colors):
    print(f"{color}: {colors.count(color)}")
```

---

### Det viktigaste att komma ihåg

- **Dictionary:** När varje uppgift har ett namn – **nyckel → värde**.
- **`.items()`:** När du behöver både nyckeln och värdet i en loop.
- **Set:** När du behöver **unika värden**, utan dubbletter.
- **`while True` + `break`:** När användaren själv ska bestämma när inmatningen slutar.
- **I bilexemplet:** Dictionary lagrar bilarna; set hjälper oss undvika att skriva ut samma färgräkning flera gånger.
