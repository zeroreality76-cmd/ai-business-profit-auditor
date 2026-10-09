# BUILD_PLAN.md – Arbetsschema (Architect-output, v2)

Följer `PROJECT_ARCHITECTURE_BLUEPRINT.md` v1.1 (§115A). Ingen kod i detta dokument. Roller och regler: `AGENTS.md`.

Status: **v2 – inarbetar `docs/SECURITY_REVIEW_PLAN.md` (fynd SEC-01–SEC-24, krav P1–P16). Väntar på omgranskning av Security Reviewer och ägarens godkännande.**

Säkerhetsfynd som ändrat planen är märkta **[SEC-nn]**.

---

## 0. Beslut

**ADR-014 (förslag): Stack för Stage 1 = blueprintens §67.**
GitHub Actions (CI/CD) · Cloudflare Pages (frontend) · Railway (API + worker) · Supabase (PostgreSQL + Auth) · Cloudflare R2 (filer) · molnbaserad AI-leverantör bakom `AIProvider`.

Skäl: minst antal rörliga delar, snabbast till live, ingen egen nätverks-/IAM-drift. AWS är inte valt. Byte kräver ny ADR eftersom allt ligger bakom adapterlagret (§49).

**Standardvärden för öppna beslut (gäller tills ägaren svarar):**

| ID | Standard |
| --- | --- |
| OB-1 AI-leverantör | Väljs vid S1F.2. Modellnamn och priser kontrolleras då mot leverantörens aktuella dokumentation. |
| OB-2 Behörighet | Matrisen i blueprint §4.1. |
| OB-3 Severity-mappning | Förslaget i blueprint §27. |
| OB-4 Lead time m.m. | Tabell i blueprint §21A. |
| OB-5 Region | EU-region för Supabase, Railway och R2 där leverantören erbjuder det. **Beslutas före S0.0** (region går ofta inte att byta) [SEC-08]. |
| OB-6 Malware-skanning | Ingen hook utan leverantör. Endast dataformat som inte kan köras (CSV, XLSX utan makron, PDF, bilder) plus hård validering, tills en molnskanner valts [SEC-06]. |
| OB-7 Fördröjd permanent radering | 7 dagar [SEC-14]. |
| OB-8 Synlighet för chattrådar | Privata per användare. Tool-anrop kontrolleras alltid mot `company_access` [SEC-03]. |
| OB-9 Operatörsåtkomst till production | Break-glass: tidsbegränsad, MFA, auditloggad [SEC-18]. |

**OB-1 (AI-leverantör, med DPA och ingen träning på kunddata) och OB-5 ska beslutas före S0.0**, eftersom första AI-användningen sker redan i S1A.5/S1A.8 [SEC-08].

**Miljöer just nu:** endast `staging` finns (ett gratis Supabase-projekt). `production` skapas före första pilotkund (Supabase Pro). Production-delarna av gaterna nedan skjuts till dess.

---

## 1. Ägarens uppgifter (kan inte automatiseras)

Saknas något av detta är det en **BLOCKER** för S0.0. Agenten skapar aldrig konton åt ägaren.

1. GitHub-repo med skrivåtkomst för agenten (molnagent kopplad till repot, eller smal token med `Contents` + `Workflows` write för detta repo).
2. Supabase: två projekt, `staging` och `production`.
3. Railway: två miljöer med API-tjänst och worker-tjänst.
4. Cloudflare: Pages-projekt och R2-buckets (staging och production).
5. API-nyckel hos AI-leverantör (behövs först vid S1A.8 för AI-mappning).
6. Hemligheter läggs i GitHub Environments (`staging`, `production`) och plattformarnas secret stores. **Aldrig i chatten eller i repot.**
7. GitHub Environment `production` ska kräva manuellt godkännande (required reviewer = ägaren).
8. Branch protection på `main`: PR krävs, CI krävs, inga direkt-pushar [SEC-10].
9. Slå på GitHub secret scanning och push protection [SEC-15].
10. Besluta OB-1 och OB-5 (se avsnitt 0) innan S0.0.
11. Supabase: lägg aldrig appens tabeller i Husappens projekt. Staging-projektet är separat [SEC-01].

---

## 2. Mänskliga kontrollpunkter

Agenterna kör själva däremellan.

| # | Punkt | Vem |
| --- | --- | --- |
| H1 | Godkänn denna plan efter Security Reviewer-granskning | Ägare |
| H2 | Skapa konton och lägg hemligheter (avsnitt 1) | Ägare |
| H3 | Godkänn production-deploy efter Stage 0 exit gate | Ägare |
| H4 | Godkänn production-deploy efter Stage 1C (första marknadsdugliga leveransen) | Ägare |
| H5 | Besvara OB-1 före S1A.8 om standard inte duger | Ägare |
| H6 | Välj pilotföretag och samla feedback (§84, §111) | Ägare |

---

## 3. Gemensamma gates för varje task

En task är klar först när (blueprint §73):

- implementation, typer, validering och felhantering finns,
- unit-tester finns, integration-test där relevant,
- auktorisering och tenant-isolering är testad,
- loggning och dokumentation finns,
- CI är grön (lint, typecheck, tester, **bypass-scan**, **arkitekturtest**, migrationstest),
- den är deployad och verifierad i staging,
- ingen bypass-, mock- eller demokod har tillkommit.

---

## 4. STAGE 0 – Foundation

### S0.0 Cloud provisioning
- **Mål:** Tomt men deploybart skelett live i staging och production via CI/CD.
- **Filer:** `.github/workflows/ci.yml`, `deploy-staging.yml`, `deploy-production.yml`, `apps/api/main.py` (endast `GET /api/v1/health`), `Dockerfile`, `infra/README.md` (vilka tjänster, vilka secret-namn, inga värden).
- **Beroenden:** H2.
- **Säkerhetskontroller [SEC-10, SEC-17]:** `CODEOWNERS` som kräver ägarens granskning för `.github/workflows/`, `migrations/`, `infra/`, `core/security.py`, RLS-policyer, bypass-scan-konfiguration och säkerhetstester. Actions låsta till commit-SHA, `permissions:` minimala, inget `pull_request_target` med PR-kod, deploy-tokens per miljö med minsta behörighet. PR-previews får aldrig använda staging- eller production-hemligheter.
- **Gate:** `GET /api/v1/health` ger 200 över HTTPS i **staging**, deployat av CI/CD. (Production tas när det skapats, före pilot.) Agenten får aldrig ändra säkerhetstester för att få en gate grön. Sådan ändring rapporteras som BLOCKER.

### S0.1 Repository
- **Mål:** Hela mappstrukturen enligt §48 och verktyg.
- **Filer:** `pyproject.toml` med låst lockfile, `src/business_auditor/...` (tomma moduler med `__init__.py`), `tests/{unit,integration,contract,golden,e2e,architecture}`, `scripts/bypass_scan.py`, `tests/architecture/test_import_rules.py`, `apps/web` (Vite + React + TS + Tailwind), `docs/adr/`, dokumentationsfilerna från §108 (skelett).
- **Tester:** Arkitekturtest (Domain får inte importera FastAPI, SQLAlchemy, AI- eller storage-SDK:er). Bypass-scan (förbjudna mönster i `src/` och `apps/`: `DISABLE_AUTH`, `DEBUG_USER`, `Mock*Provider`, `FakeStorage`, `localhost`, `docker-compose`, m.fl.).
- **Utökat [SEC-09, SEC-15]:** Bypass-scan täcker även `infra/`, `scripts/`, `.github/workflows/` och `Dockerfile`. Scan som misslyckas om hemligheter (service role, R2-nycklar, AI-nyckel) finns i frontend-bygget eller `VITE_*`-variabler. Import-linter förbjuder import av testdubletter från `src/`. **Loggredaktor** i `core/logging.py` med fält-allowlist, plus kanarietest (kända hemliga strängar får aldrig synas i logg). Förbjudet i logg: signerade URL:er, delade Sheets-länkar, sample values, prompts/AI-svar, originalfilnamn, request-/response-bodies, `jobs.payload`, JWT.
- **Gate:** CI: `lint`, `typecheck`, `tests`, `bypass-scan`, `architecture-test`, `log-canary-test`, `container-build` alla gröna.

### S0.2 Configuration
- **Mål:** Typade inställningar som vägrar starta utan obligatoriska hemligheter.
- **Filer:** `core/config.py`, `core/errors.py`, `core/logging.py` (JSON-loggar, `request_id`).
- **Tester:** Saknad secret → startup failure. Hemligheter förekommer aldrig i loggutskrift.
- **Gate:** Testerna gröna i CI. Staging startar med riktiga secrets.

### S0.3 Database
- **Mål:** Kärnschema, RLS och migrationer.
- **Filer:** Alembic-migrationer för `organizations`, `organization_memberships`, `companies`, `company_access`, `audit_log`, `jobs`, `feature_flags`. Dedikerad applikationsroll **utan** RLS-bypass. RLS-policys via `SET LOCAL app.current_company_id`.
- **Regler:** UUID, `timestamptz`, `NUMERIC(20,4)` + `currency`, constraints (§11A).
- **Tester:** Migration up/down mot molnbaserad PostgreSQL i CI. **Negativt cross-tenant-test:** användare A når aldrig företag B:s rader.
- **Säkerhetskrav [SEC-01, SEC-02, SEC-13, SEC-20]:**
  - Affärstabeller i eget schema som **inte** exponeras via Supabase Data API (eller Data API avstängt). `REVOKE ALL` för `anon` och `authenticated` i första migrationen.
  - Fyra roller: `migrator` (endast CI-migrationer), `app_api`, `app_worker` (smal policy på `jobs`, sedan `SET LOCAL` från `jobs.company_id`), `app_deleter` (endast `DELETE_COMPANY_DATA`). Ingen runtime-roll har BYPASSRLS eller DDL.
  - Policys är fail-closed: saknat tenant-värde ger noll rader. `SET LOCAL` används alltid inne i transaktion.
  - `audit_log`: runtime-roller får bara `INSERT`, ingen FK med cascade mot `companies`, definierade fält (actor, org, company, action, target, resultat, request_id, tidpunkt), ingen affärsdata.
- **Gate:** Migrationer, Data API-test (publik nyckel + testkonto nekas på varje tabell), RLS fail-closed-test, pool-läckagetest (tenant-kontext följer inte med mellan requests), `rolbypassrls = false` för alla runtime-roller. Mönstret verifierat mot aktuell Supabase-dokumentation.

### S0.4 Authentication
- **Mål:** Registrering, login, logout, password reset bakom ett auth-interface (Supabase Auth). JWT-validering i API. `TenantContext` per request. Minimal inloggningssida i `apps/web` så att etappen är användbar live.
- **Filer:** `infrastructure/auth/*`, `GET /api/v1/me`, organisations- och företagsendpoints (§61), rollkontroller enligt §4.1.
- **Tester:** Obehörig åtkomst ger 403/404. Ingen väg runt autentisering. Behörighetsmatrisen testas per roll.
- **Säkerhetskrav [SEC-03, SEC-12, SEC-19, SEC-22, SEC-23]:**
  - All auktorisering går genom **en** funktion i `core/security.py` som hämtar medlemskap och roll från databasen vid varje request, inte från JWT-claims.
  - Route × roll × (egen/främmande tenant)-matris som CI-test. Ny route utan matrispost får CI att fallera.
  - Regler: admin får inte göra sig själv eller andra till `owner`, ändra eller ta bort `owner`, eller ta bort sista `owner`.
  - JWT: fast algoritm, kontroll av `iss`, `aud`, `exp`, JWKS med cache/rotation (verifiera aktuell Supabase-modell).
  - API accepterar endast `Authorization: Bearer` (inget CSRF-behov). CORS: exakt allowlist per miljö. Strikt CSP och säkerhetsheaders (HSTS, nosniff, `Referrer-Policy: strict-origin-when-cross-origin`, `frame-ancestors 'none'`).
  - Rate limiting för auth, `upload-url`, `google-sheet`, chatt och `audits`. Verifierad e-post krävs. MFA som tillval för `owner`/`admin`.
  - Cross-tenant ger samma svar som "finns inte" (404), så resursers existens avslöjas inte.
- **Gate:** `Authentication PASS`, `Tenant isolation PASS` (hela route-matrisen, inte ett enskilt test).

### S0.5 Storage
- **Mål:** `ObjectStorage`-adapter mot R2 med signerade upp- och nedladdnings-URL:er, nyckelstruktur `organization/company/import/file`.
- **Regler:** Versionering avstängd eller permanent rensning för kundfiler (§66, §107).
- **Säkerhetskrav [SEC-05, SEC-21]:** Servern genererar hela nyckeln (`org/company/import/<uuid>`), klientens filnamn är bara sanerad metadata. TTL ≤ 10 min för uppladdning och ≤ 5 min för nedladdning (`Content-Disposition: attachment`, ingen CDN-cache). Vid registrering gör servern `HEAD` och kontrollerar storlek, och beräknar själv `sha256` i `SCANNING`. Lifecycle-regel tar bort oregistrerade objekt efter 24 h. Rate limit på `upload-url`.
- **Tester:** upload, read, delete, signerad URL går ut, åtkomst över tenantgräns nekas, användare A kan inte få signerad URL till B:s nyckel.
- **Gate:** `Storage PASS` mot riktig R2-bucket i staging.

### S0.6 Worker
- **Mål:** PostgreSQL-jobbkö med `FOR UPDATE SKIP LOCKED`, retry 30 s / 2 min / 10 min, återlämning av låsta jobb efter timeout.
- **Filer:** `apps/worker/main.py`, jobbtabell, `GET /api/v1/jobs/{id}`.
- **Säkerhetskrav [SEC-02, SEC-20]:** Workern kör som `app_worker` (ingen BYPASSRLS). Tenant-kontext tas från `jobs.company_id`, aldrig från `payload`. Payload med avvikande `company_id` avvisas. `DELETE_COMPANY_DATA` kan bara skapas av API-endpointen efter rollkontroll och är idempotent. `last_error` innehåller bara felkod och tekniskt meddelande.
- **Tester:** Concurrency-test (två workers tar aldrig samma jobb). Valideringsfel retry:as inte. Max attempts → `FAILED`. Payload med fel `company_id` avvisas.
- **Gate:** `Worker PASS`. Verifiera anslutningsläge mot Supabase (session-läge, inte transaction-pooler om det ger problem).

### STAGE 0 EXIT GATE (§74)
`Authentication`, `Tenant isolation` (route-matris, Data API-test, RLS fail-closed, pool-läckage, storage-prefix), `Migrations`, `Worker`, `Storage`, `CI` och `Deployed in staging` ska alla vara PASS. Production-deploy tas när production-miljön finns.
→ Security Reviewer granskar. Verifier kör hela sviten. **H3.**

---

## 5. STAGE 1A – Import (§75)

| Task | Mål | Nyckelkrav |
| --- | --- | --- |
| S1A.1 | `ImportBatch`, `ImportFile`, `import_status_history`, state machine | Alla övergångar loggas. Inga tysta fel (§7). Uppladdning via signerad URL (§55). |
| S1A.2 | CSV-parser | Chunkad (§98). Teckenkodning och avgränsare detekteras. Gräns för radlängd och fältstorlek [SEC-06]. |
| S1A.3 | XLS/XLSX-parser | Flera ark. `defusedxml`, skydd mot zip-bomber, `.xlsm` och andra makroformat avvisas [SEC-06]. |
| S1A.4 | PDF-textextraktion | Sid-, storleks-, tids- och minnesgräns, tvingade på serversidan i workern [SEC-06]. |
| S1A.5 | Bild/dokument-extraktion via adapter | Pydantic-validerad output. Bakgrundsjobb. `MAX_IMAGE_PIXELS` får inte stängas av. Personnummer/organisationsnummer maskas innan lagring. Användaren informeras om att bild/PDF skickas till extern AI-leverantör [SEC-06, SEC-08]. |
| S1A.6 | Google Sheets via delad länk | **SSRF-skydd [SEC-07]:** allowlist per hopp (`docs.google.com` → högst 2 redirects till `*.googleusercontent.com`, endast `https`, port 443, verifiera mot verkligt beteende). Egen resolver med IP-kontroll (privata, loopback, link-local, metadata, IPv4+IPv6), skydd mot DNS rebinding, tid- och storleksgräns, körs i workern. Länken loggas aldrig. Negativa tester: IP-literal, `localhost`, `169.254.169.254`, `[::1]`, redirect till intern host, `docs.google.com@evil`. |
| S1A.7 | Dokumentklassificerare | Okänt → `UNKNOWN`, aldrig gissning. |
| S1A.8 | Kolumnmappning nivå 1–3 | Tröskelvärden enligt §8. Max 10 sample values, personuppgiftskolumner maskas (§99). Filinnehåll märks som otillförlitlig data i prompten. AI-mappning av finansiellt kritiska fält visas alltid för användaren. Injektionstester [SEC-04]. |
| S1A.9 | UI för bekräftelse av mappning | Enkel för icke-teknisk användare. |
| S1A.10 | Datakvalitetsrapport | `GOOD` / `USABLE_WITH_WARNINGS` / `POOR` / `INSUFFICIENT`. |

**Exit gate:** Golden-filer för CSV, XLSX, PDF, bild och Google Sheet samt mappning ger förväntat resultat. Deployad i staging.

---

## 6. STAGE 1B – Normalisering (§76)

Ordning: products → suppliers → sales → purchases → inventory → financial statements → recipes → waste.

- Idempotent. Dubblettfil avvisas via `sha256` + `company_id`.
- Dubbletter inom fil flaggas i kvalitetsrapporten, kasseras inte tyst.
- Korrigerad omuppladdning: varna, fråga, markera `superseded_by` (§14). Inget raderas.
- Delsummor lagras med `is_subtotal` och räknas om (§12).
- Okänd enhet → datakvalitetsvarning (§13).

**Gate:** Golden retail- och restaurant-filer normaliseras exakt som dokumenterat. Radantal och summor stämmer mot källa.

---

## 7. STAGE 1C – Core Analytics (§77)

Ordning: revenue → purchase totals → inventory valuation → gross margin → sales velocity → dead stock → slow moving → overstock → stockout → supplier price changes → margin erosion → financial trend → anomaly engine → Money Leak Engine → prioritering.

- Allt med `Decimal` och `ROUND_HALF_EVEN`. Division med noll ger `null` eller `NO_SALES`, aldrig infinity.
- Varje `MetricValue` och `Finding` får `MetricEvidence`. Finding utan evidence skapas inte.
- `Audit.settings_snapshot` sparar alla parametrar (§21A).
- Ekonomisk effekt som lower/base/upper, aldrig falsk precision (§26).
- Do This First: max 5 actions (§28).
- Dashboard (`GET /companies/{id}/dashboard`) och findings-vyer i UI, så att etappen är användbar live.

**Exit gate:** Golden retail-dataset ger förväntade findings. **Ingen LLM används för att passera gaten.** Deployad i staging.
→ **Första marknadsdugliga leveransen (§115A).** Security Reviewer + Verifier, sedan **H4.**

---

## 8. Resterande Stage 1

| Stage | Innehåll | Gate |
| --- | --- | --- |
| **1D** Restaurang (§78) | Receptkostnad, food cost, contribution margin, itemklassning, waste, actual vs theoretical (neutral formulering: *Unexplained variance detected.*) | Golden restaurant-dataset ger förväntade findings. |
| **1E** Forecast + scenario (§79) | Moving average, weighted MA, linjär trend, seasonal naive. Pris-, kostnads-, rea-, försäljnings-, inköps- och lönescenarier. Scenarier skriver aldrig till riktig data. | Deterministiska enhetstester. Antaganden och före/efter sparas. |
| **1F** AI CFO (§80) | `AIProvider`-interface, tool registry (§37), orchestrator, policies (§39), validator, chattpersistens, UI. **Tool-scheman får aldrig ha `company_id`/`organization_id` som argument**, tenant sätts på serversidan och tools använder samma auktorisering som API:t. Chattsvar renderas utan rå HTML och utan automatiska externa länkar. Injektionsfall i AI-evals [SEC-04]. | AI-gate: *"Why did my margin fall?"* använder tool, rätt period, rätt siffror, nämner datagap, hittar inte på orsak. Evals sparade (§72). |
| **1G** PDF (§81) | ReportLab (lås version ≥ 3.6.13, kontrollera senaste) som bakgrundsjobb, all användar- och AI-text escapas före `Paragraph`, test med fientlig produktsträng. Lagrad i R2, nedladdning via kort signerad URL [SEC-11, SEC-21]. | PDF innehåller alla sektioner (§42) med verifierad data. |
| **1H** Portal (§82) | Slutpolering och full sidtäckning. | **Final acceptance (§83)** mot production: skapa konto → företag → ladda upp → mappa → audit → fynd → actions → AI-fråga → scenario → PDF, utan utvecklaringrepp. |

---


## 9A. Radering, E2E och operatörsåtkomst (från säkerhetsgranskningen)

- **Radering [SEC-14]:** `DELETE_COMPANY_DATA` täcker även storage per prefix (listning, inte bara kända nycklar). Organisation och användarkonto kan raderas. Fördröjd permanent radering = 7 dagar (OB-7). Rader med `deleted_at` döljs av RLS och repositories. Integritetsinformationen anger AI-leverantörens och plattformarnas retention.
- **E2E utan bypass [SEC-16]:** En testhjälpare som bara finns i `tests/e2e/` skapar konton via auth-leverantörens admin-API med en staging-begränsad hemlighet. Den är testinfrastruktur, inte en produktväg, och fungerar aldrig mot production.
- **Operatör [SEC-18]:** Feature flags med global scope får bara ändras av operatörsroll (break-glass, MFA, auditloggad). Ingen kundroll får det.
- **Staging:** Ingen kopiering av production-data till staging i Stage 1 (golden datasets räcker) [SEC-24].
- **`customer_reference_hash`:** HMAC med hemlig nyckel per företag, inte vanlig hash [SEC-08].

## 9. Löpande verifiering (alla stages)

- **Verifier** körs efter varje stage-gate: golden regression, bypass-scan, arkitekturtest, tenant-negativtester, E2E mot staging.
- **Security Reviewer** körs efter Stage 0, efter 1C, efter 1F och före production-release.
- **Release gate (§106):** CI grön, inga kritiska säkerhetsfynd, migration testad, rollback-plan, golden regression grön, E2E grön.

---

## 10. Efter Stage 1

Pilot med 3–10 företag (§84, §111). Go/No-Go enligt §112. Stage 2–4 planeras först därefter.


---

## 11. Säkerhetschecklista P1–P16 (för omgranskning)

| # | Krav | Besvaras av |
| --- | --- | --- |
| P1 | Varje task har negativa säkerhetstester, inte bara "auktorisering testad" | Avsnitt 3–8 |
| P2 | OB-1, OB-5, OB-6 beslutade före S0.0 | Avsnitt 0 |
| P3 | Roller, icke-exponerat schema, `REVOKE`, fail-closed-policyer | S0.3 |
| P4 | Route × roll × tenant-matris, rollregler, auktorisering från databas | S0.4 |
| P5 | Servergenererad nyckel, TTL, storlek/checksumma, lifecycle | S0.5 |
| P6 | Worker utan BYPASSRLS, tenant från `jobs.company_id` | S0.6 |
| P7 | Parsergränser, `defusedxml`, `.xlsm` avvisas, fientliga fixtures | S1A.2–S1A.5 |
| P8 | SSRF per hopp, IP-validering, rebinding, negativa tester | S1A.6 |
| P9 | Tool-scheman utan tenant-argument, injektionstester | S1A.8, 1F |
| P10 | ReportLab låst och escapad | 1G |
| P11 | Loggredaktor med allowlist och kanarietest | S0.1 |
| P12 | Branch protection, CODEOWNERS, production-godkännare, secret scanning | S0.0, avsnitt 1 |
| P13 | Radering täcker org, användare och storage-prefix | Avsnitt 9A |
| P14 | Auditlogg: endast INSERT, ingen cascade, definierat schema | S0.3 |
| P15 | E2E-autentisering utan bypass | Avsnitt 9A |
| P16 | Inga localhost, docker-compose, mock eller seed i production | Alla tasks, bypass-scan |
