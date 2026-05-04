# 05 Action Representation Outline

## Goal

Make action representation the central modeling problem of VLA.

## Core Diagrams

- Single-step action vs chunk vs token vs diffusion vs flow.
- Latency vs horizon trade-off.
- Action tokenizer encode/decode path.

## Systems To Cover

- ACT.
- Diffusion Policy.
- OpenVLA action tokens.
- π0 and π0-FAST.

## Key Insight

Action representation changes not only the output layer, but also the training objective, inference loop, evaluation failure modes, and deployment cost.

