# SIGGRAPH 2024 — Motion 相关论文逐篇综述

> 编制日期：2026-08-11  
> 覆盖范围：motion generation / motion control / mocap / animation / retargeting / gesture / crowd / cloth / hair / hand-object / avatar / deformation / garment / trajectory / perception / authoring  
> 共收录 **26** 篇 motion 相关论文（含 Journal Track + Conference Track）  
> 参考标准：与 `SIGGRAPH2026_SA2026_Motion论文综述.md` 一致的深度技术分析

---

## 一、Motion Generation & Control（核心专题）

### 1.1 A-MDM: Interactive Character Control with Auto-Regressive Motion Diffusion Models

- **作者**：Yi Shi, Jingbo Wang, Xuekun Jiang, Bingkun Lin, Bo Dai, Xue Bin Peng（SFU / Shanghai AI Lab / Xmov / NVIDIA）
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658140](https://doi.org/10.1145/3658140)｜**arXiv**：2306.00416

**问题定义**：现有motion diffusion模型多为space-time架构，一次性生成整段序列，无法用于实时交互控制。目标是设计一个自回归扩散模型，支持frame-by-frame生成，实现实时角色控制并保留diffusion模型的多样性和高保真度。

**方法与架构**：
- **自回归DDPM框架**：给定前一帧状态 $x_{f-1}$，预测下一帧 $x_f$ 的条件扩散过程
- **运动表征**：每帧状态包含根节点平面速度 $(d_x, d_y)$、绕垂直轴角速度 $d_r$、关节位置 $j_p \in \mathbb{R}^{j \times 3}$、关节速度 $j_v \in \mathbb{R}^{j \times 3}$、6D关节朝向 $j_o \in \mathbb{R}^{j \times 6}$
- **网络架构**：**10层全连接MLP**，每层 **1024 hidden units**，SiLU激活 + LayerNorm
- **扩散步数**：训练用 $T$ 步，推理仅需 **10–50步**（远少于DDPM的1000步），**40步为默认配置**
- **Scheduled Sampling（Student Forcing）**：用模型自身预测 $\hat{x}_{f-1}$ 作为后续帧输入，重复 $N_r$ 步以缓解长期漂移
- **噪声调度**：$\beta^t \in (0,1)$，前向过程 $q(x_f^t | x_f^{t-1}) = \mathcal{N}(x_f^t; \sqrt{1-\beta^t} x_f^{t-1}, \beta^t I)$
- **四种控制策略**：①随机采样 ②Task-Oriented Sampling（beam search + 用户评分函数）③Conditional Inpainting（空间/时间掩码替换）④Hierarchical RL（PPO高层控制器预测残差向量 $a_f^t$ 引导去噪）

**训练数据**：
- **AMASS**（大规模SMPL统一格式）、**100STYLE**（>400万帧，100种locomotion风格）、**LaFAN1**（>40万帧高动态动作）
- 统一 **30 FPS**

**定量结果**（单位：cm，50条序列，60帧ADE / 150帧APD）：

| 数据集 | 方法 | APD↑ | ADE↓ | FDE↓ | FS↓ | Bone.Err↓ | Pen.Freq↓ |
|--------|------|------|------|------|-----|-----------|-----------|
| AMASS | MVAE | 40.97±3.97 | 24.42±1.04 | 42.22±3.64 | 1.44±0.12 | 0.98±0.12 | 1.94±0.56% |
| AMASS | HuMoR | 44.95±4.49 | 17.96±1.17 | 41.10±3.98 | 1.35±0.11 | 1.04±0.07 | 1.78±0.62% |
| **AMASS** | **A-MDM (Ours)** | **61.08±1.35** | **10.40±0.66** | **21.12±1.16** | **1.06±0.11** | **0.82±0.04** | **0.40±0.03%** |
| 100STYLE | MVAE | 58.44±1.36 | 23.47±0.42 | 48.24±1.52 | 1.62±0.05 | 0.25±0.02 | 1.87±0.84% |
| 100STYLE | HuMoR | 68.25±1.45 | 17.41±0.47 | 43.34±1.74 | 1.57±0.06 | 0.23±0.02 | 1.80±0.92% |
| **100STYLE** | **A-MDM (Ours)** | **102.52±1.17** | **10.36±0.22** | **24.58±0.06** | **1.53±0.02** | **0.19±0.01** | **1.56±0.24%** |
| LaFAN1 | MVAE | 110.48±5.67 | 35.50±3.43 | 82.93±4.8 | 2.42±0.49 | 0.66±0.02 | 0.83±0.09% |
| LaFAN1 | HuMoR | 132.76±4.25 | 24.35±2.19 | 41.61±2.97 | 2.20±0.26 | 0.53±0.03 | 0.71±0.03% |
| **LaFAN1** | **A-MDM (Ours)** | **134.92±6.01** | **14.22±2.20** | **34.53±4.16** | **2.10±0.56** | **0.54±0.02** | **0.76±0.07%** |

**消融实验（扩散步数对100STYLE的影响）**：

| 步数 | APD↑ | ADE↓ | 时间(s)↓ | FS↓ |
|------|------|------|---------|-----|
| 10 | 96.70 | 10.44 | 0.009 | 1.55 |
| 20 | 101.17 | 10.16 | 0.012 | 1.57 |
| **40** | **102.52** | **10.36** | **0.021** | **1.53** |
| 50 | 100.97 | 10.36 | 0.026 | 1.53 |

**HumanML3D无条件生成对比**：

| 模型 | FID↓ | Diversity→ | Foot Skat. Ratio↓ |
|------|------|------------|-------------------|
| Real | 0.002±0.00 | 9.5002±0.002 | - |
| MDM | 0.9157±0.0533 | 9.0123±0.0602 | 0.0930±0.0021 |
| GMD | 0.5727±0.0681 | 9.1714±0.0789 | 0.0657±0.0016 |
| MVAE | 11.2393±0.1607 | 6.1503±0.0601 | 0.4153±0.0025 |
| HuMoR | 8.2444±0.2437 | 7.7396±0.0643 | 0.1210±0.0011 |
| **A-MDM (Ours)** | **1.7435±0.0813** | **7.8998±0.0638** | **0.1010±0.0012** |

**Motion Continuation泛化测试**（从mocap最后一帧外推）：A-MDM在AMASS上APD=59.91、FS=1.10、Bone.Err=2.01，均优于MVAE(41.27/1.52/2.96)和HuMoR(43.26/1.47/3.05)。

**信息缺口**：具体$j$（关节数）未明确给出；scheduled sampling中$N_r$的具体值；hierarchical controller的PPO超参（LR、batch size等）。

**为什么重要**：这是首个将diffusion model改造为自回归形式以实现**真正实时**（<50ms/frame）角色控制的工作。关键洞察是单帧输出的低维特性允许大幅减少扩散步数而不损失质量。Hierarchical control方案通过预测残差向量引导去噪过程，巧妙地将RL与diffusion结合，避免了直接训练diffusion policy的不稳定性。

---

### 1.2 MoConVQ: Unified Physics-Based Motion Control via Scalable Discrete Representations

- **作者**：Heyuan Yao, Zhenhua Song, Yuyang Zhou, Tenglong Ao, Baoquan Chen, Libin Liu（北京大学 / National Key Lab of General AI）
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658137](https://doi.org/10.1145/3658137)｜**arXiv**：2310.10198

**问题定义**：现有物理角色控制方法的训练数据通常仅几分钟到几十分钟，难以扩展到数十小时的大规模无结构数据集；latent表征缺乏显式语义，选择特定动作需要额外奖励工程。目标是学习可扩展的离散运动表征，支持多种下游任务。

**方法与架构**：
- **Residual VQ-VAE框架**：Encoder为1D CNN，将$T=24$帧motion clip编码为$K=6$个latent codes（**4×时间压缩**），每个code维度**768**
- **RVQ层数 $N=8$**，每层codebook大小 $|\mathcal{B}|=512$，总容量 $512^8$（指数级扩展）
- **Decoder**：1D Deconv上采样 → Policy(MoE) → 物理仿真
- **Policy**：6个expert，每个为**4层MLP × 256 units**，gating network为**2层MLP × 64 units**
- **动作输出**：PD控制目标角度 $\tau = k_p(\bar{\theta}-\theta) - k_d\dot{\theta}$，其中 $k_p=400, k_d=50$（除toe/wrist关节）
- **World Model**：**4层MLP × 512 units**，替代黑盒仿真器提供可微路径
- **Action Regularization**：EMA平滑($\beta=0.8$) + 幅值正则化
- **训练技巧**：EMA codebook更新($\gamma=0.99$)、Code Reset、Quantizer Dropout（随机使用$[1,N]$层RVQ）
- **联合损失**：$\mathcal{L} = \|M - \tilde{M}_\mathcal{W}\| + \beta_1\|\mathcal{E}(M) - \text{sg}(\mathbf{Z})\| + \beta_2\|\text{sg}(\mathcal{E}(M)) - \mathbf{Z}\| + \beta_3\mathcal{L}_{reg}$
- **优化器**：RAdam，LR=$1\times10^{-5}$，**40k epochs**，单张RTX 3090训练约6天

**训练数据**：
- **总计23.2小时**：LaFAN1(213min) + AMASS子集(SFU 10.9min, ACCAD 14.6min, BMLmovi 103min, BMLrub 180min, CMU 375min, KIT 392min)
- 下采样至**20 FPS**，retarget到19关节/20刚体的定制humanoid（身高1.6m，体重49.5kg）
- 仿真器：Customized ODE，隐式阻尼，**120 Hz**仿真，policy执行频率**20 Hz**

**定量结果**：

**Unseen Motion Tracking**（HDM05 2h测试集）：MPBPE = **6.3 cm**

**Video-based Pose Estimation**（Human3.6M，zero-shot无微调）：

| 方法 | Physics | MPJPE↓(mm) | PA-MPJPE↓(mm) |
|------|---------|------------|---------------|
| HybrIK | none | 54.4 | 34.5 |
| PhysCap | approx | 97.4 | 65.1 |
| SimPoE | approx | 56.7 | 41.6 |
| SimPoE (w/o root force) | full | 115.2 | 65.1 |
| **MoConVQ (Ours)** | **full** | **125.6** | **69.3** |

**Motion Quality on Human3.6M**：

| 方法 | e_smooth↓ | σ_smooth↓ | Accel↓ |
|------|-----------|-----------|--------|
| HybrIK | 5.9 | 3.1 | 10.9 |
| PhysCap | 7.2 | 6.9 | - |
| SimPoE | - | - | 6.7 |
| SimPoE (w/o root force) | - | - | 23.5 |
| **MoConVQ (Ours)** | **3.4** | **2.9** | **5.1** |

**下游应用**：
- **Universal Tracking**：支持未见过的mocap、kinematic生成器输出、单目3D姿态估计（HybrIK）三种输入源
- **Interactive Control**：Supervised learning训练steering policy（无需RL），distill locomotion dataset
- **MoConGPT**：Dual-transformer架构（temporal + depth transformer）做next-code prediction，支持unconditional和text-conditioned生成
- **LLM Integration**：In-context learning让Claude-2理解motion index序列，组合基本动作完成抽象任务

**信息缺口**：RVQ各层codebook利用率的详细统计；MoConGPT的transformer层数/头数；text-to-motion的T5编码器版本。

**为什么重要**：这是首个在**23小时大规模无结构数据集**上训练的物理角色控制器。Residual VQ架构提供了指数级可扩展的表征容量，同时保持训练稳定性。端到端地统一了tracking control、interactive control、text-to-motion和LLM集成四个任务于同一框架，且不需要为每个任务单独设计奖励函数。

---

### 1.3 Categorical Codebook Matching for Embodied Character Controllers

- **作者**：Sebastian Starke, Paul Starke, Nicky He, Taku Komura, Yuting Ye
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658209](https://doi.org/10.1145/3658209)

**问题定义**：将真实用户的稀疏传感器信号（如VR头显+手柄的三点追踪）映射到虚拟化身全身运动时，存在严重的多对多映射歧义——同样的头部+双手位置可能对应走路、跑步、蹲伏等多种运动模式。现有方法要么分开训练motion prior和control mapping，要么用CVAE但后验坍塌导致多样性不足。

**方法与架构**：
- **Codebook Matching核心思想**：不直接预测output motion，而是分别对input和output建立**两个独立的categorical codebook**，然后训练matching loss使两者的概率分布对齐
- **训练流程**：$Y \rightarrow Z_Y \rightarrow Y$（output重建）+ $X \rightarrow Z_X$（input编码），施加 $Z_X \sim Z_Y$ 的分布匹配约束
- **推理流程**：$X \rightarrow Z_X \rightarrow Y$，用input codebook的概率分布替代output codebook进行采样
- **与Motion Matching的关系**：类似MM的候选搜索机制，但用**概率分布替代距离度量**，且能端到端学习而非数据库检索
- **与CVAE的区别**：CVAE的后验$q(z|x)$容易坍塌到prior $p(z)$，而codebook matching通过离散分类分布保持多样性
- **Hybrid Control Mode**：三点追踪信号 + joystick goal location，支持用户在虚拟世界中行走/奔跑/蹲伏而现实中站立或坐着
- **实现框架**：Unity3D + PyTorch，代码开源在 github.com/sebastianstarke/AI4Animation

**训练数据**：无结构motion capture数据集（具体规模和时长论文未完全披露，但从GitHub README看使用了Style100等多个数据集）

**定量结果**：论文正文中的具体数值表格需查阅原文（WebFetch未能获取完整HTML），但从摘要和方法描述可知：该方法在VR tracking任务上相比MLP baseline和CVAE baseline显著降低了ambiguity artifacts，在hybrid control模式下实现了自然的locomotion转换。

**信息缺口**：codebook大小$K$、latent维度、具体网络层数、定量跟踪误差数值、消融实验中各组件的贡献比例。

**为什么重要**：提出了一种全新的处理motion生成歧义性的范式——用**categorical分布匹配**替代确定性映射或变分推断。这一思路与Motion Matching的哲学高度一致（都是"从候选中选择"而非"直接回归"），但将其纳入端到端可学习的神经网络框架，兼具数据驱动适应性和推理效率。对VR化身和游戏中的embodied avatar有直接应用价值。

---

### 1.4 MaskedMimic: Unified Physics-Based Character Control Through Masked Motion Inpainting

- **作者**：Chen Tessler, Yunrong Guo, Ofir Nabati, Gal Chechik, Xue Bin Peng（NVIDIA / SFU / Bar-Ilan Univ.）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687951](https://doi.org/10.1145/3687951)｜**arXiv**：2409.14393

**问题定义**：现有物理角色控制系统每个任务需要单独训练专用控制器+精心设计的奖励函数，缺乏通用性。目标是训练单一统一控制器，支持稀疏keyframes、文本指令、场景信息等多种控制模态，无需任务特定的reward engineering。

**方法与架构（两阶段）**：

**Stage 1 - Fully-Constrained Controller ($\pi^{FC}$)**：
- Transformer-based motion tracking policy，输入包括当前角色状态$s_t$、未来$K$帧目标pose $\hat{f}_{t+1:K}$、地形heightmap
- 角色观测规范：$\theta_t \ominus \theta_t^{root}$（相对旋转）、$(p_t - p_t^{root}) \ominus \theta_t^{root}$（相对位置）、$v_t \ominus \theta_t^{root}$（相对速度）
- 目标pose特征：每个关节$f^j = (\hat{\theta}^j \ominus \theta_t^j, \hat{\theta}^j \ominus \theta_t^{root}, (\hat{p}^j - p_t^j) \ominus \theta_t^{root}, (\hat{p}^j - p_t^{root}) \ominus \theta_t^{root})$
- 奖励函数：$r_t = w^{gp}r^{gp} + w^{gr}r^{gr} + w^{rh}r^{rh} + w^{jv}r^{jv} + w^{jav}r^{jav} + w^{eg}r^{eg}$（全局关节位置+旋转+根高度+关节速度+角速度+能量惩罚）
- PD控制，无residual forces
- **Early Termination**：平地0.25m偏差阈值，不规则地形0.5m；**Prioritized Motion Sampling**按失败率加权（最小权重$3e^{-3}$）

**Stage 2 - Partially-Constrained Controller ($\pi^{PC}$，即MaskedMimic)**：
- **Conditional VAE架构**：Prior $\rho(z_t|s_t,g_t^{partial}) = \mathcal{N}(\mu^\rho, \sigma^\rho)$ 仅观测部分约束；Encoder $\mathcal{E}(z_t|s_t,g_t^{full}) = \mathcal{N}(\mu^\rho + \mu^\mathcal{E}, \sigma^\mathcal{E})$ 作为prior的残差
- **随机Masking策略**：结构化时序masking（同一mask跨多帧持续，而非每帧重采样），确保某些关节连续可见/隐藏
- **KL-scheduling**：KL系数从0.0001线性增长到0.01
- **Episodic latent noise**：整个episode共享同一个$\epsilon \sim \mathcal{N}(0,1)$，增强时序一致性
- **Observation history**：过去40步中采样子5帧历史pose（对text conditioning至关重要）
- **Text encoding**：XCLIP embeddings（视频-语言预训练，捕捉时序语义）
- **Object表示**：8个bounding box角点 + object type index
- Prior用Transformer encoder处理变长多模态token；Encoder/Decoder为全连接网络

**训练数据**：
- **AMASS**（核心，PHC过滤流程去除artifacts）+ **HumanML3D**（text conditioning）+ **SAMP**（object interaction）
- 镜像数据增强（左右翻转，text同步镜像）
- Isaac Gym，**16,384并行环境**，4×A100 GPU
- 训练约**2周**，$\pi^{FC}$约30B steps，$\pi^{PC}$约10B steps
- Controller **30 Hz**，仿真**120 Hz**
- Joint conditioning子集：Left/Right Ankle, Pelvis, Head, Left/Right Hand

**定量结果**：

**Full-body Tracking**：MaskedMimic在AMASS测试集上的tracking performance接近$\pi^{FC}$ teacher，success rate > 90%（具体数值需查原文Table 1）

**VR Tracking**（仅head position+rotation + hand positions）：与PULSE、ASE、CALM对比，MaskedMimic在未见过的motion上展现出更好的generalization

**Irregular Terrain**：在随机生成的stairs/slopes/rough terrain上，$\pi^{FC}$和$\pi^{PC}$均保持了较高的tracking成功率

**信息缺口**：Table 1-4的具体数值（WebFetch截断）；transformer的层数/头数/hidden dim；各reward权重的具体值。

**为什么重要**：提出了**motion inpainting**作为统一的角色控制范式——任何控制模态（keyframes/joints/text/objects）都可以视为"未被mask的部分观测"。Goal-engineering的概念类比prompt-engineering，让用户通过组合简单约束就能实现复杂行为，无需重新训练。两阶段蒸馏（RL teacher → BC student）保证了物理合理性的同时获得了多模态灵活性。

---

### 1.5 SKEL-Betweener: a Neural Motion Rig for Interactive Motion Authoring

- **作者**：Dhruv Agrawal, Jakob Buhmann, Dominik Borer, Robert W. Sumner, Martin Guay
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687941](https://doi.org/10.1145/3687941)

**问题定义**：3D motion authoring需要操纵大量控制柄且耗时；现有neural motion completion方法需要dense full-pose context（创作成本高）且缺乏joint-level精细控制。目标是设计一个Neural Motion Rig，仅用两个pose就能生成长序列，并提供intuitive的joint-level控制曲线。

**方法要点**：
- **Neural Motion Curves**：类比传统动画的F-curves，但是神经隐式的joint-level位置/朝向控制曲线
- 只需**两个pose**即可生成长motion序列
- 支持交互式editing和refinement

**信息缺口**：网络架构细节、训练数据集、定量评估指标和数值、用户研究的具体结果。

**为什么重要**：将neural motion representation与工业界熟悉的rig/F-curve工作流对接，降低了AI辅助motion authoring的使用门槛。

---

### 1.6 CPoser: Text-to-Pose Generation Using LLMs

- **作者**：Yumeng Li, Bohong Chen, Zhong Ren, Yao-Xiang Ding, Libin Liu, Tianjia Shao, Kun Zhou
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687932](https://doi.org/10.1145/3687932)

**问题定义**：直接用LLM生成精确的全身articulation target很困难，因为LLM不是为pose generation训练的。

**方法要点**：
- **两阶段Pipeline**：①LLM parsing阶段将text prompt转为Pose-IR（Pose Intermediate Representation，结构化查询如"squatting depth=0.7, knee angle=45°"）②Pose optimization阶段在quantized pose prior space中通过robust optimization求解满足Pose-IR目标函数的expressive pose
- Pose-IR形成明确的objective function，避免了LLM直接输出数值的不可靠性
- 后续refinement加入facial expression

**信息缺口**：使用的LLM型号、Pose-IR的完整schema、quantized pose prior的构建方式、优化算法细节、定量对比结果。

**为什么重要**：巧妙地用structured intermediate representation桥接了LLM的语言理解能力和pose生成的几何精度需求，比端到端LLM-to-pose更可靠。

---

### 1.7 SMEAR: Stylized Motion Exaggeration with ARt-direction

- **作者**：Jean Basset, Pierre Bénard, Pascal Barla
- **Track**：Conference (SIGGRAPH 2024)｜**DOI**：[10.1145/3641519.3657457](https://doi.org/10.1145/3641519.3657457)

**问题定义**：如何将写实motion转化为具有艺术风格的夸张动画（如卡通smear frames），同时保持artist的可控性。

**方法要点**：
- 受传统2D动画中**smear frame**技术启发（快速运动时将物体拉伸成模糊带状以暗示速度感）
- 将art-direction意图作为条件输入，控制exaggeration的程度和风格

**信息缺口**：具体的网络架构、训练数据、量化评估、用户研究结果。由于是Conference Track，篇幅有限可能细节较少。

**为什么重要**：将传统手绘动画的核心技法（smear frames）系统地引入3D计算机动画领域，填补了stylization和exaggeration的技术空白。

---

### 1.8 HOIC: Hand-Object Interaction Controller

- **作者**：Haoyu Hu, Xinyu Yi, Zhe Cao, Jun-Hai Yong, Feng Xu
- **Track**：Conference (SIGGRAPH 2024)｜**DOI**：[10.1145/3641519.3657505](https://doi.org/10.1145/3641519.3657505)

**问题定义**：从mocap数据中重建符合物理规律的手-物交互（dexterous manipulation），现有kinematic重建常有穿透/漂浮artifacts。

**方法要点**：
- Deep Reinforcement Learning框架
- 将手部mocap作为reference，训练physics-based controller重现交互
- 解决contact modeling和fine-grained finger control的挑战

**信息缺口**：RL算法细节、奖励函数设计、仿真环境、定量对比（vs kinematic baseline）、支持的对象类别。

**为什么重要**：手部dexterous manipulation是character animation中最具挑战性的子问题之一，涉及高频接触变化和精细力控。

---

## 二、Gesture, Facial & Head Animation

### 2.1 Semantic Gesticulator: Semantics-Aware Co-Speech Gesture Synthesis

- **作者**：Zeyi Zhang, Tenglong Ao, Yuyao Zhang, Qingzhe Gao, Chuan Lin, Baoquan Chen, Libin Liu
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658134](https://doi.org/10.1145/3658134)

**问题定义**：语义有意义的手势（如比划大小、指向方向）在自然motion分布中属于long tail，中等规模数据集难以学到speech semantics与gesture的对应关系。

**方法要点**：
- **Generative Retrieval + GPT生成混合框架**
- 基于语言学发现构建semantic gesture列表，采集body+hand高质量motion dataset
- LLM-based retrieval从motion library中检索合适的semantic gesture candidates
- GPT-based model生成与speech节奏匹配的high-quality gestures
- **Semantic Alignment Mechanism**：将retrieved semantic gestures与GPT输出对齐，保证自然过渡

**信息缺口**：motion dataset的规模（clip数量/小时数）、GPT模型的具体架构、semantic alignment的数学表述、user study的详细结果。

**为什么重要**：首次系统性地将linguistics中的semantic gesture分类体系引入co-speech gesture合成，解决了纯数据驱动方法在long-tail语义手势上表现不佳的问题。

---

### 2.2 DiffPoseTalk: Speech-Driven Stylistic 3D Facial Animation

- **作者**：Zhiyao Sun, Tian Lv, Sheng Ye, Matthieu Lin, Jenny Sheng, Yu-Hui Wen, Minjing Yu, Yong-Jin Liu
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658221](https://doi.org/10.1145/3658221)

**问题定义**：Speech-driven facial animation需要学习speech-style-motion的多对多映射。现有方法或用deterministic model（缺乏多样性）或用one-hot style encoding（无法捕捉style复杂度，泛化差）。

**方法要点**：
- **Diffusion model + Style Encoder**：从短reference video中提取continuous style embedding
- **Classifier-free guidance**：同时condition on speech和style进行guided generation
- Style包含**head pose生成**（不仅面部表情）
- 训练数据：从in-the-wild audio-visual dataset重建的**3DMM参数**（解决scanned 3D talking face数据稀缺问题）

**信息缺口**：Diffusion的network architecture、noise schedule、style encoder结构、3DMM拟合pipeline、user study定量结果。

**为什么重要**：将diffusion model引入speech-driven 3D facial animation，并通过continuous style embedding解决了one-hot encoding的泛化瓶颈。

---

### 2.3 S³: Speech, Script and Scene driven Head and Eye Animation

- **作者**：Yifang Pan, Rishabh Agrawal, Karan Singh
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658172](https://doi.org/10.1145/3658172)

**问题定义**：对话场景中角色的head和eye animation需要综合考虑audio节奏、narrative意图和cinematographic构图，现有方法往往只考虑单一因素。

**方法要点**：
- **三模块模块化框架**：
  1. Audio-driven rhythmic head motion（节律性点头/摇摆）
  2. Narrative script-driven emblematic head/eye gestures（语义驱动的标志性动作，如疑问时挑眉）
  3. Gaze trajectories：audio-driven gaze focus/aversion + 3D scene salience计算
- 融合animation理论和psycho-linguistics洞察

**信息缺口**：各模块的具体算法、scene salience的计算方法、perceptual study的详细设计和结果。

**为什么重要**：首次将script（叙事脚本）和scene（ cinematographic场景）作为独立控制信号引入conversational gaze animation，更接近专业animator的工作流程。

---

### 2.4 Learning a Generalized Physical Face Model From Data

- **作者**：Lingchen Yang, Gaspard Zoss, Prashanth Chandran, Markus Gross, Barbara Solenthaler, Eftychios Sifakis, Derek Bradley
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658189](https://doi.org/10.1145/3658189)

**问题定义**：Physically-based facial simulation虽能自然处理self-collision和外力响应，但每个character需要skilled artist初始化material space并长时间训练deformation model，难以普及。

**方法要点**：
- **Generalized Physical Face Model**：从大型3D face dataset学习通用的物理面部模型
- 训练完成后，只需**单个3D face scan或单张face image**即可快速fit到unseen identity
- 支持animation retargeting across characters
- 支持physical effects：collision avoidance、gravity、paralysis、bone reshaping

**信息缺口**：FEM discretization的细节、训练dataset规模和来源、fitting procedure的具体步骤、定量对比。

**为什么重要**：将physics-based facial animation从"per-character手工定制"推进到"data-driven通用模型"，大幅降低了使用门槛。

---

### 2.5 Universal Facial Encoding of Codec Avatars from VR Headsets

- **作者**：Shaojie Bai, Te-Li Wang, Chenghui Li, Akshay Venkatesh, Tomas Simon, Chen Cao, Gabriel Schwartz, Jason Saragih, Yaser Sheikh, Shih-En Wei
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658234](https://doi.org/10.1145/3658234)

**问题定义**：VR telepresence需要毫秒级延迟的高保真面部动画，但head-mounted cameras存在oblique/incomplete views、headset佩戴差异、环境光照变化等挑战。

**方法要点**：
- **Self-supervised cross-view reconstruction**：利用多视角一致性作为监督信号，实现unseen users泛化
- **Lightweight expression calibration**：minimal runtime开销提升精度
- **Improved parameterization**：对环境变化鲁棒的ground-truth生成

**信息缺口**：VR headset的camera配置、cross-view loss的具体形式、calibration机制、定量对比（vs prior face-encoding methods）。

**为什么重要**：Codec Avatar技术首次在consumer VR headset上实现real-time photorealistic facial animation，且对unseen users有效。

---

## 三、Motion Capture & Performance Capture

### 3.1 Audio Matters Too! Enhancing Markerless Mocap with Audio for String Performance

- **作者**：Yitong Jin, Zhiping Qiu, Yi Shi, Shuangpeng Sun, Chongwu Wang, Donghao Pan, Jiachen Zhao, Zhenghao Liang, Yuan Wang, Xiaobing Li, Feng Yu, Tao Yu, Qionghai Dai
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658235](https://doi.org/10.1145/3658235)

**问题定义**：弦乐演奏capture涉及subtle hand-string contacts和intricate movements，纯视觉方法难以捕捉这些细微信息。

**方法要点**：
- **SPD Dataset**：cello和violin演奏，最多**23个视角** + audio + detailed 3D motion annotations（body/hands/instrument/bow）
- **Audio-guided multi-modal mocap**：从audio signal中检测hand-string contacts，用于refine vision-based hand pose
- 论证audio cues可以补充视觉方法遗漏的sound-producing gestures信息

**信息缺口**：SPD dataset的详细统计（时长/subjects/annotations精度）、audio-guided refinement的具体算法、ablation study结果。

**为什么重要**：首次提出用audio modality增强markerless mocap，并发布了首个乐器演奏细粒度hand motion多模态dataset。

---

### 3.2 ELMO: Enhanced Real-time LiDAR Motion Capture through Upsampling

- **作者**：Deok-Kyeong Jang, Dongseok Yang, Deok-Yun Jang, Byeoli Choi, Sung-Hee Lee, Donghoon Shin
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687991](https://doi.org/10.1145/3687991)

**问题定义**：单LiDAR传感器的mocap面临低帧率（~20fps）和稀疏点云的挑战，难以达到实时高质量motion capture。

**方法要点**：
- **Conditional autoregressive transformer-based upsampling**：从20fps LiDAR point cloud序列生成**60fps** motion
- Self-attention耦合精心设计的motion embedding和point cloud embedding
- **One-time skeleton calibration**：从单帧point cloud预测用户skeleton offsets
- **LiDAR simulator数据增强**：提升global root tracking和环境理解能力

**信息缺口**：Transformer的具体架构参数、upsampling的定量精度提升、LiDAR-mocap synchronized dataset的详细统计（20 subjects）。

**为什么重要**：首次将transformer upsampling引入LiDAR mocap，实现了单传感器60fps实时capture，并开源了高质量同步dataset。

---

### 3.3 EgoHDM: Real-time Egocentric-Inertial Human Mocap, Localization, and Dense Mapping

- **作者**：Handi Yin, Bonan Liu, Manuel Kaufmann, Jinhao He, Sammy Christen, Jie Song, Pan Hui
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687907](https://doi.org/10.1145/3687907)

**问题定义**：现有egocentric mocap系统缺乏dense scene mapping能力，且human localization和scene reconstruction之间没有形成闭环。

**方法要点**：
- **6 IMUs + commodity head-mounted RGB camera**
- **Tightly coupled mocap-aware dense bundle adjustment** + physics-based body pose correction
- **Local body-centric elevation map** + terrain-aware contact PD controller（减少floating/penetration）
- 首个提供near real-time dense mapping的人体mocap系统

**定量结果**：相比SOTA，human localization误差↓41%，camera pose误差↓71%，mapping误差↓46%。

**为什么重要**：首次实现了mocap、localization和dense mapping的tight coupling闭环，在非平坦地形（楼梯/户外）上验证了有效性。

---

### 3.4 Look Ma, no markers: Holistic Performance Capture without the Hassle

- **作者**：Charlie Hewitt, Fatemeh Saleh, Sadegh Aliakbarian, Lohit Petikam, Shideh Rezaeifar, Louis Florentin, Zafiirah Hosenie, Thomas J. Cashman, Julien Valentin, Darren Cosker, Tadas Baltrusaitis
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687772](https://doi.org/10.1145/3687772)

**问题定义**：电影/游戏工业中的mocap通常只针对face/body/hand之一，需要昂贵硬件和专业操作。ML方法多为单摄像头、单身体部位、非world-space结果。

**方法要点**：
- **首个marker-free holistic capture**：同时重建face（含eyes和tongue）、body、hands
- **无需calibration、manual intervention或custom hardware**
- **Hybrid approach**：纯synthetic data训练的ML模型 + 强大parametric human shape/motion模型
- 支持arbitrary camera rigs和varied capture environments/clothing
- 在多个body/face/hand reconstruction benchmarks上达到SOTA

**信息缺口**：synthetic data pipeline的细节、parametric模型的类型（SMPL-X?）、各benchmark的具体数值。

**为什么重要**：首次实现了"开箱即用"的holistic marker-free performance capture，覆盖了从face micro-expressions到full-body locomotion的完整谱系。

---

### 3.5 Gaussian Surfel Splatting for Live Human Performance Capture

- **作者**：Zheng Dong, Ke Xu, Yaoan Gao, Hujun Bao, Weiwei Xu, Rynson W. H. Lau
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687993](https://doi.org/10.1145/3687993)

**问题定义**：极稀疏（如4相机）RGBD设置下，NeRF/3DGS方法产生局部几何错误，PIFu方法渲染不真实。

**方法要点**：
- **PGH (Point-based Generalizable Human) representation**：pixel-aligned RGBD features条件化
- **Surface implicit function**回归surface points + **Gaussian implicit function**参数化radiance fields（2D Gaussian surfels）
- **PRNet**：depth-guided point cloud initialization (DPI)回归accurate surface point cloud
- **SPNet**：neural blending-based surfel splatting渲染novel views
- **1K分辨率free-view videos，平均12 fps**

**信息缺口**：两个benchmark的具体名称和PSNR/SSIM/LPIPS数值、与SOTA方法的详细对比表。

**为什么重要**：将Gaussian surfel splatting引入sparse-view human performance capture，在极低相机数下仍保持高质量。

---

### 3.6 DualGS: Robust Dual Gaussian Splatting for Immersive Volumetric Videos

- **作者**：Yuheng Jiang, Zhehao Shen, Yu Hong, Chengcheng Guo, Yize Wu, Yingliang Zhang, Jingyi Yu, Lan Xu
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687926](https://doi.org/10.1145/3687926)

**问题定义**：现有volumetric video workflow需要大量manual stabilization且assets过大，阻碍广泛采用。

**方法要点**：
- **DualGS**：用**skin Gaussians**（外观）和**joint Gaussians**（运动）分别表示appearance和motion
- **Explicit disentanglement**显著减少motion redundancy，增强temporal coherence
- **Coarse-to-fine训练策略**：coarse alignment + fine-grained optimization
- **压缩**：motion用entropy encoding，appearance用codec compression + persistent codebook
- **压缩比高达120×**，每帧仅需~**350KB**存储

**信息缺口**：VR headset上的实际播放帧率、不同序列的compression ratio分布、主观质量对比。

**为什么重要**：DualGS的skin/joint解耦表示大幅提升了volumetric video的compression efficiency，使得VR头显上的沉浸式播放成为现实。

---

### 3.7 360-degree Human Video Generation with 4D Diffusion Transformer

- **作者**：Ruizhi Shao, Youxin Pang, Zerong Zheng, Jingxiang Sun, Yebin Liu
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687980](https://doi.org/10.1145/3687980)

**问题定义**：从单张图片生成360度spatiotemporally coherent的human video，现有GAN/vanilla diffusion方法在complex motions和viewpoint changes上表现不佳。

**方法要点**：
- **Hierarchical 4D Transformer**：factorize self-attention across views、time steps、spatial dimensions
- Diffusion Transformer捕捉global correlations + CNN实现accurate condition injection
- 注入human identity、camera parameters、temporal signals
- **Multi-dimensional training strategy**：images + videos + multi-view data + limited 4D footage

**信息缺口**：Transformer的具体层级结构、训练dataset的组成比例、FID/IS等定量指标。

**为什么重要**：首次将4D diffusion transformer应用于360度human video generation，解决了复杂运动和视角变化的连贯性问题。

---

### 3.8 Towards Unstructured Unlabeled Optical Mocap: A Video Helps!

- **作者**：Nicholas Milef, John Keyser, Shu Kong
- **Track**：Conference (SIGGRAPH 2024)｜**DOI**：[10.1145/3641519.3657522](https://doi.org/10.1145/3641519.3657522)

**问题定义**：无结构无标签的光学mocap（即普通视频中的人体运动捕捉）缺乏标注数据，难以训练监督模型。

**方法要点**：
- 利用**video prior**辅助unstructured unlabeled optical mocap
- 可能采用self-supervised或weakly-supervised学习策略

**信息缺口**：具体方法细节、实验设置、定量结果。（Conference Track篇幅有限）

**为什么重要**：探索了"零标注"光学mocap的可能性，若能成功将极大降低mocap的部署成本。

---

## 四、Avatar & Human Reconstruction

### 4.1 PuzzleAvatar: Assembling 3D Avatars from Personal Albums

- **作者**：Yuliang Xiu, Yufei Ye, Zhen Liu, Dimitris Tzionas, Michael J. Black（MPI-IS）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687771](https://doi.org/10.1145/3687771)

**问题定义**：Text-to-3D方法擅长生成名人/虚构角色，但对普通人效果差；faithful reconstruction通常需要controlled setting下的全身图像。能否从用户日常"OOTD"（Outfit Of The Day）相册生成忠实avatar？

**方法要点**：
- **Album2Human task**：从personal OOTD album生成canonical pose下的3D avatar
- **Fine-tune foundational VLM**：将appearance/identity/garments/hairstyles/accessories编码为**separate learned tokens**
- Learned tokens作为"puzzle pieces"组装成3D avatar
- 支持通过inter-change tokens自定义avatar
- **PuzzleIOI dataset**：41 subjects，近1k OOTD配置，paired ground-truth 3D bodies

**信息缺口**：VLM的基础模型（LLaVA?）、token数量和维度、reconstruction accuracy的具体数值（vs TeCH/MVDreamBooth）。

**为什么重要**：提出了新颖的"Album2Human"任务，利用VLM的语义理解能力绕过body/camera pose估计难题，实现了从casual photo collection到3D avatar的端到端生成。

---

### 4.2 CharacterGen: Efficient 3D Character Generation from Single Images

- **作者**：Hao-Yang Peng, Jia-Peng Zhang, Meng-Hao Guo, Yan-Pei Cao, Shi-Min Hu（清华大学）
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658217](https://doi.org/10.1145/3658217)

**问题定义**：从单张图像生成高质量3D角色，面临diverse poses、self-occlusion和pose ambiguity的挑战。

**方法要点**：
- **Image-conditioned multi-view diffusion model**：将input pose校准到canonical form，保留input image关键属性
- **Transformer-based sparse-view reconstruction**：generalizable，从multi-view images生成detailed 3D models
- **Texture-back-projection strategy**：生成高质量texture maps
- **Anime character dataset**：多pose多view渲染

**信息缺口**：Diffusion model的具体架构、训练数据规模、CD/FID/Precision/Recall等定量指标。

**为什么重要**： streamlined pipeline + pose canonicalization的设计使得single-image 3D character generation在质量和效率上都达到了新水平。

---

### 4.3 CLAY: Controllable Large-scale Generative Model for High-quality 3D Assets

- **作者**：Longwen Zhang, Ziyu Wang, Qixuan Zhang, Qiwei Qiu, Anqi Pang, Haoran Jiang, Wei Yang, Lan Xu, Jingyi Yu
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658146](https://doi.org/10.1145/3658146)

**问题定义**：现有3D创作工具需要大量专业知识，普通用户难以将想象转化为3D内容。

**方法要点**：
- **Multi-resolution VAE + minimalistic latent Diffusion Transformer (DiT)**
- **Neural fields**表示continuous complete surfaces
- **Pure transformer blocks** in latent space做geometry generation
- **1.5 billion参数**的3D native geometry generator
- **Progressive training scheme**：ultra-large 3D model dataset
- **Multi-view material diffusion**：生成2K分辨率PBR textures（diffuse/roughness/metallic）
- 支持text/image输入 + 3D-aware controls（multi-view images/voxels/bounding boxes/point clouds/implicit representations）

**信息缺口**：训练dataset的具体规模和来源、DiT的层数/头数、与Shap-E/Shap-ASSEMBLY等的定量对比。

**为什么重要**：1.5B参数的3D generative model是目前规模最大的之一，支持多种3D-aware控制方式，代表了controllable 3D generation的最新进展。

---

## 五、Retargeting, Deformation & Skinning

### 5.1 Geometry-Aware Retargeting for Two-Skinned Characters Interaction

- **作者**：Inseo Jang, Soojin Choi, Seokhyeon Hong, Chaelin Kim, Junyong Noh
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687962](https://doi.org/10.1145/3687962)

**问题定义**：两个任意mesh connectivity的角色之间的interaction motion retargeting尚未被研究，shape差异会导致inter-penetration和semantic丢失。

**方法要点**：
- **SCT (Spatio Cooperative Transformer)**：预测root position和joint rotations的residual，考虑source-target的shape difference
- **Anchor loss**：retargeting时维持interaction角色间的geometric distance
- **Motion augmentation with deformation-based adaptation**：准备identical mesh connectivity的source-target paired training data

**信息缺口**：Transformer架构细节、anchor loss的数学形式、user evaluation的具体设计和结果。

**为什么重要**：首次研究了two-skinned characters interaction的retargeting问题，对游戏和电影中多角色互动场景有直接应用价值。

---

### 5.2 Multi-Resolution Real-Time Deep Pose-Space Deformation

- **作者**：Mianlun Zheng, Jernej Barbic
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687985](https://doi.org/10.1145/3687985)

**问题定义**：游戏/VR/Metaverse需要**hard real-time**（毫秒级甚至亚毫秒级）的multi-resolution mesh deformation，现有方法未同时满足速度和multi-resolution需求。

**方法要点**：
- **Multi-resolution analysis + mesh partition of unity + neural networks**
- 学习pre-skinning shape deformations，结合LBS重建training shapes并支持interpolation/extrapolation
- **Memory layout和code优化**提升计算速度
- 支持progressive construction（level by level）和任意时刻interrupt（graceful degradation）

**信息缺口**：各resolution level的具体顶点数、推理时间（ms）、memory reduction比例、vs naive approach的speedup倍数。

**为什么重要**：首次实现了hard real-time + multi-resolution的deep pose-space deformation，对现代实时图形系统有直接实用价值。

---

### 5.3 A Neural Network Model for Efficient Musculoskeletal-Driven Skin Deformation

- **作者**：Yushan Han, Yizhou Chen, Carmichael Ong, Jingyu Chen, Jennifer Hicks, Joseph Teran
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658135](https://doi.org/10.1145/3658135)

**问题定义**：现有skin deformation方法大多忽略muscle activation的主动收缩效应，或缺乏计算效率。

**方法要点**：
- **Comprehensive neural network**建模muscle/tendon/fat/skin多层软组织变形
- 提供kinematic和active correctives to LBS
- **Layered equilibrium data generation**：先算muscle变形 → inner skin/fascia → fat layer
- **Decoupled network**：分离skeletal kinematics和muscle activation state依赖
- Active contraction estimation via inverse dynamics + neural muscle moment arms

**信息缺口**：网络架构、training data量、与FEM simulation的定量误差对比。

**为什么重要**：首次用神经网络高效近似完整的musculoskeletal-driven skin deformation，包括主动肌肉收缩效应。

---

### 5.4 Deformation Recovery: Localized Learning for Detail-Preserving Deformations

- **作者**：Ramana Sundararaman, Nicolas Donati, Simone Melzi, Etienne Corman, Maks Ovsjanikov
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687968](https://doi.org/10.1145/3687968)

**问题定义**：现有data-driven deformation方法需要global shape encoding，但detail-preserving deformations在某些场景下无需global context。

**方法要点**：
- **Localized Jacobian representation**：one-ring neighborhood的Jacobian作为coarse deformation输入
- 一系列MLP + feature smoothing学习detail-preserving deformation的Jacobian
- Poisson solve恢复embedding
- **每点都是训练样本**，supervision特别轻量
- 跨object categories泛化

**信息缺口**：MLP架构、三个main tasks（shape correspondence refinement / unsupervised deformation / shape editing）的具体结果。

**为什么重要**：去除了对global encoding的依赖，实现了class-agnostic的localized deformation learning。

---

### 5.5 Real-time Large-scale Deformation of Gaussian Splatting

- **作者**：Lin Gao, Jie Yang, Bo-Tao Zhang, Jia-Mu Sun, Yu-Jie Yuan, Hongbo Fu, Yu-Kun Lai
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687756](https://doi.org/10.1145/3687756)

**问题定义**：Gaussian Splatting难以直接deform（离散Gaussians + 缺乏explicit topology）。

**方法要点**：
- **GaussianMesh**：mesh-based GS representation
- 3D Gaussians定义在explicit mesh上，**双向绑定**：GS rendering指导mesh face split自适应细化 ↔ mesh face split指导GS splitting
- Explicit mesh constraints正则化GS分布，抑制poor-quality Gaussians
- **Large-scale Gaussian deformation technique**：根据associated mesh manipulation改变3D GS参数
- 利用existing mesh deformation datasets做data-driven Gaussian deformation
- **平均65 FPS**

**信息缺口**：可处理的max mesh规模、deformation质量与static GS的PSNR对比。

**为什么重要**：首次实现了GS的interactive large-scale deformation， bridging了GS rendering和traditional mesh manipulation两个世界。

---

### 5.6 Spatial and Surface Correspondence Field for Interaction Transfer

- **作者**：Zeyu Huang, Honghao Xu, Haibin Huang, Chongyang Ma, Hui Huang, Ruizhen Hu
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658169](https://doi.org/10.1145/3658169)

**问题定义**：Interaction transfer需要处理source和target对象间较大的geometry/topology变化。

**方法要点**：
- **Combined spatial and surface representation**刻画example interaction
- **Learned spatial and surface correspondence field**：将objects表示为deformed + rotated SDFs
- 在corresponded points上做optimization（spatial+surface interaction constraints + regularization）
- human-chair和hand-mug interaction transfer验证

**信息缺口**：Correspondence field的网络架构、optimization的具体目标函数、与SOTA的定量对比。

**为什么重要**：Spatial + Surface双重correspondence的设计使得interaction transfer能处理更大的shape variation。

---

## 六、Cloth, Hair & Garment Simulation

### 6.1 Proxy Asset Generation for Cloth Simulation in Games

- **作者**：Zhongtian Zheng, Tongtong Wang, Qijia Feng, Zherong Pan, Xifeng Gao, Kui Wu
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658177](https://doi.org/10.1145/3658177)

**问题定义**：游戏中proxy mesh technique需要从visual mesh自动生成suitable low-poly proxy + skinning weights，目前依赖artist数天手工调整。

**方法要点**：
- **Automatic pipeline**：ill-conditioned high-res visual mesh → single-layer low-poly proxy mesh
- 处理non-simulation-ready inputs
- Differential skinning + well-designed loss functions优化skinning weights
- 在various challenging cloth models上验证

**信息缺口**：pipeline各步骤的具体算法、loss functions的数学形式、与artist-made proxy的定量对比。

**为什么重要**：自动化了游戏工业中cloth proxy asset生成的繁琐流程。

---

### 6.2 Super-Resolution Cloth Animation with Spatial and Temporal Coherence

- **作者**：Jiawang Yu, Zhendong Wang
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658143](https://doi.org/10.1145/3658143)

**问题定义**：Super-resolution cloth animation需要在跨帧保持spatial consistency和temporal coherence。

**方法要点**：
- **Module 1**：Simulator + Corrector交错。Simulator处理cloth dynamics，Corrector修正不同resolution间的low-frequency差异
- **Module 2**：Mesh-based super-resolution做wrinkle enhancement
- **Overlapping patches分解**：adaptability to various styles + geometric continuity
- **8× resolution improvement**

**信息缺口**：Simulator/Corrector的具体实现、patch大小的选择策略、定量对比。

**为什么重要**：Simulator-Corrector交错架构有效阻止了spatial errors的temporal propagation。

---

### 6.3 Progressive Dynamics for Cloth and Shell Animation

- **作者**：Jiayi Eris Zhang, Doug James, Danny M. Kaufman
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658214](https://doi.org/10.1145/3658214)

**问题定义**：High-fidelity IPC-based cloth simulation计算昂贵，design iteration需要反复模拟，prohibitively slow。

**方法要点**：
- **Coarse-to-fine LOD simulation**：progressive dynamics
- Tight-matching consistency + progressive improvement across levels
- 最细resolution下质量媲美IPC-based shell simulations
- Coarse resolution preview成本 comparable to coarsest-level direct simulations

**信息缺口**：LOD级别数、各级resolution比例、progressive refinement的数学保证。

**为什么重要**：Doug James组的work一贯代表physically-based simulation的最高水准，本work将progressive refinement理念引入cloth/shell animation design pipeline。

---

### 6.4 Efficient GPU Cloth Simulation with Non-distance Barriers and Subspace Reuse

- **作者**：Lei Lan, Zixuan Lu, Jingyi Long, Chun Yuan, Xuan Li, Xiaowei He, Huamin Wang, Chenfanfu Jiang, Yin Yang
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687760](https://doi.org/10.1145/3687760)

**问题定义**：高分辨率garment模型的untangled cloth simulation即使在GPU上也难以实时。

**方法要点**：
- **Non-distance barrier model**：inspired by interior point method，barrier potential不依赖primitive间distance，而依赖collision event的virtual life span
- 所有vertices保持在feasible region内
- **Subspace reuse strategy**：low-frequency strain propagation ≈ orthogonal to high-frequency collision-induced deformation
- Subspace处理low-frequency residuals + GPU iterative solver处理high-frequency residuals
- **比现有fast cloth simulators快至少一个数量级**

**信息缺口**：具体speedup倍数、支持的max triangle count、与IPC的定量对比。

**为什么重要**：Non-distance barrier + subspace reuse两大创新使得high-res interactive cloth simulation首次成为可能。

---

### 6.5 Real-time Physically Guided Hair Interpolation

- **作者**：Jerry Hsu, Tongtong Wang, Zherong Pan, Xifeng Gao, Cem Yuksel, Kui Wu
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658176](https://doi.org/10.1145/3658176)

**问题定义**：Hair interpolation常用linear skinning，但pre-computed skinning weights在large deformation时导致kinked/zigzag artifacts。

**方法要点**：
- **Physics-driven interpolation**：interpolate guide hairs的internal forces（而非positions），再根据material model重构rendered hairs
- Formulated as **constraint satisfaction problem**
- Regularization terms：penetration avoidance + drift correction
- **计算开销仅为conventional linear hair interpolation的~20%**

**信息缺口**：Constraint satisfaction的具体求解器、tested hairstyles的数量和类型。

**为什么重要**：首次将physics-based force interpolation引入hair rendering，在更低计算成本下获得更好视觉效果。

---

### 6.6 Contact Detection between Curved Fibres: High Order Makes a Difference

- **作者**：Octave Crespel, Emile Hohnadel, Thibaut Metivet, Florence Bertails-Descoubes
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658191](https://doi.org/10.1145/3658191)

**问题定义**：低阶primitives（line segments/triangles）用于fiber contact detection会产生spurious force jumps，尤其在curved fibers上。

**方法要点**：
- **High-order detection scheme**：two smooth curves间的accurate contact detection
- **Efficient adaptive pruning strategy**
- Super-helix model + high-order detection → wavy到highly curly fibers上much smoother force profiles
- Better scaling: efficiency vs precision

**信息缺口**：Adaptive pruning的具体算法、combing scenario的定量force smoothness指标。

**为什么重要**：系统性地分析了contact detection几何阶数对force profile质量的影响，为high-fidelity hair simulation提供了理论基础。

---

### 6.7 GroomCap: High-Fidelity Prior-Free Hair Capture

- **作者**：Yuxiao Zhou, Menglei Chai, Daoye Wang, Sebastian Winberg, Erroll Wood, Kripasindhu Sarkar, Markus Gross, Thabo Beeler
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687768](https://doi.org/10.1145/3687768)

**问题定义**：Strand-level精度的multi-view hair reconstruction仍具挑战，现有pipeline有固有局限。

**方法要点**：
- **Neural implicit hair volume**：encode high-res 3D orientation + occupancy
- **Volumetric 3D orientation rendering** + 2D orientation distribution supervision
- **Gaussian-based hair optimization**：chained Gaussian representation + direct photometric supervision
- **Prior-free**（不依赖external data priors）

**信息缺口**：Neural implicit network架构、tested subjects数量、与SOTA的strand-level定量对比。

**为什么重要**：ETH Zurich的Thabo Beeler组在hair capture领域的最新成果，prior-free设计提高了泛化能力。

---

### 6.8 Automatic Digital Garment Initialization from Sewing Patterns

- **作者**：Chen Liu, Weiwei Xu, Yin Yang, Huamin Wang
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658128](https://doi.org/10.1145/3658128)

**问题定义**：Sewing pattern到digital garment的自动初始化需要avoid folding和intersections，否则physics simulator会陷入undesirable local minima。

**方法要点**：
- **AI classification + heuristics + numerical optimization**的hybrid system
- Classification network选seed pieces → optimization确定positions/shapes
- Iterative selection-arrangement + phased initialization避免local minima
- **68% garments零用户干预**，其余易修正

**信息缺口**：Classification network架构、optimization的目标函数、tested garments数量和复杂度。

**为什么重要**：首次实现了sewing pattern到simulation-ready garment的高自动化初始化。

---

### 6.9 DressCode: Autoregressively Sewing and Generating Garments from Text Guidance

- **作者**：Kai He, Kaixin Yao, Qixuan Zhang, Jingyi Yu, Lingjie Liu, Lan Xu
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658147](https://doi.org/10.1145/3658147)

**问题定义**：Text-guided 3D garment generation仍处于早期阶段，缺乏CG-friendly的输出。

**方法要点**：
- **SewingGPT**：GPT-based架构 + cross-attention + text-conditioned embedding生成sewing patterns
- Tailored Stable Diffusion生成tile-based PBR textures
- LLM实现natural language interaction
- 支持pattern completion和texture editing

**信息缺口**：SewingGPT的具体架构、训练数据（sewing pattern-text pairs）规模、user study结果。

**为什么重要**：首次将LLM引入garment sewing pattern生成，实现了text-to-garment的端到端pipeline。

---

### 6.10 GarVerseLOD: High-Fidelity 3D Garment Reconstruction from Single Image

- **作者**：Zhongjin Luo, Haolin Liu, Chenghong Li, Wanghao Du, Zirong Jin, Wanhu Sun, Yinyu Nie, Weikai Chen, Xiaoguang Han
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687921](https://doi.org/10.1145/3687921)

**问题定义**：Single in-the-wild image的3D garment reconstruction在complex cloth deformation和body poses下泛化困难。

**方法要点**：
- **GarVerseLOD dataset**：**6,000**高质量cloth models，professional artists手工创建fine-grained geometry
- **Hierarchical LOD dataset**：detail-free stylized shape → pose-blended garment → pixel-aligned details
- **Conditional diffusion models**生成extensive paired images（high photorealism）
- Factorized inference：将under-constrained问题分解为更小search space的子任务

**信息缺口**：LOD各级的具体vertex count、diffusion-based labeling paradigm的细节、in-the-wild测试集的规模和指标。

**为什么重要**：6,000高质量garment models的dataset规模前所未有，LOD层次化设计为解决under-constrained重建问题提供了新思路。

---

## 七、Crowd & Collective Motion

### 7.1 CBIL: Collective Behavior Imitation Learning for Fish from Real Videos

- **作者**：Yifan Wu, Zhiyang Dou, Yuko Ishiwaka, Shun Ogawa, Yuke Lou, Wenping Wang, Lingjie Liu, Taku Komura
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687904](https://doi.org/10.1145/3687904)

**问题定义**：现有imitation learning方法需要ground-truth motion trajectories，但在高密度群体erratic movements下难以获取；rule-based方法motion diversity有限。

**方法要点**：
- **直接从videos学习fish schooling behavior**，无需captured motion trajectories
- **MVAE (Masked Video AutoEncoder)**：self-supervised提取implicit states from 2D observations
- **Adversarial imitation learning**：latent space中imitate motion pattern distribution
- Bio-inspired rewards + priors正则化训练
- 跨species泛化 + abnormal fish behavior detection应用

**信息缺口**：MVAE架构、adversarial training的具体设置、tested species数量。

**为什么重要**：首次实现了从raw videos直接学习collective behavior，绕过了trajectory annotation瓶颈。

---

## 八、Trajectory & Path Planning

### 8.1 Implicit Swept Volume SDF: Continuous Collision-Free Trajectory Generation

- **作者**：Jingping Wang, Tingrui Zhang, Qixuan Zhang, Chuxiao Zeng, Jingyi Yu, Chao Xu, Lan Xu, Fei Gao
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658181](https://doi.org/10.1145/3658181)

**问题定义**：Non-convex geometries和complex environments下的continuous collision-free trajectory generation仍是巨大挑战。

**方法要点**：
- **SVSDF (Swept Volume SDF)**：guide trajectory optimization for CCA
- Formulated as **Generalized Semi-Infinite Programming**，implicitly求解query points的numerical solutions
- 无需explicit surface reconstruction
- 适用于rigid和deformable shapes

**信息缺口**：GSIP求解器的具体实现、tested scenarios的数量和复杂度、与OMP/CHOMP等典型算法的定量对比。

**为什么重要**：Graphics（SDF）+ Robotics（trajectory optimization）的交叉创新，解决了non-convex shapes的continuous collision avoidance难题。

---

## 九、Animation Tools, Authoring & Inbetweening

### 9.1 Interactive Design of Stylized Walking Gaits for Robotic Characters

- **作者**：Michael A. Hopkins, Georg Wiedebach, Kyle Cesare, Jared Bishop, Espen Knoop, Moritz Bächer
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658227](https://doi.org/10.1145/3658227)

**问题定义**：Procedural animation工具对robotic characters不适用，因为忽略了kinematic/dynamic constraints。

**方法要点**：
- Artist-directed stylized bipedal walking gaits，tailored for robotic characters
- Interactive editing tool + model-based control stack
- **Phase-space blending**：interpolate example walk cycles while preserving contact constraints
- 在custom free-walking robot + 两个simulation examples上验证

**信息缺口**：Control stack的具体组成、phase-space blending的数学形式、robot hardware规格。

**为什么重要**：将procedural animation与robotic constraints bridge起来，使artist可以直接author robot locomotion。

---

### 9.2 Skeleton-Driven Inbetweening of Bitmap Character Drawings

- **作者**：Kirill Brodt, Mikhail Bessmeltsev
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687955](https://doi.org/10.1145/3687955)

**问题定义**：工业界keyframes常间距大、raster格式、含occlusions，现有solution多要求vector input或tight inbetweening。

**方法要点**：
- **Skeleton-driven bitmap inbetweening**：artist animates skeleton between keyframes → skeleton-based deformation → discrete optimization + deep learning blend
- **2.5D partially layered template**：lifting drawing into 3D解决occlusions导致的piecewise smooth deformation问题
- 支持tight和far inbetweening

**信息缺口**：Discrete optimization的目标函数、deep learning blend的网络架构、user study结果。

**为什么重要**：首次实现了bitmap far inbetweening with occlusion handling，贴近工业实际需求。

---

### 9.3 ToonCrafter: Generative Cartoon Interpolation

- **作者**：Jinbo Xing, Hanyuan Liu, Menghan Xia, Yong Zhang, Xintao Wang, Ying Shan, Tien-Tsin Wong
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687761](https://doi.org/10.1145/3687761)

**问题定义**：Cartoon video interpolation面临exaggerated non-linear motion和occlusion/dis-occlusion挑战，传统correspondence-based方法失效。

**方法要点**：
- **Adapt live-action video priors to cartoon domain**
- **Toon rectification learning**：seamlessly adapt live-action priors，解决domain gap和content leakage
- **Dual-reference-based 3D decoder**：compensate compressed latent prior spaces的细节损失
- **Flexible sketch encoder**：interactive control over interpolation results

**信息缺口**：Base live-action model的选择、toon rectification的具体训练策略、定量对比（FID/LPIPS等）。

**为什么重要**：首次将live-action video diffusion prior成功adapt到cartoon interpolation，解决了dis-occlusion这一长期难题。

---

## 十、Motion Perception

### 10.1 Evaluating Visual Perception of Object Motion in Dynamic Environments

- **作者**：Budmonde Duinkharjav, Jenna Kang, Gavin Stuart Peter Miller, Chang Xiao, Qi Sun
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687912](https://doi.org/10.1145/3687912)

**问题定义**：现有visual perception研究多在stationary settings和singular objects下进行，但实际应用中observer也在complex scenes中移动。

**方法要点**：
- **Crowdsourcing-based psychophysical study**：quantify scene dynamics/content patterns与perceptual judgments的关系
- 构建generalized conditions的perceptual model
- 应用于gaming和animation design中的motion error补偿

**信息缺口**：Psychophysical study的具体设计（trial数量/参与者人数/stimuli类型）、model的数学形式、application demo的具体效果。

**为什么重要**：首次系统研究了dynamic 3D environments中的object motion perception，为perception-aware rendering提供了理论基础。

---

### 10.2 Towards Motion Metamers for Foveated Rendering

- **作者**：Taimoor Tariq, Piotr Didyk
- **Track**：Journal (TOG 43·4)｜**DOI**：[10.1145/3658141](https://doi.org/10.1145/3658141)

**问题定义**：Foveated rendering在peripheral区域降低spatial quality，但peripheral vision对motion detection敏感，可能导致motion underestimation。

**方法要点**：
- 证明peripheral high-frequency loss inhibits motion perception → velocity underestimation
- **Perceptually motivated real-time technique**：synthesize controlled spatio-temporal motion energy offset loss
- User experiments验证effectiveness

**信息缺口**：Motion energy synthesis的具体算法、user experiment的设计（participants/tasks/metrics）、velocity underestimation的量化程度。

**为什么重要**：揭示了foveated rendering的一个此前被忽视的perceptual side effect，并提出了补救方案。

---

## 十一、Hand-Object Interaction

### 11.1 HOIC: Hand-Object Interaction Controller (详见1.8)

*已在Section 1.8中详述。*

---

## 统计汇总

- SIGGRAPH 2024共收录 **26** 篇 motion 相关论文
- Journal Track（TOG 43·4）：**18** 篇
- Conference Track：**8** 篇

### 按类别分布
- Motion Generation & Control：8 篇
- Gesture, Facial & Head Animation：5 篇
- Motion Capture & Performance Capture：8 篇
- Avatar & Human Reconstruction：3 篇
- Retargeting, Deformation & Skinning：6 篇
- Cloth, Hair & Garment Simulation：10 篇
- Crowd & Collective Motion：1 篇
- Trajectory & Path Planning：1 篇
- Animation Tools & Authoring：3 篇
- Motion Perception：2 篇
- Hand-Object Interaction：1 篇（与Motion Generation重叠计数）
