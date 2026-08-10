# CIMI4D：人-场景交互下的大规模多模态攀岩运动数据集

> **原文标题**：CIMI4D: A Large Multimodal Climbing Motion Dataset under Human-scene Interactions
> **中文译名**：CIMI4D：人-场景交互下的大规模多模态攀岩运动数据集
> **作者**：Ming Yan（严铭）, Xin Wang, Yudi Dai, Siqi Shen（申思齐，通讯作者）, Chenglu Wen, Lan Xu, Yuexin Ma, Cheng Wang
> **单位**：厦门大学（福建省智慧城市感知与计算重点实验室 / 健康医疗大数据国家研究院 / 多媒体可信感知与高效计算教育部重点实验室）、上海科技大学（智能视觉成像上海工程研究中心）
> **发表**：CVPR 2023（IEEE/CVF Conference on Computer Vision and Pattern Recognition），pp. 12977–12988
> **IEEE Xplore**：https://ieeexplore.ieee.org/document/10205133
> **arXiv**：arXiv:2303.17948
> **项目主页**：http://www.lidarhumanmotion.net/cimi4d/
> **数据说明页**：http://www.lidarhumanmotion.net/data-cimi4d

---

## 目录

- [一、论文速览（TL;DR）](#一论文速览tldr)
- [二、摘要翻译](#二摘要翻译)
- [三、研究背景与动机](#三研究背景与动机)
- [四、数据集构建](#四数据集构建)
- [五、⭐ 数据格式详解（重点）](#五-数据格式详解重点)
- [六、坐标系与投影流程（重点）](#六坐标系与投影流程重点)
- [七、标注方法：混合优化](#七标注方法混合优化)
- [八、任务与基准实验](#八任务与基准实验)
- [九、局限与未来工作](#九局限与未来工作)
- [十、上手使用建议](#十上手使用建议)
- [附录：术语对照表](#附录术语对照表)

---

## 一、论文速览（TL;DR）

| 项目 | 内容 |
|---|---|
| **解决什么问题** | 现有人体动作数据集几乎全是「地面直立正面姿态」（走、坐、跳、舞蹈、瑜伽）；**攀岩这类离地、背面、强自遮挡、强人-场景接触的动作几乎没有数据集** |
| **核心贡献** | 首个 3D 攀岩动作数据集 CIMI4D；一套基于场景+物理约束的混合优化标注方法；4 个任务的基准评测 |
| **数据规模** | 12 名受试者 × 13 面岩壁 = **42 条动作序列**，约 **18 万帧（179,838帧）** |
| **模态** | LiDAR 动态点云 + RGB 视频 + 17 路IMU 动捕 + 高精度静态场景点云 + 重建场景 Mesh + 岩点接触标注 |
| **时长** | RGB 视频 60 分钟；IMU 姿态 180 分钟；总体约 120 分钟动作 |
| **人体表示** | SMPL 模型（pose / shape / trans） |
| **主要结论** | 现有 SOTA 方法（VIBE、LiDARCap、PROX、LEMO、HuMoR 等）在 CIMI4D 上**全面显著退化**，说明攀岩动作对当前 CV 算法构成新挑战 |

**一句话总结**：这是一个用 LiDAR + IMU + RGB 三模态采集的、带高精度 3D 场景和岩点接触标注的攀岩动作数据集，专门用于研究强人-场景交互条件下的 3D 人体姿态估计、预测与生成。

---

## 二、摘要翻译

> 动作捕捉是一个长期存在的研究问题。尽管已被研究数十年，但大多数研究都聚焦于地面运动，例如行走、坐立、舞蹈等。而攀岩这类**离地动作**在很大程度上被忽视了。作为体育和消防领域的一种重要动作类型，攀岩动作由于其**复杂的背部姿态、错综复杂的人-场景交互以及困难的全局定位**而难以捕捉。由于缺乏专门的数据集，研究界对攀岩动作缺乏深入理解。
>
> 为解决这一局限，我们采集了 CIMI4D——一个大规模攀岩运动数据集（rock **CI**mbing **M**ot**I**on，4D 指 3D 空间 + 时间），来自 12 人攀爬 13 面不同岩壁。该数据集包含约 **18 万帧**的惯性姿态测量、LiDAR 点云、RGB 视频、高精度静态点云场景以及重建的场景网格。此外，我们**逐帧标注了手脚触碰的岩点（rock holds）**，以支持对人-场景交互的细致探索。
>
> 该数据集的核心是一套**混合优化流程（blending optimization）**，用于修正因长时间漂移和环境磁场干扰而失准的姿态。为评估 CIMI4D 的价值，我们执行了四项任务：有/无场景约束的人体姿态估计、姿态预测、姿态生成。实验结果表明，CIMI4D 对现有方法构成巨大挑战，并带来了广泛的研究机会。

---

## 三、研究背景与动机

### 3.1 为什么攀岩难捕捉

1. **严重自遮挡**：身体紧贴岩壁，四肢相互遮挡，单目 RGB 方法几乎失效。
2. **背面姿态为主**：主流数据集（Human3.6M、AMASS 等）以正面直立姿态训练，先验完全不匹配。
3. **人-场景紧密接触**：手脚必须精确落在岩点上，姿态与场景强耦合，不能只估计姿态而忽略场景。
4. **全局定位困难**：攀爬高度可达 20m，垂直方向大范围位移，普通相机难以获取全局轨迹。
5. **IMU 不可靠**：长时间采集会漂移；岩壁内含**钢筋**，磁场干扰导致 IMU 朝向失准。

### 3.2 与现有数据集对比（论文 Table 1）

| 数据集 | 3D 场景 | 人体点云 | 交互标注 | LiDAR | RGB 视频 | 全局轨迹 | IMU | 帧数 | 动作类型 | 场地 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|---|---|---|
| PROX | ✓ | ✓ | ✓ | | | | | 20k | 日常 | 室内 |
| LiDARHuman26M | | ✓ | | ✓ | ✓ | | ✓ | 184k | 日常 | 室外 |
| BEHAVE | ✓ | ✓ | ✓ | | | | | 15k | 日常 | 室内 |
| HPS | ✓ | | | | ✓ | ✓ | ✓ | 7k | 日常 | 室外 |
| 3DPW | | | | | ✓ | – | ✓ | 51k | 日常 | 室外 |
| HSC4D | ✓ | ✓ | | ✓ | | ✓ | ✓ | 10k | 日常 | 室外 |
| AMASS | | | | | | ✓ | ✓ | 2420分钟 | 多类 | 两者 |
| RICH | ✓ | ✓ | ✓ | | ✓ | | | 577k | 多类 | 两者 |
| SPEED21 | | | | | ✓ | | | 46k | **攀岩** | 两者 |
| **CIMI4D（本文）** | **✓** | **✓** | **✓** | **✓** | **✓** | **✓** | **✓** | **180k** | **攀岩** | **两者** |

> **要点**：SPEED21 是此前唯一公开的攀岩数据集，但**仅标注 2D 关节点、仅限速度攀岩、时长 38 分钟**。CIMI4D（120 分钟）在规模、模态丰富度、3D 场景方面全面超越，且是**唯一同时具备全部 7 项能力**的数据集。

---

## 四、数据集构建

### 4.1 采集硬件配置

| 设备 | 型号/规格 | 采样率 | 用途 |
|---|---|---|---|
| **IMU 动捕服** | Noitom（诺亦腾）惯性动捕，**17 个 IMU 传感器** | **100 FPS** | 人体姿态（pose） |
| **LiDAR** | **Ouster OS1，128 线** | **20 FPS** | 动态人体点云 + 场景，水平 360°、垂直 45° FOV，**平放**以增大垂直墙面覆盖 |
| **RGB 相机** | DJI Action 2 | **60 FPS** | RGB 视频 |
| **3D 激光扫描仪** | **Trimble X7** | – | 高精度静态场景，**约 4000 万点（40M）**，带 RGB |
| 主机 | Mini Host | – | 数据记录 |

### 4.2 采集设置

- **受试者**：12 人，涵盖**职业运动员、攀岩爱好者、初学者**三个水平层次。
  - ⚠️ 职业运动员**不提供 RGB 视频**（保密协议）。
- **场景**：13 面岩壁，分为三类：
  - **难度攀（lead climbing）**：垂直墙 + 宽墙
  - **速度攀（speed climbing）**：高度可达 **20m**
  - **抱石（bouldering）**：水平长度长
- **高精度场景**：13 面中的 **7 面**做了 Trimble X7 高精度扫描。
- **序列总数**：42 条复杂攀岩动作序列。

### 4.3 人体姿态模型（SMPL）

一段人体运动记作`M = (T, θ, β)`：

| 符号 | 含义 | 维度 | 说明 |
|---|---|---|---|
| `T` | 平移参数（全局轨迹） | **N × 3** | 骨盆（pelvis）的全局位置 |
| `θ` | 姿态参数 | **N × 24 × 3** | SMPL 24 个关节的轴角（axis-angle）旋转 |
| `β` | 体型参数 | **N × 10** | SMPL shape 参数 |
| `N` | 帧数 | – | 时序点云帧数 |

通过映射函数 `Φ` 得到三角网格：`V_k, F_k = Φ(T, θ, β)`

| 输出 | 维度 |
|---|---|
| 顶点 `V_k` | **R^(6890×3)** |
| 面片 `F_k` | **R^(13690×3)** |

---

## 五、⭐ 数据格式详解（重点）

>本节整合论文正文与官方数据说明页（`/data-cimi4d`）的权威信息。

### 5.1 数据集总体划分

CIMI4D 分为 **两大子集**：

| 子集 | 说明 | RGB 数据 | 标注文件格式 | 场景点云 |
|---|---|:---:|---|---|
| **XMU** | 厦门大学采集（志愿者：爱好者/初学者） | ✅ 有 | **`.pkl`** | 稀疏 + **稠密**（部分） |
| **ChangSha（长沙）** | 含**国家级职业运动员** | ❌ **无**（保密协议） | **`.h5py`** |仅稀疏 |

> ⚠️ **注意**：ChangSha 部分未在论文中提及，不属于论文贡献点，但数据集中包含。

---

### 5.2 XMU 子集目录结构

```text
├── root_folder
├── sequence_img/            # RGB 图像序列：视频降采样到 20fps 并与点云对齐
├── sequence_pos.csv         # 原始 IMU 位置数据
├── sequence_rot.csv         # 原始 IMU 旋转数据
├── sequence.bvh             # 原始人体动作（BVH 格式，100 fps）
├── Sparse_scene_mesh.ply    # 稀疏场景网格（LiDAR 重建）
├── Sparse_scene_points.pcd  # 稀疏场景点云（LiDAR）
├── LiDAR2Cam_e.txt          # LiDAR → 相机 的外参（+ 相机内参）
├── lidar_p.txt              # LiDAR → 世界坐标系（Mocap）的外参 RT
├── Dense_scene_mesh.ply     # 稠密场景网格（Trimble X7 高精度重建）
├── Dense_scene_points.pcd   # 稠密场景点云（高精度，约 40M 点）
├── sequence.pkl             #⭐ 核心标注文件
│   ├── 'beta'                # 受试者 SMPL 体型参数
│   ├── 'gt_pose_3D'          # 真值 3D 姿态
│   ├── 'gt_pose_2D'          # 真值 2D 姿态
│   ├── 'gt_trans'# 真值全局平移（轨迹）
│   ├── 'human_point_clouds'  # 分割出的人体点云序列
│   ├── 'IMU_pose'# 原始 IMU 姿态（未优化）
│   ├── 'IMU_trans'           # 原始 IMU 平移（未优化）
│   └── 'frame_num'           # 帧号
└── others...
```

---

### 5.3 ChangSha 子集目录结构

```text
├── root_folder
├── sequence_pos.csv         # 原始 IMU 位置数据
├── sequence_rot.csv         # 原始 IMU 旋转数据
├── sequence.bvh             # 原始人体动作（BVH，100 fps）
├── Sparse_scene_points.pcd  # 稀疏场景点云
├── sequence.json            # 序列人体信息（Sequence Human Info）
├── sequence_label.h5py      # ⭐ 核心标注文件（字段同 XMU 的 pkl）
│   ├── 'beta'
│   ├── 'gt_pose_3D'
│   ├── 'gt_pose_2D'
│   ├── 'gt_trans'
│   ├── 'human_point_clouds'
│   ├── 'IMU_pose'
│   ├── 'IMU_trans'
│   └── 'frame_num'
└── others...
```

> **与 XMU 的差异**：无 `sequence_img/`、无 `LiDAR2Cam_e.txt`、无 Dense 场景、无 mesh；标注用 h5py 而非 pkl；多一个 `sequence.json`。

---

### 5.4 核心标注字段逐项说明

| 字段 | 含义 | 推断维度 | 说明 |
|---|---|---|---|
| `beta` | SMPL 体型参数 | **(10,)** 或 **N×10** | 每个序列一个受试者，体型固定；官方称为"每条序列的关键信息" |
| `gt_pose_3D` | **优化后**的真值 3D 姿态 | **N×24×3** | SMPL轴角姿态参数 θ（经混合优化修正）。这是训练/评测应使用的标签 |
| `gt_pose_2D` | 真值 2D 姿态 | **N×J×2** | 投影到图像平面的 2D 关节点（仅 XMU 有 RGB 时有意义） |
| `gt_trans` | **优化后**的全局平移 | **N×3** | ⭐ **x, y, z 即全局轨迹**（官方明确说明） |
| `human_point_clouds` | 分割后的人体点云序列 | 每帧点数不定 | 从 LiDAR 每帧中分割出的人体点，用于点云类方法（LiDARCap/P4Transformer） |
| `IMU_pose` | **原始未优化** IMU 姿态 | **N×24×3** | 存在漂移与磁干扰误差，可作 baseline 或对比用 |
| `IMU_trans` | **原始未优化** IMU 平移 | **N×3** | 漂移严重，不可直接当真值 |
| `frame_num` | 帧号/帧数 | – | 用于时间对齐 |

> 💡 **关键区分**：`gt_*` 是经过混合优化 + 人工修正的**标签**；`IMU_*` 是设备原始输出，**不是真值**。这是使用数据时最容易搞错的地方。

---

### 5.5 文件格式与工具对照

| 扩展名 | 内容 | 推荐读取方式 |
|---|---|---|
| `.pkl` | XMU 标注（Python dict） | `pickle.load(f, encoding='latin1')` |
| `.h5py` / `.h5` | ChangSha 标注 | `h5py.File(path, 'r')` |
| `.csv` | 原始 IMU pos / rot | `pandas.read_csv` |
| `.bvh` | 原始动捕动作，**100 fps** | `bvh` / `PyMO` / Blender 导入 |
| `.pcd` | 点云（场景） | `open3d.io.read_point_cloud` |
| `.ply` | 网格（场景 mesh） | `open3d.io.read_triangle_mesh` / `trimesh` |
| `.json` | ChangSha 序列人体信息 | `json.load` |
| `.txt` | 标定外参/内参矩阵 | `numpy.loadtxt` |
| 图像目录 | `sequence_img/`，已降采样至 **20 fps** | `cv2.imread` / `PIL` |

---

### 5.6 帧率与时间同步

| 模态 | 原始帧率 | 对齐后帧率 |
|---|---|---|
| LiDAR 点云 | **20 FPS** | 20 FPS（**基准**） |
| IMU 姿态 | 100 FPS | 降采样至 **20 FPS** |
| RGB 视频 | 60 FPS | 降采样至 **20 FPS** |
| `.bvh` 原始动作 | **100 FPS**（保留原始） | — |

**同步方法**：每条序列开始时受试者**原地起跳**，用峰值检测算法在 IMU 与 LiDAR 的轨迹中自动找到高度峰值，以该峰值时间戳对齐三路数据。

> ⚠️ 若你需要 100 fps 的高频动作，请用 `.bvh`；若做多模态对齐训练，用 20 fps 的 pkl/h5py + `sequence_img/`。

---

### 5.7 ⚠️ 几个重要的数据「坑」（官方 Tips）

1. **岩点（Holds）接触数据不直接提供**
   - 原因：岩点厂商拒绝公开其**专利岩点形状**。
   - 替代方案：官方提供岩点提取算法源码 **`Holds_Extract_ASC_XMU.py`**（基于 **DBSCAN 聚类 + RANSAC 平面拟合**）。
   - 用法：自行调节 DBSCAN / RANSAC 参数，代码内含可视化辅助调参；**代码结果取反（inversion）即为各面墙的岩点**。

2. **场景 Mesh 只保留了一个样例**
   - Mesh 来自点云的**泊松重建（Poisson reconstruction）**。
   - 因重建误差会产生误导，官方**删除了其余序列的 mesh**，仅在 `XMU_1023_zpc001_V1_1` 中保留 `.ply` 作为效果参考。
   - **建议**：需要 mesh 时，自行从 `.pcd` 点云场景重建。

3. **部分序列无 RGB**
   - ChangSha 全部序列无 RGB（国家级运动员保密协议）。
   - XMU 中**`XMU_1023_ym002_V1_0`** 因隐私原因无 RGB 数据。

4. **单位陷阱（最容易出错）**
   -⚠️ **`lidar_p.txt` 外参矩阵中的平移 T 单位是毫米（mm），需除以 1000 转换为米（m）**。

5. **bbox 格式**
   - bbox = **中心点坐标 (cx, cy) + 宽 w + 高 h**（非左上-右下格式）。

6. **获取方式**
   - 非公开直链下载。需发邮件至 **yanmnn@stu.xmu.edu.cn**，注明：职称/头衔、姓名全称、单位、国家、下载用途，并接受官方 License。
   - 商业授权联系：siqishen@xmu.edu.cn / clwen@xmu.edu.cn / cwang@xmu.edu.cn

---

## 六、坐标系与投影流程（重点）

### 6.1 三套坐标系定义

| 记号 | 坐标系 | 说明 |
|---|---|---|
| `{I}` | **IMU 坐标系** | 动捕设备（Mocap）坐标系 |
| `{L}` | **LiDAR 坐标系** |激光雷达坐标系 |
| `{W}` | **全局/世界坐标系** | 最终标注所在坐标系 |

**记号约定**：下标 `k` 表示帧序号；上标 `I/L/W` 表示所属坐标系。例：LiDAR 点云帧记作 `P^L = {P_k^L, k ∈ Z+}`。

### 6.2 数据存储所在坐标系

- **标注（`gt_pose_3D`、`gt_trans`）存储在世界坐标系 `{W}`**。
- `gt_trans` 的 `x, y, z` 即**全局轨迹**。
- 为统一坐标系，官方先将 Mocap 坐标系与 LiDAR 坐标系**都转换到世界坐标系**；同时将 RGB 与 LiDAR 坐标系下的点云场景做了**配准**，以便标定初始图像与初始雷达。

### 6.3 ⭐ 投影到 2D 图像的完整链路

若你要把 SMPL 人体渲染/投影到 RGB 图像上，必须走完整的三步变换：

```text
世界坐标系 {W}  ──[ lidar_p.txt 的 RT 逆变换 ]──▶  LiDAR 坐标系 {L}
                                                        │
                                    [ LiDAR2Cam_e.txt 外参 ]
                                                        ▼
                                                 相机坐标系 {C}
                                                        │
                                    [ LiDAR2Cam_e.txt 内参 K ]
                                                        ▼
                                                2D 图像平面
```

| 步骤 | 所需文件 | 内容 |
|---|---|---|
| ① `{W}` → `{L}` | **`lidar_p.txt`** | LiDAR→世界 的 RT 矩阵（**T 单位 mm，需 /1000**） |
| ② `{L}` → `{C}` | **`LiDAR2Cam_e.txt`** | LiDAR→相机 外参矩阵 |
| ③ `{C}` → 图像 | **`LiDAR2Cam_e.txt`** | 相机内参矩阵 K |

> 💡 **实践提示**：`lidar_p.txt` 给的是 LiDAR→世界，所以从世界回到 LiDAR 需要用**逆变换**。转换前务必先把 T 除以 1000。

### 6.4 姿态初始化（预处理阶段）

世界坐标系下的运动序列 `M^W = (T^W, θ^W, β)`：
- 动捕设备直接给出 `M^I = (T^I, θ^I, β)` 中的 `T^I` 和 `θ^I`；
- 姿态初始化：`θ^W = R^{WI} · θ^I`，其中 `R^{WI}` 是从 `{I}` 到 `{W}` 的**粗标定矩阵**；
- 平移初始化：取**人体点云的重心**作为初始平移。

---

## 七、标注方法：混合优化

整个标注流水线分 **3 个阶段**：

### 7.1 阶段一：多模态数据预处理

1. 将高精度 3D 激光扫描数据转为**带颜色的点云场景**；
2. 将 LiDAR 记录的点云序列转为动态场景，并**配准静态场景与动态场景**；
3. 从每帧中**分割人体点云**；基于 IMU 测量得到 SMPL 姿态 θ；
4. 对场景、人体姿态、人体点云、RGB 视频做**帧级时间同步与朝向标定**（起跳峰值检测法）。

### 7.2 阶段二：混合优化（核心）

总损失函数：

```
L = λ_lc · L_lc + λ_ls · L_ls + L_smooth + λ_sp · L_sp
```

用**梯度下降**最小化 L，优化对象为 `M^W = (T, θ)`。

| 约束项 | 名称 | 作用 |
|---|---|---|
| **`L_lc`** | 肢体接触损失<br>(Limb Contact Loss) | 鼓励手脚与场景网格**合理接触且不穿透**。判定：某肢体位移 < **3cm** 且小于另一肢体位移时标记为"稳定"，然后对稳定肢体做邻域搜索找接触环境。`L_lc = L_lc_feet + L_lc_hand` |
| **`L_ls`** | 肢体滑动损失<br>(Limb Sliding Loss) | 消除攀爬时肢体在接触面上的**不合理滑移**。定义为稳定肢体在**相邻两帧间**的距离。`L_ls = L_ls_feet + L_ls_hands` |
| **`L_smooth`** | 平滑损失 | `L_smooth = λ_trans·L_trans + λ_joints·L_joints`。`L_trans` 平滑骨盆轨迹（最小化 LiDAR 与人体平移差异）；`L_joints` 最小化关节**平均加速度**（仅考虑躯干与颈部的稳定关节） |
| **`L_sp`** | SMPL-到-点云损失<br>(SMPL to Point Loss) | 用 **HPR（Hidden Points Removal）** 剔除 LiDAR 视角下不可见顶点，用 **ICP** 将可见顶点配准到分割人体点云，最小化两者的**3D Chamfer 距离** |

### 7.3 阶段三：人工标注

1. **姿态与平移标注**：对优化后仍有瑕疵（artifacts）的帧，手工修改pose 与 trans 参数。
2. **场景接触标注**：标注场景中**所有攀岩岩点**；对部分序列进一步标注手脚**与岩点接触的时刻**。
3. **交叉验证**：邀请 **2 位外部研究者**检查数据集，人工修正其发现的瑕疵。

### 7.4 标注质量的量化验证（论文 Table 2）

以人工标注为参考，评估优化阶段输出的误差（单位：mm，ACCEL 单位 m/s²）：

| 场景 | ACCEL ↓ | PMPJPE ↓ | MPJPE ↓ | PVE ↓ | PCK@0.5 ↑ |
|---|---|---|---|---|---|
| Vertical 1（垂直墙 1） | 0.57 | 2.04 | 6.52 | 8.08 | 0.99 |
| Horizontal 1（水平墙 1） | 0.50 | 1.96 | 4.26 | 6.27 | 0.99 |

> 误差极小（MPJPE 仅 4–7mm，PCK 达 0.99），说明标注流水线有效、数据质量高。

### 7.5 约束项消融（论文 Table 3）

不同约束组合下的优化损失（损失越小表示越符合运动约束）：

| `L_lc` | `L_smooth` | `L_sp` | Vertical 1 | Vertical 2 | Horizontal 1 | Horizontal 2 |
|:---:|:---:|:---:|---|---|---|---|
| ✗ | ✗ | ✗ | 48.28 | 60.04 | 59.83 | 47.74 |
| ✓ | ✓ | ✗ | 22.64 | 28.33 | 41.67 | 26.64 |
| ✓ | ✗ | ✓ | 33.48 | 40.44 | 44.77 | 31.44 |
| ✗ | ✓ | ✓ | 24.64 | 38.37 | 42.07 | 30.08 |
| **✓** | **✓** | **✓** | **16.24** | **23.46** | **34.34** | **20.21** |

> **结论**：三项约束全用时损失最小，说明每一项都是必要的。

### 7.6 定性对比（论文 Fig. 5）

| 方案 | 问题 |
|---|---|
| LiDAR 点云 | 有人体点云，但**无姿态与平移** |
| IMU + IMU-Trans | **平移随时间漂移**，位置错误 |
| IMU + LiDAR-Trans（仅预处理） | 质量改善，但受岩壁**钢筋磁场**干扰，**摸错岩点** |
| IMU + Opt-Trans（优化但无平滑损失） | 摸错岩点次数减少 |
| **本文（Opt-Pose & Trans）** | **准确重建姿态与平移** |

---

## 八、任务与基准实验

**数据划分**：动作序列按 **7:3** 随机划分为训练集/ 测试集。

**评价指标**：

| 指标 | 全称 | 单位 | 方向 |
|---|---|---|---|
| **PMPJPE** | Procrustes-Aligned Mean Per Joint Position Error | mm | ↓ |
| **MPJPE** | Mean Per Joint Position Error | mm | ↓ |
| **PVE** | Per Vertex Error | mm | ↓ |
| **PCK@0.5** | Percentage of Correct Keypoints | – | ↑ |
| **ACCEL** | Acceleration Error | **m/s²** | ↓ |

### 8.1 任务一& 二：3D 姿态估计（有/无场景约束）

论文 Table 4 结果（`⋆` = 在 CIMI4D 上**重新训练**；`⋄` = 在 CIMI4D 上**微调**；其余为原方法预训练模型）：

| 输入 | 方法 | ACCEL ↓ | PMPJPE ↓ | MPJPE ↓ | PVE ↓ | PCK@0.5 ↑ |
|---|---|---|---|---|---|---|
| **LiDAR** | LiDARCap | 12.39 | 222.11 | 358.13 | 422.65 | 0.50 |
| | **LiDARCap⋆** | **2.59** | **86.38** | **115.93** | **136.83** | **0.90** |
| | P4Transformer⋆ | 3.32 | 100.58 | 130.99 | 156.27 | 0.87 |
| **RGB** | VIBE | 68.02 | 287.14 | 770.77 | 857.83 | 0.17 |
| | VIBE⋄ | 57.88 | 116.78 | 161.21 | 187.70 | 0.76 |
| | MAED⋄ | 17.50 | 135.57 | 170.43 | 197.66 | 0.74 |
| | DynaBOA | 52.4 | 230.86 | 303.16 | 285.62 | 0.54 |
| **Scene** | PROX | – | 109.34 | 265.34 | 279.50 | 0.53 |
| | PROX⋄ | – | 109.33 | 147.41 | 165.12 | 0.79 |
| | LEMO | 98.3 | 317.64 | 669.38 | 359.11 | 0.45 |

**关键观察**：
- 预训练 LiDARCap 在 CIMI4D 上仅 **PCK=0.46~0.50**，性能崩塌；重训后升至 0.90，说明**必须用攀岩数据训练**。
- RGB 方法整体更差（VIBE 预训练 PCK 仅 **0.17**，MPJPE 高达 **770mm**）；微调后改善但仍远逊于原论文水平。
- **LiDAR 方法整体优于 RGB 方法**——因为攀岩自遮挡严重、人体与场景颜色相近，2D 检测（OpenPose）大量失败。
- 场景感知方法 **PROX 优于**不使用场景约束的方法，验证了 CIMI4D 提供人-场景交互标注的必要性。
- PROX / LEMO 依赖外部提供 2D 骨架，OpenPose 在此类强遮挡姿态下失效，导致关节重建严重偏差。

### 8.2 任务三：带场景约束的动作预测

- **任务定义**：基于**前一帧**预测未来的姿态 + 全局轨迹（比只预测姿态更难）。
- 因 LiDAR 优于 RGB，故基于点云做预测。
- **Baseline 设计**：以 **LiDARCap为backbone** + **条件变分自编码器（cVAE）** 建模人体运动分布。
- **对比方法**：HuMoR（ICCV 2021）。
- **结果**：
  - cVAE 预测幅度偏小但**合理**，能观察到自然平滑的手部运动；存在平移误差与穿透瑕疵。
  - **HuMoR 完全无法预测攀岩动作**——其先验来自 AMASS（日常动作），所有攀岩动作最终都退化成日常动作，并产生大量不自然与**自穿透**姿态。

### 8.3 任务四：带场景约束的动作生成

- **任务定义**：给定场景（特定岩点组合），生成物理上合理的姿态——对攀岩者规划路线有实际意义。
- **Baseline**：条件变分自编码器（cVAE），在**模型未见过的岩点**上生成姿态与平移。
- **结果**：
  - 部分岩点组合能生成职业攀岩者认可的**合理自然**姿态（图中橙色）；
  - 部分组合失败（图中红色）：**动作不自然**、**触碰到非岩点位置**；
  - 整体**生成多样性不足**，带场景约束的姿态+平移生成仍是难题。

---

## 九、局限与未来工作

论文自述三大局限：

| # | 局限 | 影响 | 未来方向 |
|---|---|---|---|
| 1 | **缺少精细手部姿态** | 导致手与岩点存在轻微**穿透** | 使用**动捕手套** + **SMPL-X** 模型 |
| 2 | **无细粒度攀岩动作类别标注** | 只记录姿态，缺少动作语义分类 | 补充细粒度攀岩动作标注 |
| 3 | **未做照片级真实的3D 场景重建** | 侧重人体运动重建，场景视觉真实度不足 | 用**神经渲染（NeRF 类）**重建场景与人体 |

---

## 十、上手使用建议

### 10.1 快速开始清单

```text
□ 1. 发邮件到 yanmnn@stu.xmu.edu.cn 申请数据（注明单位/用途，接受 License）
□ 2. 确认拿到的是 XMU 还是 ChangSha 子集 → 决定用 pickle 还是 h5py 读标注
□ 3. 读取标注：优先用 gt_pose_3D / gt_trans / beta，切勿把 IMU_* 当真值
□ 4. 需要 100fps 高频动作 → 用 .bvh；需要多模态对齐 → 用 20fps 的 pkl + sequence_img/
□ 5. 要投影到图像：lidar_p.txt（T 记得 /1000）→ LiDAR2Cam_e.txt（外参 + 内参）
□ 6. 需要岩点：跑 Holds_Extract_ASC_XMU.py（DBSCAN + RANSAC），结果取反
□ 7. 需要场景 mesh：自己从 .pcd 泊松重建，不要依赖官方 mesh（已删除大部分）
```

### 10.2 参考读取代码骨架

```python
import pickle, h5py, numpy as np

# ---- XMU 子集：pkl ----
with open('sequence.pkl', 'rb') as f:
    d = pickle.load(f, encoding='latin1')

beta      = np.asarray(d['beta'])              # SMPL 体型，(10,) 或 N×10
pose_gt   = np.asarray(d['gt_pose_3D'])        # N×24×3，优化后真值姿态（轴角）
trans_gt  = np.asarray(d['gt_trans'])# N×3，全局轨迹 (x, y, z)
pose_2d   = np.asarray(d['gt_pose_2D'])        # N×J×2，2D 关节点
pc_human  = d['human_point_clouds']            # 每帧人体点云（点数不定）
pose_imu  = np.asarray(d['IMU_pose'])          # N×24×3，原始 IMU（含漂移，非真值）
trans_imu = np.asarray(d['IMU_trans'])         # N×3，原始 IMU 平移（漂移大）
frame_num = d['frame_num']

# ---- ChangSha 子集：h5py ----
with h5py.File('sequence_label.h5py', 'r') as f:
    print(list(f.keys()))                      # 字段与 XMU 一致
    pose_gt_cs  = f['gt_pose_3D'][:]
    trans_gt_cs = f['gt_trans'][:]

# ---- 外参：注意单位换算 ----
RT_lidar2world = np.loadtxt('lidar_p.txt').reshape(4, 4)
RT_lidar2world[:3, 3] /= 1000.0                # ⚠️ mm → m，必做

# ---- 世界 → LiDAR → 相机 → 图像 ----
RT_world2lidar = np.linalg.inv(RT_lidar2world)
# 再用 LiDAR2Cam_e.txt 里的外参与内参 K 完成投影
```

> ⚠️ 上述矩阵的具体排布（是否需要 reshape、行列顺序）请以实际拿到的 `lidar_p.txt` / `LiDAR2Cam_e.txt` 文件内容为准。

### 10.3 常见踩坑速查

| 现象 | 可能原因 |
|---|---|
| 人体投影到图像后偏移巨大 | `lidar_p.txt` 的 **T 未除以 1000**（mm/m 单位错误） |
| 人体位置整体漂移、越爬越偏 | 误把 `IMU_trans` 当作真值，应用 `gt_trans` |
| 姿态看起来像日常动作而非攀岩 | 用了 AMASS 先验的模型（如 HuMoR），需在 CIMI4D 上重训 |
| 手脚穿透岩壁 | 数据本身缺精细手部姿态（已知局限）；或 mesh 来自泊松重建有误差 |
| 找不到接触/岩点标注文件 | 岩点因专利不直接提供，需运行 `Holds_Extract_ASC_XMU.py` 自行提取 |
| RGB 目录为空 | ChangSha 全部无 RGB；XMU 的 `XMU_1023_ym002_V1_0` 无 RGB |
| 场景 mesh 缺失 | 官方仅在 `XMU_1023_zpc001_V1_1` 保留样例，其余需自行重建 |
| 帧数与图像数量不匹配 | 注意 IMU 100fps / RGB 60fps 均降采样到 LiDAR 的 20fps |

---

## 附录：术语对照表

| 英文 | 中文 | 说明 |
|---|---|---|
| Motion Capture (MoCap) | 动作捕捉 | |
| Human Pose Estimation (HPE) | 人体姿态估计 | |
| Off-grounded action | 离地动作 | 与地面动作相对 |
| Rock hold / hold | 岩点 |岩壁上供手脚抓踩的凸起 |
| Lead climbing | 难度攀 / 先锋攀 | |
| Speed climbing | 速度攀 | 本数据集高度可达 20m |
| Bouldering | 抱石 | 水平长度长的低矮岩壁 |
| Blending optimization | 混合优化 | 本文核心标注方法 |
| Limb contact / sliding | 肢体接触 /滑动 | 两个关键约束 |
| Chamfer distance | 倒角距离 | 点集间距离度量 |
| HPR (Hidden Points Removal) | 隐藏点剔除 | 模拟 LiDAR 可见性 |
| ICP (Iterative Closest Point) | 迭代最近点 | 点云配准算法 |
| SMPL / SMPL-X | 参数化人体模型 | 6890 顶点 / 13690 面片 |
| DBSCAN | 密度聚类算法 | 用于岩点提取 |
| RANSAC | 随机采样一致性 | 用于平面拟合 |
| Poisson reconstruction | 泊松重建 | 点云转网格 |
| cVAE | 条件变分自编码器 | 预测/生成任务 baseline |
| Artifact | 瑕疵 / 伪影 | |
| Self-penetration | 自穿透 | 肢体互相穿模 |
| Extrinsic / Intrinsic | 外参 / 内参 | 相机标定参数 |

---

## 引用格式（BibTeX）

```bibtex
@inproceedings{yan2023cimi4d,
  title     = {CIMI4D: A Large Multimodal Climbing Motion Dataset under Human-scene Interactions},
  author    = {Yan, Ming and Wang, Xin and Dai, Yudi and Shen, Siqi and Wen, Chenglu and Xu, Lan and Ma, Yuexin and Wang, Cheng},
  booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
  pages     = {12977--12988},
  month     = {June},
  year      = {2023}
}
```

---

## 资料来源

1. 论文正文 PDF（CVPR Open Access 版，12 页）
2. 官方项目主页：http://www.lidarhumanmotion.net/cimi4d/
3. **官方数据说明页**（数据结构、字段、Tips 的权威来源）：http://www.lidarhumanmotion.net/data-cimi4d
4. IEEE Xplore 元数据：https://ieeexplore.ieee.org/document/10205133

> **说明**：本文档中数据结构、字段名、坐标系转换流程、单位换算（mm→m）、bbox 定义、岩点提取脚本等信息均来自官方数据说明页；`gt_pose_3D`、`gt_trans` 等字段的具体维度是结合论文中 SMPL 定义（θ: N×24×3、T: N×3、β: N×10）推断标注的，实际使用时建议先打印数组shape 核对。
