# SIGGRAPH 2026 与 SIGGRAPH Asia 2026 · Motion 论文逐篇综述

> 编制日期：2026-08-10
> 覆盖范围：人体动作生成、物理角色控制、动作捕捉与估计、动作编辑/风格化/重定向、面部与手部动作、四足与人形机器人运动、4D 动态角色，以及与"动作可驱动性"直接相关的上游工作（rigging / skinning / articulation）。
> 专题重点：**motion tokenizer 与离散运动表征**（FSQ / VQ-VAE / RVQ / LFQ）。

---

## 0. 先说结论

### 0.1 两个会议的可获取状态完全不同

| | SIGGRAPH 2026 | SIGGRAPH Asia 2026 |
|---|---|---|
| 会期 | 2026-07-19 ~ 07-23，洛杉矶（**已结束**） | 2026-12-01 ~ 12-04，吉隆坡 KLCC（**未举办**） |
| 论文公开度 | 完整日程 + 全量327 篇标题/摘要/DOI 已公开 | 官方**仅公布条件接收的 paper ID**（papers_1488 等，无标题） |
| 本报告收录 | **41 篇** motion 相关（含边缘条目），其中核心动作类 **18 篇** | **8 篇已确认 + 2 项待确认** |
| 可靠性 | 高（官方日程 + arXiv + 项目页三重交叉） | 中（全部依赖作者自行公布：个人主页 / GitHub / 高校新闻稿） |

SIGGRAPH Asia 2026 的条件接收结果在 **2026 年 7 月 18–22 日**才集中通知，距今仅三周，且官方匿名政策禁止评审期内标注 venue，因此 arXiv comment 字段几乎查不到（全库仅 3 条命中且均非motion）。**这份SA2026 清单必然不完整**，权威目次预计 2026 年 11 月中下旬随 ACM DL 上线。

### 0.2 关于 motion tokenizer 的核心发现

**离散运动表征在本届 SIGGRAPH 2026 集中爆发，出现了四种互不相同的量化范式并存；而 SIGGRAPH Asia 2026 目前为完全空白。**

四篇代表作及其量化取向：

| 论文 | 量化方案 | 设计动机（一句话） |
|---|---|---|
| **GPC**（NVIDIA/SFU） | FSQ，L=9 / d=40 / G=5 → 词表 59,049 | 把 token 直接绑定到**物理可执行的控制**，端到端 RL 联合训练词汇与策略 |
| **MotionBricks**（NVIDIA） | 多头并行 VQ-VAE，每头 128–256 tokens | 追求**容量可扩展性与实时吞吐**，容量从 1e3扩到 1e9 持续受益 |
| **ARDY**（ETH/NVIDIA） | FSQ 64 levels × 128 维，patch 4 帧/token | **混合表征**：离散 body latent + 显式连续 root，保住轨迹精度 |
| **EchoAvatar**（浙大） | RVQ 6 层 × K=512 × d=512，按上/下身/手三分区 | 追求**层级结构与流式纠错**，配合 LLM 自回归 |

与之对峙的是一大批明确选择连续表征的工作：SMP、TopoCap、MUSIC、ACT、AniGen、STyMo、HyperLoRA、以及 SA2026 全部 8 篇。**这条分歧在本届是显式的、有论证的，而非默认选择。**

最值得关注的一句原文级论断来自 ARDY：他们实现了 AE / VAE / FSQ 三个版本做对照，结论是**三者性能相当，但 vanilla AE 在 40 帧 horizon 上训练会发散**——也就是说，在这个设定下离散化的收益主要来自**训练稳定性**，而不是表达力。这对"是否值得引入 tokenizer"的判断是一个相当冷静的参考点。

---

# 第一部分 · 专题：Motion Tokenizer 与离散运动表征

## 1.1 GPC: Large-Scale Generative Pretraining for Transferable Motor Control

- **作者**：Yi Shi（SFU/NVIDIA）、Yifeng Jiang（NVIDIA）、Chen Tessler（NVIDIA）、Xue Bin Peng（SFU/NVIDIA）
- **Session**：Motion Generation, Warping & Control ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3799902.3811038
- **arXiv**：https://arxiv.org/abs/2606.29148 ｜依赖框架：ProtoMotions3（开源）｜数据集：https://bones.studio/datasets

**问题**：物理角色的生成式控制器多用连续 latent（VAE/GAN），容易 mode collapse，且 latent 流形存在空洞与漂移，导致不自然行为；已有的tokenize 方案训练流程又太复杂。目标是得到一个覆盖广谱人类技能、且能便捷复用到下游任务的通用控制器。

**方法与表征**：

- **FSQ（Finite Scalar Quantization，look-up-free，无显式可学习码本）**
  - 每维量化级数 L=9，潜维度 d=40，隐式码本容量 9^40
  - 量化式 ẑ_t = ⌊⌊L/2⌋·tanh(z_t)⌉，逐元素取整，梯度走 STE
  - **token grouping**：把 G 个连续 token 打包以缩短序列。最终取 **G=5 → 词表 59,049、序列长 8 tokens**
  - 分组消融（L=9, d=40）：G=8 词表 4.3e7 但参数 4.4e10、显存 1.7e2 GB（代价不可接受）；**G=5 为采用值（APD 0.34 / ADE 0.30 / 60M 参数 / 0.27 GB / 92.85 FPS）**；G=4 词表 6,561；G=1 退化到 40 tokens、仅 25.15 FPS
- Encoder 吃参考动作的未来目标状态序列 ŝ_{t:t+h}；Decoder 输出各关节 PD 控制器的目标旋转
- **端到端 RL（PPO）联合优化"运动词汇 + 控制策略"**，只用 motion-tracking 奖励（DeepMimic 式）。这是与"先监督预训练tokenizer 再冻结"路线的关键分野
- 码本学成后，训练 GPT 式因果Transformer 做 next-token prediction，推理用 nucleus (top-p) 采样
- 下游适配用 **CoLA** = DoRA（幅度/方向解耦低秩更新）+ FiLM 任务条件调制，冻结主干，**新增参数 < 1%**

**数据**：Bones/BONES-SEED **> 680 小时、约 343,000 clips**（平均 200 帧）；消融另用 AMASS（40h / 11.3k clips）、LAFAN1（4.6h）、Beyond（10.3 分钟 parkour）。角色为 23刚体 / **66 可驱动 DoF** 的定制humanoid，仿真器 Isaac Gym。

**关键结果**：

| 数据集| 方案 | 成功率 | MPJPE |
|---|---|---|---|
| Bones[680h] | MLP（无量化瓶颈，上界） | 99.98% | 25.56 mm |
| Bones[680h] | VQ-VAE | 99.94% | 37.92 mm |
| Bones[680h] | **FSQ（本文）** | **99.98%** | **34.90 mm** |
| AMASS[40h] | VQ-VAE | 99.30% | 59.28 mm |
| AMASS[40h] | **FSQ** | 99.51% | 44.43 mm |

- **码本利用率：VQ-VAE 仅 10–20%（codebook collapse），FSQ 达82.15%**
- 端到端 RL vs 冻结运动学编码器：FSQ-K（冻结）99.03% / MPJPE 78.26 / 利用率 76.34% → **端到端 RL 把 MPJPE 从 78.26 压到 34.90**，这是本文最有说服力的一组数字
- CoLA 下游鲁棒性：中等推力扰动存活率 **68.1% vs CVAE 44.4%**
- 涌现行为：受力后侧手翻式恢复、自动调步、倒地起身、前滚翻；无条件采样能出cartwheel / handstand / sideflip
- 算力：Bones 上用 24× A100 训练，训练后 RTX 4090 可推理

**信息缺口**：token 帧率 / 控制频率、未来目标窗口 h、Transformer 结构超参、下游任务定量表（论文只给定性关键帧图）。

**为什么这篇对你最重要**：它是本届唯一把"离散运动词汇"与"物理可执行控制"端到端绑定的工作。绝大多数 motion tokenizer 工作是在运动学层面做重建，token 解码出的是姿态；GPC 的 token 解码出的是 PD 目标，意味着每个 token 天然是"可执行的"。同时它给出了 FSQ vs VQ-VAE 在大规模数据（680h）下的直接对照，这个尺度上的对比此前很少见。

---

## 1.2 MotionBricks: Scalable Real-Time Motions with Modular Latent Generative Model and Smart Primitives

- **作者**：Tingwu Wang†、Olivier Dionne†、Michael De Ruyter、David Minor、Davis Rempe、Kaifeng Zhao、Mathis Petrovich、Ye Yuan、Chenran Li、Zhengyi Luo、Brian Robison、Xavier Blackwell、Bernardo Antoniazzi、Xue Bin Peng、Yuke Zhu、Simon Yuen（NVIDIA 等）
- **Session**：Motion Generation, Warping & Control ｜ **Track**：Journal（TOG 45(4)，22 页）｜ **DOI**：10.1145/3811334
- 项目页：https://nvlabs.github.io/motionbricks/ ｜ arXiv：https://arxiv.org/abs/2604.24833 ｜代码：GR00T-WholeBodyControl/motionbricks（预览版已开源）

**问题**：生成式动作合成进展巨大，但实时交互式动作控制仍由传统技术（motion matching /动画图）主导。两个瓶颈：① 实时算力约束下生成式方法的质量与可扩展性显著退化；② 工业界需要"速度指令 + 风格选择 + 精确关键帧"的细粒度多模态控制，纯文本/标签驱动模型给不了。

**方法与表征（tokenizer 细节最丰富的一篇）**：

- **多头（multi-head）VQ-VAE**，注意**不是 RVQ**：沿特征维把连续潜码并行切成 K 块，每块查各自的码本。设计动机是既不用单一巨型码本，也不按身体部位人工切分，完全数据驱动分解；多头使潜流形更鲁棒，单 token 预测错误时能**graceful degradation**
- 标准 VQ 量化损失（含 stop-gradient）+ **running-mean codebook update** 稳定训练；**FSQ 可直接替换，性能相近**（附录 B）
- **推荐每头 128–256 tokens**。容量结论原文有两处张力："FID 与关键帧精度最佳折衷在 1e6–1e7 总容量" vs 多头消融小节的"约 1e9"
- **时间压缩4×**（T帧 → T/4 tokens），数据统一 **30 FPS**，片段 T=12–64 帧、步长 4 帧随机采样
- **架构与参数量**：

| 组件 | 架构 | 参数量 |
|---|---|---|
| Tokenizer（enc/dec） | 1D 卷积 U-Net，下采样率 4（2 层），每层 3 残差卷积，1024 通道| **23.5M** |
| Pose Module | Transformer encoder，dim 1024 / 16 heads / 16 layers | **150M** |
| Root Module | dim 512 / 12 heads，层数更少 | **50M** |

  试过 Transformer 版 enc/dec：性能相近但推理明显更慢，故用卷积。Tokenizer 额外加 foot sliding loss + velocity loss（对重定向数据尤为重要）。

- **Root–Pose 解耦**：encoder 只编码局部姿态不含 root；decoder 把 root 轨迹与关键帧通过 **skip connection 在每个上采样层注入**。稀疏关键帧零填充到 T，布尔 mask 决定用 skip 特征还是隐状态；训练随机采 0–10 个关键帧，推理用前 4 帧 context + 1–4 个 target 关键帧
- 状态表示每帧 (r_g, r_l, p, q, v, c)；**旋转用全局坐标、不做 heading 规范化**（为支持爬行、翻转等 heading 定义不良的动作），改用随机旋转增强
- **4 阶段 coarse-to-fine 推理**：Smart Primitives → 关键帧约束 → Root Module（先预测帧数 T₂，再预测 root 轨迹）→ Pose Module（**masked token modeling**，cosine schedule，**推理通常单次前向**）→ Decoder
- **Smart Primitives 完全 zero-shot**：无需预定义控制类型、one-hot任务标签或微调，一切通过统一关键帧接口通信，等价于构造"全连接 motion graph"。Smart Locomotion 用四阶段渐进 root 精修（线性外推 → 临界阻尼弹簧 → 神经精修 → decoder 全身精修）；Smart Object 用 Intent Keyframes（drop-frame属性 τ + 旋转标志 ω）+ Interaction Binding。非专家用户创作单个 primitive **< 10 分钟**
- 训练：32 GPU、2,000,000 updates；H100 上 tokenizer ≈7 天、root ≈3 天、pose ≈7 天。部署 ONNX + TensorRT，UE5 原生 C++ 插件

**数据集**：自有 mocap **350k**（**700 小时 / 315,162 训练 clips / 9,300 个独立技能 / 36 类 / 163+ 表演者 / 27 关节**）；另有 70k 子集、HumanML3D、LaFAN1-G1（重定向到 Unitree G1，34 关节）。

**关键结果（350k 数据集）**：

| Method | FPS↑ | Latency↓ | FID↓ | Jnt Jit↓ | Root Jit↓ | Foot Sk↓ | Tgt Root↓ | Reach↑ |人评胜率 |
|---|---|---|---|---|---|---|---|---|---|
| Ground Truth | – | – | 0.022 | 3.47 | 1.96 | 0.0008 | – | – | – |
| Cond. Inbtwn. (2022) | 27,000 | 2.4 ms | 1.594 | 16.88 | 12.65 | 0.018 | 0.051 | 87.7% | 0.8% |
| CondMDI (2024) | 1,930 | 33.2 ms | 1.213 | 16.19 | 14.62 | 0.012 | 0.118 | 61.3% | 15.6% |
| MMM (2024) | 3,600 | 18.1 ms | 1.544 | 5.40 | 3.11 | 0.005 | 0.071 | 34.8% | 19.9% |
| CLoSD-DiP (2025) | 4,200 | 15.3 ms | 1.292 | 14.03 | 11.94 | 0.015 | 0.091 | 75.7% | 15.1% |
| **MotionBricks** | **15,000** | **2 ms** | **1.054** | **3.38** | **1.82** | **0.003** | **0.023** | **99.6%** | **86.5%** |

- **抖动 3.38 / 1.82 已优于 GT 的 3.47 / 1.96**；人评 Likert 4.06/5（40 名参与者，1,680 次成对比较）
- 实测：RTX 5090 **2 ms / 15,000 FPS**；Jetson Orin（G1 机载）**5 ms**；重规划 10 Hz 或指令变化触发（lazy replanning）
- **多头 vs 单头消融（最有价值的一张图）**：多头 tokenizer 容量从 1e3 扩到 1e9 持续改善；**单头 VQ-VAE 很快饱和且各规模下均显著更差**。固定总容量 ~1e6 时，每头 token 过多损害 NPSS（时间一致性）、过少损害 FID 与关键帧精度 → 推荐每头 128–256
- 反直觉发现：数据规模缩放时 **FID 随数据变大反而变差**（作者假设小数据过拟合导致多样性低、更贴近训练分布），而关键帧精度随数据持续提升

**信息缺口**：K（头数）最终取值、z_e/z_q 具体维度；最优总容量正文两处不一致。

---

## 1.3 ARDY: Autoregressive Diffusion with Hybrid Representation for Interactive Human Motion Generation

- **作者**：Kaifeng Zhao（ETH Zürich / NVIDIA 实习）、Mathis Petrovich、Haotian Zhang、Tingwu Wang、Siyu Tang（ETH）、Davis Rempe（NVIDIA）
- **Session**：Motion Generation, Warping & Control ｜ **Track**：Journal（TOG 45(4), Article 86，14 页）｜ **DOI**：10.1145/3811284
- arXiv：https://arxiv.org/abs/2607.08741 ｜项目页：https://research.nvidia.com/labs/sil/projects/ardy/ ｜**代码已开源**：https://github.com/nv-tlabs/ardy

**问题**：离线方法（如 Kimodo）控制精确但推理太慢无法交互；在线方法虽实时，却牺牲可控性，或因上下文窗口有限难处理复杂文本语义与长时目标。

**方法与表征（混合表征，本届最精巧的设计之一）**：

- **token x = [m_root ; x_body] ∈ R^D，D = L + 5P = 128 + 20 = 148**
  - **显式 root 特征**：每帧 (p, cos ψ, sin ψ) ∈ R^5，patch 化后每 token 20 维（P=4）
  - **潜在body 嵌入**：x_body ∈ R^128，**经FSQ 量化**
- **量化：FSQ，每特征 64 个 level（记作 "FSQ 64-128"）**
  - 同时实现并对比 **vanilla AE / VAE（KL 1e-6, 128D）/ FSQ**：**三者性能相当，但 vanilla AE 在长 horizon（40 帧）训练严重不稳定甚至发散** → FSQ 设为默认，理由是训练稳定性
  - 例外：HumanML3D benchmark 实验反而用的是 vanilla AE tokenizer
  - 容量消融：FSQ 16-32 / 64-32 / **64-128（默认）** / 64-256
- **Tokenizer 架构**：非对称条件自编码器，Enc/Dec 各 8 层 Transformer、latent dim 512、**因果自注意力**；Decoder 以根运动为条件，且内部做 **global→local 根转换**（ψ̇, ṗ_x, ṗ_z, p_y）替代全局根作条件，显著抑制脚滑；损失含脚滑项 L_skate（权重 0.01）
  - 训练：AdamAtan2，lr 2e-5，batch 128，**4,000,000 步，单张 A100-80GB**
- **时间粒度**：patch **P=4 帧/token**（消融 1/4/8）；生成窗口 **G=40 帧 = 2 秒 @ 20 fps**（消融 4/8/12/20/40）
- **两阶段交错自回归扩散去噪器**：Root Transformer 预测干净全局根运动 → **detach** → Body Transformer 预测干净 body token → 拼接 → 推理时重新加噪送回，**每个去噪步都交错执行**
  - 每个 Transformer 8 层 8 头 dim 1024，总参数约 **156M**；文本编码器 LLM2Vec（Llama-3-8B-Instruct）
  - **可变历史上下文 H：0–8 秒；超窗口未来约束上下文 F：最大 10 秒**（对比 DiP 1.00/2.00、DartControl 0.07/0.27）
  - 扩散 DDPM，默认 **10 步**（消融 1/2/3/4/10/100，4 步对多数应用可接受）
  - 约束注入：窗口内根约束直接覆写噪声 token 根部分；身体约束沿特征维拼接；超窗口约束作为额外 token（未约束 token 在注意力中mask 掉）

**数据**：Bones Rigplay Mocap（约 700 小时、>150 人、统一 27 关节、训练 ≈315k / 测试 ≈35k clips，20 fps）；HumanML3D（排除 HumanAct12 子集，重定向时**保留原始 SMPL 关节旋转**）。

**关键结果**：

| 消融项 | FID↓ | Waypoint↓ | 备注 |
|---|---|---|---|
| **ARDY 完整** | **0.027** | **0.024** | Joint rot 2.23° / Joint pos 2.5 cm / Keyframe 2.3 cm |
| Explicit representation 变体 | 0.065 | 0.203 | 纯显式表征明显更差 |
| Global root-cond. decoder | 0.028 | 0.060 | 验证 global→local 转换的价值 |
| One-stage（不分两阶段） | 0.029 | 0.164 | 验证 root/body 分离的价值 |
| Horizon 4 帧 | 0.224 | 0.850 | 短窗口严重退化 |
| Tokenizer patch P=1 | 0.152 | – | 逐帧 token 严重退化 |
| Tokenizer patch P=8 | 0.022 | – | FID 更好但 Joint pos 劣化到 0.070 |

- Tokenizer 类型对照（horizon 20）：AE 128D FID 0.033/Waypoint 0.044；VAE 128D 0.031/0.046；**FSQ 64-128 0.030/0.046** —— 三者确实相当
- HumanML3D 对比离线法：**ARDY（无优化）R-Prec 0.729 / FID 0.044 / Skate 6.28% / Error 4.15 cm / 延迟 0.15 s**；MaskControl含优化虽 Error 0.45 cm 但要 **68.65 s**
- 实时性：RTX 4090，4 步模型平均延迟 **33 ms**（≈30 次规划/秒），10 步**63 ms**
- 感知实验 240 组成对比较，ARDY 被"strongly and consistently preferred"

**这篇的方法学启示**：它把"该离散化什么"这个问题拆开来回答了——**根轨迹保持显式连续（因为精度要求高、语义清晰），身体姿态走离散 latent（因为分布复杂、需要稳定训练）**。相比一刀切地把整个姿态向量塞进码本，这是更细粒度的判断。

---

## 1.4 EchoAvatar: Real-time Generative Avatar Animation from Audio Streams

- **作者**：Bohong Chen、Yumeng Li、Yinglin Xu、Youyi Zheng、Yanlin Weng、Kun Zhou（浙江大学 CAD&CG 国家重点实验室）
- **Session**：Faces & Avatars ｜ **Track**：Conference Papers ｜ **DOI**：10.1145/3799902.3811066
- arXiv：https://arxiv.org/abs/2605.28272 ｜**项目页含代码 + 预训练模型**：https://robinwitch.github.io/EchoAvatar-Page

**问题**：既有音频驱动动作合成多为离线处理完整音频、或限定单一域（只做语音手势或只做音乐舞蹈）；交互式 avatar 需要低延迟流式全身动作生成，并且要能同时吃语音和音乐。

**方法与表征（RVQ 细节最完整的一篇）**：

- **RVQ（残差向量量化）**
  - **Q=6 层残差，每层码本 K=512，latent d=512**
  - **时间下采样 4× → token 帧率 7.5 Hz**（数据 30 FPS）
  - **码本按解剖学分区：上半身 / 下半身 / 手部共 3 套独立码本**（这是重建质量提升的最大来源）
  - Commitment权重 η=0.1；训练窗口随机采 64 帧
- **因果注意力 tokenizer**：用堆叠 attention block 替代卷积主干，causal mask 限制感受野。动机明确——CausalConv（MotionStreamer 的用法）表达能力有限、重建有视觉伪影
- **双路径时间重采样**（借鉴 DC-AE）：下采样= temporal pooling 分支 + feature concat 分支；上采样 = temporal replication + channel-expansion MLP
- **FK 辅助损失**：把 Forward Kinematics 接入优化回路，对全局关节位置/速度/加速度/足部接触一致性施加损失以抑制脚滑
- **生成主干：Qwen2.5-0.5B-Instruct**（解绑 tied embedding 以适配模态专属词表）。上下文窗口 4 秒 = 音频 token 600 个（Causal EnCodec 前 2 层 RVQ，75 Hz）+ motion token 540 个（6 层 RVQ × 3 分区，MusicGen 式flattened interleaving）
- **两个针对 RVQ 自回归的专门设计**：
  - **Hierarchical Loss Scaling**：对更深 RVQ 层的交叉熵施加单调递减权重，优先保结构一致性
  - **Hierarchical Token Corruption**：解决 conditional collapse（自回归motion prior 短路较弱的音频条件）。对选中时间步采样层深 ℓ_t ~ U(1,L)，随机化第 ℓ_t 至 L 层 token，**保留粗粒度底层不变**。双重收益：模拟真实生成伪影作数据增强 + 赋予长序列自回归纠错能力
- **Example Control**：沿用 MECo，但**仅从参考序列的第 1 级码本**抽取控制 token（语义密度集中在 primary VQ 层）
- **流式**：原生 30 FPS，chunk 8 帧 = 0.266 秒滑动窗口，吞吐约 300 tokens/s，H200 上总延迟约 177 ms < 266 ms chunk 时长；CUDA Graph 降低 kernel 调度开销
- 还提供 **tool-call 接口**，让上游 LLM 注入显式语义动作，与隐式音频驱动动作交织

**数据**：ZeroEGGS（约 2 小时，单说话人 19 种表达风格）+ Motorica（约 6 小时，5 表演者 8 舞种，手指数据用 inpainting 修复）+ BEAT2（基准）；RL 语料另采语音 1 小时与音乐 100 首。训练 2× H200 约 30 小时。

**关键结果**：

| Tokenizer 变体 | 重建 FID↓ | MPJPE↓(1e-4 m) | Trans Loss↓ | 生成 FID↓ |
|---|---|---|---|---|
| CausalConv-RVQ（基线） | 9.208 | 525.6 | 12.94 | 18.06 |
| Attn | 4.183 | 411.6 | 12.53 | 12.25 |
| Attn w/o dual-path | 12.21 | 982.4 | 48.77 | 18.55 |
| Attn w/o FK 辅助 | 8.775 | 778.5 | 55.67 | 17.68 |
| **Attn + bodypart 分区** | **1.306** | **184.1** | **9.637** | **9.465** |

- **重建 FID 从 9.208 一路压到 1.306**，其中"因果注意力"贡献 9.208→4.183，"分区码本"贡献 4.183→1.306
- **BEAT2 基准 FID 2.874（新 SOTA）**，对照 MECo 3.401、PersonaGesture 3.930、EMAGE 5.512
- 主观 1,680 次成对判断，手势与舞蹈两域均显著优于 MECo / EDGE
- **RL 对齐的诚实负面结果**：GRPO 与 DPO 都让 FID 退化（9.465→24.13 / 12.39）却提升人类偏好——mode-seeking 导致分布覆盖度下降。结论是舞蹈适配 GRPO、对话手势适配 DPO
- 奖励模型设计值得一看：用不同强度 corruption 降质 GT 构造已知退化等级，奖励模型与 GT 退化等级 **Pearson 0.9977**

**信息缺口**：因果掩码回溯帧数 p、每层码本是否异构、逐阶段延迟 ms。

---

## 1.5 SimArt（旁支参照）：稀疏 3D VQ-VAE

- **SimArt: Decomposing Monolithic Meshes into Sim-ready Articulated Assets via MLLM**
- 作者：Chuanrui Zhang（NTU）、Minghan Qin、Yuang Wang、Baifeng Xie、Hang Li（ByteDance Seed）、Ziwei Wang（NTU）
- Session：Generative 3D (2) ｜ Conference Papers ｜ DOI：10.1145/3799902.3811173

MLLM 统一框架 + **稀疏 3D VQ-VAE** 联合建模部件分解与运动学，明确目标是**降低 token 复杂度**。对象是物体部件的几何与运动学，不是人体动作序列，但"如何用稀疏化压低 token 数"这个思路对 motion tokenizer 的序列长度问题有直接参照价值。量化细节（码本大小、体素分辨率）未公开。

同一批人（Tsinghua+VAST）的 **SkinTokens / TokenRig**（arXiv 2602.04805，非本届主会）用 **FSQ-CVAE 把 skinning weights 离散化为 token**，配 Qwen3-0.6B 自回归 + GRPO，skinning 精度提升 98%–133%。如果你在关注"离散化能推广到哪些运动相关量"，这是个有意思的旁证。

---

## 1.6 横向对比表

| 维度 | GPC | MotionBricks | ARDY | EchoAvatar |
|---|---|---|---|---|
| 量化方法 | FSQ（look-up-free） | 多头 VQ-VAE（FSQ 可替换） | FSQ（对比了 AE/VAE） | RVQ |
| 级数 / 码本大小 | L=9 levels | 每头 128–256 tokens | 64 levels/特征 | 每层 K=512 |
| 潜维度 | d=40 |未公开（卷积 1024 通道） | 128（body latent） | d=512 |
| 有效词表 | 分组后 9^5= 59,049 | 总容量 1e6–1e9（消融跨 6 个数量级） | FSQ 64-128 | 6层 × 512 × 3 分区 |
| 层/头结构 | 单层 40 维 → G=5 分组 → 8 tokens/步 | **K 个并行码本（沿特征维）** | 单层 | **6 层残差 + 3 身体分区** |
| 时间粒度 | 未公开 | 4× 下采样，30 FPS → 7.5 Hz | 4 帧/token，20 fps → 5 Hz | 4× 下采样，30 FPS → 7.5 Hz |
| 窗口长度 | 未公开 | T=12–64 帧 | G=40 帧 = 2 s | 训练 64 帧，流式 chunk 8 帧 |
| Tokenizer 架构 | Enc 吃未来目标状态，Dec 出 PD 目标 | 1D 卷积 U-Net，23.5M | 8层 Transformer 因果 AE，dim 512 | 因果注意力 + 双路径重采样 + FK 损失 |
| 训练方式 | **端到端 RL（PPO）** | 监督重建 + 脚滑/速度损失 | 监督重建 + 脚滑损失 | L1 + commitment + FK |
| 上层模型 | GPT 式因果Transformer（60M） | Pose(150M) + Root(50M)，masked token modeling | 两阶段交错自回归扩散（156M） | Qwen2.5-0.5B |
| 重建质量 | 成功率 99.98% / MPJPE 34.90 mm；**利用率 82% vs VQ 10–20%** | 随容量单调改善，多头远优于单头 | FID 0.030（与 AE/VAE 相当） | **FID 1.306 / MPJPE 184.1** |
| 选它的理由 | 免维护码本，避开 codebook collapse | 容量可扩展 + 单token 错误可优雅降级 | **训练稳定性**（AE 长 horizon 发散） | 层级结构 + 流式纠错 |

**四条可直接引用的论断**：

1. **GPC**：连续 latent 控制器"prone to mode collapse and unnatural behaviors caused by drift and gaps in the latent manifold"；FSQ 相比先前 tokenized 方法"greatly simplifies the training process"。
2. **MotionBricks**：多头 VQ容量从 1e3 到 1e9 持续受益，**单头 VQ-VAE 很快饱和**；每头 token 过多损害时间一致性（NPSS）、过少损害 FID 与关键帧精度。
3. **ARDY**：AE / VAE / FSQ **性能相当**，离散化的实际收益是训练稳定性——vanilla AE 在 40 帧 horizon 上会发散。
4. **EchoAvatar**：因果卷积 tokenizer"表达能力有限、重建有视觉伪影"；改因果注意力 + 双路径重采样 + 身体分区码本，重建 FID 9.208 → 1.306。

**反方立论（TopoCap）**：明确以"骨骼结构是组合且离散的，但运动的底层物理占据一个连续、低维流形"为设计前提，因此选 Graph CVAE 连续 latent + flow matching，不做量化。这是本届对"运动是否该离散化"最直接的反面论证。

---

# 第二部分 · SIGGRAPH 2026 逐篇总结

## Session A：Motion Generation, Warping & Control（07-21，Room 403 A，Chair: YutingYe）

前三篇（GPC / MotionBricks / ARDY）已在第一部分详述，这里补齐其余三篇。

### 2.1 SMP: Reusable Score-Matching Motion Priors for Physics-Based Character Control

- 作者：Yuxuan Mu\*、Ziyu Zhang\*、Yi Shi\*、Dun Yang（SFU）、Minami Matsumoto、Kotaro Imamura、Michael Taylor（Sony Interactive Entertainment）、Guy Tevet（Stanford）、Chuan Guo（Snap）、Chang Shu、Pengcheng Xi（NRC Canada）、Xue Bin Peng（SFU/NVIDIA）
- Journal Track ｜ DOI：10.1145/3811282 ｜ arXiv：https://arxiv.org/abs/2512.03028 ｜项目页：https://yxmu.foo/smp-page

**问题**：AMP 类对抗式模仿学习虽是学motion prior 的有效手段，但判别器几乎必须为每个新控制器重训，且下游任务仍需访问原始参考动作数据，可复用性差。

**方法（连续，无量化）**：用**预训练 motion diffusion model + Score Distillation Sampling（SDS）** 构造任务/策略无关的 score-matching prior。先验一次训练后**冻结**，充当通用奖励函数（stationary reward model）。配套三个设计：
- **Ensemble Score Matching**：在固定 timestep集（如 K={22,15,8}）上平均噪声残差以降方差，给 PPO 平滑奖励
- **自适应归一化**：跟踪各噪声级 SDS 误差的运行均值 μ_i 缩放奖励，实现"即插即用"，免手工调参
- **Generative State Initialization**：用冻结扩散模型直接采样初始姿态替代 RSI，从而**完全脱离原始数据**

风格提示与合成：style-conditioned diffusion（100STYLE 训练）+ CFG 把奖励"提示"到指定风格；**空间掩码混合**两种风格的 ε 预测（如上半身 aeroplane、下半身 HighKnees），能合成数据集中不存在的复合风格。

**关键结果**（Skill Position Tracking Error [m]，越低越好）：

| Skill | DM | AMP | AMP-Frozen | SMILING | **SMP** |
|---|---|---|---|---|---|
| Walk | 0.010 | 0.010 | 0.044 | 0.042 | 0.030 |
| Run | 0.013 | 0.088 | 0.129 | 0.115 | **0.067** |
| Cartwheel | 0.243 | 0.043 | 0.419 | 0.104 | **0.043** |
| Backflip | 0.073 | 0.058 | 0.272 | 0.144 | 0.069 |
| **Average** | 0.070 | 0.046 | 0.246 | 0.092 | **0.043** |

AMP-Frozen 严重退化（0.246）说明"直接冻结判别器"这条捷径不可行，这正是 SMP 存在的理由。**数据效率**：仅用 3 秒 walk/jog/run 片段构造先验，策略即可在 [1.2, 6.8] m/s 连续速度范围内自动调整步频步态，并涌现出数据中不存在的 walk→jog→sprint 过渡。**真机**：策略直接部署到 Unitree G1，仅本体感知观测，无需运行时运动规划器。

---

### 2.2 Deep Motion Warping via Phase-Conditioned Diffusion Autoencoder

- 作者：Bowen Zheng（浙大 / 腾讯 VISVISE）、Linjun Wu（浙大）、Xinwei Jiang、Yujin Chai、Zijiao Zeng（腾讯 VISVISE）、He Wang（UCL）、Xiaogang Jin（浙大）
- Conference Papers（Article 93，11 页）｜ DOI：10.1145/3799902.3811144｜ **无 arXiv 预印本**

把动作**显式解耦为 root velocity + phase + style** 三个因子，以 phase 为条件构建扩散自编码器，支持 root warping、夸张化、time warping、风格迁移四类高层编辑，同时严格保持结构一致性。属**连续 latent + diffusion**，无量化。

技术细节、数据集与定量指标**均未找到**（无预印本或补充材料公开）。同组前作可作参照：AutoKeyframe（SIGGRAPH 2025）、Decoupling Contact for Fine-Grained Motion Style Transfer（SIGGRAPH Asia 2024）、Ultrafast and Controllable Online Motion Retargeting for Game Scenarios（TOG 44(6)）。

---

### 2.3 MultiAct: Text-to-Motion Generation from Composite Text via Tailored Attention Guidance

- 作者：Nathan Sala（TAU）、Ofir Abramovich（Reichman / CYENS）、Ariel Shamir（Reichman）、Daniel Cohen-Or（TAU）、Andreas Aristidou（University of Cyprus / CYENS）、Sigal Raab（TAU）
- Conference Papers ｜ DOI：10.1145/3799902.3811092 ｜ arXiv：https://arxiv.org/abs/2605.30925 ｜项目页：https://natsala13.github.io/multiact.github.io

**问题**：现有 T2M 模型在**复合提示**（同时发生的多个动作，如 "running while waving arms"）下脆弱，常只实现主导动作而忽略其余成分，作者称之为 **vanishing semantics**，归因于 **entangled cross-attention**——多语义成分竞争时注意力质量塌缩到少数主导 token。注意这与顺序动作（"walk, then jump"）是根本不同的问题。

**方法**：**推理时、无配对数据、免重训练、免改架构**，直接作用于预训练动作生成器。机制是自适应放大与被弱表征提示成分相关的 cross-attention 分数；关键洞察是有效调制依赖 prompt 特定选择（该调制哪些 token、哪些层），故引入轻量辅助决策方案自动确定最优参数化。不引入新动作表征，不涉及量化。

定量数值**未找到**（抓到的 HTML 仅到 Introduction）。定性上 backbone 在 "hops forward while raising his arms"、"dribbles a ball while moving backwards" 上会漏掉一个动作成分，MultiAct 能全部生成。

---

## Session B：Learning in Motion（07-23，Room 403 A）

### 2.4 R-DMesh: Video-Guided3D Animation via Rectified Dynamic Mesh Flow

- 作者：Zijie Wu（HUST / 腾讯混元）、Lixin Xu、Puhua Jiang、Sicong Liu、Chunchao Guo（腾讯混元）、Xiang Bai（HUST）
- Conference Papers ｜ DOI：10.1145/3799902.3811135 ｜ arXiv：https://arxiv.org/abs/2605.13838 ｜项目页：https://r-dmesh.github.io/ ｜**代码已开源**：https://github.com/Tencent-Hunyuan/R-DMesh

**问题**：视频驱动 3D 动画的**位姿错配困境**——用户提供的静态 mesh初始位姿几乎不会与参考视频首帧对齐，强行让mesh 跟随不匹配轨迹会导致严重几何畸变或动画失败。

**方法**：R-DMesh VAE 显式解耦为①条件 base mesh ②相对运动轨迹 ③**rectification jump offset**（在动画开始前把输入 mesh 的任意位姿"校正"到视频初始状态）。**Triflow Attention** 用逐顶点几何特征调制三条正交 flow，保证校正与动画过程的物理一致性与局部刚性。生成器是**rectified flow DiT**，以Wan2.2-TI2V-5B 提取的视频 latent 为条件。**直接预测顶点轨迹，不依赖骨骼 rigging 或形状先验**，因此能驱动拓扑变化的角色与普通物体。连续 latent，无量化。

**数据**：自建 **Video-RDMesh，> 50 万动态 mesh 序列**，专门构造以模拟位姿错配。应用含 pose/motion retargeting 与 holistic video-to-4D（结合 Hunyuan3D 把 in-the-wild 视频转高质量 4D 资产）。定量表数值未公开。

---

### 2.5 TopoCap: Learning Topology-Agnostic Motion Priors for Monocular Video-to-Animation

- 作者：Cheng-Feng Pu（清华致理书院）、Jia-Peng Zhang、Meng-Hao Guo（清华 BNRist）、Yan-Pei Cao（VAST）、Shi-Min Hu（清华）
- Journal Track ｜ DOI：10.1145/3799902.3811159 ｜ arXiv：https://arxiv.org/abs/2606.12153 ｜**数据集已公开**：https://huggingface.co/datasets/duckduckplz/Mobjaverse

**问题**：现有动捕受限于物种专属模板（如 SMPL）或需繁重手工 rigging。目标是从单目视频提取动作并零样本重定向到**任意未见骨骼拓扑**（双足→六足→无生命物体），且**无需 test-time optimization**。

**方法（本届最明确的"反离散化"立场）**：核心洞察是"**骨骼结构是组合且离散的，但运动的底层物理占据一个连续、低维流形**"。
- Stage I（Manifold Discovery）：**Graph CVAE** 把异构运动链压缩为共享定长 latent code（K×D），用 Perceiver 式瓶颈；解码器**显式条件于目标 rig 的结构嵌入**，解耦运动动力学与骨骼拓扑；解码用解析 IK 保证全局一致性
- Stage II（Generative Extraction）：把 video-to-animation 建模为**条件 flow matching**，从视觉特征预测拓扑无关 code

**数据**：**Mobjaverse**，从 Objaverse-XL 精选，**> 5,000 种独立骨骼拓扑、200 万帧**，结构多样性超既有数据集**两个数量级**。结果在人体与四足 benchmark 上优于专家模型，并对长尾 3D 生物零样本重定向；支持跨拓扑迁移（把四足跌落迁移到飞龙，保留 gait/energy/phase 高层语义）；可作为可扩展的运动数据生成引擎。具体定量表未公开。

---

### 2.6 Stylized Text-to-Motion Generation via Hypernetwork-Driven Low-Rank Adaptation（HyperLoRA）

- 作者：Junhyuk Jeon\*、Seokhyeon Hong\*、Junyong Noh（KAIST Visual Media Lab）
- Conference Papers ｜ DOI：10.1145/3799902.3811205 ｜ arXiv：https://arxiv.org/abs/2605.13333

**问题**：文本难以表达动作的细粒度风格。既有方法要么需按风格微调（LoRA-MDM：一次微调只对应单一风格），要么依赖重型 ControlNet（SMooDi：容量与算力开销大）；风格表征上，标签监督法偏向已见标签，无标签连续法缺乏显式结构。

**方法**：内容无关的 **style adapter**（MLP 堆叠）从参考动作提全局 style embedding；**hypernetwork** 把该 embedding 映射为 LoRA 低秩更新（实现上是 style-dependent FiLM 调制），在**每个去噪步**注入冻结的预训练 T2M 扩散模型。用 **supervised contrastive loss** 结构化 style latent 空间使其内容独立。引导用标准 CFG + 新的 **style encoder guidance**（用训练好的 style encoder 做梯度引导，导向目标**连续** style embedding，无需离散风格类别）。

**结果**（HumanML3D + 100STYLE）：**SRA 76.034%**（SMooDi 75.818%、LoRA-MDM 42.047%）、R-Precision 0.721、Multimodal Distance 3.519。用户研究style expression 4.215/ content preservation 4.520 / motion quality 4.050，均显著高于所有基线。消融证明 supervised contrastive loss 与 style encoder guidance 是泛化到未见风格的关键，尤其在训练风格数受限时。

---

### 2.7 MUSIC: Learning Muscle-Driven Dexterous Hand Control

- 作者：Pei Xu\*、Yufei Ye\*（Stanford）、Shuchun Sun（Clemson）、Yu Ding、Elizabeth Schumann、C. Karen Liu（Stanford）
- Journal Track ｜ DOI：10.1145/3811402 ｜ arXiv：https://arxiv.org/abs/2604.23886 ｜项目页：https://pei-xu.github.io/MUSIC

让**肌肉骨骼手**在物理仿真中完成精细灵巧控制，并能演奏参考数据集之外的新钢琴曲目。

**方法**：分层架构 = 高频肌肉级控制 + 低频 latent 空间协调。三阶段训练：① RL 训练单手tracking policy，输出高频**肌肉-肌腱激活**跟踪大规模参考轨迹；② **on-policy distillation** 把 tracking policy 蒸馏为 **VAE**，得到平滑结构化 latent（抽象掉低层肌肉动力学），decoder 作低层servo；③ 在该 latent 上训练曲目专属高层控制器，以乐谱 note events 为目标协调双手。高层建模为**去中心化多智能体 RL**（两手为独立 agent、共享双手运动学观测）+ 对抗式模仿。另提出增强的肌肉骨骼手模型。连续 VAE latent，无量化。

**结果**：15 首曲目（Bach、Beethoven、Chopin、Debussy、Joplin、Mozart、Scriabin 等），能合成按键准确的协调双手动作，泛化到参考数据之外的新乐谱；手模型在生物力学稳定性与跟踪精度上优于既有模型；**肌肉激活模式与人类 EMG 记录在多任务下一致**（生理可信度证据）。定量表未公开。另有 Emerging Technologies 现场 Demo。

---

### 2.8 MOCHI: Motion Enhancement of Collaborative Human-object Interactions

- 作者：Jiye Lee\*、Yonghun Choi、Jungdam Won†（首尔大学）
- Journal Track（TOG 45(4), Article 160，18 页）｜ DOI：10.1145/3811308 ｜ arXiv：https://arxiv.org/abs/2606.18243 ｜项目页：https://jiyewise.github.io/projects/MOCHI/
- 入选 SIGGRAPH 2026 Technical Papers Trailer；一作获 2026 WiGRAPH Rising Stars

**问题**：多人-物协作交互（MHOI）数据采集极难——遮挡频繁、需同时捕大幅身体动作与手指细节，导致三类伪影：手-物接触未对齐、运动抖动与时间不一致、手指关节细节缺失。

**方法**：两阶段。阶段一从含噪身体输入出发，通过**优化**生成物理可信且与身体姿态语义一致的手部抓握，再扩展为完整手-物交互序列。阶段二用**基于扩散的 noise optimization 框架**（使用**单人motion prior**）精修所有参与者的全身动作，优化中引入目标项把人-物与人-人交互信息编码进单人先验。连续扩散先验 + 优化，无量化。

**结果**：既适用于采集数据，也适用于生成模型输出（用 OMOMO 演示，从有限训练样本产出无限量高质量 MHOI 数据）；对参与者数量（含 >2 人）与交互类型稳健。应用含 keyframe-based MHOI creation（3D 物体由 text-to-3D 生成，标注者在 30 FPS 下每 40 帧稀疏关键帧上粗略指定位姿，方法补全为完整序列）与 object variations 数据增强。定量表未公开。

---

### 2.9 ACT: A Unified Framework for Rigging and Animating Characters with Arbitrary Topologies

- 作者：Pengyu Long（上科大 / 字节游戏）、Weirui Wang、Qingcheng Zhao、Qixuan Zhang（上科大 / Deemos）、Xiaoyang Guo、Xiaoyu Pan、Jiaqing Zhou、Tianlei Hu（字节游戏）、Wei Yang（HUST）、Lan Xu、Jingyi Yu（上科大）
- Journal Track（TOG 45(4): 161:1–161:20）｜ DOI：10.1145/3811392 ｜ **无 arXiv / 项目页**

**问题**：传统流水线把 rigging → skinning → motion synthesis 割裂为顺序阶段，忽视了**形态结构与运动功能之间的内在耦合**。

**方法**：把 rigging 与 animation 重构为同一个 hyper-kinematic 过程的互补视图，核心是在**共享 latent 空间**中建模骨骼拓扑与时序运动的**联合分布**。用 VLM 从任意 mesh 抽取语义拓扑先验，条件化一个 DiT 主干；把静态 rest pose 与动态轨迹视为统一序列，采用**任务感知 masking 策略**，在单一端到端架构内完成零样本 rigging、文本引导动作生成、动作补全。geometry-guided decoder 保证表面形变与生成运动学紧密耦合。连续共享 latent + DiT，无量化描述。

新应用：**语义驱动的拓扑编辑**、生成式 in-betweening。具体数据集与定量表未公开。

---

## Session C：Motion Transfer & Inbetweening（07-22，Room 403 A，Chair: Pei Xu）

### 2.10 Motion4Motion: Motion Transfer Across Subjects at Inference

- 作者：Ling-Hao Chen（清华 / Stepfun）、Zixin Yin（HKUST / Stepfun）、Duomin Wang、Xianfang Zeng、Gang Yu（Stepfun）
- Conference Papers ｜ DOI：10.1145/3799902.3811062 ｜ arXiv：https://arxiv.org/abs/2607.11644 ｜项目页：https://lhchen.top/Motion4Motion/

**问题**：视频动作迁移主流方法硬编码依赖预定义人体骨架并需 skeleton-conditional 训练，难以泛化到跨物种同时保留其独特运动风格。两大根本挑战：跨拓扑配对数据极度稀缺；无骨架时源/目标之间的语义对应存在歧义（椅子腿 vs 四足动物腿）。最相关的FlexiAct 依赖 per-case optimization，导致过拟合与信息泄漏。

**方法（完全 training-free）**：推理时操纵预训练视频 DiT 的自注意力。不用骨架，改为建模视频中角色的**像素级 motion flow**，用语义匹配算法建立 skeleton-free 的点到点对应。核心是**TransPE（Transferring Positional Encoding）**：把目标角色首帧的外观特征（经 diffusion inversion 缓存的 K̂, V̂）注入注意力，并用迁移后目标运动流 M_tgt 的位置信息重新做 RoPE 编码：

  K_new = [RoPE(K), RoPE(K̂, M_tgt)]，V_new = [V, V̂]，X ← Softmax(RoPE(Q)·K_newᵗ/√d)·V_new

**结果**：文本相似度、运动保真度、时间一致性、外观一致性、姿态相似度五项全面优于基线（具体数值未公开）。定性上跨物种迁移（human→panda / →goose、狐狸步态→长颈鹿、狮子扑击→斑马）保持语义姿态对齐与清晰纹理，无需骨骼先验或微调；甚至能给一张桌子赋予人类行走步态。

---

### 2.11 STyMo: Fast and Controllable Few-Shot Motion Style Transfer

- 作者：Jose LuisPonton、Alexander Winkler、Ladislav Kavan、Yuting Ye、Petr Kadleček（Meta Reality Labs）
- Journal Track ｜ DOI：10.1145/3811356 ｜项目页：https://jlpm22.github.io/stymo-project-page ｜**代码 + 处理后配对数据集已开源（MIT）**：https://github.com/facebookresearch/STyMo

**方法**：**few-shot**——仅从数秒配对动作数据学风格，**训练 1–2 分钟**。核心洞察是把风格分解为 **static component**（时不变姿态）与 **temporal component**（逐帧动力学）。三阶段：
- **Static Model**（AveragePoseModel）：对平均旋转分类以混合 K 个预计算 style chunks，捕捉持续性姿态偏移
- **Gating Model（Stylizability Gate）**：用最近邻学置信边界，预测逐帧stylizability 分数 γ；输入姿态离训练数据过远时自动降低风格强度，**防止 OOD 动作上的伪影**——这个设计在工程上很实用
- **Temporal Model**：Transformer encoder-decoder，encoder 处理先前预测，decoder 对其做 cross-attention

连续回归，无扩散无量化。**运行时控制**：姿态强度、时间夸张度、逐身体区域风格（body-part masks）、contact-preservation optimization（修正脚滑/穿地）。风格示例含 zombie dance、angry jump、Neutral/Angry/Clown/Happy/Zombie。定量数值未公开，但有完整 User Study 章节。

---

### 2.12 Skinned Motion Retargeting with Spatially Adaptive Interaction Guidance

- 作者：Soojin Choi、Seokhyeon Hong、Chaelin Kim、Junghyun Nam、Junhyuk Jeon、Junyong Noh（KAIST VML）
- Journal Track ｜ DOI：10.1145/3811354 ｜ arXiv：https://arxiv.org/abs/2605.19355 ｜项目页：https://suzyn.github.io/space_page/（code coming soon）

**问题**：在体型差异很大的角色间重定向并保留**交互语义**（自接触、近身邻近）仍困难。近期几何感知方法依赖**静态对应**（预定义区域间的空间关系），目标角色比例夸张时往往失效。

**方法**：
- **Adaptive Anchor Sampling（AAS）**：不再用静态 anchor 定义，而是动态把 anchor 重定位到目标角色的可达区域；Transformer 预测 anchor 位移，通过**可微soft projection** 约束位移后 anchor 仍位于目标几何上；引入源角色的位姿相关空间结构，使适配后的 anchor 提供结构一致的交互引导
- **Proximity-based Retargeting（PbR）**：以这些 anchor 为条件，用**图自编码器**预测目标骨骼动作
- **交替优化**：先更新 AAS 再更新 PbR，消解"邻近目标既可由 anchor 重定位、也可由姿态调整满足"的歧义

**结果**：穿透率、接触保留precision/recall/accuracy 与用户研究均优于 MotionBuilder、SAME、R2ET、MeshRet，夸张形态下优势尤为明显（数值未公开）。局限交代得很清楚：可达性损失是软约束，未精确建模关节 ROM；假设源动作无几何伪影；主要依赖逐帧空间对应、未显式建模长时动力学；假设骨骼结构一致；仅考虑地面接触。

---

### 2.13 ReActor: Reinforcement Learning for Physics-Aware Motion Retargeting

- 作者：David Müller、Agon Serifi、Sammy Christen、Ruben Grandia、Espen Knoop、Moritz Bächer（Disney Research Switzerland）
- Journal Track ｜ DOI：10.1145/3811378 ｜ arXiv:2605.06593 ｜ PDF：https://www.baecher.info/assets/pdf/Mueller2026.pdf

**问题**：把人类运动学参考重定向到机器人形态时，现有方法常产生**物理不一致**（脚滑、自碰撞、悬空/地面穿透、动力学不可行），阻碍下游模仿学习。既有优化法需预定义接触模式、易陷局部极小、需大量手工调参；学习法需大量配对数据且多假设理想球关节。

**方法**：**双层优化框架**——下层用 RL 训练 motion tracking 策略；上层精修由**稀疏语义刚体对应**定义的重定向参数，使参考动作自适应匹配机器人形态。为使优化可解，推导了上层损失的近似梯度。只需稀疏语义对应、免手工调参，并把重定向直接与物理仿真集成。

**结果**：仿真与**真机硬件**双重验证，成功迁移到与人类差异极大的形态，包括**两款人形机器人与一款四足机器人**（不同 DoF、形状、尺寸、比例），消除脚滑、悬空、自穿透三类经典伪影。定量表未公开。同组相关：AMOR（SIGGRAPH 2025）、Robot Motion Diffusion Model（SA2024）、VMP（SCA 2024）。

---

### 2.14 Adaptive Interpolation-Synthesis for Motion In-Betweening on Keyframe-Based Animation

- 作者：Anton Raël、Julien Boucher、Antoine Lhermitte（机构未找到，从"production data + Maya 集成"判断为工业团队）
- Conference Papers ｜ DOI：10.1145/3799902.3811157 ｜ arXiv：https://arxiv.org/abs/2605.02742

**问题**：motion in-betweening 是 3D 动画最耗时且最需艺术判断的环节，是主要产能瓶颈。近期深度方法虽效果好，但其**数据特性、动作风格与问题设定都与专业动画流程不符**。

**方法**：**AIS layer** 镜像动画师创作过程，动态平衡"学习到的插值"与"直接姿态合成"：
- 合成路径 p_synth_t = MLP_synth(h_t)，使模型能生成与前后关键姿态显著不同或超出其范围的姿态
- **Blending Gate** β_t = σ(MLP_β(h_t)) ∈ [0,1]^D，在**帧级与每个姿态维度**上自适应决定两条路径贡献
- 最终 p_pred_t = (1−β_t) ⊙ p_interp_t + β_t ⊙ p_synth_t

配套 **Domain-Based Algorithm** 处理输入关键姿态调度：生产环境中的 block keyframe 并非随机分布，而是编码运动语义意图的特定姿态；由于实际动画文件不显式存储 block keyframe，采用 Miura et al. [2014] 的关键姿态提取（多频率尺度上找运动速度局部极小值）作代理，并做随机增删与索引偏移增强。

**结果**：生产数据上达SOTA；**集成进 Autodesk Maya 后，动画师完成补间任务提速 3.5×**。这是本届少见的直接给出生产效率提升数字的工作。

---

### 2.15 LayerInbetween: Occlusion-Aware Stroke Correspondence and Inbetweening with Automatic Layering

- 作者：Haoran Mo、Zhongyue Guan、Yixin Hu、Zeyu Wang（机构未找到）
- 官方 session 标签仅 Animation / Simulation ｜ **无 arXiv / 项目页 / DOI 未确认**

2D 手绘动画的遮挡感知矢量笔画对应与自动补间，并自动完成图层化以减少人工。官方 highlight 只有一句方法声明，具体网络与表征未公开。**边界说明**：对象是 2D 矢量笔画而非 3D 动作序列，若你的综述严格限定 3D 人体运动可剔除。

---

## Session D：Humans & Hands – Reconstruction

这一组属于动作数据获取的上游，对 motion generation 的意义在于数据来源质量。

### 2.16 TwinPose: Person-Specific Subspaces for Multi-View 3D Pose Estimation
- 作者：Wenwu Yang、Tianyi He、Jiwei Ding、Xun Wang、Rong Zhang（浙江工商大学）、Kun Zhou（浙大）｜ Journal ｜ DOI：10.1145/3811316
- 复杂场景下稀疏多视角输入的**实时多人 3D 动捕**。构造 **instance-aware "twin poses"** 统一 2D 姿态语义与多视角几何一致性，配 person-specific subspaces。对遮挡与密集交互稳健且 **detector-agnostic**。无 arXiv/项目页，具体数值未公开。

### 2.17 AMOR: Airborne Motion Reconstruction via Homotopy-Aware Trajectory Optimization
- 作者：Chanha Kim\*、Jungdam Won†（首尔大学 IMO Lab）｜ Conference（42:1–42:10）｜ DOI：10.1145/3799902.3811070
- 现有 HMR 方法在**空中动态动作**（腾空、特技）上不稳定不真实。做法是精修：先选择可靠运动片段，再应用 **homotopy-aware轨迹优化**强制物理一致性。在 in-the-wild 与挑战性视频上真实性与稳定性优于先前方法。数值未公开。

### 2.18 FLASHand: Feed-forward reLightable and Animatable Single-view Hand Reconstruction
- 作者：Ling-Xiao Zhang、Lin Gao（中科院计算所 / 国科大）等，另有华南理工、华东师大、Cardiff ｜ Conference ｜ DOI：10.1145/3799902.3811039
- **首个前馈**从单张 RGB 重建可重光照、可驱动 3D 手部 avatar 的模型。用 **NIMBLE 手部先验 + mesh-based 2D Gaussian Splatting**，外观解耦，支持实时动画与重光照。数值未公开。

### 2.19 AGILE: Hand-object Interaction Reconstruction from Video via Agentic Generation
- 作者：Jin-Chuan Shi、Binhong Ye 等（浙大），Chunhua Shen（浙大 / 浙工大）｜ Conference ｜ DOI：10.1145/3799902.3811134
- 既有方法两大障碍：依赖 neural rendering 在重遮挡下产出碎片化、**非 simulation-ready** 的几何；依赖脆弱的 SfM 初始化在 in-the-wild 上频繁失败。AGILE 把范式从"重建"转向 **agentic generation**，产出完整可仿真的物体 mesh，**无需 SfM** 稳健估计物体位姿。对交互学习的数据供给有价值。

### 2.20 EgoForce: Forearm-Guided Camera-Space 3D Hand Pose from a Monocular Egocentric Camera
- 作者：Christen Millerdurai（DFKI / MPI-INF）、Shaoxiang Wang、Yaxu Xie（DFKI）、Vladislav Golyanik（MPI-INF）、Didier Stricker、Alain Pagani（DFKI）｜ Conference ｜ DOI：10.1145/3799902.3811047
- 以往手部姿态估计"只看手"，只能输出以手腕为原点的相对坐标，无法给绝对位置；单目有深度-尺度歧义；AR/VR 头显相机视角宽、鱼眼畸变强且型号多样。EgoForce 利用**前臂线索 + camera-aware ray-space lifting** 在相机空间恢复姿态、形状与绝对位置，跨相机模型与设备配置，面向轻量智能眼镜。

---

## Session E：Faces & Avatars

EchoAvatar 见第一部分。其余：

### 2.21 SHELLS: Topologically Consistent Multi-view 3D Head Reconstruction via Coarse-Guided Layered Surface Sampling
- 作者：Timo Bolkart、Daoye Wang、Prashanth Chandran（Google）｜ Conference ｜ DOI：10.1145/3799902.3811201
- **18k 顶点 3D 头部、0.08 秒**，比 SOTA **快 3.5×、GPU 显存少 88%**。用 projective surface-aware feature sampling 聚合 DINOv2 特征，Transformer 预测稠密语义 mesh。属面部但核心是**静态几何重建**，非动作生成，严格限定 motion 时可剔除。

### 2.22 Learning a Delighting Prior for Facial Appearance Capture in the Wild
- 作者：Yuxuan Han、Xin Ming、Tianxiao Li、Zhuofan Shen、Feng Xu（清华）、Qixuan Zhang、Lan Xu（上科大）｜ Journal ｜ DOI：10.1145/3811303
- 从随手拍的手机视频做高质量面部外观捕获。核心是外观/材质（delighting），非面部动作。**建议剔除**，此处登记以示穷尽。

### 2.23 See-through: Single-image Layer Decomposition for Anime Characters
- 作者：Jian Lin、Chengze Li（圣方济各大学）、Haoyun Qin（UPenn / Spellbrush）等 ｜ Conference ｜ DOI：10.1145/3799902.3811209
- 把静态动漫插画自动转为**可操控的 2.5D 模型**，分解为最多 19 个完全补全的语义图层（头发、脸、眼、衣、饰品）并推断绘制顺序。与角色驱动相关但本体是图像分层，**边缘相关**。

---

## Session F：4D Humans

### 2.24 StudioRecon: 4D Human-Scene Reconstruction from Low-Overlap Captures
- 作者：Minhyuk Hwang、Sangmin Kim、Seunguk Do、Daneul Kim、Jaesik Park ｜ Conference ｜ DOI：10.1145/3799902.3811165
- 用少量**低重叠**相机重建可自由探索的动态人-场景空间。指出视频扩散模型方案对人体几何不一致，做法是**解耦背景与人体**的重建流水线。细节未公开。

### 2.25–2.28 边缘条目（登记）
| 论文 | DOI | 说明 |
|---|---|---|
| Implicit Surface Compression — with DCT and Motion Compensation | 10.1145/3811298 | motion 指视频编码的运动补偿，**建议剔除** |
| ATGS: Anchored Temporal Gaussian Splatting for Long Volumetric Video | 10.1145/3811306 | 长体积视频表示/压缩，边缘 |
| VFAvatar: Feed-Forward 3D Avatar Reconstruction from Casual Image Collections | 10.1145/3799902.3811171 | Avatar 静态重建，边缘 |
| High-Fidelity 4D Cloth Capture Pipeline with a Two-Level Pattern | 10.1145/3811305 | 4D 服装捕获，动态但对象是布料，边缘 |

---

## Session G：Digital Humans & Virtual Try-On

### 2.29 DreamActor-M2: Universal Character Image Animation via Spatiotemporal In-Context Learning
- 作者：Mingshuang Luo（字节 / 中科院计算所）、Shuang Liang、Yuxuan Luo、Zhengkun Rong、Tianshu Hu、Yuan Zhang、Mingyuan Gao（字节）、Ruibing Hou、Hong Chang（计算所）、Yong Li（东南大学）｜ Conference ｜ DOI：10.1145/3799902.3811114
- 把 **motion conditioning 视为时空 in-context learning**，利用视频基础模型先验，实现从原始视频端到端的**pose-free 动作迁移**，**无需显式姿态估计**。跨复杂场景强泛化。视频扩散连续 latent，无动作量化。这个"去掉显式 pose 中间表示"的取向与 Motion4Motion 同向。

### 2.30 MACE-Dance: Motion-Appearance Cascaded Experts for Music-Driven Dance Video Generation
- 作者：Kaixing Yang、Ziqiao Peng、Puwei Wang、Jun He（人大）、Jiashu Zhu、Jiahong Wu、Xiangxiang Chu（阿里 AMAP）、Xulong Tang（Malou Tech）、Xiangyue Zhang（武大）、Hongyan Liu（清华）｜ Conference ｜ DOI：10.1145/3799902.3811202
- **级联 MoE**：motion expert 先从音乐生成 3D 舞蹈动作，appearance expert 再把参考图像动画化为时间一致视频。提出 **MA-Data** 大规模数据集与评测协议。
- ⚠️ **待你补查**：3D 舞蹈动作阶段是否使用 VQ 类表征**无公开证据**。该子领域（音乐驱动舞蹈）常用 VQ-VAE，若确实用了，这会是本届第五篇 tokenizer 相关工作。建议追一下 arXiv。
- 注意作者 Kaixing Yang、Xulong Tang 同时是 SA2026 的 CustomDance 作者，是同一条舞蹈生成研究线。

### 2.31–2.33 边缘条目
| 论文 | 作者/DOI | 说明 |
|---|---|---|
| Composing People Together: Iterative Pose-Image Generation for Multi-Person Interaction Scenes | Wenxuan Peng、Bharath Hariharan、Hadar Elor（Cornell）｜10.1145/3799902.3811129 | 联合建模人体姿态与外观、迭代组合场景，产出**静态图像**，边缘相关 |
| HumanFlow: Controllable Human Image Generation via Flow Matching | 武大Wenzhuo Fan 等 ｜10.1145/3811361 | flow matching + Control Encoder + **Token-ControlNet**（注意这里的 token 是图像 patch token，**不是运动码本**）+ MiCoGen 数据集；静态图像生成，建议剔除 |
| FabricTryOn: Taming Image Editing Models for Garment Re-Texturing | 10.1145/3799902.3811105 | 纯图像编辑，剔除 |

---

##跨 Session 的 motion 相关条目

### 2.34 AniGen: Unified S³ Fields for Animatable 3D Asset Generation
- 作者：Yi-Hua Huang\*、Chirui Chang、Ziyi Yang、Xiaojuan Qi†（HKU）、Zi-Xin Zou、Yuan-Chen Guo、Yan-Pei Cao†（VAST）、Yuting He（CUHK）、Cheng-Feng Pu（清华）｜ Session：3D Generation ｜ Journal ｜ DOI：10.1145/3811297
- arXiv：https://arxiv.org/abs/2604.08746 ｜**代码已开源**：https://github.com/VAST-AI-Research/AniGen ｜在线试玩：HF Spaces

3D 生成模型能合成视觉合理的形状但结果是**静态**；事后 auto-rigging 脆弱且常产生与生成几何拓扑不一致的骨架。**S³ Fields（Shape, Skeleton, Skin）** 把三者统一为定义在共享空间域上的互相一致的**连续场**——骨架**不表示为离散图**而是稠密向量场，蒙皮**不表示为稀疏矩阵**而是对偶特征场。配 confidence-decaying skeleton field 处理 Voronoi边界歧义，Dual Skin Field + 预训练 SkinAE 把可变基数 skinning 转为固定维度 latent（使固定架构能预测任意关节数的 rig）。两阶段 flow matching（基于 TRELLIS structured sparse latents）。

在 ArticulationXL 上与 TRELLIS\*+UniRig / Anymate / Puppeteer / RigAnything 对比，骨架结构预测与蒙皮精度均最佳，**Gromov-Wasserstein 距离**（骨架拓扑正确性）与 Skin KL 领先明显（数值未公开）。

**这篇与 SkinTokens 形成绝佳对照**：同一批人（VAST 线）在"结构表征该连续还是离散"上给出了两个相反答案——AniGen 用连续场，SkinTokens 用 FSQ 离散 token。如果你在写tokenizer 相关的related work，这对比很有分量。

### 2.35 SimArt
见1.5。

### 2.36 Go-with-the-Track: Video Compositing and Motion Control with Point Tracking
- 作者：Koichi Namekata（Eyeline Labs / Oxford）、Yash Kant、Yuancheng Xu、Emmett Steven、Paul Debevec、Ning Yu（Eyeline Labs / Netflix）等 ｜ Conference ｜ DOI：10.1145/3799902.3811093
- 以 **point tracks** 为运动条件，多参考图像联合锚定生成帧。解锁 video restylization、**keypoint- 与 mesh-driven compositing**、相机运动控制。运动控制针对视频像素/点轨迹，边缘相关。

### 2.37 ActCam: Zero-Shot Joint Camera and 3D Motion Control for Video Generation
- 作者：Omar El Khalifi、Thomas Rossi、Baptiste Bellot-Gurlet（Kinetix）、Ulysse Mizrahi（Kinetix / TAU）、Philip Torr（Oxford）、Ivan Laptev、Fabio Pizzati（MBZUAI）｜ Conference ｜ DOI：10.1145/3799902.3811224
- **zero-shot、免训练**，联合控制相机轨迹与**角色表演（character acting）**。设计上刻意贴合传统摄影技法以融入既有艺术工作流，可跨不同 backbone 架构。作者单位 Kinetix 是动作技术公司，明确含 3D motion control。

### 2.38 Reality Check: How Avatar and Face Representation Affect the Perceptual Evaluation of Synthesized Gestures
- 作者：Haoyang Du、Yinghan Xu、John Dingliana、Brian Keegan、Rachel McDonnell、Cathy Ennis ｜ Session：Perception ｜ Conference ｜ DOI：10.1145/3799902.3811161
- **合成 co-speech gesture** 的感知评估中，avatar 与面部表征方式如何**偏置**动作判断。跨多种motion source、**7 种代表性 avatar 渲染方式**的受控评估。
- **强烈建议保留**：如果你的工作要做 user study，这篇直接关系到你的评测协议是否有效——渲染方式选错，主观结论可能是渲染带来的而非动作质量带来的。

### 2.39 PEAR: Pixel-aligned Expressive HumAn Mesh Recovery
- 作者：Jiahao Wu、Yunfei Liu、Lijian Lin、Ye Zhu、Lei Zhu、Jingyi Li、Yu Li ｜ Session 未确认 ｜ DOI：10.1145/3799902.3811096
- 快速稳健的像素对齐**表达性人体网格恢复**（含手部与面部），强调速度与稳健性以便下游任务使用。细节未公开。

---

## 明确剔除清单（登记以示检索穷尽）

| 论文 | Session | 剔除理由 |
|---|---|---|
| SmoothMotionVectors | Dynamic Scenes | 视频编码 motion vector |
| Computational Design of Coordinate-Motion Assemblies | Computational Design | 机械装配体运动 |
| Kinematic Kitbashing | Computational Design | 铰接式3D 物体合成，可作articulation 上游边缘参考 |
| mpcGear | Computational Design | 齿轮机构 |
| Computational Design of Terrestrial Robots with Anisotropic Friction | Computational Design | 机器人本体设计（含运动），非动作生成 |
| Sketch2Arti | Vector Graphics | CAD 物体关节建模 |
| LoBoFit: Flexible Garment Refitting via Local Bone Mapping Blending | Garments | 服装重适配（用骨骼映射），边缘 |
| Progressing LOD Animation for Volumetric Elastodynamics | Simulation | 体积弹性动力学 LOD |
| MPM Lite / Mixed MPM / M-ABD 等 | Simulation | 通用物理求解器|
| StyleID | Perception | 面部身份识别 |
| Role-Aware Virtual Agents ... Multimodal LLM | VR/Haptics | 虚拟 agent 导航交互，偏 HCI |
| EgoRelight / BodyReLux / Pixel Cube 等 | 多 session | 人体重光照/新视角合成 |

**两个重要的负面结果**：
- **群体动画（crowds）：本届零命中**。327 篇标题与 highlight 全文检索无任何 crowd/crowds 命中，官方 session 列表也无相关 session。
- **四足/人形机器人运动无独立 session**，相关内容分散在 ReActor（真机人形+四足）、MotionBricks（Unitree G1 + LaFAN1-G1）、SMP（Unitree G1）、GPC（仿真）、TopoCap（跨拓扑四足/六足）。官方给 sess104 与 sess114 都打了 Robotics 关键词——这是本届"图形学与机器人融合"叙事的直接体现（官方新闻稿也以此为主题）。

---

# 第三部分 · SIGGRAPH Asia 2026 逐篇总结

**必读的前提说明**：以下清单**必然不完整**。SA2026 官方只公布 paper ID，无标题；条件接收在 2026-07-18~22 才通知；官方匿名政策禁止评审期内标注venue，导致 arXiv comment 字段几乎无命中（全库仅 3 条且均非 motion）。每一条都注明了证据来源，请按需自行复核。

## 3.1 已确认（有明确书面证据）

### 1. Learning Kinematic Frequency-Aware Disentanglement for Motion Style Transfer and Editing
- **作者**：Ran Dong（日本中京大学）、Haoran Xie（JAIST）、Xi Yang（吉林大学人工智能学院，通讯）
- 项目页/ arXiv：**未找到** ｜ 轨道未标注
- **问题**：运动风格同时纠缠粗粒度因素（姿态、轨迹）与细粒度因素（局部时序、次级运动、接触细节、节奏重音）。现有方法能改变表观风格，却难以迁移与控制微妙的运动细节。
- **方法**：**运动学频率感知扩散模型**。学习一个"运动学频率提取器"，在基于重构的**潜在扩散迁移框架**内实现受频率监督的粗/细风格分离；利用高、中、低频运动编码支持细粒度风格可控编辑。→ 连续 latent + diffusion，**无离散 token 迹象**
- **结果**：在保持内容结构前提下提升精细风格识别、运动质量与细粒度控制力；另发布**运动-文本基准数据集**（用于在显式内容/风格分离条件下评测细节可控风格迁移）。无具体数值。应用含角色级风格迁移、运动学细节夸张、舞蹈风格编辑。
- **证据**：https://sai.jlu.edu.cn/info/1026/5470.htm（"被 SIGGRAPH ASIA 2026 条件接收"）
- **可对比**：与 SIGGRAPH 2026 的 HyperLoRA、STyMo构成"风格迁移三种路线"——频率解耦 / hypernetwork-LoRA / few-shot 静态-时序分解。三篇放一起读会很有收获。

### 2. CustomDance: Customized 3D Dance Generation with a Coarse-to-Fine Human-Centered Interactive System
- **作者**：Xulong Tang、Kaixing Yang、Xiaohu Guo、Balakrishnan Prabhakaran、Rawan Alghofaili（主要为 UT Dallas HeXD Lab）
- **Conference Papers**（BibTeX 明写）｜项目页：https://xulongt.github.io/customdance-project-page
- **问题**：3D 舞蹈生成缺乏对多模态输入（音乐、具体动作描述）的全面且可区分的控制；生成动作统计上合理但缺乏深度、表现力与创作者意图对齐。
- **方法**：受真实编舞工作流启发的**由粗到细三阶段交互系统**：① Choreographic Motif Planning（MLLM Planner 依音乐与全局意图给时间轴 anchors + 局部创意 cues）；② Dance Phrase Generation（Retriever 从真实舞蹈短语库做**短语级检索**，支持局部文本与身体部位控制）；③ Completion and Refinement（Completer / Diagnoser / Remaker 做 diffusion 过渡补全与局部修复）
- **表征值得注意**：这是**检索（离散短语单元）+ 连续 diffusion 补全的混合路线**。虽然不是 VQ 码本，但"用真实动作短语库作为离散基元"在功能上与 token 词表有相似之处——**属于离散化的另一条路径：不学码本，直接用数据本身作码本**。对你思考 tokenizer 设计空间有参照价值。
- **结果**：仅定性演示（Popping含 "Fresno" 基本动作、汉唐风格上肢舒展）。无量化指标，代码未找到。
- **证据**：项目页顶部标注 + BibTeX `Proceedings of the SIGGRAPH Asia 2026 Conference Papers`；实验室页 https://hexd-lab.github.io/ 标注 conditionally accepted

### 3. UniMate: One Unified Model to Animate Diverse Skeletons
- **作者**：Linzhan Mou、Jiahui Lei、Zhiyang Dou、Chenyue Cai、Chaoyue Song、Adam Finkelstein、Szymon Rusinkiewicz（Princeton / UC Berkeley / MIT / NTU）
- 项目页：https://linzhanmou.com/unimate/ ｜PDF 直链有 ｜arXiv 未找到 ｜代码：https://github.com/Friedrich-M/UniMate（MIT，训练代码与数据集标 TODO）
- **问题**：自动 rigging 已能大规模产出 animation-ready 资产，但驱动它们的运动生成仍是瓶颈。现有学习式 animator 受**拓扑约束**：依赖类别专属模板、或需逐骨架微调、或推理时需参考运动。
- **方法**：**拓扑感知扩散 Transformer**，三种机制把骨架拓扑注入attention：① graph-aware attention bias（关节两两关系 + 测地距离）；② **spectral rotary position embedding**（借图拉普拉斯把 RoPE 推广到任意运动学树）；③ global topological conditioner（从 rest-pose 骨架 attention-pool 得到）。数据侧：BFS 序列化关节、按拓扑直径缩放、规范坐标系、旋转相对 rest pose 表达。→ 连续特征 + diffusion，**明确无 VQ / 码本**；"Motion Expansion" 是多提示串联续接，非 token 级自回归
- **数据**：**UniML3D，13,006 条文本配对运动序列**，覆盖 bipedal / quadrupedal / avian / marine / insectoid / serpentine / articulated rigid objects；融合 Truebones + Mixamo + Objaverse-XL，30 FPS 四路同步视图 + 多模态 LLM 生成 caption + 人工复核
- **结果**：称在质量、泛化性、效率三方面优于 SOTA；支持 zero-shot 跨拓扑迁移、in-betweening、motion expansion、文本引导编辑；实时、无测试时优化。**未给任何数值指标与 baseline 名称**
- **证据**：一作主页标注 "SIGGRAPH Asia 2026 (Conditionally Accepted)" + GitHub News "[2026-07-18] accepted"
- **与 SIGGRAPH 2026 的 TopoCap 直接对位**：两篇都在解"任意骨架拓扑"，TopoCap 走 Graph CVAE + flow matching 做 video-to-animation，UniMate 走拓扑感知 DiT 做 text-to-animation，且都自建了跨拓扑数据集（Mobjaverse 5,000+ 拓扑 / 200 万帧 vs UniML3D 13,006 序列）。这是今年"拓扑无关运动先验"的两篇必读。

### 4. ProAct: Harnessing Streaming Motion Generation and Agentic Reasoning for Realtime Embodied Social Interaction
- ⚠️ 标题有两个版本：主页/项目页版如上；arXiv v1 版为 "ProAct: A Dual-System Framework for Proactive Embodied Social Agents"。最终题名待正式出版确认。
- **作者**：Zeyi Zhang\*、Zixi Kang\*、Ruijie Zhao\*、Yusen Feng、Biao Jiang、Hanyu Ji、Libin Liu†（北京大学智能学院 刘利斌组PKU-MoCCA）
- **Journal Track**（主页原文 "ACM SIGGRAPH Asia 2026 Journal Track. (To appear)"）｜项目页：https://proactrobot.github.io/ ｜arXiv：https://arxiv.org/abs/2602.14048 ｜代码 TBD
- **问题**：具身社交智能体多为被动反应式，仅响应短时窗内当前感知；主动社交行为需长时程上下文推理与意图推断，与实时交互的严格延迟预算冲突。
- **方法**：**双系统框架**——低延迟 **Behavioral System**（streaming omni-modal LLM 级联 streaming motion generator，言语走 turn-based、动作持续运行，二者异步）+ 较慢 **Cognitive System**（Context Encoder 压缩历史为 bounded memory；Behavior Planner 预测用户动机并输出高层主动意图）。
  - 动作侧核心：**Conditional Flow Matching**（最优传输路径 + Transformer backbone）；**overlap-and-cache** 机制消除窗口边界不连续以实现 streaming；**disentangled ControlNet** 把文本控制与冻结的音频驱动基础生成器解耦，支持异步意图注入与反应式↔主动式手势无缝切换
  - → 连续 flow matching，**明确无 VQ / 码本**；自回归成分在语言侧而非动作侧
- **结果**：已部署于**真实物理人形机器人**；提出新基准 **ProActBench**（评测主动触发检测与克制）；真实用户研究中参与者与旁观者在感知主动性、社会临场感、整体参与度上一致偏好 ProAct 优于反应式变体；消融显示 DDIM baseline 推理延迟高。无 FGD/FID 数值，机器人型号与训练数据规模未找到。
- **证据**：刘利斌主页 https://libliu.info/ 标注 Journal Track，News "2026/07/20: one paper accepted to ACM SIGGRAPH Asia 2026"（全站唯一一篇）
- **与 EchoAvatar 的对位很有意思**：两篇都做"流式音频驱动动作 + LLM"，但 EchoAvatar 走 **RVQ 离散 token + Qwen 自回归**，ProAct 走 **连续 CFM + 语言侧 LLM**。同一个应用场景下两条表征路线的直接对照，值得放一起分析延迟/质量/可控性的权衡。

### 5. STEER: Steerable Dyadic Head Avatars
- **作者**：Kartik Teotia、Helge Rhodin、Hyeongwoo Kim、Marc Habermann、Christian Theobalt（MPI-INF）
- **Conference** ｜项目页：https://kartik-teotia.github.io/STEER ｜**推理代码已开源**：https://github.com/Kartik-Teotia/STEER
- **问题**：语音驱动人脸动画多把对话行为当作"音频的涌现副产品"，只暴露粗粒度序列级情感控制，导致注视接触/回避、节律性头部运动、情绪等关键非语言通道难以显式控制；且公开双人语料缺乏时间对齐的行为标注。
- **方法**：可控 3D **双人（dyadic）** 动作先验，三阶段：① 自建 tracking + annotation pipeline 从 in-the-wild 双人视频提取行为伪标签（面部追踪、头部与眼部姿态、Action Units、粗粒度情绪、运动学分解）；② **causal flow-matching transformer**，条件为目标音频 + 伙伴动作 + 情绪 + 行为控制量，输出 partner-aware 目标动作；③ Avatar bridge 把追踪参数映射到 **Universal Gaussian Head-Avatar Prior** 驱动空间，底层高斯 avatar **冻结无需重训**。显式控制三轴：gaze / head rhythm / emotion。连续 flow matching，无量化。
- **结果**：motion quality、dynamics、diversity 上优于近期dyadic baselines；partner coupling 上"保持竞争力"（非最优，作者交代得很诚实）；支持交互式实时部署（**partner-aware 对话 90fps**）。数据统计：过滤后 **10,555 片段**（训练 9,790 / 测试 765）。无 FID/LVE 数值。
- **证据**：项目页标注 "SIGGRAPH Asia 2026 Conference" / 页脚 "Kuala Lumpur"；MPI GVDH 出版页标注；GitHub About

### 6. UMA: Ultra-detailed Human Avatars via Multi-level Surface Alignment
- **作者**：Heming Zhu、Guoxing Sun、Christian Theobalt、Marc Habermann（MPI-INF Saarland / VIA）
- **Journal Track（ACM TOG）**｜项目页：https://vcai.mpi-inf.mpg.de/projects/UMA/ ｜ACM DL：https://dl.acm.org/doi/10.1145/3829365 ｜**代码 + 数据集 + 在线 Demo 均已开放**
- **问题**：可驱动着装人体 avatar 在 4K+ 分辨率、相机拉近时细节不足；根因是表面追踪不准（深度错位、表面漂移），迫使外观模型去补偿几何误差；且服装动态具随机性，无法仅由骨骼姿态解释。
- **方法**：输入**骨骼运动 + 相机视角**，输出高保真几何与外观。可驱动模板作几何骨架 + 注入**逐帧可学习连续隐编码 z_f**（建模骨骼无法解释的服装随机动态，测试时置零）；texel 超分模块稠密化可动画Gaussian textures。核心贡献 **multi-level surface alignment**：借基础 2D 视频点追踪器在顶点级与 texel 级监督 3D 形变，用级联训练把2D 点轨迹锚定到渲染 avatar 以生成一致 3D 点轨迹。连续 latent + 姿态驱动条件回归，非生成式 token 建模。
- **结果**：渲染质量与几何精度显著优于此前 SOTA；支持自由视点渲染、纱线级纹理细节数字变焦、时间上共享三角化的精细褶皱、**motion retargeting**、纹理编辑。新数据集：5 位受试者、**40 台 6K 相机**、light stage、每序列 >10 分钟（约 7.6 万训练帧 + 4.1 万测试帧）。
- **归类说明**：严格说是"给定动作后的外观/几何合成"，非动作生成。
- **证据**：MPI GVDH 出版页 "ToG (Siggraph Asia) 2026"

### 7. OMEGA-Avatar: One-shot Modeling of 360° Gaussian Avatars
- **作者**：Zehao Xia、Yiqun Wang\*（重庆大学）、Zhengda Lu、Jun Xiao（中科院）、Kai Liu（重庆大学）、Peter Wonka（KAUST）
- 项目页：https://omega-avatar.github.io/OMEGA-Avatar/ ｜arXiv：https://arxiv.org/abs/2602.11693 ｜代码 coming soon
- **问题**：单图生成高保真可动画 3D avatar，现有工作只能同时满足"前馈 / 360° 全头 / animation-ready"三者中的两个。
- **方法**：首个同时满足三者的单图 3D 高斯头部框架。两个新模块：**semantic-aware mesh deformation**（融合多视角法线优化带头发的 FLAME 头部，保持拓扑）与 **multi-view feature splatting**（可微双线性 splatting + 分层 UV 映射 + 可见性感知融合，构建共享 canonical UV 表示，实现 360° 一致性且无需逐实例优化）。驱动方式是从目标图像提取 FLAME 表情与姿态**连续参数**注入形变网格，最终经 neural refiner 增强。无离散 token。
- **结果**：360° 全头完整性显著优于 baselines，跨视角稳健保持身份；支持 Expression Reenactment 与 Animatable Full-Head Reconstruction。项目页无量化数值。
- **证据**：项目页标题下方标注 "SIGGRAPH Asia 2026" + BibTeX

### 8. HyperBones: Realtime Bone-driven Neural Garment Simulation with Hypernetwork Conditioning
- **作者未找到**（实验室新闻仅给标题）｜单位：Avinash Sharma 教授组（IIT Jodhpur CSE，兼 IIIT Hyderabad CVIT）
- 项目页 / arXiv / 代码：**均未找到**
-骨骼驱动的实时神经服装仿真，仅知使用 **hypernetwork conditioning**，其余全部未知。属骨骼驱动的次级运动，非人体动作生成本体。
- **证据**：https://3dcomputervision.github.io/ 新闻 "HyperBones Accepted at SIGGRAPH Asia 2026"（July 2026）

## 3.2 待确认（证据不足，请勿直接引用）

### 9. MoCapAnything V2: End-to-End Motion Capture for Arbitrary Skeletons
- **作者**：Kehong Gong、Zhengyu Wen、Dao Thien Phong 等 13 人（单位未找到）
- 项目页：https://animotionlab.github.io/MoCapAnythingV2/ ｜arXiv：https://arxiv.org/abs/2604.28130 ｜代码（**自述非官方复现**）：https://github.com/phongdaot/MocapAnything（MIT，317★）
- **方法**：首个 Video-to-Pose 与 Pose-to-Rotation **均可学习、联合优化**的端到端动捕框架。关键洞察：pose→rotation 的歧义源于缺失坐标系信息，故引入目标资产的**参考pose–rotation 对**，配 rest pose 锚定映射并定义旋转坐标系，把旋转预测变为受良好约束的条件问题；直接从视频回归关节位置，**不依赖 mesh 中间表示**；两阶段共享 **GL-GMHA**（Global-Local Graph-guided Multi-Head Attention）。回归式连续输出，无量化。
- **结果**：Truebones Zoo 与 Objaverse 上旋转误差**由约 17° 降至约 10°**，未见骨架上达 **6.54°**；推理比 mesh-based 流程**快约 20 倍**。权重已发布，HF Spaces 有在线 Demo。附加应用 "Dance Anything"。
- **⚠️ 为何待确认**：SA2026 标记**仅出现在这个自述"非官方"的第三方仓库**的About。arXiv comment 只有项目页链接、未提任何 venue；官方项目页静态抓取无venue 信息。前作 MoCapAnything V1 已发表于 **CVPR 2026**。

### 10. KAIST Visual Media Lab（Junyong Noh 组）：3 篇 SIGGRAPH Asia，标题全未知
- 证据：Seokhyeon Hong 主页 News "[Jul. 2026] 3 papers accepted to SIGGRAPH Asia. (1 first-authored, 2 co-authored)"，**但未写年份**
- 该主页 Publications 区 2026 年条目全部标注 SIGGRAPH 2026 或 CVPR 2026，无 SA2026 条目；已交叉核对同组 Kwan Yun、Youngseo Kim 主页，均无
- Hong 本人方向为角色动画的 generation / editing / in-betweening / retargeting / rigging，该组2024 年曾有 SA2024 TOG 论文 "Geometry-Aware Retargeting for Two Skinned Characters Interaction" → **这 3 篇很可能含 motion 相关**，但目前无任何标题证据
- 建议：查该作者 CV（assets/cv.pdf）或 Google Scholar（user=cmuRDa8AAAAJ）

## 3.3 易误收的排除记录

以下在检索中反复出现、容易被误归入 SA2026，均已核实归属其他会议：**ARDY / GPC / SMP / MotionBricks / ReActor / MOCHI / AMOR / HyperLoRA / Skinned Motion Retargeting → SIGGRAPH 2026**；**Uni-Inter、CFC（HKU Komura 组）→ SIGGRAPH Asia 2025**；**WHIP → ECCV 2026**；**EmbodMocap、OpenDance（PKU 刘利斌组）→ CVPR 2026**；**DMP: Directable Motion Retargeting through Motion Paraphrasing → ACM TOG（未标 SIGGRAPH）**；**MoTok: Bridging Semantic and Kinematic Conditions with Diffusion-based Discrete Motion Tokenizer（NTU S-Lab / Ziwei Liu 组）→ 仅 arXiv 预印本，无 venue**（这篇标题正对你的关注点，但注意它不是会议论文）。

## 3.4 SA2026 覆盖度自评

| 子领域 | 找到的条目 |
|---|---|
| human motion generation（文本/音乐/语音驱动） | CustomDance（音乐+文本）、UniMate（文本）、ProAct（语音驱动手势） |
| **motion tokenizer / 离散运动表征** | **0 篇** |
| physics-based character control | **0 篇** |
| motion capture / estimation | MoCapAnything V2（待确认） |
| motion editing / style transfer / retargeting | #1 风格迁移、UniMate（editing/in-betweening）、UMA（retargeting 应用） |
| 手部与面部动作 | STEER（头部/注视/情绪）、OMEGA-Avatar（表情驱动）；**手部 0 篇** |
| gesture / speech-driven animation | ProAct、STEER |
| 四足与人形机器人运动 | UniMate（非人形骨架）、ProAct（真机人形部署）；**纯 locomotion 0 篇** |
| 群体动画 / crowd | **0 篇** |
| 4D / 动态角色 | UMA、OMEGA-Avatar、HyperBones |

**已尝试的检索路径**（供你判断遗漏风险）：arXiv API五种字段组合（`all:` / `co:` / `abs:` / 日期范围，全库 `"SIGGRAPH Asia 2026"` 仅 3 条命中且均非 motion）；GitHub Repository Search 四种查询（14 个仓库，motion 相关 7 个）；网页搜索约 15 轮 60+ 关键词（含中/日/韩文）；定向抓取 20+ 个实验室与个人主页。**确认无 SA2026 motion 条目**的重要实验室：HKU CG（Taku Komura，2026 年 9 篇全部非 SA）、SNU Intelligent Motion Lab（Jungdam Won，4 篇全非 SA）、Awesome-Human-Motion 社区列表（81 处 SIGGRAPH 提及，"Asia 2026" 出现 0 次）。

**受限路径（可能遗漏）**：官方 paper ID 无法反查标题；Twitter/X 无法直接检索；GitHub Code Search API 未使用（可能遗漏只在 README 正文提及的项目）；微信公众号基本无法被索引，中文高校新闻稿只命中吉林大学一所。

---

# 第四部分 ·趋势观察与研究建议

## 4.1 三条主线

**第一条：离散化路线在物理控制与实时系统里站住了脚。**

值得注意的是，本届四篇 tokenizer 工作的落点全部偏向**系统性能而非生成质量**：GPC 要的是可复用的物理技能库、MotionBricks 要的是 15,000 FPS、ARDY 要的是训练稳定性与 33 ms 延迟、EchoAvatar 要的是流式纠错能力。反过来，纯追求生成质量与可控性的工作（SMP、MultiAct、HyperLoRA、以及 SA2026 全部）几乎清一色选择连续 diffusion / flow matching。

这个分布本身就是个信号：**离散 token 的当前主要价值可能在于"把运动变成可被LLM 式架构高效处理的序列"，而不是"更好地建模运动分布"**。ARDY 的 AE/VAE/FSQ 三方对照（性能相当）几乎是这个判断的直接实验证据。

**第二条：拓扑无关（topology-agnostic）成为独立议题。**

TopoCap（SIGGRAPH，Graph CVAE + flow matching，Mobjaverse 5,000+ 拓扑 / 200万帧）、UniMate（SA，拓扑感知 DiT，UniML3D 13,006 序列）、ACT（SIGGRAPH，rigging 与 animation 统一 latent）、AniGen（SIGGRAPH，S³连续场）、MoCapAnything V2（任意骨架动捕）——五篇独立工作在同一年攻同一个问题，并且都自建了跨拓扑数据集。这个方向在 2026 年正式形成了共同的问题定义与评测基础。

有意思的是，这个方向的工作**几乎全部拒绝离散化**，而且 TopoCap 给出了明确理由（"结构离散、运动连续"）。如果你要在tokenizer 方向上拓展到任意骨架，这是必须回应的反方论证。

**第三条：图形学与机器人的界限在这一届被官方叙事正式打破。**

官方新闻稿以"机器人"为主题；sess104 与 sess114 都被打上 Robotics 关键词；MotionBricks 用同一个神经模型同时驱动游戏角色与 Unitree G1；SMP 与 GPC 的策略直接部署真机；ReActor 覆盖两款人形 + 一款四足；ProAct（SA）部署真实人形机器人。**"动作生成"与"机器人动作指令"开始共享同一份数据资产与同一套模型。**

## 4.2 给你的具体建议

**如果你在做 motion tokenizer，这几个点最有信息量：**

1. **MotionBricks 的多头 vs 单头缩放曲线**是本届最硬的一组tokenizer 设计证据（单头 VQ-VAE 很快饱和，多头能吃到 1e9 容量）。如果你的方案还是单码本，这条曲线值得先复现验证。
2. **ARDY 的混合表征**提供了一个被忽视的设计维度：不要问"要不要离散化"，要问"哪些分量该离散化"。root走显式连续、body 走离散，这个切分背后是精度需求与分布复杂度的权衡。
3. **GPC 的端到端 RL 训练量化器**是范式上的突破——从"先学重建再学控制"变成"联合学词汇与策略"，MPJPE 从78.26 降到 34.90 是相当大的差距。但注意它的 token 帧率没公开，复现时这是个坑。
4. **EchoAvatar 的 Hierarchical Token Corruption** 解决的是 RVQ 自回归特有的 conditional collapse 问题，思路（保留粗粒度底层、扰动深层）可以直接迁移到其他 RVQ + AR 的motion 系统。
5. **码本利用率**这个指标值得你常规上报：GPC 明确给出 VQ-VAE 仅 10–20% vs FSQ 82.15%。很多论文不报这个数，但它直接决定了"名义词表大小"是否有意义。

**需要你自己补查的三个缺口：**

- **MACE-Dance**（SIGGRAPH 2026）的 3D 舞蹈动作阶段是否用 VQ 类表征——该子领域常用 VQ-VAE，若确认则是本届第五篇 tokenizer 相关工作。目前无公开证据，我没有做推断。
- **GPC 的 token 帧率 / 控制频率 / 未来目标窗口 h**——正文说在附录，公开 HTML 被截断。这几个数对复现是关键。
- **MotionBricks 的最优总码本容量**——正文两处表述冲突（1e6–1e7 vs ~1e9），建议查正式版或直接问作者。

**SA2026 的跟踪节奏建议：**

① 每 1–2 周重跑 arXiv `co:"SIGGRAPH Asia 2026"`（新论文会陆续挂上，随着 camera-ready 临近会有一波）；② 盯 GitHub 新建仓库描述含 "[SIGGRAPH Asia 2026]"；③ 等 KAIST Visual Media Lab 那 3 篇标题公布（该组方向与你高度重叠）；④ 等 MoCapAnything V2 官方页更新以确认 venue；⑤ **ACM DL 的 SA2026 Conference Proceedings 与 TOG 目次通常在会前约两周（2026 年 11 月中下旬）上线**，届时可获得完整权威列表——那才是做正式综述的时点。

---

# 附录：DOI 速查表（SIGGRAPH 2026）

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
| HyperLoRA (Stylized T2M) | 10.1145/3799902.3811205 | Conf |
| MUSIC | 10.1145/3811402 | Journal |
| MOCHI | 10.1145/3811308 | Journal |
| ACT | 10.1145/3811392 | Journal |
| Motion4Motion | 10.1145/3799902.3811062 | Conf |
| STyMo | 10.1145/3811356 | Journal |
| Skinned Motion Retargeting | 10.1145/3811354 | Journal |
| ReActor | 10.1145/3811378 | Journal |
| AIS (In-Betweening) | 10.1145/3799902.3811157 | Conf |
| TwinPose | 10.1145/3811316 | Journal |
| AMOR | 10.1145/3799902.3811070 | Conf |
| FLASHand | 10.1145/3799902.3811039 | Conf |
| AGILE | 10.1145/3799902.3811134 | Conf |
| EgoForce | 10.1145/3799902.3811047 | Conf |
| EchoAvatar | 10.1145/3799902.3811066 | Conf |
| SHELLS | 10.1145/3799902.3811201 | Conf |
| StudioRecon | 10.1145/3799902.3811165 | Conf |
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
| UMA (SA2026) | 10.1145/3829365 | Journal |

---

## 数据来源与可复现性

- **SIGGRAPH 2026 官方全量日程**：`https://s2026.conference-schedule.org/wp-content/linklings_snippets/wp_program_view_all_2026-07-{19..23}.txt`，解析出 85 个 session、309 条 presentation 记录
- **论文详情页**：`https://s2026.conference-schedule.org/presentation/?id=papers_XXX&sess=sessYYY`（采集中该站启用了反爬，约 60% 详情页未能直抓，关键 session 已用搜索缓存 + arXiv + 项目页交叉补齐）
- **paperdigest 全量列表**（327 篇，含标题 + highlight + 作者 + DOI）：https://resources.paperdigest.org/2026/07/siggraph-2026-papers-highlights
- **SIGGRAPH Asia 2026 条件接收 ID 列表**：https://asia.siggraph.org/2026/images/pdfs/SIGGRAPH-Asia-2026-Conditionally-Accepted-Technical-Papers.pdf
- 技术细节均来自 arXiv 全文/HTML 与项目主页；**凡未查到的字段一律标注"未找到"，未做任何推断或编造**
