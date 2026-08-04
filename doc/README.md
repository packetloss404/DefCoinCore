Defcoin Core
=============

This is the documentation index for Defcoin Core. The `doc/` tree mixes
current Defcoin Core guidance with reference material inherited from
upstream Bitcoin Core / Litecoin Core; the **Doc Status** section
below classifies each file so you know which is which. Each
inherited file also carries a `Status: inherited from upstream`
banner at the top — the index is just a quick overview.

Setup
---------------------
Defcoin Core is the reference full node and wallet for the Defcoin network. It downloads and, by
default, stores the full blockchain, so the initial sync can take a while depending on your system
and network connection.

To download Defcoin Core, visit [defcoin.io](https://www.defcoin.io/).

Running
---------------------
The following are some helpful notes on how to run Defcoin Core on your native platform.

### Unix

Unpack the files into a directory and run:

- `bin/defcoin-qt` (GUI) or
- `bin/defcoind` (headless)

### Windows

Unpack the files into a directory, and then run defcoin-qt.exe.

### macOS

Drag Defcoin Core to your applications folder, and then run Defcoin Core.

### Need Help?

* Start with the top-level [install guide](../INSTALL.md).
* See the [root README](../README.md) for current network and testing notes.
* Open an issue at https://github.com/packetloss404/DefCoinCore/issues

Building
---------------------
The following are developer notes on how to build Defcoin Core on your native platform. They are not complete guides, but include notes on the necessary libraries, compile flags, etc.

- [Dependencies](dependencies.md) *(inherited — see status table)*
- [macOS Build Notes](build-osx.md)
- [Unix Build Notes](build-unix.md)
- [Windows Build Notes](build-windows.md)
- [FreeBSD Build Notes](build-freebsd.md)
- [OpenBSD Build Notes](build-openbsd.md)
- [NetBSD Build Notes](build-netbsd.md)
- [Gitian Building Guide](gitian-building.md) *(upstream redirect only)*

Development
---------------------
The Defcoin repo's [root README](../README.md) contains the current project overview, testing entry
points, and contributor starting points.

- [Developer Notes](developer-notes.md) *(inherited)*
- [Productivity Notes](productivity.md) *(inherited)*
- [Current Defcoin release overview](../README.md)
- [Translation Process](translation_process.md) *(inherited)*
- [Translation Strings Policy](translation_strings_policy.md) *(inherited)*
- [JSON-RPC Interface](JSON-RPC-interface.md)
- [Unauthenticated REST Interface](REST-interface.md)
- [Shared Libraries](shared-libraries.md) *(inherited)*
- [BIPs](bips.md) *(inherited table, with Defcoin activation heights in the README)*
- [Dnsseed Policy](dnsseed-policy.md) *(inherited)*
- [Benchmarking](benchmarking.md) *(inherited)*
- [Multiprocess](multiprocess.md) *(inherited feature, rebranded binary names)*
- [Testnet Guide](TESTNET_GUIDE.md)

### Resources
* Discuss on GitHub: https://github.com/packetloss404/DefCoinCore

### Miscellaneous
- [Assets Attribution](assets-attribution.md) *(inherited — attribution list still upstream)*
- [defcoin.conf Configuration File](bitcoin-conf.md)
- [Files](files.md)
- [Fuzz-testing](fuzzing.md) *(inherited)*
- [Codebase recommendations](../recommendations.md)
- [Reduce Memory](reduce-memory.md) *(inherited)*
- [Reduce Traffic](reduce-traffic.md) *(inherited)*
- [Tor Support](tor.md)
- [Init Scripts (systemd/upstart/openrc)](init.md)
- [ZMQ](zmq.md)
- [PSBT support](psbt.md) *(inherited)*
- [Descriptors](descriptors.md) *(inherited feature, with Defcoin default-wallet note)*

Release documentation
---------------------
- [Release Process](release-process.md) *(adapted from upstream, with Defcoin repos/binary names)*
- [Release Notes (current template)](release-notes.md)
- [Historical Bitcoin Core release notes](release-notes/) *(upstream archival)*
- [Historical Litecoin Core release notes](litecoin-release-notes/) *(upstream archival)*
- [Litecoin Core 0.21.1 notes](release-notes-litecoin.md) *(upstream archival)*

Service / daemon reference
--------------------------
- [Init scripts source](../contrib/init/README.md) (systemd / Upstart / OpenRC / CentOS / macOS)
- [mweb/mining-changes.md](mweb/mining-changes.md) *(upstream Litecoin MWEB guide — **not active on Defcoin mainnet**; source paths exist but the feature is not a Defcoin consensus rule)*

License
---------------------
Distributed under the [MIT software license](../COPYING).

Doc Status
---------------------

| Doc | Status |
| --- | --- |
| `init.md` | Defcoin current (rewritten for daemon / paths / user) |
| `JSON-RPC-interface.md` | Defcoin current (rebranded binary names, updated ports) |
| `REST-interface.md` | Defcoin current (rebranded, port/defcoin-conf reference) |
| `zmq.md` | Defcoin current (rebranded) |
| `tor.md` | Defcoin current (already defcoin-ized) |
| `TESTNET_GUIDE.md` | Defcoin current (already defcoin-ized) |
| `bitcoin-conf.md` | Defcoin current (defcoin.conf reference) |
| `files.md` | Defcoin current (already defcoin-ized) |
| `build-unix.md` | Defcoin current (already defcoin-ized) |
| `build-windows.md` | Defcoin current (already defcoin-ized) |
| `build-osx.md` | Defcoin current (already defcoin-ized) |
| `build-freebsd.md` | Defcoin current (rebranded) |
| `build-netbsd.md` | Defcoin current (rebranded) |
| `build-openbsd.md` | Defcoin current (rebranded) |
| `build_msvc/README.md` | Defcoin current (rebranded) |
| `release-process.md` | Defcoin-adapted upstream (structure inherited, Defcoin repo/binary names) |
| `release-notes.md` | Defcoin current (in-progress release template) |
| `release-notes-litecoin.md` | Upstream archival (Litecoin 0.21.1) |
| `release-notes/` (per-version) | Upstream archival (Bitcoin Core history) |
| `litecoin-release-notes/` (per-version) | Upstream archival (Litecoin Core history) |
| `man/Makefile.am`, `man/*.1` | Defcoin current (file names + generator) |
| `contrib/init/*` | Defcoin current (renamed defcoind.*) |
| `contrib/devtools/gen-manpages.sh` | Defcoin current (DEFCOIND env vars, defcoin binary paths) |
| `developer-notes.md` | Inherited (banner added) |
| `productivity.md` | Inherited (banner added) |
| `shared-libraries.md` | Inherited (banner added) |
| `dnsseed-policy.md` | Inherited (banner added) |
| `benchmarking.md` | Inherited (banner added) |
| `reduce-memory.md` | Inherited (banner added, defcoind reference) |
| `reduce-traffic.md` | Inherited (banner added) |
| `translation_process.md` | Inherited (banner added) |
| `translation_strings_policy.md` | Inherited (banner added) |
| `fuzzing.md` | Inherited (banner added) |
| `psbt.md` | Inherited (banner added, with defcoin default-wallet note) |
| `descriptors.md` | Inherited (banner added, with defcoin default-wallet note) |
| `multiprocess.md` | Inherited feature, defcoin binary names |
| `mweb/mining-changes.md` | Upstream Litecoin reference, **not active on Defcoin mainnet** |
| `assets-attribution.md` | Inherited (attribution list still upstream) |
| `bips.md` | Inherited table, Defcoin activation heights in README |
| `dependencies.md` | Inherited table, Defcoin-specific deps in INSTALL/README |
| `gitian-building.md` | Upstream redirect only |
| `README_doxygen.md` | Inherited (doxygen config reference) |
| `Doxyfile.in` | Generated; reflects current package name |
