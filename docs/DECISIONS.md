# DECISIONS – ägarens beslut

Det här dokumentet är ägarens bekräftelse av beslut som `BUILD_PLAN.md` kräver före S0.0 (avsnitt 0, H5 och H7). Det ändrar inte arkitekturen. Avvikelser från blueprinten regleras i ADR:er under `docs/adr/`.

| Egenskap | Värde |
| --- | --- |
| **Beslutsfattare** | Projektägaren |
| **Datum** | 2026-10-09 |
| **Underlag** | `BUILD_PLAN.md` v3, `docs/SECURITY_REVIEW_PLAN.md`, `docs/SECURITY_REVIEW_PLAN_V2.md` |

## Sammanfattning

| Beslut | Utfall | Plankoppling |
| --- | --- | --- |
| ADR-015 Production skapas före första kunddata | **Godkänd** | H7, avsnitt 4A |
| ADR-016 E2E-testkonton via admin-API i staging | **Godkänd** | H7, S0.3, avsnitt 9A |
| OB-5 Region | **EU** | H5 |
| OB-6 Malware-skanning | **Standardvärdet i BUILD_PLAN** | H5 |
| OB-1 AI-leverantör | **Anthropic som enda aktiva leverantör i Stage 1**, med villkor nedan | H5 |

OB-2, OB-3, OB-4 och OB-7–OB-9 är inte föremål för något nytt beslut. Standardvärdena i `BUILD_PLAN.md` avsnitt 0 gäller tills ägaren svarar.

---

## ADR-015 – Production skapas före första kunddata – GODKÄND

- **Dokument:** `docs/adr/ADR-015-production-deferred.md`
- **Ändrar:** Blueprint §74 (S0.0-gate och Stage 0 exit gate), §67A, §68.
- **Beslut:** Stage 0 och Stage 1 byggs och verifieras i `staging`, som inte innehåller riktiga kunddata. `production` skapas före första kunddata och före H4.
- **Villkor:** **Production readiness-gate** (`BUILD_PLAN.md` avsnitt 4A) ska vara grön innan någon kunddata laddas upp. Den får inte hoppas över.

## ADR-016 – E2E-testkonton via admin-API i staging – GODKÄND

- **Dokument:** `docs/adr/ADR-016-e2e-test-accounts.md`
- **Ändrar:** Blueprint §0B (testkonton skapas genom den vanliga registreringen).
- **Beslut:** Testhjälparen finns bara i `tests/e2e/`. Den importeras aldrig från `src/`, vägrar köra mot production, använder en hemlighet begränsad till staging som inte exponeras för PR-jobb från forks, och ligger som undantag i bypass-scanens fil under CODEOWNERS.
- **Villkor:** Det är testinfrastruktur, inte en produktväg.

## OB-5 – Region: EU

- **Beslut:** EU-region för Supabase, Railway och Cloudflare R2.
- **Anmärkning:** Planen säger "där leverantören erbjuder det". Saknar en plattform EU-region ska det rapporteras som BLOCKER, inte lösas tyst. Region går ofta inte att byta i efterhand, så den ska väljas rätt när projekten skapas (S0.0).

## OB-6 – Malware-skanning: standardvärdet i BUILD_PLAN

- **Beslut:** Standardvärdet i `BUILD_PLAN.md` avsnitt 0 gäller: ingen hook utan leverantör. Endast dataformat som inte kan köras (CSV, XLSX utan makron, PDF, bilder) plus hård validering, tills en molnskanner valts.
- **Anmärkning:** Den hårda valideringen byggs i task S1A.0 (SCANNING och fientliga fixtures). Beslutet innebär inte att någon skanner finns.

## OB-1 – AI-leverantör: Anthropic som enda aktiva leverantör i Stage 1

- **Beslut:** Anthropic är den enda aktiva AI-leverantören i Stage 1.
- **Arkitektur:**
  - `AIProvider`-gränssnittet byggs **leverantörsneutralt** (blueprint §36, ADR-006). Ingen kod utanför `infrastructure/ai/` får bero på en viss leverantörs SDK, modellnamn eller svarsformat.
  - Ingen annan leverantör är aktiv eller valbar i staging eller production i Stage 1.
- **Andra leverantör senare:** En andra leverantör läggs till bakom feature flag `new_ai_provider` (blueprint §104) först när **avtal, tester och integritetsinformation** är klara för den.
- **Villkor före första kunddata:** Leverantörens villkor och personuppgiftsavtal (DPA) ska kontrolleras mot leverantörens **aktuella dokumentation** innan någon kunddata skickas. Kontrollen ska gälla de urvalskrav som `BUILD_PLAN.md` ställer på OB-1:
  - DPA finns,
  - ingen träning på kunddata,
  - kortast möjliga retention,
  - EU-behandling där det erbjuds.

  **Status: ej utförd.** Resultatet ska dokumenteras här (datum, vilka dokument som lästs, utfall) och i underbiträdeslistan i `SECURITY.md`.
- **Tills kontrollen är gjord:** Endast syntetiska testdata och golden datasets får skickas till AI-leverantören (S1A.5, S1A.8, 1F i staging). Inga uppgifter om riktiga företag eller personer.
- **Modellnamn och priser:** kontrolleras mot aktuell dokumentation vid S1A.5.

---

## Öppna villkor

| # | Villkor | Ägare | Senast |
| --- | --- | --- | --- |
| C1 | Kontrollera Anthropics villkor och DPA mot aktuell dokumentation, och dokumentera resultatet här | Ägare | Före första kunddata |
| C2 | Production readiness-gate grön (`BUILD_PLAN.md` avsnitt 4A) | Ägare, Verifier, Security Reviewer | Före första kunddata och före H4 |
| C3 | Integritetsinformation och underbiträdeslista klara innan en andra AI-leverantör aktiveras | Ägare | Innan `new_ai_provider` slås på |
| C4 | Bekräfta att EU-region finns hos Supabase, Railway och R2 när projekten skapas | Ägare | S0.0 |
