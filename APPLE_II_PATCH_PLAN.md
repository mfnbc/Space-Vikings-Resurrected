# Plan: fixing and improving the original Apple II game

Status: a plan only, nothing here is started. Written 2026-11-14 as the last item of the research
phase. Every claim about the original cites the document it comes from; anything resting on
reasoning rather than a recording is marked **(unmeasured)**.

## Aim

A patched disk that plays as the original does, minus its bugs, with a faster 3D engine. Nothing
about the rules or the economy changes. The Godot game is the reference for "what the rules are"
(`godot-remake/scripts/rules/`, held to live recordings); the patched disk is checked against the
same recordings.

## Ground rules

1. **Original disk image untouched.** Work on a copy; the patched image is built by a script from
   the original plus a list of patches, so the diff is reviewable and reproducible.
2. **Every patch is a recorded before/after.** Each one is a MAME Lua script (see `tools/mame/`)
   that reproduces the bug on the original and shows it gone on the patched disk.
3. **No change in rules unless it is a listed bug.** The existing recordings in `analysis/live/`
   must still pass on the patched disk (byte-compare, as the Godot tests do).
4. **Layers apart.** BASIC patches, machine-code patches and the engine rewrite are separate
   stages, each shippable on its own.

## Stage 0: the build chain (do first)

- A script that reads the disk's files (`tools/extract_files.py`, `tools/dos33_catalog.py`),
  applies named patches, and writes a new `.dsk`. The catalogue reader exists; a writer does not.
- Tokenised BASIC: patch by editing `detokenized/*.bas` and re-tokenising (`tools/detok.py` goes
  one way only; **a tokeniser is needed**), or patch tokens in place for same-length fixes. In-place
  is safer for the first stage.
- Machine code: patch bytes at known addresses; `tools/dis6502.py --calls` lists entry points and
  the self-modified operands that must not move.
- Test harness: boot in MAME (`apple2p`, setup in the project notes: `cage` headless, Lua frame
  hooks), run each recording, compare.

## Stage 1: BASIC bugs already found

| Bug | Where | Fix | Source |
|---|---|---|---|
| Loot write goes to `31180`, read is `38180` | `COLLECT.bas` line 960 | write to `38180` | `GROUND_COMBAT_ANALYSIS.md` ("A real, verifiable bug") |
| Primitive planet's silver set to the gold sum | `COLLECT.bas` | silver is its own sum | `godot-remake/README.md` ("COLLECT's two slips") |
| Save/load copies 17 bytes of a 16-byte record: SOL's secured flag is overwritten with the next planet's threshold on every load; at planet 20 it reads the byte after the table | the load routine, `CALL_SITES.md` near line 286 | copy 16 bytes | `CALL_SITES.md` |
| A key that is not a number loses the game in hand | the SYSTEM ERROR path (line 63999) | ignore the key | `godot-remake/README.md` ("Liberties") |
| Saved game is good for one boot only | boot/save handling | keep the save until overwritten | same |
| Missile that destroys the defender is not charged (line 1090 never reached because `RUN EX` leaves inside the `GOSUB`) | `STARSHIP SIMULATOR.bas` 1000-1090 | probably leave: it is a favour to the player, and changing it alters the economy. Decide per rule 3. | `CALL_SITES.md` 388 |

The first four are one- or two-token edits and are the best first release.

## Stage 2: bugs not yet found

The analyses were written to port the rules, not to audit the original, so the list above is
whatever tripped the port. Before Stage 3, do one audit pass: `tools/verify_callmap.py` and
`analysis/BASIC_CALL_MAP.md` list undefined GOTO targets, odd PEEK/POKE addresses and every
CALL site. Read those for mismatched read/write addresses (the `31180` pattern) and fall-through
lines. Expect a handful.

Also on the list, from `godot-remake/README.md`:
- Shore-leave repair cost uses Applesoft's `100 - ((D/100)*100)`, off by about 1e-8 for some D, so a
  bill can be a credit out. Fix: integer arithmetic.
- 3D engine's sine table: entry 25 is wrong, negative results are slightly too negative, the factor
  stays unset at two angles, and the cosines read as nothing at 64, 128 and 191. The table is data,
  so this is the cheapest engine fix: regenerate the table and compare.
- The heading turns oddly when the stick holds the nose past its stop.

## Stage 3: the 3D engine (A2-3D1)

The engine (`A2-3D1_ENGINE_ANALYSIS.md`) is a bytecode interpreter: sine-table rotation, perspective
divide, clip, then line plotting on the hi-res page with self-modifying OR/XOR plotters (opcodes 8,
9, 12, confirmed live).

**Step 3a: measure.** Use a MAME Lua cycle counter per routine over several frames of a typical
orbital scene and a ground scene. **Nothing below is measured (unmeasured)**; the order of work
follows whatever the counts say.

Candidate improvements, in the order I expect them to pay off:

1. Multiply: quarter-square table multiply in place of shift-and-add.
2. Perspective divide: reciprocal table and a multiply.
3. Transform each vertex once, not once per edge, if the interpreter currently repeats it.
4. Reject models and edges behind the camera or off-screen before rotating.
5. Line plot: table-driven row addresses, unrolled page-aligned inner loops, keeping the OR/XOR
   self-modification.

Constraints: tables compete for the 48K; the engine patches its own code at fixed addresses, so
any rearrangement must keep those; the game calls it from `SPACE SIMULATOR ASSEMBLY` at fixed
entry points, so entry points and the bytecode format must stay. The Godot port does not use the
engine, so this stage helps only the real machine and emulators.

Decision to make when it comes up: the combat artifacts (undraw residue from the XOR erase) are
authentic. Double-buffering would remove them. Default: keep them.

## Stage 4: release

- Patch list as a readable changelog, one recording per entry.
- Distribute as a patch (IPS/BPS or a script), not as the disk image, since the disk is someone
  else's work.
- Credit as in the README: Mitchell Robbins, SubLOGIC (A2-3D1 © 1979), the crack's authors.

## Open questions

- Whether a tokeniser already exists in the Applesoft tooling elsewhere; if not, write one and test
  it by round-tripping every program on the disk byte-for-byte.
- Whether the disk has free sectors for larger engine tables, or whether the tables must displace
  something.
- Legal posture of distributing patches for a still-copyrighted game: patch files only, as above.
