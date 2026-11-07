# 10 – Konfidensintervall

> **Kurs:** Python och statistisk dataanalys  
> **Video:** [Konfidensintervall – Python](https://www.youtube.com/watch?v=xjYEYBvPaSc)  
> **Placering:** `data_analysis/10_confidence_intervals.md`

## Vad lär vi oss?

När vi undersöker ett stickprov känner vi vanligtvis inte populationens verkliga medelvärde. Stickprovets medelvärde är en **punktuppskattning**. Ett **konfidensintervall (KI)** visar ett intervall av rimliga värden utifrån stickprovet och en vald statistisk metod.

Efter lektionen ska du kunna:

- tolka ett 95-procentigt konfidensintervall korrekt
- förstå medelvärde, standardavvikelse, standardfel och felmarginal
- skilja histogram med antal från histogram med täthet
- skilja datafördelning, stickprovsfördelning och bootstrapfördelning
- beräkna intervall med normalapproximation och bootstrap
- visualisera och jämföra metoderna
- förstå varför representativitet och metodens antaganden spelar roll

---

## 1. Repetition: medelvärde, variation och osäkerhet

Anta att vi vill veta den genomsnittliga längden i en stor population. Vi mäter 40 personer och beräknar medelvärdet. Ett annat slumpmässigt urval av 40 personer skulle förmodligen ge ett lite annat medelvärde. Det är **stickprovsvariation**.

| Begrepp | Betyder | Python |
|---|---|---|
| Population | Hela gruppen vi vill uttala oss om | `population` |
| Stickprov | Observationerna vi faktiskt undersöker | `sample` |
| Populationsmedelvärde, μ | Det verkliga medelvärdet, oftast okänt | `true_mean` i simuleringen |
| Stickprovsmedelvärde, x̄ | Medelvärdet från stickprovet | `np.mean(sample)` |
| Stickprovsstandardavvikelse, s | Hur enskilda värden varierar | `np.std(sample, ddof=1)` |
| Standardfel, SE | Hur stickprovsmedelvärden förväntas variera | `s / np.sqrt(n)` |

**Viktigt:** Standardavvikelse handlar om spridningen mellan *personernas värden*. Standardfel handlar om osäkerheten i *det beräknade medelvärdet*.

Formeln för ett uppskattat standardfel är:

\[
SE = \frac{s}{\sqrt{n}}
\]

Om vi fyrdubblar `n` blir standardfelet ungefär hälften så stort, om spridningen är densamma.

## 2. Skapa exempeldata och ett histogram

Läraren börjar med simulerade kroppslängder för 1 000 personer, med ett medelvärde kring 172 cm och standardavvikelse kring 7 cm.

```python
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(42)  # Samma slumpvärden vid samma körordning

heights = np.random.normal(loc=172, scale=7, size=1000)

fig, ax = plt.subplots(figsize=(8, 4))
ax.hist(heights, bins=25, edgecolor="black")
ax.set_title("Histogram of heights")
ax.set_xlabel("Height (cm)")
ax.set_ylabel("Count")
ax.grid(axis="y", alpha=0.3)
plt.show()
```

**Kodförklaring:**

- `np.random.seed(42)` gör slumpsekvensen reproducerbar. Om du kör ytterligare slumpanrop förändras nästa värden även om samma seed användes tidigare.
- `np.random.normal(...)` genererar tal från en normalfördelning. `loc` är fördelningens medelvärde, `scale` dess standardavvikelse och `size` antalet tal.
- `fig, ax = plt.subplots(...)` skapar en figur och en axel att rita i.
- `bins=25` delar in observationerna i 25 intervall.
- `edgecolor="black"` gör staplarnas kanter tydligare.

**Testa:** Ändra `bins` till 10 respektive 40. Fler staplar ger mer detalj, men kan också ge ett brusigare histogram.

### Beräkna sammanfattande statistik

```python
mean_height = np.mean(heights)
median_height = np.median(heights)
s_height = np.std(heights, ddof=1)

print(f"Mean: {mean_height:.2f} cm")
print(f"Median: {median_height:.2f} cm")
print(f"Sample standard deviation: {s_height:.2f} cm")
```

`ddof=1` använder stickprovsstandardavvikelsen. Med `ddof=0` beräknar NumPy standardavvikelsen med division med `n`, vilket ofta används när man beskriver hela den observerade populationen.

### Vad händer med extremvärden?

I videon ersätts de fem första värdena med ovanliga längder för att visa att medelvärdet påverkas mer än medianen.

```python
heights_with_outliers = heights.copy()
heights_with_outliers[:5] = [145, 198, 205, 50, 190]

print("Original mean:", np.mean(heights))
print("Mean with outliers:", np.mean(heights_with_outliers))
print("Original median:", np.median(heights))
print("Median with outliers:", np.median(heights_with_outliers))
```

`[:5]` betyder index 0 till och med 4, alltså de första fem observationerna. `copy()` gör att originalarrayen inte ändras. De extrema värdena är ett demonstrationsexempel, inte rimliga antaganden om alla kroppslängder.

## 3. Histogram med antal eller täthet

Normalt visar `ax.hist()` antal observationer per intervall. Med `density=True` normaliseras staplarna så att deras **sammanlagda area blir 1**. Då kan histogrammet jämföras med en teoretisk täthetskurva.

```python
fig, ax = plt.subplots(figsize=(8, 4))
ax.hist(heights, bins=25, density=True, edgecolor="black", alpha=0.6)
ax.set_xlabel("Height (cm)")
ax.set_ylabel("Density")
plt.show()
```

**Tänk på:** Täthet är inte samma sak som antal. Inte heller är höjden på en täthetskurva sannolikheten för ett exakt värde. För kontinuerliga variabler motsvarar sannolikheter **arean under kurvan över ett intervall**.

### Rita en normalkurva ovanpå histogrammet

Läraren skriver en egen funktion för normalfördelningens täthet (PDF = *probability density function*):

```python
def normal_pdf(x, mean, sd):
    x = np.asarray(x, dtype=float)
    if sd <= 0 or not np.isfinite(sd):
        raise ValueError("sd must be a positive, finite number")

    factor = 1.0 / (sd * np.sqrt(2.0 * np.pi))
    z = (x - mean) / sd
    return factor * np.exp(-0.5 * z**2)

x_support = np.linspace(heights.min() - 5, heights.max() + 5, 400)

fig, ax = plt.subplots(figsize=(8, 4))
ax.hist(heights, bins=25, density=True, edgecolor="black", alpha=0.6,
        label="Observed heights")
ax.plot(x_support, normal_pdf(x_support, mean_height, s_height),
        color="red", linewidth=2, label="Fitted normal curve")
ax.set_xlabel("Height (cm)")
ax.set_ylabel("Density")
ax.set_title("Histogram and normal curve")
ax.legend()
plt.show()
```

**Varför de här delarna?**

- `np.asarray(..., dtype=float)` gör om indata till numeriska värden.
- `np.isfinite(sd)` kontrollerar att standardavvikelsen inte är oändlig eller `NaN`.
- `raise ValueError(...)` stoppar funktionen med ett begripligt felmeddelande om `sd` är ogiltig.
- `np.linspace(start, stop, 400)` skapar 400 jämnt fördelade x-värden, så kurvan blir jämn.
- `z = (x - mean) / sd` anger avståndet från medelvärdet mätt i standardavvikelser.
- `np.exp(...)` ingår i normalfördelningens täthetsformel.

En kurva som följer histogrammets form kan tyda på att normalfördelningen är en rimlig modell. Om datan är skev, flertoppig eller innehåller stora extremvärden bör vi granska modellantagandet extra noga.

## 4. Tre olika fördelningar

1. **Datafördelning:** Hur enskilda observationer varierar. Exempel: personernas längder.
2. **Stickprovsfördelning:** Hur ett medelvärde skulle variera om vi tog många oberoende stickprov från populationen.
3. **Bootstrapfördelning:** Hur medelvärdet varierar när vi upprepade gånger drar med återläggning från **det stickprov vi redan har**.

De två senare visar fördelningar av **medelvärden**, inte av enskilda personers längder. Bootstrapfördelningen används för att uppskatta osäkerheten när populationen är okänd.

## 5. Vad betyder 95 % konfidens?

Ett 95-procentigt konfidensintervall konstrueras med en metod som, vid upprepade stickprov och om antagandena håller, **täcker den verkliga parametern ungefär 95 % av gångerna**.

Det betyder **inte** att:

- 95 % av personerna ligger inom intervallet
- 95 % av alla observationer ligger där
- det är 95 % sannolikhet att den fasta populationsparametern ligger i just det redan beräknade intervallet

Det verkliga medelvärdet är fast; intervallets gränser förändras med stickprovet. I en simulering vet vi ibland det verkliga medelvärdet och kan kontrollera om intervallet råkar täcka det. I verkligheten vet vi oftast inte.

## 6. Metod 1: normalapproximation

För ett tvåsidigt 95-procentigt intervall används i videon:

\[
\bar{x} \pm 1{,}96 \cdot \frac{s}{\sqrt{n}}
\]

Här är `1.96` ett kritiskt z-värde och `1.96 * SE` är **felmarginalen** (halva intervallets bredd).

### Skapa population och stickprov

Vi skapar en population med 500 000 värden, med ett ungefärligt medelvärde på 50 och standardavvikelse 10. Därefter drar vi 40 observationer.

```python
np.random.seed(23)
population = np.random.normal(loc=50, scale=10, size=500_000)
true_mean = float(np.mean(population))

n = 40
sample = np.random.choice(population, size=n, replace=True)

sample_mean = float(np.mean(sample))
s = float(np.std(sample, ddof=1))
se = s / np.sqrt(n)

print(f"Sample mean: {sample_mean:.2f}")
print(f"Standard error: {se:.2f}")
print(f"Simulated true mean: {true_mean:.2f}")
```

**Varför `replace=True`?** Samma populationsvärde får väljas mer än en gång. Det gör dragningarna oberoende i den här simuleringen. I verkliga urval utan återläggning kan en ändlig-populationskorrektion bli aktuell när en stor andel av populationen undersöks.

### Gör en återanvändbar funktion

```python
def ci_mean_normal(x, confidence=0.95):
    """Beräkna ett approximativt 95 %-KI för ett medelvärde."""
    x = np.asarray(x, dtype=float)
    if len(x) < 2:
        raise ValueError("At least two observations are required")
    if confidence != 0.95:
        raise ValueError("This example uses z=1.96 only for 95% confidence")

    mean_x = float(np.mean(x))
    s = float(np.std(x, ddof=1))
    n = len(x)
    z_critical = 1.96
    half_width = z_critical * s / np.sqrt(n)

    low = mean_x - half_width
    high = mean_x + half_width
    return low, high, mean_x, s, n

low, high, mean_x, s, n = ci_mean_normal(sample)
print(f"95% normal CI: [{low:.2f}, {high:.2f}]")
print(f"Sample mean: {mean_x:.2f}, n={n}")
```

**Kodförklaring:** `len(x)` räknar observationer; `half_width` är felmarginalen; `return` skickar tillbaka fem värden. På vänster sida packar vi upp dem i fem variabler i samma ordning.

**Statistisk fördjupning:** När populationsstandardavvikelsen är okänd och uppskattas med `s`, används ofta ett **t-intervall**, särskilt vid små stickprov. Videon använder z-approximationen för att visa grundidén. Ett representativt urval och rimliga antaganden om oberoende och stickprovsfördelning behövs för att tolkningen ska hålla.

### Rita konfidensintervallet med en felstapel

```python
fig, ax = plt.subplots(figsize=(6, 4))
ax.errorbar(0, mean_x, yerr=[[mean_x - low], [high - mean_x]],
            fmt="o", capsize=6, label="95% confidence interval")
ax.axhline(true_mean, color="red", linestyle="--",
           label="Simulated true mean")
ax.set_xticks([])
ax.set_ylabel("Mean")
ax.set_title("95% CI – normal approximation")
ax.grid(axis="y", alpha=0.3)
ax.legend()
plt.show()
```

`yerr` får här två avstånd: från punkten till den nedre gränsen och från punkten till den övre gränsen. `capsize` ritar små tvärstreck vid felstapelns ändar. Den röda linjen finns för att vi **i simuleringen** känner populationsmedelvärdet.

## 7. Metod 2: bootstrap

Bootstrap börjar med **vårt observerade stickprov**, inte med hela populationen. Vi drar många nya stickprov med **återläggning**, vart och ett lika stort som originalet. För varje nytt stickprov beräknar vi medelvärdet.

Om originalet exempelvis är `[10, 20, 30]` kan ett bootstrapstickprov vara `[20, 20, 10]`. Att ett värde upprepas är helt avsiktligt.

### Funktion för bootstrapintervall

```python
def ci_mean_bootstrap(x, B=3000, confidence=0.95):
    """Percentil-bootstrap för medelvärdet."""
    x = np.asarray(x, dtype=float)
    n = len(x)
    if n < 2 or B < 1 or not 0 < confidence < 1:
        raise ValueError("Check sample size, B and confidence")

    boot_means = np.empty(B)

    for i in range(B):
        boot_sample = np.random.choice(x, size=n, replace=True)
        boot_means[i] = np.mean(boot_sample)

    alpha = (1 - confidence) / 2
    low, high = np.percentile(boot_means,
                              [100 * alpha, 100 * (1 - alpha)])
    return float(low), float(high), float(np.mean(x)), boot_means

b_low, b_high, b_mean, boot_means = ci_mean_bootstrap(sample, B=3000)
print(f"95% bootstrap CI: [{b_low:.2f}, {b_high:.2f}]")
```

**Steg för steg:**

1. `np.empty(B)` reserverar plats för `B` medelvärden. Arrayen måste fyllas innan vi använder den.
2. `for i in range(B)` upprepar beräkningen `B` gånger.
3. `np.random.choice(x, size=n, replace=True)` drar från **stickprovet** och tillåter dubbletter.
4. `boot_means[i] = ...` sparar varje nytt medelvärde på rätt index.
5. `alpha = (1 - 0.95) / 2` blir `0.025`.
6. `np.percentile(..., [2.5, 97.5])` tar gränserna för ett **percentil-bootstrapintervall**.

**Observera:** Bootstrap är flexibel, men inte magisk. Den kan inte reparera ett snedvridet urval eller dåliga mätningar. Percentilmetoden är inte alltid tillförlitlig för mycket små stickprov, kraftig skevhet eller extrema observationer. Den är ett pedagogiskt exempel på en bootstrapmetod, inte en garanti att alla bootstrapintervall fungerar bättre än normalmetoden.

### Rita bootstrapfördelningen

```python
fig, ax = plt.subplots(figsize=(8, 4))
ax.hist(boot_means, bins=30, edgecolor="black", alpha=0.7)
ax.axvline(b_mean, color="green", linestyle="--", label="Sample mean")
ax.axvline(b_low, color="red", linestyle="--", label="2.5th percentile")
ax.axvline(b_high, color="red", linestyle="--", label="97.5th percentile")
ax.set_title("Bootstrap distribution of sample means")
ax.set_xlabel("Resampled mean")
ax.set_ylabel("Count")
ax.grid(axis="y", alpha=0.3)
ax.legend()
plt.show()
```

**Tolkning:** Varje stapel visar hur många bootstrapmedelvärden som hamnar inom ett visst intervall. De röda linjerna markerar den centrala 95-procentiga delen av den simulerade bootstrapfördelningen.

## 8. Jämför metoderna

```python
print(f"Normal:    [{low:.2f}, {high:.2f}]")
print(f"Bootstrap: [{b_low:.2f}, {b_high:.2f}]")
print(f"Sample mean: {sample_mean:.2f}")
print(f"True mean (simulation): {true_mean:.2f}")
```

| Normalapproximation | Bootstrap |
|---|---|
| Använder formeln `mean ± 1.96 * SE` | Drar många nya stickprov från originalstickprovet |
| Snabb att beräkna | Kräver upprepade beräkningar |
| Bygger på att stickprovsfördelningen kan approximeras väl | Uppskattar osäkerheten empiriskt från stickprovet |
| Ofta rimlig för större, välbeteende urval | Flexibel, men har också antaganden och begränsningar |

I lärarens exempel blir resultaten ofta ganska lika eftersom populationen är normalfördelad och stickprovet har en måttlig storlek.

## 9. Coverage – hur ofta täcker intervallet det sanna medelvärdet?

Eftersom vi har simulerat populationen vet vi dess medelvärde. Vi kan därför dra många stickprov och se hur ofta intervallen innehåller detta värde.

```python
def covers_true_mean(method="normal", n=40, trials=200):
    hits = 0

    for _ in range(trials):
        new_sample = np.random.choice(population, size=n, replace=True)

        if method == "normal":
            low_ci, high_ci, *_ = ci_mean_normal(new_sample)
        elif method == "bootstrap":
            low_ci, high_ci, *_ = ci_mean_bootstrap(new_sample, B=1500)
        else:
            raise ValueError("method must be 'normal' or 'bootstrap'")

        if low_ci <= true_mean <= high_ci:
            hits += 1

    return hits / trials

print("Normal coverage:", covers_true_mean("normal"))
print("Bootstrap coverage:", covers_true_mean("bootstrap"))
```

- `trials=200`: vi tar 200 **nya stickprov från populationen**.
- `B=1500`: för varje stickprov skapar bootstrapmetoden 1 500 **återläggningsstickprov från just det stickprovet**.
- `hits += 1`: räknaren ökar när intervallet innehåller `true_mean`.
- `hits / trials`: andelen intervall som täcker populationsmedelvärdet.
- `*_` i uppackningen: de första två returvärdena sparas och resten ignoreras.

Resultaten varierar lite mellan körningar. De behöver inte bli exakt 0,95, särskilt inte med bara 200 försök. Körningen med bootstrap kan ta en stund eftersom många beräkningar görs.

## 10. Vad påverkar intervallets bredd?

- **Större `n`** → vanligtvis mindre standardfel och smalare intervall.
- **Större variation i datan** → vanligtvis bredare intervall.
- **Högre konfidensgrad** → vanligtvis bredare intervall.
- **Snedvridet urval** → kan ge missvisande slutsatser även om intervallet är smalt.

Ett smalt intervall innebär alltså högre **precision**, inte automatiskt högre **riktighet**.

## 11. Så redovisar du resultatet

Ett tydligt resultat bör innehålla **vad** som uppskattas, **punktuppskattningen**, **intervallet**, **konfidensgraden**, **urvalsstorleken** och **metoden**.

> Det uppskattade medelvärdet var **X**, med ett **95-procentigt konfidensintervall från L till H**, beräknat med **percentil-bootstrap** utifrån **n = 40 observationer**.

Beskriv gärna också urvalets representativitet, eventuella extremvärden och om metodens antaganden verkar rimliga.

## 12. Vanliga misstag

| Misstag | Varför blir det fel? | Rätt sätt |
|---|---|---|
| `np.std(x)` när du vill uppskatta stickprovets standardavvikelse | Standard är `ddof=0` | `np.std(x, ddof=1)` |
| `replace=False` i bootstrap | Bootstrap bygger på återläggning | `replace=True` |
| Bootstrap från hela populationen | I verkligheten känner vi normalt bara stickprovet | Bootstrap från `sample` |
| `density=True` men y-axeln märks `Count` | Y-axeln visar täthet | Märk den `Density` |
| `np.percentile(..., [0.025, 0.975])` | `percentile` förväntar sig värden mellan 0 och 100 | `[2.5, 97.5]` |
| Att blanda ihop `s` och `SE` | De beskriver olika typer av variation | `SE = s / sqrt(n)` |
| Att säga att 95 % av personerna ligger i KI | KI avser en parameter, inte individerna | Förklara upprepade intervalls täckningsgrad |
| Att kalla ett snävt KI för bevis på representativitet | Systematiska fel kan finnas kvar | Granska hur urvalet gjordes |
| `np.random.choice(..., raplace=True)` | Felstavat argument | `replace=True` |
| Att använda `1.96` för varje konfidensgrad | Det kritiska värdet beror på graden | Här används 1,96 bara för 95 % |

## 13. Övningar

**Övning 1 – Begrepp.** Förklara skillnaden mellan standardavvikelse och standardfel med egna ord.

**Övning 2 – Beräkna.** Ett stickprov har `n = 100`, medelvärde 50 och standardavvikelse 10. Beräkna SE och ett approximativt 95 %-KI med z = 1,96.

**Övning 3 – Förstå kod.** Vad är skillnaden mellan `replace=True` och `replace=False`? Varför används `True` vid bootstrap?

**Övning 4 – Histogram.** Vad förändras när du skriver `density=True` i `ax.hist()`?

**Övning 5 – Tolka.** En undersökning ger ett 95 %-KI på [48, 52]. Betyder det att 95 % av observationerna ligger mellan 48 och 52? Förklara.

**Övning 6 – Jämför.** Beräkna normal- och bootstrapintervall för `sample`. Blir gränserna exakt lika? Varför eller varför inte?

**Övning 7 – Urvalsstorlek.** Om du ökar `n` från 40 till 160, vad händer ungefär med SE om standardavvikelsen är densamma?

**Övning 8 – Kodläsning.** Vad innehåller `boot_means[i]` efter ett varv i bootstraploopen?

**Övning 9 – Coverage.** Om 188 av 200 intervall täcker det verkliga medelvärdet, vad är den observerade täckningsgraden? Är det ett tecken på att metoden är fel?

**Övning 10 – Metodval.** Varför behöver du granska datans form och urvalsmetoden innan du väljer metod och tolkar resultatet?

## 14. Facit

1. Standardavvikelse beskriver spridningen mellan enskilda observationer. Standardfel beskriver hur ett uppskattat medelvärde varierar mellan tänkbara stickprov.
2. `SE = 10 / sqrt(100) = 1`. Felmarginalen är `1.96 * 1 = 1.96`. Intervallet blir `[48.04, 51.96]`.
3. `True` tillåter att samma observation väljs flera gånger. Det är grundidén i vanlig bootstrap.
4. Staplarna skalas så att deras totala area är 1; y-axeln visar täthet i stället för antal.
5. Nej. Intervallet gäller osäkerheten kring en populationsparameter, inte var 95 % av individerna ligger.
6. De kan vara lika men blir oftast inte exakt lika: normalmetoden använder en formel, bootstrap använder en empirisk fördelning från upprepade dragningar.
7. SE halveras ungefär eftersom `sqrt(160/40) = 2`.
8. Medelvärdet för det bootstrapstickprov som drogs i just det varvet.
9. `188 / 200 = 0.94`, alltså 94 %. Det kan bero på slumpvariation; 200 försök ger inte exakt 95 % varje gång.
10. Olika metoder har olika antaganden. Ingen av dem kan kompensera för exempelvis ett systematiskt snedvridet urval.

## Quick Reference

```python
import numpy as np
import matplotlib.pyplot as plt

# Statistik
mean_x = np.mean(sample)
s = np.std(sample, ddof=1)
n = len(sample)
se = s / np.sqrt(n)

# Approximativt 95 %-KI med z-metoden
low = mean_x - 1.96 * se
high = mean_x + 1.96 * se

# Ett bootstrapstickprov (samma storlek, med återläggning)
boot_sample = np.random.choice(sample, size=n, replace=True)

# Percentilgränser från en array med bootstrapmedelvärden
boot_low, boot_high = np.percentile(boot_means, [2.5, 97.5])

# Histogram med täthet
# ax.hist(sample, bins=25, density=True)
```

**Kom ihåg:** Konfidensintervallet beskriver osäkerheten kring vår uppskattning. Det kan bli smalare med mer data, men ett stort eller precist stickprov är inte automatiskt representativt.

---

**Källmaterial:** Skolans lektionstext *Konfidensintervall* och transkriberingen av videon *Konfidensintervall* (cirka 1 h 17 min). Kod har skrivits om till sammanhängande, körbara exempel och kompletterats med förklaringar. Vissa metodbegränsningar har förtydligats.
