# Grokking Lab

An interactive visualization of the **grokking phenomenon** — the counterintuitive behavior where a neural network first memorizes its training data, then, thousands of steps later, suddenly discovers a generalizing algorithm.

The training logs that power this visualization were produced by **[Grokking Simulation](https://www.kaggle.com/code/shingdev/grokking-simulation)** — a Kaggle notebook that trains a small transformer on modular arithmetic from scratch and records its behavior across 150,000 steps.

Based on [Power et al., 2022](https://arxiv.org/abs/2201.02177) and the mechanistic analysis by [Nanda et al., 2023](https://arxiv.org/abs/2301.05217).

---

## What is grokking?

A small transformer (~250K parameters) is trained to compute **(a + b) mod 97** for integers a, b ∈ [0, 96]. It's given 30% of all 9,409 possible pairs as training data.

Two things happen, separated by thousands of steps:

1. **Memorization** (~step 1,000) — the model achieves 100% train accuracy by brute-force memorizing all 2,822 training pairs. Validation accuracy stays near 0%.
2. **Grokking** (~step 7,200) — long after memorization, validation accuracy jumps suddenly from ~0% to 100%. The model has discovered the actual modular arithmetic algorithm.

The gap between these two events is the *grokking window*. Weight decay (L2 regularization = 1.0) is what forces the model to find a compressed, generalizing solution — it slowly kills the memorization circuit until a more efficient algorithmic representation takes over.

---

## Training notebook

> **[kaggle.com/code/shingdev/grokking-simulation](https://www.kaggle.com/code/shingdev/grokking-simulation)**
> Last updated: March 23, 2026

The notebook trains the model end-to-end and exports `grokking_logs.json`. Key setup:

| Hyperparameter | Value |
|---|---|
| Task | (a + b) mod 97, a, b ∈ [0, 96] |
| Train split | 30% of all pairs (~2,822 examples) |
| Val split | remaining 70% (~6,587 examples) |
| Total steps logged | 150,000 (checkpointed every 100 steps) |
| Weight decay | 1.0 (strong L2 — the key driver of grokking) |
| Model size | ~250K parameters |

The logs cover the full trajectory from random initialization through memorization, the dead zone, grokking, and long-run convergence. This visualization focuses on steps 0–20,000 where all the interesting dynamics occur.

---

## Project structure

```
grokking simulation/
├── index.html          # The full interactive visualization (self-contained)
├── grokking_logs.json  # Raw training logs — 1,500 checkpoints × 5 metrics
└── README.md           # This file
```

---

## Running it

Just open `index.html` in any modern browser. No server, no build step, no dependencies to install.

```bash
# macOS / Linux
open index.html

# Windows
start index.html
```

The only external requests are Google Fonts and Chart.js from cdnjs — both are cosmetic. The page works without internet (fonts fall back to system fonts; Chart.js is required for the chart).

---

## What's on the page

| Section | What it does |
|---|---|
| **Training run** | Animated Chart.js chart drawing train/val accuracy over steps 0–20,000. Three dashed annotation lines mark memorization, first breakthrough, and grokking. |
| **Explore the training** | Range scrubber that clips the chart in real time. Shows current phase (Memorizing / Dead zone / Transition / Grokked) with a one-line description. |
| **Auto-play** | Play/pause button with 0.5×–4× speed control. Loops the full training run as a ~18-second animation at 1×. |
| **Try it** | Enter any a, b ∈ [0, 96]. At step 1,000 the model only knows pairs it memorized. At step 10,000 it always gets the right answer. |
| **Key insights** | Three cards summarizing weight decay, model size, and training split. |

---

## Data

`grokking_logs.json` contains 1,500 checkpoints (steps 100–150,000) with five fields:

| Field | Description |
|---|---|
| `step` | Training step |
| `train_acc` | Fraction of training pairs correct (0–1) |
| `val_acc` | Fraction of unseen validation pairs correct (0–1) |
| `train_loss` | Cross-entropy loss on training set |
| `val_loss` | Cross-entropy loss on validation set |

The visualization embeds only the first 200 checkpoints (steps 100–20,000) directly in the HTML. Everything interesting happens before step 20,000 — beyond that, both metrics are flat at 100%.

---

## Technical notes

- **No frameworks.** Pure HTML + CSS + vanilla JS.
- **Chart.js 4.4.1** loaded from cdnjs for the line chart.
- **Vertical annotation lines** drawn via a custom Chart.js plugin registered inline — no `chartjs-plugin-annotation` dependency.
- **Training set simulation** uses a Mulberry32 seeded PRNG + Fisher-Yates shuffle (seed 42, 30% split) to reproduce which pairs the model saw, enabling the step-1,000 inference demo.
- **Auto-play** uses `requestAnimationFrame` with delta-time so playback speed is frame-rate independent.
- **Favicon** is an inline SVG data URI — no external file.

---

## References

- **Training notebook:** Singh, S. (2026). *Grokking Simulation.* Kaggle. https://www.kaggle.com/code/shingdev/grokking-simulation

- Power, A., Gur-Ari, Y., Garg, S., Sagun, L., & Katanforoosh, E. (2022). **Grokking: Generalization beyond overfitting on small algorithmic datasets.** *ICLR 2022 Workshop.* https://arxiv.org/abs/2201.02177

- Nanda, N., Chan, L., Lieberum, T., Smith, J., & Steinhardt, J. (2023). **Progress measures for grokking via mechanistic interpretability.** *ICLR 2023.* https://arxiv.org/abs/2301.05217
