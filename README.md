# Space Vikings Resurrected

A fan recreation, in Godot 4, of *Space Vikings*, the 1982 Apple II space war by Mitchell Robbins
(published by SubLOGIC). You command one ship in a galaxy of twenty star systems: fly it, fight the
defender and its fighters, bombard the planet, land troops, take the surrender, loot it, and go on
to the next. There is an economy under it (loot to sell, repairs to pay for, troops to enlist, bases
to build) and a crew whose morale you can wreck.

It is not the original and is not affiliated with its authors. The original is © 1982 Mitchell
Robbins, and its 3D graphics package, A2-3D1, is © 1979 SubLOGIC Company. To meet the real thing,
boot its disk in an emulator; this project exists because we wanted to understand how it was made,
and to play it again in a modern picture. [`APPRECIATION.md`](APPRECIATION.md) is for that.

## Play it

You need [Godot 4.7](https://godotengine.org/):

    godot --path godot-remake

It opens on a title screen: 1) NEW GAME, 2) CONTINUE (when a game has been saved), 3) QUIT, by keys,
arrows and Enter, or the mouse. In the flight, F1 shows the keys. The short version: the stick turns
the ship (arrows, mouse or a joystick) and 1 to 4 set the speed; Space fires; W switches missile and
laser, S the shields, B the battle-station condition (green, blue, red), A the autopilot; C opens the
command console, G the galaxy map, R the radar, H the hyperdrive. Escape goes back a screen in any
menu.

## What it keeps and what it changes

**The rules are the original's.** Every rule (flight, combat, the hyperdrive, the command computer,
ground forces, shore leave, the reports, saving) is its own class in
`godot-remake/scripts/rules/`, held to recordings of the real game running in an emulator, and
`tools/witness.py` checks that every line of the original's BASIC and machine code is either cited
by a rule or waived with a reason, so that nothing is quietly skipped.

**The picture and the feel are new.** One planet, one flight over it, from orbit to the ground with
no scene change: a lit and clouded sphere with its ground drawn on it, cities on the land, a CRT
console, a vector radar and galaxy map, a damage panel and switches, procedural sound. The places
where it parts from the original on purpose are listed in
[`godot-remake/README.md`](godot-remake/README.md) ("Liberties").

## How it was made

Starting from a cracked disk image, a photocopy of the A2-3D1 manual, and the game's own manual:
the BASIC programs were detokenised, every machine-code binary disassembled and decompiled, the real
disk run in an emulator and recorded, and the engine traced end to end. The findings are written
down, each claim citing the code it came from:

| | |
|---|---|
| [`A2-3D1_ENGINE_ANALYSIS.md`](A2-3D1_ENGINE_ANALYSIS.md) | how the game does real 3D on a 1 MHz 6502: a bytecode interpreter, a sine table, a clipper, a perspective divide |
| [`COMBAT_VFX_ANALYSIS.md`](COMBAT_VFX_ANALYSIS.md) | the cheaper tricks the fighters, lasers and explosions use instead |
| [`FLIGHT_CONTROL_AND_PROGRESSION_ANALYSIS.md`](FLIGHT_CONTROL_AND_PROGRESSION_ANALYSIS.md), [`GROUND_COMBAT_ANALYSIS.md`](GROUND_COMBAT_ANALYSIS.md), [`GAME_MECHANICS_DOCUMENTATION.md`](GAME_MECHANICS_DOCUMENTATION.md) | the mechanics, from the code |
| [`CALL_SITES.md`](CALL_SITES.md), [`analysis/`](analysis/) | what calls what, memory layout, the live recordings the rules are held to, the witness ledgers |
| [`CATALOG_FINDINGS.md`](CATALOG_FINDINGS.md) | the disk's catalogue and file formats |
| [`rea-disassembly/`](rea-disassembly/), [`detokenized/`](detokenized/), [`extracted/`](extracted/), [`disassembly/`](disassembly/) | the decompiled and extracted code and data |
| [`APPLE_II_PATCH_PLAN.md`](APPLE_II_PATCH_PLAN.md) | a plan, not started, for fixing the original's bugs and speeding its 3D engine |
| [`tools/`](tools/) | the decoders, the emulator scripts, the witness checker |

## What is in the repository

    godot-remake/     the game (Godot 4.7): scripts/rules/ (the original's rules),
                      scripts/flight/ (the flight, the console, the HUD), scripts/world/,
                      assets/, tests/
    analysis/         the research: recordings, ledgers, memory maps
    detokenized/      the game's BASIC programs, readable
    tools/            decoders, emulator scripts, tools/witness.py
    rea-disassembly/  decompiled machine code
    disk_image/, original_uploads/, extracted/
                      the cracked disk and what was taken from it, kept for the research
    modern/           the decoded shapes and the live traces the analyses cite
                      (shapes_json/, a2_3d1_json/, ship_id_json/, web/tools/, web/public/data/debug/)
    HANDOVER.md       the notes this work started from (historical)

## Tests

    godot --headless --path godot-remake res://tests/check_combat_rules.tscn    # one of the rule checks
    godot --headless --fixed-fps 60 --path godot-remake res://tests/drive_com.tscn   # one of the scene drives
    python3 tools/witness.py --check                                             # every line of the original, cited or waived
    gdlint (from gdtoolkit), run in godot-remake                                # lint, settings in .gdlintrc

The whole list, and what each holds the port to, is in
[`godot-remake/README.md`](godot-remake/README.md).

## Where it came from

This started as a fork of [`Plicerin/Space-Vikings-Resurrected`](https://github.com/Plicerin/Space-Vikings-Resurrected),
which did the first extraction and disassembly of the disk and made a first attempt at a remake in
JavaScript. That code, and the early documents that went with it, were superseded and removed from the tree
(they are in the git history); what it decoded is kept in `modern/`. Everything since is here: the
decompiled engine and its analysis, the live recordings, the witness ledgers, and the Godot game.

## Credits

*Space Vikings* is by Mitchell Robbins, whose game's title screen credits "G.M. Robbins", published
by SubLOGIC, Champaign, Illinois, September 1982. A2-3D1 is SubLOGIC's. The disk image was made and
stripped of its copy protection by others in 2017; the manuals are kept by the Internet Archive. This
is an unofficial recreation made in their honour.
