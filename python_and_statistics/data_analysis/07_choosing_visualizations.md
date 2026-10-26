# 07 – Välj rätt figur

> **Vecka 5 · Lektion 5.2 · Visualisering och explorativ dataanalys (EDA)**  
> Baserad på lärarens videogenomgång. Exemplen är pedagogiskt bearbetade och vissa tekniska detaljer har förtydligats.

## Målet med lektionen

Att välja en visualisering utifrån **frågan du vill besvara** och **vilka datatyper du har**, inte utifrån vilket diagram som ser snyggast ut.

Efter lektionen ska du kunna:

- skilja på numeriska och kategoriska variabler
- välja histogram, låddiagram, spridningsdiagram, stapeldiagram eller korstabell
- skapa och tolka diagram med Matplotlib och Pandas
- använda `groupby()`, `pd.cut()` och `pd.crosstab()`
- förstå varför olika skalor kan göra en jämförelse missvisande
- normalisera värden till intervallet 0–1
- undvika vanliga fel vid filtrering, trendlinjer och datavisualisering

---

## 1. Börja med frågan – inte med diagrammet

Lärarens genomgång utgår från ett dataset med elever, studietimmar, provpoäng och klass.

| Fråga | Datatyper | Lämplig presentation |
|---|---|---|
| Hur är provpoängen fördelade? | En numerisk | Histogram, gärna tillsammans med låddiagram |
| Finns ett samband mellan studietimmar och provpoäng? | Två numeriska | Spridningsdiagram (`scatter`) |
| Vilken klass har högst medelpoäng? | En kategorisk + en numerisk | `groupby()` och stapeldiagram |
| Hur skiljer sig spridningen mellan klasser? | En kategorisk + en numerisk | Låddiagram per klass |
| Hur stor andel i varje klass ligger i olika poängintervall? | Två kategoriska | Korstabell med andelar |

**Numerisk variabel:** ett mätbart tal, till exempel 7 studietimmar eller 82 provpoäng.  
**Kategorisk variabel:** en grupp eller etikett, till exempel klass A, B eller C.

> **Kom ihåg:** Att ett värde är skrivet som en siffra betyder inte alltid att det är numeriskt i statistisk mening. Ett elev-ID är till exempel en identifierare, inte en mätning.

## 2. Förbered data

Läraren läser in en tidigare rensad CSV-fil. Här använder vi samma tänkta kolumnnamn som i genomgången:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("resultat_stadad.csv")

print(df.head())
print(df.info())
```

- `pd` är ett kortnamn för Pandas.
- `np` är ett kortnamn för NumPy.
- `plt` är ett kortnamn för Matplotlibs `pyplot`.
- `df` är en variabel som innehåller en **DataFrame** – en tabell med rader och kolumner.
- `.head()` visar de första fem raderna.
- `.info()` visar bland annat kolumnernas datatyper och antal icke-saknade värden.

Vi förväntar oss kolumner som `student`, `studietimmar`, `provpoäng` och `klass`. **Anpassa namnen om din CSV-fil använder andra stavningar.**

### Saknar du CSV-filen? Kör detta exempel först

Alla kodavsnitt nedan kan testas med detta lilla dataset:

```python
df = pd.DataFrame({
    "student": ["Ada", "Bo", "Cleo", "Dani", "Eli", "Fatima", "Gus", "Hana"],
    "klass": ["A", "A", "A", "A", "B", "B", "B", "B"],
    "studietimmar": [2, 4, 5, 7, 1, 3, 6, 8],
    "provpoäng": [55, 67, 75, 90, 50, 62, 78, 95],
})
```

**Viktigt:** Detta är *påhittade övningsdata*, inte värden från lärarens CSV-fil. Kör importerna ovan och skapa sedan `df` med koden här om du vill prova utan filer.

---

## 3. En numerisk variabel → histogram

**Fråga:** Hur fördelar sig elevernas provpoäng?

Ett histogram delar upp en numerisk skala i intervall, så kallade **bins**, och räknar hur många observationer som ligger i varje intervall.

```python
poang = df["provpoäng"].dropna()

fig, ax = plt.subplots(figsize=(10, 6))
ax.hist(poang, bins=10)
ax.set_title("Fördelning av provpoäng")
ax.set_xlabel("Provpoäng")
ax.set_ylabel("Antal elever")
ax.grid(True, axis="y")
plt.tight_layout()
plt.show()
```

### Förklara syntaxen

| Kod | Betydelse |
|---|---|
| `df["provpoäng"]` | Hämtar kolumnen provpoäng som en Pandas Series |
| `.dropna()` | Tar bort saknade värden **ur just den serien** |
| `fig, ax = plt.subplots()` | Skapar en figur och en axel att rita på |
| `figsize=(10, 6)` | Bredd 10 tum, höjd 6 tum |
| `ax.hist(...)` | Ritar histogram |
| `bins=10` | Delar upp värdeområdet i 10 intervall |
| `axis="y"` | Visar rutnät längs y-skalan |
| `plt.show()` | Visar diagrammet |

**Tolkning:** Titta efter toppar, snedfördelning, spridning och eventuella ovanliga värden. Antalet bins påverkar hur diagrammet ser ut; prova till exempel 5, 10 och 20 och jämför.

## 4. Histogram + låddiagram → både form och spridning

Läraren placerar ett histogram ovanför ett **horisontellt låddiagram** för att kunna jämföra samma numeriska skala.

```python
poang = df["provpoäng"].dropna()

fig, (ax_hist, ax_box) = plt.subplots(
    2, 1,
    figsize=(10, 7),
    sharex=True,
    gridspec_kw={"height_ratios": [4, 1]},
)

ax_hist.hist(poang, bins=10)
ax_hist.set_title("Provpoäng: histogram och låddiagram")
ax_hist.set_ylabel("Antal")
ax_hist.grid(True, axis="y")

ax_box.boxplot(poang, orientation="horizontal")
ax_box.set_xlabel("Provpoäng")
ax_box.set_yticks([])
ax_box.grid(True, axis="x")

plt.tight_layout()
plt.show()
```

### Vad betyder de nya delarna?

- `2, 1`: två **rader**, en **kolumn**.
- `(ax_hist, ax_box)`: packar upp de två axelobjekten i två variabler.
- `sharex=True`: diagrammen delar samma x-skala.
- `orientation="horizontal"`: låddiagrammet ligger vågrätt, så provpoängen visas på x-axeln.
- `gridspec_kw={"height_ratios": [4, 1]}`: det översta diagrammet får fyra gånger så mycket höjd som det nedersta.
- `ax_box.set_yticks([])`: tar bort en onödig markering på y-axeln.

**Låddiagrammet visar:** medianen (strecket i lådan), första och tredje kvartilen (lådans kanter), samt spridning och möjliga extremvärden. Lådan innehåller de **mittersta 50 procenten** av observationerna. De så kallade *whiskers* sträcker sig vanligtvis till de mest extrema värdena inom 1,5 × IQR från respektive kvartil – **inte nödvändigtvis till absolut min och max**.

> **Förtydligande:** I äldre kodexempel används ibland `vert=False` för ett horisontellt låddiagram. I nyare Matplotlib är `orientation="horizontal"` det tydligare alternativet.

---

## 5. Två numeriska variabler → spridningsdiagram

**Fråga:** Har elever som studerar fler timmar också högre provpoäng?

```python
par = df[["studietimmar", "provpoäng"]].dropna()

x = par["studietimmar"].to_numpy()
y = par["provpoäng"].to_numpy()

fig, ax = plt.subplots(figsize=(8, 5))
ax.scatter(x, y, alpha=0.7)
ax.set_title("Studietimmar och provpoäng")
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
ax.grid(True)
plt.tight_layout()
plt.show()
```

### Varför `df[[...]]` med dubbla hakparenteser?

- `df["provpoäng"]` hämtar **en** kolumn och ger en Series.
- `df[["studietimmar", "provpoäng"]]` hämtar **två** kolumner och ger en DataFrame.
- `.dropna()` tar här bort rader där någon av de två kolumnerna saknar värde, så `x` och `y` fortsätter höra ihop elev för elev.
- `.to_numpy()` omvandlar Pandas-serien till en NumPy-array (`ndarray`).
- `alpha=0.7` gör punkterna delvis genomskinliga.

### Lägg till en trendlinje med `np.polyfit()`

```python
k, m = np.polyfit(x, y, 1)
ordning = np.argsort(x)

fig, ax = plt.subplots(figsize=(8, 5))
ax.scatter(x, y, alpha=0.7, label="Elever")
ax.plot(x[ordning], k * x[ordning] + m, label="Trendlinje")
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
ax.legend()
plt.tight_layout()
plt.show()
```

Vi använder **räta linjens ekvation**:

\[y = kx + m\]

- `k`: linjens lutning.
- `m`: värdet där linjen skär y-axeln, alltså när `x = 0`.
- `np.polyfit(x, y, 1)`: anpassar en rät linje (polynom av grad 1) med minsta kvadratmetoden.
- `np.argsort(x)`: ger index som ordnar `x` från lägst till högst. Detta gör att linjen ritas från vänster till höger utan att hoppa mellan punkterna.

**Viktigt:** Trendlinjen beskriver ett mönster i de observerade värdena. Den bevisar **inte** att fler studietimmar orsakar högre provpoäng. För `np.polyfit()` behövs också minst två giltiga punkter med varierande x-värden.

---

## 6. Två stapelserier – varför skalan kan bli missvisande

Läraren jämför provpoäng och studietimmar med staplar per elev. Problemet är att provpoäng kan ligga kring 50–100 medan studietimmar kan ligga kring 0–10. På samma y-axel ser timmarna då nästan obefintliga ut.

### Snabb variant med Pandas

```python
urval = df[["student", "provpoäng", "studietimmar"]].dropna().copy()

urval.plot(
    x="student",
    y=["provpoäng", "studietimmar"],
    kind="bar",
    figsize=(12, 5),
)
plt.ylabel("Värde – olika enheter!")
plt.tight_layout()
plt.show()
```

Pandas skapar diagrammet via Matplotlib. Det är en snabb genväg, men jämförelsen av **höjder** är svårtolkad eftersom serierna mäts i olika enheter.

### Alternativ med Matplotlib: förskjut staplarna

```python
idx = np.arange(len(urval))
bredd = 0.35

fig, ax = plt.subplots(figsize=(12, 5))
ax.bar(idx - bredd / 2, urval["provpoäng"], width=bredd, label="Provpoäng")
ax.bar(idx + bredd / 2, urval["studietimmar"], width=bredd, label="Studietimmar")
ax.set_xticks(idx)
ax.set_xticklabels(urval["student"], rotation=45, ha="right")
ax.legend()
plt.tight_layout()
plt.show()
```

- `np.arange(len(urval))` skapar positionerna `0, 1, 2, ...`.
- `width=bredd` bestämmer varje stapels bredd.
- `idx - bredd / 2` och `idx + bredd / 2` flyttar staplarna åt var sitt håll så att de inte täcker varandra.

Det löser **överlappningen**, men **inte skillnaden i skala**.

## 7. Normalisering till 0–1

För att jämföra relativa positioner inom respektive variabel visar läraren *min–max-normalisering*:

\[\text{normaliserat värde} = \frac{x - \min(x)}{\max(x) - \min(x)}\]

Exempel: om en serie innehåller `[10, 20, 30]` blir resultatet `[0.0, 0.5, 1.0]`.

```python
def min_max(serie):
    min_varde = serie.min()
    max_varde = serie.max()
    spann = max_varde - min_varde

    if spann == 0:
        return serie * 0.0  # Alla giltiga värden är lika

    return (serie - min_varde) / spann

normaliserad = urval.copy()
normaliserad["provpoäng"] = min_max(normaliserad["provpoäng"])
normaliserad["studietimmar"] = min_max(normaliserad["studietimmar"])

print(normaliserad[["provpoäng", "studietimmar"]].agg(["min", "max"]))

normaliserad.plot(
    x="student",
    y=["provpoäng", "studietimmar"],
    kind="bar",
    figsize=(12, 5),
)
plt.ylabel("Normaliserat värde (0–1)")
plt.tight_layout()
plt.show()
```

### Förstå funktionens delar

- `def min_max(serie):` definierar en funktion med parametern `serie`.
- `.min()` och `.max()` hämtar lägsta och högsta värdet.
- `spann` är skillnaden mellan högsta och lägsta värdet.
- `if spann == 0:` skyddar mot division med noll.
- `return` skickar tillbaka den beräknade serien.
- `.copy()` gör en kopia så att ursprungstabellen inte ändras.
- `.agg(["min", "max"])` räknar både min och max för kolumnerna.

**Viktig begränsning:** Normalisering gör skalorna jämförbara som **relativa positioner**, men den gör inte timmar och provpoäng till samma måttenhet. Värdet 0 betyder **lägsta observerade värde i just den serien**, inte nödvändigtvis noll timmar eller noll poäng. I små dataset kan enstaka extremvärden dessutom påverka normaliseringen mycket.

För att studera samband mellan två numeriska variabler är ett **spridningsdiagram oftast bättre** än 77 par staplar.

---

## 8. En kategorisk + en numerisk → gruppjämförelse

**Fråga:** Hur skiljer sig medelpoängen mellan klasserna?

Vi sammanfattar först med `groupby()` och `agg()`.

```python
summary = (
    df.groupby("klass", dropna=False)
      .agg(
          medel=("provpoäng", "mean"),
          std=("provpoäng", "std"),
          antal=("provpoäng", "count"),
      )
      .reset_index()
)

print(summary)
```

### Vad gör varje del?

| Del | Förklaring |
|---|---|
| `.groupby("klass")` | Samlar rader med samma klass |
| `dropna=False` | Behåller även en eventuell grupp där klass saknas |
| `.agg(...)` | Skapar flera sammanfattningskolumner |
| `("provpoäng", "mean")` | Medelvärde av provpoängen |
| `("provpoäng", "std")` | Standardavvikelse för provpoängen |
| `("provpoäng", "count")` | Antal **icke-saknade provpoäng** |
| `.reset_index()` | Gör gruppnamnet till en vanlig kolumn igen |

### Stapeldiagram med felstaplar

Läraren visar **standardfel** (*standard error*, SE), som beräknas:

\[SE = \frac{s}{\sqrt{n}}\]

Där `s` är stickprovets standardavvikelse och `n` är antalet giltiga observationer.

```python
summary["se"] = summary["std"] / np.sqrt(summary["antal"])

fig, ax = plt.subplots(figsize=(8, 5))
ax.bar(
    summary["klass"].astype(str),
    summary["medel"],
    yerr=summary["se"],
    capsize=4,
)
ax.set_title("Medelpoäng per klass med standardfel")
ax.set_xlabel("Klass")
ax.set_ylabel("Medelpoäng")
ax.grid(True, axis="y")
plt.tight_layout()
plt.show()
```

- `yerr=...` anger felstaplarnas längd.
- `capsize=4` lägger till små tvärstreck längst ut.
- **Standardavvikelse** beskriver spridningen bland individerna.
- **Standardfel** beskriver hur mycket ett skattat medelvärde typiskt varierar mellan tänkbara stickprov, under vissa antaganden.

**Obs:** `mean ± SE` är **inte automatiskt ett 95-procentigt konfidensintervall**. Vid bara en giltig observation blir standardavvikelse och SE normalt saknade (`NaN`).

## 9. Låddiagram per klass – jämför hela fördelningen

Ett stapeldiagram över medelvärden kan dölja skillnader i spridning. Läraren visar därför även ett låddiagram:

```python
fig, ax = plt.subplots(figsize=(8, 5))
df.boxplot(column="provpoäng", by="klass", ax=ax)
ax.set_title("Provpoäng per klass")
ax.set_xlabel("Klass")
ax.set_ylabel("Provpoäng")
plt.suptitle("")  # Tar bort Pandas automatiska överrubrik
plt.tight_layout()
plt.show()
```

**Läs av:** median, kvartiler, IQR, whiskers och eventuella extremvärden. Två klasser kan ha liknande medelvärde men mycket olika spridning.

---

## 10. Två kategoriska variabler → korstabell och andelar

**Fråga:** Hur stor andel av eleverna i varje klass ligger i olika poängintervall?

Provpoäng är från början numeriska. Med `pd.cut()` gör vi om dem till **kategorier**.

```python
df_copy = df.copy()

bins = [0, 60, 80, 100]

df_copy["poängintervall"] = pd.cut(
    df_copy["provpoäng"],
    bins=bins,
    right=True,
    include_lowest=True,
)

andelar = pd.crosstab(
    df_copy["klass"],
    df_copy["poängintervall"],
    normalize="index",
)

print(andelar.round(2))
```

### Förklaring

- `pd.cut(...)` delar en numerisk kolumn i intervall.
- `bins=[0, 60, 80, 100]` skapar tre intervall: ungefär 0–60, 60–80 och 80–100.
- `right=True` betyder att den **högra** gränsen ingår i respektive intervall.
- `include_lowest=True` gör att även den lägsta gränsen ingår i första intervallet.
- `pd.crosstab(rader, kolumner)` räknar kombinationer av kategorier.
- `normalize="index"` gör om antalen till **andelar per rad**, så varje klass summerar till 1 (om den har observationer som hamnar i intervallen).
- `.round(2)` visar två decimaler utan att ändra originalvärdena.

Exempel på tolkning: `0.25` i raden för klass A och kolumnen 60–80 betyder att **25 procent av de medräknade eleverna i klass A** ligger i intervallet.

**Var uppmärksam på:** Provpoäng utanför intervallet 0–100 får saknad intervallkategori och ingår inte i korstabellens beräkning. Kontrollera därför datan innan du drar slutsatser.

Vill du visa andelarna visuellt kan du använda ett staplat stapeldiagram:

```python
andelar.plot(kind="bar", stacked=True, figsize=(9, 5))
plt.title("Andel elever per poängintervall och klass")
plt.xlabel("Klass")
plt.ylabel("Andel")
plt.legend(title="Poängintervall")
plt.tight_layout()
plt.show()
```

Detta är ett **kompletterande visualiseringsexempel** utöver lärarens korstabell.

---

## 11. Vanliga misstag och hur du undviker dem

1. **Välja diagram innan frågan är tydlig.** Börja med att formulera vad du vill jämföra.
2. **Blanda olika skalor på samma y-axel.** Studietimmar och provpoäng är inte direkt jämförbara som stapelhöjder.
3. **Ta bort saknade värden separat i x och y.** Då kan observationer hamna fel. Använd `.dropna()` på båda kolumnerna samtidigt.
4. **Tro att `sharex=True` räcker för att jämföra histogram och lodrätt låddiagram.** Låddiagrammet måste ligga horisontellt för att dela samma värdeskala.
5. **Förväxla medelvärde och median.** Medelvärde påverkas ofta mer av extremvärden.
6. **Tolka whiskers som alltid min/max.** Standardregeln använder vanligen 1,5 × IQR.
7. **Tolka standardfel som standardavvikelse eller konfidensintervall.** De beskriver olika saker.
8. **Använda normalisering utan att förstå vad 0 och 1 betyder.** Det är respektive series min- och maxvärden.
9. **Glömma `.copy()` när du vill bevara originaldatan.** Kopiera innan du skapar eller ändrar kolumner.
10. **Förväxla samband och orsak.** Ett tydligt mönster i ett scatterdiagram bevisar inte orsakssamband.

## 12. Quick Reference

| Vill du … | Använd |
|---|---|
| Se de första raderna | `df.head()` |
| Kontrollera datatyper | `df.info()` |
| Välja en kolumn | `df["kolumn"]` |
| Välja flera kolumner | `df[["a", "b"]]` |
| Ta bort saknade värden | `.dropna()` |
| Skapa figur och axel | `fig, ax = plt.subplots()` |
| Visa fördelning | `ax.hist(värden, bins=10)` |
| Visa median och spridning | `ax.boxplot(värden)` |
| Visa samband | `ax.scatter(x, y)` |
| Rita trendlinje | `np.polyfit(x, y, 1)` |
| Visa staplar | `ax.bar(kategorier, värden)` |
| Visa flera stapelserier snabbt | `df.plot(kind="bar")` |
| Sammanfatta per grupp | `df.groupby("klass").agg(...)` |
| Skapa poängintervall | `pd.cut(df["poäng"], bins=[0, 60, 80, 100])` |
| Räkna andelar per grupp | `pd.crosstab(a, b, normalize="index")` |
| Lägga till felstaplar | `ax.bar(x, y, yerr=se)` |
| Visa diagrammet | `plt.show()` |

---

## 13. Övningar

Använd övningsdatan i avsnitt 2 eller din egen rensade CSV-fil.

**Övning 1 – Välj figur.** Vilken diagramtyp väljer du för (a) fördelning av provpoäng, (b) samband mellan studietimmar och provpoäng, (c) median och spridning per klass, (d) andelar i poängintervall per klass?

**Övning 2 – Histogram.** Rita ett histogram över `studietimmar` med fyra bins. Lägg till titel och axelrubriker.

**Övning 3 – Två diagram.** Placera histogrammet över provpoäng och ett horisontellt låddiagram under det. Ge det översta diagrammet tre gånger så stor höjd.

**Övning 4 – Samband.** Rita ett spridningsdiagram över studietimmar och provpoäng. Lägg till en trendlinje.

**Övning 5 – Gruppjämförelse.** Beräkna medelpoäng och antal per klass med `groupby()` och `agg()`.

**Övning 6 – Standardfel.** Beräkna även standardavvikelse och SE per klass. Vad är skillnaden mellan dessa två mått?

**Övning 7 – Normalisering.** Normalisera `[10, 20, 30]` till 0–1. Vad händer om alla värden är 10?

**Övning 8 – Andelar.** Skapa poängintervallen 0–60, 60–80 och 80–100 med `pd.cut()`. Visa andelarna per klass med `pd.crosstab(..., normalize="index")`.

---

## 14. Facit

### Facit 1

(a) Histogram. (b) Spridningsdiagram. (c) Låddiagram per klass. (d) Korstabell med radnormaliserade andelar, eventuellt ett staplat stapeldiagram.

### Facit 2

```python
fig, ax = plt.subplots()
ax.hist(df["studietimmar"].dropna(), bins=4)
ax.set_title("Fördelning av studietimmar")
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Antal elever")
plt.show()
```

### Facit 3

```python
poang = df["provpoäng"].dropna()
fig, (ax1, ax2) = plt.subplots(
    2, 1, sharex=True,
    gridspec_kw={"height_ratios": [3, 1]},
)
ax1.hist(poang, bins=5)
ax2.boxplot(poang, orientation="horizontal")
ax2.set_yticks([])
plt.tight_layout()
plt.show()
```

### Facit 4

```python
par = df[["studietimmar", "provpoäng"]].dropna()
x = par["studietimmar"].to_numpy()
y = par["provpoäng"].to_numpy()
k, m = np.polyfit(x, y, 1)
ordning = np.argsort(x)
fig, ax = plt.subplots()
ax.scatter(x, y)
ax.plot(x[ordning], k * x[ordning] + m)
plt.show()
```

### Facit 5

```python
resultat = df.groupby("klass").agg(
    medel=("provpoäng", "mean"),
    antal=("provpoäng", "count"),
)
print(resultat)
```

Med övningsdatan blir klass A:s medelpoäng `71.75` och klass B:s `71.25`, med fyra elever i varje klass.

### Facit 6

```python
resultat = df.groupby("klass").agg(
    std=("provpoäng", "std"),
    antal=("provpoäng", "count"),
)
resultat["se"] = resultat["std"] / np.sqrt(resultat["antal"])
print(resultat)
```

Standardavvikelsen beskriver **spridningen mellan elevernas provpoäng**. Standardfelet beskriver **osäkerheten/variationen i det skattade medelvärdet vid upprepade stickprov**.

### Facit 7

```python
s = pd.Series([10, 20, 30])
print(min_max(s).tolist())  # [0.0, 0.5, 1.0]

lika = pd.Series([10, 10, 10])
print(min_max(lika).tolist())  # [0.0, 0.0, 0.0]
```

När alla värden är lika är spannet noll. Funktionen returnerar därför nollor i stället för att försöka dela med noll.

### Facit 8

```python
tabell = df.copy()
tabell["intervall"] = pd.cut(
    tabell["provpoäng"],
    bins=[0, 60, 80, 100],
    right=True,
    include_lowest=True,
)
andelar = pd.crosstab(
    tabell["klass"],
    tabell["intervall"],
    normalize="index",
)
print(andelar.round(2))
```

**Tolkning:** Varje rad visar fördelningen **inom den klassen**, inte andelen av alla elever i hela datasetet.

---

## 15. Sammanfattning – den viktigaste tanken

1. **En numerisk variabel:** histogram och/eller låddiagram.
2. **Två numeriska variabler:** spridningsdiagram, eventuellt trendlinje.
3. **Kategorisk + numerisk:** `groupby()` och stapeldiagram eller låddiagram.
4. **Kategorisk + kategorisk:** korstabell, ofta med `normalize="index"`.
5. **Olika enheter eller skalor:** fundera på vad jämförelsen faktiskt betyder innan du normaliserar eller ritar staplar.

**Nästa lektion:** `08_exploratory_data_analysis.md` – Mini-EDA med Python.
