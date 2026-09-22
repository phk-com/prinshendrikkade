<!--
Titel = Conventional Commit, bijvoorbeeld: feat(forum): add quote button
Zie AGENTS.md §3. Zet een punt dat niet van toepassing is op "n.v.t.", in plaats van het weg te laten.
-->

## Waarom
Closes #<!-- issuenummer; gebruik Refs # als het issue open moet blijven -->

## Wat
<!-- Wat er verandert, kort. Screenshot of video bij UI-wijzigingen. -->

## Hoe getest
<!-- Preview-URL, welke flows, op telefoon (390px) en desktop, donker en licht -->

## Agent
<!-- Alleen als een agent meewerkte: label `agent-assisted`, plus de oorspronkelijke opdracht in 1–3 regels. Anders: n.v.t. -->

## Definition of Done (AGENTS.md §5)

**Functioneel**
- [ ] Acceptatiecriteria gehaald
- [ ] Getest in de preview op 390px en desktop, donker en licht

**Kwaliteit**
- [ ] CI groen (types, lint, tests, contracten, architectuurregels, Playwright, gitleaks)
- [ ] Tests volgens het testbeleid; bij een bugfix een test die de bug eerst liet zien
- [ ] Geen `any`, `@ts-ignore` of `eslint-disable` zonder uitleg

**Veiligheid en privacy**
- [ ] Geen geheimen of persoonsgegevens (code, fixtures, screenshots)
- [ ] Nieuwe routes lopen via `guard()`; modules gebruiken alleen `MemberView`; apps vragen minimale rechten
- [ ] Geen nieuwe dependency, of hij is hieronder uitgelegd

**Toegankelijkheid**
- [ ] Contrast AA, aanraakdoelen ≥ 44px, focus zichtbaar, werkt met het toetsenbord
- [ ] UI-teksten in het Nederlands, alleen tokens en componenten uit `@phk/ui`

**Data en uitrollen**
- [ ] Migraties achterwaarts compatibel (expand/contract), of n.v.t.
- [ ] Terugdraaien naar de vorige image is veilig, of hieronder staat waarom niet

**Documentatie**
- [ ] Gekoppeld aan een issue (`Closes #nr`); het issue staat op het bord op 👀 In review
- [ ] Ontwerpdocs of `docs/agents/` bijgewerkt bij een concept- of contractwijziging, of n.v.t.
- [ ] `AGENTS.md` alleen bij een nieuwe vaste werkregel, of n.v.t.
