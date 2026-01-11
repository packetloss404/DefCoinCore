Defcoin Core 2.0.0
==================

[![Build Status](https://github.com/packetloss404/defcoin/actions/workflows/build.yml/badge.svg)](https://github.com/packetloss404/defcoin/actions)

https://www.defcoin.io

What is Defcoin?
----------------

Defcoin is a peer-to-peer digital currency that enables instant, low-cost payments
to anyone, anywhere in the world. Defcoin uses Scrypt proof-of-work with 2-minute
block times and a simple, predictable monetary policy.

**Key Features:**
- 2-minute block time (faster than Bitcoin/Litecoin)
- Scrypt proof-of-work algorithm
- 50 DFC initial block reward with halving schedule
- Based on Litecoin 0.21.x codebase for modern features

Defcoin Core is the reference implementation that enables the use of this currency.

For more information, visit the [Defcoin website](https://www.defcoin.io) or the
[original repository](https://github.com/mspicer/defcoin).

Defcoin Tools
---------------------------------

- **https://explorer.defcoin.io**
- **https://pool.defcoin.io**

What's New in Defcoin Core 2.0.0
---------------------------------

Defcoin Core 2.0.0 is a major upgrade rebased from Litecoin 0.15.x to Litecoin 0.21.x, bringing 3+ years of improvements while preserving all Defcoin consensus rules.

### Performance & Efficiency
- **Faster sync**: Improved block download and validation
- **Reduced memory usage**: Better memory management and caching
- **Compact block relay**: Faster block propagation across the network

### Security Fixes
- Fixes for CVE-2023-33297 and CVE-2024-35202
- Improved peer connection handling
- Better DoS protection

### Wallet Improvements
- **Descriptor wallets**: Modern wallet architecture (optional)
- **Improved coin selection**: Smarter UTXO management
- **Hardware wallet support**: Better integration with external signers
- **Full backward compatibility**: Your existing wallet.dat works without changes

### Network & Protocol
- **Updated P2P protocol**: More efficient peer communication
- **Improved Tor support**: Better privacy options
- **BIP155 (addrv2)**: Support for next-gen network addresses

### Developer Features
- **Updated RPC API**: New commands and improved responses
- **Better logging**: More detailed debug information
- **Modern C++17 codebase**: Easier to maintain and extend

### What Stayed the Same
- **Scrypt proof-of-work**: Same mining algorithm
- **2-minute block time**: No change to block schedule
- **720-block difficulty retarget**: Same adjustment period
- **All consensus rules**: 100% compatible with existing chain
- **Wallet compatibility**: Your wallet.dat works without changes

Upgrading from Defcoin Core 1.x
-------------------------------

Defcoin Core 2.0.0 is a major upgrade based on Litecoin 0.21.x. To upgrade:

1. **Backup your wallet** - Copy `wallet.dat` from your Defcoin data directory
2. Close Defcoin Core 1.x
3. Install Defcoin Core 2.0.0
4. On first run, the wallet will automatically reindex the blockchain
5. Your existing wallet and addresses will work without changes

**Data Directory Locations:**
- Windows: `%APPDATA%\Defcoin\`
- Linux: `~/.defcoin/`
- macOS: `~/Library/Application Support/Defcoin/`

License
-------

Defcoin Core is released under the terms of the MIT license. See https://opensource.org/licenses/MIT.

Building
--------

See the build documentation in the `doc` folder:
- [Unix Build Notes](doc/build-unix.md)
- [Windows Build Notes](doc/build-windows.md)
- [macOS Build Notes](doc/build-osx.md)

### Quick Build (Linux)

```bash
./autogen.sh
./configure
make
make install  # optional
```

### Binary Names

- `defcoind` - Daemon
- `defcoin-qt` - GUI wallet
- `defcoin-cli` - Command-line RPC client
- `defcoin-tx` - Transaction utility
- `defcoin-wallet` - Wallet utility

Network Information
-------------------

| Parameter | Mainnet | Testnet |
|-----------|---------|---------|
| P2P Port | 1337 | 31337 |
| RPC Port | 1335 | 31335 |
| Address Prefix | D | m/n |
| Bech32 Prefix | dfc | tdfc |

Development
-----------

The `main` branch contains the latest stable release. Development happens in
feature branches.

- Original Defcoin: https://github.com/mspicer/defcoin
- Defcoin 2.0: https://github.com/packetloss404/defcoin
- Issues: https://github.com/packetloss404/defcoin/issues

Testing
-------

### Automated Testing

Run unit tests with:
```bash
make check
```

Run functional tests with:
```bash
test/functional/test_runner.py
```

### Manual Testing

After building, test sync against the existing Defcoin network:
```bash
./src/defcoind -printtoconsole
```

Credits
-------

Defcoin Core 2.0.0 is based on:
- [Litecoin Core](https://github.com/litecoin-project/litecoin) 0.21.x
- [Bitcoin Core](https://github.com/bitcoin/bitcoin)
- Original [Defcoin](https://github.com/mspicer/defcoin) by mspicer
