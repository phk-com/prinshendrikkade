# Roadmap: Prins Hendrikkade.com

**Live plan:** alleen open en actief werk. Afgeronde epics staan in **[ROADMAP-archive.md](ROADMAP-archive.md)**.
Dit bestand blijft bewust kort, zodat mensen en agents het goedkoop kunnen lezen. De spelregels staan in [AGENTS.md §4](../AGENTS.md#4-roadmap-hoe-we-bijhouden-wat-er-gebeurt).

Legenda: ✅ klaar · 🔄 actief · ⏳ gepland

---

## Open werk in één oogopslag

| Epic | Wat het is | Wie | Pickup-trigger |
|---|---|---|---|
| 🔄 **E0** | Ontwerpfase: uitgangspunten, architectuur, beveiliging, onboarding, visueel ontwerp, werkregels | Mark | Loopt. Klaar zodra de open beslissingen hieronder genomen zijn |
| ⏳ **E1** | Fundament: monorepo, CI, branch protection, VPS + Coolify, preview- en rollback-flow | | Direct na E0 |
| ⏳ **E2** | Kern: identity (passkeys), uitnodigingen en onboarding, `core-api` en module-contract, module-beheer, auditlog, `@phk/ui` | | Na E1 |
| ⏳ **E3** | Forum-module | | Na E2 (eerste echte module, bewijst het contract) |
| ⏳ **E4** | Appplatform: manifest, sandbox, SDK, speelsessies, highscores, CI-speeltest | | Na E2, kan parallel met E3 |
| ⏳ **E5** | Games-module: arcade, speelscherm, beurtlobby, en een voorbeeldgame als sjabloon | | Na E4 |
| ⏳ **E6** | Chat-module (SSE) | | Na E2 |
| ⏳ **E7** | Bouwers en agents: `/agents`, `/bouwen`, `llms.txt`, OpenAPI, `phk-publish`-skill, `docs/agents/` | | Loopt mee met E2 tot E5. Afronden vóór de livegang |

---

## Actief en gepland

### 🔄 E0: Ontwerpfase

Alles vastleggen voordat er code komt. Zie [docs/ontwerp/](ontwerp/).

* ✅ E0.1: Analyse van de oude site en de keuze om alles opnieuw te bouwen, zonder oude data
* ✅ E0.2: Architectuur (twee niveaus, git als publicatieroute, terugdraaien) en de stackkeuze (TypeScript/SvelteKit)
* ✅ E0.3: Beveiligingsontwerp (passkeys, `MemberView`, het vier-ogen-principe, AVG)
* ✅ E0.4: Onboarding van leden en agents
* ✅ E0.5: Visueel ontwerp (Claude Design: richting 1c Portaal, logo 2d Zegel) en screenshots
* ✅ E0.6: Werkregels (`AGENTS.md`), roadmap en PR-template
* ⏳ E0.7: **Open beslissingen**
  - Wie komen er in `@phk-com/security`?
  - Hosting: het Hetzner-account (aan te maken door Mark) en wie de server beheert
  - Een externe pentest vóór livegang: ja of nee?
  - De oude dump en de oude sitebestanden: verwijderen of versleuteld archiveren?

### ⏳ E1: Fundament

Het doel: een lege app die veilig van pull request naar productie gaat en terug kan.

* ⏳ E1.1: De pnpm-monorepo met het skelet `core/web` (SvelteKit), `core/db`, `core/ui` en `sdk`, TypeScript strict, ESLint en Prettier
* ⏳ E1.2: CI: typecontrole, lint, Vitest, Playwright, dependency-cruiser met de basisregels, gitleaks (bestaat al), `pnpm audit`
* ⏳ E1.3: Repo-instellingen: branch protection op `main`, CODEOWNERS, squash-only, branches automatisch verwijderen na de merge, labels (`epic`, `agent-assisted`)
* ⏳ E1.4: De server: een VPS in de EU, gehard, met Coolify, Postgres en versleutelde back-ups
* ⏳ E1.5: Uitrollen: een image per commit, een preview per pull request met nepdata, de health-check, automatisch terugdraaien
* ⏳ E1.6: Oefenen met terugdraaien: bewust een kapotte release uitrollen en controleren dat het systeem automatisch terugdraait

**Acceptatie:** een pull request krijgt een preview-URL, een merge staat binnen enkele minuten live, en een kapotte
release wordt automatisch teruggedraaid.

---

## Recent afgerond

*(nog niets. Afgeronde epics komen hier als één regel en verhuizen naar het [archief](ROADMAP-archive.md))*
