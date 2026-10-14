---
name: fantasy-football
description: Fantasy football analyst for the user's Sleeper league. Use whenever the user mentions a Sleeper league ID or asks about their fantasy team, roster, lineup, start/sit, waivers, free agents, trades, matchups, standings, or trending players.
---

# Fantasy Football Analyst (Sleeper)

All league data comes from Sleeper's free public API (no key): `https://api.sleeper.app/v1`.
A league ID is all that's needed.

## The user's saved leagues

Use these by default (analyze both unless the user names one). The user is `ryanlinni` in both.

| League | Sleeper league ID | Format | User's team |
|--------|-------------------|--------|-------------|
| Jug nation dynasty | `1336808939680989184` | 10-team dynasty, full PPR, K + DEF, rolling waivers, 3 taxi slots | The Naughty Gargamels (roster_id 3) |
| HMSD Keeper | `1312196395939164160` | 12-team keeper, half PPR + 0.5 TE bonus, no K/DEF, FAAB ($100) | Naughty Gargamel (roster_id 8) |

**HMSD Keeper rules** (from the user, plus draft history):
- TE premium: TEs get +0.5 per catch (effectively full PPR). Public value sites (FantasyCalc, KTC) ignore this, so adjust TEs up.
- Up to 3 keepers. A player can't be kept two years in a row, and you can't keep your own top two draft picks.
- Keeper cost appears to be one round earlier than the player's draft round the year before (inferred from draft history, not confirmed).

**Value sources that work:** KeepTradeCut dynasty values (`https://keeptradecut.com/dynasty-rankings?format=1`, JSON in the `<script id="ktc-players">` tag) and the FantasyCalc API (`https://api.fantasycalc.com/values/current?isDynasty=<bool>&numQbs=1&numTeams=<n>&ppr=<0.5|1>`, includes `sleeperId`).

## Loading the league (do this first, with curl + jq, saving responses to the scratchpad)

1. `GET /league/<league_id>`: name, season, scoring settings (PPR/half/std, superflex), roster positions.
2. `GET /league/<league_id>/users`: map `user_id` to `display_name` / `metadata.team_name`.
3. `GET /league/<league_id>/rosters`: `owner_id`, `players`, `starters`, `reserve` (IR), `settings` (wins/losses/fpts).
4. `GET /state/nfl`: current `week`.
5. `GET /league/<league_id>/matchups/<week>`: pairs by `matchup_id`, points, starters.
6. `GET /players/nfl`: player map (~5MB). Download once to a file and never print it. Look up IDs with jq,
   e.g. `jq '."4046" | {full_name, position, team, injury_status}'`.

For a league not listed above, list the team names and ask once which is theirs, then remember it for the conversation.

Other useful endpoints:
- Free agents: players in the map with `active == true`, a `team`, and a fantasy position, that are on no league roster.
- Trending: `/players/nfl/trending/add?lookback_hours=24&limit=25` (and `/drop`).
- Transactions: `/league/<league_id>/transactions/<week>`.
- Picks and drafts: `/league/<league_id>/traded_picks`, `/league/<league_id>/drafts`.

## Rules

- League data (rosters, matchups, scores, free agents, standings) must come from the API. Never guess or use memory.
  If the API is unreachable, say so (the environment needs `api.sleeper.app` allowed) and stop.
- Injuries, news, depth charts, projections and rankings come from web search; note how recent the info is.
- Always use player names (with position/team), never Sleeper IDs.
- Lead with the actionable answer (start X, add Y, accept/decline), then a short why. Keep it concise.
