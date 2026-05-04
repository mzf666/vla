---
title: "World Model 基础：Latent Dynamics、Imagination 与 Planning"
date: 2026-05-04
draft: true
tags: ["world-model", "model-based-rl", "dreamer"]
series: ["VLA Tutorial"]
weight: 2
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "从 Dreamer/TD-MPC 系列理解 world model 的真正目标：不是生成好看的视频，而是学到能帮助 policy improvement 的未来动力学。"
---

## 写作目标

解释 world model 在 robot learning 里的核心问题：如何把 observation 压缩成 latent state，如何在 latent space 做 action-conditioned rollout，以及 rollout error 为什么会限制 long-horizon planning。

## 文章结构

1. 为什么 world model 不等于 video generation。
2. RSSM / latent dynamics 的输入输出。
3. Dreamer 的 encoder、dynamics、reward、value、actor 如何组成闭环。
4. imagination rollout 和真实 rollout 的区别。
5. TD-MPC2 与 planning-oriented latent model。
6. 对 VLA 的启发：future prediction 应该如何影响 action？

## 必画 Excalidraw 图

- `encoder -> latent dynamics -> reward/value/decoder -> actor`
- real rollout 与 imagined rollout 的对照图
- error accumulation 随 horizon 增长的示意图

## 关键问题

- latent state 如果不能预测 pixel，但能预测 reward/action-relevant future，算不算好 world model？
- long-horizon consistency 是 architecture 问题、loss 问题，还是 data 问题？
- VLA 中加入 future prediction 时，到底是在做 representation regularization 还是 planning？

