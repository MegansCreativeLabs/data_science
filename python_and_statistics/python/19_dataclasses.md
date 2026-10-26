# Lektion 19 – Dataklasser (`@dataclass`) i Python

> **Utgångspunkt:** Lärarens video om dataklasser. Den här lektionen följer genomgångens exempel (`Item`, `Order`, `Point`, `Account`) och lägger till förklaringar av syntax, körbara kodexempel, vanliga misstag och övningar med facit.
>
> **Tidigare lektioner:** Vi har arbetat med klasser, instanser, `self`, `__init__`, `__repr__`, `__eq__`, inkapsling och properties. Lektion 17 och 18 är reserverade för videor som vi ska återkomma till.

## 1. Varför finns dataklasser?

En **klass** är en ritning för att skapa objekt. En **instans** är ett objekt som skapats från ritningen.

Ibland vill vi mest lagra några värden som hör ihop: en varas namn, pris och antal. Då behöver vi ofta skriva samma typ av kod om och om igen:

- `__init__` för att ta emot och spara värden.
- `__repr__` för att visa objektets innehåll.
- `__eq__` för att jämföra två objekts innehåll.

En **dataklass** låter Python skriva mycket av den standardkoden åt oss.

### Vanlig klass – vi skriver allt själva

```python
class Item:
    def __init__(self, name: str, price: float, quantity: int = 1):
        self.name = name
        self.price = price
        self.quantity = quantity

    def __repr__(self):
        return f"Item(name={self.name!r}, price={self.price!r}, quantity={self.quantity!r})"

    def __eq__(self, other):
        if not isinstance(other, Item):
            return NotImplemented
        return (
            self.name == other.name
            and self.price == other.price
            and self.quantity == other.quantity
        )
```

Det fungerar, men det är mycket kod för tre fält.

### Dataklass – samma grundfunktioner med mindre kod

```python
from dataclasses import dataclass

@dataclass
class Item:
    name: str
    price: float
    quantity: int = 1

item1 = Item("Banan", 5.0, 3)
item2 = Item("Banan", 5.0, 3)

print(item1)
print(item1 == item2)
```

**Utskrift:**

```text
Item(name='Banan', price=5.0, quantity=3)
True
```

Python har automatiskt skapat en konstruktor (`__init__`), en informativ representation (`__repr__`) och en innehållsbaserad jämförelse (`__eq__`).

**Viktigt:** En dataklass är fortfarande en helt vanlig Python-klass i den meningen att vi kan skapa instanser och skriva egna metoder. `@dataclass` lägger till automatiskt genererade funktioner.

---

## 2. Förstå varje del av syntaxen

```python
from dataclasses import dataclass

@dataclass
class Item:
    name: str
    price: float
    quantity: int = 1
```

| Kod | Betydelse |
|---|---|
| `from dataclasses import dataclass` | Hämtar `dataclass` från Pythons standardbibliotek. Ingen `pip install` behövs. |
| `@dataclass` | En **dekorator** som ber Python skapa standardmetoder för klassen. |
| `class Item:` | Skapar en klass som heter `Item`. |
| `name: str` | Ett fält som förväntas innehålla text. |
| `price: float` | Ett fält som förväntas innehålla ett decimaltal. |
| `quantity: int = 1` | Ett heltalsfält med standardvärdet `1`. |

En **type hint** som `: float` talar om vilken typ vi förväntar oss, men den omvandlar eller kontrollerar **inte automatiskt** värdet vid körning.

### Standardvärden

```python
one = Item("Banan", 5.0)
three = Item("Banan", 5.0, 3)

print(one.quantity)    # 1
print(three.quantity)  # 3
```

Om vi inte skickar in `quantity` används standardvärdet `1`.

**Ordningen spelar roll:** Fält utan standardvärde måste normalt komma före fält med standardvärde.

---

## 3. Dataklasser jämför värden, inte bara identitet

```python
item1 = Item("Banan", 5.0, 3)
item2 = Item("Banan", 5.0, 3)

print(item1 == item2)  # True
print(item1 is item2)  # False
```

- `==` frågar om objekten **räknas som lika**. Dataklassen jämför normalt motsvarande fält.
- `is` frågar om variablerna pekar på **exakt samma objekt**.

Två olika instanser kan alltså ha samma innehåll utan att vara samma objekt.

```python
item3 = item1
print(item1 is item3)  # True
```

Här pekar båda variablerna på samma instans.

---

## 4. Problemet med en lista som standardvärde

Läraren skapar en beställning som har en kund och en lista med varor.

Vi vill kunna skriva:

```python
order1 = Order("Anna")
order2 = Order("Emil")
```

Varje beställning ska ha **sin egen tomma lista**.

Det här är **fel** i en dataklass:

```python
from dataclasses import dataclass

@dataclass
class Order:
    customer: str
    items: list[Item] = []  # FEL!
```

Python ger ett fel som säger att vi ska använda `default_factory`. En gemensam föränderlig lista som standardvärde skulle kunna orsaka att olika objekt delar samma lista.

### Lösningen: `field(default_factory=list)`

```python
from dataclasses import dataclass, field

@dataclass
class Order:
    customer: str
    items: list[Item] = field(default_factory=list)

order1 = Order("Anna")
order2 = Order("Emil")

order1.items.append(Item("Banan", 5.0))

print(order1.items)
print(order2.items)
```

**Utskrift:**

```text
[Item(name='Banan', price=5.0, quantity=1)]
[]
```

### Vad betyder `default_factory`?

- `field(...)` används för att konfigurera ett fält.
- `default_factory=list` betyder: **skapa en ny tom lista för varje ny instans**.
- Vi skriver `list`, **inte** `list()`. Vi skickar själva funktionen/typen som Python ska anropa senare.

För en dictionary kan vi använda `field(default_factory=dict)` och för en mängd `field(default_factory=set)`.

**Extra förtydligande:** I modern Python behöver vi normalt inte importera `List` från `typing` för det här exemplet. `list[Item]` fungerar direkt.

---

## 5. `frozen=True` – förhindra vanlig ändring av fält

En vanlig dataklass är ändringsbar:

```python
@dataclass
class Point:
    x: float
    y: float

p = Point(3.0, 4.0)
p.x = 10.0
print(p)  # Point(x=10.0, y=4.0)
```

Men ibland vill vi att värdena ska vara fasta efter att objektet skapats.

```python
@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(3.0, 4.0)
print(p)

p.x = 10.0  # Ger FrozenInstanceError
```

`frozen=True` betyder att Python hindrar vanliga försök att tilldela nya värden till objektets fält.

### Hur gör vi om vi vill ha nya koordinater?

Skapa ett nytt objekt:

```python
p1 = Point(3.0, 4.0)
p2 = Point(10.0, p1.y)

print(p1)  # Point(x=3.0, y=4.0)
print(p2)  # Point(x=10.0, y=4.0)
```

Det finns också ett hjälpfunktion `replace`:

```python
from dataclasses import replace

p3 = replace(p1, x=10.0)
print(p3)  # Point(x=10.0, y=4.0)
```

**Viktig nyans:** `frozen=True` gör inte automatiskt allt inuti objektet oföränderligt. Om ett fält innehåller en lista kan listans innehåll fortfarande ändras. Det är alltså inte en fullständig säkerhetsspärr.

---

## 6. `__post_init__` – kontrollera värden när objektet skapas

En dataklass genererar `__init__` automatiskt. Men hur gör vi om vi vill kontrollera inmatade värden?

Läraren använder ett konto där saldot inte får vara negativt vid skapandet.

```python
from dataclasses import dataclass

@dataclass
class Account:
    owner: str
    balance: float = 0.0

    def __post_init__(self):
        if self.balance < 0:
            raise ValueError("Balance may not be negative")

        self.balance = float(self.balance)

account = Account("Anna", 100)
print(account)
```

**Utskrift:**

```text
Account(owner='Anna', balance=100.0)
```

### Vad händer steg för steg?

1. `Account("Anna", 100)` skapar en ny instans.
2. Dataklassens automatiska `__init__` tilldelar `owner` och `balance`.
3. Python anropar `__post_init__` direkt efteråt.
4. `if self.balance < 0:` kontrollerar om saldot är negativt.
5. `raise ValueError(...)` stoppar skapandet om värdet är ogiltigt.
6. Annars omvandlas saldot till `float`.

```python
Account("Anna", -25)
# ValueError: Balance may not be negative
```

### `__post_init__` körs inte vid senare ändringar

Detta är en av videons viktigaste lärdomar:

```python
account = Account("Anna", 100)
account.balance = -25

print(account.balance)  # -25
```

**Varför tillåts detta?** `__post_init__` körs **en gång per instans när den skapas**, inte varje gång vi ändrar ett attribut.

Om vi behöver kontrollera **varje ändring** kan vi exempelvis använda `@property` med en setter, som i lektion 16, eller designa klassen så att saldot bara ändras genom särskilda metoder.

**En detalj att känna till:** Koden ovan kontrollerar först `self.balance < 0` och konverterar sedan till `float`. Det fungerar för numeriska värden som i videon. Om användaren skickar text som `"100"` blir jämförelsen mot `0` ett `TypeError` innan konverteringen. Vid behov kan vi konvertera före kontrollen.

---

## 7. När ska vi välja dataklass eller vanlig klass?

| Situation | Ofta lämpligt val | Varför? |
|---|---|---|
| En vara med namn, pris och antal | `@dataclass` | Mest data och enkel struktur. |
| En punkt med `x` och `y` | `@dataclass` | Ett tydligt värdeobjekt. |
| En order med en lista över varor | `@dataclass` + `field(default_factory=list)` | Praktiskt att samla data, men varje order behöver egen lista. |
| Ett konto med insättningar, uttag och flera regler | Vanlig klass kan vara tydligare | Mer beteende och kontroll över tillståndet. |
| Komplicerad initiering eller arvshierarki | Överväg vanlig klass | Kan bli enklare att förstå och styra. |

Det här är **tumregler, inte absoluta förbud**. Dataklasser kan också innehålla metoder och användas med arv. Frågan är vilken lösning som gör koden tydligast.

---

## 8. Sammanhängande praktiskt exempel

Vi bygger en liten order med varor och en metod som räknar totalsumman.

```python
from dataclasses import dataclass, field

@dataclass
class Item:
    name: str
    price: float
    quantity: int = 1

    def line_total(self) -> float:
        return self.price * self.quantity


@dataclass
class Order:
    customer: str
    items: list[Item] = field(default_factory=list)

    def total(self) -> float:
        return sum(item.line_total() for item in self.items)


order = Order("Anna")
order.items.append(Item("Banan", 5.0, 3))
order.items.append(Item("Äpple", 4.0, 2))

print(order.customer)
print(order.total())
```

**Utskrift:**

```text
Anna
23.0
```

Beräkningen är `5.0 * 3 + 4.0 * 2 = 23.0`.

Här ser vi att dataklasser **får ha egna metoder**. De är inte begränsade till att enbart lagra värden.

---

## 9. Vanliga misstag

| Misstag | Vad händer? | Rätt tanke |
|---|---|---|
| Glömmer `@dataclass` | Ingen automatisk `__init__` för fälten. | Sätt dekoratorn direkt ovanför `class`. |
| Skriver `items: list = []` | Dataklassen avvisar det muterbara standardvärdet. | Använd `field(default_factory=list)`. |
| Skriver `default_factory=list()` | Skickar en färdig lista i stället för en anropbar fabrik. | Skriv `default_factory=list`. |
| Tror att `name: str` stoppar alla andra typer | Type hints kontrolleras inte automatiskt vid körning. | Lägg till validering om det behövs. |
| Tror att `__post_init__` körs vid varje ändring | Ogiltiga värden kan tilldelas senare. | Använd setter/metoder för fortlöpande kontroll. |
| Försöker skriva `p.x = 10` när `frozen=True` | `FrozenInstanceError`. | Skapa ett nytt objekt. |
| Tror att `==` och `is` är samma sak | Blandar ihop innehåll och objektidentitet. | `==` jämför värden; `is` identitet. |

---

## 10. Quick Reference

| Begrepp | Syntax | Funktion |
|---|---|---|
| Import | `from dataclasses import dataclass` | Hämtar dekoratorn. |
| Dataklass | `@dataclass` | Genererar standardmetoder. |
| Fält | `name: str` | Definierar ett fält med type hint. |
| Standardvärde | `quantity: int = 1` | Används om argumentet utelämnas. |
| Lista som standard | `field(default_factory=list)` | Ger varje instans en egen lista. |
| Skrivskyddad tilldelning | `@dataclass(frozen=True)` | Hindrar vanliga fälttilldelningar. |
| Kontroll efter init | `def __post_init__(self):` | Körs direkt efter genererad initiering. |
| Likhetsjämförelse | `item1 == item2` | Jämför normalt fältvärden. |
| Identitet | `item1 is item2` | Frågar om samma objekt. |
| Ny instans med ändring | `replace(p, x=10)` | Kopierar dataklassens värden med ändring. |

---

## 11. Övningar

Försök gärna läsa facit direkt om du behöver komma vidare i kursen. Använd sedan exemplen som repetition när du har mer tid.

**Övning 1.** Skapa en dataklass `Book` med `title: str`, `author: str` och `pages: int`. Skapa en bok och skriv ut den.

**Övning 2.** Lägg till `available: bool = True` i `Book`. Skapa en bok utan att ange `available`. Vad blir värdet?

**Övning 3.** Skapa två separata `Book`-instanser med samma värden. Testa både `==` och `is`. Förklara skillnaden.

**Övning 4.** Skapa en dataklass `Library` med `name` och en lista `books`. Varje bibliotek ska få sin egen tomma lista.

**Övning 5.** Skapa en fryst dataklass `Coordinate` med `x` och `y`. Skapa en punkt och försök ändra `x`.

**Övning 6.** Skapa en dataklass `Product` med `name` och `price`. Använd `__post_init__` för att hindra negativa priser. Undersök vad som händer om du ändrar `price` till ett negativt värde efter att instansen skapats.

---

## 12. Facit

### Facit 1

```python
from dataclasses import dataclass

@dataclass
class Book:
    title: str
    author: str
    pages: int

book = Book("Python för nybörjare", "Anna", 250)
print(book)
# Book(title='Python för nybörjare', author='Anna', pages=250)
```

### Facit 2

```python
@dataclass
class Book:
    title: str
    author: str
    pages: int
    available: bool = True

book = Book("Python för nybörjare", "Anna", 250)
print(book.available)  # True
```

### Facit 3

```python
book1 = Book("Python för nybörjare", "Anna", 250)
book2 = Book("Python för nybörjare", "Anna", 250)

print(book1 == book2)  # True: samma fältvärden
print(book1 is book2)  # False: olika instanser
```

### Facit 4

```python
from dataclasses import dataclass, field

@dataclass
class Library:
    name: str
    books: list[Book] = field(default_factory=list)

first = Library("Stadsbiblioteket")
second = Library("Skolbiblioteket")

first.books.append(Book("Python för nybörjare", "Anna", 250))
print(len(first.books))   # 1
print(len(second.books))  # 0
```

### Facit 5

```python
@dataclass(frozen=True)
class Coordinate:
    x: float
    y: float

point = Coordinate(3.0, 4.0)
# point.x = 10.0  # FrozenInstanceError om raden aktiveras
```

### Facit 6

```python
@dataclass
class Product:
    name: str
    price: float

    def __post_init__(self):
        if self.price < 0:
            raise ValueError("Priset får inte vara negativt")

product = Product("Penna", 12.0)
product.price = -5.0
print(product.price)  # -5.0
```

Det sista visar varför `__post_init__` **inte ersätter** en setter eller andra regler för senare ändringar.

---

## 13. Sammanfattning – det viktigaste att minnas

En **dataklass** är ett smidigt sätt att skapa klasser som främst innehåller data. Med `@dataclass` slipper vi skriva mycket standardkod. Med `field(default_factory=list)` ser vi till att varje instans får sin egen lista. Med `frozen=True` hindrar vi vanliga ändringar av fält. Med `__post_init__` kan vi kontrollera värden direkt efter att objektet skapats – **men inte automatiskt vid senare ändringar**.

**Minnesregel:** *Mycket data och lite logik? Tänk dataklass. Många regler och mycket beteende? Fundera på om en vanlig klass blir tydligare.*
