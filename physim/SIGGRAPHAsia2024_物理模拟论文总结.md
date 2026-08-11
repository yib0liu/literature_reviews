# SIGGRAPH Asia 2024 物理模拟论文逐篇总结

> **报告范围**：SIGGRAPH Asia 2024 Technical Papers中与物理模拟相关的论文，涵盖流体、固体、弹性体、布料、碰撞检测、微结构优化等方向。
> **数据来源**：physicsbasedanimation.com 官方收录列表 + 各论文项目页/arXiv/ACM DL。
> **可靠性标注**：【摘要原文】来自论文原始摘要；【项目页描述】来自作者公开的项目页面；【二手转述】来自主流学术报道或综述；【推断】基于方法描述的合理推断。

---

## 一、总体概览与主题脉络

SIGGRAPH Asia 2024共收录约**25篇**物理模拟相关论文（按physicsbasedanimation.com收录口径）。从主题分布来看，呈现以下趋势：

### 主题脉络

1. **流图（Flow Map）方法的完整体系化** — Georgia Tech Bo Zhu团队在Asia 2024有**4篇**论文（Particle-Laden Fluid on Flow Maps⭐Best Paper、Eulerian Vortex Method on Flow Maps🔒Replicability Stamp、Solid-Fluid Interaction on PFM、Impulse Ghost Fluid for Two-Phase Flows），加上SIGGRAPH 2024的2篇，形成了一个从理想流体→粘性流体→颗粒载荷流→固液耦合→两相流的完整技术链条。这是近年来图形学物理模拟领域最成体系的系列工作之一。

2. **GPU加速接触求解器的突破** — Barrier-Augmented Lagrangian (BAL) 将原本需要128GB CPU内存的IPC接触问题降至8GB GPU显存，实现了数十至近百倍加速。配合Cubic Barrier（ppf-contact-solver，支持180M+接触）和Non-distance Barriers（百万自由度布料交互式模拟），GPU时代的接触求解器正在发生范式转变。

3. **降阶与子空间方法的多元化发展** — Volumetric Homogenization（体均质化针织物）、Trading Spaces（自适应子空间时间积分）、Lipschitz Neural Subspace（神经子空间加速）三篇论文分别从均质化、自适应策略、学习优化三个角度推进降阶模拟。

4. **微结构设计与优化的可微分方法成熟** — Optimized Shock-protecting Microstructures（抗冲击微结构）、Polar Interpolants（薄壳均匀化）、Q3T Prisms（弹塑性壳单元）等论文展示了"可微分仿真+形状优化+3D打印验证"的完整闭环，标志着计算材料设计进入实用阶段。

5. **开源与可复现性文化加强** — 超过半数论文公开了代码，Eulerian Vortex Method甚至获得了ACM Replicability Stamp（可复现性印章），这在图形学领域仍属罕见。

### World Model 相关发现

与SIGGRAPH NA 2024及之前年份的观察一致，**World Model概念在Asia 2024主会论文中同样未出现**。Neural Implicit Reduced Fluid Simulation (NIRFS) 是最接近的工作——它在降维潜空间中用Neural ODE建模流体动力学，但每个场景需单独训练，泛化能力有限，与"通用世界模型"的愿景仍有距离。

---

## 二、分类论文详表

### 2.1 流体模拟（8篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|------|-----------|---------|---------|----------|------|--------|
| 1 | **Particle-Laden Fluid on Flow Maps** ⭐ Best Paper Award | Duowen Chen, Zhiqi Li et al. / Georgia Tech, Dartmouth (*同等贡献) | 流映射技术难以处理耗散力（粘性/阻力），只能求解Euler方程 | 双粒子系统耦合：物理沉降颗粒+虚拟流映射颗粒通过Poisson系统在背景网格上耦合；路径积分公式将粘性和阻力纳入粒子流映射 | 墨水扩散SOTA：悬浮液滴尾部膨胀断裂、环面形成解体、沉降液滴合并；无需人工涡旋分量 | 依赖特定初始条件；极端粘度比/高雷诺数待验证 | 【摘要原文】 |
| 2 | **Fluid Implicit Particles on Coadjoint Orbits (CO-FLIP)** 🏅 Honorable Mention | Mohammad Sina Nabizadeh, Ritoban Roy-Chowdhury et al. / UCSD | 混合欧拉-拉格朗日框架下如何保持能量/环量/Casimir不变量 | 从不可压Euler Hamiltonian表述出发；局部显式高阶无散插值构造修正离散Hamiltonian；几何时间积分保持能量和环量守恒（流动在余伴随轨道上）；压力投影弱意义精确 | 圣海伦火山喷发碎屑云(96×192×96低分辨率)呈现精细涡旋；标准基准测试卓越稳定性 | 主要针对理想流体（无粘）；理论框架复杂实现难度高 | 【摘要原文】 |
| 3 | **An Eulerian Vortex Method on Flow Maps** 🔒 Replicability Stamp | Sinan Wang et al. / Georgia Tech, Stanford, HKU, Purdue, U Michigan | 如何在流映射框架下发展基于涡量的欧拉方法替代冲量方法 | 线元流映射输运方程+双向推进格式(bi-directional marching)实现高精度涡量欧拉平流；新Poisson求解方案处理固体边界耦合 | 蛙跳涡/涡旋碰撞/空腔流/固液相互作用等多种示例；代码开源获Replicability Stamp | 近边界Poisson求解需特殊处理；复杂几何需更精细网格 | 【摘要原文】【项目页描述】 |
| 4 | **Solid-Fluid Interaction on Particle Flow Maps** | Duowen Chen, Zhiqi Li et al. / Georgia Tech, Purdue, Dartmouth, Tsinghua | 弹性固体与脉冲流映射的强耦合，统一表示和精确传递耦合物理量 | 流体和固体统一表示为不同长度/动力学的粒子流映射；冲量→速度传递机制；粒子路径积分沿轨迹累积耦合力；集成到欧拉-拉格朗日脉冲流体模拟器 | 游泳/坠落/风吹/燃烧等场景中涡旋生成与演化 | 极端变形/拓扑变化固体需额外处理；计算成本随粒子数增长 | 【摘要原文】 |
| 5 | **An Impulse Ghost Fluid Method for Simulating Two-Phase Flows** | Yuchen Sun et al. / Harvard, Tsinghua, Georgia Tech | 两相流中涡量与界面复杂相互作用难捕捉；动态界面上规范变量跳跃条件具有历史依赖性 | 脉冲幽灵流体方法(IGFM)：双向流映射同时增强涡量和界面输运精度；时空缓冲区将历史依赖跳跃转化为几何依赖跳跃 | 界面漩涡/涡环反射/蛙跳气泡环等跨相位涡旋演化 | 路径积分增加算法复杂度；表面张力主导的微小尺度需更高分辨率 | 【摘要原文】 |
| 6 | **Neural Implicit Reduced Fluid Simulation (NIRFS)** | Yuanyuan Tao et al. / McGill, Huawei Canada, U Montreal | 高保真流体模拟计算成本极高，如何大幅降低同时保持精细细节 | 隐式神经表示(INR/SDF)表达流体形状 + Neural ODE在降维潜空间建模阻尼Hamiltonian系统动力学；联合学习表示和动力学 | 液滴碰撞/冠状飞溅/容器晃动；体积和动量守恒接近真实模拟；**625×加速**（vs Blender FLIP） | 每个场景需单独训练，泛化有限；高分辨率解码慢(512³ ~4.5秒/帧) | 【摘要原文】 |
| 7 | **Implicit Surface Tension for SPH** (also in NA 2024 list) | Stefan Rhys Jeske et al. / RWTH Aachen | 显式SPH表面张力在高表面张力系数下不稳定 | 隐式内聚力SPH+改进线性化后向欧拉+隐式粘度强耦合 | 无条件稳定，大时间步长 | 单次步成本高于显式 | 【摘要原文】 |
| 8 | **A Dual-Particle Approach for Incompressible SPH Fluids** (also in NA 2024) | Shusen Liu et al. / 中科院软件所 | 拉伸不稳定性导致粒子聚集无法生成薄特征 | 辅助虚拟粒子存储压力信息，压力与速度/位置解耦 | 细液丝/薄液膜/小液滴准确模拟 | 额外内存开销 | 【摘要原文】 |

### 2.2 布料/针织物模拟（3篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 9 | **Volumetric Homogenization for Knitwear Simulation** | Chun Yuan*, Haoyang Shi* et al. / U Utah, LightSpeed, Style3D, UCLA (*共同一作) | 纱线级模拟极昂贵；现有均质化将整件衣物视为单一材料，无法处理非重复编织图案 | **体均质化**：每个体积单元级别局部均质化纱线材质；虚拟体积+保体积惩罚项建模弯曲扭转；伴随Gauss-Newton逐单元材质优化；GPU域分解子空间求解器 | 质量与全尺度纱线级模拟相当；训练和模拟快数个数量级；GPU求解器比纱线级快数百倍 | 需离线预计算等效材质参数；极端拉伸/剪切下均质化假设可能失效 | 【摘要原文】 |
| 10 | **Efficient Cloth Simulation Using Non-distance Barriers and Subspace Reuse** (Mil2) | Lei Lan et al. / U Utah, UCLA, CAS Software, Style3D | IPC依赖原始间距离构造障碍势，难以高效集成到GPU流水线 | (1) **非距离障碍**：基于碰撞事件"虚拟生命周期"构造障碍势，所有顶点始终在可行域内；(2) **子空间复用**：低频应变由子空间求解器处理，高频残差由GPU迭代平滑 | 百万自由度服装**毫秒级**交互模拟；保持无穿透；比现有快速模拟器快至少一个数量级 | 生命周期参数需经验调节；极度密集自接触收敛速度可能下降 | 【摘要原文】 |
| 11 | **Neural Garment Dynamic Super-Resolution** | Meng Zhang, Jun Li / Nanjing Univ. Sci. & Tech. | 高分辨率布料模拟不适合低预算设备；低分辨率丢失高频褶皱 | Mesh Graph Net提取超分辨率特征 + Hyper-Net构建褶皱残差隐式函数；粗形状校正+残差预测；roll-out迭代预测 | 小数据集训练泛化到未见体型/动作/服装；高频细节显著优于SOTA；开源代码 | 依赖低分辨率物理输入；多层衣物复杂接触效果可能下降 | 【摘要原文】【项目页描述】 |

### 2.3 接触/碰撞/距离计算（4篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 12 | **Barrier-Augmented Lagrangian for GPU-based Elastodynamic Contact (BAL)** | Dewen Guo et al. / PKU, CMU, U Utah | IPC对数障碍需直接矩阵分解，内存>128GB CPU，无法GPU扩展 | 自适应更新增广约束集改善条件数→不完全Newton-PCG替代直接分解；稀疏GPU存储格式；特征值谱近似域分解热启动 | 128GB CPU内存→**8GB GPU显存**；数十至近百倍加速；能处理高刚度问题 | 增广参数自适应策略含启发式；摩擦接触需进一步验证 | 【摘要原文】 |
| 13 | **gDist: Efficient Distance Computation between 3D Meshes on GPU** | Peng Fan et al. / Zhejiang U, Shenzhen Poisson | 大规模三角网格最大/最小距离计算效率不足 | 细粒度并行+增强AABB距离边界公式+f12-BVH（满二叉树叶节点1-2三角形）+自适应展开深度 | RTX 4090上1500万三角形距离计算**<0.4ms**；比优化CPU(PQP)最高54×；最坏情况比朴素GPU快6403×；BVTT前端仅72MB | 简单场景并行加速受限；同心球等场景剪枝效果不佳 | 【摘要原文】 |
| 14 | **A Time-Dependent Inclusion-Based Method for CCD** | Xuwen Chen et al. / PKU, BIGAI | 参数曲面CCD需在空间和时间维度同时细分，高精度时计算消耗极大 | 时间依赖包含函数(time-dependent inclusion function)提供移动曲面连续表示；消除时间维度细分需求，5D搜索简化为更低维 | 比传统包含方法**36-138×加速**；支持1/2/3阶三角形和矩形补丁；开源代码 | 极复杂参数曲面空间细分仍多；非参数表示适配需额外工作 | 【摘要原文】 |
| 15 | **A Cubic Barrier with Elasticity-Inclusive Dynamic Stiffness** | Ryoichi Ando / ZOZO | 对数障碍函数在紧密间隙条件下病态条件数导致数值不稳定 | 三次障碍函数+弹性相关动态刚度调整；良好条件数+物理真实性；天然大规模并行 | 数百万次碰撞零穿透；对数障碍失效的紧密应变限制场景保持稳定；**180M+接触**GPU加速；开源ppf-contact-solver；Blender插件AndoSim | 摩擦模型实验阶段；参数选择对极端情况敏感 | 【摘要原文】【项目页描述】 |

### 2.4 固体力学/弹性体（7篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 16 | **MiNNIE: Mixed Multigrid for Real-time Nonlinear Near-Incompressible Elastics** | Liangwang Ruan et al. / PKU, BIGAI, Taichi Graphics | 高泊松比(近不可压缩)非线性弹性体实时仿真难；传统FEM体积锁定 | 混合FEM+压力稳定化项确保多重网格收敛；拟牛顿法消除节点位移注入影响；专用GPU多重网格求解器（顶点Vanka光滑子+Schur补稠密求解器） | 支持完整泊松比范围(至0.5)；实时性能；处理大变形+单元反转+自碰撞；开源代码 | 混合FEM增加自由度内存开销；压力稳定化参数需经验 | 【摘要原文】 |
| 17 | **XPBI: Position-Based Dynamics with Smoothing Kernels Handles Continuum Inelasticity** | Chang Yu*, Xuan Li* et al. / UCLA, U Utah (*共同一作) | XPBD长期被认为仅适用于弹性约束，难以处理连续介质非弹性（弹塑性/粘塑性/颗粒物质） | 光滑核函数稳健估计速度梯度→追踪变形梯度；隐式非弹性本构融入循环更新Lagrangian增广项(plasticity in-the-loop) | 雪/沙/橡皮泥等实时/高分辨率模拟；与标准XPBD布料/水无缝耦合；bridging XPBD与连续介质非弹性力学鸿沟 | 光滑核支持域大小需手动调节；大变形网格畸变仍困难 | 【摘要原文】 |
| 18 | **Trading Spaces: Adaptive Subspace Time Integration for Contacting Elastodynamics** | Ty Trusty et al. / U Toronto, Adobe, NVIDIA | 固定子空间无法适应时变形变模式和材料异质性 | 三组件：(1)自适应子空间预言机持续评估解质量并提出候选更新；(2)子空间无关自适应模型；(3)并行时间步求解器+一对可调容差参数 | 多种非线性材料/刚度/异质性/摩擦接触下验证；用户灵活控制精度-效率权衡 | 小规模问题上自适应额外开销不划算；瞬态冲击载荷可能有短暂精度损失 | 【摘要原文】 |
| 19 | **Accelerate Neural Subspace-Based Reduced-Order Solver by Lipschitz Optimization** | Aoran Lyu et al. / SCUT, U Manchester, CUHK | 神经降阶模拟的子空间内目标函数复杂度和景观未优化 | 优化弹性项Lipschitz能量寻找优化子空间映射；cubature近似融入训练管理高内存/时间开销 | 准静态和动力学均有效；最高**6.83×加速**，保持一致精度 | Lipschitz能量增加训练复杂度；cubature采样点影响精度 | 【摘要原文】 |
| 20 | **Trust-Region Eigenvalue Filtering for Projected Newton** | Honglin Chen et al. / Columbia, Roblox, U Toronto, NVIDIA, Adobe | 特征值截断(clamping)和绝对值滤波各有优劣，如何自适应选择？ | 首次证明牛顿法/clamped PN/absolute PN可通过广义信赖域统一；模型自适应选择最优滤波策略；**改两行代码** | 大型数据集上全面超越单一策略；实现极简 | 主要Neo-Hookean类能量；动态仿真表现待评估 | 【摘要原文】 |
| 21 | **Analytic Rotation-Invariant Modelling of Anisotropic Finite Elements** | Huancheng Lin et al. / HKU, EPFL | 构建低阶、足够非线性且鲁棒的各向异性能量公式是难题 | Cauchy-Green张量不变量隐式旋转分解；低阶+旋转不变+处处光滑+至少二阶可微；揭示ARAP各向异性版本构成被动/主动弹性材料基础参数描述 | 各向异性FEM统一解析框架；反演安全性；重写加速多种现有能量函数 | 存在勘误表；主要准静态/隐式优化；生物组织参数识别挑战 | 【摘要原文】 |
| 22 | **Q3T Prisms: Linear-Quadratic Solid Shell Element for Elastoplastic Surfaces** | Juan Sebastian Montes Maestre et al. / ETH Zürich | 线性棱柱单元处理不可压缩材料和塑性变形时出现严重锁定伪影 | 厚度方向引入二次形函数（最小但有效的修改）解决非常数应变无法捕捉的问题 | 消除锁定伪影；深冲压实验与真实世界定量吻合；计算成本仅适度增加 | 粘弹性等更复杂本构扩展待研究；自适应细化未整合 | 【摘要原文】 |

### 2.5 微结构与仿生设计（3篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 23 | **Optimized Shock-protecting Microstructures** | Zizhou Huang et al. / NYU | 如何设计弹性微结构在宽变形范围内提供近乎恒定反作用力以实现最优冲击防护？ | 可微分非线性均匀化+自接触建模的形状优化；单胞几何(最多80参数)针对目标恒定应力优化；105种候选连通性拓扑搜索 | 平坦响应达**75%压缩率**（先前最佳57%）；峰值Von Mises应力降低72%；吸收总冲量增加47%；3D打印实物验证（玻璃球+金属球落体测试）；开源代码 | 仅2D extruded到3D；排除对称瓦片限制设计空间；疲劳寿命未评估 | 【摘要原文】【项目页描述】 |
| 24 | **Computational Biomimetics of Winged Seeds** | Qiqin Le et al. / Shanghai Qi Zhi, Tsinghua, PKU, Georgia Tech | 如何通过计算方法自动生成具有优异空气动力学性能的人造翅果形态？能否超越自然演化？ | 3D扫描55枚种子(14物种)建立数据集→3D微分同胚群测地坐标插值构建生物启发设计空间→概率性能目标+无梯度优化搜索最优形态 | 发现超越自然的新型设计（长距离扩散/引导飞行任务）；纸模型实物验证仿真与现实一致性 | 主要静态形态优化；空气动力学仿真简化；微型飞行器应用需进一步探索 | 【摘要原文】 |
| 25 | **Tencers: Tension-Constrained Elastic Rods** | Liliane-Joy Dandy*, Michele Vidulis* et al. / EPFL (*共同一作) | 如何将经典张拉整体(tensegrity)推广到弹性可变形杆，获得更丰富设计空间？ | 少量不可伸长缆索张拉弹性杆系；内力使初始笔直柔性杆平衡变形为3D空间曲线；逆向设计优化缆索长度和放置位置逼近给定输入曲线；稀疏性项最小化缆索数量 | 弹性张拉整体结(elastic tensegrity knots)等新结构类别；开/闭杆、打结及任意拓扑；基础材料制造多个物理模型 | 假设无杆间接触；依赖商业优化器KNITRO；动态行为/稳定性分析未深入 | 【摘要原文】 |

---

## 三、定量加速对比

| 论文 | 对比基线 | 加速倍数 | 备注 |
|------|---------|---------|------|
| BAL (GPU IPC) | CPU直接分解(128GB内存) | **数十至近百倍** + 内存从128GB降至8GB | |
| Non-distance Barriers Cloth | 现有快速布料模拟器 | **≥10×** | 百万自由度毫秒级 |
| NIRFS | Blender FLIP Fluids | **625×** | 液滴场景 |
| Volumetric Homogenization | 纱线级模拟 | **数百倍**（GPU求解器） | |
| TDIB-CCD | 传统包含方法 | **36-138×** | |
| gDist | 优化CPU(PQP) | 最高**54×**；最坏情况**6403×** | |
| Lipschitz Neural Subspace | 未优化神经子空间 | 最高**6.83×** | |
| Cubic Barrier (ppf) | 对数障碍 | 支持**180M+接触** | GPU加速 |
| MiNNIE | 线性FEM | 实时 vs 离线 | 近不可压缩 |

---

## 四、规模记录

| 类别 | 规模 | 论文 |
|------|------|------|
| 最大接触数 | 180M+接触 | Cubic Barrier (ppf-contact-solver) |
| 最大网格距离计算 | 1500万三角形 <0.4ms | gDist |
| 最大布料DOF交互 | 百万自由度毫秒级 | Non-distance Barriers Cloth |
| 最大刚体摩擦 | 9122多面体沙堆 | Primal-Dual Friction (NA 2024) |
| 最大压缩率平坦响应 | 75%压缩率 | Shock-protecting Microstructures |
| 最大密度比 | 2000:1 | Kinetic Multifluid (NA 2024) |

---

## 五、开源项目列表

| 论文 | 代码/数据地址 | 许可证 |
|------|-------------|--------|
| Eulerian Vortex Method | GitHub (获ACM Replicability Stamp) | 开源 |
| TDIB-CCD | https://github.com/xw-c/TDIB-CCD | 开源 |
| ppf-contact-solver (Cubic Barrier) | https://github.com/st-tech/ppf-contact-solver | 开源 |
| MiNNIE | https://github.com/LwRuan/MiNNIE | 开源 |
| Tencers | https://github.com/EPFL-LGG/Tencers | 开源 |
| Neural Garment SR | https://github.com/MengZephyr/... | 开源 |
| Shock Protection | https://github.com/Huangzizhou/ShockProtection | 开源 |
| Trust-Region Newton | https://github.com/honglin-c/trust-region-newton | 开源 |
| CO-FLIP | arXiv:2406.01936 | 待确认 |
| Particle-Laden Fluid | arXiv:2409.06246 | 待确认 |
| Solid-Fluid PFM | arXiv:2409.09225 | 待确认 |
| Bal (Barrier-Augmented Lagrangian) | arXiv:2407.00046 | 待确认 |

---

## 六、信息缺口声明

本报告存在以下信息缺口：

1. **"Q3T Prisms"和"A Mesh-based Simulation Framework using Automatic Code Generation"** — 这两篇在physicsbasedanimation.com列表中但未提供链接，详细信息主要来自摘要和ACM DL，方法细节可能不够完整。

2. **部分论文缺少独立项目页面** — 如"Volumetric Homogenization"、"Non-distance Barriers Cloth"、"Lipschitz Neural Subspace"等未找到独立项目页面。

3. **作者列表可能不完整** — 对于未在arXiv或项目页明确列出全部作者的论文，作者信息可能遗漏部分合著者。

4. **定量指标缺失** — 部分论文（如"Trading Spaces"、"XPBI"、"Polar Interpolants"）未给出具体的加速倍数，仅提供了定性比较。

5. **World Model相关内容稀缺** — 与SIGGRAPH NA 2024观察一致，Asia 2024主会论文中几乎没有以"World Model"为核心概念的论文。NIRFS是最接近的工作，但其每个场景需单独训练的局限性使其与通用世界模型仍有距离。

---

## 七、对话框总结

SIGGRAPH Asia 2024的物理模拟论文有几个突出亮点。**最引人注目的是Georgia Tech Bo Zhu团队的流图（Flow Map）系列工作形成了完整体系**——从SIGGRAPH 2024的Covector Fluid和Particle Flow Maps，到Asia 2024的Particle-Laden Fluid（Best Paper）、Eulerian Vortex Method（获Replicability Stamp）、Solid-Fluid Interaction、Impulse Ghost Fluid四篇论文，系统性解决了从理想流体→粘性流体→颗粒载荷流→固液耦合→两相流的完整链条。这种在一个会议上集中展示系列工作的现象在近年SIGGRAPH中较为罕见。**第二个亮点是GPU加速接触求解器的突破性进展**——BAL将IPC从128GB CPU内存降至8GB GPU显存，Cubic Barrier支持180M+接触，Non-distance Barriers实现百万自由度布料毫秒级交互，这标志着接触求解器正式进入GPU时代。**第三个趋势是"可微分仿真+形状优化+3D打印验证"的完整闭环日益成熟**——抗冲击微结构（75%压缩率平坦响应）、翅果计算仿生（超越自然设计）、张拉整体弹性杆等多个工作都通过实物验证了仿真结果，体现了计算设计从纯数字走向物理世界的趋势。**第四，学习方法与传统物理模拟的融合更加精细化**——不再是简单的"神经网络替代求解器"，而是出现了Volumetric Homogenization（体均质化）、Lipschitz Neural Subspace（优化子空间景观）、Neural Garment SR（超分辨率）等更具针对性的分工模式。**最后值得注意的是，World Model概念在主会论文中同样缺席**——这与keynote和产品层面的热度形成鲜明对比，反映出图形学物理模拟社区对"用神经网络完全替代传统求解器"持相对谨慎的态度。
