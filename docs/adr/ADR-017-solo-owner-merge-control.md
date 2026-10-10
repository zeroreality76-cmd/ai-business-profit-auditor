# ADR-017: Mergekontroll för en ensam ägare (bypass endast för pull requests)

**Status:** Godkänd av ägaren 2026-10-10 (H7). Se `docs/DECISIONS.md`.
**Datum:** 2026-10-10
**Ändrar:** `BUILD_PLAN.md` ägarpunkt 1 och 8, S0.0a, H8 och avsnitt 4A. Ändrar inte blueprinten.

## Kontext
`BUILD_PLAN.md` v3.2 (K11, V2-03) förutsatte att agenten fick en egen GitHub-identitet, att Required approvals sattes till 1 och att ägaren sedan godkände agentens pull requests.

Det går inte med nuvarande arbetssätt. Byggagenten körs via Claude Code i molnet, och pull requests öppnas under ägarens GitHub-konto (`zeroreality76-cmd`), vilket är verifierat på pull request #10. GitHub tillåter inte att den som öppnat en pull request godkänner den. Det gäller även repoägare, om ingen bypass är konfigurerad.

Claude Codes dokumentation anger att git-uppgifter ligger utanför sessionens virtuella maskin, att autentiseringen går via en proxy med begränsad nyckel och att git push är begränsat till den aktuella arbetsgrenen. Agenten kan alltså inte pusha direkt till `main`. Dokumentationen säger däremot inget om huruvida agenten kan merga pull requests.

## Beslut
1. Rulesetet `protect-main` behålls med: kräv pull request, blockera force-pushar, begränsa radering, och (när CI finns) obligatoriska statuskontroller.
2. **Require review from Code Owners** slås på. `.github/CODEOWNERS` pekar på ägaren för styrdokument, workflows, migrationer, infrastruktur och säkerhetskod.
3. **Repository admin** läggs i bypass-listan med läget **For pull requests only**. Ägaren kan då inte pusha direkt, men kan efter eget beslut merga en pull request som annars hade fastnat på granskningskravet. GitHub loggar bypass i pull requesten och i auditloggen.
4. Required approvals står på **0** tills provet i villkor C5 är gjort.
5. Ägaren mergar alla pull requests själv. Ägaren använder bypass-merge bara när (a) pull requesten rör skyddade filer och är läst, och (b) alla obligatoriska kontroller är gröna när de finns.
6. `AGENTS.md` regel 15–16: agenten mergar aldrig, ändrar aldrig regler eller inställningar och redovisar skyddade filer i varje pull request.

## Konsekvenser
- Ägaren får ett medvetet, loggat moment för ändringar i skyddade filer, i stället för ett omöjligt godkännande.
- Kontrollen är **procedurmässig, inte teknisk**. Eftersom agenten agerar med ägarens behörighet kan det inte uteslutas att en session tekniskt skulle kunna använda bypass. Det är en känd restrisk som accepteras i staging utan kunddata.
- Security Reviewer och Verifier körs som separata sessioner men under samma konto. Deras oberoende är därför också procedurmässigt.

## Verifiering (villkor C5)
Dokumentationen säger att Code Owner-kravet gäller när obligatoriska granskningar är påslagna, men säger inte hur det fungerar med noll godkännanden. Provet görs på den pull request som lägger in den här ADR:en, eftersom den rör skyddade filer (`docs/adr/`, `docs/DECISIONS.md`, `AGENTS.md`, `BUILD_PLAN.md`).

Ordning:
1. Pull requesten är öppen och inte mergad.
2. Ägaren lägger till Repository admin i bypass-listan med *For pull requests only*.
3. Ägaren slår på *Require review from Code Owners*. Required approvals står kvar på 0.
4. Ägaren laddar om pull requesten.
   - **Visas ett krav på Code Owner-granskning** och ett val att gå förbi det: provet lyckades. Ägaren mergar med bypass och noterar utfallet i `docs/DECISIONS.md` (C5).
   - **Visas en vanlig grön merge-knapp:** kravet gäller inte vid 0 godkännanden. Ägaren sätter Required approvals till 1, laddar om, mergar med bypass och noterar utfallet. Då blir bypass-merge ett krav för varje pull request.

## Omprövning (villkor C6)
Före första kunddata, och senast i Production readiness-gaten (`BUILD_PLAN.md` avsnitt 4A), beslutas något av:
a) en separat identitet för agenten, så att ägaren kan godkänna,
b) en teknisk spärr som hindrar agenten från att merga,
c) ägarens uttryckliga godkännande av att kontrollen förblir procedurmässig.

## Alternativ
- **A. Ett andra GitHub-konto för agenten.** Inte verifierat att Claude Code på webben kan kopplas till ett annat konto.
- **B. Claude Code via GitHub Actions med bot-identitet.** Annan arbetsmodell, och appens rättigheter styrs av installationen. Inte verifierat för detta projekt.
- **Required approvals = 1 utan bypass.** Avvisat: ägaren kan inte godkänna egna pull requests och skulle låsa sig själv.
