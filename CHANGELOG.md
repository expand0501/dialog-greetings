# Changelog - Dialog Greetings by Expand

- 1.0.5: public API for other mods: `dlg_greet.add_greeting_provider(fn)` (fn(npc, npc_id) returns a sound theme
  the NPC plays as his dialog greeting instead of `dlg_greet_hello`; all our checks run first) and
  `dlg_greet.walk_hello_muted()`. Used by Monolith Voice Greetings, so Monolith stalkers (silent greeting files)
  greet with other factions' lines. No change for players without such a mod.
- 1.0.4: new MCM option "No reaction to a raised weapon" (`no_weapon_reaction`, off by default). Wraps
  `xr_meet.init_meet` once; for meet fields the NPC's own meet section doesn't set (the default meet of generic
  stalkers), the parsed condlists get a copy with `=actor_has_weapon` treated as false: no `threat_na` aiming, no
  bigger stop distance, talk animation kept, `meet_stop` / `meet_hide_weapon` replaced by the normal lines. Story
  NPCs with their own meet fields keep their weapon logic. The originals are kept per NPC, so changing the option
  re-applies it to every online NPC at once (one loop over `db.storage` on Apply only).
- 1.0.3: fixed the greeting sometimes coming from another stalker nearby. Vanilla `pda.actor_menu_mode(10)` stores
  the first NPC in `db.storage` whose `is_talking()` is true as `used_npc_id`, not always the dialog partner; the
  greeting now goes to the engine's talk partner (`actor:get_talking_npc()`, modded exes), `used_npc_id` only as a
  fallback. The Rostok informant (`bar_informator_mlr`, line `bar_informer_1_mlr`, 3 m, every 100 s) says his line
  only once per level visit with "No greeting when walking past" on (new list `ONCE_THEMES`; reset on level change
  and save load).
- 1.0.2: the Duty zoneguard on the Rostok hangar catwalk ("Иди своей дорогой, сталкер", random name such as
  "Старшина Колмаков") is muted with "No greeting when walking past". His line is not a meet sound: his logic
  plays `%=play_sound(bar_dolg_hangar_bridge_hello)%` every 16-25 s while you stand in `bar_angar_warn_zone`.
  Debug log now also names the sound theme of every voice line a stalker within 20 m starts. Fixed Zhorik's
  location in the 1.0.1 texts (stairs to the bar, not the hangar catwalk).
- 1.0.1: the Rostok guards Zhorik (stairs to the bar, `bar_guard_prohodi`, 5 m) and Garik (bar door, `bar_guard_stop`,
  2 m) no longer repeat their line every time you pass them. Their vanilla meet scheme plays its own story theme
  instead of `meet_hello`, so the walking-past filter missed it; both themes are now muted with "No greeting when
  walking past".
- 1.0.0: first public release. Version label only, no code changes.
- 0.1.1: MCM "No weapon warnings when walking past" is now on by default (stalkers no longer tell you to put your
  weapon away or to stop when you walk up to them). A value already saved in MCM stays as it is.
- 0.1.0: first version. When you open a dialog with a stalker he says one of his greeting lines (own sound section
  `dlg_greet_hello`: vanilla `meet_hello` voice files, no shared cooldown). Hostile, wounded and voiceless NPCs
  stay silent; companions only with the MCM option. Per-stalker cooldown (default 60 s) and chance. Optional
  muting of the walking-past greeting (on by default) and of the walking-past weapon warnings (off by default).
  No per-frame work, no replaced game scripts.
