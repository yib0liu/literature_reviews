# SIGGRAPH Asia 2024 — Motion 相关论文逐篇总结

> 自动生成时间：2026-08-11 14:08  

> 涵盖方向：motion generation / motion control / mocap / animation / retargeting / gesture / crowd / cloth / hair / hand-object 等  

> 共收录 **23** 篇 motion 相关论文（含 Journal Track + Conference Track）


---


## Motion Generation & Control

### MaskedMimic: Unified Physics-Based Character Control Through Masked Motion Inpainting

**作者**: Chen Tessler, Yunrong Guo, Ofir Nabati, Gal Chechik, Xue Bin Peng  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687951](https://doi.org/10.1145/3687951)  

**关键词**: motion_generation, physics_based, motion_inpainting, character_control


**摘要**: Crafting a single, versatile physics-based controller that can breathe life into interactive characters across a wide spectrum of scenarios represents an exciting frontier in character animation. An ideal controller should support diverse control modalities, such as sparse target keyframes, text instructions, and scene information. While previous works have proposed physically simulated, scene-aware control models, these systems have predominantly focused on developing controllers that each specializes in a narrow set of tasks and control modalities. This work presents MaskedMimic, a novel approach that formulates physics-based character control as a general motion inpainting problem. Our key insight is to train a single unified model to synthesize motions from partial (masked) motion desc [...]


---

### CPoser: An Optimization-after-Parsing Approach for Text-to-Pose Generation Using Large Language Models

**作者**: Yumeng Li, Bohong Chen, Zhong Ren, Yao-Xiang Ding, Libin Liu, Tianjia Shao, Kun Zhou  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687932](https://doi.org/10.1145/3687932)  

**关键词**: pose_generation, text_to_pose, llm, optimization


**摘要**: Text-to-pose generation is challenging due to the complexity of natural language and human posture semantics. Utilizing large language models (LLMs) for text-to-pose generation is appealing due to their strong capabilities in text understanding and reasoning. However, as LLMs are designed for general-purpose language processing and not specifically trained for pose generation, it remains nontrivial to generate precise articulation targets for the full body using LLMs directly. To this end, we propose CPoser, a novel approach to harness the power of LLMs for text-to-pose generation, featuring a prompt parsing stage and a pose optimization stage. The parsing stage utilizes LLMs to turn text prompts into pose intermediate representations (Pose-IRs) through a set of predefined structured queri [...]


---

### 360-degree Human Video Generation with 4D Diffusion Transformer

**作者**: Ruizhi Shao, Youxin Pang, Zerong Zheng, Jingxiang Sun, Yebin Liu  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687980](https://doi.org/10.1145/3687980)  

**关键词**: human_video_generation, 4d_diffusion, novel_view, synthesis


**摘要**: We present a novel approach for generating 360-degree high-quality, spatiotemporally coherent human videos from a single image. Our framework combines the strengths of diffusion transformers for capturing global correlations across viewpoints and time, and CNNs for accurate condition injection. The core is a hierarchical 4D transformer architecture that factorizes self-attention across views, time steps, and spatial dimensions, enabling efficient modeling of the 4D space. Precise conditioning is achieved by injecting human identity, camera parameters, and temporal signals into the respective transformers. To train this model, we collect a multi-dimensional dataset spanning images, videos, multi-view data, and limited 4D footage, along with a tailored multi-dimensional training strategy. Ou [...]


---


## Gesture, Facial & Head Animation

### GaussianHeads: End-to-End Learning of Drivable Gaussian Head Avatars from Coarse-to-fine Representations

**作者**: Kartik Teotia, Hyeongwoo Kim, Pablo Garrido, Marc Habermann, Mohamed Elgharib, Christian Theobalt  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687927](https://doi.org/10.1145/3687927)  

**关键词**: head_avatar, gaussian, drivable, end_to_end


**摘要**: Real-time rendering of human head avatars is a cornerstone of many computer graphics applications, such as augmented reality, video games, and films, to name a few. Recent approaches address this challenge with computationally efficient geometry primitives in a carefully calibrated multi-view setup. Albeit producing photorealistic head renderings, they often fail to represent complex motion changes, such as the mouth interior and strongly varying head poses. We propose a new method to generate highly dynamic and deformable human head avatars from multi-view imagery in real time. At the core of our method is a hierarchical representation of head models that can capture the complex dynamics of facial expressions and head movements. First, with rich facial features extracted from raw input fr [...]


---

### VOODOO XP: Expressive One-Shot Head Reenactment for VR Telepresence

**作者**: Phong Tran, Egor Zakharov, Long-Nhat Ho, Liwen Hu, Adilbek Karmanov, Aviral Agarwal, McLean Goldwhite, Ariana Bermudez Venegas, Anh Tuan Tran, Hao Li  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687928](https://doi.org/10.1145/3687928)  

**关键词**: head_reenactment, one_shot, vr_telepresence, expressive


**摘要**: Distortion-minimizing surface parameterization is an essential step for computing 2D pieces necessary to fabricate a target 3D shape from flat material. Garment design and textile fabrication are a prominent application example. Common distortion measures quantify length, angle or area preservation in an isotropic manner, so that when applied to woven textile fabrication, they implicitly assume fabric behaves like paper, which is inextensible in all directions and does not permit shearing. However, woven fabric differs significantly from paper: it exhibits anisotropy along the yarn directions and allows for some degree of shearing. We propose a novel distortion energy based on Chebyshev nets that anisotropically penalizes shearing and stretching. Our energy formulation can be used as an op [...]


---


## Motion Capture & Performance Capture

### ELMO: Enhanced Real-time LiDAR Motion Capture through Upsampling

**作者**: Deok-Kyeong Jang, Dongseok Yang, Deok-Yun Jang, Byeoli Choi, Sung-Hee Lee, Donghoon Shin  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687991](https://doi.org/10.1145/3687991)  

**关键词**: mocap, lidar, real_time, upsampling


**摘要**: This paper introduces ELMO, a real-time upsampling motion capture framework designed for a single LiDAR sensor. Modeled as a conditional autoregressive transformer-based upsampling motion generator, ELMO achieves 60 fps motion capture from a 20 fps LiDAR point cloud sequence. The key feature of ELMO is the coupling of the self-attention mechanism with thoughtfully designed embedding modules for motion and point clouds, significantly elevating the motion quality. To facilitate accurate motion capture, we develop a one-time skeleton calibration model capable of predicting user skeleton off-sets from a single-frame point cloud. Additionally, we introduce a novel data augmentation technique utilizing a LiDAR simulator, which enhances global root tracking to improve environmental understanding. [...]


---

### EgoHDM: A Real-time Egocentric-Inertial Human Motion Capture, Localization, and Dense Mapping System

**作者**: Handi Yin, Bonan Liu, Manuel Kaufmann, Jinhao He, Sammy Christen, Jie Song, Pan Hui  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687907](https://doi.org/10.1145/3687907)  

**关键词**: mocap, egocentric, inertial, slam, localization


**摘要**: We present EgoHDM, an online egocentric-inertial human motion capture (mocap), localization, and dense mapping system. Our system uses 6 inertial measurement units (IMUs) and a commodity head-mounted RGB camera. EgoHDM is the first human mocap system that offers
            dense
            scene mapping in
            near real-time.
            Further, it is fast and robust to initialize and fully closes the loop between physically plausible map-aware global human motion estimation and mocap-aware 3D scene reconstruction. To achieve this, we design a tightly coupled mocap-aware dense bundle adjustment and physics-based body pose correction module leveraging a local body-centric elevation map. The latter introduces a novel terrain-aware contact PD controller, which enables characters to [...]


---

### Look Ma, no markers: holistic performance capture without the hassle

**作者**: Charlie Hewitt, Fatemeh Saleh, Sadegh Aliakbarian, Lohit Petikam, Shideh Rezaeifar, Louis Florentin, Zafiirah Hosenie, Thomas J. Cashman, Julien Valentin, Darren Cosker, Tadas Baltrusaitis  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687772](https://doi.org/10.1145/3687772)  

**关键词**: performance_capture, markerless, holistic, video_based


**摘要**: We tackle the problem of highly-accurate, holistic performance capture for the face, body and hands simultaneously. Motion-capture technologies used in film and game production typically focus only on face, body or hand capture independently, involve complex and expensive hardware and a high degree of manual intervention from skilled operators. While machine-learning-based approaches exist to overcome these problems, they usually only support a single camera, often operate on a single part of the body, do not produce precise world-space results, and rarely generalize outside specific contexts. In this work, we introduce the first technique for markerfree, high-quality reconstruction of the complete human body, including eyes and tongue, without requiring any calibration, manual interventio [...]


---

### Gaussian Surfel Splatting for Live Human Performance Capture

**作者**: Zheng Dong, Ke Xu, Yaoan Gao, Hujun Bao, Weiwei Xu, Rynson W. H. Lau  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687993](https://doi.org/10.1145/3687993)  

**关键词**: performance_capture, gaussian_splatting, live, real_time


**摘要**: High-quality real-time rendering using user-affordable capture rigs is an essential property of human performance capture systems for real-world applications. However, state-of-the-art performance capture methods may not yield satisfactory rendering results under a very sparse (e.g., four) capture setting. Specifically, neural radiance field (NeRF)-based methods and 3D Gaussian Splatting (3DGS)-based methods tend to produce local geometry errors for unseen performers, while occupancy field (PIFu)-based methods often produce unrealistic rendering results. In this paper, we propose a novel generalizable neural approach to reconstruct and render the performers from very sparse RGBD streams in high quality. The core of our method is a novel point-based generalizable human (PGH) representation  [...]


---

### Robust Dual Gaussian Splatting for Immersive Human-centric Volumetric Videos

**作者**: Yuheng Jiang, Zhehao Shen, Yu Hong, Chengcheng Guo, Yize Wu, Yingliang Zhang, Jingyi Yu, Lan Xu  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687926](https://doi.org/10.1145/3687926)  

**关键词**: volumetric_video, human_capture, gaussian_splatting, immersive


**摘要**: Volumetric video represents a transformative advancement in visual media, enabling users to freely navigate immersive virtual experiences and narrowing the gap between digital and real worlds. However, the need for extensive manual intervention to stabilize mesh sequences and the generation of excessively large assets in existing workflows impedes broader adoption. In this paper, we present a novel Gaussian-based approach, dubbed
            DualGS
            , for real-time and high-fidelity playback of complex human performance with excellent compression ratios. Our key idea in DualGS is to separately represent motion and appearance using the corresponding skin and joint Gaussians. Such an explicit disentanglement can significantly reduce motion redundancy and enhance temporal coherence [...]


---

### Real-time Large-scale Deformation of Gaussian Splatting

**作者**: Lin Gao, Jie Yang, Bo-Tao Zhang, Jia-Mu Sun, Yu-Jie Yuan, Hongbo Fu, Yu-Kun Lai  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687756](https://doi.org/10.1145/3687756)  

**关键词**: deformation, gaussian_splatting, real_time, large_scale


**摘要**: Neural implicit representations, including Neural Distance Fields and Neural Radiance Fields, have demonstrated significant capabilities for reconstructing surfaces with complicated geometry and topology, and generating novel views of a scene. Nevertheless, it is challenging for users to directly deform or manipulate these implicit representations with large deformations in a real-time fashion. Gaussian Splatting (GS) has recently become a promising method with explicit geometry for representing static scenes and facilitating high-quality and real-time synthesis of novel views. However, it cannot be easily deformed due to the use of discrete Gaussians and the lack of explicit topology. To address this, we develop a novel GS-based method (GaussianMesh) that enables interactive deformation.  [...]


---


## Avatar & Human Reconstruction

### PuzzleAvatar: Assembling 3D Avatars from Personal Albums

**作者**: Yuliang Xiu, Yufei Ye, Zhen Liu, Dimitris Tzionas, Michael J. Black  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687771](https://doi.org/10.1145/3687771)  

**关键词**: avatar, 3d_reconstruction, personal_photos, assembly


**摘要**: Generating
            personalized
            3D avatars is crucial for AR/VR. However, recent text-to-3D methods that generate avatars for celebrities or fictional characters, struggle with everyday people. Methods for faithful reconstruction typically require full-body images in controlled settings. What if users could just upload their personal "OOTD" (Outfit Of The Day) photo collection and get a faithful avatar in return? The challenge is that such casual photo collections contain diverse poses, challenging viewpoints, cropped views, and occlusion (albeit with a consistent outfit, accessories and hairstyle). We address this novel "
            Album2Human
            " task by developing
            PuzzleAvatar
            , a novel model that generates a faithful 3D avatar (in a c [...]


---


## Retargeting, Deformation & Skinning

### Multi-Resolution Real-Time Deep Pose-Space Deformation

**作者**: Mianlun Zheng, Jernej Barbic  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687985](https://doi.org/10.1145/3687985)  

**关键词**: pose_space_deformation, real_time, deep_learning, skin


**摘要**: We present a hard-real-time multi-resolution mesh shape deformation technique for skeleton-driven soft-body characters. Producing mesh deformations at multiple levels of detail is very important in many applications in computer graphics. Our work targets applications where the multi-resolution shapes must be generated at fast speeds ("hard-real-time", e.g., a few milliseconds at most and preferably under 1 millisecond), as commonly needed in computer games, virtual reality and Metaverse applications. We assume that the character mesh is driven by a skeleton, and that high-quality character shapes are available in a set of training poses originating from a high-quality (slow) rig such as volumetric FEM simulation. Our method combines multi-resolution analysis, mesh partition of unity, and n [...]


---

### Deformation Recovery: Localized Learning for Detail-Preserving Deformations

**作者**: Ramana Sundararaman, Nicolas Donati, Simone Melzi, Etienne Corman, Maks Ovsjanikov  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687968](https://doi.org/10.1145/3687968)  

**关键词**: deformation, shape_correspondence, learning, detail_preserving


**摘要**: We introduce a novel data-driven approach aimed at designing high-quality shape deformations based on a coarse localized input signal. Unlike previous data-driven methods that require a global shape encoding, we observe that detail-preserving deformations can be estimated reliably without any global context in certain scenarios. Building on this intuition, we leverage Jacobians defined in a one-ring neighborhood as a coarse representation of the deformation. Using this as the input to our neural network, we apply a series of MLPs combined with feature smoothing to learn the Jacobian corresponding to the detail-preserving deformation, from which the embedding is recovered by the standard Poisson solve. Crucially, by removing the dependence on a global encoding, every
            point
      [...]


---


## Cloth, Hair & Garment Simulation

### Efficient GPU Cloth Simulation with Non-distance Barriers and Subspace Reuse

**作者**: Lei Lan, Zixuan Lu, Jingyi Long, Chun Yuan, Xuan Li, Xiaowei He, Huamin Wang, Chenfanfu Jiang, Yin Yang  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687760](https://doi.org/10.1145/3687760)  

**关键词**: cloth_simulation, gpu, barrier_method, subspace


**摘要**: This paper pushes the performance of cloth simulation, making the simulation interactive even for high-resolution garment models while keeping every triangle untangled. The penetration-free guarantee is inspired by the interior point method, which converts the inequality constraints to barrier potentials. We propose a major overhaul of this modality within the projective dynamics framework by leveraging an adaptive weighting mechanism inspired by barrier formulation. This approach does not depend on the distance between mesh primitives, but on the virtual life span of a collision event and thus keeps all the vertices within feasible region. Such a non-distance barrier model allows a new way to integrate collision resolution into the simulation pipeline. Another contributor to the performan [...]


---

### GroomCap: High-Fidelity Prior-Free Hair Capture

**作者**: Yuxiao Zhou, Menglei Chai, Daoye Wang, Sebastian Winberg, Erroll Wood, Kripasindhu Sarkar, Markus Gross, Thabo Beeler  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687768](https://doi.org/10.1145/3687768)  

**关键词**: hair_capture, groom, reconstruction, prior_free


**摘要**: Despite recent advances in multi-view hair reconstruction, achieving strand-level precision remains a significant challenge due to inherent limitations in existing capture pipelines. We introduce
            GroomCap
            , a novel multi-view hair capture method that reconstructs faithful and high-fidelity hair geometry without relying on external data priors. To address the limitations of conventional reconstruction algorithms, we propose a neural implicit representation for hair volume that encodes high-resolution 3D orientation and occupancy from input views. This implicit hair volume is trained with a new volumetric 3D orientation rendering algorithm, coupled with 2D orientation distribution supervision, to effectively prevent the loss of structural information caused by undesir [...]


---

### GarVerseLOD: High-Fidelity 3D Garment Reconstruction from a Single In-the-Wild Image using a Dataset with Levels of Details

**作者**: Zhongjin Luo, Haolin Liu, Chenghong Li, Wanghao Du, Zirong Jin, Wanhu Sun, Yinyu Nie, Weikai Chen, Xiaoguang Han  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687921](https://doi.org/10.1145/3687921)  

**关键词**: garment_reconstruction, 3d, single_image, lod


**摘要**: Neural implicit functions have brought impressive advances to the state-of-the-art of clothed human digitization from multiple or even single images. However, despite the progress, current arts still have difficulty generalizing to unseen images with complex cloth deformation and body poses. In this work, we present GarVerseLOD, a new dataset and framework that paves the way to achieving unprecedented robustness in high-fidelity 3D garment reconstruction from a single unconstrained image. Inspired by the recent success of large generative models, we believe that one key to addressing the generalization challenge lies in the quantity and quality of 3D garment data. Towards this end, GarVerseLOD collects 6,000 high-quality cloth models with fine-grained geometry details manually created by p [...]


---


## Crowd & Collective Motion

### CBIL: Collective Behavior Imitation Learning for Fish from Real Videos

**作者**: Yifan Wu, Zhiyang Dou, Yuko Ishiwaka, Shun Ogawa, Yuke Lou, Wenping Wang, Lingjie Liu, Taku Komura  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687904](https://doi.org/10.1145/3687904)  

**关键词**: crowd_simulation, collective_behavior, imitation_learning, fish


**摘要**: Reproducing realistic collective behaviors presents a captivating yet formidable challenge. Traditional rule-based methods rely on hand-crafted principles, limiting motion diversity and realism in generated collective behaviors. Recent imitation learning methods learn from data but often require ground-truth motion trajectories and struggle with authenticity, especially in high-density groups with erratic movements. In this paper, we present a scalable approach, Collective Behavior Imitation Learning (CBIL), for learning fish schooling behavior
            directly from videos
            , without relying on captured motion trajectories. Our method first leverages Video Representation Learning, in which a Masked Video AutoEncoder (MVAE) extracts implicit states from video inputs in a self [...]


---


## Animation Tools & Authoring

### SKEL-Betweener: a Neural Motion Rig for Interactive Motion Authoring

**作者**: Dhruv Agrawal, Jakob Buhmann, Dominik Borer, Robert W. Sumner, Martin Guay  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687941](https://doi.org/10.1145/3687941)  

**关键词**: motion_authoring, inbetweening, neural_rig, interactive


**摘要**: Authoring 3D motions is a laborious process that requires manipulating and coordinating many control handles over time. Neural motion representations learned from large motion datasets have recently shown impressive capabilities in many motion completion tasks. However, current methods are not designed for interactive motion authoring workflows. The reasons being their requirement of a dense context of full poses, which takes considerable time to author, as well as their lack of joint-level controls for refinement. In this paper, we introduce a
            Neural Motion Rig
            called SKEL-Betweener, tailored to interactive motion authoring. SKEL-Betweener is able to generate long motion sequences from two poses only, and enables intermediate motion authoring via
            neural [...]


---

### Skeleton-Driven Inbetweening of Bitmap Character Drawings

**作者**: Kirill Brodt, Mikhail Bessmeltsev  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687955](https://doi.org/10.1145/3687955)  

**关键词**: inbetweening, skeleton_driven, 2d_animation, bitmap


**摘要**: One of the primary reasons for the high cost of traditional animation is the inbetweening process, where artists manually draw each intermediate frame necessary for smooth motion. Making this process more efficient has been at the core of computer graphics research for years, yet the industry has adopted very few solutions. Most existing solutions either require vector input or resort to tight inbetweening; often, they attempt to fully automate the process. In industry, however, keyframes are often spaced far apart, drawn in raster format, and contain occlusions. Moreover, inbetweening is fundamentally an artistic process, so the artist should maintain high-level control over it.
          
            We address these issues by proposing a novel inbetweening system for bitmap character dr [...]


---

### ToonCrafter: Generative Cartoon Interpolation

**作者**: Jinbo Xing, Hanyuan Liu, Menghan Xia, Yong Zhang, Xintao Wang, Ying Shan, Tien-Tsin Wong  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687761](https://doi.org/10.1145/3687761)  

**关键词**: cartoon_interpolation, generative, video, animation


**摘要**: We introduce ToonCrafter, a novel approach that transcends traditional correspondence-based cartoon video interpolation, paving the way for generative interpolation. Traditional methods, that implicitly assume linear motion and the absence of complicated phenomena like dis-occlusion, often struggle with the exaggerated non-linear and large motions with occlusion commonly found in cartoons, resulting in implausible or even failed interpolation results. To overcome these limitations, we explore the potential of adapting live-action video priors to better suit cartoon interpolation within a generative framework. ToonCrafter effectively addresses the challenges faced when applying live-action video motion priors to generative cartoon interpolation. First, we design a toon rectification learnin [...]


---


## Motion Perception

### Evaluating Visual Perception of Object Motion in Dynamic Environments

**作者**: Budmonde Duinkharjav, Jenna Kang, Gavin Stuart Peter Miller, Chang Xiao, Qi Sun  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687912](https://doi.org/10.1145/3687912)  

**关键词**: motion_perception, visual_evaluation, dynamic_environments


**摘要**: Precisely understanding how objects move in 3D is essential for broad scenarios such as video editing, gaming, driving, and athletics. With screen-displayed computer graphics content, users only perceive limited cues to judge the object motion from the on-screen optical flow. Conventionally, visual perception is studied with stationary settings and singular objects. However, in practical applications, we---the observer---also move within complex scenes. Therefore, we must extract object motion from a combined optical flow displayed on screen, which can often lead to mis-estimations due to perceptual ambiguities.
          We measure and model observers' perceptual accuracy of object motions in dynamic 3D environments, a universal but under-investigated scenario in computer graphics applica [...]


---


## Other Motion Topics

### Geometry-Aware Retargeting for Two-Skinned Characters Interaction

**作者**: Inseo Jang, Soojin Choi, Seokhyeon Hong, Chaelin Kim, Junyong Noh  

**Track**: SIGGRAPH Asia 2024 Journal Track (TOG 43·6) | **DOI**: [10.1145/3687962](https://doi.org/10.1145/3687962)  

**关键词**: motion_retargting, two_skinned, interaction, geometry_aware


**摘要**: Interactive motion between multiple characters is widely utilized in games and movies. However, the method for generating interactive motions considering the character's diverse mesh shape has yet to be studied. We propose a Spatio Cooperative Transformer (SCT) to retarget the interacting motions of two characters having arbitrary mesh connectivity. SCT predicts the residual of root position and joint rotations considering the shape difference between the source and target of interacting characters. In addition, we introduce an anchor loss function for SCT to maintain the geometric distance between the interacting characters when they are retargeted. We also propose a motion augmentation method with deformation-based adaptation to prepare a source-target paired dataset with an identical me [...]


---



## 统计汇总

- 本会议共收录 **23** 篇 motion 相关论文

- Journal Track（TOG）：23 篇  

- Conference Track：0 篇  


### 按类别分布

- Motion Generation & Control：3 篇
- Gesture, Facial & Head Animation：2 篇
- Motion Capture & Performance Capture：6 篇
- Avatar & Human Reconstruction：1 篇
- Retargeting, Deformation & Skinning：2 篇
- Cloth, Hair & Garment Simulation：3 篇
- Crowd & Collective Motion：1 篇
- Animation Tools & Authoring：3 篇
- Motion Perception：1 篇

