# Does a private Steam profile still work for AFK identity?

**Ticket:** [#4](https://github.com/ericvelasco-dev/AFKTournamentDota/issues/4)

**Gist:** Yes — a private Steam profile still logs the Player in and still yields a SteamID the Lobby bot can invite. Persona name and Dota friend / account ID do not require a public profile. Owned games and match history do (and match history has its own Dota expose-data switch).

This note separates **can AFK know who they are** from **can AFK know they play Dota**. It does not decide product policy.

## Sources

Primary sources used below. Live Valve wiki pages (`developer.valvesoftware.com`) currently sit behind an Anubis bot-check; those claims cite the archived Valve Developer Community snapshot.

| Id | Source | What it owns |
| --- | --- | --- |
| [S1] | [Steam Web API Documentation — Steam OpenID Provider](https://steamcommunity.com/dev) | Official third-party OpenID endpoint and Claimed ID format |
| [S2] | [Steamworks: User Authentication and Ownership](https://partner.steamgames.com/doc/features/auth) | OpenID 2.0 login, SteamID as identity, publisher-only ownership checks |
| [S3] | [OpenID Authentication 2.0](https://openid.net/specs/openid-authentication-2_0.html) | What OpenID proves (control of an Identifier), not profile data |
| [S4] | [Steamworks Web API: ISteamUser](https://partner.steamgames.com/doc/webapi/ISteamUser) | `GetPlayerSummaries` response, including a private-profile example; `GetFriendList` 401; publisher `CheckAppOwnership` |
| [S5] | [Steamworks Web API: IPlayerService](https://partner.steamgames.com/doc/webapi/IPlayerService) | `GetOwnedGames` visibility clause |
| [S6] | [Valve Developer Community: Steam Web API](https://web.archive.org/web/20240530225012/https://developer.valvesoftware.com/wiki/Steam_Web_API) (archived 2024-05-30) | Public vs private `GetPlayerSummaries` fields; `GetOwnedGames` / `GetRecentlyPlayedGames` public-profile requirement |
| [S7] | [Official TF Wiki: WebAPI/GetMatchHistory](https://wiki.teamfortress.com/wiki/WebAPI/GetMatchHistory) | Dota `account_id` (32- or 64-bit); status `15` when the user has not allowed match history |
| [S8] | [Valve Developer Community: SteamID](https://web.archive.org/web/20240530121907/https://developer.valvesoftware.com/wiki/SteamID) (archived 2024-05-30) | 64-bit SteamID bit layout; 32-bit account number is packed in, not fetched |
| [S9] | [Steamworks: steam_api.h — CSteamID](https://partner.steamgames.com/doc/api/steam_api#CSteamID) | `CSteamID` is the globally unique identifier for Steam accounts |
| [S10] | [Steamworks: ISteamMatchmaking.InviteUserToLobby](https://partner.steamgames.com/doc/api/ISteamMatchmaking#InviteUserToLobby) | Lobby invite is by `CSteamID`, not by public profile |
| [S11] | [Steam Support: Profile Privacy](https://help.steampowered.com/en/faqs/view/588C-C67D-0251-C276) | Public / Friends Only / Private, plus per-component subcategories |
| [S12] | [Steam Support: Private Games](https://help.steampowered.com/en/faqs/view/1150-C06F-4D62-4966) | Per-title hide of ownership / playtime; SteamID still visible in a matchmade session |
| [S13] | [Steam Web API Terms of Use](https://steamcommunity.com/dev/apiterms) | Sites must not collect Steam passwords; OpenID is the allowed login path |
| [S14] | [Steamworks Web API Overview](https://partner.steamgames.com/doc/webapi_overview) | Users are identified by 64-bit SteamID; obtain it via the auth doc |

## Answer in one table

| Fact | Works if the Steam profile is private? | What actually gates it |
| --- | --- | --- |
| Login / connect Steam | **Yes** | Steam OpenID after the Player signs in on Steam. Profile visibility is not a documented precondition. [S1] [S2] [S3] |
| Stable identity (64-bit SteamID) | **Yes** | Returned as the OpenID Claimed ID. [S1] [S2] |
| Dota friend ID / 32-bit account ID | **Yes** | Arithmetic on the SteamID already in hand. Not a privacy-gated lookup. [S7] [S8] |
| Persona name (display name) | **Yes (public field)** | `GetPlayerSummaries` lists `personaname` as public data and Valve’s own private-profile example still returns it. [S4] [S6] |
| Currently-in-Dota / rich presence | **No** | `gameid` / `gameextrainfo` are private data. [S6] |
| Owned games / playtime (incl. Dota 2, appid 570) | **No** | `GetOwnedGames` only if owned games / game details are visible to the caller. Public profile is not enough if Game details is Friends Only / Private, or if Dota is marked private. [S5] [S6] [S11] [S12] |
| Match history | **No** | Separate Dota “allow match history” bit. Hidden history returns status `15` even when you already have the account ID. [S7] |
| Lobby bot invite | **Yes** | Invite is addressed to a `CSteamID`. [S10] [S12] |
| Proof they own / play Dota via publisher ownership APIs | **Not available to AFK** | `CheckAppOwnership` / `GetPublisherAppOwnership` require the **publisher** key that owns that AppID. AFK is not Dota 2’s publisher. [S2] [S4] |

## Who they are (identity)

### Login does not read the community profile

Steam is an OpenID 2.0 provider. A third-party site sends the Player to Steam’s login form; after they authenticate there, the browser returns to the site with an assertion whose Claimed ID contains the 64-bit SteamID. [S1] [S2]

- Provider / OP Endpoint: `https://steamcommunity.com/openid` [S1] / `https://steamcommunity.com/openid/` [S2]
- Claimed ID format: `https://steamcommunity.com/openid/id/<steamid>` [S1] (Steamworks also documents the `http://` form of the same path). [S2]
- Steamworks: “Steam's OpenID 2.0 implementation can be used to link a users Steam account to their account on the third-party website.” After authentication, “the user's Claimed ID will contain the user's SteamID.” [S2]
- Official `/dev` page: Steam OpenID “authenticate[s] a user's SteamID without requiring them to enter their Steam username or password on your site (which would be a violation of the API Terms of Use).” [S1] [S13]

OpenID 2.0 itself only proves the end user **controls an Identifier**. Profile attributes are out of band: “The exchange of profile information, or the exchange of other information not covered in this specification, can be addressed through additional service types… OpenID Authentication provides a way to prove that an end user controls an Identifier… without the Relying Party needing access to end user credentials… or to other sensitive information such as an email address.” [S3]

Neither [S1] nor [S2] lists community visibility (Public / Friends Only / Private) as a condition of a successful OpenID assertion. The documented output of a successful login is the SteamID, not a public profile page.

Steamworks names that 64-bit value as the identity used everywhere else: “Every Steam user can be uniquely identified by a 64-bit numeric ID, known as the user's Steam ID,” carried in a `CSteamID`. [S2] [S9] The Web API “identifies individual users by using the their unique 64-bit Steam ID.” [S14]

**AFK implication:** connecting Steam as the Player’s identity works with a private profile. The series stores the SteamID from OpenID. That is enough to know *who* they are.

### Persona name

`ISteamUser/GetPlayerSummaries` is the documented way to fetch display data for a SteamID. [S4] Valve’s Web API wiki splits the payload:

**Public data** (still returned when visibility is Friends Only or Private): `steamid`, `personaname`, `profileurl`, avatars, `personastate` (forced to `0` / Offline on a private profile, with a noted exception), `communityvisibilitystate`, `profilestate`, `lastlogoff`, `commentpermission`. [S6]

**Private data** (hidden unless the profile is visible to the caller): `realname`, `primaryclanid`, `timecreated`, `gameid`, `gameserverip`, `gameextrainfo`, location fields. [S6]

`communityvisibilitystate` on an unauthenticated Web API call is only `1` (not visible: Private, Friends Only, etc.) or `3` (Public). [S6]

Steamworks’ own `GetPlayerSummaries` example includes a player with `"communityvisibilitystate": 1` who still has `"personaname": "Mister Manager"` plus avatars and `profileurl`. [S4]

**AFK implication:** showing the Steam persona name does not require a public profile. Showing “currently in Dota 2” (`gameid` / `gameextrainfo`) does. [S6]

### Dota friend ID / account ID

Dota’s Web API identifies a player with a 32-bit `account_id`. `IDOTA2Match_<ID>/GetMatchHistory` takes `account_id` as “32-bit or 64-bit account ID” and each player object returns the 32-bit `account_id`. [S7]

That 32-bit number is not a second secret lookup. A SteamID is a 64-bit structure whose **lowest 32 bits are the account number** (lowest bit = Y, next 31 bits = account number). Individual accounts use type identifier `0x0110000100000000`. [S8] `CSteamID` is that same identifier. [S9]

Conversion is local arithmetic on the SteamID OpenID already returned. Profile privacy is not involved.

In Dota client UI this 32-bit value is the **Friend ID**. AFK can derive it from the connected Steam profile even when that profile is private.

## Whether they play Dota (library, live status, matches)

### Owned games and playtime

`IPlayerService/GetOwnedGames` “returns a list of games owned by the player **if their owned games/game details are visible to you**.” [S5]

The Valve Web API wiki is stricter in wording: `GetOwnedGames` returns the library “**if the profile is publicly visible**. Private, friends-only, and other privacy settings are not supported unless you are asking for your own personal details (ie the WebAPI key you are using is linked to the steamid you are requesting).” Same rule for `GetRecentlyPlayedGames`. [S6]

Steam Support adds two finer gates on top of “profile is Public”:

1. Profile privacy has **subcategories** (Game details, inventory, etc.) independent of the top-level Public / Friends Only / Private state. [S11]
2. A Player can **mark Dota 2 private**. That hides ownership, in-game status, playtime, and activity for that title as if they did not own it. [S12]

**AFK implication:** a user-key `GetOwnedGames` call (or filtering for appid `570`) cannot be relied on to prove Dota ownership when the profile, Game details, or the Dota title itself is private. Empty / missing results are ambiguous (private vs. does not own).

### Publisher ownership checks are not a workaround

Steamworks documents `ISteamUser/CheckAppOwnership` and `ISteamUser/GetPublisherAppOwnership` as the post-OpenID way to prove a user owns an AppID. Both “require the publisher API key that owns the specified App ID” / “a publisher API key” and “**MUST** be called from a secure server.” [S2] [S4]

AFK is a third-party series, not the publisher of Dota 2 (appid 570). Those methods do not apply to checking Dota ownership.

### Match history is a second, Dota-specific switch

`GetMatchHistory` with an `account_id` “will fail if the user has their match history hidden” (unofficial but widely restated). The official TF WebAPI page, which documents this Valve method, returns:

- `status` `1` — Success
- `status` `15` — “Cannot get match history for a user that hasn't allowed it.” [S7]

That is **not** the same bit as Steam community visibility. A public Steam profile can still hide Dota match data; a private Steam profile can still expose it (if the Dota setting allows). Knowing the account ID is necessary but not sufficient. [S7]

### Friends list (not required for identity, listed for completeness)

`GetFriendList` returns 401 Unauthorized “if a user's friend list is marked as private.” [S4] The Valve wiki: it returns the list only when “Steam Community profile visibility is set to Public”; “Nothing will be returned if the profile is private.” [S6]

AFK identity does not need the friends list.

## Inviting the Player to a Lobby

AFK needs the connected Steam profile so a Lobby bot can invite the Player.

`ISteamMatchmaking::InviteUserToLobby(CSteamID steamIDLobby, CSteamID steamIDInvitee)` invites **by SteamID**. If the invitee is in-game, they get `GameLobbyJoinRequested_t`; if not, the game launches with `+connect_lobby <64-bit lobby Steam ID>`. [S10] No public-profile requirement is documented.

Steam Support’s Private Games FAQ is consistent: in a Steam-matchmade session “your Steam ID will be visible to other players on the same server,” and an invited friend can see in-game status for that session so they can join. [S12] Hiding the community profile (or marking the game private) does not remove the SteamID as an invite address.

**AFK implication:** the bot needs the 64-bit SteamID from OpenID. A private profile does not block the invite path.

## What AFK should treat as required

For the job “identify a Player and invite them to a Lobby”:

- **Required, and available on a private profile:** successful Steam OpenID, persist the 64-bit SteamID, derive the 32-bit Dota account / friend ID, invite that `CSteamID`. [S1] [S2] [S7] [S8] [S10]
- **Useful, and usually available on a private profile:** persona name via `GetPlayerSummaries`. [S4] [S6]
- **Not required for identity, and not available unless the Player opens the relevant privacy bits:** owned-games / playtime proof they own Dota, live “in Dota” presence, match history. [S5] [S6] [S7] [S11] [S12]
- **Not available to AFK at all:** publisher `CheckAppOwnership` of Dota 2. [S2] [S4]

If a later rule wants “prove they play Dota” at registration, that is a **separate product requirement**. It cannot be satisfied by OpenID alone, and a private (or Game-details-private) profile will fail it. The identity path itself should not be blocked on public-profile.

## What this note does not settle

- Whether AFK should *ask* Players to set Game details public or expose match history (product / trust, not an API fact).
- How a Dota practice-lobby bot authenticates to Valve (session tickets vs. a logged-in client). Invite addressing is settled; bot login is not.
- Whether empty `GetOwnedGames` should be treated as “private” or “doesn’t own Dota” (the API does not distinguish). [S5] [S6]
