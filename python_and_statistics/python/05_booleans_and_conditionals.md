# Lesson 05 – Booleans & Conditional Statements

> **Syfte:** Förstå hur Python fattar beslut genom att kontrollera om ett villkor är sant eller falskt.

## Quick Reference – snabb uppslagning

| Du vill… | Python | Vad det betyder |
|---|---|---|
| Spara sant/falskt | `is_student = True` | Datatypen `bool` |
| Kontrollera typen | `type(is_student)` | Ger `<class 'bool'>` |
| Kontrollera ett värdes sanningsvärde | `bool(0)` | Ger `False` |
| Jämföra lika | `a == b` | Är värdena lika? |
| Jämföra olika | `a != b` | Är värdena olika? |
| Jämföra storlek | `a < b`, `a > b` | Mindre än / större än |
| Inkludera gränsen | `a <= b`, `a >= b` | Mindre/större än **eller lika med** |
| Köra kod vid sant villkor | `if condition:` | Första kontrollen |
| Prova ett annat villkor | `elif condition:` | Bara om tidigare villkor var falska |
| Hantera övriga fall | `else:` | Om inget tidigare villkor stämde |
| Omvandla text till heltal | `int(input("Age: "))` | `input()` ger alltid text |
| Jämföra utan hänsyn till stora bokstäver | `answer.lower() == "y"` | `Y` och `y` behandlas lika |
| Testa om en bool är sann | `if is_student:` | Behöver inte `== True` |

---

## 1. Vad är en boolean?

En **boolean** (`bool`) är en datatyp som bara kan ha två värden: `True` (sant) eller `False` (falskt). De skrivs med stor begynnelsebokstav i Python.

```python
is_student = True
has_ticket = False

print(is_student)        # True
print(type(is_student))  # <class 'bool'>
```

**Varför behövs det?** Ett program måste kunna fatta beslut, till exempel om en person har en biljett eller om ett tal är större än ett annat.

## 2. Truthy och falsy – värden som räknas som sant eller falskt

Python kan tolka även andra värden som sanna eller falska när de används i ett villkor.

```python
print(bool(0))       # False
print(bool(1))       # True
print(bool(-1))      # True
print(bool(""))      # False
print(bool("Hej"))   # True
print(bool([]))      # False
print(bool([5]))     # True
print(bool(None))    # False
```

**Kom ihåg:** `0`, tom text `""`, tom lista `[]` och `None` är *falsy*. Tal som inte är noll, text som inte är tom och listor som innehåller något är *truthy*. Även texten `"False"` är truthy eftersom den inte är tom!

## 3. Jämförelseoperatorer

En **jämförelseoperator** ställer en fråga om två värden och ger `True` eller `False` som svar.

| Operator | Betyder | Exempel | Resultat |
|---|---|---|---|
| `==` | Lika med | `12 == 12` | `True` |
| `!=` | Inte lika med | `12 != 10` | `True` |
| `<` | Mindre än | `12 < 13` | `True` |
| `>` | Större än | `12 > 13` | `False` |
| `<=` | Mindre än eller lika med | `12 <= 12` | `True` |
| `>=` | Större än eller lika med | `12 >= 13` | `False` |

### `=` och `==` är inte samma sak

```python
age = 12          # = tilldelar värdet 12 till variabeln age
print(age == 12)  # == jämför värdena och ger True
```

**Minnesregel:** Ett likhetstecken *sparar*. Två likhetstecken *frågar*.

## 4. `if` – kör kod bara när villkoret är sant

```python
age = 12

if age < 13:
    print("Du är under 13 år")

print("Klar")
```

**Resultat:**

```text
Du är under 13 år
Klar
```

**Steg för steg:**

1. `age` får värdet `12`.
2. Python kontrollerar om `12 < 13` är sant.
3. Eftersom svaret är `True` körs den indragna `print()`-raden.
4. `print("Klar")` ligger **utanför** `if` och körs oavsett svaret.

### Varför fyra mellanslag?

Rader som hör till ett `if`-block måste vara **indenterade** (indragna). I Python rekommenderas fyra mellanslag. Indraget avgör vilka rader som ingår i villkoret – det är inte bara dekoration.

```python
age = 20

if age < 13:
    print("Detta skrivs inte ut")

print("Detta skrivs alltid ut")
```

## 5. Flera separata `if` jämfört med `if` + `elif`

Två fristående `if` kontrolleras **var för sig**:

```python
age = 10

if age < 13:
    print("Under 13")

if age < 18:
    print("Under 18")
```

**Resultat:** Båda meddelandena skrivs ut, eftersom båda villkoren är sanna.

Med `elif` väljer Python däremot **första villkoret som är sant** och hoppar sedan över resten av kedjan:

```python
age = 10

if age < 13:
    print("Under 13")
elif age < 18:
    print("13–17 år")
```

**Resultat:** Bara `Under 13` skrivs ut.

## 6. `if`, `elif` och `else` tillsammans

```python
age = 40

if age < 13:
    print("Under 13 år")
elif age < 18:
    print("13–17 år")
elif age <= 65:
    print("18–65 år")
else:
    print("Över 65 år")
```

**Resultat:** `18–65 år`

- `if`: kontrollera det första villkoret.
- `elif`: kontrollera nästa **om de tidigare var falska**. Det går att ha flera `elif`.
- `else`: körs när inget av villkoren ovan stämde. `else` har inget eget villkor.

**Varför spelar ordningen roll?** Python går uppifrån och ned. Ett brett villkor först kan göra att ett mer specifikt villkor aldrig nås.

## 7. `input()` ger alltid en string

```python
age = input("Hur gammal är du? ")
print(type(age))  # <class 'str'>
```

Även om användaren skriver `22` är resultatet texten `"22"`. Därför fungerar inte `age < 18` direkt: Python kan inte jämföra en `str` med en `int` med `<`.

**Lösning: omvandla en gång, direkt vid inläsningen.**

```python
age = int(input("Hur gammal är du? "))

if age < 18:
    print("Under 18 år")
else:
    print("18 år eller äldre")
```

**Obs:** Om användaren skriver något som inte kan göras till heltal, exempelvis `hej`, uppstår `ValueError`. Felhantering kommer senare i utbildningen.

## 8. Kontrollera text från användaren

```python
ready = input("Are you ready? (y/n): ")

if ready.lower() == "y":
    print("Great! Let's continue.")
elif ready.lower() == "n":
    print("Please read the information again.")
else:
    print("I don't understand your answer.")
```

**Varför `.lower()`?** Python skiljer mellan stora och små bokstäver: `"Y" == "y"` är `False`. `.lower()` gör en textversion med små bokstäver, så både `Y` och `y` kan godkännas.

**Förbättring att känna till:** `ready = input(...).strip().lower()` tar dessutom bort extra mellanslag före och efter svaret. Det bygger på strängmetoderna från föregående lektion.

## 9. Testa en boolean direkt

```python
is_student = True

if is_student:
    print("You are a student")
```

Det behövs inte `if is_student == True:`. Variabeln innehåller redan `True` eller `False`, så Python kan använda värdet direkt.

För motsatsen kan du skriva:

```python
has_ticket = False

if not has_ticket:
    print("You need a ticket")
```

`not` vänder sanningsvärdet: `not False` blir `True`.

## 10. Vanliga nybörjarmisstag

| Misstag | Varför det blir fel | Rätt tänk |
|---|---|---|
| `if age = 18:` | `=` är tilldelning, inte jämförelse | `if age == 18:` |
| Glömma `:` efter `if` | Python förväntar sig början på ett kodblock | `if age < 18:` |
| Glömma indrag | Python vet inte vilka rader som hör till villkoret | Använd fyra mellanslag |
| Två `if` när bara ett alternativ ska väljas | Båda kan köras | Använd `if` + `elif` |
| `input()` jämförs direkt med ett tal | `input()` ger text | Använd `int(input(...))` |
| `"Y"` jämförs med `"y"` | Textjämförelser är skiftlägeskänsliga | Använd `.lower()` |
| `if is_student == True:` | Fungerar, men är onödigt långt | `if is_student:` |

## 11. Repetitionsövningar

### Övning 1 – Sant eller falskt?

Försök förutsäga resultatet:

```python
print(bool("0"))
print(bool(0))
print(5 >= 5)
```

### Övning 2 – Vilket meddelande skrivs ut?

```python
age = 15

if age < 13:
    print("A")
elif age < 18:
    print("B")
else:
    print("C")
```

### Övning 3 – Bygg en enkel kontroll

Skriv ett program som frågar efter ett heltal och skriver `Positive`, `Negative` eller `Zero` beroende på talets värde.

<details>
<summary>Facit – öppna efter att du har provat</summary>

**Övning 1:** `True`, `False`, `True`. Texten `"0"` är inte tom och är därför truthy.

**Övning 2:** `B`, eftersom `15 < 18` är sant och det första villkoret är falskt.

**Övning 3:**

```python
number = int(input("Enter a number: "))

if number > 0:
    print("Positive")
elif number < 0:
    print("Negative")
else:
    print("Zero")
```

</details>

---

## Sammanfattning

En **boolean** kan vara `True` eller `False`. En **jämförelse** ger ett sådant sanningsvärde. `if` använder det för att avgöra om kod ska köras. `elif` låter oss pröva nästa alternativ och `else` fångar upp det som återstår. Kom ihåg att **indentering styr kodblocken** och att **`input()` alltid ger text**.

*Anteckningarna följer lärarens genomgång av booleans och if-satser, med extra förklaringar och några kompletterande exempel för repetition.*
