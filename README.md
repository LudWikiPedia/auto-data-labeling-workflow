# auto-data-labeling-workflow

用开源视觉模型对图像里的**车辆、行人、宠物**等常见目标做自动识别与定位，产出边界框和分割掩码作为预标注，让人只校正机器拿不准的部分，从而减少人工画框的工作量。

流程骨架参考腾讯 WeVisOne-VL 团队自己那套数据引擎（详见下方链接）：多个模型定位 → 视觉大模型当裁判校验 → 只把校验过的框往下扩成掩码。我们的类别是固定的三类，所以**不抄它「用知识图谱发现概念」那一步**，只抄校验和分流。

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
