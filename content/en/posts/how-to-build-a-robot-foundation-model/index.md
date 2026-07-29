---
title: "How to Build a Robot Foundation Model — and Why the Hard Parts Are Yours"
date: 2026-07-29
draft: false
tags: ["physical-ai", "embodied-ai", "VLA", "robotics", "foundation-models", "edge"]
summary: "A 45-minute argument for a hardware audience: LLMs got a scaling law because three prerequisites were free, robotics gets exactly one, and the two that are missing are measurement and data-engine problems bounded by mechanism and sensing."
---

## The claim this post argues

> Large language models got a scaling law because three prerequisites happened to be satisfied for free. Robotics satisfies exactly one of them. The other two — a cheap signal that tells you whether the model got better, and diverse data that already exists — are not model problems. They are measurement and data-engine problems, and both are bounded by mechanism, sensing, and calibration. On top of those sits a fourth constraint that language models never faced: physics does not wait.

The consequence is a build order, not a research agenda. If you accept the argument, the sequence of what to build and in what order follows from it, and so does the answer to where the compute lives.

## How to read this

This post assumes you know robots: kinematics, actuators, control loops, sensors, tolerances, fleet maintenance, safety cases. It assumes you know **nothing** about transformers, pretraining, tokenization, or scaling laws. Everything language-model-specific is taught in Part 2 and nowhere else [arXiv:2001.08361].

Three conventions run throughout, and they are load-bearing rather than decorative.

**Every claim is typed.** *Disclosed* means a primary source states it. *Inferred* means we derived it from disclosed facts and say so. *Undisclosed* means nobody outside the lab knows, and it goes to the gap ledger rather than getting papered over. *Marketing* means a company blog or press release with no paper and no independent reproduction — labelled inline every time, because in this field the gap between a press release and a result is unusually wide [blog: NVIDIA GEAR GR00T N1.5].

**Every number carries its reference**, and is checked mechanically against a fact ledger. A build script fails if any number in the prose is absent from that ledger or any paragraph lacks a citation [arXiv:2604.15483 §V-E].

**Each workstream runs the same five beats:** motivation and example, the gap, what exists, why it falls short, and our minimum viable path. The repetition is the point — it lets you predict where the argument is going [arXiv:2410.18647].

---

# Part 0 — Cold open: two numbers

A bimanual UR5e that had never seen a single laundry demonstration folded a shirt at **85.6% task progress and 80% success** [arXiv:2604.15483 §IX]. The model driving it had been trained on other robots doing other things.

For comparison, the same task was given to ten expert human teleoperators, each with roughly 375 hours of experience and selected from the top 2 percentile of operators. On their first attempt with that arm they scored **90.9% progress and 80.6% success** [arXiv:2604.15483 §IX]. A generalist model, with no task-specific data, landed within a fraction of a point of expert humans meeting the hardware for the first time.

That is the first number, and it is why this field is worth your time.

Here is the second. That model runs on an H100 [arXiv:2604.15483 App. D]. And a policy trained to 100% success on one camera pose drops to **0%** when the camera is moved 10 cm and rotated 20 degrees [arXiv:2409.03403].

Generality is real. Its fragility is mechanical.

Everything that follows is an attempt to take those two facts seriously at the same time: to explain where the generality comes from, and to be honest that the things standing between it and a deployed product are, disproportionately, problems in your domain rather than mine.

---

# Part 1 — What a robot foundation model actually is

![The input/output contract](figures/f01-io-contract.png)

## The contract

Strip away the vocabulary and a robot foundation model is one function:

**In:** up to 4 camera images at 448×448, up to 6 frames of recent history, the current joint configuration, and one sentence of English [arXiv:2604.15483 §IV].

**Out:** an *action chunk* — a block of H = 50 future joint targets, of which only the first 15 or 25 are executed before the model is called again [arXiv:2604.15483 §VI-B]. The loop runs at 50 Hz on most platforms and 20 Hz on the UR5e [arXiv:2604.15483 §VII].

That is the whole interface. The model is a setpoint generator. It does not replace your servo loop, it feeds it [arXiv:2604.15483 §VII].

Two properties of this contract matter more than they look. The model predicts about a second of motion but commits to only a third of it, which is what buys the time to think [arXiv:2506.07339]. And the adaptation channel is the sentence: a new task is a new instruction, not a new dataset [arXiv:2604.15483 §V].

The chunk is not an implementation detail either, and the ablation behind it is stark. Predicting a single step at a time yields **1%** success on a fine bimanual task; predicting **100** steps at a time yields **44%** [arXiv:2304.13705]. The reason is compounding error — a small deviation moves the robot to a state slightly outside the training distribution, where the next prediction is slightly worse, and the loop diverges. Committing to a block of motion divides the number of opportunities for that to happen [arXiv:2304.13705].

There is one more thing to say about the contract before moving on, because it is where the labs genuinely disagree. Nobody runs the whole model in the servo loop, so everyone splits it — but they split it differently, and the width of the seam is a real design choice. One system passes a single continuous latent vector between halves [blog: Figure Helix]. Another cross-attends to a full sequence of intermediate features [arXiv:2503.14734]. The π-series passes a **human-readable subtask string** plus a generated image [arXiv:2604.15483 §VI]. That last choice costs bandwidth and buys debuggability: when the robot does the wrong thing, you can read what it thought it was doing.

## The four tests

"Trained on a lot of data" and "foundation model" are not the same claim. Four tests separate them, and a large single-task policy fails all four.

- **Adaptation channel.** A new task means writing a new sentence, not collecting a new dataset and launching a new training run [arXiv:2604.15483 §V].
- **Cross-embodiment reuse.** The same weights drive robots with different kinematics [arXiv:2510.03342].
- **Capabilities absent from the robot data.** This is the decisive one, and it has a clean ablation behind it. Strip web pretraining from RT-2-X and it scores **0%** on emergent skills and **1%** on generalization; keep it and the same architecture scores **48.7%** and **47%** [arXiv:2310.08864 Table II]. The semantics are inherited, not learned from robot data.
- **Positive transfer.** Adding robot B's data improves robot A. Across Open X-Embodiment — 22 embodiments, 60 datasets, over 1M trajectories — RT-1-X beat per-dataset specialist methods by **50%** on average [arXiv:2310.08864].

## What generalization means, and which axes are actually demonstrated

"Generalizes" is doing a lot of work in most robotics writing. It helps to be specific about the axis, because the evidence is uneven across them.

- **New objects.** Demonstrated across multiple labs. RT-2 roughly doubled RT-1 on unseen objects and backgrounds across 280 tasks and 6 thousand evaluations [arXiv:2307.15818].
- **New scenes.** The strongest single result in the field. Training on data from 3, then 12, 22, 53, 82, and finally 104 locations improved performance monotonically, and the 104-location model roughly matched a control model trained directly on the test homes [arXiv:2504.16054].
- **New instructions.** Demonstrated, with a caveat below. Evaluation covered 14 scenarios with 3 to 6 open-ended instructions each, in unseen kitchens and bedrooms [arXiv:2604.15483 §IX-B].
- **Cross-embodiment.** Demonstrated, with a documented ceiling. RT-2-X reached **75.8%** against RT-2's **27.3%** on emergent skills [arXiv:2310.08864].
- **Long-horizon composition.** Demonstrated at roughly 10 to 15 minute horizons, such as multi-stage room cleaning [arXiv:2504.16054].
- **Novel physical conditions** — friction, payload, inertia, compliance, stiffness. **This is the weakest axis, and no lab publishes a sweep of it** [arXiv:2604.15483]. Lighting and clutter get measured; the mechanical variables do not.

Hold onto that last one. It is the first place where the published record runs out and your expertise begins.

## The skeptic beat

Two things should temper all of the above, and you should hear them from me rather than find them later.

First, the generalization may be shallower than the headline numbers suggest. Independent work finds that state-of-the-art models exhibit lexical-kinematic shortcuts and semantic feature collapse, with static benchmarks masking the degradation [arXiv:2604.18000]. Related work documents cases where vision simply overrides the language instruction [arXiv:2602.17659].

Second, the field's leading lab says so itself. π0.7's own discussion concedes that at its data scale it is "practically difficult to definitively determine which tasks are truly seen or unseen," and that the model "may well be achieving generalization primarily by remixing" skills from other situations [arXiv:2604.15483]. They argue remixing *is* compositional generalization. That is a defensible position and a contestable one, and it is the honest thing to put on the table before asking anyone to fund a program.

---

# Part 2 — The machine that worked

To understand why robotics is hard, it helps to understand precisely why language modelling turned out to be easy — not easy in engineering effort, but easy in the specific sense that throwing resources at it reliably worked.

## Pretraining and post-training, in your terms

Language model training has two stages. **Pretraining** takes a very large corpus of text and trains the model on a single task: given the text so far, predict what comes next. Text is first chopped into *tokens* — roughly word-fragments — and the model outputs a probability distribution over the next one. Training is then just: read a token, guess the next, measure how wrong you were, adjust, repeat [arXiv:2001.08361].

The reason that trivial-sounding objective is powerful is that almost every capability you have heard of is expressible in it. Translation is French text following English text. Summarization is a short passage following a long one. Code completion is code following code. You do not build a translation system and a summarization system; you build one predictor and the tasks fall out [arXiv:2001.08361].

That gives you a system that continues text plausibly. It does not give you one that follows instructions, refuses harmful requests, or reliably completes a task. **Post-training** supplies those in three moves: supervised fine-tuning on curated demonstrations of good behaviour, alignment against human preference comparisons, then reinforcement learning in environments where correctness can be checked cheaply and automatically — unit tests, arithmetic, formal proofs [arXiv:2511.14759].

Keep that third move in view. "Environments where correctness can be checked cheaply and automatically" is exactly what robotics does not have, and it is the single hardest thing to reproduce in Part 7 [arXiv:2410.21845].

The structural fact that matters most, though, is that stage one does nearly all the work — and stage one requires no new data collection whatsoever. Somebody else already wrote the internet.

## The scaling laws, precisely

The reason this became an industrial program rather than a research direction is that the returns turned out to be *predictable*.

Loss follows power laws in dataset size, parameter count, and compute, each fit over more than 7 orders of magnitude [arXiv:2001.08361]. In dataset size, loss scales as `L(D) = (5.4e13 / D)^0.095`; in parameters, as `L(N) = (8.8e13 / N)^0.076` [arXiv:2001.08361]. Concretely: ten times the data multiplies loss by **0.80**, and ten times the parameters multiplies it by **0.84** [arXiv:2001.08361].

The same work found that architectural choices — width, depth, aspect ratio — mattered only within a few percent across a wide range [arXiv:2001.08361]. **Architecture was not the lever.** That is worth sitting with, because it is the opposite of the intuition most engineers bring.

A later result refined how to spend a compute budget: model size and data should scale at roughly equal rates, giving the rule of thumb of about **20** training tokens per parameter [arXiv:2203.15556]. The demonstration was blunt — a **70B**-parameter model trained on **1.4T** tokens beat a **280B**-parameter model trained on **300B** tokens at equal compute [arXiv:2203.15556].

## Why even this is contested

One honesty beat, because this room will be asked to fund decisions on the strength of these curves. A replication found that the compute-optimal paper's own fitted constants imply roughly **70** tokens per parameter, inconsistent with both its other estimation methods and with the 20-to-1 rule it published [arXiv:2404.10102]. The scaling law is a robust *trend*. It is not a precise constant, and anyone quoting it to three significant figures is overselling.

## The three prerequisites

Strip the machine down and it ran on three conditions, all of which happened to be satisfied for free.

**One: a single objective that expresses nearly every task, whose loss is a cheap, dense, low-variance proxy for capability.** Next-token prediction covers essentially all of language, and you can measure progress on a held-out shard of text for pennies, as often as you like [arXiv:2001.08361].

**Two: data that already exists at close to zero marginal cost.** One widely used corpus distils **15T** tokens — about **44 TB** — from **96** web crawl snapshots. The curation cost roughly **1,536** GPU-hours, which is on the order of \$10k of compute for a corpus that was already lying on the internet [arXiv:2406.17557].

**Three: an architecture that converts compute into dense matrix multiplication at high utilization.** A 540-billion-parameter model sustained **46.2%** of theoretical peak throughput on real hardware [arXiv:2204.02311].

That third one deserves a note, because it is easy to mistake for an aesthetic claim about elegance. It is not. The transformer won substantially because of what it does to a GPU: nearly all its compute is dense matrix multiplication, and during training every position in a sequence can be processed at once rather than one after another [arXiv:2009.06489]. Recurrent architectures had to walk a sequence step by step; transformers do not, so they saturate the machine. A different architecture that was equally expressive but ran at a fraction of peak throughput would have lost, and probably did.

You will recognize the shape of that argument. It is the same one you make when you choose a mechanism that a manufacturing process is good at, over one that is nominally better on paper.

Hold those three. The rest of this talk is what happens when you check them against robots.

---

# Part 3 — The scorecard

![The scorecard](figures/f02-scorecard.png)

## Scoring robotics

**Prerequisite three — architecture — robotics gets for free.** Vision-language-action models *are* transformers, running on the same accelerators, with the same utilization characteristics [arXiv:2503.14734]. This is the one place where the borrowed machinery works unmodified, and it is why architecture is not where this field's problem lives.

**Prerequisite two — free data — fails completely.** There is no web-scale corpus of sensorimotor data, because the internet does not contain joint trajectories. The largest disclosed pretraining set for a generalist is about **10,000** hours of teleoperation [arXiv:2410.24164]. At a fully loaded rate of \$50 per hour — our estimate, since cost per demonstration is undisclosed by every lab in the field — that is roughly **\$500k** of pure human labour [computed: 10,000 h at \$50 per hour]. Set that against the corpus above: comparable capability-per-dollar is off by orders of magnitude, and only one of the two datasets is growing quickly.

**Prerequisite one — a cheap capability signal — fails in a subtler and more damaging way.** Robotics does have a uniform objective: predict the next action chunk. What it does not have is a cheap proxy for capability. Action loss does not tell you whether the robot can do the task; only running the robot does [arXiv:2507.05331]. Every evaluation costs physical time, physical resets, and physical wear.

This is the failure that gets least attention and causes the most waste, so Part 4 takes it first.

## The fourth constraint, which language models never faced

![The frequency stack](figures/f03-frequency-stack.png)

There is also a constraint with no analogue in language modelling at all. A language model can think for longer to answer better. A robot cannot, because the world keeps evolving while it thinks [arXiv:2506.07339].

The stack spans three rate regimes. Semantics — the vision-language model deciding what to do — runs at roughly 1 to 10 Hz. Setpoints — the action expert emitting a chunk — run at 20 to 50 Hz. Your servo loop runs at 200 Hz to 1 kHz [arXiv:2503.14734]. A 5-billion-parameter transformer cannot live in the innermost loop, and every serious system resolves this the same way: split the model and run the halves at different rates [blog: Figure Helix].

## What this derives

Three failures, and each one names a workstream rather than a research topic.

- The capability-signal failure gives you **evaluation**: without a harness that returns a real number, every downstream decision is unfalsifiable [arXiv:2507.05331].
- The free-data failure gives you the **data engine**, and a second question of what to build the model out of, which splits into **model and action representation** and **training and post-training** [arXiv:2410.18647].
- The real-time constraint gives you **runtime and edge** [arXiv:2506.07339].

Five workstreams, all derived rather than chosen. Nothing in the rest of this talk is a survey detour; each part exists because a prerequisite failed.

---

# Part 4 — Workstream: evaluation

## Why this comes first

Putting evaluation before data collection looks like an inverted priority. It is not, and the reason is the prerequisite that failed in Part 3: robotics has no cheap proxy for capability [arXiv:2507.05331].

In language modelling you can measure progress on held-out text for pennies, continuously, during training [arXiv:2001.08361]. In robotics the only instrument that reports whether the model got better is the robot itself, and every reading costs physical time, a physical reset, and physical wear [arXiv:2506.18123]. Until that instrument exists and is trusted, every downstream decision — which data to buy, which architecture to keep, whether the last change helped — is unfalsifiable. You would be spending money without a way to learn from it.

## The gap: the numbers the field reports cannot support the claims made from them

Start with arithmetic that has nothing to do with robotics. To distinguish a policy that succeeds **50%** of the time from one that succeeds **60%** of the time, at conventional significance and **80%** power, you need about **387** trials per policy [computed: two-proportion test, alpha 0.05, 80% power]. To separate **50%** from **55%**, about **1,565** [computed: two-proportion test, alpha 0.05, 80% power]. Even separating **80%** from **90%** takes about **199** [computed: two-proportion test, alpha 0.05, 80% power]. And a 95% confidence interval around an observed **60%** from **20** trials spans **0.39** to **0.78** [computed: Wilson score interval] — which is to say, it tells you almost nothing.

Now compare what gets published. One leading system reports **10** trials per task per policy [arXiv:2504.16054]. Another reports about **125** real rollouts in total across its entire evaluation [arXiv:2503.14734]. Field-typical numbers run **10** to **60** [arXiv:2506.18123]. The most rigorous published audit needed **50** real plus **200** simulated trials per condition, with Bayesian credible regions, to separate policies at all [arXiv:2507.05331].

The gap between what is reported and what would be needed is one to two orders of magnitude.

It gets worse, because the benchmarks themselves are measuring something other than what their names suggest. A **90M**-parameter probe consisting of a vision encoder and a small network — **with no language encoder at all** — scores **99.0**, **100**, **98.8** and **92.4%** across the four suites of the field's most-cited benchmark [arXiv:2606.04233]. A benchmark that a language-blind model saturates is not measuring language-conditioned manipulation. The same analysis finds only about **19.8%** of reported state-of-the-art claims on that benchmark, and **19.7%** on a second, are demonstrably statistically significant [arXiv:2606.04233].

And the sensitivity is pointed exactly at your variables rather than at semantics. Under perturbation of the robot's initial state, one leading model drops **87.6** points; under a camera viewpoint change, **78.4** points [arXiv:2510.13626]. Removing the language instruction entirely barely hurts [arXiv:2510.13626]. The benchmark is near-blind to the instruction and hypersensitive to mechanical pose.

Finally, the human running the test is part of the instrument. In a study spanning **30** tasks, **4** embodiments and **1,500** trials, the same model on the same task moved from **0%** to **100%** success depending on which person was resetting the scene — adaptive operators unconsciously place objects favourably [arXiv:2510.17950].

## What exists

- **Real-to-simulation matching.** Carefully visually-matched simulation correlates with real-world ranking at Pearson **0.924**, with a mean maximum rank violation of **0.056** [arXiv:2405.05941]. Notably, naive domain randomization *degrades* that correlation to **0.778** [arXiv:2405.05941].
- **Distributed pairwise evaluation.** Double-blind A/B comparison across **7** institutions, **4,284** episodes and **612** comparisons produces a stable ranking, and converges in roughly **100** pairwise comparisons [arXiv:2506.18123].
- **Sequential testing.** Stopping early when the result is already clear cuts trials by up to **32%** [arXiv:2506.18123]. Scoring continuous task *progress* instead of binary success saves up to **70%**, and over **450** trials on a four-policy comparison [arXiv:2603.13616].
- **Neural evaluators.** A learned world model replicated an entire distributed evaluation at rank correlation **0.989** for about **100** H100-hours of compute [arXiv:2607.01060].

## Why it falls short

Every one of those techniques attacks the statistics and none of them attacks the binding constraint, which is the **reset**. Sequential testing reduces how many trials you need; it does not make a trial cheaper. Distributed evaluation spreads the cost across institutions; it does not reduce it. The reason the field runs 10 trials instead of 387 is that a human has to walk over and put the objects back [arXiv:2510.17950].

That is a fixture problem, and it is the first of five places in this talk where the ceiling on machine learning progress is set by mechanical design.

## Our MVP

Build the instrument before buying the data.

- **Auto-reset fixtures**, so a trial costs machine time rather than human time. Every reset a person does not perform is statistical power you could not otherwise afford [arXiv:2510.17950].
- **Image-referenced scene reset**, so the scene is restored against a reference photograph rather than an operator's judgement — the documented fix for the **0%**-to-**100%** operator effect [arXiv:2510.17950].
- **Partial-credit rubrics** rather than binary success, scored against a fixed scale — the published rubrics top out at **12** and **6** points for their respective tasks [arXiv:2604.15483 §IX]. This is what makes the **70%** trial saving available [arXiv:2603.13616].
- **Sequential testing** as the default stopping rule, not a fixed trial count [arXiv:2506.18123].
- **Photon-to-torque instrumentation**, described in Part 8, which becomes the first thing this program publishes [repo: openpi websocket_policy_server.py].

The gate for this milestone is not "we have an eval harness." It is: **the harness can distinguish a 10-point difference in success rate using fewer trials than a fixed-sample design would need** [computed: two-proportion test, alpha 0.05, 80% power].

---

# Part 5 — Workstream: the data engine

## The gap: the data does not exist, and cannot be scraped

![The diversity scaling law](figures/f06-diversity-scaling.png)

The asymmetry is the whole problem, and it is worth stating in money.

One widely used text corpus contains **15T** tokens, about **44 TB**, distilled from **96** web crawl snapshots for roughly **1,536** GPU-hours of curation [arXiv:2406.17557]. Call it \$10k of compute, spent on text that already existed [computed: 1,536 GPU-hours at commodity rates].

The largest disclosed pretraining corpus for a generalist robot policy is about **10,000** hours of teleoperation, comprising **903M** timesteps across **7** robot configurations and **68** tasks [arXiv:2410.24164]. Nobody publishes cost per demonstration [arXiv:2506.18123], so we have to estimate: at a fully loaded **\$50** per hour — bracketed by a disclosed operator wage band of **\$25.25** to **\$48.00** per hour before overhead [blog: Tesla job posting via Fortune] — that corpus represents roughly **\$500k** of pure human labour [computed: 10,000 h at \$50 per hour]. That estimate is *inferred*, and we flag it as such.

So: comparable-scale text is effectively free and already written; comparable-scale robot data costs on the order of a hundred dollars per hour and must be produced by people, one episode at a time. Only one of those two corpora is growing quickly.

## The one replicated scaling law

Here is the good news, and it is better than it first appears.

Robot learning does have a reproducible power law — just not the one people usually mean. Fitting the *optimality gap* against the number of distinct environment-object pairs gives `1 − score = 0.88·(M·N)^(−0.30)`, with the separate axes fitting at exponents of **0.31** over objects and **0.26** over environments [arXiv:2410.18647]. That study rests on more than **40,000** collected demonstrations and over **15,000** real rollouts [arXiv:2410.18647].

**A warning before anyone compares exponents.** The **0.30** here and the **0.095** from Part 2 are not commensurable. One is fit against a bounded success gap, the other against unbounded cross-entropy loss [arXiv:2001.08361]. They are different quantities and the numbers cannot be lined up. Anyone who tells you robot scaling is "three times better than language scaling" has made this mistake.

The actionable finding is the second one: **demonstrations saturate.** Beyond roughly **800** total demonstrations for a given configuration, more demonstrations of the same thing buy almost nothing [arXiv:2410.18647]. What keeps paying is *diversity*. The recipe that falls out has been validated on held-out tasks: **32** environment-object pairs at **50** demonstrations each, which reached **85%** to **92.5%** success in unseen environments with novel objects — and was collected by **4** people in a single afternoon [arXiv:2410.18647].

The same shape holds at foundation-model scale. Training on **3**, then **12**, **22**, **53**, **82**, and **104** distinct locations improved generalization monotonically, and the **104**-location model roughly matched a model trained directly on the test homes [arXiv:2504.16054]. Independently, the most valuable single axis of diversity turns out to be **camera pose**, where retrieval-based selection yields up to **70%** improvement [arXiv:2506.13536].

Read those together and the instruction is unambiguous: **buy diversity, not volume.** That is a cheaper instruction than it sounds, and it is the single most important thing on this slide.

## What exists: the sources ladder

![The data sources ladder](figures/f08-sources-ladder.png)

- **Real teleoperation.** The only unambiguously in-distribution source, and the most expensive [blog: Tesla job posting via Fortune].
- **Handheld collection rigs.** A **\$371** station — a printed gripper plus an action camera — collects **111** demonstrations per hour against **35** for conventional teleoperation, a **3.2x** throughput gain, and reached **71.7%** in-the-wild success in novel environments [arXiv:2402.10329]. Against a **\$32k** conventional rig that is roughly a **100x** capital difference [arXiv:2401.02117].
- **Human egocentric video.** The best cost-per-capability on the ladder: one hour of human video is worth about **1,400** demonstrations against **135** for an hour of robot time, and the resulting policy took a laundry task from **55%** to **88%** [arXiv:2410.24221]. The catch is that video has **no action labels**, and closing that gap needs an explicit alignment mechanism, which is worth **44** points when done properly [arXiv:2509.19626].
- **Simulation.** Co-training real with simulated data took unseen-object success from **2.6%** to **9.3%** [arXiv:2406.02523]. Automated demonstration amplification turns fewer than **200** human demonstrations into over **50,000** [arXiv:2310.17596].
- **World models.** The most striking numbers and the highest variance. Generated "neural trajectories" took novel behaviours in *unseen environments* from **0.0%** to **28.5%** [arXiv:2505.12705]. The cost was **240k** samples requiring **54** hours on **1,500** GPUs [arXiv:2505.12705].
- **Autonomous on-robot collection.** **77,000+** episodes over **7** months across **20** robots, with a safety filter raising valid-task rate from **18%** to **83%** — but **no published downstream success rate**, which the authors describe as a proof of concept [blog: AutoRT].
- **Web co-training.** Already collected, and it works: mixing web vision-language data into robot training took unseen-scenario performance from **32%** to **62%** [arXiv:2307.15818].

## Heterogeneity: the engineering tax

Assume you have solved acquisition. You now have data from several sources that do not agree with each other, in four specific ways [arXiv:2310.08864].

**Action semantics.** One dataset records absolute joint positions, another end-effector deltas, another velocities. The largest cross-embodiment collection explicitly declines to reconcile these, stating that coordinate frames are not aligned across datasets [arXiv:2310.08864].

**Dimensionality.** Different robots have different numbers of joints. The common fix is to zero-pad everything to the largest, which for one system means an **18**-dimensional action vector regardless of the robot [arXiv:2410.24164].

**Camera pose.** Already flagged as the highest-value diversity axis [arXiv:2506.13536] — and the one where a **10** cm shift costs you the entire policy [arXiv:2409.03403]. Only **27%** of one major collection has wrist cameras at all, and **56%** has language annotations [arXiv:2405.12213].

**Control frequency.** Datasets range from **3** Hz to **50** Hz [arXiv:2310.08864]. The consequence deserves saying aloud: a **50**-step chunk is **1** second of motion on one robot and **2.5** seconds on another — *inferred* from the two disclosed rates [computed: 50 steps at 50 Hz and 20 Hz]. The same number means different things.

## Filter or keep?

This is genuinely unresolved, and both camps have real evidence.

**Filter.** Optimizing the mixture weights over source datasets beats uniform weighting by **38%**, and beats human-chosen weights by **32%** [arXiv:2408.14037]. A separate method reaches state of the art using under **33%** of the available data [arXiv:2506.19121].

**Keep, and label instead.** The alternative is to retain low-quality data and tell the model what it is looking at. The disclosed schema conditions on execution speed binned in **500**-step intervals, a human quality score from **1** to **5**, a boolean for whether a mistake occurred, and the control mode [arXiv:2604.15483 §V]. The whole metadata block is dropped **15%** of the time and each component another **5%**, so the model works with or without it [arXiv:2604.15483 §V]. At inference you simply ask for the good behaviour.

The result that makes this worth the annotation cost: with metadata conditioning, performance keeps improving as the dataset grows *even as average quality falls*; without it, the model gets worse on the larger, dirtier dataset [arXiv:2604.15483 §IX-E]. The same work found that deleting the most task-diverse **20%** of data hurts far more than deleting a random **20%** [arXiv:2604.15483 §IX-E].

Nobody knows the general rule. What is clear is that the choice is fleet-dependent, and that labelling converts a liability into an asset if you can afford the annotation.

## Our MVP

- **Buy diversity, not volume**, at the validated recipe: **32** environment-object pairs at **50** demonstrations each, one distinct object per environment [arXiv:2410.18647].
- **Handheld rigs for throughput**, at **\$371** a station, reserving conventional teleoperation for the cases that need force feedback [arXiv:2402.10329].
- **Metadata in the schema from day one** — speed, quality, mistake flag, control mode — because retrofitting annotations onto collected episodes costs more than recording them [arXiv:2604.15483 §V].
- **A wrench and tactile channel in the schema from day one, even though the model will not consume it yet.** This is the cheapest option this program buys. Part 10 explains why it may be the most valuable [arXiv:2505.22159].
- **Co-train on open corpora rather than collecting more yourself.** Open data can be **9.1%** of a working mixture [arXiv:2410.24164], and in-domain data as little as **2.4%** when the remainder is right [arXiv:2504.16054].

---

# Part 6 — Workstream: model and action representation

*This part and the next are the two the speaker compresses if the clock demands it.*

## The gap: the web gives you semantics, and no actions

Every modern system in this field is the same three boxes: an internet-pretrained vision-language model, a narrow bottleneck, and a decoder that emits action chunks [arXiv:2503.14734]. The backbone is borrowed and supplies meaning for free — it already knows what a mug is, and what "put it in the sink" implies [arXiv:2310.08864 Table II].

Actions are not free. The web contains no torque commands. So every genuine design argument in this workstream is about two things: how you represent an action, and how you make a multi-billion-parameter transformer respect a servo loop.

## Action representation is the central axis

![The action-representation trade-off](figures/f07-action-representation.png)

The clearest evidence available holds the backbone fixed and varies only the action head. Four configurations, same model, same benchmark [arXiv:2502.19645]:

| configuration | throughput | latency | success |
|---|---|---|---|
| autoregressive discrete tokens | 4.2 Hz | 240 ms | — |
| + chunking and parallel decoding | 108.8 Hz | 74 ms | 90.2% |
| + continuous L1 regression | 109.7 Hz | 73 ms | 95.3% |
| + diffusion, 50 steps | 10.1 Hz | 792 ms | 95.4% |

Read the last two rows carefully, because they overturn a common assumption. Diffusion buys **0.1** points of accuracy over plain L1 regression, and costs about **11x** in throughput and roughly **10x** in latency [computed: 109.7 Hz against 10.1 Hz]. The sophisticated generative head is not obviously worth it once you have chunking and enough backbone capacity [arXiv:2502.19645].

The discrete-token route deserves its own note, because it involves a real and instructive trade. Naive per-dimension binning is catastrophically wasteful at high control rates: at **50** Hz on a **14**-degree-of-freedom bimanual system it costs **700** tokens per chunk [arXiv:2501.09747]. A frequency-domain tokenizer compresses that to **53**, a **13.2x** reduction [arXiv:2501.09747]. That buys a genuine advantage in training: matching flow-matching quality on a large mixture using **5x** fewer GPU hours, with better language following [arXiv:2501.09747].

And then it loses at inference, for a reason worth internalizing. Flow matching produces a one-second chunk in about **100** ms on an RTX 4090; the token route needs about **750** ms on the same hardware, because those tokens must be decoded one at a time through the full backbone rather than refined in a few passes through a small head [arXiv:2501.09747]. **Training cost and inference cost pull in opposite directions here**, and which one binds depends on whether you are still building or already deployed.

## Action chunking, and why it exists

Covered in Part 1, and worth restating as a design rule: predicting one step at a time gives **1%** success, predicting **100** steps gives **44%** [arXiv:2304.13705]. Chunking is not an optimization, it is what makes imitation learning work at all on fine manipulation.

The cost is reactivity. A committed chunk of roughly **1** to **1.8** seconds cannot respond to an object slipping mid-motion [computed: 50 to 90 steps at 50 Hz]. That trade is managed in Part 8, not eliminated.

## Why it falls short

Three caveats belong on the record.

**Attaching an action head naively damages the backbone.** Bolting a continuous regression head onto a pretrained vision-language model significantly harms both training speed and the transfer of web knowledge, which is why production systems block gradients from the action expert back into the backbone [arXiv:2604.15483 §IV].

**Vendor frequency claims are not what you think.** A published "120 Hz" figure describes the rate at which a policy emits actions, not the rate at which the system reacts to the world [arXiv:2503.14734]. That same system's own numbers — **16** actions produced in **63.9** ms — imply roughly **250** actions per second, which the paper does not reconcile with its headline figure [computed: 16 divided by 0.0639 s]. Treat all such numbers as output rate until proven otherwise.

**Reproducibility is uneven.** One widely-cited system has no paper at all [blog: Figure Helix]. Another discloses no parameter counts, control frequencies or chunk lengths [arXiv:2510.03342]. Build on what you can actually inspect.

## Our MVP

- **Chunk, and use a flow-matching or L1 head at 5 steps.** Reach for diffusion only after you have *measured* a multimodality failure, not because a paper recommended it [arXiv:2502.19645].
- **Fine-tune an open checkpoint; do not pretrain a vision-language model.** The disclosed floors are modest: over **8** GB of VRAM to infer, **22.5** GB for low-rank fine-tuning, **70** GB for a full fine-tune [repo: openpi README].
- **Keep your existing 1 kHz controller.** The model is a setpoint generator at **20** to **50** Hz. It does not replace your control stack and should not be asked to [arXiv:2604.15483 §VII].

---

# Part 7 — Workstream: training and post-training

## The gap: there is no cheap verifier, and the hardware moves under you

Recall the third move of language post-training: reinforcement learning in environments where correctness is checked cheaply and automatically [arXiv:2511.14759]. Every word of that is false for robots.

**No cheap verifier.** "Is the shirt folded?" has no assertion you can write. The judge must itself be a learned model, and learned judges are unreliable — a dedicated benchmark concludes that no current model excels across tasks [arXiv:2601.00675]. Worse, the errors are asymmetric: a false positive teaches the policy that a failure was a success and traps it, while a false negative merely wastes a sample [arXiv:2601.00675].

**Resets cost a human.** Covered in Part 4, and it binds here twice as hard, because reinforcement learning needs far more episodes than evaluation does [arXiv:2410.21845].

**Exploration breaks hardware.** This is why essentially every method that works on real robots has a human in the loop by construction [arXiv:2410.21845].

**The hardware is non-stationary.** Gripper wear and calibration drift change the system between training iterations. The environment your policy was optimized against is not quite the one it will run in tomorrow [arXiv:2511.14759].

## The stages, honestly mapped

| language stage | robotics analogue | what breaks |
|---|---|---|
| internet pretraining | the vision-language backbone | supplies semantics, zero actions |
| cross-embodiment pretraining | large mixed-robot corpora | heterogeneity tax, Part 5 |
| supervised fine-tuning | behaviour cloning on the target robot | orders of magnitude costlier per sample |
| preference alignment | teleoperator interventions as implicit preference | an intervention *is* "not that, this" |
| verifiable-reward RL | binary task success | the verifier is learned, noisy, and per-task |

## What exists, and what actually worked on hardware

The strongest real-world result recasts reinforcement learning as conditional imitation rather than policy optimization. A distributional value model — **670M** parameters, **201** output bins — estimates progress; the advantage between consecutive states is computed, **binarized, and inserted into the prompt as a text token**; and the policy is then trained on *all* data, successes and failures alike. At inference you simply condition on the token that says "good" [arXiv:2511.14759].

The results are the most convincing in the field: throughput **more than doubled**, failure rate **at least halved**, above **90%** success on every task except diverse laundry, and **13** hours of continuous unattended espresso service [arXiv:2511.14759]. Data cost per iteration was about **300** trajectories for one task, and **600** autonomous plus **360** intervention trials for another [arXiv:2511.14759].

The detail that should change your priors: **both PPO and advantage-weighted regression underperformed this approach** [arXiv:2511.14759]. The sophisticated on-policy algorithm lost to conditioning on a text token.

Second, human-in-the-loop reinforcement learning reached **100%** success on **12** of **12** real tasks in **1** to **2.5** hours each, improving from an average of **49.7%**, and cutting cycle time **1.8x** from **9.6** to **5.4** seconds [arXiv:2410.21845]. Its verifier is the part worth stealing: a binary image classifier trained on about **200** positive and **1000** negative frames, which is roughly **5** minutes of teleoperation [arXiv:2410.21845].

Third, the cautionary tale. Reinforcement learning in simulation took a long-horizon benchmark from **17.3%** to **91.7%** starting from a single demonstration [arXiv:2509.09674]. It also discovered that it could push an object to the target rather than performing the demonstrated grasp-transport-place [arXiv:2509.09674]. That is simultaneously this field's genuine "aha" moment and its reward-hacking warning. Transferred to a real robot, the same approach drops to **38.5%** [arXiv:2509.09674].

## Why it falls short

Running reinforcement learning directly on the full backbone destabilizes training and degrades performance; the methods that work freeze the backbone and train only the action head, or alternate between online updates and supervised replay [arXiv:2501.16664]. And none of this removes the verifier problem — it relocates it into a classifier you have to train and trust.

## Our MVP

- **Build a binary success classifier first**, before any reinforcement learning. About **1000** labelled frames and **5** minutes of teleoperation, and it currently outperforms a general-purpose model used as a judge [arXiv:2410.21845].
- **Then human-in-the-loop RL on exactly one task**, the one whose failure dominates your cycle time [arXiv:2410.21845].
- **Then advantage-conditioning across the fleet**, which is what converts a per-task result into a program [arXiv:2511.14759].
- **Never run PPO on a 5-billion-parameter model.** The published comparison says it loses, and it will cost you months to rediscover that [arXiv:2511.14759].

---

# Part 8 — Workstream: runtime and edge

## The gap: every published latency number is model-only

![The latency budget](figures/f04-latency-budget.png)

This is the part where the published record is thinnest and your instruments are best.

The disclosed figures are real and useful. One system reports **73** ms total on an RTX 4090, decomposing into **14** ms of image encoding, **32** ms for the observation pass, and **27** ms across ten flow steps [arXiv:2410.24164]. Off-board over wireless it is **86** ms, of which **13** ms is network [arXiv:2410.24164]. A later generation runs **38** ms in its minimal configuration and **127** ms worst case on a single H100 [arXiv:2604.15483 App. D]. With four camera streams, the previous generation hit a **300** ms real-time barrier [arXiv:2511.14759].

Now notice what every one of those numbers excludes. They are model-only. The reference server instruments inference time and deserialization and nothing else [repo: openpi websocket_policy_server.py]. Camera exposure, image signal processing, cable transport, timestamp skew between streams, and actuator response are **nobody's published number** [repo: openpi websocket_policy_server.py].

Look at the figure again. The hatched blocks are not small, and they are not measured. That is the shape of the opportunity.

## Why latency is a correctness problem, not a comfort problem

If inference is slow, the naive expectation is that the robot moves in a jerky, stop-start way and looks bad. That is not what happens. What happens is worse.

Adjacent chunks can jump between different strategies, producing discontinuities at the seam that are themselves **out of distribution** [arXiv:2506.07339]. The robot does not merely pause — it enters a state unlike anything in its training data, and the policy's behaviour there is undefined. Late actions are also planned against a world that has since moved.

The fix is elegant and it changes the economics of this entire program. Generate the next chunk while executing the current one; freeze the actions already committed to the hardware; and let the model *inpaint* the remainder, soft-masking the middle [arXiv:2506.07339]. Measured over **480** episodes and **28** robot-hours: **no degradation at 100 ms and 200 ms of injected delay**, and continued success beyond **300** ms, where synchronous execution degrades sharply and naive temporal smoothing "does not work at all" and triggered protective stops [arXiv:2506.07339]. Baked into training by sampling delays of **0** to **12** timesteps, it costs nothing at inference and yields a **240** ms budget at **50** Hz [computed: 12 timesteps at 50 Hz].

**That result is the load-bearing premise of this program's thesis.** If a system tolerates a few hundred milliseconds of delay without losing performance, then where the compute sits stops being a capability question and becomes a cost, power, and packaging question.

## The edge reality, stated honestly

![Throughput against memory bandwidth](figures/f05-hz-per-bandwidth.png)

Anyone promising frontier-scale generality on-board today is overselling. The vendor's own measurements for one open model, under an optimized runtime, are: **35.9** Hz on an H100, **10.7** Hz on the flagship embedded module, and **4.6** Hz on the previous-generation module, with the vendor stating the latter is suitable only for slow, non-reactive tasks [repo: Isaac-GR00T hardware_recommendation.md]. The leading closed system runs on an H100 [arXiv:2604.15483 App. D].

Here is the part that should change how you write a procurement spec. The flagship module is rated at **2070** FP4 teraflops [spec: NVIDIA Jetson AGX Thor] and delivers **10.7** Hz [repo: Isaac-GR00T hardware_recommendation.md]. Its memory bandwidth is **273** GB/s against the H100's **3.35** TB/s — a **12x** gap, *inferred* by division [computed: 3.35 TB/s against 273 GB/s]. At batch size one you are streaming billions of weights through memory for every single inference, so throughput tracks **bandwidth**, not arithmetic. Independent measurement confirms it: the memory-bound action expert costs **26.20** ms on the embedded module against **7.25** ms on a consumer RTX 4090, a **3.6x** penalty on a part with vastly more nominal compute [arXiv:2602.18397].

**Read the GB/s line, not the TOPS line.**

Two more practical warnings. The metric that matters is reaction time, not throughput: time-to-first-action for one system is **80.0** ms on an RTX 4090 but **303.3** ms on an RTX 4060, where baseline inference runs at **3** Hz [arXiv:2603.19199]. A cheaper GPU does not make a reactive policy slower — it makes it *open-loop*. And published latency for the same model on the same silicon spans **46** ms to **246** ms depending on harness and precision, a **5x** spread [arXiv:2602.18397]. Always ask which stack produced the number.

## Why the 10x wins are algorithmic

Buying bigger silicon buys you a factor of two or three. Compilation gives **1.5** to **2.9x**, and an optimized inference runtime **1.5** to **3.3x** [repo: Isaac-GR00T hardware_recommendation.md]. Those are worth having and they are not where the leverage is.

The leverage is in the algorithm. Restructuring the action head took one system from **4.2** to **109.7** Hz — **26x** — while success *rose* from **76.5%** to **97.1%** [arXiv:2502.19645]. Distilling the flow model to a single evaluation took another from **274** ms to **83** ms at **98.75%** against **97.75%** [arXiv:2604.05656].

Quantization is nearly free **if it is done with knowledge of this model class**, and catastrophic otherwise. A method designed for these models holds **97.6%** against **97.1%** at full precision while shrinking the model from **4.27** GB to **1.28** GB [arXiv:2602.20309]. A general-purpose language-model quantizer applied to the same network collapses it to **76.3%** — a **21**-point drop [arXiv:2602.20309]. And there is a cliff: **3.0** bits per weight holds **94.8%**, but **2.0** bits falls to **48.0%** [arXiv:2605.24011].

## Our MVP

- **Wire asynchronous execution and real-time chunking into the runtime.** This is not yet done in the open stacks: the reference client is synchronous and blocks on every chunk [repo: openpi websocket_policy_server.py]. It is the single highest-return engineering item in this document.
- **Spend on few-step distillation and model-aware quantization before spending on silicon.** **26x** against **3x** is not a close call [arXiv:2502.19645].
- **Specify compute modules on memory bandwidth**, and treat the TOPS number as marketing until you have measured Hz on your own model [arXiv:2602.18397].
- **Stage the migration.** Off-board through the first three milestones, on-board at the fourth, once the model has shrunk and the delay tolerance is proven.

The thesis of this program, stated exactly: **edge-native means the system is designed so that compute siting is a business decision, not a capability ceiling.** Latency-absorbing training, few-step action heads, bandwidth-first compute specifications, and a migration path that moves compute on-board as the model shrinks and the silicon widens. It rests on one result and falls with it — that a properly trained system absorbs **200** ms of delay without measurable loss [arXiv:2506.07339].

---

# Part 9 — The plan

## The vertical, and why this one

The program starts with **mixed-SKU order picking** and generalizes from there. Three reasons, in order of importance.

It has real customer value from the first working policy, which means the data engine is funded by something other than optimism [arXiv:2511.14759]. It has naturally high object and scene diversity, which is precisely what the one replicated scaling law rewards — and the recipe of **32** environment-object pairs at **50** demonstrations each maps onto a picking cell almost directly [arXiv:2410.18647]. And it is contact-rich at exactly the points where it fails, which is what makes the force and tactile argument in Part 10 a business case rather than a research aspiration [arXiv:2505.22159].

Generality is not a separate research bet bolted on at the end. It is what the data engine produces once metadata conditioning lets dirty data keep paying [arXiv:2604.15483 §IX-E].

## The milestone ladder

![The build order](figures/f09-milestone-ladder.png)

| | milestone | target | gate | compute |
|---|---|---|---|---|
| **M0** | Measure | an eval harness that detects a 10-point difference, plus a published photon-to-torque budget | separate 50% from 60% at 80% power using fewer trials than a fixed-sample design; every millisecond from exposure to commanded torque accounted for | off-board |
| **M1** | Collect | diversity-first dataset with the metadata schema in place from day one | a policy fine-tuned on it generalizes to held-out object×environment pairs at the harness's significance bar | off-board |
| **M2** | First policy | a deployable picking policy, asynchronous runtime | beats the site's existing heuristic picker on the site's own metric, at significance | off-board |
| **M3** | Learn from experience | an autonomous improvement loop, not a one-shot training run | throughput doubled and failure rate halved against M2 | off-board |
| **M4** | Move the compute | same policy, same success, on an embedded module | reaction time no worse than M2's off-board figure once delay is absorbed, at 130 W or less, with 1 point or less of success loss | **on-board** |
| **M5** | Generalize | second embodiment and second site, no new pretraining run | zero-shot progress on the unseen embodiment within one harness confidence interval of the trained embodiment | **on-board** |

Every gate is falsifiable, and M4's and M5's are stated relative to the M0 harness rather than as absolute numbers assumed in advance — because assuming them in advance is how programs end up unable to tell whether they succeeded [arXiv:2507.05331].

## Why M4 is the investor slide

Through M3 the system runs off-board. That is defensible for a pilot and fatal for a business, because it prices every robot at a datacentre GPU drawing **700** W [spec: NVIDIA H100 SXM].

M4 replaces that with a **\$3,499** module drawing **40** to **130** W [spec: NVIDIA Jetson AGX Thor]. The technical work is Part 8's: model-aware quantization holding **97.6%** [arXiv:2602.20309], few-step distillation, and delay absorption already proven to **200** ms [arXiv:2506.07339].

That is the difference between a demo and a business, and it is the milestone this crew most directly de-risks — because whether it lands depends on thermal envelope, mounting, power delivery, and sensor synchronization at least as much as on the model.

## The resource model

We give this parametrically rather than as a single number, for a reason that matters under diligence: every total below is derived from a **cited unit cost** and a **named parameter you can vary**. An invented lump sum survives a first meeting and nothing after it.

**Disclosed unit anchors.**

- Teleoperation labour: a posted operator wage band of **\$25.25** to **\$48.00** per hour [blog: Tesla job posting via Fortune]; separately, roughly **\$3** per hour reported for collection centres in China [blog: Rest of World]. Both are *marketing*-class sources and are labelled as such.
- Collection rigs: **\$371** per handheld station [arXiv:2402.10329]; under **\$300** for a low-cost leader-arm rig [arXiv:2309.13037]; under **\$20k** for a conventional bimanual station [arXiv:2304.13705]; **\$32k** with a mobile base [arXiv:2401.02117]; and **\$0.6k** against roughly **\$60k** for an exoskeleton alternative [arXiv:2503.03081].
- Edge compute: **\$3,499** at **273** GB/s and **40** to **130** W [spec: NVIDIA Jetson AGX Thor]; about **\$1,999** at **204.8** GB/s and **15** to **60** W for the prior generation [spec: NVIDIA Jetson AGX Orin].
- Training compute: over **8** GB VRAM to infer, **22.5** GB for low-rank fine-tuning, **70** GB for a full fine-tune [repo: openpi README]. Pretraining a vision-language model is explicitly out of scope.
- Headcount *shape*, sanity-checked against the only disclosed breakdown available: **22** robot-hardware, **24** data-collection-and-operations, and **10** robot-infrastructure contributors [arXiv:2604.15483 App. A].

**Two figures we deliberately refuse to use.** The per-hour teleoperation-data prices circulating in vendor material have no published methodology, and we exclude them rather than dress them up [arXiv:2506.18123]. And cost per demonstration is undisclosed by every lab in this field [arXiv:2506.18123] — which is why this program instruments it from M0 and treats it as an owned metric rather than an industry constant.

The capital lever worth naming explicitly: the handheld-versus-conventional rig decision is roughly a **100x** difference in capital for a **3.2x** throughput gain, *inferred* from the two disclosed prices [computed: \$371 against \$32k]. It is the highest-leverage procurement choice in M1, and its cost is the loss of the force channel [arXiv:2402.10329].

**A worked instantiation, so the model is not abstract.** Take M1 at the validated recipe and vary nothing else.

| line | derivation | figure |
|---|---|---|
| demonstrations required | 32 pairs × 50 demos [arXiv:2410.18647] | 1,600 |
| collection hours, handheld | 1,600 ÷ 111 demos/h [arXiv:2402.10329] | ≈ 14 h |
| collection hours, conventional teleoperation | 1,600 ÷ 35 demos/h [arXiv:2402.10329] | ≈ 46 h |
| collection labour, handheld | 14 h × \$50/h [computed: 10,000 h at \$50 per hour] | ≈ \$700 |
| rig capital, 4 handheld stations | 4 × \$371 [arXiv:2402.10329] | ≈ \$1,484 |

The number that should stop the room is the last two rows. The *collection* cost of the diversity recipe that reached **85%** to **92.5%** in unseen environments is in the low thousands of dollars [arXiv:2410.18647]. Four people, one afternoon [arXiv:2410.18647].

That is not where a robot foundation model program's money goes. The money goes into the fixtures that let you evaluate it, the fleet that keeps running while you do, and the engineering that gets it onto a **130** W module [spec: NVIDIA Jetson AGX Thor]. Which is the argument of this entire talk, arriving as a spreadsheet row.

## What we will not do

- **Pretrain a vision-language model.** Fine-tune an open checkpoint [repo: openpi README].
- **Build a simulator before the harness.** Simulation's demonstrated value is in unseen-object generalization and demonstration amplification [arXiv:2406.02523], neither of which you can detect without M0.
- **Run PPO on a 5-billion-parameter model.** The published comparison says it loses to conditioning on a text token [arXiv:2511.14759].
- **Buy volume before diversity.** Demonstrations saturate around **800** per configuration; environments and objects do not [arXiv:2410.18647].

## Risk register

- **Cross-embodiment transfer may not materialize.** A **900k**-trajectory, **20**-embodiment study states plainly that its results do not yet show significant positive transfer across embodiments [arXiv:2408.11812]. M5 is the milestone most exposed, which is why it is last.
- **Edge migration may stall** if model scale outruns memory bandwidth. The mitigation is that M4's work is algorithmic — **26x** from restructuring against **3x** from silicon [arXiv:2502.19645] — so it does not depend on a vendor roadmap.
- **The pilot's task distribution may be too narrow to seed generality.** Mitigated by measuring against held-out object×environment pairs from M1 rather than at M5.
- **Reinforcement learning may reward-hack.** It has been observed doing exactly this, substituting a push for a demonstrated grasp [arXiv:2509.09674]. Mitigated by rubric-based scoring rather than binary success [arXiv:2603.13616].
- **Evaluation may fail to detect regressions** at achievable trial counts. This is the risk M0 exists to retire, and the reason it is first [arXiv:2507.05331].

---

# Part 10 — What I need from you

Five problems. Each is a genuine hole in the published record, each is bounded by mechanism, sensing, or fleet operations rather than by modelling, and each is something this room can close and I cannot.

**One: nobody has published a photon-to-torque budget.** Every disclosed latency in this entire talk is model-only; the reference implementation instruments inference and deserialization and nothing else [repo: openpi websocket_policy_server.py]. Build a hardware-triggered loop — an LED and a photodiode — that measures exposure to commanded torque on one platform, publish it, and you own a number the whole field is missing.

**Two: cross-unit variance has never been measured.** The field's most careful evaluation effort names "manufacturing differences between said robots" as a confound and quantifies it at zero [arXiv:2506.18123]. Run one policy across N nominally identical units and publish the variance budget. Every fleet-scaling claim in this field currently rests on an assumption nobody has tested.

**Three: motor current is measured and then thrown away.** The reference environment reads effort into the observation dictionary, and the policy input is images plus a **14**-dimensional joint-position vector [repo: openpi aloha_policy.py]. Force and tactile are absent from *every* frontier system. Meanwhile adding a force channel is worth **23.2%** on average and takes plug insertion from about **10%** to about **80%** — from **244** trajectories [arXiv:2505.22159]. Tactile sensing takes USB insertion from **5%** to **35%** and charger insertion from **40%** to **90%** [arXiv:2507.09160].

The blocker is not sensitivity, it is **cross-instance consistency** — a policy trained on one sensor must work on the replacement. That is a manufacturing and calibration problem, and it is solvable: swapping to a different physical skin costs **13%** against **43%** for the prior design, with normalized cross-instance variation of **0.12** against **0.54**, and a **12**-second replacement against **236** seconds [arXiv:2409.08276]. A tactile sensor that reproduces across units and survives wear is worth more than another order of magnitude of model scale.

**Four: nobody publishes uptime or cost per usable episode.** A landmark data-collection effort's only operational disclosure is that objects were replaced every **4** hours [arXiv:1806.10293]. Another reports **9,527** robot-hours and no downtime figure at all [arXiv:2104.08212]. Instrument one rig for mean time between failures, reset time, and discard rate, and you can compute a number no lab in this field can currently state.

**Five: a dynamically stable humanoid collapses when you cut power**, which means the emergency stop itself creates a fall hazard — and no humanoid safety standard exists, with the relevant document still at working-draft stage [std: ISO 25785-1]. Meanwhile the revised industrial standard now requires power-and-force-limiting compliance to be established by **physical measurement rather than calculation** [std: ISO 10218:2025]. That turns a paperwork exercise into a metrology problem, which is yours.

## Where this leaves us

The argument of this talk has been that generality in robots is real, that it comes from diversity rather than volume, and that the three things standing between it and a product are a measurement instrument, a data engine, and a power budget [arXiv:2410.18647].

None of those three is a modelling problem.

The last number I have is the most direct evidence for that claim. The team behind the strongest published generalist result lists its contributors by function: **22** in robot hardware, **24** in data collection and operations, and **10** in robot infrastructure [arXiv:2604.15483 App. A].

Over half the named team is doing your job, not mine.

---

## Bibliography

**Peer-reviewed and conference**

- *Open X-Embodiment: Robotic Learning Datasets and RT-X Models* — arXiv:2310.08864, Oct 2023 (ICRA 2024). https://arxiv.org/abs/2310.08864
- *RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control* — arXiv:2307.15818, Jul 2023 (CoRL 2023). https://arxiv.org/abs/2307.15818
- *RT-1: Robotics Transformer for Real-World Control at Scale* — arXiv:2212.06817, Dec 2022. https://arxiv.org/abs/2212.06817
- *π0.5: a Vision-Language-Action Model with Open-World Generalization* — arXiv:2504.16054, Apr 2025 (CoRL 2025). https://arxiv.org/abs/2504.16054
- *RoboArena: Distributed Real-World Evaluation of Generalist Robot Policies* — arXiv:2506.18123, Jun 2025 (CoRL 2025). https://arxiv.org/abs/2506.18123
- *Data Scaling Laws in Imitation Learning for Robotic Manipulation* — arXiv:2410.18647, Oct 2024 (ICLR 2025 Oral). https://arxiv.org/abs/2410.18647
- *Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware* (ACT / ALOHA) — arXiv:2304.13705, Apr 2023 (RSS 2023). https://arxiv.org/abs/2304.13705
- *Universal Manipulation Interface* — arXiv:2402.10329, Feb 2024 (RSS 2024). https://arxiv.org/abs/2402.10329
- *Diffusion Policy: Visuomotor Policy Learning via Action Diffusion* — arXiv:2303.04137, Mar 2023 (RSS 2023). https://arxiv.org/abs/2303.04137
- *RoboCasa: Large-Scale Simulation of Everyday Tasks* — arXiv:2406.02523, Jun 2024 (RSS 2024). https://arxiv.org/abs/2406.02523
- *SimplerEnv: Evaluating Real-World Robot Manipulation Policies in Simulation* — arXiv:2405.05941, May 2024 (CoRL 2024). https://arxiv.org/abs/2405.05941
- *RoVi-Aug: Robot and Viewpoint Augmentation* — arXiv:2409.03403, Sep 2024 (CoRL 2024, best paper). https://arxiv.org/abs/2409.03403
- *EgoMimic: Scaling Imitation Learning via Egocentric Video* — arXiv:2410.24221, Oct 2024 (ICRA 2025). https://arxiv.org/abs/2410.24221
- *EgoBridge* — arXiv:2509.19626, Sep 2025 (NeurIPS 2025). https://arxiv.org/abs/2509.19626
- *MimicGen* — arXiv:2310.17596, Oct 2023 (CoRL 2023). https://arxiv.org/abs/2310.17596
- *Scaling Laws for Neural Language Models* — arXiv:2001.08361, Jan 2020. https://arxiv.org/abs/2001.08361
- *Training Compute-Optimal Large Language Models* (Chinchilla) — arXiv:2203.15556, Mar 2022. https://arxiv.org/abs/2203.15556
- *PaLM: Scaling Language Modeling with Pathways* — arXiv:2204.02311, Apr 2022. https://arxiv.org/abs/2204.02311
- *The Hardware Lottery* — arXiv:2009.06489, Sep 2020. https://arxiv.org/abs/2009.06489
- *QT-Opt: Scalable Deep Reinforcement Learning for Vision-Based Robotic Manipulation* — arXiv:1806.10293, Jun 2018. https://arxiv.org/abs/1806.10293
- *Deep Reinforcement Learning at Scale* (MT-Opt) — arXiv:2104.08212, Apr 2021. https://arxiv.org/abs/2104.08212
- *On the Opportunities and Risks of Foundation Models* — arXiv:2108.07258, Aug 2021. https://arxiv.org/abs/2108.07258

**arXiv preprints (lab-authored, not independently reproduced)**

- *π0.7: a Steerable Generalist Robotic Foundation Model with Emergent Capabilities* — arXiv:2604.15483, Apr 2026. https://arxiv.org/abs/2604.15483
- *π0: A Vision-Language-Action Flow Model for General Robot Control* — arXiv:2410.24164, Oct 2024. https://arxiv.org/abs/2410.24164
- *π\*0.6: a VLA that Learns from Experience* (RECAP) — arXiv:2511.14759, Nov 2025. https://arxiv.org/abs/2511.14759
- *FAST: Efficient Action Tokenization for Vision-Language-Action Models* — arXiv:2501.09747, Jan 2025. https://arxiv.org/abs/2501.09747
- *Real-Time Chunking* — arXiv:2506.07339, Jun 2025. https://arxiv.org/abs/2506.07339
- *MEM: Multi-scale Embodied Memory* — arXiv:2603.03596, Mar 2026. https://arxiv.org/abs/2603.03596
- *GR00T N1: An Open Foundation Model for Generalist Humanoid Robots* — arXiv:2503.14734, Mar 2025. https://arxiv.org/abs/2503.14734
- *Gemini Robotics 1.5* — arXiv:2510.03342, Oct 2025. https://arxiv.org/abs/2510.03342
- *Gemini Robotics 1.0* — arXiv:2503.20020, Mar 2025. https://arxiv.org/abs/2503.20020
- *OpenVLA* — arXiv:2406.09246, Jun 2024. https://arxiv.org/abs/2406.09246
- *OpenVLA-OFT: Fine-Tuning Vision-Language-Action Models* — arXiv:2502.19645, Feb 2025. https://arxiv.org/abs/2502.19645
- *Octo: An Open-Source Generalist Robot Policy* — arXiv:2405.12213, May 2024. https://arxiv.org/abs/2405.12213
- *CrossFormer: Scaling Cross-Embodied Learning* — arXiv:2408.11812, Aug 2024. https://arxiv.org/abs/2408.11812
- *HIL-SERL: Human-in-the-Loop Sample-Efficient Robotic RL* — arXiv:2410.21845, Oct 2024. https://arxiv.org/abs/2410.21845
- *SimpleVLA-RL* — arXiv:2509.09674, Sep 2025. https://arxiv.org/abs/2509.09674
- *iRe-VLA* — arXiv:2501.16664, Jan 2025. https://arxiv.org/abs/2501.16664
- *DreamGen* — arXiv:2505.12705, May 2025. https://arxiv.org/abs/2505.12705
- *ForceVLA* — arXiv:2505.22159, May 2025. https://arxiv.org/abs/2505.22159
- *Tactile-VLA* — arXiv:2507.09160, Jul 2025. https://arxiv.org/abs/2507.09160
- *AnySkin* — arXiv:2409.08276, Sep 2024. https://arxiv.org/abs/2409.08276
- *Sparsh* — arXiv:2410.24090, Oct 2024. https://arxiv.org/abs/2410.24090
- *Re-Mix: Optimizing Data Mixtures for Large Scale Imitation Learning* — arXiv:2408.14037, Aug 2024. https://arxiv.org/abs/2408.14037
- *CUPID* — arXiv:2506.19121, Jun 2025. https://arxiv.org/abs/2506.19121
- *What Matters in Learning from Large-Scale Datasets for Robot Manipulation* — arXiv:2506.13536, Jun 2025. https://arxiv.org/abs/2506.13536
- *GELLO* — arXiv:2309.13037, Sep 2023. https://arxiv.org/abs/2309.13037
- *AirExo-2* — arXiv:2503.03081, Mar 2025. https://arxiv.org/abs/2503.03081
- *Mobile ALOHA* — arXiv:2401.02117, Jan 2024. https://arxiv.org/abs/2401.02117
- *QuantVLA* — arXiv:2602.20309, Feb 2026. https://arxiv.org/abs/2602.20309
- *ActQuant* — arXiv:2605.24011, May 2026. https://arxiv.org/abs/2605.24011
- *SnapFlow* — arXiv:2604.05656, Apr 2026. https://arxiv.org/abs/2604.05656
- *VLA-Perf* — arXiv:2602.18397, Feb 2026. https://arxiv.org/abs/2602.18397
- *FASTER* — arXiv:2603.19199, Mar 2026. https://arxiv.org/abs/2603.19199
- *N-SCORE* — arXiv:2603.13616, Mar 2026. https://arxiv.org/abs/2603.13616
- *RoboWorld* — arXiv:2607.01060, Jul 2026. https://arxiv.org/abs/2607.01060
- *RoboReward* — arXiv:2601.00675, Jan 2026. https://arxiv.org/abs/2601.00675
- *AgiBot World* — arXiv:2503.06669, Mar 2025. https://arxiv.org/abs/2503.06669
- *DROID* — arXiv:2403.12945, Mar 2024. https://arxiv.org/abs/2403.12945
- *FineWeb* — arXiv:2406.17557, Jun 2024. https://arxiv.org/abs/2406.17557

**Independent and critical**

- *A Careful Examination of Large Behavior Models for Multitask Dexterous Manipulation* — arXiv:2507.05331, Jul 2025. https://arxiv.org/abs/2507.05331
- *Unmasking the Illusion of Embodied Reasoning in Vision-Language-Action Models* (BeTTER) — arXiv:2604.18000, Apr 2026. https://arxiv.org/abs/2604.18000
- *When Vision Overrides Language* — arXiv:2602.17659, Feb 2026. https://arxiv.org/abs/2602.17659
- *LIBERO-Plus* — arXiv:2510.13626, Oct 2025. https://arxiv.org/abs/2510.13626
- *LIBERO-PRO* — arXiv:2510.03827, Oct 2025. https://arxiv.org/abs/2510.03827
- *RoboChallenge* — arXiv:2510.17950, Oct 2025. https://arxiv.org/abs/2510.17950
- *On the Statistical Significance of Robot Manipulation Benchmarks* — arXiv:2606.04233, Jun 2026. https://arxiv.org/abs/2606.04233
- *Chinchilla Scaling: A Replication Attempt* — arXiv:2404.10102, Apr 2024. https://arxiv.org/abs/2404.10102
- *Neural Scaling Laws in Robotics* — arXiv:2405.14005, May 2024. https://arxiv.org/abs/2405.14005

**Company blogs and press — marketing class, labelled as such wherever cited**

- Figure AI, *Helix: A Vision-Language-Action Model for Generalist Humanoid Control*, Feb 2025. https://www.figure.ai/news/helix
- Figure AI, *Helix 02*. https://www.figure.ai/news/helix-02
- Figure AI, *Figure 03 battery development*. https://www.figure.ai/news/f-03-battery-development
- NVIDIA GEAR Lab, *GR00T N1.5*, Jun 2025. https://research.nvidia.com/labs/gear/gr00t-n1_5/
- NVIDIA, *Jetson AI Lab: openpi on Thor*. https://www.jetson-ai-lab.com/tutorials/openpi_on_thor/
- Google DeepMind, *AutoRT*, Jan 2024. https://auto-rt.github.io/
- Google DeepMind, *RoboCat*, Jun 2023. https://deepmind.google/blog/robocat-a-self-improving-robotic-agent/
- Google DeepMind, *Genie 3*, Aug 2025. https://deepmind.google/blog/genie-3-a-new-frontier-for-world-models/ — parameter count **undisclosed**; the widely circulated figure is secondary-source only and is not used in this post.
- Hugging Face, *SmolVLA*, Jun 2025. https://huggingface.co/blog/smolvla
- Dyna Robotics, *DYNA-1*, 2025. https://www.dyna.co/research/dyna-1
- Fortune, *Tesla is hiring people to train Optimus*, Aug 2024. https://www.fortune.com/2024/08/19/tesla-robot-hiring-workers-optimus-training-ai
- Rest of World, *China's robot training centers*, Jan 2026. https://restofworld.org/2026/china-robots-training-centers-workers/

**Standards, datasheets, and source trees**

- ISO 10218-1:2025 and ISO 10218-2:2025, *Robotics — Safety requirements*, Feb 2025.
- ISO/TS 15066:2016, *Robots and robotic devices — Collaborative robots*. Per-region pressure tables are paywalled and were **not** verified for this post.
- ISO 25785-1, *Dynamically stable industrial mobile robots* — Working Draft.
- ANSI/A3 R15.06-2025, approved Aug 2025.
- NVIDIA Jetson AGX Thor and Jetson AGX Orin module datasheets. https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/
- NVIDIA H100 SXM datasheet. https://www.nvidia.com/en-us/data-center/h100/
- Franka Research 3 and Trossen ViperX 300 specifications.
- 1X NEO specifications. https://www.1x.tech/neo
- `openpi` — https://github.com/Physical-Intelligence/openpi (`websocket_policy_server.py`, `aloha_policy.py`)
- `Isaac-GR00T` — https://github.com/NVIDIA/Isaac-GR00T (`getting_started/hardware_recommendation.md`)
