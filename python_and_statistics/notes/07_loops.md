# Lesson 07 – Loops in Python

> **Syfte:** Förstå hur Python upprepar kod, går igenom listor och avslutar loopar. Anteckningarna bygger på lärarens genomgång och kompletteras med tydligare förklaringar.

## Quick Reference – snabb uppslagning

| Du vill… | Kod | Kom ihåg |
|---|---|---|
| Upprepa 5 gånger | `for i in range(5):` | Ger 0, 1, 2, 3, 4 |
| Börja på 3, sluta före 10 | `range(3, 10)` | 10 ingår inte |
| Hoppa med 4 | `range(0, 20, 4)` | 0, 4, 8, 12, 16 |
| Räkna baklänges | `range(5, -1, -1)` | 5, 4, 3, 2, 1, 0 |
| Gå igenom lista | `for name in names:` | `name` blir ett element i taget |
| Gå igenom index | `for i in range(len(names)):` | `i` blir 0, 1, 2… |
| Avsluta hela loopen | `break` | Inga fler varv |
| Hoppa över ett varv | `continue` | Fortsätt med nästa varv |
| Upprepa medan villkor är sant | `while count < 5:` | Se till att villkoret kan bli falskt |
| Öka en räknare | `count += 1` | Samma som `count = count + 1` |
| Kontrollera jämnt tal | `number % 2 == 0` | Resten vid division med 2 är noll |

## 1. Vad är en loop?

En **loop** är kod som upprepas. Ett enda kodblock kan köras flera gånger, exempelvis för att skriva ut tal eller behandla varje namn i en lista.

- **Iteration** = ett varv i loopen.
- **Loopvariabel** = variabeln som får ett nytt värde för varje varv.
- **Indentering** = fyra mellanslag som visar vilken kod som hör till loopen.

```python
for i in range(3):
    print(i)

# 0
# 1
# 2
```

**Steg för steg:** `range(3)` ger talen 0, 1 och 2 i tur och ordning. `i` får först värdet 0, sedan 1 och sedan 2. `print(i)` körs en gång för varje värde. När talen är slut avslutas loopen.

> **Viktigt:** `range()` skapar ett *range-objekt*, inte en färdig lista. Men när vi loopar över det får vi talen ett i taget, precis som i lärarens exempel.

## 2. `range()` – start, stopp och steg

```python
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4 (på varsin rad)
```

`range(5)` börjar på **0** och slutar **före 5**. Det blir fem varv.

```python
for i in range(3, 10):
    print(i)  # 3, 4, 5, 6, 7, 8, 9
```

`range(start, stop)` inkluderar start men inte stopp.

```python
for i in range(0, 20, 4):
    print(i)  # 0, 4, 8, 12, 16
```

`range(start, stop, step)` använder det tredje talet som **steglängd**. Här ökar `i` med 4 varje varv.

### Räkna baklänges

```python
for i in range(5, -1, -1):
    print(i)  # 5, 4, 3, 2, 1, 0
```

Här betyder första `5` att vi börjar på 5. `-1` i mitten är stoppgränsen (som inte ingår). Sista `-1` betyder att vi minskar med 1 per varv. Därför kommer **0 med**.

## 3. Loopvariabeln – vad betyder `i`?

```python
for i in range(3):
    print("Varv:", i)
```

`i` är bara ett variabelnamn. Det används ofta som förkortning för *index*, men du kan välja ett annat namn. Python ger variabeln ett nytt värde för varje varv.

## 4. Loopa över en lista – två sätt

```python
names = ["Joakim", "Hans", "Tommy"]

for name in names:
    print(name)
```

Här blir `name` först `"Joakim"`, sedan `"Hans"` och slutligen `"Tommy"`. **Detta är oftast enklast när du bara behöver värdena.**

```python
for i in range(len(names)):
    print(i, names[i])
```

Förklaring:
1. `len(names)` är 3, eftersom listan innehåller tre namn.
2. `range(3)` ger indexen 0, 1 och 2.
3. `names[i]` hämtar namnet på aktuell indexplats.

| Varv | `i` | `names[i]` |
|---|---:|---|
| 1 | 0 | Joakim |
| 2 | 1 | Hans |
| 3 | 2 | Tommy |

**Skillnaden:** `for name in names` ger själva namnet. `for i in range(len(names))` ger ett **index** som du sedan använder för att hämta namnet.

## 5. Indentering – vad hör till loopen?

```python
for i in range(3):
    print("Inuti loopen:", i)

print("Efter loopen")
```

Den indenterade raden körs tre gånger. Den sista raden är inte indenterad och körs **bara en gång**, efter loopen. Fyra mellanslag är Pythons vanliga stil enligt PEP 8.

## 6. `break` – stoppa hela loopen

```python
for i in range(10):
    if i == 5:
        break
    print(i)

# 0, 1, 2, 3, 4
```

När `i` blir 5 är villkoret sant. `break` avslutar **hela** loopen direkt. Därför skrivs inte 5 eller senare tal ut.

## 7. `continue` – hoppa till nästa varv

```python
for i in range(10):
    if i < 5:
        continue
    print(i)

# 5, 6, 7, 8, 9
```

För talen 0–4 körs `continue`. Då hoppas resten av **det varvet** över, men loopen fortsätter med nästa tal.

| | `break` | `continue` |
|---|---|---|
| Vad händer? | Hela loopen avslutas | Bara aktuellt varv hoppas över |
| Kommer nästa tal att behandlas? | Nej | Ja |

## 8. Lärarens övning – jämna tal och en räknare

**Uppgift:** Gå igenom talen 0–14, skriv ut alla jämna tal och räkna hur många det finns. Lärarens lösning använder `range(15)`, så **15 ingår inte**.

### Lösning A – kontrollera jämna tal

```python
counter = 0

for i in range(15):
    if i % 2 == 0:
        print(i)
        counter += 1

print("Antal jämna tal:", counter)
```

**Resultat:** 0, 2, 4, 6, 8, 10, 12 och 14 skrivs ut, följt av `Antal jämna tal: 8`.

- `i % 2` ger resten när `i` delas med 2.
- Resten 0 betyder att talet är jämnt.
- `counter += 1` ökar räknaren med ett för varje jämnt tal.
- `counter = 0` ligger **före** loopen så att den inte nollställs varje varv.
- Sista `print()` ligger **utanför** loopen så att slutsumman bara skrivs ut en gång.

### Lösning B – hoppa över ojämna tal

```python
counter = 0

for i in range(15):
    if i % 2 == 1:
        continue

    print(i)
    counter += 1

print("Antal jämna tal:", counter)
```

Detta ger **samma resultat**. Skillnaden är att `continue` hoppar över alla ojämna tal. Då behöver utskrift och räknare inte ligga inuti en extra `if`-nivå.

## 9. `while` – upprepa så länge något är sant

En `for`-loop är praktisk när du vill gå igenom en bestämd serie värden. En `while`-loop upprepar i stället koden **så länge villkoret är sant**.

```python
count = 0

while count < 5:
    print(count)
    count += 1

# 0, 1, 2, 3, 4
```

**Steg för steg:**
1. `count` börjar på 0.
2. Python kontrollerar `count < 5`.
3. Om det är sant skrivs `count` ut.
4. `count += 1` ökar värdet.
5. Python går tillbaka till villkoret.
6. När `count` blir 5 är villkoret falskt och loopen slutar.

Om du glömmer att ändra `count` kan villkoret förbli sant och loopen fortsätta utan att ta slut. Därför är raden `count += 1` avgörande i just det här exemplet.

## 10. `for` eller `while`?

| Välj | När? | Exempel |
|---|---|---|
| `for` | Du vill gå igenom värden eller en lista | Alla namn i `names` |
| `for` + `range()` | Du vill upprepa ett bestämt antal gånger | Fem varv |
| `while` | Du vill fortsätta medan ett villkor gäller | Så länge `count < 5` |

## 11. Vanliga nybörjarmisstag

**Misstag 1: Tro att stoppvärdet ingår.** `range(5)` ger 0–4, inte 0–5.

**Misstag 2: Blanda ihop `break` och `continue`.** `break` stoppar hela loopen; `continue` hoppar över ett varv.

**Misstag 3: Nollställa räknaren inuti loopen.** Om `counter = 0` ligger inne i loopen börjar den om varje gång.

**Misstag 4: Skriva slutsumman inuti loopen.** Då skrivs en mellansumma ut vid varje varv.

**Misstag 5: Glömma indenteringen.** Koden som ska upprepas måste ligga indenterad under `for` eller `while`.

**Misstag 6: Glömma uppdatera villkoret i `while`.** Kontrollera att loopen kan avslutas.

## 12. Miniövningar

**Övning 1:** Vad skrivs ut?

```python
for i in range(2, 8, 2):
    print(i)
```

**Övning 2:** Vad är skillnaden mellan de här två exemplen?

```python
for i in range(5):
    if i == 2:
        break
    print(i)
```

```python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

**Övning 3:** Skriv en `while`-loop som skriver ut 1, 2 och 3 och sedan avslutas.

<details>
<summary>Facit – öppna när du vill kontrollera</summary>

**1:** 2, 4, 6.

**2:** Med `break` skrivs 0 och 1 ut. Med `continue` skrivs 0, 1, 3 och 4 ut.

**3:**

```python
count = 1
while count <= 3:
    print(count)
    count += 1
```

</details>

---

**Sammanfattning:** `for` går igenom värden, `while` fortsätter medan ett villkor är sant, `break` avbryter hela loopen och `continue` hoppar till nästa varv. `range()` inkluderar start men inte stopp, och en räknare som ska samla ett resultat måste initieras före loopen.
