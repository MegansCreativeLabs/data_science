# 04 – Rensa och sammanfatta data med Pandas

> **Vecka 4 · Dataanalys · Video 4.4**  
> Mål: skapa en begriplig **data-wrangling-pipeline**: läsa → inspektera → bestämma regler → rensa → kontrollera → sammanfatta → exportera.

## 1. Vad betyder data wrangling och pipeline?

**Data wrangling** är arbetet med att göra rådata användbar: rätta datatyper, hitta saknade värden, upptäcka dubbletter och skapa nya beräkningar. En **pipeline** är en serie steg som görs i bestämd ordning.

```text
CSV-fil
   ↓
Läs in med read_csv
   ↓
Standardisera kolumnnamn
   ↓
Kontrollera saknade värden och datatyper
   ↓
Bestäm hur saknad data ska hanteras
   ↓
Kontrollera dubbletter och konvertera typer
   ↓
Skapa nyckeltal och gruppsammanfattningar
   ↓
Kontrollera rimlighet
   ↓
Exportera nya CSV-filer
```

**Viktig princip:** behåll originalfilen orörd. Spara den rensade datan som en **ny** fil, så att arbetet går att kontrollera och upprepa.

## 2. Läs in filen och standardisera kolumnnamn

```python
import pandas as pd

df = pd.read_csv("studietimmarpoäng.csv")

df.columns = (
    df.columns
      .str.strip()
      .str.replace(" ", "_", regex=False)
      .str.lower()
)

print(df.head())
```

- `read_csv` läser CSV till en DataFrame.
- `.str.strip()` tar bort blanksteg i början/slutet av **kolumnnamnen**.
- `.str.replace(" ", "_", regex=False)` ersätter vanliga mellanslag.
- `.str.lower()` gör bokstäver små.
- `df.columns = ...` uppdaterar kolumnnamnen.

Om kursfilen saknas kan du skapa en liten **övningsfil** med exempel i slutet av lektionen. Resultat från övningsfilen är inte samma som lärarens 80-radersdataset.

## 3. Hälsokontroll: hur mår datan?

```python
df.info()
print(df.isna().sum())
print(df.dtypes)
```

**Tre frågor:**

1. Hur många rader och kolumner finns?
2. Vilka kolumner har saknade värden?
3. Är siffror verkligen numeriska, eller har de lästs in som text (`object`/`string`)?

`isna()` returnerar `True` där data saknas. `.sum()` räknar antalet `True` per kolumn. `NaN` betyder ofta saknat numeriskt värde; `NaT` betyder saknat datum/tid.

## 4. NaN-policy – ett medvetet beslut

I videon väljer läraren:

- **Saknad provpoäng:** ta bort raden, eftersom provpoängen är central för analysen.
- **Saknade studietimmar:** fyll med noll, för att demonstrera metoden.

**Viktig nyansering:** Saknade studietimmar är inte samma sak som **0 studietimmar**. Om eleven bara glömt rapportera kan noll ge missvisande statistik. Videon påpekar själv att valet är tveksamt. I verklig analys ska beslutet dokumenteras och gärna jämföras med ett alternativ.

```python
df_clean = df.dropna(subset=["provpoäng"]).copy()

df_clean.loc[
    df_clean["studietimmar"].isna(),
    "studietimmar"
] = 0
```

**Förklaring rad för rad:**

- `dropna(subset=["provpoäng"])`: ta bort rader där `provpoäng` saknas, men inte automatiskt rader där någon annan kolumn saknas.
- `.copy()`: skapa en självständig tabell att arbeta vidare i.
- `df_clean.loc[radval, kolumnval] = värde`: välj **vilka rader**, **vilken kolumn**, och sätt värdet.
- `df_clean["studietimmar"].isna()` är ett booleskt radfilter; endast saknade studietimmar får `0`.

**Varför `loc`?** Det är ett tydligt sätt att göra en tilldelning direkt i den DataFrame vi vill ändra och undvika tvetydig *chained assignment*.

## 5. Dubbletter: vad räknas som samma rad?

```python
print("Exakta dubblettrader:", df_clean.duplicated().sum())

df_clean = df_clean.drop_duplicates().reset_index(drop=True)
print("Unika studentvärden:", df_clean["student"].nunique())
```

- `duplicated()` ger `True` för en upprepad rad (den första förekomsten räknas normalt inte som dubblett).
- `drop_duplicates()` tar bort exakta kopior som standard.
- `reset_index(drop=True)` ger ett nytt sammanhängande radindex.
- `nunique()` räknar antalet **unika, icke-saknade värden** i kolumnen.

**Viktigt:** `duplicated()` och `nunique()` svarar på **olika frågor**. Två olika rader kan höra till samma student och vara helt legitima (t.ex. två prov). Att samma student-ID förekommer två gånger är inte automatiskt ett fel. Om en rad per student förväntas måste du uttryckligen bestämma vad som ska göras med upprepade ID:n.

## 6. Datatyper: kategori, tal och datum

```python
df_clean["klass"] = df_clean["klass"].astype("category")
print(df_clean.dtypes)
```

`astype("category")` markerar att `klass` är en kategorivariabel. Det kan minska minnesanvändning när många rader upprepar samma kategorier. Det gör också analysens avsikt tydligare.

### Om data har en datumkolumn

```python
# Kör ENDAST om kolumnen "datum" finns:
# df_clean["datum"] = pd.to_datetime(
#     df_clean["datum"], dayfirst=True, errors="coerce"
# )
```

- `pd.to_datetime(...)` tolkar text som datum.
- `dayfirst=True` anger att dag normalt kommer före månad i tvetydiga datum.
- `errors="coerce"` gör ogiltiga datum till `NaT` i stället för att stoppa programmet.

**Observera:** kursens exempeldata i videon har ingen datumkolumn, så detta är ett fristående syntaxexempel och ska inte köras direkt mot den filen.

### Numeriska värden som råkat bli text

```python
# Exempel om kolumnen behöver konverteras:
# df_clean["provpoäng"] = pd.to_numeric(df_clean["provpoäng"], errors="coerce")
```

Efter en sådan konvertering måste vi **kontrollera saknade värden igen**, eftersom ogiltig text kan ha blivit `NaN`.

## 7. Nyckeltal: poäng per studietimme utan division med noll

I videon beräknas `provpoäng / studietimmar`. Men om timmarna är noll är kvoten inte definierad. Vi använder `.where()` för att byta ut ogiltiga nämnare mot saknade värden.

```python
saker_namnare = df_clean["studietimmar"].where(
    df_clean["studietimmar"] > 0,
    other=pd.NA
)

df_clean = df_clean.assign(
    poang_per_timme=df_clean["provpoäng"] / saker_namnare
)
```

**Läs uttrycket i ordning:**

1. `df_clean["studietimmar"] > 0` ger `True` för timmar större än noll.
2. `.where(villkor, other=pd.NA)` behåller timmarna där villkoret är sant; annars blir värdet saknat.
3. Divisionen görs **elementvis** mellan två Series.
4. `assign(poang_per_timme=...)` skapar en ny kolumn.
5. `df_clean = ...` sparar den uppdaterade tabellen i variabeln.

**Matematisk förtydligande:** problemet är **division med noll** (ett positivt tal delat med noll), inte att dela noll med ett positivt tal. `0 / 5` är helt okej och blir `0`; `5 / 0` är inte definierat.

```python
print(df_clean[["student", "klass", "studietimmar", "provpoäng", "poang_per_timme"]].head())
```

## 8. Sammanfatta per klass: `groupby` och `agg`

```python
summary_class = (
    df_clean.groupby("klass", dropna=False, observed=True)
      .agg(
          antal=("student", "count"),
          medel_poang=("provpoäng", "mean"),
          median_poang=("provpoäng", "median"),
          std_poang=("provpoäng", "std"),
          medel_timmar=("studietimmar", "mean"),
      )
      .reset_index()
)
print(summary_class)
```

| Del | Betydelse |
|---|---|
| `groupby("klass")` | En grupp för varje klass |
| `dropna=False` | Behåll även en grupp för saknad klass, om sådan finns |
| `observed=True` | Visa bara kategorier som faktiskt förekommer i data |
| `agg(...)` | Beräkna flera sammanfattningar |
| `("provpoäng", "mean")` | Beräkna medelvärde på kolumnen `provpoäng` |
| `("provpoäng", "median")` | Beräkna median |
| `("provpoäng", "std")` | Beräkna stickprovsstandardavvikelse |
| `reset_index()` | Gör gruppnamnet till vanlig kolumn |

**Skillnad mellan `dropna` och `observed`:** `dropna=False` styr om saknade gruppnycklar får en egen grupp; `observed=True` styr om oanvända, men definierade, kategorier ska visas. Det är olika inställningar.

**Fallgrop:** `count` räknar **icke-saknade** studentvärden. Om du behöver antal **rader**, använd t.ex. `antal=("student", "size")`. Om en student kan ha flera rader är antal rader inte samma sak som antal unika studenter; använd då `nunique` om det är det du behöver.

## 9. Tvärtabell: poängintervall per klass

**Bins** är intervall som vi placerar numeriska värden i. Här delas provpoäng in i `(0, 60]`, `(60, 80]` och `(80, 100]`.

```python
bins = [0, 60, 80, 100]

andelar = pd.crosstab(
    df_clean["klass"],
    pd.cut(df_clean["provpoäng"], bins=bins, right=True, include_lowest=True),
    normalize="index"
).round(2)

print(andelar)
```

**Förklaring:**

- `pd.cut(..., bins=bins)` skapar kategorier utifrån talintervall.
- `right=True` inkluderar **högergränsen**: 60 hör till första intervallet; 80 till andra.
- `include_lowest=True` gör att även exakt 0 inkluderas i det första intervallet.
- `pd.crosstab(rader, kolumner)` räknar kombinationer av två kategorier.
- `normalize="index"` gör varje **rad** till andelar som summerar till ungefär 1 (avrundning kan påverka).
- `normalize="columns"` gör i stället varje **kolumn** till andelar som summerar till 1.
- `.round(2)` avrundar till två decimaler.

**Viktigt:** värden utanför intervallen 0–100 får ingen bin och ingår inte i tabellens vanliga intervall. Kontrollera därför datan innan du tolkar andelarna.

## 10. Rimlighetskontroll med `assert`

```python
assert df_clean["provpoäng"].between(0, 100).all(), "Provpoäng utanför 0–100"
assert (df_clean["studietimmar"] >= 0).all(), "Negativa studietimmar"
```

- `between(0, 100)` ger `True` för värden inom intervallet **inklusive gränserna**.
- `.all()` kräver att samtliga värden är `True`.
- `assert villkor, "meddelande"` ger `AssertionError` om villkoret är falskt.

**Tänk på:** `assert` är användbart i övningar och under utveckling, men för viktig produktionsvalidering är en uttrycklig `if` + `raise ValueError(...)` säkrare, eftersom `assert` kan avaktiveras när Python körs med optimering. Kontrollera också saknade värden separat; `between` är inte en ersättning för en fullständig datakvalitetskontroll.

## 11. Korrelation – ett mått på linjärt samband

```python
korrelation = df_clean[["studietimmar", "provpoäng"]].corr().iloc[0, 1]
print(f"Korrelation: {korrelation:.2f}")
```

- `[[...]]` väljer två kolumner.
- `.corr()` skapar en korrelationsmatris (Pearsons r som standard).
- `.iloc[0, 1]` väljer cellen på första raden, andra kolumnen.
- `:.2f` i en f-sträng visar två decimaler.

Pearsons r ligger mellan −1 och +1. Ett positivt värde betyder att högre värden i den ena variabeln tenderar att hänga samman med högre värden i den andra. **Korrelation är inte bevis för orsakssamband.** Videon visar ungefär `0,59` för lärarens dataset; din siffra kan bli annorlunda med annan data eller andra rensningsbeslut.

## 12. Exportera resultat som CSV

```python
df_clean.to_csv("resultat_stadad.csv", index=False)
summary_class.to_csv("summary_by_class.csv", index=False)
andelar.to_csv("summary_by_class_bins.csv", index=True)
```

- `to_csv("filnamn.csv")` skriver en ny CSV-fil.
- `index=False` tar inte med DataFrames radindex som extra kolumn.
- **Här behöver `andelar` behandlas annorlunda:** efter `crosstab` är klassnamnen ett **meningsfullt index**. Om du sparar `index=False` försvinner klassnamnen ur CSV-filen. Alternativt kan du göra `andelar.reset_index().to_csv(..., index=False)`.

Filerna hamnar i aktuell arbetskatalog. Kontrollera gärna med `Path.cwd()` om du inte hittar dem.

## 13. Komplett, körbart miniexempel utan kursfil

Det här är ett **eget övningsexempel** som illustrerar videons arbetsflöde. Det använder inte lärarens ursprungliga dataset.

```python
import pandas as pd

# Rådata med saknade värden, dubblett och varierande kolumnnamn
raw = pd.DataFrame({
    " Student ": ["A", "B", "C", "D", "D", "E"],
    "Klass": ["X", "X", "Y", "Y", "Y", "X"],
    "Studietimmar": [5, None, 10, 2, 2, 0],
    "Provpoäng": [80, 70, 95, 60, 60, None],
})

# 1. Standardisera namn
df = raw.copy()
df.columns = df.columns.str.strip().str.replace(" ", "_", regex=False).str.lower()

# 2. Behåll rader med provpoäng; kopiera för säker uppdatering
df_clean = df.dropna(subset=["provpoäng"]).copy()

# 3. Demonstration: fyll saknade timmar med noll
df_clean.loc[df_clean["studietimmar"].isna(), "studietimmar"] = 0

# 4. Ta bort exakta dubbletter
df_clean = df_clean.drop_duplicates().reset_index(drop=True)

# 5. Kategori
df_clean["klass"] = df_clean["klass"].astype("category")

# 6. Nytt mått utan division med noll
nämnare = df_clean["studietimmar"].where(df_clean["studietimmar"] > 0, pd.NA)
df_clean["poang_per_timme"] = df_clean["provpoäng"] / nämnare

# 7. Gruppsammanfattning
summary = (
    df_clean.groupby("klass", observed=True)
      .agg(antal=("student", "size"), medel_poang=("provpoäng", "mean"))
      .reset_index()
)

# 8. Kontroll
assert df_clean["provpoäng"].between(0, 100).all()
assert (df_clean["studietimmar"] >= 0).all()

print(df_clean)
print(summary)

# 9. Export
# df_clean.to_csv("resultat_stadad.csv", index=False)
# summary.to_csv("summary_by_class.csv", index=False)
```

**Förväntad slutsats:** fem av sex ursprungsrader har provpoäng, och en av dessa är en exakt dubblett. Därför återstår **fyra rader**. För klass X är medelpoängen `(80 + 70) / 2 = 75`; för klass Y `(95 + 60) / 2 = 77,5`. Eleven B får saknade `poang_per_timme`, eftersom timmarna sattes till noll och vi inte delar med noll.

## 14. Vanliga misstag

| Problem | Förklaring |
|---|---|
| `df_clean = df.dropna(...)` och sedan ändringar | Lägg gärna till `.copy()` innan tilldelningar |
| `fillna(0)` överallt | Saknat är inte automatiskt lika med noll |
| `drop_duplicates()` tar bort legitima observationer | Bestäm vad en dubblett **betyder** i datan |
| `astype("category")` på numeriska mått | Använd kategori för faktiska grupper/etiketter |
| `pd.to_datetime` körs på kolumn som saknas | Kontrollera först `df.columns` |
| `provpoäng / studietimmar` | Noll i nämnaren måste hanteras |
| `groupby(...).agg(...)` med fel parenteser | Named aggregation: `namn=("kolumn", "funktion")` |
| `pd.cut` missar 0 | Lägg till `include_lowest=True` när det behövs |
| `andelar.to_csv(index=False)` | Gruppetiketter kan försvinna när de ligger i index |
| `df.iloc(0, 1)` | `iloc` är indexerare: `df.iloc[0, 1]` |

## 15. Quick Reference

```python
import pandas as pd

df = pd.read_csv("fil.csv")
df.columns = df.columns.str.strip().str.replace(" ", "_", regex=False).str.lower()
df.info()
df.isna().sum()
df.dtypes

df_clean = df.dropna(subset=["provpoäng"]).copy()
df_clean.loc[df_clean["studietimmar"].isna(), "studietimmar"] = 0
df_clean = df_clean.drop_duplicates().reset_index(drop=True)
df_clean["klass"] = df_clean["klass"].astype("category")

summary = df_clean.groupby("klass", observed=True).agg(
    medel=("provpoäng", "mean"),
    median=("provpoäng", "median"),
    antal=("student", "size"),
).reset_index()

assert df_clean["provpoäng"].between(0, 100).all()
df_clean.to_csv("rensad.csv", index=False)
```

## 16. Övningar

1. Förklara med en mening vad en pipeline är.
2. Skriv kod som räknar saknade värden i varje kolumn i `df`.
3. Ta bort rader där `provpoäng` saknas, men behåll rader där bara `studietimmar` saknas.
4. Sätt saknade `studietimmar` till noll med `loc`.
5. Räkna exakta dubbletter och ta bort dem.
6. Skapa en kolumn `poang_per_timme` utan division med noll.
7. Sammanfatta medianpoängen per klass med `groupby`.
8. Skapa en tvärtabell med poängintervallen 0–60, 60–80 och 80–100, som visar andel per klass.
9. Varför kan `index=False` vara fel när man exporterar resultatet av `pd.crosstab`?
10. Förklara varför en saknad studietimme inte säkert betyder noll timmar.

### Facit

```python
# 2
print(df.isna().sum())

# 3
df_clean = df.dropna(subset=["provpoäng"]).copy()

# 4
df_clean.loc[df_clean["studietimmar"].isna(), "studietimmar"] = 0

# 5
print(df_clean.duplicated().sum())
df_clean = df_clean.drop_duplicates().reset_index(drop=True)

# 6
nämnare = df_clean["studietimmar"].where(df_clean["studietimmar"] > 0, pd.NA)
df_clean["poang_per_timme"] = df_clean["provpoäng"] / nämnare

# 7
median_per_klass = df_clean.groupby("klass", observed=True)["provpoäng"].median()

# 8
bins = [0, 60, 80, 100]
andelar = pd.crosstab(
    df_clean["klass"],
    pd.cut(df_clean["provpoäng"], bins=bins, include_lowest=True),
    normalize="index"
)
```

**1:** En pipeline är en bestämd följd av steg som omvandlar rådata till ett användbart resultat.  
**9:** Gruppnamnen ligger ofta i index; `index=False` kan därför ta bort informationen om vilken klass varje rad gäller.  
**10:** Ett saknat värde kan bero på utebliven rapportering, inte på att studenten faktiskt studerade noll timmar.

---

**Nästa lektion:** `05_matplotlib.md` – visualisera och kommunicera resultaten.
