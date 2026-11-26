# Release

Owns where MuseScoreLite is built, the gates every build passes, and how it is signed and published.

## MUST
#### Where it is built
- Build at the head of upstream's current release branch, the `<major>.<minor>` branch of upstream's newest release tag, against the Qt version upstream's own macOS build names in `buildscripts/ci/macos/install_qt.sh`. Why: that branch is the shipped code plus untagged fixes, so each rebase is a few commits; upstream's QML compiler output for 1 Qt version does not compile against another.
- Release whenever that branch head moves, checked once a day on the development Mac, and move to the next release branch when upstream tags its first release. Why: small, frequent rebases keep the patches applying and the app current.
- Compile on the development Mac, or on the owner's second Mac with `release.sh` in build-only mode, for Apple Silicon only. Why: the compile is the heavy step, and no hosted builder costs anything.
- Sign, notarize, package and publish on the development Mac only, fetching the built app from the compile Mac. Why: the signing identity and API key stay on 1 Mac.
- Rebuild from a fresh clone only when a patch differs from the copy the build tree carries, and otherwise compile incrementally. Why: a full compile costs most of an hour; a changed patch is what makes the tree stale.
- Name every module option the lean build changes or keeps, and each option derived from it such as its QML, ON or OFF, on every build. Why: CMake keeps a cached value, so dropping an option leaves its last setting in force.
- Point the built app's library search at the packaging Mac's own Qt and dependencies, then re-sign it, before any check runs it. Why: the compile Mac's paths are baked in, and an edited binary's signature is invalid.

#### Gates, before and after packaging
- Launch the app on a score and refuse it when its log reports the main window failed to load, and when another copy is running. Why: a missing QML module leaves a windowless app; a running copy takes the score.
- Refuse, before the compile, a relative QML import of a folder no CMake list compiles in. Why: QML resolves imports only at launch, so the import builds cleanly and opens no window.
- Watch the app's sockets for 45 seconds and refuse any internet socket, and refuse a watch that sampled fewer than once every 2 seconds. Why: the app is offline by design, and a loaded machine samples too rarely to see a short request.
- Export PDF, measure positions, MIDI, MusicXML and a MuseScore file, and refuse any missing or empty. Why: a switched-off module fails only when the pipeline asks for its format.
- Refuse, in every gate, an app that ignores its quit for 10 seconds. Why: only a modal dialog holds a quit, and the reading app opens none.
- Launch on an empty data folder, through `tools/fresh-launch.sh`, and refuse any window beside the score or an ignored quit. Why: a first launch opens straight onto the score.
- Refuse, before the compile, an assignment to the implicit size of a Column, Row, Grid or Flow, and on the compile Mac any unit-test failure of the patched modules that the untouched upstream base does not share. Why: QML refuses the first only at launch, and a test that no release runs proves nothing.
- Drive every floating panel before a release: the drive refuses an empty band beside a panel's content and any window reaching past the screen's visible area, and every capture is read for a scroll bar or a clipped row. Why: a capture of 1 window cannot show where it sits, and accessibility cannot see QML scroll bars.
- Refuse, in every drive, any QML `TypeError`, `ReferenceError` or failed assignment in the app's log. Why: a binding naming something absent leaves its control dead while the window looks whole, and these lines are its only trace.
- Click a page in the floating Navigator and refuse the run unless the score view moves, against a 2nd capture taken with no click. Why: a Navigator painted at the wrong scale still draws, while its clicks land on the wrong page.
- Run every gate again on the packaged app. Why: packaging removes Qt parts and can break what ran before it.
- Run every app check on the owner's second Mac over ssh when `MSCORE_CHECK_HOST` names it, against the packaged app only, from that Mac's clone of `mslite` at the releasing commit, and refuse a clone at any other commit. Why: the checks drive menus and keys, which macOS gives only to the frontmost app, so on the releasing Mac they take the owner's screen; the unpackaged app loads Qt from this Mac's own folders and cannot start elsewhere.

#### Codex lanes on the build tree
- Inline these hazards into every lane brief that edits `mscore-release`, since a lane reads no doc outside its prompt: the tree builds with C++17 rules for the test targets (no `contains` on maps or sets, use `find`); a test mock may override only a virtual method; a QML Column, Row, Grid or Flow computes its own implicit size, so never assign it; every new QML import names a folder a CMake list compiles; a lane never builds, runs, tests or commits. Why: each hazard cost a full compile round when a lane met it unbriefed.
- Give a lane that changes shared dock code (`src/framework/dockwindow`) the whole tree alone, and drive every floating panel after it. Why: every panel's window passes through it, so a fix aimed at one panel changes all 9.

#### Signing and publishing
- Sign with the owner's Developer ID Application certificate and notarize with the App Store Connect API key, both read from the owner's secrets folder, then staple the ticket. Why: a notarized, stapled app opens on any of the owner's Macs, offline included.
- Publish each build as the only GitHub Release of `1qh/MuseScore`, named, tagged and filed (`MuseScoreLite-<hash>.dmg`) by the short hash of the upstream commit it is built on, delete every older release and its tag, and force-push the generated tree to the fork's `main`. Why: the app always tracks upstream's newest commit, so the hash is the whole version and 1 download is the only one needed.
- Publish nothing when a patch fails to apply or a gate fails, and raise a notification naming the step. Why: an unattended release must never ship a half-patched app.

## NEVER
- Never run servers for the sheet pipeline from the installed app. Cost: macOS sends a file or Launchpad open to any running app with the same bundle identifier, so a server takes it and the owner gets a Dock icon with no window.
- Never base the fork on upstream's `main`. Cost: it is the next version, thousands of commits away, and refactors daily under the patches.
- Never run the pipeline or any check from the app inside the build tree. Cost: the next release deletes that tree, and the app with it, under a running build.
- Never store a signing certificate or API key in the fork, in `mslite`, or in a hosted builder. Cost: the fork is public, and every credential stays on 1 Mac.
- Never commit on the fork under any identity but the owner's own git identity. Cost: a made-up name is a real GitHub user, credited with the owner's work.
- Never judge Gatekeeper before notarizing. Cost: macOS rejects every not-yet-notarized app, so packaging always fails.

## Pitfall
- `cp` on APFS clones a file and keeps its old modification time, so a source restored from a copy looks older than the object built from the planted line and ninja keeps the planted binary; `plant.sh` touches the restored file.
- Every built or staged app carries the release's own bundle identifier, and macOS opens a double-clicked score in any registered copy, an old staged build included; `release.sh` takes its build-tree copies out of Launch Services, and a staged copy is made under the test identifier only.
- A copy of a quarantined app is signed ad hoc and Gatekeeper refuses to open it, so `pipeline-copy.sh` drops the inherited quarantine.
- A 2nd quit request sent while the 1st is still closing a score raises the save dialog by itself, so a probe that quits with Command-Q and then at once with `terminate` reports a changed score that the owner never sees; quit once, the owner's way, wait for the exit, and only then fall back.
- A score that saves a sound absent from the installed soundfonts opens with the engine applying something else, and the app must not count that as an edit; the metronome probe's saved missing sound is the control that refuses a build asking to save on close.
- Deleting a model's items inside its reset, or before a list property announces its change, leaves the delegates still holding them reading null; every binding on them logs a TypeError, so items are handed to `deleteLater`.
- Moving a QML block out of the component that defined an id leaves its references resolving to whatever the new scope names that id; in the notation page, `notationView` is already the score's paint view.
- A cursor intersected with content that is not laid out yet is empty, and a refit that re-applies the empty result keeps it empty; keep the source rectangle and intersect it again on every refit.
- A private TMPDIR exported to a whole script also reaches osascript, which then loses the app's menu bar; give it to the app's own launch only.
- System Events exposes a menu bar only for the frontmost app, and an app loses its menu bar to accessibility after a floating panel opens until it is brought forward again; bring it forward before every menu click.
- AppKit redraws the application menu with the app's own name when the app becomes active, without changing the menu item's title, so a title equal to the target is set again through an empty one.
- A notation view's matrix is in GUI-scaled units, and its base paint multiplies the GUI scaling (2 on a Retina screen) back in; a view that paints with its own matrix and leaves that factor out draws at half size in a window sized correctly, which reads as a window that never shrank. Print the requested, item, window and painted sizes before changing any sizing code.
- The link to the compile Mac is slow, so the app comes back by `rsync`, which sends only changed files after the first copy.
- MuseScore hands a file to an already-running copy of itself and exits, so a check launching beside a running copy reads that copy's old log; the window check refuses when a copy runs.
- `install_name_tool` invalidates the code signature, and macOS then kills the app at launch with no log.
- A killed MuseScore marks its session unclean, and the next launch becomes a crash recovery that asks to restore the session and ignores the score passed in; every check and test quits the app through `quit-app.sh`, never `kill`.
- A shell reads its script as it runs, so updating `mslite` on the compile Mac while `release.sh` runs there can splice the running build; update only between builds.
- MuseScore's derived CMake options (such as a module's QML) keep their first cached value, so `release.sh` refuses a build with a module on and its QML off.

## Pairs with
- `runbook/release.md` — the commands.
- `lean-build.md` — what the gates protect.
