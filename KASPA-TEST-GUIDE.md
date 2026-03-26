# TMPP Kaspa RC1 — Two-Machine Test Guide (Windows)

## What You're Testing

Two 5090 GPUs mining Kaspa through TMPP, connected via gossip. One pool, two nodes, two miners, zero centralized infrastructure.

```
Machine 1 (You)                     Machine 2 (Your Son)
┌──────────────────────┐           ┌──────────────────────┐
│  BzMiner (5090)      │           │  BzMiner (5090)      │
│       ↓ Stratum      │           │       ↓ Stratum      │
│  TMPP Node           │◄─gossip──►│  TMPP Node           │
│       ↓ gRPC         │           │       ↓ gRPC         │
│  kaspad (mainnet)    │           │  kaspad (mainnet)    │
└──────────────────────┘           └──────────────────────┘
```

## Prerequisites

- Windows 10/11
- NVIDIA 5090 with latest drivers
- 50 GB free disk space (for Kaspa blockchain)
- Both machines on the same network (or port 9940 forwarded)

---

## Step 1: Install kaspad (both machines)

Download Kaspa node:

1. Go to https://github.com/nicknish/rusty-kaspa/releases
2. Download `rusty-kaspa-vX.X.X-windows-amd64.zip`
3. Extract to `C:\kaspa\`
4. Open **PowerShell** and run:

```powershell
cd C:\kaspa
.\kaspad.exe --utxoindex --rpclisten=0.0.0.0:16110 --rpclisten-json=0.0.0.0:18110
```

This will start syncing the Kaspa blockchain. **First sync takes 2-4 hours.** Leave it running.

You'll know it's synced when the log shows blocks at the current DAA score and stops rapidly advancing.

---

## Step 2: Install TMPP (both machines)

1. Download `tmpp-node.exe` from the releases (or build from source)
2. Place in `C:\tmpp\`
3. Open a **new PowerShell window** and run:

**Machine 1 (you):**
```powershell
cd C:\tmpp
.\tmpp-node.exe --chain=kaspa --bitcoind-url=ws://127.0.0.1:18110 --payout-address=kaspa:YOUR_KASPA_ADDRESS --dashboard-port=8080
```

**Machine 2 (your son):**
```powershell
cd C:\tmpp
.\tmpp-node.exe --chain=kaspa --bitcoind-url=ws://127.0.0.1:18110 --payout-address=kaspa:YOUR_SONS_KASPA_ADDRESS --gossip-peers=MACHINE1_IP:9940 --dashboard-port=8080
```

Replace:
- `kaspa:YOUR_KASPA_ADDRESS` — your Kaspa wallet address
- `kaspa:YOUR_SONS_KASPA_ADDRESS` — his Kaspa wallet address
- `MACHINE1_IP` — your machine's local IP (e.g., `192.168.1.100`)

To find your local IP, run in PowerShell:
```powershell
ipconfig | findstr "IPv4"
```

You should see:
```
Kaspa Stratum server listening on port 3337
Kaspa mining pool ready — miners connect to stratum+tcp://0.0.0.0:3337
Dashboard: http://0.0.0.0:8080
```

---

## Step 3: Install BzMiner (both machines)

1. Go to https://github.com/bzminer/bzminer/releases
2. Download `bzminer_vXX_windows.zip`
3. Extract to `C:\bzminer\`
4. Open the file `C:\bzminer\config.txt` in Notepad
5. Replace the contents with:

```json
[
  {
    "algorithm": "kaspa",
    "wallet": "kaspa:YOUR_ADDRESS.rig1",
    "pool": "stratum+tcp://127.0.0.1:3337",
    "password": "x"
  }
]
```

Replace `kaspa:YOUR_ADDRESS` with each machine's own Kaspa address. The `.rig1` suffix is the worker name — use `.dad-5090` and `.son-5090` to tell them apart on the dashboard.

6. Open a **new PowerShell window** and run:

```powershell
cd C:\bzminer
.\bzminer.exe
```

You should see hashrate output within seconds:
```
KAS: 1.45 GH/s | Shares: 5/5 | Pool: stratum+tcp://127.0.0.1:3337
```

---

## Step 4: Verify Everything Works

### Check the dashboard

Open a browser on either machine:
```
http://localhost:8080
```

You should see:
- **Chain: Kaspa (KAS)** with teal color
- **Active Workers:** 1 (or 2 if gossip is connected)
- **Shares** counter incrementing
- **Hashrate** showing your 5090's speed

### Check gossip connection

On Machine 2's TMPP output, look for:
```
Gossip: connected to MACHINE1_IP:9940
Gossip: peer handshake complete
```

### Check both miners on the dashboard

Go to **My Miner** tab, enter your Kaspa address. You should see:
- Your shares, your hashrate, your balance accumulating

On Machine 1's dashboard, your son's shares should also appear (via gossip).

---

## Step 5: Watch for Blocks

With two 5090s at ~1.5 GH/s each (3 GH/s total), you'll need real Kaspa network luck to find a block. On mainnet this could take hours or days depending on network hashrate.

When a block IS found:
- The TMPP node logs: `!!! KASPA BLOCK FOUND !!!`
- The dashboard shows confetti
- Credits are distributed proportionally from the PPLNS window
- Both miners get credited based on their share count

---

## Troubleshooting

### "Connection refused" on Stratum

TMPP hasn't started yet, or kaspad isn't synced. Wait for kaspad to sync and TMPP to show "Stratum server listening."

### "Job not found" errors in BzMiner

TMPP is still fetching templates from kaspad. Wait 10 seconds after TMPP starts before launching BzMiner.

### Gossip not connecting

Check that port 9940 is open between the machines:
```powershell
# On Machine 2, test connection to Machine 1:
Test-NetConnection -ComputerName MACHINE1_IP -Port 9940
```

If it fails, check Windows Firewall. Allow inbound TCP 9940 on Machine 1:
```powershell
# Run as Administrator on Machine 1:
New-NetFirewallRule -DisplayName "TMPP Gossip" -Direction Inbound -Protocol TCP -LocalPort 9940 -Action Allow
```

### BzMiner shows 0 hashrate

Make sure you have the latest NVIDIA drivers for the 5090. BzMiner needs driver support for the GPU architecture.

### kaspad won't sync

Make sure port 16111 (Kaspa P2P) is not blocked by your firewall. kaspad needs outbound access to the Kaspa network.

---

## What Success Looks Like

After 5 minutes of mining:

1. Both BzMiner instances show ~1.5 GH/s and shares being accepted
2. Both dashboards show shares in the PPLNS window
3. Gossip is connected (Machine 2 sees Machine 1 as a peer)
4. If a block is found, both miners get credited proportionally

You're running a decentralized, trustless mining pool from two gaming PCs. No registration, no third-party server, no trust required. Every share is PoW-verified, every payout will be Merkle-committed on-chain.

---

## Stopping

1. Close BzMiner (Ctrl+C in its PowerShell window)
2. Close TMPP (Ctrl+C in its PowerShell window)
3. Close kaspad (Ctrl+C in its PowerShell window)

Your earned credits are saved in `C:\tmpp\data\` and will be there when you restart.
