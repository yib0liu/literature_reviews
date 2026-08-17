# SDEdit: Guided Image Synthesis and Editing with Stochastic Differential Equations
## SDEdit：基于随机微分方程的引导图像合成与编辑

**作者**: Chenlin Meng, Yutong He, Yang Song, Jiaming Song, Jiajun Wu, Jun-Yan Zhu, Stefano Ermon  
**单位**: Stanford University / Carnegie Mellon University  
**发表**: International Conference on Learning Representations (ICLR) 2022  
**arXiv**: 2108.01073  

---

## 摘要

引导图像合成使普通用户能够以最少的努力创建和编辑照片级真实感图像。关键挑战在于平衡对用户输入（如手绘彩色笔划）的**忠实性**（faithfulness）与合成图像的**真实感**（realism）。现有的基于GAN的方法试图通过条件GAN或GAN反演来实现这种平衡，但这些方法通常具有挑战性，且往往需要为每个具体应用额外收集训练数据或设计损失函数。

为了解决这些问题，我们引入了一种新的图像合成与编辑方法——**随机微分编辑**（Stochastic Differential Editing, SDEdit），该方法基于扩散模型生成先验，通过随机微分方程（SDE）的迭代去噪来合成逼真图像。给定一个带有用户引导（以操作RGB像素的形式）的输入图像，SDEdit首先向输入添加噪声，然后通过SDE先验对结果图像进行去噪以提高其真实感。SDEdit不需要任务特定的训练或反演过程，能够自然地实现真实感与忠实性之间的平衡。根据人类感知研究，在包括基于笔划的图像合成与编辑以及图像合成等多个任务上，SDEdit在真实感方面最高超越最先进的基于GAN的方法达 **98.09%**，在总体满意度评分上最高超越 **91.72%**。

---

## 第1节 引言

现代生成模型能够从随机噪声中创建照片级真实感的图像（Karras et al., 2019; Song et al., 2021），成为视觉内容创作的重要工具。其中特别令人感兴趣的是引导图像合成与编辑——用户指定一个大致的引导（如粗略的彩色笔划），生成模型学习填充细节（见图1）。引导图像合成有两个自然的期望目标：合成的图像应该看起来真实，同时忠实于用户引导的输入，从而使有或没有艺术专业知识的人都能够从不同细节层次产生照片级真实感的图像。

现有方法通常尝试通过两种途径来实现这种平衡。第一类方法利用条件GAN（Isola et al., 2017; Zhu et al., 2017），学习从原始图像到编辑后图像的直接映射。不幸的是，对于每个新的编辑任务，这些方法都需要数据收集和模型重新训练，两者都可能既昂贵又耗时。第二类方法利用GAN反演（Zhu et al., 2016; Brock et al., 2017; Abdal et al., 2019; Gu et al., 2020; Wu et al., 2021; Abdal et al., 2020），即使用预训练的GAN将输入图像反演到潜在表示空间，随后修改该表示以生成编辑后的图像。这一过程涉及为不同图像编辑任务手动设计损失函数和优化流程。此外，它有时可能无法找到一个忠实地表示输入的潜在编码（Bau et al., 2019b）。

为了在避免上述挑战的同时平衡**真实感**和**忠实性**，我们引入了SDEdit——一种利用生成式随机微分方程（Song et al., 2021，简称SDE）的引导图像合成与编辑框架。与密切相关的扩散模型（Sohl-Dickstein et al., 2015; Ho et al., 2020）类似，基于SDE的生成模型通过迭代去噪将初始高斯噪声向量平滑地转换为逼真的图像样本，并已在无条件图像合成方面取得了与GAN相当甚至更优的性能（Dhariwal & Nichol, 2021）。SDEdit的核心直觉是"劫持"基于SDE的生成模型的生成过程，如图2所示。给定一个带有用户引导输入（如笔画画或带有笔画编辑的图像）的输入图像，我们可以添加适量的噪声来消除不希望的伪影和失真（例如笔画像素处的不自然细节），同时仍然保留用户引导输入的整体结构。然后我们用这个带噪声的输入初始化SDE，并逐步去除噪声以获得一个既真实又忠实于用户引导输入的去噪结果（见图2）。

与条件GAN不同，SDEdit不需要为每个新任务收集训练图像或用户标注；与GAN反演不同，SDEdit不需要设计额外的训练或任务特定的损失函数。SDEdit仅使用一个在 unlabeled 数据上预训练的基于SDE的生成模型：给定一个以操作RGB像素形式呈现的用户引导，SDEdit向该引导添加高斯噪声，然后运行反向SDE来合成图像。SDEdit自然地找到了真实感和忠实性之间的权衡：当我们添加更多的高斯噪声并运行更长时间的SDE时，合成的图像更真实但忠实性更低。我们可以利用这一观察结果来找到真实感和忠实性之间的正确平衡点。

我们在三个应用上展示了SDEdit：基于笔划的图像合成、基于笔划的图像编辑和图像合成。我们表明SDEdit能够从各种保真度层次的引导中产生**真实的**和**忠实的**图像。在基于笔划的图像合成实验中，根据人类评判，SDEdit在真实感得分上最高超越最先进的基于GAN的方法达98.09%，在总体满意度得分（衡量真实性和忠实性）上最高超越91.72%。在图像合成实验中，SDEdit实现了更好的忠实性得分，并在用户研究中总体满意度得分最高超越基线83.73%。我们的代码和模型将在论文发表时公开。

---

## 第2节 背景：用随机微分方程进行图像合成

### 2.1 随机微分方程简介

随机微分方程（Stochastic Differential Equations, SDEs）通过在动力学中注入随机噪声来推广常微分方程（ODEs）。SDE的解是一个随时间变化的随机变量（即随机过程），我们将其记为 **x**(t)∈ℝ^d，其中 t∈[0,1] 索引时间。在图像合成中（Song et al., 2021），我们假设 **x**(0)∼p₀=p_data 表示来自数据分布的样本，前向SDE通过高斯扩散为 t∈(0,1] 生成 **x**(t)。给定 **x**(0)，**x**(t) 服从高斯分布：

$$\mathbf{x}(t) = \alpha(t)\mathbf{x}(0) + \sigma(t)\mathbf{z}, \quad \mathbf{z} \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$$

其中 σ(t):[0,1]→[0,∞) 是一个标量函数，描述噪声 **z** 的幅度，α(t):[0,1]→[0,1] 是一个标量函数，表示数据 **x**(0) 的幅度。**x**(t) 的概率密度函数记为 p_t。

通常考虑两种类型的SDE：**方差爆炸SDE**（Variance Exploding SDE, VE-SDE）对所有 t 满足 α(t)=1，且 σ(1) 为一个较大的常数，使得 p₁ 接近 𝒩(**0**,σ²(1)**I**)；而**方差保持SDE**（Variance Preserving SDE, VP-SDE）对所有 t 满足 α²(t)+σ²(t)=1，且当 t→1 时 α(t)→0，使得 p₁ 等于 𝒩(**0**,**I**)。VE和VP SDE都在 t 从0到1的过程中将数据分布转换为随机高斯噪声。为简洁起见，正文剩余部分我们基于VE-SDE讨论细节，并在附录C中讨论VP-SDE的过程。尽管两者形式略有不同且在不同图像域上表现各异，但它们共享相同的数学直觉。

### 2.2 用VE-SDE进行图像合成

在这些定义下，我们可以将图像合成问题表述为从含噪观测 **x**(t) 中逐渐去除噪声以恢复 **x**(0)。这可以通过反向SDE（Anderson, 1982; Song et al., 2021）来实现，该反向SDE从 t=1 行进到 t=0，基于对噪声扰动分数函数 ∇_**x** log p_t(**x**) 的了解。例如，VE-SDE的采样过程由以下（反向）SDE定义：

$$d\mathbf{x}(t) = \left[-\frac{d[\sigma^2(t)]}{dt} \nabla_\mathbf{x} \log p_t(\mathbf{x})\right] dt + \sqrt{\frac{d[\sigma^2(t)]}{dt}} d\bar{\mathbf{w}}$$

其中 **w̄** 是时间从 t=1 反向流到 t=0 时的维纳过程。如果我们设置初始条件 **x**(1)∼p₁=𝒩(**0**,σ²(1)**I**)，那么 **x**(0) 的解将服从 p_data 分布。在实践中，噪声扰动分数函数可以通过去噪分数匹配（denoising score matching, Vincent, 2011）来学习。将学习到的分数模型记为 **s**_θ(**x**(t),t)，时间 t 的学习目标为：

$$\mathcal{L}_t = \mathbb{E}_{\mathbf{x}(0)\sim p_{\text{data}}, \mathbf{z}\sim\mathcal{N}(\mathbf{0},\mathbf{I})} \left[\|\sigma_t \mathbf{s}_\theta(\mathbf{x}(t),t) - \mathbf{z}\|_2^2\right]$$

其中 p_data 是数据分布，**x**(t) 的定义同公式(1)。总体训练目标是各个学习时间目标 L_t 关于 t 的加权和，各种加权方式已在 Ho et al. (2020); Song et al. (2020); Song et al. (2021) 中讨论过。

有了参数化的分数模型 **s**_θ(**x**(t),t) 来近似 ∇_**x** log p_t(**x**)，SDE的解可以用Euler-Maruyama方法来近似；从 (t+Δt) 到 t 的更新规则为：

$$\mathbf{x}(t) = \mathbf{x}(t+\Delta t) + (\sigma^2(t) - \sigma^2(t+\Delta t)) \mathbf{s}_\theta(\mathbf{x}(t),t) + \sqrt{\sigma^2(t) - \sigma^2(t+\Delta t)} \, \mathbf{z}$$

其中 **z** ∼ 𝒩(**0**, **I**)。我们可以选择从1到0的时间区间的特定离散化，初始化 **x**(0) ∼ 𝒩(**0**,σ²(1)**I**) 并通过公式(4)迭代来生成图像 **x**(0)。

---

## 第3节 用SDEdit进行引导图像合成和编辑

### 3.1 问题设置

用户以操作RGB像素的形式提供一个全分辨率图像 **x**^(g)，我们称之为"引导"（guide）。引导可能包含不同层次的细节：高层次引导仅包含粗略的彩色笔划，中等层次引导包含真实图像上的彩色笔划，低层次引导包含目标图像上的图像块。我们在图1中展示了这些引导，它们可以由非专家轻松提供。我们的目标是生成具有以下两个期望特性的全分辨率图像：

**真实性（Realism）**：图像应看起来真实（例如，由人类或神经网络评估）。

**忠实性（Faithfulness）**：图像应与引导 **x**^(g) 相似（例如，通过 L₂ 距离衡量）。

我们注意到真实性和忠实性并非正相关，因为可能存在真实但不忠实的图像（例如一张随机的真实图像），以及忠实但不真实的图像（例如引导本身）。与常规逆问题不同，我们不假设知道测量函数（即从真实图像到用户在RGB像素中创建的引导的映射是未知的），因此用于解决基于分数的模型的逆问题的技术（Dhariwal & Nichol, 2021; Kawar et al., 2021）以及需要配对数据集的方法（Isola et al., 2017; Zhu et al., 2017）在此不适用。

### 3.2 SDEdit 方法

我们的方法SDEdit利用了以下事实：反向SDE不仅可以从 t₀=1 求解，也可以从任何中间时间 t₀∈(0,1) 求解——这是先前基于SDE的生成模型未曾研究过的方法。我们需要从我们的引导中找到一个合适的初始化，从中可以求解反向SDE以获得理想的、真实的且忠实的图像。对于任意给定的引导 **x**^(g)，我们将SDEdit过程定义如下：

> 采样 **x**^(g)(t₀) ∼ 𝒩(**x**^(g); σ²(t₀)**I**)，然后通过迭代公式(4)生成 **x**(0)。

我们用 SDEdit(**x**^(g); t₀, θ) 来表示上述过程。本质上，SDEdit选择一个特定的时间 t₀，向引导 **x**^(g) 添加标准差为 σ(t₀) 的高斯噪声，然后在 t=0 处求解相应的反向SDE以生成合成的 **x**(0)。

除了SDE求解器所取的离散化步数之外，SDEdit的关键超参数是 t₀——我们在反向SDE中开始图像合成过程的时间。在下文中，我们描述一个真实感-忠实性权衡，它允许我们选择合理的 t₀ 值。

### 3.3 真实感-忠实性权衡

我们注意到，对于训练良好的SDE模型，在选择不同的 t₀ 值时存在真实感-忠实性权衡。为了说明这一点，我们聚焦于LSUN数据集，并使用高层次的笔画画作为引导来执行基于笔划的图像生成。实验细节见附录D.2。我们对同一输入考虑 t₀∈[0,1] 的不同选择。为了量化真实感，我们采用用于比较图像分布的神经方法，如核Inception分数（Bińkowski et al., 2018，简称KID）。如果合成图像与真实图像之间的KID较低，则合成图像是真实的。对于忠实性，我们测量合成图像与引导 **x**^(g) 之间的平方 L₂ 距离。从图3中我们观察到，随着 t₀ 的增加，真实感提高但忠实性下降。

真实感-忠实性权衡可以从另一个角度来解释。如果引导远离任何真实图像，那么我们必须容忍至少一定程度的偏离引导（不忠实），才能生成真实的图像。这在以下命题中得到了说明。

**命题1**：假设对所有 **x**∈𝒳 和 t∈[0,1] 有 ||**s**_θ(**x**,t)||₂² ≤ C。那么对所有 δ∈(0,1)，至少以概率 (1−δ)：

$$\|\mathbf{x}^{(g)} - \text{SDEdit}(\mathbf{x}^{(g)}; t_0, \theta)\|_2^2 \leq \sigma^2(t_0)\left(C\sigma^2(t_0) + d + 2\sqrt{-d\cdot\log\delta} - 2\log\delta\right)$$

其中 d 是 **x**^(g) 的维度。

我们在附录A中提供了证明。从高层面上看，引导与合成图像之间的差异可以分解为分数输出和随机高斯噪声两部分；两者都会随着 t₀ 的增加而增大，因此差异也随之变大。上述命题表明，为了使图像以高概率变得真实，我们必须有一个足够大的 t₀。反过来，如果 t₀ 太大，则对引导的忠实性会恶化，SDEdit将产生随机的真实图像（极端情况就是无条件图像合成）。

### 3.4 选择 t₀

我们注意到引导的质量可能会影响合成图像的整体质量。对于合理的引导，我们发现 t₀∈[0.3, 0.6] 效果良好。然而，如果引导是一张只有白色像素的图像，那么即使从模型分布中采样的最接近的"真实"样本也可能相距甚远，我们必须通过选择较大的 t₀ 来牺牲忠实性以获得更好的真实感。在交互式设置中（用户绘制基于草图的引导），我们可以初始化 t₀∈[0.3, 0.6]，用SDEdit合成候选图像，然后询问用户该样本是否应该更忠实或更真实；根据反馈，我们可以通过二分搜索获得合理的 t₀。在大规模非交互式设置中（我们得到一组已生成的引导），我们可以在随机选择的图像上执行类似的二分搜索以获得 t₀，随后在同一任务中对所有引导固定该 t₀。虽然不同的引导可能有不同的最优 t₀，但我们经验上观察到共享的 t₀ 在同一任务中对所有合理的引导都效果良好。

### 3.5 详细算法和扩展

我们在算法1中给出了VE-SDE的通用算法。由于篇幅限制，我们在附录C中描述了VP-SDE的详细算法。本质上，该算法是用于求解 SDEdit(**x**^(g); t₀, θ) 的Euler-Maruyama方法。对于我们希望保持合成图像的某些部分与引导完全相同的情况，我们还可以引入一个额外的通道来掩蔽掉我们不想编辑的图像部分。这是对正文中提到的SDEdit过程的轻微修改，我们在附录C.2中讨论了细节。

**算法1：用SDEdit进行引导图像合成与编辑（VE-SDE）**

| | |
|---|---|
| **输入**: | **x**^(g)（引导）, t₀（SDE超参数）, N（总去噪步数） |
| Δt ← t₀/N | |
| **z** ∼ 𝒩(**0**, **I**) | |
| **x** ← **x** + σ(t₀)**z** | |
| **for** n ← N **to** 1 **do** | |
| &nbsp;&nbsp;t ← t₀·n/N | |
| &nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;ϵ ← √(σ²(t) − σ²(t − Δt)) | |
| &nbsp;&nbsp;**x** ← **x** + ϵ² **s**_θ(**x**, t) + ϵ**z** | |
| **end for** | |
| **返回** **x** | |

---

## 第4节 相关工作

### 4.1 条件GANs

用于图像编辑的条件GAN（Isola et al., 2017; Zhu et al., 2017; Jo & Park, 2019; Liu et al., 2021）学习直接基于用户输入生成图像，并已在多种任务上取得成功，包括图像合成与编辑（Portenier et al., 2018; Chen & Koltun, 2017; Dekel et al., 2018; Wang et al., 2018; Park et al., 2019; Zhu et al., 2020b; Jo & Park, 2019; Liu et al., 2021）、图像修复（inpainting, Pathak et al., 2016; Iizuka et al., 2017; Yang et al., 2017; Liu et al., 2018）、照片着色（photo colorization, Zhang et al., 2016; Larsson et al., 2016; Zhang et al., 2017; He et al., 2018）、语义图像纹理与几何合成（Zhou et al., 2018; Guérin et al., 2017; Xian et al., 2018）。它们在使用用户草图或颜色进行图像编辑方面也取得了强劲的性能（Jo & Park, 2019; Liu et al., 2021; Sangkloy et al., 2017）。然而，条件模型必须在原始图像和编辑后图像上进行训练，因此对于新的编辑任务需要数据收集和模型重新训练。因此，将此类方法应用于即时图像操作仍然具有挑战性，因为每个新应用都需要训练一个新模型。与条件GAN不同，SDEdit只需要在原始图像上进行训练。因此，它可以像图1所示那样在测试时直接应用于各种编辑任务。

### 4.2 GAN反演和编辑

图像编辑的另一种主流方法是GAN反演（Zhu et al., 2016; Brock et al., 2017），其中输入首先被投影到无条件GAN的潜在空间中，然后从修改后的潜在编码合成新图像。已有多种方法在这一方向上被提出，包括为每张图像微调网络权重（Bau et al., 2019a; Pan et al., 2020; Roich et al., 2021）、选择更好或多层进行投影和编辑（Abdal et al., 2019; Abdal et al., 2020; Gu et al., 2020; Wu et al., 2021）、设计更好的编码器（Richardson et al., 2021; Tov et al., 2021）、建模图像损坏和变换（Anirudh et al., 2020; Huh et al., 2020），以及发现有意义的潜在方向（Shen et al., 2020; Goetschalckx et al., 2019; Jahanian et al., 2020; Härkönen et al., 2020）。然而，这些方法需要为不同任务定义不同的损失函数。它们还需要进行GAN反演，这在各种数据集上可能效率低下且不准确（Huh et al., 2020; Karras et al., 2020b; Bau et al., 2019b; Xu et al., 2021）。

### 4.3 其他生成模型

训练非归一化概率模型的最新进展，如基于分数的生成模型（score-based generative models, Song & Ermon, 2019; Song & Ermon, 2020; Song et al., 2021; Ho et al., 2020; Song et al., 2020; Jolicoeur-Martineau et al., 2021）和基于能量的模型（energy-based models, Ackley et al., 1985; Gao et al., 2017; Du & Mordatch, 2019; Xie et al., 2018; Xie et al., 2016; Song & Kingma, 2021），已取得与GAN相当的图像样本质量。然而，该方向的大多数先前工作集中在无条件图像生成和密度估计上，图像编辑和合成的最先进技术仍由基于GAN的方法主导。在本工作中，我们专注于最近兴起的基于随机微分方程（SDE）的生成建模，并研究其在可控图像编辑和合成任务中的应用。一项并行工作（Choi et al., 2021）使用扩散模型进行条件图像合成，其中条件可以表示为底层真实图像的已知函数。

---

## 第5节 实验

### 5.1 评估指标

我们基于**真实感**和**忠实性**来评估编辑结果。为了量化**真实感**，我们使用生成图像与目标真实图像数据集之间的核Inception分数（KID，详见附录D.2），以及通过Amazon Mechanical Turk（MTurk）对不同方法进行成对人类评估。为了量化**忠实性**，我们报告引导与编辑后输出图像之间在所有像素上求和的 L₂ 距离（归一化到 [0,1]）。对于某些实验，我们还考虑了LPIPS（Zhang et al., 2018）和MTurk人类评估。为了量化总体人类满意度得分（**真实感** + **忠实性**），我们利用MTurk人类评估在基线和SDEdit之间进行成对比较（详见附录F）。

### 5.2 基于笔划的图像合成

给定一个输入笔画画，我们的目标是**在没有配对数据可用时**生成**真实的**和**忠实的**图像。我们考虑由人类用户创建的笔画画引导（见图5）。同时，我们还提出了一种算法来自动模拟基于源图像的用户笔画画（见图4），使我们能够对SDEdit进行大规模定量评估。更多细节见附录D.2。

**基线方法：**

为了比较，我们选择了三种最先进的基于GAN的图像编辑和合成方法作为基线。第一个基线是StyleGAN2-ADA（Karras et al., 2020a）中使用的图像投影方法，其中反演通过在W+空间中最小化感知损失来完成。第二个基线是域内GAN（in-domain GAN, Zhu et al., 2020a），其中反演通过在编码器之上运行优化步骤来完成。具体而言，我们考虑两种版本的域内GAN反演技术：第一种（记为 In-domain GAN-1）仅使用编码器以最大化反演速度，而第二种（记为 In-domain GAN-2）运行额外的优化步骤以最大化反演精度。第三个基线是 e4e（Tov et al., 2021），其编码器目标被明确设计为通过鼓励将图像反演到预训练StyleGAN模型的W空间附近来平衡感知质量和可编辑性。

**结果：**

我们在图4中展示了定性比较结果。我们观察到所有基线都难以基于笔画画输入生成真实图像，而SDEdit成功地生成了保留输入笔画画笔义的逼真图像。如图5所示，SDEdit还可以为同一输入合成多样化的图像。我们在表1中展示了使用用户创建笔画引导的定量比较结果，在表2中展示了使用算法模拟笔画引导的结果。我们报告 L₂ 距离用于忠实性比较，利用MTurk（见附录F）或KID分数用于真实感比较。为了量化总体人类满意度得分（忠实性 + 真实感），我们让另一组MTurk工作人员基于**忠实性和真实性**对基线和SDEdit进行另外3000次成对比较。我们观察到SDEdit在所有评估指标上都优于GAN基线，在真实感得分上超越基线超过80%，在总体满意度得分上超越基线超过75%。更多实验细节见附录C，更多结果见附录E。

**表1：SDEdit在LSUN (bedroom) 上基于笔划的生成性能超越所有GAN基线。输入笔划由人类用户创建。最右两列代表MTurk工作人员在成对比较中更倾向SDEdit相对于基线的百分比。**

| 基线 | 忠实性得分 (L₂) ↓ | SDEdit更真实 (MTurk) ↑ | SDEdit更令人满意 (MTurk) ↑ |
|------|-------------------|----------------------|--------------------------|
| In-domain GAN-1 | 101.18 | 94.96% | 89.48% |
| In-domain GAN-2 | 57.11 | 97.87% | 89.51% |
| StyleGAN2-ADA | 68.12 | 98.09% | 91.72% |
| e4e | 53.76 | 80.34% | 75.43% |
| **SDEdit** | **32.55** | – | – |

**表2：SDEdit在忠实性和真实性上都超越所有GAN基线用于基于笔划的图像生成。输入笔划由笔划模拟算法生成。使用生成的图像和对应验证集计算KID。**

| 方法 | LSUN Bedroom | | LSUN Church | |
|------|-------------|------|------------|------|
| | L₂ ↓ | KID ↓ | L₂ ↓ | KID ↓ |
| In-domain GAN-1 | 105.23 | 0.1147 | – | – |
| In-domain GAN-2 | 76.11 | 0.2070 | – | – |
| StyleGAN2-ADA | 74.03 | 0.1750 | 72.41 | 0.1544 |
| e4e | 52.40 | 0.0464 | 68.53 | 0.0354 |
| **SDEdit (ours)** | **36.76** | **0.0030** | **37.67** | **0.0156** |

### 5.3 灵活的图像编辑

在本节中，我们展示SDEdit能够在图像编辑任务上超越现有的基于GAN的模型。我们聚焦于LSUN（bedroom, church）和CelebA-HQ数据集，实验设置的更多细节见附录D。

**基于笔划的图像编辑：**

给定一个带有笔划编辑的图像，我们希望基于用户的编辑生成一个真实且忠实的图像。我们考虑与前一个实验相同的基于GAN的基线（Zhu et al., 2020a; Karras et al., 2020a; Tov et al., 2021）。如图6所示，基线生成的结果往往会引入不希望的修改，偶尔使笔划外的区域变得模糊。相比之下，SDEdit能够生成**既真实又忠实**于输入的图像编辑，同时避免做出不希望的修改。额外结果见附录E。

**图像合成：**

我们聚焦于CelebA-HQ数据集（Karras et al., 2017）上的图像合成。给定从数据集中随机采样的图像，我们要求用户使用从其他参考图像复制的像素块以及他们想要进行修改的像素来指定他们希望编辑后的图像看起来如何（见图7）。我们将我们的方法与传统的混合算法（Burt & Adelson, 1987; Pérez et al., 2003）以及之前考虑的相同GAN基线进行比较。我们在图7中进行定性比较。对于定量比较，我们报告 L₂ 距离来量化忠实性。为了量化真实感，我们让MTurk工作人员在基线和SDEdit之间进行1500次成对比较。为了量化用户满意度得分（忠实性 + 真实感），我们让不同的工作人员再进行1500次针对SDEdit的成对比较。为了量化不希望的改变（例如身份变化），我们遵循 Bau et al. (2020) 来计算掩码LPIPS（Zhang et al., 2018）。如表3所示，我们观察到SDEdit能够生成既忠实又真实的图像，且具有比基线好得多的LPIPS分数，在总体满意度得分上最高超越基线83.73%，在真实感上最高超越75.60%。虽然我们的真实感得分略低于e4e，但SDEdit生成的图像总体上更忠实、更令人满意。更多实验细节见附录D。

**表3：CelebA-HQ上的图像合成实验。中间两列表示MTurk工作人员更倾向SDEdit的百分比。我们还报告编辑和未编辑图像之间的掩码LPIPS距离以量化掩码外的不需要改变。我们观察到SDEdit能够实现真实的编辑同时比基线更忠实，在总体满意度得分上击败基线高达83.73%。**

| 方法 | L₂ ↓ | SDEdit更真实 (MTurk) ↑ | SDEdit更令人满意 (MTurk) ↑ | LPIPS (掩码) ↓ |
|------|-------|----------------------|--------------------------|-----------------|
| Laplacian Blending | 68.45 | 75.27% | 83.73% | 0.09 |
| Poisson Blending | 63.04 | 75.60% | 82.18% | 0.05 |
| In-domain GAN | 36.67 | 53.08% | 73.53% | 0.23 |
| StyleGAN2-ADA | 69.38 | 74.12% | 83.43% | 0.21 |
| e4e | 53.90 | 43.67% | 66.00% | 0.33 |
| **SDEdit (ours)** | **21.70** | – | – | **0.03** |

---

## 第6节 结论

我们提出了随机微分编辑（SDEdit）——一种通过随机微分方程（SDE）对图像进行生成建模的引导图像编辑与合成方法，能够平衡真实感和忠实性。与通过GAN反演进行的图像编辑技术不同，我们的方法不需要针对重建输入的特定任务优化算法，特别适用于GAN反演损失难以设计或优化的数据集或任务。与条件GAN不同，我们的方法不需要为"引导"图像收集新的数据集或重新训练模型，这两者都可能既昂贵又耗时。我们证明了SDEdit在基于笔划的图像合成、基于笔划的图像编辑和图像合成方面无需任务特定训练即可超越现有的基于GAN的方法。

### 致谢

作者感谢 Kristy Choi 帮助校对。本研究得到了NSF (#1651565, #1522054, #1733686)、ONR (N00014-19-1-2145)、AFOSR (FA9550-19-1-0024)、ARO、Autodesk、Stanford HAI、Amazon ARA 和 Amazon AWS 的支持。Yang Song 获得了 Apple AI/ML 博士奖学金的支持。J.-Y. Zhu 部分得到了 Naver Corporation 的支持。

### 伦理声明

在本工作中，我们提出了SDEdit——一种基于生成式随机微分方程（SDE）的新图像合成与编辑方法。在我们的实验中，所有考虑的数据集都是开源且公开可用的，并在许可下使用。与常见的基于深度学习的图像合成和编辑算法类似，我们的方法的社会影响既有积极的也有消极的，取决于具体的应用和使用方式。从积极的一面来看，SDEdit使有或没有艺术专业知识的普通用户能够以最少的努力创建和编辑照片级真实感的图像，降低了视觉内容创作的门槛。另一方面，SDEdit可以被用来生成高质量的编辑图像，这些图像很难被人类与真实图像区分开来，可能被恶意用于欺骗人类和传播虚假信息。与常见的深度学习模型（如用于人脸编辑的基于GAN的方法）类似，SDEdit也可能被恶意用户利用并产生潜在的负面影响。在我们的代码发布中，我们将通过适当的许可证明确指定我们系统的允许用途。

我们还注意到，用于检测伪造机器生成图像的取证方法主要集中在区分基于GAN方法生成的样本。由于GAN和生成式SDE之间底层性质的差异，我们观察到用于检测GAN生成的伪造图像的最先进方法（Wang et al., 2020）难以区分基于SDE模型生成的伪造样本。例如，在LSUN bedroom数据集上，它仅成功检测到不到3%的SDEdit生成的图像，而在基于GAN的生成上能够区分高达93%。基于这些观察，我们认为随着基于SDE的方法变得越来越普遍，开发针对基于SDE模型的取证方法也至关重要。

对于人类评估实验，我们利用了Amazon Mechanical Turk（MTurk）。对于每位工作人员，评估HIT包含15个用于比较编辑图像的成对比较问题。每个任务的报酬保持在0.2美元。由于每个任务大约需要1分钟，因此时薪约为12美元。我们在附录F中提供了人类评估实验的更多细节。我们还注意到，人类评估者（MTurk工作人员）的偏差和用户（通过输入"引导"）的偏差可能会潜在地影响用于跟踪引导图像合成和编辑进展的评估指标和结果。

### 可重复性声明

- 代码将在发表时发布
- 我们使用开源数据集和对应数据集上的SDE检查点。我们没有训练任何SDE模型
- 证明在附录A中提供
- 有关SDEdit的额外详情和伪代码在附录C中提供
- 实验设置的详情在附录D中提供
- 额外实验结果在附录E中提供
- 人工评估的详情在附录F中提供

---

## 参考文献

1. **Abdal, R., Qin, Y., & Wonka, P.** (2019). Image2stylegan: How to embed images into the stylegan latent space? *CVPR*, 4432–4441.

2. **Abdal, R., Qin, Y., & Wonka, P.** (2020). Image2stylegan++: How to edit the embedded images? *CVPR*, 8296–8305.

3. **Ackley, D. H., Hinton, G. E., & Sejnowski, T. H.** (1985). A learning algorithm for boltzmann machines. *Cognitive Science*, 9(1):147–169.

4. **Anderson, B. D. O.** (1982). Reverse-time diffusion equation models. *Stochastic Processes and their Applications*, 12(3):313–326.

5. **Anirudh, R., Thiagarajan, J. J., Kailkhura, B., & Bremer, T.** (2020). Mimicgan: Robust projection onto image manifolds with corruption mimicking. *IJCV*, 1–19.

6. **Bau, D., Strobelt, H., Peebles, W., Wulff, J., Zhou, B., Zhu, J.-Y., & Torralba, A.** (2019a). Semantic photo manipulation with a generative image prior. *ACM SIGGRAPH*, 38(4):1–11.

7. **Bau, D., Zhu, J.-Y., Wulff, J., Peebles, W., Strobelt, H., Zhou, B., & Torralba, A.** (2019b). Seeing what a gan cannot generate. *ICCV*, 4502–4511.

8. **Bau, D., Liu, S., Wang, T., Zhu, J.-Y., & Torralba, A.** (2020). Rewriting a deep generative model. *ECCV*.

9. **Bińkowski, M., Sutherland, D. J., Arbel, M., & Gretton, A.** (2018). Demystifying mmd gans. *arXiv:1801.01401*.

10. **Brock, A., Lim, T., Ritchie, J. M., & Weston, J. M.** (2017). Neural photo editing with introspective adversarial networks. *ICLR*.

11. **Burt, P. J., & Adelson, E. H.** (1987). The laplacian pyramid as a compact image code. *Readings in Computer Vision*, 671–679.

12. **Chen, Q., & Koltun, V.** (2017). Photographic image synthesis with cascaded refinement networks. *ICCV*.

13. **Choi, J., Kim, S., Jeong, S., Gwon, Y., & Yoon, S.** (2021). ILVR: Conditioning method for denoising diffusion probabilistic models. *arXiv:2108.02938*.

14. **Dekel, T., Gan, C., Dilip, K., Liu, C., & Freeman, W. T.** (2018). Sparse, smart contours to represent and edit images. *CVPR*.

15. **Dhariwal, P., & Nichol, A.** (2021). Diffusion models beat gans on image synthesis. *arXiv:2105.05233*.

16. **Du, Y., & Mordatch, I.** (2019). Implicit generation and generalization in energy-based models. *arXiv:1903.08689*.

17. **Gao, R., Lu, Y., Zhou, J., Zhu, S.-C., & Wu, Y. N.** (2017). Learning energy-based models as generative convnets via multi-grid modeling and sampling. *arXiv:1709*.

18. **Goetschalckx, L., Andonian, A., Oliva, A., & Isola, P.** (2019). Ganalyze: Toward visual definitions of cognitive image properties. *ICCV*.

19. **Gu, J., Shen, Y., & Zhou, B.** (2020). Image processing using multi-code gan prior. *CVPR*.

20. **Guérin, É., Digne, J., Galin, É., Peytavie, A., Wolf, C., Benes, B., & Martinez, B.** (2017). Interactive example-based terrain authoring with conditional generative adversarial networks. *ACM TOG*, 36(6).

21. **Härkönen, E., Hertzmann, A., Lehtinen, J., & Paris, S.** (2020). Ganspace: Discovering interpretable gan controls. *NeurIPS*.

22. **He, M., Dong, C., Liao, J., Sander, P. V., & Yuan, L.** (2018). Deep exemplar-based colorization. *ACM TOG*, 37(4):1–16.

23. **Ho, J., Jain, A., & Abbeel, P.** (2020). Denoising diffusion probabilistic models. *arXiv:2006.11239*.

24. **Huh, M., Zhang, R., Zhu, J.-Y., Paris, S., & Hertzmann, A.** (2020). Transforming and projecting images into class-conditional generative networks. *ECCV*.

25. **Iizuka, S., Simo-Serra, E., & Ishikawa, H.** (2017). Globally and locally consistent image completion. *ACM TOG*, 36(4):107.

26. **Isola, P., Zhu, J.-Y., Zhou, T., & Efros, A. A.** (2017). Image-to-image translation with conditional adversarial networks. *CVPR*.

27. **Jahanian, A., Chai, L., & Isola, P.** (2020). On the "steerability" of generative adversarial networks. *ICLR*.

28. **Jo, Y., & Park, T.-H.** (2019). Sc-fegan: Face editing generative adversarial network with user's sketch and color. *ICCV*, 1745–1753.

29. **Jolicoeur-Martineau, A., Piche-Taillefer, R., Mitliagkas, I., & Combes, R.** (2021). Adversarial score matching and improved sampling for image generation. *ICLR*.

30. **Karras, T., Aila, T., Laine, S., & Lehtinen, S.** (2017). Progressive growing of gans for improved quality, stability, and variation. *arXiv:1710.10196*.

31. **Karras, T., Laine, S., & Aila, T.** (2019). A style-based generator architecture for generative adversarial networks. *CVPR*.

32. **Karras, T., Aittala, M., Hellsten, J., Laine, S., Lehtinen, J., & Aila, T.** (2020a). Training generative adversarial networks with limited data. *arXiv:2006.06676*.

33. **Karras, T., Laine, S., Aittala, M., Hellsten, J., Lehtinen, J., & Aila, T.** (2020b). Analyzing and improving the image quality of stylegan. *CVPR*.

34. **Kawar, B., Vaksman, G., & Elad, M.** (2021). Snips: Solving noisy inverse problems stochastically. *arXiv:2105.14951*.

35. **Larsson, G., Maire, M., & Shakhnarovich, G.** (2016). Learning representations for automatic colorization. *ECCV*.

36. **Laurent, B., & Massart, P.** (2000). Adaptive estimation of a quadratic functional by model selection. *Annals of Statistics*, 1302–1338.

37. **Liu, G., Reda, F. A., Shih, K. J., Wang, T.-C., Tao, A., & Catanzaro, B.** (2018). Image inpainting for irregular holes using partial convolutions. *ECCV*.

38. **Liu, H., Wan, Z., Huang, W., Song, Y., Han, I., Liao, J., Jiang, B., & Liu, W.** (2021). Deflocnet: Deep image editing via flexible low-level controls. *CVPR*, 10765–10774.

39. **Pan, X., Zhan, X., Dai, B., Lin, D., Loy, C. C., & Luo, P.** (2020). Exploiting deep generative prior for versatile image restoration and manipulation. *ECCV*.

40. **Park, T., Liu, M.-Y., Wang, T.-C., & Zhu, J.-Y.** (2019). Semantic image synthesis with spatially-adaptive normalization. *CVPR*.

41. **Pathak, D., Krahenbuhl, P., Darrell, T., & Darrell, T.** (2016). Context encoders: Feature learning by inpainting. *CVPR*.

42. **Pérez, P., Gangnet, M., & Blake, A.** (2003). Poisson image editing. *ACM SIGGRAPH*, 313–318.

43. **Portenier, T., Hu, Q., Szabó, A., Bigdeli, S. A., Favaro, P., & Zwicker, M.** (2018). Faceshop: Deep sketch-based face image editing. *ACM TOG*, 37(4).

44. **Richardson, E., Alaluf, Y., Patashnik, O., Nitzan, Y., Azar, Y., Shapiro, S., & Cohen-Or, D.** (2021). Encoding in style: a stylegan encoder for image-to-image translation. *CVPR*.

45. **Roich, D., Mokady, R., Bermano, A. H., & Cohen-Or, D.** (2021). Pivotal tuning for latent-based editing of real images. *arXiv:2106.05744*.

46. **Sangkloy, P., Lu, J., Fang, C., Yu, F., & Hays, J.** (2017). Scribbler: Controlling deep image synthesis with sketch and color. *CVPR*.

47. **Shen, Y., Gu, J., Tang, X., & Zhou, B.** (2020). Interpreting the latent space of gans for semantic face editing. *CVPR*.

48. **Sohl-Dickstein, J., Weiss, E. A., Maheswaranathan, N., & Ganguli, S.** (2015). Nonequilibrium thermodynamics and deep unsupervised learning. *arXiv:1503.03585*.

49. **Song, J., Chen, L., & Ermon, S.** (2020). Denoising diffusion implicit models. *arXiv:2010.02502*.

50. **Song, Y., & Ermon, S.** (2019). Generative modeling by estimating gradients of the data distribution. *NeurIPS*.

51. **Song, Y., & Ermon, S.** (2020). Improved techniques for training score-based generative models. *arXiv:2006.09011*.

52. **Song, Y., & Kingma, D. P.** (2021). How to train your energy-based models. *arXiv:2101.03288*.

53. **Song, Y., Sohl-Dickstein, J., Kingma, D. P., Kumar, A., Ermon, S., & Poole, B.** (2021). Score-based generative modeling through stochastic differential equations. *ICLR*.

54. **Tov, O., Alaluf, Y., Nitzan, Y., Patashnik, O., & Cohen-Or, D.** (2021). Designing an encoder for stylegan image manipulation. *ACM TOG*, 40(4):1–14.

55. **Vincent, P.** (2011). A connection between score matching and denoising autoencoders. *Neural Computation*, 23(7):1661–1674.

56. **Wang, S.-Y., Wang, O., Zhang, R., Owens, A., & Efros, A. A.** (2020). CNN-generated images are surprisingly easy to spot…for now. *CVPR*.

57. **Wang, T.-C., Liu, M.-Y., Zhu, J.-Y., Tao, A., Kautz, J., & Catanzaro, B.** (2018). High-resolution image synthesis and semantic manipulation with conditional gans. *CVPR*.

58. **Wu, Z., Lischinski, D., & Shechtman, E.** (2021). Stylespace analysis: Disentangled controls for stylegan image generation. *CVPR*.

59. **Xian, W., Sangkloy, P., Agrawal, V., Raj, A., Lu, C., Fang, C., Yu, F., & Hays, J.** (2018). Texturegan: Controlling deep image synthesis with texture patches. *CVPR*.

60. **Xie, J., Lu, Y., Zhu, S.-C., & Wu, Y. N.** (2016). A theory of generative convnet. *ICML*, 2635–2644.

61. **Xie, J., Lu, Y., Gao, J., Zhu, S.-C., & Wu, Y. N.** (2018). Cooperative learning of energy-based model and latent variable model via mcmc teaching. *AAAI*, 32.

62. **Xu, Y., Shen, Y., Zhu, J., Yang, C., & Zhou, B.** (2021). Generative hierarchical features from synthesizing images. *CVPR*, 4432–4442.

63. **Yang, C., Lu, X., Lin, Z., Shechtman, E., & Wang, O.** (2017). High-resolution image inpainting using multi-scale neural patch synthesis. *CVPR*.

64. **Zhang, H., Dana, K., Shi, P., Zhang, X., Wang, X., Tyagi, A., & Agrawal, A.** (2018). Context encoding for semantic segmentation. *CVPR*, 7151–7160.

65. **Zhang, R., Isola, P., & Efros, A. A.** (2016). Colorful image colorization. *ECCV*.

66. **Zhang, R., Isola, P., Efros, A. A., Geng, Y., Lin, S., Yu, Y., & Efros, F.** (2017). Real-time user-guided image colorization with learned deep priors. *ACM TOG*, 9(4).

67. **Zhou, Y., Zhu, Z., Bai, X., Lischinski, D., Cohen-Or, D., & Huang, H.** (2018). Non-stationary texture synthesis by adversarial expansion. *ACM TOG*, 37(4).

68. **Zhu, J., Shen, Y., Zhao, D., & Zhou, B.** (2020a). In-domain gan inversion for real image editing. *ECCV*.

69. **Zhu, J.-Y., Krähenbühl, P., Shechtman, E., & Efros, A. A.** (2016). Generative visual manipulation on the natural image manifold. *ECCV*.

70. **Zhu, J.-Y., Park, T., Isola, P., & Efros, A. A.** (2017). Unpaired image-to-image translation using cycle-consistent adversarial networks. *ICCV*.

71. **Zhu, P., Abdal, R., Qin, Y., & Wonka, P.** (2020b). Sean: Image synthesis with semantic region-adaptive normalization. *CVPR*.

---

## 附录A 证明

**证明（命题1）：**

记 **x**^(g)(0) = SDEdit(**x**^(g); t, θ)，则

$$\|\mathbf{x}^{(g)}(t_0) - \mathbf{x}^{(g)}(0)\|_2^2 = \left\|\int_{t_0}^{0} \frac{d\mathbf{x}^{(g)}(t)}{dt} dt\right\|_2^2$$

$$= \left\|\int_{t_0}^{0} \left[-\frac{d[\sigma^2(t)]}{dt} \mathbf{s}_\theta(\mathbf{x}, t; \theta)\right] dt + \sqrt{\frac{d[\sigma^2(t)]}{dt}} d\bar{\mathbf{w}}\right\|_2^2$$

$$\leq \left\|\int_{t_0}^{0} \left[-\frac{d[\sigma^2(t)]}{dt} \mathbf{s}_\theta(\mathbf{x}, t; \theta)\right] dt\right\|_2^2 + \left\|\int_{t_0}^{0} \sqrt{\frac{d[\sigma^2(t)]}{dt}} d\bar{\mathbf{w}}\right\|_2^2$$

根据对 **s**_θ(**x**, t; θ) 的假设，第一项不超过

$$C\left\|\int_{t_0}^{0} \left[-\frac{d[\sigma^2(t)]}{dt}\right] dt\right\|_2^2 = C\sigma^4(t_0)$$

其中等号仅在每个分数输出的平方 L₂ 范数为 C 且它们彼此线性相关时才成立。第二项与第一项独立，因为它仅涉及随机噪声；这等于维纳过程在时间 t=0 处的随机变量的平方 L₂ 范数，边缘分布为 ϵ ∼ 𝒩(**0**, σ²(t₀)**I**)（该边缘分布不依赖于 Euler-Maruyama 中的离散化步数）。ϵ 的平方 L₂ 范数除以 σ²(t₀) 服从自由度为 d 的 χ² 分布。根据 Laurent & Massart (2000) 的引理1，我们有以下单侧尾部界：

$$\Pr\left(\frac{\|\epsilon\|_2^2}{\sigma^2(t_0)} \geq d + 2\sqrt{d\cdot(-\log\delta)} - 2\log\delta\right) \leq \exp(\log\delta) = \delta$$

因此，至少以概率 (1−δ)，我们有：

$$\|\mathbf{x}^{(g)}(t_0) - \mathbf{x}^{(g)}(0)\|_2^2 \leq \sigma^2(t_0)\left(C\sigma^2(t_0) + d + 2\sqrt{-d\cdot\log\delta} - 2\log\delta\right)$$

证毕。∎

---

## 附录B 额外的消融研究

### B.1 用户引导质量分析

如第3节所讨论的，如果引导远离任何真实图像（例如随机噪声或具有不合理的构图），那么我们必须容忍至少一定程度的偏离引导（不忠实），才能生成真实的图像。

对于实际应用，我们在图8、图9和表4中对引导笔划的质量如何影响结果进行了额外的消融研究。具体而言，在图8中我们考虑了以下笔划输入：1) CelebA-HQ模型的人脸但细节有限，2) CelebA-HQ模型的带尖刺的人脸，3) LSUN-church模型的细节有限的建筑物，4) LSUN-church模型的一匹马。我们观察到SDEdit总体上对不同种类的用户输入具有容忍性。在表4中，我们使用模拟笔画画作为输入定量分析了用户引导质量的影响。如附录D.2所述，人类笔划模拟算法使用不同数量的颜色来生成具有不同细节层次的笔划引导。我们在图9中定性地比较了SDEdit与基线，在表4中定量比较。同样，我们观察到SDEdit对输入引导具有很高的容忍度，并且在此实验的所有设置中始终优于基线。

**表4：我们在LSUN-church数据集上比较SDEdit与基线用于基于笔画的生成定量结果。"颜色数量"表示用于生成合成笔画画的颜色数量，较少的颜色对应较少准确且较少详细的输入引导。我们观察到SDEdit在所有设置中都一致地实现更真实和更忠实的输出，并在此实验中的所有设置上超越基线。**

| 颜色数 | StyleGAN2-ADA | | e4e | | SDEdit (ours) | |
|--------|---|---|---|---|---|---|
| | KID ↓ | L₂ ↓ | KID ↓ | L₂ ↓ | KID ↓ | L₂ ↓ |
| 3 | 0.1588 | 67.22 | 0.0379 | 70.73 | **0.0233** | **36.00** |
| 6 | 0.1544 | 72.41 | 0.0354 | 68.53 | **0.0156** | **37.67** |
| 16 | 0.0923 | 69.52 | 0.0319 | 68.20 | **0.0135** | **37.70** |
| 30 | 0.0911 | 67.11 | 0.0304 | 68.66 | **0.0128** | **37.42** |
| 50 | 0.0922 | 65.28 | 0.0307 | 68.80 | **0.0126** | **37.40** |

### B.2 用SDEdit进行灵活的图像编辑

在本节中，我们进行了额外的图像编辑实验，包括编辑闭眼、张嘴和改变唇色。我们观察到SDEdit仍能取得合理的编辑结果，这表明SDEdit能够胜任灵活的图像编辑任务。

### B.3 对 t₀ 的分析

在本节中，我们提供了关于 t₀ 影响的额外分析（见图12）。如图3所示，我们可以调节 t₀ 来在忠实性和真实感之间进行权衡——较小的 t₀ 对应更忠实但较不真实的生成图像。如果我们想保留图12中的棕色笔划，我们可以减小 t₀ 以增加其忠实性，但这可能会降低其真实感。更多分析见附录D.2。

### B.4 与其他基线的额外比较

我们在图13中与 SC-FEGAN（Jo & Park, 2019）进行了额外比较。我们观察到当使用相同的笔划输入引导时，SDEdit能够比 SC-FEGAN 获得更真实的结果。我们还展示了 SC-FEGAN 在使用额外草图和笔划一起作为输入引导时的结果（见图14）。我们观察到即使在 SC-FEGAN 同时使用草图和笔划作为输入引导的情况下，SDEdit在真实感方面仍然能够超越 SC-FEGAN。

### B.5 与 Song et al. (2021) 的比较

Song et al. (2021) 提出的方法引入了一个额外的噪声条件分类器用于条件生成，分类器的性能对条件生成性能至关重要。他们的设置更接近于常规的逆问题，其中测量函数是已知的，这在第3节中已讨论。由于我们没有用于用户生成引导的已知"测量"函数，他们的方法不能直接应用于以操作像素RGB值形式进行的用户引导图像合成或编辑。为了应对这一限制，SDEdit基于用户输入初始化反向SDE并相应地调整 t₀——这是一种不同于 Song et al. (2021) 的方法（他们总是使用相同的初始化）。该技术使SDEdit能够在不学习额外任务特定模型（例如 Song et al. (2021) 中的附加分类器）的情况下实现忠实且真实的图像编辑或生成结果。

对于实际应用，我们在基于笔划的图像合成和编辑上与 Song et al. (2021) 进行了比较，其中我们不学习额外的噪声条件分类器（见图15）。事实上，我们也无法学习噪声条件分类器，因为我们没有用于用户生成引导的已知"测量"函数，而且我们只有一个随机的用户输入引导而非一组输入引导。我们观察到 Song et al. (2021) 的这种应用通过执行随机图像修复未能生成忠实的结果（见图15）。另一方面，SDEdit无需学习额外的任务特定模型（例如附加分类器）即可生成既真实又忠实的图像，并且可以直接应用于预训练的基于SDE的生成模型，从而实现使用基于SDE模型的引导图像合成和编辑。我们相信这展示了SDEdit的新颖性和贡献。

---

## 附录C SDEdit的详情

### C.1 VP和VE SDE的详情

我们遵循 Song et al. (2021) 中 VE 和 VP SDE 的定义，并采用其中相同的设置。

#### VE-SDE

具体而言，对于 VE SDE，我们选择

$$\sigma(t) = \begin{cases} 0, & t = 0 \\ \sigma_{\min}(\sigma_{\max}/\sigma_{\min})^t, & t > 0 \end{cases}$$

其中 σ_min = 0.01，σ_max 分别为 380、378、348、1348，对应 LSUN churches、bedroom、FFHQ/CelebA-HQ 256×256 和 FFHQ 1024×1024 数据集。

#### VP-SDE

对于 VP SDE，其形式为

$$d\mathbf{x}(t) = -\frac{1}{2}\beta(t)\mathbf{x}(t)dt + \sqrt{\beta(t)}d\mathbf{w}(t)$$

其中 β(t) 是一个正值函数。在实验中，我们遵循 Song et al. (2021); Ho et al. (2020); Dhariwal & Nichol (2021) 并设置

$$\beta(t) = \beta_{\min} + t(\beta_{\max} - \beta_{\min})$$

对于 Song et al. (2021); Ho et al. (2020) 训练的SDE，我们使用 β_min = 0.1 和 β_max = 20；对于 Dhariwal & Nichol (2021) 训练的SDE，模型学习基于相同的 β_min 和 β_max 选择来重新缩放方差。在这些设置下我们始终有 p₁(**x**) ≈ 𝒩(**0**, **I**)。

求解反向 VP SDE 类似于求解反向 VE SDE。具体而言，我们遵循以下的迭代规则：

$$\mathbf{x}_{n-1} = \frac{1}{\sqrt{1 - \beta(t_n)\Delta t}}\left(\mathbf{x}_n + \beta(t_n)\Delta t \, \mathbf{s}_\theta(\mathbf{x}(t_n), t_n)\right) + \sqrt{\beta(t_n)\Delta t} \, \mathbf{z}_n$$

其中 **x**_N ∼ 𝒩(**0**, **I**)，**z**_n ∼ 𝒩(**0**, **I**)，n = N, N−1, ⋯, 1。

### C.2 随机微分编辑的详情

在生成过程中，算法1中详述的过程也可以重复 K 次，如算法2所示。注意算法1是算法2的一个特例：当 K=1 时，我们恢复为算法1。对于 VE-SDE，算法3将笔画画转换为照片级真实图像，通常会修改输入的所有像素。然而，在图像合成和基于笔划的编辑等情况下，输入的某些区域已经是照片级真实的，因此我们希望保持这些区域不变。为了表示特定区域，我们使用一个二值掩码 **Ω** ∈ {0,1}^{C×H×W}，对于可编辑像素取值为1，否则为0。我们可以推广算法3以限制在 **Ω** 定义的区域中进行编辑。

对于可编辑区域，我们用前向SDE扰动输入图像并通过反转SDE生成编辑，使用与算法3中相同的过程。对于不可编辑区域，我们照常扰动它，但仔细设计反向过程以保证恢复输入。具体而言，假设 **x** ∈ ℝ^{C×H×W} 是一个高度为 H、宽度为 W、具有 C 个通道的输入图像。我们的算法首先用从 t=0 到 t=t₀ 运行的SDE扰动 **x**(0) = **x** 以获得 **x**(t₀)。之后，我们对 **Ω** ⊙ **x**(t) 和 (**1** − **Ω**) ⊙ **x**(t) 使用不同的方法进行去噪，其中 ⊙ 表示逐元素乘积，0 ≤ t ≤ t₀。对于 **Ω** ⊙ **x**(t)，我们模拟反向SDE（Song et al., 2021）并通过与 **Ω** 的逐元素乘法投影结果。对于 (**1** − **Ω**) ⊙ **x**(t)，我们将其设置为 (**1** − **Ω**) ⊙ (**x** + σ(t)**z**)，其中 **z** ∼ 𝒩(**0**, **I**)。这里我们根据 σ(t) 逐渐减小噪声幅度，以确保 **Ω** ⊙ **x**(t) 和 (**1** − **Ω**) ⊙ **x**(t) 具有相当的噪声量。此外，由于当 t→0 时 σ(t)→0，这确保了 (**1** − **Ω**) ⊙ **x**(t) 收敛到 (**1** − **Ω**) ⊙ **x**，保持 **x** 的不可编辑部分完好无损。完整的SDEdit方法（针对VE-SDE）见算法3。我们在算法4中提供了VP-SDE的算法，在算法5中提供了对应的掩码版本。

通过对算法3或算法5使用不同的输入，我们可以用一个统一的方法执行多种图像合成和编辑任务，包括但不限于以下任务：

- **基于笔划的图像合成**：我们可以通过设置 **Ω** 中的所有条目为1来恢复算法2或算法4。

- **基于笔划的图像编辑**：假设 **x**^(g) 是用笔划标记的图像，**Ω** 掩盖不是笔划像素的部分。我们可以使用算法3协调 **x**^(g) 的两个部分以获得照片级真实图像。

- **图像合成**：假设 **x**^(g) 是由两个图像的元素叠加而成的图像，**Ω** 掩盖用户不想执行编辑的区域。我们可以使用算法3或算法5进行图像合成。

**算法2：引导图像合成与编辑（VE-SDE，多轮重复版）**

| | |
|---|---|
| **输入**: | **x**^(g)（引导）, t₀（SDE超参数）, N（总去噪步数）, K（总重复数） |
| Δt ← t₀/N | |
| **for** k ← 1 **to** K **do** | |
| &nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;**x** ← **x** + σ(t₀)**z** | |
| &nbsp;&nbsp;**for** n ← N **to** 1 **do** | |
| &nbsp;&nbsp;&nbsp;&nbsp;t ← t₀·n/N | |
| &nbsp;&nbsp;&nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;&nbsp;&nbsp;ϵ ← √(σ²(t) − σ²(t − Δt)) | |
| &nbsp;&nbsp;&nbsp;&nbsp;**x** ← **x** + ϵ² **s**_θ(**x**, t) + ϵ**z** | |
| &nbsp;&nbsp;**end for** | |
| **end for** | |
| **返回** **x** | |

**算法3：带掩码的引导图像合成与编辑（VE-SDE）**

| | |
|---|---|
| **输入**: | **x**^(g)（引导）, **Ω**（编辑区域掩码）, t₀（SDE超参数）, N（总去噪步数）, K（总重复数） |
| Δt ← t₀/N | |
| **x**₀ ← **x** | |
| **for** k ← 1 **to** K **do** | |
| &nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;**x** ← (**1** − **Ω**) ⊙ **x**₀ + **Ω** ⊙ **x** + σ(t₀)**z** | |
| &nbsp;&nbsp;**for** n ← N **to** 1 **do** | |
| &nbsp;&nbsp;&nbsp;&nbsp;t ← t₀·n/N | |
| &nbsp;&nbsp;&nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;&nbsp;&nbsp;ϵ ← √(σ²(t) − σ²(t − Δt)) | |
| &nbsp;&nbsp;&nbsp;&nbsp;**x** ← (**1** − **Ω**) ⊙ (**x**₀ + σ(t)**z**) + **Ω** ⊙ (**x** + ϵ² **s**_θ(**x**, t) + ϵ**z**) | |
| &nbsp;&nbsp;**end for** | |
| **end for** | |
| **返回** **x** | |

**算法4：引导图像合成与编辑（VP-SDE）**

| | |
|---|---|
| **输入**: | **x**^(g)（引导）, t₀（SDE超参数）, N（总去噪步数）, K（总重复数） |
| Δt ← t₀/N | |
| α(t₀) ← ∏_{n=1}^{N} (1 − β(n·t₀/N)Δt) | |
| **for** k ← 1 **to** K **do** | |
| &nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;**x** ← √(α(t₀))**x** + √(1 − α(t₀))**z** | |
| &nbsp;&nbsp;**for** n ← N **to** 1 **do** | |
| &nbsp;&nbsp;&nbsp;&nbsp;t ← t₀·n/N | |
| &nbsp;&nbsp;&nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;&nbsp;&nbsp;**x** ← 1/√(1 − β(t)Δt) · (**x** + β(t)Δt **s**_θ(**x**, t)) + √(β(t)Δt) **z** | |
| &nbsp;&nbsp;**end for** | |
| **end for** | |
| **返回** **x** | |

**算法5：带掩码的引导图像合成与编辑（VP-SDE）**

| | |
|---|---|
| **输入**: | **x**^(g)（引导）, **Ω**（编辑区域掩码）, t₀（SDE超参数）, N（总去噪步数）, K（总重复数） |
| Δt ← t₀/N | |
| **x**₀ ← **x** | |
| α(t₀) ← ∏_{i=1}^{N} (1 − β(i·t₀/N)Δt) | |
| **for** k ← 1 **to** K **do** | |
| &nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;**x** ← [(**1** − **Ω**) ⊙ √(α(t₀))**x**₀ + **Ω** ⊙ √(α(t₀))**x** + √(1 − α(t₀))**z**] | |
| &nbsp;&nbsp;**for** n ← N **to** 1 **do** | |
| &nbsp;&nbsp;&nbsp;&nbsp;t ← t₀·n/N | |
| &nbsp;&nbsp;&nbsp;&nbsp;**z** ∼ 𝒩(**0**, **I**) | |
| &nbsp;&nbsp;&nbsp;&nbsp;α(t) ← ∏_{i=1}^{n} (1 − β(i·t₀/N)Δt) | |
| &nbsp;&nbsp;&nbsp;&nbsp;**x** ← {(**1** − **Ω**) ⊙ (√(α(t))**x**₀ + √(1 − α(t))**z**) + **Ω** ⊙ [1/√(1 − β(t)Δt)(**x** + β(t)Δt **s**_θ(**x**, t)) + √(β(t)Δt) **z**)]} | |
| &nbsp;&nbsp;**end for** | |
| **end for** | |
| **返回** **x** | |

---

## 附录D 实验设置

### D.1 实现详情

#### 基于笔划的图像合成

我们使用 Song et al. (2021) 发布的模型在 LSUN bedroom 和 church 数据集上训练 SDE 模型。对于笔画画，我们基于原始图像使用一种算法（见附录D.2描述）进行模拟。我们使用相同的笔画画来评估SDEdit和其他基线方法。具体而言，我们使用了VE-SDE发布的最新检查点，该检查点在 256×256 大小的图像上训练。对于测试，我们为LSUN数据集的50张测试图像每张生成50个不同的随机种子，即每种方法得到2500个生成样本。对于评估，我们比较2500张合成图像与2500张地面真实参考图像之间的KID。对于忠实性，我们计算合成图像与输入笔画画之间的 L₂ 距离。我们在LSUN bedroom和LSUN church两个数据集上进行了评估。

#### 基于笔划的图像编辑

我们使用 Song et al. (2021) 发布的 VP-SDE 检查点在 LSUN bedroom、church 和 CelebA-HQ 数据集上进行测试。对于LSUN bedroom和church中的每张测试图像，我们通过手动指定笔划编辑来应用基于笔划的编辑。对于CelebA-HQ，我们使用了先前工作（Liu et al., 2021）中准备的基于笔划的编辑。对于LSUN数据集我们使用 t₀∈[0.3, 0.6]，对于CelebA-HQ数据集也使用 t₀∈[0.3, 0.6]。

#### 图像合成

我们在CelebA-HQ数据集上进行测试。我们要求用户通过从参考图像复制像素并将其粘贴到目标图像上来执行图像合成。然后我们使用SDEdit进行图像合成。对于此任务我们使用 t₀∈[0.3, 0.6]。

### D.2 合成笔画画

#### 人类笔划模拟算法

为了从原始图像合成笔画画，我们设计了一种自动模拟人类笔画画的算法。给定一张图像，我们的算法：(1) 使用K-means聚类提取主色（K=3、6、16、30或50，取决于笔划简单度设置），(2) 对每种颜色，找到具有相似颜色的连通区域，(3) 对每个连通区域，应用中值滤波后进行边缘检测以找到笔划边界，(4) 使用样条插值生成平滑的笔划。所得的笔画画根据算法中使用的聚类数量具有不同层次的保真度。

#### KID评估

我们使用KID（Kernel Inception Distance）来衡量合成图像的质量。具体而言，我们计算合成图像与验证集中50K生成样本之间的KID。

#### 真实感-忠实性权衡

我们分析了 t₀ 对真实感和忠实性之间权衡的影响。我们使用相同的LSUN bedroom测试图像集，并用我们的算法（K=6）生成笔画画。对于 t₀ 从0到1的每个值（步长为0.1），我们为每张笔画画生成一张图像。然后我们计算每张生成图像的KID分数和 L₂ 距离。如图3所示，我们观察到随着 t₀ 的增加，KID分数降低（更真实）但 L₂ 距离增加（更不忠实）。

### D.3 训练和推理时间

对于LSUN数据集上的VE-SDE模型，在单个NVIDIA V100 GPU上使用1000个离散化步数，推理一张 256×256 图像大约需要50-100秒。对于CelebA-HQ上的VP-SDE模型，使用100个离散化步数，推理一张 256×256 图像大约需要5-10秒。这些数字可以通过更快的SDE求解器或用更少离散化步数训练的SDE模型进一步改善。

---

## 附录E 额外实验结果

### E.1 LSUN数据集上的额外结果

#### 基于笔划的图像生成

[展示了LSUN bedroom和church数据集上基于笔划生成的额外结果示例]

#### 基于笔划的图像编辑

[展示了LSUN bedroom和church数据集上基于笔划编辑的额外结果示例]

### E.2 人脸数据集上的额外结果

#### 基于笔划的图像编辑

[展示了CelebA-HQ人脸图像上基于笔划编辑的额外结果]

#### 图像合成

[展示了CelebA-HQ人脸图像上图像合成的额外结果]

#### 用基于笔划生成进行属性分类

[展示了使用基于笔划生成进行属性分类的额外结果]

### E.3 带笔画画的类条件生成

[展示了带笔画画的类条件生成的额外结果]

### E.4 额外数据集

[展示了LSUN和CelebA-HQ之外的额外数据集上的结果]

### E.5 基线的额外结果

[展示了与基线方法的额外比较结果]

---

## 附录F 人工评估

### F.1 基于笔划的图像生成

对于基于笔划生成的人类评估，我们招募了Amazon Mechanical Turk工作人员来比较SDEdit和基线方法的结果。具体而言，对于每对方法，我们向工作人员展示输入笔画画以及两种方法生成的图像，并要求他们评价哪张图像更真实。我们为每次比较收集了多位工作人员的反馈，并报告了偏好SDEdit的工作人员百分比。

### F.2 CelebA-HQ上的图像合成

对于图像合成的人类评估，我们招募了Amazon Mechanical Turk工作人员来比较SDEdit和基线方法的结果。我们要求工作人员评价哪张图像 (1) 更真实，(2) 综合考虑真实性和忠实性后更令人满意。我们为每次比较收集了多位工作人员的反馈，并报告了偏好SDEdit的工作人员百分比。

---

*本文翻译基于 arXiv:2108.01073v2 版本，力求准确传达原文的技术内容和学术表达。*