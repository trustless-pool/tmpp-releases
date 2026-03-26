# Getting Started with TMPP

## Connect in 2 Minutes (No Node Required)

You don't need to install anything. Point your miner at a TMPP hosted node and start earning.

### Step 1: Configure Your Miner

Set these in your ASIC/GPU miner's pool settings:

**Bitcoin:**
```
Pool URL:   stratum+tcp://btc.tmpp.io:3333
Worker:     YOUR_BTC_ADDRESS.rig_name
Password:   x
```

**Kaspa:**
```
Pool URL:   stratum+tcp://kas.tmpp.io:3337
Worker:     YOUR_KAS_ADDRESS.rig_name
Password:   x
```

**Dogecoin:**
```
Pool URL:   stratum+tcp://doge.tmpp.io:3334
Worker:     YOUR_DOGE_ADDRESS.rig_name
Password:   x
```

**Litecoin:**
```
Pool URL:   stratum+tcp://ltc.tmpp.io:3335
Worker:     YOUR_LTC_ADDRESS.rig_name
Password:   x
```

Replace `YOUR_ADDRESS` with your actual wallet address. Replace `rig_name` with any name for your mining device (e.g., `asic-01`, `gpu-rig`, `basement`).

### Step 2: Check Your Dashboard

Open the dashboard in your browser:

- Bitcoin: `https://btc.tmpp.io`
- Kaspa: `https://kas.tmpp.io`
- Dogecoin: `https://doge.tmpp.io`

Click **My Miner**, enter your payout address, and see:
- Your hashrate and worker status
- Shares submitted and earnings
- Estimated daily income
- Progress toward next payout
- Payout history with transaction IDs

### Step 3: Get Paid

Payouts happen automatically when your balance reaches the threshold:

| Chain | Default Threshold | Typical Time |
|---|---|---|
| Bitcoin | 0.001 BTC | Depends on hashrate |
| Kaspa | 100 KAS | Hours to days |
| Dogecoin | 50 DOGE | Hours to days |
| Litecoin | 0.01 LTC | Hours to days |

Want faster payouts? Enable **Lightning** in the My Miner tab — get paid every hour with near-zero fees.

You can adjust your payout threshold in the dashboard under **My Miner → Payout Method**.

That's it. No registration. No email. No KYC. Your address is your account.

## Why TMPP?

### Lower Fees
1% pool fee. F2Pool charges 4%. Ocean charges 2%. You keep more of what you mine.

### Trustless Verification
Every payout is Merkle-committed in the blockchain. Click **Verify on Bitcoin** in the dashboard to cryptographically prove your earnings. No other pool offers this.

### Lightning Payouts
Small miner? Enable Lightning and get paid every hour instead of waiting days to hit the on-chain threshold. Near-zero fees.

### Multi-Chain
One pool protocol for Bitcoin, Kaspa, Dogecoin, Litecoin, Bitcoin Cash, and Alephium. Same dashboard, same trustless verification, same low fee.

### No Account Required
Your payout address IS your account. No signup forms, no email verification, no passwords. Point your miner and earn.

## FAQ

**Q: How do I know TMPP is paying me fairly?**
A: Every block reward distribution is cryptographically committed in the coinbase transaction. Go to My Miner → Verify on Bitcoin to see the Merkle proof. You can independently verify your share of every block on any block explorer.

**Q: What if TMPP goes down?**
A: If you're using a hosted node and it goes down, your miner will automatically reconnect when it comes back. No shares are lost — only the time during the outage. For zero downtime, run your own TMPP node (see [SELF-HOSTING.md](SELF-HOSTING.md)).

**Q: Can I switch back to my old pool?**
A: Yes. Change the pool URL in your miner. Any unpaid balance stays in the TMPP ledger — if you reconnect later, it's still there.

**Q: Is there a minimum hashrate?**
A: No. Any hashrate works. Small miners can enable Lightning payouts for hourly payments instead of waiting for the on-chain threshold.

**Q: How is TMPP different from Ocean?**
A: Similar philosophy (trustless, transparent), but TMPP supports 6 chains (Ocean is Bitcoin-only), has lower fees (1% vs 2%), and miners can self-host nodes for zero-latency mining. Both offer on-chain payout verification.

**Q: How is TMPP different from P2Pool?**
A: P2Pool requires every miner to run a full node and produces many small on-chain payouts (expensive in fees). TMPP batches payouts efficiently and lets miners connect to hosted nodes without running anything — while still offering the same trustless guarantees via Merkle commitments.

## Get Notifications

Set up alerts in the dashboard under **My Miner → Notifications**:
- Paste a Slack, Discord, or Telegram webhook URL
- Choose which events to be notified about (payouts, worker offline, blocks found, hashrate drops)
- Platform auto-detected from the URL

## Need Help?

- Dashboard: Check the tooltip on any stat card (hover over it)
- GitHub: [github.com/tmpp-protocol/tmpp](https://github.com/tmpp-protocol/tmpp)
- Discord: [Join the TMPP community]

## Run the Pool Node on Your Mining Rig

GPU miners can run the TMPP node on the same machine as their mining software. No extra hardware needed — it uses <5% CPU. Point your miner at `localhost` instead of a remote server and get **zero latency, zero stale shares**.

```bash
# Install TMPP on your mining rig
curl -L https://github.com/trustless-pool/TMPP/releases/latest/download/tmpp-node-linux-x86_64 -o tmpp-node
chmod +x tmpp-node && ./tmpp-node --setup

# Point your GPU miner at localhost
lolMiner --algo KASPA --pool stratum+tcp://127.0.0.1:3337 --user kaspa:qr...YOUR_ADDRESS.rig1
```

## Want More Control?

Self-host your own TMPP node for zero-latency mining and complete sovereignty:
- [Self-Hosting Guide](SELF-HOSTING.md) — Why and how
- [Hardware Guide](HARDWARE-GUIDE.md) — What to buy, how to set up
- [Institutional Guide](INSTITUTIONAL.md) — For large mining operations
