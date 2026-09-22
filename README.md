# Prins Hendrikkade.com

*Authentic Since November 2002.* Dit is een herbouw van het clubhuis, nu als **open platform dat de leden zelf bouwen**.

> **Status: ontwerpfase.** Er is nog geen code. Deze repository bevat de uitgangspunten en de architectuur.
> Lees ze, stel vragen en doe voorstellen via issues of pull requests op de documenten.

## Waar het om draait

- **Chat, forum en games** voor een besloten groep vrienden. Lid word je alleen via een uitnodiging.
- **Iedere functie is een uitbreiding** die leden zelf kunnen toevoegen, aanpassen en verwijderen,
  en dat mag ook met hun eigen AI-agents (vibecoden).
- **Games** zijn HTML5-apps in een afgeschermde omgeving. Highscores worden via één uniforme SDK vastgelegd.
- **Git is de enige weg naar live:** branch → pull request → automatische controles → review → merge → uitrol → terugdraaien als het nodig is.
- **Ieder lid is beheerder**, en de kwaliteit wordt bewaakt door regels die voor iedereen gelden.
- **Veilig en privacyvriendelijk:** inloggen met passkeys, zo weinig mogelijk data, het vier-ogen-principe voor gevoelige acties, hosting in de EU.
- **Alles begint opnieuw:** er wordt geen data van de oude site overgenomen.

## Documenten

| Document | Waarover |
|---|---|
| [Architectuur](docs/ontwerp/architectuur.md) | De twee niveaus (apps en modules), het manifest, de SDK, highscores, CI, uitrollen en terugdraaien, en de hosting |
| [Beveiliging](docs/ontwerp/beveiliging.md) | Welke data we bewaren, het dreigingsmodel, inloggen, autorisatie, geheimen, de AVG |
| [Onboarding](docs/ontwerp/onboarding.md) | De uitnodigingsflow voor leden, `/agents` voor AI-agents en `/bouwen` als visuele uitleg |
| [Schermen](docs/ontwerp/schermen.md) | Screenshots van het design system en alle schermen (Claude Design) |
| [Design brief](docs/ontwerp/design-brief.md) | De opdracht voor het visuele ontwerp (richting "Portaal", logo "Zegel") |

## Genomen beslissingen

- Richting **1c "Portaal"** (indigo met koraal), logo **2d "Zegel"** (het ronde PHK-zegel).
- Lid worden alleen via een expliciete uitnodiging. Onbeperkt aantal uitnodigingen per lid, en een afkoelperiode van 7 dagen.
- Samenwerken en publiceren via GitHub (`phk-com`). `/bouwen` en `/agents` zijn bereikbaar maar `noindex`.
- Geen oude data: geen accounts, berichten, forumposts of games van de oude site.
- Pentest door een sterk AI-model zodra het platform grotendeels staat (#25).
- Beveiligingsreview door [`@phk-com/security`](https://github.com/orgs/phk-com/teams/security): markclausing, RobertTeunissen en wous2house.
- Hosting: eerst op de **NAS van wous2house** (in een eigen VM, met Coolify, achter een tunnel). Later eventueel een VPS in de EU.
- Techniek: **TypeScript** overal, met **SvelteKit**, **PostgreSQL** (Drizzle), **Better Auth** (passkeys) en een pnpm-monorepo waarin elke module een eigen package is.

## Nog open

Alle beslissingen uit de ontwerpfase zijn genomen. Wat er nu loopt, staat op het [project-bord](https://github.com/orgs/phk-com/projects/1).

## Geen geheimen in deze repository

Deze repository is (of wordt) **openbaar**. Zet er dus nooit wachtwoorden, API-sleutels, tokens, `.env`-bestanden,
databasedumps of persoonsgegevens in. Productiegeheimen staan alleen op de server.
Bij elke push en pull request controleert [gitleaks](.github/workflows/secrets.yml) de volledige historie.
Lekt er toch iets, beschouw het geheim dan als gecompromitteerd: **vervang het meteen**. Alleen verwijderen is niet genoeg.

## Meedoen

1. Lees de documenten hierboven, de werkregels in **[AGENTS.md](AGENTS.md)** (branches, review, Definition of Done)
   en wat er loopt op het **[project-bord PHK Roadmap](https://github.com/orgs/phk-com/projects/1)**.
2. **Idee of wens?** Open een [💡 idee](https://github.com/phk-com/prinshendrikkade/issues/new?template=1-idee.yml). Techniek is niet nodig.
3. **Werkt er iets niet?** Meld een [🐞 bug](https://github.com/phk-com/prinshendrikkade/issues/new?template=2-bug.yml).
4. **Wil je bouwen?** Pak een taak van het bord (zie `AGENTS.md` §4) en open een **pull request**.

Een beveiligingsprobleem meld je privé via [Report a vulnerability](https://github.com/phk-com/prinshendrikkade/security/advisories/new), niet in een issue.
