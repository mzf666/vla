---
title: "World Model 基础：Latent Dynamics、Imagination 与 Planning"
date: 2026-05-01T09:00:00+08:00
draft: true
tags: ["world-model", "model-based-rl", "dreamer", "td-mpc"]
series: ["VLA Tutorial"]
weight: 4
ShowToc: true
TocOpen: true
ShowReadingTime: true
summary: "从 Dreamer、DayDreamer、TD-MPC2 理解 world model 的真正目标：不是生成好看的视频，而是学到能帮助 policy improvement 的未来动力学。"
---

## Motivation

VLA 讨论的是“看图、听指令、输出动作”。world model 讨论的是“如果我执行某个动作，世界会怎样变化”。这两个问题看似不同，但对机器人来说最终会碰到同一个核心：模型是否理解 action 的后果。

很多人第一次接触 world model，会把它等同于 video generation。这是一个危险简化。机器人确实可能需要预测未来图像，但更重要的是预测与行动相关的未来：物体是否移动、接触是否发生、任务是否完成、失败是否不可逆。一个 world model 即使生成的画面不漂亮，只要 latent dynamics 对 action 和 reward 有用，它仍然可能是好的控制模型。

## World Model 的基本对象

最简形式的 model-based RL 假设我们学习：

$$
p_\theta(s_{t+1}, r_t, d_t \mid s_t, a_t)
$$

其中 \(s_t\) 可以是真实低维 state，也可以是从图像编码得到的 latent state；\(r_t\) 是 reward；\(d_t\) 是 done。学到这个模型后，agent 可以在模型里做 rollout，不必每一步都和真实环境交互。

现代 world model 通常不直接在 pixel space 规划，因为像素维度高、噪声多，而且很多视觉细节和控制无关。更常见的是：

$$
o_t \xrightarrow{\text{encoder}} z_t,\quad
(z_t, a_t) \xrightarrow{\text{dynamics}} z_{t+1},\quad
z_t \xrightarrow{\text{heads}} \hat{o}_t,\hat{r}_t,\hat{v}_t
$$

核心问题变成：\(z_t\) 是否保留了足够多的 action-relevant information。

## Dreamer：在 Latent Space 中想象

Dreamer 系列的关键贡献是把 representation learning、dynamics learning 和 actor-critic 放进一个闭环。它先用真实轨迹训练 latent dynamics，然后在 latent space 里展开 imagined rollout，再用这些 imagined trajectories 训练 actor 和 critic。

直觉上，Dreamer 做了三件事：

1. encoder 把高维 observation 压缩成 latent；
2. dynamics model 学习 action-conditioned transition；
3. actor 在 imagined future 中学习哪些 action 会带来更高 value。

这和 model-free RL 的区别是，policy improvement 不只依赖真实环境采样，也依赖模型内部的 imagination。对真实机器人来说，这很有吸引力，因为真实交互昂贵、慢且有安全风险。

## DayDreamer 与真实机器人

DayDreamer 把 Dreamer 思路带到真实机器人任务。它说明 world model 不只是 Atari / DeepMind Control 里的抽象算法，也可以在 manipulation、locomotion 等任务上帮助 sample efficiency。

但真实机器人会放大 world model 的问题：camera noise、contact dynamics、partial observability、actuator delay 都会让预测更难。模型预测几步还可以，预测几十步就容易漂移。这就是 error accumulation。

## TD-MPC2：Planning-Oriented Latent Model

TD-MPC2 代表另一类思路：不一定重建像素，而是学习更适合 planning 和 value prediction 的 latent。它强调 task-oriented representation：latent 的好坏由能否支持 model predictive control 和 value estimation 来判断。

这对 VLA 很重要。未来 VLA 加 world modeling 时，不一定要生成完整未来视频。更现实的目标可能是生成 action-relevant future latent，帮助 policy 判断下一段 action 是否会成功。

## Error Accumulation 为什么是核心难点

world model 的 rollout 是递归的：

$$
\hat{z}_{t+1}=f_\theta(z_t,a_t),\quad
\hat{z}_{t+2}=f_\theta(\hat{z}_{t+1},a_{t+1})
$$

一旦 \(\hat{z}_{t+1}\) 有误差，后续每一步都在错误状态上继续预测。短 horizon 的模型可能看起来很好，但 long-horizon planning 会迅速崩掉。

机器人任务尤其明显：夹爪稍微偏一点，接触状态就完全不同；接触状态错了，后续物体运动就全错。这也是为什么很多系统更偏向短 action chunk、receding horizon control，而不是一次规划完整长任务。

## 对 VLA 的启发

VLA 如果只输出动作，它可能缺少对未来后果的显式建模。world model 可以提供三种帮助。

第一，作为 auxiliary objective。让 VLA 在训练时预测 future latent 或 future image，迫使表示包含动态信息。

第二，作为 planner。policy 提出候选 action，world model 预测后果，再选择更好的 action。

第三，作为 unified generator。模型同时生成 future observation 和 action，把“看见未来”和“决定动作”放进同一个序列建模目标。

第三条是 WorldVLA、MMaDA-VLA 等前沿工作的方向，但也最难评估。关键问题不是模型能不能生成未来，而是生成的未来是否改善 closed-loop success。

## 本文结论

world model 的本质不是视频生成，而是 action-conditioned future modeling。对 robot learning 来说，一个好 world model 应该帮助 policy 更高效、更稳定、更有 foresight 地行动。

后面读 VLA + world model 的最新工作时，要始终追问：future prediction 是在 pixel、latent 还是 structured state 上做？它进入 action decision 了吗？它提升的是 sample efficiency、generalization，还是只提升了 auxiliary generation metric？
