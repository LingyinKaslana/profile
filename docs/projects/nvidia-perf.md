# 英伟达半导体科技（上海）｜ PerfWorks / PerfSDK 性能监测团队

> 驱动 GPU 硬件计数器采样、并将原始 counter 解码为指标，为 Nsight System 提供底层支持。

**时间** 2026.03 – 2026.11 · **角色** Software Engineer Intern

[← 返回首页](../../README.md)

## 轻量级 GPU 性能监测（Monitoring）

- 实现跨平台命令行、可配置指标采集与 UI 展示的 GPU 指标监测工具，100 fps 采样下 CPU 占用通常低于 2%，随 [Nsight Perf SDK 2026.3](https://developer.nvidia.com/nsight-perfsdk/getting-started/release-note-v2026.3) 发布。

![Monitoring Tool：敲一条命令，直接看 GPU 实时计数器](../../assets/monitoring-demo.gif)

### 技术实现

- **采样**：基于 Nsight Perf SDK 的 periodic sampler，按固定周期向 GPU 取硬件计数器（约 10 ms 一次，对应 100 fps 的刷新粒度）；一次请求批量读取一组 metric，避免逐 counter 触发查询。
- **线程模型**：采样与渲染分线程，中间用预分配的环形缓冲传递快照；渲染端只读最新一帧，终端刷新慢不会反向拖慢采样。
- **终端 UI**：跨平台终端抽象，用 ANSI 转义序列做整屏重绘与差异刷新，只重画数值变化的行，避免全屏刷新的终端开销。
- **可配置指标**：启动时读取配置/参数里的 metric 列表，先用 CounterAvailability 过滤当前 GPU 不支持的指标，再组装采样请求，指标名与显示格式分开配置。
- **低开销**：热路径零分配、缓冲区复用，采样不被渲染阻塞；CPU 侧只做增量更新与阈值判断，因此 100 fps 下占用通常低于 2%。
- **跨平台**：命令行解析、计时器、终端尺寸与按键处理各抽一层，Windows 与 Linux 共用同一套采样和渲染逻辑。

### 难点

- 有性能要求，所以需要无锁编程，以及一些常见的代码优化。
- 如果 CPU 卡住，GPU 端的 PMA buffer 会溢出并停止采样，需要自动重启采样。
- Monitoring 是“低频”采样工具，很多 counter 不能用，需要过滤（如 `graphics-low-speed-1.yaml`）。

## Profiler 离线解码（Offline Decode, CUDA 13.5-nsight-perf-sdk-for-ai 的核心之一）

- 定位到采样与搬运开销极低、成本几乎全在解码：解码与被测负载争抢同一 CPU，既干扰观测，又拖慢采样。
- 将解码移出采集路径——采集侧只把 counter 落成 Snapshot，事后重建 Decoder 离线解码，采样不再受解码拖累，可全速跑满全部 counter。
- 解决复现问题：重采拿不到相同数据、问题难以复现，离线解码使排查回到同一份原始数据。

## 存量代码的 AI 审查（AI Code Review）

- 存量千万行 C/C++ 的全量 AI 审查不可行——可疑点远超人工确认能力；提出只筛高置信、与业务无关的局部缺陷，经 10 万行抽样验证，检出 6 个高置信 bug、成本不到 1 美元，外推全仓不到 500 美元，极大减轻人工排查压力。
