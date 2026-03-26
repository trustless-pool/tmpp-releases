# Self-Hosting TMPP — Why Local Nodes Win

## The Problem With Centralized Pools

Every centralized pool has the same architecture: your mining hardware connects over the internet to a remote server. That server is in a data center — often on a different continent.

```
Your ASIC (Texas) ──── 50ms ────→ Pool Server (Hong Kong)
                                         │
                                    150ms round trip
                                         │
                              Every share, every template
```

This introduces latency at every step:
- **New block template:** When a block is found on the network, the pool server gets the updated template and sends it to your ASIC. Your hardware was mining a stale block for 50-150ms. That's wasted electricity.
- **Share submission:** Every share you submit travels across the internet. Network jitter, packet loss, and TCP retransmits all add up.
- **Stale shares:** On fast-block chains like Kaspa (1-second blocks), even 50ms of latency means 5% of your work is stale. That's 5% of your revenue gone.

You're also trusting the pool with:
- Your exact hashrate (competitive intelligence)
- Your IP address and location
- Accurate share counting and reward distribution
- Uptime — if their server goes down, your miners are idle

## The TMPP Self-Hosted Advantage

TMPP is designed to run in your facility, on the same network as your mining hardware.

```
Your ASIC (Texas) ── <1ms ──→ Your TMPP Node (Texas, same rack)
                                         │
                                    gossip mesh
                                         │
                              Shares counted INSTANTLY
                              Templates received INSTANTLY
```

### What Changes

| Metric | Centralized Pool | TMPP Self-Hosted |
|---|---|---|
| Share latency | 50-150ms | <1ms |
| Template latency | 50-150ms | <1ms |
| Stale rate (10-min blocks) | 0.5-2% | ~0% |
| Stale rate (1-sec blocks) | 3-10% | <0.1% |
| Privacy | Pool sees everything | Nothing leaves your facility |
| Dependency | Pool down = idle | Your node, your uptime |
| Verification | Trust the pool | Verify every payout on-chain |

### The Math

For a mining operation earning $10,000/day on a centralized pool with a 2% stale rate:

- Stale loss: **$200/day = $73,000/year**
- Pool fee (2%): **$200/day = $73,000/year**
- Total cost: **$146,000/year**

Same operation on TMPP self-hosted (1% fee, ~0% stale rate):

- Stale loss: **~$0/day**
- Pool fee (1%): **$100/day = $36,500/year**
- Total cost: **$36,500/year**

**Savings: $109,500/year.** The TMPP node runs on a $50/month server.

### For Fast-Block Chains (Kaspa, Alephium)

Kaspa has 1-second blocks. At 100ms of pool latency, roughly 10% of your work lands on a stale block. That's 10% of your revenue evaporating.

With a self-hosted TMPP node on the same LAN as your GPUs, stale rate drops below 0.1%. On Kaspa, self-hosting isn't just better — it's essential for competitive mining.

## Same Machine as Your GPUs

GPU miners don't need a separate computer. Run the TMPP node on the same machine as your mining software. It uses <5% CPU and <500MB RAM — your GPUs won't notice. Point your miner at `127.0.0.1:3337` and share latency drops to literally zero.

See the [Hardware Guide](HARDWARE-GUIDE.md#running-on-your-mining-rig-recommended-for-gpu-miners) for setup instructions.

## How It Works

### You're Still Part of the Pool

Self-hosting doesn't mean solo mining. Your TMPP node joins the global gossip mesh:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Your Facility   │     │  TMPP Hosted     │     │  Other Miner's  │
│                  │     │  Infrastructure  │     │  Facility        │
│  ASICs → TMPP   │◄───►│  Node (US-East)  │◄───►│  ASICs → TMPP   │
│  Node (local)   │     │  Node (US-West)  │     │  Node (local)   │
│                  │     │  Node (EU)       │     │                  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         gossip                gossip                  gossip
```

- Shares are counted locally (instant)
- Block templates come from your local node (instant)
- Distributions are broadcast across the gossip mesh (all nodes agree)
- Payouts are Merkle-committed in the coinbase (verifiable by anyone)
- If your node finds a block, the entire pool benefits (PPLNS)

### What Your Node Does

1. **Connects to the chain daemon** (bitcoind, kaspad, etc.) for block templates
2. **Runs Stratum V2** for your mining hardware to connect to
3. **Validates shares** and adds them to the PPLNS window
4. **Joins the gossip mesh** to share distributions with other nodes
5. **Serves the dashboard** at `http://localhost:8080`

### What It Doesn't Do

- It doesn't require registration, email, or KYC
- It doesn't send your hashrate data to a third party
- It doesn't depend on external infrastructure for share counting
- It doesn't require you to trust anyone — every payout is verifiable

## Setup

### Quick Start (5 minutes)

```bash
# Download TMPP
curl -L https://github.com/tmpp-protocol/tmpp/releases/latest/download/tmpp-node-linux-x86_64 -o tmpp-node
chmod +x tmpp-node

# Interactive setup (installs chain daemon, configures everything)
./tmpp-node --setup

# Or generate a config file and edit it
./tmpp-node --generate-config
nano ~/.tmpp/node.toml
./tmpp-node
```

### Docker

```bash
docker run -d \
  --name tmpp \
  -p 3337:3337 \
  -p 8080:8080 \
  -v tmpp-data:/data \
  tmpp/tmpp-node \
  --chain=kaspa \
  --payout-address=kaspa:qr... \
  --bitcoind-url=http://kaspad:16110
```

### Point Your Miners

Configure your ASICs/GPUs:

```
Pool URL:    stratum+tcp://YOUR_NODE_IP:3337
Worker:      YOUR_PAYOUT_ADDRESS.rig_name
```

That's it. No account creation. Your payout address IS your account.

## Hardware Requirements

| Scale | Hardware | Cost |
|---|---|---|
| 1-10 miners | Any modern PC or $5/month VPS | Negligible |
| 10-100 miners | 4 CPU cores, 8GB RAM, SSD | ~$30/month |
| 100-1000 miners | 8 cores, 16GB RAM, NVMe | ~$80/month |
| 1000+ miners | 16 cores, 32GB RAM, NVMe | ~$150/month |

The chain daemon (bitcoind, kaspad, etc.) has its own requirements. For Bitcoin, a pruned node needs ~5GB of disk. For Kaspa, the full node needs ~50GB.

## Crash Recovery Guarantee

Every share is written to a write-ahead log (WAL) before being added to the PPLNS window. If your node crashes and restarts:

- **Zero shares lost** — the WAL replays the full PPLNS window
- **Zero credits lost** — the ledger replays un-flushed credit records
- **Zero distributions lost** — reconstructed from credit records or gossip peers

Your miners can trust that every hash they computed will be counted, even through power outages, hardware failures, and software updates.

## Gossip Peers

Connect to the TMPP mesh by adding gossip peers:

```bash
tmpp-node --gossip-peers=us-east.tmpp.io:9940,eu.tmpp.io:9940
```

Or in `node.toml`:

```toml
[gossip]
port = 9940
peers = ["us-east.tmpp.io:9940", "eu.tmpp.io:9940"]
```

The more peers in the mesh, the faster distributions propagate and the more resilient the network is to individual node failures.

## Verification

Every payout is Merkle-committed in the coinbase OP_RETURN. You can verify your share of any block reward:

1. Open the dashboard at `http://localhost:8080`
2. Go to **My Miner** tab
3. Enter your payout address
4. Click **Verify on Bitcoin**
5. See the cryptographic proof that your payout is committed on-chain

No other pool offers this. Your shares are signed claims on block rewards, and the distribution is provably committed to the blockchain. Trust is replaced by math.

## Comparison

|  | Centralized Pool | Solo Mining | TMPP Self-Hosted |
|---|---|---|---|
| Latency | 50-150ms | 0ms | <1ms |
| Stale rate | 1-10% | 0% | ~0% |
| Variance | Low (shared) | Extreme | Low (shared) |
| Privacy | None | Full | Full |
| Fee | 1-4% | 0% | 1% |
| Verification | Trust pool | N/A | Merkle proof |
| Uptime dependency | Remote server | Your node | Your node |
| Setup difficulty | Easy | Hard | Easy |

TMPP self-hosting gives you the low variance of a pool, the privacy of solo mining, and better economics than both.
