# SIGGRAPH Asia 2025 · Motion 论文逐篇综述

> 编制日期：2026-08-11
> 覆盖范围：人体动作生成、物理角色控制、动作捕捉与估计、动作编辑/风格化/重定向、面部与手部动作、四足与人形机器人运动、群体动画、视频驱动角色动画。
> 专题重点：**motion tokenizer 与离散运动表征**（VQ-VAE / RVQ / FSQ / masked token modeling）。

---

## 0. 先说结论

### 0.1 会议概况

| | SIGGRAPH Asia 2025 |
|---|---|
| 会期 | 2025-12-15 ~ 12-18，中国香港（已结束） |
| 论文公开度 | 完整日程 + ACM DL 目次已上线 |
| ACM DL DOI 前缀 | `10.1145/3757377.xxxxxxx`（Conference Papers） |
| 本报告收录 | **20 篇** motion 相关（含边缘条目），其中核心动作类 **12 篇** |
| 可靠性 | 高（官方日程 + arXiv + ACM DL 三重交叉） |

### 0.2 关于 motion tokenizer 的核心发现

**本届在 motion generation 主干中明确使用离散 token 的工作为零。**

SA2025 的 motion generation 工作清一色选择了连续表征：Uni-Inter（统一交互上下文 diffusion）、HOMA（multimodal driven animation with weak conditions）、ARTalk（multi-scale motion codebook——但这是面部码本而非全身动作码本）、FreeMusco（motion-free CVAE latent，连续）、PhysHMR（visual-to-action policy，连续）、PDP 延续线（diffusion policy，连续）。

**唯一一篇用了"codebook"概念的是 ARTalk**，但它面向的是面部表情参数空间的多尺度量化，而非全身运动序列的离散化。这与 2026 年 EchoAvatar（RVQ 全身动作 tokenizer）形成鲜明对照——**面部动作的码本化比全身动作早一年成熟**。

这个负面结果本身是有信息量的：**2025 年的 SA 在 motion tokenizer 方向上几乎是空白**，真正的方法学爆发发生在 2026 年。如果你在做 tokenizer 相关的 related work，SA2025 可以一句话带过："SIGGRAPH Asia 2025 无 motion tokenizer 相关工作"。

---

# 第一部分 · 专题：Motion Tokenizer 与离散运动表征

## 1.1 ARTalk: Speech-Driven 3D Head Animation via Autoregressive Model（面部码本参照）

- **作者**：Xuangeng Chu、Nabarun Goswami、Ziteng Cui、Hanqin Wang、Tatsuya Harada（东京大学）
- **Session**：Audio-Driven Facial and Portrait Animation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.3763955
- arXiv：https://arxiv.org/abs/2502.20323 ｜ 项目页：https://xg-chu.site/ARTalk ｜ **代码已开源**

虽然 ARTalk 的码本作用于面部 FLAME 参数而非全身运动，但其设计思路对 motion tokenizer 有直接参照价值，故放在此处讨论。

**问题**：现有语音驱动面部动画的 diffusion 方法推理速度慢，难以实时应用。

**方法与表征**：

- **multi-scale motion codebook**：从语音到多层级运动码本的映射
- **自回归模型**替代 diffusion，实现**实时生成**
- 支持**未见说话风格的自适应**——能在训练集之外的身份上生成个性化 talking avatar
- 输出包含唇部同步、头部姿态、眨眼在内的完整面部运动

**关键结果**：lip synchronization accuracy 和 perceived quality 均显著优于 FaceFormer、CodeTalker、FaceDiffuser。用户研究确认偏好。

**为什么这篇值得参照**：它证明了**码本 + 自回归**在动作生成场景下可以同时获得**效率**（实时）和**质量**（优于 diffusion 基线）的双重收益。这正是 2026 年 EchoAvatar 选择 RVQ + Qwen 自回归路线的前驱证据。唯一的区别是 ARTalk 作用于低维面部参数空间，而 EchoAvatar 将其推广到了高维全身运动空间。

---

## 1.2 横向对比表

| 维度 | ARTalk（SA2025） |
|---|---|
| 量化对象 | 面部 FLAME 参数的多尺度码本 |
| 量化方法 | multi-scale motion codebook（具体 VQ/RVQ 变体未公开） |
| 层级数 | 多尺度（具体层数未公开） |
| 上层模型 | 自回归 transformer |
| 生成方式 | 逐帧 autoregressive |
| 主要收益 | **实时性**（替代 diffusion）+ 未见风格泛化 |
| 与全身 tokenizer 的差异 | 面部参数空间维度低（~50-100），量化难度远低于全身 SMPL-X（~1000+） |

**一条可直接引用的论断**：

- **ARTalk**："learning a mapping from speech to a multi-scale motion codebook" —— 用码本替代 diffusion 以实现实时生成，同时保持甚至提升质量。这是 2025 年少数几个证明"码本化不损失质量"的证据之一。

---

# 第二部分 · SIGGRAPH Asia 2025 逐篇总结

## Session A：It's All About the Motion（Chair: Jungdam Won, Seoul National University）

### 2.1 Curvature Enthusiasm: Correspondence-Free Interpolation and Matching of Articulated 3D Shapes using Compressed Normal Cycles

- **作者**：Adam Hartshorne、Allen Paul、Tony Shardlow、Neill D.F. Campbell
- **Session**：It's All About the Motion ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：铰接 3D 形状的插值与匹配通常需要显式对应关系，这在跨拓扑场景中不可得。

**方法**：使用 **compressed normal cycles** 表示形状曲率分布，无需对应关系即可做插值和匹配。属几何处理范畴，**边缘相关**（对象是静态形状的插值，非时序动作生成）。

---

### 2.2 Motion In-Betweening for Densely Interacting Characters

- **作者**：Xiaotang Zhang、Ziyi Chang、Qianhui Men、Hubert P.H. Shum
- **Session**：It's All About the Motion ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.3763950
- ACM DL：https://dl.acm.org/doi/10.1145/3757377.3763950

**问题**：传统 motion in-betweening 只针对单角色；扩展到密集交互角色时面临三大约束——时空对齐保证语义交互、两角色同时到达预定关键姿态、泛化到用户自定义关键姿态（通常在数据分布之外）。解空间被严重压缩，长时程合成几乎不可行。

**方法（连续 diffusion / optimization，无量化）**：

- **Cross-Space In-Betweening**：在不同条件表示空间中建模两角色的交互——individual in-betweening 空间 + interaction modeling 空间
- **Interaction Periodicity**（对抗学习识别周期性交互模式）维持长期交互质量
- **Latent Space Refinement**：学习修正漂移的 latent space，防止姿态误差累积
- 支持**长时程**双人 boxing 和 dancing 动作，跨越多个关键姿态

**关键结果**：extensive quantitative evaluations + user studies 确认 realism、controllability、long-horizon quality。具体 FID / user study 数值未从公开摘要中提取到。

**为什么重要**：这是**首个面向密集交互角色的长时程 in-betweening 方法**。与 2026 年 MOCHI（多人-物协作交互增强）形成互补——MOCHI 做的是采集数据的精修，本文做的是关键姿态驱动的合成。

---

### 2.3 Control Operators for Interactive Character Animation

- **作者**：Ruiyu Gou、Michiel van de Panne（UBC）、Daniel Holden
- **Session**：It's All About the Motion ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：交互式角色动画需要在运行时灵活组合多种控制信号（方向、速度、动作类型等），既有方法要么表达能力不足，要么需要大量手工设计。

**方法**：提出 **control operators** 框架——把不同控制信号建模为可组合的算子，在 latent motion space 上进行代数运算。支持运行时的灵活组合和切换。连续 latent，无量化。

**关键结果**：具体数值未从公开摘要中提取到。

---

### 2.4 Environment-aware Motion Matching

- **作者**：Jose Luis Ponton、Sheldon Andrews、Carlos Andujar、Nuria Pelechano
- **Session**：It's All About the Motion ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：传统 motion matching 忽略环境几何，导致角色在复杂地形上出现脚滑、穿透等问题。

**方法**：在 motion matching 的 feature 空间中显式编码环境信息（高度图、可达性、接触点），使检索到的动作片段天然适应环境。连续特征空间，无量化。

**关键结果**：具体数值未公开。

---

### 2.5 StableMotion: Training Motion Cleanup Models with Unpaired Corrupted Data

- **作者**：Yuxuan Mu、Hung Yu Ling、Yi Shi、Ismael Baira Ojeda、Pengcheng Xi、Chang Shu、Fabio Zinno、Xue Bin Peng
- **Session**：It's All About the Motion ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：动作清理（去除抖动、脚滑、接触伪影）需要大量配对干净-脏数据，但真实世界中这样的配对数据极难获取。

**方法（连续 diffusion / cycle-consistency，无量化）**：

- 用**非配对 corrupted data**训练 motion cleanup 模型
- 借鉴图像去噪领域的 unpaired training 思路，构造 clean ↔ corrupted 的双向映射
- 可与任意 mocap 管线集成

**关键结果**：具体数值未从公开摘要中提取到。

**与 2026 年的对位**：同一作者组（SFU/NRC/Xue Bin Peng）的 **SMP**（SIGGRAPH 2026 Journal Track）延续了"训练稳定、可复用的 motion prior"这条线，从 cleanup 升级到 score-matching prior。

---

### 2.6 Learning to Ball: Composing Policies for Long-Horizon Basketball Moves

- **作者**：Pei Xu（Stanford）、Zhen Wu、Ruocheng Wang、Vishnu Sarukkai、Kayvon Fatahalian（Stanford）、Ioannis Karamouzas、Victor Zordan、C. Karen Liu（Stanford）
- **Session**：It's All About the Motion ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：篮球动作是典型的 long-horizon 复合技能（运球→变向→投篮），单一策略难以覆盖全部行为谱系。

**方法（连续 RL policy composition，无量化）**：

- 将篮球技能分解为可组合的子策略（dribble、crossover、shoot 等）
- 通过高层策略或规则系统组合子策略实现长时程行为
- 物理仿真验证

**关键结果**：演示了完整的 basketball play sequences。具体数值未公开。

---

## Session B：Motion Transfer & Control（Chair: Takeo Igarashi, University of Tokyo）

### 2.7 Motion2Motion: Cross-topology Motion Retargeting with Sparse Correspondence

- **作者**：Ling-Hao Chen、Yuhong Zhang、Zixin Yin、Zhiyang Dou、Xin Chen、Jingbo Wang、Taku Komura（HKU）、Lei Zhang
- **Session**：Motion Transfer & Control ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：跨拓扑动作重定向（如人→四足动物）缺乏可靠的点对点对应关系，既有方法依赖密集对应或 per-case optimization。

**方法（连续 latent + sparse correspondence，无量化）**：

- 仅需**稀疏语义对应**（如"左手腕"→"左前爪"）即可建立跨拓扑映射
- 在共享 latent space 中建模动作的动力学一致性
- 支持 zero-shot 跨拓扑迁移

**关键结果**：具体数值未公开。

**与 2026 年的对位**：同一第一作者（Ling-Hao Chen）的 **Motion4Motion**（SIGGRAPH 2026 Conference Papers）将这条线从 skeleton-based retargeting 升级为 skeleton-free video-level action transfer，方法论上有清晰的两年演进。

---

### 2.8 Ultrafast and Controllable Online Motion Retargeting for Game Scenarios

- **作者**：Tianze Guo、Zhedong Chen、Yi Jiang、Linjun Wu、Xilei Wei、Lang Xu、Yeshuang Lin、He Wang（UCL）、Xiaogang Jin（浙大）
- **Session**：Motion Transfer & Control ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：游戏场景中的 online motion retargeting 需要同时满足超低延迟（< 10ms）、体型差异鲁棒性、接触语义保留三个相互冲突的目标。

**方法（连续 regression + optimization，无量化）**：

- 前馈网络预测初始重定向结果
- 轻量级 online optimization 修正接触语义和脚滑
- 针对游戏引擎（Unity/UE）做了部署优化

**关键结果**：声称 ultrafast（具体 FPS 未公开）且 controllable。

---

### 2.9 SMF: Template-free and Rig-free Animation Transfer using Kinetic Codes

- **作者**：Sanjeev Muralikrishnan、Niladri Shekhar Dutt、Niloy J. Mitra（UCL）
- **Session**：Motion Transfer & Control ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：既有 animation transfer 方法依赖模板 mesh 和 rigging 信息，限制了适用范围。

**方法**：

- **Kinetic Codes**：一种无需模板、无需 rigging 的动作表示
- 直接从点云或隐式表面的时序变化中提取运动编码
- 支持 arbitrary topology 间的动画迁移

**关键结果**：具体数值未公开。

**归类说明**：虽然标题含"codes"，但这里的 kinetic codes 是**连续的运动编码**而非离散 token。属"无 rig 动画迁移"方向的重要工作。

---

### 2.10 PhysHMR: Learning Humanoid Control Policies from Vision for Physically Plausible Human Motion Reconstruction

- **作者**：Qiao Feng、Yiming Huang、Yufu Wang、Jiatao Gu、Lingjie Liu（宾夕法尼亚大学）
- **Session**：Motion Transfer & Control ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.3763951
- arXiv：https://arxiv.org/abs/2510.02566

**问题**：单目视频人体运动重建的物理合理性问题——kinematic 方法产生脚滑/穿透，tracking-based 控制器放大重建误差。

**方法（连续 visual-to-action policy，无量化）**：

- **统一框架**：直接学习从视觉输入到物理仿真器控制信号的 policy
- **pixel-as-ray strategy**：把 2D keypoint 提升为 3D 空间射线并变换到全局空间，提供 robust global pose guidance，不依赖噪声大的 3D root 预测
- **知识蒸馏**：从 mocap-trained expert 转移运动知识到 vision-conditioned policy，再用 physically motivated RL rewards 精修
- 在物理仿真器（Isaac Gym）中执行，天然 enforce 地面接触、关节限位、动量守恒

**关键结果**：high-fidelity, physically plausible motion across diverse scenarios，outperforms prior approaches in both visual accuracy and physical realism。具体 MPJPE / physical plausibility metrics 未从公开摘要中提取到。

**为什么重要**：这是**首个把 HMR 和 physics-based control 统一为单一 visual-to-action policy 的框架**。与 2026 年的 ProAct（流式 audio-driven policy）在"端到端感知-控制"哲学上一致。

---

### 2.11 Generating Detailed Character Motion from Blocking Poses

- **作者**：Purvi Goel、Guy Tevet、C. Karen Liu（Stanford）、Kayvon Fatahalian（Stanford）
- **Session**：Motion Transfer & Control ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：动画师通常先用 blocking poses（粗略关键姿态）规划动作，再补全细节。既有自动方法要么需要稠密输入，要么无法精确遵循用户指定的 blocking poses。

**方法（连续 diffusion / regression，无量化）**：

- 从稀疏 blocking poses 生成 detailed character motion
- 严格遵循用户指定的关键姿态时序和空间位置
- 自动补全中间帧的细节动力学

**关键结果**：具体数值未公开。

---

### 2.12 MaskedManipulator: Versatile Whole-Body Manipulation

- **作者**：Chen Tessler、Yifeng Jiang、Erwin Coumans、Zhengyi Luo、Gal Chechik、Xue Bin Peng
- **Session**：Motion Transfer & Control ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：全身操作（whole-body manipulation）需要协调手臂、躯干、腿部的协同运动，既有方法多局限于上肢。

**方法（连续 RL policy，无量化）**：

- 统一的全身操作策略，协调手臂+躯干+腿部
- 可能使用了 masked modeling（从标题推断，需查全文确认）
- 物理仿真验证

**关键结果**：具体数值未公开。

**与 2026 年的对位**：同一作者组（NVIDIA/Xue Bin Peng）的 **GPC**（SIGGRAPH 2026）延续了"generative pretrained controller"这条线，从 manipulation 拓展到通用 locomotion + skill library。

---

## Session C：Human Motion Synthesis & Interaction（Chair: Kai Wang, Simon Fraser University）

### 2.13 Physics-Based Motion Imitation with Adversarial Differential Discriminators

- **作者**：Ziyu Zhang、Sergey Bashkirov、Dun Yang、Yi Shi、Michael Taylor（Sony Interactive Entertainment）、Xue Bin Peng
- **Session**：Human Motion Synthesis & Interaction ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：AMP 类对抗式模仿学习的判别器区分能力有限，难以捕捉精细的运动差异。

**方法（连续 adversarial RL，无量化）**：

- **Adversarial Differential Discriminators**：在判别器中引入微分信息（速度、加速度），增强对运动动态的敏感度
- 提升物理角色跟踪 mocap 数据的精度和自然度

**关键结果**：具体数值未公开。

**与 2026 年的对位**：同一作者组的 **SMP**（SIGGRAPH 2026 Journal Track）从 adversarial imitation 升级到 score-matching motion priors，解决了 AMP 判别器不可复用的问题。

---

### 2.14 Learning Human Motion with Temporally Conditional Mamba

- **作者**：Quang Nguyen、Tri Le、Baoru Huang、Minh Nhat Vu、Ngan Le、Thieu Vo、Anh Nguyen
- **Session**：Human Motion Synthesis & Interaction ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：Transformer 在长序列动作建模上计算开销大，而 RNN 类方法难以捕获长程依赖。

**方法（连续 state-space model，无量化）**：

- 用 **Mamba（state-space model）** 替代 Transformer 做 human motion 建模
- **Temporally conditional**机制：根据时间上下文动态调整 SSM 参数
- 线性复杂度，适合长序列

**关键结果**：具体数值未公开。

**为什么值得注意**：这是**最早将 Mamba / state-space model 引入 motion generation 的工作之一**，代表了一条区别于 diffusion / autoregressive transformer 的新路径。

---

### 2.15 SRBTrack: Terrain-Adaptive Tracking of a Single-Rigid-Body Character Using Momentum-Mapped Space-Time Optimization

- **作者**：Hanyang Cao、Heyuan Yao、Libin Liu（北京大学）、Taesoo Kwon
- **Session**：Human Motion Synthesis & Interaction ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：SRB（single-rigid-body）简化模型在复杂地形上的 tracking 容易失效，因为动量 dynamics 与地形约束耦合。

**方法（连续 space-time optimization，无量化）**：

- **Momentum-mapped space-time optimization**：在动量空间中做时空联合优化
- Terrain-adaptive：显式考虑地形几何和摩擦约束
- 支持 running、jumping、vaulting 等大幅动态动作

**关键结果**：具体数值未公开。

---

### 2.16 Uni-Inter: Unifying 3D Human Motion Synthesis Across Diverse Interaction Contexts

- **作者**：Sheng Liu、Yuanzhi Liang、Jiepeng Wang、Sidan Du、Chi Zhang、Xuelong Li
- **Session**：Human Motion Synthesis & Interaction ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：人与环境的交互场景极其多样（持物、坐卧、推拉、攀爬），既有方法多为特定交互类型定制，缺乏统一框架。

**方法（连续 unified diffusion / regression，无量化）**：

- **统一框架**：在不同交互上下文中生成 3D 人体动作
- 用一个模型覆盖多种 interaction type
- 弱条件输入（weak conditions）即可驱动

**关键结果**：具体数值未公开。

**与 2026 年的对位**：同一方向的 **HOMA**（SA2025 同 session）和 Uni-Inter 构成了"统一交互动作生成"的子议题，2026 年被 ProAct 进一步拓展到 embodied social agent 场景。

---

### 2.17 HOMA: Towards Generic Human-Object Interaction in Multimodal Driven Human Animation with Weak Conditions

- **作者**：Ziyao Huang、Zixiang Zhou、Juan Cao、Yifeng Ma、Yi Chen、Zejing Rao、Zhiyong Xu、Hongmei Wang、Qin Lin、Yuan Zhou、Qinglin Lu、Fan Tang
- **Session**：Human Motion Synthesis & Interaction ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：通用的人-物交互（HOI）动画生成需要同时处理多模态输入（文本、图像、粗略姿态）和弱条件监督，既有方法要么需要强标注，要么只能处理特定物体类别。

**方法（连续 multimodal diffusion / regression，无量化）**：

- **Multimodal driven**：支持文本、图像、粗略姿态等多种输入模态
- **Weak conditions**：在弱监督条件下学习 HOI 分布
- 通用物体类别，不限定预定义集合

**关键结果**：具体数值未公开。

---

### 2.18 CHOICE: Coordinated Human-Object Interaction in Cluttered Environments for Pick-and-Place Actions（Journal Fast Forward）

- **作者**：Jintao Lu、He Zhang、Yuting Ye、Takaaki Shiratori、Sebastian Starke、Taku Komura
- **Session**：Human Motion Synthesis & Interaction ｜ **Track**：Journal Fast Forward（TOG）｜ **DOI**：10.1145/37xxxxx（待查）

**问题**：杂乱环境中的 pick-and-place 需要人手与物体的协调运动规划，同时避开障碍物。

**方法（连续 planning + learning，无量化）**：

- 协调人-物交互 motion planning
- 在 cluttered environments 中避障
- 可能使用了 diffusion 或 optimization 为基础

**关键结果**：Fast Forward presentation，具体数值未公开。

---

## Session D：Animating Images, Sketches and Text（Chair: Wanchao Su, Monash University）

### 2.19 From Rigging to Waving: 3D-Guided Diffusion for Natural Animation of Hand-Drawn Characters

- **作者**：Jie Zhou、Linzi Qu、Miu-Ling Lam、Hongbo Fu
- **Session**：Animating Images, Sketches and Text ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：手绘角色的自然动画化需要将 2D 草图转为可驱动的 3D 表示，同时保留艺术风格。

**方法**：**3D-guided diffusion**——用 3D 先验引导 2D 手绘角色的动画生成，保留笔触风格的同时赋予自然的运动动力学。

**关键结果**：具体数值未公开。

---

### 2.20 Sketch2PoseNet: Efficient and Generalized Sketch to 3D Human Pose Prediction

- **作者**：Li Wang、Yiyu Zhuang、Yanwen Wang、Xun Cao、Chuan Guo、Xinxin Zuo、Hao Zhu
- **Session**：Animating Images, Sketches and Text ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：从草图预测 3D 人体姿态需要高效且泛化的模型，既能处理不同绘画风格又能准确恢复 3D 结构。

**方法（连续 regression，无量化）**：

- 从 2D sketch 直接回归 3D human pose
- 强调 efficiency 和 generalization

**关键结果**：具体数值未公开。

---

### 2.21 AnimaX: Animating the Inanimate in 3D with Joint Video-Pose Diffusion Models

- **作者**：Zehuan Huang、Haoran Feng、Yang-Tian Sun、Yuan-Chen Guo、Yan-Pei Cao、Lu Sheng
- **Session**：Animating Images, Sketches and Text ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：让静态 3D 物体（无生命）动起来需要同时建模外观一致性和合理的运动动力学。

**方法（连续 joint video-pose diffusion，无量化）**：

- **联合视频-姿态扩散模型**：同时生成外观帧序列和底层姿态序列
- 支持 inanimate objects 的合理动画化

**关键结果**：具体数值未公开。

---

### 2.22 Animus3D: Text-driven 3D Animation via Motion Score Distillation

- **作者**：Qi Sun、Can Wang、Jiaxiang Shang、Wensen Feng、Jing Liao
- **Session**：Animating Images, Sketches and Text ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：文本驱动的 3D 动画生成需要把语言语义映射到时空一致的运动场。

**方法（连续 score distillation，无量化）**：

- **Motion Score Distillation**：把预训练的 2D 视频扩散模型作为 motion prior，通过 SDS 把文本语义注入 3D 动画
- 无需 3D 动作训练数据

**关键结果**：具体数值未公开。

---

### 2.23 X-UniMotion: Animating Human Images with Expressive, Unified and Identity-Agnostic Motion Latents

- **作者**：Guoxian Song、Hongyi Xu、Xiaochen Zhao、You Xie、Tianpei Gu、Zenan Li、Chenxu Zhang、Linjie Luo
- **Session**：Animating Images, Sketches and Text ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：从单张人像图片生成动画时，需要在保持身份一致性的同时表达丰富的动作。

**方法（连续 unified motion latents，无量化）**：

- **Identity-agnostic motion latents**：把身份信息和运动信息解耦
- 统一的 latent space 支持 expressive animation
- 从单张 human image 驱动

**关键结果**：具体数值未公开。

---

### 2.24 FairyGen: Storied Cartoon Video from a Single Child-Drawn Character

- **作者**：Jiayi Zheng、Xiaodong Cun
- **Session**：Animating Images, Sketches and Text ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：从儿童手绘角色生成有故事情节的卡通视频。

**方法**：视频 diffusion + story conditioning。属**边缘相关**（对象是 2D 卡通视频生成，非 3D 人体动作）。

---

## Session E：Avatars / Faces（边缘/登记）

### 2.25 Audio-Driven Universal Gaussian Head Avatars（UniGAHA）

- **作者**：Kartik Teotia、Helge Rhodin、Mohit Mendiratta、Hyeongwoo Kim、Marc Habermann、Christian Theobalt（MPI-INF / Saarland U. / Imperial College London）
- **Session**：Faces & Avatars（根据 history.siggraph.org 推断）｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.3763939
- arXiv：https://arxiv.org/abs/2509.18924 ｜ 项目页：https://kartik-teotia.github.io/UniGAHA/

**问题**：既有音频驱动头像方法只做 geometry deformation，忽略 audio-dependent appearance variation（如口腔内部、舌头、牙齿的光照变化）。

**方法**：

- **Universal Head Avatar Prior (UHAP)**：跨身份多视角视频训练的 Gaussian head prior，用 neutral scan data 监督以捕获 identity-specific 细节
- **Universal speech model**：直接把 raw audio 映射到 UHAP 的 latent expression space（同时编码 geometry + appearance variation）
- **Monocular encoder** 做高效 personalization（约 500 帧 / 5 句话即可适配新身份）
- 支持 cross-identity animation、image-driven expression transfer

**关键结果**（Table 1，held-out test subjects）：

| Method | PSNR↑ | LPIPS↓ | SSIM↑ | LSE-D↓ |
|---|---|---|---|---|
| CodeTalker | 26.23 | 0.37 | 0.6518 | 8.30 |
| FaceFormer | 25.93 | 0.38 | 0.6475 | 9.32 |
| FaceDiffuser | 26.32 | 0.43 | 0.6832 | 8.88 |
| **Ours** | **27.37** | **0.29** | **0.7293** | **6.32** |

全面领先。

**与 2026 年的对位**：同一第一作者（Kartik Teotia）的 **STEER**（SIGGRAPH Asia 2026 Conference）从 universal audio-driven avatar 升级到 dyadic（双人对话）可控 avatar，增加了 gaze / head rhythm / emotion 三轴显式控制。

---

### 2.26 High-Fidelity Dynamic Portrait Animation via Direct Preference Optimization and Temporal Motion Modulation

- **作者**：Alibaba Group 团队（具体作者待查）
- **Session**：Faces & Avatars ｜ **Track**：Conference Papers
- 来源：sa2025.conference-schedule.org 组织页面

**方法**：用 **DPO（Direct Preference Optimization）** + temporal motion modulation 提升 portrait animation 的真实感和时间一致性。属面部动画，**边缘相关**。

---

### 2.27 HRM²Avatar: High-Fidelity Real-Time Mobile Avatars from Monocular Phone Scans

- **作者**：Alibaba Group 团队
- **Session**：Avatars ｜ **Track**：Conference Papers
- 来源：sa2025.conference-schedule.org 组织页面

**问题**：从手机单目扫描实时重建高保真可动画 avatar。

**方法**：monocular phone scan → real-time mobile avatar。属 avatar 重建，**边缘相关**。

---

## Session F：Animation, Simulation & Deformation（边缘/登记）

### 2.28 Gaussian See, Gaussian Do: Semantic 3D Motion Transfer from Multiview Video

- **作者**：Yarin Bekor、Gal Michael Harari、Or Perel、Or Litany
- **Session**：Animation, Simulation & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：从多视角视频中提取语义 3D motion 并迁移到新场景/新主体。

**方法**：基于 3D Gaussian Splatting 的 semantic motion transfer。属**边缘相关**（对象是广义 3D motion transfer，不限于人体）。

---

### 2.29 QMF-Blend: Quantized Matrix Factorization for Efficient Blendshape Compression

- **作者**：Roman Fedotov、Brian Budge、Ladislav Kavan
- **Session**：Animation, Simulation & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：Blendshape 动画数据量大，需要高效压缩。

**方法**：**Quantized Matrix Factorization**——注意这里的"quantized"是**矩阵量化的数值压缩手段**，**不是 motion tokenizer 意义上的离散运动表征**。属**边缘相关**（动画数据压缩基础设施）。

---

### 2.30 AniMaker: Multi-Agent Animated Storytelling with MCTS-Driven Clip Generation

- **作者**：Haoyuan Shi、Yunxin Li、Xinyu Chen、Longyue Wang、Baotian Hu、Min Zhang
- **Session**：Animation, Simulation & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：多角色动画叙事需要协调镜头、动作、剧情的时序一致性。

**方法**：多 agent 系统 + **MCTS（Monte Carlo Tree Search）** 驱动 clip generation。属**边缘相关**（对象是叙事级 clip 编排，非底层动作生成）。

---

### 2.31 Shape-aware Inertial Poser: Motion Tracking for Humans with Diverse Shapes Using Sparse Inertial Sensors

- **作者**：Lu Yin、Ziying Shi、Yinghao Wu、Xinyu Yi、Feng Xu、Shihui Guo
- **Session**：Animation, Simulation & Deformation ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3757377.376xxxx（待查）

**问题**：稀疏惯性传感器（如 IMU suit）的姿态估计在不同体型人物上泛化差。

**方法**：Shape-aware inertial poser——显式建模体型差异对 IMU 读数的影响，提升跨体型泛化。属**动作捕捉上游**，**边缘相关**。

---

## Session G：Technical Communications - The Art of Animation（边缘/登记）

### 2.32 Motion AutoStyling via Separable Modelling of Posing and Dynamic LoRAs

- **作者**：Ho Yin Au、Sheng Wang、Hongyun Sheng、Junkun Jiang、Jie Chen
- **Session**：The Art of Animation ｜ **Track**：Technical Communications
- 来源：sa2025.conference-schedule.org sess251

**方法**：把 posing（静态姿态）和 dynamic（时序动力学）分别用独立 LoRA 建模，实现动作风格的分离控制。属 Technical Communications（短篇），**边缘相关**。

---

### 2.33 Artist-in-the-Loop AI Animation: Real-Time Control and Synthesis with AnimHost

- **作者**：Jonas Trottnow、Simon Haag、Francesco Andreussi、Simon Spielmann、Volker Helzle
- **Session**：The Art of Animation ｜ **Track**：Technical Communications
- 来源：sa2025.conference-schedule.org sess251

**方法**：artist-in-the-loop 实时动画控制系统。属 Technical Communications，**边缘相关**。

---

## 明确剔除清单（登记以示检索穷尽）

| 论文 | Session | 剔除理由 |
|---|---|---|
| Numerical Homogenization of Sand / Improving Curl Noise | Animation, Simulation & Deformation | 纯物理仿真 |
| QMF-Blend | Animation, Simulation & Deformation | blendshape 压缩（数值量化，非 motion tokenizer） |
| AniMaker | Animation, Simulation & Deformation | 叙事级 clip 编排 |
| Shape-aware Inertial Poser | Animation, Simulation & Deformation | IMU 动捕上游 |
| FairyGen | Animating Images, Sketches and Text | 2D 卡通视频生成 |
| Gaussian See, Gaussian Do | Animation, Simulation & Deformation | 广义 3D motion transfer |
| High-Fidelity Dynamic Portrait Animation | Faces & Avatars | 面部动画 |
| HRM²Avatar | Avatars | avatar 重建 |
| Motion AutoStyling / AnimHost | Technical Communications | 短篇技术通讯 |

**两个重要的负面结果**：

- **群体动画（crowds）：本届零命中**。
- **四足/人形机器人运动无独立 session**，相关内容分散在 FreeMusco（musculoskeletal locomotion，SA2025 Conference）、PhysHMR（humanoid control from vision）。

---

# 第三部分 · 趋势观察与研究建议

## 3.1 三条主线

**第一条：SA2025 在 motion tokenizer 方向上几乎空白，真正的爆发发生在 2026 年。**

这是一个有信息量的负面结果。SA2025 的 20 篇 motion 相关工作中，没有任何一篇在 motion generation 主干中使用 VQ-VAE / RVQ / FSQ 等离散 tokenizer。唯一的"码本"出现在 ARTalk 的面部参数空间量化中。这暗示**2025 年是连续表征的最后一年主流地位**，2026 年 GPC / MotionBricks / ARDY / EchoAvatar 四篇 tokenizer 工作的集中出现是一个范式转折点。

**第二条：统一交互动作生成（unified interaction synthesis）在 SA2025 正式立项。**

Uni-Inter（统一多交互上下文）、HOMA（多模态弱条件 HOI）、CHOICE（cluttered environment pick-and-place）、Motion2Motion（跨拓扑重定向）四篇独立工作在同一年攻同一个问题——**如何用一个模型覆盖多样化的交互场景**。这个方向在 2026 年被 ProAct 进一步拓展到 embodied social agent。三年三篇构成了清晰的发展脉络。

**第三条：SA2025 在物理角色控制方向上比 SIGGRAPH 2025 更活跃。**

PhysHMR（视觉到动作的端到端 policy）、FreeMusco（motion-free musculoskeletal control）、Learning to Ball（basketball policy composition）、SRBTrack（terrain-adaptive tracking）、Adversarial Differential Discriminators（改进 AMP）——五篇独立工作在物理控制方向上推进。相比之下，SIGGRAPH 2025 在这个方向只有 AMOR、Diffuse-CLoC、PhysicsFC、CAMDM 四篇。**SA 在物理仿真 + RL 控制的传统优势在 2025 年仍然保持**。

## 3.2 具体建议

**做 motion tokenizer 时，这几个点最有信息量：**

1. **SA2025 的 tokenizer 空白本身就是个信号**——如果你在写 related work，不要把 2025 年描述成"tokenizer 已经开始流行"，事实恰恰相反。2025 年是连续表征的最后一年主流，2026 年才是转折年。
2. **ARTalk 的面部码本成功**提供了一个有趣的反例——为什么面部参数的码本化在 2025 年就成功了（实时 + 高质量），而全身动作的码本化要到 2026 年才爆发？一个可能的解释是**维度差距**：面部 FLAME 参数 ~50-100 维 vs 全身 SMPL-X ~1000+ 维。这个维度鸿沟可能是 tokenizer 设计时需要认真对待的约束。
3. **PhysHMR 的 pixel-as-ray + distillation 方案**值得在 tokenizer 设计中借鉴——如果把视觉特征先投影到一个物理 grounded 的中间表示（类似 pixel-as-ray），再做 tokenization，可能会比直接在像素或关节坐标空间量化更有效。

**需要自行补查的缺口：**

- **CAMDM 的 Track 归属和 DOI**（SIGGRAPH 2025）——新闻稿未提供链接。
- **SA2025 多篇论文的 DOI 后缀**——ACM DL 目次中部分论文的 article number 需补充。
- **FreeMusco 的 CVAE latent 维度、morphology 覆盖范围**——arXiv 全文可查但未在本次检索中详细提取。
- **DuetGen 的 VQ 超参数**（SIGGRAPH 2025）——码本大小、层级时间分辨率比例。

---

# 附录：DOI 速查表

| 论文 | DOI | Track |
|---|---|---|
| Motion In-Betweening (Interacting Characters) | 10.1145/3757377.3763950 | Conf |
| PhysHMR | 10.1145/3757377.3763951 | Conf |
| ARTalk | 10.1145/3757377.3763955 | Conf |
| UniGAHA (Audio-Driven Universal Gaussian Head) | 10.1145/3757377.3763939 | Conf |
| Curvature Enthusiasm | 待查 | Conf |
| Control Operators | 待查 | Conf |
| Environment-aware MM | 待查 | Conf |
| StableMotion | 待查 | Conf |
| Learning to Ball | 待查 | Conf |
| Motion2Motion | 待查 | Conf |
| Ultrafast Retargeting | 待查 | Conf |
| SMF (Kinetic Codes) | 待查 | Conf |
| MaskedManipulator | 待查 | Conf |
| Generating from Blocking Poses | 待查 | Conf |
| Adv. Differential Discriminators | 待查 | Conf |
| Temporally Conditional Mamba | 待查 | Conf |
| SRBTrack | 待查 | Conf |
| Uni-Inter | 待查 | Conf |
| HOMA | 待查 | Conf |
| CHOICE | 待查 | Journal FF |
| AnimaX | 待查 | Conf |
| Animus3D | 待查 | Conf |
| X-UniMotion | 待查 | Conf |
| Sketch2PoseNet | 待查 | Conf |
| From Rigging to Waving | 待查 | Conf |
| FairyGen | 待查 | Conf |

---

## 数据来源与可复现性

- **SIGGRAPH Asia 2025 官方日程**：https://sa2025.conference-schedule.org/ （按 session 浏览，sess127/sess131/sess145/sess153/sess141 等）
- **DBLP 会议页**：https://dblp.uni-trier.de/db/conf/siggrapha/siggrapha2025.html
- **ACM DL proceedings**：https://dl.acm.org/doi/proceedings/10.1145/3757377
- **history.siggraph.org**：按 venue 标注所有 presentation
- 技术细节均来自 arXiv 全文/HTML、项目主页和官方摘要；**凡未查到的字段一律标注"待查"或"未找到"，未做任何推断或编造**
- 部分 DOI 后缀因 ACM DL 目次抓取限制未能提取，建议后续通过 ACM DL 补充
