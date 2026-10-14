# The test backdoor (spec to hand the builder)

A test mode that lets an agent reach and inspect any moment of the game in seconds,
without a human playing. Build it early; every later feature plugs into it.

## Flags (adapt names to the project)

| Flag | Does |
|---|---|
| `-agent` | test mode on. Silent (sound cues are logged, not played), never writes the player's save/score files unless a scratch path is given |
| `-scene <id>` | start directly in a scene/state (`title`, `day.desk`, `night.2`, `ending.forever`, `replay.coffee`, …). Keep a scene table in Architecture |
| `-line N` | for scripted scenes: fast-forward to script line N (answering menus with the first option, or one whose flags are preset) and stop on the first line, hold or menu at or after N |
| `-flags a,b` / `-seed N` / `-day N` | preset story flags, RNG seed, progress |
| `-input file` | run a script: `wait N`, `next`, `choose N`, `mouse X Y`, `click X Y`, `key Name`, `shot out.png`, `shot4 out.png`, `dump out.json` |
| `-frames N` | stop after N frames |
| `-shot` / `-shot4` / `-dump` | output at the end of the run (1× / ×4 PNG, JSON state) |
| `-quit` | exit when the script or frames end |
| `-reduce-flashing`, `-fullscreen`, … | accessibility/display options, so they can be tested too |

## The dump

JSON with everything a tester would otherwise have to guess from pixels: scene id,
script line, background, visible characters/layers, active effects, timers, menu
state, flags, score, music cue + gain, the sound log (cue → file), and per-feature
blocks (gallery selection, ending phase, …). Every new feature adds its fields.

## Rules

- Deterministic: seeded RNG, fixed tick rate, no wall-clock reads in test mode.
- Key names in input scripts are checked against the keys the game reads (typos
  fail loudly).
- Every new scene, state or effect is reachable here **in the same commit** that
  adds it.
- A separate **normal-mode** jump (e.g. `-play <scene>`) is useful for the user to
  *hear* or feel a moment with sound on; keep it apart from `-agent`.
