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
- Hosting bij een Europese partij (voorstel: een VPS bij Hetzner met Coolify).

## Nog open

- Wie komen er in `@phk-com/security` (de review van beveiligingscode)?
- Welke hosting kiezen we definitief, en wie beheert de server?
- Komt er een externe pentest vóór livegang?

## Geen geheimen in deze repository

Deze repository is (of wordt) **openbaar**. Zet er dus nooit wachtwoorden, API-sleutels, tokens, `.env`-bestanden,
databasedumps of persoonsgegevens in. Productiegeheimen staan alleen op de server.
Bij elke push en pull request controleert [gitleaks](.github/workflows/secrets.yml) de volledige historie.
Lekt er toch iets, beschouw het geheim dan als gecompromitteerd: **vervang het meteen**. Alleen verwijderen is niet genoeg.

## Meedoen

1. Lees de documenten hierboven.
2. Heb je een vraag of een idee? Open een **issue**.
3. Wil je een document aanpassen? Open een **pull request**.
