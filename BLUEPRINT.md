# BLUEPRINT — 人形机器人研究蓝图

## 任务范围与技术路线

【2026-11-16】**带感知的人形移动操作（Perceptive Humanoid Loco-Manipulation）**是本项目的任务定义。机器人根据在线环境观测协调移动与物体操作，以完成物体层面的目标，例如走到桌边抓取、搬运并放置物体。环境观测可以是相机图像，也可以是机载感知模块估计的物体状态。**仿真训练与真机迁移（sim-to-real）**描述训练和部署路线，四篇工作可放在这条路线下比较。

| 工作 | 论文的任务表述 | 训练与部署 |
|---|---|---|
| VIRAL | **Autonomous humanoid loco-manipulation**。用机载感知协调移动和操作，验证走动、放置、抓取和转身。 | PPO 训练特权教师，DAgger 与行为克隆训练 RGB 学生，复用 HOMIE 执行。直接对应本项目的任务定义。[§1–2](https://arxiv.org/html/2511.15200v2) |
| PhysHSI | **Humanoid-scene interaction（HSI）**。任务包括搬箱、坐下、躺下和起身。 | 按任务用 PPO 与 AMP 训练策略，真机由 LiDAR 里程计和 AprilTag 提供物体位姿。其中搬箱属于带感知的移动操作，坐卧和起身归入场景交互。[§III–IV](https://arxiv.org/html/2510.11072v1#S3) |
| PHP | **Perceptive humanoid parkour**。根据地形选择跨步、攀爬、撑越等动作，目标是越障通行。 | PPO 训练参考动作跟踪专家，再用 DAgger 与 PPO 训练统一深度学生。归入带感知的移动控制，可对照视觉蒸馏和技能切换。[§III–IV](https://arxiv.org/html/2602.15827v2#S3) |
| SuperMimic | **Humanoid whole-body motion tracking**。根据参考动作实现平地全身跟踪。 | 通过可微动力学模型的想象轨迹更新策略，PPO 用作对照。归入全身执行层，可作为移动操作系统的底层控制候选。[§3、§6](https://arxiv.org/pdf/2610.28480#page=4) |

### 动作生成与全身跟踪

【2026-11-16】高层根据环境观测和任务目标生成全身运动参考，低层跟踪器结合本体观测执行。高层可以一次生成再执行，也可以用新观测滚动重规划。复用预训练跟踪器可省去为新任务单独做仿真强化学习，跟踪器的预训练、场景渲染或训练数据增强仍可用到仿真。[SONIC](https://arxiv.org/abs/2511.07820)

- **ExoActor / GenHOI** 从生成视频提取运动，交给 SONIC 或基于 SONIC 的控制器。ExoActor 用外部视角生成固定参考。GenHOI 用机载传感器估计物体位姿，在仿真中渲染首帧并按手物接触优化参考，真机验证抓箱、搬椅和抬桌，无需逐任务训练。[ExoActor](https://arxiv.org/html/2604.27711#S2.SS4) · [GenHOI](https://arxiv.org/html/2606.12995#S3)
- **GR00T 2 Turbo + SONIC** 用机载 RGB、本体观测和指令，以约 1 Hz 生成 SONIC 运动 token，SONIC 以 50 Hz 输出关节目标。G1 真机完成走动、抓取和投放，训练数据使用仿真扩增。[GR00T 2 §3.5](https://research.nvidia.com/labs/gear/gr00t-2/assets/gr00t2_whitepaper.pdf#page=13)
- **Workhorse / HALO** 从机载视觉滚动生成轨迹。Workhorse 用 flow matching 预测躯干、双腕和双足的轨迹，跟踪器基于 BeyondMimic 设计，按 SuperMimic 的方法单独训练。HALO 生成全身运动轨迹，跟踪器同时读深度和本体观测，训练数据与跟踪器训练均用到仿真。[Workhorse](https://arxiv.org/html/2610.09117#S4) · [HALO](https://arxiv.org/html/2611.06908#S3)
