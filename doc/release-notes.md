Defcoin Core version TBD
========================

This is the release-notes template for the next Defcoin Core release. The
structure is inherited from the upstream Litecoin Core release-notes
format and has been retargeted to Defcoin Core. Replace the placeholder
text below with content for the actual release.

A draft lives in `doc/release-notes.md` and the historical release-notes
archive lives under `doc/release-notes/`. Per-release archive notes are
copied into that directory at release time; older release notes for
upstream Litecoin Core are kept under `doc/litecoin-release-notes/` for
archival reference only and are not part of the Defcoin Core release
history.

Please report bugs using the issue tracker at GitHub:

  <https://github.com/packetloss404/DefCoinCore/issues>

To receive security and update notifications, please subscribe to:

  <https://github.com/packetloss404/DefCoinCore/security/advisories>


How to upgrade
==============

If you are running an older Defcoin Core release, shut it down. Wait until it
has completely shut down (this can take a few minutes for older versions),
then follow the steps for your platform in [INSTALL.md](../INSTALL.md).

Default data directories:

- Windows: `%APPDATA%\Defcoin\`
- Linux: `~/.defcoin/`
- macOS: `~/Library/Application Support/Defcoin/`


Compatibility
==============

Defcoin Core is supported on the same operating systems and architectures
as the upstream Litecoin Core line it was forked from. The classic Qt
Widgets client (`defcoin-qt`) is in maintenance mode — see the project
[README](../README.md) for the current support status.

Known issues are tracked in the issue tracker:

  <https://github.com/packetloss404/DefCoinCore/issues>


Notable changes
===============

Build / CI
----------

- _TODO: list build/CI changes._

P2P / network
-------------

- _TODO: list P2P and networking changes._

Wallet
------

- _TODO: list wallet changes._

RPC / CLI
---------

- _TODO: list RPC and CLI changes._

GUI (defcoin-qt)
----------------

- _TODO: list GUI changes. The Qt wallet is in maintenance mode, so
  expect only fix-level changes here._

Other
-----

- _TODO: list other changes._

Credits
=======

Thanks to everyone who contributed to this release, including the upstream
Bitcoin Core and Litecoin Core projects whose work this codebase builds on.
