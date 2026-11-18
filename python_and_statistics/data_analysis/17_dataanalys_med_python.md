# Lektion 17 – Dataanalys med Python

> **Kurs:** Pythonprogrammering och statistisk dataanalys  
> **Underlag:** Joakims tillämpningslektion och genomgång av restaurangdatasetet `tips`. Pedagogiska tillägg är markerade som **Förklaring** eller **Komplettering**.

[Se kursvideon på YouTube](https://www.youtube.com/watch?v=UVO13ZNn_10)

## 1. Helheten: frågan bestämmer metoden

![Arbetsflöde för statistisk dataanalys](arbetsflode_statistisk_dataanalys.png)

1. Formulera frågan.
2. Förstå datan: rader, kolumner, datatyper och saknade värden.
3. Utforska: sammanfattningar och diagram.
4. Skatta värden och osäkerhet.
5. Testa hypoteser eller anpassa modeller.
6. Kontrollera antaganden och residualer.
7. Tolka i sammanhanget.
8. Kommunicera resultat och begränsningar.

**Förklaring:** Arbetsflödet är inte en enkelriktad kedja. Upptäcker du ett problem i steg 6 kan du behöva gå tillbaka till steg 2 eller 3.

## 2. Fyra frågor i Joakims analys

| Analysfråga | Metod | Varför? |
|---|---|---|
| Hur stor är genomsnittlig dricksandel? | Medelvärde + 95 % konfidensintervall | Skatta ett medelvärde och dess osäkerhet |
| Är dricksandelen högre vid middag än lunch? | Ensidigt Welch t-test | Jämför medelvärden i två oberoende grupper, med bestämd riktning |
| Skiljer sig andelen rökare mellan helg och vardag? | Tvåsidigt z-test för två proportioner | Jämför två andelar utan förutbestämd riktning |
| Hur samvarierar dricks i dollar med notans storlek? | Linjär regression + residualdiagram | Undersök ett numeriskt samband |

## 3. Projekt och bibliotek

Joakim skapar en virtuell miljö och använder NumPy, Pandas, Matplotlib, Seaborn, SciPy, statsmodels och scikit-learn.

```bash
python -m pip install numpy pandas matplotlib seaborn scipy statsmodels scikit-learn ipykernel
```

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats
from statsmodels.stats.proportion import proportions_ztest, confint_proportions_2indep
from sklearn.linear_model import LinearRegression
```

## 4. Läs in och förstå data

```python
tips = sns.load_dataset('tips').copy()
print('Antal rader:', len(tips))
display(tips.head())
tips.info()
display(tips.describe())
```

Datasetet har **244 restaurangnotor**. Varje rad är en nota, inte nödvändigtvis en unik kund. Kolumnerna innehåller bland annat `total_bill`, `tip`, `sex`, `smoker`, `day`, `time` och `size`.

### Skapa nya variabler

```python
tips['tip_pct'] = tips['tip'] / tips['total_bill']
tips['weekend'] = tips['day'].isin(['Sat', 'Sun']).map({True: 'weekend', False: 'weekday'})
```

**Förklaring:** `tip_pct` är en andel: `0.16` betyder **16 %**. Multiplicera med 100 först när du vill visa den som procent. `isin` ger `True` eller `False` för varje rad; `map` översätter detta till gruppnamn.

## 5. EDA: utforska före test

```python
plt.hist(tips['tip_pct'] * 100, bins=12, edgecolor='black')
plt.xlabel('Dricksandel (%)')
plt.ylabel('Antal notor')
plt.title('Fördelning av dricksandel')
plt.show()

lunch = tips.loc[tips['time'] == 'Lunch', 'tip_pct'] * 100
dinner = tips.loc[tips['time'] == 'Dinner', 'tip_pct'] * 100
plt.boxplot([lunch, dinner], tick_labels=['Lunch', 'Middag'])
plt.ylabel('Dricksandel (%)')
plt.show()

plt.scatter(tips['total_bill'], tips['tip'], alpha=0.6)
plt.xlabel('Total nota (dollar)')
plt.ylabel('Dricks (dollar)')
plt.show()
```

**Förklaring:** Histogram visar *fördelningen* av en numerisk variabel. Boxplot hjälper oss jämföra gruppernas median och spridning. Spridningsdiagram hjälper oss se samband mellan två numeriska variabler. Diagram ger ledtrådar, inte bevis för statistisk signifikans.

## 6. Medelvärde och 95 % konfidensintervall

Joakim beräknar den genomsnittliga dricksandelen både med normalapproximation och bootstrap.

$$SE=\frac{s}{\sqrt{n}}$$

$$KI_{95\%}\approx\bar{x}\pm1.96\cdot SE$$

**Förklaring:** $s$ är stickprovets standardavvikelse, $n$ antal observationer och $SE$ standardfelet – ett mått på hur mycket ett skattat medelvärde brukar variera mellan stickprov.

```python
varden = tips['tip_pct'].to_numpy()
n = len(varden)
medel = np.mean(varden)
s = np.std(varden, ddof=1)
se = s / np.sqrt(n)
ki_normal = (medel - 1.96 * se, medel + 1.96 * se)
print('Medel (%):', medel * 100)
print('Normal-KI (%):', np.array(ki_normal) * 100)
```

### Bootstrap: upprepa stickprov med återläggning

```python
rng = np.random.default_rng(42)
boot_medel = []
for _ in range(10_000):
    urval = rng.choice(varden, size=n, replace=True)
    boot_medel.append(urval.mean())
ki_boot = np.percentile(boot_medel, [2.5, 97.5])
print('Bootstrap-KI (%):', ki_boot * 100)
```

**Förklaring:** `replace=True` betyder att samma ursprungliga observation får dras flera gånger. Vi räknar ett nytt medelvärde för vart och ett av 10 000 bootstrapstickprov och använder deras 2,5:e och 97,5:e percentil.

**Joakims resultat:** genomsnittlig dricksandel cirka **16,1 %** och ett 95-procentigt intervall på ungefär **15,4–16,8 %**.

**Komplettering:** Ett 95 % konfidensintervall betyder att metoden vid upprepad stickprovstagning täcker det sanna populationsvärdet i cirka 95 % av fallen under relevanta antaganden. Bootstrap är inte antagandefritt; urval och beroenden spelar roll.

## 7. Ensidigt Welch-test: middag mot lunch

Frågan är **om middag har högre genomsnittlig dricksandel än lunch**. Riktningen ska väljas innan resultaten undersöks.

$$H_0:\mu_{middag}=\mu_{lunch}$$

$$H_1:\mu_{middag}>\mu_{lunch}$$

```python
middag = tips.loc[tips['time'] == 'Dinner', 'tip_pct'].to_numpy()
lunch = tips.loc[tips['time'] == 'Lunch', 'tip_pct'].to_numpy()
resultat = stats.ttest_ind(middag, lunch, equal_var=False, alternative='greater')
print('Middag (%):', middag.mean() * 100)
print('Lunch (%):', lunch.mean() * 100)
print('Skillnad (procentenheter):', (middag.mean() - lunch.mean()) * 100)
print('p-värde:', resultat.pvalue)
```

**Förklaring:** `equal_var=False` väljer Welchs metod, som inte kräver att gruppernas populationsvarianser är lika. `alternative='greater'` testar om den **första** gruppens medelvärde är större.

**Joakims resultat:** middag cirka 15,9 %, lunch cirka 16,4 %, skillnad cirka **−0,5 procentenheter**, p-värde cirka **0,74**. Vi har **inte stöd** för att dricksandelen är högre vid middag. Det betyder inte att grupperna bevisats lika.

## 8. Tvåsidigt proportionstest: rökare helg/vardag

Här jämför vi två **andelar**, inte två medelvärden. Frågan har ingen bestämd riktning.

$$H_0:p_{helg}=p_{vardag}$$

$$H_1:p_{helg}\ne p_{vardag}$$

```python
helg = tips['day'].isin(['Sat', 'Sun'])
rokare = tips['smoker'].eq('Yes')

antal_rokare = np.array([(rokare & helg).sum(), (rokare & ~helg).sum()])
antal_notor = np.array([helg.sum(), (~helg).sum()])
andelar = antal_rokare / antal_notor

z, p = proportions_ztest(antal_rokare, antal_notor, alternative='two-sided')
ki = confint_proportions_2indep(
    count1=int(antal_rokare[0]), nobs1=int(antal_notor[0]),
    count2=int(antal_rokare[1]), nobs2=int(antal_notor[1]),
    compare='diff', method='newcomb'
)
print('Andelar (%):', andelar * 100)
print('Skillnad (procentenheter):', (andelar[0] - andelar[1]) * 100)
print('p-värde:', p)
print('95 % KI för skillnaden:', ki)
```

**Förklaring:** `&` betyder ”och” för två booleska serier, `~` betyder ”inte”. När vi summerar `True`/`False` räknas `True` som 1. `compare='diff'` betyder att intervallet avser **skillnaden** mellan andelarna.

**Joakims resultat:** helg cirka **37,4 %**, vardag cirka **39,5 %**, skillnad cirka **−2,1 procentenheter**, p-värde cirka **0,75**. Intervallet för skillnaden sträcker sig ungefär från **−0,151 till +0,105** uttryckt som andelar. Eftersom intervallet innehåller noll kan analysen inte påvisa en tydlig skillnad.

## 9. Enkel linjär regression: nota och dricks

```python
X = tips[['total_bill']].to_numpy()  # 2D: rader × variabler
y = tips['tip'].to_numpy()

model = LinearRegression()
model.fit(X, y)
y_hat = model.predict(X)
residualer = y - y_hat

print('Intercept:', model.intercept_)
print('Lutning:', model.coef_[0])
print('R²:', model.score(X, y))

x_linje = np.linspace(X.min(), X.max(), 100).reshape(-1, 1)
plt.scatter(X[:, 0], y, alpha=0.6)
plt.plot(x_linje[:, 0], model.predict(x_linje), color='black')
plt.xlabel('Nota (dollar)')
plt.ylabel('Dricks (dollar)')
plt.show()

plt.scatter(y_hat, residualer, alpha=0.6)
plt.axhline(0, color='black', linestyle='--')
plt.xlabel('Förutsagd dricks')
plt.ylabel('Residual')
plt.show()
```

**Joakims resultat:** $R^2\approx0.457$, alltså cirka **46 % förklarad variation** i datasetet. En ökning med **10 dollar i notan** motsvarar cirka **1,05 dollar högre förutsagd dricks** enligt modellen. Residualerna såg ut som ett relativt oregelbundet moln kring noll.

**Komplettering:** Det är inte bevis för att större nota *orsakar* högre dricks. Och $R^2$ på träningsdata är inte en utvärdering av hur modellen fungerar på nya data.

## 10. Så skriver du en slutsats

En användbar slutsats besvarar frågan, anger resultat och osäkerhet och beskriver vad som **inte** går att slå fast.

| Fråga | Försiktig slutsats från genomgången |
|---|---|
| Genomsnittlig dricksandel | Cirka 16,1 %; 95 % KI ungefär 15,4–16,8 % |
| Högre vid middag? | Nej, analysen fann inte stöd för den riktade hypotesen ($p\approx0.74$) |
| Skillnad i rökandel helg/vardag? | Ingen statistiskt säkerställd skillnad i urvalet ($p\approx0.75$) |
| Samband nota–dricks? | Positivt samband; modellen förklarar cirka 46 % av variationen |

Datasetet innehåller 244 notor i ett särskilt sammanhang. Resultaten kan inte automatiskt generaliseras till alla restauranger, och observationsdata ger inte säkra orsaksslutsatser.

## 11. Gör notebooken reproducerbar

- Skriv en kort analysfråga före varje beräkning och en tolkning efter.
- Använd ett fast slumpfrö, exempelvis `np.random.default_rng(42)`.
- Håll isär **andelar** (0,16), **procent** (16 %) och **procentenheter** (skillnaden mellan exempelvis 16 % och 15 % är 1 procentenhet).
- Kontrollera datatyper, saknade värden och eventuella avvikare.
- Kör **Restart & Run All** före inlämning.

## 12. Koppling till din Health Study

Du har redan arbetat med beskrivande statistik och figurer. Samma arbetsflöde kan användas för att formulera en fråga om blodtryck, ålder eller kolesterol, välja rätt diagram, uppskatta osäkerhet och tolka resultat. Börja alltid med **frågan och variabeltypen** innan du väljer ett test.

---

**Källa:** Joakims lektion *Dataanalys med Python*, kursens sammanfattning och transkriberad videogenomgång. **Pedagogiska kompletteringar:** förklaringar, exempel, strukturerad och förenklad Pythonkod samt förtydliganden om tolkning och begränsningar.
