# Release runbook

The commands that turn the patches in `mslite` into a published MuseScoreLite, for a new upstream release or a patch change.

## Steps
1. On the compile Mac, with `mslite` beside the build tree: `MSCORE_BUILD_ONLY=1 MSCORE_TAG=<upstream tag> sh mslite/release.sh`. It clones and patches afresh only when a patch changed, then compiles.
2. On the development Mac: `MSCORE_PACKAGE_ONLY=1 MSCORE_APP_FROM=<compile Mac>:mscore-release/applebuild/mscore.app MSCORE_TAG=<upstream tag> MSCORE_SECRETS=<secrets folder> MSCORE_CHECK_SCORE=<any .mscz> sh mslite/release.sh`. It fetches the app, runs the 3 gates, signs, notarizes, staples, runs the gates again, and publishes, replacing the disk image and notes of a release that already exists for the tag.
3. To package without publishing, add `MSCORE_SKIP_PUBLISH=1` to step 2; the disk image lands in `mscore-release/applebuild/`.

## Checks on their own
- `sh mslite/window-check.sh <mscore> <score>`, `sh mslite/net-check.sh <mscore> <score>` and `sh mslite/export-check.sh <mscore> <score>` each print `ok` or name what failed.
- `python mslite/prove.py` and `python mslite/prove_extension.py` compare served renders and extension runs with fresh launches, layout for layout.
- The sheet pipeline's suite in `muse`, run with `MUSE_MSCORE` naming the built `mscore`, proves the app still serves the pipeline.
