# Lektion 01 – Deskriptiv statistik

**Kursdel:** Vecka 4 – Deskriptiv statistik och bibliotek för dataanalys  
**Placering i GitHub:** `data_science/data_analysis/01_descriptive_statistics.md`  
**Underlag:** Lärarens video 4.1 – Deskriptiv statistik

## Vad ska du lära dig?

Deskriptiv statistik betyder **beskrivande statistik**. Vi använder tal och diagram för att sammanfatta data som vi faktiskt har samlat in. Målet är att kunna svara på tre frågor:

1. **Nivå:** Vad är ett typiskt värde?
2. **Spridning:** Hur mycket skiljer sig värdena åt?
3. **Form och samband:** Hur är värdena fördelade, och verkar två variabler hänga ihop?

Vi beskriver först de data vi har. Att dra slutsatser om en större grupp utifrån ett urval hör till **statistisk inferens**, ett annat område.

> **Kom ihåg:** Ett enda medelvärde kan dölja stora skillnader mellan två grupper. Titta ofta på både läge och spridning.

---

## 1. Vilken typ av data har vi?

En **variabel** är något vi mäter eller registrerar, till exempel en elevs studietimmar, provpoäng eller studiemetod. En **observation** är ett registrerat värde för en viss person eller händelse.

### 1.1 Kategorisk data

Kategorisk data delar in observationer i grupper. Vi räknar exempelvis **antal**, **andelar** och **typvärde** (vanligaste kategorin).

| Typ | Förklaring | Exempel |
|---|---|---|
| **Nominal** | Kategorier utan inbördes ordning | Bilmärke, stad, studiemetod |
| **Ordinal** | Kategorier med naturlig ordning, men inte nödvändigtvis lika avstånd | Låg–medel–hög, instämmer inte–instämmer helt |

Om enkätsvar kodas som 1, 2 och 3 blir de inte automatiskt numeriska mätvärden. Siffrorna kan bara vara etiketter.

**Passande diagram:** stapeldiagram.

### 1.2 Numerisk data

Numerisk data uttrycker mängder där skillnader mellan talen har mening.

| Typ | Förklaring | Exempel |
|---|---|---|
| **Diskret** | Räkningsbara antal, ofta heltal | Antal barn, antal beställningar |
| **Kontinuerlig** | Mätvärden som kan ligga mellan två tal | Tid, längd, avstånd |

Ett kontinuerligt värde kan visas avrundat, exempelvis `7.5` timmar, utan att mätningen egentligen bara kan anta tiondelar.

**Passande diagram:** histogram och låddiagram (boxplot). För två numeriska variabler kan vi använda spridningsdiagram.

---

## 2. Population, urval, parameter och stickprovsstatistik

Tänk att en skola vill veta hur många timmar eleverna studerade inför ett prov.

- **Population:** Alla elever som undersökningen handlar om, exempelvis samtliga elever i klassen.
- **Urval (stickprov):** De elever vi faktiskt samlar in uppgifter från, exempelvis 12 elever.
- **Parameter:** Ett mått för hela populationen, till exempel klassens verkliga medelvärde.
- **Stickprovsstatistik:** Ett mått beräknat från urvalet, exempelvis de 12 elevernas medelvärde.

Ett urvalsmedelvärde beskriver urvalet exakt, men är inte automatiskt exakt samma som populationens medelvärde.

---

## 3. Lägesmått – var ligger mitten?

Vi använder en liten egen exempelserie för att kunna kontrollräkna alla steg:

```python
studietimmar = [2, 4, 4, 6, 8, 12]
```

### 3.1 Medelvärde (mean)

Addera alla värden och dela med antalet värden:

\[
\text{medelvärde} = \frac{2+4+4+6+8+12}{6} = \frac{36}{6}=6
\]

```python
medel = sum(studietimmar) / len(studietimmar)
print(medel)  # 6.0
```

- `sum(...)` summerar talen.
- `len(...)` räknar hur många tal det finns.
- `/` betyder division.

**Svaghet:** Medelvärdet påverkas mycket av extremt höga eller låga värden.

### 3.2 Median (median)

Sortera talen och hitta mitten.

- **Udda antal:** det mittersta värdet.
- **Jämnt antal:** medelvärdet av de två mittersta.

I vår serie är de mittersta talen `4` och `6`:

\[
\text{median} = \frac{4+6}{2}=5
\]

```python
from statistics import median
print(median(studietimmar))  # 5.0
```

**Styrka:** Medianen påverkas mindre av enstaka extremvärden.

### 3.3 Typvärde (mode)

Det värde som förekommer oftast. Här är det `4` eftersom det förekommer två gånger.

```python
from statistics import mode
print(mode(studietimmar))  # 4
```

Typvärde är särskilt användbart för kategorisk data, exempelvis vilken studiemetod som är vanligast. Det kan finnas flera lika vanliga värden.

### 3.4 Varför räcker inte medelvärdet?

Jämför:

```python
grupp_a = [5, 5, 5, 5, 5]
grupp_b = [1, 1, 1, 2, 20]

print(sum(grupp_a) / len(grupp_a))  # 5.0
print(sum(grupp_b) / len(grupp_b))  # 5.0
```

Båda har **samma medelvärde**, men grupp A är helt jämn och grupp B har stor spridning. Grupp B:s median är dessutom `1`, inte `5`.

---

## 4. Spridningsmått – hur olika är värdena?

### 4.1 Minsta och största värdet, variationsbredd

För `[2, 4, 4, 6, 8, 12]`:

- Minsta värdet: `2`
- Största värdet: `12`
- **Variationsbredd:** `12 - 2 = 10`

```python
print(min(studietimmar))                 # 2
print(max(studietimmar))                 # 12
print(max(studietimmar) - min(studietimmar))  # 10
```

Variationsbredden är lätt att förstå men mycket känslig för extremvärden.

### 4.2 Kvartiler och IQR (interkvartilavstånd)

**Kvartiler** delar upp en sorterad datamängd i fyra delar:

- **Q1:** ungefär 25 % av värdena ligger på eller under denna nivå.
- **Q2:** medianen, ungefär 50 %.
- **Q3:** ungefär 75 % ligger på eller under denna nivå.

**IQR = Q3 − Q1**. Det beskriver bredden på de mittersta 50 procenten av värdena.

Exempel: om `Q1 = 4` och `Q3 = 10` är `IQR = 6`.

> **Viktigt:** Det finns flera etablerade metoder att beräkna kvartiler. Små datamängder kan därför ge olika Q1/Q3 i olika program. Ange metod om exakta gränsvärden är viktiga.

### 4.3 Varians

Variansen mäter hur mycket värdena varierar kring medelvärdet.

1. Beräkna medelvärdet.
2. Räkna varje värdes avvikelse från medelvärdet.
3. Kvadrera avvikelserna så att minus och plus inte tar ut varandra.
4. Summera och dela.

För **hela populationen** delar vi med `n`:

\[
\sigma^2 = \frac{\sum_{i=1}^{n}(x_i-\mu)^2}{n}
\]

För ett **stickprov** används vanligtvis `n - 1` vid uppskattning av populationsvariansen:

\[
s^2 = \frac{\sum_{i=1}^{n}(x_i-\bar{x})^2}{n-1}
\]

Här betyder `xᵢ` ett enskilt värde, `μ` populationsmedelvärdet, `x̄` urvalsmedelvärdet och `n` antalet värden.

Variansen uttrycks i **kvadrerade enheter**, exempelvis timmar², vilket är svårt att tolka direkt.

### 4.4 Standardavvikelse

**Standardavvikelsen** är kvadratroten ur variansen. Den får därmed samma enhet som ursprungsdata, exempelvis timmar.

```python
import statistics

print(statistics.pvariance(studietimmar))  # Populationsvarians
print(statistics.pstdev(studietimmar))     # Populationsstandardavvikelse
print(statistics.variance(studietimmar))   # Stickprovsvarians
print(statistics.stdev(studietimmar))      # Stickprovsstandardavvikelse
```

- **Låg standardavvikelse:** värdena ligger relativt nära medelvärdet.
- **Hög standardavvikelse:** värdena varierar mer.

Det är en användbar beskrivning av spridning, men inte ett exakt påstående om hur långt varje observation ligger från medelvärdet.

---

## 5. Normalfördelning och 68–95–99,7-regeln

En **normalfördelning** är en symmetrisk, klockformad fördelning. Flest värden ligger nära mitten och färre långt från den.

Om en variabel är ungefär normalfördelad gäller tumregeln:

| Avstånd från medelvärdet | Ungefärlig andel |
|---|---:|
| Inom ±1 standardavvikelse | 68 % |
| Inom ±2 standardavvikelser | 95 % |
| Inom ±3 standardavvikelser | 99,7 % |

Det här är **inte en allmän regel för all data**. Använd den inte okritiskt på kraftigt skeva fördelningar.

---

## 6. Z-poäng – hur ovanligt är ett värde?

En **z-poäng** visar hur många standardavvikelser ett värde ligger från medelvärdet.

\[
z = \frac{x - \text{medelvärde}}{\text{standardavvikelse}}
\]

Om medelvärdet är `7.08` timmar, standardavvikelsen `3.65` och en elev har studerat `14` timmar:

```python
z = (14 - 7.08) / 3.65
print(round(z, 2))  # 1.90
```

Tolkning: 14 timmar ligger cirka **1,9 standardavvikelser över** medelvärdet.

- `z = 0`: precis på medelvärdet.
- `z > 0`: över medelvärdet.
- `z < 0`: under medelvärdet.

Z-poäng gör det lättare att jämföra värden som mäts på olika skalor. Hur *ovanligt* ett z-värde är beror dock på fördelningens form.

---

## 7. Extremvärden (outliers) och 1,5 × IQR-regeln

Ett **extremvärde** är ett värde som sticker ut från övriga observationer.

En vanlig metod för att flagga sådana värden är:

\[
\text{Nedre gräns} = Q1 - 1.5 \times IQR
\]

\[
\text{Övre gräns} = Q3 + 1.5 \times IQR
\]

Om `Q1 = 4`, `Q3 = 10` och `IQR = 6` blir gränserna:

- Nedre: `4 - 1.5 × 6 = -5`
- Övre: `10 + 1.5 × 6 = 19`

Värdet `25` skulle då flaggas eftersom `25 > 19`.

**Ett flaggat värde är inte automatiskt fel.** Det kan vara en verklig observation, ett mätfel eller en särskild händelse. Undersök det innan du tar bort något.

---

## 8. Välj rätt diagram

| Diagram | Passar för | Vad du ser |
|---|---|---|
| **Stapeldiagram** | Kategorier | Antal eller andel per kategori |
| **Histogram** | Numeriska värden | Hur värden fördelar sig i intervall (*bins*) |
| **Låddiagram / boxplot** | Numeriska värden | Median, kvartiler, IQR och möjliga extremvärden |
| **Spridningsdiagram / scatter plot** | Två numeriska variabler | Mönster och möjliga samband |

**Stapeldiagram kontra histogram:** Ett stapeldiagram jämför namngivna kategorier. Ett histogram grupperar numeriska värden i sammanhängande intervall. Hur breda intervall du väljer kan påverka hur fördelningen ser ut.

---

## 9. Korrelation – hänger två variabler ihop?

Om varje elev har både **studietimmar** och **provpoäng** kan vi göra ett spridningsdiagram. Varje punkt motsvarar en elev.

**Pearsons korrelationskoefficient**, `r`, beskriver styrka och riktning på ett **linjärt** samband:

| r-värde | Tolkning |
|---|---|
| Nära `+1` | Starkt positivt linjärt samband |
| Nära `0` | Svagt eller inget linjärt samband |
| Nära `−1` | Starkt negativt linjärt samband |

Ett värde på `r = 0.99`, som i lärarens exempel, betyder ett mycket starkt positivt linjärt samband i de observerade uppgifterna.

**Korrelation är inte kausalitet.** Att två saker samvarierar bevisar inte att den ena orsakar den andra. Det kan finnas andra förklaringar.

---

## 10. Ett första Python-exempel: sammanfatta data

Du behöver inte kunna NumPy och Pandas ännu. Här använder vi Pythons inbyggda funktioner och modulen `statistics`.

```python
import statistics

studietimmar = [2, 4, 4, 6, 8, 12]

print("Antal:", len(studietimmar))
print("Medelvärde:", statistics.mean(studietimmar))
print("Median:", statistics.median(studietimmar))
print("Typvärde:", statistics.mode(studietimmar))
print("Min:", min(studietimmar))
print("Max:", max(studietimmar))
print("Standardavvikelse (urval):", round(statistics.stdev(studietimmar), 2))
```

**Exempelutskrift:**

```text
Antal: 6
Medelvärde: 6
Median: 5.0
Typvärde: 4
Min: 2
Max: 12
Standardavvikelse (urval): 3.58
```

Senare i vecka 4 gör vi samma slags sammanfattningar med **NumPy och Pandas**, och ritar diagram med **Matplotlib**.

---

## 11. Vanliga misstag

1. **Att räkna medelvärde på kategorier** som bara kodats med siffror.
2. **Att redovisa bara medelvärdet** när spridningen är stor eller fördelningen skev.
3. **Att blanda ihop population och urval**, särskilt när varians och standardavvikelse beräknas.
4. **Att anta att ett extremvärde är fel** och radera det utan undersökning.
5. **Att använda 68–95–99,7-regeln på all data**, även när den inte är normalfördelad.
6. **Att tolka korrelation som bevis för orsak.**
7. **Att blanda ihop histogram och stapeldiagram.**

---

## 12. Quick Reference

| Begrepp | Kort förklaring |
|---|---|
| Deskriptiv statistik | Beskriver data vi har observerat |
| Population | Hela gruppen vi är intresserade av |
| Urval | Den del av gruppen vi har data från |
| Medelvärde | Summan delad med antalet |
| Median | Mitten i sorterad data |
| Typvärde | Vanligaste värdet |
| Variationsbredd | Max minus min |
| Q1 och Q3 | Nedre och övre kvartilen |
| IQR | Q3 minus Q1; spridning i mittenhälften |
| Varians | Genomsnittlig kvadrerad avvikelse, med vald nämnare |
| Standardavvikelse | Kvadratroten ur variansen |
| Z-poäng | Avstånd från medelvärdet i standardavvikelser |
| Outlier | Värde som sticker ut |
| Korrelation | Mått på samvariation; Pearsons r gäller linjärt samband |
| Kausalitet | Att något orsakar något annat |

---

## 13. Övningar

**Övning 1 – Datatyper**  
Avgör om variablerna är nominala, ordinala, diskreta eller kontinuerliga: a) bilmärke, b) kundnöjdhet låg/medel/hög, c) antal köp, d) leveranstid i minuter.

**Övning 2 – Lägesmått**  
Beräkna medelvärde, median och typvärde för `[2, 3, 3, 4, 8]`.

**Övning 3 – Samma medelvärde**  
Två team har handläggningstiderna A: `[5, 5, 5, 5]` och B: `[1, 1, 2, 16]`. Vad är medelvärdet i respektive team? Vilket team är jämnare?

**Övning 4 – IQR**  
Om `Q1 = 6` och `Q3 = 14`, vad är IQR? Vilka gränser ger 1,5 × IQR-regeln?

**Övning 5 – Z-poäng**  
Ett prov har medelvärde 70 poäng och standardavvikelse 10. Vilken z-poäng får någon med 85 poäng?

**Övning 6 – Välj diagram**  
Välj ett diagram för a) favoritprogrammeringsspråk, b) fördelningen av leveranstider, c) samband mellan pris och antal sålda enheter.

**Övning 7 – Tolkning**  
En undersökning hittar `r = 0.9` mellan studietimmar och provpoäng. Vad får vi säga? Vad får vi *inte* säga med säkerhet?

---

## 14. Facit med förklaringar

**1.** a) Nominal: namn utan ordning. b) Ordinal: nivåer med ordning. c) Diskret: antal. d) Kontinuerlig: mätt tid.

**2.** Summa `20`, antal `5`, medelvärde `4`. Sorterade tal `[2, 3, 3, 4, 8]`, median `3`. Typvärde `3`.

**3.** A: `(5+5+5+5)/4 = 5`. B: `(1+1+2+16)/4 = 5`. Team A är jämnare: alla värden är lika. Team B har en mycket hög tid.

**4.** `IQR = 14 - 6 = 8`. Nedre gräns `6 - 1.5 × 8 = -6`. Övre gräns `14 + 1.5 × 8 = 26`.

**5.** `z = (85 - 70)/10 = 1.5`. Resultatet ligger 1,5 standardavvikelser över medelvärdet.

**6.** a) Stapeldiagram. b) Histogram (eller boxplot om fokus är median och extremvärden). c) Spridningsdiagram.

**7.** Vi får säga att urvalet visar ett starkt positivt linjärt samband. Vi får inte slå fast att fler studietimmar *orsakar* högre provpoäng enbart utifrån korrelationen.

---

## 15. Sammanfattning – det viktigaste att bära med sig

- Börja med att avgöra **vilken typ av data** du har.
- **Medelvärde** är användbart men påverkas av extremvärden; **median** är robustare.
- **Standardavvikelse** och **IQR** beskriver spridning på olika sätt.
- **Boxplot** hjälper dig att upptäcka möjliga extremvärden, men säger inte automatiskt att de är fel.
- **Spridningsdiagram** och **korrelation** visar samband, inte bevisad orsak.
- Välj mått och diagram efter **frågan du vill besvara** och **hur datan ser ut**.

**Nästa lektion:** `02_numpy.md` – NumPy och numeriska arrayer.

*Notering om videounderlaget: Lärarens video refererar till en tabell med 12 elevers värden som inte syns i transkriberingen. Därför använder räkneexemplen i denna lektion en tydligt separat, egen liten datamängd i stället för att hitta på videons saknade tabell.*
