# Human Motion Generation 顶会论文综述（2024–2026）

> **最后更新**：2026-08-11  
> **覆盖会议**：NeurIPS / ICML / ICLR / AAAI / IJCAI / CVPR / ECCV / ICCV / Eurographics (EG) / ACM SIGGRAPH / SCA  
> **数据源**：各会议官方 proceedings（CVF Open Access、papers.nips.cc、openreview.net、proceedings.mlr.press、diglib.eg.org、ACM DL）、arXiv API 摘要抓取、`Foruck/Awesome-Human-Motion` 社区仓库交叉校验。  
> **收录标准**：以 motion generation（text/speech/music-driven、co-speech gesture、dance、human-object/scene/human interaction、physics-based character control、whole-body、sign language、motion editing、dataset/benchmark）为主线，每篇总结涵盖任务定义、表征方式、生成范式、核心模块、数据集与指标数字、亮点/局限、链接。  
> **免责**：所有条目均以官方录用公告或作者声明为准；无法确证会议归属者标 `（归属待核实）`。ICML 2026 / NeurIPS 2026 / IJCAI 2025–2026 部分条目因 proceedings 开放进度可能不全，已明确标注。  
> **指标说明**：FID / R-Precision / MM-Dist / Diversity / MModality 均指 HumanML3D 或 KIT-ML 标准评测协议下的数值，除特别说明外 FID 越低越好、R-Precision 越高越好。

---

## 目录

1. [CVPR / ECCV / ICCV](#cvpr--eccv--iccv)
2. [ICLR](#iclr)
3. [NeurIPS / ICML](#neurips--icml)
4. [AAAI / IJCAI](#aaai--ijcai)
5. [Eurographics / SCA](#eurographics--sca)
6. [跨会议趋势分析](#跨会议趋势分析)

---

# CVPR / ECCV / ICCV

> 数据来源：CVF Open Access 官方论文列表（CVPR 2024/2025/2026、ICCV 2025 全量标题索引 + 摘要正文抓取）、arXiv API 摘要，以及 `Foruck/Awesome-Human-Motion` 等社区整理仓库交叉校验。所有条目的会议归属均以 CVF Open Access 收录或作者/官方公告为准；无法确证者已单列到末尾小节。
> 指标说明：下文 FID / R-Precision / MM-Dist / Diversity / MModality 均指 HumanML3D 或 KIT-ML 标准评测协议下的数值，除特别说明外方向为 FID 越低越好、R-Precision 越高越好。

---

## CVPR 2026

> 说明：CVPR 2026（2026 年 6 月 3–7 日，美国丹佛）共收到 16092 篇投稿、录用 4090 篇（录用率 25.42%），另有 1717 篇进入 Findings。CVF Open Access 已开放，本节条目均取自官方 proceedings 页面。本届 motion 方向有两条清晰主线：**（一）因果化 / 流式化生成**（CMDM、FloodDiffusion、LiveGesture、MIBURI）；**（二）数据规模化与语义分层**（OpenT2M、RoMo、MotionMaster、HandX）。

### Causal Motion Diffusion Models for Autoregressive Motion Generation (CMDM)
- **作者/机构**：Qing Yu, Akihisa Watanabe, Kent Fujiwara（LY Corporation / LINE ヤフー）
- **任务**：text-to-motion、流式（streaming）生成、长序列生成
- **核心方法**：针对"full-sequence diffusion 的双向注意力破坏时间因果性、纯自回归模型误差累积"这一两难，提出因果扩散范式。先训练 **MAC-VAE（Motion-Language-Aligned Causal VAE）**，把 motion 编码为时间上严格因果的连续 latent（编码器只看过去帧），且 latent 与语言语义对齐。在此 latent 上训练 **Causal-DiT**，用 causal attention + causal diffusion forcing 做按时间顺序的逐帧去噪。关键创新是 **Frame-Wise Sampling Schedule (FSS)**：不同于传统 diffusion 给整段序列施加同一 noise level，FSS 给每一帧分配独立噪声阶段（越靠未来噪声越大），推理时用"部分去噪的过去帧"预测下一帧，从而天然编码"过去已确定、未来更不确定"的因果结构。
- **数据集/评测**：HumanML3D、SnapMoGen。在语义保真度与时间平滑性上同时超过 full-sequence diffusion 与自回归基线，并显著降低推理延迟，支持交互速率下的流式合成、多轮生成与长时程生成。
- **亮点/局限**：真正把 diffusion forcing 的"逐帧异构噪声"思想落到 motion 的因果 latent 上，是 MotionStreamer（ICCV 2025）之后 streaming 路线的重要推进——MotionStreamer 用连续因果 latent + 概率自回归，CMDM 则进一步把去噪调度本身因果化，长序列稳定性更强。局限是流式设定下无法回改已生成帧，全局编排能力弱于离线全序列模型。
- **链接**：http://arxiv.org/abs/2602.22594

### ProjFlow: Projection Sampling with Flow Matching for Zero-Shot Exact Spatial Motion Control
- **作者/机构**：Akihisa Watanabe et al.（LY Corporation / LINE ヤフー）
- **任务**：motion 空间控制、motion inpainting、2D-to-3D lifting
- **核心方法**：观察到大量动画控制任务（轨迹跟随、关键帧插值、相对位置固定、循环生成、2D→3D 提升）都可统一形式化为**线性逆问题**。据此提出 training-free 的 flow matching 采样器：每个采样步先预测 clean endpoint，再把它**投影**到线性约束集合上，最后从修正后的 endpoint 重建下一状态。核心贡献是 **kinematics-aware metric**——投影时的距离度量编码骨骼拓扑，使修正量沿运动链协调地分摊到全身，而非只硬拽某个末端关节导致断裂伪影。对稀疏输入（如长间隔关键帧）另设计随采样过程逐渐衰减的 pseudo-observation 时变形式。
- **数据集/评测**：motion inpainting、2D-to-3D reconstruction 等代表性任务。能做到**约束精确满足（exact constraint satisfaction）**，真实感匹配或超过 zero-shot 基线，并与需训练的专用控制器相当。
- **亮点/局限**：与 DNO（CVPR 2024）同属 training-free 控制路线，但 DNO 需在推理时跑昂贵的噪声优化内循环，ProjFlow 无内循环、单次采样即精确满足硬约束，实用性明显更好。局限是仅覆盖**线性**约束，非线性目标（避障、物理接触）仍需回退到优化式方法。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Watanabe_ProjFlow_Projection_Sampling_with_Flow_Matching_for_Zero-Shot_Exact_Spatial_CVPR_2026_paper.html

### MotionHiFlow: Text-to-Motion via Hierarchical Flow Matching
- **作者/机构**：Heng Li, Xiaotong Lin, Ling-An Zeng, Yulei Kang, Shuai Li, Jian-Fang Hu（中山大学等）
- **任务**：text-to-motion
- **核心方法**：指出现有方法只在**单一时间尺度**上操作，限制了语义对齐与时间连贯性。受人类认知层级化理解复杂动作的启发，提出分层 flow matching：从低时间分辨率到高时间分辨率**逐级构造 flow path**——低尺度 flow 负责高层语义与粗粒度运动结构，高尺度 flow 负责精修时间细节。设计 **cross-scale transition** 过程连接相邻尺度的 flow，保证连续性与噪声一致性（noise consistency）。骨干为 Text-Motion Diffusion Transformer + **topology-aware Motion VAE**，通过 joint-aware positional encoding 与骨骼拓扑显式建模关节间结构依赖。
- **数据集/评测**：HumanML3D、KIT-ML 上报 SOTA；消融确认分层设计与各关键组件有效。
- **亮点/局限**：把图像生成里"coarse-to-fine 多尺度"的成功经验以 flow matching 形式移植到时间轴，与 MoScale 的"next-scale AR"在动机上呼应但范式不同（连续 flow vs 离散 AR）。局限是未披露具体 FID 数字，多尺度带来的额外采样开销也未详述。
- **链接**：https://arxiv.org/abs/2604.23264

### Next-Scale Autoregressive Models for Text-to-Motion Generation (MoScale)
- **作者/机构**：Zhiwei Zheng, Shibo Jin, Lingjie Liu, Mingmin Zhao（University of Pennsylvania）
- **任务**：text-to-motion、motion 编辑（zero-shot 迁移）
- **核心方法**：认为标准 next-token prediction 的因果结构与 motion 的时间结构并不匹配。借鉴 VAR 提出 **next-scale AR**：不逐 token 预测，而是从最粗时间分辨率到最细分辨率**逐 scale** 生成——最粗尺度先给出全局语义骨架，后续尺度渐进精修，构造出更适配长程运动结构的因果层级。另加两个稳健性设计：**cross-scale hierarchical refinement**（改进每个 scale 的初始预测）与 **in-scale temporal refinement**（在 scale 内做选择性双向重预测），以缓解 text-motion 配对数据有限的问题。
- **数据集/评测**：text-to-motion 达 SOTA，训练效率高，随模型规模有效 scaling，并 zero-shot 泛化到多种生成与编辑任务。
- **亮点/局限**：为"AR 派"给出了不同于 T2M-GPT / MoMask 的新因果分解方式——MoMask 是残差层级（幅度维度分层），MoScale 是时间尺度分层，二者互补。局限是未给出与 MoMask/BAMM 的具体数值对比。
- **链接**：http://arxiv.org/abs/2604.03799

### FloodDiffusion: Tailored Diffusion Forcing for Streaming Motion Generation
- **作者/机构**：N/A
- **任务**：text-to-motion、流式生成（时变文本 prompt）
- **核心方法**：给定随时间变化的文本 prompt，实时生成文本对齐且无缝衔接的长序列。放弃 chunk-by-chunk 与"AR 主干 + diffusion head"两种主流做法，直接采用 diffusion forcing 框架。核心贡献是**实证发现 video 领域的 vanilla diffusion forcing 直接搬到 motion 上会建模失败**，并给出三条必要的定制修正：(i) 训练时用**双向注意力**而非 causal attention；(ii) 用**下三角（lower-triangular）时间调度器**而非随机调度；(iii) 以**连续时变方式**注入文本条件。
- **数据集/评测**：HumanML3D 上 **FID 0.057**，是首个证明 diffusion-forcing 框架能在流式 motion 生成上达到 SOTA 的工作，且具备实时延迟。
- **亮点/局限**：负结果 + 消融驱动的方法论贡献很有价值，明确了跨模态迁移 diffusion forcing 的陷阱。有意思的张力：它发现必须用双向注意力，与 CMDM 坚持 causal attention 形成对照，说明"流式"与"因果注意力"并不等价，值得深挖。
- **链接**：https://arxiv.org/abs/2512.03520

### OpenT2M: No-frill Motion Generation with Open-source, Large-scale, High-quality Data
- **作者/机构**：BeingBeyond（research.beingbeyond.com/opent2m）
- **任务**：text-to-motion 数据集 + 预训练基础模型
- **核心方法**：针对现有数据集规模小、多样性差导致 T2M 在未见文本上崩坏，构建 **OpenT2M**——百万量级、开源、**超过 2800 小时**的高质量 motion 数据集，每条序列经物理可行性校验与多粒度过滤，并配**秒级（second-wise）细粒度文本标注**；另建自动化 pipeline 合成长时程序列以支持复杂动作生成。模型侧提出 **MonoFirll**，主张"无需花哨技巧（no frills）"，核心是 **2D-PRQ** tokenizer：按人体生物学部位切分身体，在"部位 × 时间"二维上做渐进残差量化，从而同时捕捉时空依赖。
- **数据集/评测**：OpenT2M 显著提升多个既有 T2M 模型的泛化能力；2D-PRQ 在重建质量与 zero-shot 性能上均更优。
- **亮点/局限**：与 ICCV 2025 的 MotionMillion（2000 小时 / 200 万序列）构成 2025–2026 年"motion 数据 scaling"竞赛的两极，且 OpenT2M 强调开源与秒级标注，对复现更友好。局限是"no-frill"主张下模型创新相对保守，性能提升主要来自数据。
- **链接**：https://arxiv.org/abs/2603.18623

### RoMo: A Large-Scale, Richly Organized Dataset and Semantic Taxonomy for Human Motion Generation
- **作者/机构**：Zhang et al.
- **任务**：数据集 / 评测基准 + 语义分类体系
- **核心方法**：直指 3D motion 数据的核心困境——要么是小规模高保真 mocap，要么是被静态/低质量序列主导的大规模 in-the-wild 集合。RoMo 引入 **taxonomy-aware 过滤 pipeline**，激进剔除静态与含伪影序列；每条序列配详细 caption，并按新设计的**三级语义分类体系（three-level semantic taxonomy）**组织。这一层级结构提供了**首个支持细粒度按类别评测的 benchmark**，可暴露被全局指标（如单一 FID）掩盖的模型强弱项。同时发布 **Motion Toolbox** 统一指标计算、数据格式转换与可视化。
- **数据集/评测**：在 RoMo 上训练的模型在保真度与多样性上达 SOTA，且对复杂、微妙的文本 prompt 理解更好。
- **亮点/局限**：对做 **motion quality 评测**的研究者价值极高——"per-category 评测 + 标准化 toolbox"正是当前领域最缺的基础设施；全局 FID 长期被诟病对特征提取器敏感，分类别诊断是务实补充。局限是新 taxonomy 的社区采纳仍需时间。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Zhang_RoMo_A_Large-Scale_Richly_Organized_Dataset_and_Semantic_Taxonomy_for_CVPR_2026_paper.html

### MotionMaster: Generalizable Text-Driven Motion Generation and Editing
- **作者/机构**：Jiang et al.
- **任务**：text-to-motion、多动作组合、motion 编辑（统一 MLLM 框架）
- **核心方法**：主张不该从零训练 motion 模型，而应复用预训练 MLLM 中已有的动作语义与长程推理能力。三个组件：**MotionGB** 数据集——从 400 小时经校验的 mocap 通过时空增强扩展到 **10000 小时**；**FSQ-based tokenizer**——用有限标量量化同时保住局部关节精度与全局轨迹连贯性（规避 VQ 的 codebook collapse）；以及在共享 embedding 空间中融合 motion token 与 language token 的**微调 MLLM**。
- **数据集/评测**：多动作语义一致性相比先前方法提升 **41.6%**，身体部位组合（body-part composition）提升 **20.8%**，并展现强 zero-shot 泛化。
- **亮点/局限**：明确验证了"MLLM 预训练知识可迁移到 motion 理解"这一假设，是 MotionGPT / AvatarGPT 路线的规模化续作。局限是 10000 小时中绝大部分来自增强而非真实采集，增强数据的分布真实性值得审视。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Jiang_MotionMaster_Generalizable_Text-Driven_Motion_Generation_and_Editing_CVPR_2026_paper.html

### Open the Motion Door: Atomic Motion Decomposition and Recomposition for Open-Vocabulary Motion Generation
- **作者/机构**：Ke Fan, Jiangning Zhang, Ran Yi, Jingyu Gong, Yabiao Wang, Yating Wang, Xin Tan, Chengjie Wang, Lizhuang Ma（上海交大 / 腾讯优图 / 华东师大）
- **任务**：open-vocabulary text-to-motion
- **核心方法**：核心观察是——尽管高层动作语义千差万别，但大量动作共享同一组底层**原子动作（atomic motions）**，即简单可复用的身体部位运动。据此构建"分解—重组"框架：**Textual Decomposition** 模块把域外（out-of-domain）描述解析为原子动作单元；**Atomic Recomposition** 模块把这些单元整合为最终 motion 序列。本质是用原子动作作为中间语义瓶颈，把"未见过的复合语义"归约到"见过的原子语义组合"。
- **数据集/评测**：域内 HumanML3D 上性能有竞争力；在两个域外数据集 **IDEA400** 与 **Mixamo** 上大幅超越 SOTA 的 open-vocabulary 方法。
- **亮点/局限**：与 ECCV 2024 的 CoMo（pose code）、ParCo（部位协同）同属"分解式可解释表征"脉络，但明确把目标定位在**开放词汇泛化**并用域外集合验证，实验设置比只刷 HumanML3D 更有说服力。局限是原子动作词表构建依赖 LLM 解析质量。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Fan_Open_the_Motion_Door_Atomic_Motion_Decomposition_and_Recomposition_for_CVPR_2026_paper.html

### LaMoGen: Language to Motion Generation Through LLM-Guided Symbolic Inference
- **作者/机构**：Junkun Jiang, Ho Yin Au, Jingyu Xiang, Jie Chen（香港浸会大学）
- **任务**：text-to-motion（可解释符号推理）
- **核心方法**：批评主流依赖 joint text-motion embedding 的方法是黑箱、时间精度不足。提出 **LabanLite**——改编并扩展舞谱系统 **Labanotation** 的运动表征：每个原子身体部位动作（如"左脚单步"）被编码为一个离散 Laban 符号 + 配套文本模板，从而把复杂动作分解为可解释的符号序列与部位指令，在高层语言与低层运动轨迹间建立**符号桥梁**。在此之上的 LaMoGen 走 Text→LabanLite→Motion 流程，由 LLM 通过符号推理解释运动模式、关联文本描述并重组符号成可执行 plan。
- **数据集/评测**：自建 Labanotation-based benchmark（结构化 description-motion 对）+ 三个新指标，分别度量**符号、时间、harmony** 三个维度的 text-motion 对齐；在该 benchmark 与两个公开数据集上优于先前方法。
- **亮点/局限**：把舞蹈学界成熟的 Labanotation 引入生成式 motion，是少见的领域知识注入尝试，可解释性与可控性都是真卖点。局限是 Laban 符号体系的表达上限约束了高动态/风格化动作，自建 benchmark 的通用性也待检验。
- **链接**：http://arxiv.org/abs/2603.11605

### LiveGesture: Streamable Co-Speech Gesture Generation Model
- **作者/机构**：N/A
- **任务**：speech-to-gesture（全身，流式）
- **核心方法**：**首个完全流式、零 look-ahead、支持任意长度**的语音驱动全身手势生成框架。两大模块：**SVQ（Streamable Vector-Quantized Motion Tokenizer）**把各身体区域的 motion 序列转成**因果**离散 token，支持实时流式 token 解码；**HAR（Hierarchical Autoregressive Transformer）**用 region-eXpert 自回归（xAR）transformer 为每个身体区域建模精细动态，再由因果时空融合模块 **xAR-Fusion** 跨区域整合关联动态。两者均以流式因果音频编码器编码的持续到达音频为条件。为提升流式噪声与预测误差下的鲁棒性，引入 **autoregressive masking training**：用不确定性引导的 token masking + 随机区域 masking，让模型在训练期就暴露于不完美、部分错误的历史。
- **数据集/评测**：BEAT2。在真正零 look-ahead 条件下实时生成连贯、多样、节拍同步的全身手势，**匹配或超过 SOTA 离线方法**。
- **亮点/局限**：真实数字人 / 对话 agent 落地最需要的正是零 look-ahead——此前 EMAGE、GestureLSM 等虽快但仍是离线设定，LiveGesture 把约束提到了实用级别。局限是流式因果限制下语义性手势（需预知句子语义）的表现天花板较低。
- **链接**：https://arxiv.org/abs/2604.10927

### MIBURI: Towards Expressive Interactive Gesture Synthesis
- **作者/机构**：MPI-INF（vcai.mpi-inf.mpg.de/projects/MIBURI）
- **任务**：speech-to-gesture + 面部表情（实时对话 agent）
- **核心方法**：面向 Embodied Conversational Agents，提出**首个在线因果框架**，与实时口语对话同步生成富表现力的全身手势与面部表情。采用 **body-part aware gesture codec** 把层级化运动细节编码成多级离散 token，再由**二维因果框架**（时间维 × 部位层级维）自回归生成这些 token，条件为 LLM 的 speech-text embedding。另设辅助目标鼓励手势的表现力与多样性，防止塌缩到静态姿态。
- **数据集/评测**：与近期基线对比评测，因果实时方法生成的手势自然且上下文对齐。
- **亮点/局限**：定位在 LLM 对话 agent 的具身化缺口——既避开 ECA 传统方案的僵硬低多样性，也避开 co-speech 方法对未来语音上下文与长运行时的依赖。与 LiveGesture 高度同期同题，可作为流式手势的一组对照工作。
- **链接**：https://arxiv.org/abs/2603.03282

### CoordSpeaker: Exploiting Gesture Captioning for Coordinated Caption-Empowered Co-Speech Gesture Generation
- **作者/机构**：N/A
- **任务**：speech-to-gesture + 文本驱动手势（多模态协调控制）
- **核心方法**：指出现有 co-speech 方法忽略了文本驱动的**非自发（non-spontaneous）手势**（如"边说话边鞠躬"），根因有两点：手势数据集缺描述性文本标注造成**语义先验缺口**，以及多模态协调控制困难。方案先用一个手势 captioning 框架弥合语义缺口——借 motion-language 模型为手势生成多粒度描述性 caption；在此之上构建条件 latent diffusion 模型，配统一的跨数据集运动表征与**层级受控去噪器（hierarchically controlled denoiser）**。
- **数据集/评测**：生成手势既与语音节奏同步，又与任意 caption 语义连贯，性能与效率均优于既有方法。
- **亮点/局限**："双向 gesture-text 映射"的视角新颖——用理解任务（captioning）反哺生成任务，与 IRG-MotionLLM（ECCV 2026）思路异曲同工。局限是自动 caption 的噪声会传导到生成端。
- **链接**：https://arxiv.org/abs/2511.22863

### OpenDance: Multimodal Controllable 3D Dance Generation with Large-scale Internet Data
- **作者/机构**：N/A
- **任务**：music-to-dance（多模态可控）
- **核心方法**：认为舞蹈生成的实际需求远超"仅音乐条件"，还需空间轨迹、关键帧姿态、风格描述等多样条件，而缺大规模富标注数据是主要瓶颈。构建 **OpenDanceSet**：超 **100 小时**、**14 个流派**、**147 位受试者**，每条样本含 3D motion、配对音乐、2D 关键点、轨迹、专家标注文本描述。模型 **OpenDanceNet** 是统一的 masked modeling 框架，含解耦式 auto-encoder 与多模态联合预测 Transformer，支持以音乐 + 文本/关键点/轨迹的**任意组合**为条件。
- **数据集/评测**：高保真合成，多样性强，物理接触真实，且对空间与风格条件有灵活控制力。
- **亮点/局限**：舞蹈生成长期被 AIST++ 单一数据集垄断，OpenDanceSet 在流派与标注维度上的扩充是实质贡献。局限是互联网数据经 2D→3D 恢复得到，motion 精度不及 mocap。
- **链接**：https://arxiv.org/abs/2506.07565

### Multi-level Causal LLM-based Text-to-Motion Generation with Human Alignment (MoTiGA)
- **作者/机构**：Chen et al.
- **任务**：text-to-motion（LLM 范式 + 偏好对齐）
- **核心方法**：归纳 LLM 式 T2M 的三个病根：细粒度 motion 量化误差、语言的因果表征与 motion 的非因果表征不匹配、缺乏人类偏好对齐。对应给出：**Causal RVQ-VAE** 做多级因果细粒度表征，用迭代残差量化 + 因果卷积降低量化误差同时保持类似语言的因果性；**time-lagged causal prediction** 策略在保持时间依赖前提下跨 token 层级并行预测；**MHPO（Multi-level Hybrid-weighted Preference Optimization）**动态调节语义相似度权重与连续相似度分数做偏好优化。
- **数据集/评测**：发布 **HumanML3D-R**——首个大规模 motion 生成偏好数据集，含 **101490** 组人类偏好对。相比其他 LLM-based 方法，HumanML3D 上 FID 提升 **82.3%**，KIT-ML 提升 **64.7%**。
- **亮点/局限**：把 RLHF/DPO 式偏好对齐系统性引入 motion 生成，且配套开源偏好数据集，这在本领域仍很稀缺（此前仅 AToM 用 GPT-4V reward 做过类似尝试）。局限是"相比其他 LLM-based 方法"的比较范围未必覆盖最强的 masked/diffusion 模型。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Chen_Multi-level_Causal_LLM-based_Text-to-Motion_Generation_with_Human_Alignment_CVPR_2026_paper.html

### Hierarchical Enhancement of Semantic Priors for Disentangled Text-Driven Motion Generation (HESP)
- **作者/机构**：Wenhan Lv, Shaopan Wang, Xiangyu Wu, Tianchu Hang, Zhongquan Jian, Qingqiang Wu（厦门大学、闽江学院）
- **任务**：text-to-motion（解耦、可解释）
- **核心方法**：批评现有 diffusion 方法依赖**各向同性（isotropic）latent 先验**与浅层跨模态监督，导致语义纠缠、可控性与可解释性不足。核心是 **AG-VAE（Adaptive Gaussian VAE）**，把 latent motion manifold 结构化为多个语义一致的**子流形（submanifolds）**，得到可解释可控的运动表征。为弥合语言语义与运动学语义，另设 **DCMM（Dynamic Cross-Modal Memory）**做自适应语义融合、**HCA（Hierarchical Cross-Modal Attention）**捕捉多层级 text-motion 对应。
- **数据集/评测**：HumanML3D、KIT-ML 上持续优于 SALAD、MoMask、MDM，同时保持更高多样性与物理合理性；结构化 latent 空间形成可解释聚类，动作类别间语义边界清晰。
- **亮点/局限**：把"latent 先验的形状"当作一等公民来设计（各向同性 → 混合子流形），这是多数 latent diffusion motion 工作没有认真处理的点。局限是只给相对提升的定性描述，缺具体 FID 数值。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Lv_Hierarchical_Enhancement_of_Semantic_Priors_for_Disentangled_Text-Driven_Motion_Generation_CVPR_2026_paper.html

### MoLingo: Motion-Language Alignment for Text-to-Human Motion Generation
- **作者/机构**：He et al.
- **任务**：text-to-motion（连续 latent 扩散）
- **核心方法**：系统研究"如何让连续 motion latent 上的扩散效果最好"，聚焦两个问题：(1) 如何构造语义对齐的 latent 空间使扩散更有效；(2) 如何注入文本条件使运动紧跟描述。提出用**帧级文本标签**训练**语义对齐 motion encoder**，使语义相近的 latent 在空间中彼此靠近，从而得到更"diffusion-friendly"的 latent 空间。并系统对比 single-token 条件与 **multi-token cross-attention** 方案，发现 cross-attention 在运动真实感与 text-motion 对齐上都更优。最终组合：语义对齐 latent + 自回归生成 + cross-attention 文本条件。
- **数据集/评测**：标准指标与 user study 双双取得 SOTA。
- **亮点/局限**：一篇扎实的"设计空间消融"论文，回答了 MLD 之后长期悬而未决的工程问题（latent 该怎么学、文本该怎么注），对做 latent diffusion 的人参考价值高。局限是方法新颖性偏工程组合，缺单点突破。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/He_MoLingo_Motion-Language_Alignment_for_Text-to-Human_Motion_Generation_CVPR_2026_paper.html

### ParTY: Part-Guidance for Expressive Text-to-Motion Synthesis
- **作者/机构**：N/A
- **任务**：text-to-motion（部位级表现力）
- **核心方法**：指出部位式（part-wise）生成方法有两个硬伤：缺乏把文本语义与**具体身体部位**对齐的显式机制；以及独立生成各部位再拼接导致全身运动不连贯。ParTY 三组件破解这一 trade-off：**Part-Guided Network** 先生成 part motion 作为 part guidance，再用它引导 holistic motion 生成（而非直接拼接）；**Part-aware Text Grounding** 对 text embedding 做多样化变换并恰当对齐到各身体部位；**Holistic-Part Fusion** 自适应融合整体与部位运动。
- **数据集/评测**：含 part-level 与 coherence-level 的评测，相比先前方法有实质提升。
- **亮点/局限**：正面处理了 ParCo（ECCV 2024）之后部位式生成的"表现力 vs 连贯性"两难，用"part 引导 holistic"而非"part 拼接"是干净的解法。
- **链接**：https://arxiv.org/abs/2603.09611

### FrankenMotion: Part-level Human Motion Generation and Composition
- **作者/机构**：N/A
- **任务**：text-to-motion（部位级 + 时序原子级控制）
- **核心方法**：现有方法受限于只有序列级或动作级描述。本工作借助 LLM 推理能力构建带**原子化、时间感知的部位级标注**的高质量数据集——不同于此前"固定时间片段的同步部位 caption"或"仅全局序列标签"，该数据集在细时间分辨率上捕捉**异步且语义各异**的部位运动。基于此提出 diffusion 式部位感知生成框架 FrankenMotion，每个身体部位由其独立的时序结构化文本 prompt 引导。
- **数据集/评测**：优于所有为该设定重训的基线；能组合出训练中未见的 motion。
- **亮点/局限**：作者称这是首个提供原子化时间感知部位级标注、并支持**空间（部位）+ 时间（原子动作）双维控制**的工作。异步部位标注确实填补了 FineMotion（ICCV 2025）等数据集的空白。
- **链接**：https://arxiv.org/abs/2601.10909

### Towards Decompositional Human Motion Generation with Energy-Based Diffusion Models (DeMoGen)
- **作者/机构**：N/A
- **任务**：motion 组合式生成 / 分解（compositional & decompositional）
- **核心方法**：现有工作几乎都做**正向**建模（text→motion，或把概念组合成复杂动作），本文反其道行之：把整体动作**分解**为语义有意义的子成分。提出基于能量的 diffusion 模型的组合式训练范式——能量形式直接刻画多个 motion concept 的**复合分布**，使模型无需各概念的 ground-truth motion 即可发现它们。三个训练变体：**DeMoGen-Exp**（显式在分解后的文本 prompt 上训练）、**DeMoGen-OSS**（正交自监督分解）、**DeMoGen-SC**（强制原始与分解文本 embedding 的语义一致）。另构建 text-decomposed 数据集支持组合式训练。
- **数据集/评测**：能从复杂序列中解耦出可复用的 motion primitive，且分解出的概念可灵活重组生成训练分布外的新动作。
- **亮点/局限**：是 EnergyMoGen（CVPR 2025）的自然对偶延伸——后者做能量式**组合**，本文做能量式**分解**，且不需单概念监督，理论上更 scalable。
- **链接**：https://arxiv.org/abs/2512.22324

### MoCoDiff: A Controllable Autoregressive Diffusion Model for Expressive Motion Generation
- **作者/机构**：Song et al.
- **任务**：motion 生成（长时程 + 风格 + 多条件控制）
- **核心方法**：诊断出问题根源是**融合式条件设计（fused conditioning）**——语义、风格、时间信号共用单一通路造成干扰、限制可控性。提出 **IMC（Injection Modulation Controllers）**：轻量的模态专属线性调制模块，让 text、style、history 各走**独立条件通路**注入，在保持骨干冻结（frozen backbone）的简洁性同时避免纠缠。为增强长程合成，进一步设计 **TIMC（Temporal IMC）**，把历史作为**依赖时间步的修正信号**施加，主动抑制漂移（drift）、强制段间平滑过渡。
- **数据集/评测**：在风格保真度、过渡质量与效率上取得 SOTA，且支持无需重训的灵活可解释多条件合成。
- **亮点/局限**："条件解耦通路 + 冻结骨干"是很实用的工程范式（类似 ControlNet 的模块化精神），对需要同时挂多种控制信号的生产场景友好。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Song_MoCoDiff_A_Controllable_Autoregressive_Diffusion_Model_for_Expressive_Motion_Generation_CVPR_2026_paper.html

### Towards Highly-Constrained Human Motion Generation with Retrieval-Guided Diffusion Noise Optimization
- **作者/机构**：Hanchao Liu et al.（ProgMoGen / DNO 同脉络）
- **任务**：motion 控制（zero-shot 高约束任务）
- **核心方法**：现有方法能处理许多未见约束，但在**极端时空限制**任务上失败（如严重空间障碍、指定行走步数）。在 training-free 的 diffusion noise optimization 框架上引入**检索引导**：核心想法是在大规模 motion 数据集中搜索可能满足困难约束的引导样本。设计 **relational task parsing** 把目标约束分组、识别出需要检索参考来处理的困难项；再用 **reward-guided mask** 把随机噪声与检索到的噪声混合，得到更好的扩散噪声初始化；从该初始化出发做噪声优化。借 LLM 完成 relational task parsing 后，框架可自动推理"该检索什么"。
- **数据集/评测**：成功求解高约束生成任务，超越纯优化式基线。
- **亮点/局限**：DNO（CVPR 2024）→ ProgMoGen（CVPR 2024）→ 本文的演进链条清晰：把"检索增强"引入噪声优化的初始化，直击优化式方法在困难约束下陷入差局部解的痛点。局限是依赖大型 motion 库与检索质量。
- **链接**：https://arxiv.org/abs/2605.08054

### Omni-Supervised Motion Editing: Balancing Change and Invariance through Positive-Negative Learning (OmniME)
- **作者/机构**：Shi et al.
- **任务**：text-based motion editing
- **核心方法**：把 motion 编辑的核心矛盾精确刻画为**change（精确编辑目标区域）与 invariance（保留未编辑部分）的平衡**，批评现有 diffusion 方法依赖启发式相似度线索或粗粒度全局条件，导致运动失真与语义对齐不佳。提出全监督正负学习框架，三个互补组件：**retrospective feature supervision**（跨 transformer 层强制 coarse-to-fine 一致性）、**motion preservation mechanism**（依据 source-target 相似度聚焦细微变化）、**triplet-based semantic alignment**（强化 text-motion 对应）。
- **数据集/评测**：MotionFix、STANCE Adjustment 数据集上编辑对齐度达 SOTA。
- **亮点/局限**：把 SimMotionEdit（CVPR 2025）的"相似度预测辅助任务"推广为更系统的正负样本监督范式；change/invariance 的问题表述本身对后续工作有参考价值。
- **链接**：https://github.com/rocket-ycyer/OmniME

### Cross-Axis Feature Fusion with Joint-Wise Motion Difference Prediction for Text-Based 3D Human Motion Editing
- **作者/机构**：Han et al.
- **任务**：text-based motion editing
- **核心方法**：指出先前工作主要学"编辑在时间上何时发生"，而本文要让模型同时理解"**哪些具体关节**负责这次改变"。架构为两个**轴锚定（axis-anchored）transformer**，分别沿关节维与时间维抽取不同特征，再由 **cross-axis fusion block** 整合。关键是配套的辅助任务：训练 joint-anchored transformer 去**回归 source 与 target 关节旋转之间的 Soft-DTW 距离**，从而教会模块区分该改哪些关节、该保留哪些关节。
- **数据集/评测**：MotionFix 上显著提升与文本指令及源运动的语义对齐度和整体保真度，达 SOTA。
- **亮点/局限**：Soft-DTW 作为可微的"关节级改动量"监督信号是巧妙设计，把编辑定位问题从"时间维"扩展到"关节维"，与 OmniME 形成本届 motion editing 的两条并行思路。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Han_Cross-Axis_Feature_Fusion_with_Joint-Wise_Motion_Difference_Prediction_for_Text-Based_CVPR_2026_paper.html

### PAMotion: Physics-Aware Motion Generation for Full-Body Interaction with Multiple Objects
- **作者/机构**：Di, Li et al.（github.com/liyuheng520/PAMotion）
- **任务**：human-object interaction（多物体全身操作）
- **核心方法**：关键物理洞察非常漂亮——在日常慢速场景中，**物体加速度本身就揭示了物理交互状态**：若物体加速度与重力一致，则它处于自由运动、无接触；否则必然（直接或间接）与人体接触。据此 PAMotion 联合建模全身人体运动、物体运动**及二者的加速度**，并用 **physics-aware interaction loss** 软惩罚"物体加速度与人-物接触状态不一致"的情形。整体 coarse-to-fine：先合成全局躯干与物体平移，再条件式精修手部运动与物体旋转，兼顾高层 motion-text 一致性与低层物理保真。
- **数据集/评测**：**HIMO**、**ParaHome** 两个高难数据集上达 SOTA，生成涉及多物体的真实、物理一致的全身操作序列。
- **亮点/局限**：用运动学量（加速度）作为接触状态的**免费监督信号**，避免引入完整物理仿真器的代价，比 CG-HOI 的显式 contact 建模更轻量。局限是"加速度—接触"推断只在慢速场景成立，快速动态交互下假设会失效（作者已明确限定）。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Di_PAMotion_Physics-Aware_Motion_Generation_for_Full-Body_Interaction_with_Multiple_Objects_CVPR_2026_paper.html

### ViHOI: Human-Object Interaction Synthesis with Visual Priors
- **作者/机构**：Songjin Cai, Linjie Zhong, Ling Guo, Changxing Ding（华南理工大学）
- **任务**：human-object interaction 生成
- **核心方法**：立论点是"物理约束很难仅用文字描述清楚"，因此提出新范式——从易获取的 **2D 图像**中抽取丰富交互先验。用大型 VLM 作为先验抽取引擎，采用 **layer-decoupled 策略**同时获得视觉先验与文本先验；再设计 **Q-Former based adapter** 把 VLM 的高维特征压缩成紧凑的 prior token，大幅便利 diffusion 模型的条件训练。训练时用数据集渲染出的 motion 图像确保视觉输入与运动序列严格语义对齐；推理时改用 text-to-image 模型合成的参考图像，以提升对未见物体与交互类别的泛化。
- **数据集/评测**：多个 benchmark 上 SOTA，泛化性显著更优。
- **亮点/局限**："训练用渲染图、推理用生成图"的设计巧妙绕开 HOI 配对数据稀缺问题，也让 T2I 模型的开放世界知识为 HOI 服务。局限是存在渲染图与生成图之间的域差距（domain gap）。
- **链接**：https://arxiv.org/abs/2603.24383

### Decoupled Generative Modeling for Human-Object Interaction Synthesis (DecHOI)
- **作者/机构**：N/A
- **任务**：human-object interaction 生成（长时程、动态场景）
- **核心方法**：批评既有方法需人工指定中间 waypoint，且把所有优化目标压在单一网络上，导致复杂度高、灵活性差，并产生人-物运动不同步或穿透等错误。DecHOI 把**路径规划与动作合成解耦**：轨迹生成器先在无需预设 waypoint 的情况下产出人与物的轨迹，动作生成器再以这些路径为条件合成细节运动。为提升接触真实感，采用**对抗训练**，判别器专注于**末端关节（distal joints）的动力学**。框架还能建模移动的交互对方，支持动态场景中的响应式长序列规划并保持 plan 一致性。
- **数据集/评测**：FullBodyManipulation、3D-FUTURE 两个 benchmark 上多数定量指标与定性评价均超越先前方法，感知实验同样更受偏好。
- **亮点/局限**："规划—执行"解耦是本届的共识性设计（另见 SceMoS、Music-to-Dance via Atomic Movements），此处的额外贡献是免 waypoint 与末端关节判别器。
- **链接**：https://arxiv.org/abs/2512.19049

### InterPhys: Physics-aware Human Motion Synthesis in a Dynamic Scene
- **作者/机构**：N/A
- **任务**：physics-based motion 合成（动态场景 human-scene / object interaction）
- **核心方法**：批评现有工作接触建模有限（通常只限于手部）因而生成物理不真实的运动。本文显式建模**人体相关力的全谱**：人-物、人-场景以及**体内动力学（internal body dynamics）**，并施加软物理约束维持**力与力矩平衡（force and torque balance）**。核心技术是新的**连续距离型力模型（continuous distance-based force model）**，把接触建模推广到任意曲面，从而不仅能刻画与静态环境的交互，还能刻画与动态移动物体的交互。
- **数据集/评测**：物理合理性显著提升，且对复杂场景泛化良好，为物理一致的人体运动生成设立了新 benchmark。
- **亮点/局限**：不依赖完整刚体仿真器而用可微的距离-力场实现"软物理"，是介于纯运动学生成与 RL 物理控制之间的折中路线，训练比 RL 稳定得多。
- **链接**：https://arxiv.org/abs/2605.01036

### OneHOI: Unifying Human-Object Interaction Generation and Editing
- **作者/机构**：N/A
- **任务**：human-object interaction 生成 + 编辑（统一）
- **核心方法**：指出 HOI 领域割裂为两个不相交家族：HOI 生成（从结构化 triplet 与 layout 合成，但无法整合 HOI 与仅物体实体的混合条件）与 HOI 编辑（用文本改交互，但难以解耦姿态与物理接触、难以扩展到多个交互）。OneHOI 用统一的 diffusion transformer 把两者归并为**单一条件去噪过程**，由共享的结构化交互表征驱动。核心 **R-DiT（Relational Diffusion Transformer）**通过角色感知与实例感知的 HOI token 建模动词中介的关系，配 layout-based **Action Grounding**、强制交互拓扑的 **Structured HOI Attention**，以及解耦多 HOI 场景的 **HOI RoPE**。以 modality dropout 在自建 **HOI-Edit-44K** 及 HOI / 物体中心数据集上联合训练。
- **数据集/评测**：支持 layout-guided、layout-free、任意 mask、混合条件控制，在 HOI 生成与编辑上均达 SOTA。
- **亮点/局限**：`<person, action, object>` triplet 作为统一结构化接口，配 HOI RoPE 解耦多交互实例，设计相当完整。需注意本文更偏图像/场景层面的 HOI 合成，与纯 3D motion 序列生成的评测口径不同。
- **链接**：https://arxiv.org/abs/2604.14062

### InterPrior: Scaling Generative Control for Physics-Based Human-Object Interactions
- **作者/机构**：Sirui Xu et al.（UIUC；InterMimic 同作者）
- **任务**：physics-based HOI、humanoid loco-manipulation 控制
- **核心方法**：立论是人类很少在"显式全身动作"层面规划交互——高层意图（如 affordance）定义目标，而协调的平衡、接触与操作应从底层物理与运动先验中**自然涌现**。InterPrior 通过大规模模仿预训练 + RL 后训练学习统一生成式控制器：先把 full-reference 模仿专家**蒸馏**成通用的、目标条件化的**变分策略**，该策略能从多模态观测与高层意图重建运动。由于大规模 HOI 的构型空间极大，蒸馏后的策略泛化不可靠，因此施加**带物理扰动的数据增强**，再做 **RL 微调**以提升在未见目标与初始状态上的能力。这些步骤把重建出的 latent skill 收敛到一个有效流形上，得到能超出训练数据泛化的 motion prior（例如与未见物体交互）。
- **数据集/评测**：验证了用户交互式控制的有效性，并展示真实机器人部署潜力。
- **亮点/局限**：是 InterMimic（CVPR 2025 Highlight）的规模化续作——从"单策略模仿多样 HOI"升级为"可生成、可泛化的 HOI motion prior"；"蒸馏 → 扰动增强 → RL 微调"三段式对 physics-based 生成很有借鉴意义。
- **链接**：https://arxiv.org/abs/2602.06035

### TeamHOI: Learning a Unified Policy for Cooperative Human-Object Interactions with Any Team Size
- **作者/机构**：Lionar et al.
- **任务**：多智能体协作 HOI、physics-based humanoid 控制
- **核心方法**：让**单一去中心化策略**处理任意数量协作智能体的 HOI。每个 agent 仅用局部观测，通过带 **teammate token** 的 Transformer 策略网络关注其他队友，从而在可变队伍规模下可扩展协调。为在协作 HOI 数据极度稀缺的情况下保证运动真实性，提出 **masked AMP（Adversarial Motion Prior）**策略：使用**单人**参考运动，但在训练中 mask 掉与物体交互的身体部位，被 mask 区域改由 task reward 引导，从而产生多样且物理合理的协作行为。另设计与队伍规模、物体形状无关的 **formation reward** 促进稳定搬运。
- **数据集/评测**：2 至 8 个 humanoid agent 的协作搬运任务、多种物体几何；单一策略即取得高成功率并在各种配置下表现出连贯协作。
- **亮点/局限**：masked AMP 用单人数据 bootstrap 多人协作先验，是绕过"协作 mocap 几乎不存在"的聪明办法；可变队伍规模的单策略泛化也很实用。局限是评测集中于搬运这一类任务。
- **链接**：https://arxiv.org/abs/2603.07988

### Stability-Driven Motion Generation for Object-Guided Human-Human Co-Manipulation
- **作者/机构**：N/A
- **任务**：human-human 协作操作（co-manipulation）
- **核心方法**：现有方法多为单角色场景或忽略**负载引发的动力学（payload-induced dynamics）**。提出 flow matching 框架：先由生成模型从物体的 **affordance 与空间配置**导出显式操作策略，用以引导 motion flow 走向成功操作；再设计**对抗式交互先验**促进自然的个体姿态与真实的人际交互；并把**稳定性驱动的仿真**嵌入 flow matching 过程，用基于采样的优化精修不稳定的交互状态，**直接调整向量场回归**以促成更有效的操作。
- **数据集/评测**：相比 SOTA 的 HOI 基线，接触精度更高、**穿透（penetration）更低**、分布保真度更好。
- **亮点/局限**：human-human-object 三方协作是相对空白的子方向，把"稳定性"作为可微/可采样目标注入向量场是本文特色。与 CVPR 2025 的 CORE4D 数据集在任务上呼应。
- **链接**：https://arxiv.org/abs/2604.20336

### Interact2Ar: Full-Body Human-Human Interaction Generation via Autoregressive Diffusion Models
- **作者/机构**：Pablo Ruiz-Ponce et al.（MixerMDM 同作者）
- **任务**：human-human interaction 生成（全身含手部）
- **核心方法**：指出既有方法因数据与学习复杂度限制往往**忽略手部运动**，且 diffusion 方法一次性生成整段序列，无法体现交互的反应性与自适应性。Interact2Ar 是**首个端到端文本条件自回归 diffusion 模型**用于全身人-人交互：通过专用的**并行分支**引入细致手部运动学，实现高保真全身生成；配合自回归 pipeline 与新的 **memory 技术**，用高效大上下文窗口适应交互的内在可变性。
- **数据集/评测**：另提出一组鲁棒 evaluator 与扩展指标专门评估全身交互。支持时序 motion 组合、对扰动的实时适应，并从 dyadic 扩展到多人场景，达 SOTA。
- **亮点/局限**：InterGen / Inter-X 之后，把交互生成从"身体骨架"推进到"含手部全身"且改为自回归以支持反应性，两点都切中要害。自建 evaluator 虽必要，但也使跨论文数值对比更困难。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Ruiz-Ponce_Interact2Ar_Full-Body_Human-Human_Interaction_Generation_via_Autoregressive_Diffusion_Models_CVPR_2026_paper.html

### Unified Number-Free Text-to-Motion Generation Via Flow Matching (UMF)
- **作者/机构**：N/A（githubhgh.github.io/umf）
- **任务**：多人 text-to-motion（人数无关 / number-free）
- **核心方法**：现有生成模型擅长固定人数但难以泛化到可变人数；已有方法用自回归递归生成，效率低且误差累积。**UMF（Unified Motion Flow）**含 **P-Flow（Pyramid Motion Flow）**与 **S-Flow（Semi-Noise Motion Flow）**，把 number-free 生成拆解为**单次（single-pass）motion prior 生成**阶段 + **多次（multi-pass）reaction 生成**阶段。用统一 latent 空间弥合异质 motion 数据集间的分布差距以实现统一训练。P-Flow 在不同噪声水平上以**层级分辨率**运作以降低计算开销；S-Flow 学习一个联合概率路径，自适应地同时执行 reaction 变换与上下文重建，缓解误差累积。
- **数据集/评测**：大量实验与 user study 证明 UMF 作为多人 motion 生成通才模型的有效性。
- **亮点/局限**：FreeMotion（ECCV 2024）之后 number-free 路线的 flow matching 升级版，"单次 prior + 多次 reaction"的分解比纯递归更稳，统一 latent 空间处理异质数据集也是务实设计。
- **链接**：https://arxiv.org/abs/2603.27040

### SceMoS: Scene-Aware 3D Human Motion Synthesis by Planning with Geometry-Grounded Tokens
- **作者/机构**：Anindita Ghosh et al.（anindita127.github.io/SceMoS）
- **任务**：scene-aware text-to-motion（human-scene interaction）
- **核心方法**：核心主张相当反直觉且实用——**结构化 2D 场景表征可以替代完整 3D 监督**。SceMoS 把全局规划与局部执行解耦：(1) 文本条件的**自回归全局运动规划器**，在从场景高处角落渲染的 **BEV（鸟瞰）图像**上操作，用 **DINOv2** 特征编码作为场景表征；(2) **几何锚定的 motion tokenizer**，通过 conditional VQ-VAE 训练，使用 **2D 局部场景高度图（heightmap）**，从而把表面物理直接嵌入离散词表。BEV 语义负责空间布局与 affordance 的全局推理，局部 heightmap 负责细粒度物理贴合，无需完整 3D 体素推理。
- **数据集/评测**：**TRUMANS** benchmark 上运动真实性与接触精度达 SOTA，同时**场景编码的可训练参数量减少 50% 以上**。
- **亮点/局限**：省掉点云/体素占据网格而只用 BEV + heightmap，工程成本与泛化性双赢，对没有高质量 3D 扫描的场景尤其有价值。局限是 2D 高度图无法表达悬空结构等复杂拓扑。
- **链接**：https://arxiv.org/abs/2602.20476

### HSI-GPT2: A Dual-Granularity Large Motion Reasoning Model with Diffusion Refinement for Human-Scene Interaction
- **作者/机构**：Wang et al.（HSI-GPT 同团队）
- **任务**：human-scene interaction 理解 + 生成（统一 motion LLM）
- **核心方法**：诊断前作 HSI-GPT 的三个短板：单粒度 codebook 过度强调低频运动细节而忽视运动语义；motion detokenizer 解码能力有限，限制 HSI 保真度；仅靠 SFT 无法捕捉高层语义与逻辑推理。HSI-GPT2 是推理增强的双粒度大型 Scene-Motion-Language 模型，以 **RL + Chain-of-Thought** 驱动：**DMoTok（Dual-granularity Motion Tokenizer）**同时保留细粒度运动细节与文本对齐的运动语义；**motion diffusion decoder** 作为 detokenizer，把 LLM 的深层语义与细节特征翻译为物理接地的人体运动；并构建 **MoCoT（Motion Chain-of-Thought）**数据引擎，扩展 **GRPO** 范式以执行长时程、组合丰富的指令。
- **数据集/评测**：标准 HSI benchmark 上在交互质量、语义对齐、行为多样性以及对未见 3D 场景的泛化上均明显占优。
- **亮点/局限**：把 GRPO + CoT 这套 LLM 推理增强完整搬进 motion 领域，并用 diffusion decoder 补上离散 token 的保真度损失——正好回应 MARDM / DisCoRD 指出的 VQ 信息损失问题。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Wang_HSI-GPT2_A_Dual-Granularity_Large_Motion_Reasoning_Model_with_Diffusion_Refinement_CVPR_2026_paper.html

### DyaDiT: A Multi-Modal Diffusion Transformer for Socially Favorable Dyadic Gesture Generation
- **作者/机构**：N/A
- **任务**：dyadic（双人）co-speech gesture 生成
- **核心方法**：指出现有方法都是"单音频流 → 单说话人运动"，不考虑社会语境、也不建模两人对话的**相互动态（mutual dynamics）**。DyaDiT 是多模态 diffusion transformer，从 **dyadic 音频信号**生成语境适宜的人体运动：输入双人音频 + 可选的**社会语境 token**；融合双方说话人信息以捕捉交互动态；用 **motion dictionary** 编码运动先验；并可选地利用对话伙伴的手势以产生更具响应性的运动。
- **数据集/评测**：在 **Seamless Interaction Dataset** 上训练。标准运动生成指标 + 定量 user study，均超越既有方法且明显更受用户偏好。
- **亮点/局限**：把 co-speech gesture 从"独白"设定推进到"对话"设定，是该子方向的必要一步；显式的社会语境 token 提供了可控性。
- **链接**：https://arxiv.org/abs/2602.23165

### HandX: Scaling Bimanual Motion and Interaction Generation
- **作者/机构**：N/A
- **任务**：双手（bimanual）motion 与交互生成、数据集 + benchmark
- **核心方法**：指出全身模型常漏掉驱动灵巧行为的细粒度线索——手指关节运动、接触时序、双手协同，且缺高保真双手序列资源。HandX 提供数据、标注、评测三位一体的统一基础：整合并按质量过滤既有数据集，另采集针对代表性不足的双手交互（含细致手指动态）的新 mocap 数据集。标注上提出**解耦范式**：先抽取代表性运动特征（如接触事件、手指弯曲度），再用 LLM 推理生成与这些特征对齐的细粒度语义描述。基于此对 diffusion 与自回归模型在多种条件模式下做 benchmark。
- **数据集/评测**：提出**手部专属指标**；观察到清晰的 scaling 趋势——更大模型 + 更大更高质量数据 → 语义更连贯的双手运动。
- **亮点/局限**：手部与全身的保真度差距是 whole-body motion 的长期短板（HumanTOMATO、Motion-X 之后仍未解决），HandX 从数据与指标两端补位。"先抽特征再让 LLM 描述"的解耦标注法可复用性强。
- **链接**：https://arxiv.org/abs/2603.28766

### Superman: Unifying Skeleton and Vision for Human Motion Perception and Generation
- **作者/机构**：N/A
- **任务**：统一 motion 感知（视频 → 3D pose）+ 生成（预测、in-betweening）
- **核心方法**：指出三重割裂：感知模型能从视频理解运动但只输出文本，生成模型无法从原始视觉输入感知；生成式 MLLM 多局限于单帧静态 SMPL 姿态，无法处理时序运动；现有 motion 词表仅由骨架数据构建，切断了与视觉域的联系。Superman 的方案有两点：**Vision-Guided Motion Tokenizer**——利用 3D 骨架与视觉数据的天然几何对齐，从两种模态**联合学习**，构建统一的跨模态 motion 词表；在此"运动语言"之上训练单一统一 MLLM 架构处理所有任务，把视频 3D 骨架姿态估计（感知）与骨架运动预测、in-betweening（生成）统一。
- **数据集/评测**：Human3.6M 等标准 benchmark 上跨所有 motion 任务达 SOTA 或有竞争力。
- **亮点/局限**："用视觉引导 motion tokenizer 的训练"是本文最有意思的点——为 token 空间同时注入视觉与几何结构，思路可延伸到其他跨模态 tokenizer。定位骨架而非 SMPL，也让方案更轻更可扩展。
- **链接**：https://arxiv.org/abs/2602.02401

### Geometric Neural Distance Fields for Learning Human Motion Priors (NRMF)
- **作者/机构**：N/A
- **任务**：motion prior、去噪、in-betweening、稀疏观测拟合
- **核心方法**：提出 **Neural Riemannian Motion Fields**——不同于 VAE 或 diffusion，这是一个**高阶（higher-order）**运动先验，把人体运动显式建模为一组神经距离场（NDF）的**零水平集（zero level set）**，分别对应 pose、transition（速度）、acceleration 动力学。框架在数学上严谨：NDF 构建在关节旋转、角速度、角加速度的**乘积空间**上，尊重底层关节结构的几何。另贡献两项：(i) 投影到合理运动集合上的**自适应步长混合算法**；(ii) 在 test-time optimization 与生成中"roll out"真实运动轨迹的**几何积分器（geometric integrator）**。
- **数据集/评测**：在 **AMASS** 上训练，跨多种输入模态与任务（去噪、in-betweening、拟合部分 2D/3D 观测）均有显著一致增益。
- **亮点/局限**：把黎曼几何严格引入 motion prior，相比在欧氏空间粗暴处理旋转的做法在理论上更干净；高阶（含加速度）建模天然利于物理合理性。局限是作为先验而非条件生成器，不直接做 text-to-motion。
- **链接**：https://arxiv.org/abs/2509.09667

### Text-Driven 3D Hand Motion Generation from Sign Language Data (HandMDM)
- **作者/机构**：N/A
- **任务**：文本驱动 3D 手部运动生成
- **核心方法**：目标是训练以自然语言描述（手形 handshape、位置、手指/手/手臂运动等特征）为条件的 3D 手部运动生成模型。关键在数据构建：以前所未有的规模自动构建 3D 手部运动与文本标签的配对——利用大规模手语视频数据集及带噪的伪标注手语类别，通过一个使用**手语属性词典**的 LLM 把类别翻译成手部运动描述，并辅以互补的 motion-script 线索。由此训练文本条件手部运动 diffusion 模型 **HandMDM**。
- **数据集/评测**：跨域鲁棒——对同一手语中的未见类别、另一种手语的手势、以及非手语手部运动均能泛化；作者将公开模型与数据。
- **亮点/局限**：巧妙地把手语视频这一"现成大规模手部运动语料"转化为 text-to-hand-motion 的训练资源，思路可迁移到其他缺数据的细粒度部位。局限是伪标注噪声与 2D→3D 恢复精度。
- **链接**：https://arxiv.org/abs/2508.15902

### SignPR: A Progressive Vector-Quantized Diffusion Framework for Sign Language Production
- **作者/机构**：Liu et al.
- **任务**：sign language production（Text2Pose）
- **核心方法**：针对手语姿态序列与口语文本在语法规则与模态上的差异，提出联合建模签名的结构与时序属性的渐进式 diffusion 框架。**结构上**做渐进结构精修：structural VQVAE 把每帧编码为语义感知、基于区域的离散表征；diffusion 过程先产出语义一致的姿态，再在文本与语义条件下逐步精修运动细节。**时序上**引入 **block-wise causal diffusion**，渐进强化时间连贯性并支持对更早已生成片段的迭代精修，从而过渡更平滑、抖动更少。
- **数据集/评测**：常用数据集上多项指标优于先前 T2P 方法，姿态序列语义忠实、运动准确、时间连贯。
- **亮点/局限**：block-wise causal diffusion 允许回改历史片段，是流式与全序列之间的折中，比纯因果自回归更利于长序列质量。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Liu_SignPR_A_Progressive_Vector-Quantized_Diffusion_Framework_for_Sign_Language_Production_CVPR_2026_paper.html

### Focal-General Diffusion Model with Semantic Consistent Guidance for Sign Language Production (FGDM)
- **作者/机构**：Yu et al.（yuyiheng-eu.github.io/fgdm）
- **任务**：sign language production（Gloss2Pose）
- **核心方法**：现有 G2P 方法把每个 pose 当作不可分割的整体，无法捕捉细粒度关节级依赖。FGDM 采用**两阶段去噪**框架协调局部关节依赖与全局连贯：**Focal 阶段**用新的 **ASGCN（Adaptive Sign GCN）**基于上下文相关性、骨骼拓扑与语义条件自适应地建模每个 pose，确保局部细节精确；**General 阶段**用 Transformer 模块精修整个 pose 序列以增强全局连贯与自然度。另引入 **SCG（Semantic Consistent Guidance）**把语义监督无缝融入 diffusion 训练，强化生成 pose 序列与目标 gloss 语义的对齐。
- **数据集/评测**：PHOENIX14T、USTC-CSL 上达 SOTA。
- **亮点/局限**："局部 GCN + 全局 Transformer"的两阶段分工与 ParTY 的 part→holistic 思路同源，在手语这种高度依赖精确手形的任务上尤为必要。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Yu_Focal-General_Diffusion_Model_with_Semantic_Consistent_Guidance_for_Sign_Language_CVPR_2026_paper.html

### Pressure2Motion: Hierarchical Human Motion Reconstruction from Ground Pressure with Text Guidance
- **作者/机构**：N/A
- **任务**：motion capture（地面压力 + 文本 → motion）
- **核心方法**：推理时只需一块**压力垫**，无需特殊光照、相机或穿戴设备，适合隐私保护、低光、低成本 mocap。任务因压力信号相对全身运动严重不定（ill-posed）而困难，故用文本 prompt 作为高层引导约束消歧。模型采用**双层特征抽取器**精确解读压力数据，接**层级 diffusion 模型**分别辨识大尺度运动轨迹与细微姿态调整；压力序列提供的物理线索与文本描述提供的语义引导共同精确引导运动估计。
- **数据集/评测**：建立 **MPL benchmark**（该新任务的首个 benchmark）；生成高保真、物理合理的运动，达 SOTA。
- **亮点/局限**："压力 + 语言先验"的组合是首创，隐私保护 mocap 有真实需求。与 CVPR 2025 的 MotionPro、CVPR 2024 的 MMVP 共同构成"压力传感 mocap"这条新兴线。
- **链接**：https://arxiv.org/abs/2511.05038

### Gaussian-Mixture Latent Flow for Stochastic 3D Human Motion Prediction
- **作者/机构**：Ma et al.
- **任务**：随机（stochastic）3D 人体运动预测
- **核心方法**：指出现有工作虽在精度与多样性上表现好，却忽视了**合理性（plausibility）**（如产生物理不真实的预测）与**不确定性量化**。提出基于 latent flow 的模型，配备**数据驱动的高斯混合（Gaussian mixture）先验**，相比传统单模先验能更有效地解耦多样的人类行为；该先验直接从训练数据的模式中导出，无需额外标注。模型**完全可逆（fully invertible）**的特性使得可通过可计算的似然（tractable likelihood）自然进行不确定性量化。
- **数据集/评测**：Human3.6M、AMASS 上在精度与合理性上均达 SOTA，同时提供可靠的不确定性估计。
- **亮点/局限**：normalizing flow 的可逆性带来精确似然，这是 diffusion / VAE 都做不到的——对需要置信度的下游应用（自动驾驶、人机协作）价值明确。多模先验也比标准高斯更契合人类行为的多模态本质。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Ma_Gaussian-Mixture_Latent_Flow_for_Stochastic_3D_Human_Motion_Prediction_CVPR_2026_paper.html

### HUMAPS-4D: A Multimodal Dataset for Human Motion Analysis with Physiological and Semantic Information
- **作者/机构**：Dabrowski et al.
- **任务**：数据集（多模态 motion 分析，生理信号 + 语义）
- **核心方法**：出于隐私法规与操作限制对视觉数据使用的日益收紧，主张用穿戴传感器（如测足底激活的智能鞋垫）推断姿态。HUMAPS-4D 整合同步的 **mocap、多视角视频、IMU、足底压力信号、sEMG 激活模式**与高层语义标注。数据来自 **32 位受试者**执行 **30 种动作**，总时长 **14 小时**；受试者在人体测量学上（年龄、身体比例、形态）差异显著，支持跨体型的鲁棒泛化。
- **数据集/评测**：建立四类 benchmark 任务：足底压力姿态重建、语义运动分割、物理信息驱动的运动学分析、隐私保护条件下的多模态融合。
- **亮点/局限**：**低层生理信号与高层运动描述符的配对**是独特之处，可训练同时受物理与语义约束的生成/推断模型。对做 physics-aware 或体型感知生成（Odoriko、HUMOS 方向）是有价值的新数据源。局限是规模仍属中小（14 小时）。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Dabrowski_HUMAPS-4D_A_Multimodal_Dataset_for_HUman_Motion_Analysis_with_Physiological_CVPR_2026_paper.html

### Beyond Mimicry: Learning Whole-Body Human-Humanoid Interaction from Human-Human Demonstrations
- **作者/机构**：Huang et al.
- **任务**：human-humanoid interaction（HHoI）、humanoid 全身控制
- **核心方法**：面对 HHoI 数据稀缺，尝试用丰富的 HHI（human-human interaction）数据替代，但首先证明**标准 retargeting 会破坏关键接触而失败**。对策是 **PAIR（Physics-Aware Interaction Retargeting）**——以接触为中心的两阶段 pipeline，跨形态差异保留接触语义，生成物理一致的 HHoI 数据。高质量数据又暴露第二个失败模式：常规模仿学习策略只是复制轨迹，缺乏交互理解。故提出 **D-STAR（Decoupled Spatio-Temporal Action Reasoner）**分层策略，解耦"何时行动"与"何处行动"：**Phase Attention**（when）与 **Multi-Scale Spatial 模块**（where）由 diffusion head 融合，产出同步的全身行为。
- **数据集/评测**：大量严格仿真验证，相比基线有显著性能提升，形成从 HHI 数据学习复杂全身交互的完整有效 pipeline。
- **亮点/局限**：两个"先证明失败再对症下药"的环节让论文逻辑很扎实；when / where 解耦对反应式协作确实关键。局限是仅在仿真中验证，未见真机结果。
- **链接**：https://openaccess.thecvf.com/content/CVPR2026/html/Huang_Beyond_Mimicry_Learning_Whole-Body_Human-Humanoid_Interaction_from_Human-Human_Demonstrations_CVPR_2026_paper.html

### Iterative Closed-Loop Motion Synthesis for Scaling the Capabilities of Humanoid Control
- **作者/机构**：N/A
- **任务**：physics-based humanoid 控制、motion 数据合成
- **核心方法**：指出数据集**固定的难度分布**限制了控制策略的性能上限，而专业 mocap 获取高质量数据受成本约束难以规模化。提出闭环自动化 motion 数据生成与迭代框架，可生成动作语义丰富的高质量数据（武术、舞蹈、格斗、体育、体操等）。框架通过物理指标与客观评估实现**策略与数据的难度迭代**，让训练出的 tracker 突破原有难度上限。
- **数据集/评测**：在 PHC 单 primitive tracker 上，仅用约 **1/10 的 AMASS 数据量**，测试集（**2201** 个 clip）平均失败率相比基线**降低 45%**。
- **亮点/局限**："难度课程 + 闭环数据生成"共同演进的思路，把 self-play / 课程学习引入 humanoid motion 数据，数据效率提升（1/10 数据、失败率降 45%）相当可观。
- **链接**：https://arxiv.org/abs/2602.21599
# ICLR

> 数据来源：ICLR 官方 proceedings（openreview.net / iclr.cc）、mlanthology.org、Awesome-Human-Motion 社区仓库交叉校验、arXiv API 摘要抓取。所有条目的会议归属均以 OpenReview 录用页面或作者声明为准。
> 指标说明：FID / R-Precision / MM-Dist / Diversity / MModality 均指 HumanML3D 或 KIT-ML 标准评测协议下的数值，除特别说明外 FID 越低越好、R-Precision 越高越好。

### ICLR 收录一览

| 年份 | 收录论文 | 核心子方向 |
|------|---------|-----------|
| **ICLR 2026** | KinemaDiff、Text2Interact、Unleashing Guidance (HOI) | physics-aware diffusion、双人交互生成、HOI guidance mechanism |
| **ICLR 2025** | CLoSD、PedGen、HGM³、LaMP、MotionDreamer、Lyu et al. (Sparse Interpretable)、DART、Motion-Agent、Think-Then-React、Ready-to-React、InterMask | simulation-diffusion 闭环控制、web-scale 弱监督行人运动、masked modeling 改进、language-motion 统一预训练、one-to-many 多样性合成、diffusion-AR hybrid 实时控制、LLM 对话框架、action-reaction reasoning、online interaction |
| **ICLR 2024** | SinMDM（spotlight）、NeRM、PriorMDM、OmniControl、Bidirectional Temporal Diffusion、Duolando | single-instance diffusion、neural implicit motion representation、composition-as-prior、joint-level 精细控制、双向时间扩散、dance accompaniment + off-policy RL |

---

## ICLR 2026

### KinemaDiff: Towards Diffusion for Coherent and Physically Plausible Human Motion Prediction
- **作者/机构**：Ye Lu, Jie Wang, Tianyi Liu, Jianjun Gao, Kim-Hui Yap（N/A）
- **任务**：随机人体运动预测（Stochastic Human Motion Prediction, HMP），强调物理合理性与骨骼一致性
- **核心方法**：提出一种结构对齐且关节感知的扩散框架，将骨骼拓扑和关节特异性动力学直接嵌入扩散过程。包含两个关键模块：(1) **Joint-Adaptive Noise Generator**——为每个关节和样本推断异质性的实例感知噪声，捕捉空间变异性并增强运动多样性；(2) **Structure-Aligned Constraint Enforcer**——通过建模历史运动的关节连接性和骨长来编码骨骼拓扑，约束每个去噪步骤以保持解剖学一致性。与以往仅在网络架构中隐式编码结构先验的方法不同，KinemaDiff 在扩散过程的每一步都显式施加物理约束。
- **数据集/评测**：在多个 HMP benchmark 上验证有效性（具体数字待补充）。
- **亮点/局限**：从"间接结构先验 + 均匀噪声"转向"直接物理约束 + 关节自适应噪声"，是 diffusion-for-prediction 路线上对物理合理性的一次系统性升级。但该方法聚焦 motion prediction 而非 generation，与 text-to-motion 主线的关联度有限。
- **链接**：https://mlanthology.org/iclr/2026/lu2026iclr-kinemadiff

### Text2Interact: High-Fidelity and Diverse Text-to-Two-Person Interaction Generation
- **作者/机构**：Qingxuan Wu, Zhiyang Dou, Chuan Guo, Yiming Huang, Qiao Feng, Bing Zhou, Jian Wang, Lingjie Liu（Meta Reality Labs / NUS 等）
- **任务**：文本驱动的两人交互运动生成（text-to-two-person interaction）
- **核心方法**：针对双人交互生成的多样性和保真度难题，提出统一的扩散框架。关键设计包括：(1) 解耦的单人运动表征与交互条件融合机制；(2) 基于文本语义的交互类型自适应采样策略；(3) 接触一致性约束确保两人交互的物理合理性。相比 InterGen / InterDiff 等早期工作，Text2Interact 在交互多样性和接触质量上有显著提升。
- **数据集/评测**：在 Inter-X / 3DPW 等双人交互数据集上验证，FID / Diversity 等指标优于 prior work（具体数字待补充）。
- **亮点/局限**：Meta Reality Labs + CVA 团队的联合工作，延续了 Chuan Guo 组在 motion generation 方向的系统布局。双人交互的文本可控性是该方向的核心难点。
- **链接**：https://ericguo5513.github.io/ （作者主页公布）

### Unleashing Guidance Without Classifiers for Human-Object Interaction Animation
- **作者/机构**：Ziyin Wang, Sirui Xu, Chuan Guo, Bing Zhou, Jian Wang, Yu-Xiong Wang, Liang-Yan Gui（Meta / UIUC / SEU）
- **任务**：人-物交互（HOI）动画生成
- **核心方法**：提出一种无需分类器的 guidance 机制用于 HOI 动画。核心洞察是：传统 classifier-free guidance 在 HOI 场景下难以同时优化运动质量和交互合理性。该方法通过重新设计 guidance 信号的计算方式，在不引入额外分类器的前提下实现更强的条件控制。与 CORE4D / HUMOTO 等近期 HOI benchmark 形成互补。
- **数据集/评测**：在 HOI 标准数据集上验证（具体数字待补充）。
- **亮点/局限**：同样来自 Chuan Guo 团队，显示该组在 ICLR 2026 上的集中发力。guidance mechanism 的改进对 diffusion-based HOI 生成有普适价值。
- **链接**：https://ericguo5513.github.io/

---

## ICLR 2025

### CLoSD: Closing the Loop between Simulation and Diffusion for Multi-Task Character Control
- **作者/机构**：Guy Tevet, Sigal Raab, Setareh Cohan, Daniele Reda, Zhengyi Luo, Xue Bin Peng, Amit Bermano, Michiel van de Panne（Tel Aviv University / UBC / NVIDIA）
- **任务**：physics-based character control（多任务角色控制）
- **核心方法**：将 motion diffusion 与 RL physics-based controller 闭环结合。架构包含两个模块：(1) **Diffusion Planner (DiP)**——一个快速响应的自回归扩散模型，根据文本提示和目标位置生成短期动作序列；(2) **RL Tracking Controller**——简单的 motion imitator，持续接收 DiP 的动作计划并提供环境反馈。关键洞察是 motion diffusion 可以作为 robust RL controller 的 on-the-fly universal planner。系统支持动态调整 diffusion 与仿真器的贡献权重。
- **数据集/评测**：在多任务 benchmark 上验证，涵盖 navigation、object striking（hand/foot）、sit-down/stand-up 等任务。相比纯 RL 或纯 diffusion 基线，CLoSD 在任务成功率和运动自然度上均有优势。
- **亮点/局限**：Tevet（MDM 一作）+ Xue Bin Peng（HumanoidVerse / AMP 作者）+ van de Panne 的强强联合。把 diffusion 从"离线生成器"升级为"在线规划器"的思路非常优雅，是 simulation-diffusion 闭环控制的代表作。局限在于推理时需要同时运行 diffusion + RL controller + 物理仿真，计算开销较大。
- **链接**：https://guytevet.github.io/CLoSD-page/ | https://iclr.cc/virtual/2025/poster/28291

### PedGen: Learning to Generate Diverse Pedestrian Movements from Web Videos with Noisy Labels
- **作者/机构**：Zeshun Liu 等（GenForce / N/A）
- **任务**：从 web 视频中以弱监督方式学习行人运动生成
- **核心方法**：利用海量 web 视频中的行人数据，在 noisy label 设定下训练行人运动生成模型。关键技术包括：(1) 从 2D 视频中提取伪 3D 运动标注的 pipeline；(2) 针对 label noise 的鲁棒训练策略；(3) 行人运动特有的多样性建模。与 PACER+ 等依赖 mocap 数据的方法形成互补，探索了 web-scale 数据在行人领域的 scaling 路径。
- **数据集/评测**：在行人运动 benchmark 上验证，展示了 web 数据带来的多样性提升。
- **亮点/局限**：web-scale weakly-supervised learning 是 motion 领域的重要趋势，PedGen 将其引入行人领域。noisy label 的处理是该工作的技术核心。
- **链接**：https://genforce.github.io/PedGen/

### HGM³: Hierarchical Generative Masked Motion Modeling with Hard Token Mining
- **作者/机构**：Minjae Jeong, Yechan Hwang, Jaejin Lee, Sungyoon Jung, Won Hwa Kim（UNIST / POSTECH）
- **任务**：text-to-motion generation
- **核心方法**：在 MoMask（CVPR 2024）masked modeling 基础上提出两项改进：(1) **Hard Token Mining (HTM)**——识别并重点掩码运动序列中难以学习的区域，引导模型关注困难组件；(2) **Hierarchical Generative Masked Model**——使用语义图在不同粒度上表示句子，使模型在不同条件级别下重建同一序列，促进复杂运动模式的全面学习。推理时采用 coarse-to-fine 渐进生成策略。
- **数据集/评测**：HumanML3D 和 KIT-ML 上均优于 prior methods。在 FID 和 R-Precision 上取得显著提升（具体数字需查原文）。
- **亮点/局限**：把 curriculum learning（hard example mining）的思想引入 masked motion modeling，简单有效。层级化语义图的设计让模型能学到从粗到细的运动结构。
- **链接**：https://mlanthology.org/iclr/2025/jeong2025iclr-hgm3

### LaMP: Language-Motion Pretraining for Motion Generation, Retrieval, and Captioning
- **作者/机构**：Zhe Li, Weihao Yuan, Yisheng He, Lingteng Qiu, Shenhao Zhu, Xiaodong Gu, Weichao Shen, Yuan Dong, Zilong Dong, Laurence T. Yang（华中科技大学 / Alibaba 等）
- **任务**：language-motion 统一预训练，覆盖 text-to-motion 生成、motion-text 检索、motion captioning 三个任务
- **核心方法**：指出 CLIP text embedding 在 motion 领域的语义对齐不足（因为 CLIP 在静态 image-text 对上预训练），提出从 language-vision 空间转向 language-motion latent 空间。具体设计：(1) 生成 motion-informative text embedding 替代 CLIP 作为文本条件；(2) 自回归 masked prediction 避免 Transformer 中的 rank collapse；(3) motion transformer 与 text transformer 通过 query token 交互实现双向检索；(4) 微调 LLM 配合 motion 特征做 captioning。还提出了 **LaMP-BertScore** 指标评估 motion-text 对齐质量。
- **数据集/评测**：在 HumanML3D、KIT-ML 等多个数据集上，三个任务均显著优于 prior methods。
- **亮点/局限**：把 motion generation/retrieval/captioning 三个任务统一到同一个 pretraining 框架下，是"motion foundation model"思路的代表作之一。LaMP-BertScore 指标也有方法论价值。与 MotionGPT / AvatarGPT 等 LLM-based 方法相比，LaMP 更强调语言-运动潜在空间的专门化对齐而非通用 LLM 微调。
- **链接**：https://github.com/thinglinks/LaMP | arXiv:2410.07093

### MotionDreamer: One-to-Many Motion Synthesis with Localized Generative Masked Transformer
- **作者/机构**：N/A
- **任务**：one-to-many motion synthesis（给定部分条件生成多样化运动）
- **核心方法**：提出 localized generative masked transformer，通过在 transformer 中引入局部化掩码策略，实现对同一条件的多样化运动合成。与 MoMask 的全局 masked modeling 不同，MotionDreamer 侧重在局部时间窗口内进行多样性采样。
- **数据集/评测**：在标准 benchmark 上验证 one-to-many 生成的多样性和质量。
- **亮点/局限**：one-to-many 生成是 motion diversity 的核心挑战之一，localized masking 是一个简洁的技术方案。
- **链接**：https://openreview.net/forum?id=d23EVDRJ6g

### Towards Unified Human Motion-Language Understanding via Sparse Interpretable Characterization
- **作者/机构**：Lyu et al.
- **任务**：human motion-language 统一理解
- **核心方法**：提出 sparse interpretable characterization 方法，将运动序列映射到稀疏可解释的语义空间，实现 motion 与 language 的统一表征。与 LaMP 的 dense embedding 路线不同，该工作强调表征的可解释性。
- **数据集/评测**：在 motion-language 理解任务上验证。
- **亮点/局限**：可解释性是 motion-language 模型的重要但未充分探索的方向。
- **链接**：https://openreview.net/forum?id=Oh8MuCacJW

### DART: A Diffusion-Based Autoregressive Motion Model for Real-Time Text-Driven Motion Control
- **作者/机构**：Zhao et al.
- **任务**：real-time text-driven motion control
- **核心方法**：将 diffusion 与 autoregressive 建模结合，实现实时文本驱动的运动控制。核心设计是在自回归框架的每一步用 diffusion 进行去噪 refinement，兼顾 AR 的 streaming 能力和 diffusion 的生成质量。与 BAD（bidirectional auto-regressive diffusion）思路相近但更侧重实时控制场景。
- **数据集/评测**：在 HumanML3D 等数据集上验证实时性能和生成质量。
- **亮点/局限**：streaming / real-time generation 是 2025 年的热点方向，DART 是较早探索 diffusion-AR hybrid 的工作之一。
- **链接**：https://zkf1997.github.io/DART/

### Motion-Agent: A Conversational Framework for Human Motion Generation with LLMs
- **作者/机构**：Qi Wu, Yubo Zhao, Yifan Wang, Xinhang Liu, Yu-Wing Tai, Chi-Keung Tang（HKUST）
- **任务**：conversational motion generation / editing / understanding
- **核心方法**：提出 Motion-Agent 对话框架，核心是 **MotionLLM**——将 motion 编码并量化为离散 token 对齐到 LLM 的词表，仅用 1-3% 参数通过 adapter 微调即可达到与从头训练的 diffusion/transformer 相当的性能。关键特性：(1) 集成 MotionLLM 与 GPT-4 无需额外训练即可通过多轮对话生成复杂运动序列；(2) 支持 motion generation、editing、understanding 多种任务的统一接口；(3) 仅需少量 adapter 参数，大幅降低训练成本。
- **数据集/评测**：在 HumanML3D 标准评测上 FID/R-Precision 与 SOTA diffusion 方法相当。对话生成能力通过 qualitative evaluation 验证。
- **亮点/局限**：把 motion generation 从"单轮 prompt → 输出"升级为"多轮对话交互"，是人机交互范式的重要演进。1-3% 参数微调即可匹敌从头训练的结果非常有吸引力。局限在于对 GPT-4 API 的依赖增加了部署成本。
- **链接**：https://mlanthology.org/iclr/2025/wu2025iclr-motionagent | https://github.com/wangyifanust/Motion-Agent

### Think-Then-React: Towards Unconstrained Human Action-to-Reaction Generation
- **作者/机构**：Wenhui Tan, Boyuan Li, Chuhao Jin, Wenbing Huang, Xiting Wang, Ruihua Song（中国人民大学高瓴人工智能学院）
- **任务**：unconstrained online action-to-reaction generation（无人机交互反应生成）
- **核心方法**：提出 **Think-Then-React (TTR)** 框架，核心是 LLM + motion encoder 的协同：(1) **Thinking**——模型观察输入动作并生成文本描述（如"对方伸手要握手"），作为语义锚点；(2) **Reacting**——基于这个"思考结果"生成对应的反应 token；(3) **Re-thinking**——每约 0.5 秒（4 tokens）重新思考一次，避免跟踪丢失。配套设计包括：**Decoupled Motion Tokenization**——将 egocentric pose（VQ-VAE 离散化）与 absolute space（2D position + orientation）解耦编码，解决多人交互中的相对位置问题。**Motion-Text Joint Pretraining**——一系列 motion-text 相关预训练任务提升多模态理解。
- **数据集/评测**：在 Inter-X 数据集（~9K action-reaction pairs）上，FID 从 ReGenNet 的 **3.988** 降至 **1.942**，Semantic Stability 实验中无 thinking 阶段时 FID 几乎翻倍。每 0.5 秒 re-thinking 是最佳平衡点。
- **亮点/局限**：人大 AIMind 团队的工作，把 LLM 的 reasoning 能力引入 action-reaction 生成，"先理解意图再反应"的思路非常符合人类社交认知。FID 51% 的降幅非常显著。局限在于对 Inter-X 这种带文本标注的数据集有较强依赖。
- **链接**：https://think-then-react.github.io/ | https://openreview.net/forum?id=UxzKcIZedp

### Ready-to-React: Online Reaction Policy for Two-Character Interaction Generation
- **作者/机构**：Cen et al.（ZJU）
- **任务**：online two-character interaction generation
- **核心方法**：提出 online reaction policy，能够在观察到对方动作的部分序列时就实时生成反应。与离线生成方法不同，Ready-to-React 强调因果性和低延迟响应，技术上可能结合了 causal attention 和 predictive modeling。
- **数据集/评测**：在双人交互数据集上验证 online 性能。
- **亮点/局限**：online / real-time 交互生成是机器人和人机交互的关键需求，该工作与 Think-Then-React 形成互补——TTR 侧重语义 reasoning，Ready-to-React 侧重时序因果性。
- **链接**：https://zju3dv.github.io/ready_to_react/

### InterMask: 3D Human Interaction Generation via Collaborative Masked Modelling
- **作者/机构**：Gohar Malik et al.
- **任务**：3D human-human interaction generation
- **核心方法**：将 MoMask 式的 masked modeling 扩展到双人交互场景，提出 collaborative masked modelling 框架。关键设计是让两个角色的运动序列在同一 masked transformer 中联合建模，通过共享注意力机制学习交互协调性。
- **数据集/评测**：在 Inter-X / Hi4D 等交互数据集上验证。
- **亮点/局限**：把 masked modeling 从单人推广到多人交互，技术路线清晰。collaborative attention 的设计是核心创新。
- **链接**：https://gohar-malik.github.io/intermask

---

## ICLR 2024

### Single Motion Diffusion (SinMDM)
- **作者/机构**：Sigal Raab, Inbal Leibovitch, Guy Tevet, Moab Arar, Amit Bermano, Daniel Cohen-Or（Tel Aviv University）
- **任务**：single-instance motion learning（从单个运动序列学习内部 motif 并生成多样化新运动）
- **核心方法**：提出 SinMDM——首个 single motion diffusion model。核心设计：(1) 显式为 single-input 任务设计的去噪网络，采用浅层架构 + local attention 层缩小感受野，避免过拟合并鼓励多样性；(2) 支持任意拓扑（人类、动物、虚构生物）的 skeleton-agnostic 设计；(3) 推理时即支持多种应用（时空 in-betweening、motion expansion、style transfer、crowd animation）无需额外训练。
- **数据集/评测**：在多种 skeleton topology 上定性定量均优于 prior methods，user study 确认质量优势。
- **亮点/局限**：ICLR 2024 spotlight。把 diffusion 从"data-rich regime"推向"data-scarce regime"，single-instance learning 的设定在 motion 领域非常新颖。local attention + shallow network 的设计原则简单但有效。局限在于 single-instance 设定的适用范围相对窄。
- **链接**：https://sinmdm.github.io/SinMDM-page/ | https://iclr.cc/virtual/2024/19122

### NeRM: Learning Neural Representations for High-Framerate Human Motion Synthesis
- **作者/机构**：Wei et al.
- **任务**：high-framerate human motion synthesis
- **核心方法**：提出 neural representation 用于连续时间的人体运动合成。受 NeRF 启发，将运动序列建模为神经隐式函数，可在任意时间点查询姿态，实现高帧率渲染。与 Implicit Motion（ECCV 2022）思路相近但在网络架构和训练策略上有改进。
- **数据集/评测**：在高帧率合成任务上验证插值质量和时间连续性。
- **亮点/局限**：neural implicit representation 在 motion 领域的应用仍属早期探索，NeRM 是高帧率场景下的有益尝试。
- **链接**：https://openreview.net/forum?id=sOJriBlOFd

### PriorMDM: Human Motion Diffusion as a Generative Prior
- **作者/机构**：Yoni Shafir, Guy Tevet, Roy Kapon, Amit Haim Bermano（Tel Aviv University）
- **任务**：利用预训练 motion diffusion model 作为 generative prior 解决三类组合问题
- **核心方法**：提出三种基于 diffusion prior 的组合形式：(1) **Sequential Composition (DoubleTake)**——inference-time 方法，用仅训练短 clip 的 prior 生成长序列动画，包含 prompted intervals 及其过渡；(2) **Parallel Composition (ComMDM)**——两人交互生成，从两个固定 prior 出发加少量双人训练样本，学习 slim communication block 协调两个 motion；(3) **Model Composition (DiffusionBlending)**——先训练 individual priors 实现特定关节的 prescribed motion，再用 interpolation mechanism 混合多个模型实现灵活 fine-grained 控制。
- **数据集/评测**：在 HumanML3D、BABEL、3DPW 上验证三种 composition 模式，与 dedicated models trained for specific tasks 对比。
- **亮点/局限**：把 MDM 从"端到端生成器"重新定位为"composable prior"，思路优雅。DoubleTake 的 inference-time long sequence 方案尤其实用。ComMDM 只需少量双人数据即可实现交互，data efficiency 高。局限在于三种 composition 都需要精心设计的 blending/interpolation 机制，通用性受限。
- **链接**：https://priormdm.github.io/priorMDM-page/ | https://mlanthology.org/iclr/2024/shafir2024iclr-human

### OmniControl: Control Any Joint at Any Time for Human Motion Generation
- **作者/机构**：Yiming Xie, Varun Jampani, Lei Zhong, Deqing Sun, Huaizu Jiang（Northeastern / Google Research）
- **任务**：flexible spatial control for text-conditioned motion generation
- **核心方法**：提出 OmniControl，可将灵活的空间控制信号注入到 text-conditioned diffusion model 中。与之前只能控制 pelvis trajectory 的方法不同，OmniControl 可以：(1) 控制任意关节在任意时间步的位置；(2) 支持多关节同时控制；(3) 通过 cross-attention 机制将空间条件融入 diffusion backbone。技术上借鉴了 ControlNet 的思想但针对 motion 序列做了专门设计。
- **数据集/评测**：在 HumanML3D 上验证 joint-level control 的精度和灵活性，qualitative results 展示了对 end-effector、root trajectory 等的精确控制。
- **亮点/局限**：joint-level controllable generation 是 motion 领域长期存在的挑战，OmniControl 提供了目前最灵活的解决方案之一。与 ControlNet 的类比使其设计理念易于理解。局限在于需要为新的控制信号重新训练 adapter。
- **链接**：https://neu-vi.github.io/omnicontrol/

### Bidirectional Temporal Diffusion Model for Temporally Consistent Human Animation
- **作者/机构**：Adiya et al.
- **任务**：temporally consistent human animation
- **核心方法**：提出 bidirectional temporal diffusion model，在时间维度上进行双向扩散去噪以保证长序列的时间一致性。与单向 autoregressive diffusion 相比，双向设计能更好地捕捉长程时间依赖。
- **数据集/评测**：在长序列动画任务上验证时间一致性和运动质量。
- **亮点/局限**：bidirectional diffusion 在 motion 领域的应用，对长序列时间一致性有理论优势。
- **链接**：https://openreview.net/forum?id=yQDFsuG9HP

### Duolando: Follower GPT with Off-Policy Reinforcement Learning for Dance Accompaniment
- **作者/机构**：Li et al.
- **任务**：dance accompaniment（双人舞中 follower 的反应性舞蹈生成）
- **核心方法**：定义了一个新任务——dance accompaniment：follower 需要根据 leader 的动作和音乐节奏生成同步的响应动作。技术方案：(1) 构建 DD100 数据集（117 分钟专业舞者双人舞表演，67 个 choreography）；(2) 提出 Duolando——基于 GPT 的模型，autoregressively 预测 follower 的下一个 tokenized motion，条件包括音乐、leader 动作和 follower 自身历史；(3) **Off-Policy Reinforcement Learning**——允许模型从 out-of-distribution 采样中探索可行轨迹，由人工定义的 reward 引导。
- **数据集/评测**：发布 DD100 benchmark，设计了多项针对双人舞协调性的评价指标。
- **亮点/局限**：定义了 dance accompaniment 这个新任务并发布数据集，任务定义本身就有贡献。off-policy RL 用于提升 OOD 稳定性是有价值的技术选择。局限在于 DD100 规模相对较小（117 分钟），且依赖专业舞者采集。
- **链接**：https://lisiyao21.github.io/projects/Duolando/ | https://github.com/jzho987/Duolando

---

## 趋势小结（ICLR 线）

ICLR 在 motion generation 领域的特色鲜明：

1. **Physics + Diffusion 的深度融合**：CLoSD 是 simulation-diffusion 闭环控制的标杆，KinemaDiff 把物理约束直接嵌入扩散过程，代表了 ICLR 偏重"learning + control"的独特视角。
2. **LLM 赋能的 reasoning 路线**：Motion-Agent（对话框架）、Think-Then-React（先思考再反应）、LaMP（language-motion 预训练）都把 LLM 的语义推理能力引入 motion，这条线在 ICLR 2025 形成气候。
3. **Interaction 的系统性突破**：ICLR 2025 一口气收录了 Think-Then-React、Ready-to-React、InterMask 三篇多人交互论文，加上 ICLR 2026 的 Text2Interact 和 HOI guidance 工作，interaction 已成为 ICLR motion 线的核心子方向。
4. **Data-efficient / Single-instance 学习**：SinMDM（single motion）、PriorMDM（composition as prior）、PedGen（web-scale noisy labels）代表了 ICLR 对 data efficiency 的一贯关注，与 CVPR 偏向 data scaling 的路线形成有趣对照。
5. **Control 的精细化**：OmniControl 实现了"any joint at any time"的控制粒度，是 motion controllability 的里程碑工作。

---

# NeurIPS / ICML

> 数据来源：Awesome-Human-Motion 社区仓库（Foruck 维护）、NeurIPS proceedings (papers.nips.cc / openreview.net)、ICML proceedings (proceedings.mlr.press)、arXiv API 摘要抓取。会议归属以官方录用公告或 OpenReview 页面为准；无法确证者标 `（归属待核实）`。
> 指标说明：FID / R-Precision / MM-Dist / Diversity / MModality 均指 HumanML3D 或 KIT-ML 标准评测协议下的数值，除特别说明外 FID 越低越好、R-Precision 越高越好。

---

## NeurIPS 2025

### HMVLM: Human Motion-Vision-Language Model via MoE LoRA
- **作者/机构**：Lei Hu, Yongjing Ye, Shihong Xia（中科院自动化所等）
- **任务**：motion-vision-language 统一理解与生成（instruction tuning）
- **核心方法**：针对"把 motion 模态注入基础语言模型时的灾难性遗忘"问题，提出 **MoE LoRA** 策略——gating network 根据输入 prompt 动态分配多个 LoRA expert 权重，实现多任务同步微调。关键设计是 **zero expert**：保留一份未经 instruction-tuning 的预训练参数副本，专门处理通用语言任务，从而在引入 motion 能力时不损害原有语言能力。表征侧采用 **body-part-specific tokenization**：把人体分成若干关节组分别 tokenize，提升空间分辨率并保持与 AR 解码兼容。
- **数据集/评测**：在多个人类 motion 下游任务上验证，包括 text-to-motion generation、motion captioning、motion retrieval 等；报告显著缓解遗忘并提升跨任务性能。
- **亮点/局限**：把 MoE + LoRA 引入 motion LLM 是务实的工程创新——zero expert 的设计简单有效，直击 multimodal instruction tuning 的核心痛点。局限是 body-part tokenization 的具体分组策略与 codebook 规模未详述。
- **链接**：https://arxiv.org/abs/2511.01463

### TransPhase: Deep Compositional Phase Diffusion for Long Motion Sequence Generation
- **作者/机构**：Ho Yin Au, Jie Chen, Junkun Jiang, Jingyu Xiang（香港浸会大学）
- **任务**：长序列 compositional motion 生成（多段语义动作拼接）
- **核心方法**：现有 T2M 模型在拼接多段语义生成的 clips 时过渡处会出现动力学不连续。提出 **Compositional Phase Diffusion**，在预训练的 **ACT-PAE（Action-Centric Motion Phase Autoencoder）** 建立的 latent motion frequency domain 上操作，包含两个模块：**SPDM（Semantic Phase Diffusion Module）** 从相邻 clips 渐进注入语义引导信息，**TPDM（Transitional Phase Diffusion Module）** 注入过渡处的相位细节。本质是在频率域同时建模"语义一致性"和"相位连续性"。固定输入序列的 phase parameter 还可做 motion inbetweening。
- **数据集/评测**：HumanML3D / KIT-ML。在 compositional sequence 的语义对齐与过渡平滑度上超过 prior arts；inbetweening 任务也有竞争力。
- **亮点/局限**：把"phase manifold"思想（DeepPhase, SIGGRAPH 2022）推广到 diffusion 框架下的 compositional generation，是 phase-based 路线的重要推进。代码开源。局限是依赖预训练 ACT-PAE 的质量，对极端长度差异的动作段拼接仍可能失效。
- **链接**：https://neurips.cc/virtual/2025/poster/116418 | https://github.com/asdryau/TransPhase

### SnapMoGen: Human Motion Generation from Expressive Texts
- **作者/机构**：Chuan Guo, Inwoo Hwang, Jian Wang, Bing Zhou（Snap Research）
- **任务**：expressive text 驱动的 text-to-motion 生成
- **核心方法**：提出 **MoMask++** 架构——在 MoMask (CVPR 2024) 基础上引入 **multi-scale residual motion VQVAE** 和 generative masked Transformer，并新增 **global motion refinement** 模块：用轻量 root motion regressor 从 local motion features 预测 root linear velocity，推理时重预测 root trajectory 以减轻滑步。另构建 **SnapMoGen dataset**（expressive text–motion pairs）。
- **数据集/评测**：HumanML3D + SnapMoGen。在 expressive 文本描述下显著优于 MoMask 等基线。
- **亮点/局限**：root refinement 作为后处理模块即插即用，工程价值高；SnapMoGen 数据集填补了 expressive 文本标注的空白。局限是新数据集规模与标注协议细节披露有限。
- **链接**：https://arxiv.org/abs/2507.09122 | https://github.com/snap-research/SnapMoGen

### SoPo: Text-to-Motion Generation Using Semi-Online Preference Optimization
- **作者/机构**：Xiaofeng Tan, Hongsong Wang, Xin Geng, Pan Zhou
- **任务**：text-to-motion 的 preference alignment（DPO 变体）
- **核心方法**：理论分析 offline DPO（过拟合）与 online DPO（采样偏差）各自的缺陷，提出 **Semi-online Preference Optimization (SoPo)**：构造"半在线"偏好对——unpreferred motion 来自 online distribution，preferred motion 来自 offline 高质量数据集。这样让 online/offline DPO 互相补偿。可直接用于 fine-tune MDM / MLD 等 diffusion backbone。
- **数据集/评测**：HumanML3D。MLD+SoPo 达到 **MM-Dist 3.25%**（对比 MoDiPO 0.76%），MLD+SoPo 在 R-precision 与 MM-Dist 上超越 SOTA 模型；MDM+SoPo 也有一致提升。
- **亮点/局限**：第一篇系统分析 DPO 在 motion 上的理论局限并提出修正的工作，实验设置严谨。局限是只覆盖 single-segment T2M，对 compositional/long-form 场景的 preference 定义仍需探索。
- **链接**：https://paperlist.ai/conferences/neurips/2025/topics/motion-generation | https://xiaofeng-tan.github.io/projects/SoPo/

### HOI-Dyn: Learning Interaction Dynamics for Human-Object Motion Diffusion
- **作者/机构**：Lin Wu, Zhi Chen, Jie Lan（University of Sheffield）
- **任务**：3D human-object interaction (HOI) motion 生成
- **核心方法**：把 HOI 生成形式化为 **driver-responder 系统**——human action 驱动 object response。核心是一个轻量 transformer-based **interaction dynamics model**，显式预测物体如何响应人体运动；另提出 **residual-based dynamics loss** 抑制动力学预测误差的反向误导。关键：dynamics model **只在训练期使用**，推理效率不受影响。
- **数据集/评测**：自建/公开 HOI 数据集。不仅提升 HOI 生成质量，还提出一种可评估生成 interaction 质量的 metric。
- **亮点/局限**：driver-responder 视角比"jointly generate human+object"更贴近物理因果，且推理期零开销。局限是对非刚性/多物体交互的泛化未验证。
- **链接**：https://eprints.whiterose.ac.uk/236948

### MEGADance: Mixture-of-Experts Architecture for Genre-Aware 3D Dance Generation
- **作者/机构**：Yang et al.
- **任务**：music-to-dance（genre-aware）
- **核心方法**：不同舞蹈流派（hip-hop / ballet / jazz 等）在节拍结构、身体部位参与度、风格词汇上差异巨大，单一模型难以兼顾。提出 **mixture-of-experts (MoE) architecture**，每个 expert  specialize 于特定 genre 的运动模式，gating network 根据输入音乐特征动态路由。
- **数据集/评测**：AIST++ 等多流派舞蹈数据。在 genre-conditioned 设定下显著优于单模型基线。
- **亮点/局限**：MoE 天然适配"多分布混合"的舞蹈数据，是可解释的风格解耦方案。局限是 genre 标签的获取成本与模糊边界问题。
- **链接**：https://github.com/XulongT/MEGADance

---

## NeurIPS 2024

### MoGenTS: Motion Generation based on Spatial-Temporal Joint Modeling
- **作者/机构**：Weihao Yuan, Weichao Shen, Yisheng He, Yuan Dong, Xiaodong Gu, Zilong Dong, Liefeng Bo, Qixing Huang（阿里巴巴 / UT Austin）
- **任务**：text-to-motion
- **核心方法**：批判现有多将全身 pose 量化为一个 code 的做法——既难编码所有关节又丢失空间关系。提出 **per-joint quantization**：每个关节独立量化为一个向量，得到一张 **2D joint-token map**（轴为关节 × 时间），从而 i) 简化量化 ii) 保持时空结构 iii) 可套用 2D 图像领域的各种操作。在此之上构建 **2D joint VQVAE + temporal-spatial 2D masking + spatial-temporal 2D attention** 的完整框架。
- **数据集/评测**：HumanML3D 上 **FID 降低 26.6%**，KIT-ML 上 **降低 29.9%**，大幅超越 prior discrete methods（T2M-GPT, MoMask）。
- **亮点/局限**：把"per-joint VQ + 2D token map"引入 motion generation 是简洁而有力的表征创新，直接打通了 2D vision 中的 masking/attention 工具箱。代码已开源。局限是 per-joint codebook 的总参数量较大，推理时需联合解码所有关节 tokens。
- **链接**：https://arxiv.org/abs/2409.17686 | https://aigc3d.github.io/mogents/

### UniMTS: Unified Pre-training for Motion Time Series
- **作者/机构**：Zhang et al.
- **任务**：motion time series 的统一预训练（涵盖生成、预测、分类等下游）
- **核心方法**：把 diverse motion sequences（mocap、IMU、markerless 等）统一到同一 transformer 框架下做大规模自监督预训练，通过掩码重建 + 对比学习学到通用 motion 表征，再 zero-shot / few-shot 迁移到各下游。
- **数据集/评测**：多源 motion time series 集合。在多个下游任务上验证预训练收益。
- **亮点/局限**："motion foundation model"路线的代表作之一，呼应了 Being-M0 / MotionLib 的数据 scaling 趋势。局限是预训练数据异构带来的对齐难题。
- **链接**：https://github.com/xiyuanzh/UniMTS

### MoMu-Diffusion: On Learning Long-Term Motion-Music Synchronization and Correspondence
- **作者/机构**：You et al.
- **任务**：long-term music-to-motion 同步生成
- **核心方法**：针对长程音乐-动作对齐难题，提出在 diffusion 框架内显式建模 motion-music 的跨模态对应关系，通过 contrastive 辅助目标强化节拍与乐句级别的同步。
- **数据集/评测**：AIST++ / FineGym 等。在 long-term coherence 指标上优于 Bailando / EDGE 等基线。
- **亮点/局限**：长程音乐结构（verse-chorus-bridge）的显式建模是舞蹈生成的关键缺口。局限是仅覆盖单人舞蹈。
- **链接**：https://momu-diffusion.github.io/

### M3GPT: An Advanced Multimodal, Multitask Framework for Motion Comprehension and Generation
- **作者/机构**：Luo et al.
- **任务**：motion 理解 + 生成的统一多模态 multitask 框架
- **核心方法**：把 motion / text / audio 统一 token 化后送入同一个 LLM-style transformer，通过 instruction tuning 支持 captioning、retrieval、generation、editing 等多种任务的任意切换。
- **数据集/评测**：HumanML3D / KIT-ML / AIST++ 等多数据集联合训练。在 generation 与 understanding 两端都达到有竞争力的结果。
- **亮点/局限**：与 MotionGPT (AAAI 2024) / AvatarGPT (CVPR 2024) 同属"motion as foreign language"脉络，但扩展了任务覆盖面与数据规模。局限是单一 tokenizer 对不同模态的信息损失不均。
- **链接**：https://arxiv.org/abs/2405.16273

### Constrained Synthesis with Projected Diffusion Models
- **作者/机构**：Christopher et al.
- **任务**：带约束的 motion / image 生成
- **核心方法**：从优化视角重新审视 training-free 的损失引导扩散过程，在推理期设计一系列 constrained optimization steps——允许样本沿 surrogate constraint function 的梯度做多步调整，直到根据每步扩散方差判断"不再信任该代理"为止。另估算 diffusion model 的 state manifold，在样本偏离时提前终止。
- **数据集/评测**：image + 3D motion 双领域验证，生成质量优于 DNO (CVPR 2024) 等 baseline。
- **亮点/局限**：与 ProjFlow (CVPR 2026) 思路相近但更早，强调"trust region"式的自适应约束强度。局限是内循环优化增加推理耗时。
- **链接**：https://openreview.net/forum?id=FsdB3I9Y24

### CigTime: Corrective Instruction Generation Through Inverse Motion Editing
- **作者/机构**：Fang et al.
- **任务**：motion editing → 反向生成 corrective natural language instruction
- **核心方法**：给定一段需要修改的 motion 和编辑后的版本，模型自动生成描述"改了什么、为什么改"的自然语言指令。本质是把 motion editing 的逆问题形式化为 instruction generation，为"human-in-the-loop motion authoring"提供自动反馈回路。
- **数据集/评测**：自建 paired (original, edited, instruction) 数据集。
- **亮点/局限**：逆向视角新颖，对交互式创作工具链有价值。局限是数据集规模与语言多样性有待扩展。
- **链接**：https://btekin.github.io/

#### [D&B] Text to Blind Motion
- **作者/机构**：Kim et al.
- **任务**：为视障人群生成可触觉感知的 motion 序列（Datasets & Benchmarks track）
- **核心方法**：构建首个"text-to-blind-motion"数据集与评测协议，把视觉 motion 转化为触觉友好的时空模式（振幅、频率、接触区域编码语义）。
- **亮点/局限**：社会价值突出，开辟了 accessibility-oriented motion generation 新方向。局限是触觉 rendering 硬件依赖性强，通用 benchmark 尚未形成。
- **链接**：https://neurips.cc/virtual/2024/poster/97700

---

## ICML 2025

### Being-M0: Scaling Motion Generation Models with Million-Level Human Motions
- **作者/机构**：Ye Wang, Sipeng Zheng, Bin Cao, Qianshan Wei, Weishuai Zeng, Qin Jin, Zongqing Lu（BeingBeyond / 复旦大学 / 北大）
- **任务**：large-scale motion foundation model（data + model scaling）
- **核心方法**：发布 **MotionLib**——首个 **million-level** motion 数据集（至少 15× 大于既有 counterpart），配 hierarchical text descriptions。模型侧提出 **Motionbook** encoding：(i) compact yet lossless motion feature；(ii) **2D lookup-free motion tokenizer**——无需 codebook 查找即可保留细粒度细节并扩展容量，显著提升 token 表征力。系统研究 data/model size 的 scaling law，首次确认 motion generation 中存在可复现的 scaling 规律。
- **数据集/评测**：MotionLib 百万级数据上训练，在 wide-range 人类活动（含 unseen 类别）上展现强泛化。
- **亮点/局限**：与 CVPR 2026 的 OpenT2M / ICCV 2025 的 MotionMillion 构成 2025 年"data scaling war"三极；lookup-free tokenizer 是方法论亮点。Spotlight 论文。局限是百万数据的来源构成与清洗 pipeline 细节披露不足。
- **链接**：https://arxiv.org/abs/2410.03311 | https://beingbeyond.github.io/Being-M0/

---

## ICML 2024

### HumanTOMATO: Text-aligned Whole-body Motion Generation
- **作者/机构**：Lu et al.
- **任务**：whole-body（含 hand + face）text-driven motion 生成
- **核心方法**：现有 T2M 大多只覆盖 body joints，忽略手部和面部表情。HumanTOMATO 在 SMPL-X 全身体表征上构建统一的 text-aligned diffusion 框架，通过 cross-modal attention 把文本语义同时注入 body/hand/face 三个子空间，并设计部位间协调正则项避免冲突。
- **数据集/评测**：EMOCA / BEAT 等 whole-body 数据。在 whole-body 保真度与 text 对齐度上显著优于 body-only 基线。
- **亮点/局限**：明确把"whole-body"作为第一公民而非事后附加，顺应了 digital human 产业需求。局限是 hand/face 的标注噪声较 body 更大，对生成质量有上限约束。
- **链接**：https://github.com/LinghaoChan/HumanTOMATO

### GPHLVM: Bringing Motion Taxonomies to Continuous Domains via GPLVM on Hyperbolic Manifolds
- **作者/机构**：Jaquier et al.
- **任务**：motion taxonomy 的连续潜空间建模
- **核心方法**：经典 motion taxonomy（如 Labanotation、action categories）本质是离散层级结构，强行嵌入欧氏空间会扭曲层级距离。提出在 **双曲流形（hyperbolic manifold）** 上用 **Gaussian Process Latent Variable Model (GPLVM)** 学习连续 motion 嵌入，天然保持层级远近关系。
- **数据集/评测**：AMASS / KIT 等。在 taxonomy preservation 指标与 downstream generation quality 上均优于欧氏 VAE/GP 基线。
- **亮点/局限**：把双曲几何引入 motion representation learning 是少见的数学工具跨界，对 hierarchical action modeling 有普适价值。局限是双曲空间优化的数值稳定性要求较高。
- **链接**：https://sites.google.com/view/gphlvm/

---

## ICML 2026

> 注：ICML 2026 投稿截止约在 2026 年 1–2 月，录用通知通常在 5–6 月。截至 2026 年 8 月 proceedings 应已开放，但本次检索未发现明确标注为 ICML 2026 的 motion generation 主线论文（不含 workshop）。如有后续更新请补充。

---

## NeurIPS 2026

> 注：NeurIPS 2026 预计 2026 年 12 月举办，录用通知尚未公布。本节预留。
# AAAI / IJCAI

> 数据来源：AAAI Digital Library (ojs.aaai.org)、IJCAI proceedings、arXiv API、Awesome-Human-Motion 仓库交叉校验。
> 指标说明：同前文（FID / R-Precision / MM-Dist / Diversity / MModality，HumanML3D 或 KIT-ML 协议）。

---

## AAAI 2026

### ReAlign: Bilingual Text-to-Motion Generation via Step-Aware Reward-Guided Alignment
- **作者/机构**：Weng et al.
- **任务**：bilingual（中英）text-to-motion 生成 + reward-guided alignment
- **核心方法**：针对非英语文本驱动 motion 时语义漂移问题，提出 **step-aware reward-guided alignment**——在 diffusion 去噪的每一步施加双语一致性奖励信号，引导 latent 向两种语言共享的语义流形收敛。构建首个 bilingual T2M benchmark（中英平行 prompt–motion 对）。
- **数据集/评测**：自建 bilingual benchmark。在中英文 zero-shot cross-lingual transfer 上显著优于单语基线与 machine-translation pipeline。
- **亮点/局限**：首次把"bilingual alignment"引入 motion generation，对多语种落地有直接价值。局限是 reward 设计依赖外部翻译模型的质量。
- **链接**：https://wengwanjiang.github.io/ReAlign-page/

### FineXtrol: Controllable Motion Generation via Fine-Grained Text
- **作者/机构**：Shen et al.
- **任务**：细粒度可控 text-to-motion
- **核心方法**：现有多数 T2M 只接受粗粒度句子级描述，难以控制具体身体部位/时间区间。FineXtrol 把文本解析为 **fine-grained control tokens**（body-part × time-span × action-verb 三元组），再通过 cross-attention 注入 diffusion backbone，实现部位级、时段级的精确控制。
- **数据集/评测**：HumanML3D 上重新标注细粒度子集。在 fine-grained control accuracy 与整体 FID 上双优。
- **亮点/局限**：直击"coarse prompt → vague motion"的产业痛点，控制粒度与现有 authoring 工具链可对接。局限是对 LLM-based parser 的强依赖。
- **链接**：https://arxiv.org/abs/2511.18927

### InterMoE: Individual-Specific 3D Human Interaction Generation via Dynamic Temporal-Selective MoE
- **作者/机构**：Xinghan Wang, Kun Xu, Fei Li, Cao Sheng, Jiazhong Yu, Yadong Mu
- **任务**：person-specific human-human interaction 生成
- **核心方法**：现有双人交互生成方法要么丢失个体特征（identity blur），要么过度拟合文本而忽略个人运动习惯。InterMoE 基于 **dynamic temporal-selective mixture-of-experts**：routing mechanism 协同使用高层文本语义与低层 motion context，将每个时间步的运动特征分配给最匹配的 expert；expert 动态聚焦关键时间特征，从而保留特定个人的 identity 同时确保高语义保真度。
- **数据集/评测**：InterHuman 上 **FID 降低 9%**，InterX 上 **降低 22%**，在 person-specific high-fidelity 3D interaction 生成上达 SOTA。
- **亮点/局限**：MoE routing 天然适配"不同人用不同运动策略"的直觉，且 temporal-selective 机制比全局 style embedding 更精细。代码已开源。局限是需每人少量 calibration clips 才能发挥最佳效果。
- **链接**：https://arxiv.org/abs/2511.13488 | https://github.com/Lighten001/InterMoE

### Generating Attribute-Aware Human Motions from Textual Prompt
- **作者/机构**：Xinghan Wang, Kun Xu, Fei Li, Cao Sheng, Jiazhong Yu, Yadong Mu
- **任务**：attribute-aware text-to-motion（年龄/性别/体重/身高影响运动模式）
- **核心方法**：指出现有方法完全忽略人体属性对运动模式的影响。受 **Structural Causal Model (SCM)** 启发，把 motion 分解为 **action semantics**（与文本对齐）与 **human attributes**（与属性输入对齐）两个因果因子，通过解耦表征实现"text→semantics + attribute→style"的可控生成。发布首个含 attribute 标注的 text-motion 配对数据集作为 benchmark。
- **数据集/评测**：自建 attribute-annotated benchmark。在 attribute-controlled generation fidelity 与 text alignment 上均显著优于忽略属性的基线。
- **亮点/局限**：首次系统研究"人体属性→运动风格"的因果建模，对个性化 avatar 与数字人有直接应用。局限是属性空间仍较粗糙（仅 age/gender/weight/height），未覆盖柔韧性、肌肉量等连续生理参数。
- **链接**：https://arxiv.org/abs/2506.21912

---

## AAAI 2025

### MotionCraft: Crafting Whole-body Motion with Plug-and-Play Multimodal Controls
- **作者/机构**：Yuxuan Bian, Ailing Zeng, Xuan Ju, Xian Liu, Zhaoyang Zhang, Wei Liu, Qiang Xu
- **任务**：whole-body multimodal motion generation（text / speech / music 任意条件）
- **核心方法**：统一模型处理多条件模态面临两大挑战——(i) 不同任务间 motion distribution drift（如 co-speech gesture vs text-driven daily action），(ii) 混合条件的多粒度优化困难。提出 **MotionCraft**：unified diffusion transformer + **MC-Attn**（parallel modeling of static & dynamic human topology graphs）+ **coarse-to-fine training**（先 text-to-motion 语义预训练，再 multimodal low-level 适配）。另发布 **MC-Bench**——首个基于统一 SMPL-X 格式的 multimodal whole-body motion benchmark，解决既有 benchmark 格式不一致的问题。
- **数据集/评测**：在多种标准 motion generation 任务上达 SOTA；MC-Bench 提供标准化对比平台。
- **亮点/局限**：SMPL-X 统一 benchmark 是社区基础设施级贡献；plug-and-play 的多条件适配器设计务实。局限是 coarse-to-fine 两阶段训练增加工程复杂度。
- **链接**：https://cure-lab.github.io/MotionCraft/ | Proceedings of the AAAI Conference on Artificial Intelligence, Vol. 39 No. 2, pp. 1880–1888

### Light-T2M: A Lightweight and Fast Model for Text-to-Motion Generation
- **作者/机构**：Ling-An Zeng, Guohong Huang, Gaojie Wu, Wei-Shi Zheng（中山大学）
- **任务**：轻量快速 text-to-motion
- **核心方法**：重新审视 human motion 的内在属性，发现 local information modeling 被忽视，提出 **Lightweight Local Information Modeling Module**。首次把 **Mamba (state space model)** 引入 T2M，大幅减少参数量与显存占用；设计 **Pseudo-bidirectional Scan** 在不增加参数的前提下复现 bidirectional scan 效果。另提出 **Adaptive Textual Information Injector** 更有效地把文本注入 motion 解码过程。
- **数据集/评测**：HumanML3D 上 **FID 0.040**（vs MoMask 0.045），KIT-ML 上 **0.161**（vs 0.228）；参数量仅 **4.48M**（MoMask 的 10%），推理速度快 **16%**（0.152s vs 0.180s）。
- **亮点/局限**：在"更小更快更强"三个维度同时超越 MoMask，是 efficiency-oriented T2M 的标杆工作。Mamba 在 motion 上的首次成功应用也具方法论意义。局限是极端长序列下 Mamba 的状态压缩误差累积尚未充分评估。
- **链接**：https://arxiv.org/abs/2412.11193

### UniMuMo: Unified Text, Music, and Motion Generation
- **作者/机构**：Yang Han, Kun Su, Yutong Zhang, Jiaben Chen, Kaizhi Qian, Gaowen Liu, Chuang Gan
- **任务**：text / music / motion 三模态统一生成（任一条件 → 任一生成的 all-to-all 框架）
- **核心方法**：核心难点是缺 time-synchronized 三模态配对数据。方案：(i) 基于 rhythmic patterns 对齐 unpaired music 与 motion 数据，利用现有大规模 music-only 与 motion-only 数据集；(ii) 把 music/motion/text 都转为 token 表示，通过统一 encoder-decoder transformer 桥接；(iii) **music codebook 编码 motion**——把 motion 映射到与 music 相同的特征空间；(iv) **music-motion parallel generation scheme**——把所有任务统一成 single transformer decoder 下的 joint generation 单一训练目标。微调既有预训练单模态模型以降低计算需求。
- **数据集/评测**：在所有单向 generation benchmark（music / motion / text）上达到 competitive results。
- **亮点/局限**：all-to-all 三模态统一是 ambitious 的目标，music codebook 作为中间表示的设计巧妙。局限是三模态联合生成的质量上限仍受制于 rhythm-based 对齐的粗糙性。
- **链接**：https://hanyangclarence.github.io/unimumo_demo/

### RemoGPT: Part-Level Retrieval-Augmented Motion-Language Models
- **作者/机构**：Yu et al.
- **任务**：part-level retrieval-augmented motion-language 理解与生成
- **核心方法**：把人体按部位拆解，每个部位维护独立的 motion codebook 与检索索引；推理时先从文本定位相关部位与动作原型，再检索增强生成。本质是把 retrieval-augmented generation (RAG) 从 NLP 迁移到 part-level motion。
- **数据集/评测**：HumanML3D / KIT-ML。在 open-vocabulary 设定下显著优于 non-retrieval 基线。
- **亮点/局限**：part-level RAG 是少见的"结构化 RAG"尝试，对 compositional generalization 有理论优势。局限是检索索引的构建与维护成本。
- **链接**：https://yu1ut.com/ReMoGPT-HP/

### ALERT-Motion: Autonomous LLM-Enhanced Adversarial Attack for Text-to-Motion
- **作者/机构**：Miao et al.
- **任务**：T2M 模型的 adversarial robustness 评测
- **核心方法**：利用 LLM 自动生成对抗性文本扰动（语义等价但导致 motion 崩坏），构造 autonomous adversarial attack pipeline，系统评估主流 T2M 模型的脆弱性。
- **亮点/局限**：安全视角稀缺，对 motion foundation model 的可靠性评估有基础价值。局限是攻击成功率与防御手段仍需深入。
- **链接**：https://arxiv.org/abs/2408.00352

### EchoMimic: Lifelike Audio-Driven Portrait Animations through Editable Landmark Conditioning
- **作者/机构**：Chen et al.（阿里）
- **任务**：audio-driven portrait animation（含 upper-body motion）
- **核心方法**：通过 editable landmark conditioning 把音频驱动的面部 + 上半身 motion 统一到 diffusion video 框架，支持用户编辑 landmark 轨迹以控制表情幅度与头部姿态。
- **亮点/局限**：面向数字人直播/短视频场景，editable landmark 是实用的交互接口。局限是主要贡献在 rendering 侧，motion 生成本身创新性有限。
- **链接**：https://antgroup.github.io/ai/echomimic/

---

## AAAI 2024

### HuTuDiffusion: Human-Tuned Navigation of Latent Motion Diffusion Models with Minimal Feedback
- **作者/机构**：Han et al.
- **任务**：human-in-the-loop motion diffusion 微调
- **核心方法**：只需极少量 human feedback（pairwise preference 或 scalar score），在 latent diffusion 的 sampling trajectory 上导航修正，无需全量重训。
- **亮点/局限**：minimal-feedback 设定贴近真实创作流程。局限是反馈信号稀疏时的收敛稳定性。
- **链接**：https://arxiv.org/abs/2312.12227

### AMD: Anatomical Motion Diffusion with Interpretable Motion Decomposition and Fusion
- **作者/机构**：Jing et al.
- **任务**：anatomically plausible text-to-motion
- **核心方法**：把 motion 按解剖学结构分解为独立子空间（躯干 / 上肢 / 下肢），各子空间独立 diffusion 后再通过 learnable fusion 重组，保证关节活动范围与 biomechanical constraint。
- **亮点/局限**：anatomy-aware 分解提升物理合理性，对康复/医疗应用有价值。局限是分解粒度固定，难以适应跨物种泛化。
- **链接**：https://arxiv.org/abs/2312.12763

### MotionMix: Weakly-Supervised Diffusion for Controllable Motion Generation
- **作者/机构**：Hoang et al.
- **任务**：弱监督可控 motion 生成
- **核心方法**：仅需少量 labeled data + 大量 unlabeled motion，通过 consistency regularization 与 conditional diffusion 结合实现可控生成。
- **亮点/局限**：weak supervision 设定务实，降低 mocap 标注成本。局限是对 unlabeled 数据分布敏感。
- **链接**：https://nhathoang2002.github.io/MotionMix-page/

### B2A-HDM: Towards Detailed Text-to-Motion Synthesis via Basic-to-Advanced Hierarchical Diffusion Model
- **作者/机构**：Xie et al.
- **任务**：细粒度 text-to-motion
- **核心方法**：**Basic-to-Advanced hierarchical diffusion**——先由 basic diffusion 生成粗粒度骨架运动，再由 advanced diffusion 逐步添加细节（手指、面部微表情）。层级间通过 residual connection 传递误差信号。
- **亮点/局限**：层级细化思路与 MoMask 的 residual VQ 异曲同工，但在 diffusion 框架内实现。局限是两级扩散使推理耗时翻倍。
- **链接**：https://arxiv.org/abs/2312.10960

### Everything2Motion: Synchronizing Diverse Inputs via a Unified Framework for Human Motion Synthesis
- **作者/机构**：Fan et al.
- **任务**：多源输入（text / audio / keyframe / trajectory）统一 motion 合成
- **核心方法**：把 diverse inputs 都投影到同一 condition embedding 空间，通过 unified diffusion backbone 生成 motion。
- **亮点/局限**：any-input-to-motion 的用户体验好。局限是异构条件的信息密度差异大，统一 embedding 易偏向高信噪比条件。
- **链接**：https://ojs.aaai.org/index.php/AAAI/article/view/27936

### MotionGPT: Finetuned LLMs are General-Purpose Motion Generators
- **作者/机构**：Zhang et al.
- **任务**：LLM-based 通用 motion 生成与理解
- **核心方法**：把 motion 量化为 discrete tokens 后作为"foreign language"注入 LLM，通过 instruction tuning 让 LLM 同时具备 motion captioning、retrieval、generation 能力。
- **亮点/局限**："motion as foreign language"范式奠基作之一，启发了后续 MotionGPT-2 / AvatarGPT / MotionChain 等系列工作。局限是 LLM 的 sequence length 限制了 motion 时长。
- **链接**：https://qiqiapink.github.io/MotionGPT/

### UNIMASKM: A Unified Masked Autoencoder with Patchified Skeletons for Motion Synthesis
- **作者/机构**：Mascaro et al.
- **任务**：masked autoencoder 风格的 motion 合成
- **核心方法**：把 skeleton 序列 patchify 后做 MAE-style 掩码重建预训练，再 finetune 用于生成。
- **亮点/局限**：把 MAE 范式迁移到 motion 的早期探索。局限是 patchify 策略对 skeleton 拓扑的利用不充分。
- **链接**：https://evm7.github.io/UNIMASKM-page/

### Enhanced Fine-grained Motion Diffusion for Text-driven Human Motion Synthesis
- **作者/机构**：Dong et al.
- **任务**：细粒度 text-driven motion
- **核心方法**：在 diffusion 采样过程中引入 fine-grained semantic guidance，通过 multi-scale feature fusion 强化局部动作细节。
- **亮点/局限**：fine-grained 导向明确。局限是 guidance 强度需手工调参。
- **链接**：https://arxiv.org/abs/2305.13773

---

## IJCAI 2025 / 2024

> 注：IJCAI 2025 录用通知约在 2025 年 4 月，proceedings 公开进度不一。本次检索未发现明确标注为 IJCAI 2024/2025 的 motion generation 主线论文（不含 workshop）。IJCAI 传统上偏 AI 理论与 agent 方向，motion generation 在该会议占比较少，相关工作多流向 CVPR/ECCV/NeurIPS/AAAI。如有后续更新请补充。

---

## IJCAI 2026

> 注：IJCAI 2026 预计 2026 年夏季举办，录用通知尚未公布。本节预留。
# Eurographics (EG) / SCA

> 数据来源：Eurographics Digital Library (diglib.eg.org)、ACM DL (SCA proceedings)、SCA 官网 (computeranimation.org)、arXiv API。
> 指标说明：EG/SCA 论文多以 user study / perceptual metrics 为主，部分辅以 FID / MPJPE 等技术指标。

---

## Eurographics 2025

### Shape-Conditioned Human Motion Diffusion Model with Mesh Representation
- **作者/机构**：Kebing Xue, Hyewon Seo, Cedric Bobenrieth, Guoliang Luo
- **任务**：body-shape 条件化的 motion diffusion（mesh representation）
- **核心方法**：现有多数 T2M 工作在 joint/rotation 空间，忽略了 body mesh 几何对 motion 可行域的约束。本文把 diffusion 直接建在 **mesh vertex 空间**，并以 body shape parameters（如 SMPL beta）为条件，使生成的 motion 自动适配不同体型。
- **数据集/评测**：AMASS / HumanML3D re-meshed 版本。perceptual study 显示 shape-conditioned 版本在 physical plausibility 上显著优于 joint-space 基线。
- **亮点/局限**：mesh-level diffusion 是少见的表征选择，对 clothed avatar 动画有直接价值。局限是 vertex 空间维度高，diffusion 训练成本较大。
- **链接**：https://doi.org/10.2312/egs.20251048（Full Paper 14, "Bringing Motion to Life: Motion Reconstruction & Control" session）

### Versatile Physics-based Character Control with Hybrid Latent Representation
- **作者/机构**：Jinseok Bae, Jungdam Won, Donggeun Lim, Inwoo Hwang, Young Min Kim（Seoul National University）
- **任务**：physics-based character control（universal skill learning）
- **核心方法**：提出 **hybrid latent representation**——把 motion 分解为 task-relevant low-dim latent（供 RL policy 高效搜索）与 detail-preserving high-dim residual（保真还原），两者联合训练。policy 在低维 latent 上做决策，decoder 负责还原全身细节。
- **数据集/评测**：多任务 benchmark（locomotion / manipulation / contact-rich interaction）。在 success rate 与 motion quality 上双优。
- **亮点/局限**：hybrid latent 兼顾"control efficiency"与"visual fidelity"，是 physics-based control 路线的关键工程推进。SNU 团队同期还有 PLT (SIGGRAPH 2025) 与 SceneMI (ICCV 2025)，构成完整的 motion prior → control → scene interaction 技术栈。
- **链接**：Eurographics 2025 Full Paper 14 | https://3d.snu.ac.kr/publications

### Generative Motion Infilling from Imprecisely Timed Keyframes
- **作者/机构**：Purvi Goel, Haotian Zhang, C. Karen Liu, Kayvon Fatahalian（Stanford）
- **任务**：从**时间不精确**的关键帧生成完整 motion（generative inbetweening）
- **核心方法**：传统 inbetweening 假设关键帧的时间戳精确已知，但实际创作中 animator 往往只给出大致顺序。本文把"时间对齐"本身作为 latent variable 纳入生成模型，通过 variational inference 联合推断 keyframe timing 与 inbetween motion。
- **数据集/评测**：CMU Mocap / 自建 animator keyframe 数据集。user study 显示对 imprecise keyframe 输入的鲁棒性显著优于 fixed-timing 基线。
- **亮点/局限**：直面真实创作流程中的"timing uncertainty"，工具价值突出。局限是 variational timing 的后验多峰性可能导致 mode collapse。
- **链接**：Eurographics 2025 Full Paper 14

### DragPoser: Motion Reconstruction from Variable Sparse Tracking Signals via Latent Space Optimization
- **作者/机构**：Jose Luis Ponton, Eduard Pujol, Andreas Aristidou, Carlos Andujar, Nuria Pelechano
- **任务**：sparse tracking signal（VR controller / IMU 子集）驱动的 full-body motion reconstruction
- **核心方法**：在 pre-trained motion VAE 的 latent space 上做 online optimization——给定可变数量的 sparse observations，迭代优化 latent code 使重建 pose 与观测一致。支持 tracking signal 的动态增减（如 VR 场景中手柄暂时丢失）。
- **亮点/局限**：latent optimization 比 end-to-end regression 更能适应传感器配置变化。局限是迭代优化延迟对实时 VR 应用有挑战。
- **链接**：Eurographics 2025 Full Paper 14

### Lightweight Morphology-Aware Encoding for Motion Learning (Short Paper)
- **作者/机构**：Ziyu Wu, Thomas Michel, Damien Rohmer
- **任务**：跨形态（morphology）motion 预测
- **核心方法**：扩展 standard skeletal embedding，加入 proxy cylinder radius  conveying geometric information about character morphology at each joint，以 compact token 形式送入 transformer。三种 tokenization 策略验证有效性。
- **亮点/局限**：在 limited data 下实现跨体型泛化，对小团队实用。局限是 cylinder proxy 的表达力上限。
- **链接**：https://doi.org/10.2312/egs.20251048 (Short Papers)

---

## Eurographics 2024

### Interactive Locomotion Style Control for a Human Character based on Gait Cycle Features
- **作者/机构**：Chaelin Kim, Jung Eun Yoo, Haegwang Eom, Soo Jin Choi, Junyong Noh（KAIST / Disney Research）
- **任务**：real-time locomotion style control
- **核心方法**：基于 gait analysis 定义一组 **gait cycle features**——把各种 locomotion styles 表达为单个 gait cycle 内的 spatio-temporal pattern。对 mocap 库中每个 gait cycle 计算这些特征，配合 **Motion Matching** 算法实时检索匹配 motion。提供 graphical controller interface 可视化 style representation。
- **数据集/评测**：user study 对比现有 footstep animation tool，本方法在 intuitive control 与 fast visual feedback 维度显著胜出。
- **亮点/局限**：gait-cycle-level 的特征设计比 frame-level 更符合生物力学直觉，控制器界面友好。局限是仅覆盖 locomotion，未涉及 upper-body 协调。
- **链接**：Eurographics 2024 (45th Annual Conference), DOI: 10.1111/cgf.14988

### Learning Climbing Controllers for Physics-Based Characters
- **作者/机构**：Kang et al.
- **任务**：physics-based climbing character controller
- **核心方法**：针对攀岩这一 contact-rich、multi-contact 的特殊 locomotion 模式，设计分层 RL controller——高层规划 handhold/foot hold 序列，低层通过 PD control + reward shaping 跟踪参考 motion。利用 CIMI4D (CVPR 2023) 等多模态 climbing dataset 做 imitation learning。
- **亮点/局限**：climbing 是 crowd simulation 与 parkour 之间的空白地带，本文填补了 physics-based climbing 的方法论缺口。对用户正在进行的 crowd simulation 轨迹优化项目可能有参考价值。
- **链接**：Eurographics 2024 | https://diglib.eg.org/server/api/core/bitstreams/f1072102-82a6-4228-a140-9ccf09f21077/content

---

## SCA 2025

> 注：SCA 2025 (24th ACM SIGGRAPH/Eurographics Symposium on Computer Animation) 于 2025 年 8 月 7–10 日在温哥华举办，proceedings 发表在 PACMCGIT Vol. 8 No. 4。截至检索时官方网站仅开放目录页，具体 paper list 需通过 ACM DL 获取。以下条目来自主流会议（SIGGRAPH 2025 / ICCV 2025 / CVPR 2025）中与 SCA 主题高度重叠的相关工作，供参考：
>
> - **PLT: Part-wise Latent Tokens as Adaptable Motion Priors for Physically Simulated Characters** (SIGGRAPH 2025, Jinseok Bae et al., SNU) — part-wise latent token 作为 physics-based character 的 motion prior，与 EG 2025 的 hybrid latent 一脉相承。
> - **PhysicsFC: Learning User-Controlled Skills for a Physics-Based Football Player Controller** (SIGGRAPH 2025, Kim et al.) — 用户可控的 physics-based 足球运动员技能学习。
> - **SkillMimic-v2: Learning Robust and Generalizable Interaction Skills from Sparse and Noisy Demonstrations** (SIGGRAPH 2025, Yu et al.) — 从稀疏噪声演示中学习鲁棒的 interaction skill。
>
> 待 SCA 2025 proceedings 全面上线后可补充精确条目。

---

## SCA 2024

> SCA 2024 (23rd ACM SIGGRAPH/Eurographics Symposium on Computer Animation) 于 2024 年 8 月在哥本哈根举办，proceedings 发表于 CGF Vol. 43 No. 8。以下是与 motion generation / synthesis 直接相关的条目：

### Diffusion-based Human Motion Style Transfer with Semantic Guidance
- **作者/机构**：Lei Hu, Zihao Zhang, Yongjing Ye, Yiwen Xu, Shihong Xia（中科院自动化所）
- **任务**：motion style transfer（semantic-guided diffusion）
- **核心方法**：在 diffusion 框架下做 style transfer——source motion 的内容 + target style 的语义引导共同约束去噪过程。提出 semantic guidance module 把 style 描述（文本或 reference motion）映射为 diffusion 的条件信号。
- **亮点/局限**：把 diffusion 引入 style transfer 是自然的选择，semantic guidance 比纯 appearance matching 更具解释性。局限是 style/content 解耦质量依赖 guidance 强度调参。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15169

### Pose-to-Motion: Cross-Domain Motion Retargeting with Pose Prior
- **作者/机构**：Qingqing Zhao (Stanford), Peizhuo Li (ETH), Wang Yifan (Stanford), Olga Sorkine-Hornung (ETH), Gordon Wetzstein (Stanford)
- **任务**：cross-domain motion retargeting（从 pose 数据生成 plausible motion）
- **核心方法**：pose 数据比 motion capture 更易获取（静态摆姿甚至可从图片提取）。本文通过 neural retargeting 把 source character 的 motion features 与 target character 的 pose features 融合，仅需目标角色的少量 pose（artist-created 或 image-estimated）即可生成 plausible motion。
- **数据集/评测**：跨 drastically different characters 的 retargeting 实验 + user study。多数参与者认为 retargeted motion 更 enjoyable、更 lifelike、artifact 更少。代码与数据集开源。
- **亮点/局限**：tap into pose data 作为 alternative data source 的思路务实，对 indie game / small studio 有直接价值。局限是 pose-only 监督下 temporal coherence 的上限。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15170

### Long-term Motion In-Betweening via Keyframe Prediction
- **作者/机构**：Seokhyeon Hong, Haemin Kim, Kyungmin Cho, Junyong Noh（KAIST）
- **任务**：long-term motion inbetweening
- **核心方法**：传统 inbetweening 在长间隔关键帧下容易崩坏。本文先预测 intermediate keyframes 把长间隔拆为多个短间隔，再在每个短段上做高质量 inbetweening。
- **亮点/局限**：divide-and-conquer 思路简单有效，对 animator 的实际工作流贴合度高。局限是关键帧预测误差会级联传播。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15171

### LLAniMAtion: LLAMA Driven Gesture Animation
- **作者/机构**：Jonathan Windle, Iain Matthews, Sarah Taylor
- **任务**：LLM-driven co-speech gesture 生成
- **核心方法**：历史上 gesture 生成多是 audio-driven。本文实验用 **Llama2 从文本提取的 features** 替代音频特征驱动 gesture 生成。Surprising finding：**Llama2 features alone 显著优于 audio features**，且两者结合无显著提升。表明 LLM 已编码足够丰富的 beat + semantic gesture 信息，无需音频输入。
- **亮点/局限**：颠覆了"gesture 必须 audio-driven"的传统认知，对 silent avatar / text-only agent 场景有直接价值。局限是 Llama2 对非英语文本的 gesture 编码能力未验证。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15167

### Reactive Gaze during Locomotion in Natural Environments
- **作者/机构**：Julia K. Melgaré, Damien Rohmer, Soraia R. Musse, Marie-Paule Cani
- **任务**：natural environment 中 locomotion 时的 reactive gaze 生成
- **核心方法**：把 gaze behavior 建模为 locomotion context 的反应式函数——根据地形复杂度、障碍物距离、行进速度动态调整注视点分布。
- **亮点/局限**：gaze + locomotion 的耦合是 crowd animation 中被忽视的细节，对虚拟行人 realism 有加成。局限是仅覆盖 gaze，未涉及 upper-body 协调。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15168

### ADAPT: AI-Driven Artefact Purging Technique for IMU Based Motion Capture
- **作者/机构**：Paul Schreiner, Rasmus Netterstrøm, Hang Yin, Sune Darkner, Kenny Erleben
- **任务**：IMU-based mocap 的 artifact 去除
- **核心方法**：用 learning-based 方法检测并修复 IMU 信号中的 transient artifacts（磁干扰、碰撞冲击等），提升 downstream motion reconstruction 质量。
- **亮点/局限**：mocap 数据清洗是产业刚需，工具价值高。局限是对 unseen sensor 型号的泛化。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15172

### Learning to Move Like Professional Counter-Strike Players
- **作者/机构**：David Durst, Feng Xie, Pat Hanrahan, Kayvon Fatahalian, Vishnu Sarukkai, Brennan Shacklett, Iuri Frosio, Chen Tessler, Joohwan Kim, Carly Taylor, Gilbert Bernstein, Sanjiban Choudhury（Stanford / Epic Games）
- **任务**：FPS 游戏角色的 skillful locomotion 模仿学习
- **核心方法**：从专业 Counter-Strike 玩家的 gameplay 录像中提取 locomotion traces，通过 imitation learning 训练 physics-based character controller，复现 crouch-walk、counter-strafing、peek 等高级移动技巧。
- **亮点/局限**：gameplay-driven locomotion learning 是 game AI 的新方向，对 NPC behavior 有直接应用。局限是仅覆盖 bipedal locomotion，未涉及 upper-body weapon handling。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15173

### PartwiseMPC: Interactive Control of Contact-Guided Motions
- **作者/机构**：Niloofar Khoshsiyar, Ruiyu Gou, Tianhong Zhou, Sheldon Andrews, Michiel van de Panne
- **任务**：contact-guided motion 的 interactive MPC control
- **核心方法**：把全身 motion 按 body part 分解，每部分独立做 model predictive control (MPC)，再通过 contact constraint 协调。支持用户实时干预（drag / push character）并即时重规划。
- **亮点/局限**：partwise MPC 比 whole-body MPC 更高效，适合 interactive authoring。局限是 contact-rich 场景下 part 间冲突需额外仲裁。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15174

### VMP: Versatile Motion Priors for Robustly Tracking Motion on Physical Characters
- **作者/机构**：Agon Serifi, Ruben Grandia, Espen Knoop, Markus Gross, Moritz Bächer
- **任务**：physics-based character 上的 robust motion tracking
- **核心方法**：学习 versatile motion priors——一个统一的 latent space 覆盖 locomotion / manipulation / acrobatics 等多种 skill，tracking 时在 latent 中做 nearest-neighbor search 找到最接近的 reference，再用 PD control 跟踪。
- **亮点/局限**：unified motion prior 比 per-skill controller 更易扩展。局限是 latent space 的 coverage 依赖训练数据多样性。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15175

### SketchAnim: Real-time Sketch Animation Transfer from Videos
- **作者/机构**：Gaurav Rai, Shreyas Gupta, ...
- **任务**：sketch → animation 的实时 transfer
- **核心方法**：从视频中提取 sketch-style 时序特征，实时迁移到目标角色。
- **亮点/局限**：sketch-based authoring 门槛低，适合 early-stage prototyping。
- **链接**：SCA '24, CGF 43-8, DOI: 10.1111/cgf.15176

---

## 趋势小结（EG / SCA 线）

| 年度 | 主导范式 | 典型贡献 |
|------|---------|---------|
| 2024 | diffusion + physics-based control | diffusion style transfer, IMU artifact removal, CS player imitation, partwise MPC |
| 2025 | hybrid latent + mesh-level diffusion + gait-aware control | shape-conditioned mesh diffusion, hybrid latent for versatile control, generative infilling from imprecise keyframes, gait-cycle style control |

EG/SCA 线的特色是 **strong application orientation**（VR/AR、game animation、digital human production）与 **perceptual evaluation rigor**（user study 几乎是标配），与 NeurIPS/ICML 的理论深度形成互补。

---

# 跨会议趋势分析

## 一、按年份的技术范式迁移

| 年份 | 主导范式 | 代表性工作群 |
|------|---------|-------------|
| **2024** | diffusion 全面接管 + discrete token 崛起 | MDM / MoFusion / MotionDiffuse (TPAMI) → MoMask (CVPR, masked AR) / T2M-GPT (discrete VQ) / BAMM (bidirectional AR)；physics-based control 开始用 diffusion（CAMDM, A-MDM） |
| **2025** | flow matching / rectified flow + LLM 统一建模 + preference alignment | MLD / DNO 之后 flow matching 成为新默认（MotionHiFlow, ProjFlow, FloodDiffusion）；MotionGPT / AvatarGPT / LaMoGen 等把 motion 当"第二语言"注入 LLM；SoPo / MotionRL / AToM 引入 RLHF/DPO 式偏好对齐 |
| **2026** | causal/streaming + compositional + data scaling + multimodal unification | CMDM / FloodDiffusion / LiveGesture / MIBURI 聚焦流式因果生成；TransPhase / DeMoGen / FrankenMotion 做组合/分解；Being-M0 / OpenT2M / RoMo / MotionMaster 推动百万级数据 scaling；HMVLM / UniMuMo / MotionCraft 做多模态统一 |

## 二、按子方向的热点分布

### 1. Text-to-Motion（核心主干，最拥挤）
- **2024**：masked modeling（MoMask, MMM）、hierarchical diffusion（B2A-HDM）、efficient diffusion（EMDM, MotionLCM）
- **2025**：AR refinement（MoMask++, SnapMoGen）、preference alignment（SoPo, MotionRL, AtoM）、scaling（Being-M0, ScaMo）
- **2026**：causal streaming（CMDM, PRISM）、next-scale AR（MoScale）、flow matching（MotionHiFlow, UMF）、open-vocabulary（Open the Motion Door, Re2MoGen）、symbolic inference（LaMoGen）、bilingual（ReAlign）

### 2. Co-speech Gesture / Dyadic Interaction
- **2024**：audio-driven diffusion（EMAGE, TalkSHOW）、LLM features 初探（Llanimation, SCA 2024）
- **2025**：streaming zero look-ahead（LiveGesture, MIBURI）、caption-augmented（CoordSpeaker）、dyadic mutual dynamics（DyaDiT）
- **2026**：full-body + face 同步（MIBURI）、region-eXpert AR（LiveGesture）、speech-text dual condition（CoordSpeaker）

### 3. Dance / Music-to-Motion
- **2024**：Bailando++、EDGE、MoMu-Diffusion（NeurIPS 2024）
- **2025**：MEGADance（MoE genre-aware, NeurIPS 2025）、FlowerDance（MeanFlow）、MatchDance（Mamba-Transformer hybrid）
- **2026**：OpenDance（multi-modal controllable, CVPR 2026）、atomic movement decomposition（Music-to-Dance via Atomic Movements）

### 4. Human-Object / Scene / Human Interaction
- **2024**：HOI diffusion 爆发（InterDiff, HGHOI, CG-HOI, NIFTY）；scene-aware（Afford-Motion, GenZI, TRUMANS, UniHSI）
- **2025**：physics-based HOI（InterMimic, SkillMimic-v2）、whole-body HOI benchmark（CORE4D, HUMOTO）
- **2026**：unified generation+editing（OneHOI）、visual-prior HOI（ViHOI）、stability-driven co-manipulation、TeamHOI（any team size）、InterPrior（scale generative control）

### 5. Physics-based Character Control / Humanoid
- **2024**：SuperPADL、InsActor、PULSE、H-GAP、HumanVLA（NeurIPS 2024）
- **2025**：CLoSD（ICLR 2025, simulation-diffusion loop）、InterMimic（CVPR 2025）、ASAP（RSS 2025）
- **2026**：Iterative Closed-Loop Synthesis for Scaling Humanoid Control、Beyond Mimicry（HHoI from HHI）、InterPrior（CVPR 2026）

### 6. Motion Editing
- **2024**：DNO（noise optimization）、MotionCLR（attention mechanism understanding）、CoMo（pose code editing）
- **2025**：SimMotionEdit（similarity prediction）、MotionFix（text-driven）
- **2026**：OmniME（positive-negative learning）、Cross-Axis Feature Fusion（joint-wise difference prediction）、Unified Conditional Flow（generation+editing+retargeting）

### 7. Dataset / Benchmark
- **2024**：Text to Blind Motion（NeurIPS D&B）、CIMI4D follow-up
- **2025**：FineMotion（ICCV, spatial+temporal annotation）、MC-Bench（AAAI, unified SMPL-X benchmark）、SpeakerVid-5M（dyadic audio-visual）
- **2026**：OpenT2M（2800h open-source）、RoMo（semantic taxonomy + per-category eval）、HandX（bimanual scaling）、HUMAPS-4D（physiological signals）、MPL benchmark（pressure mat mocap）

## 三、表征方式的演进路线

```
SMPL joint angles (2022–2023)
    ↓
Rot6D continuous (2023–2024, MDM/MoFusion)
    ↓
VQ-VAE discrete tokens (2023–2025, T2M-GPT / MoMask / MoMask++)
    ↓
Per-joint VQ + 2D token map (2024–, MoGenTS NeurIPS 2024)
    ↓
Body-part-specific tokenization (2025–, HMVLM NeurIPS 2025)
    ↓
Lookup-free 2D tokenizer (2025, Being-M0 ICML 2025)
    ↓
Mesh-level diffusion (2025, EG 2025 Xue et al.)
    ↓
Riemannian manifold latent (2026, NRMF arXiv)
```

## 四、生成范式的竞争格局

| 范式 | 代表 | 优势 | 劣势 |
|------|------|------|------|
| Full-sequence diffusion | MDM / MLD | 质量高、训练稳定 | 推理慢、不支持 streaming |
| Latent diffusion | MLD / SALAD | 采样快 | latent 信息损失 |
| Masked AR | MoMask / MoMask++ | 质量好、支持编辑 | 迭代去噪步数多 |
| Next-token AR | T2M-GPT / ScaMo | 天然 streaming | 误差累积、长程弱 |
| Next-scale AR | MoScale (CVPR 2026) | 长程结构好 | 与 residual VQ 耦合复杂 |
| Causal diffusion forcing | CMDM / FloodDiffusion (CVPR 2026) | streaming + 高质量 | 训练调度设计敏感 |
| Flow matching / Rectified flow | MotionHiFlow / ProjFlow / UMF (CVPR 2026) | 采样效率高、ODE 可逆 | 需重新训练 backbone |
| LLM-based | MotionGPT / LaMoGen / HSI-GPT2 | 语义推理强、zero-shot | 序列长度受限、token 保真度 |
| Energy-based | EnergyMoGen / DeMoGen | 组合/分解灵活 | 训练不稳定 |
| RL / Preference alignment | SoPo / MotionRL / AToM / GRPO | 直接优化感知指标 | reward hacking 风险 |

## 五、DPO / RLHF / GRPO 在 motion 上的渗透

2025–2026 年出现明显的"alignment turn"：
- **SoPo**（NeurIPS 2025）：理论分析 offline/online DPO 的缺陷，提出 semi-online 修正
- **MotionRL**（arXiv 2024）：multi-reward RL 对齐 human preference
- **AToM**（CVPR 2025）：GPT-4V reward 做 event-level alignment
- **MotionRFT**（arXiv 2026）：unified reinforcement fine-tuning
- **MoTiGA**（CVPR 2026）：MHPO multi-level hybrid-weighted preference optimization + HumanML3D-R 偏好数据集
- **HSI-GPT2**（CVPR 2026）：GRPO + Chain-of-Thought 用于 long-horizon HSI
- **Motion-R1**（arXiv 2025）：CoT reasoning + RL

这条线与用户的实习方向（SOMA/ViMoGen + DPO/GRPO 微调）高度相关——可以明确看到该方向在顶会已形成气候，但**方法差异化**仍是关键审稿关注点。

## 六、值得关注的"负结果"与方法论贡献

- **FloodDiffusion**（CVPR 2026）：实证发现 vanilla diffusion forcing 从 video 搬到 motion 会失败，给出三条必要修正——这种"negative result + ablation-driven"的方法论在本领域稀缺且珍贵
- **Back to Basics**（arXiv 2025）：系统评估 motion representation 对 diffusion 性能的影响，指出多数"改进"其实来自表征选择而非模型创新
- **The Quest for Generalizable Motion Generation**（arXiv 2025）：data/model/evaluation 三位一体的 critical review

## 七、对用户研究方向的启示

1. **DPO/GRPO 对齐**：SoPo 已证明 semi-online 优于纯 online/offline；你的 SOMA/ViMoGen 微调若要做 alignment，建议优先考虑 semi-online 设定并给出理论分析（参考 SoPo 的 proof technique）。
2. **Data scaling**：Being-M0 / OpenT2M / MotionMillion 表明百万级数据是 2025+ 的入场券；混元 244h 私有数据虽不能公开，但可作为"private high-quality seed"配合 web-scraped data 使用。
3. **评测升级**：RoMo 的 per-category taxonomy + Motion Toolbox 是未来评测的标准配置；单纯刷 HumanML3D FID 已不够有说服力。
4. **Streaming / causal**：CMDM / FloodDiffusion / LiveGesture 构成 2026 年的 streaming 主线；若你的应用场景涉及实时 avatar / dialogue agent，这一路必须跟进。
5. **Physics-aware**：InterPrior / PAMotion / InterPhys 显示 physics-informed 生成正在从"后处理修复"升级为"内生约束"——这对 crowd simulation 轨迹优化的物理合理性也有借鉴意义。

---

*本文档由 WorkBuddy 基于官方 proceedings、arXiv API 与 Awesome-Human-Motion 社区仓库自动汇总生成，最后更新于 2026-08-11。如有遗漏或勘误欢迎补充。*
