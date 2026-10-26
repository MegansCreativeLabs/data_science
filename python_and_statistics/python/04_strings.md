# Lesson 04 — Strings

**Course:** Python Programming & Statistics  
**Topic:** Text, indexing, slicing, string methods and user input  
**MegansCreativeLabs — Study Notes**

> **Målet med lektionen:** Förstå vad en `str` är, hur du hämtar delar av text och hur du använder Pythons inbyggda verktyg för att bearbeta den.
>
> Anteckningarna följer lärarens genomgång, men innehåller också extra förklaringar och tydligare exempel för repetition.

---

## Quick Reference — Strings

*Snabbguide för när du sitter och kodar. Förklaringarna finns i resten av dokumentet.*

Anta att `text = "Hello world"` och `name = "Megan"`.

| Jag vill… | Python | Resultat / betydelse |
|---|---|---|
| Skapa text | `name = "Megan"` | En `str` |
| Skriva citattecken i text | `"Let's go"` | Använd olika slags citattecken |
| Göra radbrytning | `"Hello\nWorld"` | `\n` är en ny rad |
| Slå ihop text | `"Hello " + name` | `Hello Megan` |
| Sätta in variabel i text | `f"Hello {name}"` | `Hello Megan` |
| Upprepa text | `"*" * 3` | `***` |
| Mäta längd | `len(name)` | `5` |
| Hämta första tecknet | `name[0]` | `M` |
| Hämta sista tecknet | `name[-1]` | `n` |
| Hämta del av text | `name[0:3]` | `Meg` |
| Göra stora bokstäver | `text.upper()` | `HELLO WORLD` |
| Göra små bokstäver | `text.lower()` | `hello world` |
| Stor bokstav i varje ord | `text.title()` | `Hello World` |
| Ta bort yttre mellanslag | `text.strip()` | Rensar början och slutet |
| Ersätta text | `text.replace("world", "Python")` | `Hello Python` |
| Dela text i ord | `text.split()` | `['Hello', 'world']` |
| Läsa in text | `input("Your name: ")` | Returnerar alltid en `str` |

**Kom ihåg:** Index börjar på `0`. Vid slicing `[start:stop]` ingår `start`, men inte `stop`. Strängmetoder ändrar inte originalsträngen; tilldela resultatet om du vill spara ändringen.

---

## 1. Vad är en string?

En **string** (`str`) är text i Python. Den består av en följd av tecken — till exempel bokstäver, siffror, mellanslag och skiljetecken.

```python
name = "Megan"
message = 'Hello, world!'

print(name)       # Megan
print(type(name)) # <class 'str'>
```

- Text omges av **dubbla** (`" "`) eller **enkla** (`' '`) citattecken.
- Börjar du med dubbla citattecken måste du avsluta med dubbla, och motsvarande gäller enkla.
- `print()` skriver ut själva texten, normalt utan de omgivande citattecknen.

**Varför behövs strings?** Nästan alla program hanterar text: användarnamn, filnamn, meddelanden, produktnamn och data från CSV-filer.

### Citattecken inuti text

```python
text = "Let's go!"
print(text)  # Let's go!

text = 'Let\'s go!'
print(text)  # Let's go!
```

I det andra exemplet betyder `\'` att apostrofen är en del av texten och **inte** slutet på strängen.

---

## 2. Escape sequences — specialtecken

En **backslash** (`\`) kan inleda en särskild teckensekvens i en string.

| Kod | Betydelse |
|---|---|
| `\n` | Ny rad (radbrytning) |
| `\'` | Apostrof inuti en sträng med enkla citattecken |
| `\"` | Dubbelt citattecken inuti en sträng med dubbla citattecken |
| `\\` | En faktisk backslash |

```python
print("Hello\nWorld")
# Hello
# World

print("C:\\Users\\Megan")
# C:\Users\Megan
```

**Kom ihåg:** En backslash tar inte alltid bara bort betydelsen av ett tecken. I exempelvis `\n` skapar den i stället en radbrytning.

---

## 3. Slå ihop text — concatenation

Du kan använda `+` för att sätta ihop flera strings.

```python
first_name = "Megan"
last_name = "Magnusson"

full_name = first_name + " " + last_name
print(full_name)  # Megan Magnusson
```

**Varför `" "` i mitten?** Python lägger inte till mellanslag automatiskt. Mellanslaget är också ett tecken och måste finnas med.

### Text och tal kan inte alltid plussas ihop

```python
age = 25
# print("Age: " + age)  # TypeError: str och int kan inte adderas så här

print("Age: " + str(age))  # Age: 25
```

`str(age)` gör en **textversion** av talet. Variabeln `age` fortsätter att vara ett heltal om du inte tilldelar den något annat.

---

## 4. F-strings — det smidigare sättet

En **f-string** låter dig stoppa in variabler direkt i text. Skriv `f` framför citattecknet och sätt variabelnamn mellan `{` och `}`.

```python
first_name = "Megan"
last_name = "Magnusson"
age = 25

message = f"Hej, jag heter {first_name} {last_name} och är {age} år."
print(message)
# Hej, jag heter Megan Magnusson och är 25 år.
```

**Så fungerar det:**

1. `f` talar om att Python ska tolka uttryck mellan måsvingarna.
2. `{first_name}` ersätts av variabelns värde.
3. Texten runt omkring skrivs ut som vanligt.
4. Även heltal som `age` kan användas utan att du skriver `str(age)`.

**När är det användbart?** Nästan varje gång du vill kombinera text och variabler. F-strings är oftast tydligare än många `+`.

---

## 5. Upprepa en string med `*`

```python
print("Hi! " * 3)  # Hi! Hi! Hi! 
print("-" * 20)   # --------------------
```

`*` betyder här **upprepa texten**, inte vanlig multiplikation av tal. Det kan exempelvis användas för avdelare i terminalutskrifter.

---

## 6. `len()` — hur lång är texten?

`len()` är en **inbyggd funktion** som räknar antalet tecken.

```python
name = "Megan"
print(len(name))  # 5

message = "Hi there"
print(len(message))  # 8
```

**Viktigt:** Mellanslag räknas också som tecken. `"Hi there"` innehåller två bokstäver + ett mellanslag + fem bokstäver = åtta tecken.

---

## 7. Indexering — hämta ett enda tecken

Varje tecken i en string har en **position**, som kallas **index**. Python börjar räkna på **0**, inte 1.

För texten `Megan`:

| Tecken | M | e | g | a | n |
|---|---|---|---|---|---|
| Index | 0 | 1 | 2 | 3 | 4 |
| Negativt index | -5 | -4 | -3 | -2 | -1 |

```python
name = "Megan"

print(name[0])   # M — första tecknet
print(name[1])   # e — andra tecknet
print(name[-1])  # n — sista tecknet
print(name[-2])  # a — näst sista tecknet
```

### Varför börjar Python på 0?

Det är så Python definierar positionerna i sekvenser. **Index 0 betyder första platsen**, index 1 den andra, och så vidare. Du behöver inte kunna den tekniska bakgrunden nu — det viktiga är att öva in regeln.

**Vanligt misstag:** `name[1]` är **inte** första bokstaven, utan den andra.

Om du försöker läsa en position som inte finns får du ett `IndexError`:

```python
# print(name[5])  # IndexError: string index out of range
```

---

## 8. Slicing — hämta flera tecken

**Slicing** betyder att ta ut en del av en string.

```python
name = "Megan"

print(name[0:3])  # Meg
print(name[1:4])  # ega
print(name[:2])   # Me
print(name[2:])   # gan
```

Grundformen är:

```python
text[start:stop]
```

- `start` = positionen där vi **börjar** (ingår).
- `stop` = positionen där vi **slutar** (ingår **inte**).

### Varför ingår inte `stop`?

Titta på `name[0:3]`:

| Index | 0 | 1 | 2 | 3 | 4 |
|---|---|---|---|---|---|
| Tecken | **M** | **e** | **g** | a | n |

Vi börjar på index 0 och tar tecken **fram till, men inte inklusive, index 3**. Resultatet blir `Meg` — tre tecken.

**Minnesregel:** `start` är med, `stop` är inte med.

### Negativa index

```python
name = "Megan"
print(name[-3:])  # gan — de tre sista tecknen
print(name[1:-1]) # ega — från index 1 fram till sista tecknet
```

**Observera:** Att läsa ut en slice ändrar inte originaltexten.

---

## 9. String methods — färdiga verktyg för text

En **metod** är en funktion som du anropar via ett värde eller objekt, med punktnotation som `text.upper()`.

```python
text = "hello world"

print(text.upper())  # HELLO WORLD
print(text.lower())  # hello world
print(text.title())  # Hello World
```

| Metod | Vad den gör |
|---|---|
| `.upper()` | Gör bokstäverna stora |
| `.lower()` | Gör bokstäverna små |
| `.title()` | Gör början av varje ord stor (enligt metodens regler) |
| `.strip()` | Tar bort blanktecken i början och slutet |
| `.replace(old, new)` | Ersätter matchande text |
| `.split()` | Delar upp text till en lista |

### `strip()` — städa bort extra mellanslag

```python
raw_text = "   hello world   "
clean_text = raw_text.strip()

print(clean_text)  # hello world
```

`strip()` tar bort blanktecken i **början och slutet**, inte mellanslaget **mellan** `hello` och `world`.

### `replace()` — byt ut text

```python
text = "Hello world"
print(text.replace("Hello", "Hey"))  # Hey world
```

Python skiljer på stora och små bokstäver: `"hello"` matchar inte `"Hello"`.

### `split()` — gör text till en lista

```python
text = "apple banana cherry"
fruits = text.split()

print(fruits)  # ['apple', 'banana', 'cherry']
print(type(fruits))  # <class 'list'>
```

Utan argument delar `split()` normalt upp texten vid blanktecken.

Du kan också bestämma en annan avgränsare:

```python
cheeses = "cheddar,brie,port salut"
cheese_list = cheeses.split(",")

print(cheese_list)  # ['cheddar', 'brie', 'port salut']
```

Här använder vi **kommatecken** som avgränsare. Därför hålls `port salut` ihop trots mellanslaget.

---

## 10. Varför ändras inte originaltexten?

Strings i Python är **immutable** (oföränderliga). Metoder som `.upper()` och `.strip()` skapar ett nytt textvärde i stället för att ändra originalet på plats.

```python
text = "hello"

print(text.upper())  # HELLO
print(text)          # hello

text = text.upper()
print(text)          # HELLO
```

På sista raden med tilldelning (`text = text.upper()`) får variabeln `text` referera till det **nya** värdet.

Det här är en viktig skillnad mellan att **beräkna ett resultat** och att **spara resultatet i en variabel**.

---

## 11. `input()` — låt användaren skriva text

Med den inbyggda funktionen `input()` kan programmet fråga användaren efter ett svar.

```python
name = input("What is your name? ")
print(f"Hello, {name}!")
```

Programmet väntar tills användaren skriver något och trycker Enter.

### `input()` ger alltid en string

```python
age = input("How old are you? ")
print(type(age))  # <class 'str'>
```

Även om användaren skriver `25` är värdet **texten** `"25"`, inte heltalet `25`.

Om du behöver räkna med värdet kan du konvertera det:

```python
age = int(input("How old are you? "))
print(age + 1)
```

**Obs:** `int()` ger fel om användaren skriver något som inte kan tolkas som ett heltal. Vi kommer senare att kunna hantera sådana fel med exempelvis `try` och `except`.

---

## 12. Vanliga nybörjarmisstag

| Misstag | Varför det blir fel | Rätt tänk |
|---|---|---|
| `name[1]` för första bokstaven | Python börjar på index 0 | Använd `name[0]` |
| Tro att `name[0:3]` tar med index 3 | `stop` ingår inte | Resultatet innehåller index 0, 1, 2 |
| `"Age: " + 25` | `str` och `int` kan inte plussas så | Använd f-string eller `str(25)` |
| `text.upper()` utan att spara resultatet | Originalsträngen ändras inte | `text = text.upper()` |
| `replace("hello", ...)` när texten innehåller `"Hello"` | Skillnad på stora/små bokstäver | Matcha texten exakt |
| Tro att `input()` ger ett tal | `input()` returnerar `str` | Konvertera vid behov |

---

## 13. Miniövningar — testa själv

### Övning A: Första och sista bokstaven

Skapa en variabel `word = "Python"`. Skriv ut den första och den sista bokstaven.

### Övning B: Slicing

Med samma variabel, skriv ut de tre första bokstäverna.

### Övning C: Rensa och formatera

Utgå från `raw_name = "   megan   "`. Ta bort mellanslagen och skriv ut namnet med stor första bokstav.

<details>
<summary>Visa förslag på lösningar</summary>

```python
# A
word = "Python"
print(word[0])   # P
print(word[-1])  # n

# B
print(word[:3])  # Pyt

# C
raw_name = "   megan   "
clean_name = raw_name.strip().title()
print(clean_name)  # Megan
```

</details>

---

## 14. Key Takeaways — det viktigaste

- `str` är Pythons datatyp för text.
- En string kan skapas med `'...'` eller `"..."`.
- F-strings (`f"...{variable}..."`) gör det lätt att kombinera text och variabler.
- `len()` räknar tecken, inklusive mellanslag.
- Index börjar på **0**; `-1` betyder sista tecknet.
- Slicing följer regeln **start ingår, stop ingår inte**.
- Metoder som `.strip()`, `.replace()` och `.split()` hjälper oss bearbeta text.
- Strings är oföränderliga; spara metodens resultat om du vill behålla förändringen.
- `input()` ger alltid en string.

---

*MegansCreativeLabs · Python Programming & Statistics · Lesson 04*
