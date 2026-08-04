# Defcoin Core — Consensus / Activation / RPC-Surface Audit

**Tree:** `D:\repo\DefCoinCore` @ `77343911bd3afd8b48287b838e615e72abb6b537` (clean, on `main`)
**Scope:** Read-only verification of the 26.6.8l line. No build was run; no node started.

## Preamble

This audit verifies that the Defcoin Core source tree enforces the consensus
rules documented in `README.md`, does not accidentally activate any upstream
softfork (Taproot, MWEB, Signet) on mainnet, and does not surface RPC fields
that would mislead operators into thinking an inactive feature is on.

What was checked:

- `src/chainparams.cpp` — `CMainParams`, `CTestNetParams`, `CRegTestParams`,
  and the `-signet` dispatch path.
- `src/chainparams.h`, `src/consensus/params.h`, `src/versionbitsinfo.cpp` —
  the consensus parameter struct and deployment registry.
- `src/validation.cpp`, `src/pow.cpp` — block-acceptance, retarget, subsidy,
  and time-warp.
- `src/script/interpreter.cpp`, `src/policy/policy.cpp` — script flag wiring
  and standardness.
- `src/primitives/block.h`, `src/primitives/transaction.h` — MWEB/Taproot
  field presence and serialization.
- `src/net.cpp`, `src/net.h`, `src/net_processing.cpp` — magic negotiation
  and user-agent filtering.
- `src/init.cpp` — service-bit advertisement.
- `src/mweb/*` and `src/libmw/*` — build presence and runtime gating.
- `src/rpc/*` and `src/zmq/*` — RPC / ZMQ surface for MWEB / Taproot /
  Signet fields.

**Actual mainnet activation heights found in `chainparams.cpp:68-92`:**

| Rule                | Code value |
|---------------------|------------|
| BIP34Height         | 828,326    |
| BIP65Height         | 1,828,326  |
| BIP66Height         | 1,828,326  |
| CSVHeight           | 903,168    |
| SegwitHeight        | 903,168    |
| `DEPLOYMENT_TAPROOT`| `NEVER_ACTIVE`, nStartHeight/nTimeoutHeight = int::max |
| `DEPLOYMENT_MWEB`   | `NEVER_ACTIVE`, nStartHeight/nTimeoutHeight = int::max |

Every documented mainnet height matches the code. Taproot and MWEB are
properly off. There is no `TaprootHeight` / `MWEBHeight` field on
`Consensus::Params` — the codebase uses the `BIP9Deployment` struct
(`vDeployments[DEPLOYMENT_TAPROOT / DEPLOYMENT_MWEB]`) inherited from
upstream, and the only way these get a real start is by changing the
constants in `chainparams.cpp`.

**Overall confidence: high.** The MWEB / Taproot surfaces that exist in
this tree are either (a) gated by `IsMWEBEnabled` / versionbits state, which
return `false` on mainnet, or (b) gated by the `BLOCK_HAVE_MWEB` per-block
flag. The script interpreter and the mempool's `IsStandard` both return early
for P2TR-shaped outputs while `SCRIPT_VERIFY_TAPROOT` is unset. Signet is a
documented stub.

What I am **less** confident about: I could not cross-check the BIP34 hash
`57bae90a3342fac0bae15eb2ac9a8924779984bc301ae67730dfda6df49b203c` against
the live chain (no node, no `getblockhash 828325` call). I treat the
hard-coded value as trust-but-verify.

---

## Activation-heights verification (mainnet)

| Feature                     | Documented height | Code height                                | Match? | Severity |
|-----------------------------|-------------------|--------------------------------------------|--------|----------|
| BIP34 (height in coinbase)  | 828,326           | 828,326 (`chainparams.cpp:69,87`)          | yes    | —        |
| BIP65 (CLTV)                | 1,828,326         | 1,828,326 (`chainparams.cpp:70,89`)        | yes    | —        |
| BIP66 (strict DER)          | 1,828,326         | 1,828,326 (`chainparams.cpp:70,90`)        | yes    | —        |
| CSV (BIP68/112/113)         | 903,168           | 903,168 (`chainparams.cpp:71,91`)          | yes    | —        |
| SegWit (BIP141/143/147)     | 903,168           | 903,168 (`chainparams.cpp:68,92`)          | yes    | —        |
| Taproot                     | not active        | `DEPLOYMENT_TAPROOT.nStartTime=NEVER_ACTIVE` (`chainparams.cpp:107-111`) | yes (off) | — |
| MWEB                        | not active        | `DEPLOYMENT_MWEB.nStartTime=NEVER_ACTIVE` (`chainparams.cpp:114-118`)    | yes (off) | — |
| Signet                      | not supported     | `CreateChainParams(SIGNET)` returns a `CTestNetParams` (`chainparams.cpp:479`); `feature_signet.py` raises `SkipTest("Signet not supported")` (`test/functional/feature_signet.py:44`) | yes (off) | — |

Subsidy / retarget / PoW on mainnet:

| Parameter                         | Documented                | Code                                       | Match? |
|-----------------------------------|---------------------------|--------------------------------------------|--------|
| Initial block reward              | 50 DFC                    | `50 * COIN` (`validation.cpp:1273`)        | yes    |
| Halving interval                  | 840,000 blocks            | `840000` (`chainparams.cpp:82`)            | yes    |
| Target spacing                    | 2 min                     | `2 * 60` (`chainparams.cpp:97`)            | yes    |
| Retarget interval                 | 720 blocks                | `86400 / 120 = 720` (`chainparams.cpp:96-97`) | yes |
| PoW limit                         | scrypt mainnet            | `00000fff...fff` (`chainparams.cpp:95`)    | yes    |
| Time-warp / off-by-one retarget fix | present                  | `GetNextWorkRequired` lines 43-45 (`pow.cpp:43-45`) — uses full `DifficultyAdjustmentInterval()` blocks except at first retarget after genesis | yes |

Genesis / magic / address prefixes (mainnet):

| Field                         | Value                                             | Source                       |
|-------------------------------|---------------------------------------------------|------------------------------|
| `MessageStartDefcoinMagic()`  | `0xde 0xfc 0x01 0x4e` (`defc014e`)                | `chainparams.cpp:136-139`    |
| `MessageStartLegacyMagic()`   | `0xfb 0xc0 0xb6 0xdb` (`fbc0b6db`)                | `chainparams.cpp:132-135`    |
| Legacy-only `pchMessageStart` | same as legacy magic (storage compat shim)       | `chainparams.cpp:128-131`; live P2P goes through `MessageStartDefcoinMagic()`/`MessageStartLegacyMagic()` (`chainparams.h:69-71`, `net.cpp:789-798, 3289`) |
| `nDefaultPort`                | 1337                                              | `chainparams.cpp:140`        |
| Address prefix (PUBKEY)       | 0x1E (`D`…)                                       | `chainparams.cpp:161`        |
| Bech32 HRP                    | `dfc`                                             | `chainparams.cpp:168`        |
| MWEB HRP (declared, unused on mainnet) | `dfcmweb`                                 | `chainparams.cpp:169`        |
| `bech32m_hrp`                 | **not configured**                                 | no source matches            |

Testnet:

| Field                | Value           | Source                     |
|----------------------|-----------------|----------------------------|
| `MessageStartDefcoinMagic()` | `0xdf 0xfd 0x02 0x4f` | `chainparams.cpp:257-260` |
| `MessageStartLegacyMagic()`  | `0xfc 0xc1 0xb7 0xdc` | `chainparams.cpp:253-256` |
| `nDefaultPort`       | 31337           | `chainparams.cpp:261`      |
| Address prefix       | 0x6F (`m`/`n`…) | `chainparams.cpp:275`      |
| Bech32 HRP           | `tdfc`          | `chainparams.cpp:282`      |

Regtest:

| Field                | Value           | Source                     |
|----------------------|-----------------|----------------------------|
| `MessageStartDefcoinMagic()` | `0xdd 0xfb 0x00 0x4d` | `chainparams.cpp:357-360` |
| `MessageStartLegacyMagic()`  | `0xfa 0xbf 0xb5 0xda` | `chainparams.cpp:353-356` |
| `nDefaultPort`       | 19444           | `chainparams.cpp:361`      |
| Halving interval     | 150             | `chainparams.cpp:316`      |
| BIP34/65/66/CSV      | 500/1351/1251/432 (`chainparams.cpp:318-322`) | matches regtest upstream    |
| SegwitHeight         | 0 (overridable via `-segwitheight`) | `chainparams.cpp:323,418-426` |
| Taproot deployment   | `ALWAYS_ACTIVE` | `chainparams.cpp:338`      |
| MWEB deployment      | `nStartTime=1601450001` | `chainparams.cpp:343` |

DNS seeds (mainnet) and fixed seeds (mainnet):

| Source                | Contents                                                                          | Comment |
|-----------------------|-----------------------------------------------------------------------------------|---------|
| `vSeeds` (DNS)        | `seed.defcoin.io`, `seed.defcoin.mikej.tech`, `seed.defcoin.dc903.org`, `seed.defcoincore.org`, `seed.defcoin-ng.org` (`chainparams.cpp:155-159`) | five seeds; plausible |
| `chainparams_seed_main` (fixed) | 3 IPv4 entries: 66.42.91.225:1337, 73.52.182.204:1337, 50.116.19.40:1337 (`chainparamsseeds.h:10-25`) | minimal but in the spec |
| `checkpointData`      | entries at heights 1500, 4032, 8064, 16128, 23420, 50000, 80000, 120000, 161500, 179620, 240000, 383640, 409004, 456000, 638902, 721000 (`chainparams.cpp:178-197`) | last is at 721,000; all pre-SegWit-activation (903,168), so no surface mismatch |
| `chainTxData`         | nTime 1551355823, nTxCount 798,434, dTxRate 0.01 (`chainparams.cpp:199-203`)       | last 798,434 transactions as of Mar 2019 — stale; cosmetic |

---

## MWEB / Taproot / Signet surface

| Path / file                                  | What it is                                                                                 | Gated? | Severity |
|----------------------------------------------|--------------------------------------------------------------------------------------------|--------|----------|
| `src/Makefile.am:25-28, 39, 114-154, 210-216, 366-368, 425-426, 494, 576` | `libmw.a` and `src/mweb/*.cpp` compiled unconditionally, no `MWEB_ACTIVE`/`ENABLE_MWEB` `AC_CONFIG_FILES` switch | n/a (always built) | low |
| `src/primitives/block.h:74, 91-95`           | `CBlock::mweb_block` field, only deserialized if last tx is `IsHogEx()` and `SERIALIZE_NO_MWEB` not set | yes (de)serialization gated on tx shape; runtime check at `mweb_node.cpp:34-37` and `validation.cpp:3552` | low |
| `src/primitives/transaction.h:91, 280, 310-323, 396, 514, 553-554` | `mweb_tx`, `m_hogEx`, `HasMWEBTx()`, `IsMWEBOnly()` always present; `IsHogEx()` can become `true` from unserialization of any tx with flag bit 0x08 + null `mweb_tx` | tx-level gate at `validation.cpp:580` (`mweb-before-activation` reject); block-level gate at `mweb_node.cpp:34-37` | low |
| `src/chain.h:131, 185-188, 354-360`         | `BLOCK_HAVE_MWEB` flag and per-index MWEB fields (`mweb_header`, `hogex_hash`, `mweb_amount`) | only serialized when flag set; only set at `validation.cpp:2287-2294` when `!block.mweb_block.IsNull()` (which never happens on mainnet) | low |
| `src/validation.cpp:580-587`                 | mempool rejects MWEB txs / HogEx before activation                                         | yes                                                          | —        |
| `src/validation.cpp:1971-1973`               | `SCRIPT_VERIFY_TAPROOT` only added when `VersionBitsState(... DEPLOYMENT_TAPROOT) == ACTIVE` | yes (always `defined`/`failed` on mainnet → never set) | — |
| `src/validation.cpp:3552` (`MWEB::Node::CheckBlock`) and `3770` (`MWEB::Node::ContextualCheckBlock`) | block-acceptance MWEB context-free / contextual checks                                    | yes (rejects any block with MWEB data on non-MWEB chain — `mweb_node.cpp:34-37, 41-44`) | — |
| `src/miner.cpp:150-152`                      | `fIncludeWitness` / `fIncludeMWEB` derived from `IsWitnessEnabled` / `IsMWEBEnabled`       | yes — never set on mainnet for MWEB                              | —        |
| `src/init.cpp:1883-1895`                     | `NODE_WITNESS` set whenever `SegwitHeight != max()`. `NODE_MWEB` / `NODE_MWEB_LIGHT_CLIENT` set only if MWEB is configured (i.e. `nStartTime != NEVER_ACTIVE` or `nStartHeight != max()`) | yes — mainnet never advertises MWEB service bits             | —        |
| `src/script/interpreter.cpp:1885-1887`       | `if (!(flags & SCRIPT_VERIFY_TAPROOT)) return set_success(serror);` for P2TR-shaped programs | only triggers if a P2TR-shaped output exists; SCRIPT_VERIFY_TAPROOT never set on mainnet, so any P2TR-shaped output's spend is accepted with no signature check | **medium** |
| `src/script/interpreter.cpp:1438-1450`       | `PrecomputedTransactionData::Init` treats any 34-byte OP_1 scriptPubKey as taproot for sighash precomputation | only consumed when `m_spent_outputs_ready`; affects sighash path selection, not script success | low |
| `src/script/interpreter.cpp:1108-1111`       | `OP_CHECKSIGADD` opcode                                                                    | parsed, but only legal under `SigVersion::TAPSCRIPT`, which only fires from a P2TR-shaped output | low (dead code on mainnet today) |
| `src/chainparams.cpp:107-111, 114-118, 237-244, 337-344` | `vDeployments[DEPLOYMENT_TAPROOT/MWEB]` always present; mainnet sets `NEVER_ACTIVE`; testnet sets explicit start heights (2225664/2209536); regtest sets `ALWAYS_ACTIVE` (taproot) and `nStartTime=1601450001` (mweb) | yes (gated off on mainnet) | low (testnet heights are a known artifact of inheriting Litecoin parameters; they would only matter if anyone runs Defcoin testnet) |
| `src/versionbitsinfo.cpp:9-22`               | `VersionBitsDeploymentInfo[]` lists `taproot` / `mweb` so `-vbparams=mweb:...` works       | yes — mainnet deployment is `NEVER_ACTIVE` so `getblockchaininfo` will hide them (see `blockchain.cpp:2077`) | low |
| `src/chainparams.cpp:61-66, 348`             | `GetFrozenMWEBOutputIDs()` returns one entry (`2f3a08d9...f7`); used only in regtest (`CRegTestParams::consensus.frozen_mweb_output_ids = GetFrozenMWEBOutputIDs();` `chainparams.cpp:348`). Mainnet never assigns it. | yes (regtest-only) | — |
| `src/chainparams.cpp:478-479`                | `CreateChainParams(SIGNET)` returns a `CTestNetParams()` with a `// TODO: Support SigNet` comment. `consensus.signet_blocks = false`. | yes (signet is a stub) | — |
| `src/chainparamsbase.cpp:50-51`              | `CreateBaseChainParams(SIGNET)` returns port 39332/39335, dir `signet`                     | plumbing exists; no chain params                              | low (cosmetic) |
| `src/validation.cpp:1196-1198, 3496-3499`    | Signet block solution check is gated on `consensus.signet_blocks`                          | yes (always false)                                            | —        |
| `src/chainparams.cpp:169, 283, 400`          | `mweb_hrp` is a member of `CChainParams` and is set for mainnet/testnet/regtest even though MWEB is off | not surfaced in RPC; `Bech32HRP()` is the only address-related accessor that is wired to address decoding | low |
| `src/chainparams.h:104-106`                  | `pchMessageStart`, `pchMessageStartDefcoinMagic`, `pchMessageStartLegacyMagic` all present as members; live code uses the `MessageStart*Magic()` accessors, not `MessageStart()` | yes (per the chainparams.h:67-71 comment) | low |

---

## RPC surface (non-trivial entries)

For each entry: where the field / call is and whether it's a stub, gated, or
unconditional.

| RPC entry                                                                 | Source                                                                                | Status on mainnet                                                                                                                  | Severity |
|---------------------------------------------------------------------------|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|----------|
| `getblockchaininfo().softforks.taproot` / `.mweb`                          | `blockchain.cpp:2222-2224` (`VBSoftForkDescPushBack`)                                 | hidden — `blockchain.cpp:2077` returns early for `nStartTime == NEVER_ACTIVE`. Mainnet `getblockchaininfo` will not list them.       | —        |
| `getblockchaininfo().softforks.bip34/bip66/bip65/csv/segwit`              | `blockchain.cpp:2217-2221` (`BuriedForkDescPushBack`)                                 | reported as "active" after the documented heights                                                                                   | —        |
| `getblockheader(hash, true).mweb_header` / `.mweb_amount` / `.prev_mweb_amount` | `blockchain.cpp:157-174` (`blockheaderToJSON`)                                       | all gated on `blockindex->mweb_header != nullptr` (`chain.h:186`); null on mainnet → fields never appear                          | —        |
| `getblock(hash, 1).mweb` / `getblock(hash, 1).tx[i].mweb`                  | `blockchain.cpp:221-301` (verbose `getblock`)                                         | gated on `!block.mweb_block.IsNull()`; on mainnet `mweb_block` is always null                                                       | —        |
| `getblock(hash, 1).mwebweight` on mempool entries                          | `blockchain.cpp:921-934`                                                              | only present when `tx.HasMWEBTx()`; mainnet rejects MWEB txs at mempool entry (`validation.cpp:580`)                                  | —        |
| `getrawtransaction(..., true).mweb`                                        | `blockchain.cpp:946-1003`                                                             | same gate — only when the tx is an MWEB tx                                                                                         | —        |
| `getblocktemplate.rules` advertises `mweb` / `segwit` client-rule strings   | `mining.cpp:535-540`                                                                  | presence is non-conditional; the *enforcement* of those rules is gated (`mining.cpp:731-740`): `mweb_required` is `false` on mainnet so `mweb` is never required | low (cosmetic — list contains a string an operator may misread) |
| `getblocktemplate` returns `mweb` field for the extension block            | `mining.cpp:914-917`                                                                  | only included when `mweb_block` is non-null — never on mainnet                                                                      | —        |
| `getmininginfo().currentblockmwebweight`                                   | `mining.cpp:436`                                                                      | only pushed when `BlockAssembler::m_last_block_mweb_weight` is set; mainnet never produces an MWEB block, so this is absent        | —        |
| `getnetworkinfo`                                                          | `net.cpp:585-675`                                                                     | no MWEB / Taproot / Signet fields                                                                                                  | —        |
| `getpeerinfo` / `listbanned` / etc.                                        | `rpc/net.cpp`                                                                         | no MWEB / Taproot / Signet fields                                                                                                  | —        |
| ZMQ `pubrawmweb` / `pubmwebheader` / `pubmwebleafset` topics               | not present                                                                            | only `pubhashblock`, `pubhashtx`, `pubrawblock`, `pubrawtx` are registered (`zmq/zmqnotificationinterface.cpp:35-38`)            | —        |
| `getsignetchallenge` / `getsignetinfo` (BIP325 / signet RPC)               | not implemented (`rpc/` source files have no matches)                                  | signet is a documented stub                                                                                                        | —        |
| `getmwebinfo` / `getmwebproofs` (MWEB-specific RPC)                        | not implemented (`rpc/` source files have no matches)                                  | mweb_block data is only surfaced through `getblock` verbose / `getblockheader` if the block has MWEB data — no separate RPC | —        |
| `getacceptlegacymagic` / `setacceptlegacymagic`                            | `rpc/net.cpp:892-925, 1226-1227`                                                      | implemented; relevant on mainnet during the magic-migration window                                                                  | —        |
| `getonlydefcoinuseragents` / `setonlydefcoinuseragents`                    | `rpc/net.cpp:865-888, 1224-1225`                                                      | implemented; controls the user-agent filter (`net_processing.cpp:204-211, 3080-3082, 3350-3355`)                                  | —        |

Net message types that reference MWEB:

- `NetMsgType::MWEBHEADER`, `MWEBLEAFSET`, `GETMWEBUTXOS`, `MWEBUTXOS` are
  declared unconditionally in `protocol.cpp:49-52` and added to
  `allNetMessageTypes[]` (`protocol.cpp:93-96`). On a non-MWEB chain these
  messages are simply never sent and a peer that tries to send them is
  handled by the existing `notfound` / unknown-command path; not a
  consensus risk, but the protocol surface advertises MWEB feature
  negotiation to anyone running `getpeerinfo`/version messages.

---

## Critical / High issues

Sorted by severity. None of these fork the chain; the consensus rules match
the README. The two "high" items are operator/performance issues that
should be picked up before the next release.

1. **`defaultAssumeValid = uint256{}` on mainnet (`chainparams.cpp:121`).**
   Severity: **high** (operator impact, not consensus drift).
   The null hash disables the assume-valid optimization. On first IBD the
   node will verify every script from genesis, even for blocks whose
   history has already been externally validated. For an 800k+ scrypt chain
   this multiplies the initial-sync cost noticeably. It is also explicitly
   called out by `init.cpp:1116-1117` (`LogPrintf("Validating signatures for all blocks.")`).
   Fix sketch: set
   `consensus.defaultAssumeValid = uint256S("<recent-mainnet-block-hash>");`
   in `CMainParams`. Pair with a matching
   `consensus.nMinimumChainWork` that approximates real chain work (see #2).

2. **`nMinimumChainWork = 0x...0001000000000000` on mainnet (`chainparams.cpp:120`).**
   Severity: **high** (operator impact / not consensus drift).
   The value is approximately `2^56` — about two orders of magnitude
   smaller than the realistic cumulative scrypt chain work at the current
   tip. It is non-zero only for the purpose of letting the
   `pindex->nChainWork < nMinimumChainWork` early-return not always
   trivially fire; the value itself doesn't enforce anything meaningful.
   Combined with #1, the node's IBD is purely validation-driven. Fix
   sketch: update to a real chain-work value (compute from the current
   tip, e.g. `getblockchaininfo.chainwork`).

3. **P2TR-shaped outputs are accepted without signature verification on
   mainnet (`script/interpreter.cpp:1885-1887`).**
   Severity: **medium** (dormant consensus surface).
   `if (!(flags & SCRIPT_VERIFY_TAPROOT)) return set_success(serror);`
   means a 34-byte `OP_1 0x20 <32 bytes>` output can be spent with any
   witness when `SCRIPT_VERIFY_TAPROOT` is not set in the script flags.
   `SCRIPT_VERIFY_TAPROOT` is set only when versionbits TAPROOT reaches
   `ThresholdState::ACTIVE` (`validation.cpp:1971-1973`), which on mainnet
   is never. There is currently no `bech32m_hrp` configured in chainparams
   so the wallet cannot create a P2TR address, and `policy.cpp:198-200`
   blocks P2TR spends at mempool entry, so an external party can only put
   one on-chain via a miner. The risk is forward-compatibility: if a
   future soft-fork activates Taproot on Defcoin, any P2TR outputs created
   in the interim would suddenly require valid signatures. Fix sketch:
   replace the unconditional success with an `SCRIPT_ERR_DISCOURAGE_UPGRADABLE_WITNESS_PROGRAM`
   rejection whenever a v1/32-byte program is observed but
   `SCRIPT_VERIFY_TAPROOT` is not set, so the same surface becomes a clean
   non-Taproot future-softfork slot.

4. **`pchMessageStart` field for mainnet is the legacy Litecoin magic
   (`chainparams.cpp:128-131`).**
   Severity: **low** (cosmetic / historical compat).
   `pchMessageStart[0..3] = 0xfb 0xc0 0xb6 0xdb` is identical to
   `MessageStartLegacyMagic()`. The live P2P path uses the explicit
   `MessageStartDefcoinMagic()` / `MessageStartLegacyMagic()` accessors
   (`net.cpp:789-798, 3289`), and the comment at `chainparams.h:67-68`
   documents that `MessageStart()` is preserved for "inherited
   Litecoin-derived storage paths". This is correct behavior, but anyone
   reading `chainparams.cpp` looking for the active mainnet magic will see
   the legacy value first. Fix sketch: add a comment at
   `chainparams.cpp:128-131` flagging that the field is intentionally the
   legacy magic and pointing readers to `pchMessageStartDefcoinMagic` for
   the active magic.

5. **MWEB `libmw` is built unconditionally; there is no `MWEB_ACTIVE`
   build flag (`Makefile.am:25-28, 39, 114-154, 366-368, 425-426, 494`).**
   Severity: **low** (build surface; no runtime impact because all
   activation paths are gated off on mainnet).
   The MWEB validation, miner, wallet, RPC and P2P message types are
   always compiled in, including `libmw` and the `mweb_node` checks.
   Runtime, they are dead code on mainnet (gated by `IsMWEBEnabled` /
   `IsMWEBActive` / versionbits state). If the project ever wants to
   upstream a build that supports a non-MWEB binary, it would need an
   `MWEB_ACTIVE` `configure` switch and corresponding `#ifdef`s. Fix
   sketch: not required for correctness; consider documenting in
   `contrib/devtools/` or `INSTALL.md` that the MWEB code is always
   compiled but inert on mainnet.

6. **`GetBlockSubsidy` comment is stale (`validation.cpp:1274`).**
   Severity: **low** (cosmetic).
   Comment says "Subsidy is cut in half every 210,000 blocks which will
   occur approximately every 4 years." — that's the Bitcoin default. The
   code uses `consensusParams.nSubsidyHalvingInterval`, which on Defcoin
   mainnet is 840,000 (`chainparams.cpp:82`). Fix sketch: change the
   comment to "Subsidy is cut in half every `nSubsidyHalvingInterval`
   blocks, network-specific (Defcoin mainnet: 840,000)."

7. **`m_assumed_blockchain_size = 2` GB for mainnet (`chainparams.cpp:142`).**
   Severity: **low** (operator cosmetic).
   This value is used only for the data-dir free-space warning, not for
   IBD. For a low-activity scrypt chain with 800k+ blocks, 2 GB is
   plausible but tight; the actual on-disk size depends on tx density and
   segwit adoption. Worth bumping if any operators report a misleading
   "low disk space" warning. Fix sketch: re-measure against the current
   chain and update the value.

8. **`feature_signet.py` ships with `SkipTest("Signet not supported")` at
   the top of `run_test` (`test/functional/feature_signet.py:43-44`).**
   Severity: **low** (cosmetic / test hygiene).
   The test would otherwise exercise the signet block-solution path,
   which on this tree routes through `CTestNetParams` with
   `consensus.signet_blocks = false` and would fail. The explicit skip is
   correct, but the surrounding ~50 lines of expected behavior is dead
   code. Fix sketch: replace the body of `run_test` with just the
   `raise SkipTest(...)` and a comment, or move the test to
   `test/functional/wallet/` style disabled list.

---

## Things I checked and concluded are *not* issues

- **Mainnet magic bytes.** The README documents `defc014e` (preferred) and
  `fbc0b6db` (legacy). The code defines both via
  `MessageStartDefcoinMagic()` (`chainparams.cpp:136-139`) and
  `MessageStartLegacyMagic()` (`chainparams.cpp:132-135`). The
  `-acceptlegacymagic` flag (default on) is enforced in
  `V1TransportDeserializer::TrySelectMessageStart` (`net.cpp:783-803`).
  Outbound magic selection in `UseLegacyOutboundMagic`
  (`net.cpp:107-137`) gates on `IsDefcoinMainnet() && GetAcceptLegacyMagic()`,
  and `CNode` ctor uses it via
  `V1TransportSerializer(use_legacy_outbound_magic ? ... Legacy ... : ... Defcoin ...)`
  (`net.cpp:3289`). This is correctly wired.
- **User-agent filter.** The `/Defcoin`-prefix requirement is enforced
  via `IsDefcoinPrefixedUserAgent`
  (`net_processing.cpp:226-233, 3080-3084, 3350-3355`), with
  `IsLocalP2PoolUserAgent` as a LAN exemption. `-onlydefcoinua` default
  (`net.h:46`-ish, via `DEFAULT_DEFCOIN_USER_AGENT_FILTER`) is
  initialized from args in `init.cpp:1320`. Operationally correct.
- **MWEB blocks on mainnet.** `mweb_node.cpp:31-37` rejects any block
  whose `mweb_block` is non-null when MWEB is disabled, and
  `mweb_node.cpp:41-44` rejects any block containing a HogEx tx when MWEB
  is disabled. `validation.cpp:3552` invokes `MWEB::Node::CheckBlock` on
  the context-free path, and `validation.cpp:3770` invokes
  `MWEB::Node::ContextualCheckBlock` on the contextual path. Both
  ultimately rely on `IsMWEBEnabled` returning `false`, which it does on
  mainnet because `DEPLOYMENT_MWEB` is `NEVER_ACTIVE`.
- **SegWit on mainnet.** The script flags wired in
  `validation.cpp:1940-1982` correctly turn on `SCRIPT_VERIFY_WITNESS`,
  `SCRIPT_VERIFY_P2SH`, `SCRIPT_VERIFY_DERSIG`, `SCRIPT_VERIFY_CHECKLOCKTIMEVERIFY`,
  `SCRIPT_VERIFY_CHECKSEQUENCEVERIFY`, `SCRIPT_VERIFY_NULLDUMMY` at the
  documented heights, and leave `SCRIPT_VERIFY_TAPROOT` off.
- **Time-warp / off-by-one.** The Litecoin off-by-one retarget fix is
  present at `pow.cpp:43-45` (`blockstogoback = DifficultyAdjustmentInterval()`,
  except at the first retarget after genesis). The `nActualTimespan`
  clamp (1/4 to 4× target) is also present at `pow.cpp:64-67`.
- **Subsidy.** `GetBlockSubsidy` (`validation.cpp:1266-1277`) starts at
  `50 * COIN` and halves every `nSubsidyHalvingInterval`, returning 0
  after 64 halvings. The halving interval of 840,000 (`chainparams.cpp:82`)
  matches the README.
- **Signet.** Stubbed at `chainparams.cpp:478-479`; the
  `CheckSignetBlockSolution` call at `validation.cpp:1197, 3497` is
  gated on `consensus.signet_blocks`, which is `false` everywhere on
  this tree. `-signet` builds still produce a testnet-shaped chain.
  This matches the README.
- **Checkpoint data.** `checkpointData` for mainnet
  (`chainparams.cpp:178-197`) ends at height 721,000, well below
  SegWit activation (903,168), so no checkpoint straddles a softfork
  boundary. `fCheckpointsEnabled` is the default, no `-nocheckpoints`
  surprise.

---

## Audit trail

- Verified all consensus heights in `src/chainparams.cpp:68-205` against
  the README.
- Verified MWEB block-acceptance paths in `src/validation.cpp:3552, 3770`
  and `src/mweb/mweb_node.cpp:31-230`.
- Verified taproot script flag wiring in
  `src/validation.cpp:1971-1973` and
  `src/script/interpreter.cpp:1885-1933`.
- Verified P2P magic negotiation in
  `src/net.cpp:90-110, 783-803, 3288-3292`.
- Verified user-agent filter in `src/net_processing.cpp:226-245, 3080-3084, 3350-3355`.
- Verified retarget and subsidy in `src/pow.cpp:13-87` and
  `src/validation.cpp:1266-1277`.
- Cross-checked RPC surface in `src/rpc/blockchain.cpp:135-302, 2073-2225`,
  `src/rpc/mining.cpp:407-440, 525-595, 728-740, 905-920`,
  `src/rpc/net.cpp:585-675, 860-925, 1216-1229`, and ZMQ in
  `src/zmq/zmqnotificationinterface.cpp:33-38`.
- Confirmed MWEB build presence in `src/Makefile.am:25-154, 210-216, 366-368, 425-426, 494, 576`.
- Confirmed MWEB / Taproot / Signet do not appear in `getnetworkinfo` and
  do not appear unconditionally in `getmininginfo`,
  `getblockchaininfo.softforks`, `getblock`, or `getblockheader` on
  mainnet.
