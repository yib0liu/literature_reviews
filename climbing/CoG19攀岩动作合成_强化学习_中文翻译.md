# 一种用于合成攀岩动作的强化学习方法

**原文标题**：A Reinforcement Learning Approach To Synthesizing Climbing Movements

**作者**：Kourosh Naderi、Amin Babadi、Shaghayegh Roohi、Perttu Hämäläinen

**机构**：芬兰阿尔托大学（Aalto University）计算机科学系，赫尔辛基

**发表于**：IEEE Conference on Games (CoG) 2019

**原文链接**：https://ieee-cog.org/2019/papers/paper_34.pdf

> 本文档为原论文的完整中文翻译，供个人阅读理解使用。术语首次出现时保留英文原文。

---

## 摘要

本文研究的问题是：在给定目标攀岩点（target holds）的条件下——例如由攀岩游戏的玩家指定——合成仿真人形角色的攀岩动作。我们贡献了**首个能够处理"交互式物理仿真人形攀岩、且允许多个肢体同时更换攀岩点"的深度强化学习方案**。

我们方法中的一个关键组件是**自监督回合状态初始化**（Self-Supervised Episode State Initialization，SS-ESI）。相比于"失败后将攀岩者重置回初始姿态"的基线方法，SS-ESI 能保证探索的多样性并加速学习。

我们的结果还表明：采用**多步动作参数化**（multi-step action parameterization）进行训练，既能产生更平滑的动作，也能在探索更少动作样本的情况下完成学习——代价是每个动作所需的仿真时间增加。

**关键词**：强化学习、攀岩动作、状态初始化、动作参数化

---

##一、引言

自从深度强化学习（Deep RL）在 Atari 游戏中展现出超越人类的表现[1]以来，它变得越来越流行。此后，RL 被成功应用于许多场景，例如游戏[2]、基于物理的动画[3][4]以及机器人学[5]。

然而，**如何把一个任务表述成 RL 问题、以及如何设计有意义的奖励函数，本身就是困难且高度依赖具体问题的工作**。其中一个仍未被探索、且并不简单的方向，就是合成看起来合理的人形攀岩动作。

用强化学习合成基于物理的动作已经是多年来的活跃研究领域。但许多最先进的方法依赖于高质量的参考动作，例如动作捕捉（motion capture）动画数据——而这类数据往往昂贵，甚至根据角色与环境的不同而根本无法录制。当前方法的另一个局限是：它们聚焦于**非极限动作**，比如行走（locomotion），此时智能体与环境的交互是有限的。

本文研究的问题是：在仿真的室内抱石（bouldering）环境中，基于给定的手/脚目标攀岩点位置来合成攀岩动作（见图 1）。在抱石环境中，攀岩者从墙前的地面开始，被要求先把手/脚放到某些预先指定的攀岩点上。攀岩者的目标是通过执行各种攀岩动作到达某个指定的目标点（通常在墙顶），其中一些动作可能相当有挑战性且笨重费力。

![图 1](paper/figs/p1_0.png)

**图 1**：由训练好的策略在一条攀岩路线上合成的攀岩动作。红色箭头表示用户指定的指令，红色叉号表示让某个肢体不与任何攀岩点相连（即"自由肢体"）。

本文提出了一个基于强化学习的框架，用于在物理仿真环境中求解人形攀岩问题。我们的框架能够合成**物理上合理**的攀岩动作，并且可以涉及**同时移动多于一个肢体**，或者存在**自由肢体**（free limbs，见图 1）。

攀岩问题的主要难点在于：训练需要海量的仿真经验，才能探索足够宽泛的攀岩动作范围。为缓解这一问题，受[3]中"参考状态初始化"（Reference State Initialization, RSI）方法成功经验的启发，我们比较了两种回合状态初始化策略，评估它们对探索各类动作的影响。我们还证明，使用不同的动作参数化可以合成平滑度不同的动作。我们提出的框架能够以较高的成功率探索出广泛的攀岩动作。

本文余下部分组织如下：第二节回顾部分相关工作；第三、四节介绍我们的训练环境、把攀岩问题形式化为强化学习问题，并研究一种能提升攀岩动作学习性能的探索方法；最后，第五、六、七节分别对实验、结果、局限性与未来工作进行讨论并总结全文。

---

## 二、相关工作

我们的主要贡献是用强化学习**形式化并求解**人形攀岩问题。因此我们首先回顾在连续状态与动作空间环境中学习如何行动的最新 RL 方法，然后讨论在物理仿真设定下求解攀岩问题的最先进方法。

### A. 连续环境中的强化学习

强化学习研究的是通过试错（trial and error）来学习在环境中行动，以最大化回报的过程。得益于深度神经网络的巨大进步，RL 在过去五年经历了若干重大突破[1][6][7]。尽管大量工作聚焦于像玩经典 Atari 游戏这样的**离散动作**，但在**高维连续动作空间**问题上，前沿水平也取得了极大进展[8]，我们在此简要概述。

大多数连续控制 RL 依赖从仿真数据计算出的蒙特卡洛梯度估计。然而，简单地沿梯度前进可能不稳定。为缓解此问题，**信任域**（trust region）系列方法被提出，通过限制每次更新中策略之间的偏离量来解决该问题[9]。**近端策略优化**（Proximal Policy Optimization，PPO）是这一家族的后代，它使用一个代理损失函数（surrogate loss）而非硬性限制[10]。PPO 在广泛的连续域中取得了卓越成果，已成为计算机图形学社区的默认算法[2][3]。**PPO 也是本工作中用于求解人形攀岩问题的算法，通过 Unity ML-Agent 框架[2]实现。**

一个更近期且具有最先进结果的方法称为**软演员-评论家**（Soft Actor-Critic，SAC），它利用最大熵原理来实现离策略（off-policy）学习[5]。已有工作表明它能以合理的样本复杂度成功用于真实世界的机器人问题[11]。

策略梯度方法的一个替代思路是：把策略更新表述为**将策略分布拟合到已探索过的动作上**，每个动作依据其估计的价值或优势（advantage）加权。这在 RL 与 CMA-ES 这类迭代分布拟合优化方法之间建立了桥梁。**PPO-CMA**利用受 CMA-ES 启发的分布自适应技术，解决 PPO 的过早收敛问题[12]。类似思路被用于**最大后验策略优化**（Maximum a Posteriori Policy Optimisation，MPO）[13]及其近期变体[14]，并在 DeepMind control suite[15]和 OpenAI Gym[16]环境中取得了很好的结果。

### B. 合成攀岩动作

已有许多关于攀岩动作合成的研究。

- [17] 通过对机器人质心采样、并对移动肢体求闭式解，为四肢机器人规划**单肢**攀岩动作。
- [18] 用强化学习方法规划人形攀岩者的**单肢**移动，并用迁移学习把规划迁移到新墙面；但身体的移动使用的是**逆运动学**（inverse kinematics）。本文提出的强化学习方法可用于交互式攀岩仿真或游戏。与[18]相比，我们的方法能处理**多个肢体同时移动**，且攀岩者可以有**自由肢体**，从而合成更复杂的动作。
- [19][20] 表明攀岩动作可以从基于物理的动作优化中涌现出来。但角色被限制为**一次只移动一个肢体**的缓慢、更加平衡的攀岩动作，例如先移动手再移动腿来爬墙。
- [21] 引入了"低层优化 + 高层路径规划器"的层次结构，以合成更合理、更具动态性/敏捷性的攀岩动作，但限制为**一次移动两个肢体**。在他们的工作中，智能体可以决定在缓慢与动态攀岩动作之间切换。这项工作后来在[22]中被增强：利用神经网络预测来合成更成功的过渡以及诸如动态跳跃（dyno）之类的复杂动作。
- 也有工作聚焦于子问题，例如攀岩者手指与抓握的仿真[23]。
- [24] 提出了一种名为 **2PAC** 的方法，通过接触规划与一种新的运动学约束表述，能为任意角色合成包括攀岩在内的动作，从而为角色生成准静态（quasi-static）的质心轨迹。

然而，**所有这些优化方法在计算上都很昂贵**。相比之下，RL 的优势在于：尽管训练阶段代价高昂，但训练好的神经网络策略可以非常轻量，适用于像电子游戏这样资源受限的实时应用。

---

## 三、预备知识

### A. 攀岩环境

**环境**：图 2 展示了用于训练的攀岩环境。攀岩者以 **T 字姿态**（T-pose）站在 16 个随机化的攀岩点前。每个攀岩点 $h$ 的位置记为 $x_h$。攀岩点排布为 **4×4 网格**并叠加随机扰动。自由肢体（即未附着到任何攀岩点的肢体）的位置记为 $x_{-1}$。攀岩点建模为半径 **15 cm** 的球体，带有球窝关节（ball-and-socket joint），攀岩者的手/脚可以附着于其上。

**攀岩状态**（Climbing State）：我们的攀岩者身高 **1.75 m**、体重 **70 kg**，不过把该框架推广到其他角色体型是很容易的。攀岩者有 **15 根骨骼**，共 **44 个自由度**，由 **30 个目标关节角**与除根节点外所有骨骼的 **14 个关节强度值**构成。时刻 $t$ 的攀岩者状态记为：

$$s_t = \{\langle x_b,\ v_b,\ q_b,\ \omega_b,\ \tau_b,\ I_{wall}(b),\ I_{ground}(b)\rangle \ \forall b \in B\}$$

其中 $B$ 是所有骨骼的集合，单根骨骼的状态包含 3D 位置、线速度、旋转四元数、角速度与强度。强度 $\tau_b$ 对应物理引擎中控制骨骼 $b$ 的关节马达的**最大力矩参数**。另外用两个指示函数 $I_{wall}(b)$ 与 $I_{ground}(b)$ 分别表示骨骼 $b$ 是否接触墙面与地面。

**攀岩姿势/站位**（Climbing Stance）：把攀岩点分配给攀岩者手/脚的方案称为一个 **stance**，记为

$$\sigma = \{x_{ll},\ x_{rl},\ x_{lh},\ x_{rh}\}$$

（依次为左腿、右腿、左手、右手）。其中每个 $x_i \in \sigma$ 可以被设为自由 $x_{-1}$，或附着到墙上的某个攀岩点 $x_h \in H$。在训练/攀岩开始时，攀岩者以 T 字姿态站在地面上，此时处于 stance $\sigma_0 = \{x_{-1}, x_{-1}, x_{-1}, x_{-1}\}$ 与状态 $s_0$。

### B. 强化学习

在强化学习中，我们有一个处于环境中的智能体，目标是让智能体学会如何行动，以最大化从环境获得的奖励。在每个时间步 $t$，智能体观测当前状态 $o_t$，并使用其策略 $\pi_\theta(a_t \mid o_t)$ 选择下一个动作 $a_t$；该策略是从状态到动作分布的一个映射。执行动作 $a_t$ 后，智能体获得标量奖励 $r_t$，并观测到新状态 $o_{t+1}$。目标是最大化期望累积回报，即

$$\mathbb{E}\left[\sum_{t=0}^{T}\gamma^t r_t\right]$$

其中 $\gamma \in [0,1]$ 是折扣因子。

在深度强化学习与连续控制中，策略通常用神经网络建模，参数记为 $\theta$。策略网络 $\pi_\theta(a_t \mid o_t)$ 通常输出均值 $\mu_\theta(a_t \mid o_t)$ 与协方差 $C_\theta(a_t \mid o_t)$，以确定依赖于观测的动作分布。本文使用 **ML-Agents**[2]——一个 Unity 插件，提供了 PPO[10]（一种流行的连续控制 RL 算法）的开源实现。

---

## 四、方法：用于攀岩的强化学习框架

本节介绍我们用于合成攀岩动作的强化学习框架。第 IV-A 节把攀岩动作合成形式化为强化学习问题；第 IV-C 与 IV-B 节提出一种对所学动作多样性影响巨大的探索策略；最后第 IV-D 节说明如何用不同的动作参数化来平滑攀岩动作。所有参数取值见第 V-A 节。

### A. 强化学习形式化

#### 1) 智能体的观测

在每个时间步 $t$，观测向量 $o_t$ 定义为：

$$o_t = \langle s_t,\ \sigma_c,\ \sigma_d,\ I(\sigma_c),\ I(\sigma_d),\ I(\sigma_c \neq \sigma_d)\rangle \tag{1}$$

其中 $s_t$ 与 $\sigma_c$ 是智能体的当前攀岩状态与当前 stance（如第三节所定义）。$\sigma_d$ 是由**用户或高层路径规划器**设定的手/脚位置的**期望 stance**；而 $\sigma_c$ 取决于攀岩者的当前状态，表示其当前的手/脚位置。$I(\sigma_c)$ 与 $I(\sigma_d)$ 分别指示在当前与期望 stance 中手/脚是否附着在攀岩点上。$I(\sigma_c \neq \sigma_d)$ 表示当前与期望 stance 中哪些元素彼此不同。

#### 2) 智能体的动作

在时间步 $t$ 采样得到的动作向量 $a_t$ 有 **44 个取值**，范围为 $[-1, 1]$。$a_t$ 的**前 30 个元素**表示攀岩者关节的随机化目标角度，**后 14 个元素**被映射到 $[0, \tau_{max}]$ 范围内，用作关节的强度。关节限位与 $\tau_{max}$ 的设定使攀岩者具有类人的能力限制。

#### 3) 奖励函数

定义奖励 $r_t$ 是开发该框架的挑战之一。如果只用类似 $r_t \in \{-1, 1\}$ 的**二值奖励**来表示攀岩者的成功与失败，那么由于**稀疏奖励**问题，学习会慢到无法接受。为加速学习过程，即时奖励函数应当足够有信息量，以在每个时间步引导策略让手/脚更接近目标 stance。受 DeepMimic奖励函数[3]的启发，我们采用如下**基于距离**的奖励函数：

$$r_t^{\sigma} = I(d_t)\sum_{i \in \{ll,rl,lh,rh\}}\Big[k_\sigma \exp\big(-c_\sigma \lVert \sigma_{c,i} - \sigma_{d,i} \rVert\big) + I(\sigma_{c,i} = \sigma_{d,i})\Big] - I_{ground}(s) \tag{2}$$

其中 $I(d_t)$ 是一个指示函数，用于判定智能体是否比**以往任何时刻**都更接近目标 stance。**若没有 $I(d_t)$，智能体会在目标攀岩点附近来回摆动，而不是把手脚真正附着上去**；使用 $I(d_t)$ 能产生更稳定的伸手/触点行为。$k_\sigma$ 与 $c_\sigma$ 是调参用的参数。当攀岩者的手/脚到达其期望位置 $\sigma_{d,i}$ 时，$I(\sigma_{c,i} = \sigma_{d,i})$ 取 1。

> 需要注意：原则上奖励应当是**观测与动作**的函数。因此，计算 $I(d_t)$ 所需的数据（即此前达到过的最近距离）本应加入观测中。不过，式(1)的观测在实践中似乎已经可行。

使用式(2)的奖励函数并不能保证策略产生平滑、节能的攀岩动作。为鼓励平滑动作，我们引入一个基于关节强度的奖励函数：

$$r_t^{\tau} = k_\tau I(d_t)\exp\left(-c_\tau \sum_{i \in \{31,\dots,44\}} \big(a_t[i] + 1\big)^2\right) \tag{3}$$

其中 $I(d_t)$ 的作用与式(2)相同。该函数对策略产生的采样强度值求和。$c_\tau$ 与 $k_\tau$ 是调参参数。$k_\tau$ 小于式(2)中的 $k_\sigma$，以使"平滑性"的重要性低于策略的主要目标——把手/脚移动到目标 stance。

最终，$r_t$ 由式(2)与式(3)之和给出：

$$r_t = r_t^{\sigma} + r_t^{\tau} \tag{4}$$

#### 4) 训练回合与终止条件

PPO 假设训练期间以**回合**（episode）为单位收集经验：每个回合从某个初始攀岩状态开始，运行至时间上限 $T_{episode}$ 或到达终止攀岩状态。在我们的设定中，一个攀岩状态是终止状态，当且仅当：攀岩者**到达了目标 stance** $\sigma_d$，或者攀岩者身体除脚以外的**任何部位触地**。

### B. 目标 Stance 的探索

攀岩者通过为每个回合采样一个目标 stance来探索不同的攀岩动作。然而，从给定回合的初始攀岩状态出发，并非所有过渡都是合法的。从攀岩状态 $s$ 与 stance $\sigma$ 到期望 stance $\sigma_d$ 的过渡是**合法的**，当且仅当满足以下全部**合法性条件**（Validity Conditions）：

- **连接性限制**（Connectivity Limit）：在 $\sigma$ 与 $\sigma_d$ 中都应至少有一只手是连接的；例外情形是攀岩者处于 T 字姿态（$\sigma = \sigma_0$）时，此时只需 $\sigma_d$ 至少有一只手连接。
- **距离限制**（Distance Limit）：在当前与目标 stance 中，手到手、脚到脚、手到脚之间的距离都不应超过攀岩者身体的可及范围（body reach）。
- **肢体移动限制**（Limb Movement Limit）：$\sigma$ 与 $\sigma_d$ 之间**最多只能有两个元素不同**。

本文采用如下目标 stance 采样策略：为采样一个随机目标 stance，我们首先在初始网格位置附近、以 $r = 33\text{cm}$ 的范围内**随机扰动墙上所有攀岩点**（图 2-(a)）。然后，我们为**1 个或 2 个肢体**在其移动肢体的"邻居攀岩点"中采样新的目标点，如图 2-(b) 所示。某个肢体的邻居攀岩点包括：

- $x_{-1}$，即目标可以只是"释放该肢体"（使其自由）。
- 位于该肢体阴影区域内的攀岩点（图 2-(c)）。该阴影区域是以攀岩者当前髋部位置为中心、半径 $r_{body}$ 的范围。
- 通过虚线与"移动肢体当前所附着的攀岩点"相连的那些攀岩点。

移动肢体的目标攀岩点从邻居攀岩点集合中**均匀选取**。

![图 2](paper/figs/p4_0.png)

**图 2**：(a) 初始攀岩点位置及其随机化半径 $r$，攀岩者以 T 字姿态站在墙前。(b) 攀岩者左手的邻居攀岩点。邻居点包括阴影区域内的攀岩点，以及用虚线与红点所示攀岩点相连的那些点。(c) 四个阴影区域表示基于攀岩者髋部位置、其手/脚可触及的攀岩点合法区域。

### C. 攀岩状态的探索

如[3]所示，**训练回合的初始状态分布**会对学习复杂人形动作产生巨大影响。他们的 RSI 技术将智能体初始化到从参考动捕数据中采样的随机状态。在我们的情形中，训练回合的初始攀岩状态对攀岩动作的**可行性与探索性**起着重要作用。促进攀岩动作的多样化探索，取决于是否拥有丰富的初始攀岩状态分布。我们实现并比较了以下两种攀岩状态初始化方法：

#### 1) 标准回合式 RL（基线，Baseline）

这里，类似于 MuJoCo 环境这类常见连续控制基准，我们把攀岩者初始化到随机化墙面前的 T 字姿态（如图 2 所示），并让攀岩一个目标 stance 接一个目标 stance 地持续下去。在**失败**或达到时间上限（$T_{baseline}$）后，我们重置回 T 字姿态并重新随机化墙面。失败定义为：在未达到目标 stance 的情况下到达了终止状态（见第 IV-A4 节）。

> 注意：PPO 看到的**每个经验回合只包含"移动到一个期望 stance"这一次移动**，这使得价值估计的噪声小得多，因为价值不依赖于未来的期望 stance。在初期测试中，我们发现这能极大提升性能。

#### 2) 自监督回合状态初始化（SS-ESI）

如算法 1 所述，我们**逐步扩展可能初始攀岩状态的集合 $S$**，方式是同时收集失败与成功的攀岩动作（第 9 行）。在这种情形下，每个回合只包含**一次**试图到达单个期望 stance 的攀岩移动。实际效果上，回合轨迹会从先前的轨迹**分叉**，形成一棵**探索树**（exploration tree）。

对每个训练迭代，我们在第 2 行从 T 字姿态（$s_0$）开始。在每次策略更新之前，第 4-9 行逐步在 $S$ 中收集更多攀岩状态。若至少有一只手连接到攀岩点，则把$s_e$ 加入 $S$。为了获得更多样、更均匀的攀岩动作经验，我们在算法 1 中确保：

- 训练回合的初始攀岩状态 $s$ 从**已见/已探索的攀岩状态中均匀选取**（第 5 行），并在攀岩墙上被**重新定位**（第 6 行），如图 3 所示。我们**仅在所有手脚都连接时**才做这种重定位，以避免破坏动力学仿真。
- 目标 stance 的采样分布（第 7 行）是**平衡的**，使我们采样到数量相近的脚部移动与手部移动。

#### 算法 1：在已见攀岩状态上进行均匀探索

```
1:  for iteration = 1, 2, ..., I_max do
2:      S = {s_0}                              // 初始攀岩状态分布
3:      while 训练预算 N_D 未超出 do
4:          RandomizeScene                     // 随机化场景
5:          从 S 中均匀采样 s
6:          若所有肢体均已连接，则在墙上随机重定位 s（见图 3）
7:          按第 IV-B 节采样 σ_d
8:          从 s 运行策略 π_θ，直至回合在 s_e 结束（见第 IV-A4 节）
9:          若任一只手处于连接状态，则 S = S ∪ {s_e}
10:     基于观测到的经验元组 [o_t, a_t, r_t, o_{t+1}] 用 PPO 更新 θ
```

![图 3](paper/figs/p5_0.png)

**图 3**：在选择新的期望 stance 之前（算法 1 第 7 行），攀岩者会在攀岩墙上被随机重定位，以获得更多样的动作。攀岩者未使用的那些攀岩点位置也会被随机扰动。

### D. 动作参数化

我们同时比较**单步**（single-step）与**多步**（multi-step）动作。多步动作会把策略输出的每个动作仿真 **L 步**，而不是只仿真 1 步。与单步动作相比，在**相同的仿真时间步总量**下，使用多步动作会产生**更少的经验元组**。

---

## 五、实验

本节说明我们对攀岩者的训练实验。我们用 3 种不同设定训练攀岩者：

1.使用 **1 步动作**，并按算法 1 探索攀岩动作（即第 IV-C2 节的 **SS-ESI**）。
2. 使用 **1 步动作**，并按**基线**方法探索攀岩动作（第 IV-C1 节）。
3. 使用 **4 步动作**（第 IV-D 节），并按算法 1 探索攀岩动作。

前 2 种训练设定用于比较第 IV-C1 与 IV-C2 节的状态初始化策略。第 IV-C2 节的初始化策略（算法 1，SS-ESI）表现更好，因此我们在测试 4 步动作时使用了它。

### A. 训练参数

表 I 给出实验中训练攀岩者所用的参数值。第一列是用于探索策略、动作参数化以及奖励函数设计的参数；第二列定义 PPO 的参数值。$I_{max}$、$N_D$ 与 $T_{episode}$ 分别是算法 1 中使用的最大训练迭代数、经验预算与训练回合中的最大步数。$\alpha$ 是 PPO 梯度下降更新中使用的初始学习率。$\beta$ 是 PPO 的熵损失权重，$\epsilon$ 是裁剪代理损失（clipped surrogate loss）参数。$\gamma$ 是奖励折扣因子，$\lambda$ 是正则化参数。

**表 I：训练中使用的参数**

| 名称 | 取值 | 名称 | 取值 |
|---|---|---|---|
| $r_{body}$ | 130 cm | $N_D$ | 20480 |
| $\tau_{max}$ | 250 N·m | $T_{episode}$ | 5 秒 |
| $c_\sigma$ | 4 | Minibatch size | 2048 |
| $c_\tau$ | 3.6 | Num. epoch | 3 |
| $k_\sigma$ | 0.8 / L | $\beta$ | 5.0e−3 |
| $k_\tau$ | 0.2 / L | $\gamma$ | 0.995 |
| $T_{baseline}$ | 50 秒 | $\epsilon$ | 0.2 |
| $I_{max}$ | 1e7 | $\lambda$ | 0.95 |
| $\alpha$ | 1.0e−4 | | |

价值网络与策略网络均有 **3 个隐藏层、每层 256 个单元**，激活函数为 **tanh**。

### B. 训练过程与课程学习

为了训练攀岩者一次移动多于一个肢体，我们把"采样到需要 **2 肢移动**的目标 stance"的概率，随着训练进行到最大迭代数的一半，**从 0% 逐渐提升到 50%**。

训练期间我们使用**衰减学习率**：从表 I 指定的初始值开始，衰减到 2.5e−7。不过，我们把训练划分为 **10 个片段**，每个片段占最大训练迭代数的 10%，并在每个片段开始时把学习率**重启**到初始值。这一过程有助于更好地学习攀岩动作（见图 4）。重启学习率的动机是：由于课程学习（curriculum learning）导致优化地形（optimization landscape）逐渐变化，需要让策略优化能够**重新收敛**。

### C. 评估设定

所有训练设定都必须使用**相同的攀岩动作分布**来评估。我们用算法 1 收集 **1k 个评估回合**数据（即成功或失败的攀岩移动），在训练期间**每 750k 仿真步**保存的模型上进行评估。这确保了评估分布既多样又具挑战性。此外，式(2)与式(3)中的 $k_\sigma$ 与 $k_\tau$ 在所有实验的评估中均设为 0.8 与 0.2。

---

## 六、结果

这里我们在**平均成功率**与**累积奖励**两方面比较各训练设定。我们用各自的探索策略与动作参数化训练模型，并对每 750k 训练迭代保存的每个模型收集 1k 个攀岩动作样本，采用上述评估设定。由于结果在一定程度上依赖随机种子，我们把实验**重复 5 次**。

![图 4](paper/figs/p6_0.png)

**图 4**：每 1500 万经验元组重启学习率的效果。其动机是：由于课程学习使优化地形逐渐变化，需要让策略优化能重新收敛。（红线为重启学习率，蓝线为不重启；纵轴为回合回报Episode Return。）

### A. 探索方法的可视化

图 5 展示了回合初始状态采样如何影响探索。在 1500 万经验元组之前，策略在执行成功攀岩动作上失败很多。

对于**基线**方法——失败后攀岩重置回 T 字姿态——**腿部移动在 1500 万经验元组之前几乎完全没有被探索**；并且随着训练继续，学到的策略探索出的是**手部与脚部移动分布不均衡**的结果。

而 **SS-ESI**（算法 1）在**所有训练迭代期间**都能对手部与脚部移动进行**远为均匀的探索**。

![图 5](paper/figs/p6_1.png)

**图 5**：我们提出的自监督回合状态初始化（SS-ESI）方法对训练期间成功过渡多样性的影响。绿点与蓝点分别表示攀岩者局部坐标系下手部与脚部的成功过渡。（上行为 SS-ESI，下行为 Baseline；横向依次为 1.5M / 7.5M / 15M / 75M / 150M 经验量。可见基线下蓝色的脚部移动长期严重缺失。）

### B. 定量结果

图 6 与图 7 展示了使用基线方法与所提出的自监督回合状态初始化（SS-ESI）技术的学习曲线。

正如预期，**基线在初期学习慢得多**，尽管图 6 中的差距随训练推进而缩小。此外，图 7 表明：**1 步基线方法在执行攀岩动作上的成功率低于我们的方法**，并且**这一差距不会随训练推进而消失**。我们可以得出结论：在攀岩这类复杂且长动作序列的问题中，**SS-ESI 是有帮助的**。

![图 6](paper/figs/p6_2.png)

**图 6**：来自 5 次独立训练运行的回合回报（episode return）均值与标准差。（品红：4 步动作 + 算法 1；黑色：1 步动作 + 算法 1；蓝色：1 步动作 + 基线。）

![图 7](paper/figs/p6_3.png)

**图 7**：来自 5 次独立训练运行的回合成功率（episode success rate）均值与标准差。（品红：4 步动作 + 算法 1；黑色：1 步动作 + 算法 1；蓝色：1 步动作 + 基线。）

进一步地，**使用多步动作是推荐的选择**：图中显示它带来更快的学习与更高的回合回报。作为额外好处，多步动作产生**更平滑、视觉上更悦目**的动作。对动作的目视检查表明，多步动作导致**更宽的动作弧线**（wider movement arcs），因而对攀岩者状态有更广的探索，这或许可以解释其更好的性能。

另一方面，增加每个动作的仿真步数会使**收集经验的代价更高**。如果步数增加过多，还会限制动作的复杂度与时间分辨率。不过我们的实验表明：与"在每个仿真步都采取新动作"的基线相比，**适度增加每动作步数是有益的**。

---

## 七、结论、局限性与未来工作

我们把仿真人形攀岩问题形式化为一个强化学习问题。实验表明：

1. 我们提出的**自监督回合状态初始化**方法能改善探索并加速学习；
2. 使用**多步动作**同样能改善学习，并带来视觉上更平滑、更真实的动作。

因此，我们能够为交互式物理仿真攀岩产出一个**高效的神经网络策略**。这在游戏以及真实攀岩的认知练习[25]两方面都有应用价值；而该回合状态初始化方法可能对**其他需要在无参考数据条件下训练长序列复杂动作**的动作问题也有用。

未来工作要解决的一个主要局限是：我们目前**没有建模手指与攀岩点的细节**。我们也把实验限制在**同时只移动 1 或 2 个肢体**。虽然这已能建模现实攀岩中的大量动作，但要仿真高技术水平的攀岩，还需要处理**动态的全身跳跃**（dynamic full-body leaps）。我们相信可以通过在训练课程中引入跳跃动作来扩展我们的方法，尽管这很可能需要多得多的仿真经验，以及/或者比 PPO 更高效的策略优化方法。这方面的好候选包括 **PPO-CMA**[12]、**SAC**[5] 或**相对熵正则化策略迭代**[26]。

---

## 致谢

本工作由芬兰科学院（Academy of Finland）资助，项目号 299358 与 305737。

---

## 参考文献

1. V. Mnih, K. Kavukcuoglu, D. Silver, A. A. Rusu, J. Veness, M. G. Bellemare, A. Graves, M. Riedmiller, A. K. Fidjeland, G. Ostrovski et al., "Human-level control through deep reinforcement learning," *Nature*, vol. 518, no. 7540, pp. 529–533, 2015.
2. A. Juliani, V.-P. Berges, E. Vckay, Y. Gao, H. Henry, M. Mattar, and D. Lange, "Unity: A general platform for intelligent agents," arXiv preprint arXiv:1809.02627, 2018.
3. X. B. Peng, P. Abbeel, S. Levine, and M. van de Panne, "Deepmimic: Example-guided deep reinforcement learning of physics-based character skills," *ACM Transactions on Graphics (TOG)*, vol. 37, no. 4, p. 143, 2018.
4. L. Liu and J. Hodgins, "Learning to schedule control fragments for physics-based characters using deep q-learning," *ACM TOG*, vol. 36, no. 3, p. 29, 2017.
5. T. Haarnoja, A. Zhou, P. Abbeel, and S. Levine, "Soft actor-critic: Off-policy maximum entropy deep reinforcement learning with a stochastic actor," arXiv preprint arXiv:1801.01290, 2018.
6. D. Silver, A. Huang, C. J. Maddison, A. Guez, L. Sifre, G. Van Den Driessche, J. Schrittwieser, I. Antonoglou, V. Panneershelvam, M. Lanctot et al., "Mastering the game of go with deep neural networks and tree search," *Nature*, vol. 529, no. 7587, p. 484, 2016.
7. D. Silver, J. Schrittwieser, K. Simonyan, I. Antonoglou, A. Huang, A. Guez, T. Hubert, L. Baker, M. Lai, A. Bolton et al., "Mastering the game of go without human knowledge," *Nature*, vol. 550, no. 7676, p. 354, 2017.
8. T. P. Lillicrap, J. J. Hunt, A. Pritzel, N. Heess, T. Erez, Y. Tassa, D. Silver, and D. Wierstra, "Continuous control with deep reinforcement learning," arXiv preprint arXiv:1509.02971, 2015.
9. J. Schulman, S. Levine, P. Abbeel, M. I. Jordan, and P. Moritz, "Trust region policy optimization," in *ICML*, vol. 37, 2015, pp. 1889–1897.
10. J. Schulman, F. Wolski, P. Dhariwal, A. Radford, and O. Klimov, "Proximal policy optimization algorithms," arXiv preprint arXiv:1707.06347, 2017.
11. T. Haarnoja, A. Zhou, K. Hartikainen, G. Tucker, S. Ha, J. Tan, V. Kumar, H. Zhu, A. Gupta, P. Abbeel et al., "Soft actor-critic algorithms and applications," arXiv preprint arXiv:1812.05905, 2018.
12. P. Hämäläinen, A. Babadi, X. Ma, and J. Lehtinen, "PPO-CMA: Proximal policy optimization with covariance matrix adaptation," arXiv preprint arXiv:1810.02541, 2018.
13. A. Abdolmaleki, J. T. Springenberg, Y. Tassa, R. Munos, N. Heess, and M. Riedmiller, "Maximum a posteriori policy optimisation," arXiv preprint arXiv:1806.06920, 2018.
14. A. Abdolmaleki, J. T. Springenberg, J. Degrave, S. Bohez, Y. Tassa, D. Belov, N. Heess, and M. Riedmiller, "Relative entropy regularized policy iteration," arXiv preprint arXiv:1812.02256, 2018.
15. Y. Tassa, Y. Doron, A. Muldal, T. Erez, Y. Li, D. d. L. Casas, D. Budden, A. Abdolmaleki, J. Merel, A. Lefrancq et al., "Deepmind control suite," arXiv preprint arXiv:1801.00690, 2018.
16. G. Brockman, V. Cheung, L. Pettersson, J. Schneider, J. Schulman, J. Tang, and W. Zaremba, "OpenAI Gym," arXiv preprint arXiv:1606.01540, 2016.
17. T. Bretl, "Motion planning of multi-limbed robots subject to equilibrium constraints: The free-climbing robot problem," *The International Journal of Robotics Research*, vol. 25, no. 4, pp. 317–342, 2006.
18. B. Libeau, A. Micaelli, and O. Sigaud, "Transfer of knowledge for a climbing virtual human: A reinforcement learning approach," in *ICRA'09*, IEEE, 2009, pp. 2119–2124.
19. S. Jain, Y. Ye, and C. K. Liu, "Optimization-based interactive motion synthesis," *ACM TOG*, vol. 28, no. 1, p. 10, 2009.
20. I. Mordatch, E. Todorov, and Z. Popović, "Discovery of complex behaviors through contact-invariant optimization," *ACM TOG*, vol. 31, no. 4, p. 43, 2012.
21. K. Naderi, J. Rajamäki, and P. Hämäläinen, "Discovering and synthesizing humanoid climbing movements," *ACM Trans. Graph.*, vol. 36, no. 4, pp. 43:1–43:11, Jul. 2017.
22. K. Naderi, A. Babadi, and P. Hämäläinen, "Learning physically based humanoid climbing movements," in *Computer Graphics Forum*, vol. 37, no. 8, Wiley, 2018, pp. 69–80.
23. T. Olsen, S. Andrews, and P. Kry, "Computational Climbing for Physics-Based Characters," *ACM SIGGRAPH / Eurographics SCA (posters)*, 2014.
24. S. Tonneau, P. Fernbach, A. D. Prete, J. Pettré, and N. Mansard, "2PAC: Two-point attractors for center of mass trajectories in multi-contact scenarios," *ACM TOG*, vol. 37, no. 5, p. 176, 2018.
25. K. Naderi, J. Takatalo, J. Lipsanen, and P. Hämäläinen, "Computer-aided imagery in sport and exercise: A case study of indoor wall climbing," in *Proceedings of Graphics Interface 2018*, GI 2018, pp. 93–99.
26. A. Abdolmaleki, J. T. Springenberg, J. Degrave, S. Bohez, Y. Tassa, D. Belov, N. Heess, and M. A. Riedmiller, "Relative entropy regularized policy iteration," *CoRR*, vol. abs/1812.02256, 2018.

---

*翻译整理：WorkBuddy | 原文版权归 IEEE 及原作者所有，本译文仅供学习阅读之用。*
