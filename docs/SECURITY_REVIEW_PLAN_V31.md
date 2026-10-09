# SECURITY REVIEW v3.1 – BUILD_PLAN.md (v3.1) mot SECURITY_REVIEW_PLAN_V3.md

| Egenskap | Värde |
| --- | --- |
| **Roll** | Security Reviewer (`AGENTS.md` §3) |
| **Granskat** | `BUILD_PLAN.md` v3.1, `PROJECT_ARCHITECTURE_BLUEPRINT.md` v1.2, `docs/adr/ADR-015-production-deferred.md`, `docs/adr/ADR-016-e2e-test-accounts.md`, `docs/DECISIONS.md` (main `719e860`, efter PR #6) |
| **Mot** | `docs/SECURITY_REVIEW_PLAN_V3.md`: villkoren K1–K8 och fynden V3-01–V3-06. Dessutom avsnitt 9B mot blueprint §0B, §36 och §99, samt blueprint v1.2 mot ADR-015, ADR-016 och `DECISIONS.md`. |
| **Datum** | 2026-10-09 |
| **Omfattning** | Plangranskning. Det finns ännu ingen kod. Allt gäller vad planen *kräver*, inte vad som är byggt eller testat. |

> Rapporten ändrar varken kod, infrastruktur eller plan. `BP:nn` avser radnummer i `BUILD_PLAN.md` v3.1. `BL:nn` avser radnummer i `PROJECT_ARCHITECTURE_BLUEPRINT.md` v1.2.

---

## 1. Slutsats

**Rekommendation till H1: godkänn v3.1 under villkor att fyra punkter rättas (avsnitt 7).** Den viktigaste är att blueprintens brödtext görs konsekvent med sin egen ändringslogg före S0.0, annars gäller motsägande instruktioner (V31-01).

Planen har arbetat in nästan alla villkor från förra omgången. Av de 14 punkterna är 12 åtgärdade och 2 delvis (K4 och V3-02, som är samma brist: raderingsregistrets urval i S0.7). Inga punkter är oåtgärdade. Det nya avsnittet 9B är i huvudsak förenligt med blueprint §0B, §36 och §99, med en ordningsbrist: gränssnittet byggs i 1F men behövs i S1A.5. Det finns inga öppna KRITISKA eller HÖGA fynd på plannivå.

| Resultat | Antal |
| --- | :-: |
| K1–K8: **ÅTGÄRDAD** / **DELVIS** / **EJ ÅTGÄRDAD** | 7 / 1 / 0 |
| V3-01–V3-06: **ÅTGÄRDAD** / **DELVIS** / **EJ ÅTGÄRDAD** | 5 / 1 / 0 |
| Avsnitt 9B: förenlig / med anmärkning | 5 / 1 punkt (av 6) |
| Blueprint v1.2 mot ADR och `DECISIONS.md`: loggen stämmer / brödtexten stämmer | 4 av 4 / 0 av 6 avsnitt |
| Nya fynd (V31-nn): MEDEL / LÅG | 3 / 4 |

**ÅTGÄRDAD betyder att planen kräver det, inte att något är byggt.** Återverifiering mot implementationen sker vid Stage 0 exit gate och i Production readiness-gaten (BP:192).

**Märkning:** **[Plan]** = står i planen. **[Observerat]** = kontrollerat i repot eller via GitHub-API vid granskningen. **[Verifiera]** = troligt men inte kontrollerat mot aktuell dokumentation eller juridisk rådgivning.

**Observerat vid granskningen:**
- Pull requests i detta repo, inklusive PR #6, skapas och mergas av ägarens eget konto (`zeroreality76-cmd`). Agenten har alltså ännu ingen egen identitet enligt BP:43.
- `main` är skyddad (`protected: true`), men `CODEOWNERS` finns inte i repot. H8 (BP:70) är öppen.

---

## 2. Villkoren K1–K8

| Villkor | Svar | Task-ref | Motivering |
| --- | :-: | --- | --- |
| **K1** Textsynk med ADR och beslut | **ÅTGÄRDAD** | BP:5, 26, 35, 41–53, 65, 67–68, 96 | Ägarlistan delad i `[S0.0]`/`[production]` (BP:41), Railway med tre tjänster (BP:45), S0.0:s mål är staging (BP:96), H3 omformulerad (BP:65), H5/H7 genomförda (BP:67–68), OB-5 skärpt (BP:26), status och miljörad uppdaterade (BP:5, 35). Rester, se V31-04: BP:3 hänvisar fortfarande till blueprint v1.1, BP:104 säger "före pilot" i stället för "före första kunddata", och ägarpunkterna 1, 6, 8, 9 och 11 saknar märkning. |
| **K2** Readiness-gate | **ÅTGÄRDAD** | 4A (BP:192–203), S0.8 (BP:179–184), H9 (BP:69), ägarpunkt 5 (BP:47) | C1 dokumenterad (BP:199), AI-nyckel för production först efter C1 (BP:200, 47), integritetsinformation och underbiträdeslista (BP:201), testkonto i production via ordinarie registrering (BP:202), ADR-016-hjälparen förbjuden där. S0.8 har fil, beroende på C1, säkerhetstester och gate. |
| **K3** Säkerhetstester i varje task | **ÅTGÄRDAD** | BP:86, 102, 119, 212, 216, 218, 220, 221, 236, 252, 263–267 | Raden finns i S0.0, S0.2 och i alla S1A-tasks utom S1A.2–S1A.4, samt i 1B–1H. S1A.2–S1A.4 saknar egen rad men är gate-bundna av S1A.0:s fixtures (BP:211). Planens status (BP:362) påstår rader i "S1A.1–S1A.10", vilket överdriver något. Se V31-04. |
| **K4** RLS- och raderingsurval på alla tabeller | **DELVIS** | Avsnitt 3 (BP:87–88) ✔, S0.7 (BP:169, 176) ✘ | Avsnitt 3 kräver nu *varje* tabell i app-schemat, med undantagslista under CODEOWNERS. Men S0.7 behåller det gamla urvalet: filraden säger "alla tabeller med `company_id`" (BP:169) och testet "fallerar om en tabell med `company_id` saknas i registret" (BP:176). Den som bygger S0.7 följer den raden, så `import_status_history` och organisationsnivåns tabeller kan fortfarande hamna utanför. **Åtgärd:** byt ut båda formuleringarna mot "alla tabeller i app-schemat". |
| **K5** ADR-016-tester och injektionskrav | **ÅTGÄRDAD** | S0.3 (BP:133), S1A.5 (BP:216), 1F (BP:265) | Hjälparen vägrar köra mot production, hemligheten finns bara i E2E-jobbets miljö, den importeras aldrig från `src/`, och den har en motiverad rad i undantagsfilen. S1A.5 har eget injektionskrav och 1F:s gate-kolumn kräver att injektionssetet passerar. Formuleringen "noll externa anrop" i S1A.5 är oklar, eftersom tasken själv anropar AI-leverantören. Se V31-07. |
| **K6** Rester av V2-17 | **ÅTGÄRDAD** | S0.1 (BP:110), S0.3 (BP:132), OB-10 (BP:31) | Parserfel som citerar cellinnehåll i loggförbudet, retention 24 månader som standardvärde, mekanism för inloggningshändelser via auth-hook eller jobb (verifieras mot aktuell dokumentation). Följdfynd: V31-03. |
| **K7** Första PR med enbart `CODEOWNERS`, blueprint v1.2 | **ÅTGÄRDAD** | S0.0 (BP:101, 103), blueprint v1.2 (BL:21–30) | Sekvensen är beskriven och ändringsloggen finns och pekar på ADR-015 och ADR-016. Brödtexten följer inte loggen, se V31-01. Beroendet av H8 är cirkulärt, se V31-05. |
| **K8** `companies`-policy, bootstrap, operatörstest, pekare, leverantörsregister | **ÅTGÄRDAD** | S0.3 (BP:132), S0.7 (BP:176), checklista (BP:322, 324), 1F (BP:265) | `companies` har medlemskapsbaserad policy. Första organisationen skapas med `created_by` och ägarmedlemskap i samma transaktion. Operatörstest finns (ingen kundroll kan skriva global flagga). P13 och P15 pekar rätt. Registertestet för exakt en aktiv leverantör finns. Beroende: `created_by` ska finnas i blueprint §9.1, men fältet står bara i loggen (V31-01). |

---

## 3. Fynden V3-01–V3-06

| Fynd | Allv. | Svar | Task-ref | Motivering |
| --- | :-: | :-: | --- | --- |
| V3-01 Readiness-gaten ofullständig | MEDEL | **ÅTGÄRDAD** | 4A (BP:199–202), S0.8 | Se K2. Alla fyra delar finns: C1, AI-nyckel efter C1, integritetsinformation med underbiträdeslista, testkonto i production. |
| V3-02 Kolumnbaserat urval | MEDEL | **DELVIS** | Avsnitt 3 (BP:87–88), S1A.1 (BP:212), S0.7 (BP:169, 176) | Avsnitt 3 och S1A.1 (`import_status_history` får `company_id`) är rättade. S0.7 är inte det (se K4), och blueprint §7 är oförändrad i brödtexten (BL:327). |
| V3-03 ADR-016:s testkrav | LÅG | **ÅTGÄRDAD** | S0.3 (BP:133), S0.1 (BP:111) | Alla fyra testkrav står i S0.3:s säkerhetstester. |
| V3-04 CODEOWNERS före H8 | LÅG | **ÅTGÄRDAD** | S0.0 (BP:101, 103) | Första PR med enbart `CODEOWNERS`, manuell merge, filen skyddar sig själv och `.github/`. Cirkulärt beroende, se V31-05. |
| V3-05 Detaljer och pekare | LÅG | **ÅTGÄRDAD** | S0.3 (BP:132), S0.7 (BP:176), BP:322, 324 | Se K8. Pekaren för P7 (BP:316) nämner fortfarande inte S1A.0 (V31-04). |
| V3-06 Leverantörsbegränsning i 1F | LÅG | **ÅTGÄRDAD** | 1F (BP:265), 9B punkt 6 (BP:289) | Endast `AnthropicProvider`, inga stubbade klasser (§0B), registertest för exakt en aktiv leverantör, och ett test att inaktiv leverantör inte kan väljas via konfiguration. |

---

## 4. Avsnitt 9B mot blueprint §0B, §36 och §99

| 9B | Innehåll | §0B | §36 | §99 | Svar |
| --- | --- | :-: | :-: | :-: | :-: |
| 1 | Leverantörsneutralt gränssnitt, prompter och scheman versionerade, inget läckage utanför `infrastructure/ai/` | – | ✔ (§49-gränsen) | – | **Förenlig** |
| 2 | Kontraktstestsvit i `tests/contract/ai_provider/` med testdubblett som bara finns under `tests/` | ✔ testdubletter tillåtna enbart i `tests/` och aldrig importerade av `src/` | ✔ ingen `MockProvider` i produktkod | ✔ "inga prompter eller kunddata i loggar" (§63, §99) | **Med anmärkning**, se V31-02 |
| 3 | Checklista för ny leverantör (villkor, DPA, ingen träning, retention, EU-behandling, underbiträdeslista, evals, feature flag, ägarens godkännande) | ✔ aktivering via flagga, inte falsk funktion | ✔ OB-1-kontroll mot aktuell dokumentation | ✔ motsvarar §99:s leverantörskrav | **Förenlig**, tillägg i V31-06 |
| 4 | Ingen automatisk reservväxling | ✔ | – | ✔ kunddata går aldrig till mottagare som kunden inte informerats om | **Förenlig** |
| 5 | Kärnan fungerar utan AI, AI-funktioner ger tydligt fel enligt §62 | ✔ "tydligt fel, inte en låtsasrespons" | ✔ | – | **Förenlig** (konsekvent med §8:s nivåer och §42:s "endast verifierad beräknad data") |
| 6 | Säkerhetstester: kontraktssvit, analys utan AI, inaktiv leverantör ej valbar | ✔ | ✔ | – | **Förenlig** |

**Vad som stämmer:**
- Avsnittet respekterar §0B:s förbud mot mock-lägen i drift: testdubbletten finns bara under `tests/`, och en leverantör som inte godkänts går inte att välja i staging eller production.
- Byte kräver ägarens medvetna, loggade beslut (flaggändringar finns i auditloggens händelsekatalog, BP:132).
- §99:s krav på DPA, ingen träning och dataskyddsvillkor ligger i checklistan (9B punkt 3a).

**Vad som saknas eller bör tydliggöras (V31-02, V31-06):**
1. Ordningen: `AIProvider` och `AnthropicProvider` byggs i 1F (BP:265), men AI används första gången i S1A.5 och S1A.8. Kontraktssviten skapas också i 1F.
2. 9B nämner "ChatGPT/OpenAI, Gemini eller annan" men inte uttryckligen att en ny leverantör ska vara molnbaserad. Blueprint §36 och §0A förbjuder lokala modeller.
3. Planen nämner inte att kunder ska informeras i förväg innan en ny leverantör slås på.
4. §99:s dataminimering (högst 10 sample values, maskade personuppgifter) sker uppströms adaptern. Kontraktssviten bör därför också testa gränsen där kontexten byggs, inte bara adaptern.

**Blueprint §36 och OB-1:** §36 listar tre leverantörsklasser som implementationer och säger att Stage 1 inte ska låsas till en viss leverantör. 1F implementerar bara en. Det är förenligt med gränssnittsneutralitet, men v1.2:s ändringslogg nämner det inte, och OB-1-raden i blueprinten (BL:73) säger fortfarande "Bestäms vid S1F.2", trots att `DECISIONS.md` beslutat och första användningen är S1A.5.

---

## 5. Blueprint v1.2 mot ADR-015, ADR-016 och DECISIONS.md

### 5.1 Ändringsloggen (BL:21–30)

| Loggrad | Mot | Stämmer? |
| --- | --- | :-: |
| 1: §74, §67A, §68, production före första kunddata, kompenseras av 4A | ADR-015 ("Ändrar: §74, §67A, §68") | **Ja** |
| 2: §0B, testhjälpare via admin-API, bara `tests/e2e/`, aldrig mot production | ADR-016 | **Ja** (ADR:n har fler villkor, bland annat hemlighetens begränsning och undantagsfilen, men inga motsägelser) |
| 3: §7, `import_status_history` får `company_id` | BUILD_PLAN S1A.1 (BP:212), V3-02 | **Ja** |
| 4: §9.1, `organizations` får `created_by` | BUILD_PLAN S0.3 (BP:132), V3-05 | **Ja** |

### 5.2 Brödtexten mot loggen

Loggen säger "Inga andra delar av dokumentet ändras" och ändrar samtidigt innebörden av sex avsnitt. **Brödtexten i dessa avsnitt är oförändrad och säger fortfarande det gamla:**

| Avsnitt | Rad | Vad brödtexten säger | Vad loggen/ADR/planen säger |
| --- | --- | --- | --- |
| §74 S0.0-gate | BL:1402 | `health` svarar "i **staging** och **production**" | Bara staging (ADR-015, BP:104) |
| §74 Stage 0 exit gate | BL:1430 | "`Deployed in staging and production PASS`" | Staging; production via 4A |
| §67A punkt 2–3 | BL:1318–1319 | Supabase och Railway "för staging och production" | Production först före kunddata |
| §68 | BL:1328–1333 | "Minst två molnmiljöer", production beskriven som en miljö som finns | `staging` nu, `production` före kunddata |
| §0B | BL:161 | Testkonton "skapade genom den vanliga registreringen" | Admin-API-hjälpare i staging (ADR-016) |
| §7 | BL:327 | `import_status_history` utan `company_id` | Med `company_id` (loggrad 3) |
| §9.1 | BL:373 | `id`, `name`, `created_at` | Med `created_by` (loggrad 4) |

**Konsekvens:** `AGENTS.md` anger blueprinten som första källa och att konflikter ska rapporteras som BLOCKER. En byggagent som läser §74 eller §67A ser ett krav på production i Stage 0 som den enligt ADR-015 inte ska uppfylla, och kan antingen stanna eller följa fel text. Se V31-01.

### 5.3 DECISIONS.md och ADR-filerna

| Kontroll | Resultat |
| --- | :-: |
| ADR-015 och ADR-016 har statusen "Godkänd" och `DECISIONS.md` säger samma sak | **Stämmer** |
| `DECISIONS.md` OB-5 (EU, BLOCKER om det saknas) mot BP:26 | **Stämmer** |
| `DECISIONS.md` OB-6 mot BP:27 (identisk text) | **Stämmer** |
| `DECISIONS.md` OB-1 (Anthropic, neutralt gränssnitt, `new_ai_provider`, kontroll före kunddata) mot BP:47, 199–200, 265, 9B | **Stämmer** |
| `DECISIONS.md` C1–C4 mot planen: C1 ↔ BP:199, C2 ↔ avsnitt 4A, C3 ↔ 9B punkt 3b, C4 ↔ BP:26 | **Stämmer** |
| `DECISIONS.md` "Tills kontrollen är gjord: bara syntetiska data" mot S1A.5 (BP:216) | **Stämmer** (står bara i S1A.5, inte i S1A.8 eller 1F, men produktionsnyckeln skapas inte före C1) |
| `DECISIONS.md` "Underlag: BUILD_PLAN.md v3" | **Föråldrad** (nu v3.1) |
| `DECISIONS.md` nämner inte OB-10 (retention 24 månader) | **Avsiktligt**: det är ett standardvärde, inte ett beslut |
| Blueprint OB-1-raden (BL:73) mot beslutet | **Föråldrad** (se avsnitt 4) |

---

## 6. Nya fynd (V31-nn), sorterade efter allvarlighet

### MEDEL

#### V31-01 – Blueprintens brödtext följer inte v1.2-loggen

- **Observation:** Se 5.2. Sju ställen i blueprinten (BL:161, 327, 373, 1318–1319, 1328–1333, 1402, 1430) säger motsatsen till vad loggen, ADR-015, ADR-016 och planen kräver.
- **Risk:** Foundation Builder läser blueprinten först (`AGENTS.md`). Den ser ett krav på production i S0.0 och en regel om ordinarie registrering som ADR:erna har upphävt. Resultat: antingen en onödig BLOCKER som stoppar S0.0, eller att agenten väljer själv vilken text som gäller, vilket `AGENTS.md` förbjuder ("gissa aldrig"). Det gäller särskilt §67A, som säger att saknade konton ska rapporteras som BLOCKER.
- **Åtgärd:** Architect för in ändringarna i respektive avsnitt, eller lägger en tydlig hänvisning vid varje (t.ex. "Ändrad i v1.2, se ADR-015"). Gör det före S0.0 (villkor K9 i avsnitt 7).

#### V31-02 – `AIProvider` byggs för sent för S1A.5 och S1A.8

- **Observation:** [Plan] Gränssnittet, `AnthropicProvider` och kontraktssviten i `tests/contract/ai_provider/` byggs i 1F (BP:265, 285). Bild- och dokumentextraktion (S1A.5, BP:216) och AI-mappning nivå 3 (S1A.8, BP:219) använder AI "via adapter" två etapper tidigare. Blueprintens egen byggordning har samma ordning (§75 före §80).
- **Risk:** Utan ett färdigt gränssnitt anropar S1A.5 leverantörens SDK direkt för att bli klar. Det bryter §36 och §49, och det är precis det som 9B ska förhindra. Injektionstestet i S1A.5 kräver dessutom en provider att köra mot.
- **Åtgärd:** Flytta `AIProvider`, `AnthropicProvider` och kontraktssviten (inklusive injektionssetet) till en task före S1A.5, t.ex. **S1A.0b**. 1F behåller resten (tool registry, orchestrator, policies, chatt).

#### V31-03 – Retention och gallring av `audit_log` saknar mekanism och rättslig grund

- **Observation:** [Plan] Runtime-roller får bara `INSERT` på `audit_log` (BP:131), och loggen sparas 24 månader, därefter gallring (OB-10, BP:31, 132). Ingen roll eller process är angiven för gallringen. Blueprint §66 säger att auditlogg bara får behållas "där lagligt nödvändigt".
- **Risk:** Den som bygger gallringen ger en app-roll `DELETE` på loggen, vilket försvagar oföränderligheten, eller så sker ingen gallring alls. Att behålla användar- och företags-id i 24 månader efter radering av ett företag kan strida mot §66 om det inte finns grund.
- **Åtgärd:** Ange att gallringen körs av `app_deleter`-processen (BP:174) eller en egen roll med `DELETE` enbart för detta ändamål, med eget test. Dokumentera grunden för 24 månader i S0.8:s integritetsinformation. [Verifiera] om grunden håller bör kontrolleras med juridisk rådgivning, vilket jag inte kan bedöma.

### LÅG

#### V31-04 – Föråldrade hänvisningar och överdrivna statusrader
- BP:3 säger "Följer blueprint v1.1" men blueprinten är v1.2.
- BP:104 säger "Production tas när det skapats, före pilot." ADR-015 säger "före första kunddata".
- Ägarpunkterna 1, 6, 8, 9 och 11 (BP:43, 48, 50, 51, 53) saknar `[S0.0]`/`[production]`-märkning. Skriv att omärkta punkter gäller från S0.0.
- Checklistans P7-pekare (BP:316) saknar S1A.0.
- S1A.2–S1A.4 saknar egen `Säkerhetstester`-rad och bör peka på S1A.0:s fixtures. BP:362 påstår rader i "S1A.1–S1A.10".
- `DECISIONS.md` anger underlag "BUILD_PLAN v3". Blueprint OB-1-raden (BL:73) är föråldrad.

#### V31-05 – H8 och första PR är cirkulära, och agentens identitet är ännu inte separat
- BP:100 gör H8 till beroende för S0.0, men H8-beviset kräver att `CODEOWNERS` finns (BP:101), och filen är S0.0:s första PR. Skriv det som ett förberedande steg, t.ex. **S0.0a**, före H8.
- [Observerat] Dagens PR:er skapas och mergas av samma konto, ägarens. Med Code Owner-krav kan den som skapar en PR inte godkänna den. Planen har en övergångsregel (BP:50), men den bör ha en uttrycklig tidpunkt för när approvals ändras från 0 och agenten byter identitet.

#### V31-06 – Tillägg till 9B och kundvillkor
- Lägg i checklistan (9B punkt 3) att en ny leverantör ska vara molnbaserad (§0A, §36).
- Lägg ett steg om att kunder informeras innan en ny leverantör aktiveras. [Verifiera] Hur ett sådant krav ser ut beror på de villkor som kommer att gälla mellan kund och tjänst.
- Kontraktssviten bör testa dataminimeringen (högst 10 sample values, maskerade personuppgifter) där kontexten byggs, inte bara i adaptern (§99).
- Planen har ingen task för kundvillkor eller personuppgiftsbiträdesavtal med kunderna. S0.8 täcker bara integritetsinformation. Besluta om juridisk granskning ingår i H9. [Verifiera] Jag kan inte bedöma det juridiskt.

#### V31-07 – Testfall att lägga till
- S0.3: en användare kan inte lägga till sig själv som medlem i en organisation hen inte skapade (bootstrap-policyn, BP:132).
- S1A.5: "noll externa anrop" (BP:216) bör preciseras till "inga anrop utöver AI-leverantörens, och inga verktygsanrop". Tasken själv anropar leverantören.

---

## 7. Villkor för att godkänna v3.1

### Före S0.0

| # | Åtgärd | Fynd |
| --- | --- | --- |
| K9 | Rätta blueprintens brödtext (BL:161, 327, 373, 1318–1319, 1328–1333, 1402, 1430) så att den stämmer med v1.2-loggen, eller lägg hänvisningar vid varje | V31-01 |
| K10 | Byt ut "alla tabeller med `company_id`" i S0.7 (BP:169, 176) mot "alla tabeller i app-schemat" | K4, V3-02 |
| K11 | Förtydliga H8/första PR som S0.0a, och sätt tidpunkt för byte av agentidentitet | V31-05 |

### Före respektive task

| Åtgärd | Fynd |
| --- | --- |
| Flytta `AIProvider`, `AnthropicProvider` och kontraktssviten till en task före S1A.5 | V31-02 (före Stage 1A) |
| Ange vilken roll som gallrar `audit_log` och grunden för 24 månader | V31-03 (före S0.3, S0.8) |
| Rätta hänvisningar och pekare, lägg tillägg i 9B, lägg till testfallen | V31-04, V31-06, V31-07 |

---

## 8. Status (AGENTS.md §4)

**Completed:**
- Granskning av `BUILD_PLAN.md` v3.1 mot K1–K8 och V3-01–V3-06, avsnitt 9B mot blueprint §0B, §36 och §99, samt blueprint v1.2 mot ADR-015, ADR-016 och `DECISIONS.md`.
- Resultat: K1–K8: 7 ÅTGÄRDAD, 1 DELVIS. V3-01–V3-06: 5 ÅTGÄRDAD, 1 DELVIS. 0 EJ ÅTGÄRDAD. 7 nya fynd (3 MEDEL, 4 LÅG), inga HÖGA.

**Verified (med bevis):**
- Radnummer är kontrollerade mot filerna med sökning.
- Att blueprintens brödtext (BL:161, 327, 373, 1318–1319, 1328–1333, 1402, 1430) är oförändrad är kontrollerat med sökning.
- Att PR #6 skapades och mergades av `zeroreality76-cmd` är kontrollerat via GitHub-API.
- Påståenden märkta **[Verifiera]** är inte kontrollerade mot leverantörsdokumentation eller juridisk rådgivning.

**Tests:** Inga. Det finns ingen kod, så alla bedömningar gäller plantext.

**Current issue:**
- Blueprintens brödtext säger emot sin egen ändringslogg på sju ställen (V31-01).
- S0.7 behåller det gamla tabellurvalet (K4).
- `AIProvider` byggs i 1F men behövs i S1A.5 (V31-02).
- H8 är öppen: `CODEOWNERS` saknas och agenten saknar egen identitet.
- Anthropics villkor och personuppgiftsavtal (`DECISIONS.md` C1) är inte kontrollerade, vilket jag inte har gjort.

**Next:** Architect skriver v3.2 med K9–K11. Security Reviewer granskar den mot dessa. Återverifiering mot implementationen sker vid Stage 0 exit gate. Ägaren beslutar H1 och genomför H8.
