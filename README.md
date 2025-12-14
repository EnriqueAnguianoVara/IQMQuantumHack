# Quantum Battlefield — QRW & HEQTO (Simulation + Real Hardware)

This repository contains the **final notebook used for our Quantum Battlefield project**, where two teams of units fight on an 8×8 grid using different movement policies:

- **QRW**: a *Quantum Random Walk* policy (2D walk with a 9-move coin).
- **HEQTO**: a shallow QAOA/optimizer-based policy for IQM Hardware efficiency.
- **Classical baselines**: random and greedy movement.

The notebook supports **simulation runs** (fast) and **real-hardware runs** (slow) via a Qiskit backend.

---

## Contents

- `Finaloso.ipynb` — **main deliverable** (all code + experiments).
- (Optional outputs created by the notebook)
  - `*.png` — transpiled circuit images (QRW/HEQTO hardware runs).

---

## What the notebook does

### 1) Battle engine (grid tactics)
- Grid-based battlefield (default **8×8**).
- Multiple unit types (e.g., soldier/knight/archer) with health, strength, and attack range.
- Turn-based loop:
  1. move units (policy-dependent),
  2. resolve combat,
  3. record history.

**No-stacking rule:** two allied units cannot occupy the same cell.

### 2) Scenarios
The notebook includes deterministic scenario builders (seeded) such as:
- **Front**: Quantum starts on the left, Classical on the right.
- **Quantum Surrounded**: Quantum in the center, Classical in a ring.
- **Classical Surrounded**: Classical in the center, Quantum in a ring.
- **Random**: both teams placed randomly (added in the final version).

### 3) Matchups
You can run:
- **QRW vs Classical** (random baseline),
- **HEQTO vs Classical**,
- **QRW vs HEQTO** (head-to-head).

The notebook supports animated turn-by-turn visualization (optional sleep between turns), plus aggregate benchmarking.

### 4) Real hardware (optional)
For a real backend (Qiskit `backend` object), the notebook can execute:
- **QRW on hardware** via `quantum_best_move_hw(...)` which:
  - builds the QRW circuit,
  - transpiles it for the backend,
  - runs it with `shots`,
  - reconstructs a probability distribution over (x,y),
  - samples the next move from that distribution.
- **HEQTO on hardware** via a hardware-aware policy wrapper (see HEQTO cell):
  - builds a QAOA-like circuit for tactical choice,
  - transpiles + runs on the backend,
  - decodes the most-likely bitstring into a move.

> Hardware runs are **not deterministic** (even with fixed seeds), and can be slow due to queue time.

### 5) Export battle traces to JSON
A dedicated cell exports a full battle trace to JSON so you can build a custom visualizer later.
The JSON typically includes:
- metadata (scenario, matchup, grid size, seed, shots),
- per turn: unit list with `(x, y)`, team, unit type, and health.

---

## Quickstart (simulation)

1. Create a fresh environment (recommended):
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # (Windows) .venv\Scripts\activate
   ```

2. Install dependencies:
   ```bash
   pip install numpy matplotlib pandas qiskit qiskit-aer
   ```

3. Open and run the notebook:
   ```bash
   jupyter notebook Finaloso.ipynb
   ```
   Run cells **top to bottom**.

**Simulation policies** are selected by strings like:
- `"QRW"` / `"HEQTO"` / `"CLASSIC_RANDOM"` / `"CLASSIC_GREEDY"`

---

## Quickstart (real hardware)

1. Make sure you can instantiate a real Qiskit backend object:
   - IBM Quantum, IQM, or any provider you used during the project.
   - The notebook expects a variable named `backend`.

2. In the “hardware battle” cell, set methods to:
- `"QRW_HW"` for QRW hardware policy
- `"HEQTO_HW"` for HEQTO hardware policy

3. Tune runtime parameters:
- `*_shots` (e.g., 256–2048)
- `*_opt_level` (e.g., 0–2)
- (optional) circuit image export: `*_save_png=True`

> Tip: start with **one scenario + one battle** (front only) to validate connectivity and performance.

---

## Reproducibility notes

- **Simulation** is seeded and should be repeatable if you keep:
  - the global seeds (`random`, `numpy`),
  - the scenario seed,
  - and the same notebook order of execution.
- **Hardware** is inherently noisy / non-deterministic.
  Seeds only affect *classical post-processing* (e.g., sampling from estimated probabilities).

---

## Project configuration knobs (most relevant)

You will find these variables near the top of the notebook:
- `W, H` — grid size
- `MAX_TURNS` — turn limit (draw if exceeded)
- `FULL_COMP` — team composition (counts + stats)
- `SHOW_EACH_TURN`, `SLEEP` — animation control
- `K`, `softmax_temp` — tactical sampling options (simulation)

For hardware-specific runs:
- `qrw_hw_backend`, `qrw_hw_shots`, `qrw_hw_opt_level`
- `heqto_hw_backend`, `heqto_hw_shots`, `heqto_hw_opt_level`

---

## Deliverable checklist

- Simulation matchups (QRW / HEQTO / baselines)
- Random scenario added alongside the three fixed scenarios
- Hardware-ready QRW policy with circuit export
- Hardware-ready HEQTO policy wrapper
- JSON export of battle traces for external visualization
- Scalability analysis cell (QRW + HEQTO, see notebook)

---


## Troubleshooting

- **`NameError: backend is not defined`**
  - You are running a hardware cell without having initialized a backend object.
- **Long hardware execution time**
  - Reduce `shots`, run a single battle, and expect queue delays.
- **Bitstring order issues**
  - Some backends require reversing bitstrings before decoding; the notebook exposes a `reverse_bits` flag where relevant.
