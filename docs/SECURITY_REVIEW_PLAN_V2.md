# SECURITY REVIEW v2 – BUILD_PLAN.md (v2) mot SECURITY_REVIEW_PLAN.md

| Egenskap | Värde |
| --- | --- |
| **Roll** | Security Reviewer (`AGENTS.md` §3) |
| **Granskat** | `BUILD_PLAN.md` v2 (commit `4c6b0cb`, main `69da7ae`) |
| **Mot** | `docs/SECURITY_REVIEW_PLAN.md` (SEC-01–SEC-24, checklista P1–P16 i avsnitt 4) och `BUILD_PLAN.md` avsnitt 11 |
| **Bindande underlag** | `PROJECT_ARCHITECTURE_BLUEPRINT.md` v1.1, `AGENTS.md` |
| **Datum** | 2026-10-09 |
| **Omfattning** | Plangranskning. Ingen kod finns ännu. Allt nedan gäller vad planen *kräver*, inte vad som är byggt eller testat. |

> Rapporten ändrar varken kod, infrastruktur eller plan. Hänvisningar `BP:nn` avser radnummer i `BUILD_PLAN.md` v2.

---

## 1. Slutsats

**Rekommendation till H1: godkänn inte v2 som den står. En v3 behövs.** Planen har arbetat in merparten av förra rapportens fynd, och på de flesta ställen är lösningen konkret, till exempel rollmodellen i S0.3, SSRF-kraven i S1A.6 och route-matrisen i S0.4. Men tre brister är allvarliga nog att de bör åtgärdas före godkännande:

1. Planen kräver inte att **tabeller som tillkommer efter S0.3** får RLS och negativa tester (V2-01).
2. **Radering** saknar task, schemaläggning och verifiering (V2-02).
3. Planen förutsätter att **CODEOWNERS** skyddar säkerhetsfilerna, men kräver inte inställningen som gör det verkligt (V2-03).

Dessutom avviker planen från blueprinten på två punkter utan ADR: att skjuta upp production, och att skapa E2E-konton via admin-API. Båda är BLOCKER enligt `AGENTS.md` (konfliktregeln överst och rapportformatet i §4).

| Resultat | Antal |
| --- | :-: |
| Checklista P1–P16: **JA** | 8 |
| Checklista P1–P16: **DELVIS** | 8 |
| Checklista P1–P16: **NEJ** | 0 |
| Tidigare fynd SEC-01–SEC-24: åtgärdade i planen | 13 |
| Tidigare fynd SEC-01–SEC-24: delvis åtgärdade | 11 |
| Nya fynd (V2-nn): HÖG / MEDEL / LÅG | 3 / 10 / 4 |

`AGENTS.md` anger att Security Reviewer är klar först när alla CRITICAL/HIGH-fynd är åtgärdade **och återverifierade**. Det gäller inte än. Av förra rapportens åtta HÖG-fynd är fyra bara delvis åtgärdade (SEC-04, SEC-06, SEC-08, SEC-10), och tre nya HÖG-fynd tillkommer. De tre KRITISKA (SEC-01–03) är åtgärdade **i planen**. Återverifiering av implementationen sker vid Stage 0 exit gate.

**Märkning:** **[Plan]** = står i BUILD_PLAN. **[Känt]** = etablerat beteende hos verktyg/plattform. **[Verifiera]** = troligt men inte kontrollerat mot aktuell dokumentation. Skala: *HÖG* = sannolik läcka eller allvarlig GDPR-risk, åtgärdas före H1 eller i den etapp funktionen byggs. *MEDEL* = försvagar skyddet eller spårbarheten. *LÅG* = hygien.

---

## 2. Svar per punkt P1–P16

| # | Krav (avsnitt 4) | Svar | Task-ref | Kort motivering |
| --- | --- | :-: | --- | --- |
| P1 | Negativa säkerhetstester i varje task | **DELVIS** | Avsnitt 3–8 | Stage 0 är bra. 11 av 24 task/etapper saknar negativt test, bland annat S1A.1, 1B och 1C. Se 2.1. |
| P2 | OB-1, OB-5, OB-6 beslutade före S0.0 | **DELVIS** | Avsnitt 0, BP:22, 32 | OB-1 motsäger sig själv i planen. OB-5 och OB-6 har standardvärden, men inget beslut. Se 2.2. |
| P3 | Roller, icke-exponerat schema, REVOKE, fail-closed | **JA** | S0.3 (BP:112–117) | Alla delar finns och gaten testar dem. Följdfynd V2-01 och V2-07. |
| P4 | Route × roll × tenant-matris, rollregler, auktorisering från databas | **JA** | S0.4 (BP:123–131) | Alla delar finns. Matrisens dimensioner är för snäva (V2-08). |
| P5 | Servergenererad nyckel, TTL, storlek/checksumma, lifecycle | **JA** | S0.5 (BP:136–138) | Alla delar finns. Storlek kontrolleras först efteråt (V2-15, LÅG). |
| P6 | Worker utan BYPASSRLS, tenant från `jobs.company_id` | **JA** | S0.6 (BP:143–145) | Alla delar finns. Hur `app_deleter` anropas är oklart (V2-09). |
| P7 | Parsergränser, `defusedxml`, `.xlsm`, fientliga fixtures | **DELVIS** | S1A.2–S1A.5 (BP:158–161) | Gränser, `defusedxml` och `.xlsm`-avvisning finns. **Fientliga fixtures finns inte i någon task eller gate**, bara i checklistan (BP:248). |
| P8 | SSRF per hopp, IP-validering, rebinding, negativa tester | **JA** | S1A.6 (BP:162) | Alla delar finns, inklusive alla sex negativa fall jag föreslog. |
| P9 | Tool-scheman utan tenant-argument, injektionstester | **DELVIS** | S1A.8, 1F (BP:164, 208) | Reglerna finns, men ingen gate kräver att injektionstesterna *passerar*. Se 2.3. |
| P10 | ReportLab låst och escapad | **JA** | 1G (BP:209) | Version, escaping, fientligt testrört och kort signerad URL finns. Automatisk dependency-skanning saknas (V2-05). |
| P11 | Loggredaktor med allowlist och kanarietest | **JA** | S0.1 (BP:98–99) | Allowlist, förbjudna fält och `log-canary-test` i gaten. Ordningsfel mot S0.2 (V2-16). |
| P12 | Branch protection, CODEOWNERS, production-godkännare, secret scanning | **DELVIS** | S0.0, BP:48–50 | Alla fyra nämns, men inget gör CODEOWNERS verksam och inget verifierar inställningarna. Se 2.4. |
| P13 | Radering täcker org, användare, storage-prefix | **DELVIS** | Avsnitt 9A (BP:217) | Kraven är rätt skrivna, men ingen task, ingen gate och ingen verifiering. V2-02. |
| P14 | Auditlogg: endast INSERT, ingen cascade, definierat schema | **JA** | S0.3 (BP:116) | Alla tre delar finns. Händelsekatalog och retention saknas (V2-17, LÅG). |
| P15 | E2E-autentisering utan bypass | **DELVIS** | Avsnitt 9A (BP:218) | Innehållet är rätt, men det strider mot blueprint §0B:s ordalydelse och saknar ADR, ägande task och skydd mot production. V2-13. |
| P16 | Inga localhost, docker-compose, mock eller seed i production | **DELVIS** | Bypass-scan, S0.1 | Mönsterskanning finns. Ingen kontroll att `fixtures/`/`tests/` inte hamnar i containern, och `localhost`-regeln ger falska träffar som lockar till undantag (V2-14). |

### 2.1 P1 – task för task

| Task/etapp | Negativt säkerhetstest i planen? |
| --- | :-: |
| S0.1, S0.2, S0.3, S0.4, S0.5, S0.6 | Ja |
| S1A.6 (SSRF), S1A.8 (injektion), 1G (fientlig sträng) | Ja |
| S1A.2, S1A.3, S1A.4 (gränser nämns, inga tester/fixtures) | Delvis |
| 1F (injektionsfall nämns, ingen pass-gräns) | Delvis |
| S0.0 (kontroller är konfiguration, inget som visar att de är på) | Nej |
| S1A.1 (ogiltiga tillståndsövergångar; fil registrerad mot annat bolags import) | Nej |
| S1A.5 (dokument med injektionstext; personnummer slipper maskning) | Nej |
| S1A.7, S1A.10 | Nej |
| S1A.9 (viewer försöker bekräfta mappning) | Nej |
| 1B (inga RLS-/tenant-test för de nya tabellerna) | Nej |
| 1C, 1D, 1E (t.ex. "scenarier skriver aldrig till riktig data" har inget test) | Nej |
| 1H | Nej |

Avsnitt 3 (BP:77) säger bara "auktorisering och tenant-isolering är testad". Det är exakt formuleringen P1 sade inte räcker.
**Åtgärd:** Lägg till en rad **Säkerhetstester** i varje task, med minst ett negativt fall (fel roll, fel tenant, fientlig indata, ogiltigt tillstånd). Ytterligare krav i V2-01.

### 2.2 P2 – beslut före S0.0

- **OB-1 motsäger sig själv:** Tabellen säger *"Väljs vid S1F.2"* (BP:22). Texten under säger att OB-1 *"ska beslutas före S0.0"* (BP:32), och H5 (BP:66) säger *före S1A.8*. Tre olika tidpunkter.
- **Nyckeln behövs tidigare än planen anger:** Ägarpunkt 5 (BP:46) säger att AI-nyckeln behövs först vid S1A.8, men S1A.5 (bild/dokument-extraktion via AI) kommer före.
- **OB-6** står i tabellen med ett standardvärde, men BP:32 nämner bara OB-1 och OB-5 som "före S0.0".
- **OB-5** har ett standardvärde och punkt 10 (BP:51) ber ägaren besluta, men det finns ingen punkt som stoppar S0.0 tills det är gjort. Att `S0.0` har H2 som beroende (BP:90) räcker inte, eftersom H2 handlar om konton och hemligheter.
- **Åtgärd:** En enda tidpunkt för OB-1 (före S0.0), med DPA-/träningskraven i själva tabellraden. Lägg till OB-1, OB-5 och OB-6 som uttryckliga beroenden i S0.0, och ändra H5 och ägarpunkt 5 så att de gäller S1A.5.

### 2.3 P9 – injektion

- 1F:s enda gate (BP:208) är en funktionell fråga ("Why did my margin fall?") och att evals är *sparade*. Ingen gräns för injektionsfall.
- S1A.8 (BP:164) säger "Injektionstester" men inte vad som räknas som godkänt.
- Saknas i S1A.5: dokumentinnehåll går till en LLM utan skydd som motsvarar S1A.8.
- **Åtgärd:**
  - Ett deterministiskt CI-test som läser tool-registrets scheman och fallerar om något tool har argument som heter `company_id`, `organization_id` eller liknande.
  - Ett injektionsset med förväntat utfall (t.ex. noll tool-anrop mot annat företag, noll externa länkar i svar) som måste passera i 1F-gaten, inte bara sparas.
  - Samma set, anpassat, för S1A.5.
  - Chattrådar "privata per användare" (OB-8) ska ha ett eget test.

### 2.4 P12 – branch protection och CODEOWNERS

- Ägarpunkt 8 (BP:49) säger "PR krävs, CI krävs, inga direkt-pushar", men **inte** "kräv granskning från Code Owners". [Känt] Utan den inställningen är en CODEOWNERS-fil bara en lista. `AGENTS.md` regel 11 vilar på att den är verksam.
- Ägarpunkt 1 (BP:42) ger agenten en token med `Workflows` write. Om det är *ägarens* konto som agenten använder kan ägaren inte godkänna sin egen PR. [Känt] GitHub tillåter inte självgodkännande. Den praktiska följden är att regeln stängs av eller kringgås.
- Inget steg i S0.0 eller H2 verifierar att inställningarna faktiskt är på. `AGENTS.md` förbjuder att markera en gate godkänd utan att ha kört den, och det gäller även här.
- **Åtgärd:** Se V2-03.

---

## 3. Kontroll av BUILD_PLAN avsnitt 11

Avsnitt 11 (BP:238–257) är en tabell över kraven med kolumnen "Besvaras av". Den återger P1–P16 troget, men:

- Den anger **var** planen påstås besvara kravet, inte om den gör det. Den ska därför inte tolkas som statusrapport. Min bedömning är i avsnitt 2.
- P1 är formulerad svagare än i min rapport. Mitt krav var ett eget fält **Säkerhetskontroller** med *konkreta negativa tester* i varje task. I avsnitt 11 står "Varje task har negativa säkerhetstester". Skillnaden spelar roll eftersom det är fältet som gör att ingen task glöms bort.
- Två rader pekar på avsnitt 9A (P13, P15). Det är ett fristående avsnitt utan task, filer, tester eller gate, och det är där de svagaste svaren finns.
- Avsnitt 11 saknar kolumn för *bevis* (CI-körning, testnamn). Lägg till en sådan så att Verifier kan fylla i den vid varje gate.

---

## 4. Täckning av tidigare fynd SEC-01–SEC-24

| SEC | Allv. | Status | Var i planen | Kvar |
| --- | :-: | :-: | --- | --- |
| 01 Data API kringgår backend | KRIT | **Åtgärdad** | S0.3 (BP:113, 117) | Ägarpunkt 11 (BP:52) är bra hygien, men ersätter inte S0.3. |
| 02 RLS-rollmodell | KRIT | **Åtgärdad** | S0.3, S0.6 | V2-07, V2-09 |
| 03 IDOR och rolleskalering | KRIT | **Åtgärdad** | S0.4 | V2-08 |
| 04 Prompt injection | HÖG | **Delvis** | S1A.8, 1F | Se P9 |
| 05 Signerad uppladdning | HÖG | **Åtgärdad** | S0.5 | V2-15 (LÅG) |
| 06 Filhantering och malware | HÖG | **Delvis** | OB-6, S1A.2–4 | V2-06 |
| 07 Sheets/SSRF | HÖG | **Åtgärdad** | S1A.6 | – |
| 08 Dokument till AI, personnummer | HÖG | **Delvis** | OB-1/5, S1A.5, S1A.8, 9A | V2-11, V2-12 |
| 09 Loggar | HÖG | **Åtgärdad** | S0.1 | Parserfel som citerar celler, retention hos plattformar (V2-17) |
| 10 Process och auto-deploy | HÖG | **Delvis** | S0.0, BP:48–50 | V2-03 |
| 11 ReportLab | HÖG | **Åtgärdad** | 1G | V2-05 |
| 12 §65 konkretisering | MEDEL | **Delvis** | S0.4 | Skydd mot kontouppräkning, var token lagras, var och på vilka nivåer rate limiting sker (V2-17) |
| 13 Auditlogg | MEDEL | **Delvis** | S0.3 | Händelsekatalog, retention, inloggningar (V2-17) |
| 14 Radering | MEDEL | **Delvis** | 9A | V2-02 |
| 15 Bypass-scan | MEDEL | **Delvis** | S0.1, BP:50 | Ingen secret-skanning i CI (V2-05), V2-14 |
| 16 E2E utan bypass | MEDEL | **Delvis** | 9A | V2-13 |
| 17 CI och leveranskedja | MEDEL | **Delvis** | S0.0, S0.1 | Frontend-lockfile nämns inte, bara Python (BP:96) |
| 18 Operatörsroll | MEDEL | **Delvis** | OB-9, 9A | V2-02 (ingen task, tabell, MFA-mekanism) |
| 19 Starkare gates | MEDEL | **Åtgärdad** | Stage 0 exit gate (BP:148) | – |
| 20 Jobb-payload | MEDEL | **Åtgärdad** | S0.6 | – |
| 21 Rapport-URL | LÅG | **Åtgärdad** | S0.5, 1G | – |
| 22 Säkerhetsheaders | LÅG | **Åtgärdad** | S0.4 | – |
| 23 Felsvar | LÅG | **Åtgärdad** | S0.4 (404) | – |
| 24 Production → staging | LÅG | **Åtgärdad** | 9A | – |

---

## 5. Nya fynd (V2-nn), sorterade efter allvarlighet

### HÖG

#### V2-01 – RLS och negativa tester gäller bara de sju tabellerna i S0.3

- **Observation:** [Plan] S0.3 skapar policyer för sju tabeller (BP:109). Stage 1A–1H lägger till närmare trettio tenant-tabeller (blueprint §10–§13A) (importer, produkter, försäljning, findings, chatt, rapporter m.fl.). Ingen task kräver att de får RLS, policy eller negativt test, och route-matrisen testar bara API:t, inte databasskiktet.
- **Risk:** Tabellen `sales_lines` skapas i 1B utan RLS. `app_api` har grants. En buggig repository-fråga som glömmer `company_id` läcker då mellan kunder, och det tredje försvarslagret (§51) är tyst frånvarande. Ingen gate märker det.
- **Åtgärd:**
  1. Ett katalogdrivet CI-test mot `pg_catalog`: varje tabell i app-schemat med `company_id` eller `organization_id` ska ha RLS påslaget, `FORCE ROW LEVEL SECURITY`, minst en policy, och inga grants till `anon`, `authenticated` eller `PUBLIC`.
  2. Samma uppräkning genererar ett cross-tenant-test per tabell.
  3. Lägg testet i avsnitt 3 (gemensamma gates) så att det gäller varje task, och i Stage 0 exit gate.
  4. Data API-testet i S0.3 ska också räkna upp tabeller dynamiskt, inte en fast lista.

#### V2-02 – Radering och operatörsroll saknar task, schemaläggning och verifiering

- **Observation:** Avsnitt 9A (BP:215–221) har inget stage, inga filer, inga tester och ingen gate. Blueprintens byggordning (§74–§82) har heller ingen raderingstask. Det som uttryckligen saknas:
  - Verifieringssteget efter radering (blueprint §66, tillägg): bekräfta att inga rader eller objekt med företagets id finns kvar och logga resultatet.
  - Vem som utlöser den fördröjda permanenta raderingen efter 7 dagar (OB-7). Det finns ingen schemalagd rensare.
  - Hur raderingen hålls komplett när Stage 1B–1G lägger till tabeller: ingen task kräver att nya tabeller registreras.
  - Radering av användarkonto hos auth-leverantören, och av organisation.
  - Operatörsrollen (SEC-18): ingen tabell, inget MFA-krav, ingen task. `feature_flags` skapas i S0.3 men ingen vet vem som får skriva globala flaggor.
- **Risk:** GDPR-skyldighet (blueprint §0 punkt 13) som inte byggs, eller byggs utan att någon kan bevisa att den fungerar. Radering som tyst missar nya tabeller är sannolik utan täckningskontroll.
- **Åtgärd:**
  - Skapa en uttrycklig task, t.ex. **S0.7 Radering** (eller ett eget avsnitt före Stage 1A), med mål, filer, tester och gate. Minst: `DELETE_COMPANY_DATA`, 7-dagarsrensare, verifieringsjobb, radering av storage per prefix och av auth-användare.
  - CI-test: varje tabell med `company_id` måste finnas i raderingsregistret (samma kataloguppräkning som V2-01).
  - Lägg raderingstest i definition of done för varje task som skapar tabeller.
  - Operatörsroll: beskriv roll, MFA-krav, tidsbegränsning och audit i en task eller skjut upp global flaggskrivning tills den finns, och dölj funktionen tills dess (§0B).

#### V2-03 – CODEOWNERS är verkningslöst utan de inställningar som gör det verkligt

- **Observation:** Se 2.4. Planen kräver `CODEOWNERS` (BP:91) och branch protection (BP:49) men inte Code Owner-granskning, inte separat identitet för agenten, och ingen verifiering.
- **Risk:** `AGENTS.md` regel 11 ("försvaga aldrig ett säkerhetstest") och SEC-10 bygger på att en människa ser ändringar i workflows, migrationer och RLS. Utan inställningen kan agenten ändra ett test för att få en gate grön och sedan merga själv, vilket `AGENTS.md` §2 steg 7 tillåter när CI är grön.
- **Åtgärd:**
  - Lägg i ägarpunkt 8: *"Require review from Code Owners"*, *"Do not allow bypassing"* (inklusive administratörer), och förbjud force-push.
  - Agenten använder en **egen GitHub-identitet** (app eller maskinkonto), inte ägarens personliga token.
  - Skydda även `CODEOWNERS`-filen själv och `.github/` i sin egen regel.
  - Lägg ett verifieringssteg i S0.0/H2: ägaren bekräftar (skärmdump eller API-svar) att inställningarna är på, och Verifier noterar det som bevis.
  - [Verifiera] Om repot är privat kan GitHub secret scanning kräva betald plan. Ange då en fallback (t.ex. `gitleaks` i CI).

### MEDEL

#### V2-04 – Production skjuts upp utan ADR (BLOCKER A)

- **Observation:** Blueprint §74 (S0.0-gaten och Stage 0 exit gate), §67A och §68 kräver staging **och** production. Planen skjuter production till "före första pilotkund" (BP:34, 92, 148). Ägarpunkt 2 (BP:44) kräver samtidigt två Supabase-projekt. H3 (BP:64) ber ägaren godkänna en production-deploy av en miljö som inte finns.
- **Risk:**
  - Säkerhetskonfigurationen för production (separata hemligheter, miljögodkännande, roller, Data API-test, backup/restore) körs första gången just före riktiga kunddata.
  - Fel som bara finns i production upptäcks sent.
  - [Verifiera] Staging ligger på en gratisplan; begränsningar för backup och pausning ska kontrolleras. Staging får ändå inte innehålla riktiga kunddata (§68).
- **Åtgärd:** Antingen skriv en ADR som ändrar blueprint §74, eller återställ kravet. Rekommendation: ADR plus en uttrycklig **Production readiness-gate** som upprepar Stage 0 exit gate-testerna mot production, och som måste vara grön innan *någon* kunddata laddas upp.

```text
BLOCKER A
Expected architecture: Blueprint §74 kräver S0.0/Stage 0 exit gate "Deployed in staging
  and production PASS". §67A/§68 kräver två miljöer.
Observed conflict: BUILD_PLAN v2 (BP:34, 92, 148) skjuter production till före pilot.
  Ägarpunkt 2 och H3 antar att production finns.
Possible solutions: a) skapa production nu; b) ADR som flyttar kravet, med
  production readiness-gate före första kunddata; c) lämna som det är.
Recommended solution: (b). Lägg till ADR-015 och en readiness-gate som upprepar
  säkerhetstesterna i Stage 0 exit gate mot production.
```

#### V2-05 – Dependency-, secret- och containerskanning saknas i CI-gaterna

- **Observation:** Blueprint §65 och §105 kräver "dependency scanning" och "security scan". CI-listorna i BP:79 och BP:99 innehåller varken dependency-skanning, secret-skanning eller containerskanning. `AGENTS.md` ber Security Reviewer kontrollera "dependency-scan".
- **Risk:** Ett låst ReportLab-krav (BP:209) hjälper bara tills nästa sårbarhet. En hemlighet som av misstag checkas in upptäcks först om push protection är på och verkar.
- **Åtgärd:** Lägg i S0.1-gaten: dependency-audit för Python och frontend, secret-skanning i CI, containerskanning, och krav på lockfil även för frontend. Låt gaten fallera på HIGH/CRITICAL utan dokumenterat undantag. Undantagslistan ska ligga under CODEOWNERS.

#### V2-06 – Filvalidering (SCANNING) saknar ägare, och fientliga fixtures är inte planerade

- **Observation:** OB-6 (BP:27) ersätter malware-skanning med "hård validering", men ingen task bygger den. S1A.1 är bara state machine och domän. Blueprint §56 kräver MIME-, filändelse- och innehållsvalidering (inte klientens deklarerade typ), filnamnssanering, sidgräns och bildmåttsgräns. Ordet "fixtures" förekommer bara i checklistan (BP:248).
- **Risk:** Kompensationen för att malware-skanning saknas finns inte. Zip-bomber, XXE, pixelbomber och ändelse/innehåll-avvikelser testas inte.
- **Åtgärd:** Ge SCANNING en egen task (före S1A.2) med magic-byte-kontroll, ändelse/MIME-matchning, `.xlsm` och andra makroformat avvisade, gränser tvingade i workern. Lägg fixtures för zip-bomb, XXE, pixelbomb och fel filtyp i `tests/` som uttrycklig gate för S1A.2–S1A.5. Överväg att inte stödja äldre `.xls` i Stage 1 om parsern inte kan sandlådas, eftersom det är en äldre parseryta (min bedömning, inte verifierad).

#### V2-07 – RLS för organisationsnivå ospecificerat

- **Observation:** [Plan] Policyerna bygger på `SET LOCAL app.current_company_id` (BP:109). `organizations`, `organization_memberships` och `company_access` har inget `company_id`. Blueprint §51 nämner "(och organisation)", men planen tappade det.
- **Risk:** `GET /organizations`, `POST /organizations` (första organisationen, innan något företag finns) och medlemslistor har ingen definierad RLS-modell. Antingen står de utan RLS-skydd, eller så löses det med en bred roll eller bypass.
- **Åtgärd:** Specificera en andra variabel (`app.current_user_id`) och policyer för medlemskapstabellerna, samt en definierad väg för att skapa en organisation (t.ex. policy som tillåter insert när `created_by = current_user_id`). Testa fail-closed även här.

#### V2-08 – `company_access` genomdrivs inte, och matrisen saknar dimensioner

- **Observation:** `company_access` skapas i S0.3 (BP:109) men S0.4 nämner den inte (BP:124). Matrisen (BP:125) är "roll × egen/främmande tenant".
- **Risk:** I agency-läge (§93) ser en begränsad medlem alla organisationens företag. Chattrådar som ska vara privata per användare (OB-8) kan läsas av andra medlemmar i samma företag, eftersom matrisen inte har dimensionen "samma tenant, annan användare".
- **Åtgärd:** Utöka matrisen med (a) medlem begränsad via `company_access`, (b) annan användare i samma företag. Lägg till rollen i auktoriseringsfunktionen och ett test för vardera.

#### V2-09 – Workern är den mest utsatta komponenten, men hur `app_deleter` används är oklart

- **Observation:** Workern tolkar fientliga filer, anropar AI och hämtar externa URL:er, men har enligt planen bara `app_worker` (BP:143). `app_deleter` (BP:114) är "endast DELETE_COMPANY_DATA", men planen anger inte i vilken process den körs. Ägarlistan nämner bara en API-tjänst och en worker (BP:44).
- **Risk:** Om workern även får deleter-uppgifter, ger en komprometterad parser möjlighet att radera data över tenanter. Om det inte specificeras väljer bygg-AI den bekvämaste lösningen.
- **Åtgärd:** Kör `DELETE_COMPANY_DATA` i en separat process eller tjänst med egna uppgifter, eller dokumentera i en ADR hur två anslutningar hålls isär. Ge workern minsta möjliga R2-behörighet och inga uppgifter den inte använder.

#### V2-10 – Ingen säkerhetsgranskning efter Stage 1A

- **Observation:** Avsnitt 9 (BP:226) kör Security Reviewer efter Stage 0, 1C, 1F och före release. Stage 1A och 1B går live i staging utan granskning, och 1A innehåller parsrar, SSRF, AI-extraktion och uppladdning, alltså den största attackytan.
- **Åtgärd:** Lägg Security Reviewer efter Stage 1A (före 1B) och efter 1G (rapportgenerering).

#### V2-11 – OB-1 motsäger sig och AI används före nyckeln

- **Observation:** Se 2.2. Dessutom gäller att DPA-/träningskraven står i en löptext (BP:32) och inte som kriterier i tabellraden.
- **Åtgärd:** Se 2.2. Skriv in kriterierna (DPA, ingen träning på kunddata, retention, EU-behandling) som urvalskrav i OB-1-raden.

#### V2-12 – GDPR-dokumentation från SEC-08 och SEC-14 har tappats

- **Observation:** Underbiträdeslista i `SECURITY.md`, retention hos AI-leverantör och plattformar (loggar, backuper) i integritetsinformationen, och hantering av HMAC-nyckeln för `customer_reference_hash` (BP:221) saknas. Planen säger "hemlig nyckel per företag", men inte var den förvaras, hur den roteras, eller att den förstörs vid radering.
- **Åtgärd:** Lägg till i S0.1:s dokumentskelett (BP:96) och i raderingstasken (V2-02). Förvara nyckeln i secret store eller KMS, inte bredvid datan.

#### V2-13 – E2E-konton skapas på ett sätt som blueprint §0B inte tillåter (BLOCKER B)

- **Observation:** Blueprint §0B: testkonton ska vara "skapade genom den vanliga registreringen". Avsnitt 9A (BP:218) skapar dem via auth-leverantörens admin-API. Det var min rekommendation i SEC-16, men den kräver ändå ADR eftersom blueprinten säger något annat. Dessutom: ingen task bygger testhjälparen, och "fungerar aldrig mot production" är ett påstående utan mekanism.
- **Åtgärd:** Skriv ADR som ändrar §0B för testinfrastruktur. Hemligheten (service role för staging) ska bara exponeras för E2E-jobbet, inte för PR-jobb från forks. Lägg ett test som visar att hjälparen vägrar köra om miljön är production. Hjälparen får inte importeras från `src/` och ska gå igenom bypass-scan med uttryckligt undantag under CODEOWNERS.

```text
BLOCKER B
Expected architecture: Blueprint §0B: testkonton skapas genom den vanliga registreringen.
Observed conflict: BUILD_PLAN 9A skapar dem via auth-leverantörens admin-API.
Possible solutions: a) ordinarie registrering med brevlåda som testet styr;
  b) admin-API för testinfrastruktur, dokumenterat i ADR;
  c) behåll som planen står.
Recommended solution: (b) med ADR, produktionsvakt i hjälparen och hemlighet
  begränsad till staging.
```

### LÅG

#### V2-14 – Bypass-scan: falska träffar och innehåll i containern

`localhost` förekommer legitimt i CI-service-containrar, i `HEALTHCHECK` i `Dockerfile` och i Vite-konfig. Skanning av `infra/`, `scripts/`, `.github/workflows/` och `Dockerfile` (BP:98) ger då falska träffar, och det frestar till att utesluta mappar. **Åtgärd:** Undantag i en fil under CODEOWNERS med motivering per rad. Lägg `.dockerignore` som utesluter `fixtures/`, `tests/` och `docs/`, och ett CI-test som visar att imagen inte innehåller dem (blueprint §48: fixtures är "endast för tester", §0B: inga golden datasets i production).

#### V2-15 – Storlek kontrolleras först efter uppladdning

Servern gör `HEAD` vid registrering (BP:136). Ett stort objekt hinner ligga i bucketen innan dess. **Åtgärd:** Radera objektet om gränsen överskrids. Verifiera mot R2:s dokumentation om signerade URL:er kan begränsa längd och typ [Verifiera]. Lägg regel som avbryter ofullständiga multipart-uppladdningar.

#### V2-16 – Ordningsberoenden

S0.1-gaten kräver `log-canary-test` (BP:99), men `core/logging.py` listas under S0.2 (BP:103). S0.3-gaten kräver ett testkonto mot Data API (BP:117), men hjälparen för testkonton ligger i avsnitt 9A utan task. **Åtgärd:** Flytta loggmodulen till S0.1, och låt testkontohjälparen vara en del av S0.3.

#### V2-17 – Rester av SEC-12 och SEC-13

Skydd mot kontouppräkning vid registrering/återställning; var token lagras i frontend; var och på vilka nivåer rate limiting görs; auditloggens händelsekatalog (misslyckade inloggningar, utfärdade signerade URL:er, mappningsbekräftelse, flaggändringar, raderingsverifiering) och retention; hur inloggningar loggas när de sker hos auth-leverantören; parserfel som citerar celler i förbjudna loggfält. **Åtgärd:** Lägg som krav i S0.4 respektive S0.3.

---

## 6. Åtgärdslista

### Före H1 (ska in i v3)

| Åtgärd | Fynd |
| --- | --- |
| Katalogdrivet RLS-täckningstest i gemensamma gates och Stage 0 exit gate | V2-01 |
| Raderingstask med schemaläggare, verifiering och täckningstest; operatörsroll definierad eller dold | V2-02 |
| Code Owner-krav, egen agentidentitet, verifiering av inställningar | V2-03 |
| ADR för production-uppskjutning eller återställt krav | V2-04 (BLOCKER A) |
| ADR för E2E-konton | V2-13 (BLOCKER B) |
| Ett enda beslutstillfälle för OB-1/OB-5/OB-6 som beroende i S0.0 | P2, V2-11 |
| Fält **Säkerhetstester** i varje task | P1 |

### Före respektive task

| Task | Åtgärd | Fynd |
| --- | --- | --- |
| S0.1 | Dependency-/secret-/containerskanning, frontend-lockfil, loggmodul flyttad hit | V2-05, V2-16 |
| S0.3 | Org-nivå RLS, bootstrap av första organisation, audit-händelsekatalog | V2-07, V2-17 |
| S0.4 | `company_access`, utökad matris, kontouppräkning, tokenlagring | V2-08, V2-17 |
| S0.6 | Separat process för `app_deleter` | V2-09 |
| Före S1A.2 | SCANNING-task och fientliga fixtures | V2-06, P7 |
| S1A.5, S1A.8, 1F | Injektionsset med pass-gräns, schematest för tool-argument | P9 |
| Efter 1A och 1G | Security Reviewer-checkpoint | V2-10 |
| S0.1 / 9A | `.dockerignore`, undantagsfil för bypass-scan | V2-14 |
| S0.1, raderingstask | Underbiträdeslista, HMAC-nyckelhantering | V2-12 |

---

## 7. Status (AGENTS.md §4)

**Completed:** Granskning av `BUILD_PLAN.md` v2 mot checklistan P1–P16 och SEC-01–SEC-24. 17 nya fynd (3 HÖG, 10 MEDEL, 4 LÅG) och två BLOCKER.

**Verified:** Alla hänvisningar till rader i planen och blueprinten är kontrollerade mot filerna. Frånvaro av dependency-skanning, Code Owner-krav, `.dockerignore`, DPA-krav i `SECURITY.md`, MIME/magic-byte-validering och raderingsverifiering är kontrollerad med sökning i planen. Att blueprintens byggordning saknar raderingstask är kontrollerat. Påståenden märkta **[Verifiera]** är inte kontrollerade mot leverantörsdokumentation.

**Tests:** Ingen kod finns, så inga tester har körts. Alla bedömningar gäller plantext.

**Current issue:** BLOCKER A (production uppskjuten) och BLOCKER B (E2E-konton) kräver ADR. V2-01, V2-02 och V2-03 bör in i v3 före H1.

**Next:** Architect skriver v3 med åtgärdslistan i avsnitt 6. Security Reviewer granskar v3 mot V2-01–V2-17 och återverifierar CRITICAL/HIGH vid Stage 0 exit gate.
