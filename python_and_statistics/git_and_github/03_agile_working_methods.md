# 03 – Agila arbetssätt

> **Ämne:** Git & Agila Arbetssätt · **Underlag:** lärarens genomgång om agila arbetssätt.

## Quick Reference

| Begrepp | Enkel förklaring |
|---|---|
| Agilt arbetssätt | Arbeta i mindre steg, få feedback och anpassa planen. |
| Vattenfallsmetoden | Projektets faser genomförs huvudsakligen i följd. |
| Iteration | En omgång arbete som följs av utvärdering och förbättring. |
| Sprint | En tidsbestämd arbetsperiod i Scrum. |
| Scrum | Ramverk med roller, händelser och gemensamma arbetsresultat. |
| Product backlog | Prioriterad lista över vad produkten behöver. |
| Sprint backlog | Arbetet som valts och planerats för den aktuella sprinten. |
| Inkrement | Den användbara produkt som finns efter genomfört arbete. |
| User story | Beskriver ett behov ur användarens perspektiv. |
| Acceptance criteria | Villkor som avgör om en viss user story är uppfylld. |
| Definition of Done | Gemensamma kvalitetskrav för när arbetet räknas som klart. |
| Kanban | Synliggör arbetsflödet och begränsar pågående arbete. |
| WIP | Work in Progress: uppgifter som pågår just nu. |
| XP | Extreme Programming: arbetssätt med fokus på kodkvalitet och samarbete. |

## 1. Varför arbetar man agilt?

I ett traditionellt **vattenfallsprojekt** arbetar man ofta genom större faser: krav → planering → utveckling → test → leverans. Det kan vara svårt att hantera ändrade behov sent i projektet.

**Agilt** betyder att man i stället arbetar i mindre delar, kommunicerar ofta, visar resultat tidigt och anpassar nästa steg efter återkoppling. Målet är att leverera nytta, inte bara att följa en ursprunglig plan.

**Exempel:** Ett team ska bygga en e-handelsanalys. I stället för att bygga alla rapporter innan någon får se dem börjar teamet med en enkel försäljningsgraf. Beställaren ger feedback, och teamet väljer sedan vad som är viktigast att utveckla härnäst.

## 2. Det agila manifestet

Genomgången lyfter fram fyra prioriteringar:

1. **Individer och interaktioner** framför processer och verktyg.
2. **Fungerande programvara** framför omfattande dokumentation.
3. **Kundsamarbete** framför kontraktsförhandling.
4. **Anpassning till förändring** framför att följa en plan.

**Viktigt:** Det betyder inte att verktyg, dokumentation, avtal eller planer saknar värde. Det betyder att punkterna till vänster värderas högre när man behöver prioritera.

## 3. Scrum – arbete i sprintar

Scrum är ett ramverk för att utveckla en produkt stegvis. Arbetet delas upp i **sprintar**: tidsbestämda perioder då teamet arbetar mot ett tydligt mål och skapar ett användbart resultat.

### Roller

- **Product Owner (produktägare):** ansvarar för att maximera produktens värde och prioritera produktens behov.
- **Scrum Master:** hjälper teamet att förstå och använda Scrum och undanröjer hinder för arbetssättet.
- **Utvecklarna:** bygger och testar produkten samt organiserar hur arbetet ska utföras.

### Viktiga listor och resultat

- **Product backlog:** allt som kan behöva göras, ordnat efter prioritet.
- **Sprint backlog:** sprintens mål, valda uppgifter och plan för att genomföra dem.
- **Inkrement:** den användbara version av produkten som arbetet bidragit till.

### Viktiga möten

| Händelse | Syfte |
|---|---|
| Sprint Planning | Bestämma sprintmål och planera arbetet. |
| Daily Scrum | Kort daglig avstämning för att anpassa planen. |
| Sprint Review | Visa resultat och få återkoppling. |
| Sprint Retrospective | Förbättra hur teamet samarbetar och arbetar. |

**Exempel:** Under sprintplaneringen bestämmer gruppen att bygga en funktion som läser en CSV-fil. Under sprinten skriver och testar ni koden. På review visar ni funktionen. På retrospective pratar ni om vad som fungerade och vad ni vill göra bättre nästa gång.

## 4. User stories – vad behöver användaren?

En **user story** beskriver *vem* som behöver något, *vad* personen vill göra och *varför*.

**Mall:**

> Som **[roll]** vill jag **[funktion]** så att **[nytta]**.

**Exempel från genomgångens tema:**

> Som marknadschef vill jag se en graf över kundernas åldersfördelning så att jag kan identifiera viktiga målgrupper.

### Acceptance criteria – hur vet vi att just den uppgiften är klar?

Exempel:

- Det finns ett dataset med kundernas ålder.
- Programmet delar in åldrarna i grupper.
- En graf visar fördelningen korrekt.

### Definition of Done – vad betyder "klart" för hela teamet?

Detta är **gemensamma** krav som gäller flera uppgifter. Exempel:

- Koden fungerar och relevanta tester är godkända.
- Dokumentationen är uppdaterad.
- Ändringen är granskad och sammanfogad enligt teamets rutin.

**Skillnaden:** Acceptance criteria gäller *en särskild funktion*. Definition of Done gäller *teamets gemensamma kvalitetsnivå*.

### Epic och tasks

En **epic** är ett större behov som behöver delas upp i mindre user stories. En **task** är ett konkret arbetssteg för att genomföra en story, till exempel "skriv CSV-inläsning" eller "testa saknade värden".

## 5. Kanban – se hur arbetet rör sig

Kanban visualiserar arbetsflödet med en tavla. En enkel variant:

```text
To Do  →  In Progress  →  Testing  →  Done
```

Varje uppgift representeras av ett kort som flyttas när arbetet går framåt.

**WIP-gräns** betyder att man begränsar hur många uppgifter som får vara igång samtidigt. Om `In Progress` har gränsen 2 bör teamet slutföra eller hjälpa till med pågående arbete innan en tredje uppgift startas.

**Flaskhals:** Om många kort fastnar i `Testing` behöver teamet kanske prioritera testning innan fler nya uppgifter påbörjas.

Genomgången tar även upp **ledtid**, alltså hur lång tid det tar för en uppgift att bli klar. Begreppet kan definieras olika i olika sammanhang; i Kanban skiljer man ofta mellan ledtid från beställning och cykeltid från påbörjat arbete.

## 6. Extreme Programming (XP)

XP fokuserar på hur själva programmeringen kan göras med hög kvalitet och snabb feedback.

| Teknik | Vad betyder det? |
|---|---|
| Parprogrammering | Två personer arbetar tillsammans med samma kod. |
| TDD (Test-Driven Development) | Skriv ett test först, skriv sedan kod som får testet att gå igenom. |
| Collective Code Ownership | Flera i teamet får förbättra och ta ansvar för samma kod. |
| Continuous Integration | Integrera ändringar ofta för att hitta problem tidigt. |
| Refactoring | Förbättra kodens struktur utan att ändra vad den gör. |

Läraren nämner också XP:s värden: **kommunikation, enkelhet, återkoppling, mod och respekt**.

## 7. Scrum, Kanban och XP – vad skiljer dem åt?

| | Scrum | Kanban | XP |
|---|---|---|---|
| Huvudfokus | Planering och leverans i sprintar | Jämnt och synligt arbetsflöde | Kodkvalitet och programmeringspraxis |
| Typiskt verktyg/arbetssätt | Sprint backlog, review | Tavla och WIP-gränser | Parprogrammering, TDD |
| Tidsupplägg | Tidsbestämda sprintar | Kontinuerligt flöde | Kan kombineras med iterationer |

Metoderna kan kombineras, men man behöver vara tydlig med vilka arbetssätt teamet faktiskt använder.

## 8. Vanliga missförstånd

- **"Agilt betyder ingen planering."** Nej. Man planerar ofta, men är beredd att ändra planen.
- **"Scrum Master är chef över utvecklarna."** Nej. Rollen stödjer Scrum-arbetet, inte detaljstyr varje persons uppgifter.
- **"En user story är samma sak som en task."** Nej. Storyn beskriver användarbehovet; tasks beskriver konkreta steg.
- **"Done betyder att jag har skrivit koden."** Inte nödvändigtvis. Teamets Definition of Done kan kräva testning, granskning och dokumentation.
- **"Kanban handlar bara om att flytta kort."** Nej. Begränsning av pågående arbete och förbättrat flöde är centralt.
- **"Agilt betyder att dokumentation är onödig."** Nej. Dokumentation behövs när den skapar värde.

## 9. Övningar

**Övning 1.** Förklara med egna ord skillnaden mellan vattenfall och agilt.

**Övning 2.** Vem prioriterar produktens behov i Scrum? Vem hjälper teamet att använda Scrum?

**Övning 3.** Skriv en user story för en funktion som tar bort dubbletter i ett dataset.

**Övning 4.** Ange två acceptance criteria för din user story och två exempel på Definition of Done.

**Övning 5.** En Kanban-tavla har WIP-gränsen 3 för `In Progress`. Tre uppgifter pågår. Vad bör teamet göra innan det startar en fjärde?

**Övning 6.** Vilken XP-teknik passar beskrivningen: "Vi skriver ett test som misslyckas innan vi implementerar funktionen"?

## 10. Facit och exempelsvar

1. **Vattenfall:** större faser i följd. **Agilt:** mindre leveranser med återkommande feedback och anpassning.
2. **Product Owner** prioriterar produktens behov. **Scrum Master** hjälper teamet med Scrum.
3. *Som dataanalytiker vill jag kunna ta bort dubbletter så att mina sammanställningar inte räknar samma rad flera gånger.*
4. **Acceptance criteria:** dubblettrader identifieras; funktionen returnerar ett dataset utan dessa dubbletter. **Definition of Done:** tester godkända; kod granskad enligt teamets rutin.
5. Hjälpa till att slutföra eller flytta en pågående uppgift innan ytterligare arbete startas.
6. **TDD – testdriven utveckling.**

---

### Kom ihåg

**Agilt = små steg + kommunikation + feedback + anpassning.**

**Scrum = sprintar och tydliga ansvarsområden.**

**Kanban = synligt flöde och begränsat pågående arbete.**

**XP = tekniker för bättre kod och samarbete.**

*Kommentar: Lektionsanteckningarna följer lärarens innehåll och har förtydligats med egna nybörjarexempel. Några begrepp har fått mer precisa förklaringar där föreläsningen förenklar.*
