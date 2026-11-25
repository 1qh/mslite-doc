# Lean build

Owns what MuseScoreLite leaves out, keeps and hides: playback with the piano highlight, the sheet pipeline's converter, and nothing else.

## MUST
#### Switched off through upstream's module options
- Switch off updates, test automation, VST and audio plugin hosts, the Finder Quick Look extension, translations, the Muse Sounds engine, and the Guitar Pro, MEI, MNX, BB, BWW, Capella, OVE, MuseData, TablEdit, lyrics, video and audio import or export modules. Why: none serves playback or the pipeline.
- Keep MusicXML, PDF and MIDI import and export, measure positions, notation, playback, the converter and extensions. Why: the pipeline imports MusicXML, renders PDF and positions, runs extensions, and exports MIDI for its chord-playback check.
- Keep compiled every module whose QML another module imports — braille, learning, tours, cloud and the Muse Sounds store — and hide its surfaces instead. Why: QML resolves imports only at launch, so switching such a module off builds cleanly and opens no window.

#### Hidden or removed surfaces
- Show only Home and Score in the top bar, only scores on the home page, and no online scores tab or score manager. Why: the rest are online or learning surfaces.
- Carry no cloud save, online publishing, plugins, diagnostics, update check or online help in the menus; Help keeps About and the factory reset. Why: the factory reset recovers a broken settings file.
- Save to the computer without asking where. Why: the cloud choice is the only other answer.
- Show no cloud save, publishing, video, MIDI mapping or braille preference page, and no online-sounds status or preferences. Why: each sets a feature the app does not carry.
- Open with no welcome dialog, tours or account prompt. Why: the owner opens a score and plays it.

#### Offline
- Refuse every non-local request in the app's network manager, send every non-local QML request to an empty local answer, and open no web link. Why: hidden modules still call out at launch; refusing at the source stops every caller at once.

#### Look and name
- Open in the High-contrast Black theme with accent `#5E5E5E`, text and icons `#FFFFFF` and border `#212121`, not following the system theme, with score colours inverted, and with the piano keyboard panel open. Why: the owner's own preference, applied on first launch.
- Use the system font for the interface, which upstream already does on macOS. Why: the owner wants no other interface font.
- Name the app MuseScoreLite with the bundle identifier `org.musescore.MuseScoreLite`, and keep upstream's machine-readable name. Why: its settings start fresh, and the data and extension folders the pipeline uses stay put.

#### Bundled resources
- Bundle no soundfont, no translations, plugins folder, test-automation scripts, alternate text fonts, braille tables, score templates, wallpapers or built-in extensions, and strip debug symbols. Why: the owner adds their own soundfont, and the rest serves nothing kept.
- Keep the Leland, Leland Text, Edwin, FreeSerif, FreeSans, tablature and figured-bass fonts. Why: scores print with them.

## NEVER
- Never switch off a module another module's QML imports. Cost: the app runs with no window and every build-time check passes.
- Never switch off an export format the pipeline asks for. Cost: the pipeline stops at its first song, after the release already shipped.

## Pitfall
- A switched-off braille module broke the score view's QML (`module "MuseScore.Braille" is not installed`) while the build, the command line and the pipeline all passed; only the window check sees this class.
- The lean build first dropped MIDI export, which only the pipeline's chord-playback check uses; the export check exists for this class.

## Pairs with
- `patches.md` — the `debloat` patch carries all of this.
- `release.md` — the gates that hold these decisions.
