# 01 – Git och GitHub: grunderna

**Kursvecka:** 2 · **Ämne:** Git och GitHub · **Nivå:** Nybörjare

> **Målet:** Förstå *var* ändringar finns, *varför* vi använder Git och *vad* kommandona faktiskt gör.

## Quick Reference – vanligaste kommandona

| Kommando | Vad händer? | När använder jag det? |
|---|---|---|
| `git status` | Visar vilka filer som ändrats och vad som är staged | Före och efter andra Git-kommandon |
| `git clone URL` | Kopierar ett befintligt fjärrrepository till datorn | När du ska börja arbeta med ett repo som redan finns |
| `git add filnamn.py` | Lägger en viss fils aktuella ändringar i staging area | När du vill välja vad nästa commit ska innehålla |
| `git add .` | Stage:ar ändringar i aktuell mapp och undermappar | När du har granskat och vill ta med allt relevant |
| `git commit -m "Beskriv ändringen"` | Sparar staged ändringar som en lokal version | När en lagom stor ändring är klar |
| `git push` | Skickar lokala commits till fjärrrepositoryt | När du vill dela eller säkerhetskopiera dina commits på GitHub |
| `git pull` | Hämtar och integrerar ändringar från fjärrrepositoryt | När andra har uppdaterat samma branch |
| `git log --oneline` | Visar tidigare commits i kortform | När du vill se historiken |

**Kom ihåg:** `add` väljer ändringar → `commit` sparar dem lokalt → `push` skickar commits till GitHub.

---

## 1. Vad är Git?

**Git är ett versionshanteringssystem.** Det hjälper oss att spara olika versioner av ett projekt. Vi kan se *vad* som ändrats, *när* det ändrades och ofta *vem* som gjorde ändringen.

Tänk dig att du skriver en Python-funktion. Den fungerar på måndagen. På tisdagen ändrar du den och råkar få ett fel. Med Git finns tidigare commits kvar, så du kan undersöka hur koden såg ut innan.

Git är också användbart för samarbete: flera personer kan arbeta i samma projekt och kombinera sina ändringar.

### Git är inte samma sak som GitHub

- **Git** är verktyget för versionshantering. Det fungerar även utan internet.
- **GitHub** är en tjänst där Git-repositories kan lagras och delas online.
- Ett **repository** (ofta kallat *repo*) är ett projekt som Git håller reda på, inklusive dess versionshistorik.

Läraren nämner även att det finns andra tjänster än GitHub.

## 2. De fyra platserna – varför är det så många?

Läraren beskriver fyra platser i Git-arbetsflödet:

| Plats | Enkelt förklarat | Exempel |
|---|---|---|
| **Working directory** | Filerna du redigerar just nu | Du ändrar `main.py` i VS Code |
| **Staging area** | En förberedelseyta där du väljer vad nästa commit ska innehålla | Du kör `git add main.py` |
| **Local repository** | Historiken med commits på din dator | Du kör `git commit -m "Fix calculation"` |
| **Remote repository** | En kopia av repot på en server, t.ex. GitHub | Du kör `git push` |

Det är **inte fyra olika mappar som du själv behöver skapa**. Det är fyra delar av hur Git hanterar dina filer och din historik.

### Arbetsflödet visuellt

```text
WORKING DIRECTORY       STAGING AREA       LOCAL REPOSITORY       GITHUB
  Redigera filer  --add-->  Välj ändringar --commit--> Spara version --push--> Dela

  Din dator                    Din dator             Din dator           Internet
```

**Viktig detalj:** `git add` flyttar inte bort filen från din arbetsmapp. Det sparar ett urval av ändringarna till nästa commit. Du kan fortfarande redigera filen efteråt.

## 3. Git clone – hämta ett befintligt projekt

Om projektet redan finns på GitHub kan du skapa en lokal kopia:

```bash
git clone https://github.com/EXEMPEL/projekt.git
```

Detta hämtar repositoryt och dess historik till datorn. Git checkar normalt också ut en branch så att du får arbetsfiler att redigera.

Du använder vanligtvis `clone` **en gång per lokal kopia**. När repot redan finns på datorn använder du oftast `pull` för att hämta nya ändringar.

## 4. Git add – välj ändringarna

Anta att du har redigerat `calculator.py`.

```bash
git status
git add calculator.py
git status
```

- Första `git status`: visar att filen har ändrats.
- `git add calculator.py`: väljer ändringen för nästa commit.
- Andra `git status`: visar att ändringen ligger i staging area.

Vill du stage:a alla relevanta ändringar i aktuell mapp och dess undermappar:

```bash
git add .
```

**Var försiktig med `git add .`:** kontrollera `git status` först så att du inte råkar ta med filer som inte ska delas, exempelvis lösenord, hemliga nycklar eller en virtuell miljö. En `.gitignore` hjälper Git att ignorera vissa filer.

## 5. Git commit – spara en version på datorn

När du har stage:at det du vill spara:

```bash
git commit -m "Add calculator function"
```

En **commit** är en sparad version av de staged ändringarna i ditt **lokala** repository.

- `git commit` = skapa versionen.
- `-m` = ange ett meddelande direkt i kommandot.
- Texten inom citattecken = en kort beskrivning av ändringen.

**En commit är inte samma sak som att ladda upp till GitHub.** Den finns fortfarande bara lokalt tills du pushar den.

## 6. Git push – skicka till GitHub

```bash
git push
```

Detta skickar dina lokala commits till ett konfigurerat fjärrrepository, förutsatt att du har rättigheter och att din branch är rätt inställd.

Efter en lyckad push kan andra som har åtkomst till repot hämta ändringarna.

**Skillnaden:**

```text
git add     → välj vad som ska sparas
git commit  → spara versionen på datorn
git push    → skicka versionerna till GitHub
```

## 7. Git pull – hämta andras ändringar

När du redan har repot på datorn och någon annan har gjort nya commits på GitHub:

```bash
git pull
```

`git pull` hämtar ändringar från den branch du följer och försöker integrera dem i din lokala branch. Ibland kan det uppstå en **konflikt** om två personer har ändrat samma del av en fil. Då måste konflikten lösas innan arbetet kan fortsätta.

**`clone` jämfört med `pull`:**

- `clone`: skapar en ny lokal kopia av ett repo.
- `pull`: uppdaterar ett repo som redan finns lokalt.

## 8. Praktiskt exempel – en vanlig arbetsdag

Du har redan klonat projektet och står i rätt projektmapp i terminalen.

```bash
# 1. Hämta kollegornas senaste ändringar
git pull

# 2. Redigera och spara dina filer i VS Code

# 3. Kontrollera vad som ändrats
git status

# 4. Välj ändringar
git add calculator.py

# 5. Spara dem som en lokal version
git commit -m "Improve calculator"

# 6. Dela dem via GitHub
git push
```

I ett grupprojekt använder ni ofta **egna branches och pull requests**. Det går vi igenom separat i lektionen om samarbete.

## 9. Vanliga missförstånd

### ”Jag har sparat filen i VS Code – är den på GitHub nu?”

Nej. Att spara filen uppdaterar bara arbetsfilen på datorn. Du behöver normalt `add`, `commit` och `push` för att publicera ändringen.

### ”Jag har gjort en commit – varför syns den inte på GitHub?”

För att `commit` sparar lokalt. Kör `git push` när du är redo att dela.

### ”Varför ska jag använda `git pull`?”

För att få in ändringar som andra har skickat till fjärrrepositoryt sedan du senast uppdaterade din kopia.

### ”Måste jag använda terminalen?”

Nej. VS Code har ett grafiskt gränssnitt för Git. Läraren rekommenderar ändå att lära sig terminalkommandona, så att du förstår vad som händer.

### ”Kan jag jobba vidare efter `git add`?”

Ja. Men ändringar du gör *efter* `git add` stage:as inte automatiskt. Kör `git status` och eventuellt `git add` igen innan du committar.

## 10. Ordlista

| Begrepp | Betydelse |
|---|---|
| **Versionshantering** | Att spara och följa ändringar över tid |
| **Repository / repo** | Git-projektet med dess historik |
| **Working directory** | Filerna du arbetar med |
| **Staging area** | Urvalet av ändringar inför nästa commit |
| **Commit** | En sparad version i den lokala historiken |
| **Remote** | Ett fjärrrepository, exempelvis på GitHub |
| **Clone** | Skapa en lokal kopia av ett befintligt repo |
| **Push** | Skicka lokala commits till remote |
| **Pull** | Hämta och integrera ändringar från remote |
| **Branch** | En separat utvecklingslinje; behandlas mer i senare lektion |
| **GUI** | Graphical User Interface – grafiskt gränssnitt |

## 11. Övningar

**Övning 1:** Du har ändrat `main.py` och vill spara ändringen som en lokal version. Vilka två Git-kommandon behöver du vanligtvis använda?

**Övning 2:** Du har gjort `git commit`, men ser inte ändringen på GitHub. Vilket kommando saknas?

**Övning 3:** En gruppmedlem har pushat ny kod till samma branch. Du har redan repot på din dator. Ska du använda `clone` eller `pull`?

**Övning 4:** Förklara med egna ord skillnaden mellan working directory och staging area.

**Övning 5:** Vad är skillnaden mellan Git och GitHub?

### Facit

1. `git add main.py` och `git commit -m "Beskriv ändringen"`.
2. Vanligtvis `git push`.
3. `git pull` (förutsatt att du arbetar på den branch som ska uppdateras).
4. Working directory är filerna du redigerar; staging area är det urval av ändringar som nästa commit ska innehålla.
5. Git är versionshanteringsverktyget; GitHub är en tjänst för att lagra och samarbeta kring Git-repositories online.

---

## Sammanfattning

Lärarens viktigaste poäng är att Git låter oss **versionshantera och samarbeta**. För att förstå Git behöver du hålla isär **arbetsfiler**, **staging**, **lokala commits** och **fjärrrepositoryt**.

> **Minnesregel:** *Redigera → Add → Commit → Push*. När andra har uppdaterat projektet: *Pull*.

**Källa:** Lärarens transkriberade genomgång om Git-grunder (cirka 0:00–7:16). Kompletterande förtydliganden och kommandon (`git status`, `git log`, `.gitignore`) har lagts till för praktisk användning.
