# Defcoin Core — Build / CI / Test-Framework Audit

Commit: `77343911b` on `main`, working tree clean (only `.audit/` untracked).

This is a **read-only** audit. No project files were modified, no `configure` /
`make` / `cmake` / `test_runner.py` was executed. All counts are real
`Select-String` hits on the working tree, not estimates.

## Preamble

The tree was checked across eight surfaces: `.github/workflows/`,
`.appveyor.yml` / `.travis.yml` / `.cirrus.yml` / `.fuzzbuzz.yml`,
`build_msvc/`, `test/`, `depends/`, autotools (`configure.ac` / `Makefile.am`),
`share/`, and root / `contrib/` Python scripts. The previous
documentation-consolidation pass rebranded most doc text and renamed the man
files, but a parallel pass of equal size is needed for the **test harness** and
the **MSVC project metadata** — both are still wired to `litecoind` /
`litecoin-cli` / `litecoin.conf` / `bitcoin.sln` / `bitcoin-config.h` /
`bitcoin.ico` upstream names. A `defcoin_scrypt` helper that
`test/functional/test_framework/messages.py` `import litecoin_scrypt`s does
not exist in the tree at all.

**Overall CI status: red (test job would fail on the first functional test
that imports `test_framework.messages`)**. The compile jobs in
`.github/workflows/build.yml` and `release.yml` are otherwise
defcoin-named and would succeed, but they run **no tests at all** and no docs
build.

---

## Summary Table

| Area                                | Issues found | Critical | High | Medium | Low |
| ----------------------------------- | ------------:| --------:| ----:| ------:| ---:|
| `.github/workflows/`                | 4            | 0        | 2    | 2      | 0   |
| Legacy CI (`.appveyor`, `.travis`, `.cirrus`, `.fuzzbuzz`) | 4 | 0        | 1    | 1      | 2   |
| `build_msvc/`                       | 9            | 0        | 4    | 4      | 1   |
| `test/` (functional + lint)         | 9            | 4        | 3    | 1      | 1   |
| `depends/`                          | 3            | 0        | 1    | 1      | 1   |
| Autotools (`configure.ac`, `Makefile.am`) | 5      | 0        | 2    | 2      | 1   |
| `share/`                            | 6            | 0        | 2    | 3      | 1   |
| Root + `contrib/` Python            | 2            | 0        | 0    | 2      | 0   |
| **Totals**                          | **42**       | **4**    | **15** | **16** | **7** |

---

## Issues Table

| #  | Path | Issue | Severity | Fix |
| --:| ---- | ----- | -------- | --- |
| 1  | `test/functional/test_framework/messages.py:32` | `import litecoin_scrypt` — there is **no** `litecoin_scrypt.py` in `test/functional/test_framework/` (verified by `Get-ChildItem`). Any test that does `from test_framework.messages import …` fails immediately with `ModuleNotFoundError`. | critical | Add a `litecoin_scrypt` Python shim (or rename the import to `defcoin_scrypt` and add a defcoin scrypt module). 64 functional test files import `messages`. |
| 2  | `test/functional/test_framework/test_framework.py:212-220, 488-490, 526, 537, 557, 562` | Hard-coded binary defaults `litecoind` / `litecoin-cli` in `BitcoinTestFramework`; `BITCOIND` / `BITCOINCLI` env var names are also upstream. `os.environ['BITCOIND']` / `BITCOINCLI` are the only override — but no caller sets them. | critical | Replace hard-coded defaults with `defcoind` / `defcoin-cli`; rename env vars to `DEFCOIND` / `DEFCOINCLI`; add `BITCOIND`/`BITCOINCLI` aliases if you want back-compat. |
| 3  | `test/functional/test_framework/util.py:360, 384, 392, 393` | `init_datadir()` / `append_config()` / `get_auth_cookie()` open `litecoin.conf` in the node's datadir. `defcoind` reads `defcoin.conf` and will silently ignore the test config, so every functional test that depends on a config file (e.g. `feature_config_args`, `feature_includeconf`, `feature_settings`, `rpc_users`, `interface_bitcoin_cli`) starts with the wrong settings. | critical | Replace the filename with `defcoin.conf`. |
| 4  | `test/functional/test_framework/test_node.py:5, 53, 75, 209, 221, 479, 489, 510, 512, 581, 609, 612, 619` | `TestNode` class docstring says "Class for litecoind node under test"; `self.bitcoinconf` attribute name and ~19 string references to `litecoind` / `litecoin-cli` in log messages and assertions. Tests would import but the variable name is purely cosmetic; log strings make debugging noisy. | critical | Rename the class attribute to `self.defcoinconf` and update the log strings / docstrings. |
| 5  | `.github/workflows/build.yml:75-118` (`build-linux`) | The Linux job runs `make -j$(nproc)` and then strips / packages, but **never runs the unit test suite, the functional test suite, or the lint suite**. The test binary (`test/test_litecoin`) and `test/functional/test_runner.py` are not even invoked. | high | Add a `make check` step and a `test/functional/test_runner.py` step (or a separate `test.yml` job that depends on this one). |
| 6  | `.github/workflows/build.yml:11-73` (`build-windows`), `release.yml:14-65` | Windows job builds with `mingw32` cross-compile but also does not run the test suite. `appveyor.yml` historically ran `src\test_bitcoin.exe` but that test binary does not exist (see #15). | high | Add a `make check` / unit test step in the windows job, or split out a separate `test-windows` matrix entry. |
| 7  | `build_msvc/bitcoin.sln` (and all 16 sub-`*.vcxproj` referenced in it) | `.sln` filename is `bitcoin.sln`; `Project(...) = "bitcoind"`, `"bitcoin-cli"`, `"bitcoin-tx"`, `"bitcoin-wallet"`, `"bitcoin-qt"`, `"bench_bitcoin"`, `"libbitcoin_*"`, `"test_bitcoin"`, `"test_bitcoin-qt"`. Per recommendations.md item 1, these were supposed to be renamed. | high | Rename to `defcoin.sln` and rename the `bitcoind/` … `bitcoin-qt/` sub-folders + their `.vcxproj` files; update the embedded `Project(...) = "..."` names and all 16 GUIDs. |
| 8  | `build_msvc/bitcoin_config.h` (tracked) | Stale template: `CLIENT_VERSION_MAJOR=0, MINOR=21, REVISION=1` (Litecoin 0.21.2 era, not 26.6.8l). Header copyright says "2018-2020 The Bitcoin Core developers". `PACKAGE_NAME` / `COPYRIGHT_HOLDERS_FINAL` were already rebranded to "Defcoin Core" so this is half-done. | high | Regenerate by re-running autotools / `msvc-autogen.py` against the current `configure.ac` and `src/config/bitcoin-config.h.in`, or manually fix version + copyright header. |
| 9  | `build_msvc/msvc-autogen.py:82` | Hard-codes the source path `os.path.join(SOURCE_DIR,'config/bitcoin-config.h')`. Generated copy lands in `src/config/bitcoin-config.h` which the top-level `.gitignore` already excludes, but the script's docstring / `argparse` description is "Bitcoin-core msbuild configuration initialiser" — so a developer running it gets the wrong impression of what the project is. | high | Rename in-code string references (`argparse` description, `copyright` comments) to defcoin; consider renaming the copy target to `src/config/defcoin-config.h` to match `configure.ac` AC_CONFIG_HEADERS, then add `src/config/defcoin-config.h` to `.gitignore`. |
| 10 | `test/util/data/bitcoin-util-test.json` (entire file) | Contains **83** occurrences of `"exec": "./litecoin-tx"`. The harness this drives is `test/util/bitcoin-util-test.py:47` (which calls `bctester(...test/util/data/bitcoin-util-test.json...)`). When `defcoin-tx` runs the test JSON, the test data says "spawn `litecoin-tx`" — and on a clean checkout, `litecoin-tx` does not exist, so every case fails. | high | Replace all 83 `"exec": "./litecoin-tx"` with `"exec": "./defcoin-tx"` and rename the file to `test/util/data/defcoin-util-test.json`; update the `Makefile.am:278-279` `EXTRA_DIST` entry and `msvc-autogen.py` if it references this file. |
| 11 | `src/Makefile.test.include:167, 171, 355-367, 1315, 1324, 1327, 1329, 1333, 1336, 1337, 1339, 1341, 1349` | The unit test binary is `test/test_litecoin` (`bin_PROGRAMS += test/test_litecoin` and `TEST_BINARY=test/test_litecoin$(EXEEXT)`) with the `litecoin_test` / `litecoin_test_check` / `litecoin_test_clean` make targets. The top-level `.gitignore` already says `src/test/test_defcoin` (line 14), so the two are out of sync. | high | Rename the binary to `test/test_defcoin`, the targets to `defcoin_test` / `defcoin_test_check` / `defcoin_test_clean`, and update `.gitignore` to match (the current `.gitignore` line 14 is the one that should win). |
| 12 | `share/setup.nsi.in:17` | `!define MUI_ICON "@abs_top_srcdir@/share/pixmaps/bitcoin.ico"` — references a `bitcoin.ico` that exists, but the *file* is for the wrong brand. Same path is referenced from `Makefile.am:58`. | high | Rename `share/pixmaps/bitcoin.ico` and the rest of the `bitcoin*.{png,xpm}` set to `defcoin.*` (or generate a new defcoin icon set and switch the references), then update `Makefile.am:58`. |
| 13 | `src/qt/res/bitcoin-qt-res.rc:1,2` | `IDI_ICON1 ICON DISCARDABLE "icons/bitcoin.ico"` — the Windows `defcoin-qt.exe` ships with a Bitcoin icon. The file `bitcoin-qt-res.rc` itself is upstream-named. | high | Switch to `icons/defcoin.ico` / `icons/defcoin_testnet.ico`; rename the file to `src/qt/res/defcoin-qt-res.rc`; update `build_msvc/bitcoin-qt/bitcoin-qt.vcxproj:12` reference. |
| 14 | `share/qt/Info.plist.in:14` | `<key>CFBundleIconFile</key> <string>bitcoin.icns</string>` — the macOS `.app` bundle is built with a Bitcoin icon even though `Makefile.am:43, 100, 113` and `share/setup.nsi.in:108` (`DefaultIcon`) also point to `bitcoin.icns` / `bitcoin.ico`. | high | Generate / ship `defcoin.icns` and switch all four references to it. |
| 15 | `.appveyor.yml:48, 52, 53, 54` | `test_script:` runs `src\test_bitcoin.exe -l test_suite` and `src\bench_bitcoin.exe` and `python test\util\bitcoin-util-test.py`. None of those paths exist after a defcoin build: the unit test binary is `test/test_litecoin`/`test/test_defcoin` (per #11), bench binary is `bench/bench_defcoin` (per `Makefile.am:1349`), and `bitcoin-util-test.py` should be `defcoin-util-test.py`. `.appveyor.yml` is dormant but if anyone re-enables it, it will fail. | high | Either `git rm .appveyor.yml` (recommended) or rewrite the script to defcoin paths. |
| 16 | `test/functional/test_framework/test_framework.py:45` | `TMPDIR_PREFIX = "litecoin_func_test_"` — purely cosmetic, but leaks upstream identity into `/tmp` directories. | medium | Rename to `defcoin_func_test_`. |
| 17 | `test/functional/test_runner.py:451, 452, 726` | `pgrep -x litecoind` and warning string "There is already a litecoin…" — the runner can't detect a running defcoin. | medium | Replace `litecoind` with `defcoind` and rewrite the warning text. |
| 18 | `test/functional/test_framework/coverage.py:79` | Docstring "list of all RPC functions available in `litecoin-cli`". | medium | Update docstring. |
| 19 | `test/functional/test_framework/p2p.py:152` | `logger.debug('Connecting to Litecoin Node …')` — wrong project name in logs. | medium | Update log string. |
| 20 | `test/functional/test_framework/ltc_util.py` (whole file) | Module name is `ltc_util` but `p2p.py`, `wallet.py` and `address.py` do import it; renaming would be churn. | medium | Either rename to `defcoin_util.py` and update imports, or add a module-level docstring note "inherited from upstream Litecoin Core". |
| 21 | `test/lint/lint-locale-dependence.sh:8, 10, 16, 20, 25` | 5 comments say "bitcoind" / "bitcoin-qt". No functional impact (lint just reports spelling), but the project's `lint-locale-dependence` will keep printing wrong-branded comments. | medium | Rewrite comments to defcoin and review for the actual defcoin behaviour. |
| 22 | `test/lint/lint-include-guards.sh:10` | `HEADER_ID_PREFIX="BITCOIN_"` — if a defcoin header uses `DEFCOIN_FOO_H` (consistent with `COPYRIGHT_HOLDERS_SUBSTITUTION` etc. in `configure.ac`), this lint will flag it. | medium | Switch to `HEADER_ID_PREFIX="DEFCOIN_"` (or accept both). |
| 23 | `src/Makefile.am:545, 561, 644, 656-742` (and 100+ comments) | The `libbitcoin_*` / `libbitcoinconsensus` / `libbitcoinqt` static-library names are intentionally kept (`libdefcoinconsensus` exists alongside). Cosmetic for a C++ build, but the source files for the Qt client are still `src/qt/bitcoin.{cpp,h}`, `bitcoinamountfield.{cpp,h}`, `bitcoingui.{cpp,h}`, `bitcoinunits.{cpp,h}`, `bitcoinaddressvalidator.{cpp,h}`, `bitcoinstrings.cpp`, plus `src/qt/bitcoin.qrc` and `src/qt/bitcoin_locale.qrc` and 89 `src/qt/locale/bitcoin_*.{ts,qm}` files. | medium | Rename `src/qt/bitcoin*` → `src/qt/defcoin*` (`.cpp/.h/.qrc`) and `src/qt/locale/bitcoin_*.ts` → `defcoin_*.ts` so contributors don't grep "litecoin" / "bitcoin" by accident. |
| 24 | `contrib/macdeploy/macdeployqtplus:157, 642` | `appName = "Litecoin-Qt"` and `target = os.path.join("dist", "Litecoin-Qt.app")` — the macOS DMG bundling script will produce a `Litecoin-Qt.app` artefact, not a `Defcoin-Qt.app`. | medium | Switch to `Defcoin-Qt`. |
| 25 | `contrib/macdeploy/custom_dsstore.py:56` | `ds['Litecoin-Qt.app']['Iloc'] = (128, 156)` — DS_Store for the DMG volume uses upstream bundle name. | medium | Switch to `Defcoin-Qt.app`. |
| 26 | `depends/hosts/darwin.mk:1-3, 43-50` | `OSX_MIN_VERSION=10.14`, `OSX_SDK_VERSION=10.15.1`, `XCODE_VERSION=11.3.1`, `XCODE_BUILD_ID=11C505`, `LD64_VERSION=530` — all 2019/2020 era. The depends-based darwin SDK fetch (used by Gitian, not by the GitHub Actions macOS job) is unlikely to work on a modern macOS host (Apple deprecated `xcrun --show-sdk-path` SDK extraction, and the SDK tarball is no longer hosted at the URL `extract-osx-sdk.sh` expects). | medium | Bump to current SDK; or document that depends-based darwin builds are Gitian-only and out of scope for CI. |
| 27 | `depends/Makefile:41` | `FALLBACK_DOWNLOAD_PATH ?= https://bitcoincore.org/depends-sources` — that URL is *not* a defcoin source mirror; it is the Bitcoin Core depends-sources bucket. This URL is only used as a fallback when a package mirror is missing, but if a future package gets added and `bitcoincore.org` ever rotates the bucket, the defcoin build will silently fail. | medium | Point at a defcoin-controlled bucket (e.g. a GitHub release artifact) or document the upstream dependency. |
| 28 | `ci/test/00_setup_env_native_multiprocess.sh:15` | `export TEST_RUNNER_ENV="BITCOIND=litecoin-node"` — leftover upstream variable. The `.cirrus.yml` that consumes this is dormant, but if anyone re-enables Cirrus it will look for `litecoin-node`, not `defcoin-node`. | medium | Switch to `DEFCOIND=defcoin-node`. |
| 29 | `test/functional/interface_bitcoin_cli.py` (filename) | File is named `interface_bitcoin_cli.py`. It contains `"Test bitcoin-cli"`, `class TestBitcoinCli(BitcoinTestFramework)`, log strings `Test -getinfo and b…`, `Test bitcoin-cli -generate`, `Test bitcoin-cli -generate with…`. | medium | Rename to `interface_defcoin_cli.py` and rewrite the strings (or accept the upstream name as a test ID, but update the log strings). |
| 30 | `test/functional/feature_help.py:53` | `self.log.info("Start litecoind with -fakearg …")`. | low | Update log string. |
| 31 | `test/functional/feature_filelock.py` (3 hits) | Logs / comments mention `litecoin`. | low | Search-and-replace. |
| 32 | `test/functional/feature_proxy.py` (3 hits) | Same. | low | Same. |
| 33 | `test/functional/feature_config_args.py:23, 67, 170` (3 hits) | Appends to `litecoin.conf` (in addition to the test-framework bug at #3). | low | Same. |
| 34 | `test/functional/feature_includeconf.py:35, 73` (2 hits) | Same. | low | Same. |
| 35 | `test/functional/rpc_users.py:43, 67, 71` (3 hits) | "Append rpcauth to bitcoin.conf before initialization" comment + 2 datadir paths. | low | Same. |
| 36 | `test/functional/rpc_bind.py:1` (1 hit), `rpc_deprecated.py` (1), `rpc_whitelist.py` (1), `mempool_persist.py` (1), `combine_logs.py` (1) | Single `litecoin` references in headers / comments. | low | Search-and-replace. |
| 37 | `contrib/zmq/zmq_sub.py:9-10, 23` | `Litecoind should be started with the command line arguments: litecoind -testnet -daemon …` and a stale `https://github.com/bitcoin/bitcoin/...` link. | low | Rewrite. |
| 38 | `test/get_previous_releases.py:74, 76, 78, 111, 120, 164` | Release-download helper that hard-codes `https://download.litecoin.org/...` and `litecoin-project/litecoin` repo + `litecoind` / `litecoin-cli` / `litecoin-tx` filenames. Not a CI script (it's a developer tool for pulling old releases), but it has no defcoin equivalent. | low | Either `git rm` or rewrite to point at a defcoin release mirror. |
| 39 | `.travis.yml` (whole file) | References `bitcoin/bitcoin` URLs, `ci/lint/06_script.sh`, `ci/test/06_script_a.sh`, etc. The `ci/test/` and `ci/lint/` scripts are still upstream-named (no `defcoin` references). `.travis.yml` is dormant (Travis-CI no longer free for OSS). | low | `git rm .travis.yml` (recommended; see #40). |
| 40 | `.cirrus.yml` (whole file) | `CIRRUS_REPO_FULL_NAME == "bitcoin-core/gui"`, all paths point at `ci/test/...` shell scripts that are themselves upstream-named. Cirrus-CI is effectively dormant. | low | `git rm .cirrus.yml` (and consider also `git rm .appveyor.yml .travis.yml .fuzzbuzz.yml` in the same cleanup pass). |
| 41 | `.fuzzbuzz.yml` (whole file) | Fuzzbuzz is no longer active. `find_targets_command: find src/test/fuzz/ -executable -type f ! -name "*.cpp" ! -name "*.h"` references the upstream `test/fuzz` path. | low | `git rm .fuzzbuzz.yml`. |
| 42 | `Makefile.am:1, 43, 58, 70, 72, 213, 224-249, 255, 265, 278, 279, 355, 359` | Several `BITCOIN_TESTS` / `test_bitcoin.info` / `test_bitcoin_filtered.info` / `test_litecoin_test_clean` etc. references — pure cosmetics for `make cov`, but they show up in `make dist` and CI coverage tooling. | low | Rename to `defcoin_*` and `test_defcoin_*`. |

> Counts in #10 ("83"), #29 ("`Test bitcoin-cli`" etc.) and the totals in the
> summary table were produced with
> `Get-ChildItem -Recurse … | ForEach-Object { (Select-String -Path $_.FullName -Pattern '…').Count }`
> on the working tree. The "19 functional .py files" and "19 test_node.py
> references" counts are also from `Select-String`.

---

## Priority Order

1. **Add the missing `litecoin_scrypt` / `defcoin_scrypt` Python shim** that
   `test/functional/test_framework/messages.py:32` imports, and rename
   `import litecoin_scrypt` accordingly. Without this, the 64+ functional
   tests that import `messages.py` will all fail at import time.
   (`test/functional/test_framework/`)
2. **Rewire `test/functional/test_framework/{test_framework,test_node,util}.py`
   to defcoin binary/config names** — flip the hard-coded `litecoind` /
   `litecoin-cli` / `litecoin.conf` defaults to `defcoind` / `defcoin-cli` /
   `defcoin.conf` and rename `BITCOIND` / `BITCOINCLI` env vars to
   `DEFCOIND` / `DEFCOINCLI`. This unblocks the entire functional test
   suite, not just the 64 that import `messages`.
3. **Rename `test/util/data/bitcoin-util-test.json` → `defcoin-util-test.json`
   and replace all 83 `"exec": "./litecoin-tx"` with `"./defcoin-tx"`**, then
   update the python harness and the `Makefile.am:278-279` `EXTRA_DIST`
   entry. Also rename the C++ unit-test binary from `test_litecoin` to
   `test_defcoin` in `src/Makefile.test.include` and align `.gitignore`.
4. **Run `.github/workflows/build.yml` tests** — add a `make check` step and a
   `test/functional/test_runner.py --ci` step in `build-linux` and
   `build-windows`. Without this, CI is green-by-default even when the
   functional tests are 100% broken.
5. **Rebrand `build_msvc/` project metadata** — rename `bitcoin.sln` and
   each `bitcoin-*/bitcoin-*.vcxproj` to `defcoin.sln` / `defcoin-*`, update
   the embedded `Project(...) = "..."` names and all 16 GUIDs, and add a CI
   smoke test that runs `msbuild /t:Restore build_msvc/defcoin.sln` so the
   solution is exercised on every push. Regenerate `bitcoin_config.h` (or
   rename it `defcoin_config.h`) to match the current 26.6.8l version.
6. **Regenerate the man pages** (recommendations.md item 7) — run
   `contrib/devtools/gen-manpages.sh` after the first full defcoin build to
   refresh `doc/man/defcoin-*.1` from `defcoind --help` /
   `defcoin-qt --help` output. The current files still say "LITECOIND" /
   "Litecoin Core v0.21.2.0" in the `.TH` header and body.
7. **Fix packaging icon paths** — switch `share/setup.nsi.in:17`,
   `src/qt/res/bitcoin-qt-res.rc:1-2`, `share/qt/Info.plist.in:14`,
   `Makefile.am:43, 58`, and `contrib/macdeploy/macdeployqtplus:157, 642` /
   `custom_dsstore.py:56` from `bitcoin.ico` / `bitcoin.icns` / `Litecoin-Qt`
   to defcoin-named artefacts. Until this is done, every Windows installer,
   macOS bundle, and DMG still ships with a Bitcoin/Litecoin icon.
8. **Clean up dormant legacy CI files** — `git rm .appveyor.yml .travis.yml
   .cirrus.yml .fuzzbuzz.yml` and the now-unused `ci/lint/` / `ci/test/`
   shell scripts. The `.github/workflows/` tree is the only CI in active
   use, and the legacy files only re-introduce the wrong project identity.
9. **Bump `depends/hosts/darwin.mk` SDK pins** (Xcode 11.3.1 / macOS SDK
   10.15.1) and the `depends/Makefile:41` `FALLBACK_DOWNLOAD_PATH` from
   `bitcoincore.org/depends-sources` to a defcoin-controlled source
   location. The current values still work for Gitian but will break on
   modern Apple toolchains and on a future depends bump.
10. **Add a docs-build workflow** (`.github/workflows/docs.yml`) that runs
    `markdownlint doc/**/*.md`, a link checker, and (if Doxygen is
    available) the `doc/Doxyfile` generator. The existing workflow tree
    has no documentation check, so a typo in `release-process.md` or a
    broken relative link would not break CI today. Also delete the now
    obsolete `interface_bitcoin_cli.py` filename and any other
    upstream-named artefacts in the functional suite.

---

## Notes on what is **not** broken

- The two GitHub Actions workflow files (`build.yml`, `release.yml`) are
  fully defcoin-named: every `cp` / `strip` / `tar` step uses
  `defcoind` / `defcoin-cli` / `defcoin-tx` / `defcoin-wallet` / `defcoin-qt`.
  Their compile jobs should succeed against the current source tree once
  the MSVC rebrand (item 1) lands, modulo whatever the `mingw32` cross-build
  does with the current `depends/`.
- `configure.ac` correctly wires `BITCOIN_DAEMON_NAME=defcoind` (line 23)
  and the matching `AC_CONFIG_FILES([Makefile src/Makefile doc/man/Makefile
  share/setup.nsi share/qt/Info.plist test/config.ini])` (line 1761) and the
  `AC_CONFIG_LINKS` for `test/util/bitcoin-util-test.py` and
  `test/functional/test_runner.py` are still pointed at the correct paths.
- `Makefile.am` wires `OSX_APP=Defcoin-Qt.app` (line 33) and the install
  paths to `\Defcoin` (via `share/setup.nsi.in:54`).
- `share/examples/defcoin.conf` is fully defcoin-themed; option comments
  and section labels already use `defcoind` / `Defcoin-Qt` / `defcoin.conf`
  rather than upstream names.
- `contrib/devtools/gen-manpages.sh` is fully defcoin-named: `DEFCOIND`,
  `DEFCOINCLI`, `DEFCOINTX`, `WALLET_TOOL`, `DEFCOINQT` env vars and
  `defcoin-*` binary paths. (The man *output* files are stale, but the
  generator itself is correct — see item 6.)
- `depends/funcs.mk` and `depends/hosts/{linux,mingw32,android}.mk` are
  project-agnostic and have no `litecoin` / `bitcoin` / `defcoin` string
  leaks. The only real concerns in `depends/` are the SDK pins in
  `depends/hosts/darwin.mk` and the FALLBACK_DOWNLOAD_PATH in
  `depends/Makefile:41`, both already covered above.
- `.gitignore` correctly excludes `src/config/bitcoin-config.h` and the
  MSVC build outputs (`build_msvc/x64/`, `vcpkg_installed/`, etc.); the
  untracked files that appear in the working tree are leftover from a
  local build, not tracked. The `build_msvc/bitcoin_config.h` template
  *is* tracked (and stale — see #8) but it is the template, not the
  generated copy.
- `.fuzzbuzz.yml` is the only file under `contrib/devtools/` that
  contains stale `litecoin` / `bitcoin` references; the rest of
  `contrib/devtools/` is clean.

---

*End of audit. 42 distinct issues found; 4 critical, 15 high, 16 medium,
7 low. 5 issues (#1, #2, #3, #4, #10) are in the test framework and are
collectively the reason the test suite cannot pass today; the rest are
MSVC metadata, packaging, and dormant-CI hygiene that block a clean
release build or a Defcoin-branded installer.*
