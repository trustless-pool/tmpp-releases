# TMPP Development Roadmap

## Phase 1: Core Protocol (Complete)

- [x] **Crypto module** — SHA3-256, SHA256d, keccak256, secp256k1 ECDSA sign/verify, address derivation
- [x] **Share type** — partial PoW proofs with block preimage verification, signature binding
- [x] **Merkle trees** — share tree (SHA3-256) + distribution tree (keccak256 EVM, SHA3-256 Bitcoin)
- [x] **PPLNS window** — rolling window with O(1) dedup, per-miner counts, cached Merkle root
- [x] **Pool ledger** — sled-backed balances, integer-only crediting, reorg rollback
- [x] **Validation** — 7-step share validation chain + withdrawal validation
- [x] **Gasless withdrawal** — cross-chain replay protection via chain_id
- [x] **Coinbase** — keyless pool reserve address, Bitcoin OP_RETURN encode/decode
- [x] **Gossip types** — message types, deferred credit queue, distribution verification
- [x] **Adaptive difficulty** — `adaptive_share_bits()` formula

## Phase 2: Pool Orchestrator & Completeness (Complete)

- [x] **Pool subsystem** — `PoolSubsystem` state machine (Disabled → Active → Decommissioned), share submission pipeline (local + P2P paths), difficulty auto-update, share store with height index, top miners query
- [x] **Minimum withdrawal enforcement** — `MIN_WITHDRAWAL_UNITS = 500` (0.05 VIBE anti-dust), enforced in `validate_pool_withdraw()`
- [x] **Protocol fee** — `ProtocolFee` with configurable basis points (default 50 = 0.5%), applied in `credit_block()`, verified that miner credits + fee = block reward exactly
- [x] **Solo bootstrap mode** — `BootstrapManager` with `MiningMode::Solo`/`Pool` enum, hysteresis on peer count drop, configurable `min_pool_peers` threshold

## Phase 3: Integration Surface (Complete)

- [x] **REST API module** — axum-based: POST /share, GET /balance/{addr}, GET /window, GET /config, POST /withdraw, GET /distribution/{height}, GET /proof/{height}/{addr}
- [x] **Chain integration traits** — `BlockProvider`, `MempoolSubmitter`, `ShareRootCommitter`, `RewardHook`, `DistributionBroadcaster`, blanket `TmppHost` trait

## Phase 4: Bitcoin Adaptation (Complete)

- [x] **Bitcoin share format** — `BitcoinShare` with 76-byte header preimage, SHA256d hash verification, u32 nonce
- [x] **Bitcoin block header** — `BitcoinBlockHeader` with 80-byte serialization, nBits → target expansion, `hash_meets_target()`
- [x] **SHA256d crypto** — `sha256d()`, `sha256d_hex_reversed()` (Bitcoin display convention)
- [x] **Deterministic payout transactions** — `build_deterministic_payouts()` with fee deduction, dust filtering (546 sats), deterministic address sorting, remainder assignment
- [x] **Fraud proofs** — `FraudProof` struct with `verify()`, `detect_fraud()` to compare committed distribution against actual payouts
- [x] **OP_RETURN commitment** — 80-byte `TMPP` magic payload with Merkle root + block height

## Phase 5: Production Hardening (Complete)

- [x] **Benchmarks** — criterion suite at 1K/10K/50K/100K miners
- [x] **Delta encoding** — >90% compression at 100K miners with 5% churn

## Phase 6: Bitcoin Mining Stack (Complete)

- [x] **Stratum V2 server** — SV2 binary framing, Mining Protocol messages, async TCP server
- [x] **Bitcoin RPC client** — getblocktemplate, submitblock, getblockchaininfo with JSON-RPC 2.0
- [x] **Coinbase construction** — BIP34, extranonce split, TMPP OP_RETURN, witness commitment
- [x] **Block assembly** — PreparedBlock, Merkle branches, solution detection, full block serialization
- [x] **Address parsing** — bech32/bech32m decoder for P2WPKH/P2TR
- [x] **P2P gossip network** — wire protocol, peer discovery, share/distribution relay
- [x] **TMPP node binary** — `tmpp-node` with 6 async tasks, full CLI
- [x] **Integration tests** — 9 end-to-end tests (pool lifecycle, block solution, Stratum handshake, gossip, delta, fraud proofs)
- [x] **Testnet deployment** — Docker Compose (2-node + bitcoind), regtest with CPU miners
- [x] **321 tests passing**

## Phase 7: Scaling & Security (Complete)

- [x] **Lazy Merkle root** — dirty flag, recompute only on read. Eliminates ~133M hashes/block at 10K window.
- [x] **Crash recovery** — `last_flushed_height` watermark, credit replay on startup
- [x] **zstd wire compression** — 12-25x compression on gossip frames
- [x] **Wire frame hardening** — gossip 2 MiB cap, Stratum 64 KB cap, field-level validation
- [x] **Share verification harmonization** — unified SHA256d path, eliminated SHA3 string-concat path
- [x] **Security audit** — 4-agent parallel audit: crypto, network, financial, operational (47 findings)
- [x] **Security fixes** — coinbase witness strip bounds checks, PSBT strip rewrite, crash recovery atomicity, constant-time comparisons (`subtle` crate)

## Phase 8: Miner Management & Operations (Complete)

- [x] **MinerTracker module** — per-worker stats, per-address accounts, earnings history
- [x] **Worker identity** — `address.worker_name` format, auto-registration on Stratum connect
- [x] **Hashrate estimation** — computed from share submission rate × difficulty
- [x] **Rejection tracking** — per-reason breakdown (stale, invalid PoW, duplicate, bad timestamp)
- [x] **Health monitoring** — Healthy/Degraded/Offline/Critical per worker
- [x] **Alert system** — worker offline, high reject rate, hashrate drop, stale spike
- [x] **Webhook integration** — `--webhook-url` for Slack/Discord/PagerDuty/custom
- [x] **Rich REST API** — `/api/stats`, `/api/workers`, `/api/miner/:addr`, `/api/miner/:addr/workers`, `/api/miner/:addr/earnings`, `/api/alerts`

## Phase 9: Pool Dashboard & Operator Tools (Complete)

### P1 — Immediate (Complete)

- [x] **Hashrate time-series** — `PoolStatsTracker` stores per-address and pool-wide samples, served via `/api/miner/{addr}/hashrate?period=24h` and `/api/hashrate`.
- [x] **Pool luck tracking** — rolling 10/50/100 block luck, shares-since-last-block. Served via `/api/luck`.
- [x] **Block list** — `/api/blocks` endpoint: height, timestamp, reward, finder, tx count, TMPP Merkle root, accepted status. Paginated.
- [x] **Network stats** — `/api/network`: difficulty, network hashrate, halving countdown, retarget countdown, mempool size. Sourced from bitcoind RPC.
- [x] **Earnings CSV/JSON export** — `/api/miner/{addr}/export?format=csv` with proper Content-Disposition headers.
- [x] **Watcher links** — `POST /api/watcher` creates read-only tokens, `GET /api/watch/{token}` returns limited view. Revocable.
- [x] **Configurable payout threshold** — `POST /api/miner/{addr}/settings`, `GET /api/miner/{addr}/payout`. Default 0.001 BTC, floor 10K sats.
- [x] **Dashboard upgrade** — 5-tab responsive layout (Overview, Miners & Workers, Blocks, Network, Alerts). Health badges, block list, luck, network stats, gossip topology.
- [x] **16 REST API endpoints** live and tested.

### P2 — Short-term (Complete)

- [x] **API keys with permissions** — SHA3-256 hashed, 3 scopes (read/manage/financial), IP whitelist, sled-backed.
- [x] **Worker tagging/grouping** — metadata labels (`site:Dallas`, `rack:A3`), filter by tag, aggregate by tag.
- [x] **Profitability calculator** — `/api/profitability?hashrate_th=100&power_watts=3000&electricity_kwh=0.06`.
- [x] **Earnings breakdown** — `/api/blocks/{height}/breakdown` (subsidy vs tx fees).
- [x] **Telegram/Discord/Slack notifications** — auto-detect platform from URL, Telegram Bot API 9.5 (MarkdownV2), Discord API v10 (embeds), Slack Block Kit.
- [x] **Mobile-responsive dashboard** — 3 breakpoints (900/600/380px), horizontal-scroll tables, touch-friendly tabs.

### P3 — Medium-term (Complete)

- [x] **Worker uptime tracking** — session accumulation, uptime percentage, periodic refresh.
- [x] **Custom alert thresholds** — per-miner configurable reject rate, hashrate drop, offline duration. Merges with pool defaults.
- [x] **Revenue aggregation** — daily earnings grouped by date.
- [x] **Sub-worker grouping** — hierarchical tree: address → site → rack → worker. `group_by_hierarchy()` with configurable levels, `workers_grouped_by()`. GET `/api/miner/{addr}/groups?levels=site,rack`.
- [x] **Batch worker management** — POST `/api/batch/tags` and `/api/batch/thresholds` for bulk operations across multiple workers/miners.
- [x] **Difficulty adjustment alerts** — GET `/api/difficulty` with retarget forecast (blocks until, estimated time, severity: imminent/upcoming/distant).

## Phase 10: Protocol Completion

### 10.1: Gossip Authentication (Complete)
- [x] **Signed gossip messages** — `GossipMessage::Signed` variant wraps distributions and block announcements. Signature + public key + node_id verified on receive.
- [x] **Node signing key** — secp256k1 key generated on startup, node_id derived from public key via SHA3-256. Shares signed by pool node.
- [x] **Peer identity verification** — handshake_sig + public_key in Hello/HelloAck. Verify signature over nonce, verify node_id = SHA3-256(pubkey).
- [x] **Distribution verification** — `verify_distribution_against_coinbase()` verifies gossip distribution Merkle root against committed OP_RETURN root (constant-time comparison).

### 10.2: Stratum Security (Complete)
- [x] **Noise encryption** — `server_handshake()`/`client_handshake()` async functions. `read_encrypted_frame()`/`write_encrypted_frame()` for encrypted SV2 I/O. Header + payload encrypted separately with 16-byte MAC each.
- [x] **Miner authentication** — `--require-auth` flag. Pool sends 32-byte challenge, miner signs with secp256k1 key for their address. BIP-062 enforced. Custom extension messages (0xE0-0xE3).
- [x] **Stratum V1 fallback** — full JSON-RPC implementation: `mining.subscribe`, `mining.authorize`, `mining.set_difficulty`, `mining.notify`, `mining.submit`. Protocol auto-detection (V1 starts with `{`, V2 starts with binary). V1 message encode/decode with newline framing.

### 10.3: Bitcoin Payout Execution (Complete)
- [x] **UTXO tracking** — `UtxoTracker` records coinbase outputs on block found, tracks maturity (100 confirmations), marks spent.
- [x] **Payout pipeline** — `PayoutPipeline` coordinator: periodic cycle checks spendable balance → filters credits above threshold → estimates fee via `estimatesmartfee` → builds transaction → signs with BIP143 P2WPKH sighash → tests mempool acceptance → broadcasts via `sendrawtransaction`.
- [x] **Confirmation monitoring** — pending payouts tracked, confirmations checked each cycle, confirmed payouts remove consumed UTXOs, failed payouts dropped after 20 checks.
- [x] **Real signing** — BIP143 sighash + secp256k1 ECDSA with node signing key. Proper witness stack (signature + compressed pubkey).
- [x] **RPC methods** — `listunspent`, `getrawtransaction`, `gettransaction`, `estimatesmartfee`, `testmempoolaccept` added to BitcoinRpc.
- [x] **Lightning payouts** — LND REST client (v0.20.1-beta), BOLT11 invoice + keysend, channel capacity check, batch processor, `/api/miner/{addr}/lightning` preference endpoint, `--lnd-url` + `--lnd-macaroon` CLI.

### 10.4: Advanced Block Construction (Complete)
- [x] **Witness root recomputation** — `PreparedBlock::from_template` always recomputes witness commitment from transaction wtxids (never trusts bitcoind's default). Correct even when job declaration changes the tx set.
- [x] **Stratum V2 job declaration** — `MiningJobToken` allocation, `DeclaredJob` validation (TMPP commitment check, pool output check, version validation). Miners can construct their own block templates while the pool ensures the TMPP distribution commitment is included.
- [x] **Block template optimization** — `template_optimizer.rs`: greedy fee-maximizing selection with CPFP-aware package fee rates, dependency-respecting topological ordering, weight limit enforcement (4M WU), minimum fee rate filtering. `candidates_from_template()` converts bitcoind templates.

## Phase 11: Security Hardening — Round 2 & 3 (Complete)

- [x] **3 security audit rounds** — 4-agent parallel audits (crypto, network, financial, operational). 11 CRITICAL → 0.
- [x] **Constant-time comparisons** — `subtle::ConstantTimeEq` in Merkle verification, address comparison, gossip Merkle check, Stratum auth nonce check.
- [x] **Atomic double-credit guard** — sled `compare_and_swap` in `credit_block_fast()`.
- [x] **Deterministic remainder** — lexicographic address tiebreaker for PPLNS remainder.
- [x] **Crash recovery order** — balances flushed before watermark (never ahead of persisted data).
- [x] **Checked arithmetic** — `saturating_mul`, `checked_add`, `saturating_sub` in payout pipeline.
- [x] **UTXO double-spend protection** — mark spent before broadcast, revert on failure.
- [x] **zstd decompression bomb guard** — streaming decoder with 2 MB compressed / 8 MB decompressed limits.
- [x] **Miner auth nonce replay** — used nonce tracking with 5-minute expiry, constant-time comparison.
- [x] **Noise nonce overflow guard** — assert counter < u64::MAX before increment.
- [x] **Peer nonce eviction** — ordered VecDeque with timestamp-based FIFO expiry (replaced random HashSet).
- [x] **Fee rate cap** — `MAX_FEE_RATE = 500 sat/vB` prevents runaway estimation.
- [x] **RPC timeout** — 30-second HTTP client timeout prevents hung bitcoind from freezing node.
- [x] **addr_to_bytes hardening** — SHA3-256 hash for non-EVM addresses instead of silent zero-padding.
- [x] **CPU miner block detection** — proper lexicographic byte-by-byte comparison (was `.any()`).

## Phase 12: Distribution Recovery (Complete)

- [x] **RequestDistribution / DistributionResponse** — fully wired in gossip protocol with Merkle verification.
- [x] **Startup recovery** — detects missing distributions (gossip gap) and full sled wipe (0 accounts). Queries bitcoind for chain height, requests all missing heights from peers in batches.
- [x] **credit_from_recovered_distribution()** — CAS-guarded crediting from peer-provided distribution entries.
- [x] **Signet support** — `getblocktemplate` includes `"signet"` rule for signet compatibility.

## Phase 13: First-Run Onboarding

- [x] **Interactive setup wizard** — `--setup` flag, detects existing bitcoind, interactive menu.
- [x] **Bitcoind auto-install** — downloads Bitcoin Core v28.0, platform auto-detect, curl + extract.
- [x] **Storage mode selection** — Pruned (5 GB) vs Full + txindex (600 GB).
- [x] **Network selection** — Mainnet vs Signet.
- [x] **bitcoin.conf generation** — RPC credentials (random 32-char), prune/txindex, localhost-only.
- [x] **Config persistence** — save/load `~/.tmpp/setup.json` across restarts.
- [x] **Managed bitcoind updates** — `check_for_updates()` queries GitHub releases API, `check_and_prompt_update()` runs on startup with interactive prompt (update/skip/skip-version). `perform_update()` downloads, stops bitcoind gracefully, replaces binary. Version pinning via skip file.
- [x] **Infrastructure API & webhooks** — `GET /api/node/status` (chain, sync progress, block height, reachability), `GET /api/node/updates` (current/latest version, download URL), `GET /api/node/events` (recent infra events). Webhook events: `bitcoind.update_available`, `bitcoind.sync_stalled`, `bitcoind.disk_low`, `bitcoind.version_eol`, `bitcoind.node_down`, `bitcoind.node_up`. Auto-formatted for Slack/Discord/Telegram/Generic via notifications module.

## Phase 14: Ecosystem & Adoption

- [ ] **VibeChain integration** — swap vibechain-pool internals to use TMPP as a dependency
- [ ] **arXiv paper** — formal protocol spec with pseudocode, security proofs, benchmark data
- [x] **Bitcoin signet deployment** — Docker Compose with signet bitcoind + 2 TMPP nodes + 4 CPU miners
- [x] **Mainnet deployment guide** — DEPLOYMENT.md: hardware tiers (2GB→32GB), disk/bandwidth tables, quick start (wizard + manual + Docker), full CLI reference, firewall rules, security checklist, backup/recovery, benchmark data, scaling recommendations, troubleshooting.
- [x] **Formal verification** — 8 Kani proof harnesses + 6 validation tests. Properties proven: fee conservation (fee+net=reward), credit conservation (sum=reward for 2 and 3 miners), remainder bound (<n_miners), payout no-overdraft (inputs>=outputs+fees), difficulty target leading zeros, fee overflow safety (u128 path), sats-to-BTC roundtrip. Run with `cargo kani` on Linux.
- [x] **Covenant readiness** — Full OP_CTV (BIP-119) + OP_CAT (BIP-347) implementation. CTV template hash computation, CTV locking scripts, payment tree for 1M+ miners (auto-splits at 250 outputs/tx), OP_CAT covenant verification script, covenant-aware payout builder with status detection (NotAvailable/CtvActive/CatActive/BothActive), satoshi conservation proven in tests. 11 tests. Ready to activate the moment either opcode ships.
- [x] **Third-party integrations** — Miningcore-compatible API (`/api/v1/pools/tmpp-btc/*`), BTC.com format (`/api/btccom/*`), F2Pool format (`/api/f2pool/*`), HiveOS format (`/api/hiveos/*`). 7 endpoints, 6 tests. Compatible with Foreman, Awesome Miner, minerstat, HiveOS dashboards.
- [x] **Tax reporting** — `GET /api/miner/{addr}/tax?format=koinly` exports CSV compatible with CoinTracker, Koinly, CoinLedger, TurboTax, TaxBit, and generic format. CSV formula injection protection. `?summary=true` returns JSON earnings summary. 11 tests.

## Phase 15: Scaling Optimizations (Future)

Opportunities to reduce per-block overhead further. Current: ~480 ms at 1M miners (0.08%).

### Completed
- [x] **Binary credit serialization** — replaced JSON with compact binary format. 1M entries: ~200ms → ~16ms. 10x faster.
- [x] **HashMap entry API** — avoids double-lookup per balance update in credit loop.
- [x] **Lightweight window snapshot** — removed 32 MB share_id copy (1M × 32 bytes). Snapshot now O(unique_miners) not O(total_shares).

### Potential Optimizations
- [ ] **Parallel credit computation** — split 1M miners into chunks, compute credits per chunk on separate cores (rayon). Currently single-threaded at ~10ms for the math, ~200ms for cache updates. Parallel cache updates with sharded HashMap could reduce to ~50ms on 8 cores.
- [ ] **Arena allocator for distribution** — avoid per-miner String allocation during credit computation. Use a bump allocator for the credits Vec, reuse across blocks. Saves ~50ms of allocation overhead at 1M miners.
- [ ] **Incremental distribution Merkle** — instead of rebuilding the full distribution tree each block, maintain an incremental tree and update only changed entries. With 5% churn, updates 50K of 1M entries → ~10ms instead of ~150ms. Requires careful invalidation logic.
- [ ] **SIMD hash acceleration** — use SHA256 SIMD intrinsics (sha2 crate with `asm` feature) for Merkle computation. ~2x speedup on CPUs with SHA-NI extensions (most modern x86_64). Would reduce Merkle roots from ~270ms to ~135ms.
- [ ] **Memory-mapped sled** — configure sled with memory-mapped I/O for the credit tree. Reduces flush latency and avoids serialization overhead for reads. May improve credit_block_fast by ~20%.
- [ ] **Pre-sorted address index** — maintain a sorted Vec<(u64_hash, u32_index)> alongside the HashMap for O(1) lookups without String hashing. Reduces cache update from ~200ms to ~50ms by avoiding String comparison overhead.
- [ ] **Batch gossip compression** — aggregate multiple blocks' distributions into a single zstd frame when catching up peers. Reduces per-message overhead and improves compression ratio by 2-3x during recovery.
- [ ] **Zero-copy window counts** — use a concurrent HashMap (dashmap) for per-miner share counts to avoid taking a write lock on the entire window for each share push. Reduces contention under high share rates.

### Projected Impact (all optimizations applied)

| Component | Current (1M) | Optimized | Reduction |
|---|---|---|---|
| Share Merkle (SIMD + incremental) | 120 ms | ~30 ms | 75% |
| Distribution Merkle (SIMD + incremental) | 150 ms | ~20 ms | 87% |
| Credit computation (parallel + arena) | 200 ms | ~50 ms | 75% |
| Window snapshot | 10 ms | ~5 ms | 50% |
| **Total** | **480 ms** | **~105 ms** | **78%** |
| **Overhead** | **0.08%** | **~0.02%** | |

These are theoretical projections. Each optimization should be benchmarked individually before committing. The current 0.08% overhead is already well within Bitcoin's 600-second block interval — these optimizations become relevant only if block intervals shrink (e.g., sidechains) or miner counts exceed 1M.

## Phase 16: Multi-Chain Expansion

### Tier 1 — High Impact (In Progress)
- [x] **Bitcoin (BTC)** — SHA256d, full integration, 8-node mesh proven
- [x] **Dogecoin (DOGE)** — Scrypt, onboarding, Docker compose, CPU miner chain flag, regtest blocks accepted
- [x] **Litecoin (LTC)** — Scrypt, segwit, MWEB rules, Docker compose, 2-node regtest proven
- [x] **Bitcoin Cash (BCH)** — SHA256d, no segwit, Docker compose, 2-node regtest proven (2100+ blocks)
- [x] **Kaspa (KAS)** — Full 4-step kHeavyHash (blake2b + cSHAKE256/keccak + matrix multiply + HeavyHash), 1-second blocks, kaspactl gRPC bridge, Docker compose, 166 blocks accepted on devnet
- [x] **Alephium (ALPH)** — Double-Blake3 PoW, binary TCP mining protocol (port 10973), chain index verification, 4×4 sharded, Docker compose, 148+ blocks accepted on devnet

### Tier 2 — Medium Impact
- [x] **Monero (XMR)** — RandomX (placeholder, needs C FFI for production), 2-min blocks, CryptoNote Base58 addresses, monerod JSON-RPC, tx_extra for Merkle commitment. MoneroAdapter with stagenet support. Privacy-first trustless pool.
- [ ] **Ethereum Classic (ETC)** — Etchash, $3B, large GPU mining community
- [ ] **Ravencoin (RVN)** — KawPoW, $300M, GPU-friendly

### Tier 3 — Quick Wins
- [ ] **Nervos (CKB)** — Eaglesong, $300M
- [ ] **Sia (SC)** — Blake2b, $100M, blake2 crate exists
- [ ] **Handshake (HNS)** — Blake2b+SHA3, $50M, both hashes already in codebase

## Phase 17: Architecture Refactor (Complete)

- [x] **ChainMiner trait** — `src/mining/mod.rs` with `ChainMiner` trait (`run()`, `name()`). Kaspa and Alephium extracted to `src/mining/kaspa.rs` and `src/mining/alephium.rs`. Node binary dispatches via `create_miner()`. Adding a new chain = one file + one match arm. Removed 384 lines from node.rs.
- [x] **Split node.rs** — Mining loops extracted into `src/mining/`. Dashboard route extraction deferred (axum closures with captured Arc clones resist clean separation). node.rs reduced from 2801 to 2417 lines.
- [x] **RPC trait** — Evaluated and skipped. All Bitcoin-family chains (BTC, DOGE, LTC, BCH) use identical `BitcoinRpc`. Kaspa and Alephium use the `ChainMiner` trait path instead. No second RPC implementation exists to justify a trait.
- [x] **Config consolidation** — `NodeConfig` in `src/config.rs` with TOML support. Nested sections: `daemon`, `stratum`, `pool`, `gossip`, `dashboard`, `notifications`. Loaded from `~/.tmpp/node.toml` automatically. `--generate-config` creates template. CLI flags override file config.

## Phase 18: Documentation

### Completed
- [x] **SELF-HOSTING.md** — Why self-hosting wins (latency, stale rates, privacy, economics), setup guide, hardware requirements, crash recovery guarantees, gossip mesh, verification
- [x] **DEPLOYMENT.md** — Hardware tiers, quick start, Docker, CLI reference, firewall rules, security checklist, backup/recovery, scaling recommendations
- [x] **ROADMAP.md** — This file
- [x] **WHITEPAPER.md** — Protocol specification

### Completed (this session)
- [x] **GETTING-STARTED.md** — 2-minute quickstart for hosted miners. Stratum URLs, worker format, dashboard walkthrough, Lightning payouts, FAQ, comparison with Ocean/P2Pool
- [x] **HARDWARE-GUIDE.md** — Recommended hardware (Beelink N100), step-by-step setup, network placement, systemd service, firewall rules, backups, FAQ
- [x] **INSTITUTIONAL.md** — Boardroom pitch for 50+ MW operations. Latency math ($650-750K/year recovered), block race advantage, uptime independence, audit/verification, deployment guide, security summary

### Remaining
- [ ] **OPERATOR-GUIDE.md** — Running TMPP as infrastructure: multi-node deployment, HAProxy config, rolling upgrades, monitoring with Prometheus/Grafana, backup strategy, scaling playbook
- [ ] **PROTOCOL-SPEC.md** — Formal protocol specification: Stratum V2 extensions, gossip wire format, distribution Merkle tree construction, PPLNS algorithm, payout pipeline, share validation chain
- [ ] **API-REFERENCE.md** — All REST API endpoints with request/response examples, authentication, rate limits
- [ ] **SECURITY.md** — Threat model, attack surfaces (block withholding, selfish mining, share grinding, Sybil), mitigations, audit history, constant-time comparisons, crash recovery guarantees
- [ ] **CHAIN-INTEGRATION.md** — How to add a new chain: ChainMiner trait, RPC client, PoW algorithm, Docker compose template, testing checklist
- [ ] **arXiv paper** — Formal academic paper: protocol spec with pseudocode, security proofs, benchmark data, comparison with Stratum V1/V2, P2Pool, Ocean TIDES

## Features That Don't Apply (By Design)

These are intentionally absent because TMPP is trustless and decentralized:

| Feature | Why Not |
|---|---|
| PPS/FPPS/PPS+ | Requires pool treasury to absorb variance = trusted operator |
| Sub-accounts with email/password | Your Bitcoin address IS your account. Keys > passwords. |
| KYC/AML | No central operator to enforce. Permissionless by design. |
| SOC 2 compliance | No entity to audit. The protocol IS the audit. |
| Custody services | No custody. Funds committed on-chain via Merkle proofs. |
| Forward hashrate contracts | Financial derivative requiring counterparty trust. |
| 90-day inactivity expiry | On-chain balances don't expire. Your keys, your coins. |
| Custom ASIC firmware | Hardware vendor domain. TMPP speaks standard Stratum V2. |
| Profit switching / multi-coin | Bitcoin-native. Miners choose their chain. |
| Remote ASIC management | Miner's firmware handles this, not the pool. |
