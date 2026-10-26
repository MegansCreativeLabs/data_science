# 05 – Reproducibility and requirements.txt

> **Git & Agila Arbetssätt · Lektion 05 · Vecka 2**

## Quick Reference

| Begrepp / kommando | Betydelse |
|---|---|
| Reproducerbarhet | Andra ska kunna återskapa projektets miljö och köra samma kod. |
| Virtuell miljö (`.venv`) | Isolerar projektets installerade Python-paket. |
| `python -m venv .venv` | Skapar en virtuell miljö. |
| `python -m pip install numpy pandas jupyter` | Installerar paket i vald Python-miljö. |
| `python -m pip freeze > requirements.txt` | Sparar installerade paket och versioner i en textfil. |
| `python -m pip install -r requirements.txt` | Installerar paketen från listan. |
| `.gitignore` | Talar om vilka filer och mappar Git inte ska versionshantera. |
| `git pull` | Hämtar och integrerar ändringar från fjärrlagret. |
| `git add`, `git commit`, `git push` | Förbereder, sparar lokalt och skickar ändringar till GitHub. |

## 1. Vad betyder reproducerbarhet?

Tänk att din kod fungerar på din dator, men inte hos en gruppmedlem. Det kan bero på att ni har olika paket installerade, olika paketversioner eller olika Python-versioner.

**Reproducerbarhet** betyder att en annan person ska kunna återskapa förutsättningarna för projektet och få samma resultat när koden körs med samma data. I videon ligger fokus på **Python-miljön och `requirements.txt`**.

### Exempel

Du skriver:

```python
import numpy as np
import pandas as pd

numbers = np.array([1, 2, 3, 4])
print(numbers * 3)
```

Resultat:

```text
[ 3  6  9 12]
```

Om `numpy` inte är installerat i den Python-miljö som kör koden får du exempelvis `ModuleNotFoundError`.

## 2. Vad är en virtuell miljö?

En **virtuell miljö** är en separat uppsättning installerade paket för ett visst projekt. Då behöver inte olika projekt använda samma paketversioner.

Skapa miljön i projektmappen:

```powershell
python -m venv .venv
```

Aktivera i **PowerShell**:

```powershell
.\.venv\Scripts\Activate.ps1
```

I **Git Bash**:

```bash
source .venv/Scripts/activate
```

I VS Code kan du också välja **Python: Create Environment** och sedan välja rätt interpreter/kernel för notebooken.

**Viktigt:** En virtuell miljö ligger normalt *inte* på GitHub. Varje deltagare skapar sin egen.

## 3. Installera paket

När rätt miljö är aktiv:

```bash
python -m pip install numpy pandas jupyter
```

`pip` är verktyget som installerar Python-paket. Formen `python -m pip` hjälper dig att använda `pip` som hör till den Python-interpreter du valt.

I en notebook behöver du dessutom välja rätt **kernel**. En kernel är Python-miljön som faktiskt kör notebookens celler. Det räcker inte att installera paket i en terminal om notebooken kör en annan miljö.

## 4. Vad är requirements.txt?

`requirements.txt` är en vanlig textfil som listar paket som projektet behöver. Den kan innehålla versionsnummer:

```text
numpy==2.1.3
pandas==2.2.3
jupyter==1.1.1
```

*Versionerna ovan är bara exempel, inte rekommenderade versioner för ditt projekt.*

Tecknen `==` betyder att just den versionen ska installeras.

Skapa filen från den aktiva miljön:

```bash
python -m pip freeze > requirements.txt
```

- `pip freeze` listar installerade paket och deras versioner.
- `>` skriver resultatet till filen och **ersätter** tidigare innehåll.
- Därför är det viktigt att rätt virtuella miljö är aktiv.

**Observera:** `pip freeze` kan även lista paket som installerats indirekt som beroenden. Filen beskriver installerade paket, inte automatiskt vilka paket din egen kod faktiskt använder.

## 5. Så återskapar en gruppmedlem miljön

Efter att ha klonat projektet från GitHub:

1. Öppna projektmappen i VS Code.
2. Skapa en ny `.venv` med lämplig Python-version.
3. Aktivera miljön och välj den som interpreter/kernel.
4. Installera projektets paket:

```bash
python -m pip install -r requirements.txt
```

5. Kör projektet och kontrollera att det fungerar.

Flaggan **`-r`** betyder att `pip` ska läsa paketlistan från en fil.

## 6. Vad ska sparas på GitHub?

En vanlig struktur är:

```text
my_project/
├── .gitignore
├── requirements.txt
├── reproducibility.ipynb
└── .venv/                # Finns lokalt, ska ignoreras av Git
```

Lägg exempelvis detta i `.gitignore`:

```gitignore
.venv/
__pycache__/
.ipynb_checkpoints/
```

**Spara `requirements.txt`, men inte `.venv/`.** Den första är instruktionen för att installera paketen; den andra är din egen lokala installation.

## 7. När ska requirements.txt uppdateras?

I lärarens exempel installeras ett nytt paket, `matplotlib`:

```bash
python -m pip install matplotlib
python -m pip freeze > requirements.txt
```

Sedan sparas ändringen i Git:

```bash
git add requirements.txt
git commit -m "Update requirements with matplotlib"
git push
```

En gruppmedlem kan sedan göra:

```bash
git pull
python -m pip install -r requirements.txt
```

På så sätt kan alla få med det nya paketet.

## 8. Definition of Done

Läraren kopplar reproducerbarhet till **Definition of Done**, alltså teamets gemensamma krav för att en uppgift ska räknas som klar.

Ett exempel på krav är:

> Om en ändring kräver nya Python-paket ska `requirements.txt` vara uppdaterad, och projektet ska kunna startas från en ny miljö.

Det minskar risken att kod fungerar enbart hos den som skrev den.

## 9. Vanliga misstag

| Problem | Varför det händer | Vad du kan kontrollera |
|---|---|---|
| `ModuleNotFoundError` | Paketet saknas i miljön som kör koden. | Kontrollera vald interpreter/kernel och installera paketet där. |
| Paketet är installerat men notebooken hittar det inte. | Terminalen och notebooken använder olika miljöer. | Välj rätt kernel i VS Code. |
| `.venv` syns i Git. | `.gitignore` saknas eller miljön spårades redan tidigare. | Lägg till `.venv/` i `.gitignore`; redan spårade filer måste tas bort ur Git-index separat. |
| En gruppmedlem får andra paketversioner. | `requirements.txt` är inaktuell eller har inte installerats. | Uppdatera filen och kör `pip install -r requirements.txt`. |
| `pip freeze` listar oväntade paket. | Kommandot kördes i fel miljö eller listar indirekta beroenden. | Kontrollera att projektets `.venv` är aktiv. |
| Installation misslyckas trots samma paketlista. | Exempelvis Python-version eller operativsystem skiljer sig. | Dokumentera också lämplig Python-version. |

## 10. Vad räcker inte requirements.txt till?

Det är ett viktigt första steg, men **det garanterar inte ensamt identiska resultat på alla datorer**. Python-version, operativsystem, indata och slumpmässighet kan också spela roll.

För ett nybörjarprojekt räcker det ofta långt att dokumentera Python-version, använda `.venv`, uppdatera `requirements.txt` och dela samma indata.

## 11. Övningar

**Övning 1.** Vad är skillnaden mellan `.venv` och `requirements.txt`?

**Övning 2.** Vad gör flaggan `-r` i `pip install -r requirements.txt`?

**Övning 3.** Du installerar `matplotlib` i projektet. Vilka steg behöver du göra för att gruppen ska få samma paketlista?

**Övning 4.** Du får `ModuleNotFoundError` trots att du nyss installerat paketet. Vad är den första saken att kontrollera i VS Code?

**Övning 5.** Varför ska `.venv/` normalt stå i `.gitignore`?

### Facit

1. `.venv` är den lokala miljön med installerade paket. `requirements.txt` är listan som gör det möjligt att installera motsvarande paket i en annan miljö.
2. Den säger åt `pip` att läsa paketen från den angivna filen.
3. Installera paketet i rätt `.venv`, uppdatera `requirements.txt`, gör commit och push. Gruppmedlemmarna gör pull och installerar från filen.
4. Att notebooken använder samma interpreter/kernel som den miljö där paketet installerades.
5. Miljön innehåller lokala installationsfiler som andra ska återskapa på sin egen dator, inte kopiera via Git.

---

*Baserad på lärarens genomgång om reproducerbarhet och `requirements.txt`. Extra förtydliganden om Windows, kernel och begränsningar har lagts till för att göra stegen lättare att förstå.*
