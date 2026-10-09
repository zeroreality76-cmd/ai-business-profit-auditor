# PROJECT ARCHITECTURE BLUEPRINT

AI Business Profit Auditor / AI CFO

| Egenskap | Värde |
| --- | --- |
| **Dokumentstatus** | Implementation-Ready |
| **Version** | 1.1 (omskriven till Markdown; ersätter RTF v1.0) |
| **Primärt mål** | Bygga en pilotklar produkt som körs live i molnet och kan testas på 3–10 verkliga företag innan större integrationsarbete görs. |
| **Primär marknad** | Små och medelstora företag internationellt. |
| **Första djupa vertikaler** | Retail/e-commerce samt restaurang/café. |
| **Övriga företag** | Generell företagsanalys från Stage 1; specialiserade regler läggs till stegvis. |
| **Första valuta** | SEK |
| **Valutaarkitektur** | Multi-currency från första databasmigrationen. |
| **Slutmål** | Persistent, datadriven AI-CFO som analyserar hela verksamheten, kommer ihåg historik, förutser utveckling, simulerar beslut och senare kan verkställa godkända åtgärder. |

> **Numrering:** Sektionsnumren (0–117) är oförändrade från v1.0 så att tidigare referenser fortfarande stämmer. Nya sektioner har suffix (t.ex. 0A, 11A).

---

## Ändringslogg v1.0 → v1.1

### Nya, bindande regler (efter projektägarens beslut)

1. **Allt byggs, körs och testas i molnet.** Ingenting får kräva att projektägarens eller utvecklarens egen dator kör något. Se §0A.
2. **Allt byggs för direkt, riktig användning.** Inga bypass-, demo-, mock- eller testlägen i produktkoden. Se §0B.
3. **Projektet levereras i korta etapper som alla deployas live.** Se §0C.

### Borttaget ur v1.0 (var i konflikt med reglerna ovan)

- `docker-compose.yml` och mappen `docker/` (lokal körning).
- §109 "README STARTUP" (clone → starta lokalt) – ersatt av molnbaserad uppstart.
- `OllamaProvider` (kräver lokal modellkörning).
- Miljön `development` (var i praktiken en lokal miljö) – ersatt av `staging` + `production` (+ valfria PR-previews i molnet).

### Rättade fel och motsägelser i v1.0

| # | Problem i v1.0 | Åtgärd i v1.1 |
| --- | --- | --- |
| 1 | §25, §38, §110 visade pengar som JSON-tal (`18000`, `34.2`), vilket bryter mot §101 (aldrig float för pengar). | Pengar och procent skrivs som **decimal-strängar** i alla API-/tool-kontrakt. |
| 2 | §30 listade "theft" som möjlig orsak men förbjöd samtidigt anklagelser. | "theft" borttaget. Endast neutral formulering: *Unexplained variance detected.* |
| 3 | §4.2 och §9.4 definierade olika fält för Company (`business_type`/`industry_profile` vs `industry`). | Ett enda fältset i §9.4. |
| 4 | §51 kräver `organization_id` + `company_id` på all affärsdata, men barntabeller (SalesLine m.fl.) saknade `company_id`. | Regel i §11A: `company_id` denormaliseras på alla barntabeller. |
| 5 | §107 "object storage versioning" krockar med §66 (radering måste verkligen radera). | Versionering **avstängd** för kundfiler; se §66 och §107. |
| 6 | §12 blandade delsummor (GROSS_PROFIT, EBITDA …) med atomära rader → risk för dubbelräkning. | Delsummor flaggas `is_subtotal` och räknas om av motorn; avvikelse → datakvalitetsvarning. |
| 7 | §8: "finansiellt kritiska kolumner får högre tröskel" utan värde. | Konkret standardvärde (se §8). |
| 8 | §19 använder `product_age` utan definition. | Definierad i §19. |
| 9 | §21/§22 saknade standardvärden för lead time, safety days m.m. | Konfigurationstabell i §21A. |
| 10 | §27 prioritetsformel utan mappning till severity. | Definierad som öppet beslut OB-3 samt tillfällig standardregel. |
| 11 | Datamodellen saknade Audit, Finding, Metric, Forecast, Scenario, Report, Job, FeatureFlag, Chat m.fl. | Samlad i §11A–§13A. |
| 12 | Ingen regel för korrigerade omuppladdningar utan att skriva över historik. | `superseded_by` i §14. |
| 13 | §51 RLS är orealistiskt om backend ansluter med en roll som kringgår RLS. | Konkret mönster i §51. |
| 14 | §47 listar moduler (alerts, integrations, billing) utan stage. | Stage-märkning i §47. |
| 15 | API-listan i §61 saknade endpoints som UI och §28 kräver (dashboard, actions, evidence, signerade URL:er). | Tillagda i §61. |
| 16 | Uppladdning via API-servern strider mot §97 (<2 s ack) för 50 MB-filer. | Direkt-till-storage via signerad URL i §55. |
| 17 | Personuppgifter kan följa med till extern AI-leverantör via sample values. | Regler i §99. |

### Öppna beslut (kräver projektägarens bekräftelse – bygg-AI ska använda standardvärdet tills beslut finns)

| ID | Fråga | Standard i v1.1 |
| --- | --- | --- |
| OB-1 | Vilken AI-leverantör är default i Stage 1? | Bestäms vid S1F.2; kontrollera aktuell modell-/prisdokumentation då. |
| OB-2 | Behörighetsmatris per roll (§4.1). | Förslaget i §4.1. |
| OB-3 | Exakt formel som mappar prioritetspoäng till severity (§27). | Förslaget i §27. |
| OB-4 | Standardvärden för lead time/safety stock per vertikal (§21A). | Förslaget i §21A; valideras i pilot. |
| OB-5 | Datalagring/region för EU-kunder (Supabase/Railway/R2-region). | EU-region där leverantören erbjuder det. |

---

## 0. ABSOLUTA PRODUKTPRINCIPER

Dessa regler får inte ändras av bygg-AI utan uttryckligt arkitekturbeslut (ADR).

1. **LLM får aldrig vara källa till ekonomiska siffror.**
2. Alla ekonomiska beräkningar görs deterministiskt av Python.
3. AI tolkar, förklarar, prioriterar och kommunicerar resultat.
4. Varje viktig siffra måste kunna spåras tillbaka till importerad data.
5. Osäker data ska markeras som osäker, aldrig gissas.
6. Kunden ska kunna använda systemet utan teknisk kunskap.
7. Import ska acceptera kundens befintliga filer i så hög grad som möjligt.
8. Systemet ska be kunden bekräfta osäker kolumnmappning.
9. Historisk data ska aldrig skrivas över.
10. Varje import är en separat version/snapshot.
11. Multi-tenancy byggs från dag ett.
12. En organisation ska kunna äga flera företag.
13. Kunden ska kunna radera företag och underliggande data.
14. Plattformen byggs som en **modular monolith**, inte microservices.
15. Nya vertikaler ska läggas till som analysmoduler utan att kärnan modifieras.
16. Externa AI-leverantörer ska ligga bakom adapterinterface.
17. Externa lagrings-, OCR- och integrationsleverantörer ska kunna bytas.
18. Stage 1 måste vara bra nog att kunder kan bedöma affärsnyttan på riktigt.
19. Stage 1 får inte reduceras till en demo med några grafer.
20. Funktioner utan tydlig kundnytta skjuts upp.
21. **Allt byggs, körs och testas i molnet** (§0A).
22. **Allt byggs för direkt, riktig användning – inga bypass-, demo- eller mock-lägen** (§0B).
23. **Varje etapp avslutas med en live-deploy** (§0C).

---

## 0A. MOLNKRAV – INGET LOKALT (BINDANDE)

Projektet byggs, körs och verifieras **enbart i molnet**. Lokal körning är **förbjuden** som arbetssätt och som krav.

### Förbjudet

- Att något i produkten, bygget eller verifieringen kräver att projektägarens eller en utvecklares egen dator kör en server, databas, worker, modell eller tjänst.
- `localhost`/`127.0.0.1` som beroende eller som del av acceptanskriterier.
- `docker-compose` (eller motsvarande) som sätt att köra systemet.
- Lokalt hostade AI-modeller (t.ex. Ollama). AI-leverantörer ska vara molntjänster bakom `AIProvider`.
- Lokal filsystemlagring av kunddata (`LocalStorage`-adapter får inte finnas i produktkoden).
- README-instruktioner som beskriver lokal uppstart.
- Att acceptera en gate som endast passerat "lokalt".

### Krav för molnbygget

- All källkod ligger i ett molnhostat Git-repository.
- CI/CD körs i molnet (se §105) och **deployar automatiskt** till staging; production deployas efter release gate (§106).
- Databas (PostgreSQL), autentisering, objektlagring, API, worker och frontend körs på managed molnplattformar (§67).
- Alla tester som kräver beroenden (PostgreSQL, storage) körs i CI med molnbaserade service-containrar eller mot staging-miljöns separata testresurser.
- End-to-end-tester (§69) körs mot den **deployade** staging-miljön.
- Hemligheter finns enbart i plattformarnas secret stores och CI-secrets. De skrivs aldrig in i repot, ej heller i `.env`-filer som checkas in.
- Den bygg-AI som utför arbetet ska arbeta i en molnmiljö (t.ex. molnbaserad agent kopplad till repot). Den får inte instruera användaren att installera eller starta något på sin dator.
- Detaljer om plattformsbeslut: §67, §68, ADR-011.

> Notera: En `Dockerfile` är tillåten **enbart som byggrecept som molnplattformen använder**. Den är inte en instruktion att köra något lokalt.

---

## 0B. INGA BYPASS-, DEMO- ELLER MOCKLÄGEN (BINDANDE)

Produktkoden byggs som riktig produktionskod från första raden. Ingenting får finnas som "genväg för utveckling".

### Förbjudet i produktkod, konfiguration och miljöer

- **Auth bypass:** inga hårdkodade användare, ingen "skip login", ingen `DEBUG_USER`, ingen header eller query-parameter som kringgår autentisering eller auktorisering, ingen "admin-bakdörr".
- **Tenant bypass:** ingen kodväg som ignorerar `organization_id`/`company_id`-kontroll.
- **Demo mode / sandbox mode** som visar påhittad data för riktiga användare.
- **Mock-leverantörer** i drift: `MockAIProvider`, `FakeStorage`, `StubOCR` och liknande får inte vara valbara i staging eller production. (Testdubletter får finnas **endast** i testkod under `tests/` och aldrig importeras av `src/`.)
- **Fiktiva eller hårdkodade resultat**: inga placeholder-findings, inga "exempelsiffror" i UI eller API, inga stubbade endpoints som returnerar `200 OK` utan riktig funktion.
- **Flaggor som stänger av säkerhet** (`DISABLE_AUTH`, `SKIP_VALIDATION`, `ALLOW_ANY_CORS`, `INSECURE_MODE`).
- **Seed-data i production.** Golden datasets (§70–§71) används **endast** i automatiska tester.
- **Obegränsade testkonton** med särskilda rättigheter.
- **TODO-funktioner** som ser färdiga ut men inte är det.

### Krav för produktkoden

- Om en funktion inte är färdig ska den vara **osynlig eller avstängd via feature flag (§104)** – aldrig falsk.
- Om en extern tjänst saknas ska systemet ge ett **tydligt fel** (strukturerat enligt §62), inte en låtsasrespons.
- Startup ska **misslyckas** om obligatoriska hemligheter eller konfiguration saknas (S0.2).
- Testning av riktiga flöden görs med riktiga testkonton i staging, skapade genom den vanliga registreringen och styrda av samma regler som alla andra användare.
- CI ska innehålla en **bypass-scan** (§105) som misslyckas om förbjudna mönster hittas i `src/` och `apps/`.

---

## 0C. LEVERANSMODELL – KORTA ETAPPER, ALLTID LIVE

Projektet är designat för att byggas snabbt, i korta etapper, med en riktig produkt som kan visas för marknaden så tidigt som möjligt.

1. **Varje etapp slutar med en live-deploy** till staging och, efter release gate, till production.
2. Varje deployad etapp ska vara **användbar för verklig användare** inom sitt avgränsade scope (inte en halvfärdig vy).
3. Etapperna i §115 är ordningen. Ingen etapp hoppas över; gaten för varje etapp måste passera innan nästa påbörjas.
4. Första synliga, marknadsdugliga resultat: efter **Stage 0 + Stage 1A–1C** (inloggning, uppladdning, mappning, normalisering, första Money Leak-fynd) – se §115A.
5. Under bygget gäller: **"Kör tills gaten är grön."** Bygg-AI ska arbeta sig igenom byggordningen utan att vänta på nya instruktioner, och stanna endast vid **BLOCKER** (§114) eller när ett öppet beslut (OB-n) behöver svar.
6. Tidsuppskattningar i dagar anges inte i blueprinten. Ett separat **arbetsschema** (task för task, med kommandon, gates och förväntade filer) ska tas fram som komplement till detta dokument (se §115A).

---

## 1. PRODUKTVISION

Produkten ska svara på frågan:

> **Var förlorar mitt företag pengar, varför händer det, vad bör jag göra åt det och vad blir effekten om jag gör det?**

Produkten utvecklas från:

1. **Data Audit**
2. **Business Intelligence**
3. **AI Business Advisor**
4. **AI CFO**
5. **AI CFO med godkända actions**

---

## 2. KUNDENS KÄRNUPPLEVELSE

En ny kund ska kunna:

1. skapa konto,
2. skapa företag,
3. välja bransch,
4. ange basvaluta,
5. ladda upp befintliga filer,
6. låta systemet klassificera data,
7. bekräfta eventuellt osäkra mappings,
8. starta analys,
9. få ekonomiska fynd,
10. se prioriterad handlingsplan,
11. ladda ned PDF,
12. fråga AI om resultatet,
13. simulera beslut,
14. återkomma nästa månad,
15. ladda upp ny data,
16. automatiskt få periodjämförelser.

Kunden ska inte behöva förstå: databaser, Pandas, API, JSON, SQL, LLM, embeddings, forecasting-algoritmer.

---

## 3. STAGE-MODELL

**Stage 0 — Foundation.** Teknisk grund. Ingen kundpilot förrän Stage 0-gaten är godkänd.

**Stage 1 — Pilot-Ready Intelligence.** Första kommersiellt trovärdiga versionen. Målet är att 3–10 företag ska kunna använda systemet och ge ett rättvist svar på: *Är detta tillräckligt värdefullt för att betala för?*

**Stage 2 — Deep Intelligence.** Djupare CFO-funktioner, forecasting, avancerade vertikalanalyser, alerts och bättre dokumentförståelse.

**Stage 3 — Connected AI CFO.** Direkta integrationer till bokföring, e-commerce, POS och andra affärssystem.

**Stage 4 — Benchmark & Action Layer.** Benchmarking och godkända externa handlingar.

---

## 4. STAGE 1 — EXAKT SCOPE

### 4.1 Konto och organisation

- registrering
- inloggning
- logout
- password reset
- organisation
- organisation membership
- flera företag per organisation
- roller: `owner`, `admin`, `analyst`, `viewer`

**Behörighetsmatris (förslag, OB-2):**

| Åtgärd | owner | admin | analyst | viewer |
| --- | :-: | :-: | :-: | :-: |
| Ta bort organisation | ✔ | — | — | — |
| Hantera medlemmar och roller | ✔ | ✔ | — | — |
| Skapa/ändra företag | ✔ | ✔ | — | — |
| Radera företag | ✔ | ✔ | — | — |
| Ladda upp/bekräfta mapping/processa import | ✔ | ✔ | ✔ | — |
| Köra audit, scenario, forecast, rapport | ✔ | ✔ | ✔ | — |
| Använda AI-chat | ✔ | ✔ | ✔ | ✔ |
| Läsa findings, rapporter, dashboard | ✔ | ✔ | ✔ | ✔ |

Behörighet kan i Agency-läget (§93) begränsas **per company**.

### 4.2 Företagsprofil

Varje företag ska ha de fält som definieras i §9.4 (en enda definition). Branschtyper (`business_type`):

`ecommerce`, `retail`, `restaurant`, `cafe`, `wholesale`, `construction`, `workshop`, `salon`, `professional_service`, `generic`

Stage 1 har djupa analysregler för `ecommerce`, `retail`, `restaurant`, `cafe`. Övriga använder **Generic Business Analyzer**.

---

## 5. STÖDDA INPUTFORMAT

Stage 1 ska stödja: CSV, XLS, XLSX, PDF, PNG, JPG/JPEG, Google Sheets via delad länk.

Senare: Google OAuth Sheets, e-mail ingestion, API connectors, bank feeds, POS connectors, accounting connectors.

---

## 6. DOKUMENTTYPER

Systemet ska kunna klassificera:

| Grupp | Typer |
| --- | --- |
| **Sales** | försäljningsrapport, orderexport, POS-rapport, Shopify-export |
| **Purchases** | inköpsrapport, leverantörsfaktura, receipt, purchase ledger |
| **Inventory** | lagerlista, inventory snapshot, stock report |
| **Financial** | resultaträkning, balance sheet, cash-flow report, general ledger export, kostnadsrapport |
| **Restaurant** | recipe sheet, ingredient list, supplier invoice, waste report, menu sales |
| **Unknown** | `document_type = UNKNOWN` – systemet får inte gissa. |

---

## 7. IMPORT PIPELINE

Varje import går igenom exakt följande state machine:

```text
CREATED
  ↓
UPLOADED
  ↓
SCANNING
  ↓
CLASSIFYING
  ↓
EXTRACTING
  ↓
MAPPING
  ↓
NEEDS_CONFIRMATION (optional)
  ↓
NORMALIZING
  ↓
VALIDATING
  ↓
READY_FOR_ANALYSIS
  ↓
ANALYZING
  ↓
COMPLETED
```

Felstatus: `FAILED`, `PARTIALLY_COMPLETED`, `REJECTED`.

- Varje statusändring sparas (tabell `import_status_history`: `import_batch_id`, `from_status`, `to_status`, `reason`, `created_at`).
- Inga importfel får tyst kasseras. Varje fel får felkod och användarförståelig förklaring (§62).

---

## 8. INTELLIGENT COLUMN MAPPING

Systemet ska inte kräva ett exakt mallformat. Exempel: `Artikel`, `Artikelnummer`, `Art.nr`, `SKU`, `Product ID`, `Produktkod` ska kunna mappas till `sku`.

Mapping använder tre nivåer:

- **Level 1 — deterministic aliases.** En synonymtabell.
- **Level 2 — heuristic mapping.** Datatyp + innehåll + kolumnnamn.
- **Level 3 — AI semantic mapping.** LLM får: kolumnnamn, sample values (begränsade enligt §99), förväntade canonical fields.

LLM returnerar strukturerad output (valideras med Pydantic):

```json
{
  "source_column": "Art.nr",
  "canonical_field": "sku",
  "confidence": 0.97,
  "reason": "Column contains product identifiers."
}
```

*(`confidence` är ett värde utan monetär betydelse och får vara tal; alla belopp ska vara strängar, se §101.)*

### Trösklar

| Confidence | Beslut |
| --- | --- |
| `>= 0.95` | Auto-accept (icke-finansiella fält) |
| `0.80 – 0.949` | Fråga användaren |
| `< 0.80` | Omappad |

**Finansiellt kritiska fält** (belopp, kostnad, pris, antal, datum, valuta, rabatt, moms) kräver `>= 0.98` **och** godkänd stickprovsvalidering (t.ex. att värdena parsas som tal/datum i minst 95 % av raderna). Annars ska användaren bekräfta. Ett finansiellt kritiskt fält får aldrig auto-accepteras på svag grund.

---

## 9. CANONICAL DATA MODEL

All importerad data normaliseras. Originaldata sparas separat och oförändrad.

### 9.1 Organization

`id`, `name`, `created_at`

### 9.2 User

Auth-provider hanterar credentials. Applikationen sparar endast användarprofil (`id` = auth-providerns användar-id, `display_name`, `locale`, `created_at`).

### 9.3 OrganizationMembership

`organization_id`, `user_id`, `role`

Valfri begränsning per company (Agency-läge, §93): tabell `company_access` (`membership_id`, `company_id`, `role_override`).

### 9.4 Company (enda definitionen)

| Fält | Beskrivning |
| --- | --- |
| `id` | UUID |
| `organization_id` | FK |
| `name` | Företagsnamn |
| `country_code` | ISO 3166-1 alpha-2 |
| `base_currency` | ISO 4217 |
| `timezone` | IANA, t.ex. `Europe/Stockholm` |
| `business_type` | Enum enligt §4.2 |
| `fiscal_year_start` | Månad (1–12) |
| `created_at` | UTC |
| `deleted_at` | Soft delete, se §66 |

---

## 10. IMPORT ENTITIES

**ImportBatch:** `id`, `company_id`, `period_start`, `period_end`, `status`, `source_type`, `created_by`, `created_at`, `completed_at`, `superseded_by NULL` (se §14)

**ImportFile:** `id`, `import_batch_id`, `company_id`, `object_storage_key`, `original_filename`, `mime_type`, `sha256`, `size_bytes`, `document_type`, `extraction_status`, `created_at`

**ColumnMapping:** `id`, `import_file_id`, `company_id`, `source_column`, `canonical_field`, `confidence`, `mapping_method` (`ALIAS` / `HEURISTIC` / `AI` / `USER`), `confirmed_by_user`

---

## 11. CORE BUSINESS ENTITIES

**Product** – `id`, `company_id`, `sku`, `name`, `category`, `brand`, `unit`, `active`, `first_seen_date`. Unik: `(company_id, sku)`.

**Supplier** – `id`, `company_id`, `external_reference`, `name`, `country`, `currency`

**SalesTransaction** – `id`, `company_id`, `import_batch_id`, `external_order_id`, `transaction_date`, `customer_reference_hash NULL`, `currency`, `gross_revenue`, `discount_amount`, `net_revenue`, `tax_amount`
Customer identity ska inte importeras om den inte behövs (§99).

**SalesLine** – `id`, `company_id`, `sales_transaction_id`, `product_id`, `sku_snapshot`, `product_name_snapshot`, `quantity`, `unit_price`, `discount`, `revenue`, `cost_basis NULL`

**PurchaseDocument** – `id`, `company_id`, `supplier_id`, `import_batch_id`, `invoice_number`, `invoice_date`, `currency`, `total_cost`

**PurchaseLine** – `id`, `company_id`, `purchase_document_id`, `product_id NULL`, `description`, `quantity`, `unit`, `unit_cost`, `total_cost`

**InventorySnapshot** – `id`, `company_id`, `import_batch_id`, `snapshot_date`

**InventoryItem** – `id`, `company_id`, `inventory_snapshot_id`, `product_id`, `quantity_on_hand`, `unit_cost`, `inventory_value`

---

## 11A. GEMENSAMMA REGLER FÖR ALLA AFFÄRSTABELLER

1. **Tenant-fält:** Varje affärstabell har `company_id`, och `organization_id` ska kunna härledas (direkt kolumn eller via `companies`). Barntabeller (t.ex. `SalesLine`, `PurchaseLine`, `InventoryItem`, `FinancialStatementLine`, `RecipeIngredient`) **denormaliserar `company_id`** så att RLS och repository-scoping kan göras utan joins.
2. **Proveniens:** Varje normaliserad rad har `import_batch_id` och, där det går, `source_file_id` + `source_row_ref` (radnummer/sida/cell). Detta möjliggör §15 (lineage).
3. **Pengar:** `NUMERIC(20,4)` + `currency` (ISO 4217). Aldrig `float`.
4. **Tid:** `timestamptz` (UTC). Affärsdatum som `date` tolkat i företagets timezone.
5. **Primärnycklar:** UUID.
6. **Constraints:** FK, `NOT NULL` där affärsregeln kräver det, unika nycklar för idempotens (se §76), `CHECK` för enum-värden.
7. **Master data vs händelsedata:** `Product` och `Supplier` är masterdata (får uppdateras, med historik i `*_history` eller via snapshotfälten). Transaktions-, inköps- och lagerrader är append-only.

---

## 12. FINANCIAL DATA

**FinancialStatement** – `id`, `company_id`, `import_batch_id`, `statement_type`, `period_start`, `period_end`, `currency`

`statement_type`: `PROFIT_AND_LOSS`, `BALANCE_SHEET`, `CASH_FLOW`, `OTHER`

**FinancialStatementLine** – `id`, `company_id`, `financial_statement_id`, `account_code NULL`, `account_name`, `canonical_category`, `amount`, `is_subtotal` (boolean), `source_row_ref`

Canonical categories inkluderar bland annat:

`REVENUE`, `COGS`, `GROSS_PROFIT`, `PAYROLL`, `RENT`, `MARKETING`, `SHIPPING`, `PAYMENT_FEES`, `UTILITIES`, `DEPRECIATION`, `INTEREST`, `OTHER_OPEX`, `EBITDA`, `OPERATING_PROFIT`, `NET_PROFIT`, `CASH`, `RECEIVABLES`, `PAYABLES`, `INVENTORY`, `DEBT`, `EQUITY`

**Regel mot dubbelräkning:** Kategorier som är delsummor (`GROSS_PROFIT`, `EBITDA`, `OPERATING_PROFIT`, `NET_PROFIT`) lagras med `is_subtotal = true`. Analysmotorn **räknar alltid om** delsummor från atomära rader och jämför mot importerad delsumma. Avvikelse → datakvalitetsvarning (§59), aldrig tyst korrigering.

---

## 13. RESTAURANT ENTITIES

**Recipe** – `id`, `company_id`, `import_batch_id`, `name`, `menu_product_id`, `yield_quantity`

**RecipeIngredient** – `id`, `company_id`, `recipe_id`, `ingredient_product_id`, `quantity`, `unit`

**WasteEvent** – `id`, `company_id`, `import_batch_id`, `product_id`, `date`, `quantity`, `estimated_cost`, `reason`

Enhetsomvandling (kg/g, l/dl, st) ska ske i en central modul med explicita konverteringsregler. Okänd enhet → datakvalitetsvarning, ingen gissning.

---

## 13A. ANALYS-, AI- OCH DRIFTENTITETER (saknades i v1.0)

| Entitet | Viktiga fält |
| --- | --- |
| **Audit** | `id`, `company_id`, `period_start`, `period_end`, `status`, `calculation_version`, `rule_version`, `settings_snapshot` (JSON), `created_by`, `created_at`, `completed_at` |
| **MetricValue** | `id`, `company_id`, `audit_id`, `metric_name`, `metric_version`, `period_start`, `period_end`, `value` (NUMERIC som sträng i API), `unit`, `currency NULL`, `confidence`, `dimension_key NULL` (t.ex. SKU) |
| **MetricEvidence** | `id`, `company_id`, `metric_value_id` eller `finding_id`, `source_entity_type`, `source_entity_id`, `import_batch_id`, `calculation_version` |
| **Finding** | `id`, `company_id`, `audit_id`, `category`, `severity`, `title`, `description`, `impact_lower`, `impact_base`, `impact_upper`, `currency`, `confidence`, `calculation_method`, `recommended_action`, `priority_score`, `rule_version`, `status` (`OPEN` / `DISMISSED` / `RESOLVED`) |
| **Action** | `id`, `company_id`, `audit_id`, `finding_id`, `priority`, `text`, `estimated_impact_*` |
| **Forecast** | `id`, `company_id`, `metric_name`, `method`, `history_start`, `history_end`, `forecast_start`, `forecast_end`, `points` (JSON med decimal-strängar), `interval_lower/upper`, `model_version` |
| **Scenario** | `id`, `company_id`, `type`, `assumptions` (JSON), `before` (JSON), `after` (JSON), `delta` (JSON), `calculation_version`, `created_by`, `created_at` |
| **Report** | `id`, `company_id`, `audit_id`, `object_storage_key`, `report_version`, `status`, `created_at` |
| **ChatThread / ChatMessage** | Se §41 |
| **Job** | Se §52 |
| **FeatureFlag** | `key`, `enabled`, `scope` (global / organization / company), `updated_at`, `updated_by` |
| **AIUsageLog** | `id`, `company_id`, `provider`, `model`, `prompt_version`, `tokens_in`, `tokens_out`, `purpose`, `created_at` (aldrig promptinnehåll med kunddata) |
| **PilotFeedback** | Se §111 |

Alla är tenant-scopade enligt §11A.

---

## 14. HISTORIK

Data är **append-only** som huvudregel. Ny import av `January`, `February`, `March` ersätter inte gamla records.

Systemet ska kunna beräkna: month-over-month, quarter-over-quarter, year-over-year, rolling periods, same month previous year, custom period vs custom period.

**Korrigerade omuppladdningar (nytt i v1.1):** Om en kund laddar upp en rättad fil för en period som redan finns:

1. Identisk fil (samma `sha256` för samma company) → avvisas som dubblett med tydligt besked (§76).
2. Överlappande period med annat innehåll → systemet **varnar** och frågar användaren om den nya importen **ersätter** den gamla för analysändamål.
3. Vid ja markeras den gamla batchen `superseded_by = <ny batch>`. **Inget raderas.** Analysen använder endast batcher som inte är superseded; historiken (inkl. gamla rapporter) förblir förklarbar.

---

## 15. DATA LINEAGE

Varje metric och finding måste kunna svara på: *Varifrån kommer siffran?*

Därför sparas `MetricEvidence` med: `metric_id`, `source_entity_type`, `source_entity_id`, `import_batch_id`, `calculation_version`.

För AI-svar ska relevanta evidence IDs kunna återföras till användaren.

---

## 16. ANALYSIS ENGINE

Analysmotorn är separat från API och AI.

```text
Analysis Engine
├── Metric Engine
├── Rule Engine
├── Money Leak Engine
├── Forecast Engine
├── Scenario Engine
├── Anomaly Engine
├── Recommendation Engine
└── Vertical Analyzers
```

---

## 17. METRIC ENGINE

Alla metrics definieras centralt. Varje metric har: `metric_name`, `version`, `required_data`, `calculation`, `unit`, `confidence_rules`.

Exempel: `total_revenue`, `total_purchase_cost`, `gross_profit`, `gross_margin_pct`, `inventory_cost_value`, `sales_velocity`, `days_inventory`, `inventory_turnover`, `supplier_price_change`, `average_order_value`, `revenue_growth`, `purchase_growth`, `operating_margin`.

---

## 18. CENTRALA FORMLER

### Gross margin

```text
gross_margin = (revenue - cost) / revenue
```

Vid `revenue = 0`: `null` – inte division med noll.

### Sales velocity

```text
units_sold / number_of_days
```

### Days inventory

```text
current_stock / average_daily_sales
```

Om `average_daily_sales = 0`: värdet `NO_SALES` (inte numerisk infinity i JSON).

### Inventory cost value

```text
current_stock * latest_valid_unit_cost
```

Cost source ska sparas.

Alla formler implementeras med `Decimal`, med explicit avrundningsregel (`ROUND_HALF_EVEN`, 4 decimaler internt; visning avrundas först i presentationslagret).

---

## 19. DEAD STOCK

Stage 1 använder konfigurerbar horisont. Default: `90 days`.

```text
current_stock > 0
AND sales_quantity_during_horizon = 0
AND product_age >= dead_stock_horizon
```

`product_age` = antal dagar sedan `Product.first_seen_date` (första förekomst i någon importerad data: försäljning, inköp eller lager) fram till analysperiodens slut.

Nya produkter (`product_age < horizon`) får `INSUFFICIENT_HISTORY`, inte Dead Stock.

---

## 20. SLOW MOVING STOCK

Produkt har försäljning men mycket lång inventory coverage.

- `days_inventory > 180` → warning
- `days_inventory > 365` → high

Trösklar kan anpassas per vertical.

---

## 21. OVERSTOCK

Primär metod:

```text
target_stock =
    expected_daily_sales * (target_coverage_days + supplier_lead_time_days)
    + safety_stock

overstock_units = max(current_stock - target_stock, 0)
```

När lead time saknas används vertical default (§21A) och findingen markeras med lägre confidence.

Den gamla regeln `purchases > sales * 1.5` får endast användas som **sekundär indikator**.

## 21A. STANDARDVÄRDEN (OB-4, förslag att validera i pilot)

Alla värden är konfigurerbara per företag och sparas i `Audit.settings_snapshot` så att historiska resultat förblir förklarbara.

| Parameter | Retail/E-com | Restaurang/Café |
| --- | --- | --- |
| `dead_stock_horizon_days` | 90 | 30 |
| `target_coverage_days` | 60 | 7 |
| `default_supplier_lead_time_days` | 14 | 3 |
| `safety_days` | 7 | 2 |
| `slow_moving_warning_days` | 180 | 21 |
| `slow_moving_high_days` | 365 | 45 |

Användning av ett standardvärde istället för kunddata ska alltid synas i findingens förklaring och sänka confidence.

---

## 22. STOCKOUT RISK

```text
days_inventory < lead_time_days + safety_days
```

Finding ska ange: current stock, current velocity, estimated stockout date, confidence.

---

## 23. SUPPLIER PRICE INFLATION

Per artikel: `current_unit_cost` vs `historical_unit_cost`. Analysera 30, 90 och 365 dagar. Finding genereras vid material förändring (tröskel konfigurerbar, förslag: ≥ 5 % och ≥ minsta absolutbelopp).

---

## 24. MARGIN EROSION

Systemet jämför perioder. Exempel:

```text
previous_margin = 42%
current_margin  = 34%
change          = -8 percentage points
```

Försök därefter förklara deterministiskt: cost increase, discount increase, price decrease, product mix, waste, purchase cost, fees.

När orsaken inte kan fastställas ska systemet säga: *"Cause could not be determined from available data."* LLM får aldrig hitta på orsak.

---

## 25. MONEY LEAK ENGINE

Detta är en central produktfunktion.

Finding categories:

`DEAD_STOCK`, `OVERSTOCK`, `SLOW_MOVING`, `STOCKOUT_RISK`, `MARGIN_EROSION`, `SUPPLIER_INFLATION`, `EXCESS_DISCOUNT`, `WASTE`, `LOW_MARGIN_PRODUCT`, `POOR_MENU_ITEM`, `COST_ANOMALY`, `DUPLICATE_TRANSACTION`, `REVENUE_DECLINE`, `COGS_INCREASE`, `PAYROLL_PRESSURE`, `MARKETING_INEFFICIENCY`, `CASHFLOW_RISK`

Varje finding ska innehålla (**belopp som strängar**):

```json
{
  "severity": "HIGH",
  "title": "...",
  "description": "...",
  "economic_impact": {
    "lower": "12000.00",
    "base": "18000.00",
    "upper": "24000.00",
    "currency": "SEK"
  },
  "confidence": 0.88,
  "calculation_method": "overstock_coverage_v1",
  "recommended_action": "...",
  "evidence": []
}
```

Ett finding **utan evidence får inte skapas** (ADR-009).

---

## 26. EKONOMISK EFFEKT

Systemet får inte visa falsk precision. Använd: lower estimate, base estimate, upper estimate, confidence, calculation method.

Exempel: `Estimated opportunity: SEK 31,000–43,000` är bättre än `SEK 37,284.16` när underlaget innehåller osäkerhet.

---

## 27. PRIORITERING

Prioritet beräknas från:

```text
priority_score = economic_impact × confidence × urgency × actionability
```

Returnera: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `INFORMATIONAL`.

**Tillfällig mappning (OB-3):** Rangordna alla findings efter `priority_score`. `economic_impact` = `impact_base` normaliserat mot företagets årsomsättning (i %). Förslag: `CRITICAL` ≥ 3 % av årsomsättning och confidence ≥ 0.7; `HIGH` ≥ 1 %; `MEDIUM` ≥ 0.3 %; `LOW` under det; `INFORMATIONAL` för observationer utan beräknad effekt. `urgency` och `actionability` är heltalsskalor (1–3) som definieras per findingkategori i `ANALYTICS_RULES.md`.

---

## 28. "DO THIS FIRST"

Dashboarden ska alltid ha en prioriterad handlingslista. Max fem första actions. Exempel:

1. Stop buying SKU-104
2. Review supplier X
3. Increase price of product Y
4. Clear stock Z
5. Reorder product A

Varje action visar uppskattad ekonomisk effekt (intervall).

---

## 29. RESTAURANGANALYS

När receptdata finns: `ingredient_cost_per_portion`, `menu_price`, `gross_profit_per_portion`, `food_cost_pct`, `contribution_margin`.

Systemet ska identifiera:

- populär + hög marginal
- populär + låg marginal
- låg försäljning + hög marginal
- låg försäljning + låg marginal

Rekommendation kan vara: promote, reprice, redesign recipe, renegotiate ingredient cost, remove item.

---

## 30. ACTUAL VS THEORETICAL FOOD COST

Om recipes + purchases + sales finns: `theoretical_usage` vs `actual_usage`.

Skillnad kan bero på: waste, portioning, spoilage, inventory errors (och andra operativa orsaker som data inte kan särskilja).

Systemet får **inte anklaga någon**. Använd formuleringen: *"Unexplained variance detected."*

---

## 31. GENERIC BUSINESS ANALYZER

För företag utan vertikalspecifik modul, analysera: revenue, revenue growth, purchase cost, expense growth, gross margin, payroll, rent, marketing, operating result, inventory, cash if available, anomalies, concentration risks.

---

## 32. ANOMALY ENGINE

Stage 1 ska använda robusta statistiska metoder: median, MAD, interquartile range, percentage deviation, rolling baseline.

LLM får aldrig besluta själv att en transaktion är anomalous. LLM kan förklara en anomaly efter att motorn identifierat den.

---

## 33. FORECAST ENGINE — STAGE 1

Forecasting ska vara enkel och transparent. Tillåtna modeller Stage 1: moving average, weighted moving average, linear trend, seasonal naive när tillräcklig historik finns.

Returnera: `forecast`, `confidence interval`, `model`, `available_history`. Spara även forecast period, model version. Ingen avancerad ML krävs i Stage 1.

---

## 34. SCENARIO ENGINE

Stage 1 ska stödja:

- **Price change** – Vad händer om priset höjs 8 %?
- **Cost change** – Vad händer om råvaran blir 10 % dyrare?
- **Discount clearance** – Vad händer om vi rear ut detta lager med 20 %?
- **Sales change** – Vad händer vid −15 % försäljning?
- **Purchase reduction** – Vad händer om vi stoppar inköp av denna produkt?
- **Payroll simulation** (när löneuppgifter finns) – Vad händer om personalkostnaden minskar 20 000 kr/mån?

Scenarioresultat ska **aldrig skriva till riktig data**. Varje scenario sparar antaganden samt före/efter-värden.

---

## 35. AI ARCHITECTURE

```text
AI Orchestrator
├── Provider Adapter
├── Tool Registry
├── Prompt Policies
├── Context Builder
├── Response Validator
└── Audit Logger
```

---

## 36. AI PROVIDER INTERFACE

```python
class AIProvider(Protocol):
    async def generate_structured(...)
    async def chat(...)
    async def analyze_document(...)
```

Implementationer (alla molnbaserade API-leverantörer): `OpenAIProvider`, `AnthropicProvider`, `GeminiProvider`.

Stage 1 ska inte låsas till en viss leverantör. Ingen `MockProvider` i produktkod (§0B); testdubletter ligger i `tests/`. Lokalt körda modeller (t.ex. Ollama) är förbjudna (§0A).

Modellnamn, priser och gränser ska kontrolleras mot leverantörens aktuella dokumentation vid implementation (OB-1).

---

## 37. AI FÅR INTE HA DIREKT SQL-ACCESS

LLM får endast använda godkända tools:

`get_metric`, `compare_periods`, `get_financial_summary`, `get_product_performance`, `get_inventory_analysis`, `get_supplier_price_changes`, `get_findings`, `get_finding_evidence`, `explain_finding`, `run_forecast`, `run_scenario`, `get_data_quality`

Tools anropar applikationslagret (queries/services) – aldrig SQL direkt – och är alltid tenant-scopade till den inloggade användarens aktuella company.

---

## 38. AI TOOL CONTRACT

Alla tools ska returnera strukturerad JSON. **Tal som representerar pengar eller procent returneras som decimal-strängar.**

```json
{
  "metric": "gross_margin_pct",
  "value": "34.20",
  "unit": "percent",
  "currency": null,
  "period": {
    "start": "2026-09-01",
    "end": "2026-09-30"
  },
  "confidence": 0.9,
  "evidence_ids": ["..."]
}
```

---

## 39. AI SYSTEM POLICY

AI ska instrueras:

1. Hitta aldrig på siffror.
2. Använd tools för företagsdata.
3. Ange när data saknas.
4. Skilj mellan: *company fact*, *deterministic calculation*, *estimate*, *general advice*.
5. Ge aldrig säker slutsats om data-confidence är låg.
6. Hänvisa till vilken period analysen avser.
7. Vid konflikt gäller Calculation Engine, inte LLM.
8. Finansiella recommendationer ska beskrivas som beslutsstöd.

---

## 40. AI CHAT

Stage 1-chatten ska kunna svara på bland annat:

- Varför sjönk marginalen?
- Jämför september mot augusti.
- Vilka produkter binder mest kapital?
- Vad är min mest lönsamma produkt?
- Vilken leverantör har höjt priserna mest?
- Vad händer om jag höjer priset 10 %?
- Vad bör jag göra först?
- Förklara finding #17.

Den ska även kunna svara på generell företagsekonomisk kunskap. Generell kunskap ska märkas **General guidance** och inte blandas ihop med kunddata.

---

## 41. CHAT MEMORY

Konversationer sparas. Men företagets faktiska minne ska ligga i strukturerad databas, inte i chat history.

- **ChatThread:** `id`, `company_id`, `user_id`, `created_at`
- **ChatMessage:** `id`, `thread_id`, `company_id`, `role`, `content`, `tool_calls`, `created_at`

---

## 42. REPORT ENGINE

Stage 1 genererar professionell PDF med `ReportLab`. Rapporten genereras som bakgrundsjobb (§52) och sparas i object storage.

PDF ska innehålla:

1. Executive Summary
2. Company Health
3. Top Money Leaks
4. Do This First
5. Revenue & Margin
6. Inventory
7. Purchases
8. Supplier Findings
9. Financial Trends
10. Forecast
11. Scenarios
12. Data Quality
13. Methodology
14. Assumptions
15. Disclaimer

Rapporter använder endast verifierad beräknad data.

---

## 43. DASHBOARD

Stage 1-startsidan ska visa:

- **Company Health:** revenue, gross margin, inventory value, operating result if available
- **Money Leak Estimate:** lower–upper
- **Critical Findings:** max 5
- **Actions:** max 5
- **Trend:** current vs previous period
- **AI CFO:** chat entry

Dashboarden ska besvara: Hur mår företaget? Vad är fel? Vad kostar mest? Vad ska jag göra först? Vad har ändrats? Vad kan hända härnäst?

---

## 44. FRONTEND

Rekommenderad stack: React, TypeScript, Vite, Tailwind, TanStack Query, React Router. Ingen SSR behövs i Stage 1. Frontend är statisk och hostas i molnet (§67).

Affärslogik och ekonomiska beräkningar får aldrig ligga i frontend.

---

## 45. BACKEND

Python 3.12, FastAPI, Pydantic v2, SQLAlchemy 2, Alembic, Pandas, NumPy, SciPy where needed, PyMuPDF, openpyxl, xlrd where necessary, ReportLab, httpx.

Versioner ska kontrolleras mot aktuell dokumentation och låsas (lockfile) vid S0.1.

---

## 46. ARKITEKTURMÖNSTER

Använd **Modular Monolith + Hexagonal Boundaries**.

Skäl: snabbt att bygga, enkelt att deploya, enkelt att testa, inga distribuerade systemproblem, moduler kan senare brytas ut.

---

## 47. HUVUDMODULER

| Modul | Stage |
| --- | --- |
| `auth`, `organizations`, `companies`, `audit_log` | 0 |
| `imports`, `documents`, `normalization` | 1A–1B |
| `sales`, `purchases`, `inventory`, `finance`, `recipes` | 1B |
| `analytics`, `findings`, `recommendations` | 1C–1D |
| `forecasting`, `scenarios` | 1E |
| `chat` | 1F |
| `reports` | 1G |
| `alerts` | 2 (tom modul-skelett tillåts, inga falska funktioner) |
| `integrations` | 3 (endast interface i Stage 1) |
| `billing` | Datamodell förbereds i Stage 1; implementation senare |

---

## 48. FOLDER STRUCTURE

```text
project/
├── apps/
│   ├── api/
│   │   └── main.py
│   ├── worker/
│   │   └── main.py
│   └── web/
├── src/
│   └── business_auditor/
│       ├── core/
│       │   ├── config.py
│       │   ├── errors.py
│       │   ├── logging.py
│       │   └── security.py
│       ├── domain/
│       │   ├── organizations/
│       │   ├── companies/
│       │   ├── imports/
│       │   ├── sales/
│       │   ├── purchases/
│       │   ├── inventory/
│       │   ├── finance/
│       │   └── analytics/
│       ├── application/
│       │   ├── commands/
│       │   ├── queries/
│       │   ├── services/
│       │   └── dto/
│       ├── infrastructure/
│       │   ├── database/
│       │   ├── storage/
│       │   ├── ai/
│       │   ├── documents/
│       │   ├── auth/
│       │   └── integrations/
│       ├── analytics/
│       │   ├── metrics/
│       │   ├── rules/
│       │   ├── anomalies/
│       │   ├── forecasting/
│       │   ├── scenarios/
│       │   └── verticals/
│       │       ├── generic/
│       │       ├── retail/
│       │       ├── ecommerce/
│       │       └── restaurant/
│       ├── ai/
│       │   ├── orchestrator.py
│       │   ├── tools/
│       │   ├── prompts/
│       │   └── validators/
│       └── reports/
├── migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── golden/
│   └── e2e/
├── fixtures/            # endast för tester (golden datasets)
├── scripts/             # CI/CD- och driftskript (körs i molnet)
├── infra/               # deploy-konfiguration för molnplattformarna
├── .github/workflows/   # CI/CD
├── docs/
│   ├── adr/
│   └── architecture/
├── pyproject.toml
├── Dockerfile           # byggrecept för molnplattformen (§0A)
└── README.md
```

*Borttaget från v1.0: `docker-compose.yml`, `docker/`.*

---

## 49. DEPENDENCY RULES

Tillåtet:

```text
API → Application → Domain
```

Infrastructure implementerar ports från application/domain.

Förbjudet: `Domain → FastAPI`, `Domain → SQLAlchemy`, `Domain → OpenAI`, `Domain → R2`, `Domain → Supabase`.

Analytics ska kunna köras i unit tests utan webserver eller databas. En arkitekturtest i CI ska verifiera dessa importregler.

---

## 50. DATABASE

PostgreSQL. Obligatoriskt: UUID primary keys, timezone-aware timestamps, foreign keys, migrations, unique constraints, indexes, soft delete där juridiskt relevant, tenant fields.

Alla schemaändringar sker via Alembic-migrationer (aldrig manuella ändringar i staging/production).

---

## 51. MULTI-TENANCY

Varje business record ska kunna härledas till `organization_id` och `company_id`. Backend får aldrig lita på `company_id` från klienten utan authorization check.

Defense in depth:

1. API authorization (roll + medlemskap + ev. company-begränsning).
2. Repository tenant scoping (alla queries kräver ett `TenantContext`).
3. Databas row-level security där möjligt.

**RLS-mönster (nytt i v1.1):** Backend ansluter med en dedikerad applikationsroll som **inte** kringgår RLS (inte service-/superuser-roll). Varje request/jobb sätter transaktionslokalt `SET LOCAL app.current_company_id = ...` (och organisation), och RLS-policys jämför mot detta. Service-role-nycklar används aldrig i frontend och aldrig för vanliga kundförfrågningar. Mönstret ska verifieras mot aktuell Supabase-dokumentation vid S0.3 och testas med ett negativt test (användare A når aldrig företag B:s rader).

---

## 52. BACKGROUND PROCESSING

Långa jobb ska inte köras inne i HTTP-request. Stage 1 använder PostgreSQL job queue.

Jobbtabell: `id`, `job_type`, `company_id`, `payload`, `status`, `attempts`, `available_at`, `locked_at`, `last_error`, `created_at`, `completed_at`.

Worker hämtar jobb med `FOR UPDATE SKIP LOCKED`. Det undviker separat Redis i första versionen.

Driftnotering: Workern ska ansluta på ett sätt som stödjer transaktioner och lås korrekt (direkt eller session-läge, ej transaction-pooler om det ger problem). Verifiera mot aktuell Supabase-dokumentation vid S0.6. Jobb som låsts för länge (`locked_at` äldre än timeout) ska återlämnas till kön.

---

## 53. JOB TYPES

`CLASSIFY_DOCUMENT`, `EXTRACT_DOCUMENT`, `NORMALIZE_IMPORT`, `VALIDATE_IMPORT`, `RUN_AUDIT`, `GENERATE_REPORT`, `GENERATE_FORECAST`, `SEND_ALERT` (Stage 2 – ingen implementation i Stage 1), `DELETE_COMPANY_DATA`

---

## 54. RETRY

Externa provider errors:

| Försök | Väntetid |
| --- | --- |
| 1 | 30 sek |
| 2 | 2 min |
| 3 | 10 min |

Efter max attempts: `FAILED`. Ingen oändlig retry. Valideringsfel (t.ex. trasig fil) retry:as inte.

---

## 55. OBJECT STORAGE

Raw files och rapporter lagras inte i PostgreSQL.

```python
class ObjectStorage(Protocol):
    def put(...)
    def get(...)
    def delete(...)
    def signed_url(...)
```

Stage 1: molnbaserad, S3-kompatibel object storage (§67). Ingen lokal filsystemadapter i produktkod.

Nyckelstruktur: `organization/company/import/file`.

**Uppladdningsflöde (nytt i v1.1):** Klienten ber API:t om en **signerad uppladdnings-URL** (efter behörighets- och storlekskontroll), laddar upp **direkt till storage**, och anropar sedan API:t för att registrera filen. API-servern strömmar inte stora filer. Filen valideras därefter i `SCANNING` (§56).

---

## 56. FILE SECURITY

Kontroller: MIME validation, extension validation, size limit, filename sanitization, checksum (sha256), malware scanning hook, PDF page limit, image dimension limit.

Default limits (konfigurerbara): `50 MB/file`, `20 files/import`, `200 pages/import batch`.

Uppladdade filer körs eller tolkas aldrig som kod. Innehåll valideras efter uppladdning (inte enbart via klientens deklarerade typ).

---

## 57. GOOGLE SHEETS

Stage 1: Shared-link import. Tillåt endast `docs.google.com/spreadsheets` (strikt parsning av host, schema `https`, förväntad sökvägsform; ingen redirect-följning till annan host).

Systemet ska aldrig göra godtyckliga server-side URL requests. **Skydd mot SSRF är obligatoriskt** (allowlist, inga IP-adresser, inga andra portar, timeout och storleksgräns). Stage 3: Google OAuth connector.

---

## 58. DOCUMENT EXTRACTION

```text
PDF
 ↓
Text extraction
 ↓
Was usable text recovered?
 ├── yes → structured parser
 └── no  → vision/document AI
```

Images: vision extraction. LLM/OCR-resultatet ska valideras med Pydantic-schema. Extraktion körs som bakgrundsjobb, aldrig synkront i HTTP-request.

---

## 59. DATA QUALITY ENGINE

Varje import får `quality_score`. Kontroller: missing dates, invalid currency, negative quantities, duplicate rows, impossible totals, missing SKU, conflicting totals, incomplete periods, unmapped columns, subtotal-avvikelser (§12), okända enheter (§13).

Resultat: `GOOD`, `USABLE_WITH_WARNINGS`, `POOR`, `INSUFFICIENT`.

AI får inte uttrycka starka slutsatser på `INSUFFICIENT`.

---

## 60. API VERSIONING

Alla routes ligger under `/api/v1/`.

---

## 61. STAGE 1 API

Alla paths nedan är relativa `/api/v1`.

| Område | Endpoints |
| --- | --- |
| Health | `GET /health` |
| User | `GET /me` |
| Organizations | `POST /organizations`, `GET /organizations`, `GET /organizations/{id}` |
| Members | `GET /organizations/{id}/members`, `POST /organizations/{id}/members`, `PATCH /organizations/{id}/members/{user_id}` |
| Companies | `POST /companies`, `GET /companies`, `GET /companies/{id}`, `PATCH /companies/{id}`, `DELETE /companies/{id}` |
| Imports | `POST /companies/{id}/imports`, `GET /companies/{id}/imports`, `POST /imports/{id}/upload-url`, `POST /imports/{id}/files`, `POST /imports/{id}/google-sheet`, `GET /imports/{id}`, `GET /imports/{id}/mapping`, `POST /imports/{id}/mapping/confirm`, `POST /imports/{id}/process` |
| Audits | `POST /companies/{id}/audits`, `GET /audits/{id}`, `GET /companies/{id}/audits` |
| Dashboard | `GET /companies/{id}/dashboard` |
| Findings | `GET /audits/{id}/findings`, `GET /findings/{id}`, `GET /findings/{id}/evidence` |
| Actions | `GET /audits/{id}/actions` |
| Metrics | `GET /companies/{id}/metrics`, `GET /companies/{id}/metrics/compare` |
| Forecast | `POST /companies/{id}/forecasts` |
| Scenario | `POST /companies/{id}/scenarios`, `GET /scenarios/{id}` |
| Chat | `POST /companies/{id}/chat/threads`, `POST /chat/threads/{id}/messages`, `GET /chat/threads/{id}` |
| Reports | `POST /audits/{id}/reports`, `GET /reports/{id}`, `GET /reports/{id}/download-url` |
| Jobs | `GET /jobs/{id}` (status/progress för UI) |

Alla endpoints: typade request/response-scheman, pagination för listor, standardfel (§62), auktorisering.

---

## 62. ERROR RESPONSE

```json
{
  "error": {
    "code": "IMPORT_COLUMN_MISSING",
    "message": "Required sales date column could not be identified.",
    "details": {},
    "request_id": "..."
  }
}
```

Ingen stacktrace till kund. Användarvända fel ska förklara vad användaren kan göra härnäst.

---

## 63. OBSERVABILITY

Alla requests får `request_id`. Loggar struktureras som JSON.

Logga: request, processing time, job lifecycle, AI provider usage, extraction failures, data validation, audit completion.

Logga aldrig: passwords, auth tokens, hela finansiella filer, känsliga dokumentinnehåll.

---

## 64. AUDIT LOG

Spara: login, import, delete, report generation, organization changes, permission changes, AI external calls metadata, external action execution (senare).

---

## 65. SECURITY

Obligatoriskt: HTTPS, JWT validation, role authorization, tenant isolation, signed file URLs, secrets only in environment/secret store, rate limiting, upload validation, SQL parameterization, CSRF strategy where applicable, security headers, dependency scanning.

Inga bypass-mekanismer (§0B). Ingen produktionsdata i loggar.

---

## 66. DELETION

Kunden ska kunna begära *Delete company*. Deletion job ska radera: database records, raw files, reports, chat history, derived metrics, AI cached artifacts.

Audit log får endast behållas där lagligt nödvändigt och ska inte innehålla affärsdatan.

**Tillägg v1.1:**

- Objektlagring för kundfiler och rapporter ska ha **versionering avstängd** (eller en lifecycle-regel som permanent raderar tidigare versioner), annars kvarstår raderad data.
- Databasbackuper innehåller raderad data under backupens retentionstid. Detta ska framgå i integritetsinformationen till kund och i `SECURITY.md`.
- Efter raderingsjobb ska en verifiering köras som bekräftar att inga rader/objekt med företagets id finns kvar, och resultatet loggas (utan affärsdata).

---

## 67. DEPLOYMENT — STAGE 1 (ENBART MOLN)

Mål: billig, managed cloud-hosting. Ingen komponent körs på användarens egen dator (§0A).

```text
GitHub (källkod + CI/CD)
        │
        ├── Cloudflare Pages ── React frontend (statisk)
        │
        ├── Railway ─┬── FastAPI API
        │            └── Worker
        │
        ├── Supabase ─┬── PostgreSQL
        │             └── Authentication
        │
        ├── Cloudflare R2 ── dokument/rapporter
        │
        └── Extern AI-API (molnleverantör bakom AIProvider)
```

- Plattformsval är rekommendationer; de ligger bakom adapter-/infrastrukturlagret (§49) och kan bytas via ADR.
- Priser, gränser och funktioner hos plattformarna **ska kontrolleras i aktuell dokumentation** innan val låses.
- Region för EU-kunder: se OB-5.
- Deploy sker via CI/CD från GitHub; manuella produktionsändringar är förbjudna.

### 67A. Vad projektägaren behöver tillhandahålla (konton/åtkomst)

Bygg-AI ska inte gissa eller skapa dessa åt ägaren. Saknas något ska det rapporteras som **BLOCKER** (§114):

1. Molnhostat Git-repository (t.ex. GitHub) med åtkomst för bygg-AI.
2. Supabase-projekt för staging och production.
3. Railway-projekt (API + worker) för staging och production.
4. Cloudflare-konto (Pages + R2) och ev. domän.
5. API-nyckel hos vald AI-leverantör (OB-1).
6. Plats för hemligheter: plattformarnas secret stores samt CI-secrets.

---

## 68. ENVIRONMENTS

Minst två molnmiljöer:

| Miljö | Syfte |
| --- | --- |
| `staging` | Verifiering av varje etapp, E2E-tester, golden regression. Deployas automatiskt från `main`. |
| `production` | Riktiga kunder. Deployas efter release gate (§106). |

Valfritt: automatiska **PR-previews i molnet**. Ingen miljö körs lokalt. Miljön `development` från v1.0 är borttagen.

Separata för varje miljö: databaser, storage buckets, API keys, auth configuration.

Production data får aldrig kopieras till staging utan sanitization. Staging innehåller inga riktiga kunddata.

---

## 69. TEST ARCHITECTURE

**Unit** – testa formulas, rules, scenarios, forecasting, mappings, normalization.

**Integration** – testa PostgreSQL, storage, auth, import pipeline, reports. Körs i CI med molnbaserade service-containrar.

**Contract** – testa AI provider schema, document extraction schema, API response schema.

**Golden Dataset** – verklighetsliknande fasta dataset. Audit-resultatet måste förbli stabilt mellan releases. Används endast i tester (§0B).

**End-to-End** – browser, mot deployad staging: `signup → create company → upload → confirm mapping → run audit → view findings → chat → download PDF`.

**Arkitektur- och säkerhetstester** – importregler (§49), bypass-scan (§0B), tenant-isolering (negativa tester).

---

## 70. GOLDEN DATASET 1 — RETAIL

Minst: 100 SKUs, dead stock, overstock, fast seller, supplier increase, margin erosion, stockout risk, malformed row, duplicate row. Förväntade findings dokumenteras.

---

## 71. GOLDEN DATASET 2 — RESTAURANT

Minst: 30 menu items, 60 ingredients, recipe definitions, purchase invoices, menu sales, waste events, supplier increase, food cost problem. Förväntade findings dokumenteras.

---

## 72. AI TESTING

För varje AI-answer testas: contains no unsupported company number, uses tool calls, handles missing data, distinguishes estimate, refuses to fabricate. Automatiska evals ska sparas.

---

## 73. DEFINITION OF DONE — VARJE FEATURE

En feature är inte klar förrän:

1. implementation finns,
2. types finns,
3. validation finns,
4. error handling finns,
5. unit tests finns,
6. integration test finns om relevant,
7. authorization är testad,
8. logging finns,
9. documentation finns,
10. acceptance criteria passerar,
11. den är **deployad och verifierad i staging** (§0A),
12. ingen bypass/mock/demo-kod har introducerats (§0B).

---

## 74. BUILD ORDER — STAGE 0

Bygg-AI ska följa exakt denna ordning. Varje gate verifieras i **molnet** (CI + staging).

**S0.0 Cloud provisioning (nytt)**
Skapa repository-struktur för CI/CD, koppla Supabase/Railway/Cloudflare (se §67A), lägg hemligheter i secret stores.
Gate: `GET /api/v1/health` svarar över HTTPS i **staging** och **production**, deployat via CI/CD.

**S0.1 Repository**
Skapa: backend, frontend, tests, docs, CI.
Gate: `lint PASS`, `tests PASS`, `container build PASS` (i CI), `bypass-scan PASS`.

**S0.2 Configuration**
Implementera typed settings.
Gate: saknad obligatorisk secret → startup failure.

**S0.3 Database**
Skapa: organizations, memberships, companies, audit_log, jobs, feature_flags. Alembic-migration.
Gate: migration up/down test PASS (i CI mot molnbaserad PostgreSQL).

**S0.4 Authentication**
Implementera auth adapter (Supabase Auth bakom ett interface). Registrering, login, logout, password reset.
Gate: obehörig tenant-åtkomst ger 403/404; negativt cross-tenant-test PASS; ingen väg runt autentisering.

**S0.5 Storage**
Implementera `ObjectStorage` mot molnlagring, inkl. signerade upp-/nedladdnings-URL:er.
Gate: upload/read/delete PASS; versionering avstängd/rensning verifierad (§66).

**S0.6 Worker**
Implementera PostgreSQL job queue.
Gate: failed jobs retry korrekt; låsta jobb återlämnas efter timeout.

### STAGE 0 EXIT GATE

Får endast gå vidare om: `Authentication PASS`, `Tenant isolation PASS`, `Migrations PASS`, `Worker PASS`, `Storage PASS`, `CI PASS`, `Deployed in staging and production PASS`.

---

## 75. BUILD ORDER — STAGE 1A IMPORT

- S1.1 Import domain – ImportBatch + ImportFile + status history
- S1.2 CSV – parser (chunkad, §98)
- S1.3 XLS/XLSX – parser
- S1.4 PDF – text extraction
- S1.5 Images – vision/document extraction adapter
- S1.6 Google Sheets – shared URL import (SSRF-skydd, §57)
- S1.7 Document classifier
- S1.8 Column mapper – alias + heuristic + AI fallback
- S1.9 Mapping confirmation UI
- S1.10 Data quality report

### STAGE 1A EXIT GATE

Golden files: `CSV PASS`, `XLSX PASS`, `PDF PASS`, `image PASS`, `Google Sheet PASS`, `mapping PASS`. Deployad i staging.

---

## 76. BUILD ORDER — STAGE 1B NORMALIZATION

Bygg i ordning: 1. products, 2. suppliers, 3. sales, 4. purchases, 5. inventory, 6. financial statements, 7. recipes, 8. waste.

- Varje normalizer ska vara **idempotent**.
- Import av samma fil två gånger identifieras genom checksum (`sha256` + `company_id`) och avvisas som dubblett.
- Dubbletter inom en fil (samma transaktion/rad) flaggas i datakvalitetsrapporten och kasseras inte tyst.

---

## 77. BUILD ORDER — STAGE 1C ANALYTICS

Implementera i ordning: 1. revenue, 2. purchase totals, 3. inventory valuation, 4. gross margin, 5. sales velocity, 6. dead stock, 7. slow moving, 8. overstock, 9. stockout, 10. supplier price changes, 11. margin erosion, 12. financial trend, 13. anomaly engine, 14. Money Leak Engine, 15. prioritization.

### STAGE 1C EXIT GATE

Golden retail dataset ska ge förväntade findings. **Ingen LLM används för att passera denna gate.** Deployad i staging.

---

## 78. BUILD ORDER — STAGE 1D RESTAURANT

Implementera: 1. recipe cost, 2. food cost, 3. contribution margin, 4. item classification, 5. waste impact, 6. actual vs theoretical när data finns.

Gate: Golden restaurant dataset ger förväntade findings.

---

## 79. BUILD ORDER — STAGE 1E FORECAST/SCENARIO

Implementera: 1. baseline forecast, 2. confidence interval, 3. price scenario, 4. cost scenario, 5. discount clearance, 6. sales scenario, 7. purchase stop scenario, 8. payroll scenario.

---

## 80. BUILD ORDER — STAGE 1F AI CFO

Implementera: 1. provider interface, 2. default provider, 3. tool registry, 4. company context, 5. period comparison tool, 6. product tool, 7. supplier tool, 8. finding tool, 9. scenario tool, 10. forecast tool, 11. system policies, 12. structured response validator, 13. chat persistence, 14. UI.

### AI CFO GATE

Test prompt: *"Why did my margin fall?"* AI måste: använda tool, ange korrekt period, ange korrekta siffror, förklara kända orsaker, ange datagap, inte hitta på orsak.

---

## 81. BUILD ORDER — STAGE 1G REPORTING

Bygg: 1. report DTO, 2. PDF renderer, 3. executive summary, 4. findings pages, 5. graphs, 6. recommendations, 7. methodology, 8. data-quality appendix.

---

## 82. BUILD ORDER — STAGE 1H FRONTEND

Pages: `/login`, `/dashboard`, `/companies`, `/company/:id`, `/company/:id/imports`, `/company/:id/audits`, `/audit/:id`, `/company/:id/chat`, `/company/:id/scenarios`, `/settings`.

Frontend byggs inkrementellt redan under tidigare etapper så att varje etapp är användbar live (se §0C). 1H är slutpolering och fullständig täckning.

---

## 83. STAGE 1 FINAL ACCEPTANCE TEST

En helt ny användare ska kunna, **mot production-miljön i molnet**:

`create account → create company → upload real files → correct mappings → run audit → see findings → see financial impact → receive actions → ask AI a question → run scenario → download PDF`

utan developer intervention.

---

## 84. STAGE 1 PILOT SUCCESS CRITERIA

Teknisk framgång är inte affärsframgång. För 3–10 pilotföretag samla:

- Did the audit identify something they did not know?
- Was the finding correct?
- Was it financially meaningful?
- Would they act on it?
- Would they pay?
- How much?
- Would they upload another month?
- Would they recommend it?

---

## 85. PRODUKTMETRICS

Mät: signup → first upload, upload → successful audit, audit → AI chat, audit → PDF, audit → second import, finding → action intent.

Viktigast: **second import rate.** Om kunden kommer tillbaka med nästa månads data finns verkligt värde.

---

## 86. STAGE 2 — DEEP INTELLIGENCE

Först när Stage 1-validering är positiv. Bygg: advanced forecasting, seasonal models, advanced restaurant analysis, wholesale vertical, construction vertical, workshop vertical, service-business vertical, recurring alerts, email notifications, supplier intelligence, cash-flow forecast, budget, variance analysis, more document extraction, invoice line matching, duplicate invoices, improved OCR, advanced scenario builder.

---

## 87. STAGE 3 — CONNECTED AI CFO

Connectors: Shopify, WooCommerce, Fortnox, Visma, Xero, QuickBooks, Square, Lightspeed, Toast, Google Sheets OAuth.

```text
External system
      ↓
Connector Adapter
      ↓
Canonical Data Model
      ↓
Existing Analysis Engine
```

Analytics får aldrig specialkopplas direkt till Shopify/Fortnox/etc.

---

## 88. CONNECTOR INTERFACE

```python
class DataConnector(Protocol):
    async def authorize(...)
    async def sync_sales(...)
    async def sync_purchases(...)
    async def sync_inventory(...)
    async def sync_financials(...)
```

---

## 89. STAGE 4 — BENCHMARK

Benchmarkdata ska alltid innehålla: source, industry, country, sample, period, metric definition, updated_at.

Systemet får inte säga *"Similar restaurants normally have X"* utan verifierad benchmarkkälla.

---

## 90. STAGE 4 — ACTION ENGINE

Actions kan exempelvis bli: `CREATE_CAMPAIGN_DRAFT`, `CREATE_PRICE_LIST`, `CREATE_PURCHASE_ORDER_DRAFT`, `CREATE_SUPPLIER_EMAIL`, `CREATE_BUDGET`, `CREATE_MONTHLY_REPORT`, `UPDATE_SHOPIFY_PRICE`, `PAUSE_SHOPIFY_PRODUCT`.

---

## 91. HUMAN APPROVAL

Externa förändringar får aldrig göras direkt av AI utan approval.

```text
AI recommendation
      ↓
Action draft
      ↓
User preview
      ↓
Explicit approval
      ↓
Execution
      ↓
Verification
      ↓
Audit log
```

---

## 92. BILLING

Datamodellen ska förberedas från Stage 1.

- **Free Scan:** begränsad analys, begränsat antal findings
- **Single Audit:** full audit, PDF
- **Pro:** historik, AI CFO, forecasting, scenarios, alerts
- **Agency:** flera företag, customer portfolio, white-label senare

---

## 93. ACCOUNTANT / AGENCY MODEL

Organization: *Accounting Firm AB* → Companies: *Customer A*, *Customer B*, *Customer C*, …

Behörighet per company ska kunna begränsas (`company_access`, §9.3). Detta får inte kräva senare database redesign.

---

## 94. INTERNATIONALIZATION

Från första versionen lagras all money med `amount` + `currency_code` (ISO 4217).

Ingen intern kod får anta: SEK, decimal separator, Swedish date. UI Stage 1 kan börja med svenska/engelska.

---

## 95. CURRENCY

Ingen automatisk valutaräkning behövs i Stage 1 om bolaget arbetar i en valuta. När multi-currency-transaktioner finns: spara original currency.

Senare FX-engine: `original_amount`, `original_currency`, `base_amount`, `base_currency`, `fx_rate`, `fx_rate_date`, `fx_source`.

---

## 96. ARCHITECTURAL DECISION RECORDS

| ADR | Beslut | Motivering |
| --- | --- | --- |
| ADR-001 Modular Monolith | Modular monolith. | Snabb utveckling och låg driftkostnad. Microservices avvisade initialt. |
| ADR-002 Deterministic Finance | Python räknar ekonomiska siffror. LLM tolkar endast. | — |
| ADR-003 Canonical Data Model | Alla källsystem transformeras till samma interna modell. | — |
| ADR-004 Immutable History | Imports är versionerade och ersätter aldrig historik (`superseded_by`, §14). | — |
| ADR-005 Multi-Tenant From Day One | Organisation → companies. | — |
| ADR-006 AI Provider Abstraction | Ingen vendor lock-in. | — |
| ADR-007 File-First Product | Integrationer skjuts upp tills värdet verifierats. | — |
| ADR-008 PostgreSQL Job Queue | Ingen Redis/Celery i Stage 1. | — |
| ADR-009 Explainability | Alla findings måste kunna kopplas till evidence. | — |
| ADR-010 Human Approval for Actions | Ingen autonom extern write-operation. | — |
| **ADR-011 Cloud-Only** *(ny)* | Allt byggs, körs och testas i molnet. Ingen lokal körning. | Ägarbeslut, §0A. |
| **ADR-012 No Bypass / No Demo Modes** *(ny)* | Inga bypass-, mock- eller demolägen i produktkod. | Ägarbeslut, §0B. |
| **ADR-013 Short Live Stages** *(ny)* | Varje etapp deployas live och är användbar. | Ägarbeslut, §0C. |

---

## 97. PERFORMANCE TARGETS

- Vanligt API: `p95 < 500 ms` utan tunga analyser.
- Upload acknowledgement: `< 2 seconds` (tack vare direktuppladdning, §55).
- Audit får vara asynchronous. Stage 1 target: typical small business audit `< 5 minutes`, men UI ska visa progress.

---

## 98. IMPORT SCALE TARGET — STAGE 1

Design target: `500,000` sales lines/company/import, `100,000` purchase lines, `100,000` inventory rows.

Bearbeta i chunks. Ladda inte obegränsad CSV helt i RAM.

---

## 99. PRIVACY PRINCIPLE

Minimera data. Exempel: `Customer_ID` är inte nödvändigt för grundanalys. Importera inte personuppgifter om analysen inte kräver dem.

**Tillägg v1.1 – externa AI-leverantörer:**

- Till AI-leverantör skickas endast det som behövs: kolumnnamn, **ett litet antal** sample values (t.ex. högst 10 per kolumn), och aggregerade/strukturerade tool-resultat – aldrig hela filer eller rådata i onödan.
- Kolumner som sannolikt innehåller personuppgifter (namn, e-post, telefon, adress, personnummer) maskas eller exkluderas innan de skickas.
- Leverantörsval ska kontrollera att kunddata inte används för modellträning samt aktuella dataskyddsvillkor/DPA (OB-1, OB-5).
- Kunddata får inte läggas i loggar eller prompt-historik i klartext (§63).

---

## 100. COMMON PITFALLS — FÖRBJUDET

Bygg-AI får inte:

- lägga affärslogik i FastAPI routes
- lägga SQL direkt i AI tools
- låta LLM räkna financial metrics
- skriva över gamla imports
- använda float för pengar
- hårdkoda SEK i business logic
- bygga Shopify före pilot
- bygga microservices
- göra sync OCR/AI inne i HTTP request
- generera findings utan evidence
- presentera estimat som exakta fakta
- låta AI exekvera externa actions
- importera kundpersonuppgifter utan behov
- bygga 20 vertikaler före pilot
- skapa integrationer innan Stage 1 validerats
- **köra, kräva eller instruera lokal körning (§0A)**
- **införa bypass-, demo-, mock- eller testlägen i produktkod (§0B)**
- **leverera en etapp utan live-deploy (§0C)**
- **markera en gate som godkänd utan att ha kört den**

---

## 101. MONEY DATATYPE

Använd `Decimal`, aldrig `float`, för pengar. Databas: `NUMERIC`.

**API, tool-kontrakt, JSON-lagring och LLM-indata/utdata: pengar och procent representeras som decimal-strängar** (t.ex. `"18000.00"`), aldrig som JSON-tal, så att inga flyttalsfel kan uppstå.

---

## 102. TIME

All backend-tid är UTC. Company timezone används vid periods, reports, UI, daily aggregates.

---

## 103. VERSIONING

Metrics: `calculation_version`. Rules: `rule_version`. Prompts: `prompt_version`. Reports: `report_version`. När logik ändras måste historiska resultat kunna förstås.

---

## 104. FEATURE FLAGS

Använd feature flags (tabell `feature_flags`, §13A) för: `restaurant_v2`, `advanced_forecast`, `external_actions`, `benchmark`, `new_ai_provider`.

Ingen redeploy ska behövas för att stänga av en riskabel beta-feature. Ofärdiga funktioner är avstängda **och dolda** – aldrig falska (§0B).

---

## 105. CI PIPELINE

Körs i molnet vid varje PR och push: format, lint, type check, unit tests, integration tests, security scan, **bypass-scan (§0B)**, **arkitekturtest (§49)**, migration validation, frontend tests, build backend, build frontend.

Deployment blockeras vid fail. Deploy till staging sker automatiskt vid grön `main`; efter deploy körs smoke-test + E2E mot staging.

---

## 106. RELEASE GATE

Production release kräver: all CI green, no critical security issue, migration tested, rollback plan, golden dataset regression green, E2E mot staging green.

---

## 107. BACKUPS

Pilot: managed database backups där planen tillåter.

När betalande kunder finns: daily backup, **tested restore**, retention policy.

~~Object storage versioning~~ – **gäller inte kundfiler** (§66). Skydd mot oavsiktlig radering för kundfiler löses istället med mjuk radering (`deleted_at`) och en kort fördröjd permanent radering.

Backup som aldrig restore-testats räknas inte som fungerande backup.

---

## 108. DOCUMENTATION

Repository måste innehålla: `README.md`, `ARCHITECTURE.md`, `DATA_MODEL.md`, `IMPORT_FORMATS.md`, `ANALYTICS_RULES.md`, `AI_POLICY.md`, `SECURITY.md`, `DEPLOYMENT.md`, `RUNBOOK.md`. ADR i `/docs/adr/`.

---

## 109. README — MOLNBASERAD UPPSTART (ersätter lokal uppstart i v1.0)

Ett nytt teammedlem eller en bygg-AI ska utan dold information kunna:

1. få åtkomst till repository och molnplattformarna (§67A),
2. se var hemligheterna finns (secret stores – värdena står aldrig i repot),
3. förstå hur CI/CD deployar till staging och production,
4. köra migrationer via CI/CD-steget (inte manuellt),
5. se loggar och jobbstatus i molnplattformarna,
6. köra hela testsviten via CI,
7. verifiera en deploy med smoke-test och E2E mot staging.

README får **inte** innehålla instruktioner för att starta databas, API, worker eller frontend lokalt.

---

## 110. STAGE 1 SAMPLE OUTPUT

*Illustration av format. Pengar och procent är decimal-strängar (§101). Detta är inte seed-data och får aldrig visas för användare (§0B).*

```json
{
  "company_health": {
    "revenue": "1842000.00",
    "gross_margin_pct": "34.70",
    "inventory_value": "618000.00",
    "currency": "SEK"
  },
  "money_leak_summary": {
    "lower": "126000.00",
    "upper": "184000.00",
    "currency": "SEK"
  },
  "top_findings": [
    {
      "type": "OVERSTOCK",
      "severity": "HIGH",
      "title": "Excess inventory in SKU A17",
      "estimated_impact": {
        "lower": "38000.00",
        "upper": "47000.00",
        "currency": "SEK"
      }
    }
  ],
  "actions": [
    {
      "priority": 1,
      "action": "Pause purchases of SKU A17"
    }
  ]
}
```

---

## 111. PILOT DATA COLLECTION

För varje pilotkund ska internt registreras (tabell `PilotFeedback`, ingen kunddata utöver bedömningarna): `industry`, `number_of_imports`, `files_uploaded`, `processing_success`, `findings_generated`, `findings_confirmed_correct`, `findings_useful`, `actions_intended`, `would_pay`, `price_expectation`, `second_import`.

---

## 112. GO / NO-GO EFTER PILOT

Fortsätt större utveckling om exempelvis:

- majoriteten hittar minst ett verkligt värdefullt fynd,
- majoriteten bedömer analysen som korrekt,
- flera vill använda systemet igen,
- minst några uttrycker faktisk betalningsvilja.

Stoppa eller repositionera om kunder främst säger: *"Jag ser redan detta i mitt nuvarande system."* Det är viktigare än mängden kod.

---

## 113. SLUTLIG PRODUKTDEFINITION

Produkten är inte en dashboard. Produkten är inte en chatbot. Produkten är inte ett Shopify inventory tool.

Produkten är: **Ett persistent AI-CFO-system som omvandlar företagets befintliga data till verifierbara ekonomiska fynd, prioriterade åtgärder, prognoser och beslutsstöd.**

Dess centrala loop:

```text
IMPORT DATA → UNDERSTAND DATA → NORMALIZE → CALCULATE → DETECT → QUANTIFY
→ PRIORITIZE → EXPLAIN → SIMULATE → ACT → MEASURE RESULT → LEARN FROM NEXT PERIOD
```

---

## 114. MASTER INSTRUCTION TO BUILD AI

Följ blueprinten i ordning. Hoppa inte direkt till synliga features. Gör inga arkitekturändringar för att "förenkla" utan dokumenterat ADR.

**Arbetsregler (bindande):**

1. Arbeta **enbart i molnet** (§0A). Föreslå aldrig lokal körning.
2. Bygg **riktig produktionskod** utan bypass/demo/mock (§0B).
3. Deploya varje etapp live (§0C).
4. **Kör tills gaten är grön.** Stanna inte för bekräftelser som blueprinten redan besvarar.

**Efter varje build task:** 1. implementera, 2. testa (i CI), 3. verifiera acceptance criteria, 4. rätta fel, 5. commit, 6. deploya till staging, 7. verifiera, 8. gå vidare.

**Vid osäkerhet:** 1. kontrollera blueprinten, 2. kontrollera domain contracts, 3. kontrollera ADR, 4. välj inte ny arkitektur om blueprinten redan anger lösning.

**Rapportera aldrig något som klart som inte är verifierat.** Om kod skrivits men inte testats: *"Implemented; verification still required."*

När något i blueprinten faktiskt är omöjligt eller motsägelsefullt: **STOPPA DEN DELEN.** Dokumentera:

```text
BLOCKER
Expected architecture:
Observed conflict:
Possible solutions:
Recommended solution:
```

Fortsätt inte genom att tyst uppfinna ny arkitektur.

---

## 115. MASTER BUILD SEQUENCE

```text
STAGE 0   Foundation
   ↓
STAGE 1A  Import
   ↓
STAGE 1B  Normalization
   ↓
STAGE 1C  Core Analytics
   ↓
STAGE 1D  Restaurant Analytics
   ↓
STAGE 1E  Forecast + Scenario
   ↓
STAGE 1F  AI CFO
   ↓
STAGE 1G  PDF Reporting
   ↓
STAGE 1H  Web Portal
   ↓
STAGE 1 PILOT (3–10 companies)
   ↓
GO / NO-GO
   ↓
STAGE 2   Deep Intelligence
   ↓
STAGE 3   Connected AI CFO
   ↓
STAGE 4   Benchmark + Actions
```

### 115A. Första marknadsdugliga leverans och arbetsschema

- **Första live-leverans (tidigast möjliga):** efter Stage 0 + 1A + 1B + 1C: användaren loggar in, skapar företag, laddar upp CSV/XLSX, bekräftar mappning, kör audit och ser verifierade Money Leak-fynd med Do This First. Detta är riktig produkt, inte demo.
- **Nästa dokument (ska skapas separat):** `BUILD_PLAN.md` – ett exakt arbetsschema task för task: för varje task *mål, filer att skapa, beroenden, implementationssteg, tester, gate-kommando i CI, deploy-steg, definition of done*. Bygg-AI ska därefter kunna köra planen från början till slut utan ytterligare instruktioner.

---

## 116. HIGHEST PRIORITY

Om utvecklingstid måste kapas får följande **inte** tas bort: 1. korrekt normalisering, 2. historik, 3. deterministic analytics, 4. Money Leak Engine, 5. evidence, 6. recommendations, 7. AI tools, 8. AI chat, 9. PDF, 10. period comparisons.

Kapa istället: extra UI-polish, onödiga integrationer, advanced animations, många vertikaler, avancerad ML, white-label, automatiska externa actions.

Säkerhet, tenant-isolering och molnkravet får aldrig kapas.

---

## 117. PRINCIP FÖR HELA PROJEKTET

**Avancerad under huven. Enkel framför kunden.**

Kunden ska uppleva:

1. Ladda upp företagets information.
2. Systemet förstår den.
3. Systemet hittar problemen.
4. Systemet räknar vad problemen kostar.
5. Systemet berättar vad som bör göras först.
6. Kunden kan fråga varför.
7. Kunden kan simulera alternativa beslut.
8. Nästa månad vet systemet vad som har förändrats.

Detta är blueprintens styrande produktprincip.
