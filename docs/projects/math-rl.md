# 单卡 1.3B 数学推理训练：CPT → SFT → DAPO

> 在一张 GB300 上把 1.3B 模型的 GSM8K 从 4.78% 拉到 23.73%，重点验证单卡训练与 rollout 到底怎么省时间。

**类型** 项目 · **角色** 独立实现 · **时间** 2026 · **状态** 已完成

[← 返回首页](../../README.md)

## 一、设置

- **模型**：DeepSeek-Coder-1.3B-Base；**硬件**：单张 NVIDIA GB300，约 251 GiB 显存；**精度**：参数 / 梯度 / Adam 状态用 FP32，BF16 autocast 计算。
- **评测**：GSM8K 1,319 题、MATH 5,000 题，官方 few-shot CoT、greedy、1,024 token，错误与超时都计入分母。

## 二、三个阶段

- **CPT（数学继续预训练）**：OpenWebMath 约 5B 语料跑两遍，共 **10B input token**，全参数训练；验证 loss 2.1084 → 1.7537。
- **SFT**：GSM8K + MATH 共 14,459 条训练、512 条验证，3 epoch / 1,356 steps；取第 2 epoch 的最佳点 **step 904**（val loss 0.7081）。
- **DAPO（强化学习）**：GSM8K，每题 8 个回答、每轮 32 个混合对错组（256 条回答）、每轮 4 次 optimizer update，生成上限 2,048 token，学习率 1e-6；跑完 **2,652 轮 / 10,608 次更新 / 84,864 个有效组**。

## 三、结果

| 检查点 | GSM8K | MATH |
| :-- | --: | --: |
| 原始 DeepSeek-Coder-1.3B | 4.78% | 4.16% |
| 数学 CPT 10B | 8.64% | 5.64% |
| SFT step 904 | 21.30% | 6.14% |
| 早期 RL 第 80 轮 | **23.73%** | 5.34% |

- RL 早期 GSM8K 涨了，但 MATH 从 6.14% 回落到 5.34%——这个负结果保留在表里，没有藏。
- 长训练换了另一套口径（256 题 held-out、zero-shot、2,048 token）：最佳第 800 轮 **41.41%**，第 2,600 轮 39.06%。

## 四、单卡 rollout 优化

- **完成即移出解码 batch**：不等整批最长序列结束，已完成的回答立刻退出后续 decode（没做完整的 continuous batching）。
- **microbatch 调优**：256 条回答切成 4 个 64-response minibatch，microbatch 取 16。
- **profiling 定位**：先用 profiler 看清 decode 阶段的耗时分布，再决定优化顺序，避免把 CPU 观测和队列等待混进 decode 计时。
- **效果**：最近 100 轮约 **265 秒/轮**（13.6 轮/小时）。
- **按历史难度路由选题**：对每题历史正确率做 EMA，**75% 优先选历史上还有对有错的题、25% 随机探索**。同一权重、正式 256 路采样下：uniform 108.23 s vs history 57.63 s，有效组产出约 **1.88 倍**（64 路小并发约 3.50 倍）。

## 五、局限

- 单卡、小模型、小数据的流程实验，不能当作方法有效性的论文级证据；
- GSM8K 训练题同时用于 SFT 和 RL，不做「泛化」或「RL 单独带来提升」的归因；
- 1.88 倍是短期采样对比，训练后期大量题目变成全对、有效组比例下降，倍率不保持。

## 参考

- [DeepSeekMath-Small（训练工作区快照）](https://github.com/LingyinKaslana/DeepSeekMath-Small)
- [DeepSeekMath](https://arxiv.org/abs/2402.03300) · [DAPO](https://arxiv.org/abs/2503.14476) · [OpenWebMath](https://huggingface.co/datasets/open-web-math/openwebmath)
