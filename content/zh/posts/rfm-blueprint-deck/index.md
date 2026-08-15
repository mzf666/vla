---
title: "RFM 实施蓝图 —— Pitch Deck"
date: 2026-08-15
draft: false
---

六张主线幻灯片，外加四张备用。每张只有一张图加三行字：**逻辑必须能从图上读出来**，文字只负责点名信息增量 [computed: 本 deck 的设计约束]。

---

## 1 —— 整个系统

![](figures/f01-system.png)

- 两个工厂：Model Factory 把数据变成能在机器人上跑的策略，Data Flywheel 把机器人的运行变成可训练的数据 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)
- 它们之间只有两条通路：Policy API Contract 与 Episode Contract；除此之外互不知道对方内部 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)
- evaluation 是两者之间的器官，同时卡住"什么能上车"和"下一轮采什么" [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)

---


---

## 3 —— 数据地图

![](figures/f04-data-map.png)

- 两个轴决定一份语料的用途：action grounding 决定它能训模型的哪一部分，policy relatedness 决定它能不能支撑 value 学习 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)
- operations 是这张图上的向量，每一条都有明确成本与明确失真，并自带验证义务 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)
- 自有本体上带 action grounding 的真实数据是必需品，这张图上没有任何操作能把它制造出来 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)

---

## 4 —— 数据飞轮

![](figures/f09-flywheel.png)

- 单机 128 GB/h，300 台车队一个 8 h 班次是 307 TB/day——分流必须做在机器人本体上 [computed: 128 GB/h × 8 h × 300]
- 只打分不删除；label 是挂在不可变 episode 上的可变层；dataset version 就是一份 manifest [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)
- 那份随机配额不是可选项：没有它，留下来的数据全是失败，模型学到一个永远出错的世界 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)

---

## 5 —— 评估：整条路线的瓶颈

![](figures/f11-eval.png)

- 区分 50% 与 60% 的成功率需要约 387 次试验，而公开工作每个 checkpoint 只配 100 次真机试验 [computed: two-proportion test]
- 所以先重建场景、再用 2000 次仿真试验筛选，真机只花在幸存者身上 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)
- 仿真自己的可信度必须被持续测量：rank fidelity 达不到 0.80 就换更贵的通道，而不是假装它成立 [computed: Spearman floor for screening use]

---

## 6 —— 路线图

![](figures/f12-roadmap-public.png)

- 五个阶段按顺序排列而不按日期排列，每个阶段由它消掉的风险定义，由一个客观放行条件结束 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)
- 评估建在模型之前，端侧排在飞轮闭合之后——两个次序都写明了理由和可以推翻它们的条件 [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)
- P0 到 P2 抄已经被复现过的配方；P3 是我们停止抄作业的地方，风险集中在一个被点名的阶段 [[repo: FlashRT]](https://github.com/flashrt-project/FlashRT)

---

# 备用（问答时调取）

## B1 —— 压缩链路

![](figures/f07-compression.png)

- 排序由实测决定：编译零精度代价拿 1.5 到 3.34×，少步蒸馏拿 3.3× 且精度不降 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)
- 量化买到的是显存不是速度——通用工具链只量化语言 backbone，而瓶颈在 action head [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)
- 2.5 bpw 是拐点，2 bpw 崩塌到 48.0%；规则是停在 4 bpw [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)

## B2 —— Serving

![](figures/f08-serving.png)

- batch-1 的 VLA 推理是 memory-bandwidth bound，优化目标是搬运的字节数而不是 FLOPs [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)
- 同一个前向在 Jetson Thor 上 action head 占 50%，在 RTX 4090 上只占 23%——边缘侧最大的那一项正是通用量化不碰的那一项 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)
- 已公开最好端侧成绩来自手写 kernel：44.0 ms、23 Hz，越过了解析屋顶线 [[repo: FlashRT]](https://github.com/flashrt-project/FlashRT)

## B3 —— 配比如何随阶段迁移

![](figures/f13-mixture-shift.png)

- 阶段之间变的是权重，不是建设顺序：公开与人类来源语料从 92 降到 30 [computed: 各阶段可得数据源的组合]
- on-policy rollout 从 0 升到 48，是唯一一个成本随算力和车队扩张、而不随人员编制扩张的来源 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)
- 但它扩不出新的能力包络，所以前两条曲线永远不会归零 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)

## B4 —— 我们还不知道什么

- 教师规模、机上功耗、我们自己的压缩悬崖位置，都由 P1 与 P3 实测确定 [computed: 本模块 FACTS.md 的 GAP 组]
- 接管到可测提升的转化率没有公开曲线，它决定 P2 需要多大车队 [computed: 检索未发现已披露转化曲线]
- Open bet：world action model 已报告 2 倍泛化优势，但 14B 参数、7 Hz 的形态进不了端侧预算；触发条件是压缩后能进预算的版本出现 [[arXiv:2602.15922]](https://arxiv.org/abs/2602.15922)
