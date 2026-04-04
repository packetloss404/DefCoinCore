Codebase Review Recommendations
===============================

This file consolidates the review feedback from 30 parallel reviewers:

- 10 senior-developer reviewers
- 10 Litecoin-focused reviewers
- 10 documentation reviewers

The goal was to identify the highest-value follow-up work after a broad codebase and docs review.

What Was Corrected In This Pass
-------------------------------

- Rewrote the top-level `README.md` to match current Defcoin behavior more closely.
- Expanded `INSTALL.md` into a real install/build/validation guide.
- Added `COPYING.md` as a short licensing explainer while keeping `COPYING` as the canonical text.
- Tightened `.gitignore` for modern local/dev artifacts and extra MSVC state.
- Fixed a few high-signal documentation mismatches in `doc/README.md`, `doc/README_windows.txt`,
  `doc/TESTNET_GUIDE.md`, `doc/files.md`, `doc/bitcoin-conf.md`, and `share/examples/litecoin.conf`.

Highest-Priority Follow-Ups
---------------------------

1. Resolve runtime identity and packaging drift.
   - Checked-in generated config headers and some packaging metadata still identify the project as
     Bitcoin Core or Litecoin.
   - Review `src/config/bitcoin-config.h`, `build_msvc/bitcoin_config.h`, MSVC project metadata,
     macOS bundle identifiers, and GUI/resource names.

2. Decide and document network support clearly.
   - `-signet` is exposed in config/help/docs paths but is not implemented as a real signet chain.
   - Mainnet/testnet/regtest behavior is clearer than signet behavior today.
   - Either implement signet properly or remove/disable it and scrub user-facing references.

3. Align wallet defaults with Defcoin mainnet policy.
   - Mainnet documentation correctly warns that SegWit-style features are not active, but wallet,
     RPC, and GUI defaults still expose inherited Bech32 / P2SH-SegWit behaviors in places.
   - Review address defaults, help text, and user-visible warnings before calling those paths supported.

4. Finish the test-framework rename.
   - Large parts of the test harness and previous-release tooling still look for `litecoind`,
     `litecoin-cli`, and `litecoin.conf`.
   - That creates onboarding friction and can cause false assumptions in CI and local testing.

5. Rewrite release documentation as Defcoin documentation.
   - `doc/release-notes.md` and `doc/release-process.md` still read as upstream Litecoin material.
   - Keep upstream notes as archival reference if useful, but clearly label them as upstream-only.

6. Remove tracked generated MSVC/build artifacts from git history going forward.
   - `.gitignore` improvements help only for untracked files.
   - If generated binaries, libraries, or `vcpkg_installed` content are already tracked, they still
     need a deliberate cleanup commit outside this pass.

Documentation Debt Still Worth Addressing
-----------------------------------------

- `doc/init.md` and `contrib/init/*` still contain inherited Bitcoin naming and stale RPC-auth guidance.
- `doc/zmq.md`, `doc/REST-interface.md`, and several `contrib/*` docs still contain Litecoin/Bitcoin terms.
- `build_msvc/README.md`, BSD build docs, and some contributor docs still describe upstream projects.
- The docs tree mixes current Defcoin guidance with upstream archival notes without strong labeling.

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
