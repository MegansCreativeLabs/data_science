# Lektion 02 – NumPy och numeriska arrayer

**Kurs:** Vecka 4 – Deskriptiv statistik och bibliotek för dataanalys  
**Plats:** `data_science/data_analysis/02_numpy.md`  
**Utgångspunkt:** Lärarens video om NumPy och numeriska arrayer.

## Vad ska du kunna efter lektionen?

Du ska kunna förklara vad NumPy och en `ndarray` är, hur `dtype` och `shape` fungerar, hur man skapar och formar om arrayer, utför beräkningar utan egna loopar, använder broadcasting, väljer ut rader och kolumner, filtrerar med booleska masker och räknar ut grundläggande statistik.

> **Kom ihåg:** Du behöver inte memorera all syntax. Målet är att känna igen den, förstå vad den gör och kunna slå upp den när du behöver.

---

## 1. Vad är NumPy?

**NumPy** är ett Pythonbibliotek för numeriska beräkningar. Det används när vi vill arbeta effektivt med många tal, exempelvis mätvärden, provresultat eller stora tabeller.

Vi importerar biblioteket så här:

```python
import numpy as np
```

- `import` betyder att vi hämtar in ett bibliotek.
- `numpy` är bibliotekets namn.
- `as np` ger det ett kortare namn som vi använder i resten av koden.

Om NumPy inte är installerat i din aktiva Pythonmiljö:

```powershell
python -m pip install numpy
```

I VS Code eller Jupyter behöver du också välja rätt **Python interpreter/kernel** – alltså den Pythonmiljö som ska köra koden.

## 2. Pythonlista jämfört med NumPy-array

```python
python_lista = [1, 2, 3]
nummer = np.array([1, 2, 3])

print(type(python_lista))
print(type(nummer))
```

Resultat:

```text
<class 'list'>
<class 'numpy.ndarray'>
```

**`ndarray`** betyder *N-dimensional array*: en array som kan ha en eller flera dimensioner.

En Pythonlista kan innehålla olika typer av objekt. En vanlig NumPy-array har däremot **en gemensam datatyp (`dtype`)** för sina element. Detta gör många numeriska operationer effektiva.

### Samma plustecken – olika betydelse

```python
python_lista = [1, 2, 3]
nummer = np.array([1, 2, 3])

print(python_lista + [4, 5])
print(nummer + 10)
```

Resultat:

```text
[1, 2, 3, 4, 5]
[11 12 13]
```

- **Lista + lista:** sammanfogar listorna (*konkatenering*).
- **Array + tal:** adderar talet till **varje element**.

Det är alltså inte samma sak att skriva `lista + [4, 5]` som `array + 10`.

## 3. `dtype` – vilken datatyp har elementen?

```python
heltal = np.array([1, 2, 3])
decimaltal = np.array([1.0, 2, 3])

print(heltal.dtype)
print(decimaltal.dtype)
```

Du får en heltalstyp respektive en flyttalstyp, exempelvis `int64` och `float64`. Exakt heltalsbredd kan bero på plattform och NumPy-version.

**`int`** betyder heltal. **`float`** betyder decimaltal. När vi blandar dem i en numerisk array väljer NumPy normalt en gemensam typ som kan representera värdena, till exempel `float64`.

### Byta datatyp med `astype()`

```python
värden = np.array([1.8, 2.4, 3.9])
heltal = värden.astype(np.int64)

print(heltal)
print(heltal.dtype)
```

Resultat:

```text
[1 2 3]
int64
```

**Viktigt:** Omvandlingen till heltal **kapar decimaldelen** – den avrundar inte till närmaste heltal. `astype()` ger normalt en ny array och ändrar inte den gamla variabeln automatiskt.

## 4. Dimensioner, axlar och `shape`

Det här är en av lektionens viktigaste delar.

### En dimension (1D)

```python
v = np.array([10, 20, 30, 40])

print(v)
print(v.shape)
print(v.ndim)
```

Resultat:

```text
[10 20 30 40]
(4,)
1
```

- `shape` betyder **form**.
- `(4,)` betyder **fyra element längs en enda dimension**.
- `ndim` betyder antal dimensioner: här `1`.

**Observera:** `(4,)` betyder inte specifikt ”en rad” eller ”en kolumn” i en tvådimensionell tabell. Det är en **endimensionell** array. Kommatecknet är en del av hur Python skriver en tuple med ett enda värde.

### Två dimensioner (2D): en matris

```python
m = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

print(m)
print(m.shape)
print(m.ndim)
```

Resultat:

```text
[[1 2 3]
 [4 5 6]]
(2, 3)
2
```

Tänk på matrisen som en tabell:

| | Kolumn 0 | Kolumn 1 | Kolumn 2 |
|---|---:|---:|---:|
| **Rad 0** | 1 | 2 | 3 |
| **Rad 1** | 4 | 5 | 6 |

`shape == (2, 3)` betyder alltså **2 rader och 3 kolumner**.

En minnesregel för tvådimensionella arrayer är:

```text
shape = (antal_rader, antal_kolumner)
```

### Vad är en axel (`axis`)?

- `axis=0` är riktningen **ned genom raderna**. När du summerar längs `axis=0` får du ett resultat per kolumn.
- `axis=1` är riktningen **tvärs över kolumnerna**. När du summerar längs `axis=1` får du ett resultat per rad.

```python
print(m.sum(axis=0))  # [5 7 9]  – kolumnsummor
print(m.sum(axis=1))  # [ 6 15] – radsummor
```

## 5. `arange()` och `reshape()`

```python
värden = np.arange(12)
print(värden)
```

Resultat:

```text
[ 0  1  2  3  4  5  6  7  8  9 10 11]
```

`np.arange(12)` skapar 12 värden från **0 till 11**. Precis som med `range()` är slutgränsen exkluderad.

```python
matris = np.arange(12).reshape(3, 4)
print(matris)
```

Resultat:

```text
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]
```

`reshape(3, 4)` betyder **3 rader och 4 kolumner**.

**Varför fungerar det?** Därför att `3 * 4 = 12`, alltså exakt så många element som arrayen innehåller. `reshape()` ändrar formen, inte vilka värden som finns. Om antalet platser inte stämmer får du ett fel.

```python
# Detta fungerar INTE: 12 värden får inte plats i 2 * 5 = 10 platser
# np.arange(12).reshape(2, 5)
```

## 6. Skapa arrayer snabbt

```python
print(np.zeros((2, 3)))
print(np.ones((2, 3)))
print(np.linspace(0, 1, 5))
```

Resultat:

```text
[[0. 0. 0.]
 [0. 0. 0.]]

[[1. 1. 1.]
 [1. 1. 1.]]

[0.   0.25 0.5  0.75 1.  ]
```

| Funktion | Vad gör den? | Exempel |
|---|---|---|
| `np.zeros((2, 3))` | Skapar 2 × 3 nollor | Startvärden |
| `np.ones((2, 3))` | Skapar 2 × 3 ettor | Testdata |
| `np.arange(5)` | Skapar 0, 1, 2, 3, 4 | Heltalssekvens |
| `np.linspace(0, 1, 5)` | Skapar **5 jämnt fördelade värden** från 0 till 1, inklusive båda ändpunkterna | Skala eller diagramaxel |

**Skillnad:** `arange()` styr vanligtvis **stegstorlek**, medan `linspace()` styr **antalet värden**. I lärarens exempel med `linspace(0, 1, 5)` är det alltså fem värden, inte fem mellanrum.

## 7. Vektoriserade operationer

**Vektoriserad** betyder här att vi kan uttrycka en beräkning för en hel array utan att själva skriva en `for`-loop för varje element.

```python
x = np.array([10, 20, 30, 40])
y = np.array([1, 2, 3, 4])

print(x + y)
print(x * 2)
print(x ** 2)
```

Resultat:

```text
[11 22 33 44]
[20 40 60 80]
[ 100  400  900 1600]
```

Så fungerar `x + y`:

| Element i `x` | Element i `y` | Summa |
|---:|---:|---:|
| 10 | 1 | 11 |
| 20 | 2 | 22 |
| 30 | 3 | 33 |
| 40 | 4 | 44 |

**Viktigt syntaxfel att undvika:** I Python betyder `**` upphöjt till, medan `^` är en bitvis XOR-operation. Använd alltså `x ** 2` för att kvadrera talen – **inte** `x ^ 2`.

### Vanliga statistiska beräkningar

```python
print(x.mean())  # 25.0 – medelvärde
print(x.std())   # cirka 11.18 – standardavvikelse
print(x.min())   # 10 – minsta värdet
print(x.max())   # 40 – största värdet
```

`mean` betyder **medelvärde**, inte `min`. `std` är en förkortning av *standard deviation*, alltså **standardavvikelse**.

**Viktigt från statistiklektionen:** `x.std()` använder som standard populationsstandardavvikelse (`ddof=0`). Om du vill beräkna den vanliga stickprovsstandardavvikelsen med nämnaren `n - 1`, använd `x.std(ddof=1)`.

## 8. Broadcasting – när olika former ändå kan räknas ihop

Broadcasting betyder att NumPy kan kombinera arrayer med **olika, men kompatibla former** utan att du behöver kopiera ut alla värden själv.

### Exempel A: samma rad läggs till i varje rad

```python
m = np.array([
    [1, 2, 3],
    [4, 5, 6]
])
v = np.array([10, 20, 30])

print(m.shape)  # (2, 3)
print(v.shape)  # (3,)
print(m + v)
```

Resultat:

```text
[[11 22 33]
 [14 25 36]]
```

NumPy matchar de tre elementen i `v` mot **varje rad** i `m`:

```text
[1 2 3]   + [10 20 30] = [11 22 33]
[4 5 6]   + [10 20 30] = [14 25 36]
```

### Exempel B: ett tal läggs till överallt

```python
print(m + 100)
```

```text
[[101 102 103]
 [104 105 106]]
```

### Exempel C: en kolumn påverkar varje rad

```python
kolumn = np.array([[10], [100]])  # shape (2, 1)
print(m + kolumn)
```

```text
[[ 11  12  13]
 [104 105 106]]
```

Här läggs **10** till hela första raden och **100** till hela andra raden.

### Broadcasting-regeln, steg för steg

NumPy jämför dimensionerna **från höger till vänster**. Två dimensioner är kompatibla om de:

1. är **lika stora**, eller
2. minst en av dem är **1**.

Saknas en dimension på ena sidan behandlas den som `1` när formerna jämförs.

| Form A | Form B | Fungerar? | Varför? |
|---|---|---|---|
| `(2, 3)` | `(3,)` | Ja | Jämför `(2, 3)` med `(1, 3)` |
| `(2, 3)` | `(2, 1)` | Ja | Sista dimensionen 3 mot 1 |
| `(3, 1)` | `(2,)` | Ja | `(3, 1)` och `(1, 2)` ger `(3, 2)` |
| `(3, 2)` | `(3,)` | **Nej** | Sista dimensionen 2 mot 3 |
| `(3, 2)` | `(2, 3)` | **Nej** | 2 mot 3 och 3 mot 2 |

### Varför fick läraren inte det förväntade felet?

Läraren provade att kombinera en array med formen `(3, 1)` och en med formen `(2,)`. Det **ska fungera**, eftersom NumPy tolkar formerna som `(3, 1)` och `(1, 2)` och kan skapa resultatet `(3, 2)`. Det är alltså inte en förändring i NumPy som orsakade det oväntade resultatet, utan ett exempel på själva broadcasting-regeln.

```python
första = np.array([1, 2, 3]).reshape(3, 1)
andra = np.array([10, 20])

print(första + andra)
```

```text
[[11 21]
 [12 22]
 [13 23]]
```

Ett **riktigt fel** får vi däremot här:

```python
try:
    resultat = np.ones((3, 2)) + np.array([10, 20, 30])
except ValueError as fel:
    print("Former som inte passar ihop:", fel)
```

`try` försöker köra koden. `except ValueError` fångar felet så att programmet kan fortsätta.

## 9. Indexering – välj en rad eller en cell

```python
z = np.arange(12).reshape(3, 4)
print(z)
```

```text
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]
```

Kom ihåg att Python börjar räkna index från **0**.

```python
print(z[0, 1])  # 1 – rad 0, kolumn 1
print(z[1])     # [4 5 6 7] – hela rad 1
print(z[:, 0])  # [0 4 8] – hela kolumn 0
```

I `z[rad, kolumn]` kommer alltså **raden först**, sedan kolumnen.

### Vad betyder kolon (`:`)?

Ett ensamt `:` betyder **alla värden längs den dimensionen**.

- `z[1, :]` → alla kolumner i rad 1.
- `z[:, 0]` → alla rader i kolumn 0.
- `z[:, 1:3]` → alla rader, men bara kolumnerna med index 1 och 2.

```python
print(z[:, 1:3])
```

```text
[[ 1  2]
 [ 5  6]
 [ 9 10]]
```

Precis som med vanliga listor är slutindex **exkluderat**. `1:3` tar alltså index 1 och 2, inte 3.

## 10. Boolesk mask – filtrera med `True` och `False`

En **boolesk mask** är en array av sanningsvärden (`True` och `False`). Den visar vilka element som uppfyller ett villkor.

```python
z = np.arange(12).reshape(3, 4)
mask = z % 2 == 0

print(mask)
```

Resultat:

```text
[[ True False  True False]
 [ True False  True False]
 [ True False  True False]]
```

Så läser du villkoret `z % 2 == 0`:

- `% 2` räknar ut **resten vid division med 2**.
- `== 0` frågar om resten är noll.
- Tal med resten noll är **jämna**.

Nu filtrerar vi:

```python
print(z[mask])
```

```text
[ 0  2  4  6  8 10]
```

**`True` betyder behåll**, och **`False` betyder välj inte ut**. Originalarrayen ändras inte av den här filtreringen. När vi väljer element med en mask får vi här en endimensionell array med de valda värdena.

Det här är särskilt viktigt inför **Pandas**, där vi använder samma grundidé för att välja rader som uppfyller villkor.

## 11. Z-poäng – använd NumPy tillsammans med statistik

I förra lektionen lärde vi oss att en **z-poäng** beskriver hur många standardavvikelser ett värde ligger över eller under medelvärdet.

Formeln är:

\[
z = \frac{x - \text{medelvärde}}{\text{standardavvikelse}}
\]

Lärarens exempel:

```python
scores = np.array([50, 60, 70, 80, 90], dtype=np.float64)

medel = scores.mean()
standardavvikelse = scores.std()
z = (scores - medel) / standardavvikelse

print(medel)
print(round(standardavvikelse, 2))
print(np.round(z, 2))
```

Resultat:

```text
70.0
14.14
[-1.41 -0.71  0.    0.71  1.41]
```

Så tolkar vi värdena:

- `50` ligger cirka **1,41 standardavvikelser under** medelvärdet.
- `70` är **precis på** medelvärdet och får z-poängen `0`.
- `90` ligger cirka **1,41 standardavvikelser över** medelvärdet.

Vi använder alltså NumPys vektoriserade operationer för att räkna ut **alla z-poäng på en gång**.

**Observera:** `scores.std()` använder här `ddof=0`, alltså populationsstandardavvikelse. Det är samma beräkning som lärarens exempel med cirka `14,14`.

## 12. Vanliga misstag

| Misstag | Varför blir det fel? | Rätt tanke |
|---|---|---|
| `x ^ 2` | `^` betyder bitvis XOR | Använd `x ** 2` |
| Tolka `(4,)` som en tvådimensionell rad | Arrayen har bara en dimension | Använd `ndim` för att se antalet dimensioner |
| Skriva `reshape(2, 5)` för 12 element | 2 × 5 är bara 10 platser | Antalet platser måste stämma |
| Blanda ihop `mean()` och `min()` | `mean` = medelvärde, `min` = minimum | Läs metodnamnen noggrant |
| Tro att `astype(int)` avrundar | Decimaldelen kapas | Avrunda separat om det är vad du vill |
| Tro att `z[:, 1:3]` tar kolumn 3 | Slutindex ingår inte | Väljer kolumn 1 och 2 |
| Tro att olika `shape` alltid ger fel | Broadcasting kan göra dem kompatibla | Jämför dimensionerna från höger |
| Tro att `std()` alltid använder `n-1` | Standard är `ddof=0` | Använd `ddof=1` för stickprovsvarianten |

## 13. Quick Reference

| Kod | Betydelse |
|---|---|
| `import numpy as np` | Importera NumPy |
| `np.array([1, 2, 3])` | Skapa en NumPy-array |
| `a.dtype` | Elementens datatyp |
| `a.astype(float)` | Omvandla till flyttal |
| `a.shape` | Arrayens form |
| `a.ndim` | Antal dimensioner |
| `np.arange(12)` | Skapa talen 0–11 |
| `a.reshape(3, 4)` | Forma om till 3 × 4 |
| `np.zeros((2, 3))` | Skapa nollmatris |
| `np.ones((2, 3))` | Skapa ettmatris |
| `np.linspace(0, 1, 5)` | Fem jämnt fördelade värden |
| `a + 10` | Addera 10 till varje element |
| `a ** 2` | Kvadrera varje element |
| `a.mean()` | Medelvärde |
| `a.std()` | Populationsstandardavvikelse som standard |
| `a.std(ddof=1)` | Stickprovsstandardavvikelse |
| `a.min()`, `a.max()` | Minsta och största värdet |
| `a[0, 1]` | Rad 0, kolumn 1 |
| `a[:, 0]` | Hela första kolumnen |
| `a[1, :]` | Hela andra raden |
| `a[:, 1:3]` | Kolumn 1 och 2 |
| `a[a > 10]` | Välj element större än 10 |
| `np.round(a, 2)` | Avrunda varje element till två decimaler |

---

## 14. Övningar

Försök gärna läsa och förstå koden innan du kör den. **Facit finns direkt efter övningarna**, så att du kan kontrollera utan att fastna.

### Övning 1 – Lista eller array?

Vad skriver koden ut?

```python
lista = [2, 4, 6]
array = np.array([2, 4, 6])

print(lista + [8])
print(array + 8)
```

### Övning 2 – Form

Skapa en array med talen 0–11 och forma om den till **4 rader och 3 kolumner**. Vad blir `shape`?

### Övning 3 – Vektoriserad beräkning

Du har priserna `[100, 200, 300]`. Skapa en NumPy-array och beräkna alla priser efter en höjning med 10 %.

### Övning 4 – Indexering

Utgå från:

```python
m = np.array([
    [10, 20, 30],
    [40, 50, 60]
])
```

Hur väljer du ut talet `50`, hela andra raden och hela sista kolumnen?

### Övning 5 – Boolesk filtrering

Skapa arrayen `[3, 8, 11, 14, 20]` och välj ut alla värden som är **större än 10**.

### Övning 6 – Broadcasting

Vilket resultat får du av följande kod, och varför fungerar den?

```python
m = np.array([[1, 2, 3], [4, 5, 6]])
v = np.array([10, 20, 30])
print(m + v)
```

### Övning 7 – Statistik och z-poäng

Beräkna medelvärde, standardavvikelse och z-poäng för `[10, 20, 30]` med NumPys standardinställning för `std()`.

---

## 15. Facit

### Facit 1

```text
[2, 4, 6, 8]
[10 12 14]
```

Listan får ett nytt element i slutet. Arrayen får 8 adderat till varje element.

### Facit 2

```python
a = np.arange(12).reshape(4, 3)
print(a)
print(a.shape)
```

```text
[[ 0  1  2]
 [ 3  4  5]
 [ 6  7  8]
 [ 9 10 11]]
(4, 3)
```

### Facit 3

```python
priser = np.array([100, 200, 300])
nya_priser = priser * 1.10
print(nya_priser)
```

```text
[110. 220. 330.]
```

Att multiplicera med `1.10` betyder **100 % av ursprungspriset + 10 % höjning**.

### Facit 4

```python
print(m[1, 1])  # 50
print(m[1, :])  # [40 50 60]
print(m[:, 2])  # [30 60]
```

### Facit 5

```python
a = np.array([3, 8, 11, 14, 20])
print(a[a > 10])
```

```text
[11 14 20]
```

### Facit 6

```text
[[11 22 33]
 [14 25 36]]
```

Arrayen `v` med formen `(3,)` läggs till i **varje rad** av `m` med formen `(2, 3)`. Former är kompatibla eftersom den sista dimensionen är 3 i båda.

### Facit 7

```python
a = np.array([10, 20, 30], dtype=float)
medel = a.mean()
std = a.std()
z = (a - medel) / std

print(medel)
print(round(std, 2))
print(np.round(z, 2))
```

```text
20.0
8.16
[-1.22  0.    1.22]
```

---

## 16. Kopplingen till Pandas – nästa lektion

NumPy ger oss grunderna för att räkna med numeriska arrayer. I **Pandas** arbetar vi oftare med tabeller som har **namngivna kolumner och index**. NumPys idéer – datatyper, elementvisa operationer och booleska filter – kommer därför tillbaka.

En förenklad jämförelse:

| NumPy | Pandas |
|---|---|
| Array med numeriska värden | Tabell (`DataFrame`) med namngivna kolumner |
| `a[a > 10]` | `df[df["price"] > 10]` |
| `a.mean()` | `df["price"].mean()` |

**Förtydligande:** Pandas är byggt ovanpå NumPy och använder NumPy i många situationer, men inte varje Pandaskolumn är bokstavligen en NumPy-array. Pandas har även egna datatyper och andra typer av underliggande arrayer.

**Det viktigaste att ta med dig:** `shape` berättar hur data är organiserad, `dtype` berättar vilken typ elementen har, och vektoriserade operationer och broadcasting låter dig beräkna mycket med lite kod.
