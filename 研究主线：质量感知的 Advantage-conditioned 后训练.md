---
tags:
  - RL
  - post-training
  - VLA
  - advantage-conditioned
  - data-quality
created: 2026-07-13
---

# 🎯 研究主线：质量感知的 Advantage-conditioned 后训练

> [!info] 这是什么
> 我的研究主线 spec（2026-07-13 定，撞车检查通过）。**准则：首先有效，其次新颖。**
> 一句话：RECAP 式 advantage-conditioned 后训练按**数据来源**分配标签角色（expert/correction 强制 adv=1、rollout 由 V 打分），但它假设外部数据都是干净的——我的 data_filter 每天都在证明真实采集数据不是。**研究「质量信号应该怎么进 advantage 标签」。**
> 关联：[[Advantage-conditioned 微调]]（宿主概念）｜[[强化学习后训练 主题地图]] §4 问题①｜[[paper-list]]（P0：π*₀.₆ / ROVE）

---

## 0. 研究问题

> **当示范/纠错数据本身质量参差时，质量信号应该以什么方式进入 advantage-conditioned 后训练——只调采样权重，还是改变标签角色本身？**

子问题：
1. 把质量勉强的示范强制 adv=1，对条件化信号的污染有多大？（先证有病，再谈药）
2. 先验质量信号（物理/运动学闸门）能否保护 value 模型不被脏数据带偏——从而破掉「用 V 判质量、V 又是脏数据训的」循环？
3. 低质数据的正确角色是什么：剔除、降权，还是**换一种标签方式继续用**？

---

## 1. 动机：一个裸假设 × 一份日常证据

**裸假设**（攻击面）：RECAP 原文白纸黑字——"corrections are **assumed** to be expert-quality"。[[Advantage-conditioned 微调]] 里 RISE 修正过的那条核心原则「外部 supervision 用外部标签，内部 self-evaluation 才用 V 模型」，**隐含前提是外部 supervision 可信**。整条线（RECAP → RISE → LWD）都把「来源=外部」直接等价于「质量=金标」。

**日常证据**（`~/data_filter` 项目）：真实的 pika/UMI + 遥操数据里，"专家示范"充斥 frozen-arm、spike、tracking 丢失、时间戳错乱——同是示范，四级质量档（keep_high_quality / keep_with_downweight / review / drop）差异巨大。**把 keep_with_downweight 的示范强制 adv=1，等于教策略「这种迟疑/抖动就是最优改进」。**

**为什么先有效**：两个组件（数据 curation、advantage-conditioned）各自被独立验证过强有效——PI 规模化验证 RECAP、RISE 35%→85%、curation 永远涨点。我做的是把它们**接对**，最坏情况也是一条更稳的 RECAP 流水线。且零基建豁口：testbed = `~/recap`（standalone RECAP 复现，含 openpi_libero adapter + sidecar advantage parquet），质量信号 = data_filter 现成输出，真机 = 实验室 FTC 设施。

**为什么可能新**：见 §2——curation 一族只为 BC/SFT 筛数据，没人跨到 advantage 标签这边；ROVE 跨了一半但只靠事后 critic、只管干预数据。

---

## 2. 撞车地形（2026-07-13 检查）

| 邻区 | 代表 | 他们做什么 | 与我的距离 |
| :--- | :--- | :--- | :--- |
| **头号近邻** ⚠️ | **ROVE**（[2606.17011](https://arxiv.org/abs/2606.17011)） | 人类**干预**数据当 mixed-quality，OVE 学 critic **事后**判质量给 advantage 信号；人形 | 重叠最大，必读必比较 |
| curation-for-IL | CUPID（[2506.19121](https://arxiv.org/abs/2506.19121)）/ QoQ（[2603.09056](https://arxiv.org/abs/2603.09056)）/ SCIZOR（[2505.22626](https://arxiv.org/abs/2505.22626)） | 影响函数/自监督**为 BC/SFT** 筛示范 | 一步之遥没跨过来：不碰 RL 标签 |
| 次优示范 IL | DWBC / ISW-BC / ILID 一族 | 判别器给 BC **加权** | 范式不同：加权 ≠ 条件化标签角色 |
| 质量分角色（他域） | LDA-1B（[2602.12215](https://arxiv.org/abs/2602.12215)，📥P0） | 按质量给数据分配**dynamics 训练**角色 | 同思想、不同落点——正是我读它时标的概念候选「按数据质量分配角色」，本主线是它落到 advantage 标签上 |

**对 ROVE 的三条差异化**（= 我的定位空间）：
1. **先验 vs 事后**：ROVE 纯靠学出来的 critic——但 V 自己就是拿脏数据训的，循环依赖；我的闸门信号（spike/tracking/frozen-arm/运动质量）是**独立于 V 的外部物理证据**，能破循环、保护 critic。两者正交可组合。
2. **覆盖面**：ROVE 只管干预数据；我管 **demo + correction + rollout 全分类 × 质量档位** 的系统映射。
3. **场景**：人形干预 vs 双臂 pika/UMI + 遥操**多源**异质数据（手持采集 vs 遥操作是不同的噪声分布——多源本身是变量）。

---

## 3. 方法空间（候选设计，未定死）

核心框架：把 RECAP 的一维角色分配（按**来源**）升级为二维——**来源 × 质量 → 标签角色**：

| | 高质量 | 中等（downweight 档） | 差（review/drop 档） |
| :--- | :--- | :--- | :--- |
| **外部**（demo / correction） | 强制 adv=1（原样） | **⭐ 争议格：降级给 V 打分？温和 bin？** | 剔除 |
| **内部**（rollout） | V 打分（原样） | V 打分 + 降采样权重 | 剔除 |

三个候选设计，按侵入程度从浅到深：

- **设计 C：质量保护的 value 学习**（最便宜，先做）
  质量信号只进 **V 的训练**：用 data_filter 的 sampling_weights 对 V 的回归目标降权低质 transition（或当 label noise 做鲁棒回归）。动机：整条 RECAP 线自己承认 "performance strongly depends on value model quality"——V 被脏数据带偏，则 V 打分的 rollout 标签和 advantage bin 全部连坐。**不改策略训练一行代码**，是最干净的第一刀。
- **设计 A：质量门控的角色映射**（主菜）
  实现上表的 ⭐ 格：中等质量的示范/纠错**不再强制 adv=1，降级为 V 打分**（当作 rollout 对待）——"来源外部但质量存疑"的数据，交给内部评估器裁决。在 `~/recap` 里这只是 advantage parquet 生成逻辑 + mixture_config 的改动，可实现性极高。
- **设计 B：质量进条件**（延伸，做通 A/C 再说）
  把质量档当**第二个条件 token** 喂给策略（`p(a | o, adv, quality)`），推理时 prompt（adv=高, quality=高）——类比 advantage bin 的做法把质量也条件化，让模型自己学「高质行为长什么样」。风险：条件覆盖稀疏、训练配方复杂，收益不确定。

> [!tip] 有效性排序就是实施排序
> C → A → B。C 和 A 各自独立可发信号、组合是完整故事；B 是锦上添花，不押注。

---

## 4. 最小实验（最快证伪路径）

**Testbed 1（受控，先做）：LIBERO + 合成腐蚀**
`~/recap` 已有 openpi_libero adapter。取 LIBERO 示范，按 data_filter 的失败分类学**合成腐蚀**一部分（注入 frozen-arm 停顿 / spike / 抖动），腐蚀比例 {10%, 30%, 50%} 扫——质量档位有 ground truth，变量全受控。

对照组（缺一不可）：
1. **vanilla RECAP**：全部示范强制 adv=1（现状）
2. **oracle 剔除**：直接扔掉腐蚀样本（naive filtering 上界）
3. **设计 C**：质量权重进 V 训练
4. **设计 A**：腐蚀示范降级 V 打分
5. A + C 组合

度量：任务成功率 + V 的校准误差（对腐蚀 transition 的 value 估计偏差）。

**判决读法**：
- ④/⑤ > ① —— 病是真的，药有效 → 推进
- ④/⑤ > ② —— **核心主张成立**：低质数据给对角色比扔掉更值钱（LDA-1B 论点移植到 advantage 标签），这是论文的灵魂
- ④ ≈ ② —— 角色映射不比剔除强 → 降级为 data-filtering 实证研究（仍可发，故事变小）
- ④ ≤ ① —— 前提错了（强制标签污染不成立或太弱）→ 回来重想

**Testbed 2（真实，后做）**：pika/UMI + 遥操数据 × XVLA，data_filter 四级标签是真实的——LIBERO 出信号后再上，同时验证「合成腐蚀 → 真实噪声」的迁移。

---

## 5. 成功判据 + 诚实风险

**成功判据**：混合质量数据下，质量感知角色映射**同时**优于 vanilla RECAP 和 naive 剔除，且 V 校准可量化改善。

**风险清单**：
- **效应量可能小**：advantage 二值化/分桶本身可能洗掉小的标签噪声——所以必须扫腐蚀比例，找到效应显现的区间；若全区间无效应，kill
- **ROVE 扩展版可能先发**：它加个先验信号就挤进我的格子——对策：快 + 全分类学 + 多源真实数据是它没有的
- **合成腐蚀 ≠ 真实噪声**：LIBERO 结论可能不迁移——Testbed 2 就是为此设的
- **`~/recap` 成熟度**：pipeline 能不能端到端跑通 LIBERO 还未验证（我自己最清楚，先跑通 vanilla 基线是第 0 步）
- **审稿人视角**："这不就是 data cleaning 吗"——回应必须靠 ④>② 那条判决（角色映射 > 剔除），没有它故事不成立

**Kill criteria**：三个腐蚀比例下 ④ ≈ ② 且 C 无 V 校准改善 → 本主线降级，回 [[Diffusion-Flow 策略的 RL 改进路线图]] 找下一条。

---

## 6. 与 vault / 现实资产的挂钩

- **概念**：[[Advantage-conditioned 微调]]（三类数据表 = 本主线的攻击面）｜LDA-1B 的「按数据质量分配角色」概念候选（本主线是它在 advantage 标签上的落地，读完 LDA-1B 后正式建概念笔记）
- **地图**：[[强化学习后训练 主题地图]] §4 问题①「value 模型脆」——设计 C 是直接回应
- **队列**：[[paper-list]] P0 = π*₀.₆/RECAP（宿主方法精读）+ ROVE（头号近邻）+ LDA-1B（同思想他域）
- **代码/数据**：`~/recap`（testbed）· `~/data_filter`（质量信号源，注意其 memory：筛选结果接入训练脚本是「未做」项——**那个未做项正是本主线的接口**）· 实验室 FTC/real-world-rl（真机验证出口）
- **远期搁置**（不删，标记为 future work）：QAM 内化 × 世界模型价值线——已被 Q-VGM / GigaBrain-0.5M* 部分占位，见 [[Diffusion-Flow 策略的 RL 改进路线图]] §3

---

## Backlinks
（Obsidian 自动维护）
