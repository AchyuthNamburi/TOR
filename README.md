# Tor Network with Reinforcement Learning

> A simulated onion-routing network where a Q-Learning agent learns the best path through encrypted relay nodes — by trial and error.

---

## What This Project Does

Messages are wrapped in **3 layers of AES-128 encryption** (like an onion) and relayed through 3 anonymous nodes before reaching the destination. No single relay knows both who sent the message and where it's going.

On top of that, a **Q-Learning agent** at the sender learns — from 10,000 attempts — which combination of relay nodes delivers messages fastest and most reliably, without anyone telling it which nodes are good or bad.

**Result: success rate climbed from ~75% to ~90% on its own.**

---

## Architecture

```
Sender (RL Agent)
    │
    ▼
Layer 1 Node  (4 choices: NodeA / NodeB / NodeC / NodeD)
    │  decrypts its layer → sees only next hop
    ▼
Layer 2 Node  (4 choices)
    │  decrypts its layer → sees only next hop
    ▼
Layer 3 Node  (4 choices)
    │  decrypts its layer → sees only "Destination"
    ▼
Destination
```

4 × 4 × 4 = **64 possible routes**. The agent keeps a score for each and learns which ones to prefer.

---

## How the Learning Works

The agent uses **Q-Learning** — a simple table of scores, one per route.

After each attempt it calculates a reward:

| Outcome | Reward |
|---|---|
| Message delivered | `+10 − delay − latency` |
| Send failed | `−10 − delay − latency` |
| Packet dropped | `−15 − delay − latency` |

Then updates the score:

```
new_score = old_score + 0.1 × (reward − old_score)
```

Repeat 10,000 times → the scores become a reliable ranking of all 64 routes.

---

## Results

| Metric | Value |
|---|---|
| Episodes run | ~10,400 |
| Success rate (start) | ~75% |
| Success rate (end) | ~90% |
| Routes explored | 64 / 64 |
| Best learned route | `L1_NodeC → L2_NodeD → L3_NodeD` (Q = 6.81) |

Charts in [`Results/`](Results/) — learning curve, trust heatmaps, route frequency, Q-table heatmap.

---

## Project Structure

```
├── generate_keys.py     # Generate AES keys + addresses → keys.json
├── keys.json            # Config: keys, ports, node behaviour profiles
├── node.py              # Generic relay — decrypt, forward, update trust score
├── destination.py       # Final receiver — decrypts innermost layer
├── run_all_nodes.py     # Launcher — starts all 12 nodes + destination
├── sender.py            # RL agent — chooses route, builds onion, learns
├── plot_graphs.py       # Generates result charts from logs + Q-table
├── logs/                # performance_log.csv (per-episode results)
├── route_qtable.json    # Learned Q-values for all 64 routes
└── Trust scores/        # Per-node trust scores for downstream neighbours
```

---

## Quick Start

```bash
# 1. Install dependencies
pip install pycryptodome

# 2. Generate keys (only needed once)
python generate_keys.py

# 3. Start all relay nodes + destination
python run_all_nodes.py proc

# 4. In a new terminal — run the RL experiment
python sender.py

# 5. After the run — generate charts
python plot_graphs.py
```

---

## Tech Stack

- **Python 3** — sockets, threading, subprocess
- **PyCryptodome** — AES-128-CBC encryption
- **Q-Learning** — ε-greedy, α = 0.1, γ = 0.9, ε = 0.2
- **Pandas / Matplotlib / Seaborn** — logging and visualisation

---

## Key Concepts

**Onion Routing** — each relay decrypts only its own layer and learns only the next hop, so no single node can link sender to destination.

**Q-Learning** — the agent scores every route based on past rewards. Good routes float to the top; bad ones sink. No prior knowledge needed.

**Node Personalities** — each relay has a configurable drop probability, processing delay, and queue capacity, creating a realistic environment for the agent to learn from.
