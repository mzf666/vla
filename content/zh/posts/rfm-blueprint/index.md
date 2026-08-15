---
title: "RFM 实施蓝图：模型工厂与数据飞轮"
date: 2026-08-15
draft: false
---

## 这份蓝图回答什么问题

假设你面前有一个已经把最难的一半做出来的伙伴：机器人本体已经量产、已经铺进真实场景、已经带 VR 遥操作。剩下的一半是 robot foundation model（RFM）与喂养它的数据系统。这份文档只回答一个问题——这一半具体建什么，按什么顺序建，每一步靠什么证据判断它成了 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

它不是综述，也不是论文导读。凡是能抄的地方一律抄成熟配方，只在绕不过去的地方做原创；凡是给出的数字，都能追溯到 `FACTS.md` 里的一行和它的出处 [computed: 本文的引用约定]。

## 怎么读

第 0 部分是一页纸的全景，第 1 部分是两份 contract。这两部分合起来就是完整蓝图，其余每一部分都只是它们的 zoom-in [computed: 文档结构约定]。

如果你只有十分钟，读这两部分；如果你要评估工程可行性，往下读第 2 部分和第 3 部分；如果你要判断风险如何被逐步消掉，直接跳到第 5 部分

---

# 第 0 部分 —— 一页纸的全景

![整个系统：两个工厂、两份 contract、一支车队](figures/f01-system.png)

整个系统由两个工厂组成。**Model Factory** 把数据变成一个能在机器人上跑的策略，流水线是 pretrain → post-train → experience loop → compress → serve。**Data Flywheel** 把机器人的运行变成可训练的数据，流水线是 generate → collect → clean → label → store → sample [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

两个工厂之间只有两条通路。Model Factory 交付的是一个符合 **Policy API Contract** 的模型；Data Flywheel 交付的是一批符合 **Episode Contract** 的 episode。除此之外它们互不知道对方的内部结构，这正是它们能被两个团队分头推进的原因 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。

车队让循环闭合。机器人拿到策略去干活，干活产生 episode，episode 训出更好的策略。这个循环本身就是资产，比循环里任何一次交付都更重要 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

中间那个器官是 **evaluation**。它同时卡住两个方向：没有经过筛选的模型不许上车，而它对失败的分析又决定了下一轮该采什么数据。把它画在两个工厂中间而不是塞进其中一个，是因为它属于两边 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。

有两个平台数字从一开始就压在所有设计上：电池 0.7 kWh，续航 2 h [[blog: carnewschina 2026-04-13]](https://carnewschina.com/2026/04/13/chery-begins-online-sales-of-humanoid-robot-with-a-0-7-kwh-battery-at-41400-usd/)。推理多耗一瓦，机器人就少干一会儿活；单机每小时产生 128 GB/h 的原始传感数据，没有任何回传链路吃得下 [computed: 35.665 MB/s × 3600]。这两条约束会在第 2 部分和第 3 部分反复出现。

---


---

# 第 1 部分 —— 两份 contract

![两份 contract：Episode Contract 与 Policy API Contract](figures/f03-contracts.png)

把这两份接口提到两个工厂之前，是因为后面每一个决策都从它们推导出来。只读第 0 部分和这一部分的读者，已经拿到了完整蓝图 [computed: 文档结构约定]。

## Episode Contract

一条可训练的 episode 必须包含：硬件同步的多路相机帧、关节状态与**指令力矩**、触觉通道、语言标注、遥操作者的意图与接管标记、任务结局，以及一个偏移有上界的硬件时钟 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。

关键约定是：**这份 contract 在机器人本体上强制执行，不在入库端**。入库端能做的只有拒收，而被拒收的那条 episode 已经永远丢失了 [[blog: Trossen robotic data pipeline]](https://www.trossenrobotics.com/post/robotic-data-pipeline-sensor-streams-to-training-datasets)。

指令力矩单独点出来，是因为它最常被漏掉。只记录关节状态，你能训出复现轨迹的策略，却训不出理解接触的策略；而接触恰恰是操作任务里最难的部分 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。接管标记同样如此：操作员按下接管的那一刻，本身就是一个免费的负样本标签，这一点在第 2 部分的 experience loop 里会被反复使用 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

## Policy API Contract

输入是一个 observation dict：若干路图像、机器人状态、一段语言 prompt。输出是一个 action chunk，形状为 H × DoF，外加一个标量 value 估计 [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)。

参考取值是 H 取 50，执行 horizon 取 25，在 50 Hz 控制器上对应 1.0 s 的动作块 [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)。这组数字来自已公开的实测系统，本蓝图直接沿用而不重新发明。

关键约定是：**这份 contract 冻结**。冻结之后，模型可以整代替换而不动机器人，机器人也可以换代而不必重训 backbone——后者靠一层 embodiment adapter 完成 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。

多出来的那个 value 输出值得解释一句。它在行为克隆阶段没有用处，却是第 2 部分 experience loop 的前提；把它写进冻结接口，意味着后面接上经验学习时不需要改动机器人侧的任何代码 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

---

# 第 2 部分 —— Model Factory

![Model Factory：每条边上写着下一阶段消费的产物](figures/f06-model-factory.png)

这一部分沿流水线走一遍，每个环节只交代三件事：它负责什么、接口是什么、它带来的信息增量在哪里 [computed: 本文的模块写法约定]。

## 3.1 数据配比

输入是各路语料，输出是一份 mixture manifest。这里的信息增量只有一句：**配比是训练阶段的函数，不是一个数据集** [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

同一批语料在 pretrain、post-train 与 experience loop 里的权重完全不同，而且随着车队规模变化持续漂移 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。把它固化成"我们的数据集"，等于把一个时变对象写死成了一个常量。

manifest 的形状是"来源 × 阶段"的一张权重表，而不是一个文件列表 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)：

| 来源 | pretrain | post-train | experience loop |
|---|---|---|---|
| 图文与人类视频 | 主力 | 不用 | 不用 |
| 第一人称可穿戴 | 主力 | 少量 | 不用 |
| 手持采集装置 | 参与 | 参与 | 不用 |
| 公开跨本体机器人数据 | 主力（需归一化） | 少量 | 不用 |
| 自有遥操作 | 少量 | 主力 | 作为锚点 |
| 自有 on-policy rollout | 不用 | 不用 | 主力 |
| 重建场景 | 不用 | 参与 | 评估与失败搜索 |

第 3 部分的数据地图解释这张表为什么长成这样，第 5 部分给出五个阶段上的具体数值 [computed: 见 f04 与 f13]。

## 3.2 架构与冻结的 API

输入是 observation dict，输出是 action chunk 加一个 value [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)。内部是两档结构：慢档负责语义与推理，快档负责动作生成。

参考实现取 GR00T N1.7 一类的形态：一个开源 VLM backbone，加一个独立设计的 DiT action head，公开规模是 538M 参数、16 层 [[arXiv:2607.15275]](https://arxiv.org/abs/2607.15275)。这也是本蓝图选定的起点——**沿用开源 VLM，action stack 自己做**。理由在 3.3 说明。

这里额外加一件成本极低的事：**implicit world modeling 作为辅助目标**。做法是让 policy 在生成动作的同时，去对齐未来观测的 latent 表示，只需给标准 VLA 加几个 token [[arXiv:2505.15659]](https://arxiv.org/abs/2505.15659)。它在多任务仿真基准上带来最高 26% 的提升，但对本蓝图更要紧的是另一个性质：**它让没有动作标签的第一人称人类视频也能参与 co-training** [[arXiv:2505.15659]](https://arxiv.org/abs/2505.15659)。第 3 部分的数据地图会直接用到这条通路。

## 3.3 Pretrain：先做大，这是刻意的

这一节的论证必须站得住，因为它对一个主打端侧的产品来说是反直觉的：**你不训练你打算出货的那个小模型** [[arXiv:2606.05737]](https://arxiv.org/abs/2606.05737)。

能力先在规模上出现，再通过蒸馏保留下来；同样的小架构从零训练到不了同一个位置 [[arXiv:2606.05737]](https://arxiv.org/abs/2606.05737)。端侧产品线上真正出货的模型规模落在 450M 到 690M 这个区间 [computed: SmolVLA 下界，RoboTTT 上界]，但它们的能力来自更大的教师，而不是来自在这个尺寸上直接堆数据。

配套要立一个可度量的目标，否则"端侧"永远是形容词。本蓝图用 **intelligence density**：每参数每瓦的任务成功率 [computed: 三项已披露量的组合]。它把 3.6 节和 3.7 节的工作和第 0 部分那块 700 Wh 电池连成同一条约束链 [computed: 0.7 kWh × 1000]。

需要坦白的是：目前**没有任何公开工作给出过 robot foundation model 的教师/学生规模配对** [computed: 检索未发现已披露配对]。这个数字由 P1 阶段自己定出来，它在第 6 部分的 gap ledger 里挂着。

起点的选择上还有两条被否掉的路，理由和触发条件一并写在这里 [[arXiv:2607.15275]](https://arxiv.org/abs/2607.15275)。

**直接 fork 一个开源 VLA 去做特化**，是到首个演示最快的一条路，代价是继承别人的 Episode Contract——而第 1 部分已经说明，这份 contract 决定了硬件规格 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。把它继承过来，等于让别人的传感器假设反向约束我们的板级迭代。这条路在 P0 阶段作为对照实现仍然有价值，但它不作为主线。

**连 VLM 一起从零预训练**，能拿到最完整的 IP 叙事，但它把早期资金投在一个已经被反复解决的问题上 [[blog: HuggingFace SmolVLA]](https://huggingface.co/blog/smolvla)。触发条件是具体的：当开源 VLM 的许可条款或者语义能力成为端侧成功率的瓶颈时重新评估——在那之前，这笔钱花在 action stack 和压缩链路上的回报更高。

## 3.4 Post-train

输入是精选高质量子集，输出是一个可用的任务策略。三件事：任务条件化、knowledge insulation（防止动作训练腐蚀 VLM 的语义能力）、以及 sim 与 real 的 co-training [[arXiv:2505.15659]](https://arxiv.org/abs/2505.15659)。

这一节短，因为它几乎全部继承自 3.1 节和 3.2 节。唯一值得记住的数字是每任务的数据地板：一个预训练充分的模型，用 50 到 100 条演示就能微调到可用 [[blog: DeepMind on-device]](https://deepmind.google/blog/gemini-robotics-on-device-brings-ai-to-local-robotic-devices/)。这个数字与采集侧文献给出的每任务 50 到 200 条演示落在同一区间 [[blog: dexset teleoperation guide]](https://dexset.ai/blogs/teleoperation-data-collection-robot-learning-complete-2026/)——两条独立证据链彼此吻合，说明这个地板是真实的。

## 3.5 Experience loop

输入是车队跑出来的 episode，带接管标记与结局；输出是一个更好的策略 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。做法是先从异构经验里学一个 value 函数，再用 advantage 条件化去训练策略。

信息增量在于信号的来源：**操作员按下接管的那一刻，就是最便宜的 reward signal**。它不需要标注成本，不需要设计奖励函数，而且它天然出现在真实分布上 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。第 1 部分把接管标记写进 Episode Contract，正是为了让这条通路在 P2 阶段不需要动硬件。

同样要说清它不适用的地方。experience loop 只能在策略已经尝试过的范围内提升质量，它扩不出这个范围 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。新行为、新场景、新物体仍然只能靠第 3 部分里的人类来源语料注入。把这一点讲反，是这类路线图最常见的过度承诺。

还有一个未知量：**接管到可测提升之间的转化率没有任何公开曲线** [computed: 检索未发现已披露转化曲线]。它决定 P2 需要多大车队才能闭环，同样挂在第 6 部分。

## 3.6 压缩链路

![压缩链路：左边是顺序，右边是验收](figures/f07-compression.png)

输入是大教师，输出是出货的学生。这一节的内容就是顺序，而顺序由实测排定，不由惯例排定 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。

按实测端到端收益排序：编译（TensorRT / torch.compile）给出 1.5 到 3.34× 且精度代价恰好为 0 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)；少步蒸馏把 10 步降到 1 步给出 3.3×，而且精度不降反升 [[arXiv:2606.05737]](https://arxiv.org/abs/2606.05737)；运行时加异步栈在 Jetson Orin 上给出 8.66× [[arXiv:2607.12659]](https://arxiv.org/abs/2607.12659)；视觉 token 剪枝给出 1.83×，天花板在 2.0× [[arXiv:2607.09520]](https://arxiv.org/abs/2607.09520)；单纯的 PTQ 量化给出 1.47 到 1.52× [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)。

由此得到两条要写进工程计划的结论。第一，**第一周就该做的是编译**：零精度风险，收益确定 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。第二，**量化买到的是显存，不是速度**——原因是通用工具链只量化语言 backbone，而边缘侧的瓶颈在 action head [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)。

验收标准是任务成功率的差值，不是任何代理指标。压缩悬崖的位置已经公开：4 bpw 几乎免费（96.6%），3 bpw 到 94.8%，2.5 bpw 是拐点（85.7%），2 bpw 直接崩塌到 48.0% [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)。最后一步就丢掉 37.7 pt [computed: 85.7 − 48.0]，所以本蓝图采用的规则是**停在 4 bpw**。

一条与基准文献相反的真实硅片证据值得单独记下：某自研 SoC 上出货的是 W8A16，并明确指出 W8A8 会掉成功率 [[arXiv:2606.07383]](https://arxiv.org/abs/2606.07383)。仿真基准和定制硅片在这里给出不同答案，蓝图选择相信硅片。

也要在这里就说明白：这一节和 3.7 节的证据基础比前面几节薄，也更贴近厂商自述 [[repo: FlashRT]](https://github.com/flashrt-project/FlashRT)。这不是缺陷，而是第 5 部分要讲的那件事——P3 正是这条路线上我们停止抄作业的地方。

## 3.7 Serving system

![两档速率，重叠执行](figures/f08-serving.png)

输入是压缩后的学生模型加一颗 SoC，输出是一份机器人能活在里面的延迟与功耗预算 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。

真正的硬指标是两档结构：策略跑在 5 到 30 Hz，下面接一个 50 Hz 到 1 kHz 的低层控制器，中间用 action chunk 桥接 [[arXiv:2604.24447]](https://arxiv.org/abs/2604.24447)。NVIDIA 自己给出的原则是，约 10 Hz 的推理速率足以支撑 30 FPS 的执行 [[spec: NVIDIA Jetson]](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/)。

延迟容忍度是一个结构条件，不是一个毫秒阈值。只要推理延迟不超过 H 减去执行 horizon，就永远有动作可用 [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)；π0.7 给出的对应披露值是 50 Hz 机器人上 240 ms [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483)。所以异步执行的意义在于让机器人永远不必等待，而不在于让单次前向更快。

硬件感知的核心结论只有一句：**batch-1 的 VLA 推理是 memory-bandwidth bound，优化目标是搬运的字节数，而不是 FLOPs** [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。一张表就能说明白——同一个 π0 前向，在 Jetson Thor 上视觉 6.06 ms、VLM 20.30 ms、action head 26.20 ms，端到端 52.57 ms；action head 占了 50%，而在 RTX 4090 上同一部分只占 23% [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。边缘侧最大的那一项，恰恰是通用量化工具不碰的那一项。

已公开的最好端侧成绩来自手写 kernel：π0.5 在 Jetson AGX Thor 上做到 44.0 ms、23 Hz，三路视图用 NVFP4 做到 39.78 ms [[repo: FlashRT]](https://github.com/flashrt-project/FlashRT)。这个数字越过了解析屋顶线给出的 19.0 Hz [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)，说明解析模型本身偏保守。两支团队各自放弃编译器改写 kernel 并且都赢了——这是 P3 值得投入原创的最强证据。

最后把功耗接回电池。目前唯一可引用的端侧整机功耗是某方案在 AGX Orin 上的 40 W [[arXiv:2604.24447]](https://arxiv.org/abs/2604.24447)，硅片不同，只能作为量级参考；**我们自己的功耗目标是未披露量**，由 P3 实测确定 [computed: 无同类已披露数据]。

## 3.8 Test-time adaptation 与 context scaling

最后一节讲一件容易被误判的事。直觉上，test-time 计算与端侧预算天然冲突：多花的推理算力，机器人用电池来付 [[arXiv:2604.24447]](https://arxiv.org/abs/2604.24447)。

实测结论与直觉相反。把 visuomotor context 扩到 8K 个时间步——比现有策略高三个数量级——**推理延迟不增加** [[arXiv:2607.15275]](https://arxiv.org/abs/2607.15275)。代价落在参数上：每层 DiT 加约 10M 参数的 TTT 层，16 层合计把 action head 从 538M 抬到 690M，增幅 28% [computed: (690−538)/538]。

对端侧来说这恰好是想要的那种交换：参数是一次性的显存成本，延迟是每步都要付的成本。收益也不小——比单步 context 基线提升 87%，8K context 比 1K 预训练再高 62%，并且能完整走完一个 5 min、10 阶段的装配任务，而所有基线都走不完 [[arXiv:2607.15275]](https://arxiv.org/abs/2607.15275)。

所以本蓝图把 context scaling 放在 P4 而不是无限期推迟，并且明确它属于"先付参数、后省延迟"的一类优化 [[arXiv:2607.15275]](https://arxiv.org/abs/2607.15275)。

---

# 第 3 部分 —— Data Flywheel

## 4.1 数据地图：两个轴，以及作为向量的操作

![数据地图：位置决定一份语料能训模型的哪一部分](figures/f04-data-map.png)

先说清楚为什么不按"来源"列清单。按来源列出遥操作、人类视频、仿真、公开数据集，看上去整齐，但它把三件互不相关的事混在一根轴上：数据从哪来、我们对它做了什么、它是谁在什么时候产生的 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。真正决定一份语料用途的，是下面两个属性。

**第一个轴是 action grounding**：这份数据里有没有我们这台机器人动作空间中的动作。取值从"没有"、"人手"、"夹爪代理"、"跨本体机器人"，一直到"我们自己的本体" [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。它决定这份语料**能训模型的哪一部分**。

**第二个轴是 policy relatedness**：数据是人产生的 off-policy 数据，还是当前策略自己跑出来的 on-policy 数据 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。它决定这份语料**能不能支撑 value 学习**，还是只能做行为克隆。

真实性（真实、重建、仿真、生成）不作为第三个轴，它是打在每个点上的标记——它影响的是可信度，不是资格 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。

| 区域 | 语料举例 | 能训什么 | 训不了什么 |
|---|---|---|---|
| 无 grounding，off-policy | 图文语料、人类视频 | 表征、语义、任务结构 | 动作空间里的任何东西 |
| 人手 grounding | 第一人称可穿戴 | 动作先验、affordance | 接触力、我们的运动学 |
| 夹爪代理 grounding | 手持采集装置 | 加 adapter 后的动作预训练 | 我们的全自由度、全身 |
| 跨本体机器人 | 公开机器人数据集 | 归一化后的动作预训练 | 我们本体的特有部分 |
| 我们的本体，off-policy | 我们的遥操作 | post-train、行为克隆 | value 函数、advantage |
| 我们的本体，on-policy | 我们的 rollout 与接管 | value 学习、experience loop | 当前能力包络之外的新行为 |

**操作是这张图上的向量。** 每一个操作都把数据从便宜的区域搬向昂贵的区域，都有明确的成本，也都有明确的失真 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)：

- **从间接语料里提取信号**：把巨量但与机器人无关的语料，通过过滤与标注提取出可用的语义、affordance 与任务结构。失真在于它造不出它从未观测到的通道——没有力矩，没有接触，也没有我们动作空间里的动作 [[arXiv:2505.15659]](https://arxiv.org/abs/2505.15659)。3.2 节那个 implicit world modeling 的辅助目标，就是让这条向量真正落地的机制。
- **重建后重采样**：把一次性的真实观测变成可以无限采样的生成器。它的价值不是画面好看，而是 **rank fidelity**——它给策略排的序，和真实世界给策略排的序是否一致 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。这条向量自带验证义务，第 4 部分专门讲。
- **sim co-training 与 world-model rollout 合成**：不花机器人时间，把质量往 on-policy 一侧搬。失真是动力学 gap [[arXiv:2602.15922]](https://arxiv.org/abs/2602.15922)。
- **embodiment adapter 与动作归一化**：把质量沿 grounding 轴往上搬，是这张图上最便宜的一条向量，也是跨本体公开数据之所以有价值的全部原因 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。

整个第 3 部分要做的事，一句话就能概括：**把概率质量搬到当前训练阶段需要的那个区域，并让单位有效信号的成本最低** [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

有一条边界必须写死：**我们自己本体上、带动作 grounding 的真实数据，是必需品，这张图上没有任何操作能把它制造出来** [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。每一条向量最终都要拿它当锚点，而第 4 部分的 rank fidelity 验证离了它根本无法进行。

![哪一类数据喂哪一个阶段](figures/f05-info-flow.png)

把上面的区域投影到 Model Factory 的流水线上，就得到这张图。它是整份蓝图里最实用的一张：图文与人类视频喂 pretrain 的表征；可穿戴喂动作先验与场景广度；手持装置喂接近动作空间的操作先验；公开机器人数据归一化后喂动作预训练；我们的遥操作喂 post-train 的动作 grounding；我们的 rollout 与接管喂 experience loop 的 on-policy 分布与 value；重建场景喂 evaluation [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

## 4.2 采集

![机上分流：车队回传不了自己录的东西](figures/f10-triage.png)

输入是运行中的机器人，输出是入库边界上一批符合 contract 的 episode [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。这里最容易给出错误答案，因为直觉解法在算术上就不成立。

单台机器人的未过滤传感数据率是 35.665 MB/s，在线压缩后是 0.213 MB/s，压掉 99.4% [[blog: Trossen robotic data pipeline]](https://www.trossenrobotics.com/post/robotic-data-pipeline-sensor-streams-to-training-datasets)。换算成小时是 128 GB/h [computed: 35.665 MB/s × 3600]，单机一天约 3 TB/day [computed: 128 GB/h × 24 h]；按 300 台车队、每天 8 h 班次算，是 307 TB/day [computed: 128 GB/h × 8 h × 300]。没有任何回传链路吃得下这个量。

所以分流必须在机器人本体上做，分四层：所有传感数据先进一个很短的环形缓冲；只有被触发标记的片段才落盘；落盘的片段在机上编码；充电时择机上传 [[blog: Trossen robotic data pipeline]](https://www.trossenrobotics.com/post/robotic-data-pipeline-sensor-streams-to-training-datasets)。触发条件有四类——接管、失败、novelty 分数超阈，以及一份随机配额。

那份随机配额不是可选项。**没有它，留下来的数据将全部由失败构成，模型学到的是一个永远出错的世界** [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。这是分流设计里唯一一个反直觉的地方，也是最容易在工程实现中被砍掉的地方。

Episode Contract 在这一层强制执行，理由第 1 部分已经讲过：入库端只能拒收，不能修复 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。

关于数据权利：车队工作在零售与展厅场景，会录到与业务无关的路人。人脸与音频的机上处理、留存期限、场景方同意，这些是有排期影响的工程需求，不是法务附录。可参照的做法是把人物过滤、视角刻画、质量控制与隐私审查都当作一等设计目标写进采集管线 [computed: 大规模人类视频语料的公开策展做法]。

## 4.3 清洗与 QA

输入是原始 episode，输出是带 trust score 的 episode。校验项包括时钟偏移、丢帧、关节越界、动作与状态不自洽，外加一份具名的失败分类法和近重复检测 [[blog: Trossen robotic data pipeline]](https://www.trossenrobotics.com/post/robotic-data-pipeline-sensor-streams-to-training-datasets)。

规则只有一条：**只打分，不删除**。删除是用不完整信息做出的不可逆决策——今天判定为噪声的片段，可能正是明年某个失败模式的唯一样本 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。存储很便宜，重录不可能。

一个可以拿来校准量级的公开比值：某大规模真实家庭人形数据集是 500 h、23000 条 episode、10 TB 原始数据 [[blog: Humanoids Daily HIW-500]](https://www.humanoidsdaily.com/news/bitrobot-and-hugging-face-drop-hiw-500-a-massive-10tb-real-home-humanoid-dataset)，也就是每记录小时约 20 GB [computed: 10 TB ÷ 500 h]——与压缩后的速率同量级，与未压缩的速率差两个数量级。

## 4.4 标注

输入是打过分的 episode，输出是一层标签。内容包括 VLM 自动标注加人工抽检、子任务分段、成功与 reward 标签 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

信息增量在结构上：**标签是挂在不可变 episode 上的一层可变数据**。换一套任务分类法重新标注，成本只有计算，不作废任何一条原始数据 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。把标签写回 episode 本身，等于让每一次标注体系升级都变成一次数据迁移工程。

## 4.5 存储与版本

输入是 episode 加标签，输出是一份 **manifest**——一个 episode id 列表，加上标签版本，加上配比权重 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。episode 存储按内容寻址且不可变。

每一个模型版本都钉住一份 manifest。这既是结果可复现的技术前提，也是让累积数据成为可审计资产而不是一堆文件的关键：工程意义上，那份资产就是这套 manifest 加不可变存储 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

## 4.6 采样与加载

输入是 manifest，输出是能把 GPU 喂饱的 batch。3.1 节的配比权重在这一层落地为一个采样器 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

这里有一个几乎总被跳过的工程事实：**饿死大规模机器人学习训练的是 video decode，不是算力** [[blog: Trossen robotic data pipeline]](https://www.trossenrobotics.com/post/robotic-data-pipeline-sensor-streams-to-training-datasets)。所以加载器设计与 latent 预缓存属于蓝图正文，而不是脚注。按每记录小时 20 GB 的量级 [computed: 10 TB ÷ 500 h]，解码吞吐会先于显存和算力成为瓶颈。

## 4.7 闭环

![数据飞轮：每一步交付一个具名产物](figures/f09-flywheel.png)

输入是一个候选模型，输出是一个在役策略以及它产生的 telemetry。链路是：evaluation 放行 → 模型注册表 → 灰度再分批 OTA → 影子模式对比 → telemetry 回流到 4.1 节 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

**rollback 是一等操作**，与发布同等重要。一个不能在分钟级撤回策略的车队，不敢做灰度；不敢做灰度，就只能靠离线指标决定上线，而第 4 部分会说明离线指标为什么撑不起这个决定 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。

telemetry 回流那条边，决定的是下一轮采什么。这条边如果断了，飞轮就退化成一条单向流水线——数据仍在增加，但增加的是模型已经会做的那部分 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。

---

# 第 4 部分 —— Evaluation，两个工厂之间的器官

![评估器官：重建一次，廉价筛选，真机只花在幸存者身上](figures/f11-eval.png)

这一部分只有一个论点，但它决定其余所有部分能不能推进：**真机试验的统计功效撑不起一个 gate** [computed: two-proportion test]。

把一个 50% 的成功率和一个 60% 的成功率区分开，按双比例检验需要约 387 次试验 [computed: two-proportion test]。而公开工作里每个 checkpoint 配的真机试验是 100 次 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。换句话说，仅凭真机结果，你连"这一版比上一版好 10 个点"都判定不了——更不用说 3 个点。

于是 sim 筛选就从一种省钱手段变成了必需品。结构是三段：每个部署场景重建一次；每个 checkpoint 在重建场景里跑 2000 次仿真试验；只有通过筛选的 checkpoint 才消耗那 100 次真机试验 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。仿真与真机的试验数比是 20 倍 [computed: 2000 ÷ 100]。

同一来源还报告了三件更强的事：完全不用真实数据训练的策略迁移到了 5 个不同平台；策略能连续自主运行 1 h 无人接管；而且仿真**保持了策略之间的排序**，训练进度曲线一致，空间上的成功与失败模式也对得上 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。

必须标注清楚：以上来自企业博客，没有论文，也没有独立复现 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。所以本蓝图**不假设**排序保持成立，而是在 P1 阶段用我们自己的场景把它测出来。

这就引出这个器官自己的度量：**rank fidelity**——仿真给出的策略排序，与真机排序的相关性。它必须被持续测量，而不是被假定 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。本蓝图取 0.80 作为可用于筛选的下限 [computed: Spearman floor for screening use]。

理由很直接：一个画面精美但会打乱排序的生成器，比没有生成器更糟——它会以很高的置信度把错误的 checkpoint 推上线 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。评估器官在被信任之前，必须先证明自己值得信任。

如果 P1 测出来的相关性达不到这个下限，路线图并不因此中止，而是换一条更贵的通道：sim 退化为失败搜索工具（找出策略在哪些条件下会坏），排序仍然由真机决定，代价是每个 checkpoint 的评估周期按 387 次试验的量级重新估算 [computed: two-proportion test]。这条备用通道之所以要在这里写明，是因为它决定 P2 的节奏——先把它规划进去，比在 P1 结束时才发现要好。

---

# 第 5 部分 —— 路线图

![路线图：按顺序排列，不按日期排列](figures/f12-roadmap-public.png)

五个阶段**按顺序排列，不按日期排列**。每个阶段用它消掉的风险来定义，用一个客观的放行条件来结束 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。这样承诺的是次序和证据，而不是一个需要被反复辩护的日历。


有两个次序选择是可以被质疑的，所以在这里明说

**评估建在模型之前。** P0 交付一台仪器，没有任何能力演示，这在一份提案里是很不舒服的开局。它仍然是对的：后面每一个放行条件都用评估的单位来表述，而一个无法测量自己是否在转的飞轮，和一个根本没转的飞轮，从外部看完全一样 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。把它放在第一位，等于把这条路线上最硬的约束变成第一个被消掉的风险，而不是最后一个被发现的意外。

**端侧排在飞轮闭合之后。** 对一个还在每周变化的模型做压缩，意味着整条压缩链路要反复重做，精度差值要反复重测 [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)。所以 P1 与 P2 期间明确采用离机或本地机房算力，这个过渡形态写在明处而不是藏起来。反方向的论据是商业上的——机上自主是最有说服力的演示——所以这里给出的是权衡而不是结论：如果演示价值压过工程返工成本，P3 可以提前，代价是压缩链路要多做一到两轮。

还有一件事要说在前面：**P0 到 P2 抄的是已经公开且被复现过的配方**，在这些地方做原创买不到任何东西，只会付出排期 [[arXiv:2604.15483 §3]](https://arxiv.org/abs/2604.15483)。**P3 是我们停止抄作业的地方**——它的证据基础更薄、更贴近厂商自述，同时也正因如此，它是这条路线上原创工作真正不可避免的一段 [[repo: FlashRT]](https://github.com/flashrt-project/FlashRT)。

![配比如何随阶段迁移](figures/f13-mixture-shift.png)

最后回答第 3 部分留下的那个问题：既然数据地图不是一个建设顺序，那阶段之间到底变的是什么？变的是配比。公开与人类来源语料从 P0 的 92 降到 P4 的 30；自有遥操作从 8 升到 P2 的峰值 33，再回落到 22；on-policy rollout 从 0 一路升到 48 [computed: 各阶段可得数据源的组合]。

三条曲线里最重要的是第三条。它从零开始，在 P2 飞轮闭合时才出现，之后成为最大的一块——**这是整条路线上唯一一个成本随算力和车队规模扩张、而不随人员编制扩张的数据来源** [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。P4 的放行条件之所以写成"不依赖操作员线性增长的自我提升"，指的就是这条曲线。

同时要记住 3.5 节那条边界：on-policy 数据只在既有包络内提升质量，扩不出新包络 [[arXiv:2511.19647]](https://arxiv.org/abs/2511.19647)。所以前两条曲线在 P4 仍然占到一半以上，它们的角色从"主力"变成"新颖性注入"，但它们不会归零。

---

# 第 6 部分 —— 我们还不知道什么

一份点名了自己未知项、并且给每一项配上了关掉它的那个实验的路线图，比一份处处自信的路线图更值得相信。以下八项都从 `FACTS.md` 的 gap 组直接来 [computed: 本模块 FACTS.md 的 GAP 组]。

- **教师规模。** 没有任何公开工作给出过 robot foundation model 的教师/学生规模配对 [computed: 检索未发现已披露配对]。由 P1 的蒸馏消融确定。
- **机上功耗。** 唯一可引用的同类数字来自不同硅片上的 40 W [[arXiv:2604.24447]](https://arxiv.org/abs/2604.24447)。由 P3 实测确定。
- **零售与展厅场景下的 rank fidelity。** 已公开结果覆盖的是桌面操作 [[blog: World Labs real-to-sim-to-real]](https://www.worldlabs.ai/blog/real-to-sim-to-real)。由 P1 在我们自己的场景上测出。
- **我们自己的压缩悬崖位置。** 已公开的悬崖属于另一个模型在另一个基准上 [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)。由 P3 的逐档扫描确定。
- **接管到可测提升的转化率。** 没有公开曲线 [computed: 检索未发现已披露转化曲线]。由 P2 用版本间对比测出，它同时决定 P2 需要多大的车队。
- **这个规模的车队能否产生足够的 on-policy 数据。** 车队规模与提升幅度之间的关系没有公开数据 [computed: 检索未发现已披露关系]。P2 的直接产出。
- **策略跨场景类型的迁移能力。** 车队尺度上没有被测量过 [computed: 检索未发现已披露测量]。P4 的主要问题。
- **photon-to-torque 全链路延迟。** 每一份已发表的延迟拆解都缺这一段 [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。P3 的仪器工作。

## Open bet：world action models

本蓝图整体押在 VLA 路线上，这是一个选择，应该被当作一个可以被推翻的赌注来陈述 [[arXiv:2602.15922]](https://arxiv.org/abs/2602.15922)。

另一条路线是 world action model：在预训练的视频扩散模型上，通过预测未来世界状态与动作来学习物理动力学。已报告的真机结果是对新任务与新环境的泛化能力超过同期 VLA 的 2 倍 [[arXiv:2602.15922]](https://arxiv.org/abs/2602.15922)。这个数字很难忽视。

暂不押注的理由是端侧，不是能力：该模型规模为 14B，闭环控制率为 7 Hz [[arXiv:2602.15922]](https://arxiv.org/abs/2602.15922)。这个规模比一台 0.7 kWh 的人形机器人能在机上服务的量级高一个数量级 [[blog: carnewschina 2026-04-13]](https://carnewschina.com/2026/04/13/chery-begins-online-sales-of-humanoid-robot-with-a-0-7-kwh-battery-at-41400-usd/)，而这个速率也低于两档架构对快档的要求 [[arXiv:2604.24447]](https://arxiv.org/abs/2604.24447)。

所以触发条件写得很具体：**当出现一个压缩后能进入机上延迟与功耗预算的 world action model 时，重新评估资源分配** [[arXiv:2602.15922]](https://arxiv.org/abs/2602.15922)。这是一个可证伪的条件，而且它挂在别人的路线图上——这正是一个 open bet 应该有的样子。

---

## Bibliography

- *RoboTTT: Context Scaling for Robot Policies* — arXiv:2607.15275. https://arxiv.org/abs/2607.15275
- *FLARE: Robot Learning with Implicit World Modeling* — arXiv:2505.15659. https://arxiv.org/abs/2505.15659
- *World Action Models are Zero-shot Policies (DreamZero)* — arXiv:2602.15922. https://arxiv.org/abs/2602.15922
- *Real-Time Chunking (RTC)* — arXiv:2506.07339. https://arxiv.org/abs/2506.07339
- *π0.7 technical report* — arXiv:2604.15483. https://arxiv.org/abs/2604.15483
- *VLA-Perf* — arXiv:2602.18397. https://arxiv.org/abs/2602.18397
- *ActQuant* — arXiv:2605.24011. https://arxiv.org/abs/2605.24011
- *RhinoVLA* — arXiv:2606.07383. https://arxiv.org/abs/2606.07383
- *Jetson-PI* — arXiv:2607.12659. https://arxiv.org/abs/2607.12659
- *Let It Be Simple* — arXiv:2606.05737. https://arxiv.org/abs/2606.05737
- *Characterizing VLA Models across XPUs* — arXiv:2604.24447. https://arxiv.org/abs/2604.24447
- *Energy characterization of VLA inference* — arXiv:2607.09520. https://arxiv.org/abs/2607.09520
- *Robot-Powered Data Flywheels* — arXiv:2511.19647. https://arxiv.org/abs/2511.19647
- *Real-to-Sim-to-Real*, World Labs — https://www.worldlabs.ai/blog/real-to-sim-to-real
- *The Robotic Data Pipeline*, Trossen Robotics — https://www.trossenrobotics.com/post/robotic-data-pipeline-sensor-streams-to-training-datasets
- *Humanoids-in-the-Wild 500*, Humanoids Daily — https://www.humanoidsdaily.com/news/bitrobot-and-hugging-face-drop-hiw-500-a-massive-10tb-real-home-humanoid-dataset
- *Teleoperation Data Collection: 2026 Guide* — https://dexset.ai/blogs/teleoperation-data-collection-robot-learning-complete-2026/
- *Humanoid Robot Data Collection Costs*, DataX Power — https://www.dataxpower.com/blog/humanoid-robot-data-collection-cost
- *Gemini Robotics On-Device*, Google DeepMind — https://deepmind.google/blog/gemini-robotics-on-device-brings-ai-to-local-robotic-devices/
- *SmolVLA*, Hugging Face — https://huggingface.co/blog/smolvla
- *Chery begins online sales of humanoid robot*, CarNewsChina — https://carnewschina.com/2026/04/13/chery-begins-online-sales-of-humanoid-robot-with-a-0-7-kwh-battery-at-41400-usd/
- *FlashRT* — https://github.com/flashrt-project/FlashRT
- *Isaac-GR00T* — https://github.com/NVIDIA/Isaac-GR00T
