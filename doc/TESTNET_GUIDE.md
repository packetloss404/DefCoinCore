# Defcoin Testnet & Regtest Guide

## Building Defcoin

```bash
cd /home/user/defcoin

# Generate build files
./autogen.sh

# Configure (without GUI for faster build)
./configure --without-gui

# Or with Qt wallet
./configure

# Build (use all CPU cores)
make -j$(nproc)
```

Binaries will be in `./src/`:
- `defcoind` - daemon
- `defcoin-cli` - command line interface
- `defcoin-qt` - GUI wallet (if built with Qt)
- `defcoin-tx` - transaction utility

---

## Option 1: Regtest (Easiest for Development)

Regtest is a private blockchain where you control everything. Blocks are mined instantly.

### Start Regtest Daemon

```bash
./src/defcoind -regtest -daemon
```

### Create Wallet and Mine Coins

```bash
# Create a wallet
./src/defcoin-cli -regtest createwallet "test"

# Get a new address
./src/defcoin-cli -regtest getnewaddress
# Example output: mtzR7Ygh7ukJHnCqVCLoAoV4K5vEqsXrJn

# Mine 101 blocks (need 100 confirmations for coinbase maturity)
./src/defcoin-cli -regtest generatetoaddress 101 mtzR7Ygh7ukJHnCqVCLoAoV4K5vEqsXrJn

# Check balance (should be 50 DFC from first mature block)
./src/defcoin-cli -regtest getbalance
```

### Send Transactions

```bash
# Send to another address
./src/defcoin-cli -regtest sendtoaddress <address> 10.0

# Mine a block to confirm
./src/defcoin-cli -regtest generatetoaddress 1 <your-address>
```

### Stop Regtest

```bash
./src/defcoin-cli -regtest stop
```

### Regtest Data Location

```
~/.defcoin/regtest/
```

---

## Option 2: Testnet (Multi-Node Testing)

Testnet simulates the real network with actual proof-of-work mining.

### Configuration File

Create `~/.defcoin/defcoin.conf`:

```ini
# Network
testnet=1

# RPC settings
server=1
rpcuser=defcoinrpc
rpcpassword=changethispassword
rpcallowip=127.0.0.1

# Debugging (optional)
debug=net
debug=rpc

# Accept connections
listen=1

# Add known nodes (replace with actual IPs)
# addnode=192.168.1.100:31337
# addnode=192.168.1.101:31337
```

### Start First Node (Seed Node)

```bash
# Start daemon
./src/defcoind -testnet -daemon

# Check it's running
./src/defcoin-cli -testnet getblockchaininfo
```

### Start Second Node (Connect to First)

On another machine or terminal:

```bash
./src/defcoind -testnet -daemon -addnode=<first-node-ip>:31337

# Verify connection
./src/defcoin-cli -testnet getpeerinfo
```

### Mining on Testnet

```bash
# Get address
./src/defcoin-cli -testnet getnewaddress

# Mine (requires actual CPU work - will take time)
./src/defcoin-cli -testnet generatetoaddress 1 <address>
```

Note: Testnet uses real Scrypt PoW, so mining takes actual time unlike regtest.

### Testnet Ports

| Service | Port |
|---------|------|
| P2P | 31337 |
| RPC | 31335 |

### Testnet Data Location

```
~/.defcoin/testnet4/
```

---

## Option 3: Signet (Not Configured)

Signet is a centrally-signed testnet. Not currently configured for Defcoin.

---

## Useful Commands

### Blockchain Info

```bash
./src/defcoin-cli -testnet getblockchaininfo
./src/defcoin-cli -testnet getblockcount
./src/defcoin-cli -testnet getbestblockhash
```

### Network Info

```bash
./src/defcoin-cli -testnet getnetworkinfo
./src/defcoin-cli -testnet getpeerinfo
./src/defcoin-cli -testnet getconnectioncount
```

### Wallet Info

```bash
./src/defcoin-cli -testnet getwalletinfo
./src/defcoin-cli -testnet listunspent
./src/defcoin-cli -testnet listtransactions
```

### Manual Node Connection

```bash
./src/defcoin-cli -testnet addnode <ip>:31337 add
./src/defcoin-cli -testnet addnode <ip>:31337 remove
```

---

## Troubleshooting

### "No block source available"

No peers connected. Add nodes manually:
```bash
./src/defcoin-cli -testnet addnode <ip>:31337 onetry
```

### "Cannot obtain lock on data directory"

Another instance is running. Stop it first:
```bash
./src/defcoin-cli -testnet stop
# or
pkill -9 defcoind
```

### Check Debug Log

```bash
tail -f ~/.defcoin/testnet4/debug.log
```

### Reset Testnet Data

```bash
./src/defcoin-cli -testnet stop
rm -rf ~/.defcoin/testnet4
./src/defcoind -testnet -daemon
```

---

## Network Topology for Testing

### Minimum Setup (2 Nodes)

```
[Node A] <----> [Node B]
  Miner          Wallet
```

### Better Setup (3+ Nodes)

```
        [Seed Node]
        /    |    \
   [Node A] [Node B] [Node C]
```

### Docker Setup (Future)

Consider creating Docker containers for easy multi-node testing.

---

## Testnet vs Regtest vs Mainnet

| Feature | Regtest | Testnet | Mainnet |
|---------|---------|---------|---------|
| Mining difficulty | None (instant) | Real Scrypt | Real Scrypt |
| Coins have value | No | No | Yes |
| Need peers | No | Yes | Yes |
| Block time | Instant | ~2 min | ~2 min |
| Port | 19444 | 31337 | 1337 |
| Data dir | regtest/ | testnet4/ | .defcoin/ |
| Use case | Development | Integration testing | Production |
