# auto-data-labeling-workflow

数据标注 AI 自动化工作流。目标是把「原始数据 → 模型预标注 → 人工校正 → 质检入库」这条链路里能自动化的环节交给 AI，人只处理机器拿不准的部分。

## 当前状态

仓库刚建立，方案和分工还没定。上面的链路是讨论起点，不是已确定的设计。

## 先看这里

- [Issue #1](https://github.com/LudWikiPedia/auto-data-labeling-workflow/issues/1)：三件必须先定的事（标注什么数据、谁做质检、模型拿不准怎么兜底）。**不定这三件，写代码就是白写。**
- [WeVisOne-VL 选型核实记录](docs/reference/wevisone-vl.md)：腾讯微信视觉团队开源的视觉感知模型，Apache 2.0 可商用。它训练数据那套自动标注流程就是我们要做的事。
- [20 张图试测记录表](docs/spike/wevisone-demo-trial.md)：0 元、不用装环境，照着点就能测出这条路值不值得走。

## 可见性

私有仓库，仅 LudWikiPedia 组织成员可访问。

## 成员

kumu314、AAAkym、Ludwigzky、Qiu-lin-8、Wang060919
