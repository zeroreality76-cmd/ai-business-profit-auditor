# ADR-016: E2E-testkonton skapas via auth-leverantörens admin-API i staging

**Status:** Godkänd av ägaren 2026-10-09 (H7). Se `docs/DECISIONS.md`.
**Datum:** 2026-10-09
**Ändrar:** Blueprint §0B (testkonton skapas genom den vanliga registreringen)

## Kontext
E2E-tester mot staging kräver testkonton. Ordinarie registrering kräver e-postverifiering, och rate limiting hindrar automatiska registreringar. Utan beslut finns risk att bygg-AI inför en dold testväg eller använder service role bredare än nödvändigt.

## Beslut
En testhjälpare som **bara** finns i `tests/e2e/` skapar testkonton via auth-leverantörens admin-API i staging. Hjälparen:
- importeras aldrig från `src/` (arkitekturtest),
- vägrar köra om miljön är production (eget test),
- använder en hemlighet begränsad till staging som bara exponeras för E2E-jobbet och aldrig för PR-jobb från forks,
- är en del av bypass-scanens undantagsfil under CODEOWNERS, med motivering.

Det är testinfrastruktur, inte en produktväg.

## Alternativ
a) Ordinarie registrering med en brevlåda testet styr (långsammare, skör). b) Detta beslut. c) Lämna oreglerat (avvisat).
