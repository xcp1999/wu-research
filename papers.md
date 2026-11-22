# Papers — 已读论文与研究记忆

## ASB–GAC — Humanoid Whole-Body Manipulation via Active Spatial Brain and Generalizable Action Cerebellum（简称 ASB–GAC）

**重点关注**：True

**开源情况**：完全没开[2026-10-15]（指本文系统实现；官方目前仅网页与演示公开，未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-05-20（所读 v2：2026-07-07，另核查正式版补充材料）；一作（共同）：Zhizhao Liang、Yi-Lin Wei、Xuhang Chen；通讯：Wei-Shi Zheng；单位：中山大学计算机学院、鹏城实验室（按所读 arXiv v2）；是否中稿：是，[ECCV 2026](https://link.springer.com/chapter/10.1007/978-3-032-37422-6_8)；[项目主页](https://leungchaos.github.io/Humanoid-Whole-Body-Manipulation-via-Active-Spatial-Brain-and-Generalizable-Action-Cerebellum/)，[arxiv](https://arxiv.org/abs/2605.21133)。

**核心问题**：作者针对人形机器人在复杂空间中看不全、目标不可达、任务数据昂贵的问题，组合主动观察与任务规划、几何规划、基础模型抓取和下肢控制。

**系统拆解**：按[原文 §3／图2](https://arxiv.org/html/2605.21133v2#S3)的层级，一级组件为 Active Spatial Brain（ASB）和 Generalizable Action Cerebellum（GAC）。输入语言、RGB-D、本体状态；ASB 调用 GAC 并接收反馈，最终输出相机动作、灵巧手姿和上下肢关节目标，由 PD 执行。下表将 GAC 的两个子模块分行说明。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| ASB：Active Spatial Brain | 三个模块：Active Spatial Perception、Memory Bank、Adaptive Task Planning | 观察与历史 → 调整视角、分解／重规划任务 → 调用 GAC。图像记忆只保留最近一次基座移动以来的图像与相机位姿，文本保留全程命令和反馈。 |
| GAC → Humanoid Lower-Body Locomotion | 两个 agent：Spatial Navigation、End-Effector Reachable Space Solver；下接 RL locomotion policy | RGB-D／目标位置 → 栅格与 A* 路径、可达性位置优化 → 速度／转向／高度指令 → RL 策略输出下肢关节动作。 |
| GAC → Humanoid Upper-Body Dexterous Manipulation | 两个 agent：Grasp Pose Generation、Post-grasp Trajectory Generation | 观察／指令 → 合成人手物交互图＋物体重建 → 手／物姿态估计、对齐及灵巧手重定向；随后关键点＋推／拉／放／旋转原语 → 末端轨迹 → IK 输出关节命令。 |

**训练、复用与新增**：与上表逐项对应。[正式版补充 §2–3](https://media.springernature.com/original/springer-static/esm/chp%3A10.1007%2F978-3-032-37422-6_8/MediaObjects/700396_1_En_8_MOESM1_ESM.pdf)将框架称为 training-free（作者主张）；不能据此把底层策略的预训练也理解为不存在。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| ASB | prompt 调用现成 VLM，未报告另行训练；补充表2将规划模型记为 Gem-fl.，未展开完整型号。模型预训练数据未列。 | 主动视角控制、移动条件图像记忆和基于执行反馈的重规划。 |
| GAC → 下肢 | 复用 A*／RDP，采用 HOMIE 下肢策略；作者从头训练、微调还是直接加载权重仍未披露。[原 HOMIE](https://homietele.github.io/)的 RL 经验来自 Isaac Gym 在线仿真、无需 MoCap，本文是否完全照搬训练配置未知。肩部偏移模型 Φ(z)由 MuJoCo 估计。 | 可达性求解器调整站位与高度，并接入下肢控制。 |
| GAC → 上肢 | 补充明确默认用 Nano Banana 2 生成抓握图，Hyper3D 生成物体 mesh；HaMeR 和 MegaPose 分别估计合成图中的手姿与物姿，再按 EasyHOI 方法优化深度轴，最后重定向到灵巧手。抓后轨迹模型记为 Qwen-3.5，未给后缀／checkpoint；未报告这些模型在本文中微调。图像与 mesh 是推理／逐实例优化输入，非新增动作训练集。 | 将生成、重建、姿态估计和重定向接成抓握流程；用语义关键点、参数化原语与 IK 生成抓后轨迹。 |

**关键 Q&A（2026-10-15）**：Q：作者自己训练了什么，是从头还是微调，数据从哪来？A：作者称框架 training-free，主要调用现成模型；下肢采用经过 RL 训练的 HOMIE 路线策略，具体权重初始化仍未披露。§4.3 每设置至少20条 Easy 轨迹用于 TrajBooster／Ψ₀ 对照微调。补充材料补全了此前未确定的部分模型名称；训练经历与作者亲自重训仍须区分。

## VIRAL — Visual Sim-to-Real at Scale for Humanoid Loco-Manipulation

**重点关注**：True

**开源情况**：可以sim[2026-10-15]（[官方代码](https://github.com/NVlabs/GR00T-VisualSim2Real)已提供训练／仿真评估、权重、示范与资产，需 Git LFS、Isaac Sim 5.1／Isaac Lab；真机仅提供 ONNX 导出，未找到部署入口；未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2025-11-19（所读 v2：2025-11-27，正文及附录，另核查正式版与公开代码）；一作（共同）：Tairan He、Zi Wang、Haoru Xue、Qingwei Ben；通讯：未明确标注，Linxi Fan／Yuke Zhu 标注为项目负责人；单位：NVIDIA、CMU、UC Berkeley、CUHK；是否中稿：是，[CVPR 2026](https://openaccess.thecvf.com/content/CVPR2026/html/He_VIRAL_Visual_Sim-to-Real_at_Scale_for_Humanoid_Loco-Manipulation_CVPR_2026_paper.html)；[项目主页](https://viral-humanoid.github.io/)，[arxiv](https://arxiv.org/abs/2511.15200)。

**核心问题**：用大规模仿真学习人形机器人的视觉移动操作，在不使用真机任务示范训练／微调策略的条件下，将行走、放置、抓取和转身迁移到实机。

**系统拆解**：按[原文 §2／图2](https://arxiv.org/html/2511.15200v2#S2)，训练分 Teacher → Student 两阶段，二者都以预训练 WBC 为执行基础；部署时由 Student 接收 RGB＋本体状态并调用 WBC，Teacher 不参与实机决策。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Privileged RL Teacher | 任务层策略 MLP；Delta Action Space；Reference State Initialization（RSI） | 仿真完整状态，含任务阶段、物体位姿与接触信息 → 任务决策 → 移动／转向、手臂及手指的增量指令。 |
| Vision-based Student | 视觉编码器＋融合本体状态的策略头 | 机载 RGB＋可实测本体状态 → 预测 Teacher 的动作 → 相同指令接口；无需实机物体真值状态。 |
| Whole-Body Control（WBC） | 预训练 HOMIE；扩展手指指令接口 | 累积后的移动／上身指令＋本体状态 → 下身运动控制与上身、手指目标 → 关节控制执行。 |

[实机 §3](https://arxiv.org/html/2511.15200v2#S3)：G1 在两桌间重复放物、抓新物、转身，59 次尝试成功 54 次（约91.5%），周期20.2秒；不能把“54 cycles”写成54次无失败。泛化测试限于同类任务的位置、外观与物体变化；策略推理在外部 RTX 4090 工作站运行。

**训练、复用与新增**：与上表对应。任务训练依赖仿真数据；零真机策略微调仍包含手指系统辨识与相机标定，所采真机测量用于校准仿真。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Privileged RL Teacher | 作者用 PPO 训练任务策略，经验来自在线仿真；另采集200条仿真遥操作示范，用于抽取中间状态、重置机器人与场景（RSI），不是把200条示范直接当最终学生的全部训练集。论文未明确首轮参数初始化；[公开配置](https://github.com/NVlabs/GR00T-VisualSim2Real/blob/92bf0863d4a9b6ee29849736152b7769bd45c49c/gr00t/rl/config/exp/loco_manip/walk_stand_place_grasp_turn_homie.yaml)默认续训自有 Teacher checkpoint。 | 在 WBC 接口上组合阶段奖励、增量动作和示范状态重置，降低长任务探索难度。 |
| Vision-based Student | 作者以在线 DAgger＋BC 蒸馏，Teacher／Student 各控制一半仿真环境，由 Teacher 给动作标签。论文比较 DINOv3、ResNet等，未明确真机视觉型号与冻结策略；[公开默认配置](https://github.com/NVlabs/GR00T-VisualSim2Real/blob/92bf0863d4a9b6ee29849736152b7769bd45c49c/gr00t/rl/config/exp/loco_manip/wsdpt_student_for_teacher_v8q8.002_resnet_rgb_delay.yaml)为 ImageNet 预训练 ResNet18（参与微调）＋新建 MLP，不能与论文 DINOv3 实验混写。 | 混合教师／学生访问的状态，结合视觉域随机化、tiled rendering 与分布式蒸馏。 |
| Whole-Body Control（WBC） | 直接加载 HOMIE walk／stand 权重；[公开实现](https://github.com/NVlabs/GR00T-VisualSim2Real/blob/92bf0863d4a9b6ee29849736152b7769bd45c49c/gr00t/rl/trl/trainer/ppo_trainer_homie_api.py)在 opt_homie=False 时冻结参数，默认 Teacher／Student 配置均如此。底层已有仿真 RL 训练经历，本文任务训练不重训它。 | 将现成全身控制能力作为动作接口，并接入手指控制；手指动力学通过真机抓放测量做 SysID。 |

论文训练规模为 Teacher 16张、Student 64张 L40S；这描述作者实验配置，不是已证明的最低需求。可借鉴“任务策略＋现成 WBC”的接口与仿真数据生成方法；低算力重训能否达到原结果仍需实测。

## KPI — KPI: A Promptable Kernel for Physical Interaction on Humanoids

**重点关注**：False

**开源情况**：完全没开[2026-10-15]（指本文系统实现；[官方仓库](https://github.com/kpi-robot/kpi-robot.github.io)仅含论文、演示及网页源码，未提供控制器、agent、仿真或部署入口；未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-09-28（所读 v1，同日，正文 I–V 及补充 S1，另核查官网与网页仓库）；一作（共同）：Yikai Wang，未标共同一作；通讯：Yikai Wang、Lingfeng Sun；单位：Carnegie Mellon University、Autel US、Northeastern University；是否中稿：未核实（[作者主页](https://haozhang-thu.github.io/)列为 ICRA 2027 在投）；[项目主页](https://kpi-robot.github.io/)，[arxiv](https://arxiv.org/abs/2609.36151)。

**核心问题**：轨迹没有完整表达接触时的物理要求；作者让上层同时提供轨迹与交互契约，由快速控制环在线调整手臂阻抗、参考推进及前馈力。

**系统拆解**：输入语言、RGB-D、本体状态；agent 生成参考与 contract，KPI 控手臂、未修改的 SONIC 控腿腰。按[§III／图2](https://arxiv.org/html/2609.36151v1#S3)，保留 agent、kernel 与硬件实现的层级：

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Agentic Execution → Analyzer / Detector / Verifier | GPT-6 Astra 的角色调用 | 指令与观察 → Detector 在 RGB 定位、深度反投影；Analyzer 分阶段并编写参考与 contract；Verifier 检查运动学、碰撞及阶段结果 → 通过的计划或修订请求。 |
| Agentic Execution → Executor / motion cores | 共享参数化运动生成器、手指预设 | 已验证阶段 → 组合直线、圆弧、移动、身高等 cores → 名义参考送 KPI／SONIC，返回执行状态。 |
| KPI: Contract-Driven Physical Interaction | Interaction contract；有界在线优化 | 参考、实际末端姿态／速度、估计 wrench → 100 Hz 联合优化选定通道 → 刚度 Kp、阻尼 Kd、参考推进 ρ、前馈 wrench。 |
| Realization on a Humanoid Robot → Tunable task-space impedance / Damping decomposition / Sensorless wrench estimation | 笛卡尔阻抗；关节阻尼分解；准静态估力 | 参数与误差 → 500 Hz 手臂力矩；对角阻尼在电机驱动器、跨关节项在主机。关节力矩修正、扣重力、正则最小二乘 → 200 Hz 掌部 wrench 反馈。 |
| Realization on a Humanoid Robot → Whole-body control | 预训练 SONIC；遥操作旁路 GMR＋正运动学 | Agentic 用头／双腕三点目标，teleop 用 SMPL → SONIC 控腿腰；同源末端参考由 KPI 实现手臂接触响应。 |

[§IV／图4](https://arxiv.org/html/2609.36151v1#S4)：G1＋Dex3、机载 RGB-D、无腕力传感器；绞盘、开门通过、搬箱各5次，KPI 15/15 完成，SONIC 0/15、作者集成的 MCC* 1/15。图4指标是阶段进度，不能把基线20%进度当成功率。完整搬箱只用0.85 kg，1.85／3 kg 是另行静止抬举；写字为 scripted，抽屉及跑步／跪下搬箱为 teleop。样本小，未证明广泛场景泛化。

**训练、复用与新增**：本文没有重新训练任务策略或全身跟踪器；以下在线控制优化不等于学习训练。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Agentic Execution → Analyzer / Detector / Verifier | 调用 GPT-6 Astra，未报告微调或新增训练集，也未提供代码核实冻结；RGB-D／本体状态是推理输入。 | 角色分工、几何请求、contract 生成、执行前验证及阶段后修订。 |
| Agentic Execution → Executor / motion cores | 共享参数化生成器；未报告各 core 是单独训练的 RL policy，无任务专用示范训练。 | 组合同一动作空间完成不同任务。 |
| KPI: Contract-Driven Physical Interaction | 用当前误差／速度／估计力做局部有界优化，无离线训练集，也不识别环境刚度。 | Contract 将方向、响应要求、力区间、可调通道统一，闭环适应 gain／reference／feedforward。 |
| Realization on a Humanoid Robot → Tunable task-space impedance / Damping decomposition / Sensorless wrench estimation | 复用经典阻抗与 MCC 估力思路；本文未完整披露传动效率模型的标定数据／系数，不能代入 MCC 原文细节。 | 在 G1 现有关节接口实现任务空间阻抗，将局部阻尼下沉驱动器以减小延迟。 |
| Realization on a Humanoid Robot → Whole-body control | 直接复用未修改、未重训的 SONIC，遥操作复用 GMR；具体 SONIC checkpoint 未披露。 | 分出手臂命令，同时保留 SONIC 的腿腰跟踪与实际关节状态反馈。 |

**关键 Q&A（2026-10-15）**：Q：promptable 具体指什么？A：语言经 agent 生成结构化契约 σᵢ=(方向／子空间，tracking／compliant／constrained，力区间，可调通道)，并非学习出的 prompt token；三类响应均可附加力区间。上层决定阶段目标，KPI 每10 ms适应控制参数（[§III-A](https://arxiv.org/html/2609.36151v1#S3.SS1)）。Q：力区间是硬保证吗？A：不是；它约束局部预测，可达性受控制器边界限制。估力是准静态近似，冲击和强惯性交互不在所用局部模型范围，正文“预期误差小于5 N”也不是独立真值测量结果（[补充 S1](https://arxiv.org/html/2609.36151v1#S1a)）。

## ZEST — ZEST: Zero-shot Embodied Skill Transfer for Athletic Robot Control

**重点关注**：False

**开源情况**：完全没开[2026-10-15]（指本文系统实现；本次官方检索未找到系统代码、策略权重及训练／仿真／部署入口；[G1 数据记录](https://doi.org/10.5281/zenodo.21135719)已发布，元数据列有参考轨迹、指标与实机日志，Atlas 数据不含；本次未直接读取数据包，未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-01-30（所读 v1，同日，正文及附录 S1–S6／表S1–S7，对照 PDF；正式版仅核验出版信息）；一作（共同）：Jean-Pierre Sleiman，未标共同一作，前五名作者为按贡献排序的 core contributors；通讯：Farbod Farshidian；单位：RAI Institute、Boston Dynamics；是否中稿：是，[Science Robotics 2026](https://doi.org/10.1126/scirobotics.aec7695)，2026-07-17 接收、2026-08-12 出版（[出版商登记](https://api.crossref.org/works/10.1126/scirobotics.aec7695)）；未找到独立项目主页，[机构介绍](https://rai-inst.com/resources/in-the-press/zest-zero-shot-embodied-skill-transfer-for-athletic-robot-control/)，[arxiv](https://arxiv.org/abs/2602.00401)。

**核心问题**：用统一、单阶段的运动模仿 RL 配方，将动捕、单目视频和关键帧动画变为可直接迁移真机的动态、多接触控制策略，减少逐动作奖励与接触标注工程。

**系统拆解**：总体为参考数据 → 仿真训练 → zero-shot deployment（[PDF 图3／p.9](https://arxiv.org/pdf/2602.00401v1#page=9)，HTML 图号偏移）。部署输入当前本体观测、上一步动作和下一时刻参考，输出关节残差，经参考叠加及 PD 落到执行器；没有已实现的语言规划器或在线场景感知闭环。以下名称与层级依照原文 Materials and Methods：

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Systems and Modeling | PLA 的 Locally Projected／Dynamic Armature／Nominal Armature Model；Spot 执行器模型 | 硬件与闭链参数 → 等效转子惯量近似、功率／损耗建模 → 仿真模型及 PD 增益依据。 |
| Reference Motion Pipeline | MoCap／ViCap／Keyframe Animation；Kinematic Retargeting | 动捕、视频或动画 → 视频以 MegaSaM＋TRAM 重建；人类动作做时空优化重定向 → 目标机器人参考，无需接触标签。 |
| Training Setup and MDP Formulation → Actor and Critic Observations / Actions and Joint-Level Control | 非对称 Actor–Critic；残差动作；joint-level PD | Actor：IMU、关节位置／速度、上一步动作＋下一参考状态 → 残差 a → q_cmd=参考q＋Σa → PD；Critic 另见仿真真值／接触等，仅训练用。 |
| Training Setup and MDP Formulation → Adaptive Reference State Initialization | RSI、轨迹 bins、EMA failure、categorical sampler | rollout 失败／跟踪统计 → 难片段更常被选为初始状态，保留采样下限 → 训练更均衡。 |
| Training Setup and MDP Formulation → Assistive Wrench Automatic Curriculum | 基于模型的虚拟基座力／力矩 | 基座误差、名义动力学、bin 难度 → 提供训练辅助 → 随进步退至零；不部署真机。 |
| Hardware Deployment Pipeline | ONNX、配置元数据、MuJoCo 验证、真机控制 | checkpoint＋配置 → 导出／登记、仿真验证 → 直接部署 Actor＋参考＋PD。 |

[RESULTS／表1及 DISCUSSION](https://arxiv.org/html/2602.00401v1)：Atlas、G1、Spot 展示匍匐、滚翻、舞蹈、爬箱、后空翻等；实机每项用目标技能专用策略，多技能策略仅在仿真分析。表1只统计成功 trials 的误差，姿态误差不含 yaw，不能推出全部技能高成功率。G1 上／下箱各5/5是0.75 m箱高及接近名义初始位姿；部署无箱位姿感知，分布外存在失败。

**训练、复用与新增**：策略训练全部在仿真，但零真机微调不等于完全没有使用真机建模数据。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Systems and Modeling | 解析建模／参数辨识，非神经训练；静态参数来自厂商，Spot 效率参数用实机动态日志优化辨识；部分功率限制细节因专有信息未披露。 | 闭链惯量逐级近似及 Spot 动力模型，兼顾仿真速度与迁移精度。 |
| Reference Motion Pipeline | 自采 Xsens／Vicon 动捕、手机视频、目标机器人动画为离线参考；MegaSaM／TRAM 复用，未披露再训练／冻结／checkpoint；重定向为逐轨迹优化。多技能消融库15段。 | 组合相机／场景与人体重建，时空重定向同时处理尺度、时间和非穿透约束。 |
| Training Setup and MDP Formulation → Actor and Critic Observations / Actions and Joint-Level Control | Isaac Lab 单阶段 PPO、4096并行环境，以在线仿真经验训练 MLP；未报告已有策略 checkpoint 复用或教师蒸馏，初始化细节未披露。每个实机策略约单张L4、10小时；参考提供跟踪目标，非直接 BC。 | 通用跟踪／正则／存活奖励、适度域随机化、特权 Critic、残差动作；PD 按名义惯量定增益，仿真／真机一致（表S3–S7）。 |
| Training Setup and MDP Formulation → Adaptive Reference State Initialization | 不另训网络；利用 rollout 统计更新采样分布。 | 依据失败程度增采难动作／难阶段，减少长序列与多技能训练的薄弱片段。 |
| Training Setup and MDP Formulation → Assistive Wrench Automatic Curriculum | 解析辅助控制与同一难度统计联动，无额外学习策略；最终辅助为零。 | 自动退除支持，让大幅基座旋转等难动作更易探索和收敛。 |
| Hardware Deployment Pipeline | 导出后直接推理，无真机策略微调；Critic 与虚拟辅助不部署。 | 保持策略、配置、仿真验证及发布一致，降低人工部署差错。 |

**关键 Q&A（2026-10-15）**：Q：zero-shot 是否表示新动作／新机器人不用训练？A：不是；它指仿真策略直接迁移真机。跨机器人复用配方与结构，并非同一权重；实机仍逐技能训练，作者明确未测多技能策略对训练集外动作的泛化（DISCUSSION）。Q：与 skill library 的关系？A：可借鉴技能生产流程，但尚未证明通用、可组合的运动内核；视频在离线阶段生成参考，实机不靠视觉适应任意场景。长历史／未来窗口的负面消融也限于该网络、超参数和预算，不能泛化为历史信息无用。

## RelateAnything — RelateAnything: Real-Time Open-Vocabulary Relation Prediction From Any Inputs

**重点关注**：False

**开源情况**：未核实[2026-10-15]（机器人仿真／真机接入未核实：本文是图像关系感知组件，现有分级不宜直接标为可以sim。已核实[官方代码](https://github.com/Maelic/RelateAnything)、[预训练权重及 ONNX](https://huggingface.co/maelic/relsgg-vits16plus/tree/main)、[RA-4M 数据](https://huggingface.co/datasets/maelic/RA-4M/tree/main)、训练配置／入口和图像／摄像头推理入口；未运行。自重训需准备数据并取得 DINOv3 初始权重访问）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-09-11（v1，同日；阅读正文方法、评测定义、§6主要实验、推理开销与局限及附录 A–C 训练／数据部分，结合官方结果页核对，检查源码与模型／数据文件）；一作：Maëlic Neau（单作者）；通讯：Maëlic Neau；单位：Independent Researcher；是否中稿：未核实，官方资料目前按 arXiv 预印本引用；[项目主页](https://maelic.github.io/RelateAnythingProject/)，[arxiv](https://arxiv.org/abs/2609.12552v1)。用户提供的全小写 Demo 路径返回 404，[官方大小写路径](https://maelic.github.io/RelateAnythingProject/demo/)可访问。

**核心问题**：把关系预测从固定谓词分类、检测器类别标签和单数据集标注习惯中解耦：输入图像、任意来源的区域和用户指定关系词，输出有置信度的“主体—关系—客体”三元组。其价值是可插拔的关系感知接口；接到人形机器人前还需独立的空间记忆、动作规划、执行与验证机制，此为本项目的应用判断。

**系统拆解**：整体输入为单帧图像＋框（可选 mask）＋谓词字符串，输出区域索引间的关系分数／关系图。检测器／分割器在模型之外；物体类别名不进入前向网络，类别仅在训练损失中使用。系统不输出关节命令，也没有本文自带的机器人执行器链路。[§3、Fig.2](https://arxiv.org/html/2609.12552v1#S3)，[官方架构](https://github.com/Maelic/RelateAnything/blob/main/docs/architecture.md)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| RelateAnything → Visual path | DINOv3；多层特征融合；SoftSpatialPool | 图像与区域 → 三层 patch 特征归一化融合，框坐标查询整幅特征图 → 主体、客体、union 和 contact 区域特征 |
| RelateAnything → Pair sampling | 几何预筛；learned relatedness | N(N−1) 个有向区域对 → 几何阶段保留 400 对、相关性阶段保留 128 对 → 候选对及 pair logit |
| RelateAnything → Pair representation | 几何编码；RelationTransformer；joint self-attention；DeformableRelRead | 四种区域特征＋19维几何量 → 形成512维表示，结合其他关系对和场景、在四个锚点周围稀疏读取图像 → 上下文化的关系对特征 |
| RelateAnything → Text path | PredicateTextStudent；固定 embedding bank | 用户谓词字符串 → 仅在换词表时编码为512维单位向量 → 可替换的谓词矩阵；逐帧无需运行语言模型 |
| RelateAnything → Two branches and a predicate-conditioned gate | spatial／semantic 两支；谓词门控；VocabHead | 对特征分别生成空间／语义查询，与文本求余弦相似度；仅由谓词 embedding 决定两支权重 → 结合 pair logit 并校准得到分数，可分别解码空间／语义图 |

**训练、复用与新增**：本文需要训练。主关系模型以 RA-4M、raw Visual Genome 和 HICO-DET train 混合监督训练；论文配方按图像采样占比约72.7%／6.3%／21.0%，不能把 HICO 的“5%关系占比”误读为只用了5%训练图像。RA-4M 是借用 MegaSG 图像／框、由 Gemma-4-26B-A4B-it 三阶段流程（semantic、grow、spatial）生成关系，再按确定性几何规则处理；论文报告约47.4万图像、428万关系、10,102谓词，最终训练词表19,103词包含其他语料的词汇，词汇加入不等于使用其图像关系训练。当前发布包与论文统计存在版本差异。标准配方12 epoch、4×A100、每卡32图像，主干学习率5e-5、头4e-4。几何核验只检查可约束的关系，不能证明所有动作语义正确。[§4、附录A/C](https://arxiv.org/html/2609.12552v1#S4)，[训练说明](https://github.com/Maelic/RelateAnything/blob/main/docs/training.md)，[数据生成说明](https://github.com/Maelic/RelateAnything/blob/main/datagen/README.md)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| RelateAnything → Visual path | 复用 `facebook/dinov3-vits16plus-pretrain-lvd1689m` 初始化并全量微调；pooling／融合随关系模型训练，主干没有冻结 | 用整图特征和坐标查询读取区域，避免依赖物体类别；多层融合及对象—文本辅助对齐 |
| RelateAnything → Pair sampling | 几何预筛和相关性头均训练；正例来自已标注关系对，未标注对作降权负例；物体类别只用于估计损失权重 | 限定计算预算，并把 pair 相关性与谓词身份分开建模；相关性也会学到“哪些对容易被标注”的偏差 |
| RelateAnything → Pair representation | 新增关系模块随主模型训练；几何特征计算本身不是训练；deformable read 的残差门初始为0 | 显式保留主体／客体／union／contact与几何信息，并读取框外上下文 |
| RelateAnything → Text path | 本文训练6层学生，蒸馏冻结的 dino.txt 教师；公开训练历史记录 CLIP token embedding 投影初始化；训练完成后学生和词向量用于固定目标，不随关系头训练更新。数据为关系短语、同义／反义词对和空间组合短语 | 通过反义排斥、同义保持与邻域保持等目标分开 above/below 等反义谓词；不是直接拿现成CLIP文本空间打分。[学生实现](https://github.com/Maelic/RelateAnything/blob/main/relsgg/text/student.py)、[训练历史](https://huggingface.co/datasets/maelic/OV-SGG-Bench/blob/main/text_student_v2_512/student.history.json) |
| RelateAnything → Two branches and a predicate-conditioned gate | 新增视觉投影与门控随模型训练，门控无单独标签；batch-local InfoNCE 将同义词组作正例，并对可能同时成立的未标注谓词降权，另有方向交换、sigmoid、背景抑制等辅助项；文本矩阵不经学习投影 | 让几何与语义证据按谓词自动混合，允许替换词表；将遗漏标注视为未知，而非一律负例。[损失与分数实现](https://github.com/Maelic/RelateAnything/blob/main/docs/architecture.md) |

主要证据：用真值框、每有向对限一个谓词的跨数据集评测，VG150／PSG／IndoorVG 的 mR@50 为28.2／30.6／29.5，对照 OvSGTR 为10.4／8.8／12.8；这些是关系预测结果，不是机器人任务成功率。[官方结果与评测条件](https://github.com/Maelic/RelateAnything/blob/main/docs/results.md)。SpatialSense 宏AUC为0.690，但分类准确率62.5%低于原基准仅框基线68.8%，作者明确认为这尚不足证明视觉空间推理能力（不同指标不能混比，[§6.6](https://arxiv.org/html/2609.12552v1#S6.SS6)）。主表跨数据集不等于全部关系概念未见；已发布模型的 HICO 结果包含HICO训练份额，真正零样本对照另外训练。作者也承认缺少 RA-4M 人工抽检精度、若干结论仅单随机种子；图生成的假阳性和检测漏检仍限制应用。建议作为关系感知候选先测机器人视角下的方向、接触与遮挡变化，不据此认定已有可靠三维世界模型。[§10](https://arxiv.org/html/2609.12552v1#S10)。

## Spatial-TTT — Spatial-TTT: Streaming Visual-based Spatial Intelligence with Test-Time Training

**重点关注**：False

**开源情况**：缺少关键组件[2026-10-15]（已公开模型实现、训练／VSI-Bench 评测入口、nano 权重及部分数据；完整论文模型、全量空间 QA 和稠密场景描述数据仍列为待发布，97k 数据仓当前仅含 JSONL，视频需另备；属于视频空间问答模型，不据此判定机器人仿真可用；未运行；[官方代码](https://github.com/THU-SI/Spatial-TTT)，[nano 权重](https://huggingface.co/THU-SI/Spatial-TTT-nano/tree/main)）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-03-12（所读 v1，2026-03-12；正文、图2、实验表1–5、PDF 附录 A–B／算法1，并核对公开代码与资源）；一作（共同）：Fangfu Liu、Diankun Wu、Jiawei Chi；通讯：Yongming Rao、Yueqi Duan；单位：清华大学、腾讯混元、NTU（按原文简称）；是否中稿：是，[ECCV 2026 官方页面](https://eccv.ecva.net/virtual/2026/poster/5426)；[项目主页](https://liuff19.github.io/Spatial-TTT/)，[arxiv](https://arxiv.org/abs/2603.12255)。用户提供的项目路径大小写已按官方链接修正。

**核心问题**：长视频中的空间证据分散在不同视角和时刻；作者在预训练 VLM 中加入随当前序列更新的快权重记忆，并通过稠密场景描述训练它保留空间信息，再用于视频空间问答。

**系统拆解**：输入 RGB 视频／多视角图像和语言问题，经视觉编码、混合解码器输出文本答案；没有 motor skill、执行器或机器人控制闭环。3∶1 指每四个解码器注意力层中三个改为 TTT 层，一个保留全注意力；TTT 层内部的 SWA 与 TTT 分支并行，不是多个独立 agent。[§3.2／图2](https://arxiv.org/html/2603.12255v1#S3.S2)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Vision Transformer | Qwen3-VL 视觉编码与跨模态映射 | 视频帧 → 视觉 token；问题经 tokenizer，二者送入解码器。 |
| Hybrid TTT Architecture → Self-Attention Anchor Layers | 25% 层保留标准因果 self-attention，及原 FFN | 图文 token → 全上下文注意力 → 语义／推理特征。 |
| Hybrid TTT Architecture → TTT Layers → SWA | Sliding Window Attention，与 TTT 分支共享 Q/K/V 投影 | 当前与窗口内历史 token → 局部因果注意力 → 当前块内的时空交互特征。 |
| Hybrid TTT Architecture → TTT Layers → TTT Branch | Spatial-Predictive Mechanism；SwiGLU-MLP fast weights；Muon 更新 | 视觉 Q/K/V 按时间×高×宽重排、逐通道 3×3×3 卷积 → 用先前快权重读取当前 chunk，再由该 chunk 的 K/V 自监督更新记忆 → 与 SWA 输出融合；块长 2648。3D 卷积作用于视频 token 网格，不直接构造世界坐标地图。[算法1](https://arxiv.org/pdf/2603.12255v1#page=17) |

**训练、复用与新增**：这是“离线监督训练＋测试序列内自监督快权重更新”，不是免训练。离线从 Qwen3-VL-2B-Instruct 初始化：阶段1约16k条 SceneVerse 场景图衍生的稠密描述（ScanNet 3.6k、ARKitScenes 12.5k），32帧，前两轮把 SWA 窗口从5600退火到2648；阶段2用空间 QA 继续微调，64→128帧、窗口固定2648。§4.1／附录A.1给出约3M QA（2.5M公开的 VSI-590K、VLM-3R、InternSpatial、ViCA，0.5M依 ScanNet 场景几何与语义标注生成）；§3.4却写2M，规模存在原文不一致。几何、OBB和房间面积用于构造监督，不是推理时输入或在线 RL 经验。[§3.3–4.1](https://arxiv.org/html/2603.12255v1#S3.S3)、[附录A.1](https://arxiv.org/pdf/2603.12255v1#page=16)

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Vision Transformer | 复用 Qwen3-VL-2B-Instruct；公开97k训练脚本冻结 vision 和 merger，论文完整两阶段未逐项明确冻结清单，不能直接等同。[公开配置](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/spatial_ttt_train.sh#L78) | 沿用视觉输入前端，新增重点在解码器和训练监督。 |
| Hybrid TTT Architecture → Self-Attention Anchor Layers | 继承预训练语言模型参数；论文 backbone 学习率1e−6，公开脚本训练整个 language model 与 lm_head，非仅训练 TTT。使用上述描述／QA监督；推理不更新这些慢权重。 | 以3∶1保留 anchor，避免完全替换注意力破坏语义能力。 |
| Hybrid TTT Architecture → TTT Layers → SWA | 复用并训练原注意力 Q/K/V；共享前述监督，SWA 本身不是另一份独立模型／数据集。推理仅维护滑窗 KV，不做此分支权重优化。 | 大块 TTT 下补充块内因果交互；窗口退火渐进转移跨块记忆责任。 |
| Hybrid TTT Architecture → TTT Layers → TTT Branch | 新增快权重初值、更新／门控参数、Q/K scale-shift与卷积共同离线学习，新增参数学习率1e−5；代码快权重为随机初始化后训练，卷积 Dirac 初始化。测试时复制学好的快权重初值，以当前输入 K/V 为自监督信号执行块级 Muon 更新，无需真实答案，也不重训全部 VLM；每次公开 generate 调用重新建立状态。[初始化](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/models/causal_swa_lact.py#L327)、[推理状态](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/models/spatial_ttt.py#L183) | 在 LaCT／TTT 思路上加入空间预测卷积，并用稠密描述和渐进训练学习“应记住什么”；fast weights 是非线性 MLP 隐式记忆。 |

“空间线性记忆”不应理解为显式3D地图或无限视频恒定内存：记忆是 SwiGLU-MLP 参数。TTT 层有固定滑窗和待更新缓存，但25% anchor仍访问完整上下文，公开实现保留其 DynamicCache；因此“整个模型严格线性／恒定显存”不成立。表5在352×480、1024帧时报告11.9GB／799.4TFLOPs，Qwen3-VL-2B为21.2GB／1403.1TFLOPs，证明该范围内更省资源，不能外推无限流式。[§3.2、§3.4、表5](https://arxiv.org/html/2603.12255v1#S4.S4)、[anchor缓存代码](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/models/spatial_ttt.py#L208)

作者报告 VSI-Bench 宏平均64.4（原Qwen3-VL-2B为50.3）、MindCube-Tiny准确率76.2；VSI-SUPER按1fps测10–120分钟，120分钟Recall准确率30.0、Count为38.4，并非长期可靠记忆已解决；部分基线OOM被计0，不能当作正常推理错误。VSI提升同时包含额外监督数据效应，消融移除3D卷积／稠密数据／anchor分别降至62.1／61.3／53.9。对本课题可作为空间记忆候选，尚无机器人任务闭环收益证据，也不能把nano公开权重视为复现了论文全量模型结果。[表1–4](https://arxiv.org/html/2603.12255v1#S4)

## Weave — Learning Whole-Body Dexterous Loco-Manipulation from Human–Object Interactions

**重点关注**：False

**开源情况**：可以sim[2026-10-15]（[官方仓库](https://github.com/xiaohu-art/Weave)已提供 Isaac Sim 5.1／IsaacLab 环境、G1＋Inspire 资产及训练／评估／推理入口，[数据](https://huggingface.co/datasets/appolyn/Weave)含参考轨迹与物体资产；[v0.2](https://github.com/xiaohu-art/Weave/releases/tag/v0.2)实际含 `model_31400.pt` 及训练配置，但这是在 D_test 训练的模型，不能据此复现主实验的未见序列泛化；未找到真机部署入口；未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-09-15（v1，同日；已读正文方法、实验与限制，未见独立训练附录，另核验代码与权重附配置）；一作：Liu Cao，共同一作未披露；通讯：未披露；单位：清华大学交叉信息研究院、清华大学人工智能学院、大连理工大学、香港中文大学，作者的 † 表示部分工作在 CocoMatrix 完成，不是共同一作标记；是否中稿：未核实；[项目主页](https://xiaohu-art.github.io/Weave/)，[arxiv](https://arxiv.org/abs/2609.16683v1)。

**核心问题**：人体动作重定向后，手指“碰到物体”仍不等于能稳定搬运；作者把接触和物体几何同时纳入参考构建与统一 RL 策略，让人形机器人协调行走、平衡和多指抓取。

**系统拆解**：两级流程为 Interaction-Preserving Reference Construction（下称“参考构建”）→ Contact-Rich Skill Acquisition via Reference Tracking（下称“参考跟踪技能学习”）。离线输入是人体 SMPL-X 动作、物体轨迹和几何；执行时输入本体状态、真值物体状态／几何与短时域参考，单一策略输出 29 个身体＋12 个手指驱动关节的位置目标，交给 PD 执行，未接入视觉语言高层。[§IV、图2](https://arxiv.org/html/2609.16683v1#S4)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| 参考构建 → Whole-body retargeting | GMR-style whole-body IK | 人体／物体动作 → 关节与身体对齐、地面约束及平滑惩罚 → 初始机器人交互轨迹 |
| 参考构建 → Contact-aware hand refinement | 接触吸引、force-closure、穿透与正则项 | 初始轨迹＋手指接触 → 固定骨盆和腿，联合优化双臂与手指 → 更可抓稳的交互参考 |
| 参考构建 → Approach motion completion | Kimodo、轨迹拼接 | 交互起始姿态／朝向约束 → 生成不同接近方向的行走前缀 → 接近、抓取、搬运的完整参考 |
| 参考构建 → Reference annotations | 距物体表面的三值接触标签 | 完整参考 → 标注应接触／中性／应分离 → RL 初始化、短时域指令与奖励依据 |
| 参考跟踪技能学习 | 分组编码器、SimBaV2 非对称 actor–critic、PPO；输出接 PD | 本体状态＋物体真值位姿、指尖表面向量、接触标记、BPS-SDF＋短时域参考 → 统一策略 → 身体和手指位置目标；50 Hz 控制、200 Hz 仿真，欠驱动手的其余关节按 mimic 耦合 |

主实验在 9 个已见物体上，训练序列成功率 92.5%，相同物体的未见交互序列为 65.0%；不是未见物体或真机泛化。另一实验的 95.3% 对 91.5% 是**都在 D_test 训练**后的统一策略与九个单物体策略比较，训练总预算分别为 27k 与 9×3k，不能替代前述 65.0%。当前评估代码将未触发失败且到达片段终点计为成功。[§V-B/C](https://arxiv.org/html/2609.16683v1#S5.SS2)，[eval.py](https://github.com/xiaohu-art/Weave/blob/d839eb298358b71aedd0407c7ba9bbb05c14e05e/scripts/rsl_rl/eval.py#L121)

**训练、复用与新增**：

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| 参考构建 → Whole-body retargeting | 使用 OMOMO 人物交互数据；逐轨迹 IK 优化，不是训练神经策略 | 调整骨盆水平约束与地面高度处理，生成符合机器人构型的参考 |
| 参考构建 → Contact-aware hand refinement | 对交互参考逐实例优化，不是 RL；沿用已有接触优化／抓取质量思想 | 在人体到机器人转移时同时约束手指接触、抗扰抓握质量和穿透 |
| 参考构建 → Approach motion completion | 复用 Kimodo；本文所用 checkpoint、是否微调与冻结未披露，不能把 Kimodo 原始训练当作本文训练 | 补齐数据缺失的接近阶段；训练／测试每条原始交互分别生成 3／5 个方向变体 |
| 参考构建 → Reference annotations | 几何距离规则，无学习参数 | 将正负接触监督随参考交给统一策略 |
| 参考跟踪技能学习 | 作者训练统一策略；公开训练入口新建网络，checkpoint 附配置为 `resume: false`，未见预训练控制器微调或冻结子网设置。PPO 在线仿真经验＋9物体离线参考：D_train 7,869条／19.56h，D_test 1,605条／3.67h。SimBaV2 二维权重用 Muon，其余参数用 AdamW；采用参考初始化、提前终止及动力学随机化 | 将机器人／物体跟踪与手指对向抓取、接触匹配奖励联合优化，并在多个物体和动作间共享同一策略；PD 与手指 mimic 耦合直接使用配置 |

参考与执行轨迹须分开：[当前数据卡](https://huggingface.co/datasets/appolyn/Weave)列参考 9,474 条／23.2h、仿真 rollout 8,699 条／21.2h；不能把摘要“约23小时”写成已发布真机数据。复现需显式核对 split：当前 [`train.yaml`](https://github.com/xiaohu-art/Weave/blob/d839eb298358b71aedd0407c7ba9bbb05c14e05e/configs/track/train.yaml)默认读取 D_test、4096 环境，[作者答复](https://github.com/xiaohu-art/Weave/issues/2#issuecomment-5869091444)称实验使用 8×16384 并行环境，GPU 型号与耗时未披露。作者另提供[初版重定向脚本](https://github.com/xiaohu-art/OMOMO2Isaac/blob/force/scripts/humanoid_retarget.py)，完整 Kimodo 接近动作生成链未核实。对于本课题，可将其视为低层交互技能和仿真数据来源候选；感知、参考选择／组合及真机接触迁移仍需另外实现，尚不能视为完整 VLA 或自主 skill library 系统。[§VI](https://arxiv.org/html/2609.16683v1#S6)

## Ego-VCP — Ego-Vision World Model for Humanoid Contact Planning

**重点关注**：False

**开源情况**：可以sim[2026-10-16]（已核实[官方仓库](https://github.com/HybridRobotics/Ego-VCP)提供 Isaac Lab 环境与 G1 资产、低层 policy.pt／onnx、三任务及联合 world_model.pt、数据采集／世界模型训练／MPC 仿真入口；[离线数据](https://huggingface.co/datasets/Hang917/EgoVCP_Dataset/tree/main)含 ball／wall／tunnel 的实际 npz 文件。未找到完整真机通信、相机与部署入口；论文真机演示不等于部署代码公开；未运行）。

**基本信息**：阅读日期 2026-10-16；arxiv上传：2025-10-13（所读最新 v2：2026-03-08；阅读完整方法、实验与实现代码，8页论文无独立训练附录）；一作：Hang Liu，未标共同一作；通讯：论文未明确标注；单位：University of Michigan, Ann Arbor、University of California, Berkeley、The Chinese University of Hong Kong（按v2署名）；是否中稿：是，ICRA 2026，已核[官方会议程序 WeI1I.268](https://ras.papercept.net/conferences/conferences/ICRA26/program/ICRA26_ContentListWeb_4.html)；[项目主页](https://ego-vcp.github.io/)，[arxiv](https://arxiv.org/abs/2510.11682v2)。

**核心问题**：在接触稀疏、视觉部分可观测的条件下，用随机交互离线数据训练潜空间世界模型，再在线比较候选动作后果，决定扶墙、挡球或降低身体通过拱门。它提供了真正“预测候选动作后果再选动作”的控制例子，但尚不是语言驱动的通用技能库／任务harness。

**系统拆解**：当前64×48自我视角深度＋本体感知 → 推断潜状态 → 在世界模型中滚动模拟动作序列并评分 → MPC只执行所选序列首个动作，再接收真实反馈重规划。规划动作是双手末端目标与体高，低层策略将其落实为关节目标；**真机底盘速度由人操作，不在本文高层规划动作中**。[§III、Fig.2–3、§IV-E](https://arxiv.org/html/2510.11682v2#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Low-Level Controller | 本体感知策略；PPO；关节目标跟踪 | 角速度、重力投影、关节状态及速度／末端位置／体高命令 → 控制策略 → 关节位置目标，由底层执行器跟踪 |
| Ego-Vision Humanoid World Model → observation encoder／decoder | 图像CNN、本体MLP；随机潜变量 z | 深度、本体状态及动态记忆 h → 后验编码 → z；训练时由(h,z)重建观测，促使潜状态保留任务信息 |
| Ego-Vision Humanoid World Model → recurrent dynamics／latent prior | GRU/RNN；预测先验 | 上一步(h,z,a) → 更新动态记忆h；未来无图像时从h预测z → 供候选轨迹在潜空间展开 |
| Ego-Vision Humanoid World Model → surrogate value／termination heads | Q(h,z,a)；D(h,z) | 当前或预测潜状态＋候选动作 → 估计回报与失败概率 → MPC评分；预测终止概率超过0.9时后续价值置零 |
| Value-Guided Sampling MPC | Cross-Entropy Method（CEM）；滚动时域执行 | 候选动作序列 → 世界模型预测、平均多个时刻Q值、CEM筛选 → 当前动作；论文默认1024条候选、4步时域，真机25Hz |

**训练、复用与新增**：“无示范”指世界模型数据由已经训练好的低层控制器执行平滑随机高层命令产生，不是机器人无需训练。三类仿真环境采集(o,a,r,d)，归一化动作空间用增量随机游走（步幅0.32）；世界模型训练离线完成，低层PPO仍需在线仿真经验。其灵感来自 Dreamer／JEPA，但没有复用两者预训练权重。[§III-A–C](https://arxiv.org/html/2510.11682v2#S3)、[采集与训练入口](https://github.com/HybridRobotics/Ego-VCP/tree/main/ego_vcp/scripts)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Low-Level Controller | 本文说明仿真PPO训练，非遥操作示范；公开训练入口默认不恢复checkpoint。采集／规划阶段直接加载发布的低层policy，不随世界模型更新；原始论文低层完整训练成本未量化 | 将可追踪的速度、双手目标与体高作为高层接口，避免世界模型直接搜索全部关节控制 |
| Ego-Vision Humanoid World Model → observation encoder／decoder | 新建CNN／MLP并训练，公开train_wm.py从新模型开始，未加载视觉基础模型；离线深度和本体序列以观测重建NLL监督；训练参数统一优化，无冻结预训练视觉塔 | 同时压缩视觉和自身状态，用任务相关潜变量避免规划时逐像素生成未来图像 |
| Ego-Vision Humanoid World Model → recurrent dynamics／latent prior | 与编码器联合从头训练；离线状态动作序列监督；带stop-gradient的双向KL使预测先验与观测后验一致。代码配置h=256、z=32；stop-gradient不是整个模块永久冻结 | 用记忆h处理部分可观测性，未来由先验滚动而不依赖不存在的真实观测 |
| Ego-Vision Humanoid World Model → surrogate value／termination heads | 与世界模型训练；终止标签作BCE，Q回归离线轨迹的Monte Carlo回报，非专家动作标签，也不是在线TD策略优化；代码用折扣0.99并截断终止后回报 | 学到较密集的价值评分，替代直接累加难预测的稀疏接触奖励；Q只是近似启发式，作者不声称其等于最优Q |
| Value-Guided Sampling MPC | 无单独可训练策略权重；推理时固定世界模型，在每个控制周期优化候选序列。CEM是在线搜索，不是模型微调；当前发布配置默认2000候选，与论文1024应区分 | 对多时刻Q取均值以缓解高方差估计，结合短时域和反馈重规划；该目标与标准最优控制目标一般不等价 |

实验边界：定量对比与消融均在仿真，三个任务以各自reward报告，统计跨3种随机种子与10次独立试验；不能把reward换算成成功率。§IV-A报告0.5M离线转移可完成三任务，但没有把低层PPO训练成本计入该高层数据优势，也未比较SAC等off-policy方法。Table I中4步对挡球／过拱最好，TD-MPC在扶墙reward上反而更高；所以结论是这组视觉接触任务的综合取舍，并非所有任务胜出。G1＋D435i真机展示扶墙、挡未见箱子、过拱与任务衔接，未给统一真机成功率；长期空间地图、任意目标指定及自主速度规划仍需另补。[§IV、Table I](https://arxiv.org/html/2510.11682v2#S4)。

## WAP — World Action Planner: Generalizable Robot Decision-Making with Action-Conditioned World Models

**重点关注**：False

**开源情况**：缺少关键组件[2026-10-16]（[仓库](https://github.com/XiangchengZhang/world-action-planner)已开世界模型训练／推理、WebSocket 服务、LIBERO 手工动作想象 demo；[权重](https://huggingface.co/XiangchengZhang/world-action-planner/tree/main)含仿真世界模型、diffusion policy 与逆动力学模型。未见完整 VLM 提案／评价／排序、全局—局部搜索及整套评测／真机部署入口，真机适配权重未见；训练数据路径须自行配置；未运行）。

**基本信息**：阅读日期 2026-10-16；arXiv 首次上传 2026-07-30，所读最新 v2（2026-10-02）；读正文方法／实验、附录 C–E 的实现／训练／实验关键段落，理论结论与假设已读、证明未逐行复核。一作（共同）：Xiangcheng Zhang、Runhan Huang；通讯：未披露；单位：Harvard University；是否中稿：未核实（原文标 Preprint）；[项目主页](https://worldactionplanner.github.io/)，[arXiv](https://arxiv.org/abs/2607.27599)。

**核心问题**：VLM 能提出操作目标，但难预判碰撞和局部策略能否接手；作者把低层动作转成机器人骨架图，以动作条件视频世界模型想象执行，再用 VLM 修正和筛选目标姿态。

**系统拆解**：输入语言任务、多视角图像／历史和机器人状态；VLM 提出目标→控制器生成动作→世界模型预测视频→全局修正与局部搜索→真实执行、观察、在子任务起点重规划。输出机器人动作，不是仅生成视频。下表按正文 §3 与附录 C 的组件组织；Optional Policy 和 Robot Controller 均属于 World Action Planner，不能把它们合称一个端到端策略。[原文 §3、算法1、附录C](https://arxiv.org/html/2607.27599v2)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Pose-Image Conditioning for Action-Controlled Robot World Modeling | 机器人骨架渲染、视频 VAE、DiT | 当前状态＋候选动作→仅用机器人模型计算未来关节并渲染骨架；骨架与历史图像条件化视频模型→预测环境交互视频。无未来真值关节／物体状态输入，骨架并非准确物理预测。 |
| World Action Planner → Action Proposals | VLM agent | 指令＋多视角观察→MOVE／ROTATE／GRASP／RELEASE；多视角像素点三角化成 3D 目标，依赖相机标定。 |
| World Action Planner → Optional Policy | Diffusion Transformer | 合适的初始状态＋任务→局部操作动作；候选目标后继续想象该策略，以检查能否接手。 |
| World Action Planner → Robot Controller | 逆动力学／底层控制器 | 当前与目标末端姿态→动作块；真机按 10 mm 路点调用机器人控制器。 |
| World Action Planner → Global Trajectory Optimization | VLM 反馈优化 | 候选动作的想象视频→判断碰撞等问题、提出目标位置方向修正→新动作。 |
| World Action Planner → Local Search and Refinement | GridSearch、VLM 排序 | 目标位置邻域及偏航角 ±45° 候选→分别想象并排序→选中动作执行；搜索的是目标空间。 |

**训练、复用与新增**：世界模型、可选操作策略和仿真控制器有独立训练；推理搜索不等于训练。新任务／真机实验另有世界模型适配，不能读成整套零样本。训练依据为附录 C、D.1–D.2；代码冻结情况来自公开 `world_model/algorithms/wan/wan_t2v.py`。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Pose-Image Conditioning for Action-Controlled Robot World Modeling | 基于 Wan-T2V-1.3B（代码 Wan2.1-T2V-1.3B）训练 DiT；代码冻结 VAE／文本编码器。LIBERO-90 基础训练每任务 100 轨迹＝40 示范＋40 σ=.08 噪声重放＋20 σ=.16 重放，10k 步、batch64、16 H100。新布局以原布局示范及额外 10 条噪声轨迹适配；两项新任务用 50 条 VLM 探索轨迹训练 10 epochs；真机用 10 条失败探索、100 步适配。 | 用机器人模型而非场景模拟器把动作映射成姿态图；多视角拼图预测，flow matching 仅监督视频 token。新任务不需要专家示范，不等于不需要数据。 |
| World Action Planner → Action Proposals | 直接调用 Gemini 3.0 Flash；未报告本文微调。推理时输入观察、指令与提示模板。 | VLM 生成可执行目标／原语，作为后续想象搜索的起点。 |
| World Action Planner → Optional Policy | 作者训练 12 层、8 头、512 维 diffusion transformer，初始化／冻结范围未披露；LIBERO-90 每任务 30 条专家示范，预测 40 动作、执行 10。新布局用原布局每任务 5 条示范继续训练；无此策略的新任务／真机不能归入同一训练设置。 | 将局部操作能力作为可选先验，世界模型检验后续策略 rollout。 |
| World Action Planner → Robot Controller | 仿真用 UNet diffusion inverse dynamics，以完全随机轨迹训练，无任务信息；规模／初始化未披露。真机复用 Franka 底层控制，无神经控制器训练。 | 目标姿态与低层动作分离；真机抓放还含手写升降／开合原语。 |
| World Action Planner → Global Trajectory Optimization | 无单独训练模块；复用 VLM 与世界模型，以推理反馈逐次修改候选。 | 从想象轨迹诊断问题，作方向性目标修正。 |
| World Action Planner → Local Search and Refinement | 无单独训练模块；邻域枚举、世界模型 rollout、VLM 排序均为推理；原文披露搜索尺度及提示。 | 低维目标网格搜索，可叠加可选策略 rollout。 |

作者表3报告组合任务／新布局／新任务／真机成功率 72.0／82.0／78.0／65.0%，对应 5／6／2／2 个所选任务，不能当成全套基准或跨本体部署成绩。真机是 Franka Panda、双外部相机、绿色与紫色易拉罐各 20 次，世界模型只在绿色罐探索数据适配；不是人形全身控制。不同机器人上的额外结果主要验证世界模型预测。A100 上预测 20 帧约 17 秒，全局优化约 30 秒、局部搜索约 2–3 分钟；真机只用全局优化，不能据此推定实时闭环性能。对本课题的启发是“用预测检验技能起点／后果”；是否能支撑人形控制及规划时延仍待验证。[表3、附录D.2/E](https://arxiv.org/html/2607.27599v2)

## Thea — Towards the Harness of Embodied Agents

**重点关注**：False

**开源情况**：缺少关键组件[2026-10-16]（[仓库](https://github.com/EIT-HAI/Thea)已给 harness、模型接口、评估与仿真适配代码，但[仿真说明](https://github.com/EIT-HAI/Thea/blob/main/simulation/README.md)要求自行提供 primitive policy、安装基准及实现 factory；[真机模板](https://github.com/EIT-HAI/Thea/blob/main/examples/port-template/factory.py)仍留 SDK、感知、策略等实现空位。未见论文策略权重及其训练数据／入口；不能据终端模式或适配器判定机器人系统可直接复现；未运行）。

**基本信息**：阅读日期2026-10-16；arXiv首次上传2026-08-03，所读为同日v1，当前仅此版；已读正文方法、实验与附录工具注册、概念对照和执行日志，核查官方源码。首页“Technical Report, July 2026”不是上传日期。一作Qi Wang；共同一作、通讯未披露，†明确表示Core contributors，不等同共同一作；单位Eastern Institute of Technology, Ningbo（宁波东方理工大学）；是否中稿未核实。[项目主页](https://eit-hai.github.io/thea/index.html)，[arXiv](https://arxiv.org/abs/2608.11246)。

**核心问题**：怎样让通用模型可靠调度已有机器人能力？Thea把运动能力包装为工具，再用持久场景图与独立动作评估形成“观察—调用—验证—恢复”循环；新增重点是运行框架，并非新的全身控制策略。

**系统拆解**：输入用户任务、当前图像／状态和机器人能力，输出串行工具调用，由工具后的SDK、规划器或学习策略驱动执行器；动作后重新观察并评估。以下保留[§3–4](https://arxiv.org/html/2608.11246v1)的章节层级。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| System Architecture → Agentic Loop | 单步VLM决策、执行、反馈 | 当前上下文 → 每轮至多一次工具调用 → 执行后重决策；无调用则结束任务。 |
| System Architecture → Context Engineering | Resident／Refreshed／Accumulated | 常驻知识、最新观测、执行历史 → 分类组装与压缩 → 模型上下文。 |
| System Architecture → Tool Protocol | MCP schema、BuiltinTool、pre/post hooks | 名称和参数 → 安全检查及后端执行 → 结构化结果；POST_CONDITION仅供评估器。 |
| System Architecture → Skills | SKILL.md、load_skill | 元数据常驻，正文按需加载 → 提供操作建议／约定；本身不执行动作。 |
| System Architecture → Memory | Task Notes、MEMORY.md、tool_experience | 任务记录与客观反馈 → 结束时整理候选经验 → 跨任务文本记忆。 |
| System Architecture → Safety | clearance、执行约束、失败预算 | 新鲜四向净空及动作 → 拒绝／限制底盘移动，超预算求助 → 约束执行。 |
| System Architecture → User Interaction | query_user、notify_user | 不确定性／进度 → 阻塞询问或非阻塞通知 → 用户反馈。 |
| Bridging the Gaps → Scene Graph as Context | Persistent World State、引用查询 | 感知与已确认执行结果 → 更新稳定物体ID、粗3D框及on/inside/holding等关系 → 可查询世界状态。 |
| Bridging the Gaps → Evaluation as Exit Codes | 独立Evaluator、分段停止机制 | 当前同步观测＋动作后置条件 → 判断process/success/failure及原因 → 继续、停止或恢复。 |
| Bridging the Gaps → Embodiment Profile | Operational Envelope、Perception Configuration、Base-Relative Positions | 身体尺寸、可达域、相机／夹爪配置 → 常驻描述 → 为决策提供具身约束。 |

**训练、复用与新增**：模型推理、文档更新与策略训练须分别理解；本文没有训练新的高层决策模型，但不能因此称整个机器人系统“无需训练”。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| System Architecture → Agentic Loop | 实验复用GPT-5.5，reasoning effort=high；未报告本文微调，预训练数据不属于本文数据。 | 用执行事实逐轮校正决策。 |
| System Architecture → Context Engineering | 无参数训练；消费实时观测、工具结果及已有文档。 | 区分更新频率，避免历史图像替代当前状态。 |
| System Architecture → Tool Protocol | 复用SysNav、Astribot SDK及CaP-X／ACT／π0.5。实验端到端策略每任务200条示范训练或微调；ACT、π0.5也是Thea工具背后的同一策略，不能全部算作仅基线数据。基础checkpoint、冻结范围及完整训练流程未披露。 | 统一异构能力接口，并将动作成功与函数返回分开。 |
| System Architecture → Skills | 人工编写／维护的SKILL.md和资源；没有梯度训练。 | 渐进加载程序性知识，区别于工具契约和底层motor policy。 |
| System Architecture → Memory | LLM依据执行历史整理文本；候选经验经过置信度门槛，非更新模型权重。 | 把任务笔记与长期经验分开，限制未经证实内容进入记忆。 |
| System Architecture → Safety | 几何量、规则与执行预算；非RL训练。 | 在模型之外强制检查；不是经认证的物理安全系统。 |
| System Architecture → User Interaction | 消息通道，无学习数据；实验使用飞书／Lark。 | 将人类补充信息接回闭环。 |
| Bridging the Gaps → Scene Graph as Context | 复用Astribot SDK／SysNav感知；具体感知checkpoint及冻结情况未披露。在线图更新不是模型训练。 | 用稳定引用保持对象连续性；动作引起的变化需验证后写入。 |
| Bridging the Gaps → Evaluation as Exit Codes | 复用Qwen3.7-Plus，未报告本文微调；90个检查点用于评估，非训练集。 | 不读决策理由／自报成功，仅依客观证据判断；连续策略分段检查，程序式策略结束后检查。 |
| Bridging the Gaps → Embodiment Profile | 静态人工配置，无参数训练。 | 将机器人配置外置；换文档仍须实现对应硬件和策略适配。 |

[§5、图8–9](https://arxiv.org/html/2608.11246v1)主任务定量实验在轮式双臂Astribot S1：单物抓取、多物整理、导航取放分别每方法20／20／15次；另展示Cobot Magic、G1，不等于完成同规模跨人形平台评测。评估器在三平台各类均衡的90个检查点上准确率93.3%，但6.7%的失败与13.3%的进行中状态被误判成功；这个数是动作状态分类准确率，不是任务成功率。独立评估提供可研究接口，也仍是闭环失效来源；场景图只表达粗状态，不能直接证明可抓取或成功。对当前人形方向，可复用的是上下文／工具／验证边界，底层灵巧操作、行走控制及可靠感知仍须另行解决。

## WholeBodyWAM（WBC-Grounded版）— WholeBodyWAM: Generalizing Pre-trained World-Action Priors to Humanoid Loco-Manipulation via WBC-Grounded Coordination

**重点关注**：False

**开源情况**：完全没开[2026-10-16]（范围为本文2609.16644系统：[官方项目页](https://wholebodywam.github.io/)提供方法与演示，未找到官方UWBC／CASA实现、WholeBodyWAM权重、15K训练数据、训练配置或仿真／真机部署入口；所复用DreamZero和WBC的开放材料不能替代本文系统；未运行）。

**基本信息**：阅读日期 2026-10-16；arxiv上传：2026-09-15（所读最新v1，同日；阅读8页全文的方法、训练、实验与消融，原文未附独立训练附录；核对PDF作者页与官网）；一作：Zhuo Li（标Project Lead）；等贡献：Yiming Yao、Jim Tan（两人标∗，不扩写成前三人共同一作）；通讯：Fei Chen；单位：The Chinese University of Hong Kong、The University of Hong Kong、Peking University、Φ-Institute；是否中稿：未核实；[项目主页](https://wholebodywam.github.io/)，[arxiv](https://arxiv.org/abs/2609.16644v1)。同月另有同名论文，引用需保留本篇副标题及arXiv ID。

**核心问题**：把已有world-action模型的手臂操作知识迁到人形全身动作：保留操作分支，新增统一WBC命令表示，并在手臂当前方向可操作性下降时增强操作意图对身体调整的影响。对本课题更直接的启发是“上层动作表达如何可靠落到底层控制接口”，不是现成的技能库调度或多方案仿真验收harness。

**系统拆解**：语言＋视觉历史＋本体状态 → 共享Diffusion Transformer联合生成未来视觉、操作动作和UWBC命令 → 操作分支给手臂／手指目标，UWBC按所选控制器的能力映射成命令 → 低层WBC执行。论文的概率分解只是依赖关系，**不是先生成视频、再做动作推理；也未报告显式采样多个未来并由价值函数选优的MPC过程**。[§III、Fig.2–3](https://arxiv.org/html/2609.16644v1#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| WholeBodyWAM → context encoding | T5 text encoder；Wan video VAE；state encoder | 指令、视觉历史、本体状态 → 文本特征、视频latent和状态token → 提供DiT条件 |
| WholeBodyWAM → shared DiT／Structured Action Factorization（SAF） | visual、manipulation、UWBC三类token；joint flow matching | 条件与带噪未来block → 联合去噪 → 未来视觉v、手臂关节／手指目标m、全身命令u；保留原有视觉—操作路径 |
| WholeBodyWAM → shared DiT → Coordination-Aware Self-Attention（CASA） | task-directional manipulability；门控attention bias | 当前关节构型、相邻观测末端位移 → 方向可操作性与离线voxel参考比较 → 在同一预测block内增强manipulation→UWBC注意力 |
| Unified Whole-Body Controller Interface（UWBC） | 56维语义槽；能力注册与mask | 46维共享字段（16维任务空间＋15维下肢／腰位置＋15维速度）及10维控制器残差 → 按物理语义、单位与支持能力启用／屏蔽 → 具体WBC可读命令 |
| Downstream WBC | SONIC、AMO、GEAR WBC | 操作目标和已映射命令 → 控制器按自身接口执行全身运动 → G1动作；不能把这三种控制器当作同时运行的三个agent |

**训练、复用与新增**：以DreamZero的预训练14B WAM初始化，在SIMPLE中用PICO 4 Ultra与腕部追踪器采集15K全身示范，转换为SMPL并通过GMR重定向到G1，再依槽位物理语义计算UWBC监督、时间对齐。LoRA后训练rank=4、α=4、50,000步、学习率1e-5、每GPU batch=1；之后还需任务微调，主仿真实验每任务100条示范，另测50／100／200／300条。推理16步二阶UniPC去噪。以上是本文训练，不应把引用WAM／WBC的原始训练配方直接归给本文；完整冻结清单、新模块初始化及真机微调数据量未报告。[§IV-A/B](https://arxiv.org/html/2609.16644v1#S4)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| WholeBodyWAM → context encoding | 复用预训练T5、Wan VAE；具体型号／checkpoint和是否冻结未明确。轻量state encoder随系统提供本体条件，初始化细节未报告；输入来自上述示范和运行时观测 | 让同一个条件上下文服务视觉、操作与全身意图，而不是独立生成互不协调的动作 |
| WholeBodyWAM → shared DiT／Structured Action Factorization（SAF） | 复用DreamZero14B先验，以LoRA后训练及下游示范微调；三模态加权flow-matching监督。保留先验不等于完全不更新；新UWBC投影／头的精确参数训练范围未披露 | 保留已有操作表示，同时新增显式可寻址的全身命令流；避免将所有异质命令简单拼成单一动作向量 |
| WholeBodyWAM → shared DiT → Coordination-Aware Self-Attention（CASA） | gate由运动学、关节速度限制和离线采样可操作性地图计算，不是另训的RL策略；注意力所在DiT随上述训练适应该偏置。末端位移估计任务方向，离线地图计算不是学习示范 | 当手臂沿当前方向的能力相对下降，选择性增加UWBC query对同block manipulation key的注意力，双臂取较大需求 |
| Unified Whole-Body Controller Interface（UWBC） | 槽位含义／单位与能力mask是预定义接口；训练监督由SMPL→GMR后的机器人参考动作推导，生成这些命令的网络随DiT训练 | 共享物理语义，保留控制器特定的10个残差槽；未支持字段屏蔽，不声称一个命令向量无适配驱动所有WBC |
| Downstream WBC | 复用已有SONIC／AMO／GEAR WBC，本文未明确给出它们的重训或checkpoint。跨WBC实验对每种控制器分别做任务微调，不是固定单模型直接替换控制器 | UWBC把上层意图映射到已有执行接口，降低控制器语义差异带来的适配负担 |

实验边界：SIMPLE六任务、三个扰动等级、SONIC执行时平均成功率91.9%，DreamZero为65.0%、使用同一15K后训练数据的DreamZero-PT为73.6%；去CASA／UWBC／SAF分别降到86.9%／84.7%／80.8%。跨三WBC分别微调后平均89.2%，成功率方差10.5 pp²，对Cosmos-3的35.0 pp²降低70%，不能写成零样本可迁移性。G1＋两只BrainCo Revo 2手＋头部及双腕三台D435i RGB相机的八任务真机实验，每方法／任务20次：ID／OOD成功率81.3%／68.8%，DreamZero为57.5%／40.0%；OOD任务进度0.82对0.59，即摘要的+0.23。真实对照仅DreamZero，缺DreamZero-PT等同数据真机对照；不宜将全部收益单独归因CASA或一般开放世界能力。[Table II–III、Fig.4、7、§IV](https://arxiv.org/html/2609.16644v1#S4)。

## WAA — World Action Agent: Harnessing VLMs for Robot Manipulation via World Action Rehearsal

**重点关注**：False

**开源情况**：完全没开[2026-10-16]（检查原文、arXiv、[通讯作者论文页](https://www.yingcong.me/publication/2026-zhang-world-action-agent/)及可定位公开资源，未找到本文官方系统代码、演化技能库、小模型适配权重、轨迹数据或仿真／部署入口；论文给出接口、提示词与训练配置，不能代替实现；未运行）。

**基本信息**：阅读日期2026-10-16；arXiv首次上传2026-09-24，所读为同日v1，当前仅此版；已读方法、实验及附录视图、工具、提示词、微调和失败分析。共同一作Yehang Zhang、Haojian Huang；通讯Ying-Cong Chen，Zexi Li的†表示项目负责人；单位HKUST(GZ)、CUHK、Knowin AI；是否中稿未核实，arXiv标注Working in progress；未提供独立项目主页。[arXiv](https://arxiv.org/abs/2609.29964)。

**核心问题**：怎样让VLM把视觉中的空间判断变成可修正动作？WAA构建带几何定位、虚拟动作预览与图内微调的工作空间，并把示范提炼为图文技能；执行靠运动规划和控制接口，技能库本身不是motor policy。

**系统拆解**：输入任务、场景点云／重建几何、机器人状态，形成Canvas给Main Agent；模型调用查询、提案或执行工具，动作后刷新画面。按[§3、附录C–D](https://arxiv.org/html/2609.29964v1)保留工作空间与学习路径；这里的rehearsal是目标姿态、IK及轨迹可行性预览，没有学习式接触动力学或未来视频预测。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Main Agent | VLM、query/proposal/execution tools | 任务＋当前Canvas＋控制上下文 → 选择工具和参数 → 空间查询、待执行提案或物理动作。 |
| Visual Action Workspace → Interaction-centered Canvas | Global view、两路Contact views | 场景几何＋机器人FK＋活动区域 → 优化可见性、紧凑性和视图互补性 → 可定位到3D的多视图。 |
| Visual Action Workspace → Action rehearsal | Main／Imagination Agent、IK、cuRobo | 目标点／抓取候选 → 构造姿态、规划并叠加虚拟机器人 → 可编辑提案；调用execute_action才执行。 |
| Visual Action Workspace → Closed-loop execution and in-view correction | 图内拖动、局部旋转、夹爪控制 | 已规划动作及新观测 → 执行大动作，再把Contact view像素偏差映射为有限末端位移 → 更新实态。 |
| Learning and Evolution through the Harness → Multimodal manipulation skills | 技能文本、关键状态图、Skill Agent | 当前问题＋技能 → 单独查看历史参考图并判定适用性 → 建议与不确定性，无机器人控制权限。 |
| Learning and Evolution through the Harness → Skill evolution from demonstrations and teaching | Learner、Editor、Reviewer | 示范视频或人工接管过程 → 提取证据、编辑并审核 → 新增／修订／淘汰图文技能。 |
| Learning and Evolution through the Harness → Learning to pilot | 小VLM主代理 | Canvas、任务、控制上下文及技能建议 → 模仿教师工具调用 → 替换Main Agent。 |

**训练、复用与新增**：主实验的技能演化是非参数学习；另有独立的小模型LoRA实验。两者训练数据来源、泛化口径不同。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Main Agent | 默认直接复用Gemini 3.7 Flash，主实验不微调；执行时子agent和grounding也用该模型。输入当前图像／几何是推理观测。 | 将VLM决策落到有限工具接口；终止自评与环境成功判据分离。 |
| Visual Action Workspace → Interaction-centered Canvas | 几何视角优化不训练。比较仿真场景点云、四视角RGB-D＋FK手模型、8张RGB经VGGT重建；VGGT具体checkpoint／冻结情况未交代，未报告本文微调。全部实验输入来自仿真。 | 围绕当前交互生成互补视图，将图像点击关联到空间锚点。 |
| Visual Action Workspace → Action rehearsal | 复用IK与cuRobo；VLM在虚拟提案中迭代，不以试动作训练控制策略。 | 将意图与执行分开，先看规划反馈和姿态投影再决定执行；不预测抓取后的物体动力学。 |
| Visual Action Workspace → Closed-loop execution and in-view correction | 数值规划／低层控制器，无本文motor-policy训练；局部拖动／旋转可直接执行。 | 保留视觉闭环的厘米级修正；规划可行不等同任务成功。 |
| Learning and Evolution through the Harness → Multimodal manipulation skills | 技能存适用条件、程序、参考状态图和可见成功条件；查询不更新模型参数。 | Main Agent拿完整文本，Skill Agent隔离历史图像，随当前观测变化使旧建议失效。 |
| Learning and Evolution through the Harness → Skill evolution from demonstrations and teaching | Codex GPT-5.5按API生成种子；再取LIBERO-90每个源任务一条专家轨迹，用GPT-5.5的Learner／Editor／Reviewer演化；评测前冻结库，不用LIBERO-Pro／robosuite失败继续调库。炉灶单示范案例还用对齐末端朝向，不能泛称纯视频。 | 将视频／人工指导转成可审查的程序性知识，不更新模型权重，不直接生成新motor policy。 |
| Learning and Evolution through the Harness → Learning to pilot | LIBERO-Pro六split中112条成功Gemini轨迹、1774次主代理调用；Qwen3.5-9B基座LoRA(rank8, α16)，冻结视觉编码器及多模态投影器，21.6M可训练参数。仅目标工具调用计损失，10epoch、lr5e-5；4×H20约9.5h，无独立验证集，取末轮adapter。 | 用harness记录训练小模型操作同一接口；只替换主代理，子agent和grounding仍依赖Gemini。见[附录E、表7](https://arxiv.org/html/2609.29964v1)。 |

[§4、表1](https://arxiv.org/html/2609.29964v1)LIBERO-Pro共六split、每split60次，完整方法平均75.6%基于仿真场景点云；RGB-D为71.1%，VGGT为68.9%；同框架无技能28.9%、种子技能43.3%。技能来自LIBERO-90且评测冻结，不能把“小模型也未用LIBERO-Pro训练”作为结论。Show-Harness同Gemini骨干由作者重跑，每split30次；其余基线引用原论文成绩，数据与评测条件并非全统一。robosuite三任务只检验这套操作接口的迁移，未做人形／真机验证。小模型“域外”1.7%→43.3%指LIBERO-Pro内未见seed，不是新物体、新任务或新机器人泛化；此时云端子agent仍在。[§4.3、附录E–F](https://arxiv.org/html/2609.29964v1)也显示视角误判、目标身份丢失、错误宣告成功；Reviewer审核技能知识，并非Thea式独立逐动作成功评估器。对本项目，值得验证的是空间交互接口和可审查技能记忆，不能据此推导全身稳定性、接触安全或实时性能。

## RoboHarness — Memory-Driven Orchestration of Heterogeneous Robot Policies for Long-Horizon Planning

**重点关注**：False

**开源情况**：完全没开[2026-10-16]（在本论文[官方主页](https://www.robo-harness.com/)及原文范围未见系统代码、Memory Bridge／策略卡实现、自建 500 任务定义、记忆库、真机适配权重与运行／部署入口；底层模型的公开代码不等于框架开源；未运行）。

**基本信息**：阅读日期 2026-10-16；arXiv 首次上传 2026-07-20，所读最新 v3（2026-09-29）；读正文方法／实验及附录 B–D 的训练、真机、技能实现与评测设置。一作（共同）：Jinbang Huang、Zhiyuan Li；通讯：未披露；单位按 v3：Huawei Noah’s Ark Lab、University of British Columbia、University of Toronto、McGill University、Department of Foundation Model, 2012 Labs。是否中稿：未独立核实；arXiv 作者备注称获 ECCV 2026 Agent in the World Workshop 最佳论文，[官方 workshop 页](https://interactive-intelligence-lab.github.io/eccv26-aiw-page/)未查到对应获奖名单，不能据此记为 ECCV 主会录用。[项目主页](https://www.robo-harness.com/)，[arXiv](https://arxiv.org/abs/2607.18060)；注意不是同名编号 2603.24060。

**核心问题**：异构策略擅长的任务和可接手状态不同，顺序调用容易失败；作者用 coding agent、策略能力描述、成功经验记忆与 Memory Bridge 路由和衔接策略，并依据执行反馈修改编排。

**系统拆解**：输入任务、图像、状态／物体姿态、策略卡与历史；agent 分解子任务→查询兼容性和记忆→必要时运动规划桥接→调用保留原接口的策略→检查结果、更新记忆与编排。输出经各策略及控制器执行的机器人动作；不是将各策略重训成统一动作模型。[§4、图1、附录C](https://arxiv.org/html/2607.18060v3)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Coding Agent | Codex / GPT-5.5 | 任务＋观察＋策略描述／反馈→分解、选择与调用技能、重规划；可检查代码与修改编排。 |
| Understanding Skills | 五类 Assessment | 姿态稳定性、视觉／语义相似度、图像质量、状态—策略兼容性→给 agent 的选择证据。 |
| Memory Skills → Memory Management | 策略专属链式轨迹、分层检索 | 成功轨迹保存观察、指令、末端／关节／夹爪状态及特征；先语义后视觉检索相关锚点。 |
| Memory Skills → Memory Bridge | 局部排序模型、目标筛选、运动规划 | 同子任务／夹爪状态下扩展锚点邻域→拟合进度排序→筛选靠近记忆且可达的目标状态→运动桥接、再检查并切换策略。 |
| Heterogeneous Policies as Agentic Skills → π0.5 | VLA | 语言＋原生观察／状态→动作；承担部分操作与真机抽屉交互。 |
| Heterogeneous Policies as Agentic Skills → Faster-WAM | 世界—动作模型 | 原生任务／视觉输入→视频预测和动作输出。 |
| Heterogeneous Policies as Agentic Skills → RL-post-trained OpenVLA-OFT | GRPO 后训练 VLA | 原生语言／观察→操作动作。 |
| Heterogeneous Policies as Agentic Skills → TAMP | PDDLStream、FastForward、OMPL/RRT | 目标＋对象姿态／状态→符号任务与运动计划；含物体姿态、VLM 视觉控制、记忆辅助三种模式。 |
| Evolution Skills | Policy Adaptation、Harness Refinement、Metadata Update | 成败记录→适配抓取／逻辑表示，修正编排代码／参数及策略描述→后续回合继续使用。 |

**训练、复用与新增**：没有统一联合训练不等于无训练；复用预训练模型、本文策略微调、桥接时拟合及跨回合编排适配须分开。以下按附录 B–C 核验。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Coding Agent | 直接调用 GPT-5.5，未报告本文权重微调；输入策略卡、观察与运行记录。 | 以代码 agent 编排异构能力，支持检查与修改运行逻辑。 |
| Understanding Skills | 明确冻结 SigLIP2-base-patch16-224、DINOv2-base、bge-large-en-v1.5；姿态均值／方差、曝光／Laplacian 锐度等为统计规则。兼容性复用记忆与桥接结果。 | 汇集任务／视觉／状态证据；不是训练好的成功概率判别器。 |
| Memory Skills → Memory Management | 从各策略相应数据集的成功轨迹初始化，并追加成功执行；存储／检索无新神经训练。失败也记入供 evolution 使用的执行历史，但不等同于成功轨迹库。 | 分策略记忆与时间链接，保留检索锚点前后上下文。 |
| Memory Skills → Memory Bridge | 每次从检索邻域拟合线性 pairwise ranking SVM；以相对时间顺序为排序标签、归一化机器人状态为输入；是在线拟合。 | 得分只表示局部相对进度；加近邻距离、运动可行性与阈值约束，再权衡运动成本选目标。 |
| Heterogeneous Policies as Agentic Skills → π0.5 | LIBERO 系列复用官方 pi05_libero；RMBench／BEHAVIOR 作者按任务数据训练，30k 步，未完整披露数据规模／冻结范围；真机从 pi05_libero 微调开／关抽屉各 50 条人工示范。 | 将学习操作与 TAMP 结构化执行互补。 |
| Heterogeneous Policies as Agentic Skills → Faster-WAM | 作者按原公开流程在 LIBERO 训练；本文未充分披露基础 checkpoint、数据规模与冻结范围。 | 以原生接口接入策略库，不代表四种策略均用于所有实验。 |
| Heterogeneous Policies as Agentic Skills → RL-post-trained OpenVLA-OFT | 复用 RLinf/RLinf-OpenVLAOFT-GRPO-LIBERO-90；GRPO 后训练是所用模型的来源，未见本文作者重训证据。 | 接入另一种可独立选择的操作策略。 |
| Heterogeneous Policies as Agentic Skills → TAMP | 作者实现规划器组合；几何／符号求解不是策略训练。仿真用模拟器物体姿态并加噪，真机用 ArUco 位姿；记忆模式可执行检索片段前 40%。 | 三种执行模式及记忆辅助；真机能力依赖标记感知与平台 API。 |
| Evolution Skills | SIMPACT 适配抓取，PDDLLM 学习逻辑表示；GPT-5.5 根据失败改编排／元数据及离散参数。仿真启用三项，经验和修改只在同一任务跨评测回合保留。 | 利用执行反馈修正系统，不能当成静态、空记忆的零样本评测。 |

v3 报告 LIBERO-Plus 加权成功率 91.9%、LoHo 五任务 96.0%；自建 LIBERO-Complex 500 实例 86.8%，去掉 Memory Bridge 降至 60.8%。BEHAVIOR-1K 只测六项任务，不代表全部千项。主页仍有旧版 93.2／95.2 等数字，本条以 v3 为准。真机是 UR5e＋双 RealSense D435＋ArUco：五种结构与四种干扰各 15 次，共 135 次，使用 π0.5＋TAMP，不能泛化为四策略／人形真机验证。[§5、表1、图3–5、附录B](https://arxiv.org/html/2607.18060v3)

Memory Bridge 不是分布密度估计或成功概率，也不是“进到记忆邻域就安全”的保证；关节／末端状态相近仍可能面对不同物体布局，桥接后还须结合视觉、任务状态检查。与本课题直接相关的是技能接口、可接手状态和恢复机制；异构策略能否在我们的本体上共享状态表示、获得可靠位姿及及时完成 agent 调用，仍是未验证条件。

## HomeBody（工程项目）— A Humanoid That Explores, Remembers, and Acts on Its Own

**重点关注**：False

**开源情况**：可以sim[2026-10-16]（已核实[官方仓库](https://github.com/Stanford-TML/homebody)提供MuJoCo环境、G1／Dex3与厨房资产、AMO及adapter权重、navigate／pick／place技能、VLM与无VLM脚本入口；限已提供场景与技能，未运行。REAL2SIM和REAL在当前README仍标Planned，日期分别为10月16日和23日，较本次PDF所写12日和19日延后；对应模块仍抛出NotReleased，不能据计划日期归为已发布）。

**基本信息**：阅读日期2026-10-16；arxiv上传：未提供，目前按工程项目而非已核实论文归档；首位署名：Gio Huh，未声明共同一作；通讯：未标注，C. Karen Liu与Guy Tevet的†表示共同指导；单位：Caltech、Stanford University；是否中稿：未核实。阅读范围为[项目主页](https://tml.stanford.edu/homebody/)的方法、FAQ、架构图及官方代码／资产，源码核验版本为[c09fcf3](https://github.com/Stanford-TML/homebody/tree/c09fcf39468e7bab43d744458a53ef1b1265cd6b)；网页Paper按钮未提供论文链接，不据演示补写训练或评测结论。

**核心问题**：如何让人形机器人利用环境记忆、通用VLM和可执行技能，在家庭场景里持续完成移动与抓放。核心价值是把观察、任务选择、技能参数、局部重试和失败反馈接成闭环；不是新训练一个端到端操作策略，也没有展示对多组候选动作的未来物理后果统一评分选优。

**系统拆解**：官网完整流程为Explore采集→Real2Sim重建→任务执行：先建立有尺度的环境与语义记忆，再让VLM根据当前观察和历史结果选择技能及目标，技能内部进行几何规划、执行与验证，失败回传VLM。**当前公开仿真只覆盖后半段**：加载已有厨房、初始语义与地图，再循环观察→决策→技能→结果。它使用MuJoCo；官网Real2Sim构建Isaac Sim数字孪生，不应混为同一个执行后端。[架构与FAQ](https://tml.stanford.edu/homebody/)、[运行循环](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/src/homebody/session/session.py)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Spatial context and memory | Explore；Real2Sim；地图与历史观察 | 官网：iPhone／D435i／LiDAR／关节记录→SLAM几何、VLM辅助重建、ICP对齐与关键帧记忆→空间上下文；公开版加载src_kitchen的地图、语义与初始场景描述 |
| High-level task | GPT Astra；session与技能调用反馈 | 指令、当前视图／手部状态、空间上下文及近期结果→VLM选择技能与参数，或声明done→Runner调用技能并记录结果供下轮决策 |
| Skill bank → Navigate | prepare／execute；占据图、A*、到达检查 | 地图平面目标及朝向→膨胀障碍、搜索和跟踪路径→到达／超时／碰撞等结果，失败可由高层改目标 |
| Skill bank → Pick／Place | 几何抓取／放置、手臂轨迹、局部恢复 | 像素目标及手或放置点→定位、候选抓取、站位和轨迹规划、闭手抬升／释放验证→结构化成功或失败；官网另演示开抽屉／从抽屉取物，当前公共registry只含navigate、pick、place |
| Physical execution → AMO／robot backend | AMO下身控制；上肢IK／伺服；传感与动作接口 | 速度、体姿与上肢目标→预训练AMO和关节控制→G1动作及新观测；控制循环上肢／手250Hz、AMO 50Hz；当前实际提供MuJoCo backend |

**训练、复用与新增**：作者“不需环境专门训练／新增策略学习”的声明限定于本系统的组装与部署，所用VLM、感知网络与AMO都有先前训练经历。当前代码未提供新运动策略训练流程，不能称为所有模块从零训练或未经训练。几何建图、轨迹搜索、视觉伺服与重试属于估计／规划／执行，不等于网络训练。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Spatial context and memory | 官网用采集的多传感器记录建图、对齐和重建，不是遥操作技能示范；数据量未披露。Astra复用现成模型，未报告微调；公开版只给已有场景资产，完整采集重建模块未发布 | 将地图、对象语义、历史视角与机器人定位提供给任务执行；现有代码不能验证重新探索陌生住宅到建模的全流程 |
| High-level task | 复用托管VLM，当前默认gpt-6-astra，未报告本文微调；指令、图像和执行历史是推理上下文，不是监督训练集 | 以统一技能参数和结果组织循环；Runner核对参数、传感时间戳、执行epoch与声明的错误码，并支持中止，便于界定失败发生在哪一层 |
| Skill bank → Navigate | A*与跟踪／重试规则，无独立学习导航策略或训练数据；使用已有地图与在线位姿 | 将可走性规划、执行和到达验证封装为可调用接口；不能把地图目标自动等同于物理可达并成功完成 |
| Skill bank → Pick／Place | 公开仿真用渲染深度／分割及几何计算，无单独训练的抓放policy。官网真机描述复用SAM、Fast-FoundationStereo、SAM2.1／SAMURAI等感知组件，未报告本文微调或数据量；这些真实感知误差不在当前仿真输入中复现 | 用抓取姿态、站位调整、轨迹／IK、接触及抬升验证和有限次恢复组成技能；参数／失败接口有参考价值，但不是任意学习策略无需适配即可替换 |
| Physical execution → AMO／robot backend | 直接复用AMO控制权重、adapter和归一化统计；THIRD_PARTY_NOTICES确认与上游发布文件一致，没有本文重训证据。Holosoma运行代码经适配；上游训练配方不归作本文新训练 | 以统一backend连接观测、全身动作与物理状态；公开真机backend尚为NotReleased占位，不能推断可直接部署到用户平台 |

核验边界：仿真backend直接返回渲染真值深度、分割及由模拟器直接读取的机器人／相机位姿，初始语义也由场景资产提供；不能据此认为已解决真机遮挡、定位漂移或开放词汇目标识别。技能有局部物理验证，但高层done被代码明确记为model_claim，任务物理评分另行计算，不能称为通用可靠的任务完成判定。官网真机视频与FAQ展示整链路并承认模型延迟、硬件问题，目前未见论文式统一成功率评测。依据：[仿真观测实现](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/src/homebody/backends/mujoco.py)、[技能契约](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/src/homebody/skills/contract.py)、[权重来源](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/THIRD_PARTY_NOTICES.md)。

## HANDOFF — Humanoid Agentic Task-Space Whole-Body Control via Distilled Complementary Teachers

**重点关注**：False

**开源情况**：[2026-10-16] sim复现未核实（官方WBC依赖）；sim重新训练未核实（SEED版本／依赖）；直接部署未核实（G1 WBC依赖）；缺少材料（完整planner／操作物体／任务rollout）。[官方代码](https://github.com/lzyang2000/HANDOFF)

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-06-04，所读最新v4（2026-09-29）；一作：Lizhi Yang，共同一作未披露；通讯：未披露；单位：California Institute of Technology（Caltech）、Institute for Human & Machine Cognition（IHMC）；是否中稿：未核实，当前官方标注arXiv预印本。[项目主页](https://lzyang2000.github.io/HANDOFF/)，[arxiv](https://arxiv.org/abs/2606.06493)。

**核心问题**：让上层规划器用简洁任务目标调用兼具行走、伸手与跌倒恢复的全身控制器；通过按动作部位和状态分配教师监督，将三种专长蒸馏成一个接受10维指令的策略。对本课题可关注其规划—控制接口与恢复能力，上层空间记忆和技能验证仍需另行设计。

**系统拆解**：语言＋RGB-D→任务分解与目标定位→确定性工具生成10维指令→HANDOFF结合本体历史输出29维关节位置→500 Hz关节PD执行，策略运行50 Hz。指令为`[vx, vy, ωz, z, pL³, pR³]`：平面速度3维、根部高度1维、骨盆坐标系双腕位置6维；腕朝向与夹爪另由工具控制。三个教师用于训练，部署运行一个学生策略。[§III、图2／5](https://arxiv.org/html/2606.06493v4#S3)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Agentic planner → High-level reasoner | regex parsing、LLM fallback | 语言指令→正则拆分，未匹配时调用LLM→原子任务。 |
| Agentic planner → VLM | 2D point／bounding box、RGB-D投影 | 图像与目标→二维定位、深度反投影、近距离重新查询→机器人坐标系目标；执行不依赖全局场景地图。 |
| Agentic planner → Waypoint tracker／skill selector | 路径规划、航点跟踪、腕姿态修正、离散夹爪控制 | 目标与阶段→跟踪器给速度，skill selector给高度与腕位置→10维控制指令及额外腕姿态／夹爪命令。 |
| Whole-Body Control → Whole-Body Control Teacher | 29-DoF motion tracking、asymmetric PPO、CoP／CBF数据校正 | 29维参考关节角＋11帧本体历史→模仿姿态、下蹲与双臂协调→全身教师动作；critic另用特权状态。 |
| Whole-Body Control → Locomotion Teacher | 15-DoF legs-and-waist policy、gait phase | 速度命令＋本体状态／步态相位→行走与稳定→腿腰动作；训练时参考动作驱动手臂，产生重心扰动。 |
| Whole-Body Control → Fall-recovery Teacher | 29-DoF AMP、discriminator、fallen resets | 倒地初始化及本体历史→任务奖励＋动作风格先验→起身并恢复行走的全身动作。 |
| Whole-Body Control → Context-Conditioned Behavior Composition | action-sliced KL、soft MoE student、PPO | 同一仿真状态下查询三个教师→按腿腰／手臂及站立／恢复状态分配KL监督→仅依靠10维指令与本体历史执行的学生。 |
| Whole-Body Control → Whole-Body Stability Shaping | CoM、capture point、ankle–hip–step、动量变化惩罚 | 质心、接触与动力学状态→构造平衡奖励→共同训练WBC教师、行走教师与学生；AMP保留其恢复目标。 |

**训练、复用与新增**：四阶段均使用mjlab／Rsl-rl，在线仿真PPO；公开默认配置从新网络训练，未指定外部预训练控制checkpoint。学生蒸馏阶段读取前三阶段教师，教师eval／no_grad且不参与学生优化。论文“无任务专用数据或微调”指训练完成后更换任务的执行阶段。每轮4096环境×24步；v4给出WBC／学生最多30K、行走20K、AMP 100K更新。[§III-F](https://arxiv.org/html/2606.06493v4#S3.SS6)、[训练入口与数据流程](https://github.com/lzyang2000/HANDOFF#training-pipeline-run-in-order)

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Agentic planner → High-level reasoner | 规则解析无参数训练；LLM直接调用，未报告本文微调，型号及冻结细节未披露。 | 将语言分解为可执行阶段，保留LLM兜底。 |
| Agentic planner → VLM | 复用local VLM和ChatGPT APIs，未报告本文微调；型号／checkpoint及冻结细节未披露。RGB-D和重查询为推理输入。 | 二维语义定位接深度几何，近距离更新目标以减小定位误差。 |
| Agentic planner → Waypoint tracker／skill selector | 确定性工具与几何修正，未报告单独训练技能策略；运行时使用当前目标和状态。 | 统一生成任务空间指令，并补充10维接口之外的朝向与夹爪控制；公开运行总线为18维。 |
| Whole-Body Control → Whole-Body Control Teacher | 从头PPO；离线BONES／SEED重定向人体动作作参考，仿真产生RL经验。先筛动作，再对准静态下蹲帧做CoP／CBF校正；实际保留clip数未披露。 | 修复部分参考动作的静态稳定性，再学习全身协调；论文校正条件为根高＜0.65 m、平面速度＜0.2 m/s。 |
| Whole-Body Control → Locomotion Teacher | 从头PPO；速度采样、步态和稳定奖励提供在线监督，手臂参考来自动作库；无需任务遥操作示范。 | 专门补足人体动作跟踪教师在任意速度命令下的行走能力。 |
| Whole-Body Control → Fall-recovery Teacher | 从头训练PPO actor／critic及AMP判别器；1段跌倒起身＋17段走／跑／转向参考，仓库提供18个NPZ，具体原始来源未披露。奖励为0.25风格＋0.75任务，最多40%环境使用延迟倒地重置。 | 借动作先验学习倒地到站立的转移，并保留站立后跟踪目标的能力。 |
| Whole-Body Control → Context-Conditioned Behavior Composition | 学生专家随机初始化；以PPO＋冻结教师KL训练编码器、MoE专家和门控。参考经FK生成高度／腕目标，速度独立采样；无需新增任务示范。 | 腿腰KL随命令速度由WBC平滑转向行走教师；手臂保留WBC监督；恢复状态由AMP监督全身。MoE每个expert均输出29维，仅恢复expert显式指定，其余两个自行分配直立行为。 |
| Whole-Body Control → Whole-Body Stability Shaping | 奖励设计，无单独模型或离线训练集；使用在线仿真接触、质心与动量。域随机化包含载荷、摩擦、PD、驱动力、编码器偏差、手部力和延迟。 | 将相同平衡目标加入相关教师与学生；恢复时允许手／环境接触参与支撑处理。 |

按[表II、§IV-B](https://arxiv.org/html/2606.06493v4#S4)，适配到相同腕目标接口后，HANDOFF的Robust WS为0.277 m³、feasibility 93.0%，SONIC为0.265 m³、96.2%；基线均经同一differential-IK桥接。Robust WS按前方目标凸包体积×feasibility计算，腕误差阈值15 cm，feasibility要求无跌倒且骨盆漂移＜25 cm；优势适用于该接口与指标口径。500个配对倒地初态中恢复率89.6%，成功恢复平均3.2 s；这些成功案例随后全部恢复0.4 m/s行走，Resume的分母为成功恢复案例。

[表III、§IV-D](https://arxiv.org/html/2606.06493v4#S4.SS4)五任务各500次仿真，成功率依次为取放99.8%、左右臂分别取放100%、双臂交接77.6%、蹲姿取物67.2%、取物运输放置72.2%。仿真注入实测1.2–1.6 cm定位误差，并在12 cm夹爪开口条件内使用kinematic grasps。G1＋ZED-M＋Jetson Thor展示前五类真机操作，未报告真机成功率；恢复后任务续接在图6呈现仿真画面，[官网](https://lzyang2000.github.io/HANDOFF/)明确标为仿真。

公开执行后端为MuJoCo，训练框架为mjlab／MuJoCo-Warp。内置ONNX、G1／活动夹爪XML及URDF、网格、房间／桌／垃圾桶资产已发布，已检查引用文件存在。默认sim入口创建平地G1并用刚性载荷块模拟手部质量，未启用活动夹爪；键盘／手柄／VR命令入口已提供。选择含夹爪的任务XML时会导入未发布的`deploy.sim.graspnet_scene`，操作物体本应由它注入；未见Pringles／Cheez-it等物体包、五任务评测与完整抓取rollout脚本。[默认场景与加载](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/sim/sim_policy_node.py#L1009)、[物体注入说明](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/assets/g1/g1_sim2sim_29dof_gripper.xml#L725)。

完整planner尚缺任务解析、VLM语义目标定位、路径／轨迹生成与阶段执行节点。UI可发送语言，但`/molmo/ui/prompt_submit`未见接收者；`/planner/*`目前主要是订阅和显示。MuJoCo相机RGB-D及整幅深度反投影为XYZ点云已有实现，语义目标点到操作目标的完整消费端未发布。常量含Hunyuan／gpt-5.5配置，所指`hy_client.py`、`planner_node.py`等文件缺失，论文实际调用型号仍未核实。[语言发布端](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/sim/sim_policy_node.py#L756)、[反投影](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/sim/sim_policy_node.py#L955)。

Ubuntu基础WBC试跑需匹配ROS2的Python 3.10–3.12、uv与zsh；脚本shebang及语法要求zsh，README的bash示例有出入。此次检查`uv.lock`锁定MuJoCo `3.8.1.dev913242127`的Linux x86_64／Python 3.10、3.12下载链接均返回404，因此收紧此前“sim复现／直接部署”状态，先修复依赖并实际验证；本次未启动仿真或真机。完整语言任务还需补planner、操作物体和任务执行脚本。[依赖](https://github.com/lzyang2000/HANDOFF/blob/main/uv.lock#L1806)、[启动入口](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/play_sim_hand.sh)。重训另需自行生成过滤后SEED PKL／YAML，[BONES-SEED](https://huggingface.co/datasets/bones-studio/seed)需接受许可，README所用v003与当前v004组合尚未完整核实。公开README建议约20K学生checkpoint，v4描述评估末轮，两者对应关系未核实。

**关键 Q&A（2026-10-16）**：Q：主要工作是否集中在WBC，planner从哪里来？A：主要方法贡献是WBC、10维接口和三教师蒸馏；planner属于作者为实验搭建／集成的高层栈，包含复用VLM／LLM及视觉、几何、路径／航点、技能和夹爪工具；图中模块是系统级集合。正则解析是按预设文本模式提取物体／关系并拆分任务，未匹配时用LLM兜底；论文实际规则未发布。视觉定位确定目标像素，深度反投影结合深度和相机内外参得到3D位置，确定性工具按固定规则／数值算法生成控制目标；v1–v3明确写由作者实现具体loco-manipulation栈，具体模型型号与完整集成实现仍未披露。[v1 §3.5](https://arxiv.org/html/2606.06493v1#S3.SS5)。Q：功能上可否视为增强版AMO／HOMIE？A：同属底层运动执行层。HOMIE上层给双臂关节目标，RL负责下身；AMO以上肢关节目标、躯干姿态／高度和速度为条件，通过适配器与RL协调腿腰；HANDOFF直接接双腕位置、统一29维全身动作并整合恢复。增强方向是接口和能力整合，AMO另有显式躯干RPY等控制自由度。[AMO §III](https://arxiv.org/html/2505.03738v1#S3)、[HOMIE §III](https://arxiv.org/html/2502.13013v2#S3)。

Q：WBC能执行任意10维指令吗？A：其职责是结合本体历史跟踪可达／训练覆盖范围内的速度、高度和双腕目标；部署有速度限幅、腕工作空间约束和姿态修正，超范围或相互冲突的目标可产生误差或失稳。Q：语言子任务如何成为10维命令流？A：视觉定位与深度给骨盆系目标，路径／航点跟踪器将导航目标转成速度；近目标由skill selector生成高度与双腕位置，执行中滚动更新。公开辅助代码还能用FK测得的夹爪中心误差做手目标PD修正。原文未充分披露搜索策略、站位选择、轨迹生成、阶段切换与闭爪判据，完整planner发布端也未开放，无法还原完整任务程序。[§III-G／IV-D](https://arxiv.org/html/2606.06493v4#S3.SS7)、[手目标修正](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/common/hand_command_pd.py)。

Q：10维中没有夹爪，如何抓取？A：公开`/g1/command`实际18维＝WBC任务指令10维＋双腕RPY 6维＋左右夹爪各1维。网络读取10维任务字段并结合步态／本体历史；腕姿态由执行层按模式修正或覆盖。夹爪标量0开／1闭由hardware node独立读取、映射到Dex1电机目标，经独立服务执行，绕过WBC的29维身体关节输出；抓取阶段的开闭时机由上层工具负责，完整触发逻辑尚未发布。[命令布局](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/common/command.py#L29)、[策略输入](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/policy/hand_policy.py#L443)、[夹爪执行](https://github.com/lzyang2000/HANDOFF/blob/main/deploy/real/hardware_node.py#L805)。

Q：rhoyn复现了什么，怎么跑通？A：其[issue #3](https://github.com/lzyang2000/HANDOFF/issues/3)／[#5](https://github.com/lzyang2000/HANDOFF/issues/5)报告将官方ONNX移植到自建C++／CUDA＋TensorRT框架，在MuJoCo-Warp和原生PhysX测试站立、行走、转向及抗扰动。当前[适配代码](https://github.com/rhoyn/teleop-walking-benchmark/blob/e957777baefaffe942e03f11c22429d93da3ef5d/policies/handoff/policy.cpp#L59)外部输入3维速度，高度固定0.78 m、双腕位置固定，构造106维当前帧＋11帧历史并映射29维动作到PD目标；无需重训或接入官方ROS2／planner。`handoff_with_arms`使用同一权重并接管全部29关节；15关节版本的双臂由测试程序随机驱动，#5报告后者零命令漂移47–50 cm/s、前者能站稳。第三方已提供WBC运行证据，任务范围未涵盖语言感知与抓取。Ubuntu构建仍需CUDA／TensorRT及两种引擎依赖、GPU架构适配；本次未编译运行。

Q：引用文献或作者其他仓库是否开放了同一planner？A：筛查v4全部51条引用及调用位置，并追查语言／规划相关项；III-G与IV-D没有外部planner来源引用，v1–v3 §3.5明确由作者实现具体栈。SayCan、PaLM-E、RT-2、OpenVLA、WholeBodyVLA、Psi0、Being-0、LeVERB属于背景／相关工作，HERO用于接口对比；未找到完整复用声明。补查[handoff-dev](https://github.com/lzyang2000/handoff-dev/tree/sonic-wbc-teacher)及正式库完整历史，部署部分相同，缺失的MolmoNode／planner_node／hy_client／graspnet_scene仍未找到。作者的[HY-Embodied fork](https://github.com/lzyang2000/HY-Embodied-0.5-X)只增加本地VLM安装说明，[Fast-FoundationStereoPose fork](https://github.com/lzyang2000/Fast-FoundationStereoPose)提供感知工具；二者尚缺HANDOFF集成层。HERO已公开控制训练、权重与MuJoCo桌面demo，demo使用[预设物体语法和已知仿真坐标](https://github.com/RunpeiDong/HERO/blob/416c4e007e2254524322a1771f267a57b6a69994/sim2sim/interactive_client/object_selection.js#L112)，所查代码未包含论文GroundingDINO／SAM／AnyGrasp完整感知集成。 相近系统中，[Being-0](https://github.com/BeingBeyond/Being-0)仅发布资料，其论文明确改造的[Cradle](https://github.com/BAAI-Agents/Cradle)公开了电脑操作agent框架；[SayCan](https://github.com/google-research/google-research/tree/master/saycan)公开桌面拾放Notebook；[LeVERB](https://ember-center-berkeley.github.io/LeVERB-Website/)目前开放数据集，闭环评测仍待发布。OpenVLA／Psi0有模型与代码，WholeBodyVLA未发布完整实现；上述材料均未发现现成HANDOFF 10D集成。


## Show-Harness — Show-Harness: Just a VLM Agent Can Play Robots

**重点关注**：False

**开源情况**：[2026-10-19] sim复现（RoboLab）；sim重新训练（LoRA）；缺少材料（完整真机部署）。[代码](https://github.com/showlab/Show-Harness)、[模型](https://huggingface.co/showlab/Show-Harness-VLMs)、[数据](https://huggingface.co/datasets/showlab/Show-Harness-Data)。

**基本信息**：阅读日期2026-10-19；arxiv上传：2026-09-09（所读v1）；一作（共同）：Yanzhe Chen、Zechen Bai、Zhijun Cao、Wenzheng Zeng；通讯：Mike Zheng Shou；单位：Show Lab，新加坡国立大学；是否中稿：未核实，官方引用为arXiv预印本。[项目主页](https://showlab.github.io/Show-Harness/)，[arxiv](https://arxiv.org/abs/2609.10522)。

**核心问题**：怎样把通用VLM的视觉与语言能力转为逐步机器人操作？用共享的离散语义动作连接VLM与确定性机器人解释器，同时支持大模型零样本控制、小模型示范微调和GUI采集。

**系统拆解**：输入语言、多视角RGB、本体状态与短历史；按[§3／图3](https://arxiv.org/html/2609.10522v1#S3)经过Perception → Reasoning → Action闭环。VLM选择细粒度动作，解释器更新笛卡尔位姿目标，再调用阻抗控制／IK／仿真控制器，动作后重观测。默认词表为六个MV方向、GRASP、RELEASE、DONE；按需扩展绕指定轴的旋转，双臂增加STILL。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Embodied Harness Architecture → Perception | Multi-View Guidance、Proprioception | 全局／腕部图像＋高度、接触、夹爪状态 → 视角使用约定与状态文本 → VLM感知上下文。 |
| Embodied Harness Architecture → Reasoning | 中央VLM；Subtask Planning、Situated Planning、Action Chunking、Adaptive Step、Visual Prompt | 任务、观测和历史 → 子任务分解、按证据分支规划、视觉标记及步长／分块选择 → 语义动作。Planner与Controller是同一VLM的不同调用角色；Situated Planning与Visual Prompt按场景启用。 |
| Embodied Harness Architecture → Action | Action History、Failure Recovery | 语义动作与执行状态 → 维护近期动作、识别空抓并重置夹爪／回退抓取阶段 → 下一轮上下文和恢复动作。 |
| Physically Grounded Semantic Action Interface → Embodiment grounding | 解释器g_E；参考视角到运动坐标映射、位姿积分、边界投影 | MV／旋转单元 → 标定步长更新6-DoF末端目标并限制工作区／单步幅度；Franka用Cartesian impedance，AgileX用IK与关节流，仿真用操作空间命令；夹爪命令单独下发。 |
| GUMI | GUI按键／按钮、动作队列、人机混合采集 | 人类键盘或GUI agent选择同一动作单元 → 同一解释器执行 → 保存动作前图像、语义动作及低层状态，用于监督训练。GUMI是采集接口，位于推理闭环之外。 |

**训练、复用与新增**：ZS默认Gemini-3.1 Pro（medium thinking），直接调用已有模型；FT默认Qwen3.5-2B，基于预训练权重做LoRA。两种模式共享动作接口，FT默认仅用任务、双视图和最近动作，无单独Planner，复杂推理实验再接Gemini子任务指令；基础模型预训练数据未在本文重新披露。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Embodied Harness Architecture → Perception | ZS读取多视角图像和实时状态，无本文单独感知训练；FT视觉编码器据论文冻结，使用双视图作为推理／监督输入。 | 把视角角色、夹爪高度、接触与步长信息转成可用上下文。 |
| Embodied Harness Architecture → Reasoning | ZS不微调；FT以动作文本交叉熵监督，语言侧线性层rank64 LoRA，约3%参数，论文称冻结视觉编码器与多模态投影器；实际[训练模板](https://github.com/showlab/Show-Harness/blob/main/train/configs/qwen3_5_2b_lora.yaml)为freeze_vision_tower:true、freeze_multi_modal_projector:false，投影器配置与论文有差异。真机164条episode、7774决策步；Franka101条＋AgileX63条联合训练。sim-to-real另用230条仿真episode、13523步（ManiSkill100条、RoboLab130条），与真机训练路径分开。附录真机40epoch、lr1e-4、batch32、256×256，2B单H200不足2h；当前模板30epoch，数据卡分别记真机40／仿真30。 | 让VLM用原生文本词表预测动作，无专用动作头／新增特殊token；同一接口连接零样本与轻量微调路径。 |
| Embodied Harness Architecture → Action | 短历史、空抓检测和回退主要为规则／状态逻辑；原文数据含19条Franka抓取恢复片段，用于FT监督，并非在线RL。 | 将可观测失败反馈接回动作选择；子任务完成由VLM依据图像判断，恢复针对抓取状态。 |
| Physically Grounded Semantic Action Interface → Embodiment grounding | 坐标映射、位姿积分、IK、阻抗控制及边界限制无本文学习训练；需要平台与相机标定。默认细步2cm／粗步4cm，精细实验改1cm；原语语义依赖书面方向约定。 | 将模型共享动作与平台控制拆开；更换机器人时实现对应解释器。代码还要求AgileX混合训练权重部署时交换MV_FWD／MV_BACK。 |
| GUMI | 人类键盘＋frontier VLM经浏览器采集；同一示范保存语义动作与连续轨迹。π0.5／GR00T基线用这些示范转换的连续末端轨迹微调。终止样本等处理后，[发布数据卡](https://huggingface.co/datasets/showlab/Show-Harness-Data)记真机7933训练样本、仿真13753，与原始决策步分别计数；任务数论文19、数据卡17，存在口径差异。 | 无专用遥操作硬件采集，可人工纠正agent轨迹，统一采集与策略输出接口。 |

[§5.3–5.4](https://arxiv.org/html/2609.10522v1#S5)精细任务改用1cm步长后，ZS成功率60%→82%、FT40%→65%；推理任务ZS配Situated Planning为85%，FT单独10%，接同一Gemini子任务指令后70%。跨具身FT在两种真实机械臂示范上联合训练，未验证只训一种机械臂即可迁移任意新平台。动作表示消融中，有明确物理约定的任意符号接近语义名称；无名称语义且无约定时仅1/20成功，表明方向契约与视觉反馈是关键。§6将人形、灵巧手和更丰富触觉／力反馈列为后续方向；当前实验为固定基座单／双臂平行夹爪操作。对本课题，可研究细粒度动作接口与技能级工具的粒度取舍，人形行走、平衡与全身接触控制仍需另建执行层。

已检查[仿真说明](https://github.com/showlab/Show-Harness/blob/main/docs/simulators.md)、RoboLab运行脚本／配置、短指尖USD、外部RoboLab资产来源、Qwen3.5-2B可访问基座、6组LoRA文件及real/sim数据、转换与训练入口；材料支持RoboLab仿真复现与LoRA重新训练，本次未安装运行。ManiSkill目前仅发布训练数据，仓库明确未含其runner／场景。[Franka说明](https://github.com/showlab/Show-Harness/blob/main/docs/franka.md)要求NUC上的自定义Polymetis／ZeroRPC franka_server，仓库未找到该服务端；[Piper说明](https://github.com/showlab/Show-Harness/blob/main/docs/piper.md)还依赖外部cobot_magic及名为Piper_ros_private-ros-noetic的驱动工作区，其可取得性未核实，因此完整真机部署暂记缺少材料。


**关键 Q&A（2026-10-19）**：Q：与VLA有什么区别，为何强调harness？A：本文对照π0.5／GR00T等学习连续动作的VLA；Show-Harness让模型输出有明确物理约定的离散语义微动作，由平台解释器确定性转换成末端目标并调用控制器。VLA也可使用离散token，故关键在语义约定、动作粒度和平台映射位置。ZS直接复用frontier VLM；FT经机器人示范LoRA，从广义“视觉＋语言→动作”定义看也可归入VLA策略，具体实现为语义动作预测。harness强调模型外部的观测组织、提示、规划、历史、步长／分块、失败恢复与解释器；本文主要贡献落在接口和运行框架，两种模型模式共享这一层。默认FT保留最小上下文，ZS使用完整插件组合；示例MV_LEFT→标定位移→阻抗／IK执行，区别于一次调用完整抓取技能。VLA也能接入harness，两者描述模型能力与运行系统两个层面。依据[§3–4、§5.1与附录7.4](https://arxiv.org/html/2609.10522v1)。


Q：harness是否指完成任务的系统闭环，π0.5／GR00T也有相关成分吗？A：harness是模型外部运行系统；基本控制闭环为观测→策略推理→执行一段动作→重新观测，任务层进一步组织子目标、成功检查、失败恢复和终止。openpi与Isaac-GR00T部署代码均提供观测处理、策略服务／客户端、动作执行与rollout等运行成分，可视作基础harness。π0.5原文还学习高层语义子任务，再条件生成低层动作块，需区分这项模型能力与外围运行机制；GR00T N1的System 2 VLM＋System 1动作生成属于模型架构。具体部署具备哪些记忆、独立验证或恢复模块，应逐项核对。Show-Harness将这些外围接口／运行机制作为主要研究对象，VLA研究主要围绕模型与训练。依据[π0.5原文](https://arxiv.org/html/2504.16054v1)、[openpi运行说明](https://github.com/Physical-Intelligence/openpi/blob/main/docs/remote_inference.md)、[GR00T N1原文](https://arxiv.org/html/2503.14734v1)、[当前Isaac-GR00T策略指南](https://github.com/NVIDIA/Isaac-GR00T/blob/main/getting_started/policy.md)；后者是当前工程实现，版本与Show-Harness引用的N1分开看。
## CHOREO — CHOREO: Every Humanoid Skill as a Trajectory

**重点关注**：False

**开源情况**：[2026-10-19] 完全没开源（系统实现）；[官方仓库](https://github.com/ziyisun85-ops/Project-choreo)。

**基本信息**：阅读日期2026-10-19；arxiv上传：2026-09-11（v1）；一作（共同）：Ziyi Sun、Jingwen Chen；通讯：Yuxi Wang、Xiuze Xia（据官方主页）；单位：中国海洋大学工程学院／信息科学与工程学部、中国科学院大学人工智能学院、中国科学院自动化研究所；是否中稿：未核实，作者个人主页标Under Review，未注明会议。[项目主页](https://ziyisun85-ops.github.io/Project-choreo/)，[arxiv](https://arxiv.org/abs/2609.22274)。末位作者HTML／主页为Junyu Dong，arXiv摘要页元数据为Yujun Dong，采用正文署名。

**核心问题**：异构策略、动作数据和生成模型的接口各异；作者将它们的运动输出统一为SkillMotion轨迹资产，通过边界匹配、局部插值和站立桥接实现免重训的人形多动作组合。

**系统拆解**：按[方法／图2](https://arxiv.org/html/2609.22274v1#Sx3)，离线由Source Adapter转换并验证运动，存入SkillMotion库；在线自然语言→LLM排序→运行时解析资产与构造衔接→跟踪器适配→机器人状态反馈下的低层跟踪。原始策略／生成器只在离线产出轨迹，评测在线阶段使用已注册资产。SkillMotion是数据表示；下表保留原文结构及父子关系。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Offline Registration → Source Adapter／validation gate | 源特定导入、重定向、规范化、注释、验证与隔离 | 动作片段／原环境中的RL rollout／生成模型输出 → 统一23-DoF G1、30Hz、关节顺序与坐标，补速度和接触置信度，检查格式、一致性与可跟踪性 → 合格资产入库，失败项隔离。 |
| SkillMotion | τ、z、入口／出口边界b、执行描述e | 关节位置／速度、根状态、足接触＋语义／来源＋边界窗口＋具身／Tracker配置／验证证据 → 可检索、可拼接的轨迹资产。 |
| LLM task planner | 有序技能标签／别名；runtime asset resolution | 自然语言 → 语义技能排序 → 解析到已注册SkillMotions；LLM不直接产生关节控制。 |
| Tracker-Aware Skill Composition and Execution → boundary compatibility／local transition construction | 平面位置和朝向对齐；边界评分；local seam；stable-standing bridge | 相邻轨迹 → 比较关节位置／速度、根高度／线速度与置信加权足接触 → 合适时局部衔接，间隙大时插入预验证站立参考，再连接两端 → 组合参考轨迹。 |
| Tracker-Aware Skill Composition and Execution → tracker-specific execution | reference adapter、observation adapter、frozen Tracker | 组合参考＋当前机器人状态 → 坐标转换、重采样、参考窗口和Tracker观测 → GMT等跟踪策略输出控制量 → 仿真执行。主实验GMT固定、50Hz；最终动作缩放、PD／力矩接口细节未披露。 |

**训练、复用与新增**：training-free适用于本文注册、组合与执行流程；本文未报告新增学习训练或测试时参数更新，底层跟踪器及上游模型具有既有训练经历。具体冻结与复用情况如下。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Offline Registration → Source Adapter／validation gate | 确定性转换／修正和验证，无新增学习训练。来源为2969条动作库轨迹、50条预训练RL策略rollout、50条扩散生成动作；具体动作库、RL／扩散模型、checkpoint与原训练数据未披露。它们用作注册／测试资产。 | 以运动输出接入异构来源，并保存验证证据，隔离失败资产。 |
| SkillMotion | 数据结构，无可训练参数；缺失接触可由运动学推断，无法获得时标unknown且置信度为0。摘要称2950条已准入资产，正文另报3069条导入实例，两者对应关系未解释。 | 统一轨迹、语义、接触、入口／出口和执行契约。 |
| LLM task planner | 复用LLM，具体型号、checkpoint、冻结配置与是否曾做任务微调未披露；本文未报告训练。评测前固定prompt plans，在线重规划关闭。 | 用技能标签／别名连接自然语言计划与资产注册表。 |
| Tracker-Aware Skill Composition and Execution → boundary compatibility／local transition construction | 手工特征、权重、尺度与阈值，加插值／桥接，无本文训练；站立参考预验证。预验证站立资产后，新接缝仍须通过实际序列执行检验。 | 根据边界状态选择衔接方式；正文公式(4–6)用五次多项式匹配位置、速度、加速度；摘要／引言／主页称cubic Hermite，存在表述差异，待实现澄清。 |
| Tracker-Aware Skill Composition and Execution → tracker-specific execution | GMT用于主实验与来源测试，参数冻结；兼容性测试另复用OpenTrack、TWIST2、HoloMotion、SONIC、H-ACT，各冻结。具体权重版本、原训练数据及观测配置未完整披露；本文未重训。 | 按Tracker构造参考与观测适配器；主实验每条序列固定GMT，tracker-aware routing关闭。 |

[实验／限制](https://arxiv.org/html/2609.22274v1#Sx4)：全部为MuJoCo G1仿真；130条固定任务、最长8动作、552个计划边界，固定seed，每任务一次。完成124/130（95.4%），8动作45/48（93.8%），切换成功96.7%；最强8动作基线为固定站立桥41.7%。所有方法共享冻结GMT和任务资产，主要比较衔接设计；执行恢复、在线重规划、跟踪器路由均关闭。来源测试3069条通过导入检查，其中356条做闭环rollout、353条成功（99.2%）；多Tracker测试仅30条共享轨迹、每条最多3秒。尚无真机实验、重复试验方差或物体操作任务成功指标。消融中去掉连续性排序或bridge validation均得到8动作95.8%，高于Full的93.8%；单项贡献需结合跌倒率、其他长度与重复试验判断。

[官方仓库README](https://github.com/ziyisun85-ops/Project-choreo/blob/main/README.md)明确为静态项目页；检查完整文件树及链接，发布内容为HTML/CSS/JS、PDF、图像和视频。唯一smoke_test.py检查页面本地资源引用，未提供Source Adapter、SkillMotion资产、运行／评测入口、Tracker权重与配置、仿真环境或真机部署包；截至核验日材料无法支持本文系统sim复现、sim重新训练或直接部署。本次仅阅读核验，未运行。

## HarnessPro — Recursive Harness Self-Improvement with Pareto Optimization for Performance-Cost Trade-offs

**重点关注**：False

**开源情况**：[2026-10-22] 缺少材料（完整自演化流程、GLM版快照）；[官方代码](https://github.com/pzs19/HarnessPro)。

**基本信息**：阅读日期2026-10-22；arxiv上传：2026-10-19（所读v2：2026-10-20；正文、附录A／B与代表性实现）；一作（共同）：Zhuoshi Pan、ZhiYuan Feng；通讯：H. Vicky Zhao、Jie Tang；单位：清华大学、Z.AI、上海交通大学、华威大学、Kling AI；是否中稿：审稿中，原文标Under review，会议未注明。[项目主页](https://pzs19.github.io/HarnessPro/)，[arxiv](https://arxiv.org/abs/2610.26770)。用户从[微信推文](https://mp.weixin.qq.com/s/TtjRoi5vYL5zrY0NZFbi5w)补充原论文链接；本次依据原论文与代码，推文正文未读取。

**核心问题**：让编程智能体根据历史执行轨迹自动改进harness，同时优化任务通过率和调用成本，输出适配不同预算的lite／medium／heavy方案。

**系统拆解**：按[§3–4／算法1](https://arxiv.org/html/2610.26770v2#S4)，优化对象包括指令、skills、工具、上下文机制、hooks和验证规则。先从初始harness的执行生成固定测试；随后循环“运行任务→诊断→提出三类修改计划→编辑候选→验证与筛选”，每轮最多保留三个候选。下面四项为原文agent角色，最后一项为选择算法；最终部署产物是选中的harness。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Trajectory Analyzer | 多次rollout联合诊断 | 任务、执行轨迹、测试结果、token／成本／时延 → 定位失败与冗余开销 → 带证据路径的诊断。 |
| Evolution Planner | 成本优先／均衡／性能优先三类指令；候选间代码差异比较 | 诊断、指标、harness实现及同轮候选差异 → 三份通用修改计划。 |
| Harness Editor | 独立工作区、Git版本管理 | 父harness＋计划＋诊断 → 修改指令、工具和运行规则 → 可执行候选。 |
| Test Synthesizer | 根据任务生成可执行测试 | 任务说明＋初始执行轨迹 → 一次生成测试并固定，供后续候选统一评估；最终测试集使用官方评测。 |
| Candidate filtering and selection（算法） | 初始通过率门槛、Pareto筛选、归一化加权平方距离 | 验证集通过率与成本 → 去除低于初始水平及被其他候选支配的方案 → 选择三档代表，进入下一轮。 |

**训练、复用与新增**：方法通过LLM调用和代码搜索更新harness，未报告大模型参数训练／微调。实验使用GPT-5.5＋Codex 0.147.0、GLM-5.2＋Claude Code 2.1.231。数据来自Terminal-Bench 2、SWE-Bench Pro、ProgramBench的任务说明与自行执行的轨迹；每个基准用DPP选10个训练任务，每候选每任务每轮运行3次；其余开发任务用于验证，最终测试分别留出59／100／100题。前两基准演化3轮，ProgramBench为4轮。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Trajectory Analyzer | 复用LLM；输入为训练任务的轨迹与指标，超长诊断再摘要。 | 同时解释失败及开销，并给出可追溯证据。 |
| Evolution Planner | 复用LLM；使用诊断、候选代码和指标，验证／测试题对planner隐藏。 | 三种成本—性能偏好的计划，以及候选间经验共享。 |
| Harness Editor | 复用编码agent；更新软件文件，搜索从现成Codex／Claude Code harness开始。 | 计划落地、隔离候选和可回溯版本。 |
| Test Synthesizer | 复用LLM从任务及初始轨迹合成测试；演化时使用这些测试提供反馈，官方答案与测试保留给最终评测。 | 用固定可执行测试检验候选产物；论文初步实验中失败召回率由直接LLM判分的41.7%提高到83.3%。 |
| Candidate filtering and selection（算法） | 确定性规则，无参数训练；输入为验证集统计。 | 性能门槛＋Pareto筛选；三档性能／成本权重为(1,2)、(1,1)、(2,1)。 |

[表2／§5.3](https://arxiv.org/html/2610.26770v2#S5)：GPT-5.5下，Heavy相对RHO在三个基准平均提高2.0个百分点、降低单任务成本26.1%；Terminal-Bench 2为76.3%→80.8%、每题$1.09→$0.72。报告成本覆盖最终harness执行任务的模型调用；前期演化费用与摊销周期待评估。ProgramBench报告的是逐任务测试通过比例的宏平均。主要改进包括任务条件的推理／工具预算分配、分阶段hooks、可恢复的工具输出压缩和指令精简。

[发布说明](https://github.com/pzs19/HarnessPro/blob/main/docs/evaluation.md)已提供9套GPT-5.5演化产物、三基准评测入口和运行适配器；完整递归演化入口未见，GLM版快照明确未发布。运行需Linux、Docker、模型API及另取基准数据／镜像；复用RHO运行代码、ProgramBench／Pier评测与RTK。本次仅静态核验，未运行。研究对象为软件任务；对人形harness可借鉴轨迹诊断、离线候选评测和多目标选择，机器人侧需另定义成功检查、物理执行成本及安全约束。

## WBD — Whole-Body Dexterity

**重点关注**：False

**开源情况**：[2026-10-26] 缺少材料（完整训练与真机部署包）；[项目主页](https://wholebodydex.com/)。

**基本信息**：阅读日期2026-10-26；arxiv上传：2026-10-15（v1；20页，已核对正文方法／实验及附录A–G的相关细节）；一作（共同）：Conor Mc Gartoll、An Dang、Pranay Thangeda；通讯：未明确标注，Mustafa Mukadam、Manikantan Nambi标为共同指导；单位：Amazon、加州大学圣迭戈分校、密歇根大学；是否中稿：未核实。[项目主页](https://wholebodydex.com/)，[arxiv](https://arxiv.org/abs/2610.22184)。由未来机器人Lab的[小红书笔记](https://xhslink.cn/o/65PaKZE7JKf)定位；以下结论依据论文原文与官方项目页。

**核心问题**：让机器人利用双臂与躯干的接触、滑动和滚动，在身体不同区域之间稳定转移物体，并将人类粗略动作修正为可靠的接触操作。

**系统拆解**：依据[原文§III–IV与附录G](https://arxiv.org/html/2610.22184v1)，离线流程为抓持生成→逐物体RL专家→统一控制器蒸馏；真机流程为Xsens人体动作→GMR上肢目标→WBD修正→关节执行，AMO负责下肢与平衡。专家训练中腰与下肢固定，控制14个手臂关节；手指固定为闭合姿态。本文学习的接触操作范围是手臂／躯干，腿部接触为后续方向。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Whole-Body Grasp Generation | 接触区域划分、能量优化、仿真扰动筛选与体素化 | 机器人模型、物体网格、接触区域 → 优化并验证稳定性 → 左臂／双臂／右臂抓持库及目标图。 |
| WBD-EX Training | 目标条件PPO；球体预训练→其他物体warm-start | 特权状态、物体目标位置与抓持模式 → 学习连续换抓 → 每物体专家；左右单臂换持须经过双臂模式。 |
| WBD Controller Distillation → trajectory collection | escapability初态筛选、冻结专家rollout、轨迹质量过滤 | 稳定抓持库＋已训专家 → 按换抓成功率筛选初态，采集并清理轨迹 → 蒸馏数据。 |
| WBD Controller Distillation → conditional diffusion model | 本体历史编码、物体几何条件、命令梯度引导 | 本体历史＋5维几何描述＋粗略动作命令 → 多候选去噪与选择 → 未来手臂关键点位移。 |
| WBD Controller Distillation → inverse-dynamics network | 残差网络、关节增量与执行限幅 | 历史状态＋生成的关键点运动＋几何 → 14维手臂关节目标增量。 |
| 真机接入（§IV-C／附录G-F）→ GMR／AMO | 人体动作重定向、独立下肢控制、命令发布 | Xsens动作 → GMR上肢参考供WBD修正；AMO下肢平衡／行走 → G1执行。 |

**训练、复用与新增**：学习数据主要由作者的仿真流程产生，真人遥操作用于真机测试。部署的WBD控制器经仿真训练后零样本迁移到G1；输入包含本体状态和预先测量得到的物体几何描述，专家训练可读取物体位姿、速度及接触信息，蒸馏控制器在线使用本体历史与静态几何描述。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Whole-Body Grasp Generation | 几何优化与物理筛选，无学习参数；15个球／立方体／棱柱／圆柱，尺寸80–240毫米，得到251万稳定抓持。 | 躯干分区、接触模式与稳定性筛选，形成可供RL采样的初态／目标。 |
| WBD-EX Training | 在mjlab中训练PPO专家：180／200／220毫米球体从头训练，其余物体以自训球体专家的actor、critic、优化器、归一化器及课程状态warm-start；在线仿真生成经验，随机化质量、质心、摩擦及外力。 | 按物体位置与抓持模式奖励连续换抓；逐渐增大目标距离。 |
| WBD Controller Distillation → trajectory collection | 冻结专家；对251万抓持各采样128个目标评估换抓能力，再按物体筛掉10%–60%的低分初态；rollout质量过滤后约27.4万轨迹、8.73亿帧。 | 保留空间／模式覆盖，提高可用轨迹比例29.0%→32.5%；失败轨迹可保留合格前缀，丢弃失败前100帧。 |
| WBD Controller Distillation → conditional diffusion model | 采用DexterityGen的扩散／状态编码器配置并适配；本文训练，未报告复用其预训练权重。使用上述仿真轨迹，按物体均衡采样并做左右镜像；4帧本体历史＋5维几何共269维输入，预测未来100／200毫秒关键点位移。 | 跨物体几何条件与本体历史，学习稳定换抓运动先验；粗命令引导推理。 |
| WBD Controller Distillation → inverse-dynamics network | 与扩散模型联合训练13万次AdamW更新；监督来自专家实际关节目标增量，输入混合专家和扩散生成运动；未报告外部checkpoint初始化。 | 将关键点运动转为可执行关节增量。两网络合计每个控制器约22 H100 GPU小时；推理每次生成16个候选，使用12步DDIM。 |
| 真机接入（§IV-C／附录G-F）→ GMR／AMO | 复用GMR、AMO与Xsens；本文未报告重新训练AMO，具体权重版本未披露。SONIC公开checkpoint用于对照。 | 接通遥操作、WBD上肢修正与下肢控制；实时推理使用操作者工作站RTX 5090。 |

真机实验由一位接受过8小时以上任务训练的操作者执行：篮球与15厘米纸箱各做肩部触碰、腋下夹持往返，每项10次；另做10次纸箱货架到桌面搬运。综合WBD为45/50（90%），GMR为9/50（18%），SONIC为2/50（4%）；完整搬运链分别7/10、1/10、0/10。G1部分手臂包覆防滑胶带。90%适用于这组遥操作任务。仿真泛化测试以13种物体训练、留出180毫米球体与立方体，三次训练种子；真机控制器使用全部15种物体训练。专家能力随物体变化显著，200毫米立方体及两种较大棱柱的完整绕身loop成功率接近零，表VIII给出分项结果。官网提供交互专家演示，所查官网和论文未给出完整代码仓库、训练数据下载或真机部署入口，代码仓库未能确认；未运行复现。对本项目可作为“粗动作参考→接触稳定修正”的技能后端参考，高层目标与任务编排需另行设计。

[附录E／F／G](https://arxiv.org/html/2610.22184v1)：代表性8卡H100运行中，球体预训练1.5万轮约270 GPU小时，各目标物体后训练3.5万轮约660–815 GPU小时，从头训练5万轮约1,030 GPU小时；270 GPU小时可在多个目标间摊销。上述为专家训练，抓持生成、约3.2亿次换抓评分和轨迹采集另有开销。warm-start比较每条件一个训练种子；初态筛选的部分路线提升采用选阈值的同组试验，独立采集比较提供29.0%→32.5%的可用率依据。本文核心贡献是接触模式约束下的连续换抓训练与可部署的动作修正控制器，高层任务目标仍由操作者提供。

## SuperMimic — Scaling Humanoid Whole-Body Control with One Differentiable World Model

**重点关注**：False

**开源情况**：[2026-11-05] 缺少材料（官方 sim复现、sim重新训练、G1直接部署包）；[官方交互演示](https://supermimic.github.io/demo/)。

**基本信息**：阅读日期 2026-11-05；arxiv上传：2026-10-20（v1，正文及附录A–G）；一作：Qiayuan Liao，Yiyang Shao／Junfeng Long标注等贡献；通讯：未明确标注，Pieter Abbeel／Koushil Sreenath标注共同指导；单位：UC Berkeley；是否中稿：是，CoRL 2026 Oral（arXiv及作者主页）；[项目主页](https://supermimic.github.io/)，[arxiv](https://arxiv.org/abs/2610.28480)，[论文原文](https://arxiv.org/pdf/2610.28480)。

**核心问题**：大规模动作跟踪的无模型RL训练昂贵；用一个跨动作共享的可微动力学模型生成短轨迹并反向传播，低成本训练统一人形全身控制策略。

**系统拆解**：依据[原文§3／图2、附录A–D](https://arxiv.org/pdf/2610.28480#page=4)，训练循环为 Collect data → Fit world model → Improve policy。执行时，未来10帧动作参考经motion encoder变为32维latent，与最近10帧本体观测送入shared decoder，输出29维归一化关节设定值，再映射到PD关节目标。world model与critic用于训练；真机运行跟踪策略。额外研究velocity encoder，将速度命令接到同一decoder。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Collect data | MuJoCo Warp、固定容量replay buffer、return-based reset sampling | 当前策略＋参考动作 → 加动作噪声执行、优先练习低回报片段 → 状态／实际动作／随机化上下文及监督标签。 |
| World model | Dynamics MLP＋reward／observation／termination heads，heads可学习或解析计算 | 状态、动作、摩擦／标定偏移／躯干质心等上下文 → 预测下一状态、奖励、无噪观测及终止概率。 |
| Policy → Motion encoder | MLP (2048,1024,512,256)，tanh latent | 未来10帧参考 → 32维动作latent。 |
| Policy → Shared decoder | MLP (2048,1024,512,256) | latent＋最近10帧本体观测 → 29维动作 → 默认姿态＋动作缩放 → PD执行。 |
| Critic | 每接口一个MLP (256,128)，GAE回报目标 | 预测状态、目标、仿真上下文 → 32步想象窗口以后的回报估计。 |
| Policy → Velocity encoder（下游初步实验） | MLP (256,256)，共享decoder和world model | 目标(vx,vy,ωz)＋本体历史特征 → 32维latent → 相同decoder与执行接口。 |

**训练、复用与新增**：上述学习网络由本文训练，未报告加载外部预训练checkpoint；具体随机初始化方案未披露。训练数据分为动作参考与在线仿真转移；未报告使用真机数据拟合world model。依据[§3、§4.5、附录B–G](https://arxiv.org/pdf/2610.28480#page=5)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Collect data | 复用MuJoCo Warp和G1重定向动作；BONES-SEED原始142,220段经名称／物理可行性筛选，保留129,514段、264.56小时。4096并行环境／GPU生成在线转移；LAFAN1的40段、2.45小时用于受控对照和消融。 | 50%均匀、40%当前低回报、10%历史困难权重的重置采样；有界buffer保留旧策略经验。 |
| World model | 本文持续更新；动力学MLP为(2048,2048,2048)。以buffer中8步自回归窗口监督状态与适用heads；heads逐项采用学习还是解析实现未完整列明。 | 跨所有动作共享动力学；显式输入仿真随机化参数，观测噪声在模型外添加，终止概率提供可微存活权重。 |
| Policy → Motion encoder | 本文训练；参考来自上述动作库。与decoder通过32步想象轨迹的回报梯度联合更新，采用BeyondMimic跟踪奖励的修改版。 | 用未来10帧参考编码目标；BPTT穿过固定权重的world model获取策略梯度。 |
| Policy → Shared decoder | 本文训练；监督信号来自想象回报，经验由当前策略的仿真采集产生。motion-tracking目标更新decoder；下游velocity目标对decoder权重stop-gradient。 | 单一decoder统一动作执行；不经过逐动作专家再蒸馏的主训练流程。 |
| Critic | 本文训练；目标来自想象轨迹的GAE回报，γ=0.99、λ=0.95；每接口独立critic。策略更新时其权重固定，随后拟合价值目标。 | 用窗口末端价值补足长期回报，支持短窗口学习与受扰后的恢复。 |
| Policy → Velocity encoder（下游初步实验） | 与motion接口并行训练：RL版用速度跟踪奖励，并以参考命令rollout作latent对齐；蒸馏对照版从参考动作读取速度、用跟踪奖励训练。共享decoder持续由motion目标更新，velocity目标仅更新该encoder。 | 直接用接口任务奖励学习速度控制。在672个速度阶跃响应中，RL版650项稳态误差更小，对照15项更小、7项跌倒。 |

[§4.1／附录F](https://arxiv.org/pdf/2610.28480#page=6)：单H100训练3小时超过SONIC公开checkpoint的归一化回报，失败25／129,514段；双H100训练9.8小时为0段，SONIC为179段。3小时含采样、模型与策略／critic训练，checkpoint评测另计。SONIC报告128张L40训练168小时、611小时动作数据；本文使用264.56小时。评测用两者训练集中的共同动作，每段从首帧执行一次，摩擦1.0、无随机化与观测噪声，以本文根部高度／倾斜阈值判失败，衡量训练动作覆盖。LAFAN1受控对照中，本文单卡1.2小时达到0.82回报，PPO-Sim在18小时预算内未达到。

[§4.4／§6](https://arxiv.org/pdf/2610.28480#page=8)：同一策略在G1展示跳步、侧滚、90°及360°翻转，无逐动作调参；未给出真机重复试验成功率。训练限平地，地形与物体交互列为后续工作。对当前研究，可作为参考动作／速度条件的全身执行后端候选；任务语义、物体状态与高层技能编排仍需设计接口。

官方项目页与一作主页未链接本文训练仓库；[演示许可](https://supermimic.github.io/demo/LICENSE.html)将策略／世界模型及演示程序列为专有材料，允许官网浏览器中的个人非商业交互，限制提取和复用。未找到完整训练入口、可复用checkpoint及真机部署说明；[BONES-SEED](https://huggingface.co/datasets/bones-studio/seed)提供G1数据，下载须登录接受条款。本条开源结论针对官方材料；第三方实现的局部代码核对见下方Q&A。本次静态核验，未运行仿真或部署。

**关键 Q&A（2026-11-05）**：Q：model是动力学世界模型，policy负责动作，两者像PhysHSI的AMP与policy一样共同训练吗？A：两者在同一训练过程内持续更新；所查[第三方训练循环](https://github.com/CuboiLeo/SuperMimic-Reproduced/blob/main/src/supermimic/training/loop.py#L190)按采样→更新world model→更新policy执行。[策略更新](https://github.com/CuboiLeo/SuperMimic-Reproduced/blob/main/src/supermimic/imagination/policy_trainer.py#L66)暂时固定world model与critic权重，沿想象轨迹反向传播，只更新policy，随后恢复并训练critic。世界模型学习状态转移，帮助计算动作后果及梯度；PhysHSI的AMP判别器学习参考／策略动作分布，提供风格奖励，policy以PPO更新，风格奖励在no_grad下计算。两者共同迭代的关系相似，辅助模型的职责与策略梯度路径不同。此第三方仓库为独立复现，本次只核对训练循环、模型／策略训练器与想象rollout，未运行。

Q：GT、WM监督与并行rollout分别是什么？A：所查第三方实现的[跟踪奖励](https://github.com/CuboiLeo/SuperMimic-Reproduced/blob/main/src/supermimic/envs/mdp/rewards.py#L101)为手写的身体位姿／速度参考误差评分与正则项，policy最大化想象轨迹累计回报；[WM监督](https://github.com/CuboiLeo/SuperMimic-Reproduced/blob/main/src/supermimic/world_model/world_model.py#L65)为MuJoCo实际转移状态与终止标签，用8步自回归预测误差训练。动力学主干输入状态、动作和仿真上下文，人体参考用于目标／奖励及终止判断。物理采集使用MuJoCo Warp／mjlab，[默认4096环境](https://github.com/CuboiLeo/SuperMimic-Reproduced/blob/main/src/supermimic/config.py#L25)，持续补充replay buffer；policy更新从buffer起点并行展开4096条×32步WM轨迹。两层都需要rollout，效率来自经验复用、神经动力学推演和穿过模型的动作梯度；相邻时间步依赖上一状态，多条轨迹可并行。PhysHSI则主要在Isaac Gym中采集物理轨迹，以任务／AMP奖励通过PPO更新策略。

Q：更新policy时奖励用实际状态还是WM预测，rollout指什么？A：rollout是从初态开始，反复生成动作、推进状态并记录奖励得到的一段轨迹。所查[imagine循环](https://github.com/CuboiLeo/SuperMimic-Reproduced/blob/main/src/supermimic/imagination/rollout.py)先从buffer取实际仿真起点，之后每步把WM预测状态反馈给policy；[ImagStep](https://github.com/CuboiLeo/SuperMimic-Reproduced/blob/main/src/supermimic/imagination/compiled.py#L38)先wm.step得到s_next，再由FK和手写奖励函数比较该预测状态与ref，产生想象奖励。本循环先算奖励、再推进参考帧；不调用MuJoCo的env.step。32步预测奖励、生存权重与末端critic价值构成目标J，loss=-J/H，经backward更新policy；固定WM参数仍保留预测对动作的梯度。

Q：Policy多大，结构由复现者自定吗？A：论文附录C表A.4给出motion encoder与decoder各用(2048,1024,512,256)隐藏层、32维latent。第三方实现为680→2048→1024→512→256→32与962→2048→1024→512→256→29，合计8,891,197参数；激活、初始化等未披露细节由实现补充。执行只需policy。4090服务器实测同结构FP32、batch=1、输入已在设备上，500次前向中位延迟0.114毫秒；CPU单线程0.793毫秒。范围为网络前向。

Q：另外两个SuperMimic仓库改了什么？A：[shuayyy/humanoid-amp-ppo](https://github.com/shuayyy/humanoid-amp-ppo)独立实现SuperMimic，并保留另一套AMP-PPO；将WM状态从71维扩展到含箱体位姿／速度的84维，增加箱体观测、参考、位姿奖励、终止条件与质量／摩擦随机化，比较walk／squat／lift上的SuperMimic与PPO-Sim。核心仍为8步WM监督与32步想象BPTT；采样采用失败驱动，区别于论文回报驱动。其结果文档注明旧评测两算法观测噪声不同，修正后尚未重评。[NJU-R-L-Group-Embodied-Lab/SuperMimic-Repro-Sim2Real](https://github.com/NJU-R-L-Group-Embodied-Lab/SuperMimic-Repro-Sim2Real)基于CuboiLeo复现，增加BONES-SEED数据处理／训练、GVHMR→GMR视频动作管线、自定义舞蹈微调、ONNX与G1部署（50 Hz policy、500 Hz命令、C++ DDS bridge）、sim2sim与日志检查；针对真机增加腰部耦合、延迟／阻尼／IMU饱和建模及奖励与reset选项。发布基础／舞蹈模型；v1.1单舞蹈策略加入87个真机失败状态作仿真reset，报告同checkpoint连续完成两次，父checkpoint未发布且无逐项消融。[实现审计](https://github.com/shuayyy/humanoid-amp-ppo/blob/master/docs/supermimic/paper_fidelity.md)、[真机训练配方](https://github.com/NJU-R-L-Group-Embodied-Lab/SuperMimic-Repro-Sim2Real/blob/main/scripts/train/train_lean35kfs_e24.sh)。2026-11-05核对文档与核心源码，未运行这两个仓库。

Q（2026-11-08）：小红书评论说SuperMimic与VLA解决不同问题，是什么意思？A：前者主要学习参考条件的全身动作跟踪，VLA从视觉与语言生成任务动作；帖子问的是训练范式迁移。可微动力学与想象优化可作为操作策略训练候选，须验证物体／接触建模、观测生成、奖励与迁移效果，现有动作跟踪加速结果尚未覆盖多数VLA任务。来源：[原帖](https://xhslink.cn/o/8oSVWyhgT4K)、[SuperMimic论文](https://arxiv.org/abs/2610.28480)、[OpenVLA](https://openvla.github.io/)。

## PhysHSI — Towards a Real-World Generalizable and Natural Humanoid-Scene Interaction System

[论文](https://arxiv.org/html/2510.11072v1) · [项目主页](https://why618188.github.io/physhsi/) · [代码](https://github.com/InternRobotics/PhysHSI) · 2026-11-16

**核心问题** 如何用少量人体示范训练全身策略，让 G1 在物体尺寸和位置变化时，仍能自然地搬箱、坐下、躺下和起身。

**推理数据流** 人工粗定位 + LiDAR 里程计 + 近距离 AprilTag → 物体位姿。物体位姿、尺寸、任务目标和本体观测的历史 → Actor → 29 维动作 → 缩放后加到默认关节角 → PD 计算力矩 → G1 执行。

**训练数据流** AMASS／SAMP 人体动作 → 重定向到 G1、标注物体轨迹 → 参考片段。参考片段与 Isaac Gym 在线采样的片段 → 训练 AMP 判别器 → 风格奖励与任务、正则奖励合并 → PPO 按任务更新 Actor 和 Critic。参考片段也用于初始化部分仿真回合。

**开源情况** 仿真可推理、可重新训练（6 项任务，已公开权重、训练代码和参考动作数据）。真机复现缺 G1 部署入口和 FAST-LIO／AprilTag 集成代码。

**作者** 王华艺、张文涛（并列一作）。通讯作者为陈启峰、王靖博、庞江淼。署名单位为上海人工智能实验室、香港科技大学。

**录用** 第一作者标为在审，未查到公开录用记录。

## PHP — Perceptive Humanoid Parkour: Chaining Dynamic Human Skills via Motion Matching

**重点关注**：False

**开源情况**：[2026-11-05] sim复现（发布学生示例）、sim重新训练（公开动作库与训练配方）；缺少材料（完整论文技能checkpoint、G1真机部署说明）；[官方代码](https://github.com/amazon-far/php_parkour)。

**基本信息**：阅读日期 2026-11-05；arxiv上传：2026-02-17（所读v2：2026-05-06，正文及附录）；一作（共同）：Zhen Wu、Xiaoyu Huang、Lujie Yang；通讯：未明确标注，项目页面另有team co-lead标记；单位：Amazon FAR、UC Berkeley、CMU、Stanford；是否中稿：是，[RSS 2026正式论文集](https://www.roboticsproceedings.org/rss22/p020.html)；[项目主页](https://php-parkour.github.io/)，[arxiv](https://arxiv.org/abs/2602.15827)。

**核心问题**：将稀少的高动态人体动作离线组合为多样的接近—越障—离开参考，再经特权专家和DAgger＋RL训练统一深度视觉策略，连续跨步、攀爬和撑越障碍。

**系统拆解**：依据[原文§III／图2](https://arxiv.org/html/2602.15827v2#S3)，离线为Skill Composition via Motion Matching → Motion-Tracking Experts → Unified Student。部署输入机载深度、本体状态及离散二维速度命令，学生直接输出29关节PD目标；技能选择和转换由统一学生根据观察产生。Motion matching在离线生成训练参考时使用。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Skill Composition via Motion Matching | 最近邻特征检索、进出技能窗口、inertialization、地形配对 | 重定向人体行走／跑酷片段＋速度命令 → Locomotion→Parkour Skill→Locomotion拼接；改变接近距离、速度、朝向与障碍 → 长参考轨迹及配对地形。 |
| Learning a Highly-Dynamic Visuomotor Policy → Motion-Tracking Experts | 每技能专家、PPO、global tracking、adaptive sampling | 参考动作、特权机器人状态＋0.7×0.7米height scan → 学习跟踪并适应地形扰动 → 专家动作。 |
| Learning a Highly-Dynamic Visuomotor Policy → Unified Student | depth CNN＋策略MLP、DAgger＋PPO及课程 | 深度图、本体观测、速度命令 → 视觉条件的统一动作决策 → 29关节PD目标，连续完成越障。 |

**训练、复用与新增**：与上表逐项对应；本文训练专家及学生，未报告使用外部预训练策略或视觉checkpoint。学生训练时专家固定提供标签；学生自身视觉编码器与控制网络参与训练。[§III-C／附录A–B](https://arxiv.org/html/2602.15827v2#S3.SS3)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Skill Composition via Motion Matching | 复用OmniRetarget将人体动作重定向到G1；motion matching为检索与拼接，无网络参数训练。原始库含495.5秒locomotion，各跑酷类别约1.5–12.1秒；手工标注技能起止及配对地形。 | 用共享locomotion连接异质技能，密集覆盖入场步态／距离／时机，随机化地形和干扰物；技能主体执行期间暂停匹配，保持接触动作。 |
| Learning a Highly-Dynamic Visuomotor Policy → Motion-Tracking Experts | 本文用PPO训练，数据为合成参考与在线仿真经验，沿用BeyondMimic／OmniRetarget的跟踪设计；未报告复用其权重。论文专家／学生各2万轮、16384并行环境；具体初始化方案未披露。 | global tracking纠正位置／时序漂移；adaptive sampling集中学习困难片段；全专家统一action scale=1。 |
| Learning a Highly-Dynamic Visuomotor Policy → Unified Student | 固定专家，在学生rollout上取动作标签；视觉用WARP渲染深度，3层CNN＋5层MLP。DAgger权重从1降至0.1，PPO逐渐增强；专家阈值外的学生状态屏蔽DAgger标签。未报告真机训练数据或策略微调。 | 成功回报纠正逐步模仿对短时大动作的低估；放宽终止阈值，保留DAgger约束动作自然度，均匀采样技能；模拟相机外参、深度噪声和60–80毫秒延迟。 |

[§IV／表I–II](https://arxiv.org/html/2602.15827v2#S4)：G1从起跳到站稳用3.63秒攀上1.25米平台；撑越示例峰值3.41米／秒；多障碍演示中可适应约0.5米障碍位移。导航仍由外部二维速度命令控制。单障碍仿真按每任务100个初始条件×5次测试，表I在1／2米每秒命令下跨36／58／76厘米障碍为100%／99%／95%；这些为仿真统计，真机主要报告动作与长链演示，未给对应重复成功率。表II中纯DAgger在1米每秒、76厘米攀爬仅3%，本文95%，支持RL在蒸馏中补足成功信号。

[发布说明](https://github.com/amazon-far/php_parkour/blob/main/README.md)、[训练指南](https://github.com/amazon-far/php_parkour/blob/main/wbt_training/README.md)和[仿真指南](https://github.com/amazon-far/php_parkour/blob/main/wbt_training/DEPLOY.md)提供motion-matching数据库、地形、生成脚本、固定Holosoma依赖、专家／学生训练和ONNX导出入口，以及MuJoCo启动脚本。训练需Linux／NVIDIA与IsaacSim；现成数据涵盖locomotion、低／高速step及低／高速climb-76共5类，亦可用公开数据库生成其他场景。官方student release仅发布depth_backbone.onnx＋student.onnx，覆盖示例能力；未发布原始.pt／teacher checkpoints。当前公开训练实现有自己的DAgger残差门控等配置，论文线性课程按原文另记。DEPLOY.md说明范围为sim2sim；未确认完整真机深度接入与G1部署配方。本次静态核验，未运行。对本项目可借鉴离线动作过渡数据生成与统一技能后端，语言任务规划和物体操作需另行接入。

**关键 Q&A（2026-11-05）**：Q：Policy多大？A：论文附录表VI给出学生MLP隐藏层(2048,1024,512,256,128)，深度编码器为3层CNN＋GAP、输出32维。核对官方student-assets-v1的ONNX：学生MLP输入140、输出29，共3,079,709参数；深度CNN为25,632参数，合计3,105,341参数。教师／critic隐藏层为(512,256,128)，部署使用学生与深度编码器。

Q（2026-11-10）：与 PhysHSI 有何异同？A：两者都用人体动作和在线仿真 RL 训练 G1 全身策略。PHP 将片段离线拼成带地形的逐帧参考，先训练特权跟踪专家，再用专家在学生采样状态上的动作标签和 PPO 共同训练统一深度策略。PhysHSI 按任务训练，示范用于 AMP 风格奖励和混合初态采样，策略依据物体状态与任务奖励完成交互。PHP 从单障碍训练迁移到多障碍串联，PhysHSI 验证同一交互在不同物体尺寸、位置和目标下的适应性。PHP 附录的 AMP 实验采用 MimicKit，因调参有限未列入正式对照。[PHP §III–IV](https://arxiv.org/html/2602.15827v2#S3)，[PHP 附录 C](https://arxiv.org/pdf/2602.15827v2#page=12)，[PhysHSI §III–V](https://arxiv.org/html/2510.11072v1#S3)。

## Psi-0（Ψ₀）— An Open Foundation Model Towards Universal Humanoid Loco-Manipulation

**推理数据流**

```text
头部相机图像 + 语言指令 → Qwen3-VL → 视觉语言特征
视觉语言特征 + 当前本体观测 → 动作专家 MM-DiT
  ↓
未来 H 步动作（每步 36 维）
  ├─ 28 维 → 双臂、双手关节目标
  └─  8 维 → AMO → 15 个腰腿关节目标
  ↓
G1 执行 → 采集新观测
```

**训练数据流**

| 阶段 | 数据怎么流 | 更新谁 |
|---|---|---|
| 预训练 | EgoDex 人体视频、Humanoid Everyday 机器人数据 → 双腕位姿和指尖位置 → FAST 动作 token → 预测下一步动作 token | Qwen3-VL |
| 后训练 | Humanoid Everyday 的关节动作 → 给动作加噪 → 用视觉语言特征和本体状态做条件，以 flow matching 学习生成动作块 | 动作专家从头训练，VLM 冻结 |
| 任务微调 | 每项任务 80 条 G1 遥操作轨迹 → 用示范中的全身动作块继续训练 | 主实验只微调动作专家，AMO 直接复用 |

**重点关注**：False

**开源情况**：[2026-11-10] sim复现、sim重新训练（SIMPLE任务微调）；直接部署（G1，AMO／SONIC对应版本）；[官方代码](https://github.com/physical-superintelligence-lab/Psi0)。

**基本信息**：阅读日期 2026-11-10；arXiv上传 2026-03-12（v1）；共同一作 Songlin Wei、Hongyi Jing、Boqian Li、Zhenyu Zhao；通讯 Yue Wang；单位 USC PSI Lab、NVIDIA、WorldEngine；中稿 [RSS 2026](https://roboticsproceedings.org/rss22/p021.html)；[项目主页](https://psi-lab.ai/Psi0/)，[arXiv](https://arxiv.org/abs/2603.12263)。

**核心问题**：如何用第一视角人类视频学习操作相关的视觉和动作表征，再用较少的真机示范学到 G1 可执行的灵巧移动操作。

**系统拆解**：头部图像与语言指令经 VLM 编码，动作专家结合当前本体状态生成 36 维动作序列。28 维直接指定双臂与双手关节，其余 8 维由 AMO 转成 15 个腰腿关节目标，合计 43 维关节目标。以下为论文 AMO 版本。[§III／图2–4](https://arxiv.org/html/2603.12263v1#S3)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| System-2 → Vision–Language Backbone | Qwen3-VL-2B-Instruct | 头部图像＋指令 → 视觉语言编码 → 隐特征供动作专家使用。 |
| System-1 → Action Expert | 约 500M 参数的 flow-based MM-DiT | VLM 隐特征＋本体状态 → 生成动作块 → 28 维手臂／手指关节目标＋8 维躯干、身高、速度与朝向命令。 |
| System-1 → Real-Time Action Chunking | 训练时动作前缀补全 | 保留已承诺执行的动作前缀 → 异步补全后续动作 → 连续执行。 |
| System-0 → Lower-Body Controller | AMO，预训练 RL 跟踪策略 | 8 维命令 → 15 维腰腿关节目标；与上肢目标一起执行。 |

**训练、复用与新增**：先训练视觉动作表征，再学机器人关节空间动作，最后按目标任务适配。论文 §III-B 描述可进一步端到端微调，主实验 §IV-A 明确只微调动作专家，公开 AMO 配方也冻结 VLM。[§III–IV、附录VI](https://arxiv.org/html/2603.12263v1#S3)，[AMO 微调脚本](https://github.com/physical-superintelligence-lab/Psi0/blob/main/scripts/train/psi0/finetune-real-amo-psi0.sh)

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| System-2 → Vision–Language Backbone | 从 Qwen3-VL-2B-Instruct 初始化，依次在约 829 小时 EgoDex 和约 31 小时 Humanoid Everyday 上训练。FAST tokenizer 用 50 万条 EgoDex 动作样本拟合。 | 将双腕位姿和指尖位置统一成 48 维动作，自回归预测单步动作 token。 |
| System-1 → Action Expert | 冻结 VLM，从头用 Humanoid Everyday 的手臂和手指关节动作训练 MM-DiT，缺失的下身维度填充。下游沿用该权重，每任务用 80 条 G1 遥操作轨迹微调动作专家。 | 用 flow matching 学连续动作块，真机示范补上对应任务的移动与操作协调。 |
| System-1 → Real-Time Action Chunking | 与动作专家同训，复用示范动作块。 | 随机保留干净前缀并从损失中排除，训练模型接续已经进入执行队列的动作。 |
| System-0 → Lower-Body Controller | 直接复用 AMO 策略，供真机遥操作采集与推理使用。 | 将上肢关节控制与腰腿跟踪接入同一动作接口。 |

真机主实验为 8 项分别微调的长程任务，每项 10 次。按表 III 的完整任务成功数汇总，Psi-0 为 57/80（71.25%），GR00T N1.6 为 23/80（28.75%）。全部子任务完成才计整项成功，评估者可在子任务失败后协助推进以统计后续子任务。附录多任务微调消融仅覆盖三项，并报告单项成绩下降。[§IV-B、附录IX/X](https://arxiv.org/html/2603.12263v1#S4)

官方提供预训练／后训练和任务微调代码、[权重](https://huggingface.co/USC-PSI-Lab/psi-model/tree/main/psi0)与[示范数据](https://huggingface.co/datasets/USC-PSI-Lab/psi-data/tree/main)。[SIMPLE 指南](https://github.com/physical-superintelligence-lab/SIMPLE/blob/main/docs/source/tutorials/wholebody_loco_manipulation.md)给出仿真闭环评测入口，Psi-0 仓库提供任务微调脚本。当前 README 部分旧脚本名已失效，仿真入口按任务看 [MP／解耦 WBC／SONIC 脚本](https://github.com/physical-superintelligence-lab/Psi0/tree/main/scripts/train/psi0)。后来发布的 [SONIC 真机版本](https://github.com/physical-superintelligence-lab/Psi0/blob/main/examples/psi0_for_sonic.md)配有单独权重与部署流程。本文结果对应 AMO 实验。本次静态核验。

**关键 Q&A**：Q：与 PhysHSI 的训练信号有什么区别？A：PhysHSI 的 actor 在仿真里采样，用 PPO 最大化任务与 AMP 风格回报。Psi-0 的 VLA 从离线动作标签学习，预训练预测动作 token，后训练及微调用示范动作块做 flow matching；RL 在复用的底层 AMO 控制器中。[PhysHSI §III](https://arxiv.org/html/2510.11072v1#S3)，[Psi-0 §III](https://arxiv.org/html/2603.12263v1#S3)

## ExoActor — Exocentric Video Generation as Generalizable Interactive Humanoid Control

[论文](https://arxiv.org/html/2604.27711) · [项目主页](https://baai-agents.github.io/ExoActor/)

**作者** Yanghao Zhou、Jingyu Ma（并列一作），通讯 Yu Bai。单位为北京智源人工智能研究院。**录用** 未查到公开录用记录。

**论文公开日期** 2026-04-30 · **代码与录用核查日期** 2026-11-16

**开源情况** 完整实现未公开。项目页代码入口返回 404，缺视频到动作的集成与真机运行入口。

**核心问题** 如何把第三人称场景图像和任务指令转成 G1 可执行的全身交互动作。

**推理数据流**

```text
第三人称初始图像 → 将画面中的机器人转换为人体图像
任务指令 → 分解成动作序列 → 与初始图像一起生成第三人称视频
生成视频 → GENMO 恢复全身运动，另估计手部动作
固定全身参考 + 本体观测 → SONIC → G1 跟踪，手指由独立控制接口执行
```

**训练数据流**

```text
复用预训练图像编辑、视频生成、动作估计模型和 SONIC
新任务输入 → 生成视频并恢复动作参考 → 无逐任务训练
```

## GenHOI — Contact-Aware Humanoid-Object Interaction by Imitating Generated Videos without Task-Specific Training

[论文](https://arxiv.org/html/2606.12995v2) · [项目主页](https://genhoi-humanoid.github.io/) · [代码](https://anonymous.4open.science/r/Gen-HOI-CA67)

**作者** 毕志海、Qiang Zhang，通讯马骏。单位为香港科技大学（广州）、中国科学技术大学人工通用智能研究院、香港大学、新加坡国立大学。**录用** 未查到公开录用记录。

**论文公开日期** 2026-06-11 · **代码与录用核查日期** 2026-11-16

**开源情况** sim复现（示例视频、接触估计、轨迹优化、MuJoCo 与 SONIC 入口）。真机侧公开了里程计接入与轨迹跟踪脚本，完整物体定位与部署流程未核实。无逐任务训练。

**核心问题** 如何把生成视频中的手物接触转成符合实测物体位置的全身轨迹，让 G1 能抓箱、搬椅和抬桌。

**推理数据流**

```text
机载相机测物体位姿 + LiDAR 定位 → 在 MuJoCo 重建场景并渲染首帧
首帧 + 指令 → 生成视频 → 提取全身动作及手物接触约束
动作 + 接触约束 + 实测物体位姿 → 优化 G1 参考轨迹
轨迹 + 实时里程计 → 基于 SONIC 的控制器 → 下身跟踪根部路径，上身跟踪关节参考
```

**训练数据流**

```text
复用预训练视觉模型、动作估计模型和 SONIC
单条生成视频 → 按当前物体位姿优化轨迹 → 无逐任务训练
```

## GR00T 2 — Scaling Robot Foundation Models for Generalization, Dexterity, and Adaptation

[论文](https://research.nvidia.com/labs/gear/gr00t-2/assets/gr00t2_whitepaper.pdf) · [项目主页](https://research.nvidia.com/labs/gear/gr00t-2/)

**作者** NVIDIA（技术报告集体署名）。**录用** 在 CoRL 2026 期间发布，未查到正式录用记录。

**论文公开日期** 2026-11-09（官方发布公告） · **代码与录用核查日期** 2026-11-16

**开源情况** GR00T 2 Turbo 的代码和权重尚未公开，项目页预告代码于 12 月开放。底层 SONIC 已公开。

**核心问题** 如何让机载视觉模型根据任务生成全身动作，由 SONIC 执行走动、抓取和投放等连续操作。

**推理数据流**

```text
头部及双腕 RGB + 本体观测 + 指令 → GR00T 2 Turbo（约 1 Hz）
→ 未来 1.6 秒的 SONIC 运动 token 和夹爪宽度命令
运动 token + 当前本体观测 → SONIC（50 Hz）→ 关节位置目标 → G1 PD 执行
夹爪宽度命令 → 夹爪控制
```

**训练数据流**

| 阶段 | 数据怎么流 | 更新谁 |
|---|---|---|
| 预训练 | 人体第一视角视频、手部轨迹与机器人示范 → 先预测未来视频，再用 flow matching 联合学习视频与动作 | 基于 Cosmos-Predict2.5 初始化的模型 |
| G1 移动操作 | 真机遥操作直接记录 SONIC token 和夹爪宽度，部分动作通过相同 SONIC 在仿真回放并扩增场景 → 用实机和合成示范训练 | GR00T 2 Turbo，SONIC 复用 |

## Workhorse — Learning Robust Whole-Body Humanoid Loco-Manipulation from Human Data

[论文](https://arxiv.org/html/2610.09117v2) · [项目主页](https://hybridrobotics.github.io/workhorse/)

**作者** Songbo Hu、廖洽源（并列一作）。单位为加州大学伯克利分校。**录用** 未查到公开录用记录。

**论文公开日期** 2026-10-06 · **代码与录用核查日期** 2026-11-16

**开源情况** 完全没开源（项目页注明代码尚未发布）。

**核心问题** 如何用穿戴式人体示范训练 G1，在搬箱过程中根据视觉变化持续调整全身动作。

**推理数据流**

```text
机载 RGB + 躯干、双腕、双足的位姿历史 → 视觉规划器（5 Hz）
→ 五个身体部位的未来位姿 + 本体观测 → 全身跟踪器（50 Hz）
→ 关节位置目标 → G1 执行 → 新观测用于重规划
```

**训练数据流**

| 阶段 | 数据怎么流 | 更新谁 |
|---|---|---|
| 视觉规划 | 人体胸前 RGB + 五个 Vive 位姿 → 图像和历史扰动增强 → 用 flow matching 学习未来五部位轨迹 | 视觉规划器 |
| 全身跟踪 | 人体位姿提供跟踪目标，mjlab 转移拟合动力学模型 → 在想象轨迹中优化跟踪回报 | 动力学模型与跟踪器。跟踪设计沿用 BeyondMimic，训练方法沿用 SuperMimic，按机器人和任务训练 |

## HALO — Perceptive Humanoid Motion from Simulation

[论文](https://arxiv.org/html/2611.06908v1) · [项目主页](https://apple-aiml-research.github.io/ml-halo/) · [代码](https://github.com/apple-aiml-research/ml-halo)

**作者** Siyuan Hu、Abulikemu Abuduweili。单位为 Apple。**录用** 未查到公开录用记录。

**论文公开日期** 2026-11-04 · **代码与录用核查日期** 2026-11-16

**开源情况** 公开数据生成、训练和 G1 部署代码。可按公开流程生成数据并重新训练（文档示例为坐沙发）。闭环仿真推理和直接真机运行均需自训权重，仓库未附训练集和权重。

**核心问题** 如何用仿真生成视觉和动作训练数据，让 G1 在陌生室内环境根据机载视觉完成推箱、坐椅等全身任务。

**推理数据流**

```text
语言请求 + 第一视角 RGB → VLM 生成子任务
子任务 + 最新 RGB → 动作生成器（约 3 Hz）→ 未来 5 秒的全身关节和根部轨迹
全身轨迹 + 机载深度 + 本体观测 → 运动跟踪器（50 Hz）
→ 关节位置目标 → G1 PD 执行
```

**训练数据流**

| 阶段 | 数据怎么流 | 更新谁 |
|---|---|---|
| 动作生成 | Infinigen 生成室内场景，Kimodo 生成约束动作 → 仿真渲染 RGB 与配对轨迹 → flow matching | 动作生成器 |
| 运动跟踪 | 合成轨迹与重定向 LAFAN1 → PPO 训练特权教师 → 行为克隆与 DAgger 蒸馏出深度学生 → 多 Critic PPO 微调 | 各任务教师与统一学生 |

## Kimodo — Scaling Controllable Human Motion Generation

[论文](https://arxiv.org/abs/2603.15546) · [项目主页](https://research.nvidia.com/labs/sil/projects/kimodo/) · [代码](https://github.com/nv-tlabs/kimodo)

**作者** Davis Rempe、Mathis Petrovich（共同一作）；论文未单列通讯作者。单位为 NVIDIA。**录用** 技术报告，未查到公开会议／期刊录用记录。

**论文公开日期** 2026-03-16 · **代码与录用核查日期** 2026-11-22

**开源情况** sim复现（人体／G1 动作生成，代码与权重）；缺少材料（完整原始训练流程、Rigplay 商业动捕数据）。

**核心问题** 如何通过文字和稀疏几何约束，生成多样、连续且精确满足目标的三维全身动作，作为动画或机器人参考轨迹。

**推理数据流**

```text
文字 → LLM2Vec／Llama-3-8B → 文字特征
文字特征 + 时长 + 可选约束（关键姿态、关节目标、路径点）+ 噪声动作
→ 两阶段 Transformer 扩散去噪（根部运动 → 身体运动）
→ 根部平移与骨架关节旋转序列（30 fps）→ G1 格式转换 → 机器人参考轨迹
```

**训练数据流**

| 阶段 | 数据怎么流 | 更新谁 |
|---|---|---|
| 文字动作预训练 | 带文字描述的 Rigplay 光学动捕 → 运动表示与加噪 → 预测干净动作 | 动作去噪网络 |
| 约束训练 | 动作中随机抽取姿态、末端和路径约束 → 带文字与约束去噪 | 同一动作去噪网络；G1 版本使用重定向到 G1 的数据 |

**在 HALO 中** 使用现成 G1 权重，在空场景中按路径与关键姿态生成“走近 → 转身 → 坐下”，多候选筛选后保存轨迹，供 tracker 训练或后续配场景渲染。

**本地资源** G1 动作模型约 282M 参数，另需 8B 文字编码器；官方常规生成约需 17 GB 显存，将文字编码移到 CPU 可降至 3 GB 以下。HALO 的批量生成已在 H100 上测试，200 条坐下轨迹含筛选约 2.5～3.5 小时。按 Linux x86_64、无缓存、复用 HALO 安装脚本估算：软件包约 5～10 GB，模型约 17.5 GB，首次下载合计约 23～30 GB；环境与权重建议预留 40～60 GB 磁盘，生成数据另计。估算依据为官方依赖清单与 HALO 的 `setup_kimodo.sh`，未实际下载安装测量。

## Infinigen Indoors — Photorealistic Indoor Scenes using Procedural Generation

[论文](https://arxiv.org/abs/2406.11824) · [项目主页](https://infinigen.org/) · [代码](https://github.com/princeton-vl/infinigen)

**作者** Alexander Raistrick、Lingjie Mei（与第三作者 Karhan Kayan 共同一作）；论文未单列通讯作者。单位为 Princeton University 计算机系。**录用** CVPR 2024。

**论文公开日期** 2024-06-17 · **代码与录用核查日期** 2026-11-22

**开源情况** sim复现（室内物体与场景生成、导出）；程序化生成工具，无需训练权重。

**核心问题** 如何自动生成几何、材质和布局均可变化，且满足空间与语义约束的真实感三维室内场景，用于视觉与具身训练。

**生成数据流**

```text
随机种子 + 房间参数 + 布局约束
→ 程序化生成建筑、家具、日用品及材质
→ 约束求解器安排房间与物体布局 → Blender 三维场景
→ 渲染图像与真值标注，或导出资产用于仿真
```

**训练数据流** 无神经网络训练；按程序规则与约束求解生成数据。下游视觉或机器人模型使用生成结果训练。

**在 HALO 中** 房间与 Kimodo 轨迹独立生成；HALO 烘焙房间光照并分析空闲空间，再将目标椅子与整段轨迹共同平移、旋转到房间中，回放并渲染第一视角 RGB。

**本地资源** 无大模型权重，主要依赖 Blender、`bpy` 与几何计算库。房间生成与 Cycles CUDA 烘焙可在 H100 上运行；HALO 配置单分片约 60 GB 系统内存，未给最低显存，每个房间含生成和烘焙约 30～60 分钟。按 HALO 同时安装独立 Blender 和 `bpy` 的方案，首次下载估算约 3～8 GB，环境建议预留 10～20 GB 磁盘；场景输出另计。估算依据为 HALO 的 `setup_scene_gen.sh`，未实际下载安装测量。后续 Isaac Sim RTX 图像渲染是单独阶段，按仓库要求使用 RTX GPU。
