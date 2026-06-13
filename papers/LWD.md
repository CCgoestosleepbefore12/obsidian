# LWD: Learning while Deploying

**Tags**: #paper #VLA #offline-to-online-RL #fleet
**导读**: paper-world/levels/lwd.html（交互式精读）

---

## 1. 一句话总结

fleet-scale **offline-to-online RL** 持续后训练通用 VLA 策略：16 台双臂机器人车队「部署即学习」的 data flywheel——自主 rollout + 人类干预喂回共享策略。两大件：**DIVL**（分布式隐式价值学习，保留多峰重尾回报里的罕见成功）+ **QAM**（Q-learning via Adjoint Matching，把 critic 梯度转成 flow 策略的逐步监督）。offline/online 同一 RL 目标，平均成功率 **95%**，长程任务增益最大。

## 2. 跟我的关系

- 我的方向：[[RL 与模型后训练]]
- 对照 [[RISE  Self-Improving Robot Policy with Compositional World Model|RISE]]——同样做 VLA 的 RL 后训练，但 RISE 在「想象空间」（世界模型）里练，LWD 在「真实车队」里 offline-to-online 练。
- 相关：[[Advantage-conditioned 微调]]、[[RECAP]]、[[π₀.₅]]

## 3. 待理解 / 存疑

- [ ] QAM 的 adjoint matching 完整损失式（Eq.9）
- [ ] DIVL 的 categorical 分布离散化细节
- [ ] n-step TD（长程 n=10）的冷启动机制
