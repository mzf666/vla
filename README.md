# VLA / World Model Learning Workspace

截至 `2026-05-04`，这份工作区用于系统学习以下主题：

- `Vision-Language-Action (VLA)`
- `World Model`
- `Embodied AI / Simulation`
- `Mac mini / Apple Silicon` 上可跑通的入门实验

## 目录说明

- [ROADMAP.md](ROADMAP.md): 分阶段学习路线
- [READING_LIST.md](READING_LIST.md): 论文与 survey 阅读顺序
- [EXPERIMENT_PLAN.md](EXPERIMENT_PLAN.md): repo 选择、上手顺序与实验目标
- [CLAUDE.md](CLAUDE.md): 博客站点、写作风格、Excalidraw 图表规范
- [docs/outlines](docs/outlines): 9 篇系列文章的写作大纲
- [content/zh/posts](content/zh/posts): 中文 Hugo 文章
- [content/en/posts](content/en/posts): 英文占位文章
- [static/diagrams](static/diagrams): Excalidraw 源文件与 SVG 导出图

## 建议使用方式

1. 先读 [ROADMAP.md](/Users/robot/Documents/Projects/vla/ROADMAP.md)，确定 6-8 周主线。
2. 再按 [READING_LIST.md](/Users/robot/Documents/Projects/vla/READING_LIST.md) 建立术语和方法图谱。
3. 然后按照 [EXPERIMENT_PLAN.md](/Users/robot/Documents/Projects/vla/EXPERIMENT_PLAN.md) 跑最小实验。
4. 后续把你自己的读书笔记、问题列表、实验日志也继续放在这个目录里。

## 当前学习目标

- 先分清 `world model` 和 `VLA` 的边界、重叠与最新融合趋势
- 能复述典型 pipeline:
  - `observation -> latent/state -> future prediction -> planning/policy`
  - `image + language + state -> action chunk / action tokens`
- 能在 Mac mini 上至少跑通一条完整实验链
- 能基于一个开源 repo 看懂 `dataset -> training -> evaluation -> rollout`

## 推荐起步顺序

1. `World model survey`
2. `DreamerV3`
3. `LeRobot + ACT + PushT`
4. `LeRobot + SmolVLA`
5. `OpenVLA` 作为大模型 VLA baseline 阅读对象

## 博客站点

当前目录已经是一个独立 Hugo 子站，结构与 `llm-infra`、`cli-agent` 保持一致。

常用命令:

```sh
hugo server -D
hugo --minify
```

可发布中文文章:

- `content/zh/posts/00-series-roadmap/index.md`
- `content/zh/posts/01-problem-frame/index.md`

后续文章先以 `draft: true` 保留结构化写作草稿。
