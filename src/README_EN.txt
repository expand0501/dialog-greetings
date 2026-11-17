Dialog Greetings 1.0.4 by Expand
Stalkers greet you when you start talking to them - not every time you walk past.
For S.T.A.L.K.E.R. Anomaly / G.A.M.M.A.

WHAT IT DOES
- When you open a dialog with a stalker, he says one of his own greeting lines
  ("Hello", "Hi there"...), in his own voice and with subtitles, like the vanilla greeting.
- Optional (on by default): stalkers no longer say hello every time you come within
  3 m of them while you just walk past.
- Optional (on by default): stalkers also skip the "put your weapon away" / "stop"
  lines when you walk up to them with a weapon in your hands.
- Optional (off by default): stalkers don't react to your raised weapon at all - they
  don't aim at you or stop you, as if your weapon were lowered. Story characters with
  their own rules (base checkpoints, guards) still react.

Hostile and wounded stalkers stay silent. Traders and other characters without
greeting voice lines stay silent too. Companions don't greet by default (MCM).
The same stalker doesn't greet you again for 60 seconds (MCM), so reopening a
dialog, for example after trading, doesn't repeat the greeting.

REQUIREMENTS
- Anomaly 1.5.2+ / G.A.M.M.A. No other mods needed.
- MCM (included in GAMMA) for the settings; without MCM the defaults below are used.

INSTALLATION (MO2)
1. Install this archive with MO2 ("Install a new mod from an archive") and enable it.
2. Restart the game. Safe to add or remove mid-playthrough.

MCM -> "Dialog Greetings" (all settings apply at once, no restart)
  When a dialog opens:
    Greet when a dialog opens                   - on
    Greeting chance (%)                         - 0..100, default 100
    Greet the same stalker again after (seconds) - 0..600, default 60 (0 = every time)
    Companions greet too                        - off
  When you walk past:
    No greeting when walking past               - on
    No weapon warnings when walking past        - on
    No reaction to a raised weapon              - off
  Other:
    Debug log                                   - off

COMPATIBILITY
- Doesn't replace any game script. Adds one sound section (dlg_greet_hello) that uses the
  vanilla greeting voice files, so voice packs that replace those files are used too.
- Walking-past muting filters the vanilla greeting sound (meet_hello) and, if enabled, the
  weapon warnings (meet_wait, meet_hide_weapon, meet_stop) wherever they are played.
  It also mutes three Rostok guards (Zhorik on the stairs to the bar, Garik at the bar door,
  the Duty guard on the hangar catwalk), whose line played every time you passed them.
  The Rostok informant keeps his line, but says it only once per visit.

PERFORMANCE
No per-frame work: one check when a dialog opens and one table lookup when a stalker
starts a voice line.

VERSION HISTORY
1.0.4 - New MCM option "No reaction to a raised weapon" (off by default): stalkers don't
        aim at you or stop you when your weapon is drawn. Applies at once to stalkers nearby.
1.0.3 - Fixed: opening a dialog sometimes played the greeting of another stalker nearby.
        The Rostok informant says his line only once per visit to Rostok (with "No greeting
        when walking past" on); it plays again after a level change or a save load.
1.0.2 - The Duty guard on the Rostok hangar catwalk no longer says "go your way, stalker"
        while you are near him (with "No greeting when walking past" on).
        Debug log also names every voice line of stalkers within 20 m.
1.0.1 - The Rostok guards Zhorik and Garik no longer repeat their line every time you pass
        them (with "No greeting when walking past" on).
1.0.0 - first public release (no code changes).
0.1.1 - "No weapon warnings when walking past" is now on by default.
0.1.0 - first version.

CREDITS
Author: Expand.
