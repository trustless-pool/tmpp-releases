# TMPP for Institutional Mining Operations

## Executive Summary

TMPP eliminates the three largest hidden costs of mining pool dependency: stale share losses, pool fees, and infrastructure risk. For a 50 MW mining operation, the annual value is $500K-$1M in recovered revenue. The hardware investment is a $200 mini PC.

## The Problem

Your mining operation sends every hash across the country to someone else's server. You pay them 2-4% of your revenue for this service. You lose 0.3-0.8% of your revenue to stale shares caused by network latency. And if their server goes down, your entire operation stops earning.

### Current Architecture

```
50 MW Facility        40-80ms         Pool Server
(West Texas)    ←───────────────→    (Virginia)
                                          │
  ASICs generate                    Pool validates
  shares locally                    shares remotely
                                          │
  Template updates                  Templates sent
  arrive 40-80ms late               from remote server
```

### What This Costs You

For a 50 MW operation earning $30-50M annually:

| Cost | Amount | Notes |
|---|---|---|
| Pool fee (2%) | $600K-1M/year | Paid to pool operator |
| Stale shares (0.5%) | $150K-250K/year | Lost to network latency |
| Block race losses | $50K-200K/year | Competitor closer to pool server wins ties |
| Downtime risk | Variable | Pool outage = total revenue halt |
| **Total** | **$800K-1.45M/year** | |

## The Solution

A TMPP node runs inside your facility, on your network, next to your ASICs.

### TMPP Architecture

```
50 MW Facility                          TMPP Gossip Mesh

  ASICs ── <1ms ── TMPP Node ──── gossip ──── Other TMPP Nodes
  (same switch)    (your rack)                 (global)
                       │
                   bitcoind
                   (your rack)
```

### What Changes

| Metric | Before (Remote Pool) | After (TMPP Local) |
|---|---|---|
| Share latency | 40-80ms | <1ms |
| Template latency | 40-80ms | <1ms |
| Stale rate | 0.3-0.8% | ~0% |
| Pool fee | 2-4% | 1% |
| Block submission | 40-80ms to pool → pool to network | <1ms to local node → node to network |
| Infrastructure dependency | Pool server uptime | Your server uptime |
| Hashrate privacy | Pool knows everything | Nothing leaves your facility |
| Revenue verification | Trust pool accounting | Cryptographic Merkle proof |

### Annual Impact

For a 50 MW operation at $40M annual revenue:

| Improvement | Value |
|---|---|
| Fee reduction (2% → 1%) | **$400,000/year** |
| Stale share elimination (0.5% → ~0%) | **$200,000/year** |
| Block race advantage | **$50,000-150,000/year** |
| **Total recovered revenue** | **$650,000-750,000/year** |

Hardware cost: $200 (one-time). ROI: measured in hours.

## How It Works

### Still a Pool — Not Solo Mining

Self-hosting TMPP does not mean solo mining. Your node joins a global pool via the gossip network:

- Your miners submit shares to your local node (instant)
- Your node participates in the PPLNS window with all other TMPP nodes
- When any TMPP node finds a block, all miners in the window share the reward
- Distributions are Merkle-committed in the Bitcoin coinbase — verifiable by anyone
- Variance is smoothed across the entire pool, same as any traditional pool

The difference: your shares never cross the internet. Your templates are always fresh. Your block submissions have zero latency to the chain daemon.

### Block Race Advantage

When your ASICs find a block:

**Traditional pool:** ASIC → 40ms to pool server → pool validates → pool submits to Bitcoin Core → block propagates. Total: 60-120ms before the network sees your block.

**TMPP:** ASIC → <1ms to local TMPP node → local Bitcoin Core → block propagates. Total: 5-10ms before the network sees your block.

If a competing miner finds a block at the same height at the same time, the block that propagates first wins. With TMPP, you have a 50-110ms head start in every block race. Over thousands of blocks per year, this advantage compounds.

### Uptime Independence

Your TMPP node depends on:
- Your own server (in your rack, on your UPS)
- Your own Bitcoin Core instance (in your rack)
- Internet connectivity (for gossip mesh)

It does NOT depend on:
- A third-party pool server's uptime
- A third-party pool's DDoS protection
- A third-party pool's engineering team's on-call rotation

If your internet goes down temporarily, your miners keep submitting shares to the local node. When connectivity returns, the gossip mesh syncs. No shares lost.

If a traditional pool goes down — as Foundry experienced in 2023 — every miner pointed at that pool stops earning until someone manually reconfigures to a backup. At institutional scale, this scramble costs millions in aggregate lost revenue.

## Verification and Audit

Every block reward distribution is committed to the blockchain via a Merkle root in the coinbase OP_RETURN output. Your finance team can independently verify:

1. The exact share count attributed to your address
2. The proportional reward calculation
3. The Merkle proof linking your payout to the on-chain commitment

This is not a dashboard screenshot from a pool website. It's a cryptographic proof anchored in the Bitcoin blockchain. No other pool offers this level of auditability.

For operations subject to financial audits, TMPP provides a verifiable chain of custody from hash computation to payout that no centralized pool can match.

## Deployment

### Hardware

One mini PC per chain. Recommended: Intel N100 with 16GB RAM, 500GB NVMe. Cost: $179-250.

For operations running 100+ MW, a dedicated 1U server ($500-800) provides headroom, but the N100 handles most workloads.

### Installation

```bash
# Download
curl -L https://github.com/tmpp-protocol/tmpp/releases/latest -o tmpp-node

# Setup wizard (installs Bitcoin Core, configures everything)
./tmpp-node --setup

# Or use Docker
docker-compose -f docker-compose.production.yml up -d
```

### Network

Place the TMPP node on the same network switch as your ASICs. Configure firewall to allow LAN access on port 3337 (Kaspa Stratum) and outbound on port 9940 (gossip). No inbound internet access required.

### Miner Configuration

```
Pool URL:   stratum+tcp://192.168.1.100:3337
Worker:     kaspa:YOUR_PAYOUT_ADDRESS.facility_01
```

No accounts. No registration. No KYC. The payout address IS the account.

### Monitoring

- Web dashboard: `http://192.168.1.100:8080`
- Health API: `GET /healthz` (for Nagios/Prometheus/Grafana integration)
- Webhook alerts: Slack, Discord, PagerDuty, or custom endpoint
- Per-worker health monitoring with configurable thresholds

### Redundancy

For zero-downtime operations, run two TMPP nodes behind an internal load balancer:

```
ASICs ──→ HAProxy ──┬──→ TMPP Node A (primary)
                    └──→ TMPP Node B (standby)
                         Both on gossip mesh
```

Rolling upgrades: drain Node A → upgrade → bring back → drain Node B → upgrade. Zero miner interruption.

## Security

- **Share integrity:** Every share is cryptographically signed by the miner's key and PoW-verified. Cannot be forged.
- **Credit integrity:** Atomic compare-and-swap prevents double-crediting. Write-ahead log ensures zero share loss through crashes.
- **Payout integrity:** Merkle-committed in the coinbase. Verifiable on-chain by anyone.
- **Network integrity:** Gossip messages are signed with node keys. Distribution Merkle roots are verified against coinbase commitments using constant-time comparisons.
- **Operational integrity:** Three rounds of security audits completed. Formal verification with Kani proof harnesses for core financial invariants (fee conservation, credit conservation, no-overdraft).

## Supported Chains

| Chain | Algorithm | Block Time | Status |
|---|---|---|---|
| Bitcoin (BTC) | SHA256d | 10 min | Production ready |
| Kaspa (KAS) | kHeavyHash | 1 sec | Production ready |
| Litecoin (LTC) | Scrypt | 2.5 min | Production ready |
| Dogecoin (DOGE) | Scrypt | 1 min | Production ready |
| Bitcoin Cash (BCH) | SHA256d | 10 min | Production ready |
| Alephium (ALPH) | Double-Blake3 | 64 sec | Production ready |

## Contact

For institutional deployment support, partnership inquiries, or custom integration:

[Contact information to be added]
