---
title: "如何构建机器人基础模型 —— 可能的路线与挑战"
date: 2026-07-31
draft: false
tags: ["物理AI", "具身智能", "VLA", "机器人", "基础模型", "边缘计算"]
summary: "一条第一性原理的推导链：机器人基础模型是什么、由什么组成、为什么大语言模型那套机器搬不过来、这个搬不过来在每个子系统里各要付多少代价，以及在这种前提下应该按什么顺序去造。"
---

## 这篇文章要回答的问题

> 如果真要造一个机器人基础模型，该怎么造？什么会挡住你？

推导链只有一条。先按「它必须做到什么、必须跑在哪里」把这个对象定义清楚，再把它拆成大脑和身体两个子系统。然后把造出大语言模型的那台机器搬过来，把它的三根支柱说准，逐根拿去对照机器人——机器人只占住了其中一根，另外还多背了一条文本从来没遇到过的约束。五个卡点由此浮出来。每个卡点都要在某个具体子系统里用真金白银的工程量去填。把这些工程量排好序，就是施工顺序。

结论先摆出来，方便你直接反驳：算力应该放在机器人身上；真正难的部分大多属于测量和机构，跟建模关系不大；最先该动手造的东西并不是模型。

## 怎么读这篇文章

语言模型那部分背景——transformer、预训练、规模化定律、计算强度——集中在第 3 部分讲，别处不再展开。已经熟悉的可以直接跳过，前面几部分不依赖它 [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361)。

全文贯穿三条约定，它们是承重的。

**每条论断都标了类型。** *disclosed* 指某个一手来源明确写了；*inferred* 指我们从已披露事实推出来的，并且会说明推法；*undisclosed* 指实验室外没人知道，那就送进 gap 清单，不含糊带过；*marketing* 指公司博客或新闻稿，既没有论文也没有第三方复现，每次出现都会当场标出来——这个领域里新闻稿和真实结果之间的距离大得异乎寻常 [[blog: NVIDIA GEAR GR00T N1.5]](https://research.nvidia.com/labs/gear/gr00t-n1_5/)。

**每个数字都带着出处**，并且由脚本对着一份 fact ledger 机器校验。正文里出现的数字只要在 ledger 里查不到，或者哪一段没有引用，构建脚本就会直接失败 [[arXiv:2604.15483 §V-E]](https://arxiv.org/abs/2604.15483)。

**证据到头的地方，文章会直说**，而不是顺手写一句听起来合理的话。第 6 部分整体是推测性的，并已明确标注 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483)。

---

# 第 0 部分 —— 两个数字

一台从没见过任何叠衣服演示数据的双臂 UR5e，把一件衬衫叠到了 **85.6% 的任务进度、80% 的成功率** [[arXiv:2604.15483 §IX]](https://arxiv.org/abs/2604.15483)。驱动它的模型，训练数据来自别的机器人在做别的事情。

同一个任务交给十位专家级遥操作员，每人约有 375 小时经验，从操作员里按前 2 百分位挑出来。他们第一次上手这条机械臂，拿到的是 **90.9% 的进度、80.6% 的成功率** [[arXiv:2604.15483 §IX]](https://arxiv.org/abs/2604.15483)。一个没有任何任务专用数据的通用模型，成绩落在专家第一次碰这套硬件的水平上，差距不到一个百分点。

这是第一个数字。第二个数字是：那个模型跑在一块 H100 上 [[arXiv:2604.15483 App. D]](https://arxiv.org/abs/2604.15483)；而一个在某个相机位姿下训到 100% 成功率的策略，把相机挪 10 厘米、转 20 度，成功率掉到 **0%** [[arXiv:2409.03403]](https://arxiv.org/abs/2409.03403)。

泛化是真的。它的脆弱是机械层面的。

---

# 第 1 部分 —— 机器人基础模型是什么

![四条能力轴，以及公开记录能支撑到哪一步](figures/f11-capability-axes.png)

## 四条轴，不止一条

这个词被用得太随意，值得先钉死。机器人基础模型是同时满足四项要求的系统，而有意思的地方在于：这四项，领域内满足得很不均匀 [[arXiv:2108.07258]](https://arxiv.org/abs/2108.07258)。

- **任务泛化** —— 用一句话把它 prompt 到新任务上。换活儿意味着写一句新指令，不用重新采数据、重新起训练。评测覆盖 14 个场景，每个场景 3 到 6 条开放式指令，环境是没见过的厨房和卧室 [[arXiv:2604.15483 §IX-B]](https://arxiv.org/abs/2604.15483)。
- **本体泛化** —— 用一句话把它 prompt 到新机体上。同一套权重驱动运动学不同的机器人。在 22 种本体、60 个数据集的规模上，跨本体模型平均比逐数据集训练的专用方法高出 **50%** [[arXiv:2310.08864]](https://arxiv.org/abs/2310.08864)。
- **类人交互** —— 接得住开放式指令，并且把自己打算干什么暴露出来。π 系列会输出一串人可读的子任务描述，每 4 秒刷新一次 [[arXiv:2604.15483 §VI]](https://arxiv.org/abs/2604.15483)；显式的中间推理把任务进度从 **0.55 抬到 0.67** [[arXiv:2510.03342]](https://arxiv.org/abs/2510.03342)。
- **持续进化** —— 从自己的经验里变强，跟着机体磨损调整，接得住新派给它的活。这条轴弱得非常明显：靠自主 rollout 驱动自我改进，公开的闭环只有一个 [[arXiv:2511.14759]](https://arxiv.org/abs/2511.14759)；至于适应机械磨损，没有任何一家发表过 [[arXiv:2104.08212]](https://arxiv.org/abs/2104.08212)。

第四条轴才是把模型变成产品的那条，也恰恰是至今没人交付出来的那条。记住它，第 6 部分会绕回来。

## 输入输出契约

![输入输出契约](figures/f01-io-contract.png)

把术语剥掉，这个对象就是一个函数。

**输入：** 最多 4 路相机图像，分辨率 448×448；最多 6 帧近期历史；当前关节构型；一句英文指令 [[arXiv:2604.15483 §IV]](https://arxiv.org/abs/2604.15483)。

**输出：** 一个 *action chunk* —— 一段长度 H = 50 的未来关节目标序列，其中只有前 15 或 25 步会被真正执行，然后模型被重新调用 [[arXiv:2604.15483 §VI-B]](https://arxiv.org/abs/2604.15483)。控制回路在多数平台上跑 50 Hz，在 UR5e 上跑 20 Hz [[arXiv:2604.15483 §VII]](https://arxiv.org/abs/2604.15483)。

整个接口就这么多。模型是一个 setpoint 生成器：它不接管你的伺服环，它给伺服环喂数 [[arXiv:2604.15483 §VII]](https://arxiv.org/abs/2604.15483)。有两个性质比看上去重要。它一次预测大约一秒的运动，却只承诺其中三分之一，省下来的时间正好用来思考 [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)；而适配新任务的通道是那句话——换任务等于换指令，不用换数据集 [[arXiv:2604.15483 §V]](https://arxiv.org/abs/2604.15483)。

chunk 不是实现细节，背后的消融结果相当刺眼：一次只预测一步，在一个精细双臂任务上成功率 **1%**；一次预测 **100** 步，成功率 **44%** [[arXiv:2304.13705]](https://arxiv.org/abs/2304.13705)。机理是误差累积——一点偏差把机器人推到训练分布边缘之外，下一步预测就更差一点，循环发散。一次承诺一整段运动，等于把发散的机会数直接除掉一个量级 [[arXiv:2304.13705]](https://arxiv.org/abs/2304.13705)。

## 把基础模型和大策略区分开的那个消融

「用了很多数据训出来的」和「基础模型」是两个不同的论断，有一个实验能干净地把它们分开。把网络预训练从跨本体模型里抽掉，它在 emergent skills 上得 **0%**、在泛化上得 **1%**；把预训练加回去，同样的架构拿到 **48.7%** 和 **47%** [[arXiv:2310.08864 Table II]](https://arxiv.org/abs/2310.08864)。

语义是继承来的，机器人数据教不出来。这篇文章后面所有推导都建立在这个事实上，因为它是机器人模型唯一能站在互联网已经付过的账上起步的理由 [[arXiv:2307.15818]](https://arxiv.org/abs/2307.15818)。

## 哪几条轴真的被证明了

「泛化」这个词在机器人写作里承担了太多没兑现的工作。各条轴上的证据厚薄差很多，值得逐条说清楚。

- **新物体。** 多家实验室都做出来了——在没见过的物体和背景上，大致是上一代的两倍，覆盖 280 个任务、6 千次评测 [[arXiv:2307.15818]](https://arxiv.org/abs/2307.15818)。
- **新场景。** 这是领域内最硬的单点结果。训练数据从 3 个地点，扩到 12、22、53、82，最后到 104 个地点，性能单调上升；104 地点的模型大致追平了直接在测试住宅里训练的对照模型 [[arXiv:2504.16054]](https://arxiv.org/abs/2504.16054)。
- **跨本体。** 证明了，但天花板有据可查：emergent skills 上 **75.8%** 对 **27.3%** [[arXiv:2310.08864]](https://arxiv.org/abs/2310.08864)。
- **长程组合。** 在 10 到 15 分钟量级的任务上证明了，比如多阶段房间整理 [[arXiv:2504.16054]](https://arxiv.org/abs/2504.16054)。
- **新的物理条件** —— 摩擦、负载、惯量、柔顺性、刚度。**这是最弱的一条轴，没有任何实验室发表过在这些变量上的扫描** [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483)。光照和杂乱度有人测，机械变量没人测。

有两件事应该给上面所有内容降降温，而且你最好从这里听到，别以后自己撞上。独立工作发现，当前最强的一批模型存在词汇—运动学捷径和语义特征坍缩，静态 benchmark 把这种退化盖住了 [[arXiv:2604.18000]](https://arxiv.org/abs/2604.18000)，也记录了视觉直接压过语言指令的情形 [[arXiv:2602.17659]](https://arxiv.org/abs/2602.17659)。而领域内领先的实验室自己也承认：在它那个数据规模上，「实际上很难确切判定哪些任务是真的见过、哪些没见过」，模型「很可能主要是靠把别处情境里的技能重新组合」来实现泛化的 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483)。

## 算力应该放在哪里

![云端服务与端侧部署，四个维度的对照](figures/f10-product-form.png)

这是个产品决策，而且通常被往后拖。它不该被往后拖，因为它决定了你被允许造什么东西。两个选项：一个中心服务，机器人隔着网络去查；或者一个直接装在机器上的模型。

**延迟。** 离机推理要在 73 毫秒模型时间之上再加大约 13 毫秒 Wi-Fi，合计 86 毫秒 [[arXiv:2410.24164]](https://arxiv.org/abs/2410.24164)。这个数扛得住——但扛得住的原因是有专门为此设计的算法。实时 chunking 在注入 100 毫秒和 200 毫秒延迟时没有可测的性能损失，超过 300 毫秒就崩 [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)。与此同时，当代模型挂四路相机，在单块 H100 上已经要 300 毫秒 [[arXiv:2511.14759]](https://arxiv.org/abs/2511.14759)。网络预算在网络还没接进来之前就花光了。

**功耗。** H100 是 700 W [[spec: NVIDIA H100 SXM]](https://www.nvidia.com/en-us/data-center/h100/)，Jetson Thor 是 40 到 130 W [[spec: NVIDIA Jetson AGX Thor]](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/)。给个尺度感：一台 842 Wh 电池、续航 4 小时的人形机器人，*整机*平均功率约 210 W [[spec: 1X NEO]](https://www.1x.tech/neo)，这意味着一块边缘计算模组要吃掉**整机功率预算的 19% 到 62%** [computed: 40 to 130 W against 210 W]。把 H100 装到机器人身上不是功耗问题，是物理上做不到。

**单台成本。** 一块边缘模组 \$3,499 [[spec: NVIDIA Jetson AGX Thor]](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/)，对面是一块数据中心 GPU——*每台机器人一块*，而且一直要付 [[spec: NVIDIA H100 SXM]](https://www.nvidia.com/en-us/data-center/h100/)。

**故障形态。** 策略跑在服务器上的机器人，断网的那一刻就不再是机器人了。这一条属于推理，不是实测结论，此处如实标注——但这是买家会算的账，研究者不会算 [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)。

**结论是端侧**，附一个诚实的限定：今天的前沿模型装不进去。同一个模型、同一套运行时，在 H100 上跑 35.9 Hz，在 Jetson Thor 上 10.7 Hz，在 AGX Orin 上 4.6 Hz [[repo: Isaac-GR00T hardware_recommendation.md]](https://github.com/NVIDIA/Isaac-GR00T/blob/main/getting_started/hardware_recommendation.md)。正是这个差距，让第 5 部分里的压缩工作变成承重结构，而不是可选项。

---

# 第 2 部分 —— 两个子系统：大脑与身体

![系统分解，每条边都标了传什么、多快](figures/f12-brain-body.png)

## 大脑

一个通用的视觉—语言—动作模型。它读场景，写 setpoint。它必须具备的能力：

- **视觉、语言、本体感知。** 每个已落地的系统都有——图像、关节构型向量、一句指令 [[arXiv:2604.15483 §IV]](https://arxiv.org/abs/2604.15483)。有代表性的平台暴露的是 14 维状态 [[repo: openpi aloha_policy.py]](https://github.com/Physical-Intelligence/openpi/blob/main/src/openpi/policies/aloha_policy.py)。
- **推理。** 口号意义上的「会思考」不算数，这里指的是一个后续系统真的会消费的显式中间表示。π 系列每 4 秒刷新一次 [[arXiv:2604.15483 §VI]](https://arxiv.org/abs/2604.15483)；在做过消融的地方，它值 **0.55 到 0.67** 的任务进度 [[arXiv:2510.03342]](https://arxiv.org/abs/2510.03342)。
- **力与触觉。** 研究系统里测得到，前沿模型里一个都没接。加一路力信号平均涨 **23.2%**，插拔任务从约 **10%** 抬到约 **80%**，训练集只有 244 条轨迹 [[arXiv:2505.22159]](https://arxiv.org/abs/2505.22159)。触觉把 USB 插接从 **5% 抬到 35%**，充电器插接从 **40% 抬到 90%**，分布外擦拭从 **0% 抬到 80%** [[arXiv:2507.09160]](https://arxiv.org/abs/2507.09160)。
- **音频。** 没有任何前沿机器人基础模型公开过音频输入通路。这里只把它标成一个缺口，不展开 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483)。
- **动作生成。** 表示形式的选择，第 3 部分会论证它是中枢性的一条轴 [[arXiv:2501.09747]](https://arxiv.org/abs/2501.09747)。

## 身体

这几条要求读起来像一份硬件规格书，因为它本来就是。

- **灵巧与精细控制。** 研究级机械臂接受 1 kHz 的指令 [[spec: Franka Research 3]](https://franka.de/products/franka-research-3)，而它上面那个策略产出 setpoint 的频率是 50 Hz [[arXiv:2604.15483 §VII]](https://arxiv.org/abs/2604.15483)。接触物理活在模型够不着的频段里，这正是 harness 必须存在的原因。
- **精度。** 重复定位精度的跨度：研究级机械臂优于 0.1 毫米 [[spec: Franka Research 3]](https://franka.de/products/franka-research-3)，领域内大量训练实际用的低成本机械臂是 1 毫米 [[spec: Trossen ViperX 300]](https://www.trossenrobotics.com/viperx-300)——共享同一份数据集的这些平台之间，差了大约 **10x** [computed: 1 mm against 0.1 mm]。
- **鲁棒与安全。** 面向人类环境设计的人形机器人，公开数据是 95% 反驱性、30 kg 自重、头部伤害指标低于 250 [[spec: 1X NEO]](https://www.1x.tech/neo)。标准写得很明确：降速模式上限 250 mm/s，安全等级 PL d 与 SIL 2 [std: ISO 10218:2025]；接触力上限从面部 65 N 到大腿 220 N [std: ISO/TS 15066:2016]。
- **运维成本。** 领域内最大的公开缺口。所有舰队规模的论文里，平均无故障时间都没披露 [[arXiv:2104.08212]](https://arxiv.org/abs/2104.08212)；跨台制造差异被点过名，但从来没被量化 [[arXiv:2506.18123]](https://arxiv.org/abs/2506.18123)。

## 中间那层 harness

![频率栈](figures/f03-frequency-stack.png)

没人会把一个 5B 参数的 transformer 塞进伺服环，所以每个认真的系统都会切成两半，分别跑在不同频率上：语义层 1 到 10 Hz，setpoint 层 20 到 50 Hz，伺服层 200 到 1000 Hz [[arXiv:2503.14734]](https://arxiv.org/abs/2503.14734)。有一套公开的人形栈，三层分别跑 7 到 9 Hz、200 Hz 和 1 kHz [[blog: Figure Helix]](https://www.figure.ai/news/helix)——marketing 级来源，背后没有论文。另一套全身控制栈，策略跑 2.5 Hz，控制器跑 50 Hz [[repo: Isaac-GR00T hardware_recommendation.md]](https://github.com/NVIDIA/Isaac-GR00T/blob/main/getting_started/hardware_recommendation.md)。

各家真正有分歧的地方是这道缝**开多宽**。有的系统在两半之间只传一个连续隐向量 [[blog: Figure Helix]](https://www.figure.ai/news/helix)；有的对一整串中间特征做 cross-attention [[arXiv:2503.14734]](https://arxiv.org/abs/2503.14734)；π 系列传的是一句人可读的子任务描述外加一张生成图像 [[arXiv:2604.15483 §VI]](https://arxiv.org/abs/2604.15483)。最后这个选择费带宽，换来的是可调试性：机器人做错事的时候，你能读出来它以为自己在干什么。

而整条通路——从光子到力矩——至今没有任何实验室在任何平台上完整发表过 [[repo: openpi websocket_policy_server.py]](https://github.com/Physical-Intelligence/openpi/blob/main/src/openpi/serving/websocket_policy_server.py)。

---

# 第 3 部分 —— 那台跑通了的机器，以及它搬不过来的地方

要理解机器人为什么难，先得准确理解语言建模为什么*容易*——工程量上并不容易，容易的是一个很具体的意义：往里投资源，它就可靠地起作用。

## 方法论

把智能从数据集里压缩进权重，用算力换进展，别用手工设计的特征换进展。这件事之所以是一套战略而不只是一种指望，是因为回报可以预测。loss 随数据、参数、算力呈幂律下降，指数分别是 **0.095**、**0.076**、**0.050**，拟合跨越 **7** 个数量级 [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361)。数据翻十倍，loss 乘 **0.80**；参数翻十倍，乘 **0.84** [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361)。

这种可预测性本身就是一件预算工具。它告诉某家实验室：在同等算力下，**70B** 模型配 **1.4T** token，会打赢 **280B** 模型配 **300B** token，最优配比大约是每参数 **20** 个 token [[arXiv:2203.15556]](https://arxiv.org/abs/2203.15556)。规模化定律是你决定买什么的依据——所以一个没有规模化定律的领域，是一个没法做规划的领域。

支撑那台机器跑起来的，是三根支柱。

## 支柱一 —— 数据的组织范式

一切皆 token。同一套表示覆盖翻译、摘要、代码、对话，正是这一点让「一条普适的定律」变得可寻，而不至于退化成每个任务一条曲线 [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361)。

而且语料本来就在那儿。一条开源流水线把 **96** 个 Common Crawl 快照蒸成 **15T** token、**44 TB** 文本，只花了 **1,536** GPU 小时 [[arXiv:2406.17557]](https://arxiv.org/abs/2406.17557)，折合大约 **\$10k** 的算力 [computed: 1,536 GPU-hours at commodity rates]。没有任何人被雇来生产这些文本，它是互联网存在的副产品。

在这之上，评测几乎不要钱：留出集困惑度，加上机器可以直接判定的任务结果。测试周期以分钟计，随基础设施扩展，不随人头扩展 [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361)。on-policy 数据也便宜，因为沙箱就是一次 API 调用。

## 支柱二 —— 计算范式

![三款加速器的 roofline 位置](figures/f15-roofline.png)

这里是最常被说错的一段。所谓「大模型」，真正的衡量标准是**计算强度**——每从内存搬运一个字节，能做多少次浮点运算；参数量本身说明不了什么。

每块加速器都有一个 *ridge point*：峰值算力除以内存带宽。落在它左边，芯片在等内存，标称算力是假的；落在它右边，芯片才真的在算。下面这组数字由厂商规格书推出。

| 器件 | 峰值 | 带宽 | ridge point |
|---|---|---|---|
| H100 SXM | 3,958 TFLOPS FP8 | 3.35 TB/s | 约 1,181 FLOP/byte |
| Jetson AGX Thor | 1,035 TFLOPS dense FP4 | 273 GB/s | 约 3,791 FLOP/byte |
| AGX Orin | 275 TOPS sparse INT8 | 204.8 GB/s | 约 1,343 OP/byte |

这三行的精度各不相同，所以它不是一次同口径的算力对比，也不该被那样读 [[spec: NVIDIA H100 SXM]](https://www.nvidia.com/en-us/data-center/h100/)。真正可比的是：每块器件对负载提出的比值要求。Thor 的 ridge point 比 H100 的往右挪了大约 **3.2x** [computed: 3,791 over 1,181]，意思是边缘器件对访存受限负载的惩罚，比它 TOPS 数字暗示的要重得多 [[spec: NVIDIA Jetson AGX Thor]](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/)。

实测后果在公开数据里看得见：同一个 action expert，在 Thor 上要 **26.20 毫秒**，在消费级 GPU 上只要 **7.25 毫秒**，惩罚倍数 **3.6x** [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。看 GB/s 那一行，别看 TOPS 那一行。

![实测吞吐对内存带宽](figures/f05-hz-per-bandwidth.png)

transformer 能赢，是因为序列建模足够通用，*而且*这个架构计算密度足够高，能把硬件换成能力 [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361)。这句话的两半缺一不可。

## 支柱三 —— 基础设施

这个规模上的训练本身就是一件工程作品：某个 540B 参数的训练跑出了 **46.2%** 的 model FLOPs utilization [[arXiv:2204.02311]](https://arxiv.org/abs/2204.02311)，这需要上千块加速器，以及足以扛住它们连着坏几周的容灾能力。推理是它的镜像——低延迟、高并发，再加上 test-time scaling 和一套能让长程多轮调用保持正确的 harness [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361)。

## 对照表

![机器人到底满足了哪几个前提](figures/f02-scorecard.png)

机器人满足支柱二。支柱一和支柱三，它没有以那种曾经产生威力的形态满足；此外它还多背一条文本从未面对过的约束：物理不等人 [[arXiv:2604.15483 §VII]](https://arxiv.org/abs/2604.15483)。

## 五个卡点

![五个卡点，以及各自击穿了哪根支柱](figures/f14-five-blockers.png)

这是整篇论证的枢纽。每个卡点用一句原理加一个决定性数字给出，第 4 部分再算它各要付多少代价。

**其一：没有统一的动作词表。** 文本只有一套字母表，机器人没有。动作空间要补零到 18 或 19 维，才能被拼进同一个训练混合里 [[arXiv:2410.24164]](https://arxiv.org/abs/2410.24164)。标准语料之间控制频率从 3 Hz 跨到 50 Hz [[arXiv:2310.08864]](https://arxiv.org/abs/2310.08864)，于是同样 50 步的 chunk，在一台机器上是 1 秒运动，在另一台上是 2.5 秒 [computed: 50 steps at 50 Hz and 20 Hz]。连 tokenizer 的效率都取决于机体：同一套压缩方案，在 5 Hz、7 自由度下压缩比 **1.75x**，在 50 Hz、14 自由度下是 **13.2x** [[arXiv:2501.09747]](https://arxiv.org/abs/2501.09747)。而机器人领域唯一被复现过的规模化定律，跑的是*多样性*，不是数据量——归一化分数随 pair 数以 **0.88** 乘以 −**0.30** 次幂改善，而每个 pair 的演示数在总量约 **800** 条时就饱和了 [[arXiv:2410.18647]](https://arxiv.org/abs/2410.18647)。

**其二：没有无监督语料。** 最接近网页文本的东西是第一人称人类视频，两个最大的公开语料加起来 **4,956** 小时 [computed: 3,670 h plus 1,286 h]，对面是 15T token 的网页文本 [[arXiv:2110.07058]](https://arxiv.org/abs/2110.07058)。机器人数据是*被制造出来的*：某个预训练混合大约代表 **10,000** 小时、**903M** 个时间步 [[arXiv:2410.24164]](https://arxiv.org/abs/2410.24164)，按全负担 **\$50** 每小时算，约合 **\$500k** 的人力成本 [computed: 10,000 h at \$50 per hour]。另一份数据集动用了 **100** 台机器人、**4,000** 平米场地，产出 **1,001,552** 段 episode、**2,976.4** 小时 [[arXiv:2503.06669]](https://arxiv.org/abs/2503.06669)。还有一份用了 **18** 套采集架、**50** 名采集员、**12** 个月，换来 **76k** 条轨迹 [[arXiv:2403.12945]](https://arxiv.org/abs/2403.12945)。

**其三：计算范式搬得过来，但多花的算力没买到关键指标。** 加载视觉语言 checkpoint 再挂一个 action head，是货真价实的迁移，第 1 部分那个网络预训练消融就是证据。可是把 backbone 固定、只换 action head：50 步去噪的 diffusion 跑 **10.1 Hz**、成功率 **95.4%**；连续回归跑 **109.7 Hz**、成功率 **95.3%** [[arXiv:2502.19645]](https://arxiv.org/abs/2502.19645)——吞吐差了约 **11x**，成绩差了 0.1 个百分点 [computed: 109.7 Hz against 10.1 Hz]。多烧一个数量级的算力却换不到可测收益，这跟规模化定律正好相反。

![动作表示的取舍](figures/f07-action-representation.png)

**其四：训练基础设施不是瓶颈，端侧适配可能才是。** 没有哪个机器人结果是在语言模型那个量级上被算力卡住的。领域内公开的最大一笔算力开销花在*数据生成*上——**240k** 个样本，**1,500** 块 GPU 跑 **54** 小时 [[arXiv:2505.12705]](https://arxiv.org/abs/2505.12705)。真正悬着的问题在另一端：在机器上做适配——已有端侧模型能用 **50 到 100** 条演示微调到新任务 [[arXiv:2503.20020]](https://arxiv.org/abs/2503.20020)。

**其五：推理意味着一台真机器。** 语言 harness 卡住，代价是回答慢了。策略卡住，代价是把一条过期轨迹发给一个已经变了的世界。实时 chunking 之所以存在就是因为这个，它 100–200 毫秒的容忍度和 300 毫秒的上限之所以承重也是因为这个 [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339)；而光子到力矩的延迟没有任何实验室发表过，这是一次测量失职，谈不上疏忽 [[repo: openpi websocket_policy_server.py]](https://github.com/Physical-Intelligence/openpi/blob/main/src/openpi/serving/websocket_policy_server.py)。

五个卡点里，只有一个是建模问题。其余四个属于测量、机构和数据引擎。

---

# 第 4 部分 —— 这些卡点在每个子系统里各要付多少代价

![难点网格：两个子系统，各三件事](figures/f13-difficulty-matrix.png)

六个格子。每一格都是第 3 部分某个卡点落到具体工程层面之后的本地形态。

## 大脑 / 评测 —— 来自卡点一

先造量具，再造被量的东西，因为量具现在根本不存在。

先算一笔没人算的账。在常规显著性和检验力下，要把 50% 的策略和 60% 的策略分开，每个策略需要 **387** 次试验；要把 50% 和 55% 分开需要 **1,565** 次；哪怕只是把 80% 和 90% 分开，也要 **199** 次 [computed: two-proportion test, alpha 0.05, 80% power]。而领域内实际跑 **10** 到 **60** 次 [[arXiv:2506.18123]](https://arxiv.org/abs/2506.18123)——有影响力的论文里，一篇是每任务每策略 **10** 次 [[arXiv:2504.16054]](https://arxiv.org/abs/2504.16054)，另一篇总共约 **125** 次真机 rollout [[arXiv:2503.14734]](https://arxiv.org/abs/2503.14734)。一个在 20 次试验里测出 60% 的策略，95% 置信区间是 **0.39 到 0.78** [computed: Wilson score interval]。这不叫测量。

后果已经被量化过。一次大规模审计发现，LIBERO 上只有 **19.8%** 的论断、SimplerEnv 上只有 **19.7%** 的论断在统计上显著 [[arXiv:2606.04233]](https://arxiv.org/abs/2606.04233)。同一份工作还发现，一个 **90M** 参数、完全不读指令的探针，在四个 LIBERO 子集上拿到 **99.0、100、98.8 和 92.4%** [[arXiv:2606.04233]](https://arxiv.org/abs/2606.04233)——模型不读指令也能通过的 benchmark，测的显然不是指令跟随。

更麻烦的是，这些 benchmark 恰好对硬件团队掌控的那些变量极其敏感。改变机器人初始状态，一个模型掉 **87.6** 分，另一个掉 **59.9** 分；改变相机视角，分别掉 **78.4** 和 **37.4** [[arXiv:2510.13626]](https://arxiv.org/abs/2510.13626)。在位置扰动下，标准 LIBERO 上拿 **98%** 和 **92%** 的模型直接坍到 **0.0%** [[arXiv:2510.03827]](https://arxiv.org/abs/2510.03827)。在真机 benchmark 上，同一个任务因操作员不同，成绩可以从 **0% 到 100%** [[arXiv:2510.17950]](https://arxiv.org/abs/2510.17950)。

已有的东西值得用起来。精心构造的套件上，real-to-sim 的 Pearson 相关达到 **0.924** [[arXiv:2405.05941]](https://arxiv.org/abs/2405.05941)；跨 **7** 家机构、**4,284** 段 episode 的分布式评测，约 **100** 次成对比较后收敛 [[arXiv:2506.18123]](https://arxiv.org/abs/2506.18123)；自适应试验分配最多省下 **70%** 的试验次数 [[arXiv:2603.13616]](https://arxiv.org/abs/2603.13616)；一个世界模型评测器以 **100** H100 小时的复现成本，拿到与真机评测 **0.989** 的秩相关 [[arXiv:2607.01060]](https://arxiv.org/abs/2607.01060)。

## 大脑 / 训练 —— 来自卡点二和卡点三

![数据来源阶梯](figures/f08-sources-ladder.png)

数据不存在，也爬不到，只能被工程化地造出来。好消息是，唯一那条被复现过的定律告诉了你该买什么。

![多样性规模化定律](figures/f06-diversity-scaling.png)

买多样性，别买数据量。被验证过的配方是 **32** 个「环境×物体」组合、每个 **50** 条演示，在没见过的环境里拿到 **85% 到 92.5%**，采集只用了 **4** 个人一个下午 [[arXiv:2410.18647]](https://arxiv.org/abs/2410.18647)。同一份研究覆盖超过 **40,000** 条演示和 **15,000** 次真机 rollout，发现每个 pair 的演示数在总量约 **800** 条时饱和 [[arXiv:2410.18647]](https://arxiv.org/abs/2410.18647)。

混合权重的分量比看上去重：学出来的混合权重比均匀加权高 **38%**，比人工挑的权重高 **32%** [[arXiv:2408.14037]](https://arxiv.org/abs/2408.14037)；一种数据筛选方法用不到 **33%** 的数据就追平了全量表现 [[arXiv:2506.19121]](https://arxiv.org/abs/2506.19121)。异质性则是共用语料要交的税——标准开源混合里，腕部相机覆盖率 **27%**，语言标注覆盖率 **56%** [[arXiv:2405.12213]](https://arxiv.org/abs/2405.12213)，再叠上卡点一里的补零和频率跨度。有一种做法把每种本体都压成 **16** 个固定 token，跨 **52** 个数据集、约 **200k** 条轨迹，换来 **20%** 的提升 [[arXiv:2409.20537]](https://arxiv.org/abs/2409.20537)。

对着 **327** 篇论文做的元分析给出，机器人领域在数据、模型、算力上的指数分别是 **0.276**、**0.246**、**0.141** [[arXiv:2405.14005]](https://arxiv.org/abs/2405.14005)——比语言模型的指数陡，但拟合跨度只有大约三个数量级，而不是七个。

## 大脑 / 部署 —— 来自卡点三和卡点五

压缩有悬崖，而且不缓。某个方法在 3.0 bit/权重时守住 **94.8%**，掉到 2.0 时变成 **48.0%** [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011)。机器人专用的量化方案不是可选项：一套 W4A8 拿到 **97.6%**，对面 FP16 是 **97.1%**，模型从 **4.27 GB 压到 1.28 GB** [[arXiv:2602.20309]](https://arxiv.org/abs/2602.20309)；而把通用语言模型的训练后量化直接套到同一个策略上，只剩 **76.3%**，掉了 **21** 分 [[arXiv:2602.20309]](https://arxiv.org/abs/2602.20309)。

算法与运行时的协同设计才是杠杆所在。编译带来 **1.5 到 2.9x**，优化过的推理流水线带来 **1.5 到 3.3x** [[repo: Isaac-GR00T hardware_recommendation.md]](https://github.com/NVIDIA/Isaac-GR00T/blob/main/getting_started/hardware_recommendation.md)。一个单步 flow 变体把某个策略从 **274 毫秒压到 83 毫秒**，成功率还从 **97.75%** 升到 **98.75%** [[arXiv:2604.05656]](https://arxiv.org/abs/2604.05656)。chunking 加连续动作把一个系统从 **4.2 Hz 拉到 109.7 Hz**、从 **76.5%** 拉到 **97.1%**，加速 **26x** [[arXiv:2502.19645]](https://arxiv.org/abs/2502.19645)。

对公开数据保持怀疑。同一个模型在同一块边缘器件上的延迟，不同来源之间从 **46 毫秒**跨到 **246 毫秒**，差 **5x** [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397)。

## 身体 / 硬件设计 —— 来自卡点一和卡点五

标定是一阶精度项，算不上维护琐事。某个公开数据集在发布之后，为其中 **36,000** 段 episode 重新提供了更好的相机标定 [[spec: DROID dataset site]](https://droid-dataset.github.io/)。相机挪 10 厘米、转 20 度，100% 成功率的策略坍到 **0%** [[arXiv:2409.03403]](https://arxiv.org/abs/2409.03403)。安装刚度和重复定位精度给模型划了一条它翻不过去的下限 [[spec: Franka Research 3]](https://franka.de/products/franka-research-3)。

传感是另一半。能被换掉的触觉皮肤，比精度高的触觉皮肤更重要：某个设计的跨个体性能损失是 **13%**，对照基线是 **43%**，归一化跨个体波动 **0.12** 对 **0.54**，更换耗时 **12 秒**对 **236 ± 64** 秒 [[arXiv:2409.08276]](https://arxiv.org/abs/2409.08276)。一个 **460k** 张图像的触觉预训练语料已经存在 [[arXiv:2410.24090]](https://arxiv.org/abs/2410.24090)。这些东西，一样都没有接进前沿模型。

## 身体 / 采集管线 —— 来自卡点二

采集架阶梯是这个领域里成本收益最清楚的一张表。手持工位单价 **\$371**——**\$73** 的夹爪加 **\$298** 的相机——每小时产 **111** 条演示，对面遥操作是 **35** 条，吞吐倍数 **3.2x**，在新环境里拿到 **71.7%**，迁移到研究级机械臂上还有 **90%** [[arXiv:2402.10329]](https://arxiv.org/abs/2402.10329)。低成本遥操作接口不到 **\$300** [[arXiv:2309.13037]](https://arxiv.org/abs/2309.13037)；外骨骼采集架 **\$0.6k**，替掉的是约 **\$60k** 的平台 [[arXiv:2503.03081]](https://arxiv.org/abs/2503.03081)；双臂工位不到 **\$20k** [[arXiv:2304.13705]](https://arxiv.org/abs/2304.13705)，移动版 **\$32k** [[arXiv:2401.02117]](https://arxiv.org/abs/2401.02117)。

剩下的由操作员成本决定。某西方公司公开的时薪是 **\$25.25 到 \$48.00** [[blog: Tesla job posting via Fortune]](https://fortune.com/2024/08/19/tesla-robot-hiring-workers-optimus-training-ai/)，而中国采集中心被报道的价格约为每小时 **\$3** [[blog: Rest of World]](https://restofworld.org/2026/china-robots-training-centers-workers/)——两条都是 marketing 级来源，此处如实标注。

按单位学习量算，人类视频是最便宜的来源：**1** 小时人类视频约等于 **1,400** 条演示，而 **1** 小时机器人时间只等于 **135** 条 [[arXiv:2410.24221]](https://arxiv.org/abs/2410.24221)。而且它真的有用——叠衣服成功率 **88%** 对 **55%**，没见过的布料颜色上 **85%** 对 **25%** [[arXiv:2410.24221]](https://arxiv.org/abs/2410.24221)，相关方法再加 **44** 个百分点 [[arXiv:2509.19626]](https://arxiv.org/abs/2509.19626)。仿真和生成式增广排在它上面：真实加仿真拿到 **24.4%** 和 **9.3%**，纯真实数据只有 **13.6%** 和 **2.6%** [[arXiv:2406.02523]](https://arxiv.org/abs/2406.02523)；某条流水线把不到 **200** 条演示放大成超过 **50,000** 条 [[arXiv:2310.17596]](https://arxiv.org/abs/2310.17596)；某个生成式流水线把新环境中的新行为从 **0.0%** 抬到 **28.5%** [[arXiv:2505.12705]](https://arxiv.org/abs/2505.12705)。

## 身体 / 运行时外壳 —— 来自卡点五

![至今没人发表过的延迟预算](figures/f04-latency-budget.png)

模型自己那段时间，是整个问题里*被测过*的三分之一：图像编码器 **14 毫秒**，prefix pass **32 毫秒**，flow 步 **27 毫秒** [[arXiv:2410.24164]](https://arxiv.org/abs/2410.24164)。曝光、传输、预处理、坐标变换、执行器响应，整个领域都没测 [[repo: openpi websocket_policy_server.py]](https://github.com/Physical-Intelligence/openpi/blob/main/src/openpi/serving/websocket_policy_server.py)。一份公开预算能容忍 **0 到 12** 个时间步的延迟，在 50 Hz 下就是 **240 毫秒**的窗口 [computed: 12 timesteps at 50 Hz]。

安全是架构问题，学不出来。神经策略拿不到 Performance Level 认证，所以过滤器必须放在策略之外：一个基于可达性的安全过滤器跑在 **100 Hz**，在 **66** 条真机轨迹上验证过 [[arXiv:2410.11157]](https://arxiv.org/abs/2410.11157)。相关标准在 **2025** 年做了自 **2011** 年以来的第一次修订 [std: ISO 10218:2025]，而面向动态稳定移动机器人的标准还停在 Working Draft [std: ISO 25785-1]。

---

# 第 5 部分 —— 施工顺序

![施工顺序](figures/f09-milestone-ladder.png)

六个里程碑。每个都有一个用数字表述、可以被证伪的通过条件，并且各自关掉第 4 部分网格里的一格。排序原则只有一条：先造量具，再造被量的东西。

**M0 —— 造量具。** 用序贯检验，加上相关性验证到 **0.924** 的 real-to-sim 代理，再加上最多能省 **70%** 试验的自适应分配，把「检出 10 分差距」的成本压到 **387** 次试验以下 [computed: two-proportion test, alpha 0.05, 80% power]。顺便把你自己平台的光子到力矩预算发出来，因为别人都没发 [[repo: openpi websocket_policy_server.py]](https://github.com/Physical-Intelligence/openpi/blob/main/src/openpi/serving/websocket_policy_server.py)。*关掉大脑/评测。*

**M1 —— 造数据引擎。** 从 **32** 个「环境×物体」组合乘 **50** 条演示起步，这是唯一背后有复现定律的配方 [[arXiv:2410.18647]](https://arxiv.org/abs/2410.18647)，跑在单价 **\$371** 的手持工位上 [[arXiv:2402.10329]](https://arxiv.org/abs/2402.10329)——四个工位是 **\$1,484** 的资本开支 [computed: 4 stations at \$371]。从第一天就把人类视频叠进来，每小时折合 **1,400** 条演示当量 [[arXiv:2410.24221]](https://arxiv.org/abs/2410.24221)。也从第一天就录力信号和元数据，因为事后补一路通道等于把语料重采一遍 [[arXiv:2505.22159]](https://arxiv.org/abs/2505.22159)。*关掉身体/采集，以及大脑/训练的一半。*

**M2 —— 训策略。** 视觉语言 checkpoint 加 flow-matching action expert，总参数约 **5B**——**4B** backbone 加 **860M** expert [[arXiv:2604.15483 §IV]](https://arxiv.org/abs/2604.15483)——chunk 长度 H = **50**，去噪 **5** 步 [[arXiv:2604.15483 §VII]](https://arxiv.org/abs/2604.15483)。全量微调要按 **70 GB** 显存下限做预算 [[repo: openpi README]](https://github.com/Physical-Intelligence/openpi/blob/main/README.md)。通过条件：在 M0 那把量具上，以显著性打赢现有方案。*关掉大脑/训练。*

**M3 —— 从经验里学。** 在自主数据和人工介入数据上训一个 value model，每轮迭代约 **300** 条轨迹 [[arXiv:2511.14759]](https://arxiv.org/abs/2511.14759)。通过条件：相对 M2，吞吐至少 **2x**，失败率至少减半 [[arXiv:2511.14759]](https://arxiv.org/abs/2511.14759)。这是通往第四条能力轴唯一被公开验证过的路径，也是最可能延期的一个里程碑。*关掉大脑/部署里训练相关的那半。*

**M4 —— 把算力搬上机器。** 蒸馏加量化，压到 **130 W** 的功耗上限之内 [[spec: NVIDIA Jetson AGX Thor]](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/)，成功率损失不超过 **1** 个百分点 [computed: this plan]，用的是机器人专用量化的 **97.6%** [[arXiv:2602.20309]](https://arxiv.org/abs/2602.20309)、单步 flow 的 **83 毫秒** [[arXiv:2604.05656]](https://arxiv.org/abs/2604.05656)，以及值 **1.5 到 3.3x** 的编译优化 [[repo: Isaac-GR00T hardware_recommendation.md]](https://github.com/NVIDIA/Isaac-GR00T/blob/main/getting_started/hardware_recommendation.md)。单台机器人的算力成本变成 **\$3,499** 对一块数据中心 GPU，功耗变成 **40 到 130 W** 对 **700 W** [computed: EDGE-19 against EDGE-25]。*关掉大脑/部署，并兑现第 1 部分的产品结论。*

**M5 —— 跑一支舰队。** 把平均无故障时间和重标定周期发出来，这是没有实验室披露过的两个数 [[arXiv:2104.08212]](https://arxiv.org/abs/2104.08212)。通过条件：换第二种本体、换第二个场地，不重跑预训练 [[arXiv:2408.11812]](https://arxiv.org/abs/2408.11812)。*关掉身体/硬件与身体/运行时。*

## 对 scaling 该有什么预期

按多样性做规划，别按数据量；按三个数量级的拟合做规划，别按七个 [[arXiv:2405.14005]](https://arxiv.org/abs/2405.14005)。多样性定律会饱和 [[arXiv:2410.18647]](https://arxiv.org/abs/2410.18647)。在你自己的量具复现出来之前，默认公开数字偏乐观——毕竟领域自己的审计说，只有约五分之一的对比是显著的 [[arXiv:2606.04233]](https://arxiv.org/abs/2606.04233)。


---

# 第 6 部分 —— 三个值得押的开放问题

*这一部分是推测性的，并已如此标注。fact ledger 没有覆盖到的地方，文章会直说，不会顺手写一句听起来合理的话。*

## 触觉原生的模型，以及第一人称数据

已经存在的：平均涨 **23.2%** 的力通道 [[arXiv:2505.22159]](https://arxiv.org/abs/2505.22159)，把 USB 插接从 **5% 抬到 35%** 的触觉 [[arXiv:2507.09160]](https://arxiv.org/abs/2507.09160)，把跨个体退化压在 **13%** 的可更换皮肤 [[arXiv:2409.08276]](https://arxiv.org/abs/2409.08276)，**460k** 张图像的触觉预训练语料 [[arXiv:2410.24090]](https://arxiv.org/abs/2410.24090)，以及迁移收益达 **44** 个百分点的第一人称人类数据 [[arXiv:2509.19626]](https://arxiv.org/abs/2509.19626)。

缺的是：没有任何前沿机器人基础模型把力当作一等模态吃进去，也没有任何触觉表示能在基础模型规模上跨传感器个体迁移 [[arXiv:2410.24090]](https://arxiv.org/abs/2410.24090)。这里可押的是：第一个把第 2 部分那张图里那条虚线接上的团队，会拿到别人没有的接触密集型操作能力。

## 机体与策略联合优化

这是全文最薄的一节，理由值得挑明：这项工作背后的 fact ledger 里，关于机体形态与策略联合优化的内容一条都没有，所以下面是一个问题，不是一个论断 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483)。

不过动机是有根据的。共享同一份训练语料的那些平台之间，重复定位精度差了大约 **10x** [computed: 1 mm against 0.1 mm]。没有实验室发表过在摩擦、负载或刚度上的扫描 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483)。跨台制造差异被点过名，从没被量化 [[arXiv:2506.18123]](https://arxiv.org/abs/2506.18123)。这些都指向同一件事：机体是一个自由变量，而没有人拿真正重要的那个目标——策略成功率——去优化它。把形态放进优化回路，在仿真里和策略一起演化，是显而易见的下一步；诚实的说法是，这个规模上还没有公开工作这么做。

## 持续学习与 test-time training

公开最接近的闭环，是在自主数据和介入数据上训 value model，报告吞吐至少 **2x**、失败率至少减半，并且连续服务了 **13** 小时 [[arXiv:2511.14759]](https://arxiv.org/abs/2511.14759)。公开最接近的端侧适配，是用 **50 到 100** 条演示做微调 [[arXiv:2503.20020]](https://arxiv.org/abs/2503.20020)。人在回路的强化学习在 **12** 个真机任务上做到 **12** 个全部 **100%**，每个任务 **1 到 2.5** 小时，起步只要 **20 到 30** 条种子演示 [[arXiv:2410.21845]](https://arxiv.org/abs/2410.21845)。

没人发表的是：对机械磨损的适应，以及平均无故障时间 [[arXiv:2104.08212]](https://arxiv.org/abs/2104.08212)。这一条把话题接回第 1 部分的第四条轴。一个察觉不到自己夹爪正在退化的模型，谈不上「从经验中进化」——它只是一张快照，加上不错的宣传。

---

## Gap 清单

实验室外没人知道的事，集中列在一处。每一条都是一个贡献机会，留给有条件去测它的人。

| 缺口 | 状态 |
|---|---|
| 预训练数据总量、各来源混合权重、优化器与总算力 | 未披露 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483) |
| 任何前沿模型背后的机器人舰队规模 | 未披露 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483) |
| 单条演示的成本，所有实验室 | 未披露 [[arXiv:2506.18123]](https://arxiv.org/abs/2506.18123) |
| 平均无故障时间，所有舰队规模的论文 | 未披露 [[arXiv:2104.08212]](https://arxiv.org/abs/2104.08212) |
| 跨台制造差异 | 点过名，未量化 [[arXiv:2506.18123]](https://arxiv.org/abs/2506.18123) |
| 光子到力矩的延迟，任何平台 | 未披露 [[repo: openpi websocket_policy_server.py]](https://github.com/Physical-Intelligence/openpi/blob/main/src/openpi/serving/websocket_policy_server.py) |
| 摩擦、负载、刚度上的泛化扫描 | 无实验室发表 [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483) |
| 商用人形机器人的板载算力厂商、功耗与算力 | 未披露 [[blog: Figure Helix]](https://www.figure.ai/news/helix) |

## 这些推导把我们带到哪里

泛化是真的，第 0 部分那第一个数字就是认真对待这个领域的理由。脆弱是机械层面的，第 0 部分那第二个数字就是不要轻信 demo 的理由。夹在两者之间的，是一台只跑通过一次的机器——为语言跑通的，靠三根支柱；而机器人只占住其中一根，还多背一条语言从未面对的约束。

这不构成停手的理由，它构成按特定顺序动手的理由：先量具，再数据引擎，再策略，再让它自我改进的闭环，最后是把它压进机器里的压缩。五个卡点里有四个属于测量、机构和数据引擎，这意味着大部分工作并不是建模工作——也意味着，能把这些问题关掉的人里，多数人目前并不认为自己是机器学习研究者。

---

## 参考文献

- [[arXiv:2001.08361]](https://arxiv.org/abs/2001.08361) Scaling Laws for Neural Language Models
- [[arXiv:2108.07258]](https://arxiv.org/abs/2108.07258) On the Opportunities and Risks of Foundation Models
- [[arXiv:2203.15556]](https://arxiv.org/abs/2203.15556) Training Compute-Optimal Large Language Models
- [[arXiv:2204.02311]](https://arxiv.org/abs/2204.02311) PaLM: Scaling Language Modeling with Pathways
- [[arXiv:2406.17557]](https://arxiv.org/abs/2406.17557) The FineWeb Datasets
- [[arXiv:2604.15483]](https://arxiv.org/abs/2604.15483) π0.7: a generalist robot policy
- [[arXiv:2504.16054]](https://arxiv.org/abs/2504.16054) π0.5: open-world generalization
- [[arXiv:2410.24164]](https://arxiv.org/abs/2410.24164) π0: a vision-language-action flow model
- [[arXiv:2511.14759]](https://arxiv.org/abs/2511.14759) RECAP: self-improvement from autonomous experience
- [[arXiv:2506.07339]](https://arxiv.org/abs/2506.07339) Real-Time Chunking
- [[arXiv:2310.08864]](https://arxiv.org/abs/2310.08864) Open X-Embodiment
- [[arXiv:2307.15818]](https://arxiv.org/abs/2307.15818) RT-2: vision-language-action models
- [[arXiv:2212.06817]](https://arxiv.org/abs/2212.06817) RT-1: robotics transformer
- [[arXiv:2510.03342]](https://arxiv.org/abs/2510.03342) Gemini Robotics 1.5
- [[arXiv:2503.20020]](https://arxiv.org/abs/2503.20020) Gemini Robotics On-Device
- [[arXiv:2503.14734]](https://arxiv.org/abs/2503.14734) GR00T N1
- [[arXiv:2406.09246]](https://arxiv.org/abs/2406.09246) OpenVLA
- [[arXiv:2502.19645]](https://arxiv.org/abs/2502.19645) OpenVLA-OFT
- [[arXiv:2501.09747]](https://arxiv.org/abs/2501.09747) FAST action tokenization
- [[arXiv:2304.13705]](https://arxiv.org/abs/2304.13705) ALOHA and ACT
- [[arXiv:2401.02117]](https://arxiv.org/abs/2401.02117) Mobile ALOHA
- [[arXiv:2309.13037]](https://arxiv.org/abs/2309.13037) GELLO
- [[arXiv:2503.03081]](https://arxiv.org/abs/2503.03081) AirExo-2
- [[arXiv:2402.10329]](https://arxiv.org/abs/2402.10329) Universal Manipulation Interface
- [[arXiv:2403.12945]](https://arxiv.org/abs/2403.12945) DROID
- [[arXiv:2503.06669]](https://arxiv.org/abs/2503.06669) AgiBot World
- [[arXiv:2410.18647]](https://arxiv.org/abs/2410.18647) Data scaling laws in imitation learning
- [[arXiv:2405.14005]](https://arxiv.org/abs/2405.14005) Scaling laws in robot learning, a meta-analysis
- [[arXiv:2408.14037]](https://arxiv.org/abs/2408.14037) Re-Mix: data mixture optimization
- [[arXiv:2506.19121]](https://arxiv.org/abs/2506.19121) CUPID: data curation
- [[arXiv:2405.12213]](https://arxiv.org/abs/2405.12213) Octo
- [[arXiv:2409.20537]](https://arxiv.org/abs/2409.20537) Heterogeneous Pre-trained Transformers
- [[arXiv:2408.11812]](https://arxiv.org/abs/2408.11812) CrossFormer
- [[arXiv:2410.24221]](https://arxiv.org/abs/2410.24221) EgoMimic
- [[arXiv:2509.19626]](https://arxiv.org/abs/2509.19626) EgoBridge
- [[arXiv:2110.07058]](https://arxiv.org/abs/2110.07058) Ego4D
- [[arXiv:2406.02523]](https://arxiv.org/abs/2406.02523) RoboCasa
- [[arXiv:2310.17596]](https://arxiv.org/abs/2310.17596) MimicGen
- [[arXiv:2505.12705]](https://arxiv.org/abs/2505.12705) DreamGen
- [[arXiv:2606.04233]](https://arxiv.org/abs/2606.04233) Statistical rigour in robot learning benchmarks
- [[arXiv:2510.13626]](https://arxiv.org/abs/2510.13626) Perturbation sensitivity of VLA policies
- [[arXiv:2510.03827]](https://arxiv.org/abs/2510.03827) LIBERO-PRO
- [[arXiv:2510.17950]](https://arxiv.org/abs/2510.17950) RoboChallenge
- [[arXiv:2405.05941]](https://arxiv.org/abs/2405.05941) SimplerEnv
- [[arXiv:2506.18123]](https://arxiv.org/abs/2506.18123) RoboArena
- [[arXiv:2603.13616]](https://arxiv.org/abs/2603.13616) N-SCORE
- [[arXiv:2607.01060]](https://arxiv.org/abs/2607.01060) RoboWorld evaluation
- [[arXiv:2602.20309]](https://arxiv.org/abs/2602.20309) QuantVLA
- [[arXiv:2605.24011]](https://arxiv.org/abs/2605.24011) ActQuant
- [[arXiv:2604.05656]](https://arxiv.org/abs/2604.05656) SnapFlow
- [[arXiv:2602.18397]](https://arxiv.org/abs/2602.18397) VLA inference on edge accelerators
- [[arXiv:2409.03403]](https://arxiv.org/abs/2409.03403) RoVi-Aug
- [[arXiv:2505.22159]](https://arxiv.org/abs/2505.22159) ForceVLA
- [[arXiv:2507.09160]](https://arxiv.org/abs/2507.09160) Tactile-VLA
- [[arXiv:2409.08276]](https://arxiv.org/abs/2409.08276) AnySkin
- [[arXiv:2410.24090]](https://arxiv.org/abs/2410.24090) Sparsh
- [[arXiv:2410.21845]](https://arxiv.org/abs/2410.21845) HIL-SERL
- [[arXiv:2410.11157]](https://arxiv.org/abs/2410.11157) RPCBF safety filters
- [[arXiv:2104.08212]](https://arxiv.org/abs/2104.08212) Deep RL at Scale
- [[arXiv:2604.18000]](https://arxiv.org/abs/2604.18000) Shortcut learning in VLA models
- [[arXiv:2602.17659]](https://arxiv.org/abs/2602.17659) When vision overrides language
