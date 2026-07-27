---
title: "从 VLM 到 VLA：视觉语言模型如何变成机器人 Policy"
date: 2026-04-29T09:00:00+08:00
draft: true
tags: ["vla", "vlm", "openvla", "smolvla"]
series: ["VLA Tutorial"]
weight: 6
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "拆解 VLM 到 VLA 的关键改造：视觉 backbone、语言 backbone、state/proprioception 注入、action head 与 finetune recipe。"
---

## Motivation

VLM 擅长回答图文问题，但机器人需要连续行动。VLA 的核心问题就是：如何把一个原本输出 text token 的模型，改造成能稳定输出 robot action 的 policy。

这不是简单地在 VLM 后面接一个 MLP。机器人动作有单位、频率、约束、平滑性和安全边界；机器人 observation 还包含 proprioception、历史动作、相机位姿等非互联网图文数据。VLA 的结构改造必须服务于这些控制问题。

## VLM 到 VLA 的四个改造点

第一，输入要扩展。VLM 通常吃 image 和 text；VLA 还需要 robot state，例如 joint position、end-effector pose、gripper state、历史动作。没有 proprioception，模型很难知道当前机器人自身状态。

第二，输出要改变。VLM 输出 text token；VLA 输出 action。动作可以是连续向量、离散 token、action chunk、diffusion trajectory 或 flow matching trajectory。

第三，训练数据要改变。VLM 用图文对训练；VLA 需要 robot demonstrations：

$$
(\text{image}_{1:T}, \text{instruction}, \text{state}_{1:T}, \text{action}_{1:T})
$$

第四，评估要改变。VLM 可以用 accuracy 或 benchmark score；VLA 必须看 closed-loop success。离线 action prediction loss 低，不代表机器人真的能完成任务。

## OpenVLA：开源 Generalist Baseline

OpenVLA 的重要性在于它给出了一个可研究、可 fine-tune 的 open VLA baseline。它继承 VLM 的视觉语言能力，并把动作离散化为 token，让 action prediction 可以进入 language-modeling 框架。

这种设计的优点是工程路径清楚：动作 token 可以用 cross entropy 训练，可以复用 LLM/VLM 的基础设施。缺点也明确：连续控制被离散化后会有量化误差，tokenization 设计会影响动作精度和泛化。

读 OpenVLA 时，不要只看 7B 参数。更关键的是：

- robot dataset 如何混合；
- action 如何 tokenized；
- fine-tune 时哪些模块更新；
- evaluation task 是否覆盖新物体、新背景、新指令；
- latency 是否满足真实闭环控制。

## SmolVLA：为什么小模型重要

SmolVLA 的 motivation 很现实：机器人不是云端 chatbot，不能无限依赖巨大模型。一个 VLA 如果训练、部署、迭代成本太高，就很难成为研究和产品中的基础组件。

SmolVLA 强调 affordable training、efficient inference 和 asynchronous execution。它把 VLA 从“大模型展示能力”拉回到“能不能让普通实验室和 edge device 跑起来”。这也是未来 VLA 的关键方向：性能不是只看成功率，还要看速度、显存、功耗和可复现性。

## Fine-Tuning Recipe

VLA fine-tuning 通常包含：

1. 选择 pretrained VLM/VLA checkpoint；
2. 准备 robot dataset，并统一 observation/action schema；
3. 做 action normalization 或 tokenization；
4. 训练 adapter、LoRA、action head 或部分 backbone；
5. 在 closed-loop 环境里评估；
6. 根据失败模式调整数据和动作表示。

这里最容易被忽略的是 normalization。机器人动作的量纲差异很大，如果 action scale 处理不好，模型 loss 看起来正常，rollout 也可能完全失败。

## 与传统 Policy 的公平比较

VLA 不应该只和其他 VLA 比，也应该和 ACT、Diffusion Policy、BC-Transformer 比。公平比较至少要控制：

- demonstration 数量；
- observation 类型；
- action horizon；
- policy inference frequency；
- rollout budget；
- task split。

如果 VLA 在语言泛化任务上更强，这是它的优势；如果在固定 manipulation task 上不如小模型 policy，也不是意外。VLA 的价值主要在 compositional language grounding、multi-task transfer 和 broader prior，而不是所有场景都替代专用 policy。

## 本文结论

从 VLM 到 VLA 的关键不是“加一个 action head”，而是把视觉语言表示接入 robot state、action representation、fine-tuning recipe 和 closed-loop evaluation。OpenVLA 给出开源 generalist baseline，SmolVLA 代表 efficient VLA 路线。二者合起来说明：VLA 既是模型问题，也是数据、动作和系统问题。
