# 08 – Mini-EDA med Python: bibliotekets utlåning

**Vecka 5 · Dataanalys · Lektion 08**  
**Utgångspunkt:** Lärarens genomgång om en mini-EDA för ett stadsbibliotek (cirka 2 timmar 35 minuter).

## 1. Vad ska vi lära oss?

**EDA** betyder *Exploratory Data Analysis*, på svenska **explorativ dataanalys**. Vi undersöker ett dataset för att upptäcka mönster, kontrollera datakvalitet och besvara frågor. I videon vill stadsbiblioteket veta:

- **Vad** lånas mest? Vilka genrer är vanligast?
- **Var** lånas mest? Vilka filialer toppar listan?
- **När** sker utlåningarna? Hur varierar de över tid?
- Hur ser en **typisk utlåning** ut, och hur vanligt är det med försenade lån?
- Vilka är **topp tre** bland filialer och genrer?

Målet är inte bara att rita diagram. Målet är ett **beslutsunderlag** som andra kan förstå.

## 2. Arbetsflödet – från rådata till rapport

```text
CSV-fil
   ↓
Läs in och inspektera
   ↓
Kontrollera kolumner och datatyper
   ↓
Rensa och skapa hjälpkolumner
   ↓
Beräkna nyckeltal och gruppera data
   ↓
Välj lämpliga diagram
   ↓
Tolka och sammanfatta i rapport
   ↓
Spara arbetet med Git/GitHub
```

**Viktigt:** En figur är inte en slutsats. Vi måste förklara vad den visar och vad den *inte* visar.

## 3. Projektstruktur

Läraren skapar ett GitHub-repository, klonar det, använder en virtuell miljö och skiljer på data, Pythonmoduler och rapporten.

```text
eda_biblioteket/
├── .gitignore
├── .venv/                  # ska INTE läggas på GitHub
├── data/
│   └── library_loans.csv
├── src/
│   ├── __init__.py
│   ├── io_utils.py
│   ├── metrics.py
│   └── plots.py
├── report.ipynb
├── requirements.txt
└── README.md
```

**Förklaring:** `src` betyder *source* (källkod). `io_utils.py` sköter inläsning, `metrics.py` beräknar mått, `plots.py` ritar diagram och `report.ipynb` visar analysen. Namnen ovan är ett pedagogiskt, konsekvent exempel; lärarens transkribering återger vissa filnamn otydligt.

## 4. Skapa miljön i Windows PowerShell

Öppna projektmappen i VS Code och kör:

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install pandas matplotlib numpy ipykernel jupyter
python -m pip freeze > requirements.txt
```

Om du redan har skapat `.venv` i VS Code behöver du **inte** skapa den igen. Välj sedan projektets `.venv` som **Python interpreter** och **Jupyter kernel**.

| Del | Betydelse |
|---|---|
| `python` | Kör den valda Python-tolken |
| `-m pip` | Kör `pip` via just den tolken |
| `install` | Installerar paket |
| `freeze` | Skriver ut installerade paket och versioner |
| `>` | Sparar terminalens utdata i en fil |
| `requirements.txt` | Gör det lättare att återskapa miljön |

En annan person kan installera samma beroenden med:

```powershell
python -m pip install -r requirements.txt
```

**Viktigt:** `python -m pip` använder den Python som kommandot `python` pekar på. Kontrollera därför att rätt `.venv` är aktiv. `pip freeze` tar med indirekta beroenden också. En Pythonversion bör dessutom dokumenteras i README, eftersom `requirements.txt` inte väljer Pythonversion.

## 5. Git – spara projektet i etapper

```powershell
git status
git add .
git commit -m "Add data loading and validation"
git push
```

- `git status` visar ändringar.
- `git add .` förbereder ändringarna för commit.
- `git commit` skapar en sparad version lokalt.
- `git push` skickar commits till fjärrrepositoryt.

Gör gärna en commit efter en fungerande del, till exempel inläsning, nyckeltal eller visualisering. Lägg aldrig in lösenord eller känsliga personuppgifter i ett publikt repository. Kontrollera också att CSV-filen får delas.

## 6. Läs in och undersök CSV-filen

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

df = pd.read_csv(
    "data/library_loans.csv",
    parse_dates=["checkout_date", "returned_date"]
)

print(df.head(3))
df.info()
print(df.isna().sum())
```

**Förklaring rad för rad:**

- `pd.read_csv(...)` läser CSV-filen till en **DataFrame** (en tabell i Pandas).
- `parse_dates=[...]` försöker omvandla angivna textkolumner till datum.
- `head(3)` visar de tre första raderna.
- `info()` visar kolumner, antal ifyllda värden och datatyper.
- `isna().sum()` räknar saknade värden per kolumn.

**OBS:** Kolumnnamnen måste matcha CSV-filen exakt. I videon förekommer bland annat `loan_id`, `checkout_date`, `branch`, `genre`, `item_type`, `patron_age_group`, `loan_days`, `returned_date`, `overdue_days` och `fine_amount`.

## 7. Lägg inläsningen i en återanvändbar funktion

Skapa `src/io_utils.py`:

```python
import pandas as pd

REQUIRED_COLUMNS = [
    "loan_id", "checkout_date", "branch", "genre",
    "item_type", "patron_age_group", "loan_days",
    "returned_date", "overdue_days", "fine_amount"
]


def load_data(path: str) -> pd.DataFrame:
    """Läser en CSV-fil, tolkar datum och kontrollerar kolumner."""
    df = pd.read_csv(
        path,
        parse_dates=["checkout_date", "returned_date"]
    )

    missing = [col for col in REQUIRED_COLUMNS if col not in df.columns]
    if missing:
        raise ValueError(f"Saknade kolumner: {missing}")

    return df
```

### Vad betyder syntaxen?

- `def` skapar en funktion.
- `path` är en **parameter**; filvägen du skickar in är ett **argument**.
- `path: str` är en **type hint**: vi förväntar oss en textsträng.
- `-> pd.DataFrame` anger vilken typ funktionen avses returnera.
- `"""..."""` är en **docstring**: hjälptext för funktionen.
- `REQUIRED_COLUMNS` är skrivet med stora bokstäver för att signalera en konstant enligt namnkonvention.
- `[col for col in ... if ...]` är en **list comprehension**: bygg en lista med de kolumner som saknas.
- `raise ValueError(...)` avbryter med ett tydligt felmeddelande.
- `return df` skickar tillbaka tabellen.

**Att förstå:** En type hint tvingar inte Python att kontrollera typen automatiskt. Det är information för läsare och utvecklingsverktyg.

## 8. Gör numeriska kolumner numeriska

Fortsätt i `src/io_utils.py`:

```python
def coerce_numeric(df: pd.DataFrame) -> pd.DataFrame:
    """Konverterar relevanta kolumner till numeriska värden."""
    out = df.copy()

    for col in ["loan_days", "overdue_days", "fine_amount"]:
        out[col] = pd.to_numeric(out[col], errors="coerce")

    return out
```

`df.copy()` skapar en separat tabell så att vi inte råkar ändra indata direkt. `for col in [...]` upprepar samma operation för tre kolumner. `errors="coerce"` gör värden som inte kan tolkas som tal till `NaN` (saknat värde).

**Skillnad:** `errors="raise"` skulle i stället ge ett fel när ett ogiltigt värde påträffas.

## 9. Rensa och skapa hjälpkolumner

```python
from src.io_utils import load_data, coerce_numeric

df = coerce_numeric(load_data("data/library_loans.csv"))

# Saknade eller ogiltiga lånetider används inte i denna analys.
df = df.dropna(subset=["loan_days"])
df = df.loc[df["loan_days"] > 0].copy()

# Skapa månads- och veckokolumner från utlåningsdatumet.
df["month"] = df["checkout_date"].dt.to_period("M").dt.to_timestamp()
df["week"] = df["checkout_date"].dt.to_period("W").dt.start_time
```

- `dropna(subset=[...])` tar bort rader där **just den kolumnen** saknar värde.
- `df["loan_days"] > 0` ger en **boolesk mask** med `True` eller `False` för varje rad.
- `.loc[...]` väljer rader där masken är `True`.
- `.dt` ger åtkomst till datumfunktioner i Pandas.
- `.to_period("M")` grupperar datum efter månad.
- `.dt.start_time` ger veckans startdatum.

**Viktig nyansering:** Läraren väljer bort lån med `loan_days == 0`. Det är ett **analysbeslut**, inte en universell regel. Kontrollera först om noll kan vara ett giltigt värde i ditt dataset. I genomgången minskade antalet rader från 128 till 124 efter filtreringen.

## 10. Nyckeltal – vad är typiskt?

Skapa `src/metrics.py`:

```python
import pandas as pd


def total_loans(df: pd.DataFrame) -> int:
    return int(df["loan_id"].nunique())


def average_loan_days(df: pd.DataFrame) -> float:
    return float(df["loan_days"].mean())


def overdue_rate(df: pd.DataFrame) -> float:
    return float((df["overdue_days"] > 0).mean())
```

### Förstå uttrycken

`nunique()` räknar **unika värden**. Det skiljer sig från `len(df)`, som räknar rader. Om samma låne-ID förekommer flera gånger blir resultaten olika.

`mean()` räknar **medelvärde**.

Uttrycket `df["overdue_days"] > 0` ger `True` för sena lån och `False` för övriga. När vi tar `.mean()` på dessa booleska värden räknas `True` som 1 och `False` som 0. Resultatet blir **andelen** försenade rader. Detta är bara samma sak som andelen unika lån om varje lån representeras av en rad.

```python
from src import metrics as m

print(f"Unika lån: {m.total_loans(df)}")
print(f"Genomsnittlig lånetid: {m.average_loan_days(df):.1f} dagar")
print(f"Försenade lån: {m.overdue_rate(df):.1%}")
```

**Formatering:** `:.1f` ger en decimal. `:.1%` multiplicerar visningsvärdet med 100 och lägger till procenttecken. Du ska alltså **inte** multiplicera med 100 igen om du använder `%`-formatet.

## 11. Vad lånas mest? `groupby()`

```python
def loans_by_genre(df: pd.DataFrame) -> pd.DataFrame:
    return (
        df.groupby("genre", dropna=False)["loan_id"]
          .nunique()
          .sort_values(ascending=False)
          .rename("loans")
          .reset_index()
    )

by_genre = loans_by_genre(df)
print(by_genre.head(3))
```

- `groupby("genre")`: dela upp raderna i grupper efter genre.
- `dropna=False`: låt även saknad genre vara en grupp.
- `["loan_id"].nunique()`: räkna unika låne-ID i varje grupp.
- `sort_values(ascending=False)`: flest först.
- `rename("loans")`: döp resultatkolumnen.
- `reset_index()`: gör gruppnamnet till vanlig kolumn igen.
- `head(3)`: visa **topp tre**.

## 12. Var lånas mest? Samma idé, annan kolumn

```python
def loans_by_branch(df: pd.DataFrame) -> pd.DataFrame:
    return (
        df.groupby("branch", dropna=False)["loan_id"]
          .nunique()
          .sort_values(ascending=False)
          .rename("loans")
          .reset_index()
    )

by_branch = loans_by_branch(df)
print(by_branch.head(3))
```

Här ser du fördelen med att känna igen ett **mönster**: samma kod kan användas för en ny analysfråga genom att byta grupperingskolumn.

## 13. När lånas böcker? `resample()`

```python
def loans_over_time(df: pd.DataFrame, frequency: str = "MS") -> pd.DataFrame:
    return (
        df.set_index("checkout_date")
          .sort_index()
          .resample(frequency)["loan_id"]
          .nunique()
          .rename("loans")
          .reset_index()
    )

monthly = loans_over_time(df, "MS")
weekly = loans_over_time(df, "W")
```

**Varför `set_index()`?** `resample()` grupperar data efter tidsintervall och arbetar normalt mot ett datumindex. `sort_index()` sorterar kronologiskt.

`"MS"` betyder **månadens början** (*Month Start*). `"W"` ger veckovisa intervall. `nunique()` räknar unika lån i varje period.

**Obs:** Olika tidsfrekvenser kan ge olika datumetiketter, exempelvis veckoslut för `"W"`. Tolka därför x-axeln noggrant.

## 14. Visualisera resultatet – välj efter frågan

### Stapeldiagram: jämför filialer

```python
fig, ax = plt.subplots(figsize=(9, 5))
ax.bar(by_branch["branch"], by_branch["loans"])
ax.set(title="Utlåningar per filial", xlabel="Filial", ylabel="Antal unika lån")
ax.tick_params(axis="x", rotation=30)
ax.grid(axis="y", alpha=0.3)
plt.tight_layout()
plt.show()
```

**Varför staplar?** Filial är en **kategori** och antal lån är ett **numeriskt mått**. Vi vill jämföra kategorier.

### Linjediagram: förändring över tid

```python
fig, ax = plt.subplots(figsize=(9, 5))
ax.plot(monthly["checkout_date"], monthly["loans"], marker="o")
ax.set(title="Utlåningar per månad", xlabel="Månad", ylabel="Antal unika lån")
ax.grid(alpha=0.3)
plt.tight_layout()
plt.show()
```

**Varför linje?** Månader följer en tidsordning och vi vill se hur värdet förändras.

### Histogram: hur är lånetiderna fördelade?

```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.hist(df["loan_days"].dropna(), bins=10)
ax.set(title="Fördelning av lånetider", xlabel="Lånedagar", ylabel="Antal lån")
plt.show()
```

**Varför histogram?** Vi undersöker **en numerisk variabels fördelning**, inte en jämförelse mellan namngivna kategorier.

### Boxplot: median, spridning och avvikare

```python
fig, ax = plt.subplots(figsize=(8, 3))
ax.boxplot(df["loan_days"].dropna(), vert=False)
ax.set(title="Spridning i lånetider", xlabel="Lånedagar")
plt.show()
```

Boxens mittlinje är medianen. Boxens kanter markerar första och tredje kvartilen. Punkter utanför *whiskers* kan markera avvikande värden enligt diagrammets regel – de är **inte automatiskt felaktiga observationer**.

## 15. En enkel generell diagramfunktion

I genomgången arbetar läraren med återanvändbara hjälpfunktioner för diagram. En enkel variant:

```python
import matplotlib.pyplot as plt


def plot_category_counts(summary, category_col, title):
    fig, ax = plt.subplots(figsize=(9, 5))
    ax.bar(summary[category_col], summary["loans"])
    ax.set_title(title)
    ax.set_ylabel("Antal unika lån")
    ax.tick_params(axis="x", rotation=30)
    plt.tight_layout()
    return fig, ax

fig, ax = plot_category_counts(by_genre, "genre", "Utlåningar per genre")
plt.show()
```

En funktion är värdefull när vi upprepar **samma diagramlogik** flera gånger. Den behöver inte göras komplicerad för att vara användbar.

## 16. Pivottabell – jämför två dimensioner

Mot slutet av genomgången bygger läraren en rapportmatris med **pivot table**.

```python
pivot = pd.pivot_table(
    df,
    index="branch",
    columns="genre",
    values="loan_id",
    aggfunc="nunique",
    fill_value=0
)

print(pivot)
```

- `index="branch"`: **raderna** representerar filialer.
- `columns="genre"`: **kolumnerna** representerar genrer.
- `values="loan_id"`: värdena vi räknar på.
- `aggfunc="nunique"`: räkna unika låne-ID.
- `fill_value=0`: fyll tomma kombinationer med 0.

En pivottabell kan exempelvis visa att filial A har flest deckarlån medan filial B har flest barnbokslån. **Siffrorna måste komma från din faktiska data**; dra inte slutsatsen i förväg.

Om du vill spara tabellen:

```python
pivot.to_csv("data/loans_by_branch_and_genre.csv")
```

## 17. Skriv en begriplig rapport

Läraren avslutar med en `README.md` som sammanfattar projektet, resultaten och hur det körs. En tydlig rapport kan ha följande struktur:

```markdown
# Mini-EDA – Stadsbibliotekets utlåning

## Uppdrag
Vilka genrer och filialer har flest lån? Hur varierar utlåningen över tid?

## Datakälla och metod
Beskriv CSV-filen, urvalet, borttagna rader och definitionen av "lån".

## Nyckeltal
Ange antal unika lån, genomsnittlig lånetid och andel försenade lån.

## Resultat och visualiseringar
Visa topplistor, tidsserie, histogram, boxplot och pivottabell.

## Slutsatser
Beskriv vad siffrorna stödjer och vilka begränsningar som finns.

## Så kör du projektet
Beskriv Pythonversion, installation och hur notebooken startas.
```

**Bra slutsats:** ”Filial X hade flest registrerade lån i det här datasetet.”  
**För långtgående slutsats:** ”Filial X är stadens mest uppskattade bibliotek.” Antalet lån räcker inte för att slå fast uppskattning.

## 18. Viktiga förtydliganden till videon

1. **`mean()` på booleska värden fungerar**, eftersom `True` räknas som 1 och `False` som 0.
2. **`nunique()` och antal rader är olika mått.** Kontrollera om ett lån kan förekomma på flera rader.
3. **Rensa inte automatiskt bort alla nollor.** Det måste motiveras av frågan och datans betydelse.
4. **En förseningsandel är inte en orsak.** Vi kan se hur vanligt det är, men inte varför lån är försenade.
5. **Datum måste vara riktiga datumtyper** för att `.dt` och tidsgruppering ska fungera.
6. **CSV-filen saknas i transkriberingen.** Kodexemplen visar därför arbetssättet; de verkliga tabellerna och siffrorna kan inte verifieras utan datan.
7. **Jupyter kan behålla gamla importer i minnet.** Efter ändringar i `.py`-filer kan du behöva starta om kerneln och välja **Run All**.
8. **En README ska vara begriplig även utan notebooken.** Skriv kortfattat vad resultaten betyder.

## 19. Quick Reference

| Kod | Gör |
|---|---|
| `pd.read_csv(path)` | Läser CSV till DataFrame |
| `df.head(3)` | Visar tre första raderna |
| `df.info()` | Visar datatyper och ifyllda värden |
| `df.isna().sum()` | Räknar saknade värden |
| `df.copy()` | Skapar en kopia |
| `pd.to_numeric(..., errors="coerce")` | Gör om till tal; ogiltiga värden blir `NaN` |
| `df.dropna(subset=[...])` | Tar bort rader med saknade värden i valda kolumner |
| `df.loc[mask]` | Filtrerar rader med en boolesk mask |
| `df["loan_id"].nunique()` | Räknar unika låne-ID |
| `df["loan_days"].mean()` | Räknar medellånetid |
| `df.groupby("genre")` | Grupperar efter genre |
| `.sort_values(ascending=False)` | Sorterar störst först |
| `.head(3)` | Tar topp tre |
| `.set_index("checkout_date")` | Använder datum som index |
| `.resample("MS")` | Grupperar månadsvis |
| `pd.pivot_table(...)` | Bygger sammanfattning i två dimensioner |
| `fig, ax = plt.subplots()` | Skapar figur och diagramyta |
| `ax.bar(...)` | Stapeldiagram |
| `ax.plot(...)` | Linjediagram |
| `ax.hist(...)` | Histogram |
| `ax.boxplot(...)` | Låddiagram |
| `plt.show()` | Visar diagrammet |

## 20. Övningar

Använd en egen liten DataFrame om du inte har bibliotekets CSV-fil:

```python
import pandas as pd

sample = pd.DataFrame({
    "loan_id": [1, 2, 3, 4, 5, 6],
    "branch": ["Centrum", "Norr", "Centrum", "Norr", "Centrum", "Söder"],
    "genre": ["Deckare", "Barn", "Barn", "Deckare", "Deckare", "Barn"],
    "checkout_date": pd.to_datetime([
        "2026-01-05", "2026-01-12", "2026-02-01",
        "2026-02-14", "2026-02-20", "2026-03-03"
    ]),
    "loan_days": [14, 21, 7, 28, 14, 21],
    "overdue_days": [0, 2, 0, 0, 5, 0]
})
```

1. Visa de tre första raderna.
2. Räkna hur många unika lån det finns.
3. Beräkna genomsnittlig lånetid.
4. Räkna andelen lån med `overdue_days > 0`.
5. Ta fram antal lån per filial, sorterat störst först.
6. Ta fram topp två genrer.
7. Räkna lån per månad.
8. Skapa en pivottabell med filial som rader och genre som kolumner.

## 21. Facit

```python
# 1
print(sample.head(3))

# 2
print(sample["loan_id"].nunique())  # 6

# 3
print(sample["loan_days"].mean())  # 17.5

# 4
print((sample["overdue_days"] > 0).mean())  # 0.3333... = 33,3 %

# 5
print(sample.groupby("branch")["loan_id"].nunique().sort_values(ascending=False))

# 6
print(sample.groupby("genre")["loan_id"].nunique().sort_values(ascending=False).head(2))

# 7
print(
    sample.set_index("checkout_date")
          .resample("MS")["loan_id"]
          .nunique()
)

# 8
print(pd.pivot_table(
    sample,
    index="branch",
    columns="genre",
    values="loan_id",
    aggfunc="nunique",
    fill_value=0
))
```

**Kontrollera att du förstår:** Varför är `nunique()` ett lämpligt val för lån? Varför används `resample()` för månader men `groupby()` för filialer? Varför behöver en rapport både siffror och tolkningar?

---

**Spara i GitHub:** `data_science/data_analysis/08_mini_eda_python.md`  
**Nästa steg:** Gå tillbaka till de tidigare reserverade Pythonlektionerna (14, 17 och 18) och genomgången om textfiler.
