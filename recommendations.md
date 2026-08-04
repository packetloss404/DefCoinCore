Codebase Review Recommendations
===============================

This file consolidates the review feedback from 30 parallel reviewers:

- 10 senior-developer reviewers
- 10 Litecoin-focused reviewers
- 10 documentation reviewers

The goal was to identify the highest-value follow-up work after a broad codebase and docs review.

What Was Corrected In This Pass
-------------------------------

The first doc-consolidation pass (2026-08, this iteration):

- Rewrote the top-level `README.md` to match current Defcoin behavior more closely.
- Expanded `INSTALL.md` into a real install/build/validation guide.
- Added `COPYING.md` as a short licensing explainer while keeping `COPYING` as the canonical text.
- Tightened `.gitignore` for modern local/dev artifacts and extra MSVC state.
- Fixed a few high-signal documentation mismatches in `doc/README.md`, `doc/README_windows.txt`,
  `doc/TESTNET_GUIDE.md`, `doc/files.md`, `doc/bitcoin-conf.md`, and `share/examples/defcoin.conf`.
- Rewrote `doc/init.md` and the matching `contrib/init/*` service scripts (systemd, Upstart,
  OpenRC, CentOS, macOS) to defcoin binary / path / user naming. Renamed `bitcoind.*` to
  `defcoind.*` everywhere they appear under `contrib/init/`.
- Rebranded the daemon-interface docs (`doc/JSON-RPC-interface.md`, `doc/zmq.md`,
  `doc/REST-interface.md`) to defcoin and updated the JSON-RPC port example from 8332 to 9332
  with testnet / regtest notes.
- Rebranded the BSD build docs (`doc/build-freebsd.md`, `doc/build-netbsd.md`,
  `doc/build-openbsd.md`) and the MSVC build doc (`build_msvc/README.md`).
- Retargeted `doc/release-process.md` to defcoin (repo, sigs repo, binary names, macOS bundle,
  IRC channels, publishing endpoints) while keeping the upstream Gitian structure as a starting
  point, and replaced the stale upstream `doc/release-notes.md` draft with a defcoin-targeted
  template.
- Added `Status: upstream archival reference` banners to `doc/release-notes-litecoin.md` and
  `README.md` index files in both `doc/release-notes/` (Bitcoin Core history) and
  `doc/litecoin-release-notes/` (Litecoin Core history).
- Renamed `doc/man/litecoin-*.1` to `doc/man/defcoin-*.1`, updated `doc/man/Makefile.am` to use
  the defcoin file names, and switched `contrib/devtools/gen-manpages.sh` to defcoin env vars
  and binary paths.
- Added a `Status: inherited from upstream` banner to each inherited reference doc
  (`developer-notes`, `productivity`, `shared-libraries`, `dnsseed-policy`, `benchmarking`,
  `reduce-memory`, `reduce-traffic`, `translation_process`, `translation_strings_policy`,
  `fuzzing`, `psbt`, `descriptors`, `assets-attribution`, `bips`, `dependencies`) and
  `mweb/mining-changes.md` (with an explicit `not active on Defcoin mainnet` warning).
- Added a per-doc status table at the bottom of `doc/README.md` so the index itself exposes
  the current-vs-inherited-vs-archival split.

Highest-Priority Follow-Ups
---------------------------

1. Resolve runtime identity and packaging drift.
   - Checked-in generated config headers and some packaging metadata still identify the project as
     Bitcoin Core or Litecoin.
   - Review `src/config/bitcoin-config.h`, `build_msvc/bitcoin_config.h`, MSVC project metadata,
     macOS bundle identifiers, and GUI/resource names.
   - The MSVC solution's `bitcoin.sln` / `bitcoin-qt.vcxproj` filenames, the `BUILD_BITCOIND` etc.
     AM_CONDITIONAL names in `configure.ac`, and the `libdefcoinconsensus` (already defcoin-named)
     / `bitcoin-config.h` (still upstream-named) pairing are the concrete next items.

2. Decide and document network support clearly.
   - `-signet` is exposed in config/help/docs paths but is not implemented as a real signet chain.
   - Mainnet/testnet/regtest behavior is clearer than signet behavior today.
   - Either implement signet properly or remove/disable it and scrub user-facing references.

3. Align wallet defaults with Defcoin mainnet policy.
   - SegWit is active on Defcoin mainnet (chainparams.cpp: `SegwitHeight = 903168`), so Bech32 /
     P2SH-SegWit address paths are legitimately supported rather than inactive.
   - Review address defaults, help text, and user-visible messaging so they consistently reflect
     that SegWit is active, rather than warning it is unavailable.

4. Finish the test-framework rename.
   - Large parts of the test harness and previous-release tooling still look for `litecoind`,
     `litecoin-cli`, and `litecoin.conf`.
   - That creates onboarding friction and can cause false assumptions in CI and local testing.

5. Harden release documentation as Defcoin documentation.
   - `doc/release-process.md` and the next-cycle `doc/release-notes.md` are now defcoin-targeted
     templates, but the Gitian pipeline still assumes the upstream Bitcoin Core / Litecoin Core
     layout. The next pass should either stand up a defcoin.sigs.dfc + defcoin-detached-sigs pair
     or trim the Gitian sections down to just the parts that work on the defcoin tree today.
   - The per-version archives under `doc/release-notes/` and `doc/litecoin-release-notes/` are
     explicitly labelled upstream archival; once a real Defcoin Core release is cut, the per-
     version notes for that release should land under `doc/release-notes/`.

6. Remove tracked generated MSVC/build artifacts from git history going forward.
   - `.gitignore` improvements help only for untracked files.
   - If generated binaries, libraries, or `vcpkg_installed` content are already tracked, they still
     need a deliberate cleanup commit outside this pass.

7. Regenerate the man pages.
   - `doc/man/*.1` were renamed to defcoin-* but the file contents still describe the upstream
     Litecoin 0.21.2 placeholder. Run `contrib/devtools/gen-manpages.sh` after the first full
     defcoin build to refresh the content from the actual `defcoind --help` / `defcoin-qt --help`
     output.

Documentation Debt Still Worth Addressing
-----------------------------------------

- `doc/init.md` and `contrib/init/*` still contain inherited Bitcoin naming and stale RPC-auth guidance.
  *(resolved 2026-08: full defcoin rebrand + rename of `bitcoind.*` to `defcoind.*` across systemd, Upstart, OpenRC, CentOS, and macOS init.)*
- `doc/zmq.md`, `doc/REST-interface.md`, and several `contrib/*` docs still contain Litecoin/Bitcoin terms.
  *(resolved 2026-08: `zmq.md`, `REST-interface.md`, `JSON-RPC-interface.md` rebranded; `gen-manpages.sh` switched to defcoin env vars and binary paths; `doc/man/*.1` files renamed; `man/Makefile.am` updated.)*
- `build_msvc/README.md`, BSD build docs, and some contributor docs still describe upstream projects.
  *(resolved 2026-08: `build_msvc/README.md` and the freebsd/netbsd/openbsd build docs rebranded; the upstream `bitcoin.sln`/project references in the MSVC solution itself are still on the next-pass list.)*
- The docs tree mixes current Defcoin guidance with upstream archival notes without strong labeling.
  *(resolved 2026-08: per-file `Status: inherited from upstream` banners on the inherited reference docs, a top-of-doc upstream-archival banner on the litecoin release notes, README.md files in both historical release-notes archives, and a per-doc status table in `doc/README.md`.)*
- `doc/release-process.md` still reads as upstream Litecoin material.
  *(partially resolved 2026-08: defcoin repo, sigs repo, binary names, macOS bundle id, IRC channels and publishing endpoints retargeted. The Gitian pipeline layout is still the upstream Bitcoin Core / Litecoin Core layout — adjust the defcoin-specific bits as the project's own release process hardens.)*
- `doc/release-notes.md` was the stale upstream Litecoin 0.21.2 draft.
  *(resolved 2026-08: replaced with a defcoin-targeted release-notes template; the per-version archives under `doc/release-notes/` (Bitcoin Core) and `doc/litecoin-release-notes/` (Litecoin Core) are now explicitly labelled upstream archival.)*
- `doc/mweb/mining-changes.md` still described MWEB as if it were an active Defcoin mainnet feature.
  *(resolved 2026-08: explicit `not active on Defcoin mainnet` banner at the top of the doc.)*

Suggested Next Sequence
-----------------------

1. Clean package identity and generated metadata.
2. Fix signet handling and address-policy messaging.
3. Normalize test framework binary/config names.
4. Rewrite release docs and service/init docs.
5. Do a focused user-facing GUI/RPC branding sweep.

Repository Hygiene Notes
------------------------

- `.gitignore` was updated here, but tracked files are unaffected by ignore rules.
- The current worktree already contains unrelated code changes outside this documentation pass.
- Follow-up cleanup should avoid reverting unrelated in-progress work.
