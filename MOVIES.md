# Videos

The PS5 runtime decodes the game's original `.webm` files directly. There is no
frame-extraction step and no `assets/videos/` tree to generate.

## How it plays

- The container is demuxed by `kit/src/webm.c` (a small EBML/Matroska parser
  that handles Segment, Info, Tracks, Cluster, SimpleBlock/Block and Xiph
  lacing).
- VP8 video is decoded with **libvpx**; Vorbis audio with **libvorbis** and
  **libogg**.
- Frames are decoded on a dedicated thread for the current playback timestamp.
  The thread publishes each finished RGBA frame into a double buffer, and the
  render thread only copies the newest frame into a texture, so the menu and
  dialogue never block on VP8 decoding.
- The decoder skips intermediate frames: it decodes only the blocks needed to
  reach the requested timestamp and converts just the final frame to RGBA.
- Audio is decoded once at the start of playback to a temporary WAV in the save
  directory (`video_audio.wav`), loaded with `Mix_LoadMUS`, and the WAV is
  deleted. That keeps the mixer on its existing MP3/WAV path.

The libraries come from the PacBrew ports prefix. `kit/build_class09.sh` selects
them automatically:

```sh
PACBREW_PACKAGES='vpx vorbis ogg'
```

`tools/setup-pacbrew-dependencies.sh` downloads the pinned prefix (about 346 MB)
on the first build and caches it in `kit/.deps/pacbrew`.

## Files the runtime uses

These must be in `assets/game/` (they are listed in `GAME_FILES.txt`):

```
logosplash.webm      startup logo (8.6 s, with audio)
opening.webm         New Game intro (140 s, with audio)
percpop.webm         perc-pop overlay (5.9 s, silent)
0039end.webm         ending 0039   (with audio)
0063end.webm         ending 0063
0074end.webm         ending 0074
0091end.webm         ending 0091
0103end.webm         ending 0103
0105end.webm         ending 0105
0121end.webm         ending 0121
0126end.webm         ending 0126
0127end.webm         ending 0127
0144end.webm         ending 0144
0145end.webm         ending 0145
0153end.webm         ending 0153
0161end.webm         ending 0161
0169end.webm         ending 0169
0173end.webm         ending 0173
```

The ending videos are opened by ID: the script's `ending` op for `0039` opens
`<game_dir>/0039end.webm`.

## Optional legacy fallback

If a `.webm` file is missing, the engine falls back to the older frame-sequence
player and looks for `assets/videos/<name>/` with a `manifest.txt` and
`frame_0000.jpg` (or `assets/videos/endings/<id>/`). That fallback is only for
old deployments; a normal install needs no video folders at all.

## Performance

VP8 is decoded in software on the PS5 CPU. Decoding runs on its own thread and
only publishes finished frames, so a 1080p cutscene decodes at roughly 18–25
fps while the game itself keeps running at 59.9 fps. The perc-pop overlay is
short and silent, and ending videos are left unskippable so they always play
through.
