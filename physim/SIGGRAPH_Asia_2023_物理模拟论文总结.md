# SIGGRAPH Asia 2023 物理模拟论文逐篇总结

> 会议：ACM SIGGRAPH Asia 2023（第 16 届），2023 年 12 月 12–15 日，澳大利亚悉尼
> 收录范围：技术论文（Conference Papers Track DOI 10.1145/3610548 + TOG Journal Track），筛选**物理模拟 / 物理动画**相关工作
> 覆盖主题：流体（液体、气体、湍流、多相流）、固体与弹塑性、薄壳 / 布料 / 杆、接触与碰撞、颗粒物质、神经物理与降阶模型、可微模拟、材料设计与表征、物理驱动角色
> 共收录 **35 篇**

**本届最重要的信号：出现了独立的 "Neural Physics" 技术论文 session，且最佳论文（Best Paper）颁给了 Fluid Simulation on Neural Flow Maps —— 神经表示与物理求解器的深度融合成为本届主线。这与 world model / neural simulator 方向高度相关。**

---

## 目录

| 分类 | 篇数 | 论文 |
|---|---|---|
| 一、流体模拟（含最佳论文） | 9 | Neural Flow Maps★ / HOME-LBM / Parametric Kinetic Solver / GARM-LS / Peridynamics Multi-fluid / Implicit Mixture Model / DiffFR / Flow Reconstruction / Meandering Rivers |
| 二、神经物理与降阶模型 | 6 | Neural Stress Fields / Neural Collision Fields / Contact Deformations / LiCROM / Subspace Mixed FEM / Neural Metamaterial Networks |
| 三、颗粒物质与弹塑性 | 2 | Power Plastics / Sand-Water Height-field |
| 四、薄壳、布料与杆 | 7 | Kirchhoff-Love Arbitrary Hyperelastic / Stable Discrete Bending / Progressive Shell Quasistatics / Subspace-Preconditioned GPU PD / Second-Order FE for Surfaces / Design Space of Kirchhoff Rods / ClothCombo |
| 五、材料表征与计算设计 | 4 | Systematic Poking / Non-Newtonian ViRheometry / Flexible Planar Microstructures / Sparse Stress Structures |
| 六、自然现象仿真 | 1 | Plant Wilting |
| 七、物理驱动角色与人体 | 6 | MuscleVAE / C·ASE / DROP / Adaptive SRB Tracking / Fatigued Movements / Implicit Physical Face Model |

---

## 一、流体模拟

### 1. ★ Fluid Simulation on Neural Flow Maps（**SIGGRAPH Asia 2023 最佳论文**）

- **作者**：Yitong Deng, Hong-Xing Yu, Diyang Zhang, Jiajun Wu, Bo Zhu（Dartmouth / Stanford / Georgia Tech）
- **问题**：**流图（flow map）**理论上能提供极高的对流精度——只要能精确求出长时程的流图及其 Jacobian，速度场就能被精确重构。但精确计算长时程流图要求**存储数十上百帧的时空速度场**，对 3D 大规模模拟完全不可行（单帧速度场存储都已吃力）。这使得高精度流图方法长期停留在理论上。
- **方法**：核心洞察是**不要把可学习模块塞进已有框架，而是围绕神经网络的新能力重新设计数值框架**。
  1. **物理侧**：采用基于**冲量（impulse）**的流体方程，通过对欧拉方程做**规范变换（gauge transformation）**，建立速度场与流图及其空间导数的关系。
  2. **数值侧**：提出精心设计的**双向行进（bidirectional marching）**算法，以机制对称的方式计算长时程双向流图及其 Jacobian，精度比已有算法高 **3–5 个数量级**。
  3. **表示侧**：提出新型高性能隐式神经表示 **SSNF（Spatially Sparse Neural Fields）**——将小型神经网络与「重叠、多分辨率、空间稀疏」的网格金字塔融合，紧凑且高精度地表示长时程时空速度场。SSNF 在收敛速度、压缩率、存储精度上均优于 Instant-NGP、K-Planes。
- **贡献**：让「精确但不可行」的流图方法变得可行。在 2D 点涡保持实验中平均绝对误差比 bimocq / covector fluids / MC+R 降低 **14–308 倍**；3D 涡环蛙跳（leapfrogging）实验中，物理上两对涡管永不融合，NFM 完成 5 次蛙跳后仍保持分离，而对比方法最多 3 次后完全融合。
- **意义**：**本届最值得细读的一篇，对 world model / neural physics 方向有方法论级的启示。**
  - 与常见的「神经网络端到端预测下一帧」路线截然不同：这里神经网络**只承担一件事——高保真压缩存储中间变量（速度场历史）**，物理演化仍由严格的数值方案完成。因此它既有神经表示的压缩能力，又完整保留了物理正确性与守恒性。
  - 关键设计哲学值得反复琢磨：**不要在已有框架里找地方塞网络，而要问「网络提供了什么新能力，什么数学框架能最大化利用它」**。此前不可行的算法，因为有了神经压缩而变得可行 —— 这是「AI for Science」比「用 AI 拟合数据」更高一层的做法。
  - 注意 INR 的使用方式也很特殊：通常 INR 每个场景训一次，这里却把它当作**模拟过程中不断更新的中间变量**，对性能要求苛刻得多，因此才需要专门设计 SSNF。

### 2. High-Order Moment-Encoded Kinetic Simulation of Turbulent Flows（HOME-LBM）

- **作者**：Wei Li, Tongtong Wang, Zherong Pan, Xifeng Gao, Kui Wu, Mathieu Desbrun（LightSpeed Studios / Inria 等）
- **问题**：动理学（kinetic / 格子玻尔兹曼 LBM）求解器在 GPU 上极快且比传统 Navier-Stokes 求解器更准，但**内存开销巨大**——统计力学的介观离散化要求每个格点存储的变量数比常规流体求解器多一个数量级以上。这挡住了 LBM 在游戏与商用软件（消费级硬件）上的落地。
- **方法**：提出 **HOME-LBM**——**高阶矩编码（High-Order Moment-Encoded）**格子玻尔兹曼求解器。不存储完整的速度分布函数，而只存储每个格点的**少数几个矩（moments）**，在图形学典型的模拟场景下几乎无精度损失。
- **贡献**：内存减少 **3 倍**，速度比当前最优动理学求解器快 **10 倍**，视觉输出几乎一致。
- **意义**：把 LBM 从「工作站专属」推向消费级硬件。「存矩而非存分布」是一个漂亮的压缩思路——只保留物理上真正影响宏观行为的自由度。

### 3. A Parametric Kinetic Solver for Simulating Boundary-Dominated Turbulent Flow Phenomena

- **作者**：Mengyun Liu, Xiaopei Liu（ShanghaiTech）
- **问题**：**边界层（boundary layer）流动**决定了障碍物附近与尾迹的整体流场特征，在高雷诺数湍流下精确处理极薄的边界层非常困难。传统 Navier-Stokes 求解器要靠多分辨率贴体网格（body-fitted mesh）+ 近壁 / 亚格子湍流模型，即使有 GPU 也很慢；换成 LBM 快得多，但边界只能用 cut-cell 方式处理，除非大幅提高分辨率否则牺牲精度。
- **方法**：提出**参数化边界处理模型**，在不过度提高分辨率的前提下大幅改进 cut-cell 边界处理精度。两个组成部分：（1）壁面处针对**非平衡分布函数**的半拉格朗日格式；（2）类比宏观壁面模型（wall modeling）的**纯 link-based 近壁解析介观模型**，计算简单。方法进一步扩展到运动边界。
- **贡献**：定性与定量均通过实验验证；不仅提供更精确的边界处理方式，还提供了**主动控制边界层行为**的手段（这在图形学中此前未有）。
- **意义**：与 HOME-LBM 一起构成本届 LBM 双雄。作者强调其工程实用价值——边界层控制对气动设计有真实需求。

### 4. GARM-LS: A Gradient-Augmented Reference-Map Method for Level-Set Fluid Simulation

- **作者**：Xingqiao Li, Xingyu Ni, Bo Zhu, Bin Wang, Baoquan Chen（Peking University / Georgia Tech / BIGAI）
- **问题**：Level-set 方法追踪流体界面时，**小尺度体积与界面特征（尺度接近网格大小）会迅速耗散消失**——薄膜、小液滴、细丝很难保住。
- **方法**：结合**梯度增强（gradient augmentation）**与**参考映射（reference mapping）**。核心是新的参考映射算法，**同时对流 level-set 值与其梯度**（两者对重建含小尺度体积的动态表面都至关重要）。并配套新的外推（extrapolation）算法与增强的重初始化（reinitialization）流程，构成完整管线。
- **贡献**：能保住接近网格尺度的小体积与界面特征。在雨滴碰撞、合并、飞溅等复杂表面张力流动上验证。
- **意义**：Level-set 保细节问题的扎实推进。「同时输运值与梯度」是提高界面表示阶数的经典手段，这里把它与 reference map 结合得很完整。

### 5. High Density Ratio Multi-fluid Simulation with Peridynamics

- **作者**：Han Yan, Bo Ren（Nankai University）
- **问题**：用基于粒子的离散化模拟**高密度比（high-density-ratio）**多相流（如水-空气、水-汞的混合与分离）极不稳定，是长期难题。
- **方法**：提出**近场动力学（peridynamics）混合模型理论**。借助新的**标量值体积流状态（scalar-valued volume flow states）**，设计粒子离散化方案，以**积分形式**计算多相 Navier-Stokes 方程的所有项（近场动力学的非局部积分形式天然比微分形式稳定）。另设计新的质量更新策略，在高密度比下增强相质量守恒、减少粒子体积波动。
- **贡献**：显著提升高密度比多相流（含混合与分离）的稳定性。
- **意义**：把固体力学领域的 peridynamics 理论迁移到多相流，是一个有意思的跨领域借用——非局部积分形式对处理不连续界面有天然优势。

### 6. An Implicitly Stable Mixture Model for Dynamic Multi-fluid Simulations

- **作者**：Yanrui Xu, Xiaokun Wang, Jiamin Wang, Chongming Song, Tiancheng Wang, Yalan Zhang, Jian Chang, Jian Jun Zhang, Jiri Kosinka, Alexandru Telea, Xiaojuan Ban（USTB / Bournemouth / Groningen / Utrecht）
- **问题**：粒子法混合模型（mixture model）模拟多流体交互时，若用**单一混合流场**表示所有相，**相场的数值不连续**会导致动态效果大量损失、质量与动量守恒不稳定。
- **方法**：提出 SPH 的**隐式混合模型**。不再依赖显式混合场做所有动力学计算与相间传递，而是**从混合模型计算相动量源，导出显式且连续的速度相场**；再通过**相-混合动量映射（phase-mixture momentum-mapping）**机制隐式获得混合场，确保不可压缩性、质量与动量守恒。另提出混合粘性模型，在混合物与各流体相之间建立粘性效应，避免极端惯性条件下的不稳定。
- **贡献**：相比已有混合模型，动态效果更丰富且关键不稳定因素更少，适合长时程、效率优先的 VR 场景。
- **意义**：与上一篇同属「多相流稳定性」主题，但路线不同（隐式混合场 vs peridynamics）。两篇对照阅读能看清这一问题的不同攻法。

### 7. DiffFR: Differentiable SPH-based Fluid-Rigid Coupling for Rigid Body Control

- **作者**：Zhehao Li, Qingyu Xu, Xiaohan Ye, Bo Ren, Ligang Liu（USTC / Nankai）
- **问题**：可微物理模拟在逆向设计中很有效，但要做**双向流-刚体耦合**的可微模拟有两大障碍：（1）流固交互中**无处不在的不连续接触**；（2）流体自由度（DoF）极高，梯度公式的计算代价巨大。
- **方法**：提出可微 SPH 双向流-刚体耦合模拟器，用粒子统一表示流体与固体。但**朴素地对粒子系统前向模拟求导会遇到梯度爆炸**——作者深入分析了这一不稳定性来源，给出可行的梯度计算方案；并提出高效方法，**在不对整个高自由度流体系统求导的前提下**计算流固耦合的梯度。
- **贡献**：在多样的刚体控制任务（含复杂流固交互与多刚体接触）上验证，优化速度比基线方法快达**一个数量级**。
- **意义**：可微流体的实际难点被点得很准——不是「能不能求导」，而是「求出来的梯度是否可用（不爆炸）」以及「代价是否可承受」。做可微物理的人应重点看它对梯度爆炸的分析。

### 8. Real-Time Reconstruction of Fluid Flow under Unknown Disturbance

- **作者**：Kinfung Chu, Jiawei Huang, Hidemasa Takana, Yoshifumi Kitamura（Tohoku University）
- **问题**：从真实液体中只能采集到**稀疏的拉格朗日流动信息**，如何实时重建其详细运动学信息？更难的是，液体常被**形状与运动均未知**的物体扰动。
- **方法**：用**强化学习**训练智能体，在仅以稀疏采集信息为输入、且对扰动源的运动与形状完全无知（oblivious）的情况下，复现目标流动运动学。同时用经典流体力学知识 + 梯度优化来标定 SPH 的**粘性参数**，确保底层仿真模型忠实于物理现实。
- **贡献**：对不同搅动模式均具鲁棒性，与真实世界及仿真 ground truth 定量对比验证。
- **意义**：「稀疏观测 → 完整状态重建」是典型的状态估计 / world model 问题。这里用 RL 做重建策略，思路可迁移。（发表于 TOG 43(1)）

### 9. Authoring and Simulating Meandering Rivers

- **作者**：Axel Paris, Eric Guérin, Pauline Collon, Eric Galin（LIRIS / Université de Lorraine）
- **问题**：**蜿蜒河网（meandering river）**的形态由长期地貌演化决定，如何让美术师交互式地创作并模拟这一过程？
- **方法**：从带初始低分辨率河网（编码为有向图）的地形出发，用**基于物理的迁移方程（migration equation）**加控制项模拟各河道路径的演化。方程中的**曲率相关项**能复现地貌学中已识别的现象，如河湾的下游迁移（downstream migration of bends）；控制项体现地形影响与用户定义的河道轨迹约束。模型还实现了塑造河网的突变事件：形成**牛轭湖（oxbow lake）的截弯取直（cutoff）**与**决口改道（avulsion）**。
- **贡献**：矢量化模型可交互速率运行，支持大规模河网创作；通过弯曲度（sinuosity）与波长指标与真实河流数据定量对比验证。
- **意义**：地貌过程仿真与可控创作的结合。用「物理迁移方程 + 用户控制项」实现可控性，是程序化内容生成里很实用的范式。

---

## 二、神经物理与降阶模型（与 world model 高度相关）

> 本届设有独立的 **Neural Physics** 技术论文 session，是最值得关注的方向。

### 10. Neural Stress Fields for Reduced-order Elastoplasticity and Fracture

- **作者**：Zeshun Zong, Xuan Li, Minchen Li, Maurizio M. Chiaramonte, Wojciech Matusik, Eitan Grinspun, Kevin Carlberg, Chenfanfu Jiang, Peter Yichen Chen（UCLA / CMU / Meta / MIT / Toronto）
- **问题**：弹塑性与断裂的降阶建模远比纯弹性困难。原因在于**塑性（plasticity）导致的强非线性与历史依赖**——塑性变形不可逆，系统状态依赖于整个加载历史，传统的线性子空间（如 POD）无法有效压缩。
- **方法**：提出**神经应力场（Neural Stress Fields）**混合框架：用**隐式神经表示学习 Kirchhoff 应力张量的低维流形**，与降阶的运动学表示配合。关键在于把「塑性的复杂性」封装到应力场的神经表示里，而非试图用线性子空间硬压。
- **贡献**：首个能有效处理弹塑性与断裂的降阶模型框架，实现大幅加速。
- **意义**：对做 neural physics / world model 很有参考价值。它给出的思路是：**当系统有历史依赖的强非线性时，不要压缩位形空间，而要去压缩本构响应（应力）空间**。这与 SIGGRAPH 2023 的 Data-Free Reduced-Order Kinematics（压缩低能量位形流形）形成有趣的互补——两者代表降阶的两种切入点。

### 11. Neural Collision Fields for Triangle Primitives

- **作者**：Ryan S. Zesch, Vismay Modi, Shinjiro Sueda, David I.W. Levin（Texas A&M / Toronto）
- **问题**：碰撞检测与响应本质上是**不连续、不可微**的（接触发生与否是离散事件），这与需要光滑梯度的可微仿真、优化框架天然冲突。
- **方法**：用神经场（neural field）学习三角形图元之间的**碰撞场**，得到光滑可微的碰撞表示。
- **贡献**：为三角网格提供可微的碰撞处理原语。
- **意义**：「把不可微的几何谓词学成光滑场」这一模式很有想象空间——它让接触能进入端到端的梯度优化管线。这对可微物理、基于梯度的运动规划都是有用的构件。

### 12. Learning Contact Deformations with General Collider Descriptors

- **作者**：Cristian Romero, Dan Casas, Maurizio Chiaramonte, Miguel A. Otaduy（URJC Madrid / Meta）
- **问题**：数据驱动地学习接触变形，难点在于**碰撞体（collider）的形状千变万化**——为特定碰撞体训练的模型换个形状就失效，泛化性差。
- **方法**：提出通用的**碰撞体描述子（general collider descriptors）**，把任意碰撞体编码为统一的表示后再输入网络学习接触变形。
- **贡献**：学到的接触变形模型能泛化到未见过的碰撞体形状。
- **意义**：核心贡献是**表示设计而非网络设计**——用一个好的条件编码换取泛化能力。这个教训在 neural surrogate 里普遍适用：泛化瓶颈常在输入表示，而非模型容量。

### 13. LiCROM: Linear-Subspace Continuous Reduced Order Modeling with Neural Fields

- **作者**：Yue Chang, Peter Yichen Chen, Zhecheng Wang, Maurizio M. Chiaramonte, Kevin Carlberg, Eitan Grinspun（Toronto / MIT / Meta）
- **问题**：传统降阶模型（ROM）依赖**固定的离散化**（特定网格）——子空间基向量定义在具体网格顶点上，换网格、换分辨率就要重新训练。
- **方法**：提出 **LiCROM**——用**神经场**表示线性子空间，使降阶模型变为**连续（continuous）**的。子空间基成为空间坐标的连续函数，而非离散顶点上的向量。
- **贡献**：降阶模型与离散化解耦，可用于不同网格、不同分辨率，甚至不同几何。
- **意义**：「连续表示解耦离散化」是神经场在物理仿真中最本质的价值之一。同时它保持了**线性**子空间——因此仍能享受线性 ROM 的理论保证与高效求解，只是把基函数换成了连续表示。这种「保守但扎实」的用法比全非线性自编码器更实用。

### 14. Subspace Mixed Finite Elements for Real-Time Heterogeneous Elastodynamics

- **作者**：Ty Trusty, Otman Benchekroun, Eitan Grinspun, Danny M. Kaufman, David I.W. Levin（Toronto / Adobe）
- **问题**：**异质（heterogeneous）**材料（软硬材料混合，刚度差异巨大）的实时弹性动力学很难——刚度矩阵条件数极差，导致求解收敛慢；而子空间方法在异质材料上往往失效。
- **方法**：结合**混合有限元（mixed FEM，同时以位移与应力 / 应变为未知量）**与**子空间降阶**。混合格式对刚度对比不敏感，子空间提供实时性能。
- **贡献**：异质弹性体的实时仿真。
- **意义**：混合 FEM 处理刚度病态是数值上的正解，与子空间结合是很自然但需要精心设计的组合。对需要软硬材料混合的角色 / 物体（如骨骼+软组织）有价值。

### 15. Neural Metamaterial Networks for Nonlinear Material Design

- **作者**：Yue Li, Stelian Coros, Bernhard Thomaszewski（ETH Zürich）
- **问题**：非线性超材料（metamaterial）的宏观力学建模本身就难，而**反向找出能逼近目标性能的结构参数**更难。关键障碍：有限元网格的**拓扑变化会造成不连续**，使梯度优化失效。
- **方法**：提出**神经超材料网络（NMN）**——用光滑神经表示编码整个超材料族的非线性力学。给定结构参数作为输入，NMN 返回**连续可微的应变能密度函数**，从而**由构造保证力是保守的（conservative forces）**。虽然在仿真数据上训练，但 NMN **不继承有限元网格拓扑变化带来的不连续性**，提供从参数空间到性能空间的光滑全可微映射。在此基础上，把逆向材料设计表述为以神经网络同时作为目标函数与约束的非线性规划问题。
- **贡献**：可自动设计出具有目标应力-应变曲线、指定方向刚度与泊松比曲线的材料。
- **意义**：**神经网络在这里的价值不是「快」，而是「光滑」**——它把一个因离散化而不可微的设计空间，替换成了一个天然可微的代理空间。这个动机（用网络换可微性与光滑性）非常值得记住，与 Neural Collision Fields 的动机一致。另外「输出能量密度而非直接输出力」以保证力场保守，是物理先验融入网络架构的漂亮范例。

---

## 三、颗粒物质与弹塑性

### 16. Power Plastics: A Hybrid Lagrangian/Eulerian Solver for Mesoscale Inelastic Flows

- **作者**：Ziyin Qu, Minchen Li, Yin Yang, Chenfanfu Jiang, Fernando De Goes（Pixar / CMU / Utah / UCLA）
- **问题**：介观尺度（mesoscale）非弹性流（沙、泡沫、气泡）模拟中，粒子分布质量与体积自适应难以兼顾。
- **方法**：提出混合拉格朗日/欧拉方法。核心是把**连续介质力学的更新拉格朗日（updated Lagrangian）时间离散**与**Power Particle-In-Cell** 几何表示结合，得到由优化后的**密度核（density kernels）**描述的物质点，能在空间与时间上精确追踪变化的粒子体积。为实现 CFL 量级步长的高效模拟，还提出受 **X-PBD 启发的非线性 Gauss-Seidel 隐式时间积分**，把欧拉节点速度视为原变量（primal variables）。
- **贡献**：生成高质量、体积自适应的粒子分布；在介观气泡、沙、液体、泡沫上验证。
- **意义**：Pixar 参与，工业质量导向。「Power PIC 几何表示 + 更新拉格朗日」的组合让粒子体积成为可追踪量，解决了 MPM 类方法粒子分布退化的老问题。

### 17. Real-time Height-field Simulation of Sand and Water Mixtures

- **作者**：Haozhe Su, Siyu Zhang, Zherong Pan, Mridul Aanjaneya, Xifeng Gao, Kui Wu（Rutgers / LightSpeed Studios）
- **问题**：沙水混合物（湿沙、泥浆）的行为涉及两相耦合与相变（干沙 / 饱和沙 / 泥流），完整 3D 模拟太慢，无法实时。
- **方法**：用**高度场（height-field）**表示替代完整 3D，设计沙水混合物的实时仿真方案。
- **贡献**：实时性能下表现沙水混合的关键行为。
- **意义**：与 SIGGRAPH 2023 的 Dispersive Shallow Water 呼应——**降维（3D → 高度场）+ 补物理模型**是实时仿真的高性价比路线。对游戏场景（沙滩、泥地）直接可用。

---

## 四、薄壳、布料与杆

### 18. Kirchhoff-Love Shells with Arbitrary Hyperelastic Materials

- **作者**：Jiahao Wen, Jernej Barbič（USC）
- **问题**：Kirchhoff-Love 薄壳广泛使用，但此前**只支持有限的非线性材料选项**。更深的问题是：传统布料仿真把「面内拉伸」与「弯曲」当作**两个独立过程**分别建模，这在物理上是否正确？
- **方法**：从**任意 3D 体超弹性材料**出发，解析地计算其对应的薄壳能量极限，从而推导出该材料的 Kirchhoff-Love 薄壳力学能量。这一过程**显式地识别并分离出面内拉伸项与弯曲项，且无需数值积分（quadrature）**。
- **贡献**：
  1. 理论洞察：**面内拉伸与弯曲源自同一个过程**（薄物体的体弹性），而非传统布料仿真中的两个独立过程。
  2. 支持含**奇次与偶次主伸长（principal stretches）**幂的材料，因此能直接采用此前只用于 3D 实体仿真的标准网格畸变能量，如 **Symmetric ARAP** 与 **Co-rotational**。
  3. 证明了对 Kirchhoff-Love 薄壳，**所有超弹性材料的空间坍缩为二维超弹性材料**——这一观察使得可以构建薄壳力学能量的设计界面，进而创造在大变形下呈现任意刚度曲线的薄壳材料。
- **意义**：**本届固体力学侧最有理论深度的一篇。**「拉伸与弯曲同源」的洞察纠正了布料仿真的一个长期建模习惯；「材料空间坍缩为二维」是一个漂亮且实用的结构性结论（它直接告诉你设计空间到底有多大）。

### 19. Stable Discrete Bending by Analytic Eigensystem and Adaptive Orthotropic Geometric Stiffness

- **作者**：Zhendong Wang, Yin Yang, Huamin Wang（Style3D / Utah）
- **问题**：基于二面角（dihedral angle）的离散弯曲（DAB）模型有两个长期缺陷：（1）能量 **Hessian 不定（indefinite）**，破坏隐式求解器的收敛性；（2）在**退化几何（degenerate geometry）**下不稳定，产生巨大弯曲力。
- **方法**：
  1. 针对不定性：给出 DAB 能量 Hessian **特征系统的解析表达式**。表达式揭示了 DAB 模型的特征值结构——通常有**各 4 个正、负、零特征值**。据此可解析地、高效地把不定 Hessian 投影为半正定。
  2. 针对退化几何：用从解析特征系统算出的**自适应参数**构造**正交各向异性（orthotropic）几何刚度矩阵**，来修正原本不定的几何刚度矩阵。在二面角单元的 12 个运动模态中，所得 Hessian **只保留期望的弯曲模态**——对比之下：带原始几何刚度的精确 Hessian 会保留不期望的「改变高度」模态；不带几何刚度的 Gauss-Newton 近似保留所有模态；用不当几何刚度的投影 Hessian 则一个模态都不保留。
  3. 另建议依 Kirchhoff-Love 薄板理论调整压缩刚度，避免过度压缩。
- **贡献**：同时保证半正定性与退化几何下的稳定性。
- **意义**：**数值细节做到极致的典范。**解析特征系统 + 逐模态分析的做法（Theodore Kim 一脉的风格）能把「为什么会崩」讲得极清楚。做布料 / 薄壳求解器的人应该直接采用这些解析投影公式。

### 20. Progressive Shell Quasistatics for Unstructured Meshes

- **作者**：Jiayi Eris Zhang, Jérémie Dumas, Yun (Raymond) Fei, Alec Jacobson, Doug L. James, Danny M. Kaufman（Stanford / Adobe / Toronto）
- **问题**：薄壳需要高分辨率网格才能捕捉真实力学响应，但高分辨率平衡态求解代价极高；粗网格快但有不可接受的伪影。已有的渐进式仿真（progressive simulation）框架很有前景（在逐级细化的模型层级上给出一致且逐步改进的仿真），但**严重局限于由 Loop 细分生成的网格与形状**。
- **方法**：
  1. 构建 fine-to-coarse 层级，配备为曲面仿真定制的**新型非线性 prolongation 算子**：保持静止形状（rest-shape preserving）、支持复杂曲边界、能从粗层网格重建细节几何。
  2. 提出新的、安全且高效的**保形上采样（shape-preserving upsampling）**方法，在细化过程中确保无相交（non-intersection）与应变限制（strain limits）。
- **贡献**：首次让渐进式仿真具备广泛通用性——支持任意曲壳几何、渐进式碰撞物体、曲边界、非结构三角网格，同时保证预览结果与最终结果都无自相交。在大量压力测试上捕捉了摩擦接触薄壳的起皱、折叠、扭转、屈曲行为，相比直接精细分辨率仿真有**数个数量级加速**。
- **意义**：对设计迭代场景（要快速预览又要最终精确）极有价值。「预览与最终结果一致（consistent）」这一保证是它区别于普通多分辨率方法的关键。

### 21. Subspace-Preconditioned GPU Projective Dynamics with Contact for Cloth Simulation

- **作者**：Xuan Li, Yu Fang, Lei Lan, Huamin Wang, Yin Yang, Minchen Li, Chenfanfu Jiang（UCLA / Utah / Style3D / CMU）
- **问题**：布料仿真中，**子空间积分**收敛快但丢高频细节；**可并行迭代松弛（如 Jacobi）**能抓高频但收敛慢。两者特性截然不同，如何取长补短？
- **方法**：在**投影动力学（Projective Dynamics, PD）**框架下有机耦合两者：用子空间加速 Jacobi-PD 的收敛（求解降阶全局系统并巧妙复用其预计算分解），全空间 Jacobi 迭代捕捉高频细节。与 **IPC** 无缝配合以保证无穿透。摩擦自接触用时间分裂策略处理：PD 求解中用**二次代理（quadratic proxy）**近似接触障碍函数；预分解的子空间系统矩阵用于**降阶空间 LBFGS**——LBFGS 以静止形状的降阶系统矩阵作为初始 Hessian 近似，逐步把接触信息纳入降阶系统。穿透问题通过一个惩罚修正步解决（用 Newton-PCG 最小化不含弹性的增量势能）。
- **贡献**：高分辨率布料仿真上相比现有 GPU 求解器有显著性能提升。
- **意义**：「子空间做预条件」是这篇的精髓——不是用子空间替代全空间求解，而是**用子空间加速全空间迭代的收敛**，从而两全其美。这与 SIGGRAPH 2023 的 Second-order Stencil Descent 同属 GPU 加速 IPC 的努力。

### 22. Second-Order Finite Elements for Deformable Surfaces

- **作者**：Qiqin Le, Yitong Deng, Jiamu Bu, Bo Zhu, Tao Du（Tsinghua / Dartmouth / Georgia Tech / Shanghai Qi Zhi）
- **问题**：可变形曲面（布料、薄壳）仿真几乎都用线性（一阶）三角单元，需要很密的网格才能表现光滑弯曲。
- **方法**：为可变形曲面构建**二阶有限元**格式。
- **贡献**：用更少的单元获得更高的精度与更光滑的变形。
- **意义**：与 SIGGRAPH 2023 的 High-Order IPC 同属「图形学仿真走向高阶」的趋势。高阶元在工程仿真是常规操作，图形学正在补上这一课。

### 23. The Design Space of Kirchhoff Rods

- **作者**：Christian Hafner, Bernd Bickel（ISTA）
- **问题**：Kirchhoff 杆模型已被广泛研究用于**正向预测**（给定几何与边界条件，预测如何变形）。但**逆问题**——设计一根直杆的几何，使其两端固定到支撑结构后**自动变形成目标曲线形状**——如何求解？
- **方法**：通过沿杆长改变**横截面轮廓（cross-sectional profiles）**来精细控制杆的静态平衡态。关键理论贡献：证明物理可实现的平衡态集合可以用**线性线丛（linear line complexes）**给出简洁的几何描述——这一刻画直接导出了非常高效的计算设计算法。
- **贡献**：实现为交互式软件工具，可实时把 3D 手绘样条曲线转换为弹性杆，并即时反馈设计的可行性与实用性。制作了若干物理原型，应用于室内设计与软体机器人。
- **意义**：**「可实现集合的几何刻画」是这篇最漂亮的地方**——一旦知道可达平衡态构成什么几何对象，逆设计就从盲目优化变成了投影问题，效率天差地别。这种「先搞清设计空间的结构，再设计算法」的思路很值得学习。

### 24. ClothCombo: Modeling Inter-Cloth Interaction for Draping Multi-Layered Clothes

- **作者**：Dohae Lee, Hyun Kang, In-Kwon Lee（Yonsei University）
- **问题**：学习式布料垂坠（draping）方法已有不错结果，但**多层衣物**仍然困难——建模**层间交互（inter-cloth interaction）**很不容易。
- **方法**：三阶段流程。（1）用**拓扑无关（topology-agnostic）**网络为每件衣物创建特征嵌入；（2）垂坠网络将所有衣物变形以适配目标体型与姿态（暂不考虑层间交互）；（3）**解缠（untangling）网络**预测逐顶点位移，解决衣物之间的互穿。层间交互用 **GNN** 高效建模。
- **贡献**：复杂多层场景下表现强；因对布料拓扑无关，可直接用于真实衣物在多样姿态与组合下的分层虚拟试衣。
- **意义**：与 SIGGRAPH 2023 的 Multi-Layer Thick Shells 形成对照——同一个「多层衣物」问题，一边走物理仿真路线（厚壳 FEM），一边走学习路线（GNN + 解缠）。两篇对比能看清两条路线的取舍：物理路线保证正确性，学习路线保证速度。

---

## 五、材料表征与计算设计

### 25. Capturing Animation-Ready Isotropic Materials Using Systematic Poking

- **作者**：Huanyu Chen, Danyong Zhao, Jernej Barbič（USC）
- **问题**：如何**非侵入式、原位、低成本**地测量真实弹性固体（人造材料与生物组织）的非线性弹性能量密度函数？
- **方法**：用已知半径的 **3D 打印刚性圆柱**去「戳（poke）」弹性物体，用精密力计记录接触力随压入深度的变化（用力计支架，或新颖的无约束激光装置测量压入深度）。用 FEM 建模 3D 弹性固体，弹性能量用**可压缩 Valanis-Landel 材料**（它推广了 Neo-Hookean，允许大变形下任意的拉伸行为）。然后用优化拟合非线性各向同性弹性能量，使 FEM 的接触力与压入量匹配真实测量值。
  - 关键技术点：（a）用精心设计的**三次样条**，使材料在大范围伸长下都准确、且对反转（inversion）鲁棒，因而对图形学应用是「**animation-ready**」的；（b）利用**径向对称性**把 3D 弹性静力接触问题转化为数学等价的 2D 问题，极大加速优化；（c）观察到**体积可压缩性可通过用不同半径的刚性圆柱戳压来估计**，从而避免使用光学相机，大幅简化实验。
  - 另外还显著改进了基于伸长（stretch-based）弹性材料的理论与鲁棒性，给出计算切线刚度矩阵的简洁公式，含严格证明与奇异性处理。
- **贡献**：用全 3D 仿真验证优化出的材料能匹配真实力、压入量与真实变形 3D 形状；并用 "Shore 00" 硬度计（标准材料硬度测量设备）交叉验证。
- **意义**：**实验设计极其巧妙**。「用不同半径圆柱戳压来推断可压缩性」这个观察，把一个需要光学设备的测量变成了纯力学测量。做真实材料参数标定的人必读。

### 26. Non-Newtonian ViRheometry via Similarity Analysis

- **作者**：Mitsuki Hamamichi, Kentaro Nagasawa, Masato Okada, Ryohei Seto, Yonghao Yue（Aoyama Gakuin / Wenzhou Institute）
- **问题**：如何估计剪切依赖的类流体材料（可能含**大尺度夹杂物**，如含颗粒的酱料）的三个 **Herschel-Bulkley 参数**（屈服应力 σY、幂律指数 n、稠度参数 η）？这类材料**流变仪（rheometer）往往无法给出有效测量**（大颗粒会卡住仪器）。
- **方法**：用待测材料做**溃坝（dam-break / 柱体坍塌）**实验并录像，再用仿真优化材料参数。为更好匹配流变仪中的简单剪切流，修改了弹-粘塑性 Herschel-Bulkley 模型的流动法则。
  - **关键发现**：分析优化的 loss landscape 时发现存在一个**相似性关系（similarity relation）**——在该关系内相距很远的材料参数会产生匹配的仿真结果，使参数难以区分（即参数不可辨识）。
  - **解决**：利用相似性关系对实验装置（setup）的依赖性——通过分析相似性关系的 **Hessian**，提出用**多个实验装置**联合估计来改善辨识度。
- **贡献**：与流变仪测量对比验证（对无大颗粒材料），并展示了对含大尺度夹杂物材料的应用，包括各种沙拉 / 意面酱料和粥。
- **意义**：**对参数辨识性（identifiability）的分析是这篇的真正价值。**「loss 很低不等于参数正确」——存在一整族参数给出相同观测。这个陷阱在所有基于仿真的参数标定 / 系统辨识工作中都存在，而通过分析简并方向来主动设计多组互补实验，是正确的解法。做逆问题、做 sim-to-real 标定的人应重点看这一点。

### 27. Computational Design of Flexible Planar Microstructures

- **作者**：Zhan Zhang, Christopher Brandt, Jean Jouve, Yue Wang, Tian Chen, Mark Pauly, Julian Panetta（EPFL / UC Davis / Houston）
- **问题**：机械超材料通过改变微结构来定制宏观弹性性质。**小位移区间**的能力已被充分研究（线弹性周期均质化可导出高效的最优设计算法），但许多应用涉及**大变形柔性结构**，此时几何非线性使线弹性精度迅速崩坏。有限应变下的微结构设计计算量剧增，此前**没有任何计算工具能设计在有限应变区间内模拟目标超弹性本构律的超材料**。
- **方法**：
  1. **非线性均质化加速**：高效构造微结构变形在「超材料可能经历的有限宏观应变空间」上的准确**插值器（interpolant）**。有了插值器，任意应变下的均质化能量密度、应力、切线弹性张量都可低成本计算。
  2. **设计工具**：用参数化形状优化，把有效材料性质拟合到目标本构律（在应变空间的一个区域上），产出可直接制造的几何。
- **贡献**：系统性测试——设计了一整套尽可能拟合各向同性 Hooke 定律的材料目录；通过制造并测试物理原型，证明相比传统线性超材料设计技术精度显著提升。
- **意义**：把超材料设计从小变形推到有限变形，是实用性的关键跨越（柔性器件本来就工作在大变形区）。「预构造应变空间上的插值器」是应对非线性均质化昂贵代价的正解。

### 28. Sparse Stress Structures from Optimal Geometric Measures

- **作者**：Dylan Rowe, Albert Chern（UC San Diego）
- **问题**：给定载荷与约束，找出最优结构设计是拓扑优化与形状优化的核心挑战。
- **方法**：提出全新视角——把问题转化为寻找与给定载荷力平衡的**最小张拉整体结构（minimal tensegrity structure）**（由缆绳 cables 与撑杆 struts 构成的网络）。通过应用**几何测度论（geometric measure theory）**与**压缩感知（compressive sensing）**技术，证明这个看似困难的图论问题可以化归为数值上可处理的**连续优化问题**。
- **贡献**：用只涉及**快速傅里叶变换与局部代数计算**的轻量迭代算法，即可生成稀疏支撑结构，呈现精细的分支、拱形与加固结构，且尊重给定的载荷力与障碍物。
- **意义**：**把离散图论问题化归为连续优化**是这篇的核心贡献，也是 Albert Chern 一贯的风格（用深刻的数学工具让难问题变简单）。生成结果在美学与力学上都很有说服力。

---

## 六、自然现象仿真

### 29. A Physically-inspired Approach to the Simulation of Plant Wilting

- **作者**：Filippo Maggioli, Jonathan Klein, Torsten Hädrich, Emanuele Rodolà, Wojtek Pałubicki, Sören Pirk, Dominik L. Michels（Sapienza / KAUST / Kiel / Poznań）
- **问题**：植物**枯萎（wilting）**是失水导致的力学软化过程，视觉上很有辨识度，但缺乏物理驱动的仿真方法。
- **方法**：受物理启发的方法，把植物组织的含水量变化与其力学刚度联系起来，从而驱动枝叶的下垂与卷曲。
- **贡献**：能表现从新鲜到枯萎的连续过程。
- **意义**：程序化植物动画的一个具体现象攻关。属于「把一个视觉上重要的自然现象的物理机制搞对」的工作。

---

## 七、物理驱动角色与人体

### 30. MuscleVAE: Model-Based Controllers of Muscle-Actuated Characters

- **作者**：Yusen Feng, Xiyan Xu, Libin Liu（Peking University）
- **问题**：生成生物力学上合理的**肌肉驱动（muscle-actuated）**角色运动，难点有二：（1）肌肉骨骼系统自由度极高，控制困难；（2）缺乏对**疲劳（fatigue）**的建模——而疲劳积累导致的运动风格自然演化是长时间活动真实感的关键。
- **方法**：
  1. 把**疲劳动力学模型 3CC-r** 融入广泛采用的 **Hill 型肌肉模型**，模拟肌肉疲劳的产生与恢复。
  2. 针对高自由度控制难题，提出基于 PD 控制的新颖**肌肉空间（muscle-space）控制策略**。
  3. 在此仿真与控制框架上训练生成模型 **MuscleVAE**：用 VAE 从大规模无结构运动数据集学习丰富灵活的技能隐表示，该表示**不仅编码运动特征，还编码肌肉控制与疲劳属性**。用基于模型（model-based）的方法高效训练。
- **贡献**：产出高保真运动，支持多种下游任务。
- **意义**：把疲劳纳入隐空间是很有新意的一点——它让「同一个动作在疲劳不同阶段有不同表现」成为可生成的变化维度。对我关注的 motion generation 方向，这提示了一个思路：**除了动作语义，生理状态（疲劳、发力）也可以是隐空间的可控轴**。

### 31. C·ASE: Learning Conditional Adversarial Skill Embeddings for Physics-based Characters

- **作者**：Zhiyang Dou, Xuelin Chen, Qingnan Fan, Taku Komura, Wenping Wang（HKU / Tencent AI Lab / Texas A&M）
- **问题**：物理仿真角色学习多样技能库时，如何同时获得**可控性**（能直接指定要执行哪个技能）？
- **方法**：把异质（heterogeneous）技能运动划分为若干**同质（homogeneous）**子集，训练**低层条件模型**学习条件行为分布。这种技能条件化的模仿学习天然在训练后提供对角色技能的显式控制。训练过程包含三项技术以平衡不同复杂度的技能：**focal skill sampling**（聚焦采样）、**skeletal residual forces**（骨骼残差力，缓解动力学不匹配以掌握敏捷动作）、**element-wise feature masking**（逐元素特征掩码，捕捉更一般的行为特征）。
- **贡献**：产出高度多样且真实的技能，优于当前最优模型，可复用于多种下游任务。显式的技能控制句柄允许高层策略或用户以期望的技能规格指挥角色，对交互式角色动画很有优势。
- **意义**：与 SIGGRAPH 2023 的 CALM 是同一路线的并行工作（条件对抗技能隐空间）。**「把异质技能集划分为同质子集」是让条件化真正生效的关键工程手段**——直接在混杂数据上做条件模型往往学不出清晰的技能边界。这一点对做 motion generation 的数据组织有直接参考价值。

### 32. DROP: Dynamics Responses from Human Motion Prior and Projective Dynamics

- **作者**：Yifeng Jiang, Jungdam Won, Yuting Ye, C. Karen Liu（Stanford / SNU / Meta）
- **问题**：**运动学（kinematics）生成模型**在建模海量运动数据上扩展性极好，但**没有与物理交互与推理的接口**（无法响应外力扰动）；而**仿真器在环（simulator-in-the-loop）**的学习方法物理真实度高，但训练困难，影响扩展性与采用率。如何两者兼得？
- **方法**：提出 **DROP** —— 可视为一个高度稳定、极简的物理人体仿真器，与运动学生成先验对接。利用**投影动力学（projective dynamics）**，把学到的运动先验**作为一个投影能量项（one of the projective energies）**灵活而简单地集成进去，从而将运动先验提供的控制与牛顿动力学无缝融合。作为**模型无关（model-agnostic）的插件**，DROP 能充分利用生成运动模型的最新进展来做物理运动合成。
- **贡献**：在不同运动任务与各种物理扰动下广泛评估，展示了响应的可扩展性与多样性。
- **意义**：**这是我认为对 motion generation 方向最有直接借鉴价值的一篇。**它给出了一个优雅的答案：不必在「kinematic 生成模型」与「物理仿真」之间二选一，而是把生成先验**降格为投影动力学里的一个能量项**，与其他物理能量（惯性、弹性、接触）平等地参与求解。这样物理约束天然满足，而生成模型的表达力完整保留，且可随时替换为更强的先验（model-agnostic）。对任何想给 motion 生成模型加物理性的工作，这是一个应当认真对标的框架。

### 33. Adaptive Tracking of a Single-Rigid-Body Character in Various Environments

- **作者**：Taesoo Kwon, Taehong Gu, Jaewon Ahn, Yoonsang Lee（Hanyang University）
- **问题**：DeepMimic 之后的研究主要在扩展仿真运动的技能库，但训练成本高，且策略对**未见过的环境变化**适应性差。
- **方法**：换一条路——基于**单刚体（single-rigid-body, SRB）**角色仿真的深度强化学习。用**质心动力学模型（centroidal dynamics model, CDM）**把全身角色表达为单个刚体，训练策略去跟踪参考运动。SRB 仿真表述为**二次规划（QP）**问题，策略输出让 SRB 角色跟随参考运动的动作。最终全身运动基于仿真 SRB 角色的状态、以物理上合理的方式运动学生成。
- **贡献**：由于状态与动作空间维度大幅降低，学习**样本效率极高**——在超便携笔记本上 **30 分钟**即可训练完成。所得策略无需任何额外学习即可应对训练中未经历的环境（如在不平地形上奔跑、推箱子）以及已学策略之间的转换。
- **意义**：**「降维换泛化与效率」的极佳示范。**把全身简化为单刚体看似激进，但质心动力学恰恰捕捉了平衡与接触的本质，而细节姿态可以事后运动学补全。30 分钟笔记本训练 vs 动辄数天的 GPU 训练，这个对比很有说服力——**很多时候瓶颈不在算力，而在问题表述（problem formulation）**。

### 34. Discovering Fatigued Movements for Virtual Character Animation

- **作者**：Noshaba Cheema, Rui Xu, Nam Hee Kim, Perttu Hämäläinen, Vladislav Golyanik, Marc Habermann, Christian Theobalt, Philipp Slusallek（MPI / DFKI / Aalto）
- **问题**：交互式仿真「会疲劳」的角色对动画真实感不可或缺，但**采集这类数据极其困难**——让演员反复做后空翻直到力竭，成本高且有受伤风险。因此忠实的疲劳建模研究极少。
- **方法**：基于深度强化学习的方法，**文献中首次**为全身物理仿真智能体生成**感知累积疲劳（cumulative fatigue）**的控制策略。两步：（1）用 **GAIL（生成对抗模仿学习）**学习该技能的专家策略；（2）学习疲劳策略——依据**耐力时间（endurance time）**，用**三舱室控制器（Three-Compartment Controller, 3CC）**模型，把生成的恒定力矩边界限制为关节驱动空间中**非线性、状态与时间依赖**的极限。
- **贡献**：智能体展现出随疲劳积累而变化的运动策略。
- **意义**：与 MuscleVAE 同样引入 3CC 类疲劳模型，但路线不同（RL 力矩边界约束 vs VAE 隐空间编码）。**核心价值是「用物理 / 生理模型替代无法采集的数据」**——疲劳数据采不到，但疲劳的生理机制是已知的，把机制写进约束即可让智能体自己「发现」疲劳动作。这个思路对所有数据稀缺的动作类别都适用。

### 35. An Implicit Physical Face Model Driven by Expression and Style

- **作者**：Lingchen Yang, Gaspard Zoss, Prashanth Chandran, Paulo Gotardo, Markus Gross, Barbara Solenthaler, Eftychios Sifakis, Derek Bradley（ETH Zürich / DisneyResearch|Studios / Wisconsin）
- **问题**：3D 面部动画通常靠操纵由**表情控制参数**驱动的面部变形模型（rig）。一个常被忽视的关键成分是表情的**「风格（style）」**——即某个表情是「怎么做出来的」。虽然定义表情的语义基是常见做法，但每个角色都以自己的风格执行每个表情。此前风格与表情**纠缠（entangled）**在一起，无法把一个角色的风格迁移到另一个角色。
- **方法**：提出基于**数据驱动隐式神经物理模型**的新面部模型，可由**表情与风格分别独立驱动**。核心是一个框架，能**同时为多个受试者学习隐式的、基于物理的驱动（actuations）**，仅需来自少量身份的若干任意表演捕捉序列即可训练。训练后支持对任一训练身份做泛化的物理驱动面部动画，并能外推到未见过的表演。
- **贡献**：（1）表情与风格解耦，支持角色间的**风格迁移**与不同角色风格的**混合**；（2）作为物理模型，能合成**碰撞处理**等物理效应——这是传统方法做不到的。
- **意义**：**「隐式神经物理驱动」是很值得注意的建模方式**——不直接学变形，而是学物理驱动量（actuation），再由物理模型产生变形。这样物理效应（碰撞、体积守恒）自动获得，且驱动量比变形更接近「意图」，因此解耦（表情 vs 风格）更自然。对做面部 / 多模态动作生成的工作有参考价值：**在驱动层而非输出层做解耦。**

---

## 总体观察与趋势

### 1. Neural Physics 正式成为独立方向，且范式已经收敛
本届设有独立的 **Neural Physics** session，最佳论文也颁给了 Neural Flow Maps。更重要的是，**神经网络在物理模拟中的角色已经清晰下来了**，而且**不是**「端到端预测下一帧」。归纳本届 6 篇神经物理工作，网络承担的是以下四类角色：

| 网络的角色 | 代表论文 | 为什么用网络 |
|---|---|---|
| **高保真压缩存储**中间物理量 | Neural Flow Maps (SSNF)、Neural Stress Fields | 让「精确但存不下」的算法变得可行 |
| **提供光滑性 / 可微性** | Neural Metamaterial Networks、Neural Collision Fields | 绕开离散化 / 接触带来的不连续，进入梯度优化管线 |
| **解耦离散化**（连续表示） | LiCROM | 降阶模型不再绑定特定网格 |
| **泛化条件编码** | Contact Deformations (collider descriptors) | 让学到的模型泛化到未见几何 |

**共同点：物理演化的正确性始终由传统数值方案保证，网络只解决「表示」层面的问题。** 这对做 world model / neural simulator 的人是重要参照——图形学社区已经用大量实践表明，把网络放在「表示」而非「演化」的位置上，能同时拿到神经方法的好处与物理正确性。

Neural Flow Maps 的方法论最值得吸收：**不要在已有框架里塞网络，而要问「网络提供了什么新能力，什么数学框架能最大化它」**。

### 2. 动理学（Kinetic / LBM）方法在流体领域全面开花
HOME-LBM（内存降 3 倍、速度快 10 倍）与 Parametric Kinetic Solver（边界层处理）两篇，加上 SIGGRAPH 2023 的两篇（Kinetic Two-Phase Coupling、Virtual Wind Tunnel），LBM 已成为图形学流体的主流路线之一。2023 年的攻关重点是它的两个短板：**内存开销**与**边界处理精度**，且两者都取得了实质突破。

### 3. 多相流稳定性成为集中攻关点
Peridynamics 高密度比多相流、Implicitly Stable Mixture Model 两篇从完全不同的数学路线（非局部积分形式 vs 隐式混合场 + 动量映射）攻同一个问题。说明「高密度比多相流不稳定」是当前公认的硬骨头。

### 4. 参数辨识与真实材料表征受到重视，且开始关注「可辨识性」
Systematic Poking（戳压测弹性）、Non-Newtonian ViRheometry（溃坝测流变）两篇都在做「从真实观测反求材料参数」。
**特别值得注意的是 ViRheometry 对 similarity relation 的分析**——它指出存在一整族参数产生相同观测，因此「loss 低」不等于「参数对」，并通过分析简并方向来设计多组互补实验。**这个警示对所有基于仿真的参数标定、系统辨识、sim-to-real 工作都适用**，是本届方法论层面最有价值的洞察之一。

### 5. 计算设计走向大变形 / 非线性
Flexible Planar Microstructures（有限应变超材料）、Neural Metamaterial Networks（非线性材料设计）、Design Space of Kirchhoff Rods（杆的逆设计）、Sparse Stress Structures（张拉整体拓扑优化）四篇共同显示：计算设计正从线弹性小变形走向**非线性大变形**。
共同的方法论主题是「**先搞清设计空间 / 可达集合的结构，再设计算法**」——Kirchhoff Rods 用线性线丛刻画可达平衡态，Sparse Stress 用几何测度论把图论问题连续化，Flexible Microstructures 预构造应变空间插值器。

### 6. 物理角色控制：从「加技能」转向「加物理与生理真实性」
本届角色类工作有明显的转向。SIGGRAPH 2023 侧的主题还是「学更多技能、更可控」（CALM、Character-Scene、Tennis），而 SIGGRAPH Asia 2023 侧则明显在补**物理与生理的真实性**：
- **疲劳**：MuscleVAE（编码进隐空间）、Fatigued Movements（编码进 RL 力矩约束）—— 两篇都用 3CC 生理模型，且都是**用已知机制替代无法采集的数据**
- **物理响应性**：DROP（把生成先验降格为投影动力学能量项）
- **降维求解**：Adaptive SRB Tracking（质心动力学 + QP，30 分钟笔记本训练）
- **驱动层解耦**：Implicit Physical Face Model（学 actuation 而非 deformation）

对 motion generation 方向，**DROP 与 Adaptive SRB Tracking 两篇最值得对标**：前者给出了「生成先验 + 物理求解」的优雅融合框架，后者证明了好的问题表述比算力更重要。

---

## 附：按主题速查

| 主题 | 论文编号 |
|---|---|
| 流体 - 神经表示（最佳论文） | 1 |
| 流体 - 动理学 / LBM | 2, 3 |
| 流体 - 界面追踪 | 4 |
| 流体 - 多相流稳定性 | 5, 6 |
| 流体 - 可微 / 逆问题 | 7, 8 |
| 流体 - 地貌 / 河流 | 9 |
| 神经物理 - 降阶 | 10, 13, 14 |
| 神经物理 - 接触 | 11, 12 |
| 神经物理 - 材料设计 | 15 |
| 颗粒 / 弹塑性 | 16, 17 |
| 薄壳 - 理论与本构 | 18, 22 |
| 薄壳 - 数值稳定性 | 19 |
| 薄壳 - 多分辨率 | 20 |
| 布料 - GPU 求解 | 21 |
| 杆 - 逆设计 | 23 |
| 布料 - 学习式多层 | 24 |
| 材料表征 / 参数辨识 | 25, 26 |
| 计算设计 - 超材料与结构 | 27, 28 |
| 自然现象 | 29 |
| 角色 - 肌肉与疲劳 | 30, 34 |
| 角色 - 技能隐空间 | 31 |
| 角色 - 物理与生成先验融合 | 32 |
| 角色 - 降维控制 | 33 |
| 面部 - 隐式物理 | 35 |

---

## 与 SIGGRAPH 2023（北美）的对照

| 维度 | SIGGRAPH 2023 | SIGGRAPH Asia 2023 |
|---|---|---|
| **主导线索** | IPC 生态全面扩张（向流体、高阶网格、厚壳、GPU、自适应离散化扩散） | Neural Physics 崛起（独立 session + 最佳论文） |
| **流体重点** | 理论补全（Fluid Cohomology）、固液耦合、工程精度（风洞） | 神经流图、动理学内存 / 边界优化、多相流稳定性 |
| **薄壳重点** | 厚壳与多层（Multi-Layer Thick Shells）、褶皱表示 | 本构理论（拉伸弯曲同源）、数值稳定性（解析特征系统）、多分辨率一致性 |
| **神经方法定位** | 表示子空间（Data-Free ROM、Skinning Eigenmodes） | 压缩中间量 / 提供可微性 / 解耦离散化（4 类角色已清晰） |
| **角色控制** | 扩展技能库与可控性（CALM、Tennis、Character-Scene） | 补物理与生理真实性（疲劳、投影动力学融合、降维） |
| **计算设计** | 多稳态分析（Compliant Modes、Elastic Knots） | 大变形超材料、可达集合的几何刻画 |

**若只精读三篇**，我的建议是：
1. **Fluid Simulation on Neural Flow Maps**（#1）—— 神经表示与物理求解融合的方法论范本
2. **DROP**（#32）—— 生成运动先验与物理仿真的优雅融合框架
3. **Non-Newtonian ViRheometry**（#26）—— 参数可辨识性分析，对任何 sim-to-real 标定工作都是必要警示

---

*资料来源：SIGGRAPH Asia 2023 Conference Papers 官方 TOC（DOI 10.1145/3610548，122 篇完整摘要）、ACM TOG 42(6) 期刊轨、physicsbasedanimation.com SIGGRAPH Asia 2023 汇总、SIGGRAPH Asia 2023 Full Program 会话安排交叉核对。*
