# Adopt-or-build insight: the two comments on the map vs the analysis we already have

**Map:** [AFK v1 — the way to a spec](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/1)
**Ticket this feeds:** [Adopt existing software, or build?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/7)
**Compared against:** [What existing software already runs Teams, a Week, and Game Coordinator Lobbies?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/2) ([findings](https://github.com/ericvelasco-dev/AFKTournamentDota/blob/research/existing-software/docs/research/existing-software.md)) and [Can a Game Coordinator lobby bot still run a Match in 2026?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/3) ([findings](https://github.com/ericvelasco-dev/AFKTournamentDota/blob/research/gc-lobby-bot/docs/research/gc-lobby-bot.md))
**Input:** two comments the owner posted on the map (2026-08-19): (1) links to ValvePython `lobby.py` and README; (2) a component-level adopt-or-build write-up.
**Researched:** 2026-08-20. Primary sources only for claims that were new or that conflict with closed research.

## Verdict

The two comments do **not** replace the closed research and do **not** let us drop the domain tickets.

They **do** change the shape of [Adopt existing software, or build?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/7): the whole-app question is already answered (build AFK; nothing existing is v1). What remains is **component-level** adopt-or-build.

Lock this if grilling agrees:

| Piece | Do | Do not |
| --- | --- | --- |
| Product / web app / Team / roster / Saturday clock / top-2 auto-reentry | **Build** | Adopt FACEIT, Rivals, or [UncleVasya/Dota2-EU-Ladder](https://github.com/UncleVasya/Dota2-EU-Ladder) |
| Game Coordinator Lobby worker | **Adopt [paralin/go-dota2](https://github.com/paralin/go-dota2)** (MIT; protobufs refreshed 2026-07-11; last push 2026-08-18) | Ship stock [ValvePython/dota2](https://github.com/ValvePython/dota2) 1.1.0 (last code 2021-05-02) or archived [Arcana/node-dota2](https://github.com/Arcana/node-dota2) |
| Field bracket math | **Adopt [Drarig29/brackets-manager.js](https://github.com/Drarig29/brackets-manager.js)** (MIT; v1.11.0 on 2026-05-17) as an engine **inside** the web app | Make Challonge / start.gg / Toornament the system of record |
| Player identity | **Steam OpenID** (already locked by [Does a private Steam profile still work for AFK identity?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/4)) | Spend another research ticket on it |

The comment's one serious error: calling ValvePython/dota2 "the most current" GC client. Closed research is right and the comment is wrong on that point. The rest of the comment is a useful reframe, not a new survey.

## The two comments, in one paragraph each

**Comment 1** is two source links: [`dota2/features/lobby.py`](https://github.com/ValvePython/dota2/blob/master/dota2/features/lobby.py) and the [ValvePython/dota2 README](https://github.com/ValvePython/dota2/blob/master/README.rst). Those files are the Python wrappers for `EMsgGCPracticeLobbyCreate`, invite, kick, and launch. They are already the primary source [Can a Game Coordinator lobby bot still run a Match in 2026?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/3) used. They do not add a new candidate. They also do not make ValvePython production-ready: last bump is still `v1.1.0` on 2021-05-02.

**Comment 2** is a component-level adopt-or-build note: nobody has built AFK's Team/roster/Saturday-Field rules; do not hand-roll GC protocol or double-elim math; adopt ValvePython (or Go/Node port) + [brackets-manager.js](https://github.com/Drarig29/brackets-manager.js) + Steam OpenID; treat [Dota2-EU-Ladder](https://github.com/UncleVasya/Dota2-EU-Ladder) as shape reference, not a fork; note unofficial protobuf churn.

## Point-by-point comparison

Legend: **keep** = already on the map, leave it; **add** = incorporate; **drop from the comment** = do not take that recommendation; **overlap** = same conclusion, no new ticket.

| # | Point in the two comments | What the map / closed research already says | Fit | What to do |
| --- | --- | --- | --- | --- |
| 1 | Whole app is bespoke; nobody built weekly grassroots Captains Mode with this roster lifecycle | [What existing software already runs Teams, a Week, and Game Coordinator Lobbies?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/2): nothing is AFK v1; FACEIT closest hosted; Rivals closest Discord bot; web app still has to be built | overlap | Keep. This is the whole-app half of [Adopt existing software, or build?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/7). Do not reopen FACEIT/Rivals as a product to become. |
| 2 | Adopt ValvePython/dota2 as "the most current and widely built on" GC module | [Can a Game Coordinator lobby bot still run a Match in 2026?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/3): stock `dota2` 1.1.0 last shipped 2021-05-02; maintainer said "not being actively developed"; protos not synced with retail; `steam` login bugs through 2024; **go-dota2 is the only GC lobby client still tracking retail protos in 2026** | **drop from the comment** | Do not adopt ValvePython as the production client. Keep the repo as a readable protocol reference (`create_practice_lobby` / invite / launch). Production worker: go-dota2. |
| 3 | Go port (`paralin/go-dota2`) and Node forks (`Arcana/node-dota2`, `e1ektr0/node-dota2`) — "pick by language, not by capability" | Same ticket: Arcana/node-dota2 is **archived** (last push 2022-06-08). Python protos are stale. go-dota2 last meaningful proto refresh 2026-07-11, dep bump 2026-08-04, repo push 2026-08-18. They wrap the same messages, but they are **not** equally shippable in 2026 | **drop from the comment** | Pick by **who still regenerates protobufs**, not by language preference. That currently forces a Go Lobby worker (or a thin Go sidecar). Language of the web app can stay separate. |
| 4 | `create_practice_lobby` vs `create_tournament_lobby` / Valve league admin; grassroots = practice; "worth a research ticket if we want official league status later" | Issue 3 already: grassroots Week = passworded **practice** Lobby; tournament/leagueid is a different product and the ValvePython helper was called buggy; no public create-lobby Web API | overlap | Do **not** add a research ticket now. Practice Lobby is locked enough for v1. Official Valve league status is out of this map (or fog for a later effort). |
| 5 | Adopt [brackets-manager.js](https://github.com/Drarig29/brackets-manager.js) for Field seeding and winner-determination | Issue 2 named Challonge / start.gg / Toornament as *bracket engines*, and warned none of them should be the system of record. It did **not** name this library | **add** | This is the one new library worth taking. MIT, 333 stars, v1.11.0 on 2026-05-17, storage-agnostic, `type: 'double_elimination'`, `grandFinal: 'simple' \| 'double'`, BYEs at creation, forfeit on update, companion [brackets-viewer.js](https://github.com/Drarig29/brackets-viewer.js). It keeps the web app as SoR (unlike Challonge). It does **not** decide seeding policy, leftover policy, or no-show rules — those tickets stay. |
| 6 | Steam OpenID + Web API; "don't spend a research ticket" | [Does a private Steam profile still work for AFK identity?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/4) already closed: login, SteamID, persona name, Lobby invite work on a private profile | overlap | No new ticket. [Must a Player prove they own Dota, or is a SteamID enough?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/13) is a **different** question (eligibility proof, not identity). |
| 7 | [UncleVasya/Dota2-EU-Ladder](https://github.com/UncleVasya/Dota2-EU-Ladder) is nearest complete prior art: Discord + Dota bot + stats site; wrong domain (solo-MMR inhouse); no declared license; read, don't fork | Issue 2 already scored it: closest open-source lobby + result + website combo, wrong competition object, not standing 5-stacks | overlap | Keep as a sequencing reference (invite when the Match is due, one Steam bot account per concurrent Lobby). Do not fork. GitHub still has **no SPDX license** (checked 2026-08-20); last push 2024-03-15. |
| 8 | Build bespoke: Team/roster/Captain state machine, Thursday lock, Saturday Field cadence, top-2 auto-reentry, orchestration that watches bracket state and fires the next Lobby | Destination + Notes on the map; ADRs [0001](../../docs/adr/0001-web-app-is-system-of-record.md) and [0002](../../docs/adr/0002-saturday-clock-not-booking.md) | overlap | This is exactly why the open grilling tickets exist. Libraries do not encode those rules. **Do not drop those tickets.** |
| 9 | Standing risk: GC is unofficial; Valve reshuffles protobufs; cost exists whether you adopt or build | Issue 3 + [Do we accept Valve SSA risk on a dedicated host Steam account?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/14). SSA 2.G (protocol emulation / matchmaking) and 4.C (automation); host account can be dropped without notice | overlap | Keep ticket 14. The comment restates the engineering risk; ticket 14 is the **go / redraw destination** decision. |
| 10 | FACEIT, Rivals, Challonge, ScheduleBot — **omitted** from the comment | Issue 2's coverage matrix | keep existing | Do not discard that survey. FACEIT is still the proof that Valve accepts platform-created practice Lobbies and roster invites. Rivals is still the reason Discord-first is a later map, not a shortcut. |

## Implementations and open-source we can actually use

### 1. Lobby worker — adopt go-dota2, not ValvePython

Protocol (still in 2026 retail protos, implemented in go-dota2): create practice Lobby, invite Steam IDs, launch, read `CSODOTALobby.match_outcome` or `IDOTA2Match/GetMatchDetails`.

What is **not** in any library, Python or Go:

- Force-seat another Steam ID onto Radiant/Dire (`CMsgPracticeLobbySetTeamSlot` has no `account_id`). That stays [How do Players land on Radiant and Dire if the bot cannot force-seat them?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/15).
- A supported Valve API. That stays [Do we accept Valve SSA risk on a dedicated host Steam account?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/14).
- Eight parallel Round-1 Lobbies from one Steam session. Existing inhouse bots are "one Lobby per process / account." That stays on the map's fog and on [Provision the Steam account that opens Lobbies](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/16).

ValvePython remains useful as **documentation of the message names**. It is not a 2026 runtime.

### 2. Field math — adopt brackets-manager.js inside the web app

Verified against the [README](https://github.com/Drarig29/brackets-manager.js), [getting started](https://drarig29.github.io/brackets-docs/getting-started/), and [seeding guide](https://drarig29.github.io/brackets-docs/user-guide/seeding/) (2026-08-20):

- `type: 'double_elimination'` and `settings.grandFinal: 'simple' | 'double'` exist. AFK still has to **choose** Grand Final rules on [How is a 16-Team Field paired?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/21).
- Elimination size **must be a power of two**. BYEs (`null`) are how you pad. `balanceByes` avoids BYE vs BYE. That **constrains** [How many Teams enter a Weekly Tournament, and what happens to leftovers?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/18) — it does not answer it. If Thursday lock is 20 Teams, the library will not run a 20-team elimination stage; we pad, queue, split Fields, or refuse.
- BYEs are for **creation**. Updating seeding after the stage has started treats `null` as TBD, not BYE. Structural byes are a Thursday-lock decision, not a Saturday patch.
- Forfeit is supported **on update**. That is an implementation hook for [What happens when a Team does not arrive on time for a Match?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/20), not a substitute for the wait-time / incomplete-roster **rule**.
- Isolated Fields: create **one stage per Field** (or one `tournamentId` per Field). The library will not stop us from putting 32 Teams in one double-elim; AFK's rule that Fields do not play each other is ours.
- Storage-agnostic: we persist in the web app. That matches [Is the web app the system of record, or Discord?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/5). Challonge/start.gg/Toornament would fight that ADR.

Do not treat this as "the pairing ticket is done." The library runs the draw we specify. Who is seed 1, how last week's auto-enters are placed, and who "top 2" of a Field are remain grilling.

### 3. Identity — already official

Steam OpenID Claimed ID = SteamID64. Valve forbids collecting username/password. No library search. [Must a Player prove they own Dota, or is a SteamID enough?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/13) is still open because owned-games is a **public-profile** fact, not an OpenID fact.

### 4. Whole-app prior art — reference only

| Repo / product | Use | Why not adopt |
| --- | --- | --- |
| FACEIT | Proof GC will accept platform-created Lobbies + roster invites | They are the SoR; closed; not Filipino grassroots AFK |
| Rivals | Proof a commercial bot tries to own auto-lobby + results | Discord-first; prizes; later map |
| Dota2-EU-Ladder | Invite-when-full, seat-check, record result, optional Django site | Pickup inhouse + balancer; no license; last push 2024-03 |
| ScheduleBot | One-at-a-time inhouse create/invite/launch | Pickup; no standing Team; archived `node-dota2` |
| Challonge / start.gg / Toornament | Double-elim as a service | They become SoR; no Dota Lobby |

## Challenges if we take the component-level path (the fast-track cost)

These are not reasons to reinvent the GC protocol or double-elim math. They are reasons the other tickets still exist.

1. **Valve can kill the host.** SSA 2.G and 4.C. Dedicated account isolates the blast, it does not remove it. Round 1 is eight Lobbies per Field — that is eight host sessions, and a ban wave is a Saturday outage. Ticket 14 is the destination-level gate.
2. **Protobuf churn on a cadence we do not control.** go-dota2 is current *today*. A Dota client patch can still break create/invite/launch on a Thursday. Someone has to regenerate protos. That is an ops cost in v1, not a later nice-to-have.
3. **Seating is social.** The bot invites; Players pick sides in the official client; kick-from-team is the only correction. Ten grassroots Players at 13:00 PHT will sit wrong. Ticket 15 is the UX rule; ticket 9 is what an Organizer may fix when they do not.
4. **One Steam account ≠ eight Lobbies.** Inhouse prior art is one Lobby per bot process. A 16-Team Field's Upper R1 is eight concurrent Lobbies; two Fields is sixteen. Host-account count hangs on ticket 14, then ticket 16.
5. **go-dota2 is Go; brackets-manager.js is JavaScript.** Fast-track implies a **split**: web app (JS/TS) owns Teams, Fields, and bracket state; a Go worker owns GC sessions. That is extra glue (queue, credentials, "next Match is due → fire Lobby"), not a reason to force Python onto stale protos.
6. **brackets-manager.js will happily encode a policy we have not locked.** If we plug it in before [How is a 16-Team Field paired?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/21) and leftover/no-show tickets, we will bake in `inner_outer` seeding, `grandFinal: 'simple'`, and forfeit-on-update as if they were AFK law. Adopt the engine; do not skip the rules.
7. **No license on EU-Ladder** means we cannot copy code even when the shape is right. Read it. Do not paste it.
8. **Result path still has an exception.** GC `match_outcome` and GetMatchDetails work when the Lobby produces a `match_id`. Abandoned games, GC drops, and "the bot lost the session" are [How does a finished Lobby become a Match result?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/8) and [What can an Organizer override when the Lobby fails?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/9).

## Ticket audit — keep, narrow, or do not add

Closed tickets stay closed. The comments do not reopen them.

| Ticket | After this insight | Why |
| --- | --- | --- |
| [AFK v1 — the way to a spec](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/1) | keep | Map. After ticket 7 closes, append a Decisions-so-far gist; do not paste this doc into the map. |
| [What existing software already runs Teams, a Week, and Game Coordinator Lobbies?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/2) | keep (closed) | Survey still stands. Comment 2 is a reframe of reuse, not a better survey. |
| [Can a Game Coordinator lobby bot still run a Match in 2026?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/3) | keep (closed) | Wins the conflict with comment 2 on ValvePython vs go-dota2. |
| [Does a private Steam profile still work for AFK identity?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/4) | keep (closed) | Comment 2 agrees; already done. |
| [Is the web app the system of record, or Discord?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/5) | keep (closed) | Comment 2 does not reopen Discord-first. brackets-manager.js *supports* keeping the web app as SoR. |
| [Is a Weekly Tournament a fixed Saturday-Sunday, or are Matches scheduled one by one?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/6) | keep (closed) | Cadence is ours; no library encodes 13:00 / 14:30 / 16:00 / 17:30 PHT. |
| [Adopt existing software, or build?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/7) | **narrow, then close when grilled** | Whole-app adopt is already "no." Remaining lock: go-dota2 + brackets-manager.js + Steam OpenID, build the rest. This session does not close it (HITL). |
| [How does a finished Lobby become a Match result?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/8) | **keep** | Libraries can *read* a winner. The v1 rule (auto vs Organizer confirm on failure) is still a decision. |
| [What can an Organizer override when the Lobby fails?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/9) | **keep** | No OSS encodes AFK's "unattended except exception path." |
| [How is Filipino grassroots eligibility enforced?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/10) | **keep** | Domain. Steam OpenID does not prove PH grassroots. |
| [Is a Match a single game or a series?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/11) | **keep** | BO3 vs the 1.5h Upper clock. Bracket library can store scores; it does not pick BO1. |
| [May a Team field substitutes?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/12) | keep (closed, out of scope) | Unchanged. |
| [Must a Player prove they own Dota, or is a SteamID enough?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/13) | **keep** | Not the same as "Steam OpenID exists." |
| [Do we accept Valve SSA risk on a dedicated host Steam account?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/14) | **keep — still the destination gate** | Comment restates the risk; it does not accept it. If this is "no," redraw the destination; go-dota2 becomes irrelevant. |
| [How do Players land on Radiant and Dire if the bot cannot force-seat them?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/15) | **keep** | Comment 2 skipped this gap. Closed research did not. |
| [Provision the Steam account that opens Lobbies](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/16) | **keep** (still blocked by 14) | Task, not a library choice. N accounts, not one. |
| [How does a Captain attach the other Players to a Team?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/17) | **keep** | Web SoR UX. No library. |
| [How many Teams enter a Weekly Tournament, and what happens to leftovers?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/18) | **keep, note a constraint** | brackets-manager.js **requires** power-of-two elimination size. That makes "pad with BYEs" cheaper to implement, not automatically the rule. |
| [Who auto-enters the next Weekly Tournament when there is more than one Field?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/19) | **keep** | AFK rule. No library knows "top 2" across isolated Fields. |
| [What happens when a Team does not arrive on time for a Match?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/20) | **keep, note a hook** | Forfeit-on-update exists in the engine. Wait duration, "incomplete" = 1 Player vs 5, and whether "missing" is visible next Saturday are still ours. |
| [How is a 16-Team Field paired?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/21) | **keep** | Engine ≠ policy. Seeding order, auto-enter placement, Lower drop-ins, Grand Final `simple` vs `double` stay here. |

### Do not add these tickets (the comments tempted them)

- "Research ValvePython lobby.py" — already issue 3.
- "Practice vs tournament Lobby" — already issue 3; v1 = practice.
- "Research Steam OpenID" — already issue 4.
- "Fork Dota2-EU-Ladder" — already rejected; also unlicensed.
- "Adopt FACEIT / Rivals" — already rejected as the product.

### Fog, unchanged

Still not sharp enough to ticket, and this insight does not sharpen them: server region; game mode (CM vs AP); hosting; how many host Steam accounts run Round 1 in parallel (hangs on ticket 14).

After ticket 7 closes, a later spec pass can name the split (TS web app + Go Lobby worker) without a new wayfinder ticket unless we want to lock language before `/to-spec`.

## Recommended answer for [Adopt existing software, or build?](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/7)

**Build AFK.** Adopt three pieces, not a product:

1. [paralin/go-dota2](https://github.com/paralin/go-dota2) for series-created practice Lobbies (create, invite, launch, read outcome).
2. [Drarig29/brackets-manager.js](https://github.com/Drarig29/brackets-manager.js) for Field double-elim math and progression, stored in the web app.
3. Steam OpenID for Player identity.

Do not adopt ValvePython/dota2 1.1.0, node-dota2, FACEIT, Rivals, Challonge-as-SoR, or Dota2-EU-Ladder.

## Sources

- Map comments (owner, 2026-08-19) on [AFK v1 — the way to a spec](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/1): ValvePython links; component-level write-up.
- [existing-software.md](https://github.com/ericvelasco-dev/AFKTournamentDota/blob/research/existing-software/docs/research/existing-software.md) on `research/existing-software`.
- [gc-lobby-bot.md](https://github.com/ericvelasco-dev/AFKTournamentDota/blob/research/gc-lobby-bot/docs/research/gc-lobby-bot.md) on `research/gc-lobby-bot`.
- GitHub API 2026-08-20: [ValvePython/dota2](https://github.com/ValvePython/dota2) last commit `6ccebc3` 2021-05-02; [paralin/go-dota2](https://github.com/paralin/go-dota2) MIT, push 2026-08-18, commit `2f03ac6` 2026-08-04; [Arcana/node-dota2](https://github.com/Arcana/node-dota2) archived; [UncleVasya/Dota2-EU-Ladder](https://github.com/UncleVasya/Dota2-EU-Ladder) no SPDX license, last push 2024-03-15; [Drarig29/brackets-manager.js](https://github.com/Drarig29/brackets-manager.js) MIT, v1.11.0 2026-05-17, 333 stars.
- [brackets-manager.js README](https://github.com/Drarig29/brackets-manager.js), [getting started](https://drarig29.github.io/brackets-docs/getting-started/), [seeding](https://drarig29.github.io/brackets-docs/user-guide/seeding/) (`grandFinal`, power-of-two, BYE at create, forfeit on update).
- [ValvePython/dota2 README](https://github.com/ValvePython/dota2/blob/master/README.rst) and [`dota2/features/lobby.py`](https://github.com/ValvePython/dota2/blob/master/dota2/features/lobby.py) (comment 1's sources; already used in issue 3).
