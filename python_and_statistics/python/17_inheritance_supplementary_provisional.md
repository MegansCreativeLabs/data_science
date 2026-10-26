# Lektion 17 – Arv i Python (preliminär komplettering)

> **KOMPLETTERANDE LEKTION – EJ VERIFIERAD MOT KURSMATERIALET**  
>
> **Status:** Preliminär egen komplettering. **Källa:** Allmän Pythonkunskap.

## Målet
Förstå vad **arv**, **basklass**, **underklass**, `super()` och **överskuggning av metoder** betyder.

## Snabbguide

| Begrepp | Betydelse |
|---|---|
| Basklass | Klass som en annan klass utgår från |
| Underklass | Klass som ärver från en basklass |
| `class Hund(Djur):` | `Hund` ärver från `Djur` |
| `super().__init__(...)` | Anropar basklassens konstruktor |
| Överskuggning | Underklassen definierar om en ärvd metod |
| `isinstance(hund, Djur)` | Kan bli `True` även om hunden är skapad från `Hund` |

## 1. Varför finns arv?

Om flera klasser har gemensamma egenskaper kan vi lägga det gemensamma i en **basklass**. Underklasserna får tillgång till detta utan att vi behöver kopiera all kod.

```python
class Djur:
    def __init__(self, namn):
        self.namn = namn

    def presentera(self):
        return f"Jag heter {self.namn}."

class Hund(Djur):
    pass

hund = Hund("Bella")
print(hund.presentera())  # Jag heter Bella.
```

**Rad för rad:** `class Djur:` skapar basklassen. `__init__` sparar namnet på objektet. `class Hund(Djur):` anger att `Hund` ärver från `Djur`. `pass` betyder att vi inte lägger till något nytt än. När vi skapar `Hund("Bella")` används den ärvda `__init__`-metoden.

## 2. Underklassen kan lägga till egna saker

```python
class Djur:
    def __init__(self, namn):
        self.namn = namn

class Hund(Djur):
    def __init__(self, namn, ras):
        super().__init__(namn)
        self.ras = ras

hund = Hund("Bella", "Labrador")
print(hund.namn)  # Bella
print(hund.ras)   # Labrador
```

`super().__init__(namn)` låter basklassen ta hand om `namn`. Sedan sparar underklassen sitt eget attribut `ras`. Utan anropet till `super()` skulle `self.namn` inte skapas automatiskt i det här exemplet.

## 3. Överskugga en metod

```python
class Djur:
    def ljud(self):
        return "Något ljud"

class Hund(Djur):
    def ljud(self):
        return "Voff!"

print(Hund().ljud())  # Voff!
```

Båda klasserna har en metod med samma namn. När objektet är en `Hund` används `Hund`-versionen.

## 4. `isinstance` och `type`

```python
hund = Hund()
print(type(hund) is Hund)       # True
print(isinstance(hund, Hund))  # True
print(isinstance(hund, Djur))  # True
```

`type(hund) is Hund` frågar efter exakt typ. `isinstance` tar även hänsyn till arv.

## 5. När är arv inte lämpligt?

Arv passar när något verkligen **är en sorts** något annat: `Hund` är ett `Djur`. Om något i stället **har** något annat, passar ofta **komposition** bättre. Exempelvis har en `Bil` en `Motor`; bilen är inte en sorts motor.

## 6. Vanliga misstag

- Att glömma `self` i instansmetoder.
- Att glömma `super().__init__(...)` när underklassen definierar en egen konstruktor och behöver basklassens initiering.
- Att använda arv enbart för att slippa skriva några rader kod, trots att klasserna inte har en naturlig relation.

## 7. Övningar

1. Skapa en basklass `Fordon` med attributet `marke`.
2. Skapa underklassen `Cykel` som ärver från `Fordon`.
3. Lägg till attributet `vaxlar` i `Cykel` med hjälp av `super()`.
4. Kontrollera med `isinstance` om en cykel också räknas som ett fordon.

### Facit

```python
class Fordon:
    def __init__(self, marke):
        self.marke = marke

class Cykel(Fordon):
    def __init__(self, marke, vaxlar):
        super().__init__(marke)
        self.vaxlar = vaxlar

cykel = Cykel("Trek", 7)
print(cykel.marke)
print(cykel.vaxlar)
print(isinstance(cykel, Fordon))  # True
```

---
**Kursplansstatus:** Ej verifierad. Ämnet och lektionens nummer är preliminära tills lärarens material finns tillgängligt.
