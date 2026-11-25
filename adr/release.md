# Release

Owns where MuseScoreLite is built, the gates every build passes, and how it is signed and published.

## MUST
#### Where it is built
- Build at upstream's newest release tag, against the Qt version upstream's own macOS build names in `buildscripts/ci/macos/install_qt.sh`. Why: upstream's QML compiler output for 1 Qt version does not compile against another; trigger to revisit: every upstream release.
- Compile on the owner's idle second Mac with `release.sh` in build-only mode, for Apple Silicon only. Why: the compile is the heavy step, and no hosted builder costs anything.
- Sign, notarize, package and publish on the development Mac only, fetching the built app from the compile Mac. Why: the signing identity and API key stay on 1 Mac.
- Rebuild from a fresh clone only when a patch differs from the copy the build tree carries, and otherwise compile incrementally. Why: a full compile costs most of an hour; a changed patch is what makes the tree stale.
- Name every module option the lean build changes or keeps, and each option derived from it such as its QML, ON or OFF, on every build. Why: CMake keeps a cached value, so dropping an option leaves its last setting in force.
- Point the built app's library search at the packaging Mac's own Qt and dependencies, then re-sign it, before any check runs it. Why: the compile Mac's paths are baked in, and an edited binary's signature is invalid.

#### Gates, before and after packaging
- Launch the app on a score and refuse it when its log reports the main window failed to load, and when another copy is running. Why: a missing QML module leaves a windowless app; a running copy takes the score.
- Watch the app's sockets for 45 seconds and refuse any internet socket. Why: the app is offline by design, and a hidden module's request is invisible on screen.
- Export PDF, measure positions, MIDI, MusicXML and a MuseScore file, and refuse any missing or empty. Why: a switched-off module fails only when the pipeline asks for its format.
- Run all 3 gates again on the packaged app. Why: packaging removes Qt parts and can break what ran before it.

#### Signing and publishing
- Sign with the owner's Developer ID Application certificate and notarize with the App Store Connect API key, both read from the owner's secrets folder, then staple the ticket. Why: a notarized, stapled app opens on any of the owner's Macs, offline included.
- Publish each build as a GitHub Release of `1qh/MuseScore`, tagged `<upstream tag>-muse`, with `MuseScoreLite-<version>.dmg`, and push the generated `release/<tag>` branch. Why: the other Macs install from 1 download page.
- Replace a release's disk image in place when a fix ships for the same upstream tag. Why: the download link stays the same.

## NEVER
- Never store a signing certificate or API key in the fork, in `mslite`, or in a hosted builder. Cost: the fork is public, and every credential stays on 1 Mac.
- Never commit on the fork under any identity but the owner's own git identity. Cost: a made-up name is a real GitHub user, credited with the owner's work.
- Never judge Gatekeeper before notarizing. Cost: macOS rejects every not-yet-notarized app, so packaging always fails.

## Pitfall
- The link to the compile Mac is slow, so the app comes back by `rsync`, which sends only changed files after the first copy.
- MuseScore hands a file to an already-running copy of itself and exits, so a check launching beside a running copy reads that copy's old log; the window check refuses when a copy runs.
- `install_name_tool` invalidates the code signature, and macOS then kills the app at launch with no log.
- A killed MuseScore marks its session unclean, and the next launch becomes a crash recovery that asks to restore the session and ignores the score passed in; every check and test quits the app through `quit-app.sh`, never `kill`.
- A shell reads its script as it runs, so updating `mslite` on the compile Mac while `release.sh` runs there can splice the running build; update only between builds.
- MuseScore's derived CMake options (such as a module's QML) keep their first cached value, so `release.sh` refuses a build with a module on and its QML off.

## Pairs with
- `runbook/release.md` — the commands.
- `lean-build.md` — what the gates protect.
