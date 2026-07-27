---
title: "VLA 系统与部署：Latency、Async Inference、Quantization 与 On-Device"
date: 2026-04-27T09:00:00+08:00
draft: true
tags: ["vla", "systems", "latency", "quantization", "on-device"]
series: ["VLA Tutorial"]
weight: 8
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "机器人闭环部署不是离线 benchmark：低延迟、稳定控制频率、模型压缩和 on-device 推理决定了 VLA 能不能真的动起来。"
---

## Motivation

机器人里的 VLA 不是离线 benchmark。一个模型即使在数据集上预测动作很准，如果推理太慢、显存太大、控制频率不稳定，真实机器人仍然会失败。VLA 的 SOTA 不能只看 success rate，也要看 latency、memory、power、jitter 和 failure recovery。

这就是为什么 SmolVLA、Gemini Robotics On-Device、QuantVLA、AC^2-VLA 这类工作重要。它们不是“模型小一点”的附属方向，而是在回答 VLA 能否真正部署的问题。

## Control Loop 与 Model Loop 的冲突

机器人控制通常是高频闭环。相机、状态估计、policy inference、action execution 必须持续循环。如果 policy inference 需要几百毫秒，而机器人需要几十 Hz 控制，就会出现延迟错位：模型看到的是旧状态，执行的是过时动作。

VLA backbone 往往来自 VLM/LLM，本来不是为实时控制设计的。这会产生三个系统问题：

1. latency：单次推理是否足够快；
2. jitter：推理时间是否稳定；
3. throughput：多相机、多任务、多机器人时是否能扩展。

## Action Chunking 与 Async Inference

action chunking 是最直接的系统缓解方式。模型一次输出 \(H\) 步动作，控制器逐步执行。在执行当前 chunk 时，系统可以异步准备下一段动作。

SmolVLA 强调 async inference，核心就是把 action execution 和 action prediction 解耦。这样模型不需要每个低级控制 step 都推理一次，机器人也不会完全停下来等大模型。

但 async inference 也有风险：如果环境变化太快，正在执行的 chunk 可能过时。因此系统需要重规划、interrupt、safety controller 或低级 reactive policy。

## Quantization：显存与延迟不是附属指标

QuantVLA 代表 VLA quantization 方向。对部署来说，post-training quantization 可以降低显存、提高吞吐、减少延迟。但机器人动作对数值误差敏感，不能只看 language benchmark 的 perplexity。

评估 VLA quantization 时应关注：

- action error 是否增大；
- closed-loop success 是否下降；
- long-horizon drift 是否加重；
- 不同任务和机器人 embodiment 是否同样稳定。

## Adaptive Computation

AC^2-VLA 这类 adaptive computation 方向的动机是：不是所有时刻都需要同样大的计算量。抓取前的视觉定位、接触瞬间、长距离移动、任务切换，对模型计算需求不同。

如果模型能根据 action context 动态调整计算，就有可能在不明显牺牲成功率的情况下降低平均延迟。这和 LLM serving 里的 early exit / dynamic routing 类似，但机器人里评价标准更严格：不能因为省计算导致控制不稳定。

## On-Device VLA

Gemini Robotics On-Device 把问题进一步推到本地机器人设备。on-device 的价值包括：

- 低延迟；
- 不依赖网络；
- 数据隐私；
- 更稳定的控制闭环；
- 可在边缘场景部署。

但 on-device 也意味着更小模型、更强压缩、更严格功耗约束。它不是 cloud VLA 的简单缩小版，而是一条独立系统路线。

## 本文结论

VLA deployment 的核心不是“模型能不能在论文里成功”，而是“能不能在真实闭环里稳定、低延迟、低成本地成功”。action chunking、async inference、quantization、adaptive computation、on-device serving 都是 VLA SOTA 的组成部分。未来 VLA benchmark 如果不报告 latency 和部署条件，结论是不完整的。
