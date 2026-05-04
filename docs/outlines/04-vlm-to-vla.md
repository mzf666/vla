# 04 From VLM To VLA Outline

## Goal

Show how a pretrained VLM is modified to produce robot actions.

## Core Diagrams

- VLM backbone plus robot state and action head.
- OpenVLA vs SmolVLA comparison.
- Finetuning pipeline.

## Systems To Cover

- OpenVLA.
- SmolVLA.

## Questions To Answer

- Where does proprioception enter?
- How is action represented?
- What is finetuned: full backbone, LoRA, action head, or tokenizer?
- How is closed-loop success measured?

