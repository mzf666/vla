---
title: "VLA 深入浅出：系列路线图"
date: 2026-05-04T09:00:00+08:00
draft: false
tags: ["vla", "world-model", "robot-learning", "roadmap"]
series: ["VLA Tutorial"]
weight: 1
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "从 world model、embodied imitation 到 VLA 和 VLA+world model 前沿融合的系统学习路线。"
prerequisites:
  - "强化学习基础：MDP、policy、value function、offline RL"
  - "深度学习基础：Transformer、diffusion、representation learning"
---

## Motivation

VLA（Vision-Language-Action）现在很容易被讲成一句话：把视觉、语言和动作放进一个大模型里，让机器人听指令做事。这个说法没有错，但它太容易遮住真正困难的部分。

如果只从大模型角度看 VLA，你会以为核心问题是 backbone 更大、数据更多、语言能力更强。但在机器人里，模型最后必须输出连续控制信号，必须闭环执行，必须面对 latency、distribution shift、action representation、evaluation protocol 和 real-world safety。这些问题不是把 VLM 接一个 action head 就自然解决的。

这个系列的目标是把 VLA 放回它应该属于的位置：它不是孤立出现的模型类型，而是 **imitation learning、model-based RL、vision-language pretraining、robot systems** 四条线交汇后的产物。

{{< excalidraw src="/diagrams/vla-learning-map.svg" source="/diagrams/vla-learning-map.excalidraw" alt="VLA learning roadmap" caption="系列学习路径：先建立边界，再进入 world model、imitation pipeline、VLA、动作表征、系统部署和前沿融合。" >}}

## 读者假设

我默认读者已经理解强化学习的基本对象：state、action、reward、policy、value function、model-free / model-based、offline RL / imitation learning 的差异。这个系列不会从 Bellman equation 开始。

但我不默认读者熟悉 robotics 工程栈。尤其是下面这些问题，会在系列里反复拆开：

- robot dataset 到底长什么样；
- observation 是 image、state、proprioception 还是 action history；
- action 是单步连续向量、action chunk、离散 token 还是 diffusion trajectory；
- evaluation 是 open-loop prediction 还是 closed-loop rollout；
- language grounding 对 policy learning 到底提供了什么；
- world model 预测 future image、latent state、reward、done 的区别是什么。

## 为什么学习顺序不能直接从 OpenVLA 开始

OpenVLA 是非常重要的开源 VLA baseline。它展示了一个 7B VLA 如何从大规模 robot demonstrations 中学习，并且支持对新任务 fine-tune。但如果第一站就是 OpenVLA，学习者通常会掉进三个坑。

第一，只看到模型规模，看不到 robotics pipeline。OpenVLA 不是凭空工作的，它仍然依赖 dataset normalization、action discretization / decoding、robot embodiment、rollout evaluation 等基础设施。

第二，把 language-conditioned policy 误认为所有 embodied learning 的中心。实际上，很多 manipulation 任务在没有语言时也可以由 ACT、Diffusion Policy、BC-Transformer 做得很好。VLA 的优势必须相对于这些非 VLA baseline 来理解。

第三，忽视系统约束。机器人闭环控制不是离线生成文本。inference latency、control frequency、action chunking、asynchronous execution 会直接影响成功率。SmolVLA 把 affordable deployment 和 async inference 放在核心位置，就是因为大模型推理速度本身已经成为 robotics bottleneck。

所以这个系列先从总路线开始，再讲问题边界；随后进入历史脉络、基础 pipeline、大模型 VLA 和前沿 report。

## 系列结构

<div class="roadmap-grid">
  <div class="roadmap-card"><strong>0. 路线图</strong>建立学习顺序、读者假设和产出规范。</div>
  <div class="roadmap-card"><strong>1. 问题框架</strong>区分 policy、world model、VLA 的输入、输出和训练目标。</div>
  <div class="roadmap-card"><strong>2. 历史脉络</strong>从 imitation learning、VLM、robot foundation model 到 VLA。</div>
  <div class="roadmap-card"><strong>3. World Model</strong>理解 latent dynamics、imagination rollout、planning 和 error accumulation。</div>
  <div class="roadmap-card"><strong>4. Imitation Pipeline</strong>从 LeRobot、ACT、Diffusion Policy、robomimic 看懂数据、训练、评估。</div>
  <div class="roadmap-card"><strong>5. VLM to VLA</strong>解释 VLM 如何变成能输出机器人动作的 policy。</div>
  <div class="roadmap-card"><strong>6. Action Representation</strong>比较 continuous action、chunk、tokenization、diffusion、flow matching。</div>
  <div class="roadmap-card"><strong>7. Systems</strong>讨论 latency、async inference、quantization、edge / on-device VLA。</div>
  <div class="roadmap-card"><strong>8. Frontier</strong>分析 WorldVLA、RynnVLA、Gemini Robotics、Genie 等融合趋势。</div>
</div>

## 每篇文章的固定问题

读任何 VLA 或 world model paper，都先不要问“这个模型强不强”。先问五个更基本的问题：

1. 输入是什么：image、text、proprioception、state、action history？
2. 输出是什么：next state、reward、done、single action、action chunk、action token？
3. 是否预测未来：完全不预测、latent future、pixel future、structured state future？
4. 是否使用语言：不用、可选 conditioning、原生 language-conditioned？
5. 训练目标是什么：BC、autoregression、diffusion / flow matching、RL、joint objective？

这五个问题可以把大多数方法放回同一个坐标系。比如 ACT 是 action chunk imitation policy；Dreamer 是 latent world model + actor-critic；OpenVLA 是 language-conditioned action model；WorldVLA 则尝试把 action generation 和 future generation 放进同一个自回归框架。

## 当前前沿怎么读

截至 2026-05-04，VLA 方向至少有五条值得跟踪的线。

第一是 open generalist VLA。OpenVLA 在 2024 年给出了开源 VLA baseline，SmolVLA 在 2025 年把问题推向低成本训练和消费级部署。

第二是 action representation。π0 用 flow matching 生成连续动作，π0-FAST 代表了 action tokenizer / faster serving 方向；diffusion VLA 则尝试用 denoising 过程同时建模动作和未来视觉结果。

第三是 humanoid / general robot foundation model。NVIDIA GR00T N1 把 VLA 放到 humanoid、多源数据和 synthetic data flywheel 语境下讨论。

第四是 on-device 和 low-latency。Gemini Robotics On-Device、QuantVLA、AC^2-VLA 代表一个很现实的判断：机器人不是只缺 reasoning，也缺能以足够低延迟持续闭环执行的模型。

第五是 VLA + world model 融合。WorldVLA、RynnVLA、MMaDA-VLA 这类工作都在回答同一个问题：future prediction 是辅助 representation，还是应该成为 policy 本身的一部分？

## 固定产出

每一周的学习不应该只停留在“读了几篇 paper”。更好的节奏是：

- 一页阅读笔记：用自己的话解释输入、输出、训练目标、evaluation；
- 一张 Excalidraw 方法图：必须画出数据流和 loss / rollout 的位置；
- 一个最小实验或 smoke test：哪怕只跑 CPU / MPS 上的小任务；
- 一份问题列表：哪些是 engineering choice，哪些可能是 research contribution。

## 参考入口

- OpenVLA: [An Open-Source Vision-Language-Action Model](https://huggingface.co/papers/2406.09246)
- SmolVLA: [A Vision-Language-Action Model for Affordable and Efficient Robotics](https://huggingface.co/papers/2506.01844)
- π0: [A Vision-Language-Action Flow Model for General Robot Control](https://huggingface.co/papers/2410.24164)
- WorldVLA: [Towards Autoregressive Action World Model](https://huggingface.co/papers/2506.21539)
- Gemini Robotics: [Google DeepMind model page](https://deepmind.google/models/gemini-robotics/)
- Gemini Robotics On-Device: [Google DeepMind model page](https://deepmind.google/models/gemini-robotics/gemini-robotics-on-device/)
- GR00T N1: [NVIDIA research page](https://research.nvidia.com/publication/2025-03_nvidia-isaac-gr00t-n1-open-foundation-model-humanoid-robots)
- Genie 3: [Google DeepMind model page](https://deepmind.google/models/genie/)
