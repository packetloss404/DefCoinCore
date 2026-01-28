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
- Based on Litecoin 0.21.x codebase with 0.21.4 security patches

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
All security patches from Litecoin 0.21.4 have been backported:
- **CVE-2023-33297**: Remote peer crash vulnerability
- **CVE-2024-35202**: Blocktxn message DoS (FillBlock assertion crash)
- **Mutated blocks fix**: Prevents peers from interfering with block relay
- **Miniupnpc 2.2.2**: Fixes infinite loop vulnerability (local network DoS)
- Improved peer connection handling
- Better DoS protection
- Reduced network fingerprinting

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

Consensus Parameters
--------------------

Defcoin 2.0 maintains full compatibility with the original Defcoin chain. Some newer Bitcoin/Litecoin features are disabled to preserve consensus.

| Parameter | Value | Notes |
|-----------|-------|-------|
| Block Time | 2 minutes | 120 seconds target |
| Difficulty Retarget | 720 blocks | ~1 day |
| PoW Algorithm | Scrypt (1024,1,1) | Memory-hard, ASIC-resistant |
| Initial Block Reward | 50 DFC | |
| Halving Interval | 840,000 blocks | ~3.2 years |
| Max Supply | ~84 million DFC | |

### Enabled Features

| Feature | Status | Since |
|---------|--------|-------|
| P2SH (BIP16) | **Enabled** | Genesis |
| BIP34 (Height in Coinbase) | **Enabled** | Block 710,000 |

### Disabled Features

These features are disabled to maintain chain compatibility with Defcoin 1.x:

| Feature | Status | Reason |
|---------|--------|--------|
| SegWit (BIP141) | Disabled | Never activated on original chain |
| CSV (BIP68/112/113) | Disabled | Never activated on original chain |
| BIP65 (CLTV) | Disabled | Never activated on original chain |
| BIP66 (Strict DER) | Disabled | Never activated on original chain |
| Taproot (BIP340/341) | Disabled | Requires SegWit |
| MWEB (Mimblewimble) | Disabled | New feature, may enable in v2.1+ |

> **Note:** These features were never activated on the original Defcoin network during
> the BIP9 signaling period (2018-2019). Enabling them now would break consensus with
> the existing chain. Future soft-fork activation may be considered for new features.

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

----------------------------------
## To Do

Rename the Defcoin units from mDFC and μDFC in favor of a more flavorful name similar to [Litecoin](https://github.com/litecoin-project/litecoin/blob/master/src/qt/bitcoinunits.cpp#L43).
This is stored in the file [```src/qt/bitcoinunits.cpp```](https://github.com/nah012/defcoin/blob/master/src/qt/bitcoinunits.cpp) on [lines 43 - 45](https://github.com/nah012/defcoin/blob/master/src/qt/bitcoinunits.cpp#L43-L45)
```
    case BTC: return QString("DFC");
    case mBTC: return QString("mDFC");
    case uBTC: return QString::fromUtf8("μDFC");
```
and on [lines 54 - 56](https://github.com/nah012/defcoin/blob/master/src/qt/bitcoinunits.cpp#L54-L56)
```
    case BTC: return QString("Defcoins");
    case mBTC: return QString("Milli-Defcoins (1 / 1" THIN_SP_UTF8 "000)");
    case uBTC: return QString("Micro-Defcoins (1 / 1" THIN_SP_UTF8 "000" THIN_SP_UTF8 "000)");
```

Credits
-------

Defcoin Core 2.0.0 is based on:
- [Litecoin Core](https://github.com/litecoin-project/litecoin) 0.21.x (with 0.21.4 security patches)
- [Bitcoin Core](https://github.com/bitcoin/bitcoin)
- Original [Defcoin](https://github.com/mspicer/defcoin) by mspicer
