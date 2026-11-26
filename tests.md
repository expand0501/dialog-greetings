# Dialog Greetings: automatic in-game tests

Runner: `tools\autotest\mod\gamedata\scripts\zzz_claude_autotest_dg.script`, `run.ps1 -Suite dg` (log
`claude_autotest_dg.log`). Tests the installed zip unchanged. Profile `Claude Test`, save `claude_test`.

## How the runner sees the mod
- MCM values per scenario in memory (`ui_mcm.get` wrapper), debug log on, `[DialogGreetings]` lines captured.
- Two recorders around `xr_sound.set_sound_play`:
  - **outer** (wraps the public `xr_sound.set_sound_play`, which is the mod's filter or Monolith Voice
    Greetings' wrapper on top of it): what callers ask for (xr_meet, the mod's own greeting).
  - **inner** (replaces the mod's upvalue `orig_set_sound_play`): what the filter lets through to vanilla.
  Muted = outer saw it, inner didn't.
- Mod internals through upvalues of its public functions / wrappers (only functions whose source is `dlg_greet`):
  `DEFAULTS`, `HELLO_THEMES`, `WARNING_THEMES`, `ONCE_THEMES`, `filter_condlist`, `meet_records`, `orig_set_sound_play`.
  `basics` checks every one is readable.
- Dialogs are real: `db.actor:run_talk_dialog(npc)` opens the talk window (engine → `pda.actor_menu_mode(10)` →
  `GUI_on_show("Dialog")`, the player's path); `db.actor:stop_talk()` closes it.
- NPCs: spawned `sim_default_stalker_*`, 3 m in front, goodwill 3000 to the actor, talk enabled.
- Expected values from the runner's own copy (defaults table, condlist filter, mute lists).

## Scenarios
| Scenario | Setup | Proof | UNCLEAR when |
|---|---|---|---|
| `basics` | none | version 1.0.5 (script header, text files), wrappers in place, sound theme loaded with `play_always`, no cooldown, MCM page ids/defaults = runner copy = mod `DEFAULTS`, eng strings resolve, rus same ids + cp1251, public API, `filter_condlist` on fixed condlists vs runner copy, once-per-visit logic with a stubbed vanilla | internals not readable |
| `greet` | defaults | dialog opens, inner call `dlg_greet_hello` for the partner, `sound_table[id]` = our theme, debug line `greets with` | no dialog / NPC without greeting voice set |
| `cooldown` | cooldown 60 | 2nd dialog: no greeting, line `greeted N ms ago` | |
| `cooldown_off` | cooldown 0 | 2nd dialog greets again | |
| `chance_off` | chance 0 | no greeting, line `chance roll failed` | |
| `disabled` | enabled off | no greeting, no mod line | |
| `refuse` | defaults | hostile (`is hostile`), companion info (`is a companion`), wounded (`is wounded`): no greeting each | NPC never wounded |
| `companions_on` | companions on | companion info NPC greets | |
| `partner` | 2 NPCs, `used_npc_id` set to the other one when the dialog opens | partner greets, decoy doesn't, line `used_npc_id X is not the dialog partner Y` (1.0.3 regression) | |
| `provider` | providers: error, unknown theme, real theme | error logged once, fallback, provider theme played instead of ours | no usable theme |
| `walk_mute` | defaults | direct calls: hello/guard/catwalk/warning themes muted, other themes pass; real xr_meet walk-up muted | xr_meet never calls (NPC didn't see the actor) |
| `walk_vanilla` | mute_hello + mute_warnings off | same calls all pass through | |
| `weapon_reaction` | no_weapon_reaction on | NPC's meet fields hold filtered copies (= runner copy), originals unchanged in xr_logic memo | NPC has no default meet |
| `weapon_live` | off, then on, then off via `on_option_change` | fields follow live | |
| `mcm_live` | change mute_hello / enabled live | filter and greeting follow | |
| `no_mcm` | every MCM value nil | defaults apply (greets, mutes) | |
| `mcm_open` | MCM opened on the page, Cancel | no error | |
| `saveload` | greet, save, load | wrappers installed once, greets again after load (cooldown cleared), no errors; overwrites `claude_autotest` | |

## Not automated (for Expand)
- How the greeting sounds and whether subtitles show (by ear / eye).
- Story NPCs at their place (Rostok guards, catwalk zoneguard, informant): only their theme names are checked.
- Traders / NPCs without greeting voices (no spawnable example).
- Overlap with a story dialog's own voice line.

## Results
- 2026-11-26, 1.0.5 zip, profile `Claude Test`: 17 of 18 PASS; `walk_mute` FAIL = `once_other_line` (mod bug in
  `play_once`, see CLAUDE.md). Reports: `reports\autotest-20261126-151509.txt`, `-152147.txt`.
- 2026-11-26, 1.0.6 (copied into `Claude Autotest`): basics, walk_mute, walk_vanilla PASS (`reports\autotest-20261126-153559.txt`).
