# Existing software for Teams, a Week, and Game Coordinator Lobbies

Research for [What existing software already runs Teams, a Week, and Game Coordinator Lobbies?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/2). Primary sources only (vendor docs, official APIs, first-party repos). Judged against AFK v1, not a generic inhouse.

## Question

What existing open-source or commercial software already runs some combination of standing-team registration, a double-elimination Week, Steam identity, and Dota 2 Game Coordinator Lobbies (create, invite Steam IDs, seat Radiant/Dire, launch, detect a result)?

## AFK v1 bar

From `CONTEXT.md` and the map issue [AFK v1 — the way to a spec](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/1):

- **Team**: standing Filipino grassroots 5-stack with one Captain and at least five Steam-linked Players. Registers once and enters Weeks. Not a pickup stack.
- **Week**: one weekly running, double-elimination, through to a winner without an Organizer sitting each Lobby.
- **Match / Lobby**: the series creates the Dota 2 practice lobby — invite, Radiant/Dire, launch, result. Captains do not create lobbies.
- **Player**: identified by a connected Steam profile.
- **System of record**: a web app is the intended SoR. Discord-first is still an open decision.
- Locked Week rules that no generic tournament host encodes: top 2 of a Week auto-enter the next Week; a Team may opt in at registration to re-enter; a Team may disband only after two losses in that Week.

A candidate “covers AFK v1” only if it can be the product: standing Teams, weekly double-elim, automated GC practice lobbies, Steam identity, and a web SoR the project can own or adopt without becoming a FACEIT/Rivals tenant.

## Answer

**Nothing existing is AFK v1.** FACEIT is the closest commercial stack (premade teams with a leader, Dota 2 Game Integration that creates the lobby and invites the roster, championships that include double-elimination, web as their SoR). It cannot be adopted as AFK: FACEIT is the system of record, it is closed, and it does not encode AFK’s weekly standing-team Week. Rivals is the closest Discord-first Dota 2 tournament product (it claims automated Dota 2 lobbies, brackets, live stats, and result verification) but is Discord-first and prize-pool infrastructure, not a self-hosted web SoR for standing 5-stacks. Open source splits the problem: GC lobby libraries (`ValvePython/dota2`, `paralin/go-dota2`), inhouse bots (ScheduleBot, Dota2-EU-Ladder, Sentinel), and game-agnostic bracket APIs (Challonge, start.gg, Toornament). Those pieces can be reused. The product has to be built.

## Coverage matrix

Legend: **yes** = first-party docs/repo state it; **partial** = related capability, not AFK’s shape; **no** = absent or contradicted; **n/a** = library, not a product.

| Candidate | Standing Team + Captain | Steam-linked Players | Double-elim Week | Series-created GC Lobby | Detect result | Web SoR the project owns |
|---|---|---|---|---|---|---|
| FACEIT | yes (premade team + leader) | partial (`game_player_id`) | yes (championship format) | yes (Dota 2 Game Integration) | yes (match results API) | no (FACEIT is SoR) |
| Rivals | undocumented | undocumented | claims 5v5 + weekly leagues | claims auto Dota 2 lobby | claims live stats / no self-report | no (Discord-first) |
| Tourney Bot | no | no | yes (bracket types) | no | no (manual score) | no (Discord-first) |
| Challonge | partial (named participants) | no | yes | no | no (score report API) | no (Challonge is SoR) |
| start.gg | partial (Entrant may be a Team) | no | yes | no | no (set reporting) | no (start.gg is SoR) |
| Toornament | yes (team + lineup) | partial (custom fields) | yes | no (Game API is LoL + CS:GO) | no (match reports) | no (Toornament is SoR) |
| Challengermode | partial (tournament platform) | undocumented for Dota | yes (hosts Dota DE) | no public Dota GC lobby API | via game-dev integration | no (their platform) |
| ScheduleBot (Dota) | no (pickup inhouse) | yes (Steam friend + code) | no | partial (create, invite, launch; one at a time) | no | no (Discord + Postgres) |
| Dota2-EU-Ladder | no (queued pickup, role balancer) | implied (invites) | no | yes (invite, seat, results) | yes | partial (optional Django site) |
| Dota2SentinelBot | no | Steam network | no | yes (host + track) | yes | no |
| Dota2LobbyBot | no | invite Steam users | no | yes (create, invite, start) | no | n/a (MySQL trigger) |
| ValvePython `dota2` | n/a | invite by `steam_id` | n/a | yes (full lobby API) | yes (`request_match_details`) | n/a |
| `node-dota2` | n/a | invite by Steam ID | n/a | yes (archived 2022) | yes (`requestMatchDetails`) | n/a |
| `paralin/go-dota2` | n/a | lobby ops | n/a | yes (lobby tracking + ops) | library, not a product | n/a |
| Steam OpenID + Web API | n/a | yes (Claimed ID = SteamID64) | n/a | no | yes (`GetMatchDetails.radiant_win`) | n/a |
| RD2L | no (individual signup, draft teams) | community site | weekend tourneys, not a Week | not documented as product | not documented | their site, not reusable software |

## Commercial products that run some of the loop

### FACEIT — closest hosted stack; cannot be AFK

FACEIT is a web competitive platform. The Data API exposes **teams** with a `leader`, `members`, and `team_type` including `"premade"`, plus championship **subscriptions** with `leader`, `roster`, and `substitutes`.[^faceit-data] Championships, matches, brackets, and match **results** (`results.winner`, `detailed_results`) are first-class. Rosters carry `game_player_id` / `game_player_name` as the in-game identity.[^faceit-data]

FACEIT’s own championship rule pages for Dota 2 state that **the platform creates the lobby and invites the team**:

> “A lobby will be automatically created by the Faceit Dota 2 Game Integration system. All team members will be invited to that lobby and will be able to join.”[^faceit-cup-rules]

The same wording appears on later FACEIT Dota 2 qualifier rule pages (“FACEIT Dota 2 Game Integration system… Each team member will receive an invitation”).[^faceit-ewc-rules] That is series-created GC practice-lobby automation: captains do not create the lobby.

FACEIT also documents **double-elimination** as an official tournament format (winners’ bracket, losers’ bracket, grand finals).[^faceit-de] Live FACEIT Dota 2 championships publish “Tournament System: Double Elimination Bracket” in their rules.[^faceit-ewc-de]

**What FACEIT cannot do for AFK v1**

- FACEIT is the system of record. The public API is a **read** Data API (`GET` championships, teams, matches, results), not a way to run AFK’s own web app as SoR.[^faceit-data]
- It is closed, hosted, and multi-game. AFK v1 is an open-source web app for Filipino grassroots standing 5-stacks.
- It does not encode the locked Week rules (top 2 auto-enter next Week; opt-in re-entry; disband only after two losses).
- Steam is present only as `game_player_id` on a FACEIT player, not as “Steam is required identity” for an AFK Player record the project owns.
- Brand, prizes, and anti-cheat are FACEIT product surface; AFK v1 explicitly leaves prizes and full anti-cheat out of scope.

Adopt FACEIT and you are running a FACEIT cup, not AFK.

### Rivals — closest Discord-first Dota 2 tournament bot; not the web SoR

Rivals’ organiser page is explicit: “Turn your Discord server into a fully-functioning tournament league. Rivals handles brackets, lobbies, payments, and payouts.” Feature list includes “Bracket creation and management”, “**Automatic lobby creation inside Dota 2**”, “Live stats tracking and result verification”.[^rivals-org] Player flow: register/pay in Discord, “Lobby is created automatically. Your stats are tracked live. No manual reporting.”[^rivals-players] Developer page: “Dota 2 — full end-to-end automation (lobbies, stats, results, payouts)” and “Verified Match Results… No self-reporting.”[^rivals-dev] `/tournament` configures competition (fees, format, schedule) inside Discord; a web signup (`rivalsapp.com`) exists but the product copy is “Rivals lives in Discord”.[^rivals-players]

That is the commercial product the ticket named. It claims the hard Dota piece (automated lobby + result) plus brackets and weekly leagues.

**What Rivals cannot do for AFK v1**

- **Discord-first.** AFK’s intended SoR is a web app; Discord is an open notification-channel decision, not the product.
- Public vendor pages do **not** document standing Teams (Captain + ≥5 Steam-linked Players who register once and enter Weeks). They talk players, Discord servers, 1v1, 5v5, weekly leagues, and entry fees.
- Prize-pool escrow, organiser fees (they take 10%), and payouts are the product.[^rivals-org] AFK v1 leaves paying prizes out of scope.
- No public API/docs for create-lobby / invite Steam IDs / seat Radiant/Dire / launch. Claims are marketing pages, not a GC protocol reference.
- Closed, hosted, not adoptable as the open-source AFK app.

### Tourney Bot — Discord brackets only

Tourney Bot “supports all bracket types; Single Elimination, Double Elimination, Round Robin, Swiss, Group Stages, FFA & Race” and “does not rely on any external sites” — Discord is the host.[^tourneybot] Player flow is chat → score → results → bracket.[^tourneybot]

**Cannot:** Steam identity, GC lobbies, automatic result detection, standing Dota Teams, web SoR.

## Bracket-class platforms (Week structure without a Lobby)

These can draw a double-elim Week. None of them open a Dota 2 practice lobby.

### Challonge

API v2.1 `tournament_type` enum includes `double elimination` with `double_elimination_options` (`split_participants`, `grand_finals_modifier`).[^challonge-create] Participants are created as names (optionally Challonge users); match scores are reported through the API.[^challonge-gi] Challonge will **not** automatically start a tournament at `starts_at`.[^challonge-create] Game Integration is a developer integration for *your* game, not Valve’s GC.[^challonge-gi]

**Cannot:** Steam-required Players, standing 5-stacks as a first-class roster, series-created Dota lobbies, automatic Dota result ingest. Challonge is the SoR.

### start.gg

An **Entrant** “is either a Team or Player in an Event.” Events have phases, phase groups, sets, standings.[^startgg-glossary] The platform documents **Double Elimination** as a standard format.[^startgg-de] Dota 2 events exist on the site (e.g. “the Master Cup - Dota 2 Tournament”).[^startgg-dota] The public API is GraphQL for reading/writing tournament data, not for talking to the Dota GC.[^startgg-intro]

**Cannot:** GC lobbies, Steam-as-required-identity, AFK Week lifecycle. start.gg is the SoR.

### Toornament

A tournament is players **or** teams; a team participant has a **lineup** of players with identification properties (user id, email, custom user identifier, custom fields — e.g. a game user id).[^toornament-participant] Stage type `double_elimination` is first-class (winners’ bracket, losers’ bracket, grand final `none` / `simple` / `double`).[^toornament-stage] Platform API has Teams and Team members that persist across tournaments.[^toornament-start]

The **Game API** — the only documented “workflow between the game and tournaments” — has two sections: League of Legends (tournament codes) and Counter-Strike: GO (round stats). **No Dota 2 section.**[^toornament-game] Final standings are **not** auto-computed from structure; the organiser validates ranks.[^toornament-tournament]

**Cannot:** Dota GC lobbies, automatic Dota result, Steam as required identity (only a custom field). Toornament is the SoR.

### Challengermode

Challengermode is a hosted competition platform (“tournaments, matchmaking, ladders… 200+ game titles”) with a Game Integration API aimed at **game developers** embedding Challengermode into their title.[^cm-home][^cm-gi] They host Dota 2 events advertised as “5 vs 5… Double Elimination”.[^cm-dota]

**Cannot (from public docs):** a Dota 2 GC lobby controller the project can run; standing AFK Teams as the SoR; self-host. Their Game Integration API is for publishers, not for third parties driving Valve’s GC.

## Game Coordinator clients and lobby bots

These are the only open-source pieces that actually speak Dota’s Game Coordinator. None of them are a Week.

### ValvePython `dota2` — the lobby primitive

Official docs for `dota2.features.lobby.Lobby`:[^dota2-lobby]

| AFK Lobby step | API |
|---|---|
| Create practice lobby | `create_practice_lobby(password, options)` |
| Create tournament lobby | `create_tournament_lobby(...)` (needs a Valve tournament id) |
| Invite Steam ID | `invite_to_lobby(steam_id)` |
| Seat Radiant/Dire | `join_practice_lobby_team(slot, team=DOTA_GC_TEAM)` ; `flip_lobby_teams()` ; `practice_lobby_kick_from_team(account_id)` |
| Launch | `launch_practice_lobby()` |
| Destroy / leave | `destroy_lobby()` / `leave_practice_lobby()` |
| Observe state | `EVENT_LOBBY_NEW` / `EVENT_LOBBY_CHANGED` → `CSODOTALobby` |

Match result is a **separate** feature: `request_match_details(match_id)` (rate limited to 100/day) returning `CMsgDOTAMatch`.[^dota2-match] The package is “a Python package for interacting with Dota 2 Game Coordinator”, based on `ValvePython/steam`.[^dota2-gh]

**Cannot:** Teams, Weeks, web SoR. It is a library. It requires a logged-in Steam account that plays Dota 2 as a bot (AFK map: “Provisioning the Steam account that opens Lobbies” is still unspecified). `create_tournament_lobby` is not a grassroots path — it wants Valve tournament identifiers.

### `Arcana/node-dota2` — same surface, archived

`createPracticeLobby`, `inviteToLobby(steam_id)`, `joinPracticeLobbyTeam(slot, team)`, `launchPracticeLobby`, `requestMatchDetails(match_id)`.[^nodedota2] **Archived 2022-06-11.** Maintainers point at `paralin/go-dota2`. They warn Valve can detect GC clients from traffic.[^nodedota2]

**Cannot:** be a maintained AFK dependency. Same product gap as the Python library.

### `paralin/go-dota2` — maintained GC client

“Go implementation of the DOTA2 game-coordinator client.” Lobby tracking / state management and “normal lobby operations” are marked complete. SOCache emits events for `Lobby`.[^godota2]

**Cannot:** Teams, Weeks, web UI. Library only.

### ScheduleBot (Dota Edition) — Discord inhouse, not a Week

Ticket starting point. Discord bot + Steam bot. “A Dota bot will automatically create a lobby and invite every player who have confirmed their attendance.”[^schedulebot-readme] Site: at event time it creates the lobby, invites confirmed users, starts when ten have joined (or `force-lobby-start`). Steam link is **add-the-bot-as-friend + Discord code**, not OpenID.[^schedulebot-site] Usage: `add-inhouse` requires event `--limit` ≥ 10; `--nobalance` can disable auto team balance; **one inhouse at a time** (simultaneous lobbies need multiple bot instances).[^schedulebot-usage] Depends on npm `dota2` (node-dota2) and `steam`.[^schedulebot-pkg] License Apache-2.0.

**Cannot:** standing Teams, Captain acting for a roster, double-elim Week, result detection (not in README/usage), web SoR, more than one Lobby per process. It is the generic captain-scheduled inhouse AFK’s glossary tells you to avoid.

### Dota2-EU-Ladder — inhouse league, not standing Teams

“A full-featured inhouse lobby system for Dota2 leagues.” Discord bot for signup/queue; Dota bot “automated lobbies that can invite players once queue is full, check for correct teams setup, record game results”; optional Django stats site; team balancer from preferred roles.[^eu-ladder] Powered by `ValvePython/dota2`. Used by RD2L / Clarity / Doghouse communities — as an **inhouse queue**, which matches how RD2L describes itself (individual signups, teams built via draft).[^rd2l]

**Cannot:** standing 5-stacks that register once and enter a Week; double-elim Week; AFK “captains do not create / series creates Lobby for a scheduled Team-vs-Team Match”. Closest open-source **lobby + result + website** combo, wrong competition object.

### Dota2SentinelBot — leaver-ban lobby host

“Automatically host lobbies and track the match results afterwards.” Goal is global leaver bans, not tournaments. Built on the Steam network. Can host custom-game lobbies that have Valve dedicated servers.[^sentinel]

**Cannot:** Teams, Weeks, web SoR, Radiant/Dire seating of two standing rosters.

### Dota2LobbyBot — MySQL-triggered lobby worker

“Creating lobbies with invitation users and match starting.” A third-party app inserts rows; a node process watches MySQL binlog and uses `node-dota2` to create the lobby and invite. Multiple bot accounts supported.[^lobbybot]

**Cannot:** any tournament/Team/Week logic (that is the “third-party application”). Depends on archived `node-dota2`.

## Identity and result — Valve’s official web surface

These are not products. They are the official way to do two AFK primitives without reverse-engineering the GC.

**Steam identity.** Steam is an OpenID provider. Use `https://steamcommunity.com/openid`. The Claimed ID is `https://steamcommunity.com/openid/id/<steamid>` (64-bit SteamID). Valve forbids collecting Steam username/password on your site.[^steam-openid] That is the correct AFK Player link (not ScheduleBot’s friend-code).

**Match result.** `GET https://api.steampowered.com/IDOTA2Match_<ID>/GetMatchDetails/v1?match_id=…` returns `radiant_win`, per-player `account_id` / `player_slot` (bit 0 = Dire), `lobby_type` (`1` = Practice), `leagueid`, `game_mode`.[^getmatchdetails] Practice-lobby games are visible here once you have the `match_id` (from the GC lobby object after launch, or from a player’s match history).

Together with a GC lobby client, this is enough to implement AFK’s Lobby loop without FACEIT/Rivals. It is not a Week.

## Community leagues that look like AFK but are not software

**RD2L** (`rd2l.gg`): “Individual signups; Teams built via draft; 8 week BO2 regular season.” Features include inhouses and weekend tourneys.[^rd2l] That is the opposite Team model (drafted season stacks, not standing 5-stacks that register once). Their site is an operator, not a reusable product.

No other grassroots Dota league inspected (via first-party pages) ships a public, adoptable “standing team + weekly double-elim + GC lobby” system.

## What nobody ships

Across every candidate:

1. **Standing Team as AFK defines it** — register once, Captain + ≥5 Steam-linked Players, enter Weeks — is not the object in any open-source bot. FACEIT premade teams are the commercial analogue, on FACEIT’s account graph.
2. **A Week that runs itself** — double-elim through to a winner with **series-created** Lobbies and **no organiser sitting each lobby** — is not a single open-source product. FACEIT championships + Game Integration is the commercial analogue, on FACEIT.
3. **Web app as SoR** plus GC automation. Discord bots that open lobbies keep Discord (or a side Postgres) as SoR. Bracket SaaS keep themselves as SoR and never open a Dota lobby. FACEIT/Rivals keep themselves as SoR.
4. **AFK’s locked Week rules** (top 2 auto-enter next Week; opt-in re-entry; disband only after two losses) appear in no vendor doc.
5. **Filipino grassroots** is a domain, not a feature any of these encode.

## Implication for adopt-or-build

The map’s destination includes “a locked adopt-or-build decision if existing software already covers that.” Existing software **does not cover AFK v1**.

Reasonable reuse, not adoption:

| Piece | Reuse |
|---|---|
| Double-elim pairing | Challonge / Toornament / start.gg as a *bracket engine*, or implement locally. None of them should be SoR. |
| Steam identity | Steam OpenID (official). |
| Lobby create / invite / seat / launch | `ValvePython/dota2` or `paralin/go-dota2` behind AFK’s own Lobby worker. Do not start from archived `node-dota2` or ScheduleBot. |
| Result | GC `match_id` + `IDOTA2Match/GetMatchDetails` (`radiant_win`) or GC `request_match_details`. |
| Inhouse-bot patterns | Dota2-EU-Ladder / ScheduleBot as *references* for invite-and-launch sequencing and the “one Steam bot account per concurrent Lobby” constraint. |

Build the web app. Treat FACEIT as proof that Valve’s GC will accept platform-created practice lobbies and roster invites. Treat Rivals as proof a Discord-first commercial bot is trying to own the same Dota loop — and as a reason not to become Discord-first if the web app is the SoR.

## Sources

[^faceit-data]: FACEIT for Developers, Data API v4 — Teams, Championships, Matches, Players. <https://docs.faceit.com/docs/data-api/data/>
[^faceit-cup-rules]: FACEIT, “Dota 2 5on5 Community Cup Europe” rules: automatic lobby + invite via “Faceit Dota 2 Game Integration system”. <https://www.faceit.com/en/championship/72cdf1b7-a298-4c08-96ad-92095f629562/Dota%202%205on5%20Community%20Cup%20Europe/rules>
[^faceit-ewc-rules]: FACEIT, “RES Unchained 2025” / EWC-class Dota 2 qualifier rules: “A lobby will be generated automatically through the Faceit Dota 2 Game Integration system. Each team member will receive an invitation to join this lobby.” <https://www.faceit.com/en/championship/c5ab6d4b-253a-439a-8059-48359d35b228//rules>
[^faceit-de]: FACEIT Support, “Tournament formats: Single and Double elimination”. <https://support.faceit.com/hc/en-us/articles/17048704436508-Tournament-formats-Single-and-Double-elimination>
[^faceit-ewc-de]: FACEIT, EWC Dota 2 Online Qualifier Stage 1 rules: “Tournament System: Double Elimination Bracket.” <https://www.faceit.com/championship/8106016d-669f-4635-8088-8f7d489d6b75//rules>
[^rivals-org]: Rivals, organiser product page. <https://getrivals.com/organisers>
[^rivals-players]: Rivals, player product page. <https://getrivals.com/players>
[^rivals-dev]: Rivals, developer / game-integration page (Dota 2 live). <https://getrivals.com/developers>
[^tourneybot]: Tourney Bot. <https://tourneybot.gg/>
[^challonge-create]: Challonge API v2.1, Create Tournament (`tournament_type` includes `double elimination`; `starts_at` is not an auto-start). <https://challonge.apidog.io/create-tournament-23619740e0>
[^challonge-gi]: Challonge, Game Integration (beta) — participants and score reporting. <https://challonge.apidog.io/game-integration-beta-1726722m0>
[^startgg-glossary]: start.gg Developer Portal, Glossary (Entrant = Team or Player). <https://developer.start.gg/docs/glossary/>
[^startgg-de]: start.gg blog, “Intro to Tournaments & Basic Terminology” — Double Elimination. <https://blog.start.gg/intro-to-tournaments-basic-terminology-442af2d58ee6>
[^startgg-dota]: start.gg, “the Master Cup - Dota 2 Tournament”. <https://www.start.gg/tournament/master-cup-dota-2-1/details>
[^startgg-intro]: start.gg Developer Portal, Intro. <https://developer.start.gg/docs/intro/>
[^toornament-participant]: Toornament API v2, Participant (team lineup, custom user identifier). <https://developer.toornament.com/v2/core-concepts/participant/>
[^toornament-stage]: Toornament API v2, Stages — type `double_elimination`. <https://developer.toornament.com/v2/core-concepts/structure/stage>
[^toornament-start]: Toornament API v2, Getting started (Platform API: Teams, Team members). <https://developer.toornament.com/>
[^toornament-game]: Toornament API v2, Game API overview — League of Legends and Counter-Strike: GO only. <https://developer.toornament.com/v2/doc/game_overview>
[^toornament-tournament]: Toornament API v2, Tournament — final standing is organiser-validated. <https://developer.toornament.com/v2/core-concepts/tournament/>
[^cm-home]: Challengermode home. <https://www.challengermode.com/>
[^cm-gi]: Challengermode, Game Integration API. <https://www.challengermode.com/developers/docs/game-integration-api/using-the-api?lang=en>
[^cm-dota]: Challengermode, example Dota 2 5v5 double-elimination tournament page. <https://www.challengermode.com/s/NGA/tournaments/750a041f-4fab-4907-ed2e-08dcd650897e?lang=en>
[^dota2-lobby]: ValvePython `dota2` docs, `dota2.features.lobby`. <https://dota2.readthedocs.io/en/stable/dota2.features.lobby.html>
[^dota2-match]: ValvePython `dota2` docs, `dota2.features.match` (`request_match_details`, 100/day). <https://dota2.readthedocs.io/en/stable/dota2.features.match.html>
[^dota2-gh]: ValvePython/dota2 README. <https://github.com/ValvePython/dota2>
[^nodedota2]: Arcana/node-dota2 README (archived 2022-06-11; lobby + `requestMatchDetails`). <https://github.com/Arcana/node-dota2>
[^godota2]: paralin/go-dota2 README. <https://github.com/paralin/go-dota2>
[^schedulebot-readme]: MeLlamoPablo/schedulebot `dota` branch README. <https://github.com/MeLlamoPablo/schedulebot/blob/dota/README.md>
[^schedulebot-site]: ScheduleBot for Dota 2 (project site). <https://mellamopablo.github.io/schedulebot/>
[^schedulebot-usage]: ScheduleBot usage guide (`link-steam`, `add-inhouse`, `force-lobby-start`; one lobby). <https://github.com/MeLlamoPablo/schedulebot/blob/dota/usage/usage-guide.md>
[^schedulebot-pkg]: ScheduleBot `package.json` (depends on `dota2`, `steam`). <https://github.com/MeLlamoPablo/schedulebot/blob/dota/package.json>
[^eu-ladder]: UncleVasya/Dota2-EU-Ladder README. <https://github.com/UncleVasya/Dota2-EU-Ladder>
[^sentinel]: ErkoKnoll/Dota2SentinelBot README. <https://github.com/ErkoKnoll/Dota2SentinelBot>
[^lobbybot]: sushkov/Dota2LobbyBot README. <https://github.com/sushkov/Dota2LobbyBot>
[^steam-openid]: Steam Community, Steam Web API Documentation — OpenID provider. <https://steamcommunity.com/dev>
[^getmatchdetails]: Valve WebAPI `IDOTA2Match/GetMatchDetails` (Team Fortress Wiki mirror of the official method). <https://wiki.teamfortress.com/wiki/WebAPI/GetMatchDetails>
[^rd2l]: RD2L: R Dota 2 League. <https://rd2l.gg/>
