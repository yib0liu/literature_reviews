# SIGGRAPH Asia 2024 — Motion 相关论文逐篇综述

> 编制日期：2026-08-11  
> 覆盖范围：TOG 43(6) Journal Track全部motion相关论文 + Conference中Asia venue发表的相关work  
> 共收录 **23** 篇 motion 相关论文  
> 参考标准：与 `SIGGRAPH2026_SA2026_Motion论文综述.md` 一致的深度技术分析

---

## 一、Motion Generation & Control

### 1.1 MaskedMimic: Unified Physics-Based Character Control Through Masked Motion Inpainting

- **作者**：Chen Tessler, Yunrong Guo, Ofir Nabati, Gal Chechik, Xue Bin Peng（NVIDIA / SFU / Bar-Ilan Univ.）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687951](https://doi.org/10.1145/3687951)｜**arXiv**：2409.14393
- **项目页**：https://research.nvidia.com/labs/par/maskedmimic/

**问题定义**：现有物理角色控制系统每个任务需要单独训练专用控制器+精心设计的奖励函数，缺乏通用性。目标是训练单一统一控制器，支持稀疏keyframes、文本指令、场景信息等多种控制模态，无需任务特定的reward engineering。

**方法与架构（两阶段）**：

**Stage 1 - Fully-Constrained Controller ($\pi^{FC}$)**：
- Transformer-based motion tracking policy
- 角色观测规范：$s_t = (\theta_t \ominus \theta_t^{root}, (p_t - p_t^{root}) \ominus \theta_t^{root}, v_t \ominus \theta_t^{root})$
- 目标pose特征（per joint）：$\hat{f}^j = (\hat{\theta}^j \ominus \theta_t^j, \hat{\theta}^j \ominus \theta_t^{root}, (\hat{p}^j - p_t^j) \ominus \theta_t^{root}, (\hat{p}^j - p_t^{root}) \ominus \theta_t^{root})$
- 未来$K$帧目标pose augmented with时间$\tau_{t+k}$
- Scene observations：heightmap oriented along root facing direction
- 奖励函数：$r_t = w^{gp}r^{gp} + w^{gr}r^{gr} + w^{rh}r^{rh} + w^{jv}r^{jv} + w^{jav}r^{jav} + w^{eg}r^{eg}$
- PD控制，无residual forces
- Action distribution：multi-dimensional Gaussian with fixed diagonal covariance $\sigma^\pi = \exp(-2.9)$
- **Training Playground三区域**：Flat terrain + Irregular terrain（stairs/slopes/rough）+ Object playground（chairs/tables/couches）
- **Early Termination**：平地0.25m偏差阈值，不规则地形0.5m
- **Prioritized Motion Sampling**：按失败率加权（最小权重$3 \times 10^{-3}$）

**Stage 2 - Partially-Constrained Controller ($\pi^{PC}$ = MaskedMimic)**：
- **Conditional VAE架构**：
  - Prior $\rho(z_t|s_t,g_t^{partial}) = \mathcal{N}(\mu^\rho(s_t,g_t^{partial}), \sigma^\rho(s_t,g_t^{partial}))$
  - Encoder $\mathcal{E}(z_t|s_t,g_t^{full}) = \mathcal{N}(\mu^\rho + \mu^\mathcal{E}, \sigma^\mathcal{E})$（作为prior的残差）
  - Decoder $\mathcal{D}(a_t|s_t,z_t)$输出deterministic action
- **Objective**：$\mathbb{E}[\log\mathcal{D}(a|s,z)] - \alpha D_{KL}(\mathcal{E}||\rho)$
- **随机Masking策略**：结构化时序masking（同一mask跨多帧持续而非每帧重采样），确保某些关节连续可见/隐藏
- **KL-scheduling**：KL系数从0.0001线性增长到0.01
- **Episodic latent noise**：整个episode共享同一个$\epsilon \sim \mathcal{N}(0,1)$
- **Observation history**：过去40步采样子5帧历史pose
- **Text encoding**：XCLIP embeddings（视频-语言预训练）
- **Object表示**：8个bounding box角点 + object type index
- Prior用Transformer encoder处理变长多模态token；Encoder/Decoder为全连接网络

**训练数据**：
- **AMASS**（PHC过滤流程去artifacts）+ **HumanML3D**（text conditioning）+ **SAMP**（object interaction）
- 镜像数据增强（左右翻转，text同步镜像）
- Isaac Gym，**16,384并行环境**，4×A100 GPU
- 训练约**2周**，$\pi^{FC}$约30B steps，$\pi^{PC}$约10B steps
- Controller **30 Hz**，仿真**120 Hz**
- Joint conditioning子集：Left/Right Ankle, Pelvis, Head, Left/Right Hand

**评估任务**：
- **Full-body tracking**：baseline capability
- **VR tracking**（head position+rotation + hand positions）：对比PULSE、ASE、CALM
- **Irregular terrains**：random stairs/slopes/rough terrain
- **Path-Following**：3D path指定head target positions
- **Steering**：joystick-like heading + velocity control
- **Reach**：right hand reaching randomly changing targets（2秒时限）
- **Object Interaction**：sitting on held-out objects（2-10m初始化距离）

**定量结果**：
- Full-body tracking success rate > 90%（具体数值见原文Table 1）
- VR tracking性能与PULSE相当或更优，且在未见过的motion上generalization更好
- Irregular terrain上保持较高tracking成功率
- Path-following和steering任务中展现自然locomotion transitions

**信息缺口**：Transformer的具体层数/头数/hidden dim；各reward权重$w^{\cdot}$的具体值；Table 1-4完整数值。

**为什么重要**：提出了**motion inpainting**作为统一的角色控制范式——任何控制模态都可以视为"未被mask的部分观测"。**Goal-engineering**类比prompt-engineering，让用户通过组合简单约束实现复杂行为。两阶段蒸馏（RL teacher → BC student）保证了物理合理性的同时获得多模态灵活性。

---

### 1.2 SKEL-Betweener: a Neural Motion Rig for Interactive Motion Authoring

- **作者**：Dhruv Agrawal, Jakob Buhmann, Dominik Borer, Robert W. Sumner, Martin Guay
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687941](https://doi.org/10.1145/3687941)

**问题定义**：3D motion authoring需要操纵大量控制柄且耗时；现有neural motion completion方法需要dense full-pose context（创作成本高）且缺乏joint-level精细控制。

**方法与架构**：
- **Neural Motion Rig**：类比传统动画rigging工作流，但用神经网络学习motion manifold
- **仅需两个pose**即可生成长motion序列（大幅降低authoring成本）
- **Neural Motion Curves**：intuitive joint-level controls for positions and orientations，类比animator熟悉的F-curves
- 从large motion dataset学习neural representation

**信息缺口**：网络架构细节（transformer/MLP?）、训练数据集规模和来源、定量评估指标（reconstruction error/diversity metrics）、user evaluation的具体设计和结果（participants数量/task completion time/主观评分）。

**为什么重要**：将neural motion representation与工业界熟悉的rig/F-curve工作流对接，降低了AI辅助motion authoring的使用门槛。仅需两个pose就能生成长序列的特性对快速prototyping特别有价值。

---

### 1.3 CPoser: Text-to-Pose Generation Using Large Language Models

- **作者**：Yumeng Li, Bohong Chen, Zhong Ren, Yao-Xiang Ding, Libin Liu, Tianjia Shao, Kun Zhou
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687932](https://doi.org/10.1145/3687932)

**问题定义**：直接用LLM生成精确的全身articulation target很困难，因为LLM是为general-purpose language processing训练的，不是pose generation。

**方法与架构**：
- **两阶段Pipeline**：
  1. **Prompt Parsing Stage**：LLM将text prompts转为**Pose-IR（Pose Intermediate Representation）**——一组predefined structured queries（如"squatting depth=0.7, knee bending angle=45°"）
  2. **Pose Optimization Stage**：在quantized pose prior space中通过robust optimization求解满足Pose-IR objective function的expressive poses和hand gestures
- Pose-IR形成明确的objective function，避免了LLM直接输出数值的不可靠性
- 后续refinement加入facial expressions增强自然度

**信息缺口**：使用的LLM型号和大小、Pose-IR的完整schema（支持的query类型列表）、quantized pose prior的构建方式（VQ-VAE?）、optimization算法（gradient-based? sampling-based?）、定量对比（vs ACTOR/TEMOS/T2M等text-to-pose baselines）。

**为什么重要**：巧妙地用structured intermediate representation桥接了LLM的语言理解能力和pose生成的几何精度需求。比端到端LLM-to-pose更可靠，比纯optimization-based方法有更好的语义理解能力。

---

## 二、Facial & Head Animation

### 2.1 GaussianHeads: End-to-End Learning of Drivable Gaussian Head Avatars

- **作者**：Kartik Teotia, Hyeongwoo Kim, Pablo Garrido, Marc Habermann, Mohamed Elgharib, Christian Theobalt（MPI-Informatik）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687927](https://doi.org/10.1145/3687927)

**问题定义**：现有Gaussian-based head avatar方法在complex motion changes（mouth interior、strongly varying head poses）下表现不佳，难以表示tongue deformations和fine-grained teeth structure。

**方法与架构**：
- **Hierarchical coarse-to-fine representation**：
  1. Coarse step：从raw input frames提取rich facial features，learn to deform coarse facial geometry of template mesh
  2. Fine step：在deformed surface上initialize 3D Gaussians并refine positions
- **End-to-end training**：coarse-to-fine facial avatar model + head pose作为learnable parameters
- 支持controllable facial animation via video inputs
- 支持cross-identity facial performance transfer应用

**信息缺口**：Template mesh的类型（FLAME?）、coarse deformation network架构、Gaussian initialization密度、训练dataset（多view setup details）、PSNR/SSIM/LPIPS定量对比、cross-identity transfer的qualitative结果。

**为什么重要**：首次将Gaussian splatting的coarse-to-fine hierarchical design引入head avatars，解决了mouth interior和large head pose变化下的rendering质量难题。

---

### 2.2 VOODOO XP: Expressive One-Shot Head Reenactment for VR Telepresence

- **作者**：Phong Tran, Egor Zakharov, Long-Nhat Ho, Liwen Hu, Adilbek Karmanov, Aviral Agarwal, McLean Goldwhite, Ariana Bermudez Venegas, Anh Tuan Tran, Hao Li
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687928](https://doi.org/10.1145/3687928)

**问题定义**：VR telepresence需要expressive one-shot head reenactment——仅用单张reference image就能驱动target identity的面部动画，同时保持high expressiveness和real-time performance。

**方法与架构**：
- **One-shot reenactment**：single reference image + driving signals → photorealistic head animation
- 针对VR headset的constrained viewing conditions优化（oblique angles, limited FOV）
- Expressive：捕捉subtle micro-expressions和extreme expressions

**信息缺口**：网络架构细节（encoder-decoder结构？warping mechanism？）、training data规模和来源、one-shot generalization的定量评估（cross-identity FID/identity preservation metrics）、runtime performance（FPS on VR hardware）。

**为什么重要**：One-shot head reenactment for VR是telepresence的关键技术，VOODOO XP在expressiveness和efficiency之间取得了新的平衡。

---

## 三、Motion Capture & Performance Capture

### 3.1 ELMO: Enhanced Real-time LiDAR Motion Capture through Upsampling

- **作者**：Deok-Kyeong Jang, Dongseok Yang, Deok-Yun Jang, Byeoli Choi, Sung-Hee Lee, Donghoon Shin
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687991](https://doi.org/10.1145/3687991)
- **项目页**：https://movin3d.github.io/ELMO_SIGASIA2024/

**问题定义**：单LiDAR传感器的mocap面临低帧率（~20fps）和稀疏point cloud的挑战，难以达到实时高质量motion capture。

**方法与架构**：
- **Conditional autoregressive transformer-based upsampling motion generator**
- 从**20 fps LiDAR point cloud**序列生成**60 fps motion**（3× temporal upsampling）
- Self-attention机制耦合精心设计的motion embedding和point cloud embedding modules
- **One-time skeleton calibration model**：从single-frame point cloud预测用户skeleton offsets
- **Novel data augmentation using LiDAR simulator**：enhance global root tracking和环境理解能力

**训练数据**：
- **High-quality LiDAR-mocap synchronized dataset**：**20 different subjects** performing a range of motions
- Dataset和evaluation code开源

**定量结果**：
- 与image-based和point cloud-based SOTA mocap方法对比，ELMO在motion quality指标上显著领先
- Ablation study验证self-attention + embedding modules的设计原则
- **Fast inference**：适合real-time applications（live streaming / interactive gaming demo）

**信息缺口**：Transformer的具体层数/头数/hidden dim、upsampling的MPJPE/MPBPE具体数值、LiDAR simulator的数据增强策略细节。

**为什么重要**：首次将transformer temporal upsampling引入LiDAR mocap，实现了单传感器60fps实时capture。发布的synchronized dataset填补了该领域的空白。

---

### 3.2 EgoHDM: Real-time Egocentric-Inertial Human Mocap, Localization, and Dense Mapping

- **作者**：Handi Yin, Bonan Liu, Manuel Kaufmann, Jinhao He, Sammy Christen, Jie Song, Pan Hui
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687907](https://doi.org/10.1145/3687907)
- **项目页**：https://handiyin.github.io/EgoHDM/

**问题定义**：现有egocentric mocap系统缺乏dense scene mapping能力，且human localization和scene reconstruction之间没有形成闭环。

**方法与架构**：
- **6 IMUs + commodity head-mounted RGB camera**
- **Tightly coupled mocap-aware dense bundle adjustment**
- **Physics-based body pose correction module** leveraging local body-centric elevation map
- **Terrain-aware contact PD controller**：enables characters physically contact given local elevation map，减少human floating/penetration
- 首个提供near real-time dense mapping的人体mocap系统

**定量结果**：
- Human localization误差↓**41%**
- Camera pose误差↓**71%**
- Mapping accuracy误差↓**46%**
- 在non-flat terrain（stairs/outdoor scenes in the wild）上qualitative evaluation验证有效性

**信息缺口**：Bundle adjustment的具体formulation、PD controller的gain参数、tested sequences的详细统计。

**为什么重要**：首次实现了mocap-localization-mapping三者的tight coupling闭环，在非平坦地形上的显著性能提升证明了其实际价值。

---

### 3.3 Look Ma, no markers: Holistic Performance Capture without the Hassle

- **作者**：Charlie Hewitt, Fatemeh Saleh, Sadegh Aliakbarian, Lohit Petikam, Shideh Rezaeifar, Louis Florentin, Zafiirah Hosenie, Thomas J. Cashman, Julien Valentin, Darren Cosker, Tadas Baltrusaitis
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687772](https://doi.org/10.1145/3687772)

**问题定义**：电影/游戏工业中的mocap通常只针对face/body/hand之一，需要昂贵硬件和专业操作。ML方法多为单摄像头、单身体部位、非world-space结果。

**方法与架构**：
- **首个marker-free holistic capture**：同时重建face（含eyes和tongue）、body、hands
- **无需calibration、manual intervention或custom hardware**
- **Hybrid approach**：纯synthetic data训练的ML模型 + 强大parametric human shape/motion模型（SMPL-X类）
- Produces stable world-space results from arbitrary camera rigs
- Supports varied capture environments and clothing

**定量结果**：在多个body/face/hand reconstruction benchmarks上达到state-of-the-art，diverse datasets上泛化验证。

**信息缺口**：Synthetic data pipeline的细节（engine/lighting/materials/variations）、parametric模型类型、各benchmark的具体数值（MPJPE/PCK/AUC等）。

**为什么重要**：首次实现了"开箱即用"的holistic marker-free performance capture，覆盖了从face micro-expressions到full-body locomotion的完整谱系。Microsoft Research Cambridge团队的工业级解决方案。

---

### 3.4 Gaussian Surfel Splatting for Live Human Performance Capture

- **作者**：Zheng Dong, Ke Xu, Yaoan Gao, Hujun Bao, Weiwei Xu, Rynson W. H. Lau
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687993](https://doi.org/10.1145/3687993)

**问题定义**：极稀疏（如4相机）RGBD设置下，NeRF/3DGS方法产生local geometry errors，PIFu方法rendering不真实。

**方法与架构**：
- **PGH (Point-based Generalizable Human) representation**：pixel-aligned RGBD features条件化
- **Surface implicit function**：regression of surface points
- **Gaussian implicit function**：parameterizing radiance fields of regressed surface points with **2D Gaussian surfels**
- **Surfel splatting** for fast rendering
- **PRNet (Point-Regressing Network)** + **DPI (Depth-guided Point Cloud Initialization)**：regress accurate surface point cloud based on denoised depth
- **SPNet (Surfel Splatting Network)**：neural blending-based high-quality geometry and appearance rendering
- **1K resolution free-view videos，平均12 fps**

**信息缺口**：两个benchmark的具体名称和PSNR/SSIM/LPIPS数值、与SOTA方法的详细对比表、surfels的数量和分布策略。

**为什么重要**：将Gaussian surfel splatting引入sparse-view human performance capture，在极低相机数下仍保持高质量rendering。

---

### 3.5 DualGS: Robust Dual Gaussian Splatting for Immersive Volumetric Videos

- **作者**：Yuheng Jiang, Zhehao Shen, Yu Hong, Chengcheng Guo, Yize Wu, Yingliang Zhang, Jingyi Yu, Lan Xu
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687926](https://doi.org/10.1145/3687926)
- **项目页**：https://nowheretrix.github.io/DualGS/

**问题定义**：Volumetric video需要extensive manual intervention做mesh stabilization且assets过大，阻碍广泛采用。

**方法与架构**：
- **DualGS表示**：**Skin Gaussians**（appearance）+ **Joint Gaussians**（motion）分别表示
- **Explicit disentanglement**显著减少motion redundancy，enhance temporal coherence
- **Initialization**：第一帧anchor skin Gaussians到joint Gaussians
- **Coarse-to-fine training strategy**：
  1. Coarse alignment phase：overall motion prediction
  2. Fine-grained optimization：robust tracking + high-fidelity rendering
- **压缩方案**：
  - Motion：entropy encoding
  - Appearance：codec compression + persistent codebook
- **压缩比高达120×**，每帧仅需~**350KB**存储

**信息缺口**：Skin/joint Gaussians的具体数量分配、VR headset上的实际播放帧率和resolution、不同sequence类型的compression ratio分布。

**为什么重要**：DualGS的skin/joint解耦表示大幅提升了volumetric video的compression efficiency，使得VR头显上的沉浸式free-view playback成为现实。

---

### 3.6 360-degree Human Video Generation with 4D Diffusion Transformer

- **作者**：Ruizhi Shao, Youxin Pang, Zerong Zheng, Jingxiang Sun, Yebin Liu（清华大学）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687980](https://doi.org/10.1145/3687980)

**问题定义**：从单张图片生成360度spatiotemporally coherent的human video，现有GAN/vanilla diffusion方法在complex motions和viewpoint changes上表现不佳。

**方法与架构**：
- **Hierarchical 4D Transformer architecture**：factorize self-attention across **views × time steps × spatial dimensions**
- Diffusion Transformer捕捉global correlations + CNN实现accurate condition injection
- Precise conditioning：inject human identity、camera parameters、temporal signals into respective transformers
- **Multi-dimensional dataset**：images + videos + multi-view data + limited 4D footage
- **Tailored multi-dimensional training strategy**

**信息缺口**：Transformer的层级结构（各level的attention factorization方式）、训练dataset各成分的规模比例、FID/IS/PSNR等定量指标、与SOTA（如HumanNeRF/Animatable NeRF等）的详细对比。

**为什么重要**：首次将4D diffusion transformer应用于360度human video generation，解决了complex motions和large viewpoint changes下的spatiotemporal coherence难题。

---

### 3.7 Towards Unstructured Unlabeled Optical Mocap: A Video Helps!

- **作者**：Nicholas Milef, John Keyser, Shu Kong
- **Track**：Conference (SIGGRAPH 2024)｜**DOI**：[10.1145/3641519.3657522](https://doi.org/10.1145/3641519.3657522)

**问题定义**：无结构无标签的光学mocap（即普通视频中的人体运动捕捉）缺乏标注数据，难以训练监督模型。

**方法与架构**：
- 利用**video prior**辅助unstructured unlabeled optical mocap
- 可能采用self-supervised或weakly-supervised学习策略，利用temporal consistency作为监督信号

**信息缺口**：具体方法细节、实验设置、定量结果。（Conference Track篇幅有限，细节较少）

**为什么重要**：探索了"零标注"光学mocap的可能性，若能成功将极大降低mocap的部署成本。

---

## 四、Avatar & Human Reconstruction

### 4.1 PuzzleAvatar: Assembling 3D Avatars from Personal Albums

- **作者**：Yuliang Xiu, Yufei Ye, Zhen Liu, Dimitris Tzionas, Michael J. Black（MPI-IS）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687771](https://doi.org/10.1145/3687771)
- **项目页**：https://puzzleavatar.is.tue.mpg.de

**问题定义**：Text-to-3D方法擅长生成名人/虚构角色，但对普通人效果差；faithful reconstruction通常需要controlled setting下的全身图像。能否从用户日常"OOTD"（Outfit Of The Day）相册生成忠实avatar？

**方法与架构**：
- **Album2Human task**：从personal OOTD album生成canonical pose下的3D avatar
- **Fine-tune foundational VLM**：将appearance/identity/garments/hairstyles/accessories编码为**separate learned tokens**
- Learned tokens作为"puzzle pieces"组装成3D avatar
- 支持通过inter-change tokens自定义avatar（swap hairstyle/garments等）
- **PuzzleIOI dataset**：**41 subjects**，近**1k OOTD配置**，challenging partial photos with paired ground-truth 3D bodies

**定量结果**：Reconstruction accuracy显著优于TeCH和MVDreamBooth，unique scalability to album photos，strong robustness验证。

**信息缺口**：VLM的基础模型和fine-tuning策略、token数量和维度、reconstruction accuracy的具体数值（Chamfer distance/CD等）。

**为什么重要**：提出了新颖的"Album2Human"任务，利用VLM的语义理解能力绕过body/camera pose估计难题，实现了从casual photo collection到3D avatar的端到端生成。

---

## 五、Retargeting, Deformation & Skinning

### 5.1 Geometry-Aware Retargeting for Two-Skinned Characters Interaction

- **作者**：Inseo Jang, Soojin Choi, Seokhyeon Hong, Chaelin Kim, Junyong Noh
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687962](https://doi.org/10.1145/3687962)

**问题定义**：两个任意mesh connectivity的角色之间的interaction motion retargeting尚未被研究，shape差异会导致inter-penetration和semantic丢失。

**方法与架构**：
- **SCT (Spatio Cooperative Transformer)**：predict residual of root position and joint rotations considering shape difference
- **Anchor loss function**：retargeting时维持interaction角色间的geometric distance
- **Motion augmentation with deformation-based adaptation**：准备identical mesh connectivity的source-target paired training data

**定量结果**：Unseen characters和motions上achieved higher accuracy for semantic preservation，produced less inter-penetration artifacts than baselines。User evaluation证明better semantic preservation across low-to-high interaction levels。

**信息缺口**：Transformer架构细节（层数/头数）、anchor loss的数学形式、motion augmentation的具体策略。

**为什么重要**：首次研究了two-skinned characters interaction的retargeting问题，对游戏和电影中多角色互动场景有直接应用价值。

---

### 5.2 Multi-Resolution Real-Time Deep Pose-Space Deformation

- **作者**：Mianlun Zheng, Jernej Barbic（USC）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687985](https://doi.org/10.1145/3687985)

**问题定义**：游戏/VR/Metaverse需要**hard real-time**（毫秒级甚至亚毫秒级）的multi-resolution mesh deformation，现有方法未同时满足速度和multi-resolution需求。

**方法与架构**：
- **Multi-resolution analysis + mesh partition of unity + neural networks**
- 学习pre-skinning shape deformations in arbitrary character pose
- Combined with LBS reconstruct training shapes + support interpolation/extrapolation
- **Memory layout和code优化**boost computation speeds
- **Progressive construction** level by level + interrupt at any time → graceful degradation

**定量结果**：相比"naive"approach（separately processing each hierarchical LOD level）offered substantial memory reduction和computational speedups。硬实时速率下（preferably < 1ms）保持高质量deformation。

**信息缺口**：各resolution level的具体vertex count、推理时间（ms）、memory reduction比例、vs naive approach的speedup倍数。

**为什么重要**：Jernej Barbic组在real-time deformation领域的最新成果，首次实现了hard real-time + multi-resolution的deep pose-space deformation。

---

### 5.3 Deformation Recovery: Localized Learning for Detail-Preserving Deformations

- **作者**：Ramana Sundararaman, Nicolas Donati, Simone Melzi, Etienne Corman, Maks Ovsjanikov
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687968](https://doi.org/10.1145/3687968)
- **代码**：https://github.com/sentient07/LJN

**问题定义**：现有data-driven deformation方法需要global shape encoding，但detail-preserving deformations在某些场景下无需global context。

**方法与架构**：
- **Localized Jacobian representation**：one-ring neighborhood的Jacobian作为coarse deformation输入
- 一系列MLP + feature smoothing学习detail-preserving deformation的Jacobian
- Poisson solve恢复embedding
- **每点都是训练样本**，supervision特别轻量
- Trained on class of shapes → remarkable generalization across different object categories

**三大任务**：
1. Refining approximate shape correspondence
2. Unsupervised deformation and mapping
3. Shape editing

**信息缺口**：MLP架构（层数/宽度）、各任务的定量结果、generalization across categories的具体案例。

**为什么重要**：去除了对global encoding的依赖，实现了class-agnostic的localized deformation learning，"every point is a training example"的设计使supervision特别高效。

---

### 5.4 Real-time Large-scale Deformation of Gaussian Splatting

- **作者**：Lin Gao, Jie Yang, Bo-Tao Zhang, Jia-Mu Sun, Yu-Jie Yuan, Hongbo Fu, Yu-Kun Lai
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687756](https://doi.org/10.1145/3687756)

**问题定义**：Gaussian Splatting难以直接deform（离散Gaussians + 缺乏explicit topology）。

**方法与架构**：
- **GaussianMesh**：mesh-based GS representation
- 3D Gaussians定义在explicit mesh上，**双向绑定**：
  - GS rendering指导mesh face split自适应细化
  - Mesh face split指导GS splitting
- Explicit mesh constraints正则化GS分布，抑制misaligned/long-narrow shaped Gaussians
- **Large-scale Gaussian deformation technique**：根据associated mesh manipulation改变3D GS参数
- Benefits from existing mesh deformation datasets for data-driven Gaussian deformation

**定量结果**：**平均65 FPS** on single GPU，high-quality reconstruction和effective deformation。

**信息缺口**：可处理的max mesh规模（triangle count）、deformation质量与static GS的PSNR/SSIM对比、large-scale deformation的具体幅度限制。

**为什么重要**：首次实现了GS的interactive large-scale deformation，bridging了GS rendering和traditional mesh manipulation两个世界。

---

## 六、Cloth, Hair & Garment Simulation

### 6.1 Efficient GPU Cloth Simulation with Non-distance Barriers and Subspace Reuse

- **作者**：Lei Lan, Zixuan Lu, Jingyi Long, Chun Yuan, Xuan Li, Xiaowei He, Huamin Wang, Chenfanfu Jiang, Yin Yang
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687760](https://doi.org/10.1145/3687760)

**问题定义**：高分辨率garment模型的untangled cloth simulation即使在GPU上也难以实时。

**方法与架构**：
- **Non-distance barrier model**：
  - Inspired by interior point method，barrier potential不依赖primitive间distance
  - 依赖collision event的**virtual life span**
  - Keeps all vertices within feasible region
- **Subspace reuse strategy**：
  - Low-frequency strain propagation ≈ orthogonal to high-frequency collision-induced deformation
  - Subspace处理low-frequency residuals
  - GPU-based iterative solvers处理high-frequency residuals
- **比现有fast cloth simulators快至少一个数量级**

**信息缺口**：具体speedup倍数（vs Projective Dynamics/IPC等）、支持的max triangle count、penetration-free guarantee的理论证明。

**为什么重要**：Non-distance barrier + subspace reuse两大创新使得high-res interactive cloth simulation首次成为可能，Huamin Wang和Chenfanfu Jiang两位cloth simulation权威的合作成果。

---

### 6.2 GroomCap: High-Fidelity Prior-Free Hair Capture

- **作者**：Yuxiao Zhou, Menglei Chai, Daoye Wang, Sebastian Winberg, Erroll Wood, Kripasindhu Sarkar, Markus Gross, Thabo Beeler（ETH Zurich）
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687768](https://doi.org/10.1145/3687768)

**问题定义**：Strand-level精度的multi-view hair reconstruction仍具挑战，现有pipeline有固有局限（orientation blending导致structural information loss）。

**方法与架构**：
- **Neural implicit hair volume**：encode high-res 3D orientation + occupancy from input views
- **Volumetric 3D orientation rendering algorithm** + **2D orientation distribution supervision**（prevent undesired orientation blending）
- **Gaussian-based hair optimization**：chained Gaussian representation + direct photometric supervision
- **Prior-free**（不依赖external data priors）

**信息缺口**：Neural implicit network架构（MLP层数/width）、tested subjects数量和类型、strand-level quantitative metrics（orientation error/root mean square deviation等）。

**为什么重要**：ETH Zurich的Thabo Beeler组在hair capture领域的最新成果，prior-free设计提高了泛化能力，volumetric 3D orientation rendering是核心创新。

---

### 6.3 GarVerseLOD: High-Fidelity 3D Garment Reconstruction from Single Image

- **作者**：Zhongjin Luo, Haolin Liu, Chenghong Li, Wanghao Du, Zirong Jin, Wanhu Sun, Yinyu Nie, Weikai Chen, Xiaoguang Han
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687921](https://doi.org/10.1145/3687921)

**问题定义**：Single in-the-wild image的3D garment reconstruction在complex cloth deformation和body poses下泛化困难。

**方法与架构**：
- **GarVerseLOD dataset**：**6,000**高质量cloth models，professional artists手工创建fine-grained geometry
- **Hierarchical LOD dataset**：
  - Level 1: detail-free stylized shape
  - Level 2: pose-blended garment
  - Level 3: pixel-aligned details
- Factorized inference：将under-constrained问题分解为更小search space的子任务
- **Conditional diffusion models**生成extensive paired images（high photorealism）用于in-the-wild generalization

**信息缺口**：LOD各级的具体vertex count、diffusion-based labeling paradigm的细节、in-the-wild测试集的规模和指标（CD/FID等）。

**为什么重要**：6,000高质量garment models的dataset规模前所未有，LOD层次化设计为解决under-constrained重建问题提供了新思路。

---

## 七、Crowd & Collective Motion

### 7.1 CBIL: Collective Behavior Imitation Learning for Fish from Real Videos

- **作者**：Yifan Wu, Zhiyang Dou, Yuko Ishiwaka, Shun Ogawa, Yuke Lou, Wenping Wang, Lingjie Liu, Taku Komura
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687904](https://doi.org/10.1145/3687904)

**问题定义**：现有imitation learning方法需要ground-truth motion trajectories，但在高密度群体erratic movements下难以获取；rule-based方法motion diversity有限。

**方法与架构**：
- **直接从real videos学习fish schooling behavior**，无需captured motion trajectories
- **MVAE (Masked Video AutoEncoder)**：self-supervised提取implicit states from 2D observations
  - Maps 2D observations to compact and expressive implicit states
- **Adversarial imitation learning**：latent space中imitate motion pattern distribution
- Bio-inspired rewards + priors正则化训练
- 跨species泛化 + abnormal fish behavior detection应用

**信息缺口**：MVAE架构细节、adversarial training的具体设置（discriminator architecture）、tested species数量和类型、abnormal behavior detection的accuracy。

**为什么重要**：首次实现了从raw videos直接学习collective behavior，绕过了trajectory annotation瓶颈。对生物学研究和animation都有价值。

---

## 八、Animation Tools, Authoring & Inbetweening

### 8.1 Skeleton-Driven Inbetweening of Bitmap Character Drawings

- **作者**：Kirill Brodt, Mikhail Bessmeltsev
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687955](https://doi.org/10.1145/3687955)

**问题定义**：工业界keyframes常间距大（far inbetweening）、raster format、含occlusions，现有solution多要求vector input或仅支持tight inbetweening。

**方法与架构**：
- **Skeleton-driven bitmap inbetweening**：
  1. Artist animates skeleton between keyframe poses
  2. System performs skeleton-based deformation of bitmap drawings
  3. Discrete optimization + deep learning blend deformed images
- **2.5D partially layered template**：lifting drawing into 3D解决occlusions导致的piecewise smooth deformation问题
- 支持tight和far inbetweening
- Very little annotation required（仅skeleton + two bitmap keyframes）

**定量结果**：Qualitative和quantitative comparisons + user studies demonstrate consistently outperforms state of the art，results consistent with viewers' perception。

**信息缺口**：Discrete optimization的目标函数、deep learning blend的网络架构、user study的具体设计（participants/tasks/metrics）。

**为什么重要**：首次实现了bitmap far inbetweening with occlusion handling，贴近工业实际需求。Mikhail Bessmeltsev组在2D animation工具领域的持续贡献。

---

### 8.2 ToonCrafter: Generative Cartoon Interpolation

- **作者**：Jinbo Xing, Hanyuan Liu, Menghan Xia, Yong Zhang, Xintao Wang, Ying Shan, Tien-Tsin Wong
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687761](https://doi.org/10.1145/3687761)
- **项目页**：https://doubiiu.github.io/projects/ToonCrafter

**问题定义**：Cartoon video interpolation面临exaggerated non-linear motion和occlusion/dis-occlusion挑战，传统correspondence-based方法失效。

**方法与架构**：
- **Generative cartoon interpolation**（超越traditional correspondence-based方法）
- **Toon rectification learning**：seamlessly adapt live-action video priors to cartoon domain，resolve domain gap和content leakage
- **Dual-reference-based 3D decoder**：compensate lost details due to highly compressed latent prior spaces
- **Flexible sketch encoder**：empowers users with interactive control over interpolation results

**定量结果**：Comparative evaluation demonstrates notable superiority over existing competitors，effectively handles dis-occlusion。Code和model weights开源。

**信息缺口**：Base live-action model的选择（Stable Video Diffusion?）、toon rectification的具体训练策略、FID/LPIPS/用户偏好测试的具体数值。

**为什么重要**：首次将live-action video diffusion prior成功adapt到cartoon interpolation，解决了dis-occlusion这一长期难题。开源代码和模型权重促进了社区跟进。

---

## 九、Motion Perception

### 9.1 Evaluating Visual Perception of Object Motion in Dynamic Environments

- **作者**：Budmonde Duinkharjav, Jenna Kang, Gavin Stuart Peter Miller, Chang Xiao, Qi Sun
- **Track**：Asia Journal (TOG 43·6)｜**DOI**：[10.1145/3687912](https://doi.org/10.1145/3687912)

**问题定义**：现有visual perception研究多在stationary settings和singular objects下进行，但实际应用中observer也在complex scenes中移动，combined optical flow导致perceptual ambiguities。

**方法与架构**：
- **Crowdsourcing-based psychophysical study**：quantify scene dynamics/content patterns与perceptual judgments的关系
- 构建generalized conditions的perceptual model
- 应用于gaming和animation design中的motion error补偿

**信息缺口**：Psychophysical study的具体设计（trial数量/participants/stimuli类型）、perceptual model的数学形式、application demo的具体效果量化。

**为什么重要**：首次系统研究了dynamic 3D environments中的object motion perception，为perception-aware rendering提供了理论基础。

---

## 十、Hair Simulation（补充）

### 10.1 Contact Detection between Curved Fibres: High Order Makes a Difference

*注：此论文实际发表于TOG 43(4)即SIGGRAPH 2024 Journal Track，但因主题相关性在此简要提及。详见SG2024综述Section 6.6。*

---

## 统计汇总

- SIGGRAPH Asia 2024共收录 **23** 篇 motion 相关论文
- 全部为Journal Track（TOG 43·6）

### 按类别分布
- Motion Generation & Control：3 篇
- Facial & Head Animation：2 篇
- Motion Capture & Performance Capture：7 篇
- Avatar & Human Reconstruction：1 篇
- Retargeting, Deformation & Skinning：4 篇
- Cloth, Hair & Garment Simulation：3 篇
- Crowd & Collective Motion：1 篇
- Animation Tools & Authoring：2 篇
- Motion Perception：1 篇

### 亮点趋势
- **Gaussian-based表示**在performance capture领域集中爆发（GaussianHeads、Gaussian Surfel Splatting、DualGS、Real-time Large-scale Deformation of GS）
- **Unified control frameworks**成为motion generation主流范式（MaskedMimic的motion inpainting、CPoser的LLM+parsing）
- **大规模dataset**成为garment/hair reconstruction的关键推动力（GarVerseLOD 6000 models）
- **Self-supervised learning from videos**开始进入collective behavior领域（CBIL）
