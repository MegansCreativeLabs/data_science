# Lektion 15 – Objektens utskrift, jämförelser och type hints

> **Vecka 3 · Python · Klasser och objekt**  
> Baserad på lärarens video (cirka 25 minuter). Videon fortsätter på klassen `Point` från föregående genomgång, som vi ännu inte har transkriberat (reserverad lektion 14). Därför börjar vi med en komplett, enkel version av klassen.

## Målet med lektionen

Efter lektionen ska du kunna förklara varför `print(objekt)` ibland visar en konstig teknisk text, vad `__str__`, `__repr__` och `__eq__` gör, skillnaden mellan `==` och `is`, och varför man skriver `x: int` och `-> float` i kod.

## Quick Reference

| Syntax | Betydelse |
|---|---|
| `class Point:` | Skapar en klass – en mall för objekt. |
| `Point(3, 4)` | Skapar en **instans** (ett objekt) av klassen. |
| `self.x` | Attributet `x` på just detta objekt. |
| `__init__(self, x, y)` | Körs när ett nytt objekt skapas och ger det startvärden. |
| `__str__(self)` | Bestämmer människovänlig text, exempelvis vid `print(objekt)`. |
| `__repr__(self)` | Bestämmer mer teknisk och informativ representation, exempelvis i en lista med objekt. |
| `__eq__(self, other)` | Bestämmer vad `objekt1 == objekt2` betyder. |
| `is` | Kontrollerar om två variabler syftar på **exakt samma objekt**. |
| `==` | Kontrollerar om två värden anses lika. För egna klasser kan vi definiera hur. |
| `isinstance(other, Point)` | Kontrollerar om `other` är en instans av `Point` eller en underklass. |
| `NotImplemented` | Signalerar att en viss jämförelse inte hanteras av metoden; Python kan prova annat. |
| `x: int` | **Type hint**: x förväntas vara ett heltal. |
| `-> float` | **Return type hint**: funktionen förväntas returnera ett flyttal. |
| `-> None` | Funktionen/metoden förväntas inte returnera något värde. |

---

## 1. Först: vad är en klass och en instans?

En **klass** är en beskrivning eller mall. En **instans** är ett faktiskt objekt som skapas från mallen. Ett **attribut** är ett värde som hör till objektet, och en **metod** är en funktion som hör till klassen.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print(p.x)  # 3
print(p.y)  # 4
```

**Rad för rad:**

1. `class Point:` definierar mallen.
2. `__init__` tar emot startvärden när vi skriver `Point(3, 4)`.
3. `self` betyder det objekt som håller på att skapas eller användas.
4. `self.x = x` sparar parametern `x` som ett attribut på objektet.
5. `p` är en variabel som refererar till instansen.

> **Viktigt:** Videon utgår från en tidigare skapad `Point`-klass med bland annat metoderna `move` och `distance_to_origin`. Koden ovan är en förenklad start så att den här lektionen går att läsa fristående.

## 2. Varför visar `print(p)` konstig text?

Om vi inte berättar hur objektet ska skrivas ut kan Python visa något som liknar:

```text
<__main__.Point object at 0x...>
```

Det berättar vilken sorts objekt det är, men inte tydligt att punkten har koordinaterna 3 och 4. Vi kan själv bestämma hur objektet representeras.

## 3. `__repr__` – informativ representation

`__repr__` är en **specialmetod**. Dubbelunderstrecken på båda sidorna ingår i namnet. Vi skriver inte vanligtvis `p.__repr__()` själva; Python kan använda metoden automatiskt, exempelvis via `repr(p)`.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        class_name = type(self).__name__
        return f"{class_name}(x={self.x}, y={self.y})"

p = Point(3, 4)
print(repr(p))  # Point(x=3, y=4)
```

**Vad betyder den ovanliga raden?**

- `type(self)` hämtar typen/klassen för objektet.
- `.__name__` hämtar klassens namn som text.
- `class_name` blir alltså `"Point"`.
- `f"..."` är en **f-sträng**: uttrycken i `{}` ersätts med sina värden.
- `return` lämnar tillbaka texten till Python.

Vi skulle kunna skriva `return f"Point(x={self.x}, y={self.y})"`, men läraren använder klassnamnet dynamiskt så att representationen kan fungera bättre om klassen senare får underklasser eller byter namn.

En bra `__repr__` visar viktiga attribut och hjälper vid felsökning. Den *kan* se ut som giltig kod för att skapa objektet igen, men det är inget absolut krav.

## 4. `__str__` – lättläst text för människor

`__str__` bestämmer hur objektet ska visas när vi använder `str(p)` eller `print(p)`.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"{type(self).__name__}(x={self.x}, y={self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"

p = Point(3, 4)

print(p)        # (3, 4)
print(str(p))   # (3, 4)
print(repr(p))  # Point(x=3, y=4)
```

Tänk så här:

- **`__str__`**: ”Hur vill jag visa punkten för en människa?”
- **`__repr__`**: ”Hur vill jag beskriva punkten tydligt för den som läser eller felsöker kod?”

Om `__str__` saknas använder `str(p)` normalt `__repr__` som reserv.

## 5. `==` jämfört med `is` – två olika frågor

Anta att vi skapar två punkter med samma koordinater:

```python
p1 = Point(3, 4)
p2 = Point(3, 4)
p3 = p1
```

`p1` och `p2` är **två separat skapade objekt**. `p3 = p1` skapar däremot inte en ny punkt: variablerna `p1` och `p3` hänvisar till samma objekt.

```python
print(p1 is p2)  # False
print(p1 is p3)  # True
```

En enkel bild:

```text
p1 ─────┐
        ▼
      Point(3, 4)  [objekt A]
        ▲
p3 ─────┘

p2 ────► Point(3, 4)  [objekt B]
```

`is` frågar: **Är det exakt samma objekt?**

`==` frågar: **Ska dessa objekt räknas som lika?**

För en egen klass utan anpassad `__eq__` innebär `==` normalt samma identitetsjämförelse som objektets standardbeteende. Därför blir `p1 == p2` till en början `False`, trots att koordinaterna är samma.

> Läraren visar en förenklad bild av *stack* och *heap* för att illustrera referenser. Den bilden är inte en generell regel för var Python lagrar olika datatyper. Det viktiga här är att flera variabler kan referera till samma objekt, medan två anrop till `Point(...)` skapar två skilda objekt.

## 6. `__eq__` – bestäm själv vad ”lika” betyder

Vi vill att två `Point`-objekt ska vara lika om **både `x` och `y` är lika**.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"{type(self).__name__}(x={self.x}, y={self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"

    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

p1 = Point(3, 4)
p2 = Point(3, 4)
p3 = p1
p4 = Point(4, 5)

print(p1 == p2)  # True – samma koordinater
print(p1 == p4)  # False – olika koordinater
print(p1 is p2)  # False – olika objekt
print(p1 is p3)  # True – samma objekt
```

### Förstå `__eq__` rad för rad

```python
def __eq__(self, other):
```

- `self` är objektet till vänster om `==`, exempelvis `p1`.
- `other` är objektet till höger om `==`, exempelvis `p2`.

```python
if not isinstance(other, Point):
    return NotImplemented
```

- `isinstance(other, Point)` kontrollerar om `other` är en `Point` eller en underklass.
- `not` vänder på resultatet.
- `NotImplemented` är **inte samma sak som** `False` och inte samma sak som felet `NotImplementedError`. Det betyder att metoden inte hanterar just den jämförelsen och låter Python försöka andra jämförelsevägar.

```python
return self.x == other.x and self.y == other.y
```

- `and` betyder att **båda villkoren måste vara sanna**.
- Om `x` skiljer sig, eller `y` skiljer sig, blir resultatet `False`.

**Obs:** Med `isinstance` kan även underklasser räknas som `Point`. I mer avancerad kod kan man behöva tänka igenom om olika underklasser verkligen ska jämföras som lika. Det ingår inte i lärarens grundexempel.

## 7. Type hints – visa vilka datatyper vi förväntar oss

Läraren avslutar med **type hints** (typannoteringar). De gör koden lättare att förstå och hjälper VS Code att visa bättre information.

```python
def __init__(self, x: int, y: int):
    self.x = x
    self.y = y
```

`x: int` betyder: ”Vi förväntar oss att `x` är ett heltal.”

**Det betyder inte att Python automatiskt stoppar andra datatyper.** Om du skriver `Point(2.5, 5.0)` kan koden fortfarande fungera, även om typen inte stämmer med anteckningen. En type hint är i första hand information till läsare, editor och typkontrollverktyg.

### Returtyper med `->`

```python
import math

class Point:
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

    def distance_to_origin(self) -> float:
        return math.sqrt(self.x**2 + self.y**2)

    def move(self, dx: float, dy: float) -> None:
        self.x += dx
        self.y += dy
```

- `x: float` och `y: float`: vi förväntar oss decimaltal (heltal kan också användas i praktiken).
- `-> None`: metoden ger inget värde tillbaka med `return`.
- `-> float`: metoden förväntas lämna tillbaka ett flyttal.
- `dx` och `dy`: förändring i x-led respektive y-led.
- `self.x += dx` betyder samma som `self.x = self.x + dx`.
- `math.sqrt(...)` räknar kvadratroten. Avståndet till origo `(0, 0)` beräknas med Pythagoras sats.

**Exempel:**

```python
p = Point(3, 4)
print(p.distance_to_origin())  # 5.0
p.move(2, 1)
print(p.x, p.y)                # 5 5
```

I exemplet blir koordinaterna heltal eftersom vi skickade in heltal. Typannoteringen `float` tvingar inte automatiskt fram en konvertering.

### Type hints på attribut

```python
self.x: float = x
self.y: float = y
```

Det anger vilken typ attributen förväntas ha. Det är **inte** samma sak som att skriva `self.x = float(x)`; det senare försöker faktiskt konvertera värdet.

## 8. Komplett exempel att kopiera och köra

```python
import math

class Point:
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        return f"{type(self).__name__}(x={self.x}, y={self.y})"

    def __str__(self) -> str:
        return f"({self.x}, {self.y})"

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def distance_to_origin(self) -> float:
        return math.sqrt(self.x**2 + self.y**2)

    def move(self, dx: float, dy: float) -> None:
        self.x += dx
        self.y += dy


p1 = Point(3, 4)
p2 = Point(3, 4)
p3 = p1

print(p1)                       # (3, 4)
print(repr(p1))                 # Point(x=3, y=4)
print(p1 == p2)                 # True
print(p1 is p2)                 # False
print(p1 is p3)                 # True
print(p1.distance_to_origin())  # 5.0

p1.move(2, 1)
print(p1)                       # (5, 5)
print(p3)                       # (5, 5) – samma objekt som p1!
print(p2)                       # (3, 4) – separat objekt
```

### Särskilt viktigt: varför ändras även `p3`?

Eftersom `p3 = p1` betyder att **båda namnen pekar på samma instans**. `move()` ändrar den instansens attribut. `p2` är ett annat objekt och påverkas inte.

## 9. Vanliga misstag

| Misstag | Förklaring / lösning |
|---|---|
| Skriva `_str_` | Metoden måste heta exakt `__str__` med två understreck på varje sida. |
| Använda `print()` inuti `__str__` i stället för `return` | `__str__` måste **returnera en sträng**. |
| Tro att `==` alltid betyder samma objekt | `==` kan jämföra innehåll; `is` jämför identitet. |
| Tro att `p3 = p1` kopierar objektet | Det skapar bara ett nytt namn för samma objekt. |
| Skriva `return False` för andra typer utan att tänka efter | `NotImplemented` låter Python hantera jämförelsen vidare. |
| Skriva `NotImplemented()` | `NotImplemented` är ett särskilt värde, inte en funktion. |
| Tro att `x: int` omvandlar till heltal | Det är bara en typannotering. För konvertering behövs exempelvis `int(x)`. |
| Glömma köra klassens cell igen i Jupyter | Om du ändrar klassdefinitionen måste du köra om cellen och oftast skapa nya instanser för att använda den nya versionen. |

## 10. Ordlista

- **Klass:** Mall som beskriver hur objekt skapas och fungerar.
- **Instans/objekt:** Ett konkret exemplar av en klass.
- **Attribut:** Ett värde på objektet, som `self.x`.
- **Metod:** Funktion definierad i en klass.
- **Representation:** Text som beskriver objektet.
- **Identitet:** Om två referenser avser exakt samma objekt.
- **Likhet:** Om två värden anses ha samma innehåll eller värde.
- **Referens:** Koppling från ett variabelnamn till ett objekt.
- **Type hint/typannotering:** Information om förväntad datatyp.
- **Returvärde:** Värdet en funktion lämnar tillbaka med `return`.

---

## 11. Övningar

**Övning 1.** Skapa `Point(2, 7)` och skriv ut dess `x` och `y`.

**Övning 2.** Lägg till en `__str__` som ger texten `Punkt: 2, 7` för objektet i övning 1.

**Övning 3.** Lägg till `__repr__` som ger `Point(x=2, y=7)`.

**Övning 4.** Skapa `a = Point(2, 7)`, `b = Point(2, 7)` och `c = a`. Vad ger `a is b` och `a is c`? Förklara varför.

**Övning 5.** Lägg till `__eq__` så att två punkter med samma koordinater jämförs som lika.

**Övning 6.** Skriv type hints på en metod `move(self, dx, dy)` som tar emot två `float` och inte returnerar något värde.

## 12. Facit

**1.**

```python
p = Point(2, 7)
print(p.x)  # 2
print(p.y)  # 7
```

**2.** Lägg till i klassen:

```python
def __str__(self):
    return f"Punkt: {self.x}, {self.y}"
```

**3.** Lägg till i klassen:

```python
def __repr__(self):
    return f"{type(self).__name__}(x={self.x}, y={self.y})"
```

**4.** `a is b` blir `False` eftersom de skapats var för sig. `a is c` blir `True` eftersom `c = a` hänvisar till samma objekt.

**5.**

```python
def __eq__(self, other):
    if not isinstance(other, Point):
        return NotImplemented
    return self.x == other.x and self.y == other.y
```

**6.**

```python
def move(self, dx: float, dy: float) -> None:
    self.x += dx
    self.y += dy
```

---

### Sammanfattning

`__str__` styr hur ett objekt visas för människor. `__repr__` ger en informativ representation. `__eq__` styr vad `==` betyder för dina objekt, medan `is` alltid frågar om det är exakt samma objekt. Type hints hjälper oss att förstå vad en metod tar emot och returnerar, men de tvingar inte automatiskt Python att kontrollera typerna.
