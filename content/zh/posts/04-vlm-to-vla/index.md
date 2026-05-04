---
title: "从 VLM 到 VLA：视觉语言模型如何变成机器人 Policy"
date: 2026-05-04
draft: true
tags: ["vla", "vlm", "openvla", "smolvla"]
series: ["VLA Tutorial"]
weight: 4
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "拆解 VLM 到 VLA 的关键改造：视觉 backbone、语言 backbone、state/proprioception 注入、action head 与 finetune recipe。"
---

## 写作目标

解释 VLA 不是“VLM + 机器人数据”这么简单。机器人动作空间、state/proprioception、control frequency 和 evaluation protocol 都要求对 VLM 做结构性改造。

## 文章结构

1. VLM 原本解决什么问题，为什么不能直接控制机器人。
2. OpenVLA：7B open-source VLA 的结构、数据和 fine-tuning 价值。
3. SmolVLA：小模型、社区数据、affordable training、async inference。
4. action output 的几种设计：continuous head、discrete action tokens、chunk。
5. fine-tuning recipe：LoRA、quantization、robot-specific normalization。
6. VLA 与传统 policy baseline 的公平比较。

## 必画 Excalidraw 图

- `image encoder + language model + robot state -> action head`
- OpenVLA / SmolVLA / ACT / Diffusion Policy 对比表
- finetune pipeline：pretrained checkpoint -> robot dataset -> adapter/head -> rollout

## 参考入口

- [OpenVLA](https://huggingface.co/papers/2406.09246)
- [SmolVLA](https://huggingface.co/papers/2506.01844)

