# 06 – Fördjupning i Matplotlib

**Kurs:** Vecka 5 – Visualisering och explorativ dataanalys  
**Video:** 5.1 – Fördjupning i Matplotlib  
**Placering:** `data_science/data_analysis/06_advanced_matplotlib.md`

> **Målet:** Du ska förstå hur du bygger tydliga diagram med Matplotlib, kombinerar flera diagram, visar trender och spridning samt undviker vanliga feltolkningar. Genomgången följer lärarens video, men rättar felaktig syntax och förtydligar sådant som går snabbt i videon.

## 1. Vi börjar med data från Pandas

Läraren använder två CSV-filer från tidigare lektioner:

- `resultat_stadad.csv`: en rad per student, med exempelvis `student`, `studietimmar`, `provpoang`, `klass` och `poang_per_timme`.
- `summary_by_class.csv`: en rad per klass, med exempelvis `klass`, `antal`, `medelpoang`, `medianpoang`, `std_poang` och `medeltimmar`.

**Obs:** Filnamn och kolumnnamn i koden nedan är tydliga exempel. De måste matcha exakt det som faktiskt står i dina CSV-filer. Använd `print(df.columns.tolist())` om du är osäker.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("resultat_stadad.csv")
summary_class = pd.read_csv("summary_by_class.csv")

print(df.head(3))
print(df.columns.tolist())
print(summary_class)
```

**Förklaring:** `pd.read_csv()` läser en CSV-fil och returnerar en **DataFrame** (en tabell). `head(3)` visar de tre första raderna. Vi importerar NumPy som `np` för beräkningar och Matplotlibs `pyplot` som `plt` för diagram.

### Om du saknar lärarens CSV-filer

Följande exempeldata gör att du ändå kan testa lektionen från början till slut:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.DataFrame({
    "student": ["Ada", "Bo", "Cia", "Dan", "Eva", "Finn", "Gia", "Han", "Ida"],
    "klass": ["A", "A", "A", "B", "B", "B", "C", "C", "C"],
    "studietimmar": [2, 5, 8, 1, 4, 7, 3, 6, 9],
    "provpoang": [62, 78, 92, 55, 71, 81, 60, 74, 88],
})

summary_class = (
    df.groupby("klass", as_index=False)
      .agg(antal=("provpoang", "size"),
           medelpoang=("provpoang", "mean"),
           medianpoang=("provpoang", "median"),
           std_poang=("provpoang", "std"),
           medeltimmar=("studietimmar", "mean"))
)
```

De följande kodexemplen använder dessa kolumnnamn. Kör antingen inläsningen av dina egna filer **eller** exempeldata ovan först.

---

## 2. Tre sätt att rita diagram

### A. `plt` direkt – snabbt för enstaka diagram

```python
plt.figure(figsize=(8, 5))
plt.scatter(df["studietimmar"], df["provpoang"])
plt.title("Studietimmar och provpoäng")
plt.xlabel("Studietimmar")
plt.ylabel("Provpoäng")
plt.grid(True)
plt.show()
```

`plt.figure()` skapar en figur. `plt.scatter()` använder den aktuella axeln, som Matplotlib kan skapa automatiskt. Det är smidigt, men kan bli svårare att hålla reda på vid flera diagram.

### B. Pandas `.plot()` – genväg

```python
df["provpoang"].plot(kind="hist", bins=10, title="Fördelning av provpoäng")
plt.xlabel("Poäng")
plt.ylabel("Antal")
plt.show()
```

Pandas använder Matplotlib som standard för denna typ av plottning. `kind="hist"` betyder histogram.

### C. `fig, ax = plt.subplots()` – rekommenderat arbetssätt

```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.scatter(df["studietimmar"], df["provpoang"])
ax.set_title("Studietimmar och provpoäng")
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
ax.grid(True)
plt.show()
```

- **`fig`** = hela bilden, alltså *Figure*.
- **`ax`** = området där själva diagrammet ritas, alltså *Axes* (trots namnet är det ett objekt som innehåller båda koordinataxlarna).
- **`plt.subplots()`** = skapar en figur och ett eller flera diagramområden.

**Kom ihåg:** `plt.title(...)` hör till pyplot-sättet. `ax.set_title(...)` hör till Axes-sättet. Båda fungerar, men blanda inte i onödan.

---

## 3. Figurens storlek: `figsize`

```python
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot([1, 2, 3], [2, 1, 2])
plt.show()
```

`figsize=(10, 6)` betyder **10 tum bred** och **6 tum hög**. Det är inte direkt pixlar. Pixelstorleken vid sparning beror även på **DPI** (punkter per tum):

`bredd i pixlar = bredd i tum × DPI`

Exempel: `10 × 150 = 1500` pixlar när bilden sparas med `dpi=150`.

Vill du ändra storleken i efterhand är den korrekta metoden:

```python
fig.set_size_inches(12, 7)
```

**Förtydligande från videon:** Läraren försöker hitta metoden under `plt`, men den tillhör själva figurobjektet: `fig.set_size_inches(...)`.

## 4. Marginaler: `tight_layout()`

```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.set_title("Ett diagram med marginaler")
ax.set_xlabel("X-värde")
ax.set_ylabel("Y-värde")
fig.tight_layout()
plt.show()
```

`fig.tight_layout()` justerar mellanrum så att rubriker och etiketter mer sällan krockar eller kapas. Alternativt kan du skapa figuren med `layout="constrained"`, som ofta fungerar bra för mer komplicerade figurer:

```python
fig, ax = plt.subplots(figsize=(8, 5), layout="constrained")
```

Använd normalt **ett** layoutsystem åt gången.

---

## 5. Flera linjer på samma axel

```python
x = [1, 2, 3]
y1 = [2, 1, 2]
y2 = [0.5, 1.5, 1.0]

fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, y1, marker="o", linestyle="-", linewidth=2, label="Serie 1")
ax.plot(x, y2, marker="s", linestyle="--", linewidth=2, label="Serie 2")
ax.set_title("Två serier i samma diagram")
ax.set_xlabel("X")
ax.set_ylabel("Y")
ax.legend()
ax.grid(True)
plt.show()
```

**Så fungerar det:** Varje `ax.plot(...)` lägger till en linje på **samma** diagramområde. `label` namnger linjen och `ax.legend()` visar en **teckenförklaring**.

| Parameter | Betydelse | Exempel |
|---|---|---|
| `marker` | Symbol vid datapunkter | `"o"` = cirkel, `"s"` = fyrkant |
| `linestyle` | Linjens utseende | `"-"`, `"--"`, `":"`, `"-."` |
| `linewidth` | Linjens tjocklek | `2` |
| `label` | Namn i teckenförklaringen | `"Serie 1"` |
| `alpha` | Genomskinlighet | `0.6` |

**Vanligt misstag:** `label=` skapar inte en synlig teckenförklaring av sig självt. Lägg till `ax.legend()`.

---

## 6. Flera diagram i samma figur: subplots

```python
fig, axes = plt.subplots(2, 2, figsize=(10, 8))

axes[0, 0].hist(df["provpoang"].dropna(), bins=10)
axes[0, 0].set_title("Fördelning av provpoäng")
axes[0, 0].set_xlabel("Poäng")
axes[0, 0].set_ylabel("Antal")

axes[0, 1].scatter(df["studietimmar"], df["provpoang"], alpha=0.6)
axes[0, 1].set_title("Studietimmar och provpoäng")
axes[0, 1].set_xlabel("Studietimmar")
axes[0, 1].set_ylabel("Provpoäng")

axes[1, 0].bar(summary_class["klass"], summary_class["medelpoang"])
axes[1, 0].set_title("Medelpoäng per klass")
axes[1, 0].set_xlabel("Klass")
axes[1, 0].set_ylabel("Medelpoäng")

axes[1, 1].axis("off")

fig.suptitle("Översikt av studentresultat", fontsize=14)
fig.tight_layout()
plt.show()
```

### Vad betyder `axes[0, 1]`?

Python räknar från **noll**. För ett rutnät med två rader och två kolumner:

| | Kolumn 0 | Kolumn 1 |
|---|---|---|
| Rad 0 | `axes[0, 0]` | `axes[0, 1]` |
| Rad 1 | `axes[1, 0]` | `axes[1, 1]` |

`axes` är här en **NumPy-array med Axes-objekt**. `axes[1, 1].axis("off")` döljer den tomma rutan.

**Skillnaden:** `ax` är vanligtvis ett diagramområde, medan `axes` här är en samling av flera diagramområden.

### Delade skalor

```python
fig, axes = plt.subplots(1, 2, sharey=True, figsize=(10, 4))
```

`sharey=True` betyder att diagrammen delar y-axelns skala. Det gör jämförelser lättare när de visar samma typ av mått. `sharex=True` gör motsvarande för x-axeln.

### Gemensam rubrik och utrymme

`fig.suptitle(...)` sätter en rubrik för **hela figuren**, medan `ax.set_title(...)` sätter rubrik på **ett diagram**.

```python
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
fig.suptitle("Två diagram", fontsize=14)
fig.subplots_adjust(top=0.82)
```

`top=0.82` anger var den övre kanten av diagramområdena ska ligga som andel av figurens höjd. Det betyder **inte** att 82 % av ytan ska vara en övre marginal. Om du använder `layout="constrained"` behövs ofta inte manuell justering.

---

## 7. Trendlinje i ett spridningsdiagram

Läraren visar hur vi kan komplettera punkter med en rät linje:

\[y = kx + m\]

- **`k`** = lutningen: hur mycket linjens y-värde ändras när x ökar med 1.
- **`m`** = skärningen med y-axeln, det vill säga linjens y-värde vid x = 0.

På engelska skrivs ofta samma idé som `y = mx + b`. Bokstäverna byter roll, men matematiken är densamma.

```python
# Välj bara rader där båda värdena finns
valid = df[["studietimmar", "provpoang"]].dropna()
x = valid["studietimmar"].to_numpy()
y = valid["provpoang"].to_numpy()

# Anpassa en rät linje (grad 1) till punkterna
k, m = np.polyfit(x, y, 1)

# Sortera x-värden så att linjen ritas från vänster till höger
idx = np.argsort(x)

fig, ax = plt.subplots(figsize=(9, 5))
ax.scatter(x, y, alpha=0.7, label="Studenter")
ax.plot(x[idx], k * x[idx] + m,
        linestyle="--", label=f"Trend: y = {k:.2f}x + {m:.2f}")
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
ax.set_title("Studietimmar och provpoäng med trendlinje")
ax.legend()
ax.grid(True)
fig.tight_layout()
plt.show()
```

### Ny syntax förklarad

- `to_numpy()` gör om en Pandas Series till en NumPy-array.
- `np.polyfit(x, y, 1)` hittar den räta linje som passar punkterna enligt **minsta kvadratmetoden**. `1` betyder polynom av **första graden** (en rät linje).
- `k, m = ...` är **uppackning**: två resultat sparas i varsin variabel.
- `np.argsort(x)` ger indexen som skulle sortera `x` i stigande ordning.
- `x[idx]` använder dessa index för att få sorterade x-värden.
- `k * x[idx] + m` beräknar linjens y-värden med NumPy, flera värden på en gång.
- `f"{k:.2f}"` visar `k` med **två decimaler**.

**Viktigt:** Trendlinjen visar ett **beskrivande mönster**, inte att studietimmar i sig bevisligen orsakar provpoängen. Den kan också påverkas av extremvärden.

**Extra tydlighet:** Att anpassa linjen med minsta kvadratmetoden är en enkel form av linjär regression rent matematiskt. Däremot har vi här **inte** gjort någon inferens med exempelvis hypotesprövning, konfidensintervall eller kausala slutsatser.

---

## 8. Referenslinjer för medelvärden

```python
x_mean = x.mean()
y_mean = y.mean()

fig, ax = plt.subplots(figsize=(9, 5))
ax.scatter(x, y, alpha=0.7, label="Studenter")
ax.plot(x[idx], k * x[idx] + m, linestyle="--", label="Trendlinje")

ax.axvline(x_mean, linestyle=":", label=f"Medeltimmar: {x_mean:.1f}")
ax.axhline(y_mean, linestyle=":", label=f"Medelpoäng: {y_mean:.1f}")

ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
ax.legend(loc="lower right")
plt.show()
```

- `ax.axvline(...)`: **lodrät** linje vid ett visst x-värde.
- `ax.axhline(...)`: **vågrät** linje vid ett visst y-värde.
- `loc="lower right"`: placera teckenförklaringen nere till höger.

Referenslinjer hjälper dig att se vilka observationer som ligger över respektive under medelvärdet.

---

## 9. Boxplot – jämför fördelningar mellan klasser

```python
fig, ax = plt.subplots(figsize=(8, 5))
df.boxplot(column="provpoang", by="klass", ax=ax)
ax.set_title("Provpoäng per klass")
ax.set_xlabel("Klass")
ax.set_ylabel("Poäng")
fig.suptitle("")  # Tar bort Pandas automatiska överrubrik
fig.tight_layout()
plt.show()
```

En **boxplot** (låddiagram) visar mer än bara ett medelvärde:

- **Strecket i lådan** = medianen, det mittersta värdet.
- **Lådans nedre kant** = första kvartilen, Q1.
- **Lådans övre kant** = tredje kvartilen, Q3.
- **Lådans höjd** = IQR = Q3 − Q1, intervallet som rymmer de mittersta 50 % av observationerna.
- **Whiskers (spröt)** = sträcker sig vanligtvis till de yttersta observationerna inom 1,5 × IQR från respektive kvartil, **inte automatiskt till min/max eller de yttre kvartilerna**.
- **Punkter utanför spröten** = möjliga avvikande observationer enligt boxplot-regeln. De är inte automatiskt felaktiga.

**Varför boxplot?** Två klasser kan ha samma medelvärde men olika spridning. En boxplot gör skillnaden synlig.

---

## 10. Felstaplar: standardavvikelse och standardfel

Läraren jämför två mått som inte betyder samma sak:

**Standardavvikelse (`std`)** beskriver hur mycket **enskilda observationer varierar** runt medelvärdet.

**Standardfel (`SE`)** beskriver hur mycket **ett skattat medelvärde förväntas variera mellan upprepade urval**, under vanliga statistiska antaganden.

Formeln för standardfelet för medelvärdet är:

\[SE = \frac{s}{\sqrt{n}}\]

Där `s` är stickprovets standardavvikelse och `n` är antalet observationer.

```python
kategorier = summary_class["klass"]
medel = summary_class["medelpoang"]
std = summary_class["std_poang"]
antal = summary_class["antal"]

se = std / np.sqrt(antal)

fig, ax = plt.subplots(figsize=(8, 5))
ax.bar(kategorier, medel, yerr=se, capsize=4)
ax.set_title("Medelpoäng per klass med standardfel")
ax.set_xlabel("Klass")
ax.set_ylabel("Medelpoäng")
ax.grid(True, axis="y")
fig.tight_layout()
plt.show()
```

- `yerr=se` anger hur långt felstapeln ska sträcka sig **uppåt och nedåt** från varje medelvärde.
- `capsize=4` lägger till små tvärstreck i ändarna.
- `np.sqrt(antal)` beräknar kvadratroten ur antalet i varje klass.

### Exempel med siffror

Om standardavvikelsen är 12 och antalet elever är 36:

\[SE = 12 / \sqrt{36} = 12 / 6 = 2\]

Felstaplarna visar då **medelvärde ± 2** när `yerr=se`.

**Varning:** Ett standardfel är **inte samma sak som ett 95-procentigt konfidensintervall**. För ett sådant behövs en separat beräkning och antaganden. Om du använder `yerr=std` visar du spridningen mellan individer i stället för standardfelet. Skriv alltid i rubrik eller bildtext **vad felstaplarna betyder**.

---

## 11. Ändra axelgränser och markeringar

```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.scatter(df["studietimmar"], df["provpoang"])
ax.set_xlim(0, 10)            # X-axeln visas från 0 till 10
ax.set_ylim(0, 100)           # Y-axeln visas från 0 till 100
ax.set_xticks([0, 2, 4, 6, 8, 10])
ax.set_yticks([0, 25, 50, 75, 100])
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
plt.show()
```

- `set_xlim(min, max)` = synligt intervall på x-axeln.
- `set_ylim(min, max)` = synligt intervall på y-axeln.
- `set_xticks([...])` och `set_yticks([...])` = var skalans markeringar placeras.

**Viktigt:** Att begränsa axeln kan dölja punkter och ge en missvisande bild. Särskilt i stapeldiagram är det ofta lämpligt att y-axeln börjar på noll, så staplarnas längder inte överdriver skillnader.

### Vrid långa etiketter

```python
fig, ax = plt.subplots()
ax.bar(["Klass med långt namn A", "Klass med långt namn B"], [75, 82])
ax.tick_params(axis="x", labelrotation=45)
fig.tight_layout()
plt.show()
```

Det här är ett enkelt och robust sätt att vrida texten 45 grader. Det gör kategorinamn lättare att läsa när de annars överlappar.

---

## 12. Spara diagram som bild

```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.hist(df["provpoang"].dropna(), bins=10)
ax.set_title("Fördelning av provpoäng")
ax.set_xlabel("Poäng")
ax.set_ylabel("Antal")
fig.tight_layout()
fig.savefig("histogram_provpoang.png", dpi=150, bbox_inches="tight")
plt.show()
```

`savefig()` sparar figuren som en bildfil. `dpi=150` styr upplösningen och `bbox_inches="tight"` hjälper till att få med etiketter i bildens kanter. Spara gärna **före** `plt.show()` för konsekvent beteende i olika miljöer.

---

## 13. Komplett mini-exempel – tre diagram och en sammanfattning

Det här är en sammanhängande kodcell som fungerar med **exempeldata från avsnitt 1** eller med egna data som har samma kolumner.

```python
import numpy as np
import matplotlib.pyplot as plt

valid = df[["studietimmar", "provpoang"]].dropna()
x = valid["studietimmar"].to_numpy()
y = valid["provpoang"].to_numpy()
k, m = np.polyfit(x, y, 1)
x_line = np.linspace(x.min(), x.max(), 100)

fig, axes = plt.subplots(2, 2, figsize=(11, 8))

axes[0, 0].hist(y, bins=8)
axes[0, 0].set_title("Fördelning av provpoäng")
axes[0, 0].set_xlabel("Poäng")
axes[0, 0].set_ylabel("Antal")

axes[0, 1].scatter(x, y, alpha=0.7, label="Studenter")
axes[0, 1].plot(x_line, k * x_line + m, linestyle="--", label="Trendlinje")
axes[0, 1].set_title("Studietimmar och provpoäng")
axes[0, 1].set_xlabel("Studietimmar")
axes[0, 1].set_ylabel("Provpoäng")
axes[0, 1].legend()

axes[1, 0].bar(summary_class["klass"], summary_class["medelpoang"])
axes[1, 0].set_title("Medelpoäng per klass")
axes[1, 0].set_xlabel("Klass")
axes[1, 0].set_ylabel("Medelpoäng")
axes[1, 0].set_ylim(0, 100)

axes[1, 1].axis("off")
fig.suptitle("Miniöversikt av elevresultat", fontsize=15)
fig.tight_layout()
fig.savefig("oversikt_elevresultat.png", dpi=150, bbox_inches="tight")
plt.show()
```

**Tolkning:** Histogrammet beskriver **fördelningen** av provpoäng. Spridningsdiagrammet beskriver ett möjligt **samband** mellan timmar och poäng. Stapeldiagrammet jämför **medelvärden mellan grupper**. De besvarar olika frågor om samma data.

---

## 14. Vanliga misstag och felsökning

| Problem | Vanlig orsak | Lösning |
|---|---|---|
| `NameError: name 'plt' is not defined` | Matplotlib har inte importerats | `import matplotlib.pyplot as plt` |
| `KeyError: 'provpoang'` | Kolumnnamnet matchar inte CSV-filen | `print(df.columns.tolist())` |
| `FileNotFoundError` | CSV-filen ligger inte där Python letar | Kontrollera arbetskatalog och sökväg |
| `ValueError` vid `np.polyfit` | Saknade/icke-numeriska värden eller för få användbara punkter | Rensa med `dropna()` och kontrollera datatyperna |
| `IndexError` vid `axes[0, 1]` | Du skapade inte ett rutnät med flera Axes | Använd `plt.subplots(2, 2)` |
| Legend saknas | `label` finns men `legend()` saknas | Kör `ax.legend()` |
| Text kapas | Figuren är för liten eller marginalerna trånga | Större `figsize` och `fig.tight_layout()` |
| Diagram ser missvisande ut | Olämpliga axelgränser | Kontrollera `set_xlim()` och `set_ylim()` |

**VS Code/Jupyter:** Om ett paket saknas, installera det i **samma Pythonmiljö som notebookens kernel använder**. Du kan i en notebook köra `%pip install matplotlib pandas numpy` och sedan starta om kerneln om det behövs.

---

## 15. Quick Reference

| Kod | Vad den gör |
|---|---|
| `fig, ax = plt.subplots()` | Skapar en figur och en axel |
| `plt.subplots(2, 2)` | Skapar fyra diagramområden |
| `figsize=(10, 6)` | Anger storlek i tum |
| `fig.set_size_inches(12, 7)` | Ändrar storlek efteråt |
| `ax.plot(x, y)` | Linjediagram |
| `ax.scatter(x, y)` | Spridningsdiagram |
| `ax.hist(data, bins=10)` | Histogram |
| `ax.bar(kategorier, värden)` | Stapeldiagram |
| `ax.boxplot(data)` | Låddiagram |
| `ax.set_title("...")` | Diagramrubrik |
| `fig.suptitle("...")` | Rubrik för hela figuren |
| `ax.legend()` | Visar teckenförklaring |
| `ax.grid(True)` | Visar stödlinjer |
| `ax.axvline(x)` / `ax.axhline(y)` | Lodrät/vågrät referenslinje |
| `np.polyfit(x, y, 1)` | Anpassar en rät linje |
| `ax.bar(..., yerr=se, capsize=4)` | Staplar med felstaplar |
| `ax.set_xlim(a, b)` | Ändrar x-axelns synliga intervall |
| `ax.set_yticks([...])` | Väljer y-axelns markeringar |
| `ax.tick_params(axis="x", labelrotation=45)` | Roterar x-etiketter |
| `fig.tight_layout()` | Justerar marginaler |
| `fig.savefig("bild.png", dpi=150)` | Sparar bilden |

---

## 16. Övningar

Använd exempeldata i avsnitt 1, eller dina egna CSV-filer med samma kolumner.

**Övning 1.** Skapa ett spridningsdiagram över studietimmar och provpoäng. Sätt rubrik och axeltexter.

**Övning 2.** Gör figuren 12 tum bred och 5 tum hög. Hur många pixlar blir den om du sparar med 150 DPI?

**Övning 3.** Rita två linjer på samma axel, med olika markörer och en legend.

**Övning 4.** Skapa ett rutnät med två rader och två kolumner. Lägg ett histogram överst till vänster och ett spridningsdiagram överst till höger. Dölj nedre högra rutan.

**Övning 5.** Anpassa en trendlinje till studietimmar och provpoäng. Förklara vad `k` betyder.

**Övning 6.** Skapa en boxplot med provpoäng per klass. Vad visar lådan och medianstrecket?

**Övning 7.** Beräkna standardfelet om `std = 15` och `n = 25`. Vad visar det jämfört med standardavvikelsen?

**Övning 8.** Spara ett diagram till `mitt_diagram.png` med 150 DPI.

### Facit

**1.**

```python
fig, ax = plt.subplots()
ax.scatter(df["studietimmar"], df["provpoang"])
ax.set_title("Studietimmar och provpoäng")
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
plt.show()
```

**2.**

```python
fig, ax = plt.subplots(figsize=(12, 5))
fig.savefig("stor_figur.png", dpi=150)
```

Bredd: `12 × 150 = 1800` pixlar. Höjd: `5 × 150 = 750` pixlar.

**3.**

```python
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [2, 4, 3], marker="o", label="A")
ax.plot([1, 2, 3], [1, 3, 5], marker="s", label="B")
ax.legend()
plt.show()
```

**4.**

```python
fig, axes = plt.subplots(2, 2, figsize=(10, 7))
axes[0, 0].hist(df["provpoang"], bins=8)
axes[0, 1].scatter(df["studietimmar"], df["provpoang"])
axes[1, 1].axis("off")
fig.tight_layout()
plt.show()
```

**5.**

```python
clean = df[["studietimmar", "provpoang"]].dropna()
x = clean["studietimmar"].to_numpy()
y = clean["provpoang"].to_numpy()
k, m = np.polyfit(x, y, 1)
x_line = np.linspace(x.min(), x.max(), 100)
fig, ax = plt.subplots()
ax.scatter(x, y)
ax.plot(x_line, k * x_line + m)
plt.show()
```

`k` beskriver hur många poäng trendlinjen ändras per ytterligare studietimme. Det är **inte** automatiskt en orsak–verkan-effekt.

**6.**

```python
fig, ax = plt.subplots()
df.boxplot(column="provpoang", by="klass", ax=ax)
fig.suptitle("")
plt.show()
```

Lådan täcker de mittersta 50 % av observationerna (Q1 till Q3). Strecket i lådan är medianen.

**7.** `SE = 15 / √25 = 15 / 5 = 3`. Standardavvikelsen 15 beskriver variationen bland individerna; standardfelet 3 beskriver osäkerheten i skattningen av medelvärdet under vanliga antaganden.

**8.**

```python
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [3, 2, 4])
fig.savefig("mitt_diagram.png", dpi=150)
plt.show()
```

---

## 17. Vad du ska kunna efter lektionen

- Förklara skillnaden mellan `fig`, `ax` och `axes`.
- Skapa och storleksanpassa diagram.
- Rita flera linjer på samma axel och använda `legend()`.
- Bygga rutnät med `plt.subplots()` och hitta rätt ruta med `[rad, kolumn]`.
- Lägga till trendlinjer och medelvärdeslinjer utan att dra för stora slutsatser.
- Tolka boxplots och skilja standardavvikelse från standardfel.
- Styra axelgränser och etiketter utan att vilseleda.
- Spara tydliga diagram som PNG.

**Nästa lektion:** `07_choosing_visualizations.md` – *5.2 Välj rätt figur*.
