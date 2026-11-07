# Class of '09 — unofficial PS5 homebrew runtime

A native PS5 homebrew runtime for **Class of '09**. It is not a Sony-supported
product and it is not Ren'Py. The game's `.rpyc` scripts are decompiled once into
a compact bytecode file, and a small C runtime interprets that bytecode on the
console using the PS5's VideoOut, Pad, AudioOut, and Gpu services.

- Title ID: `PPSA09009`
- Content ID: `UP0000-PPSA09009_00-CLASS0909PS5HOME`
- Launcher icon: `icon0.png`
- Runtime source: `kit/src/`
- Script exporter: `tools/export_script.py`

This repository contains **source code only**. It does not contain the game's
assets. You must own Class of '09 and generate the data files from your own copy;
see `BUILDING.md` and `MOVIES.md`.

## What works

- Full script export with every branch menu and ending reachable
  (`tools/check_reachability.py` reports 0 unreachable labels).
- Tiled 1080p scanout on real PS5 hardware, DualSense input, 48 kHz audio.
- Main menu with the original art, menu music, and an ending tracker phone in
  the bottom-right that opens with the D-pad (or Triangle).
- Opening movie on New Game, the perc-pop animation after taking the pills, and
  all fifteen `*end.webm` ending videos, each with its soundtrack.
- Rollback: Circle steps back one dialogue line; Options opens the pause menu.
- 36 save slots with thumbnails, persistent ending collection, and the
  `NewMessageSequence` animation when a new ending is unlocked.

## Repository layout

```
kit/                  the PS5 application (C sources, build system, sce_sys)
kit/src/engine.c      bytecode interpreter, UI, saves, ending tracker
kit/src/ps5_*.c       PS5 backends: video, input, audio, fonts, images, trace
kit/tools/            build, deploy and conversion scripts
kit/sce_sys/          param.json + icon0.png
tools/export_script.py    .rpyc -> script.json exporter (Python 2.7)
tools/check_reachability.py
tools/validate_export.py
GAME_FILES.txt        exact list of game files to copy into assets/game
MOVIES.md             how the videos were extracted
BUILDING.md           build and deploy instructions
```

## Controls

| Input | Action |
| --- | --- |
| Cross | advance / confirm |
| Circle | step back one line (rollback) |
| Triangle | ending tracker on the main menu |
| D-pad | navigate |
| Options / Start | pause menu |
| any button | skip the logo or intro (ending videos always play out) |

## Credits

Ported by **CatsAndGats**. Class of '09 and its assets are the property of their
developers and publishers. This is an unofficial fan project, not affiliated
with the developers, publishers, or Sony. Do not redistribute the game assets.
