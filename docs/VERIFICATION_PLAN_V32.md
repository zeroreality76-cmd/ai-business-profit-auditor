# VERIFICATION v3.2 – BUILD_PLAN.md (v3.2) och PROJECT_ARCHITECTURE_BLUEPRINT.md (v1.2)

| Egenskap | Värde |
| --- | --- |
| **Roll** | Verifier (`AGENTS.md` §3) |
| **Verifierat** | De tre bifogade filerna `BUILD_PLAN.md` (v3.2), `PROJECT_ARCHITECTURE_BLUEPRINT.md` (v1.2, ändrad) och `docs/DECISIONS.md` |
| **Mot** | `docs/SECURITY_REVIEW_PLAN_V31.md`: villkoren K9, K10 och K11 (avsnitt 7) |
| **Dessutom** | Att inget annat ändrats i blueprinten än de rader som ändringsloggen v1.2 anger |
| **Datum** | 2026-10-09 |
| **Utgångsläge** | `main` `b724e0e` (efter PR #7) |

> **Viktigt:** De verifierade filerna är bifogade filer, **inte** de som ligger i repot. `main` innehåller fortfarande `BUILD_PLAN.md` v3.1 och blueprint v1.2 i den form PR #6 mergade. Rapporten gäller exakt de filer som anges med SHA-256 i avsnitt 2. Om andra filer committas senare ska hashvärdena jämföras innan resultatet åberopas.

> **Rollanmärkning:** Villkoren K9–K11 är formulerade av Security Reviewer (samma agent som skrivit den här rapporten, i en annan roll). Verifieringen är därför gjord mekaniskt med textdiff mot fasta baslinjer och inte genom omtolkning av kraven. Jag har inte byggt, skrivit eller ändrat planen eller blueprinten.

---

## 1. Resultat

| Punkt | Svar | Radreferens | Bevis |
| --- | :-: | --- | --- |
| **K9** Blueprintens brödtext stämmer med v1.2-loggen, eller har hänvisningar vid varje ställe | **OK** | BL:162, 328, 374, 1319–1320, 1329, 1403, 1431 (och OB-1 på BL:74) | Avsnitt 3: alla sju ställen har hänvisning eller rättad text, och varje hänvisning stämmer med ADR-015, ADR-016 och planen. |
| **K10** S0.7 gäller alla tabeller i app-schemat | **OK** | BP:169, 176 (och BP:87–88 oförändrade) | Avsnitt 4: båda formuleringarna är bytta. Sökning efter `alla tabeller med` och `tabeller med company_id` i hela v3.2 ger noll träffar. |
| **K11** H8/första PR som S0.0a, och tidpunkt för byte av agentidentitet | **OK** | BP:100–101 (kompletterat av BP:43, 50, 70) | Avsnitt 5: S0.0a är ett eget förberedande steg före H8, och bytet av identitet har en uttrycklig tidpunkt. |
| Blueprinten: inga andra ändringar än ändringsloggen v1.2 anger | **OK** | 10 diffblock, BL:8, 21–32, 74, 162, 328, 374, 1319–1320, 1329, 1403, 1431 | Avsnitt 6: samtliga block motsvaras av loggrad 1–5, versionsraden eller själva loggen. Rubriker, radantal och tecken är i övrigt oförändrade. |

**Resultat: 4 av 4 OK.** Inga EJ OK.

Anmärkningar som inte underkänner något av ovanstående finns i avsnitt 7 (två felaktiga radreferenser och en oexakt etikett).

---

## 2. Metod och underlag

Jämförelserna är gjorda med `diff` på radnivå mot tre fasta baslinjer. Baslinjen för "inget annat ändrat" i blueprinten är den ursprungliga v1.1 (commit `3b1658b`), inte den mergade v1.2, eftersom v1.2:s egen ändring annars aldrig skulle kontrolleras.

| Fil | Baslinje | SHA-256 |
| --- | --- | --- |
| Blueprint v1.1 (original, `3b1658b`) | – | `9a5fc1257b58edd6462e3a49507c5acb121a483087f126bf6f41950df45cfada` |
| Blueprint v1.2 (mergad på `main`, PR #6) | – | `dcab902dc9ab3394b04b4a577f2ca1f7b183a9ef3fdadb52abd337e1d363f3e0` |
| **Blueprint, bifogad (verifierad)** | v1.1 och v1.2 | `a0265bd65678c73f65c0d584a1af4fdd2632c5aa83e8002a8333507ced179b72` |
| BUILD_PLAN v3.1 (`main`) | – | `3b3340fb9bc529cbd3be069452d570b447abc71dcdb5d93eb763642ab35ca76c` |
| **BUILD_PLAN v3.2, bifogad (verifierad)** | v3.1 | `6241a5aaf0972deeae26499da872542796f8fad8a3ca088041ffce64ae307ce5` |
| DECISIONS (`main`) | – | `dd9073d8e69cf080360747780173e3fd4d7844877429ff712ea2166ad74662df` |
| **DECISIONS, bifogad (verifierad)** | `main` | `71b210b1bd152841b8573e89b98828045009b1555760672c9b91e37db89aee39` |

Kontroller utöver diffen: inga dolda eller osynliga tecken (Unicode-kategori Cf/Cc) i varken v1.1 eller den bifogade blueprinten, inga CR-radslut i någon av dem, samma teckenkodning (UTF-8), samma rubriklista bortsett från den nya rubriken `## Ändringslogg v1.1 → v1.2`, och radantalet 1 961 → 1 973 (+12) förklaras fullt av de tolv nya raderna i ändringsloggen (BL:21–32); övriga ändringar ersätter rader på plats.

---

## 3. K9 – Blueprintens brödtext

K9 (V31 avsnitt 7): *"Rätta blueprintens brödtext så att den stämmer med v1.2-loggen, eller lägg hänvisningar vid varje."*

| Avsnitt | Rad (före → efter) | Vad som finns nu | Stämmer med | Svar |
| --- | --- | --- | --- | :-: |
| §0B | 161 → **162** | Originalmeningen kvar, plus *"(Ändrad i v1.2, ADR-016: E2E-testkonton i staging får skapas via auth-leverantörens admin-API av en testhjälpare i `tests/e2e/`.)"* | ADR-016, loggrad 2, BP:133 | **OK** |
| §7 | 327 → **328** | Tabellraden kvar, plus *"(Ändrad i v1.2: tabellen får även `company_id`, §11A.)"* | Loggrad 3, BP:213 (S1A.1) | **OK** |
| §9.1 | 373 → **374** | Fältlistan ändrad på plats till `id`, `name`, `created_by`, `created_at`, plus *"(`created_by` tillagt i v1.2)"* | Loggrad 4, BP:132 (S0.3) | **OK** |
| §67A punkt 2–3 | 1318–1319 → **1319–1320** | Båda raderna har *"(v1.2, ADR-015: `production` skapas före första kunddata.)"* | ADR-015, loggrad 1, BP:44–46 (`[S0.0]`/`[production]`) | **OK** |
| §68 | 1328 → **1329** | Rubrikmeningen har *"(v1.2, ADR-015: `staging` skapas först och `production` före första kunddata, se BUILD_PLAN avsnitt 4A)"*. Tabellen under (en rad per miljö) är oförändrad. | ADR-015, loggrad 1, BP:192 (4A) | **OK** |
| §74 S0.0-gate | 1402 → **1403** | Texten rättad till *"i **staging**"*, plus hänvisning till readiness-gaten | ADR-015, BP:104 | **OK** |
| §74 Stage 0 exit gate | 1430 → **1431** | `Deployed in staging and production PASS` ersatt med `Deployed in staging PASS`, plus hänvisning | ADR-015, BP:187 | **OK** |

**Extra, utanför de sju:** OB-1-raden (BL:73 → **74**) är ändrad från *"Bestäms vid S1F.2…"* till *"Beslutad: Anthropic som enda aktiv leverantör i Stage 1 (`docs/DECISIONS.md`). Gränssnittet och kontraktstesterna byggs före S1A.5."* Det stämmer med `DECISIONS.md` (OB-1) och BP:212 (S1A.0b). Ändringen finns i loggrad 5.

**Kvarvarande `production` i blueprinten:** Jag sökte efter alla förekomster. De återstående (BL:46, 129, 150, 153, 171, 1087, 1334, 1340, 1514, 1766, 1794) beskriver production som en miljö som finns, som släpps efter release gate, eller som mål för slutlig acceptans. Ingen av dem kräver production i Stage 0.

**Anmärkning om formen:** På fyra ställen (BL:162, 328, 1319–1320, 1329) står originaltexten kvar tillsammans med hänvisningen. K9 tillåter detta uttryckligen ("eller lägg hänvisningar vid varje"), men en läsare ser då både den gamla och den nya regeln på samma rad. Hänvisningarna anger att v1.2 gäller.

---

## 4. K10 – S0.7 gäller alla tabeller i app-schemat

K10 (V31 avsnitt 7): *"Byt ut 'alla tabeller med `company_id`' i S0.7 (BP:169, 176) mot 'alla tabeller i app-schemat'."*

| Rad | v3.1 | v3.2 | Svar |
| --- | --- | --- | :-: |
| BP:169 (Filer) | "raderingsregister (lista över alla tabeller med `company_id`)" | "raderingsregister (lista över **alla tabeller i app-schemat**, med en motiverad undantagslista under CODEOWNERS)" | **OK** |
| BP:176 (Säkerhetstester) | "Täckningstest som fallerar om en tabell med `company_id` saknas i registret." | "Täckningstest som fallerar om en tabell i app-schemat saknas i registret eller i undantagslistan [K10, V3-02]." | **OK** |

**Övriga kontroller:**
- Sökning i hela v3.2 efter `alla tabeller med`, `tabell med company_id` och `tabeller med company_id` ger **noll träffar**.
- Avsnitt 3 (BP:87 och 88) är oförändrade och säger redan "varje tabell i app-schemat", så S0.7 och avsnitt 3 är nu konsekventa.
- BP:177 (gate), BP:237 (Stage 1B) och BP:212 (S1A.1, `import_status_history` får `company_id`) är oförändrade och förenliga.

---

## 5. K11 – S0.0a och byte av agentidentitet

K11 (V31 avsnitt 7): *"Förtydliga H8/första PR som S0.0a, och sätt tidpunkt för byte av agentidentitet."*

| Del | Var | Text i v3.2 | Svar |
| --- | --- | --- | :-: |
| H8 är inte längre cirkulärt beroende av första PR | BP:100 | "inställningar bekräftade (H8, **efter S0.0a**)" | **OK** |
| S0.0a är ett eget namngivet steg | BP:101 | "**S0.0a – förberedande steg, före H8** [K7, K11, V3-04, V31-05]: en liten pull request med **enbart** `CODEOWNERS`, som ägaren mergar manuellt innan någon PR med workflows." | **OK** |
| H8 görs när filen finns | BP:101 | "H8 görs först när filen finns på `main`, och H8-beviset ska visa det." | **OK** |
| Tidpunkt för byte av agentidentitet | BP:101 | "direkt efter S0.0a och före första kod-PR skapas agentens egen identitet (GitHub-app eller maskinkonto), Required approvals sätts till 1 och Code Owner-kravet blir bindande. Från och med då kan ägaren godkänna agentens PR:er." | **OK** |

**Konsekvens med övrig text:**
- Ägarpunkt 1 (BP:43) kräver redan egen identitet, och ägarpunkt 8 (BP:50) har övergångsregeln "Required approvals = 0" under planeringsfasen. BP:101 anger nu när den fasen slutar.
- H8-raden i H-tabellen (BP:70) nämner inte S0.0a, men BP:100–101 gör sekvensen entydig och motsäger inget.
- BP:103 (CODEOWNERS skyddar sig själv och `.github/`) och BP:98 (S0.0 beroende av H2) är oförändrade.
- Planen anger inte vem som skapar identiteten. Ägarpunkt 1 och H2 (BP:64) täcker det som ägarens uppgift. Det är ingen avvikelse från K11.

---

## 6. Blueprinten: inga andra ändringar än ändringsloggen anger

Diffen v1.1 → bifogad blueprint har **10 block**. Varje block motsvaras av en loggrad:

| # | Nya rader | Ändring | Anges av |
| --- | --- | --- | --- |
| 1 | BL:8 | Versionsraden: `1.1` → `1.2 (se ändringslogg v1.1 → v1.2; …)` | Själva versionshöjningen |
| 2 | BL:21–32 | Nytt avsnitt "Ändringslogg v1.1 → v1.2" (12 rader) | Loggen själv |
| 3 | BL:74 | OB-1-raden | Loggrad 5 |
| 4 | BL:162 | §0B, hänvisning | Loggrad 2 |
| 5 | BL:328 | §7, hänvisning | Loggrad 3 |
| 6 | BL:374 | §9.1, `created_by` | Loggrad 4 |
| 7 | BL:1319–1320 | §67A punkt 2–3, hänvisning | Loggrad 1 |
| 8 | BL:1329 | §68, hänvisning | Loggrad 1 |
| 9 | BL:1403 | §74 S0.0-gate | Loggrad 1 |
| 10 | BL:1431 | §74 Stage 0 exit gate | Loggrad 1 |

- Inga andra rader skiljer sig från v1.1. Det gäller även rubriker (enda skillnaden är den nya loggrubriken på BL:21), tabeller, ADR-listan, byggordningen och alla sektionsnummer 0–117.
- Mot den mergade v1.2 (på `main`) skiljer sig den bifogade blueprinten på samma rader (utom BL:8 och loggrad 1–4, som redan fanns) och fyller i hänvisningarna samt loggrad 5. Förklaringen i BL:23 är utökad med meningen *"Där innebörden ändrats har brödtexten fått en kort hänvisning 'v1.2'."*
- Loggraderna 1–4 är oförändrade från den mergade v1.2.
- Logg v1.0 → v1.1 (BL:33–81) är oförändrad.

---

## 7. Anmärkningar (underkänner inte något ovan)

| # | Anmärkning | Var | Åtgärd (Architect) |
| --- | --- | --- | --- |
| 1 | **BUILD_PLAN avsnitt 14 anger blueprintens radnummer från före ändringen.** Raden K9 anger "BL:73, 161, 327, 373, 1318–1319, 1328, 1402, 1430". I den bifogade blueprinten är de rätta raderna 74, 162, 328, 374, 1319–1320, 1329, 1403, 1431. Varje hänvisning är en rad fel eftersom loggrad 5 lade till en rad. | BP:378 | Uppdatera radnumren, eller hänvisa till avsnitt (§0B, §7, §9.1, §67A, §68, §74) i stället för radnummer. |
| 2 | **Loggrad 5 säger "§0 OB-1", men OB-1-raden ligger inte i §0.** Den ligger i tabellen "Öppna beslut" (rubrik BL:70, raden på BL:74), medan §0 börjar på BL:82. Ändringen är rätt deklarerad, bara etiketten är oexakt. | BL:31 | Ändra etiketten till "Öppna beslut, OB-1". |
| 3 | **De verifierade filerna finns inte i repot.** `main` har BUILD_PLAN v3.1 (`3b3340f…`). Planens statusrad (BP:5) och `DECISIONS.md` (Underlag) hänvisar redan till v3.2. | – | Se "Current issue". |
| 4 | `DECISIONS.md`: den bifogade versionen ändrar enbart raden "Underlag" (BUILD_PLAN v3 → v3.2 och tillagda granskningsrapporter). Inget beslut, villkor (C1–C4) eller statusrad är ändrat. Underlaget i `main` nämnde inte heller `SECURITY_REVIEW_PLAN_V3.md`. | `DECISIONS.md` rad 9 | Inget. |

---

## 8. Spårbarhet för övriga ändringar i BUILD_PLAN v3.2

Eftersom K9–K11 är kontrollerade mot diffen har jag också mappat **varje** ändrat block v3.1 → v3.2 till en rad i planens avsnitt 14. Inga ändringar saknar förklaring.

| Block (v3.2-rader) | Ändring | Förklaras av (avsnitt 14) |
| --- | --- | --- |
| 1, 3, 5, 7 | Version, blueprint-referens, status, märkning | V31-04 |
| 41 | "Omärkta punkter gäller från S0.0" | V31-04 |
| 100–101 | S0.0a | K11 |
| 104 | Gate: "via readiness-gaten före första kunddata" | V31-04 |
| 131 | Gallring av `audit_log` | V31-03 |
| 133 | Bootstrap-test | V31-07 |
| 169, 176 | S0.7 alla tabeller | K10 |
| 184 | Kundvillkor flaggade i H9 | V31-06 |
| 212 | Ny task S1A.0b | V31-02 |
| 214–217 | Säkerhetstester i S1A.2–S1A.4, S1A.5-formulering | V31-04, V31-07 |
| 266 | 1F: gränssnittet finns från S1A.0b | V31-02 |
| 286–287 | 9B punkt 2–3 | V31-06, V31-02 |
| 317 | P7-pekare | V31-04 |
| 363 | K3-raden | V31-04 |
| 370–385 | Nytt avsnitt 14 | – |

**Vad jag inte har bedömt:** Om innehållet i V31-02–V31-07 är tillräckligt ur säkerhetssynpunkt (till exempel om S1A.0b:s testkrav räcker) är Security Reviewers uppgift. Jag har bara kontrollerat att ändringarna finns, är avgränsade och har en förklaring.

---

## 9. Status (AGENTS.md §4)

**Completed:**
- Kontroll av K9, K10 och K11 mot de bifogade filerna.
- Kontroll av att blueprinten inte har ändrats utöver ändringsloggen v1.2.
- Mappning av samtliga ändringar i BUILD_PLAN v3.1 → v3.2 till avsnitt 14.

**Verified (med bevis):**
- Radnivådiff mot v1.1, mot mergad v1.2 och v3.1, med SHA-256 för alla sju filer (avsnitt 2).
- Noll dolda tecken, noll CR-radslut och oförändrade rubriker och sektionsnummer i blueprinten.
- Noll träffar på `alla tabeller med` och `tabeller med company_id` i v3.2.
- K9: 4/4 OK. K10: OK. K11: OK. Blueprint utan andra ändringar än loggen anger: OK.

**Tests:** Inga automatiska tester finns eller har körts. Verifieringen är en textdiff och en manuell radläsning, vilket är rätt metod för dokument.

**Current issue:**
- De verifierade filerna är bifogade, **inte committade**. `main` har fortfarande BUILD_PLAN v3.1. Den här PR:en innehåller bara rapporten.
- Två mindre felaktigheter i dokumenten (anmärkning 1 och 2 i avsnitt 7).
- H8 är öppen: `CODEOWNERS` finns inte i repot ännu.
- Anthropics villkor och personuppgiftsavtal (`DECISIONS.md` C1) är inte kontrollerade.

**Next:**
1. Ägaren committar BUILD_PLAN v3.2, blueprint och `DECISIONS.md` (jämför SHA-256 mot avsnitt 2 innan merge) och rättar anmärkning 1–2 om så önskas.
2. Security Reviewer granskar v3.2 innan H1.
3. Ägaren beslutar H1 och genomför S0.0a och H8.
