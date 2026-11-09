# 12 – Statistisk inferens

**Ämne:** Python, statistik och dataanalys  
**Video:** [Statistisk inferens – lärarens genomgång](https://www.youtube.com/watch?v=DroB4qukSo4)  
**Föregående lektion:** `11_hypothesis_testing.md`  
**Placering:** `python_and_statistics/data_analysis/12_statistical_inference.md`

> **Målet:** Knyta ihop sampling, konfidensintervall och hypotesprövning till ett arbetssätt som går att använda inför A/B-testning. Den här videon är främst en **begreppsgenomgång**, så Python-exemplen nedan är kompletterande övningar – inte kod som läraren skriver i videon.

## Vad lär vi oss?

- Skilja **population, parameter, stickprov och skattning**.
- Förstå **standardavvikelse (SD)** jämfört med **standardfel (SE)**.
- Rapportera **effekt, konfidensintervall och p-värde** tillsammans.
- Välja jämförelse för **medelvärden** eller **andelar**.
- Skilja **procent** från **procentenheter**.
- Förstå **ensidiga och tvåsidiga test**, **praktisk betydelse**, **power** och **flerfaldig testning**.
- Se vilka beslut som bör fattas **innan** ett A/B-test startar.

---

## 1. Vad är statistisk inferens?

**Statistisk inferens** betyder att vi använder data från ett **stickprov** för att dra slutsatser om en **population** – och beskriver hur osäkra slutsatserna är.

Läraren jämför det med att smaka en sked ur en gryta: vi smakar inte hela grytan, men försöker ändå säga något om helheten. Om innehållet inte är ordentligt blandat kan skeden bli missvisande. På samma sätt spelar **hur stickprovet valdes** stor roll.

**Arbetsflödet:**

`Population → Stickprov → Skattning → Osäkerhet → Beslut`

| Begrepp | Vad betyder det? | Exempel |
|---|---|---|
| Population | Alla vi vill uttala oss om | Alla besökare på en webbplats |
| Parameter | Det verkliga värdet i populationen, oftast okänt | Den sanna andelen som köper |
| Stickprov | De vi faktiskt har observerat | 500 besökare |
| Skattning | Vårt beräknade värde från stickprovet | 110 av 500 köpte = 22 % |

**Varför behövs inferens?** Ett nytt stickprov hade förmodligen gett en annan skattning. Vi behöver därför skilja mellan vad vi **observerat** och vad vi **kan dra slutsatser om**.

## 2. Samplingvariation – varför stickproven ger olika svar

I videon återkommer läraren till en tidigare simulering med stickprov om **30** respektive **200** personer. Båda fördelningarna ligger runt populationens sanna medelvärde, men medelvärdena från stickprov om 30 varierar mer.

Detta kallas **samplingvariation**. Fler observationer ger ofta en stabilare skattning, **förutsatt att urvalet är rimligt gjort**.

### Python: se variationen mellan stickprov

```python
import numpy as np
import matplotlib.pyplot as plt

rng = np.random.default_rng(42)  # Samma slumpserie vid varje körning
population = rng.normal(loc=7000, scale=1800, size=100_000)

means_30 = []
means_200 = []

for _ in range(1000):
    sample_30 = rng.choice(population, size=30, replace=True)
    sample_200 = rng.choice(population, size=200, replace=True)
    means_30.append(sample_30.mean())
    means_200.append(sample_200.mean())

fig, ax = plt.subplots()
ax.hist(means_30, bins=30, alpha=0.5, label="n = 30")
ax.hist(means_200, bins=30, alpha=0.5, label="n = 200")
ax.axvline(population.mean(), color="black", linestyle="--", label="Populationsmedel")
ax.set(title="Stickprovsmedelvärden", xlabel="Medelvärde", ylabel="Antal")
ax.legend()
plt.show()
```

**Vad händer och varför?**

1. `rng.normal(...)` skapar en simulerad population med medelvärde runt 7000.
2. `rng.choice(..., replace=True)` drar slumpmässiga observationer. `replace=True` innebär att samma värde får dras igen.
3. `sample.mean()` beräknar medelvärdet i varje nytt stickprov.
4. Histogrammen visar **fördelningen av medelvärden**, inte fördelningen av enskilda personer.
5. Gruppen med `n = 200` brukar få ett **smalare histogram**, eftersom dess medelvärden varierar mindre.

## 3. Standardavvikelse och standardfel är inte samma sak

| Mått | Beskriver | Typisk formel |
|---|---|---|
| **SD**, standardavvikelse | Hur mycket **enskilda observationer** skiljer sig åt | `np.std(sample, ddof=1)` |
| **SE**, standardfel | Hur mycket **skattningen** väntas variera mellan stickprov | `SD / √n` för ett medelvärde |

För ett stickprovsmedelvärde med oberoende observationer uppskattas standardfelet som:

\[
SE = \frac{s}{\sqrt{n}}
\]

Där `s` är stickprovets standardavvikelse och `n` är antal observationer.

**Exempel:** Om `s = 20` och `n = 100`, blir `SE = 20 / 10 = 2`. Personernas värden har fortfarande en standardavvikelse på 20, men **medelvärdets** osäkerhet uppskattas till 2.

```python
sample = np.array([45, 50, 52, 48, 55, 50, 47, 53])

sd = np.std(sample, ddof=1)
se = sd / np.sqrt(len(sample))

print(f"Standardavvikelse: {sd:.2f}")
print(f"Standardfel: {se:.2f}")
```

- `ddof=1` använder stickprovsformeln för standardavvikelse.
- `len(sample)` räknar observationerna.
- `np.sqrt(...)` tar kvadratroten.

## 4. Konfidensintervall – hur osäker är skattningen?

Ett **95-procentigt konfidensintervall** (KI) ger ett intervall för den okända populationsparametern utifrån en vald metod.

**Korrekt tolkning:** Om vi upprepade samma urval och beräkningsmetod många gånger skulle ungefär 95 % av intervallen innehålla den verkliga parametern, om metodens antaganden håller.

**Inte korrekt:** ”Det är 95 % sannolikhet att den fasta parametern ligger i just detta redan beräknade intervall.”

Ett ungefärligt intervall för ett medelvärde är:

\[
\bar{x} \pm 1{,}96 \times SE
\]

`1,96` är normalapproximationens kritiska värde. När populationsstandardavvikelsen är okänd används ofta ett **t-intervall**, särskilt vid små stickprov.

```python
from scipy import stats

sample = np.array([45, 50, 52, 48, 55, 50, 47, 53])
mean = sample.mean()
se = stats.sem(sample)  # Standardfel för medelvärdet
lower, upper = stats.t.interval(
    confidence=0.95,
    df=len(sample) - 1,
    loc=mean,
    scale=se,
)

print(f"Medelvärde: {mean:.2f}")
print(f"95 % KI: [{lower:.2f}, {upper:.2f}]")
```

**Vad gör funktionerna?** `stats.sem()` räknar standardfelet. `stats.t.interval()` använder t-fördelningen och dess frihetsgrader (`df = n - 1`) för att beräkna intervallets gränser.

**Kom ihåg:** Ett smalt intervall visar **precision**, men skyddar inte mot ett snedvridet urval.

## 5. Hypotesprövning – vad berättar p-värdet?

När vi jämför två grupper kan vi formulera:

- **H₀ (nollhypotes):** Ingen skillnad i populationen.
- **H₁ (alternativhypotes):** Det finns en skillnad.

Ett **p-värde** anger, **om H₀ och testets antaganden gäller**, sannolikheten för ett testresultat som är minst lika extremt som det observerade.

Om vi i förväg valt `α = 0.05`:

| Resultat | Beslut |
|---|---|
| `p < 0.05` | Förkasta H₀; resultatet är statistiskt signifikant vid denna nivå. |
| `p >= 0.05` | Kan inte förkasta H₀; data ger inte tillräcklig evidens mot H₀. |

**Ett stort p-värde bevisar inte H₀.** Ett litet stickprov kan göra en verklig effekt svår att upptäcka.

**Ett p-värde är inte** sannolikheten att H₀ är sann och inte heller ett mått på hur viktig effekten är.

## 6. Ensidigt eller tvåsidigt test?

**Tvåsidigt:** Vi frågar om grupperna skiljer sig **åt något håll**.

\[
H_1: \mu_B - \mu_A \ne 0
\]

**Ensidigt:** Vi frågar om skillnaden går i **en förutbestämd riktning**, exempelvis:

\[
H_1: \mu_B - \mu_A > 0
\]

Lärarens viktiga poäng: **Välj riktning innan du tittar på resultatet.** Att byta till ensidigt test i efterhand för att få ett mindre p-värde ger en missvisande analys. Tvåsidigt test är ett bra standardval.

## 7. Medelvärden eller andelar – välj efter datatyp

| Utfall | Exempel | Vad jämförs? | Vanlig metod |
|---|---|---|---|
| Kontinuerligt/numeriskt | Köpesumma, blodtryck, leveranstid | **Medelvärden** | Welch-test för två oberoende grupper |
| Binärt (ja/nej) | Köp/inte köp, klick/inte klick | **Andelar** | Tvåproportions-z-test, när villkoren är rimliga |

### Exempel A: skillnad i medelvärden

```python
from scipy import stats
import numpy as np

rng = np.random.default_rng(12)
group_a = rng.normal(loc=50, scale=10, size=80)
group_b = rng.normal(loc=53, scale=14, size=80)

difference = group_b.mean() - group_a.mean()
result = stats.ttest_ind(group_b, group_a, equal_var=False)

print(f"Observerad skillnad B - A: {difference:.2f}")
print(f"Welch-test: t = {result.statistic:.3f}, p = {result.pvalue:.4f}")
```

- `loc` är normalfördelningens medelvärde, `scale` dess standardavvikelse och `size` antalet observationer.
- `equal_var=False` väljer **Welch-testet**, som inte kräver lika varians i grupperna.
- Skillnaden definieras som **B minus A**; positivt värde betyder att B:s medelvärde är högre i stickprovet.

### Exempel B: skillnad i andelar

Anta att **20 av 100** personer köper i A och **25 av 100** köper i B.

```python
buyers_a, visitors_a = 20, 100
buyers_b, visitors_b = 25, 100

rate_a = buyers_a / visitors_a
rate_b = buyers_b / visitors_b

difference_pp = (rate_b - rate_a) * 100
relative_change = (rate_b - rate_a) / rate_a * 100

print(f"A: {rate_a:.1%}")
print(f"B: {rate_b:.1%}")
print(f"Skillnad: {difference_pp:.1f} procentenheter")
print(f"Relativ förändring: {relative_change:.1f} %")
```

Detta beräknar **effekten**, inte ett p-värde. För ett hypotesprov på andelar krävs ytterligare beräkning.

## 8. Procent och procentenheter – en vanlig fälla

Om konverteringen ökar från **20 % till 25 %**:

- **Absolut skillnad:** `25 - 20 = 5 procentenheter` (ofta förkortat **pp**).
- **Relativ ökning:** `(25 - 20) / 20 = 0,25 = 25 %`.

Det är **inte** samma sak att säga ”ökade med 5 procent” och ”ökade med 5 procentenheter”. Ange alltid vilken jämförelse du menar.

## 9. Tre delar att redovisa tillsammans

En statistisk slutsats behöver vanligtvis svara på:

1. **Hur stor är effekten?** Exempel: B − A = **+5 procentenheter**.
2. **Hur osäker är skattningen?** Exempel: ett **95 % KI** för skillnaden.
3. **Hur förenligt är resultatet med H₀?** Exempel: ett **p-värde** från ett lämpligt test.

**Rapportmall:**

> Grupp B hade en skattad konverteringsgrad som var **X procentenheter högre** än grupp A (95 % KI: **L till H procentenheter**; **p = P**, metod: **M**). Resultatet bedöms i relation till vår i förväg bestämda gräns för praktisk nytta.

Fyll i verkliga värden från analysen; hitta inte på konfidensintervall eller p-värden.

## 10. Statistisk signifikans kontra praktisk betydelse

**Statistiskt signifikant** betyder att resultatet är ovanligt under H₀ vid vald signifikansnivå. Det betyder **inte automatiskt** att förändringen är värd pengar, tid eller risk.

Föreställ dig att företaget i förväg bestämmer att minst **+5 procentenheter** behövs för att en förändring ska vara värd att införa. Det är den **praktiska tröskeln**.

Läraren visar tre scenarier:

| Scenario | Konfidensintervallets läge | Tolkning |
|---|---|---|
| **A** | Helt över 0 men helt under +5 pp | Statistiskt stöd för positiv skillnad, men den verkar för liten för den praktiska tröskeln. |
| **B** | Helt över +5 pp | Statistiskt stöd och en effekt som även överstiger den praktiska tröskeln. |
| **C** | Innehåller 0 | Ingen statistiskt säkerställd skillnad vid motsvarande testnivå; osäkerhet kvarstår. |

**Viktigt:** Kopplingen mellan ett 95 % KI och ett tvåsidigt test vid `α = 0.05` förutsätter att **intervall och test bygger på förenliga metoder och antaganden**.

### Python: visualisera de tre scenarierna

```python
import matplotlib.pyplot as plt

# Illustrativa intervall – inte resultat från verklig data
labels = ["A: liten men tydlig", "B: praktiskt viktig", "C: osäker"]
means = [2.0, 7.0, 2.0]
lower = [1.0, 6.0, -2.0]
upper = [3.0, 8.0, 6.0]

fig, ax = plt.subplots(figsize=(9, 4))
for i, (mean, low, high) in enumerate(zip(means, lower, upper)):
    ax.errorbar(mean, i, xerr=[[mean - low], [high - mean]], fmt="o", capsize=4)

ax.axvline(0, color="black", linestyle="--", label="Ingen skillnad")
ax.axvline(5, color="orange", linestyle=":", label="Praktisk tröskel: +5 pp")
ax.set(yticks=range(3), yticklabels=labels,
       xlabel="Skillnad B - A (procentenheter)",
       title="Statistiskt stöd och praktisk betydelse")
ax.invert_yaxis()
ax.legend()
plt.tight_layout()
plt.show()
```

- `errorbar()` ritar en punkt med ett intervall runt.
- `axvline(0)` markerar **ingen skillnad**.
- `axvline(5)` markerar företagets **praktiska tröskel**.
- Värdena är **påhittade för att illustrera videons figur**, inte beräknade från ett dataset.

## 11. Power och två typer av fel

| Begrepp | Betydelse |
|---|---|
| **Typ I-fel** (falskt positivt) | Vi förkastar H₀ trots att H₀ är sann. |
| **Typ II-fel** (falskt negativt) | Vi förkastar inte H₀ trots att en effekt av den aktuella storleken finns. |
| **Power** (`1 - β`) | Sannolikheten att upptäcka en viss verklig effekt med det planerade testet. |

Power påverkas bland annat av:

- **Stickprovsstorlek:** större stickprov ger ofta högre power.
- **Effektstorlek:** större effekter är lättare att upptäcka.
- **Variation:** mindre brus gör skillnaden tydligare.
- **Signifikansnivå:** högre `α` kan öka power men ökar också risken för typ I-fel.

**Viktig slutsats:** ”Inte signifikant” betyder inte automatiskt ”ingen effekt”. Studien kan ha varit för liten eller för brusig.

### Tänk igenom power före datainsamlingen

Om du vill upptäcka en ökning i konvertering från 20 % till 21 % behöver du vanligtvis mycket mer data än om du vill upptäcka en ökning från 20 % till 30 %. **Ju mindre effekt du vill kunna upptäcka, desto större stickprov behövs normalt.**

## 12. Flerfaldig testning – när vi testar många saker

Om vi gör **ett** test med `α = 0.05` när H₀ är sann finns en 5-procentig risk för typ I-fel, om testet är korrekt kalibrerat.

Om vi gör **20 oberoende test**, och alla nollhypoteser är sanna, är den förväntade mängden falska positiva resultat `20 × 0.05 = 1`. Det betyder **inte** att exakt ett test måste bli signifikant.

Sannolikheten att **minst ett** av 20 oberoende test blir falskt positivt är:

\[
1 - (1 - 0{,}05)^{20} \approx 0{,}642
\]

Alltså ungefär **64 %** i detta exempel. Vid beroende tester gäller inte samma enkla formel.

```python
alpha = 0.05
number_of_tests = 20

expected_false_positives = number_of_tests * alpha
probability_at_least_one = 1 - (1 - alpha) ** number_of_tests

print(f"Förväntat antal falska positiva: {expected_false_positives:.1f}")
print(f"Sannolikhet för minst ett: {probability_at_least_one:.1%}")
```

**Vad kan vi göra?** Bestäm primärt utfall i förväg, begränsa antalet tester, skilj planerade analyser från utforskande analyser och använd vid behov korrigeringar som **Bonferroni** eller **FDR**. Videon nämner dessa metoder men går inte igenom detaljerna.

## 13. Förberedelser inför ett A/B-test

Innan du tittar på resultaten, bestäm:

1. **Population:** Vilka användare gäller slutsatsen?
2. **Primärt utfall:** Klick, köp, intäkt eller något annat?
3. **Gruppindelning:** Hur slumpas deltagarna till A och B?
4. **Effektmått:** Skillnad i medelvärden eller andelar?
5. **Hypotes:** Tvåsidig eller ensidig – och varför?
6. **Signifikansnivå:** Exempelvis `α = 0.05`.
7. **Praktisk tröskel:** Hur stor effekt krävs för att agera?
8. **Stickprovsstorlek och power:** Hur mycket data behöver samlas in?
9. **Stopptid:** När avslutas experimentet?
10. **Analysplan:** Vilket test och vilket konfidensintervall ska användas?

**Varför?** Om vi väljer utfall, test eller stopptid efter att ha tittat på resultaten kan slutsatsen bli missvisande. Randomisering hjälper oss att jämföra grupperna på ett trovärdigt sätt.

## 14. Snabbguide – vad hör ihop med vad?

| Fråga | Begrepp / metod |
|---|---|
| Vilka vill vi uttala oss om? | Population |
| Vilka har vi data om? | Stickprov |
| Vad är det okända sanna värdet? | Parameter |
| Vad räknar vi ut från datan? | Skattning |
| Hur varierar individerna? | Standardavvikelse (SD) |
| Hur varierar skattningen mellan stickprov? | Standardfel (SE) |
| Vilka parametervärden är förenliga med data? | Konfidensintervall (KI) |
| Hur ovanligt är resultatet under H₀? | P-värde |
| Är skillnaden stor nog att vara värd något? | Praktisk tröskel |
| Hur lätt kan studien upptäcka en viss effekt? | Power |
| Varför blir många tester riskabla? | Flerfaldig testning |

## 15. Vanliga misstag

1. **”Ett stort stickprov löser allt.”** Nej – systematiskt snedvridna urval förblir problematiska.
2. **”SD och SE är samma sak.”** Nej – individspridning respektive skattningens osäkerhet.
3. **”95 % av personerna ligger inom 95 % KI.”** Nej – intervallet gäller en populationsparameter.
4. **”P = 0,03 betyder 3 % chans att H₀ är sann.”** Nej – p-värdet beräknas under antagandet om H₀.
5. **”Inte signifikant betyder ingen effekt.”** Nej – låg power kan ge osäkra resultat.
6. **”Signifikant betyder viktigt.”** Nej – praktisk betydelse kräver en egen bedömning.
7. **”Från 20 % till 25 % är 5 % ökning.”** Nej – det är 5 **procentenheter**, eller 25 % **relativ** ökning.
8. **”Vi kan välja ensidigt test när vi sett resultatet.”** Nej – riktningen måste bestämmas i förväg.
9. **”20 test med α = 0,05 är samma sak som ett test.”** Nej – risken för minst ett falskt positivt fynd ökar.

## 16. Övningar

Försök gärna först, och kontrollera sedan facit.

1. Du vill undersöka genomsnittlig leveranstid för **alla** företagets beställningar, men har data för **400**. Vad är population, stickprov, parameter och skattning?
2. Vad är skillnaden mellan standardavvikelse och standardfel?
3. Om standardavvikelsen är 12 och `n = 36`, vad blir uppskattat standardfel för medelvärdet?
4. Ett 95 % KI är `[48, 54]`. Betyder det att 95 % av individerna har värden mellan 48 och 54?
5. En konverteringsgrad går från 12 % till 15 %. Hur många **procentenheter** ökar den? Hur stor är den **relativa procentuella ökningen**?
6. Vilken typ av jämförelse passar bäst för **köpbelopp** respektive **köpte/inte köpte**?
7. Förklara skillnaden mellan ett ensidigt och ett tvåsidigt test.
8. Ett test ger `p = 0.18` med `α = 0.05`. Vilket beslut tar vi? Har vi bevisat H₀?
9. Företaget kräver minst +5 pp för att införa B. Ett 95 % KI för B − A är `[1, 3]` pp. Vad kan vi säga om statistisk och praktisk betydelse?
10. Ett 95 % KI för B − A är `[6, 9]` pp. Hur skiljer sig slutsatsen från uppgift 9?
11. Vad innebär låg power? Nämn två sätt att förbättra den.
12. Varför bör vi inte testa 20 olika utfall och bara rapportera det minsta p-värdet?
13. **Kodövning:** Ändra `size=80` till `size=800` i exempel 7. Vad händer typiskt med osäkerheten i skillnaden? Varför kan p-värdet också ändras?
14. **Kodövning:** Ändra tröskeln från `5` till `3` i figuren i exempel 10. Hur förändras bedömningen av de tre scenarierna?

## 17. Facit

1. **Population:** alla företagets beställningar. **Stickprov:** 400 beställningar. **Parameter:** verklig genomsnittlig leveranstid för alla beställningar. **Skattning:** medelvärdet för de 400 observerade.
2. **SD** beskriver spridning mellan enskilda observationer. **SE** beskriver osäkerheten i en skattning mellan tänkbara stickprov.
3. `SE = 12 / √36 = 12 / 6 = 2`.
4. **Nej.** Ett 95 % KI gäller en okänd parameter, inte andelen enskilda observationer inom gränserna.
5. **3 procentenheter**; relativ ökning `(15 - 12) / 12 = 0.25 = 25 %`.
6. **Köpbelopp:** jämföra medelvärden, exempelvis Welch-test. **Köpte/inte köpte:** jämföra andelar.
7. **Tvåsidigt:** skillnad åt båda håll. **Ensidigt:** skillnad i en i förväg vald riktning.
8. `0.18 > 0.05`, så vi **kan inte förkasta H₀**. Vi har **inte** bevisat att H₀ är sann.
9. Intervallet ligger över 0, vilket ger statistiskt stöd för en positiv skillnad vid ett förenligt tvåsidigt 5 %-test. Hela intervallet ligger under +5 pp: effekten verkar **inte nå den praktiska tröskeln**.
10. Hela intervallet ligger **över både 0 och +5 pp**. Det ger stöd för en positiv effekt som också är större än den förutbestämda praktiska tröskeln.
11. Låg power betyder att en verklig effekt av den storlek vi vill upptäcka ofta missas. **Större stickprov** eller **mindre mätbrus** kan hjälpa.
12. Vi ökar risken för **falskt positiva fynd** och ger en missvisande bild om bara det mest gynnsamma resultatet visas.
13. Med större grupper blir standardfelet normalt mindre, så skillnaden kan skattas mer precist. Om den verkliga effekten inte är noll kan p-värdet oftare bli mindre; ett enskilt slumpresultat är inte garanterat.
14. Scenario A `[1, 3]` når fortfarande inte tydligt över en tröskel på +3 pp. Scenario B `[6, 8]` ligger fortsatt över tröskeln. Scenario C `[-2, 6]` innehåller både 0 och värden över tröskeln, så osäkerheten om effekten kvarstår.

---

## Sammanfattning av lärarens video

Videon är en **sammanbindande teorilektion** inför A/B-testning. Den repeterar population och stickprov, samplingvariation, SD/SE, konfidensintervall och hypotesprövning. Den introducerar också hur vi väljer metod utifrån utfallstyp, hur vi skiljer statistiskt stöd från praktisk nytta, varför **power** spelar roll och varför **många tester** kan ge falska fynd. Den viktigaste arbetsvanan är att rapportera **skattad effekt + konfidensintervall + p-värde + praktisk tolkning** och att planera analysen innan vi ser resultatet.

**Nästa steg:** Kommande lektioner om A/B-testning bygger vidare på detta arbetssätt.
