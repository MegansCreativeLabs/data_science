# Lektion 16 – Linjär regression

> **Kurs:** Pythonprogrammering och statistisk dataanalys  
> **Syfte:** Förstå hur en rak linje används för att beskriva samband och göra prognoser.

## 1. Varför använder vi regression?

Vi vill ibland undersöka hur en **numerisk variabel** hänger ihop med en annan. Exempel: Hur förändras dricks i dollar när notans storlek ökar? Vi kallar notans storlek för **förklarande variabel** (`x`) och dricksen för **responsvariabel** (`y`).

En linjär regression anpassar en rak linje till observationerna. Den beskriver ett *genomsnittligt samband* – inte en regel som varje enskild observation måste följa.

## 2. Modellen och symbolerna

$$\hat{y}=b_0+b_1x$$

| Symbol | Betydelse |
|---|---|
| $x$ | Förklarande variabel, exempelvis notans storlek |
| $y$ | Faktiskt observerat värde, exempelvis faktisk dricks |
| $\hat{y}$ | Modellens förutsagda värde (”y-hatt”) |
| $b_0$ | Skärning/intercept: modellens värde när $x=0$ |
| $b_1$ | Lutning/koefficient: förändring i prognosen när $x$ ökar med 1 |

**Exempel (påhittade värden):** Om modellen är $\hat{y}=1+0.10x$, blir prognosen för en nota på 30 dollar:

$$\hat{y}=1+0.10\cdot30=4$$

Modellen förutsäger 4 dollar i dricks. Lutningen 0,10 betyder 10 cent högre **förutsagd** dricks per extra dollar på notan. Interceptet 1 dollar vid en nota på 0 dollar behöver inte vara praktiskt rimligt.

**Viktigt:** Samband betyder inte automatiskt orsakssamband.

## 3. Residualer: hur fel blev prognosen?

$$e=y-\hat{y}$$

Om faktisk dricks var 5 dollar men modellen förutsade 4 dollar, blir residualen $5-4=+1$. Observationen ligger **över** linjen. Om faktisk dricks var 3 dollar blir residualen $3-4=-1$, alltså **under** linjen.

### Varför minsta kvadratmetoden?

Vi vill hitta en linje med små avvikelser. Att bara summera residualerna fungerar dåligt eftersom plus och minus kan ta ut varandra. Därför minimeras summan av **kvadrerade** residualer:

$$\mathrm{SSE}=\sum_{i=1}^{n}(y_i-\hat{y}_i)^2$$

Kvadreringen gör att stora fel får extra stor betydelse. **Komplettering:** Detta är en matematisk regel för hur linjen väljs, inte ett bevis för att linjen är en bra beskrivning av verkligheten.

## 4. Kontrollera med residualdiagram

Rita residualerna mot modellens förutsagda värden och en horisontell linje vid noll.

- **Oregelbundet moln runt 0:** inget uppenbart systematiskt mönster.
- **Böjd form:** en rak linje kanske inte beskriver sambandet väl.
- **Tratt:** spridningen varierar med prognosens storlek.
- **Avvikande punkt:** undersök om den är korrekt och om den påverkar modellen mycket.

Ett bra utseende är ett positivt tecken, men bevisar inte att alla modellantaganden är uppfyllda.

## 5. Vad betyder $R^2$?

$$R^2=1-\frac{\sum_i(y_i-\hat{y}_i)^2}{\sum_i(y_i-\bar{y})^2}$$

Nämnaren beskriver variationen kring medelvärdet, medan täljaren beskriver modellens kvarvarande kvadrerade fel. Ett $R^2=0.77$ betyder att modellen förklarar cirka **77 % av variationen i detta dataset** jämfört med att alltid förutsäga medelvärdet.

Det betyder **inte** att 77 % av prognoserna är korrekta, att sambandet är kausalt eller att modellen fungerar lika bra på nya data. $R^2$ kan också bli negativt på testdata.

## 6. Python: bygg och tolka en modell

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression

# Påhittade observationer: nota och dricks i dollar
nota = np.array([10, 15, 20, 25, 30, 35], dtype=float)
dricks = np.array([1.5, 2.2, 2.4, 3.8, 3.5, 4.7])

# sklearn vill ha X som en 2D-matris: (antal rader, antal variabler)
X = nota.reshape(-1, 1)
y = dricks

model = LinearRegression()
model.fit(X, y)  # Lär sig skärning och lutning från observationerna

print('Skärning:', model.intercept_)
print('Lutning:', model.coef_[0])
print('R² på träningsdata:', model.score(X, y))

# Förutsäg dricks för en nota på 28 dollar
print('Prognos:', model.predict([[28]])[0])

y_hat = model.predict(X)
residualer = y - y_hat

plt.scatter(nota, y, label='Observationer')
plt.plot(nota, y_hat, label='Regressionslinje')
plt.xlabel('Nota (dollar)')
plt.ylabel('Dricks (dollar)')
plt.legend()
plt.show()

plt.scatter(y_hat, residualer)
plt.axhline(0, color='black', linestyle='--')
plt.xlabel('Förutsagd dricks')
plt.ylabel('Residual: faktisk − förutsagd')
plt.show()
```

### Kodord att känna igen

| Kod | Vad händer? |
|---|---|
| `fit(X, y)` | Anpassar linjen till data |
| `intercept_` | Hämtar den skattade skärningen |
| `coef_[0]` | Hämtar lutningen för första variabeln |
| `predict(X_new)` | Beräknar prognoser |
| `score(X, y)` | Beräknar $R^2$ |
| `reshape(-1, 1)` | Gör en kolumn med flera observationer till en 2D-matris |

## 7. Träningsdata, interpolation och extrapolation

**Interpolation** är prognoser inom intervallet vi har observerat. **Extrapolation** är prognoser utanför intervallet, exempelvis en nota på 100 dollar när modellen tränats på 10–35 dollar. Extrapolation är mer osäker eftersom vi inte vet om sambandet fortsätter.

**Komplettering:** För att bedöma prognoser på nya observationer bör modellen utvärderas på separat testdata, inte bara på samma observationer som användes vid `fit`.

## 8. Koppling till Health Study

Du kan exempelvis undersöka om systoliskt blodtryck ($y$) samvarierar med ålder ($x$). Titta först på ett spridningsdiagram, anpassa modellen, tolka lutningen och granska residualerna. En sådan analys visar ett samband i data, inte att ålder ensamt orsakar en viss förändring.

## Kom ihåg

**Modell → prognos → residual → kontroll → tolkning.** En hög siffra på $R^2$ ersätter aldrig en kontroll av om modellen är rimlig.
