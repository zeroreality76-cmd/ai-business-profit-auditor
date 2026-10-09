# ADR-015: Production skapas före första kunddata, inte i Stage 0

**Status:** Godkänd av ägaren 2026-10-09 (H7). Se `docs/DECISIONS.md`.
**Datum:** 2026-10-09
**Ändrar:** Blueprint §74 (S0.0-gate och Stage 0 exit gate), §67A, §68

## Kontext
Blueprinten kräver att `staging` och `production` finns och är deployade i Stage 0. Ägaren har ett gratis Supabase-konto där den andra gratisplatsen redan används av ett annat projekt. Ett production-projekt med riktiga kunder bör ligga på betald plan (Pro) eftersom gratisprojekt pausas och saknar dagliga backuper.

## Beslut
Bygg och verifiera Stage 0 och Stage 1 i `staging`, som inte innehåller riktiga kunddata (§68). `production` skapas före första kunddata och före H4. Då körs en **Production readiness-gate** (BUILD_PLAN avsnitt 4A) som upprepar säkerhetstesterna i Stage 0 exit gate mot production.

## Konsekvenser
- Lägre kostnad under bygget.
- Production-specifik konfiguration (hemligheter, roller, Data API-test, backup) testas första gången sent. Readiness-gaten är kompensationen och får aldrig hoppas över.
- Ingen kunddata laddas upp i staging eller production förrän gaten är grön.

## Alternativ
a) Skapa production nu (kräver betalplan direkt). b) Detta beslut. c) Lämna oreglerat (avvisat).
