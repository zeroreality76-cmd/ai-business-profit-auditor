# BUILD_PLAN.md – Arbetsschema (Architect-output, v3.1)

Följer `PROJECT_ARCHITECTURE_BLUEPRINT.md` v1.1 (§115A). Ingen kod i detta dokument. Roller och regler: `AGENTS.md`.

Status: **v3.1 – inarbetar `docs/SECURITY_REVIEW_PLAN.md` (SEC-01–SEC-24, P1–P16), `docs/SECURITY_REVIEW_PLAN_V2.md` (V2-01–V2-17) och `docs/SECURITY_REVIEW_PLAN_V3.md` (K1–K8, V3-01–V3-06). ADR-015 och ADR-016 är godkända (`docs/DECISIONS.md`). Väntar på Security Reviewers granskning av v3.1 och ägarens godkännande (H1).**

Säkerhetsfynd som ändrat planen är märkta **[SEC-nn]** (första rapporten) **[V2-nn]** (omgranskningen) och **[V3-nn]/[Kn]** (granskningen av v3).

---

## 0. Beslut

**ADR-014 (förslag): Stack för Stage 1 = blueprintens §67.**
GitHub Actions (CI/CD) · Cloudflare Pages (frontend) · Railway (API + worker) · Supabase (PostgreSQL + Auth) · Cloudflare R2 (filer) · molnbaserad AI-leverantör bakom `AIProvider`.

Skäl: minst antal rörliga delar, snabbast till live, ingen egen nätverks-/IAM-drift. AWS är inte valt. Byte kräver ny ADR eftersom allt ligger bakom adapterlagret (§49).

**Standardvärden för öppna beslut (gäller tills ägaren svarar):**

| ID | Standard |
| --- | --- |
| OB-1 AI-leverantör | **Beslutas före S0.0** (används första gången i S1A.5). Urvalskrav: DPA finns, ingen träning på kunddata, kortast möjliga retention, EU-behandling där det erbjuds. Modellnamn och priser kontrolleras mot aktuell dokumentation vid S1A.5 [V2-11]. |
| OB-2 Behörighet | Matrisen i blueprint §4.1. |
| OB-3 Severity-mappning | Förslaget i blueprint §27. |
| OB-4 Lead time m.m. | Tabell i blueprint §21A. |
| OB-5 Region | **EU** (beslutad, `DECISIONS.md`). Saknar en plattform EU-region ska det rapporteras som BLOCKER, ingen annan region används tyst [SEC-08, K1]. |
| OB-6 Malware-skanning | Ingen hook utan leverantör. Endast dataformat som inte kan köras (CSV, XLSX utan makron, PDF, bilder) plus hård validering, tills en molnskanner valts [SEC-06]. |
| OB-7 Fördröjd permanent radering | 7 dagar [SEC-14]. |
| OB-8 Synlighet för chattrådar | Privata per användare. Tool-anrop kontrolleras alltid mot `company_access` [SEC-03]. |
| OB-9 Operatörsåtkomst till production | Break-glass: tidsbegränsad, MFA, auditloggad [SEC-18]. |
| OB-10 Retention för `audit_log` | 24 månader, därefter gallring. Innehåller aldrig affärsdata [K6]. |

**OB-1, OB-5 och OB-6 ska vara beslutade före S0.0.** De är uttryckliga beroenden i S0.0, och agenten får inte starta S0.0 förrän ägaren har bekräftat dem i `docs/DECISIONS.md` [SEC-08, V2-11, P2].

**Miljöer just nu:** endast `staging` finns (ett gratis Supabase-projekt). `production` skapas före första kunddata (Supabase Pro). Detta avviker från blueprint §74 och regleras av **ADR-015** (godkänd 2026-10-09, se `docs/DECISIONS.md`). Före första kunddata ska en **Production readiness-gate** vara grön (avsnitt 4A) [V2-04].

---

## 1. Ägarens uppgifter (kan inte automatiseras)

Punkter märkta **[S0.0]** måste finnas innan S0.0, annars **BLOCKER**. Punkter märkta **[production]** behövs först före första kunddata (ADR-015, avsnitt 4A) och är **inte** blockerande för S0.0. Agenten skapar aldrig konton åt ägaren.

1. GitHub-repo med skrivåtkomst för agenten. Agenten ska ha **egen identitet** (GitHub-app eller maskinkonto), inte ägarens personliga konto, så att ägaren kan godkänna agentens pull requests [V2-03]. Smal token (`Contents` + `Workflows` write, bara detta repo) om maskinkonto används.
2. Supabase: **[S0.0]** projekt `staging`. **[production]** projekt `production` (Pro).
3. Railway: **[S0.0]** miljö `staging` med **tre** tjänster: API, worker och `app_deleter` (S0.7). **[production]** motsvarande miljö för production.
4. Cloudflare: **[S0.0]** Pages-projekt och R2-bucket för staging. **[production]** bucket för production.
5. API-nyckel hos AI-leverantör (Anthropic). **[S0.0 – staging]** nyckel för staging, används bara med syntetiska data tills C1 är dokumenterad. **[production]** nyckel för production skapas **först efter** att C1 är dokumenterad i `DECISIONS.md` [V2-11, V3-01].
6. Hemligheter läggs i GitHub Environments (`staging`, `production`) och plattformarnas secret stores. **Aldrig i chatten eller i repot.**
7. **[production]** GitHub Environment `production` ska kräva manuellt godkännande (required reviewer = ägaren).
8. Branch protection på `main`: PR krävs, CI krävs, **Require review from Code Owners**, **Do not allow bypassing** (inklusive administratörer), inga force-pushar, inga direkt-pushar [SEC-10, V2-03]. Kräver att agenten har egen identitet (punkt 1). Under planeringsfasen, då inga kodändringar sker, gäller Required approvals = 0 och ägarens manuella merge som kontroll.
9. Slå på GitHub secret scanning och push protection [SEC-15].
10. OB-1, OB-5 och OB-6 är **beslutade** 2026-10-09 (`docs/DECISIONS.md`). Kvarstående villkor C1–C4 finns i den filen.
11. Supabase: lägg aldrig appens tabeller i Husappens projekt. Staging-projektet är separat [SEC-01].

---

## 2. Mänskliga kontrollpunkter

Agenterna kör själva däremellan.

| # | Punkt | Vem |
| --- | --- | --- |
| H1 | Godkänn denna plan efter Security Reviewer-granskning | Ägare |
| H2 | Skapa konton och lägg hemligheter (avsnitt 1) | Ägare |
| H3 | Godkänn att Stage 0 exit gate är grön i staging. Production-deploy sker först via readiness-gaten (4A), inte här [K1] | Ägare |
| H4 | Godkänn production-deploy efter Stage 1C (första marknadsdugliga leveransen) | Ägare |
| H5 | Besluta OB-1, OB-5 och OB-6 före S0.0 – **genomförd 2026-10-09** (`DECISIONS.md`) | Ägare |
| H7 | Godkänn ADR-015 och ADR-016 – **genomförd 2026-10-09** (`DECISIONS.md`) | Ägare |
| H9 | Godkänn integritetsinformationen (S0.8) före första kunddata | Ägare |
| H8 | Bekräfta att branch protection, Code Owner-krav och secret scanning är på (skärmdump) | Ägare |
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
- ingen bypass-, mock- eller demokod har tillkommit,
- **varje task har en rad `Säkerhetstester` med minst ett negativt fall** (fel roll, fel tenant, fientlig indata eller ogiltigt tillstånd) [P1],
- **katalogtestet för RLS-täckning är grönt:** **varje** tabell i app-schemat (inte bara de med `company_id`/`organization_id`) har RLS påslaget, `FORCE ROW LEVEL SECURITY`, minst en policy och inga grants till `anon`, `authenticated` eller `PUBLIC`. Undantag (t.ex. rena uppslagstabeller) står i en undantagslista under CODEOWNERS med motivering per rad. Testet räknar upp tabellerna dynamiskt och genererar ett cross-tenant-test per tabell [V2-01, V3-02, K4],
- **raderingsregistret är komplett:** varje tabell i app-schemat finns i raderingsregistret eller i en motiverad undantagslista under CODEOWNERS (S0.7) [V2-02, V3-02],
- dependency-audit, secret-skanning och containerskanning är gröna utan HIGH/CRITICAL (undantag endast via fil under CODEOWNERS) [V2-05].

---

## 4. STAGE 0 – Foundation

### S0.0 Cloud provisioning
- **Mål:** Tomt men deploybart skelett live i **staging** via CI/CD. Production-deploy tas först via readiness-gaten (4A) [K1].
- **Filer:** `.github/workflows/ci.yml`, `deploy-staging.yml`, `deploy-production.yml` (skapas men kan inte köras förrän production finns och ägaren godkänner), `apps/api/main.py` (endast `GET /api/v1/health`), `Dockerfile`, `infra/README.md` (vilka tjänster, vilka secret-namn, inga värden).
- **Beroenden:** H2.
- **Säkerhetskontroller [SEC-10, SEC-17]:** `CODEOWNERS` som kräver ägarens granskning för `.github/workflows/`, `migrations/`, `infra/`, `core/security.py`, RLS-policyer, bypass-scan-konfiguration och säkerhetstester. Actions låsta till commit-SHA, `permissions:` minimala, inget `pull_request_target` med PR-kod, deploy-tokens per miljö med minsta behörighet. PR-previews får aldrig använda staging- eller production-hemligheter.
- **Beroenden (ytterligare):** OB-1, OB-5 och OB-6 beslutade (H5, klart), ADR-015 och ADR-016 godkända (H7, klart), inställningar bekräftade (H8) [P2, V2-03].
- **Första PR [K7, V3-04]:** en liten pull request med **enbart** `CODEOWNERS`, som ägaren mergar manuellt innan någon PR med workflows. GitHub läser `CODEOWNERS` från basgrenen, så den första PR:en skyddas inte av filen. H8-beviset ska visa att filen finns.
- **Säkerhetstester:** deploy till production går inte att köra utan environment-godkännande. PR från fork får inga hemligheter.
- **Inställningar som ska verifieras (H8):** Code Owner-granskning krävs, bypass förbjudet även för administratörer, force-push blockerat, secret scanning och push protection på (om repot görs privat och funktionen kräver betald plan: `gitleaks` i CI som fallback), `CODEOWNERS` skyddar även sig själv och `.github/`. Verifier noterar bekräftelsen som bevis.
- **Gate:** `GET /api/v1/health` ger 200 över HTTPS i **staging**, deployat av CI/CD. (Production tas när det skapats, före pilot.) Agenten får aldrig ändra säkerhetstester för att få en gate grön. Sådan ändring rapporteras som BLOCKER.

### S0.1 Repository
- **Mål:** Hela mappstrukturen enligt §48 och verktyg.
- **Filer:** `docs/verification/` (bevismall per checklistrad, V3-05), `pyproject.toml` med låst lockfile, `src/business_auditor/...` (tomma moduler med `__init__.py`), `tests/{unit,integration,contract,golden,e2e,architecture}`, `scripts/bypass_scan.py`, `tests/architecture/test_import_rules.py`, `apps/web` (Vite + React + TS + Tailwind), `docs/adr/`, dokumentationsfilerna från §108 (skelett).
- **Tester:** Arkitekturtest (Domain får inte importera FastAPI, SQLAlchemy, AI- eller storage-SDK:er). Bypass-scan (förbjudna mönster i `src/` och `apps/`: `DISABLE_AUTH`, `DEBUG_USER`, `Mock*Provider`, `FakeStorage`, `localhost`, `docker-compose`, m.fl.).
- **Utökat [SEC-09, SEC-15]:** Bypass-scan täcker även `infra/`, `scripts/`, `.github/workflows/` och `Dockerfile`. Scan som misslyckas om hemligheter (service role, R2-nycklar, AI-nyckel) finns i frontend-bygget eller `VITE_*`-variabler. Import-linter förbjuder import av testdubletter från `src/`. **Loggredaktor** i `core/logging.py` med fält-allowlist, plus kanarietest (kända hemliga strängar får aldrig synas i logg). Förbjudet i logg: signerade URL:er, delade Sheets-länkar, sample values, prompts/AI-svar, originalfilnamn, request-/response-bodies, `jobs.payload`, JWT, **parserfel som citerar cellinnehåll** [K6].
- **Utökat [V2-05, V2-12, V2-14, V2-16]:** `core/logging.py` (flyttad hit från S0.2). Dependency-audit för Python och frontend, secret-skanning i CI, containerskanning, lockfil även för frontend. `.dockerignore` som utesluter `fixtures/`, `tests/` och `docs/`, plus CI-test som visar att imagen inte innehåller dem. Bypass-scanens undantag i en fil under CODEOWNERS med motivering per rad (t.ex. `localhost` i `HEALTHCHECK`). Dokumentskelett inkluderar underbiträdeslista i `SECURITY.md` och retention hos AI-leverantör och plattformar. `customer_reference_hash` använder HMAC med nyckel i secret store/KMS, roteras och förstörs vid radering av företaget.
- **Säkerhetstester:** log-canary, bypass-scan med positiva och negativa fall, test att imagen saknar `fixtures/`.
- **Gate:** CI: `lint`, `typecheck`, `tests`, `bypass-scan`, `architecture-test`, `log-canary-test`, `dependency-audit`, `secret-scan`, `container-scan`, `container-build` alla gröna.

### S0.2 Configuration
- **Mål:** Typade inställningar som vägrar starta utan obligatoriska hemligheter.
- **Filer:** `core/config.py`, `core/errors.py` (loggmodulen ligger i S0.1).
- **Tester:** Saknad secret → startup failure. Hemligheter förekommer aldrig i loggutskrift.
- **Säkerhetstester:** konfigurationsfel exponerar aldrig hemliga värden i felmeddelanden, och en miljö kan inte starta med en annan miljös nycklar (staging-nyckel i production vägras).
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
- **Utökat [V2-07, V2-17]:** Policyer även för `organizations`, `organization_memberships` och `company_access` via en andra variabel `app.current_user_id`. Definierad väg för att skapa första organisationen (insert tillåts när `created_by = current_user_id`; `created_by` läggs till i blueprint §9.1 via v1.2; ägaren blir första medlem som `owner` i samma transaktion). `companies` använder en medlemskapsbaserad policy (användaren har medlemskap eller `company_access`), eftersom `GET /companies` listar flera företag och inte passar en policy på ett enda `app.current_company_id` [V3-05, K8]. Händelsekatalog för `audit_log` (inloggningar och misslyckade inloggningar, utfärdade signerade URL:er, rapportnedladdningar, mappningsbekräftelse, flaggändringar, raderingsverifiering) och retention **24 månader (OB-10)**. Inloggningshändelser hos auth-leverantören når `audit_log` via auth-hook eller periodiskt jobb (mekanismen verifieras mot aktuell Supabase-dokumentation) [K6]. Testkontohjälparen för staging ingår här (ADR-016).
- **Säkerhetstester:** fail-closed även för organisationsnivå, katalogtestet för RLS-täckning (avsnitt 3), cross-tenant per tabell. **ADR-016-testerna [V2-13, V3-03, K5]:** testhjälparen vägrar köra när miljön är production, dess hemlighet finns bara i E2E-jobbets miljö (inte i PR-jobb från forks), den importeras aldrig från `src/`, och den har en motiverad rad i bypass-scanens undantagsfil.
- **Gate:** Migrationer, Data API-test (publik nyckel + testkonto nekas på varje tabell, tabeller räknas upp dynamiskt), RLS fail-closed-test, pool-läckagetest (tenant-kontext följer inte med mellan requests), `rolbypassrls = false` för alla runtime-roller. Mönstret verifierat mot aktuell Supabase-dokumentation.

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
- **Utökat [V2-08, V2-17]:** `company_access` genomdrivs i auktoriseringsfunktionen. Matrisen får två extra dimensioner: (a) medlem begränsad via `company_access`, (b) annan användare i samma företag (chattrådar privata enligt OB-8). Skydd mot kontouppräkning vid registrering och lösenordsåterställning. Beslut om var token lagras i frontend och hur XSS hanteras (CSP). Rate limiting definierad per endpoint och nivå.
- **Säkerhetstester:** viewer försöker skriva, begränsad medlem når inte företag utanför `company_access`, användare B läser inte användare A:s chattråd.
- **Gate:** `Authentication PASS`, `Tenant isolation PASS` (hela route-matrisen med utökade dimensioner, inte ett enskilt test).

### S0.5 Storage
- **Mål:** `ObjectStorage`-adapter mot R2 med signerade upp- och nedladdnings-URL:er, nyckelstruktur `organization/company/import/file`.
- **Regler:** Versionering avstängd eller permanent rensning för kundfiler (§66, §107).
- **Säkerhetskrav [SEC-05, SEC-21]:** Servern genererar hela nyckeln (`org/company/import/<uuid>`), klientens filnamn är bara sanerad metadata. TTL ≤ 10 min för uppladdning och ≤ 5 min för nedladdning (`Content-Disposition: attachment`, ingen CDN-cache). Vid registrering gör servern `HEAD` och kontrollerar storlek, och beräknar själv `sha256` i `SCANNING`. Lifecycle-regel tar bort oregistrerade objekt efter 24 h. Rate limit på `upload-url`.
- **Utökat [V2-15]:** Objekt som överskrider storleksgränsen raderas direkt vid registreringen. Ofullständiga multipart-uppladdningar avbryts av lifecycle-regel. Verifiera mot R2:s dokumentation om signerade URL:er kan begränsa längd och typ.
- **Tester:** överstort objekt raderas vid registrering, upload, read, delete, signerad URL går ut, åtkomst över tenantgräns nekas, användare A kan inte få signerad URL till B:s nyckel.
- **Gate:** `Storage PASS` mot riktig R2-bucket i staging.

### S0.6 Worker
- **Mål:** PostgreSQL-jobbkö med `FOR UPDATE SKIP LOCKED`, retry 30 s / 2 min / 10 min, återlämning av låsta jobb efter timeout.
- **Filer:** `apps/worker/main.py`, jobbtabell, `GET /api/v1/jobs/{id}`.
- **Säkerhetskrav [SEC-02, SEC-20]:** Workern kör som `app_worker` (ingen BYPASSRLS). Tenant-kontext tas från `jobs.company_id`, aldrig från `payload`. Payload med avvikande `company_id` avvisas. `DELETE_COMPANY_DATA` kan bara skapas av API-endpointen efter rollkontroll och är idempotent. `last_error` innehåller bara felkod och tekniskt meddelande.
- **Tester:** Concurrency-test (två workers tar aldrig samma jobb). Valideringsfel retry:as inte. Max attempts → `FAILED`. Payload med fel `company_id` avvisas.
- **Gate:** `Worker PASS`. Verifiera anslutningsläge mot Supabase (session-läge, inte transaction-pooler om det ger problem).

### S0.7 Radering och operatörsroll [V2-02, V2-09, V2-12]
- **Mål:** Kunden kan radera företag, organisation och användarkonto. Radering bevisas, inte bara påstås.
- **Filer:** `application/commands/delete_company.py`, raderingsregister (lista över alla tabeller med `company_id`), schemalagd rensare, verifieringsjobb, separat process eller tjänst för `app_deleter` (inte workern), operatörsroll.
- **Innehåll:**
  - `DELETE_COMPANY_DATA` raderar databasrader, storage per prefix (listning, inte kända nycklar), rapporter, chatt, härledda data och HMAC-nyckel för företaget. Auth-användare och organisation raderas på begäran.
  - Rader med `deleted_at` döljs av RLS och repositories. Rensaren tar bort permanent efter 7 dagar (OB-7).
  - Verifieringsjobb bekräftar att inga rader eller objekt med företagets id finns kvar och loggar resultatet utan affärsdata.
  - `app_deleter` körs i en egen process med egna uppgifter och minsta R2-behörighet. Workern får inte raderingsbehörighet. Om det inte går: ADR som beskriver hur två anslutningar hålls isär.
  - Operatörsroll (OB-9): egen roll, MFA-krav, tidsbegränsad åtkomst, auditloggad. Global feature flag-skrivning är dold tills rollen finns (§0B).
- **Säkerhetstester:** ingen kundroll (`owner` inkluderad) kan skriva en global feature flag [V3-05], användare utan rätt roll kan inte radera. Raderingstest per tabell via registret. Täckningstest som fallerar om en tabell med `company_id` saknas i registret. Verifieringsjobbet fallerar om ett objekt ligger kvar.
- **Gate:** Radering av ett testföretag i staging lämnar noll rader och noll objekt, och verifieringen är loggad. Registrets täckningstest är grönt. Nya tabeller i senare stages måste registreras (gäller i definition of done).

### S0.8 Integritetsinformation och underbiträdeslista [V3-01, K2]
- **Mål:** Kunden får korrekt information om vem som behandlar deras data, innan någon kunddata laddas upp.
- **Filer:** Integritetsinformation (svenska och engelska) som visas före första uppladdning, `SECURITY.md` med underbiträdeslista (Supabase, Railway, Cloudflare, Anthropic) och retention per mottagare.
- **Beroende:** `DECISIONS.md` C1 (kontroll av Anthropics villkor och DPA mot aktuell dokumentation) är dokumenterad, annars kan texten inte skrivas korrekt.
- **Säkerhetstester:** UI visar informationen och kräver bekräftelse före första uppladdning. Underbiträdeslistan i `SECURITY.md` stämmer mot de leverantörer som kod och infrastruktur faktiskt använder (kontrolleras av Verifier).
- **Gate:** Texten är granskad av ägaren (H9) och verifierad av Verifier. Krävs före Production readiness-gaten, inte före Stage 0 exit gate.

### STAGE 0 EXIT GATE (§74)
`Authentication`, `Tenant isolation` (utökad route-matris, katalogtest för RLS-täckning, Data API-test, RLS fail-closed inkl. organisationsnivå, pool-läckage, storage-prefix), `Migrations`, `Worker`, `Storage`, `Deletion` (S0.7), `CI` (inkl. dependency-, secret- och containerskanning) och `Deployed in staging` ska alla vara PASS. Production-deploy tas enligt ADR-015 före första kunddata.
→ Security Reviewer granskar. Verifier kör hela sviten. **H3.**

---

## 4A. Production readiness-gate (ADR-015) [V2-04]

Ska vara grön innan **någon** kunddata laddas upp, och innan H4:
- Production-miljö skapad (Supabase Pro, Railway, R2) med egna hemligheter. Inga staging-nycklar fungerar där.
- Stage 0 exit gate-testerna körs mot production: Data API-test, RLS fail-closed, route-matris (läsande), storage-prefix, radering.
- GitHub Environment `production` kräver ägarens godkännande.
- Backup aktiverad och en återställning testad (§107).
- **C1 dokumenterad** i `docs/DECISIONS.md` (Anthropics villkor och DPA kontrollerade mot aktuell dokumentation, datum, vilka dokument, utfall) [V3-01].
- **AI-nyckeln för production skapas först efter C1** och ligger då i secret store. Före det kan ingen kunddata nå leverantören.
- **Integritetsinformation och underbiträdeslista** klara (S0.8, H9).
- **Testkonto i production:** skapas genom ordinarie registrering med minimala rättigheter, används av 4A-testerna och raderas med S0.7:s raderingstest. ADR-016:s admin-API-hjälpare används aldrig här.
- Verifieringsrapport från Verifier och Security Reviewer.

---

## 5. STAGE 1A – Import (§75)

| Task | Mål | Nyckelkrav |
| --- | --- | --- |
| S1A.0 | SCANNING och fientliga fixtures | Magic-byte-kontroll, ändelse/MIME-matchning, filnamnssanering, `.xlsm` och andra makroformat avvisas, gränser för sidor, bildmått och storlek tvingade i workern. Fixtures i `tests/`: zip-bomb, XXE, pixelbomb, fel filtyp. Gate för S1A.2–S1A.5. Överväg att inte stödja `.xls` om parsern inte kan sandlådas [V2-06, P7]. |
| S1A.1 | `ImportBatch`, `ImportFile`, `import_status_history`, state machine | Alla övergångar loggas. Inga tysta fel (§7). Uppladdning via signerad URL (§55). **Säkerhetstester:** ogiltig tillståndsövergång nekas, viewer kan inte skapa import, import till annat företag nekas. `import_status_history` får `company_id` (blueprint v1.2, §11A). |
| S1A.2 | CSV-parser | Chunkad (§98). Teckenkodning och avgränsare detekteras. Gräns för radlängd och fältstorlek [SEC-06]. |
| S1A.3 | XLS/XLSX-parser | Flera ark. `defusedxml`, skydd mot zip-bomber, `.xlsm` och andra makroformat avvisas [SEC-06]. |
| S1A.4 | PDF-textextraktion | Sid-, storleks-, tids- och minnesgräns, tvingade på serversidan i workern [SEC-06]. |
| S1A.5 | Bild/dokument-extraktion via adapter | Pydantic-validerad output. Bakgrundsjobb. `MAX_IMAGE_PIXELS` får inte stängas av. Personnummer/organisationsnummer maskas innan lagring. Användaren informeras om att bild/PDF skickas till extern AI-leverantör [SEC-06, SEC-08]. **Säkerhetstester [K5, V3-01]:** injektionsset (dokument med dolda instruktioner) måste passera med noll externa anrop och noll data till annat företag, svar som bryter schemat avvisas, personnummer maskas. Endast syntetiska data skickas till AI tills C1 är dokumenterad. |
| S1A.6 | Google Sheets via delad länk | **SSRF-skydd [SEC-07]:** allowlist per hopp (`docs.google.com` → högst 2 redirects till `*.googleusercontent.com`, endast `https`, port 443, verifiera mot verkligt beteende). Egen resolver med IP-kontroll (privata, loopback, link-local, metadata, IPv4+IPv6), skydd mot DNS rebinding, tid- och storleksgräns, körs i workern. Länken loggas aldrig. Negativa tester: IP-literal, `localhost`, `169.254.169.254`, `[::1]`, redirect till intern host, `docs.google.com@evil`. |
| S1A.7 | Dokumentklassificerare | Okänt → `UNKNOWN`, aldrig gissning. **Säkerhetstester:** fientligt filnamn eller innehåll påverkar inte klassificeringen och körs aldrig som kod. |
| S1A.8 | Kolumnmappning nivå 1–3 | Tröskelvärden enligt §8. Max 10 sample values, personuppgiftskolumner maskas (§99). Filinnehåll märks som otillförlitlig data i prompten. AI-mappning av finansiellt kritiska fält visas alltid för användaren. Injektionsset med förväntat utfall (noll tool-anrop mot annat företag, noll externa länkar) måste passera [SEC-04, P9, V2-03]. |
| S1A.9 | UI för bekräftelse av mappning | Enkel för icke-teknisk användare. **Säkerhetstester:** viewer kan inte bekräfta mappning, användare i annat företag nekas, finansiellt kritiskt fält kan inte auto-bekräftas i UI. |
| S1A.10 | Datakvalitetsrapport | `GOOD` / `USABLE_WITH_WARNINGS` / `POOR` / `INSUFFICIENT`. **Säkerhetstester:** rapport för annat företag nekas, AI får inte uttrycka starka slutsatser vid `INSUFFICIENT`. |

**Exit gate:** Golden-filer för CSV, XLSX, PDF, bild och Google Sheet samt mappning ger förväntat resultat. Deployad i staging. **Security Reviewer granskar före Stage 1B startar** (avsnitt 9) [V2-10].

---

## 6. STAGE 1B – Normalisering (§76)

Ordning: products → suppliers → sales → purchases → inventory → financial statements → recipes → waste.

- Idempotent. Dubblettfil avvisas via `sha256` + `company_id`.
- Dubbletter inom fil flaggas i kvalitetsrapporten, kasseras inte tyst.
- Korrigerad omuppladdning: varna, fråga, markera `superseded_by` (§14). Inget raderas.
- Delsummor lagras med `is_subtotal` och räknas om (§12).
- Okänd enhet → datakvalitetsvarning (§13).
- **Säkerhetstester:** katalogtestet för RLS-täckning och raderingsregistret omfattar alla nya tabeller. Dubblettkontroll över företagsgräns läcker inte att en fil finns hos ett annat företag. Ersatt batch (`superseded_by`) är fortfarande läsbar men syns inte i ny analys.

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
- **Säkerhetstester:** dashboard, findings och evidence för annat företag nekas, viewer kan läsa men inte köra audit, ett finding utan evidence kan inte skapas, tool- och API-svar innehåller pengar som decimal-strängar.

**Exit gate:** Golden retail-dataset ger förväntade findings. **Ingen LLM används för att passera gaten.** Deployad i staging.
→ **Första marknadsdugliga leveransen (§115A).** Security Reviewer + Verifier, sedan **H4.**

---

## 8. Resterande Stage 1

| Stage | Innehåll | Gate |
| --- | --- | --- |
| **1D** Restaurang (§78) | Receptkostnad, food cost, contribution margin, itemklassning, waste, actual vs theoretical (neutral formulering: *Unexplained variance detected.*) | Golden restaurant-dataset ger förväntade findings. **Säkerhetstester:** recept och waste för annat företag nekas, ingen findingtext innehåller anklagande formuleringar (test mot ordlista). |
| **1E** Forecast + scenario (§79) | Moving average, weighted MA, linjär trend, seasonal naive. Pris-, kostnads-, rea-, försäljnings-, inköps- och lönescenarier. Scenarier skriver aldrig till riktig data. **Säkerhetstester:** radantal i affärstabeller är oförändrade efter scenariokörning, viewer kan inte köra scenario eller forecast. | Deterministiska enhetstester. Antaganden och före/efter sparas. |
| **1F** AI CFO (§80) | `AIProvider`-interface, tool registry (§37), orchestrator, policies (§39), validator, chattpersistens, UI. **Tool-scheman får aldrig ha `company_id`/`organization_id` som argument**, tenant sätts på serversidan och tools använder samma auktorisering som API:t. Chattsvar renderas utan rå HTML och utan automatiska externa länkar. Deterministiskt CI-test som fallerar om något tool har argument som heter `company_id`, `organization_id` eller liknande. Injektionsset måste passera i 1F-gaten, inte bara sparas. Samma set, anpassat, gäller S1A.5. Chattrådar privata per användare (OB-8) har eget test. **Endast `AnthropicProvider` implementeras** (`DECISIONS.md`). Andra leverantörsklasser får inte finnas som tomma eller stubbade klasser (§0B). Ett registertest visar exakt en aktiv leverantör när `new_ai_provider` är av. **Beredskap för leverantörsbyte:** se avsnitt 9B (kontraktstester, checklista, funktion utan AI) [SEC-04, P9, V3-06, K8]. | AI-gate: *"Why did my margin fall?"* använder tool, rätt period, rätt siffror, nämner datagap, hittar inte på orsak. Evals sparade (§72). **Injektionssetet passerar** (noll tool-anrop mot annat företag, noll externa länkar), schematestet för tool-argument är grönt. |
| **1G** PDF (§81) | ReportLab (lås version ≥ 3.6.13, kontrollera senaste) som bakgrundsjobb, all användar- och AI-text escapas före `Paragraph`, test med fientlig produktsträng. Lagrad i R2, nedladdning via kort signerad URL [SEC-11, SEC-21]. | PDF innehåller alla sektioner (§42) med verifierad data. |
| **1H** Portal (§82) | Slutpolering och full sidtäckning. | **Final acceptance (§83)** mot production: skapa konto → företag → ladda upp → mappa → audit → fynd → actions → AI-fråga → scenario → PDF, utan utvecklaringrepp. **Säkerhetstester:** E2E körs även som `viewer` och som begränsad medlem och visar att de inte når skrivfunktioner eller andra företag. |

---


## 9A. Radering, E2E och operatörsåtkomst (från säkerhetsgranskningen)

- **Radering [SEC-14] (task S0.7):** `DELETE_COMPANY_DATA` täcker även storage per prefix (listning, inte bara kända nycklar). Organisation och användarkonto kan raderas. Fördröjd permanent radering = 7 dagar (OB-7). Rader med `deleted_at` döljs av RLS och repositories. Integritetsinformationen anger AI-leverantörens och plattformarnas retention.
- **E2E utan bypass [SEC-16, ADR-016]:** En testhjälpare som bara finns i `tests/e2e/` skapar konton via auth-leverantörens admin-API med en staging-begränsad hemlighet. Den är testinfrastruktur, inte en produktväg, och fungerar aldrig mot production. Testerna för detta ligger i S0.3 [V3-03].
- **Operatör [SEC-18]:** Feature flags med global scope får bara ändras av operatörsroll (break-glass, MFA, auditloggad). Ingen kundroll får det.
- **Staging:** Ingen kopiering av production-data till staging i Stage 1 (golden datasets räcker) [SEC-24].
- **`customer_reference_hash`:** HMAC med hemlig nyckel per företag, inte vanlig hash [SEC-08].

## 9B. Beredskap för leverantörsbyte (AI)

Mål: Anthropic är enda aktiva leverantör i Stage 1, men det ska gå att byta till eller lägga till ChatGPT/OpenAI, Gemini eller annan på kort tid om Anthropic skulle ändra villkor, begränsa användning eller bli otillgängligt. Detta byggs in från början, men ingen andra leverantör aktiveras förrän den är godkänd.

1. **Leverantörsneutralt gränssnitt.** All AI-kod ligger bakom `AIProvider` (blueprint §36). Inga modellnamn, SDK-typer eller svarsformat läcker utanför `infrastructure/ai/`. Prompter, tool-scheman och structured-output-scheman är leverantörsneutrala och versionerade (`prompt_version`).
2. **Kontraktstestsvit.** `tests/contract/ai_provider/` innehåller en gemensam testsvit som **varje** leverantör måste passera: giltig structured output (Pydantic), tool-anrop och svar i rätt ordning, felmappning (tidsgräns, rate limit, avvisat innehåll) till våra felkoder, inga prompter eller kunddata i loggar, och injektionssetet. Sviten körs i CI mot gränssnittet med en testdubblett som bara finns under `tests/` (tillåtet enligt §0B). Det bevisar att gränssnittet är leverantörsneutralt utan att en andra leverantör måste vara implementerad.
3. **Checklista för ny leverantör** (`docs/AI_PROVIDER_CHECKLIST.md`, skapas i 1F): (a) villkor, DPA, ingen träning på kunddata, retention och EU-behandling kontrollerade mot aktuell dokumentation och dokumenterade i `DECISIONS.md`, (b) underbiträdeslistan i `SECURITY.md` och integritetsinformationen uppdaterade (S0.8), (c) implementation i `infrastructure/ai/`, (d) kontraktstestsviten och AI-evals (golden datasets) gröna, (e) aktivering via feature flag `new_ai_provider` och ägarens godkännande. Först därefter får kunddata skickas dit.
4. **Ingen automatisk reservväxling.** Systemet byter aldrig tyst leverantör vid fel, eftersom kunddata då kunde gå till en mottagare som kunden inte fått information om. Byte är ett medvetet, loggat beslut av ägaren/operatören.
5. **Kärnan fungerar utan AI.** Metrics, findings, forecast, scenarier, rapporter och kolumnmappning nivå 1–2 är deterministiska och beror inte på någon AI-leverantör (ADR-002). Om AI inte är tillgängligt ska import av CSV/XLSX med känd mappning och all analys fortsätta fungera, medan AI-funktioner (nivå 3-mappning, bild- och PDF-extraktion, chatt) ger ett tydligt fel enligt §62, inte en låtsasrespons (§0B).
6. **Säkerhetstester:** kontraktstestsviten ovan, ett test som visar att analys och rapport körs med AI avstängt, och ett test som visar att en inaktiv leverantör inte kan väljas via konfiguration i staging eller production.

## 9. Löpande verifiering (alla stages)

- **Verifier** körs efter varje stage-gate: golden regression, bypass-scan, arkitekturtest, tenant-negativtester, E2E mot staging.
- **Security Reviewer** körs efter Stage 0, **efter 1A (före 1B)**, efter 1C, efter 1F, **efter 1G** och före production-release [V2-10].
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
| P13 | Radering täcker org, användare och storage-prefix | S0.7 |
| P14 | Auditlogg: endast INSERT, ingen cascade, definierat schema | S0.3 |
| P15 | E2E-autentisering utan bypass | S0.3, ADR-016 |
| P16 | Inga localhost, docker-compose, mock eller seed i production | Alla tasks, bypass-scan |

**Bevis:** Verifier fyller i CI-körning och testnamn per rad vid varje gate i `docs/verification/`. En rad räknas som uppfylld först när beviset finns [V2-nn, avsnitt 3 i V2-rapporten].

---

## 12. Status mot omgranskningen (V2-01–V2-17)

| Fynd | Åtgärdat i v3 | Var |
| --- | --- | --- |
| V2-01 RLS-täckning | Katalogtest i gemensamma gates | Avsnitt 3, S0.3 |
| V2-02 Radering och operatör | Ny task S0.7 | S0.7 |
| V2-03 CODEOWNERS | Code Owner-krav, egen agentidentitet, verifiering | Avsnitt 1, S0.0, H8 |
| V2-04 Production | ADR-015 och readiness-gate | Avsnitt 0, 4A |
| V2-05 Skanning | Dependency-, secret-, containerskanning | S0.1, avsnitt 3 |
| V2-06 SCANNING | Ny task S1A.0 med fixtures | S1A.0 |
| V2-07 Org-nivå RLS | `app.current_user_id`, bootstrap | S0.3 |
| V2-08 `company_access` | Utökad matris | S0.4 |
| V2-09 `app_deleter` | Separat process | S0.7 |
| V2-10 Granskningstillfällen | Efter 1A och 1G | Avsnitt 9 |
| V2-11 OB-1 | Ett tillfälle före S0.0, kriterier i tabellraden | Avsnitt 0 |
| V2-12 GDPR-dokument | Underbiträdeslista, HMAC-nyckelhantering | S0.1, S0.7 |
| V2-13 E2E-konton | ADR-016 | S0.3, 9A |
| V2-14 Bypass-scan | `.dockerignore`, undantagsfil | S0.1 |
| V2-15 Uppladdningsstorlek | Radera objekt som överskrider gränsen, avbryt ofullständiga multipart | S0.5 |
| V2-16 Ordning | Logg flyttad till S0.1, testkontohjälpare i S0.3 | S0.1, S0.3 |
| V2-17 Rester av SEC-12/13 | Kontouppräkning, tokenlagring, händelsekatalog | S0.3, S0.4 |


---

## 13. Status mot granskningen av v3 (K1–K8, V3-01–V3-06)

| Punkt | Åtgärdat i v3.1 | Var |
| --- | --- | --- |
| K1 Textsynk med ADR och beslut | Ägarlistan delad i [S0.0]/[production], S0.0-mål, H3, H5/H7 genomförda, statusrad, OB-5 skärpt, tredje Railway-tjänst | Avsnitt 0, 1, 2, S0.0 |
| K2 / V3-01 Readiness-gate | C1, AI-nyckel först efter C1, integritetsinformation (ny S0.8, H9), testkonto i production | 4A, S0.8 |
| K3 Säkerhetstester per task | Rad tillagd i alla tasks som saknade den | S0.0, S0.2, S1A.1–S1A.10, 1B–1H |
| K4 / V3-02 Alla tabeller | RLS-test och raderingsregister gäller hela schemat med undantagslista, `import_status_history` får `company_id` | Avsnitt 3, S1A.1 |
| K5 / V3-03 | ADR-016-testerna schemalagda, injektionskrav i S1A.5 och 1F-gate | S0.3, S1A.5, 1F |
| K6 Rester av V2-17 | Parserfel i loggförbud, retention 24 månader (OB-10), mekanism för inloggningshändelser | S0.1, S0.3 |
| K7 / V3-04 | Första PR med enbart `CODEOWNERS`, blueprint v1.2-ändringslogg | S0.0, blueprint |
| Ägarens önskemål: öppet för andra AI-leverantörer | Avsnitt 9B: leverantörsneutralt gränssnitt, kontraktstestsvit, checklista, ingen auto-växling, kärnan fungerar utan AI | 1F, 9B |
| K8 / V3-05, V3-06 | `companies`-policy, bootstrap, operatörstest, checklistpekare, endast `AnthropicProvider` med registertest | S0.3, S0.7, avsnitt 11, 1F |
