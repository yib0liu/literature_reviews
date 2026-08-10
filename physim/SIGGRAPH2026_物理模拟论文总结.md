# SIGGRAPH 2026 物理模拟论文逐篇总结

> **会议信息**：SIGGRAPH 2026（第 53 届），2026 年 7 月 19–23 日，美国洛杉矶会议中心。参会 9,500+ 人、来自 69 个国家。Technical Papers Chair：Mirela Ben-Chen。
>
> **本报告范围**：共收录 **35 篇** SIGGRAPH 2026 物理模拟主线论文（流体/气体/固体/弹性体/布料/接触/求解器/降阶模型），另附 **World Model × 物理** 交叉方向 8 篇 +会议级信号，以及 SCA 2026 关联清单。
>
> **数据来源**：physicsbasedanimation.com 官方合集帖（权威清单）、各论文项目主页、arXiv、ACM Digital Library、SIGGRAPH 官方新闻稿。
>
> **可靠度约定**：正文标注 `【摘要原文】`（作者主页/arXiv/ACM DL 原文）、`【项目页描述】`、`【二手转述】`、`【推断】`。**本报告不含编造内容**，信息缺失处均已明示。

---

## 目录

- [一、总体图景：本届物理模拟的六条主线](#一总体图景本届物理模拟的六条主线)
- [二、流体 / 气体 / 多相耦合（8 篇）](#二流体--气体--多相耦合8-篇)
- [三、MPM 与弹塑性（2 篇）](#三mpm-与弹塑性2-篇)
- [四、刚体 / 仿射体 / 多体动力学（3 篇）](#四刚体--仿射体--多体动力学3-篇)
- [五、布料 / 薄壳 / 纱线（3 篇）](#五布料--薄壳--纱线3-篇)
- [六、接触与无穿透框架（2 篇）](#六接触与无穿透框架2-篇)
- [七、IPC 求解器基础设施与符号化框架（3 篇）](#七ipc-求解器基础设施与符号化框架3-篇)
- [八、碰撞检测与几何鲁棒性（3 篇）](#八碰撞检测与几何鲁棒性3-篇)
- [九、降阶模型 / LOD / 神经降阶（3 篇）](#九降阶模型--lod--神经降阶3-篇)
- [十、数值求解器与 GPU 基础设施（4 篇）](#十数值求解器与-gpu-基础设施4-篇)
- [十一、材料建模/ 外观 / 毛发（3 篇）](#十一材料建模--外观--毛发3-篇)
- [十二、World Model × 物理模拟交叉方向](#十二world-model--物理模拟交叉方向)
- [十三、奖项、趋势与开源清单](#十三奖项趋势与开源清单)
- [附录 A：SCA 2026 物理模拟论文（勿与 SIGGRAPH 混淆）](#附录-asca-2026-物理模拟论文勿与-siggraph-混淆)
- [附录 B：信息缺口声明](#附录-b信息缺口声明)

---

## 一、总体图景：本届物理模拟的六条主线

本届 Technical Papers 投稿量超 1,120 篇，为 53 年来最高。通读 35 篇物理模拟论文后，可归纳出六条清晰主线：

**1. 把计算瓶颈"解析化"，而非单纯堆算力。**
FastVEM 用无扩散提升算子的Galerkin 多重网格把压力投影加速 100×；Tube Maps 发现 SPH 边界密度积分完全由最近点局部几何决定，用管状坐标把三维数值积分降维为**一维闭式表达式**，开销降低 1–3 个数量级；Mixwell 更彻底——从圆柱搅拌齿周围的理想势流推出解析笔刷，直接取消了时间积分与中间重采样。

**2. 突破时间步的经典约束。**
ST-FLIP（Honorable Mention）把粒子视为**四维时空采样点**，沿时间轴随机化位置并用可分离4D 核做蒙特卡洛估计，从而绕开 CFL 限制，时间步比传统求解器大一个数量级。Mixed MPM 则通过速度–应力混合离散，使刚硬弹粘塑性材料能以 CFL 速率（而非受刚度限制的极小步长）推进。

**3. 摆脱 IPC 对数障碍函数的病态。**
这是本届最集中的技术共识。Barrier-free Elastodynamics 用增广拉格朗日替代障碍函数，条件数比IPC 低约**两个数量级**、最高 103× 加速；DAT 从另一方向切入，用互斥区域的空间截断结构性保证无穿透。两条路径殊途同归。

**4. 多相/多材料从"算子分裂"走向"统一整体式表述"。**
Buoyancy MPM（统一压力 + 分相速度网格）、LBM-MPM（统一连续介质 + 保水模型）、Nonlocal Monolithic（peridynamics 变分统一）三篇不约而同追求用整体式框架取代人为解耦。

**5. 基础设施的符号化与自动化。**
SymX 与 YASPS 让研究者只写单个代表性单元的符号能量表达式，系统自动完成微分、稀疏结构推断与 GPU kernel 生成，取代了手工推导一二阶导数的传统苦工。Locality-Aware AD 则把自动微分完全约束在 GPU 寄存器与共享内存内，全面超越 PyTorch/JAX/Warp/DrJIT。

**6. 鲁棒性标准从"经验调参"升级为"浮点级可证保证"。**
参数曲面 CCD 首次为参数曲面建立浮点鲁棒框架，并用有理数算术反推ground truth 构造基准；High-Order Continuous Geometrical Validity 给出即使用浮点实现仍保持保守性的高阶单元有效性判定。

值得单独指出的是，本届"物理模拟"主赛道**依然是经典数值方法，而非神经网络替代**。35 篇中明确的 learned simulator 屈指可数，JGS2-GQ 甚至是反向信号——主动把前作的数据驱动 Cubature 替换为免训练的高斯求积。AI 的真实切入点在**资产生成**与**运动控制**，详见第十二节。

---

## 二、流体 / 气体 / 多相耦合（8 篇）

### 2.1 Fast VEM Fluid Simulation 【摘要原文】

|项目 | 内容 |
|---|---|
| **作者** | Runze Zhang, Bo Ren（南开大学） |
| **出处** | ACM TOG 45(4), Article 65, 18 pp., DOI 10.1145/3811315 |
| **链接** | http://ren-bo.net/papers/zrz_tog2026.pdf |

**核心问题**：复杂边界的流体模拟需要 cut-cell 构造贴体网格，但由此产生的病态线性系统让压力投影阶段的代价高到无法承受。

**方法要点**：
- 用**二阶虚拟单元法（VEM）**离散压力，在不规则贴体网格上稳健施加不可压条件与边界条件
- VEM 多项式空间上的 PIC 对流格式
- **保凸 cut-cell**（Binary Space Cut-Cell）构造"求解器友好"的网格
- 核心创新是Galerkin 几何多重网格中的**无扩散提升算子（diffusion-free prolongation）**：仅用局部边/面投影算子 Π∇，把粗层矩阵的非零元限制在同一多面体单元内，从根本上避免粗层矩阵稠密化
- 嵌套的边界感知网格层次保证粗层自由度良置；GPU 上采用 Chebyshev 平滑器

**效果**：压力投影**最高加速 100×**（对比现有 cut-cell 求解器）；128³ 网格 + 无人机扫描的复杂几何，二阶 VEM 烟雾模拟 **<1 分钟/帧**；可处理相对厚度低至 **1e-5** 的超薄片边界与窄缝；嵌入几何 Chamfer 误差 1e-5~1e-6。

**局限**：VEM 网格质量保证是经验性指标而非定理；三角面片归属采用均匀采样 + 多数射线内外判定的启发式，对自交或开放网格无形式化保证。

---

### 2.2 Spatiotemporal FLIP (ST-FLIP)🏅Honorable Mention 【摘要原文】

| 项目 | 内容 |
|---|---|
| **全称** | Spatiotemporal FLIP for Fast Free-Surface and Two-Phase Simulation With Very Large Time Steps |
| **作者** | Bernhard Braun, Rene Winchenbach, Nils Thuerey（慕尼黑工业大学 TUM）；Jan Bender（RWTH Aachen） |
| **出处** | ACM TOG 45(4), Article 76, 20 pp., DOI 10.1145/3811289 |
| **链接** | https://ge.in.tum.de/ ; https://www.vci.rwth-aachen.de/publications/ |

**核心问题**：混合粒子–网格液体求解器在大时间步下，粒子运动的**时间欠采样**会在压力投影后产生走样驱动的自由表面伪影，因此长期受CFL 条件约束。

**方法要点**：把粒子视为**四维时空中的采样点**——除了常规的空间抖动，还沿**时间轴随机化粒子位置**，用**可分离 4D 核**做 P2G沉积，从而得到"每步时间片（time-slab）积分量"的**蒙特卡洛估计器**。虽然概念上是 4D，但在投影时坍缩为 slab 积分的 3D 网格场，因此实现上是一个轻量插件。方法复用 P2G 权重累加器作为**时空相场**，提供变系数投影权重，从而**免除每步的表面重建**。可直接接入现有 FLIP / PIC / APIC 求解器，每步额外开销可忽略。

**效果**：时间步比CFL 受限求解器**大一个数量级**；在单台工作站上完成**数十亿（multi-billion）粒子**、高有效三维分辨率的模拟，取得**数倍加速**，同时保留细致流动结构。

**产业意义**（引自官方新闻稿，TUM 的 Bernhard Braun）："对于电影或视效团队，这意味着更快的高分辨率预览、更多的创作迭代，以及能更从容地纳入制作排期的最终模拟。我们的方法可以消除高质量液体模拟的主要计算瓶颈之一。"

**关联**：前作为 SIGGRAPH 2025 的 Adaptive Phase-Field-FLIP。

---

### 2.3 Buoyancy-driven Phase Separation in the Material Point Method 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Mehrnaz Ayazi, Craig Schroeder, Tamar Shinar（加州大学河滨分校 UC Riverside） |
| **出处** | SIGGRAPH 2026 Conference Track, 83:1–11, DOI 10.1145/3799902.3811069 |
| **链接** | https://dl.acm.org/doi/10.1145/3799902.3811069 |

**核心问题**：MPM 使用单一背景网格时，不同材料的粒子从共享网格插值速度，导致**无法有效分离**——油和水一旦混合就再也分不开（如 Rayleigh-Taylor 不稳定性中出现持久性混合）。而改用完全分离的网格虽能分离，却丢失了材料之间通过网格产生的自然相互作用。

**方法要点**：关键洞察是**压缩与压力并不依赖于粒子类型**——局部邻域的压缩只是因为粒子靠得太近。因此：从**统一的速度场**更新变形梯度并计算**统一的压力力**，同时**速度使用各相独立的背景网格**。这样在弱可压 MPM 中，浮力可以自然驱动相分离，即使从完全混合的初始状态出发。此外提出**混合势（mixing potential）**，使得在无重力条件下也能由热力学驱动分离；并给出为多相粒子流体渲染生成**一致的逐相 level set** 的新算法。

**效果**：Rayleigh-Taylor 算例中相界面锐利分离，明显优于 Li & Liu [2024] 的残留混合结果。

---

### 2.4 Volume-Preserving LBM-MPM Coupling for Air-Water-Sand Mixtures 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Xiaoyu Xiao, Xiaokang Yang, Wei Li（上海交通大学）；Haoxiang Wang（清华大学自动化系）；Mathieu Desbrun（Inria / École Polytechnique） |
| **出处** | ACM TOG 45(4), Article 77 |
| **链接** | https://lwkobe.github.io/papers/XWYDL2026/ |

**核心问题**：颗粒材料与多相流体之间的动态、多尺度相互作用，其视觉复杂性来源于强耦合的小尺度结构，在计算上极具挑战。

**方法要点**：用**格子玻尔兹曼法（LBM）**求解弱可压两相流体（空气 + 水），用 **MPM** 求解颗粒沙，二者建立在**统一的连续介质表述**上（两相流体与颗粒介质的控制方程写在同一框架内）。引入**保水模型（water retention model）**刻画液体如何渗入并滞留于颗粒结构中，从而捕捉沙从干燥摩擦主导态到浸润黏性态的转变。并显式**强制混合物内流体的体积守恒**，以保证数值稳定性与物理真实性。

**效果**：可跨**大范围密度比**模拟沙的起动、输运、沉降与侵蚀。代表性算例包括沙坝溃决（breaching of sand-walled basins）、含沙浑浊流、沙结构的侵蚀性垮塌。**已开源**。

---

### 2.5 A Nonlocal Monolithic Variational Framework for Free Surface Flows 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Shusen Liu, Yuzhong Guo, Lixin Ren, Ying Qiao, Xiaowei He（中国科学院软件研究所） |
| **出处** | SIGGRAPH 2026 Conference Track |
| **链接** | https://peridynamics.com/publications/2026-Liu-NUV.pdf |

> 注：peridynamics.com 上的 PDF 标题为 "A Nonlocal **Unified** Variational Framework"，为投稿版本的命名差异。

**核心问题**：自由表面流需要同时处理不可压性、黏性与表面张力；现有粒子法大多依赖**算子分裂**，这会引入耦合伪影并限制稳定性。

**方法要点**：提出统一的**非线性优化框架**，实现三者的**整体式（monolithic）强耦合**。借助**近场动力学（peridynamics）**，把不同流体机制的离散化置于一致的变分原理之下；将流体运动重述为**关于粒子位置的非线性变分优化问题**，用**半隐式逐次代换法**求解。黏性方面摒弃传统的基于速度的Laplacian，改为把黏性耗散重构为**非局部增量势**，并将速度–位置的运动学关系直接嵌入势中，得到**基于位置的黏性模型**；沿键分解**法向变形 ε_ij 与切向变形 Δ_ij**，从而**分别处理体积黏性 λ 与剪切黏性 μ**（论文有 λ=100, μ=0 对比 λ=99, μ=1 的对照实验）。体积能受 Neo-Hookean 启发，数值上等价于 SISPH [He et al. 2025]。

**效果**：作者称这是**首个**能完整解析不可压、黏性、表面张力三者相互依赖关系的粒子法统一求解器，在复杂场景下稳定性显著提升。**代码开源**于 https://github.com/peridyno/peridyno

---

### 2.6 Stochastic Geomorphological Transport for Terrain Erosion Simulation 🏅 Honorable Mention 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Nicholas McDonald, Guillaume Cordonnier（Inria GraphDeco / Université Côte d'Azur） |
| **出处** | DOI 10.1145/3811336 |
| **链接** | https://inria.hal.science/ ; 代码 https://github.com/erosiv/geotransport |

**核心问题**：山地地形在地质时间尺度上演化，涉及水、沉积物、岩崩等输运量的复杂交互。核心难点是**输运过程与侵蚀过程的时间尺度相差数个数量级**，却需要同时模拟。

**方法要点**：提出新颖的**并行、随机、基于粒子**的方法，可在地质时间尺度上模拟输运，核心是一个**广义随机积分过程**（CUDA kernel 实现）。关键突破在于**放松了前人对速度的强假设**（如 Stream Power Law 河流功率定律），从而建立起基于**更一般形式动量守恒**的新侵蚀模型。

**效果**：数值上准确求解底层守恒律，避免了前人方法常见的伪影；能捕捉多尺度地貌特征，生成连贯的流域盆地结构，以及**辫状河、曲流（meanders）、三角洲**等动态现象；可与构造运动、风向等其他地质过程组合（论文讨论了沙丘、海岸侵蚀、洪水、岩崩等扩展）。

**局限**：依赖"侵蚀慢、输运快"的时间尺度分离假设——若侵蚀很快（如单次岩崩）或输运很慢（如冰川），该假设失效，精度下降。

**开源**：`pip install geotransport`

---

### 2.7 Mixwell: Sharp 2D Fluid Brushes for Progressive Physics-Based Mixing 🏆 Best Paper 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Doug L. James（斯坦福大学）, Ethan James |
| **出处** | ACM TOG (SIGGRAPH 2026) |
| **链接** | https://dougjam.github.io/ ; https://graphics.stanford.edu/~djames/publications |

**核心问题**：数字化的"搅拌 / 混色"效果，若走网格或粒子重采样路线，会因中间重采样带来数值耗散（画面模糊），且分辨率相关、无法真正做到渐进式。

**方法要点**：从**圆柱形搅拌齿（tine）周围的理想势流**推导出一族解析的2D 流体笔刷，其中包含一个**带尖点（cusped）的 Kelvinlet 风格正则化速度笔刷**，简洁地刻画了圆柱–流体交互。在 **Maxwell 1869 年的漂移（drift）理论**基础上，为无限笔画与有限笔画分别开发 GPU 友好的粒子漂移评估策略。为图像工作流引入**圆柱形反向漂移函数（Reverse-Drift Functions, RDF）**——编码搅拌齿插入、移动、拔出的位移场。RDF 的妙处在于它像几何建模中的 SDF 一样可**自然复合、在 shader 中链式串联**，从而建模复杂操作而完全避免中间纹理模糊。另外给出利用周期性的 RDF 复合方案。所有操作**逐样本独立求值**：无全局求解、无网格、无中间重采样。

**效果**：实时 **GLSL / HLSL** 实现，并已集成进 **Houdini 生产环境**（OpenCL, OSL）；实现渐进式、**任意分辨率**的混合与渲染，**数值耗散可忽略**。

---

### 2.8 Tube Maps: Fast SPH Boundary Handling with Tubular Coordinates 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Daria Nogina, Silvia Sellán（哥伦比亚大学，Geometry and the City lab） |
| **出处** | SIGGRAPH 2026 Conference Track, DOI 10.1145/3799902.3811118 |
| **链接** | https://gatc.cs.columbia.edu/projects/tubemaps.html |

**核心问题**：SPH 的流固耦合中，粒子化边界策略会引入**非确定性的离散误差**，而隐式（SDF 积分）方法虽然精确，但数值积分代价昂贵。

**方法要点**：Tube Maps 是 SPH 边界密度计算的**即插即用替代品**。关键观察是：边界密度积分**完全由流体粒子最近点附近的局部曲面几何决定**。把该局部几何用**管状坐标（tubular coordinates）**表达之后，原本的**三维积分退化为一维闭式表达式**，可在**常数时间**内求值，彻底消除数值求积。

**效果**：精度可比隐式方法，但边界处理开销降低 **1–3 个数量级**；支持**时变曲面固体**（含预设运动序列）。

---

## 三、MPM 与弹塑性（2 篇）

### 3.1 Mixed Material Point Methods for Stiff Elastoplasticity 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Gilles Daviet（**独作**），NVIDIA，法国 Annecy |
| **出处** | ACM TOG 45(4), Article 151, 19 pp., DOI 10.1145/3811345 |
| **链接** | https://research.nvidia.com/labs/prl/mixed_mpm |

**核心问题**：如何以 **CFL 速率**（而非受刚度限制的极小步长）仿真**刚硬的弹–粘–塑性**材料，直至不可压极限。

**方法要点**：
- 在 Daviet & Bertails-Descoubes [2016a] 的**混合（mixed，速度–应力双场）离散**基础上扩展，支持**有限应变粘弹性**与更一般的**流动法则（flow rules）**
- 隐式积分导出**良态、对称的优化问题**，且**模板紧凑（compact stencils）**，配套高效 GPU 求解器
- 系统探索多种**速度–应力离散配对**的性能/精度权衡，发现**三线性速度**常常具有竞争力（以扭转弹性梁为例证）
- 支持与刚体求解器**双向耦合**，并作为 **NVIDIA Newton 物理引擎的第一方模块**集成

**效果**：**4,900 万粒子**的沙立方体倾泻到模型城市，**单GPU 4 秒/帧**；材料覆盖颗粒物、雪到弹性固体；雪崩中弹塑性雪层的裂纹传播；混凝土 Armadillo 在工业压机下破碎；原生支持不可压流体；双向耦合下机器人会随地形调整步态。

> **产业关联**：这与 NVIDIA 官方博客所述"为 Newton 物理引擎带来雪/沙/弹性固体新求解器"高度对应（博客未点名论文标题，此对应关系为【推断】）。

---

### 3.2 MPM Lite: Linear Kernels and Integration without Particles 【摘要原文 + 项目页详述】

| 项目 | 内容 |
|---|---|
| **作者** | Xiang Feng, Yunuo Chen, Chang Yu（**同等贡献**）, Demetri Terzopoulos, Chenfanfu Jiang（UCLA）；Hao Su（UCSD）；Yin Yang（Utah）；Joe Masterjohn, Alejandro Castro（**丰田研究院 TRI**） |
| **出处** | ACM TOG 45(4), Article 152, 20 pp.；arXiv:2602.07853 |
| **链接** | https://mpmlite.github.io/ |

**核心问题**：标准 MPM 的性能瓶颈——由于采用**粒子求积**与**宽模板核**（如二次 B 样条），隐式求解代价与 **PPC（每格粒子数）成正比**。

**方法要点**：
- 粒子**只作为运动状态与材料历史的载体**；把背景笛卡尔网格视作**体素六面体网格**，用紧凑的**线性（多线性）核**把粒子状态重采样到**固定位置的求积点**（单元中心）
- 三阶段流程 **Unload → Integrate → Load**：Unload 阶段除质量/动量/速度梯度外，还把**外延 Kirchhoff 应力 τ** 传到单元中心；运动学传输沿用 **APIC**；中心→节点的权重为常量 1/2^d，可实现**无竞争的节点 gather**
- 关键创新是**避免变形梯度的非物理平均**：不做隐式应力（会导致非对称 Hessian），而是从 τ 反解**只含主拉伸的无旋参考** F_base = U diag(σ) Uᵀ（丢弃极分解的旋转部分，因为各向同性弹性能仅依赖 F 的奇异值），常见各向同性材料有**闭式解**；由此自然支持基于增量势能的优化式时间积分
- 架构上成为「重采样单元 + FEM 风格积分模块」，可**直接复用现成的非线性求解器（PCG、VBD）、预条件子与明确的边界条件处理**
- **理论误差分析**：在 Lipschitz 光滑假设下，与经典二次 B 样条 APIC 的速度及速度梯度差均为 **O(Δx²)**；无旋参考相对于保留旋转的每步速度偏差为 O(Δt²)

**效果**：相比隐式 MPM **15.9×** 加速，相比显式 MPM **1.88×** 加速；求解器复杂度与内存占用**与粒子数解耦**（仅取决于网格分辨率）；对六面体单点积分常见的**沙漏模式（hourglass）免疫**（该模式在传到单元中心时被滤除）；线动量/角动量守恒良好；材料覆盖弹塑性面条、脆性糖果、雪、沙水混合、金属、粘塑性奶油。

**局限（论文明列）**：无旋拉伸参考**依赖各向同性**，各向异性材料需额外处理；更复杂的超弹/非弹模型的应力→拉伸反解可能需要新推导或迭代；**单点单元中心积分**在弯曲主导、薄结构、应力剧变区精度下降；整体成本仍随粒子数缩放（对流/重采样/本构更新部分）；当前假设单元内**单一速度场**（多速度混合、更丰富的耦合与接触、非笛卡尔/自适应网格留待未来）。

---

## 四、刚体 / 仿射体 / 多体动力学（3 篇）

### 4.1 M-ABD: Scalable, Efficient, and Robust Multi-Affine-Body Dynamics 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Zhiyong He, Dewen Guo（Utah）、Minghao Guo, Wojciech Matusik（MIT CSAIL）、Yili Zhao（USC）、Hao Su（UCSD）、Chenfanfu Jiang（UCLA）、Peter Yichen Chen（UBC/NYU）、Yin Yang（Utah） |
| **出处** | ACM TOG 45(4), Article 99, 19 pp., DOI 10.1145/3811276 |
| **链接** | https://minsuglly.github.io/mabd |

**核心问题**：大规模铰接装配体（articulated assemblies）仿真的数值刚性与几何复杂度。传统刚体求解器因**旋转参数化**引入强非线性，多体双向耦合时问题更严重。

**方法要点**：
- 利用 **ABD 的线性运动学映射**（每个体 12 自由度仿射描述）
- 关键洞察：ABD 面向近刚体，不同材料的本构差异可忽略 → 用 **co-rotational（共旋）公式**把系统的几何非线性隔离出来 → **系统矩阵为常量，可全程预分解**，即使采用**全隐式积分**
- 将 primal 体坐标映射到由**最小关节自由度**定义的紧凑**对偶空间**，求解相应 **KKT 系统**，实现关节约束的**精确**满足与物理正确的运动传播
- 针对不同关节拓扑（链、树、闭环、不规则网络）提供**专用求解器套件**

**效果**：**单 CPU 核**上数十万刚体达到交互帧率；大时间步下稳定性优异。代表性算例：**1,076,748 连杆**的巨型滑轮系统，h=10⁻² s、**每时间步仅 1 次迭代**，单线程 **904 ms/帧**。

---

### 4.2 Heterogeneous Subspace Corrections for GPU Deformable Multibody Dynamics 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Dewen Guo（北大→Utah）、Zhendong Wang、Minchen Li（CMU）、Sheng Li、Guoping Wang（北京大学）、Huamin Wang（Style3D）、Chenfanfu Jiang（UCLA）、Yin Yang（Utah） |
| **出处** | ACM TOG 45(4), Article 100, 16 pp., DOI 10.1145/3811275 |
| **链接** | https://graphics.cs.cmu.edu/?page_id=3512 |

**核心问题**：**可变形 + 刚硬构件混合**的异质多体系统在 GPU 上难以求解。Newton–Krylov 方法虽然 matrix-free、GPU 友好，但遇到**刚性接触**与**材料刚度差异悬殊**时**病态严重、收敛极慢**。

**方法要点**（HSC，一种 Newton-CG 变体）：
- **解耦为两个 Krylov 迭代**：可变形体走 Newton-CG；仿射（近刚体）子系统走 **Neumann 级数迭代**
- 利用耦合矩阵的**低秩性**，提出 **GPU 版自适应交叉近似（ACA）**算法，大幅削减GPU 上的 SpMV 开销
- 为仿射子系统设计**专用数据结构**，并行快速装配**接触 Hessian**

**效果**：一致优于现有 GPU 仿真器；即便高分辨率模型 + 大量刚性接触，**收敛率可比拟直接求解器**。（摘要未给出具体加速倍数）

---

### 4.3 Distributed Affine Body Dynamics with Adaptive Consensus 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Jiafeng Liu, Wenhui Zhou, Xinming Pei, Lei Lan, Weiwei Xu（浙江大学 CAD&CG 国家重点实验室）、Yifan Peng（香港大学）、Huamin Wang（Style3D Research）、Yin Yang（Utah） |
| **出处** | SIGGRAPH 2026 Conference Papers, 11 pp., DOI 10.1145/3799902.3811106；arXiv:2605.15875 |
| **链接** | https://arxiv.org/abs/2605.15875 |

**核心问题**：IPC 框架下的 ABD 能对极刚固体给出**严格无穿透**保证，这对**具身智能 / 接触丰富的交互**至关重要（微小穿透即会定性改变交互结果）；但 IPC 的**全局耦合 barrier 约束**阻碍了跨多 GPU / 多计算节点的可扩展执行。

**方法要点**：基于**共识型 ADMM（consensus-based ADMM）**的分布式 ABD 表述——各计算节点**并行**求解本地 ABD 子问题，随后通过一步**全局共识**强制共享边界体的状态一致性；标题中的 **adaptive consensus** 指自适应的惩罚/权重调整。为防止跨分区同步时产生穿透，提出**保持可行性的共识机制**：将局部有效更新与经过**连续碰撞检测（CCD）认证**的全局共识步结合，仅在状态通过 CCD 检查时才应用同步。分布式执行下保持 **IPC 级鲁棒性**与全局一致性。

**效果**：收敛稳定、无穿透、多节点大场景高效扩展。代表性算例：**5,000 个"宝可梦" ABD 刚体（共 600 万三角面）**落入窄漏斗，采用双 worker 分布式，下腔有两个反向旋转的穿孔转盘持续造成堆叠碰撞，**全程无穿透**。入选 Technical Papers Trailer。

---

## 五、布料 / 薄壳 / 纱线（3 篇）

### 5.1 Better Bending 【摘要原文 + 官方新闻稿】

| 项目 | 内容 |
|---|---|
| **全称** | Better Bending: Analysis, Construction and Verification of Discrete Bending Models for Kirchhoff-Love Shells |
| **作者** | Zhen Chen, Danny M. Kaufman（**Adobe Research**）、Etienne Vouga（**德克萨斯大学奥斯汀分校**）。新闻稿标注 "All authors contributed equally" |
| **出处** | ACM TOG (SIGGRAPH 2026), DOI 10.1145/3811373 |
| **链接** | https://zhenchen-jay.github.io/publication/betterbending/ ; 代码 github.com/evouga/better-bending |

**核心问题**：图形学与力学界研究薄壳已逾四十年，但"**如何最好地离散化弯曲行为**"这一问题在很大程度上仍未被系统回答（原文："little consensus on how to best discretize them"）。

**方法要点**：系统评测图形学中 **10 个最主流的离散弯曲模型**，外加作者自提的新变体，构成一个"leaderboard"式基准。三层验证体系：
1. 解析基准测试收敛性与网格依赖性
2. **sharp-creasing（锐折痕）压力基准**——本文新设计的基准
3. 对基准领先者在大规模平衡态与动力学问题上测试完整求解行为与相对计算成本

针对暴露出的建模缺口，构造出新的能量模型 **Bending-Active Cosserat (BAC)**，并给出配套公式与算法工具，以及有基准支撑的实用选型建议。

**关键发现**（这是本文最有价值的部分）：
-结果与团队自己的预期相反——**理论上精确的模型在生产级高分辨率网格上反而崩坏**。Kaufman原话："这促使我们重新思考什么才是合适的验证基准，并推动我们设计了sharp-creasing 基准。反过来，正是这个基准带来了构造新BAC 弯曲模型所需的洞察。"
- 另一个发现关乎微妙的实现选择。Chen 原话："我们的实验揭示，**基于顶点的二次弯曲公式（QB-PL）展现出显著更好的收敛行为**。这是我事先没有预料到的，它说明即使对于成熟的弯曲能量，离散化的选择也极为重要。"
- Vouga 坦言："把一篇既很长、又与常见套路不同的论文投给 SIGGRAPH，我确实有些忐忑。但我认为这类论文对这个领域的健康是重要的。"

**适用材料谱**：布料、纸张、橡胶到成型金属薄板。

**信息缺口**：摘要与新闻稿未公布 10 个模型的具体名单与定量表格，需查 PDF 正文。

---

### 5.2 Efficient B-Spline Finite Elements for Cloth Simulation 【摘要原文 + 项目页】

| 项目 | 内容 |
|---|---|
| **作者** | Yuqi Meng, Yihao Shi, Kemeng Huang, Zixuan Lu, Ning Guo, Taku Komura, Yin Yang, Minchen Li（CMU / Utah / 浙大 / 港大 / Genesis AI） |
| **出处** | ACM TOG 45(4), Article 102, DOI 10.1145/3811278；arXiv:2506.18867 |
| **链接** | https://simulation-intelligence.github.io/BS-Cloth/ |

**核心问题**：高阶 FEM 精度更高，但因为开销大、视觉收益不明显，长期未被布料仿真采纳。

**方法要点**：采用**二次 B 样条**基函数 → 得到全局 **C¹ 连续位移场**，使膜能量与弯曲能量可以一致离散，抑制 locking 与网格依赖性。三项性能优化：
1. 对膜能量与弯曲能量**分别优化求积规则的缩减积分**
2. 针对样条结构定制的 **Hessian加速装配**
3. 基于 **partial factorization** 的优化线性求解器

接触处理用 IPC；网格贴合流水线同时支持矩形域与不规则服装域。

**效果**：**相对线性 FEM 平均 2× 加速**，同时精度、褶皱细节、鲁棒性均有提升。对比基线包括 SFEM (Liang 2023)、BHEM (Ni 2024)；基准场景为悬挂/垂坠/剪切；Δt=0.01s，i7-12700F。压力测试包括高速旋转球、直升机斜撞、Armadillo 密集条带、篮筐内多件服装堆积。**代码开源**。

---

### 5.3 Interactive Yarn-level Knitwear with Nested Douglas-Rachford Splitting 【官方日程描述】

| 项目 | 内容 |
|---|---|
| **作者** | Chun Yuan, Zixuan Lu, Haoyang Shi, Dewen Guo（Utah）、Huamin Wang（Style3D Research）、Chenfanfu Jiang（UCLA）、Zherong Pan, Kui Wu（LIGHTSPEED）、Yin Yang（Utah） |
| **出处** | ACM TOG (SIGGRAPH 2026)，入选 Technical Papers Trailer |

**核心问题**：由于长纱线相互套结产生的复杂运动学特性，在纱线级别模拟针织物极其困难。传统布料求解器把织物当作连续的弹性薄片，无法捕捉真实针脚的物理与几何细节。

**方法要点**：利用**广义 Douglas-Rachford Splitting (DRS)** 算法与**层次化分解**技术（即标题中的 nested）实现高分辨率针织物仿真；通过对复杂纱线动力学进行**凸化（convexifying）**处理，保证鲁棒且最优的收敛性；实现为**matrix-free 的 GPU 并行**。

**效果**：**百万级自由度**的服装达到交互式仿真帧率。

**信息缺口**：无 arXiv 或项目主页，未找到公开的局限性说明。

---

## 六、接触与无穿透框架（2 篇）

### 6.1 Robust and Efficient Penetration-Free Elastodynamics without Barriers 【摘要原文 + 项目页量化数据】

| 项目 | 内容 |
|---|---|
| **作者** | Juntian Zheng, Zhaofeng Luo（CMU）、Minchen Li（CMU / Genesis AI） |
| **出处** | ACM TOG 2026, DOI 10.1145/3811035；arXiv:2512.12151 |
| **链接** | https://simulation-intelligence.github.io/barrier-free/ |

> 本篇是全部35 篇中量化数据最丰富的一篇。

**核心问题**：IPC 的两大效率瓶颈——① 对数障碍函数导致系统**病态**，拖慢迭代线性求解器；② **TOI locking**，在密集碰撞场景下限制 active-set 探索，需要大量 Newton 迭代。

**方法要点**：提出**无障碍函数**的二阶约束优化框架 + 定制的**增广拉格朗日求解器**：
- 把 CCD 检出的所有必需接触对**立即纳入**，避免 TOI 锁死
- **自适应更新拉格朗日乘子而非增大罚刚度**，防止在 TOI=0 处停滞并保持系统良态
- **约束过滤与衰减机制**保持 active set 紧凑稳定
- 基于累积 TOI 的终止判据，证明有限步终止性与一阶时间积分精度
- GPU 优化实现（libuipc）

**效果（量化）**：
| 对比项 | 结果 |
|---|---|
| vs GIPC | 最高 **103×** 加速 |
| Teaser 挤压球 | 2.61M DoF / 2.25M 四面体 / 峰值 1.45M 活跃接触，**98.5× 加速，5.37 s/帧** |
| Newton 迭代数 | vs GIPC平均**减少 4.24×**（103,946 接触帧） |
| 条件数 | 比 IPC **小约 2 个数量级**，PCG 平均仅 63.1 次迭代 |
| vs Cubic Barrier [Ando 2024] | 最高 **84.4×** |
| vs OGC [Chen 2025] | 快 **4×**，且 OGC 出现 locking 伪影 |
| 大接触半径 | 本法活跃接触稳定在 ~4.3k；IPC 超 10⁷ 接触并 OOM |
| 极端测试 | 100 m/s 小猪撞板保持无穿透；Δt 细化验证一阶收敛 |

---

### 6.2 Divide and Truncate (DAT) 【摘要原文】

| 项目 | 内容 |
|---|---|
| **全称** | Divide and Truncate: A Penetration and Inversion Free Framework for Coupled Multi-physics Systems |
| **作者** | Anka He Chen, Jerry Hsu, Youssef Ayman, Miles Macklin |
| **出处** | SIGGRAPH 2026 Conference Proceedings；arXiv:2604.15513 |
| **链接** | https://ankachan.github.io/publication/2026-dat |

**核心问题**：多物理耦合系统（刚体 / 体积软体 / 薄壳 / 杆 / 动画物体）中，如何同时保证无穿透与无单元反转。

**方法要点**：将环境空间划分为**互斥区域（exclusive regions）**，把位移**截断**在各自区域内，从而在结构上保证无穿透。**Planar-DAT** 变体只限制**朝向邻近表面方向**的运动分量，切向保持自由，从而解决前人方法的**人为阻尼（artificial damping）与死锁（deadlock）**问题。方法**材料无关**（各物体无需知道对方的物理属性）、**求解器无关**（可作为后处理步骤接入任意迭代优化器）。

**信息缺口**：摘要仅定性表述，**无量化数据**（需查 PDF 的 Results 章节）。论文署名机构未在摘要页给出；仅知第一作者现为 NVIDIA Research Scientist（Utah 博士），Macklin 亦在 NVIDIA，但**机构归属未经证实**。

---

## 七、IPC 求解器基础设施与符号化框架（3 篇）

### 7.1 AGIPC: Adaptive In-Solve Algebraic Coarsening for GPU IPC 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Xuan Wang（香港大学）、Zhaofeng Luo（CMU）、Minchen Li（CMU / Genesis AI）、Taku Komura（HKU）、Kemeng Huang（HKU） |
| **出处** | SIGGRAPH Conference Papers '26, DOI 10.1145/3799902.3811199；arXiv:2605.04773 |

**核心问题**：隐式积分的成本被反复求解大型线性系统所主导（IPC 中随 DoF 呈 O(n²)）。已有的两条自适应路线都有硬伤：**显式 remeshing** 改变连通性与顶点顺序，破坏并行实现与内存局部性、破坏 IPC 障碍能量的 C² 连续性，且可能引入局部自交；**自适应子空间法**虽避免拓扑改变，但基构造/更新访存不规则且产生稠密系统矩阵，限制 GPU 效率。

**方法要点**：提出**代数式 in-solve 粗化**——在 Newton 求解内部动态降低 DoF，**无需任何显式拓扑修改**。从自细网格出发，把自适应表达为由 **per-edge tags** 控制的**选择性边折叠**；可折叠边用 **warp 级哈希映射**并行聚合为粗"**super-nodes**"，受保护边保留局部细节；由此定义隐式的粗网格，通过 GPU reduction kernel 映射并归约细尺度梯度/Hessian 来**代数装配**粗系统；用 **PCG** 求解后 prolongate 回细网格。与 IPC 障碍能量无缝集成。

**效果**：相对 SOTA GPU IPC 求解器**最高 3× 加速**，结果视觉上不可区分。

---

### 7.2 YASPS: A Symbolic Framework for Extensible, High-Performance IPC Simulation 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Xuan Tang, Kemeng Huang, Gilbert Bernstein, Minchen Li, Tzu-Mao Li |
| **链接** | https://github.com/txstc55/yasps |

**核心问题**：高性能 IPC 实现普遍**硬编码**能量、图元类型与参数化假设。新增能量或参数化时需手工推导一二阶导数、手写全局 Hessian/梯度装配逻辑，混合参数化更会引发代码组合爆炸——可扩展性成为主要障碍。

**方法要点**：引入两个**关系算子JOIN 与 UNION**，在**符号计算图**中直接编码连通性与异构参数化；对这些算子做**符号微分**自动推导局部导数，并从同一份描述推断全局梯度/Hessian 的**稀疏性与块结构**，从而避免混合参数化导致的代码爆炸。符号图被编译为 GPU kernel（局部能量求值、导数、块稀疏装配），Newton 系统用 GPU 迭代求解器求解。

**效果**：性能与 SOTA IPC 实现相当，而新增能量/参数化仅需局部的符号定义。

---

### 7.3 SymX: Energy-based Simulation from Symbolic Expressions 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | José Antonio Fernández-Fernández, Fabian Löschner, Lukas Westhofen, Andreas Longva, Jan Bender（RWTH Aachen，Computer Animation / VCI） |
| **出处** | ACM TOG 45(1), Article 5, 19 pp., DOI 10.1145/3764928 |
| **链接** | https://animation.rwth-aachen.de/publication/0596/ ; https://github.com/InteractiveComputerGraphics/SymX |

> ⚠️ **归属提示**：这是 TOG 第 45 卷第 **1** 期（issue date 2026 年 2 月），并非 45(4)的 SIGGRAPH 会议卷。physicsbasedanimation 将其归入 SIGGRAPH 2026 合集，实际归属建议以官方日程为准。

**核心问题**：优化型时间积分器（Newton 类）需要全局非线性目标函数的一、二阶导数。手工推导、实现、测试与维护这些导数极其耗时且易错，严重阻碍模型的快速迭代。

**方法要点**：开源框架。用户只需为**某一离散化下的单个代表性单元**写出能量的符号表达式；系统自动完成符号微分 → 生成优化代码 → JIT 编译 → 全局装配（含投影到正定、线性求解、line search）。

**效果**：导数性能与成熟符号引擎 SymPy **持平**；相比另一 SOTA 集成方案 TinyAD，仿真快**至少一个数量级**。已支持高阶 FEM、刚体、自适应离散化、摩擦接触、多物理耦合。

**局限**：Windows 上 JIT 编译显著慢于 Linux/macOS（源自官方 README）。

---

## 八、碰撞检测与几何鲁棒性（3 篇）

### 8.1 Floating-Point Robustness in Parametric Surface Continuous Collision Detection 【摘要原文】

| 项目 | 内容 |
|---|---|
| **全称** | Floating-Point Robustness in Parametric Surface Continuous Collision Detection: From Algorithm to Benchmarking |
| **作者** | Xuwen Chen\*, Junyu Wang\*（共同一作）, Cheng Yu, Xingyu Ni, Meng Zhang, Bin Wang, Mengyu Chu, Baoquan Chen |
| **出处** | ACM TOG (SIGGRAPH 2026), DOI 10.1145/3811310 |
| **链接** | https://xw-c.github.io/publication/sig26/ |

**核心问题**：三角网格的鲁棒 CCD 已相对成熟，但**参数曲面**因表示复杂性与更高的算法敏感性，其浮点鲁棒性仍是开放难题。

**方法要点**：提出**首个**面向参数曲面的浮点鲁棒 CCD 框架，构建于作者前作 **TDIBM**（Time-Dependent Inclusion-Based Method, SIGGRAPH Asia 2024）之上。提出新的**误差分解策略**，将**系数误差与算术误差分离**，从而支持结构化分析与安全性保证。基准构造思路很巧妙——**反转 CCD 流程**：从预设的碰撞结果出发，用**有理数算术**反推生成精确的 ground truth，覆盖典型与**近退化**情形，再用该基准深入评测多个 CCD 算法。

**信息缺口**：页面未给出量化的误报/漏报率或耗时数据。代码与数据集"将于接收后发布"。一作邮箱含 pku 前缀，疑为北京大学（【推断】）。

---

### 8.2 High-Order Continuous Geometrical Validity 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Federico Sichetti（热那亚大学）、Zizhou Huang（NYU）、Marco Attene（CNR-IMATI Genova）、Denis Zorin（NYU）、Enrico Puppo（热那亚大学）、Daniele Panozzo（NYU） |
| **链接** | https://arxiv.org/abs/2406.03756 ; https://cims.nyu.edu/gcl/papers/2025-Positive-Jacobian.pdf ; 代码 gitlab.com/fsichetti/hocgv |

> ⚠️ **归属需注意**：physicsbasedanimation 将其列入 SIGGRAPH 2026 清单，但 NYU GCL 官方页与 PDF 均标注 **TOG 2025**（arXiv:2406.03756，2024 年 6 月）。可能是 TOG 论文在 SIGGRAPH 2026 排期呈现。GCL 页面上唯一明确标注 SIGGRAPH 2026 的是 *Surface chamfering for robust tetrahedral meshing*。

**核心问题**：判定任意多项式阶数的单纯形（三角/四面体）、张量积（四边形/六面体）与混合（棱柱）单元，在沿**分段线性轨迹变形的整个时间区间内**是否保持几何有效——而非仅在离散时间步上采样检查。

**方法要点**：**保守算法**，结合**自适应 Bézier 细化**与**二分搜索**，判定单元多项式几何映射的 **Jacobian 行列式**是否/何时/何处变负。关键性质是：用**浮点算术实现时仍保持保守性保证**，且成本仅略高于现有非保守方法，可作为 drop-in 替换。

**效果**：在**高阶 IPC 弹性动力学模拟器**中验证——扭转梁例子在不用本方法时，t=3s / 7s 分别出现 1 / 161 个无效单元；扭转环在 t=0.4s / 1s 出现 11 / 31 个。启用本方法 + 自适应求积细化后**零无效单元**，且**无需手工调参**。

---

### 8.3 Untangling Surfaces via Shape and Mesh Repulsion 【摘要原文 + 项目页】

| 项目 | 内容 |
|---|---|
| **作者** | Jiří Minarčík\*（CMU / Resistant AI）, Michael Liu\*（CMU）, Keenan Crane（CMU / **Roblox Research**）, Minchen Li（CMU / **Genesis AI**） |
| **出处** | ACM TOG 45(4), Article 163, 15 pp., DOI 10.1145/3811382 |
| **链接** | https://www.cs.cmu.edu/~minchenl/ |

**核心问题**：自交在曲面网格中普遍存在，会让下游的**仿真、制造、学习**流水线失效。既有方法把自交当作局部碰撞事件处理，但"嵌入性"本质上是**全局几何性质**，无法只靠局部推理来保证。

**方法要点**：提出能量框架，**同时在形状级与网格级**施加嵌入性。
- **形状级**：高斯自接触能量 E_G =∬ (w/ε^p)·exp(−‖x−y‖²/ε²)（曲面取 p=2），耦合那些"空间上近、内蕴距离上远"的区域，且与离散化无关
- **网格级**：用 Minkowski 差的原点包含判据（A∩B≠∅ ⟺0∈A−B）构造罚能量

**效果**：**不改变网格连通性**即可消除自交。Half-Kite Lattice 算例：67,506 顶点 / 135,392 三角形 / 初始 240 个真实交叉对 → **最终剩余 0**。覆盖带边界曲面、非流形、浸入失败（Whitney umbrella）、多物体与含约束场景；在 ChaLearn 服装、ShapeNetCore v2 等数据集上优于 SOTA。

**局限**：不做重网格化或拓扑修复，因此无法处理必须改变连通性才能修复的缺陷；带宽 ε 是需要调整的超参数；部分输入需人工预处理；承诺的 Intersection-Free Mesh Dataset 尚未发布。

---

## 九、降阶模型 / LOD / 神经降阶（3 篇）

### 9.1 Low-Rank Koopman Deformables with Log-Linear Time Integration 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Yue Chang（多伦多大学 DGP）、Peter Yichen Chen、Eitan Grinspun（U Toronto）、Maurizio M. Chiaramonte（Meta，【推断】） |
| **出处** | arXiv:2602.07687（v2, 2026-06-14），SIGGRAPH NA 2026 |
| **链接** | https://arxiv.org/abs/2602.07687 |

**核心问题**：可变形体的子空间仿真仍需**逐步顺序时间积分**，在控制、初值估计等**目标函数主要依赖最终构型**的优化任务中代价高昂；且以往 DMD 类降阶模型**只能绑定单一形状与单一离散化**。

**方法要点**：
- 用**动态模态分解（DMD）参数化 Koopman 算子**的**低秩**表述，学习可变形动力学的时间演化，用**矩阵求值**直接预测未来状态以取代顺序积分 → 时间步数上呈 **log-linear（≈O(n log n)）** 缩放，可**跳过轨迹中大段时间步**而保持精度
- 提出**离散化无关（discretization-agnostic）**扩展，跨**多形状 + 多网格分辨率**学习共享动力学→ 使此前对 DMD 不可行的**快速形状优化**成为可能

**效果**：log-linear 时间缩放是摘要中唯一明确的量化结论；支持控制、初值估计、形状优化等下游任务。**具体加速倍数与误差指标需查 PDF 正文**。作者单位为【推断】，arXiv 摘要页未列。

---

### 9.2 Boundary-aware Neural Model Reduction for PDEs 【摘要原文 + 项目页】

| 项目 | 内容 |
|---|---|
| **作者** | Li Liao\*、Pengfei Shen\*（同等贡献）、Yifan Peng（香港大学 WeLight Lab / ECE） |
| **出处** | SIGGRAPH 2026 Technical Papers, 12 pp., DOI 10.1145/3799902.3811153 |
| **链接** | https://liliao20.github.io/projects/boundary-aware-neural-model-reduction/ |

> 这是 35 篇合集中最贴近"neural surrogate"的一篇。

**核心问题**：偏微分算子的**特征分析**是降阶仿真的核心，但**神经形状空间特征分析**此前基本局限于**自然 Neumann 边界条件**，导致无法直接控制**支撑、开口、换热边界**等**会改变底层算子**的边界效应。

**方法要点**：
- 把 Laplace 类算子的神经特征分析扩展到 **Dirichlet / Robin / 混合边界**，并把**边界位置与 Robin 系数作为模型输入**，构成**联合形状–边界空间**，使特征函数与频谱随几何**和**边界配置连续变化
- 三类边界统一在**同一变分神经特征分析框架**内：**距离感知提升（distance-aware lifting）**用距离场强制满足 Dirichlet 约束（约束**内建**于降阶空间，而非用惩罚项对抗），Neumann/Robin 项经**变分能量**引入；距离场梯度的"切换曲线"不连续性被显式暴露给网络
- **单一神经模型**即可学习跨形状、跨边界条件的降阶基，边界参数可被搜索、优化、跨物理算例复用

**效果**（三类应用）：
1. **腔体共振匹配**——优化滑动的压力释放开口以逼近目标频谱；移动 Dirichlet 补丁引发平滑而显著的低频频谱变化
2. **变支撑下的降阶弹性仿真**——Dirichlet 感知基在受约束边界平滑趋零，仿真始终位于**可行子空间**内，**仅 12 个基**即与ground truth 接近，优于 soft/stiff barrier + Neumann 基
3. **瞬态热分析**——散热器单胞在 Dirichlet/Neumann/Robin 混合边界下的特征基，Robin 系数控制对流换热强度

代码 Coming Soon。

---

### 9.3 Progressing Level-of-Detail Animation for Volumetric Elastodynamics 【项目页/官方日程描述】

| 项目 | 内容 |
|---|---|
| **作者** | Jiayi Eris Zhang, Doug L. James（斯坦福大学）、Danny M. Kaufman（Adobe） |
| **链接** | https://graphics.stanford.edu/~djames/publications |

**核心思路**：把 **Progressive Dynamics** 框架 [Zhang et al. 2024, 2025]从**布料与壳**推广到**体（volumetric）有限元**，构建 LOD 动画设计流水线——用**具预测性的粗分辨率预览**支持快速迭代，最终收敛到高分辨率的体弹性动力学动画（粗层预览"预示"细层结果，保证跨分辨率、跨时间的一致性）。

**信息缺口**：Stanford 与作者主页的 abstract 栏均为空（"Details coming soon"），**未找到公开的定量数据与局限说明**。标题在不同页面有"for" / "of" 两种写法，TOG 官方为 *for*。

---

## 十、数值求解器与 GPU 基础设施（4 篇）

### 10.1 JGS2-GQ: Training-free 2nd Jacobi with Gaussian Quadrature 【项目页/日程描述 + ACM 摘要片段】

| 项目 | 内容 |
|---|---|
| **作者** | Dewen Guo, Zixuan Lu, Zhiyong He, Yuqi Meng（Utah）, Bohan Wang（新加坡国立大学）, Lei Lan, Weiwei Xu（浙大 CAD&CG 国重）, Chenfanfu Jiang（UCLA）, Yin Yang（Utah） |
| **出处** | ACM TOG 45(4), Article 123, 18 pp., DOI 10.1145/3811274 |
| **链接** | http://lanlei.github.io/ |

**核心问题**：JGS2 通过给每个子问题添加"扰动子空间"来预测局部求解的全局影响，从而避免 Jacobi 迭代的过冲，但其效率依赖 **Cubature（数据驱动体积积分）**的昂贵离线预计算/训练，且对未见过的形变泛化能力差。

**方法要点**：用**免训练的高斯求积**替代数据驱动的 Cubature。

**效果**：稳定的高分辨率 GPU 仿真；支持摩擦接触；保持类Newton 的收敛性；在**新形变**上比 JGS2 快 **50%**，且完全无需任何训练。

> 这是本届一个值得注意的**反向信号**：主动把前作中的数据驱动组件替换为经典数值方法。

---

### 10.2 MeshFEM: A Block-accelerated Solver for Nonlinear Finite Elements 【未找到公开摘要】

| 项目 | 内容 |
|---|---|
| **作者** | Haleh Mohammadian, Xinzhuo Hu, Roi Poranne, Julian Panetta（UC Davis / Haifa） |
| **出处** | ACM TOG 45(4), Article 121, 22 pp., DOI 10.1145/3811386；报告时段 7/22 15:45, Room 403 B |
| **链接** | https://github.com/MeshFEM/MeshFEM |

**已确证信息**：C++ 非线性 FEM 库，支持线性/二次三角形与四面体单元，多维度、多基函数、多单元类型（shell / solid / 参数化 / hinge）、多数值类型（含自动微分类型）。核心新增件是 **BlockCatamari 分块 Cholesky** 代码（独立于 MeshFEMSparse 仓库），依赖 CHOLMOD/UMFPACK；提供 Python 绑定。

**效果**：作者 LinkedIn 帖标题称"**14× faster**"【社媒描述，非论文摘要】。

**信息缺口**：未找到官方摘要原文与论文内的量化数据。

---

### 10.3 Fast Sparse Matrix Permutation for Mesh-Based Direct Solvers 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Behrooz Zarebavani\*, Ahmed H. Mahmoud\*（共同一作）, Ana Dodik, Changcheng Yuan, Serban D. Porumbescu, John D. Owens, Maryam Mehri Dehnavi, Justin Solomon（MIT / UC Davis / Toronto 交叉团队） |
| **出处** | arXiv:2602.00898；DOI 10.1145/3799902.3811189 |
| **链接** | 代码 https://github.com/BehroozZare/fast-permute |

**核心问题**：三角网格产生的线性系统在稀疏 Cholesky 分解中，**置换（permutation）本身**的运行时开销成为瓶颈。

**方法要点**：产生 nested-dissection 风格的置换，但**故意放弃严格的平衡性与分离子最优性**，以换取快速划分与高效的消去树构建；把置换分解为 patch 级局部排序 + 分离子的紧凑商图排序。

**效果**：集成进 CPU 与 GPU 上厂商维护的稀疏 Cholesky 求解器，跨多种图形学应用（单次分解与重复分解场景），稀疏 Cholesky 求解性能最高提升 **6.27×**。

---

### 10.4 Locality-Aware Automatic Differentiation on the GPU for Mesh-Based Computations 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Ahmed H. Mahmoud, Rahul Goel, Jonathan Ragan-Kelley, Justin Solomon（MIT + UC Davis Owens Group） |
| **出处** | ACM TOG 45(4), July 2026, DOI 10.1145/3811338；arXiv:2509.00406v3 |
| **链接** | 代码 https://github.com/owensgroup/RXMesh |

**核心问题**：通用 AD 框架（PyTorch / JAX 等）会建立全局计算图、分配中间缓冲、产生 host-device 同步，这与网格计算固有的**局部性与稀疏性**严重不匹配。

**方法要点**：逐单元 forward-mode AD，全部计算**限制在寄存器与共享内存**内，直接在 GPU 上装配全局梯度、稀疏 Jacobian 与稀疏 Hessian；不建立全局计算图；支持静态与**动态变化的稀疏模式**（交互驱动）、标量与向量目标、matrix-free 的 Hessian-vector product，可对接外部 GPU 稀疏线性求解器。

**效果**：在弹性与布料仿真、曲面参数化、网格平滑、frame field 设计、ARAP 变形、球面流形优化等任务上，**一致优于** PyTorch、JAX、Warp、DrJIT、EnzymeAD、Thallo；覆盖 Newton、Gauss-Newton、L-BFGS、梯度下降等优化器。

---

### 10.5 附：Surface Chamfering for Robust Tetrahedral Meshing 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Lorenzo Diazzi（CNR IMATI）, Daniele Panozzo（NYU）, Jiacheng Dai（NYU）, Marco Attene（CNR IMATI） |
| **出处** | ACM TOG 45(4), Article 148, 15 pp., DOI 10.1145/3811395；7/23 10:55, Room 408 A |
| **链接** | 代码 https://github.com/MarcoAttene/DelOptim |

**核心问题**：生成与输入多面体**共形**的高质量四面体网格。Ruppert 的 Delaunay 精化在输入含锐角时不保证收敛。

**方法要点**：新的"**倒角（chamfering）**"预处理移除输入的所有锐角 → Delaunay 精化在修改后的输入上收敛且所有面角度有界 → 再把被倒角切掉的输入部分**重新插入**以达到精确共形，代价是原锐角附近出现少量形状不佳的四面体。全流程的数值鲁棒性依靠现代**间接几何谓词**与一种新的**隐式点**（表示输入面上的 Steiner 顶点）来保证。

**效果**：质量与 SOTA 的 tetgen 相当；但**tetgen 在 Thingi10k 的 3,942 个有效模型上失败率 37%，本方法 100% 成功**。

---

## 十一、材料建模 / 外观 / 毛发（3 篇）

### 11.1 Woodstock: Interactive Modeling of Fungal Wood Decay 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Zhanyu Yang, Bedrich Benes（普渡大学）；Nikolas Schwarz, Sören Pirk（基尔大学）；Bosheng Li（三星）；Dominik L. Michels（KAUST）；Wojtek Pałubicki（密茨凯维奇大学，波兰） |
| **出处** | ACM TOG 45(4) |
| **链接** | https://jango6324.github.io/woodstock |

**核心问题**：真菌导致的木材腐朽是复杂的**生物物理**现象，涉及从木质素、碳水化合物到防御性化学物质等多种结构组分的降解。这些底物作为**具有不同材料属性的资源**，共同决定真菌的扩展速率、腐木的结构完整性与颜色。

**方法要点**：仿真木材腐朽中**生物与力学组分的动态相互作用**：真菌定殖、化学防御、**湿度驱动断裂**。提出新颖的树木**体表示**：**顺纹理（grain-aligned）网格生成** + 内部湿度动力学 + **组织特异的健康状态**。建模**白腐菌与褐腐菌**引起的**各向异性扩散、消耗与随之而来的材料失效**；力学侧采用**基于纤维束（strand-based）**的木材力学，断裂图案与 BSP 配置相关。

**效果**：可仿真并渲染 3D 体腐朽树木，真实再现**立方体（cubical）裂纹图案的推进**、**树干中空化**、碎屑堆积，以及环境湿度对结构稳定性的影响。示例包括褐腐渐进降解的原木、木板的生物物理反馈、木材各向异性导致的不同开裂图案、橡树死后随时间演化。

---

### 11.2 Physics-Inspired Procedural Texturing of Extremely Deformable Surfaces 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Aleksei Kalinov, Mickaël Ly, Christian Hafner, Chris Wojtan（**ISTA**，Wojtan Group） |
| **出处** | ACM TOG 45(4), Article 154, DOI 10.1145/3811353 |
| **链接** | https://research-explorer.ista.ac.at/record/21923 |

**核心问题**：在动态可变形表面上贴纹理极难——涉及的**长度尺度差异不断变化**。表面运动并输运纹理时，变形区会**剧烈扭曲 UV** 导致外观退化；而直接在**像素级**响应变形来修改纹理，则会引入**重影伪影**且看起来不自然。

**方法要点**：受真实世界机制（破裂、内部结构暴露、起皱）启发，以**表面变形度量**为依据引导纹理生成；提出**两种基于波（wave-based）的过程化纹理算法**，复现**输运（advection）**与**自相似性**等物理性质。方法**完全过程化**：无需真实物理仿真，除输入 UV 外**不存储任何变形状态或历史** → 高度 GPU 并行、满足实时要求。

**效果**：在极端变形现象上验证——**流动岩浆、拉伸的橡皮泥、外溢的泥浆**。开放获取（CC-BY），PDF / 补充材料 / 视频齐备。

---

### 11.3 Curvature Space Editing of Highly-Coiled Hair 【摘要原文 + 项目页】

| 项目 | 内容 |
|---|---|
| **作者** | Alvin Shi, Florence Bertails-Descoubes（Inria ELAN，【推断】）, A.M. Darke, Theodore Kim |
| **链接** | https://alvin.pizza/curvatureSpaceEditing/ |

**核心问题**：紧密卷曲的头发几何曲率极高，标准的**基于位置**的工具难以建模与编辑。

**方法要点**：改用**材料曲率与扭率（material curvatures and twists）**空间。核心表示为**super-helices**——以**分段常数曲率/扭率**参数化的图元，其螺旋几何天然贴合卷发丝。在该曲率/扭率空间中定义编辑算子：expand / shrink / **ruffle** / interpolate / guide。提供几何量与梯度的**解析表达式**，因此高效且**完全不需要训练数据**（非学习方法）。

**效果**：应用于高度卷曲的仿真头发与程序化生成头发。对比实验：本文的曲率空间 ruffling vs. Blender 基于噪声的位置空间 scraggle 修改器（对象为 Code My Crown 的 Headwrap Curls），结论是本方法的纹理更接近真实参考照片。演示包含插值的三种变体（位置/全局曲率/局部曲率）、骨架与笼形变形控制、马尾整头卷曲化、遮眼发束、整头拉伸。

---

## 十二、World Model × 物理模拟交叉方向

用户特别关注这个方向，因此本节单独梳理，并**严格区分论文层面与会议/产品层面的信息**。

### 12.1 核心判断

**世界模型（world model）在 SIGGRAPH 2026 的论文轨基本缺席，几乎完全集中在keynote 与产品发布层面。** 而AI 与物理模拟的真实交汇点，落在**仿真就绪资产生成**与**物理感知的运动控制**这两个具体位置上，而非求解器内核。

### 12.2 已确证的交叉方向论文

#### SimArt: Decomposing Monolithic Meshes into Sim-ready Articulated Assets via MLLM 【摘要原文】

| 项目 | 内容 |
|---|---|
| **作者** | Chuanrui Zhang（南洋理工大学）, Minghan Qin, Yuang Wang, Baifeng Xie, Hang Li（**字节跳动 Seed**）, Ziwei Wang（NTU） |
| **出处** | 7/21 09:10, Room 403 B, session "Generative 3D (2)"；arXiv:2603.23386 |
| **链接** | https://SimArt-mllm.github.io ; 代码 github.com/ByteDance-Seed/SimArt |

**核心问题**：具身 AI 与物理仿真需要"sim-ready"的铰接资产，但 3D 生成仍停留在静态网格。多阶段流水线（部件分割→关节推断→组装）会累积误差，且分割缺乏 articulation-aware；统一 MLLM 路线则受**密集体素 tokenization** 的长序列与显存开销限制。

**方法要点**：基于 Qwen3-VL-8B 的统一 MLLM，从视觉/原始几何/文本联合完成部件级分解 + 运动学参数预测，输出离散部件体素 token 与结构化 **URDF**；引入**稀疏 3D VQ-VAE**（用零 token 表示未占用体素）。

**效果**：token 数比密集体素**减少约 70%**，避免 OOM；域内 type accuracy **0.928**、axis error **0.080**、origin error **0.111**，优于 Articulate-Anything、Particulate；在 PartNet-Mobility 与野外 AIGC 数据集上达 SOTA，支持物理机器人仿真。

---

#### Computational Design of Terrestrial Robots with Anisotropic Friction 【项目页/新闻稿描述】

| 项目 | 内容 |
|---|---|
| **作者** | Hang Hu, Kangbo Lyu, Changyu Hu, Zihan Li, Peiwen Yang, Minchen Li, Shuguang Li, Tao Du（CMU / **Genesis AI** / 清华大学 / **上海期智研究院**） |
| **链接** | 论文 people.iiis.tsinghua.edu.cn/~taodu/snake/snake.pdf ; 代码 github.com/Hang-Hu1/snake |

**核心思路**：**协同设计（co-design）**机器人本体与运动控制器，利用**各向异性摩擦**实现地面移动（蛇形机器人）。

**信息缺口**：未获取到摘要原文与量化数据。

---

#### ReActor: Reinforcement Learning for Physics-Aware Motion Retargeting 【二手转述】

**作者/单位**：Disney Research。会议编号 papers_1360 / sess114。

**方法要点**：**双层优化**——下层训练 RL 跟踪策略，上层同时精化由**稀疏语义对应**定义的重定向参数，从而免去手工调整对应关系。

**效果**：把人类运动学参考迁移到人形与四足形态，不产生**脚滑、悬空、自穿透**等伪影，改善下游模仿学习效果。

> ⚠️ 该描述来自中文技术媒体转述，建议以 ACM DL 原文复核。

---

#### MUSIC: Learning Muscle-Driven Dexterous Hand Control 【二手转述】

**作者**：Pei Xu, Yufei Ye, Shuchun Sun, Yu Ding, Elizabeth Schumann, C. Karen Liu（TOG 45(4), July 2026；据作者构成【推断】为斯坦福团队）。

**方法要点**：数据驱动 + 基于物理的**肌肉驱动**灵巧手控制；分层架构耦合高频肌肉级控制与低频动作协调。

**效果**：实现双手协调、产生**类人的肌肉激活模式**，并能泛化到参考数据集之外的**新钢琴曲目**。

---

#### 其他相关条目

| 论文 | 要点 | 可靠度 |
|---|---|---|
| **Kinematic Kitbashing** | 基于抽象**运动学图**装配可复用部件来合成铰接 3D 物体，用运动学感知的几何线索保证运动中连接合理，可对功能目标做优化 | 【二手转述】 |
| **MotionBricks** | 35 万+ 动作片段训练，游戏引擎级实时，**15,000 FPS 零样本动作合成**；同一模型可驱动屏幕角色与 Unitree G1 人形机器人（NVIDIA） | 【二手转述】 |
| **GPC** | 大规模动作数据集上预训练的生成式控制器，被定位为"运动控制基础模型"雏形（NVIDIA） | 【二手转述】 |
| **ARDY** | 自回归扩散，文本提示实时操控 3D 角色（NVIDIA） | 【二手转述】 |
| **Neural Particle Automata: Learning Self-Organizing Particle Dynamics** | Pajouheshgar, Kim, Süsstrunk, Jakob, Park；learned dynamics 方向，7/21 同一 session | 【仅标题推断】 |

### 12.3 Technical Workshop: Differentiable Physics for Graphics and AI（DPGA）【官方日程描述原文】

这是本届"物理 × AI"最集中的**组织性信号**。

- **时间地点**：7/20（周一）14:00–17:15，Room 406 AB，半日制，编号 twork_111 / sess224
- **组织者**：Minchen Li（CMU + Genesis AI）、Chenfanfu Jiang（UCLA）、Yin Yang（Utah）、Tuur Stuyck（**Meta Reality Labs**）
- **讲者**：Ming Lin（UMD）、Tuur Stuyck（Meta）、Daniele Panozzo（NYU）、**Miles Macklin（NVIDIA，Warp/Newton 核心作者）**、Guying Lin（CMU）、Yunuo Chen（UCLA）、Kemeng Huang（HKU）、Dewen Guo（Utah）、Ying Jiang（UCLA）、Yifei Li（MIT）
- **议题**：可微仿真如何重塑图形、机器人、3D 视觉、设计与制造；仿真与优化/学习/生成流水线深度融合后，可微性支撑**逆问题、系统辨识、physics-aware design**
- **官方明确点出的技术壁垒**：效率、鲁棒性、接触密集动力学，以及"面向应用的表示"与"仿真就绪资产"之间的落差

> **值得注意的观察**：组织者与讲者名单几乎完全覆盖了本报告第七、十节中求解器基础设施论文的作者群（Panozzo、Yin Yang 组、Minchen Li 组）。这说明在 SIGGRAPH 2026 的语境下，**基础设施派与 AI 派其实是同一批人**。

### 12.4 Keynote / 产品层面（非论文，须严格区分）

- **NVIDIA 赞助 keynote**「Next Era of Graphics — Neural Rendering, World Models, and Simulation」，7/20 15:45，讲者 Neil Ashton、Ming-Yu Liu、Edward Liu。Ming-Yu Liu（Cosmos Lab VP）讲 Cosmos 世界基础模型：mixture-of-transformers 主干统一"世界理解/预测/仿真/行动"四类任务，跨具身形态建立"通用词汇表"；当日发布 **Cosmos 3 Edge**（40 亿参数，端侧实时），并宣布 **Cosmos-Dreams**（闭环仿真器集合，单帧生成整个世界，可在单张 RTX PRO 6000 上运行）。Ashton 讲 AI physics（Earth-2，checkpoint 压缩百万倍至数百MB，秒级物理精确可视化）。Edward Liu 讲 3D-guided neural rendering / DLSS 5（2026 秋发布）。
- **Newton 物理引擎**：开源、GPU 加速、**可微**的物理引擎，由 NVIDIA 与 **Google DeepMind + Disney Research** 共同开发，基于 Warp 与 OpenUSD，GTC 2026 首发。SIGGRAPH 2026 有 NVIDIA 课程「Accelerate Robot Learning With NVIDIA Isaac Lab and Newton」。⚠️ **Newton 本身不是 SIGGRAPH 论文**。
- **NVIDIA 21 篇技术论文**：官方博客确认了数量，但仅具名 5–6 项，其余标题未公开披露。
- **Tripo AI keynote**（7/22）：主题即"3D 生成式 AI 与 world models 的全栈演进"，明确面向 robotics simulation。
- **课程 / 前沿工作坊**：3 门 NVIDIA physical AI 课程（"How To Build End-To-End Physical AI Systems for Robots"、"Accelerate Robot Learning With NVIDIA Isaac Lab and Newton"、"Simulating a Dextrous Hand For Robotics With OpenUSD"）；Frontiers Workshop「Digital Twins for Science and Industry」。7/21 的 **Physical AI Day** 是 NVIDIA 主办活动，非SIGGRAPH 论文轨。

### 12.5 趋势判断

1. **"物理模拟"今年的主赛道仍是经典数值方法，而非神经替代。** 35 篇合集里，求解器/接触/MPM/流体类占绝大多数，明确的 learned simulator 只有 Boundary-aware Neural Model Reduction 与 Neural Particle Automata 等零星几篇。JGS2-GQ 甚至是**反向信号**——主动把前作的数据驱动 Cubature 换成免训练的高斯求积。

2. **AI 的真实切入点是"资产"与"控制"，不是"求解器内核"。** SimArt（静态 mesh→URDF 铰接资产）、ArtiFixer（真实采集→干净场景）、Kinematic Kitbashing（部件→铰接物体）解决的正是 DPGA workshop 官方点出的那道落差——"应用侧表示 ↔ 仿真就绪资产"。而 MotionBricks / GPC / ARDY / ReActor / MUSIC 则集中在物理感知的运动控制与重定向。

3. **可微性是连接两侧的关键词，且已有明确的组织载体**（DPGA workshop + Newton 引擎），但**论文轨中尚未检索到任何一篇以 differentiable simulation 为标题的技术论文**。这说明该方向在 2026 年处于"社区共识已形成、代表性论文尚未沉淀"的阶段。

4. **机构信号值得注意**：Genesis AI 同时出现在 Untangling Surfaces、Efficient B-Spline Cloth、Barrier-free Elastodynamics、AGIPC、Terrestrial Robots 与 DPGA 组织方；ByteDance Seed 拿下 SimArt；Roblox Research、Meta Reality Labs、Disney Research、丰田研究院（TRI）、LIGHTSPEED、Style3D 均在场。**物理仿真基础设施正被具身智能公司系统性收编。**

---

## 十三、奖项、趋势与开源清单

### 13.1 物理模拟相关奖项

| 奖项 | 论文 |
|---|---|
| 🏆 **Best Paper** | Mixwell: Sharp 2D Fluid Brushes for Progressive Physics-Based Mixing |
| 🏅 **Honorable Mention** | Spatiotemporal FLIP for Fast Free-Surface and Two-Phase Simulation With Very Large Time Steps |
| 🏅 **Honorable Mention** | Stochastic Geomorphological Transport for Terrain Erosion Simulation |

> 另外，本届 Best Paper 还包括非物理模拟方向的 Robust Planar Maps for 3D Vectorization（CMU，Robert Fuchs & Keenan Crane）。

### 13.2 量化加速一览（便于横向对比）

| 论文 | 加速幅度 | 对比基线 |
|---|---|---|
| Robust and Efficient Penetration-Free Elastodynamics | **103×** | GIPC |
| Fast VEM Fluid Simulation | **100×**（压力投影） | 现有 cut-cell 求解器 |
| Tube Maps | **1–3 个数量级**（边界处理） | 隐式 SDF 积分法 |
| SymX | **≥1 个数量级** | TinyAD |
| MPM Lite | **15.9×** / **1.88×** | 隐式 MPM / 显式 MPM |
| MeshFEM | **14×**【社媒声明】 | — |
| ST-FLIP | 时间步大 **1 个数量级**，数倍整体加速 | CFL 受限求解器 |
| Fast Sparse Matrix Permutation | **6.27×** | 厂商稀疏 Cholesky |
| AGIPC | **3×** | SOTA GPU IPC |
| Efficient B-Spline FEM for Cloth | **2×** | 线性 FEM |
| JGS2-GQ | **1.5×** 且免训练 | JGS2 |

### 13.3 规模记录

| 论文 | 规模 |
|---|---|
| M-ABD | **1,076,748连杆**滑轮系统，单线程 904 ms/帧 |
| ST-FLIP | **数十亿粒子**，单工作站 |
| Mixed MPM | **4,900 万粒子**，单 GPU 4 秒/帧 |
| Barrier-free Elastodynamics |2.61M DoF / 峰值 1.45M 活跃接触，5.37 s/帧 |
| Distributed ABD | **5,000 刚体 / 600 万三角面**，全程无穿透 |
| Yarn-level Knitwear | **百万级 DoF** 交互帧率 |

### 13.4 开源项目清单

| 论文 | 仓库 /安装方式 |
|---|---|
| Stochastic Geomorphological Transport | github.com/erosiv/geotransport；`pip install geotransport` |
| A Nonlocal Monolithic Variational Framework | github.com/peridyno/peridyno |
| SymX | github.com/InteractiveComputerGraphics/SymX |
| YASPS | github.com/txstc55/yasps |
| Locality-Aware AD on GPU | github.com/owensgroup/RXMesh |
| MeshFEM | github.com/MeshFEM/MeshFEM |
| Fast Sparse Matrix Permutation | github.com/BehroozZare/fast-permute |
| Surface Chamfering | github.com/MarcoAttene/DelOptim |
| High-Order Continuous Geometrical Validity | gitlab.com/fsichetti/hocgv |
| Better Bending | github.com/evouga/better-bending |
| SimArt | github.com/ByteDance-Seed/SimArt |
| Terrestrial Robots with Anisotropic Friction | github.com/Hang-Hu1/snake |
| Volume-Preserving LBM-MPM | 已开源（见项目页） |
| Efficient B-Spline FEM for Cloth | 已开源（见项目页） |

---

## 附录 A：SCA 2026 物理模拟论文（勿与 SIGGRAPH 混淆）

以下 10 篇属于 **SCA 2026**（Symposium on Computer Animation），主题高度相关但**不是 SIGGRAPH 2026 论文**，容易被混淆，特此列出以供参考：

1. Dynamic Wrinkling on Coarsely-Meshed Cloth
2. Physics-Based Simulation of Contact-Induced Facial Wrinkling — Juan Sebastian Montes Maestre, Ladislav Kavan, Edmond Boyer, Ryan Goldade, Stelian Coros, Bernhard Thomaszewski。人脸皮肤是非线性分层材料，与底层组织的附着空间异质；接触时局部压缩与剪切诱发力学不稳定，产生细尺度皱纹
3. Adaptive Fluid Cohomology on Surfaces
4. Primal SPH Solver for Strongly Coupled Multiphase Simulations with High Density Ratios
5. Fluid Control with Localized Spacetime Windows
6. Closing Trajectories: Equation-Free Cyclic Animation via Koopman Surrogates
7. A Splitting Architecture for Exact Reduced Coulomb Friction
8. Weak Nonlinearity and Linear Equalization for Isotropic Hyperelastic Materials
9. OrganPhys: Scope Captures to CT-Informed, Mesh-Free, Physics Sim Ready Deformables
10. Alternating Spatial-Temporal Optimization for Continuous Collision Detection of Signed Distance Fields — Rasmus Gabelgaard Nielsen, Kenny Erleben。把 CCD 转为交替时空优化，沿 SDF 切空间驱动少量智能采样点，完全避免密集采样与三角网格；**单 CPU 核常在 1 ms 内**完成 SDF–SDF time-of-impact 查询，为已知首个直接作用于 SDF 对的实用 CCD 方法

---

## 附录 B：信息缺口声明

为保证报告可信度，以下信息缺口明确列出：

**未获取到官方摘要原文的论文**：
- MeshFEM（仅有 GitHub 说明与作者社媒的"14× faster"表述）
- Progressing Level-of-Detail Animation for Volumetric Elastodynamics（官方 abstract 标注"Details coming soon"）
- Computational Design of Terrestrial Robots with Anisotropic Friction
- ReActor、MUSIC、Kinematic Kitbashing（现有描述为中文技术媒体的二手转述）

**未公开的量化数据**：
- Divide and Truncate（DAT）：摘要仅定性，需查 PDF Results
- Heterogeneous Subspace Corrections：未给具体加速倍数
- Low-Rank Koopman Deformables：仅有 log-linear 缩放这一结论
- Floating-Point Robustness in Parametric Surface CCD：未给误报/漏报率
- Better Bending：10 个模型名单与定量表格仅存在于 PDF 正文

**作者机构未经证实（标注为推断）**：
- Divide and Truncate（DAT）
- YASPS
- Floating-Point Robustness in Parametric Surface CCD（一作邮箱疑为北大）
- Low-Rank Koopman Deformables（Chiaramonte 的 Meta 归属）
- MUSIC（斯坦福归属）
- Curvature Space Editing（Inria ELAN 归属）

**归属存疑的论文**：
- SymX：TOG 45(1) 而非 45(4)，physicsbasedanimation 将其归入 SIGGRAPH 2026 合集
- High-Order Continuous Geometrical Validity：NYU GCL 官方页与 PDF 均标注 TOG 2025（arXiv 2024年 6 月），可能是 TOG 论文在 SIGGRAPH 2026 排期呈现

**其他**：
- NVIDIA 21 篇技术论文中约 15 篇标题未公开披露
- s2026.conference-schedule.org 在本次检索环境下无法直连（TLS 受限），仅能通过搜索引擎索引读取，未做逐 session 全量遍历。因此本报告以 physicsbasedanimation.com 的 35 篇合集为主干，可能遗漏少量未被该站收录的物理模拟论文

---

*报告生成时间：2026 年 8 月 10 日*
