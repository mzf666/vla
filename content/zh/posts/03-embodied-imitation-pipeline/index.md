---
title: "非 VLA 的 Embodied Imitation Pipeline：先看懂机器人学习工程栈"
date: 2026-04-30T09:00:00+08:00
draft: true
tags: ["imitation-learning", "lerobot", "act", "diffusion-policy", "robomimic"]
series: ["VLA Tutorial"]
weight: 5
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "在进入大 VLA 之前，先用 LeRobot、ACT、Diffusion Policy、robomimic 建立 dataset、training、rollout、evaluation 的工程直觉。"
---

## Motivation

很多 VLA 文章从模型结构开始讲：vision encoder、language backbone、action head。但如果你没有先看懂 embodied imitation pipeline，后面很容易只记住“大模型名字”，看不到真实训练和评估链路。

机器人 policy repo 的核心通常不是一个 `model.py`，而是一整条闭环：dataset、normalization、policy、loss、checkpoint、rollout、metric。VLA 只是在这条链路上加入 language 和更强 backbone。

## 一个 Policy Repo 的 7 个模块

一个 embodied imitation repo 至少包含七个模块：

1. dataset：episode 如何组织，observation/action 如何存储；
2. transform：图像 resize、state normalization、action normalization；
3. policy：输入 observation，输出 action 或 action chunk；
4. loss：MSE、negative log-likelihood、diffusion denoising、chunk reconstruction；
5. trainer：batching、checkpoint、logging；
6. rollout：把 policy 放回环境或真实机器人闭环执行；
7. metric：success rate、return、completion、latency、failure mode。

这七个模块比模型名字更重要，因为任何 VLA 的实验结论都依赖它们。

## LeRobot：适合入门的最小栈

LeRobot 的价值在于把 robot learning pipeline 做成比较现代的工程形态。它提供 dataset 格式、policy 实现、训练命令和 evaluation。对入门来说，`PushT + ACT` 是很合适的第一站：任务简单，但完整包含 observation、action、training、rollout 和 success metric。

你应该重点看三个东西：

- episode 中每个 timestep 存了什么；
- action 是单步还是 chunk；
- evaluation 是 open-loop loss 还是 closed-loop success。

## ACT：Action Chunking 的直觉

ACT 的核心思想是一次预测一段 action：

$$
\pi(a_{t:t+H} \mid o_t)
$$

这比单步 action 有两个好处。第一，模型可以学习局部动作轨迹，而不是每一步都重新决定。第二，推理频率可以低于控制频率，缓解大模型 latency。

缺点是 chunk 长度会带来 trade-off。太短，无法缓解延迟；太长，遇到环境扰动时不够灵活。VLA 中的 action chunking 也是同一个问题。

## Diffusion Policy：把动作当作生成对象

Diffusion Policy 把未来动作轨迹看成一个生成分布，而不是单个均值预测。这对 manipulation 很自然，因为同一个状态下可能有多种合理动作路径。MSE 会把多峰行为平均掉，生成模型能保留多模态。

但 diffusion 的推理通常更慢，需要多步 denoising。机器人系统里，这会直接影响 closed-loop frequency。因此后来的 π0、FAST、async inference 都在和这个问题搏斗：动作生成要强，但不能慢到机器人无法实时执行。

## robomimic：工程抽象的重要性

robomimic 的价值不是某个 SOTA policy，而是它把 imitation learning 的 dataset、algorithm、observation encoder、training loop 和 evaluation 做成清楚的模块。读 robomimic 能帮你建立工程抽象：哪些代码是算法贡献，哪些只是实验管线。

这对读 VLA 论文尤其重要。很多 report 的提升来自数据、normalization、evaluation 或 deployment trick，而不是单纯 architecture。

## 最小实验协议

建议第一轮只做一件事：跑通 `LeRobot + ACT + PushT`。记录六个事实：

1. 安装是否顺利；
2. dataset schema 是什么；
3. 训练入口命令是什么；
4. action chunk size 是多少；
5. evaluation metric 是什么；
6. Mac mini / MPS / CPU 的瓶颈在哪里。

完成这个实验后，再读 SmolVLA 或 OpenVLA，你会更容易看出它们到底比 ACT 多了什么：语言输入、更强视觉语言 backbone、更复杂 action representation，还是更大规模数据。

## 本文结论

VLA 的地基是 embodied imitation pipeline。没有 dataset、normalization、rollout 和 metric 的理解，讨论 VLA 只会停留在模型命名层面。先把非 VLA 的 policy pipeline 跑通，再进入 VLA，是更稳的学习路径。
