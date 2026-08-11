# SIGGRAPH 2024 物理模拟论文逐篇总结

> **报告范围**：SIGGRAPH 2024（北美）Technical Papers中与物理模拟相关的论文，涵盖流体、气体、固体、弹性体、布料、毛发、碰撞检测、多相流、神经模拟等方向。
> **数据来源**：physicsbasedanimation.com 官方收录列表 + 各论文项目页/arXiv/ACM DL。
> **可靠性标注**：【摘要原文】来自论文原始摘要；【项目页描述】来自作者公开的项目页面；【二手转述】来自主流学术报道或综述；【推断】基于方法描述的合理推断。

---

## 一、总体概览与主题脉络

SIGGRAPH 2024共收录约**40篇**物理模拟相关论文（按physicsbasedanimation.com收录口径）。从主题分布来看，呈现以下趋势：

### 主题脉络

1. **蒙特卡洛方法的复兴** — 3篇论文（Velocity-based MC、Neural MC、以及相关工作）探索无网格的蒙特卡洛流体求解器，试图克服传统网格方法的局限性。这是近年来少见的方向回潮。

2. **流图（Flow Map）方法持续发力** — Georgia Tech Bo Zhu团队在SIGGRAPH 2024有2篇流体论文（Covector Fluid Free Surface、Eulerian-Lagrangian on Particle Flow Maps），后续在Asia 2024继续扩展为完整系列（见Asia 2024报告）。

3. **接触/碰撞求解器的系统性改进** — GIPC（解析特征系统+GPU加速）、Vertex Block Descent（块坐标下降）、Primal-Dual Non-Smooth Friction（内点法二次收敛）三篇论文从不同角度推进了IPC框架和刚体摩擦的数值效率。

4. **神经+物理深度融合** — 从ContourCraft（GNN解决交叉问题）到Neural Homogenization（纱线级织物均质化）再到Super-resolution Cloth（8倍分辨率提升），学习方法不再只是"替代"物理，而是与传统模拟器形成互补分工。

5. **多物理耦合场景扩展** — 气旋模拟（大气+热力学+微物理）、野火模拟（植被+燃烧+大气）、铁流体（磁静力+流体）、肥皂膜（涡旋+表面活性剂+薄膜张力）等论文体现了从单一物理向多物理耦合的发展趋势。

6. **可微分仿真走向实用** — DiffIPC实现了含摩擦接触的时变变形问题的通用可微求解器，支持形状优化、参数估计等下游任务。

### World Model 相关发现

与SIGGRAPH 2026/2025的观察一致，**World Model概念在主会论文中几乎未出现**。最接近的是VR-GS（高斯泼溅+物理交互）和Going with the Flow（给定姿态序列预测流体介导运动），但均未使用"world model"术语。神经流体方法（Neural MC、Neural Homogenization等）更多是替代特定求解步骤而非构建统一世界模型。

---

## 二、分类论文详表

### 2.1 流体模拟（8篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|------|-----------|---------|---------|----------|------|--------|
| 1 | **Lightning-Fast Method of Fundamental Solutions** ⭐ Best Paper Award | Jiong Chen, Florian Schäfer, Mathieu Desbrun / INRIA, Georgia Tech, Caltech | MFS/BEM稠密线性系统可扩展性差 | 利用边界积分矩阵逆的稀疏结构+高斯过程联系，提出变分预处理器（稀疏逆Cholesky分解），用于PCG | **最高10,000×加速** | 主要适用于线性椭圆型PDE；非线性扩展待研究 | 【摘要原文】【项目页描述】 |
| 2 | **Kinetic Simulation of Turbulent Multifluid Flows** | Wei Li, Kui Wu, Mathieu Desbrun / Shanghai Jiao Tong, Caltech | 分离多相流在高雷诺数和大密度比下的模拟难题 | 守恒相场LBM：D3Q7追踪每种流体分布 + HOME-LBM（D3Q27/D3Q19）编码速度场，通过界面扩散编码耦合压力/粘度/界面力 | 内存节省~80%（每节点12n-8.5 vs 43n+20浮点数）；密度比2000:1稳定；计算速度比现有LBM快2.35× | 不支持相变；自适应网格细化未集成 | 【摘要原文】 |
| 3 | **Velocity-Based Monte Carlo Fluids** | Ryusuke Sugimoto, Christopher Batty, Toshiya Hachisuka / U Toronto, Waterloo | 之前涡度基MC无法处理浮力驱动流动，且与基于速度的技术不兼容 | 算子分裂+逐点MC估计器；walk-on-boundary将投影和扩散重表述为积分问题；压力Poisson体积分转换 | 正确模拟浮力效应、散度控制、advection-reflection等 | MC方差需大量样本；补充材料有5处方程勘误 | 【摘要原文】 |
| 4 | **Neural Monte Carlo Fluid Simulation** | Pranav Jain, Ziyin Qu, Peter Yichen Chen, Oded Stein / USC, UPenn, MIT | 神经场弱约束边界条件导致复杂边界/湍流场景失败 | 显式强制边界条件 + MC压力求解器；完全无网格 | 首次实现von Kármán涡街等定性现象 | 精度未达成熟网格方法SOTA；仅2D验证 | 【摘要原文】 |
| 5 | **Going with the Flow** | Yousuf Soliman et al. / Caltech, TU Berlin | 给定姿态序列，如何高效模拟物体在流体介质中的运动响应？ | 广义刚体动力学方程：局部近似流体惯性(added mass)+阻力-升力效应；仅需积分6维ODE（3旋转+3平移） | 线性复杂度；捕捉硬币落水、种子旋转下落等现象 | 仅刚体；不支持柔性体；局部近似在极端情况下精度受限 | 【摘要原文】【项目页描述】 |
| 6 | **Lagrangian Covector Fluid with Free Surface** | Zhiqi Li*, Barnabás Börcsök* et al. / Georgia Tech (*同等贡献) | 协向量流图在处理自由表面边界条件时的根本困难 | 粒子轨迹建立精确流图；路径积分重构投影Poisson问题；解耦机制将长程流图积分转化为短程投影；自由表面→零Dirichlet，固体边界→零Neumann | 纯粒子方法SOTA；成功处理3D双排水口draining tank、波浪撞击多固体 | 大规模3D自由表面效率待验证；Voronoi构建3D开销大 | 【摘要原文】 |
| 7 | **Fluid Control with Laplacian Eigenfunctions** | Yixin Chen, David I.W. Levin, Timothy R. Langlois / U Toronto, Adobe, NVIDIA | 流体控制在效率和精度间难以平衡 | Laplacian Eigenfluids基底+伴随方法解析梯度；支持多分辨率和频率域控制 | 实时流体模拟/编辑/控制；精确目标形状匹配 | 仅2D验证；3D扩展可行性未展示 | 【摘要原文】 |
| 8 | **Eulerian-Lagrangian Fluid Simulation on Particle Flow Maps** | Junwei Zhou et al. / Georgia Tech, Dartmouth | 之前Neural Flow Maps计算效率低、内存消耗大 | 四组件框架：(1)拉格朗日粒子精确双向流图；(2)双尺度流图表示；(3)粒子→网格插值；(4)混合冲量求解器 | 相比NFM计算时间减少**49×**，内存减少**41%**，涡度保持更好 | 极稀疏区域插值精度下降；主要针对不可压缩流动 | 【摘要原文】 |

### 2.2 SPH流体（2篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|------|-----------|---------|---------|----------|------|--------|
| 9 | **Implicit Surface Tension for SPH Fluid Simulation** | Stefan Rhys Jeske et al. / RWTH Aachen | 显式表面张力在高表面张力系数下稳定性不足 | 隐式内聚力SPH模型；改进线性化后向欧拉时间离散+隐式粘度强耦合；扩展粘附力公式 | 无条件稳定，允许更大时间步长；水冠/滴水龙头等复杂场景 | 单次步计算成本高于显式；剧烈破碎精度受限于粒子分辨率 | 【摘要原文】 |
| 10 | **A Dual-Particle Approach for Incompressible SPH Fluids** | Shusen Liu, Xiaowei He et al. / 中科院软件所 | 拉伸不稳定性导致粒子聚集(clumping)，无法生成薄特征 | 引入辅助虚拟粒子专门存储压力信息；压力计算与速度/位置粒子解耦 | 准确模拟细液丝、薄液膜、小液滴；显著减少粒子聚集伪影 | 额外内存开销；压力重分布参数需调节；未扩展到多相流 | 【摘要原文】 |

### 2.3 固体/弹性体模拟（7篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 11 | **A Dynamic Duo of Finite Elements and Material Points** | Xuan Li, Minchen Li et al. / UCLA, CMU, TRI, Style3D, Utah | FEM与MPM耦合难（隐式vs显式时间积分矛盾） | IMEX异步时间分裂：FEM弹性+跨域摩擦接触隐式大步长(IPC)；MPM弹性小步长显式；两阶段牛顿法 | FEM区域与MPM区域无缝共存双向交互；支持任意余维度耦合 | IMEX分裂引入时间离散误差；接触穿透需后处理校正 | 【摘要原文】 |
| 12 | **MERCI: Mixed Curvature-based Elements for Thin Elastic Ribbons** | Raphaël Charrondière et al. / Inria, Sorbonne | 薄弹性带介于薄板和细杆之间，低阶DER精度不足，高阶效率低 | 紧凑带状单元：法向曲率沿弧长线性变化+测地挠率二次函数；混合变分策略+双边约束连接；Hessian带状结构 | 比DER快**一个数量级**；数秒内达到DER无法企及的高精度；开源代码 | 主要准静态；极度复杂自接触需更精细处理 | 【摘要原文】【项目页描述】 |
| 13 | **Position-based Nonlinear Gauss-Seidel for Quasistatic Hyperelasticity** | Yizhou Chen, Yushan Han et al. / Epic Games, UCLA, UC Davis | PBD不适用于准静态问题（神经网络训练数据生成需要） | 位置空间非线性Gauss-Seidel（非约束投影）；保留PBD稳定性+扩展预算时收敛保证；天然支持SOR/Chebyshev/多分辨率加速 | 有限迭代下稳定，更多迭代时收敛到精确解；成功用于准静态数据集生成 | 动力学校准需额外工作；收敛速度依赖材料参数 | 【摘要原文】 |
| 14 | **Stabler Neo-Hookean Simulation: Absolute Eigenvalue Filtering** | Honglin Chen et al. / Columbia, Roblox, U Toronto, NVIDIA | 高泊松比+大体积变化下特征值截断(clamping)易失败或收敛极慢 | **绝对值特征值过滤**：负特征值取绝对值而非截断为零；保持能量景观曲率信息；**改一行代码** | 高泊松比(ν=0.495)下牛顿迭代从150次降至30次；开源代码 | 低泊松比/小变形时可能略减缓收敛 | 【摘要原文】【项目页描述】 |
| 15 | **Simplicits: Mesh-free, Geometry-agnostic Elastic Simulation** | Vismay Modi et al. / U Toronto, NVIDIA, Texas A&M | 传统模拟器仅适用于四面体网格，无法处理NeRF/高斯泼溅等新表示 | 点采样+小型隐式神经网络表示空间变化权重（约化形变基）；随机扰动训练学习物理显著形变模式；蒙特卡洛采样评估弹性能量 | 5种表示(SDF/点云/CT/NeRF/GS/网格)上140+次模拟；已集成NVIDIA Kaolin库 | 需离线训练；MC噪声；精度依赖采样密度 | 【摘要原文】【项目页描述】 |
| 16 | **Preconditioned Nonlinear CG for Real-time Interior-point Hyperelasticity** | Xing Shen et al. / 网易伏羲AI Lab | IPC计算瓶颈：牛顿求解器+CCD昂贵且难GPU并行 | Jacobi预处理非线性CG(PNCG)替代牛顿法；单次遍历线搜索无需额外CCD；全GPU并行 | 10万+四面体实时模拟；已应用于机器人触觉仿真(Tac2Real ECCV 2026) | 非线性CG对步长敏感；主要tetrahedral mesh | 【摘要原文】【项目页描述】 |
| 17 | **Efficient Position-based Deformable Colon Modeling** | Marcelo Martins et al. / UFRGS Brazil, UFPel Brazil, IST Lisbon | 内窥镜模拟器过度简化管状解剖结构的导航和交互 | XPBD+Cosserat杆约束保持结肠整体形状+四面体网格捕捉局部细节；真实患者CT重建；新型管状接触检测算法 | 实时交互帧率；内窥镜导航与实际手术高度相似 | 针对结肠特定器官；快速动态响应未验证 | 【摘要原文】 |

### 2.4 布料/毛发/羽毛模拟（7篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 18 | **Modelling a Feather as a Strongly Anisotropic Elastic Shell** | Jean Jouve et al. / Inria, U Utah, Roblox, Caltech | 羽毛强各向异性（羽枝vs羽小枝刚度比~10⁴），以往各向同性近似失效 | 实验测量→三参数正交各向异性弹性壳连续介质模型；网格沿羽枝对齐+最刚硬模态替换为不可延伸约束解决锁定 | 中间尺度微结构模型验证一致性；完整羽毛+大尺度鸟类图形学场景；开源代码和数据 | 依赖网格对齐；整根羽毛（含羽轴）耦合动力学有扩展空间 | 【摘要原文】 |
| 19 | **Real-Time Physically Guided Hair Interpolation** | Jerry Hsu et al. / U Utah, LightSpeed, Roblox | 引导毛发大变形时LBS插值产生几何伪影（直发弯折、卷发锯齿） | 力基插值：从引导毛发插值内部力→基于材料模型重建渲染毛发形状；约束满足问题+快速Gauss-Seidel；漂移校正+穿透惩罚 | 视觉伪影完全消除；每根毛发独立物理属性；额外开销仅~20%；无需预计算/训练 | 依赖引导毛发模拟质量；>200k strands可扩展性未评估 | 【摘要原文】 |
| 20 | **Proxy Asset Generation for Cloth Simulation in Games** | Zhongtian Zheng et al. / PKU, LightSpeed | 从视觉网格自动生成单层低poly代理网格+蒙皮权重极其困难（需美术师数天手工调整） | 全自动流水线：复杂多层视觉网格→单层低poly代理；基于使用场景模拟；微分蒙皮+多损失函数优化蒙皮权重 | 100+种游戏服装；代理网格可低至128顶点；从天级→分钟级 | 极复杂多层服装单层代理不足；极端材质需调参 | 【摘要原文】 |
| 21 | **Neural-Assisted Homogenization of Yarn-level Cloth** | Xudong Feng et al. / Zhejiang U, U Utah, Style3D | 纱线级织物均质化在大时间步长下数值不稳定（需line search） | 神经网络自适应复杂动态行为+固有平滑性缓解稳定性；扇区基础热启动加速数据收集；三阶导数+泛化正则化 | 大时间步长下比之前模型快**两个数量级**；集成到Style3D产品 | 训练分布外极端形变精度下降；纱线滑移显著时均质化假设不准确 | 【摘要原文】 |
| 22 | **ContourCraft: Learning to Resolve Intersections in Neural Multi-garment Simulations** | Artur Grigorev et al. / ETH Zürich, MPI-IS | 基于学习的布料模拟碰撞/交叉处理未解决；一旦交叉整个动画崩溃 | HOOD(GNN)基础上：(1)布料-布料对应关系+排斥损失预防交叉；(2)新颖交叉轮廓损失(ICM变分公式化)修复交叉 | 自动修复因自动缩放产生的交叉；导出Alembic格式；开源代码 | 泛化受训练数据限制；深层自相交恢复能力有限 | 【摘要原文】【项目页描述】 |
| 23 | **Progressive Dynamics for Cloth and Shell Animation** 📰 TOG封面 | Jiayi Eris Zhang et al. / Adobe Research, Stanford | 高保真布料模拟计算代价极高，艺术家反复迭代调整参数效率极低 | 从粗到细多层次LOD物理模拟；基于IPC框架；tight-matching consistency + progressive improvement | 粗分辨率预览成本仅相当于最粗层直接模拟；"Sky Dancers"等复杂案例；后续有PD++(SIGGRAPH 2025)改进 | 原始方法稳定性和时间连续性有局限（PD++部分解决） | 【摘要原文】【项目页描述】 |
| 24 | **Super-resolution Cloth Animation With Spatial and Temporal Coherence** | Jiawang Yu, Zhendong Wang / Zhejiang U, Style3D | 粗糙网格添加精细褶皱时面临空间一致性和时间连贯性两大挑战 | 两模块：(1)模拟器-校正器交错（GNN修正不同分辨率低频差异）；(2)基于网格的超分辨率（重叠patches分解） | **8倍分辨率提升**；简单布片到复杂服装均有效；应用于Style3D MixMatch | 训练数据外极端材质/运动泛化不足；更高倍数质量衰减未研究 | 【摘要原文】 |

### 2.5 接触/碰撞/表面跟踪（7篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 25 | **GIPC: Fast and Stable Gauss-Newton Optimization of IPC Barrier Energy** | Kemeng Huang et al. / HKU, EPFL | IPC屏障函数直接用距离计算Hessian限制了性能和设计；近平行边-边接触尤其困难 | 单纯形几何度量重写IPC屏障函数；推导解析特征系统；滤波+刚度增强确保Project-Newton鲁棒收敛；全GPU实现 | 解析特征系统本身**3×加速**；全GPU进一步加速 | 主要针对IPC框架；GPU内存缓冲需手动调整 | 【摘要原文】 |
| 26 | **Vertex Block Descent** | Anka He Chen et al. / U Utah, Roblox | 弹性体求解器在数值收敛性、稳定性、计算性能间难以兼顾 | 顶点级块坐标下降+Gauss-Seidel迭代；全局变分能量单调递减；图着色最大并行化 | 计算时间显著快于XPBD/Newton；处理质量比1:2000（XPBD失败）；10368模型落入盒子(1亿四面体/100万碰撞)；开源Gaia框架 | 原始VBD主要软约束；摩擦采用lagged方式 | 【摘要原文】【项目页描述】 |
| 27 | **Primal-Dual Non-Smooth Friction for Rigid Body Animation** | Yi-Lu Chen, Mickaël Ly, Chris Wojtan / IST Austria | 光滑求解器牺牲静摩擦稳定处理；非光滑求解器收敛慢 | 原生-对偶内点算法+对数障碍函数将非光滑问题转化为光滑问题；牛顿迭代高效求解 | 9122个多面体沙堆；5185块砖城堡被摧毁；立方体指数增长质量堆叠；House of Cards在所有容差级别解决 | 主要针对刚体；需合适初始化和参数选择 | 【摘要原文】 |
| 28 | **Multi-Material Mesh-Based Surface Tracking with Implicit Topology Changes** | Peter Heiss-Synak et al. / IST Austria | 基于网格的方法保留细节但难处理拓扑变化；Level Set鲁棒但平滑特征 | 推广Wojtan 2009：显式网格保持细节+背景网格隐式曲面检测拓扑缺陷→裁剪移除→隐式重建填充→缝合；推广到非流形 | 数千相互作用气泡肥皂膜；数百万三角形非流形网格布尔并集 | 算法复杂度随材料数量增加；极端变形需频繁重网格化 | 【摘要原文】 |
| 29 | **Differentiable Voronoi Diagrams for Cell-based Mechanical Systems** | Logan Numerow et al. / ETH Zürich, CMU | 细胞力学系统中拓扑转变（分裂/合并/邻域变化）对现有方法构成重大挑战 | 每个细胞用Voronoi位点表示；界面网络形状/拓扑隐式定义；闭式一阶和二阶导数支持Newton型方法 | 胚胎卵裂过程模拟；匹配真实图像逆问题（肥皂泡沫）；比显式模型显著更快；开源代码 | Voronoi假设凸多边形/多面体；主要2D和简单3D | 【摘要原文】 |
| 30 | **Contact Detection Between Curved Fibres: High Order Makes a Difference** | Octave Crespel et al. / Inria, ELAN | 低阶基元（线段/三角形）接触检测在纤维弯曲时产生力伪影 | 分析低阶几何对接触力的影响；两条光滑曲线间高精度检测方案+自适应剪枝策略；应用于super-helices | 波浪到高度卷曲纤维范围内恢复更平滑力剖面；梳头场景鲁棒性验证 | 高维自适应剪枝开销需优化；主要Kirchhoff细弹性杆 | 【摘要原文】 |
| 31 | **A Framework for Solving Parabolic PDEs on Discrete Domains** | Leticia Mattos da Silva, Oded Stein, Justin M. Solomon / MIT | 几何处理领域缺乏处理一般非线性抛物型PDE的统一框架 | Strang splitting三步骤：(1)热方程向前推进；(2)Hamilton-Jacobi方程处理非线性（凸优化求解）；(3)再次热方程推进 | G-equation火焰前沿传播；对数域最优传输；Fokker-Planck方程；表面漩涡演化 | 仅静态表面；单个PDE；未涵盖所有边界条件类型 | 【摘要原文】 |

### 2.6 专用物理应用（8篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 32 | **Cyclogenesis: Simulating Hurricanes and Tornadoes** | Jorge Amador Herrera et al. / KAUST, CAU Kiel, AMU Poznań | 图形学缺乏视觉上令人信服且物理合理的三维气旋发展模拟 | RANS+热力学耦合：大尺度热量/水分连续性+水凝物湍流微物理+行星边界层中尺度气旋过程；动态耦合温度/湿度/风场 | 飓风完整生命周期；龙卷风非对称特征；卡特里娜飓风2005年登陆数据定量验证；交互式渲染；开源CycloCore | 空间分辨率远低于气象业务模式；微物理参数化简化 | 【摘要原文】【项目页描述】 |
| 33 | **Scintilla: Simulating Combustible Vegetation for Wildfires** | Andrzej Kokosza et al. / AMU Poznań, CAU Kiel, KAUST | 野火模拟缺乏对植被详细几何结构和燃料属性的精细建模 | 三组件：(1)新型植被表示（树枝几何+燃料含水率+草/细燃料/腐殖质空间分布）；(2)对流+燃烧+热传递动态耦合；(3)飞火(firebrand)/余烬(ember)点燃生成输运模型 | 地表火→树冠火转变；飞火随风飘散引发新火点；防火隔离带效果演示 | 大范围场景植被数据获取困难；燃烧化学反应简化；大范围实时性能挑战 | 【摘要原文】 |
| 34 | **An Induce-on-Boundary Magnetostatic Solver for Grid-based Ferrofluids** | Xingyu Ni*, Ruicheng Wang*, Bin Wang†, Baoquan Chen† / PKU, BIGAI (*共同一作†通讯) | 现有铁流体磁静力求解器需复杂线性系统求解器且可扩展性受限 | IoB求解器：基于单层势理论，仅利用物体表面点云计算磁场；边界积分方程求解避免体离散大量自由度；结合FMM加速 | Rosensweig不稳定性尖刺结构；磁铁吸引铁流体经典实验；显著减少计算时间和内存；开源代码 | 超大尺度3D性能待验证；表面点云质量影响精度 | 【摘要原文】 |
| 35 | **A Vortex Particle-on-mesh Method for Soap Film Simulation** | Ningxiao Tao et al. / PKU, Stanford, Georgia Tech, BIGAI | 缺乏精确捕捉薄膜上级联涡旋结构与膜法向运动耦合的模拟方法 | 切向速度分解为环量(circulation)+膨胀(dilatation)分量；混合粒子-网格方法分别演化；整合表面活性剂动力学+膜厚度动力学 | 首次实现薄膜上级联涡旋高保真模拟；肥皂膜破裂/合并/排液；Marangoni效应 | 膜拓扑变化需额外处理；极薄膜数值稳定性要求高 | 【摘要原文】 |
| 36 | **Efficient Debris-flow Simulation for Steep Terrain Erosion** | Aryamaan Jain, Bedrich Benes, Guillaume Cordonnier / Inria, Purdue | 传统水流侵蚀在陡坡低排水量区域失效（产生不真实均匀斜坡） | 泥石流侵蚀数学模型（图形学"热侵蚀"推广）+统一GPU侵蚀沉积算法+带洼地地形GPU近似流径路由算法 | 泥石流侵蚀疤痕+沉积扇与河流竞争；陡峭山脊+精细高频地貌特征 | 经验参数需地质条件调整；植被抑制作用简化；未考虑粒径分选 | 【摘要原文】 |
| 37 | **Real-time Wing Deformation Simulation for Flying Insects** | Qiang Chen et al. / 江西财经大学, 休斯顿大学 | 昆虫翅膀形态差异大，缺乏通用实时模拟多种昆虫翅膀真实形变的方法 | 骨骼驱动模型：反映不同昆虫种类形态特征的虚拟骨骼系统+周期性拍翅内部驱动力+简化气动力；质点-弹簧模拟固有弹性 | 蝴蝶/蜜蜂/蜻蜓等多种昆虫；弯曲波传播现象；用户研究验证视觉真实感 | 气动模型简化（未考虑前缘涡等非定常效应）；微观结构均质化近似 | 【摘要原文】 |
| 38 | **VR-GS: A Physical Dynamics-Aware Interactive Gaussian Splatting System in VR** | Ying Jiang*, Chang Yu* et al. / UCLA, HKU, Utah, Zhejiang U, Style3D, CMU, Amazon (*共同一作) | 如何在VR中与GS表示的3D内容进行直观物理交互 | 3D GS+XPBD结合；两级嵌入（高斯核→笼子网格→物理模拟）；VDB稀疏体积数据结构包围网格重建+四面体化；场景分割+多视角inpainting | 实时帧率VR交互；抛环套动物/抚摸狐狸等场景；视觉质量优于PAC-NeRF；较高SUS评分；开源代码 | 主要针对中小型场景；物理参数需手动设置；多用户协同不完善 | 【摘要原文】【项目页描述】 |
| 39 | **Differentiable Solver for Time-Dependent Deformation Problems with Contact** (DiffIPC) | Zizhou Huang*, Davi Colli Tozoni* et al. / NYU, nTop, MIT, Victoria (*共同一作) | 含摩擦接触的时变变形问题梯度-based优化因接触/摩擦不可微性而困难 | 通用可微求解器：FEM+高阶时间积分器+IPC接触/摩擦；解析推导伴随公式；伴随问题结构与正向模拟高度相似 | 形状优化（衣架应力最小化）、初始条件优化、摩擦系数优化；复杂几何数百接触体可扩展；集成开源PolyFEM | 形状优化不能直接包含障碍物；密度/质量/外力大小优化不支持；需remeshing处理大变形 | 【摘要原文】【项目页描述】 |

### 2.7 其他（1篇）

| # | 标题 | 作者/机构 | 核心问题 | 方法要点 | 效果/指标 | 局限 | 可靠性 |
|---|---|---|---|---|---|---|---|
| 40 | **A Neural Network Model for Efficient Musculoskeletal-driven Skin Deformation** | Yushan Han et al. / UCLA | 基于FEM的生物力学软组织仿真计算极其昂贵，难以用于动画管线 | 分层准静态仿真数据训练神经网络；骨骼运动学+肌肉激活度输入→皮肤表面变形输出 | 逼真肌肉鼓起/皮肤滑动/脂肪抖动；推理速度远快于直接FEM | 准静态训练数据，动态惯性效应不够准确；泛化受训练数据覆盖限制 | 【摘要原文】 |

---

## 三、定量加速对比

| 论文 | 对比基线 | 加速倍数 | 备注 |
|------|---------|---------|------|
| Lightning-Fast MFS | 未预条件MFS | **最高10,000×** | 最佳论文奖 |
| Eulerian-Lagrangian on PFM | Neural Flow Maps | 49×计算时间, 41%内存 | |
| GIPC | 原始IPC | 3×（仅特征系统）+ GPU额外加速 | |
| Neural Yarn Homogenization | 解析均质化模型 | **100×**（两个数量级） | 大时间步长下 |
| PNCG IPC | 牛顿法+CCD | 实时(10万+tet) vs 离线 | GPU实现 |
| Vertex Block Descent | XPBD/Newton | 显著快（具体倍数未给出） | 处理1:2000质量比 |
| Super-resolution Cloth | 高分辨率直接模拟 | 8×分辨率提升 | 学习增强 |
| PBNG Quasistatic | Newton法 | 6×（67s vs 430s/帧） | 肌肉碰撞 |

---

## 四、规模记录

| 类别 | 规模 | 论文 |
|------|------|------|
| 最大DOF模拟 | 1亿四面体 + 100万活跃碰撞 | Vertex Block Descent |
| 最多刚体 | 9122个多面体颗粒沙堆 | Primal-Dual Friction |
| 最大密度比 | 2000:1（溃坝场景） | Kinetic Multifluid LBM |
| 最多布料/服装示例 | 100+种游戏服装 | Proxy Asset Generation |

---

## 五、开源项目列表

以下论文公开了代码或数据（按可用性排序）：

| 论文 | 代码/数据地址 | 许可证 |
|------|-------------|--------|
| Simplicits | https://github.com/NVIDIAGameWorks/kaolin (集成Kaolin) | NVIDIA |
| Vertex Block Descent | https://github.com/AnkaChan/Gaia + Tiny Demo | 开源 |
| GIPC | https://github.com/KemengHuang/GPU_IPC | 开源 |
| Abs-EigenFilter | https://github.com/honglin-c/abs-psd | 开源 |
| PNCG IPC | https://github.com/Xingbaji/PNCG_IPC | 开源 |
| ContourCraft | https://github.com/dolorousrtur/ContourCraft | 开源 |
| MERCI | https://gitlab.inria.fr/elan-public-code/merci | CeCILL |
| Feather Shell | https://gitlab.inria.fr/elan-public-code/feather-shell/ | CeCILL |
| Covector Fluid Free Surface | https://github.com/ZhiqiLi-CG/CovectorFluidFreeSurface | 开源 |
| Particle Flow Maps | https://github.com/pfm-gatech/particle-flow-maps | 开源 |
| Cyclogenesis | https://github.com/AgrosAmad/CycloCore | 开源 |
| IoB Ferrofluid | https://github.com/Univstar/IoB-Ferrofluid | 开源 |
| VR-GS | https://github.com/YingJiang96/VR-GS | 开源 |
| DiffIPC | https://github.com/Huangzizhou/DiffIPC-data + PolyFEM | 开源 |

---

## 六、信息缺口声明

本报告存在以下信息缺口：

1. **部分论文缺少独立项目页面** — 如"A Framework for Solving Parabolic PDEs"、"Dual-Particle SPH"、"Neural Yarn Homogenization"等论文未找到独立项目页面，方法细节主要来自摘要和ACM DL。

2. **作者列表可能不完整** — 对于未在arXiv或项目页明确列出全部作者的论文（如"Super-resolution Cloth"、"Colon Modeling"），作者信息可能遗漏部分合著者。

3. **定量指标缺失** — 部分论文（如"Parabolic PDE Framework"、"Differentiable Voronoi"）未给出具体的加速倍数或误差指标，仅提供了定性比较。

4. **"A Neural Network Model for Efficient Musculoskeletal-driven Skin Deformation"** — 该论文在physicsbasedanimation.com列表中但未找到独立项目页面，详细信息较少。

5. **World Model相关内容稀缺** — 与SIGGRAPH 2025/2026观察一致，主会论文中几乎没有以"World Model"为核心概念的论文。该方向的最新进展主要集中在NVIDIA keynote和产品层面。

---

## 七、对话框总结

SIGGRAPH 2024的物理模拟论文呈现出几个值得关注的趋势。**首先，蒙特卡洛方法意外回潮**——3篇论文从不同角度探索无网格MC流体求解器，这在近年SIGGRAPH中较为罕见，反映出学界对传统网格方法局限性的反思。**其次，Bo Zhu团队（Georgia Tech）的流图（Flow Map）系列工作开始形成体系**，从Covector Fluid到Particle Flow Maps逐步解决了自由表面、粘性耗散等关键问题，这一方向在Asia 2024进一步扩展为完整的单相→多相→固液耦合链条。**第三，接触/碰撞求解器的数值基础持续深化**，GIPC的解析特征系统、Vertex Block Descent的块坐标下降、Primal-Dual Friction的内点法分别从不同数学角度推进了IPC框架的效率和适用范围。**第四，神经方法与物理模拟的融合进入深水区**——不再是简单的"用神经网络替代求解器"，而是出现了更精细的分工：ContourCraft用GNN处理交叉修复、Neural Homogenization用网络自适应复杂本构、Super-resolution用学习增强高频细节。**最后，多物理耦合和专用应用场景大幅扩展**，从气旋到野火到铁流体到肥皂膜，物理模拟正在走出传统的"水/烟/布料"舒适区。值得注意的是，**World Model概念在主会论文中几乎缺席**，这与keynote和产品层面的热度形成鲜明对比——图形学物理模拟的主流仍然是经典数值方法而非神经替代。
