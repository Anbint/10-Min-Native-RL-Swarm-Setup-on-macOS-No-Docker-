# 10-Min-Native-RL-Swarm-Setup-on-macOS-No-Docker

> Run a Gensyn RL-Swarm node **natively on macOS** (no Docker), using only Python + a virtualenv + tmux.  
> Tested on Apple Silicon (M1/M2/M3) and Intel Macs.

## What “native on macOS” actually means:

This guide:
- Does not use Docker or any Linux VM
- Uses:
  + macOS Python (python3)
  + local virtualenv (.venv)
  + tmux to keep the process running in the background

Pros:
- Lighter overhead than Docker / full VM
- Simpler debugging (everything is just a normal macOS process)
- Easy to integrate with your existing dev setup

Cons:
- Your node stops if your Mac sleeps / shuts down
- You are sharing CPU/RAM with your desktop workload
- Slightly more sensitive to local OS/CPU quirks than an Ubuntu VPS

---

## 0. Requirements

**OS**

- macOS 13+ (Ventura, Sonoma or newer recommended)

**Hardware (minimum)**

- CPU: 4 cores (Apple Silicon or Intel)
- RAM: 16 GB recommended if you multitask heavily
- Disk: 20+ GB free SSD
- Network: stable connection, no super-aggressive firewall / VPN blocking P2P

**You should be comfortable with:**

- Opening **Terminal**
- Copy–pasting commands
- Letting a tmux session run in the background

This guide is **native only**.  
## If you prefer containers, use the official **Docker** guide instead (https://docs.gensyn.ai/testnet/rl-swarm/getting-started/macos)

---

## 1. Quickstart (for power users)

If you already have Homebrew, Python, Node, etc., you can basically do:

```bash
# 1) Clone
git clone https://github.com/gensyn-ai/rl-swarm.git
cd rl-swarm

# 2) Create venv + install uv
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip uv

# 3) First run (creates wallet + opens localhost:3000)
bash run_rl_swarm.sh
# → login in browser, answer prompts, confirm node runs

# 4) Long-running node in tmux
tmux new -s swarm
cd ~/rl-swarm
source .venv/bin/activate
bash run_rl_swarm.sh
# detach with: Ctrl+B, then D
```

For a more guided, step-by-step version, keep reading 👇

⸻

## 2. Prepare your macOS environment

### 2.1. Install Xcode Command Line Tools (recommended)

These provide compilers and basic build tools:

```
xcode-select --install
```

If you see “command line tools are already installed”, you’re good.

⸻

### 2.2. Install Homebrew (if you don’t have it)

Check first:

```
brew --version
```

If it prints a version, skip this step.
If not, install Homebrew:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then follow the on-screen instructions (you may need to add eval "$(/opt/homebrew/bin/brew shellenv)" to your shell profile on Apple Silicon).

⸻

### 2.3. Install core tools (git, Python, tmux, optional Node)

Most Macs already have git and python3, but we’ll ensure they’re available and modern enough:

```bash
brew install git python tmux
```

- Optional but useful: Node.js (for the local web UI).
- RL-Swarm will start a local web UI on http://localhost:3000.
- If you ever see logs complaining about missing node or yarn, install Node:

```bash
brew install node
# (optional) enable corepack for yarn / pnpm
corepack enable
```

For many users, RL-Swarm will “just work” without extra Node/Yarn setup.

⸻

## 3. Clone the RL-Swarm repository

In Terminal:

```bash
cd ~
git clone https://github.com/gensyn-ai/rl-swarm.git
cd rl-swarm
```

This creates ~/rl-swarm with the latest code.

⸻

4. Create a Python virtual environment

We’ll isolate RL-Swarm’s Python dependencies inside .venv.

```bash
cd ~/rl-swarm

python3 -m venv .venv
source .venv/bin/activate

pip install -U pip uv
```

- .venv – your local Python environment for RL-Swarm
- uv – fast dependency resolver/runner used by the project

🔁 Later, whenever you open a new Terminal and want to interact with RL-Swarm, run:

```
cd ~/rl-swarm
source .venv/bin/activate
```

⸻

## 5. First run: create wallet & connect to Gensyn Testnet

This first run will:
- Install any needed Python/JS deps
- Start a local UI on http://localhost:3000
- Let you log into the Gensyn Testnet
- Confirm that your node can join the swarm

From inside ~/rl-swarm with the venv active:

```bash
cd ~/rl-swarm
source .venv/bin/activate
bash run_rl_swarm.sh
```

You should see something like:

```bash
>>> RL-SWARM starting...
Building server
Started server process: 12345
>> Failed to open http://localhost:3000. Please open it manually.
>> Waiting for modal userData.json to be created...
```

This means the local wallet UI is running on port 3000.

⸻

### 5.1. Log in via browser

On the same Mac:
1. Open your browser (Safari / Chrome / Arc / etc.)
2. Go to:

```
http://localhost:3000
```

3. You should see a “Welcome to the Gensyn Testnet” page.
4. Log in / connect your wallet as instructed.

Once you see a success message (e.g. “You’re connected to Gensyn Testnet”), you can close that tab.

⸻

### 5.2. Answer CLI prompts

Back in the Terminal where bash run_rl_swarm.sh is running, the script will continue and ask:

```
Would you like to push models you train in the RL swarm to the Hugging Face Hub? [y/N]
```
Type n and press Enter (unless you specifically want Hugging Face integration).

Then:

```
Enter the name of the model you want to use in huggingface repo/name format,
or press [Enter] to use the default model.
```
Just press Enter to use the default config (for example Qwen/Qwen2.5-Coder-0.5B-Instruct).

If everything is OK, you’ll eventually see logs similar to:

```bash
✅ Connected to Gensyn Testnet
👋 Hello [your-node-name]!
🐝 Joining round: 12345 / 1000000
```

The bracketed nickname (e.g. raging rabid pigeon) is your Swarm ID.

At this point, your first native run succeeded.

Press Ctrl+C to stop it and set up a long-running session.

⸻

## 6. Run RL-Swarm long-term with tmux

On macOS, closing the Terminal window will normally kill your process.

tmux lets RL-Swarm keep running even if you close the Terminal app (as long as your Mac itself stays on / awake).

### 6.1. Start a tmux session

```bash
cd ~/rl-swarm
tmux new -s swarm
```

You’re now “inside” tmux, in a session named swarm.

### 6.2. Activate venv & start the node

Inside tmux:

```bash
cd ~/rl-swarm
source .venv/bin/activate
bash run_rl_swarm.sh
```

Let it run. You should see rounds joining and training logs scrolling.

### 6.3. Detach from tmux (keep node running)

To detach and leave the node running:

	•	Press Ctrl+B, then D

You’ll be back to a normal shell, but the RL-Swarm node continues in the background.

### 6.4. Re-attach later

List sessions:
```
tmux ls
```

Re-attach:
```
tmux attach -t swarm
```

Stop the node (from inside tmux):

	•	Press Ctrl+C in the RL-Swarm terminal

Then you can exit tmux completely:
```
exit
```

⸻

## 7. Basic health checks

Once your node has been running for a bit:
1. Go to the official RL-Swarm dashboard in your browser
   (with the same wallet you used when you logged in at localhost:3000).
3. You should see your node listed under something like “YOUR NODES”:
- A node name matching the CLI nickname (e.g. raging rabid pigeon).
- Participation and Training Rewards counters that update roughly every 3 hours.

Additionally, you can:
- Look at your terminal logs in tmux and confirm:
- Rounds are joining (Joining round: ...)
- No constant fatal errors
- Check that your Mac isn’t completely overloaded:
- Use Activity Monitor → CPU / Memory tabs

<img width="663" height="108" alt="Screenshot 2025-12-06 at 19 46 41" src="https://github.com/user-attachments/assets/c2c71656-3c80-4ac8-8da2-97aa5beeea79" />
<img width="785" height="111" alt="Screenshot 2025-12-06 at 19 46 56" src="https://github.com/user-attachments/assets/93d713c0-e098-42c2-b186-180dd38bd9cb" />

If Participation and Rewards keep going up over time, your node is doing work.

⸻

## 8. When your macOS node looks “stuck”

If you see no progress (rounds not advancing, or dashboard not updating) after a few hours, try these steps in order.

⸻

### 8.1. Option 1 – Simple restart (recommended first)
```bash
# 1) Open Terminal and re-attach tmux
tmux attach -t swarm  # or: tmux ls to find the name

# 2) Stop RL-Swarm inside tmux
#    (Ctrl+C in the RL-Swarm terminal)

# 3) Exit tmux session (optional)
exit

# 4) Start a fresh tmux session
cd ~/rl-swarm
tmux new -s swarm

# 5) Inside tmux: activate venv & run again
cd ~/rl-swarm
source .venv/bin/activate
bash run_rl_swarm.sh
```

Let it run for at least one full round and check logs + dashboard.

⸻

### 8.2. Option 2 – Update to latest rl-swarm

If restart doesn’t help, update your local repo.
```bash
cd ~/rl-swarm

# (re-attach tmux and stop the node if it's running)
tmux attach -t swarm  # if needed
# Ctrl+C inside RL-Swarm
exit                  # leave tmux session if you want

# Pull latest code
git fetch origin
git pull origin main

# Rebuild venv (optional but clean)
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip uv

# Start again in a fresh tmux
tmux new -s swarm
cd ~/rl-swarm
source .venv/bin/activate
bash run_rl_swarm.sh
```

⸻

### 8.3. Option 3 – Reset node identity (swarm.pem)

Use this only if options 1 & 2 don’t fix your node.
This will generate a new PeerID and node identity.

#### 1) Backup your current identity
```bash
cd ~/rl-swarm
mkdir -p swarm-backups
cp swarm.pem swarm-backups/swarm.pem.$(date +%Y%m%d-%H%M%S) || echo "No swarm.pem to backup"
```

#### 2) Remove existing identity and restart
```bash
# Stop any running node (inside tmux)
tmux attach -t swarm  # if needed
# Ctrl+C inside RL-Swarm
exit                  # leave tmux

cd ~/rl-swarm
rm -f swarm.pem

tmux new -s swarm
cd ~/rl-swarm
source .venv/bin/activate
bash run_rl_swarm.sh
```

On the next run, RL-Swarm will create a fresh swarm.pem and a new node name will appear in the logs.

⸻

Happy swarming 🐝
If you find this helpful, feel free to star the repo or share the guide with other macOS node runners.
