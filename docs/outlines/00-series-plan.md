# VLA Tutorial Blog Plan

## Audience

The target reader is a Machine Learning PhD with reinforcement learning background but limited embodied AI engineering experience. The writing should assume familiarity with MDPs, offline RL, Transformers, and representation learning, but should explain robot datasets, action spaces, rollout evaluation, and latency constraints from first principles.

## Visual Rule

All architecture diagrams, algorithm diagrams, flowcharts, and taxonomy diagrams must be created as Excalidraw source files under `static/diagrams/*.excalidraw`, exported to SVG under `static/diagrams/*.svg`, and embedded through the `excalidraw` shortcode.

## Series Arc

| # | Article | Reader should understand after reading |
|---|---|---|
| 0 | Series roadmap | Why VLA must be learned together with world models and imitation pipelines |
| 1 | Problem frame | How to classify policy, world model, and VLA by inputs/outputs/objectives |
| 2 | World model basics | Why latent dynamics and imagination are central to model-based robot learning |
| 3 | Embodied imitation pipeline | What a robot policy repo actually contains before adding language |
| 4 | VLM to VLA | What has to change when a VLM becomes an action-producing policy |
| 5 | Action representation | Why action format is a first-class modeling decision |
| 6 | Systems and deployment | Why latency, async inference, compression, and on-device execution matter |
| 7 | VLA + world model frontier | How recent reports combine action generation with future prediction |
| 8 | Research entry points | How to turn the survey into controlled experiments and publishable questions |

## Latest Frontier Tracking As Of 2026-05-04

Open and report-style systems to track:

- OpenVLA: open 7B VLA baseline, June 2024.
- π0: VLA flow model for general robot control, October 2024.
- GR00T N1: NVIDIA humanoid foundation model report, March 2025.
- Gemini Robotics / Gemini Robotics On-Device: Google DeepMind VLA + embodied reasoning + local deployment, 2025.
- SmolVLA: affordable and efficient VLA, June 2025.
- WorldVLA: autoregressive action world model, June 2025.
- RynnVLA-002: unified VLA and world model, November 2025.
- QuantVLA and AC^2-VLA: 2026 efficient VLA deployment directions.
- MMaDA-VLA: 2026 diffusion-style unified VLA with future observation and action chunk generation.

## Writing Template

Each article should follow this structure:

1. Motivation: what confusion or failure mode this article resolves.
2. Formal frame: inputs, outputs, objective, evaluation.
3. Method intuition: explain with a concrete robot example.
4. Representative systems: 2-4 papers/repos, not a long catalog.
5. Engineering notes: what a reader can inspect or run locally.
6. Excalidraw diagram: one central diagram, not decorative.
7. Research questions: what remains unresolved.

## Draft Priority

1. Finish article 02 with DreamerV3 / TD-MPC2 diagrams.
2. Finish article 03 with LeRobot + ACT + PushT experiment notes.
3. Finish article 04 with OpenVLA and SmolVLA architecture comparison.
4. Finish article 05 with π0 and diffusion/action-token comparison.
5. Finish articles 06-08 after collecting experiment data and recent papers.

