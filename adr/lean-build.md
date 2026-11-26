# Lean build

Owns what MuseScoreLite leaves out, keeps and hides: playback with the piano highlight, the sheet pipeline's converter, and nothing else.

## MUST
#### Switched off through upstream's module options
- Switch off updates, network, cloud, learning, tours, braille, the Muse Sounds store and engine, test automation, VST and audio plugin hosts, the Finder Quick Look extension, translations, and the Guitar Pro, MEI, MNX, BB, BWW, Capella, OVE, MuseData, TablEdit and lyrics import or export modules. Why: none serves playback, the pipeline or the practice video.
- Keep MusicXML, PDF and MIDI import and export, measure positions, notation, playback, the converter and extensions. Why: the pipeline imports MusicXML, renders PDF and positions, runs extensions, and exports MIDI for its chord-playback check.
- Keep the video and audio export modules, and export the owner's practice video through `--view-mode practice`: 3840 by 2160 at 30 frames a second in HEVC through VideoToolbox with AAC, the piano keyboard painted by its own view and lit by the playback layers, the line holding the playback position above the next line in playback order, painted by the notation painter in the High-contrast Black look with the score inverted, the app's own playback cursor, 1 bar of count-in, the metronome through the whole song, and the last frame held 2 seconds. Why: every element is the app's own, so the video looks and sounds exactly like playing along in the app, with no second drawing of anything.
- Encode through the ffmpeg the Mac has installed by Homebrew, which upstream's encoder finds in `/opt/homebrew/lib` and resolves by version; bundle none. Why: the encoder is upstream's own and loads ffmpeg at run time, and a bundled copy would only add size to every other use of the app.
- Remove every QML import, registration and page of a module in the same change that switches it off. Why: QML resolves imports only at launch, so a switched-off module still imported builds cleanly and opens no window.

#### Hidden or removed surfaces
- Carry no home page and no page tabs: every start, and closing the last score, lands on the score page, and the app reopens the last score on launch. Why: the File menu's New, Open and Open Recent cover the home page.
- Carry no cloud save, online publishing, plugins, diagnostics, update check or online help in the menus; Help keeps About and the factory reset. Why: the factory reset recovers a broken settings file.
- Save to the computer without asking where. Why: the cloud choice is the only other answer.
- Carry no surface that reaches the internet, removed rather than disabled: no "Get more sounds" or "Get more effects" link in the mixer, no online, cloud, publishing, sharing, help, handbook, accessibility-statement or file-conversion action registered at all (so none appears in shortcuts either), no links in the About dialog, and no language-download, welcome-screen or update-check section in preferences. Why: the app is offline by design, and a dead control is still clutter.
- List every installed SoundFont's presets in 1 flat menu, under a large bold header per font fitted to its text, with no automatic choice row. Why: 1 click sets a sound.
- Carry no online sounds, cloud save or open, publishing, upload, file-to-score conversion, welcome dialog, first-launch setup or update prompt in the code, and no video, MIDI mapping or braille preference page. Why: each serves only the internet or a feature the app does not carry.
- Open with no welcome dialog, tours or account prompt. Why: the owner opens a score and plays it.

#### Offline
- Refuse every non-local request in the app's network manager, send every non-local QML request to an empty local answer, and open no web link. Why: hidden modules still call out at launch; refusing at the source stops every caller at once.

#### Look and name
- Open in the High-contrast Black theme with accent `#5E5E5E`, text and icons `#FFFFFF` and border `#212121`, not following the system theme, with score colours inverted. Why: the owner's own preference, applied on first launch.
- Open the score page as the sheet alone: every toolbar, the palettes, instruments and properties panels, and the status bar start hidden and come back from the View menu. Why: the whole screen reads the sheet.
- Show the piano keyboard panel when the score starts playing, docked alone at the top, the whole keyboard fitted to the window's width and the panel to the keyboard's height, and hide it when playback stops only if playback opened it. Why: the keys matter while sound plays, and a keyboard the owner opened stays where they want it.
- Light every lit key in a layer colour and never in the theme's grey: a key of a selected note in its staff's colour, the owner's own notes in theirs whether or not the score plays. Why: grey says nothing about which part a key belongs to.
- Light each sounding key in the colour of the highest layer playing it: the owner's live notes first, then staff 1, staff 2 and every later staff in order, then the chord track. Why: the owner sees at a glance which part plays which key, and their own playing above all.
- Take the layer colours, in order, from Okabe–Ito and Kelly's contrast palettes, keeping only colours of relative luminance 0.2 or more and high saturation, ordered so that no 2 neighbours share a hue family. Why: a lit key must read on a black key and on a white one, and neighbouring layers must never look alike.
- Draw the app under a merged, untitled title bar through Qt's expanded client area, with the window's automatic safe-area padding removed. Why: the title strip only repeated the score's name.
- Title the bold application menu at the left of the macOS menu bar with the open score's file name, without its extension, and with the app's name when no score is open. Why: the untitled window still says which score is open, where the eye lands.
- Use the system font for the interface, which upstream already does on macOS. Why: the owner wants no other interface font.
- Name the app MuseScoreLite with the bundle identifier `org.musescore.MuseScoreLite`, and keep its settings and saved layout under that bundle identifier, so every copy with its own identifier keeps its own. Why: the official MuseScore, the pipeline's copy and a test copy never read or write each other's settings.
- Fill the screen's visible area when the window first opens, and keep a standard Window menu holding Fill, the tiling entries and Full Screen. Why: the owner fills the screen most, and the menu makes fn+Ctrl+F work.
- Show no score tab strip, and hide invisible elements and layout marks once per opened score. Why: the sheet reads as it prints.
- Open every toggleable panel and toolbar but the piano keyboard floating, sized to its own content and kept inside the screen's visible area. Why: the sheet keeps the whole window, a panel shows only while it is wanted, and a panel that grows past the screen's edge hides its own content.
- Give the mixer only the sound picker, the volume fader and the mute and solo toggles per track, with no audio effects, aux sends or pan. Why: the owner picks a sound and sets a level, nothing more.
- Default the piano keyboard to 61 keys. Why: the owner's keyboard range, and wider keys at the same width.
- Run every command-line mode with no Dock icon. Why: the pipeline launches many converter processes.

#### Data
- Keep every piece of the app's data and the user's default folders directly in `~/mslite`, apart from any other MuseScore. Why: a shared folder let another MuseScore's saved layout override these defaults.
- Make a reset remove only the app's own named files there (logs, layout, session, recent files, caches, extensions) and never anything else. Why: the same folder holds the user's soundfonts and scores.

#### Bundled resources
- Sound live input, a MIDI keyboard or a clicked key, while the score plays, on the playback timeline, and default the audio buffer to 128 samples with no larger reserve during playback. Why: the owner plays along, and upstream mutes live notes during playback and buffers about 20 ms ahead.
- Play every track, the metronome included, only on the sound the owner picks from an installed soundfont, and leave a track with no pick silent. Why: the owner tries several soundfonts, so no default may choose for them.
- Remember the owner's last pick per staff position (staff 1, staff 2, and so on, whatever the instrument), and per the metronome and the chord track in a slot each, in `~/mslite`, and give it to every track whose score saves no pick, every new score included. Why: the owner sets a song's sounds once and every later song opens playing them.
- Let a pick a score saves win over the remembered one for that score, and leave a track whose position has no remembered pick silent. Why: every sound stays one the owner chose, and nothing is guessed.
- Treat what the audio engine applies while a score's tracks are added (a remembered pick, a saved sound absent from the installed soundfonts, output and aux settings) as no edit of the score, so opening a score never asks the owner to save it on close. Why: the remembered sound applies to every score, so a save prompt would follow every song.
- Reach the silent state through a silent synthesizer, never through a failed lookup. Why: a track with no pick keeps its mixer row and logs no error or assert.
- Carry no default sound at all: a pick the installed soundfonts lack goes straight to the silent synthesizer, with no fallback lookup of any other sound. Why: a default would choose for the owner, and one that is not bundled only adds a failed lookup and an error line.
- Bundle no soundfont, no translations, plugins folder, test-automation scripts, alternate text fonts, braille tables, score templates, wallpapers or built-in extensions, and strip debug symbols. Why: the owner adds their own soundfont, and the rest serves nothing kept.
- Keep the Leland, Leland Text, Edwin, FreeSerif, FreeSans, tablature and figured-bass fonts. Why: scores print with them.

## NEVER
- Never switch off a module while any QML still imports it. Cost: the app runs with no window and every build-time check passes.
- Never switch off an export format the pipeline asks for. Cost: the pipeline stops at its first song, after the release already shipped.

## Pitfall
- A switched-off braille module broke the score view's QML (`module "MuseScore.Braille" is not installed`) while the build, the command line and the pipeline all passed; only the window check sees this class.
- The lean build first dropped MIDI export, which only the pipeline's chord-playback check uses; the export check exists for this class.

## Pairs with
- `patches.md` — the `debloat` patch carries all of this.
- `release.md` — the gates that hold these decisions.
