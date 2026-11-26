# Release runbook

The commands that turn the patches in `mslite` into a published MuseScoreLite, for a new upstream release or a patch change.

## Steps
1. On the compile Mac, with `mslite` beside the build tree: `MSCORE_BUILD_ONLY=1 sh mslite/release.sh`. It clones and patches afresh only when a patch changed, then compiles.
2. On the development Mac: `MSCORE_PACKAGE_ONLY=1 MSCORE_APP_FROM=<compile Mac>:mscore-release/applebuild/mscore.app MSCORE_SECRETS=<secrets folder> MSCORE_CHECK_SCORE=<any .mscz> sh mslite/release.sh`. It fetches the app, runs the 4 gates on a test copy (window, network, export, fresh launch), signs, notarizes, staples, runs the gates again, and publishes, replacing the disk image and notes of a release that already exists for the tag. The compile Mac's step 1 also refuses a dead QML import, an assignment to a positioner's implicit size, and any unit-test failure outside `mslite/upstream-failing-tests.txt`.
3. On 1 Mac, compile and publish in 1 run: `MSCORE_SECRETS=<secrets folder> MSCORE_CHECK_SCORE=<any .mscz> sh mslite/release.sh`.
4. Schedule the daily check once per Mac: `sh mslite/schedule.sh <secrets folder> <any .mscz>` installs the LaunchAgent `org.mslite.auto-release`, which runs `mslite/auto-release.sh` at 04:00, logging to `~/Library/Logs/mslite-release`; it releases when the release branch head differs from the commit the current release's notes name, and raises a notification on any failure.
5. Install the released disk image into Applications, then copy it for the sheet pipeline with `sh mslite/pipeline-copy.sh <installed .app> <pipeline .app>`, which gives the copy its own bundle identifier.
6. To package without publishing, add `MSCORE_SKIP_PUBLISH=1` to step 2; the disk image lands in the build tree's `applebuild` folder.

## Checks on their own
Every check launches only a test copy under `org.musescore.MuseScoreLite.test` (made by `sh mslite/pipeline-copy.sh <app> <copy> org.musescore.MuseScoreLite.test`), whose settings `tools/test-app.sh` wipes first, with its own empty home and temp folder, so none reads the owner's settings or data or reaches the owner's open app.
- `sh mslite/tools/stage-app.sh <compile Mac>:mscore-release/applebuild/mscore.app <staging folder> <Qt folder> <deps folder>` turns the compile Mac's installed build into a runnable test copy here, without packaging; never the build tree's app, which lacks its QML modules.
- `sh mslite/window-check.sh <mscore> <score>`, `sh mslite/net-check.sh <mscore> <score>` and `sh mslite/export-check.sh <mscore> <score>` each print `ok` or name what failed; `sh mslite/tools/fresh-launch.sh <mscore> <score>` refuses any window beside the score or an ignored quit.
- `DRIVE_PANELS="Palettes,Layout,Properties,Selection filter,History,Navigator,Timeline,Mixer,Percussion" sh mslite/tools/drive-play.sh <mscore> <score> <folder> [seconds]` opens those View panels, plays the score and saves every on-screen window; read each capture for a scroll bar, a clipped row or an empty band.
- `sh mslite/tools/menu-title.sh <mscore> <launches> <score>...` reads the application menu's title 4 times per launch and refuses any that is not the score's name.
- `python3 mslite/tools/metronome-pick.py <mscore> <score>` and `python3 mslite/tools/remember-probe.py <mscore> <score>` prove a saved metronome pick and a remembered staff pick reach the audio engine.
- `sh mslite/tools/isolation-check.sh <mscore> <score>` proves a test launch cannot reach a running copy, against a listener it starts itself.
- `sh mslite/tools/close-reopen.sh <app> <score>` closes the window with its red button and reopens on the same data folder.
- `sh mslite/tools/unit-tests.sh <source tree> <test build folder> <Qt folder> <deps folder>` on the compile Mac builds and runs the patched modules' tests; `sh mslite/tools/plant.sh` plants a defect in 1 source line, rebuilds 1 target and restores it, the mutation control for a test.
- `python3 mslite/tools/palette.py` derives the key layer colours from luminance, saturation and hue.
- `python mslite/prove.py` and `python mslite/prove_extension.py` compare served renders and extension runs with fresh launches, layout for layout.
- The sheet pipeline's suite in `muse`, run with `MUSE_MSCORE` naming the pipeline copy, proves the app still serves the pipeline.
