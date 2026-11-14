# Changelog - Dialog Greetings by Expand

- 0.1.1: MCM "No weapon warnings when walking past" is now on by default (stalkers no longer tell you to put your
  weapon away or to stop when you walk up to them). A value already saved in MCM stays as it is.
- 0.1.0: first version. When you open a dialog with a stalker he says one of his greeting lines (own sound section
  `dlg_greet_hello`: vanilla `meet_hello` voice files, no shared cooldown). Hostile, wounded and voiceless NPCs
  stay silent; companions only with the MCM option. Per-stalker cooldown (default 60 s) and chance. Optional
  muting of the walking-past greeting (on by default) and of the walking-past weapon warnings (off by default).
  No per-frame work, no replaced game scripts.
