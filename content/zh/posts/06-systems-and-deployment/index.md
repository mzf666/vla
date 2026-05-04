---
title: "VLA 系统与部署：Latency、Async Inference、Quantization 与 On-Device"
date: 2026-05-04
draft: true
tags: ["vla", "systems", "latency", "quantization", "on-device"]
series: ["VLA Tutorial"]
weight: 6
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "机器人闭环部署不是离线 benchmark：低延迟、稳定控制频率、模型压缩和 on-device 推理决定了 VLA 能不能真的动起来。"
---

## 写作目标

从系统角度解释为什么 VLA deployment 的关键瓶颈不是单次准确率，而是闭环执行中的 latency、jitter、memory、power 和 safety fallback。

## 文章结构

1. robot control loop 与 LLM/VLM inference loop 的冲突。
2. action chunking 如何减少高频推理需求。
3. SmolVLA async inference：把 perception/action prediction 与 action execution 解耦。
4. QuantVLA：post-training quantization 对内存和 latency 的价值。
5. AC^2-VLA：action-context-aware adaptive computation。
6. Gemini Robotics On-Device：为什么 on-device VLA 是独立方向。

## 必画 Excalidraw 图

- synchronous control loop vs asynchronous control loop
- quantization / adaptive computation 在 VLA serving path 中的位置
- cloud / edge / on-device 三种部署拓扑

## 参考入口

- [Gemini Robotics On-Device](https://deepmind.google/models/gemini-robotics/gemini-robotics-on-device/)
- [QuantVLA](https://huggingface.co/papers/2602.20309)
- [AC^2-VLA](https://huggingface.co/papers/2601.19634)

