---
tags:
  - concept
  - RL
  - post-training
  - diffusion-policy
created: 2026-06-16
---

# 偏好式 Flow 策略优化（Flow-DPO · RPRO）

> [!info] 状态：积累中
> 这是一个**训练方法型**概念，每读到新实例就扩一行。
> 关联：[[Diffusion-Flow 策略的 RL 改进路线图]]（这是其 ③c 路线的细化）｜对照：[[Advantage-conditioned 微调]]（④改条件）[[DPO]]

**Tags**: #concept #post-training #RL #diffusion-policy

---

## 一句话定义

> [!note] 待填（你自己写）
> 提示：关键词是「flow 无闭式 log-prob」「用回归损失当隐式 reward」「成对偏好对比」「近端正则防 reward hacking」

---

## 核心思想 / 为什么这样做

连续动作的 flow/diffusion 策略**没有闭式 log-prob**，DPO 那套「用对数似然比当隐式 reward」直接搬不过来。这一路的解法分两步：

1. **用每样本的 flow-matching 回归损失当负对数似然的代理**——速度场对某动作拟合越好（回归 loss 越小），就视作策略「越认为该动作可能」。于是隐式 reward 可闭式算：

   `r_θ(s,a) = (β/2)·( ℓ_ref(s,a) − ℓ_θ(s,a) )`

   其中 ℓ 是 flow-matching 回归损失，ℓ_ref 来自冻结的参考策略。**全程不训 reward/value/critic**。

2. **在隐式 reward 上做成对偏好对比**（拉向偏好动作 a_w、推离 a_l）。

### 关键设计：防 reward hacking 的近端正则

朴素 DPO 只约束 `r(a_w) − r(a_l)` 的**差**，留了一个捷径：把 a_w、a_l 的 likelihood **一起压崩**（只要 a_l 压得更狠，差仍变大），结果策略漂离两者、塌到没见过的烂区域。

修法：加一个**对称近端正则**把隐式 reward 的**绝对值**锚回 0（在 r=0 处 loss 最小、随 |r| 对称增大），结构性禁止这条捷径。

### 副性质：对比梯度抵消
对比项只依赖 (r_w − r_l)，两个偏导恒等大反号。当正负样本相同（a_w=a_l）时它们指向同一参数梯度 → 相加为 0。于是把纯监督样本（设 a_w=a_l）塞进同一条 loss 是安全的，它自动退化成「带正则的 SFT 样本」——可在一个 batch 里混偏好对 + 监督样本而不打架。

---

## 已知实例

| 论文 | 关键实现细节 | 备注 |
| :--- | :--- | :--- |
| [[Hy-Embodied-0.5-VLA (FlowPRO)\|FlowPRO / RPRO]] | flow-matching loss 当隐式 reward + 近端正则锚定绝对值 + 对比梯度抵消混 SFT；干预-回滚 + Smooth Interpolation 收稠密 per-state 偏好对 | reward-free 真机离线 RL；防 hacking 的 per-state Flow-DPO |
| Flow-DPO | 首提「per-sample flow-matching 回归当 NLL 代理」做 DPO；RPRO 的直接前身 | _读到补细节_ |
| GRAPE | 轨迹级偏好的 flow 策略优化 | _读到补；RPRO 批评其 per-state 信号被稀释_ |
| _未来论文_ | _以后读到再填_ | |

---

## 对立思路

- **[[Advantage-conditioned 微调]]（④改条件，如 RECAP / π0.6\*）**：把 advantage 当 conditioning token 注入、纯回归目标，不做对比。RPRO 实测优于 π0.6\*，归因于「单 token 间接压力被上下文稀释」；但 advantage-conditioned **保留了 advantage 的标量幅度**，而本路线（二元偏好）**丢掉了幅度**——各有取舍。
- **③a 硬穿链（DPPO / FPO）**：on-policy policy gradient 穿去噪链，贵且不稳；本路线是离线、免 critic、更省。
- **③b 绕链（QAM / FQL）**：靠 Q/critic 驱动；本路线不训 critic。

---

## 我的判断（持续更新）

> [!question] 什么时候选「偏好式」而不是「advantage-conditioned」或「Q 驱动」？二元偏好丢掉的 advantage 幅度，值不值得用连续加权找回来（代价是什么）？
> _至少读完 Flow-DPO + GRAPE 两个实例再回来填，不要现在凭空猜_
>
> 🔬 与我的研究直接相关：我的 [[flow v_target advantage（研究想法）]] 本质是「保留 advantage 幅度的连续加权」，可视为本路线（二元 RPRO）与 ④（advantage-conditioned）的**融合/推广**——既要 per-state 直接注入（不被稀释），又要保留幅度（信息量），还要近端正则防 hacking。

---

## Backlinks
（Obsidian 自动维护）
