# Lesson 08 – Functions in Python

> **Syfte:** Förstå varför vi använder funktioner, hur de definieras och anropas, skillnaden mellan parametrar och argument, standardvärden, `return` och variablers räckvidd (*scope*). Anteckningarna bygger på lärarens genomgång av **funktioner**. **Moduler behandlas inte i den här transkriberingen**, så de får en egen genomgång senare.

## Quick Reference – snabb uppslagning

| Du vill… | Kod | Kom ihåg |
|---|---|---|
| Skapa en funktion | `def greet():` | `def` definierar funktionen |
| Köra en funktion | `greet()` | Parenteserna gör att funktionen anropas |
| Ta emot ett värde | `def greet(name):` | `name` är en **parameter** |
| Skicka in ett värde | `greet("Joakim")` | `"Joakim"` är ett **argument** |
| Ge standardvärde | `def greet(name="John"):` | Anrop utan argument fungerar |
| Ange parameter med namn | `greet(name="Hans")` | Ett *keyword argument* |
| Skicka tillbaka ett resultat | `return result` | Värdet kan sparas och användas vidare |
| Spara returvärde | `answer = calculate(5)` | Variabeln får funktionens resultat |
| Skapa lokal variabel | `x = 10` inuti funktionen | Finns normalt bara där |
| Använda global variabel | Läs `x` definierad utanför | Undvik i regel att ändra den inuti funktioner |
| Ändra global variabel | `global x` | Möjligt, men sällan bästa lösningen |

## 1. Vad är en funktion – och varför behövs den?

En **funktion** är ett namngivet kodblock som utför en uppgift. Istället för att skriva samma kod om och om igen kan du definiera funktionen en gång och **anropa** den när den behövs.

- **Definiera** = tala om vad funktionen ska göra.
- **Anropa** = be Python köra funktionen.
- **Funktionskropp** = de indenterade raderna under `def`.

```python
def greet():
    print("Hej och välkommen!")

greet()
greet()
```

**Vad händer steg för steg?**

1. `def` säger att vi skapar en funktion.
2. `greet` är namnet vi väljer.
3. `()` visar att funktionen inte kräver några värden.
4. `:` inleder funktionskroppen.
5. Den indenterade `print`-raden körs **inte** när funktionen definieras.
6. Först när Python når `greet()` körs utskriften. Här anropas funktionen två gånger.

**Vanligt misstag:** Att skriva `greet` utan parenteser är inte samma sak som att anropa `greet()`.

## 2. Parametrar och argument – vad är skillnaden?

En funktion blir mer användbar om den kan arbeta med olika värden.

```python
def greet(name):
    print(f"Hej {name}, välkommen till kursen!")

greet("Joakim")
greet("Hans")
```

**Resultat:**

```text
Hej Joakim, välkommen till kursen!
Hej Hans, välkommen till kursen!
```

Här är `name` en **parameter**: namnet på den plats där funktionen ska ta emot ett värde. `"Joakim"` och `"Hans"` är **argument**: de faktiska värden vi skickar in vid anropet.

**Minnesregel:** *Parameter i definitionen – argument i anropet.*

Du behöver inte skapa variabeln `name` innan du anropar funktionen. Python kopplar argumentet till parametern när anropet görs.

## 3. Vad händer om ett argument saknas?

```python
def greet(name):
    print(f"Hej {name}!")

greet()  # TypeError: argumentet 'name' saknas
```

Funktionen kräver ett värde för `name`, men får inget. Därför blir det fel.

### Standardvärden (default parameters)

```python
def greet(first_name="John", last_name="Doe"):
    print(f"Hej {first_name} {last_name}, välkommen till kursen!")

greet()                  # Hej John Doe, välkommen till kursen!
greet("Hans")            # Hej Hans Doe, välkommen till kursen!
greet("Joakim", "Lind")  # Hej Joakim Lind, välkommen till kursen!
```

`=` i funktionsdefinitionen anger ett **standardvärde**. Det används bara om inget annat värde skickas in för just den parametern.

> Standardvärden gör ett argument valfritt. Utan standardvärde är argumentet normalt obligatoriskt.

## 4. Positionsargument och namngivna argument

Python kopplar vanligtvis argument till parametrar i samma **ordning** som de står.

```python
def greet(first_name="John", last_name="Doe"):
    print(f"Hej {first_name} {last_name}!")

greet("Lind", "Joakim")  # Hej Lind Joakim!
```

Python vet inte att du kanske tänkte på ett efternamn först. Det första argumentet hamnar i `first_name`.

Vill du välja en särskild parameter kan du skriva dess namn:

```python
greet(last_name="Lind")  # Hej John Lind!
greet(last_name="Lind", first_name="Joakim")  # Hej Joakim Lind!
```

Detta kallas **keyword arguments** eller *namngivna argument*. Då spelar ordningen mellan de namngivna argumenten ingen roll.

## 5. `print()` och `return` är inte samma sak

Det här är en av de viktigaste skillnaderna i hela lektionen.

```python
def circle_area(radius):
    print(3.142 * radius * radius)

result = circle_area(5)
print(result)
```

Funktionen **skriver ut** ungefär `78.55`, men variabeln `result` blir `None`. Varför? Därför att `print()` bara visar något på skärmen; den skickar inte tillbaka det beräknade värdet från funktionen.

Med `return` fungerar det annorlunda:

```python
def circle_area(radius):
    area = 3.142 * radius * radius
    return area

result = circle_area(5)
print(result)  # 78.55
```

**Steg för steg:**

1. `circle_area(5)` skickar argumentet `5` till parametern `radius`.
2. `area` beräknas till `3.142 * 5 * 5`.
3. `return area` skickar tillbaka resultatet till anropsplatsen.
4. `result = ...` sparar värdet i variabeln `result`.
5. `print(result)` visar värdet när **vi** väljer att skriva ut det.

**Minnesregel:** `print()` = **visa**. `return` = **lämna tillbaka så att värdet kan användas**.

> `return` avslutar också det aktuella funktionsanropet. Kod efter en utförd `return` i samma kodväg körs inte.

## 6. Använd resultatet från en funktion i en annan

Läraren visar hur vi först räknar ut arean av en cirkel och sedan använder arean för att räkna ut volymen av en cylinder.

Formler:

- Cirkelns area = π × radie × radie.
- Cylinderns volym = area × längd (eller höjd).

```python
def circle_area(radius):
    return 3.142 * radius * radius


def cylinder_volume(area, length):
    return area * length

radius = 2
length = 10

calculated_area = circle_area(radius)
calculated_volume = cylinder_volume(calculated_area, length)

print(calculated_area)    # 12.568
print(calculated_volume)  # 125.68
```

Här är `calculated_area` ett vanligt tal som funktionen lämnat tillbaka. Det talet blir sedan ett **argument** till `cylinder_volume()`.

Det går även att anropa den ena funktionen direkt inuti den andra:

```python
volume = cylinder_volume(circle_area(2), 10)
print(volume)  # 125.68
```

Python räknar först ut `circle_area(2)` och använder sedan resultatet som första argument till `cylinder_volume()`.

### Ta emot inmatning från användaren

```python
radius = int(input("Enter the radius: "))
length = int(input("How long is your cylinder? "))

volume = cylinder_volume(circle_area(radius), length)
print(volume)
```

`input()` ger alltid **text** (`str`). `int(...)` gör om texten till ett heltal, så länge användaren skriver ett giltigt heltal. För decimaler kan du istället använda `float(...)`.

> **Komplettering:** För mer exakt π kan du använda `import math` och `math.pi`. I genomgången används det avrundade värdet `3.142` för att fokusera på funktionerna.

## 7. Lokala och globala variabler – `scope`

**Scope** betyder ungefär *räckvidd*: var i programmet ett variabelnamn är tillgängligt.

### Global variabel – skapad utanför funktionen

```python
x = 5

def show_number():
    print("Inuti:", x)

show_number()         # Inuti: 5
print("Utanför:", x) # Utanför: 5
```

Funktionen kan **läsa** `x` som skapats utanför den.

### Lokal variabel – skapad inuti funktionen

```python
def show_number():
    x = 10
    print("Inuti:", x)

show_number()  # Inuti: 10
print(x)       # NameError om inget annat x finns utanför
```

Variabeln `x` som skapas inuti funktionen är **lokal**. Den finns normalt inte tillgänglig utanför funktionen.

### Samma namn – två olika variabler

```python
x = 5

def show_number():
    x = 10
    print("Inuti:", x)

show_number()          # Inuti: 10
print("Utanför:", x)  # Utanför: 5
```

Den lokala variabeln `x` **skuggar** den globala variabeln med samma namn inuti funktionen. Den ändrar inte automatiskt värdet på den globala variabeln.

### Vad gör `global`?

```python
x = 5

def change_number():
    global x
    x = 10

change_number()
print(x)  # 10
```

`global x` säger att tilldelningen `x = 10` ska gälla den globala variabeln, inte skapa en ny lokal `x`.

Det går att göra, men läraren påpekar att man **oftast inte vill ändra globala variabler från funktioner**. Det gör koden svårare att följa. Ofta är det tydligare att använda parametrar och `return`.

```python
def change_number(x):
    return x + 5

x = 5
x = change_number(x)
print(x)  # 10
```

Här är det synligt var värdet skickas in och var det nya värdet kommer tillbaka.

### Varför kan Jupyter visa ett oväntat resultat?

Läraren visar att en tidigare definierad variabel kan ligga kvar i minnet i en Jupyter Notebook. Om du har kört `x = 5` i en cell kan `x` fortfarande finnas även om du senare tar bort raden och kör en annan cell.

**Felsökning:** Starta om notebookens kernel och kör cellerna i rätt ordning när resultatet inte stämmer med koden du ser.

## 8. Vanliga misstag

| Misstag | Varför det blir fel | Lösning |
|---|---|---|
| Definiera en funktion utan att anropa den | `def` skapar funktionen men kör inte dess kropp | Skriv `greet()` |
| Glömma ett obligatoriskt argument | Parametern saknar värde | Skicka ett argument eller ge standardvärde |
| Förväxla parameter och argument | De används på olika ställen | Parameter i `def`, argument i anrop |
| Skicka namn i fel ordning | Positionsargument följer ordningen | Använd `first_name=...`, `last_name=...` |
| Använda `print()` när ett värde behövs senare | `print()` returnerar inte beräkningen | Använd `return` |
| Försöka läsa en lokal variabel utanför funktionen | Namnet finns bara lokalt | Returnera värdet |
| Stava parametern olika | Python känner inte igen namnet | Kontrollera stavning konsekvent |
| Få märkliga värden i Jupyter | Gamla cellkörningar kan finnas kvar i minnet | Starta om kernel och kör om |

## 9. Öva själv – med facit

### Övning 1: En enkel funktion

Skapa en funktion som skriver ut `Hej!` när du anropar den.

<details>
<summary>Visa lösning</summary>

```python
def say_hello():
    print("Hej!")

say_hello()
```

</details>

### Övning 2: Parameter och argument

Skapa `double(number)` som **returnerar** dubbla talet. Spara resultatet i en variabel och skriv ut det.

<details>
<summary>Visa lösning</summary>

```python
def double(number):
    return number * 2

answer = double(7)
print(answer)  # 14
```

`number` är parametern och `7` är argumentet.

</details>

### Övning 3: Standardvärde

Skapa `welcome(name="vän")` som skriver ut en hälsning. Testa med och utan argument.

<details>
<summary>Visa lösning</summary>

```python
def welcome(name="vän"):
    print(f"Välkommen, {name}!")

welcome()          # Välkommen, vän!
welcome("Joakim")  # Välkommen, Joakim!
```

</details>

### Övning 4: Vad skrivs ut?

```python
x = 3

def example():
    x = 8
    return x

print(example())
print(x)
```

<details>
<summary>Visa svar</summary>

Först `8`, sedan `3`. Den lokala variabeln `x` inne i funktionen är inte samma variabel som den globala `x`.

</details>

## 10. Sammanfattning

- Funktioner gör kod **återanvändbar** och mer organiserad.
- `def` **definierar** funktionen; `function_name()` **anropar** den.
- **Parametrar** finns i definitionen; **argument** skickas vid anropet.
- **Standardvärden** gör att vissa argument kan utelämnas.
- **Positionsargument** följer ordningen; **namngivna argument** anger vilken parameter som ska få värdet.
- `print()` **visar** något; `return` **lämnar tillbaka** ett värde som annan kod kan använda.
- **Lokala** variabler hör till funktionen; **globala** variabler är definierade utanför.
- Använd helst **parametrar och returvärden** istället för att ändra globala variabler.

---

*Underlag: lärarens videotranskribering om funktioner, vecka 2. Förtydliganden, exempel och övningar har lagts till för att göra innehållet lättare att repetera.*
