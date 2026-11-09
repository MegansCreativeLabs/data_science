# Lektion 13 – A/B-test för andelar

> **Kurs:** Python och statistisk dataanalys · **Område:** Data Analysis  
> **Video:** [A/B för andelar](https://www.youtube.com/watch?v=xDooWEJFwYU)  
> **Bygger vidare på:** Lektion 09–12 (sampling, konfidensintervall, hypotesprövning och inferens)

## 1. Vad ska vi lära oss – och varför?

Ett **A/B-test** jämför två versioner av något, exempelvis en gammal (A) och en ny (B) kassasida. Vi vill veta om B faktiskt ger en högre **andel köp**, inte bara om B råkade få fler köp i vårt stickprov.

Efter lektionen ska du kunna kontrollera binära data, räkna andelar och effektstorlekar, formulera hypoteser, genomföra ett z-test, beräkna ett 95-procentigt konfidensintervall för skillnaden och tolka resultatet inför ett beslut.

**Arbetsflödet:** skapa/läs data → kontrollera → städa → sammanfatta → visualisera → testa hypotes → beräkna intervall → tolka effekt → rapportera.

## 2. Binära utfall: varför fungerar medelvärdet?

Ett **binärt utfall** har bara två tillåtna värden: `1` (köp) eller `0` (inget köp). Tänk dig fem besökare:

```python
outcomes = [1, 0, 1, 0, 1]
proportion = sum(outcomes) / len(outcomes)
print(proportion)  # 0.6 = 60 %
```

Medelvärdet blir andelen ettor eftersom nollorna inte bidrar till summan. Med Pandas kan vi därför använda både `sum()` för **antal lyckade utfall** och `mean()` för **andel lyckade utfall**.

**Begrepp:** `n` = antal observationer; `x` = antal ettor; `p_hat = x/n` = observerad andel. En andel som `0.28` motsvarar `28 %`.

## 3. Importera biblioteken

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats
```

- **NumPy** (`np`) används för slumpdata och matematik.
- **Pandas** (`pd`) läser, filtrerar och grupperar tabeller.
- **Matplotlib** (`plt`) ritar diagram.
- **SciPy** (`stats`) ger normalfördelningens funktioner för konfidensintervall och p-värden.

Om SciPy saknas i din miljö: `python -m pip install scipy`.

## 4. Skapa övningsdata och spara CSV

I videon skapar läraren två fiktiva grupper med **150 respektive 160 observationer** och lägger medvetet till saknade, duplicerade och ogiltiga värden. Här är ett eget reproducerbart exempel med samma idé. **Exakta resultat kan skilja sig från videon** eftersom den ursprungliga koden inte finns i transkriptet.

```python
rng = np.random.default_rng(42)  # Samma slumpsekvens vid varje körning
n_a, n_b = 150, 160

# Två grupper med olika sannolikhet för köp
outcome_a = rng.binomial(1, 0.25, size=n_a)
outcome_b = rng.binomial(1, 0.33, size=n_b)

df = pd.DataFrame({
    "group": ["A"] * n_a + ["B"] * n_b,
    "outcome": np.concatenate([outcome_a, outcome_b]),
    "date": pd.date_range("2026-01-01", periods=n_a + n_b, freq="h"),
})

# Lägg till avsiktliga problem för att öva datarensning
df.loc[2, "outcome"] = np.nan
df.loc[12, "outcome"] = np.nan
df.loc[20, "outcome"] = 2
df.loc[25, "outcome"] = -1
df = pd.concat([df, df.iloc[[0]]], ignore_index=True)

df.to_csv("ab_proportion_data.csv", index=False)
```

**Varför `rng.binomial(1, p, size=n)`?** `1` betyder ett försök per observation; varje utfall blir då 0 eller 1, med sannolikheten `p` för 1. `pd.concat` lägger här till en kopia av första raden, så vi får en dubblett att upptäcka.

## 5. Läs in och undersök data

```python
df = pd.read_csv("ab_proportion_data.csv", parse_dates=["date"])

display(df.head())
df.info()
display(df.select_dtypes(include="number").describe())
```

- `read_csv` öppnar filen; `parse_dates` tolkar datumkolumnen som datum.
- `head()` visar de första raderna, så vi ser hur tabellen är uppbyggd.
- `info()` visar datatyper och antal icke-saknade värden.
- `select_dtypes(include="number")` väljer numeriska kolumner innan `describe()` sammanfattar dem.

**Viktigt:** `describe()` ersätter inte en kontroll av ogiltiga värden. Ett `2` kan exempelvis vara numeriskt men är inte ett giltigt binärt utfall.

## 6. Hitta saknade värden, dubbletter och ogiltiga utfall

```python
missing_count = df["outcome"].isna().sum()
duplicate_count = df.duplicated().sum()

invalid_mask = (~df["outcome"].isin([0, 1])) & df["outcome"].notna()
invalid_count = invalid_mask.sum()

print("Saknade utfall:", missing_count)
print("Dubbletter:", duplicate_count)
print("Ogiltiga utfall:", invalid_count)
```

**Läs uttrycket `~df["outcome"].isin([0, 1])`:** `isin([0, 1])` ger `True` för giltiga värden; `~` vänder på svaret, så vi får `True` för *inte* 0 eller 1. `& ...notna()` gör att saknade värden inte räknas som ogiltiga *också*.

`duplicated()` hittar identiska rader, men en verklig A/B-analys behöver ofta ett **unikt användar-ID** för att upptäcka personer som registrerats flera gånger. Två identiska rader är inte automatiskt samma person.

## 7. Städa och kontrollera grupperna

```python
df_clean = df.drop_duplicates().copy()
df_clean = df_clean.dropna(subset=["outcome"])
df_clean = df_clean[df_clean["outcome"].isin([0, 1])].copy()
df_clean["outcome"] = df_clean["outcome"].astype(int)

group_sizes = df_clean.groupby("group")["outcome"].size()
assert set(group_sizes.index) == {"A", "B"}, "Grupp A eller B saknas"
assert (group_sizes >= 20).all(), "För få observationer i någon grupp"

print("Form:", df_clean.shape)
print("Gruppstorlekar:", group_sizes.to_dict())
print("Utfall:", df_clean["outcome"].value_counts().to_dict())
```

`drop_duplicates()` tar bort exakt duplicerade rader; `dropna(subset=[...])` tar bort rader där just utfallet saknas; filtret behåller bara 0 och 1. `assert` stoppar körningen om ett grundvillkor inte uppfylls.

**Observera:** `20` per grupp är bara videons enkla kontroll, **inte** en garanti för att z-testets normalapproximation fungerar. Vi kontrollerar även antal ettor och nollor senare. I riktig data ska man dessutom undersöka *varför* värden saknas innan de tas bort.

## 8. Sammanfatta grupperna med `groupby` och `agg`

```python
summary = (
    df_clean.groupby("group")["outcome"]
    .agg(observations="count", successes="sum", proportion="mean")
    .reset_index()
)
display(summary)
```

`groupby("group")` delar upp raderna i A och B. `agg(...)` gör tre beräkningar **för varje grupp**:

| Kolumn | Vad den betyder | Pandas-metod |
|---|---|---|
| `observations` | Antal giltiga observationer | `count` |
| `successes` | Antal ettor | `sum` |
| `proportion` | Andel ettor | `mean` |

`reset_index()` gör `group` till en vanlig kolumn igen. Det gör det enklare att använda tabellen i nästa steg.

## 9. Konfidensintervall för varje grupps andel

Videon börjar med ett separat ungefärligt 95-procentigt intervall för A och B:

\[
SE(\hat p) = \sqrt{\frac{\hat p(1-\hat p)}{n}},\qquad
CI = \hat p \pm 1.96\,SE(\hat p)
\]

```python
z_crit = stats.norm.ppf(0.975)  # Ungefär 1.96

ci_low = []
ci_high = []

for _, row in summary.iterrows():
    n = row["observations"]
    p = row["proportion"]
    se = np.sqrt(p * (1 - p) / n)
    ci_low.append(p - z_crit * se)
    ci_high.append(p + z_crit * se)

summary["ci_low"] = ci_low
summary["ci_high"] = ci_high

display(summary)
```

`iterrows()` går igenom tabellen rad för rad. `_` är indexet som vi inte behöver; `row` innehåller värdena för aktuell grupp. `append()` lägger ett beräknat värde sist i en lista.

**Vad gör `ppf(0.975)`?** `ppf` är *percent point function* (inversen till `cdf`). Vi vill ha 2,5 % i vardera svansen och därför 97,5 % till vänster om den övre gränsen. Resultatet blir cirka `1.96`.

**Metodbegränsning:** Detta är det enkla **Wald-intervallet** som visas i videon. Det kan fungera dåligt vid små stickprov eller andelar nära 0 och 1. Wilson-intervall är ofta bättre för en enskild andel.

## 10. Rita andelar med felstaplar

```python
fig, ax = plt.subplots(figsize=(7, 5))
x = np.arange(len(summary))

yerr = [
    summary["proportion"] - summary["ci_low"],
    summary["ci_high"] - summary["proportion"],
]

ax.bar(x, summary["proportion"], yerr=yerr, capsize=8)
ax.set_xticks(x, summary["group"])
ax.set_ylim(0, 1)
ax.set_ylabel("Andel köp")
ax.set_title("Andel per grupp med 95 % konfidensintervall")
ax.grid(axis="y", linestyle="--", alpha=0.4)
plt.show()
```

Stapelhöjden visar den observerade andelen; felstaplarna visar osäkerheten. **Jämför inte bara om felstaplarna överlappar.** För att testa en skillnad behöver vi beräkna ett intervall för **B minus A**.

## 11. Formulera hypoteserna innan testet

För populationernas verkliga andelar `p_A` och `p_B`:

- **H₀ (nollhypotes):** `p_B - p_A = 0` – ingen verklig skillnad.
- **H₁ (tvåsidig):** `p_B - p_A != 0` – någon skillnad, oavsett riktning.
- **H₁ (ensidig, om förbestämd):** `p_B > p_A` – B har högre andel.

I videon är **det tvåsidiga testet huvudanalysen**. Ett ensidigt p-värde visas också som demonstration, men riktningen får inte väljas i efterhand.

## 12. Z-test för två andelar – steg för steg

Under H₀ antar vi en gemensam populationsandel. Därför använder testet en **poolad andel**:

\[
\hat p_{pool} = \frac{x_A+x_B}{n_A+n_B}
\]

Testets standardfel och z-värde:

\[
SE_{pool} = \sqrt{\hat p_{pool}(1-\hat p_{pool})\left(\frac1{n_A}+\frac1{n_B}\right)}
\]

\[
z = \frac{\hat p_B-\hat p_A}{SE_{pool}}
\]

```python
# Hämta en rad per grupp, indexerad med gruppnamnet
by_group = summary.set_index("group")

n_a = int(by_group.loc["A", "observations"])
n_b = int(by_group.loc["B", "observations"])
x_a = int(by_group.loc["A", "successes"])
x_b = int(by_group.loc["B", "successes"])

p_a = x_a / n_a
p_b = x_b / n_b
diff = p_b - p_a

p_pooled = (x_a + x_b) / (n_a + n_b)
se_pooled = np.sqrt(p_pooled * (1 - p_pooled) * (1 / n_a + 1 / n_b))

if se_pooled == 0:
    raise ValueError("Standardfelet är noll; z-testet kan inte beräknas.")

z_stat = diff / se_pooled
p_two_sided = 2 * stats.norm.sf(abs(z_stat))
p_one_sided_b_greater = stats.norm.sf(z_stat)

print(f"Observerad skillnad B - A: {diff:.4f}")
print(f"Z-värde: {z_stat:.3f}")
print(f"Tvåsidigt p-värde: {p_two_sided:.4f}")
print(f"Ensidigt p-värde (B > A): {p_one_sided_b_greater:.4f}")
```

**Varför `set_index` och `.loc`?** Vi kan hämta värden via gruppnamn i stället för att förlita oss på radordningen. `loc["A", "successes"]` betyder: hämta värdet i raden A, kolumnen `successes`.

**Varför `sf`?** `stats.norm.sf(z)` ger arean **till höger** om z i standardnormalfördelningen. Den är samma sannolikhet som `1 - stats.norm.cdf(z)`, men numeriskt stabilare vid extrema z-värden. `abs(z_stat)` gör att ett tvåsidigt test räknar båda riktningarna lika.

**Tolkning:** Om `p < 0.05` förkastar vi H₀ vid 5 % signifikansnivå. Om `p >= 0.05` kan vi **inte förkasta H₀**. Det bevisar inte att grupperna är lika. P-värdet är **inte** sannolikheten att H₀ är sann.

## 13. Kontrollera att z-approximationen är rimlig

```python
for group in ["A", "B"]:
    n = int(by_group.loc[group, "observations"])
    x = int(by_group.loc[group, "successes"])
    print(f"{group}: {x} ettor, {n - x} nollor")
```

Vi vill ha tillräckligt många positiva och negativa utfall i **båda** grupperna. En vanlig tumregel är omkring 5–10 förväntade observationer av varje utfall. Om utfallet är mycket sällsynt eller grupperna små kan exempelvis Fishers exakta test vara lämpligare. Observationerna måste också vara tillräckligt oberoende.

## 14. Konfidensintervall för skillnaden B − A

**Här används ett annat standardfel än i z-testet.** För intervallet utgår vi från respektive grupps observerade andel (opoolat):

\[
SE_{unpooled} = \sqrt{\frac{\hat p_A(1-\hat p_A)}{n_A} + \frac{\hat p_B(1-\hat p_B)}{n_B}}
\]

\[
CI_{95\%} = (\hat p_B-\hat p_A) \pm 1.96\,SE_{unpooled}
\]

```python
se_unpooled = np.sqrt(
    p_a * (1 - p_a) / n_a + p_b * (1 - p_b) / n_b
)
ci_low = diff - z_crit * se_unpooled
ci_high = diff + z_crit * se_unpooled

print(f"Skillnad: {diff * 100:.1f} procentenheter")
print(f"95 % KI: [{ci_low * 100:.1f}, {ci_high * 100:.1f}] procentenheter")
```

**Tolka noll:** Om intervallet innehåller 0 är ingen skillnad fortfarande förenlig med data. Om hela intervallet ligger över 0 talar det för högre andel i B; under 0 talar det för lägre andel i B.

**Viktig nyans:** Det opoolade Wald-intervallet och det poolade z-testet är två olika approximationer. Nära gränsen `0.05` behöver deras slutsatser inte stämma exakt överens. För bättre intervall för två andelar finns exempelvis Newcombe-metoden.

## 15. Rita skillnaden och nollinjen

```python
fig, ax = plt.subplots(figsize=(6, 5))
ax.errorbar(
    [0], [diff],
    yerr=[[diff - ci_low], [ci_high - diff]],
    fmt="o", capsize=8,
)
ax.axhline(0, color="gray", linestyle="--", label="Ingen skillnad")
ax.set_xticks([0], ["B − A"])
ax.set_ylabel("Skillnad i andel")
ax.set_title("Skillnad B − A med 95 % konfidensintervall")
ax.legend()
ax.grid(axis="y", linestyle="--", alpha=0.4)
plt.show()
```

Punkten är vår skattade skillnad, det lodräta intervallet visar osäkerheten och den vågräta nollinjen markerar ingen skillnad. Detta diagram besvarar en annan fråga än stapeldiagrammet över separata grupper.

## 16. Absolut och relativ effektstorlek

Anta att A har 20 % köp och B har 25 % köp:

- **Absolut skillnad:** `25 - 20 = 5 procentenheter`.
- **Relativ ökning:** `(25 / 20 - 1) * 100 = 25 %`.

```python
absolute_pp = diff * 100
relative_lift = (p_b / p_a - 1) * 100 if p_a > 0 else np.nan

print(f"Absolut skillnad: {absolute_pp:.1f} procentenheter")
print(f"Relativ förändring: {relative_lift:.1f} %")
```

**Varför kontrollera `p_a > 0`?** Om A:s andel är noll kan vi inte dividera med den. `np.nan` markerar här att den relativa förändringen inte är definierad.

## 17. Skriv en begriplig slutsats

```python
def pct(value):
    return f"{value * 100:.1f} %"

conclusion = f"""
Andel A: {pct(p_a)} (n={n_a})
Andel B: {pct(p_b)} (n={n_b})
Skillnad B − A: {absolute_pp:.1f} procentenheter
95 % KI för skillnaden: [{ci_low * 100:.1f}, {ci_high * 100:.1f}] procentenheter
Tvåsidigt z-test, p-värde: {p_two_sided:.4f}
Relativ förändring: {relative_lift:.1f} %
"""
print(conclusion)
```

I **videons** exempel (inte nödvändigtvis exakt samma som vår egen simulering) är B cirka **8,2 procentenheter högre** än A, relativt cirka **32 % högre**, med ett **tvåsidigt p-värde omkring 0,117**. Intervallet sträcker sig ungefär från **−2 till +18 procentenheter**. Därför går det inte att dra slutsatsen att B säkert är bättre. Videon visar även ett ensidigt p-värde kring **0,059**, men det får bara användas som beslutsgrund om riktningen valdes i förväg.

**Exempel på en rapportmening:** ”Variant B hade en högre observerad konverteringsgrad än A, men osäkerheten är fortfarande stor och det tvåsidiga testet gav inte statistiskt signifikant stöd vid 5 %-nivån. Vi bör inte fatta ett definitivt utrullningsbeslut enbart på dessa data.”

## 18. Stopptid, power och praktisk betydelse

Läraren betonar att experimentet bör planeras **innan** vi tittar på resultaten:

- Bestäm **primärt utfall**, signifikansnivå och om testet är ensidigt eller tvåsidigt.
- Bestäm vilken förbättring som skulle vara **praktiskt värdefull**, exempelvis minst 3 procentenheter.
- Planera **stickprovsstorlek** utifrån önskad power (ofta 80 % eller 90 %).
- Bestäm **stopptid** eller en korrekt sekventiell analysmetod.

Att kontrollera p-värdet om och om igen och stoppa precis när det blir under 0,05 ökar risken för falska positiva resultat. Videon nämner en grov gissning om runt 500 observationer per grupp; det är **inte en genomförd powerberäkning** och ska inte användas som generell regel. Stickprovsstorlek beror bland annat på baslinjeandel, minsta effekt, power och signifikansnivå.

Att samla mer data **efter att ha sett ett osäkert resultat** måste också göras enligt en förutbestämd eller statistiskt giltig plan; man bör inte fortsätta tills resultatet råkar bli signifikant.

## 19. Vanliga misstag

| Misstag | Varför det blir fel | Gör så här |
|---|---|---|
| Blanda ihop procent och procentenheter | 20 % → 25 % är +5 procentenheter, inte +5 % relativt | Redovisa båda tydligt |
| Tolka `p = 0.117` som ”11,7 % chans att H₀ är sann” | P-värdet förutsätter H₀; det ger inte dess sannolikhet | Tolka som hur extremt resultatet vore under H₀ |
| Använda `mean()` innan 0/1-värden validerats | Ogiltiga värden snedvrider andelen | Kontrollera `isin([0, 1])` |
| Jämföra separata gruppers felstaplar som test | Överlappning är inget tillförlitligt test | Beräkna KI för **B − A** |
| Använda poolat SE i intervallet av slentrian | Videons test och intervall bygger på olika SE | Poolat för H₀-testet; opoolat för videons KI |
| Skriva `dropna` utan parenteser | Metoden anropas inte | `dropna(subset=["outcome"])` |
| Använda `and` för Pandas-masker | Pandas-serier kräver elementvis operator | Använd `&`, `|`, `~` och parenteser |
| Byta till ensidigt test efter resultatet | Ökar risken för falska slutsatser | Bestäm riktningen i förväg |
| Avsluta så fort `p < 0.05` | Upprepade oplanerade tester förvränger felrisken | Förbestämd stopptid eller sekventiell metod |

## 20. Snabbguide

| Fråga | Formel / metod |
|---|---|
| Hur stor andel köpte? | `p_hat = x / n` eller `mean()` på 0/1 |
| Hur många köpte? | `sum()` på 0/1 |
| Hur stor är skillnaden? | `diff = p_b - p_a` |
| Skillnad i procentenheter? | `diff * 100` |
| Relativ förändring? | `(p_b / p_a - 1) * 100` |
| Vad säger H₀? | `p_B - p_A = 0` |
| Poolad andel under H₀? | `(x_a + x_b) / (n_a + n_b)` |
| Z-värde? | `diff / se_pooled` |
| Tvåsidigt p-värde? | `2 * stats.norm.sf(abs(z_stat))` |
| 95 % KI för skillnad? | `diff ± 1.96 * se_unpooled` |
| Vad betyder KI som korsar 0? | Nollskillnad är förenlig med data |

## 21. Övningar

1. En grupp har utfallen `[1, 0, 0, 1, 1, 0, 1, 0]`. Hur många ettor finns det? Vad är andelen?
2. A har 100 besökare och 20 köp; B har 100 besökare och 25 köp. Räkna absolut skillnad i procentenheter och relativ förändring.
3. Vad gör `~df["outcome"].isin([0, 1])`? Varför lägger vi till `.notna()` i kontrollen?
4. Förklara skillnaden mellan `count`, `sum` och `mean` när `outcome` bara innehåller 0 och 1.
5. Skriv H₀ och en tvåsidig H₁ för skillnaden mellan två konverteringsgrader.
6. Varför används poolad andel vid z-testet men opoolat standardfel i videons konfidensintervall?
7. Ett 95-procentigt KI för B − A är `[−0.02, 0.18]`. Vad betyder gränserna i procentenheter? Vad säger nollinjen?
8. Ett tvåsidigt test ger `p = 0.117`. Förkastar du H₀ vid `alpha = 0.05`? Vad får du **inte** dra för slutsats?
9. Varför är det problematiskt att kontrollera p-värdet dagligen och stoppa när det är under 0,05?
10. Kör hela Python-exemplet. Ändra grupp B:s simulerade sannolikhet från `0.33` till `0.45`. Hur ändras den observerade skillnaden och p-värdet? Kör flera gånger med olika seeds.
11. Ändra `n_a` och `n_b` till 1000. Vad händer vanligen med konfidensintervallets bredd?
12. Skapa en funktion som tar `x` och `n` och returnerar andelen `x / n`. Kontrollera att `n > 0`.
13. Varför kan ett A/B-test med 10 000 observationer ändå ge missvisande slutsatser?
14. Skriv en kort rapport till en chef utifrån videons exempel: observerad effekt, osäkerhet och ett rimligt beslut.

## 22. Facit och förklaringar

**1.** Fyra ettor av åtta observationer: `4 / 8 = 0.5 = 50 %`.

**2.** A = 20 %, B = 25 %. Absolut skillnad = **5 procentenheter**. Relativ förändring = `(0.25 / 0.20 - 1) * 100 = 25 %`.

**3.** `isin([0, 1])` hittar tillåtna värden; `~` vänder masken till otillåtna. `.notna()` exkluderar saknade värden så att vi kan räkna dem separat.

**4.** `count` = antal icke-saknade rader; `sum` = antal ettor; `mean` = andelen ettor.

**5.** `H₀: p_B - p_A = 0`; `H₁: p_B - p_A != 0`.

**6.** Z-testet beräknar slumpvariationen **under antagandet att andelarna är lika**. Intervallet skattar osäkerheten kring den observerade skillnaden och använder de två observerade andelarna separat.

**7.** Från **−2 till +18 procentenheter**. Eftersom 0 ingår är både ingen skillnad och effekter åt båda håll förenliga med intervallet.

**8.** Nej, `0.117 > 0.05`. Vi kan inte förkasta H₀. Vi har **inte bevisat** att grupperna är lika eller att H₀ har 88,3 % sannolikhet att vara sann.

**9.** Oplanerade upprepade tester ökar sannolikheten att slumpen någon gång ger `p < 0.05` trots att H₀ är sann.

**10.** Med en större verklig skillnad blir den observerade skillnaden oftast större och p-värdet ofta mindre, men varje simulering varierar slumpmässigt.

**11.** Intervallet blir vanligtvis smalare eftersom standardfelet minskar när antalet observationer ökar.

**12.**

```python
def calculate_proportion(x, n):
    if n <= 0:
        raise ValueError("n måste vara större än noll")
    if not 0 <= x <= n:
        raise ValueError("x måste ligga mellan 0 och n")
    return x / n

print(calculate_proportion(28, 100))  # 0.28
```

**13.** Ett stort stickprov räddar inte ett snedvridet urval, felaktig randomisering, beroende observationer eller felaktig mätning.

**14.** Exempel: ”B gav ungefär 8,2 procentenheter högre andel köp än A, men det tvåsidiga p-värdet var cirka 0,117 och 95-procentintervallet sträckte sig ungefär från −2 till +18 procentenheter. Vi har därför inte tillräckligt starkt stöd för att B är bättre. Planera eventuell ytterligare datainsamling och beslutskriterier innan testet fortsätter.”

---

### Sammanfattning

I ett A/B-test med binära utfall räknar vi först **andelar** i varje grupp. Vi skattar sedan **skillnaden B − A**, undersöker med ett **z-test** hur ovanlig skillnaden vore under H₀ och använder ett **konfidensintervall** för att förstå osäkerheten. Till sist väger vi in **effektstorlek, praktisk betydelse och experimentets upplägg** innan vi rekommenderar ett beslut.
