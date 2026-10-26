# Lektion 05 – Introduktion till Matplotlib

**Kurs:** Vecka 4 – Deskriptiv statistik och bibliotek för dataanalys  
**Mapp:** `data_science/data_analysis/`  
**Underlag:** Genomgång 4.5 – Introduktion till Matplotlib (cirka 27 minuter)

## Vad ska du lära dig?

Efter lektionen ska du kunna:

1. Förklara varför vi visualiserar data och vad Matplotlib gör.
2. Skilja på en **figur** (`fig`) och en **axel/ritarea** (`ax`).
3. Skapa linjediagram, histogram, spridningsdiagram och stapeldiagram.
4. Förstå vad x- och y-värden, `bins`, `alpha`, etiketter och stödlinjer betyder.
5. Läsa in CSV-filer med Pandas och rita diagram från DataFrames.
6. Spara diagram som bildfiler och tolka vad diagrammen faktiskt visar.

> **Viktig tanke:** Pandas hjälper oss att läsa, rensa och sammanfatta data. Matplotlib hjälper oss att **se mönster** i samma data.

---

## 1. Vad är Matplotlib?

**Matplotlib** är ett Pythonbibliotek för datavisualisering. Vi kan använda det för att göra grafer och diagram av exempelvis provresultat, försäljning och studietimmar.

Vi använder oftast delen `pyplot`, som importeras med namnet `plt`:

```python
import matplotlib.pyplot as plt
```

**Rad för rad:**

- `import` betyder att Python laddar in ett bibliotek.
- `matplotlib.pyplot` är den del som innehåller funktioner för diagram.
- `as plt` ger biblioteket ett kortare namn. `plt` är ett vanligt namn, inte ett krav.

När vi arbetar med tabeller behöver vi även Pandas:

```python
import pandas as pd
import matplotlib.pyplot as plt
```

Om Matplotlib inte är installerat i din aktiva virtuella miljö kan du köra följande i terminalen:

```powershell
python -m pip install matplotlib
```

I en Jupyter Notebook väljer du rätt Python-kernel. Då kan du ofta visa diagram direkt under kodcellen.

---

## 2. Från Pandas till diagram

I genomgången återanvänder läraren två CSV-filer från lektion 04:

- En fil med **rensade studentdata**, inklusive studietimmar och provpoäng.
- En fil med en **sammanfattning per klass**, inklusive medelpoäng.

Exakta filnamn beror på hur du sparade dem i föregående lektion. Här använder vi exempel:

```python
import pandas as pd

# Byt filnamnen om dina exporter heter något annat.
df = pd.read_csv("resultat_stadad.csv")
sam_class = pd.read_csv("summary_by_class.csv")

print(df.head())
print(sam_class.head())
```

`pd.read_csv(...)` läser en CSV-fil och returnerar en **DataFrame** (en tabell med rader och kolumner).

`df.head()` visar de första fem raderna. Det hjälper oss att kontrollera att filen lästes in och att kolumnnamnen är rätt. **Kolumnnamnen i kodexemplen nedan är `studietimmar`, `provpoang`, `klass` och `medel_poang`.** Anpassa dem till dina faktiska CSV-filer.

### Vill du prova utan CSV-filer?

Då kan du skapa små exempeltabeller direkt i Python:

```python
import pandas as pd
import matplotlib.pyplot as plt

# Exempeldata – inte de verkliga resultaten från lärarens video.
df = pd.DataFrame({
    "klass": ["A", "A", "B", "B", "C", "C", "A", "B"],
    "studietimmar": [2, 5, 1, 4, 0, 3, 6, 2],
    "provpoang": [65, 88, 55, 76, 72, 69, 95, 61],
})

sam_class = (
    df.groupby("klass", as_index=False)
      .agg(medel_poang=("provpoang", "mean"))
)

print(sam_class)
```

**Exempelutskrift:**

```text
  klass  medel_poang
0     A    82.666667
1     B    64.000000
2     C    70.500000
```

Det här testmaterialet kan användas för samtliga diagram nedan.

---

## 3. Den viktigaste grunden: `fig` och `ax`

Läraren börjar med följande mönster:

```python
fig, ax = plt.subplots()
```

Det betyder att `plt.subplots()` skapar och lämnar tillbaka **två objekt**:

| Namn | Betydelse | Tänk så här |
|---|---|---|
| `fig` | Figuren, alltså hela bilden | Pappret som diagrammet ligger på |
| `ax` | En ritarea med koordinataxlar | Platsen på pappret där vi ritar |

Kommat i `fig, ax = ...` är **uppackning**: funktionen ger oss två saker och vi sparar dem i två variabler.

> **Vanlig språkförvirring:** `ax` står för ett `Axes`-objekt, trots att namnet låter som singularformen `axis`. Ett `Axes`-objekt är hela ritområdet och innehåller vanligtvis både x- och y-axel.

Vi kan skapa en figur och rita en linje:

```python
fig, ax = plt.subplots()

ax.plot([0, 1, 2], [1, 2, 1])
ax.set_title("Mitt första diagram")

plt.show()
```

### Vad händer rad för rad?

1. `fig, ax = plt.subplots()` skapar bilden och ritområdet.
2. `ax.plot(...)` ritar ett **linjediagram**.
3. Första listan `[0, 1, 2]` är x-värdena.
4. Andra listan `[1, 2, 1]` är y-värdena.
5. `ax.set_title(...)` skriver en rubrik ovanför diagrammet.
6. `plt.show()` visar figuren. I Jupyter visas den ofta även automatiskt, men `plt.show()` gör avsikten tydlig.

Punkterna blir `(0, 1)`, `(1, 2)` och `(2, 1)`. Matplotlib förbinder dem med en linje.

### Kan man bara ange en lista?

Ja:

```python
ax.plot([1, 2, 1])
```

Då tolkar Matplotlib listan som **y-värden** och använder automatiskt x-värdena `0, 1, 2`.

**Men:** Om x-värdena betyder faktiska årtal, timmar eller datum är det bättre att ange dem uttryckligen.

---

## 4. Rubrik, axeltexter, rutnät och marginaler

```python
fig, ax = plt.subplots()
ax.plot([0, 1, 2], [1, 2, 1])

ax.set_title("Exempel på linjediagram")
ax.set_xlabel("X-värde")
ax.set_ylabel("Y-värde")
ax.grid(True)

plt.tight_layout()
plt.show()
```

| Kod | Gör detta |
|---|---|
| `set_title("...")` | Sätter diagrammets rubrik |
| `set_xlabel("...")` | Sätter text under x-axeln |
| `set_ylabel("...")` | Sätter text bredvid y-axeln |
| `grid(True)` | Visar stödlinjer/rutnät |
| `grid(True, axis="y")` | Visar bara vågräta stödlinjer |
| `plt.tight_layout()` | Justerar marginaler så att text får plats |
| `plt.show()` | Visar diagrammet |

**Kom ihåg:** `True` är ett booleskt värde. Här betyder det att rutnätet ska visas.

---

## 5. Histogram: hur är värdena fördelade?

Ett **histogram** visar hur många observationer som hamnar i olika **numeriska intervall**.

Fråga: *Ligger de flesta provpoängen runt 70–80, eller är de jämnt utspridda?*

```python
fig, ax = plt.subplots()

ax.hist(df["provpoang"].dropna(), bins=10)
ax.set_title("Fördelning av provpoäng")
ax.set_xlabel("Provpoäng")
ax.set_ylabel("Antal studenter")
ax.grid(True, axis="y")

plt.tight_layout()
plt.show()
```

### Rad för rad: `ax.hist(...)`

- `df["provpoang"]` väljer kolumnen med provpoäng.
- `.dropna()` tar bort saknade värden **ur den serie som skickas till diagrammet**. Det raderar inte rader från `df`.
- `ax.hist(...)` räknar hur många värden som ligger i olika intervall och ritar staplar.
- `bins=10` ber Matplotlib dela upp värdena i **tio fack**.

### Vad betyder `bins`?

Föreställ dig att du sorterar provpoäng i lådor. Varje låda täcker ett intervall, exempelvis 60–70 eller 70–80. Histogrammets stapelhöjd visar hur många observationer som hamnade i lådan.

- `bins=5`: färre och bredare intervall; ger en grövre överblick.
- `bins=10`: fler intervall; visar lite mer detaljer.
- `bins=50`: många smala intervall; kan se ryckigt ut om datamängden är liten.

Det finns inget universellt bästa antal. Välj utifrån datamängden och vad du vill undersöka.

### Egna intervall

Läraren visar också att vi kan bestämma gränserna själva:

```python
fig, ax = plt.subplots()
ax.hist(df["provpoang"].dropna(), bins=[0, 60, 80, 100])
ax.set_xlabel("Provpoäng")
ax.set_ylabel("Antal studenter")
plt.show()
```

Nu blir intervallen **0–60**, **60–80** och **80–100**. De är **inte lika breda**. Det är tillåtet, men jämförelser av stapelhöjder kan då bli missvisande eftersom bredare intervall kan samla fler observationer. Vid olika intervallbredder kan man behöva använda densitet eller välja lika breda intervall.

> **Skillnad mot `pd.cut(...)` i lektion 04:** `pd.cut` delar in värden i kategorier/intervall i en DataFrame. `ax.hist` både räknar fördelningen och ritar ett diagram. Exakta gränsregler kan skilja sig åt; anta inte automatiskt att värden precis på gränsen behandlas identiskt.

---

## 6. Spara diagram som PNG

```python
fig, ax = plt.subplots()
ax.hist(df["provpoang"], bins=10)
ax.set_title("Fördelning av provpoäng")

fig.savefig("hist_provpoang.png", dpi=150, bbox_inches="tight")
plt.show()
```

| Del | Betydelse |
|---|---|
| `fig.savefig(...)` | Sparar just den här figuren som en fil |
| `"hist_provpoang.png"` | Filnamn och filformat |
| `dpi=150` | Upplösning i punkter per tum |
| `bbox_inches="tight"` | Minskar risken att etiketter kapas i den sparade bilden |

Läraren använder `plt.savefig(...)`, vilket också fungerar. `fig.savefig(...)` gör det extra tydligt **vilken figur** som sparas.

Filen sparas normalt i programmets aktuella arbetskatalog. Om du inte hittar den kan du kontrollera med:

```python
from pathlib import Path
print(Path.cwd())
```

**Tips:** Spara figuren **innan** du stänger den. I vanliga Python-skript är `plt.show()` ofta sist.

---

## 7. Spridningsdiagram: finns ett samband?

Ett **spridningsdiagram** (scatter plot) visar **två numeriska variabler** samtidigt. Varje punkt motsvarar en observation.

Här undersöker vi relationen mellan studietimmar och provpoäng:

```python
fig, ax = plt.subplots()

ax.scatter(
    df["studietimmar"],
    df["provpoang"],
    alpha=0.6
)

ax.set_title("Studietimmar jämfört med provpoäng")
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
ax.grid(True)

plt.tight_layout()
plt.show()
```

### Vad betyder argumenten?

- **Första argumentet:** `df["studietimmar"]` = x-värdena, längs den vågräta axeln.
- **Andra argumentet:** `df["provpoang"]` = y-värdena, längs den lodräta axeln.
- **`alpha=0.6`:** punkternas genomskinlighet. `0` är helt genomskinligt och `1` helt ogenomskinligt. Det gör överlappande punkter lättare att se.

Om punkterna generellt lutar uppåt åt höger kan det tyda på ett **positivt samband**: högre värden på x hänger ofta ihop med högre värden på y.

Om punkterna lutar nedåt åt höger kan det tyda på ett **negativt samband**. Om de ligger utspridda utan tydligt mönster kan sambandet vara svagt eller saknas.

### Viktig tolkning: samband är inte orsak

I lärarens exempel ser sambandet mellan studietimmar och provpoäng positivt ut. Det betyder **inte automatiskt** att extra studietimmar *orsakade* högre provpoäng. Andra faktorer kan påverka båda variablerna, exempelvis förkunskaper eller hur studietiden mättes.

**Extra viktigt i just detta dataset:** I föregående lektion ersattes vissa saknade studietimmar med `0`. En sådan nolla behöver **inte** betyda att studenten faktiskt studerade noll timmar. Punkter vid x = 0 kan därför delvis bero på datarensningen. Det är ett skäl att dokumentera sina antaganden.

---

## 8. Stapeldiagram: jämför grupper

Ett **stapeldiagram** (bar chart) passar när du vill jämföra kategorier, till exempel klass A, B och C.

```python
fig, ax = plt.subplots()

ax.bar(sam_class["klass"], sam_class["medel_poang"])
ax.set_title("Medelpoäng per klass")
ax.set_xlabel("Klass")
ax.set_ylabel("Medelpoäng")
ax.grid(True, axis="y")

plt.tight_layout()
plt.show()
```

Här kommer staplarnas:

- **x-positioner** från kolumnen `klass`.
- **höjder** från kolumnen `medel_poang`.

Kom ihåg skillnaden:

| Histogram (`hist`) | Stapeldiagram (`bar`) |
|---|---|
| Visar fördelningen av en numerisk variabel | Jämför grupper/kategorier |
| Intervallen ligger längs x-axeln | Kategorierna ligger längs x-axeln |
| Räknar vanligen observationer per intervall | Visar värden du anger, t.ex. medelpoäng |
| Exempel: hur många fick 60–70 poäng? | Exempel: vilken klass hade högst medelpoäng? |

**Viktigt:** En högre medelpoäng betyder inte nödvändigtvis att en klass är bättre i alla avseenden. Titta också på antal elever, spridning och om skillnaderna kan förklaras av andra faktorer.

---

## 9. Om etiketterna krockar: rotera dem

Om du har många kategorier eller långa namn kan x-axelns texter överlappa.

```python
fig, ax = plt.subplots()
ax.bar(sam_class["klass"], sam_class["medel_poang"])

plt.setp(ax.get_xticklabels(), rotation=45, ha="right")
plt.tight_layout()
plt.show()
```

Förklaring:

- `ax.get_xticklabels()` hämtar textetiketterna på x-axeln.
- `plt.setp(...)` ändrar egenskaper för dessa etiketter.
- `rotation=45` vrider dem 45 grader.
- `ha="right"` högerjusterar texten, vilket ofta ger bättre placering.

I nyare Matplotlib kan du också använda `ax.tick_params(axis="x", labelrotation=45)` för rotation.

---

## 10. Hur tolkar vi lärarens tre diagram?

### Histogrammet – provpoäng

I videon beskriver läraren en koncentration kring ungefär **75–80 poäng**, med färre mycket låga eller mycket höga resultat. Det är en **beskrivning av just det diagrammet**, inte en regel som gäller all data.

**Vi kan säga:** "Många observationer ligger i mitten eller den övre delen av poängskalan."

**Vi kan inte säkert säga:** "Provet var lagom svårt" enbart från histogrammet. Det kräver ytterligare information om provets syfte, svårighetsgrad och deltagarna.

### Spridningsdiagrammet – studietimmar och provpoäng

I videon beskriver läraren ett uppåtgående punktmoln och en Pearson-korrelation omkring **0,5–0,6**. Det tyder på ett positivt linjärt samband i det aktuella materialet.

**Vi kan säga:** "Högre studietimmar tenderar att sammanfalla med högre provpoäng i dessa data."

**Vi ska inte säga:** "Fler timmar ger automatiskt högre poäng." Det skulle vara ett orsakspåstående som diagrammet ensamt inte visar.

### Stapeldiagrammet – medelpoäng per klass

Läraren beskriver att klass **A har högst medelpoäng**, B ligger lägre och C lägst i exemplet. Det är en **deskriptiv jämförelse**.

För att förstå skillnaderna bättre skulle vi kunna undersöka hur många elever som finns i varje klass, spridningen inom klasserna och deras studietimmar. Videon genomför inte något statistiskt signifikanstest.

---

## 11. Så väljer du rätt diagram

| Din fråga | Välj | Varför? |
|---|---|---|
| Hur är provpoängen fördelade? | Histogram | Visar hur många värden som hamnar i olika intervall |
| Finns ett samband mellan timmar och poäng? | Spridningsdiagram | Visar två numeriska variabler tillsammans |
| Vilken klass har högst medelpoäng? | Stapeldiagram | Jämför kategorier |
| Hur ändras ett värde över tid? | Linjediagram | Visar utveckling och ordningsföljd |

**Tumregel:** Börja med att formulera frågan du vill besvara. Välj sedan diagram – inte tvärtom.

---

## 12. Ett komplett, körbart exempel

Det här exemplet kräver **inga externa CSV-filer**. Det bygger upp testdata, räknar medelpoäng och sparar tre diagram.

```python
import pandas as pd
import matplotlib.pyplot as plt

# 1. Skapa exempeldata
df = pd.DataFrame({
    "klass": ["A", "A", "B", "B", "C", "C", "A", "B"],
    "studietimmar": [2, 5, 1, 4, 0, 3, 6, 2],
    "provpoang": [65, 88, 55, 76, 72, 69, 95, 61],
})

# 2. Sammanfatta per klass
sam_class = (
    df.groupby("klass", as_index=False)
      .agg(medel_poang=("provpoang", "mean"))
)

# 3. Histogram
fig, ax = plt.subplots()
ax.hist(df["provpoang"], bins=5)
ax.set(title="Fördelning av provpoäng",
       xlabel="Provpoäng", ylabel="Antal")
ax.grid(True, axis="y")
fig.tight_layout()
fig.savefig("hist_provpoang.png", dpi=150)
plt.close(fig)

# 4. Spridningsdiagram
fig, ax = plt.subplots()
ax.scatter(df["studietimmar"], df["provpoang"], alpha=0.6)
ax.set(title="Studietimmar och provpoäng",
       xlabel="Studietimmar", ylabel="Provpoäng")
ax.grid(True)
fig.tight_layout()
fig.savefig("scatter_timmar_poang.png", dpi=150)
plt.close(fig)

# 5. Stapeldiagram
fig, ax = plt.subplots()
ax.bar(sam_class["klass"], sam_class["medel_poang"])
ax.set(title="Medelpoäng per klass",
       xlabel="Klass", ylabel="Medelpoäng")
ax.grid(True, axis="y")
fig.tight_layout()
fig.savefig("medelpoang_per_klass.png", dpi=150)
plt.close(fig)

print("Tre diagram har sparats som PNG-filer.")
```

**Förväntad utskrift:**

```text
Tre diagram har sparats som PNG-filer.
```

`plt.close(fig)` stänger figuren efter att den sparats. Det är praktiskt i skript som skapar många bilder. Vill du **se** figurerna i en notebook kan du använda `plt.show()` innan de stängs, eller utelämna `plt.close(fig)` medan du experimenterar.

---

## 13. Vanliga misstag och hur du löser dem

### `NameError: name 'plt' is not defined`

Du har glömt importen:

```python
import matplotlib.pyplot as plt
```

### `KeyError: 'provpoang'`

Kolumnen heter något annat i din DataFrame. Kontrollera:

```python
print(df.columns.tolist())
```

Till exempel är `provpoang` och `provpoäng` **olika** kolumnnamn.

### `FileNotFoundError` vid `pd.read_csv(...)`

Python hittar inte CSV-filen där programmet körs. Kontrollera aktuell mapp med `Path.cwd()` och ange rätt sökväg.

### `ValueError` för olika långa x- och y-listor

I `ax.plot(x, y)` och `ax.scatter(x, y)` måste varje x-värde ha ett motsvarande y-värde.

### Diagrammet verkar missvisande

Kontrollera om du har valt rätt diagramtyp, rätt axelrubriker, lämpliga intervall och om saknade värden har hanterats rimligt. Undvik att dra slutsatser som går längre än datan stödjer.

### Diagrammet sparas men syns inte

`savefig` **sparar** filen; `show` **visar** figuren. Det är två olika saker.

---

## 14. Quick Reference – Matplotlib

| Kod | Funktion |
|---|---|
| `import matplotlib.pyplot as plt` | Importera Matplotlibs diagramfunktioner |
| `fig, ax = plt.subplots()` | Skapa figur och ritarea |
| `ax.plot(x, y)` | Linjediagram |
| `ax.hist(data, bins=10)` | Histogram |
| `ax.scatter(x, y, alpha=0.6)` | Spridningsdiagram |
| `ax.bar(kategorier, varden)` | Stapeldiagram |
| `ax.set_title("Rubrik")` | Diagramrubrik |
| `ax.set_xlabel("X")` | Rubrik på x-axeln |
| `ax.set_ylabel("Y")` | Rubrik på y-axeln |
| `ax.grid(True, axis="y")` | Vågräta stödlinjer |
| `plt.tight_layout()` | Anpassa marginaler |
| `fig.savefig("bild.png", dpi=150)` | Spara diagram |
| `plt.show()` | Visa diagram |
| `plt.close(fig)` | Stäng figuren |
| `df["kolumn"].dropna()` | Välj kolumn utan saknade värden |

---

## 15. Övningar

Använd gärna exempel-DataFramen från avsnitt 2.

**Övning 1 – Förstå grunden.** Förklara med egna ord vad `fig` och `ax` betyder i `fig, ax = plt.subplots()`.

**Övning 2 – Linjediagram.** Rita punkterna `(0, 3)`, `(1, 5)`, `(2, 4)` som en linje. Ge diagrammet rubriken `Mitt linjediagram`.

**Övning 3 – Histogram.** Skapa ett histogram över `provpoang` med `bins=4`. Vad händer om du ändrar till `bins=8`?

**Övning 4 – Spridningsdiagram.** Rita studietimmar på x-axeln och provpoäng på y-axeln. Använd `alpha=0.5`.

**Övning 5 – Stapeldiagram.** Skapa ett stapeldiagram över medelpoängen per klass.

**Övning 6 – Spara fil.** Spara ditt stapeldiagram som `klassresultat.png` med `dpi=150`.

**Övning 7 – Tolka resultat.** En klass har högre medelpoäng än en annan. Kan vi direkt säga att läraren i den första klassen är bättre? Varför/varför inte?

**Övning 8 – Resonera om datarensning.** Varför kan det bli missvisande att ersätta okända studietimmar med `0` innan du gör ett spridningsdiagram?

---

## 16. Facit och förklaringar

### Facit 1

`fig` är hela bilden/figuren. `ax` är ritområdet där diagrammet och dess koordinataxlar finns. `plt.subplots()` skapar båda.

### Facit 2

```python
fig, ax = plt.subplots()
ax.plot([0, 1, 2], [3, 5, 4])
ax.set_title("Mitt linjediagram")
plt.show()
```

Första listan anger x-positioner och andra listan y-positioner.

### Facit 3

```python
fig, ax = plt.subplots()
ax.hist(df["provpoang"], bins=4)
ax.set_xlabel("Provpoäng")
ax.set_ylabel("Antal")
plt.show()
```

Med `bins=8` blir intervallen i regel fler och smalare. Antalet observationer ändras inte – bara hur de grupperas i bilden.

### Facit 4

```python
fig, ax = plt.subplots()
ax.scatter(df["studietimmar"], df["provpoang"], alpha=0.5)
ax.set_xlabel("Studietimmar")
ax.set_ylabel("Provpoäng")
plt.show()
```

Varje punkt representerar en rad/student i datan.

### Facit 5

```python
sam_class = (
    df.groupby("klass", as_index=False)
      .agg(medel_poang=("provpoang", "mean"))
)

fig, ax = plt.subplots()
ax.bar(sam_class["klass"], sam_class["medel_poang"])
ax.set_xlabel("Klass")
ax.set_ylabel("Medelpoäng")
plt.show()
```

`groupby` grupperar rader efter klass och `mean` räknar medelvärdet inom varje grupp.

### Facit 6

Lägg följande **före** `plt.show()` i övning 5:

```python
fig.savefig("klassresultat.png", dpi=150)
```

### Facit 7

**Nej.** Diagrammet visar en skillnad i medelpoäng, men inte vad som orsakar skillnaden. Grupperna kan exempelvis ha olika förkunskaper, antal elever eller andra förutsättningar.

### Facit 8

En saknad uppgift betyder **okänt antal timmar**, inte nödvändigtvis **noll timmar**. Om vi ersätter okänt med noll skapar vi datapunkter vid x = 0 som kanske inte motsvarar verkligheten. Det kan påverka både diagrammets utseende och korrelationen.

---

## 17. Sammanfattning – vad ska du komma ihåg?

1. **Pandas** förbereder datan. **Matplotlib** visualiserar den.
2. `fig, ax = plt.subplots()` är grundmönstret för att skapa diagram.
3. **Histogram** visar fördelning, **scatter** visar samband och **bar** jämför grupper.
4. Bra rubriker och axeltexter gör diagram möjliga att förstå.
5. `bins` påverkar hur detaljerat ett histogram blir.
6. `fig.savefig(...)` sparar diagrammet som bild.
7. **Tolka försiktigt:** ett diagram visar mönster i datan, inte automatiskt orsaker eller säkra förklaringar.

**Nästa steg:** Använd diagrammen i veckans övningsuppgifter eller på gruppens e-handelsdata, till exempel histogram över priser, spridningsdiagram över pris och antal eller stapeldiagram över försäljning per kategori.
