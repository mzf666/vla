# VLA 深入浅出 — 工作约定

> 面向有强化学习基础的 Machine Learning PhD 的 VLA / World Model 系列博客。

## 项目概况

| 项目 | 说明 |
|------|------|
| 形式 | 独立 Hugo 子站，部署到 `mzf666.github.io/vla/` |
| 语言 | 中文为主（zh），英文占位（en） |
| 主题 | PaperMod（与 `llm-infra`、`cli-agent` 保持一致） |
| 主线 | `ROADMAP.md`: 基础概念 -> 经典方法 -> 工程复现 -> 前沿方向 |
| 读者 | 有 RL / ML 基础、希望进入 embodied policy / VLA 的 PhD |
| 可视化 | Excalidraw 源文件 + SVG 导出，通过 `excalidraw` shortcode 嵌入 |
| 与主站关系 | 完全解耦，主站只加 `VLA` 导航入口 |

## 命令

- 本地预览：`hugo server -D`
- 构建：`hugo --minify`
- 新文章占位：`hugo new posts/<slug>/index.md`

## Excalidraw 约定

- 源文件：`static/diagrams/<slug>.excalidraw`
- 导出图：`static/diagrams/<slug>.svg`
- 正文嵌入：

```go-html-template
{{< excalidraw src="/diagrams/example.svg" source="/diagrams/example.excalidraw" alt="..." caption="..." >}}
```

所有流程图、算法图、方法图、taxonomy 图都必须走这条路径，不再使用 Mermaid。

## 8 篇正文规划

| # | Slug | 主题 |
|---|---|---|
| 0 | `00-series-roadmap` | 系列路线图：为什么 VLA 必须和 world model、imitation pipeline 一起学 |
| 1 | `01-problem-frame` | 问题框架：policy、world model、VLA 的边界与交集 |
| 2 | `02-world-model-basics` | World model 基础：latent dynamics、imagination、planning |
| 3 | `03-embodied-imitation-pipeline` | 非 VLA embodied imitation pipeline：LeRobot、ACT、Diffusion Policy、robomimic |
| 4 | `04-vlm-to-vla` | 从 VLM 到 VLA：视觉、语言、状态与动作输出如何接起来 |
| 5 | `05-action-representation` | Action representation：continuous、chunk、token、diffusion、flow matching |
| 6 | `06-systems-and-deployment` | 系统与部署：latency、async inference、quantization、on-device VLA |
| 7 | `07-vla-world-model-frontier` | VLA + world model 前沿：WorldVLA、RynnVLA、Gemini Robotics、Genie |
| 8 | `08-research-entry-points` | 博士研究切入点：小模型、动作表征、future prediction、小数据 regime |

## 写作风格

- 中文为主，technical terms 保留英文。
- 每篇结构：直觉动机 -> formal problem -> 方法谱系 -> 代表系统 -> 工程/实验可操作点 -> 研究问题。
- 默认读者懂 MDP、policy gradient、offline RL、sequence model，但不默认懂 robot learning 工程栈。
- 对最新 report 明确区分 paper result、company report、inference/speculation。
- 每篇都回答：输入是什么、输出是什么、是否预测未来、是否使用语言、训练目标是什么、评估在哪里做。

