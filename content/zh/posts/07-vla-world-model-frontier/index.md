---
title: "VLA + World Model 前沿：Future Prediction 是否应该成为 Policy 的一部分？"
date: 2026-05-04
draft: true
tags: ["vla", "world-model", "frontier", "gemini-robotics", "genie"]
series: ["VLA Tutorial"]
weight: 7
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "分析 WorldVLA、RynnVLA、MMaDA-VLA、Gemini Robotics、Genie 等方向，理解 VLA 与 world model 的融合路径。"
---

## 写作目标

判断 VLA + world model 融合到底解决什么问题：sample efficiency、generalization、long-horizon consistency、reasoning/foresight，还是只是多任务预训练带来的 representation gain。

{{< excalidraw src="/diagrams/vla-world-model-fusion.svg" source="/diagrams/vla-world-model-fusion.excalidraw" alt="VLA and world model fusion patterns" caption="VLA 与 world model 的融合方式：future prediction 可能是辅助目标，也可能进入 action generation 主链路。" >}}

## 文章结构

1. 融合方式 A：VLA 加 future prediction auxiliary objective。
2. 融合方式 B：joint action + image generation。
3. 融合方式 C：world model 作为 learned simulator / planner。
4. WorldVLA：autoregressive action world model。
5. MMaDA-VLA：discrete diffusion 统一 multimodal understanding、future generation 与 action chunk。
6. Gemini Robotics：VLA + embodied reasoning 的双模型路线。
7. Genie 3：interactive world model 对 agent training 的潜在影响。

## 必画 Excalidraw 图

- auxiliary future prediction vs integrated action-world generation
- Gemini Robotics 的 ER / VLA 双模型分工
- learned simulator 如何影响 policy improvement

## 参考入口

- [WorldVLA](https://huggingface.co/papers/2506.21539)
- [MMaDA-VLA](https://huggingface.co/papers/2603.25406)
- [Gemini Robotics](https://deepmind.google/models/gemini-robotics/)
- [Genie 3](https://deepmind.google/models/genie/)

