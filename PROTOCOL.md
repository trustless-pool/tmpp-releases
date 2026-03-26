# TMPP Protocol Specification

## Overview

TMPP (Trustless Mining Pool Protocol) is a decentralized mining pool protocol where every reward distribution is cryptographically committed on-chain via Merkle trees. Miners verify their payouts independently — no trust in the pool operator is required.

## Architecture

```
Miners ──Stratum──→ TMPP Node ──RPC──→ Chain Daemon (kaspad, bitcoind, etc.)
                        │
                   PPLNS Window (share tracking)
                   Share WAL (crash recovery)
                   Pool Ledger (balances)
                   Merkle Commitment (distribution proofs)
                        │
                   Gossip Mesh ←──→ Other TMPP Nodes
```

## Share Submission

Miners connect via the chain's native Stratum protocol:
- **Kaspa**: Ethereum-style JSON-RPC over TCP (port 3337)
- **Bitcoin/LTC/DOGE/BCH**: Stratum V2 binary framing (port 3333)

Each submitted share includes:
- Nonce that produces a hash meeting the share difficulty target
- Block header reference (job ID)
- Worker identity (payout address + rig name)

The pool validates every share with full PoW verification:
- **Kaspa**: 4-step kHeavyHash (blake2b → cSHAKE256 → matrix multiply → cSHAKE256)
- **Bitcoin**: double-SHA256
- **Litecoin/Dogecoin**: Scrypt(1024, 1, 1)

## PPLNS Window

Pay Per Last N Shares (N = 10,000 by default).

- Sliding window of the most recent N valid shares
- Integer-only arithmetic — no floating point, no rounding errors
- Deterministic: same shares → same distribution on every node
- Share WAL (write-ahead log) guarantees zero data loss on crash

## Reward Distribution

When a block is found:

1. **Snapshot** the PPLNS window: `{(miner_address, share_count)}` for each miner
2. **Apply protocol fee** (0.5%): `fee = block_reward × 50 / 10000`, `net = block_reward - fee`
3. **Distribute proportionally**: `miner_reward = net × miner_shares / total_shares`
4. **Remainder** (from integer division) goes to the highest-contributing miner
5. **Merkle commit**: compute `SHA3-256` Merkle root of sorted `(address, reward)` pairs
6. **Embed in coinbase**: root stored in `OP_RETURN` output (32 bytes)

### Formal Guarantees

Verified by 8 Kani formal proofs:
- **Fee conservation**: `fee + net_reward == block_reward` for all inputs
- **Credit conservation**: `sum(all_credits) == net_reward` for all distributions
- **Overflow safety**: no integer overflow in any arithmetic path

## Merkle Proof Verification

Any miner can verify their payout:

1. Fetch the distribution for a given block height
2. Sort entries by address (lowercase, lexicographic)
3. Compute leaf hash: `SHA3-256(0x00 || address_bytes || reward_u256_be)`
4. Walk the Merkle proof path with internal hashes: `SHA3-256(0x01 || left || right)`
5. Compare computed root against the on-chain commitment

Domain separation prefixes (`0x00` for leaves, `0x01` for internal nodes) prevent second-preimage attacks.

## Gossip Protocol

TMPP nodes form a P2P mesh to propagate distributions and share data.

### Message Types

| Message | Description | Signed? |
|---|---|---|
| `BroadcastDistribution` | Full distribution after block found | Yes (secp256k1) |
| `DeltaDistribution` | Incremental update (changes only) | Yes |
| `BlockFound` | Block announcement with hash | Yes |
| `ShareBatch` | Batch of validated shares | No (PoW-verified) |
| `PeerList` | Known peer addresses | No |
| `Ping` / `Pong` | Keepalive | No |

### Security

- All consensus messages (distributions, block announcements) are signed with secp256k1
- Node identity: `node_id = SHA3-256(public_key)`
- Signature verification before acceptance
- Merkle root verification before crediting
- Peer scoring with automatic ban for bad actors (threshold: -100)
- Rate limiting: 500 token bucket, 100/sec refill

### Wire Format

- 2-byte magic prefix
- 4-byte big-endian payload length
- zstd-compressed payload (max 2 MiB compressed, 8 MiB decompressed)
- Field validation: max 200K distribution entries, 4096 shares per batch

## Protocol Fee

- Fixed at 0.5% (50 basis points)
- Hardcoded in the binary — not operator-configurable
- Applied before distribution: `fee = block_reward × 50 / 10000`
- Fee recipient address is per-chain, embedded at compile time
- Verifiable: any node can check `sum(miner_payouts) + fee == block_reward`

## Share WAL (Write-Ahead Log)

Every share is persisted to disk before entering the PPLNS window:

1. Share arrives → write to WAL (sled tree, sequential key)
2. Insert into PPLNS window (in-memory)
3. On crash → `recover_from_wal()` replays all WAL entries on startup

Guarantees: zero share loss, zero credit loss, even on power failure.

## Supported Chains

| Chain | Algorithm | Stratum | Block Time | Status |
|---|---|---|---|---|
| Kaspa | kHeavyHash | JSON-RPC/TCP | 1s | Live |
| Bitcoin | SHA256d | Stratum V2 | 600s | Development |
| Litecoin | Scrypt | Stratum V2 | 150s | Development |
| Dogecoin | Scrypt | Stratum V2 | 60s | Development |
| Bitcoin Cash | SHA256d | Stratum V2 | 600s | Development |
| Alephium | Blake3 | Binary TCP | 64s | Development |

## Version

Protocol version: 1.0 (RC1)
Reference implementation: https://github.com/trustless-pool/TMPP
