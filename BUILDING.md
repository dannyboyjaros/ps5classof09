# Building and deploying

The application is built with the PS5 payload SDK and a native tool that signs
the produced ELF into a folder-style `eboot.bin`. The build is intended for
WSL2/Linux.

## 1. Requirements

- WSL2 or Linux with `bash`, `make`, `cmake`, `ninja`, `python3`, `clang-18`
- The PS5 payload SDK (the build script downloads it to `kit/.deps/native/`)
- The copy of Class of '09 you own (for the assets only)

## 2. Generate `script.json`

The exporter must run with Ren'Py's bundled Python 2.7, because it unpickles the
game's `.rpyc` files:

```bash
cd <this repository>/tools
"<path to Class of 09>/lib/py2-windows-x86_64/python.exe" export_script.py \
    "<path to Class of 09>/game"
```

This writes `build/script.json`. Copy it to `kit/assets/script.json`:

```bash
mkdir -p kit/assets
cp build/script.json kit/assets/script.json
```

Verify the export (no unreachable story labels is the expected result):

```bash
python3 tools/check_reachability.py kit/assets/script.json
python3 tools/validate_export.py kit/assets/script.json   # run from the repo root
```

## 3. Copy the game files

Copy every path listed in `GAME_FILES.txt` from your game's `game/` folder into
`kit/assets/game/`, preserving relative paths. For example:

```
Class of 09\game\images\home nicole.jpg
  -> kit/assets/game/images/home nicole.jpg
```

## 4. Extract the videos

See `MOVIES.md`. The runtime cannot decode VP8, so each video is converted to a
folder of JPEG frames plus a manifest. The four trees are:

```
kit/assets/videos/logosplash/   252 frames + audio.mp3
kit/assets/videos/opening/      3363 frames + audio.mp3
kit/assets/videos/percpop/      142 frames
kit/assets/videos/endings/<id>/ frames + audio.mp3 for the 15 endings
```

## 5. Build

```bash
cd kit
bash build_class09.sh
```

The output is `kit/dist/PPSA09009/`:

```
eboot.bin
sce_sys/param.json
sce_sys/icon0.png
sce_module/libc.prx
assets/...
```

Deploy the whole folder to `/data/homebrew/PPSA09009/` on the console (FTP on
port 2121 or the loader's file manager), then launch it from ShadowMountPlus or
another compatible homebrew loader.

Saves, ending data, and screenshots live in the title's writable
`/download0/saves` area, not in the app folder.

## Incremental engine builds

After the first full build, only `engine.c` changes can be rebuilt and signed
without copying the assets again:

```bash
cd kit
ninja -f build/app.ninja build/llvm-pie.elf build/eboot.elf
build/host/ps5-native-tool self --sign --in build/eboot.elf \
    --out dist/PPSA09009/eboot.bin --magic 0x1D3D154F
```
