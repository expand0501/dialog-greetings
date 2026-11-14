# Dialog Greetings by Expand

Goal: stalkers say their greeting line when the player opens a dialog with them, and (MCM, on by default) no longer
greet the player every time he walks past within 3 m. Standalone, no base GAMMA mod required, no replaced scripts.
Shipped readmes: `src/README_EN.txt`, `src/README_RU.txt` (cp1251).

## Files
- `src/gamedata/scripts/dlg_greet.script`: all logic (log tag `[DialogGreetings]`).
- `src/gamedata/scripts/dlg_greet_mcm.script`: MCM menu (id `dlg_greet`, single page). Defaults must match
  `DEFAULTS` in the main script.
- `src/gamedata/configs/misc/sound/script_sound_dlg_greet.ltx`: sound theme `[dlg_greet_hello]`, auto-included by
  vanilla `script_sound.ltx` (`#include "script_sound_*.ltx"`).
- `src/gamedata/configs/text/{eng,rus}/ui_st_dlg_greet_mcm.xml`: MCM strings (rus = Windows-1251).

## Version
In `mod.json`, both readme titles + history, script header, both MCM text files (comment line).

## How it works (keep)
- Dialog open hook: `pda.actor_menu_mode(10)` (vanilla `pda.script`; same code in the two enabled overrides
  "G.A.M.M.A. ZCP 1.4 Balanced Spawns" and "PDA REPLACER BTTR" (only in `v31.1 - radiation rework`)) loops
  `db.storage` for the talking NPC, `SetEvent("used_npc_id", id)`, calls `sound_theme:reset(id)` on his current
  sound, then `Register_UI("Dialog")` -> `GUI_on_show("Dialog")`. We read `GetEvent("used_npc_id")` there. Works for
  NPCs without use_callback (traders) too, which is why this hook and not `npc_on_use`; also `npc_on_use` fires
  before the reset, which would cut our line.
- Why our own theme instead of `meet_hello`: `npc_sound` keeps `played_time` / `idle_time` per THEME, not per NPC, so
  vanilla `meet_hello` (idle 7-10 s) has one cooldown shared by all stalkers. `xr_sound.set_sound_play` also refuses
  while `sound_table[id]` is set (pda's reset does not clear it). `[dlg_greet_hello]` = same path/communities,
  `idle = 0,0,100`, `play_always = true` (set_sound_play resets the current sound and plays).
  Inheriting `[dlg_greet_hello]:meet_hello` is NOT possible: script_sound.ltx includes `script_sound_*.ltx` at the
  top, before `[meet_hello]` is defined (parent must exist first).
- NPCs whose voice set has no meet_hello files (or community not listed) never get `theme.npc[id]`; we skip them
  (read-only look at `sound_theme.theme[THEME].npc`, vanilla script, not one of Quark's).
- Walking past: xr_meet (winner: "123- Modular Tweaks - Vintar0", both profiles) plays `close_snd_hello`
  (`meet_hello`, or `meet_stop` when the actor has a weapon on first meet) and `far_snd` (`meet_wait` /
  `meet_hide_weapon`) via `xr_sound.set_sound_play(id, snd)`. We wrap `xr_sound.set_sound_play` once in
  `on_game_start` (guard: `orig_set_sound_play`) and return nil for muted themes. Callers ignore the return value
  (and vanilla npc_sound has no `snd_obj`, so it is nil anyway). Our own theme name is never muted.
- Save safety: Anomaly saves through marshal (`USE_MARSHAL`), `xr_sound.actor_save_state` stores theme state by
  section name, so adding/removing a theme is safe. The mod saves nothing; `last_greet` (per-NPC cooldown) lives
  for the session and is cleared on first update / net destroy.
- Checks before greeting: actor alive, NPC stalker and alive, not hostile (`npc:relation(actor)`), not companion
  (`npcx_is_companion`, MCM), not wounded (`xr_wounded.is_wounded`), cooldown, chance.

## Profiles
- `v31.1 - radiation rework` and `G.A.M.M.A`: no conflicting mods found (no enabled mod overrides `xr_sound.script`,
  `sound_theme.script` or `[meet_hello]`). GAMMA `dev2` (2026-11-14): same, Anti-Lag doesn't touch these scripts.
- Voiced Actor mods (actor voice) don't hook the dialog opening.

## Open items
- 0.1.0 / 0.1.1 not tested in game (0.1.1: weapon warnings muted by default, Expand's request). Check: greeting on dialog open (normal stalker, Duty/Freedom base NPCs, a story NPC),
  subtitles, no greeting from traders/hostile/wounded/companions, cooldown after trade, chance 0, walking past is
  silent with mute on and vanilla with mute off, weapon warning muting, MCM live changes, debug lines in the log.
- Possible: does the NPC's greeting overlap with the first line of a story dialog that plays its own voice?
- No GitHub repository yet (ask Expand: public or private); not in the root README / SETUP tables yet.
