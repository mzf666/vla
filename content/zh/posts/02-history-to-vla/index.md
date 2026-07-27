---
title: "发展脉络：从 Imitation Learning 到 VLA"
date: 2026-05-02T09:00:00+08:00
draft: true
tags: ["history", "vla", "robot-learning", "foundation-model"]
series: ["VLA Tutorial"]
weight: 3
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "梳理 VLA 出现前后的历史线索：imitation learning、language-conditioned policy、VLM、robot foundation model 和 generalist VLA 如何接到一起。"
---

## Motivation

VLA 不是突然出现的。它看起来像“大模型进入机器人”的结果，但真正的历史脉络更长：从 behavior cloning 到 action chunking，从 language-conditioned policy 到 vision-language pretraining，从 single-task robot policy 到 generalist robot foundation model。只有把这些线索接起来，OpenVLA、SmolVLA、π0、GR00T、Gemini Robotics 这些系统才不会显得像孤立名词。

这篇文章的任务是回答一个问题：为什么在 2020 年前后，机器人学习还主要围绕 imitation learning、offline RL、diffusion policy 和 simulation benchmark；而到 2024-2026 年，VLA 会成为一条主线？

## 第一阶段：Policy Learning 先解决“怎么动”

机器人学习最早的核心问题不是语言，而是控制。给定 observation，policy 要输出 action：

$$
\pi_\theta(a_t \mid o_t)
$$

这条线的代表是 behavior cloning、DAgger、offline RL、robomimic、ACT、Diffusion Policy。它们关心的是 demonstration dataset 怎么收集、动作如何表示、closed-loop rollout 是否成功、distribution shift 如何处理。

这里的 motivation 很朴素：很多 manipulation task 不需要复杂语言，只需要从图像和状态中稳定地产生动作。ACT 的 action chunking 解决的是高频控制与 compounding error，Diffusion Policy 解决的是多峰动作分布和轨迹生成。这些工作构成了 VLA 的底座。

## 第二阶段：语言开始进入 Policy

语言进入机器人有两个不同层次。

第一层是 task specification。比如 “pick up the red block” 可以看成比 one-hot task id 更自然的 goal 表达。此时语言主要告诉 policy 要做哪个任务。

第二层是 grounding。语言不只是标签，而是和物体、空间关系、动作阶段绑定。例如 “put the mug to the left of the plate” 需要模型理解 left-of、mug、plate 和最终状态。这个能力来自视觉语言模型，也来自具身数据。

这一步的重要性在于：如果 language 只是 task id，VLA 的价值有限；如果 language 能泛化到新物体、新组合、新指令，它才真正区别于普通 imitation policy。

## 第三阶段：VLM 提供通用视觉语言表示

VLM 的出现改变了机器人学习的假设。传统 policy 往往从机器人数据里从头学习视觉表示，但机器人数据昂贵、分布窄、规模小。VLM 则从互联网图文数据中学到 object、attribute、relation 和 instruction 的表示。

VLA 的核心赌注是：这些表示虽然不是为控制训练的，但可以被改造成 action-producing policy。也就是说，从：

$$
p_\theta(\text{text token} \mid \text{image}, \text{text})
$$

变成：

$$
\pi_\theta(a_{t:t+H} \mid \text{image}, \text{instruction}, \text{robot state})
$$

真正困难的部分是动作输出。语言模型天然输出 token，机器人需要连续动作、夹爪控制、关节速度或 end-effector pose。OpenVLA 选择把动作离散化为 token；π0 选择 flow matching 生成连续动作；SmolVLA 强调小模型和 async inference。

## 第四阶段：Robot Foundation Model

VLA 变成 foundation model 的关键不是“用了 Transformer”，而是跨任务、跨机器人、跨场景的数据混合。RT-1 / RT-2、Open X-Embodiment、OpenVLA、GR00T、Gemini Robotics 都在不同程度上追求 generalist robot policy。

这里的核心矛盾是 embodiment gap。不同机器人有不同 action space、camera placement、control frequency 和 morphology。语言和视觉表示可以共享，但动作空间很难完全共享。因此很多 VLA 工作都必须回答：

- action 是否被归一化到统一空间；
- 是否需要 robot-specific head；
- 是否能跨 embodiment transfer；
- evaluation 是否只在见过的机器人上进行。

## 第五阶段：从 VLA 到 VLA + World Model

最近的趋势是把 VLA 和 world model 合流。原因很直接：只输出动作的 policy 可能缺乏 foresight；只预测未来的 world model 又未必能直接控制机器人。WorldVLA、RynnVLA、MMaDA-VLA、Gemini Robotics、Genie 这类工作都在尝试让模型不仅知道“现在该做什么”，还知道“做了以后世界会怎样”。

但这条线必须谨慎读。future prediction 可能只是 auxiliary loss，帮助表示学习；也可能成为 action generation 的一部分；还可能作为 learned simulator 支持 planning。三者贡献不同，不能混在一起。

## 历史主线总结

VLA 的发展可以压缩成一条链：

1. behavior cloning 证明 demonstration 可以训练 policy；
2. ACT / Diffusion Policy 改善 action trajectory modeling；
3. language-conditioned policy 把 task specification 从 one-hot 变成自然语言；
4. VLM 把互联网规模视觉语言表示带入机器人；
5. OpenVLA / SmolVLA / π0 把 VLM 改造成 action model；
6. GR00T / Gemini Robotics 把 VLA 推向 humanoid、reasoning 和 on-device；
7. WorldVLA / Genie 把 future prediction 重新带回 embodied intelligence。

这就是本系列后续文章的顺序：先打牢 world model 和 imitation pipeline，再解释 VLM 如何变成 VLA，最后进入 action representation、系统部署和前沿融合。
