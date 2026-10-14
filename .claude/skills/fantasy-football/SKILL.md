---
name: fantasy-football
description: Fantasy football analyst for the user's Sleeper league. Use whenever the user mentions a Sleeper league ID or asks about their fantasy team, roster, lineup, start/sit, waivers, free agents, trades, matchups, standings, or trending players.
---

# Fantasy Football Analyst (Sleeper)

All league data comes from Sleeper's free public API (no key): `https://api.sleeper.app/v1`.
A league ID is all that's needed. If the user hasn't given one this conversation, ask for it.

## Loading the league (do this first, with curl + jq, saving responses to the scratchpad)

1. `GET /league/<league_id>`: name, season, scoring settings (PPR/half/std, superflex), roster positions.
2. `GET /league/<league_id>/users`: map `user_id` to `display_name` / `metadata.team_name`.
3. `GET /league/<league_id>/rosters`: `owner_id`, `players`, `starters`, `reserve` (IR), `settings` (wins/losses/fpts).
4. `GET /state/nfl`: current `week`.
5. `GET /league/<league_id>/matchups/<week>`: pairs by `matchup_id`, points, starters.
6. `GET /players/nfl`: player map (~5MB). Download once to a file and never print it. Look up IDs with jq,
   e.g. `jq '."4046" | {full_name, position, team, injury_status}'`.

To find the user's team, list the team names and ask once which is theirs, then remember it for the conversation.

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
