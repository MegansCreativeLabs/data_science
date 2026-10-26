# Lektion 16 – Inkapsling och properties i Python

**Vecka 3 · Objektorienterad programmering · Fördjupning**

Den här lektionen bygger på lärarens genomgång av **inkapsling**, **`@property`**, **setters**, **validering** och **name mangling**. Vi börjar från grunden och använder två exempel från videon: en punkt (`Point`) och ett konto (`Account`).

## Quick Reference

| Syntax | Betydelse |
|---|---|
| `self.owner` | Publikt attribut som andra kan använda |
| `self._balance` | Internt attribut enligt Python-konvention |
| `self.__token` | Namnändras automatiskt (name mangling) |
| `@property` | Gör att en metod kan läsas som ett attribut, utan `()` |
| `@balance.setter` | Kör kod när någon tilldelar `objekt.balance = värde` |
| `raise ValueError("...")` | Stoppar en ogiltig tilldelning med ett fel |
| `float(value)` | Omvandlar värdet till ett flyttal, om det går |

## 1. Vad är inkapsling?

**En klass** är en mall för objekt. **Ett objekt**, eller en **instans**, skapas från klassen. **Attribut** är information som hör till objektet, och **metoder** är funktioner som hör till klassen.

**Inkapsling** betyder att vi visar ett enkelt sätt att använda objektet, samtidigt som vi håller detaljerna om hur det fungerar internt för sig själva.

Tänk dig ett konto: den som använder kontot behöver veta hur man läser saldot och gör en insättning. Personen behöver inte känna till hur saldot lagras bakom kulisserna.

I Python är ett inledande understreck en **överenskommelse**, inte ett lås:

```python
self.owner = "Joakim"       # Publikt: avsett att användas utifrån
self._balance = 100.0       # Internt: bör normalt inte ändras utifrån
```

`_balance` går tekniskt sett att nå utifrån. Understrecket signalerar: **det här är en intern detalj – använd hellre klassens publika gränssnitt**.

## 2. `@property`: en beräkning som ser ut som ett attribut

I videon utgår läraren från klassen `Point` och gör om en metod för avstånd till origo till en property som heter `magnitude`.

```python
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    @property
    def magnitude(self) -> float:
        return (self.x**2 + self.y**2) ** 0.5

    def move(self, dx: float, dy: float) -> None:
        self.x += dx
        self.y += dy


p = Point(3, 4)
print(p.magnitude)  # 5.0
p.move(1, 0)
print(p.magnitude)  # Cirka 5.6569
```

**Steg för steg:**

1. `@property` står direkt ovanför metoden `magnitude`.
2. Metoden använder `self.x` och `self.y` för att räkna ut avståndet till origo.
3. När vi skriver `p.magnitude` kör Python beräkningen automatiskt.
4. Vi skriver **inte** `p.magnitude()` eftersom det nu är en property.
5. När punkten flyttas beräknas ett nytt avstånd nästa gång vi läser `p.magnitude`.

Jämför:

```python
p.move(1, 0)       # Vanlig metod: parenteser
print(p.magnitude) # Property: inga parenteser
print(p.x)         # Vanligt attribut: inga parenteser
```

En property passar bra när något **upplevs som en egenskap** hos objektet. Undvik helst väldigt tunga eller oväntade beräkningar i en property, eftersom den kan köras varje gång värdet läses.

## 3. Varför behöver vi en setter?

I videons andra exempel har vi ett konto. Vi vill kunna skriva:

```python
konto.balance = 500
```

Men vi vill **inte** tillåta ett negativt saldo. Med en **setter** kan vi kontrollera värdet innan det sparas.

Det finns två olika namn att hålla reda på:

- `balance` är det **publika gränssnittet**. När någon läser eller skriver det körs property-koden.
- `_balance` är den **interna lagringsplatsen** där själva talet finns.

De heter nästan likadant eftersom de hör ihop, men de gör **olika jobb**.

## 4. Hela klassen `Account`, steg för steg

```python
class Account:
    def __init__(self, owner: str, balance: float = 0.0) -> None:
        self.owner = owner
        self._balance = 0.0
        self.balance = balance

    @property
    def balance(self) -> float:
        return self._balance

    @balance.setter
    def balance(self, value: float) -> None:
        if value < 0:
            raise ValueError("Balance may not be negative")
        self._balance = float(value)

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self.balance = self.balance + amount

    def withdraw(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Withdrawal must be positive")

        new_value = self.balance - amount
        if new_value < 0:
            raise ValueError("Insufficient funds")

        self.balance = new_value
```

### Vad händer i `__init__`?

```python
self._balance = 0.0
self.balance = balance
```

Första raden skapar lagringsplatsen. Andra raden använder **settern**, så även startvärdet kontrolleras. Om någon skriver `Account("Joakim", -100)` får personen ett `ValueError`.

`balance: float = 0.0` betyder att parametern `balance` har **standardvärdet** `0.0`. Om vi inte anger något saldo börjar kontot på noll.

### Vad händer när vi läser saldot?

```python
konto = Account("Joakim", 100)
print(konto.balance)  # 100.0
```

Python hittar `@property`-metoden och kör `return self._balance`. Det interna värdet skickas tillbaka.

### Vad händer när vi ändrar saldot?

```python
konto.balance = 500
```

Python hittar `@balance.setter`. Parametern `value` får värdet `500`. Settern kontrollerar att värdet inte är negativt och sparar sedan `500.0` i `_balance`.

### Insättning och uttag

```python
konto = Account("Joakim", 100)

konto.deposit(50)
print(konto.balance)  # 150.0

konto.withdraw(40)
print(konto.balance)  # 110.0

# konto.withdraw(200)  # ValueError: Insufficient funds
```

`deposit()` kontrollerar att insättningen är större än noll. `withdraw()` kontrollerar både att uttaget är positivt och att det finns tillräckligt med pengar.

**Viktig detalj:** I slutet av lärarens exempel visas också hur man kan skriva direkt till `_balance` *inifrån klassen*, efter en egen kontroll. Det fungerar, men i vår version använder vi `self.balance = new_value` så att all tilldelning går genom samma setter. Det är lättare att följa som nybörjare.

**En begränsning i videons design:** Även om klassen har `deposit()` och `withdraw()` kan användaren fortfarande skriva `konto.balance = 500` direkt. Vill man *förbjuda* direkt ändring behövs en annan design, till exempel en property utan publik setter. Lärarens exempel fokuserar på validering, inte på att låsa saldot helt.

## 5. Den vanliga fällan: oändlig rekursion i en setter

**Fel:**

```python
@balance.setter
def balance(self, value):
    self.balance = value  # FEL!
```

Varför? Tilldelningen `self.balance = value` anropar **samma setter igen**, som gör samma sak igen, och så vidare. Till slut får vi ett fel, vanligtvis `RecursionError`.

**Rätt:**

```python
@balance.setter
def balance(self, value):
    if value < 0:
        raise ValueError("Balance may not be negative")
    self._balance = float(value)  # Sparar i det interna attributet
```

Kom ihåg: **`balance` kontrollerar – `_balance` lagrar.**

## 6. Vad är validering och `raise`?

**Validering** betyder att vi kontrollerar om ett värde följer reglerna innan vi använder det.

```python
if amount <= 0:
    raise ValueError("Deposit must be positive")
```

- `if` undersöker villkoret.
- `amount <= 0` betyder *beloppet är mindre än eller lika med noll*.
- `raise` betyder *utlös ett fel*.
- `ValueError` betyder att värdet inte är godtagbart för operationen.

Det är skillnad på att **returnera** ett värde och att **raise** ett fel. `return` ger ett resultat; `raise` avbryter det normala flödet och signalerar att något gick fel.

## 7. Publikt, internt och name mangling

| Skrivsätt | Exempel | Tolkning |
|---|---|---|
| Utan understreck | `owner` | Publikt attribut |
| Ett inledande understreck | `_balance` | Internt enligt konvention |
| Två inledande understreck | `__token` | Python ändrar attributnamnet automatiskt inom klassen |

### Exempel från videon: `__token`

```python
class Demo:
    def __init__(self):
        self.__token = "secret"

    def reveal(self):
        return self.__token


d = Demo()
print(d.reveal())       # secret
# print(d.__token)      # AttributeError
print(d._Demo__token)   # secret – tekniskt möjligt, men bör undvikas
```

När vi skriver `__token` inne i klassen `Demo` använder Python i praktiken namnet `_Demo__token`. Detta kallas **name mangling**. Syftet är främst att minska risken för namnkrockar, särskilt vid **arv** (när en klass bygger vidare på en annan).

Det här är **inte ett säkerhetsskydd**. Det är alltså inte rätt sätt att skydda hemliga lösenord eller nycklar.

## 8. Vanliga misstag

| Misstag | Varför det blir fel | Gör så här |
|---|---|---|
| `p.magnitude()` | En property läses utan parenteser | `p.magnitude` |
| `self.balance = value` inne i `balance`-settern | Settern anropar sig själv om och om igen | Spara i `self._balance` |
| Skriva direkt till `konto._balance` utifrån | Hoppar över valideringen | Använd `konto.balance` eller klassens metoder |
| Tro att `_balance` är strikt privat | Ett understreck är bara en konvention | Behandla det som internt |
| Tro att `__token` är säkert hemligt | Name mangling är inte säkerhet | Förvara inte hemligheter så |
| Tro att `balance: float` tvingar typen automatiskt | Type hints är främst information till läsare och verktyg | Validera eller konvertera uttryckligen vid behov |

## 9. Ordlista

- **Inkapsling:** att dölja interna detaljer bakom ett tydligt gränssnitt.
- **Publikt attribut:** en egenskap som är tänkt att användas utifrån.
- **Internt attribut:** en detalj som andra helst inte ska använda direkt.
- **Property:** en metod som kan läsas som ett attribut.
- **Getter:** kod som hämtar ett värde, exempelvis metoden under `@property`.
- **Setter:** kod som körs när ett värde tilldelas.
- **Validering:** kontroll av att ett värde är tillåtet.
- **Name mangling:** Pythons automatiska namnändring av vissa attribut med två inledande understreck.
- **API:** de delar av en klass eller ett program som andra förväntas använda.

## 10. Övningar

1. Förklara med egna ord skillnaden mellan `balance` och `_balance`.
2. Varför skriver vi `p.magnitude` men `p.move(1, 0)`?
3. Vad händer när vi kör `Account("Joakim", -10)`?
4. Vad skrivs ut efter `konto = Account("A", 80)` följt av `konto.deposit(20)` och `print(konto.balance)`?
5. Vad är felet i en setter som innehåller `self.balance = value`?
6. Vad är skillnaden mellan `_token` och `__token`?

## 11. Facit

1. `balance` är det publika gränssnittet som kan kontrollera värden; `_balance` är det interna attributet som faktiskt lagrar talet.
2. `magnitude` är en property och läses utan parenteser; `move` är en vanlig metod som anropas med parenteser och argument.
3. `ValueError` utlöses eftersom settern inte accepterar negativt saldo.
4. `100.0`.
5. Den anropar sig själv upprepade gånger. Tilldelningen ska i stället ske till `self._balance`.
6. `_token` signalerar internt bruk enligt konvention. `__token` får ett namnändrat attribut via name mangling, men är inte strikt privat.

---

**Sammanfattning:** Inkapsling handlar om att göra en klass lätt och säker att använda enligt dess regler. `@property` ger ett attributliknande gränssnitt, en setter kan validera ändringar, och ett internt attribut som `_balance` lagrar själva värdet.
