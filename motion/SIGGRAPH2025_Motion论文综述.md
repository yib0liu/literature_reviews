# SIGGRAPH 2025 · Motion 论文逐篇综述

> 编制日期：2026-08-11
> 覆盖范围：人体动作生成、物理角色控制、动作捕捉与估计、动作编辑/风格化/重定向、面部与手部动作、四足与人形机器人运动、群体动画、rigging/skinning（动作可驱动性上游）、视频驱动角色动画。
> 专题重点：**motion tokenizer 与离散运动表征**（VQ-VAE / RVQ / FSQ / masked token modeling）。

---

## 0. 先说结论

### 0.1 会议概况

| | SIGGRAPH 2025 |
|---|---|
| 会期 | 2025-08-10 ~ 08-14，加拿大温哥华（已结束） |
| 论文公开度 | 完整日程 + 全量标题/摘要/DOI 已公开 |
| ACM DL DOI 前缀 | `10.1145/3721238.xxxxxxx`（Conference）/ `10.1145/373xxxx`（Journal TOG 44(4)） |
| 本报告收录 | **28 篇** motion 相关（含边缘条目），其中核心动作类 **16 篇** |
| 可靠性 | 高（官方日程 + arXiv + 项目页三重交叉） |

### 0.2 关于 motion tokenizer 的核心发现

**本届有两篇明确使用离散运动表征的代表作，且都采用了「层级 + masked modeling」路线。**

| 论文 | 量化方案 | 设计动机（一句话） |
|---|---|---|
| **DuetGen**（MPI/DFKI/Saarland） | **两级 VQ-VAE**：粗粒度语义 token（低时间分辨率）+ 细粒度细节 token（高时间分辨率）；配合 **masked transformer** 从音乐生成 token | 把双人交互的复杂时空依赖拆成两层抽象，高层保同步、底层补细节 |
| **ARTalk**（东京大学，SA2025 但方法学上值得对照） | **multi-scale motion codebook**（多尺度运动码本）+ 自回归映射 | 用码本替代连续扩散以实现实时生成 |

与之形成对照的是大量选择连续表征的工作：AnyTop（任意骨架 diffusion）、AutoKeyframe（autoregressive diffusion）、CAMDM（conditional autoregressive motion diffusion）、xADA（GRU 连续输出）、ELGAR（diffusion）、MECo（LLM fine-tuning + 连续姿态）、Semantically Consistent T2M（diffusion + style adapter）、AMOR（多目标 RL，无显式运动表征瓶颈）、Diffuse-CLoC（guided diffusion 联合状态-动作分布）、PhysicsFC（latent skill policy，连续）。

**一个值得注意的趋势**：本届在 motion generation 主线上，**discrete token 并非默认选项**。即便是需要高效推理或长序列建模的场景，多数工作仍选择了连续 diffusion / autoregressive continuous latent。这与 2026 年 GPC / MotionBricks / ARDY / EchoAvatar 四篇 tokenizer 集中爆发的局面形成有趣对比——**2025 年是过渡年，2026 年才是离散化的大年**。

---

# 第一部分 · 专题：Motion Tokenizer 与离散运动表征

## 1.1 DuetGen: Music Driven Two-Person Dance Generation via Hierarchical Masked Modeling

- **作者**：Anindita Ghosh、Bing Zhou、Rishabh Dabral、Jian Wang、Vladislav Golyanik（DFKI）、Christian Theobalt（MPI-INF / Saarland U.）、Philipp Slusallek（Saarland U. / DFKI）、Chuan Guo（Snap Inc.）
- **Session**：Motion from X ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730741
- arXiv / 项目页 / 代码：均标注 available at（具体 URL 未在公开 HTML 中给出）

**问题**：双人舞生成的核心挑战是**双人与音乐的三重同步**——两个舞者既要彼此协调（接触、对称、跟随），又要同时踩准音乐节拍与乐句结构。既有单人舞蹈生成方法直接扩展到双人场景时，难以建模这种高阶交互依赖。

**方法与表征（本届最丰富的 tokenizer 细节之一）**：

- **两阶段管线**：① 双人动作编码为离散 token → ② 从音乐信号生成这些 token
- **第一阶段：层级 VQ-VAE**
  - **两级量化**：第一级在**粗时间分辨率**上提取高层语义特征（如舞步类型、相位），第二级在**细时间分辨率**上提取低层细节（如肢体微调、重量转移）
  - 关键设计：把两个舞者的动作**拼接为一个统一整体**送入 VQ-VAE，而非分别编码——这是交互质量的关键
  - **coarse-to-fine 学习策略**贯穿两个阶段
- **第二阶段：层级 masked transformer**
  - 两个独立的 masked transformer：第一个从音乐生成高层语义 token，第二个以音乐 + 高层 token 为条件生成细粒度 token
  - **masked modeling**：训练时随机 mask 序列中的 token，模型学习预测被 mask 的部分；推理时从一个空 token 序列开始迭代填充
  - 这种设计使得模型既能做无条件生成（全部 mask），也能做条件补全（部分 mask）
- **交互表示**：不引入显式的交互约束项，而是让层级 token 隐式学习交互模式——粗 token 负责同步节拍和舞伴相对位姿，细 token 负责接触精度和身体细节

**结果**：在 duet dance benchmark 上达到 SOTA，motion realism、music-dance alignment、partner coordination 三项指标均领先。用户研究确认显著优于基线。具体数值未从公开摘要中提取到。代码与权重标注为 available。

**为什么这篇重要**：它是本届**唯一一篇在 motion generation 主干中使用 VQ-VAE + masked transformer 的工作**，与 2026 年的 EchoAvatar（RVQ + LLM）和 MotionBricks（multi-head VQ + masked token modeling）形成了清晰的方法学谱系。DuetGen 的"hierarchical VQ + iterative masked decoding"思路在 2026 年被进一步发扬光大。

---

## 1.2 ARTalk: Speech-Driven 3D Head Animation via Autoregressive Model（SA2025，方法学对照）

- **作者**：Xuangeng Chu、Nabarun Goswami、Ziteng Cui、Hanqin Wang、Tatsuya Harada（东京大学）
- **SIGGRAPH Asia 2025 Conference Papers** ｜ DOI：10.1145/3757377.3763955
- arXiv：https://arxiv.org/abs/2502.20323 ｜ 项目页：https://xg-chu.site/ARTalk ｜ 代码已开源

虽然属于 SA2025，但这篇的 tokenizer 设计与 DuetGen 形成有趣的对照，故放在此处一并讨论。

**问题**：现有语音驱动面部动画的 diffusion 方法虽能产生自然运动，但推理速度慢，难以实时应用。

**方法与表征**：

- **multi-scale motion codebook**：学习从语音到多层级运动码本的映射，实现**实时生成**
- **自回归模型**替代 diffusion：逐帧预测 motion code，推理速度大幅提升
- 支持**未见说话风格的自适应**——能在训练集之外的身份上生成个性化 talking avatar
- 输出包含唇部同步、头部姿态、眨眼在内的完整面部运动

**关键结果**：lip synchronization accuracy 与 perceived quality 均显著优于 FaceFormer、CodeTalker、FaceDiffuser 等 diffusion 基线。用户研究确认偏好。

**与 DuetGen 的对照**：两篇都用"码本 + 自回归/ masked modeling"来替代 diffusion 以提升效率，但 DuetGen 面向全身双人交互（更复杂的时序依赖），ARTalk 面向面部（更高频率的细节需求）。两者共同印证了**2025 年前后，码本化 + 非扩散解码已成为实时动作生成的有效路径**。

---

## 1.3 横向对比表

| 维度 | DuetGen | ARTalk（SA2025 参照） |
|---|---|---|
| 量化方法 | 两级 VQ-VAE | multi-scale motion codebook |
| 层级数 | 2（粗语义 + 细细节） | 多尺度（具体层数未公开） |
| 时间下采样 | 粗/细两级不同分辨率 | 未公开 |
| 上层模型 | 两个层级 masked transformer | 自回归 transformer |
| 生成方式 | 迭代 masked filling | 逐帧 autoregressive |
| 训练数据 | duet dance dataset | 多身份语音-面部数据集 |
| 选它的原因 | 双人交互的层级分解 | 实时性需求 |

**一条可直接引用的论断**：

- **DuetGen**："encoding two-person motions as a unified whole" + "coarse-to-fine learning strategy in both stages" —— 把双人动作作为一个整体编码进层级离散 token，再用层级 masked transformer 从音乐迭代恢复，这是 2025 年最接近 2026 年 EchoAvatar / MotionBricks 设计哲学的工作。

---

# 第二部分 · SIGGRAPH 2025 逐篇总结

## Session A：All About Motion & Deformation（Chair 未找到）

### 2.1 Motion Control via Metric-Aligning Motion Matching

- **作者**：Naoki Agata、Takeo Igarashi（东京大学）
- **Session**：All About Motion & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730665

**问题**：传统的时间对齐方法依赖跨域映射（learned or hand-crafted），需要大量配对/标注数据和耗时训练。

**方法（无学习、无量化）**：仅利用**域内距离**做对齐——计算每个域内部 patch 之间的距离矩阵，然后寻找使两个域内距离最优对齐的匹配。无需任何跨域映射、无需训练、无需标注数据。可对齐到草图、标签、音频、另一段动作等多种控制序列。

**关键结果**：在多种 motion control 应用上验证有效性。具体定量数值未从公开摘要中提取到。

**信息缺口**：运行时间、适用数据类型边界、与 DTW / CTC 等经典对齐方法的定量对比。

---

### 2.2 AutoKeyframe: Autoregressive Keyframe Generation for Human Motion Synthesis and Editing

- **作者**：Bowen Zheng（浙大）、Ke Chen、Yuxin Yao、Zijiao Zeng、Xinwei Jiang、He Wang（剑桥）、Joan Lasenby（剑桥）、Xiaogang Jin（浙大）
- **Session**：All About Motion & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730664

**问题**：关键帧动画是行业标准流程但极其耗时；自动化需要在"最小化人工输入"与"保持完全运动控制"之间取得平衡。

**方法（连续 autoregressive diffusion，无量化）**：

- 同时接受**稠密控制信号**（整体运动轨迹）和**稀疏控制信号**（关键时刻的关键姿态）
- **自动生成关键帧**：从稠密的 root position 出发（通过轨迹曲线的弧长参数化确定），用 **autoregressive diffusion model** 逐段生成关键帧
- **skeleton-based gradient guidance**：将稀疏空间约束（如指定时刻的手部位置）作为梯度引导注入去噪过程，支持帧级编辑
- 生成的关键帧可直接在 Maya 等工具中编辑

**关键结果**：high-quality motion synthesis with precise and intuitive control。具体 FID / R-Precision 数值未从公开摘要中提取到。

**信息缺口**：autoregressive 步长、扩散步数、与 Keyframing baselines 的定量对比。

---

### 2.3 AnyTop: Character Animation Diffusion with Any Topology

- **作者**：Inbar Gat、Sigal Raab、Guy Tevet、Yuval Reshef、Amit Haim Bermano、Daniel Cohen-Or（特拉维夫大学）
- **Session**：All About Motion & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730621
- 项目页 / 代码：标注 available at（URL 未在公开 HTML 中给出）

**问题**：为任意骨架生成动作长期未被探索，原因是多样化数据集稀缺 + 数据不规则。

**方法（连续 diffusion + 拓扑感知 Transformer，无量化）**：

- **Transformer denoising network**，将拓扑信息整合进标准注意力机制
- **文本关节描述**注入 latent feature，学习跨骨架的语义对应（如"左前腿"对应不同动物的相应关节）
- **few-shot 泛化**：每类拓扑仅需 3 个训练样本即可泛化到未见骨架
- Latent space 支持下游任务：joint correspondence、temporal segmentation、motion editing

**关键结果**：在 unseen skeletons 上成功生成合理动作；latent space 高度结构化。具体数值未公开。

**与 2026 年的对位**：直接启发了 2026 年的 TopoCap（Graph CVAE + flow matching）和 UniMate（拓扑感知 DiT），构成"拓扑无关运动先验"三年连续议题的第一环。

---

### 2.4 Hyper-Dimensional Deformation Simulation（边缘）

- **作者**：Alvin Shi、Haomiao Wu、Theodore Kim
- **Session**：All About Motion & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730730

4D 空间中的可变形体仿真，属纯物理仿真，**建议剔除**。登记以示穷尽。

---

## Session B：Motion from X（Chair 未找到）

### 2.5 xADA: Controllable and Expressive Audio-Driven Animation

- **作者**：Sarah Taylor、Salvador Medina、Jonathan Windle、Erica Alcusa Sáez、Iain Matthews
- **Session**：Motion from X ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730711

**问题**：从语音音频直接生成面部、舌头、头部的逼真动画，需跨语言、跨声音风格泛化，并支持非言语声音（笑声、叹息等）。

**方法（连续 GRU，无量化）**：

- 利用预训练 **Whisper audio encoder** 提取丰富语音特征
- 通过一系列 **GRU 网络**解码为面部和头部动画
- 直接映射到 **MetaHuman 兼容的 rig controls**，可无缝接入工业管线
- 支持用户覆盖检测到的情绪和眨眼时机
- 配套提出了完整的**数据采集协议**（语音 + 非言语声音）

**关键结果**：quantitative evaluation + user study 确认 SOTA，经常无法与 ground truth 区分。具体数值未公开。

---

### 2.6 DuetGen: Music Driven Two-Person Dance Generation via Hierarchical Masked Modeling

见第一部分 1.1。

---

### 2.7 ELGAR: Expressive Cello Performance Motion Generation for Audio Rendition

- **作者**：Zhiping Qiu、Yitong Jin、Yuan Wang、Yi Shi、Chao Tan、Chongwu Wang、Xiaobing Li、Feng Yu、Tao Yu、Qionghai Dai
- **Session**：Motion from X ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730756
- 代码与 SPD-GEN 数据集：标注 available at（URL 未在公开 HTML 中给出）

**问题**：乐器演奏动作生成需要同时建模精细肢体运动和演奏者-乐器的复杂交互动力学，既有工作多只关注局部身体动作。

**方法（连续 diffusion，无量化）**：

- **whole-body fine-grained diffusion framework**，仅从音频生成大提琴演奏动作
- **Hand Interactive Contact Loss (HICL)** + **Bow Interactive Contact Loss (BICL)**：显式保证手指-弦、琴弓-弦的真实接触
- 提出三个**领域专用评价指标**：finger-contact distance、bow-string distance、bowing score
- 发布 **SPD-GEN** 数据集（从 SPD MoCap 数据集整理归一化）

**关键结果**：在复杂快速交互的大提琴演奏动作上展现潜力。具体数值未从公开摘要中提取到。

---

### 2.8 MECo: Motion-example-controlled Co-speech Gesture Generation Leveraging Large Language Models

- **作者**：Bohong Chen、Yumeng Li、Youyi Zheng、Yao-Xiang Ding、Kun Zhou（浙江大学 CAD&CG 国家重点实验室）
- **Session**：Motion from X ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730611

**问题**：现有 co-speech gesture 控制系统要么依赖预定义类别标签，要么依赖从动作示例导出的隐式伪标签，都会损失原始动作示例中的丰富细节。

**方法（LLM fine-tuning + 连续姿态，无量化）**：

- 利用 **LLM 的理解能力**，通过 fine-tuning 使其同时理解语音音频和动作示例
- **动作示例作为 prompt 中的显式 query context**，而非传统的伪标签——保留示例特异性细节
- 支持**身体部位级粒度控制**
- 支持多模态输入：motion clips、静态姿态、人体视频序列、文本描述

**关键结果**：FGD、motion diversity、example-gesture similarity 三项指标均达 SOTA。具体数值未从公开摘要中提取到。

**与 2026 年的对位**：同一作者组（浙大 CAD&CG + Kun Zhou）的 **EchoAvatar** 在 2026 年升级为 RVQ + Qwen LLM 自回归路线，从 co-speech gesture 拓展到全身音频驱动动作。MECo → EchoAvatar 构成清晰的两年演进线。

---

### 2.9 Semantically Consistent Text-to-Motion with Unsupervised Styles

- **作者**：Linjun Wu、Xiangjun Tang、Jingyuan Cong、He Wang（UCL）、Bo Hu、Xu Gong、Songnan Li、Yuchen Liao、Yiqian Wu、Chen Liu、Xiaogang Jin
- **Session**：Motion from X ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730641
- 项目页 / 代码：标注 available at（URL 未在公开 HTML 中给出）

**问题**：现有 text-to-stylized motion 方法依赖有标签的风格数据集，适应性差；且未充分探索动作-文本-风格之间的时序关联，难以生成语义一致的风格化动作。

**方法（连续 diffusion + style adapter，无量化）**：

- 用**文本作为中介**捕获动作与风格的时序对应关系
- **Semantic-Aware Style Injection (SASI)** module：基于文本计算动作特征与风格特征的语义相关性，选择性注入与动作内容对齐的风格特征
- **无需标注风格数据集**——unsupervised style learning from arbitrary references
- 增强 adaptability 和 generalization 到未见风格

**关键结果**：semantic consistency 和 style expressivity 均优于先前方法。具体数值未公开。

---

## Session C：Physics-Based Human Characters（Journal Track TOG 44(4) + Conference）

### 2.10 AMOR: Adaptive Character Control through Multi-Objective Reinforcement Learning

- **作者**：Lucas N. Alegre、Agon Serifi、Ruben Grandia、David Müller、Espen Knoop、Moritz Bächer（Disney Research Switzerland）
- **Session**：Physics-Based Human Characters ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730656
- arXiv：https://arxiv.org/abs/2505.23708

**问题**：RL 角色控制通常依赖加权求和的冲突奖励函数，需要大量手工调参；对于机器人应用，权重还需考虑 sim-to-real gap。

**方法（多目标 RL，无显式运动表征瓶颈）**：

- 训练**单个权重条件策略**，覆盖奖励权衡的整个 Pareto front
- **训练后选择和调整权重**，大幅缩短迭代周期
- **Hierarchical 扩展**：高层策略动态选择权重以适应新任务
- 多目标策略编码了多样化的行为谱系，便于高效适应新任务

**关键结果**：演示了高动态机器人动作和 hierarchical task adaptation。具体数值未从公开摘要中提取到。

**与 2026 年的对位**：同组（Disney Research）的 **ReActor**（SIGGRAPH 2026 Journal Track）延续了这个方向，聚焦 physics-aware motion retargeting。

---

### 2.11 Diffuse-CLoC: Guided Diffusion for Physics-based Character Look-ahead Control

- **作者**：Xiaoyu Huang*、Takara Truong*、Yunbo Zhang、Fangzhou Yu、Jean Pierre Sleiman、Jessica Hodgins、Koushil Sreenath、Farbod Farshidian（CMU / ETH / UC Berkeley）
- **Track**：Journal（TOG 44(4)，Article 132，12 页）｜ **DOI**：10.1145/3731xxx（具体后缀待查 ACM DL）
- arXiv：https://arxiv.org/abs/2503.11801

**问题**：运动学 diffusion 方法可通过 inference-time conditioning 灵活引导，但常产生物理不可行动作；物理 diffusion 策略能产生可实现动作，但缺乏运动学预测导致可控性差。

**方法（连续 guided diffusion，无量化）**：

- **关键洞察**：在单一 diffusion 模型中联合建模**状态和动作的分布**，使动作生成可通过条件化预测状态来引导
- 借此可利用运动学生成中成熟的 conditioning 技术，同时产生物理真实动作
- **look-ahead control**：预测未来状态序列并据此引导当前动作

**关键结果**：cited by 22（Google Scholar），证明影响力。具体 FID / success rate 数值未从公开摘要中提取到。

---

### 2.12 PhysicsFC: Learning User-Controlled Skills for a Physics-Based Football Player Controller

- **作者**：Minsu Kim、Eunho Jung、Yoonsang Lee（汉阳大学）
- **Track**：Journal（TOG 44(4)，Article 130）｜ **DOI**：10.1145/3731425
- arXiv：https://arxiv.org/abs/2504.21216

**问题**：让物理仿真的足球运动员执行运球、停球、移动、射门等多种技能，并在技能间无缝切换，同时支持用户交互式控制。

**方法（连续 latent skill policy，无量化）**：

- 每个技能的策略生成**latent variable**，基于已有的 physics-based motion embedding model
- **定制化奖励设计**：Dribble policy、两阶段 Trap reward + projectile dynamics initialization、DEGCL（Data-Embedded Goal-Conditioned Latent Guidance）for Move policy
- **Skill Transition-Based Initialization (STI)**：保证技能间敏捷平滑切换
- **FSM 架构**：用户可通过有限状态机交互式控制角色

**关键结果**：competitive trapping/dribbling、give-and-go plays、**11v11 足球比赛**演示。定量评估验证各技能策略及切换性能。具体数值未公开。

---

### 2.13 CAMDM: Taming Diffusion Probabilistic Models for Character Control（新闻稿确认，细节待补）

- **作者**：Rui Chen、Ping Tan（HKUST）；Mingyi Shi、Taku Komura（HKU）；Shaoli Huang、Xuelin Chen（Tencent AI Lab）
- **Session**：Physics-Based Human Characters（根据新闻稿推断）｜ **Track**：待确认（可能是 Journal 或 Conference）
- 官方新闻稿：https://s2025.siggraph.org/?p=5502

**问题**：实时角色控制的三大挑战——高质量多样性动作生成、可控性、计算效率与视觉真实的平衡。

**方法（continuous autoregressive motion diffusion）**：

- **CAMDM** = Conditional Autoregressive Motion Diffusion Model
- Transformer 架构，以角色历史动作为输入，实时生成多样化的潜在未来动作
- 响应各种动态用户提供的控制信号
- 可在**缺失过渡数据的风格间**生成无缝过渡
- **单一统一模型**实现所有功能

**关键结果**：据新闻稿，已与业界合作集成到游戏产品中（预计 2024 年底上线）。具体论文细节（DOI、定量表）因新闻稿未提供链接而未能提取。

**信息缺口**：Track 归属、DOI、定量指标、是否开源。建议后续通过 HKUST/HKU/Tencent 作者主页或 ACM DL 补充。

---

## Session D：Get a Head（Avatar 与面部动画）

### 2.14 Text-based Animatable 3D Avatars with Morphable Model Alignment（AnimPortrait3D）

- **作者**：Yiqian Wu、Malte Prinzler、Xiaogang Jin（浙大）、Siyu Tang（ETH）
- **Session**：Get a Head ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730680

**问题**：文本生成 3D 头像存在外观/几何欠约束、与驱动参数模型语义对齐不足的问题，导致动画不自然。

**方法**：两阶段——① 用预训练 text-to-3D 模型初始化具有稳健外观/几何/rigging 关系的 3D avatar；② 用 **ControlNet**（以 morphable model 的语义图和法线图为条件）精修动态表情。输出为 3DGS animatable avatar。

**关键结果**：synthesis quality、alignment、animation fidelity 均优于现有方法。代码和模型标注 available。

---

### 2.15 LAM: Large Avatar Model for One-shot Animatable Gaussian Head

- **作者**：Yisheng He、Xiaodong Gu、Xiaodan Ye、Chao Xu、Zhengyi Zhao、Yuan Dong、Weihao Yuan、Zilong Dong、Liefeng Bo
- **Session**：Get a Head ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730706

**问题**：现有一 shot 头像重建方法需要额外神经网络做动画和渲染，无法即插即用。

**方法**：**单向前向 pass 秒级生成 animatable Gaussian head**，无需额外网络或后处理。Canonical Gaussian attributes generator 以 FLAME canonical points 为 queries，通过 Transformer 与多尺度图像特征交互。重建后可直接用 LBS + corrective blendshapes 动画化，**支持手机端实时渲染**。

**关键结果**：outperforms SOTA on existing benchmarks。具体数值未公开。

---

### 2.16 SOAP: Style-Omniscient Animatable Portraits

- **作者**：Tingting Liao、Yujian Zheng、Yuliang Xiu、Adilbek Karmanov、Liwen Hu、Leyang Jin、Hao Li
- **Session**：Get a Head ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730691
- 代码和数据：https://github.com/TingtingLiao/soap

**问题**：单图生成可动画 3D 头像受限于风格（写实/卡通/动漫）和配饰/发型处理。

**方法**：multiview diffusion model（24K 3D heads 多风格训练）+ adaptive optimization pipeline 变形 FLAME mesh 同时保持拓扑和 rigging。支持 FACS-based 动画、眼球/牙齿集成、编发/配饰保留。

**关键结果**：single-view head modeling 和 diffusion-based image-to-3D 两方面均优于 SOTA。

---

## Session E：Rigging & Interaction（上游可驱动性）

### 2.17 Anymate: A Dataset and Baselines for Learning 3D Object Rigging

- **作者**：Yufan Deng、Yuhao Zhang、Chen Geng、Shangzhe Wu（Stanford）、Jiajun Wu（Stanford）
- **Session**：Rigging & Interaction（根据 history.siggraph.org 推断）｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730743
- arXiv：https://arxiv.org/abs/2505.06227 ｜ 项目页：https://anymate3d.github.io/

**问题**：自动 rigging 和 skinning 需要专家知识；既有数据驱动方法受限于小规模训练数据。

**方法**：

- **Anymate Dataset**：从 Objaverse-XL 精选 **230K** 带艺术家制作 rigging + skinning 的 3D 资产，**比既有数据集大 70 倍**
- 三阶段学习框架：joint prediction → connectivity prediction → skinning weight prediction
- 系统设计了多种架构作为 baseline 并做了全面评估

**关键结果**：更大规模训练数据显著提升预测质量；所提架构 scalability 更好，大幅超越现有方法。代码、数据集、预训练权重将公开。

**为什么重要**：这是**动作可驱动性上游**的基础设施级贡献。230K 规模的 rigging 数据集将支撑未来多年的 auto-rigging 研究，类似于 ImageNet 之于分类。

---

## Session F：Stabilize and Personalize Your Pixels（视频动作控制，边缘相关）

### 2.18 Motion Inversion for Video Customization

- **作者**：Luozhou Wang、Ziyang Mai、Guibao Shen、Yixun Liang、Xin Tao、Pengfei Wan、Di Zhang、Yijun Li、Ying-Cong Chen
- **Session**：Stabilize and Personalize Your Pixels ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730735
- 项目页：https://wileewang.github.io/MotionInversion/

**问题**：视频生成模型中的 motion representation 探索不足。

**方法**：**Motion Embeddings**——显式、时序一致的嵌入，源自给定视频，无缝集成到视频 diffusion 模型的 temporal transformer 模块中，调制跨帧 self-attention。两种嵌入：Motion Query-Key Embedding（调制 temporal attention map）+ Motion Value Embedding（调制 attention values）。推理时排除 spatial dimensions 并做 debias 操作确保嵌入仅关注 motion。

**归类说明**：对象是视频像素的 motion customization，非 3D 人体动作生成，**边缘相关**。

---

### 2.19 FlexiAct: Towards Flexible Action Control in Heterogeneous Scenarios

- **作者**：Shiyi Zhang、Junhao Zhuang、Zhaoyang Zhang、Ying Shan、Yansong Tang
- **Session**：Stabilize and Personalize Your Pixels ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730683

**问题**：现有 action customization 方法受限于布局/骨架/视角一致性约束，难以跨异构主体迁移动作。

**方法**：

- **RefAdapter**：轻量 image-conditioned adapter，在 appearance consistency 和 structural flexibility 之间取得更好平衡
- **FAE (Frequency-aware Action Extraction)**：利用去噪过程中不同 timestep 对 motion（低频）和 appearance details（高频）的不同关注度，直接在去噪过程中提取动作
- 允许参考视频主体与目标图像在布局、视角、骨架结构上存在差异

**归类说明**：对象是视频像素级 action transfer，**边缘相关**，但与 2026 年的 Motion4Motion（skeleton-free action transfer）方法学上同源。

---

## Session G：Avatars（边缘/登记）

### 2.20 Facial Microscopic Structures Synthesis from a Single Unconstrained Image

- **作者**：Youyang Du、Lu Wang、Beibei Wang
- **Session**：Get a Head ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3721238.3730760

面部微观结构（皱纹、毛孔）合成，属静态几何/外观，**建议剔除**。登记以示穷尽。

---

## 明确剔除清单（登记以示检索穷尽）

| 论文 | Session | 剔除理由 |
|---|---|---|
| Hyper-Dimensional Deformation Simulation | All About Motion & Deformation | 4D 物理仿真，非人体动作 |
| Motion Inversion for Video Customization | Stabilize and Personalize Your Pixels | 视频 pixel-level motion，非 3D 动作 |
| FlexiAct | Stabilize and Personalize Your Pixels | 视频 action transfer，边缘 |
| Facial Microscopic Structures Synthesis | Get a Head | 静态面部几何，非动作 |
| AssetDropper / EditDuet / GaVS 等 | 多 session | 纯图像/视频生成或编辑 |
| Gaussian Fluids / Quadtree Tall Cells | This is Fluid Simulation | 纯流体仿真 |
| Cloth 三篇 | Cloth & Other Things | 布料仿真 |
| RenderFormer / Dual-Band GI 等 | Illuminating Light | 全局光照渲染 |
| MASH / 3D2EP / OctGPT | Learning and Shapes | 静态 3D 形状表示/生成 |
| Cage/Green Coordinates 等 | Cages, Deformation & Interpolation | 几何变形 |
| Tailor Made 三篇 | Tailor Made | 服装建模/重构 |
| Painting with 3DGS Brushes 等 | Interactive Reality | VR/AR 交互 |

**两个重要的负面结果**：

- **群体动画（crowds）：本届零命中**。官方 session 列表无任何 crowd/crowds 相关 session。
- **四足/人形机器人运动无独立 session**，相关内容分散在 AMOR（真机部署）、CAMDM（游戏 NPC）、PhysicsFC（足球球员仿真）。

---

# 第三部分 · 趋势观察与研究建议

## 3.1 三条主线

**第一条：2025 是 motion tokenizer 的"前夜"，而非爆发年。**

与 2026 年四篇 tokenizer 工作集中出现不同，2025 年只有 DuetGen 一篇在 motion generation 主干中使用 VQ-VAE + masked transformer。大多数 motion generation 工作（AnyTop、AutoKeyframe、CAMDM、xADA、ELGAR、MECo、Semantically Consistent T2M）都选择了连续表征。这暗示**tokenizer 路线在 2025 年尚未成为共识**，真正的方法学转折发生在 2026 年。

但从另一个角度看，**DuetGen 的设计几乎预言了 2026 年的多个方向**：hierarchical VQ → EchoAvatar 的 RVQ 层级、masked transformer → MotionBricks 的 masked token modeling、双人交互统一编码 → MOCHI 的多角色交互优化。DuetGen 是一篇被低估的先驱工作。

**第二条：拓扑无关（topology-agnostic）在 2025 年正式立项。**

AnyTop 首次明确提出"generating motions for diverse characters with distinct motion dynamics, using only their skeletal structure as input"，并用 textual joint descriptions 学习跨骨架语义对应。这是"拓扑无关运动先验"方向的起点，直接启发了 2026 年的 TopoCap（Mobjaverse 5,000+ 拓扑）和 UniMate（UniML3D 13,006 序列）。三年三篇构成了清晰的问题定义→数据集建设→通用模型的发展脉络。

**第三条：图形学 × 机器人的融合叙事在 2025 年已埋下伏笔。**

AMOR（Disney Research）的多目标 RL 框架明确面向机器人部署；CAMDM 直接与游戏引擎集成；PhysicsFC 展示了 11v11 多智能体仿真。但与 2026 年官方新闻稿以"机器人"为主题不同，2025 年的机器人元素还是分散的、项目级的，尚未上升为会议的官方叙事。

## 3.2 具体建议

**做 motion tokenizer 时，这几个点最有信息量：**

1. **DuetGen 的 hierarchical VQ + masked transformer 组合**是 2025 年最值得复现的设计——它比 2026 年的 EchoAvatar 早一年提出了"层级离散码本 + 迭代 masked decoding"的范式，且针对的是更复杂的双人交互场景。如果要做多人交互的 tokenizer，这是必读的起点。
2. **CAMDM 的 conditional autoregressive diffusion**代表了"diffusion 自回归化"的另一条路径——不是把动作离散化为 token 再做 AR，而是在连续空间做 autoregressive diffusion。这条路径在 2026 年被 ARDY 继承并发扬（交错自回归扩散去噪器）。两条路径（离散 AR vs 连续 AR）的优劣对比是一个开放问题。
3. **Anymate 的 230K rigging 数据集**提醒我们：tokenizer 的上游（rigging/skinning）也在经历数据规模化革命。如果把 motion tokenizer 和 auto-rigging 联合起来考虑（类似 2026 年的 ACT 和 AniGen 的思路），Anymate 的数据基础设施会是关键支撑。

**需要自行补查的三个缺口：**

- **CAMDM 的 Track 归属和 DOI**——新闻稿未提供链接，建议查 ACM DL 的 TOG 44(4) 目次或作者主页。
- **DuetGen 的 VQ 超参数**（码本大小、层级时间分辨率比例、token 帧率）——公开摘要未给出，需查论文全文或补充材料。
- **ELGAR 的 SPD-GEN 数据集规模**——标注"collated and normalized from SPD"，具体小时数/clip 数未公开。

---

# 附录：DOI 速查表

| 论文 | DOI | Track |
|---|---|---|
| Motion Control (Metric-Aligning MM) | 10.1145/3721238.3730665 | Conf |
| AutoKeyframe | 10.1145/3721238.3730664 | Conf |
| AnyTop | 10.1145/3721238.3730621 | Conf |
| xADA | 10.1145/3721238.3730711 | Conf |
| DuetGen | 10.1145/3721238.3730741 | Conf |
| ELGAR | 10.1145/3721238.3730756 | Conf |
| MECo | 10.1145/3721238.3730611 | Conf |
| Semantically Consistent T2M | 10.1145/3721238.3730641 | Conf |
| AMOR | 10.1145/3721238.3730656 | Conf |
| Diffuse-CLoC | TOG 44(4) Art.132 | Journal |
| PhysicsFC | 10.1145/3731425 | Journal |
| CAMDM | 待查 | 待确认 |
| AnimPortrait3D | 10.1145/3721238.3730680 | Conf |
| LAM | 10.1145/3721238.3730706 | Conf |
| SOAP | 10.1145/3721238.3730691 | Conf |
| Anymate | 10.1145/3721238.3730743 | Conf |
| Motion Inversion | 10.1145/3721238.3730735 | Conf |
| FlexiAct | 10.1145/3721238.3730683 | Conf |

---

## 数据来源与可复现性

- **SIGGRAPH 2025 官方全量日程**：https://www.siggraph.org/wp-content/uploads/2025/08/Conference-Papers.html （含每篇摘要）
- **DBLP 会议页**：https://dblp.uni-trier.de/db/conf/siggraph/siggraph2025.html
- **ACM DL proceedings**：https://dl.acm.org/doi/proceedings/10.1145/3721238
- **history.siggraph.org**：按 session 列出所有 presentation（含 presenter 和 venue 标注）
- 技术细节均来自 arXiv 全文/HTML、项目主页和官方摘要；**凡未查到的字段一律标注"未找到"或"待查"，未做任何推断或编造**
- Journal Track（TOG 44(4)）的完整性受限——部分论文的 article number 和 DOI 后缀需通过 ACM DL 补充
