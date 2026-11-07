# Extracting the videos

The PS5 runtime has no VP8/Vorbis decoder. Every `.webm` used by the game is
converted into a directory of JPEG frames plus a small manifest and, when the
video has sound, an MP3. The player reads the frames in order at the manifest's
frame rate.

A generated video directory looks like this:

```
kit/assets/videos/<name>/
  manifest.txt      fps=24 / frames=N / width=960 / height=540
  frame_0000.jpg
  frame_0001.jpg
  ...
  audio.mp3         (only when the source has audio)
```

The engine plays `logosplash`, `opening`, and `endings/<id>` as full-screen
movies, and `percpop` as a screens-layer overlay. All of them use the same
frame sequence format, so any of them can be regenerated independently.

## Frame extraction

Set `FFMPEG` to an ffmpeg build that can decode VP8 (a full build with libvpx is
easiest), then from the repository root:

```bash
GAME="<path to Class of 09>/game"
OUT=kit/assets/videos

# startup logo: 252 frames at 30 fps, 1920x1080, with sound
ffmpeg -y -i "$GAME/logosplash.webm" \
    -vf "fps=30" -q:v 3 -start_number 0 "$OUT/logosplash/frame_%04d.jpg"
ffmpeg -y -i "$GAME/logosplash.webm" -vn -ac 2 -ar 44100 \
    -c:a libmp3lame -b:a 128k "$OUT/logosplash/audio.mp3"

# New Game intro: 24 fps, 960x540, with sound
ffmpeg -y -i "$GAME/opening.webm" \
    -vf "fps=24,scale=960:540:flags=bicubic" -q:v 65 \
    -start_number 0 "$OUT/opening/frame_%04d.jpg"
ffmpeg -y -i "$GAME/opening.webm" -vn -ac 2 -ar 44100 \
    -c:a libmp3lame -b:a 128k "$OUT/opening/audio.mp3"

# perc-pop overlay: 24 fps, 960x540, silent
ffmpeg -y -i "$GAME/percpop.webm" \
    -vf "fps=24,scale=960:540:flags=bicubic" -q:v 68 \
    -start_number 0 "$OUT/percpop/frame_%04d.jpg"

# all fifteen endings
for f in "$GAME"/*end.webm; do
    id=$(basename "$f" end.webm)
    mkdir -p "$OUT/endings/$id"
    ffmpeg -y -i "$f" -vf "fps=20,scale=960:540:flags=bicubic" -q:v 65 \
        -start_number 0 "$OUT/endings/$id/frame_%04d.jpg"
    ffmpeg -y -i "$f" -vn -ac 2 -ar 44100 -c:a libmp3lame -b:a 128k \
        "$OUT/endings/$id/audio.mp3"
done
```

The startup logo and opening use 30/24 fps because they are short; the endings
use 20 fps to keep the 15 videos compact and to match the software player's
decode budget. The frame sizes (960x540) are upscaled to 1920x1080 by the
renderer.

## Manifests

Create a `manifest.txt` next to the frames with the real counts:

```bash
printf 'fps=24\nframes=%d\nwidth=960\nheight=540\n' \
    "$(ls "$OUT/opening" | grep -c '^frame_.*\.jpg$')" \
    > "$OUT/opening/manifest.txt"
```

Repeat for each directory, using its own frame rate and size (the logosplash
manifest uses 30 fps and 1920x1080). The runtime refuses a directory whose
manifest frame count does not match an existing `frame_0000.jpg` or whose frame
count is out of range.

## Ending IDs

The ending directory names are the four-digit ending IDs from the script:
`0039, 0063, 0074, 0091, 0103, 0105, 0121, 0126, 0127, 0144, 0145, 0153, 0161,
0169, 0173`. The runtime looks up `videos/endings/<id>/` when the script reaches
an `ending` op, and plays the ending text card after the video finishes.
