# Python # – Sampling och variation

> **Syfte:** Förstå varför medelvärdet varierar mellan stickprov, vad standardfel mäter och hur konfidensintervall och permutationstest fungerar.
>
> **Källa:** Lärarens video *Python #30 – Sampling och Variation*. Förklaringarna och kodexemplen nedan är pedagogiskt rekonstruerade från transkriberingen, inte en ordagrann kodavskrift. Några otydliga eller felsagda detaljer har rättats och markeras särskilt.

## 1. Population och stickprov

En **population** är alla individer eller observationer vi vill undersöka. Ett **stickprov** är en mindre del av populationen.

Vi känner oftast inte populationens exakta medelvärde i verkligheten. Därför tar vi ett stickprov och beräknar dess medelvärde som en uppskattning. Men om vi väljer ett nytt stickprov får vi sannolikt ett *annat* medelvärde – även om båda kommer från samma population.

**Exempel:** En stad har en miljon invånare. Vi frågar 30 slumpmässigt valda personer om en viss siffra. Om vi frågar 30 andra personer kommer deras genomsnitt sannolikt att skilja sig lite. Det behöver inte bero på att något ändrats i staden, utan på **slumpmässig variation i urvalet**.

### Fyra begrepp att hålla isär

| Begrepp | Vad betyder det? | Exempel |
|---|---|---|
| **Population** | Alla vi vill undersöka | Alla invånare i en stad |
| **Parameter** | Ett mått för hela populationen, ofta okänt | Stadens verkliga genomsnittliga inkomst |
| **Stickprov** | De observationer vi faktiskt samlar in | 200 slumpmässigt utvalda invånare |
| **Stickprovsstatistik** | Ett mått beräknat från stickprovet | Medelinkomsten för de 200 personerna |

**Så hänger det ihop:** Population → stickprov → stickprovsmedelvärde. Om vi tar många nya stickprov får vi många olika medelvärden. Fördelningen av dessa medelvärden kallas **stickprovsfördelning** (eller *samplingfördelning*).

**Viktigt:** Stickprovsfördelningen visar hur ett *beräknat mått* varierar, inte hur enskilda observationer är fördelade.

### Vanliga statistiska symboler

| Symbol | Betydelse |
|---|---|
| **n** | Antalet observationer i ett stickprov |
| **μ** (my) | Populationens verkliga medelvärde |
| **x̄** (x-streck) | Stickprovets medelvärde |
| **s** | Stickprovets standardavvikelse |
| **SE** | Standardfel: variationen i en skattning mellan stickprov |

Du behöver inte memorera alla symboler direkt. Målet är att känna igen dem när de dyker upp i formler och kod.

## 2. Skapa en simulerad population

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

np.random.seed(0)  # Gör de slumpmässiga resultaten reproducerbara

true_mean = 7000
true_sd = 1800

pop = np.random.normal(loc=true_mean, scale=true_sd, size=1_000_000)

print(pd.Series(pop).describe())
```

**Varför gör vi detta?** I simuleringen vet vi hur populationen skapades. Vi kan därför jämföra våra stickprov med den *sanna nivån* 7000. I verkliga undersökningar känner vi oftast inte den nivån.

- `np.random.seed(0)` gör att samma slumpsekvens kan återskapas.
- `np.random.normal(...)` skapar tal från en normalfördelning.
- `loc=true_mean` anger fördelningens medelvärde (7000).
- `scale=true_sd` anger fördelningens standardavvikelse (1800).
- `size=1_000_000` anger hur många värden som skapas.
- `pd.Series(pop).describe()` ger bland annat antal, medelvärde, standardavvikelse och kvartiler.

**Viktigt:** Den simulerade populationens faktiska medelvärde blir *nära* 7000, inte nödvändigtvis exakt 7000.

## 3. Ta ett slumpmässigt stickprov

```python
def sample_mean(n):
    sample = np.random.choice(pop, size=n, replace=False)
    return sample.mean()

print(sample_mean(30))
print(sample_mean(30))
```

`n` är stickprovets storlek. `np.random.choice` väljer slumpmässiga observationer ur `pop`. `replace=False` betyder att samma populationsposition inte kan väljas två gånger *inom samma stickprov*. `.mean()` beräknar medelvärdet.

**Varför en funktion?** Vi ska upprepa samma operation många gånger, och vill kunna ändra stickprovsstorleken utan att skriva om koden.

## 4. Upprepa 2 000 stickprov

```python
rounds = 2000

means_30 = np.array([sample_mean(30) for _ in range(rounds)])
means_200 = np.array([sample_mean(200) for _ in range(rounds)])
```

Här skapas **2 000 medelvärden** från stickprov med 30 observationer, och **2 000 medelvärden** från stickprov med 200 observationer.

`for _ in range(rounds)` upprepar beräkningen 2 000 gånger. Understrecket `_` betyder att vi inte behöver använda loopvariabeln. List comprehension samlar resultaten i en lista; `np.array(...)` omvandlar listan till en NumPy-array.

**Skilj på:** `n=30` är *antalet observationer i varje stickprov*. `rounds=2000` är *antalet stickprov vi tar*.

## 5. Rita samplingfördelningen

```python
fig, ax = plt.subplots(figsize=(8, 5))

ax.hist(means_30, bins=30, alpha=0.6, label="n = 30")
ax.hist(means_200, bins=30, alpha=0.6, label="n = 200")
ax.axvline(true_mean, color="black", linestyle="--", label="Sann nivå")

ax.set_title("Samplingfördelning av medelvärdet")
ax.set_xlabel("Stickprovsmedelvärde")
ax.set_ylabel("Antal")
ax.grid(axis="y", alpha=0.3)
ax.legend()
plt.show()
```

**Vad visar histogrammet?** Det visar hur *medelvärdena* fördelas när vi tar många stickprov. Det visar alltså inte fördelningen av alla enskilda personer i populationen.

- För `n=30` blir spridningen mellan stickprovsmedelvärdena större.
- För `n=200` samlas medelvärdena tätare kring den sanna nivån.
- `bins=30` delar in histogrammet i 30 intervall; det är **inte** stickprovsstorleken.
- `alpha=0.6` gör staplarna delvis genomskinliga.
- `ax.axvline(...)` ritar den lodräta referenslinjen vid 7000.

Den här fördelningen av upprepade stickprovsmedelvärden kallas **samplingfördelning** (*sampling distribution*).

## 6. Standardavvikelse kontra standardfel

Det här är lektionens viktigaste begreppsskillnad:

| Begrepp | Vad varierar? | Fråga det besvarar |
|---|---|---|
| **Standardavvikelse (SD)** | Enskilda observationer | Hur utspridda är värdena i populationen eller stickprovet? |
| **Standardfel (SE)** | Stickprovsmedelvärden | Hur mycket brukar medelvärdet variera mellan nya stickprov? |

För oberoende observationer är standardfelet för medelvärdet ungefär:

\[
SE = \frac{SD}{\sqrt{n}}
\]

```python
sd = np.std(pop, ddof=1)

se_30 = sd / np.sqrt(30)
se_200 = sd / np.sqrt(200)

print("Standardfel n=30:", round(se_30, 1))
print("Standardfel n=200:", round(se_200, 1))
```

Med `SD ≈ 1800` blir standardfelet ungefär **329** för `n=30` och **127** för `n=200`.

`np.sqrt(n)` beräknar kvadratroten av stickprovsstorleken. `ddof=1` beräknar stickprovsstandardavvikelse; i just denna mycket stora simulerade population gör det nästan ingen skillnad jämfört med `ddof=0`.

**Varför minskar SE?** När vi beräknar medelvärdet av fler observationer jämnas slumpmässiga variationer ut. Om vi fyrdubblar `n` halveras SE ungefär, eftersom `sqrt(4n) = 2sqrt(n)`.

### Precision är inte samma sak som representativitet

När vi ökar antalet observationer minskar vanligtvis den **slumpmässiga variationen** i medelvärdet. Det kallas högre **precision**. Men ett stort stickprov kan ändå vara missvisande om urvalet är snedvridet.

**Exempel:** Vi vill undersöka hur alla invånare i en stad reser till arbetet. Om vi bara frågar personer på en cykelparkering kan resultatet överskatta cyklandet, även om vi frågar 10 000 personer.

- **Precision:** hur mycket resultatet skulle variera mellan upprepade stickprov.
- **Bias (systematiskt fel):** när urvalsmetoden tenderar att dra resultatet åt ett visst håll.
- **Representativitet:** hur väl urvalet speglar populationen för den fråga vi undersöker.

**Kom ihåg:** Ett mindre standardfel löser inte automatiskt problem med bias.

### Centrala gränsvärdessatsen – varför medelvärden ofta får en klockform

Videons simulerade population är normalfördelad från början. Men även om de *enskilda observationerna* inte är normalfördelade kan **fördelningen av stickprovsmedelvärden** bli ungefär normal när stickproven är tillräckligt stora, under vanliga statistiska förutsättningar. Det är kärnan i **centrala gränsvärdessatsen**.

Det betyder **inte** att alla värden i den ursprungliga populationen blir normalfördelade. Det är *medelvärdena från många stickprov* som får en ungefärlig klockform.

**Exempel:** Enskilda köpbelopp i en webbutik kan vara mycket ojämnt fördelade. Men om vi upprepar ett slumpmässigt urval av tillräckligt många köp och beräknar medelvärdet varje gång, kan dessa medelvärden få en ungefär normal fördelning.

*Komplettering från skolans lektionsmaterial: representativitet, bias och centrala gränsvärdessatsen utvecklas här utöver videons huvudsakliga kodexempel.*

## 7. Ett 95-procentigt konfidensintervall

Ett **konfidensintervall (KI)** ger ett intervall av rimliga värden för ett okänt populationsmedelvärde, baserat på ett stickprov.

I videon introduceras ett ungefärligt 95-procentigt intervall:

\[
\bar{x} \pm 1.96 \cdot \frac{s}{\sqrt{n}}
\]

Där `x̄` är stickprovets medelvärde, `s` dess standardavvikelse och `n` antalet observationer.

```python
def ci_95(x):
    x = pd.Series(x)
    mean = x.mean()
    sd = x.std(ddof=1)
    se = sd / np.sqrt(len(x))
    z = 1.96
    return mean - z * se, mean + z * se

sample_60 = np.random.choice(pop, size=60, replace=False)
low, high = ci_95(sample_60)

print("Stickprovsmedelvärde:", round(sample_60.mean(), 1))
print("95 % KI:", round(low, 1), "till", round(high, 1))
print("Sann nivå i simuleringen:", true_mean)
```

**Vad betyder 95 %?** Om vi upprepar samma urvalsmetod många gånger och beräknar ett nytt intervall varje gång, kommer ungefär 95 % av dessa intervall att innehålla det sanna populationsmedelvärdet (när metodens antaganden är uppfyllda). Det betyder **inte** att det finns 95 % sannolikhet att ett redan beräknat, fast intervall innehåller det fasta sanna värdet.

**Förtydligande av transkriberingen:** Där hörs ungefär `1,6`, men för ett tvåsidigt 95-procentigt normalbaserat konfidensintervall används **1,96**. För små stickprov med okänd populationsstandardavvikelse används ofta en *t-fördelning* i stället. Här följer vi videons introducerande normalapproximation.

## 8. Testa hur ofta intervallen träffar rätt: coverage

I simuleringen vet vi att den sanna nivån är 7000. Därför kan vi räkna hur ofta konfidensintervallen täcker den.

```python
def coverage(n, rounds=1000):
    ok = 0

    for _ in range(rounds):
        x = np.random.choice(pop, size=n, replace=False)
        low, high = ci_95(x)
        ok += (low <= true_mean <= high)

    return ok / rounds

print("Coverage n=30:", coverage(30))
print("Coverage n=200:", coverage(200))
```

**Steg för steg:**

1. Sätt `ok = 0` för att räkna lyckade intervall.
2. Ta ett nytt stickprov.
3. Beräkna dess konfidensintervall.
4. Kontrollera om `7000` ligger mellan `low` och `high`.
5. Öka räknaren med 1 när villkoret är sant.
6. Dividera antalet träffar med antalet upprepningar.

`ok += (low <= true_mean <= high)` fungerar eftersom `True` räknas som 1 och `False` som 0 i Python.

I videon blev resultaten ungefär **94 %** för `n=30` och **95,6 %** för `n=200`. Ditt resultat kan variera beroende på slumpsekvens och kodens körordning.

## 9. Introduktion till hypotesprövning

En **hypotesprövning** undersöker om ett observerat resultat verkar ovanligt under ett visst antagande, en **nollhypotes** (`H0`).

Exempel: Vi vill undersöka om två grupper kan ha samma fördelning, och jämför deras medelvärden.

- **Nollhypotes (H0):** Gruppetiketterna spelar ingen roll; gruppernas värden är utbytbara under nollhypotesen.
- **Observerad skillnad:** `mean(B) - mean(A)`.
- **p-värde:** Hur ofta får vi en skillnad som är minst lika extrem som den observerade, om nollhypotesen gäller?

Ett stort p-värde betyder att resultatet inte är särskilt ovanligt under nollhypotesen. Ett litet p-värde betyder att det är ovanligt. Ett stort p-värde **bevisar inte** att grupperna är lika.

## 10. Permutationstest – blanda grupperna

```python
A = np.random.choice(pop, size=60, replace=False)
B = np.random.choice(pop, size=60, replace=False)

observed = B.mean() - A.mean()
all_values = np.concatenate([A, B])
n_a = len(A)
rounds = 5000

permuted_differences = np.empty(rounds)

for i in range(rounds):
    shuffled = np.random.permutation(all_values)
    permuted_differences[i] = (
        shuffled[n_a:].mean() - shuffled[:n_a].mean()
    )

p_value = np.mean(np.abs(permuted_differences) >= abs(observed))
print("Observerad skillnad:", round(observed, 1))
print("p-värde:", round(p_value, 4))
```

**Vad händer?**

1. `A` och `B` är två stickprov.
2. `observed` sparar den faktiska skillnaden i deras medelvärden.
3. `np.concatenate([A, B])` slår ihop värdena.
4. `np.random.permutation(...)` blandar värdena slumpmässigt.
5. `shuffled[:n_a]` blir en ny låtsasgrupp A och `shuffled[n_a:]` en ny låtsasgrupp B.
6. Vi sparar skillnaden mellan de omblandade grupperna.
7. Efter 5 000 omblandningar räknar vi andelen skillnader som är **minst lika extrema i absolutvärde** som den observerade skillnaden.

`np.abs(...)` gör testet **tvåsidigt**: både stora positiva och stora negativa skillnader räknas.

I videon fick läraren **p ≈ 0,7904**, ett stort p-värde. Det innebär att en minst lika stor skillnad var vanlig i permutationerna; testet ger då inte stark evidens mot nollhypotesen.

**Viktigt förtydligande:** Videon jämför också ett konfidensintervall kring *ett stickprovsmedelvärde och nivån 7000* med ett permutationstest för *skillnaden mellan två grupper*. Det är **två olika hypoteser**, så just de två resultaten kan inte direkt sägas vara samma test. Sambandet mellan konfidensintervall och p-värden gäller när de avser **samma parameter och motsvarande testmetod**.

## 11. Snabbguide – kom ihåg

| Kod/begrepp | Betydelse |
|---|---|
| **n / μ / x̄ / s / SE** | Stickprovsstorlek / populationsmedelvärde / stickprovsmedelvärde / stickprovs-SD / standardfel |
| **Bias** | Systematisk snedvridning som inte försvinner bara för att stickprovet blir större |
| **Centrala gränsvärdessatsen** | Stickprovsmedelvärden blir ofta ungefär normalfördelade för tillräckligt stora stickprov |
| `np.random.seed(0)` | Reproducerbar slumpsekvens |
| `np.random.normal(...)` | Skapa normalfördelade värden |
| `np.random.choice(..., replace=False)` | Slumpa observationer utan återläggning |
| `.mean()` | Medelvärde |
| `.std(ddof=1)` | Stickprovsstandardavvikelse |
| `np.sqrt(n)` | Kvadratroten av `n` |
| `SE = SD / sqrt(n)` | Standardfel för medelvärdet |
| `np.array([...])` | Skapa NumPy-array |
| `np.concatenate([A, B])` | Slå ihop arrayer |
| `np.random.permutation(...)` | Slumpa om ordningen |
| `np.abs(...)` | Absolutvärde |
| `np.empty(rounds)` | Reservera en array som ska fyllas med värden |
| `ax.hist(...)` | Histogram |
| `ax.axvline(...)` | Lodrät referenslinje |

## 12. Vanliga misstag

**1. Blanda ihop antal stickprov med stickprovsstorlek.** `rounds=2000` betyder 2 000 *upprepningar*, inte 2 000 personer per stickprov.

**2. Blanda ihop SD och SE.** SD beskriver spridningen bland individerna; SE beskriver spridningen mellan stickprovens medelvärden.

**3. Skriva `sd(np.sqrt(n))` i stället för `sd / np.sqrt(n)`.** En variabel med ett tal är inte en funktion. Videon visar ett liknande felmeddelande: `numpy.float64 object is not callable`.

**4. Tro att `replace=False` gäller mellan alla upprepningar.** Det gäller bara *inom varje anrop* till `np.random.choice`.

**5. Tolka ett stort p-värde som bevis för H0.** Ett stort p-värde säger bara att data inte är ovanliga under H0 enligt testet.

**6. Tolka 95 % KI som att 95 % av observationerna ligger i intervallet.** Intervallet gäller osäkerheten i **medelvärdet**, inte var 95 % av individerna ligger.

## 13. Övningar

**Övning 1 – Begrepp:** Vad är skillnaden mellan en population och ett stickprov?

**Övning 2 – Kod:** Ändra `means_30` till att använda stickprov med `n=50`. Vilken del av koden behöver ändras?

**Övning 3 – Förstå resultatet:** Varför blir histogrammet för `n=200` smalare än för `n=30`?

**Övning 4 – Beräkning:** Anta att SD är 100. Beräkna SE för `n=25` och `n=100`.

**Övning 5 – Kodläsning:** Förklara vad följande rad räknar:

```python
np.mean(np.abs(permuted_differences) >= abs(observed))
```

**Övning 6 – Tolkning:** Ett permutationstest ger `p=0.79`. Betyder det att H0 är sann med 79 % sannolikhet?

**Övning 7 – Representativitet:** Du frågar 5 000 personer på ett gym hur ofta de tränar och vill dra slutsatser om alla i kommunen. Vad kan bli fel?

**Övning 8 – Symboler:** Vad står `μ` och `x̄` för, och vilken av dem känner vi normalt inte till?

**Övning 9 – Centrala gränsvärdessatsen:** Är det de enskilda observationerna eller medelvärden från många stickprov som tenderar att bli normalfördelade?

## 14. Facit

**1.** Populationen är hela gruppen vi vill studera; stickprovet är de observationer vi faktiskt väljer ut.

**2.** Ändra till `means_50 = np.array([sample_mean(50) for _ in range(rounds)])`. Uppdatera även variabelnamn och histogrametikett om du ritar resultatet.

**3.** Medelvärden från större stickprov varierar mindre mellan upprepningar. Standardfelet minskar när `n` ökar.

**4.** `100 / sqrt(25) = 20`; `100 / sqrt(100) = 10`.

**5.** Den räknar andelen omblandade skillnader som är minst lika stora i absolutvärde som den observerade skillnaden: det simulerade tvåsidiga p-värdet.

**6.** Nej. Det betyder att en minst lika extrem skillnad uppstod i ungefär 79 % av permutationerna under testets nollhypotes.

**7.** Urvalet kan vara snedvridet eftersom personer på gymmet inte nödvändigtvis representerar alla kommuninvånare. Fler svar från samma snedvridna urval tar inte bort bias.

**8.** `μ` är populationens verkliga medelvärde, som normalt är okänt. `x̄` är medelvärdet vi beräknar från stickprovet.

**9.** Det är *medelvärdena från upprepade stickprov* som under vanliga förutsättningar tenderar att få en ungefär normal fördelning.

---

### Sammanfattning av videon

- Olika slumpmässiga stickprov ger olika medelvärden.
- **Samplingfördelningen** beskriver variationen mellan stickprovsmedelvärden.
- **Standardfelet** mäter den typiska variationen i stickprovsmedelvärdet.
- Större stickprov ger i regel mindre standardfel och smalare konfidensintervall.
- Ett **95 % konfidensintervall** är en metod som vid upprepade urval täcker den sanna parametern ungefär 95 % av gångerna.
- Ett **permutationstest** jämför en observerad skillnad med skillnader som uppstår när vi blandar gruppetiketter under nollhypotesen.

*Nästa video enligt läraren: mer om konfidensintervall. Hypotesprövning får också en egen genomgång senare.*
