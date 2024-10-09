# 灾难恢复
对于需要上线运营的游戏来讲，服务器的灾难恢复是一个必须要考虑的问题。运行中的游戏服务器的崩溃，轻则造成一局MOBA或大逃杀游戏的意外终止，重则造成MMORPG中玩家数据的回档甚至虚拟财产的损失。本章介绍ChanneldUE对于灾难恢复的支持。

## 专用服务器
ChanneldUE支持专用服务器在崩溃重启后，根据断开前保留在频道中的订阅和数据状态，恢复到崩溃前的状态。这样客户端就不需要在服务器重启后走重新连接和初始化的流程，从而保证玩家体验的连续性。同时，所有需要同步的状态数据都会在崩溃前后保持一致，如玩家和世界首领的位置等。需要注意的是，服务器上非同步的状态仍然会丢失或重置，如AI的移动目标点。如果这些状态也需要在崩溃后恢复，要通过额外的代码来实现。

> 目前，崩溃后的重启需要手动完成。未来计划会更新云部署工具以实现自动重启。

### 开启灾难恢复功能
要开启服务器的灾难恢复功能，需要在channeld网关服务的启动参数中加入`-scr=true`。下图展示如何在UE中设置全局的灾难恢复参数：

![](../images/channeld-launch-param-scr.png)

下图展示在设置了[世界场景设置类](world-settings.md)后，如何设置关卡的灾难恢复参数覆盖：

![](../images/channeld-launch-param-scr-override.png)

### 增加客户端暂停逻辑
在服务器崩溃后，到完全恢复状态之前，channeld网关服务器会丢弃所有转发给该服务器的消息。在此期间，客户端也应该暂停游戏逻辑，停止向服务器发送任何消息（除了断开消息）。channeld网关服务器会向客户端发送以下消息，用于处理暂停逻辑：

1. CHANNEL_OWNER_LOST（频道所有者丢失）：频道的所有者即客户端发送给该频道消息的处理者，一般为服务器连接。在[基本概念](basic-concepts.md)中介绍过，一个服务器连接可以是多个频道的所有者。当服务器异常断开后，其拥有的频道会广播CHANNEL_OWNER_LOST消息。客户端在收到该消息时，首先应该根据频道类型来判断是否需要暂停游戏。一般来讲，全局、子世界或空间频道需要暂停游戏；实体频道则不需要暂停游戏。

> 提示：channeld允许配置哪些频道类型在失去所有者后会广播CHANNEL_OWNER_LOST消息。在channeld的根目录下的config文件夹找到`channeld_settings_ue.json`，修改各个频道类型的`SendOwnerLostAndRecovered`字段即可。

2. CHANNEL_OWNER_RECOVERED（频道所有者恢复）：当服务器恢复后，其拥有的频道会广播CHANNEL_OWNER_RECOVERED消息。客户端在收到该消息时，应该取消暂停游戏。

以下蓝图演示了通过`ChanneldGameInstanceSubsystem`来处理频道所有者丢失和恢复事件：

![](../images/channel-owner-lost-recovery.png)

该蓝图位于[ChanneldUE示例项目](https://github.com/channeldorg/channeld-ue-demos)中，路径为`Content/Blueprints/BP_RecoveryHandler`。

将该蓝图放置到关卡中，可以实现如下效果：

![](../images/channeld-server-recovery.gif)


### 自定义频道恢复逻辑
ChanneldUE内置了全局、子世界、空间和实体四种频道类型的恢复逻辑。如果想要对其他频道类型进行灾难恢复逻辑的扩展，需要按照下面的方式增加代码。

如果频道的数据恢复逻辑比较简单，通过完整的频道数据就可以进行恢复，那么只需要在UE项目中增加一个视图类，重载`UChannelDataView::RecoverChannelData`函数即可；

如果频道的数据恢复逻辑比较复杂，需要额外的数据来进行恢复，那么还需要在channeld网关服务中增加一个频道数据扩展类型，重载`channeld.ChannelDataExtension`接口。例如：私有频道保存了玩家的背包数据，但是背包中物品的购买时间不需要同步到客户端，因此不在频道数据中。而服务器依赖于购买时间来判断物品能否退还。因此在购买逻辑中，需要将购买时间保存在私有频道数据扩展中：

```go
type PrivateChannelDataExtension struct {
	channeld.ChannelDataExtension

	itemPurchaseTime map[uint32]int64
}

func (ext *PrivateChannelDataExtension) Init(ch *channeld.Channel) {
	itemPurchaseTime := make(map[uint32]int64)
}

func (ext *PrivateChannelDataExtension) GetRecoveryDataMessage() common.Message {
	return &mygamepb.ChannelRecoveryData{
		ItemPurchaseTime: ext.spawnedObjs,
	}
}
```

然后，在`examples\channeld-ue-tps\main.go`中注册该频道数据扩展类型：

```go
	channeld.SetChannelDataExtension[PrivateChannelDataExtension](channeldpb.ChannelType_PRIVATE)
```

最后，在`UChannelDataView::RecoverChannelData`重载函数里读取`ChannelRecoveryData`消息，从而恢复服务器状态。

## channeld网关服务
channeld网关服务尚未实现灾难恢复功能。这意味着，如果channeld网关服务崩溃，所有状态都会丢失；系统重新上线需要重启channeld网关服务和所有专用服务器，并重新连接客户端。

计划在未来的版本中，首先实现channeld的状态持久化，然后实现连接和订阅的恢复。