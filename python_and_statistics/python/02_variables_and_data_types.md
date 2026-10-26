# Lesson 01 — Variables & Data Types

> **MegansCreativeLabs · Python Programming & Statistics**  
> **Ämne:** Variabler, datatyper, namnregler, kommentarer och PEP 8  
> **Källa:** Lärarens genomgång *Variabler och datatyper* (cirka 30 minuter)

## Vad kommer jag att lära mig?

Efter den här lektionen ska jag kunna:

- förklara vad en **variabel** är och varför den behövs
- skapa variabler och skriva ut deras värden
- känna igen vanliga **datatyper** i Python
- använda `type()` för att kontrollera en datatyp
- förstå hur ett värde kan ändras under programmets gång
- skriva tydliga variabelnamn enligt `snake_case`
- förstå vad kommentarer och **PEP 8** används till

---

## Quick Reference — Variabler & datatyper

*Snabb uppslagning när du programmerar. Läs avsnitten längre ner om du vill förstå varför.*

| Jag vill… | Exempel | Kom ihåg |
|---|---|---|
| Spara ett heltal | `age = 25` | `=` tilldelar ett värde |
| Spara text | `name = "Megan"` | Text skrivs inom citattecken |
| Spara ett decimaltal | `price = 19.90` | Punkt används som decimaltecken |
| Spara sant/falskt | `is_student = True` | `True` och `False` har stor begynnelsebokstav |
| Representera inget värde | `result = None` | `None` är inte samma sak som `0` |
| Skapa en lista | `numbers = [1, 2, 3]` | Flera värden i en samling |
| Skapa en dictionary | `person = {"name": "Megan"}` | Nyckel kopplas till värde |
| Skriva ut ett värde | `print(name)` | Visar värdet i terminalen |
| Kontrollera datatyp | `type(age)` | Returnerar typen, t.ex. `int` |
| Ändra ett värde | `age = 26` | Ny tilldelning ersätter den gamla |
| Öka med ett | `count = count + 1` | Använder tidigare värdet |
| Skriva en kommentar | `# Förklaring` | Kommentaren körs inte |

**Namnregler:** `first_name` (snake_case) för vanliga variabler; `MAX_ATTEMPTS` för värden som behandlas som konstanter. Python skiljer på `name` och `Name`.

---

## 1. Vad är en variabel? (`variable`)

En **variabel** är ett namn som gör att vi kan använda ett värde i programmet. Läraren beskriver det som en **låda med en etikett**: etiketten är namnet, och i lådan finns informationen.

```python
age = 25
name = "Megan"

print(age)   # 25
print(name)  # Megan
```

**Läs koden steg för steg:**

1. `age = 25` ger namnet `age` värdet `25`.
2. `name = "Megan"` ger namnet `name` textvärdet `"Megan"`.
3. `print(age)` hämtar värdet som `age` syftar på och skriver ut det.

**Varför behövs variabler?** Vi kan återanvända information och ändra ett värde på ett ställe i stället för att skriva samma sak överallt.

> **Viktigt:** `=` betyder **tilldelning** i Python: ”ge variabeln det här värdet”. Det betyder inte exakt samma sak som likhetstecknet i matematik. För att *jämföra* om två värden är lika används `==` (en kompletterande förklaring utöver lärarens huvudexempel).

### Räkna med variabler

```python
x = 5
y = 10
z = x + y

print(z)  # 15
```

Här läser Python värdena för `x` och `y`, adderar dem och tilldelar resultatet till `z`.

**Kom ihåg:** `x`, `y` och `z` fungerar i enkla exempel, men i riktiga program är beskrivande namn oftast bättre.

---

## 2. Datatyper (`data types`)

En **datatyp** berättar vilken sorts värde vi arbetar med. Det spelar roll eftersom olika värden kan användas på olika sätt. Exempelvis kan tal adderas, medan text kan sättas ihop eller delas upp.

| Datatyp | Betyder | Exempel |
|---|---|---|
| `int` | Heltal (*integer*) | `25` |
| `float` | Decimaltal | `3.14` |
| `str` | Text (*string*) | `"Hello"` |
| `bool` | Sant eller falskt (*boolean*) | `True` eller `False` |
| `NoneType` | Typen för värdet `None` (inget angivet värde) | `None` |
| `list` | Lista med flera värden | `[1, 2, 3]` |
| `dict` | Uppslag med nycklar och värden (*dictionary*) | `{"name": "Megan"}` |

### Exempel med olika typer

```python
age = 25                 # int
pi = 3.14                # float
name = "Megan"           # str
is_student = True        # bool
numbers = [1, 2, 3]     # list
person = {"name": "Megan", "age": 25}  # dict
nothing = None           # NoneType
```

**Att förstå:**

- `25` är ett heltal, men `25.0` är ett decimaltal (`float`).
- Text skrivs inom citattecken, exempelvis `"25"`. Det är **text**, inte talet `25`.
- `True`, `False` och `None` skrivs med stor begynnelsebokstav och utan citattecken.
- En lista använder hakparenteser `[]`. En dictionary använder klammerparenteser `{}`.

### Hur vet Python datatypen?

I Python behöver vi normalt inte ange datatypen i förväg. Python avgör den utifrån värdet vi tilldelar.

```python
age = 25
name = "Megan"

print(type(age))   # <class 'int'>
print(type(name))  # <class 'str'>
```

`type()` är en **inbyggd funktion** som talar om vilken typ ett värde har. `print()` visar resultatet i terminalen eller notebooken.

> **Varför står det `class`?** Läraren nämner att datatyper i Python är objekt. Du behöver inte kunna objektorienterad programmering redan nu; det räcker att känna igen att `<class 'int'>` betyder att värdet har typen `int`.

---

## 3. Variabler kan ändras

Ett variabelnamn kan tilldelas ett nytt värde senare i programmet.

```python
age = 38
print(age)  # 38

age = 25
print(age)  # 25
```

Det gamla värdet skrivs över för just det variabelnamnet.

### Vad betyder `count = count + 1`?

```python
count = 1
print(count)  # 1

count = count + 1
print(count)  # 2
```

Läs raden `count = count + 1` **från höger till vänster**:

1. Ta det nuvarande värdet på `count` → `1`.
2. Lägg till `1` → resultatet blir `2`.
3. Spara resultatet under namnet `count`.

Det är alltså en **uppdatering**, inte en matematisk ekvation.

### En variabel kan även byta datatyp

```python
value = "Hello"
print(type(value))  # <class 'str'>

value = 123
print(type(value))  # <class 'int'>
```

Python tillåter detta, men det kan göra program svårare att förstå om vi råkar ändra en typ av misstag.

---

## 4. Namnge variabler tydligt

Läraren rekommenderar **beskrivande engelska namn**. I Python använder vi vanligtvis **`snake_case`**: små bokstäver och understreck mellan ord.

```python
first_name = "Megan"
student_age = 25
total_price = 199.50
```

Jämför:

```python
x = 199.50          # Oklart vad värdet betyder
total_price = 199.50  # Tydligare
```

### Olika namnkonventioner

| Stil | Exempel | Kommentar |
|---|---|---|
| `snake_case` | `first_name` | Vanlig för variabler och funktioner i Python |
| `camelCase` | `firstName` | Vanlig i exempelvis JavaScript och Java |
| `PascalCase` | `FirstName` | Används bland annat för klassnamn |
| `SCREAMING_SNAKE_CASE` | `MAX_ATTEMPTS` | Konvention för konstanter |

En **konstant** är ett värde som är tänkt att förbli oförändrat. I Python är versaler en **namnkonvention**, inte ett tekniskt lås som hindrar ändringar.

---

## 5. Python skiljer på stora och små bokstäver

Det kallas att språket är **case-sensitive**.

```python
name = "Megan"
print(name)  # Fungerar

# print(Name)  # NameError: namnet Name finns inte
# Print(name)  # NameError: funktionen Print finns inte
```

`name` och `Name` är alltså **inte samma namn**. Samma sak gäller `print` och `Print`.

**Felsökningstips:** Om Python säger `NameError`, kontrollera stavning och stora/små bokstäver först.

---

## 6. Kommentarer (`comments`)

En kommentar börjar med `#`. Python kör inte texten efter `#` på samma rad.

```python
# Detta är en kommentar som förklarar koden
age = 25  # Här sparar vi ett heltal
print(age)
```

**Varför kommentera?** Kommentarer kan förklara *varför* vi gör något, särskilt om det inte framgår direkt av koden.

I VS Code kan du markera en rad och använda kortkommandot för att kommentera eller avkommentera den (ofta `Ctrl + /`, beroende på tangentbordslayout).

---

## 7. Kodstil och PEP 8

**PEP 8** är en stilguide med rekommendationer för hur Python-kod bör skrivas så att den blir lätt att läsa och samarbeta kring.

Läraren lyfter bland annat fram:

- beskrivande variabelnamn
- `snake_case` för variabler
- konsekvent indrag, normalt **fyra mellanslag** per nivå
- tydlig formatering och luft mellan kodavsnitt

Läs mer: [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/)

> **Skillnaden mellan regel och rekommendation:** Felaktiga indrag kan göra att Python inte kan köra koden. Många andra PEP 8-råd handlar om läsbarhet och samarbete, inte om huruvida koden fungerar.

---

## 8. Notebook-tips: körordningen spelar roll

Läraren visar att kodceller i en Jupyter Notebook kan köras i en annan ordning än de står på sidan. Det kan göra det svårt att förstå varför en variabel har ett visst värde.

```python
age = 38
```

Om du senare kör en annan cell med `age = 25`, så kommer `print(age)` att visa `25` — även om den första cellen ligger högre upp i notebooken.

**Bra vana:** Kör cellerna uppifrån och ned när du vill kontrollera att hela notebooken fungerar som tänkt.

---

## 9. Snabb repetition

**Vad är en variabel?**  
Ett namn som används för att komma åt ett värde.

**Vad är skillnaden mellan `int` och `float`?**  
`int` är heltal, medan `float` representerar decimaltal.

**Vad gör `type()`?**  
Visar vilken datatyp ett värde har.

**Vad händer vid `count = count + 1`?**  
Det nuvarande värdet ökas med ett och sparas på nytt.

**Varför `snake_case`?**  
Det är den rekommenderade namnstilen för vanliga variabler i Python.

**Vad är PEP 8?**  
En stilguide som hjälper oss skriva läsbar och konsekvent Python-kod.

---

## 10. Testa själv

Kör gärna dessa tre små exempel i en notebook eller Python-fil:

```python
# 1. Skapa och kontrollera en variabel
favorite_number = 7
print(favorite_number)
print(type(favorite_number))

# 2. Uppdatera ett värde
score = 10
score = score + 5
print(score)  # 15

# 3. Undersök skillnaden mellan text och tal
number = 10
text_number = "10"
print(type(number))
print(type(text_number))
```

**Målet är inte att memorera varje rad.** Målet är att förstå vad Python gör och kunna känna igen samma mönster nästa gång.

---

*MegansCreativeLabs · Learning, building and exploring.*
