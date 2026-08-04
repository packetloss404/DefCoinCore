# Handoff

**Last updated:** 2026-08-04
**Repo:** `D:\repo\DefCoinCore` on `main`
**Active line:** Defcoin Core 26.6.8l (Nu backend parity, classic `defcoin-qt` in maintenance mode)

## State

- Working tree clean against `origin/main`. Last commit: `77343911b update recommendations.md with the consolidation done in this pass`.
- Top-level rebrand (README, INSTALL, COPYING) and the full `doc/` consolidation pass (9 commits, `02e2b0a26` through `77343911b`) are in.
- 4 audit reports just landed in `.audit/`, **uncommitted** (this commit is the first to track them).
- No HANDOFF.md existed before this commit.

## What's in this commit

1. `.audit/upstream-drift.md` — code-drift audit vs Litecoin Core `v0.21.5.5` (the in-tree `CLIENT_VERSION_*` macro value).
2. `.audit/consensus-audit.md` — mainnet activation heights, MWEB / Taproot / Signet surface, RPC surface, P2P magic and UA-filter wiring.
3. `.audit/doc-gaps.md` — missing user-facing docs beyond the consolidation pass.
4. `.audit/build-ci-audit.md` — `.github/workflows/`, `build_msvc/`, `test/`, `depends/`, autotools, `share/`, packaging icons, dormant CI files.
5. `HANDOFF.md` — this file.

## TL;DR from the audits

- **Upstream code drift:** clean. Tree matches Litecoin `0.21.5.5` byte-for-byte on the consensus-adjacent files (`txdb.cpp`, `txmempool.cpp`, `wallet.h`, `walletdb.cpp`, `miner.cpp`, `policy.cpp`, `blockencodings.cpp`, `chainparamsbase.cpp`, `httpserver.cpp`, `util/bip32.{h,cpp}`). Embedded `leveldb` 1.22 and `univalue` 1.0.3 also match. Every file that grew beyond upstream is an intentional Nu addition (Fast Sync, MWEB) with a companion `*.agent.md`. Only real gap is the `v0.21.5.5 → v0.21.5.6` point-release delta (~4.6 KB across `net_processing.cpp` / `validation.cpp` / `chainparams.cpp`).
- **Consensus:** no critical/high consensus issues. Activation heights in `chainparams.cpp:68-92` match the README exactly. MWEB and Taproot are properly `NEVER_ACTIVE` on mainnet, magic-byte negotiation and the `/Defcoin` user-agent filter are correctly wired. Two **high** issues are operator-performance, not consensus.
- **Docs:** 18 gaps, 5 quick wins. Top: `SECURITY.md`, `doc/magic-migration.md` (cutover 2026-08-01, days away per the README), `doc/wallet-guide.md`, `doc/address-types.md`, `doc/mining-pools.md`, rebrand `src/qt/README.md`, promote `doc/TESTNET_GUIDE.md` troubleshooting into `doc/faq.md`.
- **Build / CI / test:** 42 issues, 4 critical — all in the test framework. CI is "green-by-default" because the GitHub Actions workflows run no tests; the functional suite would fail at import on `import litecoin_scrypt` in `test/functional/test_framework/messages.py:32`. MSVC rebrand half-done; packaging still ships `bitcoin.ico` / `bitcoin.icns`; `Litecoin-Qt.app` artefacts in `contrib/macdeploy/`.

## Next-pass priority order

In order, this is what the next pass should hit:

1. **Test framework rebrand (build/CI #1-4, 4 critical, blocks the whole test suite).**
   Add the missing `litecoin_scrypt` Python shim (or rename to `defcoin_scrypt`), rewire `test_framework.py` / `test_node.py` / `util.py` to `defcoind` / `defcoin-cli` / `defcoin.conf`, rename `BITCOIND` / `BITCOINCLI` env vars to `DEFCOIND` / `DEFCOINCLI`. Then `test/util/data/bitcoin-util-test.json` → `defcoin-util-test.json` (83 `litecoin-tx` exec strings), `test/test_litecoin` → `test/test_defcoin` in `src/Makefile.test.include`, align `.gitignore` line 14. Add `make check` and `test_runner.py` to `.github/workflows/build.yml` so CI actually catches this class of regression going forward.

2. **Magic-migration operator runbook (docs #2, time-sensitive).**
   Cutover to defcoin-only magic is 2026-08-01 (per the README). Write `doc/magic-migration.md`: cutover date, what `-acceptlegacymagic` does, how to verify a node is defcoin-only, the `/Defcoin` UA requirement, expected log noise during the window, rollback notes for pool/merchant operators. The code path is correct (verified in the consensus audit); this is documentation only.

3. **Slow mainnet IBD (consensus #1-2, high).**
   Set `consensus.defaultAssumeValid` to a recent mainnet block hash and update `consensus.nMinimumChainWork` to a realistic value (~2 orders of magnitude above the current `0x...0001000000000000` placeholder). New nodes currently full-validate every block from genesis.

4. **`SECURITY.md` (docs #1, critical).**
   Responsible-disclosure email/PGP, supported versions, CVE/coordinated-disclosure timeline, what is in/out of scope, security-update communication channel. 30-80 lines. Even a stub is better than nothing — GitHub's security-tab UI surfaces this file directly.

5. **MSVC rebrand (build/CI #5-7, high).**
   Rename `build_msvc/bitcoin.sln` and the 16 sub-`*.vcxproj` files to `defcoin.sln` / `defcoin-*`, update the embedded `Project(...) = "..."` names and GUIDs. Regenerate `build_msvc/bitcoin_config.h` (or rename to `defcoin_config.h`) against the current 26.6.8l `configure.ac` — the tracked template is still `CLIENT_VERSION_MAJOR=0, MINOR=21, REVISION=1` (Litecoin 0.21.2 era). Fix `build_msvc/msvc-autogen.py` argparse description ("Bitcoin-core msbuild configuration initialiser" → defcoin). Add a CI smoke test that runs `msbuild /t:Restore build_msvc/defcoin.sln`.

6. **Packaging icon + DMG rebrand (build/CI #12-14, high).**
   Switch `share/pixmaps/bitcoin.ico` (and the rest of the `bitcoin*.{png,xpm}` set) to defcoin, fix `src/qt/res/bitcoin-qt-res.rc` (`IDI_ICON1 ICON DISCARDABLE "icons/bitcoin.ico"`), fix `share/qt/Info.plist.in:14` (`bitcoin.icns`), fix `Makefile.am:43,58` and `contrib/macdeploy/macdeployqtplus:157,642` / `custom_dsstore.py:56` (`Litecoin-Qt.app` → `Defcoin-Qt.app`). Until this lands, every Windows installer, macOS bundle, and DMG ships with a Bitcoin/Litecoin icon and bundle name.

7. **Dormant P2TR surface hardening (consensus #3, medium).**
   In `src/script/interpreter.cpp:1885-1887`, replace the unconditional `set_success` for v1/32-byte witness programs with `SCRIPT_ERR_DISCOURAGE_UPGRADABLE_WITNESS_PROGRAM` when `SCRIPT_VERIFY_TAPROOT` is unset. Currently unreachable (no `bech32m_hrp` in chainparams, `policy.cpp:198-200` blocks P2TR at mempool), but a forward-compat hazard if Taproot is ever soft-forked on.

8. **Upstream backport: v0.21.5.5 → v0.21.5.6 (upstream-drift #1, critical).**
   Three files changed: `src/net_processing.cpp` (+2,715 B peer-misbehavior path), `src/validation.cpp` (+1,670 B assumeutxo robustness), `src/chainparams.cpp` (+259 B). Fetch from `https://raw.githubusercontent.com/litecoin-project/litecoin/v0.21.5.6/src/...`, re-apply the Nu / Fast-Sync additions from the companion `*.agent.md` files, rebuild. Risk: low — the merge base is clean (the byte-identical files above confirm it).

9. **`src/qt/README.md` rebrand (docs #13, medium, 1 hour).**
   The one file the previous rebrand pass missed. Still says `litecoin-qt` / `LitecoinQT` end-to-end. Re-stamp binary paths, class names, and the Qt Creator section for `defcoin-qt`.

10. **Man page content regen (recommendations.md #7, medium).**
    The man *filenames* are defcoin now (renamed in the consolidation pass), but the body still says "LITECOIND" / "Litecoin Core v0.21.2.0". Run `contrib/devtools/gen-manpages.sh` after the next full defcoin build to refresh the content from the actual `defcoind --help` / `defcoin-qt --help` output.

11. **Cleanup housekeeping (low, 1 hour total).**
    `git rm .appveyor.yml .travis.yml .cirrus.yml .fuzzbuzz.yml` — all four are dormant and the scripts they reference still carry upstream names. Point `depends/Makefile:41` `FALLBACK_DOWNLOAD_PATH` away from `https://bitcoincore.org/depends-sources` to a defcoin-controlled mirror (or document the upstream dependency). Bump `depends/hosts/darwin.mk` SDK pins (Xcode 11.3.1 / macOS SDK 10.15.1 → current) for modern Apple toolchains.

12. **Remaining doc gaps.** Full table in `.audit/doc-gaps.md`. The next-quickest are `doc/address-types.md` (legacy vs P2SH-SegWit vs bech32 on Defcoin), `doc/mining-pools.md` (replacement for the correctly-banned MWEB mining guide, with the explicit "do not pass `mweb` in `getblocktemplate.rules`" warning), `doc/community.md` (chat / explorer / faucet / contact links).

## Pointers

- `.audit/upstream-drift.md` — code drift, file-by-file.
- `.audit/consensus-audit.md` — activation heights, MWEB / Taproot / Signet, RPC surface, P2P wiring.
- `.audit/doc-gaps.md` — 18 missing docs, prioritized.
- `.audit/build-ci-audit.md` — 42 issues, 4 critical, 15 high.
- `recommendations.md` — historical follow-up list from the prior doc pass; mostly still valid, items 1 (MSVC project metadata), 4 (test-framework rename), 7 (man page regen) are the ones the audit confirmed as the next concrete work.
- `doc/README.md` — per-doc status table covering the `doc/` tree.

## Style notes

- Solo dev. Terse lowercase commit messages, `docs/` folder over one big README, commit → push after each meaningful change.
- No upstream CI runs the doc-content checks; any new docs-build workflow (markdownlint, link checker) is a separate, low-priority follow-up.
- The four audit sub-agents wrote only to `.audit/`. The project tree was not modified by the audit pass.
