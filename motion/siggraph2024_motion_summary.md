# SIGGRAPH 2024 — Motion 相关论文逐篇总结

> 自动生成时间：2026-08-11 14:08  

> 涵盖方向：motion generation / motion control / mocap / animation / retargeting / gesture / crowd / cloth / hair / hand-object 等  

> 共收录 **26** 篇 motion 相关论文（含 Journal Track + Conference Track）


---


## Motion Generation & Control

### Interactive Character Control with Auto-Regressive Motion Diffusion Models

**作者**: Yi Shi, Jingbo Wang, Xuekun Jiang, Bingkun Lin, Bo Dai, Xue Bin Peng  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658140](https://doi.org/10.1145/3658140)  

**关键词**: motion_generation, diffusion, interactive_control, character


**摘要**: Real-time character control is an essential component for interactive experiences, with a broad range of applications, including physics simulations, video games, and virtual reality. The success of diffusion models for image synthesis has led to the use of these models for motion synthesis. However, the majority of these motion diffusion models are primarily designed for offline applications, where space-time models are used to synthesize an entire sequence of frames simultaneously with a pre-specified length. To enable real-time motion synthesis with diffusion model that allows time-varying controls, we propose A-MDM (Auto-regressive Motion Diffusion Model). Our conditional diffusion model takes an initial pose as input, and auto-regressively generates successive motion frames conditione [...]


---

### MoConVQ: Unified Physics-Based Motion Control via Scalable Discrete Representations

**作者**: Heyuan Yao, Zhenhua Song, Yuyang Zhou, Tenglong Ao, Baoquan Chen, Libin Liu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658137](https://doi.org/10.1145/3658137)  

**关键词**: motion_control, physics_based, vq_representation, character


**摘要**: In this work, we present MoConVQ, a novel unified framework for physics-based motion control leveraging scalable discrete representations. Building upon vector quantized variational autoencoders (VQ-VAE) and model-based reinforcement learning, our approach effectively learns motion embeddings from a large, unstructured dataset spanning tens of hours of motion examples. The resultant motion representation not only captures diverse motion skills but also offers a robust and intuitive interface for various applications. We demonstrate the versatility of MoConVQ through several applications: universal tracking control from various motion sources, interactive character control with latent motion representations using supervised learning, physics-based motion generation from natural language des [...]


---

### Categorical Codebook Matching for Embodied Character Controllers

**作者**: Sebastian Starke, Paul Starke, Nicky He, Taku Komura, Yuting Ye  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658209](https://doi.org/10.1145/3658209)  

**关键词**: motion_control, codebook, character_controller, embodied


**摘要**: Translating motions from a real user onto a virtual embodied avatar is a key challenge for character animation in the metaverse. In this work, we present a novel generative framework that enables mapping from a set of sparse sensor signals to a full body avatar motion in real-time while faithfully preserving the motion context of the user. In contrast to existing techniques that require training a motion prior and its mapping from control to motion separately, our framework is able to learn the motion manifold as well as how to sample from it at the same time in an end-to-end manner. To achieve that, we introduce a technique called codebook matching which matches the probability distribution between two categorical codebooks for the inputs and outputs for synthesizing the character motions [...]


---

### DiffPoseTalk: Speech-Driven Stylistic 3D Facial Animation and Head Pose Generation via Diffusion Models

**作者**: Zhiyao Sun, Tian Lv, Sheng Ye, Matthieu Lin, Jenny Sheng, Yu-Hui Wen, Minjing Yu, Yong-Jin Liu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658221](https://doi.org/10.1145/3658221)  

**关键词**: facial_animation, head_pose, diffusion, speech_driven


**摘要**: The generation of stylistic 3D facial animations driven by speech presents a significant challenge as it requires learning a many-to-many mapping between speech, style, and the corresponding natural facial motion. However, existing methods either employ a deterministic model for speech-to-motion mapping or encode the style using a one-hot encoding scheme. Notably, the one-hot encoding approach fails to capture the complexity of the style and thus limits generalization ability. In this paper, we propose DiffPoseTalk, a generative framework based on the diffusion model combined with a style encoder that extracts style embeddings from short reference videos. During inference, we employ classifier-free guidance to guide the generation process based on the speech and style. In particular, our s [...]


---


## Gesture, Facial & Head Animation

### Semantic Gesticulator: Semantics-Aware Co-Speech Gesture Synthesis

**作者**: Zeyi Zhang, Tenglong Ao, Yuyao Zhang, Qingzhe Gao, Chuan Lin, Baoquan Chen, Libin Liu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658134](https://doi.org/10.1145/3658134)  

**关键词**: gesture_synthesis, co_speech, semantics, awareness


**摘要**: In this work, we present
            Semantic Gesticulator
            , a novel framework designed to synthesize realistic gestures accompanying speech with strong semantic correspondence. Semantically meaningful gestures are crucial for effective non-verbal communication, but such gestures often fall within the long tail of the distribution of natural human motion. The sparsity of these movements makes it challenging for deep learning-based systems, trained on moderately sized datasets, to capture the relationship between the movements and the corresponding speech semantics. To address this challenge, we develop a generative retrieval framework based on a large language model. This framework efficiently retrieves suitable semantic gesture candidates from a motion library in response to t [...]


---

### S3: Speech, Script and Scene driven Head and Eye Animation

**作者**: Yifang Pan, Rishabh Agrawal, Karan Singh  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658172](https://doi.org/10.1145/3658172)  

**关键词**: head_animation, eye_animation, speech_driven, scene_driven


**摘要**: We present
            S
            3
            , a novel approach to generating expressive, animator-centric 3D head and eye animation of characters in conversation. Given
            speech
            audio, a Directorial
            script
            and a cinematographic 3D
            scene
            as input, we automatically output the animated 3D rotation of each character's head and eyes.
            S
            3
            distills animation and psycho-linguistic insights into a novel modular framework for conversational gaze capturing: audio-driven rhythmic head motion; narrative script-driven emblematic head and eye gestures; and gaze trajectories computed from audio-driven gaze focus/aversion and 3D visual scene salience. Our evaluation is four-fold: we quantitative [...]


---

### Learning a Generalized Physical Face Model From Data

**作者**: Lingchen Yang, Gaspard Zoss, Prashanth Chandran, Markus Gross, Barbara Solenthaler, Eftychios Sifakis, Derek Bradley  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658189](https://doi.org/10.1145/3658189)  

**关键词**: face_model, physical_simulation, learning, data_driven


**摘要**: Physically-based simulation is a powerful approach for 3D facial animation as the resulting deformations are governed by physical constraints, allowing to easily resolve self-collisions, respond to external forces and perform realistic anatomy edits. Today's methods are data-driven, where the actuations for finite elements are inferred from captured skin geometry. Unfortunately, these approaches have not been widely adopted due to the complexity of initializing the material space and learning the deformation model for each character separately, which often requires a skilled artist followed by lengthy network training. In this work, we aim to make physics-based facial animation more accessible by proposing a generalized physical face model that we learn from a large 3D face dataset. Once t [...]


---

### Universal Facial Encoding of Codec Avatars from VR Headsets

**作者**: Shaojie Bai, Te-Li Wang, Chenghui Li, Akshay Venkatesh, Tomas Simon, Chen Cao, Gabriel Schwartz, Jason Saragih, Yaser Sheikh, Shih-En Wei  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658234](https://doi.org/10.1145/3658234)  

**关键词**: facial_capture, codec_avatar, vr_headset, encoding


**摘要**: Faithful real-time facial animation is essential for avatar-mediated telepresence in Virtual Reality (VR). To emulate authentic communication, avatar animation needs to be efficient and accurate: able to capture both extreme and subtle expressions within a few milliseconds to sustain the rhythm of natural conversations. The oblique and incomplete views of the face, variability in the donning of headsets, and illumination variation due to the environment are some of the unique challenges in generalization to unseen faces. In this paper, we present a method that can animate a photorealistic avatar in realtime from head-mounted cameras (HMCs) on a consumer VR headset. We present a self-supervised learning approach, based on a cross-view reconstruction objective, that enables generalization to [...]


---


## Motion Capture & Performance Capture

### Hand-Object Interaction Controller (HOIC): Deep Reinforcement Learning for Reconstructing Interactions with Physics

**作者**: Haoyu Hu, Xinyu Yi, Zhe Cao, Jun-Hai Yong, Feng Xu  

**Track**: SIGGRAPH 2024 Conference Track | **DOI**: [10.1145/3641519.3657505](https://doi.org/10.1145/3641519.3657505)  

**关键词**: hand_object, dexterous_manipulation, reinforcement_learning, mocap


**摘要**: Hand manipulating objects is an important interaction motion in our daily activities. We faithfully reconstruct this motion with a single RGBD camera by a novel deep reinforcement learning method to leverage physics. Firstly, we propose object compensation control which establishes direct object control to make the network training more stable. Meanwhile, by leveraging the compensation force and torque, we seamlessly upgrade the simple point contact model to a more physical-plausible surface contact model, further improving the reconstruction accuracy and physical correctness. Experiments indicate that without involving any heuristic physical rules, this work still successfully involves physics in the reconstruction of hand-object interactions which are complex motions hard to imitate with deep reinforcement learning.


---

### Audio Matters Too! Enhancing Markerless Motion Capture with Audio Signals for String Performance Capture

**作者**: Yitong Jin, Zhiping Qiu, Yi Shi, Shuangpeng Sun, Chongwu Wang, Donghao Pan, Jiachen Zhao, Zhenghao Liang, Yuan Wang, Xiaobing Li, Feng Yu, Tao Yu, Qionghai Dai  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658235](https://doi.org/10.1145/3658235)  

**关键词**: mocap, markerless, audio_enhanced, string_performance


**摘要**: In this paper, we touch on the problem of markerless multi-modal human motion capture especially for string performance capture which involves inherently subtle hand-string contacts and intricate movements. To fulfill this goal, we first collect a dataset, named String Performance Dataset (SPD), featuring cello and violin performances. The dataset includes videos captured from up to 23 different views, audio signals, and detailed 3D motion annotations of the body, hands, instrument, and bow. Moreover, to acquire the detailed motion annotations, we propose an audio-guided multi-modal motion capture framework that explicitly incorporates hand-string contacts detected from the audio signals for solving detailed hand poses. This framework serves as a baseline for string performance capture in  [...]


---

### Towards Unstructured Unlabeled Optical Mocap: A Video Helps!

**作者**: Nicholas Milef, John Keyser, Shu Kong  

**Track**: SIGGRAPH 2024 Conference Track | **DOI**: [10.1145/3641519.3657522](https://doi.org/10.1145/3641519.3657522)  

**关键词**: mocap, optical, unstructured, unlabeled, video_prior


**摘要**: Optical motion capture (mocap) requires accurately reconstructing the human body from retroreflective markers, including pose and shape. In a typical mocap setting, marker labeling is an important but tedious and error-prone step. Previous work has shown that marker labeling can be automated by using a structured template defining specific marker placements, but this places additional recording constraints. We propose to relax these constraints and solve for Unstructured Unlabeled Optical (UUO) mocap. Compared to the typical mocap setting that either labels markers or places them w.r.t a structured layout, markers in UUO mocap can be placed anywhere on the body and even on one specific limb (e.g., right leg for biomechanics research), hence it is of more practical significance. It is also more challenging. To solve UUO mocap, we exploit a monocular video captured by a single RGB camera, which does not require camera calibration. On this video, we run an off-the-shelf method to reconstruct and track a human individual, giving strong visual priors of human body pose and shape. With both the video and UUO markers, we propose an optimization pipeline towards marker identification, marker labeling, human pose estimation, and human body reconstruction. Our technical novelties include multiple hypothesis testing to optimize global orientation, and marker localization and marker-part matching to better optimize for body surface.


---


## Avatar & Human Reconstruction

### CharacterGen: Efficient 3D Character Generation from Single Images with Multi-View Pose Canonicalization

**作者**: Hao-Yang Peng, Jia-Peng Zhang, Meng-Hao Guo, Yan-Pei Cao, Shi-Min Hu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658217](https://doi.org/10.1145/3658217)  

**关键词**: character_generation, 3d, single_image, pose_canonicalization


**摘要**: BNRist, Department of Computer Science and Technology, Tsinghua University, China
          In the field of digital content creation, generating high-quality 3D characters from single images is challenging, especially given the complexities of various body poses and the issues of self-occlusion and pose ambiguity. In this paper, we present CharacterGen, a framework developed to efficiently generate 3D characters. CharacterGen introduces a streamlined generation pipeline along with an image-conditioned multi-view diffusion model. This model effectively calibrates input poses to a canonical form while retaining key attributes of the input image, thereby addressing the challenges posed by diverse poses. A transformer-based, generalizable sparse-view reconstruction model is the other core comp [...]


---

### CLAY: A Controllable Large-scale Generative Model for Creating High-quality 3D Assets

**作者**: Longwen Zhang, Ziyu Wang, Qixuan Zhang, Qiwei Qiu, Anqi Pang, Haoran Jiang, Wei Yang, Lan Xu, Jingyi Yu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658146](https://doi.org/10.1145/3658146)  

**关键词**: 3d_generation, controllable, generative_model, assets


**摘要**: In the realm of digital creativity, our potential to craft intricate 3D worlds from imagination is often hampered by the limitations of existing digital tools, which demand extensive expertise and efforts. To narrow this disparity, we introduce CLAY, a 3D geometry and material generator designed to effortlessly transform human imagination into intricate 3D digital structures. CLAY supports classic text or image inputs as well as 3D-aware controls from diverse primitives (multi-view images, voxels, bounding boxes, point clouds, implicit representations, etc). At its core is a large-scale generative model composed of a multi-resolution Variational Autoencoder (VAE) and a minimalistic latent Diffusion Transformer (DiT), to extract rich 3D priors directly from a diverse range of 3D geometries. [...]


---

### Proxy Asset Generation for Cloth Simulation in Games

**作者**: Zhongtian Zheng, Tongtong Wang, Qijia Feng, Zherong Pan, Xifeng Gao, Kui Wu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658177](https://doi.org/10.1145/3658177)  

**关键词**: cloth_simulation, proxy_assets, games, efficient


**摘要**: Simulating high-resolution cloth poses computational challenges in real-time applications. In the gaming industry, the proxy mesh technique offers an alternative, simulating a simplified low-resolution cloth geometry,
            proxy mesh.
            This proxy mesh's dynamics drive the detailed high-resolution geometry,
            visual mesh
            , through Linear Blended Skinning (LBS). However, generating a suitable proxy mesh with appropriate skinning weights from a given visual mesh is non-trivial, often requiring skilled artists several days for fine-tuning. This paper presents an automatic pipeline to convert an ill-conditioned highresolution visual mesh into a single-layer low-poly proxy mesh. Given that the input visual mesh may not be simulation-ready, our approach the [...]


---


## Retargeting, Deformation & Skinning

### A Neural Network Model for Efficient Musculoskeletal-Driven Skin Deformation

**作者**: Yushan Han, Yizhou Chen, Carmichael Ong, Jingyu Chen, Jennifer Hicks, Joseph Teran  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658135](https://doi.org/10.1145/3658135)  

**关键词**: skin_deformation, musculoskeletal, neural, efficient


**摘要**: We present a comprehensive neural network to model the deformation of human soft tissues including muscle, tendon, fat and skin. Our approach provides kinematic and active correctives to linear blend skinning [Magnenat-Thalmann et al. 1989] that enhance the realism of soft tissue deformation at modest computational cost. Our network accounts for deformations induced by changes in the underlying skeletal joint state as well as the active contractile state of relevant muscles. Training is done to approximate quasistatic equilibria produced from physics-based simulation of hyperelastic soft tissues in close contact. We use a layered approach to equilibrium data generation where deformation of muscle is computed first, followed by an inner skin/fascia layer, and lastly a fat layer between the  [...]


---

### Spatial and Surface Correspondence Field for Interaction Transfer

**作者**: Zeyu Huang, Honghao Xu, Haibin Huang, Chongyang Ma, Hui Huang, Ruizhen Hu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658169](https://doi.org/10.1145/3658169)  

**关键词**: correspondence, interaction_transfer, spatial, surface


**摘要**: In this paper, we introduce a new method for the task of interaction transfer. Given an example interaction between a source object and an agent, our method can automatically infer both surface and spatial relationships for the agent and target objects within the same category, yielding more accurate and valid transfers. Specifically, our method characterizes the example interaction using a combined spatial and surface representation. We correspond the agent points and object points related to the representation to the target object space using a learned spatial and surface correspondence field, which represents objects as deformed and rotated signed distance fields. With the corresponded points, an optimization is performed under the constraints of our spatial and surface interaction repr [...]


---


## Cloth, Hair & Garment Simulation

### Super-Resolution Cloth Animation with Spatial and Temporal Coherence

**作者**: Jiawang Yu, Zhendong Wang  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658143](https://doi.org/10.1145/3658143)  

**关键词**: cloth_animation, super_resolution, spatial_temporal, coherence


**摘要**: Creating super-resolution cloth animations, which refine coarse cloth meshes with fine wrinkle details, faces challenges in preserving spatial consistency and temporal coherence across frames. In this paper, we introduce a general framework to address these issues, leveraging two core modules. The first module interleaves a simulator and a corrector. The simulator handles cloth dynamics, while the corrector rectifies differences in low-frequency features across various resolutions. This interleaving ensures prompt correction of spatial errors from the coarse simulation, effectively preventing their temporal propagation. The second module performs mesh-based super-resolution for detailed wrinkle enhancements. We decompose garment meshes into overlapping patches for adaptability to various s [...]


---

### Progressive Dynamics for Cloth and Shell Animation

**作者**: Jiayi Eris Zhang, Doug James, Danny M. Kaufman  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658214](https://doi.org/10.1145/3658214)  

**关键词**: cloth_simulation, shell_animation, progressive_dynamics


**摘要**: We propose Progressive Dynamics, a coarse-to-fine, level-of-detail simulation method for the physics-based animation of complex frictionally contacting thin shell and cloth dynamics. Progressive Dynamics provides tight-matching consistency and progressive improvement across levels, with comparable quality and realism to high-fidelity, IPC-based shell simulations [Li et al. 2021] at finest resolutions. Together these features enable an efficient animation-design pipeline with predictive coarse-resolution previews providing rapid design iterations for a final, to-be-generated, high-resolution animation. In contrast, previously, to design such scenes with comparable dynamics would require prohibitively slow design iterations via repeated direct simulations on high-resolution meshes. We evalua [...]


---

### Real-time Physically Guided Hair Interpolation

**作者**: Jerry Hsu, Tongtong Wang, Zherong Pan, Xifeng Gao, Cem Yuksel, Kui Wu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658176](https://doi.org/10.1145/3658176)  

**关键词**: hair_simulation, interpolation, real_time, physically_guided


**摘要**: Strand-based hair simulations have recently become increasingly popular for a range of real-time applications. However, accurately simulating the full number of hair strands remains challenging. A commonly employed technique involves simulating a subset of guide hairs to capture the overall behavior of the hairstyle. Details are then enriched by interpolation using linear skinning. Hair interpolation enables fast real-time simulations but frequently leads to various artifacts during runtime. As the skinning weights are often pre-computed, substantial variations between the initial and deformed shapes of the hair can cause severe deviations in fine hair geometry. Straight hairs may become kinked, and curly hairs may become zigzags.
          This work introduces a novel physical-driven hair [...]


---

### Contact detection between curved fibres: high order makes a difference

**作者**: Octave Crespel, Emile Hohnadel, Thibaut Metivet, Florence Bertails-Descoubes  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658191](https://doi.org/10.1145/3658191)  

**关键词**: fiber_contact, hair_simulation, collision_detection, high_order


**摘要**: Computer Graphics has a long history in the design of effective algorithms for handling contact and friction between solid objects. For the sake of simplicity and versatility, most methods rely on low-order primitives such as line segments or triangles, both for the detection and the response stages. In this paper we carefully analyse, in the case of fibre systems, the impact of such choices on the retrieved contact forces. We highlight the presence of artifacts in the force response that are tightly related to the low-order geometry used for contact detection. Our analysis draws upon thorough comparisons between the high-order super-helix model and the low-order discrete elastic rod model. These reveal that when coupled to a low-order, segment-based detection scheme, both models yield spu [...]


---

### Automatic Digital Garment Initialization from Sewing Patterns

**作者**: Chen Liu, Weiwei Xu, Yin Yang, Huamin Wang  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658128](https://doi.org/10.1145/3658128)  

**关键词**: garment, simulation, sewing_patterns, initialization


**摘要**: The rapid advancement of digital fashion and generative AI technology calls for an automated approach to transform digital sewing patterns into well-fitted garments on human avatars. When given a sewing pattern with its associated sewing relationships, the primary challenge is to establish an initial arrangement of sewing pieces that is free from folding and intersections. This setup enables a physics-based simulator to seamlessly stitch them into a digital garment, avoiding undesirable local minima. To achieve this, we harness AI classification, heuristics, and numerical optimization. This has led to the development of an innovative hybrid system that minimizes the need for user intervention in the initialization of garment pieces. The seeding process of our system involves the training o [...]


---

### DressCode: Autoregressively Sewing and Generating Garments from Text Guidance

**作者**: Kai He, Kaixin Yao, Qixuan Zhang, Jingyi Yu, Lingjie Liu, Lan Xu  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658147](https://doi.org/10.1145/3658147)  

**关键词**: garment_generation, text_guidance, autoregressive, sewing


**摘要**: Apparel's significant role in human appearance underscores the importance of garment digitalization for digital human creation. Recent advances in 3D content creation are pivotal for digital human creation. Nonetheless, garment generation from text guidance is still nascent. We introduce a text-driven 3D garment generation framework, DressCode, which aims to democratize design for novices and offer immense potential in fashion design, virtual try-on, and digital human creation. We first introduce SewingGPT, a GPT-based architecture integrating cross-attention with text-conditioned embedding to generate sewing patterns with text guidance. We then tailor a pre-trained Stable Diffusion to generate tile-based Physically-based Rendering (PBR) textures for the garments. By leveraging a large lan [...]


---


## Trajectory & Path Planning

### Implicit Swept Volume SDF: Enabling Continuous Collision-Free Trajectory Generation for Arbitrary Shapes

**作者**: Jingping Wang, Tingrui Zhang, Qixuan Zhang, Chuxiao Zeng, Jingyi Yu, Chao Xu, Lan Xu, Fei Gao  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658181](https://doi.org/10.1145/3658181)  

**关键词**: trajectory_generation, collision_free, sdf, swept_volume


**摘要**: In the field of trajectory generation for objects, ensuring continuous collision-free motion remains a huge challenge, especially for non-convex geometries and complex environments. Previous methods either oversimplify object shapes, which results in a sacrifice of feasible space or rely on discrete sampling, which suffers from the "tunnel effect". To address these limitations, we propose a novel hierarchical trajectory generation pipeline, which utilizes the Swept Volume Signed Distance Field (SVSDF) to guide trajectory optimization for Continuous Collision Avoidance (CCA). Our interdisciplinary approach, blending techniques from graphics and robotics, exhibits outstanding effectiveness in solving this problem. We formulate the computation of the SVSDF as a Generalized Semi-Infinite Progr [...]


---


## Animation Tools & Authoring

### SMEAR: Stylized Motion Exaggeration with ARt-direction

**作者**: Jean Basset, Pierre Bénard, Pascal Barla  

**Track**: SIGGRAPH 2024 Conference Track | **DOI**: [10.1145/3641519.3657457](https://doi.org/10.1145/3641519.3657457)  

**关键词**: motion_stylization, exaggeration, art_direction, smear_frames


**摘要**: Smear frames are routinely used by artists for the expressive depiction of motion in animations. In this paper, we present an automatic, yet art-directable method for the generation of smear frames in 3D, with a focus on elongated in-betweens where an object is stretched along its trajectory. It takes as input a key-framed animation of a 3D mesh, and outputs a deformed version of this mesh for each frame of the animation, while providing for artistic refinement at the end of the animation process and prior to rendering. Our approach works in two steps. We first compute spatially and temporally coherent motion offsets that describe to which extent parts of the input mesh should be leading in front or trailing behind. We then describe a framework to stylize these motion offsets in order to produce elongated in-betweens at interactive rates, which we extend to the other two common smear frame effects: multiple in-betweens and motion lines. Novice users may rely on preset stylization functions for fast and easy prototyping, while more complex custom-made stylization functions may be designed by experienced artists through our geometry node implementation in Blender.


---

### Interactive Design of Stylized Walking Gaits for Robotic Characters

**作者**: Michael A. Hopkins, Georg Wiedebach, Kyle Cesare, Jared Bishop, Espen Knoop, Moritz Bächer  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658227](https://doi.org/10.1145/3658227)  

**关键词**: gait_design, robotic_character, stylized, locomotion, interactive


**摘要**: Procedural animation has seen widespread use in the design of expressive walking gaits for virtual characters. While similar tools could breathe life into robotic characters, existing techniques are largely unaware of the kinematic and dynamic constraints imposed by physical robots. In this paper, we propose a system for the artist-directed authoring of stylized bipedal walking gaits, tailored for execution on robotic characters. The artist interfaces with an interactive editing tool that generates the desired character motion in realtime, either on the physical or simulated robot, using a model-based control stack. Each walking style is encoded as a set of sample parameters which are translated into whole-body reference trajectories using the proposed procedural animation technique. In or [...]


---


## Motion Perception

### Towards Motion Metamers for Foveated Rendering

**作者**: Taimoor Tariq, Piotr Didyk  

**Track**: SIGGRAPH 2024 Journal Track (TOG 43·4) | **DOI**: [10.1145/3658141](https://doi.org/10.1145/3658141)  

**关键词**: motion_perception, metamers, foveated_rendering


**摘要**: Foveated rendering takes advantage of the reduced spatial sensitivity in peripheral vision to greatly reduce rendering cost without noticeable spatial quality degradation. Due to its benefits, it has emerged as a key enabler for real-time high-quality virtual and augmented realities. Interestingly though, a large body of work advocates that a key role of peripheral vision may be motion detection, yet foveated rendering lowers the image quality in these regions, which may impact our ability to detect and quantify motion. The problem is critical for immersive simulations where the ability to detect and quantify movement drives actions and decisions. In this work, we diverge from the contemporary approach towards the goal of foveated graphics, and demonstrate that a loss of high-frequency spa [...]


---



## 统计汇总

- 本会议共收录 **26** 篇 motion 相关论文

- Journal Track（TOG）：23 篇  

- Conference Track：3 篇  


### 按类别分布

- Motion Generation & Control：4 篇
- Gesture, Facial & Head Animation：4 篇
- Motion Capture & Performance Capture：3 篇
- Avatar & Human Reconstruction：3 篇
- Retargeting, Deformation & Skinning：2 篇
- Cloth, Hair & Garment Simulation：6 篇
- Trajectory & Path Planning：1 篇
- Animation Tools & Authoring：2 篇
- Motion Perception：1 篇

