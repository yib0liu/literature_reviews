# SIGGRAPH 2025 物理模拟论文逐篇总结

> **会议信息**：SIGGRAPH 2025（第 52 届），2025 年 8 月 10–14 日，加拿大温哥华。
>
> **本报告范围**：收录 **48 篇** SIGGRAPH 2025 物理模拟主线论文（流体/气体/固体/弹性体/杆/布料/接触/求解器/自适应/神经场），并附「物理 × AI / World Model」交叉方向分析。
>
> **数据来源**：physicsbasedanimation.com 官方合集帖（权威清单，2025-05-19 发布，共 49 条含1 条重复）、各论文项目主页、arXiv、ACM Digital Library、history.siggraph.org 官方摘要归档、SIGGRAPH 官方新闻稿。
>
> **可靠度约定**：正文标注 `【摘要原文】`（作者主页/arXiv/ACM DL/官方摘要归档原文）、`【项目页描述】`、`【二手转述】`、`【推断】`。**本报告不含编造内容**，信息缺失处均已明示。

---

## 目录

- [一、总体图景：2025 年是「数值基础设施集中收割」的一年](#一总体图景2025-年是数值基础设施集中收割的一年)
- [二、Flow Maps 家族（6 篇）](#二flow-maps-家族6-篇)
- [三、流体：大规模、自适应与新表示（5 篇）](#三流体大规模自适应与新表示5-篇)
- [四、流体：SPH / 相变 / 动理学 / 雪崩（4 篇）](#四流体sph--相变--动理学--雪崩4-篇)
- [五、固体与弹性体（6 篇）](#五固体与弹性体6-篇)
- [六、杆与绳结（3 篇）](#六杆与绳结3-篇)
- [七、布料与针织（3 篇）](#七布料与针织3-篇)
- [八、刚体动力学（3 篇）](#八刚体动力学3-篇)
- [九、接触与碰撞检测（3 篇）](#九接触与碰撞检测3-篇)
- [十、GPU 与数值基础设施（8 篇）](#十gpu-与数值基础设施8-篇)
- [十一、自适应与 LOD（2 篇）](#十一自适应与-lod2-篇)
- [十二、神经场与 AI 交叉（2 篇）](#十二神经场与-ai-交叉2-篇)
- [十三、World Model × 物理模拟：本届的准确定位](#十三world-model--物理模拟本届的准确定位)
- [十四、奖项、量化一览与开源清单](#十四奖项量化一览与开源清单)
- [附录：信息缺口声明](#附录信息缺口声明)

---

## 一、总体图景：2025 年是「数值基础设施集中收割」的一年

通读 48 篇后，最强烈的印象是：**本届的核心贡献大多不在新本构模型，而在求解器结构的重组**。至少 12 篇论文的主贡献是「让原有物理模型算得更快/更稳」，而非「模拟新现象」。五条主线：

**1. 七条互不重叠的加速路线同时取得 10×–200× 量级收益。**
预条件子（StiffGIPC 的连接性增强 MAS、Lightning-Fast BEM 的稀疏inverse-LU）、多重网格（MGPBD 在对偶空间做 AMG）、符号分析复用（Parth 的分层图分解）、二阶并行迭代（JGS2 的抗过冲修正）、异构调度（HEFT +异步 Gauss-Seidel）、随机化快速求和（Stochastic Barnes-Hut 的控制变量）、DSL/编译器（MiSo 的区间算术）。这些路线各自独立，说明「求解器还有大量未被榨取的结构性余量」是本届的集体共识。

**2. Flow Maps 从单一技术长成一个家族。**
六篇 Flow Maps 论文全部出自 Bo Zhu 组（Georgia Tech），分别攻**演化量的选择**（VPFM 改用涡量、Clebsch 改用波函数）、**自适应**（Cirrus）、**实时**（LFM）、**可压缩推广**（Compressible FM）、**内存**（EDGE 把 O(N) 降到 O(1)）。这种「一个组在一届会议上把一个方向的六个正交维度同时推进」的现象在图形学中相当罕见。

**3. 接触模型的理论收敛。**
Offset Geometric Contact 与 Geometric Contact Potential 从两个方向修正 IPC——前者改几何构造（法向偏置取代 capsule 状全向偏置）以消除伪力并**彻底弃用 CCD**，后者从连续曲面公理化推导出与离散化无关的势并**统一了既有全部障碍方法**。C⁵D 则把 CCD 从图元级提升到凸体级。三者共同指向「降低无穿透保证的成本」。

**4. CPU 的反攻。**
Domain-decomposed Projective Dynamics 明确挑战「高性能 = GPU」这一默认假设，理由务实——生产环境中 GPU 常已被渲染与着色占满。该文获**最佳论文荣誉提名**，在许多算例上不使用 GPU 却达到 SOTA GPU 算法的性能。

**5. 微分工具从一阶走向二阶与流形正确。**
Elastic Locomotion 把复步长有限差分「提升」到反向自动微分之上从而拿到二阶导、可用牛顿法；Painless Differentiable Rotation Dynamics 用 Lie 代数导数绕开所有旋转参数化的约束麻烦；Neurally Integrated FE 用神经网络拟合求积点使隐式域可微。

值得注意的是，**AI 在本届物理模拟中的位置相当克制**：论文层面只有「神经场作降阶基」（Wind Lifter，最佳论文荣誉提名）与「生成先验 + 可微物理做逆问题」（Dress-1-to-3）两条路径，**没有一篇以「用神经网络替代整个数值求解器」为核心**。World model 完全停留在 keynote 与厂商发布层面，详见第十三节。

---

## 二、Flow Maps 家族（6 篇）

六篇全部来自 Bo Zhu 组（Georgia Tech）及其合作者。Flow Maps 的基本思想是：不逐步推进速度场，而是维护从初始时刻到当前时刻的**流图（flow map）**，从而把长程输运误差大幅降低。本届的六篇分别改进了这个框架的六个不同维度。

### 2.1 Fluid Simulation on Vortex Particle Flow Maps (VPFM) 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Sinan Wang, Junwei Zhou, Fan Feng, Zhiqi Li, Yuchen Sun, Duowen Chen, Greg Turk, Bo Zhu（Georgia Tech 为主） |
| **出处** | ACM TOG 44(4), 24 pp.；DOI 10.1145/3731198；arXiv:2505.21946 |
| **链接** | https://vpfm.sinanw.com/ ；代码 github.com/pfm-gatech/VPFM |

**核心问题**：在**动态固体边界**存在的情况下，模拟复杂涡演化的不可压缩流。

**方法要点**：核心洞见是**涡量比速度或impulse 更适合在粒子流图上演化**，因而能支撑更长的流图距离。采用混合欧拉—拉格朗日表示：涡量与流图量在**涡粒子**上演化，速度在背景网格上重建。三大组件：基于涡量的粒子流图框架；粒子上的**高精度 Hessian 演化格式**；适配 VPFM 的固壁处理（同时满足 no-through 与 no-slip）。可视为用现代流图视角复兴经典的**VIC（Vortex-in-Cell）**方法。

**效果**：流图长度较 SOTA **长 3–12 倍**。

**局限**：摘要未述。依赖背景网格做速度重建（需解 Poisson），因此并非纯无网格方法。

---

### 2.2 Clebsch Gauge Fluid on Particle Flow Maps 🏅 Best Paper Honorable Mention 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Zhiqi Li, Candong Lin, Duowen Chen, Xinyi Zhou（Georgia Tech）, Shiying Xiong（浙江大学）, Bo Zhu |
| **出处** | ACM TOG 44(4):1–12 |
| **链接** | https://pearseven.github.io/PFMClebschProject/ |

**核心问题**：高阶导数在粒子上演化困难；如何在粗网格上保住小尺度涡结构。

**方法要点**：在粒子流图上演化 **Clebsch 波函数**。关键洞见是：粒子流图传输**点元素**（Clebsch 分量）的能力，优于以往 impulse gauge 或 vortex gauge 所依赖的**线元素与面元素**。三项贡献：新的**规范变换**使波函数可在粒子流图上精确传输；面向**粗网格的增强速度重建**；完整的粒子流图仿真框架。

**效果**：在蛙跳涡环、涡重联、Kelvin–Helmholtz 不稳定性上验证；teaser 显示在**分辨率低至 64** 的极粗网格下仍能保持涡结构。

**局限**：项目页未给运行时间或加速比；小尺度优势主要是相对同族的 impulse / vortex 流图方法而言。

---

### 2.3 Cirrus: Adaptive Hybrid Particle-Grid Flow Maps on GPU 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Mengdi Wang, Junlin Li, Bo Zhu（Georgia Tech）, Fan Feng（Dartmouth） |
| **链接** | https://wang-mengdi.github.io/proj/25-cirrus/ ；代码 github.com/wang-mengdi/Cirrus |

**核心问题**：流图方法此前无法真正做到自适应——因为对流与投影两步都需要自适应支持。

**方法要点**：让粒子承担**双重角色**——既传输 impulse，又作为**引导网格细化的 oracle**。网格节点与粒子**分别维护流图轨迹**：网格在粗分辨率上追踪长程流图，粒子在细分辨率上追踪长程加短程流图并辅以**流图梯度**。实现于类**八叉树自适应网格**（基本单元为 8×8×8 tile）的 GPU 框架。

**效果**：单张 **RTX 4090** 上达到有效分辨率 **512×512×2048**；同硬件下相对 PFM 有 **1.5–2× GPU 加速**；自适应网格相对均匀网格效率提升 **1–2 个数量级**。案例：赛车、四螺旋桨 WP-3D飞机（15° 攻角）、火烈鸟群、蝙蝠。

**局限**：粒子数未公布；自适应带来的守恒误差与过渡误差未在项目页量化。

---

### 2.4 Leapfrog Flow Maps for Real-Time Fluid Simulation (LFM) 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Yuchen Sun, Junlin Li, Ruicheng Wang, Sinan Wang, Zhiqi Li, Bo Zhu（Georgia Tech）, Bart G. van Bloemen Waanders（Sandia National Labs） |
| **链接** | https://yuchen-sun-cg.github.io/projects/lfm/ ；代码 github.com/yuchen-sun-cg/lfm |

**核心问题**：基于 impulse 的流图计算量太大，无法实时。

**方法要点**：**混合 velocity–impulse 方案 + leapfrog（蛙跳）格式**降低计算负担，同时保住涡结构；开发**无矩阵（matrix-free）AMGPCG** 求解器配合定制 GPU 优化，加速 impulse→velocity 投影，使**投影耗时与 impulse 演化相当**。

**效果**：案例含燃烧火球、三角翼翼尖涡、风机螺旋尾迹、双涡环连接。

**局限**：项目页与代码 README **均未给出 FPS 或分辨率**，仅有"real-time" 的定性表述；README 仅说明编译测试环境为 RTX 4090 / CUDA 12.6。蛙跳格式的稳定性与 CFL 约束未见说明。

---

### 2.5 Fluid Simulation on Compressible Flow Maps 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Duowen Chen\*, Zhiqi Li\*（共同一作）, Taiyuan Zhang, Jinjin He, Junwei Zhou, Bo Zhu, Bart G. van Bloemen Waanders |
| **链接** | https://cdwj.github.io/projects/compressible-flowmap-project-page/ ；代码 github.com/CDWJ/compressible-fm |

**核心问题**：流图方法此前局限于不可压缩流。

**方法要点**：构建统一的可压缩流图框架。三项贡献：基于**拉格朗日路径积分**的可压缩流图理论基础；**密度与能量守恒输运**的新对流格式；支持不同压力处理的统一数值框架。覆盖三类系统——高马赫数（激波、超音速）、弱可压缩（烟羽、墨扩散）、以**可压缩声学量**演化的不可压缩系统（自由表面浅水）。

**效果**：19 个算例（激波 6、浅水 10、弱可压 3），含 Mach Diamond、活塞蛙跳、卡门涡街双向耦合、墨环破裂、三角翼尾涡。

**局限**：无数值性能数据。三类系统靠不同的压力处理拼接，通用性以配置复杂度为代价。

---

### 2.6 EDGE: Epsilon-Difference Gradient Evolution for Buffer-Free Flow Maps 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Zhiqi Li\*, Ruicheng Wang\*, Junlin Li\*（共同一作）, Duowen Chen, Sinan Wang, Bo Zhu（全部 Georgia Tech） |
| **出处** | ACM TOG 44(4):1–11 |
| **链接** | https://pearseven.github.io/EDGEProject/ |

**核心问题**：欧拉流图（EFM）需缓存整段速度历史（velocity buffer），内存随流图长度线性增长。

**方法要点**：在网格上用 **Hermite 插值**精确计算流图而**完全不用速度缓冲**。结合 **Gradient Evolution**（精确一阶导）与基于**四面体的 Epsilon Difference** 格式（以低内存计算高阶导）。内存复杂度 **O(1)**，与流图长度无关。

**效果**：后向流图内存最多降 **90%**；dye drift 实验总内存从 EFM 的 **37.89 GB** 降到 EDGE 的 **10.79 GB**，四点差分变体 ED4 进一步到 **8.54 GB**；涡量守恒与 buffer-based 方法相当。

**局限**：epsilon 差分引入有限差分步长敏感性（页面未量化）；仅针对网格流图。

---

## 三、流体：大规模、自适应与新表示（5 篇）

### 3.1 Adaptive Phase-Field-FLIP for Very Large Scale Two-Phase Fluid Simulation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Bernhard Braun（TUM）, Jan Bender（RWTH Aachen）, Nils Thuerey（TUM） |
| **链接** | https://ge.in.tum.de/publications/very-large-scale-two-phase-flip/ ；代码 github.com/tum-pbs/MSBG |

**核心问题**：影视特效至今仍在用单相求解器加上非物理的 "white-water" 启发式。而大尺度水体需要真实模拟**水—气两相**——气动力正比于相对速度平方，高 Weber 数下表面张力无法稳定液体，液体会破碎成喷雾。

**方法要点**：**PF-FLIP** = Phase-Field 与 FLIP 的混合欧拉/拉格朗日方法，面向高雷诺数、高密度比的强湍流多相流。以**一致且非耗散**的方式输运质量与动量；**无需表面重建**步骤；**所有关键组件（含压力 Poisson 求解器）均空间自适应**；采用**双重多分辨率**方案耦合**无树（treeless）自适应网格与自适应粒子**；针对高密度比定制**快速自适应 Poisson 求解器**。

**效果**：**数十亿粒子**；每个维度**数千**网格单元；碎浪案例有效分辨率 **4096×1024×512**；大坝泄流水速约 **40 m/s**；全部在**单台工作站**上完成。

**局限**：页面未给加速比、内存占用与单帧耗时。

> **系列关联**：本文是 SIGGRAPH 2026 Honorable Mention 论文 ST-FLIP（Spatiotemporal FLIP）的直接前作，同一团队连续两年推进大规模两相液体模拟。

---

### 3.2 Quadtree Tall Cells for Eulerian Liquid Simulation 【项目页描述】

| 项目 | 内容 |
|---|---|
| **作者** | Fumiya Narita, Nimiko Ochiai, Takashi Kanai, Ryoichi Ando（东京大学 Kanai Lab） |
| **出处** | SIGGRAPH Conference Papers, Art. 10, 11 pp.；DOI 10.1145/3721238.3730652 |
| **链接** | https://graphics.c.u-tokyo.ac.jp/hp/en/archives/3252 |

**核心问题**：传统 tall cell 法为捕捉液面细节，在水平方向不做自适应，导致单元数偏多。

**方法要点**：对 tall cells 在**水平方向做四叉树细分**（注意是四叉树而非八叉树），2D 单元再沿垂直方向拉伸成 tall cell，构建过程远比四面体或八叉树简单。提出 tall-cell 网格上 Poisson 方程的**新变分表述**，自然处理**变尺寸单元之间的过渡**。变分视角带来两个额外好处：可直接使用成熟的**共轭梯度法**；支持**整体式（monolithic）双向刚体耦合**。

**效果**：项目页称单元数显著减少、性能优于均匀 tall cells，但**未给出任何具体数字**。

**作者勘误**（2025-07 Hindsight，经 Christopher Batty 反馈）：压力采样点的水平位置为俯视（xz面）单元中心；矩阵 [V]/[grad]/[A] 还编码了 tall cell 的**垂直内部 ghost 面积体积**，求解时由背景垂直单元面速度平均临时插入 **ghost velocity**，仅用于压力求解，求解后即丢弃。

**局限**：仅适合「深水 + 液面细节」型场景，本质依赖垂直方向的拉伸假设。

---

### 3.3 Gaussian Fluids: A Grid-Free Fluid Solver based on Gaussian Spatial Representation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jingrui Xing, Mengyu Chu†, Baoquan Chen†（北京大学）, Bin Wang（独立研究者） |
| **出处** | SIGGRAPH Conference Papers, Art. 31, 11 pp.；DOI 10.1145/3721238.3730620 |
| **链接** | https://xjr01.github.io/GaussianFluids/ |

**核心问题**：欧拉与拉格朗日离散化都会带来空间离散误差；而隐式神经表示则不够鲁棒与通用。

**方法要点**：受 **3D Gaussian Splatting** 启发，将连续速度场建模为**多个高斯函数的加权和**（Gaussian Spatial Representation, GSR）。该表示**连续可微**，可直接解析求空间微分，并用为流体动力学定制的**一阶优化**方法求解时变 PDE。天然内存高效、空间自适应。

**效果**：验证覆盖 2D/3D 多种现象、双涡环碰撞、卡门涡街、长时间稳定性，且**无需额外调参**。

**局限（作者自述）**：**一阶求解器的速度仍不及显式表示的求解器**。项目页无量化数据。

---

### 3.4 Fast Subspace Fluid Simulation with a Temporally-Aware Basis 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Siyuan Chen（多伦多大学 / 上海交通大学）, Yixin Chen, Jonathan Panuelos, Otman Benchekroun, Yue Chang, Eitan Grinspun, Zhecheng Wang（多伦多大学） |
| **链接** | https://www.dgp.toronto.edu/projects/dmd/ |

**核心问题**：空间降阶模型（ROM）压缩性好但缺乏物理直观；谱方法则相反。

**方法要点**：用**动态模态分解（DMD）**——优化的对象是**把状态从一步演化到下一步的算子**，而非状态本身，从而兼得空间 ROM 的压缩力与谱方法的直观动力学。工程适配包括：降低计算开销、引入用户自定义力输入、用 **randomized SVD** 优化内存；组合 **OptDMD 与 DMDc（DMD with Control）**实现抗噪重建与实时交互。利用 DMD 公式的**线性**特性，支持时空调制、加湍流的上采样、艺术编辑、**时间反演**与超分辨率。

**效果**：称基函数数量显著少于现有空间 ROM。验证场景为碰撞涡环、与边界交互的烟柱。

**局限**：降阶方法固有的泛化受限（依赖训练/快照数据）。页面无量化数据。

> **值得注意**：DMD 这条线在 SIGGRAPH 2026 有直接后续——Low-Rank Koopman Deformables 把同类思想用于可变形体并做到log-linear 时间缩放。

---

### 3.5 A Neural Particle Level Set Method for Dynamic Interface Tracking 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Duowen Chen, Junwei Zhou, Bo Zhu（Georgia Tech，据主页域名【推断】） |
| **链接** | https://cdwj.github.io/projects/neural-pls-project-page/ |

**核心问题**：追踪与演化**动态神经表示**的界面。

**方法要点**：一组**有向粒子（oriented particles）**兼任**界面追踪器**与**采样播种器**；用这些动态粒子演化界面，并在**多分辨率 grid-hash 结构**上构建神经表示，混合**粗稀疏距离场**与**多尺度特征编码**。结合传统 Particle Level Set 与现代隐式神经表示两端的优势。

**效果**：验证含Spike Disk、单涡、涡粒子、Armadillo 刚体旋转、3D 形变（含长时版）；流体场景含随机球落水、Armadillo 落水、运动桨叶、灯塔。

**局限**：项目页**无任何数值**（无体积守恒误差、无耗时数据）。

> **注意**：项目页顶部残留 "Solid-Fluid Interaction on Particle Flow Maps" 字样，属模板残留，非本文标题。

---

## 四、流体：SPH / 相变 / 动理学 / 雪崩（4 篇）

### 4.1 Unified Pressure, Surface Tension and Friction for SPH Fluids 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Timo Probst, Matthias Teschner（University of Freiburg, 德国） |
| **出处** | ACM TOG **44(1)**: 7:1–7:28；DOI 10.1145/3708034 |

> ⚠️ **归属提示**：这是 TOG 第 44 卷第 **1** 期（非 44(4) 会议轮），在 SIGGRAPH 2025 "Unconventional Fluids" 分会场宣讲。

**核心问题**：小尺度液滴与大水体行为差异极大，表面张力与固壁摩擦可以抵抗重力；现有表面张力格式在**物理正确性与稳定性**上常有问题；固液界面的**摩擦力**则长期被忽视。

**方法要点**：提出新的表面张力计算方法（鲁棒且给出正确量级的张力，采用**隐式**格式）；依据近期实验研究（液滴静止于斜面可用 **Coulomb 摩擦**描述），提出**遵循 Coulomb 模型的固液界面摩擦力格式**；将两种力与压力一并纳入 **IISPH 变体**的**统一求解器**，同时求解**强耦合**的表面张力、摩擦与压力。

**效果**：可再现窗玻璃上雨滴运动、斜面驻留液滴等此前难以处理的现象。

**局限**：Coulomb 摩擦为经验模型；强耦合统一求解的额外开销未见公开数据。

---

### 4.2 Controllable Complex Freezing Dynamics Simulation on Thin Films 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Yijie Liu, Taiyuan Zhang, Xiaoxiao Yan, Han Yan, Nuoming Liu, Bo Ren（南开大学计算机学院） |
| **出处** | DOI 10.1145/3731170 |
| **链接** | http://ren-bo.net/ |

**核心问题**：物理化地模拟**薄液膜（肥皂泡）结冰**的复杂动力学。

**方法要点**：考虑**相态与温度变化对表面张力的影响**，从而复现 **Marangoni 结冰**与 **"Snow-Globe Effect"**（膜上旋转的冰枝晶）。在 SOTA 的 **MELP（Moving Eulerian-Lagrangian Particles）无网格框架**之上引入新的 **Phase Map 方法**，使**枝晶（dendritic crystal）能在移动粒子上生长**，并提供对结冰图案的**精确控制**。

**效果**：可覆盖多种肥皂泡动态结冰过程，且对**复杂边界稳定**。

**局限**：无公开量化数据。

---

### 4.3 A Hybrid Near-wall Model for Kinetic Simulation of Turbulent Boundary Layer Flows 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Mengyun Liu\*, Kai Bai\*, Xiaopei Liu^（上海科技大学） |
| **出处** | ACM TOG 44(4), Art. 148；DOI 10.1145/3730829 |

**核心问题**：高雷诺数下**湍流边界层**难以正确复现。现有经验壁模型多适用于**稳态求解器**，而动态流固交互所需的**非稳态求解器**其壁模型不精确，导致近壁涡形成失真——对笛卡尔网格上的高效**格子玻尔兹曼（LBM）**求解器尤为严重。

**方法要点**：为 LBM 提出**混合近壁模型**，灵感来自**度量边界层分离程度**。模型包含**宏观与介观代数模型**两部分协同，让**低耗散 LBM 求解器自然形成合理的湍流边界层**；结合**多分辨率技术**提升精度。模型**参数化**以近似不同固体表面物理属性对边界层分布的影响，使**不同网格分辨率下可得可比的边界层流动行为**。

**效果**：与实验数据及可视化对比的严格 benchmark 覆盖多种网格分辨率；论文强调在**相对粗网格下仍保持物理一致性**（无具体倍数公开）。应用于计算辅助设计与视觉动画，并与真实实验装置照片对比。

---

### 4.4 Digital Animation of Powder-Snow Avalanches 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Filipe Nascimento, Fabricio S. Sousa, Afonso Paiva（圣保罗大学 ICMC-USP，巴西） |
| **出处** | ACM TOG 44(4), 20 pp.；DOI 10.1145/3730862 |
| **链接** | https://filipecn.github.io/psa_anim/ ；代码 github.com/filipecn/psa_anim |

**核心问题**：粉雪雪崩是最复杂的重力驱动流之一——致密雪核高速下滑，锋面雪粒与空气混合形成悬浮湍流雪尘云。目标是在复杂地形下视觉真实地模拟湍流雪云动力学。

**方法要点**：基于**有限体积法（FVM）**的物理框架加**多层模型**：**致密雪层**用适配复杂基底表面的浅水方程变体——**Savage-Hutter模型**求解；**粉雪层**建模为**可混溶两相混合物**，用 Navier-Stokes 求解；关键创新是**新的过渡层（transition layer）模型**，负责耦合两个主层，包括雪卷吸（snow entrainment）过程向雪云注入的雪质量及其**注入速度**。实现基于 **OpenFOAM v2206**（两个自定义 solver：faSavageHutterFoam、pslFoam）。

**局限与信息缺口**：**未找到公开摘要之外的量化结果与局限说明**。项目页存在 `wolfs_exp.png`，疑为与真实实验场地测量数据的验证图，但**未获证实，不作断言**。

---

## 五、固体与弹性体（6 篇）

### 5.1 CK-MPM: A Compact-Kernel Material Point Method 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Michael Liu, Xinlei Wang, Minchen Li（Minchen Li 为 CMU；其余作者单位未在检索页面明确列出） |
| **链接** | https://arxiv.org/abs/2412.10399 |

**核心问题**：MPM的粒子-网格传输核不连续导致 cell-crossing 不稳定；改用更平滑的形函数（如二次 B-spline）虽缓解问题，却带来更大核支撑、更多数值扩散与传输开销。

**方法要点**：构造 **C² 连续的紧支撑核**，一维形式为 K₁(x) = 1 − |x| + (1/2π)·sin(2π|x|)——由正弦级数抵消线性 B-spline 导数的不连续项而得。核心是**双错格网格（dual staggered grid）框架**：粒子仅与其所在单元内的网格节点关联，保证力计算一致稳定；相对线性核仅使每粒子关联节点数**翻倍**。与 APIC 及 MLS-MPM 无缝集成。

**效果**：G2P2G 传输相对二次 B-spline MPM 额外提速 **2×**；单元测试与压力测试验证线动量与角动量守恒、可处理高刚度材料、大规模场景可扩展；数值扩散低于二次 B-spline。

**局限**：原文为效率/稳定性权衡设计，隐式化适用性当时尚未验证。

---

### 5.2 Fast But Accurate: A Real-Time Hyperelastic Simulator with Robust Frictional Contact 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Ziqiu Zeng, Siyuan Luo, Fan Shi, Zhongkai Zhang（NUS Human-Centered Robotic Lab） |
| **链接** | https://nus-hrl.github.io/fba/ |

**核心问题**：超弹性、非穿透接触与摩擦三者耦合，构成强非线性且非光滑的问题，实时求解困难。

**方法要点**：在 **local-global（投影动力学式）框架**中嵌入**非线性互补条件（NCP）**以获得快速收敛；提出用**系统逆矩阵的稀疏表示**的简易求解器，使原本不适合 GPU 的 local-global 结构获得高度并行性同时保持收敛率；提出**非光滑指示函数的分裂策略**，既提升整体性能又改良互补预条件子，提高摩擦行为精度。核心贡献仅依赖标准矩阵运算。

**效果**：姜饼人算例单物体 **58.5k DoF**，在最多 **800 个接触约束**、穿越狭窄不规则障碍时，每次迭代 **11.95 ms**，每帧 5 次 local-global 迭代；兼容多种超弹性模型，对低刚度与高刚度材料均高效。

---

### 5.3 Hyper-Dimensional Deformation Simulation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Alvin Shi, Haomiao Wu, Theodore Kim（Yale University） |
| **出处** | DOI 10.1145/3721238.3730730 |
| **链接** | https://alvin.pizza/hyperdimensional-deformation/ |

**核心问题**：把可变形体仿真管线整体推广到**四维空间**。

**方法要点**：三方面工作——**网格化**：提出生成 **pentachoral mesh（五胞体网格，四面体的 4D 类比）**的简单方法；**本构**：推广变形不变量，构造 4D 超弹性能量并直接导出超维变形力；**碰撞**：在 4D 中重新表述碰撞检测与响应。对变形能与碰撞能均给出**特征分析（eigenanalysis）**，且该分析可推广到任意更高维度。

**效果**：产生一系列前所未见的视觉现象；项目页提供补充材料、视频与 Bunny 生成/旋转的开源代码。

**局限**：摘要未述；本质属探索性方法，实用性以视觉现象展示为主。

---

### 5.4 Variational Elastodynamic Simulation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Leticia Mattos Da Silva（MIT）、Silvia Sellán（MIT & Columbia）、Natalia Pacheco-Tallaj（MIT）、Justin Solomon（MIT） |
| **链接** | https://www.silviasellan.com/pdf/papers/variational-elastodynamic-simulation.pdf |

**核心问题**：变分积分器的动量守恒性质依赖可靠的非线性求根（通常是牛顿法），而牛顿法昂贵且对病态 Hessian 与差初值敏感。

**方法要点**：证明一大类各向同性弹性能量的**变分时间积分可改写为具有「隐藏凸子结构」的优化问题**；据此使用 **ADMM 加近端算子（proximal operator）步**求解。带来严格收敛性分析、**保证元素不翻转（inversion-free）**、物理不变量在容差与数值精度内守恒。

**效果**：在多个算例上提升弹性仿真任务性能（论文 11 页，正式版 14 页含附录）；代码「即将发布」。

**局限**：适用范围限于「一大类」各向同性畸变能量；ADMM 收敛速度依赖惩罚参数（摘要未展开）。

---

### 5.5 Elastic Locomotion with Mixed Second-order Differentiation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Siyuan Shen, Tianjia Shao, Kun Zhou（浙江大学 CAD&CG 国家重点实验室）、Chenfanfu Jiang（UCLA）、Sheldon Andrews（ÉTS & Roblox）、Victor Zordan（Roblox）、Yin Yang（University of Utah） |
| **出处** | Art. 148；DOI 10.1145/3721238.3730685；arXiv:2405.14595 |

**核心问题**：用户只给高层运动学描述，反求软体的肌肉激活（逆仿真问题）。软体与地面形成大面积接触，含高维不等式约束；一阶可微仿真难以收敛。

**方法要点**：接触用**内点法加对数障碍罚**建模。核心是**混合二阶微分算法**：将解析微分与数值微分结合——把**反向自动微分（AD）视作一个通用函数**（把计算过程映射到对输出损失的导数），再在 AD 计算之上「提升」**复步长有限差分（CSFD）**。为此需对弹性运动中用到的全部算术（初等函数、线性代数、矩阵运算）实现 CSFD 提升。由此可直接对逆问题使用**牛顿法**并享受二阶收敛。

**效果**：teaser 为「体操台灯」跳上矮凳、再跳上更高玻璃桌（两者摩擦不同）、空中翻转加扭转、最后落地；每个时间步（Δt = 1/40 s）优化约 **20 秒**，平均**仅需 2 次牛顿迭代**——这是一阶方法做不到的。

**局限**：每时间步 20 秒属离线优化量级。

---

### 5.6 Neurally Integrated Finite Elements for Differentiable Elasticity on Evolving Domains 【项目页描述 + 摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Gilles Daviet, Tianchang Shen, Nicholas Sharp, David I.W. Levin（NVIDIA Toronto AI Lab，项目名 FlexiSim） |
| **链接** | https://research.nvidia.com/labs/toronto-ai/flexisim ；arXiv:2410.09417 |

> ⚠️ 发表于 TOG 44(2)（2025 年 4 月期）而非会议轨；且该论文亦出现在 NVIDIA 的 SIGGRAPH Asia 2025 论文列表中，**归属轨道存疑**。

**核心问题**：3D 重建越来越多地把几何恢复为隐式函数，但要对这类形状做准确的形变仿真与优化一直很难。

**方法要点**：关键创新是**训练一个小型神经网络来拟合隐式网格单元上的求积点（quadrature points）**，实现鲁棒数值积分；与**混合有限元（Mixed FEM）**表述耦合，得到光滑、**对形状与材料双重可微**的仿真模型，把隐式曲面的演化与其弹性响应连接起来。

**效果**：支持隐式几何的正向仿真、编辑过程中的直接仿真（在雕刻工具中交互运行）、以及与可微渲染结合的**基于物理的形状与拓扑优化**——例如作为 FlexiCubes 重建管线的物理先验重建出结构自持的椅子；与 nvdiffrec 结合重建乐高推土机并依次优化自持形状与材料分布。

---

## 六、杆与绳结（3 篇）

### 6.1 ANIME-Rod: Adjustable Nonlinear Isotropic Materials for Elastic Rods 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Huanyu Chen, Jiahao Wen, Jernej Barbič（University of Southern California） |
| **出处** | ACM TOG 44(4), Art. 87, 23 pp.；DOI 10.1145/3731208 |
| **链接** | https://viterbi-web.usc.edu/~jbarbic/fcurves/ |

**核心问题**：图形学中的 rod 弹性能量都是从体积模型在**小应变线性化**假设下导出，材料被限制为单一的线性 Hooke 定律，却被普遍用于大变形。

**方法要点**：从**任意** 3D 固体非线性各向同性弹性能量密度 ψ 出发，令 rod 厚度趋零取极限，导出 rod 弹性能量，从而在**统一模型**中解释拉伸、弯曲与扭转。关键洞见：用线性理论计算三种截面变形模式（两方向弯曲加扭转，含面内与面外变形），再据此对任意 ψ 推出**5 维「宏观」大变形 rod 能量**的解析公式，自变量为局部纵向拉伸、径向缩放、两个弯曲曲率与扭率。给出所有能量项按截面直径 h 的显式展开，从而可独立调节拉伸/弯曲/扭转。

**可调性公式**：欲使小变形下拉伸/弯曲/扭转分别变为 α×、β×、γ×，则 E ← (β/α²)E，h ← √(α/β)·h，G/E ← (β/γ)·G/E，d ← (β/α)d（d 为质量密度，保总质量不变）；大变形非线性拉伸另加ψ_nonlin = p_stretch·Σᵢ(λᵢ−1)⁴。

**效果**：小变形下与线性理论一致，包含泊松效应导致的截面收缩，弯曲与扭转常数正确；大拉伸大曲率下能量与**体积 FEM 高度吻合**，而图形学常用方法明显偏离；含高分辨率 plectoneme（缠结环）算例。

**局限（作者明确指出）**：**rod 模型本身在控制非线性可弯性与可扭性上存在固有限制**——弯曲与扭转项无论 ψ 为何都对ω₀、ω₁、ω₂ 呈二次依赖；因此建议在图形应用中「放松」rod 物理以便控制非线性弯扭。

---

### 6.2 Stable Cosserat Rods 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jerry Hsu（University of Utah）、Tongtong Wang、Kui Wu（LightSpeed Studios）、Cem Yuksel（University of Utah） |
| **链接** | https://jerryhsu.io/projects/StableCosseratRods/ |

**核心问题**：Cosserat rod 广泛用于头发、树木、纱线级布料，但**鲁棒高效地求解合法四元数取向**极难——即便用很小时间步或昂贵的全局求解器仍不稳定。

**方法要点**：**位置与旋转分离优化（split position/rotation optimization）**方案，配合**闭式解的 Gauss-Seidel 准静态取向更新**。天然适合并行化。

**效果**：高刚度下仍高精度、大时间步下保持稳定；验证覆盖头发、树木、纱线级布料、弹弓、桥梁；相较 **XPBD 与 Discrete Elastic Rods 快数个数量级且更稳定**。已开源。

**局限**：摘要未列；准静态取向更新意味着旋转惯性被简化。

---

### 6.3 Fast Physics-Based Modeling of Knots and Ties using Templates 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Dewen Guo（北京大学）、Zhendong Wang、Zegao Liu（Style3D Research）、Sheng Li、Guoping Wang（北京大学）、Yin Yang（Utah）、Huamin Wang（Style3D Research） |
| **出处** | Art. 43；DOI 10.1145/3721238.3730622 |
| **链接** | https://sites.google.com/view/guo2025fpb/home |

**核心问题**：数字服装中的绳结与领带手工建模极其困难且昂贵；难点在于以**可控、无穿透**方式把布片变形为目标结型，尤其在与周围网格交互时。

**方法要点**：提出**管状参数化结模板**——以 **Bézier 曲线为中轴（medial axis）**、半径可自适应调整，从而支持结的尺寸/形状/风格定制，并借鲁棒碰撞检测保证无自交。基于该模板提出**映射与无穿透初始化**方法，把选定布料区域从 UV 空间变换为初始 3D 结形；随后对结及其周围网格做**准静态仿真**，配以快速可靠的碰撞处理方案。

**效果**：可高效建模风格与形状多样的数字结与领带，包括此前手工难以实现的构型。

**局限**：准静态设定意味着不含动态惯性效应。

---

## 七、布料与针织（3 篇）

### 7.1 Physics-inspired Estimation of Optimal Cloth Mesh Resolution 【摘要原文 + 论文局限章节】

| 项目 | 内容 |
|---|---|
| **作者** | Diyang Zhang, Zhendong Wang, Zegao Liu（Style3D Research）、Xinming Pei、Weiwei Xu（浙江大学 CAD&CG）、Huamin Wang（Style3D Research） |
| **出处** | DOI 10.1145/3721238.3730619 |

**核心问题**：**不做预仿真**的前提下，布料仿真的最优网格分辨率是多少？既要足够捕捉所有潜在褶皱细节，又不能因过密顶点浪费时间与内存。难点在褶皱分布随空间、时间与方向（各向异性）变化。

**方法要点**：基于两个因素估计——**材料刚度**与**边界条件**。材料刚度对褶皱波长与幅值的影响，套用 **Cerda & Mahadevan (2003) 的实验标度律**计算最优分辨率；同一标度律亦用于处理服装工艺引入的静态边界条件（**抽褶 shirring、折叠、缝合 stitching、充绒 down-filling**）以及人体运动碰撞压缩引起的**动态褶皱**预测区域。为使不同源分辨率之间平滑过渡，采用 **Vandeparre et al. (2011)** 的另一实验理论计算**过渡距离**，并生成 **mesh sizing map**；最后以 **Poisson 采样加 Delaunay 三角化**生成最优分辨率分布的三角网格。

**效果**：相比均匀高分辨率与动态 remeshing，在再现真实褶皱的同时降低计算成本，且无传统自适应方法的不连续性与额外开销；在抽褶连衣裙、羽绒服等复杂服装上验证。

**局限（论文明确列出）**：预计算分辨率**缺乏对意外变形的实时适应性**；静态过渡区域难以完全刻画织物行为的突变；对流动织物等不可预测交互的动态环境表现不佳；不考虑黏弹性等复杂材料属性。

---

### 7.2 Real-Time Knit Deformation and Rendering 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Tao Huang\*, Haoyang Shi\*, Mengdi Wang\*（\*共同一作）、Yuxing Qiu、Yin Yang（Utah）、Kui Wu（LightSpeed Studios） |
| **出处** | ACM TOG 44(4), Art. 52；DOI 10.1145/3731184 |
| **链接** | https://kuiwuchn.github.io/rtstitch.html |

**核心问题**：针织结构由互锁纱线构成，每根纱线含多股、每股含数十至数百根扭绞纤维；几何图元数量巨大，实时高保真仿真与渲染极难。

**方法要点**：首个把**纱线级仿真与纤维级渲染**集成的实时框架，输入为动画化的 **stitch mesh**。采用**基于结点（knot-based）的表示**建模互锁纱线接触；结点位置由底层网格插值得到，相关纱线控制点通过**受物理启发的能量表述**优化，并用 **GPU 上的 Gauss-Newton 方案**求解以达实时。优化后的控制点送入 GPU 光栅化管线，渲染为带纤维级细节的纱线。实时渲染中引入若干**分解策略（decomposition strategies）**，在环境光照下也能实现真实光照效果。

**效果**：仿真忠实再现拉伸、剪切等变形下的纱线级结构与互锁行为；渲染达接近 ground truth 的画质，同时比带纤维级几何的**路径追踪参考快 120,000×**；验证场景涵盖小块针织片、整件服装的针织仿真，以及设计管线中的纱线级松弛。

**局限**：依赖输入 stitch mesh 动画（结点位置由网格插值而非完全独立动力学）。

---

### 7.3 High-performance CPU Cloth Simulation Using Domain-decomposed Projective Dynamics 🏅 Honorable Mention 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Zixuan Lu\*, Ziheng Liu\*, Lei Lan（University of Utah）、Huamin Wang（Style3D Research）、Yuko Ishiwaka（SoftBank）、Chenfanfu Jiang（UCLA）、Kui Wu（LightSpeed）、Yin Yang（Utah） |
| **出处** | ACM TOG 44(4), Art. 51, 17 pp.；DOI 10.1145/3731182 |

**核心问题**：高性能布料仿真几乎总被等同于 GPU 加速，CPU 方法被忽视；但高端 GPU 常不可用或已被渲染与着色占用，高性能 CPU 方案能显著提升系统整体能力与用户体验。

**方法要点**：把服装模型划分为**多个（但不是海量）子网格/域（domain）**，把每域计算分配给单个 CPU 处理器。借用**投影动力学**把计算拆为 global 与 local 两步，核心贡献是**在 domain 层级对 global 与 local 两步都建立新的并行范式**，使域内计算是**顺序且轻量**的。CPU 处理单元远少于 GPU，算法通过**明智平衡并行规模与收敛性**来弥补这一劣势。

**效果**：相较现有 CPU 方法**至少快一个数量级**；在许多算例上与最先进 GPU 算法**性能相当，但不使用 GPU**。

**局限**：域分解的收敛性随域数增加而退化，需人工权衡。

> 这是本届最具「反潮流」意味的一篇，其获奖本身就是一种立场表达。

---

## 八、刚体动力学（3 篇）

### 8.1 Painless Differentiable Rotation Dynamics 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Magí Romanyà-Serrasolsas, Juan J. Casafranca, Miguel A. Otaduy（Universidad Rey Juan Carlos, Madrid；MSLab） |
| **出处** | ACM TOG 44(4), 13 pp.；DOI 10.1145/3730944 |
| **链接** | https://mslab.es/projects/Painless/ |

**核心问题**：旋转的各类参数化都有麻烦——旋转矩阵需正交性约束、四元数需归一化约束、旋转向量不唯一且导数行为差。按常规微积分对参数化求导需在导数上强加约束。

**方法要点**：用 **Lie 代数旋转导数**（so(3) 为 SO(3) 在单位旋转处的切空间，配 exp/log 映射）表述**正向与可微刚体动力学**。展示如何轻松套用到正向动力学的**增量势（incremental-potential）**表述，并提出**可微动力学 adjoint 的新定义**——关键新意在于把**步进更新约束表达为旋转**，从而可用 Lie 理论推导 adjoint 计算所需的约束 Jacobian。

**效果**：相较其他参数化（尤其流行的旋转向量参数化），导数**简洁紧凑、条件数更好、运行时效率更高**。在基础刚体问题上验证，并以 **Cosserat rods** 作为多刚体动力学示例。

---

### 8.2 Putting Rigid Bodies to Rest 【摘要原文 + 论文正文】

| 项目 | 内容 |
|---|---|
| **作者** | Hossein Baktash（CMU）、Nicholas Sharp（NVIDIA Research）、Qingnan Zhou（Adobe Research）、Keenan Crane（CMU）、Alec Jacobson（University of Toronto & Adobe Research） |
| **出处** | ACM TOG 44(4), Art. 155, 16 pp.；DOI 10.1145/3731203 |
| **链接** | https://hbaktash.github.io/projects/putting-rigid-bodies-to-rest |

**核心问题**：**不使用物理仿真**，分析与设计刚体的静止构型：给定 R³ 中刚体，在随机初始朝向、动量可忽略的假设下，找出所有静止点及物体停在各点的概率。

**方法要点**：关键观察是**滚动平衡点可从「Gauss 映射上支撑函数」的 Morse–Smale 复形中提取**。方法纯几何，**不用随机采样、不用数值时间积分**；假设动量可忽略、动力学沿重力势梯度演化。可微的**逆向版本**支持形状设计。论文指出朴素的立体角（solid angle）方法只对凸包各面全部稳定的形状才准确。

**效果**：纯几何模型对静止行为给出极其准确的预测，经数值与**实物实验（3D 打印/树脂打印骰子滚动实验）**双重验证；统计量计算比最先进刚体仿真**快数个数量级**，从而打开逆向设计之门。应用含模型自动定向、设计期稳定性反馈、自然场景朝向分布采样；逆向设计出具有目标非均匀概率的骰子——包括用经典方法几乎不可能找到的解，以及单个骰子复现「两枚普通骰子之和」或「抛两/三枚公平硬币的正面数」的统计分布；也能形变 kitten、armadillo 等非凸形状使其恰有三个等概率稳定构型。开源代码加 Blender 插件；被 Ars Technica、New Scientist、SciShow 报道。

**局限（作者明确说明）**：**不预测**给定初始高度/朝向/动量下的最终落姿，只给出基于几何的稳定构型概率分布；忽略动力学效应以及弹性形变等非刚体效应对平衡态分布的影响。

> 这是本届最反直觉的一篇——用 Morse–Smale 复形替代刚体仿真，并有实物骰子实验背书。

---

### 8.3 A Versatile Quaternion-Based Constrained Rigid Body Dynamics 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Guirec Maloisel, Ruben Grandia, Christian Schumacher, Espen Knoop, Moritz Bächer（Disney Research） |
| **出处** | ACM TOG 44(4), Art. 156, 17 pp.；DOI 10.1145/3730872 |
| **链接** | https://la.disneyresearch.com/publication/a-versatile-quaternion-based-constrained-rigid-body-dynamics/ |

**核心问题**：需要一种**保证运动学约束被满足**的受约束刚体动力学，以直接仿真具任意运动学结构的复杂机械系统。

**方法要点**：采用**隐式积分**以确保约束满足。为此推导通过**四元数时间导数**表达的相容动力学方程，四元数更新采取**加性（additive）而非乘性（multiplicative）**方式，并把**四元数单位长度作为约束**强加。支持限制三个平移或三个旋转自由度任意子集的**所有关节类型**，含位置驱动与力驱动；约束的表述方式使 **Lagrange 乘子可直接解释为关节力与力矩**。针对**冗余约束、过驱动（overactuation）与被动自由度**给出统一求解策略：消去冗余约束并在乘子张成的子空间中导航。

**效果**：由于使用标准加性更新，可直接对接**无条件稳定的隐式积分器**；仿真可轻松变为**可微**。

---

## 九、接触与碰撞检测（3 篇）

### 9.1 Offset Geometric Contact (OGC) 【摘要原文 + 论文正文】

| 项目 | 内容 |
|---|---|
| **作者** | Anka He Chen（University of Utah & NVIDIA）、Jerry Hsu、Ziheng Liu（Utah）、Miles Macklin（NVIDIA, 新西兰）、Yin Yang、Cem Yuksel（Utah） |
| **出处** | ACM TOG 44(4), Art. 160, 21 pp.；DOI 10.1145/3731205 |
| **链接** | https://ankachan.github.io/Projects/OGC |

**核心问题**：IPC 类无穿透方法对**余维（codimensional）**物体虽有保证，但计算代价高数个数量级；更关键的是 IPC 的接触模型等价于把面**向所有方向**偏置成 capsule 状体，导致接触力方向不正交，**大接触半径时产生显著伪影**。

**方法要点**：把每个面**沿其法向方向**偏置构造出体积形状，从而保证**接触力正交**，使大接触半径也不产生伪影。计算**逐顶点的位移上界（vertex-specific displacement bounds）**以保证无穿透，这既改善收敛，又**避免昂贵的连续碰撞检测（CCD）**。方法完全依赖大规模并行的**局部操作**，无全局同步，适合高效 GPU 实现；可轻松接入既有求解器（如 Vertex Block Descent）。

**效果**：实时、大规模仿真，性能比先前方法**快两个数量级以上**，且计算预算保持稳定。算例：4 万顶点/7.92 万面、边长 1 m 的方布扭转半圈（接触半径固定 5 mm，IPC 出现伪影而 OGC 正常）；纱线级布料扭转；机器人叠 T 恤；50 层布落到圆柱上共 **50 万顶点**。已开源（Gaia、Newton 两套实现）。

> **注**：该文亦出现在 NVIDIA 的 SIGGRAPH Asia 2025 论文清单中，归属存在交叉，本报告按 physicsbasedanimation 的 2025 北美清单收录。

---

### 9.2 Geometric Contact Potential 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Zizhou Huang, Maxwell Paik, Zachary Ferguson, Daniele Panozzo, Denis Zorin（New York University） |
| **出处** | ACM TOG 44(4), 24 pp.；DOI 10.1145/3731142（前身arXiv:2402.00719） |
| **链接** | https://huangzizhou.github.io/research/smooth-contact.html |

**核心问题**：障碍势（barrier potential）方法虽因严格遵守几何约束（避免相交）而鲁棒，但现有障碍势方法会产生**伪力（spurious forces）**、对几何约束满足不完美，且**强烈依赖分辨率**，参数需针对离散化仔细调节。

**方法要点**：先识别接触势应满足的一组**自然要求**——障碍性质、局部性、对形状的可微依赖、静止构型下无力（基于 candidate sets 的思想）——再由此**系统推导**出定义在光滑及分片光滑曲面上的**连续（continuum）势**。该势的表述**与曲面离散化无关**。给出该势的离散化，可作为 **IPC 表述中所用势的直接替换（drop-in replacement）**。

**效果**：在一系列有挑战性的接触算例上与其他势表述对比，展现预期行为；效率与标准 IPC 相当。理论意义上，该表述**统一了既有障碍方法**——所有近期方法都可视为该势的一个变体——并为发展替代版本（如高阶版本）奠定基础。

**局限**：当前离散化适用于分片线性曲面。

---

### 9.3 C⁵D: Sequential Continuous Convex Collision Detection Using Cone Casting 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Xiaodi Yuan, Fanbo Xiang, Hao Su（UC San Diego）、Yin Yang（University of Utah） |
| **链接** | history.siggraph.org 官方摘要归档（Session: Rigid Bodies & Contacts）；代码 https://github.com/Rabbit-Hu/c5d |

**核心问题**：在刚体或近刚体的物理仿真中，尤其要强制无相交约束时，碰撞往往是首要性能瓶颈。以往框架依赖**图元级（primitive-level）CCD**，因待处理的碰撞表面图元数量庞大而计算密集，并**严重依赖 GPU 等高级并行资源**——而在机器人策略训练等应用中，GPU 常因任务竞争或线程能力上限而不可得。

**方法要点**：提出针对**做恒定仿射运动的凸形状**的**顺序（sequential）CCD 算法**。使用**保守推进（conservative advancement）**方法迭代精化 TOI（time of impact）的下界估计，利用仿射运动的线性性与凸形状距离计算的高效性。无缝集成进 **ABD（Affine Body Dynamics）**框架。

**效果**：相较图元级 CCD 取得 **10 倍加速**；其高**单线程效率**进一步使得通过**场景级并行**获得显著吞吐提升，非常适合资源受限环境。已开源。

**局限**：限定于**凸形状**与**恒定仿射运动**假设；面向刚体/近刚体（ABD），非通用可变形体。

---

## 十、GPU 与数值基础设施（8 篇）

这是本届最密集、加速幅度最大的一组。

### 10.1 StiffGIPC: Advancing GPU IPC for Stiff Affine-Deformable Simulation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Kemeng Huang, Xinyu Lu, Huancheng Lin, Taku Komura（香港大学）, Minchen Li（CMU） |
| **出处** | ACM TOG **44(3)**, Art. 31, 20 pp.；DOI 10.1145/3735126；arXiv:2411.06224 |
| **链接** | 代码 https://github.com/KemengHuang/Stiff-GIPC |

> ⚠️ 注意是TOG 44(3)（2025 年 5 月刊），非 SIGGRAPH 2025 会议轨。

**核心问题**：IPC 鲁棒但慢；材料刚度升高时 PCG 收敛急剧变慢，现有预条件子也无法挽救。

**方法要点**：三要点——GPU 上的**连接性增强 MAS（Multilevel Additive Schwarz）预条件子**，同时兼顾刚性与柔性弹性动力学，以更低预条件成本改善 PCG 收敛；用于应变限制的 **C²-连续三次能量加解析特征系统**，使刚性膜（布料）并行友好且无膜锁定；对「弹性波不可见」的极刚体改用**仿射体动力学 ABD**，配合**哈希两级/多层归约**加速 Hessian 装配与仿射-可变形耦合。

**效果**：相较 SOTA GPU IPC 方法**最高 10× 加速**；在软/刚/混合、高分辨率、大变形、高速撞击场景中均为最快。

**局限**：需 NVIDIA GPU、依赖 METIS 做域分解。

---

### 10.2 JGS2: Near Second-order Converging Jacobi/Gauss-Seidel for GPU Elastodynamics 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Lei Lan, Zixuan Lu, Chun Yuan（Utah）, Weiwei Xu（浙大 CAD&CG）, Hao Su（UCSD）, Huamin Wang（Style3D Research）, Chenfanfu Jiang（UCLA）, Yin Yang（Utah） |
| **出处** | ACM TOG 44(4), Art. 44, 15 pp.；DOI 10.1145/3731183 |

**核心问题**：并行性与收敛性天然冲突——局部求解越轻、耦合越弱，全局收敛越慢。

**方法要点**：识别并刻画 **overshoot（过冲）**现象：局部求解器只顾降低局部能量、无视全局上下文，导致更新反而破坏全局收敛。据此推导**理论上二阶最优的抗过冲修正**，并将其改写为**可预计算形式**；借助 **Cubature 采样**使运行时开销仅略高于 Jacobi；另提出**全坐标（full-coordinate）表述**以提升预计算效率。与 IPC 无缝集成。

**效果**：收敛速度接近牛顿法的二次收敛；相较 SOTA GPU 方法**收敛性好 50×–100×**；软/刚材料均达二阶收敛。

**局限**：依赖预计算（Cubature 加子空间信息），对拓扑变化不友好。旁证：该组2026 年后续工作 JGS2-GQ 主打「免训练/免预计算」，反向印证原版预计算确是痛点。

---

### 10.3 MGPBD: A Multigrid Accelerated Global XPBD Solver 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Chunlei Li, Peng Yu, Tiantian Liu, Siyuan Yu, Yuting Xiao, Shuai Li, Aimin Hao, Yang Gao, Qinping Zhao（北京航空航天大学为主） |
| **出处** | SIGGRAPH Conference Papers, Art. 111；DOI 10.1145/3721238.3730720；arXiv:2505.13390 |
| **链接** | https://chunleili.github.io/project-page-mgpbd ；代码 github.com/chunleili/mgpbd |

**核心问题**：XPBD 的非线性 Gauss-Seidel 只擅长消高频误差，低频误差导致高分辨率/高刚度下停滞（stalling）甚至崩溃；且它丢弃系统矩阵非对角项，本质是局部求解器。

**方法要点**：首次在**对偶空间（dual space）**用 AMG——采用**非平滑聚合 UA-AMG 加 PCG**；**lazy setup** 复用 prolongator（默认每 20 帧重建一次）；用**对齐次方程 Ax=0 做 20 次 GS 扫掠、重复 6 次**生成 6 个 near-kernel 分量（对应 6 个刚体模态），达到 adaptive-SA 级收敛但成本更低。

**效果（本组量化数据最丰富的一篇）**：
- 单次迭代 **0.54 s**，对比 AMGCL **1.63 s** / AMGX **3.85 s** / MKL PARDISO **91.98 s**（约 2–3 个数量级优势）
- lazy setup 把 setup 占比从**约 2/3 降到 2%**
- 用 λ_min≈0.1 近似在 85 万四面体案例再提速 **24%**
- 人体肌肉：**166.7 万四面体、52.6 万顶点、14 万外部约束、每帧 20 次迭代、40.6 s/帧**
- 怪物肌肉 7.2 万四面体 2.3 s/帧（同预算下 XPBD 直接崩溃）
- 85 万四面体下 XPBD 发散而 MGPBD 稳定；布料 512² 下 XPBD 即使 10⁵ 次迭代仍停滞
- 可扩展性线性，**R² = 0.9978**
- 硬件：AMD 7950X + RTX 4090，CUDA 加 Taichi

**局限（原文明列）**：仅支持**静态拓扑**，需显式构建稀疏矩阵，未来拟做matrix-free AMG 以支持撕裂/断裂；全局系统引入额外振荡，需阻尼；碰撞处理简单（仅静态 SDF 位置式后处理）。

---

### 10.4 Parth: Adaptive Algebraic Reuse of Reordering in Cholesky Factorizations with Dynamic Sparsity Patterns 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Behrooz Zarebavani（多伦多大学）, Danny M. Kaufman（Adobe）, David I.W. Levin（NVIDIA / 多伦多）, Maryam Mehri Dehnavi（NVIDIA Research / 多伦多） |
| **出处** | ACM TOG 44(4)；DOI 10.1145/3731179；arXiv:2501.04011 |

**核心问题**：在 IPC 类接触仿真与局部重网格中，稀疏模式在相邻两次求解间不断变化，**符号分析（尤其 fill-reducing reordering）反而成为 Cholesky 求解器的主要瓶颈**。

**方法要点**：对矩阵对偶图做**分层图分解（HGD）**切成细粒度子图；用 Synchronizer 追踪节点增删并标记「脏」子图；Assembler 只重算受影响子图的局部置换向量并拼装全局排序，从而在时间相干时**选择性复用**排序。

**效果**：在 **17.5 万以上线性系统**上评测；最难的物理仿真中 reordering 最高 **14× 加速**，带来 Cholesky 求解 **2× 加速**（且是叠加在 Apple Accelerate / Intel MKL 之上）。IPC "Squeeze Out" 基准符号分析 7.1× → 整体 2.2×；重网格 patch 占 2% 网格时可复用 **92%** 的排序计算，求解 5.5× 加速。**集成只需 3 行代码**，fill-reduction 质量在最优的 ±5% 内。

> arXiv 早期版本报告了更高的数字（IPC reordering 最高 255×、remeshing 13×），与正式版口径不同，本报告以 v2 正文为准。

**局限**：只优化符号/排序阶段，数值分解本身未改进；依赖稀疏模式的时间相干性，若拓扑剧变则收益消失。

---

### 10.5 Stochastic Barnes-Hut Approximation for Fast Summation on the GPU 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Abhishek Madan（多伦多大学）, Nicholas Sharp, Francis Williams, Ken Museth（NVIDIA）, David I.W. Levin（多伦多 / NVIDIA） |
| **出处** | SIGGRAPH Conference Papers；DOI 10.1145/3721238.3730725；arXiv:2506.02219 |
| **链接** | https://www.dgp.toronto.edu/projects/stochastic-barnes-hut |

**核心问题**：快速求和（N 体、winding number、BEM 场求值）中，确定性 Barnes-Hut 的 LOD 截断带来偏差且 GPU 分支发散；纯随机方法收敛率差。

**方法要点**：**把 LOD 近似族视为控制变量（control variates）**，构造被近似核和的**无偏估计器**——只需估计「真值与 LOD 近似之间的残差」。天然 GPU 友好：可用采样预算控制成本、embarrassingly parallel。

**效果**：在 winding number 计算与光滑距离场求值上，达到同等中位误差时比 GPU 优化版确定性 Barnes-Hut **快最多 9.4×**；变电站电势示例：2²² 个表面样本、1000² 切片平面，每个树子域1 个样本，**8 ms 完成等效超4 万亿次粒子-粒子交互**（确定性 BH 14 ms，暴力 2800 ms），几乎无可见伪影。

**局限**：随机方法固有噪声，低样本数下有噪点；无偏性依赖控制变量质量。

---

### 10.6 Lightning-Fast Boundary Element Method 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jiong Chen（Inria Saclay）, Florian Schäfer（Georgia Tech）, Mathieu Desbrun（Inria / École Polytechnique） |
| **出处** | ACM TOG 44(4), Art. 38, 14 pp.；DOI 10.1145/3731196 |
| **链接** | PDF https://pages.saclay.inria.fr/mathieu.desbrun/pubs/CSD25.pdf |

**核心问题**：BEM 免体网格、未知量少，但边界积分方程（BIE）产生**稠密且病态**的线性系统，大规模下可扩展性与内存都很差。

**方法要点**：把 **Kaporin 稀疏近似逆思想推广到非对称预条件**——以大规模并行方式构造任意 BIE 矩阵的**稀疏 inverse-LU 分解**，用作 GMRES 的预条件子。关键词含 "screening effect"（屏蔽效应，为稀疏性提供理论依据）。

**效果（teaser）**：从梵高《鸢尾花》提取 **660 万条显著线段**作Dirichlet 边界条件，解 Laplace 方程扩散颜色，生成 9600×7413（6400 万外插像素）图像。本方法预条件下 GMRES **20 次迭代**即达相对误差 < 0.001，而常规 Jacobi 预条件需 **4200 次**——**墙钟时间加速 200 倍以上**（15 分钟 vs 每通道 2.1 天）。作者称通常可获数量级加速。

**局限**：dl.acm.org 与 Inria PDF 均抓取失败，**limitations 章节未取得**。

---

### 10.7 MiSo: A DSL for Robust and Efficient MINIMIZE and SOLVE Problems 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Federico Sichetti, Enrico Puppo（热那亚大学）, Zizhou Huang（NYU→Roblox）, Marco Attene（CNR-IMATI）, Denis Zorin, Daniele Panozzo（NYU Courant） |
| **出处** | ACM TOG 44(4), 18 pp.；DOI 10.1145/3731207 |

**核心问题**：图形学中大量问题可归约为「求带非线性约束的全局最小值（Minimize）」或「求非线性约束系统的全部解（Solve）」，但每次都要手写高度优化且数值鲁棒的代码。

**方法要点**：**领域专用语言加编译器**，为低维Minimize/Solve 问题生成高效 C++ 代码；核心是用**区间方法（interval methods）**在浮点算术下保证**保守（conservative）**结果——即不漏解、不误判。

**效果**：生成代码性能与手工优化实现**具竞争力**，验证于三类应用：**非线性轨迹的高阶连续碰撞检测、曲面-曲面求交、有限元仿真的几何有效性检查**。

**局限**：明确限定「低维」问题；区间方法的保守性通常以效率为代价。原文 limitations 章节未取得。

---

### 10.8 Automated Task Scheduling for Cloth and Deformable Body Simulations in Heterogeneous Computing Environments 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | 何成竹（厦门大学 & Style3D Research，第一作者）、王振东（Style3D）、孟昭芮（厦大）、姚俊峰（厦大，通讯）、郭诗辉（厦大）、王华民（Style3D） |
| **出处** | SIGGRAPH Conference Papers, 11 pp.；DOI 10.1145/3721238.3730625 |
| **链接** | 代码 https://github.com/ChengzhuUwU/libAtsSim |

**核心问题**：SoC 设备（CPU+GPU+NPU 同片）上，布料/软体仿真的迭代任务如何跨异构单元分配——难点在任务间依赖与通信成本，同步开销常抵消并行收益。

**方法要点**：把仿真任务（碰撞检测、约束迭代等）建模为 **DAG**，用改造后的 **HEFT（Heterogeneous Earliest Finish Time）**调度；提出**异步 Gauss-Seidel** 方法，跨单元异步数据同步以掩盖通信延迟；辅以**任务合并与定制排序策略**平衡收敛性与并行度。

**效果**：在 **Apple M 系列**处理器上验证，覆盖 **XPBD、Vertex Block Descent (VBD)、Second-order Stencil Descent (SOSD)、Jacobi-preconditioned GD** 四种求解器。XPBD 场景（多层连衣裙、篮球球衣）相较**仅用 GPU 核心提速 40%–60%**。有意思的发现：XPBD 的 CPU↔GPU 通信最频繁，反而是它比同步 GS 收敛更好的原因；VBD/SOSD 因 CPU 任务耗时长，通信不那么频繁。

**局限**：验证仅限 Apple M 系列统一内存架构；异步迭代牺牲部分收敛确定性。

---

## 十一、自适应与 LOD（2 篇）

本届自适应方向的一个重要信号：**已配套 SIGGRAPH Course「Level-of-Detail for Geometry Processing and Simulation」**（组织者 Jiayi Eris Zhang 与 Hsueh-Ti Derek Liu），说明这个议题已从「技巧」升格为「体系化课题」。

### 11.1 Progressive Dynamics++ 【摘要原文，量化数据未取得】

| 项目 | 内容 |
|---|---|
| **作者** | Jiayi Eris Zhang（Adobe & Stanford）, Doug L. James（Stanford）, Danny M. Kaufman（Adobe） |
| **出处** | ACM TOG 44(4), Art. 53, 20 pp.；DOI 10.1145/3731202 |
| **链接** | https://pcs-sim.github.io/pd++/ |

**核心问题**：原 Progressive Dynamics [Zhang et al. 2024] 在**稳定性**与**时序连续性**上存在根本缺陷（层级增多时数值爆炸、帧间跳变/抖动）。

**方法要点**：提出**通用框架**，用于构造一族「同时在时间与空间分辨率两个方向推进物理状态」的渐进动力学积分器——原 2024 方法只是其中一个成员。三项贡献：分析渐进动力学积分器的**必要稳定性条件**，给出新的稳定方法；提出**新的时序连续性定量度量**，并证明新方法显著改善连续性；对**几何一致性与细节增强（enrichment）**的权衡做定量分析，给出跨分辨率/跨时间转换时的平衡策略（用户可控）。

**效果**：展示案例 five-hat-trick——把软帽子抛到衣帽架上。用 Level 0 粗预览快速搜索大量初始条件找到「针尖上的」成功样本，再由 Levels 1–3 渐进合成到**数百万顶点**的产品级动画，且五帽结果保持一致。层级构建用 quadric error edge collapse，层级间有位置与速度两套prolongation 算子。

**信息缺口**：**未找到公开的量化表格与 limitations 原文**（PDF 为压缩流、ACM 页面被 Cloudflare 拦截）。

> **系列关联**：SIGGRAPH 2026 的 Progressing Level-of-Detail Animation for Volumetric Elastodynamics 是本文从布料/壳到体有限元的推广。

---

### 11.2 Optimal r-Adaptive In-Timestep Remeshing for Elastodynamics 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jiahao Wen（Adobe & USC）, Jernej Barbič（USC）, Danny M. Kaufman（Adobe） |
| **出处** | ACM TOG 44(4), 19 pp.；DOI 10.1145/3731204 |
| **链接** | http://www.jernejbarbic.com/radaptivity |

**核心问题**：Ferguson et al. [2023] 的 ITR（in-timestep remeshing）用增量势能作细化目标加**局部贪心**重网格，因此**不最优**——网格变细时物理解不收敛改善，还会产出低质量几何与物理伪影。

**方法要点**：三项贡献——对增量势能做「简单但关键」的修正，构造**新的重网格目标函数加带约束的模型问题**，其极小点给出**局部最优**重网格；提出新的 in-timestep 优化，**每步联合求解「新的最优网格」与「定义其上的下一物理状态」**；为实现 r-adaptivity（只移动节点、不改拓扑）另提三项技术：鲁棒计算 **L²-投影算子的导数**、**边界上自适应的约束模型**、高效的牛顿型优化器。

**效果（teaser，咀嚼橡胶棒）**：最优 ITR 的粗模型与细网格解的 **Hausdorff 距离 0.05**，而原 ITR 为 **0.16**；相比细网格仿真自由度与内存低数个数量级，**8.4× 加速**；原 ITR 用了 **8.5× 更多 DOF 却慢 73×**。

**局限**：r-adaptive 仅移动节点、不改变拓扑，故细化能力受初始网格连通性上限约束；每步联合优化本身成本不低。原文 limitations 章节未取得。

---

## 十二、神经场与 AI 交叉（2 篇）

### 12.1 Lifting the Winding Number: Precise Discontinuities in Neural Fields for Physics Simulation 🏅 Honorable Mention 【摘要原文 + 论文正文】

| 项目 | 内容 |
|---|---|
| **作者** | Yue Chang, Mengfei Liu, Zhecheng Wang（多伦多大学）, Peter Yichen Chen（MIT CSAIL）, Eitan Grinspun（多伦多大学） |
| **出处** | SIGGRAPH Conference Papers, Art. 25；DOI 10.1145/3721238.3730597；arXiv:2502.00626 |
| **链接** | https://www.dgp.toronto.edu/projects/windlifter/index.html |

> 这是本届 AI × 物理方向最有分量的一篇，也是本节量化数据最完整的一篇。

**核心问题**：切割薄壳结构会引入空间不连续。网格法需频繁重网格；降阶（ROM）仿真的基函数本质依赖几何与网格，难以表达切割带来的多样不连续；而神经场天生连续，若把不连续「烘焙」进网络权重就无法泛化到新切割位置。

**方法要点**：用**广义缠绕数（generalized winding number, GWN）**增广神经场的输入坐标，把 2D 输入**提升（lift）到 3D**——网络只需学一个处处连续的 3D 体场（更容易的问题），再由对应的**限制算子（restriction operator）**在输出端精确解析出严格不连续。关键在于：不连续**不编码在网络权重中**，因此切割位置可泛化、可交互编辑、仿真中可动态更新切割线几何。裂尖应变奇异性用 ε-邻域三次样条核平滑处理。

**效果**：
- 切割变化时 **28.5–42.4 fps**，无切割变化时 **62–166 fps**；全空间仿真约 **1.1 s/帧**，故加速 **30×–183×**
- 硬件 RTX 4090 + AMD 7950X；网络为 **5 层 128 通道 SIREN MLP**，缠绕数缩放因子 32
- 模态数 data-free k=18、data-driven k=20，且**同一套权重参数化全部 k 个位移基**
- 训练时间 **53 s 至 2760 s**（不足1 分钟到略少于1 小时），远低于需数小时/数天的同类神经方法
- 误差：训练集上所有data-driven 方法重建误差 < 0.14%，本方法最低；相同设置下比 **LiCROM 的 MSE 低一个数量级**且收敛更快；切口偏离训练集 **< 15%** 时无可见伪影
- 对比基线：LiCROM、Simplicits、DANN。真实剪纸螺旋实验定性吻合

**局限（原文明列）**：**碰撞处理缺失**——需（自）碰撞检测与响应；切割表示**仅限分段线性折线**，未来可扩展到 Bézier 等高阶参数曲线；**分布外泛化**仍具挑战——测试顺/逆时针螺旋切口不一致时最终变形错误。

> **系列关联**：该组在 SIGGRAPH Asia 2025 有直接后续 Precise Gradient Discontinuities in Neural Fields for Subspace Physics（把数值不连续扩展到梯度不连续）。

---

### 12.2 Dress-1-to-3: Single Image to Simulation-Ready 3D Outfit with Diffusion Prior and Differentiable Physics 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Xuan Li\*, Chang Yu\*, Wenxin Du\*, Ying Jiang\*, Tianyi Xie, Yunuo Chen, Chenfanfu Jiang（UCLA）, Yin Yang（Utah）（\*同等贡献） |
| **出处** | ACM TOG 44(4), 16 pp.；arXiv:2502.03449 |
| **链接** | https://dress-1-to-3.github.io/ |

**核心问题**：现有 image-to-3D 大模型输出**融合成一整块**的模型，无法用于虚拟试衣或动态服装动画等需要「服装可分离且 simulation-ready」的下游任务。单图还有深度缺失与遮挡导致的重建歧义。

**方法要点**：端到端管线四步——预训练 **image-to-sewing-pattern** 模型给出粗略缝纫图案；预训练**多视角扩散模型**生成环绕相机视图，作为人体姿态与服装形状的「伪 3D 真值」；用**可微服装模拟器**把图案缝合并披挂到 SMPL-X 姿态人体上，联合优化图案形状与物理参数加几何正则项；接纹理生成与人体动作生成模块产出动态演示。关键工程点：把 SOTA 布料仿真算法 **C-IPC（Codimensional IPC）可微化**，做 simulation-in-the-loop 优化；用 Delaunay 三角化加调和坐标实现固定拓扑可微网格参数化；检测到病态三角形时自动重网格并回拉到初始离散化以避免穿透。

**效果**：重建 3D 服装与人体和输入图的**几何对齐显著提升**；输出含缝纫图案、纹理、材质参数、姿态人体，可直接进入布料仿真或虚拟试衣。与 Neural Tailor、SewFormer 对比在面板形状预测上更干净。作者强调「只需手机拍一张照，无需专业设备」。

**局限**：单图信息有限，物理参数（如刚度）估计不充分；依赖 SMPL-X、OSX、DWPose 的姿态估计足够准确这一前提。**作者明示的未来工作**：扩展到视频输入以改善刚度等物性估计。

---

## 十三、World Model × 物理模拟：本届的准确定位

必须严格区分三个层次：**厂商keynote/发布 ≠ Course/Lab ≠ 同行评审技术论文**。

### 13.1 核心判断

**SIGGRAPH 2025 未见任何以 "world model" 为主题的 Technical Paper、Course 或 Workshop。** 相关内容 100% 集中在 NVIDIA 的 Special Address 与配套产品发布。

### 13.2 Keynote / 厂商发布层面 【摘要原文】

**NVIDIA Research Special Address**，2025-08-11 16:00 PDT，讲者 **Sanja Fidler（AI 研究 VP）、Aaron Lefohn（图形研究 VP）、Ming-Yu Liu（研究 VP, Deep Imagination Lab）**。官方描述明确包含"world foundation models essential for advancing physical AI"。黄仁勋另有特别视频致辞。

Fidler 的核心论点值得引用：「**AI 正在提升我们的仿真能力，而仿真能力反过来推动 AI 系统——两者之间存在真实且强大的耦合。**」Ming-Yu Liu：「物理 AI 需要一个触感真实的虚拟环境，一个让机器人能通过试错安全学习的平行宇宙；构建它需要五大技术：实时渲染、计算机视觉、物理运动仿真、2D/3D 生成式 AI、AI 推理。」

**Cosmos 在 2025 年 8 月的状态**（厂商发布，非论文）：
- **Cosmos Reason**：**70 亿参数**推理型视觉语言模型，面向物理 AI 与机器人，开源可商用；训练分两阶段——物理常识监督微调加基于物理反馈的强化学习；可作规划模型推断具身智能体下一步
- **Cosmos Transfer-2**：从 3D 仿真场景或空间控制输入加速生成合成数据，另有蒸馏加速版
- **Cosmos Predict**：输入图像或起始视频预测后续视频，生成未来世界状态——这是最接近经典 world model 定义的组件
- **累计下载量突破 200 万次**（2025 年 1 月发布至 8 月）
- 配套发布：**Omniverse NuRec**（3DGS 神经重建库，已整合进开源自动驾驶模拟器 CARLA）、新 Omniverse SDK（MuJoCo MJCF ↔ OpenUSD 互操作）、**Isaac Sim 5.0 / Isaac Lab 2.2 开源**、**ViPE（Video Pose Engine）**、RTX PRO 6000 Blackwell Server Edition

### 13.3 Course / Lab 层面【摘要原文】

**没有以 differentiable simulation 命名的独立 Course 或 Technical Workshop。** 相关载体有三个：

1. **Course: Level-of-Detail for Geometry Processing and Simulation** —— 组织者 **Jiayi Eris Zhang**（即 Progressive Dynamics++ 一作）与 Hsueh-Ti Derek Liu。这是第十一节两篇论文的直接课程化载体，是「自适应/LOD 已成体系化议题」的最强证据。
2. **NVIDIA Hands-On Class：Tackling Gaussian Splats, Physics Simulation, and Visualization With NVIDIA Kaolin and Warp Libraries** —— 其中 35 分钟专讲 **Warp 与 FlexiSim 的可微物理 kernel 编写**，覆盖 Simplicits 新增碰撞特性、3DGS 与网格联合仿真渲染。组织者含 **Gilles Daviet**（即第 5.6 篇作者）。
3. **NVIDIA Lab：Synthetic Data Generation for Robot Learning Pipelines With NVIDIA Cosmos** —— Isaac Sim 加 Cosmos WFM 生成合成数据，是 world model 与仿真结合的**教学层面**证据。

### 13.4 论文层面的 AI × 物理：两条路径，不含 learned simulator

**明确的负面结论**：**未检索到 SIGGRAPH 2025（北美）有以「用神经网络替代整个数值求解器」（GNN-based learned simulator、MeshGraphNets 类）为核心的技术论文。**

本届 AI × 物理只走两条路：
- **(a) 神经场作为降阶表示**：Wind Lifter（最佳论文荣誉提名）
- **(b) 生成模型加可微物理做逆问题**：Dress-1-to-3

另有Fast Subspace Fluid Simulation 属「数据驱动降阶」（DMD）而非神经网络。

这个负面结论本身很有价值——**物理模拟社区并未被端到端学习取代，反而是数值基础设施在 2025 年集中爆发**，恰与第十节的 8 篇形成呼应。

### 13.5 Physics-based character control 与 RL（session层面）

SIGGRAPH 2025 有专门 session **"Physics-Based Human Characters"**（moderator: Yuting Ye）。确证论文：

| 论文 | 要点 |
|---|---|
| **PLT: Part-Wise Latent Tokens as Adaptable Motion Priors for Physically Simulated Characters** | Jinseok Bae 等（首尔大学）。部位化 codebook 加 refinement network，支持身体追踪、复杂地形导航、肢体损伤下的point-goal 导航 |
| **Diffuse-CLoC: Guided Diffusion for Physics-based Character Look-ahead Control** |在单一扩散模型中联合建模state-action 分布，无需高层规划器 |
| **AMOR: Adaptive Character Control through Multi-Objective Reinforcement Learning** | Disney Research Zürich，Art. 85。训练后可调奖励权重（Pareto front），含真实机器人 sim-to-real |
| **PARC: Physics-based Augmentation with Reinforcement Learning for Character Controllers** | Michael Xu 等（SFU）、Xue Bin Peng（SFU/NVIDIA）。运动生成器与物理追踪控制器**迭代互相扩充数据集**，用于跑酷类地形穿越 |
| **Elastic Locomotion with Mixed Second-order Differentiation** | 见5.5，逆仿真求最优肌肉激活（非 RL，但属物理驱动运动控制） |

> **注**：PhysicsFC: Learning User-Controlled Skills for a Physics-Based Football Player Controller 出现在 **Emerging Technologies** 而非 Technical Papers，请勿混淆。

---

## 十四、奖项、量化一览与开源清单

### 14.1 物理模拟相关奖项

| 奖项 | 论文 |
|---|---|
| 🏅 **Best Paper Honorable Mention** | Clebsch Gauge Fluid on Particle Flow Maps |
| 🏅 **Best Paper Honorable Mention** | Lifting the Winding Number（神经场精确不连续） |
| 🏅 **Honorable Mention** | High-performance CPU Cloth Simulation Using Domain-decomposed Projective Dynamics |

### 14.2 量化加速一览（横向对比）

| 论文 | 加速幅度 | 对比基线 |
|---|---|---|
| Real-Time Knit Deformation and Rendering | **120,000×** | 纤维级几何路径追踪参考 |
| Lightning-Fast BEM | **>200×**（墙钟） | Jacobi 预条件 GMRES |
| MGPBD | **2–3 个数量级**（单次迭代） | AMGCL / AMGX / PARDISO |
| Offset Geometric Contact | **>2 个数量级** | 先前 IPC 类方法 |
| Stable Cosserat Rods | **数个数量级** | XPBD / Discrete Elastic Rods |
| Wind Lifter | **30×–183×** | 全空间仿真 |
| JGS2 | **50×–100×**（收敛性） | SOTA GPU 方法 |
| Domain-decomposed PD | **≥1 个数量级** | 现有 CPU 方法 |
| C⁵D | **10×** | 图元级 CCD |
| StiffGIPC | **10×** | SOTA GPU IPC |
| Stochastic Barnes-Hut | **9.4×** | GPU 确定性 Barnes-Hut |
| Optimal r-ITR | **8.4×** | 细网格仿真 |
| Fast Galerkin MG（Asia） | — | 见 Asia 报告 |
| Parth | **14×**（reordering）/ **2×**（整体） | Apple Accelerate / MKL |
| VPFM | 流图长度 **3–12×** | SOTA 流图方法 |
| EDGE | 内存降**90%** | buffer-based EFM |
| Cirrus | **1.5–2×**（vs PFM）/ **1–2 个数量级**（vs 均匀网格） | PFM / 均匀网格 |
| CK-MPM | **2×**（G2P2G） | 二次B-spline MPM |
| Automated Task Scheduling | **40%–60%** | 仅用 GPU 核心 |

### 14.3 规模记录

| 论文 | 规模 |
|---|---|
| Adaptive PF-FLIP | **数十亿粒子**，有效分辨率 4096×1024×512，单工作站 |
| MGPBD | **166.7 万四面体 / 52.6 万顶点**，40.6 s/帧 |
| Lightning-Fast BEM | **660 万边界线段**，9600×7413 输出 |
| Cirrus | 有效分辨率 **512×512×2048**，单 RTX 4090 |
| Offset Geometric Contact | 50 层布 **50 万顶点** |
| Progressive Dynamics++ | **数百万顶点**产品级动画 |
| Stochastic Barnes-Hut | 等效 **4 万亿次**粒子-粒子交互，8 ms |
| Fast But Accurate | 58.5k DoF，800 接触约束，11.95 ms/迭代 |

### 14.4 开源项目清单

| 论文 | 仓库 |
|---|---|
| VPFM | github.com/pfm-gatech/VPFM |
| Cirrus | github.com/wang-mengdi/Cirrus |
| LFM | github.com/yuchen-sun-cg/lfm |
| Compressible Flow Maps | github.com/CDWJ/compressible-fm |
| Adaptive PF-FLIP | github.com/tum-pbs/MSBG |
| Powder-Snow Avalanches | github.com/filipecn/psa_anim |
| StiffGIPC | github.com/KemengHuang/Stiff-GIPC |
| MGPBD | github.com/chunleili/mgpbd |
| C⁵D | github.com/Rabbit-Hu/c5d |
| Automated Task Scheduling | github.com/ChengzhuUwU/libAtsSim |
| Hyper-Dimensional Deformation | 见项目页 |
| Stable Cosserat Rods | 已开源（见项目页） |
| Offset Geometric Contact | Gaia、Newton 两套实现 |

---

## 附录：信息缺口声明

**未取得量化数据或limitations 原文的论文**（ACM Cloudflare 拦截或 PDF 压缩流所致）：
- Progressive Dynamics++：计时表与 limitations 章节
- Optimal r-Adaptive ITR：完整结果表与 limitations
- Lightning-Fast BEM、MiSo：limitations 章节
- Digital Animation of Powder-Snow Avalanches：分辨率、计时与真实数据验证情况
- Clebsch Gauge Fluid、LFM、Compressible Flow Maps、Quadtree Tall Cells、Gaussian Fluids、Fast Subspace Fluid、Neural PLS、Unified SPH、Freezing Thin Films、Near-wall Model：均无公开量化性能数据

**作者机构未经证实（标注为推断）**：
- CK-MPM 部分作者（仅确认 Minchen Li 为 CMU）
- Neural Particle Level Set（据主页域名推断为 Georgia Tech）
- LFM 的 Sandia National Labs 归属

**归属存疑的论文**：
- Unified Pressure, Surface Tension and Friction for SPH Fluids：TOG 44(1) 而非 44(4)
- StiffGIPC：TOG 44(3) 而非会议轨
- Neurally Integrated Finite Elements：TOG 44(2)，且同时出现在 NVIDIA 的 SIGGRAPH Asia 2025 清单中
- Offset Geometric Contact：亦出现在 NVIDIA 的 SIGGRAPH Asia 2025 清单中

**其他**：
- physicsbasedanimation 原清单 49 条中，"Elastic Locomotion with Mixed Second-order Differentiation" 重复出现两次（第 25、42 条），去重后为 48 篇
- 检索环境下conference-schedule 类站点 TLS 受限无法直连，本报告以 physicsbasedanimation.com 合集帖为主干，辅以 history.siggraph.org 官方摘要归档

---

*报告生成时间：2026 年 8 月 10 日*
