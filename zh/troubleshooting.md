# 故障排查
## Setup脚本没有下载channeld
如果您在运行Setup前，已经配置了环境变量`%CHANNELD_PATH%`，Setup脚本会认为您已经安装了channeld，并跳过下载安装步骤。插件会根据环境变量`%CHANNELD_PATH%`来运行channeld。

如果您需要下载安装channeld到插件目录中，请先删除环境变量`%CHANNELD_PATH%`，再运行Setup。

## 无法启动channeld服务
- 检查任务管理器中是否存在channeld进程，如果存在，请手动“结束任务”
- 如果输出日志中出现`failed to listen`的错误，说明端口被占用。请检查默认的端口12108和11288是否被其它程序占用，或者在channeld配置文件中修改端口号

## 游戏服务器启动后自动退出
- 检查channeld服务是否正常运行。在开启了channeld网络（`Enable Channeld Networking`）时，游戏服务器会尝试连接channeld服务。如果连接失败，游戏服务器会自动退出；
- 确认[Live Coding](https://docs.unrealengine.com/5.0/en-US/using-live-coding-to-recompile-unreal-engine-applications-at-runtime/)的设置是关闭的，否则会日志中会出现`Error: Failed to register channel data type by name`；
- 如果上述方法仍无法解决，请检查游戏服务器的日志。日志文件通常位于项目目录下的`Saved/Logs`目录中，以项目名_{数字}命名。在单服模式下，数字为2是游戏服务器日志；在多服模式下，数字为2是主服务器日志，数字从3开始是空间服务器日志。

## 无法保存蓝图
如果出现“无法保存资产”的错误提示，通常是由于游戏服务器仍在运行，导致蓝图文件被占用。请先关闭游戏服务器，再保存蓝图。

## 第二个PIE客户端无法进入场景
查看UE服务端的日志，如果出现如下信息：
```log
LogNetTraffic: Error: Received channel open command for channel that was already opened locally.
```
请检查`编辑器偏好设置 -> 关卡编辑器 -> 播放 -> Multiplayer Options`中的`单进程下的运行`，确保为**未勾选**的状态。

## 同步出现问题
- 确保使用最新生成的同步代码
- 有时候，生成的同步代码在编译后没有正常被热加载。此时需要重新编译并启动UE编辑器。
- 检查游戏服务器的日志，查看是否有同步相关的错误信息

## 刷新同步缓存后，无法找到Actor状态
如果同步Actor的状态在Channel Data Schema编辑器的`Add State`上下文菜单中没有出现，请尝试删除项目目录下的`Intermediate`目录，然后重新编译项目，启动UE编辑器。再次刷新同步缓存，状态应该会出现了。

## 生成的同步代码编译失败
该问题的产生可能是ChanneldUE插件的分支切换导致的。解决的方法是手动删除项目目录下的`Source/<项目名>/ChanneldGenerated`目录，然后重新编译项目，启动UE编辑器，再次生成同步代码。

## 角色跨服后，亮边的颜色不正确
检查空间服务器是否都在正常运行。如果空间服务器已经退出，其控制的空间频道区域就无法正常模拟，跨服也会出现问题。

## 项目中存在其它Protobuf库的冲突
ChanneldUE插件使用了Protobuf库，并以ProtobufUE模块的方式进行引用。如果您的项目中也使用了Protobuf库，则需要将ChanneldUE插件或项目对ProtobufUE模块的引用改为您自己的模块，并重新编译。注意ChannelUE引用Protobuf使用的路径是`google/protobuf/*.h`，如果您使用的Protobuf库的根路径不同，会导致ChanneldUE插件编译错误。

## 项目出现资产“空引擎版本”警告
如果项目中出现类似如下的警告信息：
```log
LogLinker: Warning: Asset 'Your_Project_Root/Plugins/channeld-ue-plugin/Content/Blueprints/BP_SpatialRegionBox.uasset' has been saved with empty engine version. The asset will be loaded but may be incompatible.
```
请在项目的`Config/DefaultEngine.ini`中添加如下内容：
```ini
[Core.System]
ZeroEngineVersionWarning=False
```