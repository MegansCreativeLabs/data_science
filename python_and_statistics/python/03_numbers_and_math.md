# Lesson 03 — Numbers & Mathematical Operations

**Course:** Python Programming & Statistics  
**Topic:** Working with numbers in Python  
**Source:** Teacher's video transcript — *Tal i Python*

> **Goal:** Understand the difference between whole numbers and decimal numbers, how Python calculates, and why different operators give different results.

---

## Quick Reference — Tal & matematik

*Hitta syntaxen snabbt. Längre ner finns förklaringar och genomräknade exempel.*

| Jag vill… | Python | Resultat / betydelse |
|---|---|---|
| Addera | `7 + 3` | `10` |
| Subtrahera | `7 - 3` | `4` |
| Multiplicera | `7 * 3` | `21` |
| Dividera | `7 / 3` | `2.333...` (`float`) |
| Heltalsdividera | `7 // 3` | `2` (avrundar nedåt) |
| Få resten vid division | `7 % 3` | `1` |
| Upphöja till | `7 ** 3` | `343` |
| Omvandla till decimaltal | `float(10)` | `10.0` |
| Omvandla till heltal | `int(3.9)` | `3` (tar bort decimaldelen) |
| Få absolutvärde | `abs(-7)` | `7` |
| Avrunda till två decimaler | `round(3.14159, 2)` | `3.14` |
| Upphöja med funktion | `pow(2, 3)` | `8` |
| Öka variabel | `number += 1` | Samma som `number = number + 1` |
| Minska variabel | `number -= 1` | Minskar med ett |

**Funktioner i `math`-modulen:**

```python
import math

math.sqrt(16)   # 4.0, kvadratrot
math.pi         # 3.141592..., pi
math.floor(3.9) # 3, avrundar nedåt
math.ceil(3.1)  # 4, avrundar uppåt
```

**Kom ihåg:** `/` ger `float` vid division av vanliga tal. `int()` *avrundar inte* till närmaste heltal; den tar bort decimaldelen (mot noll).

---

## 1. Numbers in Python: `int` and `float`

Python uses different **data types** to represent numbers. The two types introduced in this lesson are:

| Type | Swedish explanation | Example |
|---|---|---|
| `int` (*integer*) | Heltal, utan decimaldel | `7`, `0`, `-12` |
| `float` | Decimaltal | `7.0`, `3.5`, `-0.25` |

```python
age = 25
price = 19.90

print(type(age))    # <class 'int'>
print(type(price))  # <class 'float'>
```

**Varför spelar det roll?** Datatypen påverkar hur Python behandlar ett värde. Till exempel ger vanlig division (`/`) ett `float`-resultat, även när båda talen är heltal.

**Kom ihåg:** I Python skriver vi decimaler med **punkt**, inte kommatecken: `3.5`.

---

## 2. Mathematical operators — räknesätt

En **operator** är ett tecken som talar om vilken beräkning Python ska utföra.

| Operator | Meaning | Example | Result |
|---|---|---|---|
| `+` | Addition | `7 + 3` | `10` |
| `-` | Subtraktion | `7 - 3` | `4` |
| `*` | Multiplikation | `7 * 3` | `21` |
| `/` | Vanlig division | `7 / 3` | `2.3333...` |
| `//` | Heltalsdivision (*floor division*) | `7 // 3` | `2` |
| `%` | Rest vid division (*modulo*) | `7 % 3` | `1` |
| `**` | Upphöjt till (*exponentiation*) | `7 ** 3` | `343` |

```python
print(7 + 3)   # 10
print(7 - 3)   # 4
print(7 * 3)   # 21
print(7 / 3)   # 2.3333333333333335
print(7 // 3)  # 2
print(7 % 3)   # 1
print(7 ** 3)  # 343
```

### Why `/`, `//` and `%` are different

Det här är tre olika frågor som vi kan ställa om **7 delat med 3**:

- `7 / 3` → **Hur mycket blir divisionen?** Cirka `2.33`.
- `7 // 3` → **Hur många hela treor ryms i sju?** `2`.
- `7 % 3` → **Hur mycket blir över?** `1`.

Tänk: `3 + 3 + 1 = 7`. Två hela grupper om tre, och en över.

```python
print(8 // 3)  # 2 hela grupper
print(8 % 3)   # 2 blir över
```

**Viktig nyans:** `//` avrundar *nedåt* mot minus oändligheten. Med negativa tal kan det därför skilja sig från att bara ta bort decimalerna: `-7 // 3` ger `-3`.

### Why `%` is useful

Modulo används ofta för att avgöra om ett heltal är jämnt eller udda:

```python
number = 8
print(number % 2)  # 0: ingen rest, alltså jämnt tal

number = 7
print(number % 2)  # 1: en rest, alltså udda tal
```

Det är **resten**, inte resultatet av divisionen, som `%` ger.

### What does `**` mean?

```python
print(2 ** 3)  # 8
```

Det betyder `2 × 2 × 2`, alltså **2 upphöjt till 3**. Två stjärnor betyder inte multiplikation två gånger.

---

## 3. Converting numbers: `int()` and `float()`

Att **konvertera** betyder att omvandla ett värde till en annan datatyp.

```python
x = 10
price = 3.9

print(float(x))   # 10.0
print(int(price)) # 3
```

- `float(10)` gör om heltalet `10` till decimaltalet `10.0`.
- `int(3.9)` ger `3` eftersom decimaldelen tas bort.

**Viktigt:** `int()` är **inte** en vanlig avrundningsfunktion. Den tar bort decimaldelen (mot noll):

```python
print(int(3.9))   # 3
print(int(-3.9))  # -3
```

Det skiljer sig från `//` för negativa tal.

### What is a function?

En **funktion** är kod som utför en viss uppgift. Du använder den genom att skriva dess namn följt av parenteser.

```python
print(10)      # Skriver ut ett värde
print(type(10)) # Visar datatypen
print(float(10)) # Omvandlar till float
```

Det som står **inne i parenteserna** är det värde som funktionen får arbeta med. I senare lektioner lär du dig skapa egna funktioner.

---

## 4. Useful built-in math functions

Python har flera funktioner som du kan använda direkt, utan `import`.

### `abs()` — absolutvärde

```python
print(abs(-7))  # 7
```

Absolutvärdet är talets avstånd från noll. Därför är `abs(-7)` lika med `7`.

### `round()` — avrunda

```python
print(round(3.14159))     # 3
print(round(3.14159, 2))  # 3.14
print(round(3.14159, 3))  # 3.142
```

Det andra värdet i `round(number, 2)` talar om **hur många decimaler** du vill behålla.

**Bra att känna till:** Vid exakta halvvägsvärden kan Pythons `round()` skilja sig från skolans vanliga avrundningsregel. Exempelvis ger `round(2.5)` resultatet `2` och `round(3.5)` resultatet `4`. Det beror på att Python avrundar till närmaste jämna tal vid sådana fall.

### `pow()` — upphöjt till

```python
print(pow(2, 3))  # 8
print(2 ** 3)     # 8
```

Båda beräknar två upphöjt till tre.

---

## 5. Importing the `math` module

Ibland vill vi använda funktioner som inte finns direkt tillgängliga. Då kan vi **importera en modul**.

En **modul** är en samling Python-kod som innehåller användbara funktioner och andra värden. `math` följer med Python, så du behöver inte installera något extra paket för den.

```python
import math

print(math.sqrt(16))   # 4.0
print(math.pi)         # 3.141592653589793
print(math.floor(3.9)) # 3
print(math.ceil(3.1))  # 4
```

| Expression | Meaning | Result |
|---|---|---|
| `math.sqrt(16)` | Kvadratroten ur 16 | `4.0` |
| `math.pi` | Konstanten π | `3.141592653589793` |
| `math.floor(3.9)` | Avrundar nedåt | `3` |
| `math.ceil(3.1)` | Avrundar uppåt | `4` |

### Why do we write `math.sqrt()` instead of `sqrt()`?

När du skriver `import math` blir funktionerna tillgängliga **via modulens namn**:

```python
import math
print(math.sqrt(25))  # 5.0
```

Punkten betyder här ungefär **”hämta `sqrt` från modulen `math`”**.

> **Terminologi:** I videon beskrivs detta som en metod på ett objekt. Mer precist är `math` en **modul**, och `sqrt` är en **funktion i modulen**. Du behöver inte kunna hela skillnaden mellan funktioner och metoder än.

---

## 6. Updating a variable: `+=`, `-=`, `*=`, `/=`

När vi vill ändra ett värde kan vi använda en kortare skrivning.

```python
number = 10
number = number + 1
print(number)  # 11
```

Detta kan skrivas som:

```python
number = 10
number += 1
print(number)  # 11
```

**Båda betyder:** Ta det gamla värdet i `number`, lägg till `1` och spara det nya värdet i `number`.

| Short form | Same as | Meaning |
|---|---|---|
| `x += 1` | `x = x + 1` | Öka med 1 |
| `x -= 1` | `x = x - 1` | Minska med 1 |
| `x *= 3` | `x = x * 3` | Multiplicera med 3 |
| `x /= 2` | `x = x / 2` | Dividera med 2 |

```python
number = 10
number += 2
print(number)  # 12

number *= 3
print(number)  # 36

number /= 2
print(number)  # 18.0
```

Lägg märke till att `/=` ger ett `float`-värde, precis som `/`.

---

## 7. Readable code and PEP 8

Läraren påminner om att vi normalt skriver **mellanslag runt operatorer**. Det är en del av Pythons kodstil, PEP 8.

```python
# Svårare att läsa
result=7+3

# Lättare att läsa
result = 7 + 3
```

Datorn förstår båda, men den andra är tydligare för människor som ska läsa koden.

---

## 8. Common beginner mistakes

| Mistake | What happens | Better approach |
|---|---|---|
| `3,5` som decimaltal | Python tolkar kommatecknet på ett annat sätt | Skriv `3.5` |
| `7 / 3` när du vill ha hela grupper | Ger ett decimaltal | Använd `7 // 3` |
| `7 % 3` när du vill ha kvoten | Ger resten (`1`) | Använd `/` eller `//` |
| `int(3.9)` för att avrunda | Ger `3`, inte `4` | Använd `round()` när du vill avrunda |
| `sqrt(16)` efter `import math` | `sqrt` finns inte som fristående namn | Skriv `math.sqrt(16)` |
| `^` för upphöjt till | `^` betyder något annat i Python | Skriv `**` |

---

## 9. Try it yourself — mini exercises

De här små övningarna är **extra repetitionsövningar**, inte hämtade från lärarens video.

**A.** Vad tror du skrivs ut?

```python
print(10 / 4)
print(10 // 4)
print(10 % 4)
```

**B.** Prova att ändra värdet på `number` och se vad som händer:

```python
number = 5
number += 3
number *= 2
print(number)
```

**C.** Vad är skillnaden mellan följande uttryck?

```python
print(int(3.9))
print(round(3.9))
print(math.floor(3.9))  # Kom ihåg import math!
```

<details>
<summary>Show answers</summary>

- **A:** `2.5`, `2`, `2`
- **B:** `16`
- **C:** `3`, `4`, `3`

</details>

---

## 10. Key takeaways

- `int` = heltal; `float` = decimaltal.
- `/` ger vanlig division, `//` ger nedåtrundad division och `%` ger resten.
- `**` betyder upphöjt till.
- `int()` tar bort decimaldelen; `float()` gör om till decimaltal.
- `abs()`, `round()` och `pow()` är inbyggda funktioner.
- `import math` ger tillgång till bland annat `math.sqrt()`, `math.floor()`, `math.ceil()` och `math.pi`.
- `+=` och liknande är kortare sätt att uppdatera variabler.
- Mellanslag runt operatorer gör koden mer lättläst.

---

*MegansCreativeLabs · Python Programming & Statistics · Learning notes*
