# 07 VLA World Model Frontier Outline

## Goal

Analyze whether future prediction should be an auxiliary objective, a planning module, or part of the policy itself.

## Core Diagram

`static/diagrams/vla-world-model-fusion.svg`

## Systems To Cover

- WorldVLA.
- RynnVLA-002.
- MMaDA-VLA.
- Gemini Robotics + embodied reasoning.
- Genie 3 as an interactive world model.

## Evaluation Questions

- Does future prediction improve action success, or only representation quality?
- Is future prediction latent, pixel, reward, or structured state?
- Does action generation condition on predicted future?
- Is the system evaluated in closed-loop robotics or only proxy generation metrics?

