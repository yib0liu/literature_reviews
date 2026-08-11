# OmniMotion-X：通用多模态全身人体动作生成

**OmniMotion-X: Versatile Multimodal Whole-Body Motion Generation**

> **作者**：Guowei Xu¹\*、Yuxuan Bian²\*、Ailing Zeng⁵†、Mingyi Shi³、Shaoli Huang⁴、Wen Li¹†、Lixin Duan¹、Qiang Xu²
>
> ¹ 电子科技大学² 香港中文大学（中国香港）　³ 香港大学（中国香港）　⁴ 腾讯 AI Lab　⁵ 独立研究者
>
> \* 同等贡献　† 通讯作者
>
> **arXiv**：2510.19789　|　**发表**：CVPR 2026 Findings, pp. 3641–3652
> **原文由 LaTeXML 生成于** 2025-10-22

---

## 译者说明

1. **命名警告（重要）**：本文是 **OmniMotion-X**（arXiv:2510.19789），与另一篇 **OmniMotion**（arXiv:2510.14954，Continuous Masked Autoregression）是**完全不同的两篇论文**——不同作者、不同方法，仅差一个 `-X` 后缀。引用时务必附带 arXiv 编号以避免混淆。

2. **与 MotionCraft 的关系**：本文是 MotionCraft（arXiv:2407.21136）的直接续作，作者列表中 Yuxuan Bian、Ailing Zeng、Qiang Xu 均为 MotionCraft 原班人马。文中多处直接批评 MotionCraft 的"额外控制分支"设计限制了条件之间的交互。

3. **关于"多条件"的关键辨析**：本文表 1 中有一列名为 **"Mixed-condition"（混合条件）**，原文定义为"训练期间多个条件同时出现"。该列判定MotionCraft 为"无"，OmniMotion-X 为"有"。但需注意：**本文的实验表 4/5/6/7 依然按 T2M / GSTC / S2G / M2D 分任务报告指标，全文没有任何一个"两个条件同时给定"的联合定量评测**。混合条件体现在训练阶段，不体现在评测指标上。

4. **术语处理**：核心术语首次出现给出中英对照，后续沿用缩写（T2M / M2D / S2G / GSTC / HOI / HSI / HHI 等）。数学符号、指标名（FID、R-Precision、TTR 等）保留原文。表格内的对勾/叉号统一译为"有/无"以避免符号渲染问题。

5. **完整性**：本译文完整覆盖正文全部章节 + 全部 8 个表格 + 补充材料（Supplementary Material）全部 5 节 + 完整参考文献。

---

## 摘要

本文介绍 **OmniMotion-X**，一个用于全身人体动作生成的通用多模态框架，它以统一的**序列到序列（sequence-to-sequence）**方式利用**自回归扩散 Transformer**（autoregressive diffusion transformer）。

OmniMotion-X 高效支持多样的多模态任务——包括**文本到动作**（text-to-motion）、**音乐到舞蹈**（music-to-dance）、**语音到手势**（speech-to-gesture），以及**全局时空控制场景**（例如动作预测、中间帧插值、动作补全，以及关节/轨迹引导的合成）——同时也支持这些任务的**灵活组合**。

具体而言，我们提出使用**参考动作（reference motion）作为一种新颖的条件信号**，显著增强了生成内容、风格与时间动态的一致性，这对于逼真的动画至关重要。为处理多模态冲突，我们引入了**渐进式"由弱到强"混合条件训练策略**（progressive weak-to-strong mixed-condition training strategy）。

为实现高质量的多模态训练，我们构建了 **OmniMoCap-X**——迄今最大的统一多模态动作数据集，整合了 **28 个公开可获取的动捕数据源**、覆盖 **10 个不同任务**，全部标准化为 **30 fps 的 SMPL-X 格式**。为确保标注的细致与一致，我们把动作序列**渲染成视频**，并使用 **GPT-4o** 自动生成结构化、层级化的文本描述，同时捕捉低层级动作与高层级语义。

大量实验评估证实，OmniMotion-X 显著超越现有方法，在多个多模态任务上取得当前最优（state-of-the-art）性能，并能够**交互式地生成逼真、连贯、可控的长时程动作**。

> **摘要图说明**：我们提出 OmniMotion-X，一个统一的序列到序列自回归动作扩散 Transformer，专为灵活、交互式的全身人体动作生成而设计。它支持多种任务，包括文本到动作、音乐到舞蹈、语音到手势，以及全局时空可控的动作生成（涵盖动作预测、中间帧插值、动作补全与关节/轨迹引导的合成）。这些条件可以以各种方式组合，从而实现通用的动作生成。

---

## 1 引言

### 表 1：OmniMotion-X 与现有人体动作生成方法的对比

> **表1 说明**："GSTC(S)" 与 "GSTC(D)" 表示全局时空可控动作生成（Global Spatial-Temporal Controllable motion generation），其中 "S" 与 "D" 分别表示**稀疏**与**稠密**受控关节。"Reference Motion"（参考动作）来源于用户设计的动作或先前生成的动作。**"Mixed-condition"（混合条件）指训练期间多个条件同时出现。**"Datasets" 表示用于训练的数据集总数，"Hours" 表示最长训练数据集的时长。对于像 MoMask 这类在 HumanML3D 或 KIT 上分别训练的方法，采用 HumanML3D 数据集的时长。

| 模型类型 | 方法 | T2M | M2D | S2G | GSTC(S) | GSTC(D) | 参考动作 | 混合条件 | 全身 | 数据集数 | 时长(小时) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| DiT | MDM (Tevet et al. 2022) | 有 | 无 | 无 | 无 | 无 | 无 | 无 | 无 | 2 | 28.6 |
| DiT | MCM (Ling et al. 2023) | 有 | 有 | 有 | 无 | 无 | 无 | 无 | 无 | 3 | 109.8 |
| DiT | LMM (Zhang et al. 2024c) | 有 | 有 | 有 | 无 | 无 | 无 | **有** | 有 | 16 | – |
| DiT | **MotionCraft (Bian et al. 2024)** | 有 | 有 | 有 | 无 | 无 | 无 | **无** | 有 | 3 | 48.4 |
| AR | MoMask (Guo et al. 2024a) | 有 | 无 | 无 | 无 | 无 | 无 | 无 | 无 | 2 | 28.6 |
| AR | MotionGPT (Jiang et al. 2024a) | 有 | 无 | 无 | 无 | 无 | 无 | 无 | 无 | 2 | 28.6 |
| AR | $M^3$GPT (Luo et al. 2024) | 有 | 有 | 无 | 无 | 无 | 无 | 有 | 无 | 3 | 164 |
| AR | MotionLLaMA (Ling et al. 2024) | 有 | 有 | 有 | 无 | 有 | 无 | **有** | 无 | 11 | 70.1 |
| AR-DiT | AMD (Han et al. 2024) | 有 | 有 | 无 | 无 | 无 | 有 | 无 | 无 | 4 | 85.87 |
| AR-DiT | DART (Zhao, Li, and Tang 2024) | 有 | 无 | 无 | 有 | 有 | 有 | 有 | 无 | 2 | 43.5 |
| AR-DiT | **OmniMotion-X（本文）** | **有** | **有** | **有** | **有** | **有** | **有** | **有** | **有** | **28** | **286.2** |

多模态全身人体动作生成在**动画**（Chen et al. 2024b）、**游戏**（Liao et al. 2024）、**虚拟现实**（Guo et al. 2024b）与**具身智能**（Mao et al. 2024）等领域扮演着关键角色，涉及包括文本、音频、轨迹等多样的输入条件。

**有限的多模态数据与任务特定的模型设计**，阻碍了现有动作生成方法支持多模态全身人体动作生成。由于动捕数据采集与标注成本高昂，大多数数据集只聚焦于单一领域，例如文本到动作（T2M）（Guo et al. 2022；Plappert, Mandery, and Asfour 2016）、音乐到舞蹈（M2D）（Li et al. 2023, 2021）、语音到手势（S2G）（Liu et al. 2023；Yi et al. 2023）、人-物交互（HOI）（Bhatnagar et al. 2022；Fan et al. 2023）、人-场景交互（HSI）（Jiang et al. 2024b）、人-人交互（HHI）（Liang et al. 2024b；Xu et al. 2024b），且各自具有不一致的数据格式（例如 BVH、SMPL-(H/X) 与 3D 关键点）与控制条件（例如文本描述、音频与轨迹）。

要应对跨多样场景的动作生成，关键在于构建一个**统一框架**，利用大规模、多样化的数据来获得更泛化的表征。

尽管近期工作（Ling et al. 2023；Zhang et al. 2024c；Bian et al. 2024；Ling et al. 2024；Han et al. 2024）尝试在一个模型中统一多任务，但它们在**多模态控制建模、任务通用性与高质量动作生成**方面仍面临挑战（如表 1 所示）：

**（1）独立模型训练（Independent Model Training）。** 以往方法为每个模态训练单独的模型，限制了跨输入的同时控制（Han et al. 2024）。

**（2）额外控制分支（Additional Control Branches）。** 一些方法为每个条件添加独立的控制分支，**限制了条件之间的交互**（Bian et al. 2024；Ling et al. 2023）。

**（3）粒度冲突训练（Conflict Granularity Training）。** 现有方法通过把高层语义条件与低层控制组合起来做混合训练，这**妨碍了不同层级上的有效控制**，并导致优化困难（Zhang et al. 2024c；Luo et al. 2024；Ling et al. 2024）。类似现象在视频生成中也被观察到（Lin et al. 2025；Ao 2024）。

除建模挑战之外，相关工作也引入了来自多任务、多模态的大规模动作数据集（Lin et al. 2024；Zhang et al. 2024c；Liang et al. 2024a；Lu et al. 2024b；Ling et al. 2024），但它们仍存在以下显著缺陷（对比见表 2）：

**（1）动作质量低（Low Motion Quality）。** 用非动捕的动作估计数据扩充的数据集会表现出"垃圾进、垃圾出"（garbage in, garbage out）效应，导致动作质量低劣（Lin et al. 2024；Lu et al. 2024b；Zhang et al. 2024c）。

**（2）文本不一致（Text Inconsistency）。** 不一致的文本标注或由大语言模型扩写的描述，会导致文本质量参差不齐与幻觉问题（Ling et al. 2024；Lu et al. 2024b；Liang et al. 2024a）。

**（3）任务受限（Limited Tasks）。** 它们只聚焦于常见任务，限制了在 HOI、HSI、HHI 等多样任务中的适用性（Zhang et al. 2024c；Bian et al. 2024；Lu et al. 2024b）。

这些限制阻碍了面向多样场景的、统一且高质量的多模态全身人体动作生成数据集的发展。

### 表 2：OmniMoCap-X 与现有合并数据集的对比

> **表 2 说明**："Mocap Source"（动捕来源）表示动捕数据集所占比例。"Caption Source"（文本来源）指明补全缺失描述的方法："–"（不补全）、"V"（视觉信息）、"T"（文本信息）。"Hierarchical Caption"（层级文本）表示文本描述是否包含层级结构。

| 数据集 | T2M | M2D | S2G | HOI | HSI | HHI | 全身 | 动捕/总数 | 文本来源 | 层级文本 | 帧数 | 时长(小时) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Motion-X (2023-07) (Lin et al. 2024) | 有 | 无 | 无 | 无 | 无 | 无 | 有 | 2 / 9 | T (9) | 有 | 15.6M | 144.2 |
| OMG (2024-03) (Liang et al. 2024a) | 有 | 无 | 无 | 无 | 无 | 有 | 无 | 9 / 13 | – | 无 | 22.3M | 206.5 |
| MotionUnion (2024-11) (Lu et al. 2024b) | 有 | 无 | 无 | 无 | 无 | 无 | 无 | 4 / 15 | – | 无 | 30M | 260 |
| MotionVerse (2024-04) (Zhang et al. 2024c) | 有 | 有 | 有 | 无 | 无 | 无 | 有 | 5 / 16 | – | 无 | 100M | – |
| MotionHub (2024-11) (Ling et al. 2024) | 有 | 有 | 有 | 无 | 无 | 有 | 有 | 8 / 19 | V (3) / T (7) | 无 | – | 70.1 |
| **OmniMoCap-X（本文）** | **有** | **有** | **有** | **有** | **有** | **有** | **有** | **21 / 28** | **V + T (28)** | **有** | **64.3M** | **286.2** |

### 本文方案概述

为应对现有挑战，我们提出 **OmniMotion-X**（一个用于多模态全身人体动作生成的统一框架）与 **OmniMoCap-X**（最大的统一多模态动捕动作数据集）。

具体而言，OmniMotion-X 采用**扩散 Transformer**（Diffusion Transformers, DiTs）（Peebles and Xie 2023），通过**把条件 token 拼接为前缀上下文（prefix context）**来纳入多模态条件，并采用**渐进式由弱到强的混合条件训练策略**，逐步把动作约束从高层语义收紧到稠密的时空对齐。

值得注意的是，与以往方法不同，OmniMotion-X 引入了一种**新的生成范式**：把**参考动作**（例如用户提供的或模型预测的动作）用作一种**特殊条件**。这显著提升了生成动作的质量，并实现了参考动作与生成动作之间的一致性，从而构建出一种有效的**逐片段（clip-by-clip）自回归动作扩散**。这使 OmniMotion-X 能够支持具有强时间对齐性的**自回归交互式生成**。

此外，OmniMotion-X 通过**时空掩码策略**统一了各种全局时空可控生成任务。

为保证高质量的动作生成训练，我们收集了支持多样动作生成任务的高质量动捕数据集，将它们统一到具有标准世界坐标系的 **SMPL-X**（Pavlakos et al. 2019）格式下，并通过把动作渲染成视频、再用**视觉语言模型（VLMs）**标注的方式自动生成**层级文本描述**。该数据集包含约 **286.2 小时**数据，整合了多模态控制条件，支持包括 T2M、M2D、S2G、HOI、HSI 与 HHI 在内的通用任务。

### 核心贡献

综上，我们的核心贡献如下：

- 我们提出 **OmniMotion-X**，一个用于通用全身动作生成的**多模态自回归扩散 Transformer**。通过引入参考动作，OmniMotion-X 显著增强了内容、风格与时间动态的一致性生成。

- 我们提出**渐进式由弱到强的条件化策略**，以有效处理**多粒度约束**。

- 我们构建 **OmniMoCap-X**，最大的统一多模态动捕动作数据集，采用统一的 SMPL-X 格式，并提供一致、细致、结构化的文本描述，以服务于通用动作生成任务。

- 大量实验表明 OmniMotion-X 在包括 T2M、S2G、M2D 与 GSTC 等各类任务上均达到当前最优性能。这些评估是在我们**更具挑战性的测试集**上进行的，该测试集包含从 OmniMoCap-X 中跨多样场景均匀采样的 **280 个样本**。

---

## 2 相关工作

### 2.1 人体动作生成

现有方法通常依据输入条件被划分为三大类：**单模态、跨模态与多模态**。

**单模态动作生成**（Single-modal）使用与动作**同一模态**的控制条件，包括动作预测（Xu et al. 2024a；Chen et al. 2023a）、中间帧插值（in-betweening）（Cohan et al. 2024）、以及关节/轨迹引导的合成（Xie et al. 2024；Dai et al. 2024；Zhao, Li, and Tang 2024）。

**跨模态动作生成**（Cross-modal）使用来自**不同模态**的控制条件，包括：T2M 中的文本（Tevet et al. 2022；Chen et al. 2023b；Guo et al. 2024a；Lu et al. 2024a；Jiang et al. 2024a；Zhang et al. 2024d；Liang et al. 2024a；Zhang et al. 2023b, 2024b）、M2D 中的音乐（Li et al. 2023；Le et al. 2023；Siyao et al. 2022；Tseng, Castellon, and Liu 2023）、S2G 中的语音（Liu et al. 2023；Yi et al. 2023；Feng, Shin, and Yoon 2022；Chen et al. 2024a）、HOI 中的目标物体位置（Fan et al. 2023；Liu et al. 2024）、HSI 中的场景布局（Kaufmann et al. 2023；Huang et al. 2022；Jiang et al. 2024b），以及 HHI 中的伙伴动作（Xu et al. 2024b；Liang et al. 2024b）。

然而，这些方法通常依赖**任务特定的架构**，显著限制了它们的跨任务泛化能力与实际应用。

近来，研究者（Han et al. 2024；Ling et al. 2023；Zhang et al. 2024c；Bian et al. 2024；Luo et al. 2024）开始探索**多模态动作生成**，其方法可归为三大类：

- **分离模型训练（Separate Model Training）。** 这类方法（Han et al. 2024）通常为不同的条件模态训练**独立**的模型，使多模态动作生成**从根本上无法实现**。

- **额外控制分支（Additional Control Branch）。** 这类方法（Bian et al. 2024；Ling et al. 2023）在骨干架构中加入独立的控制分支，每个分支专用于一个特定条件，**导致不同条件之间的交互受限**。

- **统一多模态条件建模（Unified Multimodal Condition Modeling）。** 这类方法（Zhang et al. 2024c；Ling et al. 2024）用**单一模型**学习从多样模态到目标动作的映射，增强了多模态适应性。**然而，挑战来自各模态之间不同的约束性质**：文本提供语义引导，时空控制施加物理约束，而音频输入强制节奏对齐。这些**性质迥异的约束**常常导致可控性受损与优化困难。

### 2.2 人体动作数据集

当前的人体动作生成数据集**以任务特定为主**。

不同任务依赖各自独立的数据收集：文本到动作（T2M）使用带文本描述的数据集（Guo et al. 2022；Plappert, Mandery, and Asfour 2016；Lin et al. 2024；Punnakkal et al. 2021；Liao et al. 2024）；音乐到舞蹈（M2D）需要带音乐标注的数据集（Li et al. 2021, 2023；Le et al. 2023；Chen et al. 2021）；语音到手势（S2G）采用带配对语音的数据集（Liu et al. 2023；Yi et al. 2023；Liu et al. 2022；Feng, Shin, and Yoon 2022）。

同样地，交互类数据集在人-物（Bhatnagar et al. 2022；Fan et al. 2023；Jiang et al. 2022；Li, Wu, and Liu 2023；Zhang et al. 2024a；Liu et al. 2024；Zhan et al. 2024）、人-场景（Jiang et al. 2024b）与人-人交互（Liang et al. 2024b；Xu et al. 2024b）之间彼此割裂。这种**碎片化**限制了多模态动作生成能力。

整合这些数据集面临**动作格式不一致**的挑战。各数据集在表征上差异巨大：一些使用 SMPL（Bhatnagar et al. 2022；Liang et al. 2024b；Li et al. 2021），另一些采用 SMPL-X（Liu et al. 2023；Yi et al. 2023；Feng, Shin, and Yoon 2022），还有许多依赖 BVH（Harvey et al. 2020；Mason, Starke, and Komura 2022；Adobe；Alexanderson et al. 2023；Valle-Pérez et al. 2021）或关键点表征（Ionescu et al. 2013；Ji et al. 2018）。这种多样性使格式标准化极为困难。

近期工作（Lin et al. 2024；Liang et al. 2024a；Lu et al. 2024b；Zhang et al. 2024c；Ling et al. 2024）已尝试整合多个数据集，扩大规模并纳入部分多模态条件。然而它们仍存在若干局限：

- **低质量动作（Low-quality Motions）**：大多数整合数据集源自非动捕来源，存在显著估计误差（Lin et al. 2024；Lu et al. 2024b；Zhang et al. 2024c；Ling et al. 2024），造成"垃圾进、垃圾出"效应，损害动作生成质量。

- **文本不一致（Text Inconsistency）**：现有数据集（Liang et al. 2024a；Zhang et al. 2024c；Lu et al. 2024b）使用不一致的文本标注（例如动作标签 vs. 语义描述），或在**没有视觉动作数据**的情况下依赖大语言模型做文本精修，导致文本质量波动与幻觉（Lin et al. 2024；Ling et al. 2024）。

- **生成场景受限（Limited Generation Scenarios）**：大多数数据集不支持关键的交互任务，包括 HOI、HSI（Lin et al. 2024；Liang et al. 2024a；Lu et al. 2024b；Zhang et al. 2024c；Ling et al. 2024）与 HHI（Lin et al. 2024；Lu et al. 2024b；Zhang et al. 2024c），严重限制了它们在复杂场景中的适用性。

---

## 3 OmniMoCap-X 数据集

为应对数据层面的挑战——高质量多任务动作数据集稀缺、文本标注与动作表征不一致——我们实施了**三管齐下**的方案：（1）整合支持多任务的多样动捕数据集；（2）建立统一的动作表征；（3）开发源自视觉-文本标注的高质量动作描述。

### 表 3：OmniMoCap-X 数据集构成

> **表 3 说明**：把动作格式统一为带文本描述的 SMPL-X。我们跨各类任务挑选了 **28 个公开可获取的高质量数据集**。帧数与时长基于原始数据集 FPS 计算。MoCap 列表示数据采集方式，按质量排序为：**带人工校正的 Marker（Marker-M）**、**Vicon Marker（Marker-V）**、**IMU**、**多视角RGB（MV-RGB）**、**单视角 RGB（SV-RGB）**。Format 列指明原始动作格式。

| 任务 | 数据集 | 帧数 | 时长(小时) | 动捕方式 | 原始格式 |
|---|---|---|---|---|---|
| **T2M** | Mixamo (Adobe) | 0.4M | 1.9 | Marker-M | BVH |
| T2M | KIT (Plappert, Mandery, and Asfour 2016) | 2.3M | 6.4 | Marker-V | SMPL |
| T2M | OMOMO (Li, Wu, and Liu 2023) | 1.1M | 2.5 | Marker-V | SMPL-X |
| T2M | IDEA400 (Lin et al. 2024) | 1.7M | 15.7 | SV-RGB | SMPL-X |
| T2M | 100Style (Mason, Starke, and Komura 2022) | 4.8M | 22.1 | IMU | BVH |
| T2M | HumanML3D (Guo et al. 2022) | 26.5M | 65.7 | Marker-V | SMPL |
| **M2D** | Choreomaster (Chen et al. 2021) | 0.1M | 1.2 | Marker-M | FBX |
| M2D | FineDance (Li et al. 2023) | 0.8M | 7.7 | Marker-V | SMPL-H |
| M2D | PhantomDance (Li et al. 2022) | 1.0M | 9.5 | Marker-M | SMPL |
| M2D | AIST++ (Li et al. 2021) | 1.1M | 5.2 | MV-RGB | SMPL |
| M2D | Motorica (Alexanderson et al. 2023) | 2.7M | 12.4 | Marker-V | BVH |
| M2D | AIOZ (Le et al. 2023) | 6.6M | 60.8 | SV-RGB | SMPL |
| **S2G** | BEAT2 (Liu et al. 2023) | 6.9M | 64.2 | Marker-V | SMPL-X |
| **HHI** | HumanSC3D (Fieraru et al. 2021a) | 0.2M | 1.1 | Marker-V | SMPL-X |
| HHI | InterHuman (Liang et al. 2024b) | 4.5M | 20.8 | MV-RGB | SMPL |
| HHI | Inter-X (Xu et al. 2024b) | 16.2M | 37.5 | Marker-V | SMPL-X |
| **HOI** | ARCTIC (Fan et al. 2023) | 0.2M | 2.0 | Marker-V | SMPL-X |
| HOI | TACO (Liu et al. 2024) | 0.4M | 3.3 | Marker-V | MANO |
| HOI | Fit3D (Fieraru et al. 2021b) | 0.4M | 2.5 | Marker-V | SMPL-X |
| HOI | BEHAVE (Bhatnagar et al. 2022) | 0.4M | 4.1 | MV-RGB | SMPL-X |
| HOI | CHAIRS (Jiang et al. 2022) | 1.0M | 9.5 | Marker-V | SMPL-X |
| HOI | HOI-M³ (Zhang et al. 2024a) | 2.3M | 10.7 | MV-RGB | SMPL |
| HOI | OakInk-v2 (Zhan et al. 2024) | 4.0M | 36.8 | Marker-V | SMPL-X |
| **HSI** | EMDB (Kaufmann et al. 2023) | 0.03M | 0.3 | IMU | SMPL |
| HSI | RICH (Huang et al. 2022) | 0.1M | 0.8 | MV-RGB | SMPL-X |
| HSI | LaFAN1 (Harvey et al. 2020) | 1.0M | 4.5 | Marker-V | BVH |
| HSI | TRUMANS (Jiang et al. 2024b) | 1.2M | 11.2 | Marker-V | SMPL-X |
| HSI | CIRCLE (Araújo et al. 2023) | 4.4M | 10.1 | Marker-V | SMPL-X |
| **全部** | **OmniMoCap-X** | **64.3M** | **286.2** | 混合 | **SMPL-X** |

### 3.1 动捕数据集构成

现有方法通过纳入大量非动捕数据来**优先追求数据集规模而非质量**（Lin et al. 2024；Lu et al. 2024b；Zhang et al. 2024c；Ling et al. 2024），导致动作生成质量退化（"垃圾进、垃圾出"）。**我们的工作强调数据质量。**

我们系统性地整理了开源动捕数据集，并把它们归为五种采集类别：**带人工校正的 Marker、Marker（Vicon）、IMU、多视角 RGB、单视角 RGB**（详见表 3）。

我们的数据集支持多样的动作生成任务，包括文本到动作、音乐到舞蹈、语音到手势，以及各类交互（人-物、人-场景、人-人），提供了高质量的多模态基础。总计 **6430 万帧**、**286.2 小时**数据，我们的方案在维持更优数据质量的同时实现了全面的任务覆盖。

### 3.2 统一动作表征

我们把多样的动作格式（BVH（Lauterbach et al. 2009）、FBX（Wikipedia contributors 2024）与 SMPL(-H)（Loper et al. 2023））标准化为 **SMPL-X**（Pavlakos et al. 2019）格式，以支持我们的统一多模态模型。

首先，我们把这些格式转换为 **Motion-X**（Lin et al. 2024）的 SMPL-X 格式，包含根方向（root orientation）、姿态参数（身体、手部与下颌）、面部表情、面部形状、平移与身体形状参数。

随后，我们跨数据集**归一化平移尺度与初始根方向**，以建立一致的坐标系。此外，我们把所有数据集**重采样到 30 fps**，以增强模型捕捉时序模式的能力。不同数据集的具体转换过程与归一化方式见补充材料。

为增强泛化能力，我们把广泛使用的**仅身体表征**（Guo et al. 2022）扩展为**全身格式**。

具体而言，第 $i$ 帧的姿态是一个元组：

$$\mathbf{p}_{i}=(\dot{r}^{a},\ \dot{r}^{x},\ \dot{r}^{z},\ r^{y},\ \mathbf{j}^{p},\ \mathbf{j}^{v},\ \mathbf{c}^{f},\ \mathbf{f})$$

其中：

- $\dot{r}^{a}\in\mathbb{R}$ 是绕 **Y 轴**的**根角速度**；
- $\dot{r}^{x},\dot{r}^{z}\in\mathbb{R}$ 是 **XZ 平面**内的**根线速度**；
- $r^{y}\in\mathbb{R}$ 是**根高度**；
- $\mathbf{j}^{p}\in\mathbb{R}^{3N-1}$ 是**局部关节位置**；
- 特别地，$\mathbf{j}^{r}\in\mathbb{R}^{6N'}$ 是 SMPL-X 的**关节 6D 旋转**；
- $\mathbf{j}^{v}\in\mathbb{R}^{3N}$ 是**关节速度**；
- $\mathbf{c}^{f}$ 是通过对**脚跟与脚尖关节速度**做阈值化得到的**二值特征**，用于强调**足部与地面的接触**。

这里 $N$ 指从 SMPL-X（Pavlakos et al. 2019）中提取的 **127 个全身关节**，而 $N'$ 表示涵盖身体、手部与下颌的 **53 个关节**。

对于面部表征，我们采用 **FLAME 格式**（Li et al. 2017），把面部特征编码为 $\mathbf{f}\in\mathbb{R}^{100}$。

> **图 1**：OmniMotion-X 总览——一个用于全身人体动作生成的统一多模态自回归Transformer 扩散模型。OmniMotion-X 通过条件特定的编码器把**文本、全局动作、语音、音乐与参考动作**映射进统一空间并作为条件整合。模型融合多模态信息以产生连贯的动作，其中时空引导确保了全局动作特征的一致性。

### 3.3 一致的视觉-文本动作描述

当前数据集采用三种标注方式：

- **大语言模型精修（LLM Refinement）**（Lin et al. 2024；Lu et al. 2024b）使用像 GPT-4（Achiam et al. 2023）这样的 LLM 来增强文本描述。**然而，由于 LLM 无法直接感知动作数据、仅依赖已有文本，它们频繁引入幻觉，且无法捕捉精确的动作细节。**

- **人工标注（Manual Annotation）**（Guo et al. 2022；Punnakkal et al. 2021；Ling et al. 2024）能有效消除幻觉，但成本高昂且难以规模化，导致标注量有限、描述简单。

- **基于动作到文本模型的标注（Motion-to-Text Model-Based）**（Delmas et al. 2022；Yazdian et al. 2023）直接从原始动作生成描述。虽然这类方法能有效捕捉低层运动学细节（例如"左肘弯曲 45 度"），但**缺乏对高层语义与动作上下文的理解**。

为克服这些局限，我们提出一种**同时整合视觉与文本信息**的自动化方法进行全面的动作标注。

我们的方法把动作**渲染成视频**，并将其与已有的文本标注（例如描述、动作标签与任务类别）组合，作为输入送给当前最优的视觉语言模型（VLMs）**GPT-4o**（Achiam et al. 2023）。

这种多模态方式生成**结构化、层级化、精确**的动作描述，同时保证标注质量与表达丰富度。文本质量的详细统计见补充材料。

---

## 4 方法

### 表 4：OmniMoCap-X 测试集上的文本到动作定量结果

> **表 4 说明**：$\uparrow$（$\downarrow$）表示数值越大（越小）越好。$\rightarrow$ 表示越接近 GT 越好。**红色与蓝色分别表示最优与次优结果**（本译文以粗体标注 OmniMotion-X 结果）。所有评估重复 **20 次**，报告均值与 **95% 置信区间**。带`*` 的方法表示**在我们的 OmniMoCap-X 上重新训练**。"RM" 表示使用参考动作（Reference Motion）。

| 方法 | R-Prec Top-1 ↑ | Top-2 ↑ | Top-3 ↑ | FID ↓ | Multimodal Dist. ↓ | Diversity → | MultiModality ↑ |
|---|---|---|---|---|---|---|---|
| GT（真值） | 0.535±0.009 | 0.725±0.009 | 0.821±0.008 | 0.013±0.005 | 2.493±0.011 | 9.194±0.093 | – |
| MDM (Tevet et al. 2022) | 0.063±0.004 | 0.121±0.003 | 0.169±0.003 | 72.928±0.361 | 8.376±0.018 | 1.981±0.003 | 0.394±0.010 |
| MLD (Chen et al. 2023b) | 0.084±0.002 | 0.152±0.002 | 0.209±0.003 | 70.082±0.468 | 8.281±0.025 | 1.998±0.003 | 0.439±0.022 |
| MoMask (Guo et al. 2024a) | 0.104±0.003 | 0.163±0.003 | 0.199±0.005 | 69.361±0.365 | 8.341±0.022 | 2.123±0.002 | 0.482±0.008 |
| MoMask\* (Guo et al. 2024a) | 0.267±0.004 | 0.414±0.004 | 0.530±0.004 | 17.428±0.400 | 5.661±0.024 | 6.772±0.013 | 0.811±0.067 |
| MotionCraft (Bian et al. 2024) | 0.176±0.004 | 0.259±0.003 | 0.319±0.004 | 63.049±0.470 | 7.936±0.021 | 2.325±0.013 | 0.557±0.039 |
| MotionCraft\* (Bian et al. 2024) | 0.236±0.004 | 0.370±0.004 | 0.489±0.004 | 47.428±0.400 | 7.424±0.024 | 2.820±0.013 | 0.863±0.067 |
| **OmniMotion-X（本文）** | **0.303±0.004** | **0.464±0.006** | **0.571±0.005** | **5.040±0.293** | **4.678±0.019** | **8.650±0.095** | **1.696±0.366** |
| **OmniMotion-X（本文 + RM）** | **0.346±0.005** | **0.511±0.007** | **0.629±0.006** | **3.199±0.293** | **4.106±0.019** | **8.009±0.095** | **1.143±0.366** |

OmniMotion-X 的总览如图 1 所示。我们提出一个用于跨多任务全身人体动作生成的**统一多模态、自回归、基于 Transformer 的扩散模型**。

该模型接受：
- **文本描述** $\mathbf{c}_{t}$ 用于**语义引导**；
- **全局动作** $\mathbf{c}_{g}$ 用于**时空控制**；
- **语音** $\mathbf{c}_{s}$ 与 **音乐条件** $\mathbf{c}_{m}$ 以确保**节奏与风格连贯性**。

此外，它把**参考动作** $\mathbf{c}_{r}$ 作为**动作先验（motion prior）**纳入，该先验源自先前生成的片段或用户设计的动作，从而提供其他条件无法给出的**细粒度细节**。

再者，**混合条件设定**允许训练期间**多个条件同时出现**，使模型能够处理具有多样输入的复杂场景。

本节分两部分介绍我们的方法：**多模态建模的统一框架**，以及**确保精确动作控制的渐进训练策略**。

> **图 2**：我们提出**由弱到强的渐进训练策略**——先与文本建立"动作-语义对齐"，随后**渐进地整合更强的多模态信号**（参考动作、全局动作、语音、音乐），以增强生成质量与可控性。

### 4.1 统一多模态建模

我们使用**扩散 Transformer**（DiTs）（Peebles and Xie 2023）整合多模态条件，方式是把**条件 token 拼接为前缀上下文**，使模型能够高效处理与融合多模态信息。

#### 多模态条件

我们的框架整合了多种条件模态：

- **文本** $\mathbf{c}_{t}$：提供语义引导；
- **全局动作** $\mathbf{c}_{g}$：确保时空一致性；
- **语音** $\mathbf{c}_{s}$：使手势与唇部动作同步于节奏；
- **音乐** $\mathbf{c}_{m}$：为舞蹈提供节拍与风格信息；
- **参考动作** $\mathbf{c}_{r}$：充当动作先验。

**值得注意的是，这个参考动作条件在以往的多模态动作生成工作中被忽视了**（Zhang et al. 2024c；Ling et al. 2023；Bian et al. 2024）——它使我们的模型能够与参考动作保持精确的时空模式一致性，大幅提升动作质量与连贯性。

为充分利用每个模态并促进不同条件之间的交互，我们采用**模态特定的编码器**：
- 文本用 **T5-XXL**（Raffel et al. 2020）；
- 语音用 **wav 编码器**（Liu et al. 2023）；
- 音乐用 **Librosa**（McFee et al. 2015）；
- 动作用**分部位编码（body-wise encoding）**（Bian et al. 2024）。

这些特征通过**可学习的线性投影**对齐到动作嵌入维度，使我们能够在处理时把所有条件 token 与带噪动作 token 拼接为前缀上下文。

#### 多模态建模公式

总体而言，统一的多模态建模表述如下：

$$c=[h_{t}(f_{t}(c_{t})),\ h_{g}(f_{g}(c_{g})),\ h_{s}(f_{s}(c_{s})),\ h_{m}(f_{m}(c_{m})),\ h_{r}(f_{r}(c_{r}))] \tag{1}$$

其中 $f$ 与 $h$ 分别表示模态特定编码器与投影层。拼接后的表征 $c$ 随后作为**条件前缀上下文**送入我们的 DiT 骨干网络。

为约束动作的物理属性，我们遵循以往工作（Tevet et al. 2022；Chen et al. 2024b），**直接预测动作 $\hat{x}_{0}$ 而非噪声**。因此，我们的扩散目标定义为：

$$\mathcal{L}_{\text{simple}}=\mathbb{E}_{x_{0}\sim q(x_{0}|c),\ t\sim[1,T]}\left[\left\|x_{0}-G(x_{t},t,c)\right\|_{2}^{2}\right] \tag{2}$$

其中 $q(x_{0}|c)$ 表示数据分布，$T$ 是最大扩散步数，$G$ 表示学到的去噪函数，而 $x_{t}$ 表示第 $t$ 步的带噪动作，表示为 $x_{t}=[p_{0}^{t},p_{1}^{t},\dots,\mathbf{p}_{N}^{t}]$，其中每个 $\mathbf{p}_{i}^{t}$ 对应第 $t$ 步动作中的第 $i$ 个姿态。

#### 渐进式由弱到强训练

考虑到我们多样的**多粒度条件输入**，我们从经验上观察到：**在单阶段训练范式中使用所有条件，会导致模型难以直接学习动作与条件之间的关联。**

更进一步，**模型倾向于过拟合那些约束性强的低层控制信号**，例如细粒度的参考动作与时空关节控制。虽然这些信号看起来占据主导，但它们会**压制其他模态（例如文本）**，最终损害整体可控性。

为解决这一问题，我们实施**由弱到强的渐进训练**：如图 2 所示，初始的文本条件学习建立"动作-语义对齐"，随后**渐进地整合更细粒度的条件**，包括参考动作、全局动作与音频信号。

这种渐进方式使模型能够有效适应不同条件，在精确遵循多模态条件的同时确保高质量、灵活的动作生成。

> **图 3**：OmniMotion-X 的多样动作合成能力。OmniMotion-X 支持多种任务：(a) 文本到动作，(b) 语音到手势，(c) 音乐到舞蹈，(d) 轨迹引导动作，(e) 动作中间帧插值，(f) 动作预测。

---

## 5 实验

### 表 5：OmniMoCap-X 测试集上的全局时空可控生成定量结果

| 方法 | FID ↓ | R-Precision Top-3 ↑ | Multimodal Distance ↓ |
|---|---|---|---|
| GT（真值） | 0.013 | 0.821 | 2.493 |
| OmniControl | 63.725 | 0.392 | 8.011 |
| **OmniMotion-X** | **4.224** | **0.682** | **4.377** |

### 表 6：BEAT2 上的 S2G 与 AIST++/FineDance/PhantomDance 上的 M2D 结果

> **表 6 说明**：我们分别为 S2G 评测 $FID_{WholeBody}$、$FID_{Hands}$、$Face_{MSE}$ 与 Diversity，为 M2D 评测 $FID_{WholeBody}$、$FID_{Hands}$ 与 Diversity。

| 方法 | S2G: FID(全身) ↓ | S2G: FID(手部) ↓ | S2G: $Face_{MSE}$ ↓ | S2G: Diversity ↑ | M2D: FID(全身) ↓ | M2D: FID(手部) ↓ | M2D: Diversity ↑ |
|---|---|---|---|---|---|---|---|
| MotionCraft | 3.422 | **5.370** | 0.182 | 1.003 | **9.875** | 7.099 | 3.798 |
| **OmniMotion-X（本文）** | **2.641** | 9.095 | **0.045** | **1.664** | 16.209 | **5.827** | **4.716** |

### 5.1 实现细节

我们的方法采用 **Transformer Encoder** 架构（Vaswani et al. 2017），共**8 层**、**8 个注意力头**，隐藏维度为 $d_{\text{model}}=1536\ (128\times 12)$，其中 **128** 表示 **12 个身体部位**中每个部位的嵌入大小，前馈维度为 **3072**。

训练在**单张 H800 GPU** 上通过渐进条件化完成：

| 阶段 | 条件 | 步数 | Batch Size |
|---|---|---|---|
| 1 | 仅文本 | 460K | 48 |
| 2 | + 参考动作 | 460K | 48 |
| 3 | + 全局时空控制 | 230K | 48 |
| 4 | + 完整音频条件 | 920K | 16 |

我们使用 **AdamW** 优化，初始学习率为 $1\times 10^{-4}$（**每引入新条件时重置**），并在前 **460K** 步内以 cosine 调度衰减到 $1\times 10^{-5}$。动作参考与预测的默认长度为 **150**（帧）。更多实现细节见补充材料。

### 5.2 定量结果

我们在四个任务上评估 OmniMotion-X：**文本到动作（T2M）、全局时空可控生成（GSTC）、音乐到舞蹈（M2D）与语音到手势（S2G）**。

对 T2M 与 GSTC 的评估，测试集从所有数据集中均匀采样（**每个数据集 10 个样本**）。M2D 基准整合了来自 AIST++（Li et al. 2021）、FineDance（Li et al. 2023）与 PhantomDance（Li et al. 2022）的测试序列，S2G 评估在 BEAT2（Liu et al. 2023）上进行。

#### T2M

表 4 显示，以往方法受小规模数据集约束，**在生成多样且复杂的动作时难以泛化**。

相比之下，OmniMotion-X 展现出显著优势，不仅超越在小数据集上训练的基线方法（MDM（Tevet et al. 2022）、MLD（Chen et al. 2023b）、MoMask（Guo et al. 2024a）、MotionCraft（Bian et al. 2024）），**也超越了在我们 OmniMoCap-X 上训练的 MoMask\* 与 MotionCraft\***。

与 MoMask\* 和 MotionCraft\* 相比，OmniMotion-X 使用了**更强的文本编码器 T5-XXL**（Raffel et al. 2020），它对复杂文本的建模比 CLIP（Radford et al. 2021）更有效。

此外，它采用了**训练时带参考动作的统一多模态框架**，使模型能够学到更细致的全身动作表征，从而带来更好的生成质量与泛化能力。

#### GSTC

我们采用 OmniControl（Xie et al. 2024）的**跨关节（cross-joint）设定**，通过控制**所有关节**来模拟空间稠密控制。

如表 5 所示，由于数据集规模较小，OmniControl（Xie et al. 2024）在 GSTC 任务上难以泛化。相比之下，我们的方法展现出明显优势，能有效遵循空间稠密的控制信号。

#### M2D 与 S2G

我们直接把本方法与 MotionCraft（Bian et al. 2024）在 S2G 与 M2D 任务上对比。

如表 6 所示，OmniMotion-X 取得了有竞争力的性能。**（在某些指标上）FID 较低（原文表述为 "The lower FID"，此处指本方法在部分 FID 指标上不占优）主要是由于 S2G 与 M2D 的测试集相对较小，而 OmniMotion-X 是在 OmniMoCap-X 上训练的，导致存在一定的分布差异。此外，我们的方法展现出更优的多样性，这也可能影响 FID。**

> 译者注：这一段是作者对表 6 中"M2D 全身 FID 反而比 MotionCraft 差（16.209 vs 9.875）"的解释——归因于测试集规模小造成的分布偏移，以及多样性提升与 FID 之间的固有张力。

### 5.3 消融研究

### 表 7：训练策略消融研究

> "w/o TrSt" 表示**去掉渐进训练策略**（without Training Strategy）。

| 任务 | 方法 | FID ↓ | R-Precision Top-3 ↑ | Multimodal Dist ↓ | Diversity → |
|---|---|---|---|---|---|
| **T2M** | w/o TrSt | 9.574 | 0.232 | 6.853 | 3.118 |
| T2M | **本文** | **5.040** | **0.571** | **4.678** | **8.650** |
| **GSTC** | w/o TrSt | 10.247 | 0.491 | 6.130 | 2.438 |
| GSTC | **本文** | **4.224** | **0.686** | **4.377** | **6.292** |

表 7 给出了消融结果，并展示了**两项关键发现**：

**（1）** 去掉由粗到细（coarse-to-fine）训练会**损害文本条件对齐**，表明**更细粒度的控制可能会覆盖（override）粗粒度的语义约束**。

**（2）** **混合条件训练会损害时空控制**，原因是物理约束引入了**优化冲突**，因此**必须**采用由弱到强的条件化策略。

> 译者注：这两条结论是本文最值得注意的负面结果。它们等于承认：把多粒度条件同时投入训练存在本质性的相互干扰，本文的解法是在**训练调度**上错开（时间维度上的解耦），而非在**表示层面**把不同粒度的条件分离到不同的因子上。

### 5.4 定性结果

如图 3 所示，OmniMotion-X 能处理各种多模态生成，包括 T2M、M2D、S2G 与 GSTC（涵盖动作预测、中间帧插值与关节引导）。当与参考动作结合时，模型生成的条件动作与参考动作保持一致。更多可视化见补充视频。

---

## 6 结论与讨论

本工作介绍了 **OmniMotion-X** 与 **OmniMoCap-X**。

**OmniMotion-X** 是一个自回归扩散模型，它整合了参考动作与多模态条件，以实现精确的全身动作控制。它采用**渐进式由弱到强的条件化策略**来有效处理多粒度约束。

**OmniMoCap-X** 是最大的多模态动捕数据集，包含来自 **28 个动捕数据集**的 **286.2 小时**数据，统一为 SMPL-X 格式，并跨 **10 个任务**提供结构化、一致的文本描述。

实验证明 OmniMotion-X 超越各基线，为大规模多模态动作生成奠定了坚实基础。

### 局限与未来工作

**尽管有效，我们的方法目前缺乏场景、物体与人际交互约束，限制了其在复杂真实世界应用中的使用。此外，样本空间（sample-space）的去噪过程导致推理速度较慢。**

未来工作应聚焦于：整合交互约束以实现更强通用性；以及开发**更高效的动作表征**，以改善物理一致性与推理速度。

---

# 补充材料（Supplementary Material）

## A 数据集文本质量

### 表 8：各动作数据集的文本质量统计

> **表 8 说明**：第三列为每个数据集的文本样本数。第四列为**平均句长（按词数计）**。第五列为 **Type-Token Ratio（TTR，类符/形符比）**范围，表示**词汇多样性**（数值越高代表词汇越丰富）。第六列列出每个数据集中**最高频的十个动词**，反映动作文本所描述的主要动作。数据集按其主要任务分组：T2M、M2D、S2G、HHI、HOI、HSI。

| 任务 | 数据集 | 文本数 | 平均句长 | TTR 范围 | Top-10 动词 |
|---|---|---|---|---|---|
| **T2M** | MIXAMO (Adobe) | 2,350 | 331.54 | 0.232–0.600 | swing, step, lean, walk, stand, lift, adjust, bend, stabilize, raise |
| T2M | KIT (Plappert et al. 2016) | 5,593 | 289.67 | 0.305–0.740 | stand, walk, leave, move, turn, step, extend, swing, follow, shift |
| T2M | OMOMO (Li, Wu, and Liu 2023) | 5,619 | 278.76 | 0.330–0.581 | bend, stand, reach, extend, straighten, move, step, lean, indicate, shift |
| T2M | IDEA400 (Lin et al. 2024) | 14,753 | 279.69 | 0.288–1.000 | walk, step, lift, raise, stand, coordinate, reach, lean, swing, adjust |
| T2M | 100Style (Mason et al. 2022) | 15,747 | 394.54 | 0.225–1.000 | leave, move, swing, lean, follow, step, lift, extend, walk, run |
| T2M | HumanML3D (Guo et al. 2022) | 48,398 | 232.44 | 0.139–0.817 | stand, bend, walk, lean, turn, control, lift, reach, step, raise |
| **M2D** | Choreomaster (Chen et al. 2021) | 2,699 | 279.87 | 0.324–0.629 | extend, shift, leave, move, raise, weight, lift, bend, transition, lean |
| M2D | FineDance (Li et al. 2023) | 5,540 | 353.40 | 0.288–0.632 | leave, extend, move, shift, weight, raise, step, lift, bend, lean |
| M2D | PhantomDance (Li et al. 2022) | 6,389 | 340.24 | 0.275–1.000 | leave, extend, move, raise, shift, lift, bend, weight, stand, lean |
| M2D | AIST++ (Li et al. 2021) | 3,183 | 405.28 | 0.260–0.673 | leave, extend, swing, move, weight, shift, lean, bend, lift, raise |
| M2D | Motorica (Alexanderson et al. 2023) | 4,825 | 269.36 | 0.289–1.000 | swing, lift, lean, bend, raise, step, bring, side, lower, follow |
| M2D | AIOZ (Le et al. 2023) | 74,649 | 262.65 | 0.311–1.000 | extend, shift, leave, weight, move, raise, bend, lift, progress, involve |
| **S2G** | BEAT2 (Liu et al. 2023) | 21,576 | 233.63 | 0.331–0.942 | emphasize, stand, move, shift, indicate, raise, extend, gesture, engage, weight |
| **HHI** | HumanSC3D (Fieraru et al. 2021a) | 1,024 | 279.05 | 0.339–0.651 | stand, bend, move, lower, raise, lean, extend, lift, hold, indicate |
| HHI | InterHuman (Liang et al. 2024b) | 23,072 | 285.82 | 0.206–0.756 | extend, leave, stand, step, move, shift, raise, lean, involve, bend |
| HHI | InterX (Xu et al. 2024b) | 10,460 | 219.28 | 0.282–0.693 | stand, extend, lean, raise, move, indicate, step, shift, bend, involve |
| **HOI** | ARCTIC (Fan et al. 2023) | 1,596 | 270.38 | 0.338–1.000 | extend, move, stand, focus, reach, adjust, indicate, hold, lean, simulate |
| HOI | TACO (Liu et al. 2024) | 3,391 | 279.06 | 0.295–0.733 | lift, skim, hold, smear, reach, scoop, control, grasp, stabilize, dip |
| HOI | FIT3D (Fieraru et al. 2021b) | 1,987 | 290.21 | 0.317–1.000 | stand, extend, bend, control, move, lower, focus, reach, return, lift |
| HOI | BEHAVE (Bhatnagar et al. 2022) | 3,089 | 309.72 | 0.294–0.577 | stand, bend, lean, leave, move, shift, extend, step, lift, control |
| HOI | CHAIRS (Jiang et al. 2022) | 8,476 | 282.23 | 0.325–0.599 | stand, lean, sit, bend, seat, extend, move, adjust, shift, lower |
| HOI | HOI-M³ (Zhang et al. 2024a) | 7,717 | 346.58 | 0.209–0.909 | leave, stand, lean, adjust, move, shift, walk, indicate, extend, seat |
| HOI | OakInk-v2 (Zhan et al. 2024) | 24,854 | 266.42 | 0.280–1.000 | focus, stand, extend, adjust, move, indicate, lean, involve, control, suggest |
| HOI | NeuralDome (Zhang et al. 2023a) | 2,333 | 360.82 | 0.240–0.599 | leave, extend, adjust, move, lift, stand, shift, stabilize, control, bend |
| **HSI** | EMDB (Kaufmann et al. 2023) | 237 | 311.78 | 0.320–0.542 | leave, walk, swing, extend, move, stand, lift, pkl, lean, step |
| HSI | RICH (Huang et al. 2022) | 598 | 307.25 | 0.313–0.566 | extend, stand, bend, move, lean, leave, lower, reach, focus, control |
| HSI | LAFAN (Harvey et al. 2020) | 3,297 | 316.25 | 0.301–1.000 | leave, extend, move, swing, step, lean, walk, stand, shift, lift |
| HSI | TRUMANS (Jiang et al. 2024b) | 8,231 | 323.08 | 0.299–0.667 | stand, extend, bend, lean, reach, move, straighten, leave, shift, control |
| HSI | CIRCLE (Araújo et al. 2023) | 10,284 | 278.17 | 0.298–0.917 | reach, extend, stand, lean, bend, shift, indicate, leave, focus, involve |
| **全部** | **OmniMoCap-X** | **321,967** | **276.78** | **0.139–1.000** | stand, lean, bend, lift, raise, adjust, step, swing, control, walk |

表 8 呈现了对我们 OmniMoCap-X 集合中文本质量的全面分析，该集合包含**超过 321,000 条**跨各类动作相关任务的文本描述。这一庞大数据集是通过收集与整理已有动作数据集、把动作渲染成视频、再借助视觉语言模型（VLMs）生成丰富文本描述而创建的。

我们数据集的语言多样性由 **TTR 范围**佐证——多个子集（AIOZ、Motorica、IDEA400 等）的最大值达到 **1.0**，表明极高的词汇丰富度。整个集合的平均句长为 **276.78 词**，展现了我们动作标注的描述深度，提供的是对细微动作的详尽记述，而非简单的动作标签。

**尤其值得注意的是不同动作类别中所捕捉到的动词多样性。** 数据集展现出与各动作类型语义性质相符的**任务特定动词分布**：
- **T2M** 数据集频繁出现**移动类动词**（"walk"、"step"）；
- **M2D** 集合强调**节奏性动作**（"swing"、"lift"）；
- **HOI** 数据集包含**操作类动词**（"lift"、"hold"、"grasp"）；
- **HHI** 数据集捕捉**人际动态**（"stand"、"extend"、"lean"）。

这种语义丰富性使在 OmniMoCap-X 上训练的模型能够跨多样上下文理解并生成精确的动作描述。

跨六个不同动作相关任务（T2M、M2D、S2G、HHI、HOI、HSI）的全面覆盖，使 OmniMoCap-X 在支持**可泛化的动作理解**方面处于独特位置。通过在统一标注框架下整合多种动作范式，我们的数据集超越了任务特定集合的局限，允许**跨动作领域的迁移学习**。这种文本质量与多样性显著增强了动作生成模型理解自然语言指令并产生相应类人动作的能力。

## B 数据集预处理

### B.1 格式转换

由于许多数据集并非 SMPL 系列格式，我们把所有数据集转换为 SMPL-X 格式以保持一致性。

对于 BVH 格式的数据集，例如 Mixamo（Adobe）、100Style（Mason, Starke, and Komura 2022）、Motorica（Alexanderson et al. 2023）与 LaFAN1（Harvey et al. 2020）：

1. 我们首先把所有 BVH 文件的**参考姿态标准化为 T-pose**，以确保一致的初始化。
2. 然后在 **Blender**（Blender Foundation）中把**根节点的坐标系**与 SMPL-X（Pavlakos et al. 2019）模型对齐，其中 **负 Y 轴定义为前向方向**，**Z 轴为竖直向上方向**。
3. 为适配 SMPL-X 的拓扑，我们基于预定义的 **SMPL-X T-pose 模板**生成对应的 BVH 骨架。我们在 **MotionBuilder**（Autodesk Inc.）中执行**骨架重定向（skeleton retargeting）**，把原始动画数据映射到 SMPL-X 层级结构上。
4. 最后，我们把重定向后的 BVH 文件从**欧拉角**转换为**轴角（axis-angle）**表征，并对关节旋转与平移在时间上应用**高斯滤波**做平滑，得到稳定的 SMPL-X 参数。

对于原本为 FBX 格式的 Choreomaster数据集（Chen et al. 2021），我们先用 Blender 转换为 BVH，然后使用相同流程处理。

### B.2 归一化

为降低模型训练难度，我们对 SMPL-X 动作序列执行**时间与空间归一化**。具体而言，我们按四个步骤标准化所有动作序列：

1. 首先，把每个序列的**初始帧对齐到面向正 Z 轴**。
2. 然后，把起始帧**重定位到一个统一位置，且双脚刚好接触地面**。
3. 接着，把各数据集的**帧率统一为 30 fps**。
4. 最后，把每个动作序列**切分为 5 秒的片段（150 帧）**。

## C 更多实现细节

### C.1 文本与动作特征提取器

现有研究通常依赖预训练的动作与文本特征提取器进行评估。**然而，由于我们的数据集在规模、分布与动作表征上与已有工作存在显著差异，这些预训练提取器无法直接使用。**

因此，遵循（Lu et al. 2024a）的做法，我们使用为本数据集定制的**对比学习框架**重新训练了动作与文本特征提取器。

- **文本特征提取器**基于 Transformer encoder 架构（Petrovich, Black, and Varol 2023），把原始文本编码为语义向量 $s_{t}$。
- **动作特征提取器**同样使用 Transformer encoder，把长达 **150 帧**的动作序列编码为语义向量 $s_{m}$。

两个编码器都包含额外的**语义 token**，结构类似于 ACTOR（Petrovich, Black, and Varol 2021）中的 encoder，但**不涉及概率分布建模**。

在实现上，文本编码器接受由**预训练且冻结的 DistilBERT** 网络提取的文本特征作为输入，而动作编码器直接处理原始动作序列数据。

在对比学习框架中，我们优化特征空间，使匹配的"文本-动作"特征对 $(s_{t},s_{m})$ 在嵌入空间中彼此靠近，同时确保不匹配的对被分隔至少距离 $d$。该优化目标通过以下对比损失函数实现：

$$D_{s_{t},s_{m}}=\|s_{t}-s_{m}\|_{2} \tag{3}$$

$$\mathcal{L}_{Cta}=(1-y)(D_{s_{t},s_{m}})^{2}+(y)\{\max(0,\ d-D_{s_{t},s_{m}})\}^{2} \tag{4}$$

其中 $y$ 是二值标签：若 $s_{t}$ 与 $s_{m}$ 来自匹配的"文本-动作"对则 $y=0$，否则 $y=1$。$m>0$ 是不匹配对的**间隔（margin）**，在我们的实验中设为 **10**。

### C.2 掩码机制（Mask）

我们的模型利用**四种专门的掩码机制**：

**（1）分量注意力掩码（Component attention mask）** —— 即 Transformer 中标准的 source key padding mask，用于**调控跨不同条件输入的注意力**。在我们基于 DiT 的架构中，输入条件可能部分缺失；该掩码把缺失输入的注意力权重**置零**，使模型只关注有效输入。

**（2）全局任务相关掩码（Global task-dependent mask）** —— 一种为动作预测、插值、补全与轨迹引导生成等任务定制的**时空动态掩码**，依据任务需求**区分已观测与未观测的动作区域**。

**（3）全局动作解耦掩码（Global motion disentanglement mask）** —— 通过选取**关键关节**（例如 $k$ 个关节）用于全局表征优化，把稠密的全局动作条件转化为**稀疏-稠密混合表征**；而其余 $n-k$ 个关节则贡献于局部动作表征与噪声注入。

**（4）动作表征重建掩码（Motion representation reconstruction mask）** —— 在训练期间**过滤掉没有真值标注的关节的重建误差**。例如，它会在缺少手部标注的数据集中抑制手部关节误差，有效解决**跨数据集关节数量不匹配**的问题。

## D 文本描述生成处理

为确保描述生成过程的质量与准确性，我们选择 **GPT-4o**（当前最先进的闭源多模态模型）作为主要的描述工具。

为最大化描述的准确性与全面性，我们对描述生成流水线实施了若干关键优化：

1. **首先**，我们显著**增加了提供给模型的输入帧数**，从而实现对视频内容更全面的时序理解。
2. **其次**，我们**提升了视频渲染质量**，以确保视觉细节被清晰保留并准确传达给模型。
3. **第三**，我们进行了大量的**提示词工程（prompt engineering）**，设计最优指令以引导模型生成更精确、更细致的描述，同时捕捉视觉元素与时间动态（代表性示例见图 4）。

这些技术改进共同带来了相比基线方法**大幅提升的描述质量**。

> **图 4**：GPT-4o 描述质量示例。

## E 数据集中的交互可视化

我们的数据集支持多种交互类型，包括**人-人交互（HHI）、人-物交互（HOI）与人-场景交互（HSI）**，如图 5 所示。

具体而言：
- **人-人交互**涵盖丰富的社交场景谱系，例如对话、握手、拥抱与协作活动。
- **人-物交互**包括日常活动，如抓握、操作与工具使用。
- **人-场景交互**捕捉个体与其环境之间的动态关系，例如坐在室内沙发上、双脚踩在地毯上，以及其他情境化的行为模式。

每种交互类型都伴随详细的**时空标注与语义描述**，为多模态全身动作生成提供全面的训练样本。

> **图 5**：从左到右分别为我们数据集中的 HHI、HOI 与 HSI。

---

## 参考文献

- Achiam, J.; Adler, S.; Agarwal, S.; et al. 2023. *GPT-4 Technical Report.* arXiv:2303.08774.
- Adobe. *Mixamo.* https://www.mixamo.com
- Alexanderson, S.; Nagy, R.; Beskow, J.; and Henter, G. E. 2023. *Listen, Denoise, Action! Audio-Driven Motion Synthesis with Diffusion Models.* ACM Trans. Graph., 42(4): 44:1–44:20.
- Ao, T. 2024. *Body of Her: A Preliminary Study on End-to-End Humanoid Agent.* arXiv:2408.02879.
- Araújo, J. P.; Li, J.; Vetrivel, K.; Agarwal, R.; Wu, J.; Gopinath, D.; Clegg, A. W.; and Liu, K. 2023. *CIRCLE: Capture in Rich Contextual Environments.* CVPR, 21211–21221.
- Autodesk Inc. *Autodesk MotionBuilder.*
- Bhatnagar, B. L.; Xie, X.; Petrov, I. A.; Sminchisescu, C.; Theobalt, C.; and Pons-Moll, G. 2022. *BEHAVE: Dataset and Method for Tracking Human Object Interactions.* CVPR, 15935–15946.
- **Bian, Y.; Zeng, A.; Ju, X.; Liu, X.; Zhang, Z.; Liu, W.; and Xu, Q. 2024. *MotionCraft: Crafting Whole-Body Motion with Plug-and-Play Multimodal Controls.* arXiv:2407.21136.**
- Blender Foundation. *Blender.*
- Chen, J.; Liu, Y.; Wang, J.; Zeng, A.; Li, Y.; and Chen, Q. 2024a. *DiffSHEG: A Diffusion-Based Approach for Real-Time Speech-Driven Holistic 3D Expression and Gesture Generation.* CVPR, 7352–7361.
- Chen, K.; Tan, Z.; Lei, J.; Zhang, S.-H.; Guo, Y.-C.; Zhang, W.; and Hu, S.-M. 2021. *Choreomaster: Choreography-Oriented Music-Driven Dance Synthesis.* ACM TOG, 40(4): 1–13.
- Chen, L.-H.; Zhang, J.; Li, Y.; Pang, Y.; Xia, X.; and Liu, T. 2023a. *HumanMAC: Masked Motion Completion for Human Motion Prediction.* ICCV, 9544–9555.
- Chen, R.; Shi, M.; Huang, S.; Tan, P.; Komura, T.; and Chen, X. 2024b. *Taming Diffusion Probabilistic Models for Character Control.* ACM SIGGRAPH 2024 Conference Papers, 1–10.
- Chen, X.; Jiang, B.; Liu, W.; Huang, Z.; Fu, B.; Chen, T.; and Yu, G. 2023b. *Executing Your Commands via Motion Diffusion in Latent Space.* CVPR, 18000–18010.
- Cohan, S.; Tevet, G.; Reda, D.; Peng, X. B.; and van de Panne, M. 2024. *Flexible Motion In-Betweening with Diffusion Models.* ACM SIGGRAPH, 1–9.
- Dai, W.; Chen, L.-H.; Wang, J.; Liu, J.; Dai, B.; and Tang, Y. 2024. *MotionLCM: Real-Time Controllable Motion Generation via Latent Consistency Model.* ECCV.
- Delmas, G.; Weinzaepfel, P.; Lucas, T.; Moreno-Noguer, F.; and Rogez, G. 2022. *PoseScript: 3D Human Poses from Natural Language.* ECCV, 346–362.
- Fan, Z.; Taheri, O.; Tzionas, D.; Kocabas, M.; Kaufmann, M.; Black, M. J.; and Hilliges, O. 2023. *ARCTIC: A Dataset for Dexterous Bimanual Hand-Object Manipulation.* CVPR, 12943–12954.
- Feng, A.; Shin, S.; and Yoon, Y. 2022. *A Tool for Extracting 3D Avatar-Ready Gesture Animations from Monocular Videos.* ACM SIGGRAPH MIG, 1–7.
- Fieraru, M.; Zanfir, M.; Oneata, E.; Popa, A.-I.; Olaru, V.; and Sminchisescu, C. 2021a. *Learning Complex 3D Human Self-Contact.* AAAI, 35: 1343–1351.
- Fieraru, M.; Zanfir, M.; Pirlea, S.-C.; Olaru, V.; and Sminchisescu, C. 2021b. *AIFit: Automatic 3D Human-Interpretable Feedback Models for Fitness Training.* CVPR.
- Guo, C.; Mu, Y.; Javed, M. G.; Wang, S.; and Cheng, L. 2024a. *MoMask: Generative Masked Modeling of 3D Human Motions.* CVPR, 1900–1910.
- Guo, C.; Zou, S.; Zuo, X.; Wang, S.; Ji, W.; Li, X.; and Cheng, L. 2022. *Generating Diverse and Natural 3D Human Motions from Text.* CVPR, 5152–5161.
- Guo, X.; Zhang, M.; Xie, H.; Gu, C.; and Liu, Z. 2024b. *CrowdMoGen: Zero-Shot Text-Driven Collective Motion Generation.* arXiv:2407.06188.
- Han, B.; Peng, H.; Dong, M.; Ren, Y.; Shen, Y.; and Xu, C. 2024. *AMD: Autoregressive Motion Diffusion.* AAAI, 2022–2030.
- Harvey, F. G.; Yurick, M.; Nowrouzezahrai, D.; and Pal, C. 2020. *Robust Motion In-Betweening.* ACM TOG, 39(4).
- Huang, C.-H. P.; Yi, H.; Höschle, M.; Safroshkin, M.; Alexiadis, T.; Polikovsky, S.; Scharstein, D.; and Black, M. J. 2022. *Capturing and Inferring Dense Full-Body Human-Scene Contact.* CVPR, 13274–13285.
- Ionescu, C.; Papava, D.; Olaru, V.; and Sminchisescu, C. 2013. *Human3.6M: Large Scale Datasets and Predictive Methods for 3D Human Sensing in Natural Environments.* IEEE TPAMI, 36(7): 1325–1339.
- Ji, Y.; Xu, F.; Yang, Y.; Shen, F.; Shen, H. T.; and Zheng, W.-S. 2018. *A Large-Scale RGB-D Database for Arbitrary-View Human Action Recognition.* ACM MM, 1510–1518.
- Jiang, B.; Chen, X.; Liu, W.; Yu, J.; Yu, G.; and Chen, T. 2024a. *MotionGPT: Human Motion as a Foreign Language.* NeurIPS.
- Jiang, N.; Liu, T.; Cao, Z.; Cui, J.; Chen, Y.; Wang, H.; Zhu, Y.; and Huang, S. 2022. *Full-Body Articulated Human-Object Interaction.* ICCV.
- Jiang, N.; Zhang, Z.; Li, H.; Ma, X.; Wang, Z.; Chen, Y.; Liu, T.; Zhu, Y.; and Huang, S. 2024b. *Scaling Up Dynamic Human-Scene Interaction Modeling.* CVPR, 1737–1747.
- Kaufmann, M.; Song, J.; Guo, C.; Shen, K.; Jiang, T.; Tang, C.; Zárate, J. J.; and Hilliges, O. 2023. *EMDB: The Electromagnetic Database of Global 3D Human Pose and Shape in the Wild.* ICCV.
- Lauterbach, C.; Garland, M.; Sengupta, S.; Luebke, D.; and Manocha, D. 2009. *Fast BVH Construction on GPUs.* Computer Graphics Forum, 28: 375–384.
- Le, N.; Pham, T.; Do, T.; Tjiputra, E.; Tran, Q. D.; and Nguyen, A. 2023. *Music-Driven Group Choreography.* CVPR, 8673–8682.
- Li, B.; Zhao, Y.; Zhelun, S.; and Sheng, L. 2022. *DanceFormer: Music Conditioned 3D Dance Generation with Parametric Motion Transformer.* AAAI, 36: 1272–1279.
- Li, J.; Wu, J.; and Liu, C. K. 2023. *Object Motion Guided Human Motion Synthesis.* ACM TOG, 42(6): 1–11.
- Li, R.; Yang, S.; Ross, D. A.; and Kanazawa, A. 2021. *Learn to Dance with AIST++: Music Conditioned 3D Dance Generation.* arXiv:2101.08779.
- Li, R.; Zhao, J.; Zhang, Y.; Su, M.; Ren, Z.; Zhang, H.; Tang, Y.; and Li, X. 2023. *FineDance: A Fine-Grained Choreography Dataset for 3D Full Body Dance Generation.* ICCV, 10234–10243.
- Li, T.; Bolkart, T.; Black, M. J.; Li, H.; and Romero, J. 2017. *Learning a Model of Facial Shape and Expression from 4D Scans.* ACM Trans. Graph., 36(6): 194-1.
- Liang, H.; Bao, J.; Zhang, R.; Ren, S.; Xu, Y.; Yang, S.; Chen, X.; Yu, J.; and Xu, L. 2024a. *OMG: Towards Open-Vocabulary Motion Generation via Mixture of Controllers.* CVPR, 482–493.
- Liang, H.; Zhang, W.; Li, W.; Yu, J.; and Xu, L. 2024b. *InterGen: Diffusion-Based Multi-Human Motion Generation under Complex Interactions.* IJCV, 1–21.
- Liao, Y.; Fu, Y.; Cheng, Z.; and Wang, J. 2024. *AnimationGPT: An AIGC Tool for Generating Game Combat Motion Assets.* https://github.com/fyyakaxyy/AnimationGPT
- Lin, G.; Jiang, J.; Yang, J.; Zheng, Z.; and Liang, C. 2025. *OmniHuman-1: Rethinking the Scaling-Up of One-Stage Conditioned Human Animation Models.* arXiv:2502.01061.
- Lin, J.; Zeng, A.; Lu, S.; Cai, Y.; Zhang, R.; Wang, H.; and Zhang, L. 2024. *Motion-X: A Large-Scale 3D Expressive Whole-Body Human Motion Dataset.* NeurIPS.
- Ling, Z.; Han, B.; Li, S.; Shen, H.; Cheng, J.; and Zou, C. 2024. *MotionLLaMA: A Unified Framework for Motion Synthesis and Comprehension.* arXiv:2411.17335.
- Ling, Z.; Han, B.; Wong, Y.; Kangkanhalli, M.; and Geng, W. 2023. *MCM: Multi-Condition Motion Synthesis Framework for Multi-Scenario.* arXiv:2309.03031.
- Liu, H.; Zhu, Z.; Becherini, G.; Peng, Y.; Su, M.; Zhou, Y.; Zhe, X.; Iwamoto, N.; Zheng, B.; and Black, M. J. 2023. *EMAGE: Towards Unified Holistic Co-Speech Gesture Generation via Masked Audio Gesture Modeling.* arXiv:2401.
- Liu, H.; Zhu, Z.; Iwamoto, N.; Peng, Y.; Li, Z.; Zhou, Y.; Bozkurt, E.; and Zheng, B. 2022. *BEAT: A Large-Scale Semantic and Emotional Multi-Modal Dataset for Conversational Gestures Synthesis.* ECCV, 612–630.
- Liu, Y.; Yang, H.; Si, X.; Liu, L.; Li, Z.; Zhang, Y.; Liu, Y.; and Yi, L. 2024. *TACO: Benchmarking Generalizable Bimanual Tool-Action-Object Understanding.* CVPR, 21740–21751.
- Loper, M.; Mahmood, N.; Romero, J.; Pons-Moll, G.; and Black, M. J. 2023. *SMPL: A Skinned Multi-Person Linear Model.* Seminal Graphics Papers: Pushing the Boundaries, Vol. 2, 851–866.
- Lu, S.; Chen, L.-H.; Zeng, A.; Lin, J.; Zhang, R.; Zhang, L.; and Shum, H.-Y. 2024a. *HumanTOMATO: Text-Aligned Whole-Body Motion Generation.* ICML.
- Lu, S.; Wang, J.; Lu, Z.; Chen, L.-H.; Dai, W.; Dong, J.; Dou, Z.; Dai, B.; and Zhang, R. 2024b. *ScaMo: Exploring the Scaling Law in Autoregressive Motion Generation Model.* arXiv:2412.14559.
- Luo, M.; Hou, R.; Li, Z.; Chang, H.; Liu, Z.; Wang, Y.; and Shan, S. 2024. *M³-GPT: An Advanced Multimodal, Multitask Framework for Motion Comprehension and Generation.* arXiv:2405.16273.
- Mao, J.; Zhao, S.; Song, S.; Shi, T.; Ye, J.; Zhang, M.; Geng, H.; Malik, J.; Guizilini, V.; and Wang, Y. 2024. *Learning from Massive Human Videos for Universal Humanoid Pose Control.* arXiv:2412.14172.
- Mason, I.; Starke, S.; and Komura, T. 2022. *Real-Time Style Modelling of Human Locomotion via Feature-Wise Transformations and Local Motion Phases.* Proc. ACM Comput. Graph. Interact. Tech., 5(1): 1–18.
- McFee, B.; Raffel, C.; Liang, D.; Ellis, D. P.; McVicar, M.; Battenberg, E.; and Nieto, O. 2015. *librosa: Audio and Music Signal Analysis in Python.* SciPy, 18–24.
- Pavlakos, G.; Choutas, V.; Ghorbani, N.; Bolkart, T.; Osman, A. A.; Tzionas, D.; and Black, M. J. 2019. *Expressive Body Capture: 3D Hands, Face, and Body from a Single Image.* CVPR, 10975–10985.
- Peebles, W.; and Xie, S. 2023. *Scalable Diffusion Models with Transformers.* ICCV, 4195–4205.
- Petrovich, M.; Black, M. J.; and Varol, G. 2021. *Action-Conditioned 3D Human Motion Synthesis with Transformer VAE.* ICCV, 10985–10995.
- Petrovich, M.; Black, M. J.; and Varol, G. 2023. *TMR: Text-to-Motion Retrieval Using Contrastive 3D Human Motion Synthesis.* ICCV, 9488–9497.
- Plappert, M.; Mandery, C.; and Asfour, T. 2016. *The KIT Motion-Language Dataset.* Big Data, 4(4): 236–252.
- Punnakkal, A. R.; Chandrasekaran, A.; Athanasiou, N.; Quiros-Ramirez, A.; and Black, M. J. 2021. *BABEL: Bodies, Action and Behavior with English Labels.* CVPR, 722–731.
- Radford, A.; Kim, J. W.; Hallacy, C.; et al. 2021. *Learning Transferable Visual Models from Natural Language Supervision.* ICML, 8748–8763.
- Raffel, C.; Shazeer, N.; Roberts, A.; Lee, K.; Narang, S.; Matena, M.; Zhou, Y.; Li, W.; and Liu, P. J. 2020. *Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer.* JMLR, 21(140): 1–67.
- Siyao, L.; Yu, W.; Gu, T.; Lin, C.; Wang, Q.; Qian, C.; Loy, C. C.; and Liu, Z. 2022. *Bailando: 3D Dance Generation by Actor-Critic GPT with Choreographic Memory.* CVPR, 11050–11059.
- Tevet, G.; Raab, S.; Gordon, B.; Shafir, Y.; Cohen-Or, D.; and Bermano, A. H. 2022. *Human Motion Diffusion Model.* ICLR.
- Tseng, J.; Castellon, R.; and Liu, K. 2023. *EDGE: Editable Dance Generation from Music.* CVPR, 448–458.
- Valle-Pérez, G.; Henter, G. E.; Beskow, J.; Holzapfel, A.; Oudeyer, P.-Y.; and Alexanderson, S. 2021. *Transflower: Probabilistic Autoregressive Dance Generation with Multimodal Attention.* ACM Trans. Graph., 40(6): 195:1–195:14.
- Vaswani, A.; Shazeer, N.; Parmar, N.; Uszkoreit, J.; Jones, L.; Gomez, A. N.; Kaiser, L.; and Polosukhin, I. 2017. *Attention Is All You Need.* NeurIPS.
- Wikipedia contributors. 2024. *FBX.* Wikipedia, The Free Encyclopedia.
- Xie, Y.; Jampani, V.; Zhong, L.; Sun, D.; and Jiang, H. 2024. *OmniControl: Control Any Joint at Any Time for Human Motion Generation.* ICLR.
- Xu, G.; Tao, J.; Li, W.; and Duan, L. 2024a. *Learning Semantic Latent Directions for Accurate and Controllable Human Motion Prediction.* ECCV, 56–73.
- Xu, L.; Lv, X.; Yan, Y.; Jin, X.; Wu, S.; Xu, C.; Liu, Y.; Zhou, Y.; Rao, F.; Sheng, X.; et al. 2024b. *Inter-X: Towards Versatile Human-Human Interaction Analysis.* CVPR, 22260–22271.
- Yazdian, P. J.; Liu, E.; Lagasse, R.; Mohammadi, H.; Cheng, L.; and Lim, A. 2023. *MotionScript: Natural Language Descriptions for Expressive 3D Human Motions.* arXiv:2312.12634.
- Yi, H.; Liang, H.; Liu, Y.; Cao, Q.; Wen, Y.; Bolkart, T.; Tao, D.; and Black, M. J. 2023. *Generating Holistic 3D Human Motion from Speech.* CVPR, 469–480.
- Zhan, X.; Yang, L.; Zhao, Y.; Mao, K.; Xu, H.; Lin, Z.; Li, K.; and Lu, C. 2024. *OakInk2: A Dataset of Bimanual Hands-Object Manipulation in Complex Task Completion.* CVPR, 445–456.
- Zhang, J.; Luo, H.; Yang, H.; Xu, X.; Wu, Q.; Shi, Y.; Yu, J.; Xu, L.; and Wang, J. 2023a. *NeuralDome: A Neural Modeling Pipeline on Multi-View Human-Object Interactions.* CVPR.
- Zhang, J.; Zhang, J.; Song, Z.; Shi, Z.; Zhao, C.; Shi, Y.; Yu, J.; Xu, L.; and Wang, J. 2024a. *HOI-M³: Capture Multiple Humans and Objects Interaction within Contextual Environment.* CVPR, 516–526.
- Zhang, J.; Zhang, Y.; Cun, X.; Zhang, Y.; Zhao, H.; Lu, H.; Shen, X.; and Shan, Y. 2023b. *Generating Human Motion from Textual Descriptions with Discrete Representations.* CVPR, 14730–14740.
- Zhang, M.; Cai, Z.; Pan, L.; Hong, F.; Guo, X.; Yang, L.; and Liu, Z. 2024b. *MotionDiffuse: Text-Driven Human Motion Generation with Diffusion Model.* IEEE TPAMI.
- Zhang, M.; Jin, D.; Gu, C.; Hong, F.; Cai, Z.; Huang, J.; Zhang, C.; Guo, X.; Yang, L.; He, Y.; et al. 2024c. *Large Motion Model for Unified Multi-Modal Motion Generation.* arXiv:2404.01284.
- Zhang, Y.; Huang, D.; Liu, B.; Tang, S.; Lu, Y.; Chen, L.; Bai, L.; Chu, Q.; Yu, N.; and Ouyang, W. 2024d. *MotionGPT: Finetuned LLMs Are General-Purpose Motion Generators.* AAAI, 7368–7376.
- Zhao, K.; Li, G.; and Tang, S. 2024. *DART: A Diffusion-Based Autoregressive Motion Model for Real-Time Text-Driven Motion Control.* arXiv:2410.05260.

---

*原文由 LaTeXML 生成于 2025-10-22。本译文为完整中文翻译（含正文、全部 8 个表格、全部补充材料与参考文献），译于 2026-08-10。*
