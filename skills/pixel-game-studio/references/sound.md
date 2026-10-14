# Sound: sourcing, auditioning, processing

## Sourcing (builder)

- **Licences:** CC0 first (OpenGameArt has plenty), CC-BY second (needs a credit
  line in the in-game credits and the README). Avoid licences that forbid
  redistributing raw files if the repo is public (e.g. Sonniss GDC bundles), and be
  careful with Pixabay in a public repo (its licence bans standalone redistribution).
- **Read the licence twice:** on the page and inside the downloaded zip (some packs
  say CC0 on the page and "no reselling" inside). Record source URL, author and
  licence per file in a `README.txt` next to the sounds and in the Sound note.
- **Download blocks:** if a site serves a challenge page or blocks automated
  downloads, don't try to get around it. Skip it, say so, and offer alternatives.
- **Hygiene:** download each item into its own empty scratch folder; check that it
  decodes, has a real length (not 0 s), and the format/sample rate; then copy it in.
  Keep files small (mono for SFX, sane bitrates).
- **Variety beats one perfect file:** for anything that repeats (snores, footsteps,
  page flips), get 3–5 variants, pick at random without immediate repeats, randomise
  the gaps with the seeded RNG, and set volume by context (near/far).

## The AI can't hear

Say it in every sound report. List which files the user should listen to first and
why ("0.4 s snort, not a long snore"; "synthesized buzz"; "measured only"). Every
pick is "default, user confirms by ear" until they do.

## Auditioning (letting the user choose by ear)

1. Put candidate tracks in a gitignored folder (e.g. `assets/sound/audition/`),
   committing only a README with source, licence and cut times.
2. Add a normal-mode flag: `-play <scene> -rock <name>` (or `-track`) plays that
   scene with the candidate swapped in; `-rock list` lists them; `-line N` lands just
   before the moment that matters.
3. Write 30 s preview clips so they can run
   `for f in preview_*; do echo $f; afplay $f; done` (macOS).
4. When they name a timestamp ("try 2:12"), cut a new candidate from the original
   (WAV is easiest: Python's `wave` module, no dependencies) and give them the
   one-line command to try it.
5. Commit the winner: cut it (MP3 at frame boundaries if there is no encoder), make
   sure it outlasts what it plays under (a test with the real file length), credit
   it if CC-BY, and record the pick and the user's words.

**Syncing a drop to a visual beat:** make the sequence run on a fixed timeline
(auto-play, clicks don't skip), compute the time T from the sequence start to the
beat, and start the track at `drop − T` so the drop lands exactly on it.

## Processing (outside the game)

No ffmpeg or sox? Make a scratch venv: `python3 -m venv venv && venv/bin/pip
install numpy scipy`. Typical pass for noisy SFX:

1. Back up the originals.
2. High-pass around 50 Hz (rumble).
3. Spectral gate: noise profile from the quietest ~15% of STFT frames, soft gain
   (duck by about 18 dB), smoothed over time and frequency so it doesn't "chirp".
4. Loudness-normalize all variants to the same RMS of their loud part (about −20
   dBFS), with a soft tanh limiter at −1 dBFS so a few peaks don't hold a file
   down.
5. Report a before/after table (loudness, noise floor). Warn that heavy gating can
   sound "watery", and keep the originals so you can go gentler.
