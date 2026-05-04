---
title: "非 VLA 的 Embodied Imitation Pipeline：先看懂机器人学习工程栈"
date: 2026-05-04
draft: true
tags: ["imitation-learning", "lerobot", "act", "diffusion-policy", "robomimic"]
series: ["VLA Tutorial"]
weight: 3
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "在进入大 VLA 之前，先用 LeRobot、ACT、Diffusion Policy、robomimic 建立 dataset、training、rollout、evaluation 的工程直觉。"
---

## 写作目标

让读者理解很多 VLA repo 本质上是在已有 imitation / offline policy pipeline 上叠加 language 和更大的 backbone。先看懂基础 pipeline，后面读 SmolVLA/OpenVLA 才不会只看到模型规模。

## 文章结构

1. 一个 embodied policy repo 的 7 个核心模块。
2. LeRobot dataset 格式：episode、observation、action、metadata。
3. ACT：action chunking 为什么能缓解 compounding error 和 latency。
4. Diffusion Policy：为什么把 action trajectory 当作生成对象。
5. robomimic：dataset / algo / eval 的模块化设计。
6. PushT 作为最小闭环实验。

## 必画 Excalidraw 图

- `dataset -> dataloader -> policy -> loss -> checkpoint -> rollout -> success metric`
- ACT action chunk 与 single-step action 的对比
- open-loop evaluation 与 closed-loop rollout 的差异

## 最小实验

目标是跑通 `LeRobot + ACT + PushT`，并记录安装、数据格式、训练入口、评估指标和 Mac mini 瓶颈。

