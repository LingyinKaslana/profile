# CUDA 并行计算：Blelloch Scan 与鸟群仿真

> 把前缀和从 O(n log n) 做到 O(n)，再把它用到一个 16,384 只鸟的仿真里。

**类型** 项目 · **角色** 自研实现 · **时间** 2025 · **状态** 已完成

[← 返回首页](../../README.md)

## 挑战

stream compaction 依赖前缀和，倍增实现是 O(n log n)；鸟群仿真的邻居查询是 O(n²)，帧率只有 0.5 fps。

## 做法

- 实现 Blelloch 双阶段 Scan（上扫建树、下扫还原前缀和）用于 stream compaction
- 用 Nsight Systems 做 kernel 级 profiling 定位瓶颈：活跃线程聚集减少 warp 分歧、数据搬入共享内存、错位消除 bank conflict
- 鸟群仿真：空间网格分桶、按格排序、非空格子增量更新

## 结果

- 前缀和由 O(n log n) 降至 O(n)
- 内核耗时快于 CPU 实现
- 16,384 只鸟：0.5 fps → 45+ fps

## 关键数字

| 指标 | 数值 |
| :-- | :-- |
| 前缀和复杂度 | O(n log n) → O(n) |
| 仿真帧率 | 0.5 → 45+ fps |
| 仿真实体数 | 16,384 |
