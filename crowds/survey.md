# LLM 对话语义驱动的人群模拟 · 跨社区论文逐篇综述

> 检索范围：AI 会议（NeurIPS / ICLR / AAAI / CHI / UIST / EMNLP）、图形学会议（SIGGRAPH / TOG / SCA / Eurographics / CGF / CVPR）、机器人会议（ICRA / IROS / RA-L / T-RO / Science Robotics）、人群模拟专门会议（Motion in Games / Crowd Simulation Workshop）。
> 整理日期：2026-08-11
> 专题重点：**LLM 如何通过对话/语义推理直接驱动人群智能体的运动决策**，以及文本→人群行为的端到端生成框架。

---

## 摘要与领域全景

"LLM 对话语义驱动的人群模拟"是一个在 2024–2026 年集中爆发的交叉方向。核心观察是：**人类在人群中的导航和移动，往往受到复杂的社会交互和环境因素的影响，而这些交互很大程度上由语言和对话驱动**。传统人群模拟方法（社会力模型 SFM、ORCA/RVO 等）只处理低层运动——避碰、路径跟随、编队——导致 agent 运动呈现"机器人式"的独立移动，缺乏自然聚集、解散和信息传播等涌现行为。

研究因此分裂为四条相对独立但问题结构高度同源的技术脉络：

| 脉络 | 核心问题 | 代表社区 | 代表工作 |
|---|---|---|---|
| A. LLM 直接驱动运动决策 | 用 LLM 作为 agent 的大脑，根据对话+感知输出导航指令 | EMNLP / arXiv | Emergent Crowds (2025), PEBA-PEvo (EMNLP'25) |
| B. 文本→人群动画合成 | 从文本描述端到端生成人群行为场/轨迹 | SIGGRAPH / TOG | Text-Crowd (SIGGRAPH'24), Gen-C (TOG'25) |
| C. LLM 社会仿真基础架构 | 为大规模 LLM agent 提供空间环境中的认知架构 | UIST / EMNLP | Generative Agents (UIST'23), PersonaEvolve (EMNLP'25) |
| D. 语义引导行人仿真（机器人侧） | 用自然语言生成复杂社会场景用于社交机器人训练 | RA-L / arXiv | GROVE (2026), HuNavSim 2.0 (RA-L'23) |

**一句话总结现状**：A 脉络已证明 LLM 对话可以产生涌现的群体行为（聚集/解散/信息传播），但规模仅 30–80 agent；B 脉络实现了文本→人群行为的端到端生成，但不涉及 agent 级语义推理；C 脉络奠定了 LLM agent 在空间环境中进行社会推理的基础架构，并提出了行为真实性对齐的理论框架；D 脉络将文本→行人仿真的研究推进到机器人训练应用。**四条线尚未真正打通**——这正是当前最大的机会所在。

---

## A. LLM 直接驱动运动决策

这条线的核心设计模式是：**每个 agent 配备一个 LLM，周期性查询该 LLM 以生成对话和运动决策，底层用经典 steering 算法执行**。

### A1. Emergent Crowds Dynamics from Language-Driven Multi-Agent Interactions

- **出处**：arXiv:2508.15047, 2025（投稿中，cs.AI + cs.GR）
- **作者**：Yibo Liu, Liam Shatzel, Brandon Haworth, Teseo Schneider（University of Victoria）
- **项目页**：https://yiboliu.github.io/EmergentCrowds/ ｜ **代码已开源**：https://github.com/YiboLiu/EmergentCrowds

**问题设定**：现有去中心化 agent-based 人群模拟遵循三阶段架构——低频全局目标决策 → 中频路径规划 → 高频 steering 转向。但在非紧急情况下，人类运动主要由**社会交互、个人动机和局部环境可供性**驱动，而非固定的全局目标。现有方法不建模这些维度，导致 agent 运动"机器人化"——各自独立移动，从不自然地聚集或基于他人存在重新规划路径。

**方法——两组件架构**：

**(1) Agent 状态表示**（Figure 2）：
每个 agent a_i 拥有：
- **固定档案 P_i**：ID + MBTI 人格类型（16 种标准类别之一）+ 朋友列表 F_i = {f_ij}（友谊不必对称）
- **动态状态 s_i**：初始为空，随时间更新（如 "Increased anxiety due to witnessing the art accident"）
- **位置 ℓ_i**、**感知 p_i**（图像 I_i + 对话内容）、**运动参数 m_i**（目标 g_i + steering 参数 d_i）、**历史 h_i**（存储过去 k=50 个事件的时间戳+四类动作：position/dialog/vision/movement，FIFO 队列）

**(2) Steering 参数向量**：
d_i = {γ_i, φ_i, κ_i, α_i, κ^n_i, ρ_i, f^max_i, v^max_i}，具体为：
- γ_i：path-following 权重（朝向 waypoint w）
- φ_i：PAM 避碰权重（Predictive Avoidance Model, Karamouzas et al. 2009）
- κ_i：cohesion 权重（向邻居靠拢）
- α_i：alignment 权重（与邻居速度对齐）
- κ^n_i：cohesion 邻居列表
- ρ_i：separation 权重（避免碰撞）
- f^max_i, v^max_i：最大力和最大速度

总力计算：
```
F_i = min(f^max_i, f^φ_i + f^κ_i + f^α_i + f^ρ_i + f^o_i)
v^{t+1}_i = min(v^max_i, v^t_i + Δt·F_i)
ℓ^{t+1}_i = ℓ^t_i + Δt·v^{t+1}_i
```
其中分离力 f^ρ_i = ρ_i · Σ_{j∈N_i} (ℓ_i - ℓ_j) · exp(ρ_r / ‖ℓ_i - ℓ_j‖)，ρ_r = 0.4，障碍物力权重 W_o = 10。

**(3) 对话分组**：收集 1.5 米内的 agent 为一组，最大组大小 g_max = 6（实验发现覆盖所有场景）。

**(4) 对话系统**：对每组 G_i 调用 LLM，输入当前时间 t + 全局状态 S（环境描述 + 所有地点/目标名称与位置）+ 组内每个 agent 的完整档案，输出有序对话列表 [(agent_id, dialog_text)]。

**(5) 语言驱动运动规划器**：对每个 agent 调用 LLM，输入 agent 档案 + 当前时间 + 全局状态 + 视觉图像描述 + 当前对话 + 历史记忆，输出：① 是否改变目标 g_i；② 更新 steering 参数 m_i；③ 新状态 s'_i；④ 决策过程摘要（写入历史）；⑤ 图像简短描述（写入历史）。

**关键技术选择**：
- **LLM 调用频率**：每 k=100 帧调用一次（因 LLM 慢），仿真 50 FPS → 约每 2 秒一次 LLM 决策
- **Chain-of-thought**：强制 LLM 先描述图像再分析输出，显著提升结果质量
- **输出格式约束**：指定 JSON schema + role + task + instructions，格式错误则跳过该帧（<10 次/全部实验）
- **无训练**：纯 inference-time LLM 调用，零训练成本

**实验场景与定量设置**：

**场景 1：大学社团招新展（University Club Fair）**
- 50 个 agent（30 可动 + 20 静态工作人员），MBTI 随机分配
- 关键设定：**只有 7 个工作人员知道国际中心发免费 T 恤**
- 仿真 50 FPS，每 100 帧调用 LLM
- 硬件：NVIDIA 3090 GPU + Intel i9 CPU + Ubuntu

**涌现行为观察**：
1. **通过隐式运动参数的聚集**（Figure 4）：a_36 前往皮划艇俱乐部 → 与工作人员 a_2 和 a_22 对话 → a_2 告知免费 T 恤信息 → **系统自动调整双方 steering 参数**：
   - a_22：κ_22 从 0.0 → 1.0，a_36 加入其 cohesion 邻居 κ^n_22
   - a_36：κ_36 从 0.0 → 0.8，α_36 从 0.0 → 0.7（跟随 a_22），a_22 加入 κ^n_36
   - 两人全程并肩走到国际中心
2. **软聚集（Soft Grouping）**（Figure 6）：两个朋友 a_24（ENFJ，外向）和 a_34（ISTJ，内向目标导向）相遇聊天但最终决定不一起走——LLM 推理明确引用了人格差异
3. **通过对话耦合的自发聚集**（Figure 7）：T 恤信息通过对话链式传播，原本均匀分布的 agent 自然汇聚到国际中心
4. **四人组形成**（Figure 5）：a_42 主动请求加入三人组，群体同步目标和步行行为

**场景 2：博物馆画作坠落事故**
- 30 个 agent，每人随机分配艺术偏好
- Frame 799 引入突发事件："The Starry Night" 画作坠落碎裂

**涌现行为观察**：
1. **多样化未脚本反应**：部分 agent 走向事故现场（受对话驱动），部分继续欣赏其他画作（如 a_20 刚到达 "Liberty Leading the People"，LLM 判断无需离开）
2. **语言影响运动决策**（Figure 10）：a_19 原计划去看 "The Starry Night"，遇到 a_10/a_25/a_29 讲了个关于碎画的冷笑话后改变主意，转而和他们一起去 "The Great Wave off Kanagawa"
3. **报告行为**（Figure 9）：a_2 目睹事故后状态变为 "Increased anxiety"，自主决定前往入口报告工作人员
4. **聚集-解散动力学**（Figure 11）：Frame 724 均匀分布 → Frame 1199 走向事故 → Frame 1484 聚集围观 → Frame 2519 逐渐解散

**局限（作者自述）**：
1. 使用分布式 LLM API，**每次查询约 5 秒**，系统整体很慢
2. 底层 navigation 和 steering 可能冲突，agent 在接近目标时难以停止
3. 未做消融实验验证"涌现"声称——审稿人指出如果去掉所有社交 prompt（相同个性、无关系、无共享愿望），群体是否仍会形成？这是区分真正涌现 vs prompt 转录的关键测试
4. 无定量指标对比 baseline（纯定性关键帧图 + 对话日志）

**信息缺口**：使用的具体 LLM 型号和参数量未披露；prompt 模板全文未在正文中给出（标注在 Supplemental LABEL:app:format）；token 消耗和单次运行成本未报告；steering 参数的默认值和取值范围未给出。

**为什么这篇最重要**：它是首个将 LLM 对话直接接入 steering 参数控制的完整框架，证明了微观层面的语言-运动交互可以产生宏观涌现行为。但其缺乏消融实验和定量评估，留下了方法论上的开放性。

---

### A2. Implicit Behavioral Alignment of Language Agents in High-Stakes Crowd Simulations (PEBA-PEvo)

- **出处**：**EMNLP 2025 Main Conference**；arXiv:2509.16457
- **作者**：Yunzhe Wang, Gale M. Lucas, Burcin Becerik-Gerber, Volkan Ustun（USC + USC Institute for Creative Technologies）
- **项目页**：https://hats-ict.github.io/peba-asi/ ｜ **代码已开源**：https://github.com/HATS-ICT/PEBA-ASI ｜ **在线 Demo**：https://persona-evolve.vercel.app/

**问题定义——Behavior-Realism Gap**：
LLM-driven generative agent 的个体行为看似合理，但**群体层面的行为分布偏离专家预期或真实世界数据**。这是一个系统性问题：Generative Agents 类方法在扩展到复杂动态环境时，agent 常表现出过度合作、对不确定性反应不当、或群体行为模式不真实等现象。

**理论框架——PEBA（Persona-Environment Behavioral Alignment）**：
- 理论基础：**Lewin 行为方程 B = f(Person, Environment)**
- 形式化为**分布匹配问题**：最小化模拟行为分布与专家启发分布之间的 KL 散度
- 核心洞察：行为由人格（persona）和环境共同决定，因此可以通过迭代优化 persona 来隐式对齐行为

**算法——PEvo（PersonaEvolve）**：
1. **初始化**：N 个 agent 各有一个初始 persona（含个性字段如情绪韧性、风险承受力等）
2. **仿真**：每个 agent 用 ReAct-style LLM 架构自主观察、更新记忆、推理生成轨迹
3. **行为分类**：LLM classifier 将每条轨迹映射为高层行为类别
4. **分布计算**：统计模拟 crowd 的行为类别分布
5. **KL 散度计算**：对比模拟分布与专家基准分布
6. **Persona 选择**：识别对分布差异贡献最大的 agent
7. **Persona 更新**：LLM persona writer 修改选中 agent 的人格字段（如将保安改为 "former combat medic with high assertiveness"）
8. **迭代**：重复步骤 2-7 直到收敛

**关键技术细节**：
- **Unity 仿真环境**：学校布局，80 个 civilian agent + 1 个枪手
- **Agent 架构**：ReAct-style LLM，partial observability（仅感知局部环境和对话）
- **行为类别**：Hide / Run / Fight / Freeze 等（具体类别表待查论文正文）
- **Expert benchmark**：来自动作事件领域的专家标注的行为分布（详见 EVOLVE 前期工作）
- **四个 LLM 模型测试**：GPT-4.1 Mini / GPT-4o Mini / Gemini 2.5 Flash / DeepSeek-V3

**定量结果**：
| 指标 | 数值 |
|---|---|
| 相比 No Enforcing 基线的分布差异降低 | **84%**（从 0.47 降至 0.16） |
| 相比 Explicit Enforcing 基线的改进 | **34%** |
| 跨环境迁移到新场景（办公室）的 KL 降低 | **97.5%** |
| 迁移 persona 保留的优化收益比例 | **57%** |
| 完整 15 轮优化的成本 | **<$10**（最贵模型） |
| 单轮 token 成本 | GPT-4.1 Mini ~$0.60, DeepSeek-V3 ~$0.20 |
| Prompt token 占比 | **>95%** 总 token 消耗 |
| 成本效率比（KL 改善/$） | DeepSeek-V3 和 Gemini 2.5 Flash 约为 GPT 变体的 **3 倍** |
| 7 轮优化达到最终对齐的 | **90%** |

**三个 Baseline 对比**：
1. **No Enforcing**：agent 完全按初始 persona 行动，暴露原始 behavior-realism gap
2. **Explicit Enforcing**：直接指令 agent 执行特定行为（如 "always choose Hide"），损害情境真实性
3. **Implicit Enforcing (PEvo)**：迭代优化 persona 隐式诱导目标行为

**跨环境泛化实验**：
将在学校环境中优化的 persona 迁移到全新的办公楼场景。转移后的 persona 相比未优化基线实现 97.5% KL 降低，保留了 57% 的优化收益。完全重新训练的 persona 略优，但证明了有效跨环境泛化。

**收敛性分析**：
KL 散度在前 7 轮快速下降（达到 90% 可达成对齐），之后趋于平缓。DeepSeek-V3 和 Gemini 2.5 Flash 的单位美元 KL 改善最高。

**失败模式**：
- 偶尔出现 "freezing" 行为（agent 僵住不动）
- 环境感知有时不准确
- RL 类显式指令方法虽然情境适当但过于僵化

**与 Emergent Crowds 的关键区别**：
Emergent Crowds 关注"能否产生涌现行为"（定性展示），PEBA-PEvo 关注"产生的行为是否真实可信"（定量对齐）。前者无训练/优化，后者有系统的 persona 优化闭环。两者互补。

**信息缺口**：具体的 persona 字段结构和更新前后的示例对比在项目页上有但论文正文细节需查 PDF；LLM classifier 的具体 prompt 模板和准确率未公开；expert benchmark 的具体获取流程（多少专家、什么协议）需查 EVOLVE 前期工作。

---

### A3. Large Language Model–Enhanced Agent-Based Modeling for Intelligent Crowd Evacuation under Disaster Scenarios

- **出处**：EGU General Assembly 2026, EGU26-3725
- **作者**：Sen Yang, Yi Zhang, Chen Gu（清华大学土木工程系 + 东南大学）

**问题**：灾害疏散仿真中，传统基于规则的 agent 行为模型缺乏适应性和上下文感知能力。

**方法**：
- 每个 agent 维护内部状态（个性属性、环境感知、决策历史）
- LLM 作为决策组件嵌入疏散仿真，支持基于演化情境上下文的自适应推理和通信
- **批处理 prompting + 并行计算策略**缓解 LLM 集成的计算成本
- 支持行人 + 车辆多模态疏散动力学

**结果**：在真实世界灾害疏散场景中评估，LLM-enhanced agent 表现出比传统规则模型更灵活、上下文感知更强、更逼真的行为模式。

**信息缺口**：具体定量指标、baseline 对比、LLM 型号、agent 规模均未在摘要级别信息中找到。这是一篇会议摘要而非全文论文，技术深度有限。

---

## B. 文本/语义引导的人群动画合成

这条线的核心设计模式是：**用扩散模型/图 VAE 从文本描述直接生成人群行为场或轨迹，不涉及 agent 级 LLM 推理**。

### B1. Text-Guided Synthesis of Crowd Animation (Text-Crowd)

- **出处**：**ACM SIGGRAPH 2024 Conference Papers**；DOI: 10.1145/3799902.3811062
- **作者**：Xuebo Ji, Zherong Pan, Xifeng Gao, Jia Pan（香港大学）
- **代码**：https://github.com/MLZG/Text-Crowd

**问题**：现有 crowds 方法集中于低层导航（避碰、路径跟随），无法从文本描述自动生成具有高层语义的人群动画。

**方法——两个条件扩散模型**：

**(1) 起始-目标分布扩散模型**：
- 输入：文本描述 + 环境布局
- 输出：text-guided 的 agent 起点密度场和终点密度场
- 将文本语义转化为空间分布

**(2) 速度场扩散模型**：
- 输入：起始-目标分布 + 环境
- 输出：随时间演化的速度场 v(x,t)
- 控制多组 agent 的运动

**LLM 文本规范化**：
- 将通用脚本（如 "人们在广场上散步聊天"）规范化为结构化句子
- 提升训练稳定性和可扩展性
- 减少语言多样性导致的训练噪声

**建设性数据生成**：
- 自动生成随机环境和人群动画作为训练数据
- 解决真实 crowds 数据稀缺问题

**Baseline 对比**：
- 主要对比 ORCA-based 方法和纯 steering 方法
- 在未见过的环境和新颖场景描述上均能生成合理人群动画

**局限**：
- 不涉及 agent 级语义推理
- 行为局限于扩散模型训练分布
- 无法处理动态突发事件

**被 GROVE 用作 baseline**：GROVE 复现了 Text-Crowd 并在三个 preset 下全面超越（见下文 D1）。

---

### B2. Gen-C: Populating Virtual Worlds with Generative Crowds

- **出处**：arXiv:2504.01924, 2025（TOG 发表中）
- **作者**：Andreas Panayiotou, Panayiotis Charalambous, Ioannis Karamouzas（塞浦路斯大学 + UC Riverside）

**问题**：现有 crowds 方法集中于低层任务（避碰、路径跟随、编队），难以捕捉长期 agent-agent 和 agent-environment 交互产生的高层行为。直接从真实视频收集和标注人群数据成本高、泛化差。

**方法——三阶段流水线**：

**(1) LLM Bootstrapping 合成数据集**：
- 100 个种子句 S_in（如 "Around the campus park, students sit to have lunch, talk with others, and wave at passing friends."）
- 每个种子句用 GPT-4.1 生成 50 个 paraphrase → **5000 个文本输入/主题**
- 两个主题：University Campus + Train Station
- **两阶段查询**：
  - Q1：生成环境布局（Locations：coffee shop, park, entrance 等，含类别/位置/尺度）
  - Q2：生成 agent 行为和交互序列（事件列表）
- 输出：每个 agent 的生命周期记录 Rec_i（动作序列 + 交互）

**Action 词汇表**（13 个动作）：
wait, sit, wander, queue, enter/exit, wave at, discuss, meet, object interact, talk on phone, read, look at, carry

**Location 类别**（8 个）：
building, room, entrance, exhibit, furniture, outdoor area, item, service area

**(2) 人群场景图（Crowd Scenario Graph）表示**：
- **时间展开图**结构
- 节点 V^t_i = (i, A^t_i, L^t_i)：agent ID + 动作 + 位置 + 时间步
- **Sequence 边** E^t_i = (V^t_i, V^{t-1}_i)：同一 agent 的时间连续性
- **Share 边** E^t_{i,j} = (V^t_i, V^t_j)：不同 agent 在同一时刻共享动作（交互）
- 邻接矩阵编码：0=无边, 1=sequence 边, -1=share 边
- 子图 H_k 对应一个 agent 群体（通过 share 边连接的 agent 链）

**(3) 节点特征**（49 维）：
- Action 嵌入：16 维（13 个动作类别）
- Location 嵌入：8 维
- Agent ID 嵌入：16 维
- 归一化时间索引：1 维标量
- 全局索引：4 维正弦位置编码
- Laplacian 位置编码：top-4 特征向量

**(4) 图级条件向量 C**：
- 归一化节点数/agent 数/事件数（最大值 100/15/30）
- Action 频率向量（13 维）
- S_in 的 Sentence-BERT 嵌入（all-mpnet-base-v2, 768 维）
- 全部投影到 128 维 → 32 维条件向量

**(5) 双 VGAE 架构**（Figure 3）：
- **共享 Encoder**：2 层 GINE，hidden dim 128，dropout 0.2
  - h^(k)_v = h^(k-1)_v + GINE^(k)(h^(k-1)_v, {(h^(k-1)_u, e_uv)})
  - Global add pooling → LayerNorm → 拼接条件向量 C → FC 层 → μ, log σ²
- **VGAE-S（结构解码器）**：
  - [Z_S; C] → 3 层 MLP (dim 128) → tanh → [-1,1] 邻接矩阵
  - A_hat = U + U^T, U_ij = tanh(f_θ([Z_S; C])_ij)
  - SmoothL1 重建损失
- **VGAE-F（特征解码器）**：
  - [Z_F; C] + 位置编码 → 2 层 GINE (dim 128) → collapse 到连通分量 → softmax → action/location 类别分布
  - Cross-entropy 重建损失，γ=1.5 平衡 action 权重
- **条件先验网络**：p(Z|C) 替代固定 N(0,I)，防止 posterior collapse
- **KL 正则化**：L_KL = E_z[log q(Z|X,C) - log p(Z|C)]
- **β 循环退火**：VGAE-S β∈[0.001,3], VGAE-F β∈[0.001,1]，100 epoch 周期
- **Latent dim = 32**（两个 VGAE 相同）

**训练配置**：
- 500 epochs, batch 128, lr 5e-4 → 1.25e-4 线性衰减, Adam weight decay 3e-4
- 单张 NVIDIA RTX 4070 Ti GPU，每模型约 1 小时
- 数据集：每主题 500 场景 → University Campus 22.9k 子图, Train Station 25.1k 子图（80/15/5 划分）

**定量结果**：

**消融实验（KL 散度 ↓）**：
| Metrics | University Campus | Train Station |
|---|---|---|
| | w/o Can. | Single | Rand. | **Gen-C** | w/o Can. | Single | Rand. | **Gen-C** |
| Degree | 3.496 | 0.062 | 0.595 | **0.056** | 3.289 | 0.051 | 0.468 | **0.033** |
| CC | 0.955 | 0.034 | 0.221 | **0.031** | 0.147 | 0.053 | 0.438 | **0.038** |
| Diameter | 3.463 | 0.145 | 0.494 | 0.156 | 3.649 | 0.094 | 0.804 | 0.104 |
| APL | 0.923 | 0.276 | 0.885 | **0.147** | 0.714 | 0.144 | 1.235 | **0.099** |
| Action | 0.180 | 0.358 | 0.225 | **0.177** | 0.128 | 0.340 | 0.368 | **0.106** |
| Location | 0.024 | 0.058 | 0.024 | **0.017** | 0.019 | 0.031 | 0.047 | **0.018** |

关键发现：
- **Canonical ordering 至关重要**：w/o Can. 在结构指标上全面崩溃（Degree KL 3.496 vs 0.056）
- **双 VGAE 优于单 VGAE**：尤其在语义指标上（Action KL 0.358 vs 0.177）
- Random baseline 在特征层面尚可（因为从经验先验采样），但结构指标很差

**LLM 数据 vs 真实数据验证**：
- 30 分钟 YouTube 火车站视频人工标注对比
- Action 转移矩阵高度一致（Δ 很小）
- 归一化熵：H_LLM = 0.949 (CI [0.916, 0.970]) vs H_Real = 0.936 (CI [0.935, 0.939]) —— **置信区间重叠，序列多样性相当**

**LLM vs Gen-C 缩放对比**（Figure 5）：
- LLM 直接生成在 agent 数增加时：action 序列熵单调下降（重复行为增多）、drop rate 上升（无法满足目标 agent 数）、token 消耗和延迟急剧上升
- Gen-C 维持稳定的多样性和低延迟
- GPT-4.1-mini drop rate 0.32, GPT-4.1-nano drop rate 0.94（即使在 A_count=20 时）

**Latent 空间分析**：
| Theme | Setup | FID ↓ | MMD ↓ |
|---|---|---|---|
| Uni. Campus | Structure | 1.950 | 0.078 |
| Uni. Campus | Features | 9.286 | 0.032 |
| Train Station | Structure | 0.534 | 0.058 |
| Train Station | Features | 7.674 | 0.030 |

Cross-domain mix 时 FID/MMD 均退化，证明学到了 domain-specific 分布。

**局限（作者自述）**：
1. 不支持长时程 agent 意图建模
2. **Agent 执行中不能切换动作**，动作持续时间手动定义
3. 行为局限于预定义 13 个动作列表，需重新训练才能加入新动作
4. 未与底层 crowd simulator 集成（纯高层行为规划）

**信息缺口**：代码和数据集承诺发布但未找到公开链接；Q1/Q2 的具体 prompt 在 supplementary material 中未抓取到。

---

## C. LLM 赋能的社会仿真基础架构

### C1. Generative Agents: Interactive Simulacra of Human Behavior

- **出处**：**UIST 2023**（最高引用 UIST 论文之一）
- **作者**：Joon Sung Park, Joseph O'Brien, Carrie Cai, Meredith Morris, Percy Liang, Michael Bernstein（Stanford + Google）

**架构三要素**：
1. **Memory Stream**：以自然语言记录所有经历，带时间戳和重要性评分（recency/relevance/importance 加权检索）
2. **Reflection**：周期性将记忆综合为高层抽象反思（如 "我是咖啡师"）
3. **Planning**：基于记忆和反思生成长期目标和日常计划，动态调整

**Smallville 实验**：
- 25 个 agent 在像素风沙盒小镇中生活
- 仅给定 "一个 agent 想办情人节派对"，agents 自主传播邀请、结交新朋友、约会、准时到场
- 证明了 emergent social behavior 的可能性

**对人群模拟的意义**：
- 虽未直接处理 crowd-level 运动/导航，但奠定了 **LLM agent 在空间环境中进行社会推理** 的基础架构
- 后续几乎所有 LLM-driven crowd 工作都受其启发
- Park 的博士论文（Stanford 2025）进一步扩展到 1000+ agent 的人口级仿真和美国成年人代表性样本的行为对齐（86% 两周后决策复现准确率）

**技术细节**：
- 使用 GPT-3.5（text-davinci-003）
- 每个 agent 独立的 LLM 调用
- 空间环境为 2D 像素网格，agent 移动为离散格子跳转
- 无物理仿真，无 steering 算法

---

### C2. Generative Agent Simulations of Human Behavior（Park 博士论文）

- **出处**：Stanford University PhD Dissertation, 2025
- **作者**：Joon Sung Park
- **导师**：Michael Bernstein, Percy Liang
- **链接**：https://purl.stanford.edu/jm164ch6237

**扩展贡献**：
- **人口级仿真**：1000+ agent 的美国成年人代表性样本
- **行为对齐评估框架**：attitudinal / behavioral / diffusion 三类任务
- **案例研究**：平台设计、内容推荐、公共政策的 prototyping
- **关键数字**：AI 分身复现本人两周后同类决策准确率 **86%**，比传统人口统计标签方法提升 **47%**，种族/性别/政治立场维度预测偏差下降 **62%**

---

## D. 语义引导行人仿真（机器人社区）

### D1. GROVE: Grounded Pedestrian Simulation via Natural Language for Interactive Social Robot Navigation

- **出处**：arXiv:2606.25504, 2026（投稿中，cs.RO）
- **作者**：Duc Tai Nguyen, Volodymyr Shcherbyna, Anh Do Duc, Zhengcheng Shen, Teham Buiyan, Linh Kästner（Singapore Management University + TU Berlin + NUS）

**问题**：现有行人仿真器需要手动配置每个场景的 agent 位置/目标/行为，且对所有 agent 应用统一行为模型，导致 sim-to-real gap 严重。

**方法——三层分层架构**（Figure 2）：

**(1) Strategic Layer — RoI 提取**：
- 定义 world.yaml：人类可读 + LLM 可读的 YAML 格式，编码空间语义信息（zones/rooms/hallways + 结构元素 + 实体 + 语义属性）
- LLM 将用户 prompt 中的语义 token 与 metadata 匹配，过滤无关 zones
- 对每个选中的 RoI 构造结构化摘要（metadata + 边界坐标 + 入口坐标 + 实体信息）

**(2) Tactical Layer — RAG + Behavior Tree 生成**：
- **Behavior Tree 作为中间表示**：选择 BT 而非直接策略生成的原因是层级结构、模块化、确定性执行语义、可解释可验证
- **Node Wrapping Strategy**：自动 ID 管理模块确定性地分配和解析 agent/goal/group 标识符，减轻 LLM 的符号负担
- **新增 FollowVelocityField 节点**：支持可扩展的群体级运动控制（velocity field 控制集体流，BT 保持决策级控制）
- **RAG 模块**：ChromaDB 向量数据库存储节点描述/参数类型/语义约束/模板结构，动态注入任务相关节点规格
  - 动机：完整节点规格放入 prompt 会导致 context 过长和幻觉
  - 初步实验证实：不提供 retrieval 时幻觉节点类型增加且推理延迟上升

**(3) Operational Layer — Theta* 全局规划**：
- 将世界分解为 2D occupancy grid
- **Theta*** 计算近视线无碰撞路径（比 A* 更短更平滑）
- 在 BT 层面分析连续导航目标，插入中间 waypoint
- 保证静态障碍物避免（SFM 无法保证）

**三种 Optimized Presets**：

| Preset | 目标场景 | LLM prompt 偏向 | RAG 限制 | 导航方式 | 禁用的节点 |
|---|---|---|---|---|---|
| **Emergency** | 疏散 | 疏散语义 | 仅 FollowVelocityField | Velocity field（类比 Text-Crowd 训练信号） | 对话/roaming/随意跟随 |
| **Queuing** | 排队等待 | 排队模板 | 优先 queue 节点 | Theta* + 顺序 goal 分配 | 无 |
| **Normal** | 日常混合 | 完整社交 | 全部节点可用 | BT + Theta* | 无 |

**Token 优化**：
| Mode | Vanilla tokens | GROVE tokens | 改善 |
|---|---|---|---|
| Emergency | 14296 | 9250 | **35%** |
| Normal | 14329 | 7323 | **49%** |
| Queuing | 14343 | 8575 | **40%** |

**实验对比**：
- **Baselines**：复现的 Text-Crowd 和 TRACE & PACE
- **环境**：Hospital（主评测）/ Office / Residential
- **评估协议**：VLM-based scoring（GPT-5），三个维度 0-10 分

**定量结果（VLM 评分 ↑）**：
| Preset | Method | Alignment | Plausibility | Visual | Average |
|---|---|---|---|---|---|
| Emergency | TRACE | 2.09 | 5.55 | 4.64 | 4.09 |
| Emergency | Text-Crowd | 2.71 | 5.71 | 4.86 | 4.43 |
| **Emergency** | **GROVE** | **4.50** | **6.67** | 4.83 | **5.33** |
| Normal | Text-Crowd | 2.50 | 6.67 | 5.50 | 4.89 |
| Normal | TRACE | 4.17 | 7.17 | 5.17 | 5.50 |
| **Normal** | **GROVE** | **6.29** | **7.71** | 5.71 | **6.57** |
| Queuing | TRACE | 2.83 | 5.83 | 4.50 | 4.39 |
| Queuing | Text-Crowd | 1.50 | 7.17 | 5.17 | 4.61 |
| **Queuing** | **GROVE** | **6.86** | **7.43** | 5.57 | **6.62** |

**关键定性发现**：
- GROVE 是唯一能生成**结构化排队**的方法（agent 依次排列并逐步前进）
- GROVE 是唯一能产生**stop-and-go 模式**的方法（BT 执行特性）
- Emergency 模式下 velocity field 引导产生连贯的出口流向，无穿墙
- Text-Crowd 的 goal 采样与环境语义解耦，导致几何合法但语义不合理的位置
- TRACE 局部运动平滑但缺乏语义控制，薄墙结构中频繁穿墙

**系统集成**：
- 直接集成到 **Isaac Sim / Gazebo / RViz**
- ROS2 生态系统
- GUI 在 RViz2 内，可实时查看 velocity field / waypoint / agent pose

**局限**：
1. 多 SotA 方法组合的计算成本高
2. 假设静态障碍物地图，不支持动态环境变化
3. 未来计划引入 world-level cache 和自动化世界表示提取

**与本文主题的关系**：GROVE 不直接使用 LLM 对话驱动单个 agent 运动，而是用 LLM 生成 scenario-level 的行为树配置。但它代表了"语义→人群行为"这一范式在机器人社区的成熟形态，且直接与 Text-Crowd/TRACE 做了公平对比。

---

### D2. HuNavSim / HuNavSim 2.0

- **出处**：IEEE RA-L 2023 / arXiv 2025
- **作者**：Noé Pérez-Higueras 等（Universidad Pablo de Olavide）
- **GitHub**：https://github.com/robotics-upo/hunav_sim

**描述**：ROS2 基础的人类导航行为模拟器，基于改进的社会力模型（SFM），支持与 Gazebo/Webots/Morse 对接。

**6 种人对机器人的反应行为**（均由行为树控制）：
1. Regular：把机器人当另一人类
2. Impassive：把机器人当静态障碍物
3. Surprised：停下看机器人
4. Curious：放弃当前目标靠近机器人
5. Scared：远离机器人
6. Threatening：挡在机器人前面

HuNavSim 2.0 扩展了行为树引擎，成为 GROVE 的基础平台。

---

### D3. SocialGAIL: Faithful Crowd Simulation for Social Robot Navigation

- **出处**：**ICRA 2024**, pp. 16873-16880
- **作者**：Bo Ling, Yan Lyu, Dongxiao Li, Guanyu Gao, Yi Shi, Xueyong Xu, Weiwei Wu
- **引用**：17 次（Google Scholar）

**问题**：强化学习社交导航的训练依赖仿真器复制逼真人群行为。现有 crowd simulation 要么依赖手工规则（过于激进），要么依赖人类轨迹演示（泛化差）。

**方法**：
- **GAIL（Generative Adversarial Imitation Learning）**模仿真实行人导航
- **Attention-based 图神经网络**编码观测
- Generator-Discriminator 架构逼近真实行人行为分布
- 提出一套 crowd simulation faithfulness 评估指标

**结果**：在 goal-reaching、中间状态 faithfulness、轨迹 faithfulness、全局轨迹模式遵循方面均优于 baseline。

**与 LLM 工作的关系**：虽然不是 LLM 驱动，但提供了数据驱动逼真人群仿真的重要 baseline，且被 GROVE 引用为相关 work。

---

## E. 相关支撑技术与工具

### E1. TRACE and PACE: Controllable Pedestrian Animation via Guided Trajectory Diffusion

- **出处**：**CVPR 2023**
- **作者**：Davis Rempe, Zhengyi Luo, Xue Bin Peng 等（NVIDIA + Stanford + CMU）

**方法**：
- **TRACE**：轨迹扩散模型，test-time guidance 实现用户约束（waypoints、速度、社交群体）
- **PACE**：物理基础的人形控制器，adversarial motion learning
- **闭环系统**：TRACE 高层规划器 → PACE 低层动画器，频繁 re-planning
- **Value function 引导**：利用 RL 训练中学习的 value function 指导扩散

**意义**：为语义/文本驱动的人群控制提供了底层运动生成基础设施，被 GROVE 用作 baseline 和高保真替代方案。

---

### E2. Social-LLaVA / GSON / CoNVOI 等 VLM 社交导航工作

一系列 ICRA/IROS 2024-2025 的工作，将 VLM/LLM 用于**机器人社交导航**（robot navigating in crowds，非 crowd simulation 本身）：
- Social-LLaVA (IROS 2025)
- GSON (RA-L 2025)
- CoNVOI (IROS 2024)
- OLiVia-Nav (ICRA 2025)
- BehAV (ICRA 2025)
- AutoSpatial (IROS 2025)

这些工作与人群模拟的关系在于：它们需要逼真的人群仿真作为训练/评估环境。

---

## 跨领域综合分析

### 方法论趋同：分层 + 混合离散连续

一个引人注目的观察是，四条脉络在**互不引用或少量引用**的情况下收敛到了几乎相同的架构：

```
高层：语义/语言推理（LLM 对话 / 文本条件 / Behavior Tree）
  ↓ 提供意图/目标/约束
中层：路径/行为规划（A* / Theta* / 扩散模型 / VGAE 图）
  ↓ 提供 waypoint / 速度场 / 行为序列
底层：物理运动执行（SFM / ORCA / PAM / RL tracking）
  ↑ 反馈可行性
```

- Emergent Crowds：LLM 对话 → steering 参数 → SFM+PAM 执行
- PEBA-PEvo：LLM persona → ReAct 推理 → Unity 物理执行
- Gen-C：LLM bootstrapping → 双 VGAE 图 → 高层行为序列
- GROVE：LLM + RAG → Behavior Tree → Theta* + SFM 执行

**这种趋同强烈暗示该架构反映了问题的内在结构**，而非某个社区的偶然偏好。

### 技术路线对比总结

| 维度 | 传统方法(SFM/ORCA) | 数据驱动(GREIL/SocialGAIL) | 扩散模型(Text-Crowd/TRACE) | LLM 驱动(本文核心) |
|---|---|---|---|---|
| **控制粒度** | 低层运动(力/速度) | 中层轨迹模仿 | 高层场/轨迹 | 高层语义决策 |
| **社会交互** | 简单排斥力 | 从数据中学习 | 有限(text条件) | 丰富(对话/个性) |
| **可控性** | 参数调优 | 训练数据决定 | Test-time guidance | Prompt工程 |
| **涌现行为** | 有限(拥堵/瓶颈) | 中等 | 中等 | 丰富(群组/信息传播) |
| **计算成本** | 极低 | 中等(推理) | 中等(采样) | 极高(LLM调用) |
| **可扩展性** | 数千~数万 | 数百~数千 | 数十~数百 | 30~80(当前) |
| **真实性** | 运动学合理 | 统计逼真 | 视觉逼真 | 社会学逼真 |
| **评估体系** | 成熟(密度/流量) | 中等(FID/trajectory) | 初建(KLD/FID) | **缺失** |

### 三个尚未解决的核心难题

**难题一：可扩展性与计算成本的矛盾**
- Emergent Crowds 50 agent，PEBA-PEvo 80 agent，Generative Agents 25 agent
- 每次 LLM 调用约 5 秒（Emergent Crowds），即使批处理也难以扩展到数百 agent
- GROVE 通过 preset + RAG 将 token 消耗降低 35-49%，但仍依赖大模型
- **可能的出路**：蒸馏 LLM 社会推理能力到小模型；分层架构中只在高层用 LLM

**难题二："涌现"vs"prompt 转录"的验证**
- Emergent Crowds 审稿人尖锐指出：缺少消融实验区分真正的涌现行为和 prompt 中编码的行为
- PEBA-PEvo 通过 expert benchmark 对齐部分解决了这个问题，但对齐不等于涌现
- **需要的实验**：中性 identical prompt（无关系、无个性差异）下群体是否仍形成？关闭对话后聚集是否消失？

**难题三：评估体系缺失**
- 没有统一的 benchmark 和 metrics 评估 LLM-driven crowd simulation 的质量
- Emergent Crowds 纯定性；PEBA-PEvo 用 KL 散度但依赖 expert benchmark 质量；Gen-C 用 graph statistics KLD；GROVE 用 VLM 评分
- 四项评估互不兼容，无法横向比较

### 五个值得追的研究方向

1. **多层级整合**：如何将 LLM 的高层语义决策与底层物理逼真运动（如 TRACE/PACE）无缝整合仍是开放问题。GROVE 的 BT + Theta* + SFM 是当前最佳实践，但仍有穿墙和语义漂移问题。

2. **蒸馏与压缩**：将 LLM 的社会推理能力蒸馏到小模型或规则系统中，提升效率。PEBA-PEvo 显示 DeepSeek-V3 成本效率是 GPT 的 3 倍，但绝对成本仍需 <$10/运行。

3. **评估基准建设**：需要一个统一的 benchmark，包含多样化的场景（日常/紧急/排队/社交）、标准化的 metrics（真实性/多样性/可控性/效率）、以及人类评估协议。Reality Check (SIGGRAPH 2026) 指出渲染方式会偏置主观判断，这使评估更加复杂。

4. **多模态融合**：结合视觉(VLM)、音频、文本的多模态人群模拟。EchoAvatar (SIGGRAPH 2026) 展示了音频驱动全身动作的 RVQ 方案，可与人群模拟结合。

5. **人机混合仿真**：真实人类与 LLM agent 在同一环境中交互。这在 VR/AR 训练场景中有直接应用价值。

---

## 阅读路径建议

**若你的目标是理解 LLM 如何驱动人群运动**：
Emergent Crowds (A1，首个完整框架) → PEBA-PEvo (A2，行为对齐理论) → Generative Agents (C1，基础架构)

**若你的目标是文本→人群行为生成**：
Text-Crowd (B1，SIGGRAPH 首个) → Gen-C (B2，图 VAE 路线) → GROVE (D1，机器人侧集成)

**若你的目标是构建可用的仿真系统**：
HuNavSim 2.0 (D2，基础平台) → GROVE (D1，文本接口) → SocialGAIL (D3，数据驱动 baseline)

---

## 关键参考文献列表

1. Liu Y, Shatzel L, Haworth B, Schneider T. **Emergent Crowds Dynamics from Language-Driven Multi-Agent Interactions**. arXiv:2508.15047, 2025.
2. Wang Y, Lucas GM, Becerik-Gerber B, Ustun V. **Implicit Behavioral Alignment of Language Agents in High-Stakes Crowd Simulations (PEBA-PEvo)**. EMNLP 2025. arXiv:2509.16457.
3. Ji X, Pan Z, Gao X, Pan J. **Text-Guided Synthesis of Crowd Animation (Text-Crowd)**. ACM SIGGRAPH 2024. DOI: 10.1145/3799902.3811062.
4. Panayiotou A, Charalambous P, Karamouzas I. **Gen-C: Populating Virtual Worlds with Generative Crowds**. arXiv:2504.01924, 2025.
5. Park JS, O'Brien J, Cai CJ, Morris MR, Liang P, Bernstein MS. **Generative Agents: Interactive Simulacra of Human Behavior**. UIST 2023.
6. Park JS. **Generative Agent Simulations of Human Behavior**. PhD Dissertation, Stanford University, 2025.
7. Nguyen DT, Shcherbyna V, Do DA, Shen Z, Buiyan T, Kästner L. **GROVE: Grounded Pedestrian Simulation via Natural Language for Interactive Social Robot Navigation**. arXiv:2606.25504, 2026.
8. Yang S, Zhang Y, Gu C. **Large Language Model–Enhanced Agent-Based Modeling for Intelligent Crowd Evacuation under Disaster Scenarios**. EGU 2026.
9. Rempe D, Luo Z, Peng XB, et al. **Trace and Pace: Controllable Pedestrian Animation via Guided Trajectory Diffusion**. CVPR 2023.
10. Ling B, Lyu Y, Li D, et al. **SocialGAIL: Faithful Crowd Simulation for Social Robot Navigation**. ICRA 2024.
11. Pérez-Higueras N, Otero R, Caballero F, Merino L. **HuNavSim: A ROS 2 Human Navigation Simulator for Benchmarking Human-Aware Robot Navigation**. IEEE RA-L 2023.

---

## 检索后记：几个值得单独提出的判断

### 这个领域比预想的更"年轻"

11 篇核心论文中，**9 篇发表于 2025 年或之后**，2 篇发表于 2023-2024 年。这意味着整个方向处于爆发初期，方法论尚未定型。特别是：
- Emergent Crowds (2025.08) 和 PEBA-PEvo (2025.09) 相差仅一个月，独立提出 LLM 驱动 crowd 的思路
- Gen-C (2025.04) 和 GROVE (2026.06) 分别代表了图形学和机器人社区的回应

### 必读的三个锚点

| 锚点 | 理由 |
|---|---|
| Emergent Crowds (A1) | 首个将 LLM 对话直接接入 steering 参数的完整框架，代码已开源 |
| PEBA-PEvo (A2) | 提出 Behavior-Realism Gap 概念 + 首个 principled 的可信度保障方法 |
| Gen-C (B2) | 最丰富的技术细节（双 VGAE 架构、canonical ordering、LLM vs learned generator 对比） |

### 两个可能改变你判断的数据点

**第一，LLM 直接生成人群场景的 scaling 曲线很差。** Gen-C Figure 5 显示：随着 agent 数从 20 增到 160，vanilla LLM 的 action 序列熵单调下降（多样性丧失）、drop rate 急剧上升。这说明"直接用 LLM 写 crowd script"这条路在规模上是死胡同——必须借助 learned generator（如 Gen-C 的双 VGAE）或 hierarchical 架构（如 GROVE 的 BT）。

**第二，PEBA-PEvo 的优化成本极低。** 完整 15 轮 persona 优化仅需 <$10（最贵模型），DeepSeek-V3 仅需约 $3。这意味着行为对齐不再是计算瓶颈——真正的瓶颈是 LLM 推理延迟（~5s/query）限制了 agent 规模和仿真时长。

### 最明显的研究缺口：评估体系

四条脉络各有自己的评估方式且互不兼容：
- Emergent Crowds：定性关键帧 + 对话日志（无数值）
- PEBA-PEvo：KL 散度 vs expert benchmark
- Gen-C：graph statistics KLD + FID/MMD
- GROVE：VLM-based 0-10 评分

没有一个统一 benchmark。这对于一个声称要产生"逼真人群行为"的领域来说是不可接受的。

### 三条最有希望的打通路径

1. **A × C 闭环**：将 PEBA-PEvo 的行为对齐框架应用到 Emergent Crowds 的 steering 参数控制上——用 expert benchmark 优化 agent persona，使其涌现的群体行为不仅"有趣"而且"真实"。
2. **B × D 集成**：将 Gen-C 的高层行为图作为 GROVE 的 BT 输入源——Gen-C 生成 what（行为序列），GROVE 负责 how（几何可行执行）。
3. **LLM + 扩散混合架构**：LLM 做高层语义决策（目标选择、社交判断），扩散模型做中层轨迹生成（平滑、物理可行），SFM 做底层避碰。这可能是兼顾真实性、可控性和效率的最佳路径。

---

*综述完成时间: 2026年8月11日*
*搜索范围: AI会议(NeurIPS/ICLR/AAAI/CHI/UIST/EMNLP), 图形学(SIGGRAPH/CVPR/Eurographics/CGF), 机器人(ICRA/IROS/RA-L/T-RO), 人群模拟专门会议*
*凡未查到的字段一律标注"未找到"或"信息缺口"，未做任何推断或编造*