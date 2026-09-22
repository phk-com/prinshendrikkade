# Design brief — Prins Hendrikkade.com (2026)

## Wat is het
Een besloten community-site voor een vriendengroep die sinds november 2002 bestaat
("Authentic Since November 2002"). De oude site was PostNuke met het Infonuke-thema.
Er komt een nieuwe site. Die draait om vier dingen: **chat**, **games**, **forum** en
**modulariteit** (onderdelen moeten los aan en uit kunnen en er moeten later nieuwe bij kunnen).

Nieuws en fotoalbums komen niet terug. Er wordt **geen oude inhoud** overgenomen: geen accounts, berichten, forumposts of games. Alles begint leeg, dus lege toestanden en de eerste keer gebruiken zijn belangrijk.

## Doelgroep en gevoel
- Zo'n 20 tot 150 actieve leden die elkaar kennen: vrienden en oud-huisgenoten, rollenspelers, Diplomacy-spelers.
- Het moet voelen als een eigen clubhuis en niet als een social-media-platform. Informeel, met humor, een beetje nerdy (Magic: The Gathering, D&D-campagnes, World of Warcraft, Diplomacy).
- Een knipoog naar het early-2000s-webportaal mag (blocks, een shoutbox, highscores), maar vertaald naar een modern en rustig ontwerp. Geen retro-pastiche.
- Mobile-first: de meeste bezoeken komen van een telefoon. De chat moet op een telefoon aanvoelen als een messaging-app.

## Schermen die nodig zijn
1. **Dashboard / home** — opgebouwd uit modulaire blokken (widgets) die per module bijdragen:
   "wie is online", laatste forumposts, chat-preview, highscores van deze week, openstaande game-beurten.
   Op desktop is het een grid, op mobiel een enkele kolom. Leden kunnen blokken verbergen en herschikken
   (dat kon op de oude site ook al).
2. **Chat**
   - Kanalen, zoals #algemeen en #games, plus een kanaal per campagne. Daarnaast directe berichten.
   - Berichtlijst met avatars, tijdstempels, reacties/emoji, links met previews en "is aan het typen".
   - Mobiel: een kanaallijst, en daarop tikken opent een scherm op volledige hoogte met de invoerbalk onderaan (let op het toetsenbord).
3. **Forum**
   - Een overzicht per categorie (Algemeen, RPG, Diplomacy, Gaming, Members Only), met per forum het aantal topics en de laatste post.
   - Een topiclijst met ongelezen-markering en vastgepinde topics.
   - Een draadweergave. Lange draden moeten goed werken, met paginering en "spring naar ongelezen". Citaten moeten goed ogen.
   - Een editor met Markdown, citeren en bijlagen.
4. **Games**
   - Een arcade-overzicht: een grid met game-kaarten (thumbnail, titel, jouw beste score, de topscore).
   - Een game-speelscherm: de game staat centraal en schaalt mee (alle games zijn HTML5, gemaakt door leden). Ernaast of eronder staat een highscore-tabel.
   - Beurtgebaseerde spellen (zoals Diplomacy en campagnes) hebben een lobby: lopende potjes, "jij bent aan de beurt" en uitnodigen.
5. **Profiel en leden** — avatar, lid sinds, aantal posts, beste scores, online-status.
6. **Beheer: modules** — een lijst van modules met een aan/uit-schakelaar, instellingen per module en welke blokken een module levert.
7. **Inloggen / registreren** — dit is een besloten site, dus registreren gaat op uitnodiging.

## Modulariteit (belangrijk voor het design system)
Elke module (Chat, Forum, Games en later bijvoorbeeld een Agenda of Polls) levert:
- een navigatie-item (icoon + label),
- eigen pagina's,
- nul of meer dashboardblokken van een vaste breedte: 1, 2 of 3 kolommen.

Het design moet dus draaien op een **generiek blok-component en generieke navigatie** waar een
nieuwe module zonder nieuw ontwerp in past.

Navigatie op mobiel gaat via een onderbalk met maximaal vijf items (Home, Chat, Forum, Games, Meer).
Op desktop is dat een zijbalk.

## Design-system-wensen
- Een lichte en een donkere modus, allebei volwaardig. Donker is de standaard (het is voor een deel een gamingclub).
- Eén herkenbare accentkleur. Een variant met een tweede accent per module mag.
- Lettertypes: een karaktervolle kop (mag een beetje eigenzinnig zijn) en een zeer leesbare tekstletter. Voor chat, code en scores een monospace.
- Componenten: knoppen, invoervelden, tabs, badges (ongelezen of online), avatar met statusstip, lijstitems,
  kaarten, dashboardblok, berichtbubbel, forumpost, citaat, highscore-tabel, lege toestanden, toasts en een modal of sheet (op mobiel een bottom sheet).
- Toegankelijkheid: contrast volgens AA, aanraakdoelen van minimaal 44px en duidelijke focus-states.

## Techniek (ter info voor de ontwerper)
De site wordt gebouwd met SvelteKit (TypeScript): server-rendered HTML met lichte interactiviteit, gehost op een eigen server (VPS) in de EU. Het design system wordt Svelte-componenten met CSS-variabelen.
Er zijn geen zware SPA-animaties nodig. CSS-transities en design tokens (CSS-variabelen) zijn prima.
