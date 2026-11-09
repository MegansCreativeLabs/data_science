# Lektion 14 – Power och flerfaldig testning

> **Kurs:** Python och statistisk dataanalys  
> **Video:** [Power & Flerfaldig testning](https://www.youtube.com/watch?v=fmEk4L9IlA8)  
> **Förkunskaper:** hypotesprövning, p-värden, konfidensintervall och A/B-test för andelar.

## Vad ska vi lära oss?

När vi planerar ett experiment vill vi både **upptäcka verkliga skillnader** och **undvika att bli lurade av slumpen**. Det är två olika problem:

1. **Power:** Har experimentet tillräckligt många observationer för att upptäcka en effekt som faktiskt finns?
2. **Flerfaldig testning:** Om vi testar många hypoteser, hur begränsar vi risken att någon ser signifikant ut bara av slumpen?

I den här lektionen går vi igenom teorin och följer lärarens arbetsflöde i videon: först powerberäkning för konverteringsandelar, sedan simulering och visualisering, därefter ett exempel med medelvärden och till sist korrigering av många p-värden.

**Efter lektionen ska du kunna:** skilja typ I- och typ II-fel, förklara alfa, beta och power, välja minsta detekterbar effekt (MDE), beräkna stickprovsstorlek för andelar och medelvärden, simulera power samt använda och tolka Bonferroni och Benjamini–Hochberg (FDR).

---

## 1. Fyra möjliga utfall i ett hypotesprov

Vi börjar med två frågor: **Finns det en verklig effekt?** och **förkastar vi nollhypotesen?**

| Verkligheten | Vi förkastar inte H₀ | Vi förkastar H₀ |
|---|---|---|
| H₀ är sann (ingen effekt) | Korrekt beslut | **Typ I-fel** – falskt positivt |
| H₀ är falsk (effekt finns) | **Typ II-fel** – falskt negativt | Korrekt upptäckt |

### Typ I-fel: alfa (α)

Vi påstår att en skillnad finns trots att den inte gör det. Om testets antaganden är uppfyllda och vi använder `alpha = 0.05`, är risken för typ I-fel högst 5 % för ett enskilt test under nollhypotesen.

### Typ II-fel: beta (β)

Vi missar en effekt som faktiskt finns. Risken för det betecknas `beta`.

### Power

**Power = 1 − beta**.

Om `beta = 0.20` blir `power = 0.80`, det vill säga 80 %.

```python
beta = 0.20
power = 1 - beta
print(f"Power: {power:.0%}")  # Power: 80%
```

**Vad betyder 80 % power?** Om vi upprepar samma experiment många gånger och den *planerade* effekten verkligen finns, kommer ungefär 80 % av experimenten att ge ett signifikant resultat. Det är **inte** sannolikheten att vår hypotes är sann, och powern gäller inte automatiskt för alla tänkbara effektstorlekar.

---

## 2. Varför påverkar alfa, power och stickprovsstorlek varandra?

- **Lägre alfa** (exempelvis 0,01 i stället för 0,05) gör det svårare att förkasta H₀. Det minskar falsklarm men ger normalt lägre power om allt annat är oförändrat.
- **Större stickprov** gör uppskattningarna mer precisa och ökar normalt power.
- **Större verklig effekt** är lättare att upptäcka än en liten effekt.
- **Större variation** gör det svårare att skilja en verklig skillnad från brus.

Vi kan därför inte välja ett önskat antal deltagare utan att först fundera på vilken effekt vi behöver kunna upptäcka.

---

## 3. MDE – minsta detekterbar effekt

**MDE** står för *Minimum Detectable Effect*. I planeringen anger vi den minsta skillnad som är praktiskt viktig och som vi vill ha exempelvis 80 % power att upptäcka.

Lärarens exempel:

- Kontrollgrupp A konverterar **25 %**.
- Vi vill upptäcka en ökning på minst **2 procentenheter**.
- Grupp B behöver därför nå **27 %**.

```python
baseline_rate_a = 0.25
min_detectable_diff_pp = 2.0
min_detectable_diff = min_detectable_diff_pp / 100

target_rate_b = baseline_rate_a + min_detectable_diff

print(f"A: {baseline_rate_a:.1%}")  # 25.0%
print(f"B: {target_rate_b:.1%}")    # 27.0%
```

**Varför dividerar vi med 100?** Två procentenheter skrivs som `0.02` när vi räknar med andelar mellan 0 och 1.

**Procentenheter är inte samma sak som relativ procentuell ökning:**

```python
absolute_diff_pp = (target_rate_b - baseline_rate_a) * 100
relative_increase = (target_rate_b / baseline_rate_a - 1) * 100

print(f"Absolut skillnad: {absolute_diff_pp:.1f} procentenheter")
print(f"Relativ ökning: {relative_increase:.1f} %")
```

Resultat: **2 procentenheter**, men **8 % relativ ökning**.

MDE betyder inte att mindre effekter är omöjliga att hitta. Det betyder att vi **inte har planerat experimentet för att tillförlitligt upptäcka dem**.

---

## 4. Installera och importera biblioteken

Vi använder **NumPy**, **Pandas**, **Matplotlib** och **statsmodels**. SciPy är användbart för andra statistiska analyser.

Installera vid behov i terminalen:

```bash
python -m pip install numpy pandas matplotlib scipy statsmodels
```

Importera i notebooken:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from scipy import stats
from statsmodels.stats.power import NormalIndPower, TTestIndPower
from statsmodels.stats.proportion import (
    proportion_effectsize,
    proportions_ztest,
)
from statsmodels.stats.multitest import multipletests

rng = np.random.default_rng(42)
```

**Vad är vad?**

| Kod | Vad den gör | Varför vi behöver den |
|---|---|---|
| `np` | NumPy: numeriska beräkningar och slumpdata | Simulera experiment |
| `pd` | Pandas: tabeller och DataFrames | Sammanställa testresultat |
| `plt` | Matplotlib | Rita powerkurvan |
| `NormalIndPower` | Klass för power med två oberoende andelar, via normalapproximation | Räkna antal deltagare eller power |
| `TTestIndPower` | Klass för power för två oberoende medelvärden | Planera t-test |
| `proportion_effectsize()` | Omvandlar två andelar till **Cohen's h** | Effektstorlek för `NormalIndPower` |
| `proportions_ztest()` | Z-test för två andelar | Hypotesprov i simuleringen |
| `multipletests()` | Justerar flera p-värden | Bonferroni och FDR |
| `rng` | Slumptalsgenerator | Reproducerbara exempel |

**Begrepp:** `NormalIndPower` är en **klass**. När vi skriver `solver = NormalIndPower()` skapar vi en **instans**, alltså ett objekt som har metoder som `.solve_power()` och `.power()`.

---

## 5. Beräkna stickprovsstorlek för två andelar

Nu följer vi lärarens första beräkning.

```python
baseline_rate_a = 0.25
min_detectable_diff_pp = 2.0
min_detectable_diff = min_detectable_diff_pp / 100

target_rate_b = baseline_rate_a + min_detectable_diff
alpha = 0.05
power_target = 0.80

# Steg 1: gör om skillnaden till standardiserad effektstorlek.
effect_size = proportion_effectsize(
    prop1=baseline_rate_a,
    prop2=target_rate_b,
)

# Steg 2: skapa ett objekt som kan lösa powerproblem.
solver = NormalIndPower()

# Steg 3: räkna fram nödvändigt antal observationer i grupp A.
n_per_group = solver.solve_power(
    effect_size=effect_size,
    alpha=alpha,
    power=power_target,
    ratio=1.0,
    alternative="two-sided",
)

# Steg 4: avrunda uppåt eftersom vi behöver hela personer.
n_per_group_rounded = int(np.ceil(n_per_group))

print(f"Cohen's h: {effect_size:.4f}")
print(f"Beräknat antal per grupp: {n_per_group:.2f}")
print(f"Avrundat antal per grupp: {n_per_group_rounded}")
print(f"Totalt antal: {2 * n_per_group_rounded}")
```

### Vad gör funktionerna?

**`proportion_effectsize(prop1, prop2)`** tar två andelar, här 0,25 och 0,27, och räknar ut **Cohen's h**. Den är en standardiserad effektstorlek för andelar. Vi behöver inte memorera den matematiska omvandlingen för att förstå arbetsflödet, men vi måste veta att funktionen förväntar sig *andelar*, inte tal som 25 och 27.

**`solver.solve_power(...)`** beräknar den parameter vi inte har angett. Här har vi angett effektstorlek, alfa, önskad power och gruppfördelning, men **inte** `nobs1`. Därför löser metoden ut stickprovsstorleken.

| Argument | Förklaring |
|---|---|
| `effect_size` | Cohen's h från de två planerade andelarna |
| `alpha=0.05` | Signifikansnivå 5 % |
| `power=0.80` | Målet: upptäcka den planerade effekten i 80 % av upprepade experiment |
| `ratio=1.0` | Grupp B ska vara lika stor som grupp A |
| `alternative="two-sided"` | Vi testar skillnad åt båda håll |

**`np.ceil()`** betyder *ceiling*, avrundning **uppåt**. `int()` omvandlar resultatet till ett heltal.

> **Viktigt:** Beräkningen bygger på antaganden om baslinje, effekt, oberoende observationer och testmetod. Den garanterar inte att ett enskilt experiment blir signifikant.

---

## 6. Beräkna power när stickprovsstorleken redan är bestämd

Ibland vet vi hur många observationer vi kan samla in. Då ställer vi motsatt fråga: **Vilken power får vi med det antalet?**

```python
planned_n_per_group = 7000

planned_power = solver.power(
    effect_size=effect_size,
    nobs1=planned_n_per_group,
    alpha=alpha,
    ratio=1.0,
    alternative="two-sided",
)

print(f"Power med {planned_n_per_group} per grupp: {planned_power:.1%}")
```

**Skillnaden mellan metoderna:**

- `.solve_power()` – när vi vill **räkna fram antal deltagare** utifrån en powernivå.
- `.power()` – när vi vill **räkna fram power** utifrån ett känt antal deltagare.

I videon testar läraren också att öka antalet till exempelvis 7 600 per grupp och se hur powern förändras.

---

## 7. Simulera power: upprepa samma experiment många gånger

Den analytiska beräkningen ovan använder en matematisk modell. Vi kan också **simulera** många experiment och kontrollera hur ofta vi får `p < alpha`.

Vi *låtsas* nu att den verkliga andelen är exakt 25 % i A och 27 % i B. Det vet vi i simuleringen eftersom **vi själva bestämmer sanningen**.

```python
def simulate_power(
    n_per_group,
    true_p_a,
    true_p_b,
    alpha=0.05,
    n_simulations=3000,
    seed=42,
):
    """Returnerar andelen simulerade experiment med p < alpha."""
    rng = np.random.default_rng(seed)
    rejections = 0

    for _ in range(n_simulations):
        # Ettor = konvertering, nollor = ingen konvertering.
        a = rng.binomial(1, true_p_a, size=n_per_group)
        b = rng.binomial(1, true_p_b, size=n_per_group)

        # Antal lyckade utfall i respektive grupp.
        successes = [a.sum(), b.sum()]

        # Antal observationer i respektive grupp.
        nobs = [len(a), len(b)]

        # Testet ger tillbaka (z_statistic, p_value).
        _, p_value = proportions_ztest(
            count=successes,
            nobs=nobs,
            alternative="two-sided",
        )

        if p_value < alpha:
            rejections += 1

    return rejections / n_simulations


simulated_power = simulate_power(
    n_per_group=n_per_group_rounded,
    true_p_a=baseline_rate_a,
    true_p_b=target_rate_b,
    alpha=alpha,
    n_simulations=3000,
)

print(f"Simulerad power: {simulated_power:.1%}")
print(f"Målsatt power: {power_target:.1%}")
```

### Kodgenomgång – varför skriver vi så?

**`def simulate_power(...):`** skapar en funktion. Parametrarna gör att vi kan återanvända samma kod med olika antal deltagare, andelar och signifikansnivåer.

**`rng.binomial(1, true_p_a, size=n_per_group)`** skapar en array med nollor och ettor. Första argumentet `1` betyder att varje observation motsvarar **ett binärt försök**. `true_p_a` är sannolikheten för en etta och `size` anger hur många observationer vi vill ha.

**`a.sum()`** räknar antalet ettor. Eftersom varje konvertering är 1 blir summan antalet konverteringar.

**`len(a)`** räknar antalet observationer i grupp A.

**`proportions_ztest(count=..., nobs=...)`** jämför de två andelarna. `count` ska innehålla **antal lyckade utfall**, medan `nobs` ska innehålla **antal observationer**. Blanda inte ihop dem!

**`_, p_value = ...`** är *uppackning* av två returvärden. Understrecket `_` visar att vi inte behöver z-värdet just här. Vi sparar endast p-värdet.

**`rejections += 1`** är en kortform av `rejections = rejections + 1`. Vi räknar hur många gånger vi förkastar H₀.

**`return rejections / n_simulations`** ger andelen signifikanta resultat, alltså vår uppskattning av power.

### Varför kan resultatet skilja sig från 80 %?

Simulering innehåller slump. Med 3 000 upprepningar får vi en uppskattning som kan ligga lite över eller under 80 %. Om vi kör fler simuleringar brukar variationen minska. Vi kan också få små skillnader mellan simulering och analytisk approximation.

**Prestandatips:** 3 000 experiment med tusentals deltagare i varje grupp kan ta tid. Börja med `n_simulations=200` när du testar koden och höj när allt fungerar.

---

## 8. Rita power som funktion av stickprovsstorlek

Läraren bygger en graf för att visa att power ökar när vi får fler observationer.

Vi börjar med en **snabb analytisk kurva**, så att du slipper vänta på tusentals simuleringar. Därefter kan du prova lärarens simulerade variant.

```python
n_grid = np.linspace(
    max(50, n_per_group_rounded // 2),
    n_per_group_rounded * 2,
    num=20,
    dtype=int,
)

power_values = [
    solver.power(
        effect_size=effect_size,
        nobs1=int(n),
        alpha=alpha,
        ratio=1.0,
        alternative="two-sided",
    )
    for n in n_grid
]

fig, ax = plt.subplots(figsize=(9, 5))
ax.plot(n_grid, power_values, marker="o", label="Beräknad power")
ax.axhline(power_target, linestyle="--", label="Mål: 80 %")
ax.axvline(n_per_group_rounded, linestyle=":", label="Planerat antal")

ax.set_title("Power som funktion av stickprovsstorlek")
ax.set_xlabel("Antal observationer per grupp")
ax.set_ylabel("Power")
ax.set_ylim(0, 1)
ax.grid(alpha=0.3)
ax.legend()
plt.tight_layout()
plt.show()
```

### Nya funktioner i grafkoden

- **`np.linspace(start, stop, num=20, dtype=int)`** skapar 20 ungefär jämnt fördelade heltal mellan start och stopp. Dessa blir våra testade stickprovsstorlekar.
- **List comprehension** (`[ ... for n in n_grid ]`) kör `.power()` en gång för varje antal i listan och samlar resultaten.
- **`ax.plot(x, y)`** ritar kurvan.
- **`ax.axhline(...)`** ritar en *horisontell* referenslinje vid 80 %.
- **`ax.axvline(...)`** ritar en *vertikal* linje vid det framräknade antalet per grupp.
- **`plt.tight_layout()`** justerar marginalerna så att text och etiketter får plats.

### Om du vill följa videons simulering

Byt ut listan `power_values` mot följande (detta tar betydligt längre tid):

```python
power_values_simulated = [
    simulate_power(
        n_per_group=int(n),
        true_p_a=baseline_rate_a,
        true_p_b=target_rate_b,
        alpha=alpha,
        n_simulations=300,
        seed=42 + i,
    )
    for i, n in enumerate(n_grid)
]
```

Här används **`enumerate(n_grid)`**, som ger både ett index `i` och ett värde `n`. Vi använder indexet för att ge varje punkt en egen slumpfröinställning.

Simuleringskurvan blir normalt lite ojämn eftersom varje punkt innehåller slumpvariation. Det är förväntat.

---

## 9. Skriv ut en tydlig sammanfattning

Läraren använder en flerradig **f-string** för att kombinera text och beräknade variabler.

```python
summary_text = f"""
PLANERING AV A/B-TEST FÖR ANDELAR
Baslinje A: {baseline_rate_a:.1%}
Minsta praktiskt viktiga skillnad: +{min_detectable_diff_pp:.1f} procentenheter
Målnivå B: {target_rate_b:.1%}
Signifikansnivå: {alpha:.1%}
Målsatt power: {power_target:.0%}
Beräknat antal per grupp: {n_per_group_rounded}
Totalt antal: {2 * n_per_group_rounded}
Simulerad power: {simulated_power:.1%}
"""

print(summary_text)
```

**Varför `f"""..."""`?** Tre citattecken låter oss skriva flera rader. Bokstaven `f` gör att Python ersätter uttryck inom `{...}` med värden.

- `:.1%` visar ett tal som procent med en decimal (`0.25` blir `25.0%`).
- `:.0%` visar procent utan decimaler.
- `:.1f` visar ett vanligt tal med en decimal.

---

## 10. Power för medelvärden: Cohen's d och t-test

Powerberäkning fungerar även när utfallet är **kontinuerligt**, till exempel antal minuter eller kronor.

För två medelvärden använder vi ofta **Cohen's d**:

\[
d = \frac{\mu_B - \mu_A}{\sigma}
\]

Här är `sigma` en antagen gemensam standardavvikelse. I verkliga analyser kan gruppernas variation skilja sig åt; den här förenklade planeringsmodellen bygger på ett antagande om jämförbar spridning.

```python
mean_a = 100
mean_b = 105
standard_deviation = 25

cohens_d = (mean_b - mean_a) / standard_deviation

solver_t = TTestIndPower()

n_needed_t = solver_t.solve_power(
    effect_size=cohens_d,
    alpha=0.05,
    power=0.80,
    ratio=1.0,
    alternative="two-sided",
)

n_needed_t_rounded = int(np.ceil(n_needed_t))

print(f"Cohen's d: {cohens_d:.3f}")
print(f"Antal per grupp: {n_needed_t_rounded}")

planned_n_t = 100
power_t = solver_t.power(
    effect_size=cohens_d,
    nobs1=planned_n_t,
    alpha=0.05,
    ratio=1.0,
    alternative="two-sided",
)
print(f"Power med {planned_n_t} per grupp: {power_t:.1%}")
```

**Vad är skillnaden mot förra exemplet?**

| A/B-test för andelar | A/B-test för medelvärden |
|---|---|
| Binärt utfall: 0 eller 1 | Kontinuerligt utfall, t.ex. kronor |
| Cohen's **h** | Cohen's **d** |
| `proportion_effectsize()` | Skillnad / standardavvikelse |
| `NormalIndPower()` | `TTestIndPower()` |

I videon visar läraren också ett medelvärdesexempel där det behövs omkring **100 personer per grupp**. Antalet beror på vilka medelvärden och vilken standardavvikelse man antar; exemplet ovan använder egna tydligt angivna värden.

---

## 11. Varför blir många hypotesprövningar ett problem?

Anta att vi testar 20 helt orelaterade knappar, målgrupper eller händelser. Alla nollhypoteser är egentligen sanna, men vi använder `alpha = 0.05`.

För **ett** test är sannolikheten för ett falsklarm 5 %.

För **20 oberoende** test blir det förväntade antalet falsklarm:

\[
20 \times 0.05 = 1
\]

Det betyder **i genomsnitt ett**, inte exakt ett i varje omgång.

Sannolikheten för **minst ett** falsklarm blir:

\[
1 - (1 - 0.05)^{20} \approx 0.642
\]

```python
alpha = 0.05
m = 20

expected_false_positives = m * alpha
prob_at_least_one = 1 - (1 - alpha) ** m

print(f"Förväntat antal falsklarm: {expected_false_positives:.1f}")
print(f"Risk för minst ett falsklarm: {prob_at_least_one:.1%}")
```

Resultat: cirka **64,2 %** risk för minst ett falsklarm **om testerna är oberoende och alla nollhypoteser är sanna**. Den exakta formeln kan inte utan vidare användas när testerna är beroende av varandra.

---

## 12. Vad menas med en testfamilj?

En **testfamilj** är de hypotesprövningar som tillsammans används för att stödja ett gemensamt påstående eller beslut.

Exempel:

- Vi jämför fem olika nya knappar mot samma kontrollknapp.
- Vi analyserar samma konverteringsmål i tio målgrupper.
- Vi testar flera utfall och använder något av dem som argument för lansering.

Det är klokt att bestämma vilka test som ingår i familjen **innan vi tittar på resultaten**. Annars riskerar vi att ändra analysplanen efter vad som råkade bli signifikant.

---

## 13. Bonferroni: skydda mot minst ett falsklarm

**FWER** står för *Family-Wise Error Rate*: sannolikheten för **minst ett typ I-fel i hela testfamiljen**.

Bonferroni delar den tillåtna totala felrisken mellan testerna:

\[
\alpha_{\text{per test}} = \frac{\alpha_{\text{familj}}}{m}
\]

Om `alpha = 0.05` och `m = 20`:

```python
alpha_family = 0.05
m = 20
alpha_bonferroni = alpha_family / m

print(alpha_bonferroni)  # 0.0025
```

Nu krävs `p < 0.0025` för att ett enskilt test ska räknas som signifikant enligt denna gräns.

**Fördel:** stark kontroll av familjevis felrisk, även utan antagande om oberoende mellan testerna.

**Nackdel:** strikt metod; fler verkliga effekter kan missas när många test ingår.

**Två likvärdiga sätt att använda Bonferroni:**

1. Jämför **rått p-värde** med `0.05 / m`.
2. Multiplicera p-värdet med `m` (högst 1) och jämför **justerat p-värde** med `0.05`.

Använd **inte båda korrigeringarna samtidigt**.

---

## 14. FDR: kontrollera andelen falska fynd

**FDR** betyder *False Discovery Rate*. Den beskriver det **förväntade förhållandet mellan antalet falska upptäckter och antalet upptäckter**, med andelen definierad som 0 om vi inte gör några upptäckter.

**Benjamini–Hochberg (BH)** är en vanlig metod för att kontrollera FDR. Den sorterar p-värden och använder trösklar som beror på rangordning och antal test.

Det är en annan risk än FWER:

| Mått | Vad vill vi kontrollera? |
|---|---|
| **FWER** | Risken att **minst ett** fynd är falskt |
| **FDR** | Den **förväntade andelen** falska fynd bland fynden |

BH är ofta mindre strikt än Bonferroni, så fler verkliga signaler kan upptäckas. Den vanliga BH-garantin bygger på oberoende eller vissa former av positivt beroende mellan testerna; vid godtyckligt beroende behövs andra överväganden eller metoder.

**Viktig tolkning:** FDR på 5 % innebär inte att exakt 5 % av fynden i **just din tabell** är falska, eller att varje enskilt fynd har 5 % sannolikhet att vara falskt.

---

## 15. Lärarens exempel: skapa 20 p-värden

I videon skapas **15 p-värden som motsvarar sanna nollhypoteser** och **5 som representerar verkliga effekter**. Vi skapar ett liknande **illustrativt** dataset. Det är simulerade p-värden, inte resultat från riktiga A/B-test.

```python
rng = np.random.default_rng(7)

# Under sanna nollhypoteser är giltiga kontinuerliga p-värden
# typiskt ungefär jämnt fördelade mellan 0 och 1.
p_values_null = rng.uniform(low=0, high=1, size=15)

# Illustrativ fördelning som ger fler små p-värden.
# Detta är INTE ett generellt recept för verkliga effekter.
p_values_effect = rng.beta(a=0.5, b=20, size=5) * 0.1

p_values = np.concatenate([p_values_null, p_values_effect])
rng.shuffle(p_values)

print("Antal p-värden:", len(p_values))
print("De fem minsta:", np.sort(p_values)[:5])
```

### Förklara funktionerna

- **`rng.uniform(low=0, high=1, size=15)`** skapar 15 slumpvärden mellan 0 och 1.
- **`rng.beta(a=0.5, b=20, size=5)`** drar 5 värden ur en beta-fördelning. Med dessa parametrar hamnar många nära noll.
- **`* 0.1`** gör de illustrativa effekt-p-värdena ännu mindre. Det är ett pedagogiskt demonstrationsgrepp i videon, **inte** en realistisk modell för alla A/B-test.
- **`np.concatenate([...])`** slår ihop två arrayer till en.
- **`rng.shuffle(p_values)`** blandar ordningen **på plats**. Funktionen returnerar inte den blandade arrayen; den ändrar variabelns innehåll.

**Obs:** Det finns ingen garanti att just detta slumpförsök ger samma antal signifikanta resultat som läraren fick i sin körning.

---

## 16. Korrigera p-värden med `multipletests()`

Vi testar **samma** p-värden med båda metoderna, så jämförelsen blir rättvis.

```python
alpha_global = 0.05

reject_fdr, p_adjusted_fdr, _, _ = multipletests(
    p_values,
    alpha=alpha_global,
    method="fdr_bh",
)

reject_bonf, p_adjusted_bonf, _, _ = multipletests(
    p_values,
    alpha=alpha_global,
    method="bonferroni",
)

results = pd.DataFrame({
    "p_raw": p_values,
    "fdr_reject": reject_fdr,
    "p_adjusted_fdr": p_adjusted_fdr,
    "bonf_reject": reject_bonf,
    "p_adjusted_bonf": p_adjusted_bonf,
})

results = results.sort_values("p_raw")

display(results.head(10))
print("Antal signifikanta med FDR:", reject_fdr.sum())
print("Antal signifikanta med Bonferroni:", reject_bonf.sum())
```

### Hur fungerar `multipletests()`?

- **Första argumentet** `p_values`: en samling råa p-värden.
- **`alpha=0.05`**: den önskade nivån för respektive felriskmått.
- **`method="fdr_bh"`**: Benjamini–Hochberg, kontroll av FDR.
- **`method="bonferroni"`**: Bonferroni, kontroll av FWER.

Funktionen returnerar **fyra värden**:

1. `reject`: boolesk array med `True` där H₀ förkastas.
2. `pvals_corrected`: justerade p-värden.
3. `alphacSidak`: en beräknad Sidak-gräns.
4. `alphacBonf`: en beräknad Bonferroni-gräns.

Vi använder bara de två första. Därför skriver vi `_, _` för de två andra.

**`pd.DataFrame({...})`** skapar en tabell med en kolumn per nyckel i ordboken.

**`sort_values("p_raw")`** sorterar raderna så att de minsta råa p-värdena kommer först.

**`display(results.head(10))`** visar de tio första raderna i Jupyter.

**`reject_fdr.sum()`** räknar antal `True`, eftersom Python behandlar `True` som 1 och `False` som 0 i summeringar.

I lärarens exempel gav FDR **5 signifikanta fynd** och Bonferroni **3**. Exakta utfall i din körning kan skilja sig eftersom vi använder en egen slumptalsgenerator och inte nödvändigtvis identisk kod eller slumptalssekvens.

---

## 17. Så fungerar Benjamini–Hochberg i princip

Anta att vi har fem sorterade p-värden:

```python
p_sorted = np.array([0.001, 0.009, 0.018, 0.040, 0.200])
alpha = 0.05
m = len(p_sorted)

bh_thresholds = np.arange(1, m + 1) / m * alpha

bh_table = pd.DataFrame({
    "rank": np.arange(1, m + 1),
    "p_value": p_sorted,
    "bh_threshold": bh_thresholds,
    "below_threshold": p_sorted <= bh_thresholds,
})

display(bh_table)
```

BH jämför sorterade p-värden med gränsen

\[
\frac{i}{m}\alpha
\]

för rang `i` av `m` test. Metoden hittar den **största rang** där `p_(i) ≤ (i/m) × alpha` och förkastar alla hypoteser upp till den rangen. Det betyder att du **inte** ska tolka varje rad helt separat utan ta hänsyn till BH:s stegvisa beslutsregel.

I praktiken använder vi `multipletests()` för att undvika misstag.

---

## 18. Bonferroni eller FDR – hur väljer vi?

| Fråga | Bonferroni | FDR (BH) |
|---|---|---|
| Vilken risk kontrolleras? | Minst ett falsklarm i familjen (FWER) | Förväntad andel falska fynd (FDR) |
| Hur strikt? | Ofta striktare | Ofta mindre strikt |
| Typiskt användningsområde | Bekräftande analys, höga konsekvenser av falsklarm | Utforskande analys, många kandidater |
| Nackdel | Lägre power när många test görs | Kan acceptera vissa falska fynd bland upptäckterna |

**Exempel:** Om en enda felaktig slutsats skulle leda till ett mycket kostsamt beslut kan FWER-kontroll vara lämplig. Om vi vill hitta lovande signaler bland många möjliga förbättringar som sedan ska testas igen kan FDR vara mer ändamålsenlig.

Valet bör göras **före** resultatgranskningen, inte efter att vi sett vilken metod som ger flest fynd.

---

## 19. Hur hänger power och flerfaldig testning ihop?

Det här är lektionens viktigaste koppling:

1. Vi vill ha tillräckligt hög **power** för att upptäcka en verklig effekt.
2. När vi gör många test ökar risken för falsklarm.
3. Vi korrigerar därför p-värden eller beslutströsklar.
4. En striktare tröskel gör det svårare att hitta effekter.
5. Om vi vill behålla samma power kan vi behöva **större stickprov**.

**Exempel:** Ett A/B-test som planerats för ett enda primärt test med `alpha=0.05` kan vara underdimensionerat om vi senare kräver Bonferroni-korrigering för 20 primära test. Korrigeringen måste alltså vägas in redan i planeringen.

---

## 20. Gör en analysplan innan experimentet börjar

En bra plan anger:

1. **Primärt utfall:** Vad avgör om experimentet lyckas?
2. **MDE:** Vilken skillnad är praktiskt viktig?
3. **Baslinje:** Vad är förväntad andel eller medelnivå i kontrollgruppen?
4. **Power:** Ofta 80 % eller 90 %, beroende på behov.
5. **Alfa:** Exempelvis 0,05.
6. **Testmetod:** Z-test, t-test eller något annat.
7. **Gruppfördelning:** Till exempel 50/50.
8. **Stickprovsstorlek och stopptid:** Bestäm i förväg.
9. **Testfamiljer:** Vilka hypoteser hör ihop?
10. **Korrigering:** Bonferroni, FDR eller annan lämplig metod.

**Varför bestämma stopptiden?** Om vi tittar på p-värdet varje dag och stoppar så fort det blir signifikant kan den verkliga risken för falsklarm bli större än den planerade. Vanliga fasta testgränser är inte gjorda för obegränsad upprepad kontroll.

---

## 21. Vanliga misstag och hur du undviker dem

| Misstag | Varför det blir fel | Gör så här i stället |
|---|---|---|
| Tolka 80 % power som 80 % sannolikhet att H₁ är sann | Power är en långsiktig upptäcktsfrekvens givet en verklig effekt | Beskriv power som andel upptäckter vid upprepade experiment |
| Skriva `25` i stället för `0.25` | Funktioner för andelar förväntar sig tal mellan 0 och 1 | Använd `0.25` för 25 % |
| Blanda procent och procentenheter | 25 → 27 % är +2 procentenheter, +8 % relativt | Redovisa måtten separat |
| Avrunda stickprovsstorleken nedåt | Kan ge lägre power än målet | Använd `int(np.ceil(...))` |
| Använda samma p-värdesgräns för 20 test utan plan | Ökar risken för falsklarm | Definiera testfamilj och korrigering |
| Både justera p-värden och sänka alfa igen | Dubbelkorrigering | Jämför justerade p-värden med ursprunglig alfa |
| Säga att FDR 5 % betyder exakt 5 % falska fynd i tabellen | FDR är ett förväntat felmått | Beskriv förväntad andel över upprepade analyser |
| Tolka simulerade p-värden som verkliga experiment | Demonstrationen har påhittade fördelningar | Ange tydligt att data är simulerade |
| Göra ett t-test med powerberäkning för andelar | Fel modell för utfallstypen | Välj `TTestIndPower` för medelvärden |
| Välja korrigeringsmetod efter att ha sett resultaten | Risk för selektiv rapportering | Bestäm metod i förväg |

---

## 22. Övningar – prova själv

Försök gärna skriva koden innan du tittar på facit. Alla övningar kan göras i en notebook.

### Övning 1 – Begrepp

Förklara med egna ord vad typ I-fel, typ II-fel och power betyder.

### Övning 2 – Beräkna power

Om `beta = 0.15`, vad blir power? Skriv ett Python-uttryck.

### Övning 3 – Procentenheter

Kontrollgrupp A har 18 % konvertering. Du vill upptäcka en ökning på 3 procentenheter. Vad är målkonverteringen för B? Vad är den relativa ökningen?

### Övning 4 – MDE och antal

Använd `proportion_effectsize()` och `NormalIndPower().solve_power()` för att planera ett tvåsidigt test med A = 0,18, B = 0,21, alfa = 0,05 och power = 0,80. Avrunda antalet uppåt.

### Övning 5 – Ändra alfa

Beräkna antal per grupp för samma effekt och power som i övning 4, men med `alpha = 0.01`. Blir antalet större eller mindre? Varför?

### Övning 6 – Beräkna faktisk planerad power

Använd `.power()` för att beräkna power med 1 000 observationer per grupp för A = 0,18 och B = 0,21.

### Övning 7 – Binär simulering

Skapa 20 binära observationer med `rng.binomial(1, 0.30, size=20)`. Beräkna andelen ettor med både `sum() / len()` och `mean()`.

### Övning 8 – Simulera power

Använd funktionen `simulate_power()` med 500 observationer per grupp, A = 0,25 och B = 0,27. Jämför med 5 000 observationer per grupp. Vad händer?

### Övning 9 – Powerkurva

Rita en graf över analytisk power för 500, 1 000, 2 000, 4 000 och 8 000 observationer per grupp, med A = 0,25 och B = 0,27.

### Övning 10 – Cohen's d

Grupp A har förväntat medelvärde 50 och grupp B 55. Gemensam standardavvikelse antas vara 20. Beräkna Cohen's d och nödvändigt antal per grupp för 80 % power.

### Övning 11 – Många test

Om 30 oberoende nollhypoteser är sanna och varje test använder alfa = 0,05: vad är förväntat antal falsklarm och sannolikheten för minst ett?

### Övning 12 – Bonferroni

Du gör 10 test och vill ha familjevis alfa = 0,05. Vilken gräns ska råa p-värden jämföras med?

### Övning 13 – `multipletests()`

Korrigera p-värdena `[0.001, 0.012, 0.021, 0.040, 0.200]` med både Bonferroni och BH/FDR. Hur många hypoteser förkastas av respektive metod vid alfa = 0,05?

### Övning 14 – Resonera

Du har 50 utforskande hypoteser och vill hitta intressanta kandidater för framtida bekräftande tester. Vilken metod ligger närmast det syftet: Bonferroni eller FDR? Motivera.

### Övning 15 – Hitta buggen

Vad är problemet med följande kod?

```python
p_values = [0.001, 0.01, 0.04]
reject, adjusted, _, _ = multipletests(p_values, method="bonferroni")
final_decision = adjusted < (0.05 / len(p_values))
```

---

## 23. Facit och förklaringar

### Facit 1

**Typ I-fel:** vi förkastar en sann nollhypotes. **Typ II-fel:** vi förkastar inte en falsk nollhypotes. **Power:** sannolikheten att förkasta H₀ när den planerade effekten faktiskt finns.

### Facit 2

```python
beta = 0.15
power = 1 - beta
print(f"{power:.0%}")  # 85%
```

### Facit 3

```python
p_a = 0.18
p_b = p_a + 0.03
relative_change = (p_b / p_a - 1) * 100
print(f"B: {p_b:.0%}, relativ ökning: {relative_change:.1f}%")
```

**Svar:** B = 21 %, absolut +3 procentenheter, relativt cirka +16,7 %.

### Facit 4

```python
h = proportion_effectsize(0.18, 0.21)
n = NormalIndPower().solve_power(
    effect_size=h, alpha=0.05, power=0.80,
    ratio=1.0, alternative="two-sided"
)
print(int(np.ceil(n)))
```

Resultatet är antal **per grupp**. Kör koden för det exakta värdet i din installerade version.

### Facit 5

```python
n_stricter = NormalIndPower().solve_power(
    effect_size=h, alpha=0.01, power=0.80,
    ratio=1.0, alternative="two-sided"
)
print(int(np.ceil(n_stricter)))
```

**Svar:** Det behövs fler observationer. Lägre alfa innebär en striktare gräns för signifikans.

### Facit 6

```python
power_1000 = NormalIndPower().power(
    effect_size=h, nobs1=1000, alpha=0.05,
    ratio=1.0, alternative="two-sided"
)
print(f"{power_1000:.1%}")
```

### Facit 7

```python
rng = np.random.default_rng(42)
data = rng.binomial(1, 0.30, size=20)
print(data)
print(data.sum() / len(data))
print(data.mean())
```

De två sista raderna ger **samma andel**. Andelen behöver inte bli exakt 30 % när vi bara simulerar 20 observationer.

### Facit 8

```python
for n in [500, 5000]:
    result = simulate_power(n, 0.25, 0.27, n_simulations=300)
    print(n, f"{result:.1%}")
```

**Svar:** Power blir normalt högre med 5 000 per grupp. Slumpvariation gör att exakta resultat kan skilja sig.

### Facit 9

```python
h = proportion_effectsize(0.25, 0.27)
ns = [500, 1000, 2000, 4000, 8000]
powers = [NormalIndPower().power(effect_size=h, nobs1=n, alpha=0.05)
          for n in ns]

plt.plot(ns, powers, marker="o")
plt.xlabel("Antal per grupp")
plt.ylabel("Power")
plt.title("Power ökar med stickprovsstorlek")
plt.grid(alpha=0.3)
plt.show()
```

### Facit 10

```python
d = (55 - 50) / 20
n = TTestIndPower().solve_power(effect_size=d, alpha=0.05, power=0.80)
print("Cohen's d:", d)
print("Antal per grupp:", int(np.ceil(n)))
```

**Svar:** Cohen's d = 0,25. Stickprovsstorleken beräknas med t-testets powermodell.

### Facit 11

```python
m = 30
alpha = 0.05
print("Förväntade falsklarm:", m * alpha)
print("Risk för minst ett:", 1 - (1 - alpha) ** m)
```

**Svar:** 1,5 förväntade falsklarm och cirka 78,5 % risk för minst ett, givet oberoende tester och sanna nollhypoteser.

### Facit 12

```python
print(0.05 / 10)  # 0.005
```

**Svar:** `0.005`.

### Facit 13

```python
p = [0.001, 0.012, 0.021, 0.040, 0.200]

for method in ["bonferroni", "fdr_bh"]:
    reject, p_adjusted, _, _ = multipletests(p, alpha=0.05, method=method)
    print(method, "antal:", reject.sum())
    print("Justerade p-värden:", p_adjusted)
```

**Svar:** Bonferroni förkastar **1** hypotes. BH/FDR förkastar **3** hypoteser (de tre första p-värdena).

### Facit 14

**FDR (Benjamini–Hochberg)** passar ofta bättre när målet är att hitta många lovande kandidater i en utforskande analys, och dessa sedan ska bekräftas. Valet förutsätter att metodens antaganden är rimliga.

### Facit 15

Koden **dubbelkorrigerar**: `adjusted` innehåller redan Bonferroni-justerade p-värden, men de jämförs med den sänkta Bonferroni-gränsen igen.

Rätt:

```python
p_values = [0.001, 0.01, 0.04]
reject, adjusted, _, _ = multipletests(p_values, alpha=0.05, method="bonferroni")
final_decision = adjusted < 0.05
print(final_decision)
```

Du kan också använda `reject` direkt, eftersom `multipletests()` redan har gjort beslutet.

---

## 24. Snabbreferens

| Begrepp | Kort förklaring |
|---|---|
| **H₀** | Nollhypotes: ingen effekt enligt testet |
| **Typ I-fel** | Falskt positivt – förkastar sann H₀ |
| **Typ II-fel** | Falskt negativt – missar en verklig effekt |
| **α (alfa)** | Planerad signifikansnivå |
| **β (beta)** | Risk för typ II-fel vid given effekt |
| **Power** | `1 - beta` |
| **MDE** | Minsta praktiskt viktig effekt som experimentet planeras för |
| **Cohen's h** | Standardiserad effektstorlek för två andelar |
| **Cohen's d** | Standardiserad skillnad mellan medelvärden |
| **FWER** | Risk för minst ett typ I-fel i testfamiljen |
| **Bonferroni** | Kontrollerar FWER med `alpha / antal_test` |
| **FDR** | Förväntad andel falska upptäckter bland upptäckterna |
| **BH** | Benjamini–Hochberg: vanlig metod för att kontrollera FDR |
| `NormalIndPower().solve_power()` | Beräknar nödvändigt antal för powerplanering av andelar |
| `NormalIndPower().power()` | Beräknar power för ett bestämt antal |
| `TTestIndPower()` | Powerplanering för två oberoende medelvärden |
| `proportion_effectsize()` | Räknar Cohen's h |
| `proportions_ztest()` | Z-test för två andelar |
| `multipletests(..., method="bonferroni")` | Bonferronikorrigering |
| `multipletests(..., method="fdr_bh")` | BH/FDR-korrigering |

### Det viktigaste att ta med dig

**Power planeras före experimentet.** Vi behöver ange vilken skillnad som är värd att upptäcka och hur stor sannolikhet vi vill ha att upptäcka den.

**Flerfaldig testning måste också planeras.** Ju fler hypoteser som används för samma beslut, desto större anledning att tänka på korrigering för falska fynd.

**De två frågorna hänger ihop:** Striktare skydd mot falsklarm kan kräva fler observationer för att behålla önskad power.

---

*Materialet följer kursens teori och arbetsflödet i videon. Förtydliganden om statistiska antaganden, simuleringarnas begränsningar och kodens syntax har lagts till för att göra exemplen lättare att förstå och använda.*
