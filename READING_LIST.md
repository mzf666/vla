# Reading List

## A. 先读的 Survey

### 1. World Model 主线

1. `World Model for Robot Learning: A Comprehensive Survey`
   链接: <https://arxiv.org/pdf/2605.00080>
   用途: 作为当前主线入口，覆盖 robot learning 视角下的 world model

2. `A Comprehensive Survey on World Models for Embodied AI`
   链接: <https://arxiv.org/abs/2510.16732>
   用途: 提供更明确的 taxonomy:
   - Functionality
   - Temporal modeling
   - Spatial representation

3. `Robotic world models—conceptualization, review, and engineering best practices`
   链接: <https://pmc.ncbi.nlm.nih.gov/articles/PMC10652279/>
   用途: 补“robotics community 对 world model 的工程语义”

### 2. VLA 主线

1. `Pure Vision Language Action (VLA) Models: A Comprehensive Survey`
   链接: <https://arxiv.org/abs/2509.19012>

2. `An Anatomy of Vision-Language-Action Models: From Modules to Milestones and Challenges`
   链接: <https://arxiv.org/abs/2512.11362>

## B. 必读代表方法

### 1. World Model / Model-Based RL

- `DreamerV3`
  - 项目: <https://github.com/danijar/dreamerv3>
  - 论文主页: <https://danijar.com/project/dreamerv3/>

- `DayDreamer`
  - 项目: <https://github.com/danijar/daydreamer>
  - 论文主页: <https://danijar.com/project/daydreamer/>

- `TD-MPC2`
  - 项目: <https://github.com/nicklashansen/tdmpc2>
  - 主页: <https://www.tdmpc2.com/>

### 2. VLA

- `OpenVLA`
  - 论文: <https://arxiv.org/abs/2406.09246>
  - 代码: <https://github.com/openvla/openvla>
  - 项目页: <https://openvla.github.io/>

- `SmolVLA`
  - 论文: <https://arxiv.org/abs/2506.01844>
  - 文档: <https://huggingface.co/docs/lerobot/v0.4.3/en/smolvla>
  - 代码基座: <https://github.com/huggingface/lerobot>

## C. 最新融合方向

- `WorldVLA`
  - 论文: <https://huggingface.co/papers/2506.21539>
  - 线索: VLA 与 world model 的联合建模方向

- `RynnVLA-002`
  - 论文: <https://huggingface.co/papers/2511.17502>
  - 线索: unified VLA and world model

- `Learning Vision-Language-Action World Models for Autonomous Driving`
  - 项目页: <https://vlaworld.github.io/>

- `Gemini Robotics`
  - <https://deepmind.google/blog/gemini-robotics-brings-ai-into-the-physical-world/>

- `Gemini Robotics On-Device`
  - <https://deepmind.google/discover/blog/gemini-robotics-on-device-brings-ai-to-local-robotic-devices/>

- `Genie 2`
  - <https://deepmind.google/blog/genie-2-a-large-scale-foundation-world-model/>

- `Genie 3`
  - <https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/>

- `QuantVLA`
  - 论文: <https://huggingface.co/papers/2602.20309>
  - 线索: VLA post-training quantization、memory saving、latency reduction

- `AC^2-VLA`
  - 论文: <https://huggingface.co/papers/2601.19634>
  - 线索: action-context-aware adaptive computation

- `MMaDA-VLA`
  - 论文: <https://huggingface.co/papers/2603.25406>
  - 线索: diffusion VLA、future goal observation 与 action chunk 的联合生成

## D. 读论文时的固定问题

每篇 paper 都尽量回答以下问题:

1. observation 是什么
2. action 是什么格式
3. 是否用语言
4. 是否显式建模未来
5. world model 学的是什么空间:
   - state
   - latent
   - pixel
6. policy 怎么训练:
   - BC
   - actor-critic
   - diffusion
   - autoregression
7. evaluation 在哪里做:
   - simulation
   - real robot
   - both
8. 主要瓶颈是什么:
   - compute
   - data
   - latency
   - horizon
   - generalization
