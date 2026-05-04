---
title: "Action Representation：连续动作、Action Chunk、Token、Diffusion 与 Flow"
date: 2026-04-28T09:00:00+08:00
draft: false
tags: ["action-representation", "diffusion-policy", "flow-matching", "pi0"]
series: ["VLA Tutorial"]
weight: 7
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "动作表征是 VLA 的核心瓶颈之一：同样是 image+language 输入，不同 action representation 会改变训练目标、推理延迟和泛化能力。"
---

## Motivation

VLA 最容易被误解的地方，是把动作输出看成模型最后一层的实现细节。实际上，action representation 决定了训练目标、推理延迟、控制频率、泛化能力和失败模式。它是 VLA 的核心建模问题之一。

同样是 image + language + state 输入，模型可以输出单步连续动作、action chunk、离散 action token、diffusion trajectory 或 flow matching trajectory。每一种选择都在回答同一个问题：机器人动作到底应该被当作 regression、sequence modeling、generation，还是 dynamical flow？

## 单步连续动作

最直接的形式是：

$$
a_t = f_\theta(o_t, g)
$$

这里 \(a_t\) 可以是 joint velocity、end-effector delta pose、gripper command 等。优点是简单、推理快、控制闭环清楚。缺点是 horizon 短，每一步都重新决策，容易受 compounding error 和 perception noise 影响。

单步动作适合低延迟、小模型、任务简单的 setting。但对复杂 manipulation，局部动作必须组成稳定轨迹，单步回归往往不够。

## Action Chunk

Action chunk 输出一段动作：

$$
a_{t:t+H} = f_\theta(o_t, g)
$$

它的动机是把短期控制轨迹一次性生成，减少模型推理频率，同时让模型学习局部时序结构。ACT 和很多 VLA 系统都使用类似思想。

chunk 的关键超参是 horizon \(H\)。\(H\) 太小，无法减少 latency；\(H\) 太大，遇到扰动时不够 reactive。真实系统常用 receding horizon：每次只执行 chunk 前几步，然后重新感知和预测。

## Action Token

OpenVLA 代表了一类 action tokenization 思路：把连续动作离散化成 token，让 VLA 可以像语言模型一样预测 action token。

优点很明显：

- 可以复用 autoregressive training；
- action 和 language token 可以进入统一序列；
- cross entropy loss 稳定；
- 工程上接近 LLM/VLM。

代价也明显：

- 离散化带来量化误差；
- token vocabulary 和 binning 会影响精度；
- autoregressive action token 可能增加 latency；
- 连续控制平滑性需要额外处理。

因此 action tokenization 不是“把动作变成词”这么简单，而是一个控制精度和序列建模能力之间的 trade-off。

## Diffusion Action

Diffusion Policy 把未来动作轨迹看成一个从噪声逐步 denoise 出来的样本。它适合多峰动作分布：比如绕过障碍可以从左边也可以从右边，MSE 会平均出一条不可行轨迹，diffusion 可以采样其中一种模式。

缺点是推理慢。每次动作生成需要多个 denoising step，对机器人闭环控制不友好。可以通过少步采样、distillation、chunking 和 async execution 缓解，但系统复杂度会上升。

## Flow Matching 与 π0

π0 使用 flow matching 生成连续动作序列。它可以看成在 diffusion 和直接回归之间的一种折中：仍然建模从噪声到动作的连续生成过程，但训练和采样形式更适合高维连续动作。

π0 的重要性在于，它把 VLA 的动作输出从 token-centric 路线拉回连续控制路线。对机器人来说，连续性、平滑性和高频执行都很重要。flow matching 提供了一种更贴近 action trajectory 的建模方式。

## FAST：动作表征也是 Serving 问题

π0-FAST / action tokenizer 方向强调 faster serving。这个方向的 motivation 很明确：如果 VLA 能力很强但推理太慢，机器人闭环会失败。动作表征必须同时考虑学习效果和 serving cost。

因此评价 action representation 时，至少要看四个指标：

1. action accuracy；
2. closed-loop success；
3. inference latency；
4. robustness under perturbation。

## 本文结论

动作表征不是输出层细节，而是 VLA 的核心设计轴。continuous action 简单快速，chunk 改善短期轨迹和 latency，token 复用 LLM 工具链，diffusion 处理多模态，flow matching 兼顾连续控制和生成能力。读任何 VLA paper，都必须先问：它到底如何表示 action？
