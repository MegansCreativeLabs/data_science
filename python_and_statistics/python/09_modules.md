# Lektion 09 – Moduler och paket i Python

> **Vecka 2 · Python · Moduler**  
> Bygger på lärarens genomgång om moduler, egna `.py`-filer, import och projektstruktur.

## Quick Reference

| Syntax | Vad den gör |
|---|---|
| `import math` | Importerar en hel modul |
| `math.sqrt(16)` | Använder en funktion från modulen (`4.0`) |
| `import math_utils as mu` | Importerar en egen modul med kortnamn |
| `mu.add(3, 7)` | Anropar funktionen `add` i modulen |
| `from math_utils import multiply` | Importerar bara ett namn från modulen |
| `from src import math_utils` | Importerar en modul från mappen `src` om projektets rot är på Pythons sökväg |
| `if __name__ == "__main__":` | Kör testkod endast när filen startas direkt |
| `__init__.py` | Markerar vanligtvis en mapp som ett vanligt Python-paket |
| `cd src` / `cd ..` | Gå in i en mapp / upp en nivå i terminalen |

## 1. Vad är en modul?

En **modul** är i grunden en Python-fil (`.py`) som innehåller kod du kan använda från en annan fil. Det kan vara funktioner, variabler eller klasser.

Du har redan använt moduler när du skrivit:

```python
import math
import statistics
import numpy as np

print(math.sqrt(16))           # 4.0
print(statistics.mean([1, 2, 3, 4, 5]))  # 3
print(np.array([1, 2, 3]) * 2)           # [2 4 6]
```

**Varför?** Du slipper skriva allt själv och kan dela upp stora program i mindre delar.

**Viktig skillnad:** En vanlig Python-lista som multipliceras med två upprepas, medan en NumPy-array multiplicerar sina element:

```python
print([1, 2, 3] * 2)       # [1, 2, 3, 1, 2, 3]
print(np.array([1, 2, 3]) * 2)  # [2 4 6]
```

`math` och `statistics` följer med Python. `numpy` behöver normalt installeras separat i projektets miljö.

## 2. Skapa en egen modul

Skapa filen **`math_utils.py`**:

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b
```

Här **definierar** du funktionerna, men du kör dem inte ännu. Från en annan fil i samma mapp kan du skriva:

```python
import math_utils

result = math_utils.add(5, 10)
print(result)  # 15
```

Läs `math_utils.add(5, 10)` som: *använd funktionen `add` som finns i filen `math_utils.py`.*

## 3. Tre sätt att importera

```python
# 1. Hela modulen
import math_utils
print(math_utils.add(3, 7))

# 2. Hela modulen med ett kortare namn (alias)
import math_utils as mu
print(mu.multiply(2, 10))

# 3. En särskild funktion
from math_utils import multiply
print(multiply(2, 50))
```

**Varför fungerar sista exemplet utan `math_utils.`?** Där har namnet `multiply` importerats direkt till filen där du arbetar.

Om du bara skriver `from math_utils import multiply` blir inte `add()` automatiskt tillgänglig som ett eget namn.

## 4. Varför körs min testkod när jag importerar?

Python kör normalt kod på **toppnivå** i en modul första gången den importeras i en Python-process. Därför är detta opraktiskt i en återanvändbar modul:

```python
def add(a, b):
    return a + b

print(add(3, 7))  # Körs även vid första importen!
```

Använd istället:

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


if __name__ == "__main__":
    print(add(3, 7))        # 10
    print(multiply(2, 10))  # 20
```

### Vad betyder `__name__` och `__main__`?

- `__name__` är en särskild variabel som Python sätter automatiskt.
- När du kör `python math_utils.py` direkt får den värdet `"__main__"`.
- När du importerar `math_utils` får den normalt värdet `"math_utils"`.
- Därför körs testutskrifterna bara när du startar filen direkt.

Detta är särskilt användbart för **utility-funktioner** som du vill kunna testa separat och sedan återanvända i en notebook.

## 5. Organisera projektet i mappar

Läraren visar hur funktioner kan flyttas ut ur en notebook. En enkel struktur är:

```text
my_project/
├── notebooks/
│   └── analysis.ipynb
├── src/
│   ├── __init__.py
│   ├── math_utils.py
│   └── greetings.py
└── data/
    └── example.csv
```

- **`notebooks/`**: analyser, grafer och förklaringar.
- **`src/`**: återanvändbar Python-kod.
- **`data/`**: datafiler som CSV.

Från en fil eller notebook där projektroten finns på Pythons importsökväg:

```python
from src import math_utils

print(math_utils.add(4, 5))  # 9
```

**Observera:** Importer beror på var Python letar efter moduler, inte enbart på var filerna syns i VS Code. En notebook som körs med en annan arbetskatalog kan därför få `ModuleNotFoundError`.

## 6. Vad är ett paket?

En **modul** är en Python-fil. Ett **paket** samlar moduler under ett gemensamt namn, ofta i en mapp.

En fil som heter `src/__init__.py` gör mappen till ett *vanligt paket*. Filen får vara tom. Moderna Python-versioner kan även ha paket utan den filen (så kallade namespace packages), men lärarens metod är ett tydligt nybörjarexempel.

Skapa även `src/greetings.py`:

```python
def greet(name):
    return f"Hello {name}"
```

Då kan du använda båda modulerna:

```python
from src import math_utils, greetings

print(math_utils.multiply(3, 4))  # 12
print(greetings.greet("Joakim"))  # Hello Joakim
```

## 7. Om Python inte hittar modulen

Ett vanligt fel är:

```text
ModuleNotFoundError: No module named 'src'
```

Det betyder att Python inte hittar `src` på sin importsökväg i den aktuella körningen. Kontrollera först **var projektroten är** och **vilken arbetskatalog notebooken använder**.

I lärarens video visas en lösning med `sys.path`, där man tillfälligt lägger till en mapp där Python ska leta:

```python
import sys
from pathlib import Path

# Exempel: notebooken körs från mappen notebooks/
project_root = Path.cwd().parent
sys.path.append(str(project_root))

from src import math_utils
print(math_utils.add(4, 5))
```

**Varför `parent`?** Om arbetskatalogen är `my_project/notebooks`, är föräldramappen `my_project`.

**Viktigt:** Exemplet fungerar bara om `Path.cwd()` faktiskt är `notebooks/`. Kontrollera med `print(Path.cwd())`. Att ändra `sys.path` kan hjälpa i en notebook, men en stabil projektstruktur och korrekt körmiljö är bättre på sikt.

## 8. Terminalkommandon från videon

```bash
python math_utils.py   # Kör filen från rätt mapp
cd src                 # Gå in i mappen src
cd ..                  # Gå upp en mappnivå
```

Om du befinner dig i projektroten och filen ligger i `src` kan du också köra:

```bash
python src/math_utils.py
```

## Vanliga misstag

| Misstag | Varför det blir fel | Lösning |
|---|---|---|
| `add(2, 3)` efter bara `import math_utils` | `add` är inte importerat direkt | Skriv `math_utils.add(2, 3)` |
| Testutskrifter syns vid import | Testkoden ligger på toppnivå | Lägg den under `if __name__ == "__main__":` |
| `ModuleNotFoundError` | Python hittar inte projektmappen | Kontrollera arbetskatalog och importsökväg |
| `import math_utils` efter att filen flyttats till `src/` | Importvägen har ändrats | Använd exempelvis `from src import math_utils` |
| En gammal version av modulen används i Jupyter | Importerade moduler kan ligga kvar i minnet | Starta om kärnan eller använd lämplig omladdning |

## Övningar

**1.** Skapa `calculator.py` med funktionen `subtract(a, b)` som returnerar skillnaden. Importera och använd den från en annan fil.

**2.** Lägg till testkod under `if __name__ == "__main__":`. Kontrollera att den syns när du kör `calculator.py` direkt men inte vid import.

**3.** Skapa en mapp `src` med `__init__.py` och flytta in `calculator.py`. Prova att importera den från projektroten.

### Facit

**1. `calculator.py`:**

```python
def subtract(a, b):
    return a - b
```

**`main.py`:**

```python
import calculator
print(calculator.subtract(10, 4))  # 6
```

**2. Lägg längst ner i `calculator.py`:**

```python
if __name__ == "__main__":
    print(subtract(10, 4))
```

**3. Från projektroten:**

```python
from src import calculator
print(calculator.subtract(10, 4))  # 6
```

---

**Sammanfattning:** Moduler gör kod återanvändbar. `import` låter dig använda kod från andra filer. `if __name__ == "__main__":` skiljer testkörning från import. Paket hjälper dig att organisera flera moduler. Om importen misslyckas, börja med att undersöka var Python letar efter filer.
