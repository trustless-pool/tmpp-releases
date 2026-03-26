# Getting Started with TMPP

## Self-Host (Recommended)

Kaspa has 1-second blocks. Self-hosting drops your stale rate from 3-10% to <0.1%.

### Step 1: Download and Run

**Windows:**
```
Download tmpp-node.exe from GitHub Releases
Double-click to run, or from PowerShell:
.\tmpp-node.exe
```

**Linux:**
```bash
curl -L https://github.com/trustless-pool/tmpp-releases/releases/latest/download/tmpp-node-linux-x86_64 -o tmpp-node
chmod +x tmpp-node
./tmpp-node
```

First run walks you through setup — installs kaspad, syncs the chain, and starts the pool. Every run after that, it just starts.

### Step 2: Point Your Miner

Once TMPP shows "Kaspa Stratum server listening on port 3337":

```
Pool URL:   stratum+tcp://127.0.0.1:3337
Worker:     YOUR_KASPA_ADDRESS.rig_name
Password:   x
```

Replace `YOUR_KASPA_ADDRESS` with your Kaspa wallet address (starts with `kaspa:q...`). Replace `rig_name` with any label for your device (e.g., `ks5-01`, `iceriver-1`).

Compatible miners: IceRiver KS series, Bitmain KS5 Pro, BzMiner, lolMiner, GMiner.

### Step 3: Check the Dashboard

Open `http://localhost:8080` in your browser.

- Your hashrate and worker status
- Shares submitted and earnings
- Block found history
- Payout verification with Merkle proofs

### Step 4: Get Paid

Payouts happen automatically when your balance reaches the threshold:

| Chain | Default Threshold |
|---|---|
| Kaspa | 100 KAS |

You can adjust your payout threshold in the dashboard under **My Miner**.

That's it. No registration. No email. No KYC. Your address is your account.

## Verify Your Payouts

This is what makes TMPP different. Every block reward distribution is Merkle-committed on-chain.

1. Open the dashboard -> My Miner
2. Click **Verify Payout** next to any block
3. The dashboard rebuilds the Merkle tree and checks it against the on-chain commitment
4. If it matches, your payout was computed correctly

No other pool lets you prove your payouts are correct.

## Why TMPP?

### Lower Fees
0.5% pool fee. Most Kaspa pools charge 0.9-1%. You keep more of what you mine.

### Zero Stale Shares
Self-hosting means <1ms latency. On Kaspa's 1-second blocks, that eliminates stale shares entirely. Centralized pools waste 3-10% of your hashrate on stales.

### Trustless Verification
Every payout is Merkle-committed in the blockchain. Click **Verify Payout** in the dashboard to cryptographically prove your earnings.

### No Account Required
Your payout address IS your account. No signup forms, no email verification, no passwords.

## FAQ

**Q: How do I know TMPP is paying me fairly?**
A: Every block reward distribution is cryptographically committed in the coinbase. Go to My Miner -> Verify Payout to see the Merkle proof.

**Q: What if my node goes down?**
A: The share WAL replays on restart. Zero shares lost, zero credits lost. Your miners reconnect automatically.

**Q: Can I run TMPP on the same machine as my miner?**
A: Yes. TMPP uses <5% CPU and <500MB RAM. Point your miner at `localhost:3337`.

**Q: Is there a minimum hashrate?**
A: No. Any hashrate works. ASIC or GPU.

## Need Help?

- GitHub: [github.com/trustless-pool/tmpp-releases](https://github.com/trustless-pool/tmpp-releases)

## Want More Control?

- [Self-Hosting Guide](SELF-HOSTING.md) -- why local nodes win on Kaspa
- [Hardware Guide](HARDWARE-GUIDE.md) -- what to buy, how to set up
- [Institutional Guide](INSTITUTIONAL.md) -- for large mining operations
