# Lektion 02 – Git och GitHub: från lokalt projekt till samarbete

**Kursvecka:** 2  
**Ämne:** Git och GitHub  
**Underlag:** Lärarens genomgång om installation, lokala commits, GitHub, kloning och synkronisering.

## Quick Reference

| Kommando | Vad det gör |
|---|---|
| `git --version` | Visar installerad Git-version. |
| `git init` | Börjar versionshantera den aktuella mappen. |
| `git status` | Visar ändrade, nya och staged filer. |
| `git add filnamn` | Väljer en fil till nästa commit. |
| `git add .` | Väljer ändringar i den aktuella mappen och undermapparna. |
| `git commit -m "Beskriv ändringen"` | Sparar en lokal version av de staged ändringarna. |
| `git remote add origin URL` | Kopplar projektet till ett fjärrrepository. |
| `git branch -M main` | Byter namn på aktuell branch till `main`. |
| `git push -u origin main` | Skickar upp branchen och anger dess upstream. |
| `git push` | Skickar upp nya lokala commits till kopplad branch. |
| `git clone URL` | Hämtar ett repository till datorn. |
| `git pull` | Hämtar och integrerar ändringar från fjärrrepositoryt. |

**Minnesregel:** `add` = välj ändringar → `commit` = spara lokalt → `push` = skicka till GitHub. `pull` = hämta andras ändringar.

## 1. Vad är Git och GitHub?

**Git** är ett versionshanteringsverktyg på din dator. Det håller reda på vilka ändringar som sparats och gör det möjligt att gå tillbaka till tidigare versioner.

**GitHub** är en tjänst där Git-repositories kan lagras på nätet och delas med andra. Git fungerar även utan GitHub.

Ett **repository (repo)** är ett projekt som Git versionshanterar. En **commit** är en sparad version av de ändringar som valts. En **branch** är en utvecklingsgren, till exempel `main`.

## 2. Fyra platser i arbetsflödet

1. **Working directory:** Filerna du redigerar i VS Code.
2. **Staging area:** Ändringarna du valt med `git add`.
3. **Local repository:** Commits som sparats på datorn med `git commit`.
4. **Remote repository:** Exempelvis GitHub, dit commits skickas med `git push`.

```text
Filer i VS Code --git add--> Staging --git commit--> Lokal historik
                                                        |
                                                     git push
                                                        v
                                                      GitHub
```

Att spara med **Ctrl+S** sparar filen på datorn, men skapar ingen Git-commit. Att göra `git commit` sparar en version lokalt, men laddar inte upp den till GitHub.

## 3. Installera och kontrollera Git

I videon installerar läraren Git för Windows och använder bland annat **Git Bash** och **VS Code**. Efter installationen kan du kontrollera att Git fungerar:

```bash
git --version
```

Om Git ber dig ange identitet inför första committen:

```bash
git config --global user.name "Ditt namn"
git config --global user.email "din-epost@example.com"
```

Detta är metadata som kopplas till commits. Välj den e-postadress du vill använda för dina Git-commits.

## 4. Börja versionshantera ett nytt lokalt projekt

Öppna projektmappen i VS Code och öppna terminalen där.

```bash
git init
git status
git add .
git commit -m "Initial commit"
```

**Varför varje rad?**

- `git init` skapar Git-repositoryt och dess dolda `.git`-mapp.
- `git status` visar vad Git ser; det är bra att kontrollera före varje commit.
- `git add .` väljer de ändringar som ska ingå i nästa commit.
- `git commit -m ...` sparar en namngiven punkt i historiken.

**Viktigt:** Kör inte `git init` i en undermapp som redan tillhör ett annat Git-repository om du inte avsiktligt vill skapa ett separat repo.

## 5. Vad betyder symbolerna i VS Code?

| Symbol | Betydelse |
|---|---|
| `U` | **Untracked:** Git följer inte den nya filen ännu. |
| `M` | **Modified:** En redan spårad fil har ändrats. |
| `A` | **Added:** Filen är tillagd till staging area. |

VS Code har också en **Source Control**-vy där du kan stage:a och committa via knappar. Läraren rekommenderar att även lära sig terminalkommandona, eftersom de fungerar i många olika miljöer.

## 6. Använd `.gitignore`

En `.gitignore` talar om vilka **ospårade** filer Git normalt ska ignorera.

Exempel för ett Pythonprojekt:

```gitignore
.venv/
__pycache__/
*.pyc
```

Det betyder att den virtuella miljön, Python-cachemappar och kompilerade Pythonfiler inte ska läggas till automatiskt.

I videon visar läraren också mönster som `*.csv` och `*.txt`. **Använd inte dessa slentrianmässigt:** då kan du råka utesluta exempeldata eller textfiler som faktiskt ska finnas i projektet.

**Vanligt missförstånd:** `.gitignore` tar inte automatiskt bort filer som redan spåras av Git.

## 7. Koppla ett lokalt projekt till GitHub

Skapa ett **tomt** repository på GitHub. När du redan har ett lokalt repo med commits kan du koppla ihop dem:

```bash
git branch -M main
git remote add origin https://github.com/ANVANDARE/REPO.git
git push -u origin main
```

- `main` är namnet på branchen.
- `origin` är ett vanligt namn på GitHub-adressen, inte ett särskilt GitHub-kommando.
- `-u` anger vilken fjärrbranch den lokala branchen ska följa. Efteråt räcker ofta `git push`.

Adressen ovan är en **platshållare**: byt den till adressen GitHub visar för just ditt repo.

**Public eller private?** Ett publikt repo kan läsas av andra. Ett privat repo är begränsat till personer som har tillgång. Lägg aldrig in lösenord, tokens eller andra hemligheter i Git.

## 8. Ditt vanliga arbetsflöde

När repositoryt redan är kopplat till GitHub:

```bash
git pull
git status
git add .
git commit -m "Add lesson notes"
git push
```

**Varför `git pull` först?** Om du eller någon annan ändrat samma repository från en annan dator kan du behöva hämta de ändringarna innan du skickar upp dina egna. Om samma rader ändrats på två ställen kan du behöva lösa en konflikt.

Använd gärna `git status` före `git add .` så att du inte råkar inkludera filer du inte tänkt publicera.

## 9. Klona ett befintligt repository

När projektet redan finns på GitHub och du vill få det till en annan dator:

```bash
git clone https://github.com/ANVANDARE/REPO.git
```

Git skapar normalt en ny mapp med projektets namn. Vill du själv välja mappnamn:

```bash
git clone https://github.com/ANVANDARE/REPO.git mitt_projekt
```

Vill du klona direkt till den aktuella mappen (som behöver vara tom):

```bash
git clone https://github.com/ANVANDARE/REPO.git .
```

**Skillnaden mellan `clone` och `pull`:**

- `clone`: första gången du hämtar hela repositoryt.
- `pull`: när du redan har repositoryt och vill hämta nya ändringar.

En nedladdad ZIP-fil är inte samma sak som ett klonat Git-repository; den innehåller normalt inte Git-historiken och kopplingen till GitHub.

## 10. README och GitHub-editorn

En `README.md` är projektets introduktionssida på GitHub. Den skrivs vanligtvis i **Markdown**.

Läraren visar också att du kan trycka på **punkttangenten (`.`)** när du tittar på ett repository på GitHub för att öppna en VS Code-liknande webbredigerare.

## 11. Vanliga misstag

| Problem | Varför det händer | Vad du kontrollerar |
|---|---|---|
| Ändringen syns inte på GitHub | Du har bara sparat filen eller gjort en lokal commit. | Har du kört `git push`? |
| `nothing to commit` | Inga nya ändringar är staged, eller allt är redan committat. | Kör `git status`. |
| En fil kommer inte med | Den kan vara ignorerad eller inte staged. | Kontrollera `.gitignore` och `git status`. |
| `git push` avvisas | GitHub kan innehålla ändringar som du inte har lokalt. | Kontrollera status och hämta ändringar på ett säkert sätt. |
| Git hittar inte projektet | Terminalen står i fel mapp. | Kontrollera aktuell katalog och öppna rätt projektmapp. |

## 12. Övningar

**Övning 1:** Vad är skillnaden mellan `git add`, `git commit` och `git push`?

**Övning 2:** Du har redigerat en fil och tryckt Ctrl+S. Är ändringen nu sparad som en Git-version?

**Övning 3:** Du har redan projektet på två datorer. Du pushar en ändring från den ena. Vilket kommando använder du på den andra för att hämta ändringen?

**Övning 4:** Vad betyder `U` och `M` bredvid filer i VS Code?

**Övning 5:** Varför brukar `.venv/` stå i `.gitignore`?

### Facit

1. `add` väljer ändringar, `commit` sparar dem i lokal historik, `push` skickar commits till fjärrrepositoryt.
2. Nej. Ctrl+S sparar filen, men du behöver stage:a och committa för att skapa en Git-version.
3. `git pull`.
4. `U` betyder untracked och `M` betyder modified.
5. En virtuell miljö innehåller lokalt installerade beroenden och bör normalt återskapas i stället för att versionshanteras.

---

**Sammanfattning:** Git sparar projektets historik lokalt. GitHub gör det möjligt att dela den. Lärarens praktiska genomgång visar hur du går från installation och `git init` till `.gitignore`, första pushen, kloning och `git pull` på en annan dator. Nästa naturliga steg är branches och samarbete.
