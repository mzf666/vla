# Systematic Learning Roadmap

这份路线按 `基础概念 -> 经典方法 -> 工程复现 -> 前沿方向` 组织，目标是让你既能读 paper，也能尽快跑通 embodied / sim 实验。

## 总体原则

- 先建立概念边界，再进入 repo
- 先跑 `small and complete` 的 pipeline，再看大模型
- 先区分 `policy`、`world model`、`VLA`，再看它们如何融合
- 每一阶段都同时做三件事:
  - 读 2-4 篇核心材料
  - 画 1 张自己的方法图
  - 跑 1 个最小实验

## 阶段 0: 建立问题框架

目标:

- 搞清楚你在学的到底是哪三类东西:
  - `Imitation / offline policy learning`
  - `World model / model-based RL`
  - `VLA / language-conditioned embodied policy`

你需要回答的 5 个问题:

1. 输入是什么: `image / text / proprio / state / action history`
2. 输出是什么: `next state / reward / done / action / action chunk`
3. 是否预测未来: `no / latent / pixel / structured state`
4. 是否使用语言: `no / optional / native`
5. 训练目标是什么: `BC / diffusion / autoregression / RL / joint objective`

阶段产出:

- 用自己的话写出:
  - 什么是 world model
  - 什么是 VLA
  - VLA 和 world model 的区别与交集

## 阶段 1: 先学 World Model 基础

目标:

- 建立对 `latent dynamics`、`imagination`、`planning`、`model-based RL` 的直觉

重点理解:

- world model 不一定生成高质量视频
- 对机器人更重要的是:
  - 未来状态预测
  - reward / done prediction
  - 规划与 policy improvement
  - 长时序 consistency

建议阅读顺序:

1. 你提供的 `World Model for Robot Learning: A Comprehensive Survey`
2. `A Comprehensive Survey on World Models for Embodied AI`
3. `Dreamer / DreamerV3`
4. `DayDreamer`
5. `TD-MPC2`

这一阶段必须搞懂:

- RSSM / latent dynamics 在干什么
- 为何很多方法不直接在 pixel space 规划
- imagination rollout 和真实 rollout 的关系
- error accumulation 为什么是核心难点

阶段实验:

- 跑 `DreamerV3` 的最小例子
- 输出一张你自己的 pipeline 图:
  - `encoder -> latent dynamics -> decoder/reward/value -> actor`

## 阶段 2: 先学非 VLA 的 embodied imitation pipeline

目标:

- 在进入大 VLA 之前，先把机器人学习最基础的工程链跑通

为什么先做这一步:

- 很多 VLA repo 本质上是在已有 imitation / offline policy pipeline 上叠加语言和更大的 backbone
- 如果先不理解基础 pipeline，后面看 VLA 容易只看到“模型很大”，看不到训练结构

建议工具链:

- `LeRobot`
- `robomimic`

建议任务:

- `PushT`
- 简单 manipulation benchmark

这一阶段要吃透:

- dataset 格式
- observation 组织方式
- action chunking
- train / eval / rollout 的边界
- success metric 怎么定义

阶段实验:

- `LeRobot + ACT + PushT`
- 如果时间够，再看 `robomimic` 的 BC / BC-Transformer / Diffusion Policy

阶段产出:

- 写一页笔记:
  - “一个 embodied policy repo 最核心的 7 个模块是什么”

## 阶段 3: 进入真正的 VLA

目标:

- 理解 `vision-language model` 如何改造成机器人 policy

优先顺序:

1. `SmolVLA`
2. `OpenVLA`
3. 再看 `pi0 / pi0-FAST / GR00T` 这类更靠近前沿的体系

先看什么:

- 视觉 backbone
- 语言 backbone
- action representation:
  - continuous action
  - discretized action
  - action chunk
  - action tokens / FAST
- finetuning recipe
- evaluation setup

这一阶段要搞懂:

- VLM 到 VLA 的关键改造点
- 为何动作 tokenization 很重要
- 为什么 inference latency 在机器人里是第一等问题
- `small VLA` 和 `large VLA` 的 trade-off

阶段实验:

- `LeRobot + SmolVLA` 跑通最小训练或 smoke test
- 阅读 `OpenVLA` 的训练和 finetune 文档，不强求本地完整训练

阶段产出:

- 写一页对比:
  - `ACT vs Diffusion Policy vs SmolVLA vs OpenVLA`

## 阶段 4: 学 VLA 与 World Model 的融合趋势

目标:

- 不再把它们看成两条完全分开的线，而是看它们如何逐渐合流

关注方向:

- VLA 加未来预测
- joint policy + world modeling objective
- future latent alignment
- action-conditioned video / latent generation
- learned simulator for policy improvement

建议阅读:

- `WorldVLA`
- `VLA-World`
- `Gemini Robotics`
- `Gemini Robotics On-Device`
- `Genie 2 / Genie 3`

这一阶段要判断的问题:

1. world model 是辅助 policy，还是 policy 的一部分
2. 未来预测是在 latent 做，还是 pixel 做
3. 预测出来的未来怎么影响 action
4. 这种结构是提升 sample efficiency、generalization，还是 reasoning / foresight

阶段产出:

- 画一张 “VLA 与 world model 融合方式” 分类图

## 阶段 5: 聚焦你的研究切入点

目标:

- 从“学习别人做了什么”进入“我可能做什么”

适合博士阶段的切入问题:

- 小模型 VLA 在 Apple Silicon / edge device 上的部署边界
- VLA 中 action representation 对泛化与稳定性的影响
- language grounding 对 sim manipulation 到 real transfer 的贡献
- 在小型 benchmark 上引入 future prediction / auxiliary world modeling 是否有效
- `SmolVLA / ACT / Diffusion` 在小数据 regime 下的比较

建议做法:

1. 选一个小环境
2. 复现一个 baseline
3. 改一个变量
4. 固定 evaluation protocol

## 6-8 周建议排期

### 第 1 周

- 读 world model survey
- 整理术语
- 写出 world model / VLA / policy 的边界

### 第 2 周

- 读 DreamerV3 / DayDreamer / TD-MPC2
- 跑 DreamerV3 最小例子

### 第 3 周

- 学 LeRobot 基础
- 跑 ACT 或 PushT 相关最小训练

### 第 4 周

- 学 robomimic 的 dataset / algo / eval 结构
- 对比 BC、Transformer、Diffusion

### 第 5 周

- 读 VLA survey
- 读 SmolVLA / OpenVLA
- 整理 action representation 笔记

### 第 6 周

- 跑 SmolVLA 的最小 smoke test 或小规模 finetune
- 记录 Mac mini 上的资源瓶颈

### 第 7 周

- 学 VLA + world model 最新方向
- 读 WorldVLA / VLA-World / Gemini Robotics / Genie

### 第 8 周

- 定研究切入点
- 确定一个能在本地或小算力下持续推进的小题目

## 每周固定输出

- `1` 页阅读笔记
- `1` 张方法图
- `1` 个最小实验结果
- `1` 份问题列表:
  - 我没看懂什么
  - 哪些设计只是 engineering，哪些是 research contribution

## 明确不建议的路径

- 一上来就试图完整训练 `OpenVLA`
- 一上来就追最火、最大、最复杂的 VLA 系统
- 把高质量视频生成误当作 world model 的唯一目标
- 在不理解 dataset / action / eval protocol 的前提下比较方法
