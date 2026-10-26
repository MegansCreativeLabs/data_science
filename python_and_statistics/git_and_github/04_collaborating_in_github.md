# 04 – Samarbeta i GitHub

> **Ämne:** Git & Agila Arbetssätt · **Underlag:** lärarens genomgång ”Samarbeta i GitHub” (video #4).

## Quick Reference

| Begrepp/kommando | Vad betyder det? |
|---|---|
| `main` | Projektets huvudbranch, som teamet vill hålla stabil. |
| Branch | En separat arbetslinje där du kan göra ändringar utan att direkt ändra `main`. |
| `git branch` | Visar dina lokala brancher; `*` markerar den aktiva. |
| `git checkout -b feature/namn` | Skapar en ny branch **och** byter till den. |
| `git checkout main` | Byter till `main`. |
| `git status` | Visar ändrade filer och aktuell branch. |
| `git add .` | Lägger ändringar i staging area inför en commit. |
| `git commit -m "Meddelande"` | Sparar en lokal version av ändringarna. |
| `git push -u origin feature/namn` | Publicerar en ny branch på GitHub och kopplar den till fjärrbranchen. |
| `git pull` | Hämtar och integrerar ändringar från den spårade fjärrbranchen. |
| Issue | En uppgift, ett önskemål eller ett problem att hantera. |
| GitHub Projects | En tavla eller annan vy för att organisera projektets arbete. |
| Pull request (PR) | Förslag att föra in ändringar från en branch i en annan. |
| Review | Granskning av ändringarna i en pull request. |
| Merge | Slår samman ändringar från två brancher. |
| Merge conflict | Git behöver hjälp att avgöra hur konkurrerande ändringar ska kombineras. |

## 1. Varför samarbeta med GitHub?

**Git** versionshanterar koden. **GitHub** är en tjänst där ett team kan lagra sitt gemensamma repository, planera uppgifter och granska ändringar. Flera personer kan arbeta samtidigt utan att alla behöver ändra direkt i huvudversionen.

Lärarens arbetsflöde är:

```text
Bjud in teamet
      ↓
Skapa issues och organisera dem i Projects
      ↓
Uppdatera main och skapa en egen branch
      ↓
Skriv kod → add → commit → push
      ↓
Öppna pull request → granskning
      ↓
Lös eventuella konflikter → merge
      ↓
Uppdatera din lokala main med git pull
```

## 2. Brancher – parallella arbetslinjer

En **branch** (gren) gör att du kan arbeta på en funktion utan att dina halvfärdiga ändringar hamnar i `main`. Teamet kan exempelvis ha en branch för en graf och en annan för att läsa in CSV-filer.

```text
main:         A ─── B ─────────── E
                   \             /
feature/graf:        C ─── D ────
```

Här utvecklas funktionen separat i `C` och `D`. När den granskats kan den slås ihop med `main`.

I videon nämns också `develop` och `release`. De är **möjliga** delar av ett arbetsflöde, inte något varje projekt måste ha. En release är en publicerad version; för att kunna återvända till en viss version kan man exempelvis använda Git-taggar.

### Skapa och byta branch

```bash
git branch
# Visar lokala brancher

git checkout -b feature/age_visualization
# Skapar OCH byter till en ny branch

git branch
# * feature/age_visualization visar var du befinner dig

git checkout main
# Byter tillbaka till huvudbranchen

git checkout feature/age_visualization
# Byter tillbaka till din arbetsbranch
```

**Kom ihåg:** `git checkout main` skapar inte en ny branch. Flaggan `-b` gör att en branch skapas. Det finns också ett nyare alternativ: `git switch -c feature/age_visualization`.

## 3. Bjud in medarbetare

I genomgången öppnar läraren repositoryts **Settings → Collaborators** och bjuder in ett annat GitHub-konto. Den inbjudna personen accepterar inbjudan innan hen kan samarbeta med skrivbehörighet.

Ett **publikt repository** kan normalt läsas av andra utan inbjudan. Det är något annat än att få **behörighet att ändra** i projektet. Exakta menyer kan förändras över tid och skilja sig mellan personliga konton och organisationer.

## 4. Issues – vad ska göras?

En **issue** är en post där teamet kan beskriva ett arbete, utse ansvarig (*assignee*), sätta etiketter (*labels*) och diskutera uppgiften.

Lärarens exempel handlar om att visualisera kundernas åldersfördelning:

> **Som marknadschef vill jag se en graf över kundernas åldersfördelning, så att jag kan identifiera viktiga målgrupper.**

Det är en **user story** eftersom den berättar **vem** som vill ha något, **vad** personen vill ha och **varför**.

### Acceptance criteria – hur vet vi att uppgiften är klar?

```text
Givet att ett dataset innehåller kundernas åldrar,
när analysen körs,
så ska en graf visa fördelningen över olika åldersintervall.
```

Detta kallas ofta **Given–When–Then**. Det beskriver ett kontrollerbart resultat. En issue måste inte vara en user story; den kan också handla om exempelvis en bugg.

## 5. GitHub Projects – överblick med Kanban

I videon skapar läraren ett **Project** med en tavla och kopplar sina issues till den. Tavlan har kolumner som kan anpassas:

| To do | In progress | Review | Done |
|---|---|---|---|
| Väntar på start | Någon arbetar | Behöver granskas | Klart |

En issue kan flyttas mellan kolumnerna när status ändras. Ett Project kan samla uppgifter från **flera repositories**. Det gör att teamet kan se både vad som återstår och vem som arbetar med vad.

**Skillnaden:** En *issue* beskriver själva uppgiften. Ett *Project* hjälper teamet att **organisera och följa upp** uppgifter.

## 6. Arbeta på din branch – steg för steg

Utgå från att du redan har klonat repositoryt och står i projektmappen. Börja med att kontrollera att du inte har osparade ändringar som kan störa branchbytet.

```bash
git status
# Kontrollera om något är ändrat

git checkout main
# Gå till huvudbranchen

git pull
# Hämta senaste ändringarna till main

git checkout -b feature/age_visualization
# Skapa en ny branch utifrån uppdaterad main
```

Skapa eller redigera sedan exempelvis `visualization.py` i VS Code:

```python
def show_age_distribution():
    print("Här ska kundernas åldersfördelning visas")
```

När du vill spara ditt arbete i Git:

```bash
git status
# Se vilka filer som ändrats

git add visualization.py
# Välj filen som ska ingå i nästa commit

git commit -m "Add age visualization placeholder"
# Spara en version lokalt

git push -u origin feature/age_visualization
# Publicera branchen på GitHub första gången
```

**Varför `-u`?** Det ställer in en koppling (*upstream*) mellan din lokala branch och fjärrbranchen. Nästa gång räcker det normalt med `git push` från samma branch.

**Varför `origin`?** Det är vanligtvis namnet på fjärr-repositoryt, ofta GitHub. Det är inte ett särskilt ord som betyder GitHub i sig.

**Varför `git add visualization.py` i stället för `git add .`?** Du väljer exakt vad som ska ingå. `git add .` kan vara smidigt men kan även råka inkludera andra ändringar.

## 7. Pull request – be teamet granska din kod

När branchen har pushats öppnar du en **pull request** på GitHub.

1. Välj **base: `main`** – dit du vill föra in ändringarna.
2. Välj **compare: `feature/age_visualization`** – branchen med ditt arbete.
3. Skriv en titel och en beskrivning av vad du ändrat.
4. Koppla gärna relevant issue och be en teammedlem göra en **review**.
5. Teamet granskar kodändringarna och kan kommentera eller begära ändringar.
6. När ändringarna är godkända och eventuella kontroller passerar kan pull requesten **mergas**.

En **draft pull request** är ett sätt att visa pågående arbete som ännu inte är redo för slutlig granskning.

**Viktigt:** Att skapa en pull request är **inte samma sak som att mergea**. En PR är ett förslag. Först vid merge blir ändringarna en del av målbranchen.

### Koppla en issue till din PR

I videon kopplas pull requesten till en issue. På GitHub kan en korrekt länkad issue stängas automatiskt när pull requesten mergas till rätt branch. Ett vanligt sätt att skriva i PR-beskrivningen är:

```text
Closes #12
```

Här är `12` bara ett **exempel** på ett issue-nummer. Använd numret på den verkliga uppgiften i ditt repository.

## 8. Merge conflicts – när Git inte kan välja själv

En **merge conflict** kan uppstå om två brancher ändrat samma del av en fil på olika sätt. Git kan då inte alltid avgöra vilken version som är rätt.

Tänk dig att en branch innehåller:

```python
print("Hej från main")
```

Och en annan branch innehåller:

```python
print("Hej från feature-branchen")
```

Vid en konflikt kan Git visa markeringar i stil med:

```text
<<<<<<< HEAD
print("Hej från main")
=======
print("Hej från feature-branchen")
>>>>>>> feature/example
```

Det betyder att du behöver bestämma vad den färdiga filen ska innehålla. Du kan välja ena versionen, den andra eller kombinera båda – **om resultatet är korrekt**.

Om båda raderna behövs kan den färdiga Python-filen exempelvis bli:

```python
print("Hej från main")
print("Hej från feature-branchen")
```

Konfliktmarkeringarna ska **inte** lämnas kvar i Python-koden. Spara den lösta versionen, markera konflikten som löst och slutför sammanslagningen enligt GitHubs eller Git:s instruktioner.

**En konflikt är inte ett tecken på att du gjort något fel.** Den betyder att Git behöver ett mänskligt beslut.

## 9. Efter merge – uppdatera din dator

När pull requesten mergats på GitHub har **fjärrversionen av `main`** ändrats. Din lokala `main` uppdateras inte automatiskt.

```bash
git checkout main
git pull
```

Nu får du ned de nya ändringarna. Därefter kan du skapa nästa arbetsbranch från den uppdaterade huvudbranchen.

**Viktig skillnad:** `git push` skickar upp dina commits. `git pull` hämtar ned och integrerar andras ändringar. `git checkout` byter branch. De tre kommandona gör olika saker.

## 10. Vanliga misstag och hur du undviker dem

| Misstag | Vad händer? | Bättre arbetssätt |
|---|---|---|
| Skriva direkt i `main` i ett teamprojekt | Ändringar kan hamna i huvudversionen utan granskning. | Skapa en egen branch. |
| Skapa en ny branch från gammal `main` | Du kan sakna andras senaste ändringar. | `git checkout main` följt av `git pull` innan ny branch. |
| Tro att `git commit` uppdaterar GitHub | Commit sparas först lokalt. | Kör `git push` när du vill publicera. |
| Tro att PR betyder att koden är färdigmergad | PR är bara ett förslag. | Kontrollera review och merge-status. |
| Glömma vilken branch som är aktiv | Du riskerar att ändra fel arbetslinje. | Kör `git branch` och `git status`. |
| Köra `git add .` utan att kontrollera | Oönskade filer kan följa med. | Granska med `git status`; använd `.gitignore`. |
| Radera konfliktmarkeringar utan att förstå koden | Resultatet kan bli fel även om Git accepterar det. | Läs båda versionerna och testa resultatet. |
| Få felmeddelande vid push eftersom fjärrbranchen har nyare ändringar | Din lokala branch är inte uppdaterad. | Läs felet, hämta/integrera relevanta ändringar och lös eventuella konflikter innan ny push. |

## 11. Övningar

**Övning 1 – Begrepp.** Förklara med egna ord skillnaden mellan en *branch*, en *issue* och en *pull request*.

**Övning 2 – Rätt ordning.** Sätt dessa i rätt ordning när du ska börja en ny uppgift: `git pull`, `git checkout -b feature/test`, `git checkout main`, `git commit`, `git add`, `git push`.

**Övning 3 – Välj kommando.** Vilket kommando använder du för att (a) se aktiv branch, (b) byta till `main`, (c) skapa och byta till en ny branch, (d) publicera en ny branch på GitHub?

**Övning 4 – Projektarbete.** Skriv en issue för en funktion som ska läsa en CSV-fil och visa antalet rader. Formulera en user story och ett acceptance criterion.

**Övning 5 – Förstå konflikter.** Två personer har ändrat samma rad i samma fil. Vad betyder det om Git visar en merge conflict? Vad ska ni göra innan ni mergar?

## 12. Facit och exempel på svar

**1.** En *branch* är en separat arbetslinje för kod. En *issue* beskriver något som behöver göras. En *pull request* föreslår att ändringar från en branch ska föras in i en annan och ger teamet möjlighet att granska dem.

**2.** Först `git checkout main` → `git pull` → `git checkout -b feature/test`. Efter att du ändrat filer: `git add` → `git commit` → `git push` (första gången kan du behöva `git push -u origin feature/test`).

**3.** (a) `git branch` eller `git status`; (b) `git checkout main`; (c) `git checkout -b feature/test`; (d) `git push -u origin feature/test`.

**4.** Exempel: ”Som analytiker vill jag kunna läsa en CSV-fil och se hur många rader den innehåller, så att jag snabbt kan kontrollera datamängdens storlek.” Acceptance criterion: ”Givet att filen finns och innehåller data, när funktionen körs, så visas antalet datarader.”

**5.** Git kan inte automatiskt avgöra hur de olika ändringarna ska kombineras. Teamet måste jämföra versionerna, välja eller kombinera rätt kod, testa och därefter slutföra merge.

---

## Sammanfattning

**Issues** beskriver vad som ska göras. **Projects** ger överblick. **Branches** låter personer arbeta parallellt. **Commits** sparar versioner. **Push** publicerar dem på GitHub. **Pull requests** gör granskning möjlig. **Merge** för in godkända ändringar i huvudbranchen. **Pull** håller din lokala kopia uppdaterad.

> **Minnesregel:** Uppdatera `main` → skapa branch → arbeta → `add` → `commit` → `push` → pull request → review → merge → uppdatera `main` igen.
