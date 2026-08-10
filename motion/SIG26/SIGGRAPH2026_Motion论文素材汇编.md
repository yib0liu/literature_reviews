# SIGGRAPH 2026 — Motion 相关 Technical Papers 素材汇编

> 采集日期：2026-08-10
> 数据来源：官方日程站（s2026.conference-schedule.org，含 session 页 / presentation 页的官方 Description 与作者机构）、paperdigest SIGGRAPH 2026 全量列表（共 327 篇）、arXiv 全文/HTML、各论文项目主页、DBLP、ACM DL。
> 覆盖范围：Journal Track（TOG45(4)）+ Conference Papers Track，全部与人体动作 / 角色动画 / 物理角色控制 / motion capture / motion editing-stylization-retargeting / 面部与手部动作 / 人形与四足机器人运动 / 4D 动态网格动画 / 视频驱动动作 相关论文。
> 关于"是否涉及 tokenizer"：明确区分 **离散量化（VQ-VAE / FSQ / RVQ / LFQ 等）** 与 **连续 latent / diffusion / flow matching**。
>凡公开来源未查到的字段，一律写"未找到"，不做推断。

---

## 0. 全局速览：本届涉及"离散运动/结构 token"的论文清单

| 论文 | Session | 量化方案 | 备注 |
|---|---|---|---|
| **GPC** | Motion Generation, Warping & Control | **FSQ**，L=9 levels，d=40维，分组 G=5 → 词表 59,049，token 序列长 8 | 端到端 RL 联合训练"运动词汇+控制策略"，GPT 式next-token |
| **MotionBricks** | Motion Generation, Warping & Control | **多头（multi-head）VQ-VAE**，K 个并行码本，每头 128–256 tokens，可替换为 FSQ；4× 时间下采样 | 非RVQ，沿特征维并行切分；30 FPS，T=12–64 帧片段 |
| **ARDY**（Autoregressive Diffusion w/ Hybrid Rep.） | Motion Generation, Warping & Control | **FSQ**，64 levels × 128 维（"FSQ 64-128"），patch = 4 帧/token；同时实现了连续 AE 与 VAE 变体做对比 | 混合表征：显式 root（5 维/帧）+ 离散量化 body latent（128 维） |
| **EchoAvatar** | Faces & Avatars | **RVQ**，Q=6 层残差，每层码本 K=512，latent d=512，时间下采样 4×（→7.5 Hz token），按上半身/下半身/手部分 3 套码本 | 因果注意力 tokenizer；Qwen2.5-0.5B 自回归 |
| **SimArt** | Generative 3D (2) | **稀疏 3D VQ-VAE**（针对部件几何与运动学，非人体动作） | 与人体 motion tokenizer 非同类，但属离散 token 建模，供参考 |
| （旁证，非SIGGRAPH 2026 主会）**SkinTokens / TokenRig** | — | **FSQ-CVAE** 压缩 skinning weights 为离散 token | 同一批作者（Tsinghua+VAST，与 TopoCap 同组），arXiv 2602.04805，可作离散化"结构表征"参照 |

明确**不用**离散 token（连续 latent / diffusion / flow matching / 优化）的 motion 论文：SMP、Deep Motion Warping、MultiAct、TopoCap（Graph CVAE + flow matching）、Stylized T2M（HyperLoRA）、MUSIC（VAE 连续 latent）、MOCHI（diffusion noise optimization）、ACT（DiT + 共享 latent）、R-DMesh（VAE + rectified flow DiT）、STyMo、Skinned Motion Retargeting、ReActor、AIS(In-Betweening)、Motion4Motion、AniGen（S³ Fields + flow matching）等。

---

# Session：Motion Generation, Warping & Control（sess104）

-时间/地点：2026-07-21（周二）14:00–15:30PDT，Room 403 A
- Session Chair：YutingYe（Reality Labs Research, Meta / Meta）
- Keywords：Animation / AI-ML / Robotics / Simulation

---

## 1. GPC: Large-Scale Generative Pretraining for Transferable Motor Control

- **Session**：Motion Generation, Warping & Control（14:00–14:10）
- **作者**：Yi Shi（Simon Fraser University；NVIDIA）、Yifeng Jiang（NVIDIA）、Chen Tessler（NVIDIA）、Xue Bin Peng（Simon Fraser University；NVIDIA）
- **Track**：SIGGRAPH 2026 Conference Papers
- **链接**：
  - arXiv：https://arxiv.org/abs/2606.29148 （HTML：https://arxiv.org/html/2606.29148，提交 2026-06-28）
  - DOI：10.1145/3799902.3811038
  - 依赖框架（开源）：ProtoMotions3 https://github.com/NVLabs/ProtoMotions/
  - 数据集：Bones 数据集 https://bones.studio/datasets
  - 项目主页/代码仓库：**未找到**（论文正文未给出 GPC 独立 project page 或 repo；NVIDIA 官方声明本届论文将陆续开源）
- **官方 Description**（会议站原文）："We present Generative Pretrained Controllers, a framework for training reusable generative controllers for physically simulated characters. GPC leverages FSQ discretization with end-to-end RL to learn reusable motor skills from large-scale motion data. Once trained, the generative controller can be finetuned and adapted to perform diverse downstream tasks using naturalistic behaviors."
- **核心问题**：物理角色动画中，如何得到既能覆盖广谱人类运动技能、又能便捷复用到下游任务的通用控制器。既有生成式控制器多用连续 latent（VAE/GAN），易mode collapse、latent 流形有空洞与漂移导致不自然行为；已有 tokenize 方法训练流程复杂。
- **方法与表征**：
  - **离散化：Finite Scalar Quantization（FSQ，look-up-free，无显式可学习码本）**
    - 每维量化级数 **L = 9**；潜维度 **d = 40**；隐式码本容量 **9^40**
    - 量化式：ẑ_t = ⌊⌊L/2⌋ · tanh(z_t)⌉，逐元素取整；梯度用 **STE（直通估计器）**
    - **token grouping**：把 G 个连续 token 打包为一个 grouped token 以缩短序列。最终取 **G = 5** → 词表 **L^G = 59,049**，序列长度 **N_token = 8**
    - 分组消融（L=9, d=40）：G=8（词表 4.3e7、5 tokens、参数 4.4e10、显存 1.7e2 GB，代价过高）；**G=5（59,049 词表 / 8 tokens / APD 0.34 / ADE 0.30 / Accel 3.67 / 参数 6.0e7 / 显存 0.27GB / 92.85 FPS，采用）**；G=4（6,561 / 10 tokens / APD 0.29 / ADE 0.29）；G=2（81 / 20 tokens）；G=1（9 / 40 tokens / 25.15 FPS）
  - **编码器**输入参考动作的未来目标状态序列 ŝ_{t:t+h}（**h 具体取值：未找到**）；**解码器**输出各关节 PD 控制器的目标关节旋转
  - **端到端 RL（PPO）**联合优化"运动词汇（FSQ）+ 控制策略"，仅用motion-tracking 奖励 r_t = w_p r_p + w_v r_v + w_r r_r + w_h r_h（沿用 DeepMimic 框架）
  - 码本学成后训练 **GPT 式因果Transformer decoder** 做 next-token prediction；teacher forcing + 交叉熵；推理用 **nucleus (top-p) 采样**
  - 下游适配：**CoLA（Conditional Low-rank Adaptation）** = DoRA（幅度/方向解耦低秩更新）+ FiLM任务条件调制，W_adapted = W_0 + m·V/‖V‖_c，冻结主干，**新增参数 < 1%**；rollout 时用无条件生成模型引导的 nucleus sampling 做探索正则
  - **Token 帧率 / 控制频率 / 每 token 覆盖帧数**：**未找到**（正文仅说明每个仿真时间步产生 d 个 token，具体 Hz 在附录，公开 HTML 中被截断）
  - Transformer 层数 / 头数 / hidden dim / context length：**未找到**（正文引用附录 C.1，公开内容截断）。已知所选配置生成控制器约 **6.0e7 ≈ 60M 参数**，显存 0.27 GB，推理 **92.85 FPS**
- **数据与角色**：
  - **Bones（BONES-SEED）**：**> 680 小时**（摘要表述"over 600 hours"），约**343,000 clips**，平均长度 200 帧、中位 160 帧，从日常动作到高动态杂技
  - **AMASS**：40 小时、约 11,300 clips（消融）
  - **LAFAN1**：4.6 小时、77 序列/ 15 类 / 5 名受试者
  - **Beyond（源自 PARC）**：10.3 分钟（18,600 帧）、14 clips，parkour 类
  - SFT 用：20 秒 crouch walk 单段
  - 仿真角色（Bones 定制 humanoid）：23 刚体、**66 可驱动 DoF**（非根关节均为 3-DoF）、站立身高 1.85 m、臂展 1.75 m；仿真器 **Isaac Gym**
- **关键结果**：
  - 动作复现成功率（判据：全帧平均关节位置误差 < 0.5 m）：
    - Bones[680h]：MLP（无量化瓶颈）**99.98% / MPJPE 25.56 mm**；VQ-VAE 99.94% / 37.92；**FSQ（本文）99.98% / 34.90**
    - AMASS[40h]：MLP 99.59% / 30.26；VQ-VAE 99.30% / 59.28；**FSQ 99.51% / 44.43**
  - 端到端 RL vs 冻结运动学编码器：FSQ-K（监督预训练后冻结）99.03% / MPJPE 78.26 / 码本利用率 76.34%；**FSQ（端到端 RL）99.98% / 34.90 / 82.15%**
  - 码本利用率对照：**VQ-VAE 仅 10–20%**（codebook collapse），FSQ 保持高利用率
  - CoLA 下游鲁棒性：中等推力扰动下存活率 **GPC+CoLA 68.1% vs CVAE 44.4%**
  - SFT vs 纯 RL 微调（512 episodes）：SFT+RL Return 143.36 / 扰动 Return 125.44 / Entropy 2.57 / APD 0.24 / ADE 0.15；w/o SFT Return 230.42 / 176.35 / 5.08 / 0.26 / 0.27（SFT 降低熵与多样性，换取风格一致性，头部高度稳定在 0.8–1.3 m 的下蹲区间）
  - 下游任务（Steering / Trajectory / Target reaching / Barrier / Platform 及需跳跃爬行的场景交互）：**仅定性结果（关键帧与轨迹图），未给出各任务成功率数值表**
  - 涌现行为（定性）：手臂受力触发侧手翻式恢复、腿部受力自动调步、脊椎强扰动后自然倒地并起身、跌倒后前滚翻起身；无条件采样可生成跳跃、腾跃、舞蹈、cartwheel / handstand / sideflip
  - 算力：Bones 上FSQ tracking + 生成控制器用 **24× A100**；其余实验单卡 A100；训练后可在 **RTX 4090** 上推理
- **是否涉及 tokenizer**：**是（本届最直接相关）**。FSQ look-up-free 量化，L=9、d=40、G=5、词表 59,049、序列长 8；与 VQ-VAE 的对比（MPJPE、码本利用率、无需EMA/死码重初始化）在正文 Table 1/2；**token 帧率与每token 对应帧数未找到**。
- **信息缺口**：token 帧率/控制频率、未来目标窗口 h、Transformer 结构超参、下游任务定量表、survival rate 独立指标（正文只报 tracking success rate / Return / Pert. Return）、附录 B.5（vs Diffusion policy, Table 7）与 D.5（vs 连续先验）具体数值。

---

## 2. SMP: Reusable Score-Matching Motion Priors for Physics-Based Character Control

- **Session**：Motion Generation, Warping & Control（14:10–14:20）
- **作者**：Yuxuan Mu*、Ziyu Zhang*、Yi Shi*、Dun Yang（以上 Simon Fraser University）、Minami Matsumoto（Sony Interactive Entertainment）、Kotaro Imamura（Sony Interactive Entertainment）、Guy Tevet（Stanford University）、Chuan Guo（Snap Inc.）、Michael Taylor（Sony Interactive Entertainment）、Chang Shu（National Research Council Canada）、Pengcheng Xi（National Research Council Canada）、Xue Bin Peng（Simon Fraser University；NVIDIA）
  - *Joint First Authors。注：arXiv v1 中 Chuan Guo 署Meta、Sony 署 "Sony Playstation"，最终版为 Snap Inc. / Sony Interactive Entertainment
- **Track**：Journal Track（ACM TOG）
- **链接**：
  - arXiv：https://arxiv.org/abs/2512.03028 （PDF：https://arxiv.org/pdf/2512.03028）
  - 项目主页：https://yxmu.foo/smp-page
  - 视频：https://youtu.be/jBA2tWk6vzU
  - DOI：10.1145/3811282
  - 代码开源：**未找到**（项目页仅提供 paper/video）
- **官方 Description**："SMP builds reusable and modular reward models for training motor controllers. Once constructed from a motion dataset, the priors can be reused across diverse tasks while preserving the behaviors in the data, without requiring access to the original dataset or retraining."
- **核心问题**：对抗式模仿学习（AMP 类）虽是学习 motion prior 的有效手段，但判别器几乎必须为每个新控制器重训，且下游任务仍需保留原始参考动作数据——可复用性差。
- **方法与表征**：
  - **连续表征 + 扩散先验**，不使用离散 token。
  - 核心：用**预训练 motion diffusion model + Score Distillation Sampling（SDS）** 构造与任务/策略无关的 **Score-Matching Motion Priors**。先验一次训练、之后**冻结**，作为通用奖励函数（stationary reward model）用于 RL 训练新策略。
  - **Ensemble Score Matching（ESM）**：在固定的一组代表性timestep（例：K = {22, 15, 8}）上对噪声残差取平均以降方差，为 PPO 提供平滑奖励信号。
  - **自适应归一化**：跟踪各噪声级 SDS 误差的运行均值 μ_i 用于缩放奖励，实现"即插即用"，免去切换先验时的手工调参。
  - **Generative State Initialization（GSI）**：用冻结扩散模型直接采样初始姿态，替代传统 RSI（Reference State Initialization），从而**完全脱离原始数据**。
  - **风格提示与合成**：用style-conditioned diffusion（在 100STYLE 上训练）+ classifier-free guidance 将奖励"提示"到指定风格；通过**空间掩码混合**两种风格提示的 ε 预测（如上半身"aeroplane"、下半身"HighKnees"），可合成数据集中不存在的复合风格。
- **数据与规模**：
  - 通用先验在**跨 100 种风格**的大规模数据集上训练；训练后可重定向为 **100 个风格专属先验**，无需再访问原始数据或更新模型
  - **数据效率**：仅用 **3 秒**的walk/jog/run 片段构造先验，策略即可在 **[1.2, 6.8] m/s** 连续目标速度范围内自动调整步频与步态，并涌现出数据中不存在的 walk→jog→sprint 过渡
- **关键结果**：
  - Skill Position Tracking Error [m]（越低越好）：
    | Skill | DM | AMP | Frozen(AMP-Frozen) | SMILING | **SMP (Ours)** |
    |---|---|---|---|---|---|
    | Walk | 0.010 | 0.010 | 0.044 | 0.042 | 0.030 |
    | Run | 0.013 | 0.088 | 0.129 | 0.115 | 0.067 |
    | Spinkick | 0.073 | 0.049 | 0.324 | 0.088 | 0.059 |
    | Cartwheel | 0.243 | 0.043 | 0.419 | 0.104 | 0.043 |
    | Backflip | 0.073 | 0.058 | 0.272 | 0.144 | 0.069 |
    | Crawl | 0.006 | 0.011 | 0.285 | 0.061 | 0.011 |
    | **Average** | 0.070 | **0.046** | 0.246 | 0.092 | **0.043** |
    （表列名与数值依 arXiv-vanity 版；AMP-Frozen 严重退化说明"直接冻结判别器"不可行）
  - 任务覆盖：dodgeball（侧身/下蹲躲避投掷物且保持自然步态）、target location（多风格导航）、stair traversal（人-场景交互先验，自然落足）、人-物交互（先验联合建模角色与物体运动）
  - **真机**：SMP 训练的策略直接部署到 **Unitree G1** 人形机器人；asymmetric actor-critic + domain randomization + 仅本体感知观测；表现出自然步态、外扰恢复、敏捷技能，无需运行时运动规划器或手工启发式
- **是否涉及 tokenizer**：**否**。基于连续扩散模型的 score 蒸馏，无量化/码本。

---

## 3. MotionBricks: Scalable Real-Time Motions with Modular Latent Generative Model and Smart Primitives

- **Session**：Motion Generation, Warping & Control（14:20–14:30）
- **作者**（均含 NVIDIA）：Tingwu Wang†、Olivier Dionne†、Michael De Ruyter、David Minor、Davis Rempe、Kaifeng Zhao（NVIDIA；ETH Zürich）、Mathis Petrovich、Ye Yuan、Chenran Li、Zhengyi Luo、Brian Robison、Xavier Blackwell、Bernardo Antoniazzi、Xue Bin Peng（NVIDIA；Simon Fraser University；官网页面误拼为 "Xue Bing Peng"）、Yuke Zhu（NVIDIA；The University of Texas at Austin）、Simon Yuen（NVIDIA）
  - †Joint first authors；✉ Tingwu Wang
- **Track**：Journal Track（ACM TOG 45(4)，22 页）
- **链接**：
  - 项目主页：https://nvlabs.github.io/motionbricks/
  - arXiv：https://arxiv.org/abs/2604.24833 （HTML v1：https://arxiv.org/html/2604.24833v1 ，提交 2026-04-27）
  - DOI：10.1145/3811334
  - **代码（预览版已开源）**：GR00T-WholeBodyControl/motionbricks（含① 交互式 G1 demo ② 自包含合成训练管线 + BONES-SEED 接入说明）；完整版（嵌入 GR00T Whole-Body Control 机器人形式化+ 完整训练管线）计划约一个月后发布
  - 数据集：350k 数据集连同下载与使用说明发布于项目页；相关开源条目 BONES-SEED https://bones.studio/datasets（参考文献称 ≈140k clips）
  - 姊妹项目：Kimodo（离线生成，arXiv 2603.15546）、GEAR-SONIC、SOMA Retargeter
- **官方 Description**："MotionBricks is a real-time generative framework that transforms interactive motion control for animation and robotics. By combining a large-scale latent backbone with intuitive 'smart primitives,' it delivers high-quality, zero-shot motion synthesis at 15,000 FPS..."
- **核心问题**：生成式动作合成虽进展巨大，但实时交互式动作控制仍由传统技术主导。两大瓶颈：① **实时可扩展性**——工业应用要求实时生成海量技能，生成式方法在实时算力约束下质量与可扩展性显著退化；② **集成**——工业需要速度指令 + 风格选择 + 精确关键帧的细粒度多模态控制，现有文本/标签驱动模型无法满足。
- **方法与表征（tokenizer 细节丰富）**：
  - **离散量化**：标准 **VQ-VAE 量化损失（含 stop-gradient）**，实践用**running-mean codebook update**（Razavi et al. 2019）稳定训练；**FSQ 可替换，性能相近**（附录 B 有对比）
  - **不是 RVQ**，而是**沿特征维并行切分的多头（multi-head）量化**：K 个独立码本 {e_1,…,e_K}，连续潜码 z_e^t 沿特征维分块后各自最近邻查找得到 z_q^t。设计动机：不用单一巨型码本、也不按身体部位人工切分，完全数据驱动分解；多头量化使潜流形更鲁棒，单 token 预测错误时"graceful degradation"
  - **推荐每头 128–256 tokens**；容量结论有两处表述："FID 与关键帧精度的最佳折衷在 **1e6–1e7 tokens 总容量**"、多头消融小节建议"总码本容量约 **1e9 tokens**"（原文如此，存在张力）
  - **时间压缩**：下采样率 **4×**（T 帧 → T/4 tokens；编码器两级下采样，倍率 2 和 4）
  - **片段长度 T：12–64 帧，步长 4 帧随机采样**；**帧率统一 30 FPS**
  - Root module 用 **16 个可学习 frame-slot embedding**，4× 下采样后覆盖最多 64 帧；每个 embedding 解码为 4 帧连续 root
  - **架构与参数量**：
    | 组件 | 架构 | 参数量 |
    |---|---|---|
    | Tokenizer（enc/dec） | U-Net，下采样率 4（2 层），每层 3 个残差 **1D 卷积**，**1024 通道**，kernel 逐层增大，编解码器对称 | **23.5M** |
    | Pose Module | Transformer encoder，dim **1024**，**16 heads**，**16 layers** | **150M** |
    | Root Module | 同类结构层数更少，dim **512**，**12 heads** | **50M** |
    - 试过 Transformer 版 enc/dec：性能相近但推理明显更慢，故用卷积
    - Tokenizer 额外加 **foot sliding loss + velocity loss**（对重定向数据集尤为重要）
  - **Root–Pose 解耦**：编码器只编码局部姿态不含 root；解码器把 root 轨迹与关键帧通过 **skip connection 在每个上采样层注入**（root 特征按 4→2→1 时间堆叠后与 token embedding 沿特征维拼接）；稀疏关键帧零填充到 T，布尔 mask 决定用 skip 特征还是隐状态；训练随机采样 **0–10 个关键帧**，推理用**前 4 帧 context + 1–4 个 target 关键帧**
  - **状态表示**（每帧 (r_g, r_l, p, q, v, c)）：r_g = pelvis 投影全局位置 + heading cos/sin；r_l = pelvis 投影线速度与角速度；p,q = 除投影 root 外所有关节位置与旋转；v,c = 关节速度与接触标签。**旋转用全局坐标**（支持爬行、翻转等 heading 定义不良的动作），**不做 heading 规范化**，改用随机旋转增强
  - **4阶段 coarse-to-fine 推理**：Smart Primitives → 关键帧约束 → Root Module（Step1 预测帧数 T₂ / Step2 预测 root 轨迹，帧数分布以 4 帧为分辨率、支持 bit-wise mask 做时序控制）→ Pose Module（**masked token modeling**，cosine schedule 混合 GT/mask token；**推理通常单次前向即可**）→ Decoder（连续运动）
  - **Smart Primitives**：完全 **zero-shot**，无需预定义控制类型/one-hot 任务标签/任务描述符/微调，一切通过统一关键帧接口通信（context 关键帧 = 角色近期状态，target 关键帧 = primitive 目标），等价于构造"全连接 motion graph"
    - Smart Locomotion：四阶段渐进 root 轨迹精修（naive 线性外推 1.0 s → 临界阻尼弹簧 r(t)=e^{-γt}((r_0-r_{g,1})+(v_0+γ(r_0-r_{g,1}))t)+r_{g,1} → root module 神经精修 → decoder 结合全身细节精修）；风格控制 = 把风格关键帧放到弹簧平滑轨迹上；支持 2D blendspace 生成关键帧
    - Smart Object：Intent Keyframes（**drop-frame 属性 τ**：τ=0 硬约束需精确到位，τ>0 可提前最多 τ 帧推进；**布尔标志 ω** 指示是否绕交互 pivot 旋转）+ Interaction Binding（interaction mesh 作trigger volume 的 box collision trace 检测、物体 socket 与ray cast 摆放、以 mesh 世界变换为 pivot 锚定并旋转关键帧）
    - 创作成本：非专家用户每个 primitive **< 10 分钟**
  - **训练**：32 GPU（4 节点 × 8 卡，16 卡可得相似视觉质量）；**2,000,000 updates**，每 GPU batch 256；Adam lr 5e-5，cosine，10k warmup，衰减到 2e-6；H100上 tokenizer ≈ 7 天、root ≈ 3 天、pose ≈ 7 天
  - **部署**：ONNX + TensorRT；UE5 原生 C++ 插件（用内置实时 retargeter 转到 "Messenger"/"Guard" 角色）；机器人端把生成运动作为 tracking target 输入物理跟踪控制器（Luo et al. 2025），Jetson Orin + Unitree 官方 SDK
- **数据集（Table 2）**：
  | Dataset | Hours | Train clips | Test clips | Joints |
  |---|---|---|---|---|
  | **350k**（自有mocap） | **700** | **315,162** | **35,018** | 27 |
  | 70k / "Bones-70k"（350k 子集） | 140 | 62,132 | 35,018 | 27 |
  | HumanML3D | 28.6 | 23,206 | 2,578 | 22 |
  | LaFAN1-G1 | 4.6 | 2,362 | 262 | 34 |
  - 350k：约 700 小时高质量动捕、**9,300 个独立技能 / 36 类/ 163+ 位表演者**，涵盖locomotion、combat、sports、object interaction；测试集 = 全量 10%（一个 5% 随机划分 + 一个 5% 按技能类别划分以最大化分布差异）
  - LaFAN1-G1：LaFAN1 重定向到 Unitree G1（29 hinge + 1 free pelvis，再加 4 末端执行器 → 34 关节），长片段切为 6 s，用改造版 GMR
- **关键结果（350k，Table 3 节选）**：
  | Method | FPS↑ | Latency↓ | MMD↓ | FID↓ | Win↑ | Score↑ | Jnt Jit↓ | Root Jit↓ | Foot Sk↓ | Pene↓ | Foot Acc↑ | Tgt KF↓ | Tgt Root↓ | Reach↑ |
  |---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
  | Ground Truth | – | – | 0.0533 | 0.022 | – | – | 3.47 | 1.96 | 0.0008 | 0.0009 | – | – | – | – |
  | Cond. Inbtwn.(2022) | 27,000 | 2.4ms | 0.1093 | 1.594 | 0.8% | 1.83 | 16.88 | 12.65 | 0.018 | 0.034 | 51.8% | 0.078 | 0.051 | 87.7% |
  | CondMDI(2024) | 1,930 | 33.2ms | 0.1080 | 1.213 | 15.6% | 3.13 | 16.19 | 14.62 | 0.012 | 0.009 | 62.7% | 0.143 | 0.118 | 61.3% |
  | MMM(2024) | 3,600 | 18.1ms | 0.1176 | 1.544 | 19.9% | 3.19 | 5.40 | 3.11 | 0.005 | 0.011 | 86.5% | 0.377 | 0.071 | 34.8% |
  | Closd-DiP(2025) | 4,200 | 15.3ms | 0.1076 | 1.292 | 15.1% | 2.90 | 14.03 | 11.94 | 0.015 | 0.010 | 52.7% | 0.129 | 0.091 | 75.7% |
  | **Ours** | **15,000** | **2ms** | **0.1056** | **1.054** | **86.5%** | **4.06** | **3.38** | **1.82** | **0.003** | **0.008** | **92.6%** | **0.076** | **0.023** | **99.6%** |
  - 抖动 3.38 / 1.82 **已接近甚至优于 GT（3.47 / 1.96）**；人评胜率 86.5%、Likert 4.06/5（**40 名参与者**，1,680 级别的成对比较由 3 随机方法中选胜者）
  - 其他数据集（Table 4）本方法 FID：LaFAN1-G1 **0.891**（次优 CondMDI 1.004）、HumanML3D **0.914**（次优 CondMDI/MMM 1.054/1.068）、Bones-70k **1.090**；Reach 分别 61.1% / 98.9% / 99.6%
  - 部署实测：Desktop **RTX 5090 → 2 ms / 15,000 FPS**；**Jetson Orin（G1）→ 5 ms**；重规划频率 UE5 与 G1 均为 **10 Hz 或指令变化触发**（lazy replanning）
  - 指标定义：MMD + FID 在按 TMR（去文本分支）重训的 motion autoencoder 潜空间中计算；关键帧/root 误差用 **max per-joint position error**（而非 MPJPE，作者认为更敏感）；**Reaching Success = root 位置误差 < 5 cm 且 heading 误差 < 15°**
  - 消融：**多头 vs 单头（Fig.10）**：多头 tokenizer 从 1e3 扩到 1e9 tokens 持续改善；单头 VQ-VAE 很快饱和且各规模下均显著更差。固定总容量 ~1e6 时（Fig.11）每头 token 过多损害 NPSS（时间一致性）、过少损害 FID 与关键帧精度 → **推荐每头 128–256**。数据规模缩放（Fig.13）：FID 随数据变大反而变差（作者假设小数据过拟合导致多样性低、更贴近训练分布），而关键帧精度随数据持续提升
- **是否涉及 tokenizer**：**是（细节最丰富的一篇之一）**。多头 VQ-VAE（可换 FSQ），每头 128–256 tokens，4× 时间下采样，30 FPS，片段 12–64 帧，tokenizer 为 1D 卷积 U-Net（1024 通道，23.5M 参数）。
- **信息缺口**：z_e/z_q 具体维度数值、K（头数）最终取值；最优总码本容量正文两处不一致（1e6–1e7 vs ~1e9）；BONES-SEED 的小时数/切分未在正文出现。

---

## 4. Deep Motion Warping via Phase-Conditioned Diffusion Autoencoder

- **Session**：Motion Generation, Warping & Control（14:30–14:40）
- **作者**：Bowen Zheng（Zhejiang University；Tencent VISVISE）、Linjun Wu（Zhejiang University）、Xinwei Jiang（Tencent VISVISE）、Yujin Chai（Tencent VISVISE）、Zijiao Zeng（Tencent VISVISE）、He Wang（University College London, UCL）、Xiaogang Jin（Zhejiang University）
- **Track**：SIGGRAPH2026 Conference Papers（Article 93，11 页）
- **链接**：
  - DOI：10.1145/3799902.3811144
  - DBLP：conf/siggraph/ZhengWJCZWJ26（SIGGRAPH 2026: 93:1–93:11）
  - arXiv / 项目主页 / 代码：**未找到**（截至采集日arXiv 无对应预印本；作者主页 http://www.cad.zju.edu.cn/home/jin/pub.htm 仅列出条目，无 project page 链接）
- **官方 Description**："We present a deep motion warping framework utilizing a phase-conditioned diffusion autoencoder. By explicitly disentangling motion into root velocity, phase, and style, it enables intuitive, high-level character animation editing including root warping, exaggeration, time warping, and style transfer while strictly preserving structural consistency."
- **核心问题**：如何对已有动作做高层、直观的编辑（root 变形、夸张化、时间变形、风格迁移）而不破坏结构一致性。
- **方法与表征**：
  - **Phase-Conditioned Diffusion Autoencoder**：把动作**显式解耦为 root velocity + phase + style** 三个因子；以 phase 作为条件的扩散自编码器
  - 表征类别：**连续 latent + diffusion**（diffusion autoencoder），**无离散量化**
  - 网络层数 / latent 维度 / phase 通道数 / 训练数据 / 定量指标：**未找到**（无公开预印本或补充材料）
- **数据与结果**：**未找到**
- **是否涉及 tokenizer**：**否**（基于 phase 条件的连续 diffusion autoencoder；据官方 Description 无任何量化/码本描述）。
- **相关背景**（同组前作，供参照）：AutoKeyframe（SIGGRAPH 2025, 自回归关键帧生成）；Semantically Consistent Text-to-Motion with Unsupervised Styles（SIGGRAPH 2025）；Decoupling Contact for Fine-Grained Motion Style Transfer（SIGGRAPH Asia 2024）；Ultrafast and Controllable Online Motion Retargeting for Game Scenarios（TOG 44(6) 2025）

---

## 5. MultiAct: Text-to-Motion Generation from Composite Text via Tailored Attention Guidance

- **Session**：Motion Generation, Warping & Control（14:40–14:50）
- **作者**：Nathan Sala（Tel Aviv University）、Ofir Abramovich（Reichman University；CYENS Centre of Excellence）、Ariel Shamir（Reichman University）、Daniel Cohen-Or（Tel Aviv University）、Andreas Aristidou（University of Cyprus；CYENS Centre of Excellence）、Sigal Raab（Tel Aviv University）
- **Track**：SIGGRAPH 2026 Conference Papers
- **链接**：
  - arXiv：https://arxiv.org/abs/2605.30925 （HTML：https://arxiv.org/html/2605.30925v1 ，提交 2026-05-29）
  - 项目主页：https://natsala13.github.io/multiact.github.io
  - DOI：10.1145/3799902.3811092
  - 代码：项目页/UCY 出版页均列出 code 链接（https://graphics.cs.ucy.ac.cy/publications 标注 "DOI paper video code bibtex project page"）
- **核心问题**：现有 text-to-motion 模型在**复合提示（同时发生的多个动作）**下脆弱——常只实现主导动作而忽略其余成分（作者称**vanishing semantics**），归因于 **entangled cross-attention**：多语义成分竞争时注意力质量塌缩到少数主导 token。注意本文针对的是**同时发生**的动作（"running while waving arms"），与顺序动作（"walk, then jump"）根本不同。
- **方法与表征**：
  - **推理时（inference-time）、无配对数据（unpaired）、免重训练/免改架构**的框架，直接作用于**预训练动作生成器**之上
  - 机制：**自适应放大与被弱表征提示成分相关的 cross-attention 分数**
  - 关键洞察：有效调制依赖 prompt 特定选择（**该调制哪些 token、哪些层**），故引入**轻量辅助决策方案**自动确定最优的attention-strengthening 参数化
  - 表征类别：作用于既有扩散式 T2M backbone 的注意力上，**本身不引入新的动作表征，也不涉及量化**
  - Backbone 名称、层选择策略细节、数据集与指标数值：**未找到**（当前抓到的 HTML 仅到 Introduction；官方 session页面对此论文未打 AI/ML 标签，仅 Animation / Simulation）
- **关键结果**：定性（Fig.1：backbone 在 "hops forward while raising his arms"、"dribbles a ball while moving backwards" 上漏掉一个动作成分，MultiAct 全部生成）；论文称"在复合提示上一致优于既有基线，语义覆盖提升且保持动作真实性"，**具体数值未找到**
- **是否涉及 tokenizer**：**否**。

---

## 6. Autoregressive Diffusion with Hybrid Representation for Interactive Human Motion Generation（ARDY）

- **Session**：Motion Generation, Warping & Control（14:50–15:00）
- **作者**：Kaifeng Zhao（ETH Zürich；NVIDIA Spatial Intelligence Lab 实习）、Mathis Petrovich、Haotian Zhang、Tingwu Wang、Siyu Tang（ETH Zürich）、Davis Rempe（NVIDIA）
- **Track**：Journal Track（ACM TOG 45(4), Article 86，14 页）
- **链接**：
  - arXiv：https://arxiv.org/abs/2607.08741 （HTML：https://arxiv.org/html/2607.08741v1 ，提交 2026-07-09）
  - 项目主页（含视频/代码/模型）：https://research.nvidia.com/labs/sil/projects/ardy/
  - 代码仓库：https://github.com/nv-tlabs/ardy
  - DOI：10.1145/3811284
- **核心问题**：离线方法（如 Kimodo）控制精确但推理速度不足以交互；在线方法虽实时，却牺牲可控性，或因上下文窗口有限而难处理复杂文本语义与长时目标。
- **方法与表征（tokenizer 细节丰富）**：
  - **混合表征**：token x = [m_root ; x_body] ∈ R^D，**D = L + 5P = 128 + 20 = 148**
    - **显式 root 特征**：每帧 m_root = (p, cos ψ, sin ψ) ∈ **R^5**（p 为全局根位置，ψ 为朝向角）；patch 化后 m_root^{1:T} ∈ R^{T×5P}，默认 **P = 4 → 每token 20 维**
    - **潜在身体嵌入**：x_body ∈ **R^L，L = 128**
    - 显式 body 特征（tokenizer 输入与约束表示）：m_body = (θ ∈ R^{6j} 全局关节旋转 6D、J ∈ R^{3j−3} 非根关节位置、J̇ ∈ R^{3j} 全局关节速度、c ∈ R^4 双脚二值触地)
  - **量化：默认 FSQ（Finite Scalar Quantization），每特征量化到 64 个离散 level →记为 "FSQ 64-128"**
    - 同时实现并对比 **vanilla 连续 AE / VAE（KL 权重 1e-6, 128D）/ FSQ**：三者性能相当，但**vanilla AE 在长 horizon（如 40 帧）训练时严重不稳定甚至发散**，FSQ 训练稳定性最佳 → 设为默认
    - **例外**：HumanML3D benchmark 实验用的是 **vanilla autoencoder tokenizer**（40 帧 horizon、10 步扩散）
    - 容量消融：FSQ 16-32 / 64-32 / **64-128（默认）** / 64-256
  - **Tokenizer 架构**：**非对称条件自编码器**；Encoder 与 Decoder 各为 **8 层 Transformer、latent dim 512、因果自注意力**；Decoder 以根运动为条件，且内部做 **global→local 根转换**（ψ̇, ṗ_x, ṗ_z, p_y）替代全局根作条件，显著抑制脚滑；损失 = 重建 +脚滑损失 L_skate（按预测触地标签加权脚部速度模长，**权重 0.01**）
    - 训练：AdamAtan2，lr **2e-5**，batch **128**，**4,000,000 步**，cosine + 10k warmup，**单张 A100-80GB**，片段 1–10 秒
  - **运动基元长度与帧率**：patch **P = 4 帧/token**（消融 1/4/8）；生成窗口 **G = C·P，默认 G = 40 帧 = 2 秒 @ 20 fps**（horizon 消融 4/8/12/20/40）；**数据帧率 20 fps**
  - **两阶段自回归 Transformer 去噪器（交错式）**：① Root Transformer 预测干净的显式全局根运动 → **detach** → ② Body Transformer 预测干净的潜在 body token → 拼接为 x̂₀，推理时重新加噪送回，**每个去噪步都交错执行**，最后由 tokenizer decoder 解码
    - 每个 Transformer：**8 层、8 头、latent size 1024**；部署模型总参数约 **156M**；文本编码器 **LLM2Vec（Llama-3-8B-Instruct）**；不使用 dropout（否则丢失根约束条件输入）
    - **可变历史上下文 H：0 ~ 8 秒**；**超窗口未来约束上下文 F：最大 10 秒**（对比：DiP 1.00/2.00；DartControl 0.07/0.27；MotionStreamer 10.00/10.00 但无运动学约束）
    - 扩散：**DDPM**，默认训练与测试 **10 步**（消融 1/2/3/4/10/100；4 步对多数应用可接受）
    - 损失 L = L_hybrid + L_dec + L_goal + L_consist；优化器 AdamAtan2 lr 2e-5，batch 512，**4× A100-80GB，1,000,000 步**；CFG 文本与空间约束各 **10%** 概率随机丢弃
    - 约束注入：窗口内根约束直接覆写噪声 token 根部分；身体约束沿特征维拼接；超窗口约束作为额外 token 输入（未约束 token 在注意力中被 mask）。采样的约束类型：2D 根关键帧、2D 根轨迹、全身稀疏关键帧、全身关键帧块、稀疏末端执行器关键帧、足部接触关键帧
- **数据集**：
  - **Bones Rigplay Mocap Dataset**（专有）：约 **700 小时**、studio 级、带文本描述；**>150 人**；统一 **27 关节**骨架；数千种行为（locomotion、日常活动、手势、格斗）；原始片段 1–180 s → 裁剪 ≤10 s、下采样 20 fps；LLM 生成标签改写；按语义内容分组不相交划分约 **90/10**；**训练 ≈315k / 测试 ≈35k clips**；评测器基于 TMR 训练，R-precision 在**约 5k 唯一样本**上检索（远难于 HumanML3D batch=32）
  - **HumanML3D**：约 30 小时；**排除 HumanAct12 子集**；重定向时**保留原始 SMPL 关节旋转**（原pipeline 丢弃）；用原始 HumanML3D evaluator
- **关键结果**：
  - Bones Rigplay 架构消融（Table 2）：**ARDY** Skate(text) 0.264 / R-prec **65.47** / **FID 0.027** / Skate(cons.) 0.250 / **Joint rot 2.23°** / **Joint pos 0.025 m** / **Keyframe body 0.023 m** / Traj 0.015 m / **Waypoint 0.024 m**；Explicit representation 变体 FID 0.065 / Waypoint 0.203；Global root-cond. decoder FID 0.028 / Waypoint 0.060；One-stage FID 0.029 / Waypoint 0.164。GT: Skate 0.255 / R-prec 76.56 / FID 0.000
  - Horizon 消融：4 帧严重退化（FID 0.224 / Waypoint 0.850）；8/12/20/40 → FID 0.037/0.033/0.030/**0.027**
  - 扩散步数：1 步 FID 0.079；**4 步 FID 0.034（Waypoint 0.027）**；**10 步 FID 0.027**；100 步 FID 0.025（Traj 最优 0.009）
  - Tokenizer patch：P=1 严重退化（FID 0.152）；P=8 FID 0.022 / R-prec 68.01 但Joint pos 0.070；P=4 为综合最优
  - HumanML3D 与离线法对比（Table 4）：**ARDY（无优化）** R-Prec 0.729 / FID 0.044 / Skate **6.28%** / Error **4.15 cm** / Latency **0.15 s**；MaskControl（无优化）0.760 / 0.050 / 7.27 / 46.18 / 0.46；MaskControl（含优化）0.758 / 0.047 / 7.87 / 0.45 / **68.65 s**；**ARDY Opt** 0.721 / 0.088 / 5.87 / **0.30 cm** / 9.25 s
  - HumanML3D 与自回归法对比（Table 5，序列 9 s，给 1 s GT 历史，目标关节采样自 pelvis/双腕/双脚）：In-horizon DiP 0.609 / 0.967 / 12.29 / 9.20 vs **ARDY 0.690 / 0.092 / 7.07 / 2.48**；Out-of-horizon DiP 0.599 / 1.453 / 11.07 / 17.64 vs **ARDY 0.684 / 0.100 / 7.63 / 2.92**
  - 实时性：**RTX 4090**；4 步模型平均生成延迟 **33 ms**（≈30 次规划/秒，每次生成 2 秒动作，0 帧 buffer）；10 步模型 **63 ms**（1 帧 buffer）；latency-aware non-blocking 重规划
  - 感知实验：与 DiP 在 out-of-horizon 目标下三维度对比（运动质量/语义对齐/关节目标精度），**共 240 组成对人类比较**，ARDY 被"strongly and consistently preferred"（Table 6 具体百分比在公开 HTML 中被截断，**未找到**）
  - Demo：基于 **Viser** 的 Web 界面，动态改文本 prompt、可视化红色约束、时间轴轨道显示未来 prompt 与约束；鼠标点waypoint、键盘控制目标速度
  - **jitter 无定量指标**（用 foot skating 作代理；jitter 仅在 Limitations 定性提及）
- **是否涉及 tokenizer**：**是（用户最关心的类型）**。FSQ 64 levels × 128 维、patch 4 帧/token、20 fps、Transformer 8 层 512 维非对称条件 AE；论文对 AE / VAE / FSQ 做了直接对比，并指出 FSQ 在长 horizon 上训练稳定性最佳。

---

# Session：Learning in Motion（sess105）

- 时间/地点：2026-07-23（周四）14:00–15:30 PDT，Room 403 A
- Keywords：Animation / AI-ML / Modeling / Simulation

---

## 7. R-DMesh: Video-Guided 3D Animation via Rectified Dynamic Mesh Flow

- **Session**：Learning in Motion（14:00–14:10）
- **作者**：Zijie Wu（Huazhong University of Science and Technology；Tencent Hunyuan）、Lixin Xu（Tencent Hunyuan，project lead）、Puhua Jiang（Tencent Hunyuan）、Sicong Liu（Tencent Hunyuan）、Chunchao Guo（Tencent Hunyuan）、Xiang Bai（HUST，通讯）
- **Track**：SIGGRAPH 2026 Conference Papers
- **链接**：
  - arXiv：https://arxiv.org/abs/2605.13838（HTML v2：https://arxiv.org/html/2605.13838v2 ；v1 2026-05-13，v3 2026-06-27）
  - 项目主页：https://r-dmesh.github.io/
  - **代码（已开源）**：https://github.com/Tencent-Hunyuan/R-DMesh
  - DOI：10.1145/3799902.3811135
  - CCS：Computing methodologies → Motion capture
- **核心问题**：视频驱动 3D 动画的**位姿错配（pose misalignment）困境**——用户提供的静态 mesh 初始位姿几乎不会与参考视频首帧对齐，强行让mesh 跟随不匹配轨迹会导致严重几何畸变或动画失败。
- **方法与表征**：
  - **R-DMesh VAE**：显式解耦输入为 ① 条件 base mesh ② 相对运动轨迹 ③ **rectification jump offset**（学习把输入 mesh 的任意位姿变换到视频初始状态，在动画开始前完成"校正"）
  - **Triflow Attention**：用逐顶点几何特征调制三条正交 flow，保证校正与动画过程中的物理一致性与局部刚性
  - **生成器：Rectified Flow-based Diffusion Transformer（DiT）**，以预训练视频 latent 为条件（视频特征由 **Wan2.2-TI2V-5B** 提取），把时空先验迁移到 3D
  - 表征类别：**连续 latent（VAE）+ rectified flow**；**直接预测顶点轨迹，不依赖骨骼rigging / 形状先验**（因此可驱动拓扑变化的角色与普通物体）
  - 是否量化：**否**
- **数据与结果**：
  - **Video-RDMesh**：自建大规模数据集，**> 500k（50 万+）动态 mesh 序列**，专门构造以模拟位姿错配
  - 训练设置为same-identity video-4D 对，但可泛化到不同 identity 的参考视频，含 in-the-wild 素材
  - 应用：pose retargeting / motion retargeting、holistic video-to-4D（与 Hunyuan3D 结合把 in-the-wild 视频转为高质量 4D 资产）
  - 定量对比表数值：**未找到**（项目页与摘要仅声明"与 SOTA 的 video-to-4D 方法对比"，未列出数字）
- **是否涉及 tokenizer**：**否**（VAE 连续 latent + rectified flow DiT）。

---

## 8. TopoCap: Learning Topology-Agnostic Motion Priors for Monocular Video-to-Animation

- **Session**：Learning in Motion（14:10–14:20）
- **作者**：Cheng-Feng Pu（Zhili College, Tsinghua University）、Jia-Peng Zhang（BNRist, CS Dept, Tsinghua University）、Meng-Hao Guo（BNRist, CS Dept, Tsinghua University）、Yan-Pei Cao（VAST）、Shi-Min Hu（BNRist, CS Dept, Tsinghua University）
- **Track**：Journal Track（TOG，submissionid 934）
- **链接**：
  - arXiv：https://arxiv.org/abs/2606.12153 （HTML：https://arxiv.org/html/2606.12153 ，提交 2026-06-10）
  - **数据集（已公开）**：https://huggingface.co/datasets/duckduckplz/Mobjaverse
  - **代码仓库**：github repo "TopoCap"（作者主页 http://czpcf.github.io/ 列出）
  - DOI：10.1145/3799902.3811159
  - CCS：Motion processing；Shape representations
- **官方 Description**："TopoCap is a unified framework for motion capture and retargeting from monocular video, supporting arbitrary skeletal topologies without test-time optimization..."
- **核心问题**：现有动捕方法脆弱——受限于物种专属模板（如 SMPL）或需要繁重手工 rigging。目标：从单目视频提取动作并零样本重定向到**任意、未见过的骨骼拓扑**（双足→ 六足 → 无生命物体），且**无需test-time optimization**。
- **方法与表征**：
  - 核心洞察：**骨骼结构是组合且离散的，但运动的底层物理占据一个连续、低维的流形**
  - **Stage I（Manifold Discovery）**：用 **Graph CVAE** 把异构运动链压缩为共享、**定长的latent code（K × D）**，采用 **Perceiver式瓶颈**；解码器**显式条件于目标 rig 的结构嵌入**，从而解耦运动动力学与骨骼拓扑；解码用**解析 IK** 保证全局一致性
  - **Stage II（Generative Extraction）**：把 video-to-animation 建模为**条件 flow matching**问题，从视觉特征预测这些拓扑无关的 code
  - 表征类别：**连续 latent（Graph CVAE）+ flow matching**；**无离散量化**（注意：作者明确把"骨骼结构离散/运动流形连续"作为设计前提）
  - K、D 的具体数值：**未找到**
- **数据与结果**：
  - **Mobjaverse**：从 Objaverse-XL 精选构建，**> 5,000 种独立骨骼拓扑、200 万帧**，结构多样性超过既有数据集**两个数量级**；标签分布中双足与四足为主，含六足、蛛形、家具等重尾（Fig.2）；五步 curation pipeline 清理断裂层级、退化、无意义运动
  - 结果：在人体与四足 benchmark 上优于专家模型（对比含 Puppeteer 等优化式方法），并对长尾 3D 生物实现零样本重定向；支持跨拓扑重定向（交换目标 rig 条件 S，可把四足跌落迁移到飞龙，保留 gait/energy/phase 高层语义而适配低层运动学）；可作为**可扩展的运动数据生成引擎**
  - 具体定量表数值：**未找到**（HTML 抓取内容未含表格数字）
  - 失败情形：目标拓扑极不常见时，提取的运动可能显著偏离（Fig.6）
- **是否涉及 tokenizer**：**否**（连续流形 + flow matching）。**但同组姊妹工作 SkinTokens / TokenRig（Jia-Peng Zhang, Cheng-Feng Pu, Meng-Hao Guo, Yan-Pei Cao, Shi-Min Hu，arXiv 2602.04805）使用 FSQ-CVAE 将 skinning weights 离散化为 token，配合 Qwen3-0.6B 自回归 + GRPO 强化学习，skinning 精度提升 98%–133%、骨骼预测提升 17%–22%；代码 https://github.com/VAST-AI-Research/SkinTokens（checkpoint 名如 skin_vae_2_10_32768、articulation_xl_quantization_256_token_4）。该文非 SIGGRAPH 2026 主会论文，仅作离散结构表征参照。**

---

## 9. Stylized Text-to-Motion Generation via Hypernetwork-Driven Low-Rank Adaptation（HyperLoRA）

- **Session**：Learning in Motion（14:20–14:30）
- **作者**：Junhyuk Jeon*、Seokhyeon Hong*、Junyong Noh†（均为 Visual Media Lab, KAIST, Republic of Korea；*equal contribution）
- **Track**：SIGGRAPH 2026 Conference Papers
- **链接**：
  - arXiv：https://arxiv.org/abs/2605.13333 （提交 2026-05-13）
  - 项目主页 / 代码 / 视频：作者主页 https://seokhyeonhong.github.io/ 列出 "project page / paper / code / video"（具体 URL 未在抓取内容中给出，**独立项目页 URL：未找到**）
  - DOI：10.1145/3799902.3811205
- **核心问题**：文本难以表达动作的细粒度风格。既有风格化方法要么需要**按风格微调**（LoRA-MDM：一次微调只对应单一风格，牺牲可扩展性与泛化），要么依赖**重型 ControlNet 架构**（SMooDi：容量与算力开销大）；风格表征上，标签监督法（依赖 100STYLE 类别）偏向已见标签，无标签连续法缺乏显式结构，均限制对未见风格的泛化。
- **方法与表征**：
  - **HyperLoRA**：内容无关的 **style adapter**（MLP 堆叠）从参考动作提取**全局 style embedding**；**hypernetwork** 把该 embedding 映射为 **LoRA 低秩更新**（实现上为 style-dependent **FiLM 调制**参数），在**每个去噪步**注入**冻结的预训练 text-to-motion 扩散模型**
  - **supervised contrastive loss** 结构化 style latent 空间，使其内容独立、对未见风格鲁棒
  - 引导机制：标准 **classifier-free guidance** + 新的 **style encoder guidance**（用训练好的 style encoder 做梯度引导，把去噪过程导向目标**连续** style embedding，无需离散风格类别）
  - 表征类别：**连续 latent + diffusion**（在冻结扩散主干上做参数高效调制）；**无量化**
  - 基线参照架构：SALAD（同组 CVPR 2025，skeleton-aware latent diffusion）的 FiLM 机制（Fig.3 有对比）
- **数据与结果**：
  -数据集：**HumanML3D** + **100STYLE**（消融含"训练 25 styles"与"训练 100 styles"两种设定）
  - **Style Recognition Accuracy (SRA)：76.034%**（SMooDi 75.818%、LoRA-MDM 42.047%）；**R-Precision 0.721**；**Multimodal Distance 3.519**
  - 用户研究：style expression **4.215**、content preservation **4.520**、motion quality **4.050**，均显著高于所有基线
  - 消融：supervised contrastive loss 与 style encoder guidance均为泛化到未见风格的关键（尤其在训练风格数受限时）
  - 应用：motion style transfer（内容动作 + 参考风格）
- **是否涉及 tokenizer**：**否**。

---

## 10. MUSIC: Learning Muscle-Driven Dexterous Hand Control

- **Session**：Learning in Motion（14:30–14:40）；另有 Emerging Technologies Demo（2026-07-21 12:20–13:20，Concourse Hall，presenter Pei Xu）
- **作者**：Pei Xu*（Stanford University）、Yufei Ye*（Stanford University）、Shuchun Sun（Clemson University；官方页写作 "Shuchu Sun"）、Yu Ding（Stanford University）、Elizabeth Schumann（Stanford University）、C. Karen Liu（Stanford University）；*equal contribution
- **Track**：Journal Track（ACM TOG 45(4)）
- **链接**：
  - arXiv：https://arxiv.org/abs/2604.23886 （PDF：https://arxiv.org/pdf/2604.23886 ，提交 2026-04-26）
  - 项目主页：https://pei-xu.github.io/MUSIC
  - DOI：10.1145/3811402
  - 代码：**未找到**（项目页仅列abstract/video/method/repertoire/bibtex）
- **核心问题**：让**肌肉骨骼手（musculoskeletal hand）**在基于物理的仿真中完成精细灵巧控制，并能演奏**参考数据集之外的新钢琴曲目**。
- **方法与表征**：
  - **分层架构**：高频**肌肉级控制** + 低频**latent 空间协调**
  - **三阶段训练**：① RL 训练单手tracking policy，输出高频**肌肉-肌腱激活**以跟踪大规模参考动作数据集轨迹；② **on-policy distillation** 把 tracking policy 蒸馏为 **VAE**，得到平滑、结构化的 latent 空间（抽象掉低层肌肉动力学），VAE decoder 作为低层servo；③ 在该 latent 上训练**曲目专属高层控制器**，以乐谱抽取的 note events 为目标协调双手动作
  - 高层控制建模为**去中心化多智能体 RL（两手为独立 agent、独立策略，但共享双手运动学观测）+ 对抗式学习**做动作模仿
  - 另提出**增强的肌肉骨骼手模型**，支持手指精细控制
  - 表征类别：**连续 latent（VAE）**；**无离散量化**
  - 肌肉-肌腱单元数量、latent 维度、控制频率具体数值：**未找到**
- **数据与结果**：
  - 曲库（15 首，多风格多技术难度）：Bach English Suite / French Suite；Beethoven Für Elise / Piano Sonata No.23 / No.3；Brahms Rhapsodie Op.79；Chopin Waltz Op.64；Debussy Suite Bergamasque Passpied；Grieg Lyric Pieces Op.38；Joplin Maple Leaf Rag；Mozart K.280 / K.545；Mussorgsky Pictures at Exhibition Bydlo；Scarlatti K.208；Scriabin Piano Sonata No.5
  - 结果：能合成按键准确的协调双手动作，在基于物理的灵巧控制中达到钢琴演奏 SOTA，并**泛化到参考数据集之外的新乐谱**；肌肉骨骼手模型在**生物力学稳定性与跟踪精度**上优于既有模型；肌肉激活模式与**人类 EMG 记录**在多任务下一致（生理可信）
  - 具体定量表数值：**未找到**
- **是否涉及 tokenizer**：**否**。

---

## 11. MOCHI: Motion Enhancement of Collaborative Human-object Interactions

- **Session**：Learning in Motion（14:40–14:50）
- **作者**：Jiye Lee*（Seoul National University）、Yonghun Choi（Seoul National University）、Jungdam Won†（Seoul National University）
- **Track**：Journal Track（ACM TOG 45(4), Article 160，18 页）
- **链接**：
  - arXiv：https://arxiv.org/abs/2606.18243 （PDF：https://arxiv.org/pdf/2606.18243 ，提交 2026-06-16）
  - 项目主页：https://jiyewise.github.io/projects/MOCHI/
  - DOI：10.1145/3811308
  - 代码：**未找到**（项目页列results/applications/video/bibtex，未见code 链接）
  - 备注：入选 SIGGRAPH 2026 Technical Papers Trailer；一作获 2026 WiGRAPH Rising Stars
- **核心问题**：多人-物协作交互（**MHOI**）数据采集极难——遮挡频繁、需同时捕捉大幅身体动作与手指级细节，导致采集数据存在三类伪影：①手-物接触未对齐 ② 运动抖动与时间不一致 ③ 手指关节细节缺失或不完整。
- **方法与表征**：
  - **两阶段框架**：
    - 阶段一：从含噪身体输入出发，通过**优化**生成**物理可信且与身体姿态语义一致的手部抓握**，再把优化后的抓握扩展为完整的手-物交互序列
    - 阶段二：用**基于扩散的 noise optimization 框架**（使用**单人 motion prior**）精修所有参与者的全身动作；在优化过程中引入目标项，把**人-物与人-人交互信息编码进单人先验**
  - 表征类别：**连续扩散先验 + 优化（diffusion noise optimization）**；**无离散量化**
- **数据与结果**：
  - 适用于既有采集方法获得的数据，也适用于生成模型合成的数据（用 **OMOMO**（从物体轨迹合成人体动作）演示增强生成模型输出，从有限训练样本产出无限量高质量 MHOI 数据）
  - 对不同**参与者数量**（含 >2 人）与交互类型稳健
  - 应用：**keyframe-based MHOI creation**（3D 物体由 off-the-shelf text-to-3D 生成，标注者在 **30 FPS 下每 40 帧间隔**的稀疏关键帧上粗略指定物体与人体位姿，方法补全为完整序列；文本提示例："a mini-sized cube shaped table"、"a large gothic style chest"）；**object variations数据增强**（用同类不同几何的物体替换场景中原物体）
  - 场景示例：抬椅子、搬桌子、放下箱子
  - 具体定量表数值：**未找到**
- **是否涉及 tokenizer**：**否**。

---

## 12. ACT: A Unified Framework for Rigging and Animating Characters with Arbitrary Topologies

- **Session**：Learning in Motion（14:50–15:00）
- **作者**：Pengyu Long（ShanghaiTech University；ByteDance Games）、Weirui Wang（ShanghaiTech University；Deemos Technology）、Qingcheng Zhao（University of Toronto；Deemos Technology）、Xiaoyang Guo（ByteDance Games）、Xiaoyu Pan（ByteDance Games）、Qixuan Zhang（ShanghaiTech University；Deemos Technology）、Jiaqing Zhou（ByteDance Games）、Tianlei Hu（ByteDance Games）、Wei Yang（Huazhong University of Science and Technology）、Lan Xu（ShanghaiTech University）、Jingyi Yu（ShanghaiTech University）
- **Track**：Journal Track（ACM TOG 45(4): 161:1–161:20）
- **链接**：
  - DOI：10.1145/3811392
  - ShanghaiTech KMS 记录：https://kms.shanghaitech.edu.cn/itemDetail?id=2079371260204204034
  - arXiv / 项目主页 / 代码：**未找到**
  - 基金：NSFC W2431046；National Key R&D 2025YFA1309603；Central Guided Local S&T YDZX20253100001001
- **官方 Description**："ACT is a unified AI framework that effortlessly brings static 3D characters to life. By integrating rigging, skinning, and motion generation into a single diffusion-based pipeline, ACT automatically animates characters of any shape..."
- **核心问题**：传统流水线把 rigging → skinning → motion synthesis 割裂为顺序阶段，忽视了**形态结构与运动功能之间的内在耦合**。
- **方法与表征**：
  - 把 rigging 与 animation 重构为**同一个 hyper-kinematic 过程的互补视图**；核心洞察是在**共享 latent 空间**中建模**骨骼拓扑与时序运动的联合分布**
  - 用 **VLM（Vision Language Model）** 从任意 mesh 抽取**语义拓扑先验**，条件化一个 **Diffusion Transformer（DiT）** 主干
  - 把**静态 rest pose 与动态轨迹视为统一序列**，采用**任务感知 masking 策略**，在单一端到端架构内灵活完成：**零样本 rigging、文本引导动作生成、动作补全（motion completion）**
  - **geometry-guided decoder** 保证表面形变与生成运动学紧密耦合
  - 表征类别：**连续共享 latent + DiT diffusion**；**无离散量化描述**
  - latent 维度、序列长度、VLM 具体型号：**未找到**
- **数据与结果**：
  - 泛化：无需重训练即稳健泛化到多样的**非人形**角色
  - 新应用：**语义驱动的拓扑编辑**、**生成式 in-betweening**
  - 具体数据集与定量表：**未找到**
- **是否涉及 tokenizer**：**否**（据摘要为共享 latent 空间 + DiT；未提及量化）。

---

# Session：Motion Transfer & Inbetweening（sess114）

- 时间/地点：2026-07-22（周三）09:00–10:30PDT，Room 403 A
- Session Chair：Pei Xu（Stanford University）
- Keywords：Animation / AI-ML / Robotics / Simulation

---

## 13. Motion4Motion: Motion Transfer Across Subjects at Inference

- **Session**：Motion Transfer & Inbetweening（09:00–09:10）
- **作者**：Ling-Hao Chen（Tsinghua University；Stepfun）、Zixin Yin（The Hong Kong University of Science and Technology；Stepfun）、Duomin Wang（Stepfun）、Xianfang Zeng（Stepfun）、Gang Yu（Stepfun）
- **Track**：SIGGRAPH 2026 Conference Papers
- **链接**：
  - arXiv：https://arxiv.org/abs/2607.11644 （HTML：https://arxiv.org/html/2607.11644 ，提交 2026-07-13）
  - 项目主页：https://lhchen.top/Motion4Motion/
  - DOI：10.1145/3799902.3811062
  - 代码：**未找到**
  - CCS：Computer vision；Animation
- **核心问题**：视频动作迁移主流方法**硬编码依赖预定义人体骨架**并需 skeleton-conditional 训练，难以泛化到跨物种（不同形态动物）同时保留其独特运动风格；且跨拓扑配对数据稀缺，限制大规模训练。两大根本挑战：①跨多样拓扑的高质量配对动作数据极度稀缺 ② 无骨架时源/目标之间的**语义对应存在歧义**（如椅子腿 vs 四足动物腿）。最相关的既有尝试 FlexiAct 依赖 per-case optimization，导致过拟合与信息泄漏。
- **方法与表征**：
  - **完全 training-free**：在推理时操纵**预训练 Diffusion Transformer（DiT）** 的自注意力
  - 不用骨架，改为建模视频中角色的**像素级 motion flow**；用**语义匹配算法**建立**skeleton-free 的点到点语义对应**
  - **TransPE（Transferring Positional Encoding）**：把目标角色首帧的外观特征（经 diffusion inversion 缓存的 K̂, V̂）注入注意力，并用迁移后的目标运动流 M_tgt 的位置信息重新做 RoPE 编码：
    - K_new = [RoPE(K), RoPE(K̂, M_tgt)]；V_new = [V, V̂]；X ← Softmax(RoPE(Q)·K_newᵗ/√d)·V_new
  - 表征类别：**连续（预训练视频扩散 latent）+ 注意力操纵**；**无量化，无训练**
- **关键结果**：
  - 定量指标（人体与动物动作迁移 benchmark）：**文本相似度 TS、运动保真度 MF、时间一致性 TC、外观一致性 AC、姿态相似度 PS全面显著优于基线**（**具体数值未找到**）
  - 定性：跨物种迁移（human→panda、human→goose、狐狸步态→长颈鹿、狮子扑击→斑马）保持高质量语义姿态对齐与清晰纹理，无需骨骼先验或微调；对互联网真实图像/视频（复杂舞蹈→静态肖像）忠实保留身份、服装纹理与结构完整性；甚至能给**无生命物体（一张桌子）**赋予人类行走步态
- **是否涉及 tokenizer**：**否**。

---

## 14. STyMo: Fast and Controllable Few-Shot Motion Style Transfer

- **Session**：Motion Transfer & Inbetweening（09:10–09:20）
- **作者**：Jose Luis Ponton（工作完成于 Meta Reality Labs 实习期间）、Alexander Winkler、Ladislav Kavan、Yuting Ye、Petr Kadleček（Meta Reality Labs）
- **Track**：Journal Track（ACM TOG）
- **链接**：
  - 项目主页：https://jlpm22.github.io/stymo-project-page
  - **代码（已开源，含处理后的配对数据集）**：https://github.com/facebookresearch/STyMo （MIT License）
  - DOI：10.1145/3811356
  - arXiv：**未找到**
- **核心问题**：支持多样动作风格对创建丰富虚拟角色至关重要，但现有方法要么需要大规模风格化数据集，要么依赖无法泛化出训练分布的预训练模型。
- **方法与表征**：
  - **few-shot**：仅从**数秒配对动作数据**学习风格，**训练 1–2 分钟**
  - 核心洞察：把风格分解为 ① **static component**（时不变姿态） ② **temporal component**（逐帧动力学）
  - **三阶段训练**：
    - **Static Model**（`model_static.py`，AveragePoseModel）：分类器捕捉姿态差异与全局偏移；对平均旋转分类以混合 K 个预计算 style chunks（static deltas），捕捉持续性姿态偏移
    - **Gating Model**（`model_gating.py`，**Stylizability Gate**）：用最近邻学习置信边界，预测逐帧 stylizability 分数 γ；输入姿态离训练数据过远时自动降低风格强度，**防止 OOD 动作上的伪影**
    - **Temporal Model**（`model_temporal.py`）：Transformer encoder-decoder；encoder 处理先前预测 x_p，decoder 取 x_k 并对 encoder 做cross-attention
  - 静态与时序输出合并、由 gating 分数调制后施加到源动作上，得到最终风格化姿态
  - 表征类别：**连续（回归/Transformer 序列到序列）**；**无扩散、无量化**
  - **运行时控制**：姿态强度（static posture weight）、时间夸张度（dynamic intensity）、**逐身体区域风格**（body-part masks，`body_parts.py`）、contact-preservation optimization（`stylizer_inference.py` 内的 contact-aware optimization，GUI 中的 Postprocessing 滑杆用于修正脚滑/穿地）
  - 损失（`motion_loss.py`）：重建损失 + contact-aware 损失
  - 数据格式：BVH（source/target 配对 clip；可选 sourceStatic/targetStatic 由 `extract_average_pose.py` 抽取）
- **关键结果**：
  - 训练时间 **约 2 分钟**；输入可为任意长度的中性动作序列
  - 风格示例：zombie dance、angry jump、Neutral / Angry / Clown / Happy / Zombie 等；从细微情绪变化到夸张角色原型
  - 对多样角色形状稳健（提供骨架形变与蒙皮形变的定性对比）
  - 含Runtime Control / Generalization & Ablations / **User Study** 章节；**具体定量数值未找到**
  - 发布经过处理的配对数据集以便后续研究
- **是否涉及 tokenizer**：**否**。

---

## 15. Skinned Motion Retargeting with Spatially Adaptive Interaction Guidance

- **Session**：Motion Transfer & Inbetweening（09:20–09:30）
- **作者**：Soojin Choi、Seokhyeon Hong、Chaelin Kim、Junghyun Nam、Junhyuk Jeon、Junyong Noh（KAIST Visual Media Lab）
- **Track**：Journal Track（ACM TOG）
- **链接**：
  - arXiv：https://arxiv.org/abs/2605.19355 （提交 2026-05-19）
  - 项目主页：https://suzyn.github.io/space_page/
  - DOI：10.1145/3811354
  - 代码：项目页标注"code (coming soon)"
- **核心问题**：在体型差异很大的角色间重定向动作并保留**交互语义**（自接触 self-contact、近身邻近 near-body proximity）仍然困难。近期几何感知方法依赖**静态对应**（预定义对应区域间的空间关系），当目标角色体型比例夸张时往往失效。
- **方法与表征**：
  - **Adaptive Anchor Sampling（AAS）模块**：不再用静态 anchor 定义，而是**动态把 anchor 重定位到目标角色上可达（reachable）区域**；基于 Transformer 的 anchor refinement 预测 anchor 位移，并通过**可微 soft projection** 约束平移后的 anchor 仍位于目标角色几何上；引入源角色的**位姿相关空间结构**，使适配后的 anchor 提供结构一致的交互引导
  - **Proximity-based Retargeting（PbR）模块**：以这些 anchor 为条件，用**基于图的自编码器（graph-based autoencoder）** 预测目标骨骼动作，保持源的空间构型
  - **交替优化（alternating training）**：先更新 AAS 参数、再更新 PbR 参数，消解"邻近目标既可由 anchor 重定位、也可由姿态调整满足"的歧义，使两模块各自专精并逐步精化
  - 损失含 **anchor direction loss** L_dir（对归一化相对位移向量做方向对齐以防互穿）等
  - 表征类别：**连续（图自编码器 + Transformer 回归）**；**无量化、无扩散**
- **关键结果**：
  - 定量指标：**穿透率（penetration）、接触保留的 precision / recall / accuracy**，以及**用户研究**，均优于 **MotionBuilder、SAME、R2ET、MeshRet**；在夸张角色形态下的交互保真度优势尤为明显（**具体数值未找到**）
  - 消融证明自适应 anchor、各损失项、交替优化均为关键（缺失自适应 anchor 或交替优化会导致更高穿透率与语义保真度下降）
  - 局限：可达性损失为软约束，未精确建模关节 ROM 与角色专属能力；假设源动作无几何伪影；主要依赖逐帧空间对应、未显式建模长时动力学（可能导致运动停滞或时间连续性中断）；假设骨骼结构一致；仅考虑地面接触，未建模与外部物体的交互
- **是否涉及 tokenizer**：**否**。

---

## 16. ReActor: Reinforcement Learning for Physics-Aware Motion Retargeting

- **Session**：Motion Transfer & Inbetweening（09:30–09:40）
- **作者**：David Müller、Agon Serifi、Sammy Christen、Ruben Grandia、Espen Knoop、Moritz Bächer（均为 **Disney Research, Switzerland**）
- **Track**：Journal Track（ACM SIGGRAPH 2026）
- **链接**：
  - arXiv：arXiv:2605.06593
  - 作者提供 PDF：https://www.baecher.info/assets/pdf/Mueller2026.pdf
  - 作者主页（含 paper/video/bibtex）：https://aserifi.github.io/
  - Disney Research 出版页：https://la.disneyresearch.com/publication
  - DOI：10.1145/3811378
  - 代码：**未找到**
  - CCS：Control methods；Reinforcement learning；Animation；Mathematical optimization
- **核心问题**：把人类运动学参考动作重定向到机器人形态仍极难；现有方法常产生**物理不一致**（脚滑 foot sliding、自碰撞/自穿透、悬空/地面穿透、动力学不可行动作），阻碍下游模仿学习。既有优化法需预定义接触模式、易陷局部极小、需大量手工调参；学习法需大量源-目标配对数据且多假设理想球关节，回避物理角色的复杂性。
- **方法与表征**：
  - **双层（bilevel）优化框架**：**下层**用 RL 训练 motion tracking 策略；**上层**精修由**稀疏语义刚体对应（sparse semantic rigid-body correspondences）** 定义的重定向参数，使参考动作自适应匹配机器人形态
  - 为使优化可解，**推导了上层损失的近似梯度**
  - 只需**稀疏**语义对应，**免除手工调参**；参数化表达力足以在不同 embodiment 间保留特征性运动
  - 把重定向**直接与物理仿真集成**，产出物理可信动作，便于稳健模仿学习
  - 表征类别：**优化 + RL（无生成式表征）**；**无量化**
- **关键结果**：
  - 在仿真与**真机硬件**上验证；成功迁移到与人类差异极大的形态，包括**两款人形机器人与一款四足机器人**（不同 DoF、形状、尺寸、比例）
  - 消除脚滑、悬空、自穿透三类经典伪影
  - 具体定量表数值：**未找到**
- **是否涉及 tokenizer**：**否**。
- **同组相关**（供背景）：AMOR: Adaptive Character Control through Multi-Objective RL（SIGGRAPH 2025）；Robot Motion Diffusion Model（SIGGRAPH Asia 2024）；VMP: Versatile Motion Priors（SCA 2024）；Olaf（RA-L 2026）；Kamino（GPU 大规模并行多体仿真，arXiv 2026）

---

## 17. Adaptive Interpolation-Synthesis for Motion In-Betweening on Keyframe-Based Animation

- **Session**：Motion Transfer & Inbetweening（09:40–09:50）
- **作者**：Anton Raël、Julien Boucher、Antoine Lhermitte（机构：**未找到**；从"production data + Autodesk Maya 集成"推断为工业团队）
- **Track**：SIGGRAPH 2026 Conference Papers
- **链接**：
  - arXiv：https://arxiv.org/abs/2605.02742
  - DOI：10.1145/3799902.3811157
  - 项目主页 / 代码：**未找到**
- **核心问题**：motion in-betweening 是 3D 动画中最耗时且最需要艺术判断的环节（决定动作表现力与节奏），是主要产能瓶颈。近期深度方法虽在动作合成/补间上效果好，但其**数据特性、动作风格与问题设定都与专业动画流程不符**。
- **方法与表征**：
  - **Adaptive Interpolation-Synthesis (AIS) layer**：镜像动画师的创作过程，**动态平衡"学习到的插值"与"直接姿态合成"**
    - 合成路径：p_synth_t = MLP_synth(h_t)，使模型能生成与前后输入关键姿态显著不同或超出其范围的姿态
    - **Blending Gate**：β_t = σ(MLP_β(h_t)) ∈ [0,1]^D，在**帧级与每个姿态维度**上自适应决定两条路径的贡献
    - 最终姿态：p_pred_t = (1−β_t) ⊙ p_interp_t + β_t ⊙ p_synth_t
  - **Domain-Based Algorithm (DBA)** 输入关键姿态调度：生产环境中的 block keyframe 并非随机分布，而是编码运动语义意图的特定姿态；由于实际动画文件不显式存储 block keyframe，采用 Miura et al. [2014] 的关键姿态提取（在多个频率尺度上找运动速度局部极小值）作为代理
  - **调度增强**：对 DBA 选中的关键姿态做随机增强——随机添加/移除关键姿态、对关键姿态索引做全局与局部偏移，提升泛化
  - 损失：**多维 L1 重建损失**
  - 表征类别：**连续（自回归/MLP 门控回归）**；**无扩散、无量化**
- **关键结果**：
  - 在**生产数据**上达到 SOTA
  - **集成进 Autodesk Maya 后，动画师完成补间任务提速 3.5×**
  - 具体指标表数值：**未找到**
- **是否涉及 tokenizer**：**否**。

---

## 18. LayerInbetween: Occlusion-Aware Stroke Correspondence and Inbetweening with Automatic Layering

- **Session**：Motion Transfer & Inbetweening（09:50–10:00）
- **作者**：Haoran Mo、Zhongyue Guan、Yixin Hu、Zeyu Wang（机构：**未找到**）
- **Track**：SIGGRAPH 2026（官方 session 标签仅 Animation / Simulation，无 AI-ML）
- **链接**：DOI **未确认**；**arXiv / 项目主页 / 代码：未找到**
- **核心问题**：2D 手绘动画的**遮挡感知矢量笔画对应与自动补间**，并自动完成图层化（layering），以减少繁重人工。
- **官方 highlight（paperdigest）**："To reduce tedious effort, we present LayerInbetween, an occlusion-aware framework for vector stroke correspondence and automatic inbetweening."
- **方法与表征**：矢量笔画对应 + 自动分层 + 补间；具体网络与表征细节 **未找到**（无公开预印本）
- **是否涉及 tokenizer**：**未找到**（无公开方法描述；对象为 2D 矢量笔画，非 3D 动作序列）
- **纳入理由与边界说明**：属于 2D 动画补间（motion editing 范畴的 2D 对应物），非 3D 人体动作；若综述仅关注 3D 人体运动可剔除。

---

# Session：Humans & Hands - Reconstruction（sess110）

---

## 19. TwinPose: Person-Specific Subspaces for Multi-View 3D Pose Estimation

- **Session**：Humans & Hands - Reconstruction
- **作者**：Wenwu Yang、Tianyi He、Jiwei Ding、Xun Wang、Rong Zhang（Zhejiang Gongshang University）、Kun Zhou（Zhejiang University）
- **Track**：Journal Track（ACM TOG，DOI 10.1145/3811316）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："TwinPose is an observation-driven framework for real-time multi-person 3D motion capture from sparse multi-view inputs in complex scenes. By constructing instance-aware 'twin poses' that unify 2D pose semantics and multi-view geometric consistency, it enables robust, efficient, and detector-agnostic 3D human pose estimation, even under occlusions and dense interactions."
- **核心问题**：复杂场景下、稀疏多视角输入的**实时多人 3D 动捕**；需应对遮挡与密集交互，且不依赖特定 2D 检测器。
- **方法与表征**：构造 **instance-aware "twin poses"** 统一 2D 姿态语义与多视角几何一致性；**person-specific subspaces**（人物专属子空间）。表征类别：**连续子空间 / 判别式估计**，**无量化**。网络细节、子空间维度：**未找到**
- **关键结果**：实时；对遮挡与密集交互稳健；detector-agnostic。**具体数值未找到**
- **是否涉及 tokenizer**：**否**

---

## 20. AMOR: Airborne Motion Reconstruction via Homotopy-Aware Trajectory Optimization

- **Session**：Humans & Hands - Reconstruction
- **作者**：Chanha Kim*、Jungdam Won†（Seoul National University，Intelligent Motion Lab）
- **Track**：SIGGRAPH 2026 Conference Papers（42:1–42:10）
- **链接**：
  - DOI：10.1145/3799902.3811070
  - 实验室出版页（含 Project / Paper / Video）：https://sites.google.com/view/snuimo/publications
  - arXiv：**未找到**
- **官方 Description**："Existing Human-Mesh-Recovery methods struggle with dynamic airborne motions, often producing unstable and unrealistic results. We present a refinement approach that enforces physical consistency by selecting reliable motion segments, applying homotopy-aware trajectory optimization..."
- **核心问题**：现有 HMR方法在**空中动态动作**（腾空、特技类）上不稳定、不真实。
- **方法与表征**：**精修（refinement）方法**——先**选择可靠的运动片段**，再应用**homotopy-aware 轨迹优化**以强制物理一致性。表征类别：**轨迹优化**，**无生成式表征、无量化**。
- **关键结果**：在 in-the-wild 与挑战性视频上真实性与稳定性优于先前方法。**具体数值未找到**
- **是否涉及 tokenizer**：**否**

---

## 21. FLASHand: Feed-forward reLightable and Animatable Single-view Hand Reconstruction

- **Session**：Humans & Hands - Reconstruction
- **作者**：Ling-Xiao Zhang（ICT, CAS；UCAS）、Lin Gao（ICT, CAS；UCAS）、Wei-Hong He（South China University of Technology）、Yu-Xuan Yang（East China Normal University）、Yunbing Xing（ICT, CAS）、Yu-Kun Lai（Cardiff University）、Yiqiang Chen（ICT, CAS）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811039）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："We propose FLASHand, the first feed-forward model to reconstruct high-fidelity, relightable, and animatable 3D hand avatars from a single RGB image. By leveraging NIMBLE priors and mesh-based 2D Gaussian Splatting, FLASHand achieves instant, personalized reconstruction with disentangled appearance, supporting real-time animation and relighting."
- **核心问题**：从单张 RGB 图像即时重建可重光照、可驱动的高保真 3D 手部 avatar。
- **方法与表征**：**前馈**模型；**NIMBLE 手部先验** + **mesh-based 2D Gaussian Splatting**；外观解耦。表征类别：**连续（前馈回归 + 高斯基元）**，**无量化**。
- **关键结果**：即时、个性化重建；支持实时动画与重光照。**具体数值未找到**
- **是否涉及 tokenizer**：**否**
- **纳入理由**：手部动作/驱动（animatable hand avatar），符合"面部与手部动作"范畴；但核心偏重建与外观。

---

## 22. AGILE: Hand-object Interaction Reconstruction from Video via Agentic Generation

- **Session**：Humans & Hands - Reconstruction
- **作者**：Jin-Chuan Shi、Binhong Ye、Tao Liu、Xiaoyang Liu、Yangjinhui Xu、Junzhe He、Zeju Li、Hao Chen（Zhejiang University）、Chunhua Shen（Zhejiang University；Zhejiang University of Technology）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811134）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："AGILE reconstructs 3D hand-object interactions from monocular videos via agentic generation. It produces complete, simulation-ready object meshes and robustly estimates object pose without SfM..."
- **核心问题**：既有方法两大障碍：① 依赖 neural rendering，在重遮挡下产出碎片化、**非 simulation-ready** 的几何 ② 依赖脆弱的 **SfM 初始化**，在 in-the-wild 素材上频繁失败。
- **方法与表征**：范式从"重建"转向 **agentic generation**（用于交互学习）；产出完整、可仿真的物体 mesh；**无需 SfM** 稳健估计物体位姿。表征类别：**生成式 3D + agent 流程**；**无动作量化**。
- **关键结果**：在挑战性真实数据上结果准确稳定。**具体数值未找到**
- **是否涉及 tokenizer**：**否**

---

## 23. EgoForce: Forearm-Guided Camera-Space 3D Hand Pose from a Monocular Egocentric Camera

- **Session**：Humans & Hands - Reconstruction
- **作者**：Christen Millerdurai（DFKI；Max Planck Institute for Informatics）、Shaoxiang Wang（DFKI）、Yaxu Xie（DFKI）、Vladislav Golyanik（MPI for Informatics）、Didier Stricker（DFKI）、Alain Pagani（DFKI）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811047）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："EgoForce enables real-time 3D hand pose, shape, and absolute position recovery from a single head-mounted RGB camera. Designed for lightweight smart glasses, it uses forearm cues and camera-aware ray-space lifting..."
- **核心问题**：以往手部姿态估计"只看手"，只能输出以手腕为原点的相对坐标，无法给出手在相机前的**绝对位置**；单目存在**深度-尺度歧义**；AR/VR 头显相机视角极宽、鱼眼畸变强，且相机型号多样（透视/鱼眼/各种畸变参数）。
- **方法与表征**：利用**前臂线索（forearm cues）** + **camera-aware ray-space lifting**，在**相机空间**恢复 3D 手部姿态、形状与绝对位置；跨不同相机模型与设备配置。表征类别：**判别式回归**，**无量化**。
- **关键结果**：实时；跨多种相机模型与设备设置的精确跟踪。**具体数值未找到**
- **是否涉及 tokenizer**：**否**

---

# Session：Faces & Avatars（sess109）

---

## 24. EchoAvatar: Real-time Generative Avatar Animation from Audio Streams

- **Session**：Faces & Avatars
- **作者**：Bohong Chen、Yumeng Li、Yinglin Xu、Youyi Zheng、Yanlin Weng、Kun Zhou（通讯）（均为 State Key Lab of CAD&CG, Zhejiang University）
- **Track**：SIGGRAPH 2026 Conference Papers
- **链接**：
  - arXiv：https://arxiv.org/abs/2605.28272 （ar5iv HTML：https://ar5iv.labs.arxiv.org/html/2605.28272 ，提交 2026-05-27）
  - **项目主页（代码 + 预训练模型 + 视频均已发布）**：https://robinwitch.github.io/EchoAvatar-Page
  - DOI：10.1145/3799902.3811066
  - 资助：NSFC 62572430/ 62421003；XPLORER PRIZE
- **官方 Description**："We introduce a novel framework designed to generate continuous, coherent full-body motion from arbitrary audio streams with low latency... serves as a plug-and-play solution for transforming voice agents into interactive humanoid avatars."
- **核心问题**：既有音频驱动动作合成多为**离线处理完整音频**或**限定单一域**，很少能同时处理语音与音乐；交互式avatar 需要低延迟的流式全身动作生成。
- **方法与表征（tokenizer 细节丰富）**：
  - **量化：RVQ（残差向量量化）**
    - **量化层数（残差深度）Q = 6**；**每层码本大小 K = 512**；**latent 维度 d = 512**
    - **时间下采样率 4×** → **latent token 帧率 7.5 Hz**（30 FPS ÷ 4）
    - **码本按解剖学分区**：**上半身 / 下半身 / 手部，共 3 套独立码本**
    - Commitment 权重 **η = 0.1**；训练 batch 256、lr 4e-4、step decay；训练窗口随机采样 **64 帧**
  - **因果注意力 tokenizer**：用堆叠attention block 替代卷积主干，causal mask 把感受野限制在前 p 帧（**p 数值未找到**）。动机：CausalConv（如 MotionStreamer 用法）表达能力有限、重建有视觉伪影
  - **双路径（dual-path）时间重采样**（借鉴 DC-AE）：下采样= temporal pooling 分支 + feature concatenation 分支（经 MLP）；上采样 = temporal replication + channel-expansion MLP
  - **FK 辅助损失**：把Forward Kinematics 接入优化回路，对全局关节位置/速度/加速度/足部接触一致性施加损失以抑制脚滑
  - 姿态参数化：root velocity + height + **6D 关节旋转**
  - 总损失：L1 重建 + η·Σ_q ‖z^q − sg[ẑ^q]‖² + Φ(FK(m̂), FK(m))
  - **生成主干**：**Qwen2.5-0.5B-Instruct**（解绑 tied embedding 以适配模态专属词表）；上下文窗口 **4 秒** = 音频 token **600 个**（Causal EnCodec 前 2 层 RVQ，75 Hz）+ motion token **540 个**（6 层 RVQ × 3 身体分区，按 MusicGen 的 flattened interleaving 序列化）；微调batch 256、lr 5e-5
  - **Hierarchical Loss Scaling**：对更深 RVQ 层的交叉熵施加单调递减权重（优先结构一致性）
  - **Hierarchical Token Corruption**（解决 conditional collapse——自回归 motion prior 短路较弱音频条件）：对被选中时间步采样层深 ℓ_t ~ U(1, L)，随机化第 ℓ_t 至 L 层 token，**保留粗粒度底层不变**。双重收益：模拟真实生成伪影作数据增强；赋予长序列自回归的纠错能力
  - **Example Control**：沿用 MECo，但**仅从参考序列的第 1 级码本**抽取控制 token（语义密度集中在 primary VQ 层）
  - **三阶段课程**：Embedding Space Alignment → Acoustic-Kinematic Alignment → Exemplar-Driven Control
  - **流式**：原生**30 FPS**（渲染插值到 60 FPS）；chunk **8 帧 = 0.266秒** 滑动窗口；吞吐约 **300 tokens/s**；4 阶段（Audio Encoding / Motion Synthesis / Motion Decoding / IK 后处理）总计算延迟远低于 266 ms 的 chunk 时长（H200 与 RTX 4090 均成立；alphaXiv 概述称 H200 上总延迟约 **177 ms**，客户端加 **100 ms** buffer 平滑抖动）；CUDA Graph instantiation 降低 kernel 调度开销
  - 训练硬件：2× H200，完整流水线约 **30 小时**
  - **tool-call 接口**：让上游 LLM 注入显式语义动作，与隐式音频驱动动作交织（反应式动画 → 意图驱动行为）
  - 面部与身体解耦：面部走 ARKit blendshape + 轻量流式 speech-to-face（follow DyStream）
- **数据集**：
  - **ZeroEGGS**：约 **2 小时**，单说话人，**19 种表达风格**（语音-手势）
  - **Motorica**：约 **6 小时**，**5 位表演者、8 种舞种**（音乐-舞蹈）；手指数据用 motion inpainting 模型（以 ZeroEGGS 高保真手指为先验）修复 + Savitzky-Golay 去抖
  - **BEAT2**：对比基准
  - RL 语料：语音约 1 小时（Gemini 3 Pro 生成脚本 + ElevenLabs TTS）；音乐 100 首（YouTube，多速度多流派）
  - 面部：约 1 小时自采（Apple ARKit blendshape，iPhone 12 + Live Link Face，朗读 Harvard Sentences）
  - 骨架：Holden的 zeroeggs-retarget / motorica-retarget 标准骨架，再用 Maya 重定向到统一角色
- **关键结果**：
  - **重建/消融（Table 1）**（MPJPE 单位 1e-4 m；Trans Loss 为每帧 root 平移速度误差）：
    | 方法 | 重建 FID↓ | MPJPE↓ | Trans Loss↓ | 生成 FID↓ |
    |---|---|---|---|---|
    | Real motion | 0 | 0 | 0 | 0 |
    | CausalConv-RVQ | 9.208 | 525.6 | 12.94 | 18.06 |
    | Attn | 4.183 | 411.6 | 12.53 | 12.25 |
    | Attn (w/o dual) | 12.21 | 982.4 | 48.77 | 18.55 |
    | Attn (w/o auxiliary) | 8.775 | 778.5 | 55.67 | 17.68 |
    | Attn (w/o lookback) | 6.612 | 468.8 | 12.89 | 15.53 |
    | **Attn (w/ bodypart)** | **1.306** | **184.1** | **9.637** | **9.465** |
  - 自建测试集（Table 2，BA×1e-1）：GT FID 0 / Div 21.52 / BA_G 7.775 / BA_D 2.619；MECo 14.73 / 23.13 / 7.507 / 2.622；EDGE 18.06 / 19.71 / 8.190 / 2.668；**Ours 9.465 / 20.70 / 8.277 / 2.603**；Ours(DPO) 12.39；Ours(GRPO) 24.13；Ours(w/o corrupt) 25.92（其高 Diversity/BA_G 为病态假高——静音时仍持续输出无意义舞蹈）
  - **BEAT2 基准（Table 4，FID×1e-1）：Ours FID 2.874（新 SOTA）**，BA_G 7.342、Div 13.53；对照MECo 3.401 / 7.346 / 15.30；EMAGE 5.512；ViBES 5.257；PersonaGesture 3.930
  - 主观（Table 3 摘要，成对比较，5点 Likert 映射到[-2,2]，共 **1,680 次成对判断**）：舞蹈 Ours 0.244/0.337/0.267 vs EDGE 0.148/−0.136/0.099 vs MECo −0.508/−0.277/−0.477；手势 Ours 0.588/0.798/0.816 vs MECo 0.235/0.061/0.096 vs EDGE −0.676/−0.705/−0.748
  - **RL 对齐**：GRPO（β_G=0.01，每 prompt 30 rollout；奖励一= 自监督动作质量奖励，用不同强度 corruption 降质 GT 构造已知退化等级、FID 校准，奖励模型镜像 tokenizer 但改双向注意力且去掉量化层，与 GT 退化等级 **Pearson 0.9977**；奖励二 = InfoNCE 音频-动作对齐，音频编码器为预训练 BEATs，检索成功率约随机的 **100×**）；DPO（β_D=0.1，Best-of-N，**N=8**，人工比较标最优/最差，过滤差异不明显的对）
  - **对齐-保真度权衡**：两种 RL 均使 FID 退化（GRPO 9.465→24.13 更剧烈；DPO 9.465→12.39），但人类偏好提升（mode-seeking 导致分布覆盖度下降）。领域适配结论：舞蹈适配 GRPO，对话手势适配 DPO
  - 数据组成消融：Merged 显著优于单域（手势的 beat matching 提升最明显，+0.727）
- **是否涉及 tokenizer**：**是（RVQ 细节最完整的一篇）**。Q=6 层、每层 K=512、d=512、4× 时间下采样（token 7.5 Hz）、按上/下身/手部三分区码本、因果注意力 tokenizer + 双路径重采样 + FK 辅助损失；重建 FID 1.306 / MPJPE 184.1（1e-4 m）。
- **信息缺口**：因果掩码回溯帧数 p、每层码本是否异构、tokenizer 各损失项权重（除 η）、逐阶段延迟 ms 数值（在附录）。

---

## 25. Topologically Consistent Multi-view 3D Head Reconstruction via Coarse-Guided Layered Surface Sampling（SHELLS）

- **Session**：Faces & Avatars
- **作者**：Timo Bolkart、Daoye Wang、Prashanth Chandran（Google）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811201）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："From calibrated multi-view images, SHELLS reconstructs 18k-vertex 3D heads in 0.08 seconds. It aggregates DinoV2 features via projective surface-aware feature sampling, allowing a transformer to predict dense semantic meshes 3.5x faster with 88% less GPU memory than state-of-the-art methods."
- **核心问题**：从标定多视角图像做拓扑一致的稠密 3D 头部重建，同时兼顾速度与显存。
- **方法与表征**：**projective surface-aware feature sampling** 聚合 **DINOv2** 特征 → Transformer 预测稠密语义 mesh；**coarse-guided layered surface sampling**。表征类别：**连续前馈回归**，**无量化**。
- **关键结果**：**18k 顶点 3D 头部、0.08 秒**；比 SOTA **快 3.5×**、**GPU 显存少 88%**。
- **是否涉及 tokenizer**：**否**
- **纳入理由与边界说明**：属"面部"但核心是**静态几何重建**，非面部动作生成。若综述严格限定"motion"，可剔除或仅作背景。

---

## 26. Learning a Delighting Prior for Facial Appearance Capture in the Wild

- **Session**：Faces & Avatars
- **作者**：Yuxuan Han、Xin Ming、Tianxiao Li、Zhuofan Shen（Tsinghua University）、Qixuan Zhang（ShanghaiTech University）、Lan Xu（ShanghaiTech University）、Feng Xu（Tsinghua University）
- **Track**：Journal Track（ACM TOG，DOI 10.1145/3811303）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："A high-quality facial appearance capture method from casual smartphone videos with a powerful delighting prior."
- **是否涉及 tokenizer**：**未找到**
- **边界说明**：核心为**面部外观/材质捕获（delighting）**，非面部动作。**建议剔除**，此处仅登记以示穷尽。

---

## 27. See-through: Single-image Layer Decomposition for Anime Characters

- **Session**：Faces & Avatars
- **作者**：Jian Lin、Chengze Li（Saint Francis University）、Haoyun Qin（University of Pennsylvania；Spellbrush / Shitagaki Lab）、Kwun Wang Chan（Saint Francis University）、Yanghua Jin（Spellbrush）、Hanyuan Liu（Saint Francis University）、Chun Wang, Stephen Choy（Saint Francis University）、Xueting Liu（Saint Francis University）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811209）
- **官方 Description**："...automates the transformation of static anime illustrations into manipulatable 2.5D models... decomposes a single image into fully inpainted, semantically distinct layers with inferred drawing orders — up to 19 layers including hair, face, eyes, clothing, accessories, and more."
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **边界说明**：产出**可操控的 2.5D 模型**（"manipulatable"，与角色动画/驱动相关），但本身是图像分层与补全。**建议列为边缘相关**。
- **是否涉及 tokenizer**：**未找到**

---

# Session：4D Humans（sess111）

---

## 28. 4D Human-Scene Reconstruction from Low-Overlap Captures（StudioRecon）

- **Session**：4D Humans
- **作者**：Minhyuk Hwang、Sangmin Kim、Seunguk Do、Daneul Kim、Jaesik Park（机构：**未找到**；多方报道指为首尔国立大学工作）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811165）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 highlight（paperdigest）**："Video diffusion models have emerged as another option, but they show geometrically inconsistent results for humans. To address these limitations, we propose **StudioRecon**, a pipeline that reconstructs 4D human scenes from sparse, low-overlap cameras by **decoupling background and humans**."
- **核心问题**：用少量、**低重叠**相机画面重建可自由探索的动态（4D）人-场景空间；视频扩散模型方案对人体几何不一致。
- **方法与表征**：**解耦背景与人体**的重建流水线；具体表征细节 **未找到**
- **是否涉及 tokenizer**：**未找到**（据highlight 为重建流水线，无量化描述）
- **纳入理由**：4D 人体（动态人-场景），属"4D/动态网格动画"扩展；核心为重建而非动作生成。

---

## 29. Implicit Surface Compression — with Good Old Discrete Cosine Transform and Motion Compensation

- **Session**：4D Humans
- **Track**：Journal Track（DOI 10.1145/3811298）
- **链接**：**未找到**
- **边界说明**：使用 **DCT + motion compensation** 做隐式曲面压缩，"motion" 指视频编码意义上的运动补偿，**不是人体动作**。**建议剔除**，此处仅登记。

---

## 30. ATGS: Anchored Temporal Gaussian Splatting for Long Volumetric Video Representation

- **Session**：4D Humans；**Track**：Journal Track（DOI 10.1145/3811306）
- **边界说明**：长体积视频表示（4D 高斯），核心为表示/压缩。**建议剔除或列为边缘相关**。

## 31. VFAvatar: Feed-Forward 3D Avatar Reconstruction from Casual Image Collections

- **Session**：4D Humans；**Track**：Conference Papers（DOI 10.1145/3799902.3811171）
- **边界说明**：Avatar 静态重建，非动作。**建议列为边缘相关**。

## 32. High-Fidelity 4D Cloth Capture Pipeline with a Two-Level Pattern

- **Session**：4D Humans；**Track**：Journal Track（DOI 10.1145/3811305）
- **边界说明**：4D 服装捕获，属动态捕获但对象是布料。**建议列为边缘相关**。

---

# Session：Digital Humans & Virtual Try-On（sess152）

---

## 33. DreamActor-M2: Universal Character Image Animation via Spatiotemporal In-Context Learning

- **Session**：Digital Humans & Virtual Try-On
- **作者**：Mingshuang Luo（ByteDance；ICT, CAS）、Shuang Liang（ByteDance）、Yuxuan Luo（ByteDance）、Zhengkun Rong（ByteDance）、Tianshu Hu（ByteDance）、Ruibing Hou（ICT, CAS）、Hong Chang（ICT, CAS）、Yong Li（Southeast University）、Yuan Zhang（ByteDance）、Mingyuan Gao（ByteDance）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811114）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："...a universal character animation framework that regards motion conditioning as spatiotemporal in-context learning. It leverages video foundation model priors and enables end-to-end pose-free motion transfer from raw videos. Without explicit pose estimation, it achieves strong generalization and high-fidelity animation across complex scenes."
- **核心问题**：角色图像动画中，如何摆脱对显式姿态估计（pose/skeleton 条件）的依赖，实现从原始视频端到端的 **pose-free 动作迁移**。
- **方法与表征**：把 **motion conditioning 视为时空 in-context learning**；利用**视频基础模型先验**；**无需显式姿态估计**。表征类别：**视频扩散/基础模型的连续 latent**，**无动作量化**。层数/参数/数据集：**未找到**
- **关键结果**：跨复杂场景的强泛化与高保真动画。**具体数值未找到**
- **是否涉及 tokenizer**：**否**（视频 latent；未提及动作量化）

---

## 34. MACE-Dance: Motion-Appearance Cascaded Experts for Music-Driven Dance Video Generation

- **Session**：Digital Humans & Virtual Try-On
- **作者**：Kaixing Yang（Renmin University of China）、Jiashu Zhu（Alibaba Group, AMAP）、Xulong Tang（Malou Tech Inc）、Ziqiao Peng（Renmin University of China）、Xiangyue Zhang（Wuhan University）、Puwei Wang（Renmin University of China）、Jiahong Wu（Alibaba Group, AMAP）、Xiangxiang Chu（Alibaba Group, AMAP）、Hongyan Liu（Tsinghua University）、Jun He（Renmin University of China）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811202）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："MACE-Dance generates dance videos from music using cascaded motion and appearance experts. It first produces expressive 3D dance motion from music, then animates a reference image into a temporally coherent video. The paper also introduces MA-Data, a large-scale dataset and evaluation protocol for this task."
- **核心问题**：音乐驱动的舞蹈**视频**生成，需同时保证动作表现力与外观时间一致性。
- **方法与表征**：**级联 Mixture-of-Experts（MoE）**——先由motion expert 从音乐生成富有表现力的 **3D 舞蹈动作**，再由 appearance expert 把参考图像动画化为时间一致的视频。表征类别：**未找到具体（3D 动作阶段的表征类型未公开）**；是否量化：**未找到**
- **数据与结果**：提出 **MA-Data** 大规模数据集与评测协议（规模数字**未找到**）；**定量结果未找到**
- **是否涉及 tokenizer**：**未找到**（需查 arXiv/正文确认 3D 舞蹈动作阶段是否用 VQ 类表征——该子领域常用 VQ-VAE，但本文无公开证据，故不做推断）

---

## 35. Composing People Together: Iterative Pose-Image Generation for Multi-Person Interaction Scenes

- **Session**：Digital Humans & Virtual Try-On
- **作者**：Wenxuan Peng、Bharath Hariharan、Hadar Elor（Cornell University）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811129）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："...generating realistic multi-person interaction scenes from text. By modeling human pose and appearance together and composing scenes step by step, our approach produces images that better capture complex interactions, with improved accuracy and greater diversity compared to existing text-to-image models."
- **方法与表征**：**联合建模人体姿态与外观**、**迭代逐步组合**场景。表征类别：**图像扩散 + 姿态条件**，**无动作序列量化**。
- **边界说明**：产出为**静态图像**（多人交互姿态），非动作序列。**建议列为边缘相关**（姿态生成层面与motion 相关）。
- **是否涉及 tokenizer**：**否**

---

## 36. HumanFlow: Controllable Human Image Generation via Flow Matching

- **Session**：Digital Humans & Virtual Try-On
- **作者**：Wenzhuo Fan、Hongsheng Zheng、Jianchi Sun（Wuhan University）、Fei Fang（Wuhan Textile University）、Hong Ding（Guangxi University of Finance and Economic）、Chunxia Xiao（Wuhan University）
- **Track**：Journal Track（ACM TOG，DOI 10.1145/3811361）
- **官方 Description**："HumanFlow is a unified flow-matching framework for controllable full-body human image generation. It introduces a Control Encoder, Token-ControlNet, and topology-aware loss for structural consistency, along with the large-scale MiCoGen dataset..."
- **方法与表征**：**flow matching**；Control Encoder + **Token-ControlNet** + 拓扑感知损失。注意："Token-ControlNet" 中的 token 指 DiT 的图像 patch token，**不是运动离散码本**。
- **数据**：**MiCoGen** 大规模数据集（规模**未找到**）
- **边界说明**：**静态人体图像生成**。**建议剔除或列为边缘相关**。
- **是否涉及 tokenizer**：**否**（无运动量化）

---

## 37. FabricTryOn: Taming Image Editing Models for Garment Re-Texturing

- **Session**：Digital Humans & Virtual Try-On；DOI 10.1145/3799902.3811105
- **边界说明**：服装重贴图，纯图像编辑。**建议剔除**，仅登记。

---

# Session：Learning-adjacent / 其他 session 中的 motion 相关论文

---

## 38. AniGen: Unified S³ Fields for Animatable 3D Asset Generation

- **Session**：3D Generation（sess121）
- **作者**：Yi-Hua Huang*（The University of Hong Kong）、Zi-Xin Zou（VAST）、Yuting He（The Chinese University of Hong Kong）、Chirui Chang（HKU）、Cheng-Feng Pu（Tsinghua University）、Ziyi Yang（HKU）、Yuan-Chen Guo（VAST）、Yan-Pei Cao†（VAST）、Xiaojuan Qi†（HKU）
- **Track**：Journal Track（ACM TOG 45(4)）
- **链接**：
  - arXiv：https://arxiv.org/abs/2604.08746 （HTML：https://arxiv.org/html/2604.08746，v2 2026-04-14）
  -项目主页：https://yihua7.github.io/AniGen_web/
  - **代码（已开源）**：https://github.com/VAST-AI-Research/AniGen ；在线试玩 https://huggingface.co/spaces/VAST-AI/AniGen
  - DOI：10.1145/3811297
- **官方 Description**："AniGen generates animatable 3D assets from a single image by jointly producing geometry, skeletons, and skinning weights. Its unified S³ field representation enables consistent, high-quality rigged asset creation across diverse categories..."
- **核心问题**：3D 生成模型能合成视觉合理的形状但结果是**静态**；事后 auto-rigging脆弱且常产生与生成几何**拓扑不一致**的骨架。
- **方法与表征**：
  - **S³ Fields（Shape, Skeleton, Skin）**：把形状、骨架、蒙皮统一为定义在**共享空间域**上的互相一致的**连续场**——骨架**不表示为离散图**而是**稠密向量场**，蒙皮**不表示为稀疏矩阵**而是**对偶特征场**
  - **confidence-decaying skeleton field**：显式处理 Voronoi 边界附近骨骼预测的几何歧义，在 AE 训练中下调歧义区域权重
  - **Dual Skin Field +预训练 SkinAE**：把可变基数的skinning 转为**固定维度 latent 特征空间**，使固定架构网络能预测任意复杂度的rig（关节数无关）
  - **两阶段 flow matching**（基于 TRELLIS 的structured sparse latents）：先生成稀疏结构脚手架，再在结构化latent 空间生成稠密几何与 articulation
  - 表征类别：**连续场+ structured latent flow matching**；**无离散量化**
- **关键结果**：
  - 在 **ArticulationXL** 上与 TRELLIS*+UniRig / Anymate / Puppeteer / RigAnything 等强基线系统对比，在**骨架结构预测与蒙皮精度**上均取得最佳，尤其在**Gromov-Wasserstein 距离**（骨架拓扑正确性）与 **Skin KL** 上领先明显（**具体数值未找到**）
  - 泛化到in-the-wild 单图，覆盖动物、人形、卡通角色、植物、机械臂等；联合建模几何与 articulation 不损害几何保真度（相对纯几何生成器）
- **是否涉及 tokenizer**：**否**（连续场 + flow matching）。**对照：同组/同数据线的 SkinTokens 用 FSQ-CVAE 离散化 skinning，可作为"离散 vs 连续结构表征"的直接对比案例。**
- **纳入理由**：animate-ready 资产生成（rig + skinning），是"动作可驱动性"的上游关键环节。

---

## 39. SimArt: Decomposing Monolithic Meshes into Sim-ready Articulated Assets via MLLM

- **Session**：Generative 3D (2)（sess112）
- **作者**：Chuanrui Zhang（Nanyang Technological University）、Minghan Qin（ByteDance Seed）、Yuang Wang（ByteDance Seed）、Baifeng Xie（ByteDance Seed）、Hang Li（ByteDance Seed）、Ziwei Wang（Nanyang Technological University）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811173）
- **链接**：arXiv / 项目主页 / 代码：**未找到**
- **官方 Description**："SIMART is a unified multimodal large language model framework that converts static 3D meshes into simulation-ready articulated assets. **By jointly modeling part decomposition and kinematics with a sparse 3D VQ-VAE**, it reduces token complexity and enables scalable, high-fidelity object generation for robotics and physics-based simulation."
- **核心问题**：把整体（monolithic）静态 mesh 分解为可仿真的**铰接式（articulated）** 资产。
- **方法与表征**：**MLLM 统一框架 + 稀疏 3D VQ-VAE** 联合建模部件分解与运动学，以降低 token 复杂度。表征类别：**离散 token（sparse 3D VQ-VAE）**
- **量化细节（码本大小、稀疏体素分辨率、token 数）**：**未找到**
- **是否涉及 tokenizer**：**是（但对象是"物体部件/运动学"，不是人体动作序列）**。对研究 motion tokenizer 的取舍（如何用稀疏 VQ 降低 token 复杂度）有参照价值。
- **纳入理由**：articulation/kinematics 生成，属"运动可驱动性"上游；非人体动作。

---

## 40. Go-with-the-Track: Video Compositing and Motion Control with Point Tracking

- **Session**：Video Gen & Camera Control（sess150）
- **作者**：Koichi Namekata（Eyeline Labs；University of Oxford）、Yash Kant（Eyeline Labs；Netflix）、Zhizheng Liu（Eyeline Labs；UCLA）、Ryan Burgert（Eyeline Labs；Stony Brook）、Yuancheng Xu（Eyeline Labs；Netflix）、Kuan Heng Lin（Eyeline Labs；Columbia）、Emmett Steven（Netflix）、Julien Philip（Eyeline Labs）、Li Ma（Eyeline Labs）、Andrea Vedaldi（Oxford）、Paul Debevec（Eyeline Labs；Netflix）、Ning Yu（Eyeline Labs；Netflix）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811093）
- **官方 Description**："...a video generation framework conditioned on multiple reference images and point-tracks that jointly anchor the generated frames and the references. Our approach enables precise control over motion and multi-reference compositing, unlocking... video restylization, keypoint- and mesh-driven compositing, and camera motion control."
- **方法与表征**：**point tracks** 作为运动条件，多参考图像联合锚定；**视频扩散 latent**，无动作量化
- **边界说明**：运动控制针对**视频像素/点轨迹**，支持 **keypoint- 与 mesh-driven compositing**。**建议列为边缘相关**（若综述含视频驱动动画）。
- **是否涉及 tokenizer**：**否**

---

## 41. ActCam: Zero-Shot Joint Camera and 3D Motion Control for Video Generation

- **Session**：Video Generation（sess156）
- **作者**：Omar El Khalifi、Thomas Rossi、Oscar Fossey、Thibault Fouque（Kinetix）、Ulysse Mizrahi（Kinetix；Tel Aviv University）、Philip Torr（University of Oxford）、Ivan Laptev（MBZUAI）、Fabio Pizzati（MBZUAI）、Baptiste Bellot-Gurlet（Kinetix）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811224）
- **官方 Description**："ActCam is a zero-shot method providing joint control over camera trajectories and character acting. It integrates into preexisting artistic workflows by leveraging traditional cinematography skills for fine-grained control. Requiring no additional training, the framework avoids costly finetuning and adapts across different backbone architectures."
- **方法与表征**：**zero-shot、免训练**，联合控制相机轨迹与**角色表演（character acting，3D motion）**；可跨不同 backbone 架构。表征类别：**视频扩散 latent 上的推理时控制**，**无量化**
- **纳入理由**：明确含**3D motion control** 与角色表演控制，作者单位 Kinetix 是动作技术公司。属"视频驱动/动作控制"。
- **是否涉及 tokenizer**：**否**

---

# Session：其他含"motion 语义"但应剔除的条目（登记以示穷尽）

以下论文标题或摘要含"motion / animation"，但核心不属于人体动作/角色动画范畴，**建议从综述中剔除**：

| 论文 | Session | 剔除理由 |
|---|---|---|
| Implicit Surface Compression — with DCT and Motion Compensation | 4D Humans | motion = 视频编码的运动补偿 |
| SmoothMotionVectors: Optimizing Your Content for Video Codecs in Free View Video Compression | Dynamic Scenes |视频编码 motion vector |
| Computational Design of Coordinate-Motion Assemblies | Computational Design | 机械装配体的坐标运动，非角色 |
| Kinematic Kitbashing | Computational Design | 铰接式 3D 物体合成（运动学图组装部件），物体非角色；与 SimArt/AniGen 同属articulation 上游，可作边缘参考 |
| mpcGear: Multi-Point Conjugation Gear Mechanisms | Computational Design | 齿轮机构 |
| Computational Design of Terrestrial Robots with Anisotropic Friction | Computational Design | 机器人本体设计（含运动），非动作生成；若综述含"机器人运动"可列边缘 |
| Sketch2Arti: Sketch-based Articulation Modeling of CAD Objects | Vectors Graphics, Strokes & Sketches | CAD 物体关节建模 |
| LoBoFit: Flexible Garment Refitting via Local Bone Mapping Blending | Garments | 服装重适配（用骨骼映射），与蒙皮相关但对象是服装；可列边缘 |
| Progressing Level-of-Detail Animation for Volumetric Elastodynamics | Simulation | 体积弹性动力学 LOD，非角色动作 |
| MPM Lite / Mixed MPM / M-ABD / Heterogeneous Subspace Corrections / Distributed ABD 等 | Simulation / Affine Bodies | 通用物理求解器；可作为角色次级动力学的底层，但非 motion |
| Reality Check: How Avatar and Face Representation Affect the Perceptual Evaluation of Synthesized Gestures | Perception | **注意：这是关于"合成手势"的感知评估研究**，与动作生成评测方法学直接相关，**建议保留为边缘相关**（见下条独立登记） |
| StyleID: A Perception-Aware Dataset and Metric for Stylization-Agnostic Facial Identity Recognition | Perception | 面部身份识别，非动作 |
| Role-Aware Virtual Agents for Navigational Interaction Guided by a Multimodal LLM | VR, Haptics & Hardware | 虚拟 agent 导航交互（含行为），偏 HCI；可列边缘 |
| EgoRelight / BodyReLux / Pixel Cube / Generalizable & Relightable GS for Human NVS | VR-Haptics / Film-VFX / GS | 人体重光照/新视角合成，非动作 |
| PEAR: Pixel-aligned Expressive HumAn Mesh Recovery | （session 未确认） | HMR，单帧人体网格恢复；若综述含 mocap 可列边缘（**session 与作者未找到**） |
| MOCHI/AMOR 之外的 HMR 类 | — | 见上 |

**独立登记（边缘相关，建议保留）：**

## 42. Reality Check: How Avatar and Face Representation Affect the Perceptual Evaluation of Synthesized Gestures

- **Session**：Perception（sess143）
- **Track**：SIGGRAPH 2026 Conference Papers（DOI 10.1145/3799902.3811161）
- **作者**：Haoyang Du、Yinghan Xu、John Dingliana、Brian Keegan、Rachel McDonnell、Cathy Ennis（机构：**未找到**；从作者构成看为 Trinity College Dublin / TU Dublin 方向）
- **官方 highlight（paperdigest）**："Prior work suggests these visual choices can bias motion judgments, but controlled evidence remains limited. We address this gap with **controlled evaluations of co-speech gestures across motion sources, spanning seven representative avatar renderings** used in contemporary research and application pipelines."
- **核心问题**：**合成手势（synthesized co-speech gestures）** 的感知评估中，avatar 与面部表征方式如何**偏置**动作判断——直接关系到动作生成领域**用户研究方法学的有效性**。
- **方法与结果**：跨多种 motion source、**7 种代表性 avatar 渲染方式**的受控评估；具体统计结果 **未找到**
- **是否涉及 tokenizer**：**否**
- **纳入理由**：对 motion generation 的主观评测协议设计有直接方法学意义（尤其在做 user study 时对 avatar 渲染的选择）。

## 43. PEAR: Pixel-aligned Expressive HumAn Mesh Recovery

- **Session**：**未找到**（paperdigest 列为第 87 条；DOI 10.1145/3799902.3811096）
- **作者**：Jiahao Wu、Yunfei Liu、Lijian Lin、Ye Zhu、Lei Zhu、Jingyi Li、Yu Li（机构：**未找到**）
- **官方 highlight（paperdigest）**："These issues make current approaches difficult to apply to downstream tasks. To address these challenges, we propose **PEAR—a fast and robust framework for pixel-aligned expressive human mesh recovery**."
- **核心问题**：像素对齐的**表达性人体网格恢复**（expressive HMR，通常含手部与面部），强调速度与稳健性以便下游任务使用。
- **方法与结果**：具体细节 **未找到**
- **是否涉及 tokenizer**：**未找到**
- **纳入理由**：属 motion capture / 人体姿态与形状恢复范畴，是动作数据获取的上游。

---

# 附录 A：本届 Motion 相关论文的 DOI 速查表

| 论文 | DOI | Track |
|---|---|---|
| GPC | 10.1145/3799902.3811038 | Conf |
| SMP | 10.1145/3811282 | Journal |
| MotionBricks | 10.1145/3811334 | Journal |
| Deep Motion Warping | 10.1145/3799902.3811144 | Conf |
| MultiAct | 10.1145/3799902.3811092 | Conf |
| ARDY | 10.1145/3811284 | Journal |
| R-DMesh | 10.1145/3799902.3811135 | Conf |
| TopoCap | 10.1145/3799902.3811159 | Journal |
| Stylized T2M (HyperLoRA) | 10.1145/3799902.3811205 | Conf |
| MUSIC | 10.1145/3811402 | Journal |
| MOCHI | 10.1145/3811308 | Journal |
| ACT | 10.1145/3811392 | Journal |
| Motion4Motion | 10.1145/3799902.3811062 | Conf |
| STyMo | 10.1145/3811356 | Journal |
| Skinned Motion Retargeting | 10.1145/3811354 | Journal |
| ReActor | 10.1145/3811378 | Journal |
| AIS (Motion In-Betweening) | 10.1145/3799902.3811157 | Conf |
| TwinPose | 10.1145/3811316 | Journal |
| AMOR | 10.1145/3799902.3811070 | Conf |
| FLASHand | 10.1145/3799902.3811039 | Conf |
| AGILE | 10.1145/3799902.3811134 | Conf |
| EgoForce | 10.1145/3799902.3811047 | Conf |
| EchoAvatar | 10.1145/3799902.3811066 | Conf |
| SHELLS (Head Recon) | 10.1145/3799902.3811201 | Conf |
|4D Human-Scene Recon | 10.1145/3799902.3811165 | Conf |
| DreamActor-M2 | 10.1145/3799902.3811114 | Conf |
| MACE-Dance | 10.1145/3799902.3811202 | Conf |
| Composing People Together | 10.1145/3799902.3811129 | Conf |
| HumanFlow | 10.1145/3811361 | Journal |
| AniGen | 10.1145/3811297 | Journal |
| SimArt | 10.1145/3799902.3811173 | Conf |
| Go-with-the-Track | 10.1145/3799902.3811093 | Conf |
| ActCam | 10.1145/3799902.3811224 | Conf |
| Reality Check (Gestures) | 10.1145/3799902.3811161 | Conf |
| PEAR | 10.1145/3799902.3811096 | Conf |
| See-through | 10.1145/3799902.3811209 | Conf |
| LayerInbetween | （未确认） | Conf |

---

# 附录 B：Motion Tokenizer / 离散运动表征横向对比表（重点）

## B.1 量化方案逐项对照

| 维度 | **GPC** | **MotionBricks** | **ARDY** | **EchoAvatar** | （对照，非本届主会）**SkinTokens/TokenRig** |
|---|---|---|---|---|---|
| 量化方法 | **FSQ**（look-up-free，无显式码本） | **多头 VQ-VAE**（沿特征维并行切分；FSQ 可替换，性能相近） | **FSQ**（默认；同时实现连续 AE / VAE 对比） | **RVQ**（残差量化） | **FSQ-CVAE** |
| 每维/每层级数或码本大小 | **L = 9 levels** | **每头 128–256 tokens**（推荐值） | **64 levels/特征**（"FSQ 64-128"） | **每层 K = 512** | checkpoint 名 `skin_vae_2_10_32768`（暗示 32768 容量），`..._quantization_256_token_4` |
| 潜维度 | **d = 40** | 未找到（tokenizer 卷积 1024 通道） | **L = 128**（body latent）；token 总维D = 148 | **d = 512** | 未找到 |
| 有效词表/ 容量 | 隐式9^40；分组后 **L^G = 9^5 = 59,049** | 总容量最佳区间 **1e6–1e7 tokens**（另一处建议 ~1e9，原文存张力）；消融跨 1e3–1e9 | 消融 FSQ 16-32 / 64-32 / **64-128** / 64-256 | 6 层 × 512 × 3 分区 | 未找到 |
| 量化层/头结构 | 单层，40 维 →分组 G=5 → **8 tokens/步** | **K 个并行码本（multi-head）**，非 RVQ、非按身体部位人工切分 | 单层 FSQ | **Q = 6 层残差** + **按上半身/下半身/手部 3 套码本** | 单层 FSQ，按每根骨骼独立编码 |
| 时间压缩 / token 帧率 | **未找到**（每仿真步产生 d 个 token；控制频率未公开） | **4× 下采样**；数据 **30 FPS** → token 7.5 Hz | **patch = 4 帧/token**；数据 **20 fps** → token 5 Hz | **4× 下采样**；数据 **30 FPS** → **token 7.5 Hz** | 无时间维（静态 skinning） |
| 片段/窗口长度 | **未找到** | T = **12–64 帧**（步长 4 帧随机采样） | 生成窗口 **G = 40 帧 = 2 s**（消融 4/8/12/20/40）；训练片段 1–10 s | 训练窗口 **64 帧**；流式 chunk **8 帧 = 0.266 s** | — |
| Tokenizer 架构 | Encoder 输入未来目标状态序列 ŝ_{t:t+h}（h 未找到）；Decoder 输出关节 PD 目标旋转 | **1D 卷积 U-Net**（下采样率 4，每层 3 残差卷积，1024 通道，编解码对称），**23.5M参数**；试过 Transformer 版性能相近但更慢 | **非对称条件 AE**：Enc/Dec 各 **8 层Transformer、dim 512、因果自注意力**；Decoder 条件于根运动，内部做 global→local 根转换 | **因果注意力 block堆叠**（替代 CausalConv）+ **双路径时间重采样**（DC-AE 式）+ **FK辅助损失** | **FSQ-CVAE**，VecSet 编码器 + nested dropout + importance sampling |
| 训练方式 | **端到端 RL（PPO）** 联合优化词汇与控制策略，仅 motion-tracking 奖励；STE传梯度 | 监督重建 + foot sliding loss + velocity loss；running-mean codebook update | 监督重建 + 脚滑损失（权重 0.01）；AdamAtan2 lr 2e-5，batch 128，4M steps，单卡 A100 | L1 重建 + commitment（η=0.1）+ FK 一致性；batch 256，lr 4e-4 | BCE + MSE + Dice 复合损失 |
| 上层生成模型 | **GPT 式因果 Transformer**，next-token prediction，nucleus 采样；≈60M 参数 | **Pose Module**（Transformer，dim 1024 / 16 heads / 16 layers，150M）+ **Root Module**（dim 512 / 12 heads，50M）；**masked token modeling**（cosine schedule），推理常单次前向 | **两阶段交错自回归扩散去噪器**：Root Transformer → detach → Body Transformer，各 8 层 8 头 dim 1024，总 ~156M；DDPM 10 步（4步可用） | **Qwen2.5-0.5B-Instruct**；上下文 4 s = 音频 600 token + motion 540 token；MusicGen flattened interleaving | **Qwen3-0.6B** 自回归，骨架 + SkinTokens 单序列；GRPO 后训练 |
| 重建/量化质量指标 | Bones[680h]：**成功率 99.98% / MPJPE 34.90 mm**（VQ-VAE 99.94% / 37.92；MLP 无瓶颈 99.98% / 25.56）；AMASS[40h] 99.51% / 44.43；**码本利用率 82.15%（端到端 RL）vs 76.34%（冻结编码器）；VQ-VAE 仅 10–20%** | tokenizer 重建随容量单调改善；多头显著优于单头（单头很快饱和） | 由 tokenizer 类型消融（horizon 20）：AE 128D FID 0.033 / Waypoint 0.044；VAE 128D 0.031 / 0.046；**FSQ 64-128 0.030 / 0.046**；AE 在 40 帧 horizon 训练**发散** | **重建 FID 1.306 / MPJPE 184.1（1e-4 m）/ Trans Loss 9.637**（w/ bodypart）；CausalConv-RVQ 基线 9.208 / 525.6 / 12.94 | skinning 精度提升 **98%–133%**，骨骼预测提升 **17%–22%** |
| 下游关键数字 | 扰动存活率 CoLA 68.1% vs CVAE 44.4%；CoLA 新增参数 <1% | **FID 1.054**（350k，GT 0.022）；**15,000 FPS / 2 ms**（RTX 5090）；Reach 99.6%；抖动 3.38/1.82 接近 GT 3.47/1.96；人评胜率 86.5% | Bones Rigplay **FID 0.027**；关节旋转 **2.23°**、关节位置 **2.5 cm**、关键帧 **2.3 cm**、路点 **2.4 cm**；4 步模型 **33 ms**（RTX 4090） | **BEAT2 FID 2.874（SOTA）**；生成 FID 9.465；chunk 延迟 ≈177 ms（H200）< 266 ms | — |
| 明确的"为什么选这个量化" | FSQ 免维护码本、避免 VQ-VAE 的 codebook collapse 与不稳定训练；无需 EMA / 死码重初始化 | 多头量化使潜流形更鲁棒，单 token 错误时 graceful degradation；避免单一巨型码本，也避免人工按身体部位切分 | **FSQ 训练稳定性最佳**（vanilla AE 在长 horizon 40 帧上严重不稳定甚至发散；三者性能本身相当） | RVQ 提供层级结构，配合 Hierarchical Loss Scaling 与 Hierarchical Token Corruption；分区码本大幅提升重建 | 把高维稀疏skinning 回归转为可解的 token 序列预测 |

## B.2 可直接引用的关键论断（原文级）

- **GPC**：连续 latent 的生成式控制器"prone to mode collapse and unnatural behaviors caused by drift and gaps in the latent manifold"；FSQ 相比先前 tokenized 方法"greatly simplifies the training process"。
- **MotionBricks**：多头 VQ-VAE 使潜空间容量可从 1e3 扩展到 1e9 tokens 并持续改善，而**单头 VQ-VAE 很快饱和**；每头 token 过多损害 NPSS（时间一致性）、过少损害 FID 与关键帧精度 → 推荐每头 128–256。
- **ARDY**："vanilla AE 在长 horizon（如 40 帧）训练时严重不稳定甚至发散"，故默认 FSQ；但**性能上AE / VAE / FSQ 三者相当**——即离散化在此处的收益主要是**训练稳定性**而非表达力。
- **EchoAvatar**：CausalConv 类因果卷积 tokenizer"表达能力有限，重建常出现视觉伪影"，改用因果注意力 + 双路径重采样后重建 FID 从 9.208 → 4.183，再加分区码本 → **1.306**。
- **TopoCap（反例）**：明确采用"骨骼结构是组合且离散的，但运动的底层物理占据一个**连续、低维流形**"作为设计前提，因此选择 **Graph CVAE 连续 latent + flow matching**，不做量化。

## B.3 一句话总结（供综述定位）
本届 SIGGRAPH 在 motion 离散表征上出现了**三种不同取向的量化范式并存**：GPC 的 **FSQ + RL 端到端**（把token 直接绑定到物理可执行控制）、MotionBricks 的 **多头并行 VQ**（追求容量可扩展性与实时吞吐）、ARDY 的 **FSQ + 混合表征**（离散 body latent 与显式连续 root 并置以保住轨迹精度）、EchoAvatar 的 **RVQ + 身体分区**（追求层级化与流式纠错）。同时 SMP / TopoCap / MUSIC / ACT / AniGen 等一批工作明确选择连续 latent + diffusion/flow matching，两条路线在本届是显式对峙的。

---

# 附录 C：关于"群体动画（crowds）"与"四足/人形机器人运动"的检索结论

- **群体动画 / crowd simulation**：在 SIGGRAPH 2026 全量 327 篇的标题与 highlight 中**未检索到任何 crowd/crowds 关键词命中**；官方 session 列表中亦无 crowds 相关 session。**结论：本届无专门的群体动画技术论文（就公开列表可见范围）。**
- **四足 / 人形机器人运动**：无独立 session，相关内容分布在：
  - **ReActor**（Motion Transfer & Inbetweening）：显式重定向到两款人形 + 一款四足机器人，真机验证
  - **MotionBricks**（Motion Generation, Warping & Control）：部署到 Unitree G1，LaFAN1-G1 数据集（34 关节）
  - **SMP**：SMP 训练策略直接部署到 Unitree G1
  - **GPC**：面向物理仿真角色，架构可映射到腿式机器人（论文限于仿真）
  - **TopoCap**：四足 benchmark，跨拓扑（双足/四足/六足/飞行生物）零样本重定向
  - **Computational Design of Terrestrial Robots with Anisotropic Friction**（Computational Design）：机器人本体设计，含运动
  - 官方 session 标签打了 **Robotics** 关键词的 motion session：sess104（Motion Generation, Warping & Control）、sess114（Motion Transfer & Inbetweening）

---

# 附录 D：采集方法与可复现性说明

1. **官方全量日程**通过每日快照文件获取（该站点的 Linklings 插件数据源）：
   `https://s2026.conference-schedule.org/wp-content/linklings_snippets/wp_program_view_all_2026-07-{19..23}.txt`
   由此解析出 **85 个 session、309 条presentation 记录**（含全部 Technical Paper session 名称与论文归属）。
2. **论文详情页**（含官方 Description 与作者机构）URL 形式：
   `https://s2026.conference-schedule.org/?post_type=page&p=15&id=papers_XXX&sess=sessYYY`
   另有等价形式：`/presentation/?id=papers_XXX&sess=sessYYY`、`/session/?sess=sessYYY`、`/?p=16&post_type=page&sess=sessYYY`（session页）。
   采集过程中该站点对高频请求启用了反爬（返回 "One moment, please..." 中间页）与 TLS 层拒绝，导致约 60% 详情页未能直接抓取；对关键 session（sess104/105/114）改用搜索引擎缓存与项目主页/arXiv 交叉补齐。
3. **paperdigest 全量列表**（327 篇，含标题 + highlight + 作者 + DOI）：
   `https://resources.paperdigest.org/2026/07/siggraph-2026-papers-highlights`
4. 关键 sessionID 对照：sess104 = Motion Generation, Warping & Control；sess105 = Learning in Motion；sess109 = Faces & Avatars；sess110 = Humans & Hands - Reconstruction；sess111 = 4D Humans；sess114 = Motion Transfer & Inbetweening；sess143 = Perception；sess152 = Digital Humans & Virtual Try-On。
