# AMP 论文翻译与后续工作调研报告

## —— 从对抗运动先验到带几何约束的动作生成（2021–2026）

---

## 第一部分：AMP 论文翻译

### 论文基本信息

| 项目 | 内容 |
|------|------|
| **标题** | AMP: Adversarial Motion Priors for Stylized Physics-Based Character Control |
| **中文译名** | AMP：用于风格化基于物理的角色控制的对抗运动先验 |
| **作者** | Xue Bin Peng, Ze Ma, Pieter Abbeel, Sergey Levine, Angjoo Kanazawa |
| **机构** | UC Berkeley / Google Research |
| **发表** | ACM Transactions on Graphics (TOG) / SIGGRAPH 2021 |
| **arXiv** | 2104.02180 |
| **引用量** | 759+ (Google Scholar, 截至2026年8月) |

---

### 摘要（翻译）

为物理模拟角色合成优雅且逼真的行为一直是计算机动画领域的一项根本性挑战。利用动作跟踪的数据驱动方法是一类突出的技术，能够为广泛的行为产生高保真的动作。然而，这些基于跟踪的方法的有效性往往依赖于精心设计的目标函数，并且当应用于大型且多样化的动作数据集时，这些方法需要大量额外的机制来为角色选择在给定场景下应当跟踪的合适动作。

在本工作中，我们提出通过利用一种完全自动化的、基于对抗模仿学习的方法来消除手动设计模仿目标和动作选择机制的需求。角色应执行的高层级任务目标可以通过相对简单的奖励函数来指定，而角色行为的低层级风格则可以通过一个无结构动作片段的数据集来指定，无需任何显式的片段选择或排序。这些动作片段被用来训练一个**对抗运动先验（Adversarial Motion Prior）**，该先验通过强化学习（RL）为角色训练指定风格奖励。对抗强化学习过程会自动选择执行哪个动作，并从数据集中动态插值和泛化。

我们的系统产生的高质量动作可与最先进的基于跟踪的技术相媲美，同时还能轻松容纳大型无结构动作片段数据集。不同技能的组合会自动从运动先验中涌现，无需高层级动作规划器或对动作片段进行任务特定的标注。我们在多种复杂的模拟角色和一系列具有挑战性的运动控制任务上展示了我们框架的有效性。

---

### 引言核心内容（翻译与总结）

#### 研究动机

在计算机动画中，让虚拟角色在物理世界中自然地运动是一个长期存在的挑战。传统方法主要分为两类：

1. **运动学方法（Kinematic Methods）**：直接从动作捕捉数据库中选择和混合动作片段。优点是能产生细腻高质量的动作；缺点是在复杂环境中适应性差，无法处理物理交互。

2. **基于物理的方法（Physics-Based Methods）**：通过物理仿真和环境交互来生成动作。优点是对环境适应性强；缺点是动作质量往往不如运动学方法自然。

本文的核心思想是**融合两者的优势**：用数据驱动的kinematic方法提供low-level的细腻动作控制，用physics-based方法配合DRL算法完成high-level的路径规划和任务执行。

#### 与 DeepMimic 的关键区别

| 对比维度 | DeepMimic (2018) | AMP (2021) |
|----------|------------------|------------|
| **动作选择** | 需要显式指定跟踪哪个动作片段 | 自动从数据集中选择，无需手动指定 |
| **目标函数** | 需要精心设计跟踪奖励 | 通过对抗学习自动学习风格奖励 |
| **数据要求** | 需要结构化、标注好的动作数据 | 可直接使用无结构的原始动作数据 |
| **技能组合** | 需要高层级规划器 | 技能组合自动从先验中涌现 |
| **扩展性** | 数据集增大时难以扩展 | 能轻松容纳大规模无结构数据集 |

---

### 方法概述（翻译与总结）

#### 核心框架：对抗模仿学习

AMP 基于 **GAIL（Generative Adversarial Imitation Learning）** 的理论框架，将动作生成问题转化为一个对抗博弈：

1. **判别器（Discriminator）** $D_\phi$：区分真实动作（来自动作捕捉数据）和策略生成的动作
2. **策略（Policy）** $\pi_\theta$：通过与环境交互生成动作，试图"欺骗"判别器

#### 奖励函数设计

总奖励由两部分组成：

$$r_{total}(s_t, a_t) = r_{task}(s_t, a_t) + \omega \cdot r_{style}(s_t)$$

其中：
- $r_{task}$：**任务奖励**——高层级目标（如到达某位置），形式简单
- $r_{style} = \log D_\phi(s_t)$：**风格奖励**——由判别器给出，衡量动作的"真实性"
- $\omega$：平衡系数

#### 判别器架构

- 输入：状态转移对 $(s_t, s_{t+1})$ 的特征表示
- 网络结构：2-3层MLP
- 输出：标量概率值，表示输入来自真实数据的概率
- 训练方式：最小二乘GAN损失（LS-GAN），比标准交叉熵更稳定

#### 训练流程

```
循环执行：
  1. 用当前策略 π_θ 收集轨迹
  2. 从真实数据集和策略轨迹中采样，更新判别器 D_φ
  3. 用判别器输出计算风格奖励 r_style
  4. 结合任务奖励，用PPO更新策略 π_θ
```

#### 关键贡献总结

1. **无需手动设计模仿目标**：对抗学习自动从数据中学习什么是"好"的动作
2. **支持无结构大数据集**：不需要对动作片段进行选择、排序或标注
3. **技能自动组合**：行走、跑步、跳跃等技能的过渡和组合自然涌现
4. **通用框架**：适用于多种角色形态（人形、四足、异形）和多种任务

---

## 第二部分：AMP 后续相关工作调研（2021–2026）

### 总体发展脉络

```
2021 ── AMP (SIGGRAPH) ───────────────────────────────────────┐
                                                              │
2022 ── ASE (SIGGRAPH) ── Multi-AMP (ICRA) ──────────────────┤
                                                              ├──→ 技能空间扩展
2023 ── CALM (SIGGRAPH) ── PACER (CVPR) ── HumanMimic ───────┤     + 条件化控制
         Trace&Pace (CVPR)                                    │
                                                              │
2024 ── MaskedMimic (SIGGRAPH Asia) ── VMP (SCA/SIGGRAPH) ───┤
         PULSE (ICLR) ── RobotMDM (SIGGRAPH Asia) ────────────┤     → 统一控制
                                                              │     + 扩散模型
2025 ── PARC (SIGGRAPH) ── SRBTrack (SIGGRAPH Asia) ─────────┤
         NEAR (ICLR) ── PhysHMR (SIGGRAPH Asia) ──────────────┘     → 几何约束
                                                                      + 能量模型
```

---

### 一、技能空间扩展方向

#### 1.1 ASE: Adversarial Skill Embeddings (SIGGRAPH 2022)

**作者**: Xue Bin Peng, Yunrong Guo, Lina Halper, Sergey Levine, Sanja Fidler  
**核心贡献**: 在AMP的对抗运动先验基础上，学习可复用的**潜变量技能嵌入**。

- 将动作编码到一个连续的潜在空间中，每个点代表一种技能
- 下游任务只需在潜在空间中搜索合适的技能，而非重新训练
- 解决了AMP在处理多技能时的模式坍塌问题

#### 1.2 CALM: Conditional Adversarial Latent Models (SIGGRAPH 2023)

**作者**: Chen Tessler, Yoni Kasten, Yunrong Guo, Shie Mannor, Gal Chechik, Xue Bin Peng  
**核心贡献**: 通过**条件潜变量**实现可定向、多风格的角色控制。

- 在ASE的基础上引入条件变量（如目标方向、速度）
- 支持用户实时控制角色的运动方向和风格
- 实现了从"被动模仿"到"主动可控"的跨越

#### 1.3 Multi-AMP: Multiple Adversarial Motion Priors (ICRA 2023)

**作者**: Vollenweider et al.  
**核心贡献**: 使用**多个对抗判别器**分别学习不同风格的运动先验。

- 每个判别器对应一类技能或风格
- 策略可以灵活切换和组合不同先验
- 在轮腿四足机器人上实现了多种运动模式的实机部署

---

### 二、地形自适应与几何约束方向 ⭐

这是与您提到的"带几何约束条件的动作生成"最相关的方向。

#### 2.1 PACER: Pedestrian Animation ControllER (CVPR 2023)

**作者**: Davis Rempe, Zhengyi Luo, Xue Bin Peng, Ye Yuan, Kris Kitani, Karsten Kreis, Sanja Fidler, Or Litany  
**机构**: NVIDIA / Stanford / CMU

**核心贡献**: 开发了**地形自适应的运动控制器**，支持不规则地形行走。

**几何约束处理**:
- 在训练中引入多样化地形（楼梯、斜坡、不平地面）
- 通过AMP对抗学习保证动作自然性
- 支持不同体型的角色
- 接受2D路径点作为高层级输入

**局限**: 主要关注locomotion（移动），未涉及复杂的手脚接触约束。

#### 2.2 Trace and Pace: Guided Trajectory Diffusion + PACER (CVPR 2023)

**同一团队**在PACER基础上的进一步工作。

**核心创新**:
- **TRACE**: 用扩散模型生成可控的行人轨迹
- **PACER**: 底层物理控制器跟踪轨迹
- **闭环反馈**: PACER的价值函数反过来引导TRACE生成更易执行的轨迹

**几何约束**: 通过价值函数引导，使轨迹天然避开障碍物和困难地形。

#### 2.3 SRBTrack: Terrain-Adaptive SRB Tracking (SIGGRAPH Asia 2025)

**作者**: Hanyang Cao, Heyuan Yao, Libin Liu, Taesoo Kwon  
**机构**: KAIST / ByteDance

**核心贡献**: 基于**单刚体（Single Rigid Body, SRB）动力学**的地形自适应追踪框架。

**几何约束处理**:
- 将脚步位置投影到地形表面确定接触点
- 通过QP求解器计算符合物理规律的接触力
- 动量映射时空优化（Momentum-mapped Space-Time Optimization）保证全身运动的物理合理性
- **零样本适应**未见过的地形和动作风格

**与AMP的关系**: SRBTrack的训练使用了从大型无结构动作数据集中提取的参考运动，但不依赖对抗学习，而是结合了强化学习和监督学习。

#### 2.4 PARC: Physics-based Augmentation for Character Controllers (SIGGRAPH 2025)

**作者**: Michael Xu, Yi Shi, KangKang Yin, Xue Bin Peng  
**机构**: Simon Fraser University / NVIDIA

**核心贡献**: 通过**迭代式物理增强**解决敏捷地形穿越数据稀缺的问题。

**方法流程**:
```
小数据集 → 训练动作生成器 → 生成新地形上的合成动作
    ↓
物理追踪控制器修正（去除漂浮、滑动等伪影）
    ↓
修正后的高质量动作加入数据集 → 下一轮迭代
```

**几何约束处理**:
- 通过物理仿真自动修正不符合物理规律的动作
- 迭代过程中逐渐发现新的地形穿越技能（如跳越+攀爬的组合）
- 最终得到一个既能生成又能追踪复杂地形动作的系统

---

### 三、统一控制与通用表征方向

#### 3.1 PULSE: Physics-based Universal Motion Latent SpacE (ICLR 2024)

**作者**: Zhengyi Luo, Jinkun Cao, Josh Merel, Alexander Winkler, Jing Huang, Kris Kitani, Weipeng Xu  
**机构**: Meta Reality Labs / CMU

**核心贡献**: 提出了**通用人形运动表征**，覆盖人类运动的全面谱系。

**三层架构**:
1. **PHC+（教师）**: 改进的动作模仿器，在AMASS全数据集上训练
2. **蒸馏**: 用VAE结构将PHC+的技能压缩到32维潜在空间
3. **本体感觉条件先验**: 根据当前姿态和速度生成合理的动作分布

**关键创新——残差动作公式**:
$$a_{task} = D(\pi_{task}(z_{task} | s) + \mu^p)$$
高层策略的输出加到先验均值上，确保探索始终偏向类人行为。

**地形能力**: 在复杂地形遍历任务中，PULSE能自然地使用跳跃、跨步等策略，无需对抗风格奖励。

#### 3.2 MaskedMimic: Unified Control via Motion Inpainting (SIGGRAPH Asia 2024)

**作者**: Chen Tessler, Yunrong Guo, Ofir Nabati, Gal Chechik, Xue Bin Peng  
**机构**: NVIDIA / Bar-Ilan University

**核心贡献**: 将基于物理的角色控制统一为**运动修复（Motion Inpainting）问题**。

**核心思想**:
- 训练单一模型从**部分（掩码）运动描述**中合成完整动作
- 支持任意组合的控制模态：稀疏关键帧、文本指令、物体信息、场景信息
- **无需针对每种行为设计奖励函数**

**支持的模态**:
- 🎯 稀疏关节目标位置
- 🕹️ 摇杆转向
- 📝 文本命令
- 🏔️ 地形信息
- 🔄 任意上述组合

**意义**: 这是目前最接近"通用角色控制器"的工作，真正实现了"一个模型搞定所有任务"。

---

### 四、扩散模型与能量模型方向

#### 4.1 RobotMDM: Robot Motion Diffusion Model (SIGGRAPH Asia 2024)

**作者**: Agon Serifi, Ruben Grandia, Espen Knoop, Markus Gross, Moritz Bächer  
**机构**: ETH Zurich / Disney Research

**核心贡献**: 将**文本条件扩散模型**与物理基控制器对齐。

**方法**:
1. 训练一个**奖励代理（Reward Surrogate）**来预测下游控制任务的性能
2. 用该奖励微调扩散模型，使生成的动作既多样又符合物理规律
3. 在真实双足机器人LIME上验证

**与AMP的区别**: 不使用对抗训练，而是用可微的奖励代理替代不可微的控制任务。

#### 4.2 VMP: Versatile Motion Priors (SCA/SIGGRAPH 2024)

**同一团队**的另一项工作。

**核心贡献**: 两阶段框架——先用VAE学习运动潜在表示，再用条件策略训练追踪控制器。

**避免了对抗训练的稳定性问题**，同时在真实机器人上实现了舞蹈等动态动作。

#### 4.3 NEAR: Noise-conditioned Energy-based Annealed Rewards (ICLR 2025)

**作者**: Anish Diwan, Julen Urain, Jens Kober, Jan Peters  
**机构**: TU Delft / TU Darmstadt / DFKI

**核心贡献**: 提出了AMP的**能量模型替代方案**。

**核心思想**:
- 用去噪分数匹配（Denoising Score Matching）学习动作数据的能量函数
- 将能量函数作为奖励函数用于强化学习
- 渐进退火策略确保奖励始终有定义

**相比AMP的优势**:
- ✅ 避免了GAN训练的不稳定性
- ✅ 提供更平滑的奖励景观
- ✅ 在小数据情况下更鲁棒

**劣势**: 在大数据情况下性能略逊于AMP。

---

### 五、视觉感知与Sim-to-Real方向

#### 5.1 PhysHMR: Physics-based Human Motion Reconstruction (SIGGRAPH Asia 2025)

**作者**: Qiao Feng, Yiming Huang, Yufu Wang, Jiatao Gu, Lingjie Liu

**核心贡献**: 从单目视频直接学习**视觉到动作的物理合理重建**。

**关键创新——Pixel-as-Ray**:
- 将2D关键点提升为3D空间射线
- 提供鲁棒的全局姿态引导，不依赖噪声大的3D根节点预测
- 知识蒸馏从MoCap专家迁移到视觉条件策略

**几何约束**: 直接保证了物理合理性（无脚部漂浮、地穿透等伪影）。

#### 5.2 HumanMimic: Wasserstein Adversarial Imitation (ICRA 2024)

**作者**: Annan Tang, Takuma Hiraoka, Naoki Hiraoka, Fan Shi, Kento Kawaharazuka, Kunio Kojima, Kei Okada, Masayuki Inaba  
**机构**: 东京大学

**核心贡献**: 使用**Wasserstein距离**替代JS散度，改进了人形机器人的对抗模仿学习。

**改进点**:
- 更稳定的训练过程
- 更自然的步态转换
- 在真实人形机器人上验证

---

### 六、其他重要变体

#### 6.1 APEX: Multi-Critic RL (2025)

**核心贡献**: 解耦任务评论家和模仿评论家，提高了技能多样性和训练稳定性。

- 在约1000次迭代内达到高性能（AMP需约50000次）
- 显著改善了样本效率和步态多样性

#### 6.2 CAMP: Skill-Conditioned Policy (2025)

**核心贡献**: 技能判别器 + 条件策略，克服了AMP在多技能设置下的单峰行为倾向。

#### 6.3 BCAMP: Behavior-Controllable AMP for Quadruped (2025)

**核心贡献**: 将用户指令与对抗运动先验结合，实现四足机器人的行为可控。

---

## 第三部分：综合分析与发展趋势

### 3.1 关于"带几何约束的动作生成"

您提到的"带几何约束条件的动作生成"确实是AMP后续发展的一个重要方向。具体来说，这个方向包含以下几个层面：

| 约束类型 | 代表工作 | 处理方法 |
|----------|----------|----------|
| **足部接触约束** | PACER, SRBTrack | 地形高度图 + 脚步投影 + QP求解 |
| **全身碰撞约束** | PARC, PhysHMR | 物理仿真自动修正 + 价值函数引导 |
| **动量/动力学约束** | SRBTrack | 单刚体动力学 + 动量映射优化 |
| **环境交互约束** | MaskedMimic, PARC | 场景信息作为输入 + 迭代物理增强 |

### 3.2 技术演进趋势

```
AMP (2021)
  │
  ├──→ 技能空间: ASE → CALM → PULSE → MaskedMimic
  │     (单一先验 → 条件潜变量 → 通用表征 → 统一修复)
  │
  ├──→ 几何约束: PACER → SRBTrack → PARC
  │     (简单地形 → 零样本适应 → 迭代增强)
  │
  ├──→ 替代方案: Multi-AMP → NEAR → APEX
  │     (多判别器 → 能量模型 → 多评论家)
  │
  └──→ Sim-to-Real: HumanMimic → VMP → RobotMDM → PhysHMR
        (Wasserstein → VAE → 扩散模型 → 视觉感知)
```

### 3.3 当前挑战与未来方向

1. **手物交互**: 现有工作主要集中在locomotion，复杂的手物操作仍是开放问题
2. **长程规划**: 如何结合LLM/VLA进行语义级的长程任务规划
3. **Sim-to-Real Gap**: 虽然已有进展，但复杂动态动作的实机部署仍有差距
4. **计算效率**: 大规模训练仍需要大量GPU资源
5. **理论保证**: 对抗训练的收敛性和稳定性仍需更好的理论分析

---

## 参考文献

### AMP核心论文
1. Peng, X. B., Ma, Z., Abbeel, P., Levine, S., & Kanazawa, A. (2021). **AMP: Adversarial Motion Priors for Stylized Physics-Based Character Control**. ACM TOG / SIGGRAPH 2021.

### 前置工作
2. Peng, X. B., Abbeel, P., Levine, S., & Van de Panne, M. (2018). **DeepMimic: Example-Guided Deep Reinforcement Learning of Physics-Based Character Skills**. ACM TOG / SIGGRAPH 2018.
3. Ho, J., & Ermon, S. (2016). **Generative Adversarial Imitation Learning**. NeurIPS 2016.

### 技能空间扩展
4. Peng, X. B., Guo, Y., Halper, L., Levine, S., & Fidler, S. (2022). **ASE: Large-Scale Reusable Adversarial Skill Embeddings for Physically Simulated Characters**. ACM TOG / SIGGRAPH 2022.
5. Tessler, C., Kasten, Y., Guo, Y., Mannor, S., Chechik, G., & Peng, X. B. (2023). **CALM: Conditional Adversarial Latent Models for Directable Virtual Characters**. ACM TOG / SIGGRAPH 2023.
6. Vollenweider, D., et al. (2023). **Advanced Skills through Multiple Adversarial Motion Priors in Reinforcement Learning**. ICRA 2023.

### 地形自适应与几何约束
7. Rempe, D., Luo, Z., Peng, X. B., Yuan, Y., Kitani, K., Kreis, K., Fidler, S., & Litany, O. (2023). **Trace and Pace: Controllable Pedestrian Animation via Guided Trajectory Diffusion**. CVPR 2023.
8. Cao, H., Yao, H., Liu, L., & Kwon, T. (2025). **SRBTrack: Terrain-Adaptive Tracking of a Single-Rigid-Body Character Using Momentum-Mapped Space-Time Optimization**. SIGGRAPH Asia 2025.
9. Xu, M., Shi, Y., Yin, K. K., & Peng, X. B. (2025). **PARC: Physics-based Augmentation with Reinforcement Learning for Character Controllers**. SIGGRAPH 2025.

### 统一控制
10. Luo, Z., Cao, J., Merel, J., Winkler, A., Huang, J., Kitani, K., & Xu, W. (2024). **Universal Humanoid Motion Representations for Physics-Based Control (PULSE)**. ICLR 2024.
11. Tessler, C., Guo, Y., Nabati, O., Chechik, G., & Peng, X. B. (2024). **MaskedMimic: Unified Physics-Based Character Control Through Masked Motion Inpainting**. ACM TOG / SIGGRAPH Asia 2024.

### 扩散与能量模型
12. Serifi, A., Grandia, R., Knoop, E., Gross, M., & Bächer, M. (2024). **Robot Motion Diffusion Model: Motion Generation for Robotic Characters**. SIGGRAPH Asia 2024.
13. Serifi, A., Grandia, R., Knoop, E., Gross, M., & Bächer, M. (2024). **VMP: Versatile Motion Priors for Robustly Tracking Motion on Physical Characters**. SCA / SIGGRAPH 2024.
14. Diwan, A. A., Urain, J., Kober, J., & Peters, J. (2025). **Noise-conditioned Energy-based Annealed Rewards (NEAR): A Generative Framework for Imitation Learning from Observation**. ICLR 2025.

### Sim-to-Real
15. Tang, A., Hiraoka, T., Hiraoka, N., Shi, F., Kawaharazuka, K., Kojima, K., Okada, K., & Inaba, M. (2024). **HumanMimic: Learning Natural Locomotion and Transitions for Humanoid Robot via Wasserstein Adversarial Imitation**. ICRA 2024.
16. Feng, Q., Huang, Y., Wang, Y., Gu, J., & Liu, L. (2025). **PhysHMR: Learning Humanoid Control Policies from Vision for Physically Plausible Human Motion Reconstruction**. SIGGRAPH Asia 2025.

### 其他变体
17. Sood, et al. (2025). **APEX: Decoupled Critics for Diverse Skill Learning**.
18. Huang, et al. (2025). **CAMP: Conditional Adversarial Motion Priors with Skill Discriminator**.
19. Peng, Y., Cai, Z., Zhang, L., & Wang, X. (2025). **BCAMP: A Behavior-Controllable Motion Control Method Based on Adversarial Motion Priors for Quadruped Robot**.

---

*报告生成时间: 2026年8月11日*  
*调研范围: 2021年(AMP发表)至2026年8月*