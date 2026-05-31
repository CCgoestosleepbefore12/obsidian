---
tags:
  - inference-dynamics
  - flow-matching
created: 2026-06-01
---

# 🧮 Legato 速度场推导

> [!info] 这是什么
> 把 **Legato**（Learning Native Continuation for Action Chunking Flow Policies，[arXiv 2602.12978](https://arxiv.org/abs/2602.12978)，RSS 2026）"改 loss"的数学原理逐公式讲清楚。
> 关联：[[推理动力学 多维对比]]（§4 把它归为"③ 真内化训练目标"）｜[[强化学习后训练 主题地图]]（这套反解范式是"把 advantage 写进 v_target"研究想法的数学模板）｜标签 `#flow-matching`
> 可信度：arXiv HTML v1 逐公式核实 + 自验算 Eq 15 自洽（高）；非 PDF 逐行，个别下标若与正式版有出入以 PDF 为准。

---

## 0. 一句话

Legato 的本质：**把"推理时会施加的续接引导"写进 ODE，反解出"网络该回归的新目标 $v_\text{target}$"，使训练分布 = 推理分布。** 改的不是采样代码，是回归目标本身——所以衔接成了策略的原生能力。

---

## 1. Baseline：标准 flow matching 的约定

> [!warning] 符号方向
> Legato 的时间约定**和 π₀/FLASH 相反**：这里 $t=0$ 是纯噪声、$t=1$ 是干净动作 $\mathbf{A}$。读 π₀ 系代码时注意别串。

线性插值路径与速度目标（Eq 1-2）：

$$\mathbf{X}_t = (1-t)\,\boldsymbol{\epsilon} + t\,\mathbf{A}, \qquad \mathbf{u}^{FM}(\mathbf{X}_t, t) = \mathbf{A} - \boldsymbol{\epsilon}$$

一条从噪声到动作的直线，匀速。标准训练就是回归这个常数速度 $\mathbf{A}-\boldsymbol{\epsilon}$。

---

## 2. 续接的需求 → per-step guidance

推理时新 chunk 的**前缀动作已知**（来自上一 chunk 重叠段已算好的结果），希望新 chunk 在这些位置锚定到已知动作、之后平滑 ramp 掉。

用 **continuation vector** $\boldsymbol{\omega}\in[0,1]^H$ 编码每个 horizon 位置的引导强度，由两个标量唯一确定——推理延迟 $d$、ramp 长度 $r$：前 $d$ 步 $\omega=1$（完全锚定），接着 $r$ 步线性降到 $0$，其余 $\omega=0$（自由生成），约束 $r+s+d=H$（$s$ 为每周期执行步长）。

施加方式：**每步去噪前先把状态往已知动作 $\mathbf{A}$ 拉一下**（Eq 9-10）：

$$\mathbf{Y}_k = (\mathbf{1}-\boldsymbol{\omega})\odot\mathbf{X}_k + \boldsymbol{\omega}\odot\mathbf{A} \quad\text{[guidance]}$$
$$\mathbf{X}_{k+1} = \mathbf{Y}_k + \Delta t\cdot f_\theta(\mathbf{Y}_k, t_k) \quad\text{[denoising]}$$

这就是 RTC 那种 inpainting 衔接的"逐步连续版"——每一步都重新注入已知前缀。配套地，初始化也从 action-noise 混合出发（Eq 6-7）：

$$\boldsymbol{\epsilon}_\text{eff} = (\mathbf{1}-\boldsymbol{\omega})\odot\boldsymbol{\epsilon} + \boldsymbol{\omega}\odot\mathbf{A}, \qquad \mathbf{Y}_t = (1-t)\,\boldsymbol{\epsilon}_\text{eff} + t\,\mathbf{A}$$

---

## 3. 核心矛盾：guidance 破坏训练-推理一致性

若 $f_\theta$ 是**标准 FM 训练**出来的（它只见过没被拉扯的直线轨迹），但推理时每步都把轨迹拉偏 → 模型从未见过这种被反复拉扯的输入分布 → train-infer mismatch、误差累积。

把 guidance + denoising 两式消去 $\mathbf{X}_k$、取连续时间极限，得到推理时**真实发生的有效动力学**（Eq 11→12）：

$$\dot{\mathbf{Y}}(t) = (\mathbf{1}-\boldsymbol{\omega})\odot f_\theta(\mathbf{Y},t) - \boldsymbol{\kappa}\odot(\mathbf{Y}-\mathbf{A}), \qquad \boldsymbol{\kappa} = \boldsymbol{\omega}/\Delta t$$

轨迹实际按这个 ODE 走，**不是**按 $f_\theta$ 单独走——多出一个回拉项 $-\boldsymbol{\kappa}\odot(\mathbf{Y}-\mathbf{A})$。**这就是 mismatch 的数学根源。**

---

## 4. Legato 的解法：反解，让有效动力学 = 正确的 FM 速度

要求很干脆——**强行让推理时的有效速度，恰好等于轨迹本该遵循的标准 FM 速度** $\mathbf{u}^{FM}$（Eq 13，consistency 约束）：

$$(\mathbf{1}-\boldsymbol{\omega})\odot f_\theta(\mathbf{Y},t) - \boldsymbol{\kappa}\odot(\mathbf{Y}-\mathbf{A}) = \mathbf{u}^{FM}(\mathbf{Y},t)$$

左边是推理时真实发生的，右边是想要的。**解出网络该输出什么**（Eq 14，逐元素求逆）：

$$f_\theta(\mathbf{Y},t) = (\mathbf{1}-\boldsymbol{\omega})^{-1}\odot\big[\,\mathbf{u}^{FM}(\mathbf{Y},t) + \boldsymbol{\kappa}\odot(\mathbf{Y}-\mathbf{A})\,\big]$$

所以网络的回归目标**不再是 $\mathbf{A}-\boldsymbol{\epsilon}$**，而是这个被 reshape 过的东西。

---

## 5. 化简成训练目标（Eq 15）

沿 Legato 的插值路径，$\mathbf{u}^{FM}(\mathbf{Y}_t)=\mathbf{A}-\boldsymbol{\epsilon}_\text{eff}=(\mathbf{1}-\boldsymbol{\omega})\odot(\mathbf{A}-\boldsymbol{\epsilon})$（Eq 8）。代入 Eq 14 化简，得最终目标：

$$\boxed{\;\mathbf{v}_\text{target}(t,\mathbf{A},\boldsymbol{\epsilon},\boldsymbol{\omega}) = \big(1 - \boldsymbol{\kappa}\odot(1-t)\big)\odot(\mathbf{A}-\boldsymbol{\epsilon})\;}$$

> [!note] 自验算（关键一步）
> 关键恒等式 $\mathbf{Y}_t-\mathbf{A} = -(1-t)(\mathbf{1}-\boldsymbol{\omega})\odot(\mathbf{A}-\boldsymbol{\epsilon})$。代入 Eq 14：
> $$f_\theta = (\mathbf{1}-\boldsymbol{\omega})^{-1}\odot\big[(\mathbf{1}-\boldsymbol{\omega})(\mathbf{A}-\boldsymbol{\epsilon}) - (1-t)\boldsymbol{\kappa}\odot(\mathbf{1}-\boldsymbol{\omega})(\mathbf{A}-\boldsymbol{\epsilon})\big] = \big(1-\boldsymbol{\kappa}\odot(1-t)\big)\odot(\mathbf{A}-\boldsymbol{\epsilon})$$
> $(\mathbf{1}-\boldsymbol{\omega})$ 正好约掉。✓

损失就是对它做 L2 回归（Algorithm 1）：

$$\min_\theta\;\big\|\,f_\theta(\mathbf{Y}_t, o, t, \boldsymbol{\omega}) - \mathbf{v}_\text{target}\,\big\|_2^2$$

训练流程：采样 $t\sim\mathcal{U}(0,1)$、$\boldsymbol{\epsilon}\sim\mathcal{N}(\mathbf{0},\mathbf{I})$、随机 $(d,r)\to\boldsymbol{\omega}$ → 构造 $\boldsymbol{\epsilon}_\text{eff}, \mathbf{Y}_t$ → 算 $\mathbf{v}_\text{target}$ → 回归。

---

## 6. 数学直觉（最该记住的）

对比标准目标 $\mathbf{A}-\boldsymbol{\epsilon}$（常数），Legato 只多了一个逐元素调制因子 $\big(1-\boldsymbol{\kappa}\odot(1-t)\big)$：

- **它在做 double-counting 抵消**：推理时回拉项 $-\boldsymbol{\kappa}(\mathbf{Y}-\mathbf{A})$ 已替网络把轨迹往 $\mathbf{A}$ 推了一截，所以网络只需输出"剩下该走的"速度，否则前缀会过冲。这个因子把输出**压低**，正好抵消引导项的贡献。
- **只改幅值、不改方向**：标量因子乘在 $(\mathbf{A}-\boldsymbol{\epsilon})$ 上，几何方向不变——论文反复强调的 "preserve geometric direction, reshape magnitude"。
- **$\omega=0$ 处完美退化**：自由生成位置 $\kappa=0$ → $\mathbf{v}_\text{target}=\mathbf{A}-\boldsymbol{\epsilon}$，完全退回标准 FM。**只有续接位置被动手术，其余原封不动。**
- **$t\to1$（近终点）**：$(1-t)\to0$，因子 $\to1$，也回标准速度；$t$ 小时因子被压低（引导最强的早期阶段）。
- **$\boldsymbol{\omega}$ 进入三处**：初始化 $\boldsymbol{\epsilon}_\text{eff}$、网络条件输入 $f_\theta(\dots,\boldsymbol{\omega})$、目标里的 $\boldsymbol{\kappa}$。训练随机采 $(d,r)$，故**一个模型适配不同推理延迟**，无需重训。

---

## 7. 回到 RL 后训练（为什么这页对我重要）

Legato 给了一个**走通的、可照搬的反解范式**：

> 写出"推理时会施加的引导"对应的有效 ODE → 反解出网络该回归的新目标 $v_\text{target}$ → 回归它，保证训练=推理。

这正是 [[强化学习后训练 主题地图]] 里那条研究想法的数学模板：要把 **chunk 级 advantage 内化进 flow 策略**，套同一招——把"推理时的 Q/advantage 引导"写成有效 ODE，反解出 $v_\text{target}^{RL}$ 去回归，而非推理期再做 Q 引导。只是把 Legato 的"前缀锚定引导"换成"价值引导"。这能同时治 train-infer mismatch 和多步采样昂贵两个痛点。

---

## Backlinks
（Obsidian 自动维护）
