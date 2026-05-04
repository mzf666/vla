# Experiment Plan for Mac mini

这部分只保留对 `Apple Silicon Mac mini` 相对现实的路线。

## 机器假设

- 当前环境是 `arm64`
- 优先考虑:
  - `MPS` 可用的 PyTorch 项目
  - CPU 也能 smoke test 的项目
  - 不依赖 NVIDIA-only 训练栈的项目

## 第一梯队: 最适合起步

### 1. DreamerV3

推荐原因:

- 官方说明支持 `Linux and Mac`
- world model 结构清晰
- 非常适合学习 `latent imagination` 的完整闭环

你的目标:

- 跑通一个最小 task
- 看懂配置系统
- 明确 `world model / actor / critic` 的交互方式

### 2. LeRobot + ACT + PushT

推荐原因:

- LeRobot 官方支持 `MPS`
- `ACT` 是官方明确推荐给初学者的策略
- `PushT` 数据和训练链路短，容易形成完整闭环

你的目标:

- 跑通:
  - dataset
  - train
  - eval
- 看懂 `action chunking`

## 第二梯队: 进入 VLA

### 3. LeRobot + SmolVLA

推荐原因:

- 当前最适合入门的真正 VLA 之一
- 目标就是 affordable / efficient robotics
- 比 OpenVLA 更适合本地理解与试错

你的目标:

- 先跑 `smoke test`
- 再决定是否做小规模 finetune
- 看懂:
  - VLM backbone
  - action expert
  - async inference

注意:

- 完整训练仍然更适合 GPU
- Mac mini 更适合:
  - 安装
  - 推理
  - 小规模验证
  - 阅读代码

## 第三梯队: 作为阅读对象

### 4. OpenVLA

推荐原因:

- canonical open VLA baseline
- 论文与工程影响力都很高

不建议作为第一个本地训练目标的原因:

- 规模更大
- 工程链更重
- 对 Mac mini 不够友好

更合适的使用方式:

- 读 paper
- 读 repo
- 理解 finetune recipe
- 把它作为“大模型 VLA 参考设计”

## 备选工具链

### 5. robomimic

适合什么时候用:

- 你想彻底搞懂 imitation / offline RL 训练框架
- 你想要非常清晰的模块化代码结构

建议用途:

- 读 dataset pipeline
- 读 BC / Transformer / Diffusion Policy 的实现

### 6. ManiSkill

适合什么时候用:

- 你需要一个较现代的 manipulation simulator

注意:

- macOS 上官方说明以 `CPU simulation` 为主
- 更适合环境层面探索，不是当前首选训练主线

## 建议实验顺序

1. `DreamerV3`
2. `LeRobot + ACT + PushT`
3. `robomimic` 阅读与对照
4. `LeRobot + SmolVLA`
5. `OpenVLA` 阅读

## 每个实验都记录什么

固定记录 6 件事:

1. 安装是否顺利
2. 数据格式是什么
3. 训练入口命令是什么
4. 评估指标是什么
5. Mac mini 上的瓶颈是什么
6. 我从这个 repo 学到了什么结构性知识

## 最小成功标准

### Level 1

- 能跑通一个 world model 最小例子

### Level 2

- 能跑通一个 imitation learning embodied pipeline

### Level 3

- 能跑通一个真正 VLA 的安装与最小推理

### Level 4

- 能比较:
  - `ACT`
  - `Diffusion`
  - `SmolVLA`
  - `OpenVLA`

## 后续可以继续补的内容

- `INSTALL_NOTES.md`
- `QUESTIONS.md`
- `WEEKLY_LOG.md`
- `PAPER_NOTES/`
- `REPO_NOTES/`
