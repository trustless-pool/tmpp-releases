# TMPP Node Hardware Guide

## What You Need

A TMPP node runs on any modern computer. The hardware is commodity — a $179 mini PC from Amazon is more than enough for most mining operations.

## Recommended Hardware

### For Most Miners (1-100 ASICs/GPUs)

**Beelink S12 Pro or EQ12** — ~$179

- Intel N100 (4 cores, 3.4 GHz burst)
- 16GB DDR5 RAM
- 500GB NVMe SSD
- Dual Gigabit Ethernet
- 12W TDP (runs silently)
- Fits in your server rack or sits on a shelf

[Amazon link — search "Beelink N100 16GB 500GB"]

This is the same class of hardware that thousands of Umbrel Bitcoin nodes run on. It's proven, cheap, and silent.

### For Large Operations (100-1000+ ASICs)

**Any mini PC or 1U server with:**

- 8+ CPU cores (Intel N305, AMD Ryzen 5, or better)
- 16-32GB RAM
- 1TB NVMe SSD
- Gigabit or 10GbE networking

Cost: $200-500. Pays for itself in recovered stale shares within hours.

### For Kaspa/Alephium (Fast-Block Chains)

Same hardware. Kaspa's 1-second blocks don't require more CPU — they require less latency, which is exactly what a local node provides.

## Setup (15 Minutes)

### Step 1: Install the OS

Any Linux works. Ubuntu Server 22.04 LTS is recommended:

```bash
# Download Ubuntu Server 22.04 LTS
# Flash to USB with Rufus (Windows) or dd (Linux/Mac)
# Boot the mini PC from USB
# Follow the installer (defaults are fine)
```

### Step 2: Install TMPP

```bash
# Download the latest TMPP release
curl -L https://github.com/tmpp-protocol/tmpp/releases/latest/download/tmpp-node-linux-x86_64 -o tmpp-node
chmod +x tmpp-node

# Run the interactive setup wizard
./tmpp-node --setup
```

The wizard will:
1. Ask which chain you want to mine (Bitcoin, Kaspa, Dogecoin, etc.)
2. Download and install the chain daemon automatically
3. Ask for your payout address
4. Generate configuration files
5. Start syncing the blockchain

### Step 3: Wait for Sync

Bitcoin pruned: 1-3 days (first time only)
Kaspa: 2-4 hours
Dogecoin: 1-2 hours
Litecoin: 2-4 hours

You can monitor sync progress at `http://YOUR_NODE_IP:8080`

### Step 4: Point Your Miners

Once synced, configure your ASICs/GPUs:

```
Pool URL:    stratum+tcp://YOUR_NODE_IP:3337
Worker:      YOUR_PAYOUT_ADDRESS.rig_name
Password:    (anything or leave blank)
```

Example for a Kaspa GPU miner:
```
Pool URL:    stratum+tcp://192.168.1.100:3337
Worker:      kaspa:qr6pm...x3k9me.gpu-rig-01
```

### Step 5: Join the Gossip Mesh

Connect your node to the TMPP network:

```bash
# Edit config
nano ~/.tmpp/node.toml
```

Add peers:
```toml
[gossip]
port = 9940
peers = ["us-east.tmpp.io:9940", "eu.tmpp.io:9940"]
```

Restart the node. You're now part of the decentralized pool.

## Running on Your Mining Rig (Recommended for GPU Miners)

You don't need a separate computer. The TMPP node runs on the **same machine** as your GPUs. It uses minimal resources — less than 5% of a single CPU core and under 500MB of RAM. Your GPUs won't notice it's there.

This is actually the **best possible setup** because network latency between your miner and the pool node is literally zero — they communicate through memory, not the network.

### Setup on a GPU Mining Rig

```bash
# Install TMPP (runs alongside your mining software)
curl -L https://github.com/trustless-pool/TMPP/releases/latest/download/tmpp-node-linux-x86_64 -o tmpp-node
chmod +x tmpp-node
./tmpp-node --setup

# TMPP runs as a background service
sudo systemctl start tmpp

# Point your GPU miner at localhost
# In lolMiner:
lolMiner --algo KASPA --pool stratum+tcp://127.0.0.1:3337 --user kaspa:qr...YOUR_ADDRESS.rig1

# In BzMiner:
bzminer -a kaspa -p stratum+tcp://127.0.0.1:3337 -w kaspa:qr...YOUR_ADDRESS.rig1
```

### Why This Works

| Resource | TMPP Node Usage | Available on a Mining Rig |
|---|---|---|
| CPU | <5% of one core | Mining rigs have idle CPU (GPUs do the hashing) |
| RAM | 200-500 MB | Mining rigs typically have 8-32 GB |
| Disk | 50 GB (Kaspa full node) | SSDs are standard |
| Network | <1 Mbps | Minimal |

Your GPU mining software (lolMiner, BzMiner, GMiner) handles the hashing. The TMPP node handles the pool protocol. They coexist without competing for resources because GPUs and CPUs are independent.

### The Latency Advantage

When the TMPP node and GPU miner run on the same machine:
- Share submission: **0ms** (inter-process communication, no network)
- New job delivery: **0ms** (same machine)
- Stale rate: **0%** (physically impossible to have network delay)

On a remote pool, every share crosses the internet (40-150ms round trip). On your own machine, it's instant. For Kaspa with 1-second blocks, this difference is the difference between 5-10% stale rate and zero.

## Network Setup

### Firewall Rules

Your TMPP node needs:

| Port | Protocol | Direction | Purpose |
|---|---|---|---|
| 3337 | TCP | Inbound (LAN only) | Kaspa Stratum (miners connect here) |
| 8080 | TCP | Inbound (LAN only) | Dashboard + API |
| 9940 | TCP | Outbound | Gossip mesh (connects to other TMPP nodes) |
| 8332* | TCP | Localhost | Chain daemon RPC (*varies by chain) |

**Do NOT expose ports 3337 or 8080 to the internet** unless you intentionally want external miners to connect. For a facility running its own ASICs, these should be LAN-only.

### Optimal Network Placement

```
Internet ──→ Firewall ──→ Switch ──┬──→ ASIC Rack 1
                                   ├──→ ASIC Rack 2
                                   ├──→ ASIC Rack 3
                                   └──→ TMPP Node (same switch!)
```

The TMPP node should be on the **same network switch** as your mining hardware. This gives sub-millisecond latency for share submission and template delivery. Don't put it behind a separate firewall or on a different VLAN than your miners.

## Running as a Service

### systemd (recommended)

```bash
sudo tee /etc/systemd/system/tmpp.service << 'EOF'
[Unit]
Description=TMPP Mining Pool Node
After=network.target

[Service]
Type=simple
User=tmpp
ExecStart=/usr/local/bin/tmpp-node
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable tmpp
sudo systemctl start tmpp
```

### Monitoring

- Dashboard: `http://YOUR_NODE_IP:8080`
- Health check: `curl http://YOUR_NODE_IP:8080/healthz`
- Logs: `journalctl -u tmpp -f`

## Maintenance

### Updates

The TMPP dashboard shows a banner when a new version is available. Click **Upgrade Now** to update in-place. Or manually:

```bash
curl -L https://github.com/tmpp-protocol/tmpp/releases/latest/download/tmpp-node-linux-x86_64 -o /usr/local/bin/tmpp-node
chmod +x /usr/local/bin/tmpp-node
sudo systemctl restart tmpp
```

### Backups

TMPP stores all data in `~/.tmpp/data/` (or your configured `--data-dir`). The critical files:

- `db/` — sled database (balances, credit records, share WAL)
- `node.toml` — configuration

```bash
# Backup
tar czf tmpp-backup-$(date +%Y%m%d).tar.gz ~/.tmpp/data/

# Restore
tar xzf tmpp-backup-YYYYMMDD.tar.gz -C ~/
```

### Chain Daemon

The chain daemon (bitcoind, kaspad, etc.) needs its own disk space and maintenance. TMPP's setup wizard handles initial installation. For updates, use the dashboard notification or:

```bash
# Bitcoin Core
bitcoin-cli stop
# Download new version, replace binary
bitcoind -daemon
```

## FAQ

**Q: Can I run TMPP on a Raspberry Pi?**
A: Yes for Kaspa, Dogecoin, Litecoin (lightweight chains). Not recommended for Bitcoin (needs more RAM for block validation). An N100 mini PC is the sweet spot — more powerful, similar price, x86 compatibility.

**Q: Do I need a static IP?**
A: No. Your miners connect to the node via LAN IP. The gossip mesh uses outbound connections (your node connects to peers, not the other way around). Dynamic IPs work fine.

**Q: What if my internet goes down?**
A: Your TMPP node and miners are on the same LAN. Mining continues — shares are counted locally. When internet returns, the gossip mesh reconnects and distributions sync. You might miss other nodes' block announcements during the outage, but your own shares are safe.

**Q: Can I run multiple chains on one node?**
A: Not currently — each TMPP node runs one chain. Run separate instances for each chain (they use different ports). A single N100 can handle 2-3 chains simultaneously.

**Q: How much bandwidth does TMPP use?**
A: Minimal. Stratum V2 is a binary protocol — each share is ~100 bytes. Gossip messages are zstd-compressed. A busy node with 100 miners uses <1 Mbps. The chain daemon's initial sync uses more bandwidth than TMPP ever will.
