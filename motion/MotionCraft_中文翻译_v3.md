# MotionCraft：以即插即用的多模态控制精雕全身人体动作

**MotionCraft: Crafting Whole-Body Motion with Plug-and-Play Multimodal Controls**

> **作者**：Yuxuan Bian¹（边宇轩）、Ailing Zeng²、Xuan Ju¹、Xian Liu¹、Zhaoyang Zhang¹、Wei Liu²、Qiang Xu¹\*
> ¹ 香港中文大学（中国香港）　² 腾讯 AI Lab
> \* 通讯作者
>
> **arXiv**：2407.21136v3 [cs.CV]　|　v1 提交于 2024-07-30，本版本（v3）修订于 2024-08-25
> **DOI**：https://doi.org/10.48550/arXiv.2407.21136

---

## 译者说明

1. **关于版本**：用户提到「v3 比 v2 多3.6MB 内容」。经比对，v1 约 4.88MB、v2 约 4.91MB、v3 约 8.49MB。新增体量主要来自 **完整附录（Appendix）**：可视化结果说明、扩充版相关工作、MC-Bench 构建细节（动作表征定义、三个子集的构建流程）、MotionCraft 与检索式评估模型的实现细节、原始 HumanML3D 基准上的补充结果（表 5）、第二阶段训练策略与时序建模范式的额外消融（表 6），以及「更广泛影响与局限」章节。本译文已**完整覆盖正文 + 全部附录**。
2. **关于论文标题**：v1/v2 阶段的标题为 *Adding Multimodal Controls to Whole-body Human Motion Generation*（向全身人体动作生成中加入多模态控制），v3 已正式改名为 **MotionCraft**。方法名也从早期的ControlMM 系列命名统一为 MotionCraft / MC-Attn / MC-Bench。
3. **术语处理**：核心术语首次出现时给出中英对照，后续沿用中文或缩写（T2M / S2G / M2D 等）。数学符号、指标名（FID、Div、R-Precision 等）保留原文。

---

## 摘要

由文本、语音或音乐控制的**全身多模态动作生成**（whole-body multimodal motion generation）有着大量应用，包括视频生成与角色动画。然而，若想用一个**统一模型**去完成条件模态各异的多种生成任务，会面临两大主要挑战：

- **动作分布漂移**（motion distribution drifts）：不同任务之间的动作分布差异显著（例如「伴随语音的手势」与「文本驱动的日常动作」）；
- **混合条件的复杂优化**：不同粒度的条件（例如文本与音频）混合建模时优化困难。

此外，不同任务与数据集之间**动作格式不一致**，也阻碍了面向多模态动作生成的有效训练。

本文提出 **MotionCraft**——一个统一的扩散 Transformer（diffusion transformer），能够以即插即用（plug-and-play）的多模态控制来精雕全身动作。我们的框架采用**由粗到细（coarse-to-fine）的训练策略**：第一阶段进行「文本到动作」的语义预训练，第二阶段进行多模态低层级控制适配，以处理不同粒度的条件。为了在不同分布间有效学习并迁移动作知识，我们设计了 **MC-Attn**，用于并行建模**静态**与**动态**的人体拓扑图。为克服现有基准动作格式不统一的问题，我们提出 **MC-Bench**——首个基于统一 SMPL-X 格式的、可公开获取的多模态全身动作生成基准。大量实验表明，MotionCraft 在多种标准动作生成任务上达到了当前最优（state-of-the-art）性能。

> **图 1**：我们提出 MotionCraft，一个以即插即用多模态控制精雕全身动作的扩散 Transformer，涵盖了包括「文本到动作」（Text-to-Motion）、「语音到手势」（Speech-to-Gesture）与「音乐到舞蹈」（Music-to-Dance）在内的强健动作生成能力。

---

## 1 引言

具备多模态控制的全身人体动作生成（Zhang et al. 2024b；Liu et al. 2024a；Li et al. 2023）——即根据多模态条件产生自然、连贯的人体运动——有着大量应用，包括人体视频生成（Hu 2024）与角色动画（Zhang et al. 2023a）。

近期在**单条件**人体动作生成上的进展，已使我们能够从各种不同粒度的控制信号生成逼真的人体运动，包括文本描述（Guo et al. 2022；Zhang et al. 2023c）、音乐片段（Siyao et al. 2022；Li et al. 2023）以及语音片段（Liu et al. 2024a；Chen et al. 2024）。然而，若要在**统一模型**内把这些能力扩展到「多模态控制的全身动作生成」，会引入若干重大挑战：

**➠ 动作分布漂移（Motion distribution drifts）**
在不同条件下，动作分布往往差异显著（Zhang et al. 2024b；Ling et al. 2023）。在**文本到动作**（T2M）中，语义化的文本引导主要控制日常的躯干运动（Guo et al. 2022；Lin et al. 2023a）；而**语音到手势**（S2G）则在第一视角音频下聚焦于手势与面部表情（Liu et al. 2024a；Yi et al. 2023）；**音乐到舞蹈**（M2D）则在第三视角音乐与肢体运动之间包含更加动态、更加多变的关联（Li et al. 2023）。以往研究通常只专注于单一任务，以规避分布漂移所带来的弱生成可迁移性。

**➠ 混合条件下的优化挑战（Optimization challenges under mixed conditions）**
当前的多模态动作生成工作，会把各类控制信号——例如语义化的文本引导、第一人称语音、第三人称音乐——压缩进一个**公共隐空间**做混合建模，包括 Transformer 的 token embedding（Zhou, Wan, and Wang 2023）以及 ImageBind（Girdhar et al. 2023）所使用的特征空间。然而这种做法常常导致跨模态的对齐问题，并在**同时**学习不同粒度层级的条件时引入优化难题（Team et al. 2023）。

**➠ 全身动作格式与评测不统一（Non-uniform whole-body motion format and evaluation）**
最后，目前还没有具备统一动作表征与统一评测流水线的、高质量的多模态全身人体动作生成基准。

在本工作中，我们提出统一的动作扩散 Transformer **MotionCraft**，以即插即用的多模态控制精雕全身动作，生成与文本、语音（音乐）细粒度对齐的动作。它同时也支持在**多个条件并存**的情况下生成动作，例如文本 + 语音，或文本 + 音乐。

为了有效学习不同粒度的条件，MotionCraft 采用**两阶段、由粗到细**的多模态生成框架。第一阶段，在粗粒度文本引导下习得高层语义的动作生成能力；第二阶段，在第一阶段已**冻结**的主干网络上追加控制分支，使模型既保留语义生成能力，又能对特定低层级条件（语音或音乐）实现细粒度的即插即用控制，同时避免混合训练带来的优化混乱。

为应对不同生成场景之间的动作分布漂移，我们借助 t-SNE（t-distributed stochastic neighbor embedding）分析了人体动作的运动学与分布特征。我们发现：对应不同控制信号的动作分布，可以被**分解**为「静态人体拓扑结构」与「动态拓扑关系」，且二者在不同场景之间具有可泛化性。与现有的大型语言与视觉模型不同，动作数据的规模依然非常小，且难以规模化扩张。为建模这些以人为中心的时空属性，我们设计了 **MC-Attn**：其**空间分支**通过并行建模静态与动态人体拓扑图，在不同分布之间学习并迁移动作拓扑知识；其**时间分支**则捕捉动作序列内部的时序关系。

为克服现有基准中动作格式不一致的限制——例如 Rot6D（Guo et al. 2022）、SMPL（Loper et al. 2015）、SMPL-X（Pavlakos et al. 2019）——我们还提出 **MC-Bench**：首个基于统一全身 SMPL-X 格式的、可公开获取的多模态动作生成基准，包含数据构建与评测流水线。大量实验表明，MotionCraft 在包括文本到动作、语音到手势、音乐到舞蹈等多种标准动作生成任务上取得了有竞争力的性能。此外，我们还提供了全面的消融研究，为未来的多模态全身动作生成模型在模型设计与规模化效应方面提供洞见。

**本文贡献总结如下：**

- 我们提出 **MotionCraft**，一个两阶段、由粗到细的多模态动作生成框架，支持不同粒度的控制信号，实现高效的即插即用多模态动作生成。
- 我们设计 **MC-Attn**，这是在多模态动作生成中**首次尝试**通过建模静态与动态人体拓扑来对抗动作分布漂移。
- 我们创建 **MC-Bench**，首个公开可用、且采用统一全身动作表征 SMPL-X 的多模态全身动作生成基准。

### 表 1：MotionCraft 与以往动作生成方法的对比

MotionCraft 联合建模**静态人体骨架结构**与**动态人体拓扑关系**，从而在各类全身生成场景之间实现灵活的动作知识迁移，并支持与任意新控制信号模态的即插即用。

| 模型 | Text2Motion | Music2Dance | Speech2Gesture | 静态身体先验 | 动态身体自适应 | 全身 | 统一表征 | 即插即用 |
|---|---|---|---|---|---|---|---|---|
| FineMoGen (Zhang et al. 2023c) | ✓ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ |
| HumanTomato (Lu et al. 2023) | ✓ | ✗ | ✗ | ✓ | ✗ | ✓ | ✗ | ✗ |
| FineDance (Li et al. 2023) | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Bailando (Siyao et al. 2022) | ✗ | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ |
| EMAGE (Liu et al. 2024a) | ✗ | ✗ | ✓ | ✓ | ✗ | ✓ | ✗ | ✗ |
| TalkShow (Yi et al. 2023) | ✗ | ✗ | ✓ | ✓ | ✗ | ✓ | ✗ | ✗ |
| MCM (Ling et al. 2023) | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ |
| Motion-Verse (Zhang et al. 2024b) | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ | ✗ | ✗ |
| **MotionCraft** | **✓** | **✓** | **✓** | **✓** | **✓** | **✓** | **✓** | **✓** |

---

## 2 相关工作

### 2.1 人体动作生成模型

条件化的人体动作生成模型已取得显著进展，包括文本到动作（T2M）（Tevet et al. 2023；Zhang et al. 2023b；Liu et al. 2023；Zhang et al. 2024a, 2023c；Liang et al. 2024）、语音到手势（S2G）（Yi et al. 2023；Chen et al. 2024；Liu et al. 2022b）以及音乐到舞蹈（M2D）（Li et al. 2023；Tseng, Castellon, and Liu 2023；Siyao et al. 2022）。近来，多模态动作生成受到越来越多关注（Ling et al. 2023；Zhang et al. 2024b；Luo et al. 2024）。

- **M³-GPT**（Luo et al. 2024）把量化后的条件 token 注入大语言模型的词表，以实现动作理解与生成，但它**忽视了人体拓扑先验**的建模。
- **Motion-Verse**（Zhang et al. 2024b）引入动态注意力来评估各身体部位之间的关系，但未能捕捉**整体的静态人体拓扑**，导致泛化能力受限、优化复杂度上升。此外，它基于 ImageBind（Girdhar et al. 2023）对所有条件做混合训练，这在同时学习不同粒度条件时带来优化困难，且遇到新控制信号需要重新训练。
- **MCM**（Ling et al. 2023）尝试基于 ControlNet（Zhang, Rao, and Agrawala 2023）架构来解决混合训练的优化混乱问题，但它**完全没有建模人体拓扑结构**，导致跨生成场景的泛化能力较差。

如表 1 所示，与以往方法相比，MotionCraft 通过 MC-Attn 捕捉静态人体拓扑与领域特定的动态骨架关系、引入控制分支、并采用由粗到细的训练策略，从而在不同控制信号下生成全身动作，并具备即插即用能力。

### 2.2 人体动作生成基准

近年来已构建了多种条件化人体动作生成基准。

- **T2M**：研究者构建了涵盖动作类别（Chung et al. 2021；Trivedi, Thatipelli, and Sarvadevabhatla 2021）、序列化动作标签（Zhang et al. 2022；Guo et al. 2020）以及任意自然语言描述（Lin et al. 2023a；Guo et al. 2022；Tang et al. 2023）的数据集。
- **M2D**：AIST++（Li et al. 2021）从视频中重建了 5 小时的 SMPL（Loper et al. 2015）格式舞蹈数据。FineDance（Li et al. 2023）收集了跨 22 种风格、共 14.6 小时的舞蹈，并以 SMPL-H（Pavlakos et al. 2019）格式补充了细致的手部动作。
- **S2G**：在相关数据集（Liu et al. 2024a, 2022a；Yi et al. 2023）中，BEAT2（Liu et al. 2024a）与 BEAT（Liu et al. 2022a）因动作多样、数据量庞大，已成为最受欢迎的基准。BEAT2 在 BEAT 基础上利用 SMPL-X 与 FLAME（Kim, Kim, and Choi 2023），实现了更高质量的统一网格级（mesh-level）数据。

尽管有上述进展，目前**仍没有任何公开可用的基准**支持多模态全身动作生成的统一表征。

---

## 3 动机（Motivation）

实现「多模态控制下的全身人体动作生成」的关键挑战在于：应对不同生成场景之间的动作分布漂移（Zhang et al. 2024b），以及高效学习不同粒度的控制信号（Ling et al. 2023）。

### 3.1 动作分布漂移的解法

当前动作生成模型主要专注于**单条件**场景，因为它们难以处理不同场景之间明显的动作分布漂移（Zhou, Wan, and Wang 2023）。例如，如图 2 所示：T2M 主要涉及日常躯干运动；S2G 包含复杂的手部手势、丰富的面部表情，而下肢几乎静止；M2D 则强调多变而幅度大的肢体运动，但手部动作有限。

然而，许多以人为中心的研究（Zeng et al. 2021；Ma, Bai, and Zhou 2022）已证实：将人体骨架拓扑表示为一个**有向加权图**、以不同身体部位为顶点，可在复杂动作建模中引入运动学先验，从而提升在分布偏移下的泛化能力。此外，依据人体运动学（Loper et al. 2015；Pavlakos et al. 2019），把人体骨架**分解为静态与动态拓扑的组合**是很自然的。

举例来说：在任何场景中，根顶点（髋部）总是会显著影响其子顶点（下肢或上臂），并且成对的手臂之间存在对称交互；然而在 S2G 中，四肢与其他身体部位之间的相关性会**减弱**，而手部与面部表情之间的连接权重会**增强**。

因此，同时建模动态与静态拓扑图，能够在数据有限且分布漂移显著的情况下，高效地把动作知识泛化到不同生成任务上。

> **图 2**：不同生成任务中动作的 t-SNE 隐空间。图中展示了跨生成场景的动作分布漂移。

### 3.2 不同粒度条件的高效学习

不同动作生成场景对应着不同粒度的条件。例如，文本引导通常提供**序列级、粗粒度**的语义控制，而语音与音乐更侧重**逐帧的低层级**控制（Liu et al. 2024a；Li et al. 2023）。在单一空间内混合学习所有条件，会带来不可避免的模态对齐损失，且无法解耦各粒度的学习过程（Zhang, Rao, and Agrawala 2023；Ling et al. 2023），造成优化混乱。

受图像/视频领域其他视觉生成范式的启发——包括 StableDiffusion（Rombach et al. 2022）与 Sora（Liu et al. 2024b）——把不同条件下的生成**解耦**、并以 T2M 作为基础预训练任务，可以为后续多条件生成构建稳健的生成能力，从而实现更高效、更细粒度的多模态控制生成。

> **图 3**：MotionCraft 的架构。MotionCraft 是基于 Transformer 的扩散模型。第一阶段，MotionCraft 以文本作为语义控制引导，在多个数据集上学习粗粒度的跨场景动作知识；第二阶段，MotionCraft 冻结主干网络，同时加入即插即用的控制分支，以学习不同的低层级控制信号。MotionCraft 的核心是 MC-Attn，它通过捕捉静态与动态人体拓扑图的空间属性、并并行学习时序关系，来优化动作 token 序列的表征。

---

## 4 方法

### 4.1 MotionCraft 框架

MotionCraft 的总览见图 3。为了解耦不同粒度下的条件生成学习，我们采用**双分支架构**：一个主「文本到动作」分支，以及一个即插即用的低层级控制分支；并配合**两阶段由粗到细**的训练策略，以高效掌握跨场景、跨控制信号模态的动作拓扑知识。两个分支都使用专门设计了 MC-Attn 的动作扩散 Transformer，以捕捉静态与动态的动作拓扑属性。

#### 阶段 ①：文本到动作语义预训练

主分支 $f_m(\cdot)$ 在第一阶段（文本到动作语义预训练）中被优化，使用从 MC-Bench 中多样场景收集的「文本-动作」配对数据。

我们选择**文本**作为各个单模态数据集之间的共享条件，使 MotionCraft 得以在文本 $\mathbf{H}_{text}\in\mathbb{R}^{B\times F_t\times D_t}$ 与动作 $\mathbf{H}_{motion}\in\mathbb{R}^{B\times F_m\times D_m}$ 之间习得序列级生成能力与粗粒度的文本引导跟随能力。

总体而言，在多样生成场景下的文本引导预训练，有助于在第二阶段跟随其他低层级条件的细粒度控制。

#### 阶段 ②：多模态低层级控制适配

在低层级控制适配的微调阶段，我们的目标是建模各类条件信号 $\mathbf{H}_{c}\in\mathbb{R}^{B\times T_c\times D_c}$ 与动作序列 $\mathbf{H}_{motion}\in\mathbb{R}^{B\times F_m\times D_m}$ 之间的关联。

- 主分支所有参数 $f_m(\cdot)$ 被**冻结**，以维持其粗粒度动作生成能力与语义文本引导跟随能力。
- 随后，复制一份主分支参数 $\hat{f_m}(\cdot)$ 用于初始化控制分支，并用一个**零初始化**的线性层 $\mathbf{W}_p\in\mathbb{R}^{D_m\times D_m}$ 把二者连接起来，以防止训练早期的噪声导致模型崩塌。
- 接着，条件信号（语音、音乐或其他低层级控制信号）被送入控制分支；其中位置掩码 $\mathbf{M}_c\in\{0,1\}^{F_m}$ 负责把条件信号对齐到动作序列长度 $F_m$，对原始控制信号序列中缺失的 $F_m-T_c$ 帧置零。
- 每个控制分支层的输出通过「零桥（zero bridge）线性层」直接加到主分支对应层的输入上，从而让新的控制信号能够引导**帧级**的人体动作生成。

### 4.2 MC-Attn 设计

MotionCraft 的核心是 MC-Attn，它**并行**捕捉静态与动态人体拓扑图，从而在不可忽略的分布漂移下，增强动作拓扑知识跨多样生成场景的可迁移性。

MC-Attn 有三个关键组件：用于并行建模动作空间属性的**静态骨架图学习器**与**动态拓扑关系图学习器**，以及用于建模各身体部位随时间的帧级动态的**时间注意力**。三个模块共享同一动作表征输入 $\mathbf{H}_m\in\mathbb{R}^{B\times F_m\times D_m}$；最后一层 MC-Attn 的输出会进一步由一个 MoE（混合专家，Shazeer et al. 2017）精炼。

#### 静态骨架图学习器

流程首先构造 $N_b$ 个图顶点表征 $\mathbf{H}_{s}\in\mathbb{R}^{B\times F_m\times N_b\times D_b}$，随后初始化一个**对角单位矩阵** $\mathbf{A}_{s}\in\mathbb{R}^{N_b\times N_b}$ 作为初始静态拓扑图 $\mathcal{G}_s$ 的邻接矩阵——此时每个身体部位仅与自身相连，以避免随机连接导致训练崩塌。

通过优化，$\hat{\mathbf{A}}_{s}$捕捉到**与输入无关**的静态人体拓扑，使模型即使在数据有限的新场景下也能迅速掌握人体的基本结构。该模块输出：

$$\mathbf{E}_{s}=\hat{\mathbf{A}}_{s}\cdot\mathbf{H}_{s}$$

#### 动态拓扑关系图学习器

静态人体拓扑图虽能捕捉基本结构、促进向新分布的快速收敛，但它无法随上下文动态调整，在不断变化的场景中可能造成欠拟合（Zhang et al. 2024b）。

为此，我们引入动态拓扑关系图学习器：它建模动态分布特征，并依据控制信号来适应分布漂移，与静态拓扑结构形成互补。具体而言，该学习器将每个身体部位表示为一个动态图顶点 $\mathbf{H}_{d}\in\mathbb{R}^{B\times F_m\times N_b\times D_b}$，并以注意力分数 $\mathbf{A}_{d}\in\mathbb{R}^{B\times F_m\times N_b\times N_b}$ 作为动态人体拓扑图 $\mathcal{G}_d$ 中的边权重，从而在静态骨架学习器之外进一步增强模型调整空间结构的能力。最终输出为：

$$\mathbf{E}_{d}=\mathbf{A}_{d}\cdot\mathbf{H}_{d}$$

#### 时间注意力

多项研究表明，基础注意力已足以建模时序关系（Nie et al. 2022；Bian et al. 2024）。因此我们以每个身体部位为单元 $\mathbf{H}_{t}\in\mathbb{R}^{B\cdot N_b\times F_m\times D_b}$，基于注意力（Vaswani et al. 2017）来度量帧与帧之间的时序关系。

考虑到外部文本控制信号大多是时间维度上的序列化指令，文本信息也在此处一并建模，输出为：

$$\hat{\mathbf{E}_{t}}=\mathrm{Softmax}\!\left(\mathbf{Q}_{H_t}\cdot[\mathbf{K}_{H_t}^{T},\mathbf{K}_{H_{text}}^{T}]/\sqrt{D_b}\right)\cdot[\mathbf{V}_{H_t}^{T},\mathbf{V}_{H_{text}}^{T}]$$

其中 $\mathbf{Q}_{H_t}=\mathbf{W}^{Q_{H_t}}\mathbf{H}_t$，$\mathbf{K}_{H_t}=\mathbf{W}^{K_{H_t}}\mathbf{H}_t$，$\mathbf{K}_{H_{text}}=\mathbf{W}^{K_{H_{text}}}\mathbf{H}_{text}$，$\mathbf{V}_{H_t}=\mathbf{W}^{V_{H_t}}\mathbf{H}_t$，$\mathbf{V}_{H_{text}}=\mathbf{W}^{V_{H_{text}}}\mathbf{H}_{text}$，$[\,,\,]$ 表示拼接操作。

其他序列化控制模态（如语音与音乐）则在控制分支中建模。

MC-Attn 的最终输出综合了人体骨架的时空表征与各身体部位的时序动态：

$$\mathbf{E}=\mathbf{E}_{s}+\mathbf{E}_{d}+\mathbf{E}_{t}$$

> **图 4**：MotionCraft 与其他当前最优基线在三个代表性任务（文本到动作、语音到手势、音乐到舞蹈）上的定性结果。更详细的可视化对比见补充材料。

### 4.3 MC-Bench 构建

为避免对齐不同动作格式时的信息损失，我们从公开数据集中挑选了各自领域最具代表性的单模态数据集：

- **T2M**：HumanML3D（Guo et al. 2022），SMPL 格式；
- **M2D**：FineDance（Li et al. 2023），SMPL-H Rot-6D 格式；
- **S2G**：BEAT2（Liu et al. 2024a），SMPL-X 格式。

为实现多模态控制下的全身人体动作生成，我们把所有数据统一转换为 **SMPL-X 格式**。关键操作包括：用平均表情填补 HumanML3D 与 FineDance 中缺失的面部信息；把FineDance 从 SMPL-H Rot-6D 格式转换为轴角（axis-angle）表征，以高效对齐 SMPL-X 参数——相比官方的身体重定向（body-retargeting）方法，其对齐误差极小。

随后，我们通过对比学习方式、以检索优化目标（Lu et al. 2023）对齐文本与动作，预训练了一个动作编码器与一个文本编码器，用于对 SMPL-X 动作表征做统一评测。

对于缺少对应文本信息的 FineDance 与 BEAT2，我们生成**伪标注（pseudo-caption）**，例如「A dancer is performing a street dance in the Jazz style to the rhythm of the wildfire.（一位舞者正随着 wildfire 的节奏、以爵士风格表演街舞。）」与「A person is giving a speech, and the content is …（某人正在演讲，内容是……）」。

---

## 5 实验

### 5.1 实现细节

我们为第一阶段的「文本到动作」主干训练设计了两个模型变体：**MotionCraft-Basic** 与 **MotionCraft-Mix**，分别在 MC-Bench 的 HumanML3D 子集与**整个** MC-Bench 上训练。

第二阶段，我们使用 BEAT2（Liu et al. 2024a，大规模语音手势合成数据集）与 FineDance（Li et al. 2023，高质量编舞数据集）分别训练「语音到手势」与「音乐到舞蹈」的控制分支。

MotionCraft-Basic 与 MotionCraft-Mix 共享同一套 **4 层 Transformer**主干配置，将身体拓扑划分为 **12 个部位**，每个部位的隐藏编码维度为 **64**。

MC-Bench 使用统一的全身动作格式 SMPL-X（Pavlakos et al. 2019），采用**轴角**形式，而非关节位置或 6D 旋转。因此我们基于 OpenTMR（Lu et al. 2023），针对 SMPL-X 重新训练了动作编码器与文本编码器以用于评测。

### 表 2：MC-Bench 中 HumanML3D 上的文本到动作结果

我们将本方法与当前最优方法在文本到动作任务上做了对比。（**红色**表示最优结果，_黄色_ 表示次优结果。）

| 方法 | R-Prec Top-1 ↑ | Top-2 ↑ | Top-3 ↑ | FID ↓ | Div ↑ | MM Dist ↓ |
|---|---|---|---|---|---|---|
| GT（真值） | 0.663±0.006 | 0.807±0.002 | 0.864±0.002 | 0.000±0.000 | 36.423±0.183 | 15.567±0.036 |
| T2M-GPT (Zhang et al. 2023b) | 0.529±0.004 | 0.652±0.003 | 0.732±0.003 | 10.457±0.108 | 36.114±0.098 | 17.029±0.039 |
| MDM (Tevet et al. 2023) | 0.383±0.010 | 0.527±0.012 | 0.604±0.009 | 18.671±0.370 | 36.156±0.103 | 18.785±0.054 |
| MotionDiffuse (Zhang et al. 2024a) | 0.525±0.004 | 0.675±0.009 | 0.743±0.009 | 9.982±0.379 | 36.187±0.160 | 17.314±0.066 |
| FineMoGen (Zhang et al. 2023c) | _0.565±0.001_ | _0.710±0.004_ | _0.775±0.004_ | **7.323±0.143** | _36.324±0.069_ | _16.679±0.029_ |
| MCM (Ling et al. 2023) | 0.407±0.002 | 0.559±0.003 | 0.636±0.001 | 15.540±0.443 | 35.813±0.137 | 18.673±0.029 |
| **MotionCraft-Basic** | 0.590±0.003 | 0.743±0.002 | 0.804±0.004 | _8.477±0.102_ | 36.210±0.089 | **16.252±0.035** |
| **MotionCraft-Mix** | **0.600±0.003** | **0.747±0.004** | **0.812±0.006** | **6.707±0.081** | **36.419±0.047** | 16.334±0.059 |

### 5.2 评测指标

#### 文本到动作（Text-to-Motion）

- **FID**（Fréchet Inception Distance）：度量生成动作与真值之间的分布距离；
- **Div**（Diversity，多样性）：度量随机配对的生成动作之间的平均成对欧氏距离；
- **R-Precision**：在 32 个样本的批次内，度量与对应文本描述最接近的 top-k 动作被命中的频率；
- **MM Dist**（Multi-Modal Distance）：量化动作表征与其对应文本特征之间的平均欧氏距离。

#### 语音到手势（Speech-to-Gesture）

使用 $FID_H$、$FID_B$ 与 Div 度量质量与多样性。$FID_H$ 表示**手部**动作分布与真值手势分布之间的差异，$FID_B$ 关注**全身**动作分布之间的距离。此外，我们用 **Beat Alignment Score**（Li et al. 2021）度量动作与语音节拍之间的对齐程度，并用 **L2 Loss** 度量生成表情与真实表情之间的差异。

#### 音乐到舞蹈（Music-to-Dance）

与语音到手势类似，我们使用 $FID_H$、$FID_B$ 与 Div，分别度量手部与全身动作的音乐到动作生成质量，以及生成动作的多样性。

### 5.3 定量与定性结果

我们在三个代表性任务上评估 MotionCraft：① 文本到动作，② 语音到手势，③ 音乐到舞蹈，并分析定量与定性结果。¹ 更多可视化对比见补充材料。

> ¹ 为避免影响评测结果，生成动作与真值中缺失的表情统一用零填充。

### 表 3：MC-Bench 中 BEAT2 上的语音到手势与 FineDance 上的音乐到舞蹈结果

对 S2G，我们分别评测 $FID_H$、$FID_B$、Face L2 Loss（×10⁻⁸）、Beat Align Score（×10⁻¹）与多样性；对 M2D，评测 $FID_H$、$FID_B$ 与多样性。（**红色**表示最优，_黄色_ 表示次优。）

**语音到手势（BEAT2）**

| S2G 方法 | $FID_H$ ↓ | $FID_B$ ↓ | Face L2 Loss ↓ | Beat Align Score ↑ | Div ↑ |
|---|---|---|---|---|---|
| Talkshow | 26.713 | 74.824 | _7.791_ | 6.947 | **13.472** |
| EMAGE | 39.094 | 90.762 | **7.680** | 7.727 | _13.065_ |
| MCM | 23.946 | 71.241 | 16.983 | 7.993 | 13.167 |
| **MotionCraft-Basic** | _18.486_ | _27.023_ | 10.097 | _8.098_ | 10.334 |
| **MotionCraft-Mix** | **12.882** | **25.187** | 8.906 | **8.226** | 12.595 |

**音乐到舞蹈（FineDance）**

| M2D 方法 | $FID_H$ ↓ | $FID_B$ ↓ | Div ↑ |
|---|---|---|---|
| Edge | 93.430 | 108.507 | 13.471 |
| Finedance | 10.747 | _72.229_ | 13.813 |
| MCM | 4.717 | 78.577 | 14.890 |
| **MotionCraft-Basic** | _3.858_ | 76.248 | _16.667_ |
| **MotionCraft-Mix** | **2.849** | **67.159** | **18.483** |

#### 文本到动作生成对比

在文本到动作任务上，我们在两个基准上把 MotionCraft 与当前最优基线（Zhang et al. 2023c；Ling et al. 2023；Zhang et al. 2024b, a；Tevet et al. 2023；Zhang et al. 2023b）做了对比：其一是表 2 中 MC-Bench 的 HumanML3D 子集（全身 SMPL-X 格式），其二是采用纯张量格式的原始 HumanML3D（Guo et al. 2022）（因篇幅限制，结果见补充材料）。

在两个基准上，MotionCraft 都取得了更好的文本引导生成能力、多样性与动作生成质量。

值得注意的是，在 MC-Bench 的 HumanML3D 子集上，原始 HumanML3D 基准因**仅含躯干表征**而评测能力不足的问题得到了显著改善，从而提供了更全面、更客观的比较。这是因为全身 SMPL-X 表征要求模型生成躯干运动、手势与表情，而不仅仅是躯干。

此外，我们发现，在整个 MC-Bench 上训练的 MotionCraft-Mix 相较MotionCraft-Basic 有显著优势。这是因为 MotionCraft-Mix 能够在各类生成场景中高效迁移人体拓扑知识，从而对抗分布漂移。

可视化见图 4：MotionCraft 能够以细粒度控制跟随多样的文本描述。

### 表 4：消融研究

**(a) 模型设计消融（上半部分）**：结果表明，联合建模动态与静态人体骨架拓扑能显著提升性能，因为这提供了可对抗分布漂移的稳健拓扑知识。
**(b) 规模化影响消融（下半部分）**：我们设计了四种规模变体，其中 `**-(a, b, c)` 表示模型 `**` 具有 $a$ 个 Transformer 层、$b$ 维身体部位编码、以及总计 $c$ 的参数量。我们观察到：随着模型规模增大，三类任务上均出现**先升后降**的性能趋势。（**红色**表示最优结果。）

| 动态-空间 | 静态-空间 | HumanML3D Top-1 ↑ | Top-2 ↑ | Top-3 ↑ | FID ↓ | Div ↑ | BEAT2 $FID_H$ ↓ | $FID_B$ ↓ | Face L2 ↓ | Beat Align ↑ | Div ↑ | Finedance $FID_H$ ↓ | $FID_B$ ↓ | Div ↑ |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ✗ | ✗ | 0.583 | 0.729 | 0.794 | 8.911 | 35.954 | 15.587 | 31.839 | 12.448 | 7.908 | 11.752 | 7.088 | 150.733 | 17.984 |
| ✗ | ✓ | 0.557 | 0.706 | 0.772 | 9.041 | 36.101 | 12.929 | 27.928 | 12.287 | 8.077 | 12.230 | 5.104 | 112.186 | **18.503** |
| ✓ | ✗ | 0.582 | 0.732 | 0.798 | 8.455 | 36.241 | 15.517 | 28.631 | 12.544 | 7.708 | 11.313 | 4.972 | 102.103 | 16.385 |
| ✓ | ✓ | **0.600** | **0.747** | **0.812** | **6.707** | **36.419** | **12.882** | **25.187** | **8.906** | **8.226** | **12.595** | **2.849** | **67.159** | 18.483 |

**规模化（Scaling）变体**

| 模型变体（层数, 部位维度, 参数量） | Top-1 ↑ | Top-2 ↑ | Top-3 ↑ | FID ↓ | Div ↑ | $FID_H$ ↓ | $FID_B$ ↓ | Face L2 ↓ | Beat Align ↑ | Div ↑ | $FID_H$ ↓ | $FID_B$ ↓ | Div ↑ |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| MotionCraft-Tiny (4, 64, 77M) | 0.600 | 0.747 | 0.812 | 6.707 | **36.419** | **12.882** | 25.187 | 8.906 | **8.226** | **12.595** | 2.849 | 67.159 | **18.483** |
| MotionCraft-Small (4, 128, 130M) | **0.653** | **0.794** | 0.847 | **5.593** | 36.264 | 15.346 | 27.140 | 8.322 | 8.023 | 11.906 | **2.370** | **59.471** | 17.036 |
| MotionCraft-Small (8, 64, 145M) | 0.635 | 0.779 | 0.802 | 6.193 | 36.311 | 15.702 | 28.094 | 8.589 | 8.031 | 11.824 | 3.749 | 66.958 | 16.478 |
| MotionCraft-Medium (8, 128, 250M) | 0.647 | 0.785 | **0.854** | 5.670 | 36.384 | 14.937 | **23.498** | **8.125** | 8.089 | 10.962 | 3.904 | 75.412 | 16.507 |
| MotionCraft-Large (16, 128, 478M) | 0.604 | 0.744 | 0.809 | 7.872 | 36.169 | 15.964 | 27.476 | 9.036 | 7.969 | 10.625 | 4.837 | 77.341 | 16.426 |

#### 语音到手势生成对比

在表 3 中，我们把 MotionCraft 与 MCM（Ling et al. 2023）、Talkshow（Yi et al. 2023）与 EMAGE（Liu et al. 2024a）做了对比。我们的模型在手部与全身动作生成上都取得了良好的质量与多样性，并且在与第一视角语音的节奏对齐方面表现出色。这归功于我们由粗到细的训练策略，以及从静态与动态人体拓扑图中学到的稳健拓扑知识。

不过在**表情**方面，MotionCraft-Mix 略逊于 EMAGE 与 Talkshow。这源于HumanML3D 与 FineDance 原始数据集的限制：其面部信息是用随机或平均表情填充的，这会干扰第一训练阶段，进而影响后续的 S2G 生成。

尽管如此，我们发现 MotionCraft-Mix 相较 MotionCraft-Basic 仍有明显性能提升，这进一步证实了 MC-Attn 学到的稳健拓扑知识能够跨不同生成场景泛化。

图 4 的定性结果清楚显示，MotionCraft 能够有效跟随节拍，并生成合理的手势与唇部动作。

#### 音乐到舞蹈生成对比

如表 3 所示，MotionCraft 达到了与当前最优基线相当的性能。我们模型的两个变体在多样性上都表现良好，这归因于第一阶段的粗粒度文本到动作生成训练——它为模型装备了跨多种场景的丰富动作拓扑知识。

然而，MotionCraft-Mix 的 FID 相比 MotionCraft-Basic 有所上升。这很可能是因为 FineDance 数据集缺少必要的文本描述，导致第一阶段训练时**同一首歌的不同片段共用了完全相同的伪标注**。这种「一对多」的生成模式，会在第二阶段模型为每个片段引入对应音乐信息、试图学习「多对多」关系时造成混乱。

图 4 的定性结果显示，MotionCraft 能够依据音乐节拍生成自然的舞蹈。

### 5.4 消融研究

我们在表 4 中就MC-Attn 设计的必要性与规模化影响进行了消融探索。

> **图 5**：多模态视频生成应用——基于我们在音乐条件（上排）或语音条件（下排）下生成的动作。我们把它们投影为 2D 图像，作为 MimicMotion（Zhang et al. 2024c）的动作条件。

#### 不同的动作拓扑建模设计

关于解耦静态与动态人体拓扑图学习，我们有三点关键观察：

**① 仅建模静态拓扑**：会降低 T2M 性能，但显著提升 S2G 与 M2D 性能。
我们把这归因于：静态拓扑确保模型掌握身体部位之间的基本空间关系，增强了跨生成场景的泛化能力；但这个与输入无关的额外可学习空间结构模块，反而增加了 T2M 任务的学习难度。

**② 仅建模动态拓扑**：几乎带不来收益。
这是因为输入自适应的动态拓扑邻接矩阵在初始阶段的优化非常复杂——尤其是在需要跨分布漂移迁移拓扑知识时——难以收敛到正确的动态拓扑图（Zhang et al. 2024b）。

**③ 联合建模静态与动态拓扑**：能够有效捕捉可对抗分布漂移的动作知识，这与以人为中心的研究结论一致（Zeng et al. 2021）。静态拓扑学习人体基本结构，为各任务提供基础空间知识；动态拓扑则依据具体动作分布与控制信号做出调整。

#### 规模化（Scaling up）影响

基于对 Transformer 模型可扩展性的共识，我们探究了模型规模对任务性能的影响。我们把 MotionCraft-Mix 从 **77M 扩展到 478M**，观察到在数据有限的条件下，三类任务的性能随模型规模增大呈**先升后降**的趋势。

这验证了：增加模型参数量可以增强生成能力，但若没有相应增加高质量数据，模型性能可能反而下降。

### 5.5 应用：多模态视频生成

为展示下游应用，图 5 中我们呈现了由 MotionCraft 在 M2D 与 S2G 下驱动的两段动画视频。我们生成的动作序列可与任意现成的人体视频生成框架结合，例如 MimicMotion（Zhang et al. 2024c）、AnimateAnyone（Hu et al. 2023）与 VividPose（Wang et al. 2024），使用户能够基于特定控制信号（如语音或音乐）为任意角色定制视频。

值得注意的是，与从视频中估计的传统 2D 关键点不同，我们生成的 **3D 动作**方案允许灵活调整相机参数，以投影出不同的可见身体区域（例如图 5 中的全身或上半身）。更详细的可视化见补充材料。

---

## 6 结论

本文提出 **MotionCraft**，一个用于「即插即用多模态控制下全身人体动作生成」的统一框架，它能够跨不同生成分布泛化，并高效处理不同粒度的控制信号。

MotionCraft 采用由粗到细的训练策略，在无需承担混合训练优化负担的前提下，为不同条件（包括文本、语音与音乐）实现细粒度的即插即用控制。我们的核心设计是 **MC-Attn**，它通过并行建模静态与动态人体拓扑图，有效地在不同分布之间学习并迁移动作知识。我们还提出了 **MC-Bench**——首个基于统一全身 SMPL-X 表征、可公开获取的多模态全身动作生成基准。

大量实验表明，MotionCraft 在标准动作生成任务上取得了与当前最优基线相比具有竞争力的性能。

---

# 附录（Appendix）

> 以下附录内容为 v3 版本相较早期版本大幅扩充的部分。

## A 可视化结果

由于 PDF 静态格式与篇幅限制，更多可视化与对比可在补充材料与项目主页中获取，其中包括针对单一任务（文本到动作 T2M、音乐到舞蹈 M2D、语音到手势 S2G）的生成可视化，以及**在单条长序列内进行即插即用控制生成**的可视化。视频还演示了我们生成的动作序列在视频制作与角色动画中的应用。

## B 相关工作（扩充版）

### B.1 人体动作生成模型

条件化人体动作生成模型已取得显著进展，包括T2M（Tevet et al. 2023；Zhang et al. 2023b；Liu et al. 2023；Zhang et al. 2024a, 2023c；Liang et al. 2024）、S2G（Yi et al. 2023；Chen et al. 2024；Liu et al. 2022b）与 M2D（Li et al. 2023；Tseng, Castellon, and Liu 2023；Siyao et al. 2022）。

- **文本到动作**：相关模型（Tevet et al. 2023；Chen et al. 2023；Zhang et al. 2023b；Liu et al. 2023；Zhang et al. 2024a, 2023c；Liang et al. 2024）通过应用先进的生成模型、并对齐动作与文本特征域，实现了具有语义一致性的文本可控动作生成。
- **语音到手势**（Yi et al. 2023；Chen et al. 2024；Liu et al. 2022b）：许多工作聚焦于通过节奏对齐与角色风格学习，把语音映射到人体手势。
- **音乐到舞蹈**：大量研究（Li et al. 2023；Tseng, Castellon, and Liu 2023；Siyao et al. 2022）设计了空间与时间一致性约束，以确保模型从输入音乐中学到相应的风格与节奏。

近来，多模态动作生成受到越来越多关注（Ling et al. 2023；Zhang et al. 2024b；Luo et al. 2024）：

- **M³-GPT**（Luo et al. 2024）把量化后的条件 token 注入大语言模型词表以实现动作理解与生成，但忽视了人体拓扑先验的建模。
- **Motion-Verse**（Zhang et al. 2024b）引入动态注意力评估身体部位间的关系，但未能捕捉整体静态人体拓扑，导致泛化受限、优化复杂度上升。此外，它基于 ImageBind（Girdhar et al. 2023）对所有条件混合训练，这在同时学习不同粒度条件时造成优化困难，且对新控制信号需要重新训练。
- **MCM**（Ling et al. 2023）尝试基于 ControlNet（Zhang, Rao, and Agrawala 2023）架构解决混合训练的优化混乱，但它完全没有建模人体拓扑结构，导致跨场景泛化较差。此外，**MCM 只关注躯干运动，缺乏生成全身动作的能力**。

如表 1 所示，与以往方法相比，MotionCraft 通过 MC-Attn 捕捉静态人体拓扑与领域特定的动态骨架关系、引入控制分支、并采用由粗到细的训练策略，实现了不同控制信号下具备即插即用能力的全身动作生成。

### B.2 人体动作生成基准（扩充版）

**T2M**：研究者构建了涵盖不同抽象层级的数据集，包括动作类别（Chung et al. 2021；Trivedi, Thatipelli, and Sarvadevabhatla 2021）、序列化动作标签（Zhang et al. 2022；Guo et al. 2020）以及任意自然语言描述（Lin et al. 2023a；Guo et al. 2022；Tang et al. 2023）。

- **AMASS**（Mahmood et al. 2019）把 15 个基于光学标记的动作捕捉数据集整合为一个基于 SMPL（Loper et al. 2015）表征的综合集合。
- **HumanML3D**（Guo et al. 2022）从 AMASS（Mahmood et al. 2019）中抽取了一个高质量子集，采用 H3D 格式做**仅躯干**生成，并为每个动作片段提供来自不同标注者的三条任意自然语言描述。

**M2D**：

- **AIST++**（Li et al. 2021）从视频重建了 5 小时的 SMPL（Loper et al. 2015）格式舞蹈，但重建误差显著，且缺少手部动作捕捉。
- **FineDance**（Li et al. 2023）收集了跨 22 种风格、共 14.6 小时的舞蹈，并以 SMPL-H（Pavlakos et al. 2019）格式补充了细致手势。

**S2G**：相关数据集（Liu et al. 2024a, 2022a；Yi et al. 2023）来自伪标签（PGT）与动作捕捉两类来源。由于 PGT 中单目 3D 姿态估计存在显著误差（Gärtner et al. 2022），Mocap 数据集通常更受青睐。近来**BEAT2**（Liu et al. 2024a）与 **BEAT**（Liu et al. 2022a）因动作多样、数据量大而成为最流行的基准；BEAT2 在 BEAT 基础上使用 SMPL-X 与 FLAME（Kim, Kim, and Choi 2023），实现了更高质量的统一网格级数据。

尽管有上述进展，仍没有公开可用的基准支持多模态全身动作生成的统一表征。

## C MC-Bench 构建（详细）

### C.1 动作表征

从「仅身体」动作生成（Guo et al. 2022；Li et al. 2021；Ling et al. 2023）到「全身」动作生成（Lu et al. 2023；Li et al. 2023；Liu et al. 2024a；Zhang et al. 2024b），已有研究在生成任务中探索了多种动作表征，包括：基于 SMPL 网格参数的默认**轴角**输入（Loper et al. 2015）、**6D 旋转**（Li et al. 2023）、**四元数**（Pavlakos et al. 2019），以及从 SMPL 扩展而来的 **H3D 格式**（Guo et al. 2022）——后者额外加入了关节位置与速度等冗余信息。近年来，**SMPL-X**（Pavlakos et al. 2019）作为 SMPL 的扩展，纳入了手部建模，从而支持更细粒度的手指关节建模。

因此，出于动作表征的实用性与效率考虑，我们采用 **SMPL-X 的默认轴角输入**来建模主体与手部。

具体而言，第 $i$ 个姿态由以下元组定义：

- 绕X（Y、Z）轴的**根轴角** $\dot{r}^{r}\in\mathbb{R}^{3}$；
- 沿 X（Y、Z）轴的**根轨迹** $\dot{r}^{t}\in\mathbb{R}^{3}$；
- **局部关节轴角旋转** $\mathbf{\theta}^{r}\in\mathbb{R}^{3N}$，其中 $N$ 表示全身关节数量（包含身体关节与手部关节）。

对于面部动作表征，我们遵循 MotionX（Lin et al. 2023a），采用：

- $\mathbf{f}^{s}\in\mathbb{R}^{100}$ 表示**面部形状**；
- FLAME 格式（Kim, Kim, and Choi 2023）下的 $\mathbf{f}^{e}\in\mathbb{R}^{50}$ 表示**面部表情**；
- 下颌轴角旋转 $\mathbf{\theta}^{j}\in\mathbb{R}^{3}$ 用于**下颌**建模。

此外，我们采用标准 SMPL-X 模型的 10 维参数 $\mathbf{\theta}^{b}\in\mathbb{R}^{10}$ 表示**身体形状**。

于是，全身动作被表示为：

$$\mathbf{m}_{i}=\{\dot{r}^{r},\ \dot{r}^{t},\ \mathbf{\theta}^{r},\ \mathbf{f}^{s},\ \mathbf{f}^{e},\ \mathbf{\theta}^{j},\ \mathbf{\theta}^{b}\}$$

### C.2 文本到动作子集构建

在第一阶段的语义「文本到动作」预训练中，我们的数据主要由以下三部分构成：

1. **HumanML3D**（Guo et al. 2022）是一个具有代表性的 3D「动作-文本」数据集，包含 14,616 条高质量人体动作与 44,970 条文本描述配对。我们**没有**使用原始 HumanML3D 的「仅身体」H3D 格式，而是从其原始 AMASS（Mahmood et al. 2019）数据中以 SMPL-X 格式重新抽取对应实例，并按我们的 SMPL-X 轴角格式处理每一动作帧，对任何缺失的身体部位把相应 SMPL-X（Pavlakos et al. 2019）表征置零。文本部分依照原始 HumanML3D 的文本描述处理流程进行过滤与处理。

2. **BEAT2**（Liu et al. 2024a）是一个包含多种说话风格与说话者 ID 的「语音到手势」数据集。它提供 SMPL-X（Pavlakos et al. 2019）轴角旋转的动作表征，我们直接依据我们的全身动作格式 $\mathbf{m}_{i}=\{\dot{r}^{r},\dot{r}^{t},\mathbf{\theta}^{r},\mathbf{f}^{s},\mathbf{f}^{e},\mathbf{\theta}^{j},\mathbf{\theta}^{b}\}$ 提取相应旋转信息。文本部分，我们用简单规则生成对应的伪语义文本描述，例如「A person is giving a speech, and the content is …」。

3. **FineDance**（Li et al. 2023）是目前在编舞多样性与数据量方面领先的「音乐到舞蹈」数据集之一，原始提供基于 SMPL-H（Pavlakos et al. 2019）rot6D 的「身体 + 手部」数据表征。我们首先利用不同旋转表征之间的等价性（Zhou et al. 2019），把简化的 rot6D 旋转矩阵转换为绕XYZ 轴的轴角格式。我们**没有**使用官方的 SMPL-H → SMPL-X 重定向优化方法，而是直接把 SMPL-H 参数映射到 SMPL-X（Pavlakos et al. 2019）。我们的定性与定量实验证明这种简单做法是有效的，重定向误差可忽略不计。文本部分，我们用基础规则生成与对应音乐片段匹配的伪语义描述，例如「A dancer is performing a street dance in the Jazz style to the rhythm of the wildfire.」

### C.3 语音到手势子集构建

对 BEAT2（Liu et al. 2024a）中的所有语音，我们使用 Librosa（McFee et al. 2015）提取与**语音韵律**相关的 2 维时序语音特征。音频采样率为 **76,800 Hz**，hop size 为 **512**。我们把动作序列与对应语音切分为 **64 帧**的片段，步长为 **64 帧**。在后续生成中，我们采用 DiffSHEG（Chen et al. 2024）中基于 outpainting 的采样策略来实现**长时程**手势生成。

动作表征与语义文本描述的预处理细节已在前文详述，此处不再重复。

### C.4 音乐到舞蹈子集构建

对 FineDance（Li et al. 2023）中的所有音乐，我们使用 Librosa（McFee et al. 2015）提取 **35 维**时序音乐特征。音频采样率为 **76,800 Hz**，hop size 为 **512**。我们把动作序列与对应音乐切分为 **120 帧**的片段，步长为 **30 帧**。

动作表征与语义文本描述的预处理细节已在前文详述，此处不再重复。

### 表 5：原始 HumanML3D 基准上的文本到动作结果

我们将本方法与当前最优方法在文本到动作生成上做了对比。我们的方法取得了更好的语义相关性、保真度与多样性表现。（**红色**表示最优，_黄色_ 表示次优。）

| 方法 | R-Prec Top-1 ↑ | Top-2 ↑ | Top-3 ↑ | FID ↓ | Div ↑ | MM Dist ↓ |
|---|---|---|---|---|---|---|
| GT（真值） | 0.511±0.003 | 0.703±0.003 | 0.797±0.002 | 0.002±0.000 | 9.503±0.065 | 2.974±0.008 |
| T2M-GPT (Zhang et al. 2023b) | 0.491±0.003 | 0.680±0.003 | 0.775±0.002 | _0.116±0.004_ | 9.761±0.081 | 3.118±0.011 |
| MDM (Tevet et al. 2023) | 0.418±0.005 | 0.604±0.005 | 0.707±0.004 | 0.489±0.025 | 9.450±0.066 | 3.630±0.023 |
| MotionDiffuse (Zhang et al. 2024a) | 0.491±0.001 | 0.681±0.001 | 0.782±0.001 | 0.630±0.001 | 9.410±0.049 | 3.113±0.001 |
| FineMoGen (Zhang et al. 2023c) | **0.504±0.002** | _0.690±0.002_ | 0.784±0.002 | 0.151±0.008 | 9.263±0.094 | **2.998±0.008** |
| Motion-Verse (Zhang et al. 2024b) | 0.496±0.002 | 0.685±0.002 | 0.785±0.002 | 0.415±0.002 | 9.176±0.074 | 3.087±0.012 |
| MCM (Ling et al. 2023) | 0.494±0.003 | 0.682±0.005 | 0.777±0.003 | **0.075±0.003** | 9.484±0.074 | 3.086±0.011 |
| **MotionCraft-Basic** | _0.501±0.003_ | **0.697±0.003** | **0.796±0.002** | 0.173±0.002 | **9.543±0.098** | _3.025±0.008_ |

### 表 6：附加消融研究

我们探究了第二阶段的模型训练策略——关于**分部位编码器（解码器）**与**动作序列时序关系建模范式**。

- **「Local-Unfreeze」** 列表示：在第二阶段中，只把分部位编码器与解码器中**与特定控制信号对应的身体部位**解冻。例如在「语音到手势」任务中只解冻手部与面部的编码器与解码器；在「音乐到舞蹈」中只解冻手部的编码器与解码器。
- **「Temporal-Patching」** 列表示：对相邻帧执行 patching 操作，把指定数量的邻近帧在时间维度上压缩为单个基本建模单元，而不是把每一帧动作作为时间维度上的基本单元。

（**红色**表示最优结果。）

| Local-Unfreeze | Temporal-Patching | Top-1 ↑ | Top-2 ↑ | Top-3 ↑ | FID ↓ | Div ↑ | $FID_H$ ↓ | $FID_B$ ↓ | Face L2 ↓ | Beat Align ↑ | Div ↑ | $FID_H$ ↓ | $FID_B$ ↓ | Div ↑ |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ✗ | ✗ | **0.653** | **0.794** | **0.847** | **5.593** | **36.264** | **15.346** | 27.140 | **8.322** | 8.023 | 11.024 | **2.370** | 59.471 | **17.036** |
| ✗ | ✓ | 0.628 | 0.776 | 0.834 | 5.944 | 36.189 | 17.583 | 27.605 | 8.792 | 8.007 | 10.920 | 3.496 | 64.784 | 16.371 |
| ✓ | ✗ | – | – | – | – | – | 17.962 | **26.556** | 8.561 | **8.035** | **11.248** | 2.493 | **56.847** | 16.894 |
| ✓ | ✓ | – | – | – | – | – | 18.554 | 28.434 | 8.630 | 7.980 | 11.157 | 3.229 | 61.518 | 16.502 |

## D 实验（详细）

### D.1 MotionCraft 实现细节

我们采用 **4 层**动作扩散 Transformer 作为 MotionCraft 的主干，隐维度为 $12\times 64$，前馈嵌入大小为 **256**，其中 **12** 对应身体部位数量，**64** 表示每个部位特定隐状态的维度。

对于 MotionCraft 的控制分支，我们把复制的 MotionCraft block 数量设为 **2**，即总层数的一半。各类低层级控制信号（语音或音乐）的编码器设计与基线（Liu et al. 2024a；Li et al. 2023）保持一致。

文本编码器方面，我们使用**冻结的 CLIP ViT-B/32** 编码器，并额外增加两层 Transformer encoder 层。

扩散模型中，方差$\beta_t$ 预定义为从 **0.0001** 线性变化到 **0.02**，共 **1000** 个加噪步。遵循 MDM（Tevet et al. 2023），我们把扩散预测目标设为 $x_{start}$ 而非噪声。

模型使用 Adam 优化器训练，初始学习率为 $2\times 10^{-4}$，通过 cosine 调度衰减到 $2\times 10^{-5}$。训练在 **8× NVIDIA Tesla V100-32GB** GPU 上进行，每卡 batch size 为 **64**，耗时约 **48 小时**。

### D.2 评测中「文本-动作检索」预训练的实现细节

由于我们采用基于 SMPL-X 的轴角作为动作表征，以往研究的动作编码器与文本编码器无法直接用于评测。因此，遵循 HumanTomato（Lu et al. 2023），我们专门针对我们的 SMPL-X 轴角动作表征，重新训练了一个「文本-全身动作」检索模型，以对比学习方式评估性能。

该检索模型采用基于 VAE 的架构（Petrovich, Black, and Varol 2022），由动作编码器、文本编码器与动作解码器组成。训练目标是以下各项的加权和：

$$\min \mathcal{L}_{rec}+\lambda_{KL}\mathcal{L}_{KL}+\lambda_{E}\mathcal{L}_{E}+\lambda_{NCE}\mathcal{L}_{NCE}$$

其中四个损失项分别为：重建损失、KL（Kullback-Leibler）散度损失、跨模态嵌入相似度损失，以及 InfoNCE（Oord, Li, and Vinyals 2018）损失。超参数设为$\lambda_{KL}=1\times 10^{-5}$，$\lambda_{E}=1\times 10^{-5}$，$\lambda_{NCE}=1\times 10^{-1}$。

### D.3 文本到动作的更多结果

为提供更全面的比较，除了在 MC-Bench 中采用全身 SMPL-X 格式的 HumanML3D 子集上评测（表 2）之外，我们还在使用「仅身体」H3D 格式（含冗余信息）的**原始 HumanML3D**（Guo et al. 2022）上，把 MotionCraft 与当前最优基线（Zhang et al. 2023c；Ling et al. 2023；Zhang et al. 2024b, a；Tevet et al. 2023；Zhang et al. 2023b）做了对比。定量对比结果见表 5。可以明显看出，在原始 HumanML3D 的文本到动作基准上，MotionCraft 同样取得了更好的文本引导生成能力、多样性与动作生成质量。

值得注意的是：在 MC-Bench 的 HumanML3D 子集上，原始 HumanML3D 基准因**仅躯干表征**而表现出的「模型间性能差异极小」的评测能力局限，得到了显著改善。这一改善源于全身 SMPL-X 表征要求模型生成躯干运动、手势与表情，而非仅聚焦躯干。

### D.4 消融研究的更多结果

除表 4 中关于动静态动作拓扑建模与模型参数规模化的消融实验之外，我们还进一步探究了第二阶段的模型训练策略与动作序列时序关系建模范式，结果见表 6。

#### 第二阶段分部位编码器/解码器训练策略的消融

表 6 中的「Local-Unfreeze」列表示：在第二阶段中，只把分部位编码器与解码器中与特定控制信号对应的身体部位解冻。例如在 S2G 任务中只解冻手部与面部的编码器/解码器；在 M2D 中只解冻手部的编码器/解码器。

表 6 的第一行与第三行清楚显示：在第二阶段**完全解冻**分部位编码器与解码器，能够增强针对生成场景中特定身体部位的编解码优化，从而提升目标场景的生成能力（例如 S2G 中的手部与面部建模、M2D 中的手部建模）。反之，**部分解冻**有助于保留第一阶段文本语义预训练中学到的人体拓扑知识，从而在下游生成任务上稳定整体的全身动作生成能力。

#### 动作序列时序关系建模的消融

当前对动作序列时序关系建模有两种经典做法：把每一帧动作作为时间维度上的基本单元来做序列建模；或对相邻帧执行 patching 操作，把指定数量的邻近帧在时间维度上压缩为单个基本单元来建模。

在一般时间序列分析中，后者已被广泛证明能显著提升基于 Transformer 的模型性能（Nie et al. 2022），因为它能缓解极端值的影响、消除冗余信息，使子序列关系建模具有更高的信息密度。然而如表 6 所示，**动作序列时序动态建模的结论与一般时间序列建模恰好相反**。这一差异可归因于动作数据与一般时间序列在数据表征上的不同：

- 来自动作捕捉系统的高质量动作数据（Guo et al. 2022；Li et al. 2023；Liu et al. 2024a）通常不存在极端值问题；而一般时间序列数据（Nie et al. 2022）会受到采集环境、经济或文化等各种因素影响，质量参差不齐。
- 在 SMPL-X 轴角动作格式中，**子关节的旋转角度受其父关节影响**，这意味着由于全身拓扑结构（Loper et al. 2015；Pavlakos et al. 2019），父关节数值的微小变化就可能导致动作的显著改变。换言之，SMPL-X 中的轴角动作表征远比一般时间序列数据**敏感**，若以压缩后的子序列作为建模单元，会引入显著的累积误差。

## E 更广泛影响与局限

本节讨论 MotionCraft 可能的社会影响与局限。

### E.1 更广泛影响

首先，我们探索了多模态控制下的全身动作生成任务，并基于三个高质量的单控制信号动作生成数据集，建立了首个采用统一全身动作表征的多模态动作生成基准。这些可以为「多模态控制动作生成」研究社区提供基础。

其次，经过多模态控制下的大规模动作数据训练，我们训练好的 MotionCraft 可以作为**动作先验（motion prior）**服务于其他研究，例如 HumanTomato（Lu et al. 2023）与 VPoser（Pavlakos et al. 2019）。它还有助于应对当前动作捕捉流程中的噪声标注问题（Xia et al. 2020；Lin et al. 2023b）。

最后，富有表现力、可多模态控制、且高质量的动作生成，可以应用于各种实际下游场景，包括但不限于人体视频生成、动作动画与机器人学。

### E.2 局限

尽管本工作在多模态控制下的全身动作生成上取得了显著进展，但仍存在一些局限。

**其一**，如何为全身动作生成利用更多高质量语义文本描述，仍需进一步研究。本工作沿用以往做法，在两阶段训练中都只使用序列级语义描述，未纳入**帧级**或细粒度的全身描述。这一问题在 S2G（Liu et al. 2024a）与 M2D（Li et al. 2023）任务中尤为重要——这两类任务天然缺乏序列级语义描述，只能用伪文本描述来补充。

**其二**，当前动作表征采用轴角 SMPL-X 格式。虽然这允许通过 SMPL-X 模型（Pavlakos et al. 2019）直接渲染出相应网格，但**根旋转与根轨迹的 6D 参数**会显著影响整体动作生成质量，在模型训练中带来额外波动。未来我们将探索加入投影后的 3D关节位置——类似 HumanML3D（Guo et al. 2022）中冗余的 H3D 格式——以对根旋转与根轨迹提供额外约束。

**此外**，我们还计划统一更多多模态数据集，以推动更优秀的多模态全身动作生成模型的发展。

---

## 参考文献

- Bian, Y.; Ju, X.; Li, J.; Xu, Z.; Cheng, D.; and Xu, Q. 2024. *Multi-Patch Prediction: Adapting Language Models for Time Series Representation Learning.* ICML 2024.
- Chen, J.; Liu, Y.; Wang, J.; Zeng, A.; Li, Y.; and Chen, Q. 2024. *DiffSHEG: A Diffusion-Based Approach for Real-Time Speech-driven Holistic 3D Expression and Gesture Generation.* CVPR.
- Chen, X.; Jiang, B.; Liu, W.; Huang, Z.; Fu, B.; Chen, T.; and Yu, G. 2023. *Executing your Commands via Motion Diffusion in Latent Space.* CVPR, 18000–18010.
- Chung, J.; Wuu, C.-h.; Yang, H.-r.; Tai, Y.-W.; and Tang, C.-K. 2021. *Haa500: Human-centric atomic action dataset with curated videos.* ICCV, 13465–13474.
- Gärtner, E.; Andriluka, M.; Coumans, E.; and Sminchisescu, C. 2022. *Differentiable dynamics for articulated 3d human motion reconstruction.* CVPR, 13190–13200.
- Girdhar, R.; El-Nouby, A.; Liu, Z.; Singh, M.; Alwala, K. V.; Joulin, A.; and Misra, I. 2023. *ImageBind: One embedding space to bind them all.* CVPR, 15180–15190.
- Guo, C.; Zou, S.; Zuo, X.; Wang, S.; Ji, W.; Li, X.; and Cheng, L. 2022. *Generating Diverse and Natural 3D Human Motions From Text.* CVPR, 5152–5161.
- Guo, C.; Zuo, X.; Wang, S.; Zou, S.; Sun, Q.; Deng, A.; Gong, M.; and Cheng, L. 2020. *Action2motion: Conditioned generation of 3d human motions.* ACM MM, 2021–2029.
- Hu, L. 2024. *Animate anyone: Consistent and controllable image-to-video synthesis for character animation.* CVPR, 8153–8163.
- Hu, L.; Gao, X.; Zhang, P.; Sun, K.; Zhang, B.; and Bo, L. 2023. *Animate Anyone.* arXiv:2311.17117.
- Kim, J.; Kim, J.; and Choi, S. 2023. *FLAME: Free-form language-based motion synthesis & editing.* AAAI.
- Li, R.; Yang, S.; Ross, D. A.; and Kanazawa, A. 2021. *Learn to Dance with AIST++: Music Conditioned 3D Dance Generation.* arXiv:2101.08779.
- Li, R.; Zhao, J.; Zhang, Y.; Su, M.; Ren, Z.; Zhang, H.; Tang, Y.; and Li, X. 2023. *FineDance: A Fine-grained Choreography Dataset for 3D Full Body Dance Generation.* ICCV, 10234–10243.
- Liang, H.; Bao, J.; Zhang, R.; Ren, S.; Xu, Y.; Yang, S.; Chen, X.; Yu, J.; and Xu, L. 2024. *OMG: Towards open-vocabulary motion generation via mixture of controllers.* CVPR, 482–493.
- Lin, J.; Zeng, A.; Lu, S.; Cai, Y.; Zhang, R.; Wang, H.; and Zhang, L. 2023a. *Motion-X: A Large-scale 3D Expressive Whole-body Human Motion Dataset.* NeurIPS.
- Lin, J.; Zeng, A.; Wang, H.; Zhang, L.; and Li, Y. 2023b. *One-Stage 3D Whole-Body Mesh Recovery with Component Aware Transformer.* CVPR, 21159–21168.
- Ling, Z.; Han, B.; Wong, Y.; Kangkanhalli, M.; and Geng, W. 2023. *MCM: Multi-condition motion synthesis framework for multi-scenario.* arXiv:2309.03031.
- Liu, H.; Zhu, Z.; Becherini, G.; Peng, Y.; Su, M.; Zhou, Y.; Zhe, X.; Iwamoto, N.; Zheng, B.; and Black, M. J. 2024a. *EMAGE: Towards Unified Holistic Co-Speech Gesture Generation via Expressive Masked Audio Gesture Modeling.* CVPR, 1144–1154.
- Liu, H.; Zhu, Z.; Iwamoto, N.; Peng, Y.; Li, Z.; Zhou, Y.; Bozkurt, E.; and Zheng, B. 2022a. *BEAT: A large-scale semantic and emotional multi-modal dataset for conversational gestures synthesis.* ECCV, 612–630.
- Liu, J.; Dai, W.; Wang, C.; Cheng, Y.; Tang, Y.; and Tong, X. 2023. *Plan, posture and go: Towards open-world text-to-motion generation.* arXiv:2312.14828.
- Liu, X.; Wu, Q.; Zhou, H.; Xu, Y.; Qian, R.; Lin, X.; Zhou, X.; Wu, W.; Dai, B.; and Zhou, B. 2022b. *Learning hierarchical cross-modal association for co-speech gesture generation.* CVPR, 10462–10472.
- Liu, Y.; Zhang, K.; Li, Y.; Yan, Z.; Gao, C.; Chen, R.; Yuan, Z.; Huang, Y.; Sun, H.; Gao, J.; He, L.; and Sun, L. 2024b. *Sora: A Review on Background, Technology, Limitations, and Opportunities of Large Vision Models.* arXiv:2402.17177.
- Loper, M.; Mahmood, N.; Romero, J.; Pons-Moll, G.; and Black, M. J. 2015. *SMPL: A Skinned Multi-Person Linear Model.* ACM TOG (SIGGRAPH Asia), 34(6): 248:1–248:16.
- Lu, S.; Chen, L.-H.; Zeng, A.; Lin, J.; Zhang, R.; Zhang, L.; and Shum, H.-Y. 2023. *HumanTOMATO: Text-aligned Whole-body Motion Generation.* arXiv:2310.12978.
- Luo, M.; Hou, R.; Chang, H.; Liu, Z.; Wang, Y.; and Shan, S. 2024. *M³-GPT: An Advanced Multimodal, Multitask Framework for Motion Comprehension and Generation.* arXiv:2405.16273.
- Ma, J.; Bai, S.; and Zhou, C. 2022. *Pretrained diffusion models for unified human motion synthesis.* arXiv:2212.02837.
- Mahmood, N.; Ghorbani, N.; Troje, N. F.; Pons-Moll, G.; and Black, M. J. 2019. *AMASS: Archive of motion capture as surface shapes.* ICCV, 5442–5451.
- McFee, B.; Raffel, C.; Liang, D.; Ellis, D. P.; McVicar, M.; Battenberg, E.; and Nieto, O. 2015. *librosa: Audio and music signal analysis in python.* SciPy, 18–24.
- Nie, Y.; Nguyen, N. H.; Sinthong, P.; and Kalagnanam, J. 2022. *A time series is worth 64 words: Long-term forecasting with transformers.* arXiv:2211.14730.
- Oord, A. v. d.; Li, Y.; and Vinyals, O. 2018. *Representation learning with contrastive predictive coding.* arXiv:1807.03748.
- Pavlakos, G.; Choutas, V.; Ghorbani, N.; Bolkart, T.; Osman, A. A. A.; Tzionas, D.; and Black, M. J. 2019. *Expressive Body Capture: 3D Hands, Face, and Body from a Single Image.* CVPR, 10975–10985.
- Petrovich, M.; Black, M. J.; and Varol, G. 2022. *TEMOS: Generating diverse human motions from textual descriptions.* ECCV, 480–497.
- Rombach, R.; Blattmann, A.; Lorenz, D.; Esser, P.; and Ommer, B. 2022. *High-resolution image synthesis with latent diffusion models.* CVPR, 10684–10695.
- Shazeer, N.; Mirhoseini, A.; Maziarz, K.; Davis, A.; Le, Q.; Hinton, G.; and Dean, J. 2017. *Outrageously large neural networks: The sparsely-gated mixture-of-experts layer.* arXiv:1701.06538.
- Siyao, L.; Yu, W.; Gu, T.; Lin, C.; Wang, Q.; Qian, C.; Loy, C. C.; and Liu, Z. 2022. *Bailando: 3d dance generation by actor-critic gpt with choreographic memory.* CVPR, 11050–11059.
- Tang, Y.; Liu, J.; Liu, A.; Yang, B.; Dai, W.; Rao, Y.; Lu, J.; Zhou, J.; and Li, X. 2023. *FLAG3D: A 3D Fitness Activity Dataset with Language Instruction.* CVPR.
- Team, G.; Anil, R.; Borgeaud, S.; et al. 2023. *Gemini: a family of highly capable multimodal models.* arXiv:2312.11805.
- Tevet, G.; Raab, S.; Gordon, B.; Shafir, Y.; Cohen-or, D.; and Bermano, A. H. 2023. *Human Motion Diffusion Model.* ICLR.
- Trivedi, N.; Thatipelli, A.; and Sarvadevabhatla, R. K. 2021. *NTU-X: an enhanced large-scale dataset for improving pose-based recognition of subtle human actions.* ICVGIP, 1–9.
- Tseng, J.; Castellon, R.; and Liu, K. 2023. *EDGE: Editable dance generation from music.* CVPR, 448–458.
- Vaswani, A.; Shazeer, N.; Parmar, N.; Uszkoreit, J.; Jones, L.; Gomez, A. N.; Kaiser, Ł.; and Polosukhin, I. 2017. *Attention is all you need.* NeurIPS, 30.
- Wang, Q.; Jiang, Z.; Xu, C.; Zhang, J.; Wang, Y.; Zhang, X.; Cao, Y.; Cao, W.; Wang, C.; and Fu, Y. 2024. *VividPose: Advancing Stable Video Diffusion for Realistic Human Image Animation.* arXiv:2405.18156v1.
- Xia, X.; Liu, T.; Han, B.; Wang, N.; Gong, M.; Liu, H.; Niu, G.; Tao, D.; and Sugiyama, M. 2020. *Part-dependent label noise: Towards instance-dependent label noise.* NeurIPS, 33: 7597–7610.
- Yi, H.; Liang, H.; Liu, Y.; Cao, Q.; Wen, Y.; Bolkart, T.; Tao, D.; and Black, M. J. 2023. *Generating Holistic 3D Human Motion from Speech.* CVPR.
- Zeng, A.; Sun, X.; Yang, L.; Zhao, N.; Liu, M.; and Xu, Q. 2021. *Learning skeletal graph neural networks for hard 3d pose estimation.* ICCV, 11436–11445.
- Zhang, J.; Yan, H.; Xu, Z.; Feng, J.; and Liew, J. H. 2023a. *MagicAvatar: Multi-modal Avatar Generation and Animation.* arXiv:2308.14748.
- Zhang, J.; Zhang, Y.; Cun, X.; Huang, S.; Zhang, Y.; Zhao, H.; Lu, H.; and Shen, X. 2023b. *T2M-GPT: Generating Human Motion from Textual Descriptions with Discrete Representations.* CVPR.
- Zhang, L.; Rao, A.; and Agrawala, M. 2023. *Adding conditional control to text-to-image diffusion models.* ICCV, 3836–3847.
- Zhang, M.; Cai, Z.; Pan, L.; Hong, F.; Guo, X.; Yang, L.; and Liu, Z. 2024a. *MotionDiffuse: Text-driven human motion generation with diffusion model.* IEEE TPAMI.
- Zhang, M.; Jin, D.; Gu, C.; Hong, F.; Cai, Z.; Huang, J.; Zhang, C.; Guo, X.; Yang, L.; He, Y.; et al. 2024b. *Large motion model for unified multi-modal motion generation.* arXiv:2404.01284.
- Zhang, M.; Li, H.; Cai, Z.; Ren, J.; Yang, L.; and Liu, Z. 2023c. *FineMoGen: Fine-Grained Spatio-Temporal Motion Generation and Editing.* NeurIPS.
- Zhang, S.; Ma, Q.; Zhang, Y.; Qian, Z.; Kwon, T.; Pollefeys, M.; Bogo, F.; and Tang, S. 2022. *EgoBody: Human body shape and motion of interacting people from head-mounted devices.* ECCV, 180–200.
- Zhang, Y.; Gu, J.; Wang, L.-W.; Wang, H.; Cheng, J.; Zhu, Y.; and Zou, F. 2024c. *MimicMotion: High-Quality Human Motion Video Generation with Confidence-aware Pose Guidance.* arXiv:2406.19680.
- Zhou, Y.; Barnes, C.; Lu, J.; Yang, J.; and Li, H. 2019. *On the continuity of rotation representations in neural networks.* CVPR, 5745–5753.
- Zhou, Z.; Wan, Y.; and Wang, B. 2023. *A unified framework for multimodal, multi-part human motion synthesis.* arXiv:2311.16471.

---

*原文由 LaTeXML 生成于 2024-08-25。本译文为完整中文翻译（含全部附录），译于2026-08-10。*
