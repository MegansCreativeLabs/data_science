# Lektion 18 – Polymorfism och komposition (preliminär komplettering)

> **KOMPLETTERANDE LEKTION – EJ VERIFIERAD MOT KURSMATERIALET**  
> Vi saknar transkribering och har **inte bekräftat att lärarens lektion 18 handlar om polymorfism eller komposition**. Ämnena är valda som användbara fortsättningar på OOP, inte som en återgivning av lärarens genomgång.
>
> **Status:** Preliminär egen komplettering. **Källa:** Allmän Pythonkunskap.

## Målet
Förstå hur olika objekt kan användas på liknande sätt och hur klasser kan samarbeta utan arv.

## Snabbguide

| Begrepp | Enkel förklaring |
|---|---|
| Polymorfism | Samma metodanrop kan fungera på olika slags objekt |
| Metod | Funktion som hör till en klass |
| Komposition | Ett objekt innehåller eller använder ett annat objekt |
| Duck typing | Python tittar ofta på vad ett objekt **kan göra**, inte bara dess typ |

## 1. Polymorfism – samma anrop, olika resultat

```python
class Hund:
    def ljud(self):
        return "Voff!"

class Katt:
    def ljud(self):
        return "Mjau!"

for djur in [Hund(), Katt()]:
    print(djur.ljud())
```

**Vad händer?** Loopen går igenom två olika objekt. Båda har metoden `ljud()`. Python använder rätt metod för respektive objekt. Resultatet blir `Voff!` och `Mjau!`.

Det viktiga är att koden som anropar `ljud()` inte behöver skriva en separat `if`-sats för varje djurtyp.

## 2. Duck typing

I Python behöver två klasser inte ärva från samma basklass för att kunna användas på liknande sätt.

```python
class Epost:
    def skicka(self):
        print("Skickar e-post")

class Sms:
    def skicka(self):
        print("Skickar sms")

def skicka_meddelande(tjanst):
    tjanst.skicka()

skicka_meddelande(Epost())
skicka_meddelande(Sms())
```

Funktionen fungerar för båda objekten eftersom båda erbjuder metoden `skicka()`.

**Varning:** Om vi skickar in ett objekt utan metoden `skicka()` får vi ett `AttributeError`.

## 3. Komposition – ett objekt har ett annat objekt

```python
class Motor:
    def starta(self):
        return "Motorn startar"

class Bil:
    def __init__(self, motor):
        self.motor = motor

    def starta(self):
        return self.motor.starta()

motor = Motor()
bil = Bil(motor)
print(bil.starta())
```

`Bil` **är inte** en `Motor`, men en bil **har** en motor. Attributet `self.motor` innehåller ett annat objekt. När vi anropar `bil.starta()` skickar bilen anropet vidare till motorn.

## 4. Arv eller komposition?

- **Arv:** `Hund` **är ett** `Djur`.
- **Komposition:** `Bil` **har en** `Motor`.

Det här är en tumregel, inte en absolut lag. Välj den modell som bäst beskriver problemet och gör koden lätt att förstå.

## 5. Exempel från supportärenden

```python
class EpostNotifiering:
    def skicka(self, text):
        print(f"E-post: {text}")

class KonsolNotifiering:
    def skicka(self, text):
        print(f"Konsol: {text}")

class SupportArende:
    def __init__(self, titel, notifiering):
        self.titel = titel
        self.notifiering = notifiering

    def markera_lost(self):
        self.notifiering.skicka(f"Löst: {self.titel}")

arende = SupportArende("Kan inte logga in", KonsolNotifiering())
arende.markera_lost()
```

`SupportArende` behöver inte veta exakt vilken sorts notifiering som används. Den behöver bara ett objekt med metoden `skicka(text)`.

## 6. Vanliga misstag

- Att tro att polymorfism kräver arv. I Python kan samma metodnamn räcka.
- Att förväxla **är en** med **har en**.
- Att glömma att `self.motor` är ett objekt, inte en textsträng.

## 7. Övningar

1. Skapa klasserna `Cirkel` och `Rektangel` som båda har metoden `area()`.
2. Lägg objekten i en lista och skriv ut deras areor i en loop.
3. Skapa en klass `Dator` som får ett `Skarm`-objekt i sin konstruktor.

### Facit

```python
import math

class Cirkel:
    def __init__(self, radie):
        self.radie = radie

    def area(self):
        return math.pi * self.radie ** 2

class Rektangel:
    def __init__(self, bredd, hojd):
        self.bredd = bredd
        self.hojd = hojd

    def area(self):
        return self.bredd * self.hojd

for form in [Cirkel(2), Rektangel(3, 4)]:
    print(round(form.area(), 2))

class Skarm:
    def __init__(self, tum):
        self.tum = tum

class Dator:
    def __init__(self, skarm):
        self.skarm = skarm

dator = Dator(Skarm(27))
print(dator.skarm.tum)  # 27
```

---
**Kursplansstatus:** Ej verifierad. Ämnet och lektionens nummer är preliminära tills lärarens material finns tillgängligt.
