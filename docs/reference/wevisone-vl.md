# WeVisOne-VL 核实记录与可用性判断

> 核实日期：2026-11-05 ｜ 核实人：kumu314 ｜ 方法：只认一手来源（GitHub API、Hugging Face API、arXiv 摘要原文），二手新闻稿仅用于找线索，不作为依据。
>
> 这份文档是**给「数据标注 AI 自动化工作流」选型用**的，不是模型介绍。

## 一句话结论

腾讯微信视觉团队开源的视觉感知模型，Apache 2.0 可商用，本机跑不动但官方 demo 免费可试；**它真正的价值不在模型权重，而在它训练数据是怎么自动标出来的**——那套流程就是我们这个项目要做的东西。

## 它是什么

官方 README「Highlights」节原文（抄，未改写）：

> 🚫 **Reliable Rejection**: returns `None` for absent targets instead of hallucinating.

人话：让它找图里不存在的东西，它会答「没有」，而不是硬凑一个框出来。

官方 README「How It Works」第 1 条原文（抄，未改写）：

> **1. Data Engine.** A knowledge-graph-driven engine discovers concepts, localizes them with complementary experts verified by a VLM, and extends the verified boxes to masks and alpha mattes. Over 100M images are annotated this way.

人话拆四步：

1. 用知识图谱决定「要标哪些概念」——不是拿个固定类别表死标；
2. 让几个各有擅长的模型分别去图里找目标（互补专家）；
3. 再拿一个视觉大模型当裁判，判哪个框是对的（VLM 校验）；
4. 只对**校验通过**的框，往下扩成轮廓和抠图。

1 亿张图是这么标出来的。**这四步就是我们工作流的骨架**，比模型本身值得抄。

## 对我们项目的三个具体用处

| 我们的环节 | 它能顶什么 | 依据 |
|---|---|---|
| 模型预标注 | 开放词汇检测、按描述找目标、计数、OCR 带位置、画轮廓、抠图——一个模型全包，换任务不换模型 | README「Supported Tasks」节任务表：`detection` / `referring` / `count_box` / `count_point` / `ocr` / `ocr_grounding` / `segmentation` / `matting` |
| 质检分流 | 「答 None」可以当自动闸门：模型说没有的，直接进人工复核队列，不用逐张擦假框 | 同上，`referring` 输出「boxes or `None`」 |
| 省一套 SAM | 自带 960 万参数的像素解码器，不用再外挂分割模型 | README「How It Works」第 3 条：`a 9.6M-parameter pixel decoder` |

## 坑（这几条新闻稿没写，会直接影响我们能不能用）

1. **微调代码没放。** README「TODO」节里前五项已打勾，最后一项是空的：`- [ ] Fine-tuning code`。也就是说我们**不能**按自己的标注类别去训它，只能用现成能力 + 写提示词。如果我们的类别很专业（工业零件、医学结构之类），这条是硬限制。
2. **只支持单轮图像感知，且不做思维链。** HF 模型卡「Limitations」节原文：`currently targets single-turn image perception (video grids are rejected by the pixel decoder)`；`the model directly generates structured outputs without explicit chain-of-thought (CoT) reasoning`。人话：一次问一件事，不能追问；视频不行。
3. **抠图边缘很挑。** 同节原文：`Fine alpha boundaries remain sensitive to image resolution, severe occlusion, complex transparency, and prompt phrasing`。人话：图糊、目标被挡、半透明、提示词换个说法，结果都会变。**这条对我们最要命**——意味着提示词措辞得当成正式配置管起来，不能随手写。
4. **官方用途声明划了红线。** HF 模型卡原文：`The release is intended for research in visual grounding... It is not intended to make autonomous high-impact decisions or to identify individuals.` 人话：不许拿它做高影响的自动决策，不许用来识人。我们的流程里**必须留人工环节**，这既是工程判断也是官方要求。

## 成本真相表

| 跑法 | 能不能批量 | 花钱 | 实测/依据 |
|---|---|---|---|
| 本机（MX350 2GB 显存） | ❌ 跑不动 | — | `nvidia-smi` 读数 `GeForce MX350, 2048 MiB`；4B 权重单文件 9.66 GB，光权重就超显存好几倍 |
| HF 官方 Space demo | ❌ 一次一张手点 | **0 元**，不用注册 | 两个 demo 链接实测均返回 200 |
| 自己租 GPU 部署 | ✅ | 按卡时计费，需另查 | 官方支持 `backend="transformers"` 或 `backend="vllm"`，单卡 BF16 |
| 下载权重 | — | 流量成本 | HF API 实测：2B 仓库共 4.93 GB，4B 仓库共 9.71 GB |

## 许可证（可以商用，但有尾巴要盯）

- WeVisOne-VL 仓库 README「License」节原文：`Tencent-authored code in this repository is released under the Apache License 2.0. Third-party components keep their original licenses; please check the corresponding source files.` 人话：**腾讯写的代码** Apache 2.0，**第三方零件保留各自协议**，要逐个看源文件。
- 底座 Qwen3-VL：HF API 实测标 `license:apache-2.0` ✅
- Mask2Alpha 依赖的 SAM2：GitHub API 实测 `Apache-2.0` ✅
- 所以主链路（Qwen3-VL + 腾讯代码 + SAM2）目前看没有商用拦路石。但注意 GitHub 仓库页的 license 字段显示 `NOASSERTION`（它自动识别没认出来），Hugging Face 标的是 `apache-2.0`——**两边不是打架，是 GitHub 没识别**，别被页面图标误导。
- Mask2Alpha 不在 main 分支，在 `mask2alpha` 分支；仓库里**只有那个小头的权重**，SAM2.1-B+ 的主权重要自己 wget 下载，且会校验 SHA256。

## 评测数字（先别信）

README「Performance at a Glance」表的口径是 `against the strongest open baseline in each benchmark`——**每个基准各挑一个最强对手**，不是同一对手全场景对比，而且全是团队自测。arXiv 摘要原话：`Across over 30 benchmarks, WeVisOne-VL demonstrates leading performance`。

几行原文读数（4B 对 Best Baseline）：Dense200 密集检测 71.3 对 58.7；RefCOCOg 指代定位 81.8 对 77.6；WeRef-Bench 拒绝率 96.4 对 73.0；TotalText OCR 52.0 对 43.3。

同一份 README 的完整表里还有反例：P3M-500-NP 抠图这项 **2B 是 6.6、4B 是 7.3**（MAD 越低越好），大模型反而输小模型。所以「越大越好」不成立，我们测的时候两档都要试。

缺席断言说明：「没有第三方复现」这条，依据是仓库里 `evaluation/README.md` 是官方自己的复现说明、且 README 未列出任何外部评测机构成绩；我们本地未找到独立第三方榜单截图。**这条属于「没提到」，不等于「不存在」**，如果队友查到第三方评测请补进来。

## 链接（2026-11-05 全部实测可访问）

- 代码：https://github.com/Tencent/WeVisOne-VL （main / mask2alpha / page 三个分支）
- 4B 权重：https://huggingface.co/tencent/WeVisOne-VL-4B
- 2B 权重：https://huggingface.co/tencent/WeVisOne-VL-2B
- 官方 demo（试效果用这两个，零安装）：
  - https://huggingface.co/spaces/hugging-apps/wevisone-vl-2b
  - https://huggingface.co/spaces/hugging-apps/wevisone-vl-4b
- 项目主页：https://tencent.github.io/WeVisOne-VL/
- 技术报告：https://arxiv.org/abs/2610.31329

## 可信度声明

- **读来的**（抄自原文，可逐字复核）：上面所有英文引号内的句子、TODO 勾选状态、任务表条目、Limitations 三条。
- **数出来的**（命令实测）：显存 2048 MiB、权重 4.93/9.71 GB、许可证字段、5 个链接返回 200、GitHub 979 星。
- **估的**（不可当事实引用）：「租 GPU 多少钱」——没查价；「我们的类别它认不认得」——没拿我们的图测过，这正是下一步要做的。

## 下一步

见仓库 issue：先用官方 demo 人工试 20 张我们自己的图，证伪「它对我们的数据到底准不准」，再谈花钱租卡。
