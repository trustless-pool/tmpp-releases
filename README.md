# TMPP — Trustless Mining Pool Protocol

A decentralized mining pool where every payout is cryptographically verifiable on-chain. No registration. No trust. Your address is your account.

**420 tests. 7 security audit rounds. 8 formal verification proofs. 0 critical vulnerabilities.**

---

**RC1 — Kaspa Launch**

Kaspa was chosen for TMPP's debut because its community values decentralization; the same principle TMPP is built on. Kaspa's 1-second blocks make self-hosted pooling transformative: miners recover 3–10% of revenue lost to stale shares on centralized pools. Bitcoin, Litecoin, Dogecoin, Bitcoin Cash, and Alephium support is in development.

## Why TMPP?

Every pool today requires trust. You send hashrate and hope the operator pays you fairly. You can't verify share counts. You can't audit reward distribution. You can't prove your payout was correct.

TMPP fixes this:

- **Verifiable payouts** — every distribution is Merkle-committed in the coinbase. Click "Verify" in the dashboard to see the cryptographic proof.
- **0.5% fee** — most Kaspa pools charge 0.9–1%. F2Pool charges 2–3%. You keep more.
- **Self-host for zero latency** — run the pool node on your mining rig or in your facility. <1ms share latency vs 50–150ms on centralized pools. On Kaspa's 1-second blocks, this eliminates stale shares entirely.
- **No registration** — your payout address IS your account. Point your miner and earn.
- **Formally verified** — 8 Kani proofs mathematically guarantee fee conservation, credit conservation, and overflow safety. No other pool on any chain has this.

## Quick Start

### One Command Setup

```bash
tmpp-node
```

First run auto-detects that no config exists, walks you through setup (installs kaspad, syncs the chain), and starts mining. Every run after that, it just starts.

### Or Connect to a Hosted Node (Coming Soon)

Hosted TMPP nodes will be available after RC1 testing completes. For now, self-host for the best experience.

### Self-Host (Recommended for Kaspa)

Kaspa has 1-second blocks. At 100ms of pool latency, ~10% of your work is stale. Self-hosting drops stale rate below 0.1%.

```bash
# Download TMPP
curl -L https://github.com/trustless-pool/TMPP/releases/latest/download/tmpp-node-linux-x86_64 -o tmpp-node
chmod +x tmpp-node

# Run — auto-installs kaspad on first run
./tmpp-node
```

TMPP uses <5% CPU and <500MB RAM. Run it on the same machine as your miner or on a $150 mini PC in your rack.

## Verify Your Payouts

This is what makes TMPP different from every other pool. Every block reward distribution is hashed into a Merkle tree. The root of that tree is committed in the coinbase OP_RETURN — on-chain, permanent, verifiable by anyone.

**How to verify:**

1. Open the dashboard → My Miner
2. Click "Verify on Kaspa" next to any block
3. The dashboard fetches the distribution, rebuilds the Merkle tree, and checks the root against the on-chain commitment
4. If it matches, your payout was computed correctly. If it doesn't, the proof of fraud is in the blockchain forever.

No other pool lets you do this. DPool tells you what you earned. TMPP lets you prove it.

## How It Works

```
Your Miner ──Stratum──→ TMPP Node ──wRPC──→ kaspad
                            │
                       PPLNS Window
                       Share WAL (zero data loss)
                       Merkle-committed distributions
                            │
                       gossip mesh (other TMPP nodes)
```

1. Your miner connects via Kaspa Stratum (compatible with BzMiner, lolMiner, IceRiver, Bitmain KS-series)
2. Shares are validated with full kHeavyHash PoW verification — every share includes the block header preimage for independent proof-of-work verification
3. The PPLNS window tracks your contribution using integer-only arithmetic (no floating point, no rounding errors)
4. When a block is found, rewards are distributed proportionally
5. The distribution is Merkle-committed in the coinbase OP_RETURN — verifiable by anyone, forever

## Dashboard

Open `http://localhost:8080` after starting TMPP.

- Real-time hashrate, shares, workers
- Per-miner earnings and payout history
- **Verify on Kaspa** — cryptographic proof your payout is correct
- Network topology visualization
- Profitability calculator with fee comparison

## Architecture

| Component | Description |
|---|---|
| Kaspa Stratum Server | Ethereum-style JSON-RPC, port 3337 |
| kHeavyHash Validation | Full 4-step PoW verification (blake2b → cSHAKE256 → matrix multiply → cSHAKE256) |
| PPLNS Window | 10,000-share sliding window, integer-only arithmetic |
| Share WAL | Write-ahead log — zero data loss on crash or restart |
| Pool Ledger | Per-miner balances with sled persistence |
| Gossip Mesh | Signed P2P distribution propagation, multi-hop, self-organizing |
| Dashboard | Single-file HTML, chain-aware, live verification |

## Self-Hosting Economics

For a mining operation earning $10,000/day on Kaspa:

| | Centralized Pool | TMPP Self-Hosted |
|---|---|---|
| Stale rate (1s blocks) | 3–10% | <0.1% |
| Pool fee | 0.9–2% | 0.5% |
| Annual revenue lost | $51K–$146K | $18K |
| **Annual savings** | | **$33K–$128K** |

The TMPP node runs on a $5/month VPS or on your mining rig directly. A $150 Intel N100 mini PC in your server rack is the ideal dedicated setup.

## Security

- 420 automated tests, 0 failures
- 7 independent security audit rounds, 0 critical vulnerabilities
- 8 Kani formal verification proofs (fee conservation, credit conservation, overflow safety)
- All gossip messages signed with secp256k1
- Node signing key persisted to disk (survives restarts)
- Share WAL guarantees zero data loss on crash
- Dashboard bound to localhost by default
- Connection limits and handshake timeouts on Stratum
- Release binary signature verification (planned for GA)

## Supported Chains

| Chain | Status | Algorithm | Block Time |
|---|---|---|---|
| **Kaspa (KAS)** | **LIVE** | kHeavyHash | 1 second |
| Bitcoin (BTC) | In development | SHA256d | 10 minutes |
| Litecoin (LTC) | In development | Scrypt | 2.5 minutes |
| Dogecoin (DOGE) | In development | Scrypt | 1 minute |
| Bitcoin Cash (BCH) | In development | SHA256d | 10 minutes |
| Alephium (ALPH) | In development | Blake3 | 64 seconds |

## TMPP vs. Current Kaspa Pools

| | DPool | HeroMiners | WoolyPooly | TMPP |
|---|---|---|---|---|
| Fee | 1% | 0.9% | 0.9% | **0.5%** |
| Trustless payouts | No | No | No | **Yes — Merkle proofs** |
| Self-hosted option | No | No | No | **Yes** |
| Stale rate (Kaspa) | 3–10% | 3–10% | 3–10% | **<0.1% (local)** |
| Payout verification | Trust operator | Trust operator | Trust operator | **On-chain proof** |
| Single point of failure | Yes | Yes | Yes | **No — mesh network** |
| Formal verification | No | No | No | **8 Kani proofs** |

## Documentation

- [Getting Started](GETTING-STARTED.md) — connect in 2 minutes
- [Self-Hosting Guide](SELF-HOSTING.md) — why local nodes win on Kaspa
- [Hardware Guide](HARDWARE-GUIDE.md) — what to buy, how to set up
- [Kaspa Test Guide](KASPA-TEST-GUIDE.md) — two-machine test with GPUs
- [Institutional Guide](INSTITUTIONAL.md) — for large mining operations
- [Protocol Specification](PROTOCOL.md) — how TMPP works under the hood

## FAQ

**How do I know TMPP is paying me fairly?**
Every block reward distribution is Merkle-committed in the coinbase. Go to the dashboard → My Miner → Verify. You can independently verify your share of every block. The proof is on-chain and permanent.

**What if my node goes down?**
The share WAL replays on restart. Zero shares lost, zero credits lost. Your miners reconnect automatically.

**Is there a minimum hashrate?**
No. Any hashrate works. ASIC or GPU.

**How is this different from Ocean?**
Similar philosophy — trustless, transparent. Key differences: TMPP supports multiple chains (Ocean is Bitcoin-only), has lower fees (0.5% vs 2%), miners can self-host nodes for zero-latency mining, and payouts are Merkle-committed for cryptographic verification rather than relying on operator transparency.

**How is this different from P2Pool?**
P2Pool required every miner to validate every share (couldn't scale past ~300 miners) and produced many small on-chain payouts (expensive in fees). TMPP uses a gossip mesh and Merkle commitments instead of a sharechain — scaling to 1M+ miners at 0.06% block overhead. Miners can connect to hosted nodes or self-host, while still getting trustless verification.

**Why Kaspa first?**
Kaspa's 1-second blocks make the self-hosting advantage dramatically larger than any other chain. On Bitcoin, 100ms of latency is 0.017% of the block interval. On Kaspa, it's 10%. No other chain benefits this much from a self-hosted pool. The Kaspa community also values decentralization as a core principle — TMPP aligns with that ethos.

**Who runs this?**
TMPP is developed by an independent engineer. The protocol specification is published for transparency. The trustless design means you don't need to trust the developer — you verify your payouts cryptographically.

## Community

- GitHub: [github.com/trustless-pool/TMPP](https://github.com/trustless-pool/TMPP)
- Protocol Spec: [PROTOCOL.md](PROTOCOL.md)