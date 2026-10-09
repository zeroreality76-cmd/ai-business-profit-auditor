# SECURITY REVIEW v3 – BUILD_PLAN.md (v3) mot SECURITY_REVIEW_PLAN_V2.md

| Egenskap | Värde |
| --- | --- |
| **Roll** | Security Reviewer (`AGENTS.md` §3) |
| **Granskat** | `BUILD_PLAN.md` v3, `docs/adr/ADR-015-production-deferred.md`, `docs/adr/ADR-016-e2e-test-accounts.md`, `docs/DECISIONS.md` (main `9282ad5`, efter PR #4) |
| **Mot** | `docs/SECURITY_REVIEW_PLAN_V2.md`: avsnitt 6 (åtgärdslistan), V2-01–V2-17, BLOCKER A och B |
| **Datum** | 2026-10-09 |
| **Omfattning** | Plangranskning. Det finns ännu ingen kod. Allt gäller vad planen *kräver*, inte vad som är byggt eller testat. |

> Rapporten ändrar varken kod, infrastruktur eller plan. `BP:nn` avser radnummer i `BUILD_PLAN.md` v3.

---

## 1. Slutsats

**Rekommendation till H1: godkänn v3 under villkor att en v3.1 rättar punkterna K1–K5 (avsnitt 7) innan S0.0 startar.** Inget av dem kräver omtag av arkitekturen.

Alla tre HÖG-fynd från förra omgången (V2-01, V2-02, V2-03) är åtgärdade i planen. Båda BLOCKER är lösta genom ägarens godkännande av ADR-015 och ADR-016. Kvar är rättelser i planens text och några testkrav som ännu inte är schemalagda. Det finns inga öppna KRITISKA eller HÖGA fynd på plannivå, och inga nya HÖGA.

| Resultat | Antal |
| --- | :-: |
| V2-01–V2-17: **ÅTGÄRDAD** | 14 |
| V2-01–V2-17: **DELVIS** | 3 (V2-04, V2-13, V2-17) |
| V2-01–V2-17: **EJ ÅTGÄRDAD** | 0 |
| BLOCKER A och B | Båda **lösta** (ADR godkända) |
| Åtgärdslistan (V2 avsnitt 6), före H1: ÅTGÄRDAD / DELVIS | 4 / 3 |
| Nya fynd (V3-nn): MEDEL / LÅG | 2 / 4 |

**Vad "ÅTGÄRDAD" betyder här:** planen kräver det jag föreslog, med task och gate. Det är **inte** ett bevis att något är byggt eller fungerar. `AGENTS.md` kräver att CRITICAL/HIGH är "åtgärdade och återverifierade". Återverifieringen sker mot implementationen vid Stage 0 exit gate och i Production readiness-gaten (BP:180).

**Märkning:** **[Plan]** = står i planen. **[Observerat]** = kontrollerat i repot eller via GitHub-API vid granskningen. **[Verifiera]** = troligt men inte kontrollerat mot aktuell dokumentation.

**Observerat vid granskningen:** `main` har `protected: true` (GitHub API). Det syns inte vilka regler som gäller, och det finns varken `.github/` eller `CODEOWNERS` i repot. H8 (BP:68) är därför fortfarande öppen och Code Owner-kravet är ännu utan verkan.

---

## 2. Resultat per fynd V2-01–V2-17

| Fynd | Allv. | Svar | Task-ref | Motivering |
| --- | :-: | :-: | --- | --- |
| V2-01 RLS-täckning | HÖG | **ÅTGÄRDAD** | Avsnitt 3 (BP:85), S0.3 (BP:128–129), Stage 0 exit gate (BP:175) | Katalogtest: RLS, `FORCE`, minst en policy, inga grants till `anon`/`authenticated`/`PUBLIC`, dynamisk uppräkning, cross-tenant-test per tabell. Data API-testet räknar också upp dynamiskt. **Se V3-02:** urvalet är kolumnbaserat, vilket jag själv föreslog, och släpper igenom tabeller utan `company_id`. |
| V2-02 Radering och operatör | HÖG | **ÅTGÄRDAD** | S0.7 (BP:162–172), Avsnitt 3 (BP:86), Exit gate (BP:175) | Task med register, 7-dagarsrensare, verifieringsjobb, storage per prefix, auth-användare och organisation, HMAC-nyckel förstörs, täckningstest. Operatörsroll beskriven och global flaggskrivning dold tills den finns (BP:170), vilket var mitt andra alternativ. Operatörsrollen har inget eget test (V3-05, LÅG). |
| V2-03 CODEOWNERS | HÖG | **ÅTGÄRDAD** | Avsnitt 1 (BP:42, 49–50), S0.0 (BP:97–99), H8 (BP:68) | Egen agentidentitet, Code Owner-krav, ingen bypass även för administratörer, inga force-pushar, `CODEOWNERS` skyddar sig själv och `.github/`, `gitleaks` som fallback, bekräftelse via H8. Verkan återstår tills H8 är gjord och filen finns (V3-04). |
| V2-04 Production uppskjuten | MEDEL | **DELVIS** | ADR-015, BP:34, Avsnitt 4A (BP:180–187) | ADR finns och är godkänd, och readiness-gaten täcker det jag begärde. Men planens text säger fortfarande emot ADR:n på flera ställen, se punkt (a)–(d) nedan. Readiness-gaten är dessutom ofullständig (V3-01). |
| V2-05 Skanning i CI | MEDEL | **ÅTGÄRDAD** | S0.1 (BP:107, 109), Avsnitt 3 (BP:87) | Dependency-audit (Python och frontend), secret-skanning, containerskanning, frontend-lockfil, fallerar på HIGH/CRITICAL, undantag endast via fil under CODEOWNERS. Alla ingår i S0.1-gaten. |
| V2-06 SCANNING och fixtures | MEDEL | **ÅTGÄRDAD** | S1A.0 (BP:195) | Magic-byte, ändelse/MIME, filnamnssanering, `.xlsm` avvisas, gränser i workern, fixtures för zip-bomb, XXE, pixelbomb och fel filtyp som gate för S1A.2–S1A.5, och `.xls` ifrågasatt. |
| V2-07 Org-nivå RLS | MEDEL | **ÅTGÄRDAD** | S0.3 (BP:127–128) | `app.current_user_id`, policyer för organisationer, medlemskap och `company_access`, bootstrap via `created_by = current_user_id`, fail-closed-test. Detaljer saknas för `companies`-tabellen och första medlemsraden (V3-05, LÅG). |
| V2-08 `company_access` och matris | MEDEL | **ÅTGÄRDAD** | S0.4 (BP:143–145) | Genomdrivs i auktoriseringsfunktionen. Matrisen har dimensionerna begränsad medlem och annan användare i samma företag. Tester för båda, och för chattrådar (OB-8). |
| V2-09 Workerns exponering | MEDEL | **ÅTGÄRDAD** | S0.7 (BP:164, 169) | `app_deleter` i egen process med egna uppgifter och minsta R2-behörighet. Workern får inte raderingsbehörighet. Ägarlistan nämner dock ingen tredje Railway-tjänst (se 5.1, rad 12). |
| V2-10 Granskningstillfällen | MEDEL | **ÅTGÄRDAD** | Avsnitt 9 (BP:265) | Efter Stage 0, efter 1A (före 1B), 1C, 1F, 1G och före production-release. 1A:s exit gate (BP:207) nämner inte granskningen, men avsnitt 9 är tydligt. |
| V2-11 OB-1 | MEDEL | **ÅTGÄRDAD** | Avsnitt 0 (BP:22, 32), H5 (BP:66), ägarpunkt 5 och 10 (BP:46, 51), S0.0 (BP:98) | Ett enda tillfälle (före S0.0), kriterier i tabellraden, nyckeln knuten till S1A.5, beslut bekräftat i `DECISIONS.md`. Att kriterierna ännu inte är kontrollerade hanteras i V3-01. |
| V2-12 GDPR-dokumentation | MEDEL | **ÅTGÄRDAD** | S0.1 (BP:107), S0.7 (BP:166) | Underbiträdeslista i `SECURITY.md`, retention hos AI-leverantör och plattformar, HMAC-nyckel i secret store/KMS, roteras och förstörs vid radering. Ingen task skriver själva integritetsinformationen (V3-01). |
| V2-13 E2E-konton | MEDEL | **DELVIS** | ADR-016, S0.3 (BP:127), 9A (BP:257) | ADR godkänd och hjälparen schemalagd i S0.3. **ADR-016:s egna testkrav är inte schemalagda:** att hjälparen vägrar köra mot production, och att hemligheten bara exponeras för E2E-jobbet. Inget av det finns i S0.3-testerna (BP:128), S0.1-testerna (BP:108) eller någon gate. Importförbudet från `src/` täcks av import-lintern (BP:106). |
| V2-14 Bypass-scan och container | LÅG | **ÅTGÄRDAD** | S0.1 (BP:107–108) | `.dockerignore` (`fixtures/`, `tests/`, `docs/`), CI-test att imagen saknar dem, undantagsfil under CODEOWNERS med motivering per rad. |
| V2-15 Uppladdningsstorlek | LÅG | **ÅTGÄRDAD** | S0.5 (BP:151) | Överstor fil raderas direkt vid registrering, ofullständiga multipart avbryts, R2:s möjligheter ska verifieras. Ingen rad i testlistan (BP:152) visar att överstora objekt raderas. |
| V2-16 Ordningsberoenden | LÅG | **ÅTGÄRDAD** | S0.1 (BP:107), S0.2 (BP:113), S0.3 (BP:127) | Loggmodulen flyttad till S0.1 före `log-canary-test`, testkontohjälparen ligger i S0.3 före Data API-testet. |
| V2-17 Rester av SEC-12/13 | LÅG | **DELVIS** | S0.3 (BP:127), S0.4 (BP:143) | Händelsekatalog för `audit_log`, skydd mot kontouppräkning, beslut om tokenlagring och rate limiting per endpoint finns som krav. **Saknas:** parserfel som citerar celler i listan över förbjudet i loggar (BP:106 oförändrad), ett retentionsvärde för auditloggen (BP:127 säger bara "och retention"), och hur inloggningar som sker hos auth-leverantören når `audit_log`. |

### Punkter under V2-04 som planen måste rätta (K1)

| | Var | Problem |
| --- | --- | --- |
| a | BP:40, 43–45, 48 | Ägarlistan säger att "saknas något av detta är det en BLOCKER för S0.0", men punkt 2–4 och 7 kräver production-projekt, -miljö och -bucket. ADR-015 säger att production skapas först före kunddata. En byggagent som läser texten bokstavligt blockerar S0.0. |
| b | BP:94 | S0.0:s mål är "live i staging och production", medan gaten (BP:100) bara kräver staging. |
| c | BP:64 | H3 är "Godkänn production-deploy efter Stage 0 exit gate". Det finns ingen production att deploya till då. |
| d | BP:5, 34, 67 | Statusraden och miljöraden säger att ADR-015/016 "kräver ägarens godkännande", och H7 står kvar som öppen. De är nu godkända (`DECISIONS.md`). |

---

## 3. BLOCKER A och B

| | Status | Underlag |
| --- | :-: | --- |
| **A** Production uppskjuten utan ADR | **LÖST** | ADR-015 är godkänd (`DECISIONS.md`) och Production readiness-gate finns (BP:180–187). Den kvarstående textsynkningen följs i V2-04. |
| **B** E2E-konton via admin-API utan ADR | **LÖST** | ADR-016 är godkänd. Testkraven följs i V2-13. |

Inga nya BLOCKER har uppstått. Blueprint §74 respektive §0B är fortfarande oförändrade i sin text. ADR:erna är den mekanism som `AGENTS.md` regel 10 kräver, men en byggagent som bara läser blueprinten ser konflikten. Se 5.3.

---

## 4. Åtgärdslistan i V2 avsnitt 6

### Före H1

| Åtgärd | Svar | Task-ref |
| --- | :-: | --- |
| Katalogbaserat RLS-täckningstest i gemensamma gates och Stage 0 exit gate | **ÅTGÄRDAD** | BP:85, 175 (V3-02 om urvalet) |
| Raderingstask med schemaläggare, verifiering och täckningstest, operatörsroll definierad eller dold | **ÅTGÄRDAD** | S0.7 |
| Code Owner-krav, egen agentidentitet, verifiering av inställningar | **ÅTGÄRDAD** (plan) | BP:42, 49, 99, H8. Verkan kräver H8 och en `CODEOWNERS`-fil. |
| ADR för production-uppskjutning | **DELVIS** | ADR-015 godkänd, men K1 (a–d) och V3-01 återstår |
| ADR för E2E-konton | **DELVIS** | ADR-016 godkänd, men dess testkrav är inte schemalagda (V2-13) |
| Ett beslutstillfälle för OB-1/OB-5/OB-6 som beroende i S0.0 | **ÅTGÄRDAD** | BP:32, 66, 98, `DECISIONS.md` |
| Fält **Säkerhetstester** i varje task | **DELVIS** | Regeln finns (BP:84) och raden finns i S0.1, S0.3, S0.4, S0.7, men saknas i de flesta Stage 1-tasks, se nedan |

**Tasks utan egen `Säkerhetstester`-rad med negativt fall (K3):** S1A.1, S1A.5, S1A.7, S1A.9, S1A.10, Stage 1B, 1C, 1D, 1E och 1H. S1A.2–S1A.4 täcks av fixturerna i S1A.0, S0.5 och S0.6 har negativa fall i sina testlistor, och S0.0 och S0.2 har inga. Regeln i avsnitt 3 binder visserligen varje task, men en byggagent följer raden i sin task. Exempel som saknas: ogiltig tillståndsövergång (S1A.1), viewer bekräftar mappning (S1A.9), att tenant-tester täcker nya tabeller (1B), att scenarier inte skriver till riktig data (1E).

### Före respektive task

| Task | Åtgärd | Svar |
| --- | --- | :-: |
| S0.1 | Dependency-/secret-/containerskanning, frontend-lockfil, loggmodul flyttad hit | **ÅTGÄRDAD** |
| S0.3 | Org-nivå RLS, bootstrap av första organisationen, audit-händelsekatalog | **ÅTGÄRDAD** |
| S0.4 | `company_access`, utökad matris, kontouppräkning, tokenlagring | **ÅTGÄRDAD** |
| S0.6 | Separat process för `app_deleter` | **ÅTGÄRDAD** (flyttad till S0.7, BP:169) |
| Före S1A.2 | SCANNING-task och fientliga fixtures | **ÅTGÄRDAD** (S1A.0) |
| S1A.5, S1A.8, 1F | Injektionsset med pass-gräns, schematest för tool-argument | **DELVIS** |
| Efter 1A och 1G | Security Reviewer-checkpoint | **ÅTGÄRDAD** (BP:265) |
| S0.1 / 9A | `.dockerignore`, undantagsfil för bypass-scan | **ÅTGÄRDAD** |
| S0.1, raderingstask | Underbiträdeslista, HMAC-nyckelhantering | **ÅTGÄRDAD** |

**Varför injektionsraden är DELVIS (K5):** S1A.8 (BP:203) kräver att injektionssetet *passerar*, och 1F (BP:247) kräver det i gaten samt ett deterministiskt schematest för tool-argument. Men kravet för S1A.5 står bara i 1F-raden ("Samma set, anpassat, gäller S1A.5"), inte i S1A.5:s egen rad (BP:200), och 1F:s gate-kolumn nämner fortfarande bara att evals är "sparade". S1A.5 byggs först, och en byggagent som läser sin task ser inget injektionskrav.

---

## 5. Konsekvenskontroll: ADR-015, ADR-016, DECISIONS.md och planen

### 5.1 Samstämmighet

| # | Kontroll | Resultat |
| --- | --- | :-: |
| 1 | ADR-015 refererar Production readiness-gate i BP avsnitt 4A. Gaten finns och täcker Stage 0-testerna, godkännare, backup med testad återställning och rapport. | **Stämmer** |
| 2 | ADR-015 "production före första kunddata och före H4" mot BP:182 "innan någon kunddata laddas upp, och innan H4". | **Stämmer** |
| 3 | ADR-015 "ingen kunddata i staging eller production förrän gaten är grön" mot blueprint §68 (staging utan riktiga kunddata). | **Stämmer** |
| 4 | ADR-016 hjälparen i `tests/e2e/`, aldrig importerad från `src/`: täcks av import-linter (BP:106). Vägrar köra mot production: **ingen task schemalägger testet**. Hemlighet bara för E2E-jobbet: **ingen task schemalägger det**, bara att PR-previews inte får hemligheter (BP:97). | **Delvis** (V2-13) |
| 5 | ADR-016 undantag i bypass-scanens fil: filen finns i S0.1 (BP:107), men ingen rad nämner hjälparen. | **Stämmer i princip** |
| 6 | `DECISIONS.md` OB-1, OB-5, OB-6 mot planen: planen kräver bekräftelse där (BP:32). Bekräftelsen finns. OB-6-texten är identisk med BP:27. | **Stämmer** |
| 7 | `DECISIONS.md` OB-5: saknad EU-region ska rapporteras som BLOCKER. Planen (BP:26) säger bara "där leverantören erbjuder det", vilket kan läsas som att en annan region får användas. `DECISIONS.md` är senare och skarpare. | **Planen bör skärpas** (K1) |
| 8 | `DECISIONS.md` C1–C4 mot planen: C2 = avsnitt 4A. C1 (leverantörskontroll), C3 (integritetsinformation) och C4 (EU-region) finns **inte** som krav i planen eller i 4A. | **Saknas i planen** (V3-01) |
| 9 | `DECISIONS.md` ADR-016 och ADR-015 mot ADR-filernas statusrad: båda säger "Godkänd av ägaren 2026-10-09 (H7)". Statusraden ändrades av agent efter ägarens besked (PR #4). Merge av PR #4 är underlag för bekräftelsen. | **Stämmer** |
| 10 | `DECISIONS.md` OB-1: "enda aktiva leverantör", leverantörsneutralt gränssnitt, `new_ai_provider` för andra leverantör. Planens 1F-rad (BP:247) säger inget om att bara en leverantör implementeras. Blueprint §36 listar tre leverantörsklasser. | **Saknas i planen** (V3-06) |
| 11 | Planens H-tabell (BP:62–69): H5 och H7 är uppfyllda enligt `DECISIONS.md`, men står kvar som öppna. H3 är föråldrad. | **Rättas** (K1) |
| 12 | Ägarlistan (BP:44) listar API och worker i Railway. `S0.7` kräver en separat process eller tjänst för `app_deleter` (BP:169). | **Rättas** (K1) |
| 13 | Checklistans pekare (BP:293, 295): P13 pekar på "Avsnitt 9A" och P15 på "Avsnitt 9A", men tasken är S0.7 respektive S0.3/ADR-016. `docs/verification/` (BP:298) finns inte ännu. | **Rättas** (V3-05) |

### 5.2 Anmärkningar om ADR-015:s motivering

ADR-015 påstår att gratisprojekt i Supabase pausas och saknar dagliga backuper. [Verifiera] Jag har inte kontrollerat det mot aktuell dokumentation. Beslutet bygger inte på påståendet, men det bör kontrolleras när production-planen väljs.

### 5.3 Blueprint och ADR

Blueprint §74, §67A, §68 och §0B är oförändrade. Båda ADR:erna säger att de ändrar dem. En byggagent som läser källorna i prioritetsordningen (`AGENTS.md`: blueprint först) ser två konflikter som en ADR har löst. **Rekommendation:** Architect lägger en rad i blueprintens ändringslogg (v1.2) som pekar på ADR-015 och ADR-016, så att ingen agent rapporterar en BLOCKER som redan är beslutad.

---

## 6. Nya fynd (V3-nn), sorterade efter allvarlighet

### MEDEL

#### V3-01 – Production readiness-gaten är ofullständig och delvis ej körbar

- **Observation:** (a) `DECISIONS.md` C1 kräver att leverantörens villkor och personuppgiftsavtal kontrolleras mot aktuell dokumentation före första kunddata, och att endast syntetiska testdata skickas dit tills dess. Planens 4A (BP:180–187) nämner varken C1, integritetsinformation eller underbiträdeslista, och ingen task skriver integritetsinformationen (BP:256 förutsätter den). (b) 4A kräver att Data API-testet, route-matrisen och raderingen körs mot production. Dessa behöver ett testkonto (BP:129, 172). ADR-016 förbjuder admin-API-hjälparen i production, och planen säger inte hur ett testkonto då skapas.
- **Risk:** Kunddata kan nå en AI-leverantör vars villkor ingen har läst, utan att något i planen stoppar det, eftersom skyddet bara ligger i en beslutsfil som byggagenter inte styrs av. Readiness-gaten går inte att köra enligt sin egen text, vilket frestar till en genväg i production.
- **Åtgärd:**
  1. Lägg C1, integritetsinformation och underbiträdeslista som punkter i 4A.
  2. Låt produktionens AI-nyckel först skapas och läggas i secret store när C1 är dokumenterad i `DECISIONS.md`. Då kan ingen kunddata nå leverantören av misstag.
  3. Definiera ett testkonto i production som skapas genom ordinarie registrering med minimala rättigheter, används av 4A-testerna och raderas via S0.7:s raderingstest.
  4. Lägg en task för integritetsinformationen, och en rad i 4A.

#### V3-02 – Urvalet för RLS-täckning och raderingsregister är kolumnbaserat (fail-open)

- **Observation:** BP:85 och BP:86 gäller "varje tabell med `company_id` eller `organization_id`". Det var min egen formulering i V2-01. Blueprint §7 definierar `import_status_history` med bara `import_batch_id` (ingen `company_id`). Tabellen `organizations` har bara `id`. Användarprofilen (§9.2) har varken `company_id` eller `organization_id`.
- **Risk:** Dessa tabeller hamnar utanför både RLS-testet och raderingsregistret. Statushistorik med importmetadata (felkoder, orsaker) för ett raderat företag kan finnas kvar, och en saknad RLS-policy upptäcks inte.
- **Åtgärd:** Testa **alla** tabeller i app-schemat. Kräv RLS, `FORCE` och policy på alla, med en uttrycklig undantagslista (under CODEOWNERS) för t.ex. uppslagstabeller. Gör raderingsregistret komplett på samma sätt. Besluta (ADR eller blueprintändring) att `import_status_history` får `company_id`, i linje med §11A.

### LÅG

#### V3-03 – ADR-016:s testkrav är inte schemalagda
Se V2-13. Lägg i S0.3:s säkerhetstester: hjälparen vägrar köra mot production, hemligheten finns bara i E2E-jobbets miljö, och hjälparen har en rad i bypass-scanens undantagsfil.

#### V3-04 – CODEOWNERS är inte på plats när H8 ska bekräftas
H8 är ett beroende för S0.0 (BP:98), men `CODEOWNERS` skapas i S0.0 (BP:97). [Känt] GitHub läser filen från PR:ens basgren, så den första PR:en som lägger in `CODEOWNERS` och workflows skyddas inte av den. **Åtgärd:** En första, liten PR med enbart `CODEOWNERS` som ägaren mergar manuellt (planen anger redan manuell merge i planeringsfasen, BP:49), före alla workflow-PR:er. Låt H8-bevisen inkludera att filen finns.

#### V3-05 – Detaljer i S0.3/S0.7 och checklistans pekare
- `companies` har inget definierat policyunderlag: `GET /companies` listar flera företag och passar inte en policy som bygger på ett enda `app.current_company_id`.
- Första medlemsraden för en ny organisation (ägaren) saknar uttryckligt bootstrap-villkor, och `created_by` finns inte i blueprint §9.1.
- Operatörsrollen (BP:170) har inget test. Minst ett: ingen kundroll kan skriva global flagga.
- Checklistans rader P13 och P15 (BP:293, 295) pekar på avsnitt 9A i stället för S0.7 och S0.3/ADR-016.

#### V3-06 – Leverantörsbegränsningen i `DECISIONS.md` saknas i 1F
`DECISIONS.md` säger att Anthropic är enda aktiva leverantör och att övriga läggs till bakom `new_ai_provider`. Planen anger inte att bara en leverantör implementeras. Blueprint §36 listar `OpenAIProvider`, `AnthropicProvider` och `GeminiProvider`, och en stubbad klass vore en falsk funktion (§0B). **Åtgärd:** Skriv i 1F-raden att endast `AnthropicProvider` implementeras, att andra inte får finnas som tomma klasser, och lägg ett registertest som visar exakt en aktiv leverantör när flaggan är av.

---

## 7. Villkor för att godkänna v3 (v3.1)

### Före S0.0 (plantext, ingen arkitektur ändras)

| # | Åtgärd | Fynd |
| --- | --- | --- |
| K1 | Synka planens text med ADR-015/016 och `DECISIONS.md`: ägarlistan (BP:40–45, 48), S0.0:s mål (BP:94), H3 (BP:64), statusrad (BP:5), miljörad (BP:34), H5/H7 som genomförda, OB-5 skärpt (BP:26), tredje Railway-tjänst för `app_deleter`. | V2-04, 5.1 |
| K2 | Utöka 4A med C1, integritetsinformation, underbiträdeslista, AI-nyckel i production först efter C1, och testkontoprocedur i production. | V3-01 |
| K3 | Lägg en rad `Säkerhetstester` med minst ett negativt fall i varje task som saknar den. | Åtgärdslistan |
| K4 | Ändra RLS- och raderingsurvalet till alla tabeller, med undantagslista. | V3-02 |
| K5 | Schemalägg ADR-016:s tester, och flytta injektionskravet för S1A.5 in i dess egen rad samt 1F:s gate-kolumn. | V2-13, åtgärdslistan |

### Före respektive task

| Åtgärd | Fynd |
| --- | --- |
| K6: Parserfel i loggförbudslistan, retentionsvärde för auditloggen, mekanism för inloggningsloggning | V2-17 |
| K7: Bootstrap-PR med enbart `CODEOWNERS`; blueprintens ändringslogg pekar på ADR-015/016 | V3-04, 5.3 |
| K8: `companies`-policy, medlems-bootstrap, operatörstest, checklistans pekare, leverantörsregistertest | V3-05, V3-06 |

---

## 8. Status (AGENTS.md §4)

**Completed:**
- Granskning av `BUILD_PLAN.md` v3 mot V2-01–V2-17, BLOCKER A och B och åtgärdslistan, samt konsekvenskontroll av ADR-015, ADR-016 och `DECISIONS.md`.
- Resultat: 14 ÅTGÄRDAD, 3 DELVIS, 0 EJ ÅTGÄRDAD. Båda BLOCKER lösta. 6 nya fynd (2 MEDEL, 4 LÅG), inga HÖGA.

**Verified (med bevis):**
- Radnummer i planen är kontrollerade mot filen med sökning.
- Att `main` är skyddad (`protected: true`) och att `.github/` och `CODEOWNERS` saknas är kontrollerat via GitHub API respektive filsystemet.
- Att `import_status_history` saknar `company_id` i blueprint §7 är kontrollerat med sökning.
- ADR-filernas, `DECISIONS.md`:s och planens innehåll är läst i sin helhet.
- Påståenden märkta **[Verifiera]** är inte kontrollerade mot leverantörsdokumentation.

**Tests:** Inga. Det finns ingen kod, så alla bedömningar gäller plantext.

**Current issue:**
- Planens text säger emot ADR-015 på fyra ställen (V2-04), och Production readiness-gaten saknar C1 och en körbar testkontoprocedur (V3-01).
- ADR-016:s testkrav är inte schemalagda (V2-13).
- H8 är öppen: `main` är skyddad, men Code Owner-kravet saknar `CODEOWNERS`-fil.
- Anthropics villkor och personuppgiftsavtal är inte kontrollerade (`DECISIONS.md` C1), och jag har inte kontrollerat dem här.

**Next:** Architect skriver v3.1 med K1–K5. Security Reviewer granskar v3.1 mot dessa, och återverifierar mot implementationen vid Stage 0 exit gate. Ägaren beslutar H1 och genomför H8.
