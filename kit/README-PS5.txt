Class of '09 - unofficial PS5 homebrew runtime
Ported by CatsAndGats

WHAT THIS IS
------------
A native PS5 homebrew runtime for Class of '09. It is not a Sony-supported
product and it is not Ren'Py. The runtime interprets bytecode exported from the
copy of the game installed in this folder and draws the game with the PS5's
VideoOut framebuffer, DualSense input, and AudioOut output.

FOLDER CONTENTS
---------------
  eboot.bin                     the signed PS5 application
  sce_sys/param.json            title metadata (title ID PPSA09009)
  sce_sys/icon0.png             launcher icon
  sce_module/libc.prx           clean-room libc runtime used by the app
  assets/script.json            exported game bytecode
  assets/game/                  your copy of the PC game (589 files)
  GAME_FILES.txt                exact list of the 589 game files in assets/game

INSTALL ON THE PS5
------------------
1. Copy the complete PPSA09009 folder, keeping its name, to:

       /data/homebrew/PPSA09009/

   You can use an FTP client on port 2121 or the file manager included with
   your homebrew loader. Keep the folder name and the directory structure.
2. Launch "Class of '09" from ShadowMountPlus or another compatible homebrew
   loader. The loader mounts this folder at /app0.
3. Save files, thumbnails, and discovered endings are written to the title's
   writable /download0/saves area. They are not stored in the app folder.

Do not rename the folder or move eboot.bin out of it: the console mounts the
whole folder, and the runtime reads its data from /app0/assets.

WHICH ORIGINAL GAME FILES ARE USED
----------------------------------
The runtime uses your own legally obtained copy of Class of '09. The exact 589
relative paths are listed in GAME_FILES.txt. Copy them from the PC install's
game/ folder into assets/game/ with the same relative paths, for example:

    PC:      Class of 09\game\images\home nicole.jpg
    PS5:     /data/homebrew/PPSA09009/assets/game/images/home nicole.jpg

The 589 files are 678.4 MB: 304 images, 185 audio files, 17 gui files,
6 fonts, 59 "new message" animation frames, and 18 .webm videos. Caches,
saves, translations, .rpyc files, and unused fonts are deliberately not
included.

The videos are the game's own WebM files (logosplash, opening, percpop and
the 15 ending videos). The runtime decodes them directly with libvpx (VP8)
and libvorbis (Vorbis audio), so no frame folders are generated or needed.

One generated data file is also present:

  assets/script.json              exported by tools/export_script.py

CONTROLS
--------
  Cross           advance / confirm
  Circle          step back one line (pause in menus)
  Triangle        ending tracker (from the main menu, once an ending is found)
  D-pad           navigate
  Options/Start   in-game menu
  Any button      skip the startup logo or a movie

SAVES
-----
36 slots in nine pages of four, with thumbnails. The save/load screens use the
game's own slot artwork. Loading resumes timed pauses and movie transitions
past the wait instead of freezing on them.

LEGAL
-----
Class of '09 and its assets are the property of their developers and publishers.
This folder is for personal use with a copy of the game you own. Do not upload
or redistribute this folder or its assets. The runtime contains no jailbreak,
exploit, or DRM-circumvention code.

Unofficial fan project. Not affiliated with the developers, publishers, or Sony.
