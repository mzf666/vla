---
title: "Action Representation：连续动作、Action Chunk、Token、Diffusion 与 Flow"
date: 2026-05-04
draft: true
tags: ["action-representation", "diffusion-policy", "flow-matching", "pi0"]
series: ["VLA Tutorial"]
weight: 5
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "动作表征是 VLA 的核心瓶颈之一：同样是 image+language 输入，不同 action representation 会改变训练目标、推理延迟和泛化能力。"
---

## 写作目标

把 action representation 当作 VLA 的中心问题，而不是实现细节。不同动作表征对应不同的 inductive bias、loss、inference cost 和 closed-loop 行为。

## 文章结构

1. 单步 continuous action：简单但 horizon 短。
2. Action chunk：把短轨迹作为 policy 输出，降低控制频率压力。
3. Discretized action token：复用 language-modeling 工具链，但需要处理量化误差。
4. Diffusion action：生成整个 action trajectory，适合多峰行为。
5. Flow matching：π0 的关键设计，连续控制与高频动作的折中。
6. FAST/action tokenizer：为什么 serving speed 是 robotics 一等问题。

## 必画 Excalidraw 图

- 五种 action representation 的输入输出对照
- inference latency 与 action horizon 的 trade-off
- action tokenization 的 encode/decode 流程

## 参考入口

- [π0](https://huggingface.co/papers/2410.24164)

