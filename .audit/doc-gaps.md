# Defcoin Core — Missing User-Facing Documentation Audit

**Repo:** `D:\repo\DefCoinCore` @ `77343911b` (clean, on `main`)
**Date:** 2026-08-03
**Scope:** gaps the doc-consolidation pass did not cover.

## What I searched

Read in full: `README.md`, `INSTALL.md`, `CONTRIBUTING.md`, `COPYING.md`, `recommendations.md`,
`doc/README.md`, `doc/init.md`, `doc/tor.md`, `doc/TESTNET_GUIDE.md`, `doc/bitcoin-conf.md`,
`doc/files.md`, `doc/release-process.md`, `doc/release-notes.md`, `doc/descriptors.md`,
`doc/psbt.md`, `doc/reduce-memory.md`, `doc/reduce-traffic.md`, `doc/build-unix.md`,
`doc/build-windows.md`, `doc/mweb/mining-changes.md`, `share/examples/defcoin.conf`,
`contrib/README.md`, `contrib/pyminer/README`, `contrib/qos/README.md`,
`contrib/macdeploy/README.md`, `build_msvc/README.md` (excerpt), `depends/README.md` (excerpt),
`src/qt/README.md`, `share/rpcauth/README.md`, root `.github/ISSUE_TEMPLATE/*.md`,
`doc/README_windows.txt`. Ran repo-wide grep for: `SECURITY`, `CHANGELOG`, `CODE_OF_CONDUCT`,
`SUPPORT`, `FUNDING`, `FAQ`, `IRC`, `discord`, `forum`, `matrix`, `telegram`, `explorer`,
`faucet`, `prune`, `snapshot`, `assumeutxo`, `backup`, `recover`, `seed phrase`, `scrypt`,
`getblocktemplate`, `defc014e`, `fbc0b6db`, `magic migration`, `Dockerfile`, `docker-compose`,
`PKGBUILD`, `snapcraft`, `homebrew`, plus the GUI/mining/wallet/pool doc-glob search.

## What I did not do

I did not read the in-tree C++ source (chainparams, init, qt forms) beyond the snippets above;
consensus-activation numbers and `getblocktemplate` field names are quoted from the
in-tree `README.md`, `doc/init.md`, and `doc/mweb/mining-changes.md`. I did not verify
runtime flag names against `defcoind --help`; any new doc should be cross-checked at
build time.

## Prioritized Gap Table

| # | Proposed file | User need (story) | Scope | Priority | Effort |
|---|---|---|---|---|---|
| 1 | `SECURITY.md` (root) | "I'm a researcher and found a vulnerability — where do I send it without it being tweeted?" | Responsible-disclosure email/PGP, supported versions, CVE/coordinated-disclosure timeline, what is in/out of scope, security-update communication channel. 30-80 lines. | **critical** | 1 hour |
| 2 | `doc/magic-migration.md` (or `doc/operator-magic-migration.md`) | "The README says the network flips to defcoin-only on 2026-08-01 — how do I verify my node is ready?" | Cutover date, what `-acceptlegacymagic` does, how to confirm a node is defcoin-only (`getnetworkinfo` / debug log lines), peer UA `/Defcoin` requirement, what to do if you still see `fbc0b6db` peers, expected log noise during the window, rollback notes for pool/merchant operators. | **critical** (cutover is days away) | half a day |
| 3 | `doc/wallet-guide.md` (root or `doc/`) | "I just installed defcoin-qt — how do I receive, send, back up, and restore my wallet?" | GUI walkthrough: first launch, choosing data dir, encrypting wallet, receiving (address/QR), sending (address book, fee slider, coin control), backing up `wallet.dat` and `dumpwallet`, restoring from backup, restoring from `dumpwallet`/`importwallet`, HD-seed behavior, `dumpprivkey`/`importprivkey`, passphrase change, lock/unlock from RPC, the difference between BDB and SQLite wallet files, when to run `-zapwallettxes`. | **critical** | 1 PR-day |
| 4 | `doc/address-types.md` (or section in `wallet-guide.md`) | "Which address type should I actually use on Defcoin mainnet, and why does the README say legacy is the default?" | Side-by-side: legacy Base58 (`D…`), P2SH-SegWit (`3…` / 0x32), Bech32 (`dfc1…`). When each is appropriate, fee/payload differences, ecosystem-support status, the `getnewaddress "" "legacy\|p2sh-segwit\|bech32"` choices, the `addwitnessaddress`/`setwalletflag` migration, when to migrate an existing legacy wallet, descriptor-wallet tradeoff, what is *not* available (Taproot). | high | 1 PR-day |
| 5 | `doc/mining-pools.md` | "I'm setting up a Stratum pool for Defcoin mainnet — what `getblocktemplate` parameters and coinbase rules apply?" | Scrypt mainnet constants (target 2 min, 50 DFC subsidy, 840k halving, 720 retarget), `getblocktemplate` rules array for Defcoin (no `mweb` rule — explicit "do not pass it"), `submitblock` format, AuxPoW status (is it on Defcoin mainnet? — verify and document), address-format recommendation for coinbase, `getmininginfo`/`getnetworkhashps`/`getdifficulty`, mempool/min-fee policy defaults, MWEB warning copied from `doc/mweb/mining-changes.md`. Link to upstream Litecoin mining docs only where the behavior matches. | high | 1 PR-day |
| 6 | `doc/community.md` (or `SUPPORT.md`) | "I have a question that isn't an issue — where do I actually talk to people?" | Where to get help (chat, forum, mailing list, IRC, Discord, Matrix — whatever is real), where the block explorer lives, where the testnet faucet is, how to get on the announce list, social links, contact for security (link to `SECURITY.md`), no-promising-of-support disclaimer. | high | 1 hour (just list real links) |
| 7 | `doc/pruning-storage.md` (or `doc/operators/storage.md`) | "My VPS is out of disk — can I prune Defcoin? What breaks?" | `-prune=N` semantics, the `prune=550` floor, interaction with `-txindex=1` and `-blockfilterindex=*`, expected size at current tip, reindex after switching to/from pruned, no-snapshot caveat, `-assumevalid` interaction, restoring from pruned to full, when to use `-reindex-chainstate` vs `-reindex`. | high | half a day |
| 8 | `doc/upgrading-from-1x.md` (extend or replace the 7-line section in root `README.md`) | "I'm coming from the old `mspicer/defcoin` 1.x client — what breaks, what's the same?" | Where the data directory moved, `wallet.dat` compatibility, chainparams delta, what 1.x RPCs no longer exist, `-repairwitnessfromheight` walkthrough for post-SegWit-activation nodes, first-start behavior change, default `-dbcache` and `-maxmempool` differences, log message expectations, GUI migration path, recovery from a stale `peers.dat`. | high | half a day |
| 9 | `doc/build-troubleshooting.md` | "My build is failing with `cannot find -lboost` / BDB version error / MSVC vcpkg issue — what now?" | A. Boost version pin and how to build from `depends/`. B. BDB 4.8 dance (`./contrib/install_db4.sh`, `--with-incompatible-bdb`). C. MSVC: vcpkg, Qt static build, "MSB8036 SDK not found". D. OpenBSD `ulimit -n 2048` and `BOOST_STACKTRACE`. E. Apple Silicon (M1/M2) — Xcode CLT, `--with-secure-cxx-flags`, Homebrew prefix for Qt, `arm64` vs `x86_64` linker. F. libfmt / libsqlite "header not found". G. `make check` failures and the supported Python versions. | medium | half a day |
| 10 | `Dockerfile` (root) + `docker-compose.yml` + `doc/docker.md` | "I'd like to run a Defcoin node as a container — is there an image?" | A simple multi-stage `Dockerfile` (build stage + slim runtime with `defcoind` + `defcoin-cli` only) published to GHCR via the existing release workflow, plus a `docker-compose.yml` showing mainnet with `~/.defcoin` volume mount, and a 20-line `doc/docker.md` that says "no official image yet; build with the Dockerfile" if maintainers want to defer the image. | medium | 1 PR-day |
| 11 | `packaging/README.md` (new folder) and one canonical package per distro: e.g. `packaging/homebrew/defcoin.rb`, `packaging/aur/PKGBUILD`, `packaging/snap/snapcraft.yaml` | "How do I install Defcoin via Homebrew / AUR / Snap / Flatpak?" | At minimum: a `packaging/README.md` that lists which distros have community packages and links to them, plus stubs (or actual files) for one Homebrew formula, one AUR PKGBUILD, one Snap. The tree has only `contrib/debian/copyright` — no real Debian rules, no RPM, no Snap, no AUR, no Flatpak, no Homebrew. | medium | 1 PR-day (community) |
| 12 | `doc/faq.md` | "I keep getting 'No block source available' / 'Cannot obtain lock on data directory' / wallet won't open" | A real FAQ. `TESTNET_GUIDE.md` already lists two of these for the testnet path; promote them, add the mainnet equivalents, add: GUI won't start on Wayland, anti-virus quarantine, Windows SmartScreen, macOS Gatekeeper "developer cannot be verified" (link to `contrib/macdeploy/README.md`), "wallet.dat locked by another process", "I sent to wrong address", "my node is stuck on a stale chain tip", "fee too low to confirm". | medium | half a day |
| 13 | Rewrite `src/qt/README.md` (currently fully upstream Litecoin content — references `litecoin-qt`, `LitecoinQT`, the project name `Litecoin`) | "I want to understand the Qt codebase before sending a PR." | This file is a developer-facing repo doc, not user-facing, but the rebrand pass missed it entirely — it still says "litecoin-qt" and "LitecoinQT" throughout. Re-stamp binary paths, class names, and the Qt Creator section for `defcoin-qt`. | medium | 1 hour |
| 14 | `CODE_OF_CONDUCT.md` (root) | "What's the behavior bar in this project's spaces?" | Adopt Contributor Covenant (or write 1 page of project norms) covering the GitHub repo, any chat, the issue tracker. Tie to enforcement in `CONTRIBUTING.md`. | medium | 1 hour |
| 15 | `doc/rpc-quickstart.md` (root) | "I want to query the chain over RPC — what are the right cookie / `rpcauth` / curl commands?" | Reuse the existing `share/rpcauth/rpcauth.py` flow but show the *whole* first call end-to-end: create user, write `rpcauth` line, start daemon, `curl --cookie .cookie … getblockchaininfo`, switch to cookie, switch to TLS, common first RPC mistakes. Currently the `bitcoin-conf.md` and `init.md` cover bits but never show one complete end-to-end call. | medium | half a day |
| 16 | `doc/operators/snapshot-sync.md` | "Is there a UTXO snapshot I can `loadtxoutset` to skip IBD?" | Honest statement: 0.21.x has no `loadtxoutset` / `assumeutxo` (verify in `src/node/utxo_snapshot.h`); the `dumptxoutset` RPC exists but is for the operator's own snapshot. Document current options: full IBD, `bootstrap.dat` if/when provided, `txindex=0` + `dbcache=4096` for faster sync, third-party snapshot scripts (with a "use at your own risk, verify the chainstate hash" warning). | low | research-then-decide |
| 17 | `CHANGELOG.md` (root) | "What changed between 26.6.7 and 26.6.8?" | Generate from the `doc/release-notes/` set or link out; one entry per release version with a date and a short summary. Currently the only way to read the change history is to read each per-version note. | low | 1 hour |
| 18 | `doc/scale-and-bottlenecks.md` (or a `doc/operators/` section) | "I'm seeing 100% CPU on a 16-core box — what knob should I turn?" | Pull together the inherited `reduce-memory.md` + `reduce-traffic.md` + the `-rpcthreads`, `-par`, `-maxconnections`, `MALLOC_ARENA_MAX`, and the pruning interaction into one operator tuning page. Add a "before you tune" section about what to actually measure (`getnetworkinfo`, `getmempoolinfo`, `getrpcinfo`). | low | 1 PR-day |

## Already Adequate

These existed before the consolidation pass and cover the need — do not re-request:

- `INSTALL.md` — install / first-run / upgrade / validate path, with default data dirs and port table.
- `README.md` (root) — current state of the project, key network facts, consensus activation table, magic migration pointer, upgrade-from-1.x stub (7 lines), build quickstart, contributor start.
- `doc/README.md` — full index with the per-doc status table.
- `doc/init.md` + `contrib/init/*.service` / `.openrc` / `.openrcconf` / `.conf` / `.init` — daemon packaging and service scripts (now defcoin-rebranded).
- `doc/tor.md` — Tor / hidden service / control-port walkthrough (already defcoin-ized, ports 1337 / 31337).
- `doc/TESTNET_GUIDE.md` — build, regtest, testnet setup, common testnet errors.
- `doc/bitcoin-conf.md` + `share/examples/defcoin.conf` — configuration file format, defaults, and template.
- `doc/files.md` — full data-directory layout including BDB vs SQLite wallets.
- `doc/JSON-RPC-interface.md` / `doc/REST-interface.md` / `doc/zmq.md` — daemon interface references (rebranded, correct ports).
- `doc/release-process.md` + `doc/release-notes.md` — release workflow and current-release notes template (defcoin-adapted).
- `doc/tor.md` / `doc/reduce-memory.md` / `doc/reduce-traffic.md` — inherited-but-banner-flagged operator tuning.
- `doc/mweb/mining-changes.md` — correctly banner-flagged as "not active on Defcoin mainnet"; that warning is the doc.
- `doc/descriptors.md` / `doc/psbt.md` — inherited feature docs, banner-flagged with the "legacy wallets remain the default" note.
- `doc/man/defcoin-*.1` + `doc/man/Makefile.am` + `contrib/devtools/gen-manpages.sh` — man page files renamed and generator switched (the *content* still needs a regen — see recommendation #7 in `recommendations.md`).
- `CONTRIBUTING.md` — already defcoin-branded (lead maintainer, defcoin-specific PR-area prefixes), PR workflow, peer-review language, backport metadata, decision-making.
- `build_msvc/README.md` + the freebsd/netbsd/openbsd/unix/windows/macos build docs — all rebranded.
- `doc/release-notes/` (Bitcoin Core) and `doc/litecoin-release-notes/` (Litecoin Core) — correctly labeled upstream archival; per-release notes for the next Defcoin cut will land in the first folder.
- `.github/ISSUE_TEMPLATE/` — bug / feature / GUI / good-first-issue templates exist; `gui_issue.md` is actually named for the user need.
- `share/rpcauth/rpcauth.py` + `share/rpcauth/README.md` — RPC auth credential generator.
- `depends/README.md` and `depends/packages.md` — cross-compilation host list and package pins.

## Quick Wins (highest priority, lowest effort)

1. **Write `SECURITY.md`** (1 hour) — addresses the largest open hole. Even a stub ("email security@… , PGP fingerprint, supported versions = 26.6.x and 0.21.x") is better than nothing; GitHub's security policy UI surfaces this file directly.
2. **Write `doc/magic-migration.md`** (half a day) — the network cuts over to defcoin-only magic in days. The current README pointer is fine for developers but operators need a runbook.
3. **Add `doc/community.md`** (1 hour) — collect the real links (chat, explorer, faucet) in one place; without it, every GitHub visitor has to dig. Ask a maintainer what links are real, then write them down.
4. **Rebrand `src/qt/README.md`** (1 hour) — currently the only file under `src/qt/` that still names `litecoin-qt` / `LitecoinQT` end-to-end. The rebrand pass missed it.
5. **Promote the FAQ inline in `doc/TESTNET_GUIDE.md` to `doc/faq.md`** (half a day) — mainnet equivalents of "No block source available", "Cannot obtain lock on data directory", plus a few new ones (Gatekeeper, SmartScreen, "stuck on stale tip"). Most-used doc by far for non-developer users.

## Notes on what the consolidation pass correctly scoped out

- "Resolve runtime identity and packaging drift" (`recommendations.md` #1) covers the
  `bitcoin-config.h` / `bitcoin.sln` / AM_CONDITIONAL names — out of scope here.
- `recommendations.md` #3 ("align wallet defaults with Defcoin mainnet policy") is the
  *code* side; this audit is the *doc* side that should land first or alongside it.
- The `doc/mweb/mining-changes.md` banner is the right call — a *replacement* mining guide
  (gap #5) is the real answer, not a rewrite of the MWEB one.
- `recommendations.md` #7 ("regenerate the man pages") is separate from these gaps; the man
  page *filenames* are already correct.
