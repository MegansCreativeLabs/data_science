# Lektion 15 – Linjär algebra: från vektorer till regression

> **Kurs:** Pythonprogrammering och statistisk dataanalys  
> **Område:** Linjär algebra och grunder för regression  
> **Underlag:** Joakims genomgång *Linjär algebra* (cirka 43 minuter), kursens lektionstext och figur.  
> **Video:** [Linjär algebra – Joakim](https://www.youtube.com/watch?v=joWRgVkJ8Ec)  
> **Föregående lektion:** `14_power_and_multiple_testing.md`  
> **Nästa steg:** Linjär regression  
>
> **Om exemplen:** De matematiska huvudexemplen följer Joakims genomgång. NumPy-koden, liknelserna och vissa fördjupningar är pedagogiska kompletteringar. Några förenklingar i genomgången nyanseras för matematisk korrekthet.

## Vad ska du kunna efter lektionen?

- Förklara vad en **vektor** är och varför ordningen på talen spelar roll.
- Tolka **dimension**, **längd (norm)** och **enhetsvektor**.
- Skala och addera vektorer.
- Beräkna och tolka **skalärprodukt (prickprodukt)**.
- Förstå **projektion** som en skugga på en riktning.
- Läsa en **matris** och förstå dess antal rader och kolumner.
- Multiplicera en matris med en vektor på två sätt.
- Förklara varför **projektion** och **minsta kvadratmetoden** leder vidare till **linjär regression**.
- Känna igen hur detta ser ut i NumPy, utan att behöva göra stora matrisberäkningar för hand.

---

## 1. Varför lär vi oss linjär algebra?

Linjär algebra handlar om att arbeta med flera tal **tillsammans**. Ett enskilt tal kan beskriva en persons ålder. En vektor kan beskriva flera egenskaper hos samma person, eller samma egenskap hos många personer. En matris kan samla en hel tabell med observationer.

I **Health Study** kan en rad exempelvis innehålla ålder, vikt och blodtryck. När vi senare bygger en regressionsmodell vill vi använda vissa av dessa värden för att förutsäga ett annat. NumPy och scikit-learn utför då beräkningar som bygger på linjär algebra.

**Den röda tråden:**

`Vektorer → räkna med vektorer → matriser → kombinera variabler → hitta bästa approximation → regression`

Vi börjar med den matematiska idén, inte med avancerad kod.

## 2. Vektorer – en ordnad samling tal

En **vektor** är en samling tal där **ordningen spelar roll**.

Exempel:

$$
v=(3,4)
$$

Den har två **komponenter**: 3 och 4. Vektorn `(4, 3)` är en annan vektor.

### 2.1 Vad betyder dimension?

Antalet komponenter anger vektorns dimension:

| Vektor | Antal komponenter | Dimension |
|---|---:|---|
| `(3, 4)` | 2 | Tvådimensionell, $\mathbb{R}^2$ |
| `(2, 0, 5, 6)` | 4 | Fyrdimensionell, $\mathbb{R}^4$ |
| `(40, 75, 125)` | 3 | Tredimensionell, $\mathbb{R}^3$ |

En tvådimensionell vektor kan ritas som en pil från **origo** `(0, 0)` till punkten `(3, 4)`. Den första komponenten anger hur långt vi går i x-led, den andra hur långt vi går i y-led.

**Kom ihåg:** I ett dataset kan dimensioner representera egenskaper. Det behöver inte handla om fysiska riktningar i ett rum.

### 2.2 Vektorns längd – norm

Om pilen går 3 steg åt höger och 4 steg uppåt kan vi räkna dess längd med **Pythagoras sats**:

$$
\|v\|=\sqrt{3^2+4^2}=\sqrt{9+16}=\sqrt{25}=5
$$

Symbolen `||v||` betyder **vektorns längd**, även kallad **norm**.

För en tvådimensionell vektor `(a, b)`:

$$
\|v\|=\sqrt{a^2+b^2}
$$

Detta är det vanliga euklidiska avståndet från origo till vektorns spets.

### 2.3 Enhetsvektor – samma riktning, längd 1

En **enhetsvektor** har längden 1. Vi får den genom att dividera varje komponent med vektorns längd:

$$
\hat v=\frac{v}{\|v\|}=\left(\frac35,\frac45\right)=(0.6,0.8)
$$

Den pekar åt samma håll som `(3, 4)` men är kortare.

**Varför?** Om vi bara vill jämföra **riktning** och inte storlek kan vi normalisera vektorer till längden 1.

> **Undantag:** Nollvektorn `(0, 0)` har längden 0 och kan inte göras till en enhetsvektor genom division med sin längd.

### 2.4 Samma sak i NumPy

```python
import numpy as np

v = np.array([3, 4])
langd = np.linalg.norm(v)
enhetsvektor = v / langd

print(langd)         # 5.0
print(enhetsvektor)  # [0.6 0.8]
```

- `np.array(...)` skapar en NumPy-array.
- `np.linalg.norm(v)` räknar ut längden.
- `v / langd` dividerar **varje komponent** med samma tal.

`linalg` är NumPys del för **linear algebra**.

---

## 3. Skalning och addition

### 3.1 Skalning – multiplicera med ett tal

Vi utgår från:

$$
v=(3,4)
$$

Om vi multiplicerar med 2 får vi:

$$
2v=(6,8)
$$

Pilen blir **dubbelt så lång**, men pekar åt samma håll. Om vi multiplicerar med `0.5` blir den hälften så lång. Om vi multiplicerar med ett **negativt** tal vänds riktningen.

```python
v = np.array([3, 4])

print(2 * v)    # [6 8]
print(0.5 * v)  # [1.5 2. ]
print(-v)       # [-3 -4]
```

### 3.2 Addition – lägg ihop komponenterna

Joakims exempel:

$$
v=(3,4),\qquad w=(-2,1)
$$

Vi lägger ihop **första med första**, **andra med andra**:

$$
v+w=(3+(-2),4+1)=(1,5)
$$

```python
v = np.array([3, 4])
w = np.array([-2, 1])

print(v + w)  # [1 5]
```

**Varför?** Vi kan kombinera förändringar eller bidrag i flera riktningar. Två vektorer måste ha lika många komponenter för vanlig komponentvis addition.

---

## 4. Skalärprodukt – två vektorer ger ett tal

**Skalärprodukt** och **prickprodukt** är två namn på samma sak. På engelska heter det *dot product*.

För två tvådimensionella vektorer:

$$
a\cdot b=a_1b_1+a_2b_2
$$

Vi multiplicerar komponenterna **parvis** och adderar resultaten.

### 4.1 Joakims tre exempel

Utgå från `a = (3, 2)`.

| Andra vektorn | Beräkning | Skalärprodukt |
|---|---|---:|
| `b₁ = (2, 1.5)` | `3·2 + 2·1.5` | `9` |
| `b₂ = (-1, 1.5)` | `3·(-1) + 2·1.5` | `0` |
| `b₃ = (-2, -1)` | `3·(-2) + 2·(-1)` | `-8` |

När båda vektorerna inte är nollvektorer gäller:

- **Positiv:** vinkeln mellan vektorerna är mindre än 90°.
- **Noll:** vektorerna är vinkelräta.
- **Negativ:** vinkeln är större än 90°.

### 4.2 Viktig precisering

I videon beskrivs ett stort positivt värde som att vektorerna nästan har samma riktning. Det är en användbar första intuition, men **inte alltid korrekt**: skalärprodukten beror också på **vektorernas längder**.

Den matematiska kopplingen är:

$$
a\cdot b=\|a\|\,\|b\|\cos(\theta)
$$

Där `θ` är vinkeln mellan vektorerna. Om vi först gör båda vektorerna till **enhetsvektorer** beskriver deras skalärprodukt direkt cosinus för vinkeln.

### 4.3 NumPy

```python
a = np.array([3, 2])
b = np.array([-1, 1.5])

resultat = np.dot(a, b)
print(resultat)  # 0.0
```

Vi kan också skriva `a @ b`. Operatorn `@` används för matris- och vektorprodukter i NumPy.

---

## 5. Projektion – vektorns skugga på en riktning

Föreställ dig att du lyser på en pil så att den kastar en **vinkelrät skugga** på linjen som går i en annan pils riktning. Skuggan är **projektionen**.

Frågan är: **Hur stor del av vektorn b ligger i a:s riktning?**

Joakims exempel:

$$
a=(3,1),\qquad b=(2,4)
$$

Formeln för projektionen av `b` på en **nollskild** vektor `a` är:

$$
\mathrm{proj}_a(b)=\frac{b\cdot a}{a\cdot a}\,a
$$

Vi tar det i tre steg.

**Steg 1 – skalärprodukten mellan b och a:**

$$
b\cdot a=2\cdot3+4\cdot1=10
$$

**Steg 2 – skalärprodukten mellan a och sig själv:**

$$
a\cdot a=3^2+1^2=10
$$

**Steg 3 – skalfaktorn gånger a:**

$$
\mathrm{proj}_a(b)=\frac{10}{10}(3,1)=(3,1)
$$

**Resultat:** Projektionen av `(2, 4)` på linjen genom `(3, 1)` är `(3, 1)`.

Det betyder **inte** att `b` och `a` är samma vektor. Det betyder att den del av `b` som ligger längs `a` motsvarar just `(3, 1)`. Resten är vinkelrät mot `a`:

$$
b-\mathrm{proj}_a(b)=(2,4)-(3,1)=(-1,3)
$$

Kontroll: `(-1, 3) · (3, 1) = -3 + 3 = 0`, alltså vinkelrätt.

### 5.1 Varför är projektion viktigt?

I regression kan vi inte alltid hitta en modell som träffar varje datapunkt exakt. Vi söker då den **närmaste möjliga approximationen** bland de värden modellen kan skapa.

Projektion ger den geometriska idén bakom detta: välj den tillåtna punkt som ligger **närmast** det vi vill beskriva.

### 5.2 NumPy

```python
a = np.array([3, 1])
b = np.array([2, 4])

skalfaktor = np.dot(b, a) / np.dot(a, a)
projektion = skalfaktor * a

print(skalfaktor)  # 1.0
print(projektion)  # [3. 1.]
```

---

## 6. Matriser – flera tal ordnade i rader och kolumner

En **matris** är ett rektangulärt rutnät av tal:

$$
A=\begin{bmatrix}2&1\\1&1\end{bmatrix}
$$

Matrisen har **2 rader** och **2 kolumner**. Vi säger att dess **form** (*shape*) är `2 × 2`.

En matris kan exempelvis representera en tabell med personer som rader och egenskaper som kolumner.

### 6.1 Multiplicera matris med vektor

Joakim använder:

$$
A=\begin{bmatrix}2&1\\1&1\end{bmatrix},
\qquad
x=\begin{bmatrix}3\\4\end{bmatrix}
$$

Vi vill beräkna:

$$
Ax
$$

**Sätt A – tänk kolumner:**

Matrisens första kolumn är `(2, 1)` och den andra `(1, 1)`.

Talen i `x` säger **hur mycket vi ska använda av varje kolumn**:

$$
Ax=3\begin{bmatrix}2\\1\end{bmatrix}
+4\begin{bmatrix}1\\1\end{bmatrix}
=\begin{bmatrix}6\\3\end{bmatrix}
+\begin{bmatrix}4\\4\end{bmatrix}
=\begin{bmatrix}10\\7\end{bmatrix}
$$

**Sätt B – tänk rader:**

Vi tar skalärprodukten av varje rad i `A` med `x`:

- Första raden: `2·3 + 1·4 = 10`
- Andra raden: `1·3 + 1·4 = 7`

**Samma resultat: `(10, 7)`.**

### 6.2 Dimensionerna måste passa

Om `A` har formen `m × n` och `x` har `n` komponenter, får `Ax` **m** komponenter.

Till exempel:

`(2 × 2) @ (2 × 1) → (2 × 1)`

För två matriser gäller:

`(m × n) @ (n × p) → (m × p)`

**De inre måtten måste stämma.** De yttre anger resultatets form.

### 6.3 NumPy

```python
A = np.array([
    [2, 1],
    [1, 1]
])

x = np.array([3, 4])

y = A @ x

print(A.shape)  # (2, 2)
print(x.shape)  # (2,)
print(y)        # [10  7]
```

**Viktig detalj:** `x.shape` blir `(2,)`, inte `(2, 1)`, eftersom vi skapade en **endimensionell NumPy-array**. Om vi uttryckligen vill ha en kolumnmatris kan vi använda `x.reshape(2, 1)`. Matrisprodukten `A @ x` fungerar ändå.

---

## 7. Från matriser till linjär regression

Nu knyter vi ihop lektionen.

Anta att vi har flera personer i Health Study. Vi vill undersöka om **ålder** kan användas för att förutsäga **systoliskt blodtryck**.

En enkel linjär regressionsmodell skrivs:

$$
\hat y=\beta_0+\beta_1x
$$

Här betyder:

| Symbol | Betydelse |
|---|---|
| `x` | Personens ålder |
| `ŷ` | Modellens förutsagda blodtryck |
| `β₀` | Intercept: linjens värde när ålder = 0 |
| `β₁` | Lutning: hur mycket förutsägelsen ändras per extra år |

**Varför hatt på y?** `ŷ` betyder **förutsagt värde**. `y` utan hatt används för det **observerade värdet**.

Om vi har flera personer kan alla deras åldrar och en kolumn med ettor samlas i en matris:

$$
X=
\begin{bmatrix}
1&20\\
1&30\\
1&40
\end{bmatrix},
\qquad
\beta=
\begin{bmatrix}
\beta_0\\
\beta_1
\end{bmatrix}
$$

Kolumnen med ettor behövs för att modellen ska kunna ha ett **intercept**.

Alla förutsägelser kan nu skrivas kompakt:

$$
\hat y=X\beta
$$

Det är samma sorts **matris gånger vektor** som vi nyss räknade på!

### 7.1 Varför kan vi inte bara lösa ekvationen exakt?

Riktiga data är sällan perfekta. Två personer i samma ålder kan ha olika blodtryck. Därför kan en enda rak linje normalt inte passera genom **alla** datapunkter.

Vi behöver hitta den linje som ligger **så nära observationerna som möjligt**.

### 7.2 Residualer – modellens fel för varje observation

En **residual** är:

$$
e_i=y_i-\hat y_i
$$

Exempel:

- Observerat blodtryck: `135 mmHg`
- Förutsagt blodtryck: `130 mmHg`
- Residual: `135 − 130 = 5 mmHg`

Ett negativt residual betyder att modellen förutsade ett **högre** värde än det observerade.

### 7.3 Minsta kvadratmetoden

Vi kan inte bara summera residualerna eftersom positiva och negativa fel kan ta ut varandra. Därför **kvadrerar** vi varje residual och summerar:

$$
\mathrm{SSE}=\sum_{i=1}^{n}(y_i-\hat y_i)^2
$$

**Minsta kvadratmetoden** (*ordinary least squares*, OLS) väljer koefficienterna så att denna summa blir så liten som möjligt.

Geometriskt är detta en **ortogonal projektion** av observationsvektorn `y` på det rum som kan bildas av kolumnerna i `X`.

Det är just kopplingen som Joakim förbereder i sin genomgång:

**Projektion → närmaste möjliga värden → minsta kvadrat → regression.**

> **Fördjupning:** I praktiken löser bibliotek ofta minsta-kvadratproblemet med numeriskt stabila metoder. Vi behöver inte själva beräkna en matrisinvers för att använda regression.

---

## 8. Ett första Pythonexempel – endast för att se kopplingen

Det här är ett **pedagogiskt tillägg**, inte kod som Joakim skriver i den här videon.

```python
import numpy as np

# Påhittade exempeldata: ålder och systoliskt blodtryck
alder = np.array([20, 30, 40, 50, 60])
blodtryck = np.array([110, 115, 125, 130, 140])

# En kolumn med ettor och en med ålder
X = np.column_stack([np.ones(len(alder)), alder])

# Hitta intercept och lutning med minsta kvadrat
beta, _, _, _ = np.linalg.lstsq(X, blodtryck, rcond=None)

# Beräkna modellens förutsägelser
forutsagt = X @ beta

print("Intercept:", beta[0])
print("Lutning:", beta[1])
print("Förutsägelser:", forutsagt)
```

### Vad gör koden – rad för rad?

1. `alder` och `blodtryck` innehåller våra **påhittade observationer**.
2. `np.ones(len(alder))` skapar en etta för varje person.
3. `np.column_stack(...)` bygger en **matris** med kolumnerna ettor och ålder.
4. `np.linalg.lstsq(...)` söker koefficienter med **minsta kvadratmetoden**.
5. `beta` innehåller intercept och lutning.
6. `X @ beta` ger modellens förutsägelser genom **matrismultiplikation**.

Du behöver **inte memorera `lstsq` nu**. Det viktiga är att känna igen att `X @ beta` är samma idé som Joakims matrisexempel.

I nästa lektion kan vi använda **scikit-learn** för att arbeta med regression på ett mer praktiskt sätt.

---

## 9. Begreppslista

| Begrepp | Enkel förklaring | Engelska |
|---|---|---|
| Vektor | Ordnad samling tal | Vector |
| Komponent | Ett av talen i en vektor | Component |
| Dimension | Antal komponenter | Dimension |
| Norm | Vektorns längd | Norm |
| Enhetsvektor | Vektor med längden 1 | Unit vector |
| Skalning | Multiplicera varje komponent med ett tal | Scaling |
| Skalärprodukt | Multiplicera parvis och summera | Dot product |
| Projektion | Del av en vektor i en viss riktning | Projection |
| Matris | Rutnät av tal | Matrix |
| Form | Antal rader och kolumner | Shape |
| Koefficient | Tal som anger ett bidrag i modellen | Coefficient |
| Residual | Observerat minus förutsagt värde | Residual |
| Minsta kvadrat | Välj modellen med minsta summa kvadrerade fel | Least squares |
| Regression | Modellera samband och förutsäga numeriska värden | Regression |

## 10. Vanliga missförstånd

**”En vektor måste vara en pil.”**  
Nej. Pilen är ett sätt att **visualisera** en vektor med två eller tre komponenter. I dataanalys använder vi också vektorer med hundratals eller tusentals komponenter.

**”Ett stort positivt skalärproduktvärde betyder alltid nästan samma riktning.”**  
Nej. Vektorernas **längder** påverkar också värdet.

**”En projektion gör två vektorer identiska.”**  
Nej. Projektionen är bara den del av en vektor som ligger i en viss riktning.

**”Matrisprodukt betyder att multiplicera varje tal på samma position.”**  
Nej. Matrisprodukten `@` kombinerar rader och kolumner med **skalärprodukter**. Elementvis multiplikation med `*` är en annan operation i NumPy.

**”Regression ska träffa alla punkter exakt.”**  
Nej. Regression söker oftast en **bra approximation**, eftersom verkliga observationer varierar.

**”Samband betyder orsak.”**  
Nej. Att ålder och blodtryck samvarierar i ett dataset bevisar inte att den ena variabeln ensam orsakar den andra.

---

## 11. Sammanfattning – det viktigaste att ta med sig

1. En **vektor** är flera tal i bestämd ordning.
2. Vi kan räkna med vektorer genom **skalning, addition och skalärprodukt**.
3. En **projektion** visar hur mycket av en vektor som ligger längs en viss riktning.
4. En **matris** samlar flera tal i rader och kolumner.
5. **Matris gånger vektor** är ett sätt att kombinera matrisens kolumner med olika vikter.
6. Vid **linjär regression** används koefficienter som vikter för att skapa förutsägelser.
7. **Minsta kvadratmetoden** väljer vikterna så att summan av de kvadrerade residualerna blir så liten som möjligt.

### Koppling till Health Study del 2

När du senare skriver en regressionsmodell med `scikit-learn` behöver du framför allt förstå:

- **X:** de förklarande variablerna, till exempel ålder.
- **y:** det verkliga blodtrycket.
- **Koefficienter:** hur modellen väger variablerna.
- **ŷ:** modellens förutsagda blodtryck.
- **Residualer:** skillnaderna mellan verkliga och förutsagda värden.

Du har då den matematiska grunden för att förstå **vad biblioteket gör åt dig**.

---

## Källor och vidare läsning

**Primärkälla**

- Joakims kurslektion *Linjär algebra*, tillhörande lektionstext och transkribering: [YouTube](https://www.youtube.com/watch?v=joWRgVkJ8Ec).

**Kompletterande referenser för terminologi och Pythonmetoder**

- [NumPy – `numpy.linalg.norm`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.norm.html)
- [NumPy – `numpy.dot`](https://numpy.org/doc/stable/reference/generated/numpy.dot.html)
- [NumPy – `numpy.linalg.lstsq`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.lstsq.html)
- [scikit-learn – LinearRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html)

*Notering: Exempel och förklaringar är bearbetade för handboken. Referenserna ovan är avsedda för vidare kontroll och fördjupning; de är inte ordagranna källor till lektionstexten.*
