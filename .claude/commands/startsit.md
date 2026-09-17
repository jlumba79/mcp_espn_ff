---
description: Weekly start/sit read for your ESPN team, with live injury/role news
allowed-tools: Bash, WebSearch, WebFetch, Read
---

Produce a start/sit read for the user's fantasy team. Week: $ARGUMENTS (if empty, use the league's current week).

## Step 0 — Load league constants

Read `ESPN_LEAGUE_ID`, `ESPN_TEAM_ID`, and `ESPN_SEASON` from the `.env` file in this
directory. Never hardcode them. If `.env` is missing or those values are blank, stop and
tell the user to fill it in from `.env.example`.

`ESPN_TEAM_ID` is 1-based, matching ESPN's URL. The `espn_api` Python list is 0-based, so
the team is `league.teams[ESPN_TEAM_ID - 1]`.

Many leagues read without credentials — empty `ESPN_S2` / `SWID` is expected, not a bug.
Only if a call fails with a 401 should you tell the user to fill those in.

## Step 1 — Pull league data

Use `.venv/bin/python` in this directory. Get, for the target week:
- The matchup (opponent name, both projected totals) via `league.box_scores(week)`
- The user's full lineup with `slot_position`, `projected_points`, `injuryStatus`
- The opponent's full lineup, same fields
- Free agents by position (`league.free_agents(week=..., size=6, position=...)`) for WR/RB/TE/D-ST/K

## Step 2 — Get the news layer (DO NOT SKIP)

ESPN's projections and injury tags are stale and coarse — a player tagged QUESTIONABLE
may have not practiced all week, and a healthy player may have quietly lost his role.
This step is what makes the read worth anything.

WebSearch for current status on:
- **Every player with a non-ACTIVE injury tag**, on both rosters
- **Every player you are considering starting or benching** — even ACTIVE ones, because
  snap share and depth-chart changes don't show up in an injury tag
- Any opponent starter whose absence would swing the matchup

Search for practice participation (DNP / limited / full), snap share, target share, and
depth chart changes. Run these searches in parallel. Prefer reporting from the last 7 days;
say so explicitly if the freshest thing you find is older than that.

## Step 3 — Write the read

- **Lead with the actual decisions.** If only one or two lineup spots are genuinely in
  question, say so and don't pad the rest. "Everything else stays" is a complete answer
  for settled spots.
- **Weigh floor vs. ceiling against the matchup.** Favored by a comfortable margin →
  protect the floor. Big underdog → chase ceiling. State which situation this is.
- **A projection gap is not decisive on its own.** A 3-point edge for a player who didn't
  practice is not an edge. Say what the injury actually implies (late scratch risk,
  limited snaps) rather than restating the tag.
- **Cross-reference both rosters.** An OUT player on the opponent's bench may be a
  teammate of someone you're considering — that changes the target picture.
- **Check the opponent for mismanagement** — a QUESTIONABLE starter with a healthy,
  higher-projected bench option is a swing worth flagging, even if the user can't act on it.
- **Only recommend a waiver move if it upgrades a starting spot.** Otherwise say to save the FAAB.
- Cite sources as markdown links for every news-derived claim.

## Honesty requirements

- Claude's training cutoff predates this season. **Every** current-season factual claim —
  injuries, roles, depth charts, results — must come from Step 2 searches, never from
  recall. If a search came back thin or ambiguous, say that plainly instead of filling
  the gap with a plausible guess.
- Distinguish what the data says from what you're inferring.
- Claude can only READ this league — the MCP server and the underlying `espn_api` library
  have no write methods. Never imply a lineup change has been made. The user makes every
  move themselves in the ESPN app. End with what they need to go do.
