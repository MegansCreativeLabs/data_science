# Lektion 13 – Tuples i Python

**Kurs:** Python – vecka 3  
**Ämne:** Tuples (tupler), indexering, unpacking och funktioner som returnerar flera värden

> **Utifrån lärarens genomgång:** En tuple är ett litet paket av värden som hör ihop. Den liknar en lista, men själva tuplen går inte att ändra efter att den skapats.

## Quick Reference

| Kod | Betydelse |
|---|---|
| `punkt = (3, 4)` | Skapa en tuple med två värden |
| `punkt[0]` | Hämta första värdet (`3`) |
| `len(punkt)` | Antal värden (`2`) |
| `x, y = punkt` | Packa upp tuplen i två variabler |
| `tal[1:4]` | Skapa en ny tuple med index 1, 2 och 3 |
| `a + b` | Slå ihop två tuples till en ny |
| `tal.count(2)` | Räkna förekomster av värdet `2` |
| `tal.index(3)` | Hitta första index där värdet `3` finns |
| `head, *rest = tal` | Första värdet separat; resten som **lista** |
| `return min(a, b), max(a, b)` | Returnera två värden som en tuple |

## 1. Varför finns tuples?

Tänk dig en punkt i ett koordinatsystem. Punkten har ett **x-värde** och ett **y-värde**:

```python
x = 3
y = 4
```

De två variablerna hör ihop, men inget i koden visar tydligt att de beskriver **samma punkt**. Vi kan samla dem:

```python
punkt = (3, 4)
print(punkt)  # (3, 4)
```

- `punkt` är **variabelns namn**.
- `(3, 4)` är en **tuple** med två värden.
- Ordningen spelar roll: `3` är x-koordinaten och `4` är y-koordinaten.

En tuple är en **ordnad samling**. Det innebär att varje värde har en bestämd plats, ett **index**.

## 2. Tuple jämfört med lista

```python
koordinat_lista = [3, 4]
koordinat_tuple = (3, 4)
```

| Egenskap | Lista | Tuple |
|---|---|---|
| Skrivs med | `[]` | Vanligen `()` |
| Har index | Ja | Ja |
| Kan innehålla flera värden | Ja | Ja |
| Kan byta ut ett värde direkt | Ja | Nej |
| Vanlig användning | Samling som förändras | Värden som hör ihop och ska hållas fasta |

### En lista går att ändra

```python
koordinat_lista[0] = 10
print(koordinat_lista)  # [10, 4]
```

`[0]` betyder **första positionen**. Python börjar räkna index från noll.

### En tuple går inte att ändra på samma sätt

```python
koordinat_tuple[0] = 10
```

Det ger ett fel av typen `TypeError`: *'tuple' object does not support item assignment*.

**Oföränderlig (immutable)** betyder här att du inte kan byta ut, lägga till eller ta bort element **i själva tuplen**. Det betyder inte att variabelnamnet är låst.

### Skapa en ny tuple i stället

```python
punkt = (3, 4)
punkt = (punkt[0] + 5, punkt[1])
print(punkt)  # (8, 4)
```

Steg för steg:
1. `punkt[0]` hämtar `3`.
2. `3 + 5` blir `8`.
3. `punkt[1]` hämtar `4`.
4. `(8, 4)` skapar **en ny tuple**.
5. Variabeln `punkt` får nu referera till den nya tuplen.

**Viktig nyans:** Om en tuple innehåller en lista kan innehållet i den listan fortfarande ändras. Det är tuplens egna platser som är låsta.

```python
exempel = ([1, 2], "etikett")
exempel[0].append(3)
print(exempel)  # ([1, 2, 3], 'etikett')
```

## 3. Läsa värden: index och `len()`

```python
p = (3, 4)

print(p[0])   # 3
print(p[1])   # 4
print(len(p)) # 2
```

- `p[0]`: värdet på index 0.
- `p[1]`: värdet på index 1.
- `len(p)`: antal element i tuplen, **inte** sista indexet.

För två element är indexen alltså `0` och `1`.

## 4. Unpacking – packa upp värden

**Unpacking** innebär att vi fördelar värden från en samling till separata variabler.

```python
p = (3, 4)
x, y = p

print(x)  # 3
print(y)  # 4
```

Det är samma idé som att skriva `x = 3` och `y = 4`, men båda värdena hämtas från `p` i en enda rad.

Antalet variabler måste normalt motsvara antalet värden:

```python
p = (3, 4)
x, y, z = p  # ValueError: för få värden att packa upp
```

### Unpacking med stjärna (`*`)

```python
tal = (100, 200, 300, 400, 500)
head, *rest = tal

print(head)  # 100
print(rest)  # [200, 300, 400, 500]
```

Här får `head` första värdet. `*rest` samlar alla återstående värden i **en lista**, även om ursprunget är en tuple. Detta är en viktig detalj som är lätt att missa.

## 5. Du har redan sett tuples: `enumerate()`

Läraren visar att tuples ofta dyker upp utan att vi själva skriver parenteserna.

```python
names = ["Jonas", "Emil"]

for index, name in enumerate(names):
    print(index, name)
```

Resultat:

```text
0 Jonas
1 Emil
```

`enumerate(names)` ger oss par av typen `(index, värde)`. I varje varv packas paret upp i variablerna `index` och `name`.

Samma kod kan skrivas så här för att visa själva paren:

```python
for par in enumerate(names):
    print(par)

# (0, 'Jonas')
# (1, 'Emil')
```

## 6. Tuples när vi loopar över en dictionary

```python
person = {"name": "Joakim", "age": 38}

for key, value in person.items():
    print(key, value)
```

Resultat:

```text
name Joakim
age 38
```

`person.items()` ger nyckel–värde-par som vi kan packa upp. Varje par fungerar som en tuple med två delar:

- **key**: nyckeln, till exempel `"name"`.
- **value**: värdet som hör till nyckeln, till exempel `"Joakim"`.

## 7. Returnera flera värden från en funktion

En funktion kan lämna tillbaka flera värden på en gång genom att returnera en tuple.

```python
def min_max(a, b):
    return min(a, b), max(a, b)

low, high = min_max(7, 2)

print(low)   # 2
print(high)  # 7
```

**Vad händer?**
1. Funktionen får argumenten `7` och `2`.
2. `min(a, b)` blir `2`.
3. `max(a, b)` blir `7`.
4. `return 2, 7` returnerar tuplen `(2, 7)`.
5. `low, high = ...` packar upp tuplen till två variabler.

Vi kan även spara resultatet som **en** variabel:

```python
resultat = min_max(7, 2)
print(resultat)     # (2, 7)
print(resultat[0])  # 2
```

## 8. Slicing – välj en del av tuplen

```python
tal = (10, 20, 30, 40, 50)
print(tal[1:4])  # (20, 30, 40)
```

`[1:4]` betyder: börja på index **1** och sluta **före** index **4**.

| Index | 0 | 1 | 2 | 3 | 4 |
|---|---|---|---|---|---|
| Värde | 10 | 20 | 30 | 40 | 50 |

Slicing skapar en **ny tuple**. Originalet ändras inte.

## 9. Slå ihop tuples

```python
a = (1, 2)
b = (3, 4)
c = a + b

print(c)  # (1, 2, 3, 4)
```

Operatorn `+` sammanfogar värdena till en **ny tuple**. Den ändrar inte `a` eller `b`.

## 10. `count()` och `index()`

```python
tal = (1, 2, 3, 2)

print(tal.count(2))  # 2
print(tal.index(3))  # 2
```

- `.count(2)` svarar på **hur många gånger** värdet `2` förekommer.
- `.index(3)` svarar på **vilken indexposition** värdet `3` först finns på.

Om värdet saknas ger `.index()` ett `ValueError`.

## 11. En tuple med ett enda värde

Det här är ett vanligt nybörjarmisstag:

```python
inte_tuple = (5)
en_tuple = (5,)

print(type(inte_tuple))  # <class 'int'>
print(type(en_tuple))    # <class 'tuple'>
```

Det är **kommatecknet** som gör en tuple med ett enda element. Parenteserna hjälper oss oftast att läsa koden, men är inte ensamma avgörande.

## 12. När väljer man tuple eller lista?

Använd gärna en **tuple** när du har ett bestämt paket med värden som hör ihop, till exempel:

- En koordinat: `(3, 4)`.
- Ett datum uppdelat i delar: `(2026, 10, 26)`.
- Ett minimi- och maximivärde från en funktion: `(2, 7)`.

Använd hellre en **lista** när du har en samling som ska kunna växa, krympa eller ändras, till exempel en lista över namn eller mätvärden.

**Minnesregel från genomgången:** *Statiskt paket → tuple. Dynamisk samling → lista.*

## 13. Vanliga misstag

| Misstag | Varför det blir fel | Rätt tanke |
|---|---|---|
| `punkt[0] = 10` | Tuples är oföränderliga | Skapa en ny tuple |
| `x, y, z = (3, 4)` | Tre variabler men bara två värden | Matcha antal eller använd `*rest` |
| Tro att `*rest` blir en tuple | Stjärn-unpacking samlar resten i en lista | `rest` blir `list` |
| Tro att index börjar på 1 | Python börjar på 0 | Första elementet är `[0]` |
| `(5)` när du vill ha en tuple | Det är bara talet 5 inom parentes | Skriv `(5,)` |
| Tro att `a + b` ändrar `a` | Sammanfogning skapar en ny tuple | Spara resultatet i en variabel |

## 14. Begreppslista

| Begrepp | Enkel förklaring |
|---|---|
| **Tuple / tuppel** | Ordnad samling med fasta elementplatser |
| **Immutable / oföränderlig** | Tuplens element kan inte bytas ut efter skapandet |
| **Mutable / ändringsbar** | Innehållet kan ändras, som i en lista |
| **Index** | Platsnummer, räknat från 0 |
| **Unpacking** | Fördela samlingens värden till flera variabler |
| **Slicing** | Välja ett intervall av element |
| **Returnera** | Skicka tillbaka ett resultat från en funktion |
| **`enumerate()`** | Ger index och värde när vi itererar |
| **`.items()`** | Ger nyckel och värde från en dictionary |

## 15. Övningar

### Övning 1 – Skapa och läs
Skapa tuplen `punkt = (8, 12)`. Skriv ut båda värdena var för sig och skriv ut antalet element.

### Övning 2 – Vad händer?
Vad händer om du kör `punkt[0] = 99`? Varför?

### Övning 3 – Unpacking
Packa upp `farg = (255, 128, 0)` till variablerna `red`, `green` och `blue`.

### Övning 4 – Funktion
Skriv en funktion `min_max(a, b)` som returnerar minsta och största värdet. Anropa den med `15` och `6`.

### Övning 5 – Slicing
Utgå från `tal = (10, 20, 30, 40, 50)` och skriv kod som ger `(20, 30, 40)`.

### Övning 6 – Stjärna
Vad blir datatyperna för `head` och `rest` efter `head, *rest = (100, 200, 300)`?

---

## Facit

### 1
```python
punkt = (8, 12)
print(punkt[0])  # 8
print(punkt[1])  # 12
print(len(punkt))  # 2
```

### 2
Det blir ett `TypeError`, eftersom man inte får byta ut ett element i en tuple.

### 3
```python
farg = (255, 128, 0)
red, green, blue = farg
print(red, green, blue)  # 255 128 0
```

### 4
```python
def min_max(a, b):
    return min(a, b), max(a, b)

low, high = min_max(15, 6)
print(low, high)  # 6 15
```

### 5
```python
tal = (10, 20, 30, 40, 50)
print(tal[1:4])  # (20, 30, 40)
```

### 6
```python
head, *rest = (100, 200, 300)
print(head, type(head))  # 100 <class 'int'>
print(rest, type(rest))  # [200, 300] <class 'list'>
```

---

**Sammanfattning:** En tuple samlar värden som hör ihop i en ordnad och oföränderlig struktur. Du kan läsa värden med index, packa upp dem i variabler, välja delar med slicing och använda tuples för att returnera flera resultat från funktioner.
