# 参考运动作为先验的人形 RL（HIL × Multi-Task Reference RL）

**读于**: 2026-06-12
**Tags**: #paper #RL #imitation-learning #humanoid-control #adversarial-imitation #related
**原件**: [[HIL Hybrid Imitation Learning for Dynamic Athletic Control.pdf]] · [[Generalizing from References - Multi-Task Reference and Goal-Driven RL.pdf]]

> [!info] 这是一篇**合并笔记**——同作者（Jiashun Wang）的两篇姊妹篇
> 两篇是同一思想的 **sim 版 / real 版**，90% 方法哲学重合，价值在**差异对照**（§4）。分开写会重复，故合并。
> - **HIL**：Hybrid Imitation Learning for Dynamic Athletic Control（ACM TOG 2026，仿真角色动画，NVIDIA/CMU/SFU）
> - **Multi-Task Ref RL**：Generalizing from References using a Multi-Task Reference and Goal-Driven RL Framework（[arXiv 2602.20375](https://arxiv.org/abs/2602.20375)，真机 Unitree G1，RAI Institute + CMU）

> [!warning] 命名陷阱
> 这篇的 **HIL = Hybrid Imitation Learning**，跟 [[强化学习后训练 主题地图]] §3.5 里的 **HIL = Human-In-the-Loop**（HIL-SERL）**毫无关系**，别混。

---

## 1. 一句话总结

> [!note] 待填（你自己写）
> 提示词：「参考运动当**软塑形先验**而非硬跟踪约束」「多任务 RL 共享 goal-conditioned 观测」「推理时丢掉参考」「sim 用对抗判别器 / real 改用物理课程」
> 两篇先各写一句，再写一句共同点。

- **HIL**：_待填_
- **Multi-Task Ref RL**：_待填_
- **共同**：_待填_

---

## 2. 它们要解决什么问题

- **2.1 问题本身**：有了人类参考运动（动捕 / 网络视频），想让物理仿真角色 / 真机人形学到**自然、协调**的全身运动技能（parkour、跳、爬），同时还能**泛化到新目标 / 新障碍布局**。
- **2.2 之前为啥没解决（一对核心张力）**：
  - **硬跟踪流派**（DeepMimic / GMT / ZEST）：严格逐帧跟踪参考 → 动作自然，但**脆**，出了示范分布就垮，无法对新目标 steer。
  - **纯任务驱动 RL**：只给任务奖励 → 能泛化，但动作**退化、难看**（趴地爬、抽搐、利用仿真器 bug）。
  - **对抗模仿（AIL/AMP）**：分布匹配比逐帧跟踪灵活，但在**动态 / 接触丰富**场景里判别器目标难稳、易 mode collapse；真机上尤其难调。
  - **蒸馏流派**（MaskedMimic）：teacher/student 都只在 motion-tracking 设定下训 → 仍被参考分布锁死，遇到新交互就垮。
- **共同答案**：**把参考运动当"行为塑形先验（behavioral shaping prior）"，而不是部署时的硬约束**——见 [[参考运动作为塑形先验]]。

---

## 3. 共同方法骨架（两篇都这么干）

1. **多任务 RL**：单一 goal-conditioned 策略，**同时**训两个任务。
   - 任务 A = **模仿**：参考运动提供 dense 监督，塑形出自然技能。
   - 任务 B = **泛化 / 适应**：目标独立于参考采样，逼策略把技能迁移到新条件。
2. **共享 goal-conditioned 观测空间**：两个任务吃**同一套输入**（角色状态 + 目标条件），策略隐式沿参考推断进度，**不用显式 phase 变量 / future pose**。这是让两模式能放进一个策略的关键设计。
3. **推理时完全不需要参考运动输入**（no reference / phase at inference）——参考只在**训练时**定义奖励和目标条件。
4. **非对称 critic + 任务指示位 `k_t`**：critic 吃特权信息（仿真里可得、真机不可得）+ 一个二值 `k_t` 指明当前样本来自哪个任务——因为两任务奖励结构不同，不区分会让 value 估计崩（HIL 实测去掉 `k_t` → critic loss ×5）。
5. **PPO** 训练；学到的技能可**组合成长程序列**（连续过多个障碍 / parkour）。

---

## 4. ⭐ 关键差异对照表（本笔记的骨架）

> 两篇的「任务 A 模仿」几乎一样；**差异全在「任务 B」怎么设计、以及怎么从 sim 走向 real**。

| 维度 | **HIL**（TOG，仿真） | **Multi-Task Ref RL**（arXiv，真机） |
| :--- | :--- | :--- |
| **载体** | 仿真 SMPL 角色（Isaac Gym） | **真机 Unitree G1**（29 DoF, 1.2m, 35kg；Isaac Lab + 真机） |
| **任务 B 是什么** | **对抗模仿**（AMP 判别器，scene-conditioned） | **纯 goal-driven 泛化**（随机目标，仅任务奖励，无判别器） |
| **靠什么注入自然性** | 判别器 `D` 给 style reward（分布匹配） | 模仿任务的 dense tracking reward（当正则信号贯穿训练） |
| **有无判别器** | **有**（且把场景条件喂进 `D`） | **无**——作者明确弃用，称对抗目标「真机难稳、难调」 |
| **关键稳定技巧** | **PSI**（扰动状态初始化）缓解 mode collapse、促进技能转换 | **assistive-wrench 课程**（单标量 λ 自适应：早期给虚拟助力，逐步撤） |
| **动作参数化** | PD 目标（直接输出） | **residual** `q_cmd = q̄ + Σa_t`，**故意不**用参考关节角做 feedforward（保留偏离能力） |
| **策略网络** | Transformer + PointNet（吃场景点云） | 3 层 MLP → Gaussian |
| **观测里的 goal** | 场景点云 `c_t` + 目标位置 `l_t` | 2D 目标 root 位置 `(x, y)` |
| **任务集** | parkour（19 clips / 15 skills）+ heading 朝向 + 坐椅 | walk-climb / walk-jump / climb-down |
| **数据来源** | YouTube 视频（TRAM 估姿 + 朝向修正 + 交互式场景标注 + 物理 tracker 精修） | 单条参考运动 / behavior（walk-climb/jump 加镜像增广） |

> [!tip] 最有信息量的一行
> **「有无判别器」**：arXiv 那篇是同作者把 HIL 搬上真机时**主动做的减法**——sim 里靠对抗判别器注入自然性，real 里嫌它难调，退回"模仿任务的 tracking reward 当软正则 + 物理课程扶着"。**"sim→real 时砍掉对抗目标" 本身就是一条工程经验。** 详见 [[参考运动作为塑形先验]] 的「对立思路」。

---

## 5. 各自方法细节

### 5.1 HIL —— 两模式怎么共存

- **统一观测破局点**：motion-tracking 模式天然要 phase/future-pose 当输入，对抗模式没参考给不了 → 改用 **scene-conditioned goal 观测**（PointNet 编码场景点云 `c_t` + 目标 `l_t`），两模式同一套输入，策略隐式推断进度。
- **判别器吃场景**：`D(s_{t-n:t}, c_{t-n:t})` —— 不只判"动作自不自然"，还判"这动作在这场景下**合不合适**"。二值分类 loss + gradient penalty（AMP 式）。消融（w/o scene info）证明这条关键。
- **PSI（扰动状态初始化）**被论证为缓解 mode collapse 的桥：技能转换难学时 RL 偷懒只用几个万能技能；PSI 让转换更易学 → 逼出更丰富技能覆盖（Fig 6 skill coverage 直方图）。
- 奖励：`r_t = w^task · r^task + w^style · r^style`（tracking 模式也加 style 项）。
- 训练课程：先 **4B** 样本纯 motion-tracking 打底，再 **2B** 样本两模式并行（半 tracking / 半对抗）。4×V100，120Hz 仿真 / 30Hz 策略，PPO + GAE。

### 5.2 Multi-Task Ref RL —— MDP 设计与课程

- **两任务只在 goal / reward / 初始化上不同**，共享观测 / 动作空间：
  - 模仿任务：goal 取自参考；初始化**贴近参考**（小扰动锚定）→ 专注学高质量技能。
  - 泛化任务：goal **随机采**；初始化从**宽分布**采（标准 RL）→ 逼适应新状态新目标。
- **residual action**（`q_cmd = q̄ + Σa_t`）：故意不把参考关节角当 feedforward —— 这样任务驱动时策略才**有自由偏离参考**去改动作。
- 奖励：模仿 `r = r_track + r_reg + r_surv`；泛化 `r = r_goal + r_reg + r_surv`（稀疏目标奖励 `r_goal = -c_p‖e^xy‖² - c_o‖θ‖² + r_reach`）。
- **assistive-wrench 课程**：单标量 `λ∈[0,1]` 在线自适应（agent 避免早停 + 大致跟住参考时 λ↑）。λ 同时控制三件事：
  1. 施于 base 的**虚拟助力 wrench** 大小：`w_e = β(λ)[F_b; M_b]`，`β(λ)=(1-λ)β_max`（λ≈0 强助力 → λ→1 撤掉）。虚拟 wrench = base 位姿 tracking 的 PD 项 + 补偿名义躯干动力学的 feedforward（eq 5a/5b）。
  2. **模仿 vs 泛化任务的采样比例**：`p_imi(λ)=(1-λ)p_0 + λ p_target`，`p_0 > p_target`（从纯模仿过渡到泛化为主）。
  3. **初始状态 / 目标分布宽度**（随 λ 展开）。
- 训练：Isaac Lab，PPO，非对称 actor-critic。Actor = 3 层 MLP → Gaussian（σ 固定）。Critic = 3 层 MLP + `k_t` + 特权信息（full root state / contact forces / assistive wrench）。域随机化（摩擦 / link 质量 / 推力 / 观测噪声）做真机迁移。

---

## 6. 关键结果 / 消融

### 6.1 HIL（parkour，噪声未见场景，Table 1）

| 方法 | Skill Acc ↑ | Track Err ↓ | Task Completion ↑ |
| :--- | :--- | :--- | :--- |
| Task Reward | 0.00 | 1.82 | 0.81 |
| AMP | 0.06 | 1.49 | 0.11 |
| ASE | 0.03 | 1.63 | 0.00 |
| MaskedMimic | 0.50 | 0.41 | 0.00 |
| Task Reward w/ ws | 0.15 | 0.54 | **0.86** |
| AMP w/ ws | 0.54 | 0.37 | 0.85 |
| **HIL（Ours）** | **0.66** | **0.31** | 0.74 |

> 读法：纯 Task Reward 完成率最高（0.86）但 skill acc=0（动作退化、绕障爬过）；AMP 只有 0.11 完成率 + mode collapse（反复用同一个 vault 动作）；**HIL 在自然度（skill acc / track err 最佳）和完成率间取得最好平衡**。

**消融（Table 2）**：w/o D → 0.53/0.36/0.62；w/o PSI → 0.5/0.37/0.52；w/o scene info in D → 0.38/0.39/**0.75**；w/o `k_t`（critic 任务指示）→ 0.52/0.40/0.73（且 critic loss ×5）。

**鲁棒性**：σ=0.05 时 >70% 完成；σ=0.1 时 >50%。训练只用 5 个障碍，σ=0.03 下 20 个障碍序列仍 40% 完成。

**heading 任务（Table 3，direction / facing / return）**：HIL 0.94 / **0.97** / 227；AMP **0.95** / 0.94 / **266**；ASE 0.54/0.78/147；MaskedMimic 0.79/0.72/17。（HIL 自然度 + 任务两头都好，AMP 回报高但行为窄。）

### 6.2 Multi-Task Ref RL（真机）

- **真机 Unitree G1** 完成 walk-climb / walk-jump / climb-down（box edge 2.3m）。
- Fig 2：success rate vs 初始 x-y / yaw / box height 的鲁棒性曲线（标称值附近高成功率，偏离渐降）。
- 展示**长程 parkour 组合**：把学到的单技能串成连续序列，无需精调初始条件 / task-specific reset。
- 对照三个问题：vs tabula-rasa RL（任务成功 / 鲁棒 / 动作质量）、技能能否模块化组合、哪些组件关键（消融）。

---

## 7. 跟我有啥关系

**我的方向**：[[RL 与模型后训练]]（子方向：[[灵巧手 World Model RL]]）

> [!warning] 诚实定位
> 这两篇是**人形全身运动控制 / 角色动画**，策略是 PPO + Gaussian（MLP/Transformer），**不涉及** diffusion policy / action chunking / world model——不是我核心的 VLA/manipulation 线。但**「参考/示范怎么用」这对张力**直接命中我 [[强化学习后训练 主题地图]] §1「怎么不破坏预训练知识」。当**概念参照**用，不当竞品。

### 7.1 创新点 × 我的反应（对照表）

> 每行选一个标签：💡 启发 / 🔧 直接借鉴 / ⚠️ 需改造 / ❌ 不适用。第三列写**具体怎么用 / 为啥不用**——这是核心。

| 创新点 | 启发 / 借鉴 / 改造 / 不适用 | 怎么用 / 为啥不用（具体说） |
| :--- | :--- | :--- |
| 参考运动当**软塑形先验**而非硬跟踪约束（多任务联合） | _待填_ | _待填_ |
| 共享 goal-conditioned 观测让"模仿 + 任务"两模式共存（无 phase 变量） | _待填_ | _待填_ |
| HIL：判别器吃**场景条件**（评动作 × 场景适配性） | _待填_ | _待填_ |
| HIL：PSI 缓解 mode collapse / 促技能转换 | _待填_ | _待填_ |
| arXiv：sim→real **弃用对抗目标**，改用 assistive-wrench 物理课程 | _待填_ | _待填_ |
| arXiv：residual action 故意**不喂参考 feedforward** 以保留偏离能力 | _待填_ | _待填_ |
| 非对称 critic + 任务指示位 `k_t` 区分两种奖励结构 | _待填_ | _待填_ |
| 学到的技能可**组合成长程序列**（skill composition） | _待填_ | _待填_ |

### 7.2 其他启发（表格装不下的自由想法）

-

### 7.3 我可能要做的下一步

- [ ]

---

## 8. 待理解 / 存疑

- [ ] assistive-wrench 课程里 λ 自适应的具体触发阈值（"避免早停 + 大致跟住参考"如何量化）？
- [ ] HIL 判别器把 scene point cloud 和 state transition `s_{t-n:t}` 怎么拼接喂入？维度怎么对齐？
- [ ] **关键问号（跟我方向）**：这套"参考即软先验 + 多任务共享观测"范式，能否迁移到 VLA/manipulation？那边动作是 **action chunk + 高维视觉条件**，不是低维 proprioception——"共享 goal-conditioned 观测"这个破局点还成立吗？这正好对应 [[参考运动作为塑形先验]] 的开放问题。

---

## 9. 相关工作 / 谱系

- **硬跟踪流派**：DeepMimic · GMT · ZEST（sim2real motion tracking）—— 参考当硬约束的代表
- **对抗流派**：AMP · ASE —— 分布匹配，HIL 的任务 B 基底
- **蒸馏流派**：MaskedMimic（CVAE distillation）—— 仍锁在 tracking 设定
- **关联本 vault**：
  - [[RISE  Self-Improving Robot Policy with Compositional World Model|RISE]] —— "示范当先验"的**另一种实现**（advantage-conditioned，把改进当条件生成），跟这两篇是同一上位概念 [[参考运动作为塑形先验]] 的不同分支
  - [[Advantage-conditioned 微调]] · [[DSRL]] —— VLA 后训练侧的对照
- **概念笔记**：[[参考运动作为塑形先验]]（本批新抽）
