# 11 – Hypotesprövning

**Ämne:** Python, statistik och dataanalys  
**Video:** [Hypotesprövning – lärarens genomgång](https://www.youtube.com/watch?v=jfBuCNus-HY)  
**Föregående lektion:** `10_confidence_intervals.md`

> **Målet:** Förstå hur vi undersöker om två gruppers medelvärden skiljer sig mer än vad slumpmässig variation rimligen kan förklara. Vi följer lärarens arbetsflöde från rådata till slutsats.

## Vad lär vi oss?

- Formulera nollhypotes (H₀) och alternativhypotes (H₁).
- Förklara och tolka p-värden och signifikansnivå.
- Skilja på att **förkasta** och **inte kunna förkasta** H₀.
- Utföra vanligt t-test och Welch-test med SciPy.
- Förstå hur permutationstest skapar en nollfördelning.
- Använda bootstrap till konfidensintervall – utan att förväxla det med ett hypotesprov.
- Tolka skillnadens storlek med Cohen's d och redovisa en begriplig slutsats.

---

## 1. Varför behöver vi hypotesprövning?

Tänk dig att två grupper har följande medelvärden:

| Grupp | Medelvärde |
|---|---:|
| A | 50 |
| B | 53 |

**Fråga:** Är B verkligen högre i populationen, eller råkade vårt stickprov innehålla högre värden i B?

Tre olika frågor hör ihop:

1. **Hur stor är skillnaden?** Här är den observerade skillnaden `53 - 50 = 3`.
2. **Hur osäker är skillnaden?** Det kan ett **konfidensintervall** visa.
3. **Hur ovanlig är skillnaden om det egentligen inte finns någon effekt?** Det undersöker ett **hypotesprov**, med ett **p-värde**.

Ett p-värde berättar alltså inte hur *stor* eller *viktig* skillnaden är.

## 2. H₀ och H₁ – två konkurrerande påståenden

Vi jämför populationsmedelvärdena `μA` och `μB`:

- **H₀ (nollhypotesen):** `μB - μA = 0`. Ingen skillnad i populationens medelvärden.
- **H₁ (alternativhypotesen):** `μB - μA ≠ 0`. Det finns en skillnad, åt något håll.

Tecknet `≠` betyder **inte lika med**. Det här är ett **tvåsidigt test**: både att B är högre och att A är högre räknas som möjliga avvikelser från H₀.

**Varför formulera hypoteserna först?** För att vi inte ska välja ett test som passar resultatet först efter att vi sett siffrorna.

## 3. P-värde och signifikansnivå

**P-värdet** är sannolikheten, **under antagandet att H₀ och testets förutsättningar gäller**, att få ett testresultat minst lika extremt som det observerade.

Vi bestämmer ofta `α = 0.05` (5 %) i förväg.

| Resultat | Beslut | Vad vi kan säga |
|---|---|---|
| `p < 0.05` | Förkasta H₀ | Data ger statistisk evidens mot H₀. |
| `p >= 0.05` | Kan inte förkasta H₀ | Vi har inte tillräcklig evidens mot H₀. |

**Exempel från lärarens video:** Ett t-test ger ungefär `p = 0.1363`. Eftersom `0.1363 > 0.05` kan vi **inte förkasta H₀**.

Det betyder **inte** att vi har bevisat att grupperna är lika.

**Vanliga feltolkningar:**

- ❌ ”P-värdet är sannolikheten att H₀ är sann.”
- ❌ ”P = 0,14 betyder 14 % chans att resultatet bara är slump.”
- ❌ ”Ett litet p-värde betyder en stor och viktig effekt.”
- ✅ ”Om H₀ gäller är ett minst lika extremt resultat ungefär så här ovanligt enligt testmodellen.”

## 4. Förbered Pythonmiljön

Läraren använder NumPy, Pandas, Matplotlib och `scipy.stats`.

```bash
python -m pip install numpy pandas matplotlib scipy
```

**Varför `python -m pip`?** Det kör `pip` genom den Python-installation som kommandot `python` pekar på. Det minskar risken att paket installeras i fel miljö.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

# Gemensamma inställningar för grafer
plt.rcParams["figure.figsize"] = (8, 5)
plt.rcParams["figure.dpi"] = 120
plt.rcParams["axes.grid"] = True

# Styr bara hur många decimaler som visas
np.set_printoptions(precision=3)
pd.set_option("display.precision", 3)
```

**Förklaring:** `rcParams` ändrar standardinställningar för Matplotlib. `display.precision` ändrar hur värden visas i Pandas, **inte** hur noggrant de lagras.

## 5. Skapa övningsdata – grupp A och B

I videon skapar läraren ett syntetiskt dataset med totalt 200 observationer, två grupper, datum och mätvärden. Grupp B får ett något högre förväntat medelvärde och större spridning.

```python
rng = np.random.default_rng(42)  # Reproducerbar slump

n_total = 200
n_a = n_total // 2
n_b = n_total - n_a

metric_a = rng.normal(loc=50, scale=10, size=n_a)
metric_b = rng.normal(loc=53, scale=12, size=n_b)

df = pd.DataFrame({
    "group": ["A"] * n_a + ["B"] * n_b,
    "metric": np.concatenate([metric_a, metric_b]),
    "date": pd.date_range("2025-01-01", periods=n_total, freq="D"),
})

# Lägg avsiktligt till fem saknade mätvärden
missing_idx = rng.choice(df.index, size=5, replace=False)
df.loc[missing_idx, "metric"] = np.nan

# Lägg till en exakt kopia av en rad
extra_row = df.sample(n=1, random_state=17)
df = pd.concat([df, extra_row], ignore_index=True)

# Lägg till ett högt och ett lågt extremvärde
idx_a = df.query("group == 'A' and metric == metric", engine="python").sample(1, random_state=18).index
idx_b = df.query("group == 'B' and metric == metric", engine="python").sample(1, random_state=19).index
df.loc[idx_a, "metric"] += 80
df.loc[idx_b, "metric"] -= 60

df.to_csv("ab_data.csv", index=False)
```

**Vad betyder koden?**

- `default_rng(42)` gör slumpexemplet reproducerbart.
- `loc=50` är normalfördelningens medelvärde; `scale=10` är dess standardavvikelse.
- `size=n_a` anger hur många värden vi skapar.
- `["A"] * n_a` upprepar texten `A` 100 gånger.
- `np.concatenate(...)` lägger ihop två arrayer till en.
- `rng.choice(..., replace=False)` väljer olika index utan återläggning.
- `df.loc[rader, "metric"]` väljer specifika rader och en kolumn.
- `np.nan` betyder saknat numeriskt värde.
- `pd.concat` lägger ihop tabeller/rader.
- `to_csv(..., index=False)` sparar data utan Pandas extra indexkolumn.

**Obs:** Vi använder en modern NumPy-slumpgenerator. Därför blir våra exakta resultat inte identiska med videons, trots samma grundidé.

## 6. Läs in och undersök datan

```python
data = pd.read_csv("ab_data.csv", parse_dates=["date"])

display(data.head(10))
data.info()
display(data.describe(include="all"))

print("Saknade värden per kolumn:")
print(data.isna().sum())

dup_mask = data.duplicated(keep=False)
print("Rader som ingår i dubblettpar:", dup_mask.sum())
display(data.loc[dup_mask])
```

**Varför?** Innan vi testar hypoteser måste vi veta om datan innehåller saknade värden, fel datatyper, dubbletter och extremvärden.

- `head(10)` visar de första tio raderna.
- `info()` visar kolumner, datatyper och antal icke-saknade värden.
- `describe()` ger sammanfattande statistik.
- `isna().sum()` räknar saknade värden.
- `duplicated(keep=False)` markerar **alla rader** i dubblettgrupper. Därför kan två markerade rader motsvara **en** extra kopia.

### Städa utan att förstöra originalet

```python
clean = data.dropna(subset=["metric"]).copy()

before = len(clean)
clean = clean.drop_duplicates()
after = len(clean)

print("Dubbletter borttagna:", before - after)

counts = clean["group"].value_counts()
print(counts)
assert (counts >= 20).all(), "För få observationer i en grupp"
```

**Varför `.copy()`?** Vi arbetar med en separat DataFrame och behåller `data` som referens till ursprungliga inläsningen. `dropna(subset=["metric"])` tar bara bort rader som saknar just mätvärdet. `drop_duplicates()` tar bort extra identiska rader.

`assert` stoppar programmet om villkoret inte stämmer. **20 är lärarens pedagogiska kontrollgräns, inte en universell statistisk regel** för när ett t-test fungerar.

## 7. Utforskande dataanalys (EDA)

```python
summary = clean.groupby("group")["metric"].agg(
    ["count", "mean", "median", "std", "min", "max"]
)
display(summary)
```

`groupby("group")` delar upp datan efter A och B. `agg(...)` beräknar flera statistiska mått för varje grupp.

### Histogram – hur fördelas värdena?

```python
fig, ax = plt.subplots()

for group_name, subset in clean.groupby("group"):
    ax.hist(subset["metric"], bins=20, alpha=0.5, label=f"Grupp {group_name}")

ax.set_title("Histogram över mätvärden per grupp")
ax.set_xlabel("Mätvärde")
ax.set_ylabel("Antal")
ax.legend()
plt.show()
```

`alpha=0.5` gör staplarna halvgenomskinliga så att överlappande grupper syns. `bins=20` delar in värdena i 20 intervall.

### Boxplot – median, kvartiler och avvikare

```python
a = clean.loc[clean["group"] == "A", "metric"].to_numpy()
b = clean.loc[clean["group"] == "B", "metric"].to_numpy()

fig, ax = plt.subplots()
ax.boxplot([a, b], tick_labels=["A", "B"], showmeans=True)
ax.set_title("Boxplot per grupp")
ax.set_xlabel("Grupp")
ax.set_ylabel("Mätvärde")
plt.show()
```

- **Linjen i lådan:** medianen.
- **Lådan:** de mittersta 50 % av observationerna.
- **Punkter utanför whiskers:** möjliga avvikare enligt boxplottens regel.
- `showmeans=True` visar även medelvärdet.

### Violinplot – fördelningens form

```python
fig, ax = plt.subplots()
ax.violinplot([a, b], showmeans=True, showmedians=True)
ax.set_xticks([1, 2], ["A", "B"])
ax.set_title("Violinplot per grupp")
ax.set_xlabel("Grupp")
ax.set_ylabel("Mätvärde")
plt.show()
```

En violinplot visar en utjämnad uppskattning av fördelningens form. Bredare partier betyder större koncentration av observationer i det området.

## 8. Extremvärden – ska vi ta bort dem?

Läraren demonstrerar hur medelvärdena förändras om man tillfälligt tar bort de yttersta 1 % av värdena i hela datasetet.

```python
low, high = clean["metric"].quantile([0.01, 0.99])
clipped = clean.loc[clean["metric"].between(low, high)].copy()

comparison = pd.DataFrame({
    "Original": clean.groupby("group")["metric"].mean(),
    "Utan yttersta 1 %": clipped.groupby("group")["metric"].mean(),
})
display(comparison)
```

**Viktigt:** `quantile([0.01, 0.99])` hittar gränserna för den första och nittionionde percentilen. `between(...)` behåller värden inom gränserna. Det här är **en känslighetsanalys**, inte ett automatiskt sätt att ”förbättra” datan. I resten av lektionen använder vi `clean` och behåller extremvärdena, precis som i videon.

## 9. T-test och Welch-test

Ett t-test jämför skillnaden mellan medelvärden med hur mycket denna skillnad förväntas variera mellan stickprov.

**Oberoende grupper** innebär att värdena i A inte är parade med specifika värden i B. Mätningar på samma personer före och efter en behandling kräver i stället ett parat test.

```python
# Klassiskt t-test: antar lika populationsvarianser
standard = stats.ttest_ind(a, b, equal_var=True)

# Welch-test: kräver inte lika populationsvarianser
welch = stats.ttest_ind(a, b, equal_var=False)

print(f"Klassiskt t-test: t = {standard.statistic:.3f}, p = {standard.pvalue:.4f}")
print(f"Welch-test:      t = {welch.statistic:.3f}, p = {welch.pvalue:.4f}")
```

**Varför `equal_var=False`?** Welch-testet tar hänsyn till att gruppernas spridning kan skilja sig. Det är ofta ett bra standardval för två oberoende grupper.

**Varför kan t-värdet vara negativt?** `ttest_ind(a, b)` räknar i riktningen **A minus B**. Om B har högre medelvärde blir t-värdet negativt. Det betyder inte att testet misslyckats.

**Videons resultat:** B ligger ungefär **2,63 enheter högre** än A. Läraren får ungefär **t = −1,496** och **p = 0,1363**. Det är inte signifikant vid `α = 0,05`.

### Vad beräknar testet under huven?

För Welch-testet:

\[
SE_{B-A}=\sqrt{\frac{s_A^2}{n_A}+\frac{s_B^2}{n_B}}
\]

\[
t=\frac{\bar{x}_B-\bar{x}_A}{SE_{B-A}}
\]

Här betyder `s²` **varians**, `n` antal observationer och `SE` standardfel för skillnaden.

```python
diff = b.mean() - a.mean()
var_a = a.var(ddof=1)
var_b = b.var(ddof=1)
n_a = len(a)
n_b = len(b)

se_diff = np.sqrt(var_a / n_a + var_b / n_b)
t_manual = diff / se_diff

print(f"Observerad skillnad B - A: {diff:.3f}")
print(f"Standardfel: {se_diff:.3f}")
print(f"Manuellt t-värde (B - A): {t_manual:.3f}")
```

`ddof=1` ger stickprovsvariansen med nämnaren `n - 1`. `np.sqrt(...)` tar kvadratroten. Det manuella t-värdet får motsatt tecken jämfört med `ttest_ind(a, b)`, eftersom vi här använder **B minus A**.

Welch-testet justerar dessutom **frihetsgraderna** med Welch–Satterthwaite-approximationen. SciPy sköter den beräkningen åt oss.

## 10. Bootstrap – osäkerheten kring skillnaden

Bootstrap innebär att vi drar nya stickprov **med återläggning** inom respektive grupp. Vi beräknar skillnaden i medelvärde för varje nytt stickprov.

```python
rng = np.random.default_rng(23)
n_boot = 10_000
boot_diffs = np.empty(n_boot)

for i in range(n_boot):
    a_star = rng.choice(a, size=len(a), replace=True)
    b_star = rng.choice(b, size=len(b), replace=True)
    boot_diffs[i] = b_star.mean() - a_star.mean()

ci_low, ci_high = np.percentile(boot_diffs, [2.5, 97.5])
print(f"Observerad skillnad: {diff:.3f}")
print(f"95 % bootstrap-KI: [{ci_low:.3f}, {ci_high:.3f}]")
```

- `replace=True` betyder att samma observation får väljas flera gånger.
- `np.empty(n_boot)` reserverar plats för 10 000 resultat.
- `boot_diffs[i]` sparar skillnaden från ett av våra nya stickprov.
- `np.percentile(..., [2.5, 97.5])` tar gränserna för ett **percentilbaserat 95-procentigt konfidensintervall**.

### Viktig rättelse till en mellanberäkning i videon

Läraren prövar först att räkna ett ”bootstrap-p-värde” genom att jämföra `abs(boot_diffs)` med den observerade skillnaden. **Det är inte ett giltigt p-värde för H₀ i detta upplägg.** Bootstrapfördelningen ovan är centrerad nära den *observerade* skillnaden, inte runt noll. Därför beskriver den uppskattningens osäkerhet, men simulerar inte H₀.

**Kom ihåg:**

- **Bootstrap inom grupperna →** konfidensintervall för skillnaden.
- **Permutation av gruppetiketter →** nollfördelning och p-värde.

Det är också den skillnad som förklaras i skolans skriftliga lektionsmaterial.

## 11. Permutationstest – simulera nollhypotesen

Under H₀ antar vi, i det här testupplägget, att gruppetiketterna kan bytas utan att värdenas fördelning påverkas. Vi blandar därför observationerna, behåller gruppstorlekarna och ser vilka skillnader slumpen skapar.

```python
rng = np.random.default_rng(24)
n_perm = 10_000

all_values = np.concatenate([a, b])
perm_diffs = np.empty(n_perm)

for i in range(n_perm):
    shuffled = rng.permutation(all_values)
    perm_a = shuffled[:len(a)]
    perm_b = shuffled[len(a):]
    perm_diffs[i] = perm_b.mean() - perm_a.mean()

# Tvåsidigt p-värde; +1-korrigering för Monte Carlo-simulering
extreme_count = np.count_nonzero(np.abs(perm_diffs) >= abs(diff))
p_perm = (extreme_count + 1) / (n_perm + 1)

print(f"Observerad skillnad B - A: {diff:.3f}")
print(f"Permutationstestets p-värde: {p_perm:.4f}")
```

**Steg för steg:**

1. `np.concatenate([a, b])` slår ihop mätvärdena.
2. `rng.permutation(...)` blandar deras ordning slumpmässigt.
3. `[:len(a)]` skapar en ny A med rätt storlek.
4. `[len(a):]` ger resten till B.
5. `perm_b.mean() - perm_a.mean()` mäter skillnaden i den nya gruppindelningen.
6. `np.abs(...)` gör testet tvåsidigt: lika extrema skillnader åt båda håll räknas.
7. Andelen extrema permutationer uppskattar p-värdet.

**Varför +1?** Korrigeringen `(extreme_count + 1)/(n_perm + 1)` undviker att rapportera ett exakt nollvärde enbart på grund av ett begränsat antal simuleringar.

**Antagande:** Den enkla omblandningen är lämplig när observationerna är oberoende och utbytbara mellan grupperna under H₀. Den är inte automatiskt giltig för parade data eller när gruppernas fördelningar har olika form/spridning. Welch-testet kan vara lämpligare när just varianserna skiljer sig.

### Rita nollfördelningen

```python
fig, ax = plt.subplots()
ax.hist(perm_diffs, bins=35, alpha=0.7, edgecolor="white")
ax.axvline(diff, color="red", label="Observerad skillnad")
ax.axvline(-abs(diff), color="red", linestyle="--", label="Extrem gräns")
ax.axvline(abs(diff), color="red", linestyle="--")
ax.set_title("Permutationstest: skillnader om H₀ gäller")
ax.set_xlabel("Skillnad i medelvärde (B - A)")
ax.set_ylabel("Antal permutationer")
ax.legend()
plt.show()
```

Histogrammet visar de skillnader vi fick genom att slumpmässigt blanda gruppetiketterna. De två streckade gränserna markerar vad som räknas som **minst lika extremt** som den observerade skillnaden.

**Videons senare permutationstest:** Läraren får ungefär `p = 0,141`. Det ger samma beslut som t-testet: **H₀ kan inte förkastas**.

## 12. Konfidensintervall och hypotesprov hör ihop

Vi har redan ett 95-procentigt bootstrapintervall i `ci_low` och `ci_high`.

```python
fig, ax = plt.subplots()

ax.errorbar(
    x=[0], y=[diff],
    yerr=[[diff - ci_low], [ci_high - diff]],
    fmt="o", capsize=6
)
ax.axhline(0, color="gray", linestyle="--")
ax.set_xticks([0], ["B - A"])
ax.set_ylabel("Skillnad i medelvärde")
ax.set_title("Observerad skillnad med 95 % bootstrap-KI")
plt.show()
```

**Tolkning:** Punkten är vår bästa uppskattning av skillnaden. Felstapeln visar osäkerheten. Den streckade linjen vid **0** betyder ingen skillnad.

Om ett 95-procentigt intervall innehåller noll är en nollskillnad förenlig med intervallet. För **matchande** tvåsidiga test och intervallmetoder motsvarar detta normalt att vi inte förkastar H₀ vid 5 %. Men **bootstrap-percentilintervall och ett separat permutationstest behöver inte alltid ge exakt samma beslut**, eftersom metoderna bygger på olika beräkningar och antaganden.

## 13. Effektstorlek – hur stor är skillnaden?

En statistiskt signifikant skillnad behöver inte vara viktig i praktiken. Därför rapporterar vi även skillnaden i ursprungliga enheter och ibland en **standardiserad effektstorlek**.

**Cohen's d** mäter skillnaden i medelvärde i enheter av gruppernas gemensamma standardavvikelse.

```python
pooled_sd = np.sqrt(
    ((len(a) - 1) * a.var(ddof=1) + (len(b) - 1) * b.var(ddof=1))
    / (len(a) + len(b) - 2)
)
cohens_d = (b.mean() - a.mean()) / pooled_sd
print(f"Cohen's d (B - A): {cohens_d:.3f}")
```

**Vad betyder det?** Om `d = 0.5` är B:s medelvärde en halv gemensam standardavvikelse högre än A:s. Ofta nämns ungefär `0.2` (liten), `0.5` (medelstor) och `0.8` (stor), men dessa är **grova riktvärden**, inte universella gränser.

**Varför både p och d?** Ett stort stickprov kan göra en mycket liten skillnad statistiskt signifikant. Cohen's d beskriver storleken, men om skillnaden är praktiskt viktig beror på sammanhanget.

## 14. Sammanfatta en komplett analys

```python
print(f"Antal: A = {len(a)}, B = {len(b)}")
print(f"Medelvärde A: {a.mean():.2f}")
print(f"Medelvärde B: {b.mean():.2f}")
print(f"Skillnad B - A: {diff:.2f}")
print(f"Welch-test: p = {welch.pvalue:.4f}")
print(f"Permutationstest: p = {p_perm:.4f}")
print(f"95 % bootstrap-KI: [{ci_low:.2f}, {ci_high:.2f}]")
print(f"Cohen's d: {cohens_d:.2f}")

alpha = 0.05
if welch.pvalue < alpha:
    print("Beslut: Förkasta H₀ vid 5 % signifikansnivå.")
else:
    print("Beslut: Kan inte förkasta H₀ vid 5 % signifikansnivå.")
```

**Exempel på skriftlig slutsats från videons data:**

> Grupp B hade i stickprovet ett medelvärde som var cirka 2,63 enheter högre än grupp A. Ett Welch-test gav ungefär p = 0,136 och ett permutationstest ungefär p = 0,141. Vid signifikansnivån 0,05 kan vi därför inte förkasta nollhypotesen om lika populationsmedelvärden. Bootstrapintervallet omfattade noll, vilket visar att en nollskillnad är förenlig med uppskattningens osäkerhet. Resultatet bevisar inte att grupperna är identiska.

I en riktig rapport anger vi dessutom **exakta intervallgränser, gruppstorlekar, effektstorlek och eventuella begränsningar**. Våra egna simulerade data ovan kan ge andra siffror än videons.

## 15. Vanliga misstag

| Misstag | Varför blir det fel? | Gör så här i stället |
|---|---|---|
| ”p = 0,14 betyder 14 % sannolikhet att H₀ är sann.” | P-värdet förutsätter H₀; det beräknar inte sannolikheten för H₀. | Tolka sannolikheten för extrema data **givet H₀**. |
| ”p > 0,05 bevisar att grupperna är lika.” | Testet kan sakna styrka för att upptäcka en skillnad. | Skriv **kan inte förkasta H₀**. |
| Använda vanlig gruppvis bootstrap som nollfördelning. | Den är centrerad nära observerad skillnad, inte noll. | Använd permutationstest för p-värde när antagandena passar. |
| Ta bort alla outliers automatiskt. | Det kan snedvrida resultatet. | Undersök och dokumentera skälen. |
| Blanda ihop A−B med B−A. | Tecknet vänds. | Bestäm jämförelseriktning och håll fast vid den. |
| Köra oberoende t-test på före/efter-mätningar av samma personer. | Observationerna är parade. | Använd en metod för parade data. |
| Titta enbart på p-värdet. | Det visar inte effektens storlek eller precision. | Redovisa skillnad, KI och effektstorlek. |

## 16. Övningar

Försök gärna köra koden i en Jupyter Notebook. Facit finns direkt efteråt.

1. Skriv H₀ och H₁ för en jämförelse av två gruppers medelvärden.
2. Ett test ger `p = 0.03` och `α = 0.05`. Vilket beslut fattar du?
3. Ett annat test ger `p = 0.21`. Har du bevisat att grupperna är identiska?
4. Vad är skillnaden mellan `equal_var=True` och `equal_var=False` i `stats.ttest_ind()`?
5. Varför kan t-värdet bli negativt när du kör `stats.ttest_ind(a, b)`?
6. Vad gör `clean.dropna(subset=["metric"])`?
7. Vad betyder `replace=True` i `rng.choice()`?
8. Varför kan vi inte använda en vanlig gruppvis bootstrapfördelning direkt för att räkna p-värdet mot H₀?
9. Vad gör `rng.permutation(all_values)` i permutationstestet?
10. Ett 95-procentigt KI för B−A är `[-1.2, 4.8]`. Vad säger det om nollskillnaden?
11. Vad tillför Cohen's d som ett p-värde inte berättar?
12. Ändra övningskoden så att B har `loc=60` i stället för `53`. Kör om analysen. Vad händer med observerad skillnad, p-värde och effektstorlek? (Det exakta resultatet beror på slumputfallet.)

## 17. Facit

1. **H₀:** `μB − μA = 0`. **H₁:** `μB − μA ≠ 0`.
2. Eftersom `0.03 < 0.05` **förkastar vi H₀**.
3. Nej. Vi **kan inte förkasta H₀**, men har inte bevisat att grupperna är identiska.
4. `True` antar lika populationsvarianser (klassiskt test). `False` gör Welch-test, som inte kräver lika varianser.
5. SciPy beräknar skillnaden som A−B. Om B:s medelvärde är större blir t negativt.
6. Tar bort rader där just kolumnen `metric` saknar värde.
7. Samma observation får väljas flera gånger vid resampling.
8. Den simulerar osäkerhet runt den observerade skillnaden, inte en värld där H₀ gäller.
9. Blandar värdenas ordning så att de kan delas in i nya grupper med samma storlekar.
10. Noll ligger i intervallet. En nollskillnad är därför förenlig med detta KI.
11. Det visar den **standardiserade storleken** på den observerade skillnaden.
12. B får i regel ett högre observerat medelvärde, en större positiv B−A-skillnad och större Cohen's d. P-värdet blir ofta mindre, men kontrollera det faktiska resultatet.

## Quick Reference

```python
from scipy import stats
import numpy as np

# Gruppdata (numeriska NumPy-arrayer)
a = np.array([47, 51, 53, 49, 50])
b = np.array([52, 55, 56, 51, 54])

# Observerad skillnad
observed_diff = b.mean() - a.mean()

# Welch-test
result = stats.ttest_ind(a, b, equal_var=False)
print(result.statistic, result.pvalue)

# 95 % bootstrap-KI för B - A
rng = np.random.default_rng(123)
boot = []
for _ in range(10_000):
    a_star = rng.choice(a, len(a), replace=True)
    b_star = rng.choice(b, len(b), replace=True)
    boot.append(b_star.mean() - a_star.mean())
print(np.percentile(boot, [2.5, 97.5]))

# Tvåsidigt permutationstest
values = np.concatenate([a, b])
perms = []
for _ in range(10_000):
    shuffled = rng.permutation(values)
    perms.append(shuffled[len(a):].mean() - shuffled[:len(a)].mean())
p_value = (np.sum(np.abs(perms) >= abs(observed_diff)) + 1) / (len(perms) + 1)
print(p_value)
```

---

**Kom ihåg:** Hypotesprövning handlar inte om att bevisa en hypotes. Vi undersöker hur väl data stämmer med nollhypotesen, samtidigt som vi redovisar **effektens storlek och osäkerhet**.
