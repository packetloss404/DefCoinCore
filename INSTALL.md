Installing And Building Defcoin Core
====================================

This file is the shortest path through the repository depending on what you want to do.

Choose A Path
-------------

- Run prebuilt binaries: read [doc/README.md](doc/README.md) for platform-specific run notes.
- Build from source: use the platform build guides linked below.
- Contribute code: build first, then read [CONTRIBUTING.md](CONTRIBUTING.md).
- Validate a build: use the testing commands in the "Validate The Build" section below.

Platform Build Guides
---------------------

- [Unix build notes](doc/build-unix.md)
- [Windows build notes](doc/build-windows.md)
- [macOS build notes](doc/build-osx.md)
- [FreeBSD build notes](doc/build-freebsd.md)
- [OpenBSD build notes](doc/build-openbsd.md)
- [NetBSD build notes](doc/build-netbsd.md)
- [Dependencies overview](doc/dependencies.md)

Quick Unix Build
----------------

```bash
./autogen.sh
./configure
make
```

That produces the core binaries under `src/`. If you want the GUI wallet, install the Qt
dependencies described in the platform build notes before configuring.

Installed / built binaries:

- `defcoind`
- `defcoin-qt`
- `defcoin-cli`
- `defcoin-tx`
- `defcoin-wallet`

First Run And Upgrades
----------------------

- Default data directories:
  - Windows: `%APPDATA%\Defcoin\`
  - Linux: `~/.defcoin/`
  - macOS: `~/Library/Application Support/Defcoin/`
- Back up the wallet you actually use before upgrading. That may be `wallet.dat` or a wallet
  directory under `wallets/`.
- The first startup after an upgrade may take longer while databases are checked, rescanned, or
  reindexed as needed.
- On pruned nodes, a manual `-reindex` may still be required in some upgrade paths.

Operational Notes
-----------------

- Mainnet P2P port: `1337`
- Mainnet RPC port: `9332`
- Testnet P2P port: `31337`
- Testnet RPC port: `19332`
- Signet is not currently supported in this branch.
- Mainnet should be treated as a legacy-address network in this release; do not assume SegWit or
  Taproot interoperability.

Validate The Build
------------------

Run these after building:

```bash
make check
test/functional/test_runner.py
test/util/bitcoin-util-test.py
test/lint/lint-all.sh
```

See also:

- [test/README.md](test/README.md)
- [src/test/README.md](src/test/README.md)
- [doc/fuzzing.md](doc/fuzzing.md)

Further Reading
---------------

- [README.md](README.md)
- [Documentation index](doc/README.md)
- [Configuration file reference](doc/bitcoin-conf.md)
- [Init / service notes](doc/init.md)
- [Tor support](doc/tor.md)
