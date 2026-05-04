---
title: "博士研究切入点：从复现 Baseline 到提出可验证问题"
date: 2026-04-25T09:00:00+08:00
draft: false
tags: ["research", "vla", "experiments", "apple-silicon"]
series: ["VLA Tutorial"]
weight: 10
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "把 VLA 学习路线收束成可执行研究问题：小模型部署、动作表征、小数据 regime、future prediction auxiliary objective。"
---

## Motivation

读完整个系列后，最重要的问题不是“哪个 VLA 最强”，而是“我能在有限资源下提出什么可验证的问题”。对 PhD 来说，好的切入点应该小、清晰、可复现，并且能解释失败。

VLA 方向很容易被大公司 report 吓退，但并不是所有有价值的问题都需要训练 7B/70B 模型。动作表征、小数据 regime、future prediction auxiliary objective、部署延迟、quantization、benchmark protocol，都可以成为严肃研究问题。

## 切入点 1：小模型 VLA 与 Edge Deployment

问题：在 Apple Silicon、Jetson 或消费级 GPU 上，VLA 的可用边界在哪里？

可做实验：

- SmolVLA / ACT / Diffusion Policy 在同一任务上的 latency-success trade-off；
- 不同 quantization 对 action error 和 rollout success 的影响；
- async inference 对 closed-loop stability 的影响。

这个方向的价值在于连接模型和真实部署。很多 paper 不报告系统指标，你可以把它补上。

## 切入点 2：Action Representation Ablation

问题：continuous、chunk、token、diffusion、flow matching 在同一 benchmark 上到底差在哪里？

最小实验可以控制 dataset 和 backbone，只改变 action representation。报告：

- offline loss；
- closed-loop success；
- latency；
- failure mode；
- 对扰动的恢复能力。

这个方向容易做出清楚贡献，因为 action representation 是 VLA 的核心瓶颈，但很多 report 的比较并不完全公平。

## 切入点 3：Future Prediction 是否有用

问题：给一个 imitation policy 加 future prediction auxiliary loss，是否真的提升 closed-loop success？

可以从小任务开始：PushT、LIBERO subset、ManiSkill 简单 manipulation。baseline 是 ACT 或 Diffusion Policy，然后加入 future latent / future image prediction。关键不是生成图像好不好，而是 rollout 是否更稳定、泛化是否更好。

评价时要区分三种可能：

- future loss 改善 representation；
- future prediction 支持 planning；
- future branch 只是增加参数和正则化，没有实质帮助。

## 切入点 4：Language Grounding 的真实贡献

问题：语言到底什么时候有用？

很多固定 manipulation task 中，语言可能只是 task id。真正需要语言的 setting 应该包含组合泛化：新物体组合、新空间关系、新指令表达。你可以设计对照：

- no language；
- task id；
- template language；
- natural language；
- paraphrase / compositional split。

如果 VLA 只在 template language 上有效，它和多任务 policy 的差别并不大。

## 切入点 5：Small-Data Regime

问题：当 demonstration 很少时，VLA 的 pretrained prior 是否比专用 policy 更有价值？

这是 VLA 最合理的应用场景之一。大模型的优势不一定在充分数据下超过专用模型，而是在少数据、新任务、新指令中提供先验。

实验可以比较：

- ACT；
- Diffusion Policy；
- SmolVLA；
- OpenVLA fine-tune；
- 不同 demo 数量下的 success curve。

## 最小研究协议

一个可发表的小实验至少要有：

1. 选一个小环境：PushT、LIBERO subset、ManiSkill CPU task。
2. 复现一个 baseline：ACT 或 Diffusion Policy。
3. 改一个变量：action chunk size、tokenization、future prediction loss、quantization。
4. 固定 evaluation：same seeds、same demos、same rollout budget。
5. 报告失败模式：latency、drift、object grounding、stage hallucination、recovery failure。

5. 报告失败模式：latency、drift、object grounding、stage hallucination、recovery failure。

不要同时改三个变量。VLA 领域已经足够复杂，研究贡献往往来自控制变量后的清楚结论。

## 本文结论

VLA 的研究机会不只属于巨型实验室。只要问题定义足够清楚，小模型部署、动作表征、小数据、future prediction、language grounding 都可以形成扎实研究。关键是不要做“又训练一个 VLA”，而是做一个能解释为什么有效或为什么失败的 controlled study。
