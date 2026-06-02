Here's the updated guide with all "Say" sections converted to explanations:

---

# 🎬 Pollinet SDK — Complete Testing Guide

**A step-by-step guide to testing the Pollinet SDK — offline Solana transactions via BLE mesh relay network.**

---

## Prerequisites
- Windows PC with WSL2 enabled
- Ubuntu installed via WSL2
- Internet connection
- A GitHub account

---

## PART 1 — Setting Up Ubuntu

Ubuntu is our working environment. We start by making sure everything is fresh and up to date.

**Step 1 — Update Ubuntu:**
```bash
sudo apt update && sudo apt upgrade -y
```

**Step 2 — Install all dependencies:**
```bash
sudo apt install -y curl git build-essential pkg-config libssl-dev bzip2 python3
```
This installs all the build tools that Rust and Solana need — including python3 which we use later to configure our wallet.

---

## PART 2 — Install Rust

Pollinet's SDK is written in Rust — one of the fastest and most memory-safe programming languages out there. Rust powers the offline transaction engine under the hood.

**Step 1 — Install Rust:**
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
When prompted, press `1` then Enter for the default install. Wait for it to finish completely.

**Step 2 — Load Rust into your session:**
```bash
source $HOME/.cargo/env
```

**Step 3 — Verify Rust is installed:**
```bash
rustc --version
```
```bash
cargo --version
```
Both commands should print version numbers confirming Rust is ready.

---

## PART 3 — Install Solana CLI

The Solana CLI is our direct line to the Solana blockchain. We use it to create wallets, check balances, and interact with our local test network.

**Step 1 — Download in your Windows browser:**

Open this link in Chrome or Edge and wait for the full download (~120MB):
👉 `https://github.com/solana-labs/solana/releases/download/v1.18.26/solana-release-x86_64-unknown-linux-gnu.tar.bz2`

**Step 2 — Copy it into Ubuntu:**
```bash
cp "/mnt/c/Users/NEW USER/Downloads/solana-release-x86_64-unknown-linux-gnu.tar.bz2" ~/solana.tar.bz2
```
> ⚠️ Replace `NEW USER` with your actual Windows username. Run `ls /mnt/c/Users/` to find it.

**Step 3 — Extract it:**
```bash
tar -xf ~/solana.tar.bz2
```

**Step 4 — Add to PATH permanently:**
```bash
export PATH="$HOME/solana-release/bin:$PATH"
```
```bash
echo 'export PATH="$HOME/solana-release/bin:$PATH"' >> ~/.bashrc
```
```bash
source ~/.bashrc
```

**Step 5 — Verify Solana CLI is installed:**
```bash
solana --version
```
You should see something like `solana-cli 1.18.26`.

---

## PART 4 — Clone the Pollinet Repo

Now we download Pollinet's open-source SDK from GitHub. This is the actual code that powers the offline transaction and BLE mesh relay system.

**Step 1 — Clone:**
```bash
git clone https://github.com/pollinet/pollinet.git
```

**Step 2 — Enter the folder:**
```bash
cd pollinet
```

**Step 3 — See what's inside:**
```bash
ls
```
You'll see:
- `pollinet-sdk` — the core offline transaction engine
- `examples` — individual demo scripts
- `scripts` — the automated test runner
- `src` — the SDK source code

---

## PART 5 — Start the Local Validator ⚠️ Do This Before Anything Else

Instead of using Solana's public devnet — which has airdrop rate limits — we spin up a full Solana blockchain locally on our machine. This gives us unlimited test SOL and no restrictions.

**Step 1 — Open a SECOND Ubuntu window and run:**
```bash
cd pollinet
solana-test-validator --reset
```
This starts a full local Solana blockchain. Leave this window running for the entire session — never close it. You'll see slot numbers ticking up confirming blocks are being produced.

**Step 2 — Wait 20 seconds until you see slot numbers scrolling:**
```
Processed Slot: 10
Processed Slot: 11
Processed Slot: 12
```

**Step 3 — Back in your FIRST Ubuntu window, navigate to the pollinet folder:**
```bash
cd pollinet
```

**Step 4 — Point Solana CLI to your local validator:**
```bash
solana config set --url localhost
```

**Step 5 — Verify the validator is reachable:**
```bash
solana cluster-version
```
If this returns a version number, your local chain is healthy and ready.

---

## PART 6 — Set Up Your Wallet

We need a wallet to sign our offline transactions. This is the cryptographic identity that interacts with the Solana blockchain.

**Step 1 — Generate a new wallet:**
```bash
solana-keygen new --no-bip39-passphrase
```
> ⚠️ If you see "Refusing to overwrite" — you already have a wallet from a previous session. Skip this step.

**Step 2 — Get your wallet address:**
```bash
solana address
```
The long string printed is your public wallet address — your identity on the Solana network.

**Step 3 — Airdrop free test SOL:**
```bash
solana airdrop 10
```
Because we're on our local validator there are no rate limits — we can airdrop as much test SOL as we need instantly. No real money involved.

**Step 4 — Confirm balance:**
```bash
solana balance
```
Should show `10 SOL`.

---

## PART 7 — Configure Wallet in .env ⚠️ Critical Step

This step tells the Pollinet SDK to use your wallet for every single example and test. Skipping this causes wallet mismatch errors across all examples. We use a built-in python3 script — no external libraries needed.

**Step 1 — Convert your wallet keypair to base58 format:**
```bash
PRIVKEY=$(python3 -c "
import json
key = json.load(open('/home/alioth/.config/solana/id.json'))
b = bytes(key)
alphabet = b'123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz'
n = int.from_bytes(b, 'big')
result = []
while n:
    n, r = divmod(n, 58)
    result.append(alphabet[r:r+1])
result = b''.join(reversed(result))
print(result.decode())
")
```
> ⚠️ Replace `alioth` with your actual Ubuntu username if different.

**Step 2 — Verify the key was captured:**
```bash
echo $PRIVKEY
```
You should see a long base58 string printed out.

**Step 3 — Create the .env file with your wallet baked in:**
```bash
echo "SOLANA_URL=http://localhost:8899" > .env
```
```bash
echo "WALLET_PRIVATE_KEY=$PRIVKEY" >> .env
```

**Step 4 — Verify it looks correct:**
```bash
cat .env
```
You should see:
```
SOLANA_URL=http://localhost:8899
WALLET_PRIVATE_KEY=<your key here>
```
Every example and test script will now use this same wallet — no conflicts, no mismatches.

---

## PART 8 — Build the Project

Now we compile the entire Pollinet SDK. Rust compiles everything down to super-fast native code — this is part of why Pollinet can run efficiently even on low-power devices acting as BLE relay nodes.

**Step 1 — Make sure you're in the pollinet folder:**
```bash
cd pollinet
```

**Step 2 — Build:**
```bash
cargo build --all-targets --examples
```
This takes a few minutes the first time as Rust downloads and compiles all dependencies. After the first build everything runs much faster.

Wait for:
```
Finished dev [unoptimized + debuginfo] target(s)
```
This confirms everything compiled successfully.

---

## PART 9 — Clear Any Old Data

Before running any tests, we remove leftover data from previous sessions to ensure a clean run.

```bash
rm -f .offline_bundle.json
```
This removes any old nonce bundle files that could cause wallet mismatch or blockhash errors.

---

## PART 10 — Run the Quick Test Suite

This runs Pollinet's full automated test pipeline — from creating nonce accounts, signing transactions offline, fragmenting them into BLE-sized packets, simulating the mesh relay, all the way to confirming on chain.

```bash
./scripts/test_pollinet.sh --quick
```

What happens during this test:
- ✅ Prerequisites are checked
- ✅ Build is verified
- ✅ Nonce bundle is created with fresh accounts
- ✅ Transactions are signed offline using durable nonces
- ✅ Transactions are compressed and fragmented into BLE packet sizes
- ✅ Mesh relay is simulated
- ✅ Transactions are submitted and confirmed on the local chain

A green checkmark next to each step means that part of the pipeline succeeded.

---

## PART 11 — The M1 Demo

This is Pollinet's M1 milestone — 50 offline transactions processed end to end. This benchmark proves the system works at real scale.

```bash
./scripts/test_pollinet.sh --m1-only
```

What happens during this demo:
1. 50 nonce accounts are created on the local chain
2. 50 offline transactions are signed locally — no internet needed
3. A 5-minute offline period is simulated
4. All 50 transactions are submitted once connection is restored
5. Every transaction is confirmed on chain

This takes approximately 5-10 minutes to complete. This is exactly what would happen in a real-world scenario — a disaster zone, a censored region, or anywhere with unstable internet. Transactions hop peer to peer over Bluetooth via Pollistem nodes and eventually land on Solana.

---

## PART 12 — View the Results

Once the M1 demo completes, view the full test results:

```bash
ls test_results/
```
```bash
cat test_results/*/summary.md
```
The summary shows:
- Total tests passed
- Total tests failed
- Log files for every step
- Transaction signatures saved to `.offline_submission.json`

All tests passing confirms that 50 transactions were created offline, relayed through the simulated BLE mesh, and confirmed on the local Solana chain.

---

## What is Pollinet?

Pollinet is the resilience layer for Solana — keeping transactions alive via BLE mesh relay even when the internet goes down.

**Key components:**

**Offline transactions via durable nonces** — transactions are pre-signed locally using special nonce accounts that never expire, so they stay valid until a connection is restored.

**BLE mesh relay network** — signed transactions hop device to device over Bluetooth Low Energy until they reach a node with internet access and get broadcast to Solana.

**Pollistem nodes** — dedicated always-on relay devices that extend BLE mesh coverage and earn $POLLEN tokens for their uptime and relay activity.

**Solana resilience infrastructure** — a DePIN layer that means Solana keeps working during outages, in censored regions, disaster zones, and off-grid environments.

---

## Useful Links
- 🐦 Twitter: [@sol_pollinet](https://x.com/sol_pollinet)
- 🌐 Website: [pollinet.xyz](https://pollinet.xyz)
- 📦 SDK: [github.com/pollinet/pollinet](https://github.com/pollinet/pollinet)

---

## ⚠️ Troubleshooting

**"Refusing to overwrite" on keygen** — you already have a wallet, skip that step.

**Blockhash not found error** — wait 30 seconds and try again. The validator needs time to warm up.

**Wallet mismatch / InvalidPublicKey error** — your .env is missing or incorrect. Redo Part 7 and delete the old bundle with `rm -f .offline_bundle.json`.

**Connection refused on solana balance** — your local validator isn't running. Go to your second window and start it with `solana-test-validator --reset`.

**cargo: command not found** — run `source $HOME/.cargo/env` to reload Rust into your session.

**Cargo.toml not found** — you're not in the pollinet folder. Run `cd pollinet` first.

**DNS / network errors in Ubuntu** — run these to fix:
```bash
sudo rm -f /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "nameserver 8.8.4.4" >> /etc/resolv.conf'
```

---

## 📌 Quick Command Reference

| Step | Command | Purpose |
|---|---|---|
| 1 | `sudo apt update` | Fresh environment |
| 1 | `sudo apt install -y ... python3` | All dependencies |
| 2 | `rustc --version` | Rust installed |
| 3 | `solana --version` | Solana CLI ready |
| 4 | `git clone` | SDK downloaded |
| 5 | `solana-test-validator --reset` | Local chain running |
| 5 | `cd pollinet` | Inside project folder |
| 5 | `solana config set --url localhost` | Pointed to local |
| 6 | `solana airdrop 10` | Funded instantly |
| 7 | `python3 base58 conversion` | Wallet key extracted |
| 7 | `cat .env` | Wallet locked in |
| 8 | `cargo build --all-targets` | SDK compiled |
| 9 | `rm -f .offline_bundle.json` | Old data cleared |
| 10 | `test_pollinet.sh --quick` | Full pipeline tested |
| 11 | `test_pollinet.sh --m1-only` | 50 txns at scale |
| 12 | `cat test_results/*/summary.md` | Results confirmed |

---
