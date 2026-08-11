# EMAGE：通过表达性掩码音频-手势建模实现统一的整体性伴随语音手势生成

> **原文标题**：EMAGE: Towards Unified Holistic Co-Speech Gesture Generation via Expressive Masked Audio Gesture Modeling
> **arXiv 编号**：arXiv:2401.00374v5 [cs.CV]（CVPR 2024 Camera Ready 版本）
> **发表会议**：CVPR 2024
> **项目主页**：https://pantomatrix.github.io/EMAGE/
> **开放许可**：CC BY-NC-SA 4.0
> **本文档**：全文中文翻译（含正文、附录、全部表格与参考文献）

**作者**：Haiyang Liu¹\*、Zihao Zhu²\*、Giorgio Becherini³、Yichen Peng⁴、Mingyang Su⁵、You Zhou、Xuefei Zhe、Naoya Iwamoto、Bo Zheng、Michael J. Black³

**单位**：
¹ 东京大学（The University of Tokyo）　²庆应义塾大学（Keio University）
³ 马克斯·普朗克智能系统研究所（Max Planck Institute for Intelligent Systems）　⁴ 北陆先端科学技术大学院大学（JAIST）　⁵ 清华大学

\* 同等贡献

---

## 目录

1. [引言](#1-引言)
2. [相关工作](#2-相关工作)
3. [BEAT2 数据集](#3-beat2)
   - 3.1 [通过 MoSh++ 初始化身体参数](#31-通过-mosh-初始化身体参数)
   - 3.2 [身体参数精修](#32-身体参数精修)
   - 3.3 [从混合形变权重到 FLAME 参数](#33-从混合形变权重到-flame-参数)
4. [EMAGE 方法](#4-emage)
   - 4.1 [组合式离散面部与身体先验](#41-组合式离散面部与身体先验)
   - 4.2 [内容与节奏自注意力](#42-内容与节奏自注意力)
   - 4.3 [掩码音频条件手势建模](#43-掩码音频条件手势建模)
   - 4.4 [面部与位移解码](#44-面部与位移解码)
5. [实验](#5-实验)
   - 5.1 [数据集质量评估](#51-数据集质量评估)
   - 5.2 [模型能力评估](#52-模型能力评估)
6. [结论](#6-结论)
- [附录 A：评估指标](#附录-a评估指标)
- [附录 B：BEAT2 数据集细节](#附录-bbeat2-数据集细节)
- [附录 C：基线方法复现细节](#附录-c基线方法复现细节)
- [附录 D：EMAGE 的设置](#附录-demage-的设置)
- [附录 E：可视化 Blender 插件](#附录-e可视化-blender-插件)
- [附录 F：训练时间](#附录-f训练时间)
- [附录 G：下半身运动的重要性](#附录-g下半身运动的重要性)
- [参考文献](#参考文献)
- [译者说明](#译者说明)

---

## 摘要

我们提出 **EMAGE**，一个从**音频**与**掩码手势**生成全身人体手势的框架，涵盖**面部表情、局部身体动态、手部动作与全局位移**。为实现这一目标，我们首先引入 **BEAT2（BEAT-SMPLX-FLAME）**——一个全新的网格级（mesh-level）整体性伴随语音数据集。BEAT2 将经过 MoSh 处理的 SMPL-X 身体与 FLAME 头部参数相结合，并进一步精细化了头部、颈部与手指动作的建模，提供了一个符合学术社区标准的高质量 3D 动作捕捉数据集。

EMAGE 在训练阶段利用**掩码身体手势先验**来提升推理阶段的性能。它包含一个 **掩码音频手势Transformer（Masked Audio Gesture Transformer）**，该模块支持"音频到手势生成"与"掩码手势重建"两项任务的联合训练，从而有效编码音频与身体手势线索（body hints）。随后，从掩码手势中编码得到的身体线索被分别用于生成面部与身体动作。

此外，EMAGE 自适应地融合来自音频**节奏**与**内容**的语音特征，并利用**四个组合式 VQ-VAE** 来增强结果的保真度与多样性。实验表明，EMAGE 能够生成具有当前最优（state-of-the-art）性能的整体性手势，并且能够灵活接受预定义的时空手势输入，生成完整且与音频同步的结果。我们的代码与数据集均已公开。¹

> ¹ https://pantomatrix.github.io/EMAGE/
> † 同等贡献。

**图 1：EMAGE 总览。** 我们提出了一个掩码音频条件手势建模框架，并配套发布了一个新的整体性手势数据集 BEAT2（BEAT-SMPLX-FLAME），用于在音频以及**部分或完全被掩码的手势**的条件下，联合生成**面部表情、局部身体动态、手部动作与全局位移**。图中灰色表示可见的手势，蓝色表示我们的输出结果。

---

## 1引言

通向全身伴随语音手势生成的道路上仍存在诸多挑战。尽管已有各类基线方法被相继提出——例如针对面部 [20, 61, 15]、身体 [67, 25, 13] 和手部动作 [41, 39, 33] 的音频条件模型，以及少数尝试将它们融合的模型 [28, 65]——但**全面数据与模型的匮乏**始终是一个持续存在的障碍。

为了在这一方向上取得进展，我们提出了一个新的框架，它可以接受**部分时空预定义手势**作为输入，并自主地补全其余帧，使其与音频保持同步。这为创建逼真的数字人动画提供了一种新工具 [64, 57, 63, 71]。

为此，我们首先需要一个全面的手势数据集。虽然目前已有若干面向"音频到身体" [67]、"音频到身体与面部" [28]、"身体到手部" [41, 39] 以及通用动作 [46, 16] 的数据集，但由于不同子领域在数据格式与评估指标上存在差异，整合它们颇具挑战。这促使我们建立一个**统一的、社区标准化的基准**，用于整体性手势的训练与评估。

我们在 BEAT 数据集 [39] 的基础上进行构建，它提供了目前模态最丰富、最全面的动捕伴随语音手势数据。然而，尽管 BEAT 规模庞大、内容丰富，其数据并未以标准化格式呈现。该数据集由 iPhone ARKit 混合形变权重（blendshape weights）与 Vicon 骨架构成，在用于训练时会带来困难；例如，**缺乏网格表示**导致无法使用顶点损失（vertex loss）[52, 19]。此外，在这些多样化的模态上直接进行人类感知评估本身也非常复杂。

另一方面，SMPL-X [50] 与 FLAME [35] 是学术社区在其他人体动态相关任务（例如动作生成 [52] 与说话人头部生成 [14]）中通用的网格标准。出于促进不同任务间知识共享的动机，我们提出了 **BEAT-SMPLX-FLAME（BEAT2）**，它包含两个主要组成部分：

- **i)** 通过 MoSh++ [44, 46] 并结合硬编码物理先验得到的**精修SMPL-X 身体形状与姿态参数**；
- **ii)** 经过优化的**高质量 FLAME 头部参数**。

通过整合 SMPL-X 的身体与 FLAME 的头部，BEAT2 为伴随语音人体动画生成的多个子领域提供了一个全面的训练与评估基准。

### 表 1：伴随语音手势数据集对比

我们汇总了说话场景下的手势与面部数据集。'PGT' 与 'MC' 分别表示伪真值（Pseudo Ground Truth）与动作捕捉（Motion Capture）。BEAT2（本文）是**规模最大的动捕数据集**，提供整体性的、符合学术社区标准的网格级信息。

| 数据集 | 类型 | 年份 | 头部 | 上半身 | 手部 | 下半身 | 全局运动 | 时长（小时） |
|---|---|---|---|---|---|---|---|---|
| Trinity | Mocap | 2017 [22] | — | 3D | 3D | 3D | 3D | 4 |
| S2G | S2G-2D | 2019 [25] | 2D | 2D | 2D | — | — | 60 |
| Seq2Seq | TED-2D | 2019 [66] | — | 2D | — | — | — | 97 |
| TWH | Mocap | 2019 [32] | — | 3D | 3D | 3D | 3D | 20 |
| Trimodal | TED-3D | 2020 [67] | — | 3D | — | — | — | 97 |
| VOCA | Scan | 2020 [14] | 3D 扫描 | — | — | — | — | 0.5 |
| MeshTalk | Scan | 2021 [56] | 3D 扫描 | — | — | — | — | 2 |
| Habibie 等人 | S2G-3D | 2021 [28] | 3D | 3D | 3D | — | — | 38 |
| HA2G | TED-3D+ | 2022 [40] | — | 3D | 3D | — | — | 33 |
| BEAT | Mocap | 2022 [39] | ARKit | 3D | 3D | 3D | 3D | 76 |
| ZEEG | Mocap | 2022 [24] | — | 3D | 3D | 3D | 3D | 4 |
| Yoon 等人 | TED-SMPL | 2023 [45] | — | PGT-网格 | PGT-网格 | — | — | 30 |
| TalkShow | S2G-SMPL | 2023 [65] | PGT-网格 | PGT-网格 | PGT-网格 | — | — | 27 |
| **BEAT2（本文）** | **本文** | **2024** | **MC-网格** | **MC-网格** | **MC-网格** | **MC-网格** | **MC-网格** | **60** |

### 表 2：伴随语音手势模型对比

我们与此前在伴随语音数据集上训练、用于生成面部或身体动作的方法进行比较。第一行列出它们的**输入**，后续各行分别列出它们的**输出**。不同的解码器设计用首字母表示：'S' 表示单一（Single），'M' 表示多个（Multiple），'C' 表示级联（Cascaded）。据我们所知，EMAGE（本文）是**首个**能够接受音频与部分或完全掩码手势、并生成全身音频同步结果的方法。

| 方法 | 输入手势 | 头部 | 上半身 | 手部 | 下半身 + 全局 |
|---|---|---|---|---|---|
| S2G 2019 [25] | — | — | S | — | — |
| TriModal 2020 [67] | — | — | S | — | — |
| B2H 2020 [48] | 身体 | — | — | S | — |
| TWH 2021 [32] | 身体 | — | — | S | — |
| Habibie 等人 2021 [28] | — | M | M | M | — |
| DisCo 2022 [38] | — | — | S | S | — |
| HA2G 2022 [40] | — | — | C | C | — |
| FaceFormer 2022 [19] | — | S | — | — | — |
| Rhythmic 2022 [5] | — | — | S | — | — |
| BEAT CaMN 2022 [39] | — | — | C | C | — |
| DiffGesture 2023 [72] | — | — | S | S | — |
| CodeTalker 2023 [61] | — | S | — | — | — |
| TalkShow 2023 [65] | — | S | C | C | — |
| DiffStyleGesture 2023 [62] | — | — | S | S | S |
| BodyFormer 2023 [49] | — | — | S | — | — |
| LivelySpeaker 2023 [70] | — | — | S | S | — |
| **EMAGE（本文）2024** | **掩码** | **C** | **C** | **C** | **C** |

**图 2：BEAT2 与其他数据集的数据对比。** BEAT-SMPLX-FLAME 提供了一个全新的、网格级、动作捕捉的整体性伴随语音手势数据集，共60 小时数据。**左**：我们将精修后的 SMPL-X 身体参数（记为 Refined MoSh）与原始 BEAT 骨架 [39]、通过 AutoRegPro 重定向的结果，以及 MoSh++ [50] 的初始结果进行对比。精修结果展现出正确的颈部弯曲、合理的头颈形状比例，以及细致的手指表示。**右**：对原始 BEAT 数据集 [39] 的混合形变权重进行可视化，分别使用 ARKit 模板、基于包裹（Wrap-based）以及手工优化三种方式。我们最终采用的手工制作 FLAME 混合形变优化方案，同时展现出准确的唇部动作细节与自然的口型。

借助 BEAT2 的整体性数据，我们希望在**增强各身体部位之间连贯性**的同时，确保**音频与动作之间准确的跨模态对齐**（见图 1）。这引出了 **表达性掩码音频条件手势建模（Expressive Masked Audio-conditioned GEsture modeling，EMAGE）** 的设计——一个基于空间与时间 Transformer 的框架。

EMAGE 首先从被掩码的身体关节中聚合空间特征；随后，它借助一个**可切换的手势时序自注意力**与**音频-手势交叉注意力**，重建预训练手势的隐空间。不同前向路径的选择使得模型能够分别、有效地建模"手势到手势"与"音频到手势"这两种先验。在获得重建的隐特征后，EMAGE 使用**四个组合式预训练向量量化变分自编码器（VQ-VAE）** 解码局部面部与身体手势，并使用一个预训练的**全局运动预测器（Global Motion Predictor）** 解码全局位移。组合式 VQ-VAE 是在重建过程中保留音频相关运动的关键。

通过这些设计，EMAGE 在生成身体与面部手势方面均达到了当前最优性能。它以音频和部分初始化的手势作为输入，恢复出与音频同步、连贯的手势。此外，我们还展示了 EMAGE 如何灵活地引入额外的**非整体性**数据集来改善结果。

**本文贡献总结如下：**

1.我们发布 **BEAT2**，一个表示统一的、网格级数据集，采用经MoSh 处理的 SMPL-X 身体与 FLAME 头部参数。
2. 我们提出 **EMAGE**，一个简洁而有效的整体性手势生成框架，能够在部分手势与音频先验的条件下生成连贯的手势。
3. EMAGE 仅使用**四帧种子手势**，即可在身体与面部手势生成上均达到当前最优性能。
4. EMAGE 展示了如何利用额外的非整体性手势数据集（例如 Trinity 与 AMASS）进行训练，从而进一步提升结果的保真度与多样性，有效地整合来自不同数据集的数据。

---

## 2 相关工作

### 伴随语音动画数据集

可分为两类（见表 1）：**伪标注（PGT）** 与 **动作捕捉（mocap）**。

对于 PGT 类，Speech2Gesture 数据集 [25] 利用 OpenPose [11] 从新闻与教学视频中提取 2D 姿态。后续工作将该数据集扩展至 3D 姿态 [28] 与 SMPL-X [65]。TED 数据集 [66] 从 TED 演讲视频中提取 2D 姿态，之后被扩展以包含 3D 姿态 [67]、手指 [40] 与 SMPL-X [45]。类似地，2D、3D 关键点与网格也可从多视角录制视频 [60] 中估计得到，作为说话人头部生成的 PGT。

尽管伪标注方法理论上可以提取无限量的数据，其精度仍受限。例如，当前最优的单目 3D 姿态估计算法 [23] 在 Human3.6M 数据集 [30] 上的平均误差为 33.4 mm，而 Vicon 动捕的误差仅为 0.142 mm [47]。

对于动捕数据集，Trinity [3] 包含一位男性演员共 4 小时数据；TalkingWithHands [32] 采集了两位说话者的对话场景；ZEEG [24] 考虑了一位说话者的 12 种风格。对于面部，3D 扫描虽然精确，但因成本高昂而数量有限，例如 BIWI [21]、VOCA [14] 与 MeshTalk [56] 均不足 3 小时。这导致 ARKit 数据集 [51] 在性能与数量之间取得了一种折中。

上述数据集都是分别处理身体与面部。相比之下，BEAT [39] **首次**同时包含了 3D 身体姿态与 ARKit [8] 面部混合形变权重。然而，BEAT 缺少网格数据。

### 伴随语音动画模型

依据输出内容分类（见表 2）。除了各类基线方法与数据集 [25, 67, 14, 56, 32, 48] 之外，还有若干针对身体手势的框架 [53, 54, 2, 33] 提升了基线性能。它们通过选取特定的上半身关节 [38, 39, 5, 4, 40, 70, 49] 或全部身体关节 [72, 62] 进行训练与评估。近期在面部手势方面的改进则利用 Transformer 与离散面部先验来驱动顶点 [19, 61]。

然而，这些方法只处理面部**或**身体。将这些方法直接应用于全身会产生次优结果，因为音频与面部、身体动态的关联方式并不相同。

与我们工作最相似的是：Habibie 等人 [28] 采用单一音频编码器与多个解码器来生成面部与身体手势。TalkSHOW [65] 展示了分离音频编码器、并以自回归方式使用量化编码解码身体与手部的优势。但它缺少下半身与全局运动，且身体与面部手势生成之间没有共享信息。此外，由于其完全自回归模型的设计，它无法接受部分掩码的身体线索。

### 掩码表示学习

最早在自然语言处理领域被证明有效——基于 BERT 的模型 [17, 31, 42] 通过将掩码语言建模与 Transformer 架构相结合，提升了学习到的词嵌入在下游任务中的表现。随后，掩码自编码器（Masked AutoEncoders）[29] 通过移除并修补图像块，将掩码图像建模扩展到计算机视觉领域。这一掩码表示学习的思想此后被应用于其他模态，例如视频 [18, 12, 43]、音频 [26, 36, 37] 与点云 [69, 68, 55]。

与我们工作最相关的是 MotionBERT [73]，它提出一个时空 Transformer，通过掩码 2D 姿态来学习用于分类任务的鲁棒运动表示。与其方法不同，我们的目标是为**条件运动生成任务**获得鲁棒的运动特征，这需要在多个模态（例如音频与手势）之间的训练中取得平衡。

---

## 3 BEAT2

本节介绍我们如何从原始 BEAT 数据集 [39] 获得统一的网格级数据，即 SMPL-X [50] 与 FLAME [35] 参数。

BEAT 使用 Vicon 动作捕捉系统 [47]，并发布了 78 关节的骨架级 Biovision Hierarchy（BVH）[10] 动作文件。其面部捕捉系统使用 iPhone 12 Pro 上带深度相机的 ARKit [8]，提取 $\mathbb{R}^{51}$ 维混合形变权重。这些混合形变是基于被广泛采用的**面部动作编码系统（FACS）** 设计的。

然而，动作数据与面部数据都缺乏网格级细节（见图 2），例如形状与顶点信息。

### 3.1 通过 MoSh++ 初始化身体参数

我们使用 MoSh++ [46] 从 BEAT 动捕标记点数据初始化 SMPL-X 身体形状与姿态参数。

给定捕获的标记点位置 $\mathbf{m}\in\mathbb{R}^{T\times K\times 3}$、预定义的标记点位置偏移 $\mathbf{d}\in\mathbb{R}^{K\times 3}$，以及用户定义的顶点到标记点映射函数 $\mathcal{H}$，我们的目标是求解身体形状 $\beta\in\mathbb{R}^{300}$、姿态 $\theta\in\mathbb{R}^{T\times 55\times 3}$ 与位移参数 $\gamma\in\mathbb{R}^{T\times 3}$。

优化过程使用可微的表面顶点映射函数 $\mathcal{S}(\beta,\theta,\gamma)$ 与顶点法向函数 $\mathcal{N}(\beta,\theta_{0},\gamma_{0})$。对于每一帧，隐标记点 $\tilde{\mathbf{m}}\in\mathbb{R}^{T\times K\times 3}$ 的计算方式为：

$$\tilde{\mathbf{m}}_{i}\equiv\mathcal{S}_{\mathcal{H}}(\beta,\theta_{i},\gamma_{i})+\mathbf{d}\,\mathcal{N}_{\mathcal{H}}(\beta,\theta_{i},\gamma_{i}) \tag{1}$$

我们首先选取 12 帧来优化并固定 $\mathbf{d}$ 与 $\beta$，然后基于 $\|\tilde{\mathbf{m}}_{i}-\mathbf{m}_{i}\|^{2}$ 的各项损失，对 $i\in(1:T)$ 优化 $\theta_{i}$、$\gamma_{i}$。这些损失包括**数据项（Data Term）、表面距离能量（Surface Distance Energy）、标记点初始化正则项（Marker Initialization Regularization）、姿态与形状先验（Pose and Shape Priors）、速度恒定项（Velocity Constancy）** 以及**软组织项（Soft-Tissue Term）**（见补充材料）。总体目标函数是这些项的加权和，以在精度与合理性之间取得平衡。

### 3.2 身体参数精修

由于头部标记点是佩戴在头盔上的，MoSh++ 会产生不自然的头部形状，有时也会产生不自然的手指姿态。因此，我们用三条简洁而有效的**硬编码物理规则**来精修身体形状与姿态参数：

1. 颈部与头部长度应约为身体总长度的 $1/7$；
2. 除拇指外，手指不应向后弯曲；
3. 我们使用 Kolmogorov-Smirnov 检验，结果表明数据近似服从正态分布。随后我们采用高斯截断方法：所有落在 $3\sigma$ 范围之外的数据点都被调整到$3\sigma$ 阈值，并与相邻 10 帧进行混合。

我们在图 2 中将精修后的身体参数（记为 MoSh Refined）与原始 BEAT、重定向 SMPL-X 以及 MoSh SMPL-X 进行了对比。

**图 3：** EMAGE 采用两条训练路径：**掩码手势重建（MG2G）** 与 **音频条件手势生成（A2G）**。MG2G 路径侧重于通过一个时空 Transformer 手势编码器与一个交叉注意力手势解码器来编码鲁棒的身体线索。相比之下，A2G 路径利用这些身体线索以及分离的音频编码器，来解码预训练的面部与身体隐特征。该过程中的一个关键组件是**可切换交叉注意力层**，它对于融合身体线索与音频特征至关重要。这种融合使特征得以有效解耦并用于手势解码。一旦手势隐特征被重建，EMAGE 便使用预训练的 VQ 解码器来解码面部与局部身体运动。此外，还使用一个预训练的全局运动预测器来估计全局身体位移，进一步增强模型生成逼真且连贯手势的能力。

**图 4：CRA 与预训练 VQ-VAE 的细节。** **左**：内容-节奏注意力（Content Rhythm Attention）自适应地融合语音节奏（起始点 onset 与振幅 amplitude）与内容（来自文本脚本的预训练词嵌入）。这使得模型在特定帧上会偏向内容或节奏，从而鼓励生成具有语义感知的手势。**右**：我们通过分别重建面部、上半身、手部与下半身，预训练四个组合式 VQ-VAE，以显式解耦与音频无关的手势。

### 3.3 从混合形变权重到 FLAME 参数

给定 ARKit 混合形变权重 $\mathbf{b}_{\text{ARKit}}\in\mathbb{R}^{T\times 51}$，我们希望求得一个变换矩阵 $\mathbf{W}\in\mathbb{R}^{51\times 103}$，将其映射为 FLAME 参数 $\mathbf{b}_{\text{FLAME}}\in\mathbb{R}^{T\times(100+3)}$，其中 100 表示表情参数的维数，3 表示下颌运动。

由于 iPhone 与 FLAME 的模板网格拓扑存在差异，仅通过最小化 ARKit 模板顶点与包裹后的 FLAME 顶点之间的几何误差来优化 FLAME 参数无法得到令人满意的结果。为此，我们按照 ARKit 的 FACS 配置，在 FLAME 上发布了一套**手工制作的混合形变模板** $\mathbf{v}_{\text{template}}\in\mathbb{R}^{52}$。该方法允许直接由给定的混合形变权重驱动 FLAME 拓扑顶点 $\mathbf{v}\in\mathbb{R}^{T\times 5023\times 3}$：

$$\mathbf{v}=\mathbf{v}_{\text{template}}^{0}+\sum_{j=1}^{51}\mathbf{b}_{\text{ARKit},j}\cdot\mathbf{v}_{\text{template}}^{j} \tag{2}$$

其中 $\mathbf{b}_{\text{ARKit},j}$ 是第 $j$ 个 ARKit 混合形变的权重，$\mathbf{v}_{\text{template}}^{j}$ 是FLAME 模型模板中对应第 $j$ 个混合形变的顶点位置，$\mathbf{v}_{\text{template}}^{0}$ 表示 FLAME 模型原始模板的顶点位置。

我们通过最小化 $\|\tilde{\mathbf{v}}_{j}-\mathbf{v}_{j}\|_{2}$ 来优化 $\mathbf{W}$，其中 $\tilde{\mathbf{v}}$ 由 FLAME 的线性混合蒙皮 $\mathcal{V}(\mathbf{b}_{\text{FLAME}})$ 得到。ARKit 数据、基于包裹的方法与我们的方法之间的对比见图 2。

---

## 4 EMAGE

我们介绍 EMAGE 的细节，即 **E**xpressive **M**asked **A**udio-Conditioned **GE**sture Modeling（表达性掩码音频条件手势建模，见图 3）。

给定手势 $\mathbf{g}\in\mathbb{R}^{T\times(55\times 6+100+4+3)}$——分别表示 55 个关节的 Rot6D 旋转、$\mathbb{R}^{100}$ 维 FLAME 参数、$\mathbb{R}^{4}$ 维足部接触标签与 $\mathbb{R}^{3}$ 维全局位移——以及语音音频 $\mathbf{a}\in\mathbb{R}^{T\times sk}$，其中 $sk=sr_{\text{audio}}/fps_{\text{gestures}}$，EMAGE 联合优化**掩码手势重建**与**音频条件手势生成**。这一优化提升了推理阶段的性能，并使得系统能够利用部分掩码的手势来补全整体性手势。

为此，我们首先在 4.1 节中建模量化隐空间（参照 [65, 61]）；接着在 4.2 节中设计一个使用**内容节奏自注意力（CRA）** 的分离式语音音频编码器；随后在 4.3 节中通过掩码音频手势 Transformer 从掩码手势中学习身体线索；最后在 4.4 节中讨论针对不同身体部位的解码策略。

### 4.1 组合式离散面部与身体先验

我们在**分离的量化隐空间**中建模全身手势（图 4），原因如下。与 [65] 类似，将身体与手部分离可以提升结果的多样性。但我们**额外**分离了面部与下半身，因为它们与音频的相关性不同——即，使用单一VQ-VAE 同时编码上半身与下半身，可能导致模型忽略出现频率较低的手势，而无论这些手势是否与音频相关。具体而言，对于一位在对话中不断走动的说话者，单一模型可能只专注于恢复下半身运动，而忽略了肘部动作。

面部 $\mathbf{g}_{\text{f}}\in\mathbb{R}^{T\times 106}$、上半身 $\mathbf{g}_{\text{u}}\in\mathbb{R}^{T\times 78}$、手部 $\mathbf{g}_{\text{h}}\in\mathbb{R}^{T\times 180}$ 与下半身 $\mathbf{g}_{\text{l}}\in\mathbb{R}^{T\times(54+4)}$ 所对应的分离量化隐空间 $Q=\{\mathbf{q}_{\text{f}},\mathbf{q}_{u},\mathbf{q}_{\text{h}},\mathbf{q}_{\text{l}}\}$ 分别来自四个 VQ-VAE。

每个 VQ-VAE 通过联合优化以下损失项进行训练：

$$q_{i}=\operatorname*{arg\,min}_{q_{i}\in\mathbf{q}}\|z_{j}-q_{i}\|^{2} \tag{3}$$

$$\mathcal{L}_{\text{VQ-VAE}}=\mathcal{L}_{rec}(\mathbf{g},\hat{\mathbf{g}})+\mathcal{L}_{\text{vel}}(\mathbf{g}^{\prime},\hat{\mathbf{g}}^{\prime})+\mathcal{L}_{\text{acc}}(\mathbf{g}^{\prime\prime},\hat{\mathbf{g}}^{\prime\prime}) + \|\text{sg}[\mathbf{z}]-\mathbf{q}\|^{2}+\|\mathbf{z}-\text{sg}[\mathbf{q}]\|^{2} \tag{4}$$

其中 $\mathbf{z}$ 是以时间窗口大小 $w=1$ 编码得到的 $\mathbf{g}$。$\mathcal{L}_{\text{rec}}$、$\mathcal{L}_{\text{vel}}$ 与 $\mathcal{L}_{\text{acc}}$ 分别为测地线损失 [58] 与 L1 损失，sg 表示停止梯度（stop gradient）操作。本文中我们将承诺损失（commitment loss）[59]（最后一项）的权重设为 1。

### 4.2 内容与节奏自注意力

给定语音音频 $\mathbf{s}$，受 [4] 启发，我们采用起始点 $\mathbf{o}$ 与振幅 $\mathbf{a}$ 作为显式的音频节奏，同时使用来自转录文本的预训练嵌入 [9] $\mathbf{e}$ 作为内容。

与以往通常直接**相加**节奏与内容特征的做法不同，我们利用**自注意力**自适应地融合这些特征。这一做法源于以下观察：在特定帧上，手势可能更多地与内容（语义感知）或节奏（节拍感知）相关。

节奏与内容特征首先分别通过时序卷积网络（TCN）与线性映射，编码为时间对齐的特征 $\mathbf{r}_{1:T}$ 与 $\mathbf{c}_{1:T}$。对于每个时间步 $t\in\{1,\dots,T\}$，我们按如下方式融合节奏与内容特征：

$$\mathbf{f}_{1:T}=\alpha\times\mathbf{r}_{1:T}+(1-\alpha)\times\mathbf{c}_{1:T} \tag{5}$$

$$\alpha=\text{Softmax}(\mathcal{AT}(\mathbf{r}_{1:T},\mathbf{c}_{1:T}))$$

其中 $\mathcal{AT}$ 是一个 2 层 MLP。我们为面部与身体分别应用两个独立的 CRA 编码器。

**图 5：前向路径设计对比。** 直接融合模块 (a) 在没有精修身体特征的情况下融合音频特征，并且仅基于位置嵌入重新组合音频特征。自注意力解码器模块 (b) 被此前的掩码语言模型（MLM）[17, 31] 所采用，但在需要自回归推理的任务上受到限制。我们的设计 (c) 同时兼顾了有效的音频特征融合与自回归解码。

### 4.3 掩码音频条件手势建模

我们提出一个**掩码音频手势 Transformer** 来利用不同的训练路径（架构设计的动机见图 5）。

给定时间与空间上被掩码的手势 $\overline{\mathbf{g}}\in\mathbb{R}^{T\times 337}$，我们首先将被掩码的 token 替换为可学习的掩码嵌入 $e_{\text{mask}}\in\mathbb{R}^{256}$，因为数值零仍然代表特定的运动内容（例如 T-pose）。我们根据训练轮次，将被掩码关节与帧的比例从 0 线性提升到 95%。

一个空间卷积编码器 $\mathcal{SC}$ 首先汇总空间信息，并将空间特征压缩至 $\mathbb{R}^{T\times 256}$。随后我们采用一个（不含前馈层的）时序自注意力 $\mathcal{TSA}$ 来精修汇总后的空间特征：

$$\mathbf{h}=\mathcal{TSA}(\mathcal{SC}(\overline{\mathbf{g}})+\mathbf{p}_{t}) \tag{6}$$

其中 $\mathbf{h}\in\mathbb{R}^{T\times 512}$ 表示编码得到的**身体线索**，$\mathbf{p}_{t}$ 是可学习说话者嵌入与 PPE [19] 之和。

接着，采用一个直接的时序交叉注意力 Transformer 解码器 $\mathcal{TCAT}$ 得到重建的隐变量 $\hat{\mathbf{q}}_{\text{mg2g}}$：

$$\hat{\mathbf{q}}_{\text{mg2g}}=\mathcal{TCAT}(\mathbf{h}+\mathbf{p}_{t}) \tag{7}$$

我们在隐空间中最小化 L1 距离：

$$\mathcal{L}_{\text{mg2g}}=\|\hat{\mathbf{q}}_{\text{mg2g}}-\mathbf{q}\| \tag{8}$$

掩码手势重建编码出有效的身体线索，而关键在于如何利用这些身体线索进行手势生成。我们通过一个时序交叉注意力 $\mathcal{TCA}$ 对音频与身体线索进行**选择性融合**。随后，我们使用融合后的音频-手势特征进行音频条件手势隐变量重建：

$$\hat{\mathbf{q}}_{\text{a2g}}=\mathcal{TCAT}(\mathcal{TCA}(\mathbf{h}+\mathbf{p}_{t},\mathbf{f}_{\text{body}}),\overline{\mathbf{g}}+\mathbf{p}_{t}) \tag{9}$$

我们同时优化隐编码类别分类交叉熵损失 $\mathcal{L}_{\text{a2g-rec}}$ 与隐变量重建损失 $\mathcal{L}_{\text{a2g-cls}}$，以鼓励结果的多样性。

### 4.4 面部与位移解码

考虑到面部与身体运动的关联较弱，对其施加相同的操作——即基于身体线索重新组合音频特征——并不合理。因此，我们**直接将掩码身体线索与音频特征拼接**，用于面部隐变量的最终解码：

$$\hat{\mathbf{q}}_{\text{f}}=\mathcal{TCAT}(\mathbf{f}_{\text{face}}\oplus\mathbf{h},\mathbf{p}_{t}) \tag{10}$$

一旦从 VQ 解码器获得局部下半身运动 $\tilde{\mathbf{g}}_{l}\in\mathbb{R}^{T\times(54+4)}$，我们便用一个预训练的**全局运动预测器**估计全局位移 $\tilde{\mathbf{t}}\in\mathbb{R}^{T\times 3}$，即 $\tilde{\mathbf{t}}=\mathcal{G}(\tilde{\mathbf{g}}_{l})$，这显著减少了足部滑动（foot sliding）现象。

### 表 3：现有数据集之间的用户偏好胜率

'Top.' 表示网格的拓扑结构。结果显示，我们的 BEAT2 数据集在身体与面部两个方面均优于现有的 PGT 数据集 [65]。与此前的动捕数据集 [46] 和基于 3D 扫描的数据集 [14] 相比，它在身体方面略优，在面部方面略逊。

| 数据集 | 身体拓扑 | 面部拓扑 | 身体（%） | 面部（%） |
|---|---|---|---|---|
| VOCA [14] | — | FLAME | — | 38.3 ± 5.63 |
| AMASS [46] | SMPL-X | FLAME | 42.0 ± 3.60 | — |
| TalkSHOW [65] | SMPL-X | SMPL-X | 14.4 ± 2.19 | 26.1 ± 6.42 |
| **BEAT2（本文）** | SMPL-X | FLAME | **43.6 ± 3.38** | 35.7 ± 5.91 |

**图 6：主观示例。** 自上而下：真值（GroundTruth）、**无身体线索**的生成结果、**带身体线索**的生成结果、可见的身体线索。EMAGE 生成多样、具语义感知且与音频同步的身体手势，例如在说到 "spare time"（空闲时间）时举起双手，说到 "hike in nature"（在自然中徒步）时做出放松的手势。此外，如第三行与最后一行所示，EMAGE 能够灵活地在任意帧与任意关节上接受非音频同步的身体线索，以显式地引导或定制生成的手势，例如重复一个类似的举手动作、改变行走朝向等。注意，图中生成结果的灰色关节**并非**可见线索的直接复制。

---

## 5 实验

我们将评估分为两类：**数据集质量**与**模型能力**。

在从 BEAT 中移除手指质量较差的数据后，BEAT2 缩减为 60 小时。我们进一步根据演讲与对话章节的类型 [39]，将其划分为 **BEAT2-Standard（27 小时）** 与 **BEAT2-Additional（33 小时）**。演讲章节中的表演性手势更加多样且富有表现力，而对话章节中的自发手势则更为自然但变化较少。我们在 BEAT2-Standard 的Speaker-2 上，按 85%/7.5%/7.5% 的训练/验证/测试划分报告结果。

### 5.1 数据集质量评估

我们将本文数据集与当前最优的伪真值（PGT）数据集 TalkSHOW [65] 在面部与身体两方面进行比较。此外，我们还以 AMASS [46]（身体）与 VOCA [14]（面部）作为参考进行对比。

由于各数据集中序列长度不一，我们采样了 100 对比较样本，每对时长相同，介于 2 至 4 秒之间。在感知实验中，每位参与者在一次 10 分钟的会话中评估随机的 40 对样本，选出其认为捕捉质量最佳的序列。总共邀请了 60 位参与者。需要注意的是，参与者被要求**仅比较上半身结果**，因为 TalkSHOW 只包含上半身。结果见表 3。

### 5.2 模型能力评估

我们分别在 BEATv1.3 与 BEATv2 上报告结果。后者的面部数据经过动画师精修，手部数据经过标注员筛选。

#### 表 4：在 BEATv2 上的定量评估

我们报告 FGD $\times 10^{-1}$、BC $\times 10^{-1}$、Diversity、MSE $\times 10^{-8}$ 与 LVD $\times 10^{-5}$。对于身体手势，EMAGE 显著改善了 FGD，说明生成结果更接近真值。这体现了来自掩码手势建模的身体线索的有效性。

| 方法 | FGD ↓ | BC ↑ | Diversity ↑ | MSE ↓ | LVD ↓ |
|---|---|---|---|---|---|
| FaceFormer [19] | — | — | — | 7.787 | 7.593 |
| CodeTalker [61] | — | — | — | 8.026 | 7.766 |
| S2G [25] | 28.15 | 4.683 | 5.971 | — | — |
| Trimodal [67] | 12.41 | 5.933 | 7.724 | — | — |
| HA2G [40] | 12.32 | 6.779 | 8.626 | — | — |
| DisCo [38] | 9.417 | 6.439 | 9.912 | — | — |
| CaMN [39] | 6.644 | 6.769 | 10.86 | — | — |
| DiffStyleGesture [62] | 8.811 | 7.241 | 11.49 | — | — |
| Habibie等人 [28] | 9.040 | 7.716 | 8.213 | 8.614 | 8.043 |
| TalkSHOW [65] | 6.209 | 6.947 | **13.47** | 7.791 | 7.771 |
| **EMAGE（本文）** | **5.512** | 7.724 | 13.06 | **7.680** | **7.556** |

#### 表 5：生成结果的用户偏好胜率

结果表明，我们生成的结果被认为更加真实可信，在身体与面部手势上的用户偏好分别高出 14% 与 23%。

| 方法 | 整体（%） | 身体（%） | 面部（%） |
|---|---|---|---|
| Habibie 等人 [28] | 12.4 ± 3.70 | 15.9 ± 6.49 | 10.8 ± 3.19 |
| TalkSHOW [65] | 34.9 ± 5.79 | 40.4 ± 8.22 | 33.2 ± 6.03 |
| **EMAGE（本文）** | **52.7 ± 7.91** | **44.7 ± 8.68** | **56.0 ± 7.80** |

**评估指标。** 我们采用 FGD [67] 评估身体手势的真实感；通过计算多个身体手势片段之间的平均 L1 距离来度量 Diversity（多样性）[33]；使用 BC [34] 评估语音-动作同步性。对于面部，我们计算顶点 MSE [61] 来度量位置距离，并计算真值与生成面部顶点之间的顶点 L1 差异 LVD [65]。

**对比方法。** 我们首先通过复现的方式，分别在身体与面部上，与身体手势生成 [25, 67, 40, 38, 39, 62] 与说话人头部生成 [19, 61] 领域中具有代表性的当前最优方法进行比较。此外，我们还复现了两个此前最优的整体性流程——Habibie 等人 [28] 与 TalkSHOW [65]，它们的原始实现仅限于上半身。我们为 Habibie 等人的方法添加了一个下半身解码器，为 TalkSHOW 添加了一个下半身 VQ-VAE。

#### 5.2.1 定量与定性分析

如表 4 所示，在仅使用四帧种子姿态的情况下，我们的方法优于此前的最优算法。定性结果见图 6 与图 7。

此外，我们还进行了一项感知研究。在保持 60 位参与者的相同设置下，每位参与者评估 40 对 10 秒长的结果，判断哪一个更可信，由此得到表 5 中的胜率。

**图 7：生成面部动作的对比。** 与此前最优的说话人脸生成方法 FaceFormer [19]、CodeTalker [61]，以及整体性手势生成方法 Habibie 等人 [28]、TalkSHOW [65] 进行对比。注意，CodeTalker 在 BEATv2 上的 MSE 高于 EMAGE（表 4，越低越好），但主观上仍显得真实。EMAGE 通过同时利用面部模型与掩码身体线索，获得了良好的唇部动作。

#### 5.2.2 消融分析

**基线性能。** 如表 6 所示，我们从一个基于教师强制（teacher-force）的 Transformer 基线开始。该基线受 FaceFormer [19] 启发，采用 1 层交叉注意力 Transformer 解码器，并将来自 Wav2Vec2 [6] 的音频特征替换为我们自定义的 TCN [7] 与可训练词嵌入 [9]。

**多个 VQ-VAE 的作用。** 简单地对包括面部在内的全身运动使用单个 VQ-VAE [59, 27] 会降低面部动作的性能。这是因为 VQ-VAE 的训练目标是最小化全身的平均损失，而某些说话者最频繁的动作与音频无关。采用分离的 VQ-VAE 使模型能更好地发挥离散先验的优势。

**内容节奏自注意力的作用。** 自适应融合节奏与内容特征在 FGD 与对齐性两方面都带来了提升。它根据训练数据的分布，选择性地让当前动作更多依赖节奏或内容特征。此外，我们观察到应用 CRA 后能得到更具语义感知的结果。

**掩码身体手势线索的作用。** 所有客观指标的提升表明，我们的模型有效利用了时空手势先验，降低了采样到错误手势的可能性。更重要的是，掩码手势建模是使网络能够在特定帧接受预定义手势的关键。

#### 表 6：在 BEATv1.3 上的消融分析

| 配置 | FGD ↓ | BC ↑ | Diversity ↑ | MSE ↓ | LVD ↓ |
|---|---|---|---|---|---|
| 真值（Ground Truth） | 0 | 6.896 | 13.074 | 0 | 0 |
| 重建（Reconstruction） | 3.913 | 6.758 | 13.145 | 0.841 | 6.389 |
| 基线（Baseline） | 13.080 | 6.941 | 8.3145 | 1.442 | 9.317 |
| + VQVAE | 9.787 | 6.673 | 10.624 | 1.619 | 9.473 |
| + 4 个 VQVAE | 7.397 | 6.698 | 12.544 | 1.243 | 8.938 |
| + CRA | 6.833 | 6.783 | 12.676 | 1.186 | 8.715 |
| + 掩码线索（Masked Hints） | **5.423** | 6.794 | **13.057** | **1.180** | 9.015 |

#### 表 7：在多个数据集上训练 EMAGE

EMAGE 展现出灵活性：即使只有部分整体性手势可用，它也能在多个数据集上训练。这一做法进一步改善了 BEATv1.3 测试集上的客观指标。

| 训练数据 | 身体 | 手部 | 面部 | FGD ↓ | BC ↑ | Diversity ↑ |
|---|---|---|---|---|---|---|
| BEATX | 有 | 有 | 有 | 5.423 | 6.794 | 13.057 |
| + Trinity [22] | 有 | 无 | 无 | 5.319 | **6.843** | 13.346 |
| + AMASS [46] | 有 | 有 | 无 | **5.174** | 6.769 | **14.318** |

#### 5.2.3 多数据集训练能力

我们证明了 EMAGE 能够有效整合多个非整体性数据集进行训练：通过与 Trinity [22] 和 AMASS [46] 数据集联合训练，其中只使用 Trinity 的上半身与音频配对数据，以及 AMASS 的身体与手部数据。我们为 BEAT2、Trinity 与 AMASS 分别训练独立的 VQ-VAE，并为码本分类实现独立的 MLP 头。表 7 的结果表明，引入额外数据提升了在 BEATv1.3 测试集上的性能。

---

## 6 结论

在本工作中，我们提出了 EMAGE——一个能够接受部分手势作为输入、以补全音频同步的整体性手势的框架。它表明，利用掩码手势重建可以显著提升音频条件手势生成的性能。此外，EMAGE 的设计使其能够在多个数据集上训练，从而进一步提升性能。

与 EMAGE 一同，我们发布了 BEAT2——目前规模最大的、与 SMPL-X 和 FLAME 保持一致的多模态手势数据集。我们希望 BEAT2 能促进各子领域之间的知识与模型共享。

**声明：** You Zhou、Xuefei Zhe、Naoya Iwamoto 与 Bo Zheng 是华为东京研究中心的员工，本工作是在其雇主批准的情况下利用个人时间完成的。

MJB 利益冲突声明：https://files.is.tue.mpg.de/black/CoI_CVPR_2024.txt

---

# 补充材料

本补充文档包含七个部分：

- 评估指标（附录 A）
- BEAT2 数据集细节（附录 B）
- 基线方法复现细节（附录 C）
- EMAGE 的设置（附录 D）
- 可视化 Blender 插件（附录 E）
- 训练时间（附录 F）
- 下半身运动的重要性（附录 G）

## 附录 A：评估指标

### Fréchet 手势距离（Fréchet Gesture Distance, FGD）

FGD 越低（参照 [67]）表示真值与生成身体手势之间的分布越接近。与图像生成任务中使用的感知损失类似，FGD 基于预训练网络提取的隐特征计算：

$$\operatorname{FGD}(\mathbf{g},\hat{\mathbf{g}})=\left\|\mu_{r}-\mu_{g}\right\|^{2}+\operatorname{Tr}\left(\Sigma_{r}+\Sigma_{g}-2\left(\Sigma_{r}\Sigma_{g}\right)^{1/2}\right) \tag{11}$$

其中 $\mu_{r}$ 与 $\Sigma_{r}$ 表示真实人体手势 $\mathbf{g}$ 的隐特征分布 $z_{r}$ 的一阶与二阶矩，$\mu_{g}$ 与 $\Sigma_{g}$ 表示生成手势 $\hat{\mathbf{g}}$ 的隐特征分布 $z_{g}$ 的一阶与二阶矩。

我们使用基于骨架 CNN（SKCNN）的编码器 [1] 与基于全卷积的解码器作为预训练自编码器网络。该网络在 BEAT2-Standard 与 BEAT2-Additional 数据上共同预训练。选择 SKCNN 而非全卷积编码器，是因为其在捕捉手势特征方面能力更强，重建MSE 损失更低（0.095 对比 0.103）。

### L1 多样性（L1 Diversity）

Diversity 越高[33] 表示给定手势片段中的方差越大。我们按如下方式计算来自不同 $N$ 个动作片段的平均 L1 距离：

$$\text{L1 div.}=\frac{1}{2N(N-1)}\sum_{t=1}^{N}\sum_{j=1}^{N}\left\|p_{t}^{i}-\hat{p}_{t}^{j}\right\|_{1} \tag{12}$$

其中 $p_{t}$ 表示第 $t$ 帧中关节的位置。我们在整个测试数据集上计算多样性。此外，为计算关节位置，位移被置零，这意味着 L1 多样性仅关注局部运动。

### 节拍一致性（Beat Constancy, BC）

BC 越高（定义见 [34]）表示手势节奏与音频节拍的对齐程度越高。我们将语音的起始点视为音频节拍，并将上半身关节（不含手指）速度的局部极小值视为动作节拍。音频与手势之间的同步性按如下方式计算：

$$\text{BC}=\frac{1}{g}\sum_{b_{g}\in g}\exp\left(-\frac{\min_{b_{a}\in a}\left\|b_{g}-b_{a}\right\|^{2}}{2\sigma^{2}}\right) \tag{13}$$

其中 $g$ 与 $a$ 分别表示手势节拍与音频节拍的集合。

## 附录 B：BEAT2 数据集细节

### 统计信息

原始 BEAT 数据集 [39] 包含 30 位说话者共 76 小时数据。我们排除了说话者 8、14、19、23 与 29（共占 16 小时数据），因为其手指数据存在噪声，最终保留 25 位说话者（12 位女性、13 位男性）共 60 小时数据。

演讲与对话部分被划分为 BEAT2-standard 与 BEAT2-additional，分别包含 27 与 33 小时。我们对 BEAT2-standard 采用 85%、7.5%、7.5% 的划分，并对每位说话者保持相同比例。BEAT2-additional 用于进一步提升网络的鲁棒性。本文中呈现的结果仅基于 BEAT2-standard的Speaker-2 训练得到。

数据集包含 1762 个序列，每个序列平均长度为 65.66 秒。序列中的每段录音都是对一个日常问题的连续回答。此外，我们在表 8 中报告了 TalkShow [65] 与本文数据集在多样性与节拍一致性（BC）方面的对比。

#### 表 8：多样性与 BC 对比

局部（Local）与全局（Global）多样性分别指不含与含全局位移时关节位置的方差。

| 数据集 | BC ↑ | Diversity-L ↑ | Diversity-G ↑ |
|---|---|---|---|
| TalkShow [65] | 6.104 | 5.273 | 5.273 |
| **BEAT2（本文）** | **6.896** | **13.074** | **27.541** |

### MoSh++ 的损失项

MoSh 的优化涉及以下损失函数：数据项、表面距离能量、标记点初始化正则项、姿态与形状先验，以及速度恒定项，具体如下：

- **数据项（$E_{D}$）**：最小化模拟标记点与观测标记点之间的平方距离。此处 $\tilde{M},\beta,\Theta,\Gamma$ 分别表示隐标记点、身体形状、姿态与身体位置：

$$E_{D}(\tilde{M},\beta,\Theta,\Gamma)=\sum_{i,t}||\hat{m}(\tilde{m}_{i},\beta,\theta_{t},\gamma_{t})-m_{i,t}||^{2} \tag{14}$$

- **表面距离能量（$E_{S}$）**：确保标记点与身体表面保持规定的距离：

$$E_{S}(\beta,\tilde{M})=\sum_{i}||r(\tilde{m}_{i},S(\beta,\theta_{0},\gamma_{0}))-d_{i}||^{2} \tag{15}$$

- **标记点初始化正则项（$E_{I}$）**：惩罚估计标记点相对初始位置的偏离：

$$E_{I}(\beta,\tilde{M})=\sum_{i}||\tilde{m}_{i}-v_{i}(\beta)||^{2} \tag{16}$$

- **姿态与形状先验**：惩罚相对平均形状与姿态的偏离：

$$E_{\beta}(\beta)=(\beta-\mu_{\beta})^{T}\Sigma^{-1}_{\beta}(\beta-\mu_{\beta}) \tag{17}$$

$$E_{\theta}(\Theta)=\sum_{t}(\theta_{t}-\mu_{\theta})^{T}\Sigma^{-1}_{\theta}(\theta_{t}-\mu_{\theta}) \tag{18}$$

- **速度恒定项（$E_{u}$）**：降低标记点噪声并确保运动一致性：

$$E_{u}(\Theta)=\sum_{t=2}^{n}||\theta_{t}-2\theta_{t-1}+\theta_{t-2}||^{2} \tag{19}$$

总体目标函数是这些项的加权和，以平衡精度与合理性：

$$E(\tilde{M},\beta,\Theta,\Gamma)=\sum_{\omega\in\{D,S,\theta,\beta,I,u\}}\lambda_{\omega}E_{\omega}(\cdot) \tag{20}$$

关于头部与颈部形状优化的更多细节与伪代码见代码发布。

### FLAME 参数优化细节

为了使用 SMPL-X 模型配合来自 BEAT 数据集的 ARKit 参数驱动面部，我们通过最小化被驱动的 ARKit 模型与 FLAME 模型之间的几何误差来估计 FLAME 表情参数。

为应对网格结构差异带来的优化困难，我们使用 Faceit（一个专为制作 ARKit 混合形变而设计的 Blender 插件）构建了一个 ARKit 兼容的 FLAME 模型。通过用 BEAT 数据集中每一组 ARKit 参数驱动这个 ARKit 对齐的 FLAME 模型，我们通过最小化等价顶点之间的 L2 距离损失，获得原始 FLAME 表情参数。最终，优化后的 FLAME 表情参数可以直接应用于 SMPL-X。

对于面部身份参数，我们在使用 MoSh++ [46] 完成身体拟合后，在 SMPL-X 上保留相同的身份参数。

## 附录 C：基线方法复现细节

### 关节数量

所有基线方法输出由 $\mathbf{g}\in\mathbb{R}^{T\times(55\times 6)}$ 表示的全身关节旋转，此外还解码全局位移 $\in\mathbb{R}^{T\times 3}$。为提供全面的比较，我们同时给出上半身（不含全局运动）与全身的主观结果。

### 自回归训练

我们观察到，基于自回归训练/推理的模型（例如 FaceFormer 与 CodeTalker [19, 61]）表现不如非自回归方法。在非自回归设置中，仅使用位置嵌入作为交叉注意力音频特征的输入，尤其是在使用 Rot6D 与轴角表示进行训练时。

FaceFormer 与 CodeTalker 的网络架构基于 Transformer，最初是为使用顶点偏移表示进行训练而提出的。如表 9 所示，我们发现在使用 FLAME 参数时，非自回归训练能够提升性能。本文中的结果均采用非自回归训练方法获得。非自回归训练技术也被应用于 EMAGE 的训练。

#### 表 9：不同训练方法下的顶点误差（MSE）

'FF' 与 'CT' 分别指FaceFormer [19] 与 CodeTalker [61]。'TF'、'AR' 与 'NonAR' 分别表示教师强制（Teacher-Force）、自回归（AutoRegressive）与非自回归（Non-AutoRegressive）训练。我们在 VOCA 数据集上使用顶点损失训练，在 BEAT2 上使用 FLAME 参数损失结合顶点损失训练。结果表明，同一方法在两种表示下表现不同；在 BEAT2 上，非自回归训练表现更优。VOCA 与 BEAT2 的平均 MSE 分别在 5023 与 10475 个顶点上计算：

| 数据集 | FF-TF | FF-AR | FF-NonAR | CT-TF | CT-AR | CT-NonAR |
|---|---|---|---|---|---|---|
| VOCA (×10⁻⁷) | 6.636| **6.023** | 6.138 | 7.914 | 7.637 | **7.541** |
| BEAT2 (×10⁻⁷) | 2.167 | 3.704 | **1.195** | 2.079 | 4.120 | **1.243** |

### 对抗训练

我们在 Speech2Gesture [25]、CaMN [39] 与 Habibie 等人 [28] 的复现中省略了对抗训练，因为即使增大速度损失的权重，它们在对抗训练下的输出仍存在明显抖动。[33] 的研究也报告了在 Speech2Gesture [25] 上使用 3D 数据训练时的类似现象。

### 用于 TalkShow 的下半身 VQ-VAE

我们为 TalkShow 引入了一个额外的 VQ-VAE，利用其自回归（AR）模型联合预测上半身、手部与下半身的类别索引。全局位移与下半身关节一同编码。

## 附录 D：EMAGE 的设置

### 训练

我们训练模型 400 个epoch，并根据训练 epoch 将被掩码关节的比例从 0 线性提升至 95%。根据我们的实验，这一做法比固定掩码比例（例如 25%）更有效。学习率为 2.5e-4，我们使用 Adam 优化器，并将梯度范数裁剪到 0.99 以确保训练稳定。

### VQ-VAE 的结构

我们对四个身体部位采用相同的基于 CNN 的 VQ-VAE [27]。下采样率设为 1 以获得最佳重建质量。码本条目的特征长度为 512，码本大小设为 256。身体手势的总解码空间表示为 $\in\mathbb{R}^{T\times 256^{3}}$。VQ-VAE 训练 200 个 epoch，前 195 个 epoch 学习率为 2.5e-4，最后 5 个 epoch 降低学习率。

### 全局运动预测器

我们使用一个与 VQ-VAE 编码器和解码器的 CNN 结构相同的架构来训练全局运动预测器。输入由局部运动与预测的足部接触标签构成，维度为 $\in\mathbb{R}^{T\times 334}$，输出为全局位移 $\in\mathbb{R}^{T\times 3}$。

## 附录 E：可视化 Blender 插件

为便于直观可视化 BEAT2 数据集，我们使用 SMPL-X Blender 插件 [50]。由于最新的 SMPL-X 插件并不支持 SMPL-X 的完整面部表情范围，我们从原始 SMPL-X 模型中提取了 300 个表情网格，并将它们作为独立的混合形变目标添加到 Blender 插件中的 SMPL-X 模型内。

## 附录 F：训练时间

我们报告在单张 L4、V100 与 4090 上、批大小（BS）为 64 时获得最佳性能所需的训练时间。

**EMAGE 训练时间：**

| 硬件 | 1 说话者-1 epoch | 1 说话者-400 epoch | 25 说话者-1 epoch | 25 说话者-100 epoch | 显存 | BS |
|---|---|---|---|---|---|---|
| L4 (24G) | 239s | 26.5h | 3197s | 89.6h | 20.1G | 64 |
| V100 (32G) | 155s | 17.2h | 2073s | 58.1h | 20.1G | 64 |
| 4090 (24G) | 72s | 8.0h | 963s | 27.1h | 20.1G | 64 |

此外，为面部、手部、上半身、下半身与全局运动预训练 5 个 VQVAE，在 5 张 4090 GPU 上约需 22.4 小时。

**单个 VQVAE 训练时间：**

| 硬件 | 1 说话者-1 epoch | 1 说话者-700 epoch | 25 说话者-1 epoch | 25 说话者-100 epoch | 显存 | BS |
|---|---|---|---|---|---|---|
| L4 (24G) | 200s | 39.5h | 2760s | 74.4h | 13.8G | 64 |
| V100 (32G) | 131s | 25.5h | 1727s | 48.0h | 13.8G | 64 |
| 4090 (24G) | 61s | 11.9h | 802s | 22.4h | 13.8G | 64 |

## 附录 G：下半身运动的重要性

下半身运动使得与音频内容语义对齐的手势能够呈现更加生动、令人印象深刻的效果，例如 "hiking in nature"（在自然中徒步）配合行走手势、"playing football"（踢足球）配合踢腿动作（见原文下方配图）。与上半身相比，下半身与音频的关联更弱，但在上述情况下仍存在联系。

在 EMAGE 的实现中，我们首先用独立的 MLP 获得不同身体部位的隐变量。随后，下半身运动解码器利用"音频"、"上半身"与"手部"的全部隐变量，进行基于交叉注意力的下半身运动解码。我们还观察到，直接从音频解码会增加多样性，但会降低 BEATv1.3 上结果的连贯性。

| 下半身解码输入 | FGD ↓ | BC ↑ | Diversity ↑ | MSE ↓ | LVD ↓ |
|---|---|---|---|---|---|
| 仅音频（audio only） | 6.209 | 6.683 | **13.714** | 1.183 | 8.788 |
| 音频 + 上半身 + 手部 | **5.423** | **6.794** | 13.075 | **1.180** | **8.715** |

---

## 参考文献

1. **Aberman 等人 [2020]** Kfir Aberman, Peizhuo Li, Dani Lischinski, Olga Sorkine-Hornung, Daniel Cohen-Or, Baoquan Chen. Skeleton-aware networks for deep motion retargeting. *ACM Transactions on Graphics (TOG)*, 39(4):62–1, 2020.
2. **Ahuja 等人 [2020]** Chaitanya Ahuja, Dong Won Lee, Yukiko I Nakano, Louis-Philippe Morency. Style transfer for co-speech gesture animation: A multi-speaker conditional-mixture approach. *ECCV*, pages 248–265. Springer, 2020.
3. **Alexanderson 等人 [2020]** Simon Alexanderson, Gustav Eje Henter, Taras Kucherenko, Jonas Beskow. Style-controllable speech-driven gesture synthesis using normalising flows. *Computer Graphics Forum*, pages 487–496. Wiley Online Library, 2020.
4. **Ao 等人** Tenglong Ao, Zeyi Zhang, Libin Liu. GestureDiffuCLIP: Gesture diffusion model with clip latents. *ACM Trans. Graph.*
5. **Ao 等人 [2022]** Tenglong Ao, Qingzhe Gao, Yuke Lou, Baoquan Chen, Libin Liu. Rhythmic gesticulator: Rhythm-aware co-speech gesture synthesis with hierarchical neural embeddings. *ACM Transactions on Graphics (TOG)*, 41(6):1–19, 2022.
6. **Baevski 等人 [2020]** Alexei Baevski, Yuhao Zhou, Abdelrahman Mohamed, Michael Auli. wav2vec 2.0: A framework for self-supervised learning of speech representations. *NeurIPS*, 33:12449–12460, 2020.
7. **Bai 等人 [2018]** Shaojie Bai, J Zico Kolter, Vladlen Koltun. An empirical evaluation of generic convolutional and recurrent networks for sequence modeling. *arXiv preprint arXiv:1803.01271*, 2018.
8. **Baruch 等人 [2021]** Gilad Baruch, Zhuoyuan Chen, Afshin Dehghan, Tal Dimry, Yuri Feigin, Peter Fu, Thomas Gebauer, Brandon Joffe, Daniel Kurz, Arik Schwartz, Elad Shulman. ARKitScenes — a diverse real-world dataset for 3D indoor scene understanding using mobile RGB-D data. *NeurIPS Datasets and Benchmarks Track (Round 1)*, 2021.
9. **Bojanowski 等人 [2017]** Piotr Bojanowski, Edouard Grave, Armand Joulin, Tomas Mikolov. Enriching word vectors with subword information. *TACL*, 5:135–146, 2017.
10. **BVH [1999]** Biovision BVH. Biovision bvh. https://research.cs.wisc.edu/graphics/Courses/cs-838-1999/Jeff/BVH.html, 1999.
11. **Cao 等人 [2019]** Zhe Cao, Gines Hidalgo, Tomas Simon, Shih-En Wei, Yaser Sheikh. OpenPose: realtime multi-person 2D pose estimation using part affinity fields. *IEEE TPAMI*, 43(1):172–186, 2019.
12. **Chen 等人 [2022]** Zhe Chen, Yuchen Duan, Wenhai Wang, Junjun He, Tong Lu, Jifeng Dai, Yu Qiao. Vision transformer adapter for dense predictions. *arXiv preprint arXiv:2205.08534*, 2022.
13. **Chhatre 等人 [2023]** Kiran Chhatre, Radek Daněček, Nikos Athanasiou, Giorgio Becherini, Christopher Peters, Michael J Black, Timo Bolkart. Emotional speech-driven 3D body animation via disentangled latent diffusion. *arXiv preprint arXiv:2312.04466*, 2023.
14. **Cudeiro 等人 [2019]** Daniel Cudeiro, Timo Bolkart, Cassidy Laidlaw, Anurag Ranjan, Michael Black. Capture, learning, and synthesis of 3D speaking styles. *CVPR*, pages 10101–10111, 2019.
15. **Daněček 等人 [2023]** Radek Daněček, Kiran Chhatre, Shashank Tripathi, Yandong Wen, Michael Black, Timo Bolkart. Emotional speech-driven animation with content-emotion disentanglement. ACM, 2023.
16. **Deichler 等人 [2021]** Anna Deichler, Kiran Chhatre, Christopher Peters, Jonas Beskow. Spatio-temporal priors in 3D human motion. 2021.
17. **Devlin 等人 [2018]** Jacob Devlin, Ming-Wei Chang, Kenton Lee, Kristina Toutanova. BERT: Pre-training of deep bidirectional transformers for language understanding. *arXiv preprint arXiv:1810.04805*, 2018.
18. **Dosovitskiy 等人 [2020]** Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, 等. An image is worth 16x16 words: Transformers for image recognition at scale. *arXiv preprint arXiv:2010.11929*, 2020.
19. **Fan 等人 [2022a]** Yingruo Fan, Zhaojiang Lin, Jun Saito, Wenping Wang, Taku Komura. FaceFormer: Speech-driven 3D facial animation with transformers. *CVPR*, 2022a.
20. **Fan 等人 [2022b]** Yingruo Fan, Zhaojiang Lin, Jun Saito, Wenping Wang, Taku Komura. FaceFormer: Speech-driven 3D facial animation with transformers. *CVPR*, pages 18770–18780, 2022b.
21. **Fanelli 等人 [2010]** Gabriele Fanelli, Juergen Gall, Harald Romsdorfer, Thibaut Weise, Luc Van Gool. A 3-D audio-visual corpus of affective communication. *IEEE Transactions on Multimedia*, 12(6):591–598, 2010.
22. **Ferstl与 McDonnell [2018]** Ylva Ferstl, Rachel McDonnell. Investigating the use of recurrent motion modelling for speech gesture generation. *18th International Conference on Intelligent Virtual Agents*, pages 93–98, 2018.
23. **Gärtner 等人 [2022]** Erik Gärtner, Mykhaylo Andriluka, Erwin Coumans, Cristian Sminchisescu. Differentiable dynamics for articulated 3D human motion reconstruction. *CVPR*, pages 13190–13200, 2022.
24. **Ghorbani 等人 [2023]** Saeed Ghorbani, Ylva Ferstl, Daniel Holden, Nikolaus F. Troje, Marc-André Carbonneau. ZeroEGGS: Zero-shot example-based gesture generation from speech. *Computer Graphics Forum*, 42(1):206–216, 2023.
25. **Ginosar 等人 [2019]** Shiry Ginosar, Amir Bar, Gefen Kohavi, Caroline Chan, Andrew Owens, Jitendra Malik. Learning individual styles of conversational gesture. *CVPR*, pages 3497–3506, 2019.
26. **Gong 等人 [2021]** Yuan Gong, Yu-An Chung, James Glass. AST: Audio spectrogram transformer. *arXiv preprint arXiv:2104.01778*, 2021.
27. **Guo 等人 [2022]** Chuan Guo, Xinxin Zuo, Sen Wang, Li Cheng. TM2T: Stochastic and tokenized modeling for the reciprocal generation of 3D human motions and texts. *ECCV*, pages 580–597. Springer, 2022.
28. **Habibie 等人 [2021]** Ikhsanul Habibie, Weipeng Xu, Dushyant Mehta, Lingjie Liu, Hans-Peter Seidel, Gerard Pons-Moll, Mohamed Elgharib, Christian Theobalt. Learning speech-driven 3D conversational gestures from video. *arXiv preprint arXiv:2102.06837*, 2021.
29. **He 等人 [2022]** Kaiming He, Xinlei Chen, Saining Xie, Yanghao Li, Piotr Dollár, Ross Girshick. Masked autoencoders are scalable vision learners. *CVPR*, pages 16000–16009, 2022.
30. **Ionescu 等人 [2013]** Catalin Ionescu, Dragos Papava, Vlad Olaru, Cristian Sminchisescu. Human3.6M: Large scale datasets and predictive methods for 3D human sensing in natural environments. *IEEE TPAMI*, 36(7):1325–1339, 2013.
31. **Lan 等人 [2019]** Zhenzhong Lan, Mingda Chen, Sebastian Goodman, Kevin Gimpel, Piyush Sharma, Radu Soricut. ALBERT: A lite BERT for self-supervised learning of language representations. *arXiv preprint arXiv:1909.11942*, 2019.
32. **Lee 等人 [2019]** Gilwoo Lee, Zhiwei Deng, Shugao Ma, Takaaki Shiratori, Siddhartha S Srinivasa, Yaser Sheikh. Talking with hands 16.2M: A large-scale dataset of synchronized body-finger motion and audio for conversational motion analysis and synthesis. *ICCV*, pages 763–772, 2019.
33. **Li 等人 [2021a]** Jing Li, Di Kang, Wenjie Pei, Xuefei Zhe, Ying Zhang, Zhenyu He, Linchao Bao. Audio2Gestures: Generating diverse gestures from speech audio with conditional variational autoencoders. *ICCV*, pages 11293–11302, 2021a.
34. **Li 等人 [2021b]** Ruilong Li, Shan Yang, David A Ross, Angjoo Kanazawa. AI Choreographer: Music conditioned 3D dance generation with AIST++. *ICCV*, pages 13401–13412, 2021b.
35. **Li 等人 [2017]** Tianye Li, Timo Bolkart, Michael J. Black, Hao Li, Javier Romero. Learning a model of facial shape and expression from 4D scans. *ACM TOG (Proc. SIGGRAPH Asia)*, 36(6):194:1–194:17, 2017.
36. **Liu 与 Zhang [2020]** Haiyang Liu, Cheng Zhang. Reinforcement learning based neural architecture search for audio tagging. *IJCNN 2020*, pages 1–8. IEEE, 2020.
37. **Liu 与 Zhang [2021]** Haiyang Liu, Jihan Zhang. Improving ultrasound tongue image reconstruction from lip images using self-supervised learning and attention mechanism. *arXiv preprint arXiv:2106.11769*, 2021.
38. **Liu 等人 [2022a]** Haiyang Liu, Naoya Iwamoto, Zihao Zhu, Zhengqing Li, You Zhou, Elif Bozkurt, Bo Zheng. DisCo: Disentangled implicit content and rhythm learning for diverse co-speech gestures synthesis. *ACM MM 2022*, pages 3764–3773, 2022a.
39. **Liu 等人 [2022b]** Haiyang Liu, Zihao Zhu, Naoya Iwamoto, Yichen Peng, Zhengqing Li, You Zhou, Elif Bozkurt, Bo Zheng. BEAT: A large-scale semantic and emotional multi-modal dataset for conversational gestures synthesis. *arXiv preprint arXiv:2203.05297*, 2022b.
40. **Liu 等人 [2022c]** Xian Liu, Qianyi Wu, Hang Zhou, Yinghao Xu, Rui Qian, Xinyi Lin, Xiaowei Zhou, Wayne Wu, Bo Dai, Bolei Zhou. Learning hierarchical cross-modal association for co-speech gesture generation. *CVPR*, pages 10462–10472, 2022c.
41. **Liu 等人 [2022d]** 同上（重复条目）。Learning hierarchical cross-modal association for co-speech gesture generation. *CVPR*, pages 10462–10472, 2022d.
42. **Liu 等人 [2019]** Yinhan Liu, Myle Ott, Naman Goyal, 等. RoBERTa: A robustly optimized BERT pretraining approach. *arXiv preprint arXiv:1907.11692*, 2019.
43. **Liu 等人 [2021]** Ze Liu, Yutong Lin, Yue Cao, Han Hu, Yixuan Wei, Zheng Zhang, Stephen Lin, Baining Guo. Swin Transformer: Hierarchical vision transformer using shifted windows. *ICCV*, pages 10012–10022, 2021.
44. **Loper 等人 [2014]** Matthew M. Loper, Naureen Mahmood, Michael J. Black. MoSh: Motion and shape capture from sparse markers. *ACM TOG (Proc. SIGGRAPH Asia)*, 33(6):220:1–220:13, 2014.
45. **Lu 等人 [2023]** Shuhong Lu, Youngwoo Yoon, Andrew Feng. Co-speech gesture synthesis using discrete gesture token learning. *arXiv preprint arXiv:2303.12822*, 2023.
46. **Mahmood 等人 [2019]** Naureen Mahmood, Nima Ghorbani, Nikolaus F. Troje, Gerard Pons-Moll, Michael J. Black. AMASS: Archive of motion capture as surface shapes. *ICCV*, pages 5442–5451, 2019.
47. **Massey [2020]** Tim Massey. Vicon study of dynamic object tracking accuracy. https://www.vicon.com/resources/blog/vicon-study-of-dynamic-object-tracking-accuracy/, 2020.
48. **Ng 等人 [2021]** Evonne Ng, Shiry Ginosar, Trevor Darrell, Hanbyul Joo. Body2Hands: Learning to infer 3D hands from conversational gesture body dynamics. *CVPR*, pages 11865–11874, 2021.
49. **Pang 等人 [2023]** Kunkun Pang, Dafei Qin, Yingruo Fan, Julian Habekost, Takaaki Shiratori, Junichi Yamagishi, Taku Komura. BodyFormer: Semantics-guided 3D body gesture synthesis with transformer. *ACM TOG*, 42(4):1–12, 2023.
50. **Pavlakos 等人 [2019]** Georgios Pavlakos, Vasileios Choutas, Nima Ghorbani, Timo Bolkart, Ahmed A. A. Osman, Dimitrios Tzionas, Michael J. Black. Expressive body capture: 3D hands, face, and body from a single image. *CVPR*, pages 10975–10985, 2019.
51. **Peng 等人 [2023]**Ziqiao Peng, Haoyu Wu, Zhenbo Song, Hao Xu, Xiangyu Zhu, Jun He, Hongyan Liu, Zhaoxin Fan. EmoTalk: Speech-driven emotional disentanglement for 3D face animation. *ICCV*, pages 20687–20697, 2023.
52. **Petrovich 等人 [2021]** Mathis Petrovich, Michael J Black, Gül Varol. Action-conditioned 3D human motion synthesis with transformer VAE. *ICCV*, pages 10985–10995, 2021.
53. **Qi 等人 [2023a]** Xingqun Qi, Chen Liu, Lincheng Li, Jie Hou, Haoran Xin, Xin Yu. EmotionGesture: Audio-driven diverse emotional co-speech 3D gesture generation. *arXiv preprint arXiv:2305.18891*, 2023a.
54. **Qi 等人 [2023b]** Xingqun Qi, Jiahao Pan, Peng Li, Ruibin Yuan, Xiaowei Chi, Mengfei Li, Wenhan Luo, Wei Xue, Shanghang Zhang, Qifeng Liu, 等. Weakly-supervised emotion transition learning for diverse 3D co-speech gesture generation. *arXiv preprint arXiv:2311.17532*, 2023b.
55. **Qian 等人 [2022]** Guocheng Qian, Xingdi Zhang, Abdullah Hamdi, Bernard Ghanem. Pix4Point: Image pretrained transformers for 3D point cloud understanding. 2022.
56. **Richard 等人 [2021]** Alexander Richard, Michael Zollhöfer, Yandong Wen, Fernando De la Torre, Yaser Sheikh. MeshTalk: 3D face animation from speech using cross-modality disentanglement. *ICCV*, pages 1173–1182, 2021.
57. **Shiohara 等人 [2023]** Kaede Shiohara, Xingchao Yang, Takafumi Taketomi. BlendFace: Re-designing identity encoders for face-swapping. *ICCV*, pages 7634–7644, 2023.
58. **Tykkälä 等人 [2011]** Tommi Tykkälä, Cédric Audras, Andrew I Comport. Direct iterative closest point for real-time visual odometry. *ICCV Workshops 2011*, pages 2050–2056. IEEE, 2011.
59. **Van Den Oord 等人 [2017]** Aaron Van Den Oord, Oriol Vinyals, 等. Neural discrete representation learning. *NeurIPS*, 30, 2017.
60. **Wang 等人 [2020]** Kaisiyuan Wang, Qianyi Wu, Linsen Song, Zhuoqian Yang, Wayne Wu, Chen Qian, Ran He, Yu Qiao, Chen Change Loy. MEAD: A large-scale audio-visual dataset for emotional talking-face generation. *ECCV*, 2020.
61. **Xing 等人 [2023]** Jinbo Xing, Menghan Xia, Yuechen Zhang, Xiaodong Cun, Jue Wang, Tien-Tsin Wong. CodeTalker: Speech-driven 3D facial animation with discrete motion prior. *CVPR*, pages 12780–12790, 2023.
62. **Yang 等人 [2023a]** Sicheng Yang, Zhiyong Wu, Minglei Li, Zhensong Zhang, Lei Hao, Weihong Bao, Ming Cheng, Long Xiao. DiffuseStyleGesture: Stylized audio-driven co-speech gesture generation with diffusion models. *arXiv preprint arXiv:2305.04919*, 2023a.
63. **Yang 与 Taketomi [2022]** Xingchao Yang, Takafumi Taketomi. BareSkinNet: De-makeup and de-lighting via 3D face reconstruction. *Computer Graphics Forum*, pages 623–634. Wiley Online Library, 2022.
64. **Yang 等人 [2023b]** Xingchao Yang, Takafumi Taketomi, Yoshihiro Kanamori. Makeup extraction of 3D representation via illumination-aware image decomposition. *Computer Graphics Forum*, pages 293–307. Wiley Online Library, 2023b.
65. **Yi 等人 [2023]** Hongwei Yi, Hualin Liang, Yifei Liu, Qiong Cao, Yandong Wen, Timo Bolkart, Dacheng Tao, Michael J Black. Generating holistic 3D human motion from speech. *CVPR*, 2023.
66. **Yoon 等人 [2019]** Youngwoo Yoon, Woo-Ri Ko, Minsu Jang, Jaeyeon Lee, Jaehong Kim, Geehyuk Lee. Robots learn social skills: End-to-end learning of co-speech gesture generation for humanoid robots. *ICRA 2019*, pages 4303–4309. IEEE, 2019.
67. **Yoon 等人 [2020]** Youngwoo Yoon, Bok Cha, Joo-Haeng Lee, Minsu Jang, Jaeyeon Lee, Jaehong Kim, Geehyuk Lee. Speech gesture generation from the trimodal context of text, audio, and speaker identity. *ACM TOG*, 39(6):1–16, 2020.
68. **Yu 等人 [2022]** Xumin Yu, Lulu Tang, Yongming Rao, Tiejun Huang, Jie Zhou, Jiwen Lu. Point-BERT: Pre-training 3D point cloud transformers with masked point modeling. *CVPR*, pages 19313–19322, 2022.
69. **Zhao 等人 [2021]** Hengshuang Zhao, Li Jiang, Jiaya Jia, Philip HS Torr, Vladlen Koltun. Point Transformer. *ICCV*, pages 16259–16268, 2021.
70. **Zhi 等人 [2023]** Yihao Zhi, Xiaodong Cun, Xuelin Chen, Xi Shen, Wen Guo, Shaoli Huang, Shenghua Gao. LivelySpeaker: Towards semantic-aware co-speech gesture generation. *ICCV*, pages 20807–20817, 2023.
71. **Zhou 等人 [2022]** Yang Zhou, Jimei Yang, Dingzeyu Li, Jun Saito, Deepali Aneja, Evangelos Kalogerakis. Audio-driven neural gesture reenactment with video motion graphs. *CVPR*, pages 3418–3428, 2022.
72. **Zhu 等人 [2023a]** Lingting Zhu, Xian Liu, Xuanyu Liu, Rui Qian, Ziwei Liu, Lequan Yu. Taming diffusion models for audio-driven co-speech gesture generation. *CVPR*, pages 10544–10553, 2023a.
73. **Zhu 等人 [2023b]** Wentao Zhu, Xiaoxuan Ma, Zhaoyang Liu, Libin Liu, Wayne Wu, Yizhou Wang. MotionBERT: A unified perspective on learning human motion representations. *ICCV*, 2023b.

---

## 译者说明

**翻译版本**：本译文基于 arXiv:2401.00374**v5**（2024年 3 月 30 日提交，CVPR 2024 Camera Ready 版本）的 LaTeXML 全文，使用官方 HTML 全文提取（而非 PDF），因此保留了完整的表格结构与数学公式。

**版本历史与体积变化**：
- v1（2023-12-31）与 v2（2024-01-02）：4,339 KB
- v3（2024-03-25）与 v4（2024-03-27）：15,189 KB — 体积从约 4.3 MB 跃升至约 15 MB，主要是补充材料（附录 A–G）与大量图表在 CVPR Camera Ready 阶段被加入
- v5（2024-03-30，当前版本）：15,451 KB — 相比 v4 修正了拼写错误，并新增了**利益冲突声明**（Conflict of Interest Disclosure）

**标题**：各版本标题一致，未发生变化，均为 "EMAGE: Towards Unified Holistic Co-Speech Gesture Generation via Expressive Masked Audio Gesture Modeling"。

**术语约定**：

| 英文 | 本译文用法 |
|---|---|
| Co-Speech Gesture | 伴随语音手势 |
| Holistic | 整体性（指同时覆盖面部、身体、手部、全局位移） |
| Masked / Mask | 掩码 |
| Body Hints | 身体线索 |
| Blendshape Weights | 混合形变权重 |
| Mesh-level | 网格级 |
| Pseudo Ground Truth (PGT) | 伪真值 |
| Motion Capture (Mocap) | 动作捕捉 / 动捕 |
| Rot6D | 6 维旋转表示（保留原文写法） |
| Global Translation | 全局位移 |
| Foot Sliding | 足部滑动 |
| Codebook | 码本 |
| Commitment Loss | 承诺损失 |
| Beat Constancy (BC) | 节拍一致性 |
| Fréchet Gesture Distance (FGD) | Fréchet 手势距离 |
| Teacher-Force | 教师强制 |
| Autoregressive (AR) | 自回归 |

**保留原文的部分**：
- 所有方法名、数据集名（EMAGE、BEAT2、SMPL-X、FLAME、MoSh++、VQ-VAE、TalkSHOW、FaceFormer、CodeTalker 等）均保留英文原名，首次出现时给出中文解释。
- 所有数学公式与符号完整保留，公式编号与原文一致（1–20）。
- 参考文献保留英文原题名，作者名不作音译，仅将"et al." 译为"等人"。

**表格处理说明**：原文中表 1、表 2 在 LaTeXML 输出里是列名与行值分离的横向宽表，本译文已将其重排为标准的"每行一个方法"纵向表格，便于阅读，数据内容与原文完全一致。原文表格中使用的 ✓/✗ 符号在表 7 中已替换为"有/无"，以避免渲染兼容问题。

**图片说明**：由于本译文为纯文本/Markdown 格式，原文中的图 1–图 7 未内嵌图片，但完整翻译了每张图的图注（caption），以便读者对照原文 PDF 阅读。建议配合原文 PDF 或项目主页（https://pantomatrix.github.io/EMAGE/）的演示视频一起查看。
