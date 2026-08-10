# SIGGRAPH Asia 2026 · Motion 相关技术论文素材清单

检索日期：2026-08-10
会议时间：2026 年 12 月 1–4 日，吉隆坡会展中心（KLCC）
官方状态：Technical Papers 投稿已于 2026-05-12 截止；官方仅公开"条件接收 paper ID 列表"（无标题）。所有标题均来自作者自行公布渠道。

**分轨说明**：SIGGRAPH Asia 2026 Technical Papers 分 Journal Track（发ACM TOG）与 Conference Track（发 Conference Proceedings）两轨，本清单尽量标注。

---

## 一、已确认为 SIGGRAPH Asia 2026（有明确书面证据）

### 1. Learning Kinematic Frequency-Aware Disentanglement for Motion Style Transfer and Editing

| 项目 | 内容 |
|---|---|
| 作者 | Ran Dong, Haoran Xie, Xi Yang（杨溪为通讯作者） |
| 单位 | 吉林大学人工智能学院（杨溪）；日本中京大学（Ran Dong）；日本北陆先端科学技术大学院大学 JAIST（Haoran Xie） |
| 轨道 | 未标注（新闻稿仅称"条件接收"） |
| 项目页 / arXiv | **未找到** |
| 核心问题 | 精细运动学层面的运动风格迁移与编辑。运动风格同时纠缠粗粒度因素（姿态、轨迹）与细粒度因素（局部时序、次级运动、接触细节、节奏重音）；现有方法能改变表观风格，却难以迁移与控制微妙运动细节 |
| 方法与运动表征 | **运动学频率感知扩散模型（kinematic frequency-aware diffusion）**。学习一个"运动学频率提取器"，在**基于重构的潜在扩散（latent diffusion）迁移框架**内实现受频率监督的粗/细风格分离；利用高、中、低频运动编码支持细粒度风格可控编辑。→ 属**连续 latent + diffusion** 路线，**无离散 token / VQ-VAE / RVQ / FSQ 迹象** |
| 关键结果 | 在保持内容结构前提下提升精细风格识别能力、运动质量与细粒度风格控制力；另发布"运动–文本基准数据集"（用于在显式内容/风格分离条件下评测细节可控风格迁移）。**无具体数值指标（新闻稿未给出）** |
| 附加应用 | 角色级风格迁移、运动学细节夸张（exaggeration）、舞蹈风格编辑 |
| 开源情况 | **未找到**（无代码/数据链接） |
| **证据来源** | https://sai.jlu.edu.cn/info/1026/5470.htm ——"《Learning Kinematic Frequency-Aware Disentanglement for Motion Style Transfer and Editing》被计算机图形学顶级会议 SIGGRAPH ASIA 2026 条件接收"；亦见列表页 https://sai.jlu.edu.cn/kxyj/kycg.htm（2026-07-21） |

---

### 2. CustomDance: Customized 3D Dance Generation with a Coarse-to-Fine Human-Centered Interactive System

| 项目 | 内容 |
|---|---|
| 作者 | Xulong Tang, Kaixing Yang, Xiaohu Guo, Balakrishnan Prabhakaran, Rawan Alghofaili |
| 单位 | 主要为 The University of Texas at Dallas（HeXD Lab / Rawan Alghofaili、Xiaohu Guo、Balakrishnan Prabhakaran）；Kaixing Yang 单位未在项目页标注 → **部分未找到** |
| 轨道 | **Conference Papers**（BibTeX：`booktitle = {Proceedings of the SIGGRAPH Asia 2026 Conference Papers}`） |
| 项目页 | https://xulongt.github.io/customdance-project-page |
| arXiv | **未找到** |
| 核心问题 | 3D 舞蹈生成缺乏对多模态输入（音乐、具体动作描述）的全面且可区分的控制；生成动作统计上合理但缺乏深度、表现力与创作者意图对齐 |
| 方法与运动表征 | 受真实编舞工作流启发的**由粗到细三阶段交互系统**：① Choreographic Motif Planning（MLLM Planner 依据音乐与全局意图给出时间轴 anchors + 局部创意cues）；② Dance Phrase Generation（Retriever 从真实舞蹈短语库做**短语级检索**，支持局部文本与身体部位控制）；③ Completion and Refinement（Completer / Diagnoser / Remaker，做**diffusion-based** 过渡补全与局部修复）。→ **检索（离散短语单元）+ 连续 diffusion 补全的混合路线；无VQ-VAE / RVQ / FSQ tokenizer 描述** |
| 关键结果 | 项目页仅提供定性演示（Popping 高复杂度含 "Fresno" 基本动作；汉唐风格上肢舒展流线）。**无量化指标** |
| 开源情况 | **未找到**（项目页无code 链接） |
| **证据来源** | ① 项目页 https://xulongt.github.io/customdance-project-page 顶部标注 "SIGGRAPH Asia 2026" 且 BibTeX 明写 Conference Papers；② 实验室页 https://hexd-lab.github.io/ 标注 "Conditionally accepted to SIGGRAPH Asia 2026"；③ 新闻页 https://hexd-lab.github.io/news/ （Jul 20, 2026） |

---

### 3. UniMate: One Unified Model to Animate Diverse Skeletons

| 项目 | 内容 |
|---|---|
| 作者 | Linzhan Mou, Jiahui Lei, Zhiyang Dou, Chenyue Cai, Chaoyue Song, Adam Finkelstein, Szymon Rusinkiewicz |
| 单位 | Princeton University（Mou / Finkelstein / Rusinkiewicz，版权归Princeton）、UC Berkeley、MIT、NTU（GitHub 列出四家；**作者–单位一一对应关系未找到**） |
| 轨道 | 未细分（BibTeX 仅 `booktitle={SIGGRAPH Asia 2026}`）；一作主页标注 "(Conditionally Accepted)" |
| 项目页 | https://linzhanmou.com/unimate/ （镜像 https://linzhanm.github.io/unimate/ ）；交互 demo：interactive.html |
| 论文 PDF | https://linzhanmou.com/unimate/resources/unimate.pdf ；**arXiv 未找到** |
| 代码 | https://github.com/Friedrich-M/UniMate （MIT，37★，目前仅 assets+README） |
| 核心问题 | 自动 rigging 已能大规模产出"animation-ready"3D 资产，但驱动它们的运动生成仍是瓶颈。现有学习式 animator 受**拓扑约束**：依赖类别专属模板、或需逐骨架微调、或推理时需参考运动 |
| 方法与运动表征 | **拓扑感知扩散 Transformer（topology-aware diffusion transformer / DiT）**，三种机制把骨架拓扑注入 attention：① graph-aware attention bias（来自关节两两关系 + 测地距离）；② spectral rotary position embedding（借图拉普拉斯把 RoPE 推广到任意运动学树）；③ global topological conditioner（从 rest-pose 骨架 attention-pool 得到）。数据侧：BFS 序列化关节、按拓扑直径缩放、规范坐标系、**旋转相对 rest pose 表达**、特征统计归一化。→ **连续特征 + diffusion**，**明确无 VQ-VAE / RVQ / FSQ / codebook**；"Motion Expansion" 是多提示串联续接，非 token级自回归 |
| 数据集 | **UniML3D**：13,006 条文本配对运动序列，覆盖bipedal / quadrupedal / avian / marine / insectoid / serpentine / articulated rigid objects；来源融合 Truebones + Mixamo + Objaverse-XL；30 FPS 四路同步视图+多模态 LLM 生成 caption + 人工复核 |
| 关键结果 | 摘要称在**质量、泛化性、效率**三方面优于 SOTA baselines；支持 zero-shot 跨拓扑迁移、in-betweening、motion expansion、文本引导编辑；实时生成、无测试时优化、无需逐骨架重训。**未给出任何数值指标与 baseline 名称** |
| 开源情况 | 代码仓库已建（MIT），但训练代码与 UniML3D 数据集标注为 `[TODO]`，计划 2026 年 8 月左右发布；**权重未提及** |
| **证据来源** | ① 一作主页 http://linzhanm.github.io/ ——"SIGGRAPH Asia 2026 ... (Conditionally Accepted)" 并附完整摘要；② GitHub About "[SIGGRAPH Asia 2026] UniMate..." 及 News "[2026-07-18] UniMate is accepted to SIGGRAPH Asia 2026!"；③ 项目页顶部标注 + BibTeX |

---

### 4. ProAct（题名存在两个版本，见下）

| 项目 | 内容 |
|---|---|
| 标题（作者主页 / 项目页版） | **ProAct: Harnessing Streaming Motion Generation and Agentic Reasoning for Realtime Embodied Social Interaction** |
| 标题（arXiv v1 版） | **ProAct: A Dual-System Framework for Proactive Embodied Social Agents** ⚠️ 两者不一致，最终题名待正式出版确认 |
| 作者 | 主页版：Zeyi Zhang\*, Zixi Kang\*, Ruijie Zhao\*, Yusen Feng, Biao Jiang, Hanyu Ji, Libin Liu†（\*共同一作，†通讯）<br>arXiv v1版：Zeyi Zhang, Zixi Kang, Ruijie Zhao, Yusen Feng, Biao Jiang, Libin Liu（少Hanyu Ji） |
| 单位 | 北京大学智能学院（刘利斌Libin Liu 课题组，PKU-MoCCA）；**arXiv 摘要页未列单位，逐作者单位未找到** |
| 轨道 | **Journal Track**（主页原文："ACM SIGGRAPH Asia 2026 Journal Track. (To appear)"） |
| 项目页 | https://proactrobot.github.io/ |
| arXiv | https://arxiv.org/abs/2602.14048 （v1, 2026-02-15，cs.RO / cs.CV / cs.GR；**comment 字段只有项目页链接，未提 SIGGRAPH**） |
| 代码 | https://github.com/LuMen-ze/Social_Robot （标注 Code TBD / coming soon，**尚未发布**） |
| 核心问题 | 具身社交智能体多为被动反应式，仅响应短时窗内当前感知；主动社交行为需长时程上下文推理与意图推断，与实时交互的严格延迟预算冲突 |
| 方法与运动表征 | **双系统框架**：低延迟 **Behavioral System**（streaming omni-modal LLM 级联 streaming motion generator，言语走 turn-based、动作持续运行，二者异步）+较慢 **Cognitive System**（Context Encoder 压缩历史为bounded memory；Behavior Planner 预测用户动机并规划，输出高层主动意图）。动作侧核心：**Conditional Flow Matching（CFM）**，最优传输路径 + Transformer backbone，由高斯噪声经学习速度场生成动作；**overlap-and-cache** 机制消除窗口边界不连续以实现 streaming；**disentangled ControlNet**把文本控制与冻结的音频驱动基础生成器解耦，支持异步意图注入、反应式↔主动式手势无缝切换。→ **连续 flow matching（非 diffusion DDPM、非离散 token）；明确无 VQ-VAE / RVQ / FSQ / codebook**；自回归成分在语言侧而非动作侧 |
| 关键结果 | 已部署于**真实物理人形机器人**；提出新基准 **ProActBench**（评测主动触发检测与克制）；真实世界用户研究中参与者与旁观者在感知主动性、社会临场感、整体参与度上一致偏好 ProAct 优于反应式变体；消融显示 ControlNet 训练方案动作更多样连贯、DDIM baseline 推理延迟高；生成速度快于实时播放。**无FGD/FID 等具体数值（页面未给）**；机器人型号、训练数据规模未找到 |
|开源情况 | 代码 TBD、视频 TBD、ProActBench 是否发布未说明 |
| **证据来源** | 刘利斌主页 https://libliu.info/ （原 libliu.org 重定向）——论文条目标注 "ACM SIGGRAPH Asia 2026 Journal Track. (To appear)"，且 News 有 "2026/07/20: one paper accepted to ACM SIGGRAPH Asia 2026"（全站唯一一篇）；项目页 https://proactrobot.github.io/ 标注 SIGGRAPH Asia 2026 |

---

### 5. STEER: Steerable Dyadic Head Avatars

| 项目 | 内容 |
|---|---|
| 作者 | Kartik Teotia, Helge Rhodin, Hyeongwoo Kim, Marc Habermann, Christian Theobalt |
| 单位 | Max Planck Institute for Informatics（项目页仅列此一家；Rhodin 常挂 UBC、Kim 常挂 Imperial College → **精确对应未找到**） |
| 轨道 | **Conference**（项目页 venue 声明 "SIGGRAPH Asia 2026 Conference"） |
| 项目页 | https://kartik-teotia.github.io/STEER |
| 论文 PDF | 项目页托管 `assets/paper/STEER_arxiv.pdf`；**arXiv 编号/链接未找到** |
| 代码 | https://github.com/Kartik-Teotia/STEER （inference code + project page，6★，NOASSERTION 许可） |
| 核心问题 | 语音驱动人脸动画多把对话行为当作"音频的涌现副产品"，只暴露粗粒度序列级情感控制，导致注视接触/回避、节律性头部运动、情绪等关键非语言通道难以显式控制；且公开双人语料缺乏时间对齐的行为标注 |
| 方法与运动表征 | **可控 3D 双人（dyadic）动作先验**，三阶段：① 自建tracking + annotation pipeline，从 in-the-wild 双人视频提取行为伪标签（面部追踪、头部与眼部姿态、Action Units、粗粒度情绪、运动学分解）；② **causal flow-matching transformer**，条件为目标音频 + 伙伴动作 + 情绪 + 行为控制量，输出 partner-aware 目标动作；③ Avatar bridge：把追踪得到的参数化动作映射到 **Universal Gaussian Head-Avatar Prior** 驱动空间，底层高斯 avatar 模型**冻结、无需重训**。显式控制三轴：gaze / head rhythm / emotion。→ **连续 flow matching + causal（自回归式）Transformer；无 VQ /离散 token 描述** |
| 关键结果 | 在 motion quality、dynamics、diversity 上优于近期 dyadic motion baselines；partner coupling 上"保持竞争力"（非最优）；支持 gaze / head-rhythm / emotion 编辑与交互式实时部署（演示中 partner-aware 对话达 **90fps**）。数据统计：过滤后 **10,555** 片段（训练 9,790 / 保留测试 765）。**无 FID/LVE 等数值、未列 baseline 名称** |
| 开源情况 | 项目页提供 "Code/Data" 按钮指向 GitHub；仓库为 inference code + project page，许可 NOASSERTION；数据（标注/伪标签）承诺发布但无时间表 |
| **证据来源** | ① 项目页 https://kartik-teotia.github.io/STEER 标注 "SIGGRAPH Asia 2026 Conference" / "SA '26" / 页脚 "SIGGRAPH Asia 2026 · Kuala Lumpur"；② MPI GVDH 组出版页 https://gvdh.mpi-inf.mpg.de/publications.html 标注 "Siggraph Asia 2026" 并附摘要；③ GitHub About "STEER: Steerable Dyadic Head Avatars (SIGGRAPH Asia 2026)"；④ 一作 LinkedIn帖文"conditionally accepted to SIGGRAPH Asia 2026" |

---

### 6. UMA: Ultra-detailed Human Avatars via Multi-level Surface Alignment

| 项目 | 内容 |
|---|---|
| 作者 | Heming Zhu, Guoxing Sun, Christian Theobalt, Marc Habermann |
| 单位 | Max Planck Institute for Informatics, Saarland Informatics Campus（VCAI 部门）；Saarbrücken Research Center for Visual Computing, Interaction and AI (VIA) |
| 轨道 | **Journal Track（ACM TOG）** —— GVDH 页标注 "ToG (Siggraph Asia) 2026" |
| 项目页 | https://vcai.mpi-inf.mpg.de/projects/UMA/ |
| 论文 | ACM DL https://dl.acm.org/doi/10.1145/3829365 （DOI 10.1145/3829365）；**arXiv 未在项目页给出** |
| 代码 / 数据 / Demo | https://github.com/kv2000/UMA ；数据集 https://gvv-assets.mpi-inf.mpg.de/uma/ ；在线 Demo https://uma4.umaumau.xyz |
| 核心问题 | 可驱动着装人体 avatar 在 4K+ 分辨率、相机拉近时细节不足；根因是表面追踪不准（深度错位、表面漂移），迫使外观模型去补偿几何误差；服装动态具随机性，无法仅由骨骼姿态解释 |
| 方法与运动表征 | 输入**骨骼运动 + 相机视角** → 输出高保真几何与外观。可驱动模板 V_f 作几何骨架 + 注入**逐帧可学习连续隐编码 z_f**（建模骨骼无法解释的服装随机动态，测试时置零）；texel 超分模块对可动画 Gaussian textures 稠密化。核心贡献 **multi-level surface alignment**：借**基础 2D 视频点追踪器**在顶点级与 texel 级监督 3D 形变，并用级联训练策略把2D 点轨迹锚定到渲染 avatar 以生成一致 3D 点轨迹。→ **连续 latent（auto-decoder 风格）+ 姿态驱动的条件回归**；**非生成式 token 建模，无 VQ/diffusion/AR** |
| 关键结果 | 渲染质量与几何精度显著优于此前 SOTA（项目页未列数值，需查正文）；支持自由视点渲染、纱线级纹理细节的数字变焦、时间上共享三角化的精细褶皱、**动作重定向（motion retargeting）**、纹理编辑。新数据集：5 位受试者、**40 台 6K 相机**、light stage、每序列>10 分钟（约 7.6 万训练帧 + 4.1 万测试帧） |
| 开源情况 | 代码 + 数据集 + 在线 Demo 均已提供（数据集经 GVV-Assets 平台，通常需申请许可） |
| **证据来源** | ① MPI GVDH 出版页 https://gvdh.mpi-inf.mpg.de/publications.html——"ToG (Siggraph Asia) 2026"；② 作者 Heming Zhu 的 LinkedIn 公告"published in ACM Transactions on Graphics (TOG) and to be presented at SIGGRAPH Asia 2026"（经 https://be.linkedin.com/in/jentevandersanden 转发可见） |
| 归类说明 | 严格说是"给定动作后的外观/几何合成"（4D 动态角色 + 动作重定向应用），非动作生成 |

---

### 7. OMEGA-Avatar: One-shot Modeling of 360° Gaussian Avatars

| 项目 | 内容 |
|---|---|
| 作者 | Zehao Xia¹, Yiqun Wang¹\*（通讯）, Zhengda Lu², Kai Liu¹, Jun Xiao², Peter Wonka³ |
| 单位 | ¹重庆大学 ²中国科学院 ³KAUST |
| 轨道 | 未细分（BibTeX 仅 `booktitle={SIGGRAPH Asia 2026}`） |
| 项目页 | https://omega-avatar.github.io/OMEGA-Avatar/ |
| arXiv | https://arxiv.org/abs/2602.11693 （PDF https://arxiv.org/pdf/2602.11693.pdf ）；视频 https://youtu.be/Lhf6aVFdaTk |
| 核心问题 | 单图生成高保真可动画 3D avatar：现有工作只能同时满足"前馈/ 360° 全头 / animation-ready"三者中的两个 |
| 方法与运动表征 | 首个同时前馈、360° 完整、可动画的单图 3D 高斯头部框架。两个新模块：① **semantic-aware mesh deformation**（融合多视角法线优化带头发的 FLAME 头部，保持拓扑）；② **multi-view feature splatting**（可微双线性 splatting + 分层 UV 映射 + 可见性感知融合，构建共享 canonical UV 表示，实现 360° 一致性且无需逐实例优化）。动画驱动方式：**从目标图像提取的表情与姿态参数注入形变网格**（FLAME expression/pose 连续参数），最终经neural refiner 增强。→ **连续参数驱动 + 前馈网络；无离散 token / 动作 tokenizer** |
| 关键结果 | 在 360° 全头完整性上显著优于现有 baselines，并在不同视角下稳健保持身份；支持 Expression Reenactment 与 Animatable Full-Head Reconstruction。**项目页无量化数值** |
| 开源情况 | Code (coming soon)，**尚未发布** |
| **证据来源** | 项目页 https://omega-avatar.github.io/OMEGA-Avatar/ 标题下方独立行 "SIGGRAPH Asia 2026" + BibTeX `booktitle={SIGGRAPH Asia 2026}` |
| 归类说明 | 面部/头部动作驱动（可动画 avatar），非身体动作生成 |

---

### 8. HyperBones: Realtime Bone-driven Neural Garment Simulation with Hypernetwork Conditioning

| 项目 | 内容 |
|---|---|
| 作者 | **未找到**（实验室新闻仅给标题） |
| 单位 | Avinash Sharma 教授课题组（IIT Jodhpur CSE，兼 IIIT Hyderabad CVIT） |
| 轨道 | 未找到 |
| 项目页 / arXiv / 代码 | **均未找到** |
| 核心问题 | 由骨骼驱动的实时神经服装仿真（bone-driven neural garment simulation） |
| 方法与运动表征 | 仅知使用 **hypernetwork conditioning**；其余（是否 token/diffusion）**未找到** |
| 关键结果 | **未找到** |
| 开源情况 | **未找到** |
| **证据来源** | https://3dcomputervision.github.io/ ——"HyperBones Accepted at SIGGRAPH Asia 2026 / Our paper HyperBones: Realtime Bone-driven Neural Garment Simulation with Hypernetwork Conditioning has been accepted at SIGGRAPH Asia 2026."（July 2026） |
| 归类说明 | 骨骼驱动的次级运动（服装动力学），motion 相关但非人体动作生成本体 |

---

## 二、疑似 / 待确认

### 9. MoCapAnything V2: End-to-End Motion Capture for Arbitrary Skeletons

| 项目 | 内容 |
|---|---|
| 作者 | Kehong Gong, Zhengyu Wen, Dao Thien Phong, Mingxi Xu, Weixia He, Qi Wang, Ning Zhang, Zhengyu Li, Guanli Hou, Dongze Lian, Xiaoyu He, Mingyuan Zhang, Hanwang Zhang（13 人） |
| 单位 | **未找到**（arXiv 摘要页与 GitHub 均无单位；仅知项目页托管于 animotionlab.github.io，权重在 HF 账号 `kehong` 下） |
| 项目页 | https://animotionlab.github.io/MoCapAnythingV2/ （静态抓取只得标题，疑为 JS 渲染） |
| arXiv | https://arxiv.org/abs/2604.28130 （v1 2026-04-30 / v2 2026-05-14 / v3 2026-06-19，cs.CV） |
| 代码 | https://github.com/phongdaot/MocapAnything （MIT，317★，**自述为非官方复现**："Unofficial code release... a reimplementation based on the paper"） |
| 核心问题 | 单目视频对任意骨架做动捕。现有方案为"Video-to-Pose 网络 + 解析式 IK"分解流程：关节位置无法完全决定旋转（骨轴扭转自由度歧义），且不可微 IK 阻碍端到端优化 |
| 方法与运动表征 | 首个Video-to-Pose 与 Pose-to-Rotation **均可学习、联合优化**的端到端框架。关键洞察：pose→rotation 的歧义源于缺失坐标系信息，故引入目标资产的**参考 pose–rotation 对**，配合 rest pose 锚定映射并定义旋转坐标系，把旋转预测变为受良好约束的条件问题；直接从视频回归关节位置，**不依赖 mesh 中间表示**（mesh-free）；两阶段共享骨骼感知的 **GL-GMHA**（Global-Local Graph-guided Multi-Head Attention）模块。→ **回归式连续输出**（直接输出 BVH-ready 关节旋转，依赖 `roma` 库做连续旋转表示）；**摘要与仓库均无 VQ-VAE / RVQ / FSQ / codebook / diffusion / AR 字样** |
| 关键结果 | Truebones Zoo 与 Objaverse 上，旋转误差由约 **17°降至约 10°**，未见骨架上达 **6.54°**；推理比基于 mesh 的流程**快约 20 倍**。附加应用 "Dance Anything"（舞蹈视频 → SAM2 锁定舞者 → 目标动物复现舞蹈 + 原音频） |
| 开源情况 | 代码已开源（MIT，非官方）；预训练权重已发布 `kehong/MoCapAnythingV2-weights`；Demo 数据样例（~160MB）；HF Spaces 在线 Demo（ZeroGPU 免费）；**完整训练数据集 TODO 未发布** |
| **为何列为待确认** | SIGGRAPH Asia 2026 标记**仅出现在这个自述"非官方"的第三方仓库 About/标题**（"[SIGGRAPH ASIA 2026]: End-to-End Motion Capture for Arbitrary Skeletons"）。arXiv 2604.28130 的 comment 字段**只有项目页链接、未提任何 venue**；官方项目页静态抓取无venue 信息。→ 待官方项目页或 arXiv comment 更新确认 |
| 相关前作（非本条） | MoCapAnything（V1）已发表于 **CVPR 2026**（pp. 7089–7099）；SWiT-4D 为 arXiv:2512.10860 |

---

### 10. KAIST Visual Media Lab（Junyong Noh 组）：3 篇 SIGGRAPH Asia 论文，标题全部未知

| 项目 | 内容 |
|---|---|
| 证据 | Seokhyeon Hong 主页 https://seokhyeonhong.github.io/ News："**[Jul. 2026] 3 papers accepted to SIGGRAPH Asia. (1 first-authored, 2 co-authored)**" |
| 状态 | ⚠️ 新闻只说"SIGGRAPH Asia"，**未写年份**（按时间推断为 2026，但页面未明写） |
| 标题 / 作者 | **未找到** —— 该主页 Publications 区2026 年条目全部标注 SIGGRAPH 2026 或 CVPR 2026，无任何 SIGGRAPH Asia 2026 条目；已交叉核对同组 Kwan Yun（http://kwanyun.github.io/ ）、Youngseo Kim（https://youngseo0526.github.io/publications ）主页，均无 SIGGRAPH Asia 2026 条目 |
| 研究方向背景（供预判） | Hong 本人方向为角色动画的generation / editing / in-betweening / retargeting / rigging；该组 2024 年曾有 SIGGRAPH Asia 2024 TOG 论文 "Geometry-Aware Retargeting for Two Skinned Characters Interaction" →这3 篇很可能含 motion 相关，但**当前无任何标题证据，不做推测** |
| 建议后续 | 查该作者 CV（assets/cv.pdf）、Google Scholar（user=cmuRDa8AAAAJ）或等主页更新 |

---

## 三、检索到但确认**不属于** SIGGRAPH Asia 2026 的 motion 论文（排除记录，避免误收）

以下在检索中反复出现、易被误归入本次目标的motion 论文，均已核实归属其他会议：

- **ARDY: Autoregressive Diffusion with Hybrid Representation for Interactive Human Motion Generation**（Kaifeng Zhao, Mathis Petrovich, Haotian Zhang, Tingwu Wang, Siyu Tang, Davis Rempe）→ **SIGGRAPH 2026**（Paper Digest SIGGRAPH 2026 列表 + 官方仓库自述）
- **GPC: Large-Scale Generative Pretraining for Transferable Motor Control**（Yi Shi, Yifeng Jiang, Chen Tessler, Xue Bin Peng；FSQ tokenizer + GPT 式自回归，600+ 小时数据，99.98% 复现率，arXiv:2606.29148）→ **SIGGRAPH 2026**
- **SMP: Reusable Score-Matching Motion Priors for Physics-Based Character Control**（Yuxuan Mu等）→ **SIGGRAPH 2026**
- **Stylized Text-to-Motion Generation via Hypernetwork-Driven Low-Rank Adaptation**（Junhyuk Jeon\*, Seokhyeon Hong\*, Junyong Noh；arXiv:2605.13333）→ **SIGGRAPH 2026**
- **Skinned Motion Retargeting with Spatially Adaptive Interaction Guidance**（Soojin Choi 等；arXiv:2605.19355）→ **SIGGRAPH 2026 / ACM TOG**
- **MOCHI**、**AMOR: Airborne Motion Reconstruction...**（SNU Intelligent Motion Lab / Jungdam Won 组）→ **SIGGRAPH 2026**
- **DMP: Directable Motion Retargeting through Motion Paraphrasing** → ACM TOG (to appear)，未标 SIGGRAPH Asia
- **ReActor: Reinforcement Learning for Physics-Aware Motion Retargeting**、**MotionBricks: Scalable Real-Time Motions with Modular Latent Generative Model and Smart Primitives** → **SIGGRAPH 2026**（Awesome-Human-Motion 列表标注）
- **Uni-Inter: Unifying 3D Human Motion Synthesis Across Diverse Interaction Contexts** → **SIGGRAPH Asia 2025**（非 2026）
- **CFC: Simulating Character-Fluid Coupling...**（HKU Komura 组）→ **SIGGRAPH Asia 2025**
- **WHIP: Towards Real-World Wearable Motion Reconstruction**（MPI）→ **ECCV 2026**
- **EmbodMocap: In-the-Wild 4D Human-Scene Reconstruction for Embodied Agents**（HKU）→ **CVPR 2026**
- **OpenDance: Multimodal Controllable 3D Dance Generation...**（PKU Libin Liu 组）→ **CVPR 2026 Oral**
- **Bridging Semantic and Kinematic Conditions with Diffusion-based Discrete Motion Tokenizer（MoTok）**（NTU S-Lab / Ziwei Liu 组）→ 仅arXiv 预印本，无 venue

同时排除的 **SIGGRAPH Asia 2026 非motion 论文**（供确认检索覆盖面）：InfiniSplat（ZJU3DV，Journal Track）、EgoPlay（Snap/KAUST，Conference）、ID-V2V（Netflix/Eyeline Labs）、AnyRecon、PanoWorld、DiffusionOPD、Lift4D、UVFaceFusion（Journal Track）、S4R、Mind-Brush、X-Splat、Many-Videos Browsing、Structure–Color Disentangled Anime Hair Synthesis via Flow Line（后三者见 https://jdily.github.io/ ，均标 conditionally accepted, Conference Track）、UltraTex、PIE-PS（吉林大学）、InstructAV2AV（智源+北大）。

---

## 四、覆盖度自评（按你给出的子领域）

| 子领域 | 找到的条目 |
|---|---|
| human motion generation（文本/音乐/语音驱动） | #2 CustomDance（音乐+文本）、#3 UniMate（文本）、#4 ProAct（语音/音频驱动手势） |
| motion tokenizer / 离散运动表征 | **0篇**。所有已确认条目均为连续 latent + diffusion / flow matching / 回归；无一篇使用 VQ-VAE / RVQ / FSQ |
| physics-based character control | **0 篇**（本届相关工作 GPC、SMP、ReActor 均在 SIGGRAPH 2026） |
| motion capture / estimation | #9 MoCapAnything V2（待确认） |
| motion editing / style transfer / retargeting | #1（style transfer + editing）、#3 UniMate（editing / in-betweening）、#6 UMA（motion retargeting 应用） |
| 手部与面部动作 | #5 STEER（头部/注视/情绪）、#7 OMEGA-Avatar（表情驱动）。**手部：0 篇** |
| gesture / speech-driven animation | #4 ProAct、#5 STEER |
| 四足与人形机器人运动 | #3 UniMate（四足等非人形骨架，图形学侧）、#4 ProAct（真实人形机器人部署）。**纯机器人 locomotion：0 篇** |
| 群体动画 / crowd | **0 篇** |
| 4D / 动态角色 | #6 UMA、#7 OMEGA-Avatar、#8 HyperBones（服装次级运动） |

---

## 五、已尝试的检索路径（含负面结果）

**A. arXiv 站内 / API 检索（多种字段组合，均已跑通）**
1. `all:"SIGGRAPH Asia 2026"` →仅 **3 条**：InfiniSplat、EgoPlay、ID-V2V，**均非 motion**
2. `co:"SIGGRAPH Asia 2026"`（comment 字段） → 同上 3 条
3. `abs:"SIGGRAPH Asia 2026" OR co:"SIGGRAPH Asia 2026"` → 同上 3 条（totalResults=3）
4. `co:"SIGGRAPH Asia" AND submittedDate:[202605010000 TO 202612310000]` → 5 条，多出 StructuredEdit（仅"计划投稿"）与一篇 SIGGRAPH Asia **2025** 论文
5. 逐篇核验：arXiv:2604.28130（MoCapAnything V2）、arXiv:2602.14048（ProAct）→ 两者 comment **均未提 SIGGRAPH Asia**

**负面结论**：SIGGRAPH Asia 2026 作者极少在 arXiv comment 中标注该venue（可能与 7月才条件接收、且匿名政策要求评审期内不得标注有关），故 **arXiv API 对本任务基本无效**。

**B. GitHub 检索（Repository Search API）**
- `q="SIGGRAPH Asia 2026"` → **14 个**仓库
- `q="SIGGRAPH Asia 2026" in:description&sort=stars` → 12 个
- `q="SIGGRAPH Asia 2026" motion` → 1 个（MocapAnything）
- `q="SIGGRAPH Asia 2026" animation OR character OR dance OR gesture OR human` → 14 个
- 其中 motion/角色相关：MocapAnything(317★)、UniMate(37★)、STEER(6★)、UVFaceFusion(17★)、ID-V2V(131★)、Lift4D(379★)、S4R(1★)
- **未发现**任何 dance / gesture / crowd / physics-based control 专项仓库
- 说明：未能使用 Code Search API（需text-match 媒体类型），可能遗漏仅在 README 内文提及的项目

**C. 网页搜索（约 15 轮、60+ 关键词组合）**
- "SIGGRAPH Asia 2026" × {motion, human motion generation, character animation, motion tokenizer, VQ-VAE, physics-based character control, dance generation, music-driven, gesture, speech-driven, humanoid robot locomotion, hand motion, facial animation, retargeting, rigging, in-betweening, keyframe, human object interaction, crowd simulation, multi-agent, quadruped, animal, motion matching, neural animation, motion prior, foundation model, two-person interaction, terrain locomotion, IMU sparse sensors, 4D dynamic avatar, cloth secondary motion, motion capture markerless, pose estimation, mesh recovery, latent autoregressive, streaming}
- "conditionally accepted (to/by) SIGGRAPH Asia 2026" 多种变体
- 中文：SIGGRAPH Asia 2026 × {接收, 录用, 条件接收, 动作生成, 人体动作生成, 数字人,仿真角色, 强化学习, 手势, 语音驱动, 论文, 高校新闻稿, 公众号, 知乎, 项目主页}
- 日文「モーション 採択」、韩文「채택 모션논문」
- 平台限定：site:github.io、huggingface papers、alphaxiv、paperswithcode、awesome list

**D. 定向抓取的实验室 / 个人主页（逐页核验）**

有收获：
- 吉林大学人工智能学院科研成果列表 https://sai.jlu.edu.cn/kxyj/kycg.htm → 3 篇 SA2026（其中 1 篇 motion）
- HeXD Lab https://hexd-lab.github.io/ + /news/ → CustomDance
- Linzhan Mou http://linzhanm.github.io/ → UniMate（含完整摘要）
- Libin Liu https://libliu.info/ → ProAct（Journal Track）
- MPI GVDH https://gvdh.mpi-inf.mpg.de/publications.html → STEER + UMA（含摘要）
- I-Chao Shen https://jdily.github.io/ → 3 篇 SA2026（均非 motion）
- 3D Computer Vision Group (IIT Jodhpur) https://3dcomputervision.github.io/ → HyperBones

无收获（确认无 SIGGRAPH Asia 2026 motion 条目）：
- HKU CG (Taku Komura) https://hku-cg.github.io/ + /publication/ → 2026 年 9 篇，**0 篇 SIGGRAPH Asia 2026**（SIGGRAPH 2026×4、CVPR 2026×3、ICLR 2026×1、preprint×1）
- SNU Intelligent Motion Lab (Jungdam Won) https://sites.google.com/view/snuimo/publications → 2026 年 4 篇，**0 篇 SIGGRAPH Asia 2026**
- Seokhyeon Hong https://seokhyeonhong.github.io/ → News 有"3 papers accepted to SIGGRAPH Asia"但 Publications 区**无对应条目**（已核对原始 HTML，确认非抓取遗漏）
- Kwan Yun http://kwanyun.github.io/ 、Youngseo Kim https://youngseo0526.github.io/publications、Jiepeng Wang http://jiepengwang.github.io/ → 均无 SA2026
- Flawless AI Research https://flawlessai.com/research → 最新 SIGGRAPH Asia 条目为 2025 年UniGAHA
- Awesome Human Motion 汇总表 https://foruck.github.io/Awesome-Human-Motion → 全页 **81 处 SIGGRAPH 提及中，"Asia 2026" 出现 0 次**（该社区列表尚未收录任何 SA2026）
- KAIST Visual Media Lab 官网 https://visualmedialab.kaist.ac.kr/publications → 抓取失败（fetch failed）
- https://suzyn.github.io/ 、https://xiyang-cs.github.io/ → GitHub Pages 404

**E. 未能执行 / 受限的路径（可能造成遗漏）**
- **官方 paper ID 列表未做逐 ID 反查**：官方 PDF 只有 papers_XXXX 形式 ID，无标题，无法通过 ID 反推
- **Twitter/X 无法直接检索**：仅通过 LinkedIn 转发间接获得 STEER 与 UMA 的作者公告
- **GitHub Code Search API 未使用**（需特殊 Accept 头），可能遗漏只在 README 正文提及 SA2026 的仓库
- **本机curl 被环境阻断**，arXiv API 全部改由 WebFetch 代理，可能有截断风险（但 totalResults=3 已交叉验证多次）
- 微信公众号内容基本无法被搜索引擎索引，中文高校新闻稿只命中吉林大学一所

---

## 六、负面结果小结（如实报告）

1. **总量偏少**：截至 2026-08-10，公开可查、明确标注 SIGGRAPH Asia 2026 的 motion 相关技术论文共 **8 篇已确认 + 2 项待确认**，其中"人体动作生成"本体（非 avatar 外观/服装）严格来说只有 **#1 风格迁移、#2 舞蹈生成、#3 骨架动画生成、#4/#5 手势与头部动作** 共 5 篇。
2. **根本原因**：SIGGRAPH Asia 2026 的条件接收结果在 **2026 年 7 月 18–22 日** 前后才集中公布（多个独立来源日期一致：7/18 UniMate、7/20 CustomDance & ProAct & PanoWorld、7/21 吉大motion style、7/22 吉大另两篇），距今仅约 3 周；且官方**匿名政策**明确禁止在评审期内（2026-04-28 起至条件接收/拒稿通知前）以任何形式公开标注"under review at SIGGRAPH Asia"，导致作者集体延后公开。预期未来 1–3 个月会有大量新条目出现。
3. **离散运动表征（motion tokenizer）在本届SIGGRAPH Asia 目前为空白**：已确认的 8 篇里没有任何一篇使用 VQ-VAE / RVQ / FSQ / codebook；技术路线全部集中在 **diffusion（含 latent diffusion / DiT）与 flow matching（含 causal / streaming CFM）**，以及少数回归式方法。值得注意的是，本年度使用 FSQ tokenizer + 自回归的代表作 **GPC** 落在 SIGGRAPH 2026 而非 SIGGRAPH Asia 2026。
4. **完全未找到**的子领域：群体动画 / crowd simulation、手部动作与灵巧操作、纯 physics-based character control（RL 控制器）、四足/人形机器人 locomotion（图形学侧）。
5. **量化指标普遍缺失**：多数项目页处于"条件接收后刚上线"状态，只给定性结果；仅 MoCapAnything V2（旋转误差 17°→10°/6.54°、20× 加速）、STEER（10,555 片段、90fps）、UniMate（13,006 序列）、UMA（40×6K 相机、5 subjects）有具体数字。
6. **建议的后续跟踪节奏**：① 每 1–2 周重跑 arXiv `co:"SIGGRAPH Asia 2026"` 查询（新论文会陆续挂上）；② 盯 GitHub 新建仓库描述含 "[SIGGRAPH Asia 2026]"；③ 待KAIST Visual Media Lab 3 篇标题公布；④ 待 MoCapAnything V2 官方项目页/arXiv comment 更新以确认 venue；⑤ ACM DL 的 SIGGRAPH Asia 2026 Conference Proceedings / TOG 目次通常在会前约两周（2026 年 11 月中下旬）上线，届时可获得完整权威列表。
