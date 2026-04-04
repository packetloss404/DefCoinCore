Defcoin Core
=============

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

- [Dependencies](dependencies.md)
- [macOS Build Notes](build-osx.md)
- [Unix Build Notes](build-unix.md)
- [Windows Build Notes](build-windows.md)
- [FreeBSD Build Notes](build-freebsd.md)
- [OpenBSD Build Notes](build-openbsd.md)
- [NetBSD Build Notes](build-netbsd.md)
- [Gitian Building Guide](gitian-building.md)

Development
---------------------
The Defcoin repo's [root README](/README.md) contains the current project overview, testing entry
points, and contributor starting points.

- [Developer Notes](developer-notes.md)
- [Productivity Notes](productivity.md)
- [Current Defcoin release overview](../README.md)
- [Historical Litecoin release notes archive](litecoin-release-notes/)
- [Translation Process](translation_process.md)
- [Translation Strings Policy](translation_strings_policy.md)
- [JSON-RPC Interface](JSON-RPC-interface.md)
- [Unauthenticated REST Interface](REST-interface.md)
- [Shared Libraries](shared-libraries.md)
- [BIPS](bips.md)
- [Dnsseed Policy](dnsseed-policy.md)
- [Benchmarking](benchmarking.md)

### Resources
* Discuss on GitHub: https://github.com/packetloss404/DefCoinCore

### Miscellaneous
- [Assets Attribution](assets-attribution.md)
- [defcoin.conf Configuration File](bitcoin-conf.md)
- [Files](files.md)
- [Fuzz-testing](fuzzing.md)
- [Codebase recommendations](../recommendations.md)
- [Reduce Memory](reduce-memory.md)
- [Reduce Traffic](reduce-traffic.md)
- [Tor Support](tor.md)
- [Init Scripts (systemd/upstart/openrc)](init.md)
- [ZMQ](zmq.md)
- [PSBT support](psbt.md)

License
---------------------
Distributed under the [MIT software license](/COPYING).
