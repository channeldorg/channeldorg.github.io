# ChanneldUE 插件
ChanneldUE是为虚幻引擎专用服务器提供分布式模拟能力的开源插件。

![](../images/benchmark_entity_lod.gif)

<img src="images/cross_server_physics.gif" height="270px">

## 特性
- 无需修改引擎代码，便可将单个UE专用服务器的最大承载人数提升到100-200人 ([性能测试](zh/benchmark.md))
- 可以将多个UE专用服务器组合成一个大世界，支持上千玩家同时在线
- 支持多种应用场景，包括无缝大世界，以及传统的多房间架构和中转服务器架构
- 开箱即用的同步方案，与原生UE的开发方式无缝集成
- 灵活且可扩展的[客户端兴趣管理](zh/client-interest.md)机制
- 支持跨服交互（目前仅支持RPC、移动和刚体跨服；AI、GAS等系统的跨服需要额外集成）
- 在客户端不需要重连或重新开始的情况下进行[专用服务器灾难恢复](zh/disaster-recovery.md)
- 基于云计算的动态负载均衡能够极大节省服务器成本（开发中）
- 支持[一键上云](zh/cloud-deployment-tool.md)

## 引擎版本支持
| 覆盖功能 | UE 4.27.2 | UE 5.3.2 ~ 5.6.1 |
| ------ | ------ | ------ |
| 快速开始文档 | :white_check_mark: | :white_check_mark: |
| 示例项目 | :white_check_mark: | :white_check_mark: |
| 代码生成工具 | :white_check_mark: | :white_check_mark: `*` |
| 云部署工具 | :white_check_mark: | :x: |

`*` 开启`Live Coding`功能后，UE编辑器可以正常编译和加载生成的代码，但是其它PIE服务器和客户端进程不会加载补丁，导致无法正常运行（见[故障排查](zh/troubleshooting.md?id=注册频道数据类型失败)）。因此，建议关闭`Live Coding`功能。
