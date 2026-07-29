---
title: "How to Build a Robot Foundation Model — The Talk, One Screen Per Slide"
date: 2026-07-29
draft: false
summary: "The condensed edition: the same argument at one screen per slide, for reading in fifteen minutes instead of fifty."
---

## How to Build a Robot Foundation Model

### and why the hard parts are yours

An opener for people who know robots, and do not know language models.

---

## Two numbers

**85.6% progress / 80% success** — a shirt folded on a bimanual UR5e that saw **zero** laundry data [arXiv:2604.15483 §IX].

**90.9% / 80.6%** — ten expert teleoperators, ~375 h each, first attempt on that arm [arXiv:2604.15483 §IX].

---

## The second number

That model runs on an **H100** [arXiv:2604.15483 App. D].

Move the camera **10 cm** and **20°**: **100% → 0%** [arXiv:2409.03403].

> Generality is real. Its fragility is mechanical.

---

## What it actually is

![](figures/f01-io-contract.png)

**In:** ≤4 images, joint configuration, one sentence. **Out:** H=50 joint targets; execute 15 or 25; replan [arXiv:2604.15483 §IV].

It is a **setpoint generator**. It does not replace your servo loop [arXiv:2604.15483 §VII].

---

## Foundation model, or just a big policy?

- New task = new **sentence**, not a new dataset [arXiv:2604.15483 §V].
- Same weights, different kinematics [arXiv:2510.03342].
- Capabilities the robot data never contained [arXiv:2310.08864 Table II].
- Adding robot B's data improves robot A [arXiv:2310.08864].

---

## The decisive ablation

Strip web pretraining from the same architecture:

| | emergent skills | generalization |
|---|---|---|
| with web pretraining | 48.7% | 47% |
| without | **0%** | **1%** |

The semantics are inherited, not learned from robot data [arXiv:2310.08864 Table II].

---

## Which generalization is actually demonstrated

- objects, scenes, instructions, ~10-min composition — **yes** [arXiv:2504.16054].
- cross-embodiment — yes, with a ceiling: 75.8% vs 27.3% [arXiv:2310.08864].
- **friction, payload, inertia, compliance — no lab publishes a sweep** [arXiv:2604.15483].

---

## Before I sell you anything

Independent work finds lexical-kinematic shortcuts and semantic feature collapse [arXiv:2604.18000].

The leading lab concedes it may be **"remixing"** rather than generalizing [arXiv:2604.15483].

---

# Part 2 — The machine that worked

Why language modelling became an industrial program, in ten minutes.

---

## Pretraining

Chop text into tokens. Predict the next one. Measure the error. Repeat [arXiv:2001.08361].

Translation is French after English. Summarization is short after long. Code is code after code.

**One predictor; the tasks fall out** [arXiv:2001.08361].

---

## Post-training

1. Supervised fine-tuning on curated demonstrations.
2. Alignment against human preference comparisons.
3. RL where correctness is **cheaply and automatically checkable** [arXiv:2511.14759].

Hold onto #3. Robotics does not have it [arXiv:2410.21845].

---

## The returns are predictable

`L(D) = (5.4e13 / D)^0.095` — 10× data multiplies loss by **0.80** [arXiv:2001.08361].

`L(N) = (8.8e13 / N)^0.076` — 10× parameters multiplies it by **0.84** [arXiv:2001.08361].

Fit over more than **7** orders of magnitude [arXiv:2001.08361].

Width, depth and aspect ratio mattered within a **few percent** — architecture was **not** the lever [arXiv:2001.08361].

---

## How to spend a budget

≈ **20** training tokens per parameter [arXiv:2203.15556].

**70B** on **1.4T** tokens beat **280B** on **300B** tokens at equal compute [arXiv:2203.15556].

---

## Honesty beat

A replication found the same paper's own constants imply ≈ **70** tokens per parameter [arXiv:2404.10102].

A robust **trend**, not a precise constant. Do not quote it to three significant figures.

---

## Prerequisite 1 — a cheap capability signal

One objective covering nearly every task, whose loss is a **cheap, dense, low-variance** proxy for capability.

Measured on a held-out shard, for pennies, as often as you like [arXiv:2001.08361].

---

## Prerequisite 2 — data that already exists

**15T** tokens. **44 TB**. **96** crawl snapshots. About **1,536** GPU-hours to curate [arXiv:2406.17557].

Roughly \$10k of compute, for text somebody else already wrote [computed: 1,536 GPU-hours at commodity rates].

---

## Prerequisite 3 — an architecture the machine likes

**46.2%** of theoretical peak sustained on a 540B model [arXiv:2204.02311].

Nearly all dense matrix multiplication; every sequence position processed at once [arXiv:2009.06489].

You pick mechanisms your process is good at. So did they.

---

# Part 3 — The scorecard

![](figures/f02-scorecard.png)

---

## And a fourth constraint

![](figures/f03-frequency-stack.png)

---

# Part 4 — Evaluation

## The arithmetic nobody runs

To separate **50%** from **60%** at 80% power: ≈ **387** trials per policy [computed: two-proportion test, alpha 0.05, 80% power].

Published practice: **10** trials per task [arXiv:2504.16054]. Field-typical: **10–60** [arXiv:2506.18123].

Off by one to two orders of magnitude.

---

## The benchmarks measure something else

A **90M** probe with **no language encoder** scores **92.4–100%** across all four suites [arXiv:2606.04233].

Only ≈ **19.8%** of reported claims on it are statistically significant [arXiv:2606.04233].

---

## And they are sensitive to your variables

Robot initial-state change: **−87.6** points. Camera viewpoint: **−78.4** points [arXiv:2510.13626].

Removing the instruction entirely: barely hurts [arXiv:2510.13626].

Same model, same task, different human resetting the scene: **0% to 100%** [arXiv:2510.17950].

---

## MVP: build the instrument first

Auto-reset fixtures · image-referenced scene reset · partial-credit rubrics · sequential testing (**−70%** trials) [arXiv:2603.13616].

The binding constraint is the **reset**, and that is a fixture problem [arXiv:2510.17950].

---

# Part 5 — The data engine

![](figures/f06-diversity-scaling.png)

---

## Buy diversity, not volume

Demonstrations **saturate** past ≈ **800** per configuration [arXiv:2410.18647].

The validated recipe: **32** environment-object pairs × **50** demos → **85–92.5%** in unseen environments [arXiv:2410.18647].

Collected by **4** people in one afternoon [arXiv:2410.18647].

---

## The sources ladder

![](figures/f08-sources-ladder.png)

---

## Filter, or keep and label?

**Filter:** optimized mixture weights beat uniform by **38%** [arXiv:2408.14037].

**Keep:** with metadata conditioning, more dirty data still helps; without it, the model gets **worse** [arXiv:2604.15483 §IX-E].

Nobody knows the general rule. It is fleet-dependent.

---

## MVP: the data engine

- The **32 × 50** recipe, not volume [arXiv:2410.18647].
- **\$371** handheld stations for throughput [arXiv:2402.10329].
- Metadata in the schema **from day one** [arXiv:2604.15483 §V].
- A **wrench/tactile channel from day one**, even unused. Cheapest option we buy [arXiv:2505.22159].

---

# Part 6 — Model and action representation

![](figures/f07-action-representation.png)

---

## Diffusion is not obviously worth it

| head | throughput | latency | success |
|---|---|---|---|
| autoregressive discrete | 4.2 Hz | 240 ms | — |
| + chunking | 108.8 Hz | 74 ms | 90.2% |
| + continuous L1 | 109.7 Hz | 73 ms | 95.3% |
| + diffusion, 50 steps | 10.1 Hz | 792 ms | 95.4% |

**0.1** points for ~**11×** throughput [arXiv:2502.19645].

---

## Chunking is not an optimization

k = 1 → **1%** success. k = 100 → **44%** [arXiv:2304.13705].

Compounding error: a small deviation moves you out of distribution, and the loop diverges.

Cost: a committed chunk cannot react to a slipping object.

---

## MVP: model

- Chunk. Flow-matching or **L1** head at **5** steps [arXiv:2502.19645].
- Fine-tune an open checkpoint; do **not** pretrain a VLM. **70 GB** for a full fine-tune [repo: openpi README].
- **Keep your 1 kHz controller.** It is a setpoint generator at 20–50 Hz [arXiv:2604.15483 §VII].

---

# Part 7 — Training and post-training

## Why RL is hard here

- **No cheap verifier.** "Is the shirt folded?" has no assertion [arXiv:2601.00675].
- **Resets cost a human** [arXiv:2410.21845].
- **Exploration breaks hardware** [arXiv:2410.21845].
- **The hardware is non-stationary** — wear and drift change the system between iterations [arXiv:2511.14759].

---

## What actually worked

Estimate progress with a value model. Binarize the advantage. **Insert it into the prompt as a text token.** Train on successes *and* failures. Condition on "good" at inference [arXiv:2511.14759].

Throughput **>2×**. Failure rate **halved**. **13 h** of unattended espresso service [arXiv:2511.14759].

---

## The result that should move your priors

**PPO and advantage-weighted regression both lost** to conditioning on a text token [arXiv:2511.14759].

The sophisticated on-policy algorithm was not the answer.

---

## MVP: training

1. **Binary success classifier** — ~1000 frames, ≈ **5 minutes** of teleoperation [arXiv:2410.21845].
2. Human-in-the-loop RL on **one** task: **100%** on **12/12** in **1–2.5 h** each [arXiv:2410.21845].
3. Then advantage-conditioning across the fleet [arXiv:2511.14759].

Never PPO on a 5B model [arXiv:2511.14759].

---

# Part 8 — Runtime and edge

![](figures/f04-latency-budget.png)

---

## Latency is a correctness problem

Chunk seams jump between strategies, producing states that are **out of distribution** [arXiv:2506.07339].

The fix: generate the next chunk while executing the current one, freeze committed actions, inpaint the rest [arXiv:2506.07339].

**No degradation at 100 ms and 200 ms.** Works past **300 ms** [arXiv:2506.07339].

---

## The edge reality

![](figures/f05-hz-per-bandwidth.png)

---

## Read the GB/s line, not the TOPS line

**2070** FP4 TFLOPS [spec: NVIDIA Jetson AGX Thor] → **10.7 Hz** [repo: Isaac-GR00T hardware_recommendation.md].

**273 GB/s** vs the H100's **3.35 TB/s** — a **12×** gap [computed: 3.35 TB/s against 273 GB/s].

At batch 1 you stream every weight, every inference. Throughput tracks **bandwidth** [arXiv:2602.18397].

---

## The 10× wins are algorithmic

Bigger silicon: **1.5–3.3×** [repo: Isaac-GR00T hardware_recommendation.md].

Restructuring the head: **4.2 → 109.7 Hz**, and success **rose** 76.5% → 97.1% [arXiv:2502.19645].

Model-aware quantization holds **97.6%**; a generic LLM quantizer collapses it to **76.3%** [arXiv:2602.20309].

---

## Edge-native, stated exactly

> Compute siting is a business decision, not a capability ceiling.

Latency-absorbing training · few-step heads · bandwidth-first specs · a staged migration on-board.

It rests on one result and falls with it: **200 ms** absorbed without measurable loss [arXiv:2506.07339].

# Part 9 — Five problems that are yours

## Five problems that are yours

1. **No photon-to-torque budget exists.** Every published latency is model-only [repo: openpi websocket_policy_server.py].
2. **Cross-unit variance has never been measured** — named as a confound, quantified at zero [arXiv:2506.18123].
3. **Motor current is sensed and discarded** [repo: openpi aloha_policy.py].
4. **No MTBF, no cost per usable episode** [arXiv:1806.10293].
5. **The e-stop on a biped creates a fall hazard**, and no standard exists [std: ISO 25785-1].

---

## Why #3 is the one I would take

Force channel: **+23.2%** average; plug insertion **~10% → ~80%**, from **244** trajectories [arXiv:2505.22159].

Tactile: USB insertion **5% → 35%** [arXiv:2507.09160].

Blocker is **cross-instance consistency**, and it is solvable: **13%** vs **43%** on sensor swap, σ **0.12** vs **0.54** [arXiv:2409.08276].

---

## The last number

The team behind the strongest published generalist result, by function:

**22** robot hardware · **24** data collection and operations · **10** robot infrastructure [arXiv:2604.15483 App. A].

> Over half the named team is doing your job, not mine.
