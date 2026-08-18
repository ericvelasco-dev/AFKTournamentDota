# Can a Game Coordinator lobby bot still run a Match in 2026?

**Ticket:** [#3](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/3)
**Researched:** 2026-08-19
**Live GC login:** not performed. Findings are from first-party source (Valve/Steam terms and Web API), library source, protobufs still shipped in 2026, and GitHub issue comments from people who ran bots.

## Verdict

**Yes, unofficially, with a dedicated Steam account and no Captain host — if the bot speaks the current Game Coordinator (GC) protocol.**

A GC client can still: create a practice Lobby, invite 64-bit Steam IDs, launch the Match, and read the winner from the lobby shared object or the official Steam Web API. That is the protocol Valve's own Dota 2 client still uses; it is not a documented public API.

What is *not* first-class:

- Forcing other Players onto Radiant or Dire. The seating message has no account ID.
- Using stock ValvePython `dota2` 1.1.0 as a production client. It last shipped in 2021; its protobufs are stale.
- Any Valve blessing. The Steam Subscriber Agreement forbids protocol emulation and automation. A dedicated account can be restricted or banned without notice.

AFK v1 can automate Lobbies. It cannot do so on a supported, ToS-clean path.

## What the question needs

AFK v1 requires the series — not a Captain — to open each Lobby, invite the two Teams, seat Radiant and Dire, launch, and detect the winner unattended ([CONTEXT.md](../../CONTEXT.md)). There is no official "create a practice lobby" Web API. The only known way is to log a Steam account into Dota 2's Game Coordinator and send the same protobufs the official client sends.

## Libraries and who still maintains them

| Client | Role | Last release / last meaningful push | Status |
| --- | --- | --- | --- |
| [ValvePython/dota2](https://github.com/ValvePython/dota2) | Python Dota 2 GC client | PyPI **1.1.0**, 2021-05-02; last commit `6ccebc3689` 2021-05-02 ("bump to v1.1.0") | Unarchived, but not developed. 32 open issues. Member: "Only basic maintenance, it is not being actively developed." ([#88](https://github.com/ValvePython/dota2/issues/88#issuecomment-1173054244), 2022-07-03). Contributor: protos "not synced with retail"; depends on stale `steam` ([#88](https://github.com/ValvePython/dota2/issues/88#issuecomment-1833659857), 2023-11-30). |
| [ValvePython/steam](https://github.com/ValvePython/steam) | Python Steam CM client (login, GC transport) | PyPI **1.4.4**, 2022-12-10; last code 2023-05-05 | Open login/session/GC bugs through 2024 ([#450](https://github.com/ValvePython/steam/issues/450), [#451](https://github.com/ValvePython/steam/issues/451), [#458](https://github.com/ValvePython/steam/issues/458)). |
| [paralin/go-dota2](https://github.com/paralin/go-dota2) | Go Dota 2 GC client | Protobuf refresh 2026-07-11; dep bump 2026-08-04 | **The only GC lobby client still tracking retail protos in 2026.** Lobby ops are implemented (`CreateLobby`, `InviteLobbyMember`, `JoinLobbyTeam`, `LaunchLobby`). |
| [paralin/go-steam](https://github.com/paralin/go-steam) | Go Steam protocol | Pushed 2026-08-13 | Maintained transport for go-dota2. |
| [Arcana/node-dota2](https://github.com/Arcana/node-dota2) | Node GC plugin | Last push 2022-06-08 | **Archived.** ValvePython/dota2's README still points at it as the sibling API. |
| [DoctorMcKay/node-steam-user](https://github.com/DoctorMcKay/node-steam-user) | Node Steam client protocol | Pushed 2025-12-04 | Maintained Steam login/session layer. Not a Dota lobby bot by itself. |
| [SteamRE/SteamKit](https://github.com/SteamRE/SteamKit) | .NET Steam protocol | Pushed 2026-08-01 | Maintained. No first-party Dota lobby helper. |
| [SteamDatabase/GameTracking-Dota2](https://github.com/SteamDatabase/GameTracking-Dota2) | Extracted retail files / protos | Pushed 2026-08-18 | Live dump of what the official client currently ships. |

**Python forks of ValvePython/dota2** listed by GitHub have 0 stars. The newest (`bojowadynia/dota2`, pushed 2024-10-12; `Aluerie/dota2`, 2024-01-18) are not a maintained replacement.

**Implication for AFK:** do not ship on `pip install dota2==1.1.0`. If AFK uses a GC bot, the client must track current protobufs — today that means go-dota2, or a fork that regenerates from GameTracking-Dota2.

## Capability map

### 1. Create a practice Lobby (no Captain)

**Works at the protocol layer.** The official client still sends `k_EMsgGCPracticeLobbyCreate` / `CMsgPracticeLobbyCreate` with a `CMsgPracticeLobbySetDetails` payload (password, region, game mode, league id, pause, TV delay, visibility). That message is in go-dota2's 2026 proto tree ([`dota_gcmessages_client_match_management.proto`](https://github.com/paralin/go-dota2/blob/2f03ac628187a26dc732f52a42ad9bd061d8abc3/protocol/dota_gcmessages_client_match_management.proto)).

ValvePython wraps it as `create_practice_lobby(password, options)`, which just calls `create_tournament_lobby` and sends `EDOTAGCMsg.EMsgGCPracticeLobbyCreate` ([`dota2/features/lobby.py`](https://github.com/ValvePython/dota2/blob/6ccebc3689e107746ec32ce07fc2f5cacecc0e18/dota2/features/lobby.py)). go-dota2 wraps the same message as `CreateLobby` ([`client_lobby.go`](https://github.com/paralin/go-dota2/blob/2f03ac628187a26dc732f52a42ad9bd061d8abc3/client_lobby.go)).

There is **no** matching method on the official Steam Web API. [steamcommunity.com/dev](https://steamcommunity.com/dev) lists news, user, and item interfaces. The Dota match interfaces documented on the Team Fortress Wiki (`IDOTA2Match_570`) are **read** APIs: history, details, live league games — not lobby create ([WebAPI](https://wiki.teamfortress.com/wiki/WebAPI), last edited 2026-02-01).

`create_tournament_lobby` / `leagueid` is a different product: a Valve league ticket lobby. The library author called the tournament helper buggy ([#52](https://github.com/ValvePython/dota2/issues/52#issuecomment-637007484)). A grassroots Week should use a passworded **practice** Lobby, not a league lobby, unless AFK later obtains a Valve league id.

### 2. Invite specific Steam IDs

**Works at the protocol layer, host-only.** `CMsgInviteToLobby { steam_id }` / `k_EMsgGCInviteToLobby = 4512` is still in go-dota2's base GC messages ([`base_gcmessages.proto`](https://github.com/paralin/go-dota2/blob/2f03ac628187a26dc732f52a42ad9bd061d8abc3/protocol/base_gcmessages.proto)). ValvePython: `invite_to_lobby(steam_id)` ([`lobby.py`](https://github.com/ValvePython/dota2/blob/6ccebc3689e107746ec32ce07fc2f5cacecc0e18/dota2/features/lobby.py)). go-dota2: `InviteLobbyMember(playerID)` ([`client_lobby_members.go`](https://github.com/paralin/go-dota2/blob/2f03ac628187a26dc732f52a42ad9bd061d8abc3/client_lobby_members.go)).

The invitee must run the official Dota 2 client and accept. The bot cannot force-join another account.

### 3. Seat Radiant and Dire

**Not a first-class "put Steam ID X on Radiant slot Y" call.**

`CMsgPracticeLobbySetTeamSlot` has only `team`, `slot`, and `bot_difficulty` — no `account_id` ([2026 proto](https://github.com/paralin/go-dota2/blob/2f03ac628187a26dc732f52a42ad9bd061d8abc3/protocol/dota_gcmessages_client_match_management.proto)). Both libraries use it for **the bot's own seat** (`join_practice_lobby_team` / `JoinLobbyTeam`) or to drop a **bot difficulty** into a slot.

What the host *can* do to other accounts:

- `CMsgPracticeLobbyKick { account_id }` — remove from the Lobby.
- `CMsgPracticeLobbyKickFromTeam { account_id }` — dump them back to unassigned.

`CMsgApplyTeamToPracticeLobby { team_id }` applies a registered in-game Dota **team** to the Lobby, not ten individual Steam IDs.

ValvePython issue ["Q: A way to assign players to teams before match start?"](https://github.com/ValvePython/dota2/issues/102) (2024-12-31) is unanswered.

**Practical seating for AFK:** invite → Players pick Radiant/Dire in the official client → bot watches `CSODOTALobby` members → kick-from-team (or kick) anyone on the wrong side → they pick again. The bot itself must **not** sit in a player slot (see launch).

### 4. Launch the Match

**Works at the protocol layer, with a hard constraint: the bot never joins the game server.**

`CMsgPracticeLobbyLaunch` / `k_EMsgGCPracticeLobbyLaunch` is still generated in go-dota2 as `LaunchLobby()` ([`client_generated.go`](https://github.com/paralin/go-dota2/blob/2f03ac628187a26dc732f52a42ad9bd061d8abc3/client_generated.go)). ValvePython: `launch_practice_lobby()` sends the same message if `lobby.leader_id == steam.steam_id`.

rossengeorgiev (ValvePython member), 2021-10-02: "This package only talks to the Dota 2 game coordinator. It has no way of joining games." Bots that need to stay attached sit spectator/broadcaster so they are not required to connect, and "will retain access to the match and match details" ([#75](https://github.com/ValvePython/dota2/issues/75#issuecomment-932751547)).

Same member, 2022-07-03: "None of these module will let you connect the game, only create and manage lobbies" ([#88](https://github.com/ValvePython/dota2/issues/88#issuecomment-1173054244)).

A reporter who sat the bot in **player pool** (unassigned) could launch; the bot then disconnects from the would-be game but still sees lobby updates through to post-game ([#88](https://github.com/ValvePython/dota2/issues/88#issuecomment-1173055219)). Sitting **broadcaster** historically blocked launch because the bot never connected to the server. A 2023 howto sat the bot via `join_practice_lobby_team()` (default `PLAYER_POOL`) then `launch_practice_lobby()` ([#81](https://github.com/ValvePython/dota2/issues/81#issuecomment-1453587078)).

`all_members` on `CSODOTALobby` is the documented way to wait until both Teams are in before launch ([#68](https://github.com/ValvePython/dota2/issues/68#issuecomment-877935966)).

League-id lobbies keep running if humans leave because the GC still counts the bot as a spectator ([#87](https://github.com/ValvePython/dota2/issues/87#issuecomment-1453506839)). A passworded practice Lobby without a league id does not have that property; a full disconnect can end the Match.

### 5. Detect the winner — without Dotabuff

**Yes. Three official-or-GC paths; none need Dotabuff or OpenDota.**

#### A. Lobby shared object (same session, no extra API)

`CSODOTALobby` carries `optional EMatchOutcome match_outcome = 70`. The 2026 enum is ([`dota_shared_enums.proto`](https://github.com/paralin/go-dota2/blob/2f03ac628187a26dc732f52a42ad9bd061d8abc3/protocol/dota_shared_enums.proto)):

| Value | Name |
| --- | --- |
| 0 | `k_EMatchOutcome_Unknown` |
| 2 | `k_EMatchOutcome_RadVictory` |
| 3 | `k_EMatchOutcome_DireVictory` |
| 4 | `k_EMatchOutcome_NeutralVictory` |
| 5 | `k_EMatchOutcome_NoTeamWinner` |

On `lobby_changed`, `message.state == POSTGAME` (callers treat `3` as post-game) and `match_outcome` is 2 or 3 ([#67](https://github.com/ValvePython/dota2/issues/67#issuecomment-872681568), [#90](https://github.com/ValvePython/dota2/issues/90#issuecomment-1453395692)). The GC then emits `EMsgGCToClientMatchSignedOut` (8081) and removes the lobby SOCache ([#88](https://github.com/ValvePython/dota2/issues/88#issuecomment-1173062456)).

A 2024-05-27 comment on #90 still talks about reading that same `match_outcome` field — evidence someone was still consuming lobby post-game in 2024, not that the Python wheel is healthy.

#### B. GC match-details request (same bot account)

`request_match_details(match_id)` sends `EMsgGCMatchDetailsRequest`. The library documents a **100 requests/day** cap ([`dota2/features/match.py`](https://github.com/ValvePython/dota2/blob/6ccebc3689e107746ec32ce07fc2f5cacecc0e18/dota2/features/match.py)). The maintainer: you do **not** need the bot inside the running game; "Just get the match details after the match" ([#67](https://github.com/ValvePython/dota2/issues/67#issuecomment-872122310)).

#### C. Official Steam Web API (no GC, no Dotabuff)

`GET https://api.steampowered.com/IDOTA2Match_570/GetMatchDetails/v1?key=…&match_id=…` returns `radiant_win` (true = Radiant) plus duration, players, `lobby_type` ([WebAPI/GetMatchDetails](https://wiki.teamfortress.com/wiki/WebAPI/GetMatchDetails)). `GetMatchHistory` documents `lobby_type` **1 = Practise** ([WebAPI/GetMatchHistory](https://wiki.teamfortress.com/wiki/WebAPI/GetMatchHistory)). App id 570 is Dota 2 ([WebAPI § List of IDs](https://wiki.teamfortress.com/wiki/WebAPI)).

Requires a Steam Web API key ([steamcommunity.com/dev](https://steamcommunity.com/dev)). Terms: 100,000 calls/day, key is personal, must not violate the SSA, Valve may revoke at will ([Steam Web API Terms of Use](https://steamcommunity.com/dev/apiterms), last updated July 2010).

`GetMatchHistory` is a **list** filtered by account / league / hero. A private practice Lobby may not appear in a Player's public history; **GetMatchDetails by `match_id`** is the reliable official read if the Lobby produced a match id. The GC lobby object is the reliable *live* read.

Game State Integration is a **local official-client** HTTP push (Dota launched with `-gamestateintegration`). It is not a GC-bot interface and is not required if POSTGAME `match_outcome` is enough.

## What is broken or unofficial

The whole GC bot is unofficial. Valve never published these messages as a developer API. Libraries reverse-engineer the same protobufs the official client ships ([SteamDatabase/GameTracking-Dota2](https://github.com/SteamDatabase/GameTracking-Dota2), go-dota2 proto generator).

Concrete breakage on the Python stack:

- **Stale protobufs.** Last ValvePython proto bump was 2021-05-02 (`update protobufs`). 2022 logs already show `ERROR Dota2Client.socache: Unsupported type: 2013/2014/2015/2016` ([#88](https://github.com/ValvePython/dota2/issues/88#issuecomment-1173055219)). 2025-07-02: "SOCache object protobuf definitions in the module are out of date" ([#98](https://github.com/ValvePython/dota2/issues/98#issuecomment-3026313573)).
- **Stale Steam login.** `steam` 1.4.4 is from 2022. Open tickets: session broken after a Steam update ([#450](https://github.com/ValvePython/steam/issues/450), 2023-10), cannot log in ([#451](https://github.com/ValvePython/steam/issues/451)), cannot connect to a GC ([#458](https://github.com/ValvePython/steam/issues/458), 2024-02, pointing at a SteamDatabase protobuf change).
- **Sessions die in datacenters.** 2025-05-28: a DigitalOcean-hosted bot "don't keep the session alive. After 5 or 10 seconds, the session closes" ([#103](https://github.com/ValvePython/dota2/issues/103)). Unanswered beyond "are you calling `run_forever()`?"
- **No maintainer.** #88 (status), #90 (match result), #98 (unsupported type), #102 (seat players), #103 (session) are all still **open**.
- **People still try.** Issues filed 2024-11, 2024-12, 2025-05, 2025-11. That is demand, not a passing integration test.

go-dota2 is the opposite: proto update 2026-07-11, README marks lobby create/read/ops complete. It is still unofficial.

## Dedicated-account and ban risk

Valve's current Steam Subscriber Agreement (revision date **2026-04-20**, [store.steampowered.com/subscriber_agreement](https://store.steampowered.com/subscriber_agreement/)) is the owning source.

| Clause | What it says | Why a lobby bot hits it |
| --- | --- | --- |
| **1.C Account** | Login is personal. Do not share the account. You are responsible for activity on it. | The bot *is* the account. A dedicated host account is still a Steam account under these rules. |
| **2.A License** | Content and Services licensed for "personal, non-commercial use" except where Valve expressly allows commercial use. | A weekly public competition is not personal play. |
| **2.G Restrictions** | No reverse engineering. You may not "host or provide matchmaking services" or "emulate or redirect the communication protocols used by Valve … through protocol emulation, tunneling, modifying or adding components … without the prior written consent of Valve." | A GC bot *is* protocol emulation used to host Matches. |
| **4.C Automation** | "You may not use any form of scripts, bots, macros, or other non-human-controlled systems ('Automation') to interact with Content and Services on Steam in any manner," including faking stats and earning rewards without genuine input. | The host is a non-human-controlled GC session. |
| **4.D Enforcement** | Valve may restrict or terminate the Account or a Subscription for Automation "and is not required to provide you notice." | Dedicated account does not remove the risk; it only isolates it from a human Player. |

Steam Online Conduct (incorporated via SSA 1.B / 4.A, [store.steampowered.com/online_conduct](https://store.steampowered.com/online_conduct/)) separately forbids automatically generating Steam accounts and "engage in commercial activity."

The Steam Web API Terms ([steamcommunity.com/dev/apiterms](https://steamcommunity.com/dev/apiterms)) add: you may not use the Web API "in any way that violates the Steam Subscriber Agreement"; 100k calls/day; Valve may terminate access without notice.

No Valve document carves out "inhouse lobby host" or "tournament bot." This research did **not** find a first-party write-up of a specific lobby-bot ban wave. The risk is the contract, not an anecdote: Valve can drop the host account at any time, and 2.G is a better fit for a GC bot than the "cheat" examples in 4.B.

**Operational hygiene if AFK still takes this path:** a dedicated, empty Steam account that owns Dota 2; never a Player's main; accept total loss of that account; do not automate account *creation*; do not farm MMR/rewards on it; keep Web API use on GetMatchDetails only.

## What still works vs what does not (2026)

| Step | Protocol still exists (2026 protos) | Stock ValvePython 1.1.0 | go-dota2 (2026) | Official / ToS-clean |
| --- | --- | --- | --- | --- |
| Create practice Lobby | Yes (`EMsgGCPracticeLobbyCreate`) | Implemented; protos stale | Implemented, protos current | No public API |
| Invite Steam IDs | Yes (`EMsgGCInviteToLobby`) | Implemented | Implemented | No |
| Force-seat other Players | **No such message** | Only self-seat + kick | Same | Players pick in the official client |
| Launch | Yes (`EMsgGCPracticeLobbyLaunch`) | Implemented; bot must not need a game-server connect | Implemented | No |
| Winner without Dotabuff | Yes (`CSODOTALobby.match_outcome`; Web API `radiant_win`) | Documented by users 2021–2024 | Subscribe `cso.Lobby` | **GetMatchDetails is official** once you have `match_id` |
| Unattended Week | n/a | Fragile (login, protos, session) | Best unofficial client | Forbidden by SSA 2.G / 4.C |

## Implications for AFK

1. **The product requirement is technically possible** without a Captain-created Lobby.
2. **Treat the host Steam account as disposable infrastructure**, not as a supported integration.
3. **Do not build on ValvePython `dota2` 1.1.0.** Use a client that regenerates from current protos (go-dota2 today).
4. **Seating is social, not an RPC.** Invite, watch `CSODOTALobby` members, kick-from-team on the wrong side.
5. **Read the winner from GC POSTGAME `match_outcome`.** Keep `IDOTA2Match_570/GetMatchDetails` as the official backup. Do not take a runtime dependency on Dotabuff or OpenDota.
6. **There is no Valve-supported substitute** that creates Lobbies. League Web API methods are for Valve-listed leagues, not grassroots Weeks.

## Sources

- ValvePython/dota2 repo metadata, commits, tag `v1.1.0`, [`dota2/features/lobby.py`](https://github.com/ValvePython/dota2/blob/6ccebc3689e107746ec32ce07fc2f5cacecc0e18/dota2/features/lobby.py), [`dota2/features/match.py`](https://github.com/ValvePython/dota2/blob/6ccebc3689e107746ec32ce07fc2f5cacecc0e18/dota2/features/match.py); docs: [dota2.readthedocs.io lobby](https://dota2.readthedocs.io/en/stable/dota2.features.lobby.html), [user guide](https://dota2.readthedocs.io/en/stable/user_guide.html); PyPI [dota2 1.1.0](https://pypi.org/project/dota2/) (uploaded 2021-05-02).
- ValvePython/dota2 issues: [#52](https://github.com/ValvePython/dota2/issues/52), [#67](https://github.com/ValvePython/dota2/issues/67), [#68](https://github.com/ValvePython/dota2/issues/68), [#75](https://github.com/ValvePython/dota2/issues/75), [#81](https://github.com/ValvePython/dota2/issues/81), [#87](https://github.com/ValvePython/dota2/issues/87), [#88](https://github.com/ValvePython/dota2/issues/88), [#90](https://github.com/ValvePython/dota2/issues/90), [#98](https://github.com/ValvePython/dota2/issues/98), [#102](https://github.com/ValvePython/dota2/issues/102), [#103](https://github.com/ValvePython/dota2/issues/103).
- ValvePython/steam: repo metadata, PyPI [steam 1.4.4](https://pypi.org/project/steam/) (uploaded 2022-12-10); issues [#450](https://github.com/ValvePython/steam/issues/450), [#451](https://github.com/ValvePython/steam/issues/451), [#458](https://github.com/ValvePython/steam/issues/458).
- [paralin/go-dota2](https://github.com/paralin/go-dota2) README, `client_lobby.go`, `client_lobby_slot.go`, `client_lobby_members.go`, `client_generated.go`, protocol files at `2f03ac6281` (2026-08-04).
- [paralin/go-steam](https://github.com/paralin/go-steam), [SteamRE/SteamKit](https://github.com/SteamRE/SteamKit), [DoctorMcKay/node-steam-user](https://github.com/DoctorMcKay/node-steam-user), [Arcana/node-dota2](https://github.com/Arcana/node-dota2) (archived), [SteamDatabase/GameTracking-Dota2](https://github.com/SteamDatabase/GameTracking-Dota2).
- [Steam Subscriber Agreement](https://store.steampowered.com/subscriber_agreement/) (updated 2026-04-20), §§ 1.C, 2.A, 2.G, 4.C, 4.D.
- [Steam Online Conduct](https://store.steampowered.com/online_conduct/).
- [Steam Web API docs](https://steamcommunity.com/dev), [Steam Web API Terms of Use](https://steamcommunity.com/dev/apiterms).
- Team Fortress Wiki (Valve Web API reference): [WebAPI](https://wiki.teamfortress.com/wiki/WebAPI), [GetMatchDetails](https://wiki.teamfortress.com/wiki/WebAPI/GetMatchDetails), [GetMatchHistory](https://wiki.teamfortress.com/wiki/WebAPI/GetMatchHistory).
