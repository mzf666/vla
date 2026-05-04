---
title: "VLA + World Model 前沿：Future Prediction 是否应该成为 Policy 的一部分？"
date: 2026-04-26T09:00:00+08:00
draft: false
tags: ["vla", "world-model", "frontier", "gemini-robotics", "genie"]
series: ["VLA Tutorial"]
weight: 9
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "分析 WorldVLA、RynnVLA、MMaDA-VLA、Gemini Robotics、Genie 等方向，理解 VLA 与 world model 的融合路径。"
---

## Motivation

到 2025-2026 年，VLA 的前沿已经不只是“更大的 VLM backbone”。真正值得关注的趋势是：动作生成、未来预测、embodied reasoning、on-device deployment 正在合流。VLA 不再只是 language-conditioned behavior cloning，而开始变成具备 foresight 和系统约束的 embodied foundation model。

{{< excalidraw src="/diagrams/vla-world-model-fusion.svg" source="/diagrams/vla-world-model-fusion.excalidraw" alt="VLA and world model fusion patterns" caption="VLA 与 world model 的融合方式：future prediction 可能是辅助目标，也可能进入 action generation 主链路。" >}}

## OpenVLA 与 SmolVLA：Open Generalist Baseline

OpenVLA 的意义是开源 generalist VLA baseline。它让研究者可以在一个公开模型上做 fine-tune、动作表示、数据混合和部署实验。它不是最终答案，但它提供了共同参照。

SmolVLA 的意义是把 VLA 推向 affordable 和 efficient。它提醒我们：如果一个方法只能在巨型 GPU 集群上训练和部署，对大多数机器人实验室就不是真正可用的基础设施。小模型、异步推理、社区数据，会成为 VLA 普及的关键。

## π0 与 π0-FAST：连续动作生成路线

π0 把 VLA 的动作输出建模为 flow matching 过程，强调连续动作序列生成。它代表了一条不同于 action token 的路线：不要把控制问题完全塞进离散 token，而是保留连续控制结构。

π0-FAST / action tokenizer 方向则强调 serving speed。它说明即便动作生成方法能力强，速度仍然是第一等问题。机器人不等人，action model 必须跟控制频率协同设计。

## GR00T N1：Humanoid Foundation Model

NVIDIA GR00T N1 把 VLA 放进 humanoid robot foundation model 语境。humanoid 比桌面 manipulation 更复杂：动作维度更高、接触更复杂、任务更长、真实数据更贵。因此 synthetic data、simulation、multi-embodiment learning、policy distillation 都变得重要。

读 GR00T 这类 report 时要区分两件事：一是模型结构本身，二是数据 flywheel 和仿真基础设施。很多能力来自系统级数据生产和评估，不只是单个网络 architecture。

## Gemini Robotics：VLA + Embodied Reasoning

Gemini Robotics 的重要信号是把 embodied reasoning 和 VLA action model 放在一起讨论。机器人不只要执行低级动作，还要理解长指令、空间约束、任务阶段和安全常识。

这类系统往往有两层：

- 高层 embodied reasoning：理解任务、分解步骤、处理语义和空间关系；
- 低层 VLA policy：把当前 observation 和 instruction 转成动作。

这种分层不是倒退，而是现实工程选择。纯端到端模型很优雅，但长任务、安全约束和可解释性常常需要显式结构。

## Gemini Robotics On-Device：前沿不只在云端

On-device 方向说明 VLA 的 SOTA 不只是 benchmark success，也包括部署边界。低延迟、本地推理、隐私、断网可用，都会影响机器人是否能进入真实场景。

如果一个 VLA 需要远程 GPU，每次动作都依赖网络，它的应用边界会很窄。on-device VLA 代表的是“机器人自己能不能持续运行”的问题。

## WorldVLA / RynnVLA / MMaDA-VLA：Action 与 Future 一起建模

WorldVLA 这类工作尝试把 action generation 和 future generation 统一到同一个 autoregressive 或 generative objective 中。直觉是：如果模型能预测执行动作后的未来，它应该更懂动作后果。

MMaDA-VLA 等 diffusion-style 工作也在类似方向上探索：同时处理 multimodal understanding、future observation generation 和 action chunk generation。

这里最关键的评价问题是：

1. future prediction 是否真的进入 action decision；
2. improvement 是否体现在 closed-loop success；
3. 预测空间是 pixel、latent 还是 task state；
4. 是否只是 auxiliary loss 带来的 representation regularization。

## Genie 3：Interactive World Model 的长期意义

Genie 3 代表 large-scale interactive world model 方向。它不一定直接是机器人 policy，但它提示一个未来可能：如果 agent 可以在足够丰富的 learned world 中交互、探索、失败、重试，那么 policy learning 的数据瓶颈可能被改变。

对机器人来说，关键仍然是 sim-to-real。一个强 world model 如果不能保留真实物理接触、机器人动力学和任务约束，就只能作为预训练或规划辅助。它是否能成为 robot learning 的核心基础设施，还需要闭环验证。

## 当前 SOTA 的读法

读 2025-2026 VLA report 时，不要只看 demo video。建议按五个维度拆：

1. 模型：VLM backbone、action representation、是否有 world model；
2. 数据：真实数据、仿真数据、互联网数据、synthetic data；
3. 训练：BC、diffusion、flow matching、autoregression、joint objective；
4. 系统：latency、on-device、async inference、quantization；
5. 评估：closed-loop success、泛化 split、真实机器人、失败模式。

## 本文结论

最新 VLA 前沿正在从“VLM 输出动作”走向“动作、未来、推理、部署协同设计”。OpenVLA 给出开源基线，SmolVLA 推向高效普及，π0 强调连续动作生成，GR00T 和 Gemini Robotics 推向 embodied foundation model，WorldVLA / MMaDA-VLA / Genie 把 world modeling 带回中心。真正的 SOTA 不会只由模型大小决定，而会由数据、动作表征、未来建模和系统部署共同决定。
