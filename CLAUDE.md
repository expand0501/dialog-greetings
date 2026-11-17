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
- Story NPCs with a custom meet `close_snd_hello` theme pass the filter unless their theme is listed in
  `HELLO_THEMES` (1.0.1: Rostok guards `bar_guard_prohodi` = Zhorik, `bar_guard_stop` = Garik, both `use = false`,
  vanilla `configs\scripts\bar\bar_guard{1,2}.ltx`). Expand chose this per-name list over a general hook on
  `xr_meet.init_meet` (2026-11-15): that hook depends on xr_meet internals (several mods replace the script,
  Homestead writes `st.meet.close_snd_hello` itself), breaks live MCM toggling and hides story-state lines
  (`zat_b3_tech_drink_no_more`, Beard's angry greeting). Add new offenders to the list when found.
  Vanilla Anomaly configs (unpacked) for lookups: GitHub `Tosox/STALKER-Anomaly-gamedata`.
  Zhorik stands on the stairs to the bar (not the hangar catwalk). 1.0.2: the hangar catwalk guard is the Duty
  zoneguard (`bar_dolg_general_zoneguard_stalker`, name `GENERATE_NAME_senior_sergeant`, e.g. "Старшина Колмаков",
  has a dialog): `on_actor_in_zone = bar_angar_warn_zone | %=play_sound(bar_dolg_hangar_bridge_hello)%`, and
  `xr_effects.play_sound` also ends in `xr_sound.set_sound_play`, so logic-effect lines are muted the same way.
- Dialog partner (1.0.3): `actor:get_talking_npc()` (modded exes; in 0.9.5 and dev2 `lua_help_ex`) first, checked
  once with pcall; `used_npc_id` only as fallback, because pda's loop takes the first `is_talking()` NPC, which
  played other stalkers' greetings (Expand's report 2026-11-17; root cause = inference, confirm with debug line
  `used_npc_id X is not the dialog partner Y`).
- `ONCE_THEMES` (1.0.3): walking-past lines Expand wants kept but said once per level visit (Rostok informant
  `bar_informer_1_mlr`). Counted only when `xr_sound.sound_table[id]` is that theme after the call; `once_played`
  resets in `reset_state` (first update / net destroy), so a save load in the same level plays it again.
- Not touched (verified 2026-11-17): campfire stories/jokes/guitar, combat lines and zombied stalkers use other
  themes; mutant (zombie monster) sounds don't go through `xr_sound.set_sound_play`.
- Raised-weapon option (1.0.4, `no_weapon_reaction`, off by default; Expand asked for live apply 2026-11-17):
  wraps `xr_meet.init_meet` (guard `orig_init_meet`; same signature in vanilla, Modular Tweaks (winner, both
  profiles) and Great_Day's "108- Remove dropping weapons from damage"). After the original runs, for each of
  close/far distance, anim, close_snd_hello, far_snd that the NPC's meet section does NOT set (`ini:line_exist`)
  and that contains `actor_has_weapon`, we store the original condlist in `meet_records[st]` (weak keys, st =
  `db.storage[id].meet`) and put a filtered copy in st (entries with `=actor_has_weapon` dropped, `!` checks
  removed). Parsed condlists are memoized/shared by xr_logic, so never mutate them. init_meet returns early when the
  section is unchanged: `original_of` maps our copy back. Modular Tweaks' squad "commander_threat" follows,
  because the commander never enters `threat_na`. Expand accepted the dependency on xr_meet internals here (unlike
  the walking-past hello list); `section == "no_meet"` is skipped.
- Finding an unknown line: with MCM Debug log on, every `set_sound_play` call for a stalker within 20 m is logged
  as `[DialogGreetings] npc <id> (<name>) plays sound <theme>` (debug only; muted ones as `muted walking-past`).
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
- 1.0.0 released 2026-11-14 after Expand tested it in game (1.0.0 = version label only; 0.1.1: weapon warnings muted by default). Test list used: greeting on dialog open (normal stalker, Duty/Freedom base NPCs, a story NPC),
  subtitles, no greeting from traders/hostile/wounded/companions, cooldown after trade, chance 0, walking past is
  silent with mute on and vanilla with mute off, weapon warning muting, MCM live changes, debug lines in the log.
- 1.0.1 (2026-11-15): Zhorik confirmed silent in game; the catwalk guard still talked (different NPC, see 1.0.2).
- 1.0.2 (2026-11-15): catwalk zoneguard muted + debug logs nearby voice lines; needs an in-game check (catwalk guard
  and Garik silent with mute on, vanilla with mute off; debug lines appear).
- 1.0.3 + 1.0.4 (2026-11-17): Expand tested 1.0.4 in game ("works as far as I can tell"); released together as
  v1.0.4 (1.0.1-1.0.3 never got their own tag/release; last release before was v1.0.0).
- Possible: does the NPC's greeting overlap with the first line of a story dialog that plays its own voice?
- GitHub: `expand0501/dialog-greetings` (public). Discord descriptions: `docs/discord_en.md`, `docs/discord_ru.md`.
