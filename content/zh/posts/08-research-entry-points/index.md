---
title: "博士研究切入点：从复现 Baseline 到提出可验证问题"
date: 2026-05-04
draft: true
tags: ["research", "vla", "experiments", "apple-silicon"]
series: ["VLA Tutorial"]
weight: 8
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "把 VLA 学习路线收束成可执行研究问题：小模型部署、动作表征、小数据 regime、future prediction auxiliary objective。"
---

## 写作目标

帮助读者从“读懂别人做了什么”进入“我能做一个什么小而清晰的问题”。研究切入点必须绑定 baseline、变量、evaluation protocol 和资源约束。

## 候选方向

1. 小模型 VLA 在 Apple Silicon / edge device 上的部署边界。
2. action representation 对泛化、稳定性和 latency 的影响。
3. language grounding 对 sim manipulation 到 real transfer 的贡献。
4. future prediction auxiliary objective 在小 benchmark 上是否有效。
5. ACT / Diffusion Policy / SmolVLA 在 small-data regime 下的比较。

## 最小研究协议

1. 选一个小环境：PushT、LIBERO subset、ManiSkill CPU task。
2. 复现一个 baseline：ACT 或 Diffusion Policy。
3. 改一个变量：action chunk size、tokenization、future prediction loss、quantization。
4. 固定 evaluation：same seeds、same demos、same rollout budget。
5. 报告失败模式：latency、drift、object grounding、stage hallucination、recovery failure。

## 必画 Excalidraw 图

- research loop：baseline -> single variable -> controlled eval -> failure analysis
- small-data VLA 实验矩阵
- Apple Silicon 可跑与不可跑边界

