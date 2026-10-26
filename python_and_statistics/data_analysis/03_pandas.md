# 03 – Pandas och DataFrames

> **Vecka 4 · Dataanalys · Video 4.3**  
> Mål: förstå vad Pandas gör, läsa CSV-filer, undersöka data, välja och filtrera rader, hantera saknade värden och sammanfatta grupper.

## 1. Vad är Pandas – och varför använder vi det?

**Pandas** är ett Pythonbibliotek för tabellformade data. Föreställ dig ett Excelark: varje **rad** beskriver en observation (t.ex. en student) och varje **kolumn** beskriver en egenskap (t.ex. studietimmar). Pandas låter oss läsa, filtrera, beräkna och ändra sådana tabeller med Pythonkod.

```python
import pandas as pd
```

- `import` laddar ett bibliotek.
- `pandas` är bibliotekets namn.
- `as pd` ger ett kort alias. Därför skriver vi `pd.DataFrame(...)` i stället för `pandas.DataFrame(...)`.

**Förutsättning:** installera vid behov med `python -m pip install pandas` i projektets aktiva virtuella miljö. I en notebook måste rätt Python-kernel vara vald.

## 2. Series – en kolumn med index

```python
s = pd.Series([10, 20, 30], name="studietimmar")
print(s)
```

Ungefärlig utskrift:

```text
0    10
1    20
2    30
Name: studietimmar, dtype: int64
```

**Så läser du koden:** `pd.Series(...)` skapar en Series; listan `[10, 20, 30]` innehåller värdena; `name=` sätter ett namn; siffrorna `0, 1, 2` till vänster är **indexetiketter**. `dtype` beskriver datatypen. Exakt heltalsstorlek kan variera.

En Series är **inte bara en vanlig Pythonlista**: den har index, datatyp och Pandasmetoder. Den kan använda NumPy internt, men är inte alltid bokstavligen en NumPy-array.

```python
print(s.values)  # de underliggande värdena, ofta som ndarray
print(s.index)   # indexetiketterna
print(type(s))   # <class 'pandas.core.series.Series'>
```

## 3. DataFrame – en hel tabell

```python
df_demo = pd.DataFrame({
    "student": ["A", "B", "D"],
    "studietimmar": [10, 5, 8],
    "provpoäng": [85, 70, 78],
})
print(df_demo)
```

```text
  student  studietimmar  provpoäng
0       A            10        85
1       B             5        70
2       D             8        78
```

**Dictionary** `{...}` kopplar varje kolumnnamn (nyckel) till en lista (värden). Alla listor måste ha samma antal element. `DataFrame` skrivs med stort `D` och `F` eftersom det är namnet på en klass.

- `df_demo.columns` → kolumnetiketter.
- `df_demo.index` → radetiketter.
- `df_demo.shape` → `(3, 3)` = tre rader, tre kolumner.
- `df_demo.dtypes` → datatyp för **varje** kolumn.
- `df_demo.to_numpy()` → NumPy-representation, men blandade datatyper kan ge en gemensam `object`-typ.

## 4. Läsa in en CSV-fil

```python
df = pd.read_csv("studietimmarpoäng.csv")
print(df.head())
```

- `pd.read_csv(...)` läser filen och **returnerar en DataFrame**.
- Filvägen är relativ till programmets **aktuella arbetskatalog**, inte alltid till notebookfilens plats.
- `.head()` visar de fem första raderna; `.head(10)` visar tio.
- Om filen använder semikolon: `pd.read_csv("fil.csv", sep=";")`.

Om du får `FileNotFoundError`: kontrollera filnamn, mapp och arbetskatalog (`Path.cwd()`). Vi kan träna alla kodexempel nedan med `df_demo` om kursens CSV-fil inte finns tillgänglig.

## 5. Första kontrollen av okänd data

```python
print(df.head())
print(df.shape)
print(df.columns)
print(df.dtypes)
df.info()
print(df.describe())
```

| Kod | Vad får vi veta? |
|---|---|
| `head()` | Hur de första raderna ser ut |
| `shape` | Antal rader och kolumner |
| `columns` | Exakta kolumnnamn |
| `dtypes` | Datatyper per kolumn |
| `info()` | Antal icke-saknade värden och datatyper |
| `describe()` | Statistik för numeriska kolumner som standard |

**Viktigt:** `shape`, `columns` och `dtypes` är **attribut** (utan `()`); `head()`, `info()` och `describe()` är **metoder** (med `()`).

`describe()` visar bl.a. `count`, `mean`, `std`, `min`, `25%`, `50%`, `75%`, `max`. `50%` är medianen. `std` är stickprovsstandardavvikelse som standard. `count` räknar icke-saknade värden, inte alltid alla rader. För textkolumner kan du använda `df.describe(include="all")`.

## 6. Rena och konsekventa kolumnnamn

```python
df.columns = (
    df.columns
    .str.strip()
    .str.replace(" ", "_", regex=False)
    .str.lower()
)
```

Steg för steg:

1. `df.columns` hämtar kolumnnamnen.
2. `.str.strip()` tar bort inledande och avslutande blanksteg.
3. `.str.replace(" ", "_", regex=False)` byter vanliga mellanslag mot understreck.
4. `.str.lower()` gör bokstäverna små.
5. `df.columns = ...` **tilldelar** de nya namnen till tabellen.

Exempel: `" Prov Poäng "` → `"prov_poäng"`. Detta ändrar **kolumnnamnen**, inte cellernas värden. Kontrollera namnen efteråt, eftersom resten av koden måste använda de nya namnen.

## 7. Saknade värden: NaN och NA

Saknade uppgifter representeras ofta som `NaN`, `NaT` (datum) eller `pd.NA`, beroende på datatyp. Saknat betyder **inte automatiskt noll**.

```python
print(df.isna().sum())
```

- `isna()` skapar en tabell av `True`/`False` för varje cell.
- `sum()` räknar `True` per kolumn (`True` räknas som 1).

Om en DataFrame har 80 rader och `info()` säger att `studietimmar` har 76 `non-null`, saknas fyra värden i den kolumnen.

## 8. Räkna kategorier med value_counts

```python
print(df["klass"].value_counts())
print(df["klass"].value_counts(dropna=False))
print(df["klass"].value_counts(normalize=True))
```

- `df["klass"]` väljer en kolumn → Series.
- `.value_counts()` räknar förekomster.
- `dropna=False` inkluderar saknade kategorier.
- `normalize=True` ger **andelar** mellan 0 och 1, inte procent direkt. Multiplicera med 100 för procent.

## 9. Välja kolumner och filtrera rader

```python
print(df["studietimmar"].head())           # en kolumn → Series
print(df[["studietimmar", "provpoäng"]].head())  # flera → DataFrame
```

**Varför dubbla hakparenteser?** Yttre `[...]` väljer från DataFrame; inre `[...]` är en Pythonlista med kolumnnamn.

```python
mask = df["studietimmar"] >= 5
print(mask.head())
print(df[mask].head())
```

Masken är en Series med `True` eller `False` för varje rad. Bara `True`-raderna kommer med.

### Flera villkor

```python
# OCH: båda villkoren måste stämma
urval = df[(df["studietimmar"] >= 5) & (df["provpoäng"] >= 80)]

# ELLER: minst ett villkor måste stämma
urval_eller = df[(df["studietimmar"] >= 5) | (df["provpoäng"] >= 80)]

# INTE: vänd True till False och tvärtom
motsatsen = df[~(df["studietimmar"] >= 5)]
```

- `&` = elementvis **och**.
- `|` = elementvis **eller**.
- `~` = elementvis **inte**.
- Använd parenteser runt varje jämförelse. Vanliga `and`, `or`, `not` fungerar inte direkt på en hel Pandas-Series.
- Vid vanliga numeriska datatyper blir `NaN >= 5` falskt. För explicita saknade värden: `df["studietimmar"].isna()`.

## 10. `loc` och `iloc` – två sätt att välja rader/kolumner

```python
print(df.loc[0:3, ["student", "studietimmar"]])
print(df.iloc[0:3, 0:2])
```

| | `loc` | `iloc` |
|---|---|---|
| Utgår från | **Etiketter** | **Positioner** |
| Rader | `0:3` inkluderar normalt etiketten 3 om den finns | `0:3` ger positionerna 0, 1, 2 |
| Kolumner | Namn som `"student"` | Positioner som `0:2` |

**Fallgrop:** efter filtrering kan index vara `0, 2, 5, ...`. Då är etikett `2` inte nödvändigtvis den tredje synliga raden. `loc` använder etikett; `iloc` använder position.

## 11. Sortera, byta namn, skapa och ta bort kolumner

```python
sorterad = df.sort_values("provpoäng", ascending=False)
print(sorterad.head(10))
```

`ascending=False` betyder fallande ordning (högst först). Standard är `True` (lägst först). Ursprungliga `df` ändras normalt inte av detta anrop.

```python
omdopt = df.rename(columns={"provpoäng": "poäng"})
```

Här är `{"provpoäng": "poäng"}` en dictionary: **gammalt namn → nytt namn**.

```python
med_ny_kolumn = df.assign(
    poang_per_timme=df["provpoäng"] / df["studietimmar"]
)
utan_ny_kolumn = med_ny_kolumn.drop(columns=["poang_per_timme"])
```

`assign()` returnerar en DataFrame med ny kolumn. `drop(columns=[...])` returnerar en utan den angivna kolumnen. Om studietimmar är noll kan division ge `inf` (oändlighet); i lektion 04 visar vi en säker lösning. Att skriva `df = df.assign(...)` innebär att variabeln `df` börjar peka på resultatet – så även om metoden returnerar en ny DataFrame har du då ersatt variabelns tidigare innehåll.

## 12. Kopiera, fylla eller ta bort saknade värden

```python
fylld = df.copy()
fylld["studietimmar"] = fylld["studietimmar"].fillna(0)

rensad = df.dropna(subset=["studietimmar", "provpoäng"])
rensad = rensad.reset_index(drop=True)
```

- `copy()` skapar en separat DataFrame.
- `fillna(0)` fyller saknade värden med noll – **bara när noll är en rimlig tolkning**.
- `dropna(subset=[...])` tar bort rader där någon av de uppräknade kolumnerna saknar värde.
- `reset_index(drop=True)` ger nya index `0, 1, 2, ...` utan att spara det gamla indexet som en extra kolumn.

**Analysfråga:** Om eleven inte rapporterat studietimmar vet vi inte om eleven studerat noll timmar. Att fylla med noll kan därför sänka medelvärdet på ett missvisande sätt. I videon jämförs båda metoderna för att visa skillnaden – det finns inget universellt facit för hur saknad data ska hanteras.

## 13. Gruppera och sammanfatta med `groupby` + `agg`

```python
sammanfattning = (
    df.groupby("klass")["provpoäng"]
      .agg(antal="count", medel="mean", median="median", spridning="std")
      .reset_index()
)
print(sammanfattning)
```

**Läs inifrån och ut:**

1. `groupby("klass")` delar raderna i grupper, en per klass.
2. `["provpoäng"]` väljer kolumnen som ska räknas på.
3. `.agg(...)` beräknar flera mått per grupp.
4. `antal="count"` räknar **icke-saknade provpoäng**; det är inte nödvändigtvis antal elever om poäng saknas.
5. `reset_index()` gör gruppetiketten `klass` till en vanlig kolumn igen.

Flera kolumner kan sammanfattas samtidigt:

```python
sammanfattning2 = (
    df.groupby("klass")
      .agg(
          provpoang_medel=("provpoäng", "mean"),
          provpoang_std=("provpoäng", "std"),
          studietimmar_median=("studietimmar", "median"),
      )
      .reset_index()
)
```

Här betyder `nytt_kolumnnamn=("ursprunglig_kolumn", "funktion")` att du själv namnger resultatet.

## 14. En första visualisering

```python
import matplotlib.pyplot as plt

ax = df.plot.scatter(x="studietimmar", y="provpoäng", alpha=0.7)
ax.set_title("Studietimmar och provpoäng")
plt.show()
```

Varje punkt är en rad med två numeriska värden. `x` är vågrät axel, `y` lodrät. `alpha=0.7` gör punkterna lite genomskinliga så överlappningar syns bättre. Pandas använder här Matplotlib i bakgrunden. Ett synligt samband betyder **inte automatiskt orsakssamband**.

## 15. Vanliga fel att känna igen

| Fel/problem | Orsak och lösning |
|---|---|
| `ModuleNotFoundError: pandas` | Installera Pandas i **rätt** Pythonmiljö/kernel |
| `FileNotFoundError` | CSV-filen hittas inte i aktuell arbetskatalog |
| `KeyError: 'provpoäng'` | Kolumnen finns inte under exakt det namnet; kontrollera `df.columns` |
| `df.shape()` | Fel: `shape` är attribut, skriv `df.shape` |
| `df.head` | Visar metodobjektet; kör `df.head()` |
| `df["a", "b"]` | För flera kolumner: `df[["a", "b"]]` |
| `and` mellan Series | Använd `&` och parenteser |
| `loc` förväxlas med `iloc` | `loc` = etiketter, `iloc` = positioner |
| Noll används för saknade värden | Noll är ett verkligt tal; saknad information kan behöva förbli saknad |

## 16. Quick Reference

```python
import pandas as pd

df = pd.read_csv("fil.csv")
df.head()                    # första fem raderna
df.shape                     # (rader, kolumner)
df.columns                   # kolumnnamn
df.dtypes                    # datatyper
df.info()                    # sammanfattad struktur
df.describe()                # numerisk statistik
df.isna().sum()              # saknade värden per kolumn
df["kolumn"]                 # en kolumn
df[["a", "b"]]              # två kolumner
df[df["a"] > 5]              # filtrera rader
df.loc[:, ["a", "b"]]       # etikettbaserat urval
df.iloc[:5, :2]              # positionsbaserat urval
df.sort_values("a")          # sortera
df.dropna(subset=["a"])     # ta bort saknade i a
df.groupby("grupp")["a"].mean()  # medel per grupp
```

## 17. Övningar

Använd `df_demo` från avsnitt 3 om du saknar kursens CSV.

1. Skapa en `Series` med värdena `4, 8, 12` och namnet `poäng`.
2. Visa antalet rader och kolumner i `df_demo`.
3. Välj bara kolumnerna `student` och `provpoäng`.
4. Filtrera fram studenter med minst 8 studietimmar.
5. Sortera `df_demo` med högsta provpoäng först.
6. Lägg till en ny kolumn `godkand` som är `True` när provpoäng är minst 75.
7. Skapa en sammanfattning med medelvärdet för `provpoäng`.
8. Förklara med egna ord skillnaden mellan `loc` och `iloc`.

### Facit

```python
# 1
s = pd.Series([4, 8, 12], name="poäng")

# 2
print(df_demo.shape)  # (3, 3)

# 3
print(df_demo[["student", "provpoäng"]])

# 4
print(df_demo[df_demo["studietimmar"] >= 8])  # A och D

# 5
print(df_demo.sort_values("provpoäng", ascending=False))

# 6
resultat = df_demo.assign(godkand=df_demo["provpoäng"] >= 75)
print(resultat)

# 7
print(df_demo["provpoäng"].mean())  # 77.666...
```

**8:** `loc` väljer efter rad- och kolumnetiketter; `iloc` väljer efter deras numeriska positioner.

---

**Nästa lektion:** `04_data_cleaning.md` – vi bygger vidare på samma arbetsflöde och skapar en rensningspipeline.
