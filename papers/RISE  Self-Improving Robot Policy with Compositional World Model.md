# RISE: Self-Improving Robot Policy with Compositional World Model

**读于**: 2026-05-14  
**Tags**: #paper #world-model #VLA #model-based-rl  
**原始分析**: http://localhost:5815/papers/rise-self-improving-robot-policy

---

## 1. 一句话总结

RISE 解决的是「VLA 真机做 on-policy RL 太贵 + 仿真器有 sim2real gap」的问题——用学到的世界模型代替，在想象空间做 on-policy RL，**零真机 trial-and-error**。WM 拆成 dynamics（视频扩散）+ value（progress + TD）两块独立优化，policy 用 advantage-conditioned 微调（继承 RECAP），在 imagination loop 里循环改进。真机任务从 ~35% 提到 **85%+**。

---

## 2. 它要解决什么问题

- **2.1 问题本身**：
- **2.2 之前为啥没解决**：

---

## 3. 方法核心（三个组件）

整体设计哲学是 [[Compositional World Model]]——把 WM 拆成两个性质不同的子问题，各用最适合的架构。

### 3.1 三大组件概览

1. **Dynamics**：[[视频扩散动力学]] —— Task-Centric Batching 让模型学动作响应
2. **Value**：[[Progress + TD 价值学习]] —— 双目标补足彼此短板
3. **Policy**：[[Advantage-conditioned 微调]] —— 继承 RECAP，在想象空间自我改进

### 3.2 Dynamics vs Value 维度对照表

> 这张表跟 [[Compositional World Model]] 的抽象表对照看：同样的 7 个维度，这里填的是 RISE 的**具体实现**。

| 维度 | Dynamics | Value |
| :--- | :--- | :--- |
| 输入 | 过去多视角帧 + action chunk + 语言指令 | 多视角帧（dynamics 展开生成）+ action chunk |
| 输出空间 | H 帧 × 多视角 RGB（高维） | 标量 progress score |
| 时间结构 | 视频扩散一次预测 H 帧未来 | 单帧打 progress，整段轨迹用 TD bootstrap |
| 训练目标 (loss) | Flow-matching / 视频去噪 loss（+ Task-Centric Batching 采样策略）| progress MSE + TD error 双目标 |
| 训练数据 | 预训练：Galaxea + AgiBot World；微调：任务相关轨迹 | 与 dynamics 共享轨迹；progress 自动算 (t/T)，TD 用终态 ±1 |
| 最适架构 | GE-Base / LTX-Video（视频扩散 DiT）| π₀.₅ VLA backbone |
| 核心关注点 | 视觉真实 + 动作可控（**Task-Centric Batching 强化**）| 失败敏感（**progress + TD 双目标互补**）|

### 3.3 Task-Centric Batching 细节（RISE-specific 工程亮点）

**伪代码**：

```python
# 标准：均匀采样多任务
batch = [random_sample(all_data) for _ in range(B)]

# Task-Centric (RISE)
def sample_batch(B, dataset, k_tasks=4):
    chosen_tasks = random.sample(dataset.tasks, k_tasks)
    samples_per_task = B // k_tasks
    batch = []
    for t in chosen_tasks:
        batch.extend(random_sample(dataset[t], samples_per_task))
    return batch
```

**数字例子**（B=64, k_tasks=4）：

| 方式 | 一个 batch 长啥样 |
| :--- | :--- |
| 标准 | 64 个**不同任务**的样本 → 每对样本场景就换，模型学到「场景 → 未来」走捷径 |
| Task-Centric | 4 个任务 × 16 样本 → 同任务内场景几乎不变，模型**被迫**用动作差异解释未来差异 |

**为啥反直觉地有效**：常规思路追求 batch 内多样性，但 dynamics 模型最难学的是「**动作可控性**」——同一场景下不同动作要产出不同未来。Task-Centric 通过**故意降低场景多样性**，把动作信号从噪声里逼出来。

**消融**：去掉 Task-Centric → 完成度 70% → 40%；EPE（动作响应误差）0.54 → 0.68。

**参数估计**：论文未明说 `k_tasks`，method.md §8 估计 **k ≈ 2~4**。极端 k=1 过拟合，k=B/2 退化成均匀。

---

## 4. Self-Improving Loop（RL 主循环）

> WM 是工具，policy 才是被训练的对象。这一节讲 RISE 的"RL 部分"——dynamics 和 value 如何被串成自我改进的训练循环。通用机制见 [[Advantage-conditioned 微调]]，这里讲 RISE 的具体细节。

### 4.1 三阶段流程
![[rise流程图.png]]
### 4.2 关键工程细节

| 细节 | 推荐值 | 偏离会怎样 |
| :--- | :--- | :--- |
| **EMA decay** | 0.99 ~ 0.995 | 太高（0.999）→ π_rollout 跟不上提升；太低（0.9）→ 抖动 |
| **Offline-online 比例** | 0.6 offline | 0.1 → 5%（灾难性遗忘）；0.9 → 30%（过约束）|
| **Chain rollout 步数** | ≤ 2 | 太长 → 视频扩散误差累积，V 评分越来越不靠谱 |
| **Bin 离散化方式** | **Quantile-based** | Uniform → advantage 分布偏度导致部分桶利用率极低 |
| **Bin 数 N** | 5 ~ 10 | 论文未明说；太少粒度差，太多稀疏 |

### 4.3 推理时（部署）

仅 `π_behavior` 上线。喂 `advantage_token = N-1`（最高桶 = "最优改进"）即可。**Dynamics + Value 完全不参与推理**——零推理开销，这是论文反复强调的卖点。

```python
def deploy(π, o_t, ℓ):
    return π(o_t, ℓ, adv_token=N-1)
```

---

## 5. 关键消融（最具说服力的实验证据）

- 去 Task-Centric：完成度 70% → 40%
- 去 TD：70% → 35%
- 去 progress loss：70% → 50%
- 去 Dynamics 预训练：70% → 15%（最敏感的一项）
- Offline 比例：0.6 → 70%；0.1 → 5%；0.9 → 30%
- WM 速度对比：Cosmos > 10 min vs RISE ≈ 2 s（**进 RL 内循环的硬性条件**）

---

## 6. 跟我有啥关系

**我的方向**：[[RL 与模型后训练]]（其下子方向：[[灵巧手 World Model RL]]）

### 6.1 RISE 创新点 × 我的反应（对照表）

> 每一行选一个标签：💡 启发 / 🔧 直接借鉴 / ⚠️ 需改造 / ❌ 不适用。
> 然后在第三列写**具体怎么用 / 为啥不用**——这是这张表的核心。

| RISE 的创新点 | 启发 / 借鉴 / 改造 / 不适用 | 怎么用 / 为啥不用（具体说） |
| :--- | :--- | :--- |
| Compositional WM（拆 dynamics + value） | _待填_ | _待填_ |
| 视频扩散 dynamics（vs latent dynamics） | _待填_ | _待填_ |
| Task-Centric Batching | _待填_ | _待填_ |
| Progress + TD 双目标 value | _待填_ | _待填_ |
| Advantage-conditioned policy + bin 离散化 | _待填_ | _待填_ |
| 三类数据处理（expert/correction 强制 adv=1） | _待填_ | _待填_ |
| Imagination space RL（替代真机/sim） | _待填_ | _待填_ |

### 6.2 其他启发（表格装不下的自由想法）

- 

### 6.3 我可能要做的下一步

- [ ] 

---

## 7. 待理解 / 存疑

- [x] **RECAP / RISE 用的三类训练数据**（expert / correction / rollout）分别是什么？它们的 advantage 标签为何处理方式不同？
  - → 已搞懂：expert + correction 是**外部已验证最优** → 强制 adv = 1；rollout 是**策略自评信号** → 由 V 模型打分。RISE 修正了 RECAP "所有数据都让 V 打分"的做法。核心原则：**外部 supervision 用外部标签，内部 self-evaluation 才用 V 模型**——别让 V 的偏差污染干净的 expert 标签。详见 [[Advantage-conditioned 微调]]。
- [x] **Task-Centric Batching 的 batch 内集中多少个任务？参数怎么调？**
  - → 已搞懂：B=64 时 `k_tasks ≈ 2~4`（论文未明说，method.md §8 估计）。k=1 过拟合单任务，k=B/2 退化为均匀采样。详见 §3.3。
- [ ] Flow-matching loss 在视频扩散里具体怎么算？
- [ ] Bin 离散化时 N 取多少合适？quantile 桶的边界如何动态更新？

---

## 8. 相关工作

- 同流派：[[RECAP]]、[[π₀.₅]]
- 对照组：[[Dreamer 系列]]、[[TD-MPC2]]、[[V-JEPA-2]]
- 谱系图：[[Model-based RL 路线图]]
