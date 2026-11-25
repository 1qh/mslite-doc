# Patches

Owns what MuseScoreLite adds to upstream MuseScore: a small set of patches applied to upstream's newest release tag.

## MUST
#### The patch set
- Keep the fork as upstream's newest release tag plus 1 patch file per need in `mslite`, applied in order by `release.sh`. Why: a patch set rebases onto each release, while a diverged branch drifts.
- Treat the patch files in `mslite` as the source, and the fork's `release/<tag>` branch on GitHub as generated output. Why: `release.sh` re-clones upstream and applies the patches, overwriting any commit made on the branch.
- Change the app by editing a clean checkout, then regenerating that patch with `git diff` against the commit before it. Why: the patch is what the next release applies.
- Carry exactly these patches, in this order: `serve`, `piano-playback`, `packaging`, `flat-picker`, `debloat`. Why: each answers a need upstream leaves open; nothing else differs from upstream.
- Drop a patch the moment upstream ships its change. Why: a patch upstream already carries conflicts at the next release.

#### What each patch does
- `serve`: a `--serve` mode reading 1 JSON job per line of standard input (`{"id","job","extension","style"}`), answering `<id> ok` or `<id> failed <reason>`, and staying running. Why: the sheet pipeline keeps a pool of running converters instead of launching one per render.
- `serve`, also: every command-line mode (`--serve`, `-o`, `-j`) runs as a macOS background process, with no Dock icon or menu bar. Why: the pipeline's pool of converters otherwise fills the Dock with icons that open no window.
- `flat-picker`: the mixer's sound slot lists every SoundFont preset as 1 row under a non-clickable header per font, fonts by lowest bank and presets by (bank, program), never by name; the search field has focus on open, the list scrolls to the checked current preset, Enter picks the first match, and search results keep the headers and bare names. Why: 1 click picks any sound, with no hover; shaped to propose upstream.
- `piano-playback`: the piano keyboard panel lights the notes sounding during playback, upstream pull request 34351 fitted to the release. Why: the owner reads playback on the keyboard.
- `packaging`: the macOS packager signs with the owner's Developer ID and notarizes with an App Store Connect API key, skips the Quick Look extension when it is not built, drops the Qt parts the app never loads, and judges Gatekeeper only after notarizing and stapling. Why: upstream's packager targets its own CI and accounts.
- `debloat`: every change to the app itself — build options passing through, the hidden and removed surfaces, the blocked network, the theme, the name and the bundled resources, each listed in `lean-build.md`. Why: 1 patch keeps every lean change in 1 place to rebase.

## NEVER
- Never commit a change on the fork's release branch. Cost: the next release overwrites it.
- Never release a build whose patches did not all apply cleanly. Cost: a half-patched app looks like ours and lacks a feature.

## Pairs with
- `lean-build.md` — the content of the `debloat` patch.
- `release.md` — how the patches become a published app.
