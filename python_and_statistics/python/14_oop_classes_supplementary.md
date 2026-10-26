# Lektion 14 – OOP: Klasser i Python

> **KOMPLETTERANDE LEKTION – EJ VERIFIERAD MOT KURSMATERIALET**  
> Den här lektionen är framtagen utifrån ämnet **OOP – Klasser**. Lärarens videogenomgång saknade tillgänglig transkribering. 

> **Status:** Egen komplettering.  **Att göra senare:** Jämför med lärarens material om det blir tillgängligt.

## 1. Vad är objektorienterad programmering (OOP)?

**OOP** betyder *objektorienterad programmering*. Det är ett sätt att organisera kod genom att samla **data** och **funktioner som arbetar med datan** i samma slags struktur: ett **objekt**.

Tänk på ett supportärende. Varje ärende har information, till exempel en titel och en status. Det kan också ha beteenden, till exempel att markeras som löst.

- **Klass:** en beskrivning eller mall för hur ett objekt ska se ut.
- **Objekt/instans:** ett konkret exemplar som skapats från klassen.
- **Attribut:** information som hör till ett objekt.
- **Metod:** en funktion som hör till en klass och kan arbeta med objektets information.

## 2. Din första klass

```python
class SupportArende:
    pass

arende1 = SupportArende()
print(type(arende1))
```

Förklaring:

- `class` talar om att vi definierar en klass.
- `SupportArende` är klassens namn. Klassnamn skrivs vanligtvis med stor bokstav i början av varje ord.
- `:` markerar att ett indraget kodblock följer.
- `pass` betyder ungefär ”gör ingenting ännu”. Det gör att en tom klass är giltig Python.
- `SupportArende()` skapar **en instans** av klassen.
- `arende1` är variabeln som hänvisar till den instansen.

**Viktigt:** Klassen `SupportArende` är mallen. `arende1` är ett objekt som skapats med mallen.

## 3. `__init__` – ge objektet startvärden

Ofta vill vi att varje nytt objekt ska få egna uppgifter direkt när det skapas.

```python
class SupportArende:
    def __init__(self, titel, status):
        self.titel = titel
        self.status = status

arende1 = SupportArende("Kan inte logga in", "Öppet")
arende2 = SupportArende("Skrivaren fungerar inte", "Pågår")

print(arende1.titel)   # Kan inte logga in
print(arende2.status)  # Pågår
```

### Vad betyder varje del?

| Kod | Betydelse |
|---|---|
| `def __init__(...)` | En särskild metod som körs när en ny instans skapas. |
| `self` | Hänvisar till **det aktuella objektet**. |
| `titel`, `status` | Parametrar: värden som metoden tar emot. |
| `self.titel = titel` | Sparar parameterns värde som ett attribut på objektet. |
| `SupportArende("...", "...")` | Skapar objektet och skickar in argumenten. |
| `arende1.titel` | Hämtar attributet `titel` från just `arende1`. |

**Varför både `titel` och `self.titel`?** `titel` är värdet som kommer in i metoden. `self.titel` är platsen där värdet sparas på objektet så att det finns kvar efteråt.

## 4. Vad är `self` egentligen?

`self` är inte ett magiskt ord som Python kräver som namn, men det är den etablerade standarden. Det står för objektet som metoden arbetar med.

```python
class Hund:
    def __init__(self, namn):
        self.namn = namn

hund1 = Hund("Milo")
hund2 = Hund("Bella")

print(hund1.namn)  # Milo
print(hund2.namn)  # Bella
```

Båda hundarna kommer från samma klass, men de har **olika värden** på sina attribut.

När vi skriver `hund1.namn` tittar Python på objektet `hund1`, inte på alla hundar samtidigt.

## 5. Metoder – vad ett objekt kan göra

En metod är en funktion som definieras i en klass.

```python
class SupportArende:
    def __init__(self, titel, status="Öppet"):
        self.titel = titel
        self.status = status

    def markera_som_lost(self):
        self.status = "Löst"

    def visa_info(self):
        print(f"{self.titel}: {self.status}")

arende = SupportArende("Kan inte logga in")
arende.visa_info()       # Kan inte logga in: Öppet

arende.markera_som_lost()
arende.visa_info()       # Kan inte logga in: Löst
```

Här händer två saker:

1. `markera_som_lost()` **ändrar** objektets attribut `status`.
2. `visa_info()` **läser** attributen och skriver ut dem.

Vi anropar metoderna med punktnotation: `arende.visa_info()`.

### Varför skriver vi inte `self` i anropet?

Metoden definieras med `self`, men när vi anropar den via ett objekt (`arende.visa_info()`) skickar Python automatiskt med objektet som första argument.

## 6. Standardvärden för parametrar

I föregående exempel stod:

```python
def __init__(self, titel, status="Öppet"):
```

Det betyder att `status` får värdet `"Öppet"` om vi inte skickar in något annat.

```python
arende1 = SupportArende("Fel på lösenord")
arende2 = SupportArende("Nätverksproblem", "Pågår")
```

- `arende1.status` blir `"Öppet"`.
- `arende2.status` blir `"Pågår"`.

## 7. Klass jämfört med instans – den viktiga skillnaden

```python
class Bok:
    def __init__(self, titel):
        self.titel = titel

bok1 = Bok("Python för nybörjare")
bok2 = Bok("Lär dig Pandas")
```

| Begrepp | I exemplet |
|---|---|
| Klass | `Bok` |
| Instans/objekt | `bok1` och `bok2` |
| Attribut | `titel` |
| Attributvärde | `"Python för nybörjare"` för `bok1` |
| Skapa en instans | `Bok("Python för nybörjare")` |

**Minnesregel:** Klassen beskriver *vilken typ av sak* vi har. Instansen är *en viss sak* av den typen.

## 8. Vanliga nybörjarmisstag

### Misstag A: Glömma `self.`

```python
class Person:
    def __init__(self, namn):
        namn = namn  # Sparar INTE namnet som ett attribut
```

Rätt:

```python
class Person:
    def __init__(self, namn):
        self.namn = namn
```

### Misstag B: Glömma `self` i metodens definition

```python
class Person:
    def hej():  # Fel när metoden anropas via en instans
        print("Hej")
```

Rätt:

```python
class Person:
    def hej(self):
        print("Hej")
```

### Misstag C: Blanda ihop klassen och objektet

```python
class Bil:
    def __init__(self, marke):
        self.marke = marke

min_bil = Bil("Volvo")
print(min_bil.marke)  # Volvo
```

`Bil` är klassen. `min_bil` är instansen. Det är `min_bil` som har fått sitt specifika bilmärke.

## 9. Övningar

**Övning 1 – Läs kod**

```python
class Film:
    def __init__(self, titel, ar):
        self.titel = titel
        self.ar = ar

film1 = Film("Matrix", 1999)
```

Svara på följande:

1. Vad heter klassen?
2. Vad heter instansen?
3. Vilka två attribut finns?
4. Vad blir `film1.ar`?

**Övning 2 – Skriv en klass**

Skapa en klass `Spelare` som tar emot `namn` och `poang` i `__init__`. Skapa en instans och skriv ut namnet.

**Övning 3 – Lägg till en metod**

Utöka `Spelare` med metoden `fa_poang(self, antal)` som ökar spelarens poäng med `antal`.

## 10. Facit

**Övning 1:**

1. `Film`
2. `film1`
3. `titel` och `ar`
4. `1999`

**Övning 2 och 3 – en möjlig lösning:**

```python
class Spelare:
    def __init__(self, namn, poang):
        self.namn = namn
        self.poang = poang

    def fa_poang(self, antal):
        self.poang += antal

spelare1 = Spelare("Alex", 0)
print(spelare1.namn)  # Alex

spelare1.fa_poang(10)
print(spelare1.poang)  # 10
```

`self.poang += antal` är en kortare form av `self.poang = self.poang + antal`.

## 11. Snabbreferens

| Begrepp | Kort förklaring |
|---|---|
| OOP | Programmering där data och beteenden organiseras i objekt. |
| `class` | Definierar en klass. |
| Klass | Mall för objekt. |
| Instans/objekt | Ett exemplar skapat från en klass. |
| `__init__` | Metod som körs när instansen skapas. |
| `self` | Hänvisar till den aktuella instansen. |
| Attribut | Information som hör till objektet, till exempel `self.namn`. |
| Metod | Funktion definierad i klassen. |
| `objekt.metod()` | Anropar en metod på objektet. |
| `objekt.attribut` | Läser ett attribut från objektet. |

---

**Käll- och kursplansmarkering:** Detta är ett fristående pedagogiskt komplement, **inte** en verifierad sammanfattning av lärarens video eller en garanti för vilka moment kursen kräver. Jämför med lärarens material när det finns tillgängligt.
