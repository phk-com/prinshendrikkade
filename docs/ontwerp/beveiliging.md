# Prins Hendrikkade.com: beveiliging van leden en ledendata

Dit document is bedoeld om de aanpak vast te leggen **voordat** er code wordt geschreven.
Het hoort bij `architectuur.md`.

## 1. Uitgangspunten

1. **Wat we niet hebben, kan niet lekken.** We slaan zo weinig mogelijk op.
2. **Veilig door de structuur, niet door afspraken.** Code *kan* niet bij gevoelige data,
   in plaats van "we spreken af dat niemand erbij kijkt".
3. **Ieder lid is beheerder, maar zware acties vragen altijd bevestiging.** Een overgenomen account
   mag nooit genoeg zijn om ledendata buit te maken.
4. **Wat in git staat, is openbaar.** Er komen nooit geheimen of productiedata in de repository.
5. **Alles binnen de EU** en volgens de AVG, met aantoonbare afspraken.

---

## 2. Welke data (inventaris)

| Gegeven | Nodig? | Hoe opgeslagen |
|---|---|---|
| Gebruikersnaam / weergavenaam | Ja | Gewoon (is zichtbaar voor leden) |
| E-mailadres | Ja (uitnodiging, herstel, meldingen) | **Versleuteld** in de database, plus een hash om op te zoeken. Nooit zichtbaar voor andere leden of voor modules en apps |
| Passkeys (publieke sleutels) | Ja | Gewoon; alleen de publieke sleutel, dus waardeloos voor een aanvaller |
| Wachtwoord | Alleen als iemand geen passkey kan gebruiken | Argon2id-hash |
| TOTP-geheim en herstelcodes | Bij 2FA | Versleuteld / gehasht |
| Avatar, bio | Optioneel | Gewoon |
| Directe berichten (DM's) | Ja (chat) | Versleuteld opgeslagen. Er is **geen** beheerscherm waarin iemand DM's van anderen kan lezen |
| IP-adressen | Alleen voor beveiliging | Alleen in het beveiligingslog, en na **30 dagen** automatisch gewist |
| Geboortedatum, adres, telefoon, woonplaats, ICQ/MSN | **Nee** | Wordt niet gevraagd |

### Geen oude data

Er wordt **niets** uit de oude site (dump uit 2010) overgenomen: geen accounts, geen berichten, geen forumposts
en geen games of scores. Iedereen begint opnieuw, via een uitnodiging.

- De oude dump bevat persoonsgegevens: e-mailadressen, **wachtwoorden als ongezouten MD5**, IP-adressen
  en privéberichten. Hij hoort niet bij het nieuwe project. Hij komt nooit in de repository en nooit op de server.
  `.gitignore` en een pre-commit-controle blokkeren `*.sql` en `backup*` voor de zekerheid.
- Omdat hij niet meer nodig is, is het advies om de dump en de oude sitebestanden **veilig te verwijderen**,
  of ze versleuteld en offline te archiveren als je ze om nostalgische redenen wilt bewaren.
- De oude `config.php` bevat databasewachtwoorden. Worden die ergens nog gebruikt, dan moeten ze worden gewijzigd.

---

## 3. Dreigingsmodel: de grootste risico's voor dít platform

| # | Dreiging | Waarom juist hier | Belangrijkste maatregel |
|---|---|---|---|
| D1 | **Accountovername** | Ieder lid is beheerder, dus één zwak wachtwoord is genoeg voor de hele site | Passkeys als standaard, verplichte 2FA, bevestiging vóór zware acties |
| D2 | **Een pull request die data weglekt** (kwaadaardig, gehackt account, of een agent die "creatief" is) | Iedereen kan code voorstellen en goedkeuren | Modules kunnen structureel niet bij gevoelige data (§5), strengere review voor beveiligingscode, geen productiegeheimen in CI |
| D3 | **Een kwaadaardige of lekke app/game** | Iedereen kan games publiceren | Afgeschermd frame op een eigen domein; de SDK geeft alleen een pseudoniem en een weergavenaam |
| D4 | **Gelekte back-ups of dumps** | Back-ups worden vaak vergeten | Back-ups versleuteld, sleutel apart bewaard; geen productiedata in testomgevingen |
| D5 | **Kwetsbare afhankelijkheden** (supply chain) | Vibecoding trekt makkelijk willekeurige packages binnen | Lockfiles, `composer audit` en `npm audit` in CI, Dependabot, een review-eis voor nieuwe packages |
| D6 | **Serverovername** | Eén VPS | Een geharde server, alleen SSH met sleutels, automatische updates, firewall, geen databasepoort open naar buiten |
| D7 | **Misbruik door een lid** (nieuwsgierigheid) | Een vriendengroep met onderlinge verhoudingen | Er is geen scherm dat ledendata of DM's laat zien; export van data gaat alleen via het vier-ogen-principe; alles komt in het auditlog |

---

## 4. Inloggen en accounts

### Inloggen
- **Passkeys (WebAuthn) zijn de standaard.** Ze zijn niet te phishen, er is geen wachtwoord dat kan lekken,
  en ze werken op elke moderne telefoon en laptop.
- **Een wachtwoord mag ook, maar alleen met verplichte TOTP-2FA**, met minimaal 12 tekens. Uitgelekte
  wachtwoorden worden geweigerd via een offline lijst of de k-anonymity-API van HIBP. Die API krijgt alleen
  de eerste 5 tekens van de hash te zien.
- Er is **geen** inloggen via Google, Facebook of Apple. Dat geeft geen extra afhankelijkheid van en
  geen datadeling met Amerikaanse partijen.
- **Remmen op pogingen:** een limiet per account en per IP-adres, oplopende vertraging, en een melding
  aan het lid bij verdachte pogingen.

### Uitnodigen en registreren
- Een lid maakt een **uitnodiging**: eenmalig te gebruiken, 7 dagen geldig, en optioneel gekoppeld aan een e-mailadres.
- In het auditlog staat wie wie heeft uitgenodigd. Uitnodigingen kunnen worden ingetrokken.
- Bij de eerste keer inloggen moet het nieuwe lid meteen een passkey of 2FA instellen. Anders is het account niet te gebruiken.
- Onbeperkt aantal uitnodigingen per lid, met een **afkoelperiode van 7 dagen** voordat een nieuw lid
  gevoelige acties kan goedkeuren of als reviewer meetelt (zie `onboarding.md`).
- GitHub koppelen gaat via OAuth met de scope `read:user`, alleen om de gebruikersnaam te verifiëren en nooit om in te loggen.
  Het lidmaatschap van de GitHub-organisatie volgt het lidmaatschap van de site: wie de site verlaat, verdwijnt automatisch uit de organisatie.

### Herstel als je je passkey of telefoon kwijt bent
- 10 eenmalige **herstelcodes**, die je bij het registreren krijgt.
- **Herstel via leden:** zonder herstelcodes bevestigen **twee andere leden** met hun eigen passkey
  dat jij het bent. Dan krijg je een herstellink die 1 uur geldig is, en worden al je sessies beëindigd.
- Er is bewust **geen** herstel via alleen een e-maillink. Dan zou de beveiliging van je mailbox die van de site bepalen.

### Sessies
- Cookies `Secure`, `HttpOnly`, `SameSite=Lax`, een `__Host-`-prefix, en een nieuwe sessie-ID na het inloggen.
- Een sessie verloopt na 30 dagen, of eerder bij 14 dagen inactiviteit.
- Onder "Mijn apparaten" kun je sessies bekijken en beëindigen, en afmelden op alle apparaten.
- Een melding (in de site en per mail) bij het inloggen op een nieuw apparaat.

### Persoonlijke API-tokens (voor bots en statistieken)
- Tokens hebben scopes (`scores:read`, `profile:read`) en een verloopdatum (maximaal 90 dagen), en worden gehasht opgeslagen.
- Een token geeft **nooit** toegang tot ledenbeheer, e-mailadressen of DM's.
- Tokens worden herkend door secret scanning op GitHub (vast voorvoegsel, bijvoorbeeld `phk_`).

---

## 5. Autorisatie: "ieder lid is beheerder", maar veilig

Er zijn drie soorten acties:

| Soort | Voorbeelden | Vereiste |
|---|---|---|
| **Normaal** | Posten, chatten, spelen, module-instellingen bekijken | Ingelogd |
| **Beheer** | Module aan/uit of instellen, app uitzetten (noodschakelaar), uitnodiging maken, uitrol terugdraaien | **Opnieuw bevestigen** met je passkey of 2FA (geldig 10 min), een regel in het auditlog en een melding in de chat |
| **Gevoelig** | Een lid verwijderen of blokkeren, het herstel van een ander goedkeuren, een data-export van iemand, de toegangsregels wijzigen | Opnieuw bevestigen, plus **goedkeuring door een tweede lid** (vier ogen), plus een auditregel die niet te wijzigen is |

### Structurele afscherming in de code

Dit is de belangrijkste maatregel tegen D2, en hij wordt afgedwongen met architectuurtests in CI:

- De `users`-tabel en de tabellen voor inloggen zijn alleen bereikbaar vanuit `core/Identity`.
- Modules krijgen leden alleen via het contract `Members`. Dat geeft een `MemberView` terug met
  `id`, `weergavenaam`, `avatar` en `online`, en **nooit** een e-mailadres, IP-adres of inloggegevens.
- Een architectuurtest laat CI falen als een module `User`, `DB::table('users')` of de onderdelen
  voor versleuteling en inloggen gebruikt.
- **Elke route moet** de middleware `auth` hebben en een policy. Een test gaat alle routes langs en faalt als er een ontbreekt.
- Apps en games krijgen via de SDK alleen een **pseudoniem per app** en de weergavenaam. Een game kan
  spelers dus niet over verschillende apps heen volgen.

### Strengere review voor beveiligingscode

`core/Identity`, `core/Security`, `docs/agents/` en `AGENTS.md` (want die sturen wat agents doen), de CI-configuratie, de deployconfiguratie en `composer.json` of `package.json`
(nieuwe packages) hebben **2 goedkeuringen** nodig, waarvan minstens één van een lid uit de groep
`@phk-com/security`. Dat is een kleine groep vrijwilligers van 2 à 3 leden. Iedereen blijft beheerder;
de groep telt alleen mee bij deze paden.

---

## 6. Geheimen, omgevingen en CI

- Productiegeheimen (de `APP_KEY`, de databasewachtwoorden, de mailserver) staan **alleen in Coolify**
  op de server, en nooit in git of in GitHub Actions.
- **Preview-omgevingen per pull request draaien op nepdata** die bij het opstarten wordt gegenereerd.
  Er komt nooit een kopie van productie in terecht.
- Uitrollen naar productie gaat via een beveiligde GitHub-omgeving die alleen vanaf `main` start.
  Pull requests uit forks krijgen geen geheimen.
- Secret scanning met **gitleaks** in CI en als pre-commit-controle. GitHub push protection staat aan.
- De versleutelingssleutel voor e-mailadressen en DM's is een aparte sleutel naast de `APP_KEY`, zodat hij los kan worden vervangen.

---

## 7. Server, opslag en back-ups

- **Hosting:** Hetzner in Duitsland of Finland (of TransIP in Nederland), met een **verwerkersovereenkomst** (DPA).
- **Server:** alleen inloggen met SSH-sleutels, geen root-login, een firewall die alleen poort 80, 443 en 22 openlaat
  (en 22 bij voorkeur alleen via een VPN of Tailscale), automatische beveiligingsupdates en fail2ban.
- De database luistert alleen intern en draait onder een eigen gebruiker met minimale rechten.
- **Back-ups:** dagelijks met restic, **versleuteld**, naar opslag in de EU, 30 dagen bewaard.
  De sleutel wordt apart bewaard (bij 2 leden). Elk kwartaal wordt getest of terugzetten werkt.
- **HTTP-headers:** HSTS (preload), een strikte CSP met nonces, `X-Content-Type-Options`,
  `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy`, en `frame-ancestors` beperkt.
- Het domein voor apps (`apps.`) deelt geen cookies met het hoofddomein.

---

## 8. Controleren dat het ook echt klopt

| Wanneer | Wat |
|---|---|
| Elke pull request | Autorisatietests voor alle routes, architectuurtests (§5), tests die proberen bij data van een ander lid te komen, gitleaks, `composer audit` en `npm audit`, Larastan |
| Elke preview | Een baseline-scan met **OWASP ZAP** op de preview-URL |
| Wekelijks | Dependabot-updates en een controle van de beveiligingsheaders |
| Vóór livegang | Een review tegen de **OWASP ASVS** niveau 2 voor authenticatie, sessies en toegangscontrole; eventueel een externe pentest van een dag |
| Doorlopend | `/.well-known/security.txt`, en meldingen van mislukte inlogpogingen en beheeracties in het beveiligingslog |

---

## 9. AVG

- Een **register van verwerkingen** (klein, één pagina) en een **privacyverklaring** in gewone taal.
- Zelf regelen op de profielpagina: je data inzien, **exporteren** (JSON) en je **account verwijderen**.
  Bij verwijderen worden je posts geanonimiseerd, of verwijderd als je dat kiest.
- **Bewaartermijnen:** IP-adressen 30 dagen, het beveiligingslog 1 jaar, het auditlog van beheeracties 2 jaar,
  en data van verwijderde accounts direct weg (uit back-ups na 30 dagen).
- Een **procedure voor datalekken**: wie beslist er, melden bij de Autoriteit Persoonsgegevens binnen 72 uur,
  en de leden informeren.
- Verwerkers: alleen EU-partijen (hosting, back-ups, mail). Voor de mail bijvoorbeeld een Europese
  SMTP-dienst of die van de hoster.

---

## 10. Beslissingen die nog genomen moeten worden

1. **Passkeys als standaard**, met een wachtwoord plus verplichte 2FA als alternatief? *(advies: ja)*
2. **Herstel via twee andere leden** in plaats van een e-maillink? *(advies: ja)*
3. **Het vier-ogen-principe voor gevoelige acties** (§5)? *(advies: ja)*
4. **Een groep `@phk-com/security`** van 2 à 3 leden voor de review van beveiligingscode? Wie zitten erin?
5. **De oude dump en de oude sitebestanden:** verwijderen, of versleuteld offline archiveren?
6. **Een externe pentest vóór livegang:** wel of niet? *(ongeveer een dag werk, optioneel)*
