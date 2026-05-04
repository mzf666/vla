---
title: "问题框架：Policy、World Model 和 VLA 的边界"
date: 2026-05-04
draft: false
tags: ["vla", "world-model", "policy", "taxonomy"]
series: ["VLA Tutorial"]
weight: 1
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "先把 policy、world model、VLA 放进同一个输入输出坐标系，避免一开始就被大模型名词带偏。"
prerequisites:
  - "知道 MDP、policy、model-based RL 的基本定义"
  - "理解 imitation learning / behavior cloning 的基本形式"
---

## Motivation

进入 VLA 之前，最重要的不是记住 OpenVLA、π0、GR00T、Gemini Robotics 这些名字，而是先建立一个稳定的分类坐标系。否则你会在 paper 里看到大量相似词：policy、world model、foundation model、VLM、VLA、action model、video model、embodied reasoning model，但不知道它们到底差在哪里。

这个系列使用一个简单标准：**看输入、输出、是否预测未来、是否原生使用语言、训练目标是什么。**

{{< excalidraw src="/diagrams/vla-boundary-taxonomy.svg" source="/diagrams/vla-boundary-taxonomy.excalidraw" alt="Policy world model and VLA taxonomy" caption="Policy、world model、VLA 的边界：不要按模型名字分类，要按输入输出和训练目标分类。" >}}

## Policy：从 observation 到 action

最基础的对象是 policy：

$$
\pi(a_t \mid o_t, h_t, g)
$$

这里的 $o_t$ 是当前 observation，$h_t$ 可以是历史信息，$g$ 可以是 goal 或 language instruction。policy 的核心职责是输出 action，而不是预测未来世界本身。

在 embodied imitation learning 里，最常见的训练目标是 behavior cloning：

$$
\min_\theta \mathbb{E}_{(o, a) \sim D}[-\log \pi_\theta(a \mid o)]
$$

如果 action 是连续向量，可以做 MSE 或 Gaussian likelihood。如果 action 是 sequence 或 chunk，可以把它当作 trajectory prediction。如果 action 是 token，可以做 cross entropy。如果 action 由 diffusion / flow model 生成，训练目标会变成 denoising 或 vector field matching。

这类方法包括 BC、BC-Transformer、ACT、Diffusion Policy 等。它们不一定使用语言，也不一定使用大模型。它们是理解 VLA 的地基，因为 VLA 最后仍然必须解决 policy learning。

## World Model：从当前与动作预测未来

world model 的职责不是直接输出动作，而是学习环境动力学：

$$
p(s_{t+1}, r_t, d_t \mid s_t, a_t)
$$

在现代 deep RL 里，$s_t$ 往往不是真实 state，而是从图像压缩出来的 latent state。Dreamer 系列的核心直觉就是：不要在 pixel space 里做规划，而是在 latent dynamics 中做 imagination rollout，然后用 imagined trajectory 训练 actor 和 critic。

一个典型 world model 需要回答：

- 它预测的是 latent、pixel、reward、done，还是其中的组合；
- 它的 latent 是否足够保留 action-relevant information；
- rollout 变长以后 error accumulation 如何控制；
- policy improvement 是通过 planning、actor-critic，还是辅助 loss 间接实现。

所以 world model 不等于视频生成模型。高质量视频可能有用，但机器人更关心的是 action-conditioned future 是否对决策有帮助。

## VLA：语言条件下的 embodied policy

VLA 可以先粗略理解成：

$$
\pi(a_{t:t+H} \mid image_t, text, proprio_t, history)
$$

它的特殊之处不是“用了视觉”和“用了动作”，因为普通机器人 policy 也用视觉和动作。VLA 的关键是把 natural language instruction 放进 policy，并且通常继承 VLM 的视觉语言表示能力。

这带来三个新问题。

第一，VLM 表示如何接到 action space。VLM 原本输出文本 token，不天然输出连续控制向量。因此需要 action head、action tokenizer、diffusion head、flow head 或 discretization recipe。

第二，语言到底提供什么。语言可能只是 task id 的自然表达，也可能承载 object grounding、spatial relation、long-horizon intent 和 interactive correction。不同 benchmark 下这几个作用差别很大。

第三，闭环 latency 如何处理。VLM backbone 往往很重，如果每个 control step 都完整跑一遍模型，机器人会太慢。因此 action chunking、async inference、model compression、on-device serving 都变成一等问题。

## 三者的交集

这三类对象并不是互斥的。

一个 policy 可以不使用 world model，也可以使用 learned dynamics 做 planning。一个 world model 可以不输出 action，也可以在 unified architecture 里同时输出 future image 和 action。一个 VLA 可以只是 language-conditioned BC，也可以带 future prediction auxiliary objective。

最容易混淆的是 VLA + world model 的新工作。比如 WorldVLA 把 action generation 和 image generation 放进统一自回归框架，声称二者可以互相增强。这里需要问的不是“它是不是 world model”，而是：

- future prediction 的输出具体是什么；
- future prediction 的 loss 是否影响 action policy；
- action 生成是否依赖 predicted future；
- improvement 来自更强表示、更好 planning，还是额外数据正则化。

## 一个实用分类表

| 方法类型 | 输入 | 输出 | 是否语言条件 | 是否预测未来 | 典型训练目标 |
|---|---|---|---|---|---|
| BC policy | image/state | action | 通常否 | 否 | imitation loss |
| ACT | image/state/history | action chunk | 可选 | 否 | chunk reconstruction |
| Diffusion Policy | image/state | action trajectory | 可选 | 否 | denoising |
| Dreamer | observation/action | latent/reward/value/action | 否 | 是 | reconstruction + RL |
| OpenVLA | image/text/state | action tokens/action | 是 | 通常否 | VLA pretraining + finetune |
| SmolVLA | image/text/state | action chunk | 是 | 通常否 | efficient VLA training |
| π0 | image/text/state | continuous action sequence | 是 | 否 | flow matching |
| WorldVLA | image/text/action | future image + action | 是 | 是 | autoregressive joint objective |

## 本文的结论

VLA 不是一个替代所有 robot learning 的魔法对象。它是 language-conditioned embodied policy，在工程上继承 imitation learning pipeline，在表示上借用 VLM，在前沿上开始和 world model 融合。

后续文章会沿着这条线展开：先理解 world model 为什么重要，再跑通非 VLA 的 embodied imitation pipeline，然后再看 VLM 到 VLA 的改造点。只有这样，读 OpenVLA、SmolVLA、π0、GR00T、Gemini Robotics、WorldVLA 这些工作时，才不会只剩下“模型更大、数据更多”的印象。

