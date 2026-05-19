# Compositional World Model

> [!info] 状态：积累中  
> 这是一个**设计模式型**概念，每读到新实例就扩一行。  
> 关联：[[世界模型架构对比]]｜反例：[[Dreamer 系列]]

**Tags**: #concept #world-model #design-pattern

---

## 一句话定义

**Compositional World Model** 把"世界模型"这个整体拆成几个**性质截然不同的子问题**（典型是 dynamics + value），让每块用最适合自己的架构和训练目标**独立优化**——而不是用一个大模型同时端到端解决所有事。

---

## 为什么要拆？

把 World Model 拆成 **dynamics** 和 **value** 两个独立优化的子模型——因为它们本质是两类完全不同的问题，合一会让其中一头被拖累。

| 维度    | Dynamics | Value |
| :---- | :------- | :---- |
| 输入  | 上下文帧 + 动作 chunk + 语言指令 | 状态/帧（+ 可选动作）|
| 输出空间  | 高维（多视角图像序列）| 低维（标量）|
| 时间结构  | 逐帧自回归预测未来 | 整段轨迹的"进度感" |
| 训练目标 (loss) | 视频去噪 / 扩散 loss（重建未来帧）| 进度回归 (MSE) + TD 自举 |
| 训练数据  | **自监督**：任意机器人交互视频，无需 reward 标注 | **弱监督**：必须有终态 success/failure 标签 |
| 最适架构  | 视频扩散 / DiT | VLA / VLM backbone |
| 核心关注点 | 视觉真实 + **动作可控**（同一场景下不同动作 → 不同未来）| **失败敏感** + 进度感知 |

> [!tip] 答案在 `insights.md` 第 2 节，但**先自己想再对照**。

---

## 已知实例

| 论文 | Dynamics 怎么做 | Value 怎么做 | 应用场景 |
| :--- | :--- | :--- | :--- |
| [[RISE]] | 视频扩散 (GE/LTX) + Task-Centric Batching | π₀.₅ backbone + progress + TD | 真机 manipulation |
| [[V-JEPA-2]] | _以后读到再填_ | _以后读到再填_ | _planning_ |
| [[Video Language Planning]] | _以后读到再填_ | _以后读到再填_ | |

---

## 对立思路：Monolithic（一锅炖）

- **代表**：[[Dreamer 系列]] —— dynamics + value 都在同一个 RSSM latent 空间，actor-critic 联合训
- **优点**：端到端、参数少、可联合优化
- **缺点**：高维视觉任务下两边互相拖累；动作可控性弱

---

## 我的判断（持续更新）

> [!question] 什么时候该选 compositional？什么时候 monolithic 更好？  
> _至少读完 2-3 个实例再回来填，不要现在凭空猜_

---

## Backlinks
（Obsidian 自动维护，不用手写）
