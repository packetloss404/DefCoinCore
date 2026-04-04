Defcoin Core 2.0.0
==================

[![Build Status](https://github.com/packetloss404/DefCoinCore/actions/workflows/build.yml/badge.svg)](https://github.com/packetloss404/DefCoinCore/actions)

Defcoin Core is the reference full node and wallet implementation for the Defcoin network.
This tree is based on Litecoin 0.21.x with selected security backports, while preserving the
Defcoin mainnet rules used by the existing chain.

- Website: https://www.defcoin.io
- Repository: https://github.com/packetloss404/DefCoinCore
- Historical Defcoin source: https://github.com/mspicer/defcoin

Key Network Facts
-----------------

- Proof of work: Scrypt
- Target block time: 2 minutes
- Difficulty retarget interval: 720 blocks
- Subsidy halving interval: 840,000 blocks
- Initial block reward: 50 DFC

Important Compatibility Notes
-----------------------------

This branch intentionally keeps Defcoin mainnet aligned with the historical chain rather than
blindly enabling all newer Litecoin features.

- Mainnet does not treat SegWit, CSV, CLTV, BIP66, Taproot, or MWEB as active consensus features.
- Use legacy Base58 addresses on mainnet. Do not assume Bech32 or P2SH-SegWit compatibility.
- Descriptor wallet plumbing exists in the tree, but legacy wallets remain the default and GUI
  descriptor-wallet creation is not enabled.
- PSBT workflows are available, but dedicated external-signer integration is not documented as a
  supported Defcoin feature in this release.
- `-signet` is not currently supported in this branch.

Network Information
-------------------

| Network | P2P Port | RPC Port | Data Dir Suffix | Address Prefix |
|---------|----------|----------|-----------------|----------------|
| Mainnet | 1337 | 9332 | `(datadir root)` | `D` |
| Testnet | 31337 | 19332 | `testnet4/` | `m` / `n` |
| Regtest | 19444 | 19443 | `regtest/` | regtest only |

Build And Install
-----------------

Start with the top-level install guide, then use the platform-specific docs.

- [INSTALL.md](INSTALL.md)
- [Documentation index](doc/README.md)
- [Unix build notes](doc/build-unix.md)
- [Windows build notes](doc/build-windows.md)
- [macOS build notes](doc/build-osx.md)

Quick headless build on Unix-like systems:

```bash
./autogen.sh
./configure
make
```

If you want `defcoin-qt`, install the Qt dependencies described in the platform build notes first.

Binary Names
------------

- `defcoind`: daemon
- `defcoin-qt`: GUI wallet
- `defcoin-cli`: command-line RPC client
- `defcoin-tx`: transaction utility
- `defcoin-wallet`: wallet utility

Upgrading From 1.x
------------------

Before upgrading from older Defcoin releases:

1. Back up the wallet you actually use. That may be `wallet.dat` or a wallet directory under `wallets/`.
2. Record your current data directory location.
3. Shut down the old node cleanly.
4. Install the new binaries.
5. Expect the first startup to take longer while wallet and chain state are checked, rescanned, or reindexed as needed.
6. On pruned nodes, a manual `-reindex` may still be required in some upgrade paths.

Default data directories:

- Windows: `%APPDATA%\Defcoin\`
- Linux: `~/.defcoin/`
- macOS: `~/Library/Application Support/Defcoin/`

Testing
-------

Build the project before running the test suite so generated files such as `test/config.ini` exist.

Common validation entry points:

```bash
make check
test/functional/test_runner.py
test/util/bitcoin-util-test.py
test/lint/lint-all.sh
```

See `test/README.md`, `src/test/README.md`, and `doc/fuzzing.md` for deeper test coverage.

New Contributors Start Here
---------------------------

1. Read [INSTALL.md](INSTALL.md) for the supported build and run paths.
2. Use [doc/README.md](doc/README.md) to find your platform build notes.
3. Run the smoke-test commands in the testing section above.
4. Read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.
5. Review [recommendations.md](recommendations.md) for the current codebase and docs follow-up list.

Project Status
--------------

This repository still contains inherited Bitcoin/Litecoin naming in some tooling, tests, packaging,
and archival documentation. The top-level docs in this pass were updated to reflect current Defcoin
behavior; remaining cleanup work is tracked in [recommendations.md](recommendations.md).

License
-------

Defcoin Core is distributed under the MIT license. See [COPYING](COPYING) for the canonical license
text and [COPYING.md](COPYING.md) for a short project-specific licensing note.

Credits
-------

Defcoin Core 2.0.0 builds on work from:

- [Defcoin](https://github.com/mspicer/defcoin)
- [Litecoin Core](https://github.com/litecoin-project/litecoin)
- [Bitcoin Core](https://github.com/bitcoin/bitcoin)
