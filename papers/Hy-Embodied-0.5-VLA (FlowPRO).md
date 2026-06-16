# Hy-Embodied-0.5-VLA: From VLA Models to a Real-World Robot Learning Stack

**读于**: 2026-06-16
**Tags**: #paper #VLA #RL #post-training #flow-matching #manipulation #core
**arXiv**: [2606.14409](https://arxiv.org/abs/2606.14409)（Tencent Robotics X × Hy Team）
**原始分析**: `~/claude-papers/papers/hy-embodied-0.5-vla/`（含 flowpro-deepdive.md / index.html）｜导读关卡 `paper-world/levels/hy-embodied-0.5-vla.html`

---

## 1. 一句话总结

> [!note] 初稿，待程程用自己的话复核/重写
> HyVLA-0.5 是一条「具身 VLM + flow 动作头 + reward-free 偏好 RL + 跨本体部署」的真机学习栈。它要解决的是「把 VLA 策略可靠部署到真机、够到最后一毫米精度」——纯模仿够不到、现有 RL 依赖脆弱 reward/value 网络。核心方法贡献是 **FlowPRO**：用 flow-matching 回归损失当**隐式 reward**，做 per-state 的偏好对比优化（RPRO loss），靠**近端正则**防 DPO reward hacking、靠**对比梯度抵消**安全混 SFT，全程**不训任何 reward/value/critic**。真机四任务 3 轮后 SR 升到 94–99%，超过 DAgger 与 π0.6\*。

---

## 2. 它要解决什么问题

- **2.1 问题本身**：把一个 VLA 策略**可靠部署到真实机器人**，包括够到接触丰富任务「最后一毫米」的灵巧度，并能跨不同本体。
- **2.2 之前为啥没解决**（三家真机后训练各有死穴，§7）：
  - **SFT / DAgger**：只用正样本（专家纠正），**浪费失败轨迹**的信号。
  - **reward/value RL**（HIL-SERL、π0.6\*）：要训可靠 reward/value/advantage，接触丰富任务**稠密 reward 极难设计**。
  - **preference RL**（DPO、Flow-DPO、GRAPE）：免 critic，但**继承 plain-DPO 的 reward hacking**，per-state 信号被稀释。

---

## 3. 方法核心

backbone = [[hy-embodied-0.5（base 报告）|Hy-Embodied-0.5]] 4B MoT 具身 VLM，上接双塔 flow-matching 动作专家。三块设计：[[偏好式 Flow 策略优化（Flow-DPO·RPRO）]]（RL 后训练）、flow-matching 动作头、delta-chunk 表示。

### 3.1 组件概览
1. **动作专家**：双塔（理解塔 VLM + 生成塔 action expert），shared joint-attention 交互；conditional flow matching 生成连续动作 chunk。
2. **delta-chunk 相对 EEF 表示**：每臂 $R^{10}$ = 3D 平移 + 6D 连续旋转 + 1D 夹爪；动作是相对当前 EEF 位姿的**增量**，解耦本体运动学 → 跨本体迁移。
3. **紧凑记忆编码器**：每 4 层插时序 attention 聚合 K=6 帧历史，**零额外参数**。
4. **FlowPRO**：见 §4。

### 3.2 SFT 两轨道 vs FlowPRO RL 维度对照表

> 同一套维度填「监督微调」和「偏好式 RL」两种后训练，看清信号来源差异。

| 维度 | SFT（Track-A/B） | FlowPRO RL（§4） |
| :--- | :--- | :--- |
| 信号来源 | 演示（正样本） | 成对**成功/失败**（干预-回滚） |
| 要 reward 模型吗 | 不要 | **不要**（flow-matching loss 当隐式 reward） |
| 训练目标 | flow-matching 回归 $\mathcal{L}_{fm}$ | RPRO loss（对比 + 近端正则 + SFT 项） |
| 用失败信号吗 | ❌ 丢弃 | ✅ 作 per-state 对比负样本 |
| 数据规模 | Track-A 300/任务；Track-B 仅 UMI | 每轮少量干预对，K=3 轮迭代 |
| 解决什么 | 建立基础能力 + 跨本体先验 | 补「最后一毫米」长尾鲁棒性 |

### 3.3 工程亮点：RPRO loss（来自原文 Eq.6–9）

```
隐式 reward 代理:  ℓ_θ(s,a)=E‖v_θ(a_t,t|s)−(a−ε)‖²     (Eq.6, flow-matching 回归当 NLL 代理)
                  r_θ(s,a)=(β/2)(ℓ_ref(s,a)−ℓ_θ(s,a))   (Eq.7, ℓ_ref 来自 frozen 上一轮策略)
RPRO:  L_PRO = −E[ logσ(r_w−r_l)                          ← 对比项 L_con
                  + Σ_{a∈{a_w,a_l}} ½(logσ(r_θ(a))+logσ(−r_θ(a))) ]  ← 近端正则 L_reg
       L_RPRO = λ_PRO·L_PRO + λ_SFT·E[ℓ_θ(s,a_w)]         (Eq.9)
```

- **L_reg 防 reward hacking**：在 $r=0$ 处最小、随 $|r|$ 对称增长 → 锚定隐式 reward 绝对值。plain-DPO 只管 $r_w-r_l$ 的差，优化器可把 a_w/a_l likelihood **一起压崩**（差仍变大但策略漂离两者）；L_reg 结构性禁止这条捷径。
- **对比梯度抵消**：$a_w=a_l$ 时 $\nabla_\theta L_{con}=0$（两偏导等大反号、梯度相同），SFT 样本可安全走同一 loss。
- **数字例**（demo 复现）：健康解 r=(1,−1) → L_total 1.75；hack 解 r=(−4,−6) → L_total 5.15（L_con 相同 0.127，L_reg 暴涨惩罚）。

---

## 4. FlowPRO 迭代循环（reward-free 偏好式离线 RL）

> 通用机制见 [[偏好式 Flow 策略优化（Flow-DPO·RPRO）]]，这里讲本文具体细节。

### 4.1 每轮三步
```
① 干预-回滚收偏好对 ── rollout 出错→回退 Δ 步，执行段记 τ_l，纠正演示记 τ_w（共享初始状态）
        ↓
② Smooth Interpolation ── cubic Bézier(位置)+Slerp(姿态)+线性(夹爪) 合成缺失对应动作
                          → 稀疏轨迹级偏好补成稠密 per-state (s, a_w, a_l)
        ↓
③ RPRO loss 优化 ── 混批(新对 + 历史对 + SFT)；上一轮策略当下一轮 π_ref
        ↺ 迭代 K=3 轮
```

### 4.2 关键细节
| 细节 | 值 / 做法 | 备注 |
| :--- | :--- | :--- |
| 批混比 | k=1: 80%/20%（新对/SFT）；k≥2: 70%/15%/15%（新对/历史/SFT） | 上权新失败、replay 防回退、锚基础能力 |
| 隐式 reward | flow-matching loss 差，frozen π_ref | 不训 reward 网络 |
| 动作积分 | 10 步 Euler，δ=0.1 | 部署推理 |

### 4.3 ⚠️ reward-free ≠ human-free
偏好对靠**遥操作员实时干预**产生——不训 reward 模型，但仍是 human-in-the-loop，规模化人力成本论文未讨论。

---

## 5. 关键消融 / 实验证据

- **仿真 RoboTwin 2.0（50 任务）**：Clean 90.9 / Randomized 90.1 SOTA；超 π0 +25.0/+31.7，超 π0.5 +8.2/+13.3。
- **消融**：去记忆编码器 90.9→88.8；再去 UMI 预训 →88.1（仿真增益小，真机增益大，因域差小）。
- **真机 FlowPRO（Dobot X-Trainer，4 任务，K=3，3 seed×100 rollout）**：RPRO 99/99/98/94，SFT 起点 93/88/86/83，DAgger 95/95/95/89。RPRO **SR 最高且完成时间最短**。
- **RPRO > π0.6\***（同一偏好数据）：π0.6\* 把 advantage 当 conditioning token 注入是**间接压力、被 VLM 上下文稀释**；RPRO per-state 对比更直接。← **见 §6 与我的关系**。

---

## 6. 跟我有啥关系

**我的方向**：[[强化学习后训练 主题地图|RL + 基础模型后训练]]（子方向：[[Diffusion-Flow 策略的 RL 改进路线图|flow 策略的 RL 改进]]）。

### 6.1 创新点 × 我的反应

> [!note] 第二、三列是初稿判断，待程程复核/重写——这一栏是整篇笔记的灵魂

| 论文的创新点 | 启发/借鉴/改造/不适用 | 怎么用 / 为啥不用 |
| :--- | :--- | :--- |
| 用 flow-matching loss 当**隐式 reward**（免 critic） | 🔧 直接借鉴 | 我做 flow 策略 RL 时可直接用，绕开无闭式 log-prob 的老问题 |
| **近端正则**锚定隐式 reward 防 hacking | 🔧 直接借鉴 | 我的连续 advantage 方案也该带它做消融，防 likelihood collapse |
| **对比梯度抵消**安全混 SFT | 💡 启发 | 混合 RL+SFT 训练的干净机制，可复用 |
| π0.6\*（advantage 当 conditioning token） | ⚠️ 需改造 | **这几乎是我 [[Diffusion-Flow 策略的 RL 改进路线图\|路线④]] idea 的实现，且被 RPRO 比下去**——我必须正面回应「token 注入被稀释」的批评 |
| RPRO 只用**二元偏好**（丢 advantage 幅度） | 💡 启发（这是缝隙） | 我的差异化 = 把它推广成**保留 advantage 幅度的连续加权、直接进速度场** |
| delta-chunk 相对 EEF 表示 | 🔧 直接借鉴 | 跨本体动作表示的现成 baseline |
| Smooth Interpolation（稀疏→稠密 per-state） | 💡 启发 | 任何「轨迹级监督→稠密 per-state」场景可借 |

### 6.2 其他启发
- FlowPRO 在我那张 [[Diffusion-Flow 策略的 RL 改进路线图]] 上**补了一条新路线（③c 偏好式/隐式 reward）**——原图四条没有它。
- 它给我的旗舰 idea [[flow v_target advantage（研究想法）|v_target advantage]] 提供了**最强对标 baseline**和**清晰差异化故事**。

### 6.3 我可能要做的下一步
- [ ] 复现 RPRO loss（`~/claude-papers/.../code/flowpro_rpro_demo.py` 已有概念版），跑通对比梯度抵消
- [ ] 把对比项换成**连续 advantage 加权**，玩具连续控制任务上对照「二元 RPRO vs 连续加权」
- [ ] 设计与 π0.6\* / RPRO 的公平对照（同偏好数据、同 SFT 起点、同轮数，只换信号注入方式）→ 走 /grill → /plan 立项

---

## 7. 待理解 / 存疑

- [ ] Eq.6 用 per-sample flow-matching 回归当 NLL 代理，**偏差量级未知**——r_θ 的排序在高维动作上还可靠吗？
- [ ] Smooth Interpolation 合成的 a_w 段直接进对比 loss，**合成误差会不会反向带偏**？论文无敏感性分析。
- [ ] β、λ_PRO、λ_SFT、批混比都是经验值，换任务鲁棒性？
- [ ] FlowPRO 只验 1 平台 4 任务 K=3，长 horizon / 移动操作 / 更多本体上是否还稳？
- [ ] π0.6\* 是作者复现的 `*` 版本，对照公平性需保留。

---

## 8. 相关工作

- **前作（同源 backbone）**：[[hy-embodied-0.5（base 报告）]]（arXiv:2604.07430，base VLM 报告，真机部分只是 SFT）
- **同流派（flow 策略 RL）**：[[偏好式 Flow 策略优化（Flow-DPO·RPRO）]]（Flow-DPO / GRAPE / RPRO）
- **对照组**：π0.6\*（=RECAP，advantage-conditioned，见 [[Advantage-conditioned 微调]]）· DAgger · HIL-SERL
- **谱系图**：[[Diffusion-Flow 策略的 RL 改进路线图]]（FlowPRO 补的 ③c 新路线）· [[强化学习后训练 主题地图]]
- **动作头血缘**：π0 / π0.5（flow-matching VLA）· UMI（手持演示数据）
