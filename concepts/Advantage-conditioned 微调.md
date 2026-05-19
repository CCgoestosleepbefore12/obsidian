# Advantage-conditioned 微调

> [!info] 状态：积累中  
> 这是一个**训练方法型**概念，每读到新实例就扩一行。  
> 关联：[[RL 与模型后训练]]｜对照：[[PPO]] [[DSRL]] [[DPO]]

**Tags**: #concept #post-training #RL #VLA

---

## 一句话定义

> [!note] 待填（你自己写）  
> 提示：关键词是「把 advantage 当作条件输入」「不直接最大化奖励」「概率推断式 RL」「推理时 prompt adv=1」

---

## 核心思想

**不**训练新策略最大化奖励，而是**把 advantage 离散化后当条件**喂给原策略——让模型学"在 advantage = 高 这个条件下应该输出什么动作"。推理时只需 prompt `advantage = 1`，就能生成"最优改进"动作。

数学上是把策略改进表达成贝叶斯后验：

`π̂(a|o,ℓ) ∝ π_ref(a|o,ℓ) · p(I | A_{π_ref}(o,a,ℓ))^β`

其中 `I` 是"improvement"事件。β=1 时简化为：

`π̂(a|o,ℓ) = π_ref(a | I, o, ℓ)`

→ 训练时把 advantage 标签作为输入条件喂进 VLA，推理时把条件设为"最大改进"。

---

## Advantage 离散化（bin）

**做法**：把连续 advantage `A ∈ ℝ` 切成 **N 个桶**（bin），用桶编号 `bin ∈ {0, 1, ..., N-1}` 作为 categorical token 喂给 policy。常见 N = 5~10。

```
连续 A ──────────────── 离散 bin（N=5）
   +1.0  ━━━━━━━━━━━━━━━ bin 4   ← "最大改进"（推理时 prompt 这个）
   +0.5  ━━━━━━━━━━━━━━━ bin 3
    0.0  ━━━━━━━━━━━━━━━ bin 2   ← "中性"
   −0.5  ━━━━━━━━━━━━━━━ bin 1
   −1.0  ━━━━━━━━━━━━━━━ bin 0   ← "倒退"
```

**为啥不直接喂连续 A？**

| 原因 | 解释 |
| :--- | :--- |
| 架构兼容 | Flow-matching / diffusion / Transformer 都更擅长 categorical token，处理连续标量不稳 |
| 信号干净 | "bin = 4" 是清晰类别；A = 0.83 容易和噪声混 |
| 类比有先例 | Decision Transformer 的 return-to-go 也是离散桶 |
| 推理省事 | 部署时只需 `prompt: bin = N-1`，不需算具体 advantage |

> [!warning] 重要陷阱：**Uniform bin vs Quantile bin**  
> ❌ **均匀分桶**（按数值等分）：advantage 分布往往偏向 0，多数样本挤在中间几桶，两端利用率极低  
> ✅ **分位数分桶**（每个桶等量样本）：自动适配分布，所有桶都被充分训练  
> 这跟数据归一化里的 quantile transform 是同一思路。

---

## 三类训练数据（最关键的设计点！）

| 数据类型 | 来源 | advantage 标签 | 为什么 |
| :--- | :--- | :--- | :--- |
| **Expert** | 人类专家示范 | **强制 = 1** | 外部已验证最优；让 V 打分会引入 V 的偏差 |
| **Correction** | DAgger 风格人类介入纠错 | **强制 = 1** | 同上，外部 supervision |
| **Rollout** | 策略自跑（真机或 imagination） | **由 V 模型打分** | 内部 self-evaluation；V 是唯一信号源 |

> [!tip] 核心原则：**外部 supervision 用外部标签，内部 self-evaluation 才用 V 模型**。混在一起 = V 噪声污染干净标签。

> [!warning] RECAP vs RISE 区别  
> [[RECAP]] 原始做法：**所有数据**（包括 expert）都让 V 模型打 advantage。  
> [[RISE]] 修正：expert + correction 强制 adv=1，仅 rollout 用 V 打分。  
> 这是「self-improving 流派」的成熟工程细节，看起来小但影响很大。

---

## 为什么比 PPO / DSRL 更稳？

- **不动 reference policy 分布** → 避免 KL blow-up
- **不需要重新参数化 diffusion / flow-matching** → chunk 输出可以直接用
- **推理时仅 prompt `adv = 1`** → 零额外计算

实验证据（RISE Brick Sort 任务）：

| 方法 | 成功率 |
| :--- | :--- |
| π₀.₅（baseline）| 35% |
| π₀.₅ + PPO | 10%（↓） |
| π₀.₅ + DSRL | 10%（↓） |
| **π₀.₅ + advantage-conditioned (RISE)** | **85%（↑）** |

PPO 和 DSRL 在 chunk-output VLA 上反而把成功率打下去——advantage-conditioned 才是 work 的范式。

---

## 已知实例

| 论文 | advantage 来源 | 创新点 | 备注 |
| :--- | :--- | :--- | :--- |
| [[RECAP]] (π₀.₆) | VLM progress 模型 | **首次提出**该范式 | 离线 + 部分 on-policy |
| [[RISE]] | dynamics rollout → value (progress + TD) | 把 advantage 升级为 WM 内循环；修正 expert 数据处理 | 想象空间 on-policy |
| [[π₀.₅]] | _以后读到再填_ | | |

---

## 对立思路

- **[[PPO]]**：直接 policy gradient 最大化 reward；对 diffusion / chunk output 适配难
- **[[DSRL]]**：只在 noise input 上做 RL，不动主权重——稳但能力有限
- **[[DPO]]**：偏好对学习；需要成对偏好数据；不直接用 advantage

---

## 我的判断（持续更新）

> [!question] 在 [[RL 与模型后训练]] 方向上，advantage-conditioned 比 PPO / DPO 更适合什么场景？  
> _读完 RECAP / π₀.₆ 原文后回来填_

---

## Backlinks
（Obsidian 自动维护）
