查了几家公司的官方网页，Figure 和 1X 明确公开了用仿真 RL 训练全身控制器的方案。小鹏也确认 IRON 的步态训练用了仿真和 RL，只是公开到的细节更少。波士顿动力则在不同任务上分别展示过 MPC 和 RL。
公司 / 机器人	公开资料里，谁负责全身协调和平衡
Figure · Helix 02	S1 根据感知以 200 Hz 输出全身关节目标，S0 以 1 kHz 跟踪这些目标，处理平衡和接触。S0 用重定向后的人体动作做参考，在仿真里用 RL 训练。这是把「生成关节目标」和「稳定执行」分开的明确例子。[ Figure 官方技术博客](https://www.figure.ai/news/helix-02)
小鹏 · IRON	官方披露的步态训练流程是 人体步态采集 → 动作重定向 → 仿真强化学习 → 真机测试。产品页把 VLA 称为负责全身控制的「小脑」，但我查到的资料没有讲清它与步态控制器之间传什么指令，也没有披露是否用 PPO。[ 小鹏步态技术文章 · IRON 产品页](https://www.xpeng.com/au/insight/xpeng_iron_humanoid_robot)
https://www.xpeng.com/technology/ai_robot_iron
1X · NEO / Redwood	监督学习的运动生成器把摇杆方向等粗指令变成身体姿态轨迹，仿真 RL 训练的跟踪器根据参考轨迹和实时状态控制身体、维持平衡。官方博客还解释了为什么动捕回放和可控行走需要不同的接口。 [1X《Redwood AI: Mobility》](https://www.1x.tech/discover/redwood-mobility)
波士顿动力 · Atlas	早期 Atlas 用模型预测控制（MPC），根据实测状态和动力学模型在线计算动作，协调落脚、负载和平衡。与丰田研究院合作的示范里，上层模型给出躯干、双手和双脚目标，沿用基于 MPC 的执行接口。后续 Atlas 搬冰箱的技能则用了仿真 RL。 Atlas 的 MPC · Atlas × TRI · RL 搬冰箱
https://bostondynamics.com/blog/picking-up-momentum/
https://bostondynamics.com/blog/large-behavior-models-atlas-find-new-footing/
https://bostondynamics.com/blog/training-a-humanoid-robot-for-hard-work/
宇树 · G1	宇树公开了 Isaac Lab 强化学习 → MuJoCo 验证 → G1 真机部署的开发流程，使用 g1_ctrl 运行训练好的策略。这是可查代码的官方方案。它的 UnifoLM-WLA 基座模型也公开了包含腿部关节的动作格式，但我没有核实到该模型真机部署时，腿部输出是否直接送到电机 PD。 宇树 RL 官方仓库 · UnifoLM 动作接口
https://github.com/unitreerobotics/unitree_rl_lab#sim2real
https://github.com/unitreerobotics/unifolm-wla/blob/main/docs/robot_action_state_processing_en.md

最贴合你前面理解的是 1X 那篇。 它把「监督学习生成运动参考，RL 跟踪器负责平衡」完整写出来了。对应的工业对照是 Atlas + TRI，同样有学习出来的上层模型，底层平衡由 MPC 负责。