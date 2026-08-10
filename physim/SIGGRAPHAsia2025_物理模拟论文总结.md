# SIGGRAPH Asia 2025 物理模拟论文逐篇总结

> **会议信息**：SIGGRAPH Asia 2025（第 18 届），2025 年 12 月 15–18 日，中国香港会议展览中心。大会主席 Taku Komura（香港大学），主题 **"Generative Renaissance"**。投稿 1,106 篇，接收 201篇 conference papers 加 100 篇 journal papers，**接受率 27.2%**。
>
> **本报告范围**：收录 **24 篇** SIGGRAPH Asia 2025 物理模拟主线论文（流体/气体/多相/颗粒/固体/布料/求解器/神经场），并附「物理 × AI / World Model」交叉方向分析。
>
> **本届最重要的发现**：**CFC: Simulating Character-Fluid Coupling using a Two-Level World Model**——这是 2025–2026 三届会议中**唯一一篇把 "World Model" 写进标题的物理模拟论文**，详见第七节。
>
> **数据来源**：physicsbasedanimation.com 官方合集帖（权威清单，共 24 篇）、各论文项目主页、arXiv、ACM Digital Library、history.siggraph.org 官方摘要归档、中科大 GCL 实验室会议纪要。
>
> **可靠度约定**：正文标注 `【摘要原文】`、`【项目页描述】`、`【二手转述】`、`【推断】`。**本报告不含编造内容**，信息缺失处均已明示。

---

## 目录

- [一、总体图景：Asia 2025 的六条主线](#一总体图景asia-2025-的六条主线)
- [二、流体基础方法与涡动力学（6 篇）](#二流体基础方法与涡动力学6-篇)
- [三、多相 / 孔隙 / 颗粒（4 篇）](#三多相--孔隙--颗粒4-篇)
- [四、燃烧 / 流固耦合 / 工程应用（3 篇）](#四燃烧--流固耦合--工程应用3-篇)
- [五、求解器与数值基础设施（4 篇）](#五求解器与数值基础设施4-篇)
- [六、神经场 / 降阶/ 子空间（3 篇）](#六神经场--降阶--子空间3-篇)
- [七、World Model 核心论文：CFC 深度解析](#七world-model-核心论文cfc-深度解析)
- [八、布料 / 针织 / 形状优化（3 篇）](#八布料--针织--形状优化3-篇)
- [九、World Model × 物理模拟的会议级图景](#九world-model--物理模拟的会议级图景)
- [十、奖项、量化一览与开源清单](#十奖项量化一览与开源清单)
- [附录：信息缺口声明](#附录信息缺口声明)

---

## 一、总体图景：Asia 2025 的六条主线

**1. World Model 首次进入物理模拟论文标题。**
CFC 用**两级神经世界模型**（Fluid World Model 加 Character World Model，以外力为耦合接口）**替代** SPH 求解器，从而让强化学习能在流体环境中训练角色控制策略。这是 2025–2026 三届会议中唯一一篇标题含 world model 的物理模拟论文，也是「AI 与物理模拟」交汇最深的一次尝试——注意它是**替代**求解器而非加速求解器，与图形学社区一贯的「AI 只做辅助」姿态有本质区别。

**2. 求解器的「去线性系统化」浪潮。**
三篇论文从完全不同的路径追求同一目标：Wavelet Fluids 用不动点迭代加双投影**完全免去线性系统求解**；Implicit Position-Based Fluids 用变分能量加二阶隐式下降替代压力 Poisson 方程；Improving Curl Noise 直接构造解析无散场。这条主线与 SIGGRAPH 2025 的「更好的预条件子/多重网格」形成有趣对照——一边是把线性求解做到极致，一边是干脆绕开它。

**3. 多孔与多相是本届最密集的赛道（4 篇）。**
Porous SPH（隐式非压力力加孔隙率进压力求解）、Poro-Elasto-Capillary（饱和度感知 PPE 加 REV 表述）、Granule-In-Cell（DEM × PIC 双向耦合）、Homogenized Sand（DEM → MPM 本构提取）。共同主题是**跨尺度一致性加严格守恒**——从颗粒级/介观尺度出发，向宏观连续介质建立可验证的桥梁。

**4. 锐利界面的回归。**
HOME-FREE LBM 以 sharp interface 反攻近年主流的 diffuse interface，代价是完全忽略空气相动力学，收益是**泡沫能力加数量级性能**。这是一次明确的「用建模假设换能力」的取舍。

**5. Flow Map 从前向走向可微。**
An Adjoint Method for Differentiable Fluid Simulation on Flow Maps 抓住一个漂亮的洞察——**前向求解器与其伴随求解器共享同一条流图**，因而无需对中间数值步求导、无需存储中间变量。这是 SIGGRAPH 2025 六篇 Flow Maps 论文的自然延伸，把这个家族推向反问题领域。

**6. 仿真作为设计与控制工具。**
Fire-X（灭火工程）、UAV 混合仿真（飞控闭环设计）、RL 流固控制、PhysiOpt（生成模型的物理约束形状优化）——图形学流体与固体正大规模外溢到工程验证与具身控制。这与本届 keynote 阵容（Wayve 的合成数据、腾讯的 3D 世界生成、香港科技大学的低空经济）高度一致。

**神经方法的定位在收窄。** Neural Kinematic Bases for Fluids 明确只做**运动学基**（降维加实时加草图交互），不替代动力学求解；Precise Gradient Discontinuities 与 Force-Dual Modes 都是**降阶子空间**工作。比早期端到端学习务实得多——除 CFC 外。

---

## 二、流体基础方法与涡动力学（6 篇）

### 2.1 Viscous Vortex Dynamics on Surfaces 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Cuncheng Zhu†, Hang Yin†, Albert Chern（†共同一作）。单位据主页/补充材料托管域【推断】为 UC San Diego CSE |
| **出处** | ACM TOG 44(6)；DOI 10.1145/3763320 |
| **链接** | https://cunchengzhu.github.io/project_pages/ViscousVortex2025.html |

**核心问题**：曲面上不可压缩**黏性** Navier-Stokes 流的涡量法模拟。以往工作忽略了黏性力中的**高斯曲率依赖项**。

**方法要点**：采用涡量表述（而非速度-压力）；纳入曲率项，该项同时影响涡量方程与**Hodge 分解中调和分量**的演化。三角网格离散加 **IMEX（隐式-显式）**时间积分——黏性扩散隐式、对流显式。

**效果**：三条解析结论是本文的核心价值——跨曲率片的**涡量跳跃条件**；摩擦系数与边界曲率修正的几何对应；边界曲率对调和模态的影响。支持任意拓扑，含**不可定向曲面**；自由滑移边界下 **Kutta 条件自发涌现**。

**局限**：无公开量化性能数据；仅限曲面（2-manifold），不含体积流。

---

### 2.2 Implicit Position-Based Fluids (IPBF) 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Elie Diaz, Jerry Hsu, Eisen Montalvo-Ruiz, Chris Giles, Cem Yuksel（University of Utah，Utah Graphics Lab） |
| **出处** | SIGGRAPH Asia Conference Papers, Art. 14, 9 pp.；DOI 10.1145/3757377.3764005 |
| **链接** | https://graphics.cs.utah.edu/research/projects/ipbf/ |

**核心问题**：不可压缩性、稳定性、成本三者难以同时兼顾。

**方法要点**：不可压缩 SPH；构造专门逼近不可压条件的**变分能量**，用**二阶隐式下降**格式优化。属 position-based 而非 velocity-based 范式。

**效果**：**无条件稳定**，可承受极端大时间步；溃坝算例仅需 **2 次迭代**。对比结果清晰：IISPH/DFSPH 失稳；PBF 稳定但过压缩；SISPH 稳定但有噪声；IPBF 稳定加压缩极小。

**局限**：项目页未给帧率或密度误差的绝对数值；无 GPU 或开源信息。

---

### 2.3 Neural Kinematic Bases for Fluids 【摘要原文 + arXiv 全文】

| 项目 | 内容 |
|---|---|
| **作者** | Yibo Liu, Zhixin Fang, Sune Darkner, Noam Aigerman, Kenny Erleben, Paul Kry, Teseo Schneider（单位：哥本哈根大学/蒙特利尔大学/McGill/维多利亚大学【推断】） |
| **出处** | DOI 10.1145/3757377.3763925；arXiv:2504.15657 |

**核心问题**：无网格流体如何兼顾物理一致性与实时性。

**方法要点**：**MLP 输出一组速度场基函数** φ_k(p)，输入为评估点加域参数（圆心/半径集合 ρ）。速度 v(p) = Σφ_k(p)·α_k，**不变量与系数 α 无关**。纯物理损失（无ground truth 数据）：无散度 ∫div φ = 0、滑移边界 ∫∂Ω φ·n = 0、正交归一 ∫φ_k·φ_l = δ_kl、光滑性；用 Monte-Carlo 采样积分；边界损失用余弦相似度做**四次幂到二次幂的两阶段训练**。

**使用流程**：用户手绘流线草图 → 最小二乘拟合切向量得 α₀ → **半隐式积分**（回溯 p_o = p − v·dt，再最小二乘重投影到基空间）。

**效果**：自由度被压缩到少量基系数，实时；对训练外的圆配置泛化良好；3D 用三球体验证。

**局限**：域参数化受限（仅圆/球孔）；**纯运动学**，无真实动量方程与压力；消融显示光滑性损失、圆角处理、两阶段训练三者均不可去除。

> 第三方评述称相比 SPH/FEM 推理加速 1–2 个数量级，但**非论文原文表述**，仅供参考。

---

### 2.4 Improving Curl Noise 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | J. Andreas Bærentzen（DTU）, Jonàs Martínez（Université de Lorraine, CNRS, Inria, Loria）, Jeppe Revall Frisvad（DTU）, Sylvain Lefebvre（Inria/Loria） |
| **出处** | Art. 82, 10 pp.；DOI 10.1145/3757377.3763980 |
| **链接** | https://hal.science/hal-05308063v1 ；代码 github.com/janba/DivFree-VectorNoise |

**核心问题**：curl noise 广泛用于程序化流体外观，但流线精度受积分器精度限制。

**方法要点**：**n 维无散度矢量噪声 = n−1 个标量噪声函数梯度的 n 维叉积**，并给出任意 n 维下无散（体积保持）的证明。关键洞察：其**流线正是各标量噪声等值面的交线**，因此可将积分点**重投影**回精确流线，使**流线精度与积分器精度解耦**。

**效果**：Euler（300 步）加重投影约等于 RK4（600 步）加重投影，二者「几乎无法区分」；Blender 验证关闭重投影时流线无法闭合、开启后大量闭合。应用含图像变形、曲面纹理、隐式曲面约束噪声、各向异性 curl noise、**最高7D 点抖动**。

**局限**：程序化噪声而非 Navier-Stokes 求解器，无真实流体动力学。

---

### 2.5 An Adjoint Method for Differentiable Fluid Simulation on Flow Maps 【摘要原文/项目页】

| 项目 | 内容 |
|---|---|
| **作者** | Zhiqi Li\*, Jinjin He\*, Barnabás Börcsök, Duowen Chen, Greg Turk, Bo Zhu（Georgia Tech）；Taiyuan Zhang（Dartmouth）；Tao Du（清华大学）；Ming C. Lin（Maryland） |
| **链接** | https://pearseven.github.io/DiffFMProject/ ；arXiv:2511.01259；代码 github.com/pearseven/differentiable-flowmap |

**核心洞察**（这是本文最漂亮的地方）：**前向求解器与其伴随求解器共享同一条流图**——前向把impulse 变量从初始帧输运到当前帧以模拟涡动力学，反向把伴随变量从当前帧传回初始帧以计算梯度。

**方法要点**：在**双向流图**上直接求解伴随方程；**无需对中间数值步求导、无需存储中间变量**（这两点是传统伴随法的必需开销）；提出**长-短时间稀疏流图表示**演化伴随变量。

**效果**：**192³ 分辨率仅需 6.53 GB**；任务含 2D 形状 morphing、3D 多关键帧控制（G→R→A→P→H 字形）、从历史图像序列推断未来涡演化。

**局限**：依赖流图构造质量（梯度精度上限即流图精度）；未见与自动微分基线的耗时对比数据。

> **系列关联**：这是 SIGGRAPH 2025 六篇 Flow Maps 论文（同一团队）的直接延伸，把 Flow Map 家族从前向模拟推向反问题与可微优化。

---

### 2.6 Wavelet Fluids 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Luan Lyu, Xiaohua Ren, Wei Cao, Jian Zhu, Ziyang Ma, Enhua Wu（吴恩华）。单位：中国石油大学/澳门大学/中科院系【推断】 |
| **出处** | ACM TOG 44(6): 270:1–270:17；DOI 10.1145/3763364 |

**核心问题**：压力投影与流函数法各有**奇异性**——压力 Poisson 方程在密度趋零（气相）时病态，导致气相不可压性丧失、**气泡人为塌缩**；矢量势方程在密度趋无穷（固相）时退化，固体边界收敛差；流函数法还把线性系统维度**扩大 3 倍**。

**方法要点**：提出一种**零密度与无穷密度均良定义的新分解**；将该分解重构为**不动点迭代**，用密度无关的**无旋投影加无散投影**交替进行，**完全免去线性系统求解**；以小波变换实现两种投影。

**效果**：**同时得到压力与流函数**；统一处理自由表面与固体障碍边界；利用小波变换固有并行性做 GPU 实现，性能显著提升。

**局限**：无公开量化加速比（需查原文）。**有 2026 年勘误（Corrigendum, TOG 45(2)）**，引用时需注意。

---

## 三、多相 / 孔隙 / 颗粒（4 篇）

### 3.1 The Granule-In-Cell (GIC) Method for Simulating Sand–Water Mixtures 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Yizao Tang\*, Yuechen Zhu\*, Xingyu Ni†, Baoquan Chen†（\*共同一作，†通讯）——北京大学智能学院 / 可视计算与学习实验室（VCL） |
| **出处** | ACM TOG 44(6), Art. 265；DOI 10.1145/3763279；arXiv:2504.00745（19 页 15 图） |

**核心问题**：需在连续流体介质中捕捉单颗砂粒的随机行为——迁移、沉积、堵塞。

**方法要点**：**DEM** 捕捉颗粒级精细接触细节，**PIC** 提供连续空间表示并借粒子结构做**密度投影**。关键在于把颗粒视作**宏观输运流**而非流体的固体边界，从而实现**双向耦合**，可容纳采用不同离散方案的多种相间力，且**完全满足质量守恒方程**。

**效果**：溃坝算例中**在单一场景内**同时再现干砂、部分浸润砂、饱和砂的差异化力学性质（作者称为独有能力）；全程保持体积一致性；成功桥接介观与宏观尺度。

**局限**：arXiv 页未给粒子数、帧耗时或硬件信息；未见开源。

---

### 3.2 Implicit Incompressible Porous Flow using SPH 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Timna Böttcher, Lukas Westhofen, Stefan Rhys Jeske, Jan Bender（单位未署名；据 Bender 组【推断】为 RWTH Aachen） |
| **出处** | ACM TOG 44(6), 13 pp.；DOI 10.1145/3763325 |
| **链接** | https://srjeske.de/publications/2025-sig-asia-porous-sph/ ；代码 github.com/timnaboettcher/SPlisHSPlasH |

**核心问题**：以往 SPH 多孔流方法在粒子跨越固-流界面时**缩减粒子体积**，引发严重稳定性问题。

**方法要点**：不缩减体积、允许**相互重叠的相**；将**局部孔隙率纳入 SPH 压力求解器**，严格施加不可压缩性，使流固间压力力物理一致；内部流动改用**隐式非压力力**（以往为显式），作为**线性系统**求解并与**流体黏性加固体弹性强耦合**。

**效果**：统一捕捉**拖曳、浮力、由附着力产生的毛细作用**；扩展弹性模型使刚度随**局部饱和度**变化，从而再现**膨胀（bloating）与软化**。

**局限**：页面未给性能或迭代数数据；网页版摘要疑有截断。

---

### 3.3 Multiphase Particle-Based Simulation of Poro-Elasto-Capillary Effects 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Ruolan Li, Yanrui Xu, Yalan Zhang, Jiri Kosinka, Alexandru C. Telea, Jian Chang, Jian Jun Zhang, Xiaojuan Ban, Xiaokun Wang——北京科技大学 / 格罗宁根大学 / 乌得勒支大学 / Bournemouth University【按作者归属推断】 |
| **出处** | Art. 167, 11 pp.；DOI 10.1145/3757377.3763960 |

**核心问题**：软多孔材料中的 **PEC（孔隙-弹性-毛细）耦合**——孔隙结构演化、弹性变形、毛细压驱动润湿三者交织（如饼干吸水软化断裂、纤维海绵吸液膨胀）。已有方法把多孔介质建为静态网格或「带含水量属性的固体粒子」，缺乏对弹性、动态孔隙率、毛细相互作用的物理建模。

**方法要点**：多相粒子框架，三项贡献——物理驱动模型刻画毛细作用下的弹性与**动态孔隙结构演化**；推导**饱和度感知的压力 Poisson 方程**，在多孔介质内部与周围强制流体不可压，保证质量与动量守恒；基于**代表性体元（REV）**的表述，统一均质宏观多孔介质与含空腔嵌入结构的建模。

**效果**：与前人工作及**真实实拍素材**对比验证；支持多相流体。

**局限**：摘要未给量化性能；粒子法固有的分辨率与代价权衡。

---

### 3.4 Numerical Homogenization of Sand from Grain-level Simulations 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Yi-Lu Chen, Mickaël Ly, Chris Wojtan（ISTA，奥地利科技学院） |
| **出处** | ACM TOG 44(6), Art. 220, 23 pp.；DOI 10.1145/3763344 |
| **链接** | https://visualcomputing.ist.ac.at/publications/2025/HomogenizedSand/ |

**核心问题**：直接模拟海量刚体颗粒代价高昂；需自动从颗粒集合中提取连续介质本构参数。

**方法要点**：**周期边界条件下的数值均质化**，等效模拟无限多个相互接触的刚体；记录**有效应力-应变关系**并转换为连续介质的弹性属性与**屈服准则**；用带**改进 return mapping** 的 **MPM** 求解所得连续模型。

**效果**：用接触球集合成功**反向验证 Mohr–Coulomb 屈服面**等经典理论；推广到**非凸形状**颗粒的「异常」材料，观察到复杂 **jamming（堵塞）行为**；针对内摩擦与粘聚力**极高**的材料提出**新材料模型**。形成「刚体模拟 → 等效连续介质模拟」的完整流水线。

**关键词**：homogenization, DEM, elastoplastic, plastic flow, MPM, Drucker–Prager, Mohr–Coulomb。

**局限**：均质化假设需尺度分离，对强非均匀或瞬态场景适用性受限。

---

## 四、燃烧 / 流固耦合 / 工程应用（3 篇）

### 4.1 Kinetic Free-Surface Flows and Foams with Sharp Interfaces（HOME-FREE LBM）【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Haoxiang Wang（清华大学）, Kui Wu（Lightspeed Studios）, Hui Qiao（清华大学）, Mathieu Desbrun（Inria / École Polytechnique）, Wei Li（Lightspeed Studios & 上海交通大学） |
| **链接** | https://haoxiang-wang.com/homefree/ ；代码 github.com/qingxu-thu/Home-FSLBM |

**核心问题**：动理学多相 LBM 需同时模拟液相与空气相加**弥散界面追踪**，为捕捉小气泡分辨率代价极高且**无法做泡沫**；而已有自由表面动理学求解器行为谱系不足、中等复杂场景就常失稳。

**方法要点**：以**锐利界面（sharp interface）**重构动理学求解器，**仅模拟液相**（依据水与空气密度、黏度的巨大差异忽略空气动力学），融合单相与多相 LBM 的最新进展。

**效果**：可同时呈现湍流、**glugging（灌注咕噜声）**、气泡，以及**由表面张力互相粘连的泡沫**；气泡生长、破裂、合并快速且鲁棒；耗时仅为现有 CG 流体求解器的一小部分。

**局限**：完全忽略空气相动力学，气体压缩与气动效应无法体现；项目页未给具体加速倍数。

> 这是本届最明确的一次「用建模假设换能力」——放弃空气相换来泡沫模拟能力与数量级性能。

---

### 4.2 Fire-X: Extinguishing Fire with Stoichiometric Heat Release 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Helge Wrede¹, Anton R. Wagner¹, Sarker Miraz Mahfuz¹, Wojtek Pałubicki², Dominik L. Michels³, Sören Pirk¹——¹Kiel University（德国）、²AMU（波兰）、³KAUST（沙特） |
| **出处** | ACM TOG 44(6), 1–17；DOI 10.1145/3763338 |
| **链接** | https://helgewrede.github.io/firex/ |

**核心问题**：跨固/液/气三态的燃烧与**灭火**统一模拟。

**方法要点**：在传统流体求解器上扩展**多组分热力学与反应输运**，追踪 6 类组分：燃料、氧气、氮气、CO₂、水蒸气、残留物；燃烧反应由**化学计量相关放热（stoichiometry-dependent heat release）**控制，可区分**预混火焰与扩散火焰**的强度与组分。三项核心贡献：组分动力学与热力学反馈**紧耦合**、**蒸发建模**、用于高效灭火模拟的**混合 SPH-网格表示**。

**效果**：支持射流火、**喷雾/喷淋灭火**、燃料蒸发、缺氧或缺燃料熄灭；交互式热源与**火焰探测器**；渲染再现**层流到湍流转变**与**蓝到橙色变**；室内外场景均验证。

**局限**：摘要未给量化耗时；组分集简化为 6 类，非完整化学机理。

---

### 4.3 Fast & Stable Control of Coupled Solid-Fluid Dynamic Systems 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jie Chen（南开大学 VCIP）, Zherong Pan（LIGHTSPEED）, Bo Ren（南开大学 VCIP） |
| **出处** | Art. 39, 12 pp.；DOI 10.1145/3757377.3763997 |
| **链接** | http://ren-bo.net/ |

**核心问题**：流固耦合系统的强化学习控制不稳定，且稀疏奖励下探索代价大。

**方法要点**：四项技术组合——**twin-delayed actor-critic（TD3 类）**高效利用 off-policy 数据、加速收敛；**Boltzmann softmax 算子**降低价值函数估计偏差；新的**两步 Q 值估计器**缓解著名的**低估问题**；**FEDG（Fluid Effective Domain Guidance）**算法——将简单任务的策略与困难任务的策略**联合训练**以引导探索，缓解稀疏奖励下的过度探索需求。

**效果**：在复杂流固耦合控制基准上达 SOTA；**2D 与 3D、长时程**均稳定可靠。代表算例「双固体音乐播放器」——控制两个流体驱动器托住小球不落、击中目标琴键并按不同节拍演奏，实现长时程多目标控制。

**局限**：RL 方法固有的样本效率与跨任务泛化问题；摘要未给训练时长。

---

### 4.4 附：A Highly-Efficient Hybrid Simulation System for Flight Controller Design and Evaluation of UAVs 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jiwei Wang, Wenbin Song, Yicheng Fan, Yang Wang, Xiaopei Liu（上海科技大学 MagicLab / 信息学院） |
| **出处** | ACM TOG 44(6), Art. 198, 23 pp.；DOI 10.1145/3763283 |

**核心问题**：为特定无人机设计飞控，在**强流固交互**环境下传统经验难以套用。现有仿真器要么依赖经验模型（高效但忽略机体-气流动态交互，无法模拟窄通道内急停等强 FSI 机动），要么解算全场气流（极慢，无法支撑控制器设计迭代）。

**方法要点**：**混合建模**——远场用新提出的**自适应块（adaptive block-based）流体求解器**，机体边界附近用**参数化经验模型**，且模型参数**自动标定**。

**补充技术细节**（来自同组专利 CN122018359A，与论文高度对应）：无边界空间的高效湍流仿真；**自适应块加Laplacian 初始化**显著抑制网格块动态增删引起的**虚假波**；近场采用 **ALM/ASM**（作动线/作动面模型）免去边界层网格细化，耦合**紧支撑核**兼顾精度与稳定；**增强对流开边界**在入流出流并存工况下优于传统 Neumann 或原始对流条件；参数可**用梯度方法直接标定**。

**效果**：与**真实无人机飞行数据**对比验证；可用于**固定翼、多旋翼、混合式**无人机飞控设计，并支持**多机近距协同**与强close-proximity 效应；相比传统 CFD 效率极高，满足**实时/交互级**控制器设计迭代。

**局限**：近场为经验模型，非全解析 FSI；参数标定依赖场景。

---

## 五、求解器与数值基础设施（4 篇）

有趣的现象：本节四篇有三篇出自清华大学胡事民组或其合作者，构成本届最集中的机构信号。

### 5.1 Fast Galerkin Multigrid Method for Unstructured Meshes 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jia-Ming Lu, Tailing Yuan（独立研究者）, Zhe-Han Mo, Shi-Min Hu（胡事民），清华大学 |
| **出处** | ACM TOG 44(6), Art. 179, 16 pp.；DOI 10.1145/3763327 |
| **链接** | PDF https://cg.cs.tsinghua.edu.cn/papers/TOG-2025-Fast-Galerkin.pdf ；代码 github.com/jaimeyzzz/fgmm |

**核心问题**：多重网格理论上线性可扩展，但可变形体仿真在 GPU 上难以落地。

**方法要点**：**matrix-free 顶点块 Jacobi 平滑加 Full Approximation Scheme (FAS)**，同时支持 piecewise constant 与 linear Galerkin 两种粗化格式，避免稠密粗层矩阵。

**效果**：相对传统方法**最高 6.9× 加速**；百万顶点四面体网格约 **1 FPS**；在不同分辨率与材料刚度下均收敛稳定，极端变形与恶劣初值下仍收敛，误差更低且计算更省。Demo：百万点 Stanford dragon 跌落、百万点 bunny 布状垂坠、含大量自接触的软球。

**局限**：未找到公开的局限性表述。

---

### 5.2 A Stack-Free Parallel h-Adaptation Algorithm for Dynamically Balanced Trees on GPUs 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Lixin Ren, Xiaowei He（通讯）, Shusen Liu, Yuzhong Guo（中科院软件所 / 系统软件重点实验室 SKLCS）, Enhua Wu（软件所加澳门大学 FST） |
| **出处** | ACM TOG 44(6), Art. 180；DOI 10.1145/3763349 |
| **链接** | 代码 github.com/peridyno/peridyno |

**核心问题**：平衡树构建受 **ripple effect（涟漪效应）**的迭代本质制约，无法充分利用 GPU 并行度。

**方法要点**：把平衡树构建**重构**为「合并由若干种子点生成的 N-balanced 最小生成树（N-balanced MSTs）」；提出 **stack-free** 并行策略——用**两个 32-bit 整数寄存器作缓冲**替代整数数组栈，保证线程间负载均衡；再用 **refinement counters** 实现内部节点的并行插入与删除动态更新，避免整树重建。

**效果**：相对 SOTA [Wang et al. 2024] 达**约一个数量级**加速；支持含动态移动边界的流体仿真。

**局限**：未找到公开的局限性表述与细粒度加速表。

---

### 5.3 Reliable Iterative Dynamics (RID): A Versatile Method for Fast and Robust Simulation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jia-Ming Lu, Shi-Min Hu（通讯），清华大学 BNRist 计算机系 |
| **出处** | ACM TOG **44(3)**, Art. 29, 18 pp.（2025-05 上线，Asia 2025 journal track 宣讲）；DOI 10.1145/3734518 |
| **链接** | PDF https://cg.cs.tsinghua.edu.cn/papers/TOG-2025-Iterative-Dynamics.pdf |

**核心问题**：刚硬材料的求解困境——显式需极小时间步；隐式需大量迭代，且早期迭代产生**视觉上不可接受的中间构型**；PBD 虽能高效处理刚硬约束但受限于约束式表述，通用性差。

**方法要点**：提出 **dual descent（对偶下降）**框架的准显式（quasi-explicit）迭代求解器，**对每一次迭代的「视觉可靠性」给出理论保证**，同时对极刚系统仍保持快速稳定收敛。

**通用性**（本文最大卖点）：可无缝接入 **FEM / MPM / SPH / IPC**，覆盖弹性体、流体、碰撞处理。

**效果**：材料范围从软到「无限刚」；大时间步加极少迭代下仍视觉可靠。示例：WCSPH 流体 60 FPS，每帧 50 substeps（含表面张力与飞溅）；8 只章鱼以 5 m/s 跌落，60 FPS，每帧 100 substeps，处理复杂自碰撞与互碰。资助：NSFC 62220106003、清华-腾讯互联网创新技术联合实验室。

**局限**：未找到公开的局限性表述。

---

### 5.4 Implicit Bonded Discrete Element Method with Manifold Optimization 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jia-Ming Lu, Geng-Chen Cao, Chen-Feng Li, Shi-Min Hu（清华大学；Chen-Feng Li 为 Swansea 方向合作者） |
| **出处** | ACM TOG **44(1)**, 1–17（2025-01-28 上线，Asia 2025 宣讲）；DOI 10.1145/3711852；arXiv:2308.10459v2 |

**核心问题**：BDEM 擅长断裂建模（应力与位移不通过微分算子硬绑定），但**四元数单位长度约束极为刚硬**，绳与布还叠加剪切刚度，碰撞刚度又大幅增加 Newton 迭代次数；已有隐式积分器对 BDEM 均不奏效。

**方法要点**：为 BDEM 建立**optimization-based / variational 积分器**（允许大时间步）加**manifold optimization**（借 nullspace operator）加速四元数约束系统求解；另提出专为 BDEM 设计的元素填充（element packing）与表面重建方法。

**效果**：相较 FEM 与 MPM 的碎裂方案有**更好的尺度一致性与更真实的碰撞效果**；相较显式 BDEM 加速 **2.1–9.8×**（ACM定稿口径）。

> ⚠️ **口径差异**：arXiv v2 摘要写 "2.8 to 12 times faster than state-of-the-art"，正文示例给陶瓷盘 219K 粒子 2.4×、针织物撕裂 3.3×。本报告以 ACM 定稿的 2.1–9.8× 为主并注明此差异。

**局限**：未找到公开的局限性表述。

---

## 六、神经场/ 降阶 / 子空间（3 篇）

### 6.1 Precise Gradient Discontinuities in Neural Fields for Subspace Physics 【摘要原文，结果未公开】

| 项目 | 内容 |
|---|---|
| **作者** | Mengfei Liu\*, Yue Chang\*, Zhecheng Wang, Eitan Grinspun（University of Toronto DGP）；Peter Yichen Chen（**University of British Columbia**——注意不是 MIT） |
| **链接** | https://dgp.toronto.edu/projects/discont_grad/ |

**核心问题**：神经场天然光滑，要表达**梯度不连续**（折痕、异质材料界面）通常得把不连续位置烘焙进网络权重，界面就无法演化或编辑。

**方法要点**：在 **lifting 框架**中用**平滑截断距离函数（smoothly clamped distance function）**增广输入坐标，从而把梯度跳变编码在输入特征里而非权重里，界面因此可演化、可交互编辑。支持 discretization-agnostic 的参数化形状族仿真。可与已有 lifting 技术组合，在统一模型中**同时表达梯度不连续（折痕）与数值不连续（切割）**。

**新能力**：shape morphing、交互式折痕编辑、软-刚混合结构降阶仿真。

**信息缺口**：项目页**未提供任何数值结果，也无Limitations 章节**。资助 NSERC RGPIN-2021-03733。

> **系列关联**：这是 SIGGRAPH 2025 最佳论文荣誉提名 **Lifting the Winding Number** 的直接后续——前者处理**数值不连续**（切割），本文处理**梯度不连续**（折痕），两者可在统一模型中共存。

---

### 6.2 Force-Dual Modes: Subspace Design from Stochastic Forces 【摘要原文，结果未公开】

| 项目 | 内容 |
|---|---|
| **作者** | Otman Benchekroun, Eitan Grinspun（University of Toronto）；Maurizio Chiaramonte, Philip Allen Etter（Meta Reality Labs） |
| **链接** | https://dgp.toronto.edu/projects/force_dual_modes/ |

**核心问题**：降阶建模（ROM）中「哪个子空间对给定动力学最优」没有明确答案。

**方法要点**：对 ROM 取**统计视角**——把用户设计的**力分布**通过**线性化仿真**「推送」，得到位移上的**对偶分布**（"Force-Dual" 由此得名），再对该位移分布拟合**低秩高斯模型**得到子空间。

**理论贡献（本文最漂亮的部分）**：该框架是两类经典方法的**统一推广**——力分布为**不相关、单位方差**时退化为**线性模态分析（LMA）**子空间；力分布**低秩**时退化为**格林函数**子空间。

**适用交互**：约束惩罚、handle-based 控制、接触、肌骨驱动（teaser 为 "Baby Dragon Spring Muscles"）。

**信息缺口**：项目页**无数值结果、无 Limitations**。推断性风险（非原文）：依赖线性化推送，对强非线性大变形可能失准；子空间质量取决于力分布先验是否与真实交互匹配。

---

### 6.3 附：本届神经/降阶方向的定位观察

Asia 2025 的神经方法呈现出明确的**收窄趋势**：三篇（Neural Kinematic Bases、Precise Gradient Discontinuities、Force-Dual Modes）全部定位为**降阶表示或运动学基**，没有一篇声称替代动力学求解器。唯一的例外是 CFC——它确实用神经世界模型替代了 SPH 求解器，但代价是明确的精度损失（见第七节的RMSE 数据）。

---

## 七、World Model 核心论文：CFC 深度解析

### CFC: Simulating Character-Fluid Coupling using a Two-Level World Model 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Zhiyang Dou\*, Chen Peng\*, Xinyu Lu, Xiaohan Ye, Lixing Fang, Yuan Liu, Wenping Wang, Chuang Gan, Lingjie Liu, Taku Komura† |
| **单位** | **香港大学**（第一单位）、TransGP、UMass Amherst、香港科技大学、Texas A&M、MIT-IBM Watson AI Lab、宾夕法尼亚大学 |
| **出处** | ACM TOG 44(6), Art. 199；DOI 10.1145/3763318 |
| **链接** | https://frank-zy-dou.github.io/projects/CFC |

> ⚠️ **单位提示**：physicsbasedanimation 清单给的 people.csail.mit.edu 链接会跳转到作者主页，但**第一单位是香港大学，不是 MIT CSAIL**。MIT 侧只有 Chuang Gan（MIT-IBM Watson AI Lab）。**无 arXiv、无开源代码。**

**核心问题**：已有的physics-based character control 把环境过度简化，无法刻画身体运动与高动态环境（尤其流体）的复杂相互作用。

### 「Two-Level World Model」究竟是什么

这是本报告最需要讲准的地方。它是**两个可微神经预测器串联，以外力作为耦合接口**实现双向耦合：

| 层级 | 输入 | 输出 | 实现 |
|---|---|---|---|
| **Fluid World Model** | 流体状态 + 角色状态 | 作用于角色的外力 + 流体下一状态 | **PINN**（Lagrangian Fluid-inspired Network），损失基于**势能**构造 |
| **Character World Model** | 上一时刻角色状态 + 外力 + 策略动作 | 下一时刻角色状态 | 神经预测器 |

**两个关键点**：
1. 它**替代**了 SPH 流体求解器（原文表述为"sidestepping the computational burden of fluid simulation"），而不是加速它。这与本届其他所有神经方法（只做降阶表示）有本质区别。
2. 因网络可微，可**直接反传出 policy gradients**——这才是替代求解器的真正动机。

### 强化学习的三阶段分层设计

- **(a) 低层控制器**：在**无水**环境用 **GAIL 加 skill embeddings** 训练，得到可复用的 motion prior（基线对标 ASE/AMP）
- **(b) 高层策略**：在 CFC world model 中以**监督学习加 policy gradient** 训练
- **(c) 推理**：由高层目标驱动

### 量化数据

**运行时**（macro-step Δt = 4×10⁻² s）：

| 粒子数 | CFC FPS | SPH FPS | DiffFR FPS |
|---|---|---|---|
| 131,712 | **13.33** | 1.46 | 0.13 |
| 563,200 | **6.95** | 0.42 | — |

> ⚠️ **口径冲突需注意**：项目页文字声称 **"15–30× faster than SPH"**，但按上表逐行折算约 **7–17×**。两个口径不一致，引用时建议同时给出。

**策略训练**：directional control 高层策略约 **4.3 小时**，对比 SPH 约 **34.5 小时**（约 8×），task return 相当。

**流体精度**（相对 SPH 的粒子级 RMSE）：
- t = 4×10⁻² s：位置误差 **0.0051–0.0102 m**
- t = 2×10⁻¹ s：升至 **0.2546–0.4880 m**（约 40 倍增长）
- 高粘度（ν=0.1）误差显著小于低粘度（ν=0.01）

**动作多样性 APD**（4 秒 / 120 帧）：

| 任务 | CFC | AMP | ASE |
|---|---|---|---|
| re-locating | **99.87** | 83.43 | 71.62 |
| directional | **97.56** | 92.42 | 75.73 |

**用户研究**：均值 3.53（std 0.76），88.1% 评分 ≥3，57.1% 评 4。

**涌现行为的定量证据**：有水条件下膝与足的 Z 高度均值与方差均上升，说明策略自发学会了**自适应抬脚**。

### 能力覆盖

水中跌倒后起身、行走保持平衡、高粘度泥浆中战斗、涉水挥剑击打、双人沼泽对战、Ant 四足涉水。训练数据仅 **5K–15K 粒子加 1–3 个几何基元障碍物**。

### 局限（含推断，已标注）

项目页**无 Limitations 章节**（未找到公开的原文局限性表述）。以下为基于公开数据的**分析而非引用**：
- 长时程误差累积明显（RMSE 约 40 倍增长）
- 低粘度流体精度最差
- 训练规模 5K–15K 粒子外推到 56 万粒子存在泛化风险
- 大 Δt 下精度下降

### 为什么这篇值得单列一节

这是2025–2026 三届会议中唯一把world model 做进物理模拟论文标题的工作，而且它确实兑现了 world model 的核心承诺——**用可微的学习模型替代不可微/慢速的数值求解器，从而打通 RL 训练闭环**。它的精度数据同时也诚实地展示了这条路的代价：短时程精度可接受（毫米级），长时程误差累积到分米级。这个「能用但有边界」的定位，恰好是评估 AI 替代数值求解器这条路线现状的最佳样本。

---

## 八、布料 / 针织 / 形状优化（3 篇）

### 8.1 Neighbor-Aware Data-Driven Relaxation of Stitch Mesh Models for Knits 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Yura Hwang, Jenny Han Lin, Jerry Hsu, Benjamin Mastripolito, Cem Yuksel（University of Utah）；James McCann（CMU） |
| **出处** | Art. 77, 11 pp.；DOI 10.1145/3757377.3763890 |
| **链接** | https://graphics.cs.utah.edu/research/projects/neighbor-aware-stitch-mesh-relaxation/ |

**核心问题**：现有 mesh-level 针织仿真把针织当**均质材料**，仿真参数**依赖具体花样**——即使线圈类型相同，换个 pattern 就得重新对实物样本拟合参数。

**核心洞察**：一个线圈的物理行为不仅由自身结构决定，**也由其周围的线圈类型决定**。

**方法要点**：把 stitch mesh 模型扩展为**线圈级 neighbor-aware 材料属性**；通过线圈连接的结构分析，导出一组**有限的 four-way kernels**（每线圈考虑上下左右 **4 个邻居**），可组合表示一般 knit–purl（下针/上针）花样；设计参考花样，**实际编织并测量**，用**线性模型**反解各kernel 的 rest-length；再用这些 rest-length 做 mesh-level 松弛，并与实物样片对比验证。

**效果**：4 邻域加**仅 11 个 basis swatches** 的测量即可拟合全部 kernel。测试花样含diagonal、rib-garter-circles、garter-rib 等。

**局限**（部分由摘要限定条件推得）：仅覆盖 knit/purl，未涉及加减针、绞花、镂空；4 邻域只解释「大部分」邻域相关变形；需实物测量标定，换纱线或针号需重测；定位为快速预览与 yarn-level 仿真的初始化，非替代高精度纱线级仿真。**未找到误差数值、运行时间或与基线的定量对比表**。

---

### 8.2 A Nonconforming Formulation of Cloth 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Elias Gueidon（Reality Labs Research, Meta 加 UCLA）、Maurizio M. Chiaramonte（Reality Labs Research, Meta） |
| **出处** | Art. 78, 11 pp.；DOI 10.1145/3757377.3763989 |

> **无项目页**——这解释了 physicsbasedanimation 清单为何无链接。

**核心问题**：SOTA 布料用线性三角元加离散平均曲率建弯曲能，易出现 mesh-dependent 行为与 **locking**（高面内刚度时尤甚）；高阶格式能缓解，但 Kirchhoff–Love 要求基函数导数跨单元连续（**H² 连续**），实现代价高。

**方法要点**：提出仅需 **H¹ 连续**位移场的连续介质格式——采用**非协调（non-conforming）函数空间**，通过精心推导的**界面项弱施加切基（tangent basis）连续性**；本质是把 **Interior Penalty（内部罚）方法**适配到曲面仿真（在 Engel et al. 2002 基础上发展）。使用**标准 Lagrange 基函数**，可直接升高多项式阶数，且**保留布料界通行的面内/面外能量解耦范式**——作者论证布料由纱线构成，弯曲与厚度压缩的关系不像薄壳那样直接，因此不强行统一为薄壳连续介质。

**工程亮点**：jump/average 项在单元边上求值，而现有 FEM 布料代码本来就遍历边计算二面角，**改造成本极低**；**无需 NURBS 或细分 patch 构造**，不必重写数据结构。

**实现**：基于 **FEniCS 加 PETSc**，半隐式时间积分加 Newton-Raphson（带回溯线搜索）；硬件 Intel i9-10980XE @3.00GHz / 32GB RAM（**CPU 实现，非 GPU**）。参考文献含 CLO 3D v2024.2.214，疑作对比基线。

**局限**：未找到公开的局限性表述；从实现看为 CPU 加 FEniCS 研究原型，性能数据未在可公开抓取部分给出。

---

### 8.3 PhysiOpt: Physics-Driven Shape Optimization for 3D Generative Models 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Xiao Sean Zhan\*, Clément Jambon\*, Evan Thompson, Mina Konaković Luković（MIT）；Kenney Ng（MIT-IBM Watson AI Lab） |
| **出处** | Art. 109, 11 pp.；DOI 10.1145/3757377.3763884 |
| **链接** | https://physiopt.github.io ；代码 github.com/physiopt/physiopt |

**核心问题**：3D 生成模型输出常缺乏**物理完整性**；生成模型用连续隐式场，物理优化用 FEM，需 ad hoc 网格提取，且慢，无法嵌入快速迭代的生成式设计流程。

**方法要点**：可微物理优化器，通过一个**易实现的可微映射**直接在生成模型的**隐空间**里优化形状。循环为：隐参数 π → 解码为隐式场 φ(·,π) → 体素化为**稀疏的密度加权有限元** → **线性静力学分析**求位移 → 物理损失 J(π) → 端到端反传更新 π。支持用户指定材料、载荷、边界条件。

**兼容性**：全局隐模型（DeepSDF）、part-based 隐模型、大规模 SOTA 生成器（**TRELLIS**）。支持迭代设计：生成 → 加载优化 → inpainting 改几何 → 再优化。

**效果**：相对基于网格的 **DiffIPC** 顶点扰动式优化，产生**语义一致**的形变且**收敛更快**；结果忠于所学先验、可制造。实物验证：未优化的章鱼椅腿部塌到地面、火烈鸟高脚杯极轻载即倾倒；优化版本行为改善。被 MIT News 报道。

**局限**：项目页无 Limitations，且**未给任何数值**（无 compliance 降幅、迭代数、单步耗时）。推断性风险（非原文）：仅线性静力学，不覆盖大变形、非线性材料、动载；表达力受生成先验约束；精度受体素分辨率限制。

---

## 九、World Model × 物理模拟的会议级图景

必须严格区分：**厂商keynote/发布 ≠ Course/Lab ≠ 同行评审技术论文**。

### 9.1 论文层面

**唯一直接命中**：**CFC**（见第七节）。24篇清单中其余偏 AI 的三篇均是**神经表示/降阶**而非 world model：Neural Kinematic Bases for Fluids、Precise Gradient Discontinuities、Force-Dual Modes。

**Session 层面的确证信号**：SIGGRAPH Asia 2025 设有 Technical Papers session **"Neural & Implicit Representations for Geometry and Physics"**（Session Chair：刘利刚），主题即神经隐式表示在几何建模与物理模拟中的进展。CFC 所在 session 为 **"Physically Based Simulation & Dynamic Environments"**；多篇求解器论文集中在 **"High-Performance Simulation Algorithms"**。

### 9.2 Keynote 层面（非论文）

| Keynote | 讲者 | 与 world model 的关系 |
|---|---|---|
| **"Worlds We Make: The Rise of Synthetic Data in AI"** | Jamie Shotton（Wayve 首席科学家） | **本届最贴近 world model 叙事的keynote**——合成数据如何重塑自动驾驶与具身 AI |
| **"Crafting 3D Worlds for Intelligent Vision"** | 郭春超（腾讯 Hunyuan 3D 负责人） | 生成/重建/交互式 3D 环境，明确指向世界模拟与数字孪生 |
| 低空经济（LAE）与 SILAS 智能低空系统 | 沈向洋（香港科技大学） | 具身智能的应用侧 |

**Asiagraphics "Intelligent Graphics" Workshop**（刘利刚为组织主席之一）：9 位特邀报告，主题明确涵盖 **AIGC、World Models、具身智能、智能制造**。属 workshop 层面而非 technical paper。

### 9.3 NVIDIA 在 SIGGRAPH Asia 2025

**Featured Session**（非 keynote）：12/17 13:30–14:30，Theatre 2，**Shalini De Mello**，题为 **"From Pixels to Physics: How Advancements in Graphics are Accelerating Physical AI"**，主线是「Generative Renaissance 推动图形学向physical AI 的范式转变」。

**NVIDIA Research 论文清单（官方页共 21 项）**中与物理/角色控制相关的：
- **MaskedManipulator: Versatile Whole-Body Manipulation**（有 paper、project page、GitHub）
- **Physics-Based Motion Imitation With Adversarial Differential Discriminators**
- **Offset Geometric Contact**（本报告按 SIGGRAPH 2025 北美收录，存在归属交叉）
- **StableMotion: Training Motion Cleanup Models With Unpaired Corrupted Data**
- **Simulating 3D Thermal Fluid Dynamics in Data Centers With Soft-Constrained Physics-Informed Graph Neural Network** —— **PINN 加 GNN 的 learned simulator**，是本届 physics × AI 方向除 CFC 外最值得注意的一篇
- Practical Gaussian Process Implicit Surfaces With Sparse Convolutions
- Design for Descent: What Makes a Shape Grammar Easy to Optimize?

> ⚠️ NVIDIA 页面把 SIGGRAPH 与 SIGGRAPH Asia 的成果混列，个别条目未必是 Asia 2025 论文，引用时需逐条核实。

**Emerging Tech / Lab 层面**：Play 4D（FVV流式传输）、PITAR（LLM 驱动 XR agent）、NESI（神经显式高度场交集几何表示）；Hands-on Labs 含 **"Advancing World Simulation With 3D Gaussian Splatting"** 与 **"Synthetic Data Generation for Robot Learning Pipelines With NVIDIA Cosmos"**——这两项是培训层面的 world-model 内容，**不是论文**。

### 9.4 Physics-based character control 与 RL

| 论文 | 状态 | 要点 |
|---|---|---|
| **CFC** | 确证 | GAIL 加 skill embeddings 低层，world model 内高层策略（见第七节） |
| **Control Operators for Interactive Character Animation** | **确证，🏆 Best Paper Award** | Ruiyu Gou（University of British Columbia）。以语义化「控制算子」框架简化 ML 驱动角色控制系统的设计与训练。**这是本届角色控制方向最高规格的成果** |
| **MaskedManipulator: Versatile Whole-Body Manipulation** | 确证（NVIDIA 清单） | 含 paper/project/GitHub 三链 |
| **Physics-Based Motion Imitation With Adversarial Differential Discriminators** | 确证（NVIDIA 清单） | 附 paper 链接 |
| **StableMotion** | 确证（NVIDIA 清单） | 属 motion cleanup，与 RL 控制相邻但非控制策略 |

### 9.5 Differentiable simulation 的 course / workshop

**未找到**任何以 differentiable simulation 或 neural physics 命名的 SIGGRAPH Asia 2025 Course。已确证存在的相邻活动仅为 Asiagraphics "Intelligent Graphics" Workshop 与 NVIDIA 的 hands-on labs。

相关论文层面有 Best Paper **《Automatic Sampling for Discontinuities in Differentiable Shaders》**（Yash Belhe, UC San Diego）——属可微**渲染**而非可微**仿真**，勿混淆。

### 9.6 与 SIGGRAPH 2025 / 2026 的纵向对比

| 会议 | World model 在论文轨 | AI × 物理的主要路径 |
|---|---|---|
| **SIGGRAPH 2025** | 完全缺席 | 神经场作降阶基（Wind Lifter）、生成先验加可微物理（Dress-1-to-3） |
| **SIGGRAPH Asia 2025** | **CFC 一篇（标题命中）** | 神经运动学基、梯度不连续神经场、统计式子空间；加 CFC 的神经世界模型替代求解器 |
| **SIGGRAPH 2026** | 完全缺席 | 仿真就绪资产生成（SimArt）、物理感知运动控制（ReActor、MUSIC）；world model 全在 keynote/产品层 |

一个值得注意的观察：**world model 在物理模拟论文轨的出现是「一次性的」**——Asia 2025 有 CFC，前后两届的北美会议都没有对应工作。这说明该方向尚未形成持续的论文流，仍处于个别团队探索阶段。

---

## 十、奖项、量化一览与开源清单

### 10.1 奖项

| 奖项 | 论文 | 备注 |
|---|---|---|
| 🏆 **Best Paper Award** | Control Operators for Interactive Character Animation | Ruiyu Gou（UBC），角色控制方向 |
| 🏆 **Best Paper Award** | Automatic Sampling for Discontinuities in Differentiable Shaders | Yash Belhe（UCSD），可微渲染方向 |

> 未检索到本届物理模拟（流体/固体）方向的 Best Paper 或 Honorable Mention 名单。

### 10.2 量化一览

| 论文 | 关键数据 |
|---|---|
| Fast Galerkin Multigrid | **6.9×** 加速；百万顶点四面体约 **1 FPS** |
| Stack-Free Parallel h-Adaptation | 相对 SOTA 约**一个数量级**加速 |
| Implicit BDEM | **2.1–9.8×**（vs 显式 BDEM；arXiv 口径为 2.8–12×） |
| CFC | FPS 13.33（13.2万粒子）/ 6.95（56.3 万粒子）；训练 4.3 h vs SPH 34.5 h |
| Adjoint Flow Maps | 192³ 分辨率仅 **6.53 GB** |
| IPBF | 溃坝算例仅需 **2 次迭代** |
| Improving Curl Noise | Euler(300步)+重投影 ≈ RK4(600步)+重投影 |
| Reliable Iterative Dynamics | WCSPH 60 FPS / 50 substeps；8 章鱼 60 FPS / 100 substeps |
| Neighbor-Aware Stitch Relaxation | 仅 **11 个 basis swatches** 即可拟合全部 kernel |

### 10.3 开源项目清单

| 论文 | 仓库 |
|---|---|
| Improving Curl Noise | github.com/janba/DivFree-VectorNoise |
| Adjoint Method for Differentiable Flow Maps | github.com/pearseven/differentiable-flowmap |
| HOME-FREE LBM | github.com/qingxu-thu/Home-FSLBM |
| Implicit Incompressible Porous Flow | github.com/timnaboettcher/SPlisHSPlasH |
| Fast Galerkin Multigrid | github.com/jaimeyzzz/fgmm |
| Stack-Free Parallel h-Adaptation | github.com/peridyno/peridyno |
| PhysiOpt | github.com/physiopt/physiopt |
| MaskedManipulator | 见 NVIDIA 论文页 |

---

## 附录：信息缺口声明

**未取得量化数据或 limitations 原文的论文**：
- **CFC**：无 Limitations 章节；且项目页文字声称的 15–30× 与 FPS 表折算的 7–17× **口径冲突**
- **Precise Gradient Discontinuities**、**Force-Dual Modes**、**Neighbor-Aware Stitch Relaxation**、**PhysiOpt**：量化结果与局限性均未在可公开抓取页面出现，需 ACM 全文
- **Viscous Vortex Dynamics**、**IPBF**、**Wavelet Fluids**、**HOME-FREE LBM**、**GIC**、**Porous SPH**、**PEC**、**Homogenized Sand**、**Fire-X**、**Fast & Stable Control**、**UAV Hybrid Simulation**、**Fast Galerkin MG**、**Stack-Free h-Adaptation**、**RID**、**Implicit BDEM**、**A Nonconforming Formulation of Cloth**：均未找到公开的局限性表述

**作者机构未经证实（标注为推断）**：
- Viscous Vortex Dynamics（据主页域推断 UC San Diego）
- Neural Kinematic Bases（哥本哈根/蒙特利尔/McGill/维多利亚）
- Wavelet Fluids（中国石油大学/澳门大学/中科院系）
- Implicit Incompressible Porous Flow（据 Bender 组推断 RWTH Aachen）
- Poro-Elasto-Capillary（按作者归属推断）

**归属存疑或口径冲突的论文**：
- **Reliable Iterative Dynamics**：TOG 44(3)，2025 年 5 月已上线，Asia 2025 为 journal track 宣讲
- **Implicit BDEM**：TOG 44(1)，2025 年 1 月已上线；加速比 ACM 定稿 2.1–9.8× 与 arXiv 2.8–12× 不一致
- **Wavelet Fluids**：有 2026 年勘误（Corrigendum, TOG 45(2)）
- **Offset Geometric Contact**、**Neurally Integrated Finite Elements**：同时出现在 NVIDIA 的 SIGGRAPH Asia 2025 清单与 SIGGRAPH 2025 北美清单中，本报告按后者收录
- **CFC 第一单位**：香港大学，非 physicsbasedanimation 链接指向所暗示的 MIT CSAIL

**其他**：
- NVIDIA 官方页混列 SIGGRAPH 与 SIGGRAPH Asia 成果，个别条目归属需逐条核实
- 检索环境下 sa2025.conference-schedule.org TLS 受限无法直连，但可经搜索引擎快照读取；history.siggraph.org 按 session 收录官方摘要，是本报告的重要补充来源

---

*报告生成时间：2026 年 8 月 10 日*
