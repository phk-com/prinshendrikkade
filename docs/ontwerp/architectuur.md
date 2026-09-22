# Prins Hendrikkade.com: architectuur voor een open platform

## Uitgangspunten

0. **Alles begint opnieuw.** Er wordt geen data uit de oude site overgenomen: geen accounts, berichten, forumposts, games of scores.

1. **Iedere functie is een uitbreiding.** Chat, forum en elke game volgen dezelfde weg:
   **manifest → pull request → automatische controles → review → merge → uitrol → (terugdraaien)**.
2. **Git is de enige plek om samen te werken en te publiceren.** Er is geen apart uploadportaal.
   Wat in `main` staat, staat live.
3. **Ieder lid is beheerder.** De kwaliteit wordt daarom bewaakt door **regels in de pipeline** en niet door rangen:
   niemand kan zonder groene controles en zonder een tweede paar ogen iets live zetten, ook de beheerders niet.
4. **Alles is terug te draaien.** Elke uitrol is een onveranderlijke build die je opnieuw kunt activeren.

---

## Eén repository

```
phk-com/prinshendrikkade/
  core/                 Laravel-kern: accounts, module-contract, SDK-bridge, API
  modules/              niveau 2: PHP-modules (Chat, Forum, Games, Leden, …)
    Forum/module.json
  apps/                 niveau 1: afgeschermde apps en games (HTML/JS)
    smack-the-penguin/phk-app.json
  sdk/                  phk-sdk.js + een lokale test-SDK
  docs/agents/          bron voor /agents, /agents.md en llms.txt (zie onboarding.md)
  docs/bouwen/          bron voor de visuele uitleg op /bouwen
  AGENTS.md             de regels voor agents (verwijst naar docs/agents/)
  .github/              CI, CODEOWNERS en templates
```

Alles zit in één repository. Eén pull request kan daardoor een game, een module en de documentatie tegelijk
aanpassen, en dat wordt dan ook samen getest en samen teruggedraaid.

## Twee niveaus

Het verschil tussen de niveaus zit in wat een uitbreiding mag. De weg naar live is voor beide hetzelfde.

| | **Niveau 1: Apps** (games, widgets, kleine tools) | **Niveau 2: Modules** (chat, forum, games-platform) |
|---|---|---|
| Wat | HTML/JS/CSS in een afgeschermd frame op een apart domein | PHP-code die in de server draait |
| Toegang | Alleen via de PHK-SDK en alleen de rechten die in het manifest staan | Volledig, via het module-contract |
| Review nodig | 1 ander lid | 2 andere leden (of 1 als het alleen de eigen module betreft) |
| Extra CI | Validatie van manifest en bundel, en een headless speeltest | Tests, architectuurregels, statische analyse en migratietests |

Omdat iedereen kan mergen, is de **afscherming van apps** de belangrijkste beveiliging. Zelfs een slechte
game kan niet bij accounts of data. Modules hebben meer macht en krijgen daarom een strengere review.

---

## Niveau 1: apps en games

### Manifest: `apps/<slug>/phk-app.json`

```json
{
  "$schema": "https://prinshendrikkade.com/schema/app-v1.json",
  "slug": "smack-the-penguin",
  "name": "Smack the Penguin",
  "type": "game",
  "version": "1.2.0",
  "entry": "index.html",
  "authors": ["jod"],
  "description": "Mep de pinguïn zo ver mogelijk.",
  "thumbnail": "thumb.png",
  "viewport": { "aspect": "4:3", "minWidth": 320 },
  "permissions": ["scores", "storage"],
  "leaderboards": [
    { "id": "distance", "label": "Afstand", "unit": "m", "order": "desc", "format": "integer", "max": 100000 }
  ],
  "widgets": [{ "id": "top5", "label": "Top 5 pinguïnmeppers", "columns": 1 }]
}
```

### Hoe een app draait
- Een app wordt geserveerd op `apps.prinshendrikkade.com/<slug>/<versie>/`, in een
  `<iframe sandbox="allow-scripts">` (zonder `allow-same-origin`), met een strikte CSP.
  De app kan dus niets van buiten laden en nergens anders naartoe praten.
- De SDK praat via `postMessage` met de hoofdpagina. Die controleert de rechten en doet het API-verzoek namens het ingelogde lid.

```js
import { phk } from "https://prinshendrikkade.com/sdk/v1/phk-sdk.js";
const session = await phk.startSession();
await phk.scores.submit({ leaderboard: "distance", value: 412, session });
const top = await phk.scores.top({ leaderboard: "distance", limit: 10 });
```

### Highscores: één uniforme vastlegging
- Elke score wordt vastgelegd met: app, versie (de git-SHA), leaderboard, lid, waarde, speelsessie, tijdstip en eventuele metadata.
- De server vraagt een geldige, ongebruikte speelsessie en een minimale speelduur, begrenst het aantal inzendingen,
  en controleert `min`/`max` uit het manifest.
- Seizoenen: als een nieuwe versie de spelregels verandert, kan de maker in het manifest een nieuw seizoen starten.
  De oude scores blijven in het archief.

### Automatische controles in CI (moeten slagen voor een merge)
- Het manifest klopt met het schema, de slug is uniek, het versienummer is omhooggegaan en de rechten zijn bekende rechten.
- De bundel is kleiner dan de limiet (bijvoorbeeld 25 MB), bevat alleen toegestane bestandstypen en verwijst niet naar externe URL's.
- **Een headless speeltest** met Playwright: de app laadt zonder console-fouten, de SDK-handshake slaagt,
  de app werkt op telefoonformaat en een testscore komt aan bij de test-SDK.
- De thumbnail wordt automatisch gemaakt en er komt een preview-URL in de pull request.

---

## Niveau 2: modules

```
modules/Forum/
  module.json            naam, versie, afhankelijkheden, instellingen-schema, rechten
  src/ForumModule.php    implementeert PHK\Contracts\Module
  routes/  database/migrations/  resources/views/  tests/
```

Het contract `Module` vraagt om: `register()`, `navigation()`, `widgets()`, `settingsSchema()`,
`install()`, `uninstall()` en `healthCheck()`.

In de site (scherm 3f uit het ontwerp) kan ieder lid modules **aan- en uitzetten en instellen**. Dat zijn
instellingen, geen code, en elke wijziging komt in het auditlog. **Toevoegen of verwijderen** van een module
gaat altijd via een pull request.

**Controles in CI:**
- Pest-tests per module, plus **contracttests die automatisch voor elke module draaien**:
  installeren, deïnstalleren, heen en terug migreren, health-check, en elk dashboardblok moet kunnen renderen.
- **Architectuurregels** (Pest Arch): een module mag alleen de publieke kern-API gebruiken en niet in de
  interne code van andere modules grijpen, en debugcode mag er niet in staan.
- Larastan (statische analyse), Pint (codestijl) en `composer audit` (bekende kwetsbaarheden).
- Een smoketest met Playwright op de belangrijkste pagina's, op telefoon- en desktopformaat.

---

## Het samenwerkingsmodel: ieder lid is beheerder

| Regel | Hoe het wordt afgedwongen |
|---|---|
| Niemand pusht direct naar `main` | Branch protection, ook voor beheerders ("include administrators") |
| Alles gaat via een pull request met groene CI | Verplichte status checks |
| Altijd een tweede paar ogen | Minstens 1 goedkeuring door een ander lid; bij wijzigingen aan `core/` of `modules/` 2 goedkeuringen |
| Je eigen werk beheer je zelf | CODEOWNERS: de maker van een app of module moet wijzigingen daarin goedkeuren (tenzij hij onbereikbaar is: zie noodprocedure) |
| Geschiedenis blijft intact | Geen force-push, alleen squash-merge en getekende commits (optioneel) |
| Iedereen kan ingrijpen | Noodschakelaar in de site: een app of module direct uitzetten, met auditlog en een melding in de chat |
| Iedereen kan terugdraaien | "Revert"-knop op GitHub, of een handmatige rollback-actie (zie hieronder) |

Agents werken op precies dezelfde manier: ze maken een branch, committen, openen een pull request en
lezen de CI-uitslag. Er is geen aparte route en er zijn geen uitzonderingen. `AGENTS.md` beschrijft de regels,
en `/agents` op de site toont dezelfde inhoud (zie `onboarding.md`).

### Voor agents en vibecoders
- `AGENTS.md` in de repo: de structuur, het manifest, de SDK, de regels en hoe je lokaal test.
- `make new-game name=snake` maakt een app-skelet aan, en `make dev` start een lokale server met de test-SDK.
- `/agents` (één URL voor agents), `/llms.txt` en `/bouwen` (visueel): gegenereerd uit `docs/`, zie `onboarding.md`.
- `/api/v1/openapi.json`: de runtime-API (scores, sessies, opslag), bijvoorbeeld voor bots of statistieken.
- Een Claude Code-skill `phk-publish` in de repo die de hele flow kent, van skelet tot pull request.

---

## Uitrollen en terugdraaien

```
PR geopend ──► CI ──► preview-omgeving  pr-123.test.prinshendrikkade.com (met testdata)
merge naar main ──► CI ──► container-image  phk:<sha> ──► databaseback-up ──► migraties
      ──► nieuwe versie live ──► health-check ──fout──► automatisch terug naar de vorige image
```

- Elke build is een **container-image met de git-SHA** als tag. Terugdraaien is de vorige image starten,
  plus `migrate:rollback` als dat nodig is.
- Migraties moeten **terugdraaibaar** zijn. CI test `migrate` → `rollback` → `migrate` voor elke pull request.
- Daarnaast is een `git revert` op `main` altijd mogelijk. Dat rolt gewoon uit als een nieuwe versie.
- Dagelijkse back-ups van de database en de uploads naar opslag buiten de server (in de EU).

---

## Hosting (Europees)

Dit platform heeft meer nodig dan shared hosting: containers, previews per pull request, websockets voor de chat,
achtergrondtaken en een apart domein voor apps. Het advies is **één VPS bij een Europese partij, met een
open-source deploylaag erop**:

| Onderdeel | Voorstel | Waarom |
|---|---|---|
| Server | **Hetzner Cloud** (Duitsland/Finland), VPS met 4 GB RAM | Goedkoop, betrouwbaar, EU-bedrijf. Nederlandse alternatieven: **TransIP** VPS of **Hostnet** |
| Deploys | **Coolify** (open source, draait op de eigen VPS) | Koppelt met GitHub, maakt preview-omgevingen per pull request, geeft terugdraaien per image, regelt SSL en heeft een webinterface |
| Database | MySQL/MariaDB of PostgreSQL op dezelfde VPS | Eenvoudig; back-ups via Coolify |
| Back-ups | Hetzner Storage Box of S3-compatibele EU-opslag | Buiten de server en binnen de EU |
| Chat realtime | Laravel Reverb (websockets) op dezelfde VPS | Geen externe (Amerikaanse) dienst nodig |
| DNS/domein | Bij de huidige registrar, of TransIP | `prinshendrikkade.com`, `apps.`, `*.test.` |

Kosten: naar verwachting **zo'n €5–15 per maand** voor de VPS en de back-ups. De actuele prijzen moeten nog worden gecontroleerd.

Er is een eenvoudiger alternatief als je geen server wilt beheren: **Ploi** (een Nederlands bedrijf) als beheerlaag
op een Hetzner- of TransIP-server. Dat geeft zero-downtime deploys en terugdraaien, maar minder goede previews
per pull request.

---

## Bouwvolgorde

1. **Fundament:** de repo-structuur, `AGENTS.md`, CI, branch protection, Coolify op een VPS en een preview- en rollback-flow die werkt.
2. **Kern:** accounts en uitnodigingen, het module-contract, module-beheer en het auditlog, en het design system als Blade-componenten.
3. **Forum-module.**
4. **Appplatform:** het manifest-schema, de afgeschermde weergave, de SDK, speelsessies, highscores en de CI-speeltest.
5. **Games-module:** de arcade en lobby, en een voorbeeldgame als sjabloon.
6. **Chat-module** met Reverb.
7. **Onboarding:** de uitnodigingsflow, `/agents`, `/bouwen`, `llms.txt`, OpenAPI en de `phk-publish`-skill (zie `onboarding.md`).
