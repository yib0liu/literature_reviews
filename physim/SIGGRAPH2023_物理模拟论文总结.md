# SIGGRAPH 2023 物理模拟论文逐篇总结

> 会议：ACM SIGGRAPH 2023（第 50 届），2023 年 8 月 6–10 日，美国洛杉矶
> 收录范围：技术论文（Conference Track TOG 42(4) + Journal Track），筛选**物理模拟 / 物理动画**相关工作
> 覆盖主题：流体（液体、气体、湍流、波）、固体与弹性体、薄壳 / 布料 / 杆 / 头发、接触与碰撞、材料点法、降阶与神经加速、刚体多体、物理声音合成、物理驱动角色
> 共收录 **33 篇**

---

## 目录

| 分类 | 篇数 | 论文 |
|---|---|---|
| 一、流体模拟 | 6 | Contact Proxy Splitting / Fluid Cohomology / PolyStokes / Kinetic Two-Phase / Virtual Wind Tunnel / Dispersive Shallow Water |
| 二、固体、弹性体与弹塑性 | 5 | Second-order Stencil Descent / Nonlinear Compliant Modes / Motion From Shape Change / Somigliana Coordinates / Gigascale MPM |
| 三、薄壳、布料、杆与头发 | 6 | Multi-Layer Thick Shells / Complex Wrinkle Field / Multistable Elastic Knots / Beyond Chainmail / Sag-Free Hair Init / Hair on GPU (ADMM) |
| 四、接触与碰撞 | 7 | In-Timestep Remeshing / High-Order IPC / Fast GPU CCD / Sum-of-Squares CCD / Shortest Path to Boundary / P2M / Passive Suction Cups |
| 五、降阶模型与神经加速 | 2 | Data-Free Reduced-Order Kinematics / Fast Complementary Dynamics |
| 六、物理声音合成 | 1 | Improved Water Sound Synthesis |
| 七、物理驱动角色与机器人 | 6 | Anatomically Detailed Torso / Bidirectional GaitNet / Tennis from Broadcast / CALM / Character-Scene Interactions / DOC |

---

## 一、流体模拟

### 1. A Contact Proxy Splitting Method for Lagrangian Solid-Fluid Coupling

- **作者**：Tianyi Xie, Minchen Li, Yin Yang, Chenfanfu Jiang（UCLA / CMU / Utah）
- **问题**：SPH 流体与 FEM 固体的双向耦合中，如何在保证**零穿透**的同时兼顾效率。传统方法要么穿透，要么因接触约束导致线性系统极难求解。
- **方法**：提出统一的、无穿透的双向耦合框架，把 SPH 流体与 FEM 固体统一到基于**内点障碍函数（interior point barrier）**的摩擦接触模型下。核心创新是一种新的**算子分裂（operator splitting）时间离散方案**，引入「增广接触代理（augmented contact proxies）」——即用一个廉价的代理项承担接触耦合，从而把原本强耦合的大系统拆成若干可用定制线性求解器高效处理的子系统。
- **贡献**：把 IPC 式的严格无穿透保证从固体扩展到固-液耦合；分裂方案 + 定制求解器带来显著加速。
- **意义**：这是「IPC 路线」向流体领域的关键一步。对做 solid-fluid 交互（如物体入水、湿布料）的场景，提供了鲁棒性极强的基线。

### 2. Fluid Cohomology

- **作者**：Hang Yin, Mohammad Sina Nabizadeh, Baichuan Wu, Stephanie Wang, Albert Chern（UC San Diego）
- **问题**：涡量-流函数（vorticity-streamfunction）形式的流体方程在**非单连通域**（有孔洞、有障碍物的区域）上是不完备的——速度场的 Helmholtz 分解中除了梯度部分与旋度部分，还存在一个**调和（harmonic）分量**，其对应上同调（cohomology）空间。已有的图形学流体求解器完全忽略了这一分量的动力学。
- **方法**：从微分几何 / 代数拓扑出发，推导出流体上同调分量所缺失的运动方程，把这些分量的时间演化补进求解器。
- **贡献**：理论性补全。指出并修正了图形学流体模拟长期存在的一个系统性缺陷。
- **意义**：偏理论但影响深远。任何在带孔洞域（如管道、多连通障碍）上做涡量法模拟的工作都应考虑这一修正；这类「先把数学做对」的工作往往成为后续方法的基石。

### 3. PolyStokes: A Polynomial Model Reduction Method for Viscous Fluid Simulation

- **作者**：Jonathan Panuelos, Ryan Goldade, Eitan Grinspun, David I. W. Levin, Christopher Batty（Waterloo / Toronto）
- **问题**：粘性流体需要求解**非稳态 Stokes 问题**，这是一个大规模鞍点系统，求解代价高。
- **方法**：提出降阶流体模型：注意到流体内部区域（远离自由表面与边界）的运动往往「平滑」，于是用**不可压缩多项式向量场**来表示这些内部区域，只在边界附近保留完整自由度。这样把大量内部格点自由度压缩成少量多项式系数。
- **贡献**：在保持视觉质量的前提下显著降低 Stokes 求解规模。
- **意义**：典型的「空间自适应降阶」思路，与全局子空间方法（如 POD）不同，它是**局部的、几何驱动的**，无需预计算训练数据，工程上更好用。

### 4. Fluid-Solid Coupling in Kinetic Two-Phase Flow Simulation

- **作者**：Wei Li, Mathieu Desbrun（Inria / École Polytechnique）
- **问题**：基于格子玻尔兹曼（LBM）的动理学（kinetic）多相流求解器速度快，但在固-液耦合时会产生明显伪影，能处理的耦合类型受限。
- **方法**：在基于**相场（phase-field）+ 速度分布函数**的 LBM 多相流求解器之上，系统性改进三处数值处理：（1）动量交换（momentum exchange）；（2）界面力（interfacial forces）；（3）双向耦合。
- **贡献**：大幅减少典型伪影，显著扩展了可高效模拟的固-液耦合类型——能稳定表现冒泡（bubbling）、飞溅（splashing）、咕咚倒水（glugging）、浸润（wetting）等此前难以捕捉的现象。
- **意义**：LBM 在图形学里性能优势明显（天然并行），这篇把它的短板（耦合与界面）补齐，实用价值高。

### 5. Building a Virtual Weakly-Compressible Wind Tunnel Testing Facility

- **作者**：Chaoyang Lyu, Kai Bai, Yiheng Wu, Mathieu Desbrun, Changxi Zheng, Xiaopei Liu（ShanghaiTech / Columbia 等）
- **问题**：图形学的流体求解器通常只追求「看起来对」，达不到工业风洞测试的精度标准；而 CFD 的高精度求解器又太慢。
- **方法**：融合图形学与计算流体力学在**高性能动理学求解器**上的最新进展，构建虚拟风洞系统。采用**弱可压缩（weakly-compressible）**动理学模型，实现高效且精确的高雷诺数非稳态湍流模拟。
- **贡献**：一个既高效又达到工程精度的虚拟风洞，可用于气动外形评估。
- **意义**：图形学方法向工程仿真「反向输出」的代表作。它证明了 LBM 类方法在精度上可以对标工业标准，而非只能做视觉效果。

### 6. Generalizing Shallow Water Simulations with Dispersive Surface Waves

- **作者**：Stefan Jeschke, Chris Wojtan（NVIDIA / IST Austria）
- **问题**：高度场（height field）方法模拟大范围水面很高效，但传统浅水方程（shallow water equations）假设波速只由水深决定，**无法表现色散（dispersion）**——即不同波长的波以不同速度传播这一真实水波的核心特征。
- **方法**：提出新的高度场大水体模拟方法，把浅水模拟推广到能捕捉色散表面波行为。
- **贡献**：在保持高度场方法效率优势的同时，获得了此前只有完整 3D 求解器才能表现的色散波纹细节。
- **意义**：对海洋 / 湖面等大尺度水体的实时或准实时渲染很有价值，是「用更好的物理模型换视觉真实度」而非「用更高分辨率换真实度」。

---

## 二、固体、弹性体与弹塑性

### 7. Second-order Stencil Descent for Interior-point Hyperelasticity

- **作者**：Lei Lan, Minchen Li, Chenfanfu Jiang, Huamin Wang, Yin Yang（Utah / CMU / UCLA / Style3D）
- **问题**：基于 IPC（Incremental Potential Contact）的超弹性 FEM 模拟鲁棒性极佳，但依赖全局 Newton 求解，难以在 GPU 上高效并行。
- **方法**：提出 GPU 上的**二阶 stencil descent** 算法。核心思路是把求解从「全局线性系统」下放到**局部 stencil（单元邻域）**层面，在每个 stencil 上使用二阶（Newton 型）信息进行下降，从而既保留二阶收敛的快速性，又获得逐元素并行的可扩展性。
- **贡献**：让内点法超弹性模拟真正跑在 GPU 上，兼顾鲁棒性与速度。
- **意义**：解决了「IPC 很鲁棒但很慢」这一长期痛点。属于 IPC 加速路线上的重要一环（与 Multi-Layer Thick Shells、Subspace-Preconditioned PD 等同属这一批 Utah/UCLA/CMU 系工作）。

### 8. Nonlinear Compliant Modes for Large-deformation Analysis of Flexible Structures

- **作者**：Simon Duenser, Bernhard Thomaszewski, Roi Poranne, Stelian Coros（ETH Zürich CRL）
- **问题**：线性特征模态（linear eigenmodes）是分析结构变形行为的经典工具，但只在小变形下有效，无法解释**大变形下的非线性现象**。
- **方法**：提出线性特征模态的物理原理性扩展——**非线性柔顺模态（nonlinear compliant modes）**，用于大变形分析。
- **贡献**：能够揭示并分析**屈曲（buckling）、刚化（stiffening）、多稳态（multistability）**等纯非线性现象。
- **意义**：为柔性结构的计算设计提供了分析工具。设计柔性机构 / 软体机器人时，多稳态和屈曲往往是要主动利用的特性而非要规避的缺陷，这篇给了定量刻画手段。（发表于 TOG 42(2)，在 SIGGRAPH 2023 宣讲）

### 9. Motion From Shape Change

- **作者**：Oliver Gross, Yousuf Soliman, Marcel Padilla, Felix Knöppel, Ulrich Pinkall, Peter Schröder（TU Berlin / Caltech）
- **问题**：动画师通常只指定物体的**形状变化序列**（如变形、伸缩），但物体在世界坐标系中应该如何**整体平移和旋转**？这在自由落体、太空、水中等无外力场景下由动量守恒唯一决定。
- **方法**：抓住「无外力时动量恒定」这一物理约束，设计算法：输入一段形状变化序列（如动画师建模的），输出世界坐标系中对应的运动。
- **贡献**：把「形变」自动转化为「位移与旋转」，无需动画师手动 K 出整体运动。
- **意义**：优雅的几何力学应用。对角色的空中动作、鱼 / 蛇的游动、太空中的姿态调整等场景，能自动获得物理正确的整体运动。这类工作对 crowd / motion 方向也有启发：形变与整体运动的耦合是可以由守恒律唯一确定的。

### 10. Somigliana Coordinates: an Elasticity-derived Approach for Cage Deformation

- **作者**：Jiong Chen, Fernando de Goes, Mathieu Desbrun（Inria / Pixar）
- **问题**：笼形变形（cage deformation）是几何处理的经典工具，但传统重心坐标类方法（如 Green coordinates）缺乏物理意义，变形结果常有非自然的体积变化或剪切。
- **方法**：基于弹性力学中的 **Somigliana 恒等式**（弹性体边界积分表示），推导出**矩阵值（matrix-valued）坐标**，构造新的笼形变形器。
- **贡献**：变形行为源自弹性理论，可通过材料参数（如泊松比）控制变形风格。
- **意义**：把「几何变形」和「弹性物理」统一起来，让变形器有了可解释的物理旋钮。

### 11. A Sparse Distributed Gigascale Resolution Material Point Method

- **作者**：Yuxing Qiu, Samuel Temple Reeve, Minchen Li, Yin Yang, Stuart Ryan Slattery, Chenfanfu Jiang（UCLA / ORNL / Utah）
- **问题**：材料点法（MPM）能统一模拟弹塑性、颗粒、雪、泥等复杂材料，但受单机内存与算力限制，分辨率难以突破。
- **方法**：设计四层分布式架构（MPI 进程 / block / tile / cell 的层级稀疏数据结构），把 MPM 扩展到多节点集群。
- **贡献**：实现了高达 **10.1 亿（1.01 billion）粒子**的吉规模（gigascale）MPM 模拟。
- **意义**：MPM 的算力天花板被推高一个量级。对需要极高分辨率细节（如大规模破碎、雪崩、泥石流）的场景意义重大。（发表于 TOG 42(2)，在 SIGGRAPH 2023 宣讲）

---

## 三、薄壳、布料、杆与头发

### 12. Multi-Layer Thick Shells

- **作者**：Yunuo Chen, Tianyi Xie, Cem Yuksel, Danny M. Kaufman, Yin Yang, Chenfanfu Jiang, Minchen Li（UCLA / Utah / Adobe / CMU）
- **问题**：真实的壳状物体（如多层衣物、鞋底、复合材料）有**厚度**且常为**多层**结构。用薄壳（thin shell）模型无法表现厚度效应；用完整 3D 四面体则代价过高，且薄方向的单元易出现**剪切锁死（shear locking）**。
- **方法**：提出基于网格的连续介质**厚壳（thick shell）**模拟方法。核心是**双四分点（dual-quadrature）棱柱单元（prism element）**有限元格式，专门用于避免剪切锁死；并将其与高分辨率膜（membrane）耦合，支持多层结构。
- **贡献**：首次高效稳定地模拟（可多层的）连续介质厚壳的复杂动力学。
- **意义**：填补了「薄壳」与「完整 3D 固体」之间的空白。对服装（尤其多层叠穿）、鞋类等仿真是直接可用的工具。

### 13. Complex Wrinkle Field Evolution

- **作者**：Zhen Chen, Danny M. Kaufman, Mélina Skouras, Etienne Vouga（UT Austin / Adobe / Inria）
- **问题**：褶皱（wrinkle）是薄壳视觉真实感的关键，但要在**粗网格**上表现复杂细密的褶皱行为，需要一种既紧凑又能演化的褶皱表示。
- **方法**：提出 **Complex Wrinkle Fields（CWF）**——用复值场（复振幅 + 相位）在粗三角网格上编码褶皱，并给出其时间演化方案。
- **贡献**：在粗网格上捕捉复杂、细节丰富的褶皱行为，并支持褶皱随时间连续演化（而非逐帧独立生成）。
- **意义**：「粗网格 + 细节场」的经典范式在褶皱上的高质量实现。相位表示天然支持褶皱的插值与混合，避免了直接位移插值的鬼影。

### 14. Computational Exploration of Multistable Elastic Knots

- **作者**：Michele Vidulis, Yingying Ren, Julian Panetta, Eitan Grinspun, Mark Pauly（EPFL / UC Davis / Toronto）
- **问题**：弹性绳结（elastic knot）在张力下可以有多个稳定平衡态（多稳态），但这些状态的发现与设计缺乏系统方法。
- **方法**：提出算法化方法来**发现、研究、设计**多稳态弹性绳结，结合弹性杆模拟与平衡态搜索。
- **贡献**：把绳结的多稳态从「偶然发现」变成「可计算探索与主动设计」。
- **意义**：与 Nonlinear Compliant Modes 呼应——多稳态是柔性结构的可设计资源。对可展开结构、柔性机构设计有应用价值。

### 15. Beyond Chainmail: Computational Modeling of Discrete Interlocking Materials

- **作者**：Pengbin Tang, Stelian Coros, Bernhard Thomaszewski（ETH Zürich）
- **问题**：**离散互锁材料（Discrete Interlocking Materials, DIM）**——即由准刚性互锁单元构成的 3D 打印锁甲织物——其宏观力学行为由微观单元的接触与互锁决定，直接模拟每个单元代价极高。
- **方法**：完整流程：计算建模 → 力学表征（mechanical characterization）→ **宏观尺度均质化（homogenized）模拟**。用均质化模型替代逐单元模拟。
- **贡献**：得到能反映 DIM 特有各向异性与变形极限的宏观模型，可高效模拟大片锁甲织物。
- **意义**：数字制造与仿真的结合。DIM 有独特的「可弯不可拉」特性，均质化建模让设计-仿真闭环成为可能。

### 16. Sag-free Initialization for Strand-based Hybrid Hair Simulation

- **作者**：Jerry Hsu, Tongtong Wang, Zherong Pan, Xifeng Gao, Cem Yuksel, Kui Wu（Utah / LightSpeed Studios）
- **问题**：头发模拟启动时，发丝会在重力下「塌陷（sag）」偏离美术师精心设计的初始造型。混合式（strand + 连续介质）发丝系统的这一问题尤为棘手，因为要同时满足多套物理模型的平衡条件。
- **方法**：提出**四阶段无塌陷初始化框架**，为混合式发丝动力学系统求解稳定的**准静态（quasistatic）平衡配置**——即反求出一组内部应力 / 静止状态，使得美术师给定的造型本身就是重力下的平衡态。
- **贡献**：发丝在仿真开始时保持设计造型，不再塌陷。
- **意义**：极其实用的生产问题。美术师造型意图与物理平衡的矛盾是所有 sag-free 类工作的共同动机；这篇把它扩展到了混合式求解器。

### 17. Interactive Hair Simulation on the GPU Using ADMM

- **作者**：Gilles Daviet（NVIDIA）
- **问题**：带**库仑摩擦（Coulomb friction）**的离散弹性杆（Discrete Elastic Rods, DER）头发模拟质量高但速度慢，难以交互。摩擦是非光滑约束，难以并行化。
- **方法**：基于 **ADMM（交替方向乘子法）**设计 local-global 求解器，专门针对带库仑摩擦的 DER。ADMM 的分裂结构让局部步骤（每根发丝 / 每个约束）完全独立，可充分利用现代 GPU 的大规模并行能力。
- **贡献**：在 GPU 上实现带真实摩擦的交互式头发模拟。
- **意义**：ADMM 用于非光滑接触摩擦的成功范例。「local-global 分裂 + GPU」这一组合在本届会议多篇工作中反复出现（另见 SA 2023 的 Subspace-Preconditioned GPU PD），是当前物理求解器加速的主流方向之一。

---

## 四、接触与碰撞

### 18. In-Timestep Remeshing for Contacting Elastodynamics

- **作者**：Zachary Ferguson, Teseo Schneider, Danny M. Kaufman, Daniele Panozzo（NYU / Victoria / Adobe）
- **问题**：接触弹性动力学中，自适应网格重划分（remeshing）通常作为时间步**之间**的独立预 / 后处理步骤。这会破坏时间步内求解的一致性，可能引入穿透或能量伪影。
- **方法**：提出 **In-Timestep Remeshing**——全耦合的自适应网格算法，把重划分步骤**隐式地、紧密地嵌入时间步求解内部**，而非在步与步之间进行。
- **贡献**：重划分与接触求解在同一变分问题中协同进行，保持鲁棒性保证。
- **意义**：概念上的重要转变：把离散化本身作为求解变量的一部分。对需要自适应分辨率的接触问题（如布料起皱处细化）很有价值。

### 19. High-Order Incremental Potential Contact for Elastodynamic Simulation on Curved Meshes

- **作者**：Zachary Ferguson, Pranav Jain, Denis Zorin, Teseo Schneider, Daniele Panozzo（NYU / Victoria）
- **问题**：IPC 的原始formulation基于线性基函数与直边网格。高阶有限元（高阶基 + 曲边网格）精度更高、单元更少，但缺乏配套的鲁棒接触处理。
- **方法**：提出用于**曲面（高阶）网格**上弹性动力学的高阶有限元格式（高阶基函数），并将 IPC 接触模型扩展到这一设定。
- **贡献**：把 IPC 的无穿透保证带到高阶 / 曲边网格。
- **意义**：高阶 FEM 在工程仿真里是主流，而图形学的鲁棒接触方法多限于线性网格。这篇打通了两者，对 CAD / 工程仿真落地有意义。

### 20. Fast GPU-based Two-way Continuous Collision Handling

- **作者**：Tianyu Wang, Jiong Chen, Dongping Li, Xiaowei Liu, Huamin Wang, Kun Zhou（Zhejiang / Style3D / Inria）
- **问题**：连续碰撞检测与处理（CCD/CCH）是保证无穿透的关键，但传统方法（如 impact zone）代价高、收敛慢，难以 GPU 化。
- **方法**：提出**双向（two-way）**连续碰撞处理：交替执行**前向步（forward step）与后向步（backward step）**，以低代价迭代直至结果**条件收敛（conditionally converged）**。
- **贡献**：GPU 上快速可靠的连续碰撞处理。
- **意义**：布料 / 变形体仿真的核心底层组件。前向-后向交替是一种巧妙的构造，用低成本迭代逼近昂贵的全局无穿透求解。（TOG 42(4)，Making Contact session）

### 21. Sum-of-Squares Collision Detection for Curved Shapes and Paths

- **作者**：Paul Zhang, Zoë Marschner, Justin Solomon, Rasmus Tamstorf（MIT / CMU / Disney Research）
- **问题**：碰撞检测在处理**曲面图元与曲线轨迹**（高阶 primitive）时非常困难——判断两个曲面片在一段曲线运动中是否相交，本质上是一个难以求解的多项式非负性问题。
- **方法**：用**平方和规划（Sum-of-Squares Programming, SOSP）**作为统一框架处理这一大类难题，并提出多项技术**显著降低 SOSP 的求解代价**。
- **贡献**：为高阶图元的碰撞检测提供了统一且可计算的数学工具。
- **意义**：与 High-Order IPC 形成互补——后者解决高阶网格的接触响应，这篇解决高阶图元的检测。两篇合起来标志着图形学接触处理正从「线性 / 直边」向「高阶 / 曲边」迁移。

### 22. Shortest Path to Boundary for Self-Intersecting Meshes

- **作者**：He Chen, Elie Diaz, Cem Yuksel（Utah）
- **问题**：网格发生**自相交**后，如何判断一个内部点应该被「推」向哪个方向来解开纠缠？在自相交存在时，「内部 / 外部」本身是含糊的。
- **方法**：提出高效计算**精确最短边界路径**的方法——给定自相交网格内的一点，求出到边界的精确最短路径。
- **贡献**：在自相交存在的情况下给出了明确、精确的解纠缠方向。
- **意义**：碰撞求解的实用工具。仿真崩溃后的恢复（recovery）、初始状态含穿透的处理，都需要这种「往哪推」的可靠判据。

### 23. P2M: A Fast Solver for Querying Distance from Point to Mesh Surface

- **作者**：Chen Zong, Jiacheng Xu, Jiantao Song, Shuangmin Chen, Shiqing Xin, Wenping Wang, Changhe Tu（Shandong / Texas A&M）
- **问题**：点到网格表面的距离查询是碰撞检测、SDF 构建、物理仿真的基础操作，海量查询下性能瓶颈明显。
- **方法**：提出全新的算法范式 **P2M**，基于 KD-tree 结合「拦截表（interception table）」——预计算空间划分与图元的对应关系，把查询转化为表查找。
- **贡献**：比当前最优求解器快数倍。
- **意义**：底层加速组件，对整个仿真管线有普适的性能收益。

### 24. Constraint-Based Simulation of Passive Suction Cups

- **作者**：Antonin Bernardin, Eulalie Coevoet, Paul G. Kry, Sheldon Andrews, Christian Duriez, Maud Marchal（Inria / McGill / ÉTS）
- **问题**：吸盘（suction cup）的吸附是气压、变形、接触密封三者耦合的复杂现象，此前缺乏物理正确的仿真模型。
- **方法**：建立吸附现象的物理模型，用**基于约束（constraint-based）的formulation**模拟吸盘**内部气压变化**，并与变形和接触密封耦合。
- **贡献**：能模拟吸盘的吸附、保持、脱落全过程。
- **意义**：机器人抓取（尤其真空吸附式末端执行器）的仿真需求明确。属于「把一个具体物理现象做对」的扎实工作。（TOG 42(1)，在 SIGGRAPH 2023 宣讲）

---

## 五、降阶模型与神经加速（与 world model 相关）

### 25. Data-Free Learning of Reduced-Order Kinematics

- **作者**：Nicholas Sharp, Cristian Romero, Alec Jacobson, Etienne Vouga, Paul G. Kry, David I. W. Levin, Justin M. Solomon（NVIDIA / URJC / Toronto / UT Austin / McGill / MIT）
- **问题**：降阶模型（reduced-order model）能极大加速仿真，但主流的数据驱动子空间方法（如 POD、神经自编码器）需要先跑大量全阶仿真收集轨迹数据集——这既昂贵，又使子空间只覆盖训练轨迹附近的配置。
- **方法**：**完全不需要数据集**。用神经网络表示子空间映射（从低维隐向量到完整配置空间），并设计训练目标：直接最小化子空间像集上的**弹性势能**，使得网络学到一个「多样但低能量」的配置子流形。训练只需系统的能量函数，不需要任何轨迹。
- **贡献**：data-free 的降阶运动学学习；对任意系统只需其势能定义即可拟合子空间。
- **意义**：**这篇对 world model / neural simulator 方向最有启发**。它把「学习动力学」这一问题从「拟合数据分布」转向「拟合能量景观」——不学轨迹，而学系统的低能量流形。对做神经物理代理模型（neural surrogate）的人来说，这是一个绕开数据瓶颈、且天然具备泛化性的思路：物理定律（能量）本身就是最强的监督信号。

### 26. Fast Complementary Dynamics via Skinning Eigenmodes

- **作者**：Otman Benchekroun, Jiayi Eris Zhang, Siddhartha Chaudhuri, Eitan Grinspun, Yi Zhou, Alec Jacobson（Toronto / Stanford / Adobe）
- **问题**：**互补动力学（complementary dynamics）**能为骨骼绑定动画自动添加物理正确的次级运动（如肌肉抖动、软组织晃动），但原方法需要全空间仿真，速度慢。
- **方法**：提出降阶空间弹性动力学求解器，在**蒙皮特征模态（skinning eigenmodes）**构成的子空间中计算次级运动。该子空间是**材料敏感（material-sensitive）**且**感知绑定（rig-aware）**的——即它知道骨骼绑定已经表达了哪些运动，只补充「绑定表达不了」的那部分。
- **贡献**：大幅加速互补动力学，使其可用于交互式绑定动画增强。
- **意义**：对角色动画管线有直接价值。「rig-aware 子空间」的构造思想值得注意：子空间不必覆盖所有运动，只需覆盖已有表示的**正交补**。

---

## 六、物理声音合成

### 27. Improved Water Sound Synthesis using Coupled Bubbles

- **作者**：Kangrui Xue, Ryan M. Aronson, Jui-Hsien Wang, Timothy R. Langlois, Doug L. James（Stanford / Adobe）
- **问题**：水声主要由气泡振动产生。已有方法把每个气泡当作独立的谐振子，但真实气泡云中气泡之间存在强**声学耦合**，正是这种耦合产生了气泡云特有的**低频声发射**——独立气泡模型完全无法产生这些低频成分。
- **方法**：提出实用框架，显式建模**气泡间耦合（inter-bubble coupling）**效应来合成水声。
- **贡献**：捕捉到此前缺失的低频声学成分，水声更真实（如倒水、溪流的浑厚感）。
- **意义**：物理声音合成中「耦合 vs 独立」的经典问题。启示是：把 N 个振子的耦合矩阵解出来，往往比把单个振子模型做得更精细更重要。

---

## 七、物理驱动角色与机器人

### 28. Anatomically Detailed Simulation of Human Torso

- **作者**：Seunghwan Lee, Yifeng Jiang, C. Karen Liu（Stanford）
- **问题**：躯干（torso）是人体运动的核心，但已有角色模型往往把躯干简化为少数刚体，既不符合感知真实性，也无法表现躯干的功能性（呼吸、弯曲、扭转的解剖学约束）。
- **方法**：构建解剖学详细的躯干模型，包含**小关节面（facets）、韧带（ligaments）、椎间盘（intervertebral discs）**，通过**耦合高效有限元与刚体仿真**实现——软组织用 FEM，骨骼用刚体，两者耦合。
- **贡献**：在感知真实性与功能性上都达到高保真，同时保持在仿真与最优控制任务中的计算可行性。
- **意义**：对肌肉骨骼仿真、生物力学、以及需要真实躯干变形的角色动画有价值。「计算可行的解剖学精度」是这类工作的核心权衡。

### 29. Bidirectional GaitNet: A Bidirectional Prediction Model of Human Gait and Anatomical Conditions

- **作者**：Jungnam Park, Moon Seok Park, Jehee Lee, Jungdam Won（Seoul National University 等）
- **问题**：人体解剖条件（肌肉无力、骨骼畸形等）与其步态（gait）之间存在因果关系，但这种关系的建模通常只能单向（给定解剖条件预测步态）。
- **方法**：提出生成模型 **Bidirectional GaitNet**，学习人体解剖结构与步态之间的**双向**关系。基础是包含 **304 个 Hill 型肌腱单元**的全身可仿真肌肉骨骼模型。前向网络：解剖条件 → 步态；反向网络：观测步态 → 推断解剖条件。
- **贡献**：既能预测「这种病理会导致什么步态」，也能反推「这种步态说明什么病理」。
- **意义**：临床价值明确（步态分析辅助诊断）。双向建模的思路对任何「参数 → 行为」的仿真系统都有借鉴意义。

### 30. Learning Physically Simulated Tennis Skills from Broadcast Videos

- **作者**：Haotian Zhang, Ye Yuan, Viktor Makoviychuk, Yunrong Guo, Sanja Fidler, Xue Bin Peng, Kayvon Fatahalian（Stanford / NVIDIA / SFU）
- **问题**：物理仿真角色要掌握网球这样的高难度专业技能，需要大量高质量动捕数据——但专业运动员的动捕数据极其稀缺。
- **方法**：从**电视转播视频**中大规模采集网球比赛演示数据（姿态估计 + 球轨迹），据此学习多样的物理仿真网球技能。
- **贡献**：仿真角色能打出多样、真实的网球动作，包括与球的物理交互。
- **意义**：**「用互联网视频替代动捕」的代表作**。对我自己关注的 motion generation 方向直接相关：转播视频虽然噪声大、视角受限，但规模与技能多样性远超实验室动捕。这条路径（视频 → 物理仿真技能）比纯 kinematic motion generation 更有物理保证。

### 31. CALM: Conditional Adversarial Latent Models for Directable Virtual Characters

- **作者**：Chen Tessler, Yoni Kasten, Yunrong Guo, Shie Mannor, Gal Chechik, Xue Bin Peng（NVIDIA / Technion / SFU）
- **问题**：物理仿真角色的控制策略往往要么行为单一，要么虽多样但无法**按用户意图定向（directable）**。
- **方法**：提出 **CALM（Conditional Adversarial Latent Models）**，用条件对抗隐空间模型学习角色的技能表示。训练后隐空间可被高层策略或用户直接操纵，从而在生成多样行为的同时保持可控性。
- **贡献**：兼得行为多样性与可定向性。
- **意义**：与 SA 2023 的 C·ASE 同属「对抗式技能隐空间」路线（同一批作者群）。这条路线是当前物理角色控制的主流范式：低层学技能隐空间，高层做任务规划。

### 32. Synthesizing Physical Character-Scene Interactions

- **作者**：Mohamed Hassan, Yunrong Guo, Tingwu Wang, Michael Black, Sanja Fidler, Xue Bin Peng（NVIDIA / MPI / SFU）
- **问题**：角色与场景的交互（坐下、搬运、攀爬）如果只用运动学方法合成，往往出现穿模、脚滑、受力不合理等问题。
- **方法**：结合**对抗式模仿学习（adversarial imitation learning）**与**强化学习**，训练物理仿真角色完成场景交互任务，动作自然逼真。
- **贡献**：物理正确且自然的角色-场景交互合成。
- **意义**：物理仿真相比纯运动学生成的核心优势就在交互——接触力、平衡、碰撞都是「免费」正确的。

### 33. DOC: Differentiable Optimal Control for Retargeting Motions onto Legged Robots

- **作者**：Ruben Grandia, Farbod Farshidian, Espen Knoop, Christian Schumacher, Marco Hutter, Moritz Bächer（Disney Research / ETH Zürich）
- **问题**：把富有表现力的动画或动物运动**重定向（retarget）**到腿式机器人上很难——机器人有严格的动力学可行性约束（力矩极限、接触力、平衡），直接重定向的动作往往物理上无法执行。
- **方法**：提出**可微最优控制（Differentiable Optimal Control, DOC）**框架。把最优控制问题本身做成可微的，从而可以对「重定向参数」求梯度，联合优化「动作相似度」与「动力学可行性」。
- **贡献**：让动画 / 动物运动能高质量地转移到复杂腿式机器人上。（获 Best in Show）
- **意义**：可微优化与最优控制的结合。对 sim-to-real、机器人动画（迪士尼乐园机器人）有直接应用。

---

## 总体观察与趋势

### 1. IPC 生态的全面扩张
本届最明显的主线是 **Incremental Potential Contact（IPC）**从「单一固体接触」向各方向扩散：
- 向**流体**扩展 → Contact Proxy Splitting（固-液耦合）
- 向**高阶 / 曲边网格**扩展 → High-Order IPC
- 向**厚壳 / 多层结构**扩展 → Multi-Layer Thick Shells
- 向 **GPU 加速**扩展 → Second-order Stencil Descent
- 向**自适应离散化**扩展 → In-Timestep Remeshing

IPC 的「无穿透保证」已成为高质量仿真的事实标准，而 2023 年的主要工作是**在保持这一保证的前提下解决它的性能与适用范围问题**。

### 2. GPU 并行求解器成为标配
Second-order Stencil Descent、Interactive Hair (ADMM)、Fast GPU CCD 三篇分别在超弹性、头发、碰撞三个领域给出 GPU 方案。共同的技术模式是：**把全局耦合的求解拆成大量独立的局部问题**（stencil / local-global / 前向-后向），用迭代换并行。

### 3. 动理学（Kinetic / LBM）方法在流体领域崛起
Fluid-Solid Coupling in Kinetic Two-Phase Flow、Virtual Wind Tunnel 两篇（均有 Desbrun 参与）显示 LBM 类方法正从「快但糙」走向「快且准」，甚至能对标工业 CFD 精度。SIGGRAPH Asia 2023 有更多这一方向的工作（HOME-LBM、Parametric Kinetic Solver）。

### 4. 神经网络的角色：从「替代求解器」到「表示与降阶」
本届的神经方法（Data-Free Reduced-Order Kinematics、Fast Complementary Dynamics）都不是「用网络端到端预测下一帧」，而是**用网络表示子空间 / 流形，物理求解仍由传统数值方法完成**。
特别值得注意的是 **Data-Free Learning of Reduced-Order Kinematics** 的 data-free 范式：用能量函数而非轨迹数据集作为监督信号。这对做 world model / neural physics 的人是重要启示——**物理定律本身是比数据更强、更泛化的监督**。（这一趋势在 SIGGRAPH Asia 2023 的 Neural Physics session 与 Neural Flow Maps 中进一步强化。）

### 5. 物理角色控制：对抗式技能隐空间成为主流范式
CALM、Character-Scene Interactions、Tennis from Broadcast Videos 共享同一套方法论骨架：**对抗式模仿学习构建低层技能隐空间 + 高层策略 / 用户做任务级定向**。数据来源正从实验室动捕转向**互联网视频**（Tennis 一篇），这是可扩展性的关键突破口。

---

## 附：按主题速查

| 主题 | 论文编号 |
|---|---|
| 流体 - 固液耦合 | 1, 4 |
| 流体 - 理论基础 | 2 |
| 流体 - 降阶加速 | 3 |
| 流体 - 湍流 / 工程精度 | 5 |
| 流体 - 水面高度场 | 6 |
| 弹性体 - GPU 求解 | 7 |
| 弹性体 - 模态与非线性分析 | 8, 14 |
| 弹性体 - 几何力学 | 9, 10 |
| 弹塑性 - MPM | 11 |
| 薄壳 / 厚壳 | 12, 13 |
| 布料 / 互锁材料 | 15 |
| 头发 / 弹性杆 | 16, 17 |
| 接触 - IPC 扩展 | 18, 19 |
| 接触 - 碰撞检测 | 20, 21, 22, 23 |
| 接触 - 特定现象 | 24 |
| 降阶 / 神经加速 | 25, 26 |
| 物理声音 | 27 |
| 角色 - 解剖学仿真 | 28, 29 |
| 角色 - RL 控制 | 30, 31, 32 |
| 机器人 - 可微控制 | 33 |

---

*资料来源：ACM SIGGRAPH 2023 Conference Proceedings (TOG 42(4))、ACM TOG Journal Track、physicsbasedanimation.com SIGGRAPH 2023 汇总、Paper Digest SIGGRAPH 2023（212 篇完整清单）交叉核对。*
